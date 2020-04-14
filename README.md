---
layout: post
title: Sudoku Çözücü Uygulaması
date: '2013-12-02T00:48:00.001+02:00'
author: Bahri ABACI
categories:
- Görüntü İşleme Uygulamaları
- Lineer Cebir
- Nümerik Yöntemler
- Makine Öğrenmesi
modified_time: '2015-09-16T12:17:32.966+03:00'
thumbnail:  /assets/post_resources/sudoku_solver/thumbnail.png
---
Sudoku !["9"](https://render.githubusercontent.com/render/math?math=9) adet !["3\times 3"](https://render.githubusercontent.com/render/math?math=3%5ctimes%203) kareden oluşan ve !["3\times 3"](https://render.githubusercontent.com/render/math?math=3%5ctimes%203) her kare içerisinde birde dokuza kadar olan sayıların bir kez kullanılması ve her satir ve sütunda tüm rakamların bulunması gibi iki kurala dayalı bir oyun. Bu yazımda IMLAB görüntü işleme kütüphanesinde yer alan temel fonksiyonları kullanarak, girdi olarak verilen bir fotoğraf üzerinden sudoku karesini tespit edip, bulmacanın çözümünü ekrana yazabilen bir uygulama yazmaya çalışacağız. Yazımızda IMLAB kütüphanesi aracılığı ile doğrudan kullanacağımız yöntemler [Bağlantılı Bileşen Etiketleme]({% post_url 2012-09-26-baglantili-bilesen-etiketleme %}), [Perspektif Dönüşümü]({% post_url 2013-12-08-perspektif-donusumu %}) ve [Rakam Tanıma]({% post_url 2015-07-30-otomatik-rakam-tanima %}) için ilgili bağlantıları kullanarak detayları öğrenebilirsiniz.

<!--more-->
Sudoku bulmacasının çözümü için ilk yapmamız gereken !["9\times 9"](https://render.githubusercontent.com/render/math?math=9%5ctimes%209) luk büyük kareyi bularak verilen *sudokunun* bilgisayarımıza doğru şekilde aktarımını sağlamak olacak. Büyük kareyi tespit etmek için kullanacağımız yöntem sudokuyu oluşturan ızgaranın arka planından daha koyu olacağı ve basit bir eşikleme işlemi sonucunda, görüntüdeki en büyük bağlantılı bileşen olacağı öngörüsüne dayanmaktadır.

Bağlantılı bileşenleri tespit edilmesi için girdi resminin ikili kodlanması (siyah-beyaz dönüşümü) gerekmektedir. Bunun için IMLAB kütüphanesinde yer alan `imthreshold` fonksiyonu kullanılabilir. Ancak resmin içeriğine dayalı en iyi eşik değerini bulan [Otsu Yöntemi ile Adaptif Eşikleme]({% post_url 2012-07-24-otsu-metodu-ile-adaptif-esikleme %}) algoritmasını dahi kullandığımızda girdi görüntüsünün aydınlanmasının homojen dağılımlı olmaması durumunda görüntünün bazı bölgeleri bağlantılı bileşen etiketlemesi için istediğimiz girdiyi üretmeyecektir. Bu gibi imgelerin üzerinde aydınlanmasnın homojen dağılımlı olmadığı görüntülerde küresel(global) eşik değeri yerine yerel (local) eşik değerleri kullanılmalıdır. Bu yazımızda da IMLAB kütüphanesinde yer alan yerel eşikleme fonksiyonu `imbinarize` kullanılacaktır. Bu fonksiyon görüntüyü !["K \times L"](https://render.githubusercontent.com/render/math?math=K%20%5ctimes%20L) büyüklüğündeki bir kayar pencere ile tarayarak, her bir gözeği, !["K \times L"](https://render.githubusercontent.com/render/math?math=K%20%5ctimes%20L) pencere içerisindeki ortalama değere göre siyah yada beyaza çevirmektedir. Örnek bir girdi görüntüsü için sabit eşik, Otsu yöntemi ile bulunan eşik ve yerel eşikleme sonucu elde edilen görüntüler aşağıda verilmiştir.

| Gri İmge   |  Sabit Eşik = 127 | Sabit Eşik = Otsu Eşiği | Yerel Eşik  |
|:----------:|:-----------------:||:----------------------:|:-----------:|
![Gri İmge][threshold_gray] | ![Sabit Eşik ile Eşikleme][threshold_constant] | ![Otsu Yöntemi ile Eşikleme][threshold_otsu] | ![Yerel Eşik Yöntemi ile Eşikleme][threshold_local]

Sudoku ızgarasını ayrıştırmak için, girdi imgeleri yerel eşik algoritması ile ikili kodlandıktan sonra elde edilen imge üzerinde bağlantılı bileşen etiketleme yöntemi kullanılacaktır. IMLAB kütüphanesinde bağlantılı bileşen etiketleme için iki fonksiyon bulunmaktadır. İlk fonksiyon `bwlabel` girdi olarak aldığı ikili imge boyutlarında bir matris içerisinde her gözeğe karşı gelen bleşen numarasını çıktı olarak vermektedir. İkinci fonksiyon `bwconncomp` her bağlantılı bileşenlere ait !["x,y"](https://render.githubusercontent.com/render/math?math=x%2cy) noktalarını bir `vector_t` yapısı içerisinde döndürmektedir. Sudoku ızgarasını ayrıştırmak için ihtiyacımız olan en büyük bağlantılı bileşeni bulmak olduğundan burada `bwconncomp` fonskiyonunu kullanmak kolaylık sağlayacaktır. Aşağıda sudoku ızgarasının tespiti için yazılan kod bloğu verilmiştir.

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
|:----------:|:-----------------:||:----------------------:|:-----------:|
![Sudoku Tespiti Örnek 1][grid_highlighted_1] | ![Sudoku Tespiti Örnek 2][grid_highlighted_2] | ![Sudoku Tespiti Örnek 3][grid_highlighted_3] | ![Sudoku Tespiti Örnek 4][grid_highlighted_4]

Sudoku çerçevesinin tespiti için uygulayacağımız bir sonraki yöntem görüntüde yer alan çizgi benzeri geometrik şekillerin tespiti için kullanılan [Hough Dönüşümü](http://www.keymolen.com/2013/05/hough-transformation-c-implementation.html) yöntemi olacaktır.

### Hough Dönüşümü
Hough Dönüşümü, 1972 yılında Richard Duda ve Peter Hart tarafından görüntü içerisinde yer alan doğru parçalarının tespiti için önerilen bir yöntemdir. Ancak yöntemin sunduğu fikir oldukça genişletilebilir olduğundan kısa süre içerisinde yöntem farklı geometrik şekilleri de tespit edebilecek şekilde güncellenmiştir. Hough dönüşümünün ilk adımı kartezyen koordinatlarda bulunan bir doğrunun üzerinde yer alan (!["x,y"](https://render.githubusercontent.com/render/math?math=x%2cy)) noktalarının polar koordinatlara (!["\rho,\theta"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta)) dönüştürülmesi işlemidir. Burada !["\rho = x\cos(\theta) + y\sin(\theta)"](https://render.githubusercontent.com/render/math?math=%5crho%20%3d%20x%5ccos%28%5ctheta%29%20%2b%20y%5csin%28%5ctheta%29) verilen doğrunun orijine olan uzaklığını, !["\theta"](https://render.githubusercontent.com/render/math?math=%5ctheta) is doğruya çizilen dikmenin orijinle olan açısını göstermektedir. 

Örnek olarak !["y=x+1"](https://render.githubusercontent.com/render/math?math=y%3dx%2b1) doğrusu üzerinde yer alan noktalardan oluşan bir küçük bir seti ele alalım. Bu doğrunun orijine en yakın noktasında (!["-0.5,0.5"](https://render.githubusercontent.com/render/math?math=-0.5%2c0.5)) orijine olan uzaklığı !["\rho=0.7071"](https://render.githubusercontent.com/render/math?math=%5crho%3d0.7071) ve bu noktanın orijin ile açısı ise !["\theta=135"](https://render.githubusercontent.com/render/math?math=%5ctheta%3d135) derecedir. Farklı !["\theta"](https://render.githubusercontent.com/render/math?math=%5ctheta) değerleri için bu işlem tekrarlandığında aşağıdaki tablo elde edilir.

|        !["(x,y)"](https://render.githubusercontent.com/render/math?math=%28x%2cy%29)      |  (-5,-4)  |  (-4,-3)  |  (-3,-2)  |  (-2,-1)  |  (-1,0)  |  (0,1)  |  (1,2)  |  (2,3)  |  (3,4)  |  (4,5)  |  (5,6)  |
|:-------------------:|:------:|:------:|:------:|:------:|:------:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| !["\rho,\theta=30"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta%3d30)  | -6.330 | -4.964 | -3.598 | -2.232 | -0.866 | 0.500 | 1.866 | 3.232 | 4.598 | 5.964 | 7.330 |
| !["\rho,\theta=60"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta%3d60)  | -5.964 | -4.598 | -3.232 | -1.866 | -0.500 | 0.866 | 2.232 | 3.598 | 4.964 | 6.330 | 7.696 |
| !["\rho,\theta=135"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta%3d135) |  0.707 |  0.707 | 0.707  | 0.707  | 0.707  | 0.707 | 0.707 | 0.707 | 0.707 | 0.707 | 0.707 |

Dönüşüm işleminden beklendiği ve tablodan görüleceği üzere, !["\theta=135"](https://render.githubusercontent.com/render/math?math=%5ctheta%3d135) seçilmesi durumunda doğru üzerindeki tüm noktaların aynı !["\rho"](https://render.githubusercontent.com/render/math?math=%5crho) değerini üretmektedir. Bu olay Hough dönüşümünün temelini oluşturmaktadır.

Hough dönüşümününde kartezyen koordinatlarda yer alan tüm noktalar, !["\theta \in \[\theta_{min},\theta_{max}\]"](https://render.githubusercontent.com/render/math?math=%5ctheta%20%5cin%20%5c%5b%5ctheta_%7bmin%7d%2c%5ctheta_%7bmax%7d%5c%5d) aralığındaki tüm !["\theta"](https://render.githubusercontent.com/render/math?math=%5ctheta) değerleri için (!["\rho,\theta"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta)) parametre setinden oluşan iki boyulu bir uzaya aktarılır ve bu uzayda iki boyutlu biir histogram hesabı yapılır. Tablodan görüldüğü gibi, doğru (!["\theta"](https://render.githubusercontent.com/render/math?math=%5ctheta)) değeri seçildiğinde tüm noktalar uzayda tek bir noktaya (!["\rho,\theta"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta)) gideceğinden, iki boyutlu histogramın (!["\rho,\theta"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta)) noktasında dönüşüm yapılan nokta sayısına eşit bir tepe oluşacaktır. Doğru olmayan !["\theta"](https://render.githubusercontent.com/render/math?math=%5ctheta) değerleri için (!["\rho,\theta"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta)) çiftleri farklı konumlara dağılacağından, histogramda bu noktaların değerleri küçük kalacaktır.

`minTheta`= !["\theta_{min}"](https://render.githubusercontent.com/render/math?math=%5ctheta_%7bmin%7d) ve `maxTheta`= !["\theta_{max}"](https://render.githubusercontent.com/render/math?math=%5ctheta_%7bmax%7d) tespit edilmek istenen doğruların en küçük ve en büyük açısını göstermek üzere; Hough dönüşümünü hesaplamak için aşağıdaki kod parçası yazılmıştır. Burada `hough` ismi ile tanımlanan matris yapısı iki boyutlu histogram matrisidir.

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

Kod incelendiğinde dönüşüm yapılacak noktaların (`keyPoints`), `minTheta=-30` ve `maxTheta=120` aralığında Hough uzayına dönüştürülmekte ve noktaların uzayda düştüğü (!["\rho,\theta"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta)) noktasının içerdiği nokta sayısının histogramının hesaplandığı görülür. Yukarıda da belirttiğimiz gibi, `keyPoints` ile verilen noktaların oluşturduğu doğrular, histogram matrisi `hough` içerisinde (!["\rho,\theta"](https://render.githubusercontent.com/render/math?math=%5crho%2c%5ctheta)) noktasında doğrunun uzunluğuna eşit tepeler oluşturacaktır.

Yukarıda verilen başarılı örnekler için `hough` matrisi görselleştirildiğinde aşağıda verilen görüntü elde edilecektir. Oluşan görüntüde dikey eksende her satır !["\theta \in \[-30,120\]"](https://render.githubusercontent.com/render/math?math=%5ctheta%20%5cin%20%5c%5b-30%2c120%5c%5d) değerini, yatay eksen ise dönüşüm sonucu buluan !["\rho"](https://render.githubusercontent.com/render/math?math=%5crho) değerlerini göstermektedir. 

![Hough Dönüşümü][hough_transform]

Görüntüdeki parlak noktalar (tepe noktaları, doğru adayları) incelendiğinde toplam 20 nokta göze çarpmaktadır. Bu noktaların da kendi içerisinde !["\theta=0"](https://render.githubusercontent.com/render/math?math=%5ctheta%3d0) ve !["\theta=90"](https://render.githubusercontent.com/render/math?math=%5ctheta%3d90) açıları etrafında dağıldığı görülmektedir. Tahmin edileceği üzere bu tepe noktaları sudoku ızgarasını oluşturan doğru parçalarıdır. Bu parçalardan belirli bir uzunluktan (!["\tau = 0.4 * \min(width,height)"](https://render.githubusercontent.com/render/math?math=%5ctau%20%3d%200.4%20%2a%20%5cmin%28width%2cheight%29)) büyük olan (`hough` matrisinin değeri !["\tau"](https://render.githubusercontent.com/render/math?math=%5ctau) dan büyük olan) noktalar seçilip, görüntü üzerine çizdirildiğinde aşağıdaki görüntü oluşur.

| Başarılı Örnek   |  Başarılı Örnek | Başarılı Örnek | Başarısız Örnek  |
|:----------:|:-----------------:||:----------------------:|:-----------:|
![Sudoku Tespiti Örnek 1][all_lines_1] | ![Sudoku Tespiti Örnek 2][all_lines_2] | ![Sudoku Tespiti Örnek 3][all_lines_3] | ![Sudoku Tespiti Örnek 4][all_lines_4]

### Sudoku Çerçevesinin Bulunması

Bu aşamadan sonra sudoku çerçevesini oluşturan dikdörtgenin köşe noktaları tespit edilebilir. Görüntüden dikkat edileceği üzere, görüntü çok eğik bir şekilde çekilmedi ise, sudoku çerçevesini oluşturan hatlar `hough` matrisi üzerinde, !["45"](https://render.githubusercontent.com/render/math?math=45) dereceden küçük noktaların en küçük ve en büyük !["\rho"](https://render.githubusercontent.com/render/math?math=%5crho) değerli olanları sağ ve sol kenar olarak,  !["45"](https://render.githubusercontent.com/render/math?math=45) dereceden büyük noktaların en küçük ve en büyük !["\rho"](https://render.githubusercontent.com/render/math?math=%5crho) değerleri ise üst ve alt kenar doğruları olarak seçilebilir. Bu seçme işlemi tüm tepe adayları gezilerek basit bir şekilde tamamlanabilir.

Kenar doğruları bulunduktan sonra sudoku imgesinin hizalanması için bu doğruların kesişim noktalarının bulunması gerekmektedir. !["(\rho_1,\theta_1)"](https://render.githubusercontent.com/render/math?math=%28%5crho_1%2c%5ctheta_1%29) ve !["(\rho_2,\theta_2)"](https://render.githubusercontent.com/render/math?math=%28%5crho_2%2c%5ctheta_2%29) parametreleri ile verilen iki doğrunun kesişim noktası !["(x,y)"](https://render.githubusercontent.com/render/math?math=%28x%2cy%29) aşağıdaki adımlar ile bulunur.

 - Kesişim noktası iki doğru denklemini de sağlayacağından !["(x,y)"](https://render.githubusercontent.com/render/math?math=%28x%2cy%29), Hough denkleminde yerine konularak aşağıdaki eşitlikler elde edilir.

!["\begin{aligned}\rho_1 = x \cos(\theta_1) + y \sin(\theta_1)\\\rho_2 = x \cos(\theta_2) + y \sin(\theta_2)\\\end{aligned}"](https://render.githubusercontent.com/render/math?math=%5cbegin%7baligned%7d%5crho_1%20%3d%20x%20%5ccos%28%5ctheta_1%29%20%2b%20y%20%5csin%28%5ctheta_1%29%5c%5c%5crho_2%20%3d%20x%20%5ccos%28%5ctheta_2%29%20%2b%20y%20%5csin%28%5ctheta_2%29%5c%5c%5cend%7baligned%7d)

 - Burada !["x"](https://render.githubusercontent.com/render/math?math=x) bilinmeyeni her iki denklemde de yalnız bırakılıp birbirine eşitlenerek aşağıdaki ifade bulunur.
 
  !["\cos(\theta_2) \left(\rho_1 - y \sin(\theta_1) \right) = \cos(\theta_1) \left(\rho_2 - y \sin(\theta_2) \right)"](https://render.githubusercontent.com/render/math?math=%5ccos%28%5ctheta_2%29%20%5cleft%28%5crho_1%20-%20y%20%5csin%28%5ctheta_1%29%20%5cright%29%20%3d%20%5ccos%28%5ctheta_1%29%20%5cleft%28%5crho_2%20-%20y%20%5csin%28%5ctheta_2%29%20%5cright%29)
 
 - Bu ifade !["y"](https://render.githubusercontent.com/render/math?math=y) bilinmeyeni sağda kalacak şekilde toparlanarak aşağıdaki denklem elde edilir.
  
  !["\rho_1 \cos(\theta_2) - \rho_2 \cos(\theta_1) = y \left(\sin(\theta_1)\cos(\theta_2) - \sin(\theta_2)\cos(\theta_1) \right)"](https://render.githubusercontent.com/render/math?math=%5crho_1%20%5ccos%28%5ctheta_2%29%20-%20%5crho_2%20%5ccos%28%5ctheta_1%29%20%3d%20y%20%5cleft%28%5csin%28%5ctheta_1%29%5ccos%28%5ctheta_2%29%20-%20%5csin%28%5ctheta_2%29%5ccos%28%5ctheta_1%29%20%5cright%29) 
  
 - Sağda !["y"](https://render.githubusercontent.com/render/math?math=y) nin yanında kalan çarpım ifadesinin sinüs iki açı farkı şeklinde yazılarak !["y"](https://render.githubusercontent.com/render/math?math=y) değeri bulunur.
 
  !["y = \frac{\rho_1 \cos(\theta_2) - \rho_2 \cos(\theta_1)}{\sin(\theta_1-\theta_2)}"](https://render.githubusercontent.com/render/math?math=y%20%3d%20%5cfrac%7b%5crho_1%20%5ccos%28%5ctheta_2%29%20-%20%5crho_2%20%5ccos%28%5ctheta_1%29%7d%7b%5csin%28%5ctheta_1-%5ctheta_2%29%7d) 
 
  - Aynı işlem !["x"](https://render.githubusercontent.com/render/math?math=x) için de tekrarlanırsa !["x"](https://render.githubusercontent.com/render/math?math=x) değeri bulunur.
 
  !["x = \frac{\rho_2 \sin(\theta_1) - \rho_1 \sin(\theta_2)}{\sin(\theta_1-\theta_2)}"](https://render.githubusercontent.com/render/math?math=x%20%3d%20%5cfrac%7b%5crho_2%20%5csin%28%5ctheta_1%29%20-%20%5crho_1%20%5csin%28%5ctheta_2%29%7d%7b%5csin%28%5ctheta_1-%5ctheta_2%29%7d) 
 
**<span style="color: red;">NOT: </span> Burada !["\theta_1=\theta_2"](https://render.githubusercontent.com/render/math?math=%5ctheta_1%3d%5ctheta_2) olması durumunda (iki doğrunun paralel olması) paydanın sıfır olduğuna ve kesişim noktasının bulunmadığına dikkat edilmelidir.**

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
|:----------:|:-----------------:||:----------------------:|:-----------:|
![Sudoku Tespiti Örnek 1][all_edges_1] | ![Sudoku Tespiti Örnek 2][all_edges_2] | ![Sudoku Tespiti Örnek 3][all_edges_3] | ![Sudoku Tespiti Örnek 4][all_edges_4]

Bu adımdan sonra [Perspektif Dönüşümü]({% post_url 2013-12-08-perspektif-donusumu %}) yazımızda detaylarından bahsedilen perspektif dönüşümü işlemi uygulanarak sudoku bölgesinin sabit büyüklükteki bir imgeye dönüştürülmesi sağlanacaktır. Bu işlem için öncelikle dönüşüm matrisinin bulunması gerekmektedir. [Perspektif Dönüşümü]({% post_url 2013-12-08-perspektif-donusumu %}) yazımızda incelediğimiz dört nokta kullanılarak perspektif dönüşüm matrisinin bulunması işlemi IMLAB kütüphanesinde yer alan `pts2tform` fonksiyonu yardımıyla yapılabilir. Bu fonksiyon ile bulunan dönüşüm matrisi kütüphanede yer alan `imtransform` yöntemi ile girdi imgesine uygulanarak sudoku bölgesi elde edilir. Aşağıda perpektif düzeltmesi yapmak için kullanılan kod parçası verilmiştir.

```c
matrix_t *sudoku = matrix_create(uint8_t, 360, 360, 3);
struct point_t destination[4] = { {.x = 0, .y = 0},{.x = 359, .y = 0},{.x = 359, .y = 359},{.x = 0, .y = 359} };

// correct the image
matrix_t *transform = pts2tform(corners, destination, 4);
imtransform(image, transform, sudoku);
```

Verilen kodda `pts2tform` yardımı ile bir önceki adımda tespit edilen `corners` noktalarını, `destination` olarak belirlenen !["360 \times 360"](https://render.githubusercontent.com/render/math?math=360%20%5ctimes%20360) bir imgenin köşe noktalarına taşıyacak dönüşüm matrisi hesaplanmıştır. Ardından bu matris girdi imgesine uygulanarak `sudoku` imgesi elde edilmiştir. Aşağıda örnek olarak kullanılan dört imge için elde edilen çıktı imgeleri verilmiştir.

| Başarılı Örnek   |  Başarılı Örnek | Başarılı Örnek | Başarısız Örnek  |
|:----------:|:-----------------:||:----------------------:|:-----------:|
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
