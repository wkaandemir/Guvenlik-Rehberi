# Claude Code Talimatlar

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

Ornekler:
- `TODO_cve_2026_22708_cursor_shell_bypass.md`
- `TODO_roguepilot_copilot_token_theft.md`
- `TODO_risk_ai_uretimi_kod_guvenligi.md`

### Durum Yonetimi

- **ACIK**: Henuz islem yapilmamis veya devam ediyor
- **KAPANDI**: Tum kontrol listesi maddeleri tamamlandi

Durum degistirme: TODO dosyasindaki `| **Durum** | ACIK |` satirini `KAPANDI` olarak guncelle.

### Kategori Yapisi

CVE analizleri `_TODO/` klasorunde tutulur. Kategori etiketi TODO dosyasinin meta bilgilerinde belirtilir, fiziksel bir klasore karsilik gelmek zorunda degildir.

**Fiziksel kategori klasorleri** (genel rehber dokumanlar icin):

| Klasor | Aciklama |
|--------|----------|
| `Enjeksiyon/` | SQL Injection, Command Injection vb. |
| `Erisim_Kontrolu/` | Broken Access Control, IDOR vb. |
| `Oturum_Yonetimi/` | CSRF, Session Management vb. |
| `OWASP_Top_10/` | OWASP Top 10 2025 guvenlik aciklari |
| `Arastirmalar/` | Derinlemesine arastirma ve raporlar |

**Metadata kategori etiketleri** (TODO dosyalarinda kullanilir):

| Etiket | Aciklama |
|--------|----------|
| `Isletim_Sistemi/` | Windows, Linux, kernel ve surucu zafiyetleri |
| `Donanim/` | CPU/GPU ve donanim seviyesinde zafiyetler |
| `Mobil/` | Android/iOS ve mobil platform zafiyetleri |
| `Guvenlik_Yapilandirmasi/` | Security Misconfiguration vb. |
| `AI_IDE_Guvenligi/` | Vibecoding/AI arac zafiyetleri |
| `Kimlik_Dogrulama/` | Authentication zafiyetleri |
| `Veri_Ihlalleri/` | Buyuk olcekli veri sizintilari |
| `Tedarik_Zinciri/` | Supply chain saldirilari |

### TODO Dokuman Formati (Zorunlu)

Her TODO dosyasi `_Sablonlar/todo_sablonu.md` sablonundaki **8 bolumluk yapi**ya uygun olmalidir.
Bu format `OWASP_Top_10/A06_Insecure_Design.md` referans dosyasiyla ayni detay seviyesindedir.

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
- Yeni TODO olusturulurken bu format kullanilir, eski TODO'lar da bu formata donusturulur

## Genel Kurallar

- Turkce dokumantasyon dili kullanilir
- CVE numaralari her zaman belirtilir
- Mevcut dokumanlara link verilir
- INDEX.md ve README.md guncellenir
- `_Sablonlar/todo_sablonu.md` ana sablondur (8 bolumluk detayli format)
