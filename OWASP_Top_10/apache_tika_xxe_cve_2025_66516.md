### Güvenlik Açığı: Apache Tika XXE Zafiyeti — CVE-2025-66516

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Apache Tika'daki XXE zafiyeti, PDF gibi dosyalar üzerinden dış varlıkların işlenmesine ve dosya okuma/SSRF gibi etkilere yol açar. Oracle Fusion Middleware ve PeopleSoft gibi ürünlerde tedarik zinciri etkisiyle geniş kapsamlı risk doğurur.
*   **Etkilenen bileşenler:**
    -   Apache Tika
    -   XML parser konfigürasyonu
    -   Oracle Fusion Middleware / PeopleSoft gibi tüketici ürünler

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan, dış varlık içeren bir PDF/XML payload üretir.
2. Tika içindeki parser dış varlıkları çözer.
3. Sistem içi dosya okuma veya SSRF gerçekleşir.

**Neden (kök neden):**  
-   External entity çözümlemesinin açık olması
-   Güvenilmeyen dosya içeriğinin işlenmesi

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Tika kullanan dosya işleme hattı
*   **2. Normal istek (meşru):**
```
Standart PDF yükleme/işleme
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
XXE içeren PDF/XML içerik
```
*   **4. Analiz:** Parser dış varlığı çözer ve hedef kaynağa erişir.; Saldırgan hassas veri okur veya iç ağa istek gönderir.
*   **5. Kanıt:** Dosya işleme loglarında dış kaynak erişimi; Sunucu tarafında beklenmeyen outbound istekler

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Dosya yükleme/işleme
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Tika ve tüketici ürün güncellemelerini uygulayın.
*   **Orta vadeli:** Dosya işleme akışında dış varlık çözümlemesini kapatın.
*   **Uzun vadeli:** Tedarik zinciri bağımlılıklarını sürekli tarayın ve SBOM yönetin.

---

### 6. Örnek düzeltme kodu (Java)
```java
// Örnek: XXE koruması (genel şablon)
XMLInputFactory xif = XMLInputFactory.newInstance();
xif.setProperty(XMLInputFactory.IS_SUPPORTING_EXTERNAL_ENTITIES, false);
xif.setProperty(XMLInputFactory.SUPPORT_DTD, false);
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Tika kullanan uygulamaları belirle
*   [ ] Dosya yükleme noktalarını envanterle
**Düzeltme**
*   [ ] Güncellemeleri uygula
*   [ ] Dış varlık çözümlemesini kapat
**Doğrulama**
*   [ ] XXE test payloadlarıyla doğrulama yap
*   [ ] Outbound istekleri loglarla kontrol et

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Dosya işleme servislerinde dış ağ erişimi; Parser hataları ve DTD uyarıları
*   **Anomali Tespiti:** İç ağ adreslerine beklenmeyen bağlantılar; Yükleme sonrası anormal gecikme/çıktı
