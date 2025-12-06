🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 1.4 Le modèle TCP/IP : architecture en 4 couches

## Introduction

Dans la section précédente, nous avons découvert le modèle OSI avec ses 7 couches élégantes et bien définies. C'est un excellent modèle théorique... mais ce n'est **pas** celui qu'utilise réellement Internet !

Le modèle qui fait fonctionner Internet au quotidien, c'est le **modèle TCP/IP** (aussi appelé modèle Internet ou modèle DoD - Department of Defense). Plus simple, plus pragmatique, avec seulement **4 couches**, c'est lui qui a gagné la "guerre des protocoles" des années 1980.

**Pourquoi étudier les deux ?**
- **OSI** : Pour comprendre les concepts et communiquer avec d'autres professionnels
- **TCP/IP** : Pour comprendre comment fonctionne **vraiment** Internet

**Analogie** : OSI est comme un manuel d'anatomie parfait et détaillé. TCP/IP est comme le corps humain réel - moins parfait théoriquement, mais c'est celui qui fonctionne et qui vit !

---

## Histoire : la guerre des protocoles

### Le contexte (années 1970-1980)

Dans les années 1970-1980, plusieurs modèles de réseaux se battaient pour devenir le standard :

**Le camp OSI** (ISO) :
- Soutenu par les gouvernements et les grandes entreprises européennes
- Approche théorique et académique
- "Faisons-le parfaitement du premier coup"
- 7 couches bien définies

**Le camp TCP/IP** (DARPA/DoD) :
- Développé pour ARPANET (ancêtre d'Internet)
- Approche pragmatique et évolutive
- "Faisons quelque chose qui marche, on améliorera après"
- 4 couches flexibles

### Le vainqueur

**TCP/IP a gagné** pour plusieurs raisons :

✅ **Il existait déjà** et fonctionnait (ARPANET utilisait TCP/IP depuis 1983)

✅ **Protocoles ouverts** : Gratuits, documentés, implémentables par tous

✅ **Flexibilité** : Plus simple, plus facile à implémenter

✅ **Adoption précoce** : Les universités américaines l'ont massivement adopté

✅ **L'explosion du Web** (1991) : HTTP s'est construit sur TCP/IP

**Analogie** : C'est comme VHS vs Betamax dans les années 1980. Betamax était techniquement supérieur, mais VHS a gagné parce qu'il était déjà partout, moins cher, et "suffisamment bon".

---

## Vue d'ensemble des 4 couches

Le modèle TCP/IP est organisé en **4 couches** (numérotées de bas en haut) :

```
┌─────────────────────────────────────────┐
│  4. Application                         │  ← Protocoles utilisateur
│     (HTTP, FTP, SMTP, DNS, SSH...)      │
├─────────────────────────────────────────┤
│  3. Transport                           │  ← TCP, UDP
│     (Fiabilité et multiplexage)         │
├─────────────────────────────────────────┤
│  2. Internet                            │  ← IP (IPv4, IPv6)
│     (Routage et adressage)              │
├─────────────────────────────────────────┤
│  1. Accès réseau                        │  ← Ethernet, Wi-Fi, etc.
│     (Liaison physique)                  │
└─────────────────────────────────────────┘
```

**Comparaison rapide avec OSI** :

```
       TCP/IP             |           OSI
                          |
    4. Application        |    7. Application
                          |    6. Présentation
                          |    5. Session
    ───────────────────── | ─────────────────────
    3. Transport          |    4. Transport
    ───────────────────── | ─────────────────────
    2. Internet           |    3. Réseau
    ───────────────────── | ─────────────────────
    1. Accès réseau       |    2. Liaison de données
                          |    1. Physique
```

**Note importante** : Certaines sources parlent d'un modèle TCP/IP en **5 couches**, séparant "Accès réseau" en "Liaison" et "Physique". Les deux approches sont correctes. Nous utiliserons le modèle classique en 4 couches.

---

## Couche 1 : Accès réseau (Network Access)

### Rôle

La couche d'accès réseau gère **tout ce qui concerne la transmission physique** des données sur le support de communication (câbles, ondes radio, fibre optique).

**Analogie** : C'est l'infrastructure routière complète - l'asphalte, les marquages au sol, les panneaux, les règles de circulation locale. Tout ce qui permet physiquement aux véhicules de circuler sur une route donnée.

### Responsabilités

Cette couche combine les fonctions des couches OSI 1 et 2 :

🔌 **Physique** :
- Type de câble (cuivre, fibre)
- Connecteurs (RJ-45, USB, etc.)
- Signaux électriques ou lumineux
- Débit binaire

🔗 **Liaison de données** :
- Adressage MAC
- Tramage (création des trames)
- Détection d'erreurs (CRC)
- Contrôle d'accès au média

### Technologies concernées

**Filaires** :
- **Ethernet** (IEEE 802.3) : Le standard des réseaux locaux câblés
- **PPP** (Point-to-Point Protocol) : Connexions point à point
- **Fibre optique** : Transmissions longue distance

**Sans fil** :
- **Wi-Fi** (IEEE 802.11) : Réseaux locaux sans fil
- **Bluetooth** : Courte portée
- **4G/5G** : Réseaux mobiles

### Unité de données

**La trame (frame)**

### Exemples concrets

**Ethernet** :
```
Votre ordinateur veut envoyer des données au routeur
↓
La carte réseau met les données dans une trame Ethernet
↓
Elle ajoute l'adresse MAC de destination (le routeur)
↓
Elle envoie les signaux électriques sur le câble RJ-45
```

**Wi-Fi** :
```
Votre smartphone veut se connecter
↓
Il négocie avec le point d'accès (authentification)
↓
Il encode les données en ondes radio
↓
Il transmet sur la fréquence 2.4 GHz ou 5 GHz
```

### Caractéristiques importantes

✅ **Dépendante du média** : Cette couche change complètement selon la technologie (Ethernet ≠ Wi-Fi)

✅ **Portée locale** : Ne fonctionne que sur le réseau local (LAN)

✅ **Adressage MAC** : Identifie les cartes réseau physiques

❌ **Pas de routage** : Ne peut pas traverser plusieurs réseaux

---

## Couche 2 : Internet (ou Interréseau)

### Rôle

La couche Internet (équivalent de la couche Réseau OSI) est le **cœur du modèle TCP/IP**. Elle permet de faire voyager des paquets à travers **plusieurs réseaux différents** pour atteindre n'importe quelle destination sur la planète.

**Analogie** : C'est le système postal international. Peu importe où vous êtes dans le monde, si vous connaissez l'adresse complète du destinataire, votre lettre trouvera son chemin en passant par différents bureaux de poste, différents pays, différents modes de transport.

### Responsabilités

🌍 **Adressage universel** : Chaque machine a une adresse IP unique (ou presque)

🗺️ **Routage** : Déterminer le chemin optimal entre source et destination

📦 **Fragmentation** : Découper les paquets trop gros si nécessaire

🚚 **Acheminement de bout en bout** : Faire traverser Internet aux données

### Le protocole principal : IP (Internet Protocol)

**IP** est LA star de cette couche. C'est lui qui permet à Internet d'exister.

**Deux versions coexistent** :

**IPv4** (Internet Protocol version 4) :
- Créé dans les années 1970
- Adresses sur 32 bits : `192.168.1.1`
- ~4.3 milliards d'adresses possibles
- ⚠️ Pratiquement épuisé aujourd'hui

**IPv6** (Internet Protocol version 6) :
- Déployé depuis les années 2000
- Adresses sur 128 bits : `2001:0db8:85a3::8a2e:0370:7334`
- 340 sextillions d'adresses (340 × 10³⁶)
- 🚀 L'avenir d'Internet

### Autres protocoles de cette couche

**ICMP (Internet Control Message Protocol)** :
- Messages de diagnostic et d'erreur
- Utilisé par `ping` et `traceroute`
- Exemple : "Destination injoignable", "TTL expiré"

**ARP (Address Resolution Protocol)** :
- Fait le lien entre adresses IP et adresses MAC
- "Quelle est l'adresse MAC de 192.168.1.1 ?"

**IGMP (Internet Group Management Protocol)** :
- Gestion du multicast (diffusion vers un groupe)

### Structure d'un paquet IP (simplifié)

```
┌─────────────────────────────────────────────┐
│ En-tête IP (20-60 octets)                   │
│  ├─ Version (IPv4 ou IPv6)                  │
│  ├─ Longueur totale                         │
│  ├─ Identification                          │
│  ├─ TTL (Time To Live) : durée de vie       │
│  ├─ Protocole : TCP=6, UDP=17, ICMP=1       │
│  ├─ Checksum : détection d'erreurs          │
│  ├─ Adresse IP source : 192.168.1.100       │
│  └─ Adresse IP destination : 93.184.216.34  │
├─────────────────────────────────────────────┤
│ Données (payload)                           │
│  Contient le segment TCP ou UDP             │
└─────────────────────────────────────────────┘
```

### Le routage en action

**Exemple** : Vous à Paris envoyez des données à un serveur à Tokyo

```
Votre PC (Paris)
   ↓ 192.168.1.100
[Routeur maison] → Vers FAI français
   ↓
[Routeur FAI] → Vers nœud international
   ↓
[Routeurs intermédiaires] → Traversée de l'Europe et l'Asie
   ↓ (peut-être 10-15 routeurs)
[Routeur FAI japonais]
   ↓
[Routeur datacenter]
   ↓ 203.0.113.42
Serveur (Tokyo)
```

Chaque routeur consulte sa **table de routage** pour décider où envoyer le paquet suivant.

**Important** : Le routeur ne connaît pas tout le chemin ! Il connaît juste le "prochain saut" (next hop). C'est comme demander son chemin : chaque personne vous indique juste la prochaine direction, pas l'itinéraire complet.

### Le TTL : éviter les boucles infinies

Le **TTL (Time To Live)** est un compteur qui **décrémente à chaque routeur traversé**.

```
Départ : TTL = 64
Routeur 1 : TTL = 63
Routeur 2 : TTL = 62
...
Routeur 64 : TTL = 0 → Le paquet est détruit
```

**Pourquoi ?** Pour éviter qu'un paquet tourne indéfiniment si la configuration du routage est incorrecte.

**Analogie** : C'est comme une lettre avec un timbre qui "s'efface" un peu à chaque bureau de poste. Si elle fait trop de trajets, elle est détruite pour éviter d'encombrer le système postal.

### Unité de données

**Le paquet (packet) ou datagramme**

---

## Couche 3 : Transport

### Rôle

La couche transport assure la **communication de bout en bout entre applications**. Elle offre deux niveaux de service : fiable (TCP) ou rapide (UDP).

**Analogie** : Si la couche Internet est le système postal qui achemine les lettres à la bonne ville et la bonne rue, la couche Transport est le service qui garantit (ou non) que :
- Le colis arrive intact
- Il arrive dans l'ordre
- Rien ne manque
- Le destinataire signe un accusé de réception

### Responsabilités

🎯 **Multiplexage par ports** : Permettre à plusieurs applications de communiquer simultanément

🔢 **Segmentation** : Découper les gros messages en segments gérables

🔒 **Fiabilité (TCP)** : Garantir la livraison sans perte

📊 **Contrôle de flux** : Adapter le débit à la capacité du récepteur

🚦 **Contrôle de congestion** : S'adapter aux conditions du réseau

### Les deux protocoles majeurs

#### TCP (Transmission Control Protocol)

**Caractéristiques** :
- ✅ **Fiable** : Garantit que toutes les données arrivent
- ✅ **Ordonné** : Les données arrivent dans le bon ordre
- ✅ **Avec connexion** : Établit une session avant l'échange (handshake)
- ✅ **Contrôle de flux** : Adapte le débit
- ✅ **Contrôle de congestion** : Évite de surcharger le réseau
- ❌ **Plus lent** : Beaucoup d'overhead (en-têtes, acquittements)

**Analogie** : Un appel téléphonique
- Vous appelez (établissement de connexion)
- Vous attendez que l'autre décroche
- Vous conversez (vous assurez que l'autre a bien compris)
- Vous raccrochez (fin de connexion propre)

**Utilisé pour** :
- 🌐 Navigation web (HTTP/HTTPS)
- 📧 Email (SMTP, IMAP)
- 📁 Transfert de fichiers (FTP, SFTP)
- 🔐 Connexions sécurisées (SSH)
- 💬 Applications de messagerie

**Structure d'un segment TCP** (simplifié) :
```
┌─────────────────────────────────────────┐
│ En-tête TCP (20-60 octets)              │
│  ├─ Port source (ex: 54321)             │
│  ├─ Port destination (ex: 443 HTTPS)    │
│  ├─ Numéro de séquence                  │
│  ├─ Numéro d'acquittement               │
│  ├─ Flags (SYN, ACK, FIN, RST...)       │
│  ├─ Fenêtre (window size)               │
│  └─ Checksum                            │
├─────────────────────────────────────────┤
│ Données de l'application                │
└─────────────────────────────────────────┘
```

**Le 3-way handshake de TCP** :

```
Client                               Serveur
  │                                     │
  │───── SYN (Je veux me connecter) ───→│
  │                                     │
  │←──── SYN-ACK (OK, je suis prêt) ─── │
  │                                     │
  │───── ACK (Parfait, allons-y) ──→    │
  │                                     │
  │═══ Connexion établie ═══════════════│
  │                                     │
  │←──── Données ────→                  │
```

#### UDP (User Datagram Protocol)

**Caractéristiques** :
- ⚡ **Rapide** : Très peu d'overhead
- ❌ **Non fiable** : Pas de garantie de livraison
- ❌ **Non ordonné** : Les paquets peuvent arriver dans le désordre
- 📡 **Sans connexion** : Pas de handshake, on envoie directement
- 🎯 **Léger** : En-tête de seulement 8 octets

**Analogie** : Une carte postale
- Vous l'envoyez directement (pas de connexion à établir)
- Pas de garantie qu'elle arrive
- Pas de confirmation de réception
- Rapide et simple

**Utilisé pour** :
- 📺 Streaming vidéo/audio (Netflix, YouTube Live, Spotify)
- 🎮 Jeux en ligne (FPS, jeux d'action)
- 🔍 DNS (requêtes courtes)
- 📞 VoIP (Skype, WhatsApp calls, Zoom)
- 📡 Diffusion multicast

**Structure d'un datagramme UDP** :
```
┌─────────────────────────────────────────┐
│ En-tête UDP (8 octets seulement !)      │
│  ├─ Port source                         │
│  ├─ Port destination                    │
│  ├─ Longueur                            │
│  └─ Checksum                            │
├─────────────────────────────────────────┤
│ Données de l'application                │
└─────────────────────────────────────────┘
```

### TCP vs UDP : Quand utiliser quoi ?

| Critère | TCP | UDP |
|---------|-----|-----|
| **Fiabilité** | ✅ Garantie | ❌ Best effort |
| **Ordre** | ✅ Préservé | ❌ Pas garanti |
| **Vitesse** | 🐢 Plus lent | 🚀 Très rapide |
| **Overhead** | 📊 Important | ⚡ Minimal |
| **Connexion** | 🔌 Oui (3-way) | ❌ Non |
| **Use case** | Données critiques | Temps réel |

**Règle générale** :
- Utilisez **TCP** quand vous **ne pouvez pas vous permettre de perdre des données** (fichiers, emails, transactions)
- Utilisez **UDP** quand la **vitesse est plus importante que la perfection** (streaming, jeux, appels vocaux)

### Les ports : l'adressage des applications

Les **ports** permettent à plusieurs applications de communiquer simultanément sur la même machine.

**Analogie** : L'adresse IP est comme l'adresse d'un immeuble. Le port est comme le numéro d'appartement.

```
Adresse complète : 192.168.1.100:443
                   └─ IP ──┘ └Port┘
```

**Ports bien connus** (0-1023) :
- Port 80 : HTTP (web non sécurisé)
- Port 443 : HTTPS (web sécurisé)
- Port 22 : SSH (connexion sécurisée)
- Port 25 : SMTP (email sortant)
- Port 53 : DNS (résolution de noms)
- Port 21 : FTP (transfert de fichiers)

**Exemple pratique** :
```
Votre navigateur ouvre www.example.com
↓
DNS résout vers 93.184.216.34
↓
Votre navigateur se connecte à 93.184.216.34:443 (HTTPS)
↓
Port source aléatoire (ex: 54321) → Port destination 443
↓
La page web est téléchargée via cette connexion TCP
```

### Unité de données

- **TCP** : Segment
- **UDP** : Datagramme

---

## Couche 4 : Application

### Rôle

La couche Application contient **tous les protocoles de haut niveau** utilisés directement par les applications et les utilisateurs. C'est l'interface entre le réseau et les programmes.

**Analogie** : Si les couches inférieures sont l'infrastructure postale (routes, bureaux de tri, camions), la couche Application, c'est le contenu des lettres : les formulaires administratifs, les cartes de vœux, les factures, les contrats...

### Responsabilités

Cette couche **regroupe les fonctions des couches OSI 5, 6 et 7** :

🎭 **Session** : Gestion des connexions applicatives

🔄 **Présentation** : Encodage, compression, chiffrement

🖥️ **Application** : Services réseau pour l'utilisateur final

### Protocoles majeurs

#### Web

**HTTP (HyperText Transfer Protocol)** :
- Protocole du Web
- Port 80
- Non chiffré ⚠️

**HTTPS (HTTP Secure)** :
- HTTP + TLS/SSL
- Port 443
- Chiffré et authentifié ✅

**Exemple de requête HTTP** :
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

#### Email

**SMTP (Simple Mail Transfer Protocol)** :
- Envoi d'emails
- Port 25 (ou 587 avec authentification)
- Serveur → Serveur

**POP3 (Post Office Protocol v3)** :
- Réception d'emails
- Port 110
- Télécharge et supprime du serveur

**IMAP (Internet Message Access Protocol)** :
- Réception d'emails
- Port 143
- Synchronise avec le serveur (emails restent sur le serveur)

#### Fichiers

**FTP (File Transfer Protocol)** :
- Transfert de fichiers
- Port 21 (commandes) et 20 (données)
- Non sécurisé ⚠️

**SFTP (SSH File Transfer Protocol)** :
- Transfert sécurisé
- Port 22
- Utilise SSH pour le chiffrement

#### Autres protocoles essentiels

**DNS (Domain Name System)** :
- Résolution de noms : `www.example.com` → `93.184.216.34`
- Port 53 (UDP et TCP)
- Essentiel pour Internet !

**DHCP (Dynamic Host Configuration Protocol)** :
- Attribution automatique d'adresses IP
- Ports 67 (serveur) et 68 (client)
- "Plug and play" réseau

**SSH (Secure Shell)** :
- Connexion à distance sécurisée
- Port 22
- Remplace Telnet (obsolète et non sécurisé)

**Telnet** :
- Connexion à distance non sécurisée
- Port 23
- ⚠️ Obsolète, remplacé par SSH

**SNMP (Simple Network Management Protocol)** :
- Supervision et gestion réseau
- Ports 161 et 162
- Surveillance des équipements

### Exemples d'interactions complètes

**Exemple 1 : Charger une page web**

```
1. Vous tapez : https://www.example.com

2. DNS (Couche Application)
   → Résout www.example.com en 93.184.216.34

3. TCP (Couche Transport)
   → Établit connexion TCP vers 93.184.216.34:443
   → 3-way handshake

4. TLS (Couche Application)
   → Handshake TLS pour sécuriser la connexion
   → Échange de certificats et clés

5. HTTP (Couche Application)
   → GET /index.html HTTP/1.1
   → Le serveur répond avec le HTML

6. Votre navigateur affiche la page
```

**Exemple 2 : Envoyer un email**

```
1. Vous écrivez un email à ami@example.com

2. DNS
   → Résout le serveur de mail de example.com

3. TCP
   → Connexion au serveur SMTP (port 25/587)

4. SMTP
   → AUTH : Authentification
   → MAIL FROM: votre-adresse
   → RCPT TO: ami@example.com
   → DATA: Contenu de l'email

5. Le serveur envoie l'email au serveur du destinataire

6. Le destinataire récupère l'email via IMAP/POP3
```

### Unité de données

**Messages** (format dépend du protocole)

---

## L'encapsulation dans TCP/IP

À chaque couche, on **ajoute un en-tête** (et parfois un pied de page). C'est le principe des poupées russes.

### Processus descendant (envoi)

```
Couche Application
[Données HTTP]
         ↓ Ajoute en-tête TCP
Couche Transport
[TCP Header | Données HTTP] ← Segment TCP
         ↓ Ajoute en-tête IP
Couche Internet
[IP Header | TCP Header | Données HTTP] ← Paquet IP
         ↓ Ajoute en-tête Ethernet + CRC
Couche Accès réseau
[Eth | IP | TCP | HTTP | CRC] ← Trame Ethernet
         ↓
     Transmission physique (bits)
```

### Processus ascendant (réception)

```
     Réception physique (bits)
         ↑
Couche Accès réseau
[Eth | IP | TCP | HTTP | CRC]
         ↑ Retire Ethernet, vérifie CRC
Couche Internet
[IP Header | TCP Header | Données HTTP]
         ↑ Retire IP, vérifie destination
Couche Transport
[TCP Header | Données HTTP]
         ↑ Retire TCP, réassemble, vérifie ordre
Couche Application
[Données HTTP]
         ↑
    Application affiche le résultat
```

### Visualisation avec adresses

```
┌──────────────────────────────────────────────────────────┐
│ Eth: MAC src → MAC dest                                  │
│ ┌──────────────────────────────────────────────────────┐ │
│ │ IP: 192.168.1.100 → 93.184.216.34                    │ │
│ │ ┌──────────────────────────────────────────────────┐ │ │
│ │ │ TCP: Port 54321 → Port 443                       │ │ │
│ │ │ ┌──────────────────────────────────────────────┐ │ │ │
│ │ │ │ HTTP: GET /index.html                        │ │ │ │
│ │ │ └──────────────────────────────────────────────┘ │ │ │
│ │ └──────────────────────────────────────────────────┘ │ │
│ └──────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

**Overhead** : Chaque en-tête ajoute de l'information de contrôle. Pour envoyer 100 octets de données, vous pouvez facilement transmettre 150-200 octets au total !

---

## Comparaison TCP/IP vs OSI

### Tableau comparatif

| Aspect | TCP/IP | OSI |
|--------|--------|-----|
| **Nombre de couches** | 4 (ou 5) | 7 |
| **Nature** | Pratique, implémenté | Théorique, référence |
| **Création** | Années 1970 (DARPA) | 1984 (ISO) |
| **Adoption** | Internet entier | Modèle de référence |
| **Flexibilité** | Très flexible | Rigide |
| **Protocoles** | Suite définie (TCP, UDP, IP) | Indépendant des protocoles |

### Correspondance des couches

```
TCP/IP                      OSI

Application       ←→        Application
                           Présentation
                           Session

Transport         ←→        Transport

Internet          ←→        Réseau (Network)

Accès réseau      ←→        Liaison de données
                           Physique
```

### Pourquoi TCP/IP a gagné ?

✅ **Pragmatique** : "Ça marche" plutôt que "c'est parfait"

✅ **Ouvert** : Spécifications publiques (RFC), pas de brevets

✅ **Évolutif** : Facile d'ajouter de nouveaux protocoles

✅ **Déjà déployé** : ARPANET puis Internet l'utilisaient déjà

✅ **Simple** : 4 couches au lieu de 7

✅ **Indépendant du matériel** : Fonctionne sur n'importe quoi

---

## Forces du modèle TCP/IP

### 1. Interopérabilité universelle

N'importe quel appareil peut communiquer avec n'importe quel autre :
- Windows ↔ Linux ↔ macOS ↔ Android ↔ iOS
- Peu importe le matériel, le système d'exploitation, le fabricant

### 2. Décentralisation

Pas de point central de contrôle. Si une partie d'Internet tombe, le reste continue de fonctionner.

**Analogie** : C'est comme un réseau routier. Si une autoroute est bloquée, on peut prendre un autre chemin.

### 3. Scalabilité

TCP/IP a évolué de **4 ordinateurs** (1969) à **5+ milliards d'appareils** aujourd'hui, sans refonte majeure !

### 4. Neutralité technologique

TCP/IP fonctionne sur **n'importe quel support** :
- Câble cuivre
- Fibre optique
- Wi-Fi
- Satellite
- 4G/5G
- Infrarouge
- Même par pigeons voyageurs ! (IP over Avian Carriers - RFC 1149, un poisson d'avril !)

### 5. Innovation en bordure

L'intelligence est aux **extrémités** (vos appareils), pas au centre du réseau. Cela permet d'innover rapidement sans reconfigurer tout Internet.

**Principe de bout en bout** : Le réseau se contente de transporter les paquets. Les applications gèrent l'intelligence.

---

## Limites et défis

### 1. Sécurité non native

TCP/IP a été conçu dans les années 1970 pour un réseau de confiance (universités, militaires). La sécurité a été ajoutée après coup.

**Solutions** : TLS/SSL, IPsec, VPN, pare-feu...

### 2. Épuisement d'IPv4

~4.3 milliards d'adresses IPv4 ne suffisent plus.

**Solution** : Migration vers IPv6 (en cours, mais lente)

### 3. Qualité de Service (QoS)

TCP/IP traite tous les paquets de la même façon. Difficile de garantir des performances pour les applications critiques (vidéo, VoIP).

**Solutions** : MPLS, DiffServ, priorités...

### 4. Mobilité

TCP/IP suppose des connexions relativement stables. La mobilité (smartphones qui changent de réseau) pose des défis.

**Solutions** : Mobile IP, protocoles applicatifs adaptés

---

## Évolution de TCP/IP

TCP/IP continue d'évoluer :

**IPv6** : Nouvelle version d'IP avec adresses sur 128 bits

**HTTP/2 et HTTP/3** : Amélioration des performances web

**QUIC** : Nouveau protocole de transport basé sur UDP

**TLS 1.3** : Amélioration de la sécurité

**DoH/DoT** : DNS sur HTTPS/TLS pour la confidentialité

**Concepts** : L'architecture en couches reste, mais les protocoles s'améliorent continuellement.

---

## Conclusion

Le modèle TCP/IP est le **fondement d'Internet**. Sa simplicité (4 couches), sa flexibilité et son pragmatisme en ont fait le vainqueur de la "guerre des protocoles".

**Les 4 piliers de TCP/IP** :

1. **Accès réseau** : La connexion physique (Ethernet, Wi-Fi...)
2. **Internet (IP)** : Le routage universel entre réseaux
3. **Transport (TCP/UDP)** : La livraison fiable ou rapide aux applications
4. **Application** : Les protocoles que vous utilisez quotidiennement (HTTP, Email, DNS...)

Comprendre ce modèle, c'est comprendre comment **tout Internet fonctionne**. Chaque fois que vous :
- Visitez un site web
- Envoyez un email
- Regardez une vidéo en streaming
- Jouez en ligne
- Utilisez une app sur votre smartphone

...vous utilisez TCP/IP !

Dans les sections suivantes, nous allons approfondir chaque couche et découvrir les mécanismes fascinants qui font fonctionner notre monde hyperconnecté.

---

**À retenir** :
- 🌍 **TCP/IP** = Le modèle qui fait fonctionner Internet (vs OSI = modèle théorique)
- 4️⃣ **4 couches** : Accès réseau, Internet, Transport, Application
- 📦 **IP** = Le routage universel entre réseaux
- 🔒 **TCP** = Fiable et ordonné | ⚡ **UDP** = Rapide et léger
- 🎯 **Ports** = Multiplexage des applications (HTTP=80, HTTPS=443...)
- 📚 **Pragmatique** : "Ça marche" plutôt que "c'est parfait"
- 🚀 **Évolutif** : De 4 machines à 5+ milliards d'appareils

**Prochaine étape** : Comparons en détail OSI et TCP/IP pour consolider notre compréhension ! →

⏭️ [Comparaison OSI vs TCP/IP](/01-introduction/05-comparaison-osi-tcp-ip.md)
