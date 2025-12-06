🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.6 SMTP, POP3, IMAP : protocoles de messagerie

## Introduction

Le **courrier électronique** (email) est l'un des services les plus anciens et les plus utilisés d'Internet. Inventé dans les années 1970, il reste aujourd'hui un pilier de la communication professionnelle et personnelle, avec plus de 300 milliards d'emails envoyés chaque jour dans le monde.

Contrairement à ce qu'on pourrait penser, l'email ne repose pas sur un seul protocole mais sur **trois protocoles principaux** qui travaillent ensemble :
- **SMTP** (Simple Mail Transfer Protocol) : Pour **envoyer** les emails
- **POP3** (Post Office Protocol v3) : Pour **recevoir** les emails (méthode classique)
- **IMAP** (Internet Message Access Protocol) : Pour **gérer** les emails (méthode moderne)

Dans cette section, nous allons explorer chacun de ces protocoles en profondeur, comprendre leur fonctionnement, leurs différences, et savoir quand utiliser l'un plutôt que l'autre.

## Architecture de la messagerie électronique

### Vue d'ensemble du système

```
┌──────────────────────────────────────────────────────────────┐
│ Parcours d'un email de l'expéditeur au destinataire          │
└──────────────────────────────────────────────────────────────┘

Alice (alice@example.com) envoie un email à Bob (bob@company.org)

  1. ENVOI                    2. TRANSFERT                3. RÉCEPTION

┌──────────┐              ┌──────────┐              ┌──────────┐
│ Client   │    SMTP      │ Serveur  │    SMTP      │ Serveur  │
│ email    │─────────────>│   MTA    │─────────────>│   MDA    │
│ (Alice)  │   Port 587   │ example  │   Port 25    │ company  │
└──────────┘              └──────────┘              └────┬─────┘
                          Serveur SMTP                   │
                          sortant                        │ Stockage
                                                         │
                                                    ┌────▼─────┐
                                                    │ Boîte aux│
                                                    │ lettres  │
                                                    │ de Bob   │
                                                    └────┬─────┘
                                                         │
                                                         │ POP3/IMAP
                                                         │
                                                    ┌────▼─────┐
                                                    │ Client   │
                                                    │ email    │
                                                    │ (Bob)    │
                                                    └──────────┘

Protocoles utilisés :
├─ SMTP (port 587/25) : Alice → Serveur example.com → Serveur company.org
├─ POP3 (port 110/995) : Serveur company.org → Bob (téléchargement)
└─ IMAP (port 143/993) : Bob ↔ Serveur company.org (synchronisation)
```

### Les acteurs de l'email

**1. MUA (Mail User Agent) - Client de messagerie**
```
Rôle : Interface utilisateur pour composer et lire les emails
Exemples :
├─ Clients lourds : Outlook, Thunderbird, Apple Mail
├─ Webmails : Gmail, Outlook.com, Yahoo Mail
└─ Clients mobiles : iOS Mail, Android Gmail

Fonctions :
├─ Composer, lire, organiser emails
├─ Utilise SMTP pour envoyer
└─ Utilise POP3/IMAP pour recevoir
```

**2. MTA (Mail Transfer Agent) - Serveur de transfert**
```
Rôle : Acheminer les emails entre serveurs
Exemples :
├─ Postfix (Linux, très populaire)
├─ Sendmail (historique)
├─ Exim (utilisé par cPanel)
└─ Microsoft Exchange

Fonctions :
├─ Recevoir emails via SMTP
├─ Résoudre DNS (MX records)
├─ Transférer vers serveur destinataire
└─ Gérer files d'attente, réessais
```

**3. MDA (Mail Delivery Agent) - Agent de livraison**
```
Rôle : Livrer l'email dans la boîte aux lettres du destinataire
Exemples :
├─ Dovecot
├─ Courier
└─ Cyrus

Fonctions :
├─ Stocker emails sur disque
├─ Gérer boîtes aux lettres
├─ Servir les emails via POP3/IMAP
└─ Filtrage (spam, règles)
```

### Analogie avec le courrier postal

```
┌──────────────────────────────────────────────────────────────┐
│ Email vs Courrier postal                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Courrier postal              Email                           │
│ ───────────────              ─────                           │
│                                                              │
│ Vous écrivez lettre          MUA (client email)              │
│ ↓                            ↓                               │
│ Boîte postale locale         Serveur SMTP sortant            │
│ ↓                            ↓                               │
│ Centre de tri                MTA intermédiaires              │
│ ↓                            ↓                               │
│ Bureau de poste              Serveur SMTP destinataire       │
│ destinataire                 ↓                               │
│ ↓                            MDA (stockage)                  │
│ Boîte aux lettres            ↓                               │
│ ↓                            Boîte aux lettres (serveur)     │
│ Vous récupérez               POP3/IMAP                       │
│                              ↓                               │
│                              Vous lisez (MUA)                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## SMTP (Simple Mail Transfer Protocol)

### Présentation

```
┌──────────────────────────────────────────────────────────────┐
│ SMTP - Simple Mail Transfer Protocol                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Création : 1982                                              │
│ RFC : RFC 5321 (2008, mise à jour de RFC 821)                │
│ Ports :                                                      │
│   ├─ 25  : Transfert serveur-à-serveur (MTA ↔ MTA)           │
│   ├─ 587 : Soumission client (MUA → MTA, recommandé)         │
│   └─ 465 : SMTP over SSL (déprécié, mais encore utilisé)     │
│                                                              │
│ Protocole : Texte ASCII, commandes/réponses                  │
│ Transport : TCP                                              │
│ Direction : ENVOI uniquement (push)                          │
│                                                              │
│ Caractéristiques :                                           │
│ ├─ Simple et lisible (texte clair)                           │
│ ├─ Dialogues questions-réponses                              │
│ ├─ Codes de statut à 3 chiffres                              │
│ └─ Extensible (ESMTP)                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Commandes SMTP principales

```
┌──────────────────────────────────────────────────────────────┐
│ Commandes SMTP essentielles                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ CONNEXION ET IDENTIFICATION                                  │
│ ├─ HELO <domain>      : Identification (SMTP basique)        │
│ ├─ EHLO <domain>      : Identification (ESMTP étendu)        │
│ └─ QUIT               : Fermer connexion                     │
│                                                              │
│ TRANSFERT EMAIL                                              │
│ ├─ MAIL FROM:<sender> : Indiquer expéditeur                  │
│ ├─ RCPT TO:<recipient>: Indiquer destinataire(s)             │
│ ├─ DATA               : Début du contenu                     │
│ │   [corps du message]                                       │
│ │   .                 ← Ligne seule avec point = fin         │
│ └─ RSET               : Annuler transaction en cours         │
│                                                              │
│ EXTENSIONS ESMTP                                             │
│ ├─ AUTH <mechanism>   : Authentification                     │
│ ├─ STARTTLS           : Démarrer chiffrement TLS             │
│ ├─ SIZE <size>        : Taille message (négociation)         │
│ └─ VRFY <address>     : Vérifier adresse (souvent désactivé) │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Codes de réponse SMTP

```
┌──────────────────────────────────────────────────────────────┐
│ Codes de statut SMTP (format XYZ)                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ X = Catégorie de réponse :                                   │
│ ├─ 2xx : Succès                                              │
│ ├─ 3xx : Informations supplémentaires attendues              │
│ ├─ 4xx : Erreur temporaire (réessayer plus tard)             │
│ └─ 5xx : Erreur permanente (abandon)                         │
│                                                              │
│ CODES COURANTS :                                             │
│                                                              │
│ 220 : Service ready                                          │
│ 221 : Closing connection                                     │
│ 250 : Requested action okay, completed                       │
│ 251 : User not local, will forward                           │
│ 354 : Start mail input, end with <CRLF>.<CRLF>               │
│ 421 : Service not available                                  │
│ 450 : Mailbox temporarily unavailable                        │
│ 451 : Action aborted, error in processing                    │
│ 452 : Insufficient storage                                   │
│ 500 : Syntax error, command unrecognized                     │
│ 501 : Syntax error in parameters                             │
│ 502 : Command not implemented                                │
│ 503 : Bad sequence of commands                               │
│ 504 : Command parameter not implemented                      │
│ 550 : Mailbox unavailable (not found, access denied)         │
│ 551 : User not local                                         │
│ 552 : Storage allocation exceeded                            │
│ 553 : Mailbox name not allowed                               │
│ 554 : Transaction failed                                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Session SMTP complète

```
┌──────────────────────────────────────────────────────────────┐
│ Exemple de session SMTP (envoi d'un email)                   │
└──────────────────────────────────────────────────────────────┘

Client (alice@example.com) → Serveur SMTP (mail.example.com)

S: 220 mail.example.com ESMTP Postfix
C: EHLO client.example.com
S: 250-mail.example.com
S: 250-PIPELINING
S: 250-SIZE 10240000
S: 250-VRFY
S: 250-ETRN
S: 250-STARTTLS
S: 250-AUTH PLAIN LOGIN
S: 250-AUTH=PLAIN LOGIN
S: 250-ENHANCEDSTATUSCODES
S: 250-8BITMIME
S: 250 DSN

C: STARTTLS
S: 220 2.0.0 Ready to start TLS

[Négociation TLS - connexion chiffrée établie]

C: EHLO client.example.com
S: 250-mail.example.com
S: 250-PIPELINING
S: 250-SIZE 10240000
S: 250-VRFY
S: 250-ETRN
S: 250-AUTH PLAIN LOGIN
S: 250-ENHANCEDSTATUSCODES
S: 250-8BITMIME
S: 250 DSN

C: AUTH LOGIN
S: 334 VXNlcm5hbWU6
C: YWxpY2U=
S: 334 UGFzc3dvcmQ6
C: bXlwYXNzd29yZA==
S: 235 2.7.0 Authentication successful

C: MAIL FROM:<alice@example.com>
S: 250 2.1.0 Ok

C: RCPT TO:<bob@company.org>
S: 250 2.1.5 Ok

C: DATA
S: 354 End data with <CR><LF>.<CR><LF>
C: From: Alice <alice@example.com>
C: To: Bob <bob@company.org>
C: Subject: Réunion demain
C: Date: Wed, 06 Dec 2024 10:30:00 +0100
C: Message-ID: <abc123@example.com>
C:
C: Bonjour Bob,
C:
C: N'oublie pas la réunion demain à 14h.
C:
C: Cordialement,
C: Alice
C: .
S: 250 2.0.0 Ok: queued as 3F4A51234

C: QUIT
S: 221 2.0.0 Bye

Légende :
S: = Serveur
C: = Client
```

### Authentification SMTP

**Pourquoi l'authentification ?**
```
Sans authentification :
├─ N'importe qui peut envoyer via le serveur
├─ Serveur devient "open relay"
├─ Utilisé pour spam
└─ Serveur mis en liste noire ✗

Avec authentification :
├─ Seuls les utilisateurs autorisés peuvent envoyer
├─ Protection contre spam
├─ Traçabilité des envois
└─ Sécurité ✓
```

**Mécanismes d'authentification :**
```
AUTH PLAIN :
├─ Identifiants en base64 (pas sécurisé sans TLS !)
├─ Format : \0username\0password (encodé)
└─ Simple mais doit être utilisé avec STARTTLS

AUTH LOGIN :
├─ Dialogue username puis password
├─ Chacun encodé en base64 séparément
└─ Également nécessite TLS

AUTH CRAM-MD5 :
├─ Challenge-response avec hash MD5
├─ Pas de transmission du mot de passe
└─ Plus sécurisé mais moins supporté

Recommandation moderne :
AUTH PLAIN ou LOGIN avec STARTTLS obligatoire
```

### Relayage et MX Records

**Résolution de destination :**
```
┌──────────────────────────────────────────────────────────────┐
│ Comment SMTP trouve le serveur destinataire                  │
└──────────────────────────────────────────────────────────────┘

Email à envoyer : alice@example.com → bob@company.org

Étape 1 : Extraire le domaine
Domain : company.org

Étape 2 : Requête DNS MX (Mail eXchanger)
$ dig company.org MX

company.org.    3600    IN    MX    10 mail1.company.org.
company.org.    3600    IN    MX    20 mail2.company.org.
                              ↑
                        Priorité (plus faible = préféré)

Étape 3 : Résoudre l'IP du serveur mail
$ dig mail1.company.org A

mail1.company.org.    3600    IN    A    203.0.113.10

Étape 4 : Connexion SMTP
Serveur example.com se connecte à 203.0.113.10:25
Transfère l'email via SMTP

Étape 5 : En cas d'échec
Si mail1 ne répond pas, essayer mail2 (priorité 20)
Fallback automatique ✓
```

**Enregistrements MX typiques :**
```
# Configuration DNS pour company.org

company.org.    IN    MX    10    mail1.company.org.
company.org.    IN    MX    20    mail2.company.org.
company.org.    IN    MX    30    mail-backup.example.net.

mail1.company.org.    IN    A    203.0.113.10
mail2.company.org.    IN    A    203.0.113.11
mail-backup.example.net.    IN    A    198.51.100.50

Priorités :
├─ 10 : Serveur principal (essayé en premier)
├─ 20 : Serveur secondaire (backup)
└─ 30 : Serveur de secours externe

Avantages :
✓ Redondance (si serveur principal down)
✓ Répartition de charge possible
✓ Maintenance sans perte d'emails
```

## POP3 (Post Office Protocol v3)

### Présentation

```
┌──────────────────────────────────────────────────────────────┐
│ POP3 - Post Office Protocol version 3                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Création : 1988 (POP3 en 1996)                               │
│ RFC : RFC 1939                                               │
│ Ports :                                                      │
│   ├─ 110 : POP3 non sécurisé                                 │
│   └─ 995 : POP3S (POP3 over SSL/TLS)                         │
│                                                              │
│ Protocole : Texte ASCII, commandes simples                   │
│ Transport : TCP                                              │
│ Direction : RÉCEPTION (pull)                                 │
│                                                              │
│ Philosophie :                                                │
│ "Télécharger et supprimer"                                   │
│ ├─ Client se connecte                                        │
│ ├─ Télécharge tous les emails                                │
│ ├─ Supprime du serveur (optionnel)                           │
│ └─ Déconnexion                                               │
│                                                              │
│ Caractéristiques :                                           │
│ ├─ Simple et léger                                           │
│ ├─ Accès offline après téléchargement                        │
│ ├─ Pas de synchronisation                                    │
│ └─ Un seul appareil recommandé                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### États de session POP3

```
┌──────────────────────────────────────────────────────────────┐
│ Machine à états POP3                                         │
└──────────────────────────────────────────────────────────────┘

┌─────────────┐
│ AUTORISATION│  État initial
│             │  Commandes : USER, PASS, QUIT
└──────┬──────┘
       │
       │ Authentification réussie
       ↓
┌─────────────┐
│ TRANSACTION │  État principal
│             │  Commandes : STAT, LIST, RETR, DELE, NOOP, RSET
└──────┬──────┘
       │
       │ Commande QUIT
       ↓
┌─────────────┐
│ MISE À JOUR │  Finalisation
│             │  Application des suppressions
└─────────────┘  Fermeture connexion
```

### Commandes POP3

```
┌──────────────────────────────────────────────────────────────┐
│ Commandes POP3                                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ÉTAT AUTORISATION                                            │
│ ├─ USER <username>    : Nom d'utilisateur                    │
│ ├─ PASS <password>    : Mot de passe                         │
│ └─ QUIT               : Quitter (annule si non authentifié)  │
│                                                              │
│ ÉTAT TRANSACTION                                             │
│ ├─ STAT               : Statistiques (nombre/taille emails)  │
│ ├─ LIST [msg]         : Lister emails (ou détail d'un email) │
│ ├─ RETR <msg>         : Récupérer email                      │
│ ├─ DELE <msg>         : Marquer pour suppression             │
│ ├─ NOOP               : No operation (keep-alive)            │
│ ├─ RSET               : Annuler suppressions marquées        │
│ ├─ TOP <msg> <n>      : Récupérer n premières lignes         │
│ ├─ UIDL [msg]         : Unique ID listing                    │
│ └─ QUIT               : Quitter et appliquer suppressions    │
│                                                              │
│ RÉPONSES                                                     │
│ ├─ +OK                : Succès                               │
│ └─ -ERR               : Erreur                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Session POP3 complète

```
┌──────────────────────────────────────────────────────────────┐
│ Exemple de session POP3                                      │
└──────────────────────────────────────────────────────────────┘

Client se connecte au serveur POP3

S: +OK POP3 server ready
C: USER bob
S: +OK User accepted
C: PASS secret123
S: +OK Logged in

C: STAT
S: +OK 3 12345
   (3 messages, 12345 octets total)

C: LIST
S: +OK 3 messages
S: 1 5000
S: 2 3500
S: 3 3845
S: .

C: UIDL
S: +OK Unique ID listing
S: 1 abc123def456
S: 2 ghi789jkl012
S: 3 mno345pqr678
S: .

C: RETR 1
S: +OK 5000 octets
S: Return-Path: <alice@example.com>
S: Received: from mail.example.com ...
S: From: Alice <alice@example.com>
S: To: Bob <bob@company.org>
S: Subject: Hello
S: Date: Wed, 06 Dec 2024 10:00:00 +0100
S:
S: Contenu du message...
S: .

C: DELE 1
S: +OK Message 1 deleted

C: RETR 2
S: +OK 3500 octets
S: [Contenu du message 2...]
S: .

C: QUIT
S: +OK POP3 server signing off (2 messages deleted)

[Connexion fermée]

Légende :
S: = Serveur
C: = Client
```

### UIDL - Éviter les doublons

```
┌──────────────────────────────────────────────────────────────┐
│ UIDL (Unique ID Listing) - Gestion des messages déjà lus     │
└──────────────────────────────────────────────────────────────┘

Problème :
Si on laisse les emails sur le serveur (ne pas supprimer),
comment savoir lesquels ont déjà été téléchargés ?

Solution : UIDL
Chaque message a un identifiant unique permanent

Session 1 :
C: UIDL
S: +OK
S: 1 abc123def456
S: 2 ghi789jkl012
S: .

Client télécharge et stocke les UIDs localement :
Téléchargés : [abc123def456, ghi789jkl012]

Session 2 (lendemain) :
C: UIDL
S: +OK
S: 1 abc123def456  ← Déjà connu, skip
S: 2 ghi789jkl012  ← Déjà connu, skip
S: 3 mno345pqr678  ← Nouveau ! Télécharger
S: .

Avantages :
✓ Pas de doublons
✓ Peut laisser emails sur serveur
✓ Support multi-appareils partiel
```

### Mode "laisser sur serveur"

```
Options de téléchargement POP3 :

Mode 1 : Télécharger et supprimer (classique)
├─ RETR message
├─ DELE message
├─ Email supprimé du serveur
└─ Libère espace serveur
    ✓ Simple
    ✗ Email sur un seul appareil

Mode 2 : Laisser sur serveur
├─ RETR message
├─ Pas de DELE
├─ Email reste sur serveur
└─ Utilise UIDL pour éviter doublons
    ✓ Backup sur serveur
    ✓ Multi-appareils possible (limité)
    ✗ Quota serveur atteint rapidement

Mode 3 : Supprimer après X jours
├─ Laisse sur serveur pendant durée définie
├─ Client supprime automatiquement les anciens
└─ Compromis entre les deux
    ✓ Backup temporaire
    ✓ Libération automatique espace
```

## IMAP (Internet Message Access Protocol)

### Présentation

```
┌──────────────────────────────────────────────────────────────┐
│ IMAP - Internet Message Access Protocol                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Création : 1986 (IMAP4 en 1994, IMAP4rev1 en 2003)           │
│ RFC : RFC 3501 (IMAP4rev1)                                   │
│ Ports :                                                      │
│   ├─ 143 : IMAP non sécurisé                                 │
│   └─ 993 : IMAPS (IMAP over SSL/TLS)                         │
│                                                              │
│ Protocole : Texte ASCII, commandes étiquetées                │
│ Transport : TCP                                              │
│ Direction : RÉCEPTION et GESTION (synchronisation)           │
│                                                              │
│ Philosophie :                                                │
│ "Gestion sur le serveur"                                     │
│ ├─ Emails restent sur le serveur                             │
│ ├─ Client synchronise avec serveur                           │
│ ├─ Gestion dossiers, flags, recherche                        │
│ └─ Multi-appareils natif                                     │
│                                                              │
│ Caractéristiques :                                           │
│ ├─ Complexe mais puissant                                    │
│ ├─ Synchronisation bidirectionnelle                          │
│ ├─ Gestion hors-ligne intelligente                           │
│ ├─ Recherche côté serveur                                    │
│ └─ Multi-appareils parfait                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### États de session IMAP

```
┌──────────────────────────────────────────────────────────────┐
│ Machine à états IMAP                                         │
└──────────────────────────────────────────────────────────────┘

┌──────────────┐
│ NON          │  État initial
│ AUTHENTIFIÉ  │  Commandes : CAPABILITY, STARTTLS, LOGIN, AUTHENTICATE
└───────┬──────┘
        │
        │ Authentification réussie
        ↓
┌──────────────┐
│ AUTHENTIFIÉ  │  Connecté mais pas de boîte sélectionnée
│              │  Commandes : SELECT, EXAMINE, CREATE, DELETE, LIST
└───────┬──────┘
        │
        │ Sélection boîte (SELECT/EXAMINE)
        ↓
┌──────────────┐
│ SÉLECTIONNÉ  │  Boîte active, accès aux messages
│              │  Commandes : FETCH, STORE, SEARCH, COPY, EXPUNGE
└───────┬──────┘
        │
        │ CLOSE ou sélection autre boîte
        ↓
┌──────────────┐
│ DÉCONNEXION  │  LOGOUT
└──────────────┘
```

### Commandes IMAP principales

```
┌──────────────────────────────────────────────────────────────┐
│ Commandes IMAP essentielles                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ CONNEXION                                                    │
│ ├─ CAPABILITY           : Lister capacités serveur           │
│ ├─ LOGIN <user> <pass>  : Authentification                   │
│ ├─ AUTHENTICATE <mech>  : Auth. avancée (CRAM-MD5, etc.)     │
│ ├─ STARTTLS             : Démarrer TLS                       │
│ └─ LOGOUT               : Déconnexion                        │
│                                                              │
│ GESTION BOÎTES (MAILBOXES)                                   │
│ ├─ LIST <ref> <pattern> : Lister boîtes                      │
│ ├─ LSUB <ref> <pattern> : Lister abonnements                 │
│ ├─ CREATE <mailbox>     : Créer boîte                        │
│ ├─ DELETE <mailbox>     : Supprimer boîte                    │
│ ├─ RENAME <old> <new>   : Renommer boîte                     │
│ ├─ SUBSCRIBE <mailbox>  : S'abonner                          │
│ └─ UNSUBSCRIBE <mailbox>: Se désabonner                      │
│                                                              │
│ SÉLECTION BOÎTE                                              │
│ ├─ SELECT <mailbox>     : Sélectionner (lecture/écriture)    │
│ ├─ EXAMINE <mailbox>    : Sélectionner (lecture seule)       │
│ └─ CLOSE                : Fermer boîte sélectionnée          │
│                                                              │
│ GESTION MESSAGES                                             │
│ ├─ FETCH <set> <items>  : Récupérer messages/parties         │
│ ├─ STORE <set> <flags>  : Modifier flags                     │
│ ├─ COPY <set> <mailbox> : Copier messages                    │
│ ├─ EXPUNGE              : Supprimer définitivement           │
│ └─ UID <command>        : Version UID des commandes          │
│                                                              │
│ RECHERCHE                                                    │
│ ├─ SEARCH <criteria>    : Chercher messages                  │
│ └─ UID SEARCH           : Recherche par UID                  │
│                                                              │
│ AUTRES                                                       │
│ ├─ STATUS <mailbox>     : État boîte sans la sélectionner    │
│ ├─ CHECK                : Checkpoint                         │
│ └─ NOOP                 : Keep-alive + notifications         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Système d'étiquettes (Tags)

IMAP utilise un système d'**étiquettes** unique :

```
Format des commandes IMAP :
<tag> <commande> <arguments>

Exemple :
A001 LOGIN bob secret123
A002 LIST "" "*"
A003 SELECT INBOX

Réponses serveur :
* <données non sollicitées>
<tag> OK|NO|BAD <message>

Exemple :
* 172 EXISTS
* 1 RECENT
A003 OK [READ-WRITE] SELECT completed

Pourquoi des tags ?
├─ Permet commandes asynchrones (pipeline)
├─ Client peut envoyer plusieurs commandes
├─ Serveur répond dans n'importe quel ordre
└─ Tag permet d'associer réponse à commande

Avantage : Performance optimisée
Client peut envoyer :
A001 SELECT INBOX
A002 FETCH 1:10 (FLAGS)
A003 FETCH 11:20 (FLAGS)
Sans attendre les réponses !
```

### Flags (drapeaux) des messages

```
┌──────────────────────────────────────────────────────────────┐
│ Flags IMAP - État des messages                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ FLAGS SYSTÈME (standards)                                    │
│ ├─ \Seen        : Message lu                                 │
│ ├─ \Answered    : Message répondu                            │
│ ├─ \Flagged     : Message marqué/étoilé                      │
│ ├─ \Deleted     : Marqué pour suppression (pas encore fait)  │
│ ├─ \Draft       : Message brouillon                          │
│ └─ \Recent      : Message récemment arrivé                   │
│                                                              │
│ FLAGS PERSONNALISÉS                                          │
│ ├─ $MDNSent     : Accusé de lecture envoyé                   │
│ ├─ $Forwarded   : Message transféré                          │
│ ├─ Junk         : Spam (selon client)                        │
│ └─ Custom tags  : Labels personnalisés                       │
│                                                              │
│ OPÉRATIONS                                                   │
│ Ajouter flag :                                               │
│   STORE 5 +FLAGS (\Seen)                                     │
│                                                              │
│ Retirer flag :                                               │
│   STORE 5 -FLAGS (\Flagged)                                  │
│                                                              │
│ Remplacer flags :                                            │
│   STORE 5 FLAGS (\Seen \Answered)                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Session IMAP complète

```
┌──────────────────────────────────────────────────────────────┐
│ Exemple de session IMAP                                      │
└──────────────────────────────────────────────────────────────┘

Client se connecte au serveur IMAP

S: * OK IMAP4rev1 Server Ready
C: A001 CAPABILITY
S: * CAPABILITY IMAP4rev1 STARTTLS AUTH=PLAIN AUTH=LOGIN
S: A001 OK CAPABILITY completed

C: A002 STARTTLS
S: A002 OK Begin TLS negotiation

[Négociation TLS]

C: A003 LOGIN bob secret123
S: A003 OK Logged in

C: A004 LIST "" "*"
S: * LIST (\HasNoChildren) "." INBOX
S: * LIST (\HasNoChildren) "." Sent
S: * LIST (\HasNoChildren) "." Drafts
S: * LIST (\HasChildren) "." Work
S: * LIST (\HasNoChildren) "." Work.Projects
S: A004 OK LIST completed

C: A005 SELECT INBOX
S: * FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
S: * OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)]
S: * 172 EXISTS
S: * 1 RECENT
S: * OK [UNSEEN 12] First unseen message
S: * OK [UIDVALIDITY 1234567890]
S: * OK [UIDNEXT 173]
S: A005 OK [READ-WRITE] SELECT completed

C: A006 SEARCH UNSEEN
S: * SEARCH 12 27 54 108 145 172
S: A006 OK SEARCH completed

C: A007 FETCH 172 (FLAGS BODY[HEADER])
S: * 172 FETCH (FLAGS (\Recent) BODY[HEADER] {342}
S: Return-Path: <alice@example.com>
S: From: Alice <alice@example.com>
S: To: Bob <bob@company.org>
S: Subject: Urgent - Réunion
S: Date: Wed, 06 Dec 2024 14:30:00 +0100
S: Message-ID: <xyz789@example.com>
S:
S: )
S: A007 OK FETCH completed

C: A008 FETCH 172 BODY[TEXT]
S: * 172 FETCH (BODY[TEXT] {156}
S: Bonjour Bob,
S:
S: La réunion de demain est avancée à 10h.
S: Merci de confirmer ta présence.
S:
S: Alice
S: )
S: A008 OK FETCH completed

C: A009 STORE 172 +FLAGS (\Seen)
S: * 172 FETCH (FLAGS (\Seen \Recent))
S: A009 OK STORE completed

C: A010 COPY 172 Work.Projects
S: A010 OK [COPYUID 1234567890 172 45] COPY completed

C: A011 LOGOUT
S: * BYE IMAP4rev1 Server logging out
S: A011 OK LOGOUT completed

[Connexion fermée]
```

### Gestion des dossiers (Mailboxes)

```
┌──────────────────────────────────────────────────────────────┐
│ Hiérarchie de dossiers IMAP                                  │
└──────────────────────────────────────────────────────────────┘

Structure typique :

INBOX                      (Boîte de réception)
├─ Sent                    (Messages envoyés)
├─ Drafts                  (Brouillons)
├─ Trash                   (Corbeille)
├─ Spam                    (Spam/Indésirables)
├─ Archive                 (Archives)
├─ Work                    (Dossier travail)
│  ├─ Work.Projects        (Sous-dossier)
│  ├─ Work.Clients         (Sous-dossier)
│  └─ Work.Archive         (Sous-dossier)
└─ Personal                (Dossier personnel)
   ├─ Personal.Family
   └─ Personal.Friends

Séparateurs :
├─ "." (point) : Standard Unix (exemple: Work.Projects)
├─ "/" (slash) : Alternative (exemple: Work/Projects)
└─ Dépend du serveur

Commandes :
C: A001 CREATE Work.NewProject
S: A001 OK CREATE completed

C: A002 LIST "" "Work.*"
S: * LIST (\HasNoChildren) "." Work.Projects
S: * LIST (\HasNoChildren) "." Work.Clients
S: * LIST (\HasNoChildren) "." Work.Archive
S: * LIST (\HasNoChildren) "." Work.NewProject
S: A002 OK LIST completed

C: A003 RENAME Work.NewProject Work.CompletedProject
S: A003 OK RENAME completed

C: A004 DELETE Work.CompletedProject
S: A004 OK DELETE completed
```

### IDLE - Notifications push

```
┌──────────────────────────────────────────────────────────────┐
│ IMAP IDLE - Push notifications en temps réel                 │
└──────────────────────────────────────────────────────────────┘

Problème classique :
Client doit "poller" régulièrement pour nouveaux messages
└─ Toutes les 5 minutes : "Y a-t-il du nouveau ?"
   ✗ Latence (jusqu'à 5 minutes)
   ✗ Trafic réseau gaspillé
   ✗ Batterie mobile drainée

Solution : IMAP IDLE (RFC 2177)
Client dit : "Préviens-moi s'il y a du nouveau"
Serveur maintient connexion et notifie immédiatement

Session avec IDLE :

C: A001 SELECT INBOX
S: * 172 EXISTS
S: * 1 RECENT
S: A001 OK SELECT completed

C: A002 IDLE
S: + idling

[Client et serveur attendent...]
[Nouveau message arrive !]

S: * 173 EXISTS
S: * 2 RECENT

[Client veut envoyer une commande]
C: DONE

S: A002 OK IDLE terminated

C: A003 FETCH 173 (FLAGS BODY[HEADER])
[...]

Avantages :
✓ Notifications instantanées
✓ Économie batterie (pas de polling)
✓ Réduit trafic réseau
✓ Expérience utilisateur améliorée

Utilisé par : Gmail, Outlook, Apple Mail, etc.
```

## Comparaison POP3 vs IMAP

```
┌───────────────────────────────────────────────────────────────────────┐
│ POP3 vs IMAP - Comparaison détaillée                                  │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ Critère              │ POP3                  │ IMAP                   │
│──────────────────────┼───────────────────────┼─────────────────────── │
│ Philosophie          │ Télécharger/supprimer │ Gérer sur serveur      │
│ Stockage principal   │ Client (local)        │ Serveur                │
│ Synchronisation      │ Non                   │ Oui (bidirectionnelle) │
│ Multi-appareils      │ Difficile ✗           │ Parfait ✓✓            │
│ Hors-ligne           │ Complet après DL ✓    │ Cache intelligent     │
│ Dossiers             │ Non (locaux uniquement│ Oui (sur serveur) ✓✓  │
│ Flags/État messages  │ Local uniquement      │ Synchronisés ✓        │
│ Recherche serveur    │ Non                   │ Oui ✓                 │
│ Quota serveur        │ Peu important         │ Important ⚠️          │
│ Bande passante       │ Télécharge tout       │ Optimisé (headers)    │
│ Complexité           │ Simple ✓              │ Complexe ⚠️           │
│ Performance          │ Rapide DL initial     │ Meilleure globale ✓   │
│ Push notifications   │ Non                   │ Oui (IDLE) ✓          │
│ Usage mobile         │ Inadapté ✗            │ Optimal ✓✓            │
│ Backup               │ Client responsable    │ Serveur = backup ✓    │
│ Ports standard       │ 110 (995 SSL)         │ 143 (993 SSL)         │
│                                                                       │
│ QUAND UTILISER POP3 ?                                                 │
│ ✓ Un seul appareil utilisé                                           │
│ ✓ Connexion Internet intermittente                                   │
│ ✓ Quota serveur très limité                                          │
│ ✓ Besoin de backup local complet                                     │
│ ✓ Simplicité requise                                                 │
│                                                                       │
│ QUAND UTILISER IMAP ?                                                 │
│ ✓✓ Plusieurs appareils (laptop, phone, tablet)                       │
│ ✓✓ Besoin d'accès emails partout                                     │
│ ✓✓ Organisation avec dossiers                                        │
│ ✓✓ Travail collaboratif (boîtes partagées)                          │
│ ✓✓ Usage moderne standard                                            │
│                                                                       │
│ RECOMMANDATION : IMAP pour 95% des cas                               │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

### Scénarios d'usage

**Scénario 1 : Utilisateur moderne (recommandé IMAP)**
```
Alice utilise :
├─ Laptop professionnel
├─ Smartphone personnel
├─ Tablette à la maison
└─ Webmail occasionnellement

Besoin :
├─ Emails synchronisés partout
├─ Lire sur phone, répondre sur laptop
├─ Organisation avec dossiers
└─ Notifications push

Solution : IMAP ✓✓
└─ Tous les appareils synchronisés
└─ Organisation cohérente
└─ Expérience fluide
```

**Scénario 2 : Utilisateur simple (POP3 acceptable)**
```
Bob utilise :
├─ Un seul ordinateur fixe
├─ Connexion Internet à la maison uniquement
└─ Lit emails le soir

Besoin :
├─ Télécharger emails pour lire hors-ligne
├─ Pas besoin de synchronisation
└─ Libérer espace serveur (quota limité)

Solution : POP3 ✓
└─ Simple et efficace pour ce cas
└─ Télécharge tout localement
└─ Pas de dépendance serveur
```

**Scénario 3 : Entreprise (IMAP obligatoire)**
```
Entreprise avec :
├─ 100 employés
├─ Boîtes aux lettres partagées (support@, ventes@)
├─ Accès mobile et bureau
└─ Archivage réglementaire

Besoin :
├─ Synchronisation multi-utilisateurs
├─ Gestion centralisée
├─ Backup serveur
└─ Recherche puissante

Solution : IMAP ✓✓ (seule option viable)
└─ Gestion centralisée
└─ Collaboration possible
└─ Conformité garantie
```

## Sécurité des protocoles email

### STARTTLS vs SSL/TLS implicite

```
┌──────────────────────────────────────────────────────────────┐
│ Méthodes de chiffrement pour SMTP/POP3/IMAP                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ MÉTHODE 1 : STARTTLS (recommandée)
│ ├─ Connexion initiale non chiffrée
│ ├─ Client demande : "STARTTLS"
│ ├─ Négociation TLS sur la même connexion
│ └─ Connexion devient chiffrée
│
│ Ports STARTTLS :
│ ├─ SMTP : 587 (submission)
│ ├─ POP3 : 110
│ └─ IMAP : 143
│
│ Avantages :
│ ✓ Fallback possible (si TLS échoue)
│ ✓ Un seul port pour chiffré et non-chiffré
│ ✓ Standard moderne
│
│ MÉTHODE 2 : SSL/TLS implicite (legacy)
│ ├─ TLS dès l'établissement connexion
│ ├─ Pas de négociation
│ └─ Connexion entièrement chiffrée
│
│ Ports SSL/TLS implicite :
│ ├─ SMTPS : 465
│ ├─ POP3S : 995
│ └─ IMAPS : 993
│
│ Avantages :
│ ✓ Sécurité maximale (pas de phase non chiffrée)
│ ✓ Plus simple (pas de négociation)
│
│ IMPORTANT :
│ En 2024, toute connexion DOIT être chiffrée !
│ Ports non sécurisés (25, 110, 143) uniquement
│ pour serveur-à-serveur ou réseaux internes.
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Authentification email

```
┌──────────────────────────────────────────────────────────────┐
│ Mécanismes d'authentification                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ SMTP AUTH                                                    │
│ ├─ PLAIN : Username/password en base64                       │
│ │   └─ DOIT être utilisé avec TLS !                          │
│ ├─ LOGIN : Similaire à PLAIN (legacy)                        │
│ ├─ CRAM-MD5 : Challenge-response (plus sécurisé)             │
│ └─ OAUTH2 : Tokens (Gmail, Outlook moderne)                  │
│                                                              │
│ POP3/IMAP AUTH                                               │
│ ├─ USER/PASS : Traditionnel (POP3)                           │
│ ├─ LOGIN : Commande standard (IMAP)                          │
│ ├─ AUTHENTICATE : Mécanismes avancés                         │
│ │   ├─ PLAIN                                                 │
│ │   ├─ CRAM-MD5                                              │
│ │   └─ OAUTH2                                                │
│ └─ Certificats client (rare)                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### SPF, DKIM, DMARC

Mécanismes de **validation de l'expéditeur** (côté serveur) :

```
┌──────────────────────────────────────────────────────────────┐
│ SPF (Sender Policy Framework)                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Problème : N'importe qui peut prétendre envoyer de votre     │
│            domaine (MAIL FROM: fake@example.com)             │
│                                                              │
│ Solution SPF : Publier liste serveurs autorisés dans DNS     │
│                                                              │
│ Enregistrement DNS :                                         │
│ example.com.  TXT  "v=spf1 ip4:203.0.113.0/24 include:_spf.  │
│                     google.com -all"                         │
│                                                              │
│ Signification :                                              │
│ ├─ v=spf1 : Version SPF                                      │
│ ├─ ip4:203.0.113.0/24 : Mes serveurs                         │
│ ├─ include:_spf.google.com : + serveurs Google (GSuite)      │
│ └─ -all : Rejeter tout le reste (fail)                       │
│                                                              │
│ Validation :                                                 │
│ Serveur destinataire vérifie :                               │
│ "L'IP qui envoie est-elle autorisée pour ce domaine ?"       │
│                                                              │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│ DKIM (DomainKeys Identified Mail)                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Problème : Email peut être modifié en transit                │
│                                                              │
│ Solution DKIM : Signature cryptographique de l'email         │
│                                                              │
│ Processus :                                                  │
│ 1. Serveur expéditeur signe email avec clé privée            │
│ 2. Ajoute en-tête DKIM-Signature                             │
│ 3. Publie clé publique dans DNS                              │
│ 4. Serveur destinataire vérifie signature                    │
│                                                              │
│ En-tête ajouté :                                             │
│ DKIM-Signature: v=1; a=rsa-sha256; d=example.com;            │
│   s=2024; h=from:to:subject:date;                            │
│   bh=base64hash; b=signaturebase64                           │
│                                                              │
│ DNS :                                                        │
│ 2024._domainkey.example.com.  TXT  "v=DKIM1; k=rsa; p=..."   │
│                                                              │
│ Avantages :                                                  │
│ ✓ Prouve identité expéditeur                                 │
│ ✓ Garantit intégrité message                                 │
│ ✓ Réputation domaine                                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│ DMARC (Domain-based Message Authentication)                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Rôle : Politique sur que faire si SPF/DKIM échouent          │
│                                                              │
│ Enregistrement DNS :                                         │
│ _dmarc.example.com.  TXT  "v=DMARC1; p=reject;               │
│   rua=mailto:dmarc@example.com; pct=100"                     │
│                                                              │
│ Paramètres :                                                 │
│ ├─ p=reject : Rejeter emails qui échouent                    │
│ │   (alternatives : none, quarantine)                        │
│ ├─ rua= : Adresse pour rapports agrégés                      │
│ ├─ ruf= : Adresse pour rapports forensiques                  │
│ └─ pct= : Pourcentage emails soumis à politique              │
│                                                              │
│ Chaîne de validation :                                       │
│ Email arrive → Vérif SPF → Vérif DKIM → Politique DMARC      │
│                                                              │
│ Si SPF ET DKIM échouent → Action selon DMARC                 │
│ ├─ none : Accepter quand même (monitoring)                   │
│ ├─ quarantine : Mettre en spam                               │
│ └─ reject : Rejeter complètement                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Format des emails (RFC 5322)

### Structure d'un email

```
┌──────────────────────────────────────────────────────────────┐
│ Anatomie d'un email                                          │
└──────────────────────────────────────────────────────────────┘

┌────────────────────────────────────┐
│ EN-TÊTES (Headers)                 │
│ ──────────────────                 │
│ Return-Path: <alice@example.com>   │
│ Received: from ... [details]       │
│ From: Alice <alice@example.com>    │
│ To: Bob <bob@company.org>          │
│ Cc: Charlie <charlie@company.org>  │
│ Subject: Réunion demain            │
│ Date: Wed, 6 Dec 2024 10:00 +0100  │
│ Message-ID: <abc@example.com>      │
│ MIME-Version: 1.0                  │
│ Content-Type: text/plain;          │
│   charset="UTF-8"                  │
├────────────────────────────────────┤
│ [Ligne vide obligatoire]           │
├────────────────────────────────────┤
│ CORPS (Body)                       │
│ ─────────────                      │
│ Bonjour Bob,                       │
│                                    │
│ N'oublie pas la réunion            │
│ de demain à 14h.                   │
│                                    │
│ Cordialement,                      │
│ Alice                              │
└────────────────────────────────────┘

Sections :
1. En-têtes : Métadonnées (qui, quand, quoi, comment)
2. Ligne vide : Séparateur obligatoire
3. Corps : Contenu du message
```

### En-têtes importants

```
┌──────────────────────────────────────────────────────────────┐
│ En-têtes email standards                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ OBLIGATOIRES                                                 │
│ ├─ From: Expéditeur (adresse + nom optionnel)                │
│ ├─ Date: Date d'envoi (format RFC 5322)                      │
│ └─ Message-ID: Identifiant unique                            │
│                                                              │
│ DESTINATAIRES                                                │
│ ├─ To: Destinataire(s) principal(aux)                        │
│ ├─ Cc: Copie carbone (Carbon Copy)                           │
│ └─ Bcc: Copie cachée (Blind Carbon Copy)                     │
│                                                              │
│ SUJET ET CONTENU                                             │
│ ├─ Subject: Sujet du message                                 │
│ ├─ MIME-Version: Version MIME (1.0)                          │
│ ├─ Content-Type: Type de contenu                             │
│ └─ Content-Transfer-Encoding: Encodage                       │
│                                                              │
│ CONVERSATION                                                 │
│ ├─ In-Reply-To: Message-ID du message parent                 │
│ ├─ References: Chaîne de Message-IDs                         │
│ └─ Thread-Index: Index de fil (Exchange)                     │
│                                                              │
│ ROUTAGE                                                      │
│ ├─ Received: Ajouté par chaque serveur (trace)               │
│ ├─ Return-Path: Adresse de retour (bounces)                  │
│ └─ Reply-To: Adresse de réponse (si différente)              │
│                                                              │
│ SÉCURITÉ                                                     │
│ ├─ DKIM-Signature: Signature cryptographique                 │
│ ├─ ARC-Authentication-Results: Résultats auth.               │
│ └─ Received-SPF: Résultat validation SPF                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### MIME et pièces jointes

**MIME (Multipurpose Internet Mail Extensions)** permet d'envoyer du contenu non-ASCII :

```
┌──────────────────────────────────────────────────────────────┐
│ Email MIME multipart avec pièce jointe                       │
└──────────────────────────────────────────────────────────────┘

From: Alice <alice@example.com>
To: Bob <bob@company.org>
Subject: Rapport Q4
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="----BOUNDARY1234"

------BOUNDARY1234
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

Bonjour Bob,

Voici le rapport du Q4 en pièce jointe.

Cordialement,
Alice

------BOUNDARY1234
Content-Type: application/pdf; name="rapport-q4.pdf"
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="rapport-q4.pdf"

JVBERi0xLjQKJcOkw7zDtsOfCjIgMCBvYmoKPDwvTGVuZ3RoIDMgMCBSL0ZpbHRl
ci9GbGF0ZURlY29kZT4+CnN0cmVhbQp4nGVOywoCMQy89yt6tmBJ0yZ7ExT/wKN/
[... contenu base64 du PDF ...]

------BOUNDARY1234--
```

**Types MIME courants :**
```
text/plain              : Texte brut
text/html               : HTML
image/jpeg              : Image JPEG
image/png               : Image PNG
application/pdf         : Document PDF
application/zip         : Archive ZIP
application/msword      : Word (.doc)
application/vnd.ms-excel: Excel (.xls)
multipart/mixed         : Contenu multiple (texte + pièces jointes)
multipart/alternative   : Versions alternatives (texte + HTML)
```

### Email HTML avec alternative texte

```
From: Newsletter <news@example.com>
To: bob@company.org
Subject: Newsletter Décembre
MIME-Version: 1.0
Content-Type: multipart/alternative; boundary="----ALT5678"

------ALT5678
Content-Type: text/plain; charset="UTF-8"

NEWSLETTER DÉCEMBRE

Nouveautés du mois :
- Produit A : -20%
- Produit B : Nouveau !

Visitez notre site : https://example.com

------ALT5678
Content-Type: text/html; charset="UTF-8"

<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; }
    h1 { color: #333; }
    .highlight { background: yellow; }
  </style>
</head>
<body>
  <h1>Newsletter Décembre</h1>
  <p>Nouveautés du mois :</p>
  <ul>
    <li><span class="highlight">Produit A : -20%</span></li>
    <li>Produit B : Nouveau !</li>
  </ul>
  <p><a href="https://example.com">Visitez notre site</a></p>
</body>
</html>

------ALT5678--

Principe :
├─ Client texte : Affiche version text/plain
└─ Client HTML : Affiche version text/html

Avantage : Compatibilité maximale
```

## Configuration client email

### Paramètres typiques

```
┌──────────────────────────────────────────────────────────────┐
│ Configuration client email (Exemple : Gmail)                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ INFORMATIONS COMPTE                                          │
│ ├─ Adresse email : bob@gmail.com                             │
│ ├─ Nom affiché : Bob Smith                                   │
│ └─ Mot de passe : ********                                   │
│                                                              │
│ SERVEUR RÉCEPTION (IMAP recommandé)                          │
│ ├─ Protocole : IMAP                                          │
│ ├─ Serveur : imap.gmail.com                                  │
│ ├─ Port : 993                                                │
│ ├─ Sécurité : SSL/TLS                                        │
│ └─ Authentication : Normale (login/password)                 │
│                                                              │
│ OU RÉCEPTION (POP3 si vraiment nécessaire)                   │
│ ├─ Protocole : POP3                                          │
│ ├─ Serveur : pop.gmail.com                                   │
│ ├─ Port : 995                                                │
│ ├─ Sécurité : SSL/TLS                                        │
│ └─ Laisser copie sur serveur : Oui (recommandé)              │
│                                                              │
│ SERVEUR ENVOI (SMTP)                                         │
│ ├─ Serveur : smtp.gmail.com                                  │
│ ├─ Port : 587 (ou 465)                                       │
│ ├─ Sécurité : STARTTLS (ou SSL/TLS si port 465)              │
│ ├─ Authentification : Oui (obligatoire)                      │
│ ├─ Username : bob@gmail.com                                  │
│ └─ Password : ******** (même que réception)                  │
│                                                              │
│ OPTIONS AVANCÉES                                             │
│ ├─ Vérifier nouveaux messages : Toutes les 5 min (ou IDLE)   │
│ ├─ Dossiers à synchroniser : Tous (IMAP)                     │
│ └─ Taille cache local : 100 MB                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

🔑 **SMTP pour ENVOYER, POP3/IMAP pour RECEVOIR**

🔑 **SMTP utilise MX records DNS pour trouver serveurs destinataires**

🔑 **POP3 = Télécharger et (optionnellement) supprimer du serveur**

🔑 **IMAP = Gestion sur serveur, synchronisation multi-appareils**

🔑 **IMAP est le standard moderne recommandé (95% des cas)**

🔑 **Toujours utiliser TLS/SSL : ports 587(SMTP), 993(IMAP), 995(POP3)**

🔑 **STARTTLS (port 587) recommandé pour SMTP soumission client**

🔑 **IMAP IDLE permet notifications push en temps réel**

🔑 **SPF, DKIM, DMARC protègent contre spoofing et spam**

🔑 **MIME permet pièces jointes et HTML dans emails**

---

## Ce que nous avons appris

Dans cette section, nous avons exploré :

- ✅ L'architecture complète du système email (MUA, MTA, MDA)
- ✅ SMTP : protocole d'envoi, commandes, codes réponse, authentification
- ✅ Le routage email via MX records DNS
- ✅ POP3 : téléchargement simple, états, commandes, UIDL
- ✅ IMAP : gestion avancée, dossiers, flags, synchronisation
- ✅ La comparaison détaillée POP3 vs IMAP et quand utiliser chacun
- ✅ La sécurité : STARTTLS, SSL/TLS, SPF, DKIM, DMARC
- ✅ Le format des emails : en-têtes, MIME, pièces jointes
- ✅ IMAP IDLE pour notifications push
- ✅ Configuration pratique des clients email

## Conclusion

L'email est un système complexe qui repose sur la coopération de trois protocoles majeurs. Bien que créé il y a plusieurs décennies, il reste extraordinairement robuste et continue d'évoluer avec des extensions modernes (STARTTLS, OAUTH2, IDLE...).

**Pour les utilisateurs modernes, IMAP est le choix évident** : synchronisation parfaite entre appareils, gestion puissante, et expérience fluide. POP3 ne devrait être utilisé que dans des cas très spécifiques.

**La sécurité est cruciale** : toute communication email moderne DOIT être chiffrée (TLS/SSL), et les mécanismes SPF/DKIM/DMARC sont essentiels pour lutter contre le spam et le phishing.

Malgré l'émergence de nouvelles formes de communication (messageries instantanées, réseaux sociaux), l'email reste le pilier de la communication professionnelle et ne montre aucun signe de déclin.

---

**Fin du module Protocoles de messagerie** 👉

⏭️ [SSH et Telnet](/05-couche-application/07-ssh-telnet.md)
