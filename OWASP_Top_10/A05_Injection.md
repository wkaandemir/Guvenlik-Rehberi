### Güvenlik Açığı: Enjeksiyon (Injection)

### 1. Özet ve etki
*   **Yönetici Özeti:** Enjeksiyon, uygulamanın kullanıcı girdilerini doğrulamadan veya filtrelemeden doğrudan bir yorumlayıcıya (veritabanı, shell, LDAP vb.) göndermesi nedeniyle ortaya çıkan bir güvenlik açığıdır. Saldırganlar bu zafiyeti kullanarak kötü niyetli komutlar çalıştırabilir, veri çalabilir veya sistem kontrolü ele geçirebilir. OWASP Top 10:2025'te 5. sırada yer alır ve 2021'e göre sıralaması düşmüş olsa da hala yaygın ve tehlikelidir. Framework'lerin bu tür saldırılara karşı korumaları artsa da legacy kodlar ve özel implementasyonlar hala risk altındadır.
*   **Etkilenen bileşenler:** Veritabanı sorguları, OS komutları, LDAP sorguları, NoSQL veritabanları, ORM katmanları, API endpoint'leri

---

### 2. Teknik detay (nasıl çalışıyor)
*   Enjeksiyon saldırıları, kullanıcı girdilerinin uygulama tarafından doğrudan bir yorumlayıcıya gönderilmesiyle gerçekleşir.
*   Yaygın enjeksiyon türleri:
    *   SQL Enjeksiyonu: Veritabanı sorgularına kötü niyetli SQL kodu enjekte etmek
    *   OS Komut Enjeksiyonu: İşletim sistemi komutlarına kötü niyetli kod enjekte etmek
    *   NoSQL Enjeksiyonu: NoSQL veritabanı sorgularına kötü niyetli kod enjekte etmek
    *   LDAP Enjeksiyonu: LDAP sorgularına kötü niyetli kod enjekte etmek
    *   XPath Enjeksiyonu: XPath sorgularına kötü niyetli kod enjekte etmek
*   **Neden:** Kök neden olarak input validation eksikliği, parametreli sorguların kullanılmaması, kullanıcı girdilerinin güvenli bir şekilde işlenmemesi ve güvenli kod geliştirme pratiklerinin ihmal edilmesi sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Kullanıcı girişi ile veritabanı sorgusu yapan login formu
*   **2. Normal istek:** 
    ```
    POST /login HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    
    username=admin&password=secret123
    ```
*   **3. Manipüle edilmiş istek (PoC payload):**
    ```
    POST /login HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    
    username=admin'--&password=anything
    ```
*   **4. Analiz:** Saldırgan, username alanına SQL enjeksiyonu payload'u gönderir. Uygulama bu girdiyi doğrudan SQL sorgusuna ekler: `SELECT * FROM users WHERE username='admin'--' AND password='...'`. `--` karakterleri SQL'de yorum satırı başlangıcıdır, bu nedenle password kontrolü atlanır ve saldırgan admin olarak sisteme giriş yapar.
*   **5. Kanıt:** Yanıt olarak başarılı oturum açma ve admin yetkileriyle sistem erişimi sağlanır. Loglarda başarılı admin girişi olarak kaydedilir.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** İnternete açık, İç ağ
*   **Karmaşıklık:** Düşük

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Input validation ekleme, WAF kuralları uygulama, özel karakterleri filtreleme
*   **Orta vadeli:** Parametreli sorgular kullanma, ORM framework'leri benimseme, input sanitization mekanizmaları kurma
*   **Uzun vadeli:** Güvenli kod geliştirme pratikleri benimseme, düzenli güvenlik kod incelemeleri yapma, geliştiricilere güvenlik eğitimi verme, otomatik güvenlik testleri entegrasyonu

---

### 6. Örnek düzeltme kodu (PHP)

**Güvensiz Kod:**
```php
<?php
// SQL Enjeksiyonuna açık kod
$userId = $_GET['id'];
$query = "SELECT * FROM users WHERE id = " . $userId;  // Doğrudan kullanıcı girdini sorguya ekleme
$result = mysqli_query($conn, $query);

// OS Komut Enjeksiyonuna açık kod
$file = $_GET['filename'];
$output = shell_exec("cat " . $file);  // Doğrudan kullanıcı girdini komuta ekleme
echo $output;
?>
```

**Güvenli Kod:**
```php
<?php
// Parametreli sorgu ile SQL Enjeksiyonu önleme
$userId = $_GET['id'];
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $userId);  // Parametreyi güvenli bir şekilde bağla
$stmt->execute();
$result = $stmt->get_result();

// Input validation ile OS Komut Enjeksiyonu önleme
$file = $_GET['filename'];
// İzin verilen karakterleri kontrol et
if (preg_match('/^[a-zA-Z0-9._-]+$/', $file)) {
    // Beyaz liste ile dosya adını doğrula
    $allowedFiles = ['file1.txt', 'file2.txt', 'config.json'];
    if (in_array($file, $allowedFiles)) {
        $output = shell_exec("cat " . escapeshellarg($file));  // Shell argümanını güvenli hale getir
        echo $output;
    } else {
        echo "İzin verilmeyen dosya";
    }
} else {
    echo "Geçersiz dosya adı";
}
?>
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Tüm kullanıcı girdi alanlarını belirle
*   [ ] Veritabanı sorgularını ve komut çağrılarını envanterle
*   [ ] Mevcut input validation mekanizmalarını değerlendir

**Düzeltme**
*   [ ] Parametreli sorgular kullan
*   [ ] Input validation ve sanitization uygula
*   [ ] ORM framework'leri benimse
*   [ ] Minimum yetki ilkesini uygula

**Doğrulama**
*   [ ] Otomatik enjeksiyon tarama araçları kullan
*   [ ] Manuel güvenlik testleri yap
*   [ ] Penetrasyon testi yaptır
*   [ ] Kod incelemeleri ile enjeksiyon risklerini kontrol et

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Şüpheli SQL sorgu desenlerini izle
    *   Anormal veritabanı erişimlerini tespit et
    *   Regex: `(?i)(union|select|insert|update|delete|drop|exec|script).*--|'.*(union|select|insert|update|delete|drop|exec|script)`
*   **Anomali Tespiti:** 
    *   Normal dışı veritabanı sorgu desenleri
    *   Başarısız SQL sorgularındaki artış
    *   Uygulama normal davranışının dışında veritabanı erişimleri
    *   Şüpheli OS komut çalıştırma denemeleri

