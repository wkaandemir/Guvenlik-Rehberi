### Güvenlik Açığı: Secure Boot Güvenlik Özelliği Baypası — CVE-2026-21265

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Secure Boot mekanizmasını etkileyen bu zafiyet, kök sertifika ömrünün sona ermesiyle güven zincirinin zayıflamasına yol açar. 2011 kök sertifikalarının Haziran 2026'da sona erecek olması, güncellenmeyen sistemlerde boot sürecine kötü amaçlı bileşen yüklenmesini kolaylaştırabilir. Üretici, 2023 sertifikalarının sisteme tanımlanması gerektiğini belirtmektedir.
*   **Etkilenen bileşenler:**
    -   UEFI Secure Boot zinciri
    -   Kök sertifika (db/dbx) yönetimi
    -   Bootloader ve firmware

---

### 2. Teknik detay (nasıl çalışıyor)

1. Eski kök sertifikaların süresi dolduğunda güven zinciri zayıflar.
2. Sistem güncellenmezse Secure Boot doğrulama adımları baypas edilebilir.
3. Saldırgan boot sürecine kötü amaçlı bileşen enjekte edebilir.

**Neden (kök neden):**  
-   Sertifika yaşam döngüsü yönetiminde gecikme
-   Firmware/güncelleme zincirinin güncel tutulmaması

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** UEFI Secure Boot doğrulama zinciri
*   **2. Normal istek (meşru):**
```
İmzalı bootloader ile güvenli açılış
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
Süresi dolmuş sertifikaya dayanarak kötü amaçlı boot bileşeni
```
*   **4. Analiz:** Sistem sertifika güncellemesini almadığı için doğrulama zayıflar.; Boot bileşeni yüklenir ve erken aşamada kalıcılık elde edilir.
*   **5. Kanıt:** Boot sürecinde beklenmeyen imza/sertifika uyarıları; Firmware/boot zincirinde yetkisiz değişiklik izleri

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** Yerel/physical + yetkili erişim
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Sertifika güncellemelerini ve üretici yamalarını acilen uygulayın.
*   **Orta vadeli:** Firmware/boot bileşenlerini güncel tutacak süreç oluşturun.
*   **Uzun vadeli:** Sertifika rotasyonu ve Secure Boot yaşam döngüsünü kurumsal politika haline getirin.

---

### 6. Örnek düzeltme kodu (PowerShell)
```powershell
# Örnek: Secure Boot durumunu kontrol et
Confirm-SecureBootUEFI
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Secure Boot etkin cihaz envanteri çıkar
*   [ ] Sertifika/firmware sürüm bilgilerini doğrula
**Düzeltme**
*   [ ] Üretici tarafından önerilen sertifika güncellemelerini uygula
*   [ ] Firmware ve bootloader güncellemelerini planla
**Doğrulama**
*   [ ] Secure Boot doğrulama çıktısını kontrol et
*   [ ] Boot zincirinde yetkisiz değişiklik olmadığını doğrula

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Boot/firmware olay günlüklerinde sertifika uyarıları; Secure Boot devre dışı kalma olayları
*   **Anomali Tespiti:** Beklenmeyen boot bileşeni değişiklikleri; Firmware'de yetkisiz modifikasyon belirtileri
