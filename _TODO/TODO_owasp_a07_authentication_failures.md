# TODO: OWASP A07:2025 — Kimlik Dogrulama Hatalari

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A07:2025 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Kimlik_Dogrulama/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A07_Authentication_Failures.md](../OWASP_Top_10/A07_Authentication_Failures.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Kimlik Dogrulama Hatalari, kullanici kimligini dogrulamak icin kullanilan mekanizmalardaki zafiyetleri kapsar. Zayif parola politikalari, duz metin parola saklama, brute force korumasizligi ve MFA eksikligi gibi sorunlar bu kategoriye girer. OWASP Top 10:2025'de 7. sirada yer alir. Basarili istismar durumunda saldirganlar kullanici hesaplarina yetkisiz erisim saglayabilir.
*   **Etkilenen bilesenler:** Kimlik dogrulama sistemleri, parola yonetimi, oturum yonetimi, cok faktorlu kimlik dogrulama (MFA), API kimlik dogrulama mekanizmalari

---

### 2. Teknik detay (nasil calisiyor)
*   Kimlik dogrulama hatalari, kullanici kimligini dogrulamak icin kullanilan mekanizmalardaki zayifliklarin istismar edilmesiyle ortaya cikar.
*   Yaygın ortaya cikis bicimleri:
    *   Zayif parola politikalari (kisa, basit parolalara izin verilmesi)
    *   Parolalarin duz metin veya zayif hash algoritmalarilya saklanmasi (MD5, SHA1)
    *   Brute force saldirilarina karsi koruma olmaması (rate limiting eksikligi)
    *   Cok faktorlu kimlik dogrulama (MFA) eksikligi
    *   Oturum token'larinin guvenli yonetilmemesi
*   **Neden:** Guvenli kimlik dogrulama mekanizmalarinin dogru sekilde uygulanmamasi, guvenlik en iyi pratiklerinin takip edilmemesi ve modern saldiri tekniklerine karsi onlem alinmamasi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Web uygulamasi giris paneli
*   **2. Normal durum:**
    ```
    1. Kullanici kullanici adi ve parola girer
    2. Sistem kimlik dogrulamasi yapar
    3. Basarili ise oturum acilir, basarisiz ise hata mesaji gosterilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, admin kullanici adini belirler (ornegin "admin")
    2. Yaygin parola listesi (rockyou.txt) ile brute force araci calistirir
    3. Rate limiting olmadigi icin saniyede yuzlerce deneme yapilir
    4. Komut: hydra -l admin -P rockyou.txt target.com http-post-form "/login:user=^USER^&pass=^PASS^:F=Hatali"
    ```
*   **4. Analiz:** Uygulama, basarisiz giris denemelerine karsi herhangi bir sinirlandirma uygulamadigi icin saldirgan sinirsiz sayida parola denemesi yapabilir.
*   **5. Kanit:** Saldirgan, admin parolasini basariyla bulur ve yonetici paneline erisim saglar.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, API endpointleri, Giris panelleri
*   **Karmasiklik:** Dusuk

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Rate limiting uygulama, hesap kilitleme mekanizmasi ekleme, mevcut zayif parolalari tespit etme ve degistirmeye zorlama.
*   **Orta vadeli:** Bcrypt/Argon2 ile parola hashleme, MFA entegrasyonu, guvenli oturum yonetimi (HttpOnly, Secure, SameSite cookie flag'leri).
*   **Uzun vadeli:** Passwordless kimlik dogrulama (WebAuthn/FIDO2), merkezi kimlik yonetimi (SSO/OIDC), duzenli parola politikasi guncellemeleri ve guvenlik egitimleri.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```javascript
// Guvenli olmayan giris islemi — rate limiting yok, zayif parola saklama
const express = require('express');
const app = express();

app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    const user = await db.findUser(username);

    // Parola duz metin olarak karsilastiriliyor
    if (user && user.password === password) {
        req.session.userId = user.id;
        res.json({ success: true });
    } else {
        res.status(401).json({ error: 'Hatali kullanici adi veya parola' });
    }
});
```

**Guvenli kod:**
```javascript
const express = require('express');
const bcrypt = require('bcrypt');
const rateLimit = require('express-rate-limit');
const app = express();

// Rate limiting: 15 dakikada en fazla 5 basarisiz giris denemesi
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    message: { error: 'Cok fazla basarisiz deneme. Lutfen 15 dakika sonra tekrar deneyin.' },
    standardHeaders: true,
    legacyHeaders: false,
});

app.post('/login', loginLimiter, async (req, res) => {
    const { username, password } = req.body;
    const user = await db.findUser(username);

    // Bcrypt ile guvenli parola dogrulama
    if (user && await bcrypt.compare(password, user.passwordHash)) {
        // Guvenli oturum ayarlari
        req.session.regenerate((err) => {
            if (err) return res.status(500).json({ error: 'Oturum olusturulamadi' });
            req.session.userId = user.id;
            req.session.cookie.httpOnly = true;
            req.session.cookie.secure = true;
            req.session.cookie.sameSite = 'strict';
            req.session.cookie.maxAge = 30 * 60 * 1000; // 30 dakika
            res.json({ success: true });
        });
    } else {
        // Zamanlama saldirisini onlemek icin sabit sureli yanit
        await bcrypt.hash('dummy', 10);
        res.status(401).json({ error: 'Hatali kullanici adi veya parola' });
    }
});
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Mevcut kimlik dogrulama mekanizmalarini analiz et
*   [ ] Parola politikasini gozden gecir
*   [ ] Brute force korumasinin varligini kontrol et
*   [ ] MFA uygulamasinin durumunu degerlendir

**Duzeltme**
*   [ ] Guclu parola politikasi uygula (minimum 12 karakter, karmasiklik gereksinimleri)
*   [ ] Parolalari bcrypt/Argon2 ile hashle
*   [ ] Rate limiting ve hesap kilitleme mekanizmasi ekle
*   [ ] MFA entegrasyonunu tamamla
*   [ ] Oturum cookie'lerini guvenli yapilandir (HttpOnly, Secure, SameSite)

**Dogrulama**
*   [ ] Brute force testi gerceklestir
*   [ ] Parola hash mekanizmasini dogrula
*   [ ] MFA is akisini test et
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Basarisiz giris denemelerindeki artislari izle
    *   Anormal konum veya zamanlarda yapilan giris denemelerini tespit et
    *   Regex: `(?i)(failed login|authentication failure|invalid credentials|brute.?force)`
*   **Anomali Tespiti:**
    *   Tek bir IP adresinden gelen cok sayida basarisiz giris denemesi
    *   Anormal cografi konumlardan giris denemeleri
    *   Normal calisma saatleri disinda yapilan giris denemeleri
    *   Ayni hesaba farkli IP'lerden es zamanli erisim

---

## Notlar
- OWASP Top 10:2025'de 7. sirada yer alir.
- Detayli rehber: [A07_Authentication_Failures.md](../OWASP_Top_10/A07_Authentication_Failures.md)
