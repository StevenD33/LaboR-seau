# TP5-B1A-Reseau

## I. Préparation du lab

### 1. Préparation VMs

4. Topologie et tableau récapitulatif


Machine | net 1 | net 2 | net 12
--- | --- | --- | ---
`client1.tp5` | x | 10.5.2.10 | x
`client2.tp5` | x | 10.5.2.11 | x
`router1.tp5` | 10.5.1.254 | x | 10.5.12.1
`router2.tp5` | x | 10.5.2.254 | 10.5.12.2
`server1.tp5` | 10.5.1.10 | x | x 

### II. Lancement et configuration du lab  
+ **Checklist IP VMs**  

Pour un gain de temps j'ai directement configurér mon labo en DHCP car pourquoi se prendre la tête si on peut aller plus vite.


+ **Checklist IP Routeurs**  

Pareil j'ai config le DHCP donc du coup j'ai juste vérif avec un show running config pour voir si tout marcher bien 

+ **Checklist routes**

Ici j'ai quand meme ajouter des routes à la main au cas ou 
donc pour les routes sur NET 1 on à 

10.5.2.0 via 10.5.2.254 


10.5.1.0 via 10.5.1.254

10.5.12.2 via 10.5.12.1


Net 2 

10.5.2.0 via 10.5.2.254 


10.5.1.0 via 10.5.2.254


10.5.1.0 via 10.5.1.254


10.5.12.1 via 10.5.12.2


J'ai fait de l'overkill de route par ce que parfois j'avais des truc incompréhensible qui faisait que je ne pinguer pas donc j'ai préféré overkiller les routes et avoir un taux de fonctionnement à 100% 


### III. DHCP  
#### 1. Mise en place du serveur DHCP  

On crée une nouvelle machine ou on peut meme utiliser un routeur qui est déja sur le réseau c'est ce que j'ai fais par ce que pourquoi faire compliquer quand on peut faire simple ? 

Pour config le DHCP sur un routeur cisco il faut faire quelques manip 

pour ce faire on active  le service dhcp sur le routeur en mode config 

    service dhcp

ensuite vu que cisco c'est un peu relou il faut config un DHCP database agent ou il faut désactiver le DHCP conflict logging pour faire au plus simple j'ai désactiver le DHCP conflict logging 

    no ip dhcp conflict logging

Ensuite on assigne les addresse que l'on ne veut pas que le serveur DHCP attribue 

    ip dhcp excluded-address 10.5.2.100

ensuite on config une plage d'addresse que notre serveur dhcp attribuera pour ce faire un crée un pool d'adresse 

    ip dhcp pool name 


ensuite on le configure en entrant en mode config dessus 

et là on peut choisir un pool d'adresse par exemple 10.5.2.4/24 -> 10.5.2.15 /24

ensuite si on veut faire les choses bien on peut configurer un bail d'expiration pour libérer des adresses lorsqu'elle ne sont plus utilisé 

voici la commande à utiliser pour ce faire 
    lease {days [hours][minutes] | infinite}

aprés avec les routeurs cisco on peut aller encore plus loin pour la configuration d'un serveur dhcp sur un routeur mais pour ce TP je suis rester soft.
#### 2. Explorer un peu DHCP  

Analyse de DORA avec tcpdump 

qu'est ce DORA ? 

DORA c'est en gros pour faire simple l'échange qui est réaliser entre un pc et un serveur DHCP pour obtenir une adresse ip c'est un acronyme 

* DHCP Discover
* DHCP Offer
* DHCP Request 
* DHCP Acknowledge 

#### BOnus 

pour la sécurisation du SSH c'est tout simple 

on crée un dossier .ssh 

on y genere une clé ssh 

        ssh-keygen  -t  dsa -b 2048 -f  ~/.ssh/id_server 


on a 2 clé une pour le serveur et une pour les machines client 

on copie la clé client avec la commande scp sur une machine client et ensuite on vérifier que le fichier AuthorizedKeysFile à bien été crée avec la clé que l'on vient de générer 

si tout va bien on vérifier que l'on arrive à se connecter en ssh au serveur 

Pour le NAT je l'ai fais fonctionner vite fais mais c'est pas encore au point pour être rajouté dans le compte rendus 

