### Güvenlik Açığı: Kimlik Doğrulama Hataları (Authentication Failures)

### 1. Özet ve etki
*   **Yönetici Özeti:** Kimlik Doğrulama Hataları, kullanıcı kimliğinin doğrulanması sürecindeki zayıflıklar nedeniyle yetkisiz kullanıcıların sisteme erişim sağlayabildiği bir güvenlik açığıdır. Bu zafiyet, parola politikalarının zayıf olması, oturum yönetimindeki hatalar veya çok faktörlü kimlik doğrulamanın eksikliği gibi çeşitli şekillerde ortaya çıkabilir. OWASP Top 10:2025'te 7. sırada yer alır ve 2021'deki "Identification and Authentication Failures" kategorisinin yerini almıştır. Kimlik, modern uygulamaların güvenlik sınırı olarak kabul edildiği için bu kategori kritik önem taşır.
*   **Etkilenen bileşenler:** Login formları, parola yönetimi sistemleri, oturum yönetimi, çok faktörlü kimlik doğrulama (MFA), API kimlik doğrulama mekanizmaları, parola sıfırlama iş akışları

---

### 2. Teknik detay (nasıl çalışıyor)
*   Kimlik doğrulama, kullanıcının kim olduğunu doğrulama sürecidir. Bu süreçteki zayıflıklar, yetkisiz erişime olanak tanır.
*   Yaygın kimlik doğrulama hataları:
    *   Zayıf parola politikaları
    *   Parolaların düz metin olarak saklanması
    *   Oturum yönetimindeki hatalar
    *   Çok faktörlü kimlik doğrulamanın eksikliği
    *   Parola sıfırlama mekanizmalarındaki zayıflıklar
    *   Brute force saldırılarına karşı korumasızlık
    *   Kullanıcı枚举 saldırılarına açık olma
*   **Neden:** Kök neden olarak güvenli kimlik doğrulama standartlarının takip edilmemesi, parola politikalarının zayıf olması, oturum yönetiminin düzgün yapılandırılmaması ve çok faktörlü kimlik doğrulamanın ihmal edilmesi sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Brute force koruması olmayan login formu
*   **2. Normal istek:** 
    ```
    POST /login HTTP/1.1
    Content-Type: application/json
    
    {
      "username": "admin",
      "password": "correct_password"
    }
    ```
*   **3. Manipüle edilmiş istek (PoC payload):**
    ```
    POST /login HTTP/1.1
    Content-Type: application/json
    
    {
      "username": "admin",
      "password": "password1"
    }
    
    POST /login HTTP/1.1
    Content-Type: application/json
    
    {
      "username": "admin",
      "password": "password2"
    }
    
    // ... binlerce deneme ...
    ```
*   **4. Analiz:** Saldırgan, uygulamanın brute force koruması olmadığını fark eder ve otomatik araçlarla binlerce parola denemesi yapar. Başarılı bir denemede admin kullanıcısının parolasını bulur ve sisteme erişim sağlar.
*   **5. Kanıt:** Yanıt olarak başarılı oturum açma ve admin yetkileriyle sistem erişimi sağlanır. Loglarda çok sayıda başarısız deneme sonrası başarılı bir giriş görülür.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** İnternete açık, İç network
*   **Karmaşıklık:** Düşük

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Brute force koruması ekleme, zayıf parola politikalarını güçlendirme, parola sıfırlama mekanizmalarını düzeltme
*   **Orta vadeli:** Güçlü parola hashing algoritmaları kullanma, oturum yönetimini güvenli hale getirme, çok faktörlü kimlik doğrulama (MFA) uygulama
*   **Uzun vadeli:** Kimlik ve erişim yönetimi (IAM) stratejisi geliştirme, parolasız kimlik doğrulama yöntemlerini benimseme, düzenli kimlik doğrulama güvenlik denetimleri yapma

---

### 6. Örnek düzeltme kodu (Node.js/Express)

**Güvensiz Kod:**
```javascript
// Login endpoint - güvensiz kimlik doğrulama
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  
  // Veritabanından kullanıcıyı getir
  const user = getUserByUsername(username);
  
  // Parolayı düz metin olarak karşılaştırma
  if (user && user.password === password) {
    // Güvensiz oturum oluşturma
    req.session.userId = user.id;
    res.json({ success: true });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// Parola sıfırlama - güvensiz
app.post('/reset-password', (req, res) => {
  const { email } = req.body;
  
  // Kullanıcı varlığını kontrol etme -枚举 saldırısına açık
  const user = getUserByEmail(email);
  if (user) {
    // Tahmin edilebilir token oluşturma
    const resetToken = user.id + '_' + Date.now();
    sendResetEmail(email, resetToken);
  }
  
  res.json({ message: 'If email exists, reset link sent' });
});
```

**Güvenli Kod:**
```javascript
const bcrypt = require('bcrypt');
const rateLimit = require('express-rate-limit');
const crypto = require('crypto');

// Brute force koruması
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 dakika
  max: 5, // 15 dakikada en fazla 5 deneme
  message: 'Too many login attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

// Login endpoint - güvenli kimlik doğrulama
app.post('/login', loginLimiter, async (req, res) => {
  const { username, password } = req.body;
  
  try {
    // Veritabanından kullanıcıyı getir
    const user = await getUserByUsername(username);
    
    // Kullanıcı varlığını ve parolayı güvenli bir şekilde kontrol et
    if (user && await bcrypt.compare(password, user.passwordHash)) {
      // Güvenli oturum oluşturma
      req.session.userId = user.id;
      req.session.role = user.role;
      
      // Oturum güvenliği ayarları
      req.session.cookie.secure = true; // HTTPS üzerinden gönder
      req.session.cookie.httpOnly = true; // JavaScript erişimini engelle
      req.session.cookie.maxAge = 30 * 60 * 1000; // 30 dakika
      
      res.json({ success: true });
    } else {
      // Hata mesajını genel tutarak枚举 saldırılarını önle
      res.status(401).json({ error: 'Invalid username or password' });
    }
  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Parola sıfırlama - güvenli
app.post('/reset-password', async (req, res) => {
  const { email } = req.body;
  
  try {
    // Her zaman aynı mesajı döndürerek枚举 saldırılarını önle
    const message = 'If email exists, reset link sent';
    
    const user = await getUserByEmail(email);
    if (user) {
      // Güvenli ve rastgele token oluşturma
      const resetToken = crypto.randomBytes(32).toString('hex');
      const expiration = new Date(Date.now() + 60 * 60 * 1000); // 1 saat geçerli
      
      // Token'ı veritabanına kaydetme
      await saveResetToken(user.id, resetToken, expiration);
      
      // E-posta gönderme
      await sendResetEmail(email, resetToken);
    }
    
    res.json({ message });
  } catch (error) {
    console.error('Reset password error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Mevcut kimlik doğrulama mekanizmalarını analiz et
*   [ ] Parola politikalarını değerlendir
*   [ ] Oturum yönetimini incele

**Düzeltme**
*   [ ] Güçlü parola politikaları uygula
*   [ ] Parolaları güvenli bir şekilde hash'le
*   [ ] Brute force koruması ekle
*   [ ] Çok faktörlü kimlik doğrulama (MFA) uygula

**Doğrulama**
*   [ ] Kimlik doğrulama mekanizmalarını test et
*   [ ] Oturum yönetimini doğrula
*   [ ] Penetrasyon testi yaptır
*   [ ] Düzenli güvenlik denetimleri yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Başarısız giriş denemelerindeki artışları izle
    *   Anormal konumlardan yapılan giriş denemelerini tespit et
    *   Regex: `(?i)(login|authentication|signin).*fail|fail.*(login|authentication|signin)`
*   **Anomali Tespiti:** 
    *   Kısa sürede çok sayıda başarısız giriş denemesi
    *   Normal dışı konumlardan veya zamanlarda yapılan giriş denemeleri
    *   Aynı IP adresinden çok sayıda farklı kullanıcı hesabına giriş denemeleri
    *   Parola sıfırlama taleplerindeki anormal artışlar

