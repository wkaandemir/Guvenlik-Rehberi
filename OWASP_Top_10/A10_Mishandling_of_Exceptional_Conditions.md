### Güvenlik Açığı: İstisnai Durumların Yanlış Ele Alınması (Mishandling of Exceptional Conditions)

### 1. Özet ve etki
*   **Yönetici Özeti:** İstisnai Durumların Yanlış Ele Alınması, uygulamanın hata durumlarını güvenli bir şekilde yönetememesi nedeniyle ortaya çıkan bir güvenlik açığıdır. Bu zafiyet, hata mesajlarındaki bilgi sızıntıları, "fail-open" durumları veya hata durumlarındaki tutarsız davranışlar gibi çeşitli şekillerde ortaya çıkabilir. OWASP Top 10:2025'te 10. sırada yer alan yeni bir kategoridir ve modern uygulamaların dayanıklılığını (resilience) vurgulamaktadır. Bu kategori, uygulamanın sadece "mutlu yol" (happy path) değil, aynı zamanda hata durumlarında da güvenli kalmasını gerektirir.
*   **Etkilenen bileşenler:** Hata yönetimi katmanı, exception handler'lar, veritabanı bağlantıları, API error handling, sistem kaynakları

---

### 2. Teknik detay (nasıl çalışıyor)
*   İstisnai durumların yanlış ele alınması, uygulamanın hata durumlarını güvenli bir şekilde yönetememesidir.
*   Yaygın istisnai durum hataları:
    *   Hata mesajlarındaki bilgi sızıntıları (stack trace, sistem bilgileri)
    *   "Fail-open" durumları (hata durumunda güvenlik kontrollerinin devre dışı kalması)
    *   Kaynakların düzgün serbest bırakılmaması
    *   Hata durumlarında tutarsız davranışlar
    *   Hata loglarının yetersizliği
    *   Hata durumlarında sistem durumunun bozulması
*   **Neden:** Kök neden olarak hata yönetimi stratejisinin eksikliği, güvenlik odaklı hata yönetimi prensiplerinin ihmal edilmesi, hata durumlarının yeterince test edilmemesi ve "fail-closed" prensibinin uygulanmaması sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Hata durumunda stack trace gösteren bir web uygulaması
*   **2. Normal durum:** 
    ```
    GET /api/users/123 HTTP/1.1
    Host: example.com
    
    // Yanıt: 200 OK ve kullanıcı bilgileri
    ```
*   **3. Manipüle edilmiş istek (PoC payload):**
    ```
    GET /api/users/' HTTP/1.1
    Host: example.com
    
    // Yanıt: 500 Internal Server Error ve stack trace
    ```
*   **4. Analiz:** Saldırgan, uygulamanın hata durumunda stack trace gösterdiğini fark eder. SQL enjeksiyonu payload'u göndererek veritabanı hatası tetikler ve uygulama stack trace'i yanıtta döner. Stack trace, uygulamanın iç yapısı, veritabanı şeması ve diğer hassas bilgileri içerir.
*   **5. Kanıt:** Yanıt olarak stack trace ve sistem bilgileri görüntülenir. Bu bilgiler, saldırganın daha hedefli saldırılar yapmasını sağlar.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Orta
*   **Saldırı Yüzeyi:** İnternete açık, İç network
*   **Karmaşıklık:** Düşük

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Hata mesajlarındaki bilgi sızıntılarını engelleme, "fail-open" durumlarını düzeltme, genel hata sayfaları oluşturma
*   **Orta vadeli:** Merkezi hata yönetimi mekanizması kurma, hata loglarını güvenli hale getirme, "fail-closed" prensibini uygulama
*   **Uzun vadeli:** Kaos mühendisliği teknikleri uygulama, hata durumlarını test etme, dayanıklı sistemler tasarımı, geliştiricilere güvenli hata yönetimi eğitimi verme

---

### 6. Örnek düzeltme kodu (Java/Spring)

**Güvensiz Kod:**
```java
@RestController
public class UserController {
    
    @GetMapping("/api/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable String id) {
        try {
            // Veritabanından kullanıcıyı getir
            User user = userService.getUserById(id);
            return ResponseEntity.ok(user);
        } catch (Exception e) {
            // Hata durumunda stack trace gösterme - bilgi sızıntısı
            e.printStackTrace();
            return ResponseEntity.status(500).body(null);
        }
    }
    
    @PostMapping("/api/users")
    public ResponseEntity<User> createUser(@RequestBody User user) {
        try {
            // Kullanıcı oluştur
            User createdUser = userService.createUser(user);
            return ResponseEntity.ok(createdUser);
        } catch (Exception e) {
            // Hata durumunda detaylı hata mesajı döndürme - bilgi sızıntısı
            return ResponseEntity.status(500).body(null);
        }
    }
}

// Veritabanı bağlantısı - kaynak sızıntısı
public class DatabaseService {
    public Connection getConnection() throws SQLException {
        Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
        // Bağlantı kapatma yok - kaynak sızıntısı
        return conn;
    }
}
```

**Güvenli Kod:**
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestController
public class UserController {
    
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);
    
    @GetMapping("/api/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable String id) {
        try {
            // Girdi doğrulama
            if (!isValidId(id)) {
                logger.warn("Invalid user ID format: {}", id);
                return ResponseEntity.badRequest().build();
            }
            
            // Veritabanından kullanıcıyı getir
            User user = userService.getUserById(id);
            if (user == null) {
                logger.info("User not found with ID: {}", id);
                return ResponseEntity.notFound().build();
            }
            
            return ResponseEntity.ok(user);
        } catch (DataAccessException e) {
            // Veritabanı hatası - detayları logla, genel hata döndür
            logger.error("Database error while fetching user with ID: {}", id, e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        } catch (Exception e) {
            // Genel hata - detayları logla, genel hata döndür
            logger.error("Unexpected error while fetching user with ID: {}", id, e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    @PostMapping("/api/users")
    public ResponseEntity<User> createUser(@RequestBody User user) {
        try {
            // Girdi doğrulama
            if (!isValidUser(user)) {
                logger.warn("Invalid user data: {}", user);
                return ResponseEntity.badRequest().build();
            }
            
            // Kullanıcı oluştur
            User createdUser = userService.createUser(user);
            return ResponseEntity.ok(createdUser);
        } catch (DataAccessException e) {
            // Veritabanı hatası - detayları logla, genel hata döndür
            logger.error("Database error while creating user: {}", user, e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        } catch (Exception e) {
            // Genel hata - detayları logla, genel hata döndür
            logger.error("Unexpected error while creating user: {}", user, e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    private boolean isValidId(String id) {
        // ID formatını doğrula
        return id != null && id.matches("\\d+");
    }
    
    private boolean isValidUser(User user) {
        // Kullanıcı verisini doğrula
        return user != null && 
               user.getUsername() != null && 
               user.getEmail() != null && 
               user.getEmail().matches("[^@]+@[^@]+\\.[^@]+");
    }
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception e) {
        // Tüm istisnaları yakala, detayları logla, genel hata döndür
        logger.error("Unhandled exception: ", e);
        
        ErrorResponse errorResponse = new ErrorResponse(
            "Internal server error",
            "An unexpected error occurred. Please try again later."
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                          .body(errorResponse);
    }
    
    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<ErrorResponse> handleDataAccessException(DataAccessException e) {
        // Veritabanı istisnalarını yakala, detayları logla, genel hata döndür
        logger.error("Database access exception: ", e);
        
        ErrorResponse errorResponse = new ErrorResponse(
            "Database error",
            "A database error occurred. Please try again later."
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                          .body(errorResponse);
    }
}

// Veritabanı bağlantısı - kaynak yönetimi
public class DatabaseService {
    
    public Connection getConnection() throws SQLException {
        try {
            Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
            return conn;
        } catch (SQLException e) {
            logger.error("Failed to get database connection", e);
            throw e;
        }
    }
    
    public void executeQuery(String query) {
        Connection conn = null;
        PreparedStatement stmt = null;
        ResultSet rs = null;
        
        try {
            conn = getConnection();
            stmt = conn.prepareStatement(query);
            rs = stmt.executeQuery();
            
            // Sonuçları işle
            processResults(rs);
        } catch (SQLException e) {
            logger.error("Error executing query: {}", query, e);
            throw new RuntimeException("Database error", e);
        } finally {
            // Kaynakları düzgün bir şekilde serbest bırak
            if (rs != null) {
                try { rs.close(); } catch (SQLException e) { logger.error("Error closing result set", e); }
            }
            if (stmt != null) {
                try { stmt.close(); } catch (SQLException e) { logger.error("Error closing statement", e); }
            }
            if (conn != null) {
                try { conn.close(); } catch (SQLException e) { logger.error("Error closing connection", e); }
            }
        }
    }
}

// Hata yanıtı sınıfı
public class ErrorResponse {
    private String error;
    private String message;
    
    public ErrorResponse(String error, String message) {
        this.error = error;
        this.message = message;
    }
    
    // Getters and setters
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Mevcut hata yönetimi mekanizmalarını analiz et
*   [ ] Hata mesajlarındaki bilgi sızıntılarını belirle
*   [ ] "Fail-open" durumlarını tespit et

**Düzeltme**
*   [ ] Merkezi hata yönetimi mekanizması kur
*   [ ] Hata mesajlarındaki bilgi sızıntılarını engelle
*   [ ] "Fail-closed" prensibini uygula
*   [ ] Kaynak yönetimini düzelt

**Doğrulama**
*   [ ] Hata yönetimi mekanizmalarını test et
*   [ ] Kaos mühendisliği teknikleri uygula
*   [ ] Penetrasyon testi yaptır
*   [ ] Düzenli güvenlik denetimleri yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Hata loglarındaki artışları izle
    *   Anormal hata desenlerini tespit et
    *   Regex: `(?i)(exception|error|stack trace).*leak|leak.*(exception|error|stack trace)`
*   **Anomali Tespiti:** 
    *   Hata oranındaki anormal artışlar
    *   Normal dışı hata türleri
    *   Kaynak sızıntıları
    *   "Fail-open" durumları
