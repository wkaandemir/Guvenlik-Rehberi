### Güvenlik Açığı: LangChain Core Deserialization Enjeksiyonu (LangGrinch) — CVE-2025-68664

---

### 1. Özet ve etki

*   **Yönetici Özeti:** LangChain Core kütüphanesindeki serileştirme/tersine serileştirme zafiyeti, saldırganın model çıktıları veya kullanıcı girdileri üzerinden kötü amaçlı veri enjekte etmesine imkan verir. Bu durum API anahtarlarının sızdırılması veya kod yürütme ile sonuçlanabilir. Zafiyet yüksek kritik olarak raporlanmıştır.
*   **Etkilenen bileşenler:**
    -   LangChain Core serileştirme bileşenleri
    -   Model çıktısı işleme hattı
    -   Uygulama entegrasyon katmanı

---

### 2. Teknik detay (nasıl çalışıyor)

1. Uygulama, güvenilmeyen veriyi serileştirilmiş nesne olarak kabul eder.
2. Tersine serileştirme sırasında zararlı payload çalıştırılabilir.
3. Saldırgan hassas anahtarları sızdırır veya kod yürütür.

**Neden (kök neden):**  
-   Güvensiz deserialization
-   Model çıktısına/harici girdiye aşırı güven

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** LangChain Core ile nesne işleme hattı
*   **2. Normal istek (meşru):**
```
Şemaya uygun, güvenli JSON çıktısı
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
Zararlı içerik barındıran serileştirilmiş nesne
```
*   **4. Analiz:** Payload, deserialization sırasında yürütülür.; Uygulama içi gizli bilgiler erişilebilir hale gelir.
*   **5. Kanıt:** Uygulamada beklenmeyen komut yürütme izleri; API anahtarlarının/logların sızması

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Uygulama girişleri / model çıktıları
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Kütüphane güncellemelerini uygulayın ve riskli deserialization yollarını kapatın.
*   **Orta vadeli:** Girdi şeması doğrulama ve allowlist yaklaşımı kullanın.
*   **Uzun vadeli:** AI güvenlik denetimleri ve izolasyon (sandbox) politikaları oluşturun.

---

### 6. Örnek düzeltme kodu (Python)
```python
import json

# Örnek: Güvensiz deserialization yerine JSON + şema doğrulama
raw = get_untrusted_input()
obj = json.loads(raw)
validate_schema(obj)  # allowlist tabanlı doğrulama
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] LangChain sürüm ve kullanım envanterini çıkar
*   [ ] Model çıktılarını işleyen bileşenleri belirle
**Düzeltme**
*   [ ] Güncelleme uygula
*   [ ] Deserialization noktalarında şema doğrulama uygula
**Doğrulama**
*   [ ] Hassas anahtarların sızmadığını doğrula
*   [ ] Uygulama loglarında anomali kontrolü yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Deserialization hataları ve istisnaları; AI servislerinden beklenmedik dışa veri akışı
*   **Anomali Tespiti:** Model çıktısının normal dışı formatlarda gelmesi; API anahtarlarının loglarda görünmesi
