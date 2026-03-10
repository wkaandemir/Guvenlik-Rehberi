# TODO: Nike Veri Ihlali — Nike Ic Veri Sizintisi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | Nike Veri Ihlali |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | YENI: Veri_Ihlalleri/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Ocak 2026'da Nike, 1.4 TB buyuklugunde devasa bir ic veri ihlalini arastirmaya basladigini dogrulamistir. Bu sizintinin sirketin global stratejilerini, calisan kisisel verilerini (HR kayitlari), ic yazismalari, fikri mulkiyet belgelerini ve potansiyel olarak tedarik zinciri verilerini icerdigi degerlendirilmektedir. 1.4 TB'lik veri boyutu, tek seferlik bir sizdirmadan ziyade uzun sureli ve asama asama gerceklestirilen bir veri sizdirma operasyonuna (staged exfiltration) veya ic tehdit (insider threat) kaynakli bir olaya isaret etmektedir. Bu olay, buyuk olcekli perakende ve teknoloji sirketlerinin ic tehdit risklerine ve Veri Kaybi Onleme (DLP) eksikliklerine karsi ne kadar savunmasiz olabilecegini gosteren kritik bir ornektir.
*   **Etkilenen bilesenler:** Nike kurumsal sunuculari ve dosya paylasim altyapisi, global strateji dokumanlari, calisan kisisel verileri (HR kayitlari, maas bilgileri), ic yazismalar (e-posta, Teams/Slack), fikri mulkiyet bilgileri (tasarim, patent, urun stratejisi), potansiyel tedarik zinciri verileri (uretici sozlesmeleri, maliyet analizi)

---

### 2. Teknik detay (nasil calisiyor)
*   1.4 TB buyuklugundeki veri sizintisi, tek seferlik bir sizdirma yerine uzun sureli ve asama asama (staged exfiltration) gerceklestirilmis bir operasyona isaret etmektedir. Bu hacimde veriyi kisa surede disari cikarmak ag anomali tespit mekanizmalarini tetikleyeceginden, dusuk hacimli parcali transferler tercih edilmistir.
*   Ihlalin ic tehdit (insider threat) kaynakli olma olasiligi yuksektir: yetkili bir calisan rol geregi eristigi verileri kademeli olarak disari cikarmis olabilir. Alternatif olarak dis saldirganin bir calisanin hesabini ele gecirip ic aga uzun sureli erisim saglamis olmasi da muhtemeldir.
*   Veri siniflandirma politikalarinin yetersizligi veya uygulanmamasi, saldirganin (veya ic tehdidin) HR verileri, strateji dokumanlari ve fikri mulkiyet gibi farkli kategorilerdeki verilere ayirt etmeden ve ek yetkilendirme gerekmeden erisimini kolaylastirmistir.
*   DLP (Veri Kaybi Onleme) kontrollerinin yetersizligi veya yanlis yapilandirilmasi, bu boyutta bir sizdirmanin fark edilmeden gerceklestirilmesine olanak tanimistir. Ozellikle sifrelenmis arsivler, dosya adi degisiklikleri ve kisisel bulut depolama servisleri uzerinden yapilan transferler DLP'yi atlatmis olabilir.
*   **Neden:** Kok neden olarak ic tehdit tespit mekanizmalarinin yetersizligi (UEBA eksikligi), veri siniflandirma ve DLP politikalarinin eksik uygulanmasi, asiri genis erisim izinleri (least privilege prensibinin ihlali), buyuk hacimli veri transferlerinin izlenmemesi ve kullanici davranis analizinin (UBA) yapilmamasi gosterilebilir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Nike kurumsal ag altyapisi, dosya sunuculari, SharePoint/OneDrive alanlari ve HR veritabanlari.
*   **2. Normal durum:**
    ```
    1. Calisan, kurumsal VPN ve SSO ile ic aga baglanir
    2. Rol bazli erisim kontrolu (RBAC) ile yalnizca yetkili verilere erisir
    3. DLP araclari buyuk dosya transferlerini izler ve engeller
    4. SIEM sistemi anormal erisim paternlerini tespit eder
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    Ic tehdit senaryosu:
    1. Yetkili calisan, rol geregi eristigi verileri kademeli olarak yerel
       cihazina kopyalar (gunluk 5-10 GB, tespit esiginin altinda)
    2. Verileri kucuk parcalar halinde (50 MB'lik sifrelenmis arsivler)
       kisisel bulut depolama hizmetine yukler
    3. DLP kontrollerinden kacinmak icin sifrelenmis arsivler, dosya adi
       degisiklikleri ve steganografi kullanir
    4. Uzun bir sure boyunca (haftalar/aylar) dusuk hacimli transferler yapar

    Dis saldirgan senaryosu:
    1. Credential stuffing veya hedefli phishing ile bir calisanin hesabina erisir
    2. VPN uzerinden ic aga baglanir ve yanal hareket yapar (AD enumeration)
    3. Dosya sunuculari ve SharePoint/OneDrive alanlarindaki verilere erisir
    4. Verileri C2 (Command and Control) sunucusuna sifrelenmis kanallarla gonderir
    5. Kalicilik icin ek hesaplar veya zamanlanmis gorevler olusturur
    ```
*   **4. Analiz:** 1.4 TB'lik veri boyutu, tek seferlik degil uzun sureli bir operasyona isaret etmektedir. DLP sistemlerinin buyuk hacimli ancak parcali transferleri tespit edememesi ana acik noktadir. Veri siniflandirmasinin yoklugu veya yetersiz uygulanmasi, farkli kategorilerdeki verilere (HR, strateji, fikri mulkiyet) tek noktadan erisimi kolaylastirmistir. Sifrelenmis arsivlerin icerigi DLP tarafindan analiz edilemedigi icin hassas veri transferi gorece kosusuz gerceklestirilmistir.
*   **5. Kanit:** Nike olayi resmi olarak sorusturmaya basladigini dogrulamistir. 1.4 TB'lik verinin global stratejiler, calisan verileri ve fikri mulkiyet iceriklerini kapsadigi raporlanmistir. Ihlalin tam kapsami devam eden sorusturma sonucunda netlesecektir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** Uygulanamaz (veri ihlali, belirli bir CVE degil)
*   **Saldiri Yuzeyi:** Ic network (ic tehdit veya ele gecirilmis calisan hesabi) ve internete acik (dis saldirgan senaryosu — phishing, credential stuffing)
*   **Karmasiklik:** Orta (ic tehdit icin dusuk — yetkili erisim yeterli; dis saldirgan icin orta-yuksek — hesap ele gecirme ve yanal hareket gerekir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Etkilenen sistemlere erisim izinlerini daraltarak acil kisitlama uygulayin. Anormal veri erisim paternlerini gecmise donuk olarak inceleyin (log analizi, son 6-12 ay). DLP kurallarini buyuk hacimli cikis trafikleri ve sifrelenmis arsiv transferleri icin hemen guclendirim. Tum calisan hesaplarinda cok faktorlu kimlik dogrulamayi (MFA) dogrulayin ve sifre rotasyonu uygulayin.
*   **Orta vadeli:** Kapsamli bir veri siniflandirma programi baslatin (cok gizli, gizli, ic kullanim, genel) ve her sinif icin farkli erisim kontrolleri tanimlayin. UEBA (User and Entity Behavior Analytics) cozumu uygulayin ve ic tehdit tespitini guclendirim. En az yetki prensibini (least privilege) tum erisim kontrol mekanizmalarina uygulayin. DLP araclarini hem ag bazli (network DLP) hem de uc nokta bazli (endpoint DLP) olarak yapilandirin. Sifrelenmis arsivlerin icerik analizini destekleyen DLP cozumlerine gecin.
*   **Uzun vadeli:** Sifir Guven (Zero Trust) mimarisine gecis yapin — her erisim istegi her seferinde dogrulanmali ve yetkilendirilmelidir. Duzenli ic tehdit simulasyonlari ve red team egzersizleri gerceklestirin. Veri erisim politikalarini otomatiklestirin (Just-in-Time erisim — erisim yalnizca gerektiginde verilir ve otomatik olarak geri alinir). Calisan farkindalik egitimlerini ic tehdit senaryolarini kapsayacak sekilde genisletin. Istifa/cikis surecindeki calisanlarin veri erisim paternlerini otomatik olarak izleyen mekanizmalar kurun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```bash
# === GUVENSIZ: Ic tehdit tespiti olmayan yapilandirma ===
# Dosya erisim izlemesi yok
# DLP kurallari yetersiz veya pasif
# /etc/audit/rules.d/ altinda kural yok
# Buyuk transferler loglanmiyor
# Kullanicilar kisisel bulut hizmetlerine sinirsiz erisime sahip
```

**Guvenli kod:**
```bash
# === GUVENLI: Ic tehdit tespiti icin kapsamli dosya erisim izleme ===

# 1. Auditd kurallari (Linux sunucular icin)
# /etc/audit/rules.d/insider-threat.rules
-w /srv/corporate-data/ -p rwa -k corp_data_access
-w /srv/hr-records/ -p rwa -k hr_data_access
-w /srv/strategy-docs/ -p rwa -k strategy_access
-w /srv/ip-designs/ -p rwa -k ip_data_access

# Toplu dosya erisimi tespiti
-a always,exit -F arch=b64 -S open,openat -F dir=/srv/corporate-data/ \
    -F success=1 -k bulk_data_access

# 2. Anormal veri erisim paternlerini tespit et (gunluk analiz scripti)
#!/bin/bash
# /usr/local/bin/insider-threat-check.sh
LOG_FILE="/var/log/audit/audit.log"
THRESHOLD=500  # Gunluk 500'den fazla dosya erisimi supheli
ALERT_EMAIL="security-soc@nike.com"

ausearch -k corp_data_access --start today 2>/dev/null | \
    awk -F'uid=' '{print $2}' | awk '{print $1}' | \
    sort | uniq -c | sort -rn | \
    while read count user; do
        if [ "$count" -gt "$THRESHOLD" ]; then
            echo "[UYARI] Kullanici $user bugun $count hassas dosyaya eristi" | \
                mail -s "[INSIDER THREAT] Anormal Dosya Erisimi" "$ALERT_EMAIL"
            logger -p auth.alert \
                "Insider threat belirtisi: uid=$user erisim=$count"
        fi
    done

# 3. Kisisel bulut depolama servislerine erisimleri engelle
iptables -A OUTPUT -d drive.google.com -j DROP
iptables -A OUTPUT -d dropbox.com -j DROP
iptables -A OUTPUT -d onedrive.live.com -j DROP
iptables -A OUTPUT -d mega.nz -j DROP
```

```powershell
# === GUVENLI: Windows ortami icin ic tehdit tespiti ===

# Gelismis Denetim Ilkesi: Dosya Sistemi Denetimi etkinlestirilmeli
# Secpol.msc > Gelismis Denetim > Nesne Erisimi > Dosya Sistemi Denetimi: Basari

# Anormal dosya erisimlerini raporla
$threshold = 200
$today = (Get-Date).Date
$events = Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    Id        = 4663  # Dosya erisim olayi
    StartTime = $today
} -ErrorAction SilentlyContinue

$suspiciousUsers = $events |
    Group-Object { $_.Properties[1].Value } |
    Where-Object { $_.Count -gt $threshold } |
    Select-Object Name, Count

foreach ($user in $suspiciousUsers) {
    Write-Warning "UYARI: $($user.Name) bugun $($user.Count) dosyaya eristi"
    # SOC ekibine bildirim gonder
    Send-MailMessage -To "soc@nike.com" `
        -Subject "[INSIDER] Anormal dosya erisimi: $($user.Name)" `
        -Body "Kullanici $($user.Name) bugun $($user.Count) dosyaya erismistir." `
        -SmtpServer "smtp.internal"
}

# USB cihazlarina veri transferini engelle (GPO ile)
# Computer Config > Admin Templates > System > Removable Storage Access
# "All Removable Storage classes: Deny all access" = Enabled
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Kurumsal veri siniflandirma ve DLP politikalarini gozden gecir
*   [ ] Mevcut erisim kontrol mekanizmalarinin (RBAC) dogru yapilandirildigini dogrula
*   [ ] UEBA veya ic tehdit tespit cozumunun varligini kontrol et
*   [ ] Calisan erisim loglarini gecmise donuk olarak incele (son 6-12 ay)
*   [ ] Istifa/cikis surecindeki calisanlarin veri erisim paternlerini analiz et

**Duzeltme**
*   [ ] Ic tehdit tespit mekanizmalarini guclendir (UEBA uygulamasi)
*   [ ] Veri erisim loglarini kapsamli olarak toplama ve analiz altyapisi kur
*   [ ] DLP kontrollerini hem ag hem uc nokta bazli olarak yapilandir (sifrelenmis arsiv analizi dahil)
*   [ ] En az yetki prensibini tum erisim kontrol noktalarinda uygula
*   [ ] Kisisel bulut depolama ve dosya paylasim servislerine kurumsal agdan erisimi kisitla
*   [ ] Just-in-Time erisim mekanizmasi uygulayin

**Dogrulama**
*   [ ] DLP kontrollerinin buyuk hacimli ve parcali transferleri tespit edip engelleyebildigini test et
*   [ ] Ic tehdit simulasyonu yaparak tespit mekanizmalarinin etkinligini dogrula
*   [ ] RBAC konfigurasyonunun gereksiz genis erisim izinlerini ortadan kaldirdigini dogrula
*   [ ] Red team egzersizi ile veri sizdirma senaryosunu (staged exfiltration) simule et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Hassas veri dizinlerine gunluk erisim esik degerini asan kullanicilari tespit et
    *   Buyuk hacimli dosya transferlerini (100 MB+ tek seferde, 1 GB+ gunluk toplam) logla ve uyar
    *   Kisisel bulut depolama servislerine erisimleri izle ve engelle
    *   Regex: `(?i)(corp_data_access|hr_data_access|strategy_access|ip_data_access|bulk_data_access)`
    *   Regex: `(?i)(drive\.google\.com|dropbox\.com|onedrive\.live\.com|mega\.nz|wetransfer\.com)`
*   **Anomali Tespiti:**
    *   Tek bir kullanicinin kisa surede normalin cok uzerinde dosyaya erismesi (esik degeri: gunluk 500+)
    *   Mesai saatleri disinda veya tatil gunlerinde hassas veri sunucularina erisim
    *   Istifa/cikis surecindeki calisanlarin veri erisim paternlerinde ani artis
    *   USB cihazlarina buyuk hacimli veri transferi
    *   VPN uzerinden normalin disinda uzun sureli veya yuksek hacimli oturumlar
    *   Sifrelenmis arsiv dosyalarinin (zip, 7z, rar) olusturulmasinda ani artis

---

## Notlar
Buyuk olcekli ic veri sizintisi ornegi. 1.4 TB'lik veri boyutu Nike'in global stratejileri, calisan verileri ve fikri mulkiyetini kapsamaktadir. ESA veri ihlali (700 GB) ile birlikte Ocak 2026'nin en buyuk veri sizintilari arasinda yer almaktadir. Luxshare Precision (Apple montaj ortagi, RansomHouse grubu) ve Ledger tedarik zinciri ihlali ile ayni donemde gerceklesmistir. Ic tehdit (insider threat) ve DLP eksikligi acisindan ders cikarilacak kritik bir vaka.
