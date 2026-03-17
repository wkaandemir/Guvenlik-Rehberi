# TODO: CVE-2026-23000 — Linux mlx5e Use-After-Free

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2026-23000 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2026-23000, Linux cekirdegindeki Mellanox mlx5e Ethernet surucusunde profil degistirme (profile switch) islemi sirasinda ortaya cikan bir use-after-free (UAF) zafiyetidir. Bu acik, bellek baskisi altinda cekirdek panic'ine (kernel panic) ve potansiyel olarak yerel yetki yukseltmeye (local privilege escalation) yol acabilir. Mellanox ConnectX serisi ag adaptorlerinin veri merkezi, bulut altyapisi ve yuksek performansli hesaplama (HPC) ortamlarinda yaygin olarak kullanilmasi nedeniyle etki alani oldukca genistir. Ocak 2026 Linux yama turunda 1088'den fazla zafiyet arasinda ozellikle veri merkezi ortamlari icin en kritik aciklar arasinda siniflandirilmistir.
*   **Etkilenen bilesenler:** Linux cekirdegi mlx5e Ethernet surucu modulu (mlx5_core), Mellanox ConnectX-4/5/6/7 serisi ag adaptoru kullanan tum Linux sunucular, RDMA/RoCE yapilandirmali veri merkezi ortamlari, Kubernetes ve container orkestrasyon platformlari (SR-IOV/VF kullanan), bulut saglayici altyapilari (AWS, Azure, GCP)

---

### 2. Teknik detay (nasil calisiyor)
*   mlx5e surucu modulu, ag profili degistirme (ethtool profile switch) islemi sirasinda bir `struct mlx5e_priv` nesnesini serbest birakir (free) ve ardindan ayni bellek adresine referans vermeye devam eder. Bu klasik bir use-after-free (UAF) hatasidir.
*   Profil degistirme islemi, ornegin ethtool ile kanal sayisi, ring buffer boyutu veya RSS (Receive Side Scaling) yapilandirmasi degistirildiginde tetiklenir. Surucu eski profili temizlerken (teardown), yeni profili olusturmadan once eski bellek blogu serbest birakilir.
*   Eger bu sirada baska bir cekirdek is parcacigi (thread) serbest birakilmis bellek alanina erisirse — ozellikle bellek baskisi altinda sayfa yeniden kullanimi (page reuse) gerceklestiginde — bozulmus veri uzerinden islem yapilir. Bu durum cekirdek panic'ine, bellek bozulmasina veya dikkatli bir sekilde manipule edildiginde yetki yukseltmeye yol acabilir.
*   Saldirgan, KASAN (Kernel Address Sanitizer) tarafindan tespit edilebilir olmasi nedeniyle UAF'in tetiklenme penceresi dar olsa da, heap spraying teknikleriyle serbest birakilmis alana kontrollu veri yerlestirerek cekirdek seviyesinde kod yurutme elde edebilir.
*   **Neden:** Kok neden, mlx5e surucusunun profil degistirme kod yolunda uygun referans sayimi (reference counting) veya bellek yasam suresi yonetiminin (lifetime management) yapilmamasidir. Eski profilin bellek blogu, tum referanslar gecersiz kilinmadan serbest birakilmaktadir (CWE-416: Use After Free).

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Mellanox ConnectX ag adaptoru kullanan ve zafiyetli cekirdek surumuyle calisan bir Linux sunucu.
*   **2. Normal durum:**
    ```
    1. Sistem yoneticisi ethtool ile ag profil ayarlarini degistirir
    2. mlx5e surucu profili serbest birakir ve yenisini olusturur
    3. Ag baglantisi kisa bir kesinti sonrasi normal calismaya devam eder
    4. Bellek duzgun sekilde yonetilir, referans sayimi uygulanir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    // Kavramsal: mlx5e profil degistirme sirasinda UAF tetikleme
    // (Cekirdek modulu olarak veya yerel erisimle)

    // 1. Bellek baskisi olustur (heap'i doldur)
    // mmap() ile buyuk bellek bloklari ayir ve serbest birak
    for (int i = 0; i < 10000; i++) {
        void *p = mmap(NULL, 4096, PROT_READ|PROT_WRITE,
                       MAP_PRIVATE|MAP_ANONYMOUS, -1, 0);
        munmap(p, 4096);  // Bellek baskisi olustur
    }

    // 2. Profil degistirme tetikle (ethtool uzerinden)
    // system("ethtool -L eth0 combined 4");
    // Bu sirada mlx5e_priv struct'i serbest birakilir

    // 3. Serbest birakilan alana kontrollu veri yerlestirilir (heap spray)
    // Saldirgan, slab allocator'un ayni boyuttaki bir nesneyi
    // serbest birakilmis alana yerlestirmesini tetikler

    // 4. Eski referans uzerinden bozulmus veri okunur/yazilir
    // Cekirdek seviyesinde kod yurutme veya yetki yukseltme
    ```
*   **4. Analiz:** UAF zafiyeti, profil degistirme sirasinda `mlx5e_priv` yapisinin yasam suresi yonetimindeki hatadan kaynaklanir. Bellek baskisi altinda serbest birakilmis sayfa baska bir amacla yeniden kullanildiginda, eski pointer uzerinden yapilan okuma/yazma islemleri bozulmus veriye erisir. Heap spraying ile saldirgan kontrollu veri yerlestirildiginde cekirdek seviyesinde kod yurutme elde edilebilir.
*   **5. Kanit:** Sistemde cekirdek panic'i olusur. `dmesg` ciktisinda `BUG: KASAN: use-after-free in mlx5e_*` mesaji gorunur. Bellek bozulmasi nedeniyle ag baglantisi kaybedilebilir. Basarili istismar durumunda yetkisiz islemler cekirdek loglarinda gorunur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.8 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Yerel erisim (profil degistirme yetkisi olan kullanicilar, container ortaminda SR-IOV/VF erisimi)
*   **Karmasiklik:** Orta (UAF tetikleme kolay, ancak guvenilir istismar icin heap layout kontrolu gerekir; bellek baskisi altinda daha kolay tetiklenir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Linux cekirdegini yamali surume guncelleyin (zafiyetin duzeltildigi kernel commit'ini iceren surum). Mellanox OFED suruculerini en guncel kararlı surume guncelleyin. Profil degistirme islemlerini gerekli olmadikca sinirlayin (ethtool erisimini yalnizca root ile kisitlayin). KASAN etkinlestirerek olasi UAF tetiklemelerini erken tespit edin.
*   **Orta vadeli:** mlx5e surucu parametrelerini seccomp profilleri veya cgroup'lar ile kisitlayin. Container ortamlarinda SR-IOV/VF erisimini minimum yetki prensibiyle sinirlayin. Cekirdek guncellemelerini otomatik izleme (livepatch veya kexec) surecleri kurun. Ag yapilandirma degisikliklerini degisiklik yonetim sureci (change management) kapsamina alin.
*   **Uzun vadeli:** Cekirdek modullerinde KASAN ve KMSAN (Kernel Memory Sanitizer) ile surekli fuzzing testi uygulayin. Mellanox suruculerini Rust ile yeniden yazilmis (safe wrappers) alternatifleriyle degerlendirin. Veri merkezi ortamlarinda cekirdek guncellemelerini sifir kesinti (zero downtime) ile uygulayacak canli yama (live patching) altyapisi kurun. Donanim suruculerine ozgu guvenlik denetim protokolleri olusturun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```c
// === GUVENSIZ: mlx5e profil degistirmede referans sayimi eksik ===
// drivers/net/ethernet/mellanox/mlx5/core/en_main.c (kavramsal)

static int mlx5e_switch_profile(struct mlx5e_priv *priv,
                                struct mlx5e_profile *new_profile)
{
    struct mlx5e_profile *old_profile = priv->profile;

    // Eski profili temizle ve bellegini serbest birak
    old_profile->cleanup(priv);
    kfree(old_profile);  // Bellek serbest birakildi!

    // HATA: priv->profile hala eski adrese isaret ediyor
    // Baska bir thread bu sirada priv->profile'a erisirse UAF olusur
    priv->profile = new_profile;
    new_profile->init(priv);

    return 0;
}
```

**Guvenli kod:**
```c
// === GUVENLI: Referans sayimi ve RCU ile UAF onleme ===
// drivers/net/ethernet/mellanox/mlx5/core/en_main.c (kavramsal duzeltme)

static int mlx5e_switch_profile(struct mlx5e_priv *priv,
                                struct mlx5e_profile *new_profile)
{
    struct mlx5e_profile *old_profile;

    // 1. Profil degisimini mutex ile koru
    mutex_lock(&priv->profile_lock);

    old_profile = priv->profile;

    // 2. Once yeni profili baslat (basarisiz olursa eski profil korunur)
    int err = new_profile->init(priv);
    if (err) {
        mutex_unlock(&priv->profile_lock);
        return err;
    }

    // 3. Yeni profili RCU ile atomik olarak ata
    rcu_assign_pointer(priv->profile, new_profile);

    // 4. Tum okuyucularin eski profili birakmasini bekle
    synchronize_rcu();

    // 5. Artik guvenle eski profili temizle ve serbest birak
    old_profile->cleanup(priv);
    kfree_rcu(old_profile, rcu_head);

    mutex_unlock(&priv->profile_lock);
    return 0;
}
// Yamali cekirdek surumunu kontrol et:
// uname -r
// Mellanox OFED surucu guncellemesi:
// ofed_info | grep mlx5
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Mellanox ConnectX NIC kullanan tum sunuculari envanterle
*   [ ] Cekirdek surumlerini kontrol et ve zafiyetli surumleri belirle (`uname -r`)
*   [ ] mlx5_core surucu surumlerini dogrula (`modinfo mlx5_core | grep version`)
*   [ ] SR-IOV/VF kullanan container ve sanallasstirma ortamlarini haritala

**Duzeltme**
*   [ ] Linux cekirdek guvenlik yamasini tum etkilenen sunuculara uygula
*   [ ] Mellanox OFED surucu paketini en guncel kararlı surume guncelle
*   [ ] Profil degistirme erisimini (ethtool) yalnizca yetkili yoneticilerle sinirla
*   [ ] Container ortamlarinda SR-IOV/VF erisim izinlerini minimum yetki prensibiyle yapilandir
*   [ ] KASAN etkinlestirerek UAF tespiti icin erken uyari mekanizmasi kur

**Dogrulama**
*   [ ] Profil degistirme islemi sirasinda UAF tetikleme testini yama sonrasi tekrarla
*   [ ] Cekirdek panic loglarini incele ve `mlx5e` ile iliskili hata mesajlarinin olmadigini dogrula
*   [ ] Bellek baskisi altinda ag profil degisikligi testini gerceklestir
*   [ ] KASAN ciktisinda `use-after-free in mlx5e_*` mesajinin gorulmedigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Cekirdek loglarinda mlx5e surucusuyle iliskili hata mesajlarini izle
    *   Cekirdek panic olaylarini merkezi loglama sistemine yonlendir
    *   ethtool profil degistirme komutlarini audit logundan izle
    *   Regex: `(?i)(BUG.*KASAN.*use-after-free.*mlx5e|mlx5_core.*error|kernel.*panic.*mlx5)`
    *   Regex: `(?i)(ethtool\s+-[LGK]|mlx5e.*profile.*switch|mlx5e.*teardown)`
*   **Anomali Tespiti:**
    *   Kisa surede birden fazla profil degistirme islemi (olasi tetikleme denemesi)
    *   Mellanox NIC ile iliskili beklenmeyen cekirdek modulu yukleme/kaldirma islemleri
    *   Bellek baskisi altinda ag arayuzu performansinda ani dusus
    *   Cekirdek loglarinda `KASAN`, `slab-use-after-free` veya `BUG` anahtar kelimelerinin gorunmesi
    *   Container ortaminda SR-IOV/VF arayuzlerinde beklenmeyen yapilandirma degisiklikleri

---

## Notlar
Bellek baskisi altinda daha kolay tetiklenir. Veri merkezi ortamlari icin kritik oneme sahiptir. Mellanox ConnectX adaptorlerinin bulut saglayicilarda (AWS ENA, Azure Mellanox, GCP) yaygin kullanimi nedeniyle etki alani genistir. Ocak 2026 Linux yama turunde 1088+ zafiyet arasinda ozellikle kritik altyapi sunuculari icin oncelikli olarak siniflandirilmistir. CVE-2022-3564 (Bluetooth L2CAP UAF) ile benzer kok nedeni paylasmaktadir.
