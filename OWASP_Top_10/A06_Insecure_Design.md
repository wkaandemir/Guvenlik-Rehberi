### Güvenlik Açığı: Güvensiz Tasarım (Insecure Design)

### 1. Özet ve etki
*   **Yönetici Özeti:** Güvensiz Tasarım, uygulamanın temel mimarisinde veya tasarımında güvenlik kontrollerinin eksik olması veya yanlış uygulanması nedeniyle ortaya çıkan bir güvenlik zafiyetidir. Bu kategori, kod implementasyonundaki hatalardan ziyade tasarım seviyesindeki eksiklikleri ifade eder. OWASP Top 10:2025'te 6. sırada yer alır ve 2021'e göre sıralaması düşmüş olsa da hala önemlidir. Güvensiz tasarım, uygulamanın temelinde yer aldığı için düzeltilmesi en zor ve en maliyetli güvenlik sorunlarından biridir.
*   **Etkilenen bileşenler:** Uygulama mimarisi, iş akışları, veri modelleri, API tasarımı, oturum yönetimi, hata yönetimi, yetkilendirme mekanizmaları

---

### 2. Teknik detay (nasıl çalışıyor)
*   Güvensiz tasarım, uygulamanın temel mimarisinde güvenlik kontrollerinin eksik olması veya yanlış uygulanmasıdır.
*   Yaygın güvensiz tasarım örnekleri:
    *   Threat modeling (tehdit modelleme) yapılmaması
    *   Güvenli varsayımlarla tasarım yapılması
    *   Hata durumlarının güvenli bir şekilde ele alınmaması
    *   İş akışlarında güvenlik kontrollerinin eksik olması
    *   Minimum yetki ilkesinin uygulanmaması
    *   Güvenlik denetimlerinin tutarsız olması
    *   Veri bütünlüğü kontrollerinin eksikliği
*   **Neden:** Kök neden olarak güvenlik odaklı tasarım prensiplerinin ihmal edilmesi, threat modeling süreçlerinin eksikliği, güvenlik uzmanlarının tasarım aşamasına dahil edilmemesi ve güvenlik gereksinimlerinin yeterince dikkate alınmaması sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Parola sıfırlama iş akışı
*   **2. Normal durum:** 
    ```
    1. Kullanıcı parola sıfırlama talebinde bulunur
    2. Sistem kullanıcıya e-posta ile geçici bir link gönderir
    3. Kullanıcı link üzerinden yeni parola belirler
    ```
*   **3. Manipüle edilmiş durum (PoC payload):**
    ```
    1. Saldırgan, parola sıfırlama linkini tahmin edebilir bir yapıda olduğunu fark eder
    2. Link formatı: https://example.com/reset?token=USERID_TIMESTAMP_HASH
    3. Saldırgan, USERID ve TIMESTAMP değerlerini değiştirerek farklı kullanıcıların parolasını sıfırlamayı dener
    ```
*   **4. Analiz:** Parola sıfırlama mekanizması, token'ları tahmin edilebilir bir yapıda oluşturur ve yeterli güvenlik kontrolleri içermez. Saldırgan, bu zafiyeti kullanarak başka kullanıcıların parolalarını sıfırlayabilir.
*   **5. Kanıt:** Saldırgan, başka bir kullanıcının parolasını başarıyla sıfırlar ve hesabına erişim sağlar. Loglarda parola sıfırlama işlemi olarak kaydedilir.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** İnternete açık, İç network, Uygulama mimarisi
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Mevcut tasarım zafiyetlerini belirleme ve geçici çözümler uygulama, kritik iş akışlarına ek güvenlik kontrolleri ekleme
*   **Orta vadeli:** Threat modeling süreçleri uygulama, güvenli tasarım prensiplerini benimseme, güvenlik denetimlerini güçlendirme
*   **Uzun vadeli:** Güvenlik odaklı tasarım kültürü oluşturma, düzenli güvenlik tasarım incelemeleri yapma, geliştiricilere güvenli tasarım eğitimi verme, güvenlik gereksinimlerini geliştirme sürecine entegre etme

---

### 6. Örnek düzeltme kodu (Java/Spring)

**Güvensiz Tasarım:**
```java
// Parola sıfırlama servisi - güvensiz tasarım
@Service
public class PasswordResetService {
    
    public void resetPassword(String email) {
        User user = userRepository.findByEmail(email);
        if (user != null) {
            // Tahmin edilebilir token oluşturma
            String token = user.getId() + "_" + System.currentTimeMillis();
            
            // Token'ı veritabanına kaydetme
            user.setResetToken(token);
            userRepository.save(user);
            
            // E-posta gönderme
            emailService.sendResetEmail(email, token);
        }
    }
    
    public boolean validateToken(String token) {
        // Token doğrulaması yapmama - güvenlik açığı
        return token != null && !token.isEmpty();
    }
}
```

**Güvenli Tasarım:**
```java
// Parola sıfırlama servisi - güvenli tasarım
@Service
public class PasswordResetService {
    
    private static final int TOKEN_EXPIRATION_MINUTES = 30;
    private static final int TOKEN_LENGTH = 64;
    
    public void resetPassword(String email) {
        User user = userRepository.findByEmail(email);
        if (user != null) {
            // Güçlü ve rastgele token oluşturma
            String token = generateSecureToken();
            
            // Token'ı veritabanına kaydetme ve son kullanma tarihi belirleme
            user.setResetToken(token);
            user.setTokenExpiration(LocalDateTime.now().plusMinutes(TOKEN_EXPIRATION_MINUTES));
            userRepository.save(user);
            
            // E-posta gönderme
            emailService.sendResetEmail(email, token);
        }
        
        // Kullanıcı bulunmasa bile aynı mesajı döndürerek枚举 saldırılarını önle
    }
    
    public boolean validateToken(String token) {
        if (token == null || token.isEmpty()) {
            return false;
        }
        
        User user = userRepository.findByResetToken(token);
        if (user == null) {
            return false;
        }
        
        // Token'ın süresini kontrol etme
        return user.getTokenExpiration().isAfter(LocalDateTime.now());
    }
    
    private String generateSecureToken() {
        // Güvenli rastgele token oluşturma
        SecureRandom random = new SecureRandom();
        byte[] bytes = new byte[TOKEN_LENGTH];
        random.nextBytes(bytes);
        return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
    }
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Mevcut uygulama mimarisini ve iş akışlarını analiz et
*   [ ] Threat modeling süreçleri uygula
*   [ ] Güvenlik gereksinimlerini belirle

**Düzeltme**
*   [ ] Güvensiz tasarım desenlerini belirle ve düzelt
*   [ ] Minimum yetki ilkesini uygula
*   [ ] Güvenli hata yönetimi mekanizmaları kur
*   [ ] İş akışlarına güvenlik kontrolleri ekle

**Doğrulama**
*   [ ] Tasarım incelemeleri yap
*   [ ] Güvenlik testleri gerçekleştir
*   [ ] Penetrasyon testi yaptır
*   [ ] Düzenli güvenlik denetimleri yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   İş akışlarındaki anormal davranışları izle
    *   Tasarım seviyesindeki güvenlik ihlallerini tespit et
    *   Regex: `(?i)(design flaw|insecure workflow|business logic flaw)`
*   **Anomali Tespiti:** 
    *   Normal dışı iş akışı kullanımları
    *   İş kurallarının aşılma denemeleri
    *   Tasarım seviyesindeki güvenlik kontrollerinin atlatılması
    *   İş mantığı zafiyetlerini istismar etme denemeleri

