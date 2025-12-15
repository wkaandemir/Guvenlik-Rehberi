### Güvenlik Açığı: Kırık Erişim Kontrolü (Broken Access Control)

### 1. Özet ve etki
*   **Yönetici Özeti:** Kırık Erişim Kontrolü, uygulamanın yetkilendirme mekanizmalarının yetersiz veya hatalı olması nedeniyle yetkisiz kullanıcıların kısıtlı kaynaklara, fonksiyonlara veya verilere erişebildiği bir güvenlik açığıdır. Bu zafiyet, saldırganların sistemde yetkilerini aşarak hassas verilere erişmesine, veri değiştirmesine veya yetkisiz işlemler yapmasına olanak tanır. OWASP Top 10:2025'de en kritik risk olarak sınıflandırılmıştır ve 2021 sürümünden bu yana birinci sıradaki yerini korumaktadır.
*   **Etkilenen bileşenler:** API endpoint'leri, web uygulaması controller'ları, veritabanı erişim katmanı, middleware bileşenleri, oturum yönetimi sistemleri, mikro servisler arası iletişim katmanları

---

### 2. Teknik detay (nasıl çalışıyor)
*   Erişim kontrolü, kimlik doğrulamayı geçen kullanıcıların hangi kaynaklara ve işlemlere erişebileceğini belirleyen yetkilendirme mekanizmalarıdır.
*   Kırık erişim kontrolü genellikle şu durumlarda ortaya çıkar:
    *   Sunucu tarafında yetkilendirme kontrollerinin eksik olması
    *   İstemci tarafında yapılan erişim kontrollerine güvenilmesi
    *   Parametre manipülasyonu ile ID'lerin değiştirilmesinin engellenmemesi
    *   Rol tabanlı erişim kontrollerinin (RBAC) düzgün uygulanmaması
    *   API'lerde uygun yetkilendirme mekanizmalarının olmaması
*   **Neden:** Kök neden olarak yetkilendirme mantığının uygulama mimarisine tam entegre edilmemesi, güvenlik denetimlerinin tutarsız uygulanması ve güvenli varsayımlarla kod geliştirilmesi sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Kullanıcı profili bilgilerini getiren `/api/users/{id}` endpoint'i
*   **2. Normal istek:** 
    ```
    GET /api/users/123 HTTP/1.1
    Authorization: Bearer <user_123_token>
    ```
*   **3. Manipüle edilmiş istek (PoC payload):**
    ```
    GET /api/users/1 HTTP/1.1
    Authorization: Bearer <user_123_token>
    ```
*   **4. Analiz:** Saldırgan, kendi kullanıcı ID'si olan 123 yerine yönetici ID'si olan 1'i kullanarak yönetici bilgilerine erişmeye çalışır. Sunucu tarafında uygun yetkilendirme kontrolü yapılmadığı için istek başarılı olur.
*   **5. Kanıt:** Yanıt olarak yöneticinin hassas bilgileri (e-posta, telefon, rol vb.) döner. Uygulama loglarında normal bir kullanıcı isteği olarak kaydedilir.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** İnternete açık, İç ağ, Yetkili kullanıcı
*   **Karmaşıklık:** Düşük

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** WAF kuralı ile şüpheli parametre manipülasyonlarını engelleme, kritik endpoint'lere ek yetkilendirme katmanları ekleme
*   **Orta vadeli:** Tüm endpoint'ler için yetkilendirme middleware'i geliştirme, parametre tabanlı erişim kontrollerini güçlendirme, API güvenlik testleri yapma
*   **Uzun vadeli:** Güvenli kod geliştirme pratikleri benimseme, yetkilendirme mimarisini yeniden tasarlayma, düzenli güvenlik kod incelemeleri yapma, geliştiricilere güvenlik eğitimi verme

---

### 6. Örnek düzeltme kodu (Node.js/Express)

**Güvensiz Kod:**
```javascript
// Kullanıcı bilgilerini getiren endpoint
app.get('/api/users/:id', authenticateToken, (req, res) => {
  const userId = req.params.id;
  // Sadece kimlik doğrulaması yapılıyor, yetkilendirme kontrolü yok!
  const user = getUserById(userId);
  res.json(user);
});
```

**Güvenli Kod:**
```javascript
// Kullanıcı bilgilerini getiren endpoint
app.get('/api/users/:id', authenticateToken, authorizeUserAccess, (req, res) => {
  const userId = req.params.id;
  const user = getUserById(userId);
  res.json(user);
});

// Yetkilendirme middleware'i
function authorizeUserAccess(req, res, next) {
  const requestedUserId = req.params.id;
  const currentUserId = req.user.id;
  const currentUserRole = req.user.role;
  
  // Kullanıcı sadece kendi bilgilerine veya admin rolündeyse başka kullanıcı bilgilerine erişebilir
  if (requestedUserId === currentUserId || currentUserRole === 'admin') {
    next();
  } else {
    res.status(403).json({ error: 'Erişim reddedildi' });
  }
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Tüm endpoint'lerin erişim kontrol gerektirip gerektirmediğini belirle
*   [ ] Mevcut yetkilendirme mekanizmalarını haritala
*   [ ] Erişim kontrolü test senaryolarını oluştur

**Düzeltme**
*   [ ] Sunucu tarafında yetkilendirme kontrollerini uygula
*   [ ] İstemci tarafı kontrollerine güvenme
*   [ ] Rol tabanlı erişim kontrolü (RBAC) mekanizması kur
*   [ ] API'ler için uygun yetkilendirme katmanları ekle

**Doğrulama**
*   [ ] Yetkisiz erişim denemelerini test et
*   [ ] Otomasyon testleri ile erişim kontrollerini doğrula
*   [ ] Penetrasyon testi yaptır

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Erişim reddedilen isteklerdeki artışları izle
    *   Aynı kullanıcı tarafından farklı kullanıcı ID'lerine yapılan erişim denemelerini tespit et
    *   Regex: `(?i)(access denied|unauthorized|forbidden).*user.*\d+`
*   **Anomali Tespiti:** 
    *   Kullanıcı rolü dışında kaynaklara erişim denemeleri
    *   Kısa sürede çok sayıda farklı kullanıcı ID'sine erişim denemeleri
    *   Normal çalışma saatleri dışında yönetici fonksiyonlarına erişim denemeleri
