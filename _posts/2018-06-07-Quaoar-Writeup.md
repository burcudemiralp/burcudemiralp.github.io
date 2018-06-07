---
layout: post
title: "Quaoar Writeup"
date: 2018-06-07
excerpt: ""
comments: false
---
Quaroa isimli zafiyetli makinada root yetkilerini elde etmemiz ve bu süreçte 3 flag bulmamız isteniyor. Başlangıç seviyesi olarak kategorize edilmiş.Daha ayrıntılı bilgiye [buradan](https://www.vulnhub.com/entry/hackfest2016-quaoar,180/) ulaşabilirsiniz.

Öncelikle netdiscover ile bulunduğumuz ağı tarayıp, makinaya ait ip adresini öğreniyoruz.
<figure >
    <img src="/assets/img/quaoraip.png">
</figure>
Basit bir nmap taraması ile açık portları ve çalışan servisleri öğreniyoruz.
<figure >
    <img src="/assets/img/quaoranmap.png">
</figure>
Tabi ki ilk yöneldiğmiz web servisi  oluyor. Ziyaret ettiğimizde oldukça basit bir sayfa bizi karşılıyor. Kaynak kodunda da dikkatimizi çeken herhangi bir şey yok.
Nikto ile yaptığımız tarama sonucu , robots.txt nin varlığını ve wordpress kurulu olduğunu öğrenmiş oluyoruz.
<figure >
  <a href="/assets/img/quaoranikto.png"><img src="/assets/img/quaoranikto.png"></a>
</figure>
