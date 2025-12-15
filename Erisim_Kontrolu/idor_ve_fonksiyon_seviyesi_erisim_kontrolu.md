### Güvenlik Açığı: Insecure Direct Object Reference (IDOR) ve Fonksiyon Seviyesi Erişim Kontrolü Eksikliği

---

### 1. Özet ve etki
**Yönetici Özeti:**  
Uygulama, kullanıcıya ait kaynaklara (ör. adres, sipariş, profil) doğrudan ID veya benzeri referanslarla erişim sağlarken yeterli yetkilendirme kontrolleri uygulamamaktadır. Bu durum, bir saldırganın parametre manipülasyonu (ID değiştirerek) yoluyla başka kullanıcıların verilerine erişmesine, güncellemesine veya silmesine izin verebilir. Sonuçlar arasında **veri sızıntısı**, **yetkisiz veri değişikliği**, müşteri gizliliğinin ihlali ve ciddi **itibar / yasal riskler** yer alır. Video eğitiminde e‑ticaret adres yönetimi örneği ve proxy ile istek yakalama gösterimleri üzerinden bu zafiyetin nasıl tespit edildiği ve istismar edildiği uygulamalı olarak gösterilmiştir.

**Etkilenen bileşenler:**  
- **API endpoint'leri:** `GET /addresses/{id}`, `DELETE /addresses/{id}`, `PUT /addresses/{id}`, `POST /orders/{orderId}/items` gibi kaynak-id tabanlı uç noktalar.  
- **Veritabanı:** Adres/sipariş/profil tabloları ve bu tablolardaki birincil anahtar (ID) kolonları.  
- **Authentication / Authorization middleware:** Token/session doğrulama sonrası kaynak bazlı yetki kontrolü eksik veya yetersiz.  
- **Client-side kod / UI:** ID'leri form/URL içinde açıkça veren veya gizlemeyen front-end davranışları.  
- **Proxy / Gateway / Reverse proxy:** İsteklerin yakalanıp değiştirilebildiği geliştirme/test ortamı araçları (ör. intercepting proxy) ile gözlemlenebilir davranışlar.  

---

### 2. Teknik detay (nasıl çalışıyor)
- **Adım 1 — Kaynak referansı kullanımı:** Uygulama, kullanıcıya ait kaynakları veritabanı ID'siyle ilişkilendirir ve bu ID'yi API çağrılarında doğrudan kullanır (ör. `GET /addresses/15`).  
- **Adım 2 — İstek kabulü ve işlem:** Sunucu gelen istekteki ID parametresini alır, veritabanından ilgili kaydı çeker ve sonucu döner; ancak **kullanıcının bu kaynağa erişim hakkı** sunucu tarafında doğrulanmaz veya eksik doğrulanır.  
- **Adım 3 — Parametre manipülasyonu:** Saldırgan, kendi ID'sini başka bir ID ile değiştirerek (`15` → `12`) başka bir kullanıcının verisini görüntüleyebilir/güncelleyebilir/silebilir.  
- **Arka plan davranışı:** Tipik uygulama akışı şu şekildedir: router → authentication (kimlik doğrulama) → controller → model/veritabanı. Yetkilendirme (authorization) adımı ya eksik ya da sadece UI/route seviyesinde (client-side) uygulanmıştır; sunucu tarafında **fonksiyon seviyesi erişim kontrolü** (function-level access control) yoktur.  
- **Neden (kök neden):**  
  - **Eksik/yanlış yetkilendirme kontrolleri** (authorization checks not performed per resource).  
  - **Güvensiz varsayımlar**: "Kullanıcı sadece kendi ID'sini görebilir" gibi client-side varsayımlara güvenme.  
  - **ID'lerin tahmin edilebilir olması** (ardışık, küçük tamsayılar).  
  - **Girdi doğrulama ve erişim kontrolü ayrımının yapılmaması** (input validation var ama ownership kontrolü yok).  

---

### 3. Adım adım istismar (PoC — kavramsal)  
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

* **1. Hedef:**  
  - `https://app.example.com/api/addresses/{addressId}` — Kullanıcının kayıtlı adreslerini yöneten API. (Senaryo: kullanıcı A kendi adreslerini yönetebiliyor; saldırgan A oturumu ile başka bir kullanıcının adresini hedefliyor.)

* **2. Normal istek (meşru):**  
```http
GET /api/addresses/15 HTTP/1.1
Host: app.example.com
Authorization: Bearer <userA_token>
Accept: application/json
```
Sunucu beklenen davranış: token içindeki kullanıcı kimliği ile `addressId=15` kaydının sahibi eşleşiyorsa 200 + adres verisi döner; değilse 403/404.

* **3. Manipüle edilmiş istek (PoC payload):**  
Saldırgan aynı token ile `addressId` parametresini değiştirir:
```http
GET /api/addresses/12 HTTP/1.1
Host: app.example.com
Authorization: Bearer <userA_token>
Accept: application/json
```

* **4. Analiz:**  
  - Eğer sunucu **sadece** `addressId` ile veriyi çekip döndürüyor ve **owner check** yapmıyorsa, `12` id'li kaydın içeriği dönecektir.  
  - Eğer `DELETE` veya `PUT` endpoint'leri de benzer şekilde yetki kontrolü yapmıyorsa, saldırgan başkasının adresini silebilir veya değiştirebilir.  
  - Proxy ile istek yakalandığında, uygulamanın döndürdüğü HTTP durum kodları (200, 302, 404) ve yönlendirme davranışları saldırgan tarafından analiz edilerek hangi ID'lerin var olduğu ve hangi fonksiyonların erişilebilir olduğu anlaşılabilir.

* **5. Kanıt:**  
  - **HTTP yanıtı:** 200 OK + başka kullanıcıya ait açık adres verisi.  
  - **Loglar:** `GET /api/addresses/12` isteğinin `userA` token ile işlendiğine dair erişim logları; uygulama loglarında owner kontrolü yapılmadığına dair izler.  
  - **Davranış:** `DELETE /api/addresses/12` isteği sonrası veritabanında ilgili kaydın silinmesi; uygulama arayüzünde başka kullanıcının adresinin görünmesi.  

---

### 4. Risk değerlendirmesi
* **Kritiklik:** **Yüksek / Kritik** — Kişisel veri sızıntısı ve veri bütünlüğü ihlali doğrudan müşteri gizliliğini ve iş itibarını etkiler.  
* **Saldırı Yüzeyi:** İnternete açık API uç noktaları (özellikle kimlik doğrulaması sonrası erişilen kaynaklar). İç ağda da benzer riskler bulunur; yetkili kullanıcı oturumlarının ele geçirilmesi durumunda etki artar.  
* **Karmaşıklık:** Düşük–Orta — Parametre manipülasyonu ve ID tahmini genellikle kolaydır; başarılı istismar için özel bilgi gerekmez.

---

### 5. Kalıcı çözümler ve öneriler
**Kısa vadeli (Acil):**  
- **WAF kuralı:** Parametre tampering tespit eden kurallar ekleyin (ör. aynı token ile farklı resource-id'lere kısa sürede erişim denemeleri).  
- **Hızlı config değişikliği:** Kritik endpoint'lerde `403` döndürecek geçici kontrol katmanı ekleyin (ör. middleware ile owner check).  
- **Günlük/alert:** Şüpheli ID tarama/brute-force davranışları için anlık uyarı kuralları oluşturun.

**Orta vadeli:**  
- **Sunucu tarafı yetkilendirme:** Her resource erişiminde **ownership** ve **role-based access control (RBAC)** kontrollerini zorunlu kılın. Fonksiyon seviyesinde (controller/service) yetki kontrolleri ekleyin.  
- **ID tahminini zorlaştırma:** UUID veya sufficiently random non-sequential ID'ler kullanın; ID'leri doğrudan tahmin edilemez hale getirin.  
- **Input validation + authorization ayrımı:** Girdi doğrulama (format, tip) ile erişim kontrolünü (owner check) ayrı katmanlarda uygulayın.  
- **Test & QA:** Otomatik güvenlik testleri (fuzzing, parameter tampering testleri) CI pipeline'a ekleyin.

**Uzun vadeli:**  
- **Mimari iyileştirmeler:** Mikroservisler arası yetki modelini merkezi hale getirin (authorization service).  
- **Geliştirici eğitimi:** OWASP Top10, IDOR, access control konularında düzenli eğitimler.  
- **Güvenlik otomasyonu:** SAST/DAST araçları, API security testing (Postman/Newman ile otomatik PoC testleri) ve kod inceleme süreçlerini entegre edin.  
- **Güvenlik politikaları:** Güvenli kodlama standartları ve PR şablonlarında "resource ownership check" zorunluluğu.

---

### 6. Örnek düzeltme kodu (Node.js / Express)

**Güvensiz (örnek):**
```js
// controllers/addressController.js
app.get('/api/addresses/:id', async (req, res) => {
  const id = req.params.id;
  const address = await db.query('SELECT * FROM addresses WHERE id = $1', [id]);
  if (!address) return res.status(404).send();
  return res.json(address);
});
```
*Problem:* Token içindeki kullanıcı ile `address.owner_id` kontrolü yapılmıyor.

**Güvenli (önerilen):**
```js
// middleware/auth.js
function authenticate(req, res, next) {
  // Bearer token doğrulama, userId'yi req.user.id olarak set etme
  req.user = verifyToken(req.headers.authorization);
  if (!req.user) return res.status(401).send({ error: 'Unauthorized' });
  next();
}

// middleware/ownership.js
async function requireOwnership(req, res, next) {
  const resourceId = req.params.id;
  const row = await db.query('SELECT owner_id FROM addresses WHERE id = $1', [resourceId]);
  if (!row) return res.status(404).send();
  if (row.owner_id !== req.user.id && !req.user.isAdmin) {
    return res.status(403).send({ error: 'Forbidden' });
  }
  next();
}

// routes.js
app.get('/api/addresses/:id', authenticate, requireOwnership, async (req, res) => {
  const id = req.params.id;
  const address = await db.query('SELECT id, street, city, postal_code FROM addresses WHERE id = $1', [id]);
  return res.json(address);
});
```
**Açıklamalar:**  
- `authenticate` middleware ile kimlik doğrulama yapılır ve `req.user` set edilir.  
- `requireOwnership` middleware, veritabanından `owner_id` çekip `req.user.id` ile karşılaştırır; admin/role istisnaları burada ele alınır.  
- Endpoint sadece gerekli alanları döner (least privilege / output encoding).

---

### 7. Kontrol Listesi (Checklist)

**Hazırlık**  
* [ ] Tüm resource-id tabanlı endpoint'lerin envanterini çıkar.  
* [ ] Mevcut authentication/authorization akışını belgeleyin.  
* [ ] Kritik veri modellerinde owner/tenant alanlarının varlığını doğrulayın.

**Düzeltme**  
* [ ] Sunucu tarafı ownership kontrolü ekle (her GET/PUT/DELETE için).  
* [ ] Fonksiyon seviyesi erişim kontrollerini (function-level access control) uygulamaya dahil et.  
* [ ] ID'leri tahmin edilemez hale getir (UUID veya benzeri).  
* [ ] Girdi doğrulama ve output encoding uygulamalarını güçlendir.

**Doğrulama**  
* [ ] Otomatik test: Parametre manipülasyonu (ID tampering) testlerini CI'de çalıştır.  
* [ ] Manuel test: Proxy ile istek yakalama ve ID brute-force senaryolarını test et.  
* [ ] Penetrasyon testi: Üçüncü parti veya iç ekip tarafından kapsamlı test yapıldı mı?  
* [ ] Log ve SIEM uyarıları doğrulandı; anormallikler için alarmlar çalışıyor mu?

---

### 8. İzleme ve uyarılar
**SIEM/Log Kuralları:**  
- **Pattern 1 (ID tampering denemesi):** Aynı token ile kısa süre içinde farklı `addresses/\d+` istekleri. Regex örneği:  
  ```
  ^GET\s+/api/addresses/\d+
  ```  
  ve aynı `Authorization` header ile 5 saniye içinde > N farklı id denemesi.  
- **Pattern 2 (yetkisiz erişim):** `403 Forbidden` veya `404` yerine `200` dönen istekler; `userId` ile `owner_id` uyuşmazlığına dair anormal eşleşmeler.  
- **Pattern 3 (silme/güncelleme anormali):** `DELETE /api/addresses/\d+` veya `PUT /api/addresses/\d+` istekleri sonrası ilgili `owner_id` farklıysa alarm.  
- **Log alanları:** `timestamp`, `request_method`, `request_path`, `user_id` (token), `source_ip`, `response_status`, `response_size`, `db_query_id` (varsa).

**Anomali Tespiti:**  
- **Kısa süreli ID taramaları:** Bir token veya IP adresinden ardışık, artan/azalan ID denemeleri.  
- **Yetki yükseltme davranışı:** Normal kullanıcı token'ı ile admin-only fonksiyonlara erişim denemeleri.  
- **Beklenmeyen veri erişimi:** Kullanıcıların normalde erişmediği coğrafi/hesap verilerine erişim.  
- **Başarılı veri değişiklikleri:** Başka kullanıcıya ait veride `PUT`/`DELETE` işlemlerinin gerçekleşmesi.

---

**Kaynak:** Video eğitimindeki Web Security 101 — IDOR ve fonksiyon seviyesi erişim kontrolü anlatımı ve demo örnekleri (e‑ticaret adres yönetimi, proxy ile istek yakalama).


