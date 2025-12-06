🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.7 Technologies sans fil (Wi-Fi) et leur intégration

## Introduction

Imaginez un monde où chaque fois que vous voulez utiliser votre téléphone, vous devez le brancher avec un câble. Impensable aujourd'hui ! Le **Wi-Fi** a libéré nos appareils de leurs chaînes, nous permettant de nous connecter à Internet depuis n'importe où dans notre maison, notre bureau, ou notre café préféré.

Mais cette liberté a un prix : le Wi-Fi est plus complexe qu'Ethernet, avec ses propres défis et solutions. Dans cette section, nous allons découvrir comment les ondes radio remplacent les câbles, tout en gardant les mêmes objectifs de communication.

## Wi-Fi : Les bases

### Qu'est-ce que le Wi-Fi ?

**Wi-Fi** (Wireless Fidelity) est une technologie de réseau local **sans fil** utilisant des **ondes radio** pour transmettre les données.

**Analogie** :
- **Ethernet** = Téléphone fixe avec fil
- **Wi-Fi** = Téléphone portable (sans fil)

Les deux permettent de communiquer, mais l'un nécessite un câble, l'autre non !

### Relation avec Ethernet

Wi-Fi fait partie de la **même couche** qu'Ethernet : la couche Accès réseau.

```
┌─────────────────────────────────────────────┐
│   Couche Accès Réseau                       │
├─────────────────────────────────────────────┤
│                                             │
│   Ethernet          Wi-Fi                   │
│   (câblé)          (sans fil)               │
│      │                │                     │
│   Câbles          Ondes radio               │
│                                             │
└─────────────────────────────────────────────┘
```

**Important** : Pour les couches supérieures (IP, TCP, applications), **il n'y a aucune différence** ! Que vous soyez en Ethernet ou Wi-Fi, les paquets IP sont identiques.

```
Application voit :
    "Je veux aller sur google.com"

IP ne sait pas si c'est :
    ━━━ Câble Ethernet
    ou
    ~~~~ Ondes Wi-Fi

C'est transparent !
```

### Avantages et inconvénients

**Avantages du Wi-Fi** :
- ✅ **Mobilité** : Bougez librement dans la zone de couverture
- ✅ **Pas de câbles** : Installation simplifiée, moins de désordre
- ✅ **Flexibilité** : Facile d'ajouter des appareils
- ✅ **Accès partout** : Maison, bureau, espaces publics

**Inconvénients du Wi-Fi** :
- ❌ **Moins rapide** qu'Ethernet (généralement)
- ❌ **Moins fiable** : Interférences, obstacles
- ❌ **Portée limitée** : ~30-100m selon conditions
- ❌ **Sécurité plus complexe** : Les ondes traversent les murs
- ❌ **Latence plus élevée** : Délais légèrement supérieurs

**Comparaison rapide** :

| Critère | Ethernet | Wi-Fi |
|---------|----------|-------|
| **Vitesse max** | 10+ Gbit/s | ~10 Gbit/s (Wi-Fi 6E) |
| **Vitesse réelle** | ~950 Mbit/s (Gigabit) | ~200-600 Mbit/s |
| **Latence** | <1 ms | 2-10 ms |
| **Fiabilité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Portée** | 100m (câble) | 30-100m (air) |
| **Mobilité** | ❌ | ✅ |
| **Interférences** | Rares | Fréquentes |

## Les standards Wi-Fi : La famille 802.11

### IEEE 802.11

Le Wi-Fi est standardisé par l'**IEEE 802.11**. Chaque version améliore vitesse, portée ou efficacité.

```
┌────────────────────────────────────────────────┐
│         ÉVOLUTION DES STANDARDS Wi-Fi          │
├────────────────────────────────────────────────┤
│                                                │
│ 1997 : 802.11 (original)     - 2 Mbit/s        │
│ 1999 : 802.11b               - 11 Mbit/s       │
│ 1999 : 802.11a               - 54 Mbit/s       │
│ 2003 : 802.11g               - 54 Mbit/s       │
│ 2009 : 802.11n (Wi-Fi 4)     - 600 Mbit/s      │
│ 2013 : 802.11ac (Wi-Fi 5)    - 3,5 Gbit/s      │
│ 2019 : 802.11ax (Wi-Fi 6)    - 9,6 Gbit/s      │
│ 2020 : 802.11ax (Wi-Fi 6E)   - 9,6 Gbit/s      │
│ 2024 : 802.11be (Wi-Fi 7)    - 46 Gbit/s       │
│                                                │
└────────────────────────────────────────────────┘
```

### Tableau comparatif détaillé

| Standard | Nom marketing | Année | Fréquence | Débit max théorique | Portée intérieure | Portée extérieure |
|----------|---------------|-------|-----------|---------------------|-------------------|-------------------|
| **802.11b** | - | 1999 | 2,4 GHz | 11 Mbit/s | ~35 m | ~140 m |
| **802.11a** | - | 1999 | 5 GHz | 54 Mbit/s | ~30 m | ~120 m |
| **802.11g** | - | 2003 | 2,4 GHz | 54 Mbit/s | ~38 m | ~140 m |
| **802.11n** | Wi-Fi 4 | 2009 | 2,4 & 5 GHz | 600 Mbit/s | ~70 m | ~250 m |
| **802.11ac** | Wi-Fi 5 | 2013 | 5 GHz | 3,5 Gbit/s | ~35 m | ~100 m |
| **802.11ax** | Wi-Fi 6 | 2019 | 2,4 & 5 GHz | 9,6 Gbit/s | ~75 m | ~300 m |
| **802.11ax** | Wi-Fi 6E | 2020 | 2,4 & 5 & 6 GHz | 9,6 Gbit/s | ~75 m | ~300 m |
| **802.11be** | Wi-Fi 7 | 2024 | 2,4 & 5 & 6 GHz | 46 Gbit/s | ~100 m | ~400 m |

**Note** : Les débits réels sont généralement **30-50% des débits théoriques** à cause des interférences, de la distance, et de l'overhead du protocole.

### Les générations expliquées

#### 802.11b (1999)

**Le pionnier grand public** :
- Première vraie adoption massive
- 2,4 GHz uniquement
- Lent selon les standards actuels
- **Obsolète** aujourd'hui

**Analogie** : C'est comme la première télévision en couleur. Révolutionnaire à l'époque, mais dépassée maintenant.

#### 802.11g (2003)

**L'amélioration** :
- Toujours 2,4 GHz
- Compatibilité avec 802.11b
- Plus rapide (54 Mbit/s)
- A dominé les années 2000

#### 802.11n / Wi-Fi 4 (2009)

**La révolution MIMO** :
- **MIMO** (Multiple Input Multiple Output) : Plusieurs antennes
- **Double bande** : 2,4 GHz ET 5 GHz
- Vitesses bien meilleures
- Encore très répandu aujourd'hui

**Analogie MIMO** : Au lieu d'une seule voie de communication, imaginez plusieurs autoroutes parallèles. Plus de données peuvent circuler simultanément !

```
Une antenne :    ═══════    (1 flux)
Deux antennes :  ═══════    (2 flux simultanés)
                 ═══════
Trois antennes : ═══════    (3 flux simultanés)
                 ═══════
                 ═══════
```

#### 802.11ac / Wi-Fi 5 (2013)

**L'optimisation 5 GHz** :
- 5 GHz uniquement (plus de 2,4 GHz)
- MU-MIMO (Multi-User MIMO) : Communique avec plusieurs appareils en même temps
- Canaux plus larges (jusqu'à 160 MHz)
- Standard actuel dans beaucoup de foyers

**Analogie MU-MIMO** : Avant, le professeur parlait à un élève à la fois. Maintenant, il peut parler à plusieurs élèves simultanément sans qu'ils s'interfèrent.

#### 802.11ax / Wi-Fi 6 et 6E (2019-2020)

**L'efficacité spectrale** :
- **OFDMA** : Meilleure utilisation de la bande passante
- **Wi-Fi 6** : 2,4 + 5 GHz
- **Wi-Fi 6E** : 2,4 + 5 + **6 GHz** (nouvelle bande !)
- Meilleure performance dans les environnements encombrés
- Target Wake Time (TWT) : Économie d'énergie pour IoT

**Pourquoi "6E" ?** :
```
Wi-Fi 6  : Utilise 2,4 GHz + 5 GHz (bandes existantes)
Wi-Fi 6E : Utilise 2,4 GHz + 5 GHz + 6 GHz (nouvelle bande)
           └─→ E = Extended (étendu à 6 GHz)
```

**Avantage de 6 GHz** : Aucun ancien appareil ne l'utilise → **Pas d'interférences** !

#### 802.11be / Wi-Fi 7 (2024)

**Le futur proche** :
- Débits jusqu'à 46 Gbit/s
- Canaux ultra-larges (320 MHz)
- Multi-Link Operation (MLO) : Utilise plusieurs bandes simultanément
- Latence ultra-faible (<5 ms)
- Idéal pour VR, cloud gaming, 8K streaming

**Analogie MLO** : Au lieu de choisir entre l'autoroute A ou B, vous roulez sur les DEUX en même temps !

## Bandes de fréquences

### Les trois bandes Wi-Fi

```
┌────────────────────────────────────────────────────┐
│              BANDES DE FRÉQUENCES                  │
├────────────────────────────────────────────────────┤
│                                                    │
│  2,4 GHz : 2,400 - 2,483 GHz                       │
│  ├─→ Portée : ⭐⭐⭐⭐ (excellente)                │
│  ├─→ Vitesse : ⭐⭐ (faible)                       │
│  ├─→ Interférences : ⭐ (nombreuses)               │
│  └─→ Pénétration : ⭐⭐⭐⭐ (traverse bien)        │
│                                                    │
│  5 GHz : 5,150 - 5,850 GHz                         │
│  ├─→ Portée : ⭐⭐⭐ (bonne)                       │
│  ├─→ Vitesse : ⭐⭐⭐⭐ (élevée)                   │
│  ├─→ Interférences : ⭐⭐⭐ (modérées)             │
│  └─→ Pénétration : ⭐⭐ (affaiblie par obstacles)  │
│                                                    │
│  6 GHz : 5,925 - 7,125 GHz (Wi-Fi 6E/7)            │
│  ├─→ Portée : ⭐⭐ (limitée)                       │
│  ├─→ Vitesse : ⭐⭐⭐⭐⭐ (très élevée)            │
│  ├─→ Interférences : ⭐⭐⭐⭐⭐ (quasi nulles)     │
│  └─→ Pénétration : ⭐ (faible)                     │
│                                                    │
└────────────────────────────────────────────────────┘
```

### Analogie des ondes radio

**Pensez aux fréquences comme à des sons** :

```
2,4 GHz = Graves (basse fréquence)
    ├─→ Portent loin
    ├─→ Traversent bien les obstacles (murs)
    └─→ Mais plus "lents" (moins de données)

5 GHz = Aigus (haute fréquence)
    ├─→ Portent moins loin
    ├─→ Arrêtés par les obstacles
    └─→ Mais plus "rapides" (plus de données)

6 GHz = Ultra-aigus (très haute fréquence)
    ├─→ Portent encore moins loin
    ├─→ Très sensibles aux obstacles
    └─→ Mais ultra-rapides !
```

**Analogie musicale** : Un son grave (contrebasse) traverse les murs et se propage loin. Un son aigu (cymbale) est vite étouffé mais peut transmettre plus de nuances.

### Quand utiliser quelle bande ?

**2,4 GHz** : Idéal pour...
- 📱 Appareils IoT éloignés (sonnettes, capteurs)
- 🏠 Grande portée nécessaire
- 🧱 Beaucoup de murs à traverser
- 🔋 Appareils à batterie (consomme moins)

**5 GHz** : Idéal pour...
- 💻 Ordinateurs portables
- 📺 Streaming vidéo HD/4K
- 🎮 Gaming (latence plus faible)
- 🏢 Même pièce que le routeur

**6 GHz (Wi-Fi 6E/7)** : Idéal pour...
- 🖥️ Stations de travail exigeantes
- 🎮 Gaming compétitif / VR
- 🎬 Streaming 8K
- 👓 Réalité virtuelle/augmentée
- 📡 Aucune interférence nécessaire

### Exemple de configuration domestique

```
Maison à deux étages :

Rez-de-chaussée (routeur ici) :
├─→ 5 GHz : TV, console, PC (proximité)
└─→ 2,4 GHz : Smartphone (mobilité)

Premier étage (éloigné) :
├─→ 2,4 GHz : Tous les appareils (meilleure portée)
└─→ 5 GHz si répéteur/mesh installé

Jardin :
└─→ 2,4 GHz uniquement (portée extérieure)
```

## Canaux Wi-Fi

### Concept de canal

Un **canal** est une subdivision d'une bande de fréquence.

**Analogie** : La bande 2,4 GHz est comme une autoroute. Les canaux sont les **voies** de cette autoroute.

```
Bande 2,4 GHz (autoroute)
┌────────────────────────────────────────┐
│ Canal 1 │ Canal 6 │ Canal 11  │ ...    │
│ ─────── │ ─────── │ ────────  │        │
└────────────────────────────────────────┘
```

### Canaux 2,4 GHz

La bande 2,4 GHz est divisée en **14 canaux** (selon le pays, généralement 11 en Amérique du Nord, 13 en Europe).

**Le problème** : Les canaux se **chevauchent** !

```
Canal 1  : |═══════════|
Canal 2  :    |═══════════|
Canal 3  :       |═══════════|
Canal 4  :          |═══════════|
Canal 5  :             |═══════════|
Canal 6  :                |═══════════|
           └─→ Interférences !
```

**Solution** : Utiliser des canaux **non-chevauchants** : 1, 6, 11

```
Canal 1  : |═══════════|
                           Canal 6  : |═══════════|
                                                     Canal 11 : |═══════════|
                      Pas d'interférence ✓
```

**Conseil pratique** :
```
Si vous avez plusieurs réseaux Wi-Fi proches :
├─→ Réseau A : Canal 1
├─→ Réseau B : Canal 6
└─→ Réseau C : Canal 11

Éviter les canaux 2, 3, 4, 5, 7, 8, 9, 10, 12, 13
(ils causent des interférences)
```

### Canaux 5 GHz

**Beaucoup plus de canaux disponibles** : ~25 canaux (selon le pays)

**Avantage majeur** : Les canaux ne se chevauchent **PAS** !

```
Canal 36 : |════|
Canal 40 :       |════|
Canal 44 :             |════|
Canal 48 :                   |════|
...

Aucun chevauchement ✓
Moins d'interférences ✓
```

**Largeurs de canal variables** :
- **20 MHz** : Standard, compatible avec tous
- **40 MHz** : Double vitesse, deux canaux combinés
- **80 MHz** : Quadruple vitesse (Wi-Fi 5+)
- **160 MHz** : Octuple vitesse (Wi-Fi 6+)

**Compromis** :
```
Canal étroit (20 MHz) :
├─→ ✅ Plus de canaux disponibles
├─→ ✅ Moins d'interférences
└─→ ❌ Débit plus faible

Canal large (160 MHz) :
├─→ ✅ Débit très élevé
├─→ ❌ Peu de canaux disponibles
└─→ ❌ Plus sensible aux interférences
```

### Canaux 6 GHz

**Espace immense** : ~60 canaux non-chevauchants !

```
6 GHz = ELDORADO pour le Wi-Fi
├─→ Beaucoup de canaux
├─→ Larges (jusqu'à 320 MHz en Wi-Fi 7)
├─→ Aucun ancien appareil (pas d'interférences)
└─→ Performances exceptionnelles
```

**Le problème** : Nécessite du matériel récent (Wi-Fi 6E ou 7).

## SSID et réseaux

### Qu'est-ce qu'un SSID ?

**SSID** (Service Set Identifier) = Le **nom** de votre réseau Wi-Fi.

```
Exemple :
SSID : "MonReseauWiFi"
SSID : "CafeLibre-Invites"
SSID : "FreeWiFi"
```

**C'est ce que vous voyez** quand vous cherchez des réseaux disponibles :

```
Réseaux disponibles :
┌─────────────────────────────┐
│ 📶 MonReseauWiFi      🔒    │
│ 📶 VoisinDuDessus     🔒    │
│ 📶 FreeWiFi           🔓    │
│ 📶 CafeLibre-Invites  🔒    │
└─────────────────────────────┘
```

### SSID caché

Vous pouvez **cacher** votre SSID (ne pas le diffuser).

```
SSID visible :
"Salut, je m'appelle MonReseauWiFi, connecte-toi !"

SSID caché :
"Je suis là, mais je ne dis pas mon nom"
└─→ Il faut connaître le nom exact pour se connecter
```

**Sécurité ?** Non, ce n'est **pas une vraie protection** ! Un attaquant peut facilement découvrir les SSID cachés.

**Analogie** : C'est comme ne pas mettre de plaque sur sa porte. Ça n'empêche personne de savoir que vous habitez là, et vos visiteurs ont du mal à vous trouver !

### BSSID

**BSSID** (Basic Service Set Identifier) = L'**adresse MAC** du point d'accès.

```
SSID  : "MonReseauWiFi"         ← Nom (lisible par humain)
BSSID : 00:1A:2B:3C:4D:5E       ← Adresse MAC (identifiant unique)
```

**Pourquoi deux identifiants ?**

```
Plusieurs points d'accès peuvent avoir le MÊME SSID :
├─→ SSID : "EntrepriseXYZ" (partout dans le bâtiment)
│   ├─→ AP étage 1 : BSSID AA:AA:AA:AA:AA:AA
│   ├─→ AP étage 2 : BSSID BB:BB:BB:BB:BB:BB
│   └─→ AP étage 3 : BSSID CC:CC:CC:CC:CC:CC
└─→ Vous voyez un seul réseau, mais 3 points d'accès différents
```

**Roaming** : Votre appareil passe automatiquement d'un BSSID à l'autre en gardant la connexion.

## Modes de fonctionnement

### 1. Mode Infrastructure (le plus courant)

**Définition** : Tous les appareils se connectent à un **point d'accès** (AP - Access Point).

```
        Point d'Accès (AP)
             🔧
         ╱   │   ╲
        ╱    │    ╲
       💻   📱     💻

Tout passe par le point d'accès
A veut parler à B → A → AP → B
```

**Analogie** : C'est comme un standard téléphonique. Tous les appels passent par le standard qui dirige vers la bonne personne.

**Utilisation** :
- 🏠 Wi-Fi à la maison
- 🏢 Wi-Fi au bureau
- ☕ Wi-Fi dans les cafés
- 99% des réseaux Wi-Fi !

### 2. Mode Ad-Hoc (pair-à-pair)

**Définition** : Les appareils se connectent **directement** entre eux, sans point d'accès.

```
    💻 ←~~~~~~~~→ 📱

Communication directe
Pas de point d'accès
```

**Analogie** : C'est comme parler directement à quelqu'un, sans passer par un standard.

**Utilisation** :
- 🎮 Jeux multijoueurs locaux
- 📁 Transfert de fichiers rapide
- 🛠️ Configuration d'équipements
- Rare aujourd'hui

**Limitations** :
- Portée limitée
- Généralement limité à 2-5 appareils
- Pas d'accès Internet (sauf si un appareil partage)

### 3. Wi-Fi Direct

**Évolution moderne** de l'Ad-Hoc, plus simple et plus sécurisé.

```
📱 Smartphone ←~~~~~→ 🖨️ Imprimante
    (connection Wi-Fi Direct)

Pas besoin de routeur !
```

**Utilisation** :
- 🖨️ Impression sans fil
- 📺 Miracast (dupliquer l'écran)
- 🔊 Enceintes Bluetooth/Wi-Fi
- 📱 Transfert smartphone à smartphone

### 4. Mesh (Maillage)

**Définition** : Plusieurs points d'accès qui se **coordonnent** pour créer un réseau unifié.

```
      AP Principal
          🔧
       ╱     ╲
      🔧      🔧  Satellites mesh
     ╱ ╲    ╱  ╲
   💻   📱 💻    📱

Tous les AP collaborent
Un seul SSID visible
Roaming transparent
```

**Avantages** :
- ✅ Couverture optimale (grandes maisons)
- ✅ Un seul réseau visible
- ✅ Roaming intelligent
- ✅ Auto-optimisation

**Exemples** : Google Nest WiFi, Eero, Netgear Orbi, TP-Link Deco

**Analogie** : Au lieu d'avoir un seul émetteur radio puissant, vous avez plusieurs petits émetteurs qui se relaient le signal.

## Sécurité Wi-Fi

### L'évolution de la sécurité

```
┌────────────────────────────────────────────────┐
│         ÉVOLUTION SÉCURITÉ Wi-Fi               │
├────────────────────────────────────────────────┤
│                                                │
│  1997: Aucune (Open)        ❌ DANGEREUX       │
│                                                │
│  1999: WEP                  ❌ CASSÉ           │
│        ├─→ Clés 64/128 bits                    │
│        └─→ Cassable en <5 minutes              │
│                                                │
│  2003: WPA                  ⚠️  OBSOLÈTE       │
│        ├─→ TKIP                                │
│        └─→ Mieux que WEP mais vulnérable       │
│                                                │
│  2004: WPA2                 ✅ BON             │
│        ├─→ AES (chiffrement fort)              │
│        ├─→ Standard pendant 15 ans             │
│        └─→ Vulnérable à KRACK (2017)           │
│                                                │
│  2018: WPA3                 ✅ EXCELLENT       │
│        ├─→ SAE (meilleur échange de clés)      │
│        ├─→ Protection contre attaques offline  │
│        ├─→ Forward secrecy                     │
│        └─→ Standard actuel recommandé          │
│                                                │
└────────────────────────────────────────────────┘
```

### Réseau ouvert (Open)

**Aucun chiffrement, aucune sécurité.**

```
📱 Appareil ~~~~~~~~> 🔧 AP
   (données en clair, tout le monde peut lire)
```

**Utilisation** :
- ☕ Wi-Fi public gratuit (parfois)
- 🚫 **JAMAIS pour usage personnel !**

**Dangers** :
- 👀 Tout le trafic est visible
- 💳 Mots de passe, cartes bancaires exposés
- 🎭 Usurpation d'identité facile

**Protection si obligé d'utiliser** : VPN !

### WEP (Wired Equivalent Privacy)

**Le premier système de sécurité Wi-Fi**, cassé depuis longtemps.

```
WEP = Serrure avec une clé en papier
└─→ N'importe qui peut la crocheter en quelques minutes
```

**Pourquoi cassé ?**
- Failles cryptographiques graves
- Clé récupérable en capturant quelques milliers de paquets
- Outils gratuits pour le casser (Aircrack-ng)

**🚫 Ne JAMAIS utiliser WEP en 2025 !**

### WPA (Wi-Fi Protected Access)

**Amélioration rapide de WEP**, mais toujours vulnérable.

**TKIP** (Temporal Key Integrity Protocol) : Change la clé régulièrement

```
WEP : Même clé tout le temps    ❌
WPA : Clé change périodiquement ✓ (mais pas assez sécurisé)
```

**⚠️ Obsolète, ne plus utiliser !**

### WPA2 (2004-2018)

**Le standard de facto pendant 15 ans.**

**AES** (Advanced Encryption Standard) : Chiffrement militaire

```
WPA2-Personal (PSK) :
├─→ Clé partagée (mot de passe)
├─→ Usage : Maison, petite entreprise
└─→ Sécurité : Bonne si mot de passe fort

WPA2-Enterprise (802.1X) :
├─→ Authentification par utilisateur
├─→ Serveur RADIUS
├─→ Usage : Grandes entreprises
└─→ Sécurité : Excellente
```

**Vulnérabilité KRACK (2017)** :
- Key Reinstallation Attack
- Permet d'intercepter le trafic
- Corrigeable par mise à jour logicielle

**Encore largement utilisé**, mais WPA3 est préférable.

### WPA3 (2018-aujourd'hui)

**Le meilleur standard actuel.**

**Améliorations principales** :

#### 1. SAE (Simultaneous Authentication of Equals)

Remplace le handshake PSK vulnérable.

```
WPA2 : Handshake 4-way (vulnérable)
       ├─→ Attaque par dictionnaire offline possible
       └─→ Si mot de passe faible = compromis

WPA3 : SAE (Dragonfly)
       ├─→ Protection contre attaque offline
       └─→ Même mot de passe faible est mieux protégé
```

#### 2. Forward Secrecy

**Définition** : Même si la clé est compromise, les communications **passées** restent secrètes.

```
Attaquant capture le trafic en 2024
    ↓
En 2025, il obtient le mot de passe Wi-Fi
    ↓
WPA2 : Il peut déchiffrer tout le trafic de 2024 ❌
WPA3 : Le trafic de 2024 reste chiffré ✅
```

#### 3. Protection des réseaux ouverts (OWE)

**Opportunistic Wireless Encryption** : Chiffrement même sans mot de passe !

```
Réseau public "Cafe-WiFi" :
├─→ Pas de mot de passe (accès libre)
├─→ MAIS chiffrement automatique entre vous et l'AP
└─→ Les autres clients ne peuvent pas vous espionner
```

#### 4. 192-bit security (WPA3-Enterprise)

Pour les environnements ultra-sécurisés (gouvernement, militaire).

### Choisir sa sécurité

**Recommandations 2025** :

```
🏠 Usage domestique :
└─→ WPA3-Personal (si routeur compatible)
    ou WPA2-Personal avec mot de passe FORT

🏢 Petite entreprise :
└─→ WPA3-Personal ou WPA2-Enterprise

🏢 Grande entreprise :
└─→ WPA3-Enterprise obligatoire

☕ Accès public :
└─→ WPA3 avec OWE (chiffrement sans mot de passe)
```

**Mot de passe fort** :
```
❌ Faible : "password123", "monwifi"
❌ Moyen  : "MonWiFi2024!"
✅ Fort   : "C0rr3ct-H0rs3-B@tt3ry-St@pl3"
✅ Fort   : 16+ caractères aléatoires générés
```

## CSMA/CA : L'équivalent sans fil de CSMA/CD

### Rappel CSMA/CD (Ethernet)

```
Ethernet (câblé) : CSMA/CD
├─→ Écouter avant de parler
├─→ Si collision PENDANT la transmission
└─→ S'arrêter et réessayer
```

### CSMA/CA (Wi-Fi)

**CA** = Collision **Avoidance** (Évitement, pas détection)

**Pourquoi différent ?**

```
Problème du sans fil :
Impossible de détecter les collisions PENDANT la transmission !
    ↓
Pourquoi ?
    ↓
L'émetteur Wi-Fi est trop "fort" pour entendre les autres
(comme essayer d'entendre un murmure en criant)
    ↓
Solution : ÉVITER les collisions au lieu de les détecter
```

### Fonctionnement CSMA/CA

```
┌─────────────────────────────────────────────┐
│         CSMA/CA (Wi-Fi)                     │
├─────────────────────────────────────────────┤
│                                             │
│  1. Écouter le canal (Carrier Sense)        │
│     ├─→ Canal libre ?                       │
│     │   ├─→ OUI : Continuer                 │
│     │   └─→ NON : Attendre                  │
│     │                                       │
│  2. Attendre DIFS                           │
│     (DCF Interframe Space)                  │
│     ├─→ Petit délai obligatoire             │
│     └─→ Vérifier que le canal est libre     │
│     │                                       │
│  3. Backoff aléatoire                       │
│     ├─→ Attendre un temps aléatoire         │
│     └─→ Éviter que deux stations            │
│         démarrent en même temps             │
│     │                                       │
│  4. Transmettre                             │
│     ├─→ Envoyer la trame complète           │
│     └─→ Impossible de détecter collision    │
│     │                                       │
│  5. Attendre ACK (accusé de réception)      │
│     ├─→ ACK reçu ? ✓ Succès                 │
│     └─→ Pas d'ACK ? ✗ Échec, retransmettre  │
│                                             │
└─────────────────────────────────────────────┘
```

### ACK (Acknowledgement)

**Différence majeure avec Ethernet** : Chaque trame Wi-Fi doit être **accusée** par le destinataire.

```
Émetteur → [Trame de données] → Récepteur
           ↓
Émetteur ← [    ACK    ] ← Récepteur

Si pas d'ACK après timeout :
    ├─→ La trame est perdue
    └─→ Retransmission automatique
```

**Analogie** : C'est comme envoyer un SMS et attendre le "✓✓" de confirmation. Sans confirmation, vous renverrez le message.

### RTS/CTS (option)

**Request To Send / Clear To Send** : Mécanisme optionnel pour grandes trames.

```
A veut envoyer une GRANDE trame à B :

1. A → [RTS petit paquet] → B
   "Je veux t'envoyer des données, OK ?"

2. A ← [CTS petit paquet] ← B
   "OK, vas-y !"

3. A → [DONNÉES] → B
   (Tous les autres ont entendu le CTS et attendent)

4. A ← [ACK] ← B
   "Bien reçu !"
```

**Avantage** : Si collision sur le petit RTS, c'est moins grave qu'une collision sur une grande trame de données.

**Inconvénient** : Overhead (surcharge), donc désactivé par défaut pour petites trames.

## Problèmes spécifiques au sans fil

### 1. Le problème du nœud caché (Hidden Node)

```
Situation :
    A ~~~~~~ AP ~~~~~~ C

    A et C sont HORS DE PORTÉE l'un de l'autre
    Mais les deux communiquent avec AP
```

**Problème** :

```
A écoute → Canal libre (A n'entend pas C)
A transmet vers AP

En même temps :
C écoute → Canal libre (C n'entend pas A)
C transmet vers AP

COLLISION au niveau de AP ! ❌
Mais ni A ni C ne le savent
```

**Analogie** : Deux personnes parlent au même professeur depuis des coins opposés de la salle. Elles ne s'entendent pas mutuellement, mais le professeur reçoit un bruit confus.

**Solution** : RTS/CTS (si activé)

### 2. Le problème du nœud exposé (Exposed Node)

```
Situation :
    A ~~~~~~ B ~~~~~~ C ~~~~~~ D
```

**Problème** :

```
B transmet vers A
    ↓
C entend B et pense : "Canal occupé, je ne peux pas transmettre"
    ↓
Mais C pourrait transmettre vers D sans problème !
(D est hors de portée de B)
    ↓
Sous-utilisation du réseau ❌
```

**Analogie** : Vous voulez parler à quelqu'un à l'autre bout de la pièce, mais vous vous taisez parce que vous entendez quelqu'un parler près de vous (alors que votre conversation n'interférerait pas).

### 3. Fading (évanouissement)

**Les ondes radio s'affaiblissent et fluctuent.**

**Causes** :
- 📏 **Distance** : Plus c'est loin, plus c'est faible
- 🧱 **Obstacles** : Murs, meubles, personnes
- 🌊 **Multipath** : Réflexions, trajets multiples
- ☔ **Conditions météo** : Pluie, humidité (surtout 5 GHz+)

```
Signal parfait :     ████████████
À travers 1 mur :    ██████
À travers 3 murs :   ███
À travers 5 murs :   █
```

**Matériaux et atténuation** :

| Matériau | Atténuation 2,4 GHz | Atténuation 5 GHz |
|----------|---------------------|-------------------|
| Air (espace libre) | Référence | Référence |
| Verre | Faible | Moyenne |
| Bois | Faible-Moyenne | Moyenne |
| Plâtre | Moyenne | Forte |
| Brique | Forte | Très forte |
| Béton | Très forte | Extrême |
| Métal | Extrême | Extrême |

### 4. Interférences

**Sources d'interférences 2,4 GHz** :
- 📡 Autres réseaux Wi-Fi (voisins)
- 🍽️ Fours à micro-ondes
- 📞 Téléphones sans fil
- 🎮 Manettes Bluetooth
- 👶 Babyphones
- 💡 Certaines LED et néons

**Analogie** : C'est comme essayer de parler dans un restaurant bruyant. Plus il y a de bruit ambiant, plus c'est difficile de communiquer.

**Solution** : Utiliser 5 GHz (moins encombré) ou 6 GHz (presque vide).

### 5. Débit asymétrique

**Le débit annoncé n'est jamais atteint en pratique.**

```
Box "Wi-Fi 6 jusqu'à 3 Gbit/s" :
    ↓
Réalité :
├─→ À 2m, sans obstacle : ~800 Mbit/s (27%)
├─→ À 10m, 2 murs : ~200 Mbit/s (7%)
└─→ À 20m, étage différent : ~50 Mbit/s (1,7%)
```

**Causes** :
- 📊 Overhead du protocole (~50%)
- 📏 Distance et obstacles
- 🔀 Interférences
- 👥 Partage avec d'autres appareils

## Intégration avec Ethernet

### Point d'accès (Access Point)

Un AP fait le **pont** entre Wi-Fi et Ethernet.

```
┌─────────────────────────────────────────┐
│        Point d'Accès                    │
├─────────────────────────────────────────┤
│                                         │
│  Wi-Fi        │        Ethernet         │
│  ~~~~~ 📱     │    ━━━━━━━              │
│  ~~~~~ 💻     │         │               │
│               │     🔧 Switch           │
│               │         │               │
│               │     ━━━━━━━             │
│               │         │               │
│               │     🌐 Internet         │
│                                         │
└─────────────────────────────────────────┘
```

**Fonction** : Convertir les trames Wi-Fi ↔ trames Ethernet

**Transparent** : Pour les couches supérieures, c'est le même réseau !

### Exemple de communication mixte

```
A (Wi-Fi) veut envoyer à B (Ethernet) :

1. A crée un paquet IP :
   IP source : 192.168.1.10 (A)
   IP dest : 192.168.1.20 (B)

2. Couche Accès réseau de A :
   ├─→ Trame Wi-Fi
   ├─→ MAC source : A
   ├─→ MAC dest : AP (point d'accès)
   └─→ Envoi via ondes radio

3. Point d'accès reçoit :
   ├─→ Lit le paquet IP
   ├─→ Voit IP dest : 192.168.1.20
   ├─→ Consulte sa table : B est sur port Ethernet 3

4. Point d'accès convertit :
   ├─→ Nouvelle trame Ethernet
   ├─→ MAC source : AP
   ├─→ MAC dest : B
   └─→ Envoi via câble Ethernet

5. B reçoit :
   ├─→ Traite normalement
   └─→ B ne sait même pas que A est en Wi-Fi !
```

**Magie** : IP, TCP, HTTP ne voient **aucune différence** !

## Roaming (itinérance)

### Définition

**Roaming** : Se déplacer entre différents points d'accès sans perdre la connexion.

```
Bureau, 3 étages, même SSID "EntrepriseXYZ" :

Étage 1 : AP1 (BSSID: AA:AA:AA:AA:AA:AA)
Étage 2 : AP2 (BSSID: BB:BB:BB:BB:BB:BB)
Étage 3 : AP3 (BSSID: CC:CC:CC:CC:CC:CC)

Vous montez les escaliers avec votre laptop :
├─→ Connecté à AP1 (signal fort)
├─→ Vous montez...
├─→ Signal AP1 s'affaiblit, signal AP2 augmente
├─→ Basculement automatique vers AP2
├─→ Connexion maintenue !
└─→ Appel Zoom continue sans interruption
```

### Types de roaming

#### 1. Fast Roaming (802.11r)

**Standard** : Basculement <50 ms (imperceptible)

```
Sans 802.11r :
├─→ Déconnexion de AP1
├─→ Scan des AP disponibles
├─→ Authentification complète sur AP2
├─→ Association
└─→ Total : 200-500 ms (interruption notable)

Avec 802.11r :
├─→ Pré-authentification en arrière-plan
├─→ Basculement instantané
└─→ Total : <50 ms (imperceptible)
```

**Usage** : VoIP, visioconférence, jeux en ligne

#### 2. Roaming intelligent (Mesh / Contrôleur)

Les systèmes modernes (Mesh, contrôleurs d'entreprise) gèrent intelligemment :

```
Critères de basculement :
├─→ Force du signal (RSSI)
├─→ Charge de l'AP
├─→ Bande passante disponible
└─→ Décision optimale pour l'utilisateur
```

## Performance et optimisation

### Facteurs influençant la vitesse

```
┌────────────────────────────────────────────┐
│   FACTEURS DE PERFORMANCE Wi-Fi            │
├────────────────────────────────────────────┤
│                                            │
│  1. Standard (802.11n/ac/ax)    ⭐⭐⭐⭐⭐ │
│  2. Bande (2,4 vs 5 vs 6 GHz)   ⭐⭐⭐⭐   │
│  3. Largeur de canal             ⭐⭐⭐⭐  │
│  4. Distance                     ⭐⭐⭐⭐  │
│  5. Obstacles                    ⭐⭐⭐⭐  │
│  6. Interférences                ⭐⭐⭐    │
│  7. Nombre d'appareils           ⭐⭐⭐    │
│  8. Qualité du routeur/AP        ⭐⭐⭐    │
│                                            │
└────────────────────────────────────────────┘
```

### Conseils d'optimisation

**Position du routeur** :
```
✅ Centre de la maison
✅ En hauteur (étagère, meuble)
✅ Dégagé (pas dans un placard)
✅ Loin des interférences (micro-ondes)

❌ Coin de la maison
❌ Au sol
❌ Dans un meuble fermé
❌ Près du micro-ondes
```

**Configuration** :
```
✅ WPA3 ou WPA2 (jamais WEP/WPA)
✅ Canaux non-chevauchants (1, 6, 11)
✅ Utiliser 5 GHz si possible
✅ Désactiver les vieux standards (802.11b)
✅ Firmware à jour

❌ Réseau ouvert sans sécurité
❌ Canal auto (souvent mauvais choix)
❌ Seulement 2,4 GHz
❌ Tous les standards activés
❌ Firmware obsolète
```

**Extensions** :
```
Petite surface (<100m²) : Routeur seul suffit
Moyenne surface (100-200m²) : Répéteur Wi-Fi ou CPL
Grande surface (200m²+) : Système Mesh
Multi-étages : Mesh recommandé
```

## Points clés à retenir

1. **Wi-Fi = Réseau local sans fil** utilisant des ondes radio au lieu de câbles.

2. **Standards** : 802.11n (Wi-Fi 4), 802.11ac (Wi-Fi 5), 802.11ax (Wi-Fi 6/6E), 802.11be (Wi-Fi 7).

3. **Bandes** :
   - 2,4 GHz : Portée ++, vitesse -
   - 5 GHz : Portée -, vitesse ++
   - 6 GHz : Portée -, vitesse +++, interférences ∅

4. **Canaux** : Utiliser 1, 6, 11 en 2,4 GHz (non-chevauchants).

5. **SSID** : Nom du réseau / **BSSID** : Adresse MAC de l'AP.

6. **Modes** : Infrastructure (AP central), Ad-Hoc (pair-à-pair), Mesh (maillage).

7. **Sécurité** : WPA3 > WPA2 >> WPA > WEP. Ne jamais utiliser réseau ouvert pour usage personnel.

8. **CSMA/CA** : Évitement de collision avec ACK obligatoire (différent d'Ethernet).

9. **Problèmes sans fil** : Nœud caché, fading, interférences, débit variable.

10. **Transparent pour IP** : Les couches supérieures ne voient pas de différence entre Wi-Fi et Ethernet.

---

## Prochaine étape

Maintenant que vous maîtrisez les technologies câblées (Ethernet) et sans fil (Wi-Fi), nous allons découvrir comment **segmenter logiquement** un réseau physique en plusieurs réseaux virtuels avec les **VLANs** !

---


⏭️ [VLAN : segmentation logique des réseaux](/02-couche-acces-reseau/08-vlan.md)
