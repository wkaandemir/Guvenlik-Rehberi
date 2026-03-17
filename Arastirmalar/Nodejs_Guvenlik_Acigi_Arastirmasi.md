# **2025 Sonu Node.js ve React Ekosistemi Güvenlik Krizi: React2Shell (CVE-2025-55182) Kapsamlı Tehdit Analizi ve Savunma Mimarisi**

## **Yönetici Özeti**

2025 yılının son çeyreği, özellikle Kasım ve Aralık ayları, modern web uygulama mimarilerinin güvenliği açısından tarihi bir dönüm noktası olarak kayıtlara geçmiştir. Node.js çalışma zamanı üzerinde çalışan ve endüstri standardı haline gelmiş modern web çerçevelerini (frameworks) hedef alan, siber güvenlik literatürüne "React2Shell" olarak giren kritik bir zafiyet zinciri, küresel dijital altyapıyı tehdit etmiştir. Bu rapor, React Server Components (RSC) mimarisindeki güvensiz tersine serileştirme (deserialization) mekanizmasından kaynaklanan ve CVSS 3.1 standardına göre 10.0 (Kritik) tam puan ile derecelendirilen CVE-2025-55182 zafiyetini ve bunun Next.js ekosistemindeki yıkıcı yansıması olan CVE-2025-66478'i tüm teknik, operasyonel ve stratejik boyutlarıyla ele almaktadır.1

Söz konusu zafiyet, saldırganların herhangi bir kimlik doğrulama mekanizmasını aşmasına gerek kalmadan, tek bir manipüle edilmiş HTTP isteği ile sunucu üzerinde uzaktan kod yürütmesine (Unauthenticated Remote Code Execution \- RCE) olanak tanımaktadır.4 Bu durum, sunucunun dosya sistemine tam erişim, veritabanı bağlantılarının ele geçirilmesi ve iç ağlara sızılması gibi felaket senaryolarına yol açmaktadır. Microsoft, Trend Micro, Palo Alto Networks (Unit 42\) ve Google Threat Intelligence gibi küresel siber istihbarat kaynakları, bu zafiyetin Çin ve İran menşeli devlet destekli tehdit aktörleri (APT grupları) ile fidye yazılımı çeteleri tarafından, zafiyetin duyurulmasından sadece saatler sonra aktif olarak sömürüldüğünü doğrulamıştır.2

Rapor, zafiyetin teknik anatomisini, Node.js çekirdek ekibinin güvenlik güncellemelerini Ocak 2026'ya ertelemesiyle oluşan "güvenlik boşluğunu", aktif saldırı kampanyalarının (Secret-Hunter, Emerald, ReactOnMynuts) detaylarını ve Türkiye'deki kurumların uygulaması gereken derinlemesine savunma stratejilerini kapsamlı bir şekilde incelemektedir.

## **1\. Giriş: Modern Web Mimarisi ve Güvenlik Paradigmasının Çöküşü**

### **1.1. İstemci Taraflı Mimariden Sunucu Odaklı Yapıya Dönüş**

Son on yılda web geliştirme dünyası, tarayıcı tabanlı (Client-Side Rendering \- CSR) yaklaşımlardan, performans ve SEO gereksinimleri nedeniyle tekrar sunucu tabanlı (Server-Side Rendering \- SSR) yaklaşımlara doğru evrilmiştir. Bu dönüşümün merkezinde, JavaScript'in sunucu tarafında çalıştırılmasını sağlayan Node.js bulunmaktadır. 2025 yılına gelindiğinde, React ekosistemi bu evrimi bir adım daha ileri taşıyarak "React Server Components" (RSC) mimarisini standart hale getirmiştir. RSC, kullanıcı arayüzü bileşenlerinin sunucuda derlenip render edilmesini ve istemciye sadece gerekli veri "uçuş" (flight) protokolü ile gönderilmesini esas almaktadır.7

Ancak bu mimari değişim, güvenlik sınırlarını bulanıklaştırmıştır. Geleneksel modelde sunucu ve istemci arasında net bir ayrım ve API tabanlı bir iletişim varken, RSC modeli sunucu fonksiyonlarının (Server Functions) istemci tarafından doğrudan çağrılabilirmiş gibi davranmasına olanak tanır. Kasım 2025'te patlak veren kriz, bu yeni iletişim protokolünün (React Flight), güvenilmeyen verilerin işlenmesi (deserialization) sırasında temel güvenlik prensiplerini ihlal ettiğini ortaya koymuştur.4

### **1.2. Kasım-Aralık 2025: Kusursuz Fırtına**

2025 yılının son iki ayı, siber güvenlik ekipleri için "kusursuz fırtına" olarak nitelendirilebilecek bir dizi olaya sahne olmuştur. Bir yandan React ve Next.js gibi popüler çerçevelerde "Zero-Day" (sıfırıncı gün) açıkları ortaya çıkarken, diğer yandan bu çerçevelerin üzerinde çalıştığı temel altyapı olan Node.js platformu kendi güvenlik döngüsünde zorluklar yaşamaktaydı.

Node.js çekirdek ekibi, v25, v24 (LTS), v22 ve v20 sürümleri için planlanan rutin güvenlik güncellemelerini, karşılaşılan teknik zorluklar ve yaklaşan tatil sezonunun yaratacağı operasyonel riskler nedeniyle erteleme kararı almıştır. İlk olarak 15 Aralık 2025 olarak duyurulan güvenlik sürümü, daha sonra 18 Aralık'a ve nihai olarak **7 Ocak 2026** tarihine kaydırılmıştır.8 Bu durum, uygulama katmanında (React/Next.js) yangın varken, altyapı katmanında (Node.js) bilinen zafiyetlerin (örneğin V8 motorundaki HashDoS ve Windows yol geçişi açıkları) bir aydan fazla süreyle yamasız kalması anlamına gelmekteydi. Saldırganlar için bu zaman aralığı, sistemlere sızmak ve kalıcılık sağlamak için eşsiz bir fırsat penceresi yaratmıştır.

## **2\. Teknik Derinlik: CVE-2025-55182 (React2Shell) Anatomisi**

### **2.1. Zafiyetin Kök Nedeni: React Flight Protokolü ve Güvensiz Deserialization**

CVE-2025-55182, temelinde bir "Güvensiz Tersine Serileştirme" (Insecure Deserialization) zafiyetidir. Ancak bu zafiyeti klasik deserialization açıklarından ayıran unsur, React'in "Flight" protokolünün karmaşık yapısı ve JavaScript motorunun (V8) çalışma prensiplerinin kötüye kullanılmasıdır.

React Server Components, sunucu ve istemci arasında veri taşımak için JSON'dan daha yetenekli, ancak daha karmaşık bir serileştirme formatı kullanır. Bu format, bileşen ağacını (component tree), veri referanslarını ve sunucu aksiyonlarını (Server Actions) kodlar. Tarayıcı, bir formu gönderdiğinde veya bir sunucu fonksiyonunu çağırdığında, verileri numaralandırılmış parçalar (chunks) halinde paketler. Örneğin, 0 numaralı parça bir veri yapısını tanımlarken, 1 numaralı parça bu yapıya referans verebilir. React Flight protokolü, $X gibi özel önekler kullanarak farklı veri tiplerini ve referansları işler.4

Zafiyet, sunucunun bu gelen veri akışını çözümlerken (decoding), verinin güvenilirliğini doğrulamadan doğrudan bellek içi nesnelere dönüştürmesinden kaynaklanır. Saldırganlar, bu mekanizmayı manipüle ederek, sunucunun beklediği veri tipleri yerine, JavaScript motorunun çalışma zamanında kod yürütmesini tetikleyecek özel hazırlanmış nesneler (crafted objects) gönderebilmektedir.1

### **2.2. Dört Aşamalı Saldırı Zinciri**

Trend Micro ve Wiz Research tarafından yapılan analizler, saldırının rastgele bir kod enjeksiyonundan ziyade, JavaScript'in dil özelliklerini sömüren sofistike bir zincirleme reaksiyon olduğunu göstermektedir 5:

1. Kendi Kendine Referans Döngüsü (Self-Reference Loop Creation):  
   Saldırgan, gönderdiği payload içinde, sunucu tarafındaki deserialization işlemi sırasında sonsuz veya karmaşık bir döngü oluşturacak şekilde birbirine referans veren nesneler tanımlar. Bu, ayrıştırıcının (parser) durumunu manipüle etmek için ilk adımdır.  
2. Ördek Tiplemesi (Duck Typing) İstismarı:  
   JavaScript dinamik tipli bir dildir ve "bir nesne ördek gibi yürüyorsa ve ördek gibi vaklıyorsa, o bir ördektir" prensibine (Duck Typing) dayanır. Saldırganlar, React'in iç mekanizmalarının beklediği metotlara (örneğin .then() veya .toJSON()) sahipmiş gibi görünen sahte nesneler oluşturur. Sunucu bu nesneleri işlerken, saldırganın tanımladığı özellikleri tetikler.  
3. Kötücül Veri Enjeksiyonu (Malicious Data Injection):  
   Başlatma (initialization) aşamasında, saldırganın hazırladığı zararlı veri (örneğin bir shell komutu veya zararlı bir modül çağrısı), sunucunun belleğine yasal bir veriymiş gibi enjekte edilir.  
4. Keyfi Kod Yürütme (Remote Code Execution \- RCE):  
   Son aşamada, manipüle edilen nesneler "Blob Handler" veya benzeri bir dahili mekanizma üzerinden işlendiğinde, enjekte edilen veri çalıştırılabilir kod olarak yorumlanır. Node.js ortamında bu genellikle child\_process.exec veya vm.runInNewContext gibi fonksiyonların tetiklenmesiyle sonuçlanır ve saldırgan sunucu üzerinde komut çalıştırma yetkisi kazanır.7

### **2.3. CVSS 10.0: Neden "Maksimum" Kritiklik?**

Bu zafiyetin CVSS v3.1 skorunun **10.0** olması, tehdidin ciddiyetini matematiksel olarak kanıtlamaktadır. Aşağıdaki tablo, bu puanlamanın bileşenlerini detaylandırmaktadır:

| Metrik | Değer | Açıklama |
| :---- | :---- | :---- |
| **Saldırı Vektörü (AV)** | **Network** | Saldırı internet üzerinden, uzaktan gerçekleştirilebilir. Fiziksel erişim gerekmez. |
| **Saldırı Karmaşıklığı (AC)** | **Low** | Saldırı için özel bir zamanlama (race condition) veya karmaşık bir ön hazırlık gerekmez. Tek bir HTTP isteği yeterlidir. |
| **Ayrıcalık Gereksinimi (PR)** | **None** | Saldırganın hedef sistemde herhangi bir kullanıcı hesabına veya yetkiye ihtiyacı yoktur (Unauthenticated). |
| **Kullanıcı Etkileşimi (UI)** | **None** | Kurbanın (yönetici veya kullanıcı) herhangi bir linke tıklaması veya dosya açması gerekmez. |
| **Gizlilik Etkisi (C)** | **High** | Sunucudaki tüm verilere (veritabanı şifreleri, kaynak kod, kullanıcı verileri) erişilebilir. |
| **Bütünlük Etkisi (I)** | **High** | Sunucudaki veriler değiştirilebilir, silinebilir veya yeni zararlı dosyalar eklenebilir. |
| **Erişilebilirlik Etkisi (A)** | **High** | Sunucu çökertilebilir, veriler şifrelenebilir (fidye yazılımı) veya servis dışı bırakılabilir. |

## **3\. Ekosistemdeki Yansımalar: Next.js ve CVE-2025-66478 Krizi**

### **3.1. Upstream (React) ve Downstream (Next.js) İlişkisi**

Node.js ekosisteminde kütüphaneler birbirine sıkı sıkıya bağlıdır. React, Next.js gibi "meta-framework"lerin temelini oluşturur. React ekibinin CVE-2025-55182 kodlu zafiyeti duyurması, bu kütüphaneyi kullanan tüm çerçeveleri otomatik olarak risk altına sokmuştur. Next.js ekibi, kendi kullanıcı tabanını izlemek ve özelleştirilmiş yamalar sunmak amacıyla bu zafiyeti **CVE-2025-66478** koduyla takip etmiştir.10

Bu ayrım önemlidir çünkü birçok geliştirici sadece "Next.js" kullandığını düşünerek React güncellemelerini göz ardı edebilir. Oysa Next.js'in "App Router" mimarisi, tamamen React Server Components üzerine inşa edilmiştir. Bu mimariyi kullanan uygulamalar, zafiyetin kalbini oluşturan güvensiz Flight protokolünü varsayılan olarak kullanmaktadır.

### **3.2. Varsayılan Konfigürasyon Tehlikesi**

Upwind ve Wiz.io tarafından yapılan araştırmalar, durumu daha da vahim hale getiren bir gerçeği ortaya çıkarmıştır: **Varsayılan konfigürasyonlar savunmasızdır.** create-next-app komutu ile oluşturulan, hiçbir özel kod eklenmemiş, sadece "Merhaba Dünya" diyen bir Next.js uygulaması dahi, eğer App Router kullanıyorsa ve internete açıksa, RCE saldırısına maruz kalabilir.3

Bulut ortamlarındaki tarama verilerine göre:

* Next.js veya React kullanan bulut ortamlarının **%39**'u savunmasız sürümleri barındırmaktadır.  
* Kamuya açık (publicly exposed) Next.js uygulamalarının **%44**'ü bu zafiyetten etkilenmektedir.4

Bu istatistikler, zafiyetin sadece teorik bir risk olmadığını, internetin önemli bir bölümünü oluşturan modern web uygulamalarını doğrudan hedef aldığını göstermektedir.

### **3.3. Etkilenen Sürümler ve Bileşenler**

Sistem yöneticilerinin ve geliştiricilerin kontrol etmesi gereken spesifik sürümler şunlardır 5:

* **React:**  
  * Etkilenenler: 19.0.0, 19.1.0, 19.1.1, 19.2.0  
  * *Not: React 18.x ve öncesi, Server Components özelliğini varsayılan olarak barındırmadığı için bu spesifik RCE'den etkilenmez, ancak React 19'a geçiş yapanlar risk altındadır.*  
* **Next.js:**  
  * Etkilenenler: 15.0.0 \- 15.0.4, 15.1.0 \- 15.1.8, 15.2.x \- 15.5.6, 16.0.0 \- 16.0.6.  
  * Canary Sürümleri: 14.3.0-canary.77 ve sonrası tüm Canary sürümleri.  
  * *Not: Next.js 13.x, 14.x (Stable) ve "Pages Router" kullanan uygulamalar bu zafiyetten etkilenmemektedir.*

## **4\. Node.js Çekirdek Güvenlik Durumu (Kasım-Aralık 2025\)**

Kullanıcının sorgusu özellikle "Node.js ile sunucu kontrolünün ele geçirilmesi" konusuna odaklanmaktadır. Burada kritik bir teknik ayrım yapılmalıdır: **React2Shell zafiyeti, Node.js'in çekirdek kodunda (runtime) değil, Node.js üzerinde çalışan React kütüphanesindedir.** Ancak, saldırının başarılı olması durumunda ele geçirilen şey **Node.js işlemidir (process)**. Dolayısıyla, Node.js'in kendi güvenlik durumu, saldırı sonrası etkilerin sınırlandırılması açısından hayati önem taşır.

### **4.1. Ertelenen Kritik Güncellemeler**

React2Shell krizinin patlak verdiği günlerde, Node.js projesi kendi içinde zorlu bir süreçten geçmekteydi. Node.js Teknik Yönlendirme Komitesi (TSC), v25 (Current), v24 (LTS), v22 (LTS) ve v20 (LTS) sürümleri için planlanan güvenlik sürümünü erteleme kararı almıştır.8

* **Orijinal Tarih:** 15 Aralık 2025  
* **İlk Erteleme:** 18 Aralık 2025  
* **Nihai Tarih:** 7 Ocak 2026

Bu ertelemenin gerekçesi, "özellikle zorlu bir yamanın hazırlanması" ve "tatil sezonunda ekosistemi bozma riskinden kaçınılması" olarak açıklanmıştır. Ancak bu karar, sistem yöneticilerini zor durumda bırakmıştır. Çünkü Ocak 2026'da yayınlanacak yamalar şunları içermektedir 9:

1. **3 Adet Yüksek Seviye (High Severity) Zafiyet:** Bunların arasında V8 motorundaki "HashDoS" zafiyeti (CVE-2025-27209) ve Windows cihaz adlarıyla (CON, PRN, AUX) ilgili yol geçişi (Path Traversal) zafiyeti (CVE-2025-27210) bulunmaktadır.  
2. **1 Orta ve 1 Düşük Seviye Zafiyet.**

React2Shell saldırısına uğrayan bir sunucu, eğer Node.js seviyesindeki bu açıklar da yamalanmamışsa, saldırganın sistem içinde daha kolay hareket etmesine (lateral movement) veya sunucuyu tamamen çökertmesine (DoS) olanak tanıyabilir. Özellikle Windows sunucularda çalışan Node.js uygulamaları için CVE-2025-27210, React2Shell ile birleştiğinde dosya sistemi üzerinde tam hakimiyet kurulmasını kolaylaştırabilir.

## **5\. Tehdit İstihbaratı ve Aktif Saldırı Analizi**

React2Shell zafiyetinin 29 Kasım 2025'te raporlanıp, 3 Aralık'ta duyurulmasının hemen ardından, siber suç dünyası ve devlet destekli aktörler harekete geçmiştir.

### **5.1. Saldırı Zaman Çizelgesi ve Adaptasyon Hızı**

* **29 Kasım 2025:** Lachlan Davidson zafiyeti React ekibine özel olarak bildirdi.  
* **3 Aralık 2025:** React ve Next.js güvenlik bültenleri yayınlandı.  
* **4 Aralık 2025:** İlk kavram kanıtlama (PoC) kodları GitHub ve dark web forumlarında dolaşmaya başladı.  
* **5 Aralık 2025:** Trend Micro, "In-the-Wild" (gerçek dünyada) sömürü girişimlerinin başladığını tespit etti.7  
* **8 Aralık 2025:** Palo Alto Networks Unit 42, saldırıların otomatize edildiğini ve botnetlere entegre edildiğini raporladı.11  
* **12-15 Aralık 2025:** Google ve Microsoft, devlet destekli aktörlerin (Çin, İran) devreye girdiğini doğruladı.5

Bu hızlı adaptasyon süreci, modern web zafiyetlerinin "Time-to-Exploit" (Sömürüye Kadar Geçen Süre) aralığının saatler mertebesine indiğini göstermektedir.

### **5.2. Tanımlanan Tehdit Aktörleri ve Kampanyalar**

#### **5.2.1. Çin Menşeli APT Grupları (Earth Lamia / UNC5454)**

Google Threat Intelligence ve AWS, Çin bağlantılı **Earth Lamia** (diğer adıyla UNC5454) ve **Jackpot Panda** gruplarının bu zafiyeti hedeflediğini belirtmiştir.6 Bu gruplar genellikle siber casusluk, fikri mülkiyet hırsızlığı ve stratejik erişim sağlama motivasyonuyla hareket etmektedir. React2Shell üzerinden sistemlere sızarak, kurumsal ağların derinliklerine ilerlemek için "köprübaşı" (beachhead) oluşturmaktadırlar.

#### **5.2.2. Finansal Motivasyonlu Saldırılar (Coin Miners & Ransomware)**

En yaygın saldırı türü, ele geçirilen sunucuların işlemci gücünü sömürmeyi amaçlayan kripto para madencileridir.

* **Kampanya "Emerald":** usbx\[.\]me alan adını kullanarak, Node.js'in child\_process modülü üzerinden wget komutuyla zararlı bot indirilmektedir.7  
* **Kampanya "Sex.sh":** Adından da anlaşılacağı üzere basit ama etkili bir kabuk betiği (shell script) indirilmekte, bu betik XMRig madencisini kurmakta ve system-update-service adıyla sahte bir systemd servisi oluşturarak kalıcılık sağlamaktadır.6

#### **5.2.3. Botnetler (Mirai ve Nezha)**

Trend Micro, "ReactOnMynuts" adını verdikleri bir Mirai varyantının dağıtıldığını tespit etmiştir. Bu botnet, ele geçirilen sunucuları DDoS saldırıları için kullanmaktadır. Ayrıca "Nezha" ve "Sliver" gibi C2 (Komuta Kontrol) araçları da gözlemlenmiştir.7

### **5.3. Saldırı Teknikleri ve Yapay Zeka Kullanımı**

Saldırganların teknikleri oldukça çeşitlidir ve tespit edilmemek için gelişmiş yöntemler kullanmaktadırlar:

* **AI Destekli Kodlar:** Ele geçirilen bazı zararlı yazılımlarda (malware), yapay zeka tarafından üretildiğine dair güçlü kanıtlar (örneğin, gereksiz derecede detaylı Çince yorum satırları, mantıksal açıklamalar ve tamamlanmamış yer tutucular) bulunmuştur. Bu, saldırganların zafiyet sömürü araçlarını geliştirmek için LLM (Büyük Dil Modelleri) kullandığını göstermektedir.7  
* **Base64 Kodlama:** WAF ve IDS (Saldırı Tespit Sistemleri) imzalarından kaçmak için, gönderilen payload'lar ve komutlar Base64 ile kodlanarak gizlenmektedir.11  
* **Bellek İçi Çalıştırma (Fileless Execution):** Bazı saldırılar, diske dosya yazmadan doğrudan bellek üzerinde çalışarak adli bilişim (forensics) analizini zorlaştırmaktadır.

## **6\. Etki Analizi ve Türkiye Perspektifi**

### **6.1. Operasyonel ve Ticari Etkiler**

Bir sunucunun React2Shell ile ele geçirilmesinin etkileri yıkıcıdır:

1. **Tam Sistem Kontrolü:** Node.js süreci genellikle sunucuda geniş yetkilere sahiptir. Saldırgan, işletim sistemi komutlarını çalıştırabilir, dosya okuyup yazabilir.  
2. **Veri Sızıntısı:** .env dosyaları, veritabanı bağlantı dizeleri, AWS/Azure erişim anahtarları çalınabilir. Next.js'in "Server Functions" yapısı, kaynak kodun ifşa edilmesine de (CVE-2025-55183) yol açabilmektedir.13  
3. **Tedarik Zinciri Saldırısı:** Ele geçirilen sunucu, müşterilere zararlı JavaScript kodları dağıtmak için kullanılabilir (Watering Hole saldırısı).

### **6.2. Türkiye'deki Durum**

Türkiye'deki dijital ekosistem, özellikle e-ticaret, bankacılık ve startup dünyasında Node.js ve Next.js'i yoğun olarak kullanmaktadır.

* **USOM Bildirimi:** Ulusal Siber Olaylara Müdahale Merkezi (USOM), **TR-25-0428** numaralı bildirimiyle zafiyetin ciddiyetine dikkat çekmiş ve "JavaScript React kütüphanesindeki bu zafiyetin siber saldırganlar tarafından kullanılmasının ihtimal dahilinde olduğunu" belirtmiştir.14  
* **Sektörel Risk:** STM'nin (Savunma Teknolojileri Mühendislik ve Ticaret A.Ş.) siber tehdit raporları, Türkiye'nin siber saldırılarda hedef alınan ülkeler arasında üst sıralarda olduğunu vurgulamaktadır. Özellikle finans ve kamu sektöründe kullanılan modern web arayüzlerinin bu tür framework zafiyetlerine karşı taranması kritik önem taşımaktadır.15

## **7\. Azaltma, İyileştirme ve Savunma Stratejileri**

Bu kritik tehdide karşı, "Savunma Derinliği" (Defense in Depth) prensibine dayalı çok katmanlı bir strateji uygulanmalıdır.

### **7.1. Öncelikli Aksiyon: Yama Yönetimi (Patching)**

Tek ve kesin çözüm, etkilenen kütüphanelerin güncellenmesidir. Aşağıdaki tablo, geçilmesi gereken minimum güvenli sürümleri özetlemektedir:

| Bileşen | Etkilenen Sürüm Aralığı | Minimum Güvenli Sürüm |
| :---- | :---- | :---- |
| **React** | 19.0.0 \- 19.2.0 | **19.0.1, 19.1.2, 19.2.1+** |
| **Next.js (v15)** | 15.0.0 \- 15.0.4 | **15.0.5+** |
| **Next.js (v15.1)** | 15.1.0 \- 15.1.8 | **15.1.9+** |
| **Next.js (v15.2)** | 15.2.0 \- 15.2.5 | **15.2.6+** |
| **Next.js (v16)** | 16.0.0 \- 16.0.6 | **16.0.7+** |
| **Next.js (Canary)** | 14.3.0-canary.77+ | **15.6.0-canary.58+, 16.1.0-canary.12+** |

Next.js kullanıcıları için otomatik güncelleme aracı yayınlanmıştır:

Bash

npx fix-react2shell-next

Bu komut, projeyi tarayarak gerekli güncellemeleri otomatik olarak uygular.10 Ancak manuel kontrol (package.json incelemesi) her zaman önerilir.

### **7.2. İkincil Savunma: WAF ve Ağ Güvenliği**

Yama uygulanamayan durumlar (Legacy sistemler, test süreçleri) için Web Uygulama Güvenlik Duvarı (WAF) kuralları hayat kurtarıcıdır.

* **AWS WAF:** AWS kullanıcıları için AWSManagedRulesKnownBadInputsRuleSet (sürüm 1.24+) otomatik olarak güncellenmiş ve React2Shell imzalarını içermektedir.2  
* **Özel Kural Setleri:** Gelen POST isteklerinde, React Flight protokolüne özgü $X gibi pattern'leri ve aşırı uzun, karmaşık JSON yapılarını engelleyen kurallar tanımlanmalıdır.

### **7.3. Sırların Döndürülmesi (Secret Rotation)**

Bu adım genellikle göz ardı edilir ancak **kritiktir**. Eğer bir uygulama 4 Aralık 2025 tarihinden önce internete açıksa ve yamalanmamışsa, saldırganların ortam değişkenlerini (ENV) okumuş olması kuvvetle muhtemeldir. Bu nedenle:

* Veritabanı şifreleri,  
* AWS/Cloud API anahtarları,  
* JWT imzalama anahtarları (Secrets),  
* Ödeme sistemi (Stripe, Iyzico vb.) API anahtarları **derhal değiştirilmelidir**.10

### **7.4. Tespit ve İzleme (Detection Engineering)**

Sistem yöneticileri, SIEM ve EDR araçlarında aşağıdaki göstergeleri (IoC) aramalıdır:

* **Süreç Ağacı Anomalileri:** node işleminin altında çalışan şüpheli curl, wget, bash, powershell süreçleri.5  
* **Ağ Bağlantıları:** Bilinmeyen dış IP adreslerine (özellikle standart dışı portlara) yapılan bağlantılar.  
* **Dosya Sistemi Değişiklikleri:** /tmp, /var/tmp veya uygulamanın public dizininde oluşturulan yeni .js, .sh veya binary dosyalar.

## **8\. Sonuç**

React2Shell (CVE-2025-55182) ve Next.js (CVE-2025-66478) zafiyetleri, 2025 yılının sonunda siber güvenlik dünyasına sert bir hatırlatma yapmıştır: Uygulama katmanındaki modernizasyon ve karmaşıklık, yeni ve yıkıcı saldırı yüzeylerini de beraberinde getirmektedir. Node.js'in sağladığı güçlü sunucu yetenekleri, güvensiz çerçeve tasarımlarıyla birleştiğinde, saldırganlara sunucunun anahtarını altın tepside sunabilmektedir.

Node.js çekirdek ekibinin güvenlik güncellemelerini Ocak 2026'ya ertelemesi, altyapı ve uygulama güvenliği arasındaki senkronizasyonun önemini göstermiştir. Türkiye'deki ve dünyadaki kurumlar için bu olay, sadece bir "yama geçme" operasyonu değil, aynı zamanda yazılım tedarik zinciri güvenliği, varsayılan konfigürasyonların sıkılaştırılması ve sürekli tehdit izleme yeteneklerinin gözden geçirilmesi için bir uyarı niteliğindedir. React2Shell, Log4Shell gibi, etkisi uzun yıllar tartışılacak ve siber güvenlik eğitimlerinde vaka analizi olarak okutulacak bir kriz olarak tarihteki yerini almıştır.

#### **Alıntılanan çalışmalar**

1. Critical Security Vulnerability in React Server Components, erişim tarihi Aralık 20, 2025, [https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components](https://react.dev/blog/2025/12/03/critical-security-vulnerability-in-react-server-components)  
2. China-nexus cyber threat groups rapidly exploit React2Shell vulnerability (CVE-2025-55182) | AWS Security Blog, erişim tarihi Aralık 20, 2025, [https://aws.amazon.com/blogs/security/china-nexus-cyber-threat-groups-rapidly-exploit-react2shell-vulnerability-cve-2025-55182/](https://aws.amazon.com/blogs/security/china-nexus-cyber-threat-groups-rapidly-exploit-react2shell-vulnerability-cve-2025-55182/)  
3. Critical Security Alert: Unauthenticated RCE in React & Next.js \- Upwind, erişim tarihi Aralık 20, 2025, [https://www.upwind.io/feed/critical-security-alert-unauthenticated-rce-in-react-next-js-cve-2025-55182-cve-2025-66478](https://www.upwind.io/feed/critical-security-alert-unauthenticated-rce-in-react-next-js-cve-2025-55182-cve-2025-66478)  
4. React2Shell (CVE-2025-55182): Critical React Vulnerability | Wiz Blog, erişim tarihi Aralık 20, 2025, [https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182](https://www.wiz.io/blog/critical-vulnerability-in-react-cve-2025-55182)  
5. Defending against the CVE-2025-55182 (React2Shell) vulnerability ..., erişim tarihi Aralık 20, 2025, [https://www.microsoft.com/en-us/security/blog/2025/12/15/defending-against-the-cve-2025-55182-react2shell-vulnerability-in-react-server-components/](https://www.microsoft.com/en-us/security/blog/2025/12/15/defending-against-the-cve-2025-55182-react2shell-vulnerability-in-react-server-components/)  
6. Multiple Threat Actors Exploit React2Shell (CVE-2025-55182) | Google Cloud Blog, erişim tarihi Aralık 20, 2025, [https://cloud.google.com/blog/topics/threat-intelligence/threat-actors-exploit-react2shell-cve-2025-55182](https://cloud.google.com/blog/topics/threat-intelligence/threat-actors-exploit-react2shell-cve-2025-55182)  
7. CVE-2025-55182: React2Shell Analysis, Proof-of-Concept Chaos ..., erişim tarihi Aralık 20, 2025, [https://www.trendmicro.com/en\_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html](https://www.trendmicro.com/en_us/research/25/l/CVE-2025-55182-analysis-poc-itw.html)  
8. Blog \- Node.js, erişim tarihi Aralık 20, 2025, [https://nodejs.org/en/blog/year-2025](https://nodejs.org/en/blog/year-2025)  
9. Wednesday, January 7, 2026 Security Releases \- Node.js, erişim tarihi Aralık 20, 2025, [https://nodejs.org/en/blog/vulnerability/december-2025-security-releases](https://nodejs.org/en/blog/vulnerability/december-2025-security-releases)  
10. Security Advisory: CVE-2025-66478 \- Next.js, erişim tarihi Aralık 20, 2025, [https://nextjs.org/blog/CVE-2025-66478](https://nextjs.org/blog/CVE-2025-66478)  
11. Exploitation of Critical Vulnerability in React Server Components (Updated December 12), erişim tarihi Aralık 20, 2025, [https://unit42.paloaltonetworks.com/cve-2025-55182-react-and-cve-2025-66478-next/](https://unit42.paloaltonetworks.com/cve-2025-55182-react-and-cve-2025-66478-next/)  
12. Tuesday, July 15, 2025 Security Releases \- Node.js, erişim tarihi Aralık 20, 2025, [https://nodejs.org/en/blog/vulnerability/july-2025-security-releases](https://nodejs.org/en/blog/vulnerability/july-2025-security-releases)  
13. Next.js Security Update: December 11, 2025, erişim tarihi Aralık 20, 2025, [https://nextjs.org/blog/security-update-2025-12-11](https://nextjs.org/blog/security-update-2025-12-11)  
14. Ulusal Siber Olaylara Müdahale Merkezi \- USOM, erişim tarihi Aralık 20, 2025, [https://www.usom.gov.tr/bildirim/tr-25-0428](https://www.usom.gov.tr/bildirim/tr-25-0428)  
15. STM, 2025'in İlk Siber Raporunu Yayımladı, erişim tarihi Aralık 20, 2025, [https://www.stm.com.tr/tr/basin-bultenleri/stm-2025-ilk-siber-raporunu-yayimladi](https://www.stm.com.tr/tr/basin-bultenleri/stm-2025-ilk-siber-raporunu-yayimladi)  
16. STM, Siber Tehdit Durum Raporu'nu yayımladı \- Anadolu Ajansı, erişim tarihi Aralık 20, 2025, [https://www.aa.com.tr/tr/isdunyasi/bilisim/stm-siber-tehdit-durum-raporunu-yayimladi/685288](https://www.aa.com.tr/tr/isdunyasi/bilisim/stm-siber-tehdit-durum-raporunu-yayimladi/685288)