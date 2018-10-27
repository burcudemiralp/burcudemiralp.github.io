---
layout: post
title: "Typhoon Writeup"
date: 2018-10-27
excerpt: "27 Oct 2018"
comments: false
---
---
layout: post
title: "Typhoon Writeup"
date: 2018-10-27
excerpt: "27 Oct 2018"
comments: false
---
Typhoon, PRISMA CSI tarafından hazırlanan zafiyetli bir linux makinadır.Ayrıntılı bilgiye [bu](https://www.prismacsi.com/typhoon-vulnerable-vm/) linkten ulaşabilirsiniz.
Netdiscover aracı ile makinaya ait IP adresini 192.168.1.55 olarak buluyoruz.
<figure >
    <img src="/assets/img/tayfun/1.png">
</figure>
Ardından basit bir nmap taraması ile açık portları ve üzerinde çalışan servisleri öğreniyoruz.
<figure >
    <img src="/assets/img/tayfun/2.png">
</figure>
Üzerinde çalışan çok sayıda servis mevcut. 8080 portundan başlamayı tercih ediyorum.

http://192.168.1.55:8080/ adresini ziyaret ettiğimizde tomcat default sayfası bizi karşılıyor.Buradan servisin konfigüre edilmediğine dair bir çıkarımda bulunmak mümkün. Bu sebepten gitmeyi denediğimiz ilk dizin "manager" oluyor. 
<figure >
    <img src="/assets/img/tayfun/3.png">
</figure>
Tomcat uygulaması için default username:password ikililerinden ilk olarak tomcat:tomcat deniyoruz ve bingo! İçeride war dosyası yükleyebileceğimiz bir "file upload" kısmı var.
İlk olarak bize bind shell verecek olan bir war dosyası oluşturuyoruz.
<figure >
    <img src="/assets/img/tayfun/4.png">
</figure>
War dosyasını upload ettikten sonra, 192.168.1.55:8080/bindshell/bindshell.jsp adresine giderek dosyayı tetikliyoruz.
Böylece tomcat7 kullanıcısı ile sisteme erişim sağlıyoruz.
<figure >
    <img src="/assets/img/tayfun/5.png">
</figure>
