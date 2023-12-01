---
title: "stealth"
date: 2023-11-29T11:30:03+00:00
draft: false
tags: ["tryhackme"]
description: "Stealth Tryhackme Room"
---

![](https://i.imgur.com/gd7lkVU.png)

## nmap scans

```bash
PORT      STATE SERVICE       REASON  VERSION
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services
|_ssl-date: 2023-11-30T18:28:49+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: HOSTEVASION
|   NetBIOS_Domain_Name: HOSTEVASION
|   NetBIOS_Computer_Name: HOSTEVASION
|   DNS_Domain_Name: HostEvasion
|   DNS_Computer_Name: HostEvasion
|   Product_Version: 10.0.17763
|_  System_Time: 2023-11-30T18:28:10+00:00
| ssl-cert: Subject: commonName=HostEvasion
| Issuer: commonName=HostEvasion
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-07-28T19:06:15
| Not valid after:  2024-01-27T19:06:15
| MD5:   110c:1c21:e230:b7c7:41f5:4b6a:bf2b:9e6a
| SHA-1: 34ad:3702:1a0a:2054:88a9:ea0c:820b:da64:b1bd:fb56
| -----BEGIN CERTIFICATE-----
| MIIC2jCCAcKgAwIBAgIQMIOcafxeh79B5cu+rs/taDANBgkqhkiG9w0BAQsFADAW
| MRQwEgYDVQQDEwtIb3N0RXZhc2lvbjAeFw0yMzA3MjgxOTA2MTVaFw0yNDAxMjcx
| OTA2MTVaMBYxFDASBgNVBAMTC0hvc3RFdmFzaW9uMIIBIjANBgkqhkiG9w0BAQEF
| AAOCAQ8AMIIBCgKCAQEA2tUyXSZT7x2YueFMia0tU6xweBIvbwEXw0MBCXtHEf9A
| LqZ6aiwNSsiLeW/kfBsqw6LArZNajuGggR2uj2HLGMn9Yx2RjnMSUaVWlJnB+j7s
| YsgeVOr3Y8rFv0EPD2M6tKEZ7Zh8HoaBifHR3qeNIx+n6YcYmSjb0mUQ5kQso7SS
| L7a9Beya4aynWgHXegaCVP0CcA750BRf1Ax+tjpojoTJOarC0C1QibbDs0s6NbUY
| Z1CakxCRQlENDRau+vqqhRMxlbEfayl1YICTfMe6j3hMnVeYiPjZECt2nSe92i2p
| rnzpdZ4Xbe8tdDzGETQGkBdOCOKPk6/nh80ifpcjBQIDAQABoyQwIjATBgNVHSUE
| DDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcNAQELBQADggEBABB4
| HKrRnIrik9ef1F3Ah6r4FsdpCmZ0vXLNixsqm8IY81fNcRTogc/WFytU9gylcxRk
| LhoUqXwtQhKqMFOKcEh3Kq2+VMUvgxTxvDywFS4S02AlhWtafq8NBm5nfxxubuit
| tRO3fvdQ7mKS2hWvapW9+guEt0zJZI3Ai/C4NIq5WpbLEGSJe6DHUwXaPyFiHNYy
| 5j91hKUWbDnIy4TqiIPjhBjYhrTvi46fbGbqMpHelUGABzJ5LFfGjORMOWA1bRPz
| wuaEP62Dimr42pzbLPIgGTmBwpIXlpKdcydbJnVORxY4AfpLV6ypt2EPYS2TpKbz
| 4Fw5A8aWrShuerOI7mc=
|_-----END CERTIFICATE-----
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8000/tcp  open  http          syn-ack PHP cli server 5.5 or later
|_http-title: 404 Not Found
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
8080/tcp  open  http          syn-ack Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
|_http-title: PowerShell Script Analyser
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
8443/tcp  open  ssl/http      syn-ack Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:   a0a4:4cc9:9e84:b26f:9e63:9f9e:d229:dee0
| SHA-1: b023:8c54:7a90:5bfa:119c:4e8b:acca:eacf:3649:1ff6
| -----BEGIN CERTIFICATE-----
| MIIBnzCCAQgCCQC1x1LJh4G1AzANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDEwls
| b2NhbGhvc3QwHhcNMDkxMTEwMjM0ODQ3WhcNMTkxMTA4MjM0ODQ3WjAUMRIwEAYD
| VQQDEwlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMEl0yfj
| 7K0Ng2pt51+adRAj4pCdoGOVjx1BmljVnGOMW3OGkHnMw9ajibh1vB6UfHxu463o
| J1wLxgxq+Q8y/rPEehAjBCspKNSq+bMvZhD4p8HNYMRrKFfjZzv3ns1IItw46kgT
| gDpAl1cMRzVGPXFimu5TnWMOZ3ooyaQ0/xntAgMBAAEwDQYJKoZIhvcNAQEFBQAD
| gYEAavHzSWz5umhfb/MnBMa5DL2VNzS+9whmmpsDGEG+uR0kM1W2GQIdVHHJTyFd
| aHXzgVJBQcWTwhp84nvHSiQTDBSaT6cQNQpvag/TaED/SEQpm0VqDFwpfFYuufBL
| vVNbLkKxbK2XwUvu0RxoLdBMC/89HqrZ0ppiONuQ+X2MtxE=
|_-----END CERTIFICATE-----
|_http-title: PowerShell Script Analyser
| tls-alpn: 
|_  http/1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
47001/tcp open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         syn-ack Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
49670/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 0s, deviation: 0s, median: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 52662/tcp): CLEAN (Timeout)
|   Check 2 (port 33712/tcp): CLEAN (Timeout)
|   Check 3 (port 30307/udp): CLEAN (Timeout)
|   Check 4 (port 41933/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2023-11-30T18:28:10
|_  start_date: N/A


```
## checking the site

![](https://i.imgur.com/uV5DpSl.png)

Checking the site out as described on tryhackme we are presented a site.

![](https://i.imgur.com/JWMZg47.png)

So on the site we are only allowed to upload powershell scripts, we need to upload a script which will give us a reverse sehll to connect to the machine.

## initial foothold
Generating a reverse shell with [revshells](http://revshells.com) and uploading it on the website we get a connection back. 

*Disclaimer*
![](https://i.imgur.com/FQKIuhp.png)
After uploading the script and using netcat as your listener you'll get a shell connection back but without the terminal instant which is *'funny'* to work with. 

A better shell instance that i got was from this guy [malw0re](https://github.com/malw0re/scriptures-). So after uploading we get a clean looking shell back. Searching the folders we land on the Desktop folder and there is a file called *encodedflag*.

Decoding and reading the file we get some interesting message.

![](https://i.imgur.com/kNofE1s.png)

Following the decoded message and checking it out on the webpage we get an interesting message.
![](https://i.imgur.com/JqHJOnC.png)

## user level flag.

Checking the documents file there is a directory called tasks inside it there's a file called log.txt and from the webpage response we need to remove the log file not to alert the blue team, removing the log file and reloading the webpage we get nothing so got stuck for a while.
Then reading the *file.ps1* we see there is a directory hosting xampp

![](https://i.imgur.com/ZaomI08.png)

Under the uploads directory, we identify the a log file removing it and refreshing the page we get the first flag.

![](https://i.imgur.com/6C9PKha.png)

![](https://i.imgur.com/o4vn183.png)

## root level flag.

Having the user flag we need to look out for privesc vectors on the box and since it's a windows box i uploaded a powershell script that looks for some vectors.

![](https://i.imgur.com/q5dzts3.png)

The command is downloading the (win-priv-check.ps1) from my attacker ip address using using the Invoke-WebRequest cmdlet and saving it locally as win.ps1. Running the script it identifies apache is running as admin.

![](https://i.imgur.com/OifDB01.png)

Let's check evader's privileges and check if we can abuse any privileges.

![](https://i.imgur.com/a72wbxc.png)

so we don't have any specific privilege that we can abuse so my next step was to upload a malicious php script to the webserver using pownyshell.

checking the privileges we see it's different from the output we go previously.

![](https://i.imgur.com/A9Yszsw.png)

As we can see we can abuse the *"SetImpersonatePrivilege"* any process holding this privilege can impersonate (but not create) any token for which it is able to gethandle. You can get a privileged token from a Windows service (DCOM) making it perform an NTLM authentication against the exploit, then execute a process as SYSTEM [referenced from here.](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens)

So our privesc vector are all relate to a potato exploit and after trying and failing i landed on the *"efspotato"* exploit which worked and elevate our privileges to NT SYSTEM!!

![](https://i.imgur.com/F5WGaj8.png)

Trying to get the admin flag we are face with some errors.

![](https://i.imgur.com/wDnF2Xy.png)

So the next step was creating a user with admin privileges then rdp into the machine and get the machine.

![](https://i.imgur.com/U29uu8P.png)

Login into the machine with the created user and get the flag

![](https://i.imgur.com/jlALEK2.png)

Hope you learnt something new ✨