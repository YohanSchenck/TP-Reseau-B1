# TP 3 : Réseau

[https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/3](https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/3)

---
## I) ARP  
---
### 1) Echange ARP

* A) Générer des requêtes ARP  

Ping de John vers Marcel
```powershell
[user1@localhost ~]$ ping 172.16.72.12
PING 172.16.72.12 (172.16.72.12) 56(84) bytes of data.
64 bytes from 172.16.72.12: icmp_seq=1 ttl=64 time=1.94 ms
64 bytes from 172.16.72.12: icmp_seq=2 ttl=64 time=1.83 ms
64 bytes from 172.16.72.12: icmp_seq=3 ttl=64 time=1.71 ms
````

Table ARP de chaque VM  

**John**
```powershell
[user1@localhost ~]$ ip neigh show
172.16.72.12 dev enp0s1 lladdr 16:02:57:d9:20:ba STALE
172.16.72.1 dev enp0s1 lladdr 1e:57:dc:e3:58:64 DELAY
````

L adresse mac de Marcel est bien présente : **16:02:57:d9:20:ba**  

**Marcel**
```powershell
[user1@localhost ~]$ ip neigh show
172.16.72.1 dev enp0s1 lladdr 1e:57:dc:e3:58:64 DELAY
172.16.72.11 dev enp0s1 lladdr 5a:c2:76:4c:ab:48 STALE
```
L adresse mac de John est bien présente : **5a:c2:76:4c:ab:48**  

Vérification de l'adresse MAC de Marcel sur Marcel

```powershell
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 16:02:57:d9:20:ba brd ff:ff:ff:ff:ff:ff
    inet 172.16.72.12/24 brd 172.16.72.255 scope global noprefixroute enp0s1
       valid_lft forever preferred_lft forever
    inet6 fe80::1402:57ff:fed9:20ba/64 scope link 
       valid_lft forever preferred_lft forever
```


### 2) Analyse de trames
* B) Analyse de tram

**Utilisation de la commande tcpdump**
```powershell
[user1@localhost home]$ sudo tcpdump -i enp0s1 -c 10 -w tp3_arp.pcap not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10 packets captured
11 packets received by filter
0 packets dropped by kernel
````


**Commande pour copier le fichier en local**
```powershell
MacBook-Air-Yohan:Downloads yohan$ scp user1@172.16.72.11:/home/tp3_arp.pcap ~/Downloads/
user1@172.16.72.11's password: 
tp3_arp.pcap                                  100% 1040     1.1MB/s   00:00 
```

[ARP-Wireshark](./tp3_arp.pcap)

Le fichier contient bien un ARP request et un ARP reply

----
## II) Routage
---
### 1) Mise en place du routage 
* A) Activer le routage sur le noeud router

Les 2 cartess réseaux sur la VM Router
```powershell 
[user1@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 32:ef:dd:c4:de:32 brd ff:ff:ff:ff:ff:ff
    inet 192.168.64.3/24 brd 192.168.64.255 scope global dynamic noprefixroute enp0s1
       valid_lft 86052sec preferred_lft 86052sec
    inet6 fd86:944a:eb3:ddae:e5d0:c05a:56c1:3321/64 scope global dynamic noprefixroute 
       valid_lft 2591954sec preferred_lft 604754sec
    inet6 fe80::6f18:b4cc:c11b:d602/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: enp0s2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether ce:01:81:bb:a5:ef brd ff:ff:ff:ff:ff:ff
    inet 172.16.72.254/24 brd 172.16.72.255 scope global noprefixroute enp0s2
       valid_lft forever preferred_lft forever
    inet6 fe80::cc01:81ff:febb:a5ef/64 scope link 
       valid_lft forever preferred_lft forever
````
Il y a une carte, **enp0s1**, "Shared Network" pour la 3ème partie du TP et une carte Host only, **enp0s2**. 

Activation du routage :

```powershell
[user1@localhost ~]$ sudo firewall-cmd --list-all
[sudo] password for user1: 
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s1 enp0s2
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[user1@localhost ~]$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s1 enp0s2
[user1@localhost ~]$ sudo firewall-cmd --add-masquerade --zone=public
success
[user1@localhost ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```

* B) Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping

**NA car MACOS**

### 2) Analyse de trames

**NA car MACOS**


### 3) Accès internet

**Ajout d'une route par défaut sur John**

* A) Donnez un accès internet à vos machines

```powershell
[user1@John ~]$ ip r s 
default via 172.16.72.254 dev enp0s1 
default via 172.16.72.254 dev enp0s1 proto static metric 100 
172.16.72.0/24 dev enp0s1 proto kernel scope link src 172.16.72.11 metric 100
```

**Accès à internet**

```powershell
[user1@John ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=27.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=25.9 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=35.9 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=50.5 ms
```

**Serveur DNS**

```powershell
[user1@John ~]$ dig gitlab.com

; <<>> DiG 9.16.23-RH <<>> gitlab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33732
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;gitlab.com.			IN	A

;; ANSWER SECTION:
gitlab.com.		164	IN	A	172.65.251.78

;; Query time: 19 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Tue Oct 25 11:43:40 CEST 2022
;; MSG SIZE  rcvd: 55
```

```powershell
[user1@John ~]$ ping gitlab.com
PING gitlab.com (172.65.251.78) 56(84) bytes of data.
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=1 ttl=53 time=23.3 ms
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=2 ttl=53 time=37.3 ms
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=3 ttl=53 time=34.5 ms
```

* B) Analyse de trames

[ping-Wireshark](./tp3_ping.pcap)

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |     |
|-------|------------|--------------------|-------------------------|----------------|-----------------|-----|
| 1     | ping       | `John` `172.16.72.11` | `John` `5a:c2:76:4c:ab:48` | `8.8.8.8`      | `ce:01:81:bb:a5:ef`              |     |
| 2     | pong       | `8.8.8.8`                | `ce:01:81:bb:a5:ef`                    | `172.16.72.11`           | `5a:c2:76:4c:ab:48`            |

---
## III) DHCP
---
### 1) Mise en place du serveur DHCP

* A) Sur la machine john, vous installerez et configurerez un serveur DHCP

**Installation du serveur sur John**

```powershell
[user1@John ~]$ sudo dnf install dhcp-server -y
[sudo] password for user1: 
Rocky Linux 9 - BaseOS                          6.8 kB/s | 3.6 kB     00:00    
Rocky Linux 9 - BaseOS                          1.3 MB/s | 1.3 MB     00:01    
Rocky Linux 9 - AppStream                       9.6 kB/s | 3.6 kB     00:00    
Rocky Linux 9 - AppStream                       2.2 MB/s | 4.9 MB     00:02    
Rocky Linux 9 - Extras                          6.2 kB/s | 2.9 kB     00:00    
Dependencies resolved.
================================================================================
 Package           Architecture  Version                     Repository    Size
================================================================================
Installing:
 dhcp-server       aarch64       12:4.4.2-15.b1.el9          baseos       1.2 M
Installing dependencies:
 dhcp-common       noarch        12:4.4.2-15.b1.el9          baseos       128 k

Transaction Summary
================================================================================
Install  2 Packages

Total download size: 1.3 M
Installed size: 4.0 M
Downloading Packages:
(1/2): dhcp-common-4.4.2-15.b1.el9.noarch.rpm   448 kB/s | 128 kB     00:00    
(2/2): dhcp-server-4.4.2-15.b1.el9.aarch64.rpm  2.6 MB/s | 1.2 MB     00:00    
--------------------------------------------------------------------------------
Total                                           1.4 MB/s | 1.3 MB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : dhcp-common-12:4.4.2-15.b1.el9.noarch                  1/2 
  Running scriptlet: dhcp-server-12:4.4.2-15.b1.el9.aarch64                 2/2 
useradd warning: dhcpd s uid 177 outside of the SYS_UID_MIN 201 and SYS_UID_MAX 999 range.

  Installing       : dhcp-server-12:4.4.2-15.b1.el9.aarch64                 2/2 
  Running scriptlet: dhcp-server-12:4.4.2-15.b1.el9.aarch64                 2/2 
  Verifying        : dhcp-common-12:4.4.2-15.b1.el9.noarch                  1/2 
  Verifying        : dhcp-server-12:4.4.2-15.b1.el9.aarch64                 2/2 

Installed:
  dhcp-common-12:4.4.2-15.b1.el9.noarch  dhcp-server-12:4.4.2-15.b1.el9.aarch64 

Complete!
```

**Configuration du serveur DHCP**
```powershell
# specify DNS server's hostname or IP address
option domain-name-servers     1.1.1.1;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 172.16.72.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range 172.16.72.13 172.16.72.253;
    # specify broadcast address
    option broadcast-address 172.16.72.255;
    # specify gateway
    option routers 172.16.72.254;
}
```

**Le service est fonctionnel**
```powershell
[user1@John ~]$ systemctl status dhcpd.service
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor pre>
     Active: active (running) since Fri 2022-10-28 09:29:43 CEST; 9min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 1237 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 5877)
     Memory: 4.6M
        CPU: 12ms
     CGroup: /system.slice/dhcpd.service
             └─1237 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -gr>

Oct 28 09:29:43 John dhcpd[1237]: Sending on   Socket/fallback/fallback-net
Oct 28 09:29:43 John dhcpd[1237]: Server starting service.
Oct 28 09:29:43 John systemd[1]: Started DHCPv4 Server Daemon.
Oct 28 09:31:47 John dhcpd[1237]: DHCPDISCOVER from ba:00:66:cd:a4:bb via enp0s1
Oct 28 09:31:47 John dhcpd[1237]: DHCPREQUEST for 172.16.72.5 (172.16.72.1) fro>
Oct 28 09:31:48 John dhcpd[1237]: DHCPOFFER on 172.16.72.13 to ba:00:66:cd:a4:b>
Oct 28 09:36:37 John dhcpd[1237]: DHCPDISCOVER from ba:00:66:cd:a4:bb via enp0s1
Oct 28 09:36:37 John dhcpd[1237]: DHCPREQUEST for 172.16.72.5 (172.16.72.1) fro>
Oct 28 09:36:38 John dhcpd[1237]: DHCPOFFER on 172.16.72.13 to ba:00:66:cd:a4:b>
Oct 28 09:37:09 John dhcpd[1237]: DHCPREQUEST for 172.16.72.5 from ba:00:66:cd:>
lines 1-23
```

**Récupération adresse IP**
```powershell
[user1@Bob ~]$ sudo dhclient -v enp0s1 -s 172.16.72.11
Internet Systems Consortium DHCP Client 4.4.2b1
Copyright 2004-2019 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/enp0s1/ba:00:66:cd:a4:bb
Sending on   LPF/enp0s1/ba:00:66:cd:a4:bb
Sending on   Socket/fallback
DHCPDISCOVER on enp0s1 to 172.16.72.11 port 67 interval 4 (xid=0x628e507a)
DHCPOFFER of 172.16.72.13 from 172.16.72.11
DHCPREQUEST for 172.16.72.13 on enp0s1 to 172.16.72.11 port 67 (xid=0x628e507a)
DHCPACK of 172.16.72.13 from 172.16.72.11 (xid=0x628e507a)
```

```powershell
[user1@Bob ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether ba:00:66:cd:a4:bb brd ff:ff:ff:ff:ff:ff
    inet 172.16.72.5/24 brd 172.16.72.255 scope global dynamic noprefixroute enp0s1
       valid_lft 86109sec preferred_lft 86109sec
    inet 172.16.72.13/24 brd 172.16.72.255 scope global secondary dynamic enp0s1
       valid_lft 557sec preferred_lft 557sec
    inet6 fe80::b800:66ff:fecd:a4bb/64 scope link 
       valid_lft forever preferred_lft forever
```

**Ping passerelle**
```powershell
[user1@Bob ~]$ ping 172.16.72.254
PING 172.16.72.254 (172.16.72.254) 56(84) bytes of data.
64 bytes from 172.16.72.254: icmp_seq=1 ttl=64 time=2.20 ms
64 bytes from 172.16.72.254: icmp_seq=2 ttl=64 time=1.82 ms
64 bytes from 172.16.72.254: icmp_seq=3 ttl=64 time=2.89 ms
64 bytes from 172.16.72.254: icmp_seq=4 ttl=64 time=2.08 ms
```

**Route par défaut**
```powershell
[user1@Bob ~]$ ip r s
default via 172.16.72.254 dev enp0s1 
172.16.72.0/24 dev enp0s1 proto kernel scope link src 172.16.72.5 metric 100
```

**Ping 1.1.1.1**
```powershell
[user1@Bob ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=69.0 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=101 ms
```

**Commande dig**
```powershell
[user1@Bob ~]$ dig gitlab.com

; <<>> DiG 9.16.23-RH <<>> gitlab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55470
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;gitlab.com.			IN	A

;; ANSWER SECTION:
gitlab.com.		187	IN	A	172.65.251.78

;; Query time: 260 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Fri Oct 28 10:26:56 CEST 2022
;; MSG SIZE  rcvd: 55
```

**Ping gitlab.com**
```powershell
[user1@Bob ~]$ ping gitlab.com
PING gitlab.com (172.65.251.78) 56(84) bytes of data.
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=1 ttl=53 time=27.6 ms
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=2 ttl=53 time=31.3 ms
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=3 ttl=53 time=40.1 ms
```

### 2) Analyse de trames 
  

**Command DHCPClient**

* release
```powershell
sudo dhcpclient -r -v -i enp0s1
```

* Mise en place d'une nouvelle adresse IP sur Bob
```powershell
sudo dhcpclient -v -i enp0s1 -s 172.16.72.11
```

[tp3_DHCP](./tp3_DHCP.pcap)

