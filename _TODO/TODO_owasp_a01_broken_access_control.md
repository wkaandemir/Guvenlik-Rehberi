# TODO: OWASP A01:2025 — Kirik Erisim Kontrolu

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | OWASP A01:2025 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Erisim_Kontrolu/ |
| **Kaynak Arastirma** | [OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf](../Arastirmalar/OWASP_Top_10_2025_Kapsamli_Guvenlik_Analizi_Raporu.pdf) |
| **Tarih** | 2026-03-17 |
| **Mevcut Dokuman** | [A01_Broken_Access_Control.md](../OWASP_Top_10/A01_Broken_Access_Control.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Kirik Erisim Kontrolu, uygulamanin yetkilendirme mekanizmalarinin yetersiz veya hatali olmasi nedeniyle yetkisiz kullanicilarin kisitli kaynaklara, fonksiyonlara veya verilere erisebildigi bir guvenlik acigdir. OWASP Top 10:2025'de en kritik risk olarak siniflandirilmistir ve 2021 surumunden bu yana birinci siradaki yerini korumaktadir.
*   **Etkilenen bilesenler:** API endpoint'leri, web uygulamasi controller'lari, veritabani erisim katmani, middleware bilesenleri, oturum yonetimi sistemleri, mikro servisler arasi iletisim katmanlari

---

### 2. Teknik detay (nasil calisiyor)
*   Erisim kontrolu, kimlik dogrulamayi gecen kullanicilarin hangi kaynaklara ve islemlere erisebilecegini belirleyen yetkilendirme mekanizmalaridir.
*   Yaygın ortaya cikis bicimleri:
    *   Sunucu tarafinda yetkilendirme kontrollerinin eksik olmasi
    *   Istemci tarafinda yapilan erisim kontrollerine guvenilmesi
    *   Parametre manipulasyonu ile ID'lerin degistirilmesinin engellenmemesi (IDOR)
    *   Rol tabanli erisim kontrollerinin (RBAC) duzgun uygulanmamasi
    *   API'lerde uygun yetkilendirme mekanizmalarinin olmamasi
*   **Neden:** Yetkilendirme mantiginin uygulama mimarisine tam entegre edilmemesi, guvenlik denetimlerinin tutarsiz uygulanmasi ve guvenli varsayimlarla kod gelistirilmesi.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Kullanici profili bilgilerini getiren `/api/users/{id}` endpoint'i
*   **2. Normal durum:**
    ```
    GET /api/users/123 HTTP/1.1
    Authorization: Bearer <user_123_token>
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    GET /api/users/1 HTTP/1.1
    Authorization: Bearer <user_123_token>
    ```
*   **4. Analiz:** Saldirgan, kendi kullanici ID'si olan 123 yerine yonetici ID'si olan 1'i kullanarak yonetici bilgilerine erismeye calisir. Sunucu tarafinda uygun yetkilendirme kontrolu yapilmadigi icin istek basarili olur.
*   **5. Kanit:** Yanit olarak yoneticinin hassas bilgileri (e-posta, telefon, rol vb.) doner.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** N/A (kategori bazli risk)
*   **Saldiri Yuzeyi:** Internete acik, Ic ag, Yetkili kullanici
*   **Karmasiklik:** Dusuk

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** WAF kurali ile supheli parametre manipulasyonlarini engelleme, kritik endpoint'lere ek yetkilendirme katmanlari ekleme.
*   **Orta vadeli:** Tum endpoint'ler icin yetkilendirme middleware'i gelistirme, parametre tabanli erisim kontrollerini guclendirme, API guvenlik testleri yapma.
*   **Uzun vadeli:** Guvenli kod gelistirme pratikleri benimseme, yetkilendirme mimarisini yeniden tasarlama, duzenli guvenlik kod incelemeleri yapma, gelistiricilere guvenlik egitimi verme.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```javascript
// Sadece kimlik dogrulama, yetkilendirme kontrolu yok
app.get('/api/users/:id', authenticateToken, (req, res) => {
  const userId = req.params.id;
  const user = getUserById(userId);
  res.json(user);
});
```

**Guvenli kod:**
```javascript
app.get('/api/users/:id', authenticateToken, authorizeUserAccess, (req, res) => {
  const userId = req.params.id;
  const user = getUserById(userId);
  res.json(user);
});

function authorizeUserAccess(req, res, next) {
  const requestedUserId = req.params.id;
  const currentUserId = req.user.id;
  const currentUserRole = req.user.role;
  if (requestedUserId === currentUserId || currentUserRole === 'admin') {
    next();
  } else {
    res.status(403).json({ error: 'Erisim reddedildi' });
  }
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Tum endpoint'lerin erisim kontrol gerektirip gerektirmedigini belirle
*   [ ] Mevcut yetkilendirme mekanizmalarini haritala
*   [ ] Erisim kontrolu test senaryolarini olustur

**Duzeltme**
*   [ ] Sunucu tarafinda yetkilendirme kontrollerini uygula
*   [ ] Istemci tarafi kontrollerine guvenme
*   [ ] Rol tabanli erisim kontrolu (RBAC) mekanizmasi kur
*   [ ] API'ler icin uygun yetkilendirme katmanlari ekle

**Dogrulama**
*   [ ] Yetkisiz erisim denemelerini test et
*   [ ] Otomasyon testleri ile erisim kontrollerini dogrula
*   [ ] Penetrasyon testi yaptir

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Erisim reddedilen isteklerdeki artislari izle
    *   Ayni kullanici tarafindan farkli kullanici ID'lerine yapilan erisim denemelerini tespit et
    *   Regex: `(?i)(access denied|unauthorized|forbidden).*user.*\d+`
*   **Anomali Tespiti:**
    *   Kullanici rolu disinda kaynaklara erisim denemeleri
    *   Kisa surede cok sayida farkli kullanici ID'sine erisim denemeleri
    *   Normal calisma saatleri disinda yonetici fonksiyonlarina erisim denemeleri

---

## Notlar
- OWASP Top 10:2025'de 1. sirada yer alir (2021'den bu yana degismedi).
- Detayli rehber: [A01_Broken_Access_Control.md](../OWASP_Top_10/A01_Broken_Access_Control.md)
