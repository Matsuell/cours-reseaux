# TP1 - Premier pas réseau

Le terme *réseau* désigne au sens large toutes les fonctionnalités d'un PC permettant de se connecter à d'autres machines.  

Le protocole IP est très important, il est central dans l'utilisation du réseau moderne.

> On va voir IPv4 en cours, il existe aussi IPv6, plus récent, qui fonctionne sur les mêmes principes. Nous en parlerons aussi en cours.

---

Lorsque l'on parle de réseau, on désigne souvent par le terme *client* tout équipement qui porte une adresse IP.

Donc vos PCs sont des *clients*, et on va explorer leur *réseau* dans ce TP.

# Sommaire
- [TP1 - Premier pas réseau](#tp1---premier-pas-réseau)
- [Sommaire](#sommaire)
- [Déroulement et rendu du TP](#déroulement-et-rendu-du-tp)
- [I. Exploration locale en solo](#i-exploration-locale-en-solo)
  - [1. Affichage d'informations sur la pile TCP/IP locale](#1-affichage-dinformations-sur-la-pile-tcpip-locale)
    - [En ligne de commande](#en-ligne-de-commande)
    - [En graphique (GUI : Graphical User Interface)](#en-graphique-gui--graphical-user-interface)
  - [2. Modifications des informations](#2-modifications-des-informations)
    - [A. Modification d'adresse IP (part 1)](#a-modification-dadresse-ip-part-1)
- [II. Exploration locale en duo](#ii-exploration-locale-en-duo)
  - [1. Prérequis](#1-prérequis)
  - [2. Câblage](#2-câblage)
  - [Création du réseau (oupa)](#création-du-réseau-oupa)
  - [3. Modification d'adresse IP](#3-modification-dadresse-ip)
  - [4. Utilisation d'un des deux comme gateway](#4-utilisation-dun-des-deux-comme-gateway)
  - [5. Petit chat privé](#5-petit-chat-privé)
  - [6. Firewall](#6-firewall)
- [III. Manipulations d'autres outils/protocoles côté client](#iii-manipulations-dautres-outilsprotocoles-côté-client)
  - [1. DHCP](#1-dhcp)
  - [2. DNS](#2-dns)
- [IV. Wireshark](#iv-wireshark)
- [Bilan](#bilan)


# I. Exploration locale en solo

## 1. Affichage d'informations sur la pile TCP/IP locale

### En ligne de commande

En utilisant la ligne de commande (CLI) de votre OS :

**🌞 Affichez les infos des cartes réseau de votre PC**

>ipconfig /all

```
Carte réseau sans fil Wi-Fi :

   Description. . . . . . . . . . . . . . : Intel(R) Wi-Fi 6 AX201 160MHz
   Adresse physique . . . . . . . . . . . : DC-21-5C-96-22-51
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.73(préféré)
```

```
Carte Ethernet Ethernet :

   Description. . . . . . . . . . . . . . : Realtek PCIe GbE Family Controller
   Adresse physique . . . . . . . . . . . : 60-18-95-44-10-10
   No IP because disconnected
```

---

**🌞 Affichez votre gateway**

>ipconfig

```
Carte réseau sans fil Wi-Fi :

   Passerelle par défaut. . . . . . . . . : 10.33.19.254
```

---

**🌞 Déterminer la MAC de la passerelle**

>arp -a

```
Interface : 10.33.16.73 --- 0x7
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```

---

### En graphique (GUI : Graphical User Interface)

En utilisant l'interface graphique de votre OS :  

**🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)**

>Panneau de configuration -> Réseaux et internet -> Centre réseaux et partage -> Modifier les paramètres de la carte-> Clique droit Réseaux Ynov -> Statut -> Avancé

```
Propriété                    Valeur
Adresse IPv4                 10.33.16.73
Adresse Physique             DC-21-5C-96-22-51
Passerelle par défaut IPv4   10.33.19.254
```

---

## 2. Modifications des informations

### A. Modification d'adresse IP (part 1)  

🌞 Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :

>Panneau de configuration -> Réseaux et internet -> Centre réseaux et partage -> Modifier les paramètres de la carte-> Clique droit Réseaux Ynov -> Propriétés -> Protocole Internet version 4 (TCP/IPv4)

```
Adresse IP: 10.33.16.25
Masque de sous-réseau: 255.255.252.0
```

---

🌞 **Il est possible que vous perdiez l'accès internet. Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.**

>On perd l'accès à internet car le réseau ne parvient plus à nous identifier même si la nouvelle adresse IP reste dans le même réseau (changment du dernier octet).

---

# II. Exploration locale en duo

Owkay. Vous savez à ce stade :

- afficher les informations IP de votre machine
- modifier les informations IP de votre machine
- c'est un premier pas vers la maîtrise de votre outil de travail

On va maintenant répéter un peu ces opérations, mais en créant un réseau local de toutes pièces : entre deux PCs connectés avec un câble RJ45.

## 1. Prérequis

- deux PCs avec ports RJ45
- un câble RJ45
- **firewalls désactivés** sur les deux PCs

## 2. Câblage

Ok c'est la partie tendue. Prenez un câble. Branchez-le des deux côtés. **Bap.**

## Création du réseau (oupa)

Cette étape pourrait paraître cruciale. En réalité, elle n'existe pas à proprement parlé. On ne peut pas "créer" un réseau.

**Si une machine possède une carte réseau, et si cette carte réseau porte une adresse IP**, alors cette adresse IP se trouve dans un réseau (l'adresse de réseau). Ainsi, **le réseau existe. De fait.**  

**Donc il suffit juste de définir une adresse IP sur une carte réseau pour que le réseau existe ! Bap.**

## 3. Modification d'adresse IP

🌞 **Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau**

>Panneau de configuration -> Réseaux et internet -> Centre réseaux et partage -> Modifier les paramètres de la carte-> Clique droit Réseaux Ynov -> Propriétés -> Protocole Internet version 4 (TCP/IPv4)

```
Adresse IP: 10.10.10.155
Masque de sous-réseau: 255.255.255.0
```
---

🌞 **Vérifier à l'aide d'une commande que votre IP a bien été changée**

>ipconfig

```
Carte Ethernet Ethernet :

   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.155
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
```
---

🌞 **Vérifier que les deux machines se joignent**

- utilisez la commande `ping` pour tester la connectivité entre les deux machines

> ping 10.10.10.252

```
Envoi d’une requête 'Ping'  10.10.10.252 avec 32 octets de données :
Réponse de 10.10.10.252 : octets=32 temps=6 ms TTL=128
Réponse de 10.10.10.252 : octets=32 temps=3 ms TTL=128
Réponse de 10.10.10.252 : octets=32 temps=3 ms TTL=128
Réponse de 10.10.10.252 : octets=32 temps=3 ms TTL=128

Statistiques Ping pour 10.10.10.252:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 3ms, Maximum = 6ms, Moyenne = 3ms
```

---

🌞 **Déterminer l'adresse MAC de votre correspondant**

> arp -a

```
Interface : 10.10.10.155 --- 0xb
  Adresse Internet      Adresse physique      Type
  10.10.10.252          b4-45-06-bf-f7-76     dynamique
```
---

## 4. Utilisation d'un des deux comme gateway

🌞**Tester l'accès internet**

>ping 8.8.8.8

```
Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
Réponse de 8.8.8.8 : octets=32 temps=23 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=24 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=23 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=24 ms TTL=113
```

---

🌞 **Prouver que la connexion Internet passe bien par l'autre PC**

>Après avoir changé d'adresse IP car lorsque l'on a partagé la connexion l'adresse IP de la carte ethernet avait changé donc il a fallu modifier celle du client pour qu'il soit dans le même réseau que le serveur.

>tracert 192.168.137.1

```
Détermination de l’itinéraire vers DESKTOP-URQ404I [192.168.137.1]
avec un maximum de 30 sauts :

  1     1 ms     1 ms    <1 ms  DESKTOP-URQ404I [192.168.137.1]

Itinéraire déterminé.
```

---

## 5. Petit chat privé

🌞 **sur le PC *serveur*** avec par exemple l'IP 192.168.1.1
>nc.exe -l -p 8888

```
Client
Serveur
```

---
🌞 **sur le PC *client*** avec par exemple l'IP 192.168.1.2

>nc.exe 192.168.137.1 8888

```
Client
Serveur
```
---

🌞 **Visualiser la connexion en cours**

>netstat -a -n -b

```
[nc.exe]
  TCP    192.168.137.1:61462   20.54.232.160:443      ESTABLISHED
  CDPUserSvc_ec5778
```
---

🌞 **Pour aller un peu plus loin**

>nc.exe -l -p 8888 -s 192.168.137.25

```
netstat -a -n -b  | findstr 8888
  TCP    192.168.137.25:8888     0.0.0.0:0              LISTENING
```

---
## 6. Firewall


🌞 **Activez et configurez votre firewall**

>Pare-feu Windows Defender -> Paramètres avancés -> Clique droit (Règles de trafic entrant) -> Nouvelle règle -> Protocole et ports  -> TCP -> Ports locaux spécifiques :8888 -> Autoriser la connexion <br> Ensuite on défini seulement pour une adresse IP dans les propriétés de la règle.


>ping 192.168.137.1

```
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
```


>ping 192.168.137.25

```
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
```

---
# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP

Bon ok vous savez définir des IPs à la main. Mais pour être dans le réseau YNOV, vous l'avez jamais fait.  

C'est le **serveur DHCP** d'YNOV qui vous a donné une IP.

Une fois que le serveur DHCP vous a donné une IP, vous enregistrer un fichier appelé *bail DHCP* qui contient, entre autres :

- l'IP qu'on vous a donné
- le réseau dans lequel cette IP est valable

🌞**Exploration du DHCP, depuis votre PC**


>ipconfig /all

```
   Bail obtenu. . . . . . . . . . . . . . : mardi 4 octobre 2022 13:39:45
   Bail expirant. . . . . . . . . . . . . : mercredi 5 octobre 2022 13:39:41
   Serveur DHCP . . . . . . . . . . . . . : 10.33.19.254

```
> Chez vous, c'est votre box qui fait serveur DHCP et qui vous donne une IP quand vous le demandez.
---
## 2. DNS

🌞** Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**

>ipconfig /all

```
   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
                                       1.1.1.1
```
---
🌞 Utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main

>nslookup google.com

```
Addresses:  2a00:1450:4007:813::200e
          172.217.18.206
```

>nslookup ynov.com

```
Addresses:  2606:4700:20::ac43:4ae2
          2606:4700:20::681a:ae9
          2606:4700:20::681a:be9
          104.26.10.233
          104.26.11.233
          172.67.74.226
```

Cette commande nous retourne les différentes adresse du site passé en argument, pour ynov.com il nous renvoie plusieurs adresses , car il est présent sur plusieurs serveurs.


>nslookup 78.73.21.21

```
Serveur :   dns.google
Address:  8.8.8.8

Nom :    78-73-21-21-no168.tbcn.telia.com
Address:  78.73.21.21
```

>nslookup 22.146.54.58

```
Serveur :   dns.google
Address:  8.8.8.8

*** dns.google ne parvient pas à trouver 22.146.54.58 : Non-existent domain
```

---
# IV. Wireshark



**On peut TOUT voir.**

🌞 Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :


>ping 192.168.137.1

![Ping](./img/ping%20passerelle.png)


**On redémarre le serveur et on s'y reconnecte**

>nc.exe -l -p 8888

>nc.exe 192.168.137.1 8888 

![Netcat](./img/netcat.png)

>nslookup google.com

![DNS](./img/dns.png)


## 2. Bonus : avant-goût TCP et UDP

🌞 **Wireshark it**

```
Adresse IP de connexion: 216.58.213.174
Port de connexion: 443
```

![Youtube](./img/video-youtube.png)

  
# Bilan

**Vu pendant le TP :**

- visualisation de vos interfaces réseau (en GUI et en CLI)
- extraction des informations IP
  - adresse IP et masque
  - calcul autour de IP : adresse de réseau, etc.
- connaissances autour de/aperçu de :
  - un outil de diagnostic simple : `ping`
  - un outil de scan réseau : `nmap`
  - un outil qui permet d'établir des connexions "simples" (on y reviendra) : `netcat`
  - un outil pour faire des requêtes DNS : `nslookup` ou `dig`
  - un outil d'analyse de trafic : `wireshark`
- manipulation simple de vos firewalls

**Conclusion :**

- Pour permettre à un ordinateur d'être connecté en réseau, il lui faut **une liaison physique** (par câble ou par *WiFi*).  
- Pour réceptionner ce lien physique, l'ordinateur a besoin d'**une carte réseau**. La carte réseau porte une adresse MAC  
- **Pour être membre d'un réseau particulier, une carte réseau peut porter une adresse IP.**
Si deux ordinateurs reliés physiquement possèdent une adresse IP dans le même réseau, alors ils peuvent communiquer.  
- **Un ordintateur qui possède plusieurs cartes réseau** peut réceptionner du trafic sur l'une d'entre elles, et le balancer sur l'autre, servant ainsi de "pivot". Cet ordinateur **est appelé routeur**.
- Il existe dans la plupart des réseaux, certains équipements ayant un rôle particulier :
  - un équipement appelé *passerelle*. C'est un routeur, et il nous permet de sortir du réseau actuel, pour en joindre un autre, comme Internet par exemple
  - un équipement qui agit comme **serveur DNS** : il nous permet de connaître les IP derrière des noms de domaine
  - un équipement qui agit comme **serveur DHCP** : il donne automatiquement des IP aux clients qui rejoigne le réseau
  - **chez vous, c'est votre Box qui fait les trois :)**