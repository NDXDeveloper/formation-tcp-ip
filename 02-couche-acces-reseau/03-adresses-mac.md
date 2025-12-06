🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.3 Adresses MAC : structure et fonctionnement

## Introduction

Imaginez un immense parking où des milliers de voitures se garent chaque jour. Comment les retrouver ? Grâce à leurs **plaques d'immatriculation** ! Dans le monde des réseaux, chaque carte réseau possède son équivalent : une **adresse MAC** (Media Access Control).

L'adresse MAC est l'**identifiant physique unique** de votre équipement réseau. Contrairement aux adresses IP qui peuvent changer, l'adresse MAC est gravée dans le matériel dès sa fabrication. C'est l'ADN de votre carte réseau !

## Qu'est-ce qu'une adresse MAC ?

### Définition

Une **adresse MAC** (aussi appelée adresse physique ou adresse matérielle) est un identifiant unique de 48 bits (6 octets) attribué à chaque interface réseau :
- Carte Ethernet
- Carte Wi-Fi
- Adaptateur Bluetooth
- Interface de routeur
- Tout équipement connecté à un réseau

**Caractéristique principale** : Elle est (en théorie) **unique au monde** ! Sur les milliards d'appareils connectés, aucun ne devrait avoir la même adresse MAC.

### Analogie : La plaque d'immatriculation

L'adresse MAC, c'est comme une plaque d'immatriculation de voiture :

| Plaque d'immatriculation | Adresse MAC |
|---------------------------|-------------|
| Identifie physiquement la voiture | Identifie physiquement la carte réseau |
| Gravée sur le véhicule | Gravée dans la puce électronique |
| Unique (normalement) | Unique (normalement) |
| Utilisée pour circuler localement | Utilisée pour communiquer localement |
| Ne change pas (sauf modification) | Ne change pas (sauf modification logicielle) |

**Différence importante** : Alors qu'une voiture peut circuler entre différentes villes avec la même plaque, une adresse MAC ne sert qu'au **réseau local** (pas pour router sur Internet).

## Structure et format d'une adresse MAC

### Format de base

Une adresse MAC contient **6 octets** (48 bits), généralement représentés en **hexadécimal** :

```
┌─────────────────────────────────────────────┐
│  AA : BB : CC : DD : EE : FF                │
│  └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘        │
│   Octet1 Octet2 Octet3 Octet4 Octet5 Octet6 │
│                                             │
│  48 bits = 6 octets = 12 chiffres hexa      │
└─────────────────────────────────────────────┘
```

**Hexadécimal** : Chaque chiffre représente 4 bits (0-9 et A-F)
- `0` = 0000
- `F` = 1111
- `A5` = 10100101

### Notations possibles

L'adresse MAC `AA:BB:CC:DD:EE:FF` peut s'écrire de plusieurs façons :

| Format | Exemple | Utilisé par |
|--------|---------|-------------|
| **Deux-points** | `AA:BB:CC:DD:EE:FF` | Linux, Cisco, la plupart des systèmes |
| **Tirets** | `AA-BB-CC-DD-EE-FF` | Windows |
| **Points** | `AABB.CCDD.EEFF` | Cisco (certains équipements) |
| **Sans séparateur** | `AABBCCDDEEFF` | Programmation, bases de données |

**C'est la même adresse**, juste notée différemment !

### Exemple réel

Voici une adresse MAC que vous pourriez voir sur votre ordinateur :

```
00:1A:2B:3C:4D:5E
```

En binaire, cela donne :
```
00000000:00011010:00101011:00111100:01001101:01011110
```

**Nombre total d'adresses possibles** : 2^48 = **281 474 976 710 656** adresses (plus de 281 mille milliards) !

## Structure détaillée : OUI et NIC

Une adresse MAC est divisée en deux parties :

```
┌──────────────────┬──────────────────┐
│  00 : 1A : 2B    │  3C : 4D : 5E    │
│                  │                  │
│  OUI             │  NIC             │
│  (24 bits)       │  (24 bits)       │
│                  │                  │
│  Organizationally│  Network         │
│  Unique          │  Interface       │
│  Identifier      │  Controller      │
│                  │                  │
│  Fabricant       │  Numéro unique   │
│                  │  du fabricant    │
└──────────────────┴──────────────────┘
```

### OUI (Organizationally Unique Identifier)

Les **3 premiers octets** identifient le **fabricant** de la carte réseau.

**Exemples de OUI célèbres** :

| OUI | Fabricant |
|-----|-----------|
| `00:50:56` | VMware |
| `00:0C:29` | VMware (autre gamme) |
| `00:1B:63` | Apple |
| `B8:27:EB` | Raspberry Pi Foundation |
| `DC:A6:32` | Raspberry Pi Trading |
| `00:15:5D` | Microsoft (Hyper-V) |
| `08:00:27` | Oracle VirtualBox |
| `00:E0:4C` | Realtek |
| `00:1E:C9` | TP-Link |

**Utilisation pratique** : En voyant les 3 premiers octets, vous pouvez identifier la marque du fabricant !

**Recherche d'OUI** : Des sites comme [https://maclookup.app/](https://maclookup.app/) permettent d'identifier le fabricant d'une adresse MAC.

### NIC (Network Interface Controller)

Les **3 derniers octets** sont attribués par le fabricant lui-même pour garantir l'unicité de chaque carte qu'il produit.

**Analogie** :
- **OUI** = Le code postal (identifie la ville/fabricant)
- **NIC** = Le numéro de rue (identifie la maison spécifique)

## Les bits spéciaux : U/L et I/G

Le premier octet de l'adresse MAC contient deux bits particuliers qui ont une signification spéciale :

```
Premier octet (exemple: 00)
00000000
│    │
│    └─→ Bit I/G (Individual/Group) - Position 0
└──────→ Bit U/L (Universal/Local) - Position 1
```

### Bit I/G (Individual/Group) - Bit 0

**Position** : Le bit de poids faible du premier octet

**Signification** :
- `0` = **Unicast** (adresse individuelle, pour une seule carte)
- `1` = **Multicast** (adresse de groupe, pour plusieurs cartes)

**Exemple** :
```
00:1A:2B:3C:4D:5E  →  00 = 00000000  →  Bit 0 = 0  →  Unicast
01:00:5E:00:00:01  →  01 = 00000001  →  Bit 0 = 1  →  Multicast
```

### Bit U/L (Universal/Local) - Bit 1

**Position** : Le deuxième bit de poids faible du premier octet

**Signification** :
- `0` = **UAA** (Universally Administered Address) - Adresse globale, attribuée par le fabricant
- `1` = **LAA** (Locally Administered Address) - Adresse locale, configurée manuellement

**Exemple** :
```
00:1A:2B:3C:4D:5E  →  00 = 00000000  →  Bit 1 = 0  →  Globale (fabricant)
02:1A:2B:3C:4D:5E  →  02 = 00000010  →  Bit 1 = 1  →  Locale (modifiée)
```

### Schéma récapitulatif

```
Premier octet : 0 0 0 0 0 0 0 0
                │ │ └─────────┘
                │ │      └─────→ 6 bits normaux
                │ └────────────→ U/L : Universal (0) ou Local (1)
                └──────────────→ I/G : Individual (0) ou Group (1)
```

## Types d'adresses MAC

### 1. Adresse Unicast (la plus courante)

**Définition** : Adresse d'une **seule interface** réseau.

**Exemple** : `00:1A:2B:3C:4D:5E`

**Utilisation** : Communication point à point normale.

```
Ordinateur A                    Ordinateur B
MAC: 00:11:22:33:44:55  →  MAC: AA:BB:CC:DD:EE:FF
      "Salut B!"
```

### 2. Adresse Broadcast

**Définition** : Adresse spéciale qui cible **tous les appareils** du réseau local.

**Valeur** : `FF:FF:FF:FF:FF:FF` (tous les bits à 1)

**Utilisation** : Quand on veut parler à tout le monde.

**Exemple** : Le protocole ARP utilise le broadcast pour demander "Qui a l'adresse IP 192.168.1.10 ?"

```
Ordinateur A
    │
    ├───→ "Qui a l'IP 192.168.1.10 ?"
    │     (destination: FF:FF:FF:FF:FF:FF)
    ↓
💻  💻  💻  💻  💻
└─→ "C'est moi !" (seul celui qui a cette IP répond)
```

**Analogie** : C'est comme crier dans une salle : tout le monde entend, mais seul celui qui se sent concerné répond.

### 3. Adresse Multicast

**Définition** : Adresse d'un **groupe** d'interfaces.

**Identification** : Le bit I/G est à 1 (premier octet impair).

**Plage pour IPv4** : `01:00:5E:XX:XX:XX`

**Utilisation** : Streaming vidéo, IPTV, protocoles de routage...

**Exemple** : Un flux vidéo diffusé à tous les écrans d'un réseau d'entreprise.

```
Serveur vidéo
    │
    ├───→ Flux vers 01:00:5E:01:02:03
    ↓
📺  📺      💻      📺
↑   ↑               ↑
Groupe multicast    Pas inscrit, ne reçoit pas
```

**Analogie** : C'est comme une chaîne de radio : seuls ceux qui sont "branchés" sur cette fréquence reçoivent le message.

## Comment obtenir l'adresse MAC de votre appareil ?

### Sur Windows

```
Commande : ipconfig /all

Résultat :
Carte Ethernet Ethernet :
   Adresse physique . . . . . . . . : 00-1A-2B-3C-4D-5E
```

### Sur Linux / macOS

```
Commande : ifconfig (ou ip link show)

Résultat :
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    link/ether 00:1a:2b:3c:4d:5e
```

### Sur smartphone

**Android** : Paramètres → À propos → État → Adresse MAC Wi-Fi
**iPhone** : Réglages → Général → Informations → Adresse Wi-Fi

## Adresses MAC vs Adresses IP : Différences clés

| Critère | Adresse MAC | Adresse IP |
|---------|-------------|------------|
| **Couche** | Couche 2 (Liaison) | Couche 3 (Internet) |
| **Portée** | Réseau local uniquement | Réseau local ET Internet |
| **Longueur** | 48 bits (6 octets) | IPv4: 32 bits / IPv6: 128 bits |
| **Format** | Hexadécimal (AA:BB:CC:DD:EE:FF) | Décimal pointé (192.168.1.1) |
| **Attribution** | Fabricant (gravée) | Administrateur réseau (DHCP/statique) |
| **Changement** | Fixe (théoriquement) | Change selon le réseau |
| **Unicité** | Mondiale (théoriquement) | Unique sur le réseau |
| **Exemple** | 00:1A:2B:3C:4D:5E | 192.168.1.100 |

### Analogie complète

Imaginez envoyer une lettre à Paris :

```
┌───────────────────────────────────────────┐
│ Destinataire : Jean Dupont                │  ← Couche Application
│ Adresse IP : 12 Rue de la Paix, Paris     │  ← Couche Internet (routage)
│ Adresse MAC : Boîte aux lettres n°A42     │  ← Couche Liaison (livraison locale)
└───────────────────────────────────────────┘
```

- **L'adresse IP** vous amène à Paris, Rue de la Paix (routage longue distance)
- **L'adresse MAC** identifie la boîte aux lettres précise (livraison locale)

**L'adresse MAC change à chaque "saut"** : Quand votre paquet traverse des routeurs, l'adresse IP reste la même, mais l'adresse MAC change à chaque segment réseau !

## Pourquoi 48 bits ?

Vous vous demandez peut-être pourquoi 48 bits et pas 32 ou 64 ?

**Réponse historique** : En 1980, 48 bits semblaient **largement suffisants** :
- 281 billions d'adresses possibles
- Pensé pour des décennies d'expansion

**Mais aujourd'hui** : Avec l'explosion de l'IoT (Internet des Objets), on se rapproche de la pénurie !
- Milliards de smartphones
- Milliards d'objets connectés
- Plusieurs interfaces par appareil

**Solution** : L'IEEE recycle les OUI des fabricants disparus et encourage les adresses LAA (locales) pour certains usages.

## Unicité et conflits

### Théorie : Chaque MAC est unique

En théorie, l'IEEE garantit l'unicité en :
1. Attribuant des OUI uniques aux fabricants
2. Demandant aux fabricants de ne jamais répéter les 3 derniers octets

### Réalité : Des conflits peuvent survenir

**Causes de duplication** :
- ❌ **Contrefaçons** : Équipements clonés avec les mêmes adresses
- ❌ **Erreurs de fabrication** : Très rare, mais possible
- ❌ **Virtualisation** : Les VMs peuvent avoir des MACs similaires
- ❌ **Modification manuelle** : Changement volontaire (spoofing)

**Conséquence** : Si deux appareils sur le même réseau ont la même MAC → **chaos réseau** ! Les paquets arrivent au mauvais destinataire ou se perdent.

**Analogie** : C'est comme avoir deux maisons avec le même numéro dans la même rue : le facteur ne sait plus où livrer !

## MAC Spoofing : Changer son adresse MAC

### Qu'est-ce que c'est ?

Le **MAC spoofing** consiste à modifier l'adresse MAC de sa carte réseau par logiciel.

**Pourquoi le faire ?**

**Usages légitimes** :
- ✅ Tester la sécurité réseau
- ✅ Contourner des restrictions basées sur MAC (ex: hotspot Wi-Fi)
- ✅ Préserver sa vie privée (éviter le tracking)

**Usages malveillants** :
- ❌ Usurper l'identité d'un autre appareil
- ❌ Contourner des contrôles d'accès
- ❌ Attaques réseau

### Comment ça marche ?

Sous Linux :
```bash
sudo ifconfig eth0 down
sudo ifconfig eth0 hw ether 00:11:22:33:44:55
sudo ifconfig eth0 up
```

**Important** : Le changement est temporaire. Au redémarrage, l'adresse MAC originale revient.

### Implications de sécurité

**Mythe** : "Mon réseau est sécurisé par filtrage MAC"
**Réalité** : Le filtrage MAC seul n'est **pas une sécurité robuste** car l'adresse MAC peut être facilement usurpée.

**Vraie sécurité** :
- 🔐 Chiffrement WPA3 pour le Wi-Fi
- 🔐 802.1X pour l'authentification
- 🔐 VPN pour les communications sensibles

## Adresses MAC spéciales et réservées

### Adresses réservées par l'IEEE

| Adresse | Usage |
|---------|-------|
| `FF:FF:FF:FF:FF:FF` | Broadcast (tout le monde) |
| `01:00:5E:XX:XX:XX` | Multicast IPv4 |
| `33:33:XX:XX:XX:XX` | Multicast IPv6 |
| `01:80:C2:00:00:00` | Spanning Tree Protocol (STP) |
| `01:80:C2:00:00:02` | LACP (agrégation de liens) |

### Adresses impossibles

Certaines adresses ne seront jamais attribuées :
- `00:00:00:00:00:00` (adresse nulle, invalide)
- Adresses avec bit U/L à 1 dans les 3 premiers octets (réservées pour usage local)

## Les adresses MAC dans la pratique

### Scénario : Envoi d'un paquet sur le réseau local

Imaginons que l'ordinateur A (MAC: `AA:AA:AA:AA:AA:AA`) veut envoyer des données à l'ordinateur B (MAC: `BB:BB:BB:BB:BB:BB`) via un switch :

```
Étape 1 : A construit la trame Ethernet
┌────────────────────────────────────────┐
│ MAC destination : BB:BB:BB:BB:BB:BB    │
│ MAC source      : AA:AA:AA:AA:AA:AA    │
│ Données         : [paquet IP...]       │
└────────────────────────────────────────┘

Étape 2 : Le switch lit la MAC destination

Étape 3 : Le switch envoie uniquement vers le port où B est connecté

Étape 4 : B reçoit la trame, vérifie la MAC destination
         ↓
    C'est ma MAC ! → Je traite le paquet
    Ce n'est pas ma MAC ! → Je l'ignore
```

### Table CAM du switch

Le switch maintient une **table CAM** (Content Addressable Memory) qui associe :

| Adresse MAC | Port | Âge |
|-------------|------|-----|
| AA:AA:AA:AA:AA:AA | Port 1 | 120s |
| BB:BB:BB:BB:BB:BB | Port 3 | 45s |
| CC:CC:CC:CC:CC:CC | Port 5 | 200s |

**Comment le switch apprend** :
1. Une trame arrive sur le port 1
2. Le switch lit la MAC source : `AA:AA:AA:AA:AA:AA`
3. Il enregistre : "Cette MAC est sur le port 1"
4. Avec le temps, il connaît toutes les MACs du réseau

**Analogie** : C'est comme un standard téléphonique qui apprend quel poste est à quel bureau en écoutant qui appelle.

## Durée de vie et aging

Les entrées dans la table CAM ont une **durée de vie limitée** (généralement 300 secondes / 5 minutes).

**Pourquoi ?**
- Un appareil peut se déconnecter
- Un appareil peut changer de port (laptop déplacé)
- Évite les entrées obsolètes

**Rafraîchissement** : Chaque fois qu'une MAC envoie une trame, son entrée est rafraîchie.

## Points clés à retenir

1. **Identifiant unique** : Chaque carte réseau a une adresse MAC unique de 48 bits.

2. **Structure OUI + NIC** : 3 octets (fabricant) + 3 octets (numéro unique).

3. **Notation** : Format hexadécimal, séparée par `:` ou `-` (AA:BB:CC:DD:EE:FF).

4. **Trois types** :
   - **Unicast** : Une seule interface
   - **Broadcast** : Toutes les interfaces (FF:FF:FF:FF:FF:FF)
   - **Multicast** : Un groupe d'interfaces

5. **Portée locale** : L'adresse MAC ne fonctionne que sur le réseau local, pas sur Internet.

6. **Différente de l'IP** : MAC = couche 2 (physique), IP = couche 3 (logique/réseau).

7. **Modifiable** : Peut être changée par logiciel (MAC spoofing).

8. **Switches intelligents** : Apprennent automatiquement quelle MAC est sur quel port.

---

## Prochaine étape

Maintenant que vous savez comment les appareils s'identifient avec leur adresse MAC, nous allons découvrir comment ces adresses sont utilisées dans les **trames Ethernet**, les enveloppes qui transportent vos données sur le réseau local !

---


⏭️ [Trames Ethernet : format et analyse](/02-couche-acces-reseau/04-trames-ethernet.md)
