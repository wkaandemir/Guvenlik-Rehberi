# Guvenlik Kutuphanesi (AppSec Playbook)

Bu depo, **yapay zeka destegiyle yazilim gelistirenler** ve **yazilim guvenligine yeni baslayanlar** icin olusturulmus, yasayan bir guvenlik kutuphanesidir.

**Amaci:** Izlenen yayinlar, okunan makaleler ve guncel guvenlik aciklarini belirli bir sablona gore raporlayarak, teknik derinlikte bogulmadan herkesin anlayabilecegi ve uygulayabilecegi pratik bir rehber sunmaktir. Icerik zamanla, yeni arastirmalarla birlikte yavas yavas zenginlesecektir.

Depo icerisinde teknik guvenlik analizleri, zafiyet incelemeleri, kontrol listeleri (checklists) ve guvenli kodlama rehberleri bulunur.

> **Daha kapsamli proje indeksi icin [INDEX.md](INDEX.md) dosyasina bakin.**

## Klasor Yapisi

Depo, iceriklerin kolay erisilebilir olmasi icin iki katmanli bir yapiyla organize edilmistir:

```text
Guvenlik-Rehberi/
├── OWASP_Top_10/           # OWASP Top 10 2025 guvenlik aciklari (A01-A10)
├── Enjeksiyon/              # SQL Injection, Command Injection rehberleri
├── Erisim_Kontrolu/         # Broken Access Control, IDOR rehberleri
├── Oturum_Yonetimi/         # CSRF, Session Management rehberleri
├── Arastirmalar/            # Derinlemesine arastirma ve raporlar
├── _TODO/                   # CVE-bazli zafiyet takip dosyalari (48 TODO)
├── _Sablonlar/              # Dokuman sablonlari
└── .claude/                 # Claude Code komutlari
```

## Dosya Isimlendirme Standardi ve Icerik

Yeni bir dokuman eklerken asagidaki standarda uyulmasi onerilir:

**Kategori rehberleri:** `[Kategori]/[Baslik]_[Teknoloji].md`
- `Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md`
- `Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md`

**TODO dosyalari:** `_TODO/TODO_[slug_snake_case].md`
- `_TODO/TODO_cve_2026_22708_cursor_shell_bypass.md`

### Icerik Nasil Olmali?
Her dokuman ideal olarak su bolumleri icermelidir:
1. **Ozet ve Etki:** Zafiyet nedir, etkisi nedir?
2. **Teknik Detay:** Nasil calisiyor?
3. **PoC (Proof of Concept):** Kavramsal istismar adimlari.
4. **Cozum ve Onlemler:** Kod ornekleriyle (Secure Coding) duzeltme.
5. **Checklist:** Kontrol listesi.

### TODO Takip Sistemi
Arastirma dokumanlarindan cikarilan her zafiyet/risk icin `_TODO/` klasorunde bir kontrol listesi (checklist) dosyasi bulunur. Genel durum ozeti icin [`_TODO/OZET.md`](_TODO/OZET.md) dosyasina bakin. Yeni bir arastirma islemek icin Claude Code'da `/arastirma-isle Arastirmalar/dosya.md` komutunu kullanabilirsiniz.

## Mevcut Icerikler

### OWASP Top 10 2025
* [Kirik Erisim Kontrolu (Broken Access Control)](OWASP_Top_10/A01_Broken_Access_Control.md)
* [Guvenlik Yanlis Yapilandirmasi (Security Misconfiguration)](OWASP_Top_10/A02_Security_Misconfiguration.md)
* [Yazilim Tedarik Zinciri Hatalari (Software Supply Chain Failures)](OWASP_Top_10/A03_Software_Supply_Chain_Failures.md)
* [Kriptografik Hatalar (Cryptographic Failures)](OWASP_Top_10/A04_Cryptographic_Failures.md)
* [Enjeksiyon (Injection)](OWASP_Top_10/A05_Injection.md)
* [Guvensiz Tasarim (Insecure Design)](OWASP_Top_10/A06_Insecure_Design.md)
* [Kimlik Dogrulama Hatalari (Authentication Failures)](OWASP_Top_10/A07_Authentication_Failures.md)
* [Yazilim veya Veri Butunlugu Hatalari (Software or Data Integrity Failures)](OWASP_Top_10/A08_Software_or_Data_Integrity_Failures.md)
* [Guvenlik Kayit Tutma ve Uyari Hatalari (Security Logging and Alerting Failures)](OWASP_Top_10/A09_Security_Logging_and_Alerting_Failures.md)
* [Istisnai Durumlarin Yanlis Ele Alinmasi (Mishandling of Exceptional Conditions)](OWASP_Top_10/A10_Mishandling_of_Exceptional_Conditions.md)

### Enjeksiyon
* [MySQL2 `escape` / tip-manipulasyonu](Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md) - Node.js MySQL2 kutuphanesinin tip manipulasyonu ile authentication bypass
* [Django ORM Connector SQL Injection — CVE-2025-64459](Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md)
* [SQL Injection ile Veritabani Sorgularinin Kontrol Edilmesi](Enjeksiyon/sql_injection_veritabani_sorgulari.md)

### Erisim Kontrolu
* [IDOR ve Fonksiyon Seviyesi Erisim Kontrolu Eksikligi](Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md)

### Oturum Yonetimi
* [CSRF ve Oturum Kacirma Zafiyeti](Oturum_Yonetimi/csrf_ve_oturum_kacirma.md)

### Arastirmalar
* [React2Shell (CVE-2025-55182) Kapsamli Tehdit Analizi](Arastirmalar/Node.js%20G%C3%BCvenlik%20A%C3%A7%C4%B1%C4%9F%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md)
* [Ocak 2026 Guvenlik Aciklari Arastirmasi](Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md)
* [Keycloak Zafiyet Arastirmasi ve Analizi — CVE-2026-1529](Arastirmalar/Keycloak%20Zafiyet%20Ara%C5%9Ft%C4%B1rmas%C4%B1%20ve%20Analizi.md)
* [Vibecoding Guvenlik Aciklari Arastirmasi](Arastirmalar/Vibecoding%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) - Cursor, Copilot, Claude Code, MCP, OWASP ASI

### TODO Zafiyet Analizleri

Isletim Sistemi, Donanim, Mobil, Guvenlik Yapilandirmasi, AI IDE Guvenligi ve diger CVE-bazli analizler `_TODO/` klasorunde 8 bolumlu detayli formatta tutulmaktadir. Tam liste icin [`_TODO/OZET.md`](_TODO/OZET.md) dosyasina bakin.
