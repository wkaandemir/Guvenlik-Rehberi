# TODO: CVE-2025-66516 — Apache Tika XXE Enjeksiyonu

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-66516 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | OWASP_Top_10/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-66516, Apache Tika icerik analiz kutuphanesinde PDF dosyalari uzerinden XML External Entity (XXE) enjeksiyonuna izin veren kritik bir zafiyettir. CVSS skoru 10.0 ile en yuksek dereceye sahip olan bu acik, Oracle Fusion Middleware, PeopleSoft, kurumsal icerik yonetim sistemleri ve Apache Tika'yi gomulu olarak kullanan onlarca farkli urunu dogrudan etkilemektedir. Tek bir kutuphanedeki hatanin onlarca farkli urunu savunmasiz birakmasina "CVE Amplifikasyonu" adi verilmektedir ve bu durum yamalama surecini oldukca karmasiklastirmaktadir. Saldirganlar bu zafiyet araciligiyla sunucu tarafinda hassas dosyalari okuyabilir, dahili ag kaynaklarina erisebilir (SSRF) ve hizmet reddi (DoS) saldirisi gerceklestirebilir.
*   **Etkilenen bilesenler:** Apache Tika kutuphanesi (zafiyetli surumler), Oracle Fusion Middleware, Oracle PeopleSoft, Oracle Communications urun ailesi, kurumsal icerik yonetim sistemleri (ECM), Apache Tika'yi bagimlilk olarak kullanan tum Java uygulamalari

---

### 2. Teknik detay (nasil calisiyor)
*   Apache Tika, PDF dosyalarini islerken icerideki gomulu XML yapilarini (XMP metadata, form verileri vb.) bir XML parser araciligiyla analiz etmektedir. Varsayilan yapilandirmada bu XML parser'i, dis varliklara (external entities) referans verilmesine izin vermektedir.
*   Saldirgan, ozel olarak hazirlanmis bir PDF dosyasinin icerisine zarali bir DTD (Document Type Definition) ve XXE payload'u yerlestirmektedir. Bu payload, sunucu dosya sistemindeki dosyalari (`file:///etc/passwd`) veya dahili ag kaynaklarini (`http://internal-server/`) hedef alabilir.
*   Apache Tika bu PDF'i parse ettiginde, gomulu XML icindeki dis varlik referanslarini cozumler ve iceriklerini yanit olarak dondurur veya sunucu tarafinda isler.
*   Bu zafiyet "CVE Amplifikasyonu" kavrami icin tipik bir ornektir: Apache Tika, Oracle ekosisteminde onlarca farkli urune gomulu olarak geldigi icin tek bir kutuphane hatasi genis bir urun yelpazesini savunmasiz birakmaktadir.
*   **Neden:** Kok neden, Apache Tika'nin PDF icerisindeki XML yapilarini islerken XML parser'in varsayilan olarak dis varlik cozumlemeyi (external entity resolution) etkin birakmasidir. Guvenli varsayilan yapilandirmalarin uygulanmamasi ve ucuncu taraf bagimliliklardaki guvenlik yapilandirmalarinin denetlenmemesi bu zafiyeti olusturmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Apache Tika kutuphanesi uzerinden PDF dosya isleme servisi (orn. Oracle Fusion Middleware belge analiz bilesenler veya kurumsal icerik yonetim sistemi)
*   **2. Normal durum:**
    ```
    1. Kullanici bir PDF dosyasini sisteme yukler
    2. Apache Tika, PDF icerisindeki metni ve metadata'yi cikarir
    3. Cikartilan icerik uygulamaya dondurulur (arama indeksi, onizleme vb.)
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```xml
    <!-- PDF icerisine gomulmus zararli XMP metadata blogu -->
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE foo [
      <!ENTITY xxe SYSTEM "file:///etc/passwd">
    ]>
    <x:xmpmeta xmlns:x="adobe:ns:meta/">
      <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
        <rdf:Description>
          <dc:description>&xxe;</dc:description>
        </rdf:Description>
      </rdf:RDF>
    </x:xmpmeta>
    <!-- Saldirgan bu PDF'i sisteme yukler -->
    <!-- Tika, XMP metadata'yi parse ederken &xxe; varligini cozumler -->
    <!-- /etc/passwd icerigi metadata sonucu olarak doner -->
    ```
*   **4. Analiz:** Apache Tika'nin XML parser'i, DTD icerisinde tanimlanan dis varligi (`file:///etc/passwd`) cozumler ve icerigini `dc:description` alanina yerlestirir. Parser'da `FEATURE_SECURE_PROCESSING` veya `disallow-doctype-decl` ozellikleri varsayilan olarak aktif olmadigindan, dis varlik referanslari engellenmez. SSRF varyantinda `SYSTEM` URI'si `http://internal-server:8080/admin` gibi bir dahili adres olabilir.
*   **5. Kanit:** Saldirgan, PDF yukleme sonrasi donen yanit veya metadata ciktisindan sunucu dosya sistemi icerigini okuyabilir. Loglarda asiri buyuk XML islemleri, dis aga veya dahili servislere beklenmeyen HTTP iskeleri gorunur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 10.0
*   **Saldiri Yuzeyi:** Internete acik (PDF dosya yukleme arayuzleri, icerik yonetim sistemleri, belge isleme servisleri)
*   **Karmasiklik:** Dusuk (ozel hazirlanmis bir PDF dosyasi yeterli, kimlik dogrulama gerekli olmayabilir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Apache Tika'yi en son yamali surume guncelleyin (3.1.0+). Tum XML parser'larinda dis varlik cozumlemeyi devre disi birakin. PDF yukleme arayuzlerine dosya boyutu ve tur kisitlamalari ekleyin. Oracle urunlerinde Apache Tika bagimliligi iceren bilesenlerin yamasini oncelikli uygulayin.
*   **Orta vadeli:** Kurumsal SBOM (Software Bill of Materials) envanteri olusturun ve Apache Tika'yi gomulu olarak kullanan tum urunleri tespit edin. Oracle Critical Patch Update (CPU) Ocak 2026 yamalarini tum ilgili urun ailelerine uygulayin. Icerik isleme servislerini ag segmentasyonu ile izole edin. WAF kurallarindan XXE payload kaliplarini filtreleyin.
*   **Uzun vadeli:** Ucuncu taraf bagimliliklarin guvenlik surumlerini otomatik izleme (dependency-check, Renovate, Dependabot) sureclerini kurumsallastirin. Guvenli varsayilan XML parser yapilandirmalarini zorunlu kilin (tum projelerde). Tedarik zinciri guvenlik denetimleri icin SBOM tabanli otomasyon uygulayin. CVE Amplifikasyonu riskini azaltmak icin monolitik bagimliliklari modularize edin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```java
// === GUVENSIZ: Varsayilan Apache Tika ayarlari ile XXE'ye acik ===
import org.apache.tika.Tika;
import org.apache.tika.metadata.Metadata;
import java.io.InputStream;

public class DocumentParser {
    public String parseDocument(InputStream pdfStream) throws Exception {
        // Varsayilan Tika — XML parser dis varliklara acik
        Tika tika = new Tika();
        Metadata metadata = new Metadata();
        String content = tika.parseToString(pdfStream, metadata);
        return content; // XXE payload'u burada cozumlenir!
    }
}
```

**Guvenli kod:**
```java
// === GUVENLI: XXE korumasini etkinlestir ve parser'i sertlestir ===
import org.apache.tika.Tika;
import org.apache.tika.metadata.Metadata;
import org.apache.tika.parser.AutoDetectParser;
import org.apache.tika.parser.ParseContext;
import org.apache.tika.sax.BodyContentHandler;
import javax.xml.parsers.SAXParserFactory;
import java.io.InputStream;

public class DocumentParser {
    public String parseDocument(InputStream pdfStream) throws Exception {
        // 1. SAXParserFactory ile XXE korumasini etkinlestir
        SAXParserFactory spf = SAXParserFactory.newInstance();
        spf.setFeature("http://xml.org/sax/features/external-general-entities", false);
        spf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
        spf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
        spf.setFeature("http://javax.xml.XMLConstants/feature/secure-processing", true);

        // 2. ParseContext'e guvenli parser'i ekle
        ParseContext context = new ParseContext();
        context.set(SAXParserFactory.class, spf);

        // 3. Guvenli parser ile belgeyi isle
        AutoDetectParser parser = new AutoDetectParser();
        BodyContentHandler handler = new BodyContentHandler(-1);
        Metadata metadata = new Metadata();
        parser.parse(pdfStream, handler, metadata, context);

        return handler.toString();
    }
}
// Maven bagimliligi: Apache Tika 3.1.0+ (yamali surum)
// <dependency>
//   <groupId>org.apache.tika</groupId>
//   <artifactId>tika-core</artifactId>
//   <version>3.1.0</version>
// </dependency>
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Apache Tika kullanan tum uygulamalari ve uruntleri envanterle (Oracle urunleri dahil)
*   [ ] SBOM (Yazilim Malzeme Listesi) kontrolu yap ve Tika surumlerini tespit et
*   [ ] PDF yukleme arayuzlerini ve icerik isleme servislerini haritalandir
*   [ ] Oracle Critical Patch Update (CPU) Ocak 2026 bulteni ile etkilenen uruntleri karsilastir

**Duzeltme**
*   [ ] Apache Tika'yi guvenli surume guncelle (3.1.0+)
*   [ ] Tum bagli Oracle urunlerini Ocak 2026 CPU yamalariyla guncelle
*   [ ] XML parser'larinda dis varlik cozumlemeyi devre disi birak (disallow-doctype-decl)
*   [ ] Icerik isleme servislerini ag segmentasyonu ile izole et
*   [ ] WAF kurallarini XXE payload kaliplarini filtreleyecek sekilde guncelle

**Dogrulama**
*   [ ] PDF uzerinden XXE enjeksiyonu testini yama sonrasi tekrarla
*   [ ] Oracle urunlerinde yama durumunu ve surumleri dogrula
*   [ ] SBOM tabanli bagimlilk taramasi ile tum Tika surumlerinin guncel oldugunu kontrol et
*   [ ] SIEM kurallarinin XXE girisimlerini tespit ettigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   PDF yukleme servislerinde asiri buyuk XML islemlerini izle
    *   Apache Tika log'larinda dis varlik cozumleme hatalari ve SSRF belirtilerini tespit et
    *   Regex: `(?i)(<!DOCTYPE|<!ENTITY|SYSTEM\s+["']file:|SYSTEM\s+["']http)`
    *   Regex: `(?i)(external.entity|xxe|dtd.processing|entity.resolution)`
    *   Oracle urun loglarinda beklenmeyen dosya sistemi erisimlerini izle
*   **Anomali Tespiti:**
    *   PDF yukleme sonrasi sunucudan dahili ag adreslerine HTTP istekleri (SSRF belirtisi)
    *   Icerik isleme servisinin `/etc/passwd`, `/etc/shadow` gibi sistem dosyalarina erisim denemesi
    *   Normal disinda buyuk metadata yanitlari (veri sizdirma belirtisi)
    *   Tek kullanicidan kisa surede cok sayida PDF yukleme girisimleri
    *   Apache Tika islemlerinde CPU/bellek tuketiminin ani artmasi (billion laughs/DoS belirtisi)

---

## Notlar
Tek kutuphanedeki hata onlarca urunu etkiler (CVE Amplifikasyonu). SBOM kullanimi ve ucuncu taraf bagimlilk yonetimi kritik oneme sahiptir. Oracle Ocak 2026 CPU kapsaminda 337 yeni guvenlik yamasi yayinlanmis olup buyuk bir kismi Apache Tika gibi gomulu ucuncu taraf kutuphanelerden kaynaklanmaktadir. Iliskili Oracle urun aileleri: Fusion Middleware, Communications, Financial Services, PeopleSoft.
