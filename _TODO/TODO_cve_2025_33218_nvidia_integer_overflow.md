# TODO: CVE-2025-33218 — NVIDIA Linux GPU Surucu Tamsayi Tasmasi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-33218 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Donanim/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-33218, NVIDIA GPU surucusunun Linux cekirdek modulu (nvidia.ko) icerisinde bulunan bir tamsayi tasmasi (integer overflow) zafiyetidir. Maxwell, Pascal ve Volta serisi eski GPU'lari etkileyen bu acik, yerel bir saldirganin ozel hazirlanmis IOCTL cagrialri araciligiyla cekirdek bellegini bozmasina ve sistem yetkilerini yukseltmesine (Privilege Escalation) olanak tanimaktadir. NVIDIA, Ocak 2026'da bu eski GPU serileri icin kritik guvenlik surucusu yayinlamistir. Ozellikle makine ogrenmesi sunucularinda, veri merkezlerinde ve is istasyonlarinda yaygin olarak kullanilan bu GPU serileri, saldirganlar icin cekici bir hedef olusturmaktadir.
*   **Etkilenen bilesenler:** NVIDIA Maxwell serisi GPU'lar (GTX 900, Quadro M serisi), NVIDIA Pascal serisi GPU'lar (GTX 1000, Quadro P serisi, Tesla P serisi), NVIDIA Volta serisi GPU'lar (Tesla V100, Titan V), Linux NVIDIA cekirdek modulu (nvidia.ko) R550.144.03 oncesi surumler

---

### 2. Teknik detay (nasil calisiyor)
*   NVIDIA GPU surucusu, kullanici alani (userspace) uygulamalarindan cekirdek modulune komut iletmek icin IOCTL (Input/Output Control) arayuzunu kullanir. GPU bellek tahsisi, donanim yapilandirmasi ve performans ayarlari gibi islemler bu kanal uzerinden gerceklestirilir.
*   Zafiyetli kodda, kullanici tarafindan saglanan bir boyut degeri (ornegin bellek tahsis miktari veya tampon boyutu) isaretsiz tamsayi (unsigned integer) olarak islenmeden once yeterli sinir kontrolunden (bounds checking) gecirilmemektedir. Saldirgan, ozel olarak hesaplanmis buyuk bir deger gondererek tamsayi tasmasina neden olur. Ornegin, `size + header_size` toplami 32-bit tamsayi sinirini asarak sifira yakin kucuk bir degere "sarar" (wrap around).
*   Tasma sonucu, cekirdek modulu gercek ihtiyactan cok daha kucuk bir bellek blogu tahsis eder, ancak kullanicinin gonderdigi buyuk veriyi bu kucuk blogun icine yazmaya calisir. Bu durum yigin tabanlı tampon tasmasina (heap buffer overflow) yol acar ve cekirdek bellegindeki kritik veri yapilarinin (ornegin islem denetim bloklari, sayfa tablolari) uzerine yazilmasina neden olur.
*   Saldirgan, bu bellek bozulmasini dikkatlice kontrol ederek cekirdek duzeyinde kod yurutme elde edebilir ve root yetkilerine yukselebilir.
*   **Neden:** Kok neden, NVIDIA cekirdek modulundeki IOCTL isleyicisinde kullanici girdisinin isaretsiz tamsayi aritmetigi ile islenmesi sirasinda tasma kontrolunun yapilmamasidir. Eski GPU serilerine ait legacy kod tabaninda modern guvenlik pratiklerinin (safe integer arithmetic, input validation) yeterince uygulanmamis olmasi bu zafiyeti dogurmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** NVIDIA Maxwell/Pascal/Volta GPU surucusu yuklenmis Linux sistemi (is istasyonu, ML sunucusu veya veri merkezi sunucusu) uzerinde yerel kullanici erisimi
*   **2. Normal durum:**
    ```
    1. Kullanici alani uygulamasi (CUDA, OpenGL), /dev/nvidia0 cihazina IOCTL cagrisi yapar
    2. Cekirdek modulu, istenen boyutu dogrular ve uygun bellek blogunu tahsis eder
    3. GPU islemi gerceklestirilir ve sonuc kullanici alanina dondurulur
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    #include <fcntl.h>
    #include <sys/ioctl.h>
    #include <stdint.h>

    // Kavramsal: NVIDIA IOCTL ile integer overflow tetikleme
    int fd = open("/dev/nvidia0", O_RDWR);

    struct nvidia_alloc_request {
        uint32_t size;        // Kullanici tarafindan kontrol edilen boyut
        uint32_t flags;
        uint64_t user_buffer;
    };

    struct nvidia_alloc_request req;
    // Integer overflow: size + HEADER_SIZE (ornegin 0x40) = 0x100000000 + 0x40
    // 32-bit tasma sonucu: gercek tahsis = 0x40 byte (cok kucuk)
    req.size = 0xFFFFFFC0;  // 0xFFFFFFC0 + 0x40 = 0x00000000 (tasma!)
    req.flags = 0;
    req.user_buffer = (uint64_t)large_payload; // 4GB veri yazilacak

    ioctl(fd, NVIDIA_IOCTL_ALLOC, &req);
    // Cekirdek 0x40 byte tahsis eder ama 0xFFFFFFC0 byte yazmaya calisir
    // Heap buffer overflow → cekirdek bellegi bozulur
    ```
*   **4. Analiz:** `req.size + HEADER_SIZE` toplami 32-bit unsigned integer sinirini (0xFFFFFFFF) asar ve sifira sarar. Cekirdek modulu, tasmis (kucuk) degere gore bellek tahsis eder ancak orijinal (buyuk) degere gore veri kopyalama yapar. Bu durum heap buffer overflow olusturur. Saldirgan, cekirdek yiginindaki (heap) komsuluk iliskilerini analiz ederek, bozulan bellek bolgesine dikkatlice yerlestirilen veri ile cekirdek yapílarini (ornegin `cred` struct) manipule edebilir.
*   **5. Kanit:** Basarili istismar sonucunda saldirganin islemi root (UID 0) yetkilerine yukseltilir. Sistem loglarinda (dmesg) NVIDIA moduluyle ilgili bellek hatalari, kernel oops veya panic mesajlari gorunebilir. `/dev/nvidia*` cihazlarina beklenmeyen IOCTL cagrialri ve uid degisim olaylari gozlemlenir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.8 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Yerel erisim (GPU surucusu yuklenmis Linux sistemi, `/dev/nvidia*` cihaz dosyalarina erisim)
*   **Karmasiklik:** Orta (yerel erisim gerektirir, heap layout analizi ve exploit gelistirme bilgisi gerekli)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** NVIDIA Ocak 2026 guvenlik surucusunu uygulayin (Linux: R550.144.03+). Maxwell, Pascal ve Volta serisi GPU'larin surucu envanterini cikartin ve oncelikli guncelleme plani olusturun. Guncelleme yapilamayan sistemlerde `/dev/nvidia*` cihaz dosyalarina erisim izinlerini kisitlayin (yalnizca gerekli kullanici gruplarina izin verin). SELinux/AppArmor politikalarini sertlestirerek GPU cihaz dosyalarina erisimi sinirlandirlirin.
*   **Orta vadeli:** GPU surucusu guncellemelerini otomatik izleyen bir surec yapilandirin (NVIDIA guvenlik bultenleri). Sunucu sistemlerinde NVIDIA surucusunu konteyner izolasyonu ile sinirlandirin (NVIDIA Container Toolkit). Linux cekirdek denetim (audit) kurallarini yapilandirarak `/dev/nvidia*` uzerindeki IOCTL cagrilarini loglayin. Cok kullanicili sistemlerde GPU paylasim politikalarini gozden gecirin.
*   **Uzun vadeli:** Eski GPU serilerinin (Maxwell, Pascal) kullanim omrunu degerlendirin ve yapisal olarak yenileme planlari olusturun. GPU kullanan is yuklerini Kubernetes uzerinde izole edilmis pod'lar ile calistirin. Cekirdek modulu guvenlik testlerini CI/CD sureclere entegre edin (syzkaller, KASAN). NVIDIA surucu guncellemelerini zorunlu kilacak kurumsal politikalar belirleyin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```c
// === GUVENSIZ: IOCTL isleyicisinde tamsayi tasmasi kontrolu yok ===
static int nvidia_ioctl_alloc(struct nvidia_state *state,
                               struct nvidia_alloc_request *req) {
    // Kullanici girdisi (req->size) dogrudan kullaniliyor
    size_t total_size = req->size + HEADER_SIZE; // INTEGER OVERFLOW!
    // total_size, req->size cok buyukse sifira sarabilir

    void *buffer = kmalloc(total_size, GFP_KERNEL);
    if (!buffer) return -ENOMEM;

    // req->size kadar veri kopyalaniyor ama buffer cok kucuk!
    if (copy_from_user(buffer + HEADER_SIZE,
                       (void __user *)req->user_buffer,
                       req->size)) {          // HEAP OVERFLOW!
        kfree(buffer);
        return -EFAULT;
    }
    return 0;
}
```

**Guvenli kod:**
```c
// === GUVENLI: Tamsayi tasmasi kontrolu + boyut sinirlamasi ===
#include <linux/overflow.h>  // check_add_overflow() makrosu

#define MAX_ALLOC_SIZE  (256 * 1024 * 1024)  // Maksimum 256 MB

static int nvidia_ioctl_alloc(struct nvidia_state *state,
                               struct nvidia_alloc_request *req) {
    size_t total_size;

    // 1. Maksimum boyut kontrolu
    if (req->size > MAX_ALLOC_SIZE) {
        pr_warn("nvidia: asiri buyuk tahsis istegi: %zu\n", req->size);
        return -EINVAL;
    }

    // 2. Tamsayi tasmasi kontrolu (Linux cekirdek makrosu)
    if (check_add_overflow(req->size, (size_t)HEADER_SIZE, &total_size)) {
        pr_warn("nvidia: tamsayi tasmasi tespit edildi\n");
        return -EOVERFLOW;
    }

    // 3. Guvenli bellek tahsisi
    void *buffer = kmalloc(total_size, GFP_KERNEL);
    if (!buffer) return -ENOMEM;

    // 4. Dogrulanmis boyutla veri kopyalama
    if (copy_from_user(buffer + HEADER_SIZE,
                       (void __user *)req->user_buffer,
                       req->size)) {
        kfree(buffer);
        return -EFAULT;
    }
    return 0;
}
```

```bash
# NVIDIA surucu versiyonunu kontrol et
nvidia-smi --query-gpu=driver_version,gpu_name --format=csv

# Guvenli surume guncelle (Linux: R550.144.03+)
# Ubuntu/Debian:
sudo apt update && sudo apt install nvidia-driver-550
# RHEL/CentOS:
sudo dnf install nvidia-driver-550.144.03

# GPU cihaz dosyasi erisim izinlerini kisitla
sudo chmod 660 /dev/nvidia*
sudo chown root:gpu-users /dev/nvidia*

# Denetim kurali ekle
sudo auditctl -w /dev/nvidia0 -p rw -k nvidia_access
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] NVIDIA Maxwell, Pascal ve Volta serisi GPU envanteri cikar (nvidia-smi)
*   [ ] Mevcut surucu versiyonlarini tum sunucu ve is istasyonlarinda kontrol et
*   [ ] `/dev/nvidia*` cihaz dosyalarina kimlerin erisim hakkinin oldugunu belirle
*   [ ] SELinux/AppArmor politikalarinin GPU cihazlarini kapsayip kapsamadigini kontrol et
*   [ ] CVE-2025-33219 (Windows varyanti) ile birlikte degerlendirme yap

**Duzeltme**
*   [ ] NVIDIA Ocak 2026 guvenlik surucusunu uygula (Linux: R550.144.03+)
*   [ ] Surucu guncelleme sonrasi GPU islevselligini dogrula (nvidia-smi, CUDA testleri)
*   [ ] `/dev/nvidia*` erisim izinlerini kisitla (chmod 660, grup bazli erisim)
*   [ ] SELinux/AppArmor ile GPU IOCTL erisimini sinirlandiran politika tanimla
*   [ ] Cok kullanicili sistemlerde GPU paylasim politikalarini guncelle

**Dogrulama**
*   [ ] Integer overflow exploit testini yama sonrasi tekrarla
*   [ ] Surucu versiyonunun guvenli surumde oldugunu dogrula (nvidia-smi)
*   [ ] Cekirdek loglarinda (dmesg) NVIDIA modulu ile ilgili hata olmaydigini kontrol et
*   [ ] Audit loglarinda IOCTL izleme kurallarinin calistigini dogrula
*   [ ] GPU performans testlerini calistirarak yama sonrasi regresyon olmedigini onayla

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Linux denetim (audit) loglarinda `/dev/nvidia*` cihazlarina beklenmeyen IOCTL cagrialrini izle
    *   dmesg/journalctl'de NVIDIA modulu hata ve uyari mesajlarini filtrele
    *   Regex: `(?i)(nvidia.*overflow|nvidia.*oops|nvidia.*panic|nvidia.*fault|BUG.*nvidia)`
    *   Regex: `(?i)(integer.overflow|heap.overflow|buffer.overflow|use.after.free).*nvidia`
    *   SELinux/AppArmor ihlal loglarinda GPU cihaz dosyasi erisim redlerini izle
*   **Anomali Tespiti:**
    *   Normal kullanim disinda `/dev/nvidia*` cihazlarina erisim yapan beklenmeyen islemler
    *   Cekirdek bellek kullaniminda ani ve asiri artislar (kmalloc hatalari)
    *   GPU kullanimayan bir kullanicinin `/dev/nvidia0` dosyasina IOCTL cagrisi yapmasi
    *   Sistem uid degisiklikleri sonrasi GPU cihaz erisimi (yetki yukseltme belirtisi)
    *   Kernel panic veya oops olaylari NVIDIA modulu stack trace'i iceriyorsa

---

## Notlar
CVE-2025-33219 (Windows varyanti) ile birlikte Ocak 2026'da eski GPU serileri icin yayinlandi. Maxwell, Pascal ve Volta serisi GPU'lar artik legacy destek kategorisinde olup guvenlik yamalarinin surekliligi garanti degildir — uzun vadeli donasim yenileme planlamasi onemlidir. Bulut ve sanallastirma ortamlarinda bu GPU'lar kullanilyorsa CVE-2025-33220 (vGPU UAF) ile birlikte degerlendirilmelidir.
