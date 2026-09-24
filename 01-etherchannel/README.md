# EtherChannel

## Le problème
Brancher plusieurs liens entre deux switchs n'augmente pas la bande passante : le Spanning Tree est actif par défaut et bloque les liens en trop pour éviter les boucles. EtherChannel règle ça en regroupant plusieurs liens physiques en un seul lien logique. STP voit alors ce groupe comme un seul lien et fonctionne normalement. Il n'est pas activé automatiquement, il faut le configurer.

## PAgP et LACP
Deux protocoles permettent de négocier un EtherChannel :

- **PAgP** : propriétaire Cisco, limité à 8 liens.
- **LACP** : standard IEEE (802.3ad, puis 802.1AX), jusqu'à 16 liens dont 8 actifs et 8 en réserve.

À part cette limite de liens, le fonctionnement est le même.

## Modes de négociation
| Protocole | Modes | Résultat |
|---|---|---|
| PAgP | Desirable / Auto | Desirable+Desirable ou Desirable+Auto forment un EtherChannel, Auto+Auto non |
| LACP | Active / Passive | Active+Active ou Active+Passive fonctionnent, Passive+Passive non |
| Aucun | On | Force l'EtherChannel sans négociation ni vérification de cohérence : risqué |

## Load-balancing
La répartition se fait par flux, c'est-à-dire par communication entre deux nodes (par exemple un serveur et un PC). Pour chaque trame, le switch calcule un hash à partir de l'adresse source, de l'adresse destination, ou des deux (en MAC ou en IP), et ce résultat désigne le lien à utiliser. Un même flux passe donc toujours par le même lien. Le critère par défaut dépend du modèle, ici src-mac.

## Contraintes
Chaque interface du groupe doit avoir la même configuration : duplex, vitesse, mode switchport (access ou trunk), et VLANs autorisés et VLAN natif pour un trunk. Si un seul paramètre diffère, l'interface n'est pas incluse dans l'EtherChannel.

## Lab
Lab Packet Tracer basé sur la playlist Jeremy's IT Lab.

Deux réseaux d'accès (PC1/PC2 derrière ASW1, SRV1 derrière ASW2) sont reliés chacun à un switch de distribution par deux liens, et les deux switchs de distribution (DSW1 et DSW2) sont reliés entre eux par deux liens sur un réseau 10.0.0.0/30.

Ce que j'ai configuré :
- un EtherChannel de niveau 2 en LACP entre ASW1 et DSW1, en trunk
- un EtherChannel de niveau 2 en PAgP entre ASW2 et DSW2, en trunk
- un EtherChannel de niveau 3 statique (mode On) entre DSW1 et DSW2, avec le routage nécessaire pour que les PC joignent SRV1
- une vérification de la méthode de load-balancing par défaut (src-mac sur chaque switch), puis un passage à src-dst-ip sur DSW1 et DSW2

Sur le lien de niveau 3 entre les deux DSW, toutes les trames ont la même MAC source et la même MAC destination (celles des deux switchs). Avec src-mac, le hash donne donc toujours le même résultat et un seul lien est utilisé. Avec src-dst-ip, le hash dépend des adresses IP des machines qui communiquent, ce qui permet de répartir plusieurs flux sur les deux liens.

![Topologie](01-etherchannel/img/topologie.PNG)


