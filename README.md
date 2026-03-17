# Guvenlik Kutuphanesi (AppSec Playbook)

Bu depo, **yapay zeka destegiyle yazilim gelistirenler** ve **yazilim guvenligine yeni baslayanlar** icin olusturulmus, yasayan bir guvenlik kutuphanesidir.

**Amaci:** Izlenen yayinlar, okunan makaleler ve guncel guvenlik aciklarini belirli bir sablona gore raporlayarak, teknik derinlikte bogulmadan herkesin anlayabilecegi ve uygulayabilecegi pratik bir rehber sunmaktir. Icerik zamanla, yeni arastirmalarla birlikte zenginlesecektir.

## Klasor Yapisi

```text
Guvenlik-Rehberi/
├── _TODO/                   # CVE-bazli zafiyet takip dosyalari (59 TODO)
│   └── OZET.md              # Tum TODO'larin ozet panosu
├── _Sablonlar/              # Dokuman sablonlari
│   └── todo_sablonu.md      # 8-bolumlu zorunlu TODO formati
└── .claude/                 # Claude Code komutlari
    └── commands/
        └── arastirma-isle.md
```

## TODO Takip Sistemi

Arastirma dokumanlarindan cikarilan her zafiyet/risk icin `_TODO/` klasorunde 8 bolumlu detayli bir analiz dosyasi bulunur:

1. Meta Bilgiler
2. Ozet ve etki
3. Teknik detay (nasil calisiyor)
4. Adim adim istismar (PoC — kavramsal)
5. Risk degerlendirmesi
6. Kalici cozumler ve oneriler
7. Ornek duzeltme kodu
8. Kontrol Listesi (Checklist)
9. Izleme ve uyarilar

Genel durum ozeti icin [`_TODO/OZET.md`](_TODO/OZET.md) dosyasina bakin.

### Durum Tablosu

| Metrik | Sayi |
|--------|------|
| Toplam TODO | 59 |
| Kritik | 23 |
| Yuksek | 29 |
| Orta | 7 |

### Kategori Dagilimi

| Kategori | Sayi |
|----------|------|
| AI IDE Guvenligi | 12 |
| Guvenlik Yapilandirmasi | 10 |
| Isletim Sistemi | 10 |
| Erisim Kontrolu | 5 |
| Tedarik Zinciri | 4 |
| Donanim | 4 |
| Enjeksiyon | 3 |
| OWASP Top 10 | 3 |
| Oturum Yonetimi | 3 |
| Kimlik Dogrulama | 2 |
| Veri Ihlalleri | 2 |
| Mobil | 1 |

### Arastirma Gruplari

| # | Arastirma | TODO Sayisi |
|---|-----------|-------------|
| 1 | Ocak 2026 Guvenlik Aciklari | 24 |
| 2 | Vibecoding Guvenlik Aciklari (Subat 2026) | 19 |
| 3 | OWASP Top 10 2025 Rehber Analizi | 10 |
| 4 | Node.js Guvenlik Acigi Arastirmasi | 4 |
| 5 | Keycloak Zafiyet Arastirmasi | 1 |
| 6 | Django ORM SQL Injection | 1 |

## Yeni Arastirma Islemek

Claude Code ile yeni bir arastirma dokumanini islemek icin:

```
/arastirma-isle Arastirmalar/dosya.md
```

Bu komut arastirmayi okur, icindeki her zafiyet/riski tespit eder, TODO dosyalari olusturur ve `OZET.md` panosunu gunceller.

## Dosya Isimlendirme Kurallari

- CVE varsa: `TODO_cve_YYYY_NNNNN_kisa_aciklama.md`
- Isimli saldiri: `TODO_isim_kisa_aciklama.md`
- Sistemik risk: `TODO_risk_kisa_aciklama.md`
- Sadece kucuk harf ve alt cizgi, Turkce karakter kullanilmaz

## Lisans

Bu depo egitim ve savunma amacli olusturulmustur.
