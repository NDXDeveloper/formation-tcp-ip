🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.2 Notion de ports et de sockets

## Introduction

Les ports et les sockets sont deux concepts fondamentaux de la couche Transport, mais ils sont souvent confondus. Pourtant, ils représentent des notions distinctes et complémentaires :

- Un **port** est un **numéro** qui identifie une application ou un service
- Un **socket** est un **point de terminaison** de communication, une interface de programmation

Comprendre ces deux concepts est essentiel pour maîtriser les communications réseau. Commençons par les ports.

## Les ports : identification des applications

### Qu'est-ce qu'un port ?

Un **port** est un nombre entier compris entre **0 et 65535** qui permet d'identifier de manière unique un processus ou un service sur une machine.

**Analogie de l'immeuble revisitée** :

Imaginez un immeuble (votre ordinateur) avec son adresse postale (adresse IP). Dans cet immeuble, il y a 65 536 appartements numérotés de 0 à 65535. Chaque appartement peut héberger une application différente.

```
Immeuble "192.168.1.100"
│
├─ Appartement 80    → Serveur Web (HTTP)
├─ Appartement 443   → Serveur Web sécurisé (HTTPS)
├─ Appartement 22    → Serveur SSH
├─ Appartement 25    → Serveur Email (SMTP)
├─ Appartement 3306  → Serveur MySQL
├─ Appartement 5432  → Serveur PostgreSQL
└─ Appartement 54231 → Navigateur Chrome (connexion temporaire)
```

### Pourquoi les ports sont-ils nécessaires ?

Sans ports, un ordinateur ne pourrait faire tourner qu'**une seule application réseau à la fois**. Les ports permettent le **multiplexage** : plusieurs applications peuvent communiquer simultanément via le réseau.

**Exemple concret** : Vous êtes sur votre ordinateur et simultanément :

1. Vous consultez Gmail dans votre navigateur
2. Vous téléchargez un fichier via FTP
3. Vous discutez sur Slack
4. Vous écoutez Spotify
5. Votre système synchronise Dropbox en arrière-plan

Chaque application utilise un port différent, permettant au système d'exploitation de router correctement les données entrantes et sortantes.

### Structure d'un port

Un port est codé sur **16 bits**, d'où la plage 0-65535 (2^16 = 65536 valeurs possibles).

```
Port codé sur 16 bits :
┌─────────────────┬─────────────────┐
│  8 bits élevés  │  8 bits faibles │
└─────────────────┴─────────────────┘

Exemple : Port 443 (HTTPS)
Binaire  : 00000001 10111011
Hexadécimal : 0x01BB
Décimal : 443
```

Dans un paquet TCP ou UDP, le port source et le port destination occupent chacun 16 bits dans l'en-tête.

## Anatomie d'une connexion réseau

Une connexion réseau complète est identifiée par un **5-tuple** (quintuplet) :

```
(Protocole, IP source, Port source, IP destination, Port destination)
```

**Exemple** : Vous consultez www.example.com depuis votre ordinateur

```
┌──────────────────────────────────────────────────────┐
│ Protocole   : TCP                                    │
│ IP source   : 192.168.1.100 (votre ordinateur)       │
│ Port source : 54231 (attribué automatiquement)       │
│ IP dest.    : 93.184.216.34 (serveur example.com)    │
│ Port dest.  : 443 (HTTPS)                            │
└──────────────────────────────────────────────────────┘
```

Ce quintuplet identifie **de manière unique** cette connexion parmi toutes les connexions actives sur Internet à cet instant.

### Ports source vs ports destination

Il est crucial de comprendre la différence :

#### Port destination

Le **port destination** est le port sur lequel le **service** écoute. Il est généralement :
- **Fixe** et **connu à l'avance**
- **Standardisé** pour les services courants
- **Publié** pour que les clients sachent comment se connecter

**Exemples** :
- HTTP : port 80
- HTTPS : port 443
- SSH : port 22
- FTP : port 21
- MySQL : port 3306

#### Port source

Le **port source** est le port utilisé par le **client** pour sa connexion. Il est généralement :
- **Attribué automatiquement** par le système d'exploitation
- **Éphémère** (temporaire, libéré après la fermeture de la connexion)
- **Unique** pour chaque connexion active

**Exemple détaillé** : Vous ouvrez trois onglets dans Chrome

```
Onglet 1 : google.com
   Client : 192.168.1.100:54231 → Serveur : 172.217.14.206:443

Onglet 2 : github.com
   Client : 192.168.1.100:54232 → Serveur : 140.82.121.4:443

Onglet 3 : stackoverflow.com
   Client : 192.168.1.100:54233 → Serveur : 151.101.1.69:443
```

Les trois connexions vont vers le **même port destination** (443), mais utilisent des **ports source différents** (54231, 54232, 54233), permettant au système de distinguer les réponses.

### Comment le système choisit un port source ?

Quand une application (comme votre navigateur) initie une connexion, elle ne spécifie généralement pas de port source. Le système d'exploitation :

1. Choisit un port disponible dans la plage des **ports éphémères** (généralement 49152-65535)
2. Vérifie qu'il n'est pas déjà utilisé
3. L'attribue à la connexion
4. Le libère quand la connexion se termine

```
Application : socket.connect(("example.com", 443))
                     ↓
OS : "Je vais utiliser le port 54231 qui est libre"
                     ↓
Connexion établie : 192.168.1.100:54231 → 93.184.216.34:443
```

## Les sockets : interface de programmation

Maintenant que nous comprenons les ports, passons aux sockets.

### Qu'est-ce qu'un socket ?

Un **socket** est un **point de terminaison de communication**. Plus précisément, c'est une structure de données dans le système d'exploitation qui représente une connexion réseau et fournit une **interface de programmation** pour envoyer et recevoir des données.

**Analogie** : Si le port est le numéro d'appartement, le socket est la **porte de l'appartement** avec sa poignée et sa serrure. C'est par cette porte que les données entrent et sortent.

### Socket vs Port : quelle différence ?

C'est une source de confusion fréquente. Clarifions :

| Concept | Nature | Rôle | Exemple |
|---------|--------|------|---------|
| **Port** | Nombre (0-65535) | Identifie une application | 443, 80, 22 |
| **Socket** | Structure de données + API | Interface de communication | Objet logiciel avec méthodes send(), recv() |

**En d'autres termes** :
- Le **port** est l'**adresse** où joindre l'application
- Le **socket** est l'**outil** que l'application utilise pour communiquer

### Exemple concret avec code

```python
import socket

# Création d'un socket TCP/IP
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# → Le socket existe, mais n'est encore lié à aucun port

# Connexion à un serveur web (exemple.com:80)
sock.connect(("example.com", 80))
# → Le système attribue automatiquement un port source (ex: 54231)
# → Le socket est maintenant lié au port 54231 localement
# → La connexion est : 192.168.1.100:54231 → 93.184.216.34:80

# Envoi de données via le socket
sock.send(b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")

# Réception de données via le socket
data = sock.recv(4096)

# Fermeture du socket
sock.close()
# → Le port 54231 est libéré et peut être réutilisé
```

Dans cet exemple :
- `sock` est le **socket** (objet Python)
- `54231` est le **port source** (attribué automatiquement)
- `80` est le **port destination** (spécifié explicitement)

## Identification complète d'un socket

Un socket est identifié de manière unique par un **4-tuple** (en incluant le protocole, on a un 5-tuple) :

```
(IP locale, Port local, IP distante, Port distant)
```

**Exemple** : Deux connexions simultanées à Google

```
Connexion 1 (Onglet Chrome #1) :
   Socket 1 : (192.168.1.100, 54231, 142.250.185.46, 443)

Connexion 2 (Onglet Chrome #2) :
   Socket 2 : (192.168.1.100, 54232, 142.250.185.46, 443)
```

Bien que les deux connexions vont vers la **même adresse IP et le même port de destination**, elles sont **distinctes** car elles utilisent des **ports source différents**.

## Types de sockets

Il existe deux principaux types de sockets dans TCP/IP, correspondant aux deux protocoles de la couche Transport.

### Socket de flux (Stream Socket) - TCP

**Type** : `SOCK_STREAM`

**Caractéristiques** :
- Orienté connexion
- Fiable (garantie de livraison)
- Ordre préservé
- Bidirectionnel

**Analogie** : Appel téléphonique
- Vous composez le numéro (connexion)
- Vous parlez et écoutez (communication bidirectionnelle)
- Vous raccrochez (fermeture)
- Les mots arrivent dans l'ordre, sans perte

**Cas d'usage** :
- Navigation web (HTTP/HTTPS)
- Transfert de fichiers (FTP, SFTP)
- Email (SMTP, IMAP)
- Bases de données (MySQL, PostgreSQL)

**Exemple de code** :

```python
import socket

# Création d'un socket TCP
tcp_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Le socket se comporte comme un "tuyau" bidirectionnel fiable
tcp_socket.connect(("server.com", 8080))
tcp_socket.send(b"Hello")
response = tcp_socket.recv(1024)
tcp_socket.close()
```

### Socket de datagramme (Datagram Socket) - UDP

**Type** : `SOCK_DGRAM`

**Caractéristiques** :
- Sans connexion
- Non fiable (pas de garantie de livraison)
- Ordre non garanti
- Messages indépendants

**Analogie** : Envoyer des cartes postales
- Vous écrivez une carte et l'envoyez (pas de connexion préalable)
- Elle peut se perdre
- Si vous en envoyez plusieurs, elles peuvent arriver en désordre
- Chaque carte est indépendante

**Cas d'usage** :
- Streaming vidéo/audio
- Jeux en ligne
- DNS
- VoIP

**Exemple de code** :

```python
import socket

# Création d'un socket UDP
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Pas de connexion, envoi direct de datagrammes
udp_socket.sendto(b"Hello", ("server.com", 8080))
data, addr = udp_socket.recvfrom(1024)
udp_socket.close()
```

### Comparaison visuelle

```
Socket TCP (Stream)                Socket UDP (Datagram)
═══════════════════               ═══════════════════

Client         Serveur             Client         Serveur
  │              │                   │              │
  │──connect()──>│                   │              │
  │<─── OK ───── │                   │              │
  │              │                   │──sendto()───>│
  │──send()────> │                   │              │
  │<──recv()──── │                   │<─recvfrom()──│
  │──send()────> │                   │──sendto()───>│
  │──close()───> │                   │              │
  │              │                   │──close()──── │
                                     │              │

Connexion établie                   Pas de connexion
Ordre garanti                       Ordre non garanti
Fiable                              Non fiable
```

## Cycle de vie d'un socket

### Pour un socket TCP (client)

```
1. Création
   sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
   État : Socket existe mais n'est pas lié

2. Connexion
   sock.connect(("server.com", 80))
   État : Connexion établie, port source attribué automatiquement

3. Communication
   sock.send(b"GET / HTTP/1.1...")
   data = sock.recv(4096)
   État : Échange de données bidirectionnel

4. Fermeture
   sock.close()
   État : Connexion fermée, port libéré
```

### Pour un socket TCP (serveur)

```
1. Création
   sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

2. Liaison (bind)
   sock.bind(("0.0.0.0", 8080))
   État : Socket lié au port 8080

3. Écoute
   sock.listen(5)
   État : Socket en mode écoute, attend des connexions

4. Acceptation
   client_sock, client_addr = sock.accept()
   État : Nouveau socket créé pour chaque client

5. Communication
   data = client_sock.recv(4096)
   client_sock.send(b"Response")

6. Fermeture
   client_sock.close()  # Ferme la connexion client
   sock.close()         # Ferme le socket d'écoute
```

### Schéma complet client-serveur TCP

```
CLIENT                                SERVEUR
────────────────────────────────────────────────────────

socket()                              socket()
   │                                     │
   │                                  bind(port 8080)
   │                                     │
   │                                  listen()
   │                                     │
   │                                  accept() ← bloque
   │                                     │
connect(server, 8080) ─────────────────> │
   │                                     │
   │<──────────── Handshake ───────────> │
   │                                     │
   │                                  accept() retourne
   │                                     │
   │                                  nouveau socket
   │                                  pour ce client
   │                                     │
send("Hello") ─────────────────────────> │
   │                                     │
   │                                  recv() reçoit "Hello"
   │                                     │
   │<───────────────────────────── send("Hi")
   │                                     │
recv() reçoit "Hi"                       │
   │                                     │
close() ───────────────────────────────> │
   │                                     │
   │                                  close()
```

## Ports et sockets côté serveur

Un serveur présente un comportement particulier qui mérite d'être détaillé.

### Le socket d'écoute

Quand un serveur démarre, il crée un **socket d'écoute** (listening socket) lié à un port spécifique.

```python
# Serveur web écoutant sur le port 80
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(("0.0.0.0", 80))  # Écoute sur toutes les interfaces
server.listen(128)             # File d'attente de 128 connexions max
```

Ce socket **ne communique jamais directement** avec les clients. Son seul rôle est d'**accepter les nouvelles connexions**.

### Création de sockets de communication

Chaque fois qu'un client se connecte, `accept()` crée un **nouveau socket** dédié à ce client.

```python
while True:
    client_socket, client_address = server.accept()
    # client_socket est un NOUVEAU socket
    # Il communiquera avec ce client spécifique

    print(f"Connexion de {client_address}")
    # Traiter la requête via client_socket
    handle_client(client_socket)
```

### Exemple avec plusieurs clients

```
SERVEUR web sur port 80

Socket d'écoute : 0.0.0.0:80 (attend les connexions)
         │
         ├─> Client 1 : 203.0.113.5:54231
         │   Socket dédié : (serveur:80 ↔ 203.0.113.5:54231)
         │
         ├─> Client 2 : 198.51.100.42:54232
         │   Socket dédié : (serveur:80 ↔ 198.51.100.42:54232)
         │
         └─> Client 3 : 192.0.2.17:54233
             Socket dédié : (serveur:80 ↔ 192.0.2.17:54233)
```

**Point clé** : Le serveur utilise **le même port (80)** pour toutes les connexions, mais crée un **socket distinct** pour chaque client. Le système les différencie grâce au 4-tuple complet (IP source, port source, IP dest, port dest).

## Notion de socket réutilisable

### Le problème TIME_WAIT

Après la fermeture d'une connexion TCP, le socket entre dans un état `TIME_WAIT` pendant 2*MSL (Maximum Segment Lifetime, typiquement 2-4 minutes). Pendant ce temps, le port ne peut pas être réutilisé immédiatement.

**Problème pour les serveurs** : Si un serveur crash et redémarre, il ne peut pas se lier immédiatement au même port.

```bash
$ python server.py
Binding to port 8080...
OSError: [Errno 48] Address already in use
```

### Solution : SO_REUSEADDR

L'option `SO_REUSEADDR` permet de réutiliser un port immédiatement.

```python
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(("0.0.0.0", 8080))
server.listen()
```

Cette option est **essentielle** pour les serveurs en développement et production.

## Sockets et système d'exploitation

### Les sockets sont des ressources système

Un socket est une **ressource système** au même titre qu'un fichier ouvert. Le système d'exploitation maintient :

- Une **table des sockets** pour chaque processus
- Des **buffers** d'envoi et de réception pour chaque socket
- Des **informations d'état** (connecté, en écoute, fermé, etc.)

### Limites du système

Le nombre de sockets qu'un processus peut ouvrir est limité.

**Linux** : Vérifier la limite

```bash
$ ulimit -n
1024
```

Cela signifie qu'un processus peut ouvrir jusqu'à 1024 fichiers/sockets simultanément.

**Impact** : Un serveur web gérant 10 000 connexions simultanées doit :
1. Augmenter cette limite
2. Utiliser des techniques d'I/O non bloquantes (epoll, kqueue)
3. Optimiser la gestion des ressources

### Visualisation des sockets actifs

Vous pouvez voir les sockets actifs sur votre système :

```bash
# Linux/Mac
$ netstat -an | grep ESTABLISHED
tcp4  0  0  192.168.1.100.54231  172.217.14.206.443  ESTABLISHED
tcp4  0  0  192.168.1.100.54232  140.82.121.4.443    ESTABLISHED

# Alternative moderne
$ ss -tan
State    Recv-Q Send-Q Local Address:Port  Peer Address:Port
ESTAB    0      0      192.168.1.100:54231 172.217.14.206:443
ESTAB    0      0      192.168.1.100:54232 140.82.121.4:443
```

## Cas pratiques détaillés

### Cas 1 : Navigation web simultanée

Vous ouvrez 5 onglets dans Chrome vers des sites différents.

```
Votre ordinateur : 192.168.1.100

Onglet 1 : google.com
  Socket : (192.168.1.100:54231, 172.217.14.206:443)

Onglet 2 : github.com
  Socket : (192.168.1.100:54232, 140.82.121.4:443)

Onglet 3 : stackoverflow.com
  Socket : (192.168.1.100:54233, 151.101.1.69:443)

Onglet 4 : youtube.com
  Socket : (192.168.1.100:54234, 172.217.167.206:443)

Onglet 5 : twitter.com
  Socket : (192.168.1.100:54235, 104.244.42.129:443)
```

**Observation** :
- 5 sockets différents (5 objets dans la mémoire de Chrome)
- 5 ports source différents (54231-54235)
- Tous vers le port destination 443 (HTTPS)
- Le système sait router les réponses grâce aux ports source

### Cas 2 : Serveur web gérant plusieurs clients

Un serveur Nginx écoute sur le port 80.

```
Serveur : 93.184.216.34:80

Socket d'écoute :
  bind(93.184.216.34:80)
  listen()

3 clients se connectent simultanément :

Client A (203.0.113.5:54231)
  → Nginx crée socket A : (93.184.216.34:80, 203.0.113.5:54231)

Client B (198.51.100.42:55000)
  → Nginx crée socket B : (93.184.216.34:80, 198.51.100.42:55000)

Client C (192.0.2.17:49152)
  → Nginx crée socket C : (93.184.216.34:80, 192.0.2.17:49152)
```

**Point clé** : Le serveur utilise le **même port local (80)** mais distingue les clients grâce à leurs **IP et ports sources** différents.

### Cas 3 : Communication UDP (DNS)

Requête DNS pour résoudre `example.com`.

```
Client : 192.168.1.100
Serveur DNS : 8.8.8.8 (Google DNS)

1. Création du socket UDP
   sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

2. Envoi de la requête DNS
   sock.sendto(dns_query, ("8.8.8.8", 53))
   → Socket : (192.168.1.100:54231, 8.8.8.8:53)
   → Un seul datagramme UDP envoyé

3. Réception de la réponse
   response, addr = sock.recvfrom(512)
   → Réponse reçue de (8.8.8.8:53)

4. Fermeture
   sock.close()
```

**Différence avec TCP** :
- Pas de `connect()`, pas de handshake
- Envoi direct avec `sendto()`
- Une seule requête, une seule réponse
- Rapide mais non fiable (si la réponse se perd, il faut renvoyer la requête)

## Résumé des concepts clés

### Port

```
┌─────────────────────────────────────────┐
│ Port = Numéro entre 0 et 65535          │
│                                         │
│ Rôle : Identifier une application       │
│        sur une machine                  │
│                                         │
│ Types :                                 │
│  • Port destination (service connu)     │
│  • Port source (éphémère)               │
└─────────────────────────────────────────┘
```

### Socket

```
┌────────────────────────────────────────────┐
│ Socket = Point de terminaison              │
│          + Interface de programmation      │
│                                            │
│ Identifié par : (IP locale, Port local,    │
│                  IP distante, Port distant)│
│                                            │
│ Types :                                    │
│  • SOCK_STREAM (TCP)                       │
│  • SOCK_DGRAM (UDP)                        │
└────────────────────────────────────────────┘
```

### Relation Port-Socket

```
Un PORT peut avoir plusieurs SOCKETS associés
         │
         ├─ Socket 1 : (serveur:80, client1:54231)
         ├─ Socket 2 : (serveur:80, client2:54232)
         └─ Socket 3 : (serveur:80, client3:54233)

Tous utilisent le port 80, mais sont des sockets distincts
```

## Conclusion

Les ports et les sockets sont les fondations de toute communication réseau :

**Les ports** permettent :
- D'identifier les applications sur une machine
- Le multiplexage (plusieurs applications simultanées)
- La standardisation (port 80 = HTTP partout)

**Les sockets** offrent :
- Une abstraction simple pour les programmeurs
- Une interface uniforme (send/recv) quelle que soit la complexité sous-jacente
- Deux modes : fiable (TCP) ou rapide (UDP)

**Ensemble**, ils rendent possible :
- La navigation web simultanée sur plusieurs sites
- Les serveurs gérant des milliers de clients
- Les applications temps réel comme les jeux en ligne
- L'ensemble de l'écosystème Internet moderne

Dans la section suivante, nous explorerons la classification des ports : bien connus, enregistrés et dynamiques, pour comprendre comment les différents types de ports sont organisés et attribués.

---

**À retenir** :

- ✅ **Port** = numéro (0-65535) identifiant une application
- ✅ **Socket** = interface de programmation pour communiquer
- ✅ Une connexion = (IP source, port source, IP dest, port dest)
- ✅ **Port source** : attribué automatiquement, éphémère
- ✅ **Port destination** : fixe, connu à l'avance
- ✅ **SOCK_STREAM** (TCP) : fiable, orienté connexion
- ✅ **SOCK_DGRAM** (UDP) : rapide, sans connexion
- ✅ Un serveur utilise le même port pour plusieurs clients (sockets distincts)

⏭️ [Ports bien connus, enregistrés et dynamiques](/04-couche-transport/03-ports-categories.md)
