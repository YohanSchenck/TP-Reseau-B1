# TP 1 : Premier pas réseau

[https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/1](https://gitlab.com/it4lik/b1-reseau-2022/-/tree/main/tp/1)

## I. Exploration locale en solo

### 1) Affichage informations sur la pile TCP/IP locale

**a)**  Commande réalisée : ifconfig

interface wifi : 

- nom  = en0
- adresse MAC = 1c:57:dc:3e:e4:16
- adresse ip = 10.33.16.214

interface ethernet :

- nom = en5
- adresse MAC : 50:a0:30:02:ef:59
- adresse ip : aucune donnée car pas connecté par ethernet

b) commande réalisée : route get default | grep gateway

- adresse IP de la passerelle : 10.33.19.254

c) commande réalisée : arp -a

- adresse MAC de la passerelle : 0:c0:e7:e0:4:4e

d) Voici la direction suivie : Information Système/Réseau/

![Capture d’écran 2022-10-03 à 11.07.48.png](TP%201%20Premier%20pas%20re%CC%81seau%20768dfc0df62a4d7698ef8022207f7359/Capture_decran_2022-10-03_a_11.07.48.png)

![Capture d’écran 2022-10-03 à 11.07.57.png](TP%201%20Premier%20pas%20re%CC%81seau%20768dfc0df62a4d7698ef8022207f7359/Capture_decran_2022-10-03_a_11.07.57.png)

### 2) Modification des informations

a) Chemin suivi : Préférence Système/Réseau/TCP/IP

Configuration IPV4 : manuelle

Nouvelle adresse indiquée : 10.33.16.20

Si deux ordinateurs possèdent la même IP, la deuxième personne à obtenir l’adresse IP ne peut recevoir de message. 

## II. Exploration en duo

### 3) Modification de l’adresse IP

a) Adresse modifiée : 10.10.10.1

Masque ou sous réseau sur mac os : 255.255.255.0

b) Vérification 

```powershell
en5: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
options=400<CHANNEL_IO>
ether 50:a0:30:02:ef:59
inet6 fe80::189a:a37a:5b2d:7389%en5 prefixlen 64 secured scopeid 0x14
inet 10.10.10.1 netmask 0xffff0000 broadcast 10.10.255.255
nd6 options=201<PERFORMNUD,DAD>
media: autoselect (1000baseT <full-duplex>)
status: active
```

c) 

```markdown
MacBook-Air-Yohan:TP-Reseau-B1 yohan$ ping 10.10.10.2
PING 10.10.10.2 (10.10.10.2): 56 data bytes
64 bytes from 10.10.10.2: icmp_seq=0 ttl=128 time=1.271 ms
64 bytes from 10.10.10.2: icmp_seq=1 ttl=128 time=1.470 ms
64 bytes from 10.10.10.2: icmp_seq=2 ttl=128 time=1.368 ms
64 bytes from 10.10.10.2: icmp_seq=3 ttl=128 time=1.787 ms
64 bytes from 10.10.10.2: icmp_seq=4 ttl=128 time=1.860 ms
64 bytes from 10.10.10.2: icmp_seq=5 ttl=128 time=1.531 ms
```

d)

```powershell
(10.10.10.2) at 8:8f:c3:4b:cc:81 on en5 ifscope [ethernet]
```

L’adresse MAC du correspondant est : 8:8f:c3:4b:cc:81

### 4) utilisation d’un des deux comme gateway

a) 

```powershell
MacBook-Air-Yohan:TP-Reseau-B1 yohan$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1): 56 data bytes
64 bytes from 1.1.1.1: icmp_seq=0 ttl=54 time=22.766 ms
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=22.331 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=22.202 ms
```

b)  Après définition du serveur DNS : 1.1.1.1

```powershell
MacBook-Air-Yohan:TP-Reseau-B1 yohan$ traceroute [google.com](http://google.com/)
traceroute to [google.com](http://google.com/) (216.58.214.78), 64 hops max, 52 byte packets
1  192.168.137.1 (192.168.137.1)  2.059 ms *  1.451 ms
```

### 5) Petit chat privé

a) Partie à rajouter
On crée un numéro de port avec la commande nc.exe -l -p 8888, le port est 8888 et nous devons ensuite attendre que le client se connecte pour établir une connexion.
Le "l" signifie l'écoute et le port pour désigné un port.

b) 

```powershell
MacBook-Air-Yohan:~ yohan$ nc 192.168.137.1 8888
qalut

bien ouej

<3

salam
```

c) Visualisation de connexion

```powershell
MacBook-Air-Yohan:~ yohan$ netstat -a -n -p tcp | grep 8888
tcp4       0      0  192.168.137.2.51803    192.168.137.1.8888     ESTABLISHED
```

d) Pour aller plus loin

Si on indique pas d’interface d’écoute, il est possible de connecter à partir de n’importe quelle interface. 

### 6) Firewall

A*utoriser les ping*

*- Appuyez sur Démarrer, tapez «pare-feu Windows avec», puis lancez «Pare-feu Windows avec sécurité avancée».
- Vous allez créer une nouvelle règles: une pour autoriser les requêtes ICMPv4
- Vous allez créer deux nouvelles règles: une pour autoriser les requêtes ICMPv4
- Dans la fenêtre « Assistant Nouvelle règle entrante », sélectionnez « Personnalisé », puis cliquez sur « Suivant »
- Sur la page suivante, assurez-vous que «Tous les programmes» est sélectionné, puis cliquez sur «Suivant».
- Sur la page suivante, choisissez «ICMPv4» dans la liste déroulante «Type de protocole», puis cliquez sur le bouton «Personnaliser».
- Dans la fenêtre «Personnaliser les paramètres ICMP», sélectionnez l’option «Types ICMP spécifiques». Dans la liste des types ICMP, activez « Demande d’écho », puis cliquez sur « OK ».
- De retour dans la fenêtre « Assistant Nouvelle règle entrante », vous êtes prêt à cliquer sur « Suivant ».
- Sur la page suivante, il est plus simple de s’assurer que les options «Toute adresse IP» sont sélectionnées pour les adresses IP locales et distantes puis faite suivant
- Sur la page suivante, assurez-vous que l’option « Autoriser la connexion » est activée, puis cliquez sur « Suivant ».
- La page suivante vous permet de contrôler le moment où la règle est active- Tous sélectionner et suivant
- Et enfin donner le nom que l'on souhaite donc ping ICMP

Autoriser le traffic sur le port qu'utilise nc

- Retourner au même endroit pour créer une nouvelle règle dans le trafic entrant
- Dans la page Type de règle de l’Assistant Nouvelle règle de trafic entrant, cliquez sur Personnalisé, puis sur Suivant.
- Dans la page Programme , cliquez sur Tous les programmes, puis sur Suivant
- Dans la page Protocole et ports , sélectionnez le type de protocole que vous souhaitez autoriser. Pour limiter la règle à un numéro de port spécifié, vous devez sélectionner TCP ou UDP. Dans notre cas nous pouvons spécifié notre port et alors une échelle de port.
- Une fois que vous avez configuré les protocoles et les ports, cliquez sur Suivant.
- Dans la page Étendue cliquez sur Suivant
- Dans la page Action , sélectionnez Autoriser la connexion, puis cliquez sur Suivant.
- Dans la page Profil , sélectionnez les types d’emplacement réseau auxquels cette règle s’applique, puis cliquez sur Suivant. Pour nous on sélectionne tous.
- Dans la page Nom , tapez un nom et une description pour votre règle, puis cliquez sur Terminer, Dans notre cas je l'ai appelé TCP.*

## III) Manipulation d’autres outils/protocoles côté client

### 1) DHCP

Voilà la commande réalisée : 

```powershell
MacBook-Air-Yohan:~ yohan$ ipconfig getpacket en0
```

Voici le résultat :

```powershell
server_identifier (ip): 10.33.19.254
lease_time (uint32): 0x1502b
```

On trouve ainsi l’adresse IP du serveur DHCP ainsi que la durée de vie de cette adresse. 

Un serveur DHCP donne tous les outils pour accéder à internet (adresse ip, adresse passerelle, serveur DNS). Le lease time est une durée au bout de laquelle le serveur DHCP questionne l’ordinateur sur sa présence sur le réseau et sur sa volonté d’avoir toujours la même IP. Dans le même temps, l’ordinateur questionne le serveur DHCP sur la disponibilité de l’adresse IP. 

### 2) DNS

a)

```powershell
MacBook-Air-Yohan:~ yohan$ scutil --dns | grep nameserver
nameserver[0] : 8.8.8.8
nameserver[1] : 8.8.4.4
nameserver[2] : 1.1.1.1
```

b)

```powershell
MacBook-Air-Yohan:~ yohan$ nslookup google.com
Server:		8.8.8.8 //serveur DNS utilisé
Address:	8.8.8.8#53 //adresse du serveur dns

Non-authoritative answer:

Name:	google.com //URL 

Address: 216.58.209.238 // adresse IPV4 du serveur google
```

```powershell
MacBook-Air-Yohan:~ yohan$ nslookup ynov.com
Server:		8.8.8.8

Address:	8.8.8.8#53

Non-authoritative answer:

Name:	ynov.com

Address: 172.67.74.226 //adresse du serveur 1 d Ynov

Name:	ynov.com

Address: 104.26.10.233 //adresse du serveur 2 d Ynov

Name:	ynov.com

Address: 104.26.11.233 // adresse du serveur 3 d Ynov
```

```powershell
MacBook-Air-Yohan:~ yohan$ nslookup 231.34.113.12
Server:		8.8.8.8

Address:	8.8.8.8#53

- * server can't find 12.113.34.231.in-addr.arpa: NXDOMAIN //Il n y a pas de domaine pour cette adresse.
```

```powershell
MacBook-Air-Yohan:~ yohan$ nslookup 78.34.2.17
Server:		8.8.8.8

Address:	8.8.8.8#53

Non-authoritative answer:

17.2.34.78.in-addr.arpa	name = cable-78-34-2-17.nc.de. //Il y a un nom de domaine. 

Authoritative answers can be found from
```

### IV) Wireshark

a) 

![Untitled](TP%201%20Premier%20pas%20re%CC%81seau%20768dfc0df62a4d7698ef8022207f7359/Untitled.png)

![Untitled](TP%201%20Premier%20pas%20re%CC%81seau%20768dfc0df62a4d7698ef8022207f7359/Untitled%201.png)

Essai avec nslookup blendermarket.com

![Untitled](TP%201%20Premier%20pas%20re%CC%81seau%20768dfc0df62a4d7698ef8022207f7359/Untitled%202.png)

b) 

![Untitled](TP%201%20Premier%20pas%20re%CC%81seau%20768dfc0df62a4d7698ef8022207f7359/Untitled%203.png)

En observant le flux réseau , nous observons une répétition de flux ci dessus. Cela est sûrement dû au visionnage d’une vidéo. Voici les informations demandées :

- l’ordinateur se connecte à cette adresse IP : 2a00:1450:4007:8:
- Le port est 443.