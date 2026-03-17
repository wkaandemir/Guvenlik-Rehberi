# TODO Ozet Panosu

> Son guncelleme: 2026-03-17

## Genel Durum

| Metrik | Sayi |
|--------|------|
| **Toplam TODO** | 49 |
| **ACIK** | 49 |
| **KAPANDI** | 0 |

## Dokuman Formati

Tum TODO dosyalari **9 bolumluk detayli zafiyet analiz formatinda** hazirlanmistir:

| # | Bolum | Icerik |
|---|-------|--------|
| 1 | Ozet ve etki | Yonetici ozeti + etkilenen bilesenler |
| 2 | Teknik detay | Kok neden analizi, etkilenen mekanizma |
| 3 | Adim adim istismar (PoC) | Kavramsal PoC (egitim amacli) |
| 4 | Risk degerlendirmesi | Kritiklik, CVSS, saldiri yuzeyi, karmasiklik |
| 5 | Kalici cozumler ve oneriler | Kisa / orta / uzun vadeli |
| 6 | Ornek duzeltme kodu | Zafiyetli + guvenli kod ornekleri |
| 7 | Kontrol Listesi (Checklist) | Hazirlik / Duzeltme / Dogrulama |
| 8 | Izleme ve uyarilar | SIEM/Log kurallari, anomali tespiti |

*   **Referans format:** `OWASP_Top_10/A06_Insecure_Design.md`
*   **Sablon:** `_Sablonlar/todo_sablonu.md`
*   **Toplam kontrol maddesi (checkbox):** 703

---

## Arastirma Bazli Gruplandirma

### 1. Ocak 2026 Guvenlik Aciklari Arastirmasi (24 TODO)

| # | TODO Dosyasi | Tanimlayici | Kritiklik | Kategori | Durum |
|---|-------------|-------------|-----------|----------|-------|
| 1 | [TODO_cve_2026_20805](TODO_cve_2026_20805_windows_dwm_bilgi_ifsasi.md) | CVE-2026-20805 | Orta | Isletim_Sistemi/ | ACIK |
| 2 | [TODO_cve_2026_21509](TODO_cve_2026_21509_office_ole_com_bypass.md) | CVE-2026-21509 | Yuksek | Oturum_Yonetimi/ | ACIK |
| 3 | [TODO_cve_2026_21265](TODO_cve_2026_21265_secure_boot_bypass.md) | CVE-2026-21265 | Orta | Guvenlik_Yapilandirmasi/ | ACIK |
| 4 | [TODO_cve_2026_20854](TODO_cve_2026_20854_windows_lsass_rce.md) | CVE-2026-20854 | Yuksek | Isletim_Sistemi/ | ACIK |
| 5 | [TODO_cve_2026_20822](TODO_cve_2026_20822_windows_graphics_eop.md) | CVE-2026-20822 | Yuksek | Isletim_Sistemi/ | ACIK |
| 6 | [TODO_cve_2026_20045](TODO_cve_2026_20045_cisco_unified_cm_rce.md) | CVE-2026-20045 | Kritik | Guvenlik_Yapilandirmasi/ | ACIK |
| 7 | [TODO_cve_2026_24858](TODO_cve_2026_24858_fortios_sso_bypass.md) | CVE-2026-24858 | Kritik | Guvenlik_Yapilandirmasi/ | ACIK |
| 8 | [TODO_cve_2025_8110](TODO_cve_2025_8110_gogs_symlink.md) | CVE-2025-8110 | Kritik | Erisim_Kontrolu/ | ACIK |
| 9 | [TODO_cve_2025_68664](TODO_cve_2025_68664_langgrinch_langchain.md) | CVE-2025-68664 | Kritik | Erisim_Kontrolu/ | ACIK |
| 10 | [TODO_cve_2026_23000](TODO_cve_2026_23000_linux_mlx5e_uaf.md) | CVE-2026-23000 | Yuksek | Isletim_Sistemi/ | ACIK |
| 11 | [TODO_cve_2022_3564](TODO_cve_2022_3564_bluetooth_l2cap_rce.md) | CVE-2022-3564 | Yuksek | Isletim_Sistemi/ | ACIK |
| 12 | [TODO_cve_2025_66516](TODO_cve_2025_66516_apache_tika_xxe.md) | CVE-2025-66516 | Kritik | OWASP_Top_10/ | ACIK |
| 13 | [TODO_cve_2026_21962](TODO_cve_2026_21962_oracle_weblogic_proxy_rce.md) | CVE-2026-21962 | Kritik | Guvenlik_Yapilandirmasi/ | ACIK |
| 14 | [TODO_cve_2026_21945](TODO_cve_2026_21945_oracle_mysql_java_ssrf.md) | CVE-2026-21945 | Orta | Erisim_Kontrolu/ | ACIK |
| 15 | [TODO_android_dolby](TODO_android_dolby_zero_click_rce.md) | Android Dolby Zero-Click | Kritik | Mobil/ | ACIK |
| 16 | [TODO_cve_2025_33219](TODO_cve_2025_33219_nvidia_integer_overflow.md) | CVE-2025-33219 | Yuksek | Donanim/ | ACIK |
| 17 | [TODO_cve_2025_33218](TODO_cve_2025_33218_nvidia_integer_overflow.md) | CVE-2025-33218 | Yuksek | Donanim/ | ACIK |
| 18 | [TODO_cve_2025_33220](TODO_cve_2025_33220_nvidia_vgpu_uaf.md) | CVE-2025-33220 | Kritik | Donanim/ | ACIK |
| 19 | [TODO_amd_sb_7038](TODO_amd_sb_7038_memory_disorder.md) | AMD-SB-7038 | Orta | Donanim/ | ACIK |
| 20 | [TODO_esa_veri_ihlali](TODO_esa_veri_ihlali_ocak_2026.md) | ESA Veri Ihlali | Kritik | YENI: Veri_Ihlalleri/ | ACIK |
| 21 | [TODO_nike_veri_ihlali](TODO_nike_veri_ihlali_ocak_2026.md) | Nike Veri Ihlali | Yuksek | YENI: Veri_Ihlalleri/ | ACIK |
| 22 | [TODO_ledger_tedarik](TODO_ledger_tedarik_zinciri_veri_ihlali.md) | Ledger/Trust Wallet | Yuksek | YENI: Tedarik_Zinciri/ | ACIK |
| 23 | [TODO_ai_silahlandirilmasi](TODO_risk_ai_silahlandirilmasi_shadow_ai.md) | AI Silahlandirilmasi | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 24 | [TODO_volt_typhoon](TODO_risk_volt_typhoon_on_konumlandirma.md) | Volt Typhoon | Kritik | YENI: Tedarik_Zinciri/ | ACIK |

### 2. Node.js Guvenlik Acigi Arastirmasi (4 TODO)

| # | TODO Dosyasi | Tanimlayici | Kritiklik | Kategori | Durum |
|---|-------------|-------------|-----------|----------|-------|
| 25 | [TODO_cve_2025_55182](TODO_cve_2025_55182_react2shell_rce.md) | CVE-2025-55182 | Kritik | Oturum_Yonetimi/ | ACIK |
| 26 | [TODO_cve_2025_66478](TODO_cve_2025_66478_nextjs_rce.md) | CVE-2025-66478 | Kritik | Oturum_Yonetimi/ | ACIK |
| 27 | [TODO_cve_2025_27209](TODO_cve_2025_27209_nodejs_v8_hashdos.md) | CVE-2025-27209 | Yuksek | Enjeksiyon/ | ACIK |
| 28 | [TODO_cve_2025_27210](TODO_cve_2025_27210_nodejs_windows_path_traversal.md) | CVE-2025-27210 | Yuksek | Erisim_Kontrolu/ | ACIK |

### 3. Keycloak Zafiyet Arastirmasi ve Analizi (1 TODO)

| # | TODO Dosyasi | Tanimlayici | Kritiklik | Kategori | Durum |
|---|-------------|-------------|-----------|----------|-------|
| 29 | [TODO_cve_2026_1529](TODO_cve_2026_1529_keycloak_org_invite.md) | CVE-2026-1529 | Yuksek | YENI: Kimlik_Dogrulama/ | ACIK |

### 4. Vibecoding Guvenlik Aciklari — Subat 2026 (19 TODO)

| # | TODO Dosyasi | Tanimlayici | Kritiklik | Kategori | Durum |
|---|-------------|-------------|-----------|----------|-------|
| 30 | [TODO_cve_2026_22708](TODO_cve_2026_22708_cursor_shell_bypass.md) | CVE-2026-22708 | Kritik | YENI: AI_IDE_Guvenligi/ | ACIK |
| 31 | [TODO_cve_2026_21516](TODO_cve_2026_21516_copilot_jetbrains_rce.md) | CVE-2026-21516 | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 32 | [TODO_roguepilot](TODO_roguepilot_copilot_token_theft.md) | RoguePilot | Kritik | YENI: AI_IDE_Guvenligi/ | ACIK |
| 33 | [TODO_cve_2026_25725](TODO_cve_2026_25725_claude_code_sandbox_escape.md) | CVE-2026-25725 | Kritik | YENI: AI_IDE_Guvenligi/ | ACIK |
| 34 | [TODO_cve_2026_21852](TODO_cve_2026_21852_claude_code_api_key_leak.md) | CVE-2026-21852 | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 35 | [TODO_ai_ide_altyapi](TODO_ai_ide_altyapi_94_cve_cursor_windsurf.md) | AI-IDE 94+ CVE | Kritik | YENI: AI_IDE_Guvenligi/ | ACIK |
| 36 | [TODO_contextcrush](TODO_contextcrush_mcp_zehirlenmesi.md) | ContextCrush | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 37 | [TODO_replit_agent](TODO_replit_agent_veri_kaybi.md) | Replit Agent | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 38 | [TODO_susvibes](TODO_risk_ai_uretimi_kod_guvenligi.md) | SusVibes / AI Kod | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 39 | [TODO_owasp_asi](TODO_owasp_asi_top_10_ajan_guvenligi.md) | OWASP ASI Top 10 | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 40 | [TODO_cve_2026_21510](TODO_cve_2026_21510_windows_shell_bypass.md) | CVE-2026-21510 | Kritik | Isletim_Sistemi/ | ACIK |
| 41 | [TODO_cve_2026_21513](TODO_cve_2026_21513_mshtml_bypass.md) | CVE-2026-21513 | Kritik | Isletim_Sistemi/ | ACIK |
| 42 | [TODO_cve_2026_21519](TODO_cve_2026_21519_dwm_eop.md) | CVE-2026-21519 | Yuksek | Isletim_Sistemi/ | ACIK |
| 43 | [TODO_cve_2026_21533](TODO_cve_2026_21533_rds_eop.md) | CVE-2026-21533 | Yuksek | Isletim_Sistemi/ | ACIK |
| 44 | [TODO_cve_2026_21525](TODO_cve_2026_21525_ras_dos.md) | CVE-2026-21525 | Orta | Isletim_Sistemi/ | ACIK |
| 45 | [TODO_cve_2026_20127](TODO_cve_2026_20127_cisco_sdwan_auth_bypass.md) | CVE-2026-20127 | Kritik | Guvenlik_Yapilandirmasi/ | ACIK |
| 46 | [TODO_cve_2026_22769](TODO_cve_2026_22769_dell_recoverpoint_hardcoded.md) | CVE-2026-22769 | Kritik | Guvenlik_Yapilandirmasi/ | ACIK |
| 47 | [TODO_raph_wiggum](TODO_risk_vibecoding_gelistirici_rehaveti.md) | Raph Wiggum Dongusu | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |
| 48 | [TODO_mcp_sunucu](TODO_risk_mcp_sunucu_guvenligi.md) | MCP Sunucu Guvenligi | Yuksek | YENI: AI_IDE_Guvenligi/ | ACIK |

### 5. Bagimsiz Rehber Kaynaklari (1 TODO)

| # | TODO Dosyasi | Tanimlayici | Kritiklik | Kategori | Durum |
|---|-------------|-------------|-----------|----------|-------|
| 49 | [TODO_cve_2025_64459](TODO_cve_2025_64459_django_orm_sql_injection.md) | CVE-2025-64459 | Yuksek | Enjeksiyon/ | ACIK |

---

## Kritiklik Dagilimi

| Kritiklik | Sayi |
|-----------|------|
| Kritik | 20 |
| Yuksek | 24 |
| Orta | 5 |
| Dusuk | 0 |

## Kategori Dagilimi

| Kategori | Sayi | Durum |
|----------|------|-------|
| AI_IDE_Guvenligi/ | 12 | Klasor henuz olusturulmadi |
| Isletim_Sistemi/ | 10 | Klasor silindi — icerik _TODO'da |
| Guvenlik_Yapilandirmasi/ | 7 | Klasor silindi — icerik _TODO'da |
| Donanim/ | 4 | Klasor silindi — icerik _TODO'da |
| Erisim_Kontrolu/ | 4 | Mevcut (1 genel rehber) |
| Oturum_Yonetimi/ | 3 | Mevcut (1 genel rehber) |
| Tedarik_Zinciri/ | 2 | Klasor henuz olusturulmadi |
| Veri_Ihlalleri/ | 2 | Klasor henuz olusturulmadi |
| OWASP_Top_10/ | 1 | Mevcut (A01-A10 genel rehberler) |
| Mobil/ | 1 | Klasor silindi — icerik _TODO'da |
| Enjeksiyon/ | 2 | Mevcut (3 genel rehber) |
| Kimlik_Dogrulama/ | 1 | Klasor henuz olusturulmadi |

## Mevcut Dokuman Durumu

| Durum | Sayi |
|-------|------|
| Silindi — icerik TODO dosyasinda | 21 |
| Henuz yok | 26 |
| Arastirma dosyasinda mevcut | 1 |
| Rehber dokumani mevcut | 1 |

---

*Bu dosya `/arastirma-isle` komutu veya "Bu arastirmayi isle" tetiklemesi ile otomatik guncellenir.*
