### Güvenlik Açığı: CVE-2026-1529 — Keycloak Organizations davet jetonlarında hatalı imza doğrulaması ile yetkisiz organizasyon kaydı (CWE-347)

### 1. Özet ve etki
*   **Yönetici Özeti:** CVE-2026-1529, Keycloak Organizations (organizasyon/tenant) kayıt ve davet akışında kullanılan “action token” (JWT benzeri) doğrulamasındaki mantık hatası nedeniyle, düşük ayrıcalıklı bir kullanıcının organizasyon sınırlarını aşarak yetkisiz bir organizasyona üye olabilmesine yol açabilir. Bu durum çok kiracılı (multi-tenant) SaaS senaryolarında veri sızıntısı (gizlilik) ve yetkisiz değişiklik (bütünlük) riski doğurur.
*   **Etkilenen bileşenler:** (Mevcut dokümandaki araştırma notlarına göre)
    *   Keycloak **Organizations** modülü (organizasyon daveti/kabul akışı)
    *   `org.keycloak.services.resources.organizations` bileşeni
    *   Organizasyon daveti kabul/kayıt işleyişi (ör. `OrganizationInvitationResource` gibi kaynaklar)
    *   “Action Token” / JWT doğrulama ve claim (ör. organizasyon ID, e-posta) işleme katmanı
    *   Uygulama tarafında “organizasyon üyeliğine göre yetkilendirme” yapan servisler (dolaylı etki)

---

### 2. Teknik detay (nasıl çalışıyor)
*   Keycloak, organizasyon davetlerini “Action Tokens” yaklaşımıyla (JWT benzeri, imzalı jeton) taşıyabilir; bu modelde davet bilgileri (organizasyon kimliği, hedef e-posta vb.) jetonun içinde bulunur.
*   Zafiyet senaryosunda sorun, jetonun **kriptografik imzasının doğrulanması ve/veya imza doğrulamasının sonucu ile jeton içindeki kritik alanların güvenle ilişkilendirilmesi** aşamasında ortaya çıkar.
*   Saldırgan, meşru bir davet jetonuna erişim sağladıktan sonra, sistemin beklediği şekilde doğrulanmayan alanları (ör. organizasyon ID, e-posta) hedef organizasyona yönlendirecek biçimde “uygunsuz/uyumsuz” hale getirerek kabul akışını tetikleyebilir.
*   Özellikle “stateless davet” yaklaşımında, sunucu tarafında davetin kanonik bir kaydı bulunmadığında; kabul anında jetondaki claim’lerin **sunucu tarafındaki bir referans kayıtla** karşılaştırılması mümkün değildir. Bu da doğrulama zincirini zayıflatır.
*   **Neden (kök neden):** CWE-347 (Kriptografik İmzanın Hatalı Doğrulanması) sınıfında; imza doğrulama/claim bağlama adımlarında eksiklik ve (önceki tasarımda) davet bilgisinin sunucu tarafında kalıcı olarak tutulmaması nedeniyle, organizasyon üyeliği gibi yetki açısından kritik bir kararın yalnızca jeton verisine dayanması.

---

### 3. Adım adım istismar (PoC — kavramsal)
> **Uyarı:** Bu bölüm yalnızca eğitim ve savunma amaçlıdır. Aşağıdaki adımlar kasıtlı olarak kavramsal bırakılmıştır; üretim sistemlerinde veya yetkisiz ortamlarda denenmemelidir.
*   **1. Hedef:** Keycloak Organizations “davet kabul / organizasyona katılım” akışı (Action Token/JWT ile).
*   **2. Normal istek:** Kullanıcı, kendisine gönderilen organizasyon davet linkini açar veya UI üzerinden daveti kabul eder; istemci, davet jetonunu sunucuya iletir.
*   **3. Manipüle edilmiş istek (PoC payload):** Saldırgan, ele geçirdiği/geçerli bir davet jetonunun içinde organizasyon üyeliğini belirleyen alanları (ör. `org_id`, `email`) hedef organizasyona yönlendirecek şekilde “uyumsuz” hale getirerek aynı kabul akışını çağırır. (Jetonun gerçek formatı/alan adları sürüme göre değişebilir.)
*   **4. Analiz:** Sunucu tarafında imza doğrulama/claim bağlama hatası nedeniyle, davetin gerçekten hangi organizasyon için çıkarıldığı yeterince doğrulanmadan üyelik kaydı oluşturulabilir.
*   **5. Kanıt:** Saldırganın, yetkisi olmayan organizasyon altında kullanıcı listesinde görünmesi; audit log’larda “invite accepted / organization membership created” benzeri kayıtlar; uygulama içinde hedef organizasyon kaynaklarına erişimin açılması.

---

### 4. Risk değerlendirmesi
*   **Kritiklik:** Yüksek (araştırma notlarında CVSS v3.x: **8.1**, vektör: `AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N`).
*   **Saldırı Yüzeyi:** Ağ üzerinden; genellikle internete açık IAM katmanı. İstismar için çoğunlukla düşük ayrıcalıklı hesap (PR:L) ve geçerli bir davet jetonuna erişim önkoşulu bulunur.
*   **Karmaşıklık:** Düşük (AC:L); kullanıcı etkileşimi gerektirmeyebilir (UI:N) ve organizasyon sınırı aşımı doğrudan yetki ihlali üretir.

---

### 5. Kalıcı çözümler ve öneriler
*   **Kısa vadeli (Acil):**
    *   Organizations özelliği kritik değilse geçici olarak **devre dışı bırakın** veya davet/kabul akışını kısıtlayın.
    *   Organizasyona katılımı “self-registration” yerine yönetici onayına bağlayın; davet kullanımını tek seferlik ve kısa süreli hale getirin.
    *   Keycloak’ı internete açık çalıştırmak zorundaysanız, davet/kayıt uç noktalarına ek ağ politikaları (reverse proxy allowlist, rate limit, IP kısıtları) uygulayın.
*   **Orta vadeli:**
    *   Üretici yamalarına yükseltin (notlarda geçen öneriler: **Keycloak 26.2.13**, **26.4.9** veya **26.5.0+**).
    *   Red Hat Build of Keycloak (RHBK) kullanılıyorsa ilgili errata/imaj güncellemelerini uygulayın (notlarda: **RHSA-2026:2364**, **RHSA-2026:2366**; OpenShift bileşenleri dahil).
    *   Davet doğrulamasında jeton içeriğini, sunucu tarafında tutulan kalıcı davet kaydıyla (ör. “persistent action entity” yaklaşımı) **eşleştirin**; claim uyumsuzluğunu reddedin; replay koruması ekleyin.
*   **Uzun vadeli:**
    *   “Preview” özellikleri (Organizations gibi) üretime almadan önce threat modeling ve güvenlik test kapıları ekleyin.
    *   Organizasyon üyeliğini yetkilendirme kararlarında kullanan uygulamalar için “cross-tenant” erişim testleri (negatif testler) ekleyin.
    *   Jeton doğrulama kodu için birim/entegrasyon testleri: `issuer/audience`, `exp/nbf`, `jti` tek-seferlik kullanım, claim-bütünlük kontrolleri.

---

### 6. Örnek düzeltme kodu ([Java/Keycloak])
*   Aşağıdaki örnek, organizasyon daveti kabul ederken yalnızca jeton içeriğine güvenmek yerine, sunucu tarafındaki kalıcı davet kaydıyla eşleştirme yaklaşımını göstermektedir (kavramsal/pseudo-code).

```java
// Guvensiz yaklasim (kavramsal): Token claim'lerine dogrudan guvenmek
InvitationToken token = tokenService.parseAndVerify(rawToken);
organizationService.addMember(token.getOrganizationId(), token.getEmail());
```

```java
// Guvenli yaklasim (kavramsal): Imza + sunucu-tarafi kayit eslestirme + tek-seferlik kullanim
InvitationToken token = tokenService.parseAndStrictVerify(
    rawToken,
    expectedIssuer,
    expectedAudience
);

// Token'in benzersiz kimligini (or. jti) referans alarak daveti DB'den getir.
OrganizationInvite invite = inviteRepository.findById(token.getTokenId())
    .orElseThrow(() -> new BadRequestException("Gecersiz veya iptal edilmis davet"));

// Jetondaki kritik alanlari sunucu tarafindaki davet kaydiyla eslestir.
if (!invite.getOrganizationId().equals(token.getOrganizationId())) {
    throw new BadRequestException("Davet organizasyon bilgisi uyusmuyor");
}
if (!invite.getEmail().equalsIgnoreCase(token.getEmail())) {
    throw new BadRequestException("Davet e-posta bilgisi uyusmuyor");
}

// Replay/yeniden-kullanim ve sure kontrolleri.
if (invite.isExpired() || invite.isUsed()) {
    throw new BadRequestException("Davet suresi dolmus veya daha once kullanilmis");
}

invite.markUsed(Instant.now());
inviteRepository.save(invite);

organizationService.addMember(invite.getOrganizationId(), invite.getEmail());
```

---

### 7. Kontrol Listesi (Checklist)
**Hazırlık**
*   [ ] Keycloak sürümünü ve Organizations özelliğinin aktif olup olmadığını envanterleyin.
*   [ ] Organizasyon davet/kabul akışının hangi uygulamalarda yetkilendirme için kullanıldığını belirleyin (tenant sınırı).
*   [ ] Audit log ve erişim loglarının tutulduğunu doğrulayın (kullanıcı/organizasyon üyelik değişimleri).
**Düzeltme**
*   [ ] Üretici yamalarına yükseltin (notlara göre: 26.2.13 / 26.4.9 / 26.5.0+ veya vendor errata).
*   [ ] Organizations özelliği zorunlu değilse geçici olarak kapatın veya davet kabulünü kısıtlayın.
*   [ ] Davet kabulünde token claim’lerini sunucu tarafı kayıtla eşleştirecek kontrolleri doğrulayın (jti, orgId, email, expiry, one-time use).
**Doğrulama**
*   [ ] Test ortamında, “uyumsuz claim” içeren davetlerin reddedildiğini doğrulayın.
*   [ ] Organizasyon üyeliği değişimlerinin audit log’lara düştüğünü ve SIEM’de uyarı üretilebildiğini test edin.

---

### 8. İzleme ve uyarılar
*   **SIEM/Log Kuralları:** Aşağıdaki türde olayları korele edecek kurallar tanımlayın:
    *   Kısa sürede çok sayıda “davet kabul / organizasyona katılım” olayı (brute-force/replay sinyali).
    *   Davet kabul denemelerinde “geçersiz token”, “claim mismatch”, “signature verification failed” benzeri hata oranında ani artış.
    *   Beklenmeyen organizasyon üyeliği oluşturma (özellikle yeni/az kullanılan organizasyonlara ani katılımlar).
    *   Örnek anahtar kelimeler (ortama göre uyarlayın): `organization invite`, `invitation accepted`, `action token`, `org`, `membership`, `claim mismatch`.
*   **Anomali Tespiti:** Şüpheli kabul edilebilecek davranış örnekleri:
    *   Kullanıcının geçmişte ilişkisiz olduğu bir organizasyona aniden üye olması.
    *   Farklı organizasyonlara ardışık katılım denemeleri (yataya hareket sinyali).
    *   Davet kabul akışının alışılmadık IP/ASN’lerden veya beklenmeyen saatlerde tetiklenmesi.

Referanslar (dokümandaki araştırma notlarından):
1. Keycloak Docs – Securing Applications and Services Guide (erişim: 2026-02-10): https://www.keycloak.org/docs/25.0.6/securing_apps/index.html
2. GitLab Advisory Database – `org.keycloak:keycloak-services` (erişim: 2026-02-10): https://advisories.gitlab.com/pkg/maven/org.keycloak/keycloak-services/
3. xata.io – “How we improved Organization Invites to Keycloak” (erişim: 2026-02-10): https://xata.io/blog/how-we-improved-organization-invites-to-keycloak
4. Phase Two – “Decoding JWT Structure” (erişim: 2026-02-10): https://phasetwo.io/articles/jwts/decoding-jwt-structure/
5. Red Hat Customer Portal – CVE-2026-1529 (erişim: 2026-02-10): https://access.redhat.com/security/cve/CVE-2026-1529
6. Tenable – CVE-2026-1529 (erişim: 2026-02-10): https://www.tenable.com/cve/CVE-2026-1529
7. GitHub – `keycloak/keycloak` Releases (erişim: 2026-02-10): https://github.com/keycloak/keycloak/releases
8. Red Hat Errata – RHSA-2026:2364 (erişim: 2026-02-10): https://access.redhat.com/errata/RHSA-2026:2364
9. Red Hat Errata – RHSA-2026:2366 (erişim: 2026-02-10): https://access.redhat.com/errata/RHSA-2026:2366
10. Red Hat Errata – RHSA-2026:2365 (erişim: 2026-02-10): https://access.redhat.com/errata/RHSA-2026:2365
11. GitHub Issue – “invite-existing-user API should reject invitations for users already in the organization” (erişim: 2026-02-10): https://github.com/keycloak/keycloak/issues/45553
12. Release Notes – Red Hat build of Keycloak 26.2 (erişim: 2026-02-10): https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/26.2/html/release_notes/index
