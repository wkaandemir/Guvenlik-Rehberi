# Guvenlik-Rehberi Proje Indeksi

Bu dokuman, Guvenlik Rehberi projesinin kapsamli indeksini ve icerik haritasini sunar. Proje, yapay zeka destegiyle yazilim gelistirenler ve yazilim guvenligine yeni baslayanlar icin olusturulmus, yasayan bir guvenlik kutuphanesidir.

## Proje Hakkinda

**Amac:** Izlenen yayinlar, okunan makaleler ve guncel guvenlik aciklarini belirli bir sablona gore raporlayarak, teknik derinlikte bogulmadan herkesin anlayabilecegi ve uygulayabilecegi pratik bir rehber sunmaktir.

**Hedef Kitle:**
- Yapay zeka destegiyle yazilim gelistirenler
- Yazilim guvenligine yeni baslayanlar
- Guvenlik arastirmacilari
- DevOps ve Site Reliability muhendisleri

**Icerik Dili:** Turkce (varsayilan), Ingilizce (teknik terimler ve kaynaklar)

---

## Klasor Yapisi ve Organizasyon

Proje icerikleri iki katmandan olusur: kategori klasorleri genel rehber dokumanlari barindirir, `_TODO/` ise CVE-bazli detayli zafiyet analizlerini tutar.

```
Guvenlik-Rehberi/
├── OWASP_Top_10/           # OWASP Top 10 2025 guvenlik aciklari (A01-A10)
├── Enjeksiyon/              # SQL Injection, Command Injection rehberleri
├── Erisim_Kontrolu/         # Broken Access Control, IDOR rehberleri
├── Oturum_Yonetimi/         # CSRF, Session Management rehberleri
├── Arastirmalar/            # Derinlemesine arastirma ve raporlar
├── _TODO/                   # CVE-bazli zafiyet takip dosyalari (48 TODO)
├── _Sablonlar/              # Dokuman sablonlari
├── .claude/                 # Claude Code komutlari ve yapilandirma
├── AGENTS.md                # Repository yonergeleri
├── CLAUDE.md                # Claude Code is akisi talimatlari
├── README.md                # Ana proje aciklamasi
└── INDEX.md                 # Bu dosya - kapsamli proje indeksi
```

---

## Icerik Haritasi

### OWASP Top 10 2025

OWASP Top 10 2025 kategorileri, modern web uygulama guvenligindeki en kritik riskleri tanimlar. Her kategori icin detayli analizler mevcuttur:

| Kod | Baslik | Dokuman | CVSS |
|-----|--------|---------|------|
| A01 | Broken Access Control | [A01_Broken_Access_Control.md](OWASP_Top_10/A01_Broken_Access_Control.md) | Kritik |
| A02 | Security Misconfiguration | [A02_Security_Misconfiguration.md](OWASP_Top_10/A02_Security_Misconfiguration.md) | Yuksek |
| A03 | Software Supply Chain Failures | [A03_Software_Supply_Chain_Failures.md](OWASP_Top_10/A03_Software_Supply_Chain_Failures.md) | Yuksek |
| A04 | Cryptographic Failures | [A04_Cryptographic_Failures.md](OWASP_Top_10/A04_Cryptographic_Failures.md) | Yuksek |
| A05 | Injection | [A05_Injection.md](OWASP_Top_10/A05_Injection.md) | Kritik |
| A06 | Insecure Design | [A06_Insecure_Design.md](OWASP_Top_10/A06_Insecure_Design.md) | Orta |
| A07 | Authentication Failures | [A07_Authentication_Failures.md](OWASP_Top_10/A07_Authentication_Failures.md) | Yuksek |
| A08 | Software or Data Integrity Failures | [A08_Software_or_Data_Integrity_Failures.md](OWASP_Top_10/A08_Software_or_Data_Integrity_Failures.md) | Yuksek |
| A09 | Security Logging and Alerting Failures | [A09_Security_Logging_and_Alerting_Failures.md](OWASP_Top_10/A09_Security_Logging_and_Alerting_Failures.md) | Orta |
| A10 | Mishandling of Exceptional Conditions | [A10_Mishandling_of_Exceptional_Conditions.md](OWASP_Top_10/A10_Mishandling_of_Exceptional_Conditions.md) | Orta |

**Iliskili TODO Analizleri:**
- Apache Tika XXE - CVE-2025-66516 → [TODO](_TODO/TODO_cve_2025_66516_apache_tika_xxe.md)

---

### Enjeksiyon (Injection)

Enjeksiyon tipi zafiyetler, kullanici girdilerinin dogrulanmadan bir yorumlayiciya gonderilmesi sonucunda ortaya cikar.

| Dokuman | Teknoloji | CVE | Aciklama |
|---------|-----------|-----|----------|
| [MySQL2_Tip_Manipulasyonu_NodeJS.md](Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md) | Node.js, MySQL2 | - | Tip manipulasyonu ile authentication bypass |
| [django_orm_sql_injection_cve_2025_64459.md](Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md) | Django ORM | CVE-2025-64459 | ORM connector SQL injection |
| [sql_injection_veritabani_sorgulari.md](Enjeksiyon/sql_injection_veritabani_sorgulari.md) | Genel | - | SQL injection temel kavramlar |

**Iliskili TODO Analizleri:**
- CVE-2025-27209 Node.js V8 HashDoS → [TODO](_TODO/TODO_cve_2025_27209_nodejs_v8_hashdos.md)

**Iliskili Kategoriler:**
- OWASP A05: Injection
- Erisim Kontrolu: Authentication bypass senaryolari

---

### Erisim Kontrolu (Access Control)

Yetkisiz erisim kontrolu zafiyetleri, kullanici yetkilerinin dogru dogrulanmamasi sonucunda ortaya cikar.

| Dokuman | Teknoloji | CVE | Aciklama |
|---------|-----------|-----|----------|
| [idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md](Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md) | Genel | - | IDOR ve fonksiyon seviyesi erisim kontrolu |

**Iliskili TODO Analizleri:**
- CVE-2025-8110 Gogs Symlink → [TODO](_TODO/TODO_cve_2025_8110_gogs_symlink.md)
- CVE-2025-68664 LangGrinch → [TODO](_TODO/TODO_cve_2025_68664_langgrinch_langchain.md)
- CVE-2026-21945 Oracle MySQL SSRF → [TODO](_TODO/TODO_cve_2026_21945_oracle_mysql_java_ssrf.md)
- CVE-2025-27210 Node.js Path Traversal → [TODO](_TODO/TODO_cve_2025_27210_nodejs_windows_path_traversal.md)

**Iliskili Kategoriler:**
- OWASP A01: Broken Access Control
- Oturum Yonetimi: Yetkilendirme mekanizmalari

---

### Oturum Yonetimi (Session Management)

Oturum yonetimi zafiyetleri, kullanici oturumlarinin guvensiz olmasi veya manipule edilebilmesi durumlarini icerir.

| Dokuman | Teknoloji | CVE | Aciklama |
|---------|-----------|-----|----------|
| [csrf_ve_oturum_kacirma.md](Oturum_Yonetimi/csrf_ve_oturum_kacirma.md) | Genel | - | CSRF ve oturum kacirma zafiyeti |

**Iliskili TODO Analizleri:**
- CVE-2025-55182 React2Shell → [TODO](_TODO/TODO_cve_2025_55182_react2shell_rce.md)
- CVE-2025-66478 Next.js RCE → [TODO](_TODO/TODO_cve_2025_66478_nextjs_rce.md)
- CVE-2026-21509 Office OLE/COM → [TODO](_TODO/TODO_cve_2026_21509_office_ole_com_bypass.md)

**Iliskili Kategoriler:**
- OWASP A07: Authentication Failures
- Arastirmalar: [Node.js Guvenlik Acigi Arastirmasi](Arastirmalar/Node.js%20G%C3%BCvenlik%20A%C3%A7%C4%B1%C4%9F%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md)

---

### Arastirmalar (Research Reports)

Derinlemesine guvenlik arastirmalari ve kapsamli tehdit analizleri icerir.

| Dokuman | Konu | Aciklama |
|---------|------|----------|
| [Node.js Guvenlik Acigi Arastirmasi.md](Arastirmalar/Node.js%20G%C3%BCvenlik%20A%C3%A7%C4%B1%C4%9F%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) | React2Shell | CVE-2025-55182 ve CVE-2025-66478 kapsamli tehdit analizi |
| [Ocak 2026 Guvenlik Aciklari Arastirmasi.md](Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) | Aylik Ozet | Ocak 2026 guvenlik aciklari ozet raporu |
| [Keycloak Zafiyet Arastirmasi ve Analizi.md](Arastirmalar/Keycloak%20Zafiyet%20Ara%C5%9Ft%C4%B1rmas%C4%B1%20ve%20Analizi.md) | Keycloak | CVE-2026-1529 organizasyon davet/katilim akisi zafiyet analizi |
| [Vibecoding Guvenlik Aciklari Arastirmasi.md](Arastirmalar/Vibecoding%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) | Vibecoding / AI-IDE | Vibecoding platformlari sistemik guvenlik analizi (Cursor, Copilot, Claude Code, MCP, OWASP ASI) |

**Iliskili Dokumanlar:**
- OWASP A03: Software Supply Chain Failures
- OWASP ASI Top 10 (2026): Agentic Applications guvenlik standardi

---

## TODO Takip Sistemi

Arastirma dokumanlarindan cikarilan her zafiyet/risk icin `_TODO/` klasorunde ayri bir kontrol listesi dosyasi bulunur. Genel durum ozeti icin [`_TODO/OZET.md`](_TODO/OZET.md) dosyasina bakin.

| Bilgi | Deger |
|-------|-------|
| **Konum** | `_TODO/` klasoru |
| **Sablon** | [todo_sablonu.md](_Sablonlar/todo_sablonu.md) |
| **Ozet Panosu** | [OZET.md](_TODO/OZET.md) |
| **Tetikleme** | `/arastirma-isle Arastirmalar/dosya.md` veya "Bu arastirmayi isle" |
| **Durum** | ACIK / KAPANDI |
| **Is Akisi** | [CLAUDE.md](CLAUDE.md) |

### Kaynak Arastirmalar ve TODO Sayilari

| Arastirma Dokumani | TODO Sayisi |
|---------------------|-------------|
| [Ocak 2026 Guvenlik Aciklari Arastirmasi](Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) | 24 |
| [Vibecoding Guvenlik Aciklari Arastirmasi](Arastirmalar/Vibecoding%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) | 19 |
| [Node.js Guvenlik Acigi Arastirmasi](Arastirmalar/Node.js%20G%C3%BCvenlik%20A%C3%A7%C4%B1%C4%9F%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) | 4 |
| [Keycloak Zafiyet Arastirmasi ve Analizi](Arastirmalar/Keycloak%20Zafiyet%20Ara%C5%9Ft%C4%B1rmas%C4%B1%20ve%20Analizi.md) | 1 |
| **Toplam** | **48** |

### Kategori Dagilimi

TODO dosyalarindaki kategori etiketlerine gore dagilim:

| Kategori | Sayi | Durum |
|----------|------|-------|
| AI_IDE_Guvenligi | 12 | Metadata etiketi |
| Isletim_Sistemi | 10 | Metadata etiketi |
| Guvenlik_Yapilandirmasi | 7 | Metadata etiketi |
| Erisim_Kontrolu | 4 | Fiziksel klasor mevcut |
| Donanim | 4 | Metadata etiketi |
| Oturum_Yonetimi | 3 | Fiziksel klasor mevcut |
| Tedarik_Zinciri | 2 | Metadata etiketi |
| Veri_Ihlalleri | 2 | Metadata etiketi |
| OWASP_Top_10 | 1 | Fiziksel klasor mevcut |
| Enjeksiyon | 1 | Fiziksel klasor mevcut |
| Mobil | 1 | Metadata etiketi |
| Kimlik_Dogrulama | 1 | Metadata etiketi |

---

## Dokuman Sablonlari

Projede yeni guvenlik acigi dokumanlari olustururken kullanilan sablonlar:

| Sablon | Aciklama |
|--------|----------|
| [todo_sablonu.md](_Sablonlar/todo_sablonu.md) | Zafiyet analiz sablonu (8 bolumluk detayli format) |

### Standart Dokuman Yapisi

Her guvenlik acigi dokumani asagidaki bolumleri icermelidir:

1. **Ozet ve Etki** - Zafiyetin ne oldugu ve etkisi
2. **Teknik Detay** - Nasil calistigi
3. **PoC (Proof of Concept)** - Kavramsal istismar adimlari
4. **Risk Degerlendirmesi** - Kritiklik, saldiri yuzeyi, karmasiklik
5. **Cozum ve Onlemler** - Kisa, orta ve uzun vadeli cozumler
6. **Ornek Duzeltme Kodu** - Guvenli kod ornekleri
7. **Kontrol Listesi (Checklist)** - Uygulanabilir adimlar
8. **Izleme ve Uyarilar** - SIEM/Log kurallari

---

## Dosya Isimlendirme Standardi

Yeni bir dokuman eklerken asagidaki standarda uyulmasi onerilir:

**Kategori rehberleri:** `[Kategori]/[Baslik]_[Teknoloji].md`
- `Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md`
- `Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md`

**TODO dosyalari:** `_TODO/TODO_[slug_snake_case].md`
- `_TODO/TODO_cve_2026_22708_cursor_shell_bypass.md`
- `_TODO/TODO_roguepilot_copilot_token_theft.md`

---

## CVE Indeksi

Projedeki CVE numaralarina gore duzenli indeks:

| CVE | TODO Dosyasi | Kategori | CVSS |
|-----|-------------|----------|------|
| CVE-2022-3564 | [TODO](_TODO/TODO_cve_2022_3564_bluetooth_l2cap_rce.md) | Isletim Sistemi | 7.1 |
| CVE-2025-8110 | [TODO](_TODO/TODO_cve_2025_8110_gogs_symlink.md) | Erisim Kontrolu | - |
| CVE-2025-27209 | [TODO](_TODO/TODO_cve_2025_27209_nodejs_v8_hashdos.md) | Enjeksiyon | - |
| CVE-2025-27210 | [TODO](_TODO/TODO_cve_2025_27210_nodejs_windows_path_traversal.md) | Erisim Kontrolu | - |
| CVE-2025-33218 | [TODO](_TODO/TODO_cve_2025_33218_nvidia_integer_overflow.md) | Donanim | - |
| CVE-2025-33219 | [TODO](_TODO/TODO_cve_2025_33219_nvidia_integer_overflow.md) | Donanim | - |
| CVE-2025-33220 | [TODO](_TODO/TODO_cve_2025_33220_nvidia_vgpu_uaf.md) | Donanim | - |
| CVE-2025-55182 | [TODO](_TODO/TODO_cve_2025_55182_react2shell_rce.md) | Oturum Yonetimi | 10.0 |
| CVE-2025-64459 | [Rehber](Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md) | Enjeksiyon | - |
| CVE-2025-66478 | [TODO](_TODO/TODO_cve_2025_66478_nextjs_rce.md) | Oturum Yonetimi | 10.0 |
| CVE-2025-66516 | [TODO](_TODO/TODO_cve_2025_66516_apache_tika_xxe.md) | OWASP Top 10 | - |
| CVE-2025-68664 | [TODO](_TODO/TODO_cve_2025_68664_langgrinch_langchain.md) | Erisim Kontrolu | - |
| CVE-2026-1529 | [TODO](_TODO/TODO_cve_2026_1529_keycloak_org_invite.md) | Kimlik Dogrulama | - |
| CVE-2026-20045 | [TODO](_TODO/TODO_cve_2026_20045_cisco_unified_cm_rce.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-20127 | [TODO](_TODO/TODO_cve_2026_20127_cisco_sdwan_auth_bypass.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-20805 | [TODO](_TODO/TODO_cve_2026_20805_windows_dwm_bilgi_ifsasi.md) | Isletim Sistemi | - |
| CVE-2026-20822 | [TODO](_TODO/TODO_cve_2026_20822_windows_graphics_eop.md) | Isletim Sistemi | - |
| CVE-2026-20854 | [TODO](_TODO/TODO_cve_2026_20854_windows_lsass_rce.md) | Isletim Sistemi | - |
| CVE-2026-21265 | [TODO](_TODO/TODO_cve_2026_21265_secure_boot_bypass.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-21509 | [TODO](_TODO/TODO_cve_2026_21509_office_ole_com_bypass.md) | Oturum Yonetimi | - |
| CVE-2026-21510 | [TODO](_TODO/TODO_cve_2026_21510_windows_shell_bypass.md) | Isletim Sistemi | - |
| CVE-2026-21513 | [TODO](_TODO/TODO_cve_2026_21513_mshtml_bypass.md) | Isletim Sistemi | - |
| CVE-2026-21516 | [TODO](_TODO/TODO_cve_2026_21516_copilot_jetbrains_rce.md) | AI IDE Guvenligi | - |
| CVE-2026-21519 | [TODO](_TODO/TODO_cve_2026_21519_dwm_eop.md) | Isletim Sistemi | - |
| CVE-2026-21525 | [TODO](_TODO/TODO_cve_2026_21525_ras_dos.md) | Isletim Sistemi | - |
| CVE-2026-21533 | [TODO](_TODO/TODO_cve_2026_21533_rds_eop.md) | Isletim Sistemi | - |
| CVE-2026-21852 | [TODO](_TODO/TODO_cve_2026_21852_claude_code_api_key_leak.md) | AI IDE Guvenligi | - |
| CVE-2026-21945 | [TODO](_TODO/TODO_cve_2026_21945_oracle_mysql_java_ssrf.md) | Erisim Kontrolu | - |
| CVE-2026-21962 | [TODO](_TODO/TODO_cve_2026_21962_oracle_weblogic_proxy_rce.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-22708 | [TODO](_TODO/TODO_cve_2026_22708_cursor_shell_bypass.md) | AI IDE Guvenligi | - |
| CVE-2026-22769 | [TODO](_TODO/TODO_cve_2026_22769_dell_recoverpoint_hardcoded.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-23000 | [TODO](_TODO/TODO_cve_2026_23000_linux_mlx5e_uaf.md) | Isletim Sistemi | - |
| CVE-2026-24858 | [TODO](_TODO/TODO_cve_2026_24858_fortios_sso_bypass.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-25725 | [TODO](_TODO/TODO_cve_2026_25725_claude_code_sandbox_escape.md) | AI IDE Guvenligi | - |
| AMD-SB-7038 | [TODO](_TODO/TODO_amd_sb_7038_memory_disorder.md) | Donanim | - |

---

## Teknoloji Indeksi

Projede karsilasilan teknolojiler ve platformlar:

### Web Framework'ler
- React / Next.js
- Django

### Programlama Dilleri
- Node.js (JavaScript)
- Python
- Java

### Veritabani Sistemleri
- MySQL
- Oracle

### Isletim Sistemleri
- Windows (DWM, LSASS, Graphics, Shell, MSHTML)
- Linux (mlx5e driver, Bluetooth L2CAP)

### Donanim
- AMD CPU
- NVIDIA GPU / vGPU

### Mobil
- Android (Dolby Digital Plus)

### Ag Guvenligi
- Cisco Unified CM, SD-WAN
- FortiOS / FortiManager / FortiAnalyzer
- Dell RecoverPoint

### AI/ML ve IDE
- LangChain
- Cursor, GitHub Copilot, Claude Code
- Replit Agent, Windsurf
- MCP (Model Context Protocol)

---

## Katki Saglama

Yeni bir guvenlik acigi dokumani eklerken:

1. Uygun klasoru secin (genel rehber icin kategori klasoru, CVE analizi icin `_TODO/`)
2. Dosya isimlendirme standardina uyarak dokumani olusturun
3. `_Sablonlar/todo_sablonu.md` sablonunu kullanin
4. README.md dosyasini guncelleyin
5. Bu INDEX.md dosyasina ilgili bolume ekleyin
6. Git commit mesajinda CVE numarasini belirtin

---

## Kaynaklar

- [AGENTS.md](AGENTS.md) - Repository yonergeleri ve gelistirici klavuzu
- [CLAUDE.md](CLAUDE.md) - Claude Code is akisi talimatlari
- [README.md](README.md) - Ana proje aciklamasi ve icerik listesi

---

*Son guncelleme: 2026-03-10*
