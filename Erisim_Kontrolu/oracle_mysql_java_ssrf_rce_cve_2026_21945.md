### Güvenlik Açığı: Oracle MySQL/Java SE SSRF/RCE — CVE-2026-21945

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Oracle Ocak 2026 CPU özetinde MySQL/Java SE ailesinde SSRF ve kod yürütme etkisiyle ilişkilendirilen bir CVE olarak listelenmiştir. Teknik detaylar sınırlı olduğundan değerlendirme genel risk varsayımlarıyla yapılmıştır.
*   **Etkilenen bileşenler:**
    -   Oracle MySQL / Java SE bileşenleri
    -   HTTP istemci ve ağ erişim katmanı

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan SSRF benzeri bir akışı tetikleyebilir.
2. İç servislerin erişimi ve potansiyel RCE oluşabilir.

**Neden (kök neden):**  
-   SSRF korumalarının eksikliği
-   Girdi doğrulama zafiyeti

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** İç ağa erişebilen servis çağrıları
*   **2. Normal istek (meşru):**
```
Meşru dış HTTP isteği
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
İç ağa yönlendirilmiş SSRF isteği
```
*   **4. Analiz:** Uygulama iç servisleri erişilebilir hale getirir.; Saldırgan kritik endpoint'lere ulaşır.
*   **5. Kanıt:** Outbound isteklerde beklenmeyen hedefler; İç servis loglarında harici kaynaklı istekler

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** Uygulama HTTP çağrıları
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Oracle CPU yamalarını uygulayın.
*   **Orta vadeli:** Outbound istekleri allowlist ile sınırlandırın.
*   **Uzun vadeli:** SSRF testleri ve güvenli ağ segmentasyonu uygulayın.

---

### 6. Örnek düzeltme kodu (Java)
```java
// Örnek: Allowlist tabanlı hedef doğrulama
URI u = URI.create(url);
if (!ALLOWED_HOSTS.contains(u.getHost())) {
    throw new SecurityException("Target not allowed");
}
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] MySQL/Java SE sürüm envanteri çıkar
*   [ ] Outbound HTTP çağrılarını envanterle
**Düzeltme**
*   [ ] CPU yamalarını uygula
*   [ ] SSRF korumalarını uygula
**Doğrulama**
*   [ ] Outbound istek loglarını doğrula
*   [ ] SSRF testleriyle doğrulama yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Uygulama sunucularından iç ağa giden beklenmeyen istekler; SSRF pattern'lerine benzer URL çağrıları
*   **Anomali Tespiti:** İç servislere dış kaynaklı erişim artışı; Uygulama tarafında başarısız URL doğrulama hataları
