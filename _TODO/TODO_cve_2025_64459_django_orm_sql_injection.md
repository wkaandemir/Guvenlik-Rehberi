# TODO: CVE-2025-64459 — Django ORM Connector SQL Injection

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-64459 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Enjeksiyon/ |
| **Kaynak Arastirma** | Dogrudan rehber olarak olusturuldu |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [django_orm_sql_injection_cve_2025_64459.md](../Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Django ORM katmanindaki connector/operand isleme mantiginda, kullanici kontrollu connector verisinin dogrudan sorgu yapisina yerlestirilmesi sonucu SQL Injection zafiyeti ortaya cikmaktadir. Saldirgan, `UNION SELECT` veya ek SQL ifadeleri enjekte ederek hassas verileri (kullanici adlari, parolalar, e-posta) sizdirabilir. Veritabani turune gore (PostgreSQL vs SQLite) exploit yontemi farklilik gosterir.
*   **Etkilenen bilesenler:** Django ORM (Q/connector isleme, query builder), dinamik filtreleme/arama API endpoint'leri, PostgreSQL ve SQLite veritabani sunuculari, debug/diagnostic konfigurasyonlari.

---

### 2. Teknik detay (nasil calisiyor)
*   API, istemciden dinamik filtreler alir ve bunlari `Q` objeleri ile ORM sorgusuna donusturur. Bazi uygulamalarda connector (operand) degeri de kullanicidan alinir.
*   ORM katmani connector degerini dogrudan SQL string'inin icine yerlestirir; parametreler ayri tutulurken connector sabit bir string olarak eklenir.
*   Connector icin beklenen degerler (`AND`, `OR`, `XOR`) ile gelen degerin dogrulanmamasi, saldirganin SQL ifadeleri enjekte etmesine izin verir.
*   **Neden:** Input validation eksikligi (connector whitelist yok), guvensiz string birlestirme, veritabani-ozgu davranislarin goz ardi edilmesi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Dinamik filtre kabul eden endpoint: `GET /api/articles/search?connector=<val>&title=<val>&content=<val>`
*   **2. Normal durum:**
    ```
    GET /api/articles/search?connector=or&title=city&content=river
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    connector=%29%20UNION%20SELECT%20username,password,1,1%20FROM%20auth_user%20--%20
    ```
    URL-decode: `") UNION SELECT username,password,1,1 FROM auth_user -- `
*   **4. Analiz:** Connector dogrulanmadan SQL olusturma asamasina geciriliyor. `)` ile sol sorgu kapatilir, `UNION SELECT` ile saldirgan sorgusu eklenir, `--` ile kalan kisim yorum satiri yapilir.
*   **5. Kanit:** API cevabinda `username`, `password` gibi beklenmeyen alanlar gorunur; sunucu loglarinda `UNION SELECT` iceren sorgular gozlemlenir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** ~8.6 (tahmini — veri sizdirma dogrudan mumkun)
*   **Saldiri Yuzeyi:** Internete acik API endpoint'leri (public search/filter)
*   **Karmasiklik:** Orta — DB tipi ve kolon sayisi bilgisi gerekebilir; debug aciksa kolaylasir

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Connector parametresi icin whitelist kontrolu (`{'and','or','xor','none'}`), WAF kurali (`UNION\s+SELECT`), prod'da `DEBUG=False`.
*   **Orta vadeli:** Connector/operand degerini dogrudan SQL'e eklemek yerine ORM Q objeleriyle guvenli birlestirme. Raw SQL gerekiyorsa parametre baglama zorunlulugu. Input validation ve sanitization.
*   **Uzun vadeli:** Dinamik sorgu olusturma katmanini merkezi guvenli bir helper/servise tasima. SAST/DAST entegrasyonu. Gelistirici egitimi.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
# Guvensiz: connector dogrudan SQL stringine ekleniyor
def search_articles(request):
    connector = request.GET.get('connector', 'and')  # kullanici kontrollu
    title = request.GET.get('title', '')
    content = request.GET.get('content', '')
    raw_sql = "SELECT id, title, content FROM articles WHERE (title ILIKE %s) %s (content ILIKE %s)"
    final_sql = raw_sql % (params[0], connector, params[1])
    return execute_raw_sql(final_sql)
```

**Guvenli kod:**
```python
from django.db.models import Q
from django.http import JsonResponse, HttpResponseBadRequest

ALLOWED_CONNECTORS = {'and', 'or', 'xor', 'none'}

def safe_search_articles(request):
    connector = request.GET.get('connector', 'and').lower()
    if connector not in ALLOWED_CONNECTORS:
        return HttpResponseBadRequest("Invalid connector")

    title = request.GET.get('title', '').strip()
    content = request.GET.get('content', '').strip()

    q_title = Q(title__icontains=title) if title else Q()
    q_content = Q(content__icontains=content) if content else Q()

    if connector == 'and':
        qs = Article.objects.filter(q_title & q_content)
    elif connector == 'or':
        qs = Article.objects.filter(q_title | q_content)
    elif connector == 'xor':
        qs = Article.objects.filter((q_title | q_content) & ~(q_title & q_content))
    else:
        qs = Article.objects.filter(q_title) if title else Article.objects.filter(q_content)

    results = qs.values('id', 'title', 'content', 'publish_date')[:100]
    return JsonResponse(list(results), safe=False)
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum dinamik filtre/connector kabul eden endpoint'lerin envanterini cikar.
*   [ ] Prod ortaminda `DEBUG=False` oldugundan emin ol.
*   [ ] WAF/IPS kurallarini gozden gecir ve temel SQLi pattern'lerini etkinlestir.

**Duzeltme**
*   [ ] Connector/operand parametreleri icin whitelist dogrulamasi ekle.
*   [ ] Dinamik sorgu olusturma kodunu ORM Q objeleri veya guvenli helper fonksiyonlarla yeniden yaz.
*   [ ] Raw SQL kullaniliyorsa parametre baglamayi zorunlu kil.
*   [ ] Girdi uzunlugu, karakter seti ve tip kontrollerini uygula.

**Dogrulama**
*   [ ] Unit/integration testlerine SQLi PoC senaryolarini ekle.
*   [ ] DAST taramasi calistir ve ilgili endpoint'leri hedefle.
*   [ ] Canli ortamda anomali/alert testleri yap.

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   UNION SELECT tespiti: Regex: `(?i)\bUNION\s+SELECT\b`
    *   SQL comment injection: Regex: `--\s*$`
    *   Beklenmeyen connector degerleri: Loglarda `connector` parametresinde alfanumerik disi karakter iceren istekler.
*   **Anomali Tespiti:**
    *   Ayni IP'den kisa surede farkli connector degerleri ile istek gelmesi.
    *   Endpoint'ten donen 500 hata oraninda ani artis.
    *   API cevaplarinda beklenmeyen alanlar (username, password).
    *   DB tarafinda `information_schema` sorgularinda artis.

---

## Notlar
- Exploit davranisi veritabani turune gore degisir: PostgreSQL ortaminda daha kolay, SQLite'da farkli teknik gerektirir.
- Detayli analiz ve ornekler icin mevcut rehber dokumana bakin: [Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md](../Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md)
- Kaynak: [Django ORM Connector SQL Injection — CVE-2025-64459 (YouTube)](https://www.youtube.com/watch?v=78e6n02mkYM)
