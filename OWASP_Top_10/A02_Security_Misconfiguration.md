### Güvenlik Açığı: Güvenlik Yanlış Yapılandırması (Security Misconfiguration)

### 1. Özet ve etki
*   **Yönetici Özeti:** Güvenlik Yanlış Yapılandırması, uygulama, sunucu, veritabanı veya altyapı bileşenlerinin güvenlik ayarlarının düzgün yapılandırılmaması, varsayılan ayarların değiştirilmemesi veya gereksiz özelliklerin etkin bırakılması nedeniyle ortaya çıkan bir güvenlik açığıdır. Bu zafiyet, OWASP Top 10:2025'te 2. sırada yer alır ve bulut tabanlı altyapıların ve Infrastructure as Code (IaC) pratiklerinin yaygınlaşmasıyla artan bir risk haline gelmiştir.
*   **Etkilenen bileşenler:** Web sunucuları, uygulama sunucuları, veritabanları, bulut hizmetleri, container'lar, API gateway'leri, framework yapılandırma dosyaları, ortam değişkenleri

---

### 2. Teknik detay (nasıl çalışıyor)
*   Güvenlik yanlış yapılandırması, sistemlerin güvenli bir şekilde çalışması için gereken ayarların eksik, hatalı veya varsayılan durumda bırakılmasıdır.
*   Yaygın yanlış yapılandırma örnekleri:
    *   Varsayılan parolaların değiştirilmemesi
    *   Gereksiz servislerin çalıştırılması
    *   Hata ayıklama modunun üretim ortamında aktif bırakılması
    *   Güvenlik başlıklarının (security headers) eksik olması
    *   Bulut depolama kaynaklarının herkese açık olması
    *   Veritabanı bağlantı bilgilerinin düz metin olarak saklanması
    *   SSL/TLS yapılandırmasının zayıf veya eksik olması
*   **Neden:** Kök neden olarak güvenlik yapılandırmalarının otomatikleştirilmemesi, yapılandırma yönetimi süreçlerinin eksikliği, geliştirme ve üretim ortamları arasındaki farkların yeterince test edilmemesi ve güvenlik odaklı yapılandırma şablonlarının kullanılmaması sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Hata ayıklama modunda çalışan bir web uygulaması
*   **2. Normal istek:** 
    ```
    GET /api/users/123 HTTP/1.1
    Host: example.com
    ```
*   **3. Manipüle edilmiş istek (PoC payload):**
    ```
    GET /api/users/123?debug=true HTTP/1.1
    Host: example.com
    ```
*   **4. Analiz:** Saldırgan, uygulamanın hata ayıklama modunda çalıştığını fark eder ve debug parametresini ekler. Uygulama, hata ayıklama modunda olduğu için detaylı hata mesajları, sistem bilgileri ve potansiyel olarak hassas yapılandırma bilgilerini içeren bir yanıt döner.
*   **5. Kanıt:** Yanıt olarak uygulamanın iç yapısı, veritabanı şeması, sunucu yolu ve diğer hassas bilgiler görüntülenir.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** İnternete açık, İç ağ, Bulut altyapısı
*   **Karmaşıklık:** Düşük

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Varsayılan parolaları değiştirme, gereksiz servisleri devre dışı bırakma, hata ayıklama modlarını kapatma
*   **Orta vadeli:** Güvenlik yapılandırma şablonları oluşturma, otomatik yapılandırma denetimleri uygulama, güvenlik başlıklarını ekleme
*   **Uzun vadeli:** Infrastructure as Code (IaC) pratikleri benimseme, yapılandırma yönetimi süreçleri geliştirme, düzenli güvenlik yapılandırma denetimleri yapma, güvenlik odaklı CI/CD pipeline'ı oluşturma

---

### 6. Örnek düzeltme kodu (Nginx Yapılandırması)

**Güvensiz Yapılandırma:**
```nginx
server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        # Güvenlik başlıkları eksik
        # Hata ayıklama bilgileri açık
    }
}
```

**Güvenli Yapılandırma:**
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # SSL yapılandırması
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    
    # Güvenlik başlıkları
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'";
    
    location / {
        proxy_pass http://localhost:3000;
        
        # Hata ayıklama bilgilerini gizle
        proxy_hide_header X-Powered-By;
        proxy_hide_header Server;
        
        # Hata sayfalarını özelleştir
        error_page 404 /custom_404.html;
        error_page 500 502 503 504 /custom_50x.html;
    }
    
    # HTTP'den HTTPS'e yönlendirme
    server {
        listen 80;
        server_name example.com;
        return 301 https://$server_name$request_uri;
    }
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Tüm sistem ve uygulama yapılandırmalarını envanterle
*   [ ] Varsayılan parolaları ve ayarları belirle
*   [ ] Güvenlik yapılandırma standartları oluştur

**Düzeltme**
*   [ ] Varsayılan parolaları değiştir
*   [ ] Gereksiz servisleri devre dışı bırak
*   [ ] Hata ayıklama modlarını kapat
*   [ ] Güvenlik başlıklarını ekle
*   [ ] SSL/TLS yapılandırmasını güçlendir

**Doğrulama**
*   [ ] Yapılandırma denetim araçları ile tarama yap
*   [ ] Güvenlik başlıklarını test et
*   [ ] SSL/TLS yapılandırmasını doğrula
*   [ ] Penetrasyon testi yaptır

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Varsayılan parola kullanım denemelerini izle
    *   Hata ayıklama modu erişimlerini tespit et
    *   Regex: `(?i)(debug|test|admin|default).*password|password.*(admin|default|test)`
*   **Anomali Tespiti:** 
    *   Üretim ortamında hata ayıklama modu aktiviteleri
    *   Normal dışı port erişimleri
    *   Yapılandırma dosyalarındaki yetkisiz değişiklikler
    *   SSL/TLS sertifikalarındaki anormallikler
