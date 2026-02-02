### Güvenlik Açığı: Windows DWM Bilgi İfşası — CVE-2026-20805

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Desktop Window Manager (DWM) bileşenindeki bu zafiyet, yerel bir saldırganın ALPC üzerinden bölüm (section) adresi sızdırmasına ve ASLR korumasını baypas etmesine imkan verir. Bilgi ifşası, başka bir kod yürütme açığıyla zincirlendiğinde tam ele geçirme riskini artırır. Kaynak notlarına göre aktif sömürülen bir zafiyettir.
*   **Etkilenen bileşenler:**
    -   Windows Desktop Window Manager (DWM)
    -   ALPC iletişimi ve bölüm adresi yönetimi
    -   ASLR / bellek yerleşim korumaları
    -   Yerel kullanıcı süreçleri

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan, DWM ile ALPC üzerinden iletişim kuran yerel bir süreci hedefler.
2. Özel hazırlanmış bir ALPC isteğiyle bölüm adresi sızıntısı tetiklenir.
3. Sızdırılan adres, DWM bellek yerleşimini ortaya çıkarır ve ASLR etkisizleşir.
4. Bu bilgi, başka bir RCE/EoP açığıyla birleştirilerek saldırı zinciri oluşturur.

**Neden (kök neden):**  
-   ALPC isteklerinde yetersiz doğrulama
-   Bellek adreslerinin güvenli olmayan şekilde ifşa edilmesi

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Yerel kullanıcı bağlamında DWM/ALPC iletişimi
*   **2. Normal istek (meşru):**
```
ALPC isteği (standart parametreler)
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
ALPC isteği (bölüm adresi sızıntısı tetikleyen özel parametreler)
```
*   **4. Analiz:** İstek DWM tarafından işlenir ve bölüm adresi geri döner.; Saldırgan bu adresle ASLR baypası yapar ve ikinci aşama istismar hazırlar.
*   **5. Kanıt:** Süreç belleğine ait adreslerin beklenmedik şekilde geri dönmesi; DWM üzerinde anormal bellek erişimi veya istismar zinciri belirtileri

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** Yerel (düşük ayrıcalıklı kullanıcı)
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Ocak 2026 güvenlik güncellemelerini derhal uygulayın ve istismar göstergelerini izleyin.
*   **Orta vadeli:** Yerel saldırı yüzeyini azaltın (standart kullanıcı, uygulama kısıtları, saldırı yüzeyi azaltma politikaları).
*   **Uzun vadeli:** Düzenli yama yönetimi ve uç nokta sertleştirme programı oluşturun.

---

### 6. Örnek düzeltme kodu (PowerShell)
```powershell
# Örnek: Son yüklenen güncellemeleri gözden geçirme (KB doğrulaması için bültene bakın)
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Windows sürüm ve build envanterini çıkar
*   [ ] DWM ile ilgili aktif istismar IOC listesi topla
**Düzeltme**
*   [ ] İlgili yama paketlerini uygula
*   [ ] Yerel ayrıcalıkları ve uygulama izinlerini kısıtla
**Doğrulama**
*   [ ] Yama seviyesini doğrula
*   [ ] DWM/ALPC anomali izlemelerini aktif et

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** DWM süreç çökmesi/yeniden başlama kayıtları; ALPC çağrılarında olağandışı hata/uyarı artışı
*   **Anomali Tespiti:** DWM üzerinde beklenmeyen bellek erişimleri; Düşük ayrıcalıklı süreçlerden DWM hedefli yoğun trafik
