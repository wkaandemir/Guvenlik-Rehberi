# TODO: CVE-2026-24858 — FortiOS SSO Bypass

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-24858 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-24858, FortiOS, FortiManager ve FortiAnalyzer urunlerinde bulunan ve FortiCloud Single Sign-On (SSO) mekanizmasini bypass eden kritik bir kimlik dogrulama atlama zafiyetidir. CVSS skoru 9.4 olan bu zafiyet, bir saldirganin kendi FortiCloud hesabi ve kayitli cihazi uzerinden, SSO ozelligi aktif olan diger hesaplara bagli FortiGate firewall cihazlarina yetkisiz erisim saglamasina olanak tanimaktadir. 20 Ocak 2026'da bircoK kurumda FortiGate cihazlarinda kendiliinden olusan yerel yonetici hesaplari tespit edilmis ve Fortinet 26 Ocak'ta SSO servislerini gecici olarak durdurmak zorunda kalmistir. Aktif istismar edilen bu zafiyet, kurumsal ag guvenlik altyapisinin temelini olusturan firewall cihazlarinin kontrolunu kaybetme riski tasimaktadir.
*   **Etkilenen bilesenler:** FortiOS (tum SSO etkin surumler), FortiManager, FortiAnalyzer, FortiCloud SSO altyapisi, FortiGate firewall cihazlari, FortiAP kablosuz erisim noktalari (FortiCloud yonetimli), FortiSwitch (FortiCloud yonetimli)

---

### 2. Teknik detay (nasil calisiyor)
*   FortiCloud SSO, Fortinet'in bulut tabanli merkezi yonetim platformudur. Yoneticiler, tek bir FortiCloud hesabiyla birden fazla FortiGate, FortiManager ve FortiAnalyzer cihazini yonetebilir. SSO mekanizmasi, OAuth 2.0 tabanli bir kimlik dogrulama akisi kullanmaktadir.
*   Zafiyet, FortiCloud SSO dogrulama surecindeki bir mantik hatasindan kaynaklanmaktadir. SSO token dogrulama asamasinda, FortiCloud'un cihaz-hesap bagini (device-to-account binding) dogrulamak icin kullandigi "organizasyon kimlik kontrolu" (organization ID validation) yetersiz kalmaktadir.
*   Saldirgan, kendi gecerli FortiCloud hesabini olusturur ve bir FortiGate cihazini bu hesaba kaydeder. SSO dogrulama akisinda, saldirgan kendi cihazina ait token'in organizasyon kimligini (org_id) hedef kurumun organizasyon kimligiyle degistirerek veya token dogrulama adimini atlayarak hedef kurumun cihazlarina erisim saglar.
*   Basarili istismar sonrasinda saldirgan, hedef kurumun FortiGate cihazlarinda otomatik olarak yerel yonetici hesabi olusturabilir, firewall kurallarini degistirebilir, VPN yapilandirmalarini manipule edebilir ve ag trafigini izleyebilir.
*   **Neden:** Kok neden, FortiCloud SSO token dogrulama mantigi icerisinde organizasyon kimliginin (org_id) istemci tarafindan degistirilebilir bir parametre olarak gonderilmesi ve sunucu tarafinda bu degerin token imzasiyla kriptografik olarak baglanmamis olmasidir. Token dogrulama islemi, organizasyon kimligini bagimsiz olarak kontrol etmek yerine, istemcinin gonderdigi degere guvenmektedir (CWE-807: Reliance on Untrusted Inputs in a Security Decision).

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** FortiCloud SSO etkin FortiGate firewall cihazlari — saldirganin kendi FortiCloud hesabi ve en az bir kayitli cihazi gerektirir
*   **2. Normal durum:**
    ```
    1. Yonetici, FortiCloud SSO ile giris yapar
    2. FortiCloud, yoneticinin hesabina bagli cihazlari listeler
    3. Yonetici, yalnizca kendi organizasyonuna ait cihazlara erisebilir
    4. SSO token, organizasyon kimligi ile kriptografik olarak imzalanmistir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    # Kavramsal PoC — FortiCloud SSO organization ID bypass

    # 1. Saldirgan kendi FortiCloud hesabini olusturur
    # Hesap: attacker@evil.com
    # Organizasyon ID: ORG-ATTACKER-12345

    # 2. Kendi FortiGate cihazini bu hesaba kaydeder
    # Cihaz seri no: FGT60F-ATTACKER-001

    # 3. FortiCloud SSO giris akisini baslatir ve token alir
    # POST https://login.forticloud.com/api/v1/auth/token
    # Response: { "access_token": "eyJ...", "org_id": "ORG-ATTACKER-12345" }

    # 4. Token'daki org_id'yi hedef kurumun organizasyon kimligi ile degistirir
    # Manipule edilmis istek:
    # POST https://forticloud.com/api/v1/device/manage
    # Headers:
    #   Authorization: Bearer eyJ...
    #   X-FortiCloud-OrgId: ORG-TARGET-67890  <-- hedef kurumun org_id'si
    #
    # FortiCloud, org_id'yi token imzasiyla karsilastirmadan kabul eder

    # 5. Saldirgan, hedef kurumun FortiGate cihazlarinda yonetici erisimi kazanir
    # Otomatik yerel admin hesabi olusturulur:
    # config system admin
    #     edit "fcloud_sso_attacker"
    #     set accprofile "super_admin"
    #     set trusthost1 0.0.0.0/0
    #     next
    # end
    ```
*   **4. Analiz:** FortiCloud SSO dogrulama akisinda, access_token ve org_id ayri parametreler olarak iletilmektedir. Token imzasi (JWT signature) yalnizca kullanici kimligini ve cihaz bilgisini kapsamakta, organizasyon kimligini icermemektedir. Bu nedenle saldirgan, gecerli bir token ile farkli bir organizasyonun cihazlarina erisim talep ettiginde, FortiCloud sunucusu talebi gecerli olarak kabul eder. Ayrica, SSO uzerinden cihaza ilk erisim saglandiginda otomatik yerel admin hesabi olusturma mekanizmasi, saldirganin kalici erisim (persistence) kazanmasini kolaylastirir.
*   **5. Kanit:** Hedef FortiGate cihazinda beklenmeyen yerel yonetici hesaplari gorunur (`config system admin` listesinde). FortiCloud SSO loglarinda farkli organizasyon kimliklerinden gelen erisim istekleri tespit edilebilir. FortiGate event_log'da admin login olaylarinda bilinmeyen FortiCloud kullanicilari gorunur. 20 Ocak 2026'da birden fazla kurumda bu belirtiler gozlemlenmistir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 9.4
*   **Saldiri Yuzeyi:** Internete acik (FortiCloud SSO bulut servisi uzerinden; saldirganin yalnizca kendi FortiCloud hesabi ve internete erisimi yeterli)
*   **Karmasiklik:** Dusuk (gecerli bir FortiCloud hesabi olusturmak ucretsiz ve kolay; org_id manipulasyonu basit parametre degisikligi)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Fortinet tarafindan yayinlanan yamayi tum FortiOS, FortiManager ve FortiAnalyzer cihazlarina DERHAL uygulayin. Yama uygulanana kadar FortiCloud SSO'yu devre disi birakin ve yerel kimlik dogrulama kullananin. Tum FortiGate cihazlarinda beklenmeyen yerel yonetici hesaplarini kontrol edin ve temizleyin. Mevcut yonetici hesap parolalarini degistirin. FortiGate event_log'larini son 30 gun icin inceleyin.
*   **Orta vadeli:** FortiCloud SSO yerine yerel RADIUS/TACACS+ tabanli merkezi kimlik dogrulamaya gecin. Tum yonetici hesaplari icin Multi-Factor Authentication (MFA) zorunlulugu uygulayin. FortiGate yonetim arayuzune erisimi yalnizca yonetim VLAN'indan izin verin. Admin hesap degisikliklerini otomatik izleyen Fortinet FortiSIEM veya SIEM kurallari tanimlayin. FortiManager uzerinden merkezi yapilandirma yedekleme ve degisiklik izleme sureclerini etkinlestirin.
*   **Uzun vadeli:** Fortinet cihaz yazilim guncelleme sureclerini otomasyon hattina dahil edin (FortiManager ile otomatik firmware dagitimi). Zero Trust mimarisi ile ag yonetim katmanini izole edin — yonetim trafigi ayrı bir ag segmentinde tasinmali. Fortinet Security Fabric icerisinde anomali tespit mekanizmalarini etkinlestirin. Firmware ve yapilandirma butunluk kontrolu icin otomatik audit sureclerini olusturun. Tedarikci guvenlik bultenlerini otomatik izleyen ve onceliklendiren bir surec yapilandirin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```
# === GUVENSIZ: FortiGate — FortiCloud SSO varsayilan yapilandirma ===
# FortiCloud SSO etkin — kimlik dogrulama bypass'a acik
config system fortiguard
    set forticloud-sso enable
end

# Yonetim arayuzu tum ag'dan erisilebilir
config system interface
    edit "mgmt"
        set allowaccess https ssh ping
        set ip 192.168.1.1/24
        # Yonetim erisimi kisitlanmamis
    next
end

# Admin hesap izleme yok
# Beklenmeyen hesap olusumul fark edilmez
config system admin
    get
    # Sonuc listesinde saldirganin olusturdugu hesap gorunmeyebilir
end
```

**Guvenli kod:**
```
# === GUVENLI: FortiGate — SSO bypass'a karsi sertlestirme ===

# 1. FortiCloud SSO'yu devre disi birak (yama oncesi acil onlem)
config system fortiguard
    set forticloud-sso disable
end

# 2. Yerel admin kimlik dogrulamayi RADIUS/TACACS+ ile guclendirt
config user radius
    edit "corp-radius"
        set server "10.0.1.50"
        set secret <sifrelenmis-paylasilan-sifre>
        set auth-type auto
    next
end

config system admin
    edit "admin"
        set accprofile "super_admin"
        set trusthost1 10.0.100.0/24
        set two-factor enable
        set fortitoken <token-seri-no>
    next
end

# 3. Yonetim erisimini yalnizca yonetim VLAN'indan izin ver
config system interface
    edit "mgmt"
        set allowaccess https ssh
        set ip 10.0.100.1/24
        set trusted-hosts 10.0.100.0/24
    next
end

# 4. Beklenmeyen admin hesaplarini kontrol et ve temizle
config system admin
    get
    # Bilinmeyen hesaplari sil:
    # delete <bilinmeyen-hesap-adi>
end

# 5. Admin hesap degisikliklerini izleme icin alert yapilandir
config alertemail setting
    set admin-login-logs enable
    set configuration-changes-logs enable
    set mailto1 "secops@example.com"
    set filter-mode category
end

config log setting
    set fwpolicy-implicit-log enable
    set local-in-allow enable
    set local-in-deny-unicast enable
    set local-in-deny-broadcast enable
end

# 6. Yama sonrasi SSO'yu guvenli sekilde yeniden etkinlestir
# Yalnizca Fortinet yamasi uygulandiktan sonra:
# config system fortiguard
#     set forticloud-sso enable
# end
# execute forticloud-sso renew-certificate
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] FortiCloud SSO etkin tum FortiGate, FortiManager ve FortiAnalyzer cihazlarini envanterle
*   [ ] Tum cihazlarda beklenmeyen yerel yonetici hesaplarini kontrol et
*   [ ] FortiOS firmware surumlerini mevcut yama durumuyla karsilastir
*   [ ] FortiGate event_log'larini son 30 gun icin SSO erisim kayitlari acisindan incele
*   [ ] FortiCloud hesap aktivitelerini Fortinet destek ekibiyle dogrula

**Duzeltme**
*   [ ] Fortinet tarafindan yayinlanan yamayi tum etkilenen cihazlara DERHAL uygula
*   [ ] Yama oncesi: FortiCloud SSO'yu tum cihazlarda gecici olarak devre disi birak
*   [ ] Beklenmeyen yerel yonetici hesaplarini tespit et ve sil
*   [ ] Tum mevcut yonetici hesaplarinin parolalarini degistir
*   [ ] Yonetim arayuzu erisimini yalnizca yonetim VLAN'indan izin verecek sekilde kisitla
*   [ ] MFA zorunlulugunu tum yonetici hesaplari icin uygula
*   [ ] Admin hesap degisikliklerini izleyen SIEM/alert kurallarini tanimla

**Dogrulama**
*   [ ] SSO bypass testini yama sonrasi tekrarla (farkli org_id ile erisim denemesi)
*   [ ] Temizlenen cihazlarda yeni beklenmeyen hesap olusup olusmaadigini 7 gun boyunca izle
*   [ ] Yonetim erisim kisitlamalarinin dogru calistigini test et
*   [ ] MFA'nin tum yonetici girislerinde zorunlu oldugunu dogrula
*   [ ] SIEM kurallarinin admin hesap degisikliklerini tespit ettigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   FortiGate event_log'da beklenmeyen admin login olaylarini izle (type=event subtype=system level=information)
    *   FortiGate config_log'da admin hesap olusturma/degistirme olaylarini izle
    *   FortiCloud API loglarinda farkli organizasyon kimliklerinden gelen erisim isteklerini tespit et
    *   Regex: `(?i)(admin.*login.*forticloud|system.*admin.*created|SSO.*authentication.*bypass)`
    *   Regex: `(?i)(fcloud_sso.*admin|unexpected.*local.*admin|org_id.*mismatch)`
    *   FortiManager'da merkezi yapilandirma degisiklik loglarini izle
*   **Anomali Tespiti:**
    *   FortiGate cihazlarinda yeni yerel yonetici hesaplarinin otomatik olusturulmasi
    *   Bilinmeyen FortiCloud hesaplarindan gelen SSO giris girisimleri
    *   Firewall kurallarinda veya VPN yapilandirmalarinda beklenmeyen degisiklikler
    *   Yonetim arayuzune (HTTPS/SSH) bilinen yonetim VLAN'i disindaki adreslerden erisim girisimleri
    *   FortiGate cihazlarinda sertifika yenileme veya SSO yapilandirma degisiklikleri (persistence belirtisi)
    *   Ayni zaman diliminde birden fazla FortiGate cihazinda es zamanli admin hesap degisiklikleri (toplu istismar belirtisi)

---

## Notlar
20 Ocak 2026'da birden fazla kurumda FortiGate cihazlarinda otomatik yerel yonetici hesaplari olusumu gorulmustur. 26 Ocak'ta Fortinet, SSO servislerini gecici olarak durdurmus ve yalnizca yamali cihazlar icin bir gun sonra tekrar aktif hale getirmistir. CVSS 9.4 ile Ocak 2026'nin en yuksek etkili zafiyetlerinden biridir. Fortinet ekosistemindeki bu tur kimlik dogrulama bypass zafiyetleri, ag guvenlik altyapisinin guven koku olarak kabul edilen firewall cihazlarinin kontrolunun kaybedilmesi anlamina gelmektedir. Iliskili tehdit aktorleri: devlet destekli gruplar ve ransomware operatorleri Fortinet cihazlarini birincil hedef olarak belirlemektedir.
