🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.8 VLAN : segmentation logique des réseaux

## Introduction

Imaginez un grand open space où travaillent des employés de différents départements : comptabilité, marketing, et R&D. Bien qu'ils partagent le même espace physique, vous voudriez que chaque département ait son propre "espace privé" pour leurs discussions et documents. Les **VLANs** font exactement ça pour les réseaux : ils créent des réseaux **logiquement séparés** sur la **même infrastructure physique**.

Les VLANs (Virtual Local Area Networks) sont l'une des technologies les plus puissantes pour organiser, sécuriser et optimiser les réseaux modernes.

## Le problème : Un seul grand réseau

### Sans VLANs : Tout le monde ensemble

Imaginez une entreprise avec un seul switch et 24 ordinateurs :

```
              Switch unique
                 │
    ┌────────────┼────────────┐
    │            │            │
   💼           📊           🔬
Compta       Marketing      R&D
(8 PCs)      (8 PCs)       (8 PCs)
```

**Problèmes** :

```
1. BROADCAST STORM
   ├─→ Chaque broadcast (ARP, DHCP, etc.) va à TOUS
   ├─→ 24 machines reçoivent tous les broadcasts
   └─→ Gaspillage de bande passante

2. SÉCURITÉ
   ├─→ Comptabilité peut "voir" le trafic de R&D
   ├─→ Pas d'isolation entre départements
   └─→ Risque de fuite de données sensibles

3. PERFORMANCES
   ├─→ Un seul domaine de collision
   ├─→ Tous partagent la même bande passante
   └─→ Dégradation avec le nombre d'appareils

4. GESTION
   ├─→ Difficile de gérer 24 machines comme un bloc
   ├─→ Impossible de limiter l'accès par département
   └─→ Configuration monolithique
```

**Analogie** : C'est comme mettre tous les employés dans une seule grande salle. Tout le monde entend tout, les secrets de chacun sont exposés, et le bruit devient insupportable !

### Solution traditionnelle : Switches séparés

```
     Switch Compta    Switch Marketing    Switch R&D
         │                   │                │
        💼                  📊               🔬
```

**Avantages** :
- ✅ Isolation complète
- ✅ Sécurité améliorée

**Inconvénients** :
- ❌ **Coûteux** : 3 switches au lieu d'1
- ❌ **Inflexible** : Déplacer une personne = changer de câble physiquement
- ❌ **Complexe** : Plus d'équipements à gérer
- ❌ **Gaspillage** : Si un switch est plein et l'autre vide, on ne peut pas réutiliser

## La solution : VLANs

### Principe de base

Un **VLAN** crée des réseaux **logiquement séparés** sur un **même switch physique**.

```
              Switch unique (avec VLANs)
                      │
    ┌─────────────────┼─────────────────┐
    │                 │                 │
┌───┴───┐        ┌────┴────┐        ┌───┴────┐
│VLAN 10│        │VLAN 20  │        │VLAN 30 │
│Compta │        │Marketing│        │  R&D   │
└───────┘        └─────────┘        └────────┘
   💼                📊                 🔬
(ports 1-8)      (ports 9-16)      (ports 17-24)
```

**Magie** : Un seul switch, mais **trois réseaux complètement isolés** !

**Analogie** : C'est comme diviser votre open space avec des cloisons invisibles. Physiquement c'est le même espace, mais logiquement ce sont des pièces séparées avec leur propre acoustique !

### Définition formelle

Un **VLAN** (Virtual Local Area Network) est un **domaine de broadcast logique** créé par configuration sur les switches, indépendamment de la topologie physique.

**Caractéristiques** :
- 🏷️ Identifié par un **numéro** (VLAN ID : 1-4094)
- 🔒 **Isolation complète** entre VLANs
- 📡 Les broadcasts d'un VLAN ne traversent **pas** vers un autre VLAN
- 🔧 Configuration **logicielle** (pas de câblage physique)
- 🌉 Communication inter-VLAN nécessite un **routeur**

## Types de VLANs

### 1. VLAN basé sur port (le plus courant)

**Principe** : Chaque port du switch est assigné à un VLAN.

```
Switch 24 ports :

Ports 1-8   → VLAN 10 (Compta)
Ports 9-16  → VLAN 20 (Marketing)
Ports 17-24 → VLAN 30 (R&D)

Port 1 ━━━━━ 💻 (automatiquement dans VLAN 10)
Port 10 ━━━━ 💻 (automatiquement dans VLAN 20)
Port 20 ━━━━ 💻 (automatiquement dans VLAN 30)
```

**Avantages** :
- ✅ Simple à configurer
- ✅ Standard le plus utilisé
- ✅ Performance optimale

**Inconvénient** :
- ⚠️ Pour changer un appareil de VLAN, il faut reconfigurer le port

### 2. VLAN basé sur MAC

**Principe** : L'adresse MAC de l'appareil détermine son VLAN.

```
Table du switch :
┌──────────────────────┬──────────┐
│   MAC Address        │   VLAN   │
├──────────────────────┼──────────┤
│ 00:11:22:33:44:55    │    10    │
│ AA:BB:CC:DD:EE:FF    │    20    │
│ 12:34:56:78:9A:BC    │    30    │
└──────────────────────┴──────────┘

L'appareil AA:BB:CC:DD:EE:FF est TOUJOURS dans VLAN 20,
quel que soit le port où il se branche !
```

**Avantages** :
- ✅ Mobilité : L'appareil garde son VLAN partout
- ✅ Idéal pour les laptops qui se déplacent

**Inconvénients** :
- ❌ Configuration lourde (table MAC à maintenir)
- ❌ Sécurité faible (MAC spoofing possible)
- ❌ Peu utilisé en pratique

### 3. VLAN basé sur protocole

**Principe** : Le type de trafic détermine le VLAN.

```
Trafic IPv4   → VLAN 10
Trafic IPv6   → VLAN 20
Trafic IPX    → VLAN 30 (ancien protocole Novell)
```

**Usage** : Rare aujourd'hui, utilisé dans des cas très spécifiques.

### 4. VLAN basé sur authentification (802.1X)

**Principe** : L'utilisateur s'authentifie, et le serveur décide du VLAN.

```
Utilisateur se connecte
    ↓
Authentification auprès d'un serveur RADIUS
    ↓
Serveur répond : "Cet utilisateur appartient au VLAN 20"
    ↓
Le port est dynamiquement assigné au VLAN 20
```

**Avantages** :
- ✅ Sécurité maximale
- ✅ Flexible : VLAN suit l'utilisateur
- ✅ Centralisé : Gestion via serveur

**Inconvénients** :
- ❌ Complexe à mettre en place
- ❌ Nécessite infrastructure (serveur RADIUS)

**Usage** : Grandes entreprises, campus universitaires

## Tagging 802.1Q : La magie des VLANs

### Le problème du transport

Comment un switch sait-il à quel VLAN appartient une trame ?

```
Trame Ethernet normale (sans VLAN) :
┌─────────┬─────────┬──────────┬─────────┬─────┐
│ MAC Dst │ MAC Src │ EtherType│  Data   │ FCS │
│  6      │  6      │    2     │ 46-1500 │  4  │
└─────────┴─────────┴──────────┴─────────┴─────┘

Problème : Où est l'info du VLAN ? 🤔
```

**Solution : Tag 802.1Q**

### Structure du tag 802.1Q

Un **tag de 4 octets** est inséré dans la trame Ethernet :

```
Trame avec tag 802.1Q (VLAN) :
┌─────────┬─────────┬───────┬──────────┬─────────┬─────┐
│ MAC Dst │ MAC Src │ TAG   │ EtherType│  Data   │ FCS │
│  6      │  6      │  4    │    2     │ 46-1500 │  4  │
└─────────┴─────────┴───────┴──────────┴─────────┴─────┘
                       ↑
              4 octets ajoutés !
```

**Position** : Entre l'adresse MAC source et l'EtherType original.

### Détail du tag 802.1Q (4 octets)

```
┌────────────────────────────────────────────┐
│          TAG 802.1Q (4 octets)             │
├──────────┬──────┬─────┬────────────────────┤
│   TPID   │ PCP  │ DEI │      VID           │
│ 2 octets │3 bits│1 bit│    12 bits         │
└──────────┴──────┴─────┴────────────────────┘

TPID : Tag Protocol Identifier (toujours 0x8100)
PCP  : Priority Code Point (qualité de service)
DEI  : Drop Eligible Indicator (peut être jeté?)
VID  : VLAN Identifier (1-4094)
```

**Le plus important : VID (VLAN ID)**

```
12 bits = 4096 valeurs possibles
    ↓
VLAN ID : 0-4095
    ├─→ 0    : Réservé (priorité seulement)
    ├─→ 1    : VLAN par défaut
    ├─→ 2-4094 : VLANs utilisables
    └─→ 4095 : Réservé
```

### Exemple concret

```
Machine dans VLAN 10 envoie une trame :

AVANT tagging (sur le port access) :
┌─────────────────────────────────────────┐
│ AA:BB:CC | 00:11:22 | 0x0800 | [Data]   │
└─────────────────────────────────────────┘

APRÈS tagging (sur le trunk) :
┌──────────────────────────────────────────────────┐
│ AA:BB:CC | 00:11:22 | 0x8100,VLAN=10 | 0x0800... │
└──────────────────────────────────────────────────┘
                          ↑
                    Tag ajouté
```

### Taille de trame modifiée

**Important** : Le tag ajoute 4 octets !

```
Trame Ethernet standard :
├─→ Minimum : 64 octets
└─→ Maximum : 1518 octets

Trame avec tag 802.1Q :
├─→ Minimum : 68 octets (64 + 4)
└─→ Maximum : 1522 octets (1518 + 4)
```

**Conséquence** : Les switches doivent supporter les trames de 1522 octets (ce que font tous les switches modernes).

## Access Ports vs Trunk Ports

### Concepts fondamentaux

```
┌────────────────────────────────────────────┐
│         ACCESS PORT                        │
├────────────────────────────────────────────┤
│  • Connecte UN appareil final              │
│  • Appartient à UN SEUL VLAN               │
│  • NE tag PAS les trames                   │
│  • L'appareil ne sait pas qu'il est        │
│    dans un VLAN                            │
└────────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│         TRUNK PORT                         │
├────────────────────────────────────────────┤
│  • Connecte des switches entre eux         │
│  • Transporte PLUSIEURS VLANs              │
│  • TAG les trames (802.1Q)                 │
│  • Comme une "autoroute multi-voies"       │
└────────────────────────────────────────────┘
```

### Schéma d'ensemble

```
          SWITCH A                    SWITCH B
             │                            │
      ┌──────┼──────┐              ┌──────┼──────┐
      │      │      │              │      │      │
   [Access][Trunk][Access]     [Access][Trunk][Access]
      │      │      │              │      │      │
     💻     ═══════════════════    💻
   VLAN 10          │             VLAN 20
           (Porte plusieurs VLANs)
```

**Access Ports** :
```
💻 ━━━━━ [Port Access VLAN 10] ━━━━━ Switch
   ↑
Trame normale, pas de tag
L'ordinateur ne sait pas qu'il est dans un VLAN
```

**Trunk Port** :
```
Switch A ═══════════ [Trunk] ═══════════ Switch B
            ↑
    Trames taguées
    Plusieurs VLANs simultanés
```

### Fonctionnement détaillé

#### Access Port - Entrée de trame

```
1. Machine envoie une trame normale (sans tag)
   ┌─────────────────────────────────┐
   │ MAC | MAC | 0x0800 | Data | FCS │
   └─────────────────────────────────┘

2. Access Port reçoit (port configuré en VLAN 10)

3. Switch AJOUTE le tag VLAN 10
   ┌──────────────────────────────────────────┐
   │ MAC | MAC | 0x8100,V=10 | 0x0800 | ...   │
   └──────────────────────────────────────────┘

4. Traitement interne avec tag
```

#### Access Port - Sortie de trame

```
1. Switch a une trame taguée VLAN 10
   ┌──────────────────────────────────────────┐
   │ MAC | MAC | 0x8100,V=10 | 0x0800 | ...   │
   └──────────────────────────────────────────┘

2. Port de sortie est Access VLAN 10

3. Switch RETIRE le tag
   ┌─────────────────────────────────┐
   │ MAC | MAC | 0x0800 | Data | FCS │
   └─────────────────────────────────┘

4. Machine reçoit une trame normale
   (elle ne sait pas qu'il y a eu un tag)
```

#### Trunk Port

```
1. Trame arrive, déjà taguée
   ┌──────────────────────────────────────────┐
   │ MAC | MAC | 0x8100,V=20 | 0x0800 | ...   │
   └──────────────────────────────────────────┘

2. Trunk Port transmet avec le tag intact

3. Switch distant reçoit et lit le tag
   "Ah, c'est pour VLAN 20 !"

4. Achemine vers les ports VLAN 20
```

### Analogie complète

**Access Port** = Porte d'entrée d'un appartement
```
Vous entrez avec vos clés (pas de badge visible)
    ↓
Le building sait automatiquement que vous allez à l'étage 10
    ↓
Vous n'avez pas besoin de savoir que vous êtes dans la "zone 10"
```

**Trunk Port** = Ascenseur
```
L'ascenseur transporte des personnes de TOUS les étages
    ↓
Chaque personne a un badge indiquant son étage (tag)
    ↓
L'ascenseur dépose chacun au bon étage
```

## Native VLAN

### Concept

Le **Native VLAN** est le VLAN **non-tagué** sur un trunk port.

```
Trunk Port :
├─→ VLAN 10 : Taguées avec 0x8100, V=10
├─→ VLAN 20 : Taguées avec 0x8100, V=20
├─→ VLAN 30 : Taguées avec 0x8100, V=30
└─→ VLAN 1  : NON taguées (Native VLAN)
```

**Par défaut : VLAN 1** est le Native VLAN.

### Pourquoi un Native VLAN ?

**Raisons historiques** :
- Compatibilité avec les anciens équipements (qui ne comprennent pas les tags)
- Messages de gestion du switch (CDP, VTP, STP)
- Simplicité pour certains protocoles

**Fonctionnement** :

```
Trame arrive sur un trunk SANS tag :
    ↓
"Pas de tag ? → C'est du Native VLAN"
    ↓
Traité comme VLAN 1 (ou Native VLAN configuré)
```

### Problème de sécurité : VLAN Hopping

**Attaque possible** si Native VLAN mal configuré :

```
Attaquant sur VLAN 1 (Native)
    ↓
Envoie une trame DOUBLE-TAGUÉE :
┌────────────────────────────────────────┐
│ MAC | 0x8100,V=1 | 0x8100,V=20 | ...   │
└────────────────────────────────────────┘
        ↑ Tag 1        ↑ Tag 2

Premier switch :
├─→ Retire le premier tag (Native VLAN)
└─→ Transmet trame avec deuxième tag (VLAN 20)

Résultat : Trame arrive dans VLAN 20 ! 🔓
(alors que l'attaquant est dans VLAN 1)
```

**Protection** :
```
✅ Changer le Native VLAN (ne pas utiliser VLAN 1)
✅ Configurer le même Native VLAN des deux côtés
✅ Ne jamais utiliser le Native VLAN pour des données
✅ Désactiver les ports inutilisés
```

## Communication inter-VLAN

### Le problème

**Les VLANs sont isolés** : Par défaut, VLAN 10 ne peut PAS parler à VLAN 20.

```
VLAN 10 (192.168.10.0/24)    VLAN 20 (192.168.20.0/24)
     💻 A                           💻 B
192.168.10.10                  192.168.20.10

A ping B :
    ❌ IMPOSSIBLE
    (ils sont dans des réseaux différents)
```

**Pourquoi ?** Les VLANs sont des **domaines de broadcast séparés**. C'est comme si c'étaient des réseaux physiquement différents.

### Solution : Routage inter-VLAN

Pour que les VLANs communiquent entre eux, il faut un **routeur** (couche 3).

#### Méthode 1 : Routeur externe (router-on-a-stick)

```
        Routeur
          🌐
          │ (1 interface avec sous-interfaces)
          │
       [Trunk]
          │
        Switch
          │
    ┌─────┼─────┐
    │     │     │
 VLAN 10  │  VLAN 20
   💻     │     💻
```

**Configuration du routeur** :

```
Interface physique : eth0
    ├─→ Sous-interface eth0.10 (VLAN 10)
    │   IP : 192.168.10.1/24
    │
    └─→ Sous-interface eth0.20 (VLAN 20)
        IP : 192.168.20.1/24
```

**Flux de communication** :

```
A (VLAN 10) veut parler à B (VLAN 20) :

1. A envoie vers sa passerelle (192.168.10.1)
2. Trame arrive au switch, taguée VLAN 10
3. Switch envoie au routeur via trunk
4. Routeur reçoit sur eth0.10
5. Routeur route vers eth0.20
6. Routeur renvoie au switch avec tag VLAN 20
7. Switch livre à B dans VLAN 20
```

**Analogie** : Le routeur est comme un traducteur qui parle plusieurs langues (VLANs). Les gens de chaque langue ne se comprennent pas directement, mais le traducteur fait le lien.

#### Méthode 2 : Switch Layer 3 (SVI)

Les **switches de couche 3** peuvent router directement entre VLANs.

```
Switch Layer 3
    │
    ├─ Interface VLAN 10 (SVI)
    │  IP : 192.168.10.1/24
    │
    ├─ Interface VLAN 20 (SVI)
    │  IP : 192.168.20.1/24
    │
    └─ Routage interne (très rapide)

    ┌─────┼─────┐
    │     │     │
 VLAN 10  │  VLAN 20
   💻     │     💻
```

**SVI** (Switch Virtual Interface) : Interface logique représentant un VLAN.

**Avantages** :
- ✅ **Beaucoup plus rapide** (routage hardware)
- ✅ **Pas d'équipement externe** nécessaire
- ✅ **Simplifié** : Tout dans un appareil

**Inconvénient** :
- ❌ Plus cher (switches L3 coûtent plus que switches L2)

## Avantages des VLANs

### 1. Segmentation et organisation

```
Entreprise avec 100 employés :

Sans VLANs :
└─→ 1 gros réseau désorganisé

Avec VLANs :
├─→ VLAN 10 : Direction (5 personnes)
├─→ VLAN 20 : Comptabilité (15 personnes)
├─→ VLAN 30 : Marketing (20 personnes)
├─→ VLAN 40 : R&D (25 personnes)
├─→ VLAN 50 : Production (30 personnes)
└─→ VLAN 99 : Invités (5 personnes)

Organisation claire, gestion facilitée ✓
```

### 2. Sécurité améliorée

```
VLAN 10 (Compta) : Données sensibles
├─→ Isolation complète des autres VLANs
├─→ Pas de broadcast hors VLAN
└─→ Nécessite routeur pour accès (contrôle possible)

VLAN 99 (Invités) : Accès Internet seulement
├─→ Aucun accès aux autres VLANs
├─→ Réseau isolé
└─→ Protection du réseau interne
```

**Analogie** : C'est comme avoir des coffres-forts séparés pour chaque département au lieu d'un seul grand coffre commun.

### 3. Réduction du broadcast

```
Sans VLANs (100 machines) :
└─→ Chaque broadcast va à 100 machines
    Impact : 100× trafic

Avec VLANs (5 VLANs de 20 machines) :
└─→ Chaque broadcast va à 20 machines
    Impact : 20× trafic
    Réduction : 80% de broadcasts ! ✓
```

### 4. Flexibilité

```
Employé déménage de bureau :

Sans VLANs :
├─→ Changer physiquement de switch
├─→ Reconfigurer IP, passerelle
└─→ Temps d'arrêt

Avec VLANs :
├─→ Reconfigurer le port du switch (1 commande)
├─→ L'employé garde sa config
└─→ Quelques secondes de changement ✓
```

### 5. Efficacité des ressources

```
Un switch 48 ports au lieu de plusieurs switches :
├─→ Moins d'équipements
├─→ Moins de consommation électrique
├─→ Moins d'espace rack
└─→ Coûts réduits ✓
```

### 6. QoS (Quality of Service)

```
VLAN 10 (VoIP) : Priorité haute
VLAN 20 (Data) : Priorité normale
VLAN 30 (Backup) : Priorité basse

Le switch priorise le trafic VoIP
    ↓
Appels clairs, pas de coupures ✓
```

## Cas d'usage pratiques

### Entreprise classique

```
┌────────────────────────────────────────────┐
│         RÉSEAU ENTREPRISE                  │
├────────────────────────────────────────────┤
│                                            │
│  VLAN 1   : Management (switches, APs)     │
│  VLAN 10  : Direction                      │
│  VLAN 20  : Comptabilité                   │
│  VLAN 30  : RH                             │
│  VLAN 40  : Marketing                      │
│  VLAN 50  : IT                             │
│  VLAN 60  : Production                     │
│  VLAN 70  : Téléphonie IP (VoIP)           │
│  VLAN 80  : Caméras de surveillance        │
│  VLAN 99  : Invités (Internet seulement)   │
│                                            │
└────────────────────────────────────────────┘
```

### École ou université

```
┌────────────────────────────────────────────┐
│         RÉSEAU UNIVERSITAIRE               │
├────────────────────────────────────────────┤
│                                            │
│  VLAN 10  : Administration                 │
│  VLAN 20  : Enseignants                    │
│  VLAN 30  : Étudiants - Résidence A        │
│  VLAN 31  : Étudiants - Résidence B        │
│  VLAN 40  : Laboratoire informatique       │
│  VLAN 50  : Bibliothèque                   │
│  VLAN 60  : Recherche                      │
│  VLAN 99  : Visiteurs                      │
│                                            │
└────────────────────────────────────────────┘
```

### Hôpital

```
┌────────────────────────────────────────────┐
│         RÉSEAU HOSPITALIER                 │
├────────────────────────────────────────────┤
│                                            │
│  VLAN 10  : Administration                 │
│  VLAN 20  : Dossiers médicaux (CRITIQUE)   │
│  VLAN 30  : Équipements médicaux (IoT)     │
│  VLAN 40  : Personnel soignant             │
│  VLAN 50  : Urgences                       │
│  VLAN 60  : Imagerie (radiologie)          │
│  VLAN 99  : Invités/Patients Wi-Fi         │
│                                            │
└────────────────────────────────────────────┘
```

### Data Center

```
┌────────────────────────────────────────────┐
│         DATA CENTER                        │
├────────────────────────────────────────────┤
│                                            │
│  VLAN 10  : Management (IPMI, iDRAC)       │
│  VLAN 20  : Stockage (SAN)                 │
│  VLAN 30  : vMotion (VMware)               │
│  VLAN 40  : Production Web                 │
│  VLAN 50  : Production Base de données     │
│  VLAN 60  : DMZ (serveurs publics)         │
│  VLAN 70  : Backup                         │
│  VLAN 80  : Test/Dev                       │
│                                            │
└────────────────────────────────────────────┘
```

## Configuration conceptuelle

### Switch manageable requis

**Important** : Les VLANs nécessitent un **switch manageable** (géré).

```
Switch non-manageable :
├─→ Plug-and-play
├─→ Pas de configuration
└─→ ❌ PAS de VLANs possibles

Switch manageable :
├─→ Interface de configuration (Web, CLI)
├─→ Contrôle total
└─→ ✅ VLANs configurables
```

### Étapes de configuration (conceptuel)

#### 1. Créer les VLANs

```
Switch(config)# vlan 10
Switch(config-vlan)# name Comptabilite

Switch(config)# vlan 20
Switch(config-vlan)# name Marketing

Switch(config)# vlan 30
Switch(config-vlan)# name RD
```

#### 2. Assigner les ports aux VLANs (Access)

```
Port 1-8 pour Compta :

Switch(config)# interface range fa0/1-8
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
```

```
Port 9-16 pour Marketing :

Switch(config)# interface range fa0/9-16
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
```

#### 3. Configurer un trunk

```
Port 24 connecte à un autre switch :

Switch(config)# interface fa0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
```

#### 4. Vérification

```
Switch# show vlan brief
Switch# show interfaces trunk
Switch# show interfaces fa0/1 switchport
```

## VLANs et Wi-Fi

### SSID par VLAN

Vous pouvez mapper des **réseaux Wi-Fi** (SSID) à des VLANs :

```
Point d'Accès (AP)
    ├─→ SSID "Entreprise-Employes"  → VLAN 10
    ├─→ SSID "Entreprise-Invites"   → VLAN 99
    └─→ SSID "Entreprise-IoT"       → VLAN 50

Connexion au Wi-Fi :
├─→ Employé se connecte à "Entreprise-Employes"
├─→ Automatiquement dans VLAN 10
└─→ Accès aux ressources internes ✓

└─→ Invité se connecte à "Entreprise-Invites"
    ├─→ Automatiquement dans VLAN 99
    └─→ Internet seulement, isolé du réseau interne ✓
```

### Configuration AP

```
AP Configuration :

WLAN 1:
├─→ SSID : "Entreprise-Employes"
├─→ Sécurité : WPA3-Enterprise
└─→ VLAN : 10

WLAN 2:
├─→ SSID : "Entreprise-Invites"
├─→ Sécurité : WPA2-Personal
└─→ VLAN : 99
```

**Le trunk entre l'AP et le switch** transporte tous les VLANs.

## Bonnes pratiques

### 1. Nommage cohérent

```
✅ Bon :
VLAN 10 : "Direction"
VLAN 20 : "Comptabilite"
VLAN 30 : "Marketing"

❌ Mauvais :
VLAN 10 : "vlan10"
VLAN 20 : "V2"
VLAN 30 : "test"
```

### 2. Plan d'adressage logique

```
VLAN 10 : 192.168.10.0/24
VLAN 20 : 192.168.20.0/24
VLAN 30 : 192.168.30.0/24
...
VLAN 99 : 192.168.99.0/24

Facile à retenir : Le numéro de VLAN = le troisième octet ✓
```

### 3. Sécuriser le Native VLAN

```
❌ Ne pas utiliser VLAN 1 (défaut)
✅ Changer pour un VLAN inutilisé (ex: VLAN 999)
✅ Désactiver les ports inutilisés
```

### 4. Documenter

```
Tenir à jour :
├─→ Liste des VLANs et leur usage
├─→ Plan d'adressage IP
├─→ Mapping ports/VLANs
└─→ Diagramme réseau
```

### 5. VLAN Management séparé

```
VLAN 1 (ou dédié) : UNIQUEMENT pour la gestion
├─→ Accès SSH/HTTPS aux switches
├─→ SNMP monitoring
└─→ Isolé des données utilisateurs
```

### 6. Limiter les VLANs sur les trunks

```
❌ Mauvais :
switchport trunk allowed vlan all

✅ Bon :
switchport trunk allowed vlan 10,20,30
(Seulement les VLANs nécessaires)
```

## Limitations et considérations

### 1. Limite de VLANs

```
802.1Q : 4094 VLANs maximum (VID 1-4094)
    ↓
En pratique :
├─→ Petite entreprise : 5-20 VLANs
├─→ Moyenne entreprise : 20-100 VLANs
└─→ Grande entreprise : 100-500 VLANs

Au-delà de 500 VLANs :
└─→ Envisager VXLAN ou autres technologies
```

### 2. Complexité de gestion

```
Plus de VLANs = Plus de complexité
├─→ Documentation essentielle
├─→ Outils de gestion nécessaires
└─→ Formation du personnel IT
```

### 3. Inter-VLAN routing = bottleneck

```
Tout le trafic inter-VLAN passe par le routeur
    ↓
Si routeur surchargé → performances dégradées
    ↓
Solution : Switch Layer 3 (routage hardware)
```

### 4. VLANs ≠ Sécurité absolue

```
⚠️ VLANs = Isolation logique, PAS cryptage

Pour vraie sécurité :
├─→ Firewall entre VLANs
├─→ ACLs (Access Control Lists)
├─→ Authentification (802.1X)
└─→ Chiffrement des données sensibles
```

## VLANs privés (PVLAN)

### Concept avancé

Les **Private VLANs** permettent d'isoler les ports **au sein d'un même VLAN**.

```
VLAN 20 (normal) :
    💻 A ←→ 💻 B ←→ 💻 C
    (Tous peuvent communiquer)

VLAN 20 (avec PVLAN) :
    💻 A ←X→ 💻 B ←X→ 💻 C
     ↓        ↓        ↓
    🌐 Routeur (seul point commun)

    (Isolation entre machines du même VLAN)
```

**Usage** :
- 🏨 Hôtels : Clients isolés entre eux
- ☕ Wi-Fi public : Utilisateurs isolés
- 🏢 Data centers : Clients hébergés isolés

**Types de ports PVLAN** :
- **Promiscuous** : Communique avec tous (routeur, serveur)
- **Isolated** : Communique uniquement avec promiscuous
- **Community** : Communique dans sa communauté + promiscuous

## Schéma récapitulatif complet

```
┌───────────────────────────────────────────────────────────────┐
│                       VLANs EN ACTION                         │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│                      Routeur (L3)                             │
│                         🌐                                    │
│                          │                                    │
│                    [SVI/Sous-interfaces]                      │
│                     192.168.10.1/24 (VLAN 10)                 │
│                     192.168.20.1/24 (VLAN 20)                 │
│                          │                                    │
│                      [TRUNK]                                  │
│                    (802.1Q Tagged)                            │
│                          │                                    │
│              ┌───────Switch L2──────┐                         │
│              │                      │                         │
│         [Trunk]              [Access Ports]                   │
│              │                      │                         │
│      ┌───────┴────────┐       ┌─────┴──────┐                  │
│      │                │       │            │                  │
│  [VLAN 10]        [VLAN 20]   │            │                  │
│   Ports 1-8      Ports 9-16   │            │                  │
│      │                │       │            │                  │
│     💼              📊        │            │                  │
│   Compta         Marketing    │            │                  │
│ 192.168.10.x   192.168.20.x   │            │                  │
│                               │            │                  │
│  • Isolation complète entre VLANs          │                  │
│  • Communication via routeur seulement     │                  │
│  • Broadcasts confinés au VLAN             │                  │
│  • Flexibilité et sécurité                 │                  │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

1. **VLAN** : Réseau **logiquement séparé** sur infrastructure physique commune.

2. **Identification** : Numéro de VLAN (1-4094), le plus souvent basé sur port.

3. **Tag 802.1Q** : 4 octets ajoutés dans la trame Ethernet pour identifier le VLAN.

4. **Access Port** : Un seul VLAN, pas de tag visible pour l'appareil final.

5. **Trunk Port** : Plusieurs VLANs, trames taguées, connecte des switches.

6. **Native VLAN** : VLAN non-tagué sur trunk (défaut VLAN 1), risque de sécurité.

7. **Isolation** : Les VLANs sont **complètement isolés**, communication nécessite routeur.

8. **Avantages** :
   - Sécurité (isolation)
   - Organisation (segmentation)
   - Performance (réduction broadcast)
   - Flexibilité (gestion simplifiée)

9. **Inter-VLAN routing** : Via routeur externe ou switch L3.

10. **Switch manageable requis** : Les VLANs nécessitent configuration.

---

## Prochaine étape

Félicitations ! Vous avez terminé la couche Accès réseau. Vous comprenez maintenant :
- Ethernet et ses évolutions
- Adresses MAC et leur structure
- Trames et leur format
- Commutation et domaines de collision
- ARP (résolution IP → MAC)
- Wi-Fi et technologies sans fil
- VLANs et segmentation logique

Dans le prochain module, nous allons monter d'une couche et explorer la **couche Internet (IP)**, où nous découvrirons comment les données voyagent à travers différents réseaux pour atteindre leur destination finale !

---


⏭️ [3. La couche Internet (IP)](/03-couche-internet/README.md)
