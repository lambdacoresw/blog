# Autoload Nedir?
## Neden Autoload 'a İhtiyaç Var?
PHP uygulamaları geliştirirken, üçüncü taraf kütüphaneler kullanmanız gerekebilir ve bildiğiniz gibi, bu kütüphaneleri uygulamanızda kullanmak istiyorsanız, onları kaynak dosyalarınıza `require` veya `include` ifadelerini kullanarak dahil etmeniz gerekir.

Mesela `Foo` adında bir kütüphanedeki `Bar.php` sınıfını kullanmak istiyorsanız:
```php
require_once('/kutuphane_yolu/Foo/Bar.php');
```

veya

```php
include_once('/kutuphane_yolu/Foo/Bar.php');
```

şeklinde gerekli kütüphaneyi/sınıfları içeri almanız(import) gerekir.

Otomatik yükleme, PHP uygulamalarının geliştirilmesini daha verimli hale getiren önemli bir özelliktir. Üçüncü taraf kütüphanelerle çalışırken, her dosyada bu kütüphaneleri elle dahil etmek yerine, otomatik yükleme sayesinde bu işlemi otomatikleştirebiliriz. Bu, kodun daha temiz ve bakımı daha kolay hale gelmesini sağlar. Ayrıca, otomatik yükleme, uygulama boyutunu azaltabilir ve performansı artırabilir, çünkü yalnızca gerektiğinde dosyaları yükler. Bu nedenle, otomatik yükleme, PHP uygulamalarının daha düzenli, verimli ve ölçeklenebilir olmasına yardımcı olur.

Bu `require` veya `include` ifadeleri, küçük uygulamalar geliştirirken sorunsuz çalışabilir. Ancak uygulamanız büyüdükçe, `require` veya `include` ifadelerinin listesi giderek uzar ve bu durum biraz can sıkıcı ve bakımı zor hale gelir. Bu yaklaşımın diğer bir sorunu ise uygulamanıza tüm kütüphaneleri, hatta kullanmadığınız bölümleri bile yüklemiş olmanızdır. Bu durum, uygulamanız için daha fazla bellek kullanımına yol açar. Bu da uygulamanın daha ağır olmasına neden olabilir ve performansı olumsuz yönde etkileyebilir. Otomatik yükleme ise yalnızca gerektiğinde dosyaları yükler, böylece uygulamanızın bellek kullanımını optimize eder ve gereksiz yüklemeleri önler. Bu da uygulamanızın daha hafif ve daha verimli olmasını sağlar.

Bu sorunu aşmanın ideal yolu, sınıfları sadece gerçekten ihtiyaç duyulduğunda yüklemektir. İşte burada otomatik yükleme devreye giriyor. Temel olarak, uygulamanızda bir sınıf kullandığınızda, otomatik yükleyici bu sınıfın zaten yüklenip yüklenmediğini kontrol eder ve eğer yüklenmemişse, gerekli sınıfı belleğe yükler. Dolayısıyla sınıf, ihtiyaç duyulduğu yerde dinamik olarak yüklenir - buna **otomatik yükleme(autoloading)** denir. Otomatik yükleme kullandığınızda, tüm kütüphane dosyalarını manuel olarak dahil etmenize gerek yoktur; sadece otomatik yükleme mantığını içeren otomatik yükleyici dosyasını dahil etmeniz yeterlidir ve gerekli sınıflar dinamik olarak dahil edilir.

Bu makalenin ilerleyen bölümlerinde, Composer ile otomatik yükleme konusuna bakacağız. Ancak önce, Composer olmadan PHP'de nasıl otomatik yükleme yapabileceğimizi açıklayalım.

## Composer olmadan PHP'de Otomatik Yükleme Nasıl Çalışır
Belki farkında değilsiniz, ancak Composer olmadan PHP'de otomatik yükleme uygulamak mümkündür. Bu işlemi mümkün kılan fonksiyon `spl_autoload_register()` fonksiyonudur. `spl_autoload_register()` fonksiyonu, henüz yüklenmemiş sınıfları yüklemeye çalıştığında PHP'nin tetiklenmesini sağlamak üzere sıraya alınacak işlevleri kaydetmenize olanak tanır.

Bir örnekle nasıl çalıştığını gösterelim:
```php
<?php
function my_autoloader($class) {
  include 'lib/' . $class . '.php';
}
spl_autoload_register('my_autoloader');
$bar = new Bar();
?>
```

Yukarıdaki örnekte, `spl_autoload_register()` fonksiyonunu kullanarak `my_autoloader()` işlevini özel otomatik yükleyicimiz olarak kaydettik. Ardından, `Bar` sınıfını örneklemeye çalıştığınızda ve henüz mevcut değilse, PHP, kaydedilmiş tüm otomatik yükleyici işlevlerini sırayla yürütür ve böylece `my_autoloader` işlevi çağrılır - gerekli sınıf dosyasını içerir ve nihayetinde nesne örneği oluşturulur. Bu örnekte, Bar sınıfının `lib/Bar.php` dosyasında tanımlandığını varsayıyoruz.

Otomatik yükleme olmadan, `Bar` sınıf dosyasını dahil etmek için `require` veya `include` ifadesini kullanmanız gerekirdi. Yukarıdaki örnekte otomatik yükleme uygulaması oldukça basittir, ancak farklı türdeki sınıflar için birden fazla otomatik yükleyiciyi kaydederek bunu genişletebilirsiniz.

Pratikte, genellikle kendi otomatik yükleyicinizi yazmaya pek ihtiyaç duymazsınız. İşte bu noktada Composer devreye giriyor.

# Composer ile Otomatik Yükleme Nasıl Çalışır
Öncelikle, örnekleri takip etmek istiyorsanız sisteminize Composer'ı kurduğunuzdan emin olun. Composer ile otomatik yükleme yaparken, seçebileceğiniz farklı yöntemler bulunmaktadır.

Composer özellikle :
1. Dosya Otomatik Yükleme (File Autoloading),
2. Sınıf Haritası Otomatik Yükleme (Classmap Autoloading),
3. PSR-0 Otomatik Yükleme (PSR-0 Autoloading) ve 
4. PSR-4 Otomatik Yükleme (PSR-4 Autoloading)
olmak üzere dört farklı otomatik yükleme(autoload) yöntemi sağlar.

> Resmi Composer belgelerine göre, `PSR-4` '**autoloading**' için önerilen yöntemdir. 

Composer ile `'autoload'` kullanmak istediğinizde yapmanız gereken adımlar kısaca şöyle:

1. Projenizin veya kütüphanenizin kökünde `composer.json` dosyasını tanımlayın. Bu dosyada kullanmak istediğiniz otomatik yükleme yöntemine göre direktifler yazacaksınız.
2. Gerekli dosyaları oluşturmak için `composer dump-autoload` komutunu çalıştırın. Bu, dosyaları Composer'ın otomatik yükleme için kullanacağı şekilde oluşturur.
3. Otomatik yüklemeyi kullanmak istediğiniz dosyanın başına `require 'vendor/autoload.php'` ifadesini ekleyin.

Şimdi sırasıyla bu yöntemlere bakalım.

## 1.Dosya Otomatik Yükleme (File Autoloading) - `files` Direktifi
Bu yöntem, tüm kaynak dosyalarını yüklemenizi sağlayan `include` veya `require` ifadelerine benzer şekilde çalışır. `'files'` direktifi ile referans verilen tüm kaynak dosyalar, uygulamanız her çalıştığında yüklenir. Bu, sınıflar kullanmayan kaynak dosyalarını yüklemek için kullanışlıdır(örneğin `helper` yardımcı fonksiyonları için).

Dosya otomatik yüklemeyi kullanmak için, aşağıdaki örnekte gösterildiği gibi `composer.json` dosyasının `files` direktifine dosya listesini ekleyin:

```json
{
    "autoload": {
        "files": [
            "dizin_yolu/helper.php",
            "dizin_yolu/dummy.php",
            "dizin_yolu/db.php"
        ]
    }
}
```

Gördüğünüz gibi, Composer ile otomatik olarak yüklemek istediğimiz dosyaları `files` direktifi ile belirtebiliriz. Yukarıdaki kodları `composer.json` dosyanıza ekledikten sonra, gerekli otomatik yükleme dosyalarını oluşturmak için sadece `composer dump-autoload` komutunu çalıştırmanız gerekir. Bu dosyalar `vendor` dizini altında oluşturulacaktır. Son olarak, kaynak kodunuzda bu harici PHP dosyalarını kullanmak için şu satırı eklemelisiniz: 

```php
require 'vendor/autoload.php';
```

`require 'vendor/autoload.php'` ifadesi, gerekli dosyaların dinamik olarak yüklenmesini sağlar. Böylece, Composer tarafından oluşturulan otomatik yükleme(autoloading) dosyaları 'import' edilir ve projenizde kullanıma hazır hale gelir.

## 2.Sınıf Haritası Otomatik Yükleme (Classmap Autoloading) - `'classmap'` Direktifi
Sınıf haritası otomatik yükleme, dosya otomatik yükleme(files direktifi ile yükleme) yönteminin geliştirilmiş bir versiyonudur. Sadece bir dizin listesi sağlamanız yeterlidir ve Composer bu dizinlerdeki tüm dosyaları tarar. Her dosya için Composer, o dosyada bulunan sınıfların bir listesini yapar ve ihtiyaç duyulan herhangi bir sınıf için, Composer karşılık gelen dosyayı otomatik olarak yükler.

`composer.json` dosyanıza eklemeniz gereken direktif şu şekilde:

```json
{
    "autoload": {
        "classmap": ["lib"]
    }
}
```

`composer dump-autoload` komutunu çalıştırın. Composer, otomatik olarak yüklenebilecek sınıfların bir haritasını oluşturmak için `lib` dizinindeki dosyaları okuyacaktır.

## 3.PSR-0 Otomatik Yükleme (PSR-0 Autoloading) - `'psr-0'` Direktifi
PSR-0, '*autoloading*' için `PHP-FIG` grubu tarafından önerilen bir standarttır. PSR-0 standardında, kütüphanelerinizi tanımlamak için isim alanlarını(namespace) kullanmanız gerekir. Tam nitelikli sınıf adı `<Vendor Name>\(<Namespace>\)*<Class Name>` şeklinde olmalıdır.  Ayrıca, sınıf dosyalarının, isim alanlarına sahip dizin yapısını takip etmesi gerekmektedir.

`composer.json` dosyasında kullanmanız gereken yapı şu şekildedir:
```json
{
    "autoload": {
        "psr-0": {
            "VendorName\\Library": "src"
        }
    }
}
```
> Composer 'ın değişiklikleri uygulaması için `composer dump-autoload` yapmayı unutmayın!

PSR-0 *'autoloading'* işleminde, isim alanlarını(namespaces) dizinlere eşlemelisiniz. Yani `composer.json` dosyasında belirttiğiniz yol ismi ile dizin/klasör yapısı birebir aynı olmalıdır. Yukarıdaki örnekte, Composer'a `VendorName\Library` isim alanıyla başlayan her şeyin `src\VendorName\Library` dizininde bulunabilir olduğunu söylüyoruz.

Bir örnek olarak, `src\VendorName\Library` dizininde `Bar` adında bir sınıf olduğunu varsayalım. Öncelikle bu sınıfın tam yolu `src\VendorName\Library\Bar.php` şeklinde olmalıdır. Sonrasında `Bar.php` sınıfı şu şekilde başlamalıdır:

```php
<?php
namespace VendorName\Library;
class Bar
{
    //...... 
}
?>
```

Gördüğünüz gibi `Bar` sınıfı, `VendorName\Library` isim alanı(namespace) içerisinde tanımlanmıştır.

Ve tabi bu sınıfı kendi PHP kaynak kodunuzda kullanmak için yapmanız gereken:

```php
<?php
require 'vendor/autoload.php';
 
$bar = new VendorName\Library\Bar();
?>
```

Böylece Composer, `Bar` sınıfını `src\VendorName\Library` dizinine giderek otomatik bir şekilde bulup kullanımınıza sunacaktır.

## PSR-4 Otomatik Yükleme (PSR-4 Autoloading) - `'psr-4'` Direktifi
PSR-4, isim alanlarını(namespace) kullanmanızı gerektiren PSR-0 otomatik yüklemesine benzerdir, ancak isim alanlarını dizin yapısıyla birebir eşlemenize gerek yoktur.

PSR-0 '*autoloading*' işleminde, isim alanlarını(namespace) dizin yapısına birebir eşlemek zorundasınız. `VendorName\Library\Bar` sınıfını otomatik olarak yüklemek istiyorsanız, bu sınıfın `src\VendorName\Library\Bar.php` dosyasında bulunması gerekir. PSR-4 otomatik yüklemesinde ise dizin yapısını kısaltabilirsiniz, bu da PSR-0 otomatik yüklemesine göre çok daha basit bir dizin yapısı sağlar.

`composer.json` dosyanıza eklemeniz gereken kod:
```json
{
    "autoload": {
        "psr-4": {
            "VendorName\\Library\\": "src"
        }
    }
}
```

> PSR-0 direktifine çok benziyor ama 'Library' kelimesinden sonra eklenen çift ters bölü işaretlerine dikkat edin. PSR-0 'da yok.

Bu yöntemle, Composer'a `VendorName\Library` isim alanıyla başlayan her şeyin `src` dizininde bulunabileceğini söyleriz. Böylece, `VendorName\Library` dizinlerini oluşturmanıza gerek kalmaz. 
Örneğin, `VendorName\Library\Bar` sınıfını istediğinizde, Composer `src\Bar.php` dosyasını yüklemeye çalışacaktır.

> Tekrar söylemek gerekirse PSR-4, PSR-0 ile hemen hemen aynıdır. PSR-0 yönteminde isim alanları(namespace) ile dizin/klasör yapısı birebir aynı olması gerekirken; PSR-4 yönteminde isim alanları(namespace) ile dizin/klasör yapısı birebir aynı olmak zorunda değildir. PSR-0 'da `src\VendorName\Library\Bar.php` dosyası aynen `src\VendorName\Library` klasöründe olması gerekirken, PSR-4 'te böyle bir zorunluluk yoktur, istenen klasör yapısı kullanılabilir. Ama dikkat edilmesi gereken nokta, her iki yöntemde de `Bar` sınıfında 'namespace' bildirimini yapmanız gerekmesidir. Kısaca her iki yöntemde de 'namespace' bildirimi aynıdır: `namespace VendorName\Library`

Görüldüğü gibi PSR-4 yöntemi daha esnek ve temiz bir yöntem sağlar, iç içe klasörler oluşturmanız gerekmez.

PSR-4, otomatik yükleme için önerilen yöntemdir ve PHP topluluğunda geniş ölçüde kabul görmektedir.

# Son Sözler
Genel olarak PHP ile otomatik yükleme(autoload) işlemleri bu şekilde yapılmaktadır. Başka yöntemlerde olmasına rağmen, en çok kullanılanları bu 4 yöntemdir, özellikle PSR-4. Zaten güncel PHP kütüphanelerini incelerseniz(Laravel, Symfony, ...) bu yöntemleri nasıl kullandıklarını daha detaylı bir şekilde görebilirsiniz. 