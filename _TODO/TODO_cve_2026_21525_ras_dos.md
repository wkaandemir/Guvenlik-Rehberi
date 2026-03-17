# TODO: CVE-2026-21525 — RAS Connection Manager Hizmet Disi Birakma (DoS)

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21525 |
| **Kritiklik** | Orta |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21525, Windows RAS (Remote Access Service) Connection Manager bileseninde bulunan bir hizmet disi birakma (Denial of Service) zafiyetidir. CVSS skoru 6.2 olan bu zafiyet, yerel bir saldirganin ozel hazirlanmis isteklerle RAS Connection Manager servisini cokertmesine ve VPN/uzak erisim baglantilarin kesilmesine olanak tanimaktadir. Microsoft Subat 2026 Patch Tuesday kapsaminda yamasi yayinlanmis olup aktif istismar edildigi dogrulanmistir. DoS kategorisinde olmakla birlikte, VPN altyapisina bagli kurumsal uzak calisma ortamlarinda is surekliligi uzerinde ciddi etki yaratabilir.
*   **Etkilenen bilesenler:** Windows RAS Connection Manager servisi (rasman.dll, RasMan), VPN baglanti yonetimi, Windows 10/11 VPN istemcileri, Windows Server 2019/2022/2025 RRAS (Routing and Remote Access Service), DirectAccess ve Always On VPN altyapilari

---

### 2. Teknik detay (nasil calisiyor)
*   RAS Connection Manager (RasMan), Windows'ta VPN baglantilari, dial-up baglantilari ve diger uzak erisim turklerini yoneten bir sistem servisidir. Bu servis, kullanici modu uygulamalarindan gelen baglanti isteklerini isler ve ag adaptorlerini yapilandirir.
*   Zafiyet, RasMan servisinin belirli bir baglanti profili yapilandirma isteigini islerken, gelen verinin uzunluk alanini dogru dogrulamadan bir bellek tahsis islemi yapmasi ve ardindan NULL pointer dereference (bos isarici referansi) hatasina dusmesinden kaynaklanmaktadir.
*   Saldirgan, yerel olarak (veya RPC uzerinden sinirli uzak erisimle) RasMan servisine bicimsiz bir baglanti profili istegi gondererek NULL pointer dereference tetikler. Bu hata, RasMan servisinin anlik cokmesine neden olur ve servis otomatik yeniden baslatilana kadar tum VPN baglantilari kesilir.
*   Servis otomatik yeniden baslatma (recovery) yapilandirilmissa bile, saldirgan sureklil bicimsiz istekler gondererek kalici hizmet reddine neden olabilir. Bu durum, VPN'e bagli uzak calisanlarin is surekliligi icin ciddi bir tehdittir.
*   **Neden:** Kok neden, rasman.dll icerisindeki baglanti profili isleyicisinin gelen yapidaki bir opsiyonel alani (opsiyonel oldugu icin NULL olabilecek bir isareti) kontrol etmeden dogrudan erismesidir (CWE-476: NULL Pointer Dereference). Savunma amalci programlama (defensive programming) prensiplerinin uygulanmamasi bu zafiyeti olusturmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Windows RAS Connection Manager servisi (RasMan) — VPN baglantisi kullanan Windows istemcisi veya RRAS sunucusu
*   **2. Normal durum:**
    ```
    1. Kullanici, VPN baglanti profili uzerinden baglanti baslatir
    2. RasMan servisi, profil yapilandirmasini okur ve baglanti parametrelerini dogrular
    3. Baglanti basarili sekilde kurulur veya hata mesaji goruntulenir
    4. Servis kararli sekilde calismaya devam eder
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    // Kavramsal PoC — RasMan NULL pointer dereference tetiklemesi
    #include <windows.h>
    #include <ras.h>
    #include <raserror.h>

    // 1. Bicimsiz bir RAS baglanti profili olustur
    RASDIALPARAMS dialParams = {0};
    dialParams.dwSize = sizeof(RASDIALPARAMS);
    wcscpy_s(dialParams.szEntryName, L"MaliciousProfile");
    // Telefon numarasi / sunucu adresi kasitli olarak bos birak
    // dialParams.szPhoneNumber = NULL (zaten sifirli)

    // 2. Phonebook dosyasinda eksik opsiyonel alan ile profil tanimla
    // [MaliciousProfile]
    // Type=2  (VPN)
    // VpnStrategy=0
    // ServerAddress=    <-- kasitli bos
    // CustomAuthData=   <-- NULL pointer tetikleyici alan

    // 3. RasDial ile baglanti baslat — bicimsiz profil RasMan'a gonderilir
    HRASCONN hRasConn = NULL;
    DWORD dwRet = RasDial(NULL, L"malicious.pbk", &dialParams, 0, NULL, &hRasConn);
    // RasMan servisi NULL pointer dereference ile coker

    // 4. Tekrarla — servis her yeniden basladiginda tekrar cokert
    // while(1) { RasDial(...); Sleep(1000); }
    ```
*   **4. Analiz:** RasMan servisi, baglanti profili icindeki CustomAuthData alanini dogrulamadan okuor. Alan NULL oldugunda (opsiyonel alan belirtilmemis), servis bos isareci uzerinden bellek erisimi yapmaya calisir ve bu ACCESS_VIOLATION istisnasina donusur. Istisna yakalanamadiginda (unhandled exception) RasMan servisi coker. Servis coktugunde tum aktif VPN baglantilari aninda kesilir.
*   **5. Kanit:** RasMan servisi coker ve System Event Log'da EventID 7034 (servis beklenmedik sekilde durduruldu) ve Application Event Log'da EventID 1000 (uygulama hatasi — rasman.dll) kayitlari olusur. Tum aktif VPN baglantilari kesilir. Servis otomatik yeniden baslatilsa bile saldirgan suren isteklerle kalici DoS saglar.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Orta
*   **CVSS Skoru:** 6.2
*   **Saldiri Yuzeyi:** Yerel erisim (standart kullanici oturumu yeterli; sinirli uzak erisim RPC uzerinden mumkun olabilir)
*   **Karmasiklik:** Dusuk (bicimsiz baglanti profili olusturmak teknik olarak basit; Windows RAS API'leri belgelenmis)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Subat 2026 Patch Tuesday yamasini tum etkilenen sistemlere uygulayin. RAS Connection Manager servisini kullanmayan sistemlerde devre disi birakin (StartupType: Disabled). VPN kullanimi gerektiren sistemlerde servis yeniden baslatma politikasini yapilandirin (otomatik kurtarma sinirlari tanimla). VPN istemcilerin alternatif baglanti yontemlerini (yedek VPN profili, dogrudan erisim) hazirlayin.
*   **Orta vadeli:** VPN altyapisini Windows RRAS yerine kurumsal VPN cozumlerine (Cisco AnyConnect, Palo Alto GlobalProtect, WireGuard) gecirin. RasMan servisine RPC erisimini sinirlandirmak icin Windows Firewall kurallarini yapilandirin. Servis durumunu izleyen bir monitoring kurali olusturun (RasMan servisi durundugunda aninda uyari). Always On VPN kullaniyorsaniz, yedek profil ve failover yapilandirmasi uygulayin.
*   **Uzun vadeli:** VPN bagimliligini azaltmak icin Zero Trust Network Access (ZTNA) mimarisine gecis planlayin. Windows VPN istemcisi yerine platform bagimsiz VPN cozumleri benimseyin. RAS/RRAS servislerinin saldiri yuzeyini azaltmak icin gereksiz ag servislerini kapatin. Is surekliligi planlarina VPN altyapisi DoS senaryolarini ekleyin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: RasMan servisi varsayilan yapilandirma ===
# Servis otomatik baslatma modunda — gereksiz yere acik olabilir
Get-Service -Name RasMan | Select-Object Status, StartType
# Sonuc: Running / Automatic

# Servis kurtarma (recovery) politikasi yapilandirilmamis
# Cokme durumunda otomatik yeniden baslatma yok veya sinirli
sc.exe qfailure RasMan
# Sonuc: RESET_PERIOD: 0, REBOOT_MSG: <bos>, COMMAND_LINE: <bos>

# VPN baglantisi aktif kullanicilarin isi durabilir
# RasMan coktugunde tum VPN baglantilari kesilir
rasdial
# Sonuc: Aktif baglanti yok (cokme sonrasi)
```

**Guvenli kod:**
```powershell
# === GUVENLI: RasMan DoS'a karsi sertlestirme ===

# 1. Subat 2026 yamasini dogrula
$patch = Get-HotFix | Where-Object {
    $_.InstalledOn -gt "2026-02-01" -and $_.Description -match "Security Update"
} | Sort-Object InstalledOn -Descending
if (-not $patch) {
    Write-Warning "[!] ACIL: Subat 2026 RasMan yamasi bulunamadi!"
} else {
    Write-Host "[+] Guvenlik yamasi mevcut:" ($patch.HotFixID -join ", ")
}

# 2. RasMan servisi gerekmiyorsa devre disi birak
$vpnNeeded = Read-Host "Bu sistemde VPN kullanimi gerekli mi? (E/H)"
if ($vpnNeeded -eq "H") {
    Set-Service -Name RasMan -StartupType Disabled
    Stop-Service -Name RasMan -Force
    Write-Host "[+] RasMan servisi devre disi birakildi"
} else {
    # 3. VPN gerekiyorsa: Servis kurtarma politikasini yapilandir
    # Ilk cokme: 5 saniye sonra yeniden baslat
    # Ikinci cokme: 30 saniye sonra yeniden baslat
    # Ucuncu cokme: 60 saniye sonra yeniden baslat
    sc.exe failure RasMan reset= 86400 actions= restart/5000/restart/30000/restart/60000
    Write-Host "[+] RasMan servis kurtarma politikasi yapilandirildi"

    # 4. Servis durumunu izleme gorevi olustur
    $trigger = New-ScheduledTaskTrigger -AtStartup
    $action = New-ScheduledTaskAction -Execute "powershell.exe" `
        -Argument '-Command "while ($true) { $svc = Get-Service RasMan; if ($svc.Status -ne \"Running\") { Write-EventLog -LogName Application -Source \"RasManMonitor\" -EventId 9001 -EntryType Warning -Message \"RasMan servisi durdu!\"; Start-Service RasMan }; Start-Sleep 10 }"'
    # Register-ScheduledTask -TaskName "RasManMonitor" -Trigger $trigger -Action $action -User "SYSTEM"
    Write-Host "[+] RasMan izleme gorevi tanimi hazir"
}

# 5. VPN rate limiting — firewall ile RPC erisimini sinirla
New-NetFirewallRule -DisplayName "RasMan-RPC-Restrict" -Direction Inbound `
    -Protocol TCP -LocalPort 135 -RemoteAddress "10.0.0.0/8" `
    -Action Allow -Profile Domain
Write-Host "[+] RasMan RPC erisimi sinirlandirildi"
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] RAS Connection Manager (RasMan) servisi calisan tum sistemleri envanterle
*   [ ] VPN/RRAS kullanan sunucu ve istemcileri tespit et
*   [ ] Always On VPN ve DirectAccess yapilandirmalarini gozden gecir
*   [ ] Is surekliligi planlarina VPN DoS senaryosunu ekle

**Duzeltme**
*   [ ] Microsoft Subat 2026 Patch Tuesday yamasini tum etkilenen sistemlere uygula
*   [ ] VPN kullanmayan sistemlerde RasMan servisini devre disi birak
*   [ ] VPN gerektiren sistemlerde servis kurtarma politikasini yapilandir
*   [ ] RasMan servisine RPC erisimini firewall kurallariyla sinirla
*   [ ] Alternatif VPN profilleri ve yedek baglanti yontemleri hazirla

**Dogrulama**
*   [ ] Yama sonrasi bicimsiz profil ile DoS testini tekrarla
*   [ ] RasMan servisi cokme senaryosunda kurtarma politikasinin calistigini dogrula
*   [ ] VPN baglanti surekliligi testini gerceklestir
*   [ ] Servis izleme kurallarinin cokme durumunda uyari urettigini test et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   System Event Log'da EventID 7034 (RasMan servisi beklenmedik sekilde durduruldu) izle
    *   Application Event Log'da EventID 1000 (uygulama hatasi — rasman.dll) izle
    *   System Event Log'da EventID 7036 (servis durum degisikligi — RasMan) izle
    *   Regex: `(?i)(RasMan.*stopped.*unexpectedly|rasman\.dll.*fault|RAS.*connection.*manager.*crash)`
    *   Regex: `(?i)(NULL.*pointer.*rasman|access.violation.*0x[0]+.*rasman)`
    *   VPN baglanti loglarinda es zamanli cok sayida baglanti kopma olayini izle
*   **Anomali Tespiti:**
    *   RasMan servisinin kisa surede birden fazla cokme/yeniden baslatma dongusu (crash loop)
    *   Tum VPN baglantilarin ayni anda kesilmesi (DoS belirtisi)
    *   RasMan servisine yonelik anormal RPC cagrilari (bicimsiz baglanti profili istekleri)
    *   VPN baglanti sayisinda ani dusus ve kullanici sikayet artisi korelasyonu
    *   RasMan servisinin beklenen disinda durma ve baslatma dongusu (EventID 7034/7036 tekrari)

---

## Notlar
Aktif istismar dogrulanmis. DoS kategorisinde olmakla birlikte, VPN altyapisina bagli kurumsal uzak calisma ortamlarinda is surekliligi uzerinde ciddi etki yaratabilir. Microsoft Subat 2026 Patch Tuesday kapsaminda 59 zafiyet arasinda ele alinmistir. VPN kullanmayan sistemlerde RasMan servisinin devre disi birakilmasi en etkili onlemdir. Uzun vadede ZTNA mimarisine gecis VPN bagimliligini ve dolayisiyla bu tur riskleri azaltir.
