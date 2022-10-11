# TP3 : On va router des trucs

Au menu de ce TP, on va revoir un peu ARP et IP histoire de **se mettre en jambes dans un environnement avec des VMs**.

Puis on mettra en place **un routage simple, pour permettre à deux LANs de communiquer**.

![Reboot the router](./pics/reboot.jpeg)

## Sommaire

- [TP3 : On va router des trucs](#tp3--on-va-router-des-trucs)
  - [Sommaire](#sommaire)
  - [0. Prérequis](#0-prérequis)
  - [I. ARP](#i-arp)
    - [1. Echange ARP](#1-echange-arp)
    - [2. Analyse de trames](#2-analyse-de-trames)
  - [II. Routage](#ii-routage)
    - [1. Mise en place du routage](#1-mise-en-place-du-routage)
    - [2. Analyse de trames](#2-analyse-de-trames-1)
    - [3. Accès internet](#3-accès-internet)
  - [III. DHCP](#iii-dhcp)
    - [1. Mise en place du serveur DHCP](#1-mise-en-place-du-serveur-dhcp)
    - [2. Analyse de trames](#2-analyse-de-trames-2)

## 0. Prérequis

➜ Pour ce TP, on va se servir de VMs Rocky Linux. 1Go RAM c'est large large. Vous pouvez redescendre la mémoire vidéo aussi.  

➜ Vous aurez besoin de deux réseaux host-only dans VirtualBox :

- un premier réseau `10.3.1.0/24`
- le second `10.3.2.0/24`
- **vous devrez désactiver le DHCP de votre hyperviseur (VirtualBox) et définir les IPs de vos VMs de façon statique**

➜ Les firewalls de vos VMs doivent **toujours** être actifs (et donc correctement configurés).

➜ **Si vous voyez le p'tit pote 🦈 c'est qu'il y a un PCAP à produire et à mettre dans votre dépôt git de rendu.**

## I. ARP

Première partie simple, on va avoir besoin de 2 VMs.

| Machine  | `10.3.1.0/24` |
|----------|---------------|
| `john`   | `10.3.1.11`   |
| `marcel` | `10.3.1.12`   |

```schema
   john               marcel
  ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     │
  └─────┘    └───┘    └─────┘
```

> Référez-vous au [mémo Réseau Rocky](../../cours/memo/rocky_network.md) pour connaître les commandes nécessaire à la réalisation de cette partie.

### 1. Echange ARP

🌞**Générer des requêtes ARP**

- effectuer un `ping` d'une machine à l'autre
- observer les tables ARP des deux machines
- repérer l'adresse MAC de `john` dans la table ARP de `marcel` et vice-versa
- prouvez que l'info est correcte (que l'adresse MAC que vous voyez dans la table est bien celle de la machine correspondante)
  - une commande pour voir la MAC de `marcel` dans la table ARP de `john`
  - et une commande pour afficher la MAC de `marcel`, depuis `marcel`

>ping 10.3.1.12 depuis John(10.3.1.11)

```
7 packets transmitted, 7 received, 0% packet loss, time 6120ms
rtt min/avg/max/mdev = 0.447/0.675/0.894/0.183 ms
[mat@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.383 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.902 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.567 ms
64 bytes from 10.3.1.12: icmp_seq=4 ttl=64 time=0.637 ms
64 bytes from 10.3.1.12: icmp_seq=5 ttl=64 time=0.968 ms
64 bytes from 10.3.1.12: icmp_seq=6 ttl=64 time=0.454 ms
^C
--- 10.3.1.12 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5136ms
```

>ping 10.3.1.11 depuis Marcel(10.3.1.12)

```
[root@localhost ~]# ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.501 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.891 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.516 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=64 time=0.469 ms
^C
--- 10.3.1.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3112ms
```

>ip neigh show Depuis 10.3.1.11 John

```
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3d REACHABLE
10.3.1.12 dev enp0s8 lladdr 08:00:27:31:a0:e4 STALE
```

>ip neigh show Depuis 10.3.1.12 Marcel

```
10.3.1.11 dev enp0s8 lladdr 08:00:27:b1:c7:51 STALE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3d REACHABLE
```

>Adresse MAC John grâce à la table ARP de Marcel:

```08:00:27:b1:c7:51```

>Adresse MAC Marcel grâce à la table ARP de John:

```08:00:27:31:a0:e4```

>ip  a depuis Marcel(10.3.1.12) pour obtenir sa MAC

```
link/ether 08:00:27:31:a0:e4 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe31:a0e4/64 scope link
       valid_lft forever preferred_lft forever
```

```08:00:27:31:a0:e4```


### 2. Analyse de trames

🌞**Analyse de trames**

- utilisez la commande `tcpdump` pour réaliser une capture de trame
- videz vos tables ARP, sur les deux machines, puis effectuez un `ping`

>sudo ip neigh flush all depuis Marcel et John
>ip neigh show 
```
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3d REACHABLE
```

>sudo tcpdump -i enp0s8 -c 10 -w mon_fichier.pcap not port 22

>ping 10.3.1.11 depuis 10.3.1.12

🦈 **Capture réseau `tp3_arp.pcapng`** qui contient un ARP request et un ARP reply

Source ```./src/tp3_arp.pcapng```

> **Si vous ne savez pas comment récupérer votre fichier `.pcapng`** sur votre hôte afin de l'ouvrir dans Wireshark, et me le livrer en rendu, demandez-moi.

## II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |

> Je les appelés `marcel` et `john` PASKON EN A MAR des noms nuls en réseau 🌻

```schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘
```

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

> Cette étape est nécessaire car Rocky Linux c'est pas un OS dédié au routage par défaut. Ce n'est bien évidemment une opération qui n'est pas nécessaire sur un équipement routeur dédié comme du matériel Cisco.

>sudo firewall-cmd --list-all
>sudo firewall-cmd --get-active-zone

```
public
interfaces: enp0s3 enp0s8
```

>sudo firewall-cmd --add-masquerade --zone=public 
```success```

>sudo firewall-cmd --add-masquerade --zone=public --permanent

````success``

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- il faut ajouter une seule route des deux côtés
- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre

>sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enps08 depuis Marcel(10.3.2.12)
>sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enps08 depuis John(10.3.1.11)

>ping 10.3.2.12 depuis John(10.3.1.11)

```
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.815 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.04 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.963 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=0.832 ms
64 bytes from 10.3.2.12: icmp_seq=5 ttl=63 time=1.73 ms
^C
--- 10.3.2.12 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4058ms
```
>ping 10.3.1.11 depuis Marcel(10.3.2.12)

```
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=0.973 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=0.885 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=0.982 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=63 time=1.74 ms
^C
--- 10.3.1.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
```

![THE SIZE](./pics/thesize.png)

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

>sudo ip neigh flush all depuis Marcel et John et routeur 

>ping 10.3.2.12 depuis John(10.3.1.11)

```
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.43 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.901 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.684 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=0.985 ms
64 bytes from 10.3.2.12: icmp_seq=5 ttl=63 time=0.549 ms
64 bytes from 10.3.2.12: icmp_seq=6 ttl=63 time=1.14 ms
64 bytes from 10.3.2.12: icmp_seq=7 ttl=63 time=0.636 ms
^C
--- 10.3.2.12 ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6107ms
```

>ip neigh show depuis John

```
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3d DELAY
10.3.1.254 dev enp0s8 lladdr 08:00:27:34:bf:c8 STALE
```

>ip neigh show depuis Marcel

```
10.3.2.1 dev enp0s8 lladdr 0a:00:27:00:00:42 DELAY
10.3.2.254 dev enp0s8 lladdr 08:00:27:8c:72:c6 STALE
```

>ip neigh show depuis Routeur

```
10.3.2.12 dev enp0s8 lladdr 08:00:27:31:a0:e4 STALE
10.3.1.11 dev enp0s3 lladdr 08:00:27:b1:c7:51 STALE
```

```
Lors du ping de John à Marcel John a demandé à la gateway (10.3.1.254) si elle savait où était Marcel, le routeur lui a répondu qu'il était dans le réseau 10.3.2.0/24 de gateway (10.3.2.254) 
```

- videz les tables ARP des trois noeuds
- effectuez un `ping` de `john` vers `marcel`
- regardez les tables ARP des trois noeuds
- essayez de déduire un peu les échanges ARP qui ont eu lieu
- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange

Par exemple (copiez-collez ce tableau ce sera le plus simple) :

| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------  |----------------|----------------------------|
| 1     | Requête ARP | x         |`john``08:00:27:8c:72:c6`  | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP |x          |`Marcel``08:00:27:31:a0:e4`| x              | `john` `08:00:27:8c:72:c6` |
| 3     | Ping        | 10.3.2.254|`john` `08:00:27:8c:72:c6` | 10.3.2.12      |`Marcel``08:00:27:31:a0:e4` |
| 4     | Pong        | 10.3.2.12 |`Marcel``08:00:27:31:a0:e4`| 10.3.2.254     | `john` `08:00:27:8c:72:c6` |

> Vous pourriez, par curiosité, lancer la capture sur `john` aussi, pour voir l'échange qu'il a effectué de son côté.

🦈 **Capture réseau `tp3_routage_marcel.pcapng`**

Source ```./src/tp3_routage_marcel.pcapng```

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

>sudo ip route add default via 10.3.1.254 dev enp0s8 pour John (10.3.1.11)
>sudo ip route add default via 10.3.2.254 dev enp0s8 pour Marcel (10.3.2.12)

>ping 172.67.74.226 (ynov)

```
PING 172.67.74.226 (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226: icmp_seq=1 ttl=53 time=21.1 ms
64 bytes from 172.67.74.226: icmp_seq=2 ttl=53 time=21.4 ms
^C
--- 172.67.74.226 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
```

```
PING 172.67.74.226 (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226: icmp_seq=1 ttl=53 time=22.3 ms
64 bytes from 172.67.74.226: icmp_seq=2 ttl=53 time=22.8 ms
64 bytes from 172.67.74.226: icmp_seq=3 ttl=53 time=23.3 ms
64 bytes from 172.67.74.226: icmp_seq=4 ttl=53 time=23.3 ms
^C
--- 172.67.74.226 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
```



- ajoutez une carte NAT en 3ème inteface sur le `router` pour qu'il ait un accès internet
- ajoutez une route par défaut à `john` et `marcel`
  - vérifiez que vous avez accès internet avec un `ping`
  - le `ping` doit être vers une IP, PAS un nom de domaine
- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
  - puis avec un `ping` vers un nom de domaine

🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
- capturez le ping depuis `john` avec `tcpdump`
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

>sudo tcpdump -i enp0s8 -c 10 -w tp3_routage_internet.pcap not port 22 | ping 8.8.8.8

| ordre | type trame | IP source          | MAC source              | IP destination     | MAC destination |     |
|-------|------------|--------------------|-------------------------|----------------    |-----------------|-----|
| 1     | ping       | `john` `10.3.1.11` | `john` `AA:BB:CC:DD:EE` | `8.8.8.8`          | ?               |     |
| 2     | pong       | `8.8.8.8`          | ...                     | `john` `10.3.1.11` | ...             | ... |

🦈 **Capture réseau `tp3_routage_internet.pcapng`**

Source: ```./src/tp3_routage_internet.pcapng```

## III. DHCP

On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests.

| Machine  | `10.3.1.0/24`              | `10.3.2.0/24` |
|----------|----------------------------|---------------|
| `router` | `10.3.1.254`               | `10.3.2.254`  |
| `john`   | `10.3.1.11`                | no            |
| `bob`    | oui mais pas d'IP statique | no            |
| `marcel` | no                         | `10.3.2.12`   |

```schema
   john               router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └─┬─┘    └─────┘    └───┘    └─────┘
   john        │
  ┌─────┐      │
  │     │      │
  │     ├──────┘
  └─────┘
```

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `john`
- créer une machine `bob`
- faites lui récupérer une IP en DHCP à l'aide de votre serveur

> Il est possible d'utilise la commande `dhclient` pour forcer à la main, depuis la ligne de commande, la demande d'une IP en DHCP, ou renouveler complètement l'échange DHCP (voir `dhclient -h` puis call me et/ou Google si besoin d'aide).

🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
  - une route par défaut
  - un serveur DNS à utiliser
- récupérez de nouveau une IP en DHCP sur `bob` pour tester :
  - `marcel` doit avoir une IP
    - vérifier avec une commande qu'il a récupéré son IP
    - vérifier qu'il peut `ping` sa passerelle
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande
    - vérifier que la route fonctionne avec un `ping` vers une IP
  - il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne
    - vérifier un `ping` vers un nom de domaine

### 2. Analyse de trames

🌞**Analyse de trames**

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP
- demander une nouvelle IP afin de générer un échange DHCP
- exportez le fichier `.pcapng`

🦈 **Capture réseau `tp2_dhcp.pcapng`**