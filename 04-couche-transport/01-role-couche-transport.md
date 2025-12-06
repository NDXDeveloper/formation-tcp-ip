🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.1 Rôle de la couche Transport

## Introduction

La couche Transport occupe une position stratégique dans le modèle TCP/IP : elle fait le pont entre le monde des applications (où nous, humains, interagissons avec les logiciels) et le monde du réseau (où les machines acheminent des paquets). Sans elle, Internet tel que nous le connaissons serait tout simplement impossible.

Pour comprendre son rôle, partons d'un constat simple : **la couche Internet (IP) ne suffit pas**.

## Le problème : les limitations d'IP

IP (Internet Protocol) fait remarquablement bien son travail : acheminer des paquets d'une machine A vers une machine B, potentiellement à travers le monde entier. Mais IP s'arrête là. Il présente plusieurs limitations fondamentales :

### 1. Impossibilité de différencier les applications

Imaginez que votre ordinateur (IP : 192.168.1.50) reçoit un paquet provenant d'un serveur (IP : 93.184.216.34). Votre système d'exploitation sait que le paquet est arrivé, mais **pour quelle application** ?

- Est-ce pour votre navigateur web qui charge une page ?
- Pour votre client email qui télécharge des messages ?
- Pour votre application de musique en streaming ?
- Pour votre client Torrent qui télécharge un fichier ?

**IP ne peut pas répondre à cette question.** Il livre simplement le paquet à la machine, sans indication sur le destinataire final.

### 2. Aucune garantie de livraison

IP fonctionne en mode "best effort" (meilleur effort). Cela signifie :

- Les paquets **peuvent se perdre** (routeur saturé, câble défectueux, interférence Wi-Fi)
- Les paquets **peuvent arriver en désordre** (routes différentes, changements de routage)
- Les paquets **peuvent être dupliqués** (retransmission au niveau liaison en cas d'erreur)
- Les paquets **peuvent être corrompus** (malgré le checksum IP qui ne détecte pas toutes les erreurs)

**Exemple concret** : Vous envoyez un email de 50 Ko. IP le découpe en ~35 paquets. Si IP se contente de les envoyer, il n'y a aucune garantie que :
- Tous les 35 paquets arrivent
- Ils arrivent dans l'ordre (paquet 1, puis 2, puis 3...)
- Aucun n'est dupliqué

Sans couche Transport, votre email arriverait corrompu ou incomplet.

### 3. Absence de contrôle de flux

Si un serveur puissant (connexion 10 Gbps) envoie des données à votre smartphone (connexion 4G limitée), IP n'a aucun mécanisme pour ralentir l'envoi. Le serveur peut littéralement **noyer** votre appareil sous un déluge de données que celui-ci ne peut pas traiter.

### 4. Pas de notion de connexion

IP traite chaque paquet indépendamment. Il n'y a aucune notion de "session" ou de "connexion" entre deux machines. Chaque paquet est une entité isolée qui ignore totalement les paquets qui l'ont précédé ou qui le suivront.

## La solution : la couche Transport

La couche Transport résout élégamment tous ces problèmes. Voici ses rôles principaux.

## Rôle 1 : Multiplexage et démultiplexage

C'est le rôle **fondamental** de la couche Transport, celui sans lequel rien d'autre ne serait possible.

### Qu'est-ce que le multiplexage ?

Le **multiplexage** est le processus qui permet à plusieurs applications sur une même machine d'envoyer des données simultanément sur le réseau. Le **démultiplexage** est l'opération inverse : diriger les données entrantes vers la bonne application.

### Comment ça fonctionne : les ports

La couche Transport utilise des **numéros de port** pour identifier les applications. Un port est simplement un nombre entre 0 et 65535.

**Analogie postale améliorée** :
- L'adresse IP est l'adresse de l'immeuble : "123 Rue du Réseau"
- Le port est le numéro d'appartement : "Apt 80" (HTTP), "Apt 443" (HTTPS), "Apt 22" (SSH)

### Exemple concret de multiplexage

Vous êtes sur votre ordinateur (IP : 192.168.1.100) et vous :
1. Consultez `google.com` dans votre navigateur
2. Téléchargez vos emails depuis `gmail.com`
3. Discutez sur Discord

Voici ce qui se passe au niveau Transport :

```
Votre ordinateur (192.168.1.100)
│
├─ Navigateur Chrome
│  └─ Connexion vers google.com (172.217.14.206:443)
│     Source: 192.168.1.100:54231
│     Destination: 172.217.14.206:443
│
├─ Client Email
│  └─ Connexion vers gmail.com (172.253.115.109:993)
│     Source: 192.168.1.100:54232
│     Destination: 172.253.115.109:993
│
└─ Application Discord
   └─ Connexion vers Discord (162.159.130.233:443)
      Source: 192.168.1.100:54233
      Destination: 162.159.130.233:443
```

**Points clés** :
- Chaque application utilise un **port source différent** (54231, 54232, 54233)
- Ces ports sont attribués **automatiquement** par le système d'exploitation
- Quand une réponse arrive de `172.217.14.206:443` vers `192.168.1.100:54231`, le système sait que c'est pour Chrome
- Quand une réponse arrive vers le port 54232, c'est pour le client email

### Le démultiplexage en action

Quand votre ordinateur reçoit un paquet :

1. **Couche Accès réseau** : "Ce paquet est pour moi (adresse MAC correcte)"
2. **Couche Internet** : "Ce paquet est pour mon IP 192.168.1.100"
3. **Couche Transport** : "Ce paquet est pour le port 54231, je le donne à l'application qui écoute sur ce port" → Chrome reçoit les données

Sans ce mécanisme, votre ordinateur ne pourrait faire tourner qu'une seule application réseau à la fois !

## Rôle 2 : Segmentation et réassemblage

Les applications génèrent souvent de grandes quantités de données. La couche Transport doit découper ces données en morceaux gérables.

### Pourquoi segmenter ?

**Contrainte physique** : Les réseaux imposent une taille maximale de paquet. Pour Ethernet, c'est généralement 1500 octets (MTU - Maximum Transmission Unit). La couche Transport doit s'adapter à cette contrainte.

**Exemple** : Vous envoyez un fichier de 100 Ko via une application de partage de fichiers.

```
Application génère: 100 000 octets de données
                           ↓
Couche Transport découpe en: ~70 segments de ~1400 octets chacun
                           ↓
Chaque segment reçoit un en-tête Transport (20-60 octets)
                           ↓
Couche Internet ajoute l'en-tête IP (20 octets minimum)
                           ↓
Résultat: ~70 paquets IP envoyés sur le réseau
```

### Le réassemblage

Côté réception, la couche Transport doit :
1. Recevoir tous les segments (potentiellement dans le désordre)
2. Les réordonner correctement
3. Reconstituer le flux de données original
4. Le transmettre à l'application

**Avec TCP** : Les segments sont numérotés, permettant un réassemblage précis même si les segments arrivent dans le désordre.

**Avec UDP** : Pas de numérotation. Si les datagrammes arrivent en désordre, c'est à l'application de gérer (si nécessaire).

## Rôle 3 : Établissement et gestion de connexions (TCP uniquement)

TCP introduit la notion de **connexion**, un concept absent des couches inférieures.

### Qu'est-ce qu'une connexion ?

Une connexion est un **canal logique bidirectionnel** établi entre deux applications. Elle a :
- Un **début** : établissement (handshake)
- Une **durée** : échange de données
- Une **fin** : fermeture propre

### Pourquoi c'est important ?

Cela permet de :
- **Synchroniser** les deux parties (accord sur les paramètres de communication)
- **Maintenir un état** (chaque côté sait où il en est dans l'échange)
- **Détecter les problèmes** (si un côté disparaît, l'autre le détecte)

**Exemple concret** : Connexion SSH à un serveur distant

```
Votre terminal                    Serveur SSH
     │                                 │
     │─────── SYN ──────────────────>  │  Demande de connexion
     │                                 │
     │<────── SYN+ACK ───────────────  │  Accord + synchronisation
     │                                 │
     │─────── ACK ──────────────────>  │  Confirmation
     │                                 │
     │═══════ CONNEXION ÉTABLIE ═══════│
     │                                 │
     │<────── Données échangées ─────> │
     │                                 │
     │─────── FIN ──────────────────>  │  Demande de fermeture
     │                                 │
     │<────── ACK ───────────────────  │  Accusé de réception
     │<────── FIN ───────────────────  │  Fermeture de l'autre côté
     │─────── ACK ──────────────────>  │  Confirmation finale
     │                                 │
     │═══════ CONNEXION FERMÉE ════════│
```

Cette gestion explicite garantit que les deux parties sont d'accord sur l'état de la communication.

## Rôle 4 : Fiabilité de la transmission (TCP uniquement)

TCP transforme le service "best effort" non fiable d'IP en un service **fiable**.

### Mécanismes de fiabilité

#### 1. Accusés de réception (ACK)

Chaque segment envoyé doit être **confirmé** par le destinataire.

```
Expéditeur                        Récepteur
    │                                  │
    │───── Segment #1000 ───────────>  │
    │                                  │
    │<──── ACK 1001 ─────────────────  │  "J'ai reçu jusqu'à 1000"
    │                                  │
    │───── Segment #1001 ───────────>  │
    │                                  │
    │<──── ACK 1002 ─────────────────  │  "J'ai reçu jusqu'à 1001"
```

#### 2. Retransmission en cas de perte

Si un ACK n'arrive pas dans un délai raisonnable, le segment est **renvoyé**.

```
Expéditeur                        Récepteur
    │                                  │
    │───── Segment #1000 ───────────>  │
    │                                  │
    │<──── ACK 1001 ─────────────────  │
    │                                  │
    │───── Segment #1001 ──────X       │  PERDU !
    │                                  │
    │... timeout (pas d'ACK) ...       │
    │                                  │
    │───── Segment #1001 (RETRANS)──>  │  Renvoi automatique
    │                                  │
    │<──── ACK 1002 ─────────────────  │  "Reçu maintenant !"
```

#### 3. Détection et élimination des doublons

Si un segment est reçu deux fois (à cause d'une retransmission inutile), le récepteur **détecte** le doublon grâce au numéro de séquence et l'ignore.

#### 4. Réordonnancement

Les segments qui arrivent en désordre sont **réordonnés** avant d'être transmis à l'application.

```
Envoi : Segment #1 → Segment #2 → Segment #3
                          ↓
Réception : Segment #1 → Segment #3 → Segment #2 (arrivé en retard)
                          ↓
Livré à l'application : Segment #1 → Segment #2 → Segment #3 (ordre restauré)
```

### Résultat

Du point de vue de l'application, **TCP garantit** :
- Toutes les données envoyées arrivent
- Dans le bon ordre
- Sans duplication
- Sans corruption

C'est comme si l'application communiquait via un "tuyau parfait" alors que le réseau sous-jacent est imparfait.

## Rôle 5 : Contrôle de flux

Le contrôle de flux empêche un expéditeur rapide de **submerger** un récepteur lent.

### Le problème

```
Serveur puissant (10 Gbps)  ────>  Smartphone (4G: 20 Mbps)
```

Si le serveur envoie à pleine vitesse, le smartphone ne peut pas suivre. Ses buffers de réception se remplissent et débordent → **perte de données**.

### La solution : la fenêtre de réception

Le récepteur annonce une **fenêtre** : "Je peux recevoir N octets supplémentaires".

```
Smartphone                        Serveur
    │                                  │
    │<─────────────────────────────────│
    │  "Ma fenêtre = 65535 octets"     │
    │                                  │
    │<───── 32768 octets ────────────  │
    │                                  │
    │─────────────────────────────────>│
    │  "Ma fenêtre = 32767 octets"     │ (fenêtre réduite)
    │                                  │
    │<───── 16384 octets ────────────  │ (le serveur ralentit)
    │                                  │
    │─────────────────────────────────>│
    │  "Ma fenêtre = 16383 octets"     │
```

Le serveur ne peut **jamais** envoyer plus que ce que la fenêtre permet. Cela adapte automatiquement le débit à la capacité du récepteur.

## Rôle 6 : Contrôle de congestion (TCP uniquement)

Le contrôle de congestion empêche de **saturer le réseau** lui-même.

### Le problème

Même si le récepteur peut suivre, le **réseau** entre les deux peut être congestionné. Si trop de données sont envoyées, les routeurs intermédiaires saturent et commencent à **perdre des paquets**.

### La solution : adaptation au réseau

TCP détecte la congestion (via la perte de paquets) et **réduit son débit** automatiquement.

**Exemple** : Téléchargement d'un fichier volumineux

```
Début du téléchargement:
│ Débit: 1 Mbps    (démarrage lent)
│ ↓
│ Débit: 2 Mbps    (augmentation progressive)
│ ↓
│ Débit: 4 Mbps
│ ↓
│ Débit: 8 Mbps
│ ↓
│ Perte de paquets détectée ! (congestion)
│ ↓
│ Débit: 4 Mbps    (réduction immédiate)
│ ↓
│ Débit: 5 Mbps    (réaugmentation prudente)
│ ↓
│ Débit: 6 Mbps
```

Ce mécanisme protège le réseau d'un effondrement global. Si tous les utilisateurs envoyaient à pleine vitesse sans contrôle, Internet deviendrait inutilisable.

## Rôle 7 : Fourniture d'un service adapté aux besoins

La couche Transport offre **deux types de services**, via TCP et UDP, pour s'adapter aux différents besoins des applications.

### Service orienté connexion avec fiabilité (TCP)

**Caractéristiques** :
- Connexion explicite établie/fermée
- Livraison garantie
- Ordre préservé
- Contrôle de flux et de congestion

**Coût** :
- Latence plus élevée (handshakes, ACKs, retransmissions)
- Overhead (en-têtes plus gros, mécanismes de contrôle)

**Applications** : Web, email, transfert de fichiers, SSH, bases de données

### Service sans connexion, non fiable (UDP)

**Caractéristiques** :
- Pas d'établissement de connexion
- Envoi immédiat
- Pas de garantie de livraison
- Pas de contrôle de flux/congestion

**Avantages** :
- Latence minimale
- Overhead minimal
- Débit maximal

**Applications** : Streaming vidéo/audio, jeux en ligne, DNS, VoIP

### Tableau comparatif

| Critère | TCP | UDP |
|---------|-----|-----|
| **Fiabilité** | Garantie absolue | Aucune garantie |
| **Ordre** | Préservé | Non préservé |
| **Connexion** | Oui (établissement/fermeture) | Non |
| **Latence** | Plus élevée | Minimale |
| **Overhead** | Important (20-60 octets) | Minimal (8 octets) |
| **Contrôle de flux** | Oui | Non |
| **Adapté pour** | Données critiques | Données temps réel |

## Interaction avec les autres couches

La couche Transport ne fonctionne pas de manière isolée. Voyons comment elle interagit avec ses voisines.

### Avec la couche Application (au-dessus)

La couche Transport **fournit une API** aux applications via les **sockets**.

```python
# Exemple Python : une application utilise TCP
import socket

# Création d'un socket TCP
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connexion à un serveur web
sock.connect(("www.example.com", 80))

# Envoi d'une requête HTTP
sock.send(b"GET / HTTP/1.1\r\nHost: www.example.com\r\n\r\n")

# Réception de la réponse
response = sock.recv(4096)
```

L'application n'a pas à se soucier :
- De la segmentation des données
- Des retransmissions
- Du routage
- De l'encapsulation

**La couche Transport gère tout cela automatiquement**.

### Avec la couche Internet (en-dessous)

La couche Transport **utilise les services d'IP** :

1. Elle crée un segment TCP ou UDP
2. Elle demande à IP de l'acheminer vers l'adresse destination
3. IP encapsule le segment dans un paquet IP
4. IP route le paquet vers la destination

**Important** : La couche Transport fait **confiance aveuglément** à IP pour l'acheminement, mais **compense ses défaillances** (pertes, désordre) via ses propres mécanismes.

## Exemple complet : envoi d'un email

Suivons le rôle de la couche Transport lors de l'envoi d'un email de 10 Ko.

### Côté émission (votre ordinateur)

1. **Application (Outlook)** : "Envoyer cet email de 10240 octets au serveur SMTP"
2. **Couche Transport (TCP)** :
   - Établit une connexion avec le serveur (port 587)
   - Segmente les 10240 octets en 8 segments de ~1280 octets
   - Ajoute un en-tête TCP à chaque segment (numéros de séquence, ports, etc.)
   - Envoie les segments via IP
3. **Couche Internet (IP)** : Route chaque paquet vers le serveur

### Pendant la transmission

4. **Réseau** : Certains paquets prennent des routes différentes, l'un est perdu
5. **Couche Transport (TCP côté émetteur)** :
   - Détecte l'absence d'ACK pour le segment perdu
   - Le retransmet automatiquement
   - Continue d'envoyer les autres segments

### Côté réception (serveur email)

6. **Couche Internet (IP)** : Reçoit les paquets (certains en désordre)
7. **Couche Transport (TCP)** :
   - Extrait les segments TCP des paquets IP
   - Les réordonne grâce aux numéros de séquence
   - Envoie des ACK pour confirmer la réception
   - Reconstitue le flux de 10240 octets
   - Détecte que tous les segments sont arrivés
8. **Application (Serveur SMTP)** : Reçoit l'email complet, intact, prêt à être traité

### Ce qui est invisible pour les applications

- Les 8 segments
- La perte et retransmission d'un segment
- Le désordre d'arrivée
- Les ACKs échangés
- Les numéros de séquence

**L'application ne voit qu'un flux continu et fiable de données**. C'est toute la magie de la couche Transport.

## Conclusion

La couche Transport est le **chef d'orchestre invisible** qui rend Internet utilisable. Sans elle :

- Chaque application devrait gérer la fiabilité, le contrôle de flux, la segmentation, etc.
- Les développeurs devraient réinventer ces mécanismes pour chaque application
- Le code serait complexe, bogué, et inefficace
- Internet ne pourrait pas supporter des milliards d'utilisateurs simultanés

Grâce à TCP et UDP, la couche Transport offre :
- **Une abstraction puissante** : les applications voient un canal de communication simple
- **Une flexibilité** : choix entre fiabilité (TCP) et performance (UDP)
- **Une normalisation** : tous les systèmes parlent le même langage

Dans les sections suivantes, nous plongerons dans les détails techniques de ces mécanismes fascinants, en commençant par comprendre précisément ce que sont les ports et les sockets.

---

**À retenir** :

- ✅ La couche Transport assure le **multiplexage** (plusieurs applications sur une machine)
- ✅ Elle offre deux services : **TCP** (fiable) et **UDP** (rapide)
- ✅ TCP ajoute **fiabilité, ordre, contrôle de flux et de congestion** au-dessus d'IP
- ✅ UDP est **minimal** : juste le multiplexage, sans garanties
- ✅ Les **ports** identifient les applications (0-65535)
- ✅ La couche Transport **masque la complexité** du réseau aux applications

⏭️ [Notion de ports et de sockets](/04-couche-transport/02-ports-sockets.md)
