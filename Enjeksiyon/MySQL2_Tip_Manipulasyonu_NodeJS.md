### Güvenlik Açığı: MySQL2 `escape` / tip-manipülasyonu (detaylı doküman ve checklist)

### 1. Özet ve etki
**Kısa özet:** Node.js uygulamasında parametrik sorgular kullanılıyor olsa bile, MySQL2 kütüphanesinin `escape`/`format` mekanizmasının gelen değerin *tipine* göre farklı davranması ve `stringifyObjects` gibi konfigürasyonların varsayılan olarak `false` olması, saldırganın frontend’den gönderdiği verinin tipini (ör. object veya array) değiştirerek sorgunun mantığını manipüle etmesine ve sonuçta **authentication bypass** gibi kritik etkiler yaratmasına olanak tanır.  
**Etkilenen bileşenler:** uygulama katmanı (input validation), MySQL2 connector, SQL String kütüphanesi (escape/format).

---

### 2. Teknik detay (nasıl çalışıyor)
- **Tip bazlı escaping:** MySQL2, gelen `value`'nun tipine göre farklı escape yolları seçer; örneğin `array` gelirse `arrayToList` işlevi, `buffer` gelirse buffer dönüşümü, `object` gelirse farklı bir yol izlenir. Bu tip ayrımı, sorgu oluşturulurken beklenmeyen SQL parçalarının oluşmasına neden olabilir.  
- **`stringifyObjects` davranışı:** Connector konfigürasyonunda `stringifyObjects` parametresi `false` ise obje tipleri doğrudan `objectToValues` yoluna düşer; bu durumda obje anahtarları (attribute isimleri) sorguda kolon adı gibi işlenebilir ve sorgunun mantığı değiştirilebilir.  
- **Pratik sonuç:** Saldırgan `password` alanına bir obje göndererek sorgunun sağ tarafını `password = password = true` veya benzeri bir yapıya dönüştürebilir; bu da sorgunun her zaman `true` döndürmesine ve kimlik doğrulamanın atlatılmasına yol açar.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Aşağıdaki PoC örneği yalnızca eğitim/defans amaçlıdır. Yetkisiz sistemlerde test etmek yasa dışıdır.

1. **Hedef:** `/login` endpoint’i; body: `{ username, password }` bekliyor. Sunucu MySQL2 ile parametrik sorgu kullanıyor.  
2. **Normal istek:** `{ "username":"admin", "password":"secret" }` → sorgu parametreleri escape edilerek çalışır; başarısızsa giriş reddedilir.  
3. **Manipüle edilmiş istek (PoC payload):**
```json
{
  "username": "admin",
  "password": { "Mehmet": 1 }
}
```
4. **Neden çalışır:** MySQL2/SQL String, obje tipini `objectToValues` ile işler; obje anahtarı (`Mehmet`) sorguda kolon gibi ele alınır ve değer `1` olarak escape edilir; sonuçta sorgu mantığı `password = password = 1` veya `... AND (password = password) = true` gibi bir yapıya indirgenebilir ve sorgu `true` döner; authentication bypass gerçekleşir.  
5. **Doğrulama:** Veritabanı loglarında oluşturulan nihai SQL sorgusunu inceleyerek (MySQL log veya connector debug) saldırının etkisi doğrulanır; sorgunun sağ tarafının sabit `true` döndüğü görülür.

---

### 4. Risk değerlendirmesi
- **CVE potansiyeli:** Bu tür davranışlar uygulama mantığına bağlı olarak kritik (RCE değilse bile tam hesap ele geçirme) güvenlik açığına dönüşebilir.  
- **Saldırı yüzeyi:** Tüm giriş noktaları (API endpointleri, form inputları) risk altındadır; özellikle kimlik doğrulama, yetkilendirme ve sorgu oluşturan tüm kod parçaları.  
- **Zorluk seviyesi:** Orta (saldırganın sadece JSON tipini değiştirmesi yeterli); tespit edilmesi zor olabilir çünkü sorgu oluşturulurken escaping uygulanıyor gibi görünür ancak tip manipülasyonu mantığı değiştirir.

---

### 5. Kalıcı çözümler ve öneriler
**Kısa vadeli (acil):**
- **Input tip kontrolü (en acil):** Tüm girişlerde `username` ve `password` gibi kritik alanlar için **tip doğrulaması** yapın; örneğin `typeof password === 'string'` veya JSON şeması doğrulaması ile sadece beklenen tipleri kabul edin.  
- **`stringifyObjects` ayarı:** Connector konfigürasyonunda `stringifyObjects: true` olarak ayarlamak, obje geldiğinde onu JSON string’e çevirir ve obje attribute’larının sorgu kolonuna dönüşmesini engeller; ancak bu değişiklik mevcut uygulama davranışını bozabilir, dikkatle test edilmelidir.

**Orta vadeli:**
- **Global tip zorunluluğu:** Tüm API katmanında JSON şema doğrulaması (AJV, Joi, Zod vb.) kullanarak beklenen tipleri ve formatları zorunlu kılın.  
- **Sorgu oluşturma katmanını güçlendirme:** ORM veya query builder kullanıyorsanız, parametrelerin tiplerini zorunlu kılan katmanlar ekleyin; ham connector kullanımını minimize edin.  
- **Unit/integration testleri:** Fuzzing ve tip-manipülasyonu testleri ekleyin (ör. password alanına array/object gönderildiğinde testler başarısız olmalı).  
- **Logging & monitoring:** Veritabanı sorgu loglarını ve uygulama hatalarını izleyin; beklenmeyen tiplerin geldiğine dair uyarılar oluşturun.

**Uzun vadeli:**
- **Tip güvenli diller:** Kritik güvenlik kodu ve auth katmanları için tip zorunluluğu olan diller (Go, Rust, TypeScript + runtime schema) tercih edin; ancak dil tek başına çözüm değildir—doğru validasyon şarttır.  
- **Güvenlik incelemeleri:** Dependency chain (SQL String gibi) ve connector kütüphanelerinin güvenlik davranışlarını düzenli olarak gözden geçirin.

---

### 6. Örnek düzeltme kodu (Node.js, Express + MySQL2)
**A. Bağlantı konfigürasyonu (stringifyObjects örneği):**
```js
const mysql = require('mysql2');
const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: process.env.DB_PASS,
  database: 'login',
  // Dikkat: true yapmak mevcut uygulamayı bozabilir; test edin
  stringifyObjects: true
});
```
**B. Giriş endpoint’inde tip kontrolü (Joi örneği):**
```js
const Joi = require('joi');

const loginSchema = Joi.object({
  username: Joi.string().min(1).max(100).required(),
  password: Joi.string().min(1).max(200).required()
});

app.post('/login', async (req, res) => {
  const { error, value } = loginSchema.validate(req.body);
  if (error) return res.status(400).json({ error: 'Invalid input' });

  const { username, password } = value;
  // parametrik sorgu ile güvenli kullanım
  const [rows] = await pool.promise().query(
    'SELECT * FROM accounts WHERE username = ? AND password = ?',
    [username, password]
  );
  // ...
});
```

---

### 7. Kontrol Listesi (Checklist) — Uygulanabilir adımlar
**Hazırlık**
- [ ] Tüm giriş noktalarını envanterleyin (API, formlar, batch işlemler).  
- [ ] MySQL2 ve SQL String gibi connector/escape kütüphanelerinin sürümlerini belirleyin.

**Hızlı düzeltmeler**
- [ ] `username` ve `password` için **runtime tip doğrulaması** ekleyin (string zorunlu).  
- [ ] Connector konfigürasyonunu gözden geçirip `stringifyObjects` değerini değerlendirin; test ortamında `true` ile test edin.  
- [ ] Kritik endpoint’lerde (auth, admin) ek logging açın: gelen payload tipleri ve oluşturulan SQL sorguları (sadece güvenli ortamlarda, hassas veri maskelenmiş).  

**Orta vadeli**
- [ ] Tüm API’lerde JSON şema doğrulaması (AJV/Joi/Zod) uygulayın.  
- [ ] Unit/integration testlerine tip-fuzzing senaryoları ekleyin (object/array gönderimleri).  
- [ ] Sorgu oluşturma katmanını soyutlayın; ham string formatlama yerine parametrik/ORM kullanımını zorunlu kılın.  

**Uzun vadeli**
- [ ] Güvenlik incelemeleri: dependency chain audit (özellikle SQL String ve connector kütüphaneleri).  
- [ ] Kod inceleme süreçlerine tip-manipülasyonu testlerini dahil edin.  
- [ ] Kritik bileşenler için tip-zorunluluğu olan diller veya runtime schema kullanımı değerlendirin.

**Kontrol ve doğrulama**
- [ ] Test ortamında PoC payload ile doğrulama yapın (yetkili ekip tarafından).  
- [ ] Production’da benzer payload’ların gelmesini tespit edecek IDS/alert kuralları oluşturun.  
- [ ] Düzeltme sonrası regression testleri çalıştırın.

---

### 8. İzleme ve uyarılar (örnek kurallar)
- **Uyarı 1:** Bir endpoint’e gelen `password` alanı `object` veya `array` ise yüksek öncelikli uyarı.  
- **Uyarı 2:** Oluşturulan SQL sorgusunda `WHERE` koşulunun sağ tarafı sabit `true` döndürecek şekilde görünüyorsa (ör. `= 1` veya `= true`) alarm.  
- **Uyarı 3:** `stringifyObjects` konfigürasyon değişiklikleri deploy edildiğinde otomatik test tetikle.

---

### 9. Kaynakça
- [MySQL2 Tip Manipülasyonu ve SQL Injection - YouTube](https://youtu.be/Owz-1fRV4bo?si=A3QHCvCPJoBSFviE)