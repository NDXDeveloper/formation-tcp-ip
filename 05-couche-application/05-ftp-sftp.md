🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.5 FTP et SFTP

## Introduction

Le transfert de fichiers est l'une des fonctions les plus fondamentales d'un réseau. Que ce soit pour partager des documents, déployer un site web, sauvegarder des données ou distribuer des logiciels, nous avons besoin de protocoles fiables pour déplacer des fichiers d'un ordinateur à un autre.

**FTP (File Transfer Protocol)** et **SFTP (SSH File Transfer Protocol)** sont deux protocoles majeurs dédiés au transfert de fichiers :
- **FTP** : Le protocole historique, créé en 1971, simple mais non sécurisé
- **SFTP** : La version moderne et sécurisée, basée sur SSH

Dans cette section, nous allons explorer ces deux protocoles en profondeur, comprendre leurs différences, leurs forces et leurs faiblesses, et savoir quand utiliser l'un plutôt que l'autre.

## FTP (File Transfer Protocol)

### Présentation et historique

```
┌──────────────────────────────────────────────────────────┐
│ FTP - File Transfer Protocol                             │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ Création : 1971 (l'un des plus anciens protocoles)       │
│ RFC : RFC 959 (1985, toujours la référence)              │
│ Port : 21 (contrôle) + 20 (données en mode actif)        │
│ Transport : TCP                                          │
│ Sécurité : Aucune par défaut (texte clair)               │
│                                                          │
│ Usage : Transfert de fichiers client ↔ serveur           │
│ Caractéristique unique : 2 connexions distinctes         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Pourquoi FTP a été créé ?**

Dans les années 1970, les réseaux étaient balbutiants et il fallait un moyen standardisé de transférer des fichiers entre ordinateurs de différents fabricants (IBM, DEC, etc.). FTP a été conçu pour être :
- Simple et efficace
- Indépendant du système d'exploitation
- Capable de gérer différents types de fichiers (texte, binaire)
- Robuste face aux pannes réseau

### Architecture : Deux connexions

FTP est **unique** parmi les protocoles applicatifs car il utilise **deux connexions TCP distinctes** :

```
┌──────────────────────────────────────────────────────────┐
│ Architecture FTP à deux canaux                           │
└──────────────────────────────────────────────────────────┘

Client FTP                           Serveur FTP
    │                                     │
    │                                     │
    │  ═══════════════════════════════    │
    │  CANAL CONTRÔLE (Control Channel)   │
    │  ═══════════════════════════════    │
    │                                     │
    │  • Port : 21                        │
    │  • Connexion persistante            │
    │  • Commandes : USER, PASS, LIST...  │
    │  • Réponses : codes 200, 500...     │
    │                                     │
    ├─────────────────────────────────────┤
    │                                     │
    │  ═══════════════════════════════    │
    │  CANAL DONNÉES (Data Channel)       │
    │  ═══════════════════════════════    │
    │                                     │
    │  • Port : 20 (actif) ou dynamique   │
    │  • Connexion temporaire             │
    │  • Transfert fichiers               │
    │  • Listing répertoires              │
    │  • Fermée après chaque transfert    │
    │                                     │
    └─────────────────────────────────────┘
```

**Canal de contrôle :**
```
Rôle : Envoyer commandes et recevoir réponses
Durée : Reste ouvert pendant toute la session
Protocole : Texte ASCII (lisible par l'homme)

Exemple de dialogue :
Client → Serveur : USER bob
Serveur → Client : 331 Password required for bob
Client → Serveur : PASS secret123
Serveur → Client : 230 User logged in, proceed
Client → Serveur : LIST
Serveur → Client : 150 Opening data connection
```

**Canal de données :**
```
Rôle : Transférer le contenu (fichiers, listings)
Durée : Ouvert uniquement pendant le transfert
Mode : Binaire ou ASCII

Exemple :
Client → Serveur : RETR document.pdf
Serveur ouvre canal données → Envoie le fichier
Fichier transféré → Canal fermé
Serveur → Client : 226 Transfer complete
```

### Modes de connexion : Actif vs Passif

L'une des particularités de FTP est qu'il existe deux modes de connexion pour le canal de données.

#### Mode Actif (PORT)

```
┌──────────────────────────────────────────────────────────┐
│ Mode FTP Actif (PORT) - Historique                       │
└──────────────────────────────────────────────────────────┘

Étape 1 : Client initie connexion contrôle
Client (port 50000) ──────────────→ Serveur (port 21)
                    Connexion TCP

Étape 2 : Client envoie commande PORT
Client → Serveur : "PORT 192,168,1,42,195,80"
                   (= IP 192.168.1.42, port 50000)
                   "Je t'écoute sur ce port pour les données"

Étape 3 : Client demande un fichier
Client → Serveur : "RETR fichier.txt"

Étape 4 : SERVEUR initie connexion données
Serveur (port 20) ──────────────→ Client (port 50000)
                    Connexion TCP inverse !

Étape 5 : Transfert de données
Serveur → Client : [Contenu du fichier]

Étape 6 : Fermeture connexion données
Serveur ──X────────────────────── Client
```

**Diagramme de séquence :**
```
Client (192.168.1.42)              Serveur (203.0.113.10)
       │                                    │
       │ SYN (port 50000 → 21)              │
       ├───────────────────────────────────>│
       │ SYN-ACK                            │
       │<───────────────────────────────────┤
       │ ACK - Connexion contrôle établie   │
       ├───────────────────────────────────>│
       │                                    │
       │ USER bob                           │
       ├───────────────────────────────────>│
       │ 331 Password required              │
       │<───────────────────────────────────┤
       │ PASS secret                        │
       ├───────────────────────────────────>│
       │ 230 Logged in                      │
       │<───────────────────────────────────┤
       │                                    │
       │ PORT 192,168,1,42,195,80           │
       ├───────────────────────────────────>│
       │ 200 PORT command OK                │
       │<───────────────────────────────────┤
       │                                    │
       │ RETR fichier.txt                   │
       ├───────────────────────────────────>│
       │ 150 Opening data connection        │
       │<───────────────────────────────────┤
       │                                    │
       │ SYN (port 20 → 50000) ← Inverse ! │
       │<───────────────────────────────────┤
       │ SYN-ACK                            │
       ├───────────────────────────────────>│
       │ ACK - Connexion données établie    │
       │<───────────────────────────────────┤
       │                                    │
       │ [Transfert du fichier...]          │
       │<═══════════════════════════════════┤
       │                                    │
       │ 226 Transfer complete              │
       │<───────────────────────────────────┤
       │ Connexion données fermée           │
       │                                    │
```

**Problème majeur du mode actif :**
```
┌──────────────────────────────────────────────────────────┐
│ Problème avec les pare-feu / NAT                         │
└──────────────────────────────────────────────────────────┘

Client derrière NAT/firewall :

Internet                 Pare-feu            Client
                            │                  │
Serveur FTP (20) ──X───────>│                  │
                   Bloqué ! │                  │
                            │                  │
                            ↓                  │
                     Connexion entrante        │
                     non sollicitée            │
                     = Bloquée par défaut      │

Le serveur ne peut pas initier la connexion vers le client !
→ Mode actif inutilisable dans 90% des cas modernes
```

#### Mode Passif (PASV)

Le mode passif résout le problème du mode actif :

```
┌──────────────────────────────────────────────────────────┐
│ Mode FTP Passif (PASV) - Moderne                         │
└──────────────────────────────────────────────────────────┘

Étape 1 : Client initie connexion contrôle
Client (port 50000) ──────────────→ Serveur (port 21)

Étape 2 : Client envoie commande PASV
Client → Serveur : "PASV"
                   "Je veux que TU écoutes, je me connecterai"

Étape 3 : Serveur ouvre un port et écoute
Serveur → Client : "227 Entering Passive Mode (203,0,113,10,195,80)"
                   (= IP 203.0.113.10, port 50000)

Étape 4 : CLIENT initie connexion données
Client (port 50001) ──────────────→ Serveur (port 50000)
                    Connexion sortante = OK firewall ✓

Étape 5 : Client demande fichier
Client → Serveur : "RETR fichier.txt"

Étape 6 : Transfert de données
Serveur → Client : [Contenu du fichier]

Étape 7 : Fermeture connexion données
Client ──X────────────────────── Serveur
```

**Diagramme de séquence :**
```
Client (192.168.1.42)              Serveur (203.0.113.10)
       │                                    │
       │ [Connexion contrôle établie]       │
       │ [Authentification OK]              │
       │                                    │
       │ PASV                               │
       ├───────────────────────────────────>│
       │                                    │
       │                    Serveur ouvre   │
       │                    port dynamique  │
       │                    (ex: 50000)     │
       │                                    │
       │ 227 Passive (203,0,113,10,195,80)  │
       │<───────────────────────────────────┤
       │                                    │
       │ SYN (port 50001 → 50000)           │
       ├───────────────────────────────────>│
       │ SYN-ACK                            │
       │<───────────────────────────────────┤
       │ ACK - Connexion données établie ✓  │
       ├───────────────────────────────────>│
       │                                    │
       │ RETR fichier.txt                   │
       ├───────────────────────────────────>│
       │ 150 Opening data connection        │
       │<───────────────────────────────────┤
       │                                    │
       │ [Transfert du fichier...]          │
       │<═══════════════════════════════════┤
       │                                    │
       │ 226 Transfer complete              │
       │<───────────────────────────────────┤
```

**Avantages du mode passif :**
```
✓ Compatible avec NAT
✓ Compatible avec les pare-feu
✓ Client initie toutes les connexions (sortantes)
✓ Mode par défaut de tous les clients modernes

C'est pour cela que le mode passif est devenu la norme !
```

### Commandes FTP principales

FTP utilise des commandes textuelles simples :

```
┌──────────────────────────────────────────────────────────┐
│ Commandes FTP essentielles                               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ AUTHENTIFICATION                                         │
│ ├─ USER <username>    : Nom d'utilisateur                │
│ ├─ PASS <password>    : Mot de passe                     │
│ └─ QUIT               : Fermer la connexion              │
│                                                          │
│ NAVIGATION                                               │
│ ├─ PWD                : Print Working Directory          │
│ ├─ CWD <dir>          : Change Working Directory         │
│ ├─ CDUP               : Remonter d'un niveau             │
│ └─ LIST [<path>]      : Lister fichiers/dossiers         │
│                                                          │
│ TRANSFERT FICHIERS                                       │
│ ├─ RETR <filename>    : Télécharger un fichier           │
│ ├─ STOR <filename>    : Uploader un fichier              │
│ ├─ DELE <filename>    : Supprimer un fichier             │
│ └─ RNFR/RNTO          : Renommer un fichier              │
│                                                          │
│ GESTION RÉPERTOIRES                                      │
│ ├─ MKD <dirname>      : Créer répertoire                 │
│ └─ RMD <dirname>      : Supprimer répertoire             │
│                                                          │
│ CONFIGURATION                                            │
│ ├─ TYPE <A|I>         : ASCII ou Image (binaire)         │
│ ├─ PORT <address>     : Mode actif                       │
│ ├─ PASV               : Mode passif                      │
│ └─ SYST               : Type de système serveur          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Codes de réponse FTP

Le serveur répond avec des codes numériques à 3 chiffres :

```
┌──────────────────────────────────────────────────────────┐
│ Codes de réponse FTP (Format : XYZ)                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ X = Catégorie :                                          │
│ ├─ 1xx : Positive Preliminary (en attente)               │
│ ├─ 2xx : Positive Completion (succès)                    │
│ ├─ 3xx : Positive Intermediate (info supplémentaire)     │
│ ├─ 4xx : Transient Negative (erreur temporaire)          │
│ └─ 5xx : Permanent Negative (erreur permanente)          │
│                                                          │
│ Y = Sous-catégorie :                                     │
│ ├─ x0x : Syntaxe                                         │
│ ├─ x1x : Information                                     │
│ ├─ x2x : Connexion                                       │
│ └─ x3x : Authentification                                │
│                                                          │
│ CODES COURANTS :                                         │
│                                                          │
│ 125 : Data connection open, transfer starting            │
│ 150 : File status okay, about to open data connection    │
│ 200 : Command okay                                       │
│ 220 : Service ready for new user                         │
│ 221 : Service closing control connection                 │
│ 226 : Closing data connection, transfer complete         │
│ 230 : User logged in, proceed                            │
│ 331 : Username okay, need password                       │
│ 350 : Requested file action pending                      │
│ 421 : Service not available, closing connection          │
│ 425 : Can't open data connection                         │
│ 426 : Connection closed, transfer aborted                │
│ 450 : File unavailable (busy)                            │
│ 500 : Syntax error, command unrecognized                 │
│ 501 : Syntax error in parameters                         │
│ 502 : Command not implemented                            │
│ 503 : Bad sequence of commands                           │
│ 530 : Not logged in                                      │
│ 550 : File unavailable (not found, no permission)        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Session FTP complète exemple

```
┌──────────────────────────────────────────────────────────┐
│ Exemple de session FTP complète                          │
└──────────────────────────────────────────────────────────┘

# Connexion
Client : [Connexion TCP au port 21]
Serveur : 220 Welcome to FTP Server

# Authentification
Client : USER bob
Serveur : 331 Password required for bob

Client : PASS mypassword
Serveur : 230 User bob logged in

# Informations système
Client : SYST
Serveur : 215 UNIX Type: L8

Client : PWD
Serveur : 257 "/home/bob" is current directory

# Lister fichiers
Client : PASV
Serveur : 227 Entering Passive Mode (203,0,113,10,195,80)

Client : LIST
Serveur : 150 Opening ASCII mode data connection for file list
Serveur : [Sur canal données]
          drwxr-xr-x   2 bob  users     4096 Dec 06 10:00 documents
          -rw-r--r--   1 bob  users    12345 Dec 05 15:30 rapport.pdf
          -rw-r--r--   1 bob  users     5678 Dec 04 09:15 image.jpg
Serveur : 226 Transfer complete

# Télécharger un fichier
Client : TYPE I
Serveur : 200 Type set to I (binary)

Client : PASV
Serveur : 227 Entering Passive Mode (203,0,113,10,195,81)

Client : RETR rapport.pdf
Serveur : 150 Opening BINARY mode data connection for rapport.pdf (12345 bytes)
Serveur : [Transfert du fichier sur canal données...]
Serveur : 226 Transfer complete (12345 bytes in 0.5 seconds)

# Uploader un fichier
Client : PASV
Serveur : 227 Entering Passive Mode (203,0,113,10,195,82)

Client : STOR nouveau.txt
Serveur : 150 Opening BINARY mode data connection for nouveau.txt
Client : [Envoie le fichier sur canal données...]
Serveur : 226 Transfer complete (8901 bytes received)

# Créer répertoire
Client : MKD archives
Serveur : 257 "archives" directory created

# Changer de répertoire
Client : CWD archives
Serveur : 250 CWD command successful

# Déconnexion
Client : QUIT
Serveur : 221 Goodbye
```

### Modes de transfert : ASCII vs Binaire

```
┌──────────────────────────────────────────────────────────┐
│ Modes de transfert FTP                                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ MODE ASCII (TYPE A)                                      │
│ ├─ Usage : Fichiers texte (.txt, .html, .csv...)         │
│ ├─ Conversion : Fins de lignes adaptées                  │
│ │   • Unix : LF (\n)                                     │
│ │   • Windows : CRLF (\r\n)                              │
│ │   • Mac ancien : CR (\r)                               │
│ ├─ Exemple : Transfert .txt Windows → Unix               │
│ │   "Bonjour\r\nMonde\r\n" → "Bonjour\nMonde\n"          │
│ └─ ⚠️ Ne jamais utiliser pour binaires !                 │
│                                                          │
│ MODE BINAIRE/IMAGE (TYPE I)                              │
│ ├─ Usage : Tous fichiers binaires                        │
│ ├─ Transfert : Byte-par-byte, aucune conversion          │
│ ├─ Exemples : .pdf, .jpg, .exe, .zip, .mp3...            │
│ └─ ✓ Mode par défaut recommandé                          │
│                                                          │
│ RÈGLE D'OR :                                             │
│ En cas de doute → MODE BINAIRE                           │
│ Les fichiers texte fonctionnent en binaire,              │
│ mais les binaires sont corrompus en ASCII !              │
│                                                          │
└──────────────────────────────────────────────────────────┘

Exemple d'erreur classique :
Fichier image.jpg transféré en mode ASCII
→ Octets 0x0A (LF) convertis en 0x0D 0x0A
→ Fichier corrompu
→ Image impossible à ouvrir ✗
```

### Problèmes de sécurité de FTP

```
┌──────────────────────────────────────────────────────────┐
│ Vulnérabilités critiques de FTP                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ 1. AUTHENTIFICATION EN CLAIR                             │
│    ├─ USER bob                                           │
│    ├─ PASS secret123  ← Visible dans Wireshark !         │
│    └─ ✗ Interception triviale (sniffing)                 │
│                                                          │
│ 2. DONNÉES EN CLAIR                                      │
│    ├─ Fichiers transférés sans chiffrement               │
│    ├─ Contenu lisible par quiconque écoute               │
│    └─ ✗ Confidentialité : ZÉRO                           │
│                                                          │
│ 3. ATTAQUES MAN-IN-THE-MIDDLE                            │
│    ├─ Pas d'authentification du serveur                  │
│    ├─ Attaquant peut se faire passer pour serveur        │
│    └─ ✗ Impossible de vérifier identité                  │
│                                                          │
│ 4. ATTAQUES "BOUNCE"                                     │
│    ├─ Client peut demander transfert serveur → cible     │
│    ├─ Serveur devient proxy non voulu                    │
│    └─ ✗ Scan de ports, contournement firewall            │
│                                                          │
│ 5. PORTS DYNAMIQUES                                      │
│    ├─ Mode passif utilise ports élevés aléatoires        │
│    ├─ Difficile à sécuriser avec firewall                │
│    └─ ⚠️ Large plage de ports à ouvrir                   │
│                                                          │
│ CONCLUSION :                                             │
│ FTP classique est TOTALEMENT INADAPTÉ                    │
│ pour tout usage nécessitant sécurité !                   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Solutions "sécurisées" pour FTP

**FTPS (FTP over SSL/TLS)**
```
┌──────────────────────────────────────────────────────────┐
│ FTPS = FTP + TLS/SSL                                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ Principe : Envelopper FTP dans TLS                       │
│                                                          │
│ Deux variantes :                                         │
│                                                          │
│ 1. FTPS Explicite (FTPES)                                │
│    ├─ Port : 21 (comme FTP)                              │
│    ├─ Commande : AUTH TLS                                │
│    ├─ Client demande chiffrement                         │
│    └─ ✓ Compatible avec serveurs FTP standards           │
│                                                          │
│ 2. FTPS Implicite                                        │
│    ├─ Port : 990 (dédié)                                 │
│    ├─ TLS dès la connexion                               │
│    ├─ Pas de négociation                                 │
│    └─ ⚠️ Moins flexible                                  │
│                                                          │
│ Avantages :                                              │
│ ✓ Chiffrement authentification et données                │
│ ✓ Certificats serveur (authentification)                 │
│                                                          │
│ Inconvénients :                                          │
│ ✗ Toujours deux connexions                               │
│ ✗ Problèmes avec NAT/firewall (pire que FTP)             │
│ ✗ Configuration complexe                                 │
│ ✗ Support inégal selon clients                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Problème avec FTPS et les pare-feu :**
```
FTPS + NAT = Cauchemar

Le canal contrôle est chiffré
→ Le pare-feu/NAT ne peut pas lire les commandes PORT/PASV
→ Ne peut pas anticiper les connexions de données
→ Échecs de connexion fréquents

Solutions partielles :
├─ ALG (Application Layer Gateway) dans firewall
├─ Mais support limité et buggy
└─ Beaucoup d'administrateurs abandonnent FTPS
```

## SFTP (SSH File Transfer Protocol)

### Présentation

```
┌──────────────────────────────────────────────────────────┐
│ SFTP - SSH File Transfer Protocol                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ Création : ~1990s (avec SSH)                             │
│ RFC : RFC 4251-4254 (SSH), draft SFTP                    │
│ Port : 22 (SSH)                                          │
│ Transport : TCP sur SSH                                  │
│ Sécurité : Totale (chiffrement + authentification)       │
│                                                          │
│ Usage : Transfert de fichiers sécurisé                   │
│ Caractéristique : UNE SEULE connexion (tunnel SSH)       │
│                                                          │
│ ⚠️ ATTENTION : SFTP ≠ FTPS                               │
│    Ce sont des protocoles complètement différents !      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**SFTP vs FTP vs FTPS - Clarification :**
```
┌──────────────────────────────────────────────────────────┐
│ Comparaison : FTP, FTPS, SFTP                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ FTP (File Transfer Protocol)                             │
│ ├─ Protocole original (1971)                             │
│ ├─ Deux connexions (contrôle + données)                  │
│ ├─ Ports : 21, 20                                        │
│ ├─ Sécurité : Aucune                                     │
│ └─ Usage moderne : Déconseillé                           │
│                                                          │
│ FTPS (FTP over SSL/TLS)                                  │
│ ├─ FTP + couche TLS                                      │
│ ├─ Deux connexions (les deux chiffrées)                  │
│ ├─ Ports : 21/990                                        │
│ ├─ Sécurité : Bonne                                      │
│ └─ Usage : Compliqué (NAT/firewall)                      │
│                                                          │
│ SFTP (SSH File Transfer Protocol)                        │
│ ├─ Protocole distinct, basé sur SSH                      │
│ ├─ Une seule connexion (tunnel SSH)                      │
│ ├─ Port : 22                                             │
│ ├─ Sécurité : Excellente                                 │
│ └─ Usage : Standard moderne recommandé ✓                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Architecture SFTP

```
┌──────────────────────────────────────────────────────────┐
│ Architecture SFTP (une seule connexion)                  │
└──────────────────────────────────────────────────────────┘

Client SFTP                          Serveur SSH/SFTP
    │                                       │
    │ ═══════════════════════════════════   │
    │    TUNNEL SSH CHIFFRÉ (port 22)       │
    │ ═══════════════════════════════════   │
    │                                       │
    │  ┌──────────────────────────────┐     │
    │  │  • Authentification SSH      │     │
    │  │    (password ou clé publique)│     │
    │  ├──────────────────────────────┤     │
    │  │  • Commandes SFTP            │     │
    │  │    (ls, get, put, rm...)     │     │
    │  ├──────────────────────────────┤     │
    │  │  • Transfert de données      │     │
    │  │    (dans le même tunnel)     │     │
    │  └──────────────────────────────┘     │
    │                                       │
    │  Tout est multiplexé dans             │
    │  UNE SEULE connexion TCP !            │
    │                                       │
    └───────────────────────────────────────┘

Avantages :
✓ Un seul port (22) à ouvrir
✓ Compatible NAT/firewall naturellement
✓ Tout est chiffré (commandes + données)
✓ Authentification forte (clés SSH)
✓ Plus simple à configurer que FTPS
```

### Processus de connexion SFTP

```
┌──────────────────────────────────────────────────────────┐
│ Établissement d'une session SFTP                         │
└──────────────────────────────────────────────────────────┘

Étape 1 : Connexion TCP
Client ──────────────────→ Serveur (port 22)
        SYN / SYN-ACK / ACK

Étape 2 : Handshake SSH
├─ Échange versions SSH
│  Client → Serveur : "SSH-2.0-OpenSSH_8.9"
│  Serveur → Client : "SSH-2.0-OpenSSH_8.9"
│
├─ Échange clés de chiffrement (Diffie-Hellman)
│  → Génération clé de session
│
└─ Le canal est maintenant CHIFFRÉ ✓

Étape 3 : Authentification
Option A : Par mot de passe
├─ Client envoie username + password (chiffrés)
└─ Serveur valide

Option B : Par clé publique (recommandé)
├─ Client envoie username + clé publique
├─ Serveur vérifie dans authorized_keys
├─ Serveur envoie challenge
├─ Client signe avec clé privée
└─ Serveur valide signature ✓

Étape 4 : Subsystem SFTP
Client demande : "subsystem sftp"
Serveur lance : /usr/lib/openssh/sftp-server
→ Session SFTP active

Étape 5 : Opérations SFTP
Client peut maintenant :
├─ Lister fichiers
├─ Télécharger/uploader
├─ Créer/supprimer
└─ Gérer permissions
```

### Commandes SFTP

SFTP a sa propre syntaxe de commandes (différente de FTP) :

```
┌──────────────────────────────────────────────────────────┐
│ Commandes SFTP principales                               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ NAVIGATION SERVEUR (distant)                             │
│ ├─ pwd              : Répertoire courant distant         │
│ ├─ cd <dir>         : Changer répertoire distant         │
│ ├─ ls [<path>]      : Lister fichiers distants           │
│ └─ ls -la           : Lister avec détails/cachés         │
│                                                          │
│ NAVIGATION CLIENT (local)                                │
│ ├─ lpwd             : Répertoire courant local           │
│ ├─ lcd <dir>        : Changer répertoire local           │
│ └─ lls [<path>]     : Lister fichiers locaux             │
│                                                          │
│ TRANSFERT FICHIERS                                       │
│ ├─ get <remote>     : Télécharger fichier                │
│ ├─ get -r <dir>     : Télécharger répertoire (récursif)  │
│ ├─ put <local>      : Uploader fichier                   │
│ ├─ put -r <dir>     : Uploader répertoire                │
│ └─ reget <remote>   : Reprendre téléchargement           │
│                                                          │
│ GESTION FICHIERS/DOSSIERS                                │
│ ├─ mkdir <dir>      : Créer répertoire                   │
│ ├─ rmdir <dir>      : Supprimer répertoire (vide)        │
│ ├─ rm <file>        : Supprimer fichier                  │
│ ├─ rename <old> <new> : Renommer                         │
│ ├─ ln -s <src> <dst> : Créer lien symbolique             │
│ └─ chmod <mode> <file> : Modifier permissions            │
│                                                          │
│ AUTRES                                                   │
│ ├─ df               : Espace disque                      │
│ ├─ ! <command>      : Exécuter commande locale           │
│ ├─ help / ?         : Aide                               │
│ ├─ version          : Version protocole                  │
│ └─ exit / quit / bye : Quitter                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Session SFTP interactive exemple

```
┌──────────────────────────────────────────────────────────┐
│ Exemple de session SFTP complète                         │
└──────────────────────────────────────────────────────────┘

$ sftp bob@example.com
bob@example.com's password: ********
Connected to example.com.

sftp> pwd
Remote working directory: /home/bob

sftp> ls -la
drwxr-xr-x    3 bob      users        4096 Dec  6 10:00 .
drwxr-xr-x   10 root     root         4096 Nov 20 08:00 ..
-rw-------    1 bob      users         220 Nov 20 08:00 .bash_logout
drwxr-xr-x    2 bob      users        4096 Dec  5 15:30 documents
-rw-r--r--    1 bob      users       12345 Dec  5 15:30 rapport.pdf
-rw-r--r--    1 bob      users        5678 Dec  4 09:15 image.jpg

sftp> cd documents
sftp> pwd
Remote working directory: /home/bob/documents

sftp> lpwd
Local working directory: /home/alice/Downloads

sftp> get rapport.pdf
Fetching /home/bob/documents/rapport.pdf to rapport.pdf
/home/bob/documents/rapport.pdf     100%   12KB  12.0KB/s   00:01

sftp> lcd ../Desktop
sftp> lpwd
Local working directory: /home/alice/Desktop

sftp> put presentation.pptx
Uploading presentation.pptx to /home/bob/documents/presentation.pptx
presentation.pptx                   100% 2048KB   2.0MB/s   00:01

sftp> mkdir archives
sftp> cd archives

sftp> put -r photos/
Uploading photos/ to /home/bob/documents/archives/photos
photos/IMG001.jpg                   100% 1024KB   1.0MB/s   00:01
photos/IMG002.jpg                   100%  987KB 987.0KB/s   00:01
photos/IMG003.jpg                   100% 1156KB   1.1MB/s   00:01

sftp> chmod 644 rapport.pdf
Changing mode on /home/bob/documents/rapport.pdf

sftp> df -h
    Size     Used    Avail   (root)    %Capacity
  48.0GB   28.5GB   17.0GB   19.5GB          59%

sftp> bye
```

### Authentification par clé SSH

L'authentification par clé est la méthode **recommandée** pour SFTP :

```
┌──────────────────────────────────────────────────────────┐
│ Configuration authentification par clé SSH               │
└──────────────────────────────────────────────────────────┘

Étape 1 : Générer paire de clés (sur le CLIENT)
──────────────────────────────────────────────

$ ssh-keygen -t ed25519 -C "bob@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/bob/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase): ********
Enter same passphrase again: ********

Créé :
├─ /home/bob/.ssh/id_ed25519     (clé privée, SECRÈTE !)
└─ /home/bob/.ssh/id_ed25519.pub (clé publique)

Étape 2 : Copier clé publique sur le SERVEUR
─────────────────────────────────────────────

Option A : ssh-copy-id (automatique)
$ ssh-copy-id bob@example.com
bob@example.com's password: ********
Number of key(s) added: 1

Option B : Manuelle
1. Afficher clé publique :
   $ cat ~/.ssh/id_ed25519.pub
   ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... bob@example.com

2. Se connecter au serveur et ajouter à authorized_keys :
   $ ssh bob@example.com
   $ mkdir -p ~/.ssh
   $ chmod 700 ~/.ssh
   $ echo "ssh-ed25519 AAAAC3..." >> ~/.ssh/authorized_keys
   $ chmod 600 ~/.ssh/authorized_keys

Étape 3 : Tester connexion sans mot de passe
────────────────────────────────────────────

$ sftp bob@example.com
Connected to example.com.
sftp>

✓ Connexion directe, sans demander mot de passe !
(Ou seulement passphrase de la clé si configurée)
```

**Avantages clés SSH vs mot de passe :**
```
✓ Sécurité renforcée (clé privée jamais transmise)
✓ Résistant aux attaques brute-force
✓ Peut désactiver authentification par mot de passe
✓ Automatisation possible (scripts, cron)
✓ Gestion centralisée (une clé pour plusieurs serveurs)
```

### Fonctionnalités avancées SFTP

**Reprise de transfert :**
```
Transfert interrompu ?

SFTP peut reprendre là où il s'est arrêté :

sftp> get -a huge_file.iso
Resuming transfer of huge_file.iso from byte 524288000
huge_file.iso                       50% 2048MB   5.0MB/s   05:30 ETA

Avantage : Pas besoin de retransférer depuis le début !
```

**Transfert récursif :**
```
Copier des arborescences complètes :

sftp> put -r /home/bob/project
Uploading project/ to /remote/project
project/src/main.c                  100%   5KB   5.0KB/s   00:00
project/src/utils.c                 100%   3KB   3.0KB/s   00:00
project/include/header.h            100%   2KB   2.0KB/s   00:00
project/README.md                   100%   1KB   1.0KB/s   00:00
project/Makefile                    100%   512B  512B/s    00:00

Tous les fichiers et sous-dossiers copiés en une commande !
```

**Opérations batch :**
```
Exécuter plusieurs commandes automatiquement :

$ sftp -b commands.txt bob@example.com

# Contenu de commands.txt :
cd uploads
put file1.txt
put file2.txt
mkdir archives
put -r documents/
chmod 644 *.txt
bye

Utile pour scripts et automatisation !
```

## Cas d'usage et recommandations

### Quand utiliser FTP ?

```
┌──────────────────────────────────────────────────────────┐
│ Scénarios où FTP (non sécurisé) peut être acceptable    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ ✓ FTP Anonyme public (lecture seule)
│   ├─ Téléchargement de logiciels open source
│   ├─ Archives publiques
│   ├─ Pas de données sensibles
│   └─ Exemple : ftp.gnu.org, ftp.debian.org
│
│ ✓ Réseau local isolé (pas Internet)
│   ├─ LAN d'entreprise sécurisé physiquement
│   ├─ Pas d'accès externe
│   └─ Performance maximale (pas de chiffrement)
│
│ ✓ Compatibilité avec matériel ancien
│   ├─ Imprimantes anciennes
│   ├─ Équipements industriels
│   └─ Pas de support SSH disponible
│
│ ✗ JAMAIS pour :
│   ├─ Authentification avec mot de passe réel
│   ├─ Transfert données confidentielles
│   ├─ Accès depuis Internet
│   └─ Tout usage professionnel moderne
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Quand utiliser FTPS ?

```
┌──────────────────────────────────────────────────────────┐
│ Scénarios où FTPS peut être approprié                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ ✓ Migration depuis FTP existant
│   ├─ Infrastructure FTP déjà en place
│   ├─ Besoin de sécuriser sans tout changer
│   └─ Clients supportent FTPS
│
│ ✓ Compatibilité avec systèmes legacy
│   ├─ Applications anciennes requérant FTP
│   ├─ Mise à niveau vers FTPS possible
│   └─ SFTP pas disponible/supporté
│
│ ⚠️ MAIS attention aux :
│   ├─ Problèmes NAT/firewall complexes
│   ├─ Configuration serveur délicate
│   └─ Support inégal selon clients
│
│ Recommandation : Préférer SFTP si possible
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Quand utiliser SFTP ?

```
┌──────────────────────────────────────────────────────────┐
│ Scénarios où SFTP est recommandé (la majorité)           │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ ✓✓ Tout transfert de fichiers sécurisé moderne
│    └─ C'est le standard actuel !
│
│ ✓✓ Accès depuis Internet
│    ├─ Employés en télétravail
│    ├─ Partenaires externes
│    └─ Déploiements d'applications
│
│ ✓✓ Automatisation (scripts, CI/CD)
│    ├─ Authentification par clé
│    ├─ Pas d'interaction humaine
│    └─ Intégration facile (scp, rsync over SSH)
│
│ ✓✓ Environnements avec NAT/firewall
│    ├─ Un seul port (22) à ouvrir
│    ├─ Compatible automatiquement
│    └─ Pas de configuration spéciale
│
│ ✓✓ Transferts confidentiels
│    ├─ Documents sensibles
│    ├─ Code source propriétaire
│    ├─ Données personnelles (RGPD)
│    └─ Tout ce qui nécessite confidentialité
│
│ ✓✓ Gestion de serveurs Linux/Unix
│    ├─ SSH déjà présent et configuré
│    ├─ SFTP inclus automatiquement
│    └─ Pas d'installation supplémentaire
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Alternatives modernes

```
┌──────────────────────────────────────────────────────────┐
│ Alternatives à FTP/SFTP selon le contexte                │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ SCP (Secure Copy)                                        │
│ ├─ Basé sur SSH (comme SFTP)                             │
│ ├─ Syntaxe simple : scp file.txt user@host:/path         │
│ ├─ Pas interactif (ligne de commande)                    │
│ └─ Usage : Copies ponctuelles rapides                    │
│                                                          │
│ rsync (sur SSH)                                          │
│ ├─ Synchronisation intelligente                          │
│ ├─ Transfère seulement les différences                   │
│ ├─ Très efficace pour sauvegardes                        │
│ └─ Usage : Synchro répertoires, backups                  │
│                                                          │
│ WebDAV / HTTP(S)                                         │
│ ├─ Basé sur HTTP (port 80/443)                           │
│ ├─ Compatible pare-feu (HTTP autorisé partout)           │
│ ├─ Intégration navigateurs/explorateurs                  │
│ └─ Usage : Partage fichiers web, clouds                  │
│                                                          │
│ Cloud Storage (S3, Google Drive, Dropbox...)             │
│ ├─ APIs REST sur HTTPS                                   │
│ ├─ Scalabilité automatique                               │
│ ├─ Versioning, partage, collaboration                    │
│ └─ Usage : Stockage cloud, partage moderne               │
│                                                          │
│ Solutions d'entreprise (Nextcloud, Seafile...)           │
│ ├─ Auto-hébergées ou cloud                               │
│ ├─ Interface web + sync clients                          │
│ ├─ Collaboration, versioning, permissions                │
│ └─ Usage : Remplacement complet FTP en entreprise        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Comparaison complète

```
┌──────────────────────────────────────────────────────────────────────┐
│ Tableau comparatif : FTP, FTPS, SFTP                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│ Critère           │ FTP          │ FTPS         │ SFTP
│───────────────────┼──────────────┼──────────────┼──────────────────
│ Sécurité          │ Aucune ✗     │ TLS/SSL ✓    │ SSH ✓✓
│ Chiffrement       │ Non          │ Oui          │ Oui
│ Authentification  │ Clair ✗      │ Chiffré ✓    │ Chiffré/Clé ✓✓
│ Ports             │ 20, 21       │ 21/990 + dyn.│ 22 uniquement
│ Connexions        │ 2 (ctrl+data)│ 2 (chiffrées)│ 1 (tunnel)
│ NAT/Firewall      │ Difficile ⚠️ │ Très diff. ✗ │ Facile ✓✓
│ Configuration     │ Simple       │ Complexe ⚠️  │ Moyenne
│ Performance       │ Excellente   │ Bonne        │ Bonne
│ Compatibilité     │ Universelle  │ Variable ⚠️  │ Excellente ✓
│ Reprise transfert │ Partielle    │ Oui          │ Oui ✓
│ Automatisation    │ Possible ⚠️  │ Difficile    │ Facile ✓✓
│ Support moderne   │ Déprécié ✗   │ Déclinant    │ Standard ✓✓
│
│ Usage recommandé :
│ ├─ FTP : FTP anonyme public, réseaux isolés
│ ├─ FTPS : Migration FTP legacy avec besoin sécurité
│ └─ SFTP : Tout le reste (99% des cas) ✓✓✓
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

## Clients FTP/SFTP courants

### Clients graphiques

```
┌──────────────────────────────────────────────────────────┐
│ Clients FTP/SFTP graphiques populaires                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ FileZilla                                                │
│ ├─ Plateformes : Windows, macOS, Linux                   │
│ ├─ Protocoles : FTP, FTPS, SFTP                          │
│ ├─ Gratuit et open source                                │
│ ├─ Interface intuitive                                   │
│ └─ ✓ Le plus populaire                                   │
│                                                          │
│ WinSCP (Windows uniquement)                              │
│ ├─ Plateforme : Windows                                  │
│ ├─ Protocoles : SFTP, SCP, FTP                           │
│ ├─ Gratuit et open source                                │
│ ├─ Intégration avec PuTTY                                │
│ └─ ✓ Excellent pour Windows                              │
│                                                          │
│ Cyberduck                                                │
│ ├─ Plateformes : macOS, Windows                          │
│ ├─ Protocoles : FTP, SFTP, WebDAV, S3...                 │
│ ├─ Interface élégante                                    │
│ └─ ✓ Idéal pour macOS                                    │
│                                                          │
│ Transmit (macOS)                                         │
│ ├─ Plateforme : macOS                                    │
│ ├─ Commercial (payant)                                   │
│ ├─ Très poli, rapide                                     │
│ └─ ✓ Premium pour Mac                                    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Clients en ligne de commande

```
Ligne de commande FTP/SFTP :

# FTP natif (tous les OS)
ftp ftp.example.com

# SFTP (Linux/Mac, OpenSSH)
sftp user@example.com

# SCP (copie rapide)
scp file.txt user@example.com:/path/

# rsync over SSH (synchronisation)
rsync -avz -e ssh /local/ user@example.com:/remote/

# lftp (client avancé, Linux)
lftp sftp://user@example.com

# curl (téléchargement)
curl -u user:pass ftp://example.com/file.txt
```

## Points clés à retenir

🔑 **FTP utilise 2 connexions distinctes (contrôle + données)**

🔑 **Mode passif (PASV) nécessaire avec NAT/firewall (serveur écoute, client se connecte)**

🔑 **FTP classique est non sécurisé : authentification et données en clair**

🔑 **FTPS = FTP + TLS/SSL, mais complexe avec NAT/firewall**

🔑 **SFTP ≠ FTPS : protocoles complètement différents**

🔑 **SFTP = protocole moderne sur SSH (port 22), une seule connexion**

🔑 **SFTP est le standard recommandé pour transferts sécurisés**

🔑 **Authentification SSH par clé publique recommandée pour SFTP**

🔑 **Mode binaire par défaut pour éviter corruption des fichiers**

🔑 **Alternatives : SCP, rsync, WebDAV, cloud storage selon besoins**

---

## Ce que nous avons appris

Dans cette section, nous avons exploré :

- ✅ FTP : architecture à deux connexions, modes actif/passif
- ✅ Les problèmes de sécurité critiques de FTP
- ✅ FTPS : tentative de sécurisation avec TLS/SSL
- ✅ SFTP : protocole moderne sur SSH, complètement différent de FTP
- ✅ L'architecture simplifiée de SFTP (une connexion tunnel)
- ✅ L'authentification par clé SSH pour SFTP
- ✅ Les commandes et usages pratiques
- ✅ Quand utiliser FTP, FTPS ou SFTP
- ✅ Les alternatives modernes (SCP, rsync, cloud)
- ✅ Les clients graphiques et en ligne de commande

## Conclusion

FTP a joué un rôle historique majeur dans l'évolution d'Internet, mais son architecture à deux connexions et son absence totale de sécurité le rendent inadapté aux usages modernes. FTPS tente de corriger les problèmes de sécurité mais hérite des complications architecturales.

**SFTP** est le vainqueur clair pour les transferts de fichiers sécurisés modernes :
- Architecture simple (une connexion)
- Sécurité excellente (SSH)
- Compatible NAT/firewall naturellement
- Largement supporté
- Standard de facto actuel

Pour les nouveaux projets, choisissez **SFTP** par défaut. FTP ne devrait être utilisé que dans des cas très spécifiques (FTP anonyme public, réseaux isolés) où la sécurité n'est pas une préoccupation.

---

**Prochaine section : Protocoles de messagerie (SMTP, POP3, IMAP)** 👉

⏭️ [SMTP, POP3, IMAP : protocoles de messagerie](/05-couche-application/06-smtp-pop3-imap.md)
