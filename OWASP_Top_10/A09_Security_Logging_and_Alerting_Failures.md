### Güvenlik Açığı: Güvenlik Kayıt Tutma ve Uyarı Hataları (Security Logging and Alerting Failures)

### 1. Özet ve etki
*   **Yönetici Özeti:** Güvenlik Kayıt Tutma ve Uyarı Hataları, güvenlik olaylarının yetersiz kaydedilmesi, kritik olayların tespit edilememesi veya uyarı mekanizmalarının eksikliği nedeniyle saldırıların fark edilemediği veya geç tespit edildiği bir güvenlik açığıdır. Bu zafiyet, saldırganların sistemde uzun süre fark edilmeden kalmasına olanak tanır. OWASP Top 10:2025'te 9. sırada yer alır ve 2021'deki "Logging and Monitoring Failures" kategorisinin yerini almıştır. "Alerting" kelimesinin eklenmesi, sadece kayıt tutmanın değil, aynı zamanda uyarı mekanizmalarının da önemini vurgulamaktadır.
*   **Etkilenen bileşenler:** Loglama sistemleri, SIEM (Security Information and Event Management), izleme mekanizmaları, uyarı sistemleri, olay yönetimi

---

### 2. Teknik detay (nasıl çalışıyor)
*   Güvenlik kayıt tutma ve uyarı hataları, güvenlik olaylarının yetersiz kaydedilmesi veya tespit edilememesi durumudur.
*   Yaygın kayıt tutma ve uyarı hataları:
    *   Kritik güvenlik olaylarının kaydedilmemesi
    *   Yetersiz log detayı veya bağlamı
    *   Logların güvenli bir şekilde saklanmaması
    *   Uyarı mekanizmalarının eksikliği
    *   Logların düzenli olarak incelenmemesi
    *   Zaman damgalarının eksik veya tutarsız olması
    *   Logların manipülasyona açık olması
*   **Neden:** Kök neden olarak güvenlik olaylarının yeterince önemsenmemesi, loglama stratejisinin eksikliği, uyarı mekanizmalarının kurulmaması ve logların düzenli olarak incelenmemesi sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Yetersiz loglama ve uyarı mekanizmasına sahip bir uygulama
*   **2. Normal durum:** 
    ```
    // Başarısız giriş denemesi
    POST /login HTTP/1.1
    Content-Type: application/json
    
    {
      "username": "admin",
      "password": "wrong_password"
    }
    
    // Log: "Login failed"
    ```
*   **3. Manipüle edilmiş durum (PoC payload):**
    ```
    // Saldırgan, çok sayıda başarısız giriş denemesi yapar
    // ancak uygulama bu denemeleri yeterince detaylı loglamaz
    
    POST /login HTTP/1.1
    Content-Type: application/json
    
    {
      "username": "admin",
      "password": "wrong_password1"
    }
    
    POST /login HTTP/1.1
    Content-Type: application/json
    
    {
      "username": "admin",
      "password": "wrong_password2"
    }
    
    // ... 1000 deneme ...
    
    // Sonunda başarılı giriş
    POST /login HTTP/1.1
    Content-Type: application/json
    
    {
      "username": "admin",
      "password": "correct_password"
    }
    
    // Log: "Login successful"
    ```
*   **4. Analiz:** Uygulama, başarısız giriş denemelerini yeterince detaylı loglamaz ve uyarı mekanizması yoktur. Saldırgan, bu zafiyeti kullanarak brute force saldırısı yapar ve sonunda başarılı olur. Loglarda sadece "Login failed" ve "Login successful" gibi genel mesajlar görülür, bu da saldırının tespit edilmesini zorlaştırır.
*   **5. Kanıt:** Saldırgan, admin hesabına erişim sağlar. Loglarda normal bir giriş olarak kaydedilir ve saldırı tespit edilemez.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Orta
*   **Saldırı Yüzeyi:** İnternete açık, İç network, Loglama sistemleri
*   **Karmaşıklık:** Düşük

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Kritik güvenlik olaylarını loglama, uyarı mekanizmaları kurma, log detayını artırma
*   **Orta vadeli:** Merkezi loglama sistemi kurma, SIEM entegrasyonu yapma, logları güvenli bir şekilde saklama
*   **Uzun vadeli:** Güvenlik olay yönetimi stratejisi geliştirme, düzenli log incelemeleri yapma, otomatik uyarı mekanizmaları kurma, geliştiricilere güvenlik loglama eğitimi verme

---

### 6. Örnek düzeltme kodu (Node.js/Express)

**Güvensiz Kod:**
```javascript
// Login endpoint - yetersiz loglama
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  
  // Veritabanından kullanıcıyı getir
  const user = getUserByUsername(username);
  
  if (user && user.password === password) {
    console.log('Login successful'); // Yetersiz loglama
    req.session.userId = user.id;
    res.json({ success: true });
  } else {
    console.log('Login failed'); // Yetersiz loglama
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// Hassas işlem - loglama yok
app.delete('/users/:id', (req, res) => {
  const userId = req.params.id;
  
  // Kullanıcıyı sil
  deleteUser(userId);
  
  // Herhangi bir loglama yok!
  res.json({ success: true });
});
```

**Güvenli Kod:**
```javascript
const winston = require('winston');
const { combine, timestamp, printf } = winston.format;

// Güvenli loglama formatı
const logFormat = printf(({ level, message, timestamp, userId, ip, userAgent, action }) => {
  return `${timestamp} [${level}] User: ${userId || 'anonymous'}, IP: ${ip}, Action: ${action}, Details: ${message}`;
});

// Logger yapılandırması
const logger = winston.createLogger({
  level: 'info',
  format: combine(
    timestamp(),
    logFormat
  ),
  transports: [
    new winston.transports.File({ filename: 'security.log' }),
    new winston.transports.Console()
  ]
});

// Login endpoint - detaylı loglama
app.post('/login', (req, res) => {
  const { username } = req.body;
  const ip = req.ip;
  const userAgent = req.get('User-Agent');
  
  // Veritabanından kullanıcıyı getir
  const user = getUserByUsername(username);
  
  if (user && user.password === req.body.password) {
    // Başarılı giriş detaylı loglama
    logger.info('User login successful', {
      userId: user.id,
      ip,
      userAgent,
      action: 'LOGIN_SUCCESS'
    });
    
    req.session.userId = user.id;
    res.json({ success: true });
  } else {
    // Başarısız giriş detaylı loglama
    logger.warn('User login failed', {
      userId: user ? user.id : null,
      ip,
      userAgent,
      action: 'LOGIN_FAILURE',
      details: `Username: ${username}`
    });
    
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

// Hassas işlem - detaylı loglama
app.delete('/users/:id', authenticate, authorizeAdmin, (req, res) => {
  const userId = req.params.id;
  const adminId = req.user.id;
  const ip = req.ip;
  const userAgent = req.get('User-Agent');
  
  // Kullanıcıyı silmeden önce loglama
  logger.warn('User deletion initiated', {
    userId: adminId,
    targetUserId: userId,
    ip,
    userAgent,
    action: 'USER_DELETION'
  });
  
  try {
    // Kullanıcıyı sil
    deleteUser(userId);
    
    // Başarılı silme loglama
    logger.info('User deletion successful', {
      userId: adminId,
      targetUserId: userId,
      ip,
      userAgent,
      action: 'USER_DELETION_SUCCESS'
    });
    
    res.json({ success: true });
  } catch (error) {
    // Hata loglama
    logger.error('User deletion failed', {
      userId: adminId,
      targetUserId: userId,
      ip,
      userAgent,
      action: 'USER_DELETION_FAILURE',
      details: error.message
    });
    
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Brute force tespiti için middleware
const bruteForceProtection = {};
const MAX_ATTEMPTS = 5;
const BLOCK_TIME = 15 * 60 * 1000; // 15 dakika

app.use((req, res, next) => {
  const ip = req.ip;
  const now = Date.now();
  
  // IP'nin mevcut durumunu kontrol et
  if (bruteForceProtection[ip]) {
    if (bruteForceProtection[ip].attempts >= MAX_ATTEMPTS) {
      if (now - bruteForceProtection[ip].lastAttempt < BLOCK_TIME) {
        // IP bloke edildi, logla ve uyar
        logger.warn('IP blocked due to brute force attempts', {
          ip,
          action: 'BRUTE_FORCE_BLOCK',
          details: `Attempts: ${bruteForceProtection[ip].attempts}`
        });
        
        // Uyarı mekanizması burada tetiklenebilir
        // sendAlert('BRUTE_FORCE_BLOCK', { ip, attempts: bruteForceProtection[ip].attempts });
        
        return res.status(429).json({ error: 'Too many attempts, please try again later' });
      } else {
        // Blok süresi doldu, sıfırla
        bruteForceProtection[ip] = { attempts: 0, lastAttempt: now };
      }
    }
  }
  
  next();
});
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Mevcut loglama mekanizmalarını analiz et
*   [ ] Kritik güvenlik olaylarını belirle
*   [ ] Uyarı gerektiren senaryoları tanımla

**Düzeltme**
*   [ ] Detaylı güvenlik loglaması uygula
*   [ ] Merkezi loglama sistemi kur
*   [ ] Uyarı mekanizmaları oluştur
*   [ ] Logları güvenli bir şekilde sakla

**Doğrulama**
*   [ ] Loglama mekanizmalarını test et
*   [ ] Uyarı sistemlerini doğrula
*   [ ] Penetrasyon testi yaptır
*   [ ] Düzenli log incelemeleri yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Kritik güvenlik olaylarını izle
    *   Anormal kullanıcı davranışlarını tespit et
    *   Regex: `(?i)(login|authentication|authorization).*fail|fail.*(login|authentication|authorization)`
*   **Anomali Tespiti:** 
    *   Kısa sürede çok sayıda başarısız giriş denemesi
    *   Normal dışı konumlardan yapılan erişim denemeleri
    *   Aynı IP adresinden çok sayıda farklı kullanıcı hesabına giriş denemeleri
    *   Hassas işlemlerdeki anormal artışlar
