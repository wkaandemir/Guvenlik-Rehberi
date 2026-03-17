# TODO: OWASP A10:2025 — Istisnai Durumlarin Yanlis Ele Alinmasi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A10:2025 |
| **Kritiklik** | Orta |
| **Kategori Klasoru** | OWASP_Top_10/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A10_Mishandling_of_Exceptional_Conditions.md](../OWASP_Top_10/A10_Mishandling_of_Exceptional_Conditions.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Istisnai Durumlarin Yanlis Ele Alinmasi, uygulamalarin hata durumlarini guvenli bir sekilde yonetememesi sonucu ortaya cikan guvenlik zafiyetlerini kapsar. Stack trace bilgi sizintisi, fail-open durumlar, kaynak sizintilari ve tutarsiz hata davranislari bu kategorinin baslica sorunlaridir. OWASP Top 10:2025'de 10. sirada yer alan yeni bir kategoridir ve uygulamanin dayanikliligi (resilience) konusuna odaklanir.
*   **Etkilenen bilesenler:** Hata yonetimi mekanizmalari, istisna yakalama bloklari, kaynak yonetimi, API hata yanitlari, guvenlik kontrol mekanizmalari

---

### 2. Teknik detay (nasil calisiyor)
*   Istisnai durumlarin yanlis ele alinmasi, uygulamalarin beklenmedik hata durumlarinda guvenli bir sekilde davranamamasi sonucu ortaya cikar.
*   Yaygın ortaya cikis bicimleri:
    *   Stack trace ve debug bilgilerinin kullaniciya gosterilmesi (bilgi sizintisi)
    *   Fail-open durumlar: hata durumunda guvenlik kontrollerinin devre disi kalmasi
    *   Kaynak sizintilari: hata durumunda veritabani baglantilari, dosya taniticilari gibi kaynaklarin serbest birakilmamasi
    *   Tutarsiz hata davranislari: farkli hata turleri icin farkli davranislar sergilenmesi (kullanici enumeration)
*   **Neden:** Hata yonetimi mekanizmalarinin sistematik olarak tasarlanmamasi, "happy path" odakli gelistirme yaklasimi, kaynak yonetimi en iyi pratiklerinin uygulanmamasi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Hata yonetimi zayif olan web uygulamasi
*   **2. Normal durum:**
    ```
    1. Kullanici gecerli parametrelerle istek gonderir
    2. Uygulama istegi isler ve sonuc dondurur
    3. Hata durumunda genel bir hata mesaji gosterilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, hatali parametre gonderir: GET /api/user?id=abc' OR '1'='1
    2. Uygulama islenmeyen bir istisna firlatir
    3. Stack trace yanit olarak dondurulur:
       java.sql.SQLException: Invalid parameter
         at com.app.dao.UserDAO.findById(UserDAO.java:42)
         at com.app.controller.UserController.getUser(UserController.java:28)
       Database: PostgreSQL 15.2
       Server: Apache Tomcat 10.1.5
    4. Saldirgan, veritabani turu, sunucu surumu ve kod yapisini ogrenir
    ```
*   **4. Analiz:** Uygulama, hata durumunda stack trace bilgisini filtrelemeden kullaniciya dondurdugu icin saldirgan iç yapi hakkinda detayli bilgi elde eder.
*   **5. Kanit:** Saldirgan, elde ettigi bilgilerle (veritabani turu, framework surumu, sinif isimleri) daha hedefli saldirilari planlar.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Orta
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, API endpointleri, Hata sayfalari
*   **Karmasiklik:** Dusuk

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Uretim ortaminda stack trace gosterimini devre disi birakma, genel hata sayfalari olusturma, kritik hata durumlarinda fail-closed prensibi uygulama.
*   **Orta vadeli:** Merkezi hata yonetimi mekanizmasi (GlobalExceptionHandler) uygulama, kaynak yonetimi icin try-with-resources/finally bloklari kullanma, tutarli hata yanitlari tasarlama.
*   **Uzun vadeli:** Kaos muhendisligi pratikleri uygulama, dayaniklilik (resilience) testleri yurutme, hata yonetimi standartlari ve politikalari olusturma.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```java
// Stack trace sizintisi ve kaynak sizintisi
@RestController
public class UserController {

    @GetMapping("/api/user")
    public User getUser(@RequestParam Long id) {
        Connection conn = dataSource.getConnection();
        PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
        stmt.setLong(1, id);
        ResultSet rs = stmt.executeQuery();
        // Hata durumunda connection kapatilmiyor (kaynak sizintisi)
        // Stack trace dogrudan kullaniciya donuyor
        if (rs.next()) {
            return mapUser(rs);
        }
        throw new RuntimeException("Kullanici bulunamadi: " + id);
    }
}
```

**Guvenli kod:**
```java
// Merkezi hata yonetimi ve guvenli kaynak yonetimi
@RestController
public class UserController {

    @GetMapping("/api/user")
    public ResponseEntity<Object> getUser(@RequestParam Long id) {
        // try-with-resources ile guvenli kaynak yonetimi
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?")) {

            stmt.setLong(1, id);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return ResponseEntity.ok(mapUser(rs));
                }
                return ResponseEntity.status(404).body(
                    new ErrorResponse("NOT_FOUND", "Istenen kaynak bulunamadi")
                );
            }
        } catch (SQLException e) {
            // Hata detaylarini logla ama kullaniciya gosterme
            logger.error("Veritabani hatasi - kullanici sorgusu: {}", id, e);
            return ResponseEntity.status(500).body(
                new ErrorResponse("INTERNAL_ERROR", "Islem sirasinda bir hata olustu")
            );
        }
    }
}

// Tutarli hata yanit sinifi
public class ErrorResponse {
    private String code;
    private String message;
    private String timestamp;

    public ErrorResponse(String code, String message) {
        this.code = code;
        this.message = message;
        this.timestamp = Instant.now().toString();
    }

    // getter metodlari
    public String getCode() { return code; }
    public String getMessage() { return message; }
    public String getTimestamp() { return timestamp; }
}

// Merkezi hata yonetimi — tum controller'lar icin gecerli
@ControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAllExceptions(Exception ex) {
        // Hata detaylarini logla
        logger.error("Beklenmeyen hata olustu", ex);

        // Kullaniciya genel hata mesaji dondur (stack trace yok)
        ErrorResponse error = new ErrorResponse(
            "INTERNAL_ERROR",
            "Islem sirasinda beklenmeyen bir hata olustu. Lutfen daha sonra tekrar deneyin."
        );
        return ResponseEntity.status(500).body(error);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(IllegalArgumentException ex) {
        logger.warn("Gecersiz parametre: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse("BAD_REQUEST", "Gecersiz istek parametresi");
        return ResponseEntity.status(400).body(error);
    }
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Mevcut hata yonetimi mekanizmalarini analiz et
*   [ ] Bilgi sizintisi yapan hata sayfalarini tespit et
*   [ ] Fail-open durumlarini belirle
*   [ ] Kaynak sizintisi potansiyeli olan kod bloklarini tespit et

**Duzeltme**
*   [ ] Merkezi hata yonetimi mekanizmasi (GlobalExceptionHandler) uygula
*   [ ] Uretim ortaminda stack trace gosterimini devre disi birak
*   [ ] Fail-closed prensibini tum guvenlik kontrollerine uygula
*   [ ] Kaynak yonetimi icin try-with-resources/finally bloklari kullan
*   [ ] Tutarli ve bilgi sizintisi olmayan hata yanitlari tasarla

**Dogrulama**
*   [ ] Hata yanitlarinda bilgi sizintisi olmadigini dogrula
*   [ ] Fail-closed mekanizmalarini test et
*   [ ] Kaynak sizintisi testleri gerceklestir
*   [ ] Kaos muhendisligi senaryolari uygula
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Hata loglarindaki ani artislari izle
    *   Anormal hata desenlerini tespit et (ayni endpoint'den tekrarlayan hatalar)
    *   Kaynak tukenme uyarilarini izle
    *   Regex: `(?i)(unhandled exception|stack trace|internal server error|resource leak|out of memory|connection pool exhausted)`
*   **Anomali Tespiti:**
    *   Belirli bir endpoint'den gelen hata oraninda ani artis
    *   Kaynak kullaniminda (bellek, baglanti havuzu) anormal artis
    *   Ayni kullanicidan tekrarlayan hata ureten istekler
    *   Uygulama yeniden baslama sikliginda artis

---

## Notlar
- OWASP Top 10:2025'de 10. sirada yer alan yeni bir kategoridir.
- Bu kategori, uygulamanin dayanikliligi (resilience) konusunu vurgular.
- Detayli rehber: [A10_Mishandling_of_Exceptional_Conditions.md](../OWASP_Top_10/A10_Mishandling_of_Exceptional_Conditions.md)
