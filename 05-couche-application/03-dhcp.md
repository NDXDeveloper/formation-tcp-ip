🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.3 DHCP (Dynamic Host Configuration Protocol)

## Introduction

Imaginez que vous arrivez dans un hôtel. À la réception, on vous donne :
- Une clé de chambre (avec un numéro)
- Le code WiFi
- L'adresse du restaurant
- Le numéro à composer pour la réception

Sans cette étape, vous seriez perdu dans l'hôtel sans savoir quelle chambre utiliser ni comment accéder aux services.

**DHCP fait exactement la même chose pour votre ordinateur sur un réseau** : quand vous vous connectez, il vous fournit automatiquement toutes les informations nécessaires pour communiquer sur le réseau.

DHCP (Dynamic Host Configuration Protocol) est un protocole qui **automatise la configuration réseau** des appareils. C'est l'un des protocoles les plus utilisés au quotidien, même si vous n'en avez probablement jamais entendu parler : chaque fois que vous vous connectez à un WiFi, DHCP travaille en arrière-plan.

## Le problème résolu par DHCP

### Avant DHCP : Configuration manuelle

Dans les années 1980-1990, chaque ordinateur devait être configuré **manuellement** :

```
┌────────────────────────────────────────────────────────┐
│ Configuration manuelle d'un PC (années 1990)           │
├────────────────────────────────────────────────────────┤
│                                                        │
│ Administrateur doit configurer :                       │
│                                                        │
│ ✎ Adresse IP : 192.168.1.42                           │
│ ✎ Masque de sous-réseau : 255.255.255.0               │
│ ✎ Passerelle par défaut : 192.168.1.1                 │
│ ✎ Serveur DNS primaire : 192.168.1.10                 │
│ ✎ Serveur DNS secondaire : 8.8.8.8                    │
│                                                        │
│ Pour CHAQUE ordinateur du réseau !                     │
└────────────────────────────────────────────────────────┘
```

### Problèmes de la configuration manuelle

**1. Laborieux et chronophage**
```
Réseau de 100 ordinateurs :
├─ 10 minutes par machine
├─ 100 machines × 10 minutes = 1000 minutes
└─ = 16,7 heures de travail !
```

**2. Erreurs humaines**
```
Erreurs courantes :
├─ Typo dans l'adresse IP : 192.168.1.24 au lieu de 192.168.1.42
├─ Mauvaise passerelle : 192.168.1.2 au lieu de 192.168.1.1
├─ Duplication d'IP : Deux machines avec 192.168.1.42
└─ Oubli de paramètres : Pas de DNS configuré

Résultat : Machine non fonctionnelle ou conflits réseau
```

**3. Gestion des changements**
```
Scénario : Changement de serveur DNS
├─ Ancien DNS : 192.168.1.10
├─ Nouveau DNS : 192.168.1.20
└─ Action : Reconfigurer manuellement 100 machines !
```

**4. Gestion des adresses IP**
```
Problème de suivi :
├─ Qui utilise quelle adresse IP ?
├─ Quelles adresses sont libres ?
├─ Feuille Excel ? Registre papier ?
└─ Conflits d'IP inévitables
```

**5. Mobilité impossible**
```
Ordinateur portable :
├─ Au bureau : 192.168.1.42
├─ À la maison : 192.168.0.15
├─ Au café : 10.0.0.123
└─ Reconfiguration manuelle à chaque fois !
```

### La solution : DHCP

DHCP résout tous ces problèmes en **automatisant** la configuration :

```
┌────────────────────────────────────────────────────────┐
│ Avec DHCP (depuis 1993)                                │
├────────────────────────────────────────────────────────┤
│                                                        │
│ 1. Ordinateur se connecte au réseau                    │
│ 2. Envoie une requête DHCP : "J'ai besoin de config"   │
│ 3. Serveur DHCP répond automatiquement :               │
│    • Adresse IP : 192.168.1.42                         │
│    • Masque : 255.255.255.0                            │
│    • Passerelle : 192.168.1.1                          │
│    • DNS : 192.168.1.10, 8.8.8.8                       │
│ 4. Ordinateur configuré et opérationnel                │
│                                                        │
│ Temps total : 1-2 secondes                             │
│ Intervention humaine : 0                               │
└────────────────────────────────────────────────────────┘
```

## Qu'est-ce que DHCP ?

### Définition

**DHCP (Dynamic Host Configuration Protocol)** est un protocole réseau qui permet à un serveur de **distribuer automatiquement** des configurations réseau aux clients qui en font la demande.

```
┌─────────────────────────────────────────────────────────┐
│                    Rôle de DHCP                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Serveur DHCP = "Distributeur automatique de config"    │
│                                                         │
│  Client se connecte → Reçoit automatiquement :          │
│  ├─ Adresse IP (temporaire)                             │
│  ├─ Masque de sous-réseau                               │
│  ├─ Passerelle par défaut (routeur)                     │
│  ├─ Serveurs DNS                                        │
│  └─ Autres paramètres optionnels                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Caractéristiques principales

| Caractéristique | Description |
|-----------------|-------------|
| **Automatique** | Aucune intervention manuelle nécessaire |
| **Dynamique** | Les IPs sont attribuées temporairement (bail) |
| **Centralisé** | Une seule configuration pour tout le réseau |
| **Flexible** | S'adapte aux arrivées/départs de machines |
| **Réutilisable** | Les IPs libérées sont réattribuées |

### Historique

```
1984 : RARP (Reverse ARP)
       └─ Premier système d'auto-configuration (très limité)

1985 : BOOTP (Bootstrap Protocol)
       └─ Configuration automatique pour diskless workstations
       └─ IPs statiques (pas dynamique)

1993 : DHCP (RFC 1531, puis RFC 2131 en 1997)
       └─ Basé sur BOOTP mais avec gestion dynamique
       └─ Standard actuel

1999 : DHCPv6 (RFC 3315)
       └─ Version pour IPv6
```

## Fonctionnement de base

### Architecture client-serveur

DHCP suit le modèle **client-serveur** classique :

```
┌──────────────────────────────────────────────────────────┐
│                  Architecture DHCP                       │
└──────────────────────────────────────────────────────────┘

          Réseau local (192.168.1.0/24)

┌─────────────┐         ┌──────────────┐         ┌─────────────┐
│   Client    │         │   Serveur    │         │   Client    │
│   DHCP      │         │    DHCP      │         │   DHCP      │
│             │         │              │         │             │
│  (Demande   │◄───────►│  (Distribue  │◄───────►│  (Demande   │
│   config)   │         │    config)   │         │   config)   │
│             │         │              │         │             │
│ Pas d'IP    │         │ Pool d'IPs : │         │ Pas d'IP    │
│  encore     │         │ .50 - .200   │         │  encore     │
└─────────────┘         └──────────────┘         └─────────────┘
      │                        │                        │
      └────────────────────────┴────────────────────────┘
                    Switch / Point d'accès
```

### Composants essentiels

**1. Client DHCP**
```
Rôle : Demandeur de configuration
Localisation : Sur chaque appareil (PC, smartphone, imprimante...)
Fonctionnement :
├─ Détecte connexion réseau
├─ Envoie requête de configuration
├─ Reçoit et applique les paramètres
└─ Renouvelle périodiquement le bail

Exemples :
• Windows : Service "Client DHCP"
• Linux : dhclient, dhcpcd, NetworkManager
• Intégré dans tous les OS modernes
```

**2. Serveur DHCP**
```
Rôle : Distributeur de configurations
Localisation :
├─ Box Internet (Freebox, Livebox...)
├─ Routeur d'entreprise
├─ Serveur dédié (Windows Server, Linux avec isc-dhcp-server)
└─ Point d'accès WiFi

Responsabilités :
├─ Maintenir un pool d'adresses IP disponibles
├─ Attribuer des IPs aux clients
├─ Suivre les attributions (qui a quelle IP)
├─ Gérer les baux (durée d'attribution)
└─ Récupérer les IPs non utilisées
```

**3. Pool d'adresses (DHCP scope)**
```
Définition : Plage d'adresses IP que le serveur peut distribuer

Exemple de configuration :
┌─────────────────────────────────────────────────────┐
│ Réseau : 192.168.1.0/24                             │
│                                                     │
│ Adresses réservées (configuration statique) :       │
│ • 192.168.1.1       : Routeur                       │
│ • 192.168.1.10      : Serveur DNS                   │
│ • 192.168.1.20      : Imprimante réseau             │
│ • 192.168.1.30-49   : Serveurs                      │
│                                                     │
│ Pool DHCP (distribution dynamique) :                │
│ • 192.168.1.50-200  : Clients DHCP                  │
│   └─ 151 adresses disponibles                       │
│                                                     │
│ Non utilisées :                                     │
│ • 192.168.1.201-254 : Réservées pour expansion      │
└─────────────────────────────────────────────────────┘
```

### Exemple simple de fonctionnement

```
┌──────────────────────────────────────────────────────────┐
│ Scénario : Connexion d'un laptop au WiFi                 │
└──────────────────────────────────────────────────────────┘

État initial :
Laptop : Pas d'adresse IP
Serveur DHCP : Pool disponible 192.168.1.50-200

Étape 1 : Connexion WiFi
├─ Laptop se connecte au point d'accès
└─ Carte réseau active, mais pas d'IP

Étape 2 : Demande DHCP (processus DORA, détaillé plus tard)
├─ Laptop diffuse : "J'ai besoin d'une configuration !"
├─ Serveur DHCP répond : "Voici ta config :"
│   • IP : 192.168.1.50
│   • Masque : 255.255.255.0
│   • Passerelle : 192.168.1.1
│   • DNS : 192.168.1.10, 8.8.8.8
│   • Bail : 24 heures
└─ Laptop accepte et configure

Étape 3 : Configuration appliquée
├─ Laptop maintenant : 192.168.1.50/24
├─ Route par défaut : 192.168.1.1
├─ DNS configurés
└─ Prêt à communiquer !

Durée totale : 1-2 secondes
```

## Informations fournies par DHCP

### Paramètres obligatoires

DHCP fournit toujours ces paramètres essentiels :

**1. Adresse IP**
```
Exemple : 192.168.1.50
Rôle : Identifiant unique du client sur le réseau
Durée : Temporaire (durée du bail)
```

**2. Masque de sous-réseau**
```
Exemple : 255.255.255.0 (ou /24)
Rôle : Définit la taille du réseau local
Permet au client de savoir si une IP est locale ou distante
```

**3. Passerelle par défaut (Default Gateway)**
```
Exemple : 192.168.1.1
Rôle : Adresse du routeur pour accéder à Internet/autres réseaux
Toutes les requêtes hors réseau local passent par cette IP
```

### Paramètres courants optionnels

**4. Serveurs DNS**
```
Exemple :
├─ DNS primaire : 192.168.1.10
└─ DNS secondaire : 8.8.8.8

Rôle : Résolution de noms de domaine
Sans DNS : www.google.com ne fonctionne pas
Avec DNS : www.google.com → 142.250.185.196
```

**5. Nom de domaine**
```
Exemple : entreprise.local
Rôle : Domaine de recherche par défaut
Permet d'utiliser des noms courts : "serveur1" → "serveur1.entreprise.local"
```

**6. Serveurs NTP (Network Time Protocol)**
```
Exemple : 192.168.1.5, pool.ntp.org
Rôle : Synchronisation de l'horloge
Crucial pour logs, certificats SSL, etc.
```

**7. Serveur WINS (Windows Internet Name Service)**
```
Exemple : 192.168.1.15
Rôle : Résolution de noms NetBIOS (ancien, surtout Windows)
Moins utilisé aujourd'hui (remplacé par DNS)
```

### Paramètres avancés

DHCP peut fournir plus de 200 options différentes :

```
Option 1   : Masque de sous-réseau
Option 3   : Passerelle par défaut
Option 6   : Serveurs DNS
Option 12  : Nom d'hôte
Option 15  : Nom de domaine DNS
Option 42  : Serveurs NTP
Option 51  : Durée du bail
Option 66  : Serveur TFTP (pour boot réseau)
Option 67  : Nom du fichier de boot
Option 121 : Routes statiques
...
```

**Exemple de configuration reçue complète :**
```
┌─────────────────────────────────────────────────────┐
│ Configuration DHCP reçue par un client              │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Adresse IP assignée : 192.168.1.50                  │
│ Masque de sous-réseau : 255.255.255.0               │
│ Durée du bail : 86400 secondes (24 heures)          │
│                                                     │
│ Passerelle par défaut : 192.168.1.1                 │
│                                                     │
│ Serveurs DNS :                                      │
│  • Primaire : 192.168.1.10                          │
│  • Secondaire : 8.8.8.8                             │
│                                                     │
│ Domaine DNS : entreprise.local                      │
│ Nom d'hôte : laptop-bob                             │
│                                                     │
│ Serveurs NTP :                                      │
│  • 192.168.1.5                                      │
│  • pool.ntp.org                                     │
│                                                     │
│ Serveur TFTP : 192.168.1.20 (pour PXE boot)         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## Le concept de bail (Lease)

### Qu'est-ce qu'un bail DHCP ?

Un **bail** (lease) est une **attribution temporaire** d'une adresse IP à un client.

```
┌─────────────────────────────────────────────────────┐
│ Analogie : Location d'appartement                   │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Bail immobilier :                                   │
│ • Locataire loue appartement pour 1 an              │
│ • À la fin, peut renouveler ou partir               │
│ • Si non renouvelé, appartement disponible          │
│                                                     │
│ Bail DHCP :                                         │
│ • Client "loue" une IP pour X heures                │
│ • À la fin, peut renouveler ou libérer              │
│ • Si non renouvelé, IP retourne dans le pool        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Durée typique du bail

```
Type de réseau                  Durée typique    Raison
──────────────────────────────────────────────────────────
Réseau domestique               24 heures        Équilibre
Box Internet                    24-72 heures     Stabilité

Réseau d'entreprise (câblé)     7 jours          Postes fixes
Réseau d'entreprise (WiFi)      8-24 heures      Plus de mobilité

Café, hôtel, aéroport          1-2 heures       Forte rotation
Hotspot public                  30-60 minutes    Très forte rotation

Réseau de développement         Infini (ou long) Stabilité tests
Laboratoire                     7+ jours         IPs stables
```

### Cycle de vie d'un bail

```
┌──────────────────────────────────────────────────────┐
│ Timeline d'un bail de 24 heures                      │
└──────────────────────────────────────────────────────┘

T = 0h (Attribution)
├─ Client reçoit IP : 192.168.1.50
├─ Bail : 24 heures (86400 secondes)
└─ Échéance : T + 24h

T = 12h (50% du bail écoulé - T1)
├─ Client tente de renouveler automatiquement
├─ Envoie requête au serveur DHCP
└─ Si succès : bail renouvelé pour 24h supplémentaires
└─ Si échec : continue d'utiliser l'IP, réessaiera

T = 21h (87.5% du bail écoulé - T2)
├─ Si renouvellement T1 a échoué
├─ Client tente à nouveau (en broadcast)
└─ Peut accepter offre d'un autre serveur DHCP

T = 24h (Expiration)
├─ Si aucun renouvellement réussi :
│   └─ Client DOIT arrêter d'utiliser l'IP
│   └─ Recommence le processus depuis le début
└─ Si renouvelé : cycle continue normalement

T = 36h (après renouvellement à T=12h)
├─ Nouveau point 50% : tente renouvellement
└─ Et ainsi de suite...
```

### Pourquoi des baux temporaires ?

**Avantages des baux temporaires :**

```
1. Gestion efficace des adresses IP
   ├─ IPs automatiquement récupérées si machine déconnectée
   ├─ Pool d'IPs plus petit que nombre d'appareils possible
   └─ Exemple : 100 IPs pour 300 utilisateurs (mais 80 connectés max)

2. Adaptabilité
   ├─ Machines mobiles changent de réseau
   ├─ IP libérée automatiquement après départ
   └─ Disponible pour nouvel arrivant

3. Flexibilité de configuration
   ├─ Changement de DNS propagé au renouvellement
   ├─ Modification de configuration réseau simplifiée
   └─ Pas besoin de reconfigurer manuellement

4. Détection d'appareils inactifs
   ├─ Bail non renouvelé = appareil éteint/parti
   └─ Nettoyage automatique des tables
```

**Inconvénients :**

```
1. Overhead réseau
   ├─ Trafic de renouvellement périodique
   └─ Généralement négligeable

2. IP peut changer
   ├─ Problème pour serveurs
   └─ Solution : réservation DHCP ou IP statique

3. Dépendance au serveur DHCP
   ├─ Si serveur tombe avant renouvellement
   └─ Clients perdent connectivité à l'expiration
```

## Types de distribution d'adresses

DHCP supporte plusieurs modes d'attribution :

### 1. Attribution dynamique (Dynamic Allocation)

Le mode **standard** et le plus courant :

```
Principe :
├─ IP attribuée temporairement depuis un pool
├─ Durée = bail (lease time)
├─ IP peut changer entre deux connexions
└─ IP retourne au pool après expiration

Exemple :
Laptop se connecte lundi : reçoit 192.168.1.50
Laptop se déconnecte lundi soir
Laptop se reconnecte mardi : peut recevoir 192.168.1.75
```

**Avantages :**
```
✓ Utilisation optimale des adresses disponibles
✓ Gestion automatique complète
✓ Aucune configuration par machine
```

**Inconvénients :**
```
✗ IP peut changer
✗ Pas adapté pour serveurs/imprimantes
```

### 2. Attribution automatique (Automatic Allocation)

Mode **permanent** mais automatisé :

```
Principe :
├─ IP attribuée automatiquement depuis un pool
├─ Mais attribution PERMANENTE
├─ Même machine = toujours même IP
└─ IP libérée seulement si explicitement demandé

Exemple :
Laptop se connecte lundi : reçoit 192.168.1.50
Laptop se déconnecte et reconnecte : toujours 192.168.1.50
```

**Avantages :**
```
✓ IP stable sans configuration manuelle
✓ Facilite logs et monitoring
```

**Inconvénients :**
```
✗ IPs jamais récupérées automatiquement
✗ Pool peut s'épuiser
✗ Peu utilisé en pratique
```

### 3. Attribution manuelle / Réservation DHCP (Manual Allocation)

Mode **réservé** pour machines spécifiques :

```
Principe :
├─ Administrateur configure mapping MAC ↔ IP
├─ Machine avec cette MAC reçoit toujours cette IP
├─ Reste géré par DHCP (renouvellement, etc.)
└─ Combine avantages DHCP + IP fixe

Configuration serveur DHCP :
┌─────────────────────────────────────────────────┐
│ Réservations :                                  │
│ • MAC: 00:1A:2B:3C:4D:5E → IP: 192.168.1.20     │
│   (Imprimante bureau)                           │
│                                                 │
│ • MAC: 00:11:22:33:44:55 → IP: 192.168.1.30     │
│   (Serveur NAS)                                 │
│                                                 │
│ • MAC: AA:BB:CC:DD:EE:FF → IP: 192.168.1.40     │
│   (Caméra de sécurité)                          │
└─────────────────────────────────────────────────┘
```

**Avantages :**
```
✓ IP fixe et prévisible
✓ Reste géré par DHCP (DNS, passerelle auto)
✓ Centralisé (pas de config sur la machine)
✓ Facilite gestion pare-feu, règles réseau
```

**Inconvénients :**
```
✗ Configuration manuelle nécessaire
✗ Doit connaître l'adresse MAC
```

**Cas d'usage typiques :**
```
Réservations DHCP recommandées pour :
• Imprimantes réseau
• Serveurs (fichiers, print, etc.)
• Points d'accès WiFi
• Caméras IP
• Équipements réseau (switches managés)
• Machines nécessitant un accès depuis l'extérieur
```

## Transport et protocoles

### Ports utilisés

DHCP utilise le protocole **UDP** :

```
┌─────────────────────────────────────────────────┐
│ Ports DHCP                                      │
├─────────────────────────────────────────────────┤
│                                                 │
│ Serveur DHCP écoute sur : Port UDP 67           │
│ Client DHCP écoute sur :  Port UDP 68           │
│                                                 │
└─────────────────────────────────────────────────┘

Communication :
Client (port 68) ←→ Serveur (port 67)
```

### Pourquoi UDP et pas TCP ?

```
Raisons du choix UDP :

1. Simplicité
   └─ Pas besoin de connexion établie

2. Broadcast initial
   └─ Client n'a pas encore d'IP
   └─ Ne peut pas établir connexion TCP

3. Léger
   └─ Échange rapide (4 messages seulement)
   └─ Overhead TCP inutile

4. Sans état
   └─ Serveur ne maintient pas de connexion
   └─ Plus scalable
```

### Utilisation du broadcast

Au démarrage, le client **n'a pas d'adresse IP**, il ne peut donc pas envoyer de requête à une IP spécifique.

```
Problème :
┌─────────────────────────────────────────────────┐
│ Client vient de s'allumer                       │
│ • Pas d'adresse IP : ???                        │
│ • Ne sait pas où est le serveur DHCP            │
│ • Comment contacter le serveur ?                │
└─────────────────────────────────────────────────┘

Solution : Broadcast
┌─────────────────────────────────────────────────┐
│ Client envoie en BROADCAST                      │
│ • Source : 0.0.0.0 (pas d'IP encore)            │
│ • Destination : 255.255.255.255 (broadcast)     │
│ • Port source : 68                              │
│ • Port destination : 67                         │
│                                                 │
│ → Tous les appareils du réseau reçoivent        │
│ → Serveur DHCP répond                           │
└─────────────────────────────────────────────────┘
```

**Exemple de requête DHCP initiale :**
```
Couche Ethernet :
├─ MAC source : 00:1A:2B:3C:4D:5E (client)
└─ MAC destination : FF:FF:FF:FF:FF:FF (broadcast)

Couche IP :
├─ IP source : 0.0.0.0 (pas encore d'IP)
└─ IP destination : 255.255.255.255 (broadcast)

Couche UDP :
├─ Port source : 68
└─ Port destination : 67

Données : Message DHCP DISCOVER
```

## Où trouve-t-on un serveur DHCP ?

### Réseaux domestiques

```
┌─────────────────────────────────────────────────┐
│ Box Internet (Freebox, Livebox, etc.)           │
├─────────────────────────────────────────────────┤
│                                                 │
│ Serveur DHCP intégré :                          │
│ • Pool : 192.168.1.50 - 192.168.1.150           │
│ • Durée bail : 24 heures                        │
│ • DNS : Ceux du FAI                             │
│ • Passerelle : 192.168.1.1 (la box elle-même)   │
│                                                 │
└─────────────────────────────────────────────────┘

Tous les appareils domestiques :
├─ Smartphone
├─ Laptop
├─ Smart TV
├─ Console de jeux
├─ Objets connectés
└─ Tous reçoivent config automatique de la box
```

### Réseaux d'entreprise

```
┌─────────────────────────────────────────────────┐
│ Option 1 : Serveur dédié                        │
├─────────────────────────────────────────────────┤
│ • Windows Server avec rôle DHCP                 │
│ • Linux avec isc-dhcp-server ou dnsmasq         │
│ • Appliance réseau dédiée                       │
│                                                 │
│ Avantages :                                     │
│ ✓ Configuration fine                            │
│ ✓ Haute disponibilité (failover)                │
│ ✓ Logs détaillés                                │
│ ✓ Intégration Active Directory                  │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ Option 2 : Routeur/Switch managé                │
├─────────────────────────────────────────────────┤
│ • Fonction DHCP intégrée                        │
│ • Suffit pour PME                               │
│                                                 │
│ Avantages :                                     │
│ ✓ Pas de serveur supplémentaire                 │
│ ✓ Configuration simple                          │
└─────────────────────────────────────────────────┘
```

### Réseaux segmentés et DHCP Relay

Dans les grandes organisations, les réseaux sont segmentés en VLANs :

```
Problème :
┌─────────────────────────────────────────────────┐
│ Broadcast ne traverse PAS les routeurs          │
│                                                 │
│ VLAN 10 (192.168.10.0/24)                       │
│ └─ Client DHCP broadcast → Ne sort pas du VLAN  │
│                                                 │
│ VLAN 20 (192.168.20.0/24)                       │
│ └─ Serveur DHCP ici → N'entend pas le client    │
└─────────────────────────────────────────────────┘

Solution : DHCP Relay Agent (IP Helper)
┌─────────────────────────────────────────────────┐
│                                                 │
│ VLAN 10 : Client broadcast DHCP                 │
│     ↓                                           │
│ Routeur : Intercepte le broadcast               │
│     │     Relay agent configuré                 │
│     ↓                                           │
│ VLAN 20 : Transfère en unicast au serveur DHCP  │
│     ↓                                           │
│ Serveur DHCP : Répond                           │
│     ↓                                           │
│ Routeur : Transfère la réponse                  │
│     ↓                                           │
│ VLAN 10 : Client reçoit la config               │
│                                                 │
└─────────────────────────────────────────────────┘

Configuration routeur :
interface vlan10
  ip helper-address 192.168.20.10  (serveur DHCP)
```

## Avantages et limitations de DHCP

### Avantages

```
✅ Automatisation complète
   └─ Zéro configuration manuelle nécessaire
   └─ Gain de temps considérable

✅ Élimination des erreurs
   └─ Pas de typos, conflits d'IP évités
   └─ Configuration cohérente

✅ Gestion centralisée
   └─ Un seul endroit pour changer la config
   └─ Changement de DNS propagé à tous

✅ Mobilité
   └─ Appareils fonctionnent sur différents réseaux
   └─ Connexion automatique

✅ Utilisation efficace des IPs
   └─ Pool partagé dynamiquement
   └─ Moins d'IPs nécessaires que d'appareils

✅ Facilite scaling
   └─ Nouveau switch ? Ajout de pool
   └─ Expansion du réseau simplifiée

✅ Flexibilité
   └─ Baux courts ou longs selon besoin
   └─ Réservations pour machines spécifiques
```

### Limitations et points d'attention

```
⚠️ Point unique de défaillance
   └─ Si serveur DHCP tombe, nouveaux clients bloqués
   └─ Solution : Haute disponibilité (2+ serveurs)

⚠️ Sécurité
   └─ N'importe qui peut demander une IP
   └─ Rogue DHCP servers (serveurs pirates)
   └─ Solution : DHCP snooping, 802.1X

⚠️ Dépendance réseau
   └─ Broadcast ne traverse pas routeurs (sans relay)
   └─ Nécessite relay agents pour réseaux segmentés

⚠️ Inadapté pour certains usages
   └─ Serveurs publics : besoin IP fixe
   └─ Équipements critiques : préférer IP statique
   └─ DMZ : souvent configuration manuelle

⚠️ Latence au démarrage
   └─ 1-2 secondes pour obtenir config
   └─ Négligeable mais existe

⚠️ Complexité du dépannage
   └─ Problème réseau : DHCP ? DNS ? Passerelle ?
   └─ Nécessite outils de diagnostic
```

## Cas d'usage typiques

### Réseau domestique

```
Configuration typique box Internet :
┌─────────────────────────────────────────────────┐
│ Réseau : 192.168.1.0/24                         │
│ Serveur DHCP : 192.168.1.1 (la box)             │
│                                                 │
│ Pool DHCP : 192.168.1.10 - 192.168.1.100        │
│ Durée bail : 24 heures                          │
│ DNS : 212.27.40.240 (FAI) + 8.8.8.8 (Google)    │
│ Passerelle : 192.168.1.1                        │
│                                                 │
│ Appareils typiques :                            │
│ • Laptop : 192.168.1.10 (dynamique)             │
│ • Smartphone : 192.168.1.11 (dynamique)         │
│ • Smart TV : 192.168.1.12 (dynamique)           │
│ • Imprimante : 192.168.1.50 (réservation)       │
└─────────────────────────────────────────────────┘
```

### Réseau d'entreprise

```
Configuration typique PME (100 employés) :
┌─────────────────────────────────────────────────┐
│ Réseau : 10.0.0.0/16                            │
│ Serveur DHCP : 10.0.0.10 (serveur dédié)        │
│                                                 │
│ VLAN 10 (Employés) : 10.0.10.0/24               │
│ • Pool : 10.0.10.50 - 10.0.10.250               │
│ • Durée bail : 8 heures                         │
│                                                 │
│ VLAN 20 (Invités) : 10.0.20.0/24                │
│ • Pool : 10.0.20.50 - 10.0.20.250               │
│ • Durée bail : 2 heures                         │
│ • DNS : 8.8.8.8 (pas de DNS interne)            │
│                                                 │
│ VLAN 30 (Serveurs) : 10.0.30.0/24               │
│ • IPs statiques (pas de DHCP)                   │
│                                                 │
│ VLAN 40 (Imprimantes/IoT) : 10.0.40.0/24        │
│ • Réservations DHCP uniquement                  │
│                                                 │
│ DNS : 10.0.0.5 (Active Directory)               │
│ Passerelle : 10.0.0.1                           │
└─────────────────────────────────────────────────┘
```

### Café / Hôtel (Hotspot public)

```
Configuration hotspot public :
┌─────────────────────────────────────────────────┐
│ Réseau : 172.16.0.0/16                          │
│                                                 │
│ Pool DHCP : 172.16.10.1 - 172.16.50.254         │
│ • ~10 000 adresses disponibles                  │
│ • Durée bail : 1 heure (forte rotation)         │
│                                                 │
│ Passerelle : 172.16.0.1                         │
│ DNS : 8.8.8.8, 1.1.1.1 (publics)                │
│                                                 │
│ Captive portal :                                │
│ • Redirection navigateur vers page d'accueil    │
│ • Acceptation CGU avant accès Internet          │
└─────────────────────────────────────────────────┘
```

## Visualisation d'une configuration DHCP

### Voir sa configuration (client)

**Windows :**
```cmd
C:\> ipconfig /all

Carte réseau sans fil Wi-Fi :
   Adresse IPv4. . . . . . . . . . . : 192.168.1.50
   Masque de sous-réseau. . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . : 192.168.1.1
   Serveur DHCP . . . . . . . . . . : 192.168.1.1
   Bail obtenu. . . . . . . . . . . : jeudi 5 décembre 2024 10:30:00
   Bail expirant. . . . . . . . . . : vendredi 6 décembre 2024 10:30:00
   Serveurs DNS. . . . . . . . . . : 192.168.1.1
                                      8.8.8.8
```

**Linux :**
```bash
$ ip addr show
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.50/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86115sec preferred_lft 86115sec

$ cat /var/lib/dhcp/dhclient.leases
lease {
  interface "eth0";
  fixed-address 192.168.1.50;
  option subnet-mask 255.255.255.0;
  option routers 192.168.1.1;
  option dhcp-lease-time 86400;
  option dhcp-server-identifier 192.168.1.1;
  option domain-name-servers 192.168.1.1, 8.8.8.8;
  renew 5 2024/12/05 22:30:00;
  expire 6 2024/12/06 10:30:00;
}
```

**macOS :**
```bash
$ ipconfig getpacket en0
op = BOOTREPLY
htype = 1
dp_flags = 0
hlen = 6
hops = 0
xid = 0x12345678
secs = 0
ciaddr = 0.0.0.0
yiaddr = 192.168.1.50
siaddr = 192.168.1.1
giaddr = 0.0.0.0
chaddr = 00:1a:2b:3c:4d:5e
options:
  subnet_mask (1): 255.255.255.0
  router (3): 192.168.1.1
  domain_name_server (6): 192.168.1.1, 8.8.8.8
  lease_time (51): 0x15180 (86400)
  dhcp_message_type (53): ACK (0x5)
  server_identifier (54): 192.168.1.1
```

## DHCP vs Configuration statique

### Quand utiliser DHCP ?

```
✅ Utilisez DHCP pour :
────────────────────────
• Postes de travail utilisateurs
• Laptops et appareils mobiles
• Smartphones, tablettes
• Invités et visiteurs
• Appareils personnels (BYOD)
• Appareils IoT grand public
• Tout appareil non critique qui change de réseau
```

### Quand utiliser IP statique ?

```
✅ Utilisez IP statique pour :
─────────────────────────────
• Serveurs (web, fichiers, email, DNS)
• Routeurs et passerelles
• Imprimantes d'entreprise (ou réservation DHCP)
• Équipements réseau (switches, points d'accès)
• Caméras de sécurité (ou réservation DHCP)
• Équipements critiques nécessitant IP prévisible
• Serveurs accessibles depuis Internet
• Équipements en DMZ
```

### Compromis : Réservation DHCP

```
✅ Utilisez réservation DHCP pour :
────────────────────────────────────
• Imprimantes réseau
• Points d'accès WiFi
• Caméras IP
• NAS (Network Attached Storage)
• Équipements nécessitant IP fixe mais config DHCP

Avantages vs IP statique :
• Configuration centralisée
• DNS, passerelle automatiques
• Facilite gestion
• Pas de config sur l'appareil
```

## Points clés à retenir

🔑 **DHCP automatise la configuration réseau des appareils**

🔑 **Élimine les erreurs et le travail manuel de configuration IP**

🔑 **Attribution temporaire via des baux (leases) de durée configurable**

🔑 **Fournit IP, masque, passerelle, DNS et autres paramètres automatiquement**

🔑 **Utilise UDP sur ports 67 (serveur) et 68 (client)**

🔑 **Broadcast initial car client n'a pas encore d'IP**

🔑 **Trois modes : dynamique (standard), automatique (permanent), manuel (réservation)**

🔑 **Serveur DHCP souvent intégré dans box/routeur domestique**

🔑 **Réservation DHCP combine avantages IP fixe + gestion automatique**

🔑 **Point unique de défaillance : haute disponibilité recommandée en entreprise**

---

## Ce que nous avons appris

Dans cette section, nous avons exploré :

- ✅ Le problème résolu par DHCP (automatisation vs configuration manuelle)
- ✅ Les composants : client, serveur, pool d'adresses
- ✅ Les informations fournies par DHCP (IP, masque, passerelle, DNS...)
- ✅ Le concept de bail (lease) et son cycle de vie
- ✅ Les trois types d'attribution (dynamique, automatique, manuelle)
- ✅ Le transport UDP et l'utilisation du broadcast
- ✅ Les avantages et limitations de DHCP
- ✅ Les cas d'usage typiques (domestique, entreprise, hotspot)
- ✅ Quand utiliser DHCP vs IP statique vs réservation

## Pour aller plus loin

Maintenant que vous comprenez le rôle et les concepts de base de DHCP, nous allons explorer en détail le **processus DORA** (Discover, Offer, Request, Acknowledge) : les quatre étapes précises qui permettent à un client d'obtenir sa configuration réseau.

---

**Prochaine section : Processus DORA (Discover, Offer, Request, Acknowledge)** 👉

⏭️ [Processus DORA](/05-couche-application/03.1-dhcp-processus-dora.md)
