---
layout: post
title: Sudoku Çözücü Uygulaması
slug: sudoku-solver
author: Bahri ABACI
categories:
- Görüntü İşleme Uygulamaları
- Lineer Cebir
- Nümerik Yöntemler
- Makine Öğrenmesi
references: "Use of the Hough Transformation to Detect Lines and Curves in Pictures"
thumbnail:  /assets/post_resources/sudoku_solver/thumbnail.png
---
Sudoku <img src="assets/post_resources/math//4383b081cba8f285e7854426f9ea1e6d.svg?invert_in_darkmode" align=middle width=8.219209349999991pt height=21.18721440000001pt/> adet <img src="assets/post_resources/math//46e42d6ebfb1f8b50fe3a47153d01cd2.svg?invert_in_darkmode" align=middle width=36.52961069999999pt height=21.18721440000001pt/> kareden oluşan ve <img src="assets/post_resources/math//46e42d6ebfb1f8b50fe3a47153d01cd2.svg?invert_in_darkmode" align=middle width=36.52961069999999pt height=21.18721440000001pt/> her kare içerisinde birde dokuza kadar olan sayıların bir kez kullanılması ve her satir ve sütunda tüm rakamların bulunması gibi iki kurala dayalı bir oyun. Bu yazımda IMLAB görüntü işleme kütüphanesinde yer alan temel fonksiyonları kullanarak, girdi olarak verilen bir fotoğraf üzerinden sudoku karesini tespit edip, bulmacanın çözümünü ekrana yazabilen bir uygulama yazmaya çalışacağız. Yazımızda IMLAB kütüphanesi aracılığı ile doğrudan kullanacağımız yöntemler [Bağlantılı Bileşen Etiketleme]({% post_url 2012-09-26-baglantili-bilesen-etiketleme %}), [Perspektif Dönüşümü]({% post_url 2013-12-08-perspektif-donusumu %}) ve [Rakam Tanıma]({% post_url 2015-07-30-otomatik-rakam-tanima %}) için ilgili bağlantıları kullanarak detayları öğrenebilirsiniz.

<!--more-->
Sudoku bulmacasının çözümü için ilk yapmamız gereken <img src="assets/post_resources/math//0c309043919f7fe787c69e3d848fedd7.svg?invert_in_darkmode" align=middle width=36.52961069999999pt height=21.18721440000001pt/> luk büyük kareyi bularak verilen *sudokunun* bilgisayarımıza doğru şekilde aktarımını sağlamak olacak. Büyük kareyi tespit etmek için kullanacağımız yöntem sudokuyu oluşturan ızgaranın arka planından daha koyu olacağı ve basit bir eşikleme işlemi sonucunda, görüntüdeki en büyük bağlantılı bileşen olacağı öngörüsüne dayanmaktadır.

Bağlantılı bileşenleri tespit edilmesi için girdi resminin ikili kodlanması (siyah-beyaz dönüşümü) gerekmektedir. Bunun için IMLAB kütüphanesinde yer alan `imthreshold` fonksiyonu kullanılabilir. Ancak resmin içeriğine dayalı en iyi eşik değerini bulan [Otsu Yöntemi ile Adaptif Eşikleme]({% post_url 2012-07-24-otsu-metodu-ile-adaptif-esikleme %}) algoritmasını dahi kullandığımızda girdi görüntüsünün aydınlanmasının homojen dağılımlı olmaması durumunda görüntünün bazı bölgeleri bağlantılı bileşen etiketlemesi için istediğimiz girdiyi üretmeyecektir. Bu gibi imgelerin üzerinde aydınlanmasnın homojen dağılımlı olmadığı görüntülerde küresel(global) eşik değeri yerine yerel (local) eşik değerleri kullanılmalıdır. Bu yazımızda da IMLAB kütüphanesinde yer alan yerel eşikleme fonksiyonu `imbinarize` kullanılacaktır. Bu fonksiyon görüntüyü <img src="assets/post_resources/math//ba4d5d21c97d8488fa733e12b201b3ed.svg?invert_in_darkmode" align=middle width=46.41543389999999pt height=22.465723500000017pt/> büyüklüğündeki bir kayar pencere ile tarayarak, her bir gözeği, <img src="assets/post_resources/math//ba4d5d21c97d8488fa733e12b201b3ed.svg?invert_in_darkmode" align=middle width=46.41543389999999pt height=22.465723500000017pt/> pencere içerisindeki ortalama değere göre siyah yada beyaza çevirmektedir. Örnek bir girdi görüntüsü için sabit eşik, Otsu yöntemi ile bulunan eşik ve yerel eşikleme sonucu elde edilen görüntüler aşağıda verilmiştir.

| Gri İmge   |  Sabit Eşik = 127 | Sabit Eşik = Otsu Eşiği | Yerel Eşik  |
|:----------:|:-----------------:|:----------------------:|:-----------:|
![Gri İmge][threshold_gray] | ![Sabit Eşik ile Eşikleme][threshold_constant] | ![Otsu Yöntemi ile Eşikleme][threshold_otsu] | ![Yerel Eşik Yöntemi ile Eşikleme][threshold_local]

Sudoku ızgarasını ayrıştırmak için, girdi imgeleri yerel eşik algoritması ile ikili kodlandıktan sonra elde edilen imge üzerinde bağlantılı bileşen etiketleme yöntemi kullanılacaktır. IMLAB kütüphanesinde bağlantılı bileşen etiketleme için iki fonksiyon bulunmaktadır. İlk fonksiyon `bwlabel` girdi olarak aldığı ikili imge boyutlarında bir matris içerisinde her gözeğe karşı gelen bleşen numarasını çıktı olarak vermektedir. İkinci fonksiyon `bwconncomp` her bağlantılı bileşenlere ait <img src="assets/post_resources/math//0acac2a2d5d05a8394e21a70a71041b4.svg?invert_in_darkmode" align=middle width=25.350096749999988pt height=14.15524440000002pt/> noktalarını bir `vector_t` yapısı içerisinde döndürmektedir. Sudoku ızgarasını ayrıştırmak için ihtiyacımız olan en büyük bağlantılı bileşeni bulmak olduğundan burada `bwconncomp` fonskiyonunu kullanmak kolaylık sağlayacaktır. Aşağıda sudoku ızgarasının tespiti için yazılan kod bloğu verilmiştir.

```c
matrix_t *image = imread("test.bmp");
matrix_t *gray_image = matrix_create(uint8_t);
matrix_t *binary_image = matrix_create(uint8_t);

// convert image into grayscale
rgb2gray(image, gray_image);
imbinarize(gray_image, 32, 32, 0, binary_image);
bwinvert(binary_image);

uint32_t numberOfComponenets = 0;
vector_t **connnectionList = bwconncomp(binary_image, &numberOfComponenets);

// print some debug information
printf("Number of connected components: %d\n", numberOfComponenets);

// find the largest connected component block (which is assumed to be the sudoku)
uint32_t largestPixelCount = 0;
uint32_t largestID = 0;

uint32_t i = 0;
for (i = 0; i < numberOfComponenets; i++)
{
    if (length(connnectionList[i]) > largestPixelCount)
    {
        largestPixelCount = length(connnectionList[i]);
        largestID = i;
    }
}

// paint it for better visualization
highlight(image, connnectionList[largestID], RGB(120, 200, 120));
```

Kodun farklı imgeler üzerinde ürettiği sonuçlardan bazıları aşağıdaki tabloda verilmiştir. Verilen örnek çıktıların bazılarında ızgaranın dışında kalan alanlarında ızgara ile gruplandığı görülmektedir. Ancak sudoku çerçevesinin tespiti için uygulayacağımız bir sonraki yöntemde bu taşmalar bir sorun yaratmayacaktır.

| Başarılı Örnek   |  Başarılı Örnek | Başarılı Örnek | Başarısız Örnek  |
|:----------:|:-----------------:|:----------------------:|:-----------:|
![Sudoku Tespiti Örnek 1][grid_highlighted_1] | ![Sudoku Tespiti Örnek 2][grid_highlighted_2] | ![Sudoku Tespiti Örnek 3][grid_highlighted_3] | ![Sudoku Tespiti Örnek 4][grid_highlighted_4]

Sudoku çerçevesinin tespiti için uygulayacağımız bir sonraki yöntem görüntüde yer alan çizgi benzeri geometrik şekillerin tespiti için kullanılan [Hough Dönüşümü](http://www.keymolen.com/2013/05/hough-transformation-c-implementation.html) yöntemi olacaktır.

### Hough Dönüşümü
Hough Dönüşümü, 1972 yılında Richard Duda ve Peter Hart tarafından görüntü içerisinde yer alan doğru parçalarının tespiti için önerilen bir yöntemdir. Ancak yöntemin sunduğu fikir oldukça genişletilebilir olduğundan kısa süre içerisinde yöntem farklı geometrik şekilleri de tespit edebilecek şekilde güncellenmiştir. Hough dönüşümünün ilk adımı kartezyen koordinatlarda bulunan bir doğrunun üzerinde yer alan (<img src="assets/post_resources/math//0acac2a2d5d05a8394e21a70a71041b4.svg?invert_in_darkmode" align=middle width=25.350096749999988pt height=14.15524440000002pt/>) noktalarının polar koordinatlara (<img src="assets/post_resources/math//def77fc2af8352ad288ac2a99dd74ded.svg?invert_in_darkmode" align=middle width=23.978294999999992pt height=22.831056599999986pt/>) dönüştürülmesi işlemidir. Burada <img src="assets/post_resources/math//0006b76bf9d824b50cc55aa2d1cff076.svg?invert_in_darkmode" align=middle width=158.1409797pt height=24.65753399999998pt/> verilen doğrunun orijine olan uzaklığını, <img src="assets/post_resources/math//27e556cf3caa0673ac49a8f0de3c73ca.svg?invert_in_darkmode" align=middle width=8.17352744999999pt height=22.831056599999986pt/> is doğruya çizilen dikmenin orijinle olan açısını göstermektedir. 

Örnek olarak <img src="assets/post_resources/math//d11607948567c4852557e5b32b867341.svg?invert_in_darkmode" align=middle width=68.2722249pt height=21.18721440000001pt/> doğrusu üzerinde yer alan noktalardan oluşan bir küçük bir seti ele alalım. Bu doğrunun orijine en yakın noktasında (<img src="assets/post_resources/math//8581ef5d8e3a3c89615c6d3e4156ad9c.svg?invert_in_darkmode" align=middle width=62.10060284999999pt height=21.18721440000001pt/>) orijine olan uzaklığı <img src="assets/post_resources/math//687d89538a97949cdf6ac9cc11fd88c2.svg?invert_in_darkmode" align=middle width=76.07878575pt height=21.18721440000001pt/> ve bu noktanın orijin ile açısı ise <img src="assets/post_resources/math//54a881e18ba106fac536cdf0b0af0d28.svg?invert_in_darkmode" align=middle width=54.748785299999994pt height=22.831056599999986pt/> derecedir. Farklı <img src="assets/post_resources/math//27e556cf3caa0673ac49a8f0de3c73ca.svg?invert_in_darkmode" align=middle width=8.17352744999999pt height=22.831056599999986pt/> değerleri için bu işlem tekrarlandığında aşağıdaki tablo elde edilir.

|        <img src="assets/post_resources/math//7392a8cd69b275fa1798ef94c839d2e0.svg?invert_in_darkmode" align=middle width=38.135511149999985pt height=24.65753399999998pt/>      |  (-5,-4)  |  (-4,-3)  |  (-3,-2)  |  (-2,-1)  |  (-1,0)  |  (0,1)  |  (1,2)  |  (2,3)  |  (3,4)  |  (4,5)  |  (5,6)  |
|:-------------------:|:------:|:------:|:------:|:------:|:------:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| <img src="assets/post_resources/math//cf10ba20be4233451efbcf0d51bb5951.svg?invert_in_darkmode" align=middle width=62.33434349999999pt height=22.831056599999986pt/>  | -6.330 | -4.964 | -3.598 | -2.232 | -0.866 | 0.500 | 1.866 | 3.232 | 4.598 | 5.964 | 7.330 |
| <img src="assets/post_resources/math//318fbecd7361d9728aea674732ad8434.svg?invert_in_darkmode" align=middle width=62.33434349999999pt height=22.831056599999986pt/>  | -5.964 | -4.598 | -3.232 | -1.866 | -0.500 | 0.866 | 2.232 | 3.598 | 4.964 | 6.330 | 7.696 |
| <img src="assets/post_resources/math//bd61b3b73d5dd0793f56da0cffdbf0d2.svg?invert_in_darkmode" align=middle width=70.55355284999999pt height=22.831056599999986pt/> |  0.707 |  0.707 | 0.707  | 0.707  | 0.707  | 0.707 | 0.707 | 0.707 | 0.707 | 0.707 | 0.707 |

Dönüşüm işleminden beklendiği ve tablodan görüleceği üzere, <img src="assets/post_resources/math//54a881e18ba106fac536cdf0b0af0d28.svg?invert_in_darkmode" align=middle width=54.748785299999994pt height=22.831056599999986pt/> seçilmesi durumunda doğru üzerindeki tüm noktaların aynı <img src="assets/post_resources/math//6dec54c48a0438a5fcde6053bdb9d712.svg?invert_in_darkmode" align=middle width=8.49888434999999pt height=14.15524440000002pt/> değerini üretmektedir. Bu olay Hough dönüşümünün temelini oluşturmaktadır.

Hough dönüşümününde kartezyen koordinatlarda yer alan tüm noktalar, <img src="assets/post_resources/math//5f962ddca763944c1396295c53415f27.svg?invert_in_darkmode" align=middle width=112.47193484999998pt height=24.65753399999998pt/> aralığındaki tüm <img src="assets/post_resources/math//27e556cf3caa0673ac49a8f0de3c73ca.svg?invert_in_darkmode" align=middle width=8.17352744999999pt height=22.831056599999986pt/> değerleri için (<img src="assets/post_resources/math//def77fc2af8352ad288ac2a99dd74ded.svg?invert_in_darkmode" align=middle width=23.978294999999992pt height=22.831056599999986pt/>) parametre setinden oluşan iki boyulu bir uzaya aktarılır ve bu uzayda iki boyutlu biir histogram hesabı yapılır. Tablodan görüldüğü gibi, doğru (<img src="assets/post_resources/math//27e556cf3caa0673ac49a8f0de3c73ca.svg?invert_in_darkmode" align=middle width=8.17352744999999pt height=22.831056599999986pt/>) değeri seçildiğinde tüm noktalar uzayda tek bir noktaya (<img src="assets/post_resources/math//def77fc2af8352ad288ac2a99dd74ded.svg?invert_in_darkmode" align=middle width=23.978294999999992pt height=22.831056599999986pt/>) gideceğinden, iki boyutlu histogramın (<img src="assets/post_resources/math//def77fc2af8352ad288ac2a99dd74ded.svg?invert_in_darkmode" align=middle width=23.978294999999992pt height=22.831056599999986pt/>) noktasında dönüşüm yapılan nokta sayısına eşit bir tepe oluşacaktır. Doğru olmayan <img src="assets/post_resources/math//27e556cf3caa0673ac49a8f0de3c73ca.svg?invert_in_darkmode" align=middle width=8.17352744999999pt height=22.831056599999986pt/> değerleri için (<img src="assets/post_resources/math//def77fc2af8352ad288ac2a99dd74ded.svg?invert_in_darkmode" align=middle width=23.978294999999992pt height=22.831056599999986pt/>) çiftleri farklı konumlara dağılacağından, histogramda bu noktaların değerleri küçük kalacaktır.

`minTheta`= <img src="assets/post_resources/math//e93ce0648cd6156570f497ee1873ba2c.svg?invert_in_darkmode" align=middle width=32.15866499999999pt height=22.831056599999986pt/> ve `maxTheta`= <img src="assets/post_resources/math//20d4bcfec3faaaae8404a53c8fc76fb2.svg?invert_in_darkmode" align=middle width=33.96649739999999pt height=22.831056599999986pt/> tespit edilmek istenen doğruların en küçük ve en büyük açısını göstermek üzere; Hough dönüşümünü hesaplamak için aşağıdaki kod parçası yazılmıştır. Burada `hough` ismi ile tanımlanan matris yapısı iki boyutlu histogram matrisidir.

```c
uint32_t mapSize = 2 * sqrt(width*width + height*height) + 1;
uint32_t mapSizeHalf = (mapSize - 1) / 2;

int minTheta = -30;
int maxTheta = 120;

matrix_t *hough = matrix_create(float, (maxTheta - minTheta + 1), mapSize);

int i = 0, j = 0;
for (i = 0; i < length(keyPoints); i++)
{
    struct point_t p = at(struct point_t, keyPoints, i);

    int theta = 0;
    for (theta = minTheta; theta <= maxTheta; theta++)
    {
        float t = (float)(theta) * 3.14159265359 / 180;
        int rho = p.x * cos(t) + p.y * sin(t);
        
        atf(hough, theta - minTheta, mapSizeHalf + rho, 0) += 1.0f;
    }
}
```

Kod incelendiğinde dönüşüm yapılacak noktaların (`keyPoints`), `minTheta=-30` ve `maxTheta=120` aralığında Hough uzayına dönüştürülmekte ve noktaların uzayda düştüğü (<img src="assets/post_resources/math//def77fc2af8352ad288ac2a99dd74ded.svg?invert_in_darkmode" align=middle width=23.978294999999992pt height=22.831056599999986pt/>) noktasının içerdiği nokta sayısının histogramının hesaplandığı görülür. Yukarıda da belirttiğimiz gibi, `keyPoints` ile verilen noktaların oluşturduğu doğrular, histogram matrisi `hough` içerisinde (<img src="assets/post_resources/math//def77fc2af8352ad288ac2a99dd74ded.svg?invert_in_darkmode" align=middle width=23.978294999999992pt height=22.831056599999986pt/>) noktasında doğrunun uzunluğuna eşit tepeler oluşturacaktır.

Yukarıda verilen başarılı örnekler için `hough` matrisi görselleştirildiğinde aşağıda verilen görüntü elde edilecektir. Oluşan görüntüde dikey eksende her satır <img src="assets/post_resources/math//2b4b2cd71b46b834a961b297bede9fd4.svg?invert_in_darkmode" align=middle width=98.58447719999998pt height=24.65753399999998pt/> değerini, yatay eksen ise dönüşüm sonucu buluan <img src="assets/post_resources/math//6dec54c48a0438a5fcde6053bdb9d712.svg?invert_in_darkmode" align=middle width=8.49888434999999pt height=14.15524440000002pt/> değerlerini göstermektedir. 

![Hough Dönüşümü][hough_transform]

Görüntüdeki parlak noktalar (tepe noktaları, doğru adayları) incelendiğinde toplam 20 nokta göze çarpmaktadır. Bu noktaların da kendi içerisinde <img src="assets/post_resources/math//30a4f32d24106f7cce4e8023e9be212b.svg?invert_in_darkmode" align=middle width=38.310366599999995pt height=22.831056599999986pt/> ve <img src="assets/post_resources/math//6a83ff6e99cdf828f1d481ca953b4914.svg?invert_in_darkmode" align=middle width=46.529575949999995pt height=22.831056599999986pt/> açıları etrafında dağıldığı görülmektedir. Tahmin edileceği üzere bu tepe noktaları sudoku ızgarasını oluşturan doğru parçalarıdır. Bu parçalardan belirli bir uzunluktan (<img src="assets/post_resources/math//4c789beffe18ea6c4011dbc3957e5db4.svg?invert_in_darkmode" align=middle width=203.44603124999995pt height=24.65753399999998pt/>) büyük olan (`hough` matrisinin değeri <img src="assets/post_resources/math//0fe1677705e987cac4f589ed600aa6b3.svg?invert_in_darkmode" align=middle width=9.046852649999991pt height=14.15524440000002pt/> dan büyük olan) noktalar seçilip, görüntü üzerine çizdirildiğinde aşağıdaki görüntü oluşur.

| Başarılı Örnek   |  Başarılı Örnek | Başarılı Örnek | Başarısız Örnek  |
|:----------:|:-----------------:|:----------------------:|:-----------:|
![Sudoku Tespiti Örnek 1][all_lines_1] | ![Sudoku Tespiti Örnek 2][all_lines_2] | ![Sudoku Tespiti Örnek 3][all_lines_3] | ![Sudoku Tespiti Örnek 4][all_lines_4]

### Sudoku Çerçevesinin Bulunması

Bu aşamadan sonra sudoku çerçevesini oluşturan dikdörtgenin köşe noktaları tespit edilebilir. Görüntüden dikkat edileceği üzere, görüntü çok eğik bir şekilde çekilmedi ise, sudoku çerçevesini oluşturan hatlar `hough` matrisi üzerinde, <img src="assets/post_resources/math//f2ebeadd36ad2620cbe7f02c861c9da3.svg?invert_in_darkmode" align=middle width=16.438418699999993pt height=21.18721440000001pt/> dereceden küçük noktaların en küçük ve en büyük <img src="assets/post_resources/math//6dec54c48a0438a5fcde6053bdb9d712.svg?invert_in_darkmode" align=middle width=8.49888434999999pt height=14.15524440000002pt/> değerli olanları sağ ve sol kenar olarak,  <img src="assets/post_resources/math//f2ebeadd36ad2620cbe7f02c861c9da3.svg?invert_in_darkmode" align=middle width=16.438418699999993pt height=21.18721440000001pt/> dereceden büyük noktaların en küçük ve en büyük <img src="assets/post_resources/math//6dec54c48a0438a5fcde6053bdb9d712.svg?invert_in_darkmode" align=middle width=8.49888434999999pt height=14.15524440000002pt/> değerleri ise üst ve alt kenar doğruları olarak seçilebilir. Bu seçme işlemi tüm tepe adayları gezilerek basit bir şekilde tamamlanabilir.

Kenar doğruları bulunduktan sonra sudoku imgesinin hizalanması için bu doğruların kesişim noktalarının bulunması gerekmektedir. <img src="assets/post_resources/math//d29033078baeb5382a2cdf16bace2004.svg?invert_in_darkmode" align=middle width=51.05601269999999pt height=24.65753399999998pt/> ve <img src="assets/post_resources/math//6019b6872a4c2c15833ecb853f61f6f5.svg?invert_in_darkmode" align=middle width=51.05601269999999pt height=24.65753399999998pt/> parametreleri ile verilen iki doğrunun kesişim noktası <img src="assets/post_resources/math//7392a8cd69b275fa1798ef94c839d2e0.svg?invert_in_darkmode" align=middle width=38.135511149999985pt height=24.65753399999998pt/> aşağıdaki adımlar ile bulunur.

 - Kesişim noktası iki doğru denklemini de sağlayacağından <img src="assets/post_resources/math//7392a8cd69b275fa1798ef94c839d2e0.svg?invert_in_darkmode" align=middle width=38.135511149999985pt height=24.65753399999998pt/>, Hough denkleminde yerine konularak aşağıdaki eşitlikler elde edilir.

<p align="center"><img src="assets/post_resources/math//0d4ab5f6b2b7539ee135b7e9170dfe1a.svg?invert_in_darkmode" align=middle width=179.35108785pt height=41.09589pt/></p>

 - Burada <img src="assets/post_resources/math//332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode" align=middle width=9.39498779999999pt height=14.15524440000002pt/> bilinmeyeni her iki denklemde de yalnız bırakılıp birbirine eşitlenerek aşağıdaki ifade bulunur.
 
  <p align="center"><img src="assets/post_resources/math//95e4e3694cfd8428896a3024515ab60c.svg?invert_in_darkmode" align=middle width=343.5654024pt height=16.438356pt/></p>
 
 - Bu ifade <img src="assets/post_resources/math//deceeaf6940a8c7a5a02373728002b0f.svg?invert_in_darkmode" align=middle width=8.649225749999989pt height=14.15524440000002pt/> bilinmeyeni sağda kalacak şekilde toparlanarak aşağıdaki denklem elde edilir.
  
  <p align="center"><img src="assets/post_resources/math//e24f90f3b2d9e7e17d7e121ee94f3d9a.svg?invert_in_darkmode" align=middle width=424.64232855pt height=16.438356pt/></p> 
  
 - Sağda <img src="assets/post_resources/math//deceeaf6940a8c7a5a02373728002b0f.svg?invert_in_darkmode" align=middle width=8.649225749999989pt height=14.15524440000002pt/> nin yanında kalan çarpım ifadesinin sinüs iki açı farkı şeklinde yazılarak <img src="assets/post_resources/math//deceeaf6940a8c7a5a02373728002b0f.svg?invert_in_darkmode" align=middle width=8.649225749999989pt height=14.15524440000002pt/> değeri bulunur.
 
  <p align="center"><img src="assets/post_resources/math//18b1a831edfa8cf59e12bb9c4a90260c.svg?invert_in_darkmode" align=middle width=189.62850719999997pt height=38.83491479999999pt/></p> 
 
  - Aynı işlem <img src="assets/post_resources/math//332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode" align=middle width=9.39498779999999pt height=14.15524440000002pt/> için de tekrarlanırsa <img src="assets/post_resources/math//332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode" align=middle width=9.39498779999999pt height=14.15524440000002pt/> değeri bulunur.
 
  <p align="center"><img src="assets/post_resources/math//5ed9c4fa1aeb7a61ae67a551ce4521b8.svg?invert_in_darkmode" align=middle width=186.721359pt height=38.83491479999999pt/></p> 
 
**<span style="color: red;">NOT: </span> Burada <img src="assets/post_resources/math//e1fe350e0c446abae239e20242001531.svg?invert_in_darkmode" align=middle width=51.27842114999999pt height=22.831056599999986pt/> olması durumunda (iki doğrunun paralel olması) paydanın sıfır olduğuna ve kesişim noktasının bulunmadığına dikkat edilmelidir.**

Aşağıda verilen kod parçası `rho1`,`theta1` ve `rho2`,`theta2` parametreleri ile tanımlanan iki doğrunun kartezyen koordinatlardaki kesişim noktasını bulmak için yazılmıştır.

 ```c
// convert degree to radian
float t1 = theta1 * 3.14159265359 / 180;
float t2 = theta2 * 3.14159265359 / 180;

// if the lines are parallel to each other, no intersection point is possible
if(sin(t1 - t2) == 0.0f)
{
    return 0;
}
// compute the intersection point
else
{
    x = (rho2 * sin(t1) - rho1 * sin(t2)) / sin(t1 - t2);
    y = (rho1 * cos(t2) - rho2 * cos(t1)) / sin(t1 - t2);
}

 ```

kesişim noktasının bulunması işlemi bir önceki adımda elde ettiğimiz dört kenar doğrusuna uygulanması durumunda sudoku çerçevesinin kenar noktaları tespit edilebilir. Aşağıda bu işlemler sonucunda tespit edilen köşe noktaları ve kenar doğruları çizilmiştir.

| Başarılı Örnek   |  Başarılı Örnek | Başarılı Örnek | Başarısız Örnek  |
|:----------:|:-----------------:|:----------------------:|:-----------:|
![Sudoku Tespiti Örnek 1][all_edges_1] | ![Sudoku Tespiti Örnek 2][all_edges_2] | ![Sudoku Tespiti Örnek 3][all_edges_3] | ![Sudoku Tespiti Örnek 4][all_edges_4]

Bu adımdan sonra [Perspektif Dönüşümü]({% post_url 2013-12-08-perspektif-donusumu %}) yazımızda detaylarından bahsedilen perspektif dönüşümü işlemi uygulanarak sudoku bölgesinin sabit büyüklükteki bir imgeye dönüştürülmesi sağlanacaktır. Bu işlem için öncelikle dönüşüm matrisinin bulunması gerekmektedir. [Perspektif Dönüşümü]({% post_url 2013-12-08-perspektif-donusumu %}) yazımızda incelediğimiz dört nokta kullanılarak perspektif dönüşüm matrisinin bulunması işlemi IMLAB kütüphanesinde yer alan `pts2tform` fonksiyonu yardımıyla yapılabilir. Bu fonksiyon ile bulunan dönüşüm matrisi kütüphanede yer alan `imtransform` yöntemi ile girdi imgesine uygulanarak sudoku bölgesi elde edilir. Aşağıda perpektif düzeltmesi yapmak için kullanılan kod parçası verilmiştir.

```c
matrix_t *sudoku = matrix_create(uint8_t, 360, 360, 3);
struct point_t destination[4] = { {.x = 0, .y = 0},{.x = 359, .y = 0},{.x = 359, .y = 359},{.x = 0, .y = 359} };

// correct the image
matrix_t *transform = pts2tform(corners, destination, 4);
imtransform(image, transform, sudoku);
```

Verilen kodda `pts2tform` yardımı ile bir önceki adımda tespit edilen `corners` noktalarını, `destination` olarak belirlenen <img src="assets/post_resources/math//74b8b842bcac8c68e1bdfc9b6023f706.svg?invert_in_darkmode" align=middle width=69.40644809999999pt height=21.18721440000001pt/> bir imgenin köşe noktalarına taşıyacak dönüşüm matrisi hesaplanmıştır. Ardından bu matris girdi imgesine uygulanarak `sudoku` imgesi elde edilmiştir. Aşağıda örnek olarak kullanılan dört imge için elde edilen çıktı imgeleri verilmiştir.

| Başarılı Örnek   |  Başarılı Örnek | Başarılı Örnek | Başarısız Örnek  |
|:----------:|:-----------------:|:----------------------:|:-----------:|
![Sudoku Tespiti Örnek 1][sudoku_1] | ![Sudoku Tespiti Örnek 2][sudoku_2] | ![Sudoku Tespiti Örnek 3][sudoku_3] | ![Sudoku Tespiti Örnek 4][sudoku_4]

Bu aşamadan sonra yapılması gereken elde edilen sudoku imgesini parçalara yaırarak rakam tanıma işlemi yapmaktır. Ancak şu ana kadar kullandığım yöntemler istenilen sınıflandırma başarısını elde edemediği için rakam tanıma ile ilgili detayları daha sonraya bırakıyorum. Projenin güncel ve kısmen çalışır haline yazının [GitHub sayfası](https://github.com/cescript/imlab_sudoku_solver_app) üzerinden erişebilirsiniz.

**Referanslar**
* Duda, R. O. and P. E. Hart, "Use of the Hough Transformation to Detect Lines and Curves in Pictures," Comm. ACM, Vol. 15, pp. 11–15 (January, 1972)

[RESOURCES]: # (List of the resources used by the blog post)
[threshold_gray]: /assets/post_resources/sudoku_solver/threshold/1/gray.png
[threshold_constant]: /assets/post_resources/sudoku_solver/threshold/1/threshold_127.png
[threshold_otsu]: /assets/post_resources/sudoku_solver/threshold/1/threshold_otsu.png
[threshold_local]: /assets/post_resources/sudoku_solver/threshold/1/threshold_local.png

[grid_highlighted_1]: /assets/post_resources/sudoku_solver/bwconncomp/grid_highlighted_6.png
[grid_highlighted_2]: /assets/post_resources/sudoku_solver/bwconncomp/grid_highlighted_8.png
[grid_highlighted_3]: /assets/post_resources/sudoku_solver/bwconncomp/grid_highlighted_9.png
[grid_highlighted_4]: /assets/post_resources/sudoku_solver/bwconncomp/grid_highlighted_11.png

[hough_transform]: /assets/post_resources/sudoku_solver/hough_transform.png

[all_lines_1]: /assets/post_resources/sudoku_solver/all_lines/all_lines_6.png
[all_lines_2]: /assets/post_resources/sudoku_solver/all_lines/all_lines_8.png
[all_lines_3]: /assets/post_resources/sudoku_solver/all_lines/all_lines_9.png
[all_lines_4]: /assets/post_resources/sudoku_solver/all_lines/all_lines_11.png

[all_edges_1]: /assets/post_resources/sudoku_solver/all_edges/all_edges_6.png
[all_edges_2]: /assets/post_resources/sudoku_solver/all_edges/all_edges_8.png
[all_edges_3]: /assets/post_resources/sudoku_solver/all_edges/all_edges_9.png
[all_edges_4]: /assets/post_resources/sudoku_solver/all_edges/all_edges_11.png

[sudoku_1]: /assets/post_resources/sudoku_solver/sudoku/sudoku_6.png
[sudoku_2]: /assets/post_resources/sudoku_solver/sudoku/sudoku_8.png
[sudoku_3]: /assets/post_resources/sudoku_solver/sudoku/sudoku_9.png
[sudoku_4]: /assets/post_resources/sudoku_solver/sudoku/sudoku_11.png
