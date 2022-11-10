# TP 4: Réseau

[https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/4](https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/4)

**TP fait sur MACOS**

---
## I) First step
---

* A & B)Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP  

Commandes
* lsof
```powershell
sudo lsof -Pi -a -i4 -i6
```
* netstat
```powershell
netstat -n -p TCP
```
```powershell
netstat -n -p UDP
```


**Youtube**

[Youtube](./Youtube_Wireshark.pcapng)

* Port client : 55402
* IP & Port serveur : 77.136.192.83:443
* TCP

Résultat netstat
```powershell
tcp4       0      0  10.33.17.6.55402       77.136.192.83.443      ESTABLISHED
```


**Discord**

[Discord](./CallDiscord_Wireshark.pcapng)

* Port client : 62320
* IP & Port serveur : 35.214.249.235:50004
* UDP

Résultat lsof
```powershell
Discord   2407          yohan   64u  IPv4 0x674b385ca723b305      0t0    UDP *:62320
```


**Application Mail** 

[Mail](./Mail_Wireshark.pcapng)
* Port client : 49846
* IP & Port serveur : 64.233.167.109:993
* TCP

Résultat netstat
```powershell
tcp4       0      0  10.33.17.6.49846       64.233.167.109.993     ESTABLISHED
```

**Page Web**

[PageWeb](./Web_Wireshark.pcapng)
* Port client : 55506
* IP & Port serveur : 23.223.220.23:443 
* TCP

Résultat nestat
```powershell
tcp4       0      0  10.33.17.6.55506       23.223.220.23.443      ESTABLISHED
```

**Plan**

[Plan](./Plan_Wireshark.pcapng)
* Port client : 55559
* IP & Port serveur : 17.248.180.102:443 
* TCP

Résultat netstat
```powershell
tcp4       0      0  10.33.17.6.55559       17.248.180.102.443     ESTABLISHED
```

---
## II) Mise en place
---

### 1) SSH

* A) Examinez le trafic dans Wireshark

[SSH_Wireshark](./SSH_Wireshark.pcapng)

* B) Demandez aux OS 

**MACOS**
```powershell
MacBook-Air-Yohan:~ yohan$ sudo lsof -Pi -a -i4 -i6
ssh       9499          yohan    4u  IPv4 0x674b386175dc1fc5      0t0    TCP 172.16.72.1:52476->node1:22 (ESTABLISHED)
```

**LINUX**
```powershell
[user1@node1 ~]$ ss
tcp   ESTAB 0      0                     172.16.72.11:ssh    172.16.72.1:52476
```

### 2) Routage

III) DNS

### 2) Setup

* A) Dans le rendu je veux 

**fichier conf principal**

```powershell
[user1@dns-server ~]$ sudo cat /etc/named.conf
[sudo] password for user1: 
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
	listen-on port 53 { 127.0.0.1; any; };
	listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-validation yes;

	managed-keys-directory "/var/named/dynamic";
	geoip-directory "/usr/share/GeoIP";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";

	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "tp4.b1" IN {
     type master;
     file "tp4.b1.db";
     allow-update { none; };
     allow-query {any; };
};

zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp4.b1.rev";
     allow-update { none; };
     allow-query { any; };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

**Fichier de zone de nom --> IP**

```powershell
[user1@dns-server ~]$ sudo cat /var/named/tp4.b1.db
$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns-server IN A 10.4.1.201
node1      IN A 10.4.1.11
```

**Fichier de zone inverse IP --> nom**

```powershell
[user1@dns-server ~]$ sudo cat /var/named/tp4.b1.rev
$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

;Reverse lookup for Name Server
201 IN PTR dns-server.tp4.b1.
11 IN PTR node1.tp4.b1.
```

**Démarrage du service DNS**

```powershell
[user1@dns-server ~]$ sudo systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
     Active: active (running) since Thu 2022-11-10 10:49:46 CET; 18s ago
   Main PID: 9891 (named)
      Tasks: 4 (limit: 5877)
     Memory: 16.1M
        CPU: 43ms
     CGroup: /system.slice/named.service
             └─9891 /usr/sbin/named -u named -c /etc/named.conf

Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: network unreachable resolving './NS/IN': 2001:503:c27::2:30#53
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa/IN: loaded serial 0
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: zone 1.0.0.127.in-addr.arpa/IN: loaded serial 0
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: zone localhost.localdomain/IN: loaded serial 0
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: zone localhost/IN: loaded serial 0
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: all zones loaded
Nov 10 10:49:46 dns-server.tp4.b1 systemd[1]: Started Berkeley Internet Name Domain (DNS).
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: running
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: managed-keys-zone: Initializing automatic trust anchor management for zone '.'; DNSKEY ID 20326 is now trusted, waiving the normal 30-day waiting period.
Nov 10 10:49:46 dns-server.tp4.b1 named[9891]: resolver priming query complete
```

**Commande ss**
```powershell
[user1@dns-server ~]$ sudo ss -ltupn
udp               UNCONN             0                  0                              172.16.72.201:53                                  0.0.0.0:*                 users:(("named",pid=9891,fd=19))
tcp               LISTEN             0                  10                             172.16.72.201:53                                  0.0.0.0:*                 users:(("named",pid=9891,fd=21))
```

* B) Ouvrez le bon port dans le firewall

```powershell
[user1@dns-server ~]$ sudo firewall-cmd --add-port=53/tcp --permanent
[sudo] password for user1: 
success
[user1@dns-server ~]$ sudo firewall-cmd --add-port=53/udp --permanent
success
[user1@dns-server ~]$ sudo firewall-cmd --reload
```

### 3) Test

* A) Sur la machine node1.tp4.b1


```powershell
[user1@node1 ~]cat /etc/resolv.conf 
# Generated by NetworkManager
search tp4.b1
#nameserver 1.1.1.1
nameserver 172.16.72.201
```

**node1**
```powershell
[user1@node1 ~]$ dig node1.tp4.b1

; <<>> DiG 9.16.23-RH <<>> node1.tp4.b1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32743
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8901faff4b956af101000000636cd3e8d4a6c49205f80d62 (good)
;; QUESTION SECTION:
;node1.tp4.b1.			IN	A

;; ANSWER SECTION:
node1.tp4.b1.		86400	IN	A	10.4.1.11

;; Query time: 0 msec
;; SERVER: 172.16.72.201#53(172.16.72.201)
;; WHEN: Thu Nov 10 12:29:21 CET 2022
;; MSG SIZE  rcvd: 85
```

**dns-server**
```powershell
[user1@node1 ~]$ dig dns-server.tp4.b1

; <<>> DiG 9.16.23-RH <<>> dns-server.tp4.b1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3152
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7c429c19408dcf2101000000636cd4255c0c7bed0a82748d (good)
;; QUESTION SECTION:
;dns-server.tp4.b1.		IN	A

;; ANSWER SECTION:
dns-server.tp4.b1.	86400	IN	A	10.4.1.201

;; Query time: 0 msec
;; SERVER: 172.16.72.201#53(172.16.72.201)
;; WHEN: Thu Nov 10 12:30:21 CET 2022
;; MSG SIZE  rcvd: 90
```

**google.com**

```powershell
[user1@node1 ~]$ dig google.com @172.16.72.201

; <<>> DiG 9.16.23-RH <<>> google.com @172.16.72.201
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20724
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 4cffb2e8086ac51001000000636cd2e5dab24654a2d22332 (good)
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		300	IN	A	216.58.214.78

;; Query time: 810 msec
;; SERVER: 172.16.72.201#53(172.16.72.201)
;; WHEN: Thu Nov 10 12:25:02 CET 2022
;; MSG SIZE  rcvd: 83
```

* B) Sur votre PC 

```powershell
MacBook-Air-Yohan:~ yohan$  dig node1.tp4.b1 @172.16.72.201

; <<>> DiG 9.10.6 <<>> node1.tp4.b1 @172.16.72.201
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18428
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;node1.tp4.b1.			IN	A

;; ANSWER SECTION:
node1.tp4.b1.		86400	IN	A	10.4.1.11

;; Query time: 5 msec
;; SERVER: 172.16.72.201#53(172.16.72.201)
;; WHEN: Thu Nov 10 11:47:00 CET 2022
;; MSG SIZE  rcvd: 57
```

[DNS_Wireshark](./DNS_Wireshark.pcapng)


