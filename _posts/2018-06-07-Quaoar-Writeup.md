---
layout: post
title: "Quaoar Writeup"
date: 2018-06-07
excerpt: "07 Jun 2018"
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
Tabi ki ilk yöneldiğimiz web servisi  oluyor. Ziyaret ettiğimizde oldukça basit bir sayfa bizi karşılıyor. Kaynak kodunda da dikkatimizi çeken herhangi bir şey yok.
Nikto ile yaptığımız tarama sonucu , robots.txt nin varlığını ve wordpress kurulu olduğunu öğreniyoruz.
192.168.1.50/robots.txt adresini ziyaret ettiğimizde iki entry ile karşılaşıyoruz , wordpress in varlığını zaten biliyorduk. Wordpress dizinine gittiğimizde bizi bir blog karşılıyor. Manuel olarak bir keşif yapmadan  wpscan ile tema ve pluginlerin barındarabileceği zafiyetlere bakıyoruz. 
```
wpscan --url http://192.168.1.50/wordpress/ --enumerate vp
```
Tarama sonucu, zafiyetli iki plugin olduğunu görüyoruz.
<figure >
    <img src="/assets/img/quaorawpscan.png">
</figure>
Sql injection zafiyeti için [exploit-db](https://www.exploit-db.com/exploits/41438/) üzerinden daha ayrıntılı bir araştırma yaptığımızda zafiyet barındıran sayfaların silinmiş olduğunu görüyoruz.
LFI zafiyeti için yine [exploit-db](https://www.exploit-db.com/exploits/40290/) üzerinden bir araştırma yapıyoruz.
Payload şu şekilde:
"http://server/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd".
<figure >
    <img src="/assets/img/quaoralfi.png">
</figure>
/etc/passwd dosyasından wpadmin kullanıcısını öğrenmiş oluyoruz.Bunun dışında [bu github hesabından](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion%20-%20Path%20Traversal) ulaştığımız , işimize yarayabilecek payloadlar deniyoruz. Fakat işimize yarar pek bir şey çıkmıyor. Wp-config.php sayfası için birkaç deneme yapıyoruz fakat muhtemelen /wordpress dizininden başka bir yere taşınmış.
İkinci plugin de XSS zafiyeti barındırıyor.
<figure >
    <img src="/assets/img/quaorawpscan2.png">
</figure>
Fakat zafiyet barındıran sayfa silinmiş. Buradan da çok bir şey elde edemeyip, temaları tarıyoruz. 
```
wpscan --url http://192.168.1.50/wordpress/ --enumerate vt
```
Zafiyetli herhangi bir tema bulunmadığını da böylece görmüş oluyoruz.Ardından mevcut kullanıcıları öğrenebilmek için yine bir tarama yapıyoruz. 
```
wpscan --url http://192.168.1.50/wordpress/ --enumerate u
```
<figure >
    <img src="/assets/img/quaorawpscan3.png">
</figure>
Admin ve wpadmin şeklinde iki kullanıcının bulunduğunu görüyoruz. Hemen altında da admin default username'inin hala kullanıldığını söylüyor. Aslında burda aklımıza gelen ilk şeyin admin:admin,admin:123456 gibi ikilileri denemek olması gerekirdi. Bunun yerine bilgisayarıma işkence etmeyip seçip, brufe force ile admin:admin ikilisini elde ettim.
```
hydra -l admin -P rockyou.txt -vV -f -t 2 192.168.1.50 http-post-form "/wordpress/wp-login.php:log=^USER^&pwd=^PASS^:login_error"
```
<figure >
    <img src="/assets/img/giphy.gif">
</figure>
Wp-admin paneline eriştikten sonra, ilk işimiz reverse shell yüklemek oluyor.Bunun için appearance>editor kısmından bir temayı seçip,herhangi bir template'e bize reverse shell verecek olan [php kodumuzu](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) ekliyoruz. 
Bir dinleme başlatıp, kendi php kodumuzu eklemiş olduğumuz sayfayı tarayıcıdan çağırdığımızda www-data kullanıcısı ile shell elde etmiş oluyoruz.
<figure >
    <img src="/assets/img/quaorashell.png">
</figure>
Aldığımız shell'i etkileşimli  hale getirdikten sonra wpadmin kullanıcısının home dizininde ilk flag ile karşılaşıyoruz.
<figure >
    <img src="/assets/img/quaoraspawn.png">
</figure>
Flag değerini md5 decode ediyoruz:QuaoarWordpress

Ardından sistem üzerinde bilgi toplamak için [github adresindeki](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh) scripti çalıştırıyoruz.
İlk olarak bu çekirdek versiyonu için local bir exploit olup olmadığını araştırıyoruz. C ile yazılmış birden fazla local exploit bulunmasına rağmen, içeride gcc bulunmaması sebebiyle işimize yaramıyor.
<figure >
    <img src="/assets/img/quaoarsystem.png">
</figure>
Post exploitation için best practice olarak kabul edilen şeylerden birini yapıp, var olan cron dosyalarını inceliyoruz.
<figure >
    <img src="/assets/img/quaoracron.png">
</figure>
<figure >
    <img src="/assets/img/quaoracron2.png">
</figure>
İkinci flagi de böylece elde etmiş oluyoruz. Sistem üzerinde biraz daha gezindikten sonra,en başta ulaşmaya çalıştığımız wp-config dosyasının yerini arıyoruz.
<figure >
    <img src="/assets/img/quaorafind.png">
</figure>
Dosyayı okuduğumuzda database bağlantısının gerçekleştiğini ve database için root:rootpassword! ikilisini öğrenmiş oluyoruz.
<figure >
    <img src="/assets/img/quaoracrential.png">
</figure>
Database için kullanılan parolanın başka yerlerdede kullabileceğini düşünerek şansımızı deniyoruz.
<figure >
    <img src="/assets/img/quaoraroot.png">
</figure>
Böylece root yetkileriyle birlikte 3. flagi de elde etmiş oluyoruz. 
<figure >
    <img src="/assets/img/happy.gif">
</figure>
