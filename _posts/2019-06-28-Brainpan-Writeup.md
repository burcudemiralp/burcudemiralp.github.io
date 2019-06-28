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

Daha sonra incelemek üzere exe dosyasını indiriyoruz. 

9999 portunda çalışan servis hakkında fikir sahibi olabilmek adına 9999 portu ile bağlantı kurmaya çalışıyoruz.

<figure >
    <img src="/assets/img/brainpan4.png">
</figure>

