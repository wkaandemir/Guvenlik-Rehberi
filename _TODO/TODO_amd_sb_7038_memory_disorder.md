# TODO: AMD-SB-7038 — AMD Memory Disorder Yan Kanal Saldirisi

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | AMD-SB-7038 |
| **Kritiklik** | Orta |
| **Kategori Klasoru** | Donanim/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** AMD islemcilerde "bellek siralama" (memory reordering) ve "sira disi yurutme" (out-of-order execution) mekanizmalarini istismar eden yeni bir yan kanal (side-channel) saldirisi tespit edilmistir. "MEMORY DISORDER" olarak adlandirilan bu arastirma, saldirganlarin geleneksel zamanlama olcumleri yapmadan, yalnizca bellek siralama paternlerini gozlemleyerek diger sureclerden hassas veri sizdirabilecegini gostermektedir. Bu zafiyet, performans artirici donanim ozelliklerinin guvenlik riskleri olusturabileceginin bir baska kanitdir ve bulut ortamlari ile coklu kiracili (multi-tenant) sunucularda ozellikle endise vericidir.
*   **Etkilenen bilesenler:** AMD islemciler (modern mimariler), bellek siralama birimi (memory ordering unit), sira disi yurutme motoru (out-of-order execution engine), islemci onbellegi (cache), coklu kiracili bulut ortamlari, sanallastirma platformlari

---

### 2. Teknik detay (nasil calisiyor)
*   Modern islemciler performansi artirmak icin talimatlari program sirasina gore degil, veri hazirligina gore yeniden siralar (out-of-order execution). Bellek islemleri de benzer sekilde yeniden siralanabilir (memory reordering).
*   MEMORY DISORDER saldirisi, bu yeniden siralama mekanizmasinin davranissal kaliplarini gozlemleyerek yan kanal olusturur. Saldirgan, kendi surecindeki bellek erisim kaliplarini kontrol eder ve hedef surecin bellek erisimlerinin yeniden siralama davranisini nasil etkiledigini olcer.
*   Geleneksel yan kanal saldirilarindan (Spectre, Meltdown) farkli olarak, bu saldiri zamanlama olcumlerine (timing measurements) bagimli degildir. Bunun yerine, bellek siralama kurallarinin ihlal edilip edilmedigini gozlemler.
*   Islemcinin bellek siralama birimi, farkli sureclerin bellek erisimlerini birbirinden izole etmekte yetersiz kaldiginda, bir surecin bellek erisim patterni baska bir surecin bellek erisim davranisini etkileyebilir. Bu etki, saldirgan tarafindan olculebilir bir bilgi sizintisina donusturulur.
*   **Neden:** Kok neden, modern islemci mimarilerinde performans optimizasyonlarinin (out-of-order execution, speculative execution, memory reordering) guvenlik izolasyonundan once gelecek sekilde tasarlanmasidir. Donanim seviyesindeki bu tasarim tercihi, yazilim tabanli azaltmalari zorunlu kilmaktadir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Ayni fiziksel islemcide calisan farkli bir surecin veya sanal makinenin hassas bellegi.
*   **2. Normal durum:**
    ```
    1. Islemci, Surec A ve Surec B'nin bellek erisimlerini bagimsiz olarak isler
    2. Bellek siralama kurallari (memory ordering rules) her surec icin tutarli uygulanir
    3. Surec A, Surec B'nin bellek erisim kaliplarini gozlemleyemez
    4. Donanim izolasyonu, surecler arasi bilgi sizintisini onler
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan (Surec A), belirli bir bellek erisim kalıbı olusturur:
       - Belirli cache satirlarini belirli bir sirada okur/yazar
       - Bu kalip, islemcinin bellek siralama birimini belirli bir duruma getirir
    2. Hedef surec (Surec B), hassas veri uzerinde islem yapar:
       - Ornegin bir sifreleme anahtariyla bellek operasyonu gerceklestirir
    3. Saldirgan, kendi bellek erisimlerinin yeniden siralama davranisini olcer:
       - Surec B'nin bellek erisimi, siralama biriminin durumunu degistirir
       - Bu degisiklik, Surec A'nin bellek erisim sirasini etkiler
    4. Saldirgan, gozlemlenen siralama degisikliklerinden hedef surecin
       bellek erisim kalibini cikarir ve hassas veriyi bit bit yeniden olusturur
    ```
*   **4. Analiz:** Saldiri, zamanlama olcumu yerine bellek siralama davranisini kullanarak mevcut yan kanal savunmalarini (ornegin constant-time uygulamalari) atlayabilir. Islemcinin bellek siralama birimi, farkli surecler arasinda tam izolasyon saglamadigi icin bilgi sizintisi olusur.
*   **5. Kanit:** Basarili istismar durumunda saldirgan, ayni fiziksel islemcide calisan baska bir surecin (veya sanal makinenin) sifreleme anahtarlari, parola hash'leri veya diger hassas verilerini parcaciklar halinde elde edebilir. Saldirinin bant genisligi dusuktur ancak yuksek degerli hedefler icin yeterli olabilir.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Orta
*   **CVSS Skoru:** Henuz resmi CVSS atanmamistir (arastirma nitelikli — tahmini 5.0-6.5 arasi)
*   **Saldiri Yuzeyi:** Yerel erisim (ayni fiziksel islemci uzerinde kod calistirma gerektir), bulut ortamlarinda ayni host uzerindeki diger kiracilara erisim
*   **Karmasiklik:** Yuksek (akademik arastirma seviyesinde uzmanlık gerektirir, pratik istismar zor)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** AMD islemci envanterini cikarip etkilenen modelleri belirleyin. AMD firmware/mikrocode guncellemelerini uygulayin. Hassas is yuklerini (sifreleme, anahtar yonetimi) izole edilmis islemci cekirdeklerine sabitleyin (CPU pinning).
*   **Orta vadeli:** Coklu kiracili bulut ortamlarinda hassas is yuklerini ayri fiziksel sunuculara tasimak icin planlama yapin. Islemci duzeyi izolasyon mekanizmalarini (cgroup, CPU sets) kullanaarak hassas surecleri farkli cekirdeklere atanin. Hypervisor seviyesinde ek izolasyon kontrolleri uygulayin.
*   **Uzun vadeli:** Donanim tedarikci seciminde yan kanal dayanikliligi kriterini onceliklendirin. Yeni nesil islemcilerde (AMD Zen 5+) bellek siralama izolasyonunun iyilestirildigini dogrulayin. Kriptografik uygulamalarda yan kanal direncli algoritmalar (constant-time, blinding) kullanin. "Defense in depth" prensibiyle donanim + yazilim + mimari katmanlarda savunma derinligi olusturun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli durum (izolasyon eksikligi):**
```bash
# Hassas is yukleri herhangi bir cekirdekte calistirilir — yan kanal riski
taskset -c 0-7 /opt/crypto-service/keymanager
# Diger kullanicilar/surecler de ayni cekirdekleri paylasir
```

**Guvenli yapilandirma:**
```bash
# 1. AMD islemci modelini ve mikrocode versiyonunu kontrol et
cat /proc/cpuinfo | grep -E "model name|microcode"
lscpu | grep -E "Model name|Vulnerability"

# 2. Mevcut yan kanal azaltmalarini kontrol et
cat /sys/devices/system/cpu/vulnerabilities/*

# 3. Mikrocode guncellemesini uygula
sudo apt update && sudo apt install amd64-microcode  # Debian/Ubuntu
sudo dnf install linux-firmware  # RHEL/Fedora
sudo dracut --force  # Initramfs guncelle
sudo reboot  # Yeni mikrocode icin yeniden baslat

# 4. Hassas is yuklerini izole edilmis cekirdeklere sabitle
# /etc/cgconfig.conf:
# group sensitive_workload {
#   cpuset {
#     cpuset.cpus = "0-3";     # Yalnizca ilk 4 cekirdek
#     cpuset.mems = "0";       # Yalnizca ilk NUMA dugumu
#     cpuset.cpu_exclusive = "1";  # Ozel kullanim
#   }
# }

# Hassas servisi izole cekirdeklerde calistir
sudo cgexec -g cpuset:sensitive_workload /opt/crypto-service/keymanager

# 5. Kernel parametreleri ile ek izolasyon
# /etc/default/grub GRUB_CMDLINE_LINUX'e ekle:
# isolcpus=0-3 nohz_full=0-3 rcu_nocbs=0-3
# sudo update-grub && sudo reboot
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] AMD islemci envanteri cikar (model, mimari nesil, mikrocode surumu)
*   [ ] Hassas is yuklerini belirle (sifreleme, anahtar yonetimi, finansal islemler)
*   [ ] Coklu kiracili ortamlarda islemci paylasim durumunu degerlendir
*   [ ] Mevcut yan kanal azaltmalarinin durumunu kontrol et

**Duzeltme**
*   [ ] AMD firmware/mikrocode guncellemesini uygula
*   [ ] Hassas is yuklerini izole cekirdeklere sabitle (CPU pinning)
*   [ ] Hypervisor seviyesinde izolasyon kontrollerini guclendir
*   [ ] Kriptografik uygulamalarda constant-time implementasyonlari dogrula

**Dogrulama**
*   [ ] Yan kanal saldiri testini arastirma ortaminda tekrarla
*   [ ] Bellek erisim paternlerini izleme altyapisini kur
*   [ ] Mikrocode guncelleme sonrasi performans etkisini olc
*   [ ] Izolasyon kontrollerinin etkinligini dogrula

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   AMD mikrocode guncelleme durumunu periyodik olarak kontrol et
    *   Hassas sureclerin CPU cekirdek atamalarindaki degisiklikleri izle
    *   Anormal bellek erisim kaliplarini hardware performance counter'lar uzerinden tespit et
    *   Regex: `(?i)(memory.*?disorder|side.?channel|amd.?sb.?7038|speculative.*?execution.*?leak)`
*   **Anomali Tespiti:**
    *   Ayni fiziksel islemcide cok sayida bellek erisim hatasi (page fault) olusturan surecler
    *   Hassas sureclerin beklenmedik cekirdek degisiklikleri (CPU migration)
    *   Performance counter'larda anormal cache miss oranlari
    *   Sifreleme islemleri sirasinda beklenmedik performans dalgalanmalari

---

## Notlar
Performans ve guvenlik arasindaki temel gerilimi yansitan arastirma nitelikli bir zafiyet. Spectre/Meltdown ailesinin bir devami olarak degerlendirilir ancak zamanlama olcumlerine bagimli olmamasi nedeniyle mevcut bazi savunmalari atlayabilir. Pratik istismar zorlugu yuksektir, ancak bulut ortamlarinda ve coklu kiracili sistemlerde stratejik oneme sahiptir. AMD'nin resmi guvenlik bulteni AMD-SB-7038 olarak yayinlanmistir.
