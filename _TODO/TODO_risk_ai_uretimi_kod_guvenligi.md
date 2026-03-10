# TODO: AI-Uretimi Kod Guvenlik Riski (SusVibes) — Yuksek Zafiyet Orani

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | AI-Uretimi Kod Guvenlik Riski (SusVibes) |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | YENI: AI_IDE_Guvenligi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Subat_2026.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Subat_2026.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Subat 2026'da yayimlanan iki buyuk arastirma — Carnegie Mellon Universitesi/VerSprite ortakligi ile gerceklestirilen SusVibes calismasi ve Tenzai arastirmasi — vibecoding ile uretilen kodun guvenlik profilini sayisal verilerle ortaya koymustur. SusVibes bulgularina gore Claude 4 Sonnet gibi gelismis modellerin fonksiyonel basari orani %61 olsa da, uretilen cozumlerin yalnizca %10.5'i guvenlidir. Daha da endise verici olarak, AI modellerine "guvenli kod yaz" talimati vermek guveniligi artirmak yerine fonksiyonel dogrulugu %6 oraninda dusurmektedir. Tenzai arastirmasi ise Replit, Claude Code, Cursor, Devin ve Codex gibi araclarla insa edilen 15 tam olcekli uygulamada 69 kritik guvenlik acigi saptamistir — hicbiri CSP, HSTS veya X-Frame-Options basliklarini varsayilan olarak icermemektedir. Bu veriler, AI tarafindan uretilen kodun %24.7 ila %45 oraninda guvenlik acigi barindirdigini ve vibecoding'in bir "guvenlik borcu katalizoru" oldugunu kanitlamaktadir.
*   **Etkilenen bilesenler:** Tum vibecoding ve AI destekli kod uretimi platformlari (Cursor, Claude Code, GitHub Copilot, Replit Agent, Devin, Codex, Windsurf), AI tarafindan uretilen web uygulamalari (eksik guvenlik basliklari), veritabani erisim katmanlari (SQL injection, SSRF), oturum yonetim mekanizmalari, CI/CD pipeline'lari (SAST kapisi eksikligi)

---

### 2. Teknik detay (nasil calisiyor)
*   **SusVibes Arastirmasi (Carnegie Mellon/VerSprite):** AI modellerinin guvenlik acisindan performansini olcmek icin tasarlanmis kapsamli bir benchmark calismasi. Claude 4 Sonnet, GPT-4o, Gemini Pro gibi modeller degerlendirilmistir. Sonuclar, fonksiyonel dogrulugun guvenlik ile dogru orantili olmadigini gostermektedir — model bir gorevi "dogru" tamamlasa bile urettigi kodun buyuk cogunlugu guvenlik perspektifinden kusurludur.
*   AI modellerinin en sik yaptigi guvenlik hatalari sunlardir: zamanlama saldirilarina acik kod uretmek (timing side-channels — ornegin sifre karsilastirmasinda erken cikis), oturum yonetimi eksiklikleri (session fixation, token yenileme hatasi), yetersiz girdi sanitasyonu (XSS, SQL injection), SSRF filtresi eklemeyi unutmak ve guvenlik basliklarini (CSP, HSTS, X-Frame-Options) varsayilan olarak dahil etmemek.
*   **"Guvenli kod yaz" Paradoksu:** Modellere acikca "guvenli kod uret" talimati verildiginde, modeller aşiri savunmaci (overly defensive) kod uretmekte, bu da fonksiyonel dogrulugu dusururken guvenligi anlamli olcude artirmamaktadir.
*   **Tenzai Arastirmasi:** En populer 15 vibecoding uygulamasinda sistematik guvenlik testi. Tum araclar, bir "link onizleme" ozelligi insaa ederken dahili aglara erisimi engelleyecek SSRF filtrelerini eklemeyi unutmustur. Hicbir uygulama CSP, HSTS veya X-Frame-Options basliklarini varsayilan olarak icermemektedir.
*   **"Raph Wiggum Dongusu":** Gelistiricilerin AI ciktisini okumadan dogrudan kod tabanina dahil etmesi fenomeni. AI destekli kodlama, kod kopyalamayi 4 kat artirmis ve refactoring (yeniden duzenleme) islemlerini %25'ten %10'un altina dusurmustir.
*   **Neden:** Kok neden, AI modellerinin guvenlik bilgisini fonksiyonel dogrulukla ayni oncelikte ogrenememesi, vibecoding paradigmasinin "hiz" ve "kolaylik" odakli olmasi nedeniyle guvenlik kontrollerinin ihmal edilmesi ve gelistiricilerin AI ciktisini elesstierel denetim yapmadan kabul etmesidir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** AI tarafindan uretilmis bir web uygulamasi (vibecoding ile olusturulmus).
*   **2. Normal durum:**
    ```
    1. Gelistirici, AI aracina "kullanici giris sayfasi yaz" der
    2. AI, fonksiyonel olarak calisan bir login formu uretir
    3. Uretilen kod incelenmeden uretime alinir (Raph Wiggum Dongusu)
    4. Uygulama internete acilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```python
    # AI tarafindan uretilmis GUVENSIZ login kodu (kavramsal):

    # Zafiyet 1: Zamanlama saldirisi (timing attack)
    def check_password(stored: str, provided: str) -> bool:
        return stored == provided  # Erken cikis — zamanlama farki!
        # '==' operatoru ilk farkli karakter'de durur,
        # saldirgan yanit suresinden sifre karakterlerini cikarabilir

    # Zafiyet 2: SQL Injection
    def login(username: str, password: str):
        query = f"SELECT * FROM users WHERE name='{username}'"
        # Girdi sanitasyonu yok! ' OR '1'='1 ile bypass

    # Zafiyet 3: Eksik guvenlik basliklari
    # app.get('/', (req, res) => {
    #   res.send(page)  // CSP, HSTS, X-Frame-Options yok!
    # })

    # Zafiyet 4: SSRF'ye acik link onizleme
    # async function previewLink(url) {
    #   const resp = await fetch(url)  // Dahili ag filtresi yok!
    #   return resp.text()
    # }
    ```
*   **4. Analiz:** AI modeli, fonksiyonel olarak calisan (login basarili) ancak guvenlik acisindan kusurlu kod uretmistir. Zamanlama saldirisi, sifre karsilastirmada `==` operatorunun erken cikis yapmasi nedeniyle mumkundur — sabit zamanli karsilastirma (`hmac.compare_digest`) kullanilmamistir. SQL injection, parametreli sorgu yerine string birlestirme kullanilmasindan kaynaklanir. Eksik guvenlik basliklari, clickjacking ve XSS saldirilarina kapi acar. SSRF filtresi eksikligi, dahili ag kaynaklarina (AWS metadata, ic servisler) yetkisiz erisime yol acar.
*   **5. Kanit:** SusVibes arastirmasi, fonksiyonel olarak dogru cozumlerin yalnizca %10.5'inin guvenli oldugunu kanitlamistir. Tenzai, 15 uygulamada 69 kritik hata saptamistir. AI uretimi kodun %24.7-%45 oraninda zafiyet barindirmasi, bagimssiz arastirmalarla dogrulanmistir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** Uygulanamaz (sistemik risk, belirli bir CVE degil; ancak tespit edilen bireysel zafiyetler CVSS 7.0-9.8 araligindadir)
*   **Saldiri Yuzeyi:** Internete acik (AI uretimi web uygulamalari, API'ler, public servisler)
*   **Karmasiklik:** Dusuk (AI uretimi zafiyetler genellikle temel guvenlik kontrollerinin eksikligi — SSRF filtresi, CSP basligi, parametreli sorgu — nedeniyle kolayca istismar edilebilir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** CI/CD pipeline'ina AI uretimi kod icin zorunlu SAST (Semgrep, Snyk, SonarQube) taramasi ekleyin. Her AI uretimi commit'i guvenlik basliklari (CSP, HSTS, X-Frame-Options) ve SSRF filtresi kontrolunden gecirin. Git pre-commit hook ile guvenlik kontrollerinin (auth, csrf, sanitize) kaldirilis durumunu tespit edin. Mevcut AI uretimi kodu retroaktif olarak guvenlik taramasindan gecirin.
*   **Orta vadeli:** SCA (Software Composition Analysis) ile AI'nin ekleedigi bagimliliklari dogrulayin — hayali (hallucinated) veya typosquatting kutuphaneleri tespit eden araclar kullanin. "Mega prompt" yerine "atomik gorevler" stratejisine gecin (ornegin "CRM yap" yerine "OAuth2 middleware yaz"). AI uretimi kodu isaretleyen (AI-generated code tagging) mekanizmalar uygulayin ve bu kodun zorunlu insan incelemesinden gecmesini saglayin. Kod inceleme sureclerini AI uretimi kodun yaygin zafiyetlerine (SSRF, XSS, zamanlama saldirisi, oturum hatasi) odakli egitimlerle guclendirin.
*   **Uzun vadeli:** AI modelleri icin guvenlik odakli fine-tuning ve benchmark standartlari (SusVibes benchmark'i) gelistirilmesini ve benimsenmesini destekleyin. Altyapi duzeyinde guvenlik izolasyonu uygulayin — AI kod uretimindeki guvenlik hatalarini telafi etmek icin NGINX/Cloudflare Zero Trust gibi ag gecidi kontrolleri kullanin. Gelistiricilere "AI destekli muhendislik" ile "vibecoding" arasindaki farki anlatan kapsamli egitim programlari olusturun. Kurumsal vibecoding guvenlik politikasi yayimlayip uyumu periyodik olarak denetleyin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
# === GUVENSIZ: AI tarafindan uretilmis tipik zafiyetli kod ===

# Zafiyet 1: Zamanlama saldirisi
def check_password(stored_hash: str, provided: str) -> bool:
    import hashlib
    provided_hash = hashlib.sha256(provided.encode()).hexdigest()
    return stored_hash == provided_hash  # Zamanlama farki var!

# Zafiyet 2: SQL Injection
def get_user(username: str):
    query = f"SELECT * FROM users WHERE name = '{username}'"
    return db.execute(query)  # Girdi sanitasyonu yok!

# Zafiyet 3: Eksik guvenlik basliklari
from flask import Flask
app = Flask(__name__)
# CSP, HSTS, X-Frame-Options eklenmemis
```

**Guvenli kod:**
```python
# === GUVENLI: AI uretimi kodun duzeltilmis hali ===
import hmac
import hashlib
from flask import Flask
from flask_talisman import Talisman

# Zafiyet 1 Duzeltme: Sabit zamanli karsilastirma (timing-safe)
def check_password_safe(stored_hash: str, provided: str) -> bool:
    provided_hash = hashlib.sha256(provided.encode()).hexdigest()
    # hmac.compare_digest sabit zamanda calisir — zamanlama saldirisi onlenir
    return hmac.compare_digest(stored_hash, provided_hash)

# Zafiyet 2 Duzeltme: Parametreli sorgu (parameterized query)
def get_user_safe(username: str):
    query = "SELECT * FROM users WHERE name = %s"
    return db.execute(query, (username,))  # Parametreli — SQL injection onlenir

# Zafiyet 3 Duzeltme: Guvenlik basliklari (Flask-Talisman)
app = Flask(__name__)
Talisman(app,
    content_security_policy={
        'default-src': "'self'",
        'script-src': "'self'",
        'style-src': "'self' 'unsafe-inline'",
    },
    force_https=True,                    # HSTS otomatik
    frame_options='DENY',                # X-Frame-Options
    strict_transport_security=True,
    strict_transport_security_max_age=31536000,
)
```

```yaml
# === CI/CD Pipeline: AI uretimi kod icin zorunlu guvenlik kapisi ===
# .github/workflows/ai-code-security-gate.yml
name: AI Code Security Gate
on: [push, pull_request]
jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Semgrep SAST Taramasi
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/owasp-top-ten
            p/python
            p/javascript
            p/typescript

      - name: Snyk Guvenlik Taramasi
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Guvenlik Basliklari Kontrolu
        run: |
          # CSP, HSTS, X-Frame-Options varligini kontrol et
          if ! grep -rq "Content-Security-Policy\|content_security_policy" src/; then
            echo "HATA: CSP basligi bulunamadi!"
            exit 1
          fi
          if ! grep -rq "Strict-Transport-Security\|force_https\|strict_transport" src/; then
            echo "HATA: HSTS basligi bulunamadi!"
            exit 1
          fi
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Projelerdeki AI uretimi kod oranini degerlendir
*   [ ] Mevcut SAST/SCA araclarini ve CI/CD pipeline guvenlik kapisi varligini kontrol et
*   [ ] SusVibes ve Tenzai arastirma bulgularini gelistirme ekibiyle paylas
*   [ ] Kod inceleme sureclerinin AI uretimi koda ozel dikkat gosterip gostermedigini degerlendir

**Duzeltme**
*   [ ] CI/CD pipeline'ina Semgrep/Snyk gibi zorunlu SAST kapisi ekle
*   [ ] Her AI uretimi commit icin guvenlik taramasi zorunlu kil
*   [ ] SCA ile AI'nin ekledigi bagimliliklari dogrula (hayali/typosquatting kutuphaneler)
*   [ ] Git pre-commit hook ile guvenlik kontrollerinin (auth, csrf, sanitize, CSP) kaldirilmasini engelle
*   [ ] "Mega prompt" yerine "atomik gorevler" stratejisine gecis baslat
*   [ ] AI uretimi kodu isaretleyen (tagging) mekanizma uygulayin

**Dogrulama**
*   [ ] AI uretimi kodda SSRF, XSS, SQL injection ve zamanlama saldirisi taramasi yap
*   [ ] Eksik guvenlik basliklari (CSP, HSTS, X-Frame-Options) kontrolu gerceklestir
*   [ ] CI/CD kapisi devreye girdiginde guvensiz commit'lerin engellendigi dogrula
*   [ ] Kod inceleme sureclerinin AI uretimi zafiyetleri yakaladigi simule et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   CI/CD pipeline'inda SAST taramasindan gecemeyen commit'leri izle
    *   AI uretimi commit'lerde guvenlik basligi kaldirilmasini veya auth kontrolu silinmesini tespit et
    *   Production ortaminda eksik guvenlik basliklari ile sunulan sayfalari izle
    *   Regex: `(?i)(semgrep.*fail|snyk.*high|security.*gate.*block)`
    *   Regex: `(?i)(removed.*csp|removed.*hsts|deleted.*auth.*check|removed.*csrf)`
*   **Anomali Tespiti:**
    *   Kisa surede cok sayida SAST uyarisi alan commit'ler (AI uretimi yuksek zafiyetli kod belirtisi)
    *   Guvenlik basliklarinin (CSP, HSTS) production ortamda eksik olmasi
    *   AI uretimi commit'te auth/permission kontrollerinin kaldirilmasi
    *   Yeni eklenen bagimliliklarin (npm/pip paketi) bilinen paket registrilerinde bulunmamasi (hallucinated dependency)
    *   Kod kopyalama (code duplication) oraninda ani artis (AI destekli kodlama belirtisi)

---

## Notlar
SusVibes (Carnegie Mellon/VerSprite) ve Tenzai arastirmalari vibecoding guvenlik risklerinin en kapsamli kantidir. Vibecoding "guvenlik borcu katalizoru" olarak tanimlanmaktadir. "Raph Wiggum Dongusu" — AI ciktisini okumadan kabul etme fenomeni — Wikipedia'ya gore AI destekli kodlamanin kod kopyalamayi 4x artirdigini ve refactoring'i %25'ten %10'un altina dusurdugunu gostermektedir. "Guvenli kod yaz" talimati paradoksal olarak fonksiyonel dogrulugu dusurmektedir. OWASP ASI09 (Human-Agent Trust Exploitation) ile dogrudan iliskilidir.
