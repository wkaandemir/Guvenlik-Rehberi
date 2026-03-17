# TODO: OWASP A02:2025 — Guvenlik Yanlis Yapilandirmasi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A02:2025 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Guvenlik_Yapilandirmasi/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A02_Security_Misconfiguration.md](../OWASP_Top_10/A02_Security_Misconfiguration.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Guvenlik Yanlis Yapilandirmasi, uygulama, sunucu, veritabani veya bulut altyapisinda varsayilan veya hatali yapilandirmalarin kullanilmasi sonucu ortaya cikan guvenlik aciklaridir. Varsayilan parolalar, acik kalan debug modlari, eksik guvenlik basliklari ve hatali bulut depolama erisim izinleri bu kategorinin en yaygin ornekleridir. OWASP Top 10:2025'de 2. sirada yer almaktadir.
*   **Etkilenen bilesenler:** Web sunuculari (Nginx, Apache, IIS), uygulama sunuculari, veritabani sistemleri, bulut depolama servisleri (S3, Azure Blob), CI/CD pipeline'lari, konteyner ortamlari, SSL/TLS yapilandirmalari, guvenlik basliklari

---

### 2. Teknik detay (nasil calisiyor)
*   Guvenlik yanlis yapilandirmasi, sistemlerin varsayilan ayarlarla uretim ortamina alinmasi veya guvenlik ayarlarinin bilincsizce gevsetilmesi durumunda ortaya cikar.
*   Yaygın ortaya cikis bicimleri:
    *   Varsayilan kullanici adi ve parolalarin degistirilmemesi (admin/admin, root/root)
    *   Uretim ortaminda debug veya hata ayiklama modunun acik birakilmasi
    *   HTTP guvenlik basliklarinin (X-Frame-Options, CSP, HSTS) eksik olmasi
    *   Bulut depolama birimlerinin herkese acik erisimle yapilandirilmasi
    *   SSL/TLS yapilandirmasinda zayif sifreleme takimlarinin kullanilmasi
    *   Gereksiz servislerin ve portlarin acik birakilmasi
*   **Neden:** Guvenlik yapilandirmalarinin sistematik olarak yonetilmemesi, uretim oncesi kontrol listelerinin eksikligi, Infrastructure as Code (IaC) yaklasiminin benimsenmemesi ve yapilandirma sapmalarinin izlenmemesi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Uretim ortaminda debug modu acik birakilan bir web uygulamasi
*   **2. Normal durum:**
    ```
    GET /api/status HTTP/1.1
    Host: hedef-uygulama.com

    HTTP/1.1 200 OK
    {"status": "ok"}
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    GET /api/status?debug=true HTTP/1.1
    Host: hedef-uygulama.com

    HTTP/1.1 200 OK
    {
      "status": "ok",
      "debug": {
        "db_host": "10.0.1.5",
        "db_user": "app_prod",
        "db_pass": "P@ssw0rd123!",
        "stack_trace": "...",
        "environment": "production"
      }
    }
    ```
*   **4. Analiz:** Debug modu acik birakildigi icin saldirgan, `debug=true` parametresi ekleyerek veritabani baglanti bilgileri, ortam degiskenleri ve stack trace gibi hassas bilgilere erisir.
*   **5. Kanit:** Yanit icerisinde veritabani kimlik bilgileri, ic ag IP adresleri ve uygulama yapilandirma detaylari aciga cikar.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, Ic ag, Bulut ortami
*   **Karmasiklik:** Dusuk

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Tum varsayilan parolalari degistirme, uretim ortaminda debug modunu kapatma, kritik guvenlik basliklarini ekleme, herkese acik bulut depolama birimlerini kisitlama.
*   **Orta vadeli:** Guvenlik yapilandirma standartlari olusturma, gereksiz servisleri ve portlari kapatma, SSL/TLS yapilandirmasini guclendirme, CI/CD pipeline'ina guvenlik kontrolleri ekleme.
*   **Uzun vadeli:** Infrastructure as Code (IaC) yaklasimini benimseme, yapilandirma sapma izleme (drift detection) sistemi kurma, otomatik uyumluluk taramalari yapma, CIS Benchmark'larina uyum saglama.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```nginx
# Guvenlik basliklari eksik, debug bilgileri acik
server {
    listen 80;
    server_name hedef-uygulama.com;

    location / {
        proxy_pass http://backend:3000;
    }
}
```

**Guvenli kod:**
```nginx
server {
    listen 443 ssl http2;
    server_name hedef-uygulama.com;

    # SSL/TLS yapilandirmasi
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
    ssl_prefer_server_ciphers on;

    # Guvenlik basliklari
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Content-Security-Policy "default-src 'self';" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Sunucu bilgilerini gizle
    server_tokens off;
    proxy_hide_header X-Powered-By;

    location / {
        proxy_pass http://backend:3000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum sistem ve servislerin yapilandirma envanterini cikar
*   [ ] Varsayilan parola kullanan hesaplari tespit et
*   [ ] Acik debug modlarini ve gereksiz servisleri belirle

**Duzeltme**
*   [ ] Varsayilan parolalari guclu parolalarla degistir
*   [ ] Uretim ortaminda debug ve hata ayiklama modlarini kapat
*   [ ] HTTP guvenlik basliklarini ekle (X-Frame-Options, CSP, HSTS vb.)
*   [ ] SSL/TLS yapilandirmasini guclendir (zayif sifreleme takimlarini kaldir)
*   [ ] Gereksiz servisleri, portlari ve modulleri kapat
*   [ ] Bulut depolama erisim izinlerini kisitla

**Dogrulama**
*   [ ] Guvenlik basliklarini securityheaders.com ile dogrula
*   [ ] SSL/TLS yapilandirmasini ssllabs.com ile test et
*   [ ] Otomatik yapilandirma tarama araclari (ScoutSuite, Prowler) calistir
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Varsayilan parola ile basarili/basarisiz giris denemelerini izle
    *   Debug endpoint'lerine yapilan erisimleri tespit et
    *   Yapilandirma dosyalarindaki degisiklikleri izle
    *   Regex: `(?i)(debug|trace|verbose)\s*[=:]\s*(true|on|1|enabled)`
*   **Anomali Tespiti:**
    *   Varsayilan kullanici adlariyla giris denemeleri (admin, root, test)
    *   Uretim ortaminda beklenmeyen debug ciktilari
    *   Yapilandirma dosyalarinda yetkisiz degisiklikler
    *   Bulut depolama birimlerine beklenmeyen disaridan erisim

---

## Notlar
- OWASP Top 10:2025'de 2. sirada yer alir.
- Detayli rehber: [A02_Security_Misconfiguration.md](../OWASP_Top_10/A02_Security_Misconfiguration.md)
