---
layout: post
title: "Active Directory Attack-DCSync"
date: 2018-08-10
excerpt: "07 August 2018"
comments: false
---
DCSync internal testlerde, privilege escalation aşamasından sonra post exploitation amacıyla kullanılan bir atak türüdür. DCSync ve gerekli haklar ile birlikte, domain üzerinde ki tüm kullanıcılara ait parola hashleri,  Domain Controller üzerinde herhangi bir kod çalıştırmadan elde edilebilmektedir.Yani parola hashlerini elde etmek için kullanılan diğer yöntemlere göre daha sessiz çalıştığı söylenebilir.

Atak temelde, Domain Controller davranışının taklit edilip ,DRS(Directory Replication Service) protokolü aracılığıyla hedef Domain Controller'a parola hashlerini alabilmek için istek göndermesine dayanır.


NTDS.dit dosyası işletim sistemi tarafından kullanılan bir dosya olduğu için, doğrudan başka bir yere kopyalanması mümkün değildir. Bi

```` 
Domain üzerindeki tüm kullanıcıların parola hashleri NTDS.dit dosyasında saklanır.Bu dosya kullanıcılar, gruplar, grup üyelikleri gibi Active Directory bilgilerini depolayan bir veritabanıdır.Yalnızca Domain Controller üzerinde bulunur ve dosya yolu C:\Windows\NTDS\NTDS.dit şeklindedir.
````
