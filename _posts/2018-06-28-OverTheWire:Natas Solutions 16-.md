---
layout: post
title: "OverTheWire:Natas Solutions 0-15"
date: 2018-06-28
excerpt: "26 Jun 2018"
comments: false
---
#### Level 16

For security reasons, we now filter even more on certain characters.


<figure>
<img src="/assets/img/natas/natas161.png">
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

grep -i " " dictionary.txt burada çift tırnaklar arasında natas17 parolası olmuş olucaktı ve bu komutun çıktısı boş olucaktı.
