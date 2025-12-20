### Güvenlik Açığı: SQL Injection ile Veritabanı Sorgularının Kontrol Edilmesi

---

### 1. Özet ve etki

**Yönetici Özeti:**  
Uygulama, kullanıcı girdilerini doğrudan SQL sorgularına dahil ederek veritabanı motorunun saldırgan kontrollü SQL ifadelerini çalıştırmasına izin vermektedir. Bu zafiyet sayesinde saldırganlar yetkisiz veri okuma (veri sızıntısı), veri bütünlüğünü bozma (INSERT/UPDATE/DELETE), veritabanı şeması keşfi (information_schema sorguları) ve potansiyel olarak veri tabanını silme veya uzak kod yürütme (dolaylı RCE senaryoları) gerçekleştirebilir. Etki kritik olup müşteri verisi, kimlik bilgileri ve iş sürekliliği açısından ciddi risk oluşturur.

**Etkilenen bileşenler:**  
- Web uygulaması giriş noktaları (GET/POST parametreleri, form alanları, URL parametreleri)  
- API endpointleri (ör. `/search`, `/product?id=`, `/user?name=`)  
- Veritabanı erişim katmanı (DB client kütüphaneleri, ORM konfigürasyonları)  
- Input parsing ve validation middleware'leri  
- Logging/exception handling mekanizmaları (hata gizleme veya hataları sessizce yutan kod)  
- Yönetim panelleri ve debug endpointleri (geliştirme ortamı konfigürasyonları)

---

### 2. Teknik detay (nasıl çalışıyor)

- **Adım 1 — Girdi kabulü:** Kullanıcıdan gelen veri (URL parametresi, form alanı, JSON body) uygulama tarafından doğrulanmadan veya uygun şekilde temizlenmeden alınır.  
- **Adım 2 — Sorgu oluşturma:** Alınan girdi doğrudan string birleştirme veya formatlama ile SQL sorgusuna eklenir; örn. `query = "SELECT * FROM users WHERE username = '" + user + "';"`.  
- **Adım 3 — Veritabanı yürütmesi:** Oluşturulan SQL ifadesi veritabanı motoruna gönderilir. Eğer girdi içinde SQL kontrol karakterleri veya ek ifadeler varsa, veritabanı bunları saldırganın istediği şekilde yorumlar ve çalıştırır.  
- **Adım 4 — Yan etkiler:** Sorgu sonucu uygulama tarafından döndürülür veya loglanır; saldırgan bu çıktıyı kullanarak ek keşif ve sömürü adımları gerçekleştirir (ör. `UNION SELECT`, `information_schema` sorguları, stacked queries).  

**Arka planda kod davranışı:**  
- Güvensiz kod genellikle string interpolation, `sprintf` benzeri fonksiyonlar veya doğrudan `+` ile SQL oluşturur.  
- ORM kullanılsa bile raw query veya dinamik SQL oluşturma noktaları risklidir.  
- Bazı veritabanı sürücüleri stacked queries (aynı istek içinde birden fazla SQL) destekleyebilir; bu, `; DROP TABLE users;` gibi zararlı eklemelere izin verebilir.  
- Hata yönetimi zayıfsa uygulama SQL hata mesajlarını doğrudan döner; bu da bilgi sızıntısına yol açar.

**Neden (kök neden):**  
- **Input validation eksikliği** ve **parametrizasyon yokluğu**.  
- Kullanıcı girdilerinin tip/uzunluk/format kontrollerinin yapılmaması.  
- Güvenli sorgu oluşturma (prepared statements / parameterized queries) kullanılmaması.  
- Veritabanı davranışlarının (ör. string vs numeric coercion, farklı RDBMS davranışları) göz ardı edilmesi.

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

* **1. Hedef:**  
  - Senaryo: `https://app.example.com/product?id=<param>` endpointi ürün detaylarını döndürüyor. Arka uç MySQL benzeri bir veritabanına bağlanıyor.

* **2. Normal istek:**  
  ```
  GET /product?id=42 HTTP/1.1
  Host: app.example.com
  ```
  - Beklenen backend sorgusu (güvenli olmayan örnek):  
    ```sql
    SELECT id, name, price FROM products WHERE id = 42;
    ```

* **3. Manipüle edilmiş istek (PoC payload):**  
  - Basit doğrulama atlatma: `id=42 OR 1=1`  
    ```
    GET /product?id=42%20OR%201=1 HTTP/1.1
    ```
  - UNION ile veri sızdırma: `id=42 UNION SELECT username,password FROM users -- `  
    ```
    GET /product?id=42%20UNION%20SELECT%20username,password%20FROM%20users%20--%20 HTTP/1.1
    ```
  - Information schema keşfi: `id=42 UNION SELECT table_name, column_name FROM information_schema.columns WHERE table_schema=database() -- `

* **4. Analiz:**  
  - Eğer uygulama `id` parametresini doğrudan sorguya ekliyorsa, `OR 1=1` ifadesi WHERE koşulunu her zaman true yapar ve tüm ürünleri döndürebilir.  
  - `UNION SELECT` ile saldırgan veritabanındaki başka tablolardan veri çekebilir; döndürülen kolon sayıları/uyumlu tipler sağlanırsa hassas veriler elde edilir.  
  - `information_schema` sorguları tablo/kolon isimlerini ortaya çıkarır; bu bilgi sonraki hedeflemeler için kullanılır.  
  - Eğer DB stacked queries destekliyorsa `; DROP TABLE users; --` gibi ifadelerle veri silme gerçekleştirilebilir.

* **5. Kanıt:**  
  - Uygulama yanıtında beklenmeyen satırlar veya kullanıcı bilgileri görünmesi.  
  - Sunucu loglarında çalıştırılan SQL ifadelerinin içeriklerinde saldırgan payload'ları.  
  - Veritabanı loglarında `UNION SELECT` veya `information_schema` sorgularının görülmesi.  
  - Hata mesajları: `SQL syntax error near 'UNION SELECT ...'` veya `Column count doesn't match` gibi hatalar.  
  - Beklenmeyen artışlar: aynı IP'den kısa sürede çok sayıda farklı `id` parametresi ile istek.

---

### 4. Risk değerlendirmesi

* **Kritiklik:** Kritik — doğrudan hassas veri sızıntısı ve veri manipülasyonu riski mevcut.  
* **Saldırı Yüzeyi:** İnternete açık endpointler üzerinden istismar edilebilir; ayrıca iç ağdaki servisler de risk altında olabilir.  
* **Karmaşıklık:** Düşük–Orta — temel payloadlar kolayca denenebilir; daha karmaşık sızıntılar için bilgi keşfi gerekebilir.

---

### 5. Kalıcı çözümler ve öneriler

**Kısa vadeli (Acil):**  
- Kritik endpointler için WAF kuralı ekleyin (ör. `UNION`, `' OR 1=1`, `information_schema`, `--`, `; DROP` içeren patternleri engelle).  
- Hata mesajlarını kullanıcıya göstermeyi durdurun; detaylı SQL hatalarını gizleyin.  
- Geliştirme/production konfigürasyonlarını kontrol edin; debug modunu kapatın.

**Orta vadeli:**  
- Tüm veritabanı sorgularını **parametrized queries / prepared statements** ile yeniden yazın.  
- Input validation: tip kontrolü, uzunluk sınırı, whitelist (özellikle ID, sayısal parametreler için).  
- ORM kullanılıyorsa raw query kullanımını minimize edin; gerekiyorsa parametrizasyonu zorunlu kılın.  
- Güvenli kütüphaneler ve güncel DB sürücüleri kullanın.

**Uzun vadeli:**  
- Mimari: DB erişim katmanını tek bir güvenli servis/katmana toplayın; tüm sorgular buradan geçsin.  
- Güvenlik eğitimleri: geliştiricilere SQLi, parametrizasyon, secure coding eğitimi verin.  
- CI/CD: otomatik statik analiz, SAST ve dinamik testler (DAST) ekleyin; PR pipeline'ına SQLi testleri dahil edin.  
- Düzenli pentest ve kod incelemeleri planlayın.

---

### 6. Örnek düzeltme kodu (Python Flask + MySQL)

**Güvensiz kod örneği (Kötü uygulama):**
```python
# Güvensiz: doğrudan string birleştirme
def get_product(db, product_id):
    query = "SELECT id, name, price FROM products WHERE id = %s;" % product_id
    cursor = db.cursor()
    cursor.execute(query)
    return cursor.fetchall()
```

**Güvenli kod örneği (Prepared statement ile):**
```python
# Güvenli: parametrized query kullanımı
def get_product(db, product_id):
    # Tip kontrolü: product_id sadece sayısal olmalı
    try:
        pid = int(product_id)
    except ValueError:
        raise ValueError("Invalid product id")

    query = "SELECT id, name, price FROM products WHERE id = %s;"
    cursor = db.cursor()
    # Parametreler ayrı verilir; DB driver parametreleri güvenli şekilde kaçışlar
    cursor.execute(query, (pid,))
    return cursor.fetchall()
```

**Flask endpoint örneği:**
```python
from flask import Flask, request, jsonify
app = Flask(__name__)

@app.route('/product')
def product():
    product_id = request.args.get('id', '')
    try:
        rows = get_product(db_connection, product_id)
    except ValueError:
        return jsonify({"error": "Bad request"}), 400
    return jsonify(rows)
```

**Node.js örneği (mysql2 kullanarak):**
```javascript
// Güvenli: prepared statements
const mysql = require('mysql2/promise');

async function getProduct(conn, productId) {
  const pid = parseInt(productId, 10);
  if (Number.isNaN(pid)) throw new Error('Invalid id');
  const [rows] = await conn.execute('SELECT id, name, price FROM products WHERE id = ?', [pid]);
  return rows;
}
```

**Açıklamalar:**  
- Parametreler sorgudan ayrı tutulur; DB driver parametreleri güvenli şekilde kaçışlar.  
- Ek olarak tip kontrolü (ör. `int` dönüşümü) ile beklenmeyen string ifadeler engellenir.  
- ORM kullanılıyorsa `where('id', id)` gibi ORM parametre bağlama yöntemleri tercih edilmelidir.

---

### 7. Kontrol Listesi (Checklist)

**Hazırlık**  
*   [ ] Tüm kullanıcı girdilerinin kabul noktalarını envanterle (endpoints, params, headers, cookies).  
*   [ ] Kritik endpointleri (DB yazma/okuma) belirle ve önceliklendir.  
*   [ ] Geliştirme ortamında debug ve verbose SQL hatalarını kapat.

**Düzeltme**  
*   [ ] Tüm SQL sorgularını parametrized queries ile yeniden yaz.  
*   [ ] Input validation kurallarını uygula (tip, uzunluk, whitelist).  
*   [ ] ORM raw query kullanımını azalt; gerekiyorsa review süreci ekle.  
*   [ ] WAF kurallarını kısa vadede devreye al.  
*   [ ] Hata mesajlarını sanitize et ve kullanıcıya detaylı SQL hatası gösterme.

**Doğrulama**  
*   [ ] Otomatik test: SQLi payloadlarını içeren DAST taraması çalıştır.  
*   [ ] Manuel pentest: kritik endpointler için SQLi testleri yap.  
*   [ ] Loglarda anormallik ve hata örüntülerini kontrol et.  
*   [ ] CI pipeline'ına SAST/DAST testlerini entegre et.

---

### 8. İzleme ve uyarılar

**SIEM/Log Kuralları:**  
- **Regex örnekleri** (örnek amaçlı, sistemin regex motoruna göre uyarlanmalı):  
  - SQL meta karakterleri içeren parametreler:  
    ```
    (?i)(\bUNION\b|\bSELECT\b.*\bFROM\b|--|;|/\*|\bINFORMATION_SCHEMA\b|' OR '1'='1'|'\s*OR\s*1=1)
    ```
  - `information_schema` veya `INFORMATION_SCHEMA` sorguları:  
    ```
    (?i)information_schema|table_schema|column_name
    ```
  - Stacked query veya tehlikeli komutlar:  
    ```
    (?i);\s*(DROP|ALTER|TRUNCATE|DELETE|INSERT)\b
    ```
  - Hata tabanlı sızıntı göstergeleri (SQL syntax error mesajları):  
    ```
    (?i)SQL syntax|syntax error|mysql_fetch|pg_query\(|ORA-00933
    ```

**Log alanları ve örnek kurallar:**  
- **Web server access logs:** uzun query string içeren GET istekleri, aynı IP’den kısa sürede çok sayıda farklı parametre.  
- **Application logs:** SQL hata mesajları, exception stack trace içeren kayıtlar. Bu kayıtlar maskelenmeli ve güvenli bir şekilde saklanmalı.  
- **DB audit logs:** `UNION`, `information_schema` sorguları, yüksek sayıda `SELECT` sorgusu yapan kullanıcı hesapları.

**Anomali Tespiti:**  
- Aynı IP’den kısa sürede artan `UNION` veya `information_schema` içeren istekler.  
- Beklenmeyen kolon sayısı hataları veya syntax error artışı.  
- Normal kullanıcı davranışından farklı olarak tek bir endpoint’e yoğun istek (fuzzing belirtisi).  
- Uygulama tarafından döndürülen veri yapısında beklenmeyen alanlar (ör. kullanıcı tablosundan dönen username/password gibi hassas alanlar).

---

## Son notlar
- **Önceliklendirme:** Kritik endpointler (kullanıcı kimlik doğrulama, ödeme, yönetim) öncelikli olarak parametrizasyon ve input validation ile güvence altına alınmalıdır.  
- **Test önerisi:** Düzeltmeler yapıldıktan sonra hem otomatik DAST hem de manuel doğrulama (ör. blind/boolean/time-based SQLi testleri) gerçekleştirilmelidir.  
- **Eğitim:** Geliştirici ekipleri için kısa, uygulamalı secure coding oturumları (parametrizasyon, ORM güvenli kullanımı, input validation) planlayın.

## Kaynakça
- [SQL Injection ile Veritabanı Sorgularının Kontrol Edilmesi - YouTube](https://www.youtube.com/watch?v=WtHnT73NaaQ&list=PLwP4ObPL5GY940XhCtAykxLxLEOKCu0nT&index=1)


