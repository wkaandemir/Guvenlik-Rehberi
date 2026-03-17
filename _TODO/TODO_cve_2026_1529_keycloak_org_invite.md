# TODO: CVE-2026-1529 — Keycloak Organizations Davet Jetonu Bypass

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-1529 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | YENI: Kimlik_Dogrulama/ |
| **Kaynak Arastirma** | [Keycloak_Zafiyet_Arastirmasi_ve_Analizi.md](../Arastirmalar/Keycloak_Zafiyet_Arastirmasi_ve_Analizi.md) |
| **Tarih** | 2026-02-10 |
| **Mevcut Dokuman** | [Keycloak_Zafiyet_Arastirmasi_ve_Analizi.md](../Arastirmalar/Keycloak_Zafiyet_Arastirmasi_ve_Analizi.md) |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-1529, Keycloak kimlik ve erisim yonetimi platformunun Organizations modulunde bulunan bir davet jetonu (action token/JWT) dogrulama bypass zafiyetidir (CWE-347: Kriptografik Imzanin Hatali Dogrulanmasi). CVSS 8.1 ile derecelendirilen bu acik, dusuk ayricalikli bir kullanicinin organizasyon davet jetonlarindaki claim degerlerini manipule ederek, yetkisiz bir organizasyona uye olmasina olanak tanimaktadir. Cok kiracili (multi-tenant) SaaS senaryolarinda, bir kiraciinn verisinin baska bir kiraci tarafindan okunmasi veya degistirilmesi gibi ciddi sonuclara yol acabilir. Keycloak 26.2.13, 26.4.9 ve 26.5.0 surumlerinde yamali olup Red Hat Build of Keycloak icin de ilgili errataalar yayinlanmistir.
*   **Etkilenen bilesenler:** Keycloak Organizations modulu (preview/GA), Keycloak 26.2.12 ve oncesi, 26.4.8 ve oncesi surumler, Red Hat Build of Keycloak (RHBK) ilgili surumler, Organizations ozelligini kullanan tum Keycloak kurulumlari, cok kiracili SaaS uygulamalari ve organizasyon tabanli yetkilendirme sistemleri

---

### 2. Teknik detay (nasil calisiyor)
*   Keycloak Organizations modulu, kullanicilari organizasyonlara davet etmek icin "action token" adli JWT (JSON Web Token) tabanli davet jetonlari kullanir. Bu jetonlar, davet edilen kullanicinin e-posta adresi, hedef organizasyon kimliigi (orgId), jeton benzersiz kimligi (jti), son kullanma tarihi (exp) ve issuer/audience bilgilerini icerir.
*   Zafiyet, davet jetonundaki claim degerlerinin (ozellikle `orgId`) sunucu tarafinda yeterince dogrulanmamasindan kaynaklanir. Keycloak, jetonun kriptografik imzasini dogrular ancak jetonundaki `orgId` claim'inin, sunucu tarafindaki davet kaydı (invitation record) ile eslestilip eslestirilmedigini kontrol etmez.
*   Saldirgan, kendi organizasyonuna ait gecerli bir davet jetonu alir (veya kendisi olusturur), ardindan jeton icindeki `orgId` degerini hedef organizasyonun kimligi ile degistirir. Eger imza dogrulamasi sadece jetonun butunlugunu (integrity) degil, claim degerlerinin sunucu tarafindaki kayitlarla eslesmesini de kontrol etmiyorsa, saldirgan bu manipule edilmis jetonla hedef organizasyona uye olarak eklenir.
*   Ek olarak, "stateless davet" yaklasimi (jeton icindeki bilgilere tamamen guvenme) kullanildiginda, jeton replay saldiirilarina da acik olabilir: ayni jeton birden fazla kez kullanilabilir.
*   **Neden:** Kok neden, davet jetonu dogrulama surecinde yalnizca kriptografik imza butunlugune guvenilmesi ve jeton claim'lerinin sunucu tarafindaki yetkilendirme kayitlariyla (invitation database record) karsilastirilmamasidir. "Stateless" jeton yaklasiminin, "guvenilir ancak dogrulanmamis" claim degerlerine dayali is mantigi kararlarini mozumlemesi bu zafiyeti dogurmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Keycloak Organizations modulu etkin olan cok kiracili SaaS platformu, dusuk ayricalikli kullanici hesabi
*   **2. Normal durum:**
    ```
    1. Organizasyon A yoneticisi, kullanici@ornek.com adresine davet gonderir
    2. Keycloak, orgId=A, email=kullanici@ornek.com iceren bir JWT davet jetonu olusturur
    3. Kullanici, davet linkini tikladiginda jeton dogrulanir
    4. Keycloak, imza dogrulamasindan sonra kullaniciyi Organizasyon A'ya ekler
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```bash
    # 1. Saldirgan, kendi organizasyonuna (Org-X) ait bir davet jetonu alir
    INVITE_TOKEN="eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."

    # 2. Jetonu decode et (base64)
    echo $INVITE_TOKEN | cut -d. -f2 | base64 -d | jq .
    # Cikti: {"orgId":"org-x-id","email":"saldirgan@ornek.com","jti":"abc123",...}

    # 3. orgId claim'ini hedef organizasyona degistir
    # NOT: Imza dogrulama zayifsa veya claim kontrolu yoksa gecerli olur
    MODIFIED_PAYLOAD=$(echo '{"orgId":"hedef-org-id","email":"saldirgan@ornek.com","jti":"abc123","exp":9999999999}' | base64 -w0)
    MODIFIED_TOKEN="${HEADER}.${MODIFIED_PAYLOAD}.${SIGNATURE}"

    # 4. Manipule edilmis jetonla davet kabulunu gonder
    curl -X POST "https://keycloak.hedef.com/realms/master/protocol/openid-connect/token" \
      -H "Content-Type: application/x-www-form-urlencoded" \
      -d "grant_type=urn:ietf:params:oauth:grant-type:token-exchange" \
      -d "subject_token=$MODIFIED_TOKEN" \
      -d "subject_token_type=urn:ietf:params:oauth:token-type:jwt"

    # 5. Saldirgan, hedef organizasyona uye olarak eklenir
    # Hedef organizasyonun kullanicilarina, kaynaklarina ve verilerine erisim saglanir
    ```
*   **4. Analiz:** Keycloak'un davet kabul mekanizmasi, gelen jetonun kriptografik imzasini dogrular (gecerli anahtar ile imzalandigini kontrol eder). Ancak, jetonun `orgId` claim'ini sunucu tarafindaki davet kaydiyla eslestirilmemektedir. Saldirgan, kendi organizasyonuna ait gecerli bir jetonu alir, `orgId` degerini hedef organizasyonla degistirir — imza dogrulamasi claim degerlerinin butunlugunu degil, yalnizca jetonun butunlugunu kontrol ettigi icin bu degisiklik tespit edilmez. "Stateless" davet yaklasiminda jeton icindeki bilgilere dogrudan guvenildigi icin sunucu tarafinda karsilastirma yapilmaz.
*   **5. Kanit:** Saldirgan, yetkisiz olarak hedef organizasyona uye olur. Hedef organizasyonun kullanicilari, kaynaklari ve yapilandirmasi saldirganin erisim alanina girer. Audit loglarinda, saldirganin hesabinin beklenmeyen bir organizasyona katildigi kaydedilir. Cok kiracili senaryoda, bir kiracinin verileri baska bir kiraci tarafindan okunabilir veya degistirilebilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 8.1 (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N)
*   **Saldiri Yuzeyi:** Internete acik (Keycloak web arayuzu ve API — dusuk ayricalikli kullanici hesabi yeterli)
*   **Karmasiklik:** Dusuk (JWT manipulasyonu standart araclarla yapilabilir, ozel bilgi gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Keycloak'u yamali surume yukselt: 26.2.13, 26.4.9 veya 26.5.0+. Red Hat Build of Keycloak kullaniliyorsa RHSA-2026:2364 ve RHSA-2026:2366 errata guncellemelerini uygulayin. Organizations ozelligi zorunlu degilse gecici olarak kapatin veya davet kabulunu yonetici onayina baglayin. Mevcut organizasyon uyeliklerini gozden gecirin ve yetkisiz uyelikleri arastirin. Davet/kayit uc noktalarina rate limit ve IP kisitlamalari ekleyin.
*   **Orta vadeli:** Davet kabulunde jeton claim'lerini sunucu tarafindaki davet kaydiyla eslesiren dogrulama katmani ekleyin (jti, orgId, email, expiry, one-time use). Organizasyona katilimi yonetici onayina baglayin. Davet jetonlarini tek seferlik (one-time use) ve kisa sureli (ornegin 24 saat) yapin. Audit loglarinda organizasyon uyeligi degisimlerini SIEM'e aktarin ve uyari kurali tanimlayin. Cross-tenant erisim testlerini duzenli olarak gerceklestirin.
*   **Uzun vadeli:** Keycloak Organizations modulunu "preview" asamasindaki ozelliklerin uretimine alinmadan once threat modeling yapilmasini zorunlu kilan politika ile yapilandirlarin. Jeton tabanli yetkilendirme mekanizmalarinda "stateless" yaklasim yerine, sunucu tarafinda kayit eslestirme zorunlulugu getirilsin. Keycloak guvenlik bultenleri icin otomatik izleme ve yamala sureci olusturun. Kimlik ve erisim yonetimi platformunun duzenli guvenlik denetimini (penetrasyon testi) planlayin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```java
// === GUVENSIZ: Token claim'lerine dogrudan guvenmek ===
public void acceptInvitation(String rawToken) {
    // Yalnizca kriptografik imza dogrulanir
    InvitationToken token = tokenService.parseAndVerify(rawToken);

    // Token claim'lerine dogrudan guvenilerek uyelik eklenir
    // orgId sunucu kaydiyla eslestirilMEZ — manipulasyona acik!
    organizationService.addMember(
        token.getOrganizationId(),  // Manipule edilmis olabilir!
        token.getEmail()
    );
}
```

**Guvenli kod:**
```java
// === GUVENLI: Imza + sunucu-tarafi kayit eslestirme + tek-seferlik kullanim ===
public void acceptInvitation(String rawToken) {
    // 1. Kati kriptografik dogrulama (issuer, audience, algoritma kontrolu)
    InvitationToken token = tokenService.parseAndStrictVerify(
        rawToken, expectedIssuer, expectedAudience
    );

    // 2. Token'in benzersiz kimligini referans alarak daveti DB'den getir
    OrganizationInvite invite = inviteRepository.findById(token.getTokenId())
        .orElseThrow(() -> new BadRequestException("Gecersiz veya iptal edilmis davet"));

    // 3. Jetondaki kritik alanlari sunucu kaydıyla eslestir
    if (!invite.getOrganizationId().equals(token.getOrganizationId())) {
        auditLogger.warn("Organizasyon uyusmazligi: token={}, db={}",
            token.getOrganizationId(), invite.getOrganizationId());
        throw new BadRequestException("Davet organizasyon bilgisi uyusmuyor");
    }
    if (!invite.getEmail().equalsIgnoreCase(token.getEmail())) {
        auditLogger.warn("E-posta uyusmazligi: token={}, db={}",
            token.getEmail(), invite.getEmail());
        throw new BadRequestException("Davet e-posta bilgisi uyusmuyor");
    }

    // 4. Replay/yeniden-kullanim ve sure kontrolleri
    if (invite.isExpired()) {
        throw new BadRequestException("Davet suresi dolmus");
    }
    if (invite.isUsed()) {
        auditLogger.warn("Davet yeniden kullanim denemesi: jti={}", token.getTokenId());
        throw new BadRequestException("Davet daha once kullanilmis");
    }

    // 5. Tek seferlik kullanim — daveti kullanilmis olarak isaretle
    invite.markUsed(Instant.now());
    inviteRepository.save(invite);

    // 6. Dogrulanmis bilgilerle uyelik ekle
    organizationService.addMember(invite.getOrganizationId(), invite.getEmail());
    auditLogger.info("Organizasyon uyeligi eklendi: org={}, email={}",
        invite.getOrganizationId(), invite.getEmail());
}
```

```bash
# Keycloak yamali surume guncelleme
# Minimum guvenli surumler: 26.2.13 / 26.4.9 / 26.5.0+
bin/kc.sh show-config | grep version

# Keycloak'u guncelle ve yeniden derle
bin/kc.sh build --db=postgres
bin/kc.sh start --optimized

# Red Hat Build of Keycloak guncelleme:
# RHSA-2026:2364, RHSA-2026:2366 erratalarini uygula
# podman pull registry.redhat.io/rhbk/keycloak-rhel9:latest
# veya: dnf update -y rh-sso7-keycloak

# Mevcut organizasyon uyeliklerini kontrol et
# Keycloak Admin Console → Organizations → Her org icin uyelik listesini gozden gecir
# Beklenmeyen uyelikleri arastir ve gerekirse kaldir

# Audit loglari kontrol et
grep -i "organization.*member\|invite.*accept" /var/log/keycloak/keycloak.log
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Keycloak surumunu ve Organizations ozelliginin aktif olup olmadigini kontrol et
*   [ ] Organizasyon davet/kabul akisinin hangi uygulamalarda yetkilendirme icin kullanildigini belirle
*   [ ] Audit log ve erisim loglarinin aktif ve eksiksiz tutuldugunu dogrula
*   [ ] "Stateless davet" yaklasimi kullanilip kullanilmadigini kontrol et
*   [ ] Cok kiracili senaryolarda tenant sinirlarinin nasil uygulandigini belirle

**Duzeltme**
*   [ ] Keycloak'u yamali surume yukselt (26.2.13, 26.4.9 veya 26.5.0+)
*   [ ] Red Hat Build of Keycloak kullaniliyorsa ilgili errata guncellemelerini uygula
*   [ ] Organizations ozelligi zorunlu degilse gecici olarak kapat veya davet kabulunu kisitla
*   [ ] Davet kabulunde jeton claim'lerini sunucu tarafindaki kayitla eslestiren dogrulama ekle
*   [ ] Organizasyona katilimi yonetici onayina bagla
*   [ ] Davet kullanımını tek seferlik ve kisa sureli (24 saat) hale getir
*   [ ] Davet/kayit uc noktalarina rate limit ve IP kisitlamalari ekle

**Dogrulama**
*   [ ] Test ortaminda "uyumsuz claim" iceren davetlerin reddedildigini dogrula
*   [ ] Cross-tenant erisim testleri (negatif testler) yap — farkli organizasyon ID ile kabul denemesi
*   [ ] Replay korumasi testini yap (ayni jeton ikinci kez kullanildiginda red)
*   [ ] Organizasyon uyeligi degisimlerinin audit log'lara dustugunu ve SIEM'de uyari uretildigini test et
*   [ ] Jeton dogrulama birim testleri: issuer/audience, exp/nbf, jti tek-seferlik kullanim, claim butunluk kontrolleri

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Keycloak audit loglarinda organizasyon uyeligi degisimlerini (ekleme, cikarma) izle
    *   Davet kabul islemlerinde jeton dogrulama hatalarini ve reddedilen istekleri filtrele
    *   Regex: `(?i)(organization.*member|invite.*accept|invite.*reject|token.*invalid|claim.*mismatch)`
    *   Regex: `(?i)(orgId.*mismatch|email.*mismatch|token.*replay|token.*expired|jti.*duplicate)`
    *   Keycloak Admin olaylari loglarinda beklenmeyen organizasyon yapilandirma degisikliklerini izle
*   **Anomali Tespiti:**
    *   Bir kullanicinin kisa surede birden fazla organizasyona uye olmasi
    *   Ayni jeton kimliginin (jti) birden fazla kez kullanilmasi (replay saldirisi)
    *   Organizasyon davet kabullerinde basarisiz dogrulama oraninin ani artmasi
    *   Beklenmeleyen IP adreslerinden veya cografi konumlardan davet kabul islemleri
    *   Yonetici onayı olmadan organizasyona yeni uye eklenmesi

---

## Notlar
CVSS 8.1 (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N). CWE-347: Kriptografik Imzanin Hatali Dogrulanmasi. Keycloak Organizations "preview" ozelliklerinin uretimine alinmadan once threat modeling yapilmasi gerektigi vurgulaniyor. Iliskili Red Hat erratalari: RHSA-2026:2364, RHSA-2026:2365, RHSA-2026:2366. Cok kiracili SaaS senaryolarinda bu zafiyet veri sizintisi ve yetkisiz degisiklik acisindaan yuksek risklidir. Arastirma dokumani tam teknik detay icermektedir.
