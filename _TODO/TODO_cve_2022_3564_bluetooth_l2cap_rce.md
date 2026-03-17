# TODO: CVE-2022-3564 — Bluetooth L2CAP RCE

## Meta Bilgiler
| Alan | Deger |
|------|-------|
| **Durum** | ACIK |
| **Tanimlayici** | CVE-2022-3564 |
| **Kritiklik** | Yuksek |
| **Kategori Klasoru** | Isletim_Sistemi/ |
| **Kaynak Arastirma** | [Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md](../Arastirmalar/Ocak_2026_Guvenlik_Aciklari_Arastirmasi.md) |
| **Tarih** | 2026-02-02 |
| **Mevcut Dokuman** | Silindi — icerik bu TODO dosyasinda | |

---

### 1. Ozet ve etki
*   **Yonetici Ozeti:** CVE-2022-3564, Linux cekirdegindeki Bluetooth L2CAP (Logical Link Control and Adaptation Protocol) altsistemine ait bir use-after-free (serbest birakildiktan sonra kullanma) zafiyetidir. Ilk olarak 2022'de raporlanmis olmasina ragmen, Ocak 2026 Linux yama turunda bu acik yeniden somurulur hale geldiginden tekrar guncellenmsitir. Fiziksel yakinliktaki bir saldirgan, Bluetooth agi uzerinden ozel hazirlanmis L2CAP paketleri gondererek cekirdek (kernel) baglaminda kod yurutebilir. Bu durum, hedef cihaz uzerinde tam SYSTEM yetkileriyle erisim saglanmasina yol acar. Ozellikle IoT cihazlari, gomulu Linux sistemleri ve Bluetooth destekli sunucular icin ciddi risk olusturmaktadir.
*   **Etkilenen bilesenler:** Linux cekirdegi (net/bluetooth/l2cap_core.c), Bluetooth L2CAP altsistemi, BlueZ Bluetooth protokol stack'i, Bluetooth destekli tum Linux cihazlari (sunucular, IoT, gomulu sistemler, Android cihazlar)

---

### 2. Teknik detay (nasil calisiyor)
*   Linux cekirdeginin L2CAP altisteminde, bir Bluetooth baglantisi sirasinda `l2cap_reassemble_sdu()` fonksiyonu, gelen L2CAP fragmentlerini yeniden birlestirir. Bu fonksiyondaki bir yarisch durumu (race condition), bir kanalin kapatilmasi sirasinda bellekteki SDU (Service Data Unit) tampon belleginin serbest birakilmasindan sonra tekrar erisilebilmesine neden olur.
*   Saldirgan, Bluetooth eslesmesi (pairing) islemi sirasinda veya baglanti kurulurkene zamanlama manipulasyonu ile bu yarisch durumunu tetikler. Serbest birakilmis bellek blogu, saldirganin kontrol ettigi veriyle doldurulabilir ve bu veri daha sonra cekirdek tarafindan islenir.
*   Use-after-free durumu, saldirganin cekirdek bellegi uzerinde okuma/yazma islemleri yapmasina ve sonucta cekirdek baglaminda kod yurutmesine (kernel RCE) yol acar.
*   **Neden:** Kok neden, L2CAP kanal kapatma ve SDU yeniden birlestirme islemlerinin esitlenmemis (unsynchronized) calismasidir. Kanal kapatilirken SDU tampon belleginin hala referans edilebilir durumda kalmasi, klasik bir CWE-416 (Use After Free) ornegidir. 2022'deki yamalar yeterince kapsamli olmamis ve belirli kernel yapilandirmalarinda yeniden somurulme potansiyeli ortaya cikmistir.

---

### 3. Adim adim istismar (PoC — kavramsal)
> **Uyari:** Bu bolum yalnizca egitim ve savunma amaclidir.
*   **1. Hedef:** Bluetooth servisi etkin olan Linux sunucu veya IoT cihazi
*   **2. Normal durum:**
    ```
    1. Bluetooth cihazi kesif modundadir (discoverable)
    2. Istemci L2CAP kanali acar ve veri aktarimi yapar
    3. Islem tamamlaninca kanal guvenli sekilde kapatilir
    4. SDU tampon bellegi serbest birakilir
    ```
*   **3. Manipule edilmis durum (PoC payload):**
    ```
    1. Saldirgan, Bluetooth menzilindeki hedef cihazi tarar
    2. L2CAP kanal acma istegi gonderir
    3. Kanal acildiktan hemen sonra, fragmentli SDU paketleri gonderir
    4. Es zamanli olarak kanal kapatma istegi gonderir (race condition)
    5. l2cap_reassemble_sdu() serbest birakilmis SDU tamponuna erisir (UAF)
    6. Saldirgan, serbest birakilmis bellek bolgesini kontrol ettigi
       payload ile doldurur (heap spray teknigi)
    7. Cekirdek, zehirli bellek verisini isler — RCE elde edilir
    ```
*   **4. Analiz:** Yarisch durumu (race condition), kanal kapatma ve SDU yeniden birlestirme arasindaki zamanlama boslugundan yararlanir. Saldirganin fiziksel yakinlikta olmasi gerekir (Bluetooth menzili ~10-100m), ancak guclendirmis antenlere yuksek menzilli saldirilar da mumkundur. Basarili istismar durumunda saldirgan, cekirdek baglaminda calisir — bu da SELinux dahil tum kullanici alani guvenlik onlemlerini atlatabilecegi anlamina gelir.
*   **5. Kanit:** Saldirgan, cekirdek yetkilerine sahip olarak hedef sistemde tam kontrol saglar. Rootkit yukleme, trafik dinleme, veri sizdirma ve kalici arka kapi olusturma mumkundur. Loglarda beklenmeyen Bluetooth baglanti girisimleri ve cekirdek hatasi (kernel oops/panic) kayitlari gorunur.

---

### 4. Risk degerlendirmesi
*   **Kritiklik:** Yuksek
*   **CVSS Skoru:** 7.1 (yakin erisim gereksinimi nedeniyle Network degil Adjacent Vector)
*   **Saldiri Yuzeyi:** Yakin cevre (Bluetooth menzili — fiziksel yakinlik gerektirir)
*   **Karmasiklik:** Yuksek (race condition zamanlama manipulasyonu gerektirir)

---

### 5. Kalici cozumler ve oneriler
*   **Kisa vadeli (Acil):** Linux cekirdegini Ocak 2026 guvenlik yamalarinni iceren surume guncelleyin. Bluetooth servisine ihtiyac duymayan sistemlerde servisi ve kernel modulunu devre disi birakin. Bluetooth cihazlarinin "kesif modu"nu (discoverable) kapalin.
*   **Orta vadeli:** Bluetooth etkin sistemlerin envanterini cikartin ve gereksiz Bluetooth servislerini kapatinin. IoT cihazlarinda firmware guncellemelerini planlayin. Bluetooth trafigini SIEM'de izlemeye alin (hci log kaynaklari). Cekirdek parametreleriyle L2CAP bilesenini sinirlayim.
*   **Uzun vadeli:** Bluetooth protokol stack'inin Rust ile yeniden yazilmasi gibi bellek guvenligi projelerini takip edin (BlueZ modernizasyonu). IoT cihazlarda otomatik firmware guncelleme altyapisi kurun. Bluetooth guvenlik denetimleri icin fuzzing test sureclerini olusturun. Fiziksel guvenlik politikalarini Bluetooth menzili risklerini kapsayacak sekilde genisletin.

---

### 6. Ornek duzeltme kodu

**Zafiyetli kod:**
```c
// === GUVENSIZ: L2CAP kanal kapatmada SDU tampon korumasi yok ===
static void l2cap_reassemble_sdu(struct l2cap_chan *chan,
                                  struct sk_buff *skb) {
    // SDU tamponuna erisilir — kanal kapatilmis olabilir!
    if (chan->sdu) {
        skb_copy_bits(skb, 0, skb_put(chan->sdu, skb->len), skb->len);
    }
    // chan->sdu serbest birakilmissa → use-after-free!
}
```

**Guvenli kod:**
```c
// === GUVENLI: Mutex kilidi ve durum kontrolu eklenmis ===
static void l2cap_reassemble_sdu(struct l2cap_chan *chan,
                                  struct sk_buff *skb) {
    /* 1. Kanal kilidini al — race condition onlemi */
    l2cap_chan_lock(chan);

    /* 2. Kanalin hala gecerli ve bagli oldugunu kontrol et */
    if (chan->state != BT_CONNECTED || !chan->sdu) {
        l2cap_chan_unlock(chan);
        kfree_skb(skb);
        return;
    }

    /* 3. Guvenli SDU birlestirme */
    skb_copy_bits(skb, 0, skb_put(chan->sdu, skb->len), skb->len);

    l2cap_chan_unlock(chan);
}
```

---

### 7. Kontrol Listesi (Checklist)
**Hazirlik**
*   [ ] Bluetooth servisi etkin olan tum Linux cihazlarini envanterle
*   [ ] Bluetooth servisinin is gereksinimine yonelik ihtiyac analizini yap
*   [ ] Mevcut cekirdek versiyonlarini tespit et ve yamali surumlerle karsilastir
*   [ ] IoT ve gomulu cihazlarda Bluetooth durumunu kontrol et

**Duzeltme**
*   [ ] Linux cekirdegini Ocak 2026+ guvenlik yamasini iceren surume guncelle
*   [ ] Gereksiz Bluetooth servislerini ve modullerini devre disi birak
*   [ ] Bluetooth cihazlarin kesif modunu (discoverable) kapat
*   [ ] IoT cihazlarinda firmware guncellemelerini uygula

**Dogrulama**
*   [ ] Guncelleme sonrasi L2CAP exploit testini (race condition senaryosu) tekrarla
*   [ ] Bluetooth modulu devre disi birakilmis sistemlerde modulun yuklenemedigini dogrula
*   [ ] SIEM'de Bluetooth baglanti loglarinin izlendigini dogrula
*   [ ] Bluetooth fuzzing taramasi ile ek zafiyetlerin bulunmadigini kontrol et

---

### 8. Izleme ve uyarilar
*   **SIEM/Log Kurallari:**
    *   Bluetooth HCI (Host Controller Interface) loglarinda beklenmeyen baglanti girisimlerini izle
    *   Cekirdek loglarinda (dmesg/journalctl) L2CAP ve Bluetooth ile ilgili hata mesajlarini tespit et
    *   Regex: `(?i)(l2cap.*error|bluetooth.*use.after.free|bt.*oops|hci.*unauthorized)`
    *   Regex: `(?i)(kernel.*panic.*bluetooth|l2cap.*reassemble.*fault)`
*   **Anomali Tespiti:**
    *   Kisa surede cok sayida Bluetooth eslestirme denemesi (brute force belirtisi)
    *   Bilinmeyen MAC adreslerinden gelen Bluetooth baglanti girisimleri
    *   Cekirdek hatasi (kernel oops) sikligi artisi — ozellikle Bluetooth moduluyle iliskili
    *   Bluetooth servisi devre disi birakilmis sistemlerde modulun beklenmedik sekilde yuklenmesi

---

## Notlar
CVE-2022-3564 eski bir zafiyet olmasina ragmen, Ocak 2026 Linux yama turunda yeniden somurulme potansiyeli nedeniyle guncellanmistir. Fiziksel yakinlik gereksinimi saldiri yuzeyini sinirlandirsa da, IoT cihazlarin yayginligi ve halka acik alanlardaki Bluetooth cihazlarin sayisi goz onune alindiginda risk onemlidir. Guclendirilmis Bluetooth antenleri ile menzil 100 metrenin uzerine cikabilir.
