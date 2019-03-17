
# B1 Réseau 2018 - TP6

# TP 6 - Une topologie qui ressemble un peu à quelque chose, enfin ? 
Au menu :
* on va enfin aborder le **routage dynamique**
  * parce que saisir des routes à la main c'est chiant !
  * *"chiant"* ça veut dire : long, fastidieux, **et donc, terrain propice à l'erreur humaine**
* on va utiliser le protocole de routage **OSPF**
  * de la façon la plus simple possible
* nous configurerons nous-mêmes **un routeur qui fait du NAT**
  * ou presque
  * pour que les clients accèdent à internet
* il y aura **un serveur DHCP dans le réseau des clients**
  * pour distribuer les infos nécessaires aux clients
  * IP, adresse de gateway
* il y aura **un réseau dédié aux serveurs**
  * un serveur a une IP fixe, pas de DHCP ici donc
  * un serveur au moins fera tourner un serveur web (ou un `netcat`)
* et un peu de DNS et de NTP
    * comme ça, tout sera fait maison : routage, adressage IP, DNS, DHCP, NAT, NTP ! Boom.
* **les clients pourront joindre le réseau de serveurs et internet**
* en bonus : 
  * approfondir OSPF
    * jouer avec authentification, coûts, loopback interfaces, etc.
  * utilisation de switch Cisco + mise en place de VLAN



### Adressage IP de chacune des machines

Machines | `10.6.100.0/30` | `10.6.100.4/30` | `10.6.100.8/30` | `10.6.100.12/30` | `10.6.101.0/30` | `10.6.201.0/24` | `10.6.202.0/24`
--- | --- | --- | --- | --- | --- | --- | --- 
`r1.tp6.b1` | `10.6.100.1` | `10.6.100.5` | - | - | - | - | `10.6.202.254`
`r2.tp6.b1` | `10.6.100.2` | - |  `10.6.100.9` | - | - | - | -
`r3.tp6.b1` | - | - | `10.6.100.10` | `10.6.100.14` | `10.6.101.1` | - | -
`r4.tp6.b1` | - |  `10.6.100.6` | - | `10.6.100.13` | - | - | -
`r5.tp6.b1` | - | - | - | - |  `10.6.101.2` |  `10.6.201.254` | -
`client1.tp6.b1` | - | - | - | - | - |  `DHCP` | -
`client2.tp6.b1` | - | - | - | - | - |  `DHCP` | -
`server1.tp6.b1` | - | - | - | - | - | - | `DHCP`

Serveur 1 qui ping le client 1 


    [root@serveur1 ~]# ping client1
    PING client1 (10.6.201.42) 56(84) bytes of data.
    64 bytes from client1 (10.6.201.42): icmp_seq=1 ttl=60 time=61.4 ms
    64 bytes from client1 (10.6.201.42): icmp_seq=2 ttl=60 time=88.2 ms
    64 bytes from client1 (10.6.201.42): icmp_seq=3 ttl=60 time=48.1 ms

le serveur 1 qui ping le client 2

    [root@serveur1 ~]# ping client2
    PING client2 (10.6.201.18) 56(84) bytes of data.
    64 bytes from client2 (10.6.201.18): icmp_seq=1 ttl=60 time=70.2 ms
    64 bytes from client2 (10.6.201.18): icmp_seq=2 ttl=60 time=68.1 ms


client 2 vers client 1

    [root@client2 etc]# ping client1
    PING client1 (10.6.201.42) 56(84) bytes of data.
    64 bytes from client1 (10.6.201.42): icmp_seq=1 ttl=64 time=1.33 ms
    64 bytes from client1 (10.6.201.42): icmp_seq=2 ttl=64 time=1.25 ms
    64 bytes from client1 (10.6.201.42): icmp_seq=3 ttl=64 time=0.458 ms


Qui porte quel service ? Pour qui est ce service ? Pourquoie ?

Service | Qui porte le service ? | Pour qui ? | Pourquoi ? 
--- | --- | --- | ---
**NAT** | `r4.tp6.b1` | tout le monde (routeurs & VMs) | Le NAT permet d'accéder à l'extérieur, il permet de sortir du LAN. Toutes les machines peuvent en avoir besoin dans notre petite infra
**Serveur Web** | `server1.tp6.b1` | réseau client `10.6.201.0/24` | Le serveur Web symbolise un service d'infra en interne. Dispo pour nos clients. Plus de détails dans la section dédiée.
**DHCP** | `client2.tp6.b1` | réseau client `10.6.201.0/24` | Le DHCP (qui permet d'attribuer des IPs automatiquement) c'est pour des clients. Pas pour des serveurs. Un serveur, on veut qu'il ait une IP fixe. 
**DNS** | `server1.tp6.b1` | tout le monde (routeurs & VMs) | Le DNS nous permettra de résoudre les noms de domaines en local et nous passer du fichier `/etc/hosts`
**NTP** | `server1.tp6.b1` | réseau serveur `10.6.202.0/24` | Le NTP, qui permet la synchronisation de l'heure, est souvent indispensable pourdes serveurs mais totalement négligeable pour des clients (genre vos PCs, s'ils sont pas à l'heure, tout le monde s'en fout)

## 1. NAT : accès internet


j'ai internet sur r1 

    router1#telnet trip-hop.net 80
    Translating "trip-hop.net"...domain server (8.8.8.8) [OK]
    Trying trip-hop.net (213.186.33.4, 80)... Open
    GET/
    HTTP/1.1 400 Bad Request
    Server: squid
    Mime-Version: 1.0
    Date: Tue, 05 Mar 2019 16:12:35 GMT
    Content-Type: text/html;charset=utf-8
    Content-Length: 4104
    X-Squid-Error: ERR_INVALID_REQ 0
    Vary: Accept-Language
    Content-Language: fr
    X-Cache: MISS from PF1-BOR1FR
    X-Cache-Lookup: NONE from PF1-BOR1FR:3128
    Connection: close

et on vérifie la connexion sur les Vm mais attention on ne peut pas faire de curl car on n'a pas de service DNS donc on fait un dig avec le dns de google 

    [root@client1 etc]# dig google.com @8.8.8.8
    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             287     IN      A       216.58.213.174

    ;; Query time: 144 msec
    ;; SERVER: 8.8.8.8#53(8.8.8.8)
    ;; WHEN: Tue Mar 05 15:45:30 CET 2019
    ;; MSG SIZE  rcvd: 55


## 2. Serveur Web

je fais un curl avec le client 1 de mon serveur web pour savoir si il est up et si il est accessible 

    [root@client1 ~]$ curl server1
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
        <head>
            <title>Test Page for the Nginx HTTP Server on Fedora</title>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
            <style type="text/css">
            

## 3. Serveur DHCP

J'ai décidé de configuré 2 routeur pour faire le DHCP de mes clients et de mon serveur mais toutefois j'ai fais en sortes que mon serveur ait toujours la meme adresse ip


 pour éviter les problemes avec le dns ou ce genre de chose 


liste des commandes 

    no ip dhcp conflict logging

    ip dhcp excluded-address (ce que je veux pour ce qui concerne le serveur j'ai tout exclu sauf une seul adresse ip pour faire en sorte qu'il ait tout le temps la meme adresse )

    ip dhcp pool name

je vérifie ensuite si ce que j'ai fait c'est bien 


    [root@client1 ~]$ sudo dhclient -v ens33
    Internet Systems Consortium DHCP Client 4.2.5
    Copyright 2004-2013 Internet Systems Consortium.
    All rights reserved.
    For info, please visit https://www.isc.org/software/dhcp/

    Listening on LPF/ens33/08:00:27:69:75:ef
    Sending on   LPF/ens33/08:00:27:69:75:ef
    Sending on   Socket/fallback
    DHCPDISCOVER on ens33 to 255.255.255.255 port 67 interval 5 (xid=0x522506a8)
    DHCPREQUEST on ens333 to 255.255.255.255 port 67 (xid=0x522506a8)
    DHCPOFFER from 10.6.201.254
    DHCPACK from 10.6.201.254 (xid=0x522506a8)
    bound to 10.6.201.42 -- renewal in 291 seconds.

## 4. Serveur DNS

je vérifie si je peux curl google sans l'aide d'un autre dns que le miens 

    [root@client1 ~]curl google.com
    <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
    <TITLE>301 Moved</TITLE></HEAD><BODY>
    <H1>301 Moved</H1>
    The document has moved
    <A HREF="http://www.google.com/">here</A>.
    </BODY></HTML>



## 4. Serveur NTP

je vérifie que ça marche sur mon client 1 

    [root@client1 ~]$ chronyc sources
    210 Number of sources = 1
    MS Name/IP address         Stratum Poll Reach LastRx Last sample
    ===============================================================================
    ^* server1                       3   7   377    94    -11ms[  -15ms] +/-   70ms
    [root@client1 ~]$ chronyc tracking
    Reference ID    : 0A06CA0A (server1)
    Stratum         : 4
    Ref time (UTC)  : Tue Mar 12 23:38:03 2019
    System time     : 0.000877020 seconds fast of NTP time
    Last offset     : +0.000998453 seconds
    RMS offset      : 0.227384850 seconds
    Frequency       : 7.671 ppm fast
    Residual freq   : +0.112 ppm
    Skew            : 19.082 ppm
    Root delay      : 0.104015709 seconds
    Root dispersion : 0.008994878 seconds
    Update interval : 114.2 seconds
    Leap status     : Normal

# Bilan

* on a des **clients** 
  * configurés automatiquement dès qu'on les ajoute grâce au serveur DHCP
    * IP 
    * passerelle
    * serveur DNS
  * peuvent joindre internet sans pb avec un navigateur web
  * peuvent joindre le serveur web (NGINX) en interne
* on a des **serveurs**
  * ils sont synchros au niveau de l'heure grâce à NTP
  * ils sont en charge de la résolution de nom locale avec le serveur DNS
  * ils fournissent des services d'infra (symbolisé par notre serveur web NGINX)
* on a des **routeurs**
  * qui mettent en place un routage dynamique (OSPF)
  * qui fournissent un accès WAN (Internet) grâce au NAT


# Aller plus loin

# SSH 
pour la sécurisation du SSH c'est tout simple 

on crée un dossier .ssh sur le serveur

on y genere une clé ssh 

        ssh-keygen  -t  dsa -b 2048 -f  ~/.ssh/id_server 


on a 2 clé une pour le serveur et une pour les machines client 

on copie la clé client avec la commande scp sur une machine client et ensuite on vérifier que le fichier AuthorizedKeysFile à bien été crée avec la clé que l'on vient de générer 