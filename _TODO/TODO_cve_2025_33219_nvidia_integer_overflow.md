# TODO: CVE-2025-33219 — NVIDIA Windows GPU Surucu Tamsayi Tasmasi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2025-33219 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Donanim/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2025-33219, NVIDIA GPU surucusunun Windows cekirdek modu bileseni (nvlddmkm.sys) icerisinde bulunan bir tamsayi tasmasi (integer overflow) zafiyetidir. CVE-2025-33218'in Windows platformu karsiligi olan bu acik, Maxwell, Pascal ve Volta serisi eski GPU'lari etkiler. Yerel bir saldirgan, ozel hazirlanmis DeviceIoControl cagrialri ile Windows cekirdek bellegini bozarak SYSTEM yetkilerine yukselebilir. NVIDIA, Ocak 2026'da bu eski GPU serileri icin guvenlik surucusu yayinlamistir (Windows: 553.62+). Ozellikle oyun bilgisayarlari, tasarim is istasyonlari ve eski donanim kullanan kurumsal masaustu sistemleri bu zafiyetten dogrudan etkilenmektedir.
*   **Etkilenen bilesenler:** NVIDIA Maxwell serisi GPU'lar (GeForce GTX 900, Quadro M serisi), NVIDIA Pascal serisi GPU'lar (GeForce GTX 1000, Quadro P serisi, Tesla P serisi), NVIDIA Volta serisi GPU'lar (Tesla V100, Titan V), Windows NVIDIA cekirdek modu surucusu (nvlddmkm.sys) 553.62 oncesi surumler

---

### 2. Teknik detay (nasil calisiyor)
*   Windows isletim sisteminde NVIDIA GPU surucusu, `nvlddmkm.sys` adli cekirdek modu surucusu (kernel-mode driver) olarak calisir. Kullanici alani uygulamalari (DirectX, CUDA, OpenGL), `DeviceIoControl` Windows API'si araciligiyla bu surucuye komut iletir.
*   Zafiyetli kodda, kullanicidan alinan bir boyut parametresi 32-bit isaretsiz tamsayi (ULONG/DWORD) olarak islenir ve ek bir baslik boyutuyla toplanir. Toplam deger 32-bit sinirini astiginda tamsayi tasmasi (wrap around) meydana gelir ve cekirdek modulu gercekten ihtiyac duyulanin cok altinda bir bellek blogu tahsis eder.
*   Sonrasinda, orijinal (buyuk) boyut degerine gore kullanici verileri bu kucuk blogun icine kopyalanir ve Windows cekirdek yigin (NonPagedPool) bellegi uzerine yazilir. Bu bellek bozulmasi, saldirganin EPROCESS yapilarini veya token bilgilerini manipule etmesine olanak tanirlir.
*   Windows ortaminda bu zafiyet ozellikle tehlikelidir cunku NVIDIA surucusu, masaustu ortaminda hemen hemen her kullanicinin sisteme giris yaptigi anda yuklu olarak calisir ve `/dev/nvidia*` erisim izinleri gibi bir Linux mekanizmasi varsayilan olarak bulunmaz.
*   **Neden:** Kok neden, CVE-2025-33218 ile aynidir: NVIDIA cekirdek surucusundeki IOCTL (DeviceIoControl) isleyicisinde kullanici girdisinin isaretsiz tamsayi aritmetigi ile islenmesi sirasinda tasma kontrolunun eksik olmasi. Windows cekirdek surucusunun legacy kod tabaninda Safe Integer fonksiyonlarinin (RtlULongAdd, RtlSizeTAdd) kullanilmamis olmasi bu zafiyeti dogurmustur.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** NVIDIA Maxwell/Pascal/Volta GPU surucusu yuklenmis Windows sistemi (is istasyonu, oyun bilgisayari veya kurumsal masaustu) uzerinde yerel kullanici oturumu
*   **2. Normal durum:**
    ```
    1. DirectX/CUDA uygulamasi, CreateFile ile NVIDIA cihazina handle acar
    2. DeviceIoControl ile GPU bellek tahsis istegi gonderir
    3. nvlddmkm.sys, istenen boyutu dogrular ve NonPagedPool'dan bellek tahsis eder
    4. GPU islemi tamamlanir ve sonuc kullanici alanina dondurulur
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```c
    #include <windows.h>

    // Kavramsal: NVIDIA DeviceIoControl ile integer overflow tetikleme
    HANDLE hDevice = CreateFileW(
        L"\\\\.\\NvidiaDev0", GENERIC_READ | GENERIC_WRITE,
        0, NULL, OPEN_EXISTING, 0, NULL
    );

    typedef struct {
        DWORD size;          // Kullanici tarafindan kontrol edilen boyut
        DWORD flags;
        LPVOID user_buffer;
    } NVIDIA_ALLOC_REQUEST;

    NVIDIA_ALLOC_REQUEST req;
    // Integer overflow: size + HEADER_SIZE (0x40) tasma yaratir
    // 0xFFFFFFC0 + 0x40 = 0x100000000 → 32-bit'te 0x00000000
    req.size = 0xFFFFFFC0;
    req.flags = 0;
    req.user_buffer = pLargePayload;

    DWORD bytesReturned;
    DeviceIoControl(hDevice, NVIDIA_IOCTL_ALLOC,
                    &req, sizeof(req),
                    NULL, 0, &bytesReturned, NULL);
    // Cekirdek 0x40 byte tahsis eder ama 0xFFFFFFC0 byte yazmaya calisir
    // NonPagedPool overflow → EPROCESS/token manipulasyonu mumkun
    ```
*   **4. Analiz:** Windows cekirdeginde NonPagedPool uzerindeki heap overflow, EPROCESS yapisindaki Token alaninin manipulasyonu icin kullanilabilir. Saldirgan, mevcut isleminin (process) guvenlik jetonu (Token) isareticisini SYSTEM isleminin jetonuyla degistirerek tam sistem yetkilerine ulasabilir. Windows Kernel Pool allocator'un on gorulebilir bellek yerlesimi (deterministic layout), istismari kolaylastirir.
*   **5. Kanit:** Basarili istismar sonucunda saldirganin cmd.exe islemi `NT AUTHORITY\SYSTEM` yetkisine yukseltilir. Windows Olay Goruntuleyicisi'nde (Event Viewer) nvlddmkm.sys ile ilgili mavi ekran (BSOD) veya hata olaylari kaydedilebilir. Sistem kararliligi bozulabilir ve tekrarlanan girisimler mavi ekrana neden olabilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.8 (AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H)
*   **Saldiri Yuzeyi:** Yerel erisim (GPU surucusu yuklenmis Windows sistemi, standart kullanici oturumu yeterli)
*   **Karmasiklik:** Orta (yerel erisim gerektirir, kernel pool layout analizi ve exploit gelistirme bilgisi gerekli, ancak GPU surucusu varsayilan olarak tum kullanicilara acik)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** NVIDIA Ocak 2026 guvenlik surucusunu uygulayin (Windows: 553.62+). GeForce Experience veya NVIDIA App uzerinden otomatik guncellemeyi zorlaylin. Maxwell, Pascal ve Volta serisi GPU envanterini cikartin ve oncelikli guncelleme plani olusturun. Kurumsal ortamlarda SCCM/Intune ile toplu surucu dagitimi yapin.
*   **Orta vadeli:** Windows Defender Exploit Guard (ASR kurallari) ile cekirdek surucusune beklenmeyen DeviceIoControl cagrilarini kisitlayin. NVIDIA surucusu guncellemelerini WSUS/SCCM ile merkezi olarak yonetin. Windows Olay Iletme (WEF) ile nvlddmkm.sys hatalarini merkezi SIEM'e aktarin. Standart kullaniciların GPU cihaz isleyicisine dogrudan erisim yetkisini kisitlayin (Group Policy).
*   **Uzun vadeli:** Eski GPU serilerinin (Maxwell, Pascal) kurumsal kullanim omrunu degerlendirin ve donanim yenileme planlari olusturun. Driver Signing ve HVCI (Hypervisor-protected Code Integrity) politikalarini etkinlestirerek cekirdek surucusu butunlugunu koruyun. Windows Sandbox veya Application Guard ile GPU gerektiren uygulamalari izole edin. Surucu guvenlik bultenleri icin otomatik izleme ve uyari sistemi kurun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```c
// === GUVENSIZ: Windows cekirdek surucusunde tamsayi tasmasi kontrolu yok ===
NTSTATUS NvidiaIoctlAlloc(PDEVICE_OBJECT DeviceObject,
                           PIRP Irp) {
    PNVIDIA_ALLOC_REQUEST req = Irp->AssociatedIrp.SystemBuffer;

    // Kullanici girdisi dogrudan kullaniliyor - integer overflow!
    ULONG totalSize = req->Size + HEADER_SIZE;

    // totalSize, req->Size cok buyukse sifira sarabilir
    PVOID buffer = ExAllocatePoolWithTag(NonPagedPool, totalSize, 'NVDA');
    if (!buffer) return STATUS_INSUFFICIENT_RESOURCES;

    // req->Size kadar veri kopyalaniyor ama buffer cok kucuk!
    RtlCopyMemory((PUCHAR)buffer + HEADER_SIZE,
                  req->UserBuffer,
                  req->Size);         // HEAP OVERFLOW!

    return STATUS_SUCCESS;
}
```

**Guvenli kod:**
```c
// === GUVENLI: Safe Integer + boyut sinirlamasi + dogrulama ===
#include <ntintsafe.h>  // RtlULongAdd, RtlSizeTAdd

#define MAX_ALLOC_SIZE  (256 * 1024 * 1024)  // Maksimum 256 MB

NTSTATUS NvidiaIoctlAlloc(PDEVICE_OBJECT DeviceObject,
                           PIRP Irp) {
    PNVIDIA_ALLOC_REQUEST req = Irp->AssociatedIrp.SystemBuffer;
    ULONG totalSize = 0;

    // 1. Maksimum boyut kontrolu
    if (req->Size > MAX_ALLOC_SIZE) {
        return STATUS_INVALID_PARAMETER;
    }

    // 2. Safe integer toplama — tasma durumunda hata dondurur
    NTSTATUS status = RtlULongAdd(req->Size, HEADER_SIZE, &totalSize);
    if (!NT_SUCCESS(status)) {
        // Tamsayi tasmasi tespit edildi
        return STATUS_INTEGER_OVERFLOW;
    }

    // 3. Guvenli bellek tahsisi
    PVOID buffer = ExAllocatePoolWithTag(NonPagedPool, totalSize, 'NVDA');
    if (!buffer) return STATUS_INSUFFICIENT_RESOURCES;

    // 4. Dogrulanmis boyutla veri kopyalama
    __try {
        ProbeForRead(req->UserBuffer, req->Size, 1);
        RtlCopyMemory((PUCHAR)buffer + HEADER_SIZE,
                      req->UserBuffer,
                      req->Size);
    } __except(EXCEPTION_EXECUTE_HANDLER) {
        ExFreePoolWithTag(buffer, 'NVDA');
        return GetExceptionCode();
    }

    return STATUS_SUCCESS;
}
```

```powershell
# NVIDIA surucu versiyonunu kontrol et
nvidia-smi --query-gpu=driver_version,gpu_name --format=csv

# GeForce Experience ile guncelle veya manuel surucu indirme:
# https://www.nvidia.com/download/find.aspx
# Minimum guvenli surum: 553.62+ (Windows)

# Kurumsal dagitim (SCCM/Intune):
# NVIDIA surucu MSI paketini dagit
# msiexec /i nvidia_driver_553.62.msi /quiet /norestart

# Windows Olay Goruntuleyicisinde nvlddmkm.sys hatalarini kontrol et
Get-WinEvent -FilterHashtable @{LogName='System'; ProviderName='nvlddmkm'} -MaxEvents 20
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] NVIDIA Maxwell, Pascal ve Volta serisi GPU envanteri cikar (nvidia-smi, DXDIAG)
*   [ ] Mevcut surucu versiyonlarini tum Windows sistemlerde kontrol et
*   [ ] Kurumsal surucu dagitim yontemini belirle (SCCM, Intune, GPO, GeForce Experience)
*   [ ] CVE-2025-33218 (Linux varyanti) ile birlikte degerlendirme yap
*   [ ] Mevcut Windows Defender Exploit Guard (ASR) politikalarini gozden gecir

**Duzeltme**
*   [ ] NVIDIA Ocak 2026 guvenlik surucusunu uygula (Windows: 553.62+)
*   [ ] Surucu guncelleme sonrasi GPU islevselligini dogrula (nvidia-smi, DxDiag)
*   [ ] Kurumsal ortamda toplu surucu dagitimi gerceklestir (SCCM/Intune)
*   [ ] Windows Defender ASR kurallarini GPU cihaz erisimi icin yapilandir
*   [ ] HVCI (Hypervisor-protected Code Integrity) politikasini etkinlestir

**Dogrulama**
*   [ ] Integer overflow exploit testini yama sonrasi tekrarla
*   [ ] Surucu versiyonunun 553.62+ oldugunu dogrula
*   [ ] Windows Olay Goruntuleyicisinde nvlddmkm.sys hata olmaydigini kontrol et
*   [ ] GPU performans testlerini calistirarak regresyon olmedigini onayla
*   [ ] SIEM'e aktarilan Windows olaylarina NVIDIA uyari kurallarinin calistigini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Windows System Event Log'da nvlddmkm.sys hata olaylarini izle (Event ID 4101, 14, 13)
    *   Windows Security Event Log'da cekirdek modulu hatalarini ve mavi ekran (bugcheck) olaylarini filtrele
    *   Regex: `(?i)(nvlddmkm|nvidia.*error|nvidia.*fault|VIDEO_TDR_FAILURE|DRIVER_IRQL)`
    *   Regex: `(?i)(integer.overflow|pool.corruption|nonpaged.pool|heap.overflow).*nvid`
    *   Windows Defender Exploit Guard uyari loglarini izle (ASR kural ihlalleri)
*   **Anomali Tespiti:**
    *   Standart kullanicilardan GPU cihaz isleyicisine (\\.\NvidiaDev*) beklenmeyen DeviceIoControl cagrialri
    *   nvlddmkm.sys ile ilgili mavi ekran (BSOD) sikliginda artis
    *   GPU kullanmayan islemlerden NVIDIA cihaz dosyalarina erisim girisimleri
    *   Sistem yetki yukseltme olaylari (token degisikligi) GPU surucusu etkinligi sonrasinda
    *   Tekrarlayan TDR (Timeout Detection and Recovery) olaylari (surucu kilitlenme belirtisi)

---

## Notlar
CVE-2025-33218 (Linux) ile birlikte Ocak 2026'da eski GPU serileri icin yayinlandi. Windows platformunda istismar, Linux'a kiyasla varsayilan erisim izinleri nedeniyle daha kolay olabilir (tum oturum acmis kullanicilar GPU'ya erisebilir). Kurumsal ortamlarda SCCM/Intune ile merkezi surucu yonetimi ve HVCI politikasi etkinlestirmesi onemlidir. Bulut ve sanallastirma ortamlarinda CVE-2025-33220 (vGPU UAF) ile birlikte degerlendirilmelidir.
