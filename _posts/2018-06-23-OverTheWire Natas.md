---
layout: post
title: "OverTheWire Natas"
date: 2018-06-26
excerpt: "26 Jun 2018"
comments: false
---

Natas, sunucu taraflı web güvenlik temellerini öğretmek amacıyla hazırlanmış, basit düzeyden baya ileriye kadar seviyelendirilmiş bir challenge diyebiliriz.Toplamda 32 bölümden oluşuyor ve her bölüme bu url "http://natasX.natas.labs.overthewire.org" üzerinden erişiyoruz.Daha ayrıntılı bilgiye [buradan](http://overthewire.org/wargames/natas/) ulaşabilirsiniz.
#### Level 0
natas0:natas0 username passwordü ile giriş yapıyoruz.Bize verilen hint "You can find the password for the next level on this page." şeklinde.Kaynak kodu inceliyoruz.

>> natas1:gtVrDuiDfck831PqWsLEZy5gyDz1clto

#### Level 1
"You can find the password for the next level on this page, but rightclicking has been blocked!"
Kaynak kodu incelemek için rightclick yerine, F12 Developer Tools kullanıyoruz.
>> natas2:ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi

#### Level 2
"There is nothing on this page."
Kaynak kodu incelediğimizde files/pixel.png şeklinde bir path ile karşılaşıyoruz. files/ dizinine gittiğimizde "Directory Indexing" zafiyeti sayesinde users.txt dosyasına erişiyoruz.
>> natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14

#### Level 3
Yine "There is nothing on this page" şeklinde bir hint verilmiş. Kaynak kodunu incelediğimizde "No more information leaks!! Not even Google will find it this time..." ile karşılaşıyoruz.Google bile bulamaz ifadesi bize tabi ki robots.txt'yi çağrıştırıyor.http://natas3.natas.labs.overthewire.org/robots.txt adresine gittiğimizde /s3cr3t/ dizinini görüyoruz.Dizine gittiğimizde yine "Directory Indexing" zafiyeti sayesinde users.txt dosyasına erişiyoruz.
>> natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ

#### Level 4
"Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/" ".

Bu açıklamadan HTTP headerları ile ilgilenmemiz gerektiğini anlıyoruz."Refresh page" linkine tıkladığımızda isteğe "referrer" başlığı ekleniyor.Bu başlık değerini soruda bizden istenen şekilde düzenlediğimizde level 5 için parolayı elde ediyoruz.
<figure>
<img src="/assets/img/natas/natas41.png">
</figure>
<figure>
<img src="/assets/img/natas/natas42.png">
</figure>
>>natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq

#### Level 5
"Access disallowed. You are not logged in" 
Burp ile isteği daha ayrıntılı incelediğimizde cookie başlığında bulunan "loggedin" değerini 1 yaptığımızda level 6 için parolayı elde ediyoruz.
<figure>
<img src="/assets/img/natas/natas51">
</figure>
<figure>
<img src="/assets/img/natas/natas52.png">
</figure>
>>natas6:aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1

#### Level 6
Basit bir form bizi karşılıyor.Önceki levellerden farklı olarak server tarafında çalışan kodları da görmemiz mümkün. Kodu incelediğimizde include komutu ile includes/secret.inc sayfasının bu koda dahil edildiğini görüyoruz.Sayfaya gittiğimizde secret değişkeninin burada tanımlanmış olduğunu görüyoruz.Değişkenin değerini input olarak girdiğimizde level 7 için parolayı elde etmiş oluyoruz.

>> natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9

#### Level 7

Home ve About sayfalarına götüren iki link mevcut.Linklerin birine tıkladığımızda, get isteği  ile page parametresiyle alınan değer sayfa içerisine include ediliyor. LFI zafiyeti sayesinde /etc/natas_webpass/natas8 dosyasını page parametresine verdiğimizde level 8 için parolayı elde diyoruz.
>> natas8:DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe

#### Level 8

{% highlight php %}
<?
$encodedSecret = "3d3d516343746d4d6d6c315669563362";
function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}
if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
{% endhighlight %}

POST isteği ile alınan değer, sırasıyla base64_encode(),strrev(),bin2hex() fonksiyonlarına tabi tutulması sonucu elde edilen değerin  $encodedSecret değişkenine eşit olması halinde natas9 için parolayı elde edebileceğiz.
{% highlight php %}
<?
$encodedSecret = "3d3d516343746d4d6d6c315669563362";
echo base64_decode(strrev(hex2bin($encodedSecret));
?>
{% endhighlight % }
