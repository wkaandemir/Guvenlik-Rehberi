# TODO: CVE-2026-20822 — Windows Graphics EoP

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-20822 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-20822, Windows Graphics bileseninde (win32kfull.sys cekirdek surucu) bulunan bir yetki yukseltme (Elevation of Privilege) zafiyetidir. CVSS skoru 7.8 olan bu zafiyet, dusuk yetkili bir yerel kullanicinin cekirdek (kernel) duzeyinde bellek bozulmasini tetikleyerek SYSTEM yetkilerine erismesine olanak tanimaktadir. Zafiyet, CVE-2026-20805 (DWM bilgi ifsasi) ile birlestirildiginde ASLR atlamali tam saldiri zinciri olusturabilmekte ve bu durum Windows masaustu ortamlarinda ciddi bir risk profili cizmektedir. Microsoft Ocak 2026 Patch Tuesday kapsaminda yamasi yayinlanmis olup kurumsal ortamlarda yaygin etkiye sahiptir.
*   **Etkilenen bilesenler:** Windows Graphics suruculeri (win32kfull.sys, win32kbase.sys), Windows GDI (Graphics Device Interface) altyapisi, Windows 10/11 tum surumleri, Windows Server 2019/2022/2025 Desktop Experience kurulumlar, DirectX grafik alt sistemi

---

### 2. Teknik detay (nasil calisiyor)
*   Windows Graphics alt sistemi (win32kfull.sys), kullanici modu uygulamalarindan gelen grafik cizim isteklerini cekirdek duzeyinde isleyen kritik bir surucudur. Bu surucu, GDI nesnelerini (bitmap, font, pen, brush vb.) cekirdek havuzunda (kernel pool) yonetir.
*   Zafiyet, win32kfull.sys icerisindeki bir grafik nesne isleyicisinin (handle) yasam dongusu yonetimindeki hatadan kaynaklanmaktadir. Belirli bir GDI nesne turunun serbest birakilmasi (free) sirasinda referans sayacinin dogru dusurulmemesi, use-after-free (UAF) durumuna yol acmaktadir.
*   Saldirgan, UAF durumunu tetikledikten sonra serbest birakilan bellek alanini kontrollu veriyle (kernel pool spraying) doldurarak cekirdek belleginde rastgele yazma (arbitrary write) primitifi elde eder. Bu primitif, surec token'inin yetkilerini SYSTEM seviyesine yukseltmek icin kullanilir.
*   CVE-2026-20805'ten elde edilen DWM bellek adresi, ASLR bypass olarak bu exploit'e entegre edilir ve basari oranini dramatik sekilde arttirir.
*   **Neden:** Kok neden, win32kfull.sys'deki GDI nesne referans sayacinin (reference counter) belirli bir cagri sirasinda atomik olmayan bir sekilde guncellenmesidir. Iki es zamanli islem (race condition) nesnenin erken serbest birakilmasina yol acmakta ve saldirgan bu zamanlama penceresini istismar etmektedir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Windows Graphics cekirdek surucusu (win32kfull.sys) — dusuk yetkili bir kullanici oturumundan SYSTEM yetkilerine yukseltme
*   **2. Normal durum:**
    ```
    1. Uygulama, CreateBitmap/CreateFont gibi GDI API'leri ile grafik nesneleri olusturur
    2. win32kfull.sys, nesneleri cekirdek havuzunda tahsis eder ve referans sayacini yonetir
    3. Nesne kullanimi bitince DeleteObject cagrisiyla guvenli sekilde serbest birakilir
    4. Referans sayaci 0'a dusunce bellek ayrilan havuza geri doner
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    // Kavramsal PoC — Win32k GDI UAF ile yetki yukseltme
    // 1. Cok sayida GDI bitmap nesnesi olustur (heap grooming)
    HBITMAP hBitmaps[4096];
    for (int i = 0; i < 4096; i++) {
        hBitmaps[i] = CreateBitmap(0x60, 1, 1, 32, NULL);
    }

    // 2. Her iki bitmap'i serbest birak — bellekte bosluklar olustur
    for (int i = 0; i < 4096; i += 2) {
        DeleteObject(hBitmaps[i]);
    }

    // 3. Race condition tetikle — es zamanli islemlerle UAF olustur
    // Thread A: Nesneye erisim istegi (NtGdiDdDDIGetContextSchedulingPriority)
    // Thread B: Ayni nesneyi serbest birak (NtGdiDdDDIDestroyContext)
    // Zamanlama penceresi: ~50-100 mikrosaniye

    // 4. Serbest kalan bellek alanina kontrollu veri yaz (pool spraying)
    // Sahte GDI nesne basliginda token adresini hedefle
    PVOID fakeObj = HeapAlloc(...);
    // DWM ASLR leak (CVE-2026-20805) ile hesaplanan adres kullanilir
    *(PULONG64)((PBYTE)fakeObj + TOKEN_OFFSET) = systemTokenAddress;

    // 5. Token degistirme ile SYSTEM yetkisi elde et
    // NtQuerySystemInformation ile dogrula
    ```
*   **4. Analiz:** win32kfull.sys'deki race condition, iki ayri thread'in ayni GDI nesnesine es zamanli erisimi sirasinda ortaya cikar. Referans sayaci atomik olarak guncellenmediginden, Thread B nesneyi serbest birakirken Thread A hala nesneye referans tutmaktadir. Saldirgan bu zamanlama penceresini, serbest birakilan bellek alanini kontrollu veriyle doldurarak istismar eder. KASLR (Kernel ASLR) CVE-2026-20805'ten elde edilen adreslerle bypass edilir.
*   **5. Kanit:** Basarili istismar sonrasinda saldirganin sureci SYSTEM yetkilerine sahip olur (whoami komutu "nt authority\system" dondurur). Security Event Log'da EventID 4672 (ozel yetki atamasi) beklenmeyen bir kullanici icin kayit olusur. Cekirdek duzeyinde bugcheck (mavi ekran) olmadan sessiz yetki yukseltme gerceklesir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.8
*   **Saldiri Yuzeyi:** Yerel erisim (kullanici oturumu gerektirir; ancak phishing veya RDP ile ilk erisim sonrasi kullanilabilir)
*   **Karmasiklik:** Orta (race condition zamanlama penceresi dar, ancak heap grooming ile basari orani arttirilabilir; CVE-2026-20805 ile birlestirildiginde etki kritik seviyeye cikar)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Ocak 2026 Patch Tuesday yamasini tum Windows sistemlerine uygulayin. CVE-2026-20805 yamasini da es zamanli uygulayin (saldiri zincirini kirmak icin). win32kfull.sys icin Exploit Protection ayarlarini etkinlestirin (DEP, CFG, ACG).
*   **Orta vadeli:** Kritik sunucularda Windows Server Core kurulumuna gecis yapin (GDI saldiri yuzeyini ortadan kaldirir). Standart kullanici hesaplarinin yetkilerini en az yetki prensibine gore sinirlandirin. EDR cozumlerinde cekirdek istismar girisimleri icin davranissal tespit kurallarini etkinlestirin. Saldiri yuzeyi azaltma (ASR) kurallarini devreye alin.
*   **Uzun vadeli:** Win32k tabanli cekirdek suruculerinin saldiri yuzeyini azaltmak icin Windows 11+ surumlerinde win32k isolation (AppContainer) ozelligini benimseyin. Hardware-enforced Stack Protection (Intel CET) destekli islemci ve firmware kullanimini yayginlastirin. Cekirdek suruculerindeki bellek guvenligi icin Kernel Data Protection (KDP) ve Hypervisor-Protected Code Integrity (HVCI) politikalarini zorunlu hale getirin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: Varsayilan Windows grafik surucusu ayarlari ===
# win32kfull.sys icin Exploit Protection yapilandirilmamis
Get-ProcessMitigation -Name dwm.exe
# Sonuc: CFG = OFF, DEP = OptIn, ACG = OFF

# Grafik surucusu guncelleme durumu kontrol edilmiyor
Get-WmiObject Win32_VideoController | Select-Object Name, DriverVersion, DriverDate
# Eski surucu — yama uygulanmamis

# Yetki yukseltme girisimlerini izleyen SIEM kurali yok
# EventID 4672 (ozel yetki atamasi) filtresiz birakiliyor
```

**Guvenli kod:**
```powershell
# === GUVENLI: Windows Graphics EoP'ye karsi sertlestirme ===

# 1. Ocak 2026 Patch Tuesday yamasini dogrula
$securityPatches = Get-HotFix | Where-Object {
    $_.InstalledOn -gt "2026-01-10" -and $_.Description -match "Security Update"
} | Sort-Object InstalledOn -Descending
if (-not $securityPatches) {
    Write-Warning "[!] Ocak 2026 guvenlik yamasi bulunamadi — ACIL guncelleme gerekli"
} else {
    Write-Host "[+] Guvenlik yamalari kurulu:" ($securityPatches.HotFixID -join ", ")
}

# 2. Grafik surucusu surumunu dogrula
$gpu = Get-WmiObject Win32_VideoController | Select-Object Name, DriverVersion, DriverDate
Write-Host "[*] GPU Surucusu: $($gpu.Name) v$($gpu.DriverVersion) ($($gpu.DriverDate))"

# 3. Win32k istismar korumasini etkinlestir
Set-ProcessMitigation -Name dwm.exe -Enable DEP, CFG, ForceRelocateImages
Set-ProcessMitigation -System -Enable DEP, CFG, ForceRelocateImages
Write-Host "[+] Exploit Protection etkinlestirildi (DEP + CFG + ASLR)"

# 4. Yetki yukseltme girisimlerini izle
$suspiciousEvents = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4672  # Ozel yetki atamasi
} -MaxEvents 50 -ErrorAction SilentlyContinue
$suspiciousEvents | Where-Object {
    $_.Message -notmatch "SYSTEM|LOCAL SERVICE|NETWORK SERVICE"
} | ForEach-Object {
    Write-Warning "[!] Beklenmeyen yetki yukseltme: $($_.TimeCreated) — $($_.Message.Substring(0,200))"
}

# 5. Attack Surface Reduction (ASR) kurallarini etkinlestir
Add-MpPreference -AttackSurfaceReductionRules_Ids d4f940ab-401b-4efc-aadc-ad5f3c50688a `
    -AttackSurfaceReductionRules_Actions Enabled
Write-Host "[+] ASR kurallari etkinlestirildi"
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Windows grafik surucusu kullanan tum sistemlerin envanterini cikar
*   [ ] CVE-2026-20805 ile zincirleme risk analizini tamamla
*   [ ] Mevcut Exploit Protection ve ASR yapilandirmalarini denetle
*   [ ] Etkilenen sistemlerdeki kullanici yetki profillerini gozden gecir

**Duzeltme**
*   [ ] Microsoft Ocak 2026 Patch Tuesday yamasini tum etkilenen sistemlere uygula
*   [ ] CVE-2026-20805 yamasini da es zamanli uygula (saldiri zincirini kir)
*   [ ] win32kfull.sys / dwm.exe icin Exploit Protection'i etkinlestir (DEP, CFG, ASLR)
*   [ ] Attack Surface Reduction (ASR) kurallarini devreye al
*   [ ] Server ortamlarinda Desktop Experience yerine Server Core'a gecis planla

**Dogrulama**
*   [ ] Yama sonrasi GDI UAF exploit testini tekrarla (kalem testi)
*   [ ] Exploit Protection ayarlarinin aktif oldugunu dogrula
*   [ ] CVE-2026-20805 ile zincirleme saldiri senaryosunu test et
*   [ ] SIEM kurallarinin yetki yukseltme girisimlerini tespit ettigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Security Event Log'da EventID 4672 (ozel yetki atamasi) — beklenmeyen kullanicilar icin filtrele
    *   Security Event Log'da EventID 4688 (yeni surec olusturma) — SYSTEM olarak calisan beklenmeyen surecleri izle
    *   Sysmon EventID 10 (ProcessAccess) — win32kfull.sys veya dwm.exe hedefli erisim girisimlerini tespit et
    *   Regex: `(?i)(win32k.*use.after.free|GDI.*pool.*corruption|token.*privilege.*escalat)`
    *   Regex: `(?i)(4672.*S-1-5-18|SeDebugPrivilege.*unexpected.user)`
*   **Anomali Tespiti:**
    *   Standart kullanici hesabindan SYSTEM yetkisine gecis yapan surec zincirleri
    *   Kisa surede cok sayida GDI nesne olusturma/silme dongusu (heap grooming belirtisi)
    *   dwm.exe veya explorer.exe'nin beklenmeyen alt surecleri (cmd.exe, powershell.exe) baslatmasi
    *   Win32k cekirdek surucusunde hata raporlari (WER) veya mini dump olusumu
    *   CVE-2026-20805 ile ayni zaman diliminde DWM ALPC anormallikleriyle birlesik yetki yukseltme girisimleri

---

## Notlar
CVE-2026-20805 ile birlestirildiginde tam saldiri zinciri olusturabilir. Microsoft Ocak 2026 Patch Tuesday kapsaminda 114 zafiyet arasinda ele alinmistir. Win32k surucusu, Windows cekirdeginde en sik istismar edilen bilesenlerin basinda gelmektedir; bu nedenle Server Core kurulumuna gecis uzun vadede en etkili cozumdur. EoP kategorisindeki 58 zafiyet arasinda en yuksek etkiye sahip olanlardan biridir.
