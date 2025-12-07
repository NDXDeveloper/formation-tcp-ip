🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.2 Sockets TCP : création, connexion, échange, fermeture

## Introduction

Les **sockets TCP** (SOCK_STREAM) sont au cœur de la plupart des applications réseau modernes. De votre navigateur web à votre client de messagerie, en passant par les API REST et les bases de données, TCP est partout où la **fiabilité** prime sur la vitesse absolue.

Cette section explore en profondeur le cycle de vie complet d'une socket TCP : de sa création à sa fermeture, en passant par l'établissement de connexion et l'échange de données. Nous verrons comment TCP garantit la fiabilité, comment gérer les erreurs, et quelles sont les meilleures pratiques pour des applications robustes.

## Rappel : pourquoi TCP ?

Avant de plonger dans le code, rappelons les garanties offertes par TCP :

- ✅ **Fiabilité** : Tous les octets envoyés arrivent (ou vous êtes notifié de l'échec)
- ✅ **Ordre** : Les données arrivent dans l'ordre d'envoi
- ✅ **Détection d'erreurs** : Checksums pour détecter la corruption
- ✅ **Contrôle de flux** : Adaptation au débit du récepteur
- ✅ **Contrôle de congestion** : Adaptation aux conditions réseau

- ❌ **Coût** : Overhead protocolaire, latence du handshake

## Cycle de vie d'une connexion TCP

### Vue d'ensemble

```
CLIENT                                  SERVEUR
------                                  -------

socket()                                socket()
   |                                       |
   |                                    bind()
   |                                       |
   |                                    listen()
   |                                       |
connect() -------- SYN --------->      accept()
   |        <----- SYN-ACK ------          |
   |        ------- ACK --------->         |
   |                                       |
   |         [CONNEXION ÉTABLIE]           |
   |                                       |
send() -------- données -------->      recv()
   |                                       |
recv() <------- données ---------      send()
   |                                       |
close() ------- FIN --------->             |
   |        <----- FIN-ACK ------          |
   |        ------- ACK --------->      close()
   |                                       |
   X         [CONNEXION FERMÉE]           X
```

### Les trois phases

1. **Établissement** : 3-way handshake (SYN, SYN-ACK, ACK)
2. **Transfert de données** : Échange bidirectionnel
3. **Terminaison** : 4-way handshake (FIN, ACK, FIN, ACK)

## Phase 1 : Création et établissement de connexion

### Côté serveur : préparation à l'écoute

#### Étape 1 : socket() - Création

```python
import socket

# Création d'une socket TCP/IPv4
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

**Que se passe-t-il ?**
- Le noyau alloue une structure de données pour la socket
- Un descripteur de fichier est retourné (sur UNIX/Linux)
- Aucune communication réseau n'a encore eu lieu

**Équivalent en Go** :
```go
package main

import (
    "net"
    "log"
)

func main() {
    // Création implicite via Listen
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatal(err)
    }
    defer listener.Close()
}
```

**Équivalent en Node.js** :
```javascript
const net = require('net');

// Création et configuration en une seule fois
const server = net.createServer((socket) => {
    // Gestion des connexions
});
```

#### Étape 2 : setsockopt() - Configuration (optionnelle mais recommandée)

```python
# Permet de réutiliser l'adresse immédiatement après fermeture
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

**Cas d'usage réel** : Sans SO_REUSEADDR, après un arrêt brutal (Ctrl+C), vous devez attendre environ 60 secondes (TIME_WAIT) avant de relancer le serveur sur le même port.

```bash
# Première exécution
python server.py
^C

# Immédiatement après
python server.py
# OSError: [Errno 48] Address already in use

# Avec SO_REUSEADDR, cela fonctionne immédiatement
```

**Autres options utiles** :

```python
# Activer TCP keep-alive pour détecter les connexions mortes
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)

# Définir la taille du buffer de réception (pour haute performance)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 262144)  # 256 KB
```

#### Étape 3 : bind() - Liaison à une adresse

```python
# Écouter sur toutes les interfaces, port 8080
server_socket.bind(('0.0.0.0', 8080))

# Ou uniquement localhost
server_socket.bind(('127.0.0.1', 8080))

# Ou une interface spécifique
server_socket.bind(('192.168.1.10', 8080))
```

**Anatomie de bind()** :
```python
bind((address, port))
#     ^^^^^^^^  ^^^^
#     |         └─ Port (0-65535)
#     └─ Adresse IP ou '0.0.0.0' pour toutes les interfaces
```

**Choix de l'adresse** :

| Adresse | Signification | Cas d'usage |
|---------|---------------|-------------|
| `0.0.0.0` | Toutes les interfaces | Production (accessible de l'extérieur) |
| `127.0.0.1` | Localhost uniquement | Développement, sécurité |
| `192.168.1.10` | Interface spécifique | Multi-homing, isolement |

**Erreurs courantes** :

```python
# Erreur 1 : Port < 1024 sans privilèges
server_socket.bind(('0.0.0.0', 80))
# PermissionError: [Errno 13] Permission denied

# Solution : utiliser sudo ou un port > 1024

# Erreur 2 : Port déjà utilisé
server_socket.bind(('0.0.0.0', 8080))
# OSError: [Errno 48] Address already in use

# Solution : SO_REUSEADDR ou changer de port
```

**Cas d'usage réel : serveur multi-interface**

Un serveur avec plusieurs interfaces réseau peut choisir sur laquelle écouter :

```python
import socket
import netifaces

# Lister toutes les interfaces
for interface in netifaces.interfaces():
    addrs = netifaces.ifaddresses(interface)
    if netifaces.AF_INET in addrs:
        for addr in addrs[netifaces.AF_INET]:
            print(f"Interface {interface}: {addr['addr']}")

# Écouter uniquement sur l'interface VPN
server_socket.bind(('10.8.0.1', 8080))
```

#### Étape 4 : listen() - Passage en mode écoute

```python
# Backlog de 5 connexions en attente
server_socket.listen(5)
```

**Que se passe-t-il ?**
- La socket passe en mode **passif** (prête à accepter des connexions)
- Le noyau maintient une **file d'attente** de connexions entrantes
- Le paramètre `backlog` définit la taille maximale de cette file

**Comprendre le backlog** :

```
Connexions entrantes :  C1  C2  C3  C4  C5  C6  C7
                         |   |   |   |   |   |   |
File d'attente (5):     [C1][C2][C3][C4][C5] X   X
                                              |   |
                                     Rejetées (SYN flood protection)
```

**Cas d'usage réel : dimensionnement du backlog**

```python
import socket

# Serveur web haute charge
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(('0.0.0.0', 80))

# Backlog élevé pour gérer les pics de trafic
# Nginx utilise typiquement 511, Apache 128-256
server.listen(511)
```

**Backlog trop petit** :
```
# Si backlog = 5 et 10 clients se connectent simultanément
# → 5 connexions acceptées immédiatement
# → 5 connexions reçoivent un RST (connection refused)
```

**Backlog trop grand** :
```
# Consommation mémoire inutile si peu de connexions
# Sur Linux, limité par /proc/sys/net/core/somaxconn (défaut: 128)
```

#### Étape 5 : accept() - Acceptation des connexions

```python
# Bloque jusqu'à ce qu'un client se connecte
client_socket, client_address = server_socket.accept()

print(f"Client connecté depuis {client_address[0]}:{client_address[1]}")
```

**Valeurs retournées** :

1. `client_socket` : **Nouvelle socket** pour communiquer avec CE client
2. `client_address` : Tuple `(IP, port)` du client

**Point crucial** : `accept()` **crée une nouvelle socket**

```python
# server_socket : socket d'écoute (1 seule)
#    ├── client_socket_1 : communication avec client 1
#    ├── client_socket_2 : communication avec client 2
#    └── client_socket_3 : communication avec client 3
```

**Exemple complet côté serveur** :

```python
import socket

def start_server(host='0.0.0.0', port=8080):
    # 1. Création
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # 2. Configuration
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    # 3. Liaison
    server.bind((host, port))
    print(f"Serveur lié à {host}:{port}")

    # 4. Écoute
    server.listen(5)
    print("Serveur en écoute...")

    try:
        while True:
            # 5. Acceptation (boucle infinie)
            client_sock, client_addr = server.accept()
            print(f"Connexion acceptée depuis {client_addr}")

            # Traitement du client (section suivante)
            handle_client(client_sock, client_addr)

    except KeyboardInterrupt:
        print("\nArrêt du serveur")
    finally:
        server.close()

def handle_client(sock, addr):
    # À implémenter
    pass

if __name__ == "__main__":
    start_server()
```

### Côté client : initiation de connexion

#### connect() - Connexion au serveur

```python
import socket

# 1. Création de la socket
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. Connexion au serveur
client_socket.connect(('example.com', 80))

print("Connecté au serveur!")
```

**Que se passe-t-il sous le capot ?**

```
1. Résolution DNS : example.com → 93.184.216.34
2. 3-way handshake TCP :
   Client → SYN → Serveur
   Client ← SYN-ACK ← Serveur
   Client → ACK → Serveur
3. La connexion est établie
```

**Durée typique** :
- Résolution DNS : 10-100 ms
- 3-way handshake : 1 RTT (Round-Trip Time)
- **Total** : 20-200 ms selon la distance

**Cas d'usage : mesure de latence**

```python
import socket
import time

def measure_connection_time(host, port):
    start = time.time()

    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((host, port))

    elapsed = (time.time() - start) * 1000  # en ms
    sock.close()

    print(f"Temps de connexion à {host}:{port} : {elapsed:.2f} ms")
    return elapsed

# Test
measure_connection_time('google.com', 443)
measure_connection_time('example.com', 80)
```

**Gestion des timeouts** :

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Timeout de 5 secondes pour la connexion
client.settimeout(5.0)

try:
    client.connect(('192.168.1.100', 9999))
except socket.timeout:
    print("Timeout : serveur injoignable")
except ConnectionRefusedError:
    print("Connexion refusée : aucun serveur n'écoute")
except OSError as e:
    print(f"Erreur réseau : {e}")
finally:
    client.close()
```

**Cas d'usage réel : connection pooling**

Les applications performantes réutilisent les connexions plutôt que d'en créer de nouvelles :

```python
import socket
from queue import Queue

class ConnectionPool:
    def __init__(self, host, port, pool_size=10):
        self.host = host
        self.port = port
        self.pool = Queue(maxsize=pool_size)

        # Pré-créer les connexions
        for _ in range(pool_size):
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.connect((host, port))
            self.pool.put(sock)

    def get_connection(self):
        """Emprunte une connexion du pool"""
        return self.pool.get()

    def return_connection(self, sock):
        """Remet une connexion dans le pool"""
        self.pool.put(sock)

    def close_all(self):
        """Ferme toutes les connexions"""
        while not self.pool.empty():
            sock = self.pool.get()
            sock.close()

# Utilisation
pool = ConnectionPool('api.example.com', 443, pool_size=20)

# Emprunter une connexion
conn = pool.get_connection()
# ... utiliser la connexion ...
pool.return_connection(conn)  # Réutilisable par un autre thread

pool.close_all()
```

**Exemple : bibliothèque `requests` en Python**

```python
import requests

# requests utilise un pool de connexions par défaut
session = requests.Session()

# Ces requêtes réutilisent la même connexion TCP
for i in range(10):
    response = session.get('https://api.example.com/data')
    print(response.status_code)

# Une seule connexion TCP établie pour les 10 requêtes
# Gain : 9 × (temps de handshake) économisé
```

### Le 3-way handshake en détail

Comprenons ce qui se passe lors de `connect()` :

```
CLIENT (192.168.1.5)              SERVEUR (93.184.216.34:80)
     |                                       |
     |                                       |
     | État: CLOSED                          | État: LISTEN
     |                                       |
[connect()]                                  |
     |                                       |
     | ─────── SYN (seq=1000) ─────>         |
     | État: SYN_SENT                        |
     |                                       | [accept() débloque]
     |                              État: SYN_RECEIVED
     |                                       |
     | <─── SYN-ACK (seq=5000, ack=1001) ──  |
     |                                       |
     | État: ESTABLISHED                     |
     | ─────── ACK (ack=5001) ─────>         |
     |                                       |
     |                              État: ESTABLISHED
     |                                       |
[connect() retourne]                [accept() retourne]
     |                                       |
```

**Numéros de séquence** :
- Client choisit un numéro initial aléatoire (ex: 1000)
- Serveur choisit le sien (ex: 5000)
- Chaque octet envoyé incrémente le numéro de séquence

**Cas d'usage : détection de SYN flood**

Une attaque DDoS courante consiste à envoyer des millions de SYN sans jamais répondre au SYN-ACK :

```python
# Côté serveur, protection via SYN cookies
# (géré par le noyau, pas l'application)

# Sur Linux :
# echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# Surveillance de l'attaque
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.bind(('0.0.0.0', 80))
sock.listen(1024)  # Backlog élevé

while True:
    try:
        client, addr = sock.accept()
        # Si beaucoup de SYN sans ACK final,
        # accept() bloque et la file se remplit
    except Exception as e:
        print(f"Possible SYN flood : {e}")
```

## Phase 2 : Échange de données

Une fois la connexion établie, les deux parties peuvent échanger des données de manière bidirectionnelle.

### send() - Envoi de données

```python
# Envoi de données (bytes)
message = "Hello, Server!"
bytes_sent = client_socket.send(message.encode('utf-8'))

print(f"Envoyé {bytes_sent} octets")
```

**Point crucial** : `send()` ne garantit PAS l'envoi de toutes les données !

```python
message = b"x" * 10000  # 10 000 octets

bytes_sent = sock.send(message)
print(bytes_sent)  # Pourrait afficher 8192, pas 10000 !
```

**Pourquoi ?**
- Le **buffer d'envoi** du socket est plein
- Le récepteur est lent (fenêtre TCP petite)
- Limitation réseau

**Solution : sendall()**

```python
# sendall() garantit l'envoi de toutes les données
# (ou lève une exception en cas d'erreur)
client_socket.sendall(message.encode('utf-8'))
```

**Implémentation manuelle de sendall()** :

```python
def send_all(sock, data):
    """Envoie toutes les données, même si plusieurs appels nécessaires"""
    total_sent = 0
    data_length = len(data)

    while total_sent < data_length:
        sent = sock.send(data[total_sent:])

        if sent == 0:
            raise RuntimeError("Socket connexion rompue")

        total_sent += sent

    return total_sent

# Utilisation
bytes_sent = send_all(client_socket, b"Long message...")
```

**Cas d'usage réel : envoi d'un fichier**

```python
import socket
import os

def send_file(sock, file_path):
    """Envoie un fichier via une socket TCP"""
    file_size = os.path.getsize(file_path)

    # 1. Envoyer la taille du fichier (8 octets, big-endian)
    sock.sendall(file_size.to_bytes(8, byteorder='big'))

    # 2. Envoyer le contenu du fichier
    with open(file_path, 'rb') as f:
        sent = 0
        while sent < file_size:
            chunk = f.read(4096)  # Lire par blocs de 4 KB
            if not chunk:
                break
            sock.sendall(chunk)
            sent += len(chunk)

            # Affichage de progression
            progress = (sent / file_size) * 100
            print(f"Envoi : {progress:.1f}%", end='\r')

    print(f"\nFichier envoyé : {sent} octets")

# Utilisation
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('192.168.1.100', 9999))
send_file(client, 'large_file.zip')
client.close()
```

### recv() - Réception de données

```python
# Recevoir jusqu'à 4096 octets
data = client_socket.recv(4096)

print(f"Reçu : {data.decode('utf-8')}")
```

**Points cruciaux sur recv()** :

1. **Bloquant par défaut** : attend qu'il y ait des données
2. **Retourne au maximum N octets**, mais peut en retourner moins
3. **Retourne b''** (bytes vide) si la connexion est fermée
4. **Ne respecte pas les frontières des messages**

**Problème : frontières de messages**

TCP est un **flux d'octets**, pas un flux de messages :

```python
# Émetteur
sock.send(b"Message1")
sock.send(b"Message2")
sock.send(b"Message3")

# Récepteur (plusieurs scénarios possibles)
data = sock.recv(1024)
# Peut recevoir : b"Message1"
# Peut recevoir : b"Message1Message2"
# Peut recevoir : b"Message1Message2Message3"
# Peut recevoir : b"Message1Mes"  (message tronqué !)
```

**Solution 1 : Protocole avec longueur**

```python
def send_message(sock, message):
    """Envoie un message préfixé par sa longueur"""
    msg_bytes = message.encode('utf-8')
    length = len(msg_bytes)

    # Envoyer la longueur (4 octets, big-endian) puis le message
    sock.sendall(length.to_bytes(4, byteorder='big'))
    sock.sendall(msg_bytes)

def recv_message(sock):
    """Reçoit un message préfixé par sa longueur"""
    # 1. Recevoir exactement 4 octets (longueur)
    length_bytes = recv_exact(sock, 4)
    length = int.from_bytes(length_bytes, byteorder='big')

    # 2. Recevoir exactement 'length' octets (message)
    msg_bytes = recv_exact(sock, length)
    return msg_bytes.decode('utf-8')

def recv_exact(sock, n):
    """Reçoit exactement n octets"""
    data = b''
    while len(data) < n:
        packet = sock.recv(n - len(data))
        if not packet:
            raise RuntimeError("Connexion fermée prématurément")
        data += packet
    return data

# Utilisation
send_message(client_socket, "Hello, this is a complete message!")
message = recv_message(client_socket)
print(message)
```

**Solution 2 : Délimiteur**

```python
def send_message_delim(sock, message):
    """Envoie un message terminé par '\n'"""
    sock.sendall(message.encode('utf-8') + b'\n')

def recv_message_delim(sock):
    """Reçoit un message jusqu'au délimiteur '\n'"""
    buffer = b''
    while True:
        chunk = sock.recv(1)
        if not chunk:
            raise RuntimeError("Connexion fermée")
        if chunk == b'\n':
            break
        buffer += chunk
    return buffer.decode('utf-8')

# Utilisation
send_message_delim(client_socket, "Message ligne 1")
send_message_delim(client_socket, "Message ligne 2")

msg1 = recv_message_delim(client_socket)
msg2 = recv_message_delim(client_socket)
```

**Cas d'usage réel : protocole HTTP**

HTTP utilise `\r\n` comme délimiteur et une en-tête `Content-Length` :

```python
def recv_http_response(sock):
    """Parse une réponse HTTP simple"""
    # Recevoir les headers (jusqu'à \r\n\r\n)
    headers = b''
    while b'\r\n\r\n' not in headers:
        chunk = sock.recv(1024)
        if not chunk:
            break
        headers += chunk

    # Extraire Content-Length
    headers_str = headers.decode('utf-8')
    content_length = 0
    for line in headers_str.split('\r\n'):
        if line.startswith('Content-Length:'):
            content_length = int(line.split(':')[1].strip())
            break

    # Recevoir le body
    body = recv_exact(sock, content_length)

    return headers_str, body.decode('utf-8')

# Exemple d'utilisation
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('example.com', 80))
client.sendall(b'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n')

headers, body = recv_http_response(client)
print("Headers:", headers)
print("Body:", body[:200])  # Premiers 200 caractères
client.close()
```

### Patterns de lecture avancés

#### Pattern 1 : Lecture par ligne (buffering)

```python
class SocketLineReader:
    """Lit une socket ligne par ligne avec buffering"""

    def __init__(self, sock, buffer_size=4096):
        self.sock = sock
        self.buffer = b''
        self.buffer_size = buffer_size

    def readline(self):
        """Lit une ligne complète (jusqu'à \n)"""
        while b'\n' not in self.buffer:
            chunk = self.sock.recv(self.buffer_size)
            if not chunk:
                # Connexion fermée
                if self.buffer:
                    line = self.buffer
                    self.buffer = b''
                    return line
                return None
            self.buffer += chunk

        # Extraire la ligne du buffer
        line, self.buffer = self.buffer.split(b'\n', 1)
        return line + b'\n'

# Utilisation
reader = SocketLineReader(client_socket)
while True:
    line = reader.readline()
    if line is None:
        break
    print(line.decode('utf-8').strip())
```

#### Pattern 2 : Lecture avec timeout

```python
import socket
import select

def recv_with_timeout(sock, size, timeout=5.0):
    """recv() avec timeout personnalisé"""
    # Utiliser select pour attendre des données
    ready = select.select([sock], [], [], timeout)

    if ready[0]:
        return sock.recv(size)
    else:
        raise socket.timeout("Timeout lors de la réception")

# Utilisation
try:
    data = recv_with_timeout(client_socket, 4096, timeout=3.0)
except socket.timeout:
    print("Aucune donnée reçue pendant 3 secondes")
```

#### Pattern 3 : Lecture complète jusqu'à fermeture

```python
def recv_all(sock, buffer_size=4096):
    """Reçoit toutes les données jusqu'à ce que le serveur ferme"""
    chunks = []
    while True:
        chunk = sock.recv(buffer_size)
        if not chunk:
            break
        chunks.append(chunk)
    return b''.join(chunks)

# Utilisation (cas d'un serveur qui envoie puis ferme)
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('example.com', 80))
client.sendall(b'GET / HTTP/1.0\r\n\r\n')  # HTTP/1.0 ferme après réponse

response = recv_all(client)
print(response.decode('utf-8'))
client.close()
```

### Communication bidirectionnelle simultanée

TCP est **full-duplex** : envoi et réception peuvent se faire simultanément.

```python
import threading
import socket

def receiver_thread(sock):
    """Thread dédié à la réception"""
    while True:
        try:
            data = sock.recv(1024)
            if not data:
                print("Serveur déconnecté")
                break
            print(f"Reçu : {data.decode('utf-8')}")
        except:
            break

def sender_thread(sock):
    """Thread dédié à l'envoi"""
    while True:
        message = input("Vous : ")
        if message.lower() == 'quit':
            break
        sock.sendall(message.encode('utf-8'))

# Client de chat simple
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('chat.example.com', 9999))

# Lancer les deux threads
recv_thread = threading.Thread(target=receiver_thread, args=(client,))
send_thread = threading.Thread(target=sender_thread, args=(client,))

recv_thread.start()
send_thread.start()

send_thread.join()
client.close()
```

**Cas d'usage réel : WebSocket après upgrade**

WebSocket utilise TCP en full-duplex pour la communication temps réel :

```javascript
// Côté navigateur (JavaScript)
const ws = new WebSocket('ws://example.com:8080');

// Réception (asynchrone)
ws.onmessage = (event) => {
    console.log('Reçu:', event.data);
};

// Envoi (peut se faire pendant réception)
ws.send('Hello server!');
ws.send('Another message');
```

## Phase 3 : Fermeture de connexion

### shutdown() vs close()

Il existe deux façons de fermer une socket :

#### shutdown() - Fermeture partielle

```python
import socket

# Fermeture de l'envoi uniquement (half-close)
sock.shutdown(socket.SHUT_WR)  # Plus d'envois, mais réception possible

# Fermeture de la réception uniquement
sock.shutdown(socket.SHUT_RD)  # Plus de réceptions, mais envoi possible

# Fermeture complète
sock.shutdown(socket.SHUT_RDWR)  # Équivalent à SHUT_RD + SHUT_WR
```

**Cas d'usage : signaler la fin d'envoi**

```python
def send_file_with_shutdown(sock, file_path):
    """Envoie un fichier puis signale la fin"""
    with open(file_path, 'rb') as f:
        while True:
            chunk = f.read(4096)
            if not chunk:
                break
            sock.sendall(chunk)

    # Signaler qu'on a fini d'envoyer (envoie FIN)
    sock.shutdown(socket.SHUT_WR)

    # On peut encore recevoir une réponse
    response = sock.recv(1024)
    print(f"Confirmation : {response}")

    # Fermeture finale
    sock.close()
```

#### close() - Fermeture complète

```python
# Ferme complètement la socket et libère les ressources
sock.close()
```

**Différence cruciale** :

```python
# shutdown() → Envoie FIN au pair, mais garde le descripteur
sock.shutdown(socket.SHUT_RDWR)
# sock existe encore, peut être utilisé pour setsockopt(), etc.

# close() → Libère le descripteur
sock.close()
# sock n'est plus valide, toute utilisation causera une erreur
```

### Le 4-way handshake de fermeture

```
CLIENT                           SERVEUR
  |                                 |
  | [close() ou shutdown(SHUT_WR)]  |
  |                                 |
  | ──────── FIN ──────>            |
  |                                 |
  | État: FIN_WAIT_1                |
  |                        État: CLOSE_WAIT
  | <──────── ACK ─────             |
  |                                 |
  | État: FIN_WAIT_2                |
  |                                 |
  |                   [close() ou shutdown()]
  |                                 |
  | <──────── FIN ─────             |
  |                                 |
  |                        État: LAST_ACK
  | ──────── ACK ──────>            |
  |                                 |
  | État: TIME_WAIT (2 × MSL)      État: CLOSED
  |                                 |
  | [après timeout]                 |
  |                                 |
  État: CLOSED                      |
```

**TIME_WAIT expliqué** :

Après avoir envoyé le dernier ACK, le client attend **2 × MSL** (Maximum Segment Lifetime, typiquement 60 secondes sur Linux) pour s'assurer que le serveur a bien reçu l'ACK.

**Conséquence** : Une socket en TIME_WAIT **occupe encore le port** !

```python
# Premier démarrage
server.bind(('0.0.0.0', 8080))  # OK

# Ctrl+C pour arrêter

# Redémarrage immédiat
server.bind(('0.0.0.0', 8080))  # Erreur : Address already in use

# Solution : SO_REUSEADDR
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

### Fermeture gracieuse vs brutale

#### Fermeture gracieuse

```python
import socket

def graceful_shutdown(sock, timeout=5.0):
    """Fermeture gracieuse avec timeout"""
    try:
        # 1. Arrêter l'envoi
        sock.shutdown(socket.SHUT_WR)

        # 2. Vider le buffer de réception (lire jusqu'au FIN du pair)
        sock.settimeout(timeout)
        while True:
            data = sock.recv(1024)
            if not data:
                break

    except socket.timeout:
        print("Timeout lors de la fermeture gracieuse")

    except Exception as e:
        print(f"Erreur lors de la fermeture : {e}")

    finally:
        # 3. Fermeture finale
        sock.close()

# Utilisation
graceful_shutdown(client_socket)
```

#### Fermeture brutale (RST)

```python
import socket
import struct

# Option SO_LINGER pour fermeture brutale
sock.setsockopt(socket.SOL_SOCKET, socket.SO_LINGER, struct.pack('ii', 1, 0))
# Paramètres : (activer, timeout)
# timeout = 0 → fermeture immédiate avec RST

sock.close()  # Envoie RST au lieu de FIN
```

**Cas d'usage : fermeture brutale**

- Client malveillant détecté → fermeture immédiate
- Conditions d'erreur critiques
- Shutdown d'urgence du serveur

**Conséquences du RST** :
- Données en transit peuvent être perdues
- Le pair reçoit une erreur "Connection reset by peer"
- Pas de TIME_WAIT (libération immédiate du port)

### Gestion des connexions fermées par le pair

```python
def handle_client(sock, addr):
    """Gestion robuste d'un client"""
    try:
        while True:
            data = sock.recv(1024)

            # Connexion fermée par le client
            if not data:
                print(f"Client {addr} s'est déconnecté proprement")
                break

            # Traiter les données
            response = process_data(data)
            sock.sendall(response)

    except ConnectionResetError:
        print(f"Client {addr} a fermé brutalement (RST)")

    except BrokenPipeError:
        print(f"Tentative d'écriture sur socket fermée par {addr}")

    except Exception as e:
        print(f"Erreur avec client {addr} : {e}")

    finally:
        sock.close()
        print(f"Connexion avec {addr} fermée")

def process_data(data):
    # Traitement métier
    return b"ACK: " + data
```

## États de connexion TCP

Une socket TCP traverse plusieurs états durant son cycle de vie :

```
CLOSED → LISTEN (serveur) ou SYN_SENT (client)
       ↓                              ↓
    ESTABLISHED ← ← ← ← ← ← ← ← ESTABLISHED
       ↓
    FIN_WAIT_1 / CLOSE_WAIT
       ↓
    FIN_WAIT_2 / LAST_ACK
       ↓
    TIME_WAIT
       ↓
    CLOSED
```

**Visualiser les états** (Linux/macOS) :

```bash
# Afficher toutes les connexions TCP
netstat -ant

# Ou avec ss (plus moderne)
ss -tan

# Exemple de sortie :
# State      Recv-Q Send-Q Local Address:Port  Peer Address:Port
# LISTEN     0      128    0.0.0.0:8080        0.0.0.0:*
# ESTABLISHED 0     0      192.168.1.5:54321   93.184.216.34:80
# TIME_WAIT  0      0      192.168.1.5:54322   93.184.216.34:80
```

**Cas d'usage : monitoring des connexions**

```python
import subprocess
import re

def count_tcp_states():
    """Compte les connexions TCP par état (Linux)"""
    output = subprocess.check_output(['ss', '-tan']).decode('utf-8')

    states = {}
    for line in output.split('\n'):
        match = re.match(r'^(\w+)\s+', line)
        if match:
            state = match.group(1)
            states[state] = states.get(state, 0) + 1

    return states

# Utilisation
states = count_tcp_states()
for state, count in states.items():
    print(f"{state}: {count}")

# Exemple de sortie :
# LISTEN: 15
# ESTABLISHED: 42
# TIME_WAIT: 8
# CLOSE_WAIT: 2
```

## Exemple complet : serveur echo multi-clients

```python
import socket
import threading

class EchoServer:
    def __init__(self, host='0.0.0.0', port=9999):
        self.host = host
        self.port = port
        self.server_socket = None
        self.clients = []

    def start(self):
        """Démarre le serveur"""
        # 1. Création et configuration
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        # 2. Liaison et écoute
        self.server_socket.bind((self.host, self.port))
        self.server_socket.listen(10)

        print(f"Serveur echo démarré sur {self.host}:{self.port}")

        try:
            while True:
                # 3. Acceptation des connexions
                client_socket, client_address = self.server_socket.accept()
                print(f"Nouvelle connexion : {client_address}")

                # 4. Thread dédié pour chaque client
                client_thread = threading.Thread(
                    target=self.handle_client,
                    args=(client_socket, client_address)
                )
                client_thread.daemon = True
                client_thread.start()

                self.clients.append(client_thread)

        except KeyboardInterrupt:
            print("\nArrêt du serveur...")
        finally:
            self.server_socket.close()

    def handle_client(self, sock, addr):
        """Gère un client individuel"""
        try:
            # Envoyer un message de bienvenue
            welcome = f"Bienvenue sur le serveur echo! Votre IP: {addr[0]}\n"
            sock.sendall(welcome.encode('utf-8'))

            while True:
                # Recevoir des données
                data = sock.recv(4096)

                if not data:
                    # Client déconnecté proprement
                    break

                # Echo : renvoyer les données reçues
                message = data.decode('utf-8').strip()
                print(f"[{addr[0]}:{addr[1]}] {message}")

                # Commande spéciale pour déconnexion
                if message.lower() == 'quit':
                    sock.sendall(b"Goodbye!\n")
                    break

                # Renvoyer le message
                sock.sendall(f"ECHO: {message}\n".encode('utf-8'))

        except ConnectionResetError:
            print(f"Client {addr} déconnecté brutalement")

        except Exception as e:
            print(f"Erreur avec {addr} : {e}")

        finally:
            print(f"Fermeture connexion {addr}")
            sock.close()

if __name__ == "__main__":
    server = EchoServer(port=9999)
    server.start()
```

**Client correspondant** :

```python
import socket
import threading
import sys

def receive_messages(sock):
    """Thread de réception"""
    while True:
        try:
            data = sock.recv(4096)
            if not data:
                print("\nServeur déconnecté")
                sys.exit(0)
            print(data.decode('utf-8'), end='')
        except:
            break

def main():
    # Connexion au serveur
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(('localhost', 9999))

    # Thread de réception
    recv_thread = threading.Thread(target=receive_messages, args=(client,))
    recv_thread.daemon = True
    recv_thread.start()

    # Boucle d'envoi
    try:
        while True:
            message = input()
            client.sendall((message + '\n').encode('utf-8'))

            if message.lower() == 'quit':
                break

    except KeyboardInterrupt:
        pass
    finally:
        client.close()

if __name__ == "__main__":
    main()
```

**Test** :

```bash
# Terminal 1 : serveur
python echo_server.py

# Terminal 2 : client 1
python echo_client.py
Hello
# ECHO: Hello

# Terminal 3 : client 2
python echo_client.py
World
# ECHO: World
```

## Pièges courants et bonnes pratiques

### ❌ Piège 1 : Ne pas vérifier la valeur de retour de send()

```python
# MAUVAIS
sock.send(large_data)  # Peut ne pas tout envoyer!

# BON
sock.sendall(large_data)  # Garantit l'envoi complet
```

### ❌ Piège 2 : Oublier que recv() peut retourner moins de données

```python
# MAUVAIS
data = sock.recv(1024)
# Suppose que toutes les données sont dans 'data'

# BON
data = recv_exact(sock, expected_length)  # Fonction vue plus haut
```

### ❌ Piège 3 : Ne pas gérer les exceptions réseau

```python
# MAUVAIS
data = sock.recv(1024)
# Crash si le réseau tombe

# BON
try:
    data = sock.recv(1024)
except (ConnectionError, socket.timeout) as e:
    handle_error(e)
```

### ❌ Piège 4 : Oublier close() (fuite de descripteurs)

```python
# MAUVAIS
sock = socket.socket(...)
sock.connect(...)
# Oubli de close() → fuite de ressources

# BON
with socket.socket(...) as sock:
    sock.connect(...)
    # close() automatique
```

### ❌ Piège 5 : Bloquer le thread principal avec accept()

```python
# MAUVAIS (serveur mono-thread)
while True:
    client, addr = server.accept()
    handle_client(client)  # Bloque pour les autres clients!

# BON (serveur multi-thread ou asynchrone)
while True:
    client, addr = server.accept()
    threading.Thread(target=handle_client, args=(client,)).start()
```

### ✅ Bonne pratique 1 : Toujours définir des timeouts

```python
# Timeout pour éviter les blocages infinis
sock.settimeout(30.0)  # 30 secondes
```

### ✅ Bonne pratique 2 : Utiliser SO_REUSEADDR sur les serveurs

```python
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

### ✅ Bonne pratique 3 : Protocole avec longueur ou délimiteur

```python
# Définir un protocole clair pour les frontières de messages
# Option 1 : Longueur fixe au début
# Option 2 : Délimiteur (\n, \0, etc.)
# Option 3 : Protocole existant (HTTP, WebSocket, etc.)
```

### ✅ Bonne pratique 4 : Logging et monitoring

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def handle_client(sock, addr):
    logger.info(f"Client connecté : {addr}")
    try:
        # ...
    except Exception as e:
        logger.error(f"Erreur avec {addr} : {e}")
    finally:
        logger.info(f"Client déconnecté : {addr}")
        sock.close()
```

## Cas d'usage réels d'entreprise

### 1. API Gateway (type Kong, Nginx)

```python
# Simplifié : proxy TCP qui forward les requêtes
class TCPProxy:
    def __init__(self, listen_port, backend_host, backend_port):
        self.listen_port = listen_port
        self.backend = (backend_host, backend_port)

    def start(self):
        server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server.bind(('0.0.0.0', self.listen_port))
        server.listen(100)

        while True:
            client, addr = server.accept()
            threading.Thread(target=self.proxy, args=(client,)).start()

    def proxy(self, client_sock):
        # Connexion au backend
        backend_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        backend_sock.connect(self.backend)

        # Bidirectional relay
        threading.Thread(target=self.forward, args=(client_sock, backend_sock)).start()
        threading.Thread(target=self.forward, args=(backend_sock, client_sock)).start()

    def forward(self, source, destination):
        try:
            while True:
                data = source.recv(4096)
                if not data:
                    break
                destination.sendall(data)
        finally:
            source.close()
            destination.close()

# Utilisation : proxy du port 8080 vers localhost:80
proxy = TCPProxy(8080, 'localhost', 80)
proxy.start()
```

### 2. Load Balancer simple (Round-Robin)

```python
class LoadBalancer:
    def __init__(self, listen_port, backends):
        self.listen_port = listen_port
        self.backends = backends
        self.current = 0

    def get_backend(self):
        """Round-robin selection"""
        backend = self.backends[self.current]
        self.current = (self.current + 1) % len(self.backends)
        return backend

    def start(self):
        server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server.bind(('0.0.0.0', self.listen_port))
        server.listen(100)

        while True:
            client, addr = server.accept()
            backend = self.get_backend()
            threading.Thread(target=self.handle, args=(client, backend)).start()

    def handle(self, client, backend):
        try:
            # Connexion au backend sélectionné
            backend_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            backend_sock.connect(backend)

            # Proxy bidirectionnel
            # ... (similaire à l'exemple précédent)
        except:
            client.close()

# Utilisation
lb = LoadBalancer(80, [
    ('backend1.local', 8080),
    ('backend2.local', 8080),
    ('backend3.local', 8080)
])
lb.start()
```

### 3. Health Check de serveurs

```python
def check_server_health(host, port, timeout=2.0):
    """Vérifie si un serveur répond"""
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        sock.connect((host, port))
        sock.close()
        return True
    except:
        return False

# Monitoring
backends = [
    ('web1.example.com', 80),
    ('web2.example.com', 80),
    ('web3.example.com', 80)
]

for host, port in backends:
    if check_server_health(host, port):
        print(f"✓ {host}:{port} - OK")
    else:
        print(f"✗ {host}:{port} - DOWN")
```

## Récapitulatif

| Phase | Opérations | Points clés |
|-------|-----------|-------------|
| **Création** | socket() | Choix AF_INET/IPv6, SOCK_STREAM |
| **Serveur : Écoute** | bind() → listen() → accept() | SO_REUSEADDR, backlog approprié |
| **Client : Connexion** | connect() | 3-way handshake, timeout |
| **Échange** | send()/recv() ou sendall() | Frontières messages, buffering |
| **Fermeture** | shutdown() → close() | Gracieuse vs brutale, TIME_WAIT |

## Points clés à retenir

✅ **TCP garantit fiabilité et ordre** mais avec un coût en latence

✅ **send() ne garantit pas l'envoi complet** → utiliser sendall()

✅ **recv() ne respecte pas les frontières** → protocole nécessaire

✅ **accept() crée une nouvelle socket** par client

✅ **Toujours gérer les exceptions** réseau (timeout, reset, etc.)

✅ **SO_REUSEADDR** évite "Address already in use"

✅ **Fermeture gracieuse** (shutdown + recv final) recommandée

✅ **TIME_WAIT** peut bloquer un port pendant 60s

## Prochaines étapes

Maintenant que vous maîtrisez les sockets TCP, nous allons voir :

- **Section 8.3** : Sockets UDP et leurs particularités (sans connexion)
- **Section 8.4** : Gestion robuste des erreurs réseau
- **Section 8.5** : I/O non-bloquant et modes asynchrones
- **Section 8.6** : Multiplexage (select, poll, epoll) pour gérer des milliers de connexions

---


⏭️ [Sockets UDP : envoi et réception de datagrammes](/08-programmation-reseau/03-sockets-udp.md)
