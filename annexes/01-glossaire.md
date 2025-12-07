🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Glossaire des termes réseau

## Introduction

Ce glossaire regroupe les termes techniques essentiels rencontrés dans la formation TCP/IP. Les définitions sont organisées alphabétiquement et incluent des références croisées vers des concepts connexes.

**Convention de notation :**
- Les termes connexes sont indiqués en *italique*
- Les acronymes sont suivis de leur développement complet
- Les numéros de RFC pertinents sont mentionnés entre parenthèses

---

## A

**ACK (Acknowledgement)**
Signal d'acquittement dans le protocole *TCP* confirmant la bonne réception de données. Le flag ACK dans l'en-tête TCP indique que le champ numéro d'acquittement est valide.

**Adresse IP (Internet Protocol Address)**
Identifiant numérique unique attribué à chaque interface réseau. En *IPv4*, elle est codée sur 32 bits (ex: 192.168.1.10). En *IPv6*, elle est codée sur 128 bits (ex: 2001:0db8:85a3::8a2e:0370:7334).

**Adresse MAC (Media Access Control)**
Identifiant physique unique de 48 bits attribué à chaque carte réseau par le fabricant. Format hexadécimal séparé par des deux-points ou tirets (ex: 00:1A:2B:3C:4D:5E). Également appelée adresse physique ou adresse Ethernet.

**AH (Authentication Header)**
Protocole *IPsec* assurant l'authentification et l'intégrité des paquets IP, mais sans chiffrement. Voir aussi *ESP*.

**API (Application Programming Interface)**
Interface de programmation permettant aux applications d'interagir avec les protocoles réseau. L'*API Socket* est l'interface standard pour la programmation réseau.

**ARP (Address Resolution Protocol)**
Protocole de résolution d'adresse permettant de découvrir l'*adresse MAC* correspondant à une *adresse IP* donnée sur un réseau local. Fonctionne par diffusion (*broadcast*) de requêtes.

**AS (Autonomous System)**
Ensemble de réseaux IP sous une administration commune partageant une politique de routage unique. Identifié par un numéro ASN. Concept fondamental pour *BGP*.

**ASCII (American Standard Code for Information Interchange)**
Encodage de caractères sur 7 bits utilisé dans de nombreux protocoles applicatifs (*SMTP*, *FTP*, *HTTP*).

---

## B

**Backbone**
Dorsale d'un réseau, infrastructure centrale à haute capacité interconnectant les différentes parties du réseau. Sur Internet, les backbones sont gérés par les grands opérateurs (*Tier 1*).

**Bande passante (Bandwidth)**
Capacité maximale de transmission d'un canal de communication, exprimée en bits par seconde (bps). Ne pas confondre avec le *débit* effectif qui peut être inférieur.

**BGP (Border Gateway Protocol)**
Protocole de routage inter-domaines utilisé pour échanger les informations de routage entre *systèmes autonomes* sur Internet. C'est le protocole qui fait fonctionner l'Internet global.

**Bit**
Unité élémentaire d'information binaire (0 ou 1). Les débits réseau s'expriment en bits par seconde (bps, Kbps, Mbps, Gbps).

**Broadcast**
Diffusion d'un message à tous les hôtes d'un réseau local. En IPv4, l'adresse de broadcast d'un réseau se termine par tous les bits à 1 (ex: 192.168.1.255 pour le réseau 192.168.1.0/24).

**Buffer**
Zone mémoire temporaire pour stocker des données en attente de traitement. Les buffers réseau permettent de gérer les différences de vitesse entre émetteur et récepteur.

---

## C

**Cache**
Mémoire temporaire stockant des données fréquemment accédées pour améliorer les performances. Exemples : *cache DNS*, cache ARP, cache web.

**CDN (Content Delivery Network)**
Réseau de distribution de contenu constitué de serveurs répartis géographiquement pour rapprocher les ressources des utilisateurs finaux et améliorer les performances.

**Checksum**
Somme de contrôle calculée sur un ensemble de données pour détecter les erreurs de transmission. Utilisée dans les en-têtes *IP*, *TCP*, *UDP*, *ICMP*.

**CIDR (Classless Inter-Domain Routing)**
Méthode d'allocation d'adresses IP et de routage qui remplace le système de classes. Notation avec masque de sous-réseau (ex: 192.168.1.0/24).

**Circuit Breaker**
Patron de conception logiciel empêchant les appels répétés à un service défaillant, laissant le temps au système de récupérer.

**Classe d'adresse IP**
Ancien système de division des adresses IPv4 en classes A, B, C, D et E selon les premiers bits. Remplacé par *CIDR* mais encore utilisé dans certains contextes pédagogiques.

**Client**
Dans un modèle client-serveur, programme initiant une connexion pour demander des services. Opposé au *serveur*.

**Commutation (Switching)**
Technique de transfert de *trames* basée sur les *adresses MAC* au niveau de la couche 2. Les commutateurs (*switches*) créent des domaines de collision séparés.

**Congestion**
Saturation d'un réseau lorsque la demande dépasse la capacité disponible. TCP implémente des mécanismes de *contrôle de congestion*.

**Connexion**
Association logique entre deux points de communication. TCP établit des connexions orientées connexion via le *three-way handshake*.

**Couche (Layer)**
Niveau d'abstraction dans un modèle en couches (*OSI* ou *TCP/IP*). Chaque couche offre des services à la couche supérieure et utilise les services de la couche inférieure.

---

## D

**Datagramme**
Unité de données autonome transmise sur un réseau sans garantie de livraison. Utilisé notamment par *UDP* et au niveau de la *couche Internet*.

**Débit (Throughput)**
Quantité de données effectivement transférées par unité de temps. Peut être inférieur à la *bande passante* théorique en raison des protocoles, de la congestion, etc.

**Décapsulation**
Processus d'extraction des données utiles en retirant successivement les en-têtes des différentes couches lors de la réception. Inverse de l'*encapsulation*.

**DHCP (Dynamic Host Configuration Protocol)**
Protocole permettant l'attribution automatique de configurations réseau (*adresse IP*, *masque*, *passerelle*, serveurs *DNS*) aux clients. Processus *DORA* : Discovery, Offer, Request, Acknowledgement.

**DNS (Domain Name System)**
Système hiérarchique et distribué de résolution de noms de domaine en *adresses IP*. Fonctionne comme l'annuaire d'Internet.

**Domaine de broadcast**
Ensemble des hôtes pouvant recevoir un message en *broadcast*. Délimité par les routeurs.

**Domaine de collision**
Segment de réseau où les trames peuvent entrer en collision. Les *commutateurs* segmentent les domaines de collision.

**DoS (Denial of Service)**
Attaque visant à rendre un service indisponible en le saturant de requêtes. Variante *DDoS* : attaque distribuée depuis plusieurs sources.

**DORA**
Séquence des quatre étapes du processus *DHCP* : Discovery, Offer, Request, Acknowledgement.

**Dual Stack**
Configuration réseau supportant simultanément *IPv4* et *IPv6* pour assurer la transition progressive.

---

## E

**Encapsulation**
Processus d'ajout d'en-têtes successifs aux données à chaque couche du modèle lors de l'émission. Inverse de la *décapsulation*.

**En-tête (Header)**
Métadonnées ajoutées au début d'une unité de données (paquet, segment, trame) contenant les informations de contrôle du protocole.

**ESP (Encapsulating Security Payload)**
Protocole *IPsec* fournissant confidentialité (chiffrement), authentification et intégrité. Plus complet que *AH*.

**Ethernet**
Technologie de réseau local la plus répandue, opérant principalement en couche 2 (*liaison de données*). Définit le format des *trames* et l'accès au média.

---

## F

**Fenêtre glissante (Sliding Window)**
Mécanisme TCP permettant l'envoi de plusieurs segments sans attendre d'acquittement pour chacun, optimisant ainsi le débit.

**Firewall (Pare-feu)**
Dispositif de sécurité filtrant le trafic réseau selon des règles définies. Peut opérer aux couches 3 (*filtrage de paquets*), 4 ou 7 (*inspection applicative*).

**Flag**
Indicateur binaire dans un en-tête de protocole. Exemples dans TCP : SYN, ACK, FIN, RST, PSH, URG.

**FTP (File Transfer Protocol)**
Protocole de transfert de fichiers utilisant deux connexions TCP : une pour les commandes (port 21) et une pour les données (port 20 en mode actif).

**Full-duplex**
Mode de communication permettant la transmission simultanée dans les deux sens. TCP et les réseaux Ethernet modernes sont full-duplex.

---

## G

**Passerelle (Gateway)**
Dispositif interconnectant des réseaux différents, généralement un routeur. La *passerelle par défaut* (default gateway) est le routeur vers lequel sont envoyés les paquets destinés à d'autres réseaux.

**gRPC**
Framework RPC moderne utilisant *HTTP/2* et *Protocol Buffers* pour des communications performantes entre microservices.

---

## H

**Handshake**
Échange initial entre deux parties pour établir les paramètres d'une communication. Exemples : *three-way handshake* TCP, handshake TLS.

**Header (En-tête)**
Voir *En-tête*.

**Hop**
Saut entre deux routeurs sur le chemin d'un paquet. Le champ *TTL* limite le nombre de hops.

**Hôte (Host)**
Tout dispositif connecté à un réseau avec une *adresse IP* : ordinateur, serveur, smartphone, imprimante réseau, etc.

**HTTP (HyperText Transfer Protocol)**
Protocole de la couche application pour le transfert de documents hypermédia. Versions : HTTP/1.1, *HTTP/2*, *HTTP/3*.

**HTTP/2**
Évolution majeure d'HTTP introduisant le multiplexage, la compression des en-têtes et le server push sur une connexion TCP unique.

**HTTP/3**
Dernière version d'HTTP utilisant *QUIC* (sur UDP) au lieu de TCP pour améliorer les performances et réduire la latence.

**HTTPS (HTTP Secure)**
HTTP sécurisé par *TLS/SSL*, chiffrant les communications entre client et serveur. Port standard : 443.

---

## I

**ICMP (Internet Control Message Protocol)**
Protocole de la couche Internet pour les messages de contrôle et d'erreur. Utilisé par *ping* et *traceroute*.

**IGMP (Internet Group Management Protocol)**
Protocole permettant aux hôtes de signaler leur appartenance à des groupes *multicast* aux routeurs.

**IMAP (Internet Message Access Protocol)**
Protocole de messagerie permettant la gestion des emails directement sur le serveur. Port 143 (993 en sécurisé). Plus évolué que *POP3*.

**Interface**
Point de connexion entre un dispositif et un réseau. Chaque interface possède une ou plusieurs *adresses IP*.

**Internet**
Réseau mondial interconnectant des millions de réseaux via le protocole *TCP/IP*.

**Intranet**
Réseau privé d'organisation utilisant les technologies Internet (*TCP/IP*, web, email).

**IoT (Internet of Things)**
Internet des objets, ensemble de dispositifs connectés (capteurs, actuateurs) communiquant via IP.

**IP (Internet Protocol)**
Protocole de la couche Internet responsable de l'adressage et du routage des paquets. Versions : *IPv4* et *IPv6*.

**IPsec (IP Security)**
Suite de protocoles assurant la sécurité des communications IP par authentification et chiffrement. Utilisé dans les *VPN*.

**IPv4 (Internet Protocol version 4)**
Version historique d'IP utilisant des adresses sur 32 bits (environ 4,3 milliards d'adresses). Format : xxx.xxx.xxx.xxx.

**IPv6 (Internet Protocol version 6)**
Nouvelle version d'IP avec des adresses sur 128 bits, résolvant la pénurie d'adresses IPv4. Format hexadécimal avec 8 groupes de 16 bits.

**ISP (Internet Service Provider)**
Fournisseur d'accès Internet offrant la connectivité aux particuliers et entreprises.

---

## J

**Jitter**
Variation de la latence dans les transmissions réseau. Critique pour les applications temps réel (VoIP, visioconférence).

**JSON (JavaScript Object Notation)**
Format léger d'échange de données textuelles, très utilisé dans les API web modernes.

---

## K

**Keep-alive**
Mécanisme maintenant une connexion active en envoyant périodiquement des messages. Présent dans TCP, HTTP/1.1, et de nombreux protocoles applicatifs.

---

## L

**LAN (Local Area Network)**
Réseau local couvrant une zone géographique limitée (bureau, bâtiment, campus).

**Latence (Latency)**
Temps de propagation d'un paquet d'un point à un autre. Mesurée en millisecondes (ms). Différente du *débit*.

**Load Balancing (Répartition de charge)**
Distribution du trafic entre plusieurs serveurs pour optimiser les ressources et améliorer la disponibilité. Peut s'effectuer aux couches 4 (TCP/UDP) ou 7 (application).

**Localhost**
Adresse de bouclage désignant la machine locale. En IPv4 : 127.0.0.1, en IPv6 : ::1.

**Long Polling**
Technique où le client maintient une requête HTTP ouverte jusqu'à ce que le serveur ait des données à envoyer. Alternative au *polling* classique.

---

## M

**MAC (Media Access Control)**
Voir *Adresse MAC*.

**MAN (Metropolitan Area Network)**
Réseau métropolitain couvrant une ville ou une agglomération.

**Masque de sous-réseau (Subnet Mask)**
Valeur indiquant quelle partie d'une *adresse IP* identifie le réseau et quelle partie identifie l'hôte. Exemple : 255.255.255.0 ou /24 en notation *CIDR*.

**Métrique**
Valeur numérique utilisée par les protocoles de routage pour comparer des routes. Peut représenter le nombre de *hops*, la bande passante, la latence, etc.

**MTU (Maximum Transmission Unit)**
Taille maximale d'un paquet pouvant être transmis sans fragmentation. Typiquement 1500 octets pour Ethernet.

**Multicast**
Transmission d'un même paquet à un groupe d'hôtes identifiés par une adresse multicast. Économise la bande passante comparé au *broadcast* ou *unicast* multiples.

**Multiplexage**
Partage d'une ressource de communication entre plusieurs flux. Exemples : multiplexage de ports en TCP/UDP, multiplexage de flux en *HTTP/2*.

---

## N

**NAT (Network Address Translation)**
Mécanisme permettant à plusieurs hôtes d'un réseau privé de partager une adresse IP publique. Variante *PAT* utilise aussi les ports.

**Netmask**
Voir *Masque de sous-réseau*.

**NTP (Network Time Protocol)**
Protocole de synchronisation d'horloge permettant de maintenir l'heure précise sur les systèmes connectés.

---

## O

**OSI (Open Systems Interconnection)**
Modèle de référence à 7 couches pour les communications réseau : Physique, Liaison, Réseau, Transport, Session, Présentation, Application.

**OSPF (Open Shortest Path First)**
Protocole de routage interne à état de lien, utilisant l'algorithme de Dijkstra pour calculer les routes optimales.

---

## P

**Paquet (Packet)**
Unité de données au niveau de la couche réseau/Internet contenant en-tête IP et données encapsulées.

**PAT (Port Address Translation)**
Extension de *NAT* utilisant les numéros de *port* source pour multiplexer plusieurs connexions sur une seule adresse IP publique.

**Payload**
Données utiles transportées par un protocole, excluant les en-têtes et métadonnées.

**PDU (Protocol Data Unit)**
Unité de données à chaque couche du modèle : bit (physique), trame (liaison), paquet (réseau), segment/datagramme (transport), données (application).

**Ping**
Utilitaire utilisant *ICMP* Echo Request/Reply pour tester la connectivité et mesurer le temps de réponse (RTT).

**PKI (Public Key Infrastructure)**
Infrastructure à clés publiques pour la gestion des certificats numériques. Fondation de *TLS/SSL*.

**Polling**
Technique où le client interroge régulièrement le serveur pour détecter de nouvelles données. Inefficace comparé au *long polling* ou *WebSocket*.

**POP3 (Post Office Protocol version 3)**
Protocole simple de récupération d'emails les téléchargeant depuis le serveur. Port 110 (995 en sécurisé).

**Port**
Numéro de 16 bits identifiant un processus ou service au niveau transport. Plage : 0-65535. Catégories : well-known (0-1023), registered (1024-49151), dynamic (49152-65535).

**PPP (Point-to-Point Protocol)**
Protocole de liaison point à point utilisé historiquement pour les connexions dial-up et encore en DSL.

**Protocol Buffers**
Format de sérialisation binaire développé par Google, utilisé notamment dans *gRPC*.

**Proxy**
Serveur intermédiaire relayant les requêtes entre clients et serveurs. Un *reverse proxy* protège et répartit la charge vers les serveurs backend.

---

## Q

**QoS (Quality of Service)**
Mécanismes de priorisation du trafic réseau pour garantir des performances minimales aux applications critiques.

**QUIC (Quick UDP Internet Connections)**
Protocole transport sur *UDP* développé par Google, combinant les avantages de TCP et TLS avec moins de latence. Base de *HTTP/3*.

---

## R

**RFC (Request for Comments)**
Document technique décrivant les standards Internet. Exemples : RFC 791 (IPv4), RFC 793 (TCP), RFC 2616 (HTTP/1.1).

**RIP (Routing Information Protocol)**
Protocole de routage à vecteur de distance utilisant le nombre de *hops* comme métrique. Limité à 15 hops.

**Round-Trip Time (RTT)**
Temps aller-retour d'un paquet entre deux points. Mesuré par *ping*. Important pour TCP qui ajuste ses timeouts selon le RTT.

**Routage (Routing)**
Processus de sélection du chemin optimal pour acheminer un paquet d'un réseau source vers un réseau destination.

**Routeur (Router)**
Équipement de couche 3 interconnectant des réseaux IP et prenant des décisions de routage basées sur les *adresses IP* de destination.

**RST (Reset)**
Flag TCP utilisé pour réinitialiser brutalement une connexion, généralement en cas d'erreur.

---

## S

**SDN (Software-Defined Networking)**
Architecture réseau séparant le plan de contrôle (décisions) du plan de données (forwarding), permettant une gestion programmable centralisée.

**Segment**
Unité de données TCP incluant l'en-tête TCP et les données applicatives.

**Serveur (Server)**
Programme attendant et répondant aux requêtes des *clients*. Écoute sur un *port* spécifique.

**Session**
Connexion logique maintenue entre deux applications. TCP gère les sessions au niveau transport.

**SFTP (SSH File Transfer Protocol)**
Protocole sécurisé de transfert de fichiers encapsulé dans *SSH*. Plus sécurisé que *FTP*.

**SMTP (Simple Mail Transfer Protocol)**
Protocole d'envoi d'emails entre serveurs et depuis clients. Port 25 (ou 587 avec STARTTLS).

**SNMP (Simple Network Management Protocol)**
Protocole de supervision et gestion des équipements réseau.

**Socket**
Point de terminaison d'une communication réseau, identifié par une adresse IP et un numéro de port. Interface de programmation réseau standard.

**Sous-réseau (Subnet)**
Division d'un réseau IP en réseaux plus petits pour optimiser l'utilisation des adresses et améliorer les performances.

**SSH (Secure Shell)**
Protocole sécurisé pour l'accès distant et l'exécution de commandes. Port 22. Remplace *Telnet*.

**SSL (Secure Sockets Layer)**
Ancien protocole de sécurisation (remplacé par *TLS*). Le terme SSL/TLS est souvent utilisé de manière interchangeable.

**Stateful**
Mécanisme conservant l'état des connexions. Un *firewall stateful* suit les connexions établies.

**Stateless**
Mécanisme ne conservant pas d'état entre les requêtes. *HTTP* et *UDP* sont stateless par nature.

**Subnetting**
Voir *Sous-réseau*.

**Switch (Commutateur)**
Équipement de couche 2 commutant les *trames* selon les *adresses MAC*. Les switches modernes peuvent avoir des fonctions de couche 3.

**SYN (Synchronize)**
Flag TCP utilisé pour initier une connexion (*three-way handshake*). Première étape : SYN, deuxième : SYN-ACK, troisième : ACK.

---

## T

**Table de routage**
Structure de données dans un routeur listant les réseaux connus et les interfaces pour les atteindre.

**TCP (Transmission Control Protocol)**
Protocole de transport orienté connexion, fiable, avec contrôle de flux et de congestion. Utilisé par HTTP, SMTP, SSH, etc.

**TCP/IP**
Famille de protocoles formant la base d'Internet, organisée en 4 couches : Accès réseau, Internet, Transport, Application.

**Telnet**
Protocole d'accès distant en texte clair (non sécurisé). Remplacé par *SSH*. Port 23.

**Three-way Handshake**
Établissement de connexion TCP en trois étapes : SYN → SYN-ACK → ACK.

**Timeout**
Délai maximal d'attente d'une réponse avant de considérer qu'une opération a échoué.

**TLS (Transport Layer Security)**
Protocole cryptographique sécurisant les communications. Successeur de *SSL*. Utilisé dans *HTTPS*, SMTPS, IMAPS.

**Topologie**
Organisation physique ou logique d'un réseau : bus, étoile, anneau, maillage, arbre.

**Traceroute**
Utilitaire affichant le chemin (liste des routeurs) parcouru par un paquet jusqu'à sa destination. Utilise *ICMP* ou UDP.

**Trame (Frame)**
Unité de données au niveau de la couche liaison (couche 2), incluant en-têtes et données encapsulées.

**TTL (Time To Live)**
Champ IP limitant la durée de vie d'un paquet (nombre de *hops*). Décrémenté à chaque routeur, le paquet est détruit si TTL atteint 0.

**Tunnel**
Encapsulation d'un protocole dans un autre pour traverser un réseau. Utilisé dans les *VPN*, *IPv6* sur IPv4, etc.

---

## U

**UDP (User Datagram Protocol)**
Protocole de transport sans connexion, non fiable, sans contrôle de flux. Utilisé pour DNS, streaming, VoIP, jeux en ligne.

**Unicast**
Transmission point à point d'un paquet d'un émetteur vers un seul destinataire.

**URL (Uniform Resource Locator)**
Adresse web complète : protocole, nom de domaine, chemin. Exemple : https://www.example.com/page.html

**UTF-8**
Encodage de caractères Unicode largement utilisé sur Internet, compatible avec ASCII.

---

## V

**VLAN (Virtual Local Area Network)**
Réseau local virtuel segmentant logiquement un réseau physique au niveau de la couche 2.

**VoIP (Voice over IP)**
Transmission de la voix sur réseau IP. Utilise généralement *UDP* pour minimiser la latence.

**VPN (Virtual Private Network)**
Réseau privé virtuel créant un *tunnel* chiffré à travers un réseau public. Utilise souvent *IPsec* ou TLS.

---

## W

**WAN (Wide Area Network)**
Réseau étendu couvrant une large zone géographique, interconnectant des *LAN*.

**WebSocket**
Protocole permettant une communication bidirectionnelle full-duplex sur une connexion TCP unique, initiée via HTTP. Port 80 ou 443.

**Well-known Port**
Ports 0-1023 réservés aux services standards (HTTP:80, HTTPS:443, SSH:22, DNS:53, etc.).

**Wi-Fi**
Technologie de réseau local sans fil basée sur les standards IEEE 802.11.

**Wireshark**
Analyseur de protocoles réseau open-source permettant la capture et l'inspection détaillée des trames.

---

## X-Z

**Zero Trust**
Modèle de sécurité ne faisant confiance à aucune entité par défaut, même à l'intérieur du périmètre réseau.

---

## Symboles et Notations

**/n (Notation CIDR)**
Indique le nombre de bits du masque de sous-réseau. Exemple : /24 = 255.255.255.0

**::** (Double deux-points IPv6)**
Représente une ou plusieurs séquences de zéros en IPv6. Exemple : 2001:db8::1 équivaut à 2001:0db8:0000:0000:0000:0000:0000:0001

---

*Ce glossaire est un document de référence vivant. Pour des définitions plus détaillées, consultez les sections correspondantes de la formation et les RFC officielles.*



⏭️ [B. Tableau récapitulatif des ports standards](/annexes/02-ports-standards.md)
