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

Uygulamanın kullanıcıdan input alıyor olması ve ve statik analizde karşılaşılan strcpy gibi fonksiyonlar ilk olarak buffer overflow zafiyetini hatırlatıyor. Daha ayrıntılı incelemeye Windows 7 işletim sistemi üzerinde devam ediyoruz. İncelemeye başlamadan önce buffer overflow zafiyetinin ne olduğu, nasıl exploit edildiğine değinelim.

<figure >
    <img src="/assets/img/bof1.png">
</figure>

Uygulamayı Immunity Debugger içerisinde açıp, çalıştırıyoruz. 

<figure >
    <img src="/assets/img/brainpan6.PNG">
</figure>






{% highlight python %}

{% endhighlight %}

