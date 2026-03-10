# TODO: Android Dolby Digital Plus Zero-Click — Android Dolby Zero-Click RCE

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | Android Dolby Digital Plus Zero-Click |
| **Kritiklik** | Kritik |
| **Kategori Klasoru** | Mobil/ |
| **Kaynak Arastirma** | [Arastirmalar/Ocak 2026 Güvenlik Açıkları Araştırması.md](../Arastirmalar/Ocak%202026%20G%C3%BCvenlik%20A%C3%A7%C4%B1klar%C4%B1%20Ara%C5%9Ft%C4%B1rmas%C4%B1.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** Android Ocak 2026 Guvenlik Bulteni'nde raporlanan bu zafiyet, Dolby Digital Plus kodek bilesenindeki kritik bir bellek bozulma hatasini icermektedir. Saldirganlar, bir ses dosyasindaki metadata alanlarini (evolution data) manipule ederek, kurbanin hicbir etkilesimi olmadan (zero-click) cihazda uzaktan kod yurutebilmektedir. WhatsApp, Telegram gibi mesajlasma uygulamalari uzerinden gonderilen zararli ses dosyasi, kullanici tarafindan oynatilmasa bile otomatik medya indirme mekanizmasi sayesinde cihazin ele gecirilmesi icin yeterlidir. Google, bu sorunu kendi Pixel cihazlari icin daha erken cozmesine ragmen, Samsung, Xiaomi ve diger OEM ureticilerin cihazlari Ocak ayi boyunca buyuk risk altinda kalmistir.
*   **Etkilenen bilesenler:** Android isletim sistemi (ozellikle Google disindaki OEM ureticilerin cihazlari), Dolby Digital Plus ses kodek kutuphanesi, medya islemci (MediaCodec) altyapisi, otomatik medya indirmeyi destekleyen tum mesajlasma uygulamalari (WhatsApp, Telegram, Signal vb.)

---

### 2. Teknik detay (nasil calisiyor)
*   Dolby Digital Plus kodek kutuphanesi, ses dosyalarindaki metadata alanlarini (ozellikle "evolution data" blogu) ayristirirken (parse ederken) girdi boyutunu yeterince dogrulamaz. Manipule edilmis bir metadata alani, heap tabali bellek tasmasina (heap buffer overflow) neden olur.
*   Saldirganlarin hazirladigi ozel ses dosyasi, metadata icinde asiri uzun veya beklenmeyen formatta "evolution data" icermektedir. Kodek kutuphanesi bu veriyi cozumlerken, bellekte tahsis edilen alani asar ve bitisik bellek bolgelerini uzerine yazar.
*   Android'in medya islemci altyapisi (MediaCodec), gelen ses dosyalarini arka planda (otomatik indirme etkinse) islemektedir. Bu nedenle kullanici dosyayi acmasa bile, mesajlasma uygulamasi dosyayi indirip kodek kutuphanesine gonderdiginde zafiyet tetiklenir.
*   **Neden:** Kok neden, Dolby Digital Plus kodek kutuphanesinde evolution data metadata alaninin boyut dogrulamasinin yapilmamasidir. Ek olarak, Android'in medya islemci mimarisinde otomatik medya isleme akisinin guvenlik siniri olmadan calismasi, sifir tiklamali saldiriyi mumkun kilmaktadir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Dolby Digital Plus destekli Android cihaz (OEM ureticiler, Ocak 2026 yamasi uygulanmamis)
*   **2. Normal durum:**
    ```
    1. Kullanici bir WhatsApp mesaji alir (ses dosyasi ekli)
    2. Otomatik medya indirme etkinse, dosya arka planda indirilir
    3. Dolby kodek kutuphanesi ses metadata'sini parse eder
    4. Kullanici dosyayi actiginda ses oynatilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, evolution data alaninda asiri buyuk payload iceren
       ozel bir Dolby Digital Plus (.ec3) ses dosyasi hazirlar
    2. Dosya WhatsApp veya Telegram uzerinden kurbana gonderilir
    3. Otomatik medya indirme dosyayi arka planda indirir
    4. MediaCodec, kodek kutuphanesine dosyayi gonderir
    5. Kodek kutuphanesi evolution data'yi parse ederken heap overflow olusur
    6. Saldirganin payload'u bellek uzerinde calistirilir — RCE elde edilir
    7. Kullanici hicbir etkilesimde bulunmamistir (zero-click)
    ```
*   **4. Analiz:** Zafiyet, medya dosyasinin indirilmesi asamasinda tetiklenir; kullanicinin dosyayi acmasina veya oynatmasina gerek yoktur. Evolution data alani icinde bellege yerlestirilen shellcode, MediaCodec sureci baglaminda calisarak cihaz uzerinde komut yurutme yetkisi kazandirir.
*   **5. Kanit:** Saldirgan, cihaz uzerinde uzaktan kod yurutme yetkisi elde eder. Kamera, mikrofon, dosya sistemi ve ag erisimi saglanabilir. Loglarrda anormal medya isleme hatalari ve beklenmeyen kodek cokmeleri gorunur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Kritik
*   **CVSS Skoru:** 9.8 (tahmini — sifir tiklamali, kimlik dogrulama gerektirmeyen RCE)
*   **Saldiri Yuzeyi:** Internete acik (mesajlasma uygulamalari uzerinden uzaktan tetiklenir)
*   **Karmasiklik:** Dusuk (hazirlanmis ses dosyasinin mesaj ile gonderilmesi yeterli)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Android Ocak 2026 guvenlik guncellemesini tum cihazlara uygulayin. Tum mesajlasma uygulamalarinda otomatik medya indirmeyi devre disi birakin. MDM (Mobile Device Management) ile kurumsal cihazlarda guncellemeyi zorunlu kilin.
*   **Orta vadeli:** OEM ureticilerin guvenlik yamasi dagitim surelerini takip edin ve SLA'lara dahil edin. Kurumsal cihaz politikalarinda yalnizca Ocak 2026+ yama seviyesi olan cihazlara kurumsal kaynaklara erisim izni verin. Medya dosya turlerini ag gecidinde (proxy/firewall) filtreleyin.
*   **Uzun vadeli:** Android Enterprise zorunlu guncelleme politikalarini standartlastirin. Kodek kutuphane guncellemelerini Project Mainline uzerinden modularize edin. Mobil tehdit savunmasi (MTD) cozumlerini kurumsal cihazlara dagitarak anomali tespiti saglayin. Fuzzing tabanli kodek guvenlik denetimleri icin surecc olusturun.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```c
// === GUVENSIZ: Evolution data boyut kontrolu yapilmiyor ===
int parse_evolution_data(uint8_t *buf, size_t buf_len) {
    dolby_evolution_t evo;
    // Sabit boyutlu buffer'a kopyala — overflow riski
    memcpy(evo.data, buf, buf_len); // buf_len kontrolsuz!
    return decode_evolution_params(&evo);
}
```

**Guvenli kod:**
```c
// === GUVENLI: Boyut siniri ve dogrulama eklenmis ===
#define MAX_EVOLUTION_DATA_SIZE 4096

int parse_evolution_data(uint8_t *buf, size_t buf_len) {
    dolby_evolution_t evo;

    // 1. Boyut dogrulamasi
    if (buf_len > MAX_EVOLUTION_DATA_SIZE) {
        ALOGE("Evolution data boyutu siniri asildi: %zu > %d",
              buf_len, MAX_EVOLUTION_DATA_SIZE);
        return -EINVAL;
    }

    // 2. Guvenli kopyalama
    memset(&evo, 0, sizeof(evo));
    memcpy(evo.data, buf, buf_len);
    evo.data_len = buf_len;

    return decode_evolution_params(&evo);
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Android cihaz envanteri cikar ve Dolby kodek versiyonlarini kontrol et
*   [ ] Kurumsal cihazlarin Android guvenlik yama seviyelerini tespit et
*   [ ] Mesajlasma uygulamalarindaki otomatik medya indirme ayarlarini gozden gecir
*   [ ] OEM ureticilerin Ocak 2026 yamasi dagitim takvimlerini kontrol et

**Duzeltme**
*   [ ] Android Ocak 2026 guvenlik guncellemesini (2026-01-05) tum cihazlara uygula
*   [ ] Mesajlasma uygulamalarinda otomatik medya indirmeyi devre disi birak
*   [ ] MDM ile guncelleme uyumlulugunu ve dagitimini zorunlu kil
*   [ ] Kurumsal proxy/firewall'da .ec3 ve supheli ses dosyalarini filtrele

**Dogrulama**
*   [ ] Guncelleme sonrasi zarali ses dosyasiyla zero-click testini tekrarla
*   [ ] Kodek guvenlik yamasinin basariyla uygulandigini dogrula (adb shell getprop)
*   [ ] MTD cozumlerinin anomali tespitini basariyla yaptigini dogrula
*   [ ] SIEM'de medya islemci hatalarini izleyen kurallari aktive et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Android cihazlardan gelen medya islemci (MediaCodec) cokme loglarini izle
    *   Mesajlasma uygulamalarindan gelen anormal buyuklukte ses dosyasi indirmelerini tespit et
    *   Regex: `(?i)(mediacodec.*crash|dolby.*fatal|evolution.*overflow|codec.*segfault)`
    *   MDM uzerinden yama seviyesi 2026-01-05 altinda kalan cihazlari raporla
*   **Anomali Tespiti:**
    *   Tek kaynaktan kisa surede cok sayida ses dosyasi gonderilmesi
    *   Medya islemci surecinin beklenmeyen alt surecler baslatmasi (shell, curl, wget)
    *   Kodek kutuphanesi cokme sikligi artisi (crash spike)
    *   Cihazdan disari dogru anormal ag trafigi (C2 baglantisi belirtisi)

---

## Notlar
Google kendi Pixel cihazlarini once yamalamistir, ancak Samsung, Xiaomi, Oppo ve diger OEM ureticilerin cihazlari Ocak 2026 ayi boyunca buyuk risk altinda kalmistir. Android ekosistemindeki parca parca yama dagitim modeli, bu tur kritik zafiyetlerin genis pencerede somurulmesine firsat vermektedir. Sifir tiklamali (zero-click) saldiri vektoru, kullanici farkindalik egitimlerinin etkisiz kaldigi senaryolari temsil etmektedir.
