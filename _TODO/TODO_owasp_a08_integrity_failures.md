# TODO: OWASP A08:2025 — Yazilim veya Veri Butunlugu Hatalari

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A08:2025 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Tedarik_Zinciri/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A08_Software_or_Data_Integrity_Failures.md](../OWASP_Top_10/A08_Software_or_Data_Integrity_Failures.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Yazilim veya Veri Butunlugu Hatalari, yazilim guncellemeleri, kritik veriler ve CI/CD boru hatlari gibi bilesenlerin butunluk dogrulamasi yapilmadan kullanilmasi durumunda ortaya cikan guvenlik zafiyetlerini kapsar. Imzasiz kod veya guncellemeler, guvensiz deserialization ve checksum dogrulamasi eksikligi bu kategorinin baslica sorunlaridir. OWASP Top 10:2025'de 8. sirada yer alir.
*   **Etkilenen bilesenler:** Yazilim guncelleme mekanizmalari, CI/CD boru hatlari, deserialization islemleri, paket yoneticileri, veri butunlugu kontrolleri

---

### 2. Teknik detay (nasil calisiyor)
*   Yazilim veya veri butunlugu hatalari, yazilim ve verilerin butunluk dogrulamasi yapilmadan islenmesi veya dagitilmasi durumunda ortaya cikar.
*   Yaygın ortaya cikis bicimleri:
    *   Imzasiz kod veya guncelleme dosyalarinin dagitilmasi
    *   Guvensiz deserialization (ornegin Python pickle, Java ObjectInputStream)
    *   Checksum veya hash dogrulamasi yapilmadan dosya indirilmesi
    *   CI/CD boru hatlarinda butunluk kontrollerinin eksikligi
    *   Ucuncu parti bagimliliklarin dogrulanmadan kullanilmasi
*   **Neden:** Yazilim gelistirme ve dagitim sureclerinde butunluk dogrulamasi adimlarinin atlanmasi, guvensiz deserialization kutuphanelerinin kullanilmasi ve tedarik zinciri guvenliginin ihmal edilmesi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Yazilim guncelleme mekanizmasi
*   **2. Normal durum:**
    ```
    1. Uygulama, guncelleme sunucusundan yeni surum dosyasini indirir
    2. Indirilen dosya uygulanir ve yazilim guncellenir
    3. Kullanici yeni surumu kullanmaya baslar
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, ag trafikinde man-in-the-middle (MITM) konumuna gecer
    2. Guncelleme istegini yakalar ve kendi hazirladigi zararli dosya ile degistirir
    3. Uygulama, imza dogrulamasi yapmadigi icin zararli dosyayi indirir ve calistirir
    4. Saldirganin kodu hedef sistemde calisir
    ```
*   **4. Analiz:** Guncelleme mekanizmasi, indirilen dosyanin dijital imzasini veya hash degerini dogrulamadigindan, saldirgan meşru guncelleme dosyasini kendi zararli dosyasiyla degistirebilir.
*   **5. Kanit:** Saldirganin hazirladigi zararli kod, guncelleme sureci sirasinda hedef sistemde calistirilir ve tam erisim saglanir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, CI/CD boru hatlari, Tedarik zinciri
*   **Karmasiklik:** Orta

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Mevcut guncelleme mekanizmalarinda dijital imza dogrulamasi ekleme, guvensiz deserialization kullanimlarini tespit etme ve devre disi birakma, kritik bagimliliklarin checksum dogrulamasini yapma.
*   **Orta vadeli:** CI/CD boru hatlarinda butunluk kontrolleri uygulama, guvenli deserialization kutuphanelerine gecis (pickle yerine JSON), SHA-256 ve HMAC tabanli dogrulama mekanizmalari kurma.
*   **Uzun vadeli:** Tedarik zinciri guvenlik politikasi olusturma, SBOM (Software Bill of Materials) uygulamasi baslatma, kod imzalama altyapisi kurma, duzenli bagimlIlik guvenlik taramalari.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
import pickle
import urllib.request

# Guvensiz guncelleme indirme — imza dogrulamasi yok
def download_update(url):
    response = urllib.request.urlopen(url)
    data = response.read()
    with open('update.bin', 'wb') as f:
        f.write(data)
    return True

# Guvensiz deserialization — pickle ile keyfi kod calistirma mumkun
def load_user_data(data_bytes):
    return pickle.loads(data_bytes)
```

**Guvenli kod:**
```python
import json
import hashlib
import urllib.request
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import padding

# Guvenli guncelleme indirme — RSA dijital imza dogrulamasi
def download_update_secure(url, signature_url, public_key_path):
    # Guncelleme dosyasini indir
    response = urllib.request.urlopen(url)
    data = response.read()

    # Imza dosyasini indir
    sig_response = urllib.request.urlopen(signature_url)
    signature = sig_response.read()

    # Acik anahtari yukle
    with open(public_key_path, 'rb') as key_file:
        public_key = serialization.load_pem_public_key(key_file.read())

    # Dijital imza dogrulamasi
    try:
        public_key.verify(
            signature,
            data,
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )
    except Exception as e:
        raise ValueError(f"Imza dogrulamasi basarisiz: {e}")

    # SHA-256 checksum dogrulamasi
    calculated_hash = hashlib.sha256(data).hexdigest()
    print(f"Guncelleme dosyasi SHA-256: {calculated_hash}")

    with open('update.bin', 'wb') as f:
        f.write(data)
    return True

# Guvenli deserialization — JSON kullanimi
def load_user_data_secure(data_bytes):
    # pickle yerine JSON ile guvenli deserialization
    return json.loads(data_bytes.decode('utf-8'))
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Mevcut butunluk dogrulama mekanizmalarinin envanterini cikar
*   [ ] Deserialization kullanimlarini tespit et
*   [ ] CI/CD boru hatti guvenlik durumunu degerlendir
*   [ ] Ucuncu parti bagimliliklarin listesini olustur

**Duzeltme**
*   [ ] Tum guncelleme mekanizmalarina dijital imza dogrulamasi ekle
*   [ ] Guvensiz deserialization kullanimlarini guvenli alternatiflere gecir (pickle -> JSON)
*   [ ] SHA-256 ve HMAC tabanli checksum dogrulamasi uygula
*   [ ] CI/CD boru hatlarinda butunluk kontrolleri kur
*   [ ] Bagimliliklari pinleme ve hash dogrulamasi ekle

**Dogrulama**
*   [ ] Imza dogrulama mekanizmasini test et
*   [ ] Deserialization guvenlik testleri gerceklestir
*   [ ] CI/CD boru hatti guvenlik taramasi yap
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Imza dogrulama hatalarini izle
    *   Deserialization islemlerindeki anormallikleri tespit et
    *   CI/CD boru hatlarindaki yetkisiz degisiklikleri izle
    *   Regex: `(?i)(signature verification failed|invalid checksum|deserialization error|untrusted source)`
*   **Anomali Tespiti:**
    *   Beklenmeyen kaynaklardan gelen guncelleme dosyalari
    *   Deserialization islemlerinde anormal veri kaliplari
    *   CI/CD boru hatlarinda yetkisiz degisiklikler
    *   Bagimliliklarda beklenmeyen surum degisiklikleri

---

## Notlar
- OWASP Top 10:2025'de 8. sirada yer alir.
- Detayli rehber: [A08_Software_or_Data_Integrity_Failures.md](../OWASP_Top_10/A08_Software_or_Data_Integrity_Failures.md)
