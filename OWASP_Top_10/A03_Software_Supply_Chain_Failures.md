### Güvenlik Açığı: Yazılım Tedarik Zinciri Hataları (Software Supply Chain Failures)

### 1. Özet ve etki
*   **Yönetici Özeti:** Yazılım Tedarik Zinciri Hataları, uygulama geliştirme sürecinde kullanılan dış bağımlılıkların, kütüphanelerin, framework'lerin veya altyapı bileşenlerinin güvenlik açıkları içermesi veya bu bileşenlerin yetkisiz kişilerce değiştirilmesi nedeniyle ortaya çıkan bir güvenlik riskidir. OWASP Top 10:2025'te 3. sırada yer alan bu kategori, 2021'deki "Vulnerable and Outdated Components" kategorisinin yerini almış ve daha geniş bir kapsama sahiptir. Bu zafiyet, Shai-Hulud solucanı ve tj-actions bulaşması gibi büyük ölçekli saldırılara neden olmuştur.
*   **Etkilenen bileşenler:** Package manager'lar (npm, PyPI, Maven), CI/CD pipeline'ları, container imajları, third-party kütüphaneler, open source bileşenler, derleme araçları, kod depoları

---

### 2. Teknik detay (nasıl çalışıyor)
*   Yazılım tedarik zinciri, kodun geliştirilmesinden dağıtımına kadar olan tüm süreci kapsar. Bu süreçteki herhangi bir zayıflık, tüm uygulamayı riske atabilir.
*   Yaygın tedarik zinciri zafiyetleri:
    *   Güvenlik açığı içeren bağımlılıkların kullanılması
    *   İmzasız veya doğrulanmamış paketlerin kullanılması
    *   CI/CD pipeline'larındaki güvenlik açıkları
    *   Container imajlarındaki zayıflıklar
    *   Malicious kod içeren kütüphanelerin entegrasyonu
    *   Geliştirici hesaplarının ele geçirilmesi
*   **Neden:** Kök neden olarak bağımlılık yönetimi süreçlerinin eksikliği, bileşenlerin güvenlik denetimlerinin yapılmaması, dijital imza ve doğrulama mekanizmalarının kullanılmaması ve tedarik zinciri güvenliğine yeterince önem verilmemesi sayılabilir.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır.
*   **1. Hedef:** Popüler bir open source kütüphaneye yapılan malicious kod enjeksiyonu
*   **2. Normal durum:** 
    ```
    // package.json
    {
      "dependencies": {
        "popular-library": "1.2.3"
      }
    }
    ```
*   **3. Manipüle edilmiş durum (PoC payload):**
    ```
    // Saldırgan, kütüphanenin yeni sürümüne malicious kod ekler
    // package.json
    {
      "dependencies": {
        "popular-library": "1.2.4"  // Malicious sürüm
      }
    }
    ```
*   **4. Analiz:** Saldırgan, popüler bir kütüphanenin yeni sürümüne zararlı kod ekler. Geliştiriciler bu kütüphaneyi güncellediğinde, malicious kod uygulamaya entegre olur. Bu kod, hassas verileri çalmak veya yetkisiz erişim sağlamak için tasarlanmıştır.
*   **5. Kanıt:** Uygulama, normal çalışırken arka planda hassas verileri saldırganın sunucusuna gönderir. Loglarda anormal dış ağ bağlantıları görülür.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Kritik
*   **Saldırı Yüzeyi:** İnternete açık, İç ağ, Geliştirme ortamları
*   **Karmaşıklık:** Orta

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):** Bilinen güvenlik açığı içeren bağımlılıkları güncelleme, şüpheli paketleri kaldırma, imzasız paketleri kullanmaktan kaçınma
*   **Orta vadeli:** Bağımlılık tarama araçları kullanma, SBOM (Software Bill of Materials) oluşturma, dijital imza doğrulama mekanizmaları kurma
*   **Uzun vadeli:** Güvenli tedarik zinciri stratejisi geliştirme, CI/CD güvenliği sağlama, geliştiricilere güvenlik eğitimi verme, düzenli bağımlılık güvenlik denetimleri yapma

---

### 6. Örnek düzeltme kodu (Node.js/npm)

**Güvensiz Yaklaşım:**
```json
// package.json
{
  "name": "vulnerable-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "4.17.1",  // Eski sürüm, bilinen güvenlik açıkları var
    "lodash": "4.17.15",  // Eski sürüm, prototype pollution açığı var
    "request": "2.88.0"   // Bakımı durmuş kütüphane
  }
}
```

**Güvenli Yaklaşım:**
```json
// package.json
{
  "name": "secure-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2",  // Güvenli sürüm
    "lodash": "^4.17.21",  // Güvenlik açığı düzeltilmiş sürüm
    "axios": "^1.4.0"      // Bakımı devam eden alternatif
  },
  "scripts": {
    "audit": "npm audit",
    "audit-fix": "npm audit fix",
    "check-deps": "npm-check-updates"
  }
}
```

**Güvenlik Kontrolü Script'i:**
```javascript
// scripts/dependency-check.js
const { execSync } = require('child_process');
const fs = require('fs');

// Bağımlılık denetimi
try {
  console.log('Bağımlılık güvenlik denetimi yapılıyor...');
  const auditResult = execSync('npm audit --json', { encoding: 'utf8' });
  const auditData = JSON.parse(auditResult);
  
  if (auditData.vulnerabilities && Object.keys(auditData.vulnerabilities).length > 0) {
    console.error('Güvenlik açığı içeren bağımlılıklar bulundu:');
    console.error(JSON.stringify(auditData.vulnerabilities, null, 2));
    process.exit(1);
  }
  
  console.log('Bağımlılık güvenlik denetimi başarılı.');
} catch (error) {
  console.error('Bağımlılık denetimi sırasında hata:', error.message);
  process.exit(1);
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Tüm bağımlılıkları envanterle
*   [ ] SBOM (Software Bill of Materials) oluştur
*   [ ] Bağımlılık güvenlik denetim araçları kur

**Düzeltme**
*   [ ] Güvenlik açığı içeren bağımlılıkları güncelle
*   [ ] Bakımı durmuş kütüphaneleri alternatiflerle değiştir
*   [ ] Dijital imza doğrulama mekanizmaları kur
*   [ ] CI/CD pipeline'ını güvenli hale getir

**Doğrulama**
*   [ ] Otomatik bağımlılık taramaları yap
*   [ ] Container imajlarını güvenlik taramasından geçir
*   [ ] Düzenli güvenlik denetimleri yap
*   [ ] Bağımlılık güncelleme süreçleri oluştur

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** 
    *   Şüpheli paket kurulumlarını izle
    *   Anormal dış ağ bağlantılarını tespit et
    *   Regex: `(?i)(npm|pip|mvn|docker).*install|package.*install|pip install`
*   **Anomali Tespiti:** 
    *   Normal dışı paket güncelleme aktiviteleri
    *   Bilinmeyen kaynaklardan yapılan indirmeler
    *   CI/CD pipeline'larındaki yetkisiz değişiklikler
    *   Container imajlarındaki anormallikler
