Sen kıdemli bir Uygulama Güvenliği Mühendisi ve Teknik Yazarsın.
**Konuşma geçmişimizdeki** veya az önce bahsettiğim **Zafiyet Özeti / Ham Notlar** bilgisini temel alarak, bu bilgileri projenin standartlarına uygun, detaylı, teknik ve profesyonel bir güvenlik dokümanına dönüştürmeni istiyorum.

Amacımız: Konuşma geçmişindeki dağınık notlardan yola çıkarak, geliştiricilerin güvenlik açığını anlayıp kalıcı olarak çözebileceği tam kapsamlı bir rehber oluşturmak.

Lütfen çıktıyı tam olarak aşağıdaki yapı ve başlıklara sadık kalarak, **Türkçe** dilinde hazırla.

---

### İSTENEN ÇIKTI YAPISI:

### Güvenlik Açığı: [Profesyonel Zafiyet Başlığı]

### 1. Özet ve etki
*   **Yönetici Özeti:** Zafiyetin ne olduğunu, neden kaynaklandığını ve iş/sistem üzerindeki kritik etkisini (Veri sızıntısı, RCE, İtibar kaybı vb.) profesyonel bir dille özetle.
*   **Etkilenen bileşenler:** (Konuşma geçmişinden çıkarım yaparak; API endpoint'leri, veritabanı, parser kütüphaneleri, middleware vb. listele)

---

### 2. Teknik detay (nasıl çalışıyor)
*   Mekanizmayı adım adım teknik maddeler halinde açıkla.
*   Arka planda kodun veya sistemin nasıl davrandığını anlat.
*   **Neden:** Kök nedeni belirt (Örn: Input validation eksikliği, güvensiz deserialization, output encoding yapılmaması).

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** (Senaryolaştırılmış hedef URL veya Fonksiyon)
*   **2. Normal istek:** (Sistemin beklediği meşru HTTP isteği veya kod örneği)
*   **3. Manipüle edilmiş istek (PoC payload):** (Saldırganın gönderdiği zararlı veri)
*   **4. Analiz:** Payload sisteme girdiğinde ne tetikleniyor? (Teknik analiz)
*   **5. Kanıt:** Loglarda, ekranda veya sistem davranışında saldırının başarılı olduğu nasıl görülür?

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** (Notlara göre tahmin et: Düşük / Orta / Yüksek / Kritik)
*   **Saldırı Yüzeyi:** (İnternete açık, İç ağ, Yetkili kullanıcı vb.)
*   **Karmaşıklık:** (Saldırının zorluk derecesi)

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** (WAF kuralı, config değişikliği, geçici yama)
*   **Orta vadeli:** (Güvenli kütüphane kullanımı, kod refactoring, input validation)
*   **Uzun vadeli:** (Mimari iyileştirmeler, eğitim, CI/CD güvenlik testleri)

---

### 6. Örnek düzeltme kodu ([Dil/Teknoloji])
*   Notlardaki teknolojiye uygun (Node.js, PHP, Python, Java vb.) **Güvenli Kod** örneği oluştur.
*   Gerekirse "Güvensiz" ve "Güvenli" kod bloklarını karşılaştır.
*   Kodun içine açıklayıcı yorum satırları ekle.

---

### 7. Kontrol Listesi (Checklist)
Pratik ve uygulanabilir adımları kutucuklu liste (`[ ]`) halinde yaz:
**Hazırlık**
*   [ ] ...
**Düzeltme**
*   [ ] ...
**Doğrulama**
*   [ ] ...

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Saldırıyı tespit etmek için loglarda aranacak pattern'ler (Regex veya anahtar kelimeler).
*   **Anomali Tespiti:** Hangi davranışlar şüpheli kabul edilmeli?
