---
layout: post
title: "LLMNR/NBT-NS Poisoining and SMB Relay Attack"
date: 2018-08-17
excerpt: "17 August 2018"
comments: false
---

LLMNR(Link Local Multicast Name Resolution) ve NBT-NS(NetBios Name Service), Windows işletim sistemlerinde isim çözümlenmesini ve iletişimi sağlayan iki bileşendir. DNS server üzerinde, sorguların başarısız olması durumunda, isim çözümlemeye LLMNR ve NBT-NS devam eder.

````
LLMNR DNS'e alternatif bir protokol değildir.DNS sorgularının başarısız olduğu durumlara karşın geliştirimiş bir çözümdür.

NetBIOS ise local network üzerinde, sistemlerin birbirleri ile iletişime geçmek için kullandıkları bir API'dir, protokol değildir.
````


