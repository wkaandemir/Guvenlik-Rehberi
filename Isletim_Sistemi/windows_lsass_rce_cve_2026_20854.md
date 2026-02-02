### Güvenlik Açığı: Windows LSASS Uzaktan Kod Yürütme — CVE-2026-20854

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Windows LSASS bileşenini etkileyen bu RCE zafiyeti, kimlik doğrulama altyapısının hedef alınmasına ve yüksek ayrıcalıkla kod yürütmeye yol açabilir. Ocak 2026 özetinde CVE ID tablo halinde yer alsa da teknik detaylar sınırlıdır.
*   **Etkilenen bileşenler:**
    -   LSASS (Local Security Authority Subsystem Service)
    -   Kimlik doğrulama ve yetkilendirme akışları
    -   Etki alanı/kurumsal kimlik altyapısı

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan, LSASS'a etki eden özel bir girdi/istek akışı tetikler.
2. Bellek işleme hatası RCE ile sonuçlanabilir.
3. Başarılı istismar yüksek ayrıcalıkla sistem kontrolüne yol açar.

**Neden (kök neden):**  
-   Bellek yönetimi hatası
-   Kimlik doğrulama girdi doğrulamasında zayıflık

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** LSASS kimlik doğrulama arayüzleri
*   **2. Normal istek (meşru):**
```
Standart kimlik doğrulama trafiği
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
Özel hazırlanmış kimlik doğrulama isteği
```
*   **4. Analiz:** İşleme sırasında bellek bozulması oluşur.; Saldırgan kodu LSASS bağlamında yürütür.
*   **5. Kanıt:** LSASS çökmesi veya yeniden başlama kayıtları; Kimlik altyapısında beklenmeyen yetki yükselmeleri

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Ağ/kimlik doğrulama altyapısı
*   **Karmaşıklık:** Bilinmiyor (muhtemelen orta)

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Microsoft güvenlik güncellemelerini derhal uygulayın.
*   **Orta vadeli:** LSASS erişimini ve kimlik doğrulama yollarını ağ seviyesinde sınırlandırın.
*   **Uzun vadeli:** Kimlik altyapısı sertleştirme ve sürekli zafiyet yönetimi programı uygulayın.

---

### 6. Örnek düzeltme kodu (PowerShell)
```powershell
# Örnek: Son güncellemeleri kontrol et
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Etki alanı denetleyici ve istemci envanteri çıkar
*   [ ] Kimlik doğrulama trafiği için log toplama hazırla
**Düzeltme**
*   [ ] Güncellemeleri uygula
*   [ ] LSASS'a ağ erişimini kısıtla
**Doğrulama**
*   [ ] LSASS süreç stabilitesini ve logları doğrula
*   [ ] Yetki yükseltme anomali taraması yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** LSASS çökmesi/yeniden başlama olayları; Kimlik doğrulama hatalarında ani artış
*   **Anomali Tespiti:** Kısa sürede çoklu başarısız oturum açma; LSASS tarafından başlatılan beklenmeyen süreçler
