# Dynamic Routing — Partie 1 : Floating Static Route

Première partie d'une série sur le routage dynamique (suivront RIP & EIGRP, puis OSPF). Ce lab se concentre sur les routes statiques flottantes, utilisées comme secours à un protocole dynamique.

## Le problème
Une route statique reste dans la table de routage même si le lien qu'elle emprunte tombe, et peut continuer à envoyer des paquets vers un chemin mort. Le routage dynamique règle ça : chaque routeur annonce ses routes à ses voisins et les retire automatiquement quand elles ne sont plus valables. En cas de coupure d'un lien, si une boucle de secours existe, les routeurs redirigent le trafic vers un chemin encore actif.

## IGP et EGP
Deux catégories de protocoles :
- **IGP** (Interior Gateway Protocol), utilisé à l'intérieur d'une même structure.
- **EGP** (Exterior Gateway Protocol), utilisé pour relier deux structures différentes.

Exemple : une entreprise utilise un IGP en interne, un EGP pour se connecter à son FAI, le FAI utilise un IGP en interne, etc.

## Distance Vector et Link State
Deux familles d'algorithmes IGP :
- **Distance Vector** (RIP, EIGRP) : chaque routeur envoie à ses voisins directs la liste des réseaux qu'il connaît et leur métrique (« routing by rumor »). Un routeur ne connaît que la distance et la direction vers un réseau, rien sur la topologie complète.
- **Link State** (OSPF, IS-IS) : chaque routeur partage ses informations pour que tous construisent la même carte du réseau, et calcule lui-même le meilleur chemin. Ça réagit plus vite en cas de panne, mais ça demande plus de CPU.

Quand deux chemins ont exactement la même métrique, ECMP (Equal Cost Multi-Path) permet de les utiliser en même temps, contrairement à STP qui n'en garde qu'un.

## Metric et Administrative Distance (AD)
Chaque protocole calcule sa métrique différemment (hop count pour RIP, bande passante et délai pour EIGRP, coût pour OSPF et IS-IS), donc les métriques de deux protocoles différents ne sont pas comparables entre elles.

L'AD sert d'intermédiaire : elle traduit la métrique de chaque protocole en un score commun, et c'est l'AD la plus basse qui est retenue quand plusieurs protocoles proposent une route vers le même préfixe. Une AD de 255 signifie que la route est considérée invalide et n'est pas installée. Ce tableau de valeurs peut être modifié pour favoriser un protocole par rapport à un autre.

Ordre de choix d'une route : longest prefix match d'abord, puis l'AD la plus basse, puis la métrique la plus basse (l'AD ne départage que des routes vers le même préfixe).

## Floating Static Route
On peut fixer manuellement l'AD d'une route statique en la précisant à la fin de la commande `ip route`. En lui donnant une AD plus élevée que celle du protocole dynamique en place, la route statique n'est jamais utilisée tant que le protocole dynamique fonctionne, elle devient un secours automatique en cas de panne.

## Lab
Lab Packet Tracer basé sur la playlist Jeremy's IT Lab.

Enterprise A (R1, R2) route en OSPF vers son réseau interne (10.0.1.0/24 avec PC1, 10.0.2.0/24 avec SRV1) et vers Internet via deux FAI (ISPA et ISPB). PC1 et SRV1 ne sont normalement pas sur le même routeur, donc leur communication passe par le lien direct entre R1 et R2.

Ce que j'ai configuré :
- vérification des routes utilisées par PC1 vers SRV1 (lien direct R1-R2) et vers un serveur externe (route par défaut vers le FAI)
- une route statique flottante sur R1 et R2 pour joindre le réseau distant en cas de panne du lien R1-R2, avec une AD supérieure à celle d'OSPF pour qu'elle reste inactive tant qu'OSPF fonctionne
- coupure de l'interface reliant R1 et R2 pour vérifier que la route statique flottante prend le relais, confirmée par un ping de PC1 vers SRV1

![Topologie](topologie.png)
