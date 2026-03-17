# TODO: CVE-2026-21265 — Secure Boot Bypass

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21265 |
| **Kritiklik** | Orta |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21265, Windows Secure Boot mekanizmasini etkileyen bir guvenlik ozelligi bypass zafiyetidir. 2011 yilinda yayinlanan Microsoft kok sertifikalarinin Haziran 2026'da sona erecek olmasi, firmware duzeyinde guven zincirinin kopmasina neden olmaktadir. CVSS skoru 6.4 olan bu zafiyet, guncellenmemis sistemlerde saldirganin Secure Boot dogrulamasini asarak sistem acilis surecine (boot process) rootkit veya bootkit enjekte etmesine olanak tanir. Kamuoyuna aciklanmis (publicly disclosed) olan bu zafiyet, fiziksel erisimi olan veya firmware duzeyinde yazma yetkisi elde etmis saldirganlar icin ciddi bir tehdit olusturmaktadir.
*   **Etkilenen bilesenler:** UEFI Secure Boot mekanizmasi, Microsoft 2011 kok sertifikalari (KEK ve db), Windows Boot Manager (bootmgfw.efi), BitLocker boot butunluk dogrulamasi, Windows 10/11 ve Windows Server 2019/2022/2025, tum UEFI tabanli PC ve sunucu donanimlari

---

### 2. Teknik detay (nasil calisiyor)
*   Secure Boot, sistem acilisi sirasinda yalnizca guvenilir imzali yazilimlarin yuklenmesine izin veren UEFI tabanli bir guvenlik mekanizmasidir. Bu mekanizma, bir sertifika zinciri (PK > KEK > db) kullanarak her boot bileseninin (bootloader, cekirdek vb.) dijital imzasini dogrular.
*   2011 yilinda yayinlanan Microsoft kok sertifikalari, bu guven zincirinin temelini olusturmaktadir. Bu sertifikalarin Haziran 2026'da sona ermesi planlanmakta olup, suresi dolan sertifikalar artik yeni imza dogrulamalari yapamamaktadir.
*   Zafiyet, suresi dolmus sertifika ile imzalanmis eski veya manipule edilmis boot bilesenlerinin Secure Boot tarafindan hala kabul edilebilmesinden kaynaklanmaktadir. Saldirgan, suresi dolmus sertifika zincirine dayanan bir bootloader imzalayarak veya eski bir zafiyetli bootloader'i (orn. onceki Secure Boot bypass'larda kullanilan shim'ler) yukleyerek koruma mekanizmasini asabilir.
*   Basarili istismar durumunda saldirgan, isletim sistemi yuklenmeden once cekirdek duzeyinde rootkit veya bootkit yurutur. Bu seviyedeki kotu amacli yazilim, isletim sisteminin guvenlik mekanizmalarindan (antivirus, EDR) tamamen gorunnmez olur.
*   **Neden:** Kok neden, Microsoft'un 2011 kok sertifikalari icin vade sonu planlamasini zamaninda tamamlayamamasi ve 2023 sertifikalarina gecis surecinin yeterince hizli yayginlastirilamamasidir. Ayrica, Secure Boot revocation (iptal) listesinin (dbx) guncel tutulmamasinin saldiri yuzeyini genisletmektedir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** UEFI Secure Boot etkin bir Windows sistemi — fiziksel erisim veya firmware yazma yetkisi gerektirir
*   **2. Normal durum:**
    ```
    1. Sistem acildiginda UEFI firmware, Secure Boot politikasini uygular
    2. Windows Boot Manager (bootmgfw.efi) dijital imzasi db sertifikalariyla dogrulanir
    3. Imza gecerli ise boot islemi devam eder; gecersiz ise sistem boot'u durdurur
    4. Sertifika zinciri: PK (Platform Key) > KEK > db (Authorized Signatures)
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    # Kavramsal PoC — Secure Boot bypass suresi dolmus sertifika ile

    # 1. Hedef sistemin UEFI degiskenlerini oku (fiziksel erisim veya UEFI shell)
    # EFI Shell> dmpstore db -b

    # 2. Suresi dolmus 2011 Microsoft sertifikasini kullanarak
    #    manipule edilmis bir bootloader imzala
    # signtool sign /fd SHA256 /f expired_ms_2011.pfx /t <timestamp> evil_bootmgr.efi

    # 3. Manipule edilmis bootloader'i EFI System Partition'a yerlestir
    # mountvol S: /S
    # copy evil_bootmgr.efi S:\EFI\Microsoft\Boot\bootmgfw.efi

    # 4. Bootloader, rootkit payload'unu cekirdek yuklenmeden once calistirir
    # Isletim sistemi normal gorunur ancak cekirdek seviyesinde persistence kazanilmistir

    # 5. BitLocker TPM dogrulamasi da bypass edilebilir cunku
    #    TPM boot olcumleri (PCR) degismis sertifika zincirini algilamaz
    ```
*   **4. Analiz:** Suresi dolmus sertifika ile imzalanmis bootloader, dbx (iptal listesi) guncellenmemis sistemlerde hala gecerli sayilir. UEFI firmware, sertifika vade kontrolunu katı sekilde uygulamiyorsa veya zaman kaynagi manipule edilebiliyorsa, saldirgan gecmiste gecerli olan ancak artik guvenilmemesi gereken bir imza ile boot bilesenini gecerli kilar. Saldiri fiziksel erisim veya firmware yazma yetkisi (onceki bir exploit ile elde edilmis) gerektirir.
*   **5. Kanit:** Basarili istismar sonrasinda sistem normal gorununr ancak acilis sirasinda rootkit/bootkit aktiftir. msinfo32.exe'de "Secure Boot State: Off" veya "Secure Boot State: Unsupported" gorunebilir. Measured Boot loglari (TCGLog) manipule edilmis boot bilesenlerini icerin ancak karsilastirma referansi yoksa tespit zorlasir. PowerShell ile Confirm-SecureBootUEFI komutu beklenmeyen sonuclar dondurur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Orta
*   **CVSS Skoru:** 6.4
*   **Saldiri Yuzeyi:** Fiziksel erisim veya firmware yazma yetkisi (uzaktan dogrudan istismar edilemez; ancak baska bir zafiyet ile firmware erisimi saglandiysa zincirleme kullanilabilir)
*   **Karmasiklik:** Yuksek (fiziksel erisim, UEFI degisken manipulasyonu ve imza araci gerektir; ancak surecin iyi belgelenmis olmasi nedeniyle bilgili saldirganlar icin uygulanabilir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Tum sistemlerde Secure Boot sertifika durumunu kontrol edin ve 2011 sertifikalarinin vade bilgisini dogrulayin. Microsoft'un yayinladigi 2023 kok sertifika guncelleme paketini (KB50xxxxx) tum sistemlere uygulayin. Secure Boot dbx (iptal listesi) guncellemesini firmware uzerinden uygulayin. BitLocker kurtarma anahtarlarini sertifika gecisi oncesinde yedekleyin.
*   **Orta vadeli:** UEFI firmware'i tum donanimlarda en son surume guncelleyin. Tum boot bilesenlerinin 2023 sertifikasi ile yeniden imzalandigini dogrulayin. Measured Boot ve TPM tabanli Remote Attestation mekanizmalarini devreye alin. Disk sifreleme (BitLocker) PCR politikalarini yeni sertifika zincirine gore yeniden yapilandirin.
*   **Uzun vadeli:** Firmware guncelleme sureclerini WSUS/SCCM/Intune uzerinden otomatik hale getirin. UEFI Secure Boot sertifika yasam dongusu yonetimini kurumsal degisim yonetimi (change management) surecine entegre edin. Donanim tabanli guven kokleri (Hardware Root of Trust — TPM 2.0, Intel Boot Guard) ile boot zincirini ek katmanlarla koruyun. Sertifika vade sonu izleme otomasyonu kurun (90 gun oncesinden uyari).

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: Secure Boot suresi dolmus sertifika ile calisiyor ===
# Sertifika durumu kontrol edilmiyor
Confirm-SecureBootUEFI
# Sonuc: True (ancak arka planda suresi dolmak uzere olan 2011 sertifikasi)

# Mevcut db sertifikalari incelenmiyor
# Suresi dolmus sertifikalar hala guven veritabaninda
Get-SecureBootUEFI -Name db | Out-Null
# Sertifika vade kontrolu yapilmiyor

# dbx (iptal listesi) guncellenmemis
# Eski zafiyetli bootloader'lar hala kabul ediliyor
Get-SecureBootUEFI -Name dbx | Out-Null
# Son guncelleme tarihi bilinmiyor

# Firmware guncelleme durumu kontrol edilmiyor
Get-WmiObject -Class Win32_BIOS | Select-Object SMBIOSBIOSVersion
# Eski firmware — guvenlik yamalari eksik
```

**Guvenli kod:**
```powershell
# === GUVENLI: Secure Boot sertifika gecisi ve dogrulama ===

# 1. Secure Boot durumunu dogrula
$secureBootState = Confirm-SecureBootUEFI
if (-not $secureBootState) {
    Write-Warning "[!] Secure Boot DEVRE DISI — acil mudahale gerekli"
}

# 2. db sertifikalarini kontrol et ve vade durumunu raporla
try {
    $dbCerts = Get-SecureBootUEFI -Name db
    $dbCerts | ForEach-Object {
        $cert = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new($_.Bytes)
        $daysRemaining = ($cert.NotAfter - (Get-Date)).Days
        $status = if ($daysRemaining -lt 0) { "SURESI DOLMUS" }
                  elseif ($daysRemaining -lt 180) { "UYARI: $daysRemaining gun kaldi" }
                  else { "GECERLI" }
        [PSCustomObject]@{
            Subject     = $cert.Subject
            NotAfter    = $cert.NotAfter
            Status      = $status
            Thumbprint  = $cert.Thumbprint
        }
    } | Format-Table -AutoSize
} catch {
    Write-Warning "[!] Secure Boot sertifikalari okunamadi: $_"
}

# 3. Firmware surum kontrolu
$bios = Get-WmiObject -Class Win32_BIOS
Write-Host "[*] BIOS Surumu: $($bios.SMBIOSBIOSVersion)"
Write-Host "[*] BIOS Tarihi: $($bios.ReleaseDate)"

# 4. Microsoft 2023 sertifika guncelleme durumunu dogrula
$certUpdate = Get-HotFix | Where-Object {
    $_.Description -match "Security Update" -and $_.InstalledOn -gt "2026-01-01"
}
if ($certUpdate) {
    Write-Host "[+] Guvenlik guncellemeleri mevcut:" ($certUpdate.HotFixID -join ", ")
} else {
    Write-Warning "[!] 2023 sertifika guncelleme paketi bulunamadi — acil uygulama gerekli"
}

# 5. BitLocker kurtarma anahtarini yedekle (sertifika gecisi oncesi)
$blVolumes = Get-BitLockerVolume | Where-Object { $_.ProtectionStatus -eq "On" }
$blVolumes | ForEach-Object {
    $recoveryKey = ($_.KeyProtector | Where-Object { $_.KeyProtectorType -eq "RecoveryPassword" }).RecoveryPassword
    Write-Host "[*] BitLocker Volume: $($_.MountPoint) — Kurtarma Anahtari: $recoveryKey"
    Write-Warning "[!] Bu anahtari guvenli bir konumda yedekleyin!"
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum UEFI Secure Boot etkin sistemlerin envanterini cikar
*   [ ] Mevcut db/dbx sertifika durumlarini ve vade tarihlerini kontrol et
*   [ ] 2011 sertifikasina bagli bootloader ve shim surumlerini tespit et
*   [ ] BitLocker kurtarma anahtarlarini tum sistemler icin yedekle
*   [ ] UEFI firmware surumlerini donanim ureticisinin guvenlik listesiyle karsilastir

**Duzeltme**
*   [ ] Microsoft 2023 kok sertifika guncelleme paketini tum sistemlere uygula
*   [ ] UEFI firmware'i en son guvenli surume guncelle (donanim ureticisi bulteni)
*   [ ] Secure Boot dbx (iptal listesi) guncellemesini uygula
*   [ ] BitLocker TPM PCR politikalarini yeni sertifika zincirine gore yapilandir
*   [ ] Measured Boot ve Remote Attestation mekanizmalarini etkinlestir

**Dogrulama**
*   [ ] Secure Boot zincirinin 2023 sertifikalariyla dogru calistigini dogrula
*   [ ] Boot surecinde manipulasyon testini (eski bootloader ile) tekrarla
*   [ ] BitLocker'in yeni sertifika zinciriyle sorunsuz calistigini dogrula
*   [ ] Firmware guncelleme sonrasi tum boot olcumlerinin (PCR) dogru oldugunu kontrol et
*   [ ] Sertifika vade izleme otomasyonunun uyari urettigini test et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   System Event Log'da Secure Boot dogrulama hatalari izle (EventID 12 — TPM boot olcumu basarisiz)
    *   Microsoft-Windows-Kernel-Boot EventID 280 (Secure Boot policy violation) izle
    *   Regex: `(?i)(secure.boot.*violation|boot.*integrity.*fail|certificate.*expir)`
    *   Regex: `(?i)(dbx.*update.*fail|UEFI.*variable.*tamper|boot.*policy.*change)`
    *   Firmware guncelleme olaylarini (WSUS/SCCM/Intune) merkezi logla ve takip et
*   **Anomali Tespiti:**
    *   Secure Boot durumunun beklenmedik sekilde "Off" veya "Unsupported" olarak raporlanmasi
    *   EFI System Partition (ESP) uzerinde beklenmeyen dosya degisiklikleri (bootmgfw.efi, shim.efi)
    *   TPM PCR degerlerinin onceki baseline'dan sapmasi (measured boot anomalisi)
    *   BitLocker kurtarma moduna beklenmeyen gecisler (boot zinciri degisikligi belirtisi)
    *   Firmware surumunun ureticinin guvenli listesinde bulunmamasi

---

## Notlar
Haziran 2026 son tarihi kritik — guncellenmemis sistemler boot sureci manipulasyonuna acik kalacak. Microsoft, 2023 sertifikalarina gecis icin simdiden aksiyon alinmasini vurgulamaktadir. BitLocker kullanan sistemlerde sertifika gecisi oncesinde kurtarma anahtarlarinin yedeklenmesi zorunludur; aksi halde veri erisimi kaybedilebilir. Kamuoyuna aciklanmis olmasi nedeniyle saldirganlar icin bilgi bariyeri dusuktur.
