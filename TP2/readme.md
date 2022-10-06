# TP2 : Ethernet, IP, et ARP

Dans ce TP on va approfondir trois protocoles, qu'on a survolé jusqu'alors :

- **IPv4** *(Internet Protocol Version 4)* : gestion des adresses IP
  - on va aussi parler d'ICMP, de DHCP, bref de tous les potes d'IP quoi !
- **Ethernet** : gestion des adresses MAC
- **ARP** *(Address Resolution Protocol)* : permet de trouver l'adresse MAC de quelqu'un sur notre réseau dont on connaît l'adresse IP


# Sommaire

- [TP2 : Ethernet, IP, et ARP](#tp2--ethernet-ip-et-arp)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. Setup IP](#i-setup-ip)
- [II. ARP my bro](#ii-arp-my-bro)
- [II.5 Interlude hackerzz](#ii5-interlude-hackerzz)
- [III. DHCP you too my brooo](#iii-dhcp-you-too-my-brooo)

# 0. Prérequis

# I. Setup IP

Le lab, il vous faut deux machines : 

- les deux machines doivent être connectées physiquement
- vous devez choisir vous-mêmes les IPs à attribuer sur les interfaces réseau, les contraintes :
  - IPs privées (évidemment n_n)
  - dans un réseau qui peut contenir au moins 1000 adresses IP (il faut donc choisir un masque adapté)
  - oui c'est random, on s'exerce c'est tout, p'tit jog en se levant c:
  - le masque choisi doit être le plus grand possible (le plus proche de 32 possible) afin que le réseau soit le plus petit possible

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

- vous renseignerez dans le compte rendu :
  - les deux IPs choisies, en précisant le masque
  - l'adresse de réseau
  - l'adresse de broadcast
- vous renseignerez aussi les commandes utilisées pour définir les adresses IP *via* la ligne de commande

>IP1: 10.172.89.1 /22 <br>
>IP2: 10.172.89.2 /22 <br>
>Adresse réseau: 10.172.88.0 <br>
>Adresse Broadcast: 10.172.91.255

>netsh interface ip set address "Ethernet" static 10.172.89.2 255.255.252.0 

>netsh interface ip set address "Ethernet" static 10.172.89.1 255.255.252.0 


🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

> ping 10.172.89.1 depuis 10.172.89.2

```
Envoi d’une requête 'Ping'  10.172.89.1 avec 32 octets de données :
Réponse de 10.172.89.1 : octets=32 temps=5 ms TTL=128
Réponse de 10.172.89.1 : octets=32 temps=3 ms TTL=128
Réponse de 10.172.89.1 : octets=32 temps=3 ms TTL=128
Réponse de 10.172.89.1 : octets=32 temps=3 ms TTL=128

Statistiques Ping pour 10.172.89.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 3ms, Maximum = 5ms, Moyenne = 3ms
```

🌞 **Wireshark it**

>Type de paquets ICMP pour la request: 8 (Echo request)
>Type de paquets ICMP pour la reply: 0 (Echo reply)

🦈 **PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP**

>Source: ./src/ping.pcapng

# II. ARP my bro

🌞 **Check the ARP table**

>arp -a

```
  Adresse Internet      Adresse physique      Type
  10.172.89.1           b4-45-06-bf-f7-76     dynamique
```

MAC du Bynome: ```b4-45-06-bf-f7-76```

>Gateway du réseau

```
Adresse Internet      Adresse physique      Type
10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```
MAC du gateway : ```00-c0-e7-e0-04-4e ```

🌞 **Manipuler la table ARP**

>arp -d

Avant 

```
Interface : 10.172.89.2 --- 0xb
  Adresse Internet      Adresse physique      Type
  10.172.89.1           b4-45-06-bf-f7-76     dynamique
  10.172.91.255         ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

Après avoir été vidée

```
Interface : 10.172.89.2 --- 0xb
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

Il n'y a plus l'adresse IP de notre machine ni l'adresse de broadcast.


🌞 **Wireshark it**

>Request 

```
Adresse MAC source : 60:18:95:44:10:10 (MAC de ma machine)
Adresse MAC de destination: ff:ff:ff:ff:ff:ff (Adresse MAC Broadcast)
```

>Reply
```
Adresse MAC source : b4:45:06:bf:f7:76 (Autre pc de la LAN)
Adresse MAC destination: 60:18:95:44:10:10 (MAC de ma machine)
```

🦈 **PCAP qui contient les trames ARP**

Source: ./src/ARP.pacpng

# II.5 Interlude hackerzz


# III. DHCP you too my brooo

🌞 **Wireshark it**

>Discover
```
Source: dc:21:5c:96:22:51 (Moi)
Destination: ff:ff:ff:ff:ff:ff (Broadcast)
```

>Offer
```
Source: 00:c0:e7:e0:04:4e 
Destination: dc:21:5c:96:22:51 (Moi)
```

>Request
```
Source: dc:21:5c:96:22:51 (Moi)
Destination: ff:ff:ff:ff:ff:ff (Broadcast)
```

>ACK
```
Source: 00:c0:e7:e0:04:4e
Destination: dc:21:5c:96:22:51 (Moi)
```

>IP à utiliser:
```10.33.16.170```

>IP de la passerelle
```10.33.19.254```

>DNS:
```8.8.8.8```


🦈 **PCAP qui contient l'échange DORA**

Source : ./src/DORA.pcapng