🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.5 Commutation (switching) et domaines de collision

## Introduction

Imaginez une salle de classe où tout le monde doit lever la main et attendre son tour pour parler. C'est épuisant et inefficace ! Maintenant, imaginez que chaque élève puisse avoir une conversation privée avec son voisin sans déranger les autres. C'est exactement le changement révolutionnaire qu'ont apporté les **switches** (commutateurs) aux réseaux Ethernet.

Dans cette section, nous allons découvrir comment les switches ont transformé les réseaux d'autoroutes encombrées en systèmes de communication intelligents et efficaces.

## Rappel : Le problème des hubs

### Comment fonctionnait un hub ?

Un **hub** (concentrateur) était un équipement réseau simple mais primitif :

```
         Hub (répéteur multiports)
            ┌───────┐
     ┌──────┤   🔊  ├──────┐
     │      └───────┘      │
     │          │          │
    💻          💻         💻
    A           B          C
```

**Fonctionnement** :
```
1. A envoie une trame vers B
2. Le hub RÉPÈTE sur TOUS les ports
3. A, B et C reçoivent TOUS la trame
4. Seul B la traite, A et C l'ignorent
```

**Analogie** : C'est comme utiliser un mégaphone dans une pièce. Tout le monde entend, même ceux qui ne sont pas concernés.

### Les problèmes des hubs

- ❌ **Gaspillage de bande passante** : Tout le monde reçoit toutes les trames
- ❌ **Collisions** : Si A et C parlent en même temps → collision !
- ❌ **Sécurité faible** : Tout le monde peut "écouter" tout le trafic
- ❌ **Performance dégradée** : Plus il y a d'appareils, pire c'est
- ❌ **Half-duplex obligatoire** : Impossible d'envoyer et recevoir simultanément

**Domaine de collision unique** : Tous les appareils partagent le même "espace de discussion".

## Le switch : Une révolution

### Qu'est-ce qu'un switch ?

Un **switch** (commutateur en français) est un équipement réseau **intelligent** qui :
- Apprend où se trouve chaque appareil
- Envoie les trames **uniquement au destinataire**
- Crée des **connexions dédiées** entre les appareils

```
         Switch (intelligent)
            ┌───────┐
     ┌──────┤  🧠   ├──────┐
     │      └───────┘      │
     │          │          │
    💻         💻         💻
    A          B          C
```

**Fonctionnement** :
```
1. A envoie une trame vers B
2. Le switch LIT l'adresse MAC de destination
3. Le switch envoie UNIQUEMENT vers le port de B
4. C ne reçoit RIEN (et peut communiquer avec un autre appareil)
```

**Analogie** : C'est comme un standard téléphonique intelligent. Quand A appelle B, seuls A et B sont connectés. C peut avoir sa propre conversation en parallèle.

## Le cœur du switch : La table d'adresses MAC

### Structure de la table

Le switch maintient une **table d'adresses MAC** (aussi appelée table CAM - Content Addressable Memory) :

```
┌──────────────────────┬──────────┬────────────┐
│   Adresse MAC        │   Port   │    Âge     │
├──────────────────────┼──────────┼────────────┤
│ 00:11:22:33:44:55    │    1     │   45 sec   │
│ AA:BB:CC:DD:EE:FF    │    3     │  120 sec   │
│ 12:34:56:78:9A:BC    │    5     │   10 sec   │
│ FF:EE:DD:CC:BB:AA    │    2     │  200 sec   │
└──────────────────────┴──────────┴────────────┘
```

**Signification** :
- **Adresse MAC** : Identifiant unique de l'appareil
- **Port** : Sur quel port physique l'appareil est connecté
- **Âge** : Depuis combien de temps cette entrée est connue

### Le processus d'apprentissage

Le switch **apprend automatiquement** où se trouvent les appareils. Voici comment :

#### Étape 1 : Démarrage (table vide)

```
Switch vient d'être allumé
Table d'adresses : [ VIDE ]

Port 1: 💻 A (MAC: AA:AA:AA:AA:AA:AA)
Port 2: 💻 B (MAC: BB:BB:BB:BB:BB:BB)
Port 3: 💻 C (MAC: CC:CC:CC:CC:CC:CC)

Le switch ne sait pas encore qui est où !
```

#### Étape 2 : Première trame (A → B)

```
A envoie une trame vers B
┌─────────────────────────────────┐
│ MAC Source: AA:AA:AA:AA:AA:AA   │ ← Le switch lit ça !
│ MAC Dest:   BB:BB:BB:BB:BB:BB   │
│ Données: "Salut B!"             │
└─────────────────────────────────┘

🧠 Le switch pense :
   "Cette trame vient du port 1"
   "La source est AA:AA:AA:AA:AA:AA"
   "Donc A est sur le port 1 !"

Table mise à jour :
┌──────────────────────┬──────┐
│ AA:AA:AA:AA:AA:AA    │  1   │ ← Nouvelle entrée !
└──────────────────────┴──────┘
```

**Mais le switch ne sait pas où est B !**

```
Action du switch :
1. Apprentissage de A ✓
2. Destination B inconnue ?
3. → FLOODING (envoi sur TOUS les ports sauf le port source)

Port 1: (source, pas d'envoi)
Port 2: ← Envoi vers B
Port 3: ← Envoi vers C aussi (au cas où)
```

#### Étape 3 : Réponse de B (B → A)

```
B répond à A
┌─────────────────────────────────┐
│ MAC Source: BB:BB:BB:BB:BB:BB   │ ← Le switch lit ça !
│ MAC Dest:   AA:AA:AA:AA:AA:AA   │
│ Données: "Salut A!"             │
└─────────────────────────────────┘

🧠 Le switch pense :
   "Cette trame vient du port 2"
   "La source est BB:BB:BB:BB:BB:BB"
   "Donc B est sur le port 2 !"

Table mise à jour :
┌──────────────────────┬──────┐
│ AA:AA:AA:AA:AA:AA    │  1   │
│ BB:BB:BB:BB:BB:BB    │  2   │ ← Nouvelle entrée !
└──────────────────────┴──────┘

Cette fois, le switch CONNAÎT A :
→ Envoi UNIQUEMENT vers le port 1 (pas de flooding)
```

#### Étape 4 : Communications futures (A ↔ B)

```
Maintenant que les deux sont connus :

A → B : Switch envoie directement vers port 2
B → A : Switch envoie directement vers port 1

C peut communiquer avec quelqu'un d'autre en même temps !
Pas d'interférence, pas de collision.
```

### Schéma complet du processus

```
┌─────────────────────────────────────────────────────────┐
│         PROCESSUS D'APPRENTISSAGE DU SWITCH             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Trame reçue sur un port                                │
│         ↓                                               │
│  ┌──────────────────────────────────┐                   │
│  │ Lire l'adresse MAC SOURCE        │                   │
│  └──────────────┬───────────────────┘                   │
│                 ↓                                       │
│  ┌──────────────────────────────────┐                   │
│  │ MAC source dans la table ?       │                   │
│  └────┬─────────────────────┬───────┘                   │
│       │ NON                 │ OUI                       │
│       ↓                     ↓                           │
│  Ajouter entrée        Rafraîchir âge                   │
│  (MAC, Port, Âge=0)                                     │
│                                                         │
│         ↓                                               │
│  ┌──────────────────────────────────┐                   │
│  │ Lire l'adresse MAC DESTINATION   │                   │
│  └──────────────┬───────────────────┘                   │
│                 ↓                                       │
│  ┌──────────────────────────────────┐                   │
│  │ MAC dest dans la table ?         │                   │
│  └────┬─────────────────────┬───────┘                   │
│       │ NON                 │ OUI                       │
│       ↓                     ↓                           │
│  FLOODING               Envoi vers                      │
│  (tous les ports        le port                         │
│   sauf source)          spécifique                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Domaines de collision

### Définition

Un **domaine de collision** est une zone du réseau où des trames peuvent entrer en collision.

**Avec un hub** :
```
        Hub
         │
    ┌────┼────┐
    │    │    │
   💻    💻   💻

UN SEUL domaine de collision
Si deux machines parlent en même temps → COLLISION !
```

**Avec un switch** :
```
       Switch
         │
    ┌────┼────┐
    │    │    │
   💻    💻   💻

TROIS domaines de collision séparés
Chaque connexion est isolée → PAS de collision !
```

### Analogie : Routes vs autoroutes

**Hub (domaine de collision unique)** :
```
Route à une voie alternée
    ═══════════
    ↑    ↓
    A    B

Si A et B roulent en même temps → accident !
```

**Switch (domaines de collision séparés)** :
```
Autoroute à deux voies séparées
    ───────────  Voie A → B
    ───────────  Voie B → A

A et B peuvent circuler simultanément !
```

### Microsegmentation

Le switch crée une **microsegmentation** : chaque port est son propre domaine de collision.

**Avantages** :
- ✅ **Pas de collisions** entre les ports
- ✅ **Full-duplex** possible (envoi et réception simultanés)
- ✅ **Bande passante dédiée** par port
- ✅ **Meilleures performances** globales

**Exemple chiffré** :

```
Réseau avec hub (10 Mbit/s) :
- 10 ordinateurs
- Bande passante PARTAGÉE : 10 Mbit/s ÷ 10 = 1 Mbit/s par machine

Réseau avec switch (10 Mbit/s par port) :
- 10 ordinateurs
- Bande passante DÉDIÉE : 10 Mbit/s × 10 = 100 Mbit/s total
- Chaque machine a 10 Mbit/s garantis !
```

## Domaines de broadcast

### Définition

Un **domaine de broadcast** est une zone où les trames broadcast (FF:FF:FF:FF:FF:FF) se propagent.

**Différence avec domaine de collision** :

```
DOMAINE DE COLLISION : Où les trames PEUVENT se percuter
DOMAINE DE BROADCAST : Où les trames broadcast ARRIVENT
```

### Comportement du switch avec le broadcast

```
A envoie un broadcast (MAC dest: FF:FF:FF:FF:FF:FF)
    ↓
Le switch DOIT envoyer sur TOUS les ports
(sauf le port source)
    ↓
Tous les appareils du switch reçoivent la trame

       Switch
         │
    ┌────┼────┐
    │    │    │
   💻    💻   💻
   ↑     ↑    ↑
   Tous reçoivent le broadcast !
```

**Important** : Le switch **ne crée PAS** de domaines de broadcast séparés. Un switch = un domaine de broadcast.

**Pour séparer les domaines de broadcast**, il faut :
- Un **routeur** (couche 3)
- Des **VLANs** (Virtual LANs)

### Exemple pratique : ARP

Le protocole ARP utilise le broadcast :

```
Ordinateur A : "Qui a l'IP 192.168.1.10 ?"
    ↓
Trame broadcast (FF:FF:FF:FF:FF:FF)
    ↓
Le switch envoie à tous
    ↓
💻 B : "Pas moi (IP 192.168.1.5)"
💻 C : "C'est moi ! (IP 192.168.1.10, MAC CC:CC:CC:CC:CC:CC)"
💻 D : "Pas moi (IP 192.168.1.15)"
    ↓
C répond en unicast à A
```

## Modes de commutation

Les switches utilisent différentes **stratégies** pour transférer les trames. Chacune a ses avantages et inconvénients.

### 1. Store-and-Forward (Stocker et transférer)

**Méthode** : Le switch reçoit **toute la trame**, vérifie le FCS, puis la transmet.

```
┌────────────────────────────────────────┐
│  SWITCH EN MODE STORE-AND-FORWARD      │
├────────────────────────────────────────┤
│                                        │
│  Réception complète de la trame        │
│         ↓                              │
│  Stockage en mémoire (buffer)          │
│         ↓                              │
│  Vérification du FCS                   │
│         ↓                              │
│  ┌──────────────────┐                  │
│  │ FCS valide ?     │                  │
│  └───┬──────────┬───┘                  │
│      │ OUI      │ NON                  │
│      ↓          ↓                      │
│  Transmission  Rejet                   │
│                (trame corrompue)       │
│                                        │
└────────────────────────────────────────┘
```

**Avantages** :
- ✅ **Fiable** : Ne transmet que des trames valides
- ✅ **Détection d'erreurs** : Les trames corrompues sont éliminées
- ✅ **Adaptable** : Peut gérer différentes vitesses de ports

**Inconvénients** :
- ❌ **Latence plus élevée** : Doit attendre toute la trame

**Latence** : ~10-40 microsecondes (selon la taille de la trame)

**Analogie** : C'est comme un facteur qui ouvre le colis, vérifie que tout est intact, puis le reconditionne avant de livrer.

### 2. Cut-Through (Commutation à la volée)

**Méthode** : Le switch commence à transmettre **dès qu'il a lu l'adresse MAC de destination** (après 14 octets).

```
┌────────────────────────────────────────┐
│  SWITCH EN MODE CUT-THROUGH            │
├────────────────────────────────────────┤
│                                        │
│  Réception des premiers octets         │
│         ↓                              │
│  Lecture MAC destination (6 octets)    │
│         ↓                              │
│  Consultation table MAC                │
│         ↓                              │
│  TRANSMISSION IMMÉDIATE                │
│  (le reste de la trame suit)           │
│         ↓                              │
│  Pas de vérification FCS !             │
│                                        │
└────────────────────────────────────────┘
```

**Avantages** :
- ✅ **Latence très faible** : ~2-10 microsecondes
- ✅ **Rapidité** : Idéal pour les réseaux performants

**Inconvénients** :
- ❌ **Pas de vérification** : Transmet même les trames corrompues
- ❌ **Propagation d'erreurs** : Les erreurs se répandent dans le réseau

**Analogie** : C'est comme un facteur qui lit l'adresse et jette immédiatement le colis par-dessus la clôture sans vérifier le contenu.

### 3. Fragment-Free (Sans fragment)

**Méthode** : Compromis entre Store-and-Forward et Cut-Through. Le switch lit les **64 premiers octets** avant de transmettre.

```
┌────────────────────────────────────────┐
│  SWITCH EN MODE FRAGMENT-FREE          │
├────────────────────────────────────────┤
│                                        │
│  Réception des 64 premiers octets      │
│         ↓                              │
│  Vérification de la taille             │
│  (pas de runt frame)                   │
│         ↓                              │
│  La plupart des collisions             │
│  se produisent dans ces 64 octets      │
│         ↓                              │
│  Si OK → Transmission                  │
│                                        │
└────────────────────────────────────────┘
```

**Pourquoi 64 octets ?**
99% des erreurs (dont les collisions) se produisent dans les 64 premiers octets !

**Avantages** :
- ✅ **Latence modérée** : Plus rapide que Store-and-Forward
- ✅ **Fiabilité acceptable** : Élimine la plupart des trames problématiques

**Inconvénients** :
- ⚠️ **Compromis** : Ni le plus rapide, ni le plus fiable

**Analogie** : C'est comme un facteur qui vérifie rapidement l'état du colis (pas déchiré, bonne taille), puis le livre sans tout inspecter.

### Comparaison des modes

| Critère | Store-and-Forward | Fragment-Free | Cut-Through |
|---------|-------------------|---------------|-------------|
| **Latence** | ~10-40 μs | ~6-15 μs | ~2-10 μs |
| **Fiabilité** | ✓✓✓ Maximale | ✓✓ Bonne | ✓ Faible |
| **Vérification FCS** | Oui | Non | Non |
| **Détection collisions** | Oui | Oui | Non |
| **Usage typique** | Réseaux d'entreprise | Compromis | Gaming, trading |
| **Propagation erreurs** | Non | Limitée | Oui |

### Adaptive Cut-Through (moderne)

Beaucoup de switches modernes sont **adaptatifs** :

```
Démarrage : Mode Cut-Through (rapide)
    ↓
Détection d'erreurs fréquentes ?
    ↓
Passage automatique en Store-and-Forward
    ↓
Erreurs réduites ?
    ↓
Retour en Cut-Through
```

**Avantage** : Le meilleur des deux mondes !

## Aging et maintenance de la table

### Durée de vie des entrées

Les entrées dans la table MAC ont une **durée de vie limitée** (généralement **300 secondes / 5 minutes**).

**Pourquoi ?**

```
Scénario 1 : Appareil déconnecté
├─→ Sans aging : entrée reste à jamais
└─→ Avec aging : entrée supprimée après 5 min

Scénario 2 : Appareil déplacé de port
├─→ Sans aging : confusion (vieille entrée incorrecte)
└─→ Avec aging : nouvelle entrée remplace l'ancienne

Scénario 3 : Adresse MAC modifiée
├─→ Sans aging : double entrée (conflit)
└─→ Avec aging : ancienne entrée expire
```

### Rafraîchissement

Chaque fois qu'une MAC **envoie** une trame, son timer est **remis à zéro**.

```
Ordinateur A envoie régulièrement des données
    ↓
Compteur d'âge remis à 0 à chaque trame
    ↓
L'entrée reste toujours dans la table
    ↓
A ne parle plus pendant 6 minutes
    ↓
L'entrée expire et est supprimée
```

### Exemple de vieillissement

```
Table MAC du switch :

Temps T=0 (toutes les entrées fraîches)
┌──────────────────────┬──────┬──────────┐
│ MAC                  │ Port │ Âge      │
├──────────────────────┼──────┼──────────┤
│ AA:AA:AA:AA:AA:AA    │  1   │ 0 sec    │
│ BB:BB:BB:BB:BB:BB    │  2   │ 0 sec    │
│ CC:CC:CC:CC:CC:CC    │  3   │ 0 sec    │
└──────────────────────┴──────┴──────────┘

Temps T=100s (A et B ont parlé récemment, pas C)
┌──────────────────────┬──────┬──────────┐
│ MAC                  │ Port │ Âge      │
├──────────────────────┼──────┼──────────┤
│ AA:AA:AA:AA:AA:AA    │  1   │ 5 sec    │ ← A vient de parler
│ BB:BB:BB:BB:BB:BB    │  2   │ 20 sec   │ ← B a parlé il y a 20s
│ CC:CC:CC:CC:CC:CC    │  3   │ 100 sec  │ ← C n'a rien dit
└──────────────────────┴──────┴──────────┘

Temps T=310s (C n'a toujours rien dit)
┌──────────────────────┬──────┬──────────┐
│ MAC                  │ Port │ Âge      │
├──────────────────────┼──────┼──────────┤
│ AA:AA:AA:AA:AA:AA    │  1   │ 45 sec   │
│ BB:BB:BB:BB:BB:BB    │  2   │ 80 sec   │
│ [SUPPRIMÉE]          │      │          │ ← C a expiré !
└──────────────────────┴──────┴──────────┘
```

## Cas particuliers

### 1. Trames inconnues (Unknown Unicast)

Que fait le switch si la MAC destination **n'est pas dans la table** ?

```
A envoie vers une MAC inconnue : XX:XX:XX:XX:XX:XX
    ↓
Switch consulte sa table
    ↓
"Je ne connais pas XX:XX:XX:XX:XX:XX !"
    ↓
FLOODING : Envoi sur TOUS les ports (sauf source)
    ↓
Si XX existe, il répondra
    ↓
Le switch apprendra où il est
```

**C'est similaire au broadcast**, mais pour des unicasts inconnus.

### 2. Table pleine

La table MAC a une taille limitée (ex: 8000 entrées pour un petit switch, 128000+ pour un gros switch).

**Que se passe-t-il si la table est pleine ?**

```
Nouvelle MAC détectée, mais table pleine
    ↓
Option A : Supprimer l'entrée la plus ancienne
Option B : Flooding de toutes les trames vers cette MAC
Option C : Refus d'apprentissage (rare)
```

**Attaque par saturation** : Un attaquant peut générer des milliers de MACs aléatoires pour remplir la table, forçant le switch à agir comme un hub !

**Protection** : Port Security (limite le nombre de MACs par port).

### 3. Adresses MAC multiples sur un port

Cas typique : Un autre switch ou un hub connecté sur un port.

```
Switch A, Port 1
    ↓
Connecté à Switch B
    ↓
Switch B a 3 ordinateurs :
    ├─→ 💻 MAC: AA:AA:AA:AA:AA:AA
    ├─→ 💻 MAC: BB:BB:BB:BB:BB:BB
    └─→ 💻 MAC: CC:CC:CC:CC:CC:CC

Table de Switch A :
┌──────────────────────┬──────┐
│ AA:AA:AA:AA:AA:AA    │  1   │ ← Toutes sur port 1 !
│ BB:BB:BB:BB:BB:BB    │  1   │
│ CC:CC:CC:CC:CC:CC    │  1   │
└──────────────────────┴──────┘
```

C'est **normal et fonctionnel**. Le Switch A sait juste que toutes ces MACs sont "de l'autre côté" du port 1.

## Avantages des switches sur les hubs

### Résumé comparatif complet

| Caractéristique | Hub | Switch |
|-----------------|-----|--------|
| **Intelligence** | Aucune | Apprentissage des MACs |
| **Domaines de collision** | 1 (partagé) | 1 par port |
| **Bande passante** | Partagée | Dédiée par port |
| **Collisions** | Fréquentes | Éliminées (full-duplex) |
| **Sécurité** | Faible (tout visible) | Meilleure (isolation) |
| **Performance** | Dégradée avec + d'appareils | Stable |
| **Full-duplex** | Non | Oui |
| **Latence** | Très faible | Faible à modérée |
| **Prix** | Très bon marché | Abordable |
| **Usage moderne** | Obsolète | Standard |

### Exemple de gain de performance

```
Scénario : 10 ordinateurs, 100 Mbit/s

Avec Hub :
├─→ Bande passante totale : 100 Mbit/s PARTAGÉS
├─→ Par ordinateur : ~10 Mbit/s (en moyenne)
├─→ Collisions fréquentes → perte de performance
└─→ Performance réelle : ~5-7 Mbit/s par machine

Avec Switch :
├─→ Bande passante par port : 100 Mbit/s DÉDIÉS
├─→ Par ordinateur : 100 Mbit/s garantis
├─→ Pas de collisions
└─→ Performance réelle : 95+ Mbit/s par machine

Gain : 13x à 19x de performance !
```

## Spanning Tree Protocol (STP) - Aperçu

### Le problème des boucles

Que se passe-t-il si on connecte des switches en **boucle** ?

```
    Switch A
    ╱      ╲
Switch B ──── Switch C
```

**Scénario catastrophique** :

```
1. A envoie un broadcast
2. Il va vers B et C
3. B le renvoie à C, C à A, A à B...
4. ∞ BOUCLE INFINIE !
5. Le réseau sature en quelques secondes (broadcast storm)
```

### La solution : STP

Le **Spanning Tree Protocol** (IEEE 802.1D) :
- Détecte automatiquement les boucles
- **Bloque** certains ports pour éliminer les boucles
- Crée une topologie en **arbre** (sans boucle)
- Maintient la redondance (si un lien tombe, les ports bloqués s'activent)

```
    Switch A (Root)
    ╱      ╲
Switch B ──🚫── Switch C
         (port bloqué)

Une seule voie active entre tous les switches
Pas de boucle, pas de broadcast storm !
```

**Nous approfondirons STP dans des sections avancées**, mais sachez qu'il est **actif par défaut** sur tous les switches modernes et vous protège automatiquement.

## Switch manageable vs non-manageable

### Switch non-manageable (unmanaged)

**Caractéristiques** :
- Plug-and-play (branchez, ça marche)
- Pas de configuration
- Pas d'interface d'administration
- Fonctions de base uniquement

**Usage** : Réseau domestique, petit bureau

**Prix** : 20-50€ pour 5-8 ports

### Switch manageable (managed)

**Caractéristiques** :
- Interface de configuration (web, CLI)
- VLANs
- QoS (Quality of Service)
- Port mirroring (pour Wireshark)
- Agrégation de liens
- Statistiques détaillées
- Sécurité avancée (802.1X, Port Security)

**Usage** : Entreprise, data center

**Prix** : 100€ à plusieurs milliers d'euros

### Schéma des fonctionnalités

```
┌─────────────────────────────────────────────────┐
│         SWITCH NON-MANAGEABLE                   │
├─────────────────────────────────────────────────┤
│  • Commutation de base                          │
│  • Apprentissage MAC                            │
│  • Full-duplex auto                             │
│  • STP de base                                  │
│  • Aucune configuration                         │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│         SWITCH MANAGEABLE                       │
├─────────────────────────────────────────────────┤
│  • Tout ce qui précède +                        │
│  • VLANs (segmentation logique)                 │
│  • QoS (priorité du trafic)                     │
│  • SNMP (monitoring)                            │
│  • Port mirroring (analyse)                     │
│  • Agrégation de liens (LAG)                    │
│  • ACLs (filtrage)                              │
│  • 802.1X (authentification)                    │
│  • Port Security (limitation MACs)              │
│  • SSH/HTTPS pour configuration                 │
│  • Firmware upgradable                          │
└─────────────────────────────────────────────────┘
```

## Points clés à retenir

1. **Switch vs Hub** : Le switch est intelligent (apprend les MACs), le hub répète bêtement.

2. **Table d'adresses MAC** : Le switch associe chaque MAC à un port et apprend automatiquement.

3. **Microsegmentation** : Chaque port = un domaine de collision séparé.

4. **Domaine de collision** : Zone où des collisions peuvent se produire (éliminées par les switches).

5. **Domaine de broadcast** : Zone où les broadcasts se propagent (un switch = un domaine de broadcast).

6. **Modes de commutation** :
   - **Store-and-Forward** : Le plus fiable (vérifie FCS)
   - **Cut-Through** : Le plus rapide (latence minimale)
   - **Fragment-Free** : Compromis (64 premiers octets)

7. **Aging** : Les entrées MAC expirent après 300 secondes d'inactivité.

8. **Flooding** : Envoi sur tous les ports pour MACs inconnues et broadcasts.

9. **Full-duplex** : Émission et réception simultanées, élimine les collisions.

10. **STP** : Évite les boucles réseau en bloquant certains ports.

---

## Prochaine étape

Maintenant que vous comprenez comment les switches acheminent intelligemment les trames avec les adresses MAC, nous allons découvrir **ARP (Address Resolution Protocol)**, le protocole magique qui fait le lien entre les adresses IP et les adresses MAC !

---


⏭️ [ARP (Address Resolution Protocol)](/02-couche-acces-reseau/06-arp.md)
