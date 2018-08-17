---
layout: post
title: "LLMNR/NBT-NS Poisoining and SMB Relay Attack"
date: 2018-08-17
excerpt: "17 August 2018"
comments: false
---

Link Local Multicast Name Resolution(LLMNR) ve NetBios Name Service(NBT-NS), Windows işletim sistemlerinde isim çözümlenmesini ve iletişimi sağlayan iki bileşendir. DNS server üzerinde, sorguların başarısız olması durumunda, isim çözümlemeye LLMNR ve NBT-NS devam eder.

Local network üzerinde bulunmayan testlab.com bilgisayarına ping atılmak istenirse, "testlab.com" adını çözümleyebilmek adına bu şekilde bir hiyerarşi takip edilecektir.
..*Öncelikle host dosyası(C:\Windows\System32\drivers\etc\hosts) kontrol edilir.
..*Local DNS cache kontrolü yapılır. (ipconfig /displaydns ile öğrebilebilir)
..*Ardından local network üzerinden bulunan DNS server'a DNS sorgusu gönderilir.
..*En son olarakta LLMNR ve NTB-NS sorguları gönderilir.

````
LLMNR DNS'e alternatif bir protokol değildir.DNS sorgularının başarısız olduğu durumlara karşın geliştirimiş bir çözümdür.

NetBIOS ise local network üzerinde, sistemlerin birbirleri ile iletişime geçmek için kullandıkları bir API'dir, protokol değildir.
````
