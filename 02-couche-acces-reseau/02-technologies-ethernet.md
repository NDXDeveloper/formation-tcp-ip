🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.2 Technologies Ethernet : principes et évolution

## Introduction

**Ethernet** est la technologie de réseau local la plus répandue au monde. Si vous avez déjà branché un câble réseau dans votre ordinateur, vous avez utilisé Ethernet ! Inventée dans les années 1970, cette technologie a non seulement survécu mais a prospéré, s'adaptant constamment aux besoins croissants de vitesse et de fiabilité.

**Analogie** : Si Internet est l'autoroute qui relie les villes, Ethernet est le réseau de rues dans votre quartier. Simple, efficace, et utilisé quotidiennement par des milliards d'appareils.

## La naissance d'Ethernet

### Contexte historique

À la fin des années 1960, les ordinateurs étaient isolés. Chaque machine était une île. Robert Metcalfe et son équipe au Xerox PARC ont révolutionné cela en 1973 en inventant Ethernet.

**Le défi** : Comment permettre à plusieurs ordinateurs de partager un même câble pour communiquer ?

**La solution Ethernet** : Un système où les machines "écoutent avant de parler" et savent gérer les collisions quand deux parlent en même temps.

### L'origine du nom

Le nom "Ethernet" vient de l'**éther luminifère**, un concept scientifique ancien (aujourd'hui abandonné) selon lequel les ondes lumineuses voyageaient dans un milieu invisible omniprésent. Metcalfe a choisi ce nom pour évoquer l'idée d'un **média partagé invisible** portant les données.

## Principe de fonctionnement fondamental

### Le bus partagé (concept original)

Dans les premiers réseaux Ethernet, tous les ordinateurs étaient connectés au **même câble coaxial**, comme des maisons le long d'une même rue :

```
    Ordinateur A    Ordinateur B    Ordinateur C
         |               |               |
    ═════╪═══════════════╪═══════════════╪═════  ← Câble coaxial partagé
         |               |               |
    Ordinateur D    Ordinateur E    Ordinateur F
```

**Problème** : Si A veut parler à C, tout le monde entend ! C'est comme crier dans un couloir : tout le monde peut écouter.

### CSMA/CD : la règle de politesse

Pour éviter le chaos, Ethernet utilise un protocole appelé **CSMA/CD** :

**CSMA** = **C**arrier **S**ense **M**ultiple **A**ccess (Accès multiple avec détection de porteuse)
**CD** = **C**ollision **D**etection (Détection de collision)

#### Fonctionnement étape par étape

Imaginez une conversation de groupe où tout le monde partage le même microphone :

**1. Écouter avant de parler (Carrier Sense)**
```
Ordinateur A veut parler
    ↓
Il écoute : quelqu'un parle déjà ?
    ├─→ OUI : j'attends
    └─→ NON : je peux parler !
```

**2. Parler et surveiller (Collision Detection)**
```
Ordinateur A commence à transmettre
    ↓
Il surveille en même temps
    ↓
Collision détectée ?
    ├─→ OUI : STOP ! Envoyer signal de brouillage (jam signal)
    └─→ NON : Continuer la transmission
```

**3. Gérer la collision**
```
Collision détectée !
    ↓
Attendre un temps ALÉATOIRE
    ↓
Réessayer (retourner à l'étape 1)
```

**Pourquoi un temps aléatoire ?** Si deux ordinateurs attendent exactement le même temps, ils vont recréer une collision ! Le temps aléatoire évite ce problème.

**Analogie** : C'est comme deux personnes qui se croisent dans un couloir étroit et qui font tous deux un pas du même côté. Elles doivent attendre un instant différent pour ne pas répéter le problème !

### L'algorithme de back-off exponentiel

Quand une collision se produit, l'attente n'est pas totalement aléatoire. Ethernet utilise le **back-off exponentiel binaire** :

- **1ère collision** : Attendre 0 ou 1 unité de temps (choix aléatoire)
- **2ème collision** : Attendre entre 0 et 3 unités de temps
- **3ème collision** : Attendre entre 0 et 7 unités de temps
- **4ème collision** : Attendre entre 0 et 15 unités de temps
- ...et ainsi de suite jusqu'à 10 collisions

**Après 16 collisions** : Abandon, l'erreur est remontée aux couches supérieures.

**Analogie** : C'est comme appeler quelqu'un au téléphone. Si c'est occupé, vous attendez 1 minute. Si c'est encore occupé, vous attendez 5 minutes. Si c'est toujours occupé, vous attendez 15 minutes... jusqu'à abandonner.

## Évolution des topologies

### 1. Bus (années 1970-1980)

Le câble coaxial partagé. Simple mais problématique :
- ❌ Un câble coupé = tout le réseau en panne
- ❌ Difficile à étendre
- ❌ Performances qui se dégradent avec le nombre de machines

```
═══╪═══╪═══╪═══╪═══  ← Si coupé ici...
   💻  💻  💻  💻  💻  ← Tout le réseau est mort !
```

### 2. Étoile avec Hub (années 1990)

Introduction des **hubs (concentrateurs)** et câbles à paires torsadées :

```
           Hub
          ╱ | ╲
         ╱  |  ╲
        💻  💻  💻
```

**Fonctionnement du Hub** : Il reçoit un signal et le **répète sur tous les ports**. C'est toujours un média partagé, mais plus facile à gérer physiquement.

**Analogie** : Le hub est comme un mégaphone : ce que dit une personne est répété à tout le monde.

### 3. Étoile avec Switch (années 1990-aujourd'hui)

Les **switches (commutateurs)** ont révolutionné Ethernet :

```
          Switch
          ╱ | ╲
         ╱  |  ╲
        💻  💻  💻
```

**Différence cruciale avec le hub** : Le switch est **intelligent**. Il apprend quelle machine est sur quel port et envoie les données **uniquement au destinataire**.

**Comparaison Hub vs Switch** :

| Hub | Switch |
|-----|--------|
| Répète tout à tout le monde | Envoie seulement au destinataire |
| Média partagé (collisions possibles) | Communications isolées (pas de collisions) |
| Toutes les machines partagent la bande passante | Chaque machine a sa propre bande passante |
| Obsolète aujourd'hui | Standard moderne |

**Analogie** :
- **Hub** = Crier dans une pièce (tout le monde entend)
- **Switch** = Parler directement à l'oreille de quelqu'un (conversation privée)

## Évolution des vitesses

Ethernet n'a cessé de devenir plus rapide. Voici les principales générations :

### Tableau récapitulatif

| Nom | Vitesse | Année | Câble typique | Distance max |
|-----|---------|-------|---------------|--------------|
| **10BASE5** | 10 Mbit/s | 1980 | Coaxial épais | 500 m |
| **10BASE2** | 10 Mbit/s | 1985 | Coaxial fin | 185 m |
| **10BASE-T** | 10 Mbit/s | 1990 | Paire torsadée (Cat 3) | 100 m |
| **100BASE-TX** (Fast Ethernet) | 100 Mbit/s | 1995 | Paire torsadée (Cat 5) | 100 m |
| **1000BASE-T** (Gigabit Ethernet) | 1 Gbit/s | 1999 | Paire torsadée (Cat 5e/6) | 100 m |
| **10GBASE-T** | 10 Gbit/s | 2006 | Paire torsadée (Cat 6a/7) | 100 m |
| **25G/40G/100G Ethernet** | 25-100 Gbit/s | 2010s | Fibre optique | Plusieurs km |
| **400G Ethernet** | 400 Gbit/s | 2017 | Fibre optique | Plusieurs km |

### Décodage de la nomenclature

Prenons **100BASE-TX** comme exemple :

- **100** : Vitesse en Mégabits par seconde (Mbit/s)
- **BASE** : Signalisation en bande de base (par opposition à large bande)
- **TX** : Type de média (T = Twisted pair/paire torsadée, X = variante spécifique)

Autres suffixes courants :
- **T** : Twisted pair (paire torsadée cuivre)
- **F** : Fiber (fibre optique)
- **LX, SX** : Types de fibre (Long/Short wavelength)
- **2, 5** : Types de coaxial (2 = fin, 5 = épais)

### Mise en perspective des vitesses

Pour vous donner une idée concrète :

**10 Mbit/s** (1990)
- Télécharger un MP3 de 3 Mo : ~2,4 secondes
- Télécharger un film de 700 Mo : ~9 minutes

**100 Mbit/s** (1995)
- Télécharger un MP3 de 3 Mo : ~0,24 seconde
- Télécharger un film de 700 Mo : ~56 secondes

**1 Gbit/s** (1999)
- Télécharger un MP3 de 3 Mo : ~0,024 seconde (instantané)
- Télécharger un film de 700 Mo : ~5,6 secondes

**10 Gbit/s** (2006)
- Télécharger un film 4K de 50 Go : ~40 secondes

**100 Gbit/s** (2010s)
- Télécharger toute votre bibliothèque musicale de 500 Go : ~40 secondes

## Types de câbles Ethernet

### Câble à paires torsadées (le plus courant)

C'est le câble que vous voyez partout, avec une prise **RJ45** au bout :

```
    ┌─────┐
    │ RJ45│
    └──┬──┘
       │
    ═══╪═══  ← 8 fils torsadés 2 par 2
       │
    ┌──┴──┐
    │ RJ45│
    └─────┘
```

**Pourquoi torsadés ?** Les paires sont entrelacées pour réduire les interférences électromagnétiques.

**Catégories de câbles** :

| Catégorie | Vitesse max | Usage typique |
|-----------|-------------|---------------|
| **Cat 5** | 100 Mbit/s | Obsolète |
| **Cat 5e** | 1 Gbit/s | Standard actuel pour la maison |
| **Cat 6** | 10 Gbit/s (55m) / 1 Gbit/s (100m) | Bureaux modernes |
| **Cat 6a** | 10 Gbit/s (100m) | Data centers |
| **Cat 7/8** | 40 Gbit/s | Installations futures |

### Câble droit vs croisé

**Câble droit (straight-through)** : Pour connecter un ordinateur à un switch/hub
```
Ordinateur  ═══════  Switch
   Pin 1 ────────────→ Pin 1
   Pin 2 ────────────→ Pin 2
   Pin 3 ────────────→ Pin 3
   ...
```

**Câble croisé (crossover)** : Pour connecter directement deux ordinateurs
```
Ordinateur A  ═══════  Ordinateur B
   Pin 1 ────────────→ Pin 3
   Pin 2 ────────────→ Pin 6
   Pin 3 ────────────→ Pin 1
   Pin 6 ────────────→ Pin 2
```

**Note moderne** : Aujourd'hui, grâce à l'**Auto-MDI/MDIX**, la plupart des équipements détectent automatiquement le type de câble et s'adaptent. Vous n'avez presque plus besoin de câbles croisés !

### Fibre optique

Pour les longues distances et très hautes vitesses :

**Avantages** :
- ✅ Distances énormes (plusieurs kilomètres)
- ✅ Pas d'interférences électromagnétiques
- ✅ Très hautes vitesses (100+ Gbit/s)
- ✅ Sécurisée (difficile à intercepter)

**Inconvénients** :
- ❌ Plus coûteuse
- ❌ Plus fragile
- ❌ Installation plus complexe

**Types** :
- **Monomode (SMF)** : Un seul faisceau lumineux, longues distances (10-80 km)
- **Multimode (MMF)** : Plusieurs faisceaux, distances courtes (300-500 m), moins cher

## Half-duplex vs Full-duplex

### Half-duplex (historique)

**Définition** : Une seule direction à la fois, comme un talkie-walkie.

```
Ordinateur A ⇄ Ordinateur B

Temps 1 : A parle  →  B écoute
Temps 2 : A écoute ←  B parle
```

- Utilisé avec les hubs et bus partagés
- CSMA/CD nécessaire
- Bande passante partagée

### Full-duplex (moderne)

**Définition** : Les deux directions simultanément, comme un téléphone.

```
Ordinateur A ⇄ Ordinateur B

A envoie  →  → →  B reçoit
A reçoit  ← ← ←  B envoie
(simultanément !)
```

- Standard avec les switches
- **Pas de collisions** (chemins séparés)
- **Double la bande passante effective**
- CSMA/CD désactivé

**Analogie** :
- **Half-duplex** = Route à une voie avec alternance
- **Full-duplex** = Autoroute à deux voies séparées

## Ethernet moderne : Power over Ethernet (PoE)

Une innovation récente permet de transmettre **électricité et données** sur le même câble Ethernet :

**Applications typiques** :
- 📷 Caméras IP de surveillance
- 📞 Téléphones VoIP
- 📡 Points d'accès Wi-Fi
- 🚪 Systèmes de contrôle d'accès

**Standards** :
- **PoE (802.3af)** : 15,4 W par port
- **PoE+ (802.3at)** : 30 W par port
- **PoE++ (802.3bt)** : Jusqu'à 100 W par port

**Avantage** : Un seul câble pour tout ! Simplifie l'installation dans les lieux sans prises électriques accessibles.

## Pourquoi Ethernet a survécu

Malgré 50 ans d'existence, Ethernet domine toujours. Pourquoi ?

### 1. **Adaptabilité**
Ethernet a su évoluer du coaxial à la fibre, de 10 Mbit/s à 400 Gbit/s, sans changer ses principes fondamentaux.

### 2. **Rétrocompatibilité**
Un équipement Gigabit peut communiquer avec du Fast Ethernet. L'**autonégociation** permet aux appareils de s'accorder automatiquement sur la meilleure vitesse commune.

### 3. **Standardisation ouverte**
Défini par l'**IEEE 802.3**, accessible à tous, pas de propriétaire unique.

### 4. **Économie d'échelle**
Produit massivement = prix très bas = adoption massive = encore plus produit...

### 5. **Simplicité**
Facile à comprendre, à installer, à dépanner.

## Les standards IEEE 802.3

Ethernet est standardisé par l'**IEEE** (Institute of Electrical and Electronics Engineers) sous la désignation **802.3** :

| Standard | Description |
|----------|-------------|
| **802.3** | Ethernet 10 Mbit/s original |
| **802.3u** | Fast Ethernet 100 Mbit/s |
| **802.3ab** | Gigabit Ethernet sur paire torsadée |
| **802.3ae** | 10 Gigabit Ethernet |
| **802.3af/at/bt** | Power over Ethernet |
| **802.3bz** | 2.5G/5GBASE-T |

Chaque lettre après "802.3" représente une évolution ou une variante spécifique.

## Schéma récapitulatif de l'évolution

```
┌─────────────────────────────────────────────────────────┐
│          ÉVOLUTION D'ETHERNET (1973-2025)               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1973 ┃ Invention (Xerox PARC)                          │
│       ┃ • Bus coaxial partagé                           │
│       ┃ • 3 Mbit/s                                      │
│       ┃                                                 │
│  1980 ┃ 10BASE5 (Thick Ethernet)                        │
│       ┃ • Câble coaxial épais                           │
│       ┃ • 10 Mbit/s                                     │
│       ┃                                                 │
│  1985 ┃ 10BASE2 (Thin Ethernet)                         │
│       ┃ • Câble coaxial fin                             │
│       ┃                                                 │
│  1990 ┃ 10BASE-T + Hubs                                 │
│       ┃ • Paires torsadées                              │
│       ┃ • Topologie étoile                              │
│       ┃                                                 │
│  1995 ┃ Fast Ethernet 100 Mbit/s + Switches             │
│       ┃ • Full-duplex                                   │
│       ┃ • Fin de CSMA/CD en pratique                    │
│       ┃                                                 │
│  1999 ┃ Gigabit Ethernet 1 Gbit/s                       │
│       ┃                                                 │
│  2003 ┃ Power over Ethernet (PoE)                       │
│       ┃                                                 │
│  2006 ┃ 10 Gigabit Ethernet                             │
│       ┃                                                 │
│  2010s┃ 40G/100G Ethernet (data centers)                │
│       ┃                                                 │
│  2017 ┃ 400G Ethernet                                   │
│       ┃                                                 │
│  2020s┃ 800G/1.6T en développement                      │
│       ┃                                                 │
└─────────────────────────────────────────────────────────┘
```

## Points clés à retenir

1. **Ethernet = Standard dominant** : Plus de 85% des réseaux locaux utilisent Ethernet.

2. **CSMA/CD** : La méthode historique de gestion des collisions (obsolète avec les switches modernes).

3. **Évolution continue** : De 10 Mbit/s à 400+ Gbit/s, Ethernet s'adapte constamment.

4. **Switches > Hubs** : Les switches modernes ont éliminé les collisions et multiplié les performances.

5. **Full-duplex** : Communication bidirectionnelle simultanée, standard aujourd'hui.

6. **Câbles** : Paires torsadées (Cat 5e/6) pour la plupart des usages, fibre pour les hautes performances.

7. **Standards ouverts** : IEEE 802.3 assure l'interopérabilité universelle.

---

## Prochaine étape

Maintenant que vous comprenez comment Ethernet fonctionne physiquement, nous allons découvrir comment les machines s'identifient sur le réseau avec les **adresses MAC**, l'équivalent des plaques d'immatriculation du monde réseau !

---


⏭️ [Adresses MAC : structure et fonctionnement](/02-couche-acces-reseau/03-adresses-mac.md)
