# TODO: Volt Typhoon — Kritik Altyapi On Konumlandirma

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | Volt Typhoon |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | YENI: Tedarik_Zinciri/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Volt Typhoon, Cin Halk Cumhuriyeti baglantili devlet destekli bir siber tehdit grubudur (APT). Bu grubun temel amaci veri calmak degil, kritik altyapi sistemlerinde (elektrik, su, ulasim, telekomünikasyon) "uyuyan hucreler" (sleeper cells) olusturarak, olasi bir jeopolitik kriz aninda bu sistemleri felc edecek yikici saldirilari tetikleyebilecek on konumlandirma (pre-positioning) saglamaktir. CISA, FBI, NSA ve Five Eyes istihbarat ortakligi tarafindan yapilin ortak aciklamalar, Volt Typhoon'un ABD ve muttefik ulkelerdeki kritik altyapi aglarinda yillardir gizlice faaliyet gosterdigi uyarisinda bulunmustur. Check Point 2026 Siber Guvenlik Raporu, "dayaniklilik onlemeden daha onemlidir" ilkesini 2026 siber stratejilerinin ana temasi olarak belirlenmistir.
*   **Etkilenen bilesenler:** Perimeter cihazlari (yonlendiriciler, guvenlik duvarlari, VPN cihazlari), SCADA/ICS sistemleri, elektrik dagitim altyapisi, su aritma tesisleri, ulasim yonetim sistemleri, telekomünikasyon altyapisi, kurumsal ag yonlendiricileri ve switch'ler

---

### 2. Teknik detay (nasil calisiyor)
*   Volt Typhoon, geleneksel kotu amacli yazilim (malware) kullanmak yerine, hedef sistemlerde zaten mevcut olan yasal araclari kullanan "living off the land" (LotL) tekniklerini benimser. Bu yaklasim, standart guvenlik araclari tarafindan tespit edilmeyi son derece zorlastirir.
*   Grup, oncelikle internete acik perimeter cihazlarina (Fortinet, Cisco, NETGEAR yonlendiriciler) sizar. Bilinen veya sifirinci gun zafiyetlerini kullanarak bu cihazlarda kalici erisim saglar. Cihaz uzerindeki yasal yonetim araclarini (SSH, WMI, PowerShell) kullanarak ag icerisinde yanal hareket (lateral movement) gerceklestirir.
*   Saldirganlar, ele gecirdikleri perimeter cihazlarda kuçuk yapilandirma degisiklikleri yapar: ek yonetici hesaplari, planlanmamis gorevler (cron jobs) veya tunel konfigurasyonlari ekler. Bu degisiklikler, rutin denetimlerde kolayca gozden kacabilecek kadar incedir.
*   Amac ani bir saldiri degil, uzun vadeli stratejik konumlanmadir. Grup, kritik altyapi aglarinda aylarca hatta yillarca sessizce kalarak, komuta kontrol (C2) altyapisini korur ve kriz aninda aktive edilmeyi bekler.
*   **Neden:** Kok neden, perimeter cihazlarinin guvenlik denetimlerinin yetersizligi, "living off the land" tekniklerinin geleneksel imza tabanli tespit sistemlerini atlatmasi ve kritik altyapi aglarinda segmentasyon eksikligidir. Ayrica, birçok kurumun OT (Operasyonel Teknoloji) aglarinin IT aglarindan yeterince izole edilmemesi, saldirganin BT tarafindan OT sistemlerine gecis yapmasini kolaylastirmaktadir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Kritik altyapi kurumunun internete acik perimeter cihazlari (VPN, yonlendirici, guvenlik duvari)
*   **2. Normal durum:**
    ```
    1. Perimeter cihazi internetten gelen trafigi filtreler
    2. VPN uzerinden yetkili kullanicilar ic aga baglanir
    3. SCADA/ICS sistemleri izole OT aginda calisir
    4. Yonetim arayuzu yalnizca yetkili IP'lerden erisilebilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Volt Typhoon, perimeter cihazinda bilinen bir zafiyeti somurur
       (orn: Fortinet CVE veya Cisco RCE)
    2. Cihaz uzerinde yasal gorunen bir yonetici hesabi olusturur
    3. LotL teknikleriyle (SSH, WMI) ag icerisinde yanal hareket yapar
    4. Kritik sunucularda planlanmamis cron job/scheduled task ekler
    5. SCADA/ICS agina erisim saglayacak tünel konfigurasyonu yapar
    6. C2 altyapisini minimal trafik ile korur (DNS-over-HTTPS, encrypted)
    7. Aylarca sessiz kalir — kriz aninda aktive edilmeyi bekler
    ```
*   **4. Analiz:** LotL teknikleri, standart EDR/IDS imzalarini atlatir cunku kotu amacli yazilim degil, sistemin kendi araclari kullanilir. Perimeter cihazlardaki degisiklikler, firmware guncellemeleri veya yapilandirma yedekleri ile karsilastirma yapilmadigi surece tespit edilemez. OT agi izolasyonu zayifsa, IT agindaki kompromize dogrudan fiziksel surecleri (elektrik, su) etkileyebilir.
*   **5. Kanit:** Birden fazla kritik altyapi kurumunda, yillar once olusturulmus ancak hicbir yasal operasyona ait olmayan yonetici hesaplari, SSH anahtarlari ve tunel konfigurasyonlari tespit edilmistir. CISA KEV katalogu ve FBI aciklamalari bu bulgurlari dogrulamistir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** Uygulanamaz (sistemik APT tehdidi — tek bir zafiyet degil, stratejik kampanya)
*   **Saldiri Yuzeyi:** Internete acik (perimeter cihazlari, VPN, yonetim arayuzleri)
*   **Karmasiklik:** Yuksek (devlet destekli kaynaklarla uzun vadeli operasyon gerektirir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Tum perimeter cihazlarinin yapilandirmalarini bilinen iyi durumlarla (golden config) karsilastirin. Beklenmeyen yonetici hesaplarini, SSH anahtarlarini ve planlanmamis gorevleri arayin. Perimeter cihazlarinin firmware'lerini en son guvenlik surumleriyle guncelleyin. CISA'nin Volt Typhoon IoC listesini tum ag ve endpoint izleme sistemlerine yukleyin.
*   **Orta vadeli:** Sifir Guven (Zero Trust) mimarisine gecis planlayim ve uygulayin. IT ve OT aglarini guclu segmentasyonla birbirinden izole edin. Perimeter cihazlari icin yapilandirma degisiklik izleme (configuration drift detection) uygulayin. Ag trafigi anomali tespiti icin NDR (Network Detection and Response) cozumu dagitinin.
*   **Uzun vadeli:** Kritik altyapi sistemleri icin "assume breach" (ihlal varsayimi) prensibine dayali dayaniklilik planlari gelistirin. Duezenli olarak "threat hunting" (tehdit avi) operasyonlari yurutuun. OT sistemleri icin bagimsiz guvenlik denetimi ve penetrasyon testi sureci olusturun. Ulusal CERT/CSIRT ekipleriyle istihbarat paylasim agina katilim saglayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```bash
# === GUVENSIZ: Perimeter cihazi denetimi yapilmiyor ===
# Yonlendiricide ek hesap kontrolu yok
# Cron job'lar izlenmiyor
# Yapilandirma degisiklikleri karsilastirilmiyor
cat /etc/passwd  # Sadece elle bakilir, otomatik denetim yok
```

**Guvenli kod:**
```bash
#!/bin/bash
# === GUVENLI: Otomatik perimeter cihazi denetim betiği ===

GOLDEN_CONFIG="/opt/security/golden-configs"
ALERT_EMAIL="soc@kurum.gov.tr"

# 1. Beklenmeyen hesaplari tespit et
echo "=== HESAP DENETIMI ==="
UNEXPECTED=$(awk -F: '$3 >= 1000 && $7 != "/sbin/nologin" {print $1}' /etc/passwd \
  | grep -vFf /opt/security/authorized_users.txt)
if [ -n "$UNEXPECTED" ]; then
    echo "UYARI: Beklenmeyen hesaplar: $UNEXPECTED"
    echo "$UNEXPECTED" | mail -s "Volt Typhoon IoC: Beklenmeyen hesap" "$ALERT_EMAIL"
fi

# 2. Planlanmamis cron job'lari ara
echo "=== CRON DENETIMI ==="
for user in $(cut -f1 -d: /etc/passwd); do
    crontab -u "$user" -l 2>/dev/null | grep -v "^#" | grep -v "^$" | while read -r job; do
        if ! grep -qF "$job" /opt/security/authorized_crons.txt; then
            echo "UYARI: Planlanmamis cron [$user]: $job"
        fi
    done
done

# 3. Yapilandirma degisiklik tespiti (drift detection)
echo "=== YAPILANDIRMA DRIFT TESPITI ==="
diff <(cat /etc/network/interfaces) "$GOLDEN_CONFIG/interfaces.golden" || \
    echo "UYARI: Ag yapilandirmasi degismis!"

# 4. Anormal ag baglantilari
echo "=== AG ANOMALILERI ==="
ss -tunap | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort -u | while read -r ip; do
    if ! grep -qF "$ip" /opt/security/known_destinations.txt; then
        echo "UYARI: Bilinmeyen baglanti: $ip"
    fi
done
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum perimeter cihazlarini (yonlendirici, VPN, firewall) envanterle
*   [ ] Her cihazin "golden config" yedegini olustur ve guvenli depolanda sakla
*   [ ] CISA Volt Typhoon IoC listesini indir ve guvenlik araclarina yukle
*   [ ] IT-OT ag segmentasyon durumunu degerlendir

**Duzeltme**
*   [ ] Perimeter cihazlarinda beklenmeyen hesaplari, SSH anahtarlarini ve cron job'lari kaldır
*   [ ] Tum cihaz firmware'lerini en son guvenlik surumleriyle guncelle
*   [ ] IT ve OT aglari arasinda guclu segmentasyon uygula (VLAN, firewall kurallari)
*   [ ] Sifir Guven mimarisine gecis planini baslat
*   [ ] DNS-over-HTTPS ve encrypted C2 traigini tespit edecek NDR kurallari ekle

**Dogrulama**
*   [ ] Yapilandirma drift tespiti (golden config karsilastirmasi) duzenlii calistir
*   [ ] Threat hunting operasyonu ile uyuyan hucre gostergelerini ara
*   [ ] Penetrasyon testi ile perimeter cihazlarin guclendirilmesini dogrula
*   [ ] OT agi izolasyonunun etkinligini test et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Perimeter cihazlarinda yeni hesap olusturma ve SSH anahtar ekleme islemlerini izle
    *   Planlanmamis yapilandirma degisikliklerini tespit et (configuration drift alerts)
    *   LotL arac kullanimini izle: beklenmeyen PowerShell, WMI, SSH oturumlari
    *   Regex: `(?i)(new.user|ssh.key.added|cron.modified|scheduled.task.created)`
    *   Regex: `(?i)(volt.typhoon|living.off.the.land|lotl|pre.positioning)`
    *   CISA KEV IoC adresleriyle ag trafigi eslesmeleri
*   **Anomali Tespiti:**
    *   Perimeter cihazlardan disari dogru dusuk hacimli ama duzenli DNS-over-HTTPS trafigi (C2 belirtisi)
    *   Mesai saatleri disinda yonetim arayuzu erisimleri
    *   IT agindan OT agina dogru beklenmeyen trafik akisi
    *   Uzun suredir kullanilmayan hesaplarla yapilan oturum acma girisimleri
    *   Firmware veya yapilandirma dosyalarinda beklenmeyen degisiklikler

---

## Notlar
"Dayaniklilik onlemeden daha onemlidir" prensibi 2026 siber stratejisinin ana temasi olmustur. Volt Typhoon'un jeopolitik motivasyonu, geleneksel siber suc gruplarindan temel farkidir — amac finansal kazanc degil, stratejik ustunluk ve cakisma aninda yikici kapasite saglamaktir. CISA, FBI ve NSA'nin ortak uyarilari, bu tehdidin onceliklendirmesi gereken en ust duzey risk oldugunu vurgulamaktadir. Check Point 2026 raporu ve WEF Global Cybersecurity Outlook 2026 bu tehdidi ayrintili olarak ele almistir.
