---
layout: post
title: "LLMNR/NBT-NS Poisoining and SMB Relay Attack"
date: 2018-08-17
excerpt: "17 August 2018"
comments: false
---
#### LLMNR/NBT-NS Nedir?
Link Local Multicast Name Resolution(LLMNR) ve NetBIOS Name Service(NBT-NS), Windows işletim sistemlerinde isim çözümlenmesini ve iletişimi sağlayan iki bileşendir. DNS server üzerinde, sorguların başarısız olması durumunda, isim çözümlemeye LLMNR ve NBT-NS devam eder.

Local network üzerinde bulunmayan klmn.local bilgisayarına ping atılmak istenirse, "klmn.local" adını çözümleyebilmek adına bu şekilde bir hiyerarşi takip edilecektir.

+ Öncelikle host dosyası kontrol edilir.
+ Local DNS cache kontrolü yapılır. 
+ Ardından local network üzerinden bulunan DNS server'a DNS sorgusu gönderilir.
+ En son olarakta LLMNR ve NTB-NS sorguları gönderilir.

>> LLMNR DNS'e alternatif bir protokol değildir.DNS sorgularının başarısız olduğu durumlara karşın geliştirimiş bir çözümdür.

>> NetBIOS ise local network üzerinde, sistemlerin birbirleri ile iletişime geçmek için kullandıkları bir API olup, protokol değildir.

#### LLMNR/NBT-BS Nasıl Çalışır ?

Arp protokolünün işleyişine benzer bir mantıkla çalışırlar. Çözümlenmeye çalışılan isim için LLMNR protokolü multicast(224.0.0.252), NBT-NS ise broadcast yayın yaparak ağdaki cihazlara istek paketi gönderirler. 
<figure >
    <img src="/assets/img/llmnr2.PNG">
</figure>

+ Paket 4-6: Yapılan DNS sorgusu başarısız oluyor.
+ Paket 8-15: klmn.local ismini çözümleyebilmek için LLMNR protokolü aracılığıyla ağda multicast yayın yapılıyor.
+ Paket 18: klmn.local ismini çözümleyebilmek için NBT-NS  aracılığıyla ağda broadcast yayın yapılıyor.
