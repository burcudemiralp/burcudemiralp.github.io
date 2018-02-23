---
layout: post
title: "Nebula Write Up"
date: 2018-02-23
comments: false
---
  Nebula, Linux Exploitation üzerine pratik yapma imkanı sunan zafiyetli bir sanal makinadır. Daha ayrıntılı bilgiye
[ buradan ](https://exploit-exercises.com/nebula/) ulaşabilirsiniz.

### Level 00

Bu level için bizden beklenen flag00 kullanıcısına ait, suid bitine sahip olan, çalıştırılabilir bir dosya bulmamız. 

{% raw %}
    level00@nebula:~$ find / -type f -user flag00 -perm -u=s 2>/dev/null
    /bin/.../flag00
    /rofs/bin.../flag00
{% endraw %}
Dosyayı incelediğimizde ELF türünde bir dosya olduğunu görüyoruz. Çalıştırdığımızda,
{% raw %}
    level00@nebula:~$ /bin/.../flag00
    Congrats, now run getflag to get your flag !
    flag00@nebula:~$
{% endraw %}
flag değerini elde edebilmek için `getflag` dosyasını çalıştırmamız gerektiğini görüyoruz. Dosyanın yerini bulabilmek için,
