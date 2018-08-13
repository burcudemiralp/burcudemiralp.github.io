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
Atağı gerçekleştirebilmek için, ilgili kullanıcı için domain üzerinde "Replicating Changes Permission" ayarlarının aktif edilmesi gerekmektedir.Varsayılan olarak Domain Admins, Enterprise Admins, Administrators grupları ve Domain Controller hesapları için bu izinler aktiftir. Bu izinlere sahip olan herhangi bir domain kullanıcısı da bu atağı gerçekleştirebilir.

<figure >
    <img src="/assets/img/r.jpg">
</figure>

DCSync çalıştırıldığında öncelikle belirtilen domaine ait Domain Controller'ı bulur, ardından kullanıcılara ait bilgileri kopyalamak için (DRS protokolü aracılığıyla) Domain Controller' a istekte bulunur.

DCSync mimikatz ve empire ile kullanmak mümkündür.

<figure >
    <img src="/assets/img/desk1.jpg">
</figure>

<figure >
    <img src="/assets/img/desk2">
</figure>

Meterpreter shell üzerinde , DCSync çalıştırabilmek için kiwi modülü yüklenebilir.


<figure >
    <img src="/assets/img/desk3">
</figure>

<figure >
    <img src="/assets/img/desk4">
</figure>

