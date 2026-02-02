# Güvenlik Kütüphanesi (AppSec Playbook)

Bu depo, **yapay zeka desteğiyle yazılım geliştirenler** ve **yazılım güvenliğine yeni başlayanlar** için oluşturulmuş, yaşayan bir güvenlik kütüphanesidir.

**Amacı:** İzlenen yayınlar, okunan makaleler ve güncel güvenlik açıklarını belirli bir şablona göre raporlayarak, teknik derinlikte boğulmadan herkesin anlayabileceği ve uygulayabileceği pratik bir rehber sunmaktır. İçerik zamanla, yeni araştırmalarla birlikte yavaş yavaş zenginleşecektir.

Depo içerisinde teknik güvenlik analizleri, zafiyet incelemeleri, kontrol listeleri (checklists) ve güvenli kodlama rehberleri bulunur.

## 📂 Klasör Yapısı

Depo, içeriklerin kolay erişilebilir olması için teknoloji ve alana göre kategorize edilmiştir:

```text
Guvenlik-Rehberi-DB/
├── Enjeksiyon/              # SQL Injection, Command Injection vb.
├── Erisim_Kontrolu/         # Broken Access Control, IDOR vb.
├── Oturum_Yonetimi/         # CSRF, Session Management vb.
├── Arastirmalar/            # Derinlemesine araştırma ve raporlar
├── Isletim_Sistemi/         # Windows, Linux, kernel ve sürücü zafiyetleri
├── Donanim/                 # CPU/GPU ve donanım seviyesinde zafiyetler
├── Mobil/                   # Android/iOS ve mobil platform zafiyetleri
├── Kimlik_Dogrulama/        # Broken Authentication vb.
├── Hassas_Veri/             # Sensitive Data Exposure vb.
├── Guvenlik_Yapilandirmasi/ # Security Misconfiguration vb.
├── OWASP_Top_10/            # OWASP Top 10 2025 güvenlik açıkları
├── _Sablonlar/              # Doküman şablonları
└── references.md            # Kaynaklar ve referanslar
```

## 📝 Dosya İsimlendirme Standardı & İçerik

Yeni bir doküman eklerken aşağıdaki standarda uyulması önerilir:

**Format:** `[Zafiyet_Kategorisi]/[Zafiyet_Basligi]_[Teknoloji].md`

**Örnekler:**
*   `Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md`
*   `Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md`
*   `Oturum_Yonetimi/csrf_ve_oturum_kacirma.md`
*   `Guvenlik_Yapilandirmasi/Docker_Security_Hardening_DevOps.md`
*   `Kimlik_Dogrulama/JWT_Zafiyetleri_Common.md`

### İçerik Nasıl Olmalı?
Her doküman ideal olarak şu bölümleri içermelidir:
1.  **Özet ve Etki:** Zafiyet nedir, etkisi nedir?
2.  **Teknik Detay:** Nasıl çalışır?
3.  **PoC (Proof of Concept):** Kavramsal istismar adımları.
4.  **Çözüm ve Önlemler:** Kod örnekleriyle (Secure Coding) düzeltme.
5.  **Checklist:** Kontrol listesi.

> **Not:** Kaynaklar ve referanslar repo kökündeki `references.md` dosyasında tutulmaktadır.

## 🚀 Mevcut İçerikler

### OWASP Top 10 2025
*   [Güvenlik Açığı: Kırık Erişim Kontrolü (Broken Access Control)](OWASP_Top_10/A01_Broken_Access_Control.md) - Kırık Erişim Kontrolü, uygulamanın yetkilendirme mekanizmalarının yetersiz veya hatalı olması nedeniyle yetkisiz kullanıcıların kısıtlı kaynakla...
*   [Güvenlik Açığı: Güvenlik Yanlış Yapılandırması (Security Misconfiguration)](OWASP_Top_10/A02_Security_Misconfiguration.md) - Güvenlik Yanlış Yapılandırması, uygulama, sunucu, veritabanı veya altyapı bileşenlerinin güvenlik ayarlarının düzgün yapılandırılmaması, varsayı...
*   [Güvenlik Açığı: Yazılım Tedarik Zinciri Hataları (Software Supply Chain Failures)](OWASP_Top_10/A03_Software_Supply_Chain_Failures.md) - Yazılım Tedarik Zinciri Hataları, uygulama geliştirme sürecinde kullanılan dış bağımlılıkların, kütüphanelerin, framework'lerin veya altyapı bil...
*   [Güvenlik Açığı: Kriptografik Hatalar (Cryptographic Failures)](OWASP_Top_10/A04_Cryptographic_Failures.md) - Kriptografik Hatalar, hassas verilerin korunması için kullanılan şifreleme algoritmalarının, protokollerin veya uygulamaların zayıf veya yanlış ...
*   [Güvenlik Açığı: Enjeksiyon (Injection)](OWASP_Top_10/A05_Injection.md) - Enjeksiyon, uygulamanın kullanıcı girdilerini doğrulamadan veya filtrelemeden doğrudan bir yorumlayıcıya (veritabanı, shell, LDAP vb.) göndermes...
*   [Güvenlik Açığı: Güvensiz Tasarım (Insecure Design)](OWASP_Top_10/A06_Insecure_Design.md) - Güvensiz Tasarım, uygulamanın temel mimarisinde veya tasarımında güvenlik kontrollerinin eksik olması veya yanlış uygulanması nedeniyle ortaya ç...
*   [Güvenlik Açığı: Kimlik Doğrulama Hataları (Authentication Failures)](OWASP_Top_10/A07_Authentication_Failures.md) - Kimlik Doğrulama Hataları, kullanıcı kimliğinin doğrulanması sürecindeki zayıflıklar nedeniyle yetkisiz kullanıcıların sisteme erişim sağlayabil...
*   [Güvenlik Açığı: Yazılım veya Veri Bütünlüğü Hataları (Software or Data Integrity Failures)](OWASP_Top_10/A08_Software_or_Data_Integrity_Failures.md) - Yazılım veya Veri Bütünlüğü Hataları, verilerin veya kodların bütünlüğünü doğrulama mekanizmalarının eksikliği veya zayıflığı nedeniyle yetkisiz...
*   [Güvenlik Açığı: Güvenlik Kayıt Tutma ve Uyarı Hataları (Security Logging and Alerting Failures)](OWASP_Top_10/A09_Security_Logging_and_Alerting_Failures.md) - Güvenlik Kayıt Tutma ve Uyarı Hataları, güvenlik olaylarının yetersiz kaydedilmesi, kritik olayların tespit edilememesi veya uyarı mekanizmaları...
*   [Güvenlik Açığı: İstisnai Durumların Yanlış Ele Alınması (Mishandling of Exceptional Conditions)](OWASP_Top_10/A10_Mishandling_of_Exceptional_Conditions.md) - İstisnai Durumların Yanlış Ele Alınması, uygulamanın hata durumlarını güvenli bir şekilde yönetememesi nedeniyle ortaya çıkan bir güvenlik açığı...

### Enjeksiyon
*   [Güvenlik Açığı: MySQL2 `escape` / tip-manipülasyonu (detaylı doküman ve checklist)](Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md) - Node.js uygulamasında parametrik sorgular kullanılıyor olsa bile, MySQL2 kütüphanesinin `escape`/`format` mekanizmasının gelen değerin *tipine* ...
*   [Güvenlik Açığı: Django ORM Connector SQL Injection — CVE-2025-64459](Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md)
*   [Güvenlik Açığı: SQL Injection ile Veritabanı Sorgularının Kontrol Edilmesi](Enjeksiyon/sql_injection_veritabani_sorgulari.md)

### Erişim Kontrolü
*   [Güvenlik Açığı: Insecure Direct Object Reference (IDOR) ve Fonksiyon Seviyesi Erişim Kontrolü Eksikliği](Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md)

### Oturum Yönetimi
*   [Güvenlik Açığı: Oturum Çerezlerinin Otomatik Gönderilmesi ile Tetiklenen CSRF ve Oturum Kaçırma Zafiyeti](Oturum_Yonetimi/csrf_ve_oturum_kacirma.md)
*   [Güvenlik Açığı: Node.js ve React Ekosistemi - React2Shell (CVE-2025-55182) & Next.js RCE (CVE-2025-66478)](Oturum_Yonetimi/Nextjs_React_RCE_Analizi.md)

### Araştırmalar
*   [2025 Sonu Node.js ve React Ekosistemi Güvenlik Krizi: React2Shell (CVE-2025-55182) Kapsamlı Tehdit Analizi ve Savunma Mimarisi](Arastirmalar/Node.js Güvenlik Açığı Araştırması.md)

### İşletim Sistemi
*   [Güvenlik Açığı: Windows DWM Bilgi İfşası — CVE-2026-20805](Isletim_Sistemi/windows_dwm_info_disclosure_cve_2026_20805.md)
*   [Güvenlik Açığı: Windows Graphics Yetki Yükseltme — CVE-2026-20822](Isletim_Sistemi/windows_graphics_eop_cve_2026_20822.md)
*   [Güvenlik Açığı: Windows LSASS Uzaktan Kod Yürütme — CVE-2026-20854](Isletim_Sistemi/windows_lsass_rce_cve_2026_20854.md)
*   [Güvenlik Açığı: Linux Mellanox mlx5e Use-After-Free — CVE-2026-23000](Isletim_Sistemi/linux_mlx5e_uaf_cve_2026_23000.md)
*   [Güvenlik Açığı: Bluetooth L2CAP Uzaktan Kod Yürütme — CVE-2022-3564](Isletim_Sistemi/bluetooth_l2cap_rce_cve_2022_3564.md)

### Donanım
*   [Güvenlik Açığı: AMD MEMORY DISORDER Yan Kanalı — AMD-SB-7038](Donanim/amd_sb_7038_memory_disorder_side_channel.md)
*   [Güvenlik Açığı: NVIDIA GPU Sürücü Integer Overflow — CVE-2025-33218](Donanim/nvidia_driver_integer_overflow_cve_2025_33218.md)
*   [Güvenlik Açığı: NVIDIA GPU Sürücü Integer Overflow — CVE-2025-33219](Donanim/nvidia_driver_integer_overflow_cve_2025_33219.md)
*   [Güvenlik Açığı: NVIDIA vGPU Use-After-Free İzolasyon İhlali — CVE-2025-33220](Donanim/nvidia_vgpu_uaf_cve_2025_33220.md)

### Mobil
*   [Güvenlik Açığı: Android Dolby Digital Plus Zero-Click RCE](Mobil/android_dolby_digital_plus_zero_click.md)

### Ocak 2026 Analizleri
#### İşletim Sistemi
*   [Güvenlik Açığı: Windows DWM Bilgi İfşası — CVE-2026-20805](Isletim_Sistemi/windows_dwm_info_disclosure_cve_2026_20805.md)
*   [Güvenlik Açığı: Windows Graphics Yetki Yükseltme — CVE-2026-20822](Isletim_Sistemi/windows_graphics_eop_cve_2026_20822.md)
*   [Güvenlik Açığı: Windows LSASS Uzaktan Kod Yürütme — CVE-2026-20854](Isletim_Sistemi/windows_lsass_rce_cve_2026_20854.md)
*   [Güvenlik Açığı: Linux Mellanox mlx5e Use-After-Free — CVE-2026-23000](Isletim_Sistemi/linux_mlx5e_uaf_cve_2026_23000.md)
*   [Güvenlik Açığı: Bluetooth L2CAP Uzaktan Kod Yürütme — CVE-2022-3564](Isletim_Sistemi/bluetooth_l2cap_rce_cve_2022_3564.md)

#### Donanım
*   [Güvenlik Açığı: AMD MEMORY DISORDER Yan Kanalı — AMD-SB-7038](Donanim/amd_sb_7038_memory_disorder_side_channel.md)
*   [Güvenlik Açığı: NVIDIA GPU Sürücü Integer Overflow — CVE-2025-33218](Donanim/nvidia_driver_integer_overflow_cve_2025_33218.md)
*   [Güvenlik Açığı: NVIDIA GPU Sürücü Integer Overflow — CVE-2025-33219](Donanim/nvidia_driver_integer_overflow_cve_2025_33219.md)
*   [Güvenlik Açığı: NVIDIA vGPU Use-After-Free İzolasyon İhlali — CVE-2025-33220](Donanim/nvidia_vgpu_uaf_cve_2025_33220.md)

#### Mobil
*   [Güvenlik Açığı: Android Dolby Digital Plus Zero-Click RCE](Mobil/android_dolby_digital_plus_zero_click.md)

#### Güvenlik Yapılandırması
*   [Güvenlik Açığı: Cisco Unified CM Uzaktan Kod Yürütme — CVE-2026-20045](Guvenlik_Yapilandirmasi/cisco_unified_cm_rce_cve_2026_20045.md)
*   [Güvenlik Açığı: FortiOS/FortiManager/FortiAnalyzer FortiCloud SSO Baypası — CVE-2026-24858](Guvenlik_Yapilandirmasi/fortios_sso_bypass_cve_2026_24858.md)
*   [Güvenlik Açığı: Oracle HTTP Server / WebLogic Proxy Plug-in RCE — CVE-2026-21962](Guvenlik_Yapilandirmasi/oracle_http_server_weblogic_proxy_rce_cve_2026_21962.md)
*   [Güvenlik Açığı: Secure Boot Güvenlik Özelliği Baypası — CVE-2026-21265](Guvenlik_Yapilandirmasi/secure_boot_bypass_cve_2026_21265.md)

#### Erişim Kontrolü
*   [Güvenlik Açığı: Gogs PutContents API Symlink Zafiyeti — CVE-2025-8110](Erisim_Kontrolu/gogs_putcontents_symlink_cve_2025_8110.md)
*   [Güvenlik Açığı: LangChain Core Deserialization Enjeksiyonu (LangGrinch) — CVE-2025-68664](Erisim_Kontrolu/langchain_core_langgrinch_cve_2025_68664.md)
*   [Güvenlik Açığı: Oracle MySQL/Java SE SSRF/RCE — CVE-2026-21945](Erisim_Kontrolu/oracle_mysql_java_ssrf_rce_cve_2026_21945.md)

#### Oturum Yönetimi
*   [Güvenlik Açığı: Microsoft Office OLE/COM Koruma Baypası — CVE-2026-21509](Oturum_Yonetimi/office_ole_com_bypass_cve_2026_21509.md)

#### OWASP Top 10
*   [Güvenlik Açığı: Apache Tika XXE Zafiyeti — CVE-2025-66516](OWASP_Top_10/apache_tika_xxe_cve_2025_66516.md)
