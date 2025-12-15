### Güvenlik Açığı: Kriptografik Hatalar (Cryptographic Failures)

### 1. Özet ve etki
*   **Yönetici Özeti:** Kriptografik Hatalar, hassas verilerin korunması için kullanılan şifreleme algoritmalarının, protokollerin veya uygulamaların zayıf veya yanlış kullanılması nedeniyle ortaya çıkan güvenlik açıklarıdır. Bu zafiyet, veri gizliliğinin ve bütünlüğünün ihlal edilmesine neden olabilir. OWASP Top 10:2025'te 4. sırada yer alır ve 2021'e göre sıralaması düşmüş olsa da etkisi hala kritik düzeydedir, özellikle düzenleyici gereksinimler (GDPR, KVKK vb.) nedeniyle önemini korumaktadır.
*   **Etkilenen bileşenler:** Veri depolama sistemleri, veri iletişim katmanları, oturum yönetimi, parola saklama mekanizmaları, API güvenliği, anahtar yönetimi sistemleri

---

### 2. Teknik detay (nasıl çalışıyor)
*   Kriptografik hatalar, verilerin şifrelenmesi, şifrelerin saklanması veya dijital imzaların doğrulanması gibi kriptografik işlemlerdeki zayıflıklardır.
*   Yaygın kriptografik hatalar:
    *   Zayıf şifreleme algoritmalarının kullanılması (MD5, SHA1, DES vb.)
    *   Sabit veya zayıf anahtarların kullanılması
    *   Şifrelerin düz metin olarak saklanması
    *   SSL/TLS yapılandırmasındaki hatalar
    *   Rastgele sayı üreteçlerinin zayıf olması
    *   Kriptografik nonce'ların tekrar kullanılması
    *   Anahtar yönetimi süreçlerinin eksikliği
*   **Neden:** Kök neden olarak kriptografi konusunda yetersiz bilgi, standart kriptografik kütüphaneler yerine özel algoritmalar geliştirme, anahtar yönetimi süreçlerinin eksikliği ve kriptografik implementasyonlardaki hatalar sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** MD5 ile hash'lenmiş parola veritabanı
*   **2. Normal durum:** 
    ```
    // Veritabanında saklanan hash'lenmiş parola
    username: "admin", password_hash: "5f4dcc3b5aa765d61d8327deb882cf99"  // "password" MD5 hash'i
    ```
*   **3. Manipüle edilmiş durum (PoC payload):**
    ```
    // Saldırgan, rainbow table kullanarak hash'i çözer
    // veya MD5'in hızlı olmasını kullanarak brute force saldırısı yapar
    ```
*   **4. Analiz:** Saldırgan, MD5'in hızlı olmasını ve collision saldırılarına açık olmasını kullanarak hash'lenmiş parolaları çözer. MD5, modern saldırılara karşı korumalı değildir ve kolayca kırılabilir.
*   **5. Kanıt:** Saldırgan, çözdüğü parola ile sisteme yetkisiz erişim sağlar. Loglarda başarılı yetkilendirme denemeleri görülür.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek
*   **Saldırı Yüzeyi:** İnternete açık, İç ağ, Veri depolama sistemleri
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Zayıf şifreleme algoritmalarını güçlü olanlarla değiştirme, SSL/TLS yapılandırmasını düzeltme, düz metin parola saklamayı durdurma
*   **Orta vadeli:** Güçlü şifreleme kütüphaneleri kullanma, anahtar yönetimi sistemi kurma, parola hashing için bcrypt, scrypt veya Argon2 kullanma
*   **Uzun vadeli:** Kriptografik standartları takip etme, düzenli kriptografik denetimler yapma, geliştiricilere kriptografi eğitimi verme, anahtar rotasyon süreçleri oluşturma

---

### 6. Örnek düzeltme kodu (Python)

**Güvensiz Kod:**
```python
import hashlib
import base64
from Crypto.Cipher import AES

# Zayıf parola hashing
def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

# Zayıf şifreleme
def encrypt_data(data, key):
    # Sabit IV kullanımı - güvenlik açığı
    iv = b'1234567890123456'
    cipher = AES.new(key.encode('utf-8'), AES.MODE_CBC, iv)
    return base64.b64encode(cipher.encrypt(data))

# Parolayı düz metin olarak saklama
def save_user(username, password):
    hashed_password = hash_password(password)
    # Veritabanına kaydetme işlemi
    pass
```

**Güvenli Kod:**
```python
import hashlib
import base64
import os
import bcrypt
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

# Güçlü parola hashing
def hash_password(password):
    # bcrypt ile salt'lı hashing
    salt = bcrypt.gensalt()
    return bcrypt.hashpw(password.encode('utf-8'), salt)

# Güçlü şifreleme
def encrypt_data(data, key):
    # Her seferinde rastgele IV oluştur
    iv = os.urandom(16)
    cipher = AES.new(key.encode('utf-8'), AES.MODE_CBC, iv)
    encrypted_data = cipher.encrypt(pad(data.encode('utf-8'), AES.block_size))
    # IV'i şifreli veriyle birlikte döndür
    return base64.b64encode(iv + encrypted_data)

# Güvenli parola saklama
def save_user(username, password):
    hashed_password = hash_password(password)
    # Veritabanına kaydetme işlemi
    pass

# Güvenli parola doğrulama
def verify_password(stored_password, provided_password):
    return bcrypt.checkpw(provided_password.encode('utf-8'), stored_password)
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Tüm kriptografik kullanımlarını envanterle
*   [ ] Kullanılan algoritmaları ve protokolleri belirle
*   [ ] Kriptografik standartları araştır

**Düzeltme**
*   [ ] Zayıf şifreleme algoritmalarını güçlü olanlarla değiştir
*   [ ] Parola hashing için bcrypt, scrypt veya Argon2 kullan
*   [ ] SSL/TLS yapılandırmasını güçlendir
*   [ ] Anahtar yönetimi sistemi kur

**Doğrulama**
*   [ ] Kriptografik implementasyonları test et
*   [ ] SSL/TLS yapılandırmasını doğrula
*   [ ] Anahtar yönetimi süreçlerini denetle
*   [ ] Penetrasyon testi yaptır

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Zayıf şifreleme algoritması kullanımını izle
    *   Başarısız parola doğrulama denemelerini tespit et
    *   Regex: `(?i)(md5|sha1|des|rc4).*encrypt|encrypt.*(md5|sha1|des|rc4)`
*   **Anomali Tespiti:** 
    *   Normal dışı şifreleme/şifre çözme aktiviteleri
    *   Kriptografik anahtarlara yetkisiz erişim denemeleri
    *   SSL/TLS handshake hataları
    *   Parola kırma saldırıları
