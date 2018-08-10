---
layout: post
title: "Active Directory Attack-DCSync"
date: 2018-08-10
excerpt: "07 August 2018"
comments: false
---
DCSync internal testlerde, privilege escalation aşamasından sonra post exploitation amacıyla kullanılan bir atak türüdür. DCSync ve gerekli haklar ile birlikte, domain üzerinde ki tüm kullanıcılara ait parola hashleri,  Domain Controller üzerinde herhangi bir kod çalıştırmadan elde edilebilmektedir.Yani parola hashlerini elde etmek için kullanılan diğer yöntemlere göre daha sessiz çalıştığı söylenebilir.

Atak temelde, Domain Controller davranışının taklit edilip, DRS(Directory Replication Service) protokolü aracılığıyla hedef Domain Controller'a parola hashlerini alabilmek için istek göndermesine dayanır.

```` 
Domain üzerindeki tüm kullanıcıların parola hashleri NTDS.dit dosyasında saklanır.

Dosya kullanıcılar, gruplar, grup üyelikleri gibi Active Directory bilgilerini depolayan bir veritabanıdır.

İşletim sistemi tarafından  kullanılan bir dosya olduğu için, doğrudan başka bir yere kopyalanması mümkün değildir.

Yalnızca Domain Controller üzerinde bulunur ve dosya yolu C:\Windows\NTDS\NTDS.dit şeklindedir.
````

````
DSR, Domain Controller'lar arasında bilgi kopyalanmasını ve Active Directory yönetimini sağlar.
````
Atağı gerçekleştirebilmek için, ilgili kullanıcı için domain üzerinde "Replicating Changes Permission" ayarlarının aktif edilmesi gerekmektedir.Varsayılan olarak Domain Admins, Enterprise Admins, Administrators grupları ve Domain Controller hesapları için bu izinler aktiftir.

<figure >
    <img src="/assets/img/r.jpg">
</figure>
