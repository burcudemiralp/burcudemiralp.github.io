---
layout: post
title: "LLMNR/NBT-NS Poisoning"
date: 2018-08-17
excerpt: "17 August 2018"
comments: false
---
#### LLMNR/NBT-NS Nedir?
Link Local Multicast Name Resolution(LLMNR) ve NetBIOS Name Service(NBT-NS), Windows işletim sistemlerinde isim çözümlenmesini ve iletişimi sağlayan iki bileşendir. DNS server üzerinde, sorguların başarısız olması durumunda, isim çözümlemeye LLMNR ve NBT-NS devam eder.

>> LLMNR DNS'e alternatif bir protokol değildir.DNS sorgularının başarısız olduğu durumlara karşın geliştirimiş bir çözümdür.

>> NetBIOS ise local network üzerinde, sistemlerin birbirleri ile iletişime geçmek için kullandıkları bir API olup, protokol değildir.


#### LLMNR/NBT-BS Nasıl Çalışır ?

Arp protokolünün işleyişine benzer bir mantıkla çalışırlar. Çözümlenmeye çalışılan isim için LLMNR protokolü multicast(224.0.0.252), NBT-NS ise broadcast yayın yaparak ağdaki cihazlara istek paketi gönderir. Çözümlenmek istenen ada sahip olan bilgisayar isteğe yanıt verir.

Local network üzerinde bulunmayan klmn.local bilgisayarına ping atılmak istenirse, "klmn.local" adını çözümleyebilmek adına bu şekilde bir hiyerarşi takip edilecektir.

+ Öncelikle host dosyası kontrol edilir.
+ Local DNS cache kontrolü yapılır. 
+ Ardından local network üzerinden bulunan DNS server'a DNS sorgusu gönderilir.
+ En son olarakta LLMNR ve NTB-NS sorguları gönderilir.

<figure >
    <img src="/assets/img/llmnr2.PNG">
</figure>

+ Paket 4-6: klmn.local ismi için DNS sorgusu yapılıyor ve başarısızlıkla sonuçlanıyor.
+ Paket 8-15: klmn.local ismini çözümleyebilmek için LLMNR protokolü aracılığıyla ağda multicast yayın yapılıyor.
+ Paket 18: klmn.local ismini çözümleyebilmek için NBT-NS  aracılığıyla ağda broadcast yayın yapılıyor.

#### LLMNR/NBT-BS Poisoning

Ağa katılmış olan bir saldırgan gelen LLMNR ve NBT-NS isteklerini dinleyip, sahte cevaplar üreterek hedefe gönderir. Arp poisoning olayında olduğu gibi, cevapları doğrulayan bir mekanizma yoktur. Bu sebepten ötürü kurban aldığı cevapları doğru kabul eder. Bundan sonrası için trafik saldırgan üzerinden devam eder.

<figure >
    <img src="/assets/img/llmnr3.png">
</figure>

#### LLMNR/NBT-BS Poisoning ile NTLMv2 Hash Elde Etme

Hatalı DNS sorgularının, dosya paylaşımı,yazıcı paylaşımı gibi SMB protokolünün kullanıldığı durumlarda meydana gelmesi sonucu gerçekleştirilebilen bir ataktır. 
```
SMB (Server Message Block ) dosyaları, yazıcıları ve serial portları paylaşmak için kullanılan bir protokoldür.
SMB server'lar ağ üzerindeki dosya sistemini ve diğer kaynakları istemciler için hazır hale getirirler.

SMB sunucuları yetkilendirme için NTLMv2 Challenge/Response Authentication yöntemini kullanırlar. Süreç şu şekildedir:
    + İstemci sunucuya bir login isteği gönderir.(Type 1 Message)
    + Sunucu bir takım extra bilgilerle birlikte challenge denilen random bir string gönderir.(Type 2 Message)
    + İstemci kendi parolasına ait hash ile birlikte challenge'ı şifreler ve response olarak sunucuya gönderir.(Type 3 Message)
    + Sunucu gelen response'u decrypt eder.Çıktı, gönderdiği challenge ile eşleşiyorsa istemciyi yetkilendirir.

````
Saldırgan broadcast paketlerine sahte cevaplar üretmekle birlikte, SMB server gibi davranır ve kurban oturum açabilmek için login isteği gönderir. Saldırgan bir challenge göndererek, kurbanın kendi parola hashi ile şifrelemesini ister. Response ulaştıktan sonra, kurbana hata mesajı döner.

<figure >
    <img src="/assets/img/aa.png">
</figure>

Atağı gerçekleştirmek için [Responder](https://github.com/SpiderLabs/Responder) aracı kullanılabilir.

````
Responder LLMNR,NBT-NS  broadcast paketlerini yakalayabilen bir python scriptidir.
Bununla birlikte sahte authentication sunucuları oluşturarak LLMNR ve NBT-NS poisoning gerçekleştirir
````
Responder ile bir dinleme başlatılıyor. -I parametresi ile dinlemenin yapılacağı network arayüzü belirtiliyor.
<figure >
    <img src="/assets/img/3.png">
</figure>

Kullanıcı domain üzerinde mevcut olmayan bir klasöre erişmeyi deniyor.

<figure >
    <img src="/assets/img/pp1.png">
</figure>

Böylece isteği yapan kullanıcıya ait NTLMv2 hashi elde ediliyor.
<figure >
    <img src="/assets/img/pp2.png">
</figure>

Atak sırasında oluşan trafik şu şekilde:

<figure >
    <img src="/assets/img/paket.png">
</figure>

+ Paket 1-7: testlab için isim sorgusu ve LLMNR poisoning
+ Paket 8-11: TCP bağlantısının sağlanması ve SMB bağlantı isteği
+ Paket 12-17: Saldırganın hashi yakalamasıyla sonuçlanan NTLM challenge/response authentication işlemi(NTLM mesajları SMB paketleri içine gömülü)

>> Elde edilen hash NTLMv2 olduğu için doğrudan "Pass The Hash" ataklarında kullanılamaz. Elde edilen hashi clear-text parolaya çevirmek için Hashcat, John the Ripper gibi araçlar kullanılabilir.
