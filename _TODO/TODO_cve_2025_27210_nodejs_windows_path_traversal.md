# TODO: CVE-2025-27210 — Node.js Windows Yol Gecisi Zafiyeti

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-27210 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Erisim_Kontrolu/ |
| **Kaynak Arastirma** | [Node.js Güvenlik Açığı Araştırması.md](../Arastirmalar/Node.js%20G%C3%BCvenlik%20A%C3%A7%C4%B1%C4%9F%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2025-12-20 |
| **Mevcut Dokuman** | Henuz yok |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-27210, Node.js'in Windows isletim sistemi uzerindeki dosya yolu (path) islemlerinde, Windows'a ozel rezerve cihaz adlarinin (CON, PRN, AUX, NUL, COM1-COM9, LPT1-LPT9) yeterince filtrelenmemesinden kaynaklanan bir yol gecisi (Path Traversal) zafiyetidir. Saldirganlar, bu rezerve isimleri dosya yolu parametrelerinde kullanarak Node.js uygulamalarinin beklenen dizin sinirlarinin disina cikmalarini saglayabilir, yetkisiz dosya okuma ve yazma islemleri gerceklestirebilir. Zafiyet, React2Shell (CVE-2025-55182) ile birlestirildiginde Windows sunucularda dosya sistemi uzerinde tam hakimiyet kurulmasini kolaylastirabilir. Node.js v20, v22, v24 ve v25 surumlerini etkiler ve 7 Ocak 2026 guvenlik yamasinin bir parcasidir.
*   **Etkilenen bilesenler:** Windows uzerinde calisan Node.js v20.x, v22.x, v24.x, v25.x surumlerinin `path` ve `fs` modulleri, kullanici girdisinden dosya yolu olusturan tum Node.js web uygulamalari (dosya yukleme, indirme, statik dosya servisi), Windows IIS arkasinda calisan Node.js servisleri

---

### 2. Teknik detay (nasil calisiyor)
*   Windows isletim sistemi, MS-DOS'tan miras kalan belirli dosya adlarini "rezerve cihaz adlari" olarak ozel isler. `CON` (konsol), `PRN` (yazici), `AUX` (yardimci), `NUL` (bos cihaz), `COM1`-`COM9` (seri portlar) ve `LPT1`-`LPT9` (paralel portlar) gibi isimler, dosya uzantisi eklense bile (ornegin `CON.txt`) Windows tarafindan cihaz olarak yorumlanir.
*   Node.js'in `path.resolve()`, `path.normalize()` ve `path.join()` fonksiyonlari, bu Windows'a ozel rezerve isimleri yeterince filtrelememektedir. Saldirgan, dosya yolu parametresine `CON`, `PRN` gibi degerler veya bunlarin `.txt`, `.log` gibi uzantili varyantlarini gondererek, uygulamanin beklenen dizin yapisinin disindaki kaynaklara erisim saglayabilir.
*   Ozellikle `AUX` ve `CON` cihaz adlari kullanildiginda, Node.js uygulamasi beklenmedik dosya sistemi davranislari sergileyebilir: dosya yazma islemleri farkli hedeflere yonlendirilebilir, dosya okuma islemleri sistem cihazlarindan veri cekebilir veya uygulamanin sandbox sinirlarini asmasi mumkun olabilir.
*   Bu zafiyet, ozellikle React2Shell (CVE-2025-55182) ile birlikte kullanildiginda tehlikeli bir saldiri zinciri olusturur: React2Shell ile sunucuda kod yurutme kazanan saldirgan, bu path traversal zafiyetini kullanarak Windows dosya sisteminde sinirsiz hareket edebilir.
*   **Neden:** Kok neden, Node.js'in `path` modulunun Windows rezerve cihaz adlarini platform-bagimli bir guvenlik kontrolu olarak filtrelememesidir. Cross-platform uyumlulugun onceliklenmesi nedeniyle Windows'a ozel guvenlik kontrolleri ihmal edilmis ve kullanici girdisinden gelen dosya yollarinin yeterince sanitize edilmemesi bu zafiyete yol acmistir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Windows sunucusunda calisan Node.js web uygulamasi (dosya yukleme/indirme endpoint'i, statik dosya servisi veya log goruntuleme arayuzu)
*   **2. Normal durum:**
    ```
    1. Kullanici, dosya indirme endpointine gecerli bir dosya adi gonderir: GET /download?file=rapor.pdf
    2. Node.js, path.join(uploadDir, 'rapor.pdf') ile dosya yolunu olusturur
    3. Dosya, beklenen dizin icinden okunur ve kullaniciya iletilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```http
    GET /download?file=....\\....\\windows\\system32\\drivers\\etc\\hosts HTTP/1.1
    Host: hedef-sunucu.com

    # Alternatif: Windows rezerve cihaz adlariyla bypass
    GET /download?file=CON HTTP/1.1
    GET /download?file=AUX.txt HTTP/1.1
    GET /download?file=....\\CON\\....\\windows\\win.ini HTTP/1.1

    # path.normalize() bypass varyanti:
    GET /download?file=..%5c..%5cwindows%5csystem32%5cdrivers%5cetc%5chosts HTTP/1.1
    ```
*   **4. Analiz:** Node.js'in `path.join()` fonksiyonu, Windows rezerve cihaz adlarini (`CON`, `AUX`, `PRN` vb.) dosya yolu bileseni olarak kabul eder ve bunlari filtrelemez. `path.normalize()` ise `..\\` dizilerini cozumlerken bu ozel isimleri atlayabilir. Sonuc olarak, saldirganin gonderdigi yol, `uploadDir` sinirlarinin disina cikar ve Windows dosya sistemindeki keyfi dosyalara erismesine izin verir. Windows'un buyuk/kucuk harf duyarsizligi (`con`, `CON`, `Con` hepsi gecerli) durumu daha da zorlastirir.
*   **5. Kanit:** Saldirgan, `C:\Windows\System32\drivers\etc\hosts` veya `C:\Windows\win.ini` gibi sistem dosyalarinin icerigini HTTP yaniti olarak okuyabilir. Dosya yazma endpointlerinde ise beklenen dizin disina dosya yazarak sistem yapilandirmasini degistirebilir veya web shell yerlestirebilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.5 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N)
*   **Saldiri Yuzeyi:** Internete acik Windows Node.js sunuculari (dosya isleme arayuzleri, API servisleri)
*   **Karmasiklik:** Dusuk (standart path traversal teknikleri ile saldiri mumkun, Windows'a ozel cihaz adlari ek bypass yontemi saglar)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Node.js'i 7 Ocak 2026 guvenlik surumune guncelleyin. Dosya yolu parametrelerinde Windows rezerve cihaz adlarini (CON, PRN, AUX, NUL, COM1-9, LPT1-9) filtreleyen ozel bir dogrulama fonksiyonu uygulayin. Tum dosya erisim endpointlerinde `path.resolve()` sonucunun beklenen temel dizin (`baseDir`) icinde kaldigini kontrol edin. Windows sunucularinda Node.js uygulamalarini minimum yetkili kullanici hesabiyla calistirin.
*   **Orta vadeli:** Dosya yukleme ve indirme islemlerinde kullanici girdisinden dosya yolu olusturmak yerine, UUID tabanli dosya adlandirma sistemi kullanin. Statik dosya servisi icin `serve-static` veya `send` gibi guvenlik kontrolu iceren kutuphaneleri tercih edin. Windows dosya sistemi erisim loglarini SIEM'e aktarin ve anomali tespiti yapilandirin.
*   **Uzun vadeli:** Mumkun oldugunda Node.js uygulamalarini Linux konteynerlerine tasiyarak Windows'a ozel path traversal risklerini ortadan kaldirin. CI/CD pipeline'inda path traversal guvenlik testlerini (OWASP ZAP, Burp Suite) otomatik calistirin. Dosya sistemi erisim katmaninda zorunlu sandbox (chroot benzeri) mekanizmalar uygulayin. Node.js guvenlik surumlerini otomatik izleyen uyari sistemi kurun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```javascript
// === GUVENSIZ: Windows cihaz adlarini ve path traversal'i filtrelemeden dosya erisimi ===
const path = require('path');
const fs = require('fs');
const express = require('express');
const app = express();

const uploadDir = 'C:\\app\\uploads';

app.get('/download', (req, res) => {
    const fileName = req.query.file;
    // Dogrudan kullanici girdisinden yol olusturuluyor — path traversal'a acik!
    const filePath = path.join(uploadDir, fileName);
    fs.readFile(filePath, (err, data) => {
        if (err) return res.status(404).send('Dosya bulunamadi');
        res.send(data);
    });
});
```

**Guvenli kod:**
```javascript
// === GUVENLI: Windows cihaz adlari + path traversal engelleme + baseDir kontrolu ===
const path = require('path');
const fs = require('fs');
const express = require('express');
const app = express();

const uploadDir = path.resolve('C:\\app\\uploads');

// Windows rezerve cihaz adlari (buyuk/kucuk harf duyarsiz)
const WINDOWS_RESERVED = /^(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])(\.|$)/i;
const PATH_TRAVERSAL = /\.\.[\/\\]/;
const NULL_BYTE = /\0/;

function sanitizeFilePath(userInput, baseDir) {
    // 1. Null byte kontrolu
    if (NULL_BYTE.test(userInput)) {
        throw new Error('Null byte tespit edildi');
    }

    // 2. Path traversal kontrolu
    if (PATH_TRAVERSAL.test(userInput)) {
        throw new Error('Path traversal tespit edildi');
    }

    // 3. Windows rezerve cihaz adlarini engelle
    const basename = path.basename(userInput);
    if (WINDOWS_RESERVED.test(basename)) {
        throw new Error('Rezerve Windows cihaz adi tespit edildi');
    }

    // 4. Normalize et ve baseDir icinde kaldigini dogrula
    const resolved = path.resolve(baseDir, userInput);
    if (!resolved.startsWith(baseDir + path.sep) && resolved !== baseDir) {
        throw new Error('Dosya yolu izin verilen dizin sinirlarinin disinda');
    }

    return resolved;
}

app.get('/download', (req, res) => {
    try {
        const safePath = sanitizeFilePath(req.query.file, uploadDir);
        fs.readFile(safePath, (err, data) => {
            if (err) return res.status(404).json({ error: 'Dosya bulunamadi' });
            res.send(data);
        });
    } catch (e) {
        res.status(400).json({ error: e.message });
    }
});
```

```bash
# Node.js'i yamali surume guncelle (7 Ocak 2026 guvenlik surumu)
node --version
nvm install --lts && nvm use --lts

# Windows sunucularinda Node.js'i minimum yetkili kullanici ile calistir
# PowerShell: Servis kullanicisi olustur
# New-LocalUser -Name "nodeapp" -NoPassword
# Icacls C:\app /grant nodeapp:(OI)(CI)RX
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Windows uzerinde calisan tum Node.js uygulamalarini envanterle
*   [ ] Dosya sistemi erisimi yapan endpointleri belirle (yukleme, indirme, statik dosya, log goruntuleme)
*   [ ] Mevcut yol dogrulama ve sanitasyon mekanizmalarini gozden gecir
*   [ ] path.join(), path.resolve(), path.normalize() kullanimlarini kod tabaninda ara
*   [ ] React2Shell (CVE-2025-55182) zafiyetinin de yamandigini dogrula (zincirleme risk)

**Duzeltme**
*   [ ] Node.js'i 7 Ocak 2026 guvenlik surumune guncelle (v20, v22, v24, v25)
*   [ ] Windows rezerve cihaz adlarini filtreleyen dosya yolu dogrulama fonksiyonu ekle
*   [ ] Tum dosya erisim endpointlerinde baseDir kontrolu uygula
*   [ ] path.resolve() sonucunun her zaman beklenen dizin icinde kaldigini dogrulayan middleware ekle
*   [ ] Node.js uygulamasini minimum yetkili kullanici hesabiyla calistir

**Dogrulama**
*   [ ] Windows cihaz adlari (CON, PRN, AUX, NUL, COM1-9, LPT1-9) ile yol gecisi testini tekrarla
*   [ ] `..\\` ve `%5c` kodlamali path traversal testlerini uygula
*   [ ] React2Shell + Path Traversal zincirleme saldiri senaryosunu test et
*   [ ] Dosya sistemi erisim loglarini SIEM'de izle ve anomali kurallarini dogrula
*   [ ] CI/CD pipeline'inda otomatik path traversal tarama sonuclarini kontrol et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Windows dosya sistemi erisim loglarinda (Security Event ID 4663) beklenmeyen dizin erisimlerini izle
    *   Node.js uygulama loglarinda dosya yolu hatalarini ve `ENOENT`/`EACCES` hatalarini filtrele
    *   Regex: `(?i)(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])(\.|\\|\/|$)`
    *   Regex: `(?i)(\.\.[\\/]|%2e%2e[\\/]|%252e%252e[\\/]|\.\.%5c|\.\.%2f)`
    *   IIS/reverse proxy loglarinda path traversal belirtilerini (`..\\`, `..%5c`) ara
*   **Anomali Tespiti:**
    *   Node.js uygulamasinin beklenen dizin (uploadDir) disindaki dosyalara erisim denemesi
    *   Tek IP adresinden kisa surede farkli dosya yolu varyantlari iceren istekler (fuzzing belirtisi)
    *   Dosya indirme endpointinden sistem dosyalarinin (`win.ini`, `hosts`, `SAM`) talep edilmesi
    *   Node.js islemi altindan Windows sistem dizinlerine (`C:\Windows\`, `C:\Users\`) beklenmeyen erisim
    *   Dosya yazma endpointinde beklenen uzanti disinda dosya olusturma girisimleri (`.exe`, `.bat`, `.ps1`)

---

## Notlar
Yalnizca Windows sunucularini etkiler — Linux/macOS Node.js kurulumlari bu zafiyetten etkilenmez. React2Shell (CVE-2025-55182) ile birlestirildiginde risk seviyesi onemli olcude artar: ilk RCE ile sunucuya sizen saldirgan, path traversal ile dosya sisteminde sinirsiz hareket edebilir. Ocak 2026 guvenlik surumunun bir parcasidir (CVE-2025-27209 ile birlikte degerlendirilmeli). Windows'a ozel rezerve cihaz adlari, klasik `../` path traversal'in otesinde ek bir bypass vektoru olusturur.
