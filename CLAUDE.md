# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proje Hakkinda

Turkce guvenlik rehberi ve zafiyet izleme deposu. Kod icermez — tamami Markdown dokumanlardan olusur. Build, lint veya test komutu yoktur.

## Repo Mimarisi (Iki Katmanli Model)

**Katman 1 — Genel rehber dokumanlar** (fiziksel klasorler):

| Klasor | Icerik |
|--------|--------|
| `OWASP_Top_10/` | A01-A10 genel guvenlik acigi rehberleri |
| `Enjeksiyon/` | SQL Injection, Command Injection vb. rehberler |
| `Erisim_Kontrolu/` | Broken Access Control, IDOR vb. rehberler |
| `Oturum_Yonetimi/` | CSRF, Session Management vb. rehberler |
| `Arastirmalar/` | Derinlemesine arastirma raporlari (islenmemis veya islenmis) |

**Katman 2 — CVE/zafiyet izleme** (tek klasor, metadata ile kategorize):

| Klasor | Icerik |
|--------|--------|
| `_TODO/` | Her zafiyet icin bir `TODO_*.md` dosyasi + `OZET.md` panosu |
| `_Sablonlar/` | `todo_sablonu.md` — TODO dosyalari icin zorunlu 8-bolum sablonu |

Kategori bilgisi TODO dosyalarinin meta tablosundaki **Kategori Klasoru** alaninda tutulur, ayri fiziksel klasor olusturulmaz.

## Slash Komutlari

- `/arastirma-isle <dosya_yolu>` — Bir arastirma dokumanini isleyerek icindeki her zafiyet icin TODO dosyasi olusturur

## Arastirma Isleme Is Akisi

### Tetikleme
- "Bu arastirmayi isle" veya "Bu dosyayi isle" ifadeleri
- `/arastirma-isle Arastirmalar/dosya.md` slash komutu
- "@dosya isle" kaliplari

### 6 Adimli Algoritma

1. **Oku**: Belirtilen arastirma dokumanini tamamen oku
2. **Cikar**: Her zafiyet/riski tespit et (CVE, isimli saldiri, sistemik risk)
3. **Siniflandir**: Kritiklik belirle, uygun kategori etiketini ata
4. **Olustur**: Her zafiyet icin `_TODO/TODO_[slug].md` dosyasi olustur
5. **Guncelle**: `_TODO/OZET.md` ozet panosunu guncelle
6. **Raporla**: Kullaniciya ozet tablo sun

### Zafiyet Tanimlama Kurallari

**CVE Tanima:**
- Regex: `CVE-\d{4}-\d{4,7}`
- Her CVE ayri bir TODO dosyasi olur

**Isimli Saldiri Tanima:**
- Buyuk harfle baslayan veya tamami buyuk harf olan teknik terimler (orn: React2Shell, RoguePilot, ContextCrush, LangGrinch)
- Bilinen saldiri grubu isimleri (orn: Volt Typhoon, Earth Lamia)
- Her isimli saldiri ayri bir TODO olur

**Sistemik Risk Tanima:**
- "Guvenlik borcu", "tedarik zinciri riski", "veri ihlali" gibi ifadeler
- Belirli bir CVE veya isme sahip olmayan ama aksiyonellebilir riskler
- Onemli olanlar gruplanarak TODO olusturulur

### Kritiklik Belirleme Kurallari

| CVSS Skoru | Kritiklik |
|------------|-----------|
| 9.0 - 10.0 | Kritik |
| 7.0 - 8.9 | Yuksek |
| 4.0 - 6.9 | Orta |
| 0.1 - 3.9 | Dusuk |
| CVSS yok (aktif istismar) | Yuksek |
| CVSS yok (teorik risk) | Orta |

### Dosya Isimlendirme Standardi

Format: `_TODO/TODO_[slug_snake_case].md`

Kurallar:
- CVE numarasi varsa: `TODO_cve_YYYY_NNNNN_kisa_aciklama.md`
- Isimli saldiri: `TODO_isim_kisa_aciklama.md`
- Sistemik risk: `TODO_risk_kisa_aciklama.md`
- Sadece kucuk harf, tire yerine alt cizgi
- Turkce karakter kullanilmaz (ascii only)

### Durum Yonetimi

- **ACIK**: Henuz islem yapilmamis veya devam ediyor
- **KAPANDI**: Tum kontrol listesi maddeleri tamamlandi

Durum degistirme: TODO dosyasindaki `| **Durum** | ACIK |` satirini `KAPANDI` olarak guncelle.

### Kategori Etiketleri (metadata icin)

`Isletim_Sistemi/`, `Donanim/`, `Mobil/`, `Guvenlik_Yapilandirmasi/`, `AI_IDE_Guvenligi/`, `Kimlik_Dogrulama/`, `Veri_Ihlalleri/`, `Tedarik_Zinciri/`

### TODO Dokuman Formati (Zorunlu)

Her TODO dosyasi `_Sablonlar/todo_sablonu.md` sablonundaki **8 bolumluk yapi**ya uygun olmalidir.

**Zorunlu bolumler (sirasi degismez):**

1. **Meta Bilgiler** — Durum tablosu (ACIK/KAPANDI, CVE, kritiklik, kategori, kaynak, tarih, mevcut dokuman)
2. **Ozet ve etki** — Yonetici ozeti + etkilenen bilesenler
3. **Teknik detay (nasil calisiyor)** — Kok neden analizi, etkilenen mekanizma
4. **Adim adim istismar (PoC — kavramsal)** — Egitim amacli kavramsal PoC
5. **Risk degerlendirmesi** — Kritiklik, CVSS, saldiri yuzeyi, karmasiklik
6. **Kalici cozumler ve oneriler** — Kisa/orta/uzun vadeli cozumler
7. **Ornek duzeltme kodu** — Zafiyetli + guvenli kod ornekleri (ilgili dilde)
8. **Kontrol Listesi (Checklist)** — Hazirlik / Duzeltme / Dogrulama
9. **Izleme ve uyarilar** — SIEM/Log kurallari, regex kaliplari, anomali tespiti

**Kurallar:**
- Hicbir bolum atlanamaz, icerik zafiyete ozgu doldurulur
- Kod ornekleri ilgili dilde (Java, Python, PowerShell, bash vb.) ve dil etiketi ile yazilir
- PoC bolumu yalnizca egitim ve savunma amaclidir, uyari notu zorunludur

## Genel Kurallar

- Turkce dokumantasyon dili kullanilir
- CVE numaralari her zaman belirtilir
- Dosya iceriginde Turkce karakter kullanilabilir, dosya adlarinda kullanilmaz (ascii only)
- Mevcut dokumanlara link verilir
- `_TODO/OZET.md` her TODO ekleme/silme isleminden sonra guncellenir
