TP 2 - Exploration du réseau d'un point de vue client
======================
I. Exploration locale en solo
==========
1. Affichage d'informations sur la pile TCP/IP locale
-----------
Pour afficher la configuration et l'adressage de nos cartes réseaux on utilise la commande ipcconfig/all 


    Carte réseau sans fil Wi-Fi :
    nom : Killer Wireless-n/a/ac 1535 Wireless Network Adapter
    Adresse MAC : 9C-B6-D0-20-69-A9
    adresse IP : 10.33.3.13
    Broadcast : 10.33.3.255
    Network : 10.33.0.0

    Carte Ethernet Ethernet :
    Nom :  Killer E2500 Gigabit Ethernet Controller
    Adresse MAC : 30-9C-23-18-11-28
    Adresse ip : Aucune 
    Broadcast: Aucun 
    Network: Aucun

    Serveur DHCP 10.33.3.254
    Gateway : 10.33.3.253

Pour connaitre la gateway il suffit de faire un ipconfig /all 

pour afficher les informations réseau d'une carte réseau via l'interface graphique de windows.

Il faut aller dans le panneau de configuration dans l'onglet réseau et internet,
il faut ensuite aller dans l'onglet centre réseau et partage puis modifier les parametre de la carte là une nouvelle fenetre s'ouvre avec la liste des cartes réseaux du pc

pour accéder aux infos d'une carte réseaux il faut faire un clic droit dessus et choisir Statut  et dans cet nouvelle fenetre on à un onglet détails qui nous montrera la config réseau de la carte 
![img](https://github.com/StevenDias33/LaboR-seau/blob/master/TP%201/Tp-1.png)

l'adresse ip MAC et la gateway sont la meme que dans l'exercie en CLI ce qui est logique 

La gateway du réseaux ingésup sert à connecter le réseaux LAN ingésup aux réseau WAN et donc de pouvoir accéder à internet il permet aussi de sécuriser son accés et d'interdire certaines requete comme les requetes ICMP pour eviter les DOS et Ddos 

    Premiere adresse IP du réseau 10.33.0.1
    Derniere adresse IP du réseau 10.33.3.254

j'ai changé mon adresse ip en 10.33.3.14

Pour scanner les adresses ip disponnible j'ai utilisé l'outils advanced ip scanner mais on peut aussi utiliser nmap en ligne de commande 

donc avec le logiciel j'ai toutes les adresses ip utilisé sur le réseau et j'en conclue donc que je peux utiliser l'adresse 10.33.3.14

le logiciel utilise le protocole TCP et fait des requetes TCP pour scanner l'ensemble du réseau et ainsi obtenir les adresses ip utilisé sauf que par mesure de sécurité certaines adresses ne sont pas scannable donc le logiciel les mets en tant qu'inconnu donc il se peut toujours qu'elle soit utilisé 

Du coup j'ai modifié mon adresse ip et j'accede bien à internet 


ensuite il faut faire un nmap du réseau pour vérifier que l'adresse ip que l'on veut prendre n'est pas utilisé 
malheuresement le firewall du réseau bloque les requetes ping il faudra donc faire des requetes tcp pour faire scanner le réseau pour ce faire j'ai utilisé un logiciel appellé advanced ip scanner 

ensuite j'ai modifié mon adresse ip en 10.33.3.20 car elle était disponnible aprés étude du scan 

le DHCP est un protocole de configuration dynamiques des hotes 

Pour la seconde parti du TP je me suis connecté en filaire au pc de mon voisin et j'ai changer mon adresse ip en 192.168.0.1 et lui en 192.168.0.2 

aprés ajout de régle dans le firewall j'arrive à communiquer avec lui et à le ping 

    Envoi d’une requête 'Ping'  192.168.0.2 avec 32 octets de données :
    Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128
    Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128
    Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128
    Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128

    Statistiques Ping pour 192.168.0.2:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :


Pour l'exercice suivant mon voisin à servi de passerelle par défaut pour me donner internet pour cela il a juste fallu modifier des parametres de partages et j'ai eu accés à internet

ensuite on a utiliser netcat pour communiquer ensemble depuis le CMD pareil aucun soucis il a juste fallu re config les ip et c'était bon 

j'ai ensuite effectuer une capture de la tram lors de la comunication net cat le .pcap et trouvable sur mon git dans le dossier TP1
