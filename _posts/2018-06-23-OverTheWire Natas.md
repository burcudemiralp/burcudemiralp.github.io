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
