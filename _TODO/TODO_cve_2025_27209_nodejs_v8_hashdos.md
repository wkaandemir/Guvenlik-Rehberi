# TODO: CVE-2025-27209 — Node.js V8 HashDoS Zafiyeti

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-27209 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Enjeksiyon/ |
| **Kaynak Arastirma** | [Nodejs_Guvenlik_Acigi_Arastirmasi.md](../Arastirmalar/Nodejs_Guvenlik_Acigi_Arastirmasi.md) |
| **Tarih** | 2025-12-20 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-27209, Node.js'in temelini olusturan V8 JavaScript motorundaki hash tablosu uygulamasinda bulunan bir Hash-flooding Denial of Service (HashDoS) zafiyetidir. Saldirganlar, ozel olarak hazirlanmis ve hash carpismasi (collision) yaratan anahtarlar iceren HTTP istekleri gondererek, V8 motorunun dahili hash tablolarinda O(n^2) karmasikliginda islemlere neden olabilmektedir. Bu durum, sunucunun CPU kaynaklarini tuketir ve hizmet reddi (DoS) durumuna yol acar. Node.js v20, v22, v24 (LTS) ve v25 surumlerini etkileyen bu zafiyet, Aralik 2025'ten Ocak 2026'ya ertelenen guvenlik yamasinin bir parcasi olarak bir aydan fazla yamasiz kalmis, React2Shell krizi sirasinda saldirganlar icin ek bir firsat penceresi olusturmustur.
*   **Etkilenen bilesenler:** Node.js v20.x, v22.x, v24.x (LTS) ve v25.x surumlerinin V8 JavaScript motoru, JSON body parse eden tum Express/Fastify/Koa uygulamalari, URL query string isleyen endpointler, hash tabanli veri yapilarini (Object, Map) yogun kullanan API servisleri

---

### 2. Teknik detay (nasil calisiyor)
*   V8 JavaScript motoru, JavaScript nesnelerini (Object) dahili olarak hash tablosu veri yapisinda depolar. Her anahtar (property name), bir hash fonksiyonundan gecirilerek tablodaki yerine yerlestirilir. Normal kosullarda bu islem O(1) karmasikligindadir ve son derece hizlidir.
*   HashDoS saldirisinda, saldirgan V8'in kullandigi hash fonksiyonunun zayifligindan yararlanarak, ayni hash degerini ureten cok sayida farkli anahtar hesaplar. Bu anahtarlar ayni hash kovasina (bucket) duser ve hash tablosu arama islemi O(1) yerine O(n) karmasikligina, ekleme islemi ise O(n^2) karmasikligina yukselir.
*   Saldirgan, ozel olarak hesaplanmis binlerce anahtari bir JSON body icinde veya URL query parametresi olarak sunucuya gonderir. Node.js'in JSON.parse() veya querystring modulu bu verileri islediginde, V8 motorunun hash tablosu islemi dramatik sekilde yavaslayarak CPU'yu doyurur.
*   Bu zafiyet ozellikle Express.js'in `express.json()` middleware'i gibi varsayilan yapilandirmalarda nesne derinligi ve parametre sayisi sinirlamasi olmadigindan kolaylikla tetiklenebilir.
*   **Neden:** Kok neden, V8 motorunun hash fonksiyonunun belirli girdi kaliplarina karsi yeterli randomizasyona sahip olmamasi ve hash tablosu uygulamasinin carpismaya direncli bir mekanizma (ornegin, randomize edilen seed veya kriptografik hash) barindirmamasidir. Ayrica, Node.js uygulama katmaninda varsayilan istek boyutu ve parametre sinirlamalarinin bulunmamasi, saldiri yuzeyini genisletmistir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Node.js tabanli web API servisi (Express.js, Fastify, Koa vb.) uzerinde JSON body veya query string kabul eden herhangi bir endpoint
*   **2. Normal durum:**
    ```
    1. Istemci, API'ye normal bir JSON istegi gonderir: {"name": "Ali", "age": 30}
    2. Node.js (V8) JSON'u parse eder, her anahtar hash tablosuna O(1) ile eklenir
    3. Islem milisaniyeler icinde tamamlanir ve yanit doner
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```javascript
    // Saldirgan, V8 hash fonksiyonunda carpismaya neden olan anahtarlar hesaplar
    // Ornek: Ayni hash degerini ureten binlerce anahtar
    const maliciousPayload = {};
    // V8 hash collision generator ile uretilmis anahtarlar
    const collisionKeys = generateV8HashCollisions(50000);
    collisionKeys.forEach(key => {
        maliciousPayload[key] = "x";
    });

    // HTTP POST istegi olarak gonder
    fetch('https://hedef-api.com/endpoint', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(maliciousPayload) // ~50.000 carpisan anahtar
    });
    // Sunucu CPU'su %100'e cikarak yanitlamaz hale gelir
    ```
*   **4. Analiz:** V8 motoru gelen JSON'u parse ettiginde, 50.000 anahtarin hepsi ayni hash kovasina duser. Her yeni anahtar eklenirken, mevcut tum anahtarlarla karsilastirilmasi gerekir. Bu islem toplamda yaklasik 50.000 x 50.000 / 2 = 1.25 milyar karsilastirma anlamina gelir. Tek bir HTTP istegi, sunucu CPU'sunu saniyeler hatta dakikalar boyunca mesgul eder. Event loop bloke oldugundan diger tum istekler de yanitlanamaz.
*   **5. Kanit:** Hedef sunucunun CPU kullanimi %100'e cikar ve HTTP yanitlari gecikir veya zaman asimina ugrar. Izleme araclari, V8 motorunda asiri uzun sureli JSON parse islemleri ve event loop gecikmelerini (event loop lag) kaydeder. Tekrarlanan isteklerle sunucu tamamen eristilemez hale gelir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.5 (AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H)
*   **Saldiri Yuzeyi:** Internete acik (JSON API endpointleri, web uygulamalari, GraphQL servisleri)
*   **Karmasiklik:** Dusuk (hash carpismasi hesaplama araci ile otomatik payload uretimi mumkun, kimlik dogrulama gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Node.js'i 7 Ocak 2026 guvenlik surumune guncelleyin (v20.x, v22.x, v24.x veya v25.x yamali surumleri). Tum JSON parse eden endpointlerde istek boyutu sinirlamasi (`limit: '100kb'`) ve parametre sayisi sinirlamasi (`parameterLimit: 1000`) uygulayin. Rate limiting middleware ekleyin (IP basina istek/dakika siniri). Kritik API servislerinin onune reverse proxy (nginx/HAProxy) ile ek boyut ve hiz sinirlamasi koyun.
*   **Orta vadeli:** Express.js `extended: false` ayariyla querystring modulu kullanin (qs yerine daha guvenli). Node.js uygulamalarinda `--max-old-space-size` ve `--max-semi-space-size` ile bellek sinirlamasi yapin. Load balancer'larda istek zaman asimi (timeout) degerlerini 30 saniyenin altinda tutun. Uygulama performans izleme (APM) araclariyla event loop lag metriklerini takip edin.
*   **Uzun vadeli:** V8 motorunun hash tablosu uygulamasinda randomize seed kullanimini zorunlu kilan Node.js surumlerine gecis planlayin. Tum API servislerinde giris dogrulama ve semalastirma (JSON Schema) uygulayin. CI/CD pipeline'inda DoS dayaniklilik testlerini otomatik calistirin. Node.js guvenlik surumu duyurularini otomatik izleyen ve uyari ureten bir surec kurun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```javascript
// === GUVENSIZ: Hash tabanli yapilarda boyut/derinlik siniri yok ===
const express = require('express');
const app = express();

// Varsayilan JSON parser — sinirsiz nesne boyutu ve derinligi
app.use(express.json());

// Varsayilan URL-encoded parser — qs modulu ile sinirsiz parametre
app.use(express.urlencoded({ extended: true }));

app.post('/api/data', (req, res) => {
    // req.body dogrudan islenir — HashDoS'a acik
    const keys = Object.keys(req.body);
    res.json({ count: keys.length });
});

app.listen(3000);
```

**Guvenli kod:**
```javascript
// === GUVENLI: Istek boyutu, parametre siniri ve rate limiting ===
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

// 1. JSON body boyutu ve parametre sayisini sinirla
app.use(express.json({
    limit: '100kb',           // Maksimum body boyutu
    parameterLimit: 1000,     // Maksimum parametre sayisi
    strict: true              // Yalnizca dizi ve nesne kabul et
}));

// 2. URL-encoded veriler icin sinir (qs yerine querystring kullan)
app.use(express.urlencoded({
    limit: '100kb',
    parameterLimit: 100,
    extended: false           // Daha guvenli: ic ice nesne destegi yok
}));

// 3. Rate limiting — IP basina istek siniri
const apiLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 dakika
    max: 100,                   // Pencere basina 100 istek
    standardHeaders: true,
    legacyHeaders: false,
    message: { error: 'Istek siniri asildi, lutfen bekleyin.' }
});
app.use('/api/', apiLimiter);

// 4. Istek zaman asimi — uzun sureli islemleri kes
app.use((req, res, next) => {
    req.setTimeout(10000); // 10 saniye
    next();
});

app.post('/api/data', (req, res) => {
    const keys = Object.keys(req.body);
    if (keys.length > 500) {
        return res.status(400).json({ error: 'Cok fazla parametre' });
    }
    res.json({ count: keys.length });
});

app.listen(3000);
```

```bash
# Node.js'i yamali surume guncelle
node --version
# nvm ile guncelle (7 Ocak 2026 guvenlik surumu):
nvm install --lts
nvm use --lts
# Surum dogrulamasi:
node -e "console.log(process.versions)"
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Node.js surumlerini tum sunucularda kontrol et (v20, v22, v24, v25)
*   [ ] Hash tabanli veri yapilarini yogun kullanan endpointleri belirle (JSON API, GraphQL, query string)
*   [ ] Mevcut istek boyutu ve parametre sinirlamalarini gozden gecir
*   [ ] DoS savunmasizlik testleri icin izole test ortami hazirla
*   [ ] APM/izleme araclarinda event loop lag metriklerinin aktif oldugunu dogrula

**Duzeltme**
*   [ ] Node.js'i 7 Ocak 2026 guvenlik surumune guncelle (tum LTS ve Current dallari)
*   [ ] express.json() ve express.urlencoded() middleware'larinda limit ve parameterLimit ayarla
*   [ ] Rate limiting middleware (express-rate-limit) ekle ve yapilandir
*   [ ] Reverse proxy (nginx/HAProxy) katmaninda istek boyutu ve zaman asimi sinirlamasi uygula
*   [ ] `extended: false` ayariyla querystring modulune gec

**Dogrulama**
*   [ ] HashDoS saldiri simulasyonunu yama sonrasi tekrarla ve CPU etkisini olc
*   [ ] Event loop lag metriklerini yuk altinda dogrula (< 100ms)
*   [ ] Rate limiting kurallarinin dogru calistigini test et
*   [ ] SIEM'de asiri hash hesaplama ve CPU spike uyarilarini dogrula
*   [ ] Tum Node.js surumlerinin yamali oldugunu CI/CD pipeline'inda kontrol et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Node.js uygulamalarinda event loop lag metriklerini izle (> 500ms esik degeri)
    *   JSON parse surelerini ve istek isleme surelerini logla ve anormal artislari tespit et
    *   Regex: `(?i)(event.loop.lag|parse.time|request.timeout|ENOMEM|heap.out.of.memory)`
    *   Regex: `(?i)(JSON\.parse|body-parser|content-length:\s*[1-9]\d{5,})`
    *   Reverse proxy loglarinda asiri buyuk POST body'leri filtrele (Content-Length > 100KB)
*   **Anomali Tespiti:**
    *   Tek IP adresinden kisa surede cok sayida buyuk JSON body iceren POST istekleri
    *   Node.js islemi CPU kullaniminin %90'in uzerine cikmasi ve diger isteklerin zaman asimina ugramasi
    *   Event loop gecikme suresinin normal degerlerin 10 katini asmasi
    *   Istek basina isleme suresinin ortalamadan 100 kat fazla olmasi
    *   Sunucu bellek kullaniminin ani ve asiri artmasi (hash tablosu genislemesi)

---

## Notlar
Guvenlik yamasi 15 Aralik 2025'ten 7 Ocak 2026'ya ertelendi — bu sure zarfinda zafiyet yamasiz kaldi. React2Shell (CVE-2025-55182) krizi sirasinda saldirganlar bu zafiyeti lateral movement ve DoS icin kullanabilir. Node.js Ocak 2026 guvenlik surumundeki 3 yuksek oncelikli zafiyetten biri. CVE-2025-27210 (Windows Path Traversal) ile birlikte degerlendirilmeli.
