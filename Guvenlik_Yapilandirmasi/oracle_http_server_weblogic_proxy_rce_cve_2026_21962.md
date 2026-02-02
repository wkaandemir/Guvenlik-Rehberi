### Güvenlik Açığı: Oracle HTTP Server / WebLogic Proxy Plug-in RCE — CVE-2026-21962

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Oracle HTTP Server ve WebLogic Server Proxy Plug-in bileşenlerini etkileyen bu zafiyet, kimliği doğrulanmamış saldırganlara uzaktan tam erişim sağlayabilir. Ocak 2026 sonunda PoC yayınlanması DMZ'deki sistemler için riski artırmıştır.
*   **Etkilenen bileşenler:**
    -   Oracle HTTP Server
    -   WebLogic Server Proxy Plug-in
    -   DMZ'deki yayınlanmış servisler

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan proxy plug-in'e özel hazırlanmış HTTP isteği gönderir.
2. Bileşen hatalı girdi doğrulaması nedeniyle RCE'ye açık hale gelir.
3. Yetkisiz uzaktan erişim sağlanır.

**Neden (kök neden):**  
-   Girdi doğrulama eksikliği
-   Kimlik doğrulama/erişim kontrolü zayıflığı

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Proxy Plug-in üzerinden erişilen HTTP uçları
*   **2. Normal istek (meşru):**
```
Standart HTTP isteği
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
Manipüle edilmiş HTTP isteği
```
*   **4. Analiz:** İstek plug-in tarafından işlenirken komut yürütme tetiklenir.; Saldırgan sistem erişimi elde eder.
*   **5. Kanıt:** Web sunucusu loglarında olağandışı istekler; Sunucu üzerinde yetkisiz dosya/komut izleri

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Internet'e açık servisler
*   **Karmaşıklık:** Düşük-Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Oracle CPU yamalarını derhal uygulayın.
*   **Orta vadeli:** Proxy plug-in'i yalnızca gerekli erişimlerle sınırlandırın ve WAF ekleyin.
*   **Uzun vadeli:** DMZ segmentasyonunu güçlendirin ve düzenli zafiyet taraması yapın.

---

### 6. Örnek düzeltme kodu (Metin (Şablon))
```text
# Örnek (şablon):
# 1) Oracle CPU yamasını uygula
# 2) Proxy plug-in erişimini sınırla
# 3) DMZ/WAF kurallarını güncelle
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] OHS/WebLogic sürüm envanteri çıkar
*   [ ] DMZ'deki servisleri listele
**Düzeltme**
*   [ ] CPU yamalarını uygula
*   [ ] WAF/ACL ile erişimi sınırla
**Doğrulama**
*   [ ] Sürüm doğrulaması yap
*   [ ] PoC trafiğine karşı logları kontrol et

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Proxy plug-in'e gelen olağandışı HTTP istekleri; Web server hata loglarında ani artış
*   **Anomali Tespiti:** DMZ sunucularında beklenmeyen süreçler; Kısa sürede çoklu başarısız istek
