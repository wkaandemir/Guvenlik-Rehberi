# TODO: CVE-2026-21519 — Desktop Window Manager (DWM) Yetki Yukseltme

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21519 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21519, Windows Desktop Window Manager (DWM) bileseninde bulunan bir tip karisikligi (type confusion) hatasindan kaynaklanan yetki yukseltme (Elevation of Privilege) zafiyetidir. CVSS skoru 7.8 olan bu zafiyet, dusuk yetkili bir yerel saldirganin DWM sureci uzerinden SYSTEM yetkilerine erismesine olanak tanimaktadir. Microsoft Subat 2026 Patch Tuesday kapsaminda yamasi yayinlanmis olup, CISA tarafindan aktif istismar edildigi dogrulanmistir. Vibecoding ortamlarinda calisan otonom kodlama ajanlarinin bu zafiyet araciligiyla tam sistem kontrolu elde etmesi olasiligi nedeniyle modern gelistirme ortamlari icin ek risk tasimaktadir.
*   **Etkilenen bilesenler:** Windows Desktop Window Manager (dwm.exe), DWM Core cekirdek bileseni (dwmcore.dll), DirectComposition API, Windows 10/11 tum surumleri, Windows Server 2019/2022/2025 Desktop Experience kurulumlar

---

### 2. Teknik detay (nasil calisiyor)
*   DWM (Desktop Window Manager), Windows masaustu bilesim motorudur ve SYSTEM yetkileriyle calisan kritik bir surectir. DirectComposition API uzerinden pencere efektleri, animasyonlar ve gorsel bilesim islemlerini yonetir.
*   Zafiyet, DWM'nin DirectComposition nesne agaci icerisindeki bir gorsel kaynak (visual resource) nesnesini islerken, nesnenin turunu yanlis yorumlamasindan (type confusion) kaynaklanmaktadir. DWM, belirli bir nesne turunu beklerken farkli bir turdeki nesne islenmekte ve bu tip uyumsuzlugu bellekteki veri yapilarinin yanlis offset'lerden okunmasina yol acmaktadir.
*   Saldirgan, kullanici modundan DirectComposition API cagrialri araciligiyla DWM'ye ozel hazirlanmis nesne talepleri gondererek tip karisikligi hatasini tetikler. Bu hata, DWM'nin SYSTEM baglaminda calisan bellek alaninda kontrollu bellek okuma/yazma primitifi saglar.
*   Elde edilen bellek primitifi, DWM surecinin token yapisini manipule ederek saldirganin SYSTEM yetkilerine yukselmesini saglar. Istismar tamamiyla yerel erisimle gerceklesir ve kullanici etkilesimi gerektirmez.
*   **Neden:** Kok neden, DirectComposition nesne agacinda tip dogrulama (type checking) mekanizmasinin yetersiz olmasisir. DWM, nesne turunu belirlemek icin bir tur alanini (type field) okumakta, ancak bu alanin saldirgan tarafindan manipule edilebilmesi nedeniyle tip karisikligi olusumaktadir. Nesne turu ile bellek duzeni arasindaki uyumsuzluk, out-of-bounds okuma/yazma'ya donusur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** DWM sureci (dwm.exe) — dusuk yetkili bir kullanici oturumundan SYSTEM yetkilerine yukseltme
*   **2. Normal durum:**
    ```
    1. Uygulama, DirectComposition API uzerinden gorsel efekt (visual) olusturur
    2. DWM, nesne agacinda yeni gorsel kaynagi tahsis eder ve turunu kaydeder
    3. Nesne islendiginde tur alani kontrol edilir ve dogru bellek ofset'i kullanilir
    4. Islem tamamlandiginda nesne guvenli sekilde serbest birakilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    // Kavramsal PoC — DWM type confusion ile EoP
    #include <dcomp.h>

    // 1. DirectComposition cihazi olustur
    IDCompositionDevice* pDevice = NULL;
    DCompositionCreateDevice(NULL, IID_IDCompositionDevice, (void**)&pDevice);

    // 2. Gorsel nesne agaci olustur
    IDCompositionVisual* pVisual = NULL;
    pDevice->CreateVisual(&pVisual);

    // 3. Type confusion tetikle — beklenen tur: Surface, verilen tur: VirtualSurface
    // DWM, Surface bellek ofset'leriyle VirtualSurface verisini okumaya calisir
    IDCompositionVirtualSurface* pSurface = NULL;
    pDevice->CreateVirtualSurface(0x1000, 0x1000, DXGI_FORMAT_B8G8R8A8_UNORM,
        DXGI_ALPHA_MODE_PREMULTIPLIED, &pSurface);

    // 4. Nesneyi gorsel agaca bagla — tur karisikligi tetiklenir
    pVisual->SetContent(pSurface);  // DWM tarafinda type confusion olusur

    // 5. Tip karisikligi nedeniyle DWM belleginde kontrollu yazma primitifi elde et
    // Bu primitif ile DWM surec token'ini SYSTEM token'iyla degistir
    // HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwmPid);
    // NtWriteVirtualMemory ile token degistirme

    pDevice->Commit();  // DWM'ye nesne agacini isle komutu gonder
    // SYSTEM yetkileri elde edildi
    ```
*   **4. Analiz:** DWM, IDCompositionSurface ve IDCompositionVirtualSurface nesnelerini farkli bellek duzenleriyle (layout) depolar. Surface nesnesi 0x80 byte, VirtualSurface ise 0xA0 byte buyuklugundedir. Type confusion durumunda DWM, Surface ofset'leriyle VirtualSurface verisini okur ve 0x80-0xA0 arasindaki alandaki veriler bitisik bellek bolgesinden okunur (out-of-bounds read). Bu veri, heap spraying ile kontrol edildiginde SYSTEM token adresini isaret edecek sekilde ayarlanabilir.
*   **5. Kanit:** Basarili istismar sonrasinda saldirganin sureci SYSTEM yetkilerine sahip olur. Security Event Log'da EventID 4672 (ozel yetki atamasi) beklenmeyen bir kullanici icin olusur. DWM sureci cokmeden sessiz yetki yukseltme gerceklesir. whoami komutu "nt authority\system" dondurur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.8
*   **Saldiri Yuzeyi:** Yerel erisim (kullanici oturumu gerektirir; vibecoding ajanlarinin calistigi ortamlarda ajan uzerinden tetiklenebilir)
*   **Karmasiklik:** Orta (DirectComposition API bilgisi ve heap spraying teknigi gerektirir; ancak aktif istismar edildigi icin saldiri araci mevcut)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Subat 2026 Patch Tuesday yamasini tum Windows sistemlerine uygulayin. DWM sureci icin Exploit Protection ayarlarini etkinlestirin (DEP, CFG, SEHOP, ForceRelocateImages). Vibecoding ajanlarinin calistigi ortamlarda ajan yetkilerini standart kullanici seviyesiyle sinirlandirin.
*   **Orta vadeli:** Kritik sunucularda Windows Server Core (GUI'siz) kurulumuna gecis yapin — DWM saldiri yuzeyini tamamen ortadan kaldirir. EDR cozumlerinde DWM surecine yonelik anormal DirectComposition API cagrilarini izleyen davranissal kurallar tanimlayin. Standart kullanici hesaplari icin en az yetki prensibini uygulayin. Vibecoding platformlari icin sandbox izolasyonu zorunlu kilin.
*   **Uzun vadeli:** Masaustu bilesim motorunu kullanici modu izolasyonuna tasima (Windows 11+ surumlerinde planlanan mimari degisiklik). Hardware-enforced Stack Protection (Intel CET) ve Kernel Data Protection (KDP) mekanizmalarini yayginlastirin. Win32k ve DWM cekirdek saldiri yuzeyini azaltan AppContainer izolasyon politikalarini zorunlu kilin. Otonom kodlama ajanlarini calistiran ortamlarin guvenlik denetimini periyodik olarak gerceklestirin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: DWM sureci varsayilan korumalarla calisiyor ===
# Exploit Protection yapilandirilmamis
Get-ProcessMitigation -Name dwm.exe
# Sonuc: DEP = OptIn, CFG = OFF, SEHOP = OFF

# DWM surec butunlugu izlenmiyor
# Type confusion istismari sessiz gerceklesir

# Vibecoding ajanlari yuksek yetkiyle calisiyor
# Ajan SYSTEM yetkisine eristiyse tum sistemi kontrol edebilir
whoami
# Sonuc: ajan-kullanici (ancak EoP sonrasi nt authority\system olabilir)
```

**Guvenli kod:**
```powershell
# === GUVENLI: DWM type confusion EoP'ye karsi sertlestirme ===

# 1. Subat 2026 Patch Tuesday yamasini dogrula
$patch = Get-HotFix | Where-Object {
    $_.InstalledOn -gt "2026-02-01" -and $_.Description -match "Security Update"
} | Sort-Object InstalledOn -Descending | Select-Object -First 5
if (-not $patch) {
    Write-Warning "[!] ACIL: Subat 2026 DWM yamasi bulunamadi!"
} else {
    Write-Host "[+] Guvenlik yamalari kurulu:" ($patch.HotFixID -join ", ")
}

# 2. DWM sureci icin Exploit Protection etkinlestir
Set-ProcessMitigation -Name dwm.exe -Enable DEP, CFG, SEHOP, ForceRelocateImages
Write-Host "[+] dwm.exe icin Exploit Protection etkinlestirildi"

# 3. Sistem geneli CFG (Control Flow Guard) zorunlu kil
Set-ProcessMitigation -System -Enable CFG, DEP, ForceRelocateImages
Write-Host "[+] Sistem geneli korumalar aktif"

# 4. Yetki yukseltme girisimlerini izle
$events = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'; Id = 4672
} -MaxEvents 50 -ErrorAction SilentlyContinue
$suspicious = $events | Where-Object {
    $_.Message -notmatch "SYSTEM|LOCAL SERVICE|NETWORK SERVICE|DWM-"
}
if ($suspicious) {
    foreach ($evt in $suspicious) {
        Write-Warning "[!] Beklenmeyen yetki yukseltme: $($evt.TimeCreated)"
    }
}

# 5. DWM sureci butunluk kontrolu
$dwmProc = Get-Process -Name dwm -ErrorAction SilentlyContinue
if ($dwmProc) {
    $dwmModules = $dwmProc.Modules | Where-Object { $_.ModuleName -match "dwmcore|dwm" }
    foreach ($mod in $dwmModules) {
        $fileHash = Get-FileHash $mod.FileName -Algorithm SHA256
        Write-Host "[*] DWM Modul: $($mod.ModuleName) — Hash: $($fileHash.Hash.Substring(0,16))..."
    }
}

# 6. Sysmon ile DWM'ye yonelik anormal API cagrilarini izle
# Sysmon yapilandirmasina ekle:
# <RuleGroup groupRelation="or">
#   <ProcessAccess onmatch="include">
#     <TargetImage condition="is">C:\Windows\System32\dwm.exe</TargetImage>
#     <GrantedAccess condition="is">0x1F0FFF</GrantedAccess> <!-- PROCESS_ALL_ACCESS -->
#   </ProcessAccess>
# </RuleGroup>
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] DWM bileseni calisan tum Windows masaustu ve server (Desktop Experience) sistemlerini envanterle
*   [ ] Mevcut Exploit Protection ve CFG yapilandirmalarini denetle
*   [ ] Vibecoding ajanlarinin calistigi ortamlari ve yetki seviyelerini tespit et
*   [ ] CISA KEV katalogu ile aktif somuru durumunu dogrula

**Duzeltme**
*   [ ] Microsoft Subat 2026 Patch Tuesday yamasini tum etkilenen sistemlere uygula
*   [ ] DWM sureci icin Exploit Protection ayarlarini etkinlestir (DEP, CFG, SEHOP, ASLR)
*   [ ] Sistem genelinde Control Flow Guard (CFG) politikasini devreye al
*   [ ] Vibecoding ajanlarinin yetkilerini standart kullanici seviyesiyle sinirlandir
*   [ ] Server ortamlarinda Desktop Experience yerine Server Core'a gecis planla

**Dogrulama**
*   [ ] Yama sonrasi DWM type confusion EoP testini tekrarla
*   [ ] Exploit Protection ayarlarinin aktif oldugunu Get-ProcessMitigation ile dogrula
*   [ ] Yetki yukseltme girisimlerinin SIEM/EDR tarafindan tespit edildigini dogrula
*   [ ] Vibecoding ajanlarinin kisitlanmis yetkilerle dogru calistigini test et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Security Event Log'da EventID 4672 (ozel yetki atamasi) — dwm.exe veya DirectComposition iliskili sureclerde beklenmeyen kullanicilar icin izle
    *   Sysmon EventID 10 (ProcessAccess) — dwm.exe'ye PROCESS_ALL_ACCESS ile erisim girisimlerini tespit et
    *   Application Event Log'da DWM ve dwmcore.dll ile iliskili hata/cokme olaylarini izle
    *   Regex: `(?i)(dwm\.exe.*type.confusion|dwmcore.*access.violation|DirectComposition.*invalid.type)`
    *   Regex: `(?i)(4672.*S-1-5-18.*unexpected|dwm.*token.*manipulat)`
*   **Anomali Tespiti:**
    *   Standart kullanici hesabindan SYSTEM yetkisine gecis yapan surec zincirleri (ozellikle dwm.exe uzerinden)
    *   DirectComposition API cagrilarinda anormal nesne olusturma/yok etme oruntuleri (yuksek frekans)
    *   DWM surecinin beklenmeyen alt surecleri baslatmasi (cmd.exe, powershell.exe)
    *   dwm.exe bellek kullaniminda ani ve beklenmeyen artislar (heap spraying belirtisi)
    *   Vibecoding ajanlarinin olagan disi sistem cagrilari yapmasi (CreateProcess, NtWriteVirtualMemory)

---

## Notlar
Aktif istismar dogrulanmis. Microsoft Subat 2026 Patch Tuesday kapsaminda 59 zafiyet arasinda ele alinmistir. DWM, Windows cekirdek saldiri yuzeyinin en kritik bilesenlerinden biridir ve SYSTEM yetkileriyle calismasi nedeniyle yetki yukseltme aciklari yuksek etkiye sahiptir. Vibecoding ortamlarinda calisan ajanlarin bu zafiyet araciligiyla tam sistem kontrolu elde etmesi, otonom ajan guvenlik risklerinin somut bir ornegir. CVE-2026-20805 (DWM bilgi ifsasi, Ocak 2026) ile ayni bilesen ailesinde yer almaktadir.
