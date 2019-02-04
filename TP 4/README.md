# B1 Réseau 2018 - TP4



# TP 4 - Spéléologie réseau : descente dans les couches

# Sommaire
* [Préparation d'une VM "patron"](#préparation-dune-vm-patron)
* I. [Mise en place du lab](#i-mise-en-place-du-lab)
* II. [Spéléologie Réseau](#ii-spéléologie-réseau)
  * 1. [ARP](#1-arp)
  * 2. [Interception de trafic avec Wireshark](#2-wireshark)
    * [ARP et `ping`](#a-interception-darp-et-ping)
    * [netcat](#b-interception-dune-communication-netcat)
    * [Trafic Web (HTTP)](#c-interception-dun-trafic-http)
* [Annexe 1 : Installation d'une interface graphique sur CentOS 7](#annexe-1--installation-dun-client-graphique)

--- 
# Préparation d'une VM "patron"
Bon c'est rigolo d'installer CentOS, mais c'est vite chiant. Nos hyperviseurs permettent de cloner des machines déjà existantes. Sauf que vos machines précédentes, vous les avez bien pourries !  

Vous allez réaliser **une nouvelle installation de CentOS**, configurer le minimum, et l'éteindre. **Vous ne rallumerez plus jamais cette VM**, elle ne servira qu'à être clonée. Cela accélerera grandement la mise en place de nos TPs ! (ce ne sont que des choses qu'on a déjà fait au [TP précédent](../3/README.md#i-création-et-utilisation-simples-dune-vm-centos))

**Installation et configuration de la VM "patron"** :

installation de la VM patron comme éxigé par la sujet aucun problememe rencontré grace au clonage de VMWARE et sa config réseau plus avancé que virtual box 

## 1. Création des réseaux
on va avoir 2 réseau différent: 

le 10.2.0.0 et le 10.1.0.0 avec un routeur qui servira de passerelle dans les deux sens 

## 2. Création des VMs

Simmple clonnage à partir de VMWare en attribuant les bonnes cartes réseaux 
```
client  <--net1--> router <--net2--> server
```

---

**Checklist (à faire sur toutes les machines)** :
* [X] Désactiver SELinux
  * déja fait dans le patron
* [X] Installation de certains paquets réseau
  * déja fait dans le patron
* [X] **Désactivation de la carte NAT**
  * déja fait dans le patron
* [x] [Définition des IPs statiques](../../cours/procedures.md#définir-une-ip-statique)
* [x] La connexion SSH doit être fonctionnelle
  * une fois fait, vous avez vos trois fenêtres SSH ouvertes, une dans chaque machine
* [x] [Définition du nom de domaine](../../cours/procedures.md#changer-son-nom-de-domaine)
* [x] [Remplissage du fichier `/etc/hosts`](../../cours/procedures.md#editer-le-fichier-hosts)
* [x] `client1` ping `router1.tp4` sur l'IP `10.1.0.254`
* [x] `server1` ping `router1.tp4` sur l'IP `10.2.0.254`

---

**Petit tableau récapitulatif** :

Machine | `net1` | `net2`
--- | --- | ---
`client1.tp4` | `10.1.0.10` | X
`router1.tp4` | `10.1.0.254` | `10.2.0.254`
`server1.tp4` | X | `10.2.0.10` 

## 3. Mise en place du routage statique

Config route client 1 :

10.2.0.254 via 10.1.0.254


10.2.0.10 via 10.1.0.254



Config route client serveur 1 :

10.1.0.254 via 10.2.0.254


10.1.0.10 via 10.2.0.254




# II. Spéléologie réseau


## 1. ARP

ARP est le protocole qui permet de connaître la MAC d'une machine quand on connaît son IP.  



on voit la table ARP se remplir au fur et à mesure que les VM comuniquent entre elle tant qu'il n'y a pas de communication la table ARP reste vide 



## 2. Wireshark

Wireshark permet d'analyser et de snifer les trames qui sont échanger dans le réseau pour cet exercice l'objectif était de comprendre ce qui s'est passé lorsque l'on a fait nos requete ARP etc etc 


## A. Interception d'ARP et `ping`

interception basique d'une requete arp et d'un ping on lance l'analyser et on lance un ping et une requeter ARP wireshark s'occupe de les récupérer on a plus qu'a les analayser 

## B. Interception d'une communication `netcat`

pareil qu'au dessus mais avec du netcat 
## C. Interception d'un trafic HTTP (BONUS)

meme principe que le reste et cela nous permet de voir comment marche les comms en HTTP avec un serveur web 
### Install et config du serveur Web

