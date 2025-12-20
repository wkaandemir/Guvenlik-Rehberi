### Güvenlik Açığı: Node.js ve React Ekosistemi - React2Shell (CVE-2025-55182) & Next.js RCE (CVE-2025-66478)

### 1. Özet ve etki
*   **Yönetici Özeti:** 2025 yılının son çeyreğinde ortaya çıkan React2Shell (CVE-2025-55182), React Server Components (RSC) mimarisini etkileyen ve CVSS 3.1 standardına göre 10.0 (Kritik) tam puan alan bir uzaktan kod yürütme (RCE) zafiyetidir. Bu zafiyet, saldırganların herhangi bir kimlik doğrulaması olmadan, tek bir manipüle edilmiş HTTP isteği ile sunucu üzerinde tam kontrol sağlamasına olanak tanır. Next.js gibi popüler çerçevelerin "App Router" yapısını kullanan tüm güncellenmemiş sürümleri doğrudan etkilenmektedir. İşletmeler için bu durum, veri sızıntısı, sistemin fidye yazılımlarınca ele geçirilmesi ve itibar kaybı gibi felaket senaryoları anlamına gelmektedir.
*   **Etkilenen bileşenler:**
    *   **React:** 19.0.0, 19.1.0, 19.1.1, 19.2.0 sürümleri.
    *   **Next.js:** App Router kullanan 15.0.x, 15.1.x, 15.2.x, 16.0.x sürümleri ve belirli Canary bültenleri.
    *   **Protokol:** React Flight (RSC iletişim protokolü).
    *   **Altyapı:** Node.js çalışma zamanı (Runtime).

---

### 2. Teknik detay (nasıl çalışıyor)
*   **Mekanizma:** Zafiyet, React Server Components'in sunucu ve istemci arasında veri taşıma mekanizması olan "Flight" protokolündeki güvensiz tersine serileştirme (insecure deserialization) işleminden kaynaklanır. Flight protokolü, bileşen ağacını ve verileri transfer ederken `$X` gibi özel etiketler kullanır.
*   **Arka Plan Davranışı:** Sunucu, gelen bu karmaşık veri yapısını (stream) çözerken (decoding), verinin içeriğini doğrulamadan doğrudan bellekteki JavaScript nesnelerine dönüştürür.
*   **Neden:** Temel kök neden, **Güvensiz Deserialization (Insecure Deserialization)** ve **Input Validation (Girdi Doğrulama)** eksikliğidir. Sunucu, saldırganın gönderdiği manipüle edilmiş nesne yapılarını (örneğin kendine referans veren döngüler veya sahte metodlar) güvenli varsayarak işler. Bu durum, JavaScript'in dinamik yapısı (Duck Typing) sayesinde, sıradan bir veri işleme sürecinin, sunucuda komut çalıştırabilen bir kod akışına dönüşmesine neden olur.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.

*   **1. Hedef:** Next.js App Router üzerinde çalışan ve internete açık bir "Server Action" veya standart bir sayfa endpoint'i.
*   **2. Normal istek:** İstemci, sunucuya form verisi veya bir bileşen durumu gönderir. Bu genellikle arka planda React Flight formatında kodlanmış bir HTTP POST isteğidir.
*   **3. Manipüle edilmiş istek (PoC payload):** Saldırgan, standart Flight verisi yerine, içinde kötü niyetli JavaScript nesne yapıları barındıran özel bir payload gönderir. Bu payload, sunucudaki deserilizasyon motorunu yanıltmak için "self-reference" (kendine referans) döngüleri ve `then()` veya `toJSON()` gibi tetikleyici metodlar içerir.
*   **4. Analiz:** Sunucu gelen veriyi ayrıştırmaya (parse) başlar. Manipüle edilmiş nesne, ayrıştırma sürecinin bir parçası olarak `node:child_process` gibi kritik Node.js modüllerini belleğe yükler ve çalıştırır.
*   **5. Kanıt:** Saldırı başarılı olduğunda, sunucu üzerinde saldırganın belirlediği komutlar (örneğin `id`, `whoami` veya ters bağlantı açan bir shell komutu) çalıştırılır. Loglarda beklenmedik `sh` veya `cmd.exe` süreçlerinin başladığı görülür.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** **Kritik (10.0/10.0)**
*   **Saldırı Yüzeyi:** **İnternete Açık (Network) - Kimlik Doğrulama Gerekmez (Unauthenticated).** Zafiyet varsayılan konfigürasyonda bile tetiklenebilir.
*   **Karmaşıklık:** **Düşük.** Özel bir yarış durumu (race condition) veya karmaşık ön hazırlık gerektirmez; tek bir istek yeterlidir.

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):**
    *   **WAF Kuralları:** AWS WAF veya Cloudflare üzerinde, React Flight protokolüne özgü şüpheli desenleri (örneğin aşırı uzun JSON yapıları, `$X` içeren karmaşık patternler) engelleyen kurallar devreye alınmalıdır.
    *   **Secret Rotation:** Sistem ele geçirilmiş olabileceği varsayımıyla, tüm API anahtarları, veritabanı şifreleri ve environment değişkenleri (.env) derhal değiştirilmelidir.
*   **Orta vadeli:**
    *   **Yama Yönetimi:** React ve Next.js kütüphaneleri "Minimum Güvenli Sürüm"e yükseltilmelidir.
        *   React: **19.0.1+**, **19.1.2+**, **19.2.1+**
        *   Next.js: **15.0.5+**, **15.1.9+**, **16.0.7+**
    *   **Otomatik Düzeltme:** Next.js kullanıcıları `npx fix-react2shell-next` aracını kullanarak projelerini otomatik olarak güncelleyebilirler.
*   **Uzun vadeli:**
    *   **CI/CD Güvenlik Testleri:** Bağımlılık taramaları (SCA) derleme sürecine entegre edilmeli ve kritik zafiyet içeren paketlerin canlıya çıkması engellenmelidir.
    *   **Runtime Koruması:** Node.js süreçlerini izleyen ve şüpheli çocuk süreç (child process) oluşumlarını engelleyen RASP (Runtime Application Self-Protection) çözümleri değerlendirilmelidir.

---

### 6. Örnek düzeltme kodu (Node.js/Next.js)

Bu zafiyet kütüphane seviyesinde olduğu için, "Güvenli Kod" yazmaktan ziyade "Güvenli Bağımlılık Yönetimi" esastır. Geliştiricilerin `package.json` dosyasında yapması gereken değişiklik aşağıdadır:

**Güvensiz Yapılandırma (`package.json`):**
```json
{
  "dependencies": {
    "react": "19.0.0",       // KRİTİK ZAFİYETLİ
    "react-dom": "19.0.0",   // KRİTİK ZAFİYETLİ
    "next": "15.0.0"         // KRİTİK ZAFİYETLİ
  }
}
```

**Güvenli Yapılandırma (`package.json`):**
```json
{
  "dependencies": {
    "react": "^19.0.1",      // YAMALANMIŞ SÜRÜM
    "react-dom": "^19.0.1",  // YAMALANMIŞ SÜRÜM
    "next": "^15.0.5"        // YAMALANMIŞ SÜRÜM
  }
}
```
*Not: Sürüm numaralarını güncelledikten sonra `npm install` veya `yarn install` komutunu çalıştırarak `package-lock.json` veya `yarn.lock` dosyasını güncellemeyi unutmayın.*

---

### 7. Kontrol Listesi (Checklist)

**Hazırlık**
*   [ ] Projedeki mevcut React ve Next.js sürümlerini tespit et (`npm list react next`).
*   [ ] Uygulamanın internete açık olup olmadığını ve App Router kullanıp kullanmadığını doğrula.
*   [ ] Acil durum iletişim planını hazırla.

**Düzeltme**
*   [ ] `package.json` dosyasında React ve Next.js sürümlerini güvenli versiyonlara güncelle.
*   [ ] Bağımlılıkları yükle (`npm update` / `npm install`).
*   [ ] Eğer Next.js kullanılıyorsa, `npx fix-react2shell-next` aracını çalıştır.
*   [ ] Tüm environment değişkenlerini (Secrets) yenile (Database password, AWS keys vb.).

**Doğrulama**
*   [ ] `npm audit` komutu ile kritik zafiyet kalmadığını teyit et.
*   [ ] Uygulamayı test ortamında çalıştır ve temel fonksiyonların bozulmadığını (regresyon testi) doğrula.
*   [ ] Mümkünse WAF loglarını izleyerek saldırı girişimlerinin engellendiğini gözlemle.

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Loglarda aşağıdaki anahtar kelimeler ve desenler aranmalıdır:
    *   Node.js süreci altından çıkan şüpheli komutlar: `wget`, `curl`, `bash`, `sh`, `powershell`, `cmd.exe`.
    *   Hata loglarında anormal deserialization hataları veya "stack overflow" benzeri döngüsel hata mesajları.
    *   Dosya sistemi erişimleri: `/tmp`, `/var/tmp` veya `public` klasörlerine yapılan yazma istekleri.
*   **Anomali Tespiti:**
    *   Normal trafik akışının dışında, tek bir IP'den gelen ve sunucuda yüksek CPU kullanımına (Crypto mining şüphesi) neden olan istekler.
    *   Uygulamanın bilinen API endpoint'leri dışında, garip path'lere veya portlara yapılan bağlantı denemeleri.

---

### 9. Kaynakça

1. Critical Security Vulnerability in React Server Components, erişim tarihi Aralık 20, 2025, [https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components](https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components)
2. China-nexus cyber threat groups rapidly exploit React2Shell vulnerability (CVE-2025-55182) | AWS Security Blog, erişim tarihi Aralık 20, 2025, [https://aws.amazon.com/blogs/security/china-nexus-cyber-threat-groups-rapidly-exploit-react2shell-vulnerability-cve-2025-55182/](https://aws.amazon.com/blogs/security/china-nexus-cyber-threat-groups-rapidly-exploit-react2shell-vulnerability-cve-2025-55182/)
3. Critical Security Alert: Unauthenticated RCE in React & Next.js \- Upwind, erişim tarihi Aralık 20, 2025, [https://www.upwind.io/feed/critical-security-alert-unauthenticated-rce-in-react-next-js-cve-2025-55182-cve-2025-66478](https://www.upwind.io/feed/critical-security-alert-unauthenticated-rce-in-react-next-js-cve-2025-55182-cve-2025-66478)
4. React2Shell (CVE-2025-55182): Critical React Vulnerability | Wiz Blog, erişim tarihi Aralık 20, 2025, [https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182](https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182)
5. Defending against the CVE-2025-55182 (React2Shell) vulnerability ..., erişim tarihi Aralık 20, 2025, [https://www.microsoft.com/en-us/security/blog/2025/12/15/defending-against-the-cve-2025-55182-react2shell-vulnerability-in-react-server-components/](https://www.microsoft.com/en-us/security/blog/2025/12/15/defending-against-the-cve-2025-55182-react2shell-vulnerability-in-react-server-components/)
6. Multiple Threat Actors Exploit React2Shell (CVE-2025-55182) | Google Cloud Blog, erişim tarihi Aralık 20, 2025, [https://cloud.google.com/blog/topics/threat-intelligence/threat-actors-exploit-react2shell-cve-2025-55182](https://cloud.google.com/blog/topics/threat-intelligence/threat-actors-exploit-react2shell-cve-2025-55182)
7. CVE-2025-55182: React2Shell Analysis, Proof-of-Concept Chaos ..., erişim tarihi Aralık 20, 2025, [https://www.trendmicro.com/en\_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html](https://www.trendmicro.com/en_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html)
8. Blog \- Node.js, erişim tarihi Aralık 20, 2025, [https://nodejs.org/en/blog/year-2025](https://nodejs.org/en/blog/year-2025)
9. Wednesday, January 7, 2026 Security Releases \- Node.js, erişim tarihi Aralık 20, 2025, [https://nodejs.org/en/blog/vulnerability/december-2025-security-releases](https://nodejs.org/en/blog/vulnerability/december-2025-security-releases)
10. Security Advisory: CVE-2025-66478 \- Next.js, erişim tarihi Aralık 20, 2025, [https://nextjs.org/blog/CVE-2025-66478](https://nextjs.org/blog/CVE-2025-66478)
11. Exploitation of Critical Vulnerability in React Server Components (Updated December 12), erişim tarihi Aralık 20, 2025, [https://unit42.paloaltonetworks.com/cve-2025-55182-react-and-cve-2025-66478-next/](https://unit42.paloaltonetworks.com/cve-2025-55182-react-and-cve-2025-66478-next/)
12. Tuesday, July 15, 2025 Security Releases \- Node.js, erişim tarihi Aralık 20, 2025, [https://nodejs.org/en/blog/vulnerability/july-2025-security-releases](https://nodejs.org/en/blog/vulnerability/july-2025-security-releases)
13. Next.js Security Update: December 11, 2025, erişim tarihi Aralık 20, 2025, [https://nextjs.org/blog/security-update-2025-12-11](https://nextjs.org/blog/security-update-2025-12-11)
14. Ulusal Siber Olaylara Müdahale Merkezi \- USOM, erişim tarihi Aralık 20, 2025, [https://www.usom.gov.tr/bildirim/tr-25-0428](https://www.usom.gov.tr/bildirim/tr-25-0428)
15. STM, 2025'in İlk Siber Raporunu Yayımladı, erişim tarihi Aralık 20, 2025, [https://www.stm.com.tr/tr/basin-bultenleri/stm-2025-ilk-siber-raporunu-yayimladi](https://www.stm.com.tr/tr/basin-bultenleri/stm-2025-ilk-siber-raporunu-yayimladi)
16. STM, Siber Tehdit Durum Raporu'nu yayımladı \- Anadolu Ajansı, erişim tarihi Aralık 20, 2025, [https://www.aa.com.tr/tr/isdunyasi/bilisim/stm-siber-tehdit-durum-raporunu-yayimladi/685288](https://www.aa.com.tr/tr/isdunyasi/bilisim/stm-siber-tehdit-durum-raporunu-yayimladi/685288)
