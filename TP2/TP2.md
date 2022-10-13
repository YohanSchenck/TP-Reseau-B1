# TP 2 : Réseau

[https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/2](https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/2)

# I) Setup

a)

Voici la commande réalisée :

```powershell
MacBook-Air-Yohan:~ yohan$ networksetup -setmanual AX88179A 10.10.10.2 255.255.252.0 10.33.19.254
```

Cela fonctionne bien comme en témoigne les lignes suivantes 

```powershell
en5: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
options=400<CHANNEL_IO>
ether 50:a0:30:02:ef:59
inet6 fe80::18fa:ef4e:de55:fa18%en5 prefixlen 64 secured scopeid 0x7
inet 10.10.10.2 netmask 0xfffffc00 broadcast 10.10.11.255
nd6 options=201<PERFORMNUD,DAD>
media: autoselect (1000baseT <full-duplex>)
status: active
```

Mon ip : 10.10.10.2

L’ip de mon partenaire : 10.10.10.127

L’adresse de réseau : 10.10.8.0

L’adresse de broadcast : 10.10.11.255

b)

```powershell
MacBook-Air-Yohan:~ yohan$ ping 10.10.10.127
PING 10.10.10.127 (10.10.10.127): 56 data bytes
64 bytes from 10.10.10.127: icmp_seq=0 ttl=128 time=1.779 ms
64 bytes from 10.10.10.127: icmp_seq=1 ttl=128 time=2.007 ms
64 bytes from 10.10.10.127: icmp_seq=2 ttl=128 time=2.031 ms
64 bytes from 10.10.10.127: icmp_seq=3 ttl=128 time=2.073 ms
```

c)  
  
[IP-Wireshark](./Wireshark/DHCP1.pcapng)

Pour le ping : type 8 (Echo Request)

Pour le pong : type 0 (Echo Reply)

# II) ARP my bro

a) 

```powershell
MacBook-Air-Yohan:~ yohan$ arp -a
? (10.10.10.127) at 8:8f:c3:a:24:53 on en5 ifscope [ethernet]
? (10.33.16.72) at a6:a4:44:9d:9f:41 on en0 ifscope [ethernet]
? (10.33.19.89) at 3c:6:30:33:47:ed on en0 ifscope [ethernet]
? (10.33.19.254) at 0:c0:e7:e0:4:4e on en0 ifscope [ethernet]
? (224.0.0.251) at 1:0:5e:0:0:fb on en0 ifscope permanent [ethernet]
```

L’adresse MAC de mon partenaire : 8:8f:c3:a:24:53

L’adresse MAC de la gateway : 0:c0:e7:e0:4:4e

b)

```powershell
MacBook-Air-Yohan:~ yohan$ arp -a
? (10.33.16.52) at 56:ac:3:b8:3c:d4 on en0 ifscope [ethernet]
? (10.33.16.100) at c6:20:1c:b0:d:47 on en0 ifscope [ethernet]
? (10.33.16.131) at 9e:84:cb:ea:9e:c6 on en0 ifscope [ethernet]
? (10.33.16.199) at 4c:3:4f:e7:6a:fd on en0 ifscope [ethernet]
? (10.33.17.7) at 8a:d8:b7:7f:27:b1 on en0 ifscope [ethernet]
? (10.33.17.112) at 18:65:90:ce:f5:63 on en0 ifscope [ethernet]
? (10.33.17.150) at a4:83:e7:76:1a:b5 on en0 ifscope [ethernet]
? (10.33.17.177) at e2:4:cd:4:bd:99 on en0 ifscope [ethernet]
? (10.33.17.185) at 20:69:80:c3:94:99 on en0 ifscope [ethernet]
? (10.33.17.186) at 2a:df:c3:65:de:ef on en0 ifscope [ethernet]
? (10.33.17.187) at 14:7d:da:99:2a:b7 on en0 ifscope [ethernet]
? (10.33.18.84) at b2:9e:52:49:d5:e1 on en0 ifscope [ethernet]
? (10.33.18.221) at 78:4f:43:87:f5:11 on en0 ifscope [ethernet]
? (10.33.19.254) at 0:c0:e7:e0:4:4e on en0 ifscope [ethernet]
```

Suite à la commande arp -a -d 

```powershell
MacBook-Air-Yohan:~ yohan$ arp -a
? (10.33.16.72) at a6:a4:44:9d:9f:41 on en0 ifscope [ethernet]
? (10.33.16.73) at a2:12:60:6b:b7:9d on en0 ifscope [ethernet]
? (10.33.16.134) at 4e:6f:c7:b9:64:af on en0 ifscope [ethernet]
? (10.33.17.1) at 9c:3e:53:8c:90:1b on en0 ifscope [ethernet]
? (10.33.17.7) at 8a:d8:b7:7f:27:b1 on en0 ifscope [ethernet]
? (10.33.17.168) at 42:70:74:58:43:37 on en0 ifscope [ethernet]
? (10.33.19.254) at 0:c0:e7:e0:4:4e on en0 ifscope [ethernet]
```

Suite au ping 

```powershell
MacBook-Air-Yohan:~ yohan$ arp -a
? (10.10.10.127) at 8:8f:c3:a:24:53 on en5 ifscope [ethernet]
? (10.33.16.4) at ac:5f:3e:79:e6:34 on en0 ifscope [ethernet]
? (10.33.16.15) at 3e:da:c8:2a:ea:5e on en0 ifscope [ethernet]
? (10.33.16.42) at 8e:5d:2f:a5:51:a5 on en0 ifscope [ethernet]
? (10.33.16.64) at 36:e:6f:d2:af:fc on en0 ifscope [ethernet]
```

L’adresse MAC de mon camarade est bien présente. 

c) [ARP-Wireshark](./Wireshark/ARP.pcapng)  
  
  L'arp broadcast fait une requête pour savoir à qui appartient l'adresse IP 10.10.10.127 et la donner à l'adresse IP 10.10.10.2. 
  L'arp reply est la réponse et donne l'adresse MAC à l'adresse ip 10.10.10.2  
    

    
    

# III) DHCP you too my bro
  
[DHCP-Wireshark](./Wireshark/DHCP1.pcapng)

- D 

    - adresse source : **0.0.0.0**
    - adresse destination : **255.255.255.255**
- O
    - adresse source : **10.33.19.254**
    - adresse destination : **10.33.19.6**
    - identification des éléments (IP, passerelle et DNS)
        - Your (client) IP address : 10.33.19.6
        - Router : 10.33.19.254
        - Domain Name Server  :
            - 8.8.8.8
            - 8.8.4.4
            - 1.1.1.1
- R
    - adresse source : **0.0.0.0**
    - adresse destination : **255.255.255.255**
    - identification des éléments (IP, passerelle et DNS)
        - Requested IP :  10.33.19.6
        - DHCP Server Identifier :  10.33.19.254
- A
    - adresse source : **10.33.19.254**
    - adresse destination : **10.33.17.6**
    - identification des éléments (IP, passerelle et DNS)
        - Your (client) IP address : 10.33.19.6
        - Router : 10.33.19.254
        - Domain Name Server :
            - 8.8.8.8
            - 8.8.4.4
            - 1.1.1.1