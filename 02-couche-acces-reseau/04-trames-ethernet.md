🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.4 Trames Ethernet : format et analyse

## Introduction

Imaginez que vous voulez envoyer un cadeau à un ami. Vous ne mettez pas simplement l'objet dans une boîte : vous ajoutez l'adresse du destinataire, votre adresse de retour, peut-être un papier bulle pour protéger le contenu, et vous scellez le tout. Une **trame Ethernet**, c'est exactement ça : l'enveloppe qui emballe vos données pour qu'elles voyagent en sécurité sur le réseau local.

Dans cette section, nous allons décortiquer cette enveloppe et comprendre chaque élément qui la compose.

## Qu'est-ce qu'une trame ?

### Définition

Une **trame** (frame en anglais) est l'unité de données de la couche Accès réseau. C'est le **conteneur** qui transporte les informations d'un appareil à un autre sur un réseau local Ethernet.

**Analogie postale complète** :

```
┌──────────────────────────────────────────────────────┐
│  📮 ENVELOPPE POSTALE                                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  De: Alice (rue des Lilas)       ← Adresse source    │
│  À: Bob (avenue du Parc)         ← Adresse dest.     │
│                                                      │
│  ┌────────────────────────────┐                      │
│  │  [Lettre]                  │  ← Contenu/données   │
│  │  Cher Bob...               │                      │
│  └────────────────────────────┘                      │
│                                                      │
│  Cachet de la poste: ✓         ← Vérification        │
└──────────────────────────────────────────────────────┘
```

### Encapsulation rappel

Souvenez-vous du principe d'encapsulation :

```
Application    :  [Données]
    ↓
Transport      :  [En-tête TCP/UDP] [Données]
    ↓
Internet       :  [En-tête IP] [En-tête TCP/UDP] [Données]
    ↓
Accès réseau   :  [En-tête Ethernet] [En-tête IP] [TCP/UDP] [Données] [Trailer]
                  └────────────────────────────────────────────────────────────┘
                                    TRAME ETHERNET
```

La trame Ethernet est donc la **dernière couche d'emballage** avant la transmission physique.

## Structure d'une trame Ethernet II

Il existe plusieurs formats de trames Ethernet, mais le plus utilisé aujourd'hui est **Ethernet II** (aussi appelé DIX Ethernet, pour DEC-Intel-Xerox). Voici sa structure complète :

```
┌────────────┬────────────┬──────────┬──────────┬─────────┬─────────┬─────┐
│ Préambule  │    SFD     │   MAC    │   MAC    │  Type/  │ Données │ FCS │
│            │            │   Dest.  │  Source  │ Longueur│ Payload │     │
│  7 octets  │  1 octet   │ 6 octets │ 6 octets │ 2 octets│46-1500  │4 oct│
│            │            │          │          │         │ octets  │     │
└────────────┴────────────┴──────────┴──────────┴─────────┴─────────┴─────┘
     ↑            ↑           ↑          ↑          ↑          ↑        ↑
  Synchro    Début de   Qui reçoit ?  Qui envoie? Type de  Contenu  Contrôle
             trame                                protocole           d'erreur
```

**Taille totale** : Minimum 64 octets, Maximum 1518 octets (sans VLAN)

Décortiquons chaque élément !

## 1. Préambule (7 octets)

### Rôle

Le préambule est une **séquence de synchronisation** qui prépare le récepteur à recevoir la trame.

**Contenu** : Une alternance de bits `1` et `0` répétée :
```
10101010 10101010 10101010 10101010 10101010 10101010 10101010
```

**Pourquoi ?**
- ⏱️ **Synchronisation d'horloge** : Permet au récepteur de caler son horloge sur celle de l'émetteur
- 🎯 **Préparation** : Signale "Attention, une trame arrive !"
- 🔧 **Stabilisation** : Donne le temps aux circuits électroniques de se stabiliser

**Analogie** : C'est comme le "1, 2, 3, partez !" avant une course. Tout le monde se prépare et se cale sur le même rythme.

### Particularité

Le préambule **n'est pas considéré comme faisant partie de la trame** par les logiciels réseau. Il est géré uniquement par le matériel (carte réseau).

## 2. SFD - Start Frame Delimiter (1 octet)

### Rôle

Le **délimiteur de début de trame** marque la fin du préambule et le **début réel des données**.

**Contenu** : `10101011`

Remarquez les deux `1` consécutifs à la fin : c'est ce qui le distingue du préambule et signale "C'est parti, les vraies données arrivent maintenant !".

**Analogie** : C'est le coup de pistolet du départ de la course. Après ce signal, c'est du sérieux !

```
Préambule : 10101010 10101010 10101010...  "Préparez-vous..."
SFD       : 10101011                        "TOP ! Départ !"
            └──────┘ ← Ces deux 1 marquent le changement
```

## 3. Adresse MAC de destination (6 octets)

### Rôle

Indique **qui doit recevoir** cette trame.

**Format** : 6 octets en hexadécimal
**Exemple** : `AA:BB:CC:DD:EE:FF`

**Types possibles** :
- **Unicast** : Une seule machine (ex: `00:1A:2B:3C:4D:5E`)
- **Broadcast** : Toutes les machines (`FF:FF:FF:FF:FF:FF`)
- **Multicast** : Un groupe de machines (ex: `01:00:5E:XX:XX:XX`)

**Fonctionnement** :

```
Trame arrive sur le réseau
    ↓
Chaque carte réseau lit la MAC destination
    ↓
┌──────────────────────────────────────┐
│ C'est ma MAC ?                       │
├────────────┬─────────────────────────┤
│ OUI ✓      │ NON ✗                   │
│ Je traite  │ J'ignore (je jette)     │
│ la trame   │ la trame                │
└────────────┴─────────────────────────┘
```

**Exception** : En **mode promiscuous** (utilisé par Wireshark), la carte accepte TOUTES les trames, même celles qui ne lui sont pas destinées. Utile pour l'analyse réseau !

## 4. Adresse MAC source (6 octets)

### Rôle

Indique **qui envoie** cette trame.

**Format** : 6 octets en hexadécimal
**Exemple** : `12:34:56:78:9A:BC`

**Utilité** :
- 📨 Le destinataire sait qui lui écrit
- 🔄 Permet de répondre à l'expéditeur
- 📊 Les switches apprennent la topologie réseau
- 🔍 Aide au dépannage et à l'audit

**Analogie** : C'est l'adresse de retour sur une enveloppe. Si vous voulez répondre, vous savez où envoyer votre réponse !

## 5. EtherType / Longueur (2 octets)

### Rôle

Ce champ a **deux usages possibles** selon sa valeur, ce qui peut être source de confusion !

### Cas 1 : EtherType (Ethernet II) - Valeur ≥ 1536

Indique le **type de protocole** contenu dans les données.

**Valeurs courantes** :

| Valeur hexa | Valeur déc | Protocole |
|-------------|------------|-----------|
| `0x0800` | 2048 | **IPv4** |
| `0x0806` | 2054 | **ARP** |
| `0x86DD` | 34525 | **IPv6** |
| `0x8100` | 33024 | **VLAN** (802.1Q) |
| `0x8863` | 34915 | **PPPoE** Discovery |
| `0x8864` | 34916 | **PPPoE** Session |

**Exemple** :
```
EtherType = 0x0800
    ↓
"Les données qui suivent sont un paquet IPv4"
    ↓
La carte réseau sait qu'elle doit passer la trame au pilote IP
```

### Cas 2 : Longueur (802.3) - Valeur < 1536

Indique la **longueur des données** qui suivent.

**Usage** : Principalement dans l'ancien standard 802.3 (rarement utilisé aujourd'hui).

### Comment distinguer ?

```
┌─────────────────────────────────────────┐
│ Valeur du champ (2 octets)              │
├────────────┬────────────────────────────┤
│ < 1536     │ ≥ 1536                     │
│ (0x0600)   │ (0x0600)                   │
├────────────┼────────────────────────────┤
│ = Longueur │ = Type de protocole        │
│ (802.3)    │ (Ethernet II)              │
└────────────┴────────────────────────────┘
```

**En pratique** : Vous rencontrerez presque toujours des **valeurs ≥ 1536**, donc ce champ indique le type de protocole.

## 6. Données / Payload (46 à 1500 octets)

### Rôle

C'est le **cœur de la trame** : les données réelles transportées !

**Contenu** : Généralement un paquet de la couche supérieure (IP, ARP, etc.)

```
┌────────────────────────────────────────┐
│          PAYLOAD (Données)             │
├────────────────────────────────────────┤
│                                        │
│  ┌────────────────────────────────┐    │
│  │  En-tête IP                    │    │
│  ├────────────────────────────────┤    │
│  │  En-tête TCP/UDP               │    │
│  ├────────────────────────────────┤    │
│  │  Données applicatives          │    │
│  │  (HTTP, DNS, FTP...)           │    │
│  └────────────────────────────────┘    │
│                                        │
└────────────────────────────────────────┘
```

### Tailles contraintes

**Minimum : 46 octets**
- Si les données sont plus petites → **padding** (remplissage avec des zéros)
- Pourquoi 46 ? Pour garantir une taille minimale de trame de 64 octets

**Maximum : 1500 octets**
- C'est le **MTU** (Maximum Transmission Unit) Ethernet standard
- Les données plus grandes doivent être **fragmentées** par la couche IP

**Exemple de padding** :

```
Données réelles : 30 octets
    ↓
Trop petit ! (minimum 46)
    ↓
Ajout de 16 octets de padding (généralement des 0x00)
    ↓
Total : 46 octets ✓

[Données: 30 octets][Padding: 0x00 0x00 0x00... (16 octets)]
```

**Analogie** : C'est comme un colis postal. Il y a une taille minimum (une enveloppe ne peut pas être trop petite) et une taille maximum (un gros paquet doit être divisé en plusieurs colis).

## 7. FCS - Frame Check Sequence (4 octets)

### Rôle

Le **code de vérification** qui détecte les erreurs de transmission.

**Méthode** : CRC-32 (Cyclic Redundancy Check 32 bits)

### Comment ça marche ?

**À l'émission** :
```
1. Calcul d'un CRC sur tous les champs de la trame
   (sauf le préambule et le FCS lui-même)
2. Ajout du résultat (4 octets) à la fin de la trame
```

**À la réception** :
```
1. Recalcul du CRC sur les données reçues
2. Comparaison avec le FCS reçu
   ├─→ Identique ? ✓ Trame valide, on traite
   └─→ Différent ? ✗ Trame corrompue, on jette !
```

**Analogie** : C'est comme un code-barres sur un produit. Si le scanner ne peut pas le lire ou si le code est invalide, le produit est refusé.

### Ce que FCS fait et ne fait pas

**✓ Détecte** :
- Bits inversés (1→0 ou 0→1)
- Pertes de données
- Interférences électromagnétiques
- Erreurs de transmission

**✗ Ne corrige PAS** :
- Si erreur détectée → trame jetée
- C'est aux couches supérieures (TCP) de redemander les données

**Pourquoi ne pas corriger ?**
- Simple et rapide (crucial pour la performance)
- Ethernet assume que le réseau local est fiable
- La correction est coûteuse en calcul

## Schéma récapitulatif annoté

```
TRAME ETHERNET COMPLÈTE (64-1518 octets)
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  ╔═══════════╦═══════╗                                               │
│  ║ Préambule ║  SFD  ║  ← Couche physique uniquement                 │
│  ║ 7 octets  ║ 1 oct ║     (invisible pour les logiciels)            │
│  ╚═══════════╩═══════╝                                               │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    EN-TÊTE ETHERNET (14 octets)              │    │
│  ├──────────────┬──────────────┬────────────────────────────────┤    │
│  │ MAC Dest.    │ MAC Source   │ EtherType                      │    │
│  │ 6 octets     │ 6 octets     │ 2 octets                       │    │
│  │              │              │                                │    │
│  │ Qui reçoit ? │ Qui envoie ? │ Type de protocole (ex: 0x0800) │    │
│  └──────────────┴──────────────┴────────────────────────────────┘    │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    DONNÉES / PAYLOAD                         │    │
│  │                    46-1500 octets                            │    │
│  │                                                              │    │
│  │  Contient généralement :                                     │    │
│  │  - En-tête IP                                                │    │
│  │  - En-tête TCP/UDP                                           │    │
│  │  - Données applicatives                                      │    │
│  │                                                              │    │
│  │  (+ padding si données < 46 octets)                          │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                    FCS (Frame Check Sequence)                │    │
│  │                    4 octets                                  │    │
│  │                                                              │    │
│  │  Checksum CRC-32 pour détecter les erreurs                   │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

## Tailles de trame : minimum et maximum

### Taille minimale : 64 octets

**Pourquoi 64 octets minimum ?**

Cela vient du protocole CSMA/CD et de la détection de collision :

```
Temps nécessaire pour détecter une collision =
    2 × (temps de propagation sur le segment le plus long)
```

**Calcul** :
- Longueur max d'un segment : 2500 mètres (coaxial 10BASE5)
- Vitesse de propagation : ~200 000 km/s
- Temps d'aller-retour : ~25 microsecondes
- À 10 Mbit/s : Au moins 64 octets doivent être émis pendant ce temps

**Composition des 64 octets minimum** :
```
En-tête (14) + Données min (46) + FCS (4) = 64 octets
```

Si vos données font moins de 46 octets → padding obligatoire !

### Taille maximale : 1518 octets (standard)

**Composition** :
```
En-tête (14) + Données max (1500) + FCS (4) = 1518 octets
```

**Le MTU : 1500 octets**

Le **MTU** (Maximum Transmission Unit) est la taille maximale des **données** qu'une trame peut transporter. Pour Ethernet standard, c'est **1500 octets**.

**Pourquoi cette limite ?**
- ⚖️ Compromis entre efficacité et temps de transmission
- 💾 Contraintes mémoire des équipements (années 1980)
- 🔄 Éviter qu'une trame monopolise le réseau trop longtemps

**Jumbo Frames** :

Des trames plus grandes (jusqu'à 9000 octets) existent pour des réseaux spécialisés (data centers) :
```
Trame standard : 1518 octets
Jumbo Frame   : jusqu'à 9216 octets
    ↓
Avantage : Moins d'overhead (moins de trames pour les mêmes données)
Inconvénient : Pas supporté partout, nécessite configuration
```

### Avec VLAN : 1522 octets

Si vous utilisez des VLANs (802.1Q), 4 octets supplémentaires sont ajoutés :

```
┌──────┬──────┬──────┬─────────┬─────┬─────┐
│ Dest │ Src  │ VLAN │ Type    │ Data│ FCS │
│ 6    │ 6    │ 4    │ 2       │ ... │ 4   │
└──────┴──────┴──────┴─────────┴─────┴─────┘
         14         +4   = 18 octets d'en-tête

Total max : 18 + 1500 + 4 = 1522 octets
```

## Exemple concret de trame

Prenons un exemple réel : Un ordinateur A ping un ordinateur B.

### Contexte
- **Ordinateur A** : MAC `00:11:22:33:44:55`, IP `192.168.1.10`
- **Ordinateur B** : MAC `AA:BB:CC:DD:EE:FF`, IP `192.168.1.20`
- **Action** : A envoie une requête ICMP Echo (ping) à B

### La trame (simplifiée en hexa)

```
PRÉAMBULE (non montré, géré par le hardware)

EN-TÊTE ETHERNET (14 octets) :
AA BB CC DD EE FF    ← MAC destination (B)
00 11 22 33 44 55    ← MAC source (A)
08 00                ← EtherType (0x0800 = IPv4)

DONNÉES (46+ octets) :
[En-tête IP : 20 octets]
45 00 00 3C ...      ← Version IPv4, longueur totale, etc.
C0 A8 01 0A          ← IP source : 192.168.1.10
C0 A8 01 14          ← IP dest : 192.168.1.20

[En-tête ICMP + Données : 8+ octets]
08 00 ...            ← Type ICMP (08 = Echo Request)
[Données du ping...]

[Padding si nécessaire]

FCS (4 octets) :
A3 F1 29 8E          ← Checksum calculé (exemple)
```

### Interprétation

```
1. La carte réseau de B reçoit la trame
2. Elle lit : MAC dest = AA:BB:CC:DD:EE:FF
3. "C'est moi !" → Elle accepte la trame
4. Elle vérifie le FCS → ✓ Pas d'erreur
5. Elle lit EtherType = 0x0800 → "C'est de l'IP"
6. Elle passe le contenu (payload) au pilote IP
7. Le pilote IP traite le paquet et voit l'ICMP
8. Il passe à la couche ICMP
9. ICMP répond avec un Echo Reply
```

## Différence entre Ethernet II et 802.3

### Ethernet II (DIX)

**C'est le standard dominant aujourd'hui.**

```
┌─────────┬─────────┬─────────┬─────────┬─────┐
│ MAC Dst │ MAC Src │EtherType│  Data   │ FCS │
│  6      │  6      │  2      │ 46-1500 │  4  │
└─────────┴─────────┴─────────┴─────────┴─────┘
                        ↑
                    Type de protocole (≥ 1536)
```

### IEEE 802.3

**Version standardisée par l'IEEE, moins utilisée.**

```
┌─────────┬─────────┬────────┬──────┬─────────┬─────┐
│ MAC Dst │ MAC Src │ Length │ LLC  │  Data   │ FCS │
│  6      │  6      │  2     │ 3-8  │ 43-1497 │  4  │
└─────────┴─────────┴────────┴──────┴─────────┴─────┘
                        ↑        ↑
                    Longueur   En-tête LLC/SNAP
                    (< 1536)   (indique le protocole)
```

**Différence clé** :
- **Ethernet II** : Champ EtherType indique directement le protocole
- **802.3** : Champ Length + en-tête LLC pour indiquer le protocole

**En pratique** : Ethernet II a gagné. Presque tous les réseaux l'utilisent.

## Analyse avec Wireshark (aperçu théorique)

Wireshark est l'outil de référence pour capturer et analyser les trames. Voici ce que vous verriez :

### Capture d'une trame HTTP

```
Frame 42: 1514 bytes on wire
    Arrival Time: Dec 6, 2025 10:30:15.123456
    Frame Length: 1514 bytes
    Capture Length: 1514 bytes

Ethernet II, Src: 00:11:22:33:44:55, Dst: aa:bb:cc:dd:ee:ff
    Destination: aa:bb:cc:dd:ee:ff (aa:bb:cc:dd:ee:ff)
    Source: 00:11:22:33:44:55 (00:11:22:33:44:55)
    Type: IPv4 (0x0800)

Internet Protocol Version 4
    Version: 4
    Header Length: 20 bytes
    Source: 192.168.1.10
    Destination: 192.168.1.20

Transmission Control Protocol
    Source Port: 54321
    Destination Port: 80 (HTTP)

Hypertext Transfer Protocol
    GET /index.html HTTP/1.1\r\n
    Host: example.com\r\n
    ...
```

**Wireshark décode automatiquement** :
1. La couche Ethernet (MAC, EtherType)
2. La couche IP (adresses IP, protocole)
3. La couche Transport (ports, TCP/UDP)
4. La couche Application (HTTP, DNS, etc.)

C'est comme un oignon : Wireshark épluche chaque couche !

## Inter-Frame Gap (IFG)

Un détail important souvent oublié : les trames ne sont **pas envoyées dos à dos**.

### Définition

L'**Inter-Frame Gap** (IFG) est le **temps d'attente minimum** entre deux trames successives.

**Durée** : 96 bits-temps (= 12 octets-temps)

```
[Trame 1][IFG: 96 bits][Trame 2][IFG: 96 bits][Trame 3]
         └────────────┘         └────────────┘
          Pause obligatoire
```

**Pourquoi ?**
- 🔄 Permet aux équipements de traiter la trame reçue
- ⚡ Évite la saturation des buffers
- 🔧 Temps pour mettre à jour les statistiques, tables, etc.

**Analogie** : C'est comme une respiration entre deux phrases. Cela permet à l'auditeur d'assimiler ce qui vient d'être dit.

### Calcul pour différentes vitesses

| Vitesse | Temps de l'IFG |
|---------|----------------|
| 10 Mbit/s | 9,6 μs |
| 100 Mbit/s | 0,96 μs |
| 1 Gbit/s | 0,096 μs (96 ns) |
| 10 Gbit/s | 9,6 ns |

Plus le réseau est rapide, plus l'IFG est court en durée absolue, mais il représente toujours 96 bits-temps.

## Performance : Overhead Ethernet

### Calcul de l'efficacité

Quelle partie de la bande passante est vraiment utilisée pour VOS données ?

**Pour une trame de taille maximale (1500 octets de données)** :

```
Trame complète :
- Préambule + SFD : 8 octets
- En-tête Ethernet : 14 octets
- Données : 1500 octets
- FCS : 4 octets
- IFG : 12 octets (équivalent)
TOTAL : 1538 octets transmis

Efficacité = 1500 / 1538 = 97,5% ✓
```

**Pour une trame de taille minimale (46 octets de données)** :

```
Trame complète :
- Préambule + SFD : 8 octets
- En-tête Ethernet : 14 octets
- Données : 46 octets
- FCS : 4 octets
- IFG : 12 octets
TOTAL : 84 octets transmis

Efficacité = 46 / 84 = 54,8%
```

**Conclusion** : Les **grandes trames sont plus efficaces** ! C'est pourquoi les Jumbo Frames sont utilisées dans les data centers.

### Overhead en détail

```
Pour 1500 octets de données utiles :

Overhead Ethernet = 38 octets (8+14+4+12)
    ↓
Overhead = 38/1538 = 2,5%
    ↓
Bande passante perdue : 2,5%
Bande passante utile : 97,5%
```

## Cas particuliers et extensions

### Baby Giant Frames

Certains switchs supportent des trames légèrement plus grandes pour accommoder les tags VLAN sans réduire le MTU :

```
Standard : 1518 octets max
Baby Giant : 1522-1526 octets
    ↓
Permet d'ajouter des tags (VLAN, QoS) sans fragmenter
```

### Super Jumbo Frames

Dans certains environnements spécialisés :

```
Standard : 1500 octets MTU
Jumbo : 9000 octets MTU
Super Jumbo : 16000+ octets MTU (très rare)
```

## Erreurs courantes et dépannage

### Problème 1 : Trame trop petite (< 64 octets)

**Nom** : Runt frame

**Causes** :
- Collision mal gérée
- Problème de carte réseau
- Câble défectueux

**Symptôme** : La trame est jetée immédiatement

### Problème 2 : Trame trop grande (> 1518 octets)

**Nom** : Giant frame ou Jabber frame

**Causes** :
- Carte réseau défectueuse (ne s'arrête pas d'émettre)
- Configuration incorrecte (Jumbo Frames non supportées)

**Symptôme** : La trame est jetée ou cause des problèmes sur le réseau

### Problème 3 : FCS invalide

**Nom** : CRC error ou FCS error

**Causes** :
- Interférences électromagnétiques
- Câble endommagé
- Connecteur desserré
- Longueur de câble excessive

**Symptôme** : Trame rejetée, perte de données

### Problème 4 : Late collision

**Définition** : Collision détectée après l'émission de 64 octets

**Causes** :
- Segment trop long
- Trop de répéteurs
- Configuration duplex incorrecte (half vs full)

**Impact** : La trame est perdue et doit être retransmise

## Points clés à retenir

1. **Structure complète** : Préambule (7) + SFD (1) + En-tête (14) + Données (46-1500) + FCS (4)

2. **Tailles** :
   - **Minimum** : 64 octets (avec en-tête et FCS)
   - **Maximum** : 1518 octets (standard), 1522 avec VLAN
   - **MTU** : 1500 octets (données seulement)

3. **Adresses MAC** : Destination (6) + Source (6) = 12 octets

4. **EtherType** : Indique le protocole encapsulé (0x0800 = IPv4, 0x0806 = ARP, 0x86DD = IPv6)

5. **FCS** : Détecte les erreurs avec CRC-32, mais ne les corrige pas

6. **Padding** : Si données < 46 octets → ajout de zéros pour atteindre 46

7. **IFG** : 96 bits-temps de pause entre chaque trame

8. **Efficacité** : Meilleure avec des trames grandes (97,5%) qu'avec des petites (55%)

---

## Prochaine étape

Maintenant que vous comprenez la structure des trames Ethernet, nous allons découvrir comment les **switches** utilisent ces informations pour acheminer intelligemment les trames dans un réseau, et comprendre le concept crucial de **domaines de collision** !

---


⏭️ [Commutation (switching) et domaines de collision](/02-couche-acces-reseau/05-commutation-domaines-collision.md)
