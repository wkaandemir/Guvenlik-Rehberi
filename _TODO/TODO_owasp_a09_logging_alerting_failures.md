# TODO: OWASP A09:2025 — Guvenlik Kayit Tutma ve Uyari Hatalari

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A09:2025 |
| **Kritiklik** | Orta |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A09_Security_Logging_and_Alerting_Failures.md](../OWASP_Top_10/A09_Security_Logging_and_Alerting_Failures.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Guvenlik Kayit Tutma ve Uyari Hatalari, guvenlik olaylarinin yeterince loglanmamasi, izlenmemesi ve uyari mekanizmalarinin eksik olmasi nedeniyle saldirilarin zamaninda tespit edilememesi sorununu kapsar. Yetersiz loglama, saldirganin faaliyetlerinin fark edilmesini engelleyerek saldirilarin uzun sureler boyunca devam etmesine neden olur. OWASP Top 10:2025'de 9. sirada yer alir.
*   **Etkilenen bilesenler:** Loglama altyapisi, SIEM sistemleri, uyari mekanizmalari, olay mudahale surecleri, denetim kayitlari

---

### 2. Teknik detay (nasil calisiyor)
*   Guvenlik kayit tutma ve uyari hatalari, kritik guvenlik olaylarinin yeterli detayda loglanmamasi ve bu loglarin etkin bir sekilde izlenmemesi durumunda ortaya cikar.
*   Yaygın ortaya cikis bicimleri:
    *   Kritik olaylarin (basarisiz giris, yetki degisiklikleri, hassas veri erisimi) loglanmamasi
    *   Log kayitlarinda yetersiz detay (IP adresi, kullanici bilgisi, zaman damgasi eksikligi)
    *   Uyari mekanizmalarinin olmamasi veya yanlis yapilandirilmasi
    *   Loglarin yerel olarak saklanmasi ve kolayca manipule edilebilmesi
    *   Log verilerin yeterli sure saklanmamasi
*   **Neden:** Loglama ve izleme altyapisina yeterli yatirim yapilmamasi, guvenlik operasyonlari ekiplerinin yetersizligi ve olay mudahale sureclerinin tanimlanmamis olmasi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Yetersiz loglama yapan web uygulamasi
*   **2. Normal durum:**
    ```
    1. Kullanici giris yapar, basarili ve basarisiz giris denemeleri loglanir
    2. Guvenlik ekibi loglari izler ve anormallikleri tespit eder
    3. Uyari mekanizmasi devreye girer ve ekip mudahale eder
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, brute force saldirisi baslatir (saniyede yuzlerce deneme)
    2. Uygulama, basarisiz giris denemelerini loglamadigindan saldiri fark edilmez
    3. Saldirgan parolayi bulur ve sisteme erisir
    4. Erisim sonrasi yapilan islemler de loglanmaz
    5. Saldiri haftalarca fark edilmeden devam eder
    ```
*   **4. Analiz:** Loglama mekanizmasi basarisiz giris denemelerini kaydetmedigi ve uyari mekanizmasi bulunmadigi icin saldiri tespit edilemez.
*   **5. Kanit:** Saldirgan, sisteme eristikten sonra uzun sure fark edilmeden faaliyet gosterir; olay ancak dissal bir bildirim ile ortaya cikar.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Orta
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Tum uygulama katmanlari, altyapi bilesenleri
*   **Karmasiklik:** Dusuk

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Kritik guvenlik olaylari icin loglama ekle (basarisiz giris, yetki degisiklikleri, hassas veri erisimi), temel uyari mekanizmalari kur.
*   **Orta vadeli:** Merkezi loglama altyapisi kur (ELK Stack, Splunk), SIEM entegrasyonu sagla, otomatik uyari kurallari tanimla, brute force tespiti icin middleware ekle.
*   **Uzun vadeli:** 7/24 guvenlik operasyonlari merkezi (SOC) olustur, olay mudahale planlari tanimla, duzenli log analizi ve guvenlik denetimleri yap, log saklama politikasi belirle.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```javascript
const express = require('express');
const app = express();

// Loglama yok — guvenlik olaylari kayit altina alinmiyor
app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    const user = await db.findUser(username);

    if (user && await verifyPassword(password, user.passwordHash)) {
        req.session.userId = user.id;
        res.json({ success: true });
    } else {
        res.status(401).json({ error: 'Giris basarisiz' });
    }
});
```

**Guvenli kod:**
```javascript
const express = require('express');
const winston = require('winston');
const app = express();

// Merkezi ve yapilandirilmis guvenlik logger'i
const securityLogger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    defaultMeta: { service: 'auth-service' },
    transports: [
        new winston.transports.File({ filename: 'logs/security.log' }),
        new winston.transports.File({ filename: 'logs/security-error.log', level: 'error' }),
    ],
});

// Brute force tespiti middleware
const loginAttempts = new Map();
const BRUTE_FORCE_THRESHOLD = 5;
const BRUTE_FORCE_WINDOW = 15 * 60 * 1000; // 15 dakika

function detectBruteForce(ip) {
    const now = Date.now();
    const attempts = loginAttempts.get(ip) || [];
    const recentAttempts = attempts.filter(t => now - t < BRUTE_FORCE_WINDOW);
    loginAttempts.set(ip, [...recentAttempts, now]);

    if (recentAttempts.length >= BRUTE_FORCE_THRESHOLD) {
        securityLogger.error('Brute force saldirisi tespit edildi', {
            event: 'BRUTE_FORCE_DETECTED',
            ip: ip,
            attemptCount: recentAttempts.length,
            window: '15 dakika',
        });
        return true;
    }
    return false;
}

app.post('/login', async (req, res) => {
    const clientIp = req.ip;
    const { username, password } = req.body;

    // Brute force kontrolu
    if (detectBruteForce(clientIp)) {
        return res.status(429).json({ error: 'Cok fazla deneme. Lutfen bekleyin.' });
    }

    const user = await db.findUser(username);

    if (user && await verifyPassword(password, user.passwordHash)) {
        securityLogger.info('Basarili giris', {
            event: 'LOGIN_SUCCESS',
            username: username,
            ip: clientIp,
            userAgent: req.get('User-Agent'),
            timestamp: new Date().toISOString(),
        });
        req.session.userId = user.id;
        res.json({ success: true });
    } else {
        securityLogger.warn('Basarisiz giris denemesi', {
            event: 'LOGIN_FAILURE',
            username: username,
            ip: clientIp,
            userAgent: req.get('User-Agent'),
            timestamp: new Date().toISOString(),
        });
        res.status(401).json({ error: 'Giris basarisiz' });
    }
});
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Mevcut loglama altyapisini analiz et
*   [ ] Kritik guvenlik olaylarini belirle (basarisiz giris, yetki degisikligi, hassas veri erisimi)
*   [ ] Log saklama gereksinimlerini tanimla

**Duzeltme**
*   [ ] Merkezi loglama altyapisi kur (ELK Stack, Splunk veya benzeri)
*   [ ] Tum kritik olaylar icin yapilandirilmis loglama ekle
*   [ ] Uyari mekanizmalarini yapilandir (brute force, anormal erisim)
*   [ ] Log manipulasyonunu onlemek icin merkezi ve degistirilemez log saklama uygula
*   [ ] SIEM entegrasyonunu tamamla

**Dogrulama**
*   [ ] Loglama kapsamini dogrula (tum kritik olaylar loglanıyor mu)
*   [ ] Uyari mekanizmalarini test et
*   [ ] Log saklama surelerini kontrol et
*   [ ] Olay mudahale tatbikati gerceklestir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Basarisiz giris denemelerindeki ani artislari izle
    *   Anormal erisim desenlerini tespit et (farkli IP'lerden ayni hesaba erisim)
    *   Hassas veri erisim kayitlarini izle
    *   Regex: `(?i)(login failure|unauthorized access|permission denied|access denied|brute.?force)`
*   **Anomali Tespiti:**
    *   Belirli bir IP'den gelen basarisiz giris denemelerinde ani artis
    *   Normal calisma saatleri disinda yapilan erisimler
    *   Yonetici hesaplarinda olagan disi aktiviteler
    *   Log hacminde ani dusus veya artis (log manipulasyonu isareti)

---

## Notlar
- OWASP Top 10:2025'de 9. sirada yer alir.
- 2025 surumunde "Alerting" kelimesinin eklenmesi, uyari mekanizmalarinin onemini vurgular.
- Detayli rehber: [A09_Security_Logging_and_Alerting_Failures.md](../OWASP_Top_10/A09_Security_Logging_and_Alerting_Failures.md)
