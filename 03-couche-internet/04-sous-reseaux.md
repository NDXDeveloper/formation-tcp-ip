🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.4 Sous-réseaux (subnetting) : calculs et conception

## Introduction : pourquoi diviser un réseau ?

Imaginez que vous gérez un immeuble de 250 appartements. Vous pourriez avoir une seule boîte aux lettres géante pour tout le monde, mais ce serait le chaos ! À la place, vous organisez : un étage par boîte aux lettres, puis des casiers individuels par appartement.

Le **subnetting** (découpage en sous-réseaux) fait exactement cela avec les adresses IP. Au lieu d'avoir un grand réseau unique avec tous les appareils mélangés, on le **divise en sous-réseaux plus petits** pour mieux organiser, sécuriser et gérer le réseau.

**Définition simple** : Le subnetting consiste à emprunter des bits de la partie hôte pour créer une nouvelle partie "sous-réseau", permettant ainsi de diviser un réseau en plusieurs réseaux plus petits.

## Pourquoi faire du subnetting ?

### 1. Organisation et séparation logique

Dans une entreprise, vous voulez séparer :
- Le réseau des comptables
- Le réseau des informaticiens
- Le réseau des invités
- Le réseau des imprimantes

**Sans subnetting** : Tous les appareils sur le même réseau, impossible de contrôler qui accède à quoi.

**Avec subnetting** : Chaque département a son propre sous-réseau.

```
Réseau de l'entreprise : 10.0.0.0/16
    ↓ Division en sous-réseaux ↓

Comptabilité    : 10.0.1.0/24
Informatique    : 10.0.2.0/24
Marketing       : 10.0.3.0/24
Imprimantes     : 10.0.4.0/24
Invités         : 10.0.5.0/24
```

### 2. Sécurité

Les sous-réseaux permettent d'appliquer des **règles de sécurité différentes** :
- Le réseau invités ne peut pas accéder aux serveurs
- Les imprimantes ne peuvent pas accéder à Internet
- Seul le service IT peut accéder aux serveurs de production

### 3. Performance

Un réseau avec 1000 appareils génère beaucoup de **trafic broadcast** (messages envoyés à tous).

**Problème** : Les broadcasts ralentissent tout le réseau.

**Solution** : Diviser en 10 sous-réseaux de 100 appareils réduit considérablement le trafic broadcast.

**Analogie** : C'est comme diviser une grande salle de classe bruyante en petits groupes de travail. Chaque groupe peut travailler sans être dérangé par les autres.

### 4. Optimisation de l'utilisation des adresses

Vous avez un réseau `/24` (254 hôtes) mais seulement 3 sites :
- Site A : 100 ordinateurs
- Site B : 50 ordinateurs
- Site C : 30 ordinateurs

Sans subnetting, vous gaspilleriez des adresses. Avec subnetting, vous dimensionnez chaque sous-réseau précisément.

### 5. Gestion du routage

Les routeurs utilisent les sous-réseaux pour **optimiser les tables de routage** :
- Au lieu de connaître 1000 routes individuelles
- Ils connaissent 10 routes de sous-réseaux

**Résultat** : Routage plus rapide et tables plus petites.

## Concept fondamental : emprunter des bits

### La transformation magique

Rappelez-vous : une adresse IP a deux parties :

```
┌─────────────────┬─────────────────┐
│  PARTIE RÉSEAU  │  PARTIE HÔTE    │
└─────────────────┴─────────────────┘
```

Le subnetting consiste à **emprunter des bits à la partie hôte** pour créer une **partie sous-réseau** :

```
┌─────────────────┬──────────────┬──────────────┐
│  PARTIE RÉSEAU  │  SOUS-RÉSEAU │  HÔTE        │
└─────────────────┴──────────────┴──────────────┘
```

**Analogie** : Vous avez un code postal (réseau), vous ajoutez un numéro de rue (sous-réseau), puis un numéro d'appartement (hôte).

### Exemple visuel

Prenons un réseau `192.168.1.0/24` :

```
AVANT le subnetting :
192.168.1.0/24
└─réseau──┘└hôte┘  → 254 hôtes possibles

APRÈS le subnetting en /26 :
192.168.1.0/26
└─réseau──┘└SR┘└H┘  → 4 sous-réseaux de 62 hôtes chacun

SR = Sous-Réseau (2 bits empruntés)
H = Hôte (6 bits restants)
```

## Les règles d'or du subnetting

### Règle 1 : Puissances de 2

Le nombre de sous-réseaux et d'hôtes est toujours une **puissance de 2** :

```
1 bit emprunté  = 2^1 = 2 sous-réseaux
2 bits empruntés = 2^2 = 4 sous-réseaux
3 bits empruntés = 2^3 = 8 sous-réseaux
4 bits empruntés = 2^4 = 16 sous-réseaux
...
```

### Règle 2 : Le compromis

Plus vous créez de sous-réseaux, **moins il y a d'hôtes par sous-réseau** :

```
Réseau de départ : 192.168.1.0/24 (254 hôtes)

/25 → 2 sous-réseaux de 126 hôtes chacun
/26 → 4 sous-réseaux de 62 hôtes chacun
/27 → 8 sous-réseaux de 30 hôtes chacun
/28 → 16 sous-réseaux de 14 hôtes chacun
```

**Analogie** : Diviser un gâteau en plus de parts = des parts plus petites.

### Règle 3 : Les formules essentielles

```
Nombre de sous-réseaux = 2^(bits empruntés)
Nombre d'hôtes par sous-réseau = 2^(bits d'hôtes restants) - 2
Taille d'un sous-réseau = 2^(bits d'hôtes restants)
```

## Méthode 1 : Division d'un /24 (la plus courante)

### Exemple 1 : Diviser en 2 sous-réseaux

**Besoin** : Diviser `192.168.1.0/24` en **2 sous-réseaux égaux**.

#### Étape 1 : Calculer combien de bits emprunter

```
2 sous-réseaux = 2^1
→ Il faut emprunter 1 bit
```

#### Étape 2 : Calculer le nouveau masque

```
Original : /24 (255.255.255.0)
+ 1 bit emprunté = /25 (255.255.255.128)
```

**Comment obtenir 128 ?**
```
Dernier octet du masque :
1000 0000 (en binaire) = 128 (en décimal)
└─ 1 bit à 1, les 7 autres à 0
```

#### Étape 3 : Calculer la taille de chaque sous-réseau

```
Bits d'hôtes restants : 32 - 25 = 7 bits
Taille du bloc = 2^7 = 128 adresses
```

#### Étape 4 : Énumérer les sous-réseaux

```
Sous-réseau 1 : 192.168.1.0/25
  Plage : 192.168.1.0 à 192.168.1.127
  Première IP utilisable : 192.168.1.1
  Dernière IP utilisable : 192.168.1.126
  Broadcast : 192.168.1.127

Sous-réseau 2 : 192.168.1.128/25
  Plage : 192.168.1.128 à 192.168.1.255
  Première IP utilisable : 192.168.1.129
  Dernière IP utilisable : 192.168.1.254
  Broadcast : 192.168.1.255
```

#### Visualisation

```
┌─────────────────────────────────────────────┐
│        192.168.1.0/24 (254 hôtes)           │
├──────────────────────┬──────────────────────┤
│ 192.168.1.0/25       │ 192.168.1.128/25     │
│ (126 hôtes)          │ (126 hôtes)          │
│                      │                      │
│ .0 à .127            │ .128 à .255          │
└──────────────────────┴──────────────────────┘
```

### Exemple 2 : Diviser en 4 sous-réseaux

**Besoin** : Diviser `192.168.1.0/24` en **4 sous-réseaux égaux**.

#### Étape 1 : Bits à emprunter

```
4 sous-réseaux = 2^2
→ Il faut emprunter 2 bits
```

#### Étape 2 : Nouveau masque

```
Original : /24
+ 2 bits = /26 (255.255.255.192)

Dernier octet : 1100 0000 = 192
```

#### Étape 3 : Taille des blocs

```
Bits d'hôtes : 32 - 26 = 6 bits
Taille du bloc = 2^6 = 64 adresses
```

#### Étape 4 : Les 4 sous-réseaux

```
Sous-réseau 1 : 192.168.1.0/26
  Plage : .0 à .63
  Utilisables : .1 à .62 (62 hôtes)
  Broadcast : .63

Sous-réseau 2 : 192.168.1.64/26
  Plage : .64 à .127
  Utilisables : .65 à .126 (62 hôtes)
  Broadcast : .127

Sous-réseau 3 : 192.168.1.128/26
  Plage : .128 à .191
  Utilisables : .129 à .190 (62 hôtes)
  Broadcast : .191

Sous-réseau 4 : 192.168.1.192/26
  Plage : .192 à .255
  Utilisables : .193 à .254 (62 hôtes)
  Broadcast : .255
```

**Astuce** : Les sous-réseaux commencent toujours à des multiples de la taille du bloc (0, 64, 128, 192).

#### Visualisation

```
┌─────────────────────────────────────────────────────────┐
│           192.168.1.0/24 (254 hôtes)                    │
├─────────────┬─────────────┬─────────────┬───────────────┤
│ .0/26       │ .64/26      │ .128/26     │ .192/26       │
│ (62 hôtes)  │ (62 hôtes)  │ (62 hôtes)  │ (62 hôtes)    │
└─────────────┴─────────────┴─────────────┴───────────────┘
```

### Exemple 3 : Diviser en 8 sous-réseaux

**Besoin** : Diviser `192.168.1.0/24` en **8 sous-réseaux égaux**.

```
8 = 2^3 → 3 bits empruntés
Nouveau masque : /27 (255.255.255.224)
Taille du bloc : 2^5 = 32 adresses
Hôtes par sous-réseau : 32 - 2 = 30

Les 8 sous-réseaux :
1. 192.168.1.0/27     (.0 à .31)
2. 192.168.1.32/27    (.32 à .63)
3. 192.168.1.64/27    (.64 à .95)
4. 192.168.1.96/27    (.96 à .127)
5. 192.168.1.128/27   (.128 à .159)
6. 192.168.1.160/27   (.160 à .191)
7. 192.168.1.192/27   (.192 à .223)
8. 192.168.1.224/27   (.224 à .255)
```

## Tableau récapitulatif pour un /24

Voici un tableau pratique pour diviser rapidement un réseau `/24` :

```
┌────────────┬──────────┬────────────┬─────────────┬──────────────────────┐
│ Sous-      │ Bits     │ Nouveau    │ Hôtes par   │ Taille du bloc       │
│ réseaux    │ empruntés│ masque     │ sous-réseau │ (saut entre SR)      │
├────────────┼──────────┼────────────┼─────────────┼──────────────────────┤
│ 2          │ 1        │ /25 (.128) │ 126         │ 128                  │
│ 4          │ 2        │ /26 (.192) │ 62          │ 64                   │
│ 8          │ 3        │ /27 (.224) │ 30          │ 32                   │
│ 16         │ 4        │ /28 (.240) │ 14          │ 16                   │
│ 32         │ 5        │ /29 (.248) │ 6           │ 8                    │
│ 64         │ 6        │ /30 (.252) │ 2           │ 4                    │
└────────────┴──────────┴────────────┴─────────────┴──────────────────────┘
```

**Utilisation** : Vous voulez 8 sous-réseaux ?
- Regardez la ligne "8"
- Nouveau masque : /27
- Incrémentez de 32 en 32 : .0, .32, .64, .96, etc.

## Méthode 2 : Subnetting d'autres classes

### Diviser un réseau /16

**Exemple** : Diviser `172.16.0.0/16` en 256 sous-réseaux.

```
256 sous-réseaux = 2^8
→ 8 bits à emprunter

Nouveau masque : /24 (255.255.255.0)

Les sous-réseaux :
172.16.0.0/24
172.16.1.0/24
172.16.2.0/24
...
172.16.255.0/24

Total : 256 sous-réseaux de 254 hôtes chacun
```

**Visualisation** :
```
172.16.0.0/16 original
    ↓
172.16.[0-255].0/24
      └─ Cette partie varie pour chaque sous-réseau
```

### Diviser un réseau /8

**Exemple** : Diviser `10.0.0.0/8` en 256 sous-réseaux.

```
256 = 2^8 → 8 bits empruntés
Nouveau masque : /16

Les sous-réseaux :
10.0.0.0/16
10.1.0.0/16
10.2.0.0/16
...
10.255.0.0/16

Chaque sous-réseau : 65 534 hôtes
```

## Méthode rapide : le "tableau magique"

Voici une méthode ultra-rapide pour calculer de tête :

### Tableau des valeurs de masque

```
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ /24 │ /25 │ /26 │ /27 │ /28 │ /29 │ /30 │ /31 │ /32 │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  0  │ 128 │ 192 │ 224 │ 240 │ 248 │ 252 │ 254 │ 255 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Taille du bloc :
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ 256 │ 128 │  64 │  32 │  16 │   8 │   4 │   2 │   1 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Nombre d'hôtes :
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ 254 │ 126 │  62 │  30 │  14 │   6 │   2 │   2 │   1 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
```

**Exemple d'utilisation** :

Question : Quels sont les sous-réseaux de `192.168.1.0/26` ?

1. Regardez /26 : masque = `.192`, bloc = `64`
2. Comptez par blocs de 64 : `.0`, `.64`, `.128`, `.192`
3. Voilà vos 4 sous-réseaux !

## Calcul à partir du nombre d'hôtes nécessaires

### Méthode inverse : combien de bits pour X hôtes ?

**Question** : J'ai besoin d'un sous-réseau pour 50 ordinateurs. Quel masque utiliser ?

#### Étape 1 : Trouver la puissance de 2 supérieure

```
50 hôtes
+ 2 (adresse réseau + broadcast)
= 52 adresses nécessaires minimum

Puissances de 2 :
32 (2^5) = trop petit ❌
64 (2^6) = OK ! ✅

→ Il faut 6 bits pour les hôtes
```

#### Étape 2 : Calculer le masque

```
32 bits totaux - 6 bits d'hôtes = 26 bits de réseau
→ Masque : /26
```

**Réponse** : Utilisez un `/26` (62 hôtes utilisables).

### Tableau de référence rapide

```
┌──────────────────┬──────────────┬──────────────────────┐
│ Hôtes nécessaires│ Bits hôtes   │ Masque (pour un /24) │
├──────────────────┼──────────────┼──────────────────────┤
│ 2-6              │ 3            │ /29                  │
│ 7-14             │ 4            │ /28                  │
│ 15-30            │ 5            │ /27                  │
│ 31-62            │ 6            │ /26                  │
│ 63-126           │ 7            │ /25                  │
│ 127-254          │ 8            │ /24                  │
└──────────────────┴──────────────┴──────────────────────┘
```

## VLSM : masques de longueur variable

### Qu'est-ce que VLSM ?

**VLSM** (Variable Length Subnet Mask) permet d'utiliser **différentes tailles de sous-réseaux** au sein d'un même réseau.

**Avantage** : Optimisation maximale de l'utilisation des adresses.

### Exemple pratique

Votre entreprise a `192.168.1.0/24` et ces besoins :

```
Site A : 100 ordinateurs
Site B : 50 ordinateurs
Site C : 20 ordinateurs
Liaison routeur-routeur : 2 adresses
```

#### Sans VLSM (gaspillage)

```
Site A : 192.168.1.0/25   (126 hôtes) → gaspille 26 adresses
Site B : 192.168.1.128/25 (126 hôtes) → gaspille 76 adresses
Site C : impossible, plus d'adresses !
```

#### Avec VLSM (optimisé)

**Règle** : Commencer par le plus gros besoin.

```
1. Site A (100 hôtes) :
   Besoin : 2^7 = 128 adresses (102 + réseau + broadcast)
   → 192.168.1.0/25 (126 hôtes utilisables) ✅

2. Site B (50 hôtes) :
   Besoin : 2^6 = 64 adresses
   → 192.168.1.128/26 (62 hôtes utilisables) ✅

3. Site C (20 hôtes) :
   Besoin : 2^5 = 32 adresses
   → 192.168.1.192/27 (30 hôtes utilisables) ✅

4. Liaison routeurs (2 hôtes) :
   → 192.168.1.224/30 (2 hôtes utilisables) ✅
```

**Résultat** :
```
┌─────────────────────────────────────────────────────────┐
│              192.168.1.0/24 (256 adresses)              │
├──────────────────────┬────────────┬─────────┬───────────┤
│ Site A               │ Site B     │ Site C  │ Liaison   │
│ .0/25 (128 adr)      │ .128/26    │ .192/27 │ .224/30   │
│                      │ (64 adr)   │ (32 adr)│ (4 adr)   │
└──────────────────────┴────────────┴─────────┴───────────┘

Adresses utilisées : 128 + 64 + 32 + 4 = 228
Adresses libres : 256 - 228 = 28 adresses pour l'expansion
```

### Visualisation VLSM

```
192.168.1.0/24 (256 adresses)
│
├─ 192.168.1.0/25     ─┐
│                      │ 128 adr (Site A - 100 hôtes)
│                      │
├─ 192.168.1.128/26   ─┤
│                      │ 64 adr (Site B - 50 hôtes)
│                      │
├─ 192.168.1.192/27   ─┤
│                      │ 32 adr (Site C - 20 hôtes)
│                      │
├─ 192.168.1.224/30   ─┤
│                      │ 4 adr (Liaison routeurs)
│                      │
└─ 192.168.1.228      ─┘ Adresses libres pour expansion
   à .255
```

## Cas pratiques réels

### Cas 1 : Réseau domestique avec VLAN

**Scénario** : Vous avez un réseau `192.168.1.0/24` et voulez :
- VLAN 10 : Famille (50 appareils)
- VLAN 20 : Invités (20 appareils)
- VLAN 30 : IoT (30 appareils)

**Solution** :

```
VLAN 10 Famille : 192.168.1.0/26
  → 62 hôtes, plage .1 à .62

VLAN 20 Invités : 192.168.1.64/27
  → 30 hôtes, plage .65 à .94

VLAN 30 IoT : 192.168.1.96/27
  → 30 hôtes, plage .97 à .126

Libre pour expansion : .128 à .255
```

### Cas 2 : Entreprise multi-sites

**Scénario** : Réseau `10.0.0.0/16` pour 3 sites :

```
Siège social : 10.0.0.0/18    (16 382 hôtes)
Agence Paris : 10.0.64.0/20   (4 094 hôtes)
Agence Lyon  : 10.0.80.0/22   (1 022 hôtes)
```

**Explication** :
- `/18` = 2^14 hôtes = suffisant pour le siège
- `/20` = 2^12 hôtes = suffisant pour Paris
- `/22` = 2^10 hôtes = suffisant pour Lyon

### Cas 3 : Datacenter avec plusieurs réseaux

```
Réseau datacenter : 172.16.0.0/16

Serveurs web      : 172.16.0.0/24   (254 serveurs)
Serveurs DB       : 172.16.1.0/24   (254 serveurs)
Serveurs app      : 172.16.2.0/24   (254 serveurs)
Backup            : 172.16.3.0/25   (126 serveurs)
Management        : 172.16.3.128/26 (62 serveurs)
Liaisons routeurs : 172.16.3.192/26 (plusieurs /30)
```

## Erreurs courantes à éviter

### ❌ Erreur 1 : Oublier le -2 pour les hôtes

```
FAUX : /26 → 2^6 = 64 hôtes
VRAI : /26 → 2^6 - 2 = 62 hôtes (on enlève réseau et broadcast)
```

### ❌ Erreur 2 : Mauvais alignement des sous-réseaux

```
FAUX : 192.168.1.10/26 ❌ (ne commence pas sur un multiple de 64)
VRAI : 192.168.1.0/26 ✅
VRAI : 192.168.1.64/26 ✅
VRAI : 192.168.1.128/26 ✅
```

**Règle** : Un sous-réseau /26 doit commencer sur un multiple de 64.

### ❌ Erreur 3 : Confondre masque et CIDR

```
Masque : 255.255.255.0   ← Notation décimale pointée
CIDR   : /24             ← Notation préfixe

Ce sont deux façons d'écrire la même chose !
```

### ❌ Erreur 4 : Utiliser l'adresse réseau ou broadcast

```
Réseau : 192.168.1.0/26

192.168.1.0   ❌ Adresse réseau (non assignable)
192.168.1.1   ✅ Première IP utilisable
...
192.168.1.62  ✅ Dernière IP utilisable
192.168.1.63  ❌ Adresse broadcast (non assignable)
```

## Outils et astuces pratiques

### Calculateurs en ligne

Pour vérifier vos calculs :
- **ipcalc** (ligne de commande Linux)
- **SubnetCalc** (Windows)
- Sites web : subnet-calculator.com, calculator.net

### Commande Linux/Mac

```bash
# Installer ipcalc
sudo apt install ipcalc  # Ubuntu/Debian
brew install ipcalc       # Mac

# Utiliser
ipcalc 192.168.1.0/26

# Résultat :
Address:   192.168.1.0
Netmask:   255.255.255.192 = 26
Network:   192.168.1.0/26
HostMin:   192.168.1.1
HostMax:   192.168.1.62
Broadcast: 192.168.1.63
Hosts/Net: 62
```

### Méthode du "quick subnet"

Pour les masques courants, mémorisez simplement :

```
/30 → blocs de 4   → .0, .4, .8, .12, .16...
/29 → blocs de 8   → .0, .8, .16, .24, .32...
/28 → blocs de 16  → .0, .16, .32, .48, .64...
/27 → blocs de 32  → .0, .32, .64, .96, .128...
/26 → blocs de 64  → .0, .64, .128, .192
/25 → blocs de 128 → .0, .128
```

## Bonnes pratiques de conception

### 1. Prévoir de la croissance

**Ne jamais** dimensionner exactement :
```
Besoin : 50 hôtes
FAUX : Utiliser /26 (62 hôtes) → aucune marge ❌
VRAI : Utiliser /25 (126 hôtes) → 76 hôtes de marge ✅
```

**Règle** : Prévoir 50-100% de croissance.

### 2. Standardiser quand possible

Dans une entreprise, utilisez la même taille de sous-réseau par type :

```
Tous les bureaux     : /24 (254 hôtes)
Toutes les liaisons  : /30 (2 hôtes)
Tous les serveurs    : /25 (126 hôtes)
```

**Avantage** : Facilite la gestion et la documentation.

### 3. Documenter les allocations

Maintenez un tableau :

```
┌──────────────────┬────────────────────┬────────────┬────────┐
│ Sous-réseau      │ Utilisation        │ VLAN       │ Hôtes  │
├──────────────────┼────────────────────┼────────────┼────────┤
│ 10.0.1.0/24      │ Comptabilité       │ VLAN 10    │ 45/254 │
│ 10.0.2.0/24      │ Informatique       │ VLAN 20    │ 23/254 │
│ 10.0.3.0/24      │ Marketing          │ VLAN 30    │ 18/254 │
│ 10.0.4.0/26      │ Imprimantes        │ VLAN 40    │ 8/62   │
│ 10.0.5.0/25      │ Serveurs           │ VLAN 50    │ 35/126 │
└──────────────────┴────────────────────┴────────────┴────────┘
```

### 4. Réserver des plages

```
10.0.0.0/8 - Plan d'adressage entreprise
│
├─ 10.0-63.x.x    : Réservé pour les sites
├─ 10.64-127.x.x  : Réservé pour les datacenters
├─ 10.128-191.x.x : Réservé pour le cloud
└─ 10.192-255.x.x : Libre pour expansion future
```

## Résumé visuel complet

```
┌──────────────────────────────────────────────────────────────┐
│              SUBNETTING - CONCEPTS CLÉS                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🎯 But : Diviser un réseau en sous-réseaux plus petits      │
│                                                              │
│  📐 Principe : Emprunter des bits à la partie hôte           │
│     ┌─────────┬──────────┬────────┐                          │
│     │ Réseau  │Sous-rés. │ Hôte   │                          │
│     └─────────┴──────────┴────────┘                          │
│                                                              │
│  🔢 Formules :                                               │
│     • Nombre de sous-réseaux = 2^(bits empruntés)            │
│     • Hôtes par sous-réseau = 2^(bits restants) - 2          │
│     • Taille d'un bloc = 2^(bits restants)                   │
│                                                              │
│  📊 Diviser un /24 :                                         │
│     /25 → 2 SR de 126 hôtes (blocs de 128)                   │
│     /26 → 4 SR de 62 hôtes (blocs de 64)                     │
│     /27 → 8 SR de 30 hôtes (blocs de 32)                     │
│     /28 → 16 SR de 14 hôtes (blocs de 16)                    │
│                                                              │
│  🎨 VLSM : Tailles variables optimisent l'utilisation        │
│                                                              │
│  ⚠️  Toujours retirer 2 : réseau + broadcast                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ Le subnetting **divise un réseau** en sous-réseaux plus petits

✅ On **emprunte des bits** de la partie hôte pour créer la partie sous-réseau

✅ Plus de sous-réseaux = **moins d'hôtes** par sous-réseau (compromis)

✅ Formule : **2^(bits empruntés)** = nombre de sous-réseaux

✅ Formule : **2^(bits restants) - 2** = hôtes par sous-réseau

✅ Les sous-réseaux commencent sur des **multiples de la taille du bloc**

✅ **VLSM** permet d'optimiser en utilisant des tailles variables

✅ Toujours prévoir de la **marge de croissance**

✅ Les adresses **réseau et broadcast ne sont pas assignables**

✅ Utiliser `/30` pour les **liaisons point-à-point** (2 hôtes)

## Pour aller plus loin

Maintenant que vous maîtrisez le subnetting, vous êtes prêt à explorer :

- Les **adresses privées** vs publiques (RFC 1918)
- Le **NAT** qui permet de traduire les adresses
- Le **supernetting** (agrégation de routes)
- Les techniques d'**optimisation des tables de routage**

---

**💡 Challenge pratique** : Prenez le réseau de votre domicile (probablement un `/24`) et imaginez comment vous le découperiez si vous deviez séparer :
- Les ordinateurs de la famille
- Les appareils IoT (caméras, thermostats...)
- Le réseau invité
- Les serveurs domestiques

Quel masque utiliseriez-vous pour chaque segment ?

---

*Dans la section suivante, nous allons découvrir les adresses privées vs publiques et comprendre pourquoi certaines adresses ne peuvent pas être utilisées sur Internet...*

⏭️ [Adresses publiques vs privées (RFC 1918)](/03-couche-internet/05-adresses-publiques-privees.md)
