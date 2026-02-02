### Güvenlik Açığı: Microsoft Office OLE/COM Koruma Baypası — CVE-2026-21509

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Microsoft Office'te OLE/COM güvenlik kararlarının yanlış uygulanması, özel hazırlanmış bir dosyanın açılmasıyla uzaktan kod yürütmeye imkan verir. Önizleme bölmesi tetiklemese de kullanıcı dosyayı açtığında saldırı gerçekleşebilir. Microsoft 365 için sunucu tarafında düzeltme uygulanmış, Office 2016/2019 için manuel güncelleme ve konfigürasyon ihtiyacı vurgulanmıştır.
*   **Etkilenen bileşenler:**
    -   Microsoft Office (Word/Excel/PowerPoint)
    -   OLE/COM nesne işleme katmanı
    -   Dosya açma akışı ve güvenlik kararları
    -   Kullanıcı uç noktaları

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan, OLE/COM içeren özel bir Office dosyası hazırlar.
2. Kullanıcı dosyayı açtığında güvenlik kontrolü baypas edilir.
3. OLE/COM nesnesi işlenirken saldırgan kodu yürütülür.

**Neden (kök neden):**  
-   Güvenilmeyen girdilere aşırı güven
-   OLE/COM güvenlik kararlarında mantık hatası

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Kurbanın Office uygulaması (dosya açma akışı)
*   **2. Normal istek (meşru):**
```
Güvenli bir .docx/.xlsx dosyası açma
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
OLE/COM içeren manipüle edilmiş Office dosyası
```
*   **4. Analiz:** Dosya açıldığında OLE/COM bileşeni güvenlik bariyerini aşar.; Saldırgan kodu kullanıcı bağlamında yürütülür.
*   **5. Kanıt:** Office süreçlerinden beklenmeyen child-process çalışması; Kullanıcı profilinde anormal dosya/komut yürütme izleri

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Internet kaynaklı dosya/ekler
*   **Karmaşıklık:** Düşük-Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Acil güncellemeleri uygulayın ve şüpheli Office eklerini engelleyin.
*   **Orta vadeli:** ASR/Endpoint politikalarıyla OLE/COM çalıştırmayı kısıtlayın ve e-posta filtrelerini sıkılaştırın.
*   **Uzun vadeli:** Kullanıcı farkındalığı, sandboxing ve güvenli dosya açma süreçleri oluşturun.

---

### 6. Örnek düzeltme kodu (PowerShell)
```powershell
# Örnek: Son güncellemeleri kontrol et (KB listesi için resmi bülteni takip edin)
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Office sürüm envanterini çıkar
*   [ ] E-posta ekleri ve dosya paylaşım kanallarını gözden geçir
**Düzeltme**
*   [ ] Acil güncellemeleri uygula
*   [ ] OLE/COM tabanlı içeriğe yönelik politikaları sıkılaştır
**Doğrulama**
*   [ ] Yama seviyesini doğrula
*   [ ] Office süreç davranışını EDR ile izle

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Office süreçlerinden (winword/excel/powerpnt) beklenmeyen child-process üretimi; OLE/COM ile ilişkili hata/uyarı logları
*   **Anomali Tespiti:** Kullanıcı etkileşimi olmadan başlayan Office süreçleri; Office tarafından başlatılan sistem komutları
