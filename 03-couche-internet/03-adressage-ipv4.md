🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.3 Adressage IPv4 : classes, notation CIDR

## Introduction : l'importance de l'adressage

Vous savez maintenant qu'un paquet IPv4 contient deux adresses IP : celle de l'expéditeur et celle du destinataire. Mais comment fonctionnent ces adresses ? Comment sont-elles organisées ? Comment les routeurs savent-ils où envoyer un paquet juste en regardant son adresse ?

L'**adressage IP** est l'un des concepts les plus fondamentaux des réseaux. C'est grâce à lui qu'Internet peut connecter des milliards d'appareils à travers le monde.

**Analogie** : Pensez au système d'adresses postales. Une adresse comme "15 rue Victor Hugo, 75016 Paris, France" est hiérarchique :
- **France** → le pays (réseau global)
- **75016 Paris** → la ville et le code postal (sous-réseau)
- **rue Victor Hugo** → la rue (sous-sous-réseau)
- **15** → le numéro de maison (l'appareil individuel)

Les adresses IP fonctionnent de manière similaire : elles sont hiérarchiques et permettent de localiser un appareil sur le réseau mondial.

## Structure d'une adresse IPv4

Une adresse IPv4 est un **nombre de 32 bits** (32 chiffres binaires).

### Représentation binaire

En binaire, une adresse IPv4 ressemble à ceci :

```
11000000.10101000.00000001.00001010
```

Pas très lisible, n'est-ce pas ? C'est pourquoi on utilise une notation plus humaine.

### Notation décimale pointée

Pour faciliter la lecture, on divise les 32 bits en **4 octets** (4 groupes de 8 bits) et on convertit chaque octet en décimal :

```
Binaire :  11000000 . 10101000 . 00000001 . 00001010
           ↓          ↓          ↓          ↓
Décimal :  192      . 168      . 1        . 10

Adresse IP : 192.168.1.10
```

**Règles** :
- Chaque octet peut aller de **0 à 255** (car 2^8 = 256 valeurs possibles)
- Les octets sont séparés par des **points**

**Exemples d'adresses valides** :
- `8.8.8.8` (DNS de Google)
- `192.168.1.1` (adresse typique d'une box Internet)
- `172.16.0.1`
- `10.0.0.1`

**Exemples d'adresses invalides** :
- `192.168.1.256` ❌ (256 > 255)
- `192.168.1` ❌ (seulement 3 octets)
- `192.168.1.1.1` ❌ (5 octets)

### Combien d'adresses IPv4 existe-t-il ?

Avec 32 bits, on peut créer :

```
2^32 = 4 294 967 296 adresses possibles

Soit environ 4,3 milliards d'adresses
```

Cela semble beaucoup, mais avec la croissance d'Internet, ce n'est plus suffisant aujourd'hui. C'est l'une des raisons de la création d'IPv6.

## Partie réseau vs partie hôte

Une adresse IP est composée de **deux parties** :

```
┌─────────────────────────────────────┐
│  PARTIE RÉSEAU  │  PARTIE HÔTE      │
│  (Network)      │  (Host)           │
└─────────────────────────────────────┘
```

### La partie réseau (Network)

Identifie **le réseau** auquel appartient l'appareil.

**Analogie** : C'est comme le code postal et le nom de la ville dans une adresse postale.

Tous les appareils du même réseau partagent la même partie réseau.

### La partie hôte (Host)

Identifie **l'appareil spécifique** au sein du réseau.

**Analogie** : C'est comme le numéro de rue et le numéro de maison dans une adresse postale.

Chaque appareil du réseau a une partie hôte différente.

### Exemple concret

Prenons deux ordinateurs sur le même réseau local :

```
Ordinateur A : 192.168.1.10
Ordinateur B : 192.168.1.20

Partie réseau : 192.168.1    (identique pour les deux)
Partie hôte   : .10 et .20   (différente pour chaque machine)
```

Ces deux ordinateurs sont sur le **même réseau** (même partie réseau) mais sont des **appareils différents** (parties hôtes différentes).

## Le masque de sous-réseau

**Problème** : Comment savoir où s'arrête la partie réseau et où commence la partie hôte ?

**Solution** : Le **masque de sous-réseau** (subnet mask).

### Qu'est-ce qu'un masque de sous-réseau ?

C'est un nombre de 32 bits (comme une adresse IP) qui indique quels bits appartiennent à la partie réseau.

**Règle simple** :
- Les bits à **1** dans le masque → partie **réseau**
- Les bits à **0** dans le masque → partie **hôte**

### Exemple

```
Adresse IP   : 192.168.1.10
Masque       : 255.255.255.0

En binaire :
Adresse IP   : 11000000.10101000.00000001.00001010
Masque       : 11111111.11111111.11111111.00000000
               └─────────réseau──────────┘└─hôte─┘
```

Ici, les 24 premiers bits (3 premiers octets) sont la partie réseau, et les 8 derniers bits (dernier octet) sont la partie hôte.

**Comment l'utiliser** :
- Partie réseau : `192.168.1` (les 3 premiers octets)
- Partie hôte : `.10` (le dernier octet peut varier de 0 à 255)

### Masques courants

```
255.255.255.0   = /24  → 24 bits de réseau, 8 bits d'hôtes
255.255.0.0     = /16  → 16 bits de réseau, 16 bits d'hôtes
255.0.0.0       = /8   → 8 bits de réseau, 24 bits d'hôtes
255.255.255.128 = /25  → 25 bits de réseau, 7 bits d'hôtes
```

## Les classes d'adresses (système historique)

À l'origine, les adresses IPv4 étaient organisées en **classes** (A, B, C, D, E). Ce système est aujourd'hui obsolète mais important à comprendre historiquement.

### Classe A : les très grands réseaux

**Plage** : `1.0.0.0` à `126.255.255.255`

**Premier bit** : Toujours `0`

**Structure** :
```
┌─────┬───────────────────────────────┐
│  N  │        H     H     H          │  N = Network, H = Host
└─────┴───────────────────────────────┘
  8 bits      24 bits

Masque par défaut : 255.0.0.0 (/8)
```

**Caractéristiques** :
- Premier octet : 1 à 126
- Masque : `255.0.0.0`
- Nombre de réseaux : 126
- Nombre d'hôtes par réseau : 16 777 214 (2^24 - 2)

**Exemple** : `10.0.0.0/8`

**Analogie** : Réservé aux très grandes organisations (entreprises multinationales, gouvernements). Chaque réseau de classe A peut contenir plus de 16 millions d'appareils !

**Note** : `127.x.x.x` est réservé pour le loopback (localhost).

### Classe B : les réseaux moyens

**Plage** : `128.0.0.0` à `191.255.255.255`

**Premiers bits** : Toujours `10`

**Structure** :
```
┌─────────────┬─────────────────────┐
│    N    N   │      H      H       │
└─────────────┴─────────────────────┘
   16 bits         16 bits

Masque par défaut : 255.255.0.0 (/16)
```

**Caractéristiques** :
- Premier octet : 128 à 191
- Masque : `255.255.0.0`
- Nombre de réseaux : 16 384
- Nombre d'hôtes par réseau : 65 534 (2^16 - 2)

**Exemple** : `172.16.0.0/16`

**Analogie** : Pour les organisations de taille moyenne (universités, grandes entreprises). Chaque réseau peut contenir environ 65 000 appareils.

### Classe C : les petits réseaux

**Plage** : `192.0.0.0` à `223.255.255.255`

**Premiers bits** : Toujours `110`

**Structure** :
```
┌───────────────────────┬───────┐
│    N    N    N        │   H   │
└───────────────────────┴───────┘
      24 bits            8 bits

Masque par défaut : 255.255.255.0 (/24)
```

**Caractéristiques** :
- Premier octet : 192 à 223
- Masque : `255.255.255.0`
- Nombre de réseaux : 2 097 152
- Nombre d'hôtes par réseau : 254 (2^8 - 2)

**Exemple** : `192.168.1.0/24`

**Analogie** : Pour les petites organisations ou réseaux domestiques. Chaque réseau peut contenir jusqu'à 254 appareils (largement suffisant pour une maison ou un petit bureau).

### Classe D : multicast

**Plage** : `224.0.0.0` à `239.255.255.255`

**Premiers bits** : Toujours `1110`

**Usage** : Réservée pour le **multicast** (diffusion vers un groupe d'appareils).

**Exemple** : Streaming vidéo en direct vers plusieurs destinataires simultanément.

### Classe E : réservée

**Plage** : `240.0.0.0` à `255.255.255.255`

**Premiers bits** : Toujours `1111`

**Usage** : Réservée pour des usages expérimentaux. Non utilisée sur Internet.

### Tableau récapitulatif des classes

```
┌─────────┬────────────────────┬──────────────┬────────────────┬─────────────┐
│ Classe  │ Premier octet      │ Masque défaut│ Bits réseau    │ Bits hôte   │
├─────────┼────────────────────┼──────────────┼────────────────┼─────────────┤
│    A    │   1 - 126          │ 255.0.0.0    │      8         │     24      │
│    B    │   128 - 191        │ 255.255.0.0  │      16        │     16      │
│    C    │   192 - 223        │ 255.255.255.0│      24        │      8      │
│    D    │   224 - 239        │ N/A          │ Multicast      │     N/A     │
│    E    │   240 - 255        │ N/A          │ Réservée       │     N/A     │
└─────────┴────────────────────┴──────────────┴────────────────┴─────────────┘
```

### Pourquoi le système de classes est-il obsolète ?

Le système de classes était trop **rigide** :

❌ **Gaspillage d'adresses** : Une entreprise ayant besoin de 300 adresses devait prendre une classe B (65 534 adresses), gaspillant 65 234 adresses !

❌ **Manque de flexibilité** : Impossible d'avoir un réseau avec, par exemple, 500 adresses (entre classe C et B)

❌ **Épuisement rapide** : Les adresses IPv4 s'épuisaient trop vite

**Solution** : La notation **CIDR** (Classless Inter-Domain Routing)

## La notation CIDR : l'adressage moderne

Introduite en 1993, **CIDR** (prononcé "cider") a remplacé le système de classes.

### Principe de CIDR

Au lieu d'utiliser des classes fixes, CIDR permet de **spécifier exactement combien de bits sont utilisés pour la partie réseau**.

### Notation CIDR : /XX

On écrit l'adresse IP suivie d'un **slash** et du **nombre de bits de la partie réseau** :

```
192.168.1.0/24
            └─ 24 bits de réseau, donc 8 bits pour les hôtes
```

**Lecture** : "192.168.1.0 slash 24"

### Conversion CIDR ↔ Masque de sous-réseau

```
/24 = 255.255.255.0     (24 bits à 1)
/16 = 255.255.0.0       (16 bits à 1)
/8  = 255.0.0.0         (8 bits à 1)
/25 = 255.255.255.128   (25 bits à 1)
/30 = 255.255.255.252   (30 bits à 1)
```

### Exemples de notation CIDR

**Exemple 1** : `192.168.1.0/24`
```
Adresse   : 192.168.1.0
Masque    : 255.255.255.0
Réseau    : 192.168.1.0
Plage     : 192.168.1.0 à 192.168.1.255
Hôtes     : 192.168.1.1 à 192.168.1.254 (254 adresses utilisables)
```

**Exemple 2** : `10.0.0.0/8`
```
Adresse   : 10.0.0.0
Masque    : 255.0.0.0
Réseau    : 10.0.0.0
Plage     : 10.0.0.0 à 10.255.255.255
Hôtes     : Plus de 16 millions d'adresses !
```

**Exemple 3** : `172.16.50.0/23`
```
Adresse   : 172.16.50.0
Masque    : 255.255.254.0
Réseau    : 172.16.50.0
Plage     : 172.16.50.0 à 172.16.51.255
Hôtes     : 510 adresses utilisables
```

### Calcul du nombre d'hôtes

**Formule** :

```
Nombre d'hôtes = 2^(32 - préfixe) - 2

Le "-2" vient du fait qu'on retire :
- L'adresse du réseau (tous les bits hôtes à 0)
- L'adresse de broadcast (tous les bits hôtes à 1)
```

**Exemples** :

```
/24 → 2^(32-24) - 2 = 2^8 - 2 = 256 - 2 = 254 hôtes
/25 → 2^(32-25) - 2 = 2^7 - 2 = 128 - 2 = 126 hôtes
/26 → 2^(32-26) - 2 = 2^6 - 2 = 64 - 2 = 62 hôtes
/27 → 2^(32-27) - 2 = 2^5 - 2 = 32 - 2 = 30 hôtes
/28 → 2^(32-28) - 2 = 2^4 - 2 = 16 - 2 = 14 hôtes
/29 → 2^(32-29) - 2 = 2^3 - 2 = 8 - 2 = 6 hôtes
/30 → 2^(32-30) - 2 = 2^2 - 2 = 4 - 2 = 2 hôtes
```

### Cas d'usage du /30

Le `/30` est particulièrement intéressant pour les **liaisons point-à-point** entre routeurs :

```
Réseau : 10.0.0.0/30

Adresse réseau    : 10.0.0.0   (non utilisable)
Routeur A         : 10.0.0.1   ✅
Routeur B         : 10.0.0.2   ✅
Adresse broadcast : 10.0.0.3   (non utilisable)

Total : 2 adresses utilisables, parfait pour 2 routeurs !
```

## Comprendre l'adresse réseau et le broadcast

### Adresse réseau

C'est l'adresse où **tous les bits de la partie hôte sont à 0**.

**Rôle** : Identifie le réseau lui-même (utilisée dans les tables de routage).

**Exemple** :
```
Réseau 192.168.1.0/24
Adresse réseau : 192.168.1.0

Binaire : 11000000.10101000.00000001.00000000
                                      └─hôte à 0
```

### Adresse de broadcast

C'est l'adresse où **tous les bits de la partie hôte sont à 1**.

**Rôle** : Envoyer un message à **tous** les appareils du réseau simultanément.

**Exemple** :
```
Réseau 192.168.1.0/24
Adresse broadcast : 192.168.1.255

Binaire : 11000000.10101000.00000001.11111111
                                      └─hôte à 1
```

**Utilisation pratique** : Quand votre ordinateur se connecte à un réseau, il envoie un message en broadcast pour demander "Qui est le serveur DHCP ?" (requête DHCP Discover).

### Récapitulatif visuel

```
Réseau 192.168.1.0/24

192.168.1.0     ← Adresse réseau (non assignable)
192.168.1.1     ← Première adresse utilisable (souvent la box/routeur)
192.168.1.2     ← Deuxième adresse utilisable
  ...
192.168.1.254   ← Dernière adresse utilisable
192.168.1.255   ← Adresse de broadcast (non assignable)

Total : 254 adresses utilisables pour des appareils
```

## Tableau de référence CIDR

Voici un tableau pratique pour les préfixes les plus courants :

```
┌────────┬─────────────────┬───────────────┬───────────────────┐
│ CIDR   │ Masque          │ Bits hôtes    │ Nb d'hôtes        │
├────────┼─────────────────┼───────────────┼───────────────────┤
│ /8     │ 255.0.0.0       │ 24            │ 16 777 214        │
│ /16    │ 255.255.0.0     │ 16            │ 65 534            │
│ /24    │ 255.255.255.0   │ 8             │ 254               │
│ /25    │ 255.255.255.128 │ 7             │ 126               │
│ /26    │ 255.255.255.192 │ 6             │ 62                │
│ /27    │ 255.255.255.224 │ 5             │ 30                │
│ /28    │ 255.255.255.240 │ 4             │ 14                │
│ /29    │ 255.255.255.248 │ 3             │ 6                 │
│ /30    │ 255.255.255.252 │ 2             │ 2                 │
│ /31    │ 255.255.255.254 │ 1             │ 2 (cas spécial)   │
│ /32    │ 255.255.255.255 │ 0             │ 1 (hôte unique)   │
└────────┴─────────────────┴───────────────┴───────────────────┘
```

### Cas spéciaux

**Le /32** : Représente une **adresse unique** (aucun bit pour les hôtes). Utilisé pour spécifier un seul appareil :
```
192.168.1.10/32 → Uniquement cette adresse
```

**Le /31** : Introduit par RFC 3021 pour les **liaisons point-à-point**. Pas d'adresse réseau ni de broadcast, les 2 adresses sont utilisables.

## Exemples pratiques d'adressage

### Exemple 1 : Réseau domestique typique

```
Réseau : 192.168.1.0/24

Box Internet      : 192.168.1.1
Ordinateur PC     : 192.168.1.10
Smartphone        : 192.168.1.20
Tablette          : 192.168.1.30
Smart TV          : 192.168.1.40
Imprimante        : 192.168.1.50

Tous ces appareils peuvent communiquer directement
car ils partagent la même partie réseau (192.168.1)
```

### Exemple 2 : Entreprise avec plusieurs services

```
Réseau informatique : 10.10.0.0/24
  - Serveurs : 10.10.0.10 à 10.10.0.50
  - Postes de travail : 10.10.0.100 à 10.10.0.200

Réseau téléphonie VoIP : 10.20.0.0/24
  - Téléphones IP : 10.20.0.1 à 10.20.0.254

Réseau invités : 10.30.0.0/24
  - Appareils invités : 10.30.0.1 à 10.30.0.254

Ces trois réseaux sont séparés et nécessitent un routeur
pour communiquer entre eux.
```

### Exemple 3 : Sous-division d'un réseau /24

Vous avez un réseau `192.168.1.0/24` et voulez le diviser en 4 sous-réseaux égaux :

```
Réseau original : 192.168.1.0/24 (254 hôtes)
                  ↓ Division en 4 ↓
Sous-réseau 1 : 192.168.1.0/26   (62 hôtes chacun)
Sous-réseau 2 : 192.168.1.64/26
Sous-réseau 3 : 192.168.1.128/26
Sous-réseau 4 : 192.168.1.192/26
```

*Nous verrons le subnetting (découpage en sous-réseaux) en détail dans la section suivante.*

## Adresses spéciales à connaître

### 0.0.0.0

**Signification** : "Cette machine" ou "toute adresse"

**Usage** :
- Dans une table de routage : route par défaut
- Dans un bind de serveur : écouter sur toutes les interfaces

### 127.0.0.0/8 (Loopback)

**Plage** : `127.0.0.1` à `127.255.255.254`

**Usage** : Boucle locale (l'ordinateur communique avec lui-même)

**Exemple** :
```
ping 127.0.0.1 → Teste la pile TCP/IP de votre machine

127.0.0.1 = localhost
```

**Analogie** : C'est comme s'envoyer une lettre à soi-même.

### 255.255.255.255 (Broadcast limité)

**Signification** : Broadcast sur le réseau local uniquement

**Usage** : Diffusion limitée au segment réseau local (ne traverse pas les routeurs)

### 169.254.0.0/16 (APIPA)

**Plage** : `169.254.0.1` à `169.254.255.254`

**Signification** : Auto-configuration sans DHCP (APIPA - Automatic Private IP Addressing)

**Quand ça apparaît** : Votre ordinateur ne trouve pas de serveur DHCP, il s'attribue automatiquement une adresse dans cette plage.

**Symptôme** : Si vous voyez `169.254.x.x` sur votre PC, c'est souvent signe d'un problème de connexion réseau !

## Comment déterminer si deux adresses sont sur le même réseau ?

**Méthode** : Appliquer le masque avec une opération **ET logique** (AND) sur les deux adresses.

### Exemple

```
Adresse A : 192.168.1.10
Adresse B : 192.168.1.50
Masque    : 255.255.255.0

Étape 1 : Appliquer le masque à A
192.168.1.10 AND 255.255.255.0 = 192.168.1.0

Étape 2 : Appliquer le masque à B
192.168.1.50 AND 255.255.255.0 = 192.168.1.0

Résultat : 192.168.1.0 = 192.168.1.0
→ Même réseau ! ✅
```

### Contre-exemple

```
Adresse A : 192.168.1.10
Adresse B : 192.168.2.10
Masque    : 255.255.255.0

192.168.1.10 AND 255.255.255.0 = 192.168.1.0
192.168.2.10 AND 255.255.255.0 = 192.168.2.0

Résultat : 192.168.1.0 ≠ 192.168.2.0
→ Réseaux différents ! ❌ (besoin d'un routeur pour communiquer)
```

## Visualisation complète d'un réseau

```
Réseau : 192.168.10.0/24

┌──────────────────────────────────────────────────────────┐
│                  Réseau 192.168.10.0/24                  │
│                  Masque 255.255.255.0                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  192.168.10.0      ← Adresse réseau                      │
│                                                          │
│  192.168.10.1      ← Routeur / Box (gateway)             │
│  192.168.10.2      ← Serveur web                         │
│  192.168.10.10     ← PC Bureau 1                         │
│  192.168.10.11     ← PC Bureau 2                         │
│  192.168.10.20     ← Smartphone                          │
│  192.168.10.100    ← Imprimante                          │
│  ...                                                     │
│  192.168.10.254    ← Dernière adresse utilisable         │
│                                                          │
│  192.168.10.255    ← Adresse broadcast                   │
│                                                          │
│  Total : 254 adresses utilisables                        │
└──────────────────────────────────────────────────────────┘
```

## Conseils pratiques

### 🎯 Pour votre réseau domestique

La plupart des box utilisent :
- `192.168.1.0/24` (Orange, SFR)
- `192.168.0.0/24` (Bouygues)
- `192.168.178.0/24` (anciennes Freebox)
- `192.168.1.0/24` avec la box en `192.168.1.254` (Free Révolution)

### 🎯 Pour mémoriser les masques courants

```
/8  → 255.0.0.0       → "Un octet"
/16 → 255.255.0.0     → "Deux octets"
/24 → 255.255.255.0   → "Trois octets" (le plus courant)
```

Pour les autres, retenez la formule :
```
/25 → 255.255.255.128 (128 = 2^7)
/26 → 255.255.255.192 (192 = 128+64 = 2^7+2^6)
/27 → 255.255.255.224 (224 = 128+64+32)
```

### 🎯 Astuces de diagnostic

```bash
# Voir votre adresse IP (Linux/Mac)
ip addr show
# ou
ifconfig

# Voir votre adresse IP (Windows)
ipconfig

# Exemple de sortie
eth0: 192.168.1.10/24
      ↑ votre IP    ↑ notation CIDR
```

## Résumé visuel

```
┌──────────────────────────────────────────────────────────────┐
│           ADRESSAGE IPv4 - CONCEPTS CLÉS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  📍 32 bits → 4 octets → notation décimale pointée           │
│     11000000.10101000.00000001.00001010 = 192.168.1.10       │
│                                                              │
│  🔀 Deux parties : RÉSEAU + HÔTE                             │
│     192.168.1 . 10                                           │
│     └─réseau─┘ └hôte┘                                        │
│                                                              │
│  😷 Masque de sous-réseau : sépare réseau et hôte            │
│     255.255.255.0 = /24                                      │
│                                                              │
│  📊 Classes (historique) : A, B, C, D, E                     │
│     → Remplacées par CIDR (flexible)                         │
│                                                              │
│  🎯 Notation CIDR : 192.168.1.0/24                           │
│     → 24 bits de réseau, 8 bits d'hôtes                      │
│     → 2^8 - 2 = 254 hôtes utilisables                        │
│                                                              │
│  🚫 Adresses spéciales :                                     │
│     - .0 = adresse réseau                                    │
│     - .255 = broadcast                                       │
│     - 127.x.x.x = loopback (localhost)                       │
│     - 169.254.x.x = APIPA (problème DHCP)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ Une adresse IPv4 = **32 bits** = **4 octets** (0-255 chacun)

✅ Notation décimale pointée : **192.168.1.10**

✅ Deux parties : **RÉSEAU** (identifie le réseau) + **HÔTE** (identifie l'appareil)

✅ Le **masque de sous-réseau** sépare réseau et hôte

✅ Notation **CIDR** (/24, /16...) = nombre de bits de la partie réseau

✅ Formule : **2^(32-préfixe) - 2** = nombre d'hôtes

✅ **Adresse réseau** (hôtes à 0) et **broadcast** (hôtes à 1) ne sont pas assignables

✅ `/24` est le plus courant pour les réseaux locaux (254 hôtes)

✅ Classes A/B/C sont **obsolètes**, CIDR est la norme moderne

✅ `127.0.0.1` = **localhost** (votre propre machine)

## Pour aller plus loin

Maintenant que vous maîtrisez l'adressage IPv4 de base, vous êtes prêt pour :

- Le **subnetting** : découper un réseau en sous-réseaux plus petits
- Les **adresses privées** vs publiques (RFC 1918)
- Le **NAT** : comment traduire les adresses
- Et bien plus encore !

---

**💡 Testez vos connaissances** : Ouvrez un terminal et tapez `ipconfig` (Windows) ou `ip addr` (Linux/Mac). Identifiez votre adresse IP, votre masque (ou préfixe CIDR), et calculez combien d'appareils peuvent être sur votre réseau !

---

*Dans la section suivante, nous allons apprendre à découper des réseaux en sous-réseaux (subnetting), une compétence essentielle pour tout administrateur réseau...*

⏭️ [Sous-réseaux (subnetting) : calculs et conception](/03-couche-internet/04-sous-reseaux.md)
