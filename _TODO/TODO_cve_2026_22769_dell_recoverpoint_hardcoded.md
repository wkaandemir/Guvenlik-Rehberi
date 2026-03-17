# TODO: CVE-2026-22769 — Dell RecoverPoint Sabit Kodlanmis Kimlik Bilgileri

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-22769 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [Vibecoding_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Vibecoding_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-22769, Dell RecoverPoint for Virtual Machines urununde sabit kodlanmis (hardcoded) kimlik bilgilerinin bulunmasindan kaynaklanan kritik bir zafiyettir. Bu acik, saldirganin RecoverPoint yonetim arayuzune yetkisiz erisim saglamasina, kurumsal yedekleme ve felaket kurtarma (disaster recovery) sistemlerini kontrol etmesine ve yedekleme verilerini manipule etmesine veya yok etmesine olanak tanir. Yedekleme altyapisi bir organizasyonun son savunma hatti oldugu icin bu zafiyetin istismari, fidye yazilimi senaryolarinda veri kurtarma seceneminin ortadan kaldirilmasina neden olabilir. Subat 2026 guvenlik raporlarinda Dell RecoverPoint zafiyeti, kurumsal altyapi bilesenlerindeki en tehlikeli aciklar arasinda siniflandirilmistir.
*   **Etkilenen bilesenler:** Dell RecoverPoint for Virtual Machines (tum zafiyetli surumler), RecoverPoint yonetim konsolu ve API, kurumsal yedekleme ve felaket kurtarma altyapisi, RecoverPoint ile entegre VMware vSphere ortamlari, yedekleme veritabanlari ve replikasyon kanallari

---

### 2. Teknik detay (nasil calisiyor)
*   Dell RecoverPoint for Virtual Machines, kurulumu sirasinda yonetim islevleri icin sabit kodlanmis (hardcoded) bir servis hesabi ve sifre cifti icermektedir. Bu kimlik bilgileri firmware icerisine gomulmustur ve varsayilan yapilandirmada degistirilemez veya devre disi birakilamaz.
*   Saldirgan, sabit kodlanmis kimlik bilgilerini kullanarak RecoverPoint yonetim arayuzune (web konsolu veya SSH) dogrudan erisim saglayabilir. Bu erisim, yedekleme politikalarini degistirme, replikasyonu durdurma, yedekleme verilerini silme veya manipule etme yetkisi verir.
*   Sabit kodlanmis kimlik bilgileri, urunun farkli kurulumlarinda ayni oldugundan, tek bir kurulumdan elde edilen bilgiler tum RecoverPoint ortamlarinda kullanilabilir.
*   Bu zafiyet ozellikle fidye yazilimi saldirilarinda kritik hale gelmektedir: saldirgan once yedekleme altyapisini devre disi birakir, ardindan uretim verilerini sifrelediginde organizasyonun kurtarma secenegi kalmaz.
*   **Neden:** Kok neden, Dell'in RecoverPoint firmware'ine gelistirme veya bakim amacli sabit kodlanmis kimlik bilgileri gommes ve bu bilgilerin uretim ortaminda devre disi birakilmasini saglayan bir mekanizma sunmamasidir. CWE-798 (Use of Hard-Coded Credentials) kategorisindedir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Dell RecoverPoint for Virtual Machines yonetim arayuzu (web konsolu, SSH, REST API).
*   **2. Normal durum:**
    ```
    1. Yedekleme yoneticisi, kisisel kimlik bilgileriyle RecoverPoint konsoluna baglanir
    2. Yedekleme politikalari ve replikasyon gorevleri belirlenir
    3. Yalnizca yetkili kullanicilar yedekleme verilerine erisir
    4. Erisim loglanir ve denetlenir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```bash
    # Saldirgan, sabit kodlanmis servis hesabi bilgilerini kullanir:
    # (Bu bilgiler firmware analizi veya kamuya acik kaynaklardan elde edilebilir)

    # 1. SSH ile RecoverPoint yonetim arayuzune baglanma
    ssh hardcoded_service@recoverpoint.internal.corp -p 22
    # Sifre: [firmware'den elde edilen sabit sifre]

    # 2. Yedekleme politikalarini listeleme
    # GET /api/v1/backup-policies HTTP/1.1
    # Authorization: Basic [base64(hardcoded_service:password)]

    # 3. Replikasyonu durdurma ve yedekleri silme
    # DELETE /api/v1/replications/all
    # DELETE /api/v1/snapshots/all

    # 4. Fidye yazilimi senaryosu: Yedekleme yok edildikten sonra
    #    uretim verilerini sifreleme (kurtarma secenegi kalmiyor)
    ```
*   **4. Analiz:** Sabit kodlanmis kimlik bilgileri tum RecoverPoint kurulumlarinda aynidir. Saldirganin ag erisimi varsa (ozellikle ic agdan), kimlik bilgilerini kullanarak dogrudan yonetim yetkisi kazanir. Yedekleme altyapisi genellikle ag segmentasyonunda ihmal edilen bir alan oldugu icin erisim kolaydir.
*   **5. Kanit:** Saldirgan RecoverPoint yonetim konsoluna tam erisim saglar. Yedekleme politikalarini listeleyebilir, degistirebilir veya tum yedekleme verilerini yok edebilir. SSH ve API erisim loglarinda hardcoded servis hesabi adiyla yetkisiz oturumlar gorunur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Ic network (RecoverPoint yonetim arayuzune ag erisimi gereken her nokta) ve potansiyel olarak internete acik (yanlis yapilandirilmis ortamlarda)
*   **Karmasiklik:** Dusuk (sabit kodlanmis kimlik bilgileri kamuya aciklandiginda dogrudan kullanilabilir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Dell'in yayimladigi guvenlik yamasini derhal uygulayin. RecoverPoint yonetim arayuzune ag erisimini yalnizca yetkili yonetim VLAN'iyla sinirlayin. Varsayilan ve sabit kodlanmis kimlik bilgilerini degistirin (yama bunu mumkun kiliyorsa). SSH ve web konsolu erisimini IP beyaz listesi ile kisitlayin. Yedekleme sistemlerinin bagimsiz, cevrimdisi (air-gapped) kopyalarini olusturun.
*   **Orta vadeli:** RecoverPoint yonetim arayuzunu ag segmentasyonu ile izole edin (ayri yonetim VLAN'i). Cok faktorlu kimlik dogrulama (MFA) uygulayin. Yedekleme altyapisini duzeli penetrasyon testine dahil edin. Tum RecoverPoint erisimlerini merkezi SIEM'e yonlendirin. Hardcoded credential taramasi icin detect-secrets veya benzeri araclari CI/CD pipeline'ina entegre edin.
*   **Uzun vadeli:** Yedekleme altyapisi icin Sifir Guven (Zero Trust) mimarisine gecin — her erisim istegi dogrulanmali ve yetkilendirilmelidir. 3-2-1 yedekleme stratejisini uygulayin (3 kopya, 2 farkli medya, 1 cevrimdisi). Tedarikci guvenlik denetimi sureclerini olgunlastirin ve firmware analizi iceren guvenlik degerlendirmeleri zorunlu kilin. Yedekleme sistemleri icin degismezlik (immutability) ozelligi olan cozumlere gecis yapin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```bash
# === GUVENSIZ: Sabit kodlanmis kimlik bilgileri ile RecoverPoint erisimi ===
# Dell RecoverPoint firmware'inde gomulu servis hesabi
# /opt/recoverpoint/config/service.conf (orneksel)
SERVICE_USER="rp_service"
SERVICE_PASS="Dell@RP2025!Static"  # Tum kurulumlarda ayni!

# SSH erisimi kisitlanmamis:
# /etc/ssh/sshd_config
PermitRootLogin yes
PasswordAuthentication yes
# AllowUsers kisitlamasi yok — herkes baglanabilir
```

**Guvenli kod:**
```bash
# === GUVENLI: Sabit kimlik bilgilerini kaldir, erisimi sertlestir ===

# 1. Dell guvenlik yamasini uygula (hardcoded hesabi devre disi birakir)
# https://www.dell.com/support/security adresinden kontrol et
# Yama sonrasi servis hesabi otomatik devre disi birakilir

# 2. SSH erisimini sertlestir (/etc/ssh/sshd_config):
PermitRootLogin no
PasswordAuthentication no              # Yalnizca anahtar tabanli
AllowUsers rpadmin@10.0.100.0/24       # Yalnizca yonetim VLAN'indan
MaxAuthTries 3
LoginGraceTime 30

# 3. Ag segmentasyonu: RecoverPoint'u ayri yonetim VLAN'ina tasi
# switch(config)# vlan 500
# switch(config-vlan)# name Backup_Mgmt
# switch(config)# interface vlan 500
# switch(config-if)# ip access-group BACKUP_ACL in

# 4. Hardcoded credential taramasi (CI/CD entegrasyonu)
pip install detect-secrets
detect-secrets scan --all-files /opt/recoverpoint/ --baseline .secrets.baseline
detect-secrets audit .secrets.baseline

# 5. Yedekleme butunluk kontrolu (gunluk cron job):
#!/bin/bash
# /usr/local/bin/backup-integrity-check.sh
BACKUP_HASH_FILE="/var/log/recoverpoint/backup_hashes.sha256"
ALERT_EMAIL="security@corp.internal"

# Yedekleme dosyalarinin hash'lerini dogrula
if ! sha256sum -c "$BACKUP_HASH_FILE" --quiet 2>/dev/null; then
    echo "KRITIK: Yedekleme butunlugu ihlal edildi!" | \
        mail -s "[ALERT] RecoverPoint Backup Integrity Failure" "$ALERT_EMAIL"
    logger -p auth.crit "RecoverPoint yedekleme butunlugu ihlali tespit edildi"
fi
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Dell RecoverPoint for Virtual Machines kurulumlarini tum ortamlarda envanterle
*   [ ] Mevcut RecoverPoint surumlerini kontrol et ve zafiyetli surumleri belirle
*   [ ] Varsayilan ve sabit kodlanmis kimlik bilgilerinin degistirilip degistirilmedigini dogrula
*   [ ] RecoverPoint yonetim arayuzune ag erisimi olan tum noktalari haritala

**Duzeltme**
*   [ ] Dell guvenlik yamasini tum RecoverPoint kurulumlarinda uygula
*   [ ] Sabit kodlanmis servis hesabini devre disi birak veya sifresini degistir
*   [ ] SSH erisimini yalnizca anahtar tabanli ve IP kisitlamali olarak yapilandir
*   [ ] RecoverPoint yonetim arayuzunu ayri VLAN ile ag segmentasyonuna al
*   [ ] Cok faktorlu kimlik dogrulama (MFA) uygulayin
*   [ ] Cevrimdisi (air-gapped) yedekleme kopyalari olusturun

**Dogrulama**
*   [ ] Sabit kodlanmis kimlik bilgileriyle erisim testini yama sonrasi tekrarla (engellenmeli)
*   [ ] Ag segmentasyonunun yetkisiz erisimi onledigini penetrasyon testi ile dogrula
*   [ ] Yedekleme butunluk kontrollerinin duzgun calistigini test et
*   [ ] SIEM kurallarinin RecoverPoint'a yetkisiz erisim girisimlerini tespit ettigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   RecoverPoint yonetim arayuzune basarisiz ve basarili giris denemelerini izle
    *   Sabit kodlanmis servis hesabi adiyla yapilan tum erisim girisimlerini kritik uyari olarak isaretleyin
    *   Yedekleme politikasi degisikliklerini ve replikasyon durdurma islemlerini logla
    *   Regex: `(?i)(rp_service|recoverpoint.*login|hardcoded.*credential|service.*account.*auth)`
    *   Regex: `(?i)(backup.*delete|replication.*stop|snapshot.*remove|policy.*change)`
*   **Anomali Tespiti:**
    *   Mesai saatleri disinda RecoverPoint yonetim konsoluna erisim
    *   Bilinmeyen IP adreslerinden RecoverPoint SSH veya API erisimi
    *   Yedekleme politikalarinda veya replikasyon ayarlarinda beklenmeyen degisiklikler
    *   Kisa surede birden fazla yedekleme silme veya durdurma islemi
    *   RecoverPoint'tan dis aga beklenmeyen veri transferi (yedekleme verisi sizdirma belirtisi)

---

## Notlar
Hardcoded credentials (CWE-798) kurumsal yedekleme sistemleri icin en tehlikeli zafiyet turlerinden biridir. Yedekleme altyapisi bir organizasyonun son savunma hatti oldugu icin, fidye yazilimi gruplarinin ilk hedefi yedekleme sistemlerini etkisiz hale getirmektir. Dell RecoverPoint zafiyeti, vibecoding platformlarinin kurumsal aglara entegrasyonu sirasinda da ciddi veri sizintilarina zemin hazirlayabilir. Subat 2026 guvenlik raporlarinda Cisco Catalyst SD-WAN Manager (CVE-2026-20127) ile birlikte kurumsal altyapi bilesenlerindeki en kritik aciklar arasinda yer almistir.
