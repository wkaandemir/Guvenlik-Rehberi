# Guvenlik Kutuphanesi (AppSec Playbook)

Bu depo, **yapay zeka destegiyle yazilim gelistirenler** ve **yazilim guvenligine yeni baslayanlar** icin olusturulmus, yasayan bir guvenlik kutuphanesidir.

**Amaci:** Izlenen yayinlar, okunan makaleler ve guncel guvenlik aciklarini belirli bir sablona gore raporlayarak, teknik derinlikte bogulmadan herkesin anlayabilecegi ve uygulayabilecegi pratik bir rehber sunmaktir. Icerik zamanla, yeni arastirmalarla birlikte zenginlesecektir.

Depo icerisinde teknik guvenlik analizleri, zafiyet incelemeleri, kontrol listeleri (checklists) ve guvenli kodlama rehberleri bulunur.

## Klasor Yapisi

Depo, iki katmanli bir yapiyla organize edilmistir:

```text
Guvenlik-Rehberi/
├── OWASP_Top_10/           # OWASP Top 10 2025 guvenlik aciklari (A01-A10)
├── Enjeksiyon/              # SQL Injection, Command Injection rehberleri
├── Erisim_Kontrolu/         # Broken Access Control, IDOR rehberleri
├── Oturum_Yonetimi/         # CSRF, Session Management rehberleri
├── Arastirmalar/            # Derinlemesine arastirma ve raporlar
├── _TODO/                   # CVE-bazli zafiyet takip dosyalari (59 TODO)
├── _Sablonlar/              # Dokuman sablonlari
└── .claude/                 # Claude Code komutlari
```

**Katman 1 — Rehber dokumanlar:** Kategori klasorlerinde (`OWASP_Top_10/`, `Enjeksiyon/` vb.) genel guvenlik rehberleri yer alir.

**Katman 2 — Zafiyet takibi:** `_TODO/` klasorunde her CVE/zafiyet icin ayri bir analiz dosyasi tutulur. Kategori bilgisi dosya icerisindeki metadata alaninda belirtilir.

## TODO Takip Sistemi

Arastirma dokumanlarindan cikarilan her zafiyet/risk icin `_TODO/` klasorunde 9 bolumlu detayli bir analiz dosyasi bulunur:

1. Meta Bilgiler — 2. Ozet ve etki — 3. Teknik detay — 4. PoC (kavramsal) — 5. Risk degerlendirmesi — 6. Cozumler — 7. Ornek kod — 8. Checklist — 9. Izleme ve uyarilar

Genel durum ozeti icin [`_TODO/OZET.md`](_TODO/OZET.md) dosyasina bakin.

Yeni bir arastirma islemek icin Claude Code'da:
```
/arastirma-isle Arastirmalar/dosya.md
```

## Mevcut Icerikler

### OWASP Top 10 2025
| # | Baslik | Dosya |
|---|--------|-------|
| A01 | Kirik Erisim Kontrolu (Broken Access Control) | [A01](OWASP_Top_10/A01_Broken_Access_Control.md) |
| A02 | Guvenlik Yanlis Yapilandirmasi (Security Misconfiguration) | [A02](OWASP_Top_10/A02_Security_Misconfiguration.md) |
| A03 | Yazilim Tedarik Zinciri Hatalari (Software Supply Chain Failures) | [A03](OWASP_Top_10/A03_Software_Supply_Chain_Failures.md) |
| A04 | Kriptografik Hatalar (Cryptographic Failures) | [A04](OWASP_Top_10/A04_Cryptographic_Failures.md) |
| A05 | Enjeksiyon (Injection) | [A05](OWASP_Top_10/A05_Injection.md) |
| A06 | Guvensiz Tasarim (Insecure Design) | [A06](OWASP_Top_10/A06_Insecure_Design.md) |
| A07 | Kimlik Dogrulama Hatalari (Authentication Failures) | [A07](OWASP_Top_10/A07_Authentication_Failures.md) |
| A08 | Yazilim veya Veri Butunlugu Hatalari (Software or Data Integrity Failures) | [A08](OWASP_Top_10/A08_Software_or_Data_Integrity_Failures.md) |
| A09 | Guvenlik Kayit Tutma ve Uyari Hatalari (Security Logging and Alerting Failures) | [A09](OWASP_Top_10/A09_Security_Logging_and_Alerting_Failures.md) |
| A10 | Istisnai Durumlarin Yanlis Ele Alinmasi (Mishandling of Exceptional Conditions) | [A10](OWASP_Top_10/A10_Mishandling_of_Exceptional_Conditions.md) |

### Enjeksiyon
- [MySQL2 `escape` / Tip Manipulasyonu (Node.js)](Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md)
- [Django ORM Connector SQL Injection — CVE-2025-64459](Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md)
- [SQL Injection ile Veritabani Sorgularinin Kontrol Edilmesi](Enjeksiyon/sql_injection_veritabani_sorgulari.md)

### Erisim Kontrolu
- [IDOR ve Fonksiyon Seviyesi Erisim Kontrolu Eksikligi](Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md)

### Oturum Yonetimi
- [CSRF ve Oturum Kacirma Zafiyeti](Oturum_Yonetimi/csrf_ve_oturum_kacirma.md)

### Arastirmalar
- [Ocak 2026 Guvenlik Aciklari Arastirmasi](Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) — 24 CVE/risk analizi
- [Node.js Guvenlik Acigi Arastirmasi](Arastirmalar/Nodejs_Guvenlik_Acigi_Arastirmasi.md) — React2Shell, Next.js, V8 HashDoS
- [Keycloak Zafiyet Arastirmasi ve Analizi](Arastirmalar/Keycloak_Zafiyet_Arastirmasi_ve_Analizi.md) — CVE-2026-1529
- [Vibecoding Guvenlik Aciklari Arastirmasi](Arastirmalar/Vibecoding_Guvenlik_Aciklari_Arastirmasi.md) — Cursor, Copilot, Claude Code, MCP, OWASP ASI

### TODO Zafiyet Analizleri

Isletim Sistemi, Donanim, Mobil, Guvenlik Yapilandirmasi, AI IDE Guvenligi ve diger CVE-bazli analizler `_TODO/` klasorunde 8 bolumlu detayli formatta tutulmaktadir. Tam liste ve kritiklik dagilimi icin [`_TODO/OZET.md`](_TODO/OZET.md) dosyasina bakin.

| Kritiklik | Sayi |
|-----------|------|
| Kritik | 23 |
| Yuksek | 29 |
| Orta | 7 |

## Katki

Yeni dokuman eklerken:
- **Kategori rehberleri:** `[Kategori]/dosya_adi.md` formatinda, ilgili klasore ekleyin
- **Zafiyet analizi:** `_TODO/TODO_[slug].md` formatinda, `_Sablonlar/todo_sablonu.md` sablonunu kullanin
- Dosya adlarinda Turkce karakter kullanmayin (ascii only)
- CVE numaralarini her zaman belirtin

## Lisans

Bu depo egitim ve savunma amacli icerik barindirmaktadir. PoC bolumleri yalnizca egitim amaclidir.
