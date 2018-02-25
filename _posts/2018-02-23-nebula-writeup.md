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
Dosyaları incelediğimizde ELF türünde olduklarını görüyoruz. Dosyalardan birini çalıştırdığımızda,
{% raw %}
    level00@nebula:~$ /bin/.../flag00
    Congrats, now run getflag to get your flag !
    flag00@nebula:~$
{% endraw %}
flag değerini elde edebilmek için `getflag` dosyasını çalıştırmamız gerektiği bilgisine erişiyoruz.
{% raw %}
    flag00@nebula:~$ find / -name getflag
    /bin/getflag
    /rofs/bin/getflag
    flag00@nebula:~$ /bin/getflag
    You have successfully executed getflag on a target account
{% endraw %}

### Level 01

Bu bölümde bizden beklenen /home/flag01 dizini altında bulunan c programında, istenilen programın çalıştırılmasına sebebiyet veren zafiyeti bulmamız.Programa ait kaynak kod da aşağıdaki gibi.

{% highlight c %}
{% raw %}
    #include <stdlib.h>
    #include <unistd.h>
    #include <string.h>
    #include <sys/types.h>
    #include <stdio.h>

    int main(int argc, char **argv, char **envp)
    {
     gid_t gid;
     uid_t uid;
     gid = getegid();
     uid = geteuid();

     setresgid(gid, gid, gid);
     setresuid(uid, uid, uid);

     system("/usr/bin/env echo and now what?");
    }
{% endraw %}
{% endhighlight %}
Kaynak kodu daha iyi anlayabilmek adına, programda bulunan bazı fonksiyonlara bakalım :+1:

> **gid_t getegid(void);** /* çağırılan processin effective group-id değerini döndürüyor. */

> **uid_t geteuid(void);** /* çağırılan processin effective user-id değerini döndürüyor. */

> **int setresgid(gid_t rgid,gid_t egid,git_t sgid);** /* çağırılan processin real,effective,saved  group-id değerlerini set ediyor. */

> **int setresuid(uid_t ruid,uid_t euid,uid_t suid);** /* çağırılan processin real,effective,saved user-id değerlerini set ediyor. */


