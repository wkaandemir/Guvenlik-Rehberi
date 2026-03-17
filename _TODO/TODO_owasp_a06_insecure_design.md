# TODO: OWASP A06:2025 — Guvensiz Tasarim

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A06:2025 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | OWASP_Top_10/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A06_Insecure_Design.md](../OWASP_Top_10/A06_Insecure_Design.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Guvensiz Tasarim, uygulamanin temel mimarisinde veya tasariminda guvenlik kontrollerinin eksik olmasi veya yanlis uygulanmasi nedeniyle ortaya cikan bir guvenlik zafiyetidir. Kod implementasyonundaki hatalardan ziyade tasarim seviyesindeki eksiklikleri ifade eder. OWASP Top 10:2025'de 6. sirada yer alir. Duzeltilmesi en zor ve en maliyetli guvenlik sorunlarindan biridir.
*   **Etkilenen bilesenler:** Uygulama mimarisi, is akislari, veri modelleri, API tasarimi, oturum yonetimi, hata yonetimi, yetkilendirme mekanizmalari

---

### 2. Teknik detay (nasil calisiyor)
*   Guvensiz tasarim, uygulamanin temel mimarisinde guvenlik kontrollerinin eksik olmasi veya yanlis uygulanmasidir.
*   Yaygın ortaya cikis bicimleri:
    *   Threat modeling (tehdit modelleme) yapilmamasi
    *   Guvenli varsayimlarla tasarim yapilmasi
    *   Hata durumlarinin guvenli bir sekilde ele alinmamasi
    *   Is akislarinda guvenlik kontrollerinin eksik olmasi
    *   Minimum yetki ilkesinin uygulanmamasi
*   **Neden:** Guvenlik odakli tasarim prensiplerinin ihmal edilmesi, threat modeling sureclerinin eksikligi, guvenlik uzmanlarinin tasarim asamasina dahil edilmemesi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Parola sifirlama is akisi
*   **2. Normal durum:**
    ```
    1. Kullanici parola sifirlama talebinde bulunur
    2. Sistem kullaniciya e-posta ile gecici bir link gonderir
    3. Kullanici link uzerinden yeni parola belirler
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, parola sifirlama linkini tahmin edilebilir oldugunu fark eder
    2. Link formati: https://example.com/reset?token=USERID_TIMESTAMP
    3. Saldirgan, USERID ve TIMESTAMP degerlerini degistirerek farkli kullanicilarin parolasini sifirlamayi dener
    ```
*   **4. Analiz:** Parola sifirlama mekanizmasi, token'lari tahmin edilebilir bir yapida olusturur ve yeterli guvenlik kontrolleri icermez.
*   **5. Kanit:** Saldirgan, baska bir kullanicinin parolasini basariyla sifirlar ve hesabina erisim saglar.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, Ic network, Uygulama mimarisi
*   **Karmasiklik:** Orta

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Mevcut tasarim zafiyetlerini belirleme ve gecici cozumler uygulama, kritik is akislarina ek guvenlik kontrolleri ekleme.
*   **Orta vadeli:** Threat modeling surecleri uygulama, guvenli tasarim prensiplerini benimseme, guvenlik denetimlerini guclendirme.
*   **Uzun vadeli:** Guvenlik odakli tasarim kulturu olusturma, duzenli guvenlik tasarim incelemeleri yapma, gelistiricilere guvenli tasarim egitimi verme.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```java
// Tahmin edilebilir token olusturma
public void resetPassword(String email) {
    User user = userRepository.findByEmail(email);
    if (user != null) {
        String token = user.getId() + "_" + System.currentTimeMillis();
        user.setResetToken(token);
        userRepository.save(user);
        emailService.sendResetEmail(email, token);
    }
}
```

**Guvenli kod:**
```java
public void resetPassword(String email) {
    User user = userRepository.findByEmail(email);
    if (user != null) {
        // Guclu ve rastgele token olusturma
        SecureRandom random = new SecureRandom();
        byte[] bytes = new byte[64];
        random.nextBytes(bytes);
        String token = Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);

        user.setResetToken(token);
        user.setTokenExpiration(LocalDateTime.now().plusMinutes(30));
        userRepository.save(user);
        emailService.sendResetEmail(email, token);
    }
    // Kullanici bulunmasa bile ayni mesaji dondurerek enumeration saldirisini onle
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Mevcut uygulama mimarisini ve is akislarini analiz et
*   [ ] Threat modeling surecleri uygula
*   [ ] Guvenlik gereksinimlerini belirle

**Duzeltme**
*   [ ] Guvensiz tasarim desenlerini belirle ve duzelt
*   [ ] Minimum yetki ilkesini uygula
*   [ ] Guvenli hata yonetimi mekanizmalari kur
*   [ ] Is akislarina guvenlik kontrolleri ekle

**Dogrulama**
*   [ ] Tasarim incelemeleri yap
*   [ ] Guvenlik testleri gerceklestir
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Is akislarindaki anormal davranislari izle
    *   Tasarim seviyesindeki guvenlik ihlallerini tespit et
    *   Regex: `(?i)(design flaw|insecure workflow|business logic flaw)`
*   **Anomali Tespiti:**
    *   Normal disi is akisi kullanimlari
    *   Is kurallarinin asilma denemeleri
    *   Tasarim seviyesindeki guvenlik kontrollerinin atlatilmasi

---

## Notlar
- OWASP Top 10:2025'de 6. sirada yer alir.
- Detayli rehber: [A06_Insecure_Design.md](../OWASP_Top_10/A06_Insecure_Design.md)
