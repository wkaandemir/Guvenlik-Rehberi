### Güvenlik Açığı: Django ORM Connector SQL Injection — CVE-2025-64459

---

### 1. Özet ve etki

**Yönetici Özeti:**  
Django ORM katmanındaki connector/operand işleme mantığında, kullanıcı kontrollü connector verisinin (ör. `AND`, `OR`, `XOR` gibi operand yerine) doğrudan sorgu yapısına yerleştirilmesi sonucu **SQL Injection** zafiyeti ortaya çıkmaktadır. Saldırgan, uygun payload ile ORM tarafından oluşturulan SQL sorgusunun ortasına kendi `UNION SELECT` veya ek SQL ifadelerini enjekte ederek uygulama katmanına veri sızdırabilir; hassas veriler (kullanıcı adları, parolalar, e‑posta vb.) elde edilebilir ve uygulama davranışı manipüle edilebilir. Zafiyetin etkisi **Veri Sızıntısı** ve **İtibar Kaybı** ile sınırlı kalmayıp, uygun koşullarda uygulama mantığını bozarak daha geniş etkiler yaratabilir (ör. yetkisiz veri erişimi). Bu zafiyetin veritabanı türüne göre (Postgres vs SQLite) exploit edilebilirlik ve sömürü yöntemi farklılıkları vardır; Postgres ortamında exploit daha kolay/etkili olabilir.

**Etkilenen bileşenler:**  
- API endpoint'leri (dinamik filtreleme/arama endpoint'leri).  
- Django ORM katmanı (Q/connector işleme, query builder).  
- Veritabanı sunucusu (özellikle PostgreSQL; SQLite davranışı farklıdır).  
- Debug/diagnostic konfigürasyonları (debug=true durumunda hata ayıklama çıktıları saldırgan için bilgi sızıntısı sağlayabilir).  
- Middleware veya request-parsing katmanı (gelen filtre/connector verisini doğrulamadan ORM'e ileten kod).

---

### 2. Teknik detay (nasıl çalışıyor)

1. **Girdi kabulü ve Q objesi oluşturma:** API, istemciden dinamik filtreler alır; bu filtreler `Q` objeleri veya benzeri yapı ile ORM sorgusuna dönüştürülür. Bazı uygulamalarda connector (operand) değeri de kullanıcıdan alınır ve sorgu yapısına dahil edilir (ör. `connector="or"`).  
2. **Sorgu oluşturma:** ORM katmanı, gelen Q/connector yapısını SQL'e çevirirken connector değerini doğrudan sorgu string'inin içine yerleştirir; parametreler ayrı tutulurken connector sabit bir string olarak eklenir.  
3. **Eksik doğrulama:** Connector için beklenen değerler (ör. `AND`, `OR`, `XOR`, `NONE`) ile gelen değerin doğrulanmaması veya kısıtlanmaması, saldırganın connector alanına özel karakterler ve SQL ifadeleri enjekte etmesine izin verir.  
4. **Parametre yerleştirme farkları:** Bazı veritabanları (Postgres) parametreli sorgu desteği sayesinde sağ tarafı ayrı parametre olarak alırken, SQLite gibi diğer DB'lerde ORM string birleştirme/formatlama yoluna gidebilir; bu durum escaping/parametrization davranışını değiştirir ve exploit yöntemini etkiler.  
5. **Exploit mekanizması:** Saldırgan, connector alanını kapatma (ör. `)`) ve ardından `UNION SELECT ...` gibi bir payload ekleyerek uygulamanın sol tarafındaki orijinal sorgu ile kendi sorgusunu birleştirir; sonuç uygulama katmanına döner ve serializer/response pipeline aracılığıyla hassas veriler sızdırılabilir.  
6. **Hata tabanlı bilgi sızıntısı:** Debug açıkken veya hatalı formatlama durumlarında uygulama hata mesajları (ör. `Render not enough arguments for format string`, SQL syntax error) saldırgana sorgu yapısını ve kolon sayısını öğrenme imkanı verir; bu da exploit sürecini kolaylaştırır.

**Neden (kök neden):**  
- **Input validation eksikliği:** Connector/operand değerinin whitelist ile kısıtlanmaması.  
- **Güvensiz string birleştirme:** ORM veya uygulama katmanında connector değerinin doğrudan SQL string'ine yerleştirilmesi.  
- **Veritabanı-özgü davranışların göz ardı edilmesi:** Farklı DB motorlarının parametre/escaping davranışlarının dikkate alınmaması.  
- **Geliştirme/Debug konfigürasyonları:** Debug modunda ayrıntılı hata çıktılarının üretimi, saldırgana bilgi sızıntısı sağlar.

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

**1. Hedef:**  
`GET /api/articles/search?connector=<connector>&title=<title>&content=<content>` gibi dinamik filtre kabul eden bir endpoint; backend Django ORM ile sorgu oluşturuyor ve connector parametresini sorgu yapısına doğrudan yerleştiriyor.

**2. Normal istek (meşru):**
```
GET /api/articles/search?connector=or&title=city&content=river
```
Uygulama arka planda güvenli şekilde şu gibi bir ORM çağrısı bekler:
```python
# (örnek, conceptual)
q1 = Q(title__icontains='city')
q2 = Q(content__icontains='river')
if connector == 'or':
    qs = Article.objects.filter(q1 | q2)
```

**3. Manipüle edilmiş istek (PoC payload):**
Saldırgan connector parametresine şu değeri gönderir:
```
connector=%29%20UNION%20SELECT%20username,password,1,1%20FROM%20auth_user%20--%20
```
URL-decode edilmiş hali:
```
") UNION SELECT username,password,1,1 FROM auth_user -- 
```

**4. Analiz (payload tetiklenmesi):**
- Uygulama connector değerini doğrulamadan SQL oluşturma aşamasına geçirir.  
- ORM tarafından oluşturulan SQL sorgusu, saldırganın gönderdiği connector ile birleşir; örneğin:
```sql
SELECT id, title, content, publish_date FROM articles WHERE (title ILIKE %s) ) UNION SELECT username,password,1,1 FROM auth_user --  AND (content ILIKE %s)
```
- `)` ile sol sorgu kapatılmış, `UNION SELECT` ile saldırganın sorgusu eklenmiş, `--` ile geri kalan orijinal sorgu yorum satırına alınmıştır.  
- Sonuç olarak `auth_user` tablosundan `username` ve `password` alanları uygulama tarafından döndürülebilir ve serializer/response pipeline aracılığıyla istemciye yansıyabilir.

**5. Kanıt (başarının gözlemlenmesi):**
- API cevabında beklenmeyen alanlar veya kullanıcı bilgileri görünmesi (ör. `username`, `password` değerleri).  
- Sunucu loglarında veya debug çıktısında oluşan SQL sorgusunun içinde `UNION SELECT` ifadesinin görünmesi.  
- Hata durumlarında `Render not enough arguments for format string` veya SQL syntax error mesajları; bu mesajlar saldırganın sorgu yapısını öğrenmesine işaret eder.

---

### 4. Risk değerlendirmesi

- **Kritiklik:** **Yüksek — Kritik** (Veri sızıntısı doğrudan mümkün; hassas kullanıcı verileri açığa çıkabilir).  
- **Saldırı Yüzeyi:** İnternete açık API endpoint'leri (public search/filter endpoint'leri).  
- **Karmaşıklık:** **Orta** — Saldırganın DB tipini ve kolon sayısını öğrenmesi gerekebilir; debug veya bilgi sızıntısı varsa kolaylaşır. Postgres ortamında exploit daha kolay olabilir; SQLite farklı davranış gösterir ve exploit karmaşıklığını etkiler.

---

### 5. Kalıcı çözümler ve öneriler

**Kısa vadeli (Acil):**
- **Connector whitelist kontrolü:** Hemen tüm endpoint'lerde connector parametresini kabul etmeden önce sadece beklenen değerlerle sınırla (ör. `{'and','or','xor','none'}`); aksi halde 400 döndür.  
- **WAF kuralı:** `UNION\s+SELECT` ve tipik SQL injection pattern'lerini engelleyen WAF kuralları ekle.  
- **Debug kapatma:** Prod ortamında `DEBUG=False` olduğundan emin ol; hata mesajları bilgi sızdırmamalı.  
- **Geçici rate-limit:** Şüpheli istekleri hızla sınırlamak için IP bazlı rate limiting uygula.

**Orta vadeli:**
- **Kod düzeltmeleri:** Connector/operand değerini doğrudan SQL string'ine eklemeyi bırak; connector mantığını uygulama içinde enum/whitelist ile kontrol ederek ORM Q objeleriyle güvenli şekilde birleştir.  
- **Parametrization:** Raw SQL gerekiyorsa, tüm kullanıcı girdilerini parametrele ve DB driver'ın parametre bağlama mekanizmasını kullan.  
- **Input validation & sanitization:** Tüm filtre parametreleri için tip ve içerik doğrulaması (regex, max length, allowed chars).  
- **Testler:** Unit/integration testlerine SQL injection senaryolarını ekle.

**Uzun vadeli:**
- **Mimari iyileştirme:** Dinamik sorgu oluşturma katmanını merkezi, güvenli bir helper/servise taşı; tek bir yerde connector doğrulaması ve sorgu inşası yapılsın.  
- **Geliştirici eğitimi:** ORM güvenliği, parametrized queries ve DB-özgü davranışlar konusunda eğitimler düzenle.  
- **CI/CD güvenlik testleri:** SAST/DAST araçlarını pipeline'a entegre et; PR'lerde dinamik sorgu oluşturma kodu için ekstra inceleme zorunlu kıl.  
- **Sürüm ve bağımlılık yönetimi:** Django ve DB driver güncellemelerini takip et; güvenlik yamalarını hızlıca uygulama süreci oluştur.

---

### 6. Örnek düzeltme kodu (Python / Django)

**Güvensiz (örnek hatalı yaklaşım):**
```python
# Güvensiz: connector doğrudan SQL stringine ekleniyor
def search_articles(request):
    connector = request.GET.get('connector', 'and')  # kullanıcı kontrollü
    title = request.GET.get('title', '')
    content = request.GET.get('content', '')

    # Hatalı: connector doğrudan formatlanıyor
    raw_sql = "SELECT id, title, content, publish_date FROM articles WHERE (title ILIKE %s) %s (content ILIKE %s)"
    params = [f"%{title}%", f"%{content}%"]
    # Burada connector stringi doğrudan ekleniyor -> SQLi riski
    final_sql = raw_sql % (params[0], connector, params[1])
    return execute_raw_sql(final_sql)
```

**Güvenli (önerilen):**
```python
from django.db.models import Q
from django.http import JsonResponse, HttpResponseBadRequest

ALLOWED_CONNECTORS = {'and', 'or', 'xor', 'none'}

def safe_search_articles(request):
    # 1) Girdi doğrulama
    connector = request.GET.get('connector', 'and').lower()
    if connector not in ALLOWED_CONNECTORS:
        return HttpResponseBadRequest("Invalid connector")

    title = request.GET.get('title', '').strip()
    content = request.GET.get('content', '').strip()

    # 2) Güvenli Q objesi oluşturma (parametrized, ORM kullanımı)
    q_title = Q(title__icontains=title) if title else Q()
    q_content = Q(content__icontains=content) if content else Q()

    # 3) Connector'a göre güvenli birleştirme
    if connector == 'and':
        qs = Article.objects.filter(q_title & q_content)
    elif connector == 'or':
        qs = Article.objects.filter(q_title | q_content)
    elif connector == 'xor':
        # XOR mantığını ORM ile güvenli şekilde uygulama örneği
        qs = Article.objects.filter((q_title | q_content) & ~(q_title & q_content))
    else:  # 'none' veya default
        qs = Article.objects.filter(q_title) if title else Article.objects.filter(q_content)

    # 4) Limit ve pagination uygulama
    results = qs.values('id', 'title', 'content', 'publish_date')[:100]
    return JsonResponse(list(results), safe=False)
```

**Açıklamalar:**  
- Connector değeri **whitelist** ile sınırlandırıldı.  
- ORM `Q` objeleri kullanılarak sorgu güvenli şekilde inşa edildi; hiçbir kullanıcı girdisi SQL string'ine doğrudan eklenmedi.  
- Eğer raw SQL kaçınılmazsa, `cursor.execute(sql, params)` şeklinde parametre bağlama kullanılmalı; string formatlama (`%` veya `.format`) kullanılmamalıdır.

---

### 7. Kontrol Listesi (Checklist)

**Hazırlık**
*   [ ] Tüm dinamik filtre/connector kabul eden endpoint'lerin envanterini çıkar.  
*   [ ] Prod ortamında `DEBUG=False` olduğundan emin ol.  
*   [ ] WAF/IPS kurallarını gözden geçir ve temel SQLi pattern'lerini etkinleştir.

**Düzeltme**
*   [ ] Connector/operand parametreleri için whitelist doğrulaması ekle.  
*   [ ] Dinamik sorgu oluşturma kodunu ORM `Q` objeleri veya güvenli helper fonksiyonlarla yeniden yaz.  
*   [ ] Raw SQL kullanılıyorsa parametre bağlamayı zorunlu kıl; string interpolation kullanma.  
*   [ ] Girdi uzunluğu, karakter seti ve tip kontrollerini uygula.

**Doğrulama**
*   [ ] Unit/integration testlerine SQLi PoC senaryolarını ekle.  
*   [ ] DAST taraması çalıştır ve ilgili endpoint'leri hedefle.  
*   [ ] Canlı ortamda anomali/alert testleri yap (ör. WAF tetiklemeleri, SIEM kuralları).

---

### 8. İzleme ve uyarılar

**SIEM / Log Kuralları (örnek pattern'ler):**
- **UNION SELECT tespiti (case-insensitive):**  
  Regex: `(?i)\bUNION\s+SELECT\b`  
- **SQL comment injection (satır sonu yorum):**  
  Regex: `--\s*$`  
- **Render/format hataları (Python debug):**  
  Aranacak anahtar kelimeler: `Render not enough arguments for format string`, `format string` , `TypeError: not enough arguments for format string`  
- **Beklenmeyen connector değerleri:**  
  Loglarda `connector` parametresinde alfanümerik dışı karakter veya `)` `%` `UNION` içeren istekler.  
- **Sorgu içinde `auth_user` veya `out_users` gibi tablo isimleri:**  
  Regex: `(?i)\b(auth_user|users|out_users)\b` (sorgu loglarında görünmesi şüpheli olabilir).

**Anomali Tespiti (şüpheli davranışlar):**
- Aynı IP veya API anahtarından kısa sürede çok sayıda farklı connector değeri ile istek gelmesi.  
- Endpoint’ten dönen 500 hata oranında ani artış; özellikle hata mesajlarında format veya SQL syntax hataları.  
- Beklenmeyen alanların (ör. `username`, `password`) API cevaplarında görünmesi.  
- DB tarafında metadata sorguları (tabloların listelenmesi) veya `information_schema` sorgularında artış.  
- WAF/IPS tarafından `UNION SELECT` veya benzeri pattern'ler için tetiklenen alarmların artması.

---

**Kaynak:** Bu doküman, Django ORM katmanında Q/connector kullanımından kaynaklanan CVE-2025-64459 zafiyetinin saldırgan bakış açısından yapılan analizine dayanır; örnekler ve davranış farkları video analizinden türetilmiştir.

- [Django ORM Connector SQL Injection — CVE-2025-64459 - YouTube](https://www.youtube.com/watch?v=78e6n02mkYM)


