# TODO: CVE-2026-21513 — MSHTML Framework Guvenlik Ozelligi Bypass

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21513 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21513, Windows isletim sisteminde bulunan MSHTML (Trident) rendering motorunda tespit edilen bir guvenlik ozelligi atlatma (security feature bypass) zafiyetidir. CVSS skoru 8.8 olan bu acik, Subat 2026 Patch Tuesday'de yamalanan alti sifir gun zafiyetinden biri olarak aktif istismar edilmektedir. Saldirganlar, ozel olarak hazirlanan icerikler araciligiyla Windows guvenlik uyarilarini (SmartScreen, Mark of the Web — MotW) devre disi birakarak kurbanlarin sistemlerinde kotu amacli kod yurutebilmektedir. MSHTML motoru, Internet Explorer'in kaldirilmasina ragmen Windows'un pek cok bileseninde (Office, Outlook, Microsoft 365 uygulamalari) gomulu olarak kullanilmaya devam ettigi icin etkisi genis bir yelpazeyi kapsamaktadir.
*   **Etkilenen bilesenler:** Windows 10/11 tum surumler, Windows Server 2016/2019/2022/2025, MSHTML (Trident) rendering motoru, Internet Explorer uyumluluk modu, Microsoft Office ve Outlook gomulu HTML yorumlayicilari, Edge IE modu

---

### 2. Teknik detay (nasil calisiyor)
*   MSHTML (Trident), Internet Explorer'in temelini olusturan HTML rendering motorudur. IE'nin kaldirilmasina ragmen Windows, OLE nesneleri, ActiveX bilesenleri ve gomulu HTML iceriklerini islemek icin bu motoru kullanmaya devam etmektedir.
*   Zafiyet, MSHTML'nin URL semalari ve dosya yollari uzerindeki guvenlik kararlarini verirken guvensiz girdilere asiri guvenmesinden kaynaklanan bir mantik hatasidir. Ozellikle, Mark of the Web (MotW) etiketlerinin ve SmartScreen kontrollerinin belirli dosya turleri veya URL semalari icin atlanabilmesi mumkun hale gelmektedir.
*   Saldirganlar, kurbani ozel hazirlanmis bir web sayfasi, e-posta eki veya Office dosyasi uzerinden hedef alabilmektedir. Icerik acildiginda MSHTML motoru devreye girer ve guvenlik uyarilari goruntulenmeden kotu amacli kod yurutulur.
*   Bu zafiyet, CVE-2026-21510 (Windows Shell bypass) ile birlikte zincirlendiginde daha etkili bir saldiri yuzeyine sahip olmaktadir; her iki acik da Subat 2026 Patch Tuesday'de aktif istismar durumunda yamalandi.
*   **Neden:** Kok neden, MSHTML motorunun URL semasi ve dosya yolu dogrulamasindaki yetersiz girdi sanitasyonudur. Guvenlik kararlari (MotW etiketleme, SmartScreen kontrolu) verirken kullanici kontrolleri girdilerin yeterince dogrulanmamasi, bypass olanagi yaratmaktadir. Ayrica MSHTML'nin eski bir mimari uzerine insa edilmis olmasi ve modern guvenlik standartlarina tam uyumlu olamamasi bu tur zafiyetlerin tekrarlamasina neden olmaktadir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** MSHTML motoru uzerinden HTML icerik isleyen Windows sistemi (orn. Outlook veya Office).
*   **2. Normal durum:**
    ```
    1. Kullanici, internetten indirilen bir dosyayi acar
    2. Windows, dosyanin Mark of the Web (MotW) etiketini kontrol eder
    3. SmartScreen uyarisi goruntulenir: "Bu dosya guvenilmeyen bir kaynaktan geldi"
    4. Kullanici uyariyi kabul ederse dosya acilir; etmezse islem iptal edilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, ozel hazirlanmis bir HTML dosyasi olusturur
    2. Dosya icinde MSHTML motorunun URL semasi dogrulamasini
       atlatan bir yol manipulasyonu yer alir (orn. ozel karakterler
       veya alternatif protokol semalariyla MotW kontrolunun atlanmasi)
    3. Dosya, Outlook e-posta eki veya web sayfasi baglantisi
       olarak kurbana gonderilir
    4. Kurban dosyayi actiginda MSHTML motoru devreye girer
    5. MotW etiketleri ve SmartScreen uyarilari goruntulenmez
    6. Gomulu ActiveX veya script icerigi dogrudan yurutulur
    ```
*   **4. Analiz:** MSHTML motoru, dosyanin kaynagini ve guvenlik durumunu degerlendirirken URL semasi veya dosya yolu uzerindeki manipulasyonu tespit edemez. Bu nedenle MotW etiketi atlanir ve SmartScreen kontrolu devre disi kalir. Saldirgan, bu guvenlik atlatmasini kullanarak kurbanlarin sistemlerinde rastgele kod yurutebilir.
*   **5. Kanit:** Basarili istismar durumunda sistem uzerinde saldirganin secimi dogrultusunda kod yurutulur. Olay gunluklerinde MSHTML'nin bir HTML icerik isleme olayina ait kayit bulunur ancak SmartScreen engelleme kaydi olusmaz. Kurbanin hicbir guvenlik uyarisi gormemesi saldirinin sessizce gerceklestigini gosterir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 8.8
*   **Saldiri Yuzeyi:** Internete acik (e-posta ekleri, web sayfalari, Office dosyalari)
*   **Karmasiklik:** Dusuk (kullanicinin dosyayi acmasi yeterli; Onizleme Bolmesi uzerinden tetiklenmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft Subat 2026 Patch Tuesday guvenlik guncellemesini tum etkilenen sistemlere hemen uygulayin. Internet Explorer uyumluluk modunu Edge tarayicisinda devre disi birakin. MSHTML ActiveX kontrollerini kayit defteri uzerinden engelleyin. E-posta guvenlik aglarinda (gateway) HTML iceren ekleri tarama kurallarini sikilaistirin.
*   **Orta vadeli:** Grup Ilkesi (Group Policy) ile MSHTML/IE modunu kurumsal duzeyde devre disi birakin. Office belgelerindeki gomulu HTML iceriklerinin calistirilmasini kisitlayan makro ve ActiveX politikalari uygulayin. Kullanicilara MSHTML bypass temelli phishing saldirilari hakkinda farkindalik egitimi verin. Attack Surface Reduction (ASR) kurallarini etkinlestirin.
*   **Uzun vadeli:** MSHTML motoruna bagli eski uygulamalari modern alternatiflerle degistirin. Sifir Guven (Zero Trust) mimarisi cercevesinde tum dosya erisimlerini dogrulayin. Uygulama beyaz listeleme (Application Whitelisting) mekanizmalarini devreye alin. Microsoft'un MSHTML'yi tamamen kullanim disi birakacagi guncellemeleri yakindan takip edin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli durum:**
```powershell
# MSHTML / Internet Explorer modu aktif — varsayilan yapilandirma
# IE modu acik: saldirganlar MSHTML uzerinden MotW bypass yapabilir
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" `
    -Name "InternetExplorerIntegrationLevel" -ErrorAction SilentlyContinue
# Deger: 1 (IE modu aktif) veya mevcut degil (varsayilan: acik)

# ActiveX kontrolleri engellenmemis
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\3" `
    -Name "1200" -ErrorAction SilentlyContinue
# Deger: 0 (ActiveX: Etkin) — bypass icin kullanilabilir
```

**Guvenli yapilandirma:**
```powershell
# 1. Subat 2026 Patch Tuesday yamasini dogrula
$patch = Get-HotFix | Where-Object {
    $_.Description -match "Security" -and $_.InstalledOn -gt "2026-02-01"
}
if (-not $patch) {
    Write-Warning "KRITIK: Subat 2026 guvenlik yamasi eksik! Hemen uygulayin."
    # Install-WindowsUpdate -KBArticleID "KB5XXXXXX" -AcceptAll -AutoReboot
}

# 2. Internet Explorer modunu devre disi birak (Edge Group Policy)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" `
    -Name "InternetExplorerIntegrationLevel" -Value 0 -Type DWord

# 3. MSHTML ActiveX kontrollerini engelle (Internet bolgesinde)
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\3" `
    -Name "1200" -Value 3 -Type DWord   # ActiveX: Devre disi
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\3" `
    -Name "2000" -Value 3 -Type DWord   # ActiveX: İndirme devre disi
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\3" `
    -Name "1201" -Value 3 -Type DWord   # Script icin guvenli olmayan ActiveX: Devre disi

# 4. Attack Surface Reduction (ASR) kurallarini etkinlestir
# MSHTML/IE uzerinden cocuk surec olusturulmasini engelle
Set-MpPreference -AttackSurfaceReductionRules_Ids d4f940ab-401b-4efc-aadc-ad5f3c50688a `
    -AttackSurfaceReductionRules_Actions Enabled

# 5. Office'te MSHTML icerik calistirmayi kisitla
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Office\16.0\Common\Security" `
    -Name "DisableAllActiveX" -Value 1 -Type DWord
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Kurumsal ortamda MSHTML/IE modu kullanan sistemleri envanterleyin
*   [ ] Internet Explorer uyumluluk modunun hangi uygulamalarda aktif oldugunu belirleyin
*   [ ] Mevcut ASR kurallarinin durumunu kontrol edin
*   [ ] E-posta guvenlik aglarindaki HTML ek tarama politikalarini gozden gecirin

**Duzeltme**
*   [ ] Microsoft Subat 2026 Patch Tuesday guvenlik guncellemesini uygulayin
*   [ ] Edge tarayicisinda IE modunu Group Policy ile devre disi birakin
*   [ ] MSHTML ActiveX kontrollerini kayit defteri uzerinden engelleyin
*   [ ] ASR kurallarini MSHTML/IE uzerinden surec olusturulmasini engelleyecek sekilde yapilandirin
*   [ ] Office uygulamalarinda ActiveX calistirmayi devre disi birakin

**Dogrulama**
*   [ ] MSHTML bypass senaryosunu kontrolllu ortamda test edin ve yamarin engelledigini dogrulayin
*   [ ] Yama durumunu tum etkilenen sistemlerde dogrulayin (Get-HotFix)
*   [ ] IE modu devre disi iken uyumluluk gerektiren uygulamalarin calismaya devam ettigini dogrulayin
*   [ ] ASR kurallarinin dogru callistigini olay gunluklerinden dogrulayin

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Windows Olay Gunlugu'nde MSHTML/Trident motoru tarafindan tetiklenen surec olusturma olaylarini izleyin
    *   SmartScreen atlatma girisimlerini tespit edin
    *   Regex: `(?i)(mshtml|trident|iexplore).*?(bypass|security\s*feature|smartscreen|motw)`
    *   Ek Regex (ActiveX tetikleme): `(?i)clsid.*?(activex|ole|embed).*?(internet|restricted)`
    *   Windows Defender Application Guard olaylarini izleyin: Event ID 1116, 1117
*   **Anomali Tespiti:**
    *   MSHTML motoru uzerinden beklenmeyen dosya turlerinin acilmasi
    *   MotW etiketi olmayan internetten indirilen dosyalarin calistirilmasi
    *   Office uygulamalarindan MSHTML araciligiyla cocuk surec (cmd.exe, powershell.exe) olusturulmasi
    *   IE modu devre disi iken MSHTML render olaylarinin tetiklenmesi
    *   Kisa surede cok sayida kullanicida MSHTML ile iliskili dosya acma olaylari (yaygin phishing gostergesi)

---

## Notlar
Aktif istismar durumunda. Subat 2026 Patch Tuesday'de yamalanan 6 sifir gununden biridir. CVE-2026-21510 (Windows Shell bypass) ile birlikte kullanilabilir. MSHTML motoru, IE'nin kaldirilmasina ragmen Windows ekosisteminde hala genis kullanim alanina sahiptir; bu nedenle uzun vadede MSHTML bagimliliginin azaltilmasi stratejik bir hedeftir. Vibecoding ortamlarinda calisan AI ajanlarinin dis kaynaklardan veri cekme senaryolarinda da MSHTML bypass riski bulunmaktadir.
