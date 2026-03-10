# TODO: CVE-2026-21533 — Remote Desktop Services Yetki Yukseltme

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21533 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Vibecoding Guvenlik Aciklari Arastirmasi (Ham Notlar).md](../Arastirmalar/Vibecoding%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1%20(Ham%20Notlar).md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21533, Windows Remote Desktop Services (RDS) bileseninde bulunan bir yetki yukseltme (Elevation of Privilege) zafiyetidir. CVSS skoru 7.8 olan bu zafiyet, Remote Desktop oturumu uzerinden erisim saglayan dusuk yetkili bir saldirganin, RDS oturum yonetimi mekanizmasindaki bir hatadan yararlanarak SYSTEM yetkilerine erismesine olanak tanimaktadir. Microsoft Subat 2026 Patch Tuesday kapsaminda yamasi yayinlanmis olup, CISA tarafindan aktif istismar edildigi dogrulanmistir. Kurumsal ortamlarda RDS/Terminal Services yaygin olarak kullanildigi icin genis bir etki alanina sahiptir ve ozellikle cok kullanicili sunucu ortamlarinda kritik risk olusturmaktadir.
*   **Etkilenen bilesenler:** Remote Desktop Services (termsrv.dll), RDS Session Broker, RDP oturum yonetimi altyapisi, Windows Server 2019/2022/2025 RDS rolleri, Windows 10/11 uzak masaustu istemcileri, RDS Gateway ve RDS Web Access bilesenleri

---

### 2. Teknik detay (nasil calisiyor)
*   Remote Desktop Services (RDS), birden fazla kullanicinin ayni sunucuya es zamanli olarak uzak masaustu oturumu acmasini saglayan Windows Server altyapisidir. Her kullanici oturumu izole bir oturum ortaminda (session) calisir ve oturum yonetimi SYSTEM yetkileriyle calisan termsrv.dll tarafindan gerceklestirilir.
*   Zafiyet, RDS oturum yonetim mekanizmasindaki bir named pipe (adlandirilmis kanal) isleyicisinde bulunmaktadir. Oturum degistirme (session switching) islemi sirasinda, termsrv.dll belirli bir named pipe uzerinden oturum kimlik bilgilerini transfer ederken erisim kontrolu (ACL) dogrulamasini yetersiz sekilde yapmaktadir.
*   Saldirgan, dusuk yetkili bir RDP oturumundan, oturum yonetimi named pipe'ina ozel hazirlanmis bir istek gondererek SYSTEM baglaminda calisan oturum yonetim islemini manipule eder. Named pipe impersonation (kimlik taklidi) teknigi ile saldirgan, SYSTEM token'ini elde eder.
*   Cok kullanicili RDS ortamlarinda bu zafiyet ozellikle tehlikelidir: bir kullanici, diger kullanicilarin oturum verilerine erisebilir, sunucu uzerinde tam yetki kazanabilir ve lateral movement icin altyapi olusturabilir.
*   **Neden:** Kok neden, termsrv.dll'deki oturum degistirme named pipe'inin istemci kimligini dogrulamak icin ImpersonateNamedPipeClient kullanirken, impersonation seviyesini (SecurityImpersonation vs SecurityIdentification) kontrol etmemesidir. Bu eksiklik, dusuk yetkili bir istemcinin SYSTEM baglaminda islem yapmasina olanak tanir (CWE-269: Improper Privilege Management).

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Remote Desktop Services sunucusu — dusuk yetkili bir RDP oturumundan SYSTEM yetkilerine yukseltme
*   **2. Normal durum:**
    ```
    1. Kullanici, RDP istemcisi ile sunucuya baglanir ve oturum acar
    2. termsrv.dll, kullanici oturumunu izole bir Session ortaminda baslatir
    3. Oturum degistirme istekleri named pipe uzerinden SYSTEM seviyesinde islenir
    4. Named pipe ACL'si yalnizca yetkili sistem sureclerin erisimini saglar
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    // Kavramsal PoC — RDS named pipe impersonation ile EoP
    #include <windows.h>

    // 1. RDS oturum yonetim named pipe'ini tespit et
    // pipe adi: \\.\pipe\TermService_Session_<SessionID>
    HANDLE hPipe;
    WCHAR pipeName[256];
    DWORD sessionId;
    ProcessIdToSessionId(GetCurrentProcessId(), &sessionId);
    swprintf_s(pipeName, L"\\\\.\\pipe\\TermService_Session_%d", sessionId);

    // 2. Named pipe'a baglan ve oturum degistirme istegi gonder
    hPipe = CreateFile(pipeName, GENERIC_READ | GENERIC_WRITE,
        0, NULL, OPEN_EXISTING, 0, NULL);

    // 3. Oturum bilgisi istegi — bicimsiz yapida impersonation tetikleyici
    BYTE request[128] = {0};
    request[0] = 0x03;  // SESSION_SWITCH komut kodu
    request[4] = 0xFF;  // Hedef oturum: SYSTEM (Session 0)
    DWORD written;
    WriteFile(hPipe, request, sizeof(request), &written, NULL);

    // 4. termsrv.dll, istegi SYSTEM baglaminda isler
    // Named pipe impersonation ile SYSTEM token'i elde edilir
    // ImpersonateNamedPipeClient basarili — SecurityImpersonation seviyesinde

    // 5. SYSTEM token'i ile yeni surec baslat
    HANDLE hToken;
    // OpenThreadToken(GetCurrentThread(), TOKEN_ALL_ACCESS, FALSE, &hToken);
    // CreateProcessWithTokenW(hToken, 0, L"cmd.exe", ...);
    // SYSTEM yetkisiyle cmd.exe acilir
    ```
*   **4. Analiz:** termsrv.dll, named pipe uzerinden gelen oturum degistirme isteigini islerken ImpersonateNamedPipeClient fonksiyonunu cagirir. Ancak impersonation sonrasi istemcinin SecurityImpersonation seviyesinde olup olmadigini dogrulayamaz. Saldirganin named pipe istemcisi, SYSTEM baglaminda calisan termsrv.dll'nin token'ini devralir. Bu, klasik bir named pipe impersonation yetki yukseltme teknigidir.
*   **5. Kanit:** Basarili istismar sonrasinda saldirgan SYSTEM yetkisiyle cmd.exe veya powershell.exe acar. Security Event Log'da EventID 4672 (ozel yetki atamasi) dusuk yetkili RDP kullanicisi icin olusur. EventID 4624 (basarili oturum acma) Type 10 (RemoteInteractive) ile baslaayan ve ardindan SYSTEM erisimi gorulen kayitlar tespit edilebilir. RDS oturum loglarinda anormal oturum degistirme istekleri gorunur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.8
*   **Saldiri Yuzeyi:** Ag tabanli (RDP oturumu uzerinden; RDP 3389/TCP portuna erisim ve gecerli kullanici kimlik bilgileri gerekir)
*   **Karmasiklik:** Orta (named pipe impersonation teknigi iyi belgelenmis; ancak dogru named pipe'i ve zamanlama penceresini bulmak teknik bilgi gerektirir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Subat 2026 Patch Tuesday yamasini tum RDS sunucularina oncelikli uygulayin. Network Level Authentication (NLA) zorunlulugunu etkinlestirin. RDP erisimini yalnizca yetkili kullanici gruplarina kisitlayin ve varsayilan porttu (3389) degistirin. Gereksinime gore RDS oturumlarini sinirli sayida tutun.
*   **Orta vadeli:** RDP erisimini dogrudan internet erisimi yerine VPN/RDS Gateway uzerinden sinirlandirin. RDS sunuculari icin kullanici oturum izolasyonunu sertlestirin (Session isolation level). Named pipe erisim kontrollerini denetleyin. Multi-Factor Authentication (MFA) zorunlulugunu tum RDP oturumlari icin uygulayin. PAW (Privileged Access Workstation) mimarisi ile admin oturumlarini izole edin.
*   **Uzun vadeli:** RDS/Terminal Services yerine modern uzak erisim cozumlerine (Azure Virtual Desktop, Citrix DaaS) gecis planlayin. Zero Trust Network Access (ZTNA) ile RDP port aciklarini ortadan kaldirin. RDS sunuculari icin Credential Guard ve HVCI korumasini etkinlestirin. Oturum tabanli mikro-segmentasyon ile kullanici oturumlari arasinda ag izolasyonu saglayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: RDS varsayilan yapilandirma — named pipe istismarina acik ===
# NLA etkin degil — kimlik dogrulama oturum acildiktan SONRA yapilir
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
    -Name "UserAuthentication"
# Sonuc: 0 (NLA devre disi)

# RDP portu varsayilan (3389) — tarama kolayligi
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
    -Name "PortNumber"
# Sonuc: 3389

# RDP erisimi kisitlanmamis — tum kullanicilar baglanabiliyor
Get-LocalGroupMember -Group "Remote Desktop Users"
# Sonuc: Genis kullanici listesi veya Everyone grubu
```

**Guvenli kod:**
```powershell
# === GUVENLI: RDS EoP'ye karsi sertlestirme ===

# 1. Subat 2026 Patch Tuesday yamasini dogrula
$patch = Get-HotFix | Where-Object {
    $_.InstalledOn -gt "2026-02-01" -and $_.Description -match "Security Update"
}
if (-not $patch) {
    Write-Warning "[!] ACIL: Subat 2026 RDS yamasi bulunamadi!"
} else {
    Write-Host "[+] Guvenlik yamasi mevcut:" ($patch.HotFixID -join ", ")
}

# 2. Network Level Authentication (NLA) zorla
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
    -Name "UserAuthentication" -Value 1 -Type DWord
Write-Host "[+] NLA etkinlestirildi"

# 3. RDP portunu degistir (varsayilan 3389'dan farkli)
$newPort = 3390
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
    -Name "PortNumber" -Value $newPort -Type DWord
# Firewall kuralini guncelle
New-NetFirewallRule -DisplayName "RDP-Custom-Port" -Direction Inbound `
    -LocalPort $newPort -Protocol TCP -Action Allow -Profile Domain
Write-Host "[+] RDP portu $newPort olarak degistirildi"

# 4. RDP erisimini yalnizca yetkili gruba kisitla
$allowedGroup = "RDP-Yetkili-Kullanicilar"
# Mevcut uyeleri temizle ve yalnizca yetkili grubu ekle
Get-LocalGroupMember -Group "Remote Desktop Users" | ForEach-Object {
    Remove-LocalGroupMember -Group "Remote Desktop Users" -Member $_.Name -ErrorAction SilentlyContinue
}
Add-LocalGroupMember -Group "Remote Desktop Users" -Member $allowedGroup
Write-Host "[+] RDP erisimi '$allowedGroup' grubuna sinirlandirildi"

# 5. Oturum sinirlamasi ve zaman asimi yapilandir
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" `
    -Name "MaxConnectionTime" -Value 28800000 -Type DWord  # 8 saat
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" `
    -Name "MaxIdleTime" -Value 1800000 -Type DWord  # 30 dakika
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" `
    -Name "MaxDisconnectionTime" -Value 3600000 -Type DWord  # 1 saat
Write-Host "[+] RDS oturum zaman asimlari yapilandirildi"

# 6. RDP oturumlarini izle
$rdpEvents = Get-WinEvent -LogName "Microsoft-Windows-TerminalServices-LocalSessionManager/Operational" `
    -MaxEvents 20 -ErrorAction SilentlyContinue
$rdpEvents | Format-Table TimeCreated, @{N='Message';E={$_.Message.Substring(0,120)}} -Wrap
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] RDS/Terminal Services rolleri etkin tum sunuculari envanterle
*   [ ] Mevcut NLA, port ve erisim kontrolu yapilandirmalarini denetle
*   [ ] RDP kullanan tum kullanici ve servis hesaplarini listele
*   [ ] RDS Gateway ve Web Access bilesenlerin durumunu kontrol et

**Duzeltme**
*   [ ] Microsoft Subat 2026 Patch Tuesday yamasini tum RDS sunucularina uygula
*   [ ] NLA zorunlulugunu etkinlestir
*   [ ] RDP erisimini yalnizca yetkili kullanici grubuna kisitla
*   [ ] RDP portunu varsayilandan degistir ve firewall kurallarini guncelle
*   [ ] Oturum zaman asimi ve sinirlamalarini yapilandir
*   [ ] MFA zorunlulugunu tum RDP oturumlari icin uygula

**Dogrulama**
*   [ ] Yama sonrasi named pipe impersonation EoP testini tekrarla
*   [ ] NLA'nin dogru calistigini ve kimlik dogrulama oncesi oturum acilmadigini dogrula
*   [ ] RDP erisim kisitlamalarinin beklenildigi gibi calistigini test et
*   [ ] SIEM kurallarinin yetki yukseltme girisimlerini tespit ettigini dogrula
*   [ ] Oturum izolasyonunun farkli kullanicilar arasinda etkili oldugunu dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Security Event Log'da EventID 4672 (ozel yetki atamasi) — RDP oturumu baslatmis kullanicilar icin izle
    *   Security Event Log'da EventID 4624 Type 10 (RemoteInteractive) sonrasi SYSTEM erisimi gorulmesi
    *   Microsoft-Windows-TerminalServices-LocalSessionManager/Operational loglarinda oturum degistirme olaylarini izle
    *   Regex: `(?i)(termsrv.*impersonat|named.pipe.*TermService.*Session|RDP.*privilege.*escalat)`
    *   Regex: `(?i)(4672.*RemoteInteractive|session.*switch.*SYSTEM|pipe.*impersonation.*token)`
    *   Sysmon EventID 17/18 (Named Pipe Created/Connected) — TermService ile iliskili pipe olaylarini izle
*   **Anomali Tespiti:**
    *   Dusuk yetkili RDP kullanicisinin SYSTEM yetkilerine gecis yapmasi (EventID 4672 + 4624 korelasyonu)
    *   RDS oturumlari arasinda beklenmeyen oturum degistirme istekleri
    *   TermService named pipe'ina standart disi sureclerin baglanmasi
    *   Tek bir RDP oturumundan kisa surede birden fazla named pipe baglantisi (brute-force belirtisi)
    *   RDS sunucusu uzerinde SYSTEM yetkisiyle calisan beklenmeyen sureclerin (cmd.exe, powershell.exe) olusturumlmasi

---

## Notlar
Aktif istismar dogrulanmis. Microsoft Subat 2026 Patch Tuesday kapsaminda 59 zafiyet arasinda ele alinmistir. Kurumsal ortamlarda RDS/Terminal Services yaygin olarak kullanildigi icin genis etki alanina sahiptir. Cok kullanicili sunucu ortamlarinda bir kullanicinin diger kullanicilarin oturum verilerine erismesi ve tam sunucu kontrolu elde etmesi mumkundur. Named pipe impersonation, Windows yetki yukseltme saldirilarinda sik kullanilan ve iyi belgelenmis bir tekniktir.
