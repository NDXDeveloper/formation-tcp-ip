🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.11 Tables de routage et algorithmes

## Introduction : la carte routière du réseau

Imaginez que vous êtes dans une ville inconnue et que vous devez vous rendre à différents endroits. Sans GPS, vous consulteriez une **carte routière** qui vous indique : "Pour aller au musée, prenez la rue principale vers le nord", "Pour aller à la gare, tournez à gauche au prochain carrefour".

Une **table de routage** est exactement cela pour un routeur : c'est sa **carte personnelle** qui lui indique, pour chaque destination possible, **quelle direction prendre** et **par quelle route** (interface réseau) envoyer les paquets.

Mais contrairement à une carte papier statique, les tables de routage sont **dynamiques** et peuvent être mises à jour automatiquement grâce à des **algorithmes intelligents** qui calculent les meilleurs chemins en temps réel.

## Qu'est-ce qu'une table de routage ?

### Définition

**Table de routage** = Base de données locale dans chaque routeur (et chaque ordinateur !) contenant les informations nécessaires pour **acheminer les paquets** vers leur destination.

**Contenu** : Liste de **routes**, chacune spécifiant :
- Quelle(s) destination(s) elle concerne
- Vers où envoyer les paquets (next hop)
- Par quelle interface réseau
- Quel est le "coût" de cette route

### Où trouve-t-on des tables de routage ?

**Partout !**

```
✅ Dans votre PC/smartphone
   → Table simple avec route par défaut

✅ Dans votre box Internet
   → Table avec vos réseaux locaux + route vers FAI

✅ Dans les routeurs de votre FAI
   → Tables avec des milliers de routes

✅ Dans les routeurs d'Internet
   → Tables avec des centaines de milliers de routes (BGP)
```

**Analogie** : Tout le monde a une "carte mentale" de comment se déplacer :
- Vous : "Pour aller au travail, je tourne à gauche"
- Centre de tri postal : "Pour Paris, camion n°3"
- Aéroport : "Pour Tokyo, porte 24"

## Structure d'une entrée de table de routage

Une **route** (ligne dans la table) contient typiquement ces informations :

```
┌──────────────────────────────────────────────────────────┐
│               UNE ENTRÉE DE ROUTE                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  📍 Destination : Vers quel(s) réseau(x) ?               │
│     Exemple : 192.168.10.0/24                            │
│                                                          │
│  😷 Masque : Quelle partie est le réseau ?               │
│     Exemple : 255.255.255.0 ou /24                       │
│                                                          │
│  🚪 Gateway (Next Hop) : Prochain routeur                │
│     Exemple : 10.0.0.2                                   │
│     Spécial : "0.0.0.0" = directement connecté           │
│                                                          │
│  🔌 Interface : Par quelle sortie ?                      │
│     Exemple : eth0, eth1, wlan0                          │
│                                                          │
│  📊 Métrique : Quel est le "coût" ?                      │
│     Exemple : 10, 100, 256                               │
│                                                          │
│  ⏱️ Âge/TTL : Depuis combien de temps ?                  │
│     Exemple : 30 secondes, permanent                     │
│                                                          │
│  📋 Source : Comment apprise ?                           │
│     Exemple : Static, OSPF, RIP, Connected               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Détail de chaque champ

#### 1. Destination (Network)

**Rôle** : Réseau de destination pour lequel cette route est valable.

**Format** : Adresse IP réseau + masque (CIDR)

```
Exemples :
  192.168.1.0/24      → Réseau local
  10.0.0.0/8          → Tout le réseau 10.x.x.x
  172.16.50.0/24      → Sous-réseau spécifique
  0.0.0.0/0           → TOUT (route par défaut)
  8.8.8.8/32          → Un seul hôte (route hôte)
```

**Interprétation** :
```
192.168.1.0/24 signifie :
  "Cette route s'applique à toutes les adresses
   de 192.168.1.0 à 192.168.1.255"
```

#### 2. Masque de sous-réseau (Netmask/Prefix)

**Rôle** : Définir quelle portion de l'adresse IP est le réseau.

**Formats** :
```
Notation classique : 255.255.255.0
Notation CIDR      : /24

Les deux sont équivalents !
```

**Importance** : Permet au routeur de déterminer si une adresse IP correspond à cette route.

#### 3. Gateway (Passerelle / Next Hop)

**Rôle** : Adresse IP du **prochain routeur** vers lequel transmettre les paquets.

**Valeurs possibles** :

```
1. Adresse IP d'un routeur : 10.0.0.2
   → Envoyer le paquet à ce routeur

2. 0.0.0.0 ou "On-link" ou "*"
   → Destination directement connectée
   → Pas de routeur intermédiaire
   → Utiliser ARP directement

3. Gateway même que destination (route hôte)
   → Cas particulier
```

**Exemple** :

```
Route A : 10.0.0.0/24 via 192.168.1.1
  → Pour atteindre 10.0.0.x, passer par 192.168.1.1

Route B : 192.168.1.0/24 via 0.0.0.0
  → Réseau local, livraison directe (pas de gateway)
```

#### 4. Interface de sortie (Interface)

**Rôle** : Par quelle **interface réseau physique** envoyer le paquet.

**Noms courants** :

```
Linux/Unix :
  eth0, eth1, eth2      → Interfaces Ethernet
  wlan0, wlan1          → Interfaces WiFi
  lo                    → Loopback (localhost)
  ppp0                  → Connexion dial-up/VPN
  tun0, tap0            → Tunnels VPN

Windows :
  Local Area Connection
  Ethernet0
  Wi-Fi

Cisco :
  GigabitEthernet0/0
  FastEthernet0/1
  Serial0/0/0
```

**Pourquoi important ?** Un routeur peut avoir plusieurs interfaces réseau :

```
┌─────────────────────────────┐
│        ROUTEUR              │
│                             │
│  eth0 → Réseau A            │
│  eth1 → Réseau B            │
│  eth2 → Internet            │
│                             │
└─────────────────────────────┘

Pour 192.168.1.0/24 → Interface eth0
Pour 10.0.0.0/24    → Interface eth1
Pour 0.0.0.0/0      → Interface eth2
```

#### 5. Métrique (Metric/Cost)

**Rôle** : "Coût" de cette route. **Plus bas = meilleur**.

**Usage** : Quand plusieurs routes mènent à la même destination, le routeur choisit celle avec la métrique la plus faible.

```
Destination : 10.0.0.0/24

Route 1 : via 192.168.1.1, métrique 10
Route 2 : via 192.168.1.2, métrique 5   ← CHOISIE (plus faible)
Route 3 : via 192.168.1.3, métrique 20

Routeur sélectionne : Route 2
```

**Valeurs typiques** :

```
0     : Interface locale (directement connectée)
1     : Route statique par défaut
10    : Route OSPF vers réseau rapide
20    : Route RIP
100   : Route OSPF vers réseau lent
256   : Route apprise dynamiquement avec mauvais coût
```

#### 6. Source de la route (Protocol/Source)

**Rôle** : Comment cette route a-t-elle été apprise ?

**Types** :

```
┌──────────────┬──────────────────────────────────────┐
│ Source       │ Signification                        │
├──────────────┼──────────────────────────────────────┤
│ C (Connected)│ Interface directement connectée      │
│ S (Static)   │ Route configurée manuellement        │
│ R (RIP)      │ Apprise via protocole RIP            │
│ O (OSPF)     │ Apprise via protocole OSPF           │
│ B (BGP)      │ Apprise via protocole BGP            │
│ D (EIGRP)    │ Apprise via protocole EIGRP (Cisco)  │
│ K (Kernel)   │ Route système (Linux)                │
└──────────────┴──────────────────────────────────────┘
```

**Importance** : La source détermine la **distance administrative** (priorité entre protocoles).

## Exemples de tables de routage

### Table de routage sur Linux

```bash
$ ip route show

default via 192.168.1.1 dev eth0 proto dhcp metric 100
10.0.0.0/24 dev eth1 proto kernel scope link src 10.0.0.1
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
```

**Décryptage ligne par ligne** :

```
Ligne 1 : default via 192.168.1.1 dev eth0 proto dhcp metric 100

  default        = 0.0.0.0/0 (route par défaut, toutes destinations)
  via 192.168.1.1 = gateway (prochain saut)
  dev eth0       = interface de sortie
  proto dhcp     = apprise via DHCP
  metric 100     = coût de la route

→ "Pour tout ce qui n'a pas de route spécifique,
   envoyer à 192.168.1.1 via eth0"
```

```
Ligne 2 : 10.0.0.0/24 dev eth1 proto kernel scope link src 10.0.0.1

  10.0.0.0/24    = destination (réseau 10.0.0.x)
  dev eth1       = interface de sortie
  proto kernel   = route système (auto-créée)
  scope link     = directement accessible (pas de gateway)
  src 10.0.0.1   = notre IP sur ce réseau

→ "Pour 10.0.0.x, c'est directement connecté sur eth1"
```

```
Ligne 3 : 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10

  192.168.1.0/24 = destination (votre réseau local)
  dev eth0       = interface de sortie
  scope link     = directement connecté
  src 192.168.1.10 = notre IP sur ce réseau

→ "Pour 192.168.1.x, c'est mon réseau local sur eth0"
```

### Table de routage sur Windows

```powershell
C:\> route print

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination    Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0      192.168.1.1  192.168.1.10     50
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
      192.168.1.0    255.255.255.0         On-link    192.168.1.10    306
    192.168.1.10  255.255.255.255         On-link    192.168.1.10    306
    192.168.1.255  255.255.255.255         On-link    192.168.1.10    306
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
===========================================================================
```

**Décryptage** :

```
Ligne 1 : 0.0.0.0/0 via 192.168.1.1
  Route par défaut → gateway 192.168.1.1
  Metric 50 (relativement prioritaire)

Ligne 2 : 127.0.0.0/8 On-link
  Réseau loopback (localhost)
  On-link = directement accessible, pas de gateway

Ligne 3 : 192.168.1.0/24 On-link
  Réseau local directement connecté

Ligne 4 : 192.168.1.10/32 On-link
  Notre propre IP (route hôte)

Ligne 5 : 192.168.1.255/32 On-link
  Adresse broadcast du réseau local

Ligne 6 : 224.0.0.0/4 On-link
  Plage multicast

Ligne 7 : 255.255.255.255/32 On-link
  Broadcast limité
```

### Table de routage Cisco

```cisco
Router# show ip route

Codes: C - connected, S - static, R - RIP, O - OSPF, B - BGP
       D - EIGRP, EX - EIGRP external

Gateway of last resort is 203.0.113.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 203.0.113.1
C     192.168.1.0/24 is directly connected, GigabitEthernet0/0
C     10.0.0.0/24 is directly connected, GigabitEthernet0/1
O     172.16.0.0/16 [110/20] via 10.0.0.2, 00:05:23, GigabitEthernet0/1
R     192.168.10.0/24 [120/2] via 10.0.0.3, 00:00:15, GigabitEthernet0/1
```

**Décryptage** :

```
Gateway of last resort is 203.0.113.1 to network 0.0.0.0
  → Route par défaut pointe vers 203.0.113.1

S*    0.0.0.0/0 [1/0] via 203.0.113.1
  S    = Route Statique
  *    = Route par défaut (candidate)
  [1/0]= [Distance administrative / Métrique]

C     192.168.1.0/24 is directly connected, GigabitEthernet0/0
  C    = Connected (directement connecté)
  Interface Gi0/0

O     172.16.0.0/16 [110/20] via 10.0.0.2, 00:05:23, Gi0/1
  O    = Route OSPF
  [110/20] = [AD OSPF=110 / Métrique OSPF=20]
  via 10.0.0.2 = next hop
  00:05:23 = apprise il y a 5 min 23 sec

R     192.168.10.0/24 [120/2] via 10.0.0.3, 00:00:15, Gi0/1
  R    = Route RIP
  [120/2] = [AD RIP=120 / 2 sauts]
  Apprise récemment (15 secondes)
```

## Processus de recherche dans la table (Routing Lookup)

### Algorithme de correspondance

Quand un paquet arrive, le routeur doit **trouver la bonne route**. Voici le processus :

```
┌──────────────────────────────────────────────────────────┐
│         ALGORITHME DE ROUTING LOOKUP                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1️⃣ Extraire l'IP de destination du paquet               │
│     Exemple : 172.16.50.100                              │
│                                                          │
│  2️⃣ Parcourir TOUTES les entrées de la table             │
│     Vérifier si l'IP correspond à chaque destination     │
│                                                          │
│  3️⃣ Identifier TOUTES les correspondances                │
│     Plusieurs routes peuvent matcher                     │
│                                                          │
│  4️⃣ Sélectionner la route la PLUS SPÉCIFIQUE             │
│     = Celle avec le masque le plus long                  │
│                                                          │
│  5️⃣ Si égalité de spécificité : comparer les métriques   │
│     Choisir la métrique la plus faible                   │
│                                                          │
│  6️⃣ Si toujours égalité : load balancing                 │
│     Répartir le trafic entre les routes équivalentes     │
│                                                          │
│  7️⃣ Si AUCUNE correspondance : route par défaut          │
│     Utiliser 0.0.0.0/0 si elle existe                    │
│                                                          │
│  8️⃣ Si pas de route par défaut : ÉCHEC                   │
│     Envoyer ICMP Destination Unreachable                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Exemple de lookup détaillé

**Table de routage** :

```
┌────────────────────┬──────────┬────────────┬──────────┐
│ Destination        │ Masque   │ Gateway    │ Métrique │
├────────────────────┼──────────┼────────────┼──────────┤
│ 172.16.50.0        │ /24      │ 10.0.0.2   │ 10       │
│ 172.16.0.0         │ /16      │ 10.0.0.3   │ 20       │
│ 172.0.0.0          │ /8       │ 10.0.0.4   │ 30       │
│ 0.0.0.0            │ /0       │ 10.0.0.1   │ 100      │
└────────────────────┴──────────┴────────────┴──────────┘
```

**Paquet arrive** : Destination = `172.16.50.100`

```
Étape 1 : Tester toutes les routes

172.16.50.100 appartient à 172.16.50.0/24 ?
  Réseau : 172.16.50.0 à 172.16.50.255
  172.16.50.100 dedans ? ✅ OUI

172.16.50.100 appartient à 172.16.0.0/16 ?
  Réseau : 172.16.0.0 à 172.16.255.255
  172.16.50.100 dedans ? ✅ OUI

172.16.50.100 appartient à 172.0.0.0/8 ?
  Réseau : 172.0.0.0 à 172.255.255.255
  172.16.50.100 dedans ? ✅ OUI

172.16.50.100 appartient à 0.0.0.0/0 ?
  Réseau : 0.0.0.0 à 255.255.255.255 (TOUT)
  172.16.50.100 dedans ? ✅ OUI
```

**Étape 2 : Sélectionner la plus spécifique**

```
Correspondances :
  ✅ 172.16.50.0/24  ← /24 (le plus spécifique)
  ✅ 172.16.0.0/16   ← /16
  ✅ 172.0.0.0/8     ← /8
  ✅ 0.0.0.0/0       ← /0 (le moins spécifique)

Règle : Plus le masque est long, plus c'est spécifique

/24 > /16 > /8 > /0

→ CHOISIE : 172.16.50.0/24
```

**Étape 3 : Utiliser cette route**

```
Route sélectionnée : 172.16.50.0/24 via 10.0.0.2

Action :
  1. Next hop = 10.0.0.2
  2. Envoyer le paquet à 10.0.0.2
  3. Décrémenter TTL
  4. Recalculer checksum
  5. Transmettre
```

### Cas d'égalité : même spécificité

```
Table avec deux routes identiques :

┌────────────────────┬──────────┬────────────┬──────────┐
│ Destination        │ Masque   │ Gateway    │ Métrique │
├────────────────────┼──────────┼────────────┼──────────┤
│ 10.0.0.0           │ /24      │ 192.168.1.2│ 10       │
│ 10.0.0.0           │ /24      │ 192.168.1.3│ 20       │
└────────────────────┴──────────┴────────────┴──────────┘

Paquet vers : 10.0.0.50

Correspondances :
  ✅ Les deux routes matchent avec /24 (égalité)

Départage par métrique :
  Route 1 : métrique 10  ← CHOISIE (plus faible)
  Route 2 : métrique 20
```

### ECMP : Equal-Cost Multi-Path

Si plusieurs routes ont **la même spécificité ET la même métrique**, le routeur peut faire du **load balancing** :

```
Table :
┌────────────────────┬──────────┬────────────┬──────────┐
│ Destination        │ Masque   │ Gateway    │ Métrique │
├────────────────────┼──────────┼────────────┼──────────┤
│ 10.0.0.0           │ /24      │ 192.168.1.2│ 10       │
│ 10.0.0.0           │ /24      │ 192.168.1.3│ 10       │ (même!)
└────────────────────┴──────────┴────────────┴──────────┘

Action : Répartir le trafic entre les deux chemins
  - Paquet 1 → 192.168.1.2
  - Paquet 2 → 192.168.1.3
  - Paquet 3 → 192.168.1.2
  - etc.

Méthodes :
  • Round-robin (à tour de rôle)
  • Hash de la connexion (même connexion = même chemin)
  • Per-packet (chaque paquet peut prendre un chemin différent)
```

**Avantage** : Utilisation optimale de la bande passante disponible.

## Algorithmes de routage fondamentaux

Les tables de routage ne sont pas magiques. Elles sont construites par des **algorithmes** qui calculent les meilleurs chemins.

Il existe deux grandes familles d'algorithmes :

```
┌─────────────────────────────────────────────────────────┐
│              TYPES D'ALGORITHMES                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1️⃣ VECTEUR DE DISTANCE (Distance Vector)               │
│     • Chaque routeur connaît la distance vers les       │
│       destinations                                      │
│     • Partage ses distances avec les voisins            │
│     • Algorithme : Bellman-Ford                         │
│     • Exemples : RIP, EIGRP (hybride)                   │
│                                                         │
│  2️⃣ ÉTAT DE LIEN (Link State)                           │
│     • Chaque routeur connaît la topologie complète      │
│     • Calcule lui-même les meilleurs chemins            │
│     • Algorithme : Dijkstra (SPF)                       │
│     • Exemples : OSPF, IS-IS                            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Algorithme de Bellman-Ford (Vecteur de distance)

#### Principe général

**Idée** : "Je ne connais pas tout le réseau, mais je connais mes voisins directs. Ils me disent quelles destinations ils peuvent atteindre et à quel coût."

**Analogie** : Demander son chemin en voyage
```
Vous : "Comment aller à Paris ?"
Voisin A : "Je connais un chemin, 200 km"
Voisin B : "Moi aussi, 150 km"
→ Vous choisissez B (plus court)
```

#### Fonctionnement pas à pas

```
Topologie simple :

R1 ←─(coût 1)─→ R2 ←─(coût 1)─→ R3
│                                 │
└────────(coût 10)────────────────┘
```

**Itération 0** : Chaque routeur connaît seulement ses voisins directs

```
Table R1 :
┌─────────────┬──────────┬───────────┐
│ Destination │ Via      │ Coût      │
├─────────────┼──────────┼───────────┤
│ R1 (moi)    │ Direct   │ 0         │
│ R2          │ Direct   │ 1         │
│ R3          │ Direct   │ 10        │
└─────────────┴──────────┴───────────┘
```

**Itération 1** : Les routeurs échangent leurs tables

```
R2 dit à R1 : "Je peux atteindre R3 avec coût 1"

R1 calcule :
  - Chemin direct vers R3 : coût 10
  - Via R2 : coût vers R2 (1) + coût R2→R3 (1) = 2

2 < 10 → Meilleur chemin trouvé !

Table R1 mise à jour :
┌─────────────┬──────────┬───────────┐
│ Destination │ Via      │ Coût      │
├─────────────┼──────────┼───────────┤
│ R1 (moi)    │ Direct   │ 0         │
│ R2          │ Direct   │ 1         │
│ R3          │ Via R2   │ 2  ← MàJ! │
└─────────────┴──────────┴───────────┘
```

**Convergence** : Après quelques itérations, tous les routeurs connaissent les meilleurs chemins.

#### Formule de Bellman-Ford

```
Pour chaque destination D :
  Coût[D] = min( Coût[voisin V] + Coût[V → D] )
            pour tous les voisins V

En français :
  "Le meilleur coût vers D est le minimum de :
   (coût pour atteindre un voisin)
   + (ce que ce voisin annonce pour atteindre D)"
```

#### Avantages

```
✅ Simple à comprendre et implémenter
✅ Peu gourmand en CPU
✅ Peu de mémoire nécessaire
✅ Bon pour petits réseaux
```

#### Inconvénients

```
❌ Convergence lente (itérations multiples)
❌ Problème du "count to infinity"
❌ Pas de vue d'ensemble du réseau
❌ Sensible aux boucles de routage
❌ Mises à jour périodiques même sans changement
```

#### Le problème "Count to Infinity"

**Scénario** :

```
R1 ─(1)─ R2 ─(1)─ R3 ─(1)─ R4

Tout va bien, R1 peut atteindre R4 avec coût 3 (via R2, R3)

Soudain : Le lien R3-R4 tombe !

┌──────────────────────────────────────────────────────────┐
│ Temps T0 : Lien R3-R4 tombe                              │
├──────────────────────────────────────────────────────────┤
│ R3 : "R4 inaccessible, coût = ∞"                         │
└──────────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────────┐
│ Temps T1 : R2 n'a pas encore reçu la mise à jour         │
├──────────────────────────────────────────────────────────┤
│ R2 dit à R3 : "J'ai une route vers R4, coût 2"           │
│ R3 pense : "Ah super ! Via R2, coût 3"                   │
│ → Mais en réalité, la route de R2 PASSE PAR R3 !         │
│ → BOUCLE DE ROUTAGE                                      │
└──────────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────────┐
│ Temps T2 : R2 reçoit la mise à jour de R3                │
├──────────────────────────────────────────────────────────┤
│ R2 : "R3 dit coût 3, alors mon coût = 4"                 │
│ R2 dit à R1 : "J'ai une route vers R4, coût 4"           │
└──────────────────────────────────────────────────────────┘
                      ↓
┌──────────────────────────────────────────────────────────┐
│ Temps T3 : Et ainsi de suite...                          │
├──────────────────────────────────────────────────────────┤
│ Les coûts augmentent : 3, 4, 5, 6, 7... jusqu'à infini   │
│ = "COUNT TO INFINITY"                                    │
│                                                          │
│ Solution : Limiter l'infini (RIP : max = 16 sauts)       │
└──────────────────────────────────────────────────────────┘
```

**Solutions** :
- **Split Horizon** : Ne pas renvoyer une route par l'interface d'où elle vient
- **Route Poisoning** : Annoncer immédiatement coût = infini
- **Hold-down timers** : Ignorer les mises à jour pendant un temps après une panne

### Algorithme de Dijkstra (État de lien / SPF)

#### Principe général

**Idée** : "Je connais TOUTE la topologie du réseau (tous les routeurs, tous les liens). Je calcule MOI-MÊME le chemin le plus court vers chaque destination."

**Analogie** : GPS moderne
```
Vous avez une carte complète de la ville
Vous voyez TOUS les itinéraires possibles
Vous calculez vous-même le chemin optimal
```

#### Fonctionnement pas à pas

**Topologie** :

```
        (2)
    R1─────R2
    │╲     │
(1) │ ╲(4) │(1)
    │  ╲   │
    R4   ╲ R3
     ╲(3) ╲│(1)
      ╲────R5
```

**But** : R1 veut calculer le chemin le plus court vers tous les autres routeurs.

**Initialisation** :

```
┌──────────┬──────────────┬─────────────┐
│ Routeur  │ Coût depuis  │ Via         │
│          │ R1           │             │
├──────────┼──────────────┼─────────────┤
│ R1       │ 0 (moi)      │ -           │
│ R2       │ ∞            │ ?           │
│ R3       │ ∞            │ ?           │
│ R4       │ ∞            │ ?           │
│ R5       │ ∞            │ ?           │
└──────────┴──────────────┴─────────────┘

Visités : [R1]
Non-visités : [R2, R3, R4, R5]
```

**Itération 1** : Partir de R1, examiner ses voisins

```
Voisins de R1 :
  - R2 : coût 2
  - R4 : coût 1
  - R5 : coût 4

Mise à jour :
┌──────────┬──────────────┬─────────────┐
│ Routeur  │ Coût depuis  │ Via         │
│          │ R1           │             │
├──────────┼──────────────┼─────────────┤
│ R1       │ 0            │ -           │
│ R2       │ 2 ← MàJ      │ Direct      │
│ R3       │ ∞            │ ?           │
│ R4       │ 1 ← MàJ      │ Direct      │
│ R5       │ 4 ← MàJ      │ Direct      │
└──────────┴──────────────┴─────────────┘

Choisir le non-visité avec coût min : R4 (coût 1)
Visités : [R1, R4]
```

**Itération 2** : Examiner les voisins de R4

```
Voisins de R4 :
  - R1 : déjà visité, ignorer
  - R5 : coût depuis R1 via R4 = 1 + 3 = 4

Coût actuel vers R5 : 4
Nouveau coût via R4 : 4
→ Égalité, on garde (ou on peut avoir les deux)

Choisir le non-visité avec coût min : R2 (coût 2)
Visités : [R1, R4, R2]
```

**Itération 3** : Examiner les voisins de R2

```
Voisins de R2 :
  - R1 : déjà visité
  - R3 : coût depuis R1 via R2 = 2 + 1 = 3

Mise à jour R3 :
┌──────────┬──────────────┬─────────────┐
│ R3       │ 3 ← MàJ      │ Via R2      │
└──────────┴──────────────┴─────────────┘

Choisir le non-visité avec coût min : R3 (coût 3)
Visités : [R1, R4, R2, R3]
```

**Itération 4** : Examiner les voisins de R3

```
Voisins de R3 :
  - R2 : déjà visité
  - R5 : coût depuis R1 via R3 = 3 + 1 = 4

Coût actuel vers R5 : 4
Nouveau coût via R3 : 4
→ Égalité (deux chemins équivalents)

Visités : [R1, R4, R2, R3, R5]
Tous visités → FIN
```

**Table finale** :

```
┌──────────┬──────────────┬─────────────────────┐
│ Routeur  │ Coût depuis  │ Chemin optimal      │
│          │ R1           │                     │
├──────────┼──────────────┼─────────────────────┤
│ R1       │ 0            │ -                   │
│ R2       │ 2            │ R1 → R2             │
│ R3       │ 3            │ R1 → R2 → R3        │
│ R4       │ 1            │ R1 → R4             │
│ R5       │ 4            │ R1 → R2 → R3 → R5   │
│          │              │ ou R1 → R4 → R5     │
│          │              │ ou R1 → R5 (direct) │
└──────────┴──────────────┴─────────────────────┘
```

#### Pseudo-code de Dijkstra

```
function Dijkstra(source):
    pour chaque nœud v :
        distance[v] = ∞
        précédent[v] = null
        ajouter v à Q (ensemble non-visités)

    distance[source] = 0

    tant que Q n'est pas vide :
        u = nœud dans Q avec distance[u] minimale
        retirer u de Q

        pour chaque voisin v de u :
            alt = distance[u] + coût(u, v)
            si alt < distance[v] :
                distance[v] = alt
                précédent[v] = u

    retourner distance, précédent
```

#### Avantages

```
✅ Convergence rapide
✅ Vue complète de la topologie
✅ Calcul optimal garanti (chemin le plus court)
✅ Pas de problème de boucle
✅ Mises à jour déclenchées uniquement par changements
✅ Scalable pour grands réseaux
```

#### Inconvénients

```
❌ Plus complexe à implémenter
❌ Plus gourmand en CPU (calcul SPF)
❌ Plus gourmand en mémoire (topologie complète)
❌ Overhead initial plus important (échange de LSAs)
❌ Nécessite synchronisation de la base de données
```

### Comparaison Bellman-Ford vs Dijkstra

```
┌──────────────────────┬────────────────────┬──────────────────┐
│ Critère              │ Bellman-Ford (DV)  │ Dijkstra (LS)    │
├──────────────────────┼────────────────────┼──────────────────┤
│ Connaissance         │ Voisins uniquement │ Topologie entière│
│ Calcul               │ Distribué          │ Local (SPF)      │
│ Convergence          │ Lente              │ Rapide           │
│ Complexité CPU       │ Faible             │ Moyenne/Élevée   │
│ Mémoire              │ Faible             │ Élevée           │
│ Boucles              │ Possibles          │ Impossibles      │
│ Updates              │ Périodiques        │ Déclenchées      │
│ Scalabilité          │ Petits réseaux     │ Grands réseaux   │
│ Exemples protocoles  │ RIP                │ OSPF, IS-IS      │
└──────────────────────┴────────────────────┴──────────────────┘
```

### Visualisation comparative

**Bellman-Ford** :

```
"Je suis aveugle sauf pour mes voisins immédiats.
 Je leur fais confiance pour me dire comment aller partout."

R1: "Hé R2, comment vas-tu vers R5 ?"
R2: "Je vais vers R5 avec coût 3"
R1: "OK, alors moi j'irai vers R5 via toi, coût 1+3=4"
```

**Dijkstra** :

```
"J'ai une carte complète. Je calcule moi-même les meilleurs chemins."

R1: [Regarde la carte complète]
R1: "Je vois 3 chemins possibles vers R5"
R1: [Calcule SPF]
R1: "Le chemin via R2-R3 est optimal (coût 4)"
```

## Types d'entrées de routes

### 1. Routes connectées (Connected)

**Définition** : Réseaux **directement attachés** aux interfaces du routeur.

**Création** : Automatique dès qu'une interface est configurée et active.

```
Configuration :
  Interface eth0 : 192.168.1.1/24

Route créée automatiquement :
  192.168.1.0/24 dev eth0 scope link

Signification : "Tout ce qui est en 192.168.1.x
                 est directement accessible sur eth0"
```

**Caractéristiques** :
- Distance administrative : 0 (priorité maximale)
- Pas de next hop (On-link)
- Ne peuvent pas être supprimées manuellement (sauf en désactivant l'interface)

### 2. Routes statiques (Static)

**Définition** : Routes **configurées manuellement** par l'administrateur.

**Création** : Commande manuelle.

```bash
# Linux
ip route add 10.0.0.0/24 via 192.168.1.1

# Windows
route add 10.0.0.0 mask 255.255.255.0 192.168.1.1

# Cisco
ip route 10.0.0.0 255.255.255.0 192.168.1.1
```

**Caractéristiques** :
- Distance administrative : 1 (très prioritaire)
- Permanentes (sauf suppression manuelle ou interface down)
- Utilisées pour routes critiques, routes par défaut
- Pas d'adaptation automatique aux changements

**Cas d'usage** :

```
✅ Route par défaut vers Internet
   ip route add default via 192.168.1.1

✅ Petits réseaux stables
   Peu de changements, contrôle total souhaité

✅ Routes de secours (floating static)
   Avec métrique élevée, prend le relais si route dynamique tombe
```

### 3. Routes dynamiques (Dynamic)

**Définition** : Routes **apprises automatiquement** via protocoles de routage.

**Types** :
- RIP : Distance administrative 120
- OSPF : Distance administrative 110
- EIGRP : Distance administrative 90
- BGP externe : Distance administrative 20
- BGP interne : Distance administrative 200

**Caractéristiques** :
- Mises à jour automatiques
- S'adaptent aux changements de topologie
- Convergence après pannes
- Overhead de trafic (updates protocole)

## Maintenance de la table de routage

### Ajout manuel de routes (Linux)

```bash
# Route vers un réseau
sudo ip route add 10.0.0.0/24 via 192.168.1.1 dev eth0

# Route par défaut
sudo ip route add default via 192.168.1.1

# Route vers un hôte spécifique
sudo ip route add 8.8.8.8/32 via 192.168.1.1

# Route avec métrique spécifique
sudo ip route add 172.16.0.0/16 via 192.168.1.1 metric 100
```

### Suppression de routes (Linux)

```bash
# Supprimer une route spécifique
sudo ip route del 10.0.0.0/24

# Supprimer la route par défaut
sudo ip route del default
```

### Ajout de routes (Windows)

```powershell
# Route vers un réseau
route add 10.0.0.0 mask 255.255.255.0 192.168.1.1

# Route persistante (survit au redémarrage)
route -p add 10.0.0.0 mask 255.255.255.0 192.168.1.1

# Route par défaut
route add 0.0.0.0 mask 0.0.0.0 192.168.1.1
```

### Suppression de routes (Windows)

```powershell
# Supprimer une route
route delete 10.0.0.0

# Supprimer toutes les routes vers une destination
route delete 10.0.0.0 mask 255.255.255.0
```

### Configuration Cisco

```cisco
! Route statique
ip route 10.0.0.0 255.255.255.0 192.168.1.1

! Route par défaut
ip route 0.0.0.0 0.0.0.0 203.0.113.1

! Route avec distance administrative personnalisée (floating static)
ip route 10.0.0.0 255.255.255.0 192.168.1.2 200

! Supprimer une route
no ip route 10.0.0.0 255.255.255.0 192.168.1.1
```

## Diagnostic et troubleshooting

### Commandes essentielles

**Linux/Mac** :

```bash
# Afficher la table de routage
ip route show
route -n
netstat -rn

# Tester une route spécifique
ip route get 8.8.8.8

# Exemple de sortie :
8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.10

# Tracer le chemin
traceroute google.com
mtr google.com  # MTR = meilleur outil
```

**Windows** :

```powershell
# Afficher la table de routage
route print
netstat -r

# Tester une route
Test-NetRoute -DestinationPrefix 8.8.8.8

# Tracer le chemin
tracert google.com
```

**Cisco** :

```cisco
# Afficher la table de routage
show ip route

# Afficher une route spécifique
show ip route 10.0.0.0

# Afficher uniquement les routes statiques
show ip route static

# Afficher les routes OSPF
show ip route ospf

# Déboguer le processus de routage
debug ip routing
debug ip packet
```

### Problèmes courants

**1. Route manquante**

```
Symptôme : Impossible de joindre un réseau

Diagnostic :
  1. Vérifier la table : ip route show
  2. Chercher la destination : ip route get <IP>
  3. Si aucune route → Ajouter manuellement ou vérifier protocole

Solution :
  sudo ip route add 10.0.0.0/24 via 192.168.1.1
```

**2. Route incorrecte**

```
Symptôme : Trafic part dans la mauvaise direction

Diagnostic :
  ip route get 10.0.0.50
  → Vérifier le next hop retourné

Solution :
  1. Supprimer la mauvaise route
  2. Ajouter la bonne route
```

**3. Métrique sous-optimale**

```
Symptôme : Trafic prend un chemin lent alors qu'un rapide existe

Diagnostic :
  ip route show | grep 10.0.0.0
  → Comparer les métriques

Solution :
  Ajuster les métriques pour favoriser le bon chemin
```

**4. Boucle de routage**

```
Symptôme : Paquets tournent en rond, TTL expires

Diagnostic :
  traceroute google.com
  → Voir si les mêmes IPs reviennent

  1  192.168.1.1
  2  10.0.0.1
  3  10.0.0.2
  4  10.0.0.1  ← Boucle détectée !
  5  10.0.0.2

Solution :
  1. Identifier les routeurs en boucle
  2. Corriger leurs tables de routage
  3. Vérifier les protocoles de routage
```

## Résumé visuel

```
┌──────────────────────────────────────────────────────────┐
│           TABLES DE ROUTAGE - CONCEPTS CLÉS              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  📊 Structure d'une entrée :                             │
│     • Destination + Masque                               │
│     • Gateway (next hop)                                 │
│     • Interface de sortie                                │
│     • Métrique (coût)                                    │
│     • Source (Static, OSPF, RIP...)                      │
│                                                          │
│  🔍 Processus de lookup :                                │
│     1. Trouver toutes les correspondances                │
│     2. Choisir la plus spécifique (masque le plus long)  │
│     3. Si égalité : métrique la plus faible              │
│     4. Si aucune : route par défaut (0.0.0.0/0)          │
│                                                          │
│  🧮 Algorithmes fondamentaux :                           │
│     • Bellman-Ford : Vecteur de distance                 │
│       → Simple, distribué, lent                          │
│     • Dijkstra (SPF) : État de lien                      │
│       → Complexe, centralisé, rapide                     │
│                                                          │
│  📝 Types de routes :                                    │
│     • Connected (AD=0) : Directement attachées           │
│     • Static (AD=1) : Configurées manuellement           │
│     • Dynamic (AD variable) : Protocoles automatiques    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ **Table de routage** = base de données indiquant où envoyer les paquets

✅ Chaque entrée contient : **destination, masque, gateway, interface, métrique**

✅ **Lookup** : recherche la route la **plus spécifique** (masque le plus long)

✅ **Bellman-Ford** : algorithme distribué, simple, pour petits réseaux (RIP)

✅ **Dijkstra (SPF)** : algorithme centralisé, optimal, pour grands réseaux (OSPF)

✅ **Count to infinity** : problème majeur des algorithmes à vecteur de distance

✅ **Routes connectées** (AD=0) sont toujours prioritaires

✅ **Routes statiques** (AD=1) très prioritaires, configuration manuelle

✅ **ECMP** : load balancing entre routes équivalentes

✅ **TTL** protège contre les boucles de routage infinies

## Pour aller plus loin

Maintenant que vous maîtrisez les tables de routage et les algorithmes fondamentaux, vous êtes prêt pour :

- **Protocoles de routage** : RIP, OSPF, EIGRP, BGP en détail
- **Routage statique vs dynamique** : comparaison approfondie
- **Configuration pratique** : mise en œuvre sur équipements réels
- **Optimisation** : techniques avancées de routage
- **Troubleshooting avancé** : résolution de problèmes complexes

---

**💡 Expérience pratique** :
```bash
# Explorer votre table de routage
ip route show  # Linux
route print    # Windows

# Analyser chaque ligne :
# - Quelle est votre route par défaut ?
# - Quels réseaux sont directement connectés ?
# - Vers quel gateway part votre trafic Internet ?

# Tester le lookup
ip route get 8.8.8.8  # Linux
# Quelle route est sélectionnée ? Pourquoi ?
```

---

*Dans la section suivante, nous allons explorer en détail les protocoles de routage dynamique : RIP, OSPF, et BGP...*

⏭️ [Protocoles de routage](/03-couche-internet/12-protocoles-routage.md)
