# TODO: OWASP A04:2025 — Kriptografik Hatalar

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A04:2025 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A04_Cryptographic_Failures.md](../OWASP_Top_10/A04_Cryptographic_Failures.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Kriptografik Hatalar, hassas verilerin korunmasinda kullanilan sifreleme mekanizmalarinin zayif, hatali veya eksik uygulanmasindan kaynaklanan guvenlik aciklaridir. Zayif hash algoritmalari (MD5, SHA1), eskimis sifreleme standartlari (DES, RC4), sabit veya zayif anahtarlar, duz metin parola depolama ve zayif rastgele sayi uretimleri bu kategorinin baslica ornekleridir. OWASP Top 10:2025'de 4. sirada yer almaktadir.
*   **Etkilenen bilesenler:** Parola depolama sistemleri, veritabani sifreleme katmanlari, SSL/TLS yapilandirmalari, API iletisim kanallari, dosya sifreleme mekanizmalari, anahtar yonetim sistemleri, rastgele sayi ureticileri, dijital imza altyapilari

---

### 2. Teknik detay (nasil calisiyor)
*   Kriptografik hatalar, hassas verilerin aktarim sirasinda (in-transit) veya depolama sirasinda (at-rest) yeterli duzey kriptografik koruma saglanmadiginda ortaya cikar.
*   Yaygın ortaya cikis bicimleri:
    *   Parolalarin duz metin veya zayif hash algoritmalariyla (MD5, SHA1) depolanmasi
    *   Eskimis sifreleme algoritmalarinin kullanilmasi (DES, 3DES, RC4, Blowfish)
    *   Sabit (hardcoded) sifreleme anahtarlarinin kaynak kodda bulunmasi
    *   Sifreleme islemlerinde sabit veya tekrar eden IV (Initialization Vector) kullanilmasi
    *   Zayif rastgele sayi ureticileri (PRNG) ile kriptografik materyal uretilmesi
    *   SSL/TLS yapilandirmasinda zayif sifreleme takimlarinin kabul edilmesi
*   **Neden:** Kriptografi konusunda gelistirici bilgi eksikligi, eski sistemlerdeki miras kodlarin guncellenmemesi, performans kaygilarinin guvenligin onune gecirilmesi ve kriptografik standartlarin takip edilmemesi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** MD5 ile hashlenmis parola veritabani
*   **2. Normal durum:**
    ```
    Kullanici kayit olur -> Parola MD5 ile hashlenir -> Hash veritabaninda saklanir
    Ornek: "P@ssw0rd123" -> MD5 -> "482c811da5d5b4bc6d497ffa98491e38"
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    # Saldirgan veritabani dump'ina erisir
    # Rainbow table veya hashcat ile MD5 hashleri kirar
    hashcat -m 0 -a 0 hashes.txt rockyou.txt
    # Sonuc: 482c811da5d5b4bc6d497ffa98491e38:P@ssw0rd123
    # Tum MD5 hashler saniyeler icinde cozulur
    ```
*   **4. Analiz:** MD5 algoritmasi kriptografik olarak kirilmistir. Onceden hesaplanmis rainbow table'lar veya GPU destekli brute-force araclari ile MD5 hashleri saniyeler icinde orijinal degerlerine donusturulur. Tuzlama (salt) kullanilmamasi durumu daha da agirlastirir.
*   **5. Kanit:** Saldirgan, veritabanindaki tum kullanici parolalarini duz metin olarak elde eder ve bu parolalarla hesaplara erisir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, Ic ag, Veritabani erisimi
*   **Karmasiklik:** Orta

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Duz metin parola depolayan sistemleri tespit edip bcrypt/Argon2 ile yeniden hashleme, SSL/TLS yapilandirmasinda zayif sifreleme takimlarini devre disi birakma, sabit sifreleme anahtarlarini degistirme.
*   **Orta vadeli:** Tum kriptografik uygulamalarin envanterini cikarma, AES-256-GCM gibi modern sifreleme algoritmalaarina gecis yapma, rastgele IV uretimi icin kriptografik olarak guvenli PRNG kullanma, anahtar yonetim sistemi (KMS) kurma.
*   **Uzun vadeli:** Kriptografik standartlar politikasi olusturma, anahtar rotasyon sureclerini otomatiklestirme, gelistiricilere kriptografi egitimi verme, kuantum sonrasi kriptografi hazirliklarini baslatma.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
import hashlib

# Zayif: MD5 ile parola hashleme, tuzlama yok
def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

# Zayif: Sabit IV ile sifreleme
from Crypto.Cipher import AES
key = b'sabit_anahtar_16'  # Sabit anahtar
iv = b'sabit_iv_1234567'   # Sabit IV - her seferinde ayni
cipher = AES.new(key, AES.MODE_CBC, iv)
```

**Guvenli kod:**
```python
import bcrypt
import os
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes

# Guvenli: bcrypt ile parola hashleme (otomatik tuzlama)
def hash_password(password):
    salt = bcrypt.gensalt(rounds=12)
    return bcrypt.hashpw(password.encode('utf-8'), salt)

def verify_password(password, hashed):
    return bcrypt.checkpw(password.encode('utf-8'), hashed)

# Guvenli: Rastgele IV ile AES-256-GCM sifreleme
def encrypt_data(plaintext, key):
    iv = os.urandom(16)  # Her sifreleme icin benzersiz rastgele IV
    cipher = Cipher(algorithms.AES(key), modes.GCM(iv))
    encryptor = cipher.encryptor()
    ciphertext = encryptor.update(plaintext) + encryptor.finalize()
    return {
        'iv': iv,
        'ciphertext': ciphertext,
        'tag': encryptor.tag
    }

# Anahtar uretimi: Kriptografik olarak guvenli rastgele
encryption_key = os.urandom(32)  # AES-256 icin 32 byte
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum kriptografik uygulamalarin envanterini cikar (hash, sifreleme, anahtar yonetimi)
*   [ ] Kullanilan algoritmalari ve anahtar uzunluklarini belirle
*   [ ] Duz metin veya zayif hashlenmiş parolalari tespit et

**Duzeltme**
*   [ ] MD5/SHA1 kullanan parola hashleme mekanizmalarini bcrypt/Argon2 ile degistir
*   [ ] Eskimis sifreleme algoritmalarini (DES, RC4) AES-256-GCM ile degistir
*   [ ] Sabit sifreleme anahtarlarini tespit et ve anahtar yonetim sistemiyle degistir
*   [ ] Sabit IV kullanimini rastgele IV uretimi ile degistir
*   [ ] SSL/TLS yapilandirmasini guclendir (TLS 1.2+ zorunlu, zayif cipher'lari kaldir)
*   [ ] Kriptografik olarak guvenli rastgele sayi ureticisi kullan

**Dogrulama**
*   [ ] SSL/TLS yapilandirmasini ssllabs.com ile test et (A+ hedefle)
*   [ ] Kriptografik uygulamalari kod incelemesi ile dogrula
*   [ ] Anahtar rotasyon sureclerinin calistigini test et
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Zayif sifreleme algoritmasi kullanimiyla ilgili uyarilari izle
    *   Cok sayida basarisiz parola dogrulama denemesini tespit et (brute-force gostergesi)
    *   SSL/TLS handshake hatalarini ve zayif cipher kullanımini izle
    *   Regex: `(?i)(md5|sha1|des|rc4|blowfish|weak.*(cipher|hash|encrypt))`
*   **Anomali Tespiti:**
    *   Kisa surede cok sayida hash hesaplama istegi (rainbow table/brute-force gostergesi)
    *   SSL sertifika suresi dolmak uzere olan servislerin tespiti
    *   Anahtar yonetim sisteminde beklenmeyen erisim denemeleri
    *   Zayif TLS surumleriyle (TLS 1.0/1.1) baglanti denemeleri

---

## Notlar
- OWASP Top 10:2025'de 4. sirada yer alir.
- Detayli rehber: [A04_Cryptographic_Failures.md](../OWASP_Top_10/A04_Cryptographic_Failures.md)
