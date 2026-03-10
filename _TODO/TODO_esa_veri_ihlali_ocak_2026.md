# TODO: ESA Veri Ihlali — Avrupa Uzay Ajansi Veri Sizintisi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | ESA Veri Ihlali |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | YENI: Veri_Ihlalleri/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** 7 Ocak 2026'da Avrupa Uzay Ajansi (ESA), 700 GB'tan fazla hassas verinin sizdirildigini resmi olarak dogrulamistir. "Scattered Lapsus$ Hunters" adli tehdit grubu, ESA sunucularina aylar oncesinden sizdiklarini ve SpaceX, Airbus ile Thales Alenia Space gibi buyuk ortaklara ait operasyonel prosedürleri, uzay araci tasarimlarini ve sozlesme detaylarini ele gecirdiklerini iddia etmistir. Bu olay, savunma ve uzay sanayisindeki siber casusluk faaliyetlerinin ne kadar ileri gidebilecegini gosteren stratejik bir veri ihlalidır. Sizdirilan verilerin ulusal guvenlik boyutu tasimasi, uluslararasi ortaklik iliskilerine potansiyel zarari ve uzay teknolojisi fikri mulkiyetinin ele gecirilmesi olayin kritikligini en ust seviyeye cikarmaktadir.
*   **Etkilenen bilesenler:** ESA sunuculari ve ic ag altyapisi, SpaceX/Airbus/Thales Alenia Space ortaklik verileri, uzay araci tasarim dokumanlari, operasyonel prosedurler, sozlesme ve ortaklik detaylari, calisan ve proje bilgileri, potansiyel ulusal guvenlik ve savunma sanayi verileri

---

### 2. Teknik detay (nasil calisiyor)
*   Scattered Lapsus$ Hunters grubu, ESA'nin ic agina aylar oncesinden sizarak kalici erisim (persistent access) saglamistir. Grubun Advanced Persistent Threat (APT) benzeri uzun sureli gizli erisim stratejisi kullandigi degerlendirilmektedir.
*   Ilk erisim vektoru olarak kimlik avci (phishing) kampanyasi, zayif dis servis istismari veya ucuncu taraf tedarikci uzerinden yanal erisim (supply chain pivot) kullanilmis olabilir. Grup, Lapsus$ kolunun bilinen sosyal muhendislik ve SIM swap tekniklerini de kullanmaktadir.
*   Ic agda yanal hareket (lateral movement) yapilarak farkli departmanlara ve ortak kuruluslara ait verilere erisim saglanmistir. Pass-the-hash, credential dumping ve Active Directory istismari gibi teknikler kullanilmis olabilir.
*   700 GB'lik veri boyutu, uzun sureli ve genis kapsamli bir sizdirma operasyonuna isaret etmektedir. Veriler buyuk olasilikla asama asama (staged exfiltration), dusuk hacimli parcalar halinde ve sifrelenmis kanallar uzerinden disari cikarilmistir.
*   **Neden:** Kok neden olarak yetersiz ic ag segmentasyonu (flat network), uzun sureli yetkisiz erisimin tespit edilememesi (yuksek dwell time), veri siniflandirma politikalarinin eksik veya yetersiz uygulanmasi, Veri Kaybi Onleme (DLP) kontrollerinin buyuk hacimli ancak parcali transferleri yakalayamayacak sekilde yapilandirilmasi ve ortak kuruluslarla paylasilan veri alanlarinin yeterince izole edilmemesi gosterilebilir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** ESA sunucu altyapisi, ic ag segmentleri ve ortak kuruluslarin paylasimli veri alanlari.
*   **2. Normal durum:**
    ```
    1. Yetkili personel VPN ve cok faktorlu kimlik dogrulama ile ic aga baglanir
    2. Dosya sunuculari ve proje veritabanlarina rol bazli erisim kontrolu uygulanir
    3. Buyuk hacimli veri transferleri DLP araclari tarafindan izlenir
    4. Ag trafigi SIEM sistemi tarafindan analiz edilir ve anomaliler raporlanir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, hedefli kimlik avci (spear phishing) veya zayif dis servis
       uzerinden ilk erisimi saglar (initial access — MITRE T1566/T1190)
    2. Credential dumping ile ek hesap bilgileri ele gecirilir (T1003)
    3. Yanal hareket ile ic ag segmentlerine yayilir (pass-the-hash, T1550)
    4. Ortak kuruluslara ait dosya sunucularina ve proje havuzlarina erisir
    5. Verileri kucuk parcalar halinde (50-100 MB) sifrelenmis kanallar
       uzerinden (DNS tünelleme, HTTPS exfil) disari cikarir (T1048)
    6. Uzun sureli kalicilik icin backdoor ve gizli hesaplar olusturur (T1136)
    ```
*   **4. Analiz:** Saldirganlar, ic agda yeterli segmentasyon olmamasindan faydalanarak farkli departmanlara ait verilere tek bir erisim noktasindan ulasmistir. DLP kontrollerinin buyuk hacimli ancak parcali (staged) cikis trafiklerini tespit edememis olmasi, veri sizdirma isleminin aylar boyunca fark edilmeden devam etmesine olanak tanimistir. Ortak kuruluslarin (SpaceX, Airbus, Thales) verileriyle ayni segmentte bulunmasi erisim kapsamini dramatik olarak genisletmistir.
*   **5. Kanit:** 700 GB'lik verinin sizdirildigi grup tarafindan kamuya aciklanmistir. SpaceX, Airbus ve Thales Alenia Space'e ait operasyonel prosedurler, uzay araci tasarimlari ve sozlesme detaylarinin ele gecirildigi iddia edilmistir. ESA olayi resmi olarak dogrulamistir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** Uygulanamaz (veri ihlali, belirli bir CVE degil)
*   **Saldiri Yuzeyi:** Internete acik (dis servisler uzerinden ilk erisim) ve ic network (yanal hareket ile genis kapsam)
*   **Karmasiklik:** Yuksek (APT benzeri uzun sureli operasyon, gizli kalma becerisi, coklu teknik gerektiren saldiri zinciri)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Tum ic ag erisim kimlik bilgilerini sifirlayin ve zorunlu sifre rotasyonu uygulayin. Yetkisiz hesaplari ve beklenmeyen erisim noktalarini tespit edip derhal devre disi birakin. Etkilenen ortaklari (SpaceX, Airbus, Thales) resmi olarak bilgilendirin ve ortak incident response sureci baslatin. Kritik veri sunucularina acil erisim kisitlamasi uygulayin. DLP kurallarini buyuk hacimli cikis trafikleri icin hemen guclendirim.
*   **Orta vadeli:** Ic ag segmentasyonunu guclendirin — her departman ve ortak verisi farkli VLAN'larda izole edilmelidir. DLP (Veri Kaybi Onleme) cozumlerini uygulayin ve 100 MB uzerindeki cikis trafiklerini otomatik engelleme/uyari kapsamina alin. Tum hassas verileri siniflandirin (cok gizli, gizli, ic kullanim, genel) ve siniflandirmaya uygun erisim kontrolleri uygulayin. Uzun sureli yetkisiz erisimi tespit edecek "dwell time" izleme mekanizmalari kurun. UEBA (User and Entity Behavior Analytics) cozumu uygulayin.
*   **Uzun vadeli:** Sifir Guven (Zero Trust) mimarisine gecis yapin — her erisim istegi dogrulanmali ve yetkilendirilmelidir. Ortak kuruluslarla guvenlik standartlari (ISO 27001, NIST CSF, ITAR) cercevesinde denetim mutabakatlari imzalayin. Red team egzersizleri ile APT senaryolarini periyodik olarak simule edin. Ulusal ve uluslararasi siber istihbarat paylasim aglarina (CERT-EU, ENISA) aktif katilim saglayin. Veri erisim ve sizdirma senaryolari icin otonom izleme ve cevap (SOAR) platformlari kurun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```bash
# === GUVENSIZ: Yetersiz ag segmentasyonu ve DLP eksikligi ===
# Tum ic ag tek bir duz (flat) network'te — zafiyetli yapilandirma
# /etc/network/interfaces (ornek)
auto eth0
iface eth0 inet static
    address 10.0.0.100/16  # /16 alt agi: tum departmanlar ve ortaklar ayni segmentte

# DLP veya cikis trafik izlemesi yok
# Buyuk dosya transferleri loglanmiyor
# Ortak kuruluslarin verileri ayni ag segmentinde
```

**Guvenli kod:**
```bash
# === GUVENLI: Ag segmentasyonu, DLP ve izleme ile guvenlestirme ===

# 1. VLAN bazli segmentasyon (switch konfigurasyonu)
# Her departman ve ortak verisi ayri VLAN'da
# switch(config)# vlan 100
# switch(config-vlan)# name ESA_Engineering
# switch(config)# vlan 200
# switch(config-vlan)# name Partner_SpaceX
# switch(config)# vlan 300
# switch(config-vlan)# name Partner_Airbus
# switch(config)# vlan 400
# switch(config-vlan)# name Partner_Thales
# Inter-VLAN routing yalnizca acikca izin verilen trafik icin

# 2. Auditd ile hassas veri erisim izleme
# /etc/audit/rules.d/esa-data-protection.rules
-w /opt/classified-data/ -p rwa -k classified_access
-w /srv/partner-data/spacex/ -p rwa -k partner_spacex_access
-w /srv/partner-data/airbus/ -p rwa -k partner_airbus_access
-w /srv/partner-data/thales/ -p rwa -k partner_thales_access
-w /etc/ssh/sshd_config -p wa -k ssh_config_change

# 3. DLP: Buyuk dosya transferlerini izle ve engelle
iptables -A OUTPUT -p tcp --dport 443 -m connbytes \
    --connbytes 104857600: --connbytes-dir both \
    --connbytes-mode bytes \
    -j LOG --log-prefix "LARGE_TRANSFER: "

iptables -A OUTPUT -p tcp --dport 443 -m connbytes \
    --connbytes 524288000: --connbytes-dir both \
    --connbytes-mode bytes \
    -j DROP  # 500MB ustu transferleri engelle

# 4. Yetkisiz hesap tespiti (gunluk cron job)
#!/bin/bash
EXPECTED_USERS="/etc/security/authorized_users.list"
for user in $(awk -F: '$3 >= 1000 {print $1}' /etc/passwd); do
    if ! grep -qx "$user" "$EXPECTED_USERS"; then
        echo "[KRITIK] Beklenmeyen kullanici hesabi: $user"
        logger -p auth.crit "Yetkisiz hesap tespit edildi: $user"
    fi
done

# 5. DNS tunelleme tespiti (anormal DNS trafigi)
# /etc/suricata/rules/local.rules
# alert dns any any -> any any (msg:"DNS Tunnel Suspect"; \
#   dns.query; content:"."; offset:50; sid:1000001; rev:1;)
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Savunma ve uzay sektorundeki veri siniflandirma politikalarini gozden gecir
*   [ ] Tum ic ag erisim noktalarini ve kullanici hesaplarini envanterle
*   [ ] Ortak kuruluslarin (SpaceX, Airbus, Thales) bilgilendirilmesini sagla
*   [ ] Mevcut DLP ve SIEM cozumlerinin etkinligini degerlendir
*   [ ] Dwell time tespit kapasitesini analiz et (ortalama tespit suresi ne kadar?)

**Duzeltme**
*   [ ] Tum erisim kimlik bilgilerini sifirla ve cok faktorlu kimlik dogrulamayi zorunlu kil
*   [ ] Ic ag segmentasyonunu uygula (departman ve ortak bazli VLAN izolasyonu)
*   [ ] DLP kontrollerini buyuk hacimli ve parcali cikis trafikleri icin aktif hale getir
*   [ ] Hassas verileri siniflandirma politikasina gore etiketle ve erisim kisitla
*   [ ] Beklenmeyen hesaplari ve backdoor'lari tespit edip kaldir
*   [ ] DNS tunelleme ve sifrelenmis kanal uzerinden veri sizdirma tespiti icin kurallar ekle

**Dogrulama**
*   [ ] Yetkisiz erisim noktalarini penetrasyon testi ile dogrula
*   [ ] DLP kontrollerinin parcali (staged) buyuk transferleri engelleyip engellemedigini test et
*   [ ] Ic ag segmentasyonunun yanal hareketi onleyip onlemedigini dogrula
*   [ ] Red team egzersizi ile APT senaryosunu simule et
*   [ ] SIEM uyarilarinin anormal veri cikislarini tespit edip etmedigini test et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Buyuk hacimli cikis trafiklerini izle (100 MB+ tek seferde veya 500 MB+ gunluk toplam)
    *   Hassas dosya sunucularina mesai saatleri disinda erisimleri tespit et
    *   Beklenmeyen hesap olusturmalarini ve yetki degisikliklerini izle
    *   Regex: `(?i)(LARGE_TRANSFER|classified_access|partner_.*_access|unauthorized.*account)`
    *   Regex: `(?i)(useradd|adduser|net\s+user\s+\/add|backdoor|persistence)`
*   **Anomali Tespiti:**
    *   Normal disinda yuksek hacimli disari yonlu (egress) ag trafigi
    *   Tek bir kullanicinin farkli departman VLAN'larina kisa surede erisim denemesi (yanal hareket belirtisi)
    *   Mesai saatleri disinda veya tatil gunlerinde hassas dosya sunucularina erisim
    *   Yeni olusturulmus hesaplardan yonetici duzeyi islemler
    *   Sifrelenmis kanal uzerinden uzun sureli dusuk hacimli surekli veri transferi (staged exfiltration belirtisi)
    *   Anormal DNS sorgu hacimleri veya uzunluklari (DNS tunelleme belirtisi)

---

## Notlar
Stratejik siber casusluk ornegi. Scattered Lapsus$ Hunters grubunun ESA'ya aylarca fark edilmeden sizmayi basarmasi, "dwell time" sorununu ve ic tehdit tespit mekanizmalarinin yetersizligini gostermektedir. Savunma ve uzay sanayisinin hedef alinmasina kritik bir ornek. Nike veri ihlali (1.4 TB) ve Ledger tedarik zinciri ihlali ile birlikte Ocak 2026'nin en buyuk veri sizintilari arasinda yer almaktadir. MITRE ATT&CK frameworku ile haritalama: Initial Access (T1566), Credential Access (T1003), Lateral Movement (T1550), Exfiltration (T1048), Persistence (T1136).
