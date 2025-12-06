🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.7 SSH et Telnet

## Introduction

L'**accès distant** à un ordinateur ou un serveur est l'une des fonctionnalités les plus fondamentales de l'administration système. Que ce soit pour gérer un serveur web, administrer un routeur, ou simplement exécuter des commandes sur une machine distante, nous avons besoin de protocoles qui permettent de prendre le contrôle d'un système à distance.

**Telnet** et **SSH** sont les deux protocoles majeurs d'accès distant en ligne de commande :
- **Telnet** : Le protocole historique (1969), simple mais **totalement non sécurisé**
- **SSH** : Le standard moderne (1995), sécurisé par cryptographie

Dans cette section, nous allons explorer ces deux protocoles, comprendre pourquoi Telnet est obsolète et dangereux, et découvrir la puissance et la flexibilité de SSH qui l'a complètement remplacé dans les environnements modernes.

## Telnet (Telecommunication Network)

### Présentation et historique

```
┌──────────────────────────────────────────────────────────────┐
│ Telnet - Telecommunication Network                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Création : 1969 (l'un des tout premiers protocoles)          │
│ RFC : RFC 854 (1983)                                         │
│ Port : 23                                                    │
│ Transport : TCP                                              │
│ Sécurité : AUCUNE (texte clair total) ✗                      │
│                                                              │
│ Usage : Accès distant en ligne de commande                   │
│ Principe : Terminal virtuel sur machine distante             │
│                                                              │
│ Statut actuel : OBSOLÈTE et DANGEREUX                        │
│ Remplacé par : SSH (depuis ~2000)                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Contexte historique :**
```
1969 : Création de Telnet pour ARPANET
├─ Internet n'existe pas encore
├─ Réseau fermé, académique, confiance totale
├─ Sécurité n'est pas une préoccupation
└─ Objectif : Simplicité et interopérabilité

Problème :
Quand Internet s'ouvre au public (années 1990),
Telnet révèle ses failles critiques :
├─ Tout en texte clair (mots de passe visibles)
├─ Pas d'authentification du serveur
├─ Pas de protection contre interception
└─ Vulnérable à toutes les attaques réseau

Résultat : Abandon quasi-total au profit de SSH
```

### Fonctionnement de Telnet

```
┌──────────────────────────────────────────────────────────────┐
│ Architecture Telnet                                          │
└──────────────────────────────────────────────────────────────┘

Client Telnet                           Serveur Telnet
(192.168.1.42)                         (203.0.113.10:23)
      │                                        │
      │ Connexion TCP au port 23               │
      ├───────────────────────────────────────>│
      │                                        │
      │ Négociation options (NVT)              │
      │<──────────────────────────────────────>│
      │                                        │
      │ Login: bob                             │
      ├───────────────────────────────────────>│
      │ Password: ********                     │
      ├───────────────────────────────────────>│
      │                                        │
      │ Shell ouvert sur serveur               │
      │                                        │
      │ ls -la                                 │
      ├───────────────────────────────────────>│
      │ [Résultat commande]                    │
      │<───────────────────────────────────────┤
      │                                        │
      │ cat /etc/passwd                        │
      ├───────────────────────────────────────>│
      │ [Contenu fichier]                      │
      │<───────────────────────────────────────┤
      │                                        │
      │ exit                                   │
      ├───────────────────────────────────────>│
      │ Connexion fermée                       │
      │                                        │

TOUT est en TEXTE CLAIR !
Visible avec Wireshark ou tcpdump
```

### Session Telnet exemple

```
┌──────────────────────────────────────────────────────────────┐
│ Exemple de session Telnet                                    │
└──────────────────────────────────────────────────────────────┘

$ telnet 203.0.113.10
Trying 203.0.113.10...
Connected to 203.0.113.10.
Escape character is '^]'.

Ubuntu 20.04.3 LTS
server01 login: bob
Password: secret123

Last login: Wed Dec 06 10:00:00 from 192.168.1.42

bob@server01:~$ whoami
bob

bob@server01:~$ pwd
/home/bob

bob@server01:~$ ls -la
total 32
drwxr-xr-x 4 bob  bob  4096 Dec  6 10:00 .
drwxr-xr-x 3 root root 4096 Nov 15 09:00 ..
-rw------- 1 bob  bob   220 Nov 15 09:00 .bash_logout
-rw------- 1 bob  bob  3526 Nov 15 09:00 .bashrc
drwx------ 2 bob  bob  4096 Dec  5 14:30 .ssh
-rw-r--r-- 1 bob  bob   807 Nov 15 09:00 .profile

bob@server01:~$ cat important.txt
Données confidentielles...

bob@server01:~$ exit
logout
Connection closed by foreign host.
```

### Le problème critique de Telnet

```
┌──────────────────────────────────────────────────────────────┐
│ Capture réseau d'une session Telnet (Wireshark)              │
└──────────────────────────────────────────────────────────────┘

Paquet 1 :
Source: 192.168.1.42:50234 → Destination: 203.0.113.10:23
Data: "bob\r\n"
      ↑
      Login visible !

Paquet 2 :
Source: 192.168.1.42:50234 → Destination: 203.0.113.10:23
Data: "secret123\r\n"
      ↑
      Mot de passe EN CLAIR !

Paquet 3 :
Source: 192.168.1.42:50234 → Destination: 203.0.113.10:23
Data: "cat /etc/shadow\r\n"
      ↑
      Commandes visibles !

Paquet 4 :
Source: 203.0.113.10:23 → Destination: 192.168.1.42:50234
Data: "root:$6$encrypted...:18950:0:99999:7:::\r\n"
      ↑
      Réponses visibles !

┌──────────────────────────────────────────────────────────────┐
│ Conséquence : TOUT est interceptable                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Un attaquant sur le réseau peut :                            │
│ ✗ Voir tous les identifiants                                 │
│ ✗ Voir tous les mots de passe                                │
│ ✗ Voir toutes les commandes                                  │
│ ✗ Voir toutes les réponses                                   │
│ ✗ Modifier les paquets (man-in-the-middle)                   │
│ ✗ Se faire passer pour le serveur                            │
│                                                              │
│ Sécurité : ZÉRO                                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Telnet aujourd'hui : usages résiduels

```
┌──────────────────────────────────────────────────────────────┐
│ Rares cas où Telnet est encore acceptable                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ✓ Test de connectivité à un service                          │
│   └─ telnet smtp.example.com 25                              │
│   └─ Tester si un port est ouvert                            │
│   └─ JAMAIS pour authentification !                          │
│                                                              │
│ ✓ Accès à équipements anciens (derniers recours)             │
│   ├─ Vieux routeurs sans SSH                                 │
│   ├─ Équipements industriels legacy                          │
│   └─ UNIQUEMENT sur réseau local isolé                       │
│                                                              │
│ ✓ Protocole de configuration initial                         │
│   ├─ Première config d'un switch                             │
│   ├─ Console série → Telnet local                            │
│   └─ Puis activer SSH et DÉSACTIVER Telnet                   │
│                                                              │
│ ✗ JAMAIS pour :                                              │
│   ├─ Accès distant via Internet                              │
│   ├─ Administration de serveurs                              │
│   ├─ Tout usage nécessitant authentification                 │
│   └─ Production moderne                                      │
│                                                              │
│ RÈGLE : Si SSH est disponible, UTILISEZ SSH !                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## SSH (Secure Shell)

### Présentation

```
┌──────────────────────────────────────────────────────────────┐
│ SSH - Secure Shell                                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Création : 1995 (SSH-1), 2006 (SSH-2, version actuelle)      │
│ Créateur : Tatu Ylönen (Finlande)                            │
│ RFC : RFC 4251-4254 (SSH-2)                                  │
│ Port : 22                                                    │
│ Transport : TCP                                              │
│ Sécurité : TOTALE (chiffrement fort) ✓✓✓                     │
│                                                              │
│ Usage : Accès distant sécurisé en ligne de commande          │
│ Principe : Tunnel chiffré + authentification forte           │
│                                                              │
│ Implémentations principales :                                │
│ ├─ OpenSSH (open source, le plus utilisé)                    │
│ ├─ PuTTY (Windows)                                           │
│ └─ Dropbear (embarqué, léger)                                │
│                                                              │
│ Statut : STANDARD UNIVERSEL depuis ~2000                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Pourquoi SSH a été créé ?**
```
1995 : Attaque par sniffing à l'Université d'Helsinki
├─ Tatu Ylönen découvre que des mots de passe
│   circulent en clair sur le réseau (Telnet, FTP...)
├─ Décide de créer un protocole sécurisé
├─ Développe SSH-1 en quelques mois
└─ Le rend open source

Adoption fulgurante :
├─ 1995-1996 : Utilisé par 20 000 utilisateurs
├─ 1999 : SSH-2 (protocole refondu, plus sûr)
├─ 2000s : Remplace Telnet massivement
└─ Aujourd'hui : Standard de facto universel

SSH a sauvé la sécurité d'Internet !
```

### Architecture SSH

```
┌──────────────────────────────────────────────────────────────┐
│ Couches du protocole SSH                                     │
└──────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────┐
│ SSH-USERAUTH (Couche Authentification)             │
│ ├─ Authentification par mot de passe               │
│ ├─ Authentification par clé publique               │
│ └─ Autres méthodes (GSSAPI, keyboard-interactive)  │
└────────────────────────────────────────────────────┘
                       ↓
┌────────────────────────────────────────────────────┐
│ SSH-CONNECTION (Couche Connexion)                  │
│ ├─ Canaux (channels) multiples                     │
│ ├─ Session shell interactive                       │
│ ├─ Exécution commande unique                       │
│ ├─ Port forwarding (tunneling)                     │
│ └─ Transfert X11                                   │
└────────────────────────────────────────────────────┘
                       ↓
┌────────────────────────────────────────────────────┐
│ SSH-TRANSPORT (Couche Transport)                   │
│ ├─ Établissement connexion chiffrée                │
│ ├─ Échange de clés (Diffie-Hellman)                │
│ ├─ Chiffrement symétrique (AES, ChaCha20)          │
│ ├─ Intégrité (HMAC-SHA2)                           │
│ └─ Compression optionnelle                         │
└────────────────────────────────────────────────────┘
                       ↓
┌────────────────────────────────────────────────────┐
│ TCP (Port 22)                                      │
└────────────────────────────────────────────────────┘
```

### Établissement d'une connexion SSH

```
┌──────────────────────────────────────────────────────────────┐
│ Processus complet de connexion SSH                           │
└──────────────────────────────────────────────────────────────┘

Client                                   Serveur
  │                                         │
  │ 1. CONNEXION TCP                        │
  ├───────────────────────────────────────> │
  │ SYN / SYN-ACK / ACK (port 22)           │
  │                                         │
  │ 2. ÉCHANGE DE VERSIONS                  │
  ├───────────────────────────────────────> │
  │ "SSH-2.0-OpenSSH_8.9"                   │
  │<─────────────────────────────────────── ┤
  │ "SSH-2.0-OpenSSH_8.9"                   │
  │                                         │
  │ 3. ÉCHANGE DE CLÉS (Key Exchange)       │
  │    ─────────────────────────────        │
  │ • Algorithmes supportés                 │
  │<──────────────────────────────────────> │
  │ • Diffie-Hellman échange                │
  │<──────────────────────────────────────> │
  │ • Génération clé de session             │
  │   → Clé symétrique partagée ✓           │
  │                                         │
  │ 4. VÉRIFICATION SERVEUR                 │
  │    ────────────────────                 │
  │ • Serveur envoie sa clé publique        │
  │<─────────────────────────────────────── ┤
  │ • Client vérifie empreinte (fingerprint)│
  │   "ECDSA key fingerprint is SHA256:..." │
  │   Connu ? Oui ✓ / Non → Demander user   │
  │                                         │
  │ ═══════════════════════════════════     │
  │ CANAL MAINTENANT CHIFFRÉ                │
  │ ═══════════════════════════════════     │
  │                                         │
  │ 5. AUTHENTIFICATION CLIENT              │
  │    ───────────────────────              │
  │ Option A : Mot de passe                 │
  │ "bob" + "password123" (chiffrés) ──────>│
  │                                         │
  │ Option B : Clé publique (préféré)       │
  │ "bob" + signature avec clé privée ─────>│
  │ Serveur vérifie avec clé publique ✓     │
  │                                         │
  │ 6. SESSION ÉTABLIE                      │
  │    ──────────────                       │
  │ Shell ouvert sur serveur ✓              │
  │ Toutes les communications chiffrées     │
  │                                         │

Durée totale : ~200-500ms
Sécurité : MAXIMALE ✓
```

### Session SSH interactive

```
┌──────────────────────────────────────────────────────────────┐
│ Exemple de connexion SSH                                     │
└──────────────────────────────────────────────────────────────┘

$ ssh bob@example.com
The authenticity of host 'example.com (203.0.113.10)' can't be established.
ECDSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'example.com' (ECDSA) to the list of known hosts.
bob@example.com's password: ********

Welcome to Ubuntu 22.04.3 LTS
Last login: Wed Dec 06 10:00:00 2024 from 192.168.1.42

bob@server:~$ whoami
bob

bob@server:~$ uname -a
Linux server 5.15.0-91-generic #101-Ubuntu SMP x86_64 GNU/Linux

bob@server:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   12G   36G  25% /
/dev/sda2       100G   45G   50G  47% /home

bob@server:~$ exit
logout
Connection to example.com closed.
```

### Avec clé SSH (pas de mot de passe)

```
$ ssh bob@example.com
Welcome to Ubuntu 22.04.3 LTS
Last login: Wed Dec 06 11:00:00 2024 from 192.168.1.42

bob@server:~$ █

Aucun mot de passe demandé !
Authentification par clé publique ✓
Plus sécurisé ET plus pratique
```

## Authentification SSH

### Méthode 1 : Par mot de passe

```
┌──────────────────────────────────────────────────────────────┐
│ Authentification par mot de passe                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Processus :                                                  │
│ 1. Client envoie username                                    │
│ 2. Client envoie password (CHIFFRÉ via tunnel SSH)           │
│ 3. Serveur vérifie dans /etc/shadow                          │
│ 4. Si OK → Session ouverte ✓                                 │
│                                                              │
│ Avantages :                                                  │
│ ✓ Simple à configurer                                        │
│ ✓ Pas de gestion de clés                                     │
│ ✓ Fonctionne immédiatement                                   │
│                                                              │
│ Inconvénients :                                              │
│ ✗ Vulnérable aux attaques brute-force                        │
│ ✗ Nécessite mémoriser/taper mot de passe                     │
│ ✗ Pas pratique pour automatisation                           │
│ ✗ Impossible de révoquer sans changer mot de passe           │
│                                                              │
│ Sécurité :                                                   │
│ ⚠️ Acceptable mais moins sûr que clés                        │
│ ⚠️ DOIT être combiné avec :                                  │
│    ├─ Mots de passe forts (12+ caractères)                   │
│    ├─ Fail2ban (bloque après N échecs)                       │
│    └─ Restriction IP (AllowUsers, AllowGroups)               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Méthode 2 : Par clé publique (recommandée)

```
┌──────────────────────────────────────────────────────────────┐
│ Authentification par clé publique (cryptographie asymétrique)│
└──────────────────────────────────────────────────────────────┘

Principe :
├─ Client possède paire de clés : privée + publique
├─ Clé publique copiée sur serveur
├─ Clé privée reste TOUJOURS sur client (SECRÈTE)
└─ Serveur défie client de prouver possession clé privée

┌────────────────┐              ┌────────────────┐
│    CLIENT      │              │    SERVEUR     │
│                │              │                │
│ ┌────────────┐ │              │ ┌────────────┐ │
│ │Clé privée  │ │              │ │Clé publique│ │
│ │ (secrète)  │ │              │ │  de Bob    │ │
│ │id_ed25519  │ │              │ │            │ │
│ └────────────┘ │              │ │authorized_ │ │
│                │              │ │   keys     │ │
│ ┌────────────┐ │              │ └────────────┘ │
│ │Clé publique│ │─────────────>│                │
│ │            │ │  (copiée une │                │
│ │id_ed25519  │ │   fois)      │                │
│ │   .pub     │ │              │                │
│ └────────────┘ │              │                │
└────────────────┘              └────────────────┘

Processus d'authentification :
1. Client : "Je suis bob"
2. Serveur : "Prouve-le ! Voici un challenge aléatoire : X"
3. Client : Signe X avec sa clé privée → signature
4. Serveur : Vérifie signature avec clé publique de bob
5. Si signature valide → Authentifié ✓
```

**Génération de clés SSH :**

```bash
┌──────────────────────────────────────────────────────────────┐
│ Création d'une paire de clés SSH                             │
└──────────────────────────────────────────────────────────────┘

$ ssh-keygen -t ed25519 -C "bob@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/bob/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase): ********
Enter same passphrase again: ********

Your identification has been saved in /home/bob/.ssh/id_ed25519
Your public key has been saved in /home/bob/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8 bob@example.com
The key's randomart image is:
+--[ED25519 256]--+
|     .o.o.       |
|    .. =.o       |
|   .  + B.       |
|    .  X.o       |
|     .S.=.o      |
|    .ooo==.      |
|   ..o.o*+E      |
|    o.++oo..     |
|     .=o.o..     |
+----[SHA256]-----+

Fichiers créés :
├─ /home/bob/.ssh/id_ed25519     (CLÉ PRIVÉE - SECRÈTE !)
│   └─ Permissions : 600 (lecture seule propriétaire)
└─ /home/bob/.ssh/id_ed25519.pub (clé publique)
    └─ Peut être partagée librement

Types de clés disponibles :
├─ ed25519 : Recommandé (moderne, rapide, sûr) ✓✓
├─ rsa (4096 bits) : Standard, compatible
├─ ecdsa : Bon mais moins que ed25519
└─ dsa : Obsolète, à éviter ✗

Passphrase :
├─ Protection supplémentaire de la clé privée
├─ Même si clé volée, inutilisable sans passphrase
└─ Recommandé pour clés importantes ✓
```

**Copie de la clé publique sur le serveur :**

```bash
┌──────────────────────────────────────────────────────────────┐
│ Méthode 1 : ssh-copy-id (automatique)                        │
└──────────────────────────────────────────────────────────────┘

$ ssh-copy-id bob@example.com
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed:
                      "/home/bob/.ssh/id_ed25519.pub"
bob@example.com's password: ********

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bob@example.com'"
and check to make sure that only the key(s) you wanted were added.

┌──────────────────────────────────────────────────────────────┐
│ Méthode 2 : Manuelle                                         │
└──────────────────────────────────────────────────────────────┘

# Sur le client : afficher clé publique
$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOMqqnkVzrm0SdG6UOoqKLsabgH5C9okWi0dh2l9GKJl bob@example.com

# Se connecter au serveur et ajouter à authorized_keys
$ ssh bob@example.com
bob@example.com's password: ********

bob@server:~$ mkdir -p ~/.ssh
bob@server:~$ chmod 700 ~/.ssh
bob@server:~$ echo "ssh-ed25519 AAAAC3N..." >> ~/.ssh/authorized_keys
bob@server:~$ chmod 600 ~/.ssh/authorized_keys
bob@server:~$ exit

# Tester connexion sans mot de passe
$ ssh bob@example.com
Welcome to Ubuntu...
bob@server:~$ █

✓ Connexion réussie sans mot de passe !
```

**Structure du fichier authorized_keys :**

```bash
# /home/bob/.ssh/authorized_keys sur le serveur

# Clé simple
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOMqqnkVzrm... bob@laptop

# Clé avec restrictions
from="192.168.1.0/24" ssh-ed25519 AAAAC3NzaC1lZDI... bob@work

# Clé avec commande forcée (pour automatisation)
command="/usr/local/bin/backup.sh" ssh-ed25519 AAAAC3... backup-script

# Clé avec options multiples
no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-ed25519 AAAAC3... restricted

Options disponibles :
├─ from="pattern" : Restreint IPs autorisées
├─ command="cmd" : Force exécution commande spécifique
├─ no-port-forwarding : Désactive tunneling
├─ no-X11-forwarding : Désactive X11
├─ no-agent-forwarding : Désactive agent forwarding
└─ environment="VAR=value" : Variables d'environnement
```

### Avantages des clés SSH

```
┌──────────────────────────────────────────────────────────────┐
│ Pourquoi les clés SSH sont supérieures                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ✓✓ SÉCURITÉ                                                  │
│   ├─ Impossible à deviner par brute-force                    │
│   ├─ 256 bits = 2^256 possibilités (inviolable)              │
│   └─ Résistant aux attaques quantiques (ed25519)             │
│                                                              │
│ ✓✓ AUTOMATISATION                                            │
│   ├─ Scripts, cron, CI/CD sans interaction                   │
│   ├─ Pas de prompt mot de passe                              │
│   └─ Gestion centralisée possible                            │
│                                                              │
│ ✓✓ GESTION                                                   │
│   ├─ Plusieurs clés pour différents usages                   │
│   ├─ Révocation facile (retirer de authorized_keys)          │
│   ├─ Audit : Qui a accès ? (liste des clés)                  │
│   └─ Expiration possible via certificats SSH                 │
│                                                              │
│ ✓✓ PRATIQUE                                                  │
│   ├─ Connexion sans taper mot de passe                       │
│   ├─ SSH agent mémorise clé déverrouillée                    │
│   └─ Peut authentifier vers 100 serveurs avec 1 clé          │
│                                                              │
│ ✓✓ TRAÇABILITÉ                                               │
│   ├─ Commentaire identifie source (bob@laptop)               │
│   ├─ Logs indiquent quelle clé utilisée                      │
│   └─ Audit trail complet                                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Fonctionnalités avancées SSH

### Tunneling SSH (Port Forwarding)

SSH peut créer des **tunnels sécurisés** pour d'autres protocoles :

#### Port Forwarding Local (Local Port Forwarding)

```
┌──────────────────────────────────────────────────────────────┐
│ Port Forwarding Local                                        │
└──────────────────────────────────────────────────────────────┘

Problème :
Serveur web interne (192.168.10.50:80) non accessible depuis Internet
Mais vous avez SSH vers serveur jump (203.0.113.10)

Solution :
$ ssh -L 8080:192.168.10.50:80 user@203.0.113.10

Signification :
-L [local_port]:[destination_host]:[destination_port]

Résultat :

Votre machine                Jump Server              Serveur Web
(localhost)                 (203.0.113.10)         (192.168.10.50)
     │                            │                       │
     │ Navigateur → localhost:8080│                       │
     ├───────────────────────────>│                       │
     │        Tunnel SSH chiffré  │                       │
     │                            ├──────────────────────>│
     │                            │ HTTP non chiffré      │
     │                            │ (réseau interne OK)   │
     │                            │<──────────────────────┤
     │<───────────────────────────┤                       │
     │                            │                       │

Utilisation :
1. Exécuter commande SSH avec -L
2. Ouvrir navigateur : http://localhost:8080
3. Trafic passe par tunnel SSH vers 192.168.10.50:80

Cas d'usage :
✓ Accès base de données interne
✓ Interface admin non exposée
✓ Contournement pare-feu sortant
✓ Sécurisation protocole non chiffré
```

#### Port Forwarding Remote (Remote Port Forwarding)

```
┌──────────────────────────────────────────────────────────────┐
│ Port Forwarding Remote (Reverse Tunnel)                      │
└──────────────────────────────────────────────────────────────┘

Problème :
Vous êtes derrière NAT, serveur externe veut accéder à votre service local

Solution :
$ ssh -R 8080:localhost:80 user@public-server.com

Signification :
-R [remote_port]:[local_host]:[local_port]

Résultat :

Votre machine            Serveur Public           Utilisateur externe
(derrière NAT)           (public-server.com)
     │                        │                         │
     │<───────────────────────┤                         │
     │   Tunnel SSH           │                         │
     │                        │<────────────────────────┤
     │                        │ Connexion à :8080       │
     │                        │                         │
     │<───────────────────────┤                         │
     │   Requête transférée   │                         │
     │   via tunnel           │                         │

Port 8080 sur serveur public → redirigé vers localhost:80 chez vous

Cas d'usage :
✓ Démo service local à client distant
✓ Webhook vers machine de dev
✓ Accès temporaire sans VPN
✓ Contournement NAT
```

#### Tunnel Dynamique (SOCKS Proxy)

```
┌──────────────────────────────────────────────────────────────┐
│ Dynamic Port Forwarding (SOCKS Proxy)                        │
└──────────────────────────────────────────────────────────────┘

$ ssh -D 1080 user@jump-server.com

Signification :
-D [local_port] : Crée un proxy SOCKS sur port local

Résultat :
SSH crée un proxy SOCKS5 sur localhost:1080

Configuration navigateur :
├─ Type : SOCKS5
├─ Hôte : localhost
├─ Port : 1080
└─ Tout le trafic passe par tunnel SSH

Votre machine          Jump Server            Internet
     │                      │                     │
     │ Navigateur           │                     │
     │ via SOCKS:1080       │                     │
     ├─────────────────────>│                     │
     │   Tunnel SSH         │                     │
     │                      ├────────────────────>│
     │                      │ Requête HTTP        │
     │                      │<────────────────────┤
     │<─────────────────────┤ Réponse             │

Avantages :
✓ Tout le trafic navigateur chiffré
✓ Contourne censure/filtrage
✓ Change adresse IP apparente
✓ Protection WiFi public

Cas d'usage :
✓ Navigation sécurisée sur WiFi public
✓ Accès ressources régionales
✓ Contournement restrictions réseau
✓ VPN léger DIY
```

### Exécution de commandes à distance

```bash
┌──────────────────────────────────────────────────────────────┐
│ Commandes SSH sans session interactive                       │
└──────────────────────────────────────────────────────────────┘

# Exécuter une commande unique
$ ssh user@server 'uname -a'
Linux server 5.15.0-91-generic #101-Ubuntu SMP x86_64 GNU/Linux

# Commande avec pipe local
$ ssh user@server 'cat /var/log/syslog' | grep error

# Commandes multiples
$ ssh user@server 'cd /var/www && ls -la && df -h'

# Redirection de sortie locale
$ ssh user@server 'mysqldump -u root database' > backup.sql

# Script distant
$ ssh user@server 'bash -s' < local-script.sh

# Script avec arguments
$ ssh user@server 'bash -s' < script.sh arg1 arg2

# Commande sudo (attention : mot de passe)
$ ssh -t user@server 'sudo systemctl restart nginx'
       ↑
       Option -t : Force allocation pseudo-terminal (requis pour sudo)

Cas d'usage :
✓ Automatisation (cron, scripts)
✓ Monitoring distant
✓ Déploiement applications
✓ Administration batch
✓ Orchestration serveurs
```

### Transfert de fichiers via SSH

**SCP (Secure Copy) :**

```bash
┌──────────────────────────────────────────────────────────────┐
│ SCP - Secure Copy Protocol                                   │
└──────────────────────────────────────────────────────────────┘

# Copier fichier local → distant
$ scp file.txt user@server:/path/to/destination/

# Copier fichier distant → local
$ scp user@server:/path/to/file.txt ./local-dir/

# Copier répertoire récursivement
$ scp -r /local/dir user@server:/remote/dir/

# Avec port SSH non standard
$ scp -P 2222 file.txt user@server:/path/

# Préserver timestamps et permissions
$ scp -p file.txt user@server:/path/

# Verbose (debug)
$ scp -v file.txt user@server:/path/

# Limite bande passante (1000 Kbit/s)
$ scp -l 1000 bigfile.iso user@server:/path/

Exemple pratique :
$ scp report.pdf bob@server.com:~/Documents/
report.pdf                    100% 2048KB   2.0MB/s   00:01
```

**SFTP (déjà couvert dans section précédente) :**
```bash
# SFTP = Sous-système SSH dédié aux fichiers
$ sftp user@server
sftp> ls
sftp> get remote-file.txt
sftp> put local-file.txt
sftp> exit

Avantages SFTP vs SCP :
✓ Session interactive
✓ Navigation dans répertoires
✓ Reprise de transfert
✓ Plus moderne
```

**rsync over SSH :**

```bash
┌──────────────────────────────────────────────────────────────┐
│ rsync - Synchronisation intelligente via SSH                 │
└──────────────────────────────────────────────────────────────┘

# Synchroniser répertoire
$ rsync -avz -e ssh /local/dir/ user@server:/remote/dir/

Options :
├─ -a : Archive (préserve tout : permissions, timestamps, etc.)
├─ -v : Verbose
├─ -z : Compression
├─ -e ssh : Utiliser SSH comme transport
├─ --delete : Supprimer fichiers n'existant plus en source
└─ --progress : Afficher progression

# Sauvegarde incrémentielle
$ rsync -avz --delete /home/bob/ backup@server:/backups/bob/

# Dry-run (simulation, pas de modification)
$ rsync -avz --dry-run /source/ user@server:/dest/

# Exclure fichiers
$ rsync -avz --exclude='*.tmp' --exclude='.git' /project/ user@server:/project/

Avantages rsync :
✓✓ Transfère seulement les différences (rapide)
✓✓ Reprise automatique si interruption
✓✓ Préservation complète métadonnées
✓✓ Idéal pour backups et synchro
```

### Transfert X11 (Affichage graphique distant)

```bash
┌──────────────────────────────────────────────────────────────┐
│ X11 Forwarding - Applications graphiques via SSH             │
└──────────────────────────────────────────────────────────────┘

# Activer X11 forwarding
$ ssh -X user@server

# ou -Y (trusted, moins sécurisé mais plus compatible)
$ ssh -Y user@server

# Lancer application graphique
user@server:~$ firefox &
user@server:~$ gimp &
user@server:~$ gedit document.txt &

→ Fenêtres s'affichent sur votre écran local !
→ Mais l'application s'exécute sur le serveur distant
→ Tout chiffré via tunnel SSH

Configuration requise :

Serveur (/etc/ssh/sshd_config) :
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes

Client :
Serveur X11 installé :
├─ Linux : Déjà présent (X.org)
├─ macOS : XQuartz requis
└─ Windows : Xming, VcXsrv, ou WSL2

Limitations :
⚠️ Latence réseau visible (surtout animations)
⚠️ Bande passante consommée
⚠️ Mieux pour outils admin que jeux/vidéos
```

### SSH Agent

```bash
┌──────────────────────────────────────────────────────────────┐
│ SSH Agent - Gestion des clés en mémoire                      │
└──────────────────────────────────────────────────────────────┘

Problème :
Clé SSH avec passphrase → demandée à chaque connexion
Solution : SSH Agent mémorise clé déverrouillée

# Démarrer agent (généralement auto au login)
$ eval "$(ssh-agent -s)"
Agent pid 12345

# Ajouter clé à l'agent
$ ssh-add ~/.ssh/id_ed25519
Enter passphrase for /home/bob/.ssh/id_ed25519: ********
Identity added: /home/bob/.ssh/id_ed25519 (bob@laptop)

# Lister clés chargées
$ ssh-add -l
256 SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8 bob@laptop (ED25519)

# Supprimer clés de l'agent
$ ssh-add -D
All identities removed.

# Connexion SSH utilise automatiquement clé de l'agent
$ ssh server1
$ ssh server2
$ ssh server3
→ Pas de passphrase redemandée ! ✓

Agent Forwarding :
$ ssh -A user@jump-server
user@jump:~$ ssh internal-server
→ Agent local forwarded, pas besoin de clé sur jump-server

⚠️ Sécurité : Ne pas forwarder agent vers serveurs non fiables
```

## Configuration SSH

### Configuration client (~/.ssh/config)

```bash
┌──────────────────────────────────────────────────────────────┐
│ ~/.ssh/config - Configuration client SSH                     │
└──────────────────────────────────────────────────────────────┘

# Exemple de configuration
# ~/.ssh/config

# Valeurs par défaut globales
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    ForwardAgent no

# Serveur de production
Host prod
    HostName server.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_prod
    ForwardAgent no

# Serveur de dev (raccourci)
Host dev
    HostName dev.internal.com
    User bob
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 3306 localhost:3306

# Jump host (bastion)
Host bastion
    HostName bastion.company.com
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519_bastion

# Serveurs derrière bastion (ProxyJump)
Host internal-*
    ProxyJump bastion
    User bob

Host internal-db
    HostName 10.0.1.50

Host internal-web
    HostName 10.0.1.51

# GitHub
Host github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes

Usage :
$ ssh prod              → Se connecte à server.example.com:2222
$ ssh dev               → Se connecte à dev.internal.com (+ forward port 3306)
$ ssh internal-db       → Via bastion automatiquement
$ ssh internal-web      → Via bastion automatiquement

Avantages :
✓ Noms courts et mémorisables
✓ Configuration centralisée
✓ Pas besoin de se souvenir des détails
✓ Partage facile (versionner le fichier)
```

### Configuration serveur (/etc/ssh/sshd_config)

```bash
┌──────────────────────────────────────────────────────────────┐
│ /etc/ssh/sshd_config - Configuration serveur SSH             │
└──────────────────────────────────────────────────────────────┘

# Configuration sécurisée recommandée

# Port SSH (changer si exposé Internet)
Port 22
# Ou port non-standard : Port 2222

# Écouter sur adresses spécifiques
#ListenAddress 0.0.0.0
ListenAddress 203.0.113.10

# Protocole SSH version 2 uniquement (sécurité)
Protocol 2

# Clés hôte (identité du serveur)
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key

# Authentification
PermitRootLogin no              # ✓✓ Jamais login root direct
PubkeyAuthentication yes        # ✓✓ Activer clés publiques
PasswordAuthentication no       # ✓ Désactiver mots de passe (après setup clés)
PermitEmptyPasswords no         # ✓✓ Jamais de mots de passe vides
ChallengeResponseAuthentication no

# Restreindre utilisateurs
AllowUsers bob alice deploy
# Ou groupes
AllowGroups sshusers admins

# Limites de connexion
MaxAuthTries 3                  # Max 3 tentatives
MaxSessions 10                  # Max 10 sessions simultanées
LoginGraceTime 30               # 30s pour s'authentifier

# Sécurité supplémentaire
X11Forwarding no                # Désactiver si pas nécessaire
PermitTunnel no                 # Désactiver tunneling si pas nécessaire
AllowAgentForwarding no         # Désactiver agent forwarding par défaut
AllowTcpForwarding no           # Désactiver port forwarding par défaut

# Keep-alive
ClientAliveInterval 300         # Ping client toutes les 5 min
ClientAliveCountMax 2           # Déconnexion après 2 pings sans réponse

# Logging
SyslogFacility AUTH
LogLevel VERBOSE                # Logs détaillés pour audit

# Bannière (message d'avertissement)
Banner /etc/ssh/banner.txt

# Appliquer changements :
$ sudo systemctl restart sshd
```

### Sécurisation SSH

```
┌──────────────────────────────────────────────────────────────┐
│ Checklist sécurité SSH                                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ✓✓ ESSENTIEL (à faire immédiatement)                         │
│ ├─ Désactiver login root (PermitRootLogin no)                │
│ ├─ Utiliser clés publiques (PubkeyAuthentication yes)        │
│ ├─ Désactiver mots de passe (PasswordAuthentication no)      │
│ ├─ Utiliser SSH-2 uniquement (Protocol 2)                    │
│ └─ Mettre à jour OpenSSH régulièrement                       │
│                                                              │
│ ✓ RECOMMANDÉ                                                 │
│ ├─ Changer port (Port 2222) si exposé Internet               │
│ ├─ Restreindre utilisateurs (AllowUsers/AllowGroups)         │
│ ├─ fail2ban : Ban IP après échecs répétés                    │
│ ├─ Firewall : Limiter IPs autorisées                         │
│ ├─ Clés ed25519 (meilleures que RSA)                         │
│ └─ Passphrase sur clés privées                               │
│                                                              │
│ ✓ AVANCÉ                                                     │
│ ├─ Authentification 2FA (Google Authenticator)               │
│ ├─ Certificats SSH (au lieu de clés simples)                 │
│ ├─ Bastion host (serveur jump dédié)                         │
│ ├─ SSH audit logs centralisés                                │
│ └─ Surveillance anomalies (SIEM)                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### fail2ban pour SSH

```bash
┌──────────────────────────────────────────────────────────────┐
│ fail2ban - Protection contre attaques brute-force            │
└──────────────────────────────────────────────────────────────┘

Installation :
$ sudo apt install fail2ban

Configuration (/etc/fail2ban/jail.local) :
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3                # Ban après 3 échecs
findtime = 600              # Dans les 10 dernières minutes
bantime = 3600              # Ban pour 1 heure
# bantime = -1              # Ban permanent

Action :
1. Client échoue authentification 3× en 10 minutes
2. fail2ban détecte pattern dans logs
3. Ajoute règle firewall : DROP pour cette IP
4. Client banni pendant 1 heure (ou permanent)

Commandes utiles :
$ sudo fail2ban-client status sshd
$ sudo fail2ban-client set sshd unbanip 203.0.113.50
$ sudo tail -f /var/log/fail2ban.log

Résultat : Brute-force deviennent inefficaces ✓
```

## Comparaison Telnet vs SSH

```
┌───────────────────────────────────────────────────────────────────────┐
│ Telnet vs SSH - Comparaison complète                                  │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ Critère              │ Telnet               │ SSH                     │
│──────────────────────┼──────────────────────┼──────────────────────── │
│ Sécurité             │ Aucune ✗✗✗           │ Totale ✓✓✓              │
│ Chiffrement          │ Non                  │ Oui (AES, ChaCha20)     │
│ Auth. serveur        │ Non                  │ Oui (clé hôte)          │
│ Auth. client         │ Mot de passe clair   │ Clé publique/password   │
│ Intégrité données    │ Non                  │ Oui (HMAC)              │
│ Port défaut          │ 23                   │ 22                      │
│ Année création       │ 1969                 │ 1995                    │
│ Complexité           │ Simple               │ Moyenne                 │
│ Performance          │ Légèrement meilleure │ Excellente              │
│ Overhead             │ Minimal              │ Faible (~10%)           │
│ Fonctionnalités      │ Terminal uniquement  │ Terminal + tunnels +    │
│                      │                      │ transfert fichiers      │
│ Compression          │ Non                  │ Oui (optionnelle)       │
│ Port forwarding      │ Non                  │ Oui ✓                   │
│ Agent forwarding     │ Non                  │ Oui ✓                   │
│ X11 forwarding       │ Non                  │ Oui ✓                   │
│ Transfert fichiers   │ Non                  │ Oui (SCP, SFTP) ✓       │
│ Multi-canal          │ Non                  │ Oui ✓                   │
│ Session persistence  │ Non                  │ Possible (mosh, tmux)   │
│ Vulnérabilités       │ Toutes attaques ✗    │ Rares, patchées ✓       │
│ Sniffing réseau      │ Trivial ✗            │ Impossible ✓            │
│ MITM attack          │ Facile ✗             │ Difficile ✓             │
│ Replay attack        │ Possible ✗           │ Impossible ✓            │
│ Usage moderne        │ Obsolète ✗           │ Standard universel ✓    │
│ Compliance           │ Interdit (PCI-DSS)   │ Requis ✓                │
│                                                                       │
│ VERDICT :                                                             │
│ Telnet : ABANDONNÉ depuis ~2000, dangereux                            │
│ SSH : STANDARD UNIVERSEL, seul choix acceptable                       │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

## Cas d'usage SSH

```
┌──────────────────────────────────────────────────────────────┐
│ Cas d'usage typiques SSH                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ ✓✓ ADMINISTRATION SERVEURS                                   │
│   ├─ Gestion serveurs Linux/Unix                             │
│   ├─ Maintenance à distance                                  │
│   ├─ Déploiement applications                                │
│   └─ Monitoring et diagnostics                               │
│                                                              │
│ ✓✓ DÉVELOPPEMENT                                             │
│   ├─ Accès environnements de dev/test                        │
│   ├─ Git over SSH (GitHub, GitLab)                           │
│   ├─ Édition fichiers distants (VS Code Remote)              │
│   └─ Débogage applications distantes                         │
│                                                              │
│ ✓✓ AUTOMATISATION                                            │
│   ├─ Scripts déploiement (CI/CD)                             │
│   ├─ Tâches cron distantes                                   │
│   ├─ Orchestration (Ansible)                                 │
│   └─ Backups automatisés                                     │
│                                                              │
│ ✓✓ TRANSFERT FICHIERS                                        │
│   ├─ SCP : Copies rapides                                    │
│   ├─ SFTP : Transferts interactifs                           │
│   ├─ rsync : Synchronisation/backups                         │
│   └─ Alternative FTP sécurisée                               │
│                                                              │
│ ✓✓ TUNNELING                                                 │
│   ├─ Accès bases de données internes                         │
│   ├─ VPN léger (SOCKS proxy)                                 │
│   ├─ Contournement pare-feu                                  │
│   └─ Sécurisation protocoles non chiffrés                    │
│                                                              │
│ ✓ RÉSEAU                                                     │
│   ├─ Bastion/Jump host                                       │
│   ├─ Configuration routeurs/switches                         │
│   ├─ VPN site-to-site                                        │
│   └─ Reverse tunnels (accès NAT)                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

🔑 **Telnet est OBSOLÈTE et DANGEREUX - tout en texte clair**

🔑 **SSH est le standard universel pour accès distant sécurisé**

🔑 **SSH chiffre TOUT : authentification, commandes, données**

🔑 **Authentification par clé publique > mot de passe (sécurité + pratique)**

🔑 **Clés ed25519 recommandées (modernes, rapides, sûres)**

🔑 **SSH permet tunneling (port forwarding) pour sécuriser autres protocoles**

🔑 **Configuration serveur cruciale : désactiver root, mots de passe, utiliser clés**

🔑 **fail2ban protège contre attaques brute-force**

🔑 **SSH Agent mémorise clés déverrouillées (pratique)**

🔑 **SFTP et SCP pour transfert fichiers sécurisé via SSH**

---

## Ce que nous avons appris

Dans cette section, nous avons exploré :

- ✅ Telnet : protocole historique, totalement non sécurisé, obsolète
- ✅ SSH : architecture, établissement connexion, couches protocole
- ✅ Authentification SSH : mot de passe vs clés publiques
- ✅ Génération et gestion des clés SSH (ssh-keygen, ssh-copy-id)
- ✅ Tunneling SSH : local, remote, dynamic (SOCKS)
- ✅ Transfert de fichiers : SCP, SFTP, rsync over SSH
- ✅ Fonctionnalités avancées : X11, Agent, exécution commandes
- ✅ Configuration client (~/.ssh/config) et serveur (sshd_config)
- ✅ Sécurisation SSH : bonnes pratiques, fail2ban
- ✅ Comparaison détaillée Telnet vs SSH

## Conclusion

SSH a révolutionné l'administration système et la sécurité réseau depuis sa création en 1995. En remplaçant Telnet et autres protocoles non sécurisés, il a éliminé une classe entière de vulnérabilités qui mettaient en danger les systèmes informatiques.

**SSH est bien plus qu'un simple remplacement de Telnet** : c'est une plateforme complète offrant authentification forte, chiffrement robuste, tunneling flexible, et transfert de fichiers sécurisé. Sa versatilité en fait un outil indispensable pour tout professionnel de l'IT.

**Règle d'or moderne** : Si vous avez encore du Telnet en production, désactivez-le **immédiatement** et migrez vers SSH. Il n'existe aucune excuse pour utiliser Telnet en 2024 sur des systèmes accessibles via réseau.

L'investissement dans la maîtrise de SSH (clés, tunneling, configuration) est l'un des plus rentables qu'un administrateur système ou développeur puisse faire. C'est un outil quotidien qui améliore à la fois la sécurité et la productivité.

---

**Fin du module SSH et Telnet - Félicitations, vous avez complété le cours TCP/IP !** 🎉

⏭️ [SNMP : supervision réseau](/05-couche-application/08-snmp.md)
