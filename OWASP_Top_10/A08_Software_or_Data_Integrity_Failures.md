### Güvenlik Açığı: Yazılım veya Veri Bütünlüğü Hataları (Software or Data Integrity Failures)

### 1. Özet ve etki
*   **Yönetici Özeti:** Yazılım veya Veri Bütünlüğü Hataları, verilerin veya kodların bütünlüğünü doğrulama mekanizmalarının eksikliği veya zayıflığı nedeniyle yetkisiz değişikliklerin tespit edilemediği bir güvenlik açığıdır. Bu zafiyet, saldırganların verileri değiştirmesine, kod enjekte etmesine veya sistem davranışını manipüle etmesine olanak tanır. OWASP Top 10:2025'te 8. sırada yer alır ve 2021'e göre sıralaması değişmemiştir. Bu kategori, özellikle CI/CD pipeline'ları, otomatik güncellemeler ve veri işleme sistemleri için kritik önem taşır.
*   **Etkilenen bileşenler:** CI/CD pipeline'ları, otomatik güncelleme sistemleri, veri işleme katmanları, API'ler, veritabanları, imza doğrulama mekanizmaları

---

### 2. Teknik detay (nasıl çalışıyor)
*   Yazılım veya veri bütünlüğü hataları, verilerin veya kodların yetkisiz değişikliklere karşı korunmaması veya bu değişikliklerin tespit edilememesi durumudur.
*   Yaygın bütünlük hataları:
    *   İmzasız kod veya güncellemelerin kabul edilmesi
    *   Veri bütünlüğü kontrollerinin eksikliği
    *   Güvenli olmayan deserialize işlemleri
    *   CI/CD pipeline'larındaki güvenlik açıkları
    *   Otomatik güncelleme mekanizmalarındaki zayıflıklar
    *   Checksum veya hash doğrulamasının yapılmaması
*   **Neden:** Kök neden olarak dijital imza doğrulama mekanizmalarının eksikliği, veri bütünlüğü kontrollerinin ihmal edilmesi, güvenli olmayan deserialize işlemleri ve CI/CD güvenliğinin yetersiz olması sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** İmzasız güncellemeleri kabul eden bir uygulama
*   **2. Normal durum:** 
    ```
    // Güncelleme kontrolü
    if (updateAvailable) {
      downloadUpdate();
      installUpdate();
    }
    ```
*   **3. Manipüle edilmiş durum (PoC payload):**
    ```
    // Saldırgan, malicious güncelleme dosyası oluşturur
    // ve sunucudaki meşru güncelleme dosyasıyla değiştirir
    
    // Uygulama, güncellemeyi indirir ve kurar
    if (updateAvailable) {
      downloadUpdate(); // Malicious güncelleme indirilir
      installUpdate(); // Malicious güncelleme kurulur
    }
    ```
*   **4. Analiz:** Uygulama, güncellemelerin bütünlüğünü doğrulamadan indirir ve kurar. Saldırgan, bu zafiyeti kullanarak malicious kod içeren bir güncelleme oluşturur ve meşru güncellemeyle değiştirir. Uygulama bu güncellemeyi doğrulamadan kurar ve malicious kod sistemde çalışır.
*   **5. Kanıt:** Uygulama güncellendikten sonra arka planda şüpheli aktiviteler gösterir. Loglarda normal güncelleme işlemi olarak kaydedilir.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** İnternete açık, İç network, CI/CD pipeline'ları
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** İmzasız güncellemeleri engelleme, veri bütünlüğü kontrolleri ekleme, güvenli olmayan deserialize işlemlerini düzeltme
*   **Orta vadeli:** Dijital imza doğrulama mekanizmaları kurma, CI/CD pipeline'ını güvenli hale getirme, veri bütünlüğü kontrollerini güçlendirme
*   **Uzun vadeli:** Güvenli yazılım geliştirme yaşam döngüsü (SDLC) benimseme, düzenli bütünlük denetimleri yapma, geliştiricilere güvenlik eğitimi verme, otomatik güvenlik testleri entegrasyonu

---

### 6. Örnek düzeltme kodu (Python)

**Güvensiz Kod:**
```python
import requests
import pickle
import hashlib

# İmzasız güncelleme indirme
def download_update(url):
    response = requests.get(url)
    with open('update.bin', 'wb') as f:
        f.write(response.content)
    return True

# Güvensiz deserialize
def load_data(data):
    # Güvenli olmayan pickle kullanımı - kod enjeksiyonuna açık
    return pickle.loads(data)

# Zayıf veri bütünlüğü kontrolü
def verify_file_integrity(filepath, expected_hash):
    with open(filepath, 'rb') as f:
        file_content = f.read()
    
    # MD5 kullanımı - zayıf hash algoritması
    file_hash = hashlib.md5(file_content).hexdigest()
    return file_hash == expected_hash
```

**Güvenli Kod:**
```python
import requests
import json
import hashlib
import hmac
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives.serialization import load_pem_public_key
from cryptography.hazmat.backends import default_backend

# İmzalı güncelleme indirme ve doğrulama
def download_and_verify_update(url, signature_url, public_key_path):
    # Güncelleme dosyasını indir
    update_response = requests.get(url)
    update_data = update_response.content
    
    # İmzayı indir
    signature_response = requests.get(signature_url)
    signature = signature_response.content
    
    # Genel anahtarı yükle
    with open(public_key_path, 'rb') as f:
        public_key = load_pem_public_key(f.read(), backend=default_backend())
    
    # İmzayı doğrula
    try:
        public_key.verify(
            signature,
            update_data,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )
        
        # İmza doğrulandı, güncellemeyi kaydet
        with open('update.bin', 'wb') as f:
            f.write(update_data)
        return True
    except Exception as e:
        print(f"İmza doğrulama hatası: {e}")
        return False

# Güvenli deserialize
def load_data(data):
    try:
        # JSON kullanarak güvenli deserialize
        return json.loads(data)
    except json.JSONDecodeError:
        print("Geçersiz veri formatı")
        return None

# Güçlü veri bütünlüğü kontrolü
def verify_file_integrity(filepath, expected_hash):
    with open(filepath, 'rb') as f:
        file_content = f.read()
    
    # SHA-256 kullanımı - güçlü hash algoritması
    file_hash = hashlib.sha256(file_content).hexdigest()
    return file_hash == expected_hash

# HMAC ile veri bütünlüğü kontrolü
def verify_data_with_hmac(data, expected_hmac, secret_key):
    calculated_hmac = hmac.new(
        secret_key.encode('utf-8'),
        data,
        hashlib.sha256
    ).hexdigest()
    
    # Zamanlama saldırılarını önlemek için sabit zamanlı karşılaştırma
    return hmac.compare_digest(calculated_hmac, expected_hmac)
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Tüm veri ve kod bütünlüğü mekanizmalarını envanterle
*   [ ] İmza doğrulama süreçlerini analiz et
*   [ ] Deserialize işlemlerini incele

**Düzeltme**
*   [ ] Dijital imza doğrulama mekanizmaları kur
*   [ ] Veri bütünlüğü kontrolleri ekle
*   [ ] Güvenli deserialize yöntemleri kullan
*   [ ] CI/CD pipeline'ını güvenli hale getir

**Doğrulama**
*   [ ] İmza doğrulama mekanizmalarını test et
*   [ ] Veri bütünlüğü kontrollerini doğrula
*   [ ] Penetrasyon testi yaptır
*   [ ] Düzenli güvenlik denetimleri yap

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   İmza doğrulama hatalarını izle
    *   Veri bütünlüğü ihlallerini tespit et
    *   Regex: `(?i)(signature verification|data integrity|checksum).*fail|fail.*(signature verification|data integrity|checksum)`
*   **Anomali Tespiti:** 
    *   İmzasız kod veya güncelleme kurulum denemeleri
    *   Veri bütünlüğü kontrol hatalarındaki artış
    *   Deserialize işlemlerindeki anormallikler
    *   CI/CD pipeline'larındaki yetkisiz değişiklikler
