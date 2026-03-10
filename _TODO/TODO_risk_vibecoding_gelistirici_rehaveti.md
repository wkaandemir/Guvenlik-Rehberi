# TODO: Vibecoding Gelistirici Rehaveti (Raph Wiggum Dongusu)

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | Vibecoding Gelistirici Rehaveti (Raph Wiggum Dongusu) |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | YENI: AI_IDE_Guvenligi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Subat_2026.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Subat_2026.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Subat 2026'da StackOverflow ve Reddit uzerindeki tartismalarda "Raph Wiggum Dongusu" olarak adlandirilan bir fenomen yogun bicimde ele alinmistir: gelistiricilerin yapay zeka ciktisini hic okumadan dogrudan ana kod tabanina dahil etmesi. Wikipedia ve bagimsiz arastirmalara gore AI destekli kodlama, kod kopyalamayi (code duplication) 4 kat artirmis ve refactoring (yeniden duzenleme) islemlerini %25'ten %10'un altina dusurmmustur. Bu rehavet, AI modellerinin sentaks olarak dogru ancak mantiksal olarak hatali veya guvenlik acisindan kusurlu kod uretme egilimiyle (confident garbage) birlestirildiginde ciddi sonuclar dogmaktadir. Vibecoding sirasinda AI, "kodun calismasi niyetine" odaklanirken gelisitiricinin ekleedigi guvenlik kontrolu anahtar kelimelerini (auth, check_permission, csrf_protect) yanlislikla silebilmekte ve bu durum insan gozuyle fark edilememektedir. SusVibes arastirmasi, AI uretimi fonksiyonel olarak dogru cozumlerin yalnizca %10.5'inin guvenli oldugunu kanitlamistir. Kisa surede uretkenlik artisi gibi gorunen bu yaklasim, orta ve uzun vadede yonetilmesi cok daha maliyetli bir "guvenlik borcu" biriktirmektedir.
*   **Etkilenen bilesenler:** Vibecoding kullanan tum gelistirme ekipleri ve projeleri, AI destekli kod inceleme surecleri, uretim ortamina alinan AI uretimi kod, CI/CD pipeline'lari (SAST kapisi eksikligi), kod tabanlari (artan kod kopyalama ve azalan refactoring), kimlik dogrulama (auth) ve yetkilendirme mekanizmalari (AI tarafindan silinme riski)

---

### 2. Teknik detay (nasil calisiyor)
*   **Raph Wiggum Dongusu:** Gelistirici, AI aracina bir gorev verir. AI, kod uretir. Gelistirici, kodu okumadan veya yuzeysel bir bakisla kabul eder ve dogrudan ana dala (main branch) dahil eder. Bu dongu, AI uretimi kodun kalite ve guvenlik denetiminden gecmeden uretime alinmasina neden olur. Isim, Simpsons karakteri Ralph Wiggum'un "I'm helping!" repliginden gelmektedir — gelistirici kendini uretken sanirken aslinda guvenlik borcu biriktirmektedir.
*   **Kod Kopyalama Artisi:** AI destekli kodlama, benzer islevleri farkli yerlerde yeniden uretme egilimindedir (DRY — Don't Repeat Yourself prensibinin ihlali). Bu durum bakiim zorlugunu arttirir ve bir zafiyetin duzeltilmesi gereken noktaarin sayisini cattirir.
*   **Refactoring Dususu:** AI araclariyla calisan gelistiriciler, mevcut kodu iyilestirmek yerine yeni kod uretmeyi tercih etmektedir. Refactoring oraninin %25'ten %10'un altina dusmesi, teknik borcun birikmesini hizlandirmaktadir.
*   **Confident Garbage:** AI modelleri, sentaks olarak dogru ancak mantiksal olarak hatali veya asiri verimsiz kod uretme egilimindedir. Ozellikle veritabani sorgularinda AI'nin urettigi kodun test ortaminda calisip uretim ortaminda (buyuk veri setlerinde) sistem kilitlenmelerine yol actigi raporlanmistir.
*   **Guvenlik Duvarlarinin Silinmesi:** Vibecoding sirasinda AI, gorev tamamlama odakli calisirken mevcut koddaki guvenlik kontrollerini (auth middleware, CSRF token kontrolu, girdi sanitasyonu) "gereksiz" olarak degerlendirip yanlislikla silebilir. Bu degisiklik buyuk bir diff icerisinde kolayca gozden kacabilir.
*   **Neden:** Kok neden, AI araclarinin gelistiricilerde yanlis bir guven duygusu yaratmasi (otomasyon yanliligi — automation bias), kod inceleme surecleirinin AI uretimi kodun hacmine yetisememesi, ve "hiz" ile "kolaylik" odakli vibecoding kulturunun guvenlik denetimlerini ihmal etmesidir. OWASP ASI09 (Human-Agent Trust Exploitation) kategorisiyle dogrudan iliskilidir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Vibecoding ile gelistirilen bir web uygulamasinin kimlik dogrulama katmani.
*   **2. Normal durum:**
    ```
    1. Gelistirici, mevcut auth middleware'ini korur
    2. Her yeni endpoint, auth kontrolunden gecmeden eklenmez
    3. Kod incelemesinde guvenlik kontrolleirinin varligi dogrulanir
    4. SAST araclari CI/CD'de zorunlu olarak calisir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```typescript
    // Gelistirici AI'ya "kullanici profil sayfasi ekle" der
    // AI, mevcut auth middleware'ini kaldirarak yeni endpoint uretir:

    // --- ONCE (guvenli) ---
    // app.get('/profile', authMiddleware, csrfProtect, (req, res) => {
    //   const user = req.user;  // auth ile dogrulanmis
    //   res.render('profile', { user });
    // });

    // --- SONRA (AI uretimi — guvenlik kontrolleri silinmis) ---
    app.get('/profile', (req, res) => {
      // AI, authMiddleware ve csrfProtect'i "gereksiz" olarak kaldirmis!
      const userId = req.query.id;  // Girdi dogrulamasi yok
      const user = db.query(
        `SELECT * FROM users WHERE id = '${userId}'`  // SQL injection!
      );
      res.render('profile', { user });
      // CSP, X-Frame-Options basliklari yok
    });

    // Gelistirici, diff'i okumadan "Looks good, merge" der (Raph Wiggum)
    // Sonuc: Auth bypass + SQL injection + Clickjacking
    ```
*   **4. Analiz:** AI, gorev tamamlama (profil sayfasi ekleme) odakli calisirken mevcut guvenlik kontrollerini (authMiddleware, csrfProtect) "gereksiz bagimlilik" olarak degerlendirip kaldirmistir. Gelistirici, buyuk bir diff icerisinde bu degisikligi fark etmemis ve kodu dogrudan merge etmistir. Sonuc, kimlik dogrulama bypass'i, SQL injection ve clickjacking zafiyetlerinin uretim ortamina alinmasidir. SusVibes verileirine gore bu tip senaryo AI uretimi kodun %89.5'inde gerceklesebilir (yalnizca %10.5'i guvenli).
*   **5. Kanit:** Auth middleware kaldirildiktan sonra /profile endpoint'ine kimlik dogrulamasi olmadan erisim mumkun hale gelir. SQL injection ile tum kullanici verileri cikarilabilir. Bu tip zafiyetler, Tenzai arastirmasinda 15 vibecoding uygulamasinin tamaminda tespit edilmistir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** Uygulanamaz (insani/kulturel risk faktoru, belirli bir CVE degil; ancak sonuclari auth bypass, SQL injection gibi CVSS 8.0-9.8 zafiyetlere yol acar)
*   **Saldiri Yuzeyi:** Internete acik (AI uretimi web uygulamalari dogrudan erisime acilir)
*   **Karmasiklik:** Dusuk (saldirgan perspektifinden — AI uretimi zafiyetler genellikle temel guvenlik kontrollerinin eksikligi nedeniyle kolayca istismar edilir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Git pre-commit hook ile guvenlik kontrollerinin (auth, check_permission, csrf, sanitize, Content-Security-Policy) kaldirilmasini tespit eden otomatik kontrol mekanizmasi ekleyin. CI/CD pipeline'inda zorunlu SAST taramasi uygulayin. Kod inceleme surecinde AI uretimi kodu ozel isaretleyen (tagging) mekanizma kurun. "Her AI ciktisi commit edilmeden once en az bir insan tarafindan okunmalidir" kuralini zorunlu kilin.
*   **Orta vadeli:** "Mega prompt" (tek bir devasa komutla uygulama yapma) yerine "atomik gorevler" stratejisine gecin — her AI gorevini kucuk, test edilebilir parcalara bolerek sonuclari bireysel olarak dogrulayin. AI uretimi kodu otomatik olarak isaretleyen ve zorunlu guvenlik incelemesine alan tooling gelistirin. Kod kopyalama (duplication) analizi ile refactoring ihtiyacini surekli izleyin. Gelistiricilere AI guvenlik farkindalik egitimi verin — "confident garbage" ornekleri ve gercek dunyadan vaka analizi.
*   **Uzun vadeli:** Gelistirme kulturunde "vibecoding" ile "AI destekli muhendislik" arasindaki farki kurumsallastirin. AI uretimi kodun guvenlik kalite puanini (security quality score) olcen ve raporlayan araclar gelistirin. Kod tabani saglik metrikleri (duplication rate, refactoring rate, auth coverage) icin surekli izleme dashboard'u kurun. OWASP ASI09 standardina uyumlu gelistirici egitim programlarini periyodik olarak tekrarlayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```bash
# === GUVENSIZ: AI uretimi kodu kontrolsuz kabul etme (Raph Wiggum) ===
# Kod inceleme sureci yok veya yetersiz
# AI ciktisi dogrudan merge ediliyor
# Guvenlik kontrolleri silinmis olabilir — kimse kontrol etmiyor

# Pre-commit hook yok:
# .git/hooks/pre-commit dosyasi bos veya mevcut degil

# CI/CD'de SAST kapisi yok:
# Pipeline dogrudan deploy ediyor
```

**Guvenli kod:**
```bash
#!/bin/bash
# === GUVENLI: Git pre-commit hook — AI uretimi kodda guvenlik kontrolu ===
# .git/hooks/pre-commit

STAGED=$(git diff --cached --name-only)
EXIT_CODE=0

echo "=== AI Kod Guvenlik Kontrolu ==="

for file in $STAGED; do
    # 1. Auth/guvenlik kontrolu silme tespiti
    if git diff --cached "$file" | grep -E "^\-.*authMiddleware|^\-.*check_permission|^\-.*csrf|^\-.*sanitize|^\-.*authenticate"; then
        echo "[KRITIK] $file: Guvenlik kontrolu kaldiriliyor!"
        echo "  Lutfen 'git diff --cached $file' ile inceleyin."
        EXIT_CODE=1
    fi

    # 2. Guvenlik basliklari kontrolu
    if git diff --cached "$file" | grep -E "^\-.*Content-Security-Policy|^\-.*X-Frame-Options|^\-.*Strict-Transport-Security"; then
        echo "[KRITIK] $file: Guvenlik basligi kaldiriliyor!"
        EXIT_CODE=1
    fi

    # 3. Guvensiz kalip tespiti (yeni eklenen satirlarda)
    if git diff --cached "$file" | grep -E "^\+.*shell=True|^\+.*eval\(|^\+.*exec\(|^\+.*innerHTML\s*="; then
        echo "[UYARI] $file: Potansiyel guvensiz kalip tespit edildi."
        EXIT_CODE=1
    fi

    # 4. SQL injection kalip tespiti
    if git diff --cached "$file" | grep -E '^\+.*f".*SELECT|^\+.*f".*INSERT|^\+.*f".*DELETE|^\+.*\$\{.*\}.*WHERE'; then
        echo "[UYARI] $file: Potansiyel SQL injection — parametreli sorgu kullanin."
        EXIT_CODE=1
    fi
done

# 5. Kod kopyalama kontrolu (basit esik)
TOTAL_ADDED=$(git diff --cached --stat | tail -1 | awk '{print $4}')
if [ "${TOTAL_ADDED:-0}" -gt 500 ]; then
    echo "[BILGI] Buyuk commit (${TOTAL_ADDED}+ satir eklenmis)."
    echo "  AI uretimi kod olabilir — kapsamli inceleme onerilir."
fi

if [ $EXIT_CODE -ne 0 ]; then
    echo ""
    echo "Commit engellendi. Guvenlik sorunlarini duzeltin veya"
    echo "kasitli ise 'git commit --no-verify' kullanin (onerilmez)."
fi
exit $EXIT_CODE
```

```yaml
# === CI/CD: AI uretimi kod icin zorunlu guvenlik kapisi ===
# .github/workflows/security-review.yml
name: AI Code Security Review
on: [pull_request]
jobs:
  security-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Tam gecmis — diff analizi icin

      - name: Auth Kontrolu Silme Tespiti
        run: |
          # PR'daki degisikliklerde guvenlik kontrolu silinmis mi?
          REMOVED_AUTH=$(git diff origin/main...HEAD | grep -c "^\-.*auth\|^\-.*permission\|^\-.*csrf" || true)
          if [ "$REMOVED_AUTH" -gt 0 ]; then
            echo "::error::$REMOVED_AUTH guvenlik kontrolu kaldirildigi tespit edildi!"
            echo "Lutfen PR'i dikkatlice inceleyin."
            exit 1
          fi

      - name: Semgrep SAST Taramasi
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/owasp-top-ten

      - name: Kod Kopyalama Analizi
        run: |
          # jscpd ile kod kopyalama oranini kontrol et
          npx jscpd --threshold 15 --reporters console src/
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Ekibin AI destekli gelistirme pratiklerini degerlendir (vibecoding vs. AI destekli muhendislik)
*   [ ] Mevcut kod inceleme sureclerinin AI uretimi kodun hacmine yetip yetmedigini analiz et
*   [ ] Kod kopyalama oranini ve refactoring metriklerini olc
*   [ ] Pre-commit hook ve CI/CD guvenlik kapisi varligini kontrol et

**Duzeltme**
*   [ ] Git pre-commit hook ile guvenlik kontrol silme tespiti mekanizmasi ekle
*   [ ] CI/CD pipeline'inda zorunlu SAST (Semgrep/Snyk) kapisi uygulayin
*   [ ] Kod inceleme surecinde AI uretimi kodu isaretleyen (tagging) mekanizma kurun
*   [ ] "Her AI ciktisi commit edilmeden once en az bir insan tarafindan okunmalidir" kuralini uygulayin
*   [ ] "Mega prompt" yerine "atomik gorevler" stratejisine gecis icin ekip egitimi verin
*   [ ] Zorunlu AI guvenlik farkindalik egitimi duzenleyin

**Dogrulama**
*   [ ] Pre-commit hook'un guvenlik kontrolu kaldirma girisimlerini tespit ettigini test et
*   [ ] CI/CD kapisinin guvensiz commit'leri engelledigini dogrula
*   [ ] Kod kopyalama oraninin kabul edilebilir esik degerinin altinda oldugunu kontrol et
*   [ ] Ekibin AI uretimi kodu inceleme aliskanligini periyodik olarak degerlendir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   CI/CD pipeline'inda SAST taramasindan gecemeyen AI uretimi commit'leri izle
    *   Pre-commit hook tarafindan engellenen commit sayisini ve nedenlerini raporla
    *   Uretim ortaminda guvenlik basliklari eksik olan sayfalari tespit et
    *   Regex: `(?i)(removed.*auth|removed.*csrf|removed.*permission|deleted.*middleware)`
    *   Regex: `(?i)(pre.commit.*blocked|security.*gate.*failed|sast.*violation)`
*   **Anomali Tespiti:**
    *   Tek bir gelistiricinin kisa surede cok sayida buyuk commit yapmasi (AI tabanli hizli uretim belirtisi)
    *   Kod kopyalama oraninda ani artis (AI uretimi tekrarlayan kod belirtisi)
    *   Refactoring commit oraninin anlamli olcude dusmesi
    *   Auth middleware veya guvenlik kontrollerinin PR'larda siklikla kaldirilmasi
    *   SAST taramasinda yuksek sayida uyari alan commit'lerin artmasi
    *   `--no-verify` flag'i ile yapilan commit sayisinda artis (hook atlama)

---

## Notlar
"Raph Wiggum Dongusu" — vibecoding dunyasinin en yaygin ve sinsi guvenlik riskidir. AI destekli kodlama kod kopyalamayi 4x artirmis, refactoring'i %25'ten %10'un altina dusurmustur. "Confident garbage" fenomeni — sentaks olarak dogru ancak mantiksal olarak hatali veya guvensiz kod — ozellikle veritabani sorgulari ve oturum yonetiminde kritik sorunlara yol acmaktadir. SusVibes arastirmasi, "guvenli kod yaz" talimatinin paradoksal olarak fonksiyonel dogrulugu %6 dusurdugunu gostermistir. OWASP ASI09 (Human-Agent Trust Exploitation) ile dogrudan iliskilidir. Cozum, "mega prompt" yerine "atomik gorevler" ve zorunlu insan incelemesi kulturudur.
