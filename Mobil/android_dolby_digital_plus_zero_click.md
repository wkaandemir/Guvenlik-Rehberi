### Güvenlik Açığı: Android Dolby Digital Plus Zero-Click RCE

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Dolby Digital Plus kodek bileşenindeki zafiyet, ses dosyası metadata alanlarının manipülasyonu ile sıfır tıklama (zero-click) saldırılarına imkan verir. Zararlı ses dosyası mesajlaşma uygulamalarına ulaştığında oynatılmasa bile kod yürütme tetiklenebilir. Google'ın kendi cihazlarında erken düzeltme uyguladığı, ekosistemdeki diğer üreticilerin ise risk altında kaldığı belirtilmiştir.
*   **Etkilenen bileşenler:**
    -   Android medya framework/kodek bileşenleri
    -   Dolby Digital Plus parser
    -   Mesajlaşma uygulamaları üzerinden dosya alımı

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan, metadata alanı manipüle edilmiş ses dosyası hazırlar.
2. Dosya alındığında arka planda işlenir ve hata tetiklenir.
3. Kod yürütme kullanıcı etkileşimi olmadan gerçekleşebilir.

**Neden (kök neden):**  
-   Metadata alanlarının yeterince doğrulanmaması
-   Kodek parser hatası

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Android medya işleme hattı
*   **2. Normal istek (meşru):**
```
Standart ses dosyasının alınması
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
Manipüle edilmiş Dolby Digital Plus ses dosyası
```
*   **4. Analiz:** Dosya alımı sırasında parser hatası tetiklenir.; Kod yürütme arka planda gerçekleşir.
*   **5. Kanıt:** Medya servislerinde çökme kayıtları; Beklenmeyen uygulama davranışları

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Mesajlaşma uygulamaları / dosya alımı
*   **Karmaşıklık:** Düşük-Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Üretici güvenlik yamalarını uygulayın ve otomatik medya indirmeyi kısıtlayın.
*   **Orta vadeli:** Uygulama izinlerini gözden geçirin ve riskli uygulamaları sınırlayın.
*   **Uzun vadeli:** Mobil cihaz güncelleme politikası oluşturun.

---

### 6. Örnek düzeltme kodu (adb)
```bash
# Örnek: Güvenlik yaması seviyesini kontrol et
adb shell getprop ro.build.version.security_patch
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Android cihaz envanteri çıkar
*   [ ] Güvenlik yama seviyelerini belirle
**Düzeltme**
*   [ ] OEM yamalarını uygula
*   [ ] Otomatik medya indirme ayarlarını kısıtla
**Doğrulama**
*   [ ] Yama seviyesini doğrula
*   [ ] Medya servislerinin stabilitesini izle

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Medya servis çökme logları; Mesajlaşma uygulamalarında anormal dosya işleme hataları
*   **Anomali Tespiti:** Dosya alımı sonrası uygulama crash artışı; Cihazda beklenmeyen arka plan süreçleri
