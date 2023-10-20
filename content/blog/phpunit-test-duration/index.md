---
title: PHPUnit ile Testlerde Süre Limiti
date: "2023-07-24T22:12:03.284Z"
---

Merhaba, bu yazımda PHPUnit'e extension yazma ve çalışan testlerin belirli bir süre sonrasında hata vermesini sağlayacağız.

> PHPUnit: PHP için kullanılan bir birim testi aracıdır.

> Birim Test: Birim testi, bir uygulamanın birim adı verilen test edilebilir en küçük parçalarının düzgün çalışması için ayrı ayrı incelendiği bir yazılım geliştirme sürecidir.

Öncelikle extension içerisinde kullanacağız ve bize yardımcı olacak bir sınıf oluşturmamız gerekiyor.

```php
<?php  

namespace Tests;  
 
final class TimeoutMaxDuration  
{  
    // Default olarak her bir testin süreceği maksimum süreyi belirtiyor.
    private const DEFAULT = 10; // seconds.  
      
    public static int $timeout = self::DEFAULT;  
      
    /**  
     * Çalışan her bir testten sonrasında süreyi sıfırlamak için kullanacağız.
     */
    public static function resetTimeout(): void  
    {  
        self::$timeout = self::DEFAULT;  
    }  
}
```

Yukarıda oluşturduğumuz `TimeoutMaxDuration` sınıfı `Extension` geliştirmesini yaparken kullanacağımız sınıf.

Bunu oluşturduktan sonra `TimeoutPHPUnitExtension` adında bir sınıf oluşturuyoruz. Bu sınıf her test bittikten sonra çalışacak ve testin ne kadar sürdüğünü karşılaştıracak. Bunun için PHPUnit'in `AfterTestHook` interface'ini implement etmemiz gerekiyor. Bu sayede `executeAfterTest` methodunu da kullanarak her test çalıştıktan sonra yapmak istediğimiz işi yazabiliriz.

```php
<?php  
  
namespace Tests;  
  
use PHPUnit\Framework\TestCase;  
use PHPUnit\Runner\AfterTestHook;  
  
class TimeoutPHPUnitExtension extends TestCase implements AfterTestHook  
{  
    public function executeAfterTest(string $test, float $time): void  
    {  
        $maxDuration = TimeoutMaxDuration::$timeout;  
        self::assertLessThan(  
            $maxDuration,  
            $time,  
            sprintf(  
                'The test "%s" was too long, it tooks "%f" seconds and the maximum allowed is "%d"',  
                $test,  
                $time,  
                $maxDuration  
            )  
        );  
      
        TimeoutMaxDuration::resetTimeout();  
    }  
}
```

Yukarıdaki kodu incelediğimiz zaman yaptığı işler şu şekilde sıralanıyor.
1. `TimeoutMaxDuration::$timeout` ile kullanıcının tanımladığı (veya default olarak tanımlanan) maksimum süreyi alıyor.
2. PHPUnit'in `assertLessThan` methodunu kullanarak testin çalıştığı süre ile bizim tanımladığımız maksimum süre ile karşılaştıyor ve testin çalışma süresi, bizim tanımladığımız bizim timeout süremizden daha büyükse bir hata oluşturuyor.
3. Her test bittikten sonra timeout değerini default olarak güncelliyor.

Şu ana kadar yaptığımız işlemler extension'ımızı oluşturmamız sağladı. Bu extension'ın çalıştırılması için phpunit'in konfigürasyon (phpunit.xml) dosyasında tanımlamasını yapmamız gerekiyor.

```diff
<?xml version="1.0" encoding="UTF-8"?>  
<phpunit>  
    ...
+    <extensions>
+        <extension class="Tests\TimeoutPHPUnitExtension" />
+    </extensions>
</phpunit>
```

Extension olarak ekledikten sonra testinizi çalıştırıp süreleri kontrol edebilir ve uzun süren testler için güncellemelerinizi yapabilirsiniz.

Eğer özel olarak bir test için farklı bir timeout süresi vermek isterseniz, ilgili test kodu içerisinde timeout süresini değiştirmeniz mümkün.

```php
<?php  
  
namespace Tests\Unit;  
  
use Tests\TestCase;  
use Tests\TimeoutMaxDuration;  
  
class ExampleTest extends TestCase 
{
    public function exampleTestCase(): void
    {
        TimeoutMaxDuration::$timeout = 100;
        ...
    }
}
```
