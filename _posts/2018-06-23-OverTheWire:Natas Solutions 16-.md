---
layout: post
title: "OverTheWire:Natas Solutions 0-15"
date: 2018-06-26
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
