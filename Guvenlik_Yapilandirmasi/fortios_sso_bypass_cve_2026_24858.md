### Güvenlik Açığı: FortiOS/FortiManager/FortiAnalyzer FortiCloud SSO Baypası — CVE-2026-24858

---

### 1. Özet ve etki

*   **Yönetici Özeti:** FortiCloud SSO mekanizmasını baypas eden bu zafiyet, saldırganın kendi FortiCloud hesabı ve kayıtlı cihazı üzerinden diğer hesapların cihazlarına erişim sağlamasına izin verir. Ocak 2026'da birçok kurumda kendiliğinden oluşan yerel yönetici hesapları raporlanmıştır. Fortinet, SSO hizmetini geçici olarak durdurup yamalı cihazlar için yeniden açmıştır.
*   **Etkilenen bileşenler:**
    -   FortiOS, FortiManager, FortiAnalyzer
    -   FortiCloud Single Sign-On (SSO)
    -   Kimlik doğrulama ve yetkilendirme akışları

---

### 2. Teknik detay (nasıl çalışıyor)

1. SSO akışında hesaplar arası yetkilendirme hatası bulunur.
2. Saldırgan, kendi FortiCloud hesabıyla diğer hesaplara bağlı cihazlara erişir.
3. Erişim sonrası yerel yönetici hesapları oluşturulabilir.

**Neden (kök neden):**  
-   Yetkilendirme/tenant izolasyonu hatası
-   SSO güven sınırının yanlış uygulanması

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** FortiCloud SSO ile yönetilen firewall cihazları
*   **2. Normal istek (meşru):**
```
Meşru FortiCloud hesabı ile kendi cihazlarına erişim
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
SSO baypası ile farklı tenant cihazlarına erişim
```
*   **4. Analiz:** SSO doğrulaması yanlış bağlandığı için erişim genişler.; Saldırgan yerel admin hesabı ekleyerek kalıcılık sağlar.
*   **5. Kanıt:** Cihazlarda beklenmeyen yerel yönetici hesapları; FortiCloud erişim loglarında olağandışı tenant geçişleri

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** FortiCloud SSO üzerinden uzaktan
*   **Karmaşıklık:** Düşük-Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Yamaları uygulayın ve SSO kullanımını geçici olarak kısıtlayın.
*   **Orta vadeli:** Tüm yerel admin hesaplarını denetleyin, kimlik bilgilerini rotasyonla değiştirin.
*   **Uzun vadeli:** SSO entegrasyonları için düzenli güvenlik denetimleri uygulayın.

---

### 6. Örnek düzeltme kodu (Metin (Şablon))
```text
# Örnek (şablon):
# 1) FortiCloud SSO durumunu gözden geçir
# 2) Yamalı sürüme yükselt
# 3) Yerel admin hesaplarını denetle ve rotasyon uygula
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Fortinet ürün envanterini çıkar
*   [ ] SSO kullanım durumunu ve tenant eşleşmelerini kontrol et
**Düzeltme**
*   [ ] Yamaları uygula
*   [ ] Yerel admin hesaplarını temizle ve parolaları yenile
**Doğrulama**
*   [ ] SSO erişim loglarını gözden geçir
*   [ ] Yeni yetkisiz admin oluşmadığını doğrula

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** FortiCloud erişim loglarında anormal tenant geçişleri; Cihazlarda yeni yerel admin hesabı oluşturma olayları
*   **Anomali Tespiti:** Kısa sürede çoklu cihaz erişimi; SSO üzerinden beklenmeyen yönetici oturumları
