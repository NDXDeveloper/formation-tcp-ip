🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 1.3 Le modèle OSI : les 7 couches expliquées

## Introduction

Dans la section précédente, nous avons découvert que les protocoles s'empilent en **couches**. Mais combien de couches faut-il ? Que fait chaque couche ? Comment s'organisent-elles ?

C'est exactement la question que s'est posée l'**ISO (International Organization for Standardization)** dans les années 1970. Leur réponse : le **modèle OSI** (Open Systems Interconnection), un cadre conceptuel qui divise la communication réseau en **7 couches distinctes**.

**Important** : Le modèle OSI n'est pas un protocole en lui-même, c'est un **modèle de référence**, une sorte de "plan d'architecte" qui aide à comprendre et à organiser les protocoles réseau.

---

## Pourquoi créer un modèle en couches ?

### Le problème initial

Imaginez que vous deviez construire un système de communication complet d'un seul bloc. Vous devriez gérer simultanément :
- Les signaux électriques sur le câble
- Les adresses des machines
- La fiabilité de la transmission
- Le formatage des données
- La sécurité
- L'interface utilisateur
- Et bien plus encore...

Ce serait **extrêmement complexe** et **impossible à maintenir** !

### La solution : la modularité

**Analogie : Construire une maison**

Vous ne construisez pas une maison d'un seul coup. Vous procédez par étapes :
1. Les **fondations**
2. La **structure** (murs, planchers)
3. La **toiture**
4. L'**électricité et plomberie**
5. L'**isolation**
6. Les **finitions**
7. La **décoration**

Chaque étape a ses spécialistes, ses outils, ses règles. Un électricien n'a pas besoin de savoir comment couler des fondations. Un décorateur n'a pas besoin de comprendre le système électrique.

**C'est exactement le principe du modèle OSI !**

### Les avantages de la modélisation en couches

✅ **Séparation des préoccupations** : Chaque couche a une responsabilité claire

✅ **Indépendance** : On peut modifier une couche sans affecter les autres

✅ **Interopérabilité** : Différentes technologies peuvent coexister à chaque couche

✅ **Facilité de dépannage** : On peut isoler les problèmes par couche

✅ **Innovation** : On peut améliorer une couche sans tout reconstruire

---

## Vue d'ensemble des 7 couches

Le modèle OSI est numéroté de **bas en haut** (de 1 à 7) :

```
┌─────────────────────────────────────────┐
│  7. Application                         │  ← L'utilisateur et les programmes
├─────────────────────────────────────────┤
│  6. Présentation                        │  ← Formatage et chiffrement
├─────────────────────────────────────────┤
│  5. Session                             │  ← Gestion des sessions
├─────────────────────────────────────────┤
│  4. Transport                           │  ← Livraison fiable
├─────────────────────────────────────────┤
│  3. Réseau                              │  ← Routage et adressage
├─────────────────────────────────────────┤
│  2. Liaison de données                  │  ← Transfert sur le lien local
├─────────────────────────────────────────┤
│  1. Physique                            │  ← Transmission des bits
└─────────────────────────────────────────┘
```

**Mnémotechnique** : "**P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way"
- **P**hysique
- **D**onnées (liaison de)
- **N**etwork (Réseau)
- **T**ransport
- **S**ession
- **P**résentation
- **A**pplication

---

## Couche 1 : Physique

### Rôle

La couche physique s'occupe de la **transmission brute des bits** sur un support physique (câble, fibre optique, ondes radio).

**Analogie** : C'est la route elle-même. Pas les voitures, pas les règles de circulation, juste l'asphalte, les ponts et les tunnels.

### Responsabilités

- 📡 **Type de support** : Câble cuivre, fibre optique, Wi-Fi, Bluetooth...
- ⚡ **Caractéristiques électriques** : Voltage, fréquence, modulation
- 🔌 **Connecteurs physiques** : RJ-45 (Ethernet), USB, fibre LC/SC...
- 📊 **Débit binaire** : Combien de bits par seconde peuvent passer ?
- 🎚️ **Topologie** : Bus, étoile, anneau, maillage...

### Exemples concrets

- Un câble Ethernet Cat 6
- Une puce Wi-Fi qui envoie des ondes radio
- La fibre optique qui transporte des impulsions lumineuses
- Le Bluetooth de votre casque

### Unité de données

**Le bit** (0 ou 1)

### Ce que cette couche ne fait PAS

- ❌ Elle ne comprend pas ce que signifient les bits
- ❌ Elle ne connaît pas les adresses
- ❌ Elle ne détecte pas les erreurs (enfin, très peu)

---

## Couche 2 : Liaison de données (Data Link)

### Rôle

La couche liaison organise les bits en **trames** et gère la communication entre deux équipements **directement connectés** sur le même réseau local.

**Analogie** : C'est le code de la route entre deux intersections. Elle définit comment les voitures se comportent sur un tronçon de route donné, comment elles ne se rentrent pas dedans, et comment on sait quelle voiture va où.

### Responsabilités

- 🏷️ **Adressage local** : Adresses MAC (Media Access Control)
- 📦 **Tramage** : Organisation des bits en trames avec début et fin
- 🚦 **Contrôle d'accès au média** : Qui peut transmettre et quand ?
- ✅ **Détection d'erreurs** : CRC (Cyclic Redundancy Check)
- 🔄 **Contrôle de flux local** : Ne pas submerger le récepteur

### Sous-couches

La couche 2 est souvent divisée en deux sous-couches :

**LLC (Logical Link Control)** :
- Indépendante du média
- Interface avec la couche réseau

**MAC (Media Access Control)** :
- Dépendante du média (Ethernet, Wi-Fi...)
- Gère l'accès physique au support

### Exemples concrets

**Ethernet** :
- Adresses MAC : `00:1A:2B:3C:4D:5E`
- Évite les collisions (plusieurs machines qui transmettent en même temps)
- Détecte les erreurs avec CRC

**Wi-Fi (802.11)** :
- Utilise aussi des adresses MAC
- Gère le partage des ondes radio (CSMA/CA)
- Acquittements des trames

**Switches** (commutateurs) :
- Opèrent à cette couche
- Utilisent les adresses MAC pour acheminer les trames

### Unité de données

**La trame (frame)**

```
┌────────┬─────────┬──────────┬──────┬─────┐
│ Début  │ Dest.   │ Source   │ Data │ CRC │
│ trame  │ MAC     │ MAC      │      │     │
└────────┴─────────┴──────────┴──────┴─────┘
```

### Portée

Cette couche fonctionne **uniquement sur le réseau local** (LAN). Elle ne peut pas router entre différents réseaux.

---

## Couche 3 : Réseau (Network)

### Rôle

La couche réseau gère le **routage** des données à travers **plusieurs réseaux** pour atteindre la destination finale, même si elle est à l'autre bout du monde.

**Analogie** : C'est le GPS et le système d'autoroutes. Il trouve le meilleur chemin entre votre ville et une ville lointaine, en passant par plusieurs autoroutes et villes intermédiaires.

### Responsabilités

- 🌍 **Adressage logique global** : Adresses IP (IPv4, IPv6)
- 🗺️ **Routage** : Déterminer le meilleur chemin vers la destination
- 📦 **Fragmentation et réassemblage** : Découper les gros paquets si nécessaire
- 🚛 **Acheminement inter-réseaux** : Faire voyager les données à travers Internet

### Exemples concrets

**IP (Internet Protocol)** :
- IPv4 : `192.168.1.10`
- IPv6 : `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

**Routeurs** :
- Opèrent à cette couche
- Décident où envoyer les paquets
- Maintiennent des tables de routage

**ICMP** :
- Protocole de messages de contrôle
- Utilisé par `ping` et `traceroute`

### Unité de données

**Le paquet (packet)**

```
┌──────────────────────────────────────────┐
│ En-tête IP                               │
│  ├─ Adresse IP source                    │
│  ├─ Adresse IP destination               │
│  ├─ TTL (Time To Live)                   │
│  └─ Protocole (TCP, UDP...)              │
├──────────────────────────────────────────┤
│ Données                                  │
└──────────────────────────────────────────┘
```

### Pourquoi c'est important

Sans cette couche, vous ne pourriez communiquer qu'avec les machines **directement** connectées à votre réseau local. Pas d'Internet !

---

## Couche 4 : Transport

### Rôle

La couche transport assure la **livraison de bout en bout** entre deux applications, avec ou sans garantie de fiabilité selon le protocole utilisé.

**Analogie** : C'est le service de livraison. La couche réseau (couche 3) vous a amené à la bonne ville et au bon quartier. La couche transport s'assure que le colis arrive à la bonne personne, dans la bonne boîte aux lettres, intact et dans l'ordre.

### Responsabilités

- 🎯 **Adressage des applications** : Ports (HTTP=80, HTTPS=443, SSH=22...)
- 📦 **Segmentation et réassemblage** : Découper les messages en segments
- 🔒 **Fiabilité (optionnelle)** : Garantir la livraison sans perte
- 🔢 **Contrôle de séquence** : Remettre dans l'ordre
- 🚰 **Contrôle de flux** : Ne pas submerger le récepteur
- 🚦 **Contrôle de congestion** : S'adapter aux conditions du réseau

### Les deux principaux protocoles

**TCP (Transmission Control Protocol)** :
- ✅ Fiable : garantit la livraison
- ✅ Ordre préservé
- ✅ Contrôle de flux et de congestion
- ❌ Plus lent (overhead)
- 🔌 Avec connexion

**Analogie** : Un courrier recommandé avec accusé de réception

**UDP (User Datagram Protocol)** :
- ⚡ Rapide et léger
- ❌ Pas de garantie de livraison
- ❌ Pas d'ordre garanti
- ✅ Faible latence
- 📡 Sans connexion

**Analogie** : Une carte postale. Plus rapide, mais peut se perdre.

### Exemples d'utilisation

**TCP** :
- Pages web (HTTP/HTTPS)
- Email (SMTP, IMAP)
- Transfert de fichiers (FTP, SFTP)
- SSH

**UDP** :
- Streaming vidéo/audio en direct
- Jeux en ligne (où la vitesse prime)
- DNS (requêtes courtes)
- VoIP (appels téléphoniques)

### Unité de données

**Le segment (TCP) ou datagramme (UDP)**

```
┌──────────────────────────────────────────┐
│ En-tête TCP/UDP                          │
│  ├─ Port source                          │
│  ├─ Port destination                     │
│  ├─ Numéro de séquence (TCP)             │
│  ├─ Numéro d'acquittement (TCP)          │
│  └─ Checksum                             │
├──────────────────────────────────────────┤
│ Données de l'application                 │
└──────────────────────────────────────────┘
```

---

## Couche 5 : Session

### Rôle

La couche session gère l'**établissement, le maintien et la terminaison** des sessions de communication entre applications.

**Analogie** : C'est comme la réceptionniste d'un hôtel qui gère votre séjour du check-in au check-out, en gardant trace de votre chambre et de vos services.

### Responsabilités

- 🔓 **Établissement de session** : Authentification, négociation de paramètres
- 💾 **Maintien de session** : Garder la session active, gérer les points de reprise
- 🔒 **Terminaison de session** : Fermeture propre
- 🔄 **Synchronisation** : Points de contrôle (checkpoints) pour reprendre après interruption
- 🎭 **Contrôle du dialogue** : Half-duplex ou full-duplex

### Exemples concrets

**NetBIOS** :
- Sessions entre applications Windows

**RPC (Remote Procedure Call)** :
- Appels de fonctions à distance

**SQL sessions** :
- Connexion à une base de données

**Web sessions** :
- Cookies de session HTTP
- Tokens d'authentification

### Note importante

Dans la pratique moderne avec TCP/IP, beaucoup de fonctionnalités de cette couche sont gérées par les couches 4 (Transport) et 7 (Application). C'est l'une des raisons pour lesquelles le modèle TCP/IP ne distingue pas explicitement cette couche.

---

## Couche 6 : Présentation

### Rôle

La couche présentation s'occupe de la **représentation** et du **format** des données. Elle traduit les données entre le format réseau et le format utilisable par l'application.

**Analogie** : C'est le traducteur et le diplomate. Il s'assure que deux personnes parlant des langues différentes peuvent se comprendre, et que les conventions culturelles sont respectées.

### Responsabilités

- 🔤 **Traduction de formats** : ASCII, EBCDIC, Unicode...
- 🗜️ **Compression** : Réduire la taille des données
- 🔐 **Chiffrement** : Sécuriser les données
- 📊 **Encodage** : Images (JPEG, PNG), vidéos (MP4, H.264), etc.

### Exemples concrets

**Encodage de caractères** :
- UTF-8, UTF-16
- ASCII, ISO-8859-1

**Formats d'images** :
- JPEG, PNG, GIF, WebP

**Compression** :
- ZIP, GZIP, Brotli

**Chiffrement** :
- SSL/TLS (transformation des données en transit)
- Chiffrement des fichiers

**Sérialisation** :
- JSON, XML
- Protocol Buffers, MessagePack

### Exemple pratique

Quand vous envoyez un emoji 😊 :
1. Votre application l'encode en UTF-8 : `0xF0 0x9F 0x98 0x8A`
2. Ces octets peuvent être compressés
3. Ils peuvent être chiffrés (HTTPS)
4. Ils sont transmis sur le réseau
5. À l'arrivée, le processus inverse se produit

### Note importante

Comme la couche session, beaucoup de fonctionnalités de présentation sont aujourd'hui intégrées directement dans les applications (couche 7) plutôt que dans une couche séparée.

---

## Couche 7 : Application

### Rôle

La couche application fournit les **services réseau directement aux applications** et aux utilisateurs finaux. C'est l'interface entre le réseau et les programmes que vous utilisez.

**Analogie** : C'est la vitrine du magasin, le guichet de la banque, l'interface utilisateur. C'est ce que vous voyez et avec quoi vous interagissez directement.

### Responsabilités

- 🌐 **Services aux utilisateurs** : Email, Web, transfert de fichiers...
- 🔍 **Identification des ressources** : URLs, noms de domaine
- 📧 **Protocoles spécifiques aux applications**

### Exemples de protocoles

**Web** :
- HTTP/HTTPS : Navigation web
- WebSocket : Communication bidirectionnelle en temps réel

**Email** :
- SMTP : Envoi d'emails
- IMAP/POP3 : Réception d'emails

**Fichiers** :
- FTP : Transfert de fichiers
- SFTP : Transfert sécurisé
- SMB : Partage de fichiers Windows

**Autres** :
- DNS : Résolution de noms de domaine
- DHCP : Attribution automatique d'adresses IP
- SSH : Connexion à distance sécurisée
- SNMP : Supervision réseau
- Telnet : Connexion à distance (non sécurisée)

### Unité de données

**Les données de l'application** (message, requête, réponse...)

### Exemples concrets que vous utilisez tous les jours

**Navigateur web** :
```
Vous tapez : https://www.example.com
↓
DNS : Convertit le nom en adresse IP
↓
HTTP/HTTPS : Demande la page web
↓
Le serveur renvoie le HTML
↓
Votre navigateur affiche la page
```

**Email** :
```
Vous cliquez sur "Envoyer"
↓
SMTP : Votre client envoie l'email à votre serveur
↓
SMTP : Votre serveur l'envoie au serveur du destinataire
↓
IMAP : Le destinataire récupère l'email
↓
Il le lit dans son client
```

---

## Le voyage complet des données à travers les 7 couches

Prenons un exemple concret : **Vous envoyez un email**.

### Côté émetteur (du haut vers le bas)

```
┌─────────────────────────────────────────────────────────┐
│ 7. APPLICATION                                          │
│    Vous écrivez votre email dans Outlook/Gmail          │
│    Protocole : SMTP                                     │
│    ↓ "Envoyer ce message"                               │
├─────────────────────────────────────────────────────────┤
│ 6. PRÉSENTATION                                         │
│    Encode le texte en UTF-8                             │
│    Peut compresser les pièces jointes                   │
│    ↓ Données formatées                                  │
├─────────────────────────────────────────────────────────┤
│ 5. SESSION                                              │
│    Établit une session SMTP avec le serveur             │
│    ↓ Session active                                     │
├─────────────────────────────────────────────────────────┤
│ 4. TRANSPORT                                            │
│    Découpe en segments TCP                              │
│    Ajoute port source et destination (25 pour SMTP)     │
│    ↓ Segments TCP                                       │
├─────────────────────────────────────────────────────────┤
│ 3. RÉSEAU                                               │
│    Ajoute adresses IP source et destination             │
│    Détermine le routage                                 │
│    ↓ Paquets IP                                         │
├─────────────────────────────────────────────────────────┤
│ 2. LIAISON                                              │
│    Ajoute adresses MAC                                  │
│    Crée des trames Ethernet                             │
│    ↓ Trames                                             │
├─────────────────────────────────────────────────────────┤
│ 1. PHYSIQUE                                             │
│    Convertit en signaux électriques/lumineux            │
│    Transmet sur le câble/fibre/Wi-Fi                    │
│    ↓ Bits (0 et 1)                                      │
└─────────────────────────────────────────────────────────┘
        │
        │ 📡 Transmission physique
        ↓
```

### Côté récepteur (du bas vers le haut)

```
        ↓ 📡 Réception physique
┌─────────────────────────────────────────────────────────┐
│ 1. PHYSIQUE                                             │
│    Reçoit les signaux, les convertit en bits            │
│    ↑                                                    │
├─────────────────────────────────────────────────────────┤
│ 2. LIAISON                                              │
│    Vérifie l'adresse MAC, retire l'en-tête Ethernet     │
│    Vérifie le CRC (pas d'erreurs)                       │
│    ↑                                                    │
├─────────────────────────────────────────────────────────┤
│ 3. RÉSEAU                                               │
│    Vérifie l'adresse IP de destination                  │
│    Retire l'en-tête IP                                  │
│    ↑                                                    │
├─────────────────────────────────────────────────────────┤
│ 4. TRANSPORT                                            │
│    Réassemble les segments TCP                          │
│    Vérifie l'intégrité, envoie les acquittements        │
│    ↑                                                    │
├─────────────────────────────────────────────────────────┤
│ 5. SESSION                                              │
│    Gère la session SMTP                                 │
│    ↑                                                    │
├─────────────────────────────────────────────────────────┤
│ 6. PRÉSENTATION                                         │
│    Décode l'UTF-8                                       │
│    Décompresse les pièces jointes                       │
│    ↑                                                    │
├─────────────────────────────────────────────────────────┤
│ 7. APPLICATION                                          │
│    Le serveur de mail traite le message SMTP            │
│    Le destinataire voit l'email dans sa boîte           │
└─────────────────────────────────────────────────────────┘
```

### L'encapsulation : des poupées russes

À chaque couche descendante, on **ajoute un en-tête** (header). C'est comme des enveloppes gigognes :

```
Couche 7 : [Données]
Couche 4 : [En-tête TCP | Données]
Couche 3 : [En-tête IP | En-tête TCP | Données]
Couche 2 : [En-tête Ethernet | En-tête IP | En-tête TCP | Données | CRC]
Couche 1 : Bits transmis physiquement
```

À la réception, on **retire** chaque en-tête couche par couche (décapsulation).

---

## Communication horizontale vs verticale

### Communication verticale

Les données **descendent** la pile OSI côté émetteur, et **remontent** la pile côté récepteur.

### Communication horizontale (logique)

Chaque couche communique **logiquement** avec sa couche homologue :
- La couche 4 émettrice "parle" à la couche 4 réceptrice
- La couche 3 émettrice "parle" à la couche 3 réceptrice
- Etc.

**Analogie** : Quand vous envoyez une lettre :
- Vous (Application) écrivez au destinataire (Application)
- Le facteur (Physique) parle au facteur distant (Physique)
- Mais vous, vous n'interagissez jamais directement avec le facteur du destinataire !

```
Émetteur                              Récepteur

[App] ←------- logiquement ------→ [App]
  ↓                                  ↑
[Pres] ←------ logiquement ------→ [Pres]
  ↓                                  ↑
[Sess] ←------ logiquement ------→ [Sess]
  ↓                                  ↑
[Trans] ←----- logiquement ------→ [Trans]
  ↓                                  ↑
[Réseau] ←---- logiquement ------→ [Réseau]
  ↓                                  ↑
[Liaison] ←--- logiquement ------→ [Liaison]
  ↓                                  ↑
[Physique] ====== physiquement ====== [Physique]
```

---

## Qui opère à quelle couche ?

### Équipements réseau

| Équipement | Couche(s) | Rôle |
|------------|-----------|------|
| **Câbles, fibres** | 1 (Physique) | Transport physique des signaux |
| **Hub** | 1 (Physique) | Répète les signaux (obsolète) |
| **Switch** | 2 (Liaison) | Commutation par adresse MAC |
| **Routeur** | 3 (Réseau) | Routage entre réseaux (IP) |
| **Pare-feu** | 3-4-7 | Filtrage de sécurité |
| **Load Balancer** | 4 ou 7 | Répartition de charge |

### Protocoles

| Protocole | Couche |
|-----------|--------|
| **Ethernet, Wi-Fi** | 2 |
| **IP, ICMP** | 3 |
| **TCP, UDP** | 4 |
| **HTTP, FTP, SMTP, DNS** | 7 |
| **SSL/TLS** | 5-6 (entre Transport et Application) |

---

## Forces et limites du modèle OSI

### Forces ✅

- 📚 **Excellent outil pédagogique** : Aide à comprendre les réseaux
- 🗂️ **Organisation claire** : Chaque couche a un rôle défini
- 🌍 **Standard de référence** : Vocabulaire commun dans l'industrie
- 🔧 **Dépannage facilité** : Permet d'isoler les problèmes par couche
- 🧩 **Modularité** : Changements indépendants par couche

### Limites ❌

- 📖 **Modèle théorique** : Peu de protocoles l'implémentent strictement
- 🐌 **Trop complexe** : 7 couches semblent excessives pour la pratique
- ⏰ **Créé dans les années 1970** : Ne reflète pas toutes les technologies modernes
- 🎭 **Couches 5 et 6 floues** : Leurs rôles se chevauchent souvent avec les couches 4 et 7
- 💻 **TCP/IP règne** : C'est le modèle en 4 couches qui a gagné dans la réalité

---

## OSI aujourd'hui : toujours pertinent ?

**Oui, comme modèle conceptuel !**

Même si personne n'implémente strictement les 7 couches OSI, le modèle reste **extrêmement utile** :

### 1. Pour l'apprentissage
C'est un excellent cadre pour comprendre comment fonctionnent les réseaux de manière structurée.

### 2. Pour la communication
Les professionnels disent "problème de couche 2" ou "il faut un pare-feu de couche 7" - tout le monde comprend.

### 3. Pour le dépannage
On peut systématiquement vérifier chaque couche :
```
Problème de connexion Internet ?

1. Le câble est branché ? (Couche 1)
2. Le switch voit-il la machine ? (Couche 2)
3. L'adresse IP est correcte ? (Couche 3)
4. Le port TCP fonctionne ? (Couche 4)
5. L'application répond-elle ? (Couche 7)
```

### 4. Pour la conception
Lors de la conception d'architectures, on réfléchit en termes de couches pour séparer les responsabilités.

---

## Comparaison avec le modèle TCP/IP (aperçu)

Le modèle OSI (7 couches) est théorique. Le modèle TCP/IP (4 couches) est ce qui est réellement implémenté dans Internet.

```
    OSI (7 couches)          TCP/IP (4 couches)

7.  Application          ┐
6.  Présentation         │── Application
5.  Session              ┘

4.  Transport            ──── Transport

3.  Réseau               ──── Internet

2.  Liaison              ┐
1.  Physique             ┘── Accès réseau
```

Nous explorerons le modèle TCP/IP en détail dans la section suivante.

---

## Conclusion

Le modèle OSI est une **boussole intellectuelle** qui nous aide à naviguer dans la complexité des réseaux. Même s'il n'est pas implémenté tel quel dans la réalité, ses principes fondamentaux demeurent :

- 🧩 **Modularité** : Diviser pour mieux régner
- 🔀 **Séparation des responsabilités** : Chaque couche son rôle
- 🔓 **Interfaces standardisées** : Communication claire entre couches
- 🌐 **Interopérabilité** : Des systèmes différents peuvent communiquer

Comprendre OSI, c'est comme comprendre l'anatomie avant d'étudier la médecine. C'est la base qui vous permettra de maîtriser TCP/IP, les protocoles modernes, et de devenir un expert réseau.

---

**À retenir** :
- 📚 OSI = modèle de **référence** en 7 couches, pas une implémentation
- 🔢 Numérotation de **bas en haut** : 1 (Physique) → 7 (Application)
- 📦 **Encapsulation** : Chaque couche ajoute son en-tête
- 🔧 **Utile pour** : Comprendre, communiquer, dépanner
- 🎯 Chaque couche a une **responsabilité claire** et distincte
- 🌍 Les couches 1-4 gèrent le **transport**, les couches 5-7 gèrent les **applications**

**Prochaine étape** : Découvrons le modèle TCP/IP, celui qui fait vraiment fonctionner Internet ! →

⏭️ [Le modèle TCP/IP : architecture en 4 couches](/01-introduction/04-modele-tcp-ip.md)
