---
layout: post
title: "Quaoar Writeup"
date: 2018-06-07
excerpt: ""
comments: false
---
Quaoar isimli zafiyetli makinada root yetkilerini elde etmemiz ve bu süreçte 3 flag bulmamız isteniyor. Başlangıç seviyesi olarak kategorize edilmiş.Daha ayrıntılı bilgiye [buradan](https://www.vulnhub.com/entry/hackfest2016-quaoar,180/) ulaşabilirsiniz.

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
192.168.1.50/robots.txt adresini ziyaret ettiğimizde iki entry ile karşılaşıyoruz , wordpress in varlığını zaten biliyorduk. Wordpress dizinine gittiğimizde bizi bir blog karşılıyor. Manuel olarak bir keşif yapmadan  wpscan ile tema ve pluginlerin barındarabileceği zafiyetleri kontrol ediyoruz. 
#### wpscan --url http://192.168.1.46/wordpress/ --enumerate vp  
Tarama sonucu, zafiyetli iki plugin olduğunu görüyoruz.
<figure >
    <img src="/assets/img/quaorawpscan.png">
</figure>
Sql injection zafiyeti için [exploit-db](https://www.exploit-db.com/exploits/41438/) üzerinden daha ayrıntılı bir araştırma yaptığımızda zafiyet barındıran sayfaların silinmiş olduğunu görüyoruz.
LFI zafiyeti için yine [exploit-db](https://www.exploit-db.com/exploits/40290/) üzerinden bir araştırma yapıyoruz.
Payload şu şekilde:

http://server/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd.
<figure >
    <img src="/assets/img/quaoralfi.png">
</figure>
/etc/passwd dosyasından wpadmin kullanıcısını öğrenmiş oluyoruz.Bunun dışında [bu github hesabından](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion%20-%20Path%20Traversal) ulaştığımız , işimize yarabilecek payloadlar deniyoruz. Fakat işimize yarar pek bir şey çıkmıyor.wp-config.php sayfası için birkaç deneme yapıyoruz fakat muhtemelen /wordpress dizininden başka bir yere taşınmış.
