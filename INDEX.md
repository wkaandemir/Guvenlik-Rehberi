# Guvenlik-Rehberi Proje Indeksi

Bu doküman, Guvenlik Rehberi projesinin kapsamlı indeksini ve içerik haritasını sunar. Proje, yapay zeka desteğiyle yazılım geliştirenler ve yazılım guvenligine yeni baslayanlar icin olusturulmus, yasayan bir guvenlik kutuphanesidir.

## Proje Hakkinda

**Amaç:** Izlenen yayinlar, okunan makaleler ve guncel guvenlik aciklarini belirli bir sablona gore raporlayarak, teknik derinlikte bogulmadan herkesin anlayabilecegi ve uygulayabilecegi pratik bir rehber sunmaktir.

**Hedef Kitle:**
- Yapay zeka destegiyle yazilim gelistirenler
- Yazilik guvenligine yeni baslayanlar
- Guvenlik araştirmacilari
- DevOps ve Site Reliability mühendisleri

**Icerik Dili:** Turkce (varsayilan), Ingilizce (teknik terimler ve kaynaklar)

---

## Klasor Yapisi ve Organizasyon

Proje icerikleri, guvenlik kategorilerine ve teknoloji alanlarina gore organize edilmistir:

```
Guvenlik-Rehberi/
├── OWASP_Top_10/           # OWASP Top 10 2025 guvenlik aciklari
├── Enjeksiyon/              # SQL Injection, Command Injection vb.
├── Erisim_Kontrolu/         # Broken Access Control, IDOR vb.
├── Oturum_Yonetimi/         # CSRF, Session Management vb.
├── Guvenlik_Yapilandirmasi/ # Security Misconfiguration vb.
├── Arastirmalar/            # Derinlemesine arastirma ve raporlar
├── Isletim_Sistemi/         # Windows, Linux, kernel ve surucu zafiyetleri
├── Donanim/                 # CPU/GPU ve donanim seviyesinde zafiyetler
├── Mobil/                   # Android/iOS ve mobil platform zafiyetleri
├── _Sablonlar/              # Dokuman sablonlari
├── AGENTS.md                # Repository yonergeleri
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

**Iliskili CVE Analizleri:**
- [Apache Tika XXE - CVE-2025-66516](OWASP_Top_10/apache_tika_xxe_cve_2025_66516.md) (A05 - Injection)

---

### Enjeksiyon (Injection)

Enjeksion tipi zafiyetler, kullanici girdilerinin dogrulanmadan bir yorumlayiciya gonderilmesi sonucunda ortaya cikar.

| Dokuman | Teknoloji | CVE | Aciklama |
|---------|-----------|-----|----------|
| [MySQL2_Tip_Manipulasyonu_NodeJS.md](Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md) | Node.js, MySQL2 | - | Tip manipulasyonu ile authentication bypass |
| [django_orm_sql_injection_cve_2025_64459.md](Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md) | Django ORM | CVE-2025-64459 | ORM connector SQL injection |
| [sql_injection_veritabani_sorgulari.md](Enjeksiyon/sql_injection_veritabani_sorgulari.md) | Genel | - | SQL injection temel kavramlar |

**Iliskili Kategoriler:**
- OWASP A05: Injection
- Erisim Kontrolu: Authentication bypass senaryolari

---

### Erisim Kontrolu (Access Control)

Yetkisiz erisim kontrolu zafiyetleri, kullanici yetkilerinin dogru dogrulanmamasi sonucunda ortaya cikar.

| Dokuman | Teknoloji | CVE | Aciklama |
|---------|-----------|-----|----------|
| [idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md](Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md) | Genel | - | IDOR ve fonksiyon seviyesi erisim kontrolu |
| [gogs_putcontents_symlink_cve_2025_8110.md](Erisim_Kontrolu/gogs_putcontents_symlink_cve_2025_8110.md) | Gogs | CVE-2025-8110 | Symlink zafiyeti ile dosya erisimi |
| [langchain_core_langgrinch_cve_2025_68664.md](Erisim_Kontrolu/langchain_core_langgrinch_cve_2025_68664.md) | LangChain | CVE-2025-68664 | Deserialization enjeksiyonu (LangGrinch) |
| [oracle_mysql_java_ssrf_rce_cve_2026_21945.md](Erisim_Kontrolu/oracle_mysql_java_ssrf_rce_cve_2026_21945.md) | Oracle, MySQL, Java | CVE-2026-21945 | SSRF/RCE zafiyeti |

**Iliskili Kategoriler:**
- OWASP A01: Broken Access Control
- Oturum Yonetimi: Yetkilendirme mekanizmalari

---

### Oturum Yonetimi (Session Management)

Oturum yonetimi zafiyetleri, kullanici oturumlarinin guvensiz olmasi veya manipule edilebilmesi durumlarini icerir.

| Dokuman | Teknoloji | CVE | Aciklama |
|---------|-----------|-----|----------|
| [csrf_ve_oturum_kacirma.md](Oturum_Yonetimi/csrf_ve_oturum_kacirma.md) | Genel | - | CSRF ve oturum kacirma zafiyeti |
| [Nextjs_React_RCE_Analizi.md](Oturum_Yonetimi/Nextjs_React_RCE_Analizi.md) | Next.js, React | CVE-2025-55182, CVE-2025-66478 | React2Shell kapsamli analiz |
| [office_ole_com_bypass_cve_2026_21509.md](Oturum_Yonetimi/office_ole_com_bypass_cve_2026_21509.md) | Microsoft Office | CVE-2026-21509 | OLE/COM koruma bypass |

**Iliskili Kategoriler:**
- OWASP A07: Authentication Failures
- Arastirmalar: [Node.js Güvenlik Açığı Araştırması](Arastirmalar/Node.js Güvenlik Açığı Araştırması.md)

---

### Guvenlik Yapilandirmasi (Security Configuration)

Güvenlik yanlis konfigürasyonlari, sistem ve uygulama ayarlarinin guvensiz yapilmasi sonucunda ortaya cikar.

| Dokuman | Teknoloji | CVE | Aciklama |
|---------|-----------|-----|----------|
| [cisco_unified_cm_rce_cve_2026_20045.md](Guvenlik_Yapilandirmasi/cisco_unified_cm_rce_cve_2026_20045.md) | Cisco Unified CM | CVE-2026-20045 | Uzaktan kod yurutme |
| [fortios_sso_bypass_cve_2026_24858.md](Guvenlik_Yapilandirmasi/fortios_sso_bypass_cve_2026_24858.md) | FortiOS | CVE-2026-24858 | FortiCloud SSO bypass |
| [oracle_http_server_weblogic_proxy_rce_cve_2026_21962.md](Guvenlik_Yapilandirmasi/oracle_http_server_weblogic_proxy_rce_cve_2026_21962.md) | Oracle HTTP Server | CVE-2026-21962 | WebLogic Proxy RCE |
| [secure_boot_bypass_cve_2026_21265.md](Guvenlik_Yapilandirmasi/secure_boot_bypass_cve_2026_21265.md) | Secure Boot | CVE-2026-21265 | Secure Boot guvenlik ozelligi bypass |

**Iliskili Kategoriler:**
- OWASP A02: Security Misconfiguration
- Isletim Sistemi: Sistem seviyesi konfigurasyonlar

---

### Arastirmalar (Research Reports)

Derinlemesine guvenlik arastirmalari ve kapsamli tehdit analizleri icerir.

| Dokuman | Konu | Aciklama |
|---------|------|----------|
| [Node.js Güvenlik Açığı Araştırması.md](Arastirmalar/Node.js Güvenlik Açığı Araştırması.md) | React2Shell | CVE-2025-55182 ve CVE-2025-66478 kapsamli tehdit analizi |
| [Ocak 2026 Güvenlik Açıkları Araştırması.md](Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md) | Aylik Ozet | Ocak 2026 guvenlik aciklari ozet raporu |
| [Keycloak Zafiyet Araştırması ve Analizi.md](Arastirmalar/Keycloak Zafiyet Araştırması ve Analizi.md) | Keycloak | CVE-2026-1529 organizasyon davet/katilim akisi zafiyet analizi |

**Iliskili Dokumanlar:**
- [Nextjs_React_RCE_Analizi.md](Oturum_Yonetimi/Nextjs_React_RCE_Analizi.md) - Daha teknik detayli analiz
- OWASP A03: Software Supply Chain Failures

---

### Isletim Sistemi (Operating System)

Isletim sistemi seviyesindeki guvenlik zafiyetlerini icerir.

| Dokuman | Platform | CVE | Aciklama |
|---------|----------|-----|----------|
| [windows_dwm_info_disclosure_cve_2026_20805.md](Isletim_Sistemi/windows_dwm_info_disclosure_cve_2026_20805.md) | Windows | CVE-2026-20805 | DWM bilgi ifsas |
| [windows_graphics_eop_cve_2026_20822.md](Isletim_Sistemi/windows_graphics_eop_cve_2026_20822.md) | Windows | CVE-2026-20822 | Graphics yetki yukseltme |
| [windows_lsass_rce_cve_2026_20854.md](Isletim_Sistemi/windows_lsass_rce_cve_2026_20854.md) | Windows | CVE-2026-20854 | LSASS uzaktan kod yurutme |
| [linux_mlx5e_uaf_cve_2026_23000.md](Isletim_Sistemi/linux_mlx5e_uaf_cve_2026_23000.md) | Linux | CVE-2026-23000 | Mellanox mlx5e Use-After-Free |
| [bluetooth_l2cap_rce_cve_2022_3564.md](Isletim_Sistemi/bluetooth_l2cap_rce_cve_2022_3564.md) | Bluetooth | CVE-2022-3564 | L2CAP uzaktan kod yurutme |

**Iliskili Kategoriler:**
- Donanim: Driver seviyesi zafiyetler
- Guvenlik Yapilandirmasi: Sizdirma ayarlari

---

### Donanim (Hardware)

Donanim seviyesindeki guvenlik zafiyetlerini icerir.

| Dokuman | Platform | CVE/SB | Aciklama |
|---------|----------|--------|----------|
| [amd_sb_7038_memory_disorder_side_channel.md](Donanim/amd_sb_7038_memory_disorder_side_channel.md) | AMD CPU | AMD-SB-7038 | MEMORY DISORDER yan kanali |
| [nvidia_driver_integer_overflow_cve_2025_33218.md](Donanim/nvidia_driver_integer_overflow_cve_2025_33218.md) | NVIDIA GPU | CVE-2025-33218 | Integer Overflow |
| [nvidia_driver_integer_overflow_cve_2025_33219.md](Donanim/nvidia_driver_integer_overflow_cve_2025_33219.md) | NVIDIA GPU | CVE-2025-33219 | Integer Overflow |
| [nvidia_vgpu_uaf_cve_2025_33220.md](Donanim/nvidia_vgpu_uaf_cve_2025_33220.md) | NVIDIA vGPU | CVE-2025-33220 | Use-After-Free izolasyon ihlali |

**Iliskili Kategoriler:**
- Isletim Sistemi: Driver etkilesimleri

---

### Mobil (Mobile)

Mobil platform guvenlik zafiyetlerini icerir.

| Dokuman | Platform | Aciklama |
|---------|----------|----------|
| [android_dolby_digital_plus_zero_click.md](Mobil/android_dolby_digital_plus_zero_click.md) | Android | Dolby Digital Plus Zero-Click RCE |

**Iliskili Kategoriler:**
- Isletim Sistemi: Mobil OS seviyesi zafiyetler

---

## Dokuman Sablonlari

Projde yeni guvenlik acigi dokumanlari olustururken kullanilan sablonlar:

| Sablon | Aciklama |
|--------|----------|
| [guvenlik_acigi_prompt_sablonu.md](_Sablonlar/guvenlik_acigi_prompt_sablonu.md) | Standart guvenlik acigi raporlama sablonu |

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

**Format:** `[Zafiyet_Kategorisi]/[Zafiyet_Basligi]_[Teknoloji].md`

**Ornekler:**
- `Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md`
- `Erisim_Kontrolu/idor_ve_fonksiyon_seviyesi_erisim_kontrolu.md`
- `Oturum_Yonetimi/csrf_ve_oturum_kacirma.md`
- `Guvenlik_Yapilandirmasi/Docker_Security_Hardening_DevOps.md`
- `Kimlik_Dogrulama/JWT_Zafiyetleri_Common.md`

---

## CVE Indeksi

Projedeki CVE numaralarina gore duzenli indeks:

| CVE | Dokuman | Kategori | CVSS |
|-----|---------|----------|------|
| CVE-2025-66516 | [apache_tika_xxe](OWASP_Top_10/apache_tika_xxe_cve_2025_66516.md) | OWASP Top 10 | - |
| CVE-2025-68664 | [langchain_core_langgrinch](Erisim_Kontrolu/langchain_core_langgrinch_cve_2025_68664.md) | Erisim Kontrolu | - |
| CVE-2025-8110 | [gogs_putcontents_symlink](Erisim_Kontrolu/gogs_putcontents_symlink_cve_2025_8110.md) | Erisim Kontrolu | - |
| CVE-2025-55182 | [React2Shell](Oturum_Yonetimi/Nextjs_React_RCE_Analizi.md) | Oturum Yonetimi | 10.0 |
| CVE-2025-66478 | [Next.js RCE](Oturum_Yonetimi/Nextjs_React_RCE_Analizi.md) | Oturum Yonetimi | 10.0 |
| CVE-2025-33218 | [nvidia_driver_int_overflow](Donanim/nvidia_driver_integer_overflow_cve_2025_33218.md) | Donanim | - |
| CVE-2025-33219 | [nvidia_driver_int_overflow](Donanim/nvidia_driver_integer_overflow_cve_2025_33219.md) | Donanim | - |
| CVE-2025-33220 | [nvidia_vgpu_uaf](Donanim/nvidia_vgpu_uaf_cve_2025_33220.md) | Donanim | - |
| CVE-2025-64459 | [django_orm_sql_injection](Enjeksiyon/django_orm_sql_injection_cve_2025_64459.md) | Enjeksiyon | - |
| CVE-2026-20045 | [cisco_unified_cm_rce](Guvenlik_Yapilandirmasi/cisco_unified_cm_rce_cve_2026_20045.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-20805 | [windows_dwm_info_disclosure](Isletim_Sistemi/windows_dwm_info_disclosure_cve_2026_20805.md) | Isletim Sistemi | - |
| CVE-2026-20822 | [windows_graphics_eop](Isletim_Sistemi/windows_graphics_eop_cve_2026_20822.md) | Isletim Sistemi | - |
| CVE-2026-20854 | [windows_lsass_rce](Isletim_Sistemi/windows_lsass_rce_cve_2026_20854.md) | Isletim Sistemi | - |
| CVE-2026-21265 | [secure_boot_bypass](Guvenlik_Yapilandirmasi/secure_boot_bypass_cve_2026_21265.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-21509 | [office_ole_com_bypass](Oturum_Yonetimi/office_ole_com_bypass_cve_2026_21509.md) | Oturum Yonetimi | - |
| CVE-2026-21945 | [oracle_mysql_java_ssrf_rce](Erisim_Kontrolu/oracle_mysql_java_ssrf_rce_cve_2026_21945.md) | Erisim Kontrolu | - |
| CVE-2026-21962 | [oracle_http_server_weblogic_proxy_rce](Guvenlik_Yapilandirmasi/oracle_http_server_weblogic_proxy_rce_cve_2026_21962.md) | Guvenlik Yapilandirmasi | - |
| CVE-2026-23000 | [linux_mlx5e_uaf](Isletim_Sistemi/linux_mlx5e_uaf_cve_2026_23000.md) | Isletim Sistemi | - |
| CVE-2026-24858 | [fortios_sso_bypass](Guvenlik_Yapilandirmasi/fortios_sso_bypass_cve_2026_24858.md) | Guvenlik Yapilandirmasi | - |
| CVE-2022-3564 | [bluetooth_l2cap_rce](Isletim_Sistemi/bluetooth_l2cap_rce_cve_2022_3564.md) | Isletim Sistemi | - |
| AMD-SB-7038 | [amd_memory_disorder](Donanim/amd_sb_7038_memory_disorder_side_channel.md) | Donanim | - |

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
- Windows (DWM, LSASS, Graphics)
- Linux (mlx5e driver)

### Donanim
- AMD CPU
- NVIDIA GPU / vGPU

### Mobil
- Android

### Ag Guvenligi
- Cisco Unified CM
- FortiOS / FortiManager / FortiAnalyzer

### AI/ML
- LangChain

---

## Katki Saglama

Yeni bir guvenlik acigi dokumani eklerken:

1. Uygun klasoru secin veya yeni bir klasor olusturun
2. Dosya isimlendirme standardina uyarak dokumani olusturun
3. [_Sablonlar/guvenlik_acigi_prompt_sablonu.md](_Sablonlar/guvenlik_acigi_prompt_sablonu.md) yapisi kullanin
4. README.md dosyasini guncelleyin
5. Bu INDEX.md dosyasina ilgili bolume ekleyin
6. Git commit mesajinda CVE numarasini belirtin

---

## Kaynaklar

- [AGENTS.md](AGENTS.md) - Repository yonergeleri ve gelistirici klavuzu
- [README.md](README.md) - Ana proje aciklamasi ve icerik listesi

---

*Son guncelleme: 2026-02-02*
