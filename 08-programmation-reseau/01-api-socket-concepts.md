🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.1 L'API Socket : concepts fondamentaux

## Introduction

L'**API Socket** (ou *Berkeley Sockets*) est l'interface de programmation fondamentale pour toute communication réseau. Créée dans les années 1980, elle reste aujourd'hui la base de pratiquement toutes les communications réseau, des simples requêtes HTTP aux systèmes distribués complexes.

Que vous utilisiez Python, Java, C, Go, Rust ou JavaScript, vous utilisez directement ou indirectement cette API. Comprendre ses concepts fondamentaux vous permettra de :

- **Maîtriser les abstractions de haut niveau** (bibliothèques HTTP, frameworks web)
- **Déboguer efficacement** les problèmes réseau
- **Optimiser les performances** de vos applications
- **Concevoir des protocoles personnalisés** si nécessaire

## Qu'est-ce qu'une socket ?

### Définition conceptuelle

Une **socket** est un **point de terminaison de communication** dans un réseau. C'est l'abstraction logicielle qui permet à un processus d'envoyer et de recevoir des données via le réseau.

On peut comparer une socket à :
- **Une prise téléphonique** : elle permet la connexion entre deux parties
- **Une boîte aux lettres** : elle a une adresse et permet d'échanger des messages
- **Un port maritime** : les données arrivent et repartent par ce point précis

### Définition technique

Plus formellement, une socket est définie par une combinaison de :

```
Socket = (Adresse IP, Numéro de port, Protocole)
```

**Exemple concret** :
```
Socket serveur web = (192.168.1.10, 80, TCP)
Socket client = (10.0.0.5, 54321, TCP)
```

Cette combinaison unique identifie de manière non ambiguë un point de communication sur le réseau.

### Socket vs Port : clarification

⚠️ **Attention à la confusion courante** :

- Un **port** est un numéro (0-65535) qui identifie un service sur une machine
- Une **socket** inclut le port mais aussi l'adresse IP et le protocole

**Analogie** : Si votre adresse IP est un immeuble, le port est le numéro d'appartement, et la socket est l'appartement complet avec ses occupants.

## Histoire et origine de l'API Socket

### Berkeley Sockets (1983)

L'API Socket a été développée à l'**Université de Californie à Berkeley** dans le cadre du projet BSD Unix (Berkeley Software Distribution). Elle a été conçue pour fournir une interface de programmation unifiée pour TCP/IP.

**Pourquoi ce design a-t-il perduré ?**

1. **Simplicité conceptuelle** : abstraction claire et intuitive
2. **Flexibilité** : supporte plusieurs protocoles (TCP, UDP, etc.)
3. **Portabilité** : standardisée sur tous les systèmes UNIX/Linux
4. **Performance** : interface directe avec le kernel

### Standardisation POSIX

L'API a été standardisée par POSIX (Portable Operating System Interface), garantissant sa présence sur tous les systèmes UNIX-like (Linux, macOS, BSD, Solaris, etc.).

Windows a créé sa propre variante appelée **Winsock** (Windows Sockets), largement compatible avec l'API POSIX mais avec quelques différences.

## Les types de sockets

L'API Socket supporte plusieurs types de sockets, correspondant à différents protocoles et besoins.

### 1. **SOCK_STREAM** (Sockets de flux)

**Caractéristiques** :
- Orienté **connexion**
- Communication **bidirectionnelle** et **fiable**
- Garantit l'**ordre** des données
- Utilise **TCP** comme protocole de transport

**Analogie** : Un appel téléphonique — connexion établie, conversation bidirectionnelle, ordre garanti.

**Cas d'usage réels** :
- Serveurs web (HTTP/HTTPS)
- Transfert de fichiers (FTP, SFTP)
- Bases de données (MySQL, PostgreSQL)
- SSH, Telnet
- Applications de chat avec historique

**Exemple en Python** :
```python
import socket

# Création d'une socket TCP (SOCK_STREAM)
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Cette socket utilisera TCP automatiquement
```

**Exemple en JavaScript (Node.js)** :
```javascript
const net = require('net');

// Création d'un serveur TCP
const server = net.createServer((socket) => {
    // 'socket' est une SOCK_STREAM
    console.log('Client connecté');
});
```

**Pourquoi choisir SOCK_STREAM ?**
- ✅ Vous avez besoin de **fiabilité** (pas de perte de données)
- ✅ L'**ordre** des messages est crucial
- ✅ Vous transférez des **volumes importants** de données
- ❌ La latence absolue n'est pas critique (overhead de TCP)

### 2. **SOCK_DGRAM** (Sockets de datagrammes)

**Caractéristiques** :
- **Sans connexion**
- Communication **non fiable** (pas de garantie de livraison)
- **Pas d'ordre garanti**
- Utilise **UDP** comme protocole de transport

**Analogie** : Envoyer des cartes postales — pas de garantie de réception, peut arriver dans le désordre.

**Cas d'usage réels** :
- Streaming vidéo/audio (Zoom, Netflix)
- Jeux en ligne multijoueurs
- DNS (requêtes de résolution de noms)
- VoIP (Skype, WhatsApp calls)
- IoT avec capteurs (où la perte occasionnelle est acceptable)
- Diffusion multicast

**Exemple en Python** :
```python
import socket

# Création d'une socket UDP (SOCK_DGRAM)
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Envoi d'un datagramme (pas besoin d'établir une connexion)
sock.sendto(b"Hello", ("192.168.1.100", 9999))
```

**Exemple en Go** :
```go
package main

import (
    "net"
    "fmt"
)

func main() {
    // Socket UDP
    conn, _ := net.DialUDP("udp", nil, &net.UDPAddr{
        IP:   net.ParseIP("192.168.1.100"),
        Port: 9999,
    })

    conn.Write([]byte("Hello"))
}
```

**Pourquoi choisir SOCK_DGRAM ?**
- ✅ **Faible latence** est prioritaire
- ✅ Perte occasionnelle de paquets est **acceptable**
- ✅ Communication **temps réel** (streaming)
- ✅ **Diffusion** vers plusieurs destinataires (multicast/broadcast)
- ❌ Vous ne pouvez pas vous permettre de perdre des données

**Cas d'usage réel : jeux vidéo multijoueurs**

Dans un jeu FPS (First Person Shooter) comme Call of Duty, les positions des joueurs sont envoyées via UDP :
- Si un paquet contenant une position est perdu, le suivant arrivera rapidement
- La latence doit être minimale (< 50ms idéalement)
- Perdre une position occasionnellement n'est pas critique (le joueur continue de bouger)

### 3. **SOCK_RAW** (Sockets brutes)

**Caractéristiques** :
- Accès **direct** aux protocoles de niveau inférieur (IP, ICMP)
- Permet de **créer des paquets personnalisés**
- Nécessite généralement des **privilèges administrateur**

**Cas d'usage réels** :
- Outils de diagnostic réseau (ping, traceroute)
- Scanners de ports (nmap)
- Analyseurs de paquets (Wireshark)
- Firewalls personnalisés
- VPN et tunneling

**Exemple : implémentation de ping en Python** :
```python
import socket
import struct

# Nécessite des privilèges root/admin
sock = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.IPPROTO_ICMP)

# Construction manuelle d'un paquet ICMP Echo Request
# ...
```

⚠️ **Note** : L'utilisation de SOCK_RAW est avancée et rarement nécessaire pour le développement applicatif classique.

## Domaines de communication (Address Family)

Les sockets peuvent opérer dans différents **domaines** (ou *address families*), qui définissent le type d'adresses utilisées.

### AF_INET (IPv4)

Le domaine le plus courant pour les communications réseau Internet.

**Caractéristiques** :
- Adresses IPv4 (32 bits) : `192.168.1.1`
- Utilisé pour la majorité des communications Internet actuelles

**Exemple** :
```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(("93.184.216.34", 80))  # Exemple d'IP
```

### AF_INET6 (IPv6)

Pour les communications utilisant IPv6.

**Caractéristiques** :
- Adresses IPv6 (128 bits) : `2001:0db8:85a3::8a2e:0370:7334`
- De plus en plus utilisé avec l'épuisement des adresses IPv4

**Exemple** :
```python
import socket

sock = socket.socket(socket.AF_INET6, socket.SOCK_STREAM)
sock.connect(("2606:2800:220:1:248:1893:25c8:1946", 80))  # example.com en IPv6
```

**Cas d'usage moderne** : Les services cloud majeurs (AWS, Google Cloud) encouragent l'utilisation d'IPv6, notamment pour les applications conteneurisées.

### AF_UNIX (ou AF_LOCAL)

Pour les communications **locales** entre processus sur la même machine.

**Caractéristiques** :
- Utilise des **chemins de fichiers** comme adresses
- Plus rapide que TCP en local (pas de stack réseau)
- Très utilisé pour les communications inter-processus (IPC)

**Exemple** :
```python
import socket

sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
sock.connect("/tmp/my-app.sock")
```

**Cas d'usage réels** :
- **Docker** : le daemon Docker écoute sur `/var/run/docker.sock`
- **PostgreSQL** : connexions locales via `/var/run/postgresql/.s.PGSQL.5432`
- **Nginx** : communication avec PHP-FPM via sockets UNIX
- **systemd** : notifications et communications de services

**Pourquoi utiliser AF_UNIX plutôt que localhost TCP ?**

```
Comparaison de performance (même machine) :
AF_UNIX : ~100 000 messages/sec
TCP localhost : ~40 000 messages/sec
```

✅ **Avantages** :
- Plus rapide (pas de stack TCP/IP)
- Pas de ports à gérer
- Sécurité via permissions fichiers

❌ **Limitations** :
- Uniquement local (pas de réseau)
- Pas portable vers Windows (utilise Named Pipes)

## Les opérations fondamentales sur les sockets

Quelle que soit le type de socket, certaines opérations sont communes.

### Cycle de vie d'une socket TCP (client)

```
1. socket()     → Création de la socket
2. connect()    → Connexion au serveur
3. send/recv    → Échange de données
4. close()      → Fermeture
```

**Exemple complet en Python** :
```python
import socket

# 1. Création
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. Connexion
client.connect(("example.com", 80))

# 3. Envoi de données
request = b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n"
client.send(request)

# Réception de données
response = client.recv(4096)
print(response.decode())

# 4. Fermeture
client.close()
```

### Cycle de vie d'une socket TCP (serveur)

```
1. socket()     → Création de la socket
2. bind()       → Association à une adresse/port
3. listen()     → Mise en écoute
4. accept()     → Acceptation des connexions (bloquant)
5. send/recv    → Échange avec chaque client
6. close()      → Fermeture
```

**Exemple complet en Python** :
```python
import socket

# 1. Création
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. Liaison à une adresse et un port
server.bind(("0.0.0.0", 8080))

# 3. Mise en écoute (max 5 connexions en attente)
server.listen(5)
print("Serveur en écoute sur le port 8080...")

# 4. Acceptation des connexions (boucle infinie)
while True:
    client_socket, client_address = server.accept()
    print(f"Connexion depuis {client_address}")

    # 5. Communication
    data = client_socket.recv(1024)
    client_socket.send(b"Hello from server")

    # Fermeture de la connexion client
    client_socket.close()
```

### Cycle de vie d'une socket UDP

UDP étant sans connexion, le cycle est simplifié :

```
Client:                  Serveur:
1. socket()              1. socket()
2. sendto()              2. bind()
3. recvfrom()            3. recvfrom()
4. close()               4. sendto()
                         5. close()
```

**Exemple client UDP** :
```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.sendto(b"Hello UDP", ("192.168.1.100", 9999))
response, server = sock.recvfrom(1024)
sock.close()
```

**Exemple serveur UDP** :
```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(("0.0.0.0", 9999))

while True:
    data, client_address = sock.recvfrom(1024)
    print(f"Reçu de {client_address}: {data}")
    sock.sendto(b"ACK", client_address)
```

## Description détaillée des opérations clés

### socket() - Création

Crée un descripteur de socket (similaire à un descripteur de fichier).

**Signature** :
```python
socket(family, type, protocol=0)
```

**Paramètres** :
- `family` : AF_INET, AF_INET6, AF_UNIX
- `type` : SOCK_STREAM, SOCK_DGRAM, SOCK_RAW
- `protocol` : généralement 0 (auto), ou IPPROTO_TCP, IPPROTO_UDP

**Cas d'usage** : Toujours la première étape.

### bind() - Liaison

Associe la socket à une adresse IP et un port spécifiques.

**Quand l'utiliser ?**
- **Serveurs** : toujours (pour écouter sur un port)
- **Clients** : rarement (le système assigne automatiquement)

**Exemple** :
```python
sock.bind(("0.0.0.0", 8080))
# 0.0.0.0 = écoute sur toutes les interfaces réseau
```

**Erreurs courantes** :
```
Address already in use
```
→ Le port est déjà utilisé par un autre processus

**Solution** :
```python
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

### listen() - Mise en écoute (TCP uniquement)

Place la socket en mode passif, prête à accepter des connexions.

**Paramètre** : *backlog* - nombre max de connexions en attente

```python
sock.listen(5)  # Max 5 connexions en file d'attente
```

**Cas d'usage réel** : Un serveur web reçoit 100 requêtes simultanées, mais ne peut en traiter que 10 à la fois. Le *backlog* permet de mettre les autres en attente plutôt que de les rejeter.

### accept() - Acceptation de connexion (TCP uniquement)

**Bloque** jusqu'à ce qu'un client se connecte, puis retourne :
1. Une **nouvelle socket** pour communiquer avec ce client
2. L'**adresse** du client

```python
client_sock, client_addr = server_sock.accept()
print(f"Client: {client_addr[0]}:{client_addr[1]}")
```

⚠️ **Point crucial** : `accept()` crée une **nouvelle socket** pour chaque client. La socket d'origine continue d'écouter.

**Schéma** :
```
Server Socket (port 8080)
    ├── Client Socket 1 (192.168.1.10:54321)
    ├── Client Socket 2 (192.168.1.11:54322)
    └── Client Socket 3 (192.168.1.12:54323)
```

### connect() - Connexion (TCP)

Initie une connexion vers un serveur.

```python
sock.connect(("example.com", 80))
```

**Que se passe-t-il sous le capot ?**
1. Résolution DNS de "example.com"
2. Three-way handshake TCP (SYN, SYN-ACK, ACK)
3. La connexion est établie

**Erreurs courantes** :
- `Connection refused` : aucun serveur n'écoute sur ce port
- `Connection timeout` : serveur injoignable ou firewall
- `Connection reset` : le serveur a fermé la connexion brutalement

### send() / recv() - Envoi et réception (TCP)

**send()** :
```python
bytes_sent = sock.send(b"Hello")
```
⚠️ Retourne le **nombre d'octets réellement envoyés** (peut être < taille du message)

**recv()** :
```python
data = sock.recv(4096)  # Lit jusqu'à 4096 octets
```
⚠️ **Bloque** jusqu'à réception de données (ou timeout)

**Bonne pratique : boucle d'envoi complète** :
```python
def send_all(sock, data):
    total_sent = 0
    while total_sent < len(data):
        sent = sock.send(data[total_sent:])
        if sent == 0:
            raise RuntimeError("Socket connection broken")
        total_sent += sent
```

### sendto() / recvfrom() - Envoi et réception (UDP)

Pour UDP, chaque datagramme inclut l'adresse de destination/source :

```python
# Envoi
sock.sendto(b"Data", ("192.168.1.100", 9999))

# Réception
data, sender_address = sock.recvfrom(1024)
print(f"Reçu de {sender_address}: {data}")
```

### close() - Fermeture

Libère les ressources de la socket.

```python
sock.close()
```

**Bonne pratique : utiliser un context manager** :
```python
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.connect(("example.com", 80))
    sock.send(b"GET / HTTP/1.1\r\n\r\n")
    data = sock.recv(4096)
# sock.close() appelé automatiquement
```

## Options de socket (setsockopt / getsockopt)

Les sockets ont de nombreuses options configurables.

### SO_REUSEADDR

Permet de réutiliser une adresse immédiatement après fermeture :

```python
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

**Cas d'usage** : Éviter "Address already in use" lors du redémarrage d'un serveur.

### SO_KEEPALIVE

Active les messages TCP keep-alive pour détecter les connexions mortes :

```python
sock.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)
```

**Cas d'usage réel** : Un serveur de chat doit détecter si un client s'est déconnecté sans fermer proprement la socket.

### SO_RCVBUF / SO_SNDBUF

Taille des buffers de réception/envoi :

```python
sock.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 65536)  # 64 KB
```

**Cas d'usage** : Applications haute performance nécessitant de gros buffers.

### TCP_NODELAY

Désactive l'algorithme de Nagle (TCP) :

```python
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)
```

**L'algorithme de Nagle** attend d'accumuler des données avant d'envoyer, réduisant le nombre de petits paquets. Utile pour des gros transferts, problématique pour des applications temps réel.

**Cas d'usage** : Jeux en ligne, trading haute fréquence où chaque milliseconde compte.

## Modes bloquant vs non-bloquant (aperçu)

Par défaut, les opérations socket sont **bloquantes** :

```python
data = sock.recv(1024)  # Bloque jusqu'à réception de données
```

On peut passer en **mode non-bloquant** :

```python
sock.setblocking(False)
# ou
sock.settimeout(5.0)  # Timeout de 5 secondes
```

**Impact** :
- Mode bloquant : le thread s'arrête en attendant
- Mode non-bloquant : retourne immédiatement (erreur si pas de données)

Nous approfondirons ces concepts dans les sections sur le multiplexage I/O (8.5-8.6).

## Gestion des erreurs

Les opérations socket peuvent échouer pour de nombreuses raisons.

### Exceptions courantes (Python)

```python
import socket

try:
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(("example.com", 80))
except socket.gaierror as e:
    # Erreur de résolution DNS
    print(f"Erreur DNS: {e}")
except socket.timeout:
    # Timeout de connexion
    print("Timeout")
except ConnectionRefusedError:
    # Aucun serveur n'écoute
    print("Connexion refusée")
except OSError as e:
    # Autres erreurs réseau
    print(f"Erreur réseau: {e}")
finally:
    sock.close()
```

### Codes d'erreur système

Sur Linux/UNIX, les erreurs sont indiquées par `errno` :

- `ECONNREFUSED` : Connection refused
- `ETIMEDOUT` : Timeout
- `EHOSTUNREACH` : Host unreachable
- `EADDRINUSE` : Address already in use

## Exemple complet : serveur HTTP minimaliste

Mettons en pratique ces concepts avec un serveur HTTP basique :

```python
import socket

def handle_request(client_socket):
    """Traite une requête HTTP simple"""
    request = client_socket.recv(4096).decode()
    print(f"Requête reçue:\n{request}")

    # Réponse HTTP simple
    response = (
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html\r\n"
        "\r\n"
        "<html><body><h1>Hello from Socket API!</h1></body></html>"
    )

    client_socket.send(response.encode())
    client_socket.close()

def main():
    # Création de la socket serveur
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    # Liaison au port 8080
    server.bind(("0.0.0.0", 8080))
    server.listen(5)

    print("Serveur HTTP en écoute sur http://localhost:8080")

    try:
        while True:
            # Acceptation des connexions
            client_socket, client_address = server.accept()
            print(f"Connexion de {client_address}")

            # Traitement de la requête
            handle_request(client_socket)

    except KeyboardInterrupt:
        print("\nArrêt du serveur...")
    finally:
        server.close()

if __name__ == "__main__":
    main()
```

**Test** :
```bash
python server.py
# Dans un autre terminal :
curl http://localhost:8080
```

**Limitations** :
- ⚠️ Un seul client à la fois (pas de concurrence)
- ⚠️ Pas de timeout
- ⚠️ Pas de gestion d'erreurs robuste

Nous améliorerons cela dans les sections suivantes.

## Cas d'usage réel : architecture d'un CDN

Les **CDN** (Content Delivery Networks) comme Cloudflare ou Akamai utilisent massivement l'API Socket :

1. **Sockets d'écoute multiples** : Un serveur edge écoute sur les ports 80 (HTTP) et 443 (HTTPS)
2. **Pools de connexions** : Maintiennent des milliers de sockets ouvertes simultanément
3. **Options optimisées** : TCP_NODELAY, SO_REUSEADDR, buffers ajustés
4. **Multiplexage I/O** : epoll/kqueue pour gérer 100 000+ connexions par serveur

```python
# Pseudo-code simplifié d'un serveur edge CDN
import socket
import select

# Socket HTTP
http_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
http_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
http_sock.bind(("0.0.0.0", 80))
http_sock.listen(1024)  # Backlog élevé

# Socket HTTPS
https_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
https_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
https_sock.bind(("0.0.0.0", 443))
https_sock.listen(1024)

# Multiplexage avec epoll (Linux) pour gérer des milliers de connexions
epoll = select.epoll()
epoll.register(http_sock.fileno(), select.EPOLLIN)
epoll.register(https_sock.fileno(), select.EPOLLIN)

# Boucle événementielle
while True:
    events = epoll.poll()
    for fd, event in events:
        # Traitement des événements...
```

## Récapitulatif des concepts clés

| Concept | Description | Importance |
|---------|-------------|------------|
| **Socket** | Point de terminaison de communication | Fondamental |
| **SOCK_STREAM** | TCP - fiable, orienté connexion | Applications critiques |
| **SOCK_DGRAM** | UDP - rapide, non fiable | Temps réel, streaming |
| **AF_INET/AF_INET6** | Domaines IPv4/IPv6 | Réseau Internet |
| **AF_UNIX** | Communication locale IPC | Performance locale |
| **bind()** | Associe socket à adresse/port | Serveurs |
| **listen()** | Met en écoute (TCP) | Serveurs TCP |
| **accept()** | Accepte connexions entrantes | Serveurs TCP |
| **connect()** | Initie connexion (TCP) | Clients TCP |
| **send/recv** | Échange de données (TCP) | Communication |
| **sendto/recvfrom** | Échange de données (UDP) | Communication |

## Points clés à retenir

✅ **L'API Socket est universelle** : même concepts dans tous les langages

✅ **TCP (SOCK_STREAM)** : fiabilité, ordre, connexion → applications critiques

✅ **UDP (SOCK_DGRAM)** : vitesse, sans connexion → temps réel, streaming

✅ **Client vs Serveur** : cycles de vie différents (bind/listen/accept pour serveur)

✅ **Options socket** : SO_REUSEADDR, TCP_NODELAY, etc. pour optimiser

✅ **Gestion d'erreurs** : indispensable (réseau = imprévisible)

## Prochaines étapes

Maintenant que vous maîtrisez les concepts fondamentaux, nous allons approfondir :

- **Section 8.2** : Sockets TCP en détail (connexion, échange, fermeture)
- **Section 8.3** : Sockets UDP et leurs particularités
- **Section 8.4** : Gestion robuste des erreurs réseau
- **Section 8.5-8.6** : I/O non-bloquant et multiplexage pour la performance

---


⏭️ [Sockets TCP : création, connexion, échange, fermeture](/08-programmation-reseau/02-sockets-tcp.md)
