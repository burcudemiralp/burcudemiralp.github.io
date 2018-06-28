---
layout: post
title: "OverTheWire Natas"
date: 2018-06-26
excerpt: "26 Jun 2018"
comments: false
---

Natas, sunucu taraflı web güvenlik temellerini öğretmek amacıyla hazırlanmış, basit düzeyden baya ileriye kadar seviyelendirilmiş bir challenge diyebiliriz.Toplamda 32 bölümden oluşuyor ve her bölüme bu url "http://natasX.natas.labs.overthewire.org" üzerinden erişiyoruz.Daha ayrıntılı bilgiye [buradan](http://overthewire.org/wargames/natas/) ulaşabilirsiniz.


[Level 0](#level-0)|[Level 1](#level-1)

[Level 2](#level-2)|[Level 3](#level-3)

[Level 4](#level-4)|[Level 5](#level-5)

[Level 6](#level-6)|[Level 7](#level-7)

[Level 8](#level-8)|[Level 9](#level-9)

[Level 10](#level-10)|[Level 11](#level-11)

#### Level 0
natas0:natas0 username password'ü ile giriş yapıyoruz.Bize verilen hint "You can find the password for the next level on this page." şeklinde.Kaynak kodu inceliyoruz.

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
Basit bir form bizi karşılıyor.Önceki levellerden farklı olarak server tarafında çalışan kodları da görmemiz mümkün. Kodu incelediğimizde include komutu ile includes/secret.inc sayfasının bu sayfaya dahil edildiğini ve  secret değişkeninin secret.ini sayfasında  tanımlanmış olduğunu görüyoruz.Değişkenin değerini input olarak girdiğimizde level 7 için parolayı elde etmiş oluyoruz.

>> natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9

#### Level 7

Home ve About sayfalarına götüren iki link mevcut.Linklerin birine tıkladığımızda, page parametresiyle alınan değer sayfa içerisine include ediliyor. LFI zafiyeti sayesinde /etc/natas_webpass/natas8 dosyasını page parametresine verdiğimizde level 8 için parolayı elde diyoruz.
>> natas8:DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe

#### Level 8
Kaynak kod şu şekilde:
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

POST isteği ile alınan değer, sırasıyla base64_encode(),strrev(),bin2hex() fonksiyonlarına tabi tutulması sonucu elde edilen değerin  $encodedSecret değişkenine eşit olması halinde natas9 için parolayı elde edebileceğiz.$encodedString değişkenine sırasıyla hex2bin(),strrev(),base64_decode() fonksiyonlarını uyguladığımızda oubWYf2kBq değerini elde ediyoruz.
>> natas9:W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl

#### Level 9
Kaynak kod şu şekilde:
{% highlight php %}
<?

$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
{% endhighlight %}
İstemciden alınan değer $key değişkenine atanıyor ve bu değişken doğrudan passthru fonksiyonu içerisindeki ifadeye yerleştiriliyor.Passthru, system fonksiyonu gibi sistem komutları çalıştırıyor. Yani kod "Command Injection" zafiyeti barındırıyor. 

"; cat /etc/natas_webpass/natas10" # " şeklinde bir input girdiğimizde,passthru fonksiyonu içerisinde ki ifade
 grep -i ; cat /etc/natas_webpass/natas10" #  dictionary.txt gibi bir hal alıyor.
 
 >> natas:nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu

#### Level 10

 Level 9 ile çok benzer bir senaryo.Fakat "For security reasons, we now filter on certain characters" şeklinde bir mesaj bırakılmış.
 {% highlight php %}
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
{% endhighlight %}
Linux sistemler için iki komutu birbirinden ayırabilecek ";","|" gibi karakterler filtrelenmiş. Burda atlanmaması gereken nokta grep komutunun dosya okuyabileceği.Yani /etc/natas_webpass/natas11 dosyasını grep ile okuyacağız.
"a /etc/natas_webpass/natas11 #" şeklinde bir input girdiğimizde, passthru fonksiyonu içerisinde ki ifade
"grep -i a /etc/natas_webpass/natas11 # dictionary.txt" şeklini alıyor.Yani /etc/natas_webpass/natas11 dosyasında içerisinde a veya A harfi geçen satırları getiriyor.Parola içerisinde a harfi mevcutsa, parolaya ulaşmış olacağız. A için başarısız oluyor, c harfi için aynı payloadı yazdığımızda level 11 için parolayı elde ediyoruz.
>> natas11:U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK

#### Level 11

<figure>
<img src="/assets/img/natas/natas111.png">
</figure>

Genel bir fikir sahibi olmak adına fonksiyonları inceliyoruz.

<figure>
<img src="/assets/img/natas/natas112.png">
</figure>

İlk olarak loadData fonksiyonu çağırılıyor.

 {% highlight php %}
<?
     function loadData($def) {
        global $_COOKIE;
      $mydata = $def;
      if(array_key_exists("data", $_COOKIE)) {
      $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
      if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
           if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
           $mydata['showpassword'] = $tempdata['showpassword'];
           $mydata['bgcolor'] = $tempdata['bgcolor'];
           }
      }
      }
      return $mydata;
      }
?>
{% endhighlight %}


Eğer cookie başlığında "data"  mevcut ise, bu değer base64_decode(),xor_encyrpt(),json_decode() işlemlerinden geçtikten sonra $data değişkenine atanıyor. Data değerinin bulunmaması durumunda  defaultdata değişkeni kullanılmaya devam ediyor.

Daha sonra çağırılan saveData fonksiyonunda;

 {% highlight php %}
<?
  function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
    }
     
?>
{% endhighlight %}

Data değişkeni json_encode(),xor_encrypt() ve base64_encode() işlemlerinden geçtikten sonra cookie başlığına ekleniyor.

Bizi ilgilendiren kısım ise burası.

<figure>
<img src="/assets/img/natas/natas113.png">
</figure>


Data değişkeninde "showpassword" değeri "yes" e eşitse, level 12 için parola ekrana bastırılıyor.Yapmamız gereken cookie değerini düzenleyip "showpassword" değerini "yes" e eşitlemek.

Cookie yapısını biliyoruz : array( "showpassword"=>"no", "bgcolor"=>"#ffffff").

Bu yapıyı gerektiği gibi encode edebilirsek istediğimizi elde etmiş olacağız.Bunun için saveData fonksiyonunda ki sıralamayı takip etmemiz gerekiyor.

Xor_encrypt fonksiyonu nasıl çalışıyor buna bakıyoruz.
 {% highlight php %}
<?
      function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
    }
?>
{% endhighlight %}

Her şey iyi hoş fakat şifrelemede kullanılan keyi bilmiyoruz.Biraz araştırma sonucu aşağıdaki denkleme ulaşıyoruz.


``` 
    plaintext XOR key        = ciphertext
    plaintext XOR ciphertext = key
```


Burada plaintext'i "array( "showpassword"=>"no", "bgcolor"=>"#ffffff")" ve ciphertext'i yani cookie değerini biliyoruz.İki değeri xor_encyrpt() fonksiyonu ile şifrelediğimizde keyi elde etmiş olacağız.

 {% highlight php %}
<?
     $key = json_encode(array( "showpassword"=>"no", "bgcolor"=>"#ffffff"));
     $text = base64_decode('ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=');
     $outText = '';
     // Iterate through each character
      for($i=0;$i<strlen($text);$i++) {
      $outText .= $text[$i] ^ $key[$i % strlen($key)];
             }
     echo $outText;
?>
{% endhighlight %}

Burdan "qw8J" keyine ulaşıyoruz. Geriye istediğimiz cookie değerini bu key ile şifrelemek kalıyor.

 {% highlight php %}
<?
        function xor_encrypt($in) {
        $key = 'qw8J';
        $text = $in;
        $outText = '';

        // Iterate through each character
        for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
        }
        return $outText;
        }

        echo  base64_encode(xor_encrypt(json_encode(array( "showpassword"=>"yes", "bgcolor"=>"#ffffff"))));
    
?>
{% endhighlight %}

Bu scriptin çalıştırılması sonucu "ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK" cookie değerini elde ediyoruz.


<figure>
<img src="/assets/img/natas/natas114.png">
</figure>

<figure>
<img src="/assets/img/natas/natas115.png">
</figure>

>> natas12:EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3

#### Level 12

Dosya yükleyebileceğimiz bir ekran karşılıyor bizi.

<figure>
<img src="/assets/img/natas/natas121.png">
</figure>

.php uzantılı bir dosya atmayı deniyorum.

<figure>
<img src="/assets/img/natas/natas122.png">
</figure>

<figure>
<img src="/assets/img/natas/natas123.png">
</figure>

Yüklediğim dosyanın ismi ve uzantısından bağımsız olarak kaydedildi,jpg formatında olmadığı için  yüklediği adrese gittiğimde dosyaya erişemedim.


 {% highlight php %}
<?
function makeRandomPath($dir, $ext) { 
    do { 
    $path = $dir."/".genRandomString().".".$ext; 
    } while(file_exists($path)); 
    return $path; 
} 

function makeRandomPathFromFilename($dir, $fn) { 
    $ext = pathinfo($fn, PATHINFO_EXTENSION); 
    return makeRandomPath($dir, $ext); 
} 

if(array_key_exists("filename", $_POST)) { 
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]); 


        if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) { 
        echo "File is too big"; 
    } else { 
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) { 
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded"; 
        } else{ 
            echo "There was an error uploading the file, please try again!"; 
        } 
    } 
} else {
?>
{% endhighlight %}

Kaynak koda göz attığımızda, dosyaların upload/dizini altına, random bir string ile kaydedildiğini görüyoruz.Dosya uzantısıda  post isteğindeki "filename" parametresinden belirleniyor.Dosyanın kaydedileceği path belirlendikten sonra, eğer boyutu 1000 byte dan büyük değilse hedef dizine kaydediliyor.Bununla birlikte hedef dizin ekrana bastırılıyor.

php uzantılı bir dosya kaydedebilmek için post isteğinde ki "filename" parametresini düzenlememiz gerekiyor.


```
 $_FILES karşıdan yüklenen tüm dosyaların bilgisini içerir.
 
 $_FILES['kullanici_dosyasi']['tmp_name'] Yüklenen dosyanın sunucuda saklandığı sıradaki geçici dosya adıdır.

```

<figure>
<img src="/assets/img/natas/natas126.png">
</figure>


<figure>
<img src="/assets/img/natas/natas127.png">
</figure>
İlgili dizine gittiğimizde level 13'e ait parolayı öğreniyoruz.

>> natas13:jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY
