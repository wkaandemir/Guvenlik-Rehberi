# TODO: AI Silahlandirilmasi / Shadow AI — AI Tabanli Tehditler ve Kontrolsuz AI Kullanimi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | AI Silahlandirilmasi / Shadow AI |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | YENI: AI_IDE_Guvenligi/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** 2026 yilinda yapay zeka, siber saldirilarin hizini ve olcegini dramatik bir sekilde artirmaktadir. Tehdit aktorleri, otonom LLM ajanlari kullanarak kurbana ozel, hatasiz ve ikna edici phishing zincirleri olusturmakta; deepfake teknolojisi uzerinden sesli ve goruntulu dolandiriciliklar (vishing) ile kurumsal onay sureclerini bypasslamaktadir. Bunun yaninda, kurumlar icin en buyuk ic tehditlerden biri "Shadow AI" olarak tanimlanan fenomendir: calisanlarin veya gelistiricilerin IT departmaninin bilgisi ve onayı disinda kontrolsuz AI araclari kullanmasi, sistemlere gizli zafiyetler enjekte etmekte veya hassas kurumsal verilerin disari sizmasina neden olmaktadir. WEF (World Economic Forum) 2026 raporuna gore CEO'larin en buyuk korkusu artik fidye yazilimlari degil, AI tabanli dolandiriciliklar ve manipulasyonlardir. Bu iki tehdit — AI silahlandirilmasi ve Shadow AI — birlestiginde kurumsal guvenlik duvarlari hem disindan hem de icindan zayiflamaktadir.
*   **Etkilenen bilesenler:** Kurumsal e-posta ve iletisim sistemleri (phishing hedefi), finans ve onay surecleri (deepfake vishing hedefi), gelistirici ortamlari ve CI/CD pipeline'lari (Shadow AI uzerinden zafiyet enjeksiyonu), kurumsal veri guvenligi (kontrolsuz AI araclar uzerinden veri sizdirma), SOC/SIEM altyapisi (AI hizinda saldiri tespiti kapasitesi)

---

### 2. Teknik detay (nasil calisiyor)
*   **AI Silahlandirilmasi (Industrialization of AI Attacks):** Tehdit aktorleri, otonom LLM ajanlarini siber saldiri zincirlerinin her asamasinda kullanmaktadir. Kesfiften (OSINT toplama) ilk erisime (hedefli phishing), yanal hareketten (privilege escalation) veri sizdirmaya kadar tum surec AI ile otomatize edilmektedir. Ozellikle spear phishing kampanyalarinda LLM'ler, hedef kisinin LinkedIn profili, blog yazilari ve sosyal medya paylasimlarini analiz ederek kisisellestirilmis, dilbilgisi hatasiz ve ikna edici mesajlar uretmektedir.
*   **Deepfake Vishing:** Ses ve goruntu deepfake teknolojisi, telefon veya video konferans uzerinden kurumsal onay sureclerini atlatmak icin kullanilmaktadir. Saldirganlar, CFO veya CEO'nun sesini taklit ederek acil para transferi talep edebilmekte veya IT yoneticisinin goruntusunu kullanarak sifre sifirlama islemleri baslayabilmektedir.
*   **Shadow AI:** Calisan ve gelistiricilerin IT departmaninin bilgisi disinda ChatGPT, Claude, Copilot gibi AI araclarini kullanmasi. Bu araclar uzerinden hassas kurumsal veriler (kod, musteri bilgileri, stratejik dokumanlar) ucuncu taraf sunucularina gonderilir. AI araclari tarafindan uretilen kod, incelenmeden uretim ortamina dahil edildiginde gizli zafiyetler (SSRF, XSS, SQL injection) enjekte edilebilir.
*   **Zscaler ve Check Point raporlarina gore:** AI destekli saldiri araclari, geleneksel saldiri araclarinin 10-50 katı hizinda kesfif ve istismar gerceklestirebilmektedir. Savunma tarafinin tespit ve mudahale suresi (MTTD/MTTR) AI saldirisi hizina yetismemektedir.
*   **Neden:** Kok neden, AI araclarinin erisilebilirliginin herkese acik hale gelmesi (demokratizasyon), kurumsal AI kullanim politikalarinin yetersizligi, AI hizinda saldiri tespit kapasitesinin bulunmamasi ve gelistiricilerin AI uretimi kodu dogrulamadan kabul etmesidir (Raph Wiggum Dongusu).

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Kurumsal finans departmani (deepfake vishing) ve gelistirme ekibi (Shadow AI uzerinden zafiyet enjeksiyonu).
*   **2. Normal durum:**
    ```
    1. Calisan, onaylanmis iletisim kanallari uzerinden talimat alir
    2. Finans islemleri cift onay mekanizmasi ile dogrulanir
    3. Gelistiriciler yalnizca onaylanmis araclari kullanir ve kod incelemesi yapilir
    4. IT departmani kurumsal aglardaki tum araclari ve veri akislarini izler
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    AI Silahlandirilmasi — Deepfake Vishing Senaryosu:
    1. Saldirgan, hedef sirketin CEO'sunun herkese acik konusma kayitlarindan
       (YouTube, podcast) ses modeli olusturur (3-5 dakika kayit yeterli)
    2. CFO'yu arar ve CEO'nun sesini taklit ederek acil transfer talep eder:
       "Bugun saat 15:00'e kadar 2.3M EUR'yu su hesaba gonder,
        bu gizli bir satin alma anlasmasi icin — kimseye soyleme."
    3. Caller ID spoofing ile CEO'nun telefon numarasini gosterir

    Shadow AI — Zafiyet Enjeksiyonu Senaryosu:
    1. Gelistirici, izinsiz olarak yerel ChatGPT/Claude ile API endpoint kodu yazar
    2. AI, SSRF filtresi olmayan bir link onizleme fonksiyonu uretir:
       fetch(userProvidedUrl)  // Dahili aga erisim filtresi yok!
    3. Kod, incelenmeden uretim ortamina deploy edilir
    4. Saldirgan, SSRF uzerinden ic aga erisim saglar:
       GET /preview?url=http://169.254.169.254/latest/meta-data/
    ```
*   **4. Analiz:** Deepfake vishing'de ses kalitesi artik insan kulaginin ayirt edemeyecegi seviyeye ulasmistir. Cift onay mekanizmasi yalnizca ayni kanal uzerinden dogrulama yaparsa (ornegin telefon ile onay) etkisiz kalir. Shadow AI senaryosunda gelistirici, AI uretimi kodun guvenlik boyutunu degerlendirmeden uretime alir. Tenzai arastirmasina gore test edilen tum AI araclari "link onizleme" fonksiyonu uretirken SSRF filtresini eklemeyi unutmustur.
*   **5. Kanit:** WEF 2026 raporuna gore deepfake dolandiricilik vakalari 2025'e gore %340 artmistir. SusVibes arastirmasi, AI uretimi kodun %24.7-%45 oraninda guvenlik acigi barindirdigini gostermektedir. Tenzai arastirmasi, 15 vibecoding uygulamasinda 69 kritik guvenlik acigi saptamistir — hicbiri CSP, HSTS veya SSRF filtresi icermemektedir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** Uygulanamaz (risk kategorisi, belirli bir CVE degil)
*   **Saldiri Yuzeyi:** Internete acik (AI silahlandirilmasi — phishing, deepfake) ve ic sistem (Shadow AI — kontrolsuz arac kullanimi, zafiyet enjeksiyonu)
*   **Karmasiklik:** Dusuk (AI araclari herkese acik; deepfake modeli olusturmak icin 5 dakikalik ses kaydı yeterli; Shadow AI icin calisan sadece yetkisiz araci kullanir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Kurumsal AI kullanim politikasi olusturun ve tum calisanlara duyurun. Shadow AI tespiti icin ag cikis trafikinde bilinen AI API endpointlerini (api.openai.com, api.anthropic.com, api.cohere.ai) izleyin. Finans islemlerinde cift kanal dogrulama uygulayin (telefon ile alinan talimat, farkli kanaldan — ornegin yuz yuze veya kurumsal chat — dogrulanmali). Deepfake farkindalik egitimi verin.
*   **Orta vadeli:** AI uretimi kodu CI/CD pipeline'inda zorunlu SAST (Semgrep, Snyk) taramasindan gecirin. Shadow AI tespit mekanizmasi kurun — proxy/firewall loglarinda yetkisiz AI API cagrilarini izleyin ve uyari olusturun. Deepfake ses ve goruntu analizi yapan dogrulama araclari (voice verification, liveness detection) kurumsal onay sureclerine entegre edin. UEBA (User and Entity Behavior Analytics) ile calisan AI kullanim paternlerini izleyin.
*   **Uzun vadeli:** AI tabanli savunma araclari (AI-powered SOC) ile AI hizinda saldiri tespiti kapasitesi olusturun. Kurumsal "AI kabul edilebilir kullanim" (AI AUP) politikasini zorunlu kilin ve uyumsuzluk durumlarinda otomatik engelleme uygulayin. AI egitim verilerine ve model cikislarina kurumsal veri siniflandirma politikalarini uygulayin (DLP for AI). Gelistiricilere "AI destekli muhendislik" ile "vibecoding" arasindaki farki anlatan guvenlik egitimi programlari olusturun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
# === GUVENSIZ: Shadow AI tespit mekanizmasi yok ===
# Calisanlar istedigi AI API'sine sinirsiz erisime sahip
# AI uretimi kod incelenmeden uretime aliniyor
# Deepfake dogrulama mekanizmasi yok

# Ornek: AI uretimi guvensiz link onizleme (SSRF'ye acik)
import requests

def preview_link(url: str):
    # AI tarafindan uretildi — SSRF filtresi yok!
    response = requests.get(url)  # Dahili aga erisim engellenmemis
    return response.text[:500]
```

**Guvenli kod:**
```python
# === GUVENLI: Shadow AI tespiti ve AI uretimi kod guvenligi ===
import re
import ipaddress
from urllib.parse import urlparse
import requests

# ---- 1. Shadow AI Tespit Sistemi (Proxy Log Analizi) ----
KNOWN_AI_ENDPOINTS = [
    "api.openai.com", "api.anthropic.com",
    "generativelanguage.googleapis.com",
    "api.cohere.ai", "api.mistral.ai", "api.together.xyz",
    "api.replicate.com", "api.perplexity.ai",
]

# API anahtar paternleri
AI_KEY_PATTERNS = [
    re.compile(r'sk-[a-zA-Z0-9]{48,}'),       # OpenAI
    re.compile(r'sk-ant-[a-zA-Z0-9]{20,}'),    # Anthropic
    re.compile(r'key-[a-zA-Z0-9]{32,}'),       # Cohere
]

def detect_shadow_ai(log_line: str) -> dict:
    """Proxy loglarinda yetkisiz AI API kullanimini tespit et."""
    result = {"detected": False, "endpoint": None, "has_key": False}

    for endpoint in KNOWN_AI_ENDPOINTS:
        if endpoint in log_line:
            result["detected"] = True
            result["endpoint"] = endpoint
            break

    for pattern in AI_KEY_PATTERNS:
        if pattern.search(log_line):
            result["has_key"] = True
            break

    if result["detected"]:
        import logging
        logging.warning(
            f"[SHADOW AI] Yetkisiz AI API erisimi: {result['endpoint']}"
        )
    return result

# ---- 2. SSRF Korunmali Link Onizleme (AI uretimi kod duzeltmesi) ----
BLOCKED_IP_RANGES = [
    ipaddress.ip_network('10.0.0.0/8'),
    ipaddress.ip_network('172.16.0.0/12'),
    ipaddress.ip_network('192.168.0.0/16'),
    ipaddress.ip_network('169.254.0.0/16'),  # AWS metadata
    ipaddress.ip_network('127.0.0.0/8'),
]

def is_internal_url(url: str) -> bool:
    """URL'nin dahili aga isaret edip etmedigini kontrol et."""
    parsed = urlparse(url)
    try:
        ip = ipaddress.ip_address(parsed.hostname)
        return any(ip in network for network in BLOCKED_IP_RANGES)
    except (ValueError, TypeError):
        # DNS cozumleme yapip IP'yi kontrol et
        import socket
        try:
            resolved_ip = socket.gethostbyname(parsed.hostname)
            ip = ipaddress.ip_address(resolved_ip)
            return any(ip in network for network in BLOCKED_IP_RANGES)
        except socket.gaierror:
            return True  # Cozulemezse engelle

def preview_link_safe(url: str) -> str:
    """SSRF korunmali guvenli link onizleme."""
    if is_internal_url(url):
        raise ValueError(f"Dahili ag adreslerine erisim engellendi: {url}")

    # Yalnizca HTTP/HTTPS protokolu
    parsed = urlparse(url)
    if parsed.scheme not in ('http', 'https'):
        raise ValueError(f"Desteklenmeyen protokol: {parsed.scheme}")

    response = requests.get(
        url,
        timeout=5,
        allow_redirects=False,  # Yonlendirme ile SSRF bypass onlemi
        headers={"User-Agent": "SafePreview/1.0"}
    )
    return response.text[:500]
```

```bash
# ---- 3. Ag Duzeyinde Shadow AI Tespiti (iptables/nftables) ----
#!/bin/bash
# /usr/local/bin/shadow-ai-monitor.sh

AI_ENDPOINTS=(
    "api.openai.com"
    "api.anthropic.com"
    "generativelanguage.googleapis.com"
    "api.cohere.ai"
    "api.mistral.ai"
)

for endpoint in "${AI_ENDPOINTS[@]}"; do
    # Yetkisiz AI API trafikini logla (engellemek icin DROP kullanin)
    iptables -A FORWARD -d "$endpoint" -j LOG \
        --log-prefix "SHADOW_AI: " --log-level warning
done

# Deepfake farkindalik icin finans islem dogrulama proseduru:
# 1. Telefon ile alinan transfer talepleri DAIMA farkli kanaldan dogrulanmali
# 2. Video konferans uzerinden alinan talimatlar icin "liveness check" uygulanmali
# 3. Belirli tutarin uzerindeki islemler icin yuz yuze onay zorunlu
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Kurumsal AI arac kullanimini envanterle (onaylanmis ve onaylanmamis)
*   [ ] Shadow AI politikasi olustur ve tum calisanlara duyur
*   [ ] Mevcut ag cikis trafiginde AI API endpointlerine erisimleri analiz et
*   [ ] Deepfake farkindalik egitimi ihtiyacini degerlendir
*   [ ] AI uretimi kod oranini projelerde belirle

**Duzeltme**
*   [ ] Kurumsal AI kabul edilebilir kullanim politikasi (AI AUP) tanimla ve uygula
*   [ ] Proxy/firewall'da yetkisiz AI API endpointlerini izle ve uyari olustur
*   [ ] CI/CD pipeline'ina AI uretimi kod icin zorunlu SAST taramasi ekle
*   [ ] Finans islemlerinde cift kanal dogrulama mekanizmasi uygulayin
*   [ ] Deepfake ses/goruntu dogrulama araclari degerlendirin ve pilotlayin
*   [ ] UEBA cozumu ile calisan AI kullanim paternlerini izleyin

**Dogrulama**
*   [ ] Shadow AI tespit mekanizmalarinin yetkisiz AI API kullanımını yakaladıgını test et
*   [ ] AI uretimi kodda SSRF, XSS ve eksik guvenlik basliklari (CSP, HSTS) taramasi yap
*   [ ] Deepfake vishing simulasyonu ile finans departmani direncini test et
*   [ ] Cift kanal dogrulama mekanizmasinin deepfake senaryolarini engelleyebildigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Kurumsal agdan bilinen AI API endpointlerine yapilan istekleri izle
    *   AI API anahtari paternlerini ag trafikinde tespit et (anahtar sizdirma riski)
    *   Finans islem loglarinda tek kanal dogrulama ile yapilan yuksek tutarli transferleri uyar
    *   Regex: `(?i)(api\.openai\.com|api\.anthropic\.com|api\.cohere\.ai|api\.mistral\.ai)`
    *   Regex: `(?i)(sk-[a-zA-Z0-9]{48}|sk-ant-[a-zA-Z0-9]{20}|key-[a-zA-Z0-9]{32})`
*   **Anomali Tespiti:**
    *   Kurumsal agdan AI API endpointlerine mesai saatleri disinda veya yuksek frekansta istek
    *   Gelistirici ortamlarinda onaylanmamis AI araclarina (ChatGPT, Claude web) erisim
    *   Tek bir kullanicinin kisa surede birden fazla farkli AI servisine istek gondermesi
    *   Finans departmanindan tek kanal dogrulama ile gerceklestirilen acil transfer talepleri
    *   AI uretimi commit'lerde guvenlik basliklari (CSP, HSTS, X-Frame-Options) eksikligi

---

## Notlar
WEF 2026 raporuna gore CEO'larin en buyuk korkusu AI tabanli dolandiriciliklardir. Shadow AI, kurumsal guvenlik icin en buyuk ic tehditlerden biridir — calisanlarin %60'indan fazlasinin IT bilgisi disinda AI araclari kullandigi tahmin edilmektedir. Yapay zekanin silahlandirilmasi (Industrialization of AI Attacks) ve kontrolsuz AI kullanimi birlestiginde kurumsal guvenlik hem disindan hem de icindan zayiflamaktadir. Zscaler ve Check Point 2026 raporlari, AI destekli saldiri araclarinin geleneksel araclarin 10-50 kati hizinda kesfif ve istismar gerceklestirebildigini belgelmistir.
