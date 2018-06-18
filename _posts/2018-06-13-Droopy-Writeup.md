---
layout: post
title: "Droopy Writeup"
date: 2018-06-13
excerpt: "13 Jun 2018"
comments: false
---
Droopy imajını indirmek/daha ayrıntılı bilgiye ulaşmak için [bu](https://www.vulnhub.com/entry/droopy-v02,143/) linki kullanabilirsiniz.
Netdiscover ile bulunduğumuz ağı tarayarak makinaya ait ip adresini 192.168.1.48 olarak buluyoruz.
<figure >
    <img src="/assets/img/droopy/droopip.png">
</figure>
Nmap taraması ile açık portları ve üzerinde çalışan servisleri öğreniyoruz.Makina üzerinde yalnızca web servisi çalışıyor.
<figure >
    <img src="/assets/img/droopy/droopynmap.png">
</figure>
http://192.168.1.48 adresini ziyaret ettiğimizde bizi bir login ekranı karşılıyor. Makina için " Grab a copy of the rockyou wordlist" şeklinde bir hint verilmiş olması ilerisi hakkında fikir veriyor. Admin:admin gibi default username:password ikililerini deniyoruz. Fakat default olarak bırakılmamış.
Daha fazla bilgi toplayabilir miyiz düşüncesiyle fuzzing yapıyoruz. 
<figure >
    <img src="/assets/img/droopy/droopydirb.png">
</figure>
İlk ziyaret ettiğimiz /robots.txt oluyor.Birden fazla entry ile karşılaşıyoruz. Dizinleri ziyaret ettiğimizde ilk etapta çok işe yarar bir şey yok gibi görünüyor.
Anasayfada ve dizinlerde karşımıza çıkan Drupal üzerine kısa bir araştırma yapıyoruz.
```
Drupal ücretsiz, açık kaynaklı bir içerik yönetim sistemi ya da içerik yönetime odaklı bir altyapı yazılımıdır.
```
Sisteme ait versiyon bilgisini öğrenebilmek için robots.txt de bulunan dosyalardan bazılarını tekrar gözden geçiriyoruz.http://192.168.1.48/CHANGELOG.txt adresini ziyaret ettiğimizde versiyon bilgisi hakkında bilgi sahibi oluyoruz.
<figure >
    <img src="/assets/img/droopy/droopyversion.png">
</figure>
Drupal 7.30 için exploit araştırdığımızda Drupal 7.X versiyonlarına ait bir SQLi zafiyetinin bulunduğunu görüyoruz.Ve bu zafiyet sayesinde admin kullanıcısının parolasının değiştirmek mümkün oluyor.[Bu github adresinde ki](https://gist.github.com/milankragujevic/61eb72df71b69df80e86) php kodunu çalıştırıyoruz.
<figure >
    <img src="/assets/img/droopy/droopyexploit.png"><p>Yeni IP adresi: 192.168.1.49</p></img>
</figure>
Admin kullanıcısına ait parola "admin" olarak değiştirilmiş oldu.Böylece admin:admin ikilisi ile giriş yapabiliyoruz.
Drupal servisinin PHP tabanlı çalıştığını biliyoruz. Bize reverse shell vereceh PHP kodunu ekleyecek uygun bir yer arıyoruz.Dizin yapısını bilmememiz  sebebiyle kısa bir araştırma yapıyoruz. 
Öncelikle Modules kısmından PHP Filter modülünü etkinleştiriyoruz.
Ardından Content>Add Content>Article kısmından, [adresindeki](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) php reverse shell kodunu ekliyoruz.Text formatını PHP Code olarak değiştiriyoruz.Değişikleri kaydettiğimizde www-data kullanıcısı ile bağlantı elde etmiş oluyoruz.
<figure >
    <img src="/assets/img/droopy/droopyshell.png">
</figure>
Çekirdek versiyonunu öğrendikten sonra, bu versiyon için local exploit araştırıyoruz.
<figure >
    <img src="/assets/img/droopy/droopykernel.png">
</figure>
3.13 çekirdek versiyonu için "CVE-2015-1328 Overlayfs Privilege Escalation" exploiti mevcut. [Github adresinde](https://github.com/lucyoa/kernel-exploits/tree/master/overlayfs) ki exploiti çalıştırıyoruz.
Böylece root yetkilerine erişmiş oluyoruz.
<figure >
    <img src="/assets/img/droopy/droopyroot.png">
</figure>
<figure >
    <img src="/assets/img/droopy/hello.gif">
</figure>
