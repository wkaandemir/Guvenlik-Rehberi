### Güvenlik Açığı: Gogs PutContents API Symlink Zafiyeti — CVE-2025-8110

---

### 1. Özet ve etki

*   **Yönetici Özeti:** Gogs'un PutContents API'sindeki hatalı symlink yönetimi, kimliği doğrulanmış saldırganın depo sınırlarının dışına dosya yazmasına izin verir. Bu sayede .git/config içindeki sshCommand alanı manipüle edilerek sunucuda kod yürütme elde edilebilir. Bazı raporlarda geniş ölçekli istismar gözlenmiştir.
*   **Etkilenen bileşenler:**
    -   Gogs API (PutContents)
    -   Dosya sistemi ve symlink çözümleme
    -   Git repo yapılandırmaları (.git/config)

---

### 2. Teknik detay (nasıl çalışıyor)

1. Saldırgan PutContents API'yi kullanarak dosya yazma işlemi başlatır.
2. Symlink doğrulaması yetersiz olduğu için depo dışına yazım yapılır.
3. Saldırgan .git/config üzerinde sshCommand gibi alanları manipüle eder.
4. Repo işlemleri tetiklendiğinde RCE gerçekleşebilir.

**Neden (kök neden):**  
-   Symlink çözümlemede yeterli yol doğrulaması yapılmaması
-   Depo sınırı (path traversal) kontrolünün zayıf olması

---

### 3. Adım adım istismar (PoC — kavramsal)

> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Gogs PutContents API
*   **2. Normal istek (meşru):**
```
Depo içinde yeni dosya yazımı
```
*   **3. Manipüle edilmiş istek (PoC payload):**
```
Symlink üzerinden depo dışına yazım ve .git/config manipülasyonu
```
*   **4. Analiz:** Symlink hedefi depo dışına yönlendirir.; Konfigürasyon değişikliği sonraki Git işlemlerini etkiler.
*   **5. Kanıt:** Repo dışında beklenmeyen dosya oluşumu; .git/config içinde yetkisiz değişiklikler

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** Kimliği doğrulanmış kullanıcı
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Gogs yamalarını uygulayın ve PutContents erişimini kısıtlayın.
*   **Orta vadeli:** Repo dosya sistemi izinlerini ve servis hesabı yetkilerini daraltın.
*   **Uzun vadeli:** Depo güvenliği için sürekli tarama ve erişim denetimi uygulayın.

---

### 6. Örnek düzeltme kodu (Shell)
```bash
# Örnek: Repo köklerinde symlink tespiti
find /path/to/gogs-repos -type l -print
```

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] Gogs sürüm ve repo envanterini çıkar
*   [ ] PutContents kullanımını ve API erişimlerini gözden geçir
**Düzeltme**
*   [ ] Yamaları uygula
*   [ ] Repo dizin izinlerini sıkılaştır
**Doğrulama**
*   [ ] Symlink ve .git/config bütünlük kontrolü yap
*   [ ] Yetkisiz dosya yazım denemelerini izle

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** PutContents API çağrılarında olağandışı yoğunluk; Repo dışında dosya yazımı/konfig değişiklikleri
*   **Anomali Tespiti:** Yetkisiz Git konfigürasyon değişiklikleri; Repo dışında beklenmeyen dosya oluşumu
