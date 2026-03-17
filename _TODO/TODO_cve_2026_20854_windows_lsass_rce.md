# TODO: CVE-2026-20854 — Windows LSASS RCE

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-20854 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-20854, Windows Local Security Authority Subsystem Service (LSASS) bileseninde tespit edilen bir uzaktan kod yurutme (Remote Code Execution) zafiyetidir. CVSS skoru 7.5 olan bu zafiyet, saldirganin ag uzerinden ozel olarak hazirlanmis kimlik dogrulama istekleri gondererek LSASS sureci baglaminda rastgele kod yurutmesine olanak tanimaktadir. LSASS, Windows kimlik dogrulama altyapisinin temel tasidir ve Active Directory ortamlarinda Kerberos, NTLM, LDAP gibi protokolleri yonetmektedir. Bu zafiyetin istismari, domain controller'lar uzerinde tam yetki ele gecirme, kimlik bilgisi calinimi ve genis capli lateral movement icin kullanilabilir.
*   **Etkilenen bilesenler:** LSASS sureci (lsass.exe), Windows kimlik dogrulama protokolleri (Kerberos, NTLM, LDAP), Active Directory Domain Controller'lar, Windows 10/11 ve Windows Server 2019/2022/2025, domain-joined tum Windows sistemleri

---

### 2. Teknik detay (nasil calisiyor)
*   LSASS (Local Security Authority Subsystem Service), Windows isletim sisteminde kimlik dogrulama, yetkilendirme ve guvenlik politikalarinin uygulanmasindan sorumlu olan cekirdek servisdir. SYSTEM yetkileriyle calisan bu surec, kullanici parolalarinin hash'lerini, Kerberos biletlerini ve NTLM tokenlarini bellekte tutar.
*   Zafiyet, LSASS'in belirli bir kimlik dogrulama protokolu uzantisini (authentication package) islerken gelen veri yapisinin boyutunu dogru sekilde dogrulamamasindan kaynaklanmaktadir. Ozel hazirlanmis bir kimlik dogrulama istegindeki fazla buyuk veya bicim bozuk alan, yigin tabanli tampon tasmasi (heap-based buffer overflow) tetiklemektedir.
*   Saldirgan, ag uzerinden (445/TCP — SMB veya 88/TCP — Kerberos veya 389/TCP — LDAP portu) hedef sisteme ozel hazirlanmis bir kimlik dogrulama paketi gondererek bu tampon tasmasini tetikler. Basarili istismar durumunda LSASS sureci baglaminda (SYSTEM yetkileri) rastgele kod yurutulur.
*   Domain controller'lar birincil hedeftir cunku LSASS burada tum domain kimlik bilgilerini isler ve saldirgan tam domain hakimiyeti elde edebilir.
*   **Neden:** Kok neden, LSASS'in kimlik dogrulama paketi isleyicisinde gelen verinin uzunluk alanini (length field) tampon boyutuyla karsilastirmadan dogrudan bellege kopyalamasidir. Sinir kontrolu eksikligi (CWE-120: Buffer Copy without Checking Size of Input) bu heap overflow'u olusturmaktadir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Windows Domain Controller uzerindeki LSASS sureci — Kerberos (88/TCP), LDAP (389/TCP) veya SMB (445/TCP) portu uzerinden erisilebilir
*   **2. Normal durum:**
    ```
    1. Istemci, domain controller'a Kerberos AS-REQ veya NTLM Type 1 mesaji gonderir
    2. LSASS, kimlik dogrulama isteigini isler ve kullanici kimligini dogrular
    3. Basarili dogrulamada TGT (Ticket Granting Ticket) veya NTLM yaniti doner
    4. Tum islem LSASS bellek alaninda guvenli tampon sinirlari icinde gerceklesir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```python
    # Kavramsal PoC — LSASS heap overflow tetiklemesi
    import socket
    import struct

    target_dc = "192.168.1.10"
    kerberos_port = 88

    # 1. Normal Kerberos AS-REQ yapisini olustur
    as_req = bytearray()
    as_req += b'\x30\x82'  # ASN.1 SEQUENCE

    # 2. Kimlik dogrulama paketi uzantisindaki veri alanini
    #    tampon sinirinin otesine tasir (4096 byte yerine 8192)
    malicious_padata = b'\x00' * 8192  # Heap overflow tetikleyici

    # 3. Bicimsiz PA-DATA alanini AS-REQ'e ekle
    as_req += struct.pack('>H', len(malicious_padata))
    as_req += malicious_padata

    # 4. Hedefe gonder
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((target_dc, kerberos_port))
    sock.send(as_req)
    # LSASS heap overflow tetiklenir — SYSTEM yetkisiyle kod yurutme
    ```
*   **4. Analiz:** LSASS, gelen PA-DATA (Pre-Authentication Data) alaninin boyutunu kontrol etmeden sabit boyutlu bir heap tamponuna kopyalamaktadir. 4096 byte sinirinin otesindeki veri, bitisik heap bloklarini ezerek saldirganin heap metadata'sini kontrol etmesine olanak tanir. Heap metadata manipulasyonu ile kod yonlendirme (code redirection) saglanaarak SYSTEM baglaminda rastgele kod yurutulur.
*   **5. Kanit:** Basarili istismar sonrasinda LSASS sureci icinde saldirganin shell kodu calisir. Saldirgan, domain controller uzerinde SYSTEM yetkisine sahip olur ve ntds.dit veritabanina erisebilir. System Event Log'da LSASS cokme olaylari (EventID 1000/1001) veya beklenmeyen kimlik dogrulama hatalari (EventID 4625 serisi) gorunur. Ag izlemede hedef DC'ye yonelik asiri buyuk Kerberos/LDAP paketleri tespit edilebilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.5
*   **Saldiri Yuzeyi:** Ag tabanli (domain controller portlarina erisim yeterli; Kerberos 88/TCP, LDAP 389/TCP, SMB 445/TCP)
*   **Karmasiklik:** Orta (heap overflow istismari deterministic degil; ancak heap spraying teknikleriyle basari orani arttirilabilir; kimlik dogrulama gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Ocak 2026 Patch Tuesday yamasini tum domain controller'lara oncelikli uygulayin. LSASS surec korumasini (RunAsPPL — Protected Process Light) etkinlestirin. Domain controller'lara erisimi yalnizca yetkili ag segmentlerinden izin verecek sekilde firewall kurallariyla sinirlandirin. Windows Defender Credential Guard'i etkinlestirin.
*   **Orta vadeli:** Ag segmentasyonu ile domain controller'lari izole bir VLAN'a alin. Kerberos ve LDAP trafiginde anormal paket boyutlarini tespit eden IDS/IPS kurallari tanimlayin. LSASS bellek dumplarini engelleyin (Windows Defender icinde LSASS korumasi). Tum domain admin hesaplarinda MFA zorunlulugu uygulayin. Active Directory topolojisinde Tier-0/1/2 segmentasyonunu devreye alin.
*   **Uzun vadeli:** Virtualization-Based Security (VBS) ve Hypervisor-Protected Code Integrity (HVCI) ile LSASS'i sanal izolasyona alin. Parola hash'lerinin bellekte tutulma suresini kisaltmak icin Kerberos armoring ve FAST (Flexible Authentication Secure Tunneling) protokolunu uygulayin. NTLM protokolunu tamamen devre disi birakip yalnizca Kerberos kimlik dogrulamaya gecin. PAW (Privileged Access Workstation) mimarisini Tier-0 yonetim icin zorunlu kilin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: LSASS varsayilan yapilandirma — koruma yok ===
# RunAsPPL devre disi — LSASS bellegi koruma altinda degil
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL"
# Sonuc: RunAsPPL = 0 veya ayar mevcut degil

# Credential Guard etkin degil
Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard
# Sonuc: VirtualizationBasedSecurityStatus = 0

# LSASS bellek dump'ina izin var
# Mimikatz, procdump gibi araclar kimlik bilgilerini cikartabilir
# Task Manager > lsass.exe > Create Dump File — engellenmemis

# Domain controller portlari (88, 389, 445) tum agdan erisilebilir
# Ag segmentasyonu uygulanmamis
```

**Guvenli kod:**
```powershell
# === GUVENLI: LSASS sertlestirme ve koruma ===

# 1. LSASS korumasini etkinlestir (RunAsPPL)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "RunAsPPL" -Value 1 -Type DWord
Write-Host "[+] LSASS RunAsPPL etkinlestirildi (yeniden baslatma gerekli)"

# 2. Windows Defender Credential Guard'i etkinlestir
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" `
    -Name "EnableVirtualizationBasedSecurity" -Value 1 -Type DWord
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" `
    -Name "RequirePlatformSecurityFeatures" -Value 3 -Type DWord
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "LsaCfgFlags" -Value 1 -Type DWord
Write-Host "[+] Credential Guard etkinlestirildi (UEFI kilidi ile)"

# 3. LSASS bellek dump'larini engelle (ASR kurali)
Add-MpPreference -AttackSurfaceReductionRules_Ids 9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2 `
    -AttackSurfaceReductionRules_Actions Enabled
Write-Host "[+] LSASS bellek dump korumasi etkinlestirildi"

# 4. Ocak 2026 yamasini dogrula
$patch = Get-HotFix | Where-Object {
    $_.InstalledOn -gt "2026-01-10" -and $_.Description -match "Security Update"
}
if (-not $patch) {
    Write-Warning "[!] ACIL: Ocak 2026 LSASS yamasi bulunamadi!"
} else {
    Write-Host "[+] Guvenlik yamasi mevcut:" ($patch.HotFixID -join ", ")
}

# 5. Domain controller portalarina ag segmentasyonu uygula
# Windows Firewall kurali — yalnizca yetkili subnet'lerden erisim
New-NetFirewallRule -DisplayName "DC-Kerberos-Restrict" -Direction Inbound `
    -LocalPort 88 -Protocol TCP -RemoteAddress "10.0.1.0/24","10.0.2.0/24" `
    -Action Allow -Profile Domain
New-NetFirewallRule -DisplayName "DC-LDAP-Restrict" -Direction Inbound `
    -LocalPort 389 -Protocol TCP -RemoteAddress "10.0.1.0/24","10.0.2.0/24" `
    -Action Allow -Profile Domain
Write-Host "[+] Domain controller port kisitlamalari uygulanadi"
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum domain controller'lari ve LSASS'i aktif kullanan sunuculari envanterle
*   [ ] Mevcut LSASS koruma durumunu (RunAsPPL, Credential Guard) denetle
*   [ ] Domain controller'lara ag erisim haritasini cikar (hangi segmentlerden erisilebiliyor)
*   [ ] Active Directory Tier-0 varliklarin risk analizini tamamla

**Duzeltme**
*   [ ] Microsoft Ocak 2026 Patch Tuesday yamasini tum domain controller'lara ONCELIKLI uygula
*   [ ] LSASS RunAsPPL korumasini etkinlestir
*   [ ] Windows Defender Credential Guard'i devreye al
*   [ ] LSASS bellek dump'larini engelleyen ASR kuralini etkinlestir
*   [ ] Domain controller portlarina (88, 389, 445) ag segmentasyonu uygula
*   [ ] NTLM kullanimini kisitla ve Kerberos'a gecisi planla

**Dogrulama**
*   [ ] Yama sonrasi LSASS heap overflow exploit testini tekrarla
*   [ ] RunAsPPL ve Credential Guard'in aktif oldugunu dogrula
*   [ ] Mimikatz/procdump ile LSASS bellek dump'inin engellenmesini test et
*   [ ] IDS/IPS kurallarinin anormal Kerberos/LDAP paketlerini tespit ettigini dogrula
*   [ ] Domain controller ag segmentasyonunun calistigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   System Event Log'da EventID 1000/1001 (LSASS surec cokmeleri) izle
    *   Security Event Log'da EventID 4625 (basarisiz oturum acma girisimleri) seri halde tekrarlayan kayitlari tespit et
    *   Security Event Log'da EventID 4776 (NTLM kimlik dogrulama) basarisiz girisimleri izle
    *   Regex: `(?i)(lsass\.exe.*crash|lsass.*heap.*overflow|buffer.*overrun.*lsa)`
    *   Regex: `(?i)(4625.*Status:\s*0xC000006[5-9]|Kerberos.*malformed.*ticket)`
    *   Domain controller'lara gelen Kerberos (88/TCP) ve LDAP (389/TCP) trafiginde anormal paket boyutlarini izle (>4096 byte PA-DATA)
*   **Anomali Tespiti:**
    *   LSASS surecinin beklenmeyen alt surecleri baslatmasi (cmd.exe, powershell.exe, mshta.exe)
    *   Domain controller uzerinde LSASS bellek kullaniminin ani artmasi
    *   Tek bir kaynaktan kisa surede cok sayida basarisiz kimlik dogrulama girisimleri (brute-force veya exploit spray)
    *   LSASS surecinin yeni ag baglantilari acmasi (C2 callback belirtisi)
    *   ntds.dit veya SYSTEM/SAM hive dosyalarina beklenmeyen erisim girisimleri (DCSync veya credential theft belirtisi)

---

## Notlar
Kimlik dogrulama altyapisindaki en kritik bileseni hedef alir. Domain controller'lar birincil risk altindadir. Lateral movement ve tam domain ele gecirme icin kullanilabilir. Microsoft Ocak 2026 Patch Tuesday kapsaminda 114 zafiyet arasinda ele alinmistir. LSASS, Mimikatz gibi araclarin birincil hedefi olmaya devam etmekte olup RunAsPPL ve Credential Guard korumalari istismar sonrasi etkiyi azaltsa da yama uygulamasi zorunludur.
