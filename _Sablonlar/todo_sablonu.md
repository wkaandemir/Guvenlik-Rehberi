# TODO: [Zafiyet Tanimlayici] — [Kisa Baslik]

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | [CVE-XXXX-XXXXX veya Isim] |
| **Kritiklik** | [Dusuk / Orta / Yuksek / Kritik] |
| **Kategori Klasoru** | [Mevcut klasor veya YENI: onerilen_klasor/] |
| **Kaynak Arastirma** | [link] |
| **Tarih** | [YYYY-MM-DD] |
| **Mevcut Dokuman** | [link veya "Henuz yok"] |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** [1-3 cumle — zafiyetin ne oldugu, kimi etkiledigi, neden onemli oldugu]
*   **Etkilenen bilesenler:** [Etkilenen yazilim, surum, platform veya altyapi bilesenleri]

---

### 2. Teknik detay (nasil calisiyor)
*   [Zafiyetin teknik aciklamasi — kok neden, nasil ortaya cikiyor]
*   [Etkilenen protokol, API, fonksiyon veya mekanizma]
*   **Neden:** [Kok neden analizi — neden bu zafiyet olusabildi]

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** [Hedef bilesen veya servis]
*   **2. Normal durum:**
    ```
    [Normal is akisi adimlari]
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    [Saldiri adimlari veya ornek payload]
    ```
*   **4. Analiz:** [Neden calisiyor — teknik aciklama]
*   **5. Kanit:** [Basarili istismar durumunda beklenen sonuc]

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** [Dusuk / Orta / Yuksek / Kritik]
*   **CVSS Skoru:** [Varsa CVSS 3.x skoru]
*   **Saldiri Yuzeyi:** [Internete acik / Ic network / Yerel erisim]
*   **Karmasiklik:** [Dusuk / Orta / Yuksek]

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** [Hemen uygulanacak gecici onlemler ve yamalar]
*   **Orta vadeli:** [Yapilandirma degisiklikleri, guncelleme planlari]
*   **Uzun vadeli:** [Mimari iyilestirmeler, politika degisiklikleri, egitim]

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```
[Zafiyetli kod ornegi — dil etiketiyle]
```

**Guvenli kod:**
```
[Duzeltilmis kod ornegi — dil etiketiyle]
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] [Envanter/tespit adimlari]

**Duzeltme**
*   [ ] [Yama/yapilandirma adimlari]

**Dogrulama**
*   [ ] [Test/dogrulama adimlari]

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   [Izlenecek log kaynaklari ve olaylar]
    *   Regex: `[Ornek regex kaliplari]`
*   **Anomali Tespiti:**
    *   [Izlenmesi gereken anormal davranislar]

---

## Notlar
[Ek bilgiler, iliskili CVE'ler, referanslar]
