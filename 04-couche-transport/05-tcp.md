🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.5 TCP (Transmission Control Protocol)

## Vue d'ensemble

TCP (Transmission Control Protocol) est l'un des protocoles fondamentaux de la suite TCP/IP et certainement le plus utilisé au niveau de la couche Transport. Créé dans les années 1970 et standardisé dans la RFC 793 (puis révisé dans la RFC 9293), TCP est conçu pour fournir une **communication fiable, orientée connexion** entre deux applications sur un réseau.

Si UDP peut être comparé à l'envoi d'une lettre simple, TCP ressemble davantage à un appel téléphonique : une connexion est établie avant l'échange, les deux parties communiquent de manière bidirectionnelle, et la connexion est fermée proprement à la fin.

## Position dans la pile TCP/IP

```
┌─────────────────────────────────────┐
│   Couche Application                │
│   (HTTP, FTP, SSH, SMTP...)         │
├─────────────────────────────────────┤
│   Couche Transport                  │
│   ┌─────────┐      ┌─────────┐      │
│   │   TCP   │      │   UDP   │      │  ← Nous sommes ici
│   └─────────┘      └─────────┘      │
├─────────────────────────────────────┤
│   Couche Internet (IP)              │
├─────────────────────────────────────┤
│   Couche Accès réseau               │
└─────────────────────────────────────┘
```

TCP opère au niveau de la couche Transport et s'appuie sur IP (couche Internet) pour acheminer les paquets à travers le réseau. Il ajoute à IP toute l'intelligence nécessaire pour garantir une transmission fiable des données.

## Pourquoi TCP ?

### Le problème à résoudre

Imaginez que vous devez transférer un fichier de 10 Mo entre deux ordinateurs via Internet. Plusieurs problèmes peuvent survenir :

1. **Perte de paquets** : Certains paquets peuvent être perdus en route (congestion, erreurs réseau)
2. **Désordonnancement** : Les paquets peuvent arriver dans le désordre (routes différentes)
3. **Duplication** : Un même paquet peut arriver plusieurs fois
4. **Corruption** : Les données peuvent être altérées durant le transport
5. **Surcharge** : L'émetteur peut envoyer trop vite pour le récepteur ou le réseau

IP seul ne résout aucun de ces problèmes. C'est là qu'intervient TCP.

### La solution TCP

TCP résout ces problèmes en ajoutant plusieurs mécanismes sophistiqués :

- ✅ **Fiabilité** : Garantit que toutes les données arrivent correctement
- ✅ **Ordre** : Remet les données dans le bon ordre
- ✅ **Contrôle de flux** : Adapte la vitesse d'envoi aux capacités du récepteur
- ✅ **Contrôle de congestion** : Adapte la vitesse d'envoi aux capacités du réseau
- ✅ **Connexion établie** : Crée un canal logique entre deux applications

## Exemple concret : Chargement d'une page web

Lorsque vous accédez à `https://www.exemple.com`, voici ce qui se passe avec TCP :

```
Client (votre navigateur)          Serveur web
      |                                  |
      |  1. Établissement connexion TCP  |
      |  (3-way handshake)               |
      |--------------------------------->|
      |<---------------------------------|
      |--------------------------------->|
      |                                  |
      |  2. Requête HTTP                 |
      |  GET / HTTP/1.1                  |
      |--------------------------------->|
      |                                  |
      |  3. Réponse HTTP                 |
      |  (Page HTML, CSS, images...)     |
      |<---------------------------------|
      |  [ACK]                           |
      |--------------------------------->|
      |<---------------------------------|
      |  [ACK]                           |
      |--------------------------------->|
      |                                  |
      |  4. Fermeture connexion          |
      |  (4-way handshake)               |
      |--------------------------------->|
      |<---------------------------------|
      |--------------------------------->|
      |<---------------------------------|
```

**Ce que TCP gère automatiquement** :
- Si un paquet contenant une partie de l'image est perdu → **retransmission automatique**
- Si les paquets arrivent dans le désordre → **réorganisation**
- Si le serveur envoie trop vite → **ralentissement automatique**
- Si le réseau est congestionné → **adaptation du débit**

## Les garanties fondamentales de TCP

### 1. Communication orientée connexion

Contrairement à UDP, TCP établit une **connexion logique** avant tout échange de données. C'est comme composer un numéro de téléphone et attendre que quelqu'un décroche avant de parler.

```
État initial : CLOSED

1. Demande de connexion (SYN)
2. Acceptation + confirmation (SYN-ACK)
3. Confirmation finale (ACK)

État final : ESTABLISHED (connexion active)
```

### 2. Transmission fiable avec acquittements

Chaque segment TCP envoyé doit être **acquitté** (acknowledged) par le récepteur. Si l'acquittement n'arrive pas dans un délai donné, le segment est retransmis.

```
Émetteur                    Récepteur
   |                            |
   |--- Segment 1 (seq=100) --->|
   |<-- ACK 201 ----------------|  (Bien reçu)
   |                            |
   |--- Segment 2 (seq=200) --->|
   |         ❌ (perdu)         |
   |                            |
   |--- Timeout → Retrans. ---->|
   |<-- ACK 301 ----------------|  (Reçu cette fois)
```

### 3. Ordre garanti

Les données sont numérotées et remises dans l'ordre au destinataire, même si les paquets arrivent dans le désordre sur le réseau.

```
Envoyé : A → B → C → D
Arrivé :  B → D → A → C
Remis à l'application : A → B → C → D ✓
```

### 4. Contrôle de flux

TCP empêche l'émetteur de submerger le récepteur en utilisant une **fenêtre de réception** qui indique combien de données le récepteur peut encore accepter.

```
Récepteur : "J'ai 64 Ko de buffer disponible"
Émetteur : "OK, je n'enverrai pas plus de 64 Ko sans ACK"
```

### 5. Contrôle de congestion

TCP détecte la congestion du réseau et adapte son débit en conséquence pour éviter d'aggraver la situation.

```
Débit initial : Lent
           ↓
Aucune perte détectée → Augmentation progressive
           ↓
Perte détectée ! → Réduction rapide du débit
           ↓
Nouvelle augmentation progressive...
```

## Applications typiques de TCP

TCP est utilisé par la majorité des applications nécessitant une communication fiable :

| Protocole | Port | Usage |
|-----------|------|-------|
| **HTTP/HTTPS** | 80/443 | Navigation web |
| **FTP** | 20/21 | Transfert de fichiers |
| **SMTP** | 25 | Envoi d'emails |
| **SSH** | 22 | Accès distant sécurisé |
| **Telnet** | 23 | Accès distant (non sécurisé) |
| **IMAP** | 143 | Récupération d'emails |
| **POP3** | 110 | Récupération d'emails |
| **MySQL** | 3306 | Base de données |
| **PostgreSQL** | 5432 | Base de données |
| **Redis** | 6379 | Cache/Base de données |

## Le coût de la fiabilité

Toutes ces garanties ont un coût :

### 1. Latence initiale
L'établissement de connexion nécessite un **aller-retour complet** (3-way handshake) avant de pouvoir envoyer des données.

```
Temps d'établissement ≈ 1 RTT (Round Trip Time)
Si RTT = 50 ms → 50 ms avant d'envoyer la première donnée
```

### 2. Overhead de bande passante
Chaque segment TCP transporte :
- **20 octets minimum** d'en-tête TCP
- **20 octets minimum** d'en-tête IP
- Les acquittements consomment de la bande passante

```
Payload utile : 1000 octets
En-têtes TCP/IP : 40 octets
Overhead : 40/1040 ≈ 3.8%
```

### 3. Retards dus aux retransmissions
En cas de perte, TCP attend avant de retransmettre, ce qui peut introduire des délais.

```
Segment perdu à t=0
Timeout à t=200 ms
Retransmission à t=200 ms
ACK reçu à t=250 ms
→ Délai total : 250 ms au lieu de 50 ms
```

### 4. Head-of-line blocking
Si un segment est perdu, tous les segments suivants (même reçus) sont bloqués jusqu'à sa réception.

```
Segments envoyés : 1, 2, 3, 4, 5
Segment 3 perdu
L'application ne reçoit rien tant que 3 n'est pas retransmis
Même si 4 et 5 sont déjà arrivés !
```

## Quand utiliser TCP ?

TCP est le choix approprié quand :

- ✅ **La fiabilité est prioritaire** : Pas de perte de données acceptable
- ✅ **L'ordre compte** : Les données doivent arriver dans l'ordre
- ✅ **La latence n'est pas critique** : Quelques millisecondes de plus sont acceptables
- ✅ **Connexions longues** : La communication dure suffisamment pour amortir le coût de la connexion

**Exemples parfaits** :
- Transfert de fichiers (FTP, HTTP)
- Emails (SMTP, IMAP)
- Pages web (HTTP/HTTPS)
- Bases de données
- API REST
- Sessions SSH

## Exemple comparatif : TCP vs UDP

Prenons l'envoi d'un message de 1000 octets :

### Avec UDP :
```
1. Envoi immédiat du datagramme
2. Aucune vérification
3. Si perdu → perdu définitivement

Temps total : ~10 ms (1 RTT/2)
Garantie : Aucune
```

### Avec TCP :
```
1. Établissement connexion (50 ms)
2. Envoi du segment
3. Attente de l'ACK (50 ms)
4. Si pas d'ACK → retransmission

Temps total : ~100-150 ms minimum
Garantie : Livraison certaine
```

Pour un message unique, UDP est 10× plus rapide. Mais pour une session de 1000 messages, TCP n'a qu'un surcoût de 5-10% car la connexion est réutilisée.

## La terminologie TCP

Avant d'aller plus loin, il est important de comprendre le vocabulaire spécifique à TCP :

- **Segment** : L'unité de données TCP (équivalent du "paquet" pour IP ou "datagramme" pour UDP)
- **Connexion** : Le canal logique établi entre deux points
- **Socket** : Le point d'extrémité d'une connexion (IP + port)
- **Flux** (stream) : Le flot continu de données sur une connexion TCP
- **Séquence** : Numérotation des octets transmis
- **Acquittement** (ACK) : Confirmation de réception
- **Fenêtre** : Quantité de données pouvant être envoyées sans ACK
- **MSS** (Maximum Segment Size) : Taille maximale des données dans un segment

## Structure conceptuelle d'une connexion TCP

Une connexion TCP peut être vue comme un **tube bidirectionnel** entre deux applications :

```
Application A                     Application B
     |                                   |
     | Buffer d'émission (TX)            |
     | Buffer de réception (RX)          |
     |                                   |
  [Socket A]                         [Socket B]
     |                                   |
     |======= Connexion TCP =============|
     |                                   |
     |  Données A→B                      |
     |---------------------------------> |
     |  ACK B→A                          |
     |<--------------------------------- |
     |  Données B→A                      |
     |<--------------------------------- |
     |  ACK A→B                          |
     |---------------------------------> |
```

Chaque direction est **indépendante** :
- A peut envoyer à B pendant que B envoie à A
- Chaque direction a ses propres numéros de séquence
- Chaque direction a son propre contrôle de flux

## Ce que nous allons étudier

Dans les sections suivantes, nous approfondirons chaque aspect de TCP :

1. **Caractéristiques détaillées** (4.5.1) : Fiabilité, ordre, contrôle
2. **Format du segment** (4.5.2) : Structure de l'en-tête TCP
3. **Établissement de connexion** (4.5.3) : Le fameux 3-way handshake
4. **Fermeture de connexion** (4.5.4) : Le 4-way handshake
5. **Numéros de séquence** (4.5.5) : Comment TCP numérote les données
6. **Fenêtre glissante** (4.5.6) : Optimisation du débit
7. **Contrôle de flux** (4.5.7) : Adaptation au récepteur
8. **Contrôle de congestion** (4.5.8) : Adaptation au réseau
9. **Retransmissions** (4.5.9) : Gestion des pertes
10. **États de connexion** (4.5.10) : Le cycle de vie complet

## Conclusion

TCP est un protocole remarquablement sophistiqué qui a fait ses preuves depuis plus de 40 ans. Il transforme le réseau IP, intrinsèquement **non fiable**, en un canal de communication **fiable et ordonné** sur lequel peuvent s'appuyer des applications complexes.

Son succès repose sur un équilibre subtil entre :
- **Performance** : Utilisation efficace de la bande passante
- **Fiabilité** : Garantie de livraison des données
- **Adaptabilité** : Réaction aux conditions du réseau

Comprendre TCP en profondeur est essentiel pour :
- 🔧 Diagnostiquer des problèmes réseau
- ⚡ Optimiser les performances d'applications
- 🏗️ Concevoir des architectures distribuées efficaces
- 🛡️ Identifier des attaques réseau

Dans les sections suivantes, nous explorerons chacun de ces mécanismes en détail pour vous donner une compréhension complète de ce protocole fondamental.

---

**Points clés à retenir** :
- TCP est un protocole **orienté connexion** et **fiable**
- Il garantit la **livraison ordonnée** de toutes les données
- Il implémente un **contrôle de flux** et un **contrôle de congestion**
- Ces garanties ont un **coût en latence et en overhead**
- TCP est idéal pour les applications nécessitant une **communication fiable**
- La majorité du trafic Internet utilise TCP (HTTP, HTTPS, SSH, etc.)

⏭️ [Caractéristiques : fiabilité, ordre, contrôle](/04-couche-transport/05.1-tcp-caracteristiques.md)
