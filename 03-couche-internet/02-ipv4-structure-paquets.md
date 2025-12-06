🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.2 IPv4 : structure et format des paquets

## Introduction : anatomie d'un paquet IP

Vous avez compris que la couche Internet envoie des **paquets IP** à travers les réseaux. Mais qu'est-ce qu'un paquet IP exactement ? Comment est-il construit ? Quelles informations contient-il ?

Un paquet IP, c'est comme une **enveloppe postale sophistiquée** : elle contient non seulement les adresses de l'expéditeur et du destinataire, mais aussi tout un ensemble d'informations techniques nécessaires à son acheminement correct.

Dans cette section, nous allons ouvrir cette enveloppe et examiner chaque élément qu'elle contient.

## La structure générale d'un paquet IPv4

Un paquet IPv4 se compose de deux parties principales :

```
┌─────────────────────────────────────────┐
│         EN-TÊTE IP (Header)             │  ← 20 à 60 octets
│    (informations de routage)            │
├─────────────────────────────────────────┤
│         DONNÉES (Payload)               │  ← Variable
│    (ce que vous voulez transmettre)     │
└─────────────────────────────────────────┘
```

### 1. L'en-tête (Header)

C'est la partie **administrative** du paquet. Elle contient toutes les métadonnées nécessaires pour :
- Identifier l'expéditeur et le destinataire
- Router le paquet correctement
- Gérer sa durée de vie
- Vérifier son intégrité

**Taille** : Au minimum 20 octets, et jusqu'à 60 octets si des options sont utilisées.

### 2. Les données (Payload)

C'est le **contenu utile** que vous voulez transmettre. Il s'agit généralement d'un segment TCP ou d'un datagramme UDP provenant de la couche Transport.

**Taille** : Variable, théoriquement jusqu'à 65 515 octets, mais en pratique limitée par le MTU du réseau (généralement 1500 octets pour Ethernet).

**Analogie** :
- L'en-tête = l'enveloppe avec toutes les informations postales
- Les données = la lettre à l'intérieur de l'enveloppe

## L'en-tête IPv4 en détail : les 20 octets essentiels

Voici la structure standard d'un en-tête IPv4 (sans options) :

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source IP Address                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination IP Address                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if IHL > 5)                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Ne paniquez pas ! Nous allons décortiquer chaque champ un par un. 😊

## Les champs de l'en-tête IPv4 expliqués simplement

### 🔹 Version (4 bits)

**Rôle** : Indique la version du protocole IP utilisé.

**Valeur** : `4` pour IPv4 (logique !), `6` pour IPv6.

**Pourquoi c'est utile** : Cela permet aux routeurs de savoir comment interpréter le reste du paquet. Un routeur qui reçoit un paquet avec Version = 6 sait qu'il doit le traiter comme un paquet IPv6.

```
Version = 4 (0100 en binaire) → C'est un paquet IPv4
```

**Analogie** : C'est comme le logo sur une enveloppe qui indique quel service postal l'a émise (La Poste, UPS, DHL...).

---

### 🔹 IHL - Internet Header Length (4 bits)

**Rôle** : Indique la longueur de l'en-tête IP en mots de 32 bits (4 octets).

**Valeur** : Minimum `5` (5 × 4 = 20 octets), maximum `15` (15 × 4 = 60 octets).

**Pourquoi c'est utile** : Comme l'en-tête peut avoir une taille variable (avec ou sans options), ce champ permet de savoir où commence la partie données.

```
IHL = 5 → En-tête de 20 octets (standard, sans options)
IHL = 6 → En-tête de 24 octets (4 octets d'options)
```

**Exemple pratique** : Si IHL = 5, vous savez que les données commencent au 21ème octet du paquet.

---

### 🔹 Type of Service / ToS ou DSCP (8 bits)

**Rôle** : Indique la priorité et le type de traitement souhaité pour ce paquet.

**Utilisation moderne** : Ce champ est principalement utilisé pour la **QoS (Quality of Service)** - prioriser certains trafics (VoIP, vidéo) sur d'autres (téléchargement).

**Structure** :
- 6 premiers bits : **DSCP** (Differentiated Services Code Point) - indique la classe de service
- 2 derniers bits : **ECN** (Explicit Congestion Notification) - signale la congestion réseau

```
Exemple :
DSCP = 46 (EF - Expedited Forwarding) → Trafic VoIP prioritaire
DSCP = 0  (BE - Best Effort)          → Trafic normal
```

**Analogie** : C'est comme choisir entre "courrier normal" et "courrier prioritaire" à La Poste. Le prioritaire sera traité en premier.

---

### 🔹 Total Length (16 bits)

**Rôle** : Indique la taille totale du paquet (en-tête + données) en octets.

**Valeur** : De 20 octets (en-tête minimal sans données) à 65 535 octets (maximum théorique).

**Pourquoi c'est utile** : Permet de savoir où se termine le paquet, surtout important lors de la fragmentation.

```
Total Length = 1500 → Paquet de 1500 octets au total
Si IHL = 5 (20 octets d'en-tête) → 1480 octets de données
```

**Note pratique** : Sur Ethernet, la taille maximale d'une trame est généralement 1518 octets, ce qui limite les paquets IP à environ 1500 octets (MTU).

---

### 🔹 Identification (16 bits)

**Rôle** : Identifiant unique attribué à chaque paquet (ou groupe de fragments) émis par une source.

**Pourquoi c'est utile** : Utilisé principalement pour la **fragmentation**. Si un paquet est découpé en plusieurs fragments, tous les fragments partageront le même numéro d'identification.

```
Paquet original : Identification = 12345
    ↓ Fragmentation ↓
Fragment 1 : Identification = 12345
Fragment 2 : Identification = 12345
Fragment 3 : Identification = 12345
```

Cela permet au destinataire de savoir quels fragments vont ensemble et de les réassembler correctement.

**Analogie** : C'est comme un numéro de colis sur un meuble IKEA livré en plusieurs cartons. Tous les cartons ont le même numéro de commande.

---

### 🔹 Flags (3 bits)

**Rôle** : Contrôle la fragmentation du paquet.

**Les 3 bits** :
- **Bit 0** : Réservé, toujours à 0
- **Bit 1 (DF - Don't Fragment)** : Si à 1, interdit la fragmentation du paquet
- **Bit 2 (MF - More Fragments)** : Si à 1, indique qu'il y a d'autres fragments à venir

```
Flags = 010 (DF=1, MF=0) → "Ne me fragmente pas !"
Flags = 001 (DF=0, MF=1) → "Je suis un fragment et il y en a d'autres"
Flags = 000 (DF=0, MF=0) → "Je suis le dernier fragment" ou "je ne suis pas fragmenté"
```

**Exemple pratique** : Certaines applications (comme les jeux en ligne) mettent DF=1 pour éviter la fragmentation qui augmenterait la latence. Si un routeur ne peut pas transmettre le paquet sans le fragmenter, il le rejette et envoie un message ICMP "Fragmentation needed but DF set".

---

### 🔹 Fragment Offset (13 bits)

**Rôle** : Indique la position de ce fragment dans le paquet original (en unités de 8 octets).

**Pourquoi c'est utile** : Permet de reconstituer le paquet original dans le bon ordre.

```
Paquet original de 2400 octets découpé en 3 fragments :

Fragment 1 : Offset = 0    (octets 0-799)
Fragment 2 : Offset = 100  (octets 800-1599, car 100 × 8 = 800)
Fragment 3 : Offset = 200  (octets 1600-2399, car 200 × 8 = 1600)
```

**Note** : L'offset est en multiples de 8 octets, donc le maximum est 8191 × 8 = 65 528 octets.

**Analogie** : Si vous découpez un film en plusieurs parties, l'offset indique "Partie 1 sur 3", "Partie 2 sur 3", etc.

---

### 🔹 Time to Live / TTL (8 bits)

**Rôle** : Limite la durée de vie du paquet pour éviter qu'il ne circule indéfiniment.

**Fonctionnement** :
- Initialisé à une certaine valeur (généralement 64, 128 ou 255)
- Chaque routeur traversé décrémente le TTL de 1
- Si TTL atteint 0, le paquet est détruit et un message ICMP "Time Exceeded" est renvoyé

```
Source → TTL = 64
Routeur 1 → TTL = 63
Routeur 2 → TTL = 62
[...]
Routeur 64 → TTL = 0 → ❌ Paquet détruit
```

**Utilité pratique** :
1. **Éviter les boucles infinies** : Si une mauvaise configuration crée une boucle de routage, le paquet sera détruit après X sauts
2. **Outil de diagnostic** : La commande `traceroute` utilise intelligemment le TTL pour cartographier le chemin

**Analogie** : C'est comme une batterie qui se décharge à chaque relais. Quand elle est vide, le paquet "meurt".

---

### 🔹 Protocol (8 bits)

**Rôle** : Indique quel protocole de la couche supérieure (Transport) est encapsulé dans les données.

**Valeurs courantes** :
- `1` = ICMP (Internet Control Message Protocol)
- `6` = TCP (Transmission Control Protocol)
- `17` = UDP (User Datagram Protocol)
- `89` = OSPF (protocole de routage)

```
Protocol = 6 → Les données contiennent un segment TCP
Protocol = 17 → Les données contiennent un datagramme UDP
```

**Pourquoi c'est utile** : Cela permet au système d'exploitation destinataire de savoir à quel module passer les données une fois décapsulées.

**Analogie** : C'est comme le code-barres sur un colis qui indique dans quel département du magasin il doit être livré.

---

### 🔹 Header Checksum (16 bits)

**Rôle** : Somme de contrôle pour détecter les erreurs dans l'en-tête (et uniquement l'en-tête).

**Fonctionnement** :
1. À l'émission, un calcul mathématique est fait sur tous les champs de l'en-tête
2. Le résultat est stocké dans ce champ
3. À chaque routeur, le checksum est recalculé et comparé
4. Si différent → l'en-tête est corrompu → paquet rejeté

```
Calcul du checksum :
1. Diviser l'en-tête en mots de 16 bits
2. Additionner tous les mots
3. Prendre le complément à 1 du résultat
```

**Important** : Le checksum ne protège QUE l'en-tête, pas les données. C'est TCP ou UDP qui vérifient l'intégrité des données.

**Note** : Comme le TTL change à chaque routeur, le checksum doit être recalculé à chaque saut !

**Analogie** : C'est comme un code de sécurité sur une enveloppe qui permet de détecter si elle a été ouverte ou altérée en transit.

---

### 🔹 Source IP Address (32 bits)

**Rôle** : L'adresse IP de l'émetteur du paquet.

**Format** : 4 octets, généralement notés en décimal pointé (ex: `192.168.1.10`).

```
Binaire    : 11000000.10101000.00000001.00001010
Décimal    : 192.168.1.10
```

**Pourquoi c'est utile** :
- Le destinataire sait d'où vient le paquet
- Permet de renvoyer une réponse
- Utilisé pour le filtrage et la sécurité

**Note** : Cette adresse peut être modifiée (spoofing), c'est pourquoi on ne peut pas toujours lui faire confiance pour la sécurité.

---

### 🔹 Destination IP Address (32 bits)

**Rôle** : L'adresse IP du destinataire du paquet.

**Format** : 4 octets, même format que l'adresse source.

**Pourquoi c'est utile** :
- Les routeurs utilisent cette adresse pour déterminer où envoyer le paquet
- C'est le champ le plus important pour le routage !

```
Exemple :
Source      : 192.168.1.10  (vous)
Destination : 93.184.216.34 (example.com)
```

**Analogie** : C'est l'adresse de livraison sur votre colis Amazon. Sans elle, impossible de savoir où l'envoyer !

---

### 🔹 Options (variable, 0 à 40 octets)

**Rôle** : Champs optionnels pour des fonctionnalités avancées (rarement utilisés).

**Exemples d'options** :
- **Record Route** : Enregistrer l'adresse IP de chaque routeur traversé
- **Timestamp** : Enregistrer l'heure de passage à chaque routeur
- **Source Routing** : Spécifier le chemin exact que doit suivre le paquet

**Pourquoi rarement utilisées** :
- Ralentissent le traitement (les routeurs doivent analyser chaque option)
- Peuvent poser des problèmes de sécurité
- La plupart des routeurs modernes ignorent ou rejettent les options

**Si présentes** : L'en-tête peut faire jusqu'à 60 octets au lieu de 20.

---

## Exemple concret d'un paquet IPv4

Imaginons que vous envoyez un ping à `8.8.8.8` (serveur DNS de Google). Voici à quoi pourrait ressembler l'en-tête :

```
Version            : 4
IHL                : 5 (20 octets d'en-tête)
Type of Service    : 0 (trafic normal)
Total Length       : 84 octets (20 en-tête + 64 données)
Identification     : 54321
Flags              : DF=1, MF=0 (ne pas fragmenter)
Fragment Offset    : 0
Time to Live       : 64
Protocol           : 1 (ICMP)
Header Checksum    : 0xb861
Source IP          : 192.168.1.10 (votre ordinateur)
Destination IP     : 8.8.8.8 (DNS Google)
Options            : Aucune
```

**Lecture** :
- C'est un paquet IPv4 avec un en-tête standard de 20 octets
- Il transporte 64 octets de données ICMP (un ping)
- Il peut traverser jusqu'à 64 routeurs
- Il ne doit pas être fragmenté
- Il va de votre IP locale vers le serveur DNS de Google

## Visualisation de l'encapsulation complète

Voici comment un paquet HTTP est encapsulé à travers les couches :

```
┌─────────────────────────────────────────────────────────┐
│                   Trame Ethernet                        │
│ ┌─────────────────────────────────────────────────────┐ │
│ │  En-tête Ethernet (14 octets)                       │ │
│ │  - MAC source, MAC destination, Type                │ │
│ ├─────────────────────────────────────────────────────┤ │
│ │             Paquet IP (20+ octets)                  │ │
│ │  ┌───────────────────────────────────────────────┐  │ │
│ │  │  En-tête IP (20 octets)                       │  │ │
│ │  │  - Version, TTL, Protocol, IPs...             │  │ │
│ │  ├───────────────────────────────────────────────┤  │ │
│ │  │          Segment TCP (20+ octets)             │  │ │
│ │  │  ┌─────────────────────────────────────────┐  │  │ │
│ │  │  │  En-tête TCP (20 octets)                │  │  │ │
│ │  │  │  - Ports, numéros de séquence...        │  │  │ │
│ │  │  ├─────────────────────────────────────────┤  │  │ │
│ │  │  │       Données HTTP                      │  │  │ │
│ │  │  │  "GET / HTTP/1.1..."                    │  │  │ │
│ │  │  └─────────────────────────────────────────┘  │  │ │
│ │  └───────────────────────────────────────────────┘  │ │
│ └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

Chaque couche ajoute son propre en-tête, comme des poupées russes !

## Taille typique d'un paquet IPv4

Sur un réseau Ethernet standard :

```
MTU Ethernet         : 1500 octets (Maximum Transmission Unit)
  - En-tête IP       :   20 octets (minimum)
  - En-tête TCP      :   20 octets (minimum)
  ──────────────────────────────────
  = Données TCP      : 1460 octets (maximum pour une connexion TCP standard)
```

C'est pourquoi, quand vous téléchargez un gros fichier, il est découpé en milliers de paquets de ~1460 octets chacun.

## Les champs les plus importants à retenir

Si vous ne deviez retenir que l'essentiel :

- 🔑 **Version** : 4 pour IPv4
- 🔑 **TTL** : Évite les boucles infinies
- 🔑 **Protocol** : Indique TCP (6), UDP (17) ou ICMP (1)
- 🔑 **Source IP** : D'où vient le paquet
- 🔑 **Destination IP** : Où va le paquet (utilisé pour le routage)
- 🔑 **Total Length** : Taille du paquet
- 🔑 **Identification + Flags + Offset** : Gestion de la fragmentation

## Cas pratique : analyser un paquet avec Wireshark

Si vous capturez un paquet avec Wireshark (nous verrons cet outil en détail plus tard), vous verrez exactement ces champs :

```
Internet Protocol Version 4, Src: 192.168.1.10, Dst: 8.8.8.8
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00
    Total Length: 84
    Identification: 0xd431 (54321)
    Flags: 0x4000, Don't fragment
        0... .... .... .... = Reserved bit: Not set
        .1.. .... .... .... = Don't fragment: Set
        ..0. .... .... .... = More fragments: Not set
    Fragment offset: 0
    Time to live: 64
    Protocol: ICMP (1)
    Header checksum: 0xb861
    Source: 192.168.1.10
    Destination: 8.8.8.8
```

Vous pouvez maintenant comprendre chaque ligne !

## Limites et évolution vers IPv6

L'en-tête IPv4, bien que génial, a quelques limitations :

- ❌ **Espace d'adressage limité** : 32 bits = ~4,3 milliards d'adresses (insuffisant aujourd'hui)
- ❌ **En-tête complexe** : Trop de champs optionnels ralentissent le traitement
- ❌ **Pas de sécurité native** : Aucun chiffrement intégré
- ❌ **Checksum coûteux** : Doit être recalculé à chaque routeur

C'est pourquoi IPv6 a été créé avec :
- 128 bits d'adressage (340 undécillions d'adresses !)
- En-tête simplifié et fixe
- Sécurité intégrée (IPsec)
- Pas de checksum (laissé aux couches supérieures)

Nous explorerons IPv6 en détail dans une section dédiée.

## Résumé visuel

```
┌───────────────────────────────────────────────────────────┐
│            EN-TÊTE IPv4 (20 octets minimum)               │
├───────────────────────────────────────────────────────────┤
│  🏷️  Version (4) + IHL (5)      → Identification du paquet│
│  ⚡ Type of Service              → Priorité / QoS         │
│  📏 Total Length                 → Taille totale          │
│  🔢 Identification               → ID pour fragmentation  │
│  🚩 Flags + Fragment Offset      → Gestion fragments      │
│  ⏱️  Time to Live (TTL)          → Durée de vie           │
│  📦 Protocol                     → TCP/UDP/ICMP           │
│  ✅ Header Checksum              → Vérification intégrité │
│  📤 Source IP Address            → D'où ça vient          │
│  📥 Destination IP Address       → Où ça va (ROUTAGE!)    │
│  ⚙️  Options (facultatives)      → Rarement utilisées     │
├───────────────────────────────────────────────────────────┤
│                    DONNÉES                                │
│         (Segment TCP ou Datagramme UDP)                   │
└───────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ Un paquet IPv4 = **en-tête (20-60 octets)** + **données (variable)**

✅ L'en-tête contient **toutes les informations nécessaires au routage**

✅ Les champs critiques : **Version, TTL, Protocol, adresses IP source/destination**

✅ Le **TTL** empêche les boucles infinies en limitant le nombre de sauts

✅ Le champ **Protocol** indique ce qu'il y a dans les données (TCP=6, UDP=17, ICMP=1)

✅ La **fragmentation** permet de découper les gros paquets (Identification, Flags, Offset)

✅ Le **checksum** vérifie l'intégrité de l'en-tête (pas des données)

✅ La taille maximale pratique sur Ethernet : **1500 octets (MTU)**

## Pour aller plus loin

Maintenant que vous comprenez la structure d'un paquet IPv4, vous êtes prêt à explorer :

- Comment fonctionnent les adresses IP et leur notation
- Comment calculer des sous-réseaux
- Comment les routeurs utilisent ces adresses pour acheminer les paquets
- Comment IPv6 améliore ce design

---

**💡 Astuce pratique** : Pour visualiser de vrais paquets IPv4, installez Wireshark (gratuit) et lancez une capture pendant que vous naviguez sur le web. Vous verrez défiler des milliers de paquets avec tous ces champs remplis. C'est fascinant !

---

*Dans la section suivante, nous allons explorer l'adressage IPv4 en profondeur : classes d'adresses, notation CIDR, et comment interpréter une adresse comme 192.168.1.0/24...*

⏭️ [Adressage IPv4 : classes, notation CIDR](/03-couche-internet/03-adressage-ipv4.md)
