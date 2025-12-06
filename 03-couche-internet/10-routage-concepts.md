🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.10 Routage : concepts fondamentaux

## Introduction : le GPS d'Internet

Imaginez que vous devez vous rendre de Paris à Tokyo. Vous ne prenez pas un avion direct qui traverse la Terre en ligne droite ! Au lieu de cela, vous faites plusieurs **escales** : Paris → Londres → New York → Los Angeles → Tokyo. À chaque aéroport, on vous indique vers **quel prochain vol** vous diriger pour vous rapprocher de votre destination.

Le **routage** sur Internet fonctionne exactement de cette manière. Vos paquets ne voyagent pas directement de votre ordinateur à un serveur à l'autre bout du monde. Ils **sautent de routeur en routeur**, et chaque routeur décide **vers quel routeur suivant** les envoyer pour les rapprocher de leur destination.

C'est ce processus de **sélection du chemin optimal** qu'on appelle le **routage**.

## Qu'est-ce que le routage ?

### Définition simple

**Routage** = Le processus par lequel les paquets IP trouvent leur chemin d'une source à une destination à travers un réseau interconnecté.

**En termes simples** :
```
Le routage répond à la question :
"Vers OÙ dois-je envoyer ce paquet pour qu'il arrive à destination ?"
```

### Les acteurs du routage

**1. Les routeurs** :
```
Équipements réseau spécialisés qui :
  - Reçoivent des paquets
  - Examinent l'adresse IP de destination
  - Décident vers quel routeur suivant les envoyer
  - Transmettent les paquets

Analogie : Panneaux de signalisation sur l'autoroute
```

**2. Les tables de routage** :
```
Base de données dans chaque routeur contenant :
  - Les réseaux connus
  - Vers où envoyer les paquets pour chaque réseau
  - Les "coûts" de chaque chemin

Analogie : Carte routière avec distances et directions
```

**3. Les protocoles de routage** :
```
Mécanismes qui permettent aux routeurs de :
  - Découvrir les réseaux disponibles
  - Échanger des informations de routage
  - Calculer les meilleurs chemins
  - S'adapter aux changements

Analogie : Applications GPS qui partagent les infos trafic
```

## Routage vs Commutation : quelle différence ?

### La commutation (switching) - Couche 2

**Principe** : Transmettre des trames en se basant sur les **adresses MAC** au sein d'un **même réseau local**.

```
┌────────────────────────────────────────────┐
│         Réseau Local (LAN)                 │
│    192.168.1.0/24                          │
│                                            │
│  PC A          Switch         PC B         │
│  .10     ←──────┼──────→      .20          │
│  MAC: AA        │             MAC: BB      │
│                 │                          │
│  Le switch connaît les MAC,                │
│  pas besoin de routeur                     │
└────────────────────────────────────────────┘

Switch = Couche 2 (Liaison de données)
Décision basée sur : Adresse MAC
Portée : Un seul réseau local
```

**Analogie** : Distribution du courrier **dans un immeuble**. Le concierge connaît tous les appartements et distribue directement.

### Le routage (routing) - Couche 3

**Principe** : Transmettre des paquets en se basant sur les **adresses IP** entre **différents réseaux**.

```
┌─────────────────┐        ┌─────────────────┐
│  Réseau A       │        │  Réseau B       │
│  192.168.1.0/24 │        │  10.0.0.0/24    │
│                 │        │                 │
│  PC A (.10)     │        │  PC B (.10)     │
└────────┬────────┘        └────────┬────────┘
         │                          │
         │       ┌──────────┐       │
         └──────→│ ROUTEUR  │←──────┘
                 └──────────┘

Routeur = Couche 3 (Réseau)
Décision basée sur : Adresse IP
Portée : Entre différents réseaux
```

**Analogie** : Système postal **entre villes**. Chaque centre de tri décide vers quelle ville envoyer le courrier ensuite.

### Tableau comparatif

```
┌──────────────────┬──────────────────┬──────────────────┐
│ Critère          │ Commutation (L2) │ Routage (L3)     │
├──────────────────┼──────────────────┼──────────────────┤
│ Adresse utilisée │ MAC              │ IP               │
│ Équipement       │ Switch           │ Routeur          │
│ Portée           │ Réseau local     │ Inter-réseaux    │
│ Domaine          │ Domaine broadcast│ Entre domaines   │
│ Intelligence     │ Simple (table)   │ Complexe (algos) │
│ Vitesse          │ Très rapide      │ Plus lent        │
│ Décision         │ Lookup direct    │ Calcul du chemin │
└──────────────────┴──────────────────┴──────────────────┘
```

## Concepts clés du routage

### 1. Le saut (hop)

**Définition** : Un **saut** = le passage d'un routeur à un autre.

```
Chemin de votre PC à Google :

PC → Routeur 1 → Routeur 2 → Routeur 3 → Google
     └─hop 1─┘   └─hop 2─┘   └─hop 3─┘

Total : 3 sauts (hops)
```

**Analogie** : Nombre d'escales dans un voyage en avion.

**Importance** :
- Plus il y a de sauts, plus la latence augmente
- Le TTL décrémente à chaque saut
- Le traceroute compte les sauts

### 2. Le routeur de passerelle (gateway)

**Définition** : Le **premier routeur** que vos paquets rencontrent en sortant de votre réseau local.

```
Votre réseau domestique : 192.168.1.0/24

┌──────────────────────────────────────────┐
│  PC         Smartphone      Tablette     │
│  .10           .20            .30        │
│   │            │              │          │
│   └────────────┴──────────────┘          │
│                 │                        │
│         ┌───────▼────────┐               │
│         │  Box/Routeur   │               │
│         │  192.168.1.1   │ ← GATEWAY     │
│         │  (Passerelle)  │               │
│         └───────┬────────┘               │
└─────────────────┼────────────────────────┘
                  │
              INTERNET
```

**Configuration sur votre PC** :
```
IP Address     : 192.168.1.10
Subnet Mask    : 255.255.255.0
Default Gateway: 192.168.1.1  ← C'est ici !

Signification : "Pour tout ce qui n'est pas sur mon réseau local,
                 envoie à 192.168.1.1 qui saura quoi faire"
```

**Analogie** : Le gateway est comme la **porte de sortie** de votre maison vers le monde extérieur.

### 3. Le prochain saut (next hop)

**Définition** : L'adresse IP du **prochain routeur** vers lequel transmettre un paquet.

```
Table de routage simplifiée d'un routeur :

┌────────────────────┬──────────────────┬──────────┐
│ Réseau Destination │ Next Hop         │ Interface│
├────────────────────┼──────────────────┼──────────┤
│ 192.168.1.0/24     │ Directement      │ eth0     │
│ 10.0.0.0/24        │ 203.0.113.2      │ eth1     │
│ 172.16.0.0/16      │ 203.0.113.5      │ eth1     │
│ 0.0.0.0/0 (défaut) │ 203.0.113.1      │ eth1     │
└────────────────────┴──────────────────┴──────────┘

Pour un paquet vers 10.0.0.50 :
  Routeur consulte la table
  Trouve : 10.0.0.0/24 → next hop = 203.0.113.2
  Envoie le paquet à 203.0.113.2
```

**Analogie** : Sur un GPS, c'est l'instruction "Tournez à droite au prochain carrefour" plutôt que "Votre destination finale est à 50 km".

### 4. La route par défaut (default route)

**Définition** : La route utilisée quand **aucune autre route spécifique** ne correspond.

**Notation** : `0.0.0.0/0` (correspond à **toutes les adresses**)

```
Question du routeur : "Où envoyer un paquet vers 8.8.8.8 ?"

Recherche dans la table :
  ❌ 192.168.1.0/24 ? Non, pas dans cette plage
  ❌ 10.0.0.0/24 ? Non plus
  ❌ 172.16.0.0/16 ? Non plus
  ✅ 0.0.0.0/0 (défaut) ? OUI ! Correspond à tout

→ Utiliser la route par défaut
→ Next hop : 203.0.113.1 (vers Internet)
```

**Analogie** : C'est le panneau "Toutes directions" sur une route. Si vous ne savez pas où aller, suivez cette direction.

**Sur votre PC** :
```bash
# Linux/Mac
ip route show
default via 192.168.1.1 dev eth0  ← Route par défaut

# Windows
route print
0.0.0.0          0.0.0.0      192.168.1.1  ← Route par défaut
```

### 5. Routage direct vs routage indirect

#### Routage direct

**Définition** : La destination est sur le **même réseau local** que la source.

```
PC A (192.168.1.10) → PC B (192.168.1.20)
Même réseau : 192.168.1.0/24

Processus :
  1. PC A vérifie : 192.168.1.20 sur mon réseau ? OUI
  2. Pas besoin de routeur !
  3. ARP pour trouver la MAC de PC B
  4. Envoi direct via le switch
```

**Schéma** :
```
┌──────────────────────────────────┐
│     Réseau 192.168.1.0/24        │
│                                  │
│  PC A (.10) ────→ PC B (.20)     │
│         Communication directe    │
└──────────────────────────────────┘
```

**Analogie** : Parler à votre voisin de palier sans passer par le hall d'entrée.

#### Routage indirect

**Définition** : La destination est sur un **réseau différent**, nécessite un ou plusieurs routeurs.

```
PC A (192.168.1.10) → Serveur (8.8.8.8)
Réseaux différents

Processus :
  1. PC A vérifie : 8.8.8.8 sur mon réseau ? NON
  2. Envoyer au gateway (192.168.1.1)
  3. Gateway routera vers le prochain saut
  4. Et ainsi de suite jusqu'à destination
```

**Schéma** :
```
┌─────────────────┐     ┌─────────┐     ┌─────────────┐
│ Réseau A        │     │ Routeur │     │  Internet   │
│ 192.168.1.0/24  │     │         │     │             │
│                 │     │         │     │  8.8.8.8    │
│  PC A (.10) ────┼────→│ (.1)    │─────┼→ Google DNS │
└─────────────────┘     └─────────┘     └─────────────┘
        └─────────── Routage indirect ─────────┘
```

**Analogie** : Envoyer une lettre à l'étranger via plusieurs centres de tri.

## Le processus de décision de routage

### Étape par étape

Voici comment un routeur décide où envoyer un paquet :

```
Un paquet arrive avec destination : 172.16.50.100

┌────────────────────────────────────────────────────┐
│ ÉTAPE 1 : Extraire l'IP de destination             │
│   → 172.16.50.100                                  │
└────────────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────────────┐
│ ÉTAPE 2 : Consulter la table de routage            │
│                                                    │
│   Réseau A : 192.168.1.0/24   → Next hop: eth0     │
│   Réseau B : 172.16.0.0/16    → Next hop: eth1     │
│   Réseau C : 10.0.0.0/8       → Next hop: eth2     │
│   Défaut   : 0.0.0.0/0        → Next hop: eth3     │
└────────────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────────────┐
│ ÉTAPE 3 : Trouver la correspondance                │
│                                                    │
│   172.16.50.100 appartient à 172.16.0.0/16 ? OUI!  │
│   → Utiliser cette route                           │
└────────────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────────────┐
│ ÉTAPE 4 : Sélectionner le next hop                 │
│   → Interface eth1                                 │
└────────────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────────────┐
│ ÉTAPE 5 : Décrémenter le TTL                       │
│   TTL = 64 → TTL = 63                              │
│   Si TTL = 0 : détruire le paquet (Time Exceeded)  │
└────────────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────────────┐
│ ÉTAPE 6 : Recalculer le checksum de l'en-tête      │
│   (car TTL a changé)                               │
└────────────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────────────┐
│ ÉTAPE 7 : Transmettre le paquet                    │
│   → Envoyer sur l'interface eth1                   │
└────────────────────────────────────────────────────┘
```

### Algorithme de correspondance : le plus spécifique gagne

**Règle d'or** : Quand plusieurs routes correspondent, le routeur choisit la **plus spécifique** (masque le plus long).

```
Table de routage :

┌────────────────────┬──────────┬─────────┐
│ Réseau             │ Masque   │ Next Hop│
├────────────────────┼──────────┼─────────┤
│ 192.168.1.0        │ /24      │ R1      │
│ 192.168.0.0        │ /16      │ R2      │
│ 0.0.0.0            │ /0       │ R3      │
└────────────────────┴──────────┴─────────┘

Paquet vers : 192.168.1.50

Correspondances :
  ✅ 192.168.1.0/24 ? OUI (192.168.1.50 dedans)
  ✅ 192.168.0.0/16 ? OUI (192.168.1.50 dedans aussi)
  ✅ 0.0.0.0/0 ?      OUI (correspond à tout)

Sélection : /24 (le plus spécifique)
→ Next hop : R1
```

**Pourquoi ?**

```
/24 = 255.255.255.0   = 24 bits de réseau (très spécifique)
/16 = 255.255.0.0     = 16 bits de réseau (moins spécifique)
/0  = 0.0.0.0         = 0 bits de réseau (le moins spécifique)

Plus le masque est long (/32 > /24 > /16 > /8 > /0),
plus la route est spécifique et prioritaire !
```

**Analogie** :
```
/24 = "15 rue Victor Hugo, 75016 Paris" (très précis)
/16 = "75016 Paris" (moins précis)
/0  = "France" (très vague)

Si vous avez une lettre pour "15 rue Victor Hugo, 75016",
vous utilisez l'adresse la plus précise !
```

### Exemple pratique complet

```
Configuration réseau :

┌─────────────────────────────────────────────────────┐
│                   ROUTEUR R1                        │
│                                                     │
│  eth0 : 192.168.1.1/24  (Réseau local)              │
│  eth1 : 203.0.113.1/30  (Vers Internet)             │
│                                                     │
│  Table de routage :                                 │
│  ┌─────────────────┬────────────┬──────────┐        │
│  │ Destination     │ Next Hop   │ Interface│        │
│  ├─────────────────┼────────────┼──────────┤        │
│  │ 192.168.1.0/24  │ Connected  │ eth0     │        │
│  │ 203.0.113.0/30  │ Connected  │ eth1     │        │
│  │ 0.0.0.0/0       │ 203.0.113.2│ eth1     │        │
│  └─────────────────┴────────────┴──────────┘        │
└─────────────────────────────────────────────────────┘
```

**Scénario 1** : PC (192.168.1.10) envoie à 192.168.1.20

```
1. R1 reçoit le paquet sur eth0
2. Destination : 192.168.1.20
3. Recherche dans la table :
   → 192.168.1.0/24 → Connected sur eth0
4. Destination sur le MÊME réseau que l'interface d'arrivée
5. R1 fait un ARP pour trouver la MAC de 192.168.1.20
6. Transmet directement (routage direct)
```

**Scénario 2** : PC (192.168.1.10) envoie à 8.8.8.8

```
1. R1 reçoit le paquet sur eth0
2. Destination : 8.8.8.8
3. Recherche dans la table :
   ❌ 192.168.1.0/24 ? Non (8.8.8.8 pas dedans)
   ❌ 203.0.113.0/30 ? Non
   ✅ 0.0.0.0/0 ? Oui ! (défaut)
4. Next hop : 203.0.113.2
5. Transmet sur eth1 vers 203.0.113.2
6. Ce routeur décidera de la suite (routage indirect)
```

## Métrique et coût de route

### Qu'est-ce qu'une métrique ?

**Définition** : Valeur numérique représentant le **"coût"** d'une route.

**Usage** : Quand plusieurs routes mènent à la même destination, le routeur choisit celle avec la **métrique la plus faible**.

```
Destination : 10.0.0.0/24

Route 1 : via Routeur A, métrique = 5
Route 2 : via Routeur B, métrique = 10
Route 3 : via Routeur C, métrique = 3

→ Routeur choisit Route 3 (métrique la plus faible)
```

### Types de métriques

Différents protocoles de routage utilisent différentes métriques :

```
┌────────────────┬──────────────────────────────────┐
│ Protocole      │ Métrique utilisée                │
├────────────────┼──────────────────────────────────┤
│ RIP            │ Nombre de sauts (hop count)      │
│ OSPF           │ Coût (basé sur bande passante)   │
│ EIGRP          │ Composite (BP, délai, charge...) │
│ BGP            │ AS Path length + policies        │
│ Statique       │ Configurée manuellement          │
└────────────────┴──────────────────────────────────┘
```

#### Métrique RIP : Nombre de sauts

```
Chemin A : PC → R1 → R2 → Destination
           (3 sauts)

Chemin B : PC → R3 → R4 → R5 → Destination
           (4 sauts)

RIP choisit : Chemin A (moins de sauts)
```

**Limite** : Ne considère PAS la bande passante !

```
Chemin A : 3 sauts, mais des liens 10 Mbps
Chemin B : 4 sauts, mais des liens 1 Gbps

RIP choisira A, mais B serait plus rapide !
```

#### Métrique OSPF : Coût

**Formule** : Coût = Bande passante de référence / Bande passante du lien

```
Bande passante de référence : 100 Mbps (par défaut)

Lien 100 Mbps : Coût = 100/100 = 1
Lien 10 Mbps  : Coût = 100/10  = 10
Lien 1 Gbps   : Coût = 100/1000 = 1 (arrondi)

Route totale = Somme des coûts de tous les liens
```

**Exemple** :

```
Chemin A : Lien 100 Mbps (coût 1) + Lien 10 Mbps (coût 10)
           = Coût total : 11

Chemin B : 3 × Lien 100 Mbps (coût 1 chacun)
           = Coût total : 3

OSPF choisit : Chemin B (coût plus faible)
```

### Métrique administrative (Administrative Distance)

**Problème** : Que faire si **deux protocoles différents** proposent des routes vers la même destination ?

```
RIP dit : "J'ai une route vers 10.0.0.0/24 avec métrique 5"
OSPF dit : "J'ai une route vers 10.0.0.0/24 avec métrique 100"

Impossible de comparer ! Métriques différentes.
```

**Solution** : La **distance administrative** (AD) = degré de confiance dans le protocole.

```
┌─────────────────────┬──────────────────┐
│ Source de la route  │ AD (Cisco)       │
├─────────────────────┼──────────────────┤
│ Interface directe   │ 0 (max confiance)│
│ Route statique      │ 1                │
│ EIGRP summary       │ 5                │
│ eBGP                │ 20               │
│ EIGRP interne       │ 90               │
│ OSPF                │ 110              │
│ RIP                 │ 120              │
│ EIGRP externe       │ 170              │
│ iBGP                │ 200              │
│ Inconnu             │ 255 (rejetée)    │
└─────────────────────┴──────────────────┘

Plus l'AD est faible, plus le protocole est fiable.
```

**Exemple de décision** :

```
RIP propose : 10.0.0.0/24 métrique 5, AD 120
OSPF propose : 10.0.0.0/24 métrique 100, AD 110

Routeur choisit : OSPF (AD 110 < AD 120)
→ Peu importe que RIP ait une meilleure métrique !
```

## Routage asymétrique

### Le phénomène

**Important** : L'aller et le retour d'une connexion **peuvent emprunter des chemins différents** !

```
PC A (Paris) ──┐                    ┌── Serveur (Tokyo)
               │                    │
               ↓                    ↑
         ┌──────────┐        ┌──────────┐
         │ Route A  │        │ Route B  │
         │ (Europe) │        │ (Asie)   │
         └──────────┘        └──────────┘
               │                    │
               └──→ Aller ──→──────┘

               ┌────←── Retour ─────┐
               │                    │
         ┌──────────┐        ┌───────────┐
         │ Route C  │        │ Route D   │
         │(Atlantique)       │(Pacifique)│
         └──────────┘        └───────────┘
```

**Pourquoi ?**

```
Chaque routeur prend une décision LOCALE et INDÉPENDANTE :
  - Routeurs en Europe optimisent vers l'est (A)
  - Routeurs en Asie optimisent vers l'ouest (D)
  - Les chemins peuvent être complètement différents !
```

**Conséquences** :

```
✅ Optimisation : Chaque sens prend le meilleur chemin
⚠️ Troubleshooting : Traceroute aller ≠ traceroute retour
⚠️ Firewall : Doit gérer le trafic dans les deux sens
⚠️ Latence : Peut varier selon le sens
```

**Exemple pratique** :

```bash
# Traceroute aller (depuis votre PC)
$ traceroute google.com
1  192.168.1.1
2  10.0.0.1
3  paris-router.fr
4  frankfurt-router.de
5  google.com

# Traceroute retour (depuis Google vers vous)
# → Pourrait passer par Amsterdam, Londres, etc.
# Pas nécessairement le même chemin !
```

## Le routage en action : exemple complet

### Topologie réseau

```
┌──────────────┐         ┌──────────────┐          ┌──────────────┐
│  Réseau A    │         │  Réseau B    │          │  Réseau C    │
│ 192.168.1.0  │         │  10.0.0.0    │          │  172.16.0.0  │
│    /24       │         │    /24       │          │    /24       │
│              │         │              │          │              │
│  PC A        │         │              │          │  Serveur     │
│  .10         │         │              │          │  .50         │
└──────┬───────┘         └──────────────┘          └──────┬───────┘
       │                        │                         │
       │eth0                    │eth1               eth0  │
    ┌──▼────────┐          ┌────▼───────┐          ┌──────▼─────┐
    │ Routeur 1 │ eth1─────│ Routeur 2  │─────eth2 │ Routeur 3  │
    │192.168.1.1│ 10.0.0.1 │  10.0.0.2  │172.16.0.1│172.16.0.254│
    └───────────┘          │203.0.113.1 │          └────────────┘
                           └────────────┘
```

### Tables de routage

**Routeur 1** :
```
┌─────────────────┬────────────┬──────────┐
│ Destination     │ Next Hop   │ Interface│
├─────────────────┼────────────┼──────────┤
│ 192.168.1.0/24  │ Connected  │ eth0     │
│ 10.0.0.0/24     │ Connected  │ eth1     │
│ 172.16.0.0/24   │ 10.0.0.2   │ eth1     │
└─────────────────┴────────────┴──────────┘
```

**Routeur 2** :
```
┌─────────────────┬────────────┬──────────┐
│ Destination     │ Next Hop   │ Interface│
├─────────────────┼────────────┼──────────┤
│ 192.168.1.0/24  │ 10.0.0.1   │ eth1     │
│ 10.0.0.0/24     │ Connected  │ eth1     │
│ 172.16.0.0/24   │ Connected  │ eth2     │
│ 0.0.0.0/0       │ Connected  │ eth3     │
└─────────────────┴────────────┴──────────┘
```

**Routeur 3** :
```
┌─────────────────┬────────────┬──────────┐
│ Destination     │ Next Hop   │ Interface│
├─────────────────┼────────────┼──────────┤
│ 192.168.1.0/24  │ 172.16.0.1 │ eth0     │
│ 10.0.0.0/24     │ 172.16.0.1 │ eth0     │
│ 172.16.0.0/24   │ Connected  │ eth0     │
└─────────────────┴────────────┴──────────┘
```

### Traçage d'un paquet : PC A → Serveur

**Paquet** : 192.168.1.10 → 172.16.0.50

```
┌──────────────────────────────────────────────────────┐
│ ÉTape 1 : PC A (192.168.1.10)                        │
├──────────────────────────────────────────────────────┤
│ Crée le paquet :                                     │
│   Source : 192.168.1.10                              │
│   Dest   : 172.16.0.50                               │
│   TTL    : 64                                        │
│                                                      │
│ Décision : 172.16.0.50 sur mon réseau ?              │
│   Mon réseau : 192.168.1.0/24                        │
│   172.16.0.50 pas dedans → routage indirect          │
│                                                      │
│ Action : Envoyer au gateway (192.168.1.1)            │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ ÉTAPE 2 : Routeur 1 (192.168.1.1)                    │
├──────────────────────────────────────────────────────┤
│ Reçoit sur eth0                                      │
│ Destination : 172.16.0.50                            │
│                                                      │
│ Consultation table de routage :                      │
│   192.168.1.0/24 ? Non                               │
│   10.0.0.0/24 ? Non                                  │
│   172.16.0.0/24 ? OUI !                              │
│   Next hop : 10.0.0.2 via eth1                       │
│                                                      │
│ Décrémente TTL : 64 → 63                             │
│ Transmet vers 10.0.0.2 (Routeur 2)                   │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ ÉTAPE 3 : Routeur 2 (10.0.0.2)                       │
├──────────────────────────────────────────────────────┤
│ Reçoit sur eth1                                      │
│ Destination : 172.16.0.50                            │
│                                                      │
│ Consultation table de routage :                      │
│   172.16.0.0/24 ? OUI ! Connected sur eth2           │
│                                                      │
│ Décrémente TTL : 63 → 62                             │
│ ARP pour trouver MAC de 172.16.0.1 (Routeur 3)       │
│ Transmet sur eth2                                    │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ ÉTAPE 4 : Routeur 3 (172.16.0.254)                   │
├──────────────────────────────────────────────────────┤
│ Reçoit sur eth0                                      │
│ Destination : 172.16.0.50                            │
│                                                      │
│ Consultation table de routage :                      │
│   172.16.0.0/24 ? OUI ! Connected sur eth0           │
│                                                      │
│ Destination sur le MÊME réseau que l'arrivée         │
│ → Routage direct                                     │
│                                                      │
│ Décrémente TTL : 62 → 61                             │
│ ARP pour trouver MAC de 172.16.0.50                  │
│ Transmet directement au Serveur                      │
└──────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────┐
│ ÉTAPE 5 : Serveur (172.16.0.50)                      │
├──────────────────────────────────────────────────────┤
│ Reçoit le paquet                                     │
│ Destination = sa propre IP → C'est pour moi !        │
│ Décapsule et traite le paquet                        │
│                                                      │
│ Prépare la réponse (chemin inverse)                  │
└──────────────────────────────────────────────────────┘
```

**Résumé du chemin** :
```
PC A → R1 → R2 → R3 → Serveur

Total : 3 sauts (hops)
TTL final : 61 (commencé à 64)
```

## Boucles de routage

### Le problème

**Boucle de routage** = Situation où un paquet **tourne en rond** entre plusieurs routeurs.

```
Routeur A dit : "Pour 10.0.0.0/24, va chez B"
Routeur B dit : "Pour 10.0.0.0/24, va chez C"
Routeur C dit : "Pour 10.0.0.0/24, va chez A"

Résultat : A → B → C → A → B → C → ...
           Boucle infinie ! ∞
```

**Schéma** :

```
     ┌────────────┐
     │ Routeur A  │
     └──────┬─────┘
            │
            │ "10.0.0.0/24 → B"
            ↓
     ┌────────────┐
     │ Routeur B  │
     └──────┬─────┘
            │
            │ "10.0.0.0/24 → C"
            ↓
     ┌────────────┐
     │ Routeur C  │
     └──────┬─────┘
            │
            │ "10.0.0.0/24 → A"
            ↓
         (Retour à A !)
```

### Protection : le TTL

**Heureusement**, le TTL **empêche les boucles infinies** :

```
Paquet créé : TTL = 64
     ↓
Routeur A : TTL = 63
     ↓
Routeur B : TTL = 62
     ↓
Routeur C : TTL = 61
     ↓
Routeur A : TTL = 60
     ↓
... (boucle) ...
     ↓
Routeur X : TTL = 1
     ↓
Routeur Y : TTL = 0 → ❌ PAQUET DÉTRUIT
     ↓
ICMP Time Exceeded envoyé à la source
```

**Sans TTL** : Le paquet tournerait indéfiniment, saturant le réseau !

### Causes de boucles

```
1. Configuration manuelle erronée
   Routeur A : route vers B
   Routeur B : route vers A

2. Convergence lente des protocoles de routage
   Réseau change, mais tous les routeurs
   n'ont pas encore la nouvelle information

3. Redistribution de routes mal configurée
   Entre deux protocoles de routage différents
```

### Prévention

```
✅ Protocoles de routage modernes avec mécanismes anti-boucle :
   - OSPF : SPF algorithm
   - BGP : AS Path (rejette si boucle détectée)
   - RIP : Split horizon, route poisoning

✅ Conception soigneuse du réseau
   Architecture hiérarchique claire

✅ Outils de monitoring
   Détecter les anomalies rapidement
```

## Types de routage : vue d'ensemble

Nous avons vu les concepts fondamentaux. Le routage peut être implémenté de deux manières :

### 1. Routage statique

**Configuration manuelle** des routes par l'administrateur.

```
Avantages :
  ✅ Contrôle total
  ✅ Prévisible
  ✅ Pas de trafic de protocole
  ✅ Sécurisé (pas de découverte auto)

Inconvénients :
  ❌ Pas d'adaptation automatique
  ❌ Erreurs humaines possibles
  ❌ Non scalable (beaucoup de routes)
  ❌ Maintenance lourde
```

**Cas d'usage** :
- Petits réseaux simples
- Routes critiques fixes
- Connexion à Internet (route par défaut)

### 2. Routage dynamique

**Protocoles automatiques** qui découvrent et maintiennent les routes.

```
Avantages :
  ✅ Adaptation automatique aux pannes
  ✅ Scalable (gros réseaux)
  ✅ Découverte automatique
  ✅ Calcul des meilleurs chemins

Inconvénients :
  ❌ Consomme des ressources (CPU, RAM, BP)
  ❌ Complexe à configurer
  ❌ Peut introduire des problèmes
  ❌ Convergence prend du temps
```

**Cas d'usage** :
- Réseaux moyens à grands
- Réseaux redondants
- Internet (BGP)

**Nous explorerons ces protocoles en détail dans les sections suivantes !**

## Visualisation : de votre PC à Google

```
Vous tapez : www.google.com

┌─────────────────────────────────────────────────────────┐
│ 1. Résolution DNS                                       │
│    www.google.com → 142.250.185.46                      │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 2. Routage local (votre PC)                             │
│    142.250.185.46 sur mon réseau ? NON                  │
│    → Envoyer au gateway 192.168.1.1                     │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 3. Votre box (192.168.1.1)                              │
│    Table : 0.0.0.0/0 → FAI (via NAT)                    │
│    Traduit votre IP privée → IP publique                │
│    → Envoie au routeur FAI                              │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 4-12. Routeurs intermédiaires (FAI, Internet)           │
│    Chaque routeur :                                     │
│      - Consulte sa table de routage                     │
│      - Sélectionne le meilleur next hop                 │
│      - Décrémente le TTL                                │
│      - Transmet le paquet                               │
│    Route optimale calculée par protocoles (BGP, OSPF...)│
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 13. Routeur de Google                                   │
│    142.250.185.46 → Connected sur réseau local          │
│    → Livraison directe au serveur                       │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 14. Serveur Google (142.250.185.46)                     │
│    Reçoit le paquet, traite la requête HTTP             │
│    Prépare la réponse (chemin peut être différent !)    │
└─────────────────────────────────────────────────────────┘

Total : ~10-15 sauts en moyenne
Temps : ~15-30 ms en Europe
```

## Résumé visuel

```
┌──────────────────────────────────────────────────────────┐
│              ROUTAGE - CONCEPTS FONDAMENTAUX             │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  🎯 Définition : Sélection du chemin pour les paquets    │
│                                                          │
│  🔑 Concepts clés :                                      │
│     • Saut (hop) : passage d'un routeur à l'autre        │
│     • Gateway : premier routeur de sortie                │
│     • Next hop : prochain routeur sur le chemin          │
│     • Route par défaut : 0.0.0.0/0 (toutes directions)   │
│                                                          │
│  📊 Décision de routage :                                │
│     1. Consulter la table de routage                     │
│     2. Trouver la correspondance la plus spécifique      │
│     3. Sélectionner le next hop                          │
│     4. Décrémenter le TTL                                │
│     5. Transmettre le paquet                             │
│                                                          │
│  ⚖️ Métriques :                                          │
│     • Valeurs comparant différentes routes               │
│     • RIP : nombre de sauts                              │
│     • OSPF : coût (bande passante)                       │
│     • Plus faible = meilleure                            │
│                                                          │
│  🔄 Routage asymétrique : aller ≠ retour                 │
│                                                          │
│  🚫 TTL empêche les boucles infinies                     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ **Routage** = processus de sélection du **chemin** pour les paquets

✅ **Routeur** = équipement qui prend des **décisions de routage**

✅ **Commutation (L2)** vs **Routage (L3)** : MAC vs IP, local vs inter-réseaux

✅ **Gateway** = premier routeur de sortie de votre réseau

✅ **Next hop** = prochain routeur sur le chemin

✅ **Route par défaut** (0.0.0.0/0) = utilisée quand aucune route spécifique

✅ **Plus spécifique gagne** : /24 prioritaire sur /16 sur /0

✅ **Métrique** = coût d'une route (plus faible = meilleure)

✅ **TTL** empêche les boucles infinies (décrémenté à chaque hop)

✅ **Routage asymétrique** : aller et retour peuvent emprunter des chemins différents

## Pour aller plus loin

Maintenant que vous maîtrisez les concepts fondamentaux du routage, vous êtes prêt pour :

- **Tables de routage** : structure détaillée et manipulation
- **Routage statique vs dynamique** : avantages et inconvénients
- **Protocoles de routage** : RIP, OSPF, BGP
- **Algorithmes de routage** : Dijkstra, Bellman-Ford
- **Optimisation et convergence** : comment les réseaux s'adaptent

---

**💡 Expérience pratique** :
```bash
# Voir votre table de routage locale
# Linux/Mac
ip route show
route -n

# Windows
route print

# Identifier :
# - Votre route par défaut (0.0.0.0)
# - Votre gateway
# - Les routes vers vos réseaux locaux
```

---

*Dans la section suivante, nous allons explorer les tables de routage en détail et apprendre à les lire et les interpréter...*

⏭️ [Tables de routage et algorithmes](/03-couche-internet/11-tables-routage.md)
