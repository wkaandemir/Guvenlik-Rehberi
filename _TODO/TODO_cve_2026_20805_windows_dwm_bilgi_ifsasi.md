# TODO: CVE-2026-20805 — Windows DWM Bilgi Ifsasi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-20805 |
| **Kritiklik** | Orta |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-20805, Windows Desktop Window Manager (DWM) bileseninde tespit edilen bir bilgi ifsasi (information disclosure) zafiyetidir. Yerel bir saldirgan, uzak bir ALPC (Advanced Local Procedure Call) portu uzerinden DWM surecinin bellek adreslerini (section address) elde ederek Address Space Layout Randomization (ASLR) korumasini etkisiz kilabilmektedir. CVSS skoru 5.5 olan bu zafiyet, CISA tarafindan "Bilinen Somrulen Zafiyetler" (KEV) kataloguna eklenmis olup vahsi ortamda aktif istismar edildigi dogrulanmistir. Tek basina kritik olmasa da, CVE-2026-20822 gibi bir yetki yukseltme zafiyetiyle birlestirildiginde tam saldiri zincirine olanak taniyan bir basamak gorevindedir.
*   **Etkilenen bilesenler:** Windows Desktop Window Manager (dwm.exe), ALPC port altyapisi, Windows 10/11 ve Windows Server 2019/2022/2025 surumleri, DWM uzerinden pencere yonetimi yapan tum Windows masaustu ortamlari

---

### 2. Teknik detay (nasil calisiyor)
*   DWM (Desktop Window Manager), Windows masaustu arayuzunun pencere bilesimini (composition) yoneten ve SYSTEM yetkileriyle calisan kritik bir sistem surectir. DWM, diger sureclere grafik bilgisi aktarmak icin ALPC (Advanced Local Procedure Call) portlari kullanmaktadir.
*   Zafiyet, DWM'nin bir ALPC portuna yanit verirken, paylasilan bellek bolumunun (shared section) temel adresini (base address) icerik olarak dondurmesinden kaynaklanmaktadir. Bu adres normalde gizli tutulmasi gereken dahili bellek duzenini ifsa etmektedir.
*   Saldirgan, bu bellek adresini okuyarak DWM surecinin bellekteki tam konumunu (ASLR offset'ini) ogrenir. Bu bilgi, bellek tabanli istismar tekniklerinin (ROP zincirleri, heap spraying vb.) basarili olabilmesi icin gereken en kritik bilgidir.
*   Zafiyet tek basina kod yurutme saglamaz; ancak CVE-2026-20822 (Windows Graphics EoP) gibi ayri bir bellek bozulma zafiyetiyle birlestirildiginde ASLR atlamali tam saldiri zinciri olusturulabilir.
*   **Neden:** Kok neden, DWM'nin ALPC yanitlarinda bellek bolum adresini yeterince temizlemeden (sanitize) dondurmesidir. ALPC mesajlari uzerinden paylasilan veri yapilarinda, dahili isaret bilgilerinin (pointer) yalnizca yetkili sureclere iletilmesini saglayan erisim kontrolu mekanizmasi bulunmamaktadir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Windows DWM sureci (dwm.exe) uzerinden ALPC portu — hedef sistem masaustu ortaminda calisan herhangi bir Windows 10/11 istemcisi veya Windows Server Desktop Experience kurulumu
*   **2. Normal durum:**
    ```
    1. Bir uygulama (orn. explorer.exe) pencere bilesimi icin DWM'ye ALPC baglantisi kurar
    2. DWM, pencere cizim islemlerini yonetir ve yalnizca grafik icerigini paylasir
    3. Bellek adresleri gibi dahili bilgiler ALPC yanitlarinda maskelenir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    // Kavramsal PoC — DWM ALPC portu uzerinden adres ifsasi
    HANDLE hAlpcPort;
    ALPC_PORT_ATTRIBUTES portAttr = {0};

    // 1. DWM'nin ALPC portuna baglan
    NtAlpcConnectPort(&hAlpcPort, L"\\RPC Control\\DwmApiPort", NULL, &portAttr, ...);

    // 2. Section bilgisi iceren bir ALPC mesaji gonder
    ALPC_MESSAGE msg = {0};
    msg.RequestType = DWM_QUERY_SECTION_INFO;
    NtAlpcSendWaitReceivePort(hAlpcPort, 0, &msg, NULL, &response, ...);

    // 3. Yanittaki section base address degerini oku
    PVOID leakedAddress = response.SectionBaseAddress;
    // Bu adres DWM'nin ASLR offset'ini ifsa eder
    printf("[*] DWM Section Base: 0x%p\n", leakedAddress);

    // 4. Elde edilen adres, ayri bir EoP exploit'inde (CVE-2026-20822) kullanilir
    ```
*   **4. Analiz:** DWM'nin ALPC yanit mesajinda section base address alanini temizlemeden gondermesi, dusuk yetkili bir surece bile DWM bellek duzeni hakkinda bilgi sizdirir. Bu bilgi ASLR'yi bypass etmek icin yeterlidir cunku saldirgan artik bellek adreslerini tahmin etmek yerine dogrudan bilir. Istismar icin yerel erisim yeterlidir; uzaktan tetiklenemez.
*   **5. Kanit:** Saldirgan, salinan ALPC yanitindan DWM surecinin section base adresini elde eder. Bu adres, Windbg veya benzer bir debugger ile dogrulanabildigi gibi, ikinci bir exploit (EoP) icinde ROP gadget adresleri hesaplamak icin kullanilir. Basarili istismar durumunda Security Event Log'da EventID 4656 (ALPC port erisimi) ve EventID 4663 (nesne erisim girisimleri) kayitlari olusur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Orta
*   **CVSS Skoru:** 5.5
*   **Saldiri Yuzeyi:** Yerel erisim (lokal kullanici oturumu gerektirir; internete dogrudan acik degil, ancak uzak masaustu veya ilk erisim sonrasi kullanilabilir)
*   **Karmasiklik:** Dusuk (ALPC portuna baglanmak icin standart Windows API'leri yeterli; kimlik dogrulama veya ozel yetki gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Ocak 2026 Patch Tuesday yamasini tum Windows sistemlerine uygulayin (KB50xxxxx). CISA KEV katalogundaki bu zafiyet icin federal kurumlarda 30 gunluk zorunlu yama suresi bulunmaktadir. DWM sureci icin Exploit Protection ayarlarini etkinlestirin (ForceRelocateImages, DEP, SEHOP).
*   **Orta vadeli:** Yetki yukseltme zincirini kirmak icin ASLR'yi tum kritik sureclerde zorunlu kilin (Set-ProcessMitigation). Standart kullanici hesaplarinin ALPC portlarina erisim yetkilerini kisitlayin. Endpoint Detection and Response (EDR) araclarinda DWM'ye yonelik anormal ALPC baglantilari icin ozel kural tanimlayin. CVE-2026-20822 ile zincirleme riski nedeniyle her iki yamayi birlikte onceliklendirin.
*   **Uzun vadeli:** Masaustu bilesenleri icin saldiri yuzeyini azaltmak amaciyla Windows Server Core (GUI'siz) kurulumlarini tercih edin. Mikro-segmentasyon ile is istasyonlari arasinda lateral movement'i sinirlandirin. Windows Insider ve beta kanallari uzerinden erken yama testlerini otomasyon hattina dahil edin. Bellek guvenligi icin Hardware-enforced Stack Protection (CET) destekli islemcileri yayginlastirin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: DWM sureci varsayilan korumalarla calisiyor ===
# ASLR yalnizca varsayilan (opt-in) modunda — ek zorlama yok
# DWM icin Exploit Protection ayarlari yapilandirilmamis
Get-ProcessMitigation -Name dwm.exe
# Sonuc: ForceRelocateImages = OFF, DEP = OptIn, SEHOP = OFF

# ALPC portlarina erisim kontrolu yok
# Tum yerel kullanicilar DWM ALPC portuna baglanabiliyor
Get-Process -Name dwm | ForEach-Object {
    # Surece bagli handle'lar denetlenmiyor
    $_.HandleCount  # Anormal artislar fark edilmiyor
}
```

**Guvenli kod:**
```powershell
# === GUVENLI: DWM sureci icin Exploit Protection ve izleme ===

# 1. DWM sureci icin zorunlu ASLR ve ek korumalari etkinlestir
Set-ProcessMitigation -Name dwm.exe -Enable ForceRelocateImages, DEP, SEHOP, CFG
Write-Host "[+] dwm.exe icin Exploit Protection etkinlestirildi"

# 2. Sistem genelinde zorunlu ASLR politikasini uygula
Set-ProcessMitigation -System -Enable ForceRelocateImages
Write-Host "[+] Sistem geneli ASLR zorlamasi aktif"

# 3. Yama durumunu dogrula (Ocak 2026 Patch Tuesday)
$patch = Get-HotFix | Where-Object {
    $_.InstalledOn -gt "2026-01-10" -and $_.Description -match "Security Update"
} | Sort-Object InstalledOn -Descending | Select-Object -First 5
if ($patch) {
    Write-Host "[+] Guvenlik yamalari kurulu:" ($patch.HotFixID -join ", ")
} else {
    Write-Warning "[!] Ocak 2026 guvenlik yamasi bulunamadi — acil guncelleme gerekli"
}

# 4. DWM ALPC baglantilari icin izleme kurali
$dwmProc = Get-Process -Name dwm -ErrorAction SilentlyContinue
if ($dwmProc) {
    $handles = $dwmProc.HandleCount
    if ($handles -gt 5000) {
        Write-Warning "[!] DWM handle sayisi anormal: $handles — ALPC istismari olabilir"
    }
}

# 5. Sysmon kurali ile DWM'ye ALPC baglantilari izle
# Sysmon yapilandirmasina ekle:
# <RuleGroup groupRelation="or">
#   <ProcessAccess onmatch="include">
#     <TargetImage condition="is">C:\Windows\System32\dwm.exe</TargetImage>
#   </ProcessAccess>
# </RuleGroup>
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] DWM bileseni calisan tum Windows masaustu ve server (Desktop Experience) sistemlerini envanterle
*   [ ] CISA KEV katalogu ile aktif somuru durumunu dogrula
*   [ ] CVE-2026-20822 ile zincirleme risk haritasini olustur
*   [ ] Mevcut ASLR ve Exploit Protection yapilandirmalarini denetle

**Duzeltme**
*   [ ] Microsoft Ocak 2026 Patch Tuesday yamasini tum etkilenen sistemlere uygula
*   [ ] DWM sureci icin Exploit Protection ayarlarini etkinlestir (ForceRelocateImages, DEP, SEHOP)
*   [ ] Sistem genelinde zorunlu ASLR politikasini devreye al
*   [ ] EDR/Sysmon kurallarini DWM'ye yonelik anormal ALPC erisimlerini tespit edecek sekilde guncelle
*   [ ] Gereksiz masaustu servislerini server ortamlarinda devre disi birak (Server Core'a gecis planla)

**Dogrulama**
*   [ ] Yama sonrasi ALPC portu uzerinden adres ifsasi testini tekrarla
*   [ ] Exploit Protection ayarlarinin aktif oldugunu Get-ProcessMitigation ile dogrula
*   [ ] SIEM/EDR kurallarinin DWM anormal ALPC erisimlerini tespit ettigini dogrula
*   [ ] CVE-2026-20822 ile zincirleme istismar senaryosunu yama sonrasi tekrar test et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Windows Security Event Log'da EventID 4656 (ALPC nesne erisimi) ve EventID 4663 (nesne erisim girisimleri) dwm.exe icin izle
    *   Sysmon EventID 10 (ProcessAccess) ile dwm.exe hedefli erisim girisimlerini tespit et
    *   Regex: `(?i)(dwm\.exe|DwmApiPort|ALPC.*section.*address)`
    *   Regex: `(?i)(NtAlpcConnectPort|NtAlpcSendWaitReceivePort).*dwm`
    *   DWM surec handle sayisindaki ani artislari izle (threshold: >5000)
*   **Anomali Tespiti:**
    *   Standart kullanici hesaplarindan dwm.exe'ye yonelik ProcessAccess olaylari (normal disinda yalnizca csrss.exe ve winlogon.exe baglanir)
    *   DWM surecine ait ALPC portlarinda kisa surede tekrarlayan baglanti girisimleri (brute-force pattern)
    *   CVE-2026-20822 ile iliskili grafik surucusu cokmeleri veya yetki yukseltme girisimlerinin ayni zaman diliminde gozlenmesi (saldiri zinciri belirtisi)
    *   Dusuk yetkili sureclerin DWM bellek alanina okuma erisimi talep etmesi

---

## Notlar
CISA KEV'de aktif somuru dogrulanmis. Saldiri zincirinde CVE-2026-20822 (Windows Graphics EoP) ile birlestirilme potansiyeli var. Microsoft Ocak 2026 Patch Tuesday kapsaminda 114 zafiyet arasinda ele alinmistir. ASLR bypass olarak kendi basina sinirli etkiye sahip olsa da, zincirleme saldiri senaryosunda kritik basamak gorevi gormektedir.
