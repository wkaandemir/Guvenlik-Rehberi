# TODO: CVE-2025-33220 — NVIDIA vGPU Use-After-Free Izolasyon Ihlali

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-33220 |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Donanim/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-33220, NVIDIA vGPU (Virtual GPU) yaziliminda bulunan kritik bir use-after-free (UAF) zafiyetidir. Sanallastirma ortamlarinda GPU kaynaklarini sanal makineler arasinda paylastirmak icin kullanilan vGPU mimarisinde, serbest birakilmis bir bellek bolgesine tekrar erisim yapilmasindan kaynaklanan bu acik, bir misafir (guest) sanal makinenin ana bilgisayar (host) sistem bellegine erisim saglamasina olanak tanimaktadir. Bu durum, bulut bilisim ve sanallastirma ortamlarinin temel guvenlik prensibi olan kiracilari arasi izolasyonu (tenant isolation) kirmaktadir. Bulut saglayicilari, veri merkezleri ve GPU-as-a-Service ortamlari bu zafiyetten dogrudan etkilenmektedir.
*   **Etkilenen bilesenler:** NVIDIA vGPU yazilimi (Virtual GPU Manager), VMware vSphere/ESXi uzerinde NVIDIA vGPU, Citrix Hypervisor/XenServer uzerinde NVIDIA vGPU, KVM/QEMU uzerinde NVIDIA vGPU (Red Hat Virtualization, Proxmox), NVIDIA GRID vGPU ve vComputeServer lisanslari, AWS, Azure, GCP gibi bulut saglayicilarinda NVIDIA GPU ornekleri

---

### 2. Teknik detay (nasil calisiyor)
*   NVIDIA vGPU mimarisi, fiziksel GPU kaynaklarini birden fazla sanal makine (VM) arasinda paylastirmak icin hypervisor katmaninda bir vGPU Manager calistirir. Her VM, kendisine atanmis sanal GPU uzerinden GPU islemlerini gerceklestirir ve vGPU Manager bu istekleri fiziksel GPU'ya yonlendirir.
*   vGPU Manager, her VM icin GPU bellek tahsisleri, komut kuyrugu (command queue) ve durum bilgilerini yoneten veri yapilari olusturur. Bu veri yapilari, VM'in GPU islemleri sirasinda dinamik olarak tahsis edilir ve serbest birakilir.
*   Use-after-free zafiyeti, bir GPU kaynak nesnesinin (ornegin bir channel veya context nesnesi) serbest birakilmasindan sonra, hala gecerli bir isareticisi (pointer) bulunan baska bir kod yolunun bu nesneye erismeye devam etmesiyle tetiklenir. Tipik senaryo: VM icindeki GPU surucusu belirli bir sirada kaynak olusturma ve yok etme islemleri gerceklestirdiginde, vGPU Manager'daki referans sayaci (reference count) hatasi nedeniyle nesne erken serbest birakilir ancak referans hala aktif kalir.
*   Saldirgan, misafir VM icerisinden bu yaris durumunu (race condition) tetikleyerek serbest birakilmis bellek bolgesini kontrol edebilir. Serbest birakilan bellek blogu, host tarafinda baska bir amacla yeniden tahsis edildiginde, saldirganin VM'i bu bellek bolgesinin icerigini okuyabilir veya degistirebilir. Bu durum, host bellegindeki diger VM'lerin verileri, sifreleme anahtarlari veya yonetim bilgilerine erisim saglar.
*   **Neden:** Kok neden, vGPU Manager'daki kaynak yasam dongusu (lifecycle) yonetiminde referans sayma (reference counting) hatasidir. GPU kaynagi serbest birakilirken tum referanslarin gecersiz kilinmamasi (dangling pointer) ve es zamanli erisim (concurrency) kontrolundeki eksiklikler bu use-after-free zafiyetini dogurmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** NVIDIA vGPU Manager calistiran hypervisor (VMware ESXi, Citrix Hypervisor, KVM) uzerinde misafir VM erisimi olan saldirgan
*   **2. Normal durum:**
    ```
    1. VM icindeki uygulama, GPU bellek tahsisi ister (CUDA/OpenGL)
    2. Misafir GPU surucusu, vGPU Manager'a kaynak tahsis istegi iletir
    3. vGPU Manager, host GPU belleginden izole bir bolge tahsis eder
    4. VM, yalnizca kendi tahsis edilen bellek bolgesine erisebilir
    5. Islem tamamlandiginda kaynak serbest birakilir ve referanslar temizlenir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    // Kavramsal: Misafir VM icerisinden vGPU UAF tetikleme
    // 1. GPU kanali (channel) olustur
    NvChannel *channel = nvCreateChannel(gpu_device);

    // 2. Kanal uzerinde GPU bellek nesnesi tahsis et
    NvMemObject *mem = nvAllocMemory(channel, GPU_MEM_SIZE);

    // 3. Asenkron islem baslat (vGPU Manager referans tutar)
    nvSubmitAsyncWork(channel, mem);

    // 4. Referans hala aktifken kanali yok et (race condition)
    // vGPU Manager, channel'i serbest birakir ama async referans hala var
    nvDestroyChannel(channel);

    // 5. Serbest birakilan bellek bolgesi uzerinden veri oku
    // Host bellegi yeniden tahsis ettiginde, diger VM verileri gorunur
    for (int i = 0; i < SPRAY_COUNT; i++) {
        NvMemObject *spray = nvAllocMemory(new_channel, GPU_MEM_SIZE);
        // spray, serbest birakilan channel bellegini kaplayabilir
        char *leaked_data = nvMapMemory(spray);
        // leaked_data, host veya diger VM verilerini icerebilir
        if (contains_host_data(leaked_data)) {
            exfiltrate(leaked_data);
        }
    }
    ```
*   **4. Analiz:** vGPU Manager, `nvDestroyChannel` cagrildiginda channel nesnesini serbest birakir ancak `nvSubmitAsyncWork` tarafindan tutulan referansi gecersiz kilmaz. Asenkron islem tamamlandiginda veya iptal edildiginde, serbest birakilmis bellek bolgesine erismye calisir (dangling pointer dereference). Saldirgan, heap spraying teknigi ile serbest birakilan bellek bolgesini kendi kontrol ettigi verilerle doldurarak, host bellegindeki verileri okuyabilir veya vGPU Manager'in kod akisini manipule edebilir.
*   **5. Kanit:** Basarili istismar durumunda, misafir VM icinden host bellegindeki veriler (diger VM'lerin GPU bellegi, sifreleme anahtarlari, vGPU Manager yapilandirmasi) okunabilir. Host uzerinde vGPU Manager loglarinda bellek bozulmasi hatalari, beklenmeyen channel durumlari veya GPU recovery olaylari gorunebilir. En kotu durumda, saldirgan host cekirdeginde kod yurutme elde ederek tum sanallastirma ortamini ele gecirebilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 8.8 (AV:L/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H — Scope Changed: misafirden host'a gecis)
*   **Saldiri Yuzeyi:** Sanallastirma ortami (vGPU etkinlestirilmis hypervisor uzerinde misafir VM erisimi)
*   **Karmasiklik:** Orta (VM icinden erisim gerektirir, race condition tetikleme ve heap spraying bilgisi gerekli, ancak bulut ortamlarinda VM erisimi kolay elde edilebilir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** NVIDIA vGPU yazilimini en son guvenli surume guncelleyin. vGPU Manager ve misafir suruculerini ayni anda guncelleyin (uyumluluk icin). Kritik is yuklerini barindiran VM'leri vGPU kullanmayan hypervisor nodlarina gecici olarak tasiyin. vGPU Manager loglarini yuksek oncelikli olarak izlemeye alin.
*   **Orta vadeli:** vGPU kullanan sanallastirma ortamlarinda kiracilari arasi izolasyonu guclendirecek ek kontroller uygulayin (ayri fiziksel GPU'lar, ayri NUMA node'lar). Hypervisor duzeyinde GPU bellek izolasyonu icin IOMMU/VT-d yapilandirmasini dogrulayin. vGPU atamalarinda "dedicated" (ozel) mod kullanarak kaynak paylasimlni sinirlandirlarin. Duzenli guvenlik taramasi ve penetrasyon testi dahil edin.
*   **Uzun vadeli:** GPU sanallastirma mimarisini NVIDIA MIG (Multi-Instance GPU) veya SR-IOV tabanli donanim izolasyonuna gecirin (Ampere ve sonrasi GPU'lar). Bulut ortamlarinda "gizli bilisim" (confidential computing) yeteneklerine sahip GPU cozumlerini degerlendirin. vGPU guvenlik bultenleri icin otomatik izleme ve yamala sureci olusturun. Sanallastirma ortami guvenlik denetimlerini yillik olarak gerceklestirin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```c
// === GUVENSIZ: Referans sayma hatasi — dangling pointer ===
struct vgpu_channel {
    int ref_count;
    void *gpu_memory;
    struct async_work *pending_work;
};

void vgpu_destroy_channel(struct vgpu_channel *ch) {
    // Asenkron islem hala referans tutuyor olabilir!
    free_gpu_memory(ch->gpu_memory);
    kfree(ch);  // Nesne serbest birakildi ama pending_work hala ch'ye isaret ediyor
    // HATA: ch->pending_work->channel artik dangling pointer!
}

void vgpu_async_complete(struct async_work *work) {
    // USE-AFTER-FREE: work->channel zaten serbest birakilmis!
    struct vgpu_channel *ch = work->channel;
    ch->gpu_memory = NULL;  // Serbest birakilmis belleye yazma!
}
```

**Guvenli kod:**
```c
// === GUVENLI: Atomik referans sayma + null kontrolu + serilestirilmis erisim ===
#include <linux/kref.h>
#include <linux/mutex.h>

struct vgpu_channel {
    struct kref refcount;       // Atomik referans sayaci
    struct mutex lock;          // Erisim kilidi
    void *gpu_memory;
    struct async_work *pending_work;
    bool is_destroyed;          // Yok etme durumu flagi
};

// Referans sayaci sifira dustugunde cagrilan temizleme fonksiyonu
static void vgpu_channel_release(struct kref *kref) {
    struct vgpu_channel *ch = container_of(kref, struct vgpu_channel, refcount);
    free_gpu_memory(ch->gpu_memory);
    ch->gpu_memory = NULL;
    kfree(ch);
}

void vgpu_destroy_channel(struct vgpu_channel *ch) {
    mutex_lock(&ch->lock);
    ch->is_destroyed = true;

    // Asenkron isleri iptal et
    if (ch->pending_work) {
        cancel_async_work(ch->pending_work);
    }
    mutex_unlock(&ch->lock);

    // Referans sayacini azalt — sifira dustugunde otomatik temizlenir
    kref_put(&ch->refcount, vgpu_channel_release);
}

void vgpu_async_complete(struct async_work *work) {
    struct vgpu_channel *ch = work->channel;

    // Referans al — basarisizsa nesne zaten serbest birakilmis
    if (!kref_get_unless_zero(&ch->refcount))
        return;  // Nesne artik yok, guvenli cikis

    mutex_lock(&ch->lock);
    if (!ch->is_destroyed && ch->gpu_memory) {
        // Guvenli islem
        finalize_gpu_work(ch);
    }
    mutex_unlock(&ch->lock);

    kref_put(&ch->refcount, vgpu_channel_release);
}
```

```bash
# vGPU yazilim versiyonunu kontrol et
nvidia-smi vgpu --query | grep "Driver Version"

# Host uzerinde vGPU Manager versiyonu
nvidia-smi -q | grep "Driver Version"

# vGPU guvenli surume guncelle (host + guest ayni anda)
# VMware ESXi:
esxcli software vib install -d /tmp/NVIDIA-vGPU-VMware-*.zip
# KVM/QEMU:
rpm -Uvh nvidia-vgpu-manager-*.rpm
systemctl restart nvidia-vgpu-mgr

# vGPU izolasyonunu dogrula
nvidia-smi vgpu -q | grep -A10 "VM ID"
# IOMMU etkinligini dogrula
dmesg | grep -i "IOMMU\|DMAR\|VT-d"
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] vGPU kullanan tum sanallastirma ortamlarini (hypervisor node'lari) envanterle
*   [ ] vGPU Manager ve misafir surucu versiyonlarini kontrol et
*   [ ] Kiracilari arasi izolasyon kontrollerini (IOMMU/VT-d, ayri GPU atamasi) gozden gecir
*   [ ] vGPU lisans tiplerini (GRID, vComputeServer) ve atama modlarini (shared, dedicated) belirle
*   [ ] CVE-2025-33218 ve CVE-2025-33219 ile birlikte degerlendirme yap

**Duzeltme**
*   [ ] NVIDIA vGPU yazilimini guvenli surume guncelle (host vGPU Manager + guest suruculer)
*   [ ] Guncelleme oncesi VM snapshot'lari al ve geri alma plani hazirla
*   [ ] IOMMU/VT-d yapilandirmasinin etkin oldugunu dogrula
*   [ ] Kritik is yuklerini barindiran VM'ler icin dedicated GPU atamasi yapilandir
*   [ ] vGPU Manager loglarini merkezi SIEM'e yonlendir

**Dogrulama**
*   [ ] Guest-to-host bellek erisim testini yama sonrasi tekrarla
*   [ ] Farkli VM'ler arasinda GPU bellek izolasyonunu dogrula
*   [ ] vGPU Manager ve misafir surucu surum uyumlulugunuu kontrol et
*   [ ] Hypervisor duzeyinde GPU kaynak izolasyon testlerini calistir
*   [ ] vGPU performans testlerini gerceklestirerek regresyon olmaydigini onayla

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   vGPU Manager loglarinda bellek bozulmasi, channel hatalari ve recovery olaylarini izle
    *   Hypervisor loglarinda GPU ile ilgili hata ve uyari mesajlarini filtrele
    *   Regex: `(?i)(vgpu.*error|vgpu.*fault|use.after.free|channel.*destroy|gpu.*recovery)`
    *   Regex: `(?i)(nvidia-vgpu-mgr|nvidia.*memory.*corrupt|invalid.*channel|dangling.*ref)`
    *   Misafir VM loglarinda GPU surucusu hata ve recovery mesajlarini izle
*   **Anomali Tespiti:**
    *   Bir VM'in GPU bellek kullaniminin tahsis edilen limitin uzerine cikmasi
    *   vGPU Manager'da anormal channel olusturma/yok etme frekansi (saniyede yuzlerce islem)
    *   Misafir VM'den host bellek adres alanina erisme girisimleri (IOMMU ihlalleri)
    *   GPU recovery olaylarinin normalden fazla gerçeklesmesi (driver reset sirkligi)
    *   Bir VM'in GPU islem kuyrugunda diger VM'lere ait veri kalintilari

---

## Notlar
Sanallastirma izolasyonunu kiran kritik zafiyet — bulut ortamlari icin en yuksek oncelik. CVE-2025-33218 (Linux integer overflow) ve CVE-2025-33219 (Windows integer overflow) ile birlikte degerlendirilmelidir. NVIDIA MIG (Multi-Instance GPU) mimarisi, Ampere ve sonrasi GPU'larda donanim duzeyinde izolasyon saglayarak bu tur yazilim tabanli izolasyon ihlallerini onleyebilir — uzun vadeli gecis plani olusturulmalidir. Bulut saglayicilari (AWS, Azure, GCP) kendi altyapilarinda yamalamayi genellikle hizli gerceklestirir, ancak on-premise vGPU kurulumlari ozel dikkat gerektirir.
