# uk-traffic-accidents-analysis-project
# UK Trafik Kazaları ve Akışı Analizi

## 1. Proje Tanımı ve Amacı

Bu proje, Birleşik Krallık'taki (UK) trafik kazası verileri ile trafik akışı verilerini birleştirerek kazaların nedenlerini, zaman içindeki eğilimlerini ve trafik yoğunluğu ile ilişkilerini incelemeyi amaçlamaktadır. Proje kapsamında aşağıdaki temel sorulara yanıt aranmıştır:

* Trafik akışının değişmesi kazaları nasıl etkiler?
* Kaza oranlarını artıran veya azaltan çevresel ve yol koşulları nelerdir?
* Zaman içinde (yıllık, aylık, günlük) kaza oranlarını tahmin edebilir miyiz?
* Kırsal ve kentsel alanlardaki kaza paternleri nasıl farklılaşır?

## 2. Veri Kaynakları ve Tanımı

Projede iki ana veri seti kullanılmıştır:

### 2.1. UK Trafik Kazaları Veri Seti (Accidents)

* **Kaynak:** [Birleşik Krallık Karayolları İstatistikleri](https://www.gov.uk/government/statistics/road-accidents-and-safety-statistics) (Genellikle DfT - Department for Transport tarafından yayınlanır.)
* **İçerik:** 2005-2014 yılları arasındaki trafik kazalarına ilişkin detaylı bilgiler. Her satır tek bir kazayı temsil eder.
* **Önemli Sütunlar:**
    * `Accident_Index`: Kaza indeksi (Benzersiz ID)
    * `Date`: Kaza tarihi
    * `Time`: Kaza saati
    * `Accident_Severity`: Kazanın şiddeti (1=Ölümcül, 2=Ciddi, 3=Hafif Yaralanmalı)
    * `Location_Easting_OSGR`, `Location_Northing_OSGR`, `Longitude`, `Latitude`: Coğrafi konum bilgileri
    * `Day_of_Week`: Haftanın günü
    * `Weather_Conditions`: Hava koşulları
    * `Road_Surface_Conditions`: Yol yüzey koşulları
    * `Light_Conditions`: Işık koşulları
    * `Road_Type`: Yol tipi
    * `Speed_limit`: Hız limiti
    * `Urban_or_Rural_Area`: Kentsel (1) veya Kırsal (2) alan bilgisi
    * `Number_of_Vehicles`, `Number_of_Casualties`: Kazaya karışan araç ve yaralı sayısı
    * `Year`: Kazanın gerçekleştiği yıl (türetilmiş)

### 2.2. UK Yıllık Ortalama Günlük Trafik Akışı Veri Seti (AADF - Annual Average Daily Flow)

* **Kaynak:** [Birleşik Krallık Karayolları İstatistikleri](https://www.gov.uk/government/statistics/road-traffic-statistics) (DfT)
* **İçerik:** Birleşik Krallık'taki belirli kontrol noktalarından geçen araç sayılarının yıllık ortalama günlük akışları (AADF).
* **Önemli Sütunlar:**
    * `AADFYear`: Trafik akışı verisinin toplandığı yıl
    * `LocalAuthority`: Yerel otorite
    * `RoadCategory`: Yol kategorisi
    * `Easting`, `Northing`, `Lat`, `Lon`: Kontrol noktasının coğrafi konumu
    * `AllMotorVehicles`: Tüm motorlu araçların yıllık ortalama günlük akışı (kullanılan ana metrik)

## 3. Metodoloji

Proje, aşağıdaki adımları içermektedir:

### 3.1. Veri Toplama ve Birleştirme

* 2005-2014 yılları arasındaki üç ayrı `accidents` CSV dosyası tek bir DataFrame'de birleştirildi.
* `ukTrafficAADF` veri seti yüklendi.
* `accidents` veri setinden `Year` sütunu türetildi.
* `accidents` ve `ukTrafficAADF` veri setleri doğrudan birleştirilemediği için, her iki veri setinden de **yıllık özetler** çıkarıldı ve bu yıllık özetler `Year` sütunu üzerinden birleştirildi. Bu, trafik akışı ve kaza sayısı arasındaki genel yıllık eğilimi analiz etmek için yapıldı.

### 3.2. Veri Ön İşleme ve Temizleme

* **Tarih ve Saat Dönüşümü:** `Date` ve `Time` sütunları `datetime` objelerine dönüştürüldü. Yanlış formatlı veya eksik değerler `NaT` (Not a Time) olarak işaretlendi ve uygun şekilde işlendi (genellikle eksik `Time` değerleri için en sık görülen saat kullanıldı).
* **Eksik Değer Yönetimi:** Yüksek oranda eksik değere sahip sütunlar (örneğin `Junction_Detail`, `Special_Conditions_at_Site`, `Carriageway_Hazards`) analizden çıkarıldı. Kalan eksik değerler, kategorik sütunlarda mod (en sık görülen değer) ile, sayısal sütunlarda ise ortalama/medyan ile dolduruldu.
* **Kategorik Değişkenlerin Dönüşümü:** Modelleme için kategorik özellikler (örn. `Weather_Conditions`, `Road_Type`) One-Hot Encoding kullanılarak sayısal formata dönüştürüldü.
* **Yeni Özellik Türetme:** `Date` sütunundan `Month` (Ay) ve `Hour` (Saat) gibi yeni zaman tabanlı özellikler türetilerek mevsimsel analizler için kullanıldı.

### 3.3. Keşifçi Veri Analizi (EDA)

Çeşitli görselleştirmeler kullanılarak veri setindeki paternler, dağılımlar ve ilişkiler incelendi. Başlıca görselleştirmelerden bazıları:

* Yıllara, aylara, haftanın günlerine ve günün saatlerine göre kaza sayıları dağılımları.
* Hava, yol yüzeyi, ışık koşulları, yol tipi ve hız limitlerine göre kaza dağılımları.
* Kentsel ve kırsal alanlardaki kaza paternleri ve kaza şiddeti dağılımı.
* Yıllık trafik akışı ile toplam kaza sayısı arasındaki ilişki.

### 3.4. Modelleme

Proje kapsamında iki ana modelleme yaklaşımı uygulandı:

#### 3.4.1. Zaman Serisi Tahmini (Prophet)

* **Amaç:** Aylık kaza sayılarındaki trendleri ve mevsimsel paternleri anlayarak gelecekteki kaza oranlarını tahmin etmek.
* **Model:** Facebook Prophet kütüphanesi kullanıldı.
* **Veri:** `Date` ve kaza sayılarının aylık toplamını içeren bir zaman serisi oluşturuldu.

#### 3.4.2. Kaza Şiddeti Sınıflandırması (Random Forest)

* **Amaç:** Kaza şiddetini (ölümcül, ciddi, hafif) etkileyen faktörleri belirlemek ve gelecekteki kazaların şiddetini tahmin etmek.
* **Model:** Random Forest Classifier kullanıldı.
* **Veri:** Temizlenmiş ve one-hot encoded özellikler (`X`) ve kaza şiddeti (`Accident_Severity`, `y`) kullanıldı.
* **Sınıf Dengesizliğiyle Mücadele:** Veri setindeki `Accident_Severity` sınıfının (özellikle ölümcül ve ciddi kazaların az olması) dengesizliği nedeniyle model performansını artırmak için **Class Weights (`class_weight='balanced'`)** yöntemi uygulanmıştır. Ayrıca **SMOTE (Synthetic Minority Over-sampling Technique)** yöntemi de denenmiştir.

## 4. Bulgular ve Sonuçlar

### 4.1. Keşifçi Veri Analizi Bulguları

* **Trafik Akışı ve Kaza İlişkisi:** Yıllık ortalama trafik akışı ile toplam kaza sayısı arasında pozitif bir korelasyon gözlemlenmiştir. Trafik akışı arttıkça kaza sayısı da artma eğilimindedir.
* **Zaman Paternleri:** Kazalarda genel olarak yıllara göre bir düşüş eğilimi, ancak belirgin dalgalanmalar (örneğin 2012'de ani artış) mevcuttur. Kazalar aylara göre mevsimsel bir patern göstermekte (sonbahar aylarında yoğunlaşma), hafta içi özellikle Cuma günleri ve günün yoğun saatlerinde (sabah ve öğleden sonra işe gidiş-geliş) zirve yapmaktadır.
* **Çevresel ve Yol Koşulları:** Kazaların büyük çoğunluğu **"güzel hava" ve "kuru yol"** koşullarında meydana gelmiştir. Gündüz vakti ve aydınlık koşullarda da kazalar daha sık yaşanmaktadır. **Tek şeritli yollar** ve **30 mph hız limitine sahip yollar** en yüksek kaza sayısına ev sahipliği yapmaktadır.
* **Coğrafi Farklılıklar:** Kentsel alanlarda kaza sayısı kırsal alanlara göre belirgin şekilde daha yüksektir. Hem kentsel hem de kırsal alanlarda hafif kazalar çoğunluğu oluşturmaktadır.

### 4.2. Modelleme Sonuçları

#### 4.2.1. Zaman Serisi Tahmini (Prophet)

Prophet modeli, aylık kaza sayılarındaki genel düşüş trendini ve güçlü yıllık mevsimsel paternleri (yılın belirli aylarında kaza artışı/azalışı) başarılı bir şekilde yakalamıştır. Modelin geçmişteki ani dalgalanmaları (örneğin 2012'deki ani artış) tam olarak yakalayamadığı gözlemlenmiştir.

#### 4.2.2. Kaza Şiddeti Sınıflandırması (Random Forest)

Model performansını artırmak ve sınıf dengesizliğiyle başa çıkmak için sınıf ağırlıklı Random Forest modeli kullanılmıştır.

* **Özellik Önem Dereceleri:** Analizler sonucunda **`Hour` (günün saati), `Day_of_Week` (haftanın günü) ve `Speed_limit` (hız limiti)** kaza şiddetini tahmin etmede en önemli faktörler olarak belirlenmiştir. Diğer önemli faktörler arasında `Urban_or_Rural_Area`, `Road_Type`, `Weather_Conditions` ve `Light_Conditions` bulunmaktadır.

* **Model Performansı (Sınıf Ağırlıklı Random Forest):**
    * **Ölümcül Kazalar (Sınıf 1):** Modelin `recall` değeri **%0'dan %42'ye** yükselerek ölümcül kazaları tespit etme yeteneğinde önemli bir iyileşme sağlamıştır. Ancak, `precision` değeri hala düşüktür (%3), bu da modelin diğer sınıfları (özellikle hafif kazaları) yanlışlıkla ölümcül olarak sınıflandırma eğiliminde olduğunu gösterir.
    * **Ciddi Kazalar (Sınıf 2):** `recall` değeri **%1'den %24'e** yükselmiştir. `precision` değeri %15 seviyesindedir.
    * **Hafif Kazalar (Sınıf 3):** Yüksek `precision` (%88) ve makul `recall` (%62) değerleri ile model, hafif kazaları hala çok iyi tespit etmektedir.
    * **Genel Değerlendirme:** Sınıf ağırlıklandırması, modelin azınlık sınıfları (ölümcül ve ciddi kazalar) üzerindeki tahmin yeteneğini önemli ölçüde artırmıştır. Her ne kadar `precision` değerleri düşük kalsa da, bu kritik kazaların daha yüksek bir oranda tespit edilmesi, yol güvenliği açısından önemli bir kazanımdır. SMOTE ile yapılan deneme de benzer sonuçlar verdiğinden, sınıf ağırlıklı model tercih edilmiştir.

## 5. Sonuç ve Öneriler

Bu proje, Birleşik Krallık'taki trafik kazaları ve trafik akışı arasındaki karmaşık ilişkileri anlamak için kapsamlı bir analiz sunmuştur. Elde edilen bulgular, kaza önleme stratejileri geliştirmek için değerli içgörüler sağlamaktadır.

**Temel Çıkarımlar:**

* **Trafik yoğunluğu kaza sayısıyla doğrudan ilişkilidir.** Yoğun saatlerde ve yoğun günlerde kaza riski artmaktadır.
* **Kentsel alanlar ve 30 mph hız limitli yollar, kaza sayısının en yüksek olduğu bölgelerdir.**
* **"Güzel hava" ve "kuru yol" koşullarında bile kazalar yaygındır**, bu da sürücü davranışlarının (hız, dikkat eksikliği) önemini vurgulayabilir.
* **Kaza şiddetini tahmin etmede en etkili faktörler günün saati, haftanın günü ve hız limitidir.**

**Gelecek Çalışmalar ve İyileştirme Önerileri:**

* **Model Performansını Artırma:** Kaza şiddeti sınıflandırma modelinde, özellikle ölümcül ve ciddi kazalar için `precision` değerlerini artırmaya yönelik daha ileri sınıf dengesizliği teknikleri (örn. Focal Loss, Under-sampling + Over-sampling kombinasyonları) veya daha karmaşık algoritmalar (örn. LightGBM, XGBoost) denenebilir.
* **Daha Detaylı Zaman Serisi Analizi:** Prophet modelindeki 2012-2013 gibi anormalliklerin nedenleri (örn. ekonomik değişimler, yasal düzenlemeler, büyük olaylar) daha detaylı araştırılabilir.
* **Mekansal Analiz:** Coğrafi konum verileri (enlem/boylam) kullanılarak kazaların yoğunlaştığı "sıcak noktalar" (hotspots) belirlenebilir ve bu bölgeler için özel yol güvenliği önlemleri önerilebilir.
* **Veri Kalitesi İyileştirmesi:** "Unknown" gibi yüksek sayıdaki kategorik değerler için daha fazla veri temizliği veya akıllı doldurma stratejileri geliştirilebilir.

Bu proje, Birleşik Krallık'ta yol güvenliğini artırmak ve kazaları azaltmak için veri odaklı kararlar alınmasına yönelik önemli bir başlangıç noktası sunmaktadır.
