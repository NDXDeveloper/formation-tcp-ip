🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Exemples de code socket

## Introduction

Cette section présente des exemples pratiques de programmation réseau utilisant l'**API Socket**. Bien que ce tutoriel se concentre sur les aspects théoriques de TCP/IP, comprendre comment les applications utilisent concrètement ces protocoles est essentiel pour avoir une vision complète.

**Objectif de cette annexe :**
- Illustrer les concepts TCP/IP à travers du code
- Montrer comment les applications interagissent avec la couche transport
- Fournir des exemples de référence pour vos propres projets
- Démontrer les différences entre TCP et UDP en pratique

**⚠️ Note importante :** Ces exemples sont pédagogiques et simplifiés. Pour du code de production, des considérations supplémentaires de sécurité, de gestion d'erreurs et de performance sont nécessaires.

---

## Qu'est-ce que l'API Socket ?

### Définition

Une **socket** (prise réseau) est une interface de programmation permettant aux applications de communiquer via le réseau. C'est le point de terminaison d'une communication bidirectionnelle entre deux programmes.

**Métaphore :** Si le réseau est le système téléphonique, une socket est comme une prise téléphonique permettant de brancher un appareil pour communiquer.

### Identification d'une socket

Une socket est identifiée par une combinaison unique :

```
Socket = (Adresse IP, Numéro de port, Protocole)
```

**Exemple de socket :**
```
IP: 192.168.1.10
Port: 8080
Protocole: TCP
```

### Types de sockets principaux

| Type | Protocole | Caractéristiques |
|------|-----------|------------------|
| **SOCK_STREAM** | TCP | Orienté connexion, fiable, flux d'octets |
| **SOCK_DGRAM** | UDP | Sans connexion, non fiable, datagrammes |
| **SOCK_RAW** | IP brut | Accès direct à IP, nécessite privilèges root |

---

## Architecture Client-Serveur

La plupart des applications réseau suivent le modèle **client-serveur** :

### Rôle du serveur

```
1. Créer une socket
2. Lier (bind) la socket à une adresse IP et un port
3. Écouter (listen) les connexions entrantes
4. Accepter (accept) les connexions
5. Échanger des données
6. Fermer la connexion
```

### Rôle du client

```
1. Créer une socket
2. Se connecter (connect) au serveur
3. Échanger des données
4. Fermer la connexion
```

### Diagramme de flux TCP

```
SERVEUR                                CLIENT
   |                                      |
   | socket()                             |
   | bind()                               |
   | listen()                             |
   |                                      | socket()
   | accept() -------- BLOQUÉ             |
   |                                      | connect() ----→
   | ←---- 3-WAY HANDSHAKE (SYN,SYN-ACK,ACK) ----→
   |                                      |
   | accept() retourne                    | connect() retourne
   |                                      |
   | recv()/send() ←------ DONNÉES -----→ recv()/send()
   |                                      |
   | close() ←--------- FIN/ACK --------→ close()
   |                                      |
```

---

## Organisation des exemples

Les exemples sont organisés par langage de programmation :

### Structure des dossiers

```
05-exemples-code/
├── README.md (ce fichier)
├── python/
│   ├── README.md
│   ├── 01-tcp-simple/
│   ├── 02-tcp-multi-clients/
│   ├── 03-udp/
│   ├── 04-http-simple/
│   └── 05-chat/
├── go/
│   ├── README.md
│   ├── 01-tcp-simple/
│   ├── 02-concurrent-server/
│   ├── 03-udp/
│   └── 04-websocket/
└── javascript/
    ├── README.md
    ├── 01-node-tcp/
    ├── 02-node-udp/
    ├── 03-websocket/
    └── 04-http-server/
```

### Niveaux de difficulté

- **Niveau 1** : Exemples basiques (echo server, client simple)
- **Niveau 2** : Gestion multi-clients, threads/async
- **Niveau 3** : Protocoles applicatifs, gestion d'erreurs avancée
- **Niveau 4** : Applications complètes (chat, proxy, etc.)

---

## Concepts fondamentaux illustrés

### 1. Socket TCP simple - Concept général

#### Serveur TCP minimal (pseudo-code)

```
// Créer une socket TCP
socket = create_socket(TCP)

// Lier à une adresse et un port
bind(socket, address="0.0.0.0", port=8080)

// Écouter les connexions (backlog=5)
listen(socket, backlog=5)

// Boucle infinie
while true:
    // Accepter une connexion (bloquant)
    client_socket = accept(socket)

    // Recevoir des données
    data = receive(client_socket, buffer_size=1024)

    // Traiter et répondre
    response = process(data)
    send(client_socket, response)

    // Fermer la connexion client
    close(client_socket)
```

#### Client TCP minimal (pseudo-code)

```
// Créer une socket TCP
socket = create_socket(TCP)

// Se connecter au serveur
connect(socket, address="192.168.1.10", port=8080)

// Envoyer des données
send(socket, "Hello, Server!")

// Recevoir la réponse
response = receive(socket, buffer_size=1024)

// Afficher la réponse
print(response)

// Fermer la connexion
close(socket)
```

---

### 2. Socket UDP simple - Concept général

#### Serveur UDP minimal (pseudo-code)

```
// Créer une socket UDP
socket = create_socket(UDP)

// Lier à une adresse et un port
bind(socket, address="0.0.0.0", port=8080)

// Boucle infinie
while true:
    // Recevoir un datagramme (bloquant)
    data, client_address = receive_from(socket, buffer_size=1024)

    // Traiter et répondre
    response = process(data)
    send_to(socket, response, client_address)
```

**Note :** Pas de connexion établie, chaque datagramme est indépendant.

#### Client UDP minimal (pseudo-code)

```
// Créer une socket UDP
socket = create_socket(UDP)

// Envoyer un datagramme (pas de connexion préalable)
server_address = ("192.168.1.10", 8080)
send_to(socket, "Hello, Server!", server_address)

// Recevoir la réponse
response, server_address = receive_from(socket, buffer_size=1024)

// Afficher la réponse
print(response)

// Fermer la socket
close(socket)
```

---

## Exemples comparatifs par langage

### Python - Echo Server TCP

```python
import socket

# Créer une socket TCP/IP
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Lier la socket au port
server_address = ('localhost', 8080)
server_socket.bind(server_address)

# Écouter les connexions entrantes
server_socket.listen(1)
print(f"Serveur en écoute sur {server_address}")

while True:
    # Attendre une connexion
    print("En attente d'une connexion...")
    client_socket, client_address = server_socket.accept()

    try:
        print(f"Connexion de {client_address}")

        # Recevoir les données
        data = client_socket.recv(1024)
        print(f"Reçu: {data.decode()}")

        if data:
            # Renvoyer les données (echo)
            client_socket.sendall(data)

    finally:
        # Fermer la connexion client
        client_socket.close()
```

**Points clés Python :**
- `socket.AF_INET` : famille d'adresses IPv4
- `socket.SOCK_STREAM` : socket TCP
- `bind()` : attache la socket à une adresse
- `listen(1)` : backlog de 1 connexion en attente
- `accept()` : bloquant, retourne une nouvelle socket pour le client
- `recv(1024)` : lit jusqu'à 1024 octets

---

### Go - Echo Server TCP

```go
package main

import (
    "fmt"
    "net"
    "io"
)

func main() {
    // Écouter sur le port 8080
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        fmt.Println("Erreur:", err)
        return
    }
    defer listener.Close()

    fmt.Println("Serveur en écoute sur :8080")

    for {
        // Accepter une connexion
        conn, err := listener.Accept()
        if err != nil {
            fmt.Println("Erreur accept:", err)
            continue
        }

        fmt.Println("Connexion de", conn.RemoteAddr())

        // Gérer la connexion
        go handleConnection(conn)
    }
}

func handleConnection(conn net.Conn) {
    defer conn.Close()

    // Echo: copier ce qui est reçu vers la sortie
    io.Copy(conn, conn)
}
```

**Points clés Go :**
- `net.Listen()` : crée un listener TCP
- `listener.Accept()` : bloquant, retourne une connexion
- `go handleConnection(conn)` : goroutine pour gestion concurrente
- `io.Copy()` : copie efficace entre lecteur et écrivain
- Gestion d'erreur idiomatique Go

---

### JavaScript/Node.js - Echo Server TCP

```javascript
const net = require('net');

// Créer un serveur TCP
const server = net.createServer((socket) => {
    console.log('Client connecté:', socket.remoteAddress);

    // Événement: données reçues
    socket.on('data', (data) => {
        console.log('Reçu:', data.toString());
        // Echo: renvoyer les données
        socket.write(data);
    });

    // Événement: connexion fermée
    socket.on('end', () => {
        console.log('Client déconnecté');
    });

    // Événement: erreur
    socket.on('error', (err) => {
        console.error('Erreur socket:', err);
    });
});

// Écouter sur le port 8080
server.listen(8080, () => {
    console.log('Serveur en écoute sur le port 8080');
});
```

**Points clés Node.js :**
- `net.createServer()` : crée un serveur TCP
- Modèle événementiel (on 'data', 'end', 'error')
- Asynchrone par défaut (non-bloquant)
- `socket.write()` : envoie des données
- Gestion d'événements pour le cycle de vie de la connexion

---

## Patterns communs de programmation socket

### 1. Gestion des connexions multiples

**Problème :** Un serveur doit gérer plusieurs clients simultanément.

**Solutions :**

#### A. Multi-threading (Python)

```python
import socket
import threading

def handle_client(client_socket, address):
    print(f"Nouvelle connexion: {address}")
    try:
        while True:
            data = client_socket.recv(1024)
            if not data:
                break
            client_socket.sendall(data)
    finally:
        client_socket.close()

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('localhost', 8080))
server_socket.listen(5)

while True:
    client_socket, address = server_socket.accept()
    # Créer un thread par client
    client_thread = threading.Thread(
        target=handle_client,
        args=(client_socket, address)
    )
    client_thread.start()
```

#### B. Asynchrone (Python avec asyncio)

```python
import asyncio

async def handle_client(reader, writer):
    addr = writer.get_extra_info('peername')
    print(f"Connexion de {addr}")

    while True:
        data = await reader.read(1024)
        if not data:
            break
        writer.write(data)
        await writer.drain()

    writer.close()
    await writer.wait_closed()

async def main():
    server = await asyncio.start_server(
        handle_client, 'localhost', 8080
    )
    async with server:
        await server.serve_forever()

asyncio.run(main())
```

#### C. Goroutines (Go)

```go
func main() {
    listener, _ := net.Listen("tcp", ":8080")

    for {
        conn, _ := listener.Accept()
        go handleConnection(conn) // Goroutine légère
    }
}
```

---

### 2. Protocole avec délimiteurs

**Problème :** TCP est un flux d'octets sans délimitation de messages.

**Solution :** Utiliser un délimiteur (ex: newline `\n`)

```python
import socket

def send_message(sock, message):
    """Envoie un message délimité par newline"""
    sock.sendall((message + '\n').encode())

def receive_message(sock):
    """Reçoit un message jusqu'au newline"""
    buffer = ""
    while True:
        chunk = sock.recv(1)
        if not chunk:
            return None
        char = chunk.decode()
        if char == '\n':
            return buffer
        buffer += char

# Utilisation
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('localhost', 8080))

send_message(sock, "Hello")
response = receive_message(sock)
print(f"Réponse: {response}")
```

---

### 3. Protocole avec longueur préfixée

**Problème :** Délimiteurs insuffisants pour données binaires.

**Solution :** Préfixer chaque message avec sa longueur.

```python
import struct

def send_message(sock, data):
    """Envoie données préfixées par leur longueur (4 bytes)"""
    # Préfixe: longueur sur 4 octets (big-endian)
    length = len(data)
    prefix = struct.pack('>I', length)
    sock.sendall(prefix + data)

def receive_message(sock):
    """Reçoit données préfixées par leur longueur"""
    # Lire le préfixe (4 octets)
    prefix = receive_exact(sock, 4)
    if not prefix:
        return None

    # Extraire la longueur
    length = struct.unpack('>I', prefix)[0]

    # Lire exactement length octets
    return receive_exact(sock, length)

def receive_exact(sock, n):
    """Reçoit exactement n octets"""
    data = b''
    while len(data) < n:
        chunk = sock.recv(n - len(data))
        if not chunk:
            return None
        data += chunk
    return data
```

**Explication :**
- `struct.pack('>I', length)` : encode la longueur sur 4 octets, big-endian
- `>` : big-endian (network byte order)
- `I` : unsigned int (4 octets)

---

### 4. Timeouts

**Problème :** Opérations bloquantes peuvent attendre indéfiniment.

**Solution :** Définir des timeouts.

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Timeout de connexion : 5 secondes
sock.settimeout(5.0)

try:
    sock.connect(('example.com', 80))
except socket.timeout:
    print("Connexion timeout")
except socket.error as e:
    print(f"Erreur: {e}")

# Timeout de lecture : 10 secondes
sock.settimeout(10.0)

try:
    data = sock.recv(1024)
except socket.timeout:
    print("Lecture timeout")
```

---

### 5. Gestion propre des erreurs

**Erreurs courantes :**

| Exception | Cause |
|-----------|-------|
| `ConnectionRefusedError` | Aucun serveur n'écoute sur ce port |
| `TimeoutError` | Opération trop longue |
| `BrokenPipeError` | L'autre côté a fermé la connexion |
| `OSError: Address already in use` | Port déjà utilisé |

**Bonnes pratiques :**

```python
import socket
import time

def connect_with_retry(host, port, max_retries=3):
    """Connexion avec retry automatique"""
    for attempt in range(max_retries):
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.connect((host, port))
            return sock
        except ConnectionRefusedError:
            print(f"Tentative {attempt + 1} échouée, retry...")
            time.sleep(2 ** attempt)  # Backoff exponentiel
        except Exception as e:
            print(f"Erreur: {e}")
            return None

    print("Échec après toutes les tentatives")
    return None
```

---

## Différences TCP vs UDP en code

### Comparaison côte à côte

#### Serveur Echo TCP

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(('localhost', 8080))
sock.listen(1)

while True:
    conn, addr = sock.accept()
    data = conn.recv(1024)
    conn.sendall(data)  # Echo
    conn.close()
```

#### Serveur Echo UDP

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(('localhost', 8080))

while True:
    data, addr = sock.recvfrom(1024)
    sock.sendto(data, addr)  # Echo
```

### Différences clés

| Aspect | TCP | UDP |
|--------|-----|-----|
| **Type** | `SOCK_STREAM` | `SOCK_DGRAM` |
| **Connexion** | `listen()`, `accept()` | Aucune connexion |
| **Réception** | `recv()` retourne bytes | `recvfrom()` retourne (bytes, address) |
| **Envoi** | `send()` / `sendall()` | `sendto(data, address)` |
| **État** | Connexion établie | Sans état |
| **Fiabilité** | Garantie | Aucune garantie |

---

## Options de socket courantes

### SO_REUSEADDR

**Problème :** Après arrêt d'un serveur, le port reste bloqué quelques minutes (TIME_WAIT).

**Solution :**

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Permet de réutiliser immédiatement le port
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

sock.bind(('localhost', 8080))
```

### TCP_NODELAY

**Problème :** Algorithme de Nagle retarde l'envoi de petits paquets.

**Solution :** Désactiver Nagle pour applications temps réel.

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('example.com', 8080))

# Désactiver l'algorithme de Nagle
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)
```

### SO_KEEPALIVE

**Utilité :** Détecter les connexions mortes.

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Activer keep-alive
sock.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)

# Configurer les paramètres (Linux)
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPIDLE, 60)
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPINTVL, 10)
sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPCNT, 3)
```

---

## Sérialisation des données

### Formats courants

#### 1. JSON (texte, lisible)

```python
import socket
import json

# Envoi
data = {"message": "Hello", "id": 123}
sock.sendall(json.dumps(data).encode() + b'\n')

# Réception
raw_data = sock.recv(1024)
data = json.loads(raw_data.decode())
```

**Avantages :** Lisible, debuggable, universel
**Inconvénients :** Verbeux, plus lent

#### 2. Protocol Buffers (binaire, efficace)

```python
import socket
import message_pb2  # Généré depuis .proto

# Envoi
msg = message_pb2.Message()
msg.text = "Hello"
msg.id = 123
serialized = msg.SerializeToString()

length = len(serialized)
sock.sendall(struct.pack('>I', length) + serialized)

# Réception
length_data = sock.recv(4)
length = struct.unpack('>I', length_data)[0]
serialized = sock.recv(length)

msg = message_pb2.Message()
msg.ParseFromString(serialized)
```

**Avantages :** Compact, rapide, typé
**Inconvénients :** Nécessite définition de schéma

#### 3. MessagePack (binaire, simple)

```python
import socket
import msgpack

# Envoi
data = {"message": "Hello", "id": 123}
packed = msgpack.packb(data)
sock.sendall(packed)

# Réception
packed = sock.recv(1024)
data = msgpack.unpackb(packed)
```

**Avantages :** Compact, pas de schéma requis
**Inconvénients :** Moins compact que Protobuf

---

## Exemples d'applications complètes

### Chat TCP simple (architecture)

**Serveur :**
```
1. Maintenir une liste de clients connectés
2. Pour chaque message reçu:
   - Identifier l'émetteur
   - Broadcaster à tous les autres clients
3. Gérer déconnexions
```

**Client :**
```
1. Se connecter au serveur
2. Thread 1: Lire input utilisateur et envoyer
3. Thread 2: Recevoir messages et afficher
```

### Proxy HTTP simple (architecture)

```
1. Écouter connexions clients
2. Pour chaque requête HTTP:
   - Parser la requête
   - Établir connexion au serveur cible
   - Relayer la requête
   - Relayer la réponse au client
3. Logger les requêtes
```

### Service de découverte UDP (architecture)

```
Serveur:
1. Écouter sur broadcast UDP
2. Répondre aux requêtes de découverte

Client:
1. Envoyer requête en broadcast
2. Collecter les réponses (timeout)
3. Afficher services disponibles
```

---

## Considérations de sécurité

### 1. Validation des entrées

**Toujours valider ce qui vient du réseau :**

```python
def handle_message(data):
    # Limiter la taille
    if len(data) > MAX_MESSAGE_SIZE:
        raise ValueError("Message trop grand")

    # Valider le format
    try:
        msg = json.loads(data)
    except json.JSONDecodeError:
        raise ValueError("JSON invalide")

    # Valider le contenu
    if 'type' not in msg or msg['type'] not in ALLOWED_TYPES:
        raise ValueError("Type de message invalide")

    return msg
```

### 2. Rate limiting

```python
from collections import defaultdict
import time

class RateLimiter:
    def __init__(self, max_requests, window_seconds):
        self.max_requests = max_requests
        self.window = window_seconds
        self.requests = defaultdict(list)

    def allow_request(self, client_id):
        now = time.time()

        # Nettoyer les anciennes requêtes
        self.requests[client_id] = [
            req_time for req_time in self.requests[client_id]
            if now - req_time < self.window
        ]

        # Vérifier la limite
        if len(self.requests[client_id]) >= self.max_requests:
            return False

        self.requests[client_id].append(now)
        return True
```

### 3. Authentification simple

```python
import hashlib
import secrets

def generate_token():
    return secrets.token_hex(32)

def verify_token(provided_token, stored_token):
    # Comparaison à temps constant (évite timing attacks)
    return secrets.compare_digest(provided_token, stored_token)
```

---

## Debugging et outils

### 1. Logging

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

def handle_client(sock, addr):
    logger.info(f"Nouvelle connexion: {addr}")

    try:
        data = sock.recv(1024)
        logger.debug(f"Reçu {len(data)} octets de {addr}")
        # ...
    except Exception as e:
        logger.error(f"Erreur avec {addr}: {e}", exc_info=True)
```

### 2. Capture réseau intégrée

```python
def log_packet(data, direction="SEND"):
    """Affiche un paquet en hexadecimal"""
    hex_data = data.hex()
    print(f"{direction}: {hex_data}")
    print(f"{direction} (ASCII): {data.decode('ascii', errors='replace')}")

# Utilisation
data = b"Hello, World!"
log_packet(data, "SEND")

response = sock.recv(1024)
log_packet(response, "RECV")
```

### 3. Tests unitaires

```python
import unittest
import socket

class TestEchoServer(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        # Démarrer le serveur dans un thread
        cls.server_thread = start_server_thread()
        time.sleep(0.5)  # Attendre le démarrage

    def test_echo(self):
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect(('localhost', 8080))

        message = b"Test message"
        sock.sendall(message)
        response = sock.recv(1024)

        self.assertEqual(response, message)
        sock.close()
```

---

## Ressources par langage

### Python
- Documentation officielle : `socket` module
- Asyncio pour programmation asynchrone
- Bibliothèques : `socketserver`, `asyncio`

### Go
- Package `net` standard
- Excellente gestion de la concurrence (goroutines)
- Bibliothèques : `gorilla/websocket`, `gRPC`

### JavaScript/Node.js
- Module `net` pour TCP/UDP
- Module `http`/`https` pour HTTP
- Bibliothèques : `socket.io`, `ws` (WebSocket)

---

## Prochaines étapes

Explorez les sous-sections pour des exemples détaillés :

1. **[Python Examples](python/README.md)** 🟢🟡
   - Exemples complets avec explications ligne par ligne
   - TCP, UDP, HTTP, Chat
   - Approches synchrone et asynchrone

2. **[Go Examples](go/README.md)** 🟡🔴
   - Serveurs concurrents performants
   - Patterns Go idiomatiques
   - WebSocket et gRPC

3. **[JavaScript Examples](javascript/README.md)** 🟢🟡
   - Node.js pour backend
   - WebSocket pour temps réel
   - Programmation événementielle

---

## Conseils pour l'apprentissage

### Progression suggérée

1. **Commencer simple :** Echo server TCP monothread
2. **Ajouter complexité :** Multi-clients avec threads
3. **Protocole applicatif :** Implémenter un protocole simple
4. **Asynchrone :** Découvrir async/await ou goroutines
5. **Projet complet :** Chat, proxy, ou service de votre choix

### Expérimentation

**Essayez de :**
- Capturer le trafic avec Wireshark pendant vos tests
- Simuler des pannes réseau (déconnexion brutale)
- Mesurer les performances (requêtes/seconde)
- Implémenter des protocoles existants (HTTP simple, SMTP basique)

### Erreurs courantes à éviter

1. **Ne pas vérifier les valeurs de retour** de `recv()`
2. **Supposer que `recv()` retourne tout le message** (TCP = flux!)
3. **Oublier de fermer les sockets** (fuites de descripteurs)
4. **Bloquer le thread principal** avec des opérations I/O
5. **Ignorer les erreurs** réseau

---

## Conclusion

La programmation socket est la façon dont les applications **utilisent réellement** TCP/IP. Ces exemples vous montrent :

- Comment les concepts théoriques (3-way handshake, ports, etc.) se traduisent en code
- Les différences pratiques entre TCP et UDP
- Les défis réels de la programmation réseau
- Les patterns de conception courants

**Prochaine étape :** Explorez les exemples détaillés dans votre langage préféré et expérimentez !

---


*Ces exemples sont fournis à des fins pédagogiques. Pour du code de production, consultez les bonnes pratiques de sécurité et de performance spécifiques à votre langage et cas d'usage.*

⏭️ [E.1 Exemples Python](/annexes/05-exemples-code/python/README.md)
