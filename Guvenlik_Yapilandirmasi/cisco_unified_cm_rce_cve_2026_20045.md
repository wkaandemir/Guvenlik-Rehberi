### Güvenlik Açığı: Cisco Unified CM Uzaktan Kod Yürütme — CVE-2026-20045

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Cisco Unified Communications Manager üzerinde kimlik doğrulaması gerektirmeden uzaktan komut yürütmeye izin veren kritik bir zafiyettir. Web tabanlı yönetim arayüzüne gönderilen özel HTTP istekleri işletim sistemi seviyesinde komut çalıştırılmasına yol açabilir. Zafiyetin aktif olarak sömürüldüğü bildirilmiştir.
*   **Etkilenen bileşenler:**
    -   Cisco Unified Communications Manager (Unified CM)
    -   Web tabanlı yönetim arayüzü
    -   HTTP istek işleme katmanı

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan, yönetim arayüzüne özel hazırlanmış HTTP isteği gönderir.
2. Girdi doğrulaması yetersiz olduğu için komut enjeksiyonu oluşur.
3. Komutlar sistem seviyesinde çalıştırılır ve yetki yükseltilir.

**Neden (kök neden):**  
-   Yetersiz input validation (CWE-94 benzeri)
-   Yönetim arayüzünün internete açık olması

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Unified CM web yönetim arayüzü
*   **2. Normal istek (meşru):**
```
Yönetim arayüzüne standart HTTP isteği
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
Komut içeren manipüle HTTP isteği
```
*   **4. Analiz:** İstek sunucu tarafında işlenirken komut satırı çalışır.; Saldırgan sistemde kalıcılık sağlayabilir.
*   **5. Kanıt:** Web yönetim loglarında olağandışı istekler; Sistem seviyesinde beklenmeyen komut çalıştırma izleri

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Ağa açık yönetim arayüzü
*   **Karmaşıklık:** Düşük-Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Cisco yamalarını derhal uygulayın ve yönetim arayüzünü sınırlandırın.
*   **Orta vadeli:** Yönetim arayüzünü yalnızca yönetim ağına taşıyın, WAF/ACL uygulayın.
*   **Uzun vadeli:** Ağ segmentasyonu ve güvenli yönetim erişim politikaları oluşturun.

---

### 6. Örnek düzeltme kodu (Metin (Şablon))
```text
# Örnek (şablon):
# 1) Ürün sürümünü doğrula
# 2) Yönetim arayüzünü yalnızca yönetim ağından erişilebilir kıl
# 3) Üretici yamasını uygula
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Unified CM envanterini çıkar
*   [ ] Yönetim arayüzünün internet erişimini kontrol et
**Düzeltme**
*   [ ] Yamaları uygula
*   [ ] ACL/WAF ile yönetim erişimini kısıtla
**Doğrulama**
*   [ ] Sürüm doğrulaması yap
*   [ ] Yönetim arayüzü erişim denemelerini loglarla izle

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Yönetim arayüzüne başarısız/olağandışı HTTP istekleri; Beklenmeyen sistem komut yürütme logları
*   **Anomali Tespiti:** Kısa sürede çoklu yönetim isteği; Sistem kullanıcılarında olağandışı değişiklikler
