# TODO: Ledger / Trust Wallet Tedarik Zinciri — Kripto Cuzdani Tedarik Zinciri Ihlali

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | Ledger / Trust Wallet Tedarik Zinciri |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | YENI: Tedarik_Zinciri/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Ocak 2026'da kripto para ekosisteminde iki ayri tedarik zinciri ihlali yasanmistir. Kripto donanim cuzdani ureticisi Ledger, ucuncu taraf odeme islemcisi Global-e'nin hacklenmesi sonucu musteri verilerini (isim, siparis bilgileri) sizdirmistir. Ayri bir olayda, Binance istiraki Trust Wallet'ta "Shai-Hulud 2.0" adli kendini kopyalayan bir solucan (worm) uzerinden kullanicilardan 8.5 milyon dolar calinmistir. Bu olaylar, finansal kuruluslarin yalnizca kendi sistemlerini degil, tum dijital tedarik zincirlerini korumak zorunda oldugunu kanitlamaktadir.
*   **Etkilenen bilesenler:** Ledger musteri veritabani (isim, siparis bilgileri), Global-e odeme islemci altyapisi, Trust Wallet mobil uygulamasi ve kullanici fonlari, Binance ekosistemi

---

### 2. Teknik detay (nasil calisiyor)
*   **Ledger / Global-e:** Ledger, odeme islemlerini ucuncu taraf saglayici Global-e uzerinden yurutmektedir. Saldirganlar, Global-e'nin altyapisini hedef alarak musteri isim ve siparis bilgilerine erisim saglamistir. Ledger'in kendi sistemleri dogrudan ihlal edilmemis, ancak tedarik zincirindeki zayif halka uzerinden veriler sizdirilmistir.
*   **Trust Wallet / Shai-Hulud 2.0:** "Shai-Hulud 2.0" adli kendini kopyalayan bir solucan, Trust Wallet kullanicilarin islem onay mekanizmalarini manipule ederek fonlari saldirganin adresine yonlendirmistir. Solucan, cuzdan uygulamasinin islem imzalama surecindeki dogrulama eksikliklerinden faydalanmistir.
*   Her iki olayda da asil hedef (Ledger, Trust Wallet) dogrudan saldirilmamis, bunun yerine ucuncu taraf entegrasyonlari ve bagimliliklari istismar edilmistir.
*   **Neden:** Kok neden olarak ucuncu taraf tedarikci guvenlik denetimlerinin yetersizligi, API erisim izinlerinin genis tutulmasi, webhook ve odeme islemci entegrasyonlarinda guvenlik dogrulamasinin eksikligi ve tedarik zinciri risk yonetiminin ihmal edilmesi gosterilebilir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Kripto cuzdan ekosistemi — ucuncu taraf odeme islemcisi (Ledger/Global-e) ve mobil cuzdan uygulamasi (Trust Wallet).
*   **2. Normal durum:**
    ```
    Ledger senaryosu:
    1. Musteri Ledger web sitesinden donanim cuzdani siparis eder
    2. Odeme islemcisi Global-e, odeme bilgilerini guvenli bir sekilde isler
    3. Siparis bilgileri sifrelenmis kanallar uzerinden iletilir

    Trust Wallet senaryosu:
    1. Kullanici Trust Wallet uzerinden kripto transfer yapar
    2. Islem detaylari kullaniciya gosterilir ve onay istenir
    3. Imzalanan islem blockchain agina gonderilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    Ledger senaryosu:
    1. Saldirgan, Global-e altyapisina sizar (API zafiyeti veya kimlik hirsizligi)
    2. Odeme islemcisinin veritabanindaki Ledger musteri kayitlarina erisir
    3. Isim, adres ve siparis bilgilerini toplu olarak disari cikarir

    Trust Wallet senaryosu:
    1. Shai-Hulud 2.0 solucani kullanicinin cihazina ulasir
    2. Solucan, Trust Wallet islem onay ekranindaki alici adresini saldirganinkiyle degistirir
    3. Kullanici gorsel olarak dogru gorunen ancak manipule edilmis islemi onaylar
    4. Solucan kendini diger cihazlara kopyalamak icin ag taramasi yapar
    ```
*   **4. Analiz:** Ledger olayinda, tedarik zincirindeki zayif halka (ucuncu taraf odeme islemcisi) hedef alinarak ana sirketin kendi guvenlik kontrolleri atlatilmistir. Trust Wallet olayinda ise kullanici arayuzundeki islem dogrulama eksikligi istismar edilmistir. Her iki olay da "dogrudan saldiramiyorsan, bagimliligini saldiri" stratejisinin ornekleridir.
*   **5. Kanit:** Ledger, musteri verilerinin sizdigini resmi olarak kabul etmistir. Trust Wallet kullanicilarindan toplam 8.5 milyon dolar calinmistir. Shai-Hulud 2.0 solucani guvenlik arastirmacilari tarafindan analiz edilmistir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** Uygulanamaz (tedarik zinciri ihlali, belirli bir CVE degil)
*   **Saldiri Yuzeyi:** Internete acik (ucuncu taraf API'leri ve odeme entegrasyonlari)
*   **Karmasiklik:** Orta (ucuncu taraf altyapisina sizma gerektirir; solucan dagitimi ise daha dusuk karmasiklikta)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Tum ucuncu taraf API entegrasyonlarini envanterleyin ve erisim izinlerini gozden gecirin. Global-e gibi odeme islemcileri ile veri paylasim kapsamini minimize edin. Etkilenen musteri hesaplarina bildirim gonderin ve kimlik avina karsi uyari yayin. Trust Wallet kullanicilarini guncel surume yonlendirin ve 2FA'yi zorunlu kilin.
*   **Orta vadeli:** Tedarik zinciri guvenlik denetim sureclerini olusturun — her ucuncu taraf saglayicisi icin yillik guvenlik degerlendirmesi yapin. Webhook ve API entegrasyonlarinda imza dogrulamasi (HMAC-SHA256) zorunlu kilin. API anahtarlarini periyodik olarak dondurun (90 gunluk rotasyon). Islem imzalama surecinde coklu dogrulama katmani ekleyin (hardware wallet confirmation + UI dogrulama).
*   **Uzun vadeli:** Yazilim Materyal Listesi (SBOM) kullanimi ile tum ucuncu taraf bagimliliklarini seffaflastirin. Tedarikci guvenlik standartlari (SOC 2 Type II, ISO 27001) cercevesinde sozlesmeler yapin. Sifir Guven mimarisini ucuncu taraf entegrasyonlarina kadar genisletin. Kripto islemler icin bagimsiz dogrulama katmanlari (coklu imza, zaman gecikmelı onay) uygulayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```python
# Ucuncu taraf webhook dogrulamasi OLMADAN islem isleme — zafiyetli
import json
from flask import Flask, request

app = Flask(__name__)

@app.route("/webhook/payment", methods=["POST"])
def handle_payment_webhook():
    # HATALI: Imza dogrulamasi yok — herhangi biri sahte webhook gonderebilir
    data = request.get_json()
    order_id = data["order_id"]
    status = data["status"]
    # Dogrudan veritabanina yaz
    db.execute(
        "UPDATE orders SET status = %s WHERE id = %s",
        (status, order_id)
    )
    return "OK", 200
```

**Guvenli kod:**
```python
# Ucuncu taraf webhook dogrulamasi ILE guvenli islem isleme
import hmac
import hashlib
import json
import time
from flask import Flask, request, abort

app = Flask(__name__)

WEBHOOK_SECRET = vault_client.get_secret("payment/webhook_secret")
MAX_TIMESTAMP_DRIFT = 300  # 5 dakika

def verify_webhook_signature(payload: bytes, signature: str, timestamp: str) -> bool:
    """Ucuncu taraf webhook imzasini dogrula."""
    # 1. Zaman damgasi kontrolu (replay attack onlemi)
    if abs(time.time() - int(timestamp)) > MAX_TIMESTAMP_DRIFT:
        return False
    # 2. HMAC-SHA256 imza dogrulamasi
    expected = hmac.new(
        WEBHOOK_SECRET.encode(), payload, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature)

@app.route("/webhook/payment", methods=["POST"])
def handle_payment_webhook():
    # Imza ve zaman damgasi dogrula
    signature = request.headers.get("X-Webhook-Signature", "")
    timestamp = request.headers.get("X-Webhook-Timestamp", "0")

    if not verify_webhook_signature(request.data, signature, timestamp):
        abort(403, "Gecersiz webhook imzasi")

    data = request.get_json()
    order_id = data.get("order_id")
    status = data.get("status")

    # Parametre dogrulamasi
    if not order_id or status not in ("completed", "failed", "refunded"):
        abort(400, "Gecersiz parametreler")

    # Guvenli veritabani islemi
    db.execute(
        "UPDATE orders SET status = %s WHERE id = %s AND status = 'pending'",
        (status, order_id)
    )
    return "OK", 200


# API anahtari rotasyonu icin otomasyon
def rotate_api_key(provider: str, key_id: str) -> str:
    """Ucuncu taraf API anahtarini periyodik olarak dondur."""
    key_age = get_key_age(provider, key_id)
    if key_age.days > 90:
        new_key = provider_api.rotate_key(key_id)
        vault_client.put_secret(f"api/{provider}", new_key)
        revoke_old_key(provider, key_id)
        return new_key
    return None
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum ucuncu taraf entegrasyonlarini envanterle (odeme islemcileri, API'ler, SDK'lar)
*   [ ] Her tedarikci icin paylasilan veri kapsamini belgele
*   [ ] Mevcut tedarikci guvenlik degerlendirme raporlarini gozden gecir
*   [ ] API anahtarlarinin yasini ve rotasyon durumunu kontrol et

**Duzeltme**
*   [ ] Webhook entegrasyonlarina HMAC-SHA256 imza dogrulamasi ekle
*   [ ] API erisim izinlerini en az yetki prensibiyle kisitla
*   [ ] API anahtarlari icin 90 gunluk otomatik rotasyon uygulamasi kur
*   [ ] Ucuncu taraf veri paylasim kapsamini minimize et (yalnizca gerekli alanlar)
*   [ ] Kripto islemlerde coklu imza (multi-signature) dogrulamasi uygulayin

**Dogrulama**
*   [ ] Webhook imza dogrulamasinin sahte istekleri reddettiyini test et
*   [ ] API erisim kisitlamalarinin dogru calistigini dogrula
*   [ ] Tedarikci guvenlik uyumlulugunu (SOC 2, ISO 27001) dogrula
*   [ ] Islem manipulasyonu senaryolarini simule et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Ucuncu taraf API'lerden gelen anormal hacimli veri isteklerini izle
    *   Webhook imza dogrulama basarisizliklarini logla ve izle
    *   API anahtari kullaniminda anormal patern degisikliklerini tespit et
    *   Regex: `(?i)(webhook.*signature.*fail|invalid.*hmac|api.*key.*rotat|unauthorized.*api)`
    *   Regex (kripto islemi): `(?i)(transaction.*redirect|address.*change|wallet.*drain)`
*   **Anomali Tespiti:**
    *   Tek bir ucuncu taraf API anahtariyla kisa surede normalin uzerinde istek sayisi
    *   Webhook kaynaginin IP adresinin bilinen tedarikci IP araliginin disina cikmasi
    *   Kripto islemlerinde alici adresinin son anda degismesi
    *   Ucuncu taraf servislerden gelen yanit surelerinde ani degisimler (MITM belirtisi)
    *   API anahtari rotasyonu yapilmadan 90 gunun asilmasi

---

## Notlar
Finansal kuruluslarin sadece kendi sistemlerini degil, tum dijital ekosistemlerini korumalari gerektir. Ledger olayinda asil hedef dogrudan saldiri almamis, ucuncu taraf odeme islemcisi (Global-e) uzerinden veri sizintisi gerceklesmistir. Trust Wallet'ta Shai-Hulud 2.0 solucani 8.5 milyon dolar hasar vermistir. Oracle January 2026 CPU'sundaki Apache Tika zafiyeti (CVE-2025-66516, CVSS 10.0) ile birlikte, "CVE Amplifikasyonu" kavrami altinda tedarik zinciri risklerinin sistematik olarak ele alinmasi gerekmektedir.
