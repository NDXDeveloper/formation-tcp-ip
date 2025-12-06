🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.8 SNMP : supervision réseau

## Introduction

Imaginez que vous gérez un réseau d'entreprise avec 100 routeurs, 200 switches, 50 serveurs et des milliers d'utilisateurs. Comment savoir si tout fonctionne correctement ? Quel équipement consomme le plus de bande passante ? Quel serveur a un disque presque plein ? Quel switch a un port défaillant ?

**SNMP (Simple Network Management Protocol)** est le protocole standard qui permet de **surveiller, gérer et diagnostiquer** les équipements réseau de manière centralisée. Créé en 1988, il reste aujourd'hui le protocole de supervision le plus utilisé au monde.

SNMP permet de :
- **Monitorer** l'état des équipements en temps réel
- **Collecter** des statistiques (CPU, mémoire, trafic réseau...)
- **Recevoir des alertes** automatiques en cas de problème
- **Configurer** certains paramètres à distance
- **Générer des rapports** et graphiques de performance

Dans cette section, nous allons explorer SNMP en profondeur : son architecture, ses différentes versions, son fonctionnement, et comment l'utiliser efficacement pour superviser un réseau.

## Présentation de SNMP

### Définition et historique

```
┌──────────────────────────────────────────────────────────────┐
│ SNMP - Simple Network Management Protocol                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Création : 1988                                              │
│ RFC : RFC 1157 (v1), RFC 1901-1908 (v2), RFC 3410-3418 (v3)  │
│ Port : UDP 161 (requêtes), UDP 162 (traps)                   │
│ Transport : UDP (TCP rare, non standard)                     │
│                                                              │
│ Versions :                                                   │
│ ├─ SNMPv1 (1988) : Première version, sécurité faible         │
│ ├─ SNMPv2c (1996) : Améliorations, toujours community        │
│ └─ SNMPv3 (2002) : Sécurité forte, standard actuel ✓         │
│                                                              │
│ Usage : Supervision et gestion équipements réseau            │
│                                                              │
│ Principe :                                                   │
│ "Interroger périodiquement les équipements                   │
│  pour collecter leurs statistiques"                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Pourquoi "Simple" ?**
```
SNMP se voulait simple à l'origine :
├─ Protocole UDP (léger)
├─ Opérations basiques (GET, SET)
├─ Facile à implémenter
└─ Overhead minimal

Réalité :
├─ Concepts complexes (MIB, OID)
├─ Trois versions incompatibles
├─ Sécurité v3 compliquée
└─ "Simple" est ironique aujourd'hui 😅

Mais reste le standard universel !
```

### Philosophie SNMP

```
┌──────────────────────────────────────────────────────────────┐
│ Principe de fonctionnement SNMP                              │
└──────────────────────────────────────────────────────────────┘

Modèle Manager/Agent :

┌─────────────────┐
│  NMS (Manager)  │  Network Management System
│  Station de     │  • Serveur de supervision (Nagios, PRTG...)
│  supervision    │  • Envoie requêtes
│                 │  • Reçoit réponses et alertes
│                 │  • Affiche graphiques, alertes
└────────┬────────┘
         │
         │ Réseau IP
         │
    ┌────┴────┬────────┬────────┬────────┐
    │         │        │        │        │
┌───▼───┐ ┌───▼──┐ ┌───▼───┐ ┌───▼──┐ ┌───▼────┐
│Agent  │ │Agent │ │Agent  │ │Agent │ │Agent   │
│SNMP   │ │SNMP  │ │SNMP   │ │SNMP  │ │SNMP    │
├───────┤ ├──────┤ ├───────┤ ├──────┤ ├────────┤
│Routeur│ │Switch│ │Serveur│ │Imprim│ │Firewall│
└───────┘ └──────┘ └───────┘ └──────┘ └────────┘

Flux d'informations :

1. Polling (Manager → Agent) :
   Manager : "Quelle est ta charge CPU ?"
   Agent   : "CPU = 45%"

2. Trap (Agent → Manager) :
   Agent   : "ALERTE ! Interface eth0 down !"
   Manager : [Déclenche alarme]
```

## Architecture SNMP

### Les composants

**1. Manager (NMS - Network Management System)**
```
Rôle : Station de supervision centralisée

Fonctions :
├─ Envoyer requêtes aux agents (polling)
├─ Recevoir réponses
├─ Recevoir traps (alertes)
├─ Stocker données historiques
├─ Générer graphiques et rapports
└─ Déclencher alertes

Exemples de logiciels :
├─ Nagios / Icinga (open source)
├─ Zabbix (open source)
├─ PRTG Network Monitor (commercial)
├─ SolarWinds (commercial)
├─ LibreNMS (open source)
└─ Observium (open source)
```

**2. Agent SNMP**
```
Rôle : Logiciel sur équipement supervisé

Fonctions :
├─ Écouter sur port UDP 161
├─ Répondre aux requêtes du manager
├─ Collecter informations locales
├─ Envoyer traps vers manager
└─ Exposer MIB (base d'informations)

Implémentation :
├─ net-snmp (Linux/Unix, le plus utilisé)
├─ Intégré dans routeurs Cisco, HP, Juniper...
├─ Windows SNMP Service
├─ Agents dédiés pour applications (MySQL, Apache...)
└─ Agents matériels (imprimantes, onduleurs...)

Emplacement : Sur chaque équipement supervisé
```

**3. MIB (Management Information Base)**
```
Rôle : Base de données hiérarchique d'informations

Définition :
├─ Structure arborescente d'objets
├─ Chaque objet a un OID unique
├─ Décrit variables disponibles
└─ Format texte (ASN.1)

Exemple d'objet MIB :
Nom   : sysUpTime
OID   : 1.3.6.1.2.1.1.3.0
Type  : TimeTicks
Accès : read-only
Desc  : "Temps depuis dernier redémarrage"

Types de MIB :
├─ MIB-II (standard, RFC 1213)
├─ MIB privées (constructeur : Cisco, HP...)
└─ MIB d'entreprise (applications spécifiques)
```

### Hiérarchie OID

Les **OID (Object Identifier)** sont des identifiants uniques organisés en arbre :

```
┌──────────────────────────────────────────────────────────────┐
│ Arbre OID SNMP (simplifié)                                   │
└──────────────────────────────────────────────────────────────┘

                          . (root)
                            │
                ┌───────────┼───────────┐
                │           │           │
              iso(1)      itu(0)     joint(2)
                │
            org(3)
                │
            dod(6)
                │
            internet(1)
                │
        ┌───────┼───────┬───────┬───────┐
        │       │       │       │       │
    directory mgmt(2) experimental private security
      (1)      │      (3)      (4)     (5)
               │
        ┌──────┴──────┬────────┬────────┐
        │             │        │        │
      mib-2(1)     interfaces system   ip
                     (2)       (1)      (4)
                       │        │
                   ifNumber  sysDescr
                   (1)       (1)

OID complet :
1.3.6.1.2.1.1.1.0
│ │ │ │ │ │ │ └─ Instance (0 = scalar)
│ │ │ │ │ │ └─── sysDescr
│ │ │ │ │ └───── system
│ │ │ │ └─────── mib-2
│ │ │ └───────── mgmt
│ │ └─────────── internet
│ └───────────── dod
└─────────────── org

Notation :
├─ Numérique : 1.3.6.1.2.1.1.1.0
└─ Textuelle : iso.org.dod.internet.mgmt.mib-2.system.sysDescr.0
```

**OID MIB-II standards (RFC 1213) :**

```
┌──────────────────────────────────────────────────────────────┐
│ OID MIB-II courants (1.3.6.1.2.1.x)                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ .1.1   : system (informations système)                       │
│   .1.1.1 : sysDescr (description système)                    │
│   .1.1.2 : sysObjectID                                       │
│   .1.1.3 : sysUpTime (uptime depuis boot)                    │
│   .1.1.4 : sysContact (contact admin)                        │
│   .1.1.5 : sysName (nom hôte)                                │
│   .1.1.6 : sysLocation (localisation physique)               │
│                                                              │
│ .1.2   : interfaces (interfaces réseau)                      │
│   .1.2.1 : ifNumber (nombre d'interfaces)                    │
│   .1.2.2.1.x.Y : ifTable (table des interfaces)              │
│     .1.2.2.1.1.Y : ifIndex                                   │
│     .1.2.2.1.2.Y : ifDescr (nom interface)                   │
│     .1.2.2.1.5.Y : ifSpeed (vitesse en bps)                  │
│     .1.2.2.1.7.Y : ifAdminStatus (état admin)                │
│     .1.2.2.1.8.Y : ifOperStatus (état opérationnel)          │
│     .1.2.2.1.10.Y : ifInOctets (octets reçus)                │
│     .1.2.2.1.16.Y : ifOutOctets (octets envoyés)             │
│                                                              │
│ .1.4   : ip (statistiques IP)                                │
│ .1.5   : icmp (statistiques ICMP)                            │
│ .1.6   : tcp (statistiques TCP)                              │
│ .1.7   : udp (statistiques UDP)                              │
│ .1.25  : host (ressources système)                           │
│   .1.25.2.3.1.x : hrStorageTable (stockage)                  │
│   .1.25.3.3.1.x : hrProcessorTable (CPU)                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Note : Y = index de l'interface (1, 2, 3...)
```

## Opérations SNMP

### Types d'opérations

```
┌──────────────────────────────────────────────────────────────┐
│ Opérations SNMP (PDU - Protocol Data Units)                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ MANAGER → AGENT (Requêtes)                                   │
│                                                              │
│ GET                                                          │
│ ├─ Récupérer valeur d'un OID spécifique                      │
│ ├─ Exemple : "Donne-moi sysUpTime"                           │
│ └─ Réponse : Valeur unique                                   │
│                                                              │
│ GET-NEXT                                                     │
│ ├─ Récupérer OID suivant dans l'arbre                        │
│ ├─ Utilisé pour parcourir MIB                                │
│ └─ Exemple : Explorer toutes les interfaces                  │
│                                                              │
│ GET-BULK (v2c et v3 uniquement)                              │
│ ├─ Récupérer plusieurs OID en une requête                    │
│ ├─ Plus efficace que GET-NEXT multiple                       │
│ └─ Gain de performance significatif                          │
│                                                              │
│ SET                                                          │
│ ├─ Modifier valeur d'un OID                                  │
│ ├─ Exemple : Changer sysContact                              │
│ └─ ⚠️ Dangereux si mal utilisé                               │
│                                                              │
│ AGENT → MANAGER (Notifications)                              │
│                                                              │
│ TRAP (v1, v2c, v3)                                           │
│ ├─ Notification asynchrone (non sollicitée)                  │
│ ├─ Envoyée quand événement se produit                        │
│ ├─ Exemple : "Interface down !"                              │
│ └─ UDP, pas de confirmation (fire-and-forget)                │
│                                                              │
│ INFORM (v2c et v3 uniquement)                                │
│ ├─ Comme TRAP mais avec accusé de réception                  │
│ ├─ Manager confirme réception                                │
│ └─ Plus fiable que TRAP                                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Exemple de dialogue SNMP

```
┌──────────────────────────────────────────────────────────────┐
│ Session SNMP typique (polling)                               │
└──────────────────────────────────────────────────────────────┘

Manager (192.168.1.10)            Agent (192.168.1.50)
     │                                   │
     │ GET sysDescr.0                    │
     ├──────────────────────────────────>│
     │ Community: public                 │
     │                                   │
     │                       [Lit valeur]│
     │ Response: sysDescr.0              │
     │<──────────────────────────────────┤
     │ "Cisco IOS 15.2, Router"          │
     │                                   │
     │ GET sysUpTime.0                   │
     ├──────────────────────────────────>│
     │                                   │
     │ Response: sysUpTime.0             │
     │<──────────────────────────────────┤
     │ "157345823" (centièmes de sec)    │
     │ = 18 jours, 5h, 16min             │
     │                                   │
     │ GET ifOperStatus.1                │
     ├──────────────────────────────────>│
     │ (état interface #1)               │
     │                                   │
     │ Response: ifOperStatus.1          │
     │<──────────────────────────────────┤
     │ "1" (up)                          │
     │                                   │

Manager enregistre valeurs, génère graphiques, vérifie seuils
```

### TRAP - Alerte asynchrone

```
┌──────────────────────────────────────────────────────────────┐
│ TRAP SNMP - Notification événement                          │
└──────────────────────────────────────────────────────────────┘

Scénario : Interface réseau tombe

Agent (192.168.1.50)              Manager (192.168.1.10)
     │                                    │
     │ [Interface eth0 down]              │
     │                                    │
     │ TRAP linkDown                      │
     ├───────────────────────────────────>│
     │ Port UDP 162                       │
     │ OID: linkDown (1.3.6.1.6.3.1.1.5.3)│
     │ Variables:                         │
     │  - ifIndex: 1                      │
     │  - ifDescr: eth0                   │
     │  - ifOperStatus: down(2)           │
     │                                    │
     │                  [Déclenche alerte]│
     │                  [Email admin]     │
     │                  [SMS]             │
     │                  [Log]             │

Pas de réponse de Manager (TRAP = fire-and-forget)

Traps standards (v1) :
├─ coldStart (0) : Redémarrage complet
├─ warmStart (1) : Redémarrage application
├─ linkDown (2) : Interface down
├─ linkUp (3) : Interface up
├─ authenticationFailure (4) : Auth SNMP échouée
└─ egpNeighborLoss (5) : Perte voisin EGP

Traps personnalisées : Définies par constructeur/application
```

## Versions de SNMP

### SNMPv1 (1988)

```
┌──────────────────────────────────────────────────────────────┐
│ SNMPv1 - Version originale                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ RFC : RFC 1157                                               │
│                                                              │
│ Sécurité : Community String                                  │
│ ├─ Chaîne de caractères simple (mot de passe)                │
│ ├─ Transmise EN CLAIR dans paquets UDP ✗                     │
│ ├─ Pas de chiffrement                                        │
│ └─ Pas d'authentification réelle                             │
│                                                              │
│ Community strings :                                          │
│ ├─ "public" : Lecture seule (défaut)                         │
│ └─ "private" : Lecture/écriture (défaut)                     │
│                                                              │
│ Opérations supportées :                                      │
│ ├─ GET                                                       │
│ ├─ GET-NEXT                                                  │
│ ├─ SET                                                       │
│ └─ TRAP                                                      │
│                                                              │
│ Types de données :                                           │
│ ├─ INTEGER                                                   │
│ ├─ OCTET STRING                                              │
│ ├─ NULL                                                      │
│ ├─ OBJECT IDENTIFIER                                         │
│ └─ IpAddress, Counter, Gauge, TimeTicks                      │
│                                                              │
│ Problèmes :                                                  │
│ ✗ Sécurité inexistante                                       │
│ ✗ Community en clair = sniffable                             │
│ ✗ Pas de confirmation pour TRAP                              │
│ ✗ GET-NEXT inefficace pour gros volumes                      │
│                                                              │
│ Statut actuel : OBSOLÈTE mais encore utilisé (legacy)        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Capture d'un paquet SNMPv1 :**

```
┌──────────────────────────────────────────────────────────────┐
│ Wireshark - Paquet SNMP GET (v1)                             │
└──────────────────────────────────────────────────────────────┘

Simple Network Management Protocol
    version: v1 (0)
    community: public  ← EN CLAIR ! Visible dans Wireshark
    data: get-request (0)
        request-id: 12345
        error-status: noError (0)
        error-index: 0
        variable-bindings: 1 item
            1.3.6.1.2.1.1.1.0: Value (NULL)
                Object Name: 1.3.6.1.2.1.1.1.0 (sysDescr.0)
                Value (NULL)

Problème : "public" visible en clair
→ Attaquant sur réseau peut lire et rejouer requêtes
```

### SNMPv2c (1996)

```
┌──────────────────────────────────────────────────────────────┐
│ SNMPv2c - Version améliorée                                  │
├──────────────────────────────────────────────────────────────┤
│
│ RFC : RFC 1901-1908 (initialement), puis RFC 3416-3418
│ "c" = Community-based (même sécurité que v1)
│
│ Sécurité : Identique à v1 ✗
│ ├─ Community string en clair
│ └─ Aucune amélioration sécurité
│
│ Améliorations fonctionnelles :
│
│ ✓ GET-BULK
│   ├─ Récupération multiple valeurs en une requête
│   ├─ Remplace GET-NEXT répétés
│   └─ Gain performance majeur
│
│ ✓ INFORM
│   ├─ Trap avec accusé de réception
│   └─ Plus fiable que TRAP v1
│
│ ✓ Nouveaux types de données
│   ├─ Counter64 (compteurs 64 bits)
│   └─ Gestion hautes valeurs
│
│ ✓ Meilleure gestion erreurs
│   └─ Codes d'erreur plus détaillés
│
│ Avantages :
│ ✓ Performance améliorée (GET-BULK)
│ ✓ Compatibilité ascendante avec v1
│ ✓ Plus largement supporté
│
│ Inconvénients :
│ ✗ Toujours aucune sécurité
│ ✗ Community en clair
│
│ Statut : Très utilisé (compromis fonctionnalité/simplicité)
│
└──────────────────────────────────────────────────────────────┘
```

### SNMPv3 (2002)

```
┌──────────────────────────────────────────────────────────────┐
│ SNMPv3 - Version sécurisée (standard actuel)                 │
├──────────────────────────────────────────────────────────────┤
│
│ RFC : RFC 3410-3418
│
│ SÉCURITÉ ENFIN ! ✓✓✓
│
│ Architecture modulaire USM/VACM :
│
│ USM (User-based Security Model)
│ ├─ Authentification
│ │  ├─ HMAC-MD5 (96 bits)
│ │  ├─ HMAC-SHA (96/128/192/256/384/512 bits)
│ │  └─ Vérifie identité + intégrité
│ │
│ ├─ Chiffrement (Privacy)
│ │  ├─ DES (56 bits, obsolète)
│ │  ├─ 3DES (168 bits)
│ │  ├─ AES-128, AES-192, AES-256 ✓
│ │  └─ Protège confidentialité
│ │
│ └─ Protection anti-replay
│    └─ Timestamps, numéros de séquence
│
│ VACM (View-based Access Control Model)
│ ├─ Contrôle d'accès granulaire
│ ├─ Définir qui peut lire/écrire quoi
│ └─ Groupes, vues, contextes
│
│ Niveaux de sécurité :
│
│ noAuthNoPriv
│ ├─ Pas d'authentification
│ ├─ Pas de chiffrement
│ └─ = SNMPv1/v2c (non recommandé)
│
│ authNoPriv
│ ├─ Authentification (HMAC-SHA) ✓
│ ├─ Pas de chiffrement
│ └─ Intégrité garantie
│
│ authPriv ✓✓ (RECOMMANDÉ)
│ ├─ Authentification (HMAC-SHA) ✓
│ ├─ Chiffrement (AES) ✓
│ └─ Sécurité maximale
│
│ Avantages :
│ ✓✓ Sécurité forte (enfin !)
│ ✓✓ Chiffrement bout-en-bout
│ ✓✓ Authentification mutuelle
│ ✓ Contrôle d'accès granulaire
│ ✓ Protection anti-replay
│
│ Inconvénients :
│ ✗ Configuration complexe
│ ✗ Overhead performance (chiffrement)
│ ✗ Moins supporté que v2c (legacy)
│
│ Statut : STANDARD RECOMMANDÉ pour nouvelles installations
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Configuration SNMPv3 exemple (net-snmp) :**

```bash
# /etc/snmp/snmpd.conf

# Créer utilisateur avec authentification + chiffrement
createUser monitorUser SHA "AuthPassword123!" AES "PrivPassword456!"

# Définir accès
rouser monitorUser authPriv

# Vue (quels OID accessibles)
view systemview included .1.3.6.1.2.1.1
view systemview included .1.3.6.1.2.1.2

# Groupe avec vue
group MyROGroup usm monitorUser
access MyROGroup "" usm authPriv exact systemview none none

# Requête depuis client :
snmpget -v3 -l authPriv \
  -u monitorUser \
  -a SHA -A "AuthPassword123!" \
  -x AES -X "PrivPassword456!" \
  192.168.1.50 sysDescr.0
```

### Comparaison des versions

```
┌───────────────────────────────────────────────────────────────────────┐
│ SNMPv1 vs v2c vs v3                                                   │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ Critère          │ v1           │ v2c          │ v3
│──────────────────┼──────────────┼──────────────┼─────────────────────
│ Année            │ 1988         │ 1996         │ 2002
│ Sécurité         │ Aucune ✗     │ Aucune ✗     │ Forte ✓✓✓
│ Authentification │ Community    │ Community    │ Username/Password
│ Chiffrement      │ Non          │ Non          │ Oui (AES) ✓
│ GET-BULK         │ Non          │ Oui ✓        │ Oui ✓
│ INFORM           │ Non          │ Oui ✓        │ Oui ✓
│ Counter64        │ Non          │ Oui ✓        │ Oui ✓
│ Complexité       │ Simple       │ Simple       │ Complexe
│ Performance      │ Bonne        │ Meilleure ✓  │ Bonne (overhead)
│ Configuration    │ Facile       │ Facile       │ Difficile
│ Support          │ Universel    │ Universel    │ Bon (moderne)
│ Usage moderne    │ Legacy       │ Courant      │ Recommandé ✓
│
│ RECOMMANDATIONS :
│ ├─ Nouveau déploiement : SNMPv3 (sécurité) ✓✓
│ ├─ Réseau interne confiance : v2c acceptable
│ ├─ Internet/WAN : v3 OBLIGATOIRE
│ └─ Legacy : v1 seulement si pas le choix
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

## Utilisation pratique de SNMP

### Outils en ligne de commande

**net-snmp (Linux/Unix) :**

```bash
┌──────────────────────────────────────────────────────────────┐
│ net-snmp - Suite d'outils SNMP                               │
└──────────────────────────────────────────────────────────────┘

# Installation
sudo apt install snmp snmp-mibs-downloader  # Debian/Ubuntu
sudo yum install net-snmp net-snmp-utils    # RHEL/CentOS

# Activer MIB textuelles (Ubuntu)
sudo sed -i 's/mibs :/# mibs :/g' /etc/snmp/snmp.conf

# ────────────────────────────────────────────────────────

# snmpget - Récupérer une valeur
snmpget -v2c -c public 192.168.1.50 sysDescr.0
SNMPv2-MIB::sysDescr.0 = STRING: Cisco IOS Software, Version 15.2

# Avec OID numérique
snmpget -v2c -c public 192.168.1.50 1.3.6.1.2.1.1.1.0

# ────────────────────────────────────────────────────────

# snmpwalk - Parcourir un sous-arbre
snmpwalk -v2c -c public 192.168.1.50 system
SNMPv2-MIB::sysDescr.0 = STRING: Cisco IOS...
SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.9.1.1
SNMPv2-MIB::sysUpTime.0 = Timeticks: (157345823) 18 days, 5:16:38
SNMPv2-MIB::sysContact.0 = STRING: admin@example.com
SNMPv2-MIB::sysName.0 = STRING: router01
SNMPv2-MIB::sysLocation.0 = STRING: Datacenter A, Rack 5

# Parcourir toutes les interfaces
snmpwalk -v2c -c public 192.168.1.50 interfaces

# ────────────────────────────────────────────────────────

# snmpbulkwalk - Version optimisée (v2c/v3)
snmpbulkwalk -v2c -c public 192.168.1.50 ifTable

# ────────────────────────────────────────────────────────

# snmpset - Modifier une valeur
snmpset -v2c -c private 192.168.1.50 \
  sysContact.0 s "nouvel-admin@example.com"

# Types de données :
# i = INTEGER
# u = unsigned INTEGER
# t = TIMETICKS
# a = IPADDRESS
# o = OBJID
# s = STRING
# x = HEX STRING
# d = DECIMAL STRING

# ────────────────────────────────────────────────────────

# snmptranslate - Traduire OID
snmptranslate -On SNMPv2-MIB::sysDescr.0
.1.3.6.1.2.1.1.1.0

snmptranslate -Of .1.3.6.1.2.1.1.1.0
.iso.org.dod.internet.mgmt.mib-2.system.sysDescr.0

# ────────────────────────────────────────────────────────

# snmptable - Afficher table formatée
snmptable -v2c -c public 192.168.1.50 ifTable

SNMP table: IF-MIB::ifTable
 ifIndex ifDescr           ifSpeed      ifOperStatus
       1 FastEthernet0/0   100000000    up(1)
       2 FastEthernet0/1   100000000    down(2)
       3 GigabitEthernet0  1000000000   up(1)

# ────────────────────────────────────────────────────────

# SNMPv3 avec authentification
snmpget -v3 -l authPriv \
  -u myuser \
  -a SHA -A "AuthPass123" \
  -x AES -X "PrivPass456" \
  192.168.1.50 sysDescr.0
```

### Surveillance du trafic réseau

```bash
┌──────────────────────────────────────────────────────────────┐
│ Monitorer trafic interface avec SNMP                         │
└──────────────────────────────────────────────────────────────┘

# Récupérer trafic interface 1
snmpget -v2c -c public 192.168.1.50 \
  ifInOctets.1 ifOutOctets.1

IF-MIB::ifInOctets.1 = Counter32: 1234567890
IF-MIB::ifOutOctets.1 = Counter32: 9876543210

# Script pour calculer bande passante

#!/bin/bash
HOST="192.168.1.50"
COMMUNITY="public"
INTERFACE=1
INTERVAL=5

# Première mesure
IN1=$(snmpget -v2c -c $COMMUNITY -Oqv $HOST ifInOctets.$INTERFACE)
OUT1=$(snmpget -v2c -c $COMMUNITY -Oqv $HOST ifOutOctets.$INTERFACE)

sleep $INTERVAL

# Deuxième mesure
IN2=$(snmpget -v2c -c $COMMUNITY -Oqv $HOST ifInOctets.$INTERFACE)
OUT2=$(snmpget -v2c -c $COMMUNITY -Oqv $HOST ifOutOctets.$INTERFACE)

# Calcul bande passante (octets/sec puis Mbps)
IN_BPS=$(( ($IN2 - $IN1) / $INTERVAL ))
OUT_BPS=$(( ($OUT2 - $OUT1) / $INTERVAL ))

IN_MBPS=$(echo "scale=2; $IN_BPS * 8 / 1000000" | bc)
OUT_MBPS=$(echo "scale=2; $OUT_BPS * 8 / 1000000" | bc)

echo "Interface $INTERFACE :"
echo "  Débit entrant  : $IN_MBPS Mbps"
echo "  Débit sortant  : $OUT_MBPS Mbps"

# Résultat :
# Interface 1 :
#   Débit entrant  : 45.23 Mbps
#   Débit sortant  : 12.67 Mbps
```

### Logiciels de supervision

**Nagios / Icinga :**

```bash
┌──────────────────────────────────────────────────────────────┐
│ Nagios - Système de supervision open source                  │
└──────────────────────────────────────────────────────────────┘

# Configuration check SNMP

# /etc/nagios/objects/commands.cfg
define command {
    command_name    check_snmp
    command_line    $USER1$/check_snmp -H $HOSTADDRESS$ \
                    -C $ARG1$ -o $ARG2$ $ARG3$
}

# /etc/nagios/objects/hosts/router01.cfg
define host {
    use                 generic-router
    host_name           router01
    address             192.168.1.50
    _SNMPCOMMUNITY      public
}

define service {
    use                 generic-service
    host_name           router01
    service_description CPU Load
    check_command       check_snmp!public!.1.3.6.1.4.1.9.9.109.1.1.1.1.5.1!-w 70 -c 90
}

define service {
    use                 generic-service
    host_name           router01
    service_description Interface Fa0/0 Status
    check_command       check_snmp!public!ifOperStatus.1!-r 1
}

# Alertes si CPU > 70% (warning) ou > 90% (critical)
# Alerte si interface pas "up(1)"
```

**PRTG Network Monitor :**

```
┌──────────────────────────────────────────────────────────────┐
│ PRTG - Interface graphique (Windows)                         │
└──────────────────────────────────────────────────────────────┘

Configuration :
1. Add Device → IP: 192.168.1.50
2. SNMP Settings :
   ├─ Version : v2c
   └─ Community : public

3. Auto-Discovery :
   ├─ PRTG découvre automatiquement :
   │  ├─ Interfaces réseau
   │  ├─ CPU / Mémoire
   │  └─ Disques

4. Sensors créés automatiquement :
   ├─ SNMP Traffic (chaque interface)
   ├─ SNMP CPU Load
   ├─ SNMP Memory
   └─ SNMP Disk Space

5. Graphiques générés en temps réel
6. Alertes configurables
7. Rapports automatiques

Interface web : https://prtg.example.com
```

**Zabbix :**

```yaml
┌──────────────────────────────────────────────────────────────┐
│ Zabbix - Supervision enterprise open source                  │
└──────────────────────────────────────────────────────────────┘

# Template pour routeur Cisco

zabbix_export:
  version: '6.0'
  templates:
    - template: 'Cisco Router SNMP'
      name: 'Cisco Router SNMP'
      groups:
        - name: 'Templates/Network devices'
      items:
        - name: 'Device description'
          type: SNMP_AGENT
          snmp_oid: 1.3.6.1.2.1.1.1.0
          key: system.descr

        - name: 'Uptime'
          type: SNMP_AGENT
          snmp_oid: 1.3.6.1.2.1.1.3.0
          key: system.uptime

        - name: 'CPU utilization'
          type: SNMP_AGENT
          snmp_oid: 1.3.6.1.4.1.9.9.109.1.1.1.1.5.1
          key: system.cpu.util

      discovery_rules:
        - name: 'Network interfaces discovery'
          type: SNMP_AGENT
          snmp_oid: 'discovery[{#IFNAME},1.3.6.1.2.1.2.2.1.2]'
          item_prototypes:
            - name: 'Interface {#IFNAME}: Bits received'
              type: SNMP_AGENT
              snmp_oid: '1.3.6.1.2.1.2.2.1.10.{#SNMPINDEX}'

      triggers:
        - name: 'High CPU usage on {HOST.NAME}'
          expression: 'last(/Cisco Router SNMP/system.cpu.util)>90'
          priority: HIGH

Fonctionnalités :
✓ Auto-discovery appareils et services
✓ Templates réutilisables
✓ Graphiques temps réel et historiques
✓ Alertes multi-canal (email, SMS, Slack...)
✓ API pour automatisation
✓ Dashboards personnalisables
```

## MIB courantes et exemples

### Informations système

```bash
# Description système
$ snmpget -v2c -c public router01 sysDescr.0
SNMPv2-MIB::sysDescr.0 = STRING: Cisco IOS Software, C2960 Software

# Uptime (centièmes de seconde)
$ snmpget -v2c -c public router01 sysUpTime.0
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (157345823) 18 days, 5:16:38

# Contact administrateur
$ snmpget -v2c -c public router01 sysContact.0
SNMPv2-MIB::sysContact.0 = STRING: admin@example.com

# Nom d'hôte
$ snmpget -v2c -c public router01 sysName.0
SNMPv2-MIB::sysName.0 = STRING: router01.example.com

# Localisation
$ snmpget -v2c -c public router01 sysLocation.0
SNMPv2-MIB::sysLocation.0 = STRING: Datacenter Paris, Rack A5
```

### Interfaces réseau

```bash
# Nombre d'interfaces
$ snmpget -v2c -c public router01 ifNumber.0
IF-MIB::ifNumber.0 = INTEGER: 24

# Liste des interfaces
$ snmpwalk -v2c -c public router01 ifDescr
IF-MIB::ifDescr.1 = STRING: FastEthernet0/1
IF-MIB::ifDescr.2 = STRING: FastEthernet0/2
IF-MIB::ifDescr.3 = STRING: GigabitEthernet0/1
...

# État opérationnel (1=up, 2=down, 3=testing)
$ snmpwalk -v2c -c public router01 ifOperStatus
IF-MIB::ifOperStatus.1 = INTEGER: up(1)
IF-MIB::ifOperStatus.2 = INTEGER: down(2)
IF-MIB::ifOperStatus.3 = INTEGER: up(1)

# Vitesse interface (bps)
$ snmpget -v2c -c public router01 ifSpeed.1
IF-MIB::ifSpeed.1 = Gauge32: 100000000  (100 Mbps)

# Trafic entrant/sortant (octets)
$ snmpwalk -v2c -c public router01 ifInOctets
IF-MIB::ifInOctets.1 = Counter32: 1234567890
IF-MIB::ifInOctets.2 = Counter32: 987654321

$ snmpwalk -v2c -c public router01 ifOutOctets
IF-MIB::ifOutOctets.1 = Counter32: 9876543210
IF-MIB::ifOutOctets.2 = Counter32: 1234567890

# Erreurs et pertes
$ snmpget -v2c -c public router01 ifInErrors.1
IF-MIB::ifInErrors.1 = Counter32: 42

$ snmpget -v2c -c public router01 ifOutDiscards.1
IF-MIB::ifOutDiscards.1 = Counter32: 15
```

### Ressources système (HOST-RESOURCES-MIB)

```bash
# CPU Load (dépend du constructeur)
# Cisco
$ snmpwalk -v2c -c public router01 1.3.6.1.4.1.9.9.109.1.1.1.1.5
...cpmCPUTotal5minRev.1 = Gauge32: 45 percent

# Net-SNMP (Linux)
$ snmpget -v2c -c public server01 .1.3.6.1.4.1.2021.11.9.0
UCD-SNMP-MIB::ssCpuUser.0 = INTEGER: 15

# Mémoire
$ snmpwalk -v2c -c public server01 hrStorageTable
HOST-RESOURCES-MIB::hrStorageIndex.1 = INTEGER: 1
HOST-RESOURCES-MIB::hrStorageDescr.1 = STRING: Physical memory
HOST-RESOURCES-MIB::hrStorageSize.1 = INTEGER: 8388608 KBytes
HOST-RESOURCES-MIB::hrStorageUsed.1 = INTEGER: 5242880 KBytes

# Disques
$ snmpwalk -v2c -c public server01 hrStorageDescr | grep "^/"
HOST-RESOURCES-MIB::hrStorageDescr.31 = STRING: /
HOST-RESOURCES-MIB::hrStorageDescr.32 = STRING: /home
HOST-RESOURCES-MIB::hrStorageDescr.33 = STRING: /var

$ snmpget -v2c -c public server01 \
  hrStorageSize.31 hrStorageUsed.31
HOST-RESOURCES-MIB::hrStorageSize.31 = INTEGER: 51475068 (partition /)
HOST-RESOURCES-MIB::hrStorageUsed.31 = INTEGER: 12345678
# Calcul : 12345678 / 51475068 = 24% utilisé
```

### Processus en cours

```bash
# Liste des processus
$ snmpwalk -v2c -c public server01 hrSWRunName
HOST-RESOURCES-MIB::hrSWRunName.1 = STRING: "systemd"
HOST-RESOURCES-MIB::hrSWRunName.573 = STRING: "sshd"
HOST-RESOURCES-MIB::hrSWRunName.842 = STRING: "apache2"
HOST-RESOURCES-MIB::hrSWRunName.1024 = STRING: "mysqld"

# Performance d'un processus
$ snmpget -v2c -c public server01 hrSWRunPerfCPU.1024
HOST-RESOURCES-MIB::hrSWRunPerfCPU.1024 = INTEGER: 1523 centiseconds

$ snmpget -v2c -c public server01 hrSWRunPerfMem.1024
HOST-RESOURCES-MIB::hrSWRunPerfMem.1024 = INTEGER: 524288 KBytes
```

## Configuration agent SNMP

### Linux (net-snmp)

```bash
┌──────────────────────────────────────────────────────────────┐
│ Configuration agent SNMP sur Linux (net-snmp)                │
└──────────────────────────────────────────────────────────────┘

# Installation
sudo apt install snmpd  # Debian/Ubuntu
sudo yum install net-snmp  # RHEL/CentOS

# Configuration : /etc/snmp/snmpd.conf

# ──────────── SNMPv1/v2c ────────────

# Community en lecture seule (restreindre IP)
rocommunity public 192.168.1.0/24
# ou pour toutes IPs (dangereux) :
# rocommunity public

# Community en lecture/écriture
rwcommunity private 192.168.1.10

# ──────────── SNMPv3 ────────────

# Créer utilisateur
createUser monitorUser SHA "MyAuthPass123" AES "MyPrivPass456"

# Donner accès lecture seule
rouser monitorUser authPriv

# ──────────── INFORMATIONS SYSTÈME ────────────

syslocation "Datacenter Paris, Rack A5, U10"
syscontact "admin@example.com"
sysname "server01.example.com"

# ──────────── ÉCOUTE ────────────

# Écouter sur toutes interfaces
agentAddress udp:161
# Ou restreindre à IP spécifique
# agentAddress udp:192.168.1.50:161

# ──────────── EXTENSIONS ────────────

# Activer disques/CPU/processus
disk / 10%
load 12 10 5
proc sshd
proc apache2

# ──────────── TRAPS ────────────

# Envoyer traps vers manager
trap2sink 192.168.1.10 public
# ou v3
# trapsess -v3 -u trapuser -a SHA -A AuthPass -x AES -X PrivPass 192.168.1.10

# Redémarrer service
sudo systemctl restart snmpd
sudo systemctl enable snmpd

# Vérifier écoute
sudo netstat -ulnp | grep 161
udp        0      0 0.0.0.0:161             0.0.0.0:*                           12345/snmpd

# Tester localement
snmpwalk -v2c -c public localhost system
```

### Windows

```powershell
┌──────────────────────────────────────────────────────────────┐
│ Configuration SNMP sur Windows                               │
└──────────────────────────────────────────────────────────────┘

# Installation via PowerShell (Windows Server)
Install-WindowsFeature SNMP-Service -IncludeManagementTools

# Ou via interface graphique :
# Server Manager → Add Roles and Features → Features → SNMP Service

# Configuration : services.msc → SNMP Service → Properties

Agent Tab :
├─ Contact : admin@example.com
└─ Location : Office Building 2, Floor 3

Traps Tab :
├─ Community name : public
└─ Trap destinations : 192.168.1.10 (manager)

Security Tab :
├─ Accepted community names :
│  ├─ public (READ ONLY)
│  └─ private (READ WRITE)
└─ Accept SNMP packets from these hosts :
   ├─ 192.168.1.0 (subnet)
   └─ 192.168.1.10 (manager specific)

# Redémarrer service
Restart-Service SNMP

# Tester
snmpwalk -v2c -c public 127.0.0.1 system
```

### Routeur Cisco

```
┌──────────────────────────────────────────────────────────────┐
│ Configuration SNMP sur routeur/switch Cisco                  │
└──────────────────────────────────────────────────────────────┘

! Connexion au routeur
Router> enable
Router# configure terminal

! ──────────── SNMPv2c ────────────

! Community lecture seule
Router(config)# snmp-server community public RO

! Community lecture/écriture
Router(config)# snmp-server community private RW

! Avec ACL pour restreindre IPs
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# snmp-server community public RO 10

! ──────────── SNMPv3 ────────────

! Créer groupe
Router(config)# snmp-server group MonitorGroup v3 priv

! Créer utilisateur
Router(config)# snmp-server user monitoruser MonitorGroup v3 \
                 auth sha MyAuthPass priv aes 128 MyPrivPass

! ──────────── INFORMATIONS ────────────

Router(config)# snmp-server contact admin@example.com
Router(config)# snmp-server location "Datacenter A, Rack 12"

! ──────────── TRAPS ────────────

! Activer traps
Router(config)# snmp-server enable traps snmp linkdown linkup
Router(config)# snmp-server enable traps config
Router(config)# snmp-server enable traps cpu threshold

! Destination traps
Router(config)# snmp-server host 192.168.1.10 version 2c public

! ──────────── VÉRIFICATION ────────────

Router(config)# exit
Router# show snmp
Router# show snmp community
Router# show snmp user

! Sauvegarder
Router# write memory
```

## Sécurité SNMP

### Bonnes pratiques

```
┌──────────────────────────────────────────────────────────────┐
│ Checklist sécurité SNMP                                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ✓✓ ESSENTIEL
│
│ ├─ Utiliser SNMPv3 avec authPriv si possible ✓✓
│ │  └─ Seule version sécurisée
│ │
│ ├─ Changer communities par défaut ✓✓
│ │  ├─ "public" et "private" = connus de tous
│ │  └─ Utiliser chaînes complexes aléatoires
│ │
│ ├─ Restreindre IPs autorisées ✓✓
│ │  ├─ ACL sur agent
│ │  ├─ Firewall
│ │  └─ Seulement manager(s) autorisé(s)
│ │
│ ├─ Lecture seule par défaut ✓✓
│ │  └─ Éviter RW (SET) sauf nécessaire
│ │
│ └─ Désactiver si non utilisé ✓
│    └─ Principe de moindre privilège
│
│ ✓ RECOMMANDÉ
│
│ ├─ Isoler traffic SNMP (VLAN management)
│ ├─ Chiffrer avec VPN si via Internet
│ ├─ Logs et monitoring accès SNMP
│ ├─ Traps authentication failure activées
│ ├─ Rotations régulières communities/passwords
│ └─ Limiter OID accessibles (vues)
│
│ ✗ À ÉVITER
│
│ ├─ SNMPv1/v2c depuis Internet ✗✗
│ ├─ Communities "public"/"private" ✗✗
│ ├─ Accès RW (écriture) non justifié ✗
│ ├─ Accès depuis 0.0.0.0/0 ✗
│ └─ Agent SNMP sur équipements critiques exposés ✗
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Vulnérabilités connues

```
┌──────────────────────────────────────────────────────────────┐
│ Risques de sécurité SNMP                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ SNIFFING (v1/v2c)                                            │
│ ├─ Community string en clair                                 │
│ ├─ Données non chiffrées                                     │
│ ├─ Attaquant capture traffic → Récupère community            │
│ └─ Mitigation : SNMPv3 ou VPN/IPsec                          │
│                                                              │
│ BRUTE-FORCE                                                  │
│ ├─ Tenter différentes community strings                      │
│ ├─ Facile si "public"/"private" par défaut                   │
│ └─ Mitigation : Communities complexes + fail2ban             │
│                                                              │
│ AMPLIFICATION DDoS                                           │
│ ├─ Attaquant spoof IP victime                                │
│ ├─ Envoie GET-BULK vers agents SNMP publics                  │
│ ├─ Réponses massives vers victime                            │
│ └─ Mitigation : Bloquer SNMP depuis Internet                 │
│                                                              │
│ INFORMATION DISCLOSURE                                       │
│ ├─ SNMP révèle beaucoup d'infos système                      │
│ ├─ Topologie réseau, versions logiciels, users...            │
│ └─ Mitigation : RO uniquement, limiter OID accessibles       │
│                                                              │
│ MODIFICATION NON AUTORISÉE (SET)                             │
│ ├─ Si RW community compromise                                │
│ ├─ Attaquant peut changer configuration                      │
│ └─ Mitigation : RW seulement si nécessaire, ACL strictes     │
│                                                              │
│ DEFAULT CREDENTIALS                                          │
│ ├─ Équipements avec "public"/"private" par défaut            │
│ ├─ Jamais changés par administrateurs                        │
│ └─ Mitigation : Audit et changement systématique             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Cas d'usage SNMP

```
┌──────────────────────────────────────────────────────────────┐
│ Applications typiques de SNMP                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ✓✓ MONITORING INFRASTRUCTURE                                 │
│   ├─ Surveillance bande passante réseau                      │
│   ├─ Disponibilité équipements (uptime)                      │
│   ├─ État interfaces (up/down)                               │
│   ├─ Utilisation CPU/Mémoire serveurs                        │
│   └─ Espace disque disponible                                │
│                                                              │
│ ✓✓ ALERTING                                                  │
│   ├─ Traps interface down → Email/SMS admin                  │
│   ├─ CPU > 90% → Alerte                                      │
│   ├─ Disque > 85% plein → Warning                            │
│   └─ Équipement redémarré → Notification                     │
│                                                              │
│ ✓✓ CAPACITY PLANNING                                         │
│   ├─ Collecte historique trafic réseau                       │
│   ├─ Tendances utilisation ressources                        │
│   ├─ Prévision besoins futurs                                │
│   └─ Optimisation infrastructure                             │
│                                                              │
│ ✓ INVENTAIRE                                                 │
│   ├─ Découverte automatique équipements                      │
│   ├─ Identification constructeur/modèle                      │
│   ├─ Versions firmware/OS                                    │
│   └─ Configuration matériel                                  │
│                                                              │
│ ✓ CONFIGURATION                                              │
│   ├─ Modification paramètres à distance (SET)                │
│   ├─ Moins courant (risque sécurité)                         │
│   └─ Préférer SSH/API dédiées                                │
│                                                              │
│ ✓ FACTURATION (ISP)                                          │
│   ├─ Mesure consommation bande passante clients              │
│   ├─ Facturation au volume                                   │
│   └─ Détection dépassements quotas                           │
│                                                              │
│ ✓ TROUBLESHOOTING                                            │
│   ├─ Diagnostic problèmes réseau                             │
│   ├─ Identification goulets d'étranglement                   │
│   └─ Corrélation événements                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

🔑 **SNMP = protocole standard supervision/gestion équipements réseau**

🔑 **Architecture Manager/Agent : Manager interroge, Agent répond**

🔑 **MIB = base de données hiérarchique, OID = identifiants uniques**

🔑 **Opérations : GET (lire), SET (écrire), TRAP (alerte)**

🔑 **SNMPv1/v2c = non sécurisés (community en clair)**

🔑 **SNMPv3 = sécurisé (authentification + chiffrement) - RECOMMANDÉ**

🔑 **UDP port 161 (requêtes), UDP port 162 (traps)**

🔑 **Polling = interrogation périodique, Trap = notification événement**

🔑 **Community "public"/"private" à CHANGER immédiatement**

🔑 **SNMP révèle beaucoup d'infos → Sécuriser et restreindre accès**

---

## Ce que nous avons appris

Dans cette section, nous avons exploré :

- ✅ Le rôle de SNMP dans la supervision réseau
- ✅ L'architecture Manager/Agent et le concept de MIB
- ✅ La hiérarchie des OID et leur notation
- ✅ Les opérations SNMP (GET, SET, TRAP, INFORM)
- ✅ Les trois versions (v1, v2c, v3) et leurs différences de sécurité
- ✅ L'utilisation pratique avec net-snmp et les outils CLI
- ✅ Les logiciels de supervision (Nagios, Zabbix, PRTG)
- ✅ Les MIB standards (système, interfaces, ressources)
- ✅ La configuration d'agents SNMP (Linux, Windows, Cisco)
- ✅ Les bonnes pratiques de sécurité et vulnérabilités

## Conclusion

SNMP est le protocole de supervision réseau le plus utilisé au monde depuis plus de 30 ans. Malgré ses limitations (notamment de sécurité dans les versions v1/v2c), il reste incontournable pour monitorer et gérer des infrastructures réseau de toute taille.

**Pour les déploiements modernes** :
- **Privilégiez SNMPv3** avec authentification et chiffrement (authPriv)
- Si v2c nécessaire, utilisez-le **uniquement sur réseaux de confiance**
- **Changez les communities par défaut** ("public"/"private")
- **Restreignez l'accès** avec ACL et firewall
- **Limitez aux permissions minimales** (lecture seule préférable)

SNMP n'est pas parfait, et de nouvelles alternatives émergent (Netconf, Restconf, gRPC), mais sa **simplicité, ubiquité et support universel** garantissent qu'il restera pertinent encore longtemps.

Pour tout administrateur réseau ou système, **maîtriser SNMP est essentiel** : c'est votre fenêtre sur la santé et les performances de votre infrastructure.

---

**Fin du module SNMP - Félicitations pour avoir complété ce tutoriel TCP/IP complet !** 🎉

⏭️ [NTP : synchronisation temporelle](/05-couche-application/09-ntp.md)
