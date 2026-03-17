# TODO: OWASP A05:2025 — Enjeksiyon

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A05:2025 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Enjeksiyon/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A05_Injection.md](../OWASP_Top_10/A05_Injection.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Enjeksiyon zafiyetleri, kullanici tarafindan saglanan girdilerin yeterli dogrulama, filtreleme veya temizleme yapilmadan dogrudan yorumlayici motora (SQL, OS komut satiri, NoSQL, LDAP, XPath vb.) gonderilmesi sonucu ortaya cikar. Saldirgan, uygulamanin beklenen komut yapisini degistirerek yetkisiz veri erisimi, veri degisikligi, komut calistirma ve hatta tam sistem ele gecirme gerceklestirebilir. OWASP Top 10:2025'de 5. sirada yer almaktadir.
*   **Etkilenen bilesenler:** Veritabani erisim katmanlari (SQL, NoSQL), isletim sistemi komut calistirma katmanlari, LDAP dizin servisleri, XPath sorgu islemcileri, ORM katmanlari, e-posta basliklari, HTTP basliklari, XML cozumleyicileri

---

### 2. Teknik detay (nasil calisiyor)
*   Enjeksiyon zafiyetleri, uygulamanin kullanici girdilerini guvenilir veri olarak kabul edip dogrudan yorumlayici motora iletmesinden kaynaklanir. Saldirgan, girdiye ozel karakterler ve komut parcalari ekleyerek uygulamanin komut yapisini degistirir.
*   Yaygın ortaya cikis bicimleri:
    *   **SQL Injection:** Kullanici girdisinin dogrudan SQL sorgusuna eklenmesi (`SELECT * FROM users WHERE id = '` + userInput + `'`)
    *   **OS Command Injection:** Kullanici girdisinin dogrudan sistem komutuna iletilmesi (`exec("ping " + userInput)`)
    *   **NoSQL Injection:** MongoDB gibi NoSQL veritabanlarinda operatorlerin manipule edilmesi (`{"$gt": ""}`)
    *   **LDAP Injection:** LDAP sorgularinda filtre manipulasyonu
    *   **XPath Injection:** XML verilerini sorgulayan XPath ifadelerinin manipulasyonu
*   **Neden:** Kullanici girdilerine guvenilmesi, parametreli sorgu yerine string birlestirme ile sorgu olusturulmasi, girdi dogrulama ve temizleme islemlerinin eksikligi, en az yetki ilkesinin uygulanmamasi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Kullanici giris formu ve arka plandaki SQL sorgusu
*   **2. Normal durum:**
    ```
    Kullanici Adi: admin
    Parola: P@ssw0rd123

    SQL Sorgusu: SELECT * FROM users WHERE username='admin' AND password='P@ssw0rd123'
    Sonuc: Basarili giris (eger kimlik bilgileri dogruysa)
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    Kullanici Adi: admin'--
    Parola: herhangibirsey

    SQL Sorgusu: SELECT * FROM users WHERE username='admin'--' AND password='herhangibirsey'
    Sonuc: '--' sonrasi yorum satiri olur, parola kontrolu atlanir
    ```
*   **4. Analiz:** Saldirgan, kullanici adi alanina `admin'--` girerek SQL sorgusunun yapisini degistirir. Tek tirnak (`'`) string degerini kapatir, `--` ise SQL'de yorum satiri baslatir. Bu sayede parola kontrolu tamamen atlanir ve saldirgan `admin` kullanicisi olarak giris yapar.
*   **5. Kanit:** Saldirgan, gecerli parola bilmeden yonetici hesabina erisir. Daha ileri saldirilarla (`UNION SELECT`, `INTO OUTFILE`) tum veritabani dokulebilir veya dosya sistemine yazilabilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, Ic ag, API katmani
*   **Karmasiklik:** Dusuk

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** WAF kurallariyla bilinen enjeksiyon kaliplarini engelleme, kritik girdi noktalarinda temel dogrulama kontrolleri ekleme, veritabani kullanicilarinin yetkilerini kisitlama (en az yetki ilkesi).
*   **Orta vadeli:** Tum veritabani sorgularini parametreli sorgu (prepared statement) kullanacak sekilde yeniden yazma, ORM kullanarak veritabani erisimini soyutlama, kapsamli girdi dogrulama ve temizleme katmani olusturma, SAST/DAST araclariyla otomatik tarama yapma.
*   **Uzun vadeli:** Guvenli kod gelistirme standartlari benimseme, tum gelistiricilere enjeksiyon onleme egitimi verme, DevSecOps surecleriyle guvenlik testlerini CI/CD pipeline'ina entegre etme, duzenli penetrasyon testleri yaptirma.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```php
// SQL Injection - Dogrudan string birlestirme
$username = $_POST['username'];
$password = $_POST['password'];
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysqli_query($conn, $query);

// OS Command Injection - Dogrudan komut calistirma
$ip = $_GET['ip'];
$output = shell_exec("ping -c 4 " . $ip);
echo "<pre>$output</pre>";
```

**Guvenli kod:**
```php
// SQL Injection onleme - Parametreli sorgu (Prepared Statement)
$username = $_POST['username'];
$password = $_POST['password'];

$stmt = $conn->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();

// Daha guvenli yaklasim: Parola hash dogrulama ile
$stmt = $conn->prepare("SELECT id, password_hash FROM users WHERE username = ?");
$stmt->bind_param("s", $username);
$stmt->execute();
$result = $stmt->get_result();
$user = $result->fetch_assoc();
if ($user && password_verify($password, $user['password_hash'])) {
    // Basarili giris
}

// OS Command Injection onleme - escapeshellarg kullanimi
$ip = $_GET['ip'];
// Girdi dogrulama: Sadece gecerli IP adreslerine izin ver
if (filter_var($ip, FILTER_VALIDATE_IP)) {
    $output = shell_exec("ping -c 4 " . escapeshellarg($ip));
    echo "<pre>" . htmlspecialchars($output) . "</pre>";
} else {
    echo "Gecersiz IP adresi.";
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum kullanici girdi noktalarinin envanterini cikar (formlar, API parametreleri, HTTP basliklari)
*   [ ] Mevcut veritabani sorgularinda string birlestirme kullanan noktalari tespit et
*   [ ] OS komut calistiran fonksiyonlari (exec, system, shell_exec, popen) belirle

**Duzeltme**
*   [ ] Tum SQL sorgularini parametreli sorgu (prepared statement) ile degistir
*   [ ] ORM kullanarak veritabani erisim katmanini soyutla
*   [ ] Tum kullanici girdileri icin beyaz liste tabanli dogrulama uygula
*   [ ] OS komut calistirmada escapeshellarg/escapeshellcmd kullan
*   [ ] Veritabani kullanici yetkilerini en az yetki ilkesine gore kisitla
*   [ ] WAF kurallarini enjeksiyon kaliplari icin yapilandir

**Dogrulama**
*   [ ] SAST araclariyla (SonarQube, Semgrep) statik kod analizi yap
*   [ ] DAST araclariyla (OWASP ZAP, Burp Suite) dinamik tarama yap
*   [ ] Manuel enjeksiyon testleri gerceklestir
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Veritabani hata loglarinda SQL syntax hatalarini izle (basarisiz enjeksiyon denemesi gostergesi)
    *   Web sunucu loglarinda supheli SQL kaliplarini tespit et
    *   Anormal veritabani sorgu surelerini ve veri hacimlerini izle
    *   Regex: `(?i)(union\s+select|or\s+1\s*=\s*1|'\s*--\s*|;\s*drop\s+table|exec\s*\(|xp_cmdshell)`
*   **Anomali Tespiti:**
    *   Tek bir kullanicidan kisa surede cok sayida veritabani hatasi
    *   Beklenenden buyuk veri transferleri (veri sizdirma gostergesi)
    *   Veritabani kullanicisi ile beklenmeyen tablo veya schema erisim denemeleri
    *   Web uygulamasindan sistem komutu calistirma girisimlerinin tespiti

---

## Notlar
- OWASP Top 10:2025'de 5. sirada yer alir.
- Detayli rehber: [A05_Injection.md](../OWASP_Top_10/A05_Injection.md)
