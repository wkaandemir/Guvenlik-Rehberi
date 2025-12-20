### Güvenlik Açığı: Oturum Çerezlerinin Otomatik Gönderilmesi ile Tetiklenen CSRF ve Oturum Kaçırma Zafiyeti

### 1. Özet ve etki
* **Yönetici Özeti:**  
  Uygulama, oturum yönetimini tarayıcı çerezlerine (session cookie) dayandırmakta ve `Set-Cookie` ile atılan oturum belirteçleri tarayıcı tarafından hedef host/port/path kurallarına göre otomatik olarak gönderilmektedir. Bu davranış, üçüncü taraf veya kötü niyetli sayfalardan gelen isteklerin (Cross‑Site Request Forgery — CSRF) hedef uygulama üzerinde yetkili kullanıcı adına işlem yapmasına izin verebilir. Ayrıca çerezlerin uygun güvenlik öznitelikleri (`HttpOnly`, `Secure`, `SameSite`) eksik veya yanlış yapılandırılmışsa, oturum kaçırma, kimlik doğrulama atlatma ve veri sızıntısı riskleri artar. İş/sistem üzerindeki etkiler: yetkisiz işlem gerçekleştirme (para transferi, ayar değişikliği), kullanıcı veri sızıntısı, hesap ele geçirme, hizmet bütünlüğünün bozulması ve itibar kaybı.
* **Etkilenen bileşenler:**  
  - **API endpoint'leri:** Oturum gerektiren POST/PUT/DELETE endpoint'leri (ör. `/account/transfer`, `/user/settings`, `/orders/create`).  
  - **Oturum yönetimi katmanı:** Session store (in-memory, Redis, DB) ve session token üretim/yenileme kodu.  
  - **HTTP yanıt başlıkları:** `Set-Cookie` üretimi yapan middleware veya framework konfigürasyonu.  
  - **Frontend:** Tarayıcıda çalışan JavaScript kodu ve form gönderimleri.  
  - **Reverse proxy / CDN / WAF:** Cookie yönlendirme ve header manipülasyonu kuralları.  
  - **Authentication middleware:** Token doğrulama, CSRF koruma mekanizmaları (varsa) ve eksiklikleri.

---

### 2. Teknik detay (nasıl çalışıyor)
- **Adım 1 — Oturum oluşturma:** Kullanıcı kimlik doğrulaması sonrası sunucu `Set-Cookie: session=<token>; Path=/; Domain=example.com` gibi bir yanıt gönderir. Tarayıcı bu çerezi yerel cookie store'a kaydeder.  
- **Adım 2 — Oturumun otomatik eklenmesi:** Tarayıcı, aynı domain/path kurallarına uyan sonraki tüm isteklerde (GET/POST/PUT/DELETE) otomatik olarak `Cookie: session=<token>` başlığını ekler. Bu, kullanıcı her istekte tekrar kimlik doğrulaması yapmadan oturumun devam etmesini sağlar.  
- **Adım 3 — CSRF senaryosu:** Kötü niyetli bir site (attacker.com) kullanıcıyı kandırarak veya sosyal mühendislikle example.com üzerinde yetkili bir kullanıcıyı hedef alan bir istek tetikler (ör. `<form action="https://example.com/account/transfer" method="POST">...`). Tarayıcı, form submit edildiğinde hedef domain ile eşleşen çerezi otomatik ekler; sunucu isteği yetkili kullanıcı isteği olarak işler.  
- **Adım 4 — Güvenlik özniteliklerinin eksikliği:** Eğer `SameSite` uygun değilse veya `HttpOnly`/`Secure` eksikse:  
  - `SameSite=None` ve `Secure` olmayan çerezler üçüncü taraf isteklerde gönderilebilir.  
  - `HttpOnly` yoksa XSS ile çerez çalınabilir.  
- **Arka planda kod davranışı:** Sunucu, gelen istekte sadece çerezdeki session token'ı doğrular; ek bir CSRF token doğrulaması veya isteğin kaynağına dair kontrol yoksa istek yetkili kabul edilir.  
- **Neden (kök neden):**  
  - **Eksik/yanlış çerez öznitelikleri** (`SameSite`, `HttpOnly`, `Secure`) ve **CSRF koruması yokluğu**.  
  - **Stateful oturum modeline** dayalı tasarım ve isteklerin yalnızca çerez varlığına göre yetkilendirilmesi.  
  - **Sunucu tarafında origin/referer kontrolü veya anti‑CSRF token doğrulaması uygulanmaması.**

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

* **1. Hedef:**  
  - `https://example.com/account/transfer` — POST, oturum gerektirir, body: `amount`, `to_account`.
* **2. Normal istek (meşru):**
```http
POST /account/transfer HTTP/1.1
Host: example.com
Cookie: session=eyJhbGciOiJI...
Content-Type: application/x-www-form-urlencoded

amount=100&to_account=TR1234567890
```
* **3. Manipüle edilmiş istek (PoC payload):**  
  - Kötü niyetli sayfa HTML:
```html
<html>
  <body>
    <form id="csrfForm" action="https://example.com/account/transfer" method="POST">
      <input type="hidden" name="amount" value="1000">
      <input type="hidden" name="to_account" value="TRATTACKER000">
    </form>
    <script>
      // Otomatik gönderim; kullanıcı farkında olmadan istek gönderilir
      document.getElementById('csrfForm').submit();
    </script>
  </body>
</html>
```
* **4. Analiz:**  
  - Kullanıcı (oturumlu) kötü niyetli sayfayı ziyaret ettiğinde tarayıcı, `example.com` için saklı `session` çerezini otomatik olarak hedef isteğe ekler. Sunucu çerezdeki token'ı doğrular ve isteği yetkili kullanıcı isteği olarak işler; transfer gerçekleşir.  
  - Eğer `SameSite=Strict` veya `Lax` (uygun şekilde) ayarlıysa ve istek cross-site ise tarayıcı çerezi göndermez; saldırı başarısız olur. Eğer `HttpOnly` eksikse XSS ile token çalınabilir; `Secure` eksikse çerez HTTP üzerinden sızabilir.  
* **5. Kanıt:**  
  - **Sunucu logları:** Başarılı POST istekleri, `200`/`201` veya işlem loglarında `transfer` işlemi; `user_id` ile ilişkilendirilmiş beklenmeyen alıcı hesap.  
  - **İşlem geçmişi:** Hedef kullanıcının hesap hareketlerinde beklenmeyen transfer.  
  - **Access log:** İstek `Referer` veya `Origin` başlığında attacker.com görünmesi (bazı tarayıcılar referer gönderebilir).  
  - **SIEM uyarısı:** Ani yüksek değerli transferler veya aynı kullanıcıdan farklı IP'lerden kısa sürede işlem.

---

### 4. Risk değerlendirmesi
* **Kritiklik:** Yüksek — yetkili kullanıcı adına istek yapılması doğrudan finansal/işlemsel zarar oluşturabilir; veri bütünlüğü ve gizliliği tehlikeye girer.  
* **Saldırı Yüzeyi:** İnternete açık — saldırganın hedef kullanıcıyı tarayıcıda etkilemesi yeterlidir (ör. sosyal mühendislik, kötü reklam, e‑posta).  
* **Karmaşıklık:** Düşük ila Orta — PoC basit HTML form submit ile çalışır; kullanıcı etkileşimi (ziyaret) gerektirir ancak teknik karmaşıklık düşüktür.

---

### 5. Kalıcı çözümler ve öneriler
* **Kısa vadeli (Acil):**  
  - `Set-Cookie` yanıtlarında **`SameSite=Strict` veya `SameSite=Lax`** (uygulama akışına göre) ve **`Secure; HttpOnly`** ekleyin. Eğer üçüncü taraf entegrasyonları varsa `SameSite=None; Secure` kullanımı dikkatle değerlendirilmelidir.  
  - Kritik endpoint'ler için **POST isteklerinde Referer/Origin kontrolü** ekleyin (sunucu tarafında doğrulayın).  
  - WAF üzerinde geçici kural: `Content-Type: application/x-www-form-urlencoded` ile gelen cross-site POST isteklerini, `Origin` veya `Referer` başlığı olmayan istekleri veya beklenmeyen `Origin` değerlerini engelle veya incelemeye al.  
* **Orta vadeli:**  
  - **Anti‑CSRF token** uygulayın (synchronizer token pattern veya double-submit cookie). Her state‑changing istekte sunucu tarafı token doğrulaması zorunlu olsun.  
  - Oturum yönetimini gözden geçirip **session token yenileme** (rotasyon) ve kısa ömürlü token kullanımı uygulayın.  
  - `HttpOnly` çerezleri kullanarak XSS kaynaklı çerez hırsızlığını azaltın.  
  - Tüm uygulama trafiğini **HTTPS** ile zorunlu kılın; HSTS başlığı uygulayın.  
* **Uzun vadeli:**  
  - Mimari olarak **stateless token (JWT)** veya **stateful session** tercihlerini güvenlik gereksinimlerine göre yeniden değerlendirin; token imzalama, revocation listeleri, refresh token stratejileri uygulayın.  
  - Güvenlik eğitimleri: geliştiricilere CSRF, SameSite, çerez öznitelikleri ve güvenli oturum yönetimi konularında düzenli eğitim verin.  
  - CI/CD pipeline içine **automated security tests** (SAST, DAST, dependency scanning) ekleyin; PR aşamasında CSRF koruması kontrolü yapın.  
  - Uygulama güvenlik mimarisi için threat modeling ve periyodik pentest planlayın.

---

### 6. Örnek düzeltme kodu (Node.js / Express)

**Güvensiz (örnek):**
```js
// app.js (GÜVENSİZ örnek)
const express = require('express');
const app = express();

app.post('/account/transfer', (req, res) => {
  // Sadece cookie'deki session'a güveniliyor, CSRF kontrolü yok
  const session = req.cookies.session;
  if (!session || !isValidSession(session)) return res.status(401).send('Unauthorized');
  // İşlem gerçekleştiriliyor
  transfer(req.body.to_account, req.body.amount, session.userId);
  res.send('OK');
});
```

**Güvenli (önerilen):**
```js
// app-secure.js (ÖNERİLEN)
// Gereksinimler: express, cookie-parser, csurf, helmet, express-session (veya redis-store)
const express = require('express');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const csurf = require('csurf');
const helmet = require('helmet');

const app = express();
app.use(helmet());
app.use(express.urlencoded({ extended: false }));
app.use(express.json());
app.use(cookieParser());

// Session konfigürasyonu (örnek Redis store önerilir)
app.use(session({
  name: 'session',
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,      // JS ile erişilemez
    secure: true,        // sadece HTTPS
    sameSite: 'lax',     // CSRF riskini azaltır; uygulama akışına göre 'strict' tercih edilebilir
    maxAge: 1000 * 60 * 60 // 1 saat
  }
}));

// CSRF koruması (synchronizer token)
const csrfProtection = csurf({ cookie: false }); // token session içinde tutulur
app.use((req, res, next) => {
  // GET isteklerinde token'ı frontend'e ver
  if (req.method === 'GET') {
    res.locals.csrfToken = req.csrfToken ? req.csrfToken() : null;
  }
  next();
});

// Örnek: transfer formu GET -> token verilir
app.get('/account/transfer', csrfProtection, (req, res) => {
  res.send(`<form method="POST" action="/account/transfer">
              <input type="hidden" name="_csrf" value="${req.csrfToken()}">
              <input name="to_account">
              <input name="amount">
              <button type="submit">Transfer</button>
            </form>`);
});

// POST isteğinde CSRF token doğrulanır
app.post('/account/transfer', csrfProtection, (req, res) => {
  const user = req.session.user;
  if (!user) return res.status(401).send('Unauthorized');

  // Ek kontroller: amount limitleri, 2FA gereksinimi vs.
  if (req.body.amount > 10000) {
    // yüksek tutarlı işlemler için ek doğrulama (2FA) iste
    return res.status(403).send('Additional verification required');
  }

  transfer(req.body.to_account, req.body.amount, user.id);
  res.send('Transfer completed');
});
```

**Açıklamalar:**  
- `sameSite: 'lax'` çoğu durumda CSRF riskini azaltır; kritik işlemler için `strict` veya ek CSRF token doğrulaması kullanılmalıdır.  
- `HttpOnly` ve `Secure` çerez öznitelikleri XSS ve network‑level sızıntıları azaltır.  
- `csurf` gibi kütüphaneler synchronizer token pattern uygular; alternatif olarak double‑submit cookie pattern de kullanılabilir.

---

### 7. Kontrol Listesi (Checklist)

**Hazırlık**
* [ ] Tüm oturum çerezlerinin `HttpOnly` olarak ayarlandığını doğrula.  
* [ ] Tüm oturum çerezlerinin `Secure` ve uygun `SameSite` değerine sahip olduğunu doğrula.  
* [ ] Kritik endpoint'lerin listesi çıkarıldı (state‑changing işlemler).  

**Düzeltme**
* [ ] Sunucu tarafında CSRF token doğrulaması uygulandı (tüm state‑changing POST/PUT/DELETE).  
* [ ] Referer/Origin kontrolü eklendi ve whitelist oluşturuldu.  
* [ ] WAF kuralları ile şüpheli cross‑site POST istekleri engellendi/izleme altına alındı.  
* [ ] HTTPS ve HSTS zorunlu hale getirildi.  
* [ ] Session timeout ve token rotasyonu uygulandı.  

**Doğrulama**
* [ ] Oturum çerezleri tarayıcıda test edilerek cross‑site isteklerde gönderilip gönderilmediği kontrol edildi.  
* [ ] Oturum açmış kullanıcı ile kötü niyetli sayfa PoC testi (kontrollü ortamda) yapıldı ve saldırı başarısızlığı doğrulandı.  
* [ ] Otomatik testler (DAST) ile CSRF zafiyet taraması eklendi.  
* [ ] Penetrasyon testi ile yüksek riskli senaryolar doğrulandı.

---

### 8. İzleme ve uyarılar
* **SIEM/Log Kuralları:**  
  - **Regex / Pattern örnekleri:**  
    - Yüksek değerli transfer POST istekleri: `POST .* /account/transfer` ve `amount=(?:[1-9]\d{4,})` (örn. 10000 üzeri) — tetiklendiğinde uyarı.  
    - Cross‑site kaynaklı istekler: `Origin: (?!https:\/\/(www\.)?example\.com)` veya `Referer: (?!https:\/\/(www\.)?example\.com)` — bu başlıklarla gelen state‑changing istekleri işaretle.  
    - CSRF token eksikliği: `POST .*` isteklerinde `_csrf` veya `X-CSRF-Token` başlığı yoksa uyar.  
    - Aynı kullanıcı hesabından kısa sürede farklı IP'lerden gelen yüksek riskli işlemler.  
  - **Örnek SIEM kuralı (mantık):**  
    - Eğer `POST /account/transfer` ve (`Origin` yok veya Origin ∉ whitelist) ve `amount > threshold` → yüksek öncelikli uyarı oluştur.  
* **Anomali Tespiti:**  
  - Beklenmeyen `Referer`/`Origin` değerleri ile gelen state‑changing istekleri.  
  - Aynı session token ile farklı coğrafi IP'lerden kısa sürede gelen istekler.  
  - Kullanıcı davranışında ani değişiklikler: normalde hiç transfer yapmayan bir kullanıcıdan yüksek tutarlı transfer.  
  - CSRF token doğrulama hatalarının artışı (ör. 401/403 sayıları).  
* **Loglama önerileri:**  
  - Her state‑changing istekte `user_id`, `session_id` (hashlenmiş), `origin`, `referer`, `client_ip`, `user_agent`, `timestamp` kaydedilsin.  
  - Hassas veriler loglanmasın (tam hesap numaraları, parolalar). Loglarda sadece işlem meta‑verisi ve anonimleştirilmiş kimlik bilgileri tutulmalı.  
* **Uyarı yanıtı (playbook):**  
  - Uyarı alındığında otomatik olarak ilgili hesabı geçici kilitleme veya ek doğrulama (2FA) tetikleme seçeneği.  
  - Güvenlik ekibine bildirim, olay incelemesi ve gerekirse kullanıcı bilgilendirmesi.

---

### 9. Kaynakça
- [Oturum Çerezlerinin Otomatik Gönderilmesi ile Tetiklenen CSRF - YouTube](https://www.youtube.com/watch?v=CKHai0OW6BY&list=PLwP4ObPL5GY940XhCtAykxLxLEOKCu0nT&index=3)


