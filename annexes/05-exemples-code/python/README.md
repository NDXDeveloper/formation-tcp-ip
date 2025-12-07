🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Exemples Python

## Introduction

Python est un excellent langage pour apprendre la programmation réseau grâce à :
- Sa syntaxe claire et lisible
- Sa bibliothèque `socket` standard puissante
- Sa gestion simple des chaînes et bytes
- Ses options pour la programmation synchrone et asynchrone

**Versions Python :** Ces exemples sont compatibles Python 3.7+

**Prérequis :**
- Connaissances de base en Python
- Compréhension des concepts TCP/IP (ports, adresses IP, TCP vs UDP)
- Terminal pour exécuter les exemples

---

## Organisation des exemples

```
python/
├── README.md (ce fichier)
├── 01-tcp-simple/
│   ├── echo_server.py
│   ├── echo_client.py
│   └── README.md
├── 02-tcp-multi-clients/
│   ├── server_threading.py
│   ├── server_asyncio.py
│   ├── client.py
│   └── README.md
├── 03-udp/
│   ├── udp_server.py
│   ├── udp_client.py
│   └── README.md
├── 04-http-simple/
│   ├── http_server.py
│   ├── http_client.py
│   └── README.md
└── 05-chat/
    ├── chat_server.py
    ├── chat_client.py
    └── README.md
```

---

## 01. TCP Simple - Echo Server

### Concept

Un **echo server** renvoie exactement ce qu'il reçoit. C'est l'exemple "Hello World" de la programmation réseau.

**Architecture :**
```
Client                          Server
  |                                |
  | 1. Connexion TCP               |
  |------------------------------> |
  |                                |
  | 2. Envoi: "Hello"              |
  |------------------------------> |
  |                                |
  | 3. Réponse: "Hello"            |
  |<------------------------------ |
  |                                |
  | 4. Fermeture                   |
  |------------------------------> |
```

### echo_server.py - Version basique

```python
#!/usr/bin/env python3
"""
Echo Server TCP - Version simple
Renvoie tout ce qu'il reçoit au client
"""

import socket

# Configuration
HOST = '127.0.0.1'  # Localhost (interface loopback)
PORT = 8080         # Port d'écoute (> 1024 pour éviter root)

def main():
    # Créer une socket TCP/IPv4
    # AF_INET = IPv4, SOCK_STREAM = TCP
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # Option SO_REUSEADDR : permet de réutiliser le port immédiatement
    # Utile en développement pour éviter "Address already in use"
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    # Lier la socket à l'adresse et au port
    # bind() associe la socket à une interface réseau spécifique
    server_socket.bind((HOST, PORT))

    # Passer en mode écoute
    # listen(backlog) : backlog = nombre max de connexions en attente
    server_socket.listen(5)

    print(f"[SERVEUR] Démarré sur {HOST}:{PORT}")
    print(f"[SERVEUR] En attente de connexions...")

    try:
        while True:
            # Accepter une connexion entrante (BLOQUANT)
            # Retourne: (socket_client, adresse_client)
            client_socket, client_address = server_socket.accept()

            print(f"\n[CONNEXION] Nouveau client: {client_address}")

            try:
                # Recevoir les données du client
                # recv(buffer_size) : lit jusqu'à buffer_size octets
                # Retourne b'' si connexion fermée
                data = client_socket.recv(1024)

                if data:
                    # Décoder et afficher
                    message = data.decode('utf-8')
                    print(f"[REÇU] {len(data)} octets: {message}")

                    # Echo : renvoyer les données
                    # sendall() garantit l'envoi de toutes les données
                    client_socket.sendall(data)
                    print(f"[ENVOYÉ] Echo renvoyé au client")
                else:
                    print(f"[INFO] Client fermé la connexion")

            except Exception as e:
                print(f"[ERREUR] Lors du traitement: {e}")

            finally:
                # Toujours fermer la socket client
                client_socket.close()
                print(f"[DÉCONNEXION] Client {client_address} déconnecté")

    except KeyboardInterrupt:
        print("\n[SERVEUR] Arrêt demandé (Ctrl+C)")

    finally:
        # Fermer la socket serveur
        server_socket.close()
        print("[SERVEUR] Socket fermée")

if __name__ == '__main__':
    main()
```

### Explications détaillées

#### 1. Création de la socket

```python
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

**Paramètres :**
- `AF_INET` : Famille d'adresses IPv4
- `SOCK_STREAM` : Socket orientée connexion (TCP)

**Alternative IPv6 :**
```python
server_socket = socket.socket(socket.AF_INET6, socket.SOCK_STREAM)
```

#### 2. Option SO_REUSEADDR

```python
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

**Pourquoi ?**
- Après fermeture d'un serveur, TCP maintient le port en état `TIME_WAIT` (~2 minutes)
- Sans cette option, relancer le serveur immédiatement provoque "Address already in use"
- En développement, cette option est très utile

**Attention :** En production, évaluez les implications de sécurité.

#### 3. Bind - Liaison au port

```python
server_socket.bind((HOST, PORT))
```

**Significations de HOST :**
- `'127.0.0.1'` : Localhost uniquement (pas accessible depuis le réseau)
- `'0.0.0.0'` : Toutes les interfaces (accessible depuis le réseau)
- `'192.168.1.10'` : Interface spécifique

**Ports :**
- 0-1023 : Well-known, nécessitent root/admin
- 1024-49151 : Registered
- 49152-65535 : Dynamic/Private

#### 4. Listen - Mode écoute

```python
server_socket.listen(5)
```

**Paramètre backlog = 5 :**
- Maximum 5 connexions en attente dans la queue
- Si 6ème client se connecte pendant traitement, il attendra ou sera rejeté
- Taille typique : 5-128

#### 5. Accept - Accepter une connexion

```python
client_socket, client_address = server_socket.accept()
```

**Comportement :**
- **Bloquant** : attend qu'un client se connecte
- Retourne une **nouvelle socket** pour communiquer avec ce client
- `client_address` est un tuple : `('192.168.1.10', 54321)`

**Au niveau TCP :**
```
Client                Server
SYN      ---------->
         <----------  SYN-ACK
ACK      ---------->
                     accept() retourne
```

#### 6. Recv - Réception de données

```python
data = client_socket.recv(1024)
```

**Caractéristiques importantes :**
- Lit **jusqu'à** 1024 octets (peut en lire moins)
- Retourne `b''` (bytes vide) si connexion fermée proprement
- **Bloquant** : attend qu'il y ait des données
- TCP = flux, pas de garantie de recevoir tout en un coup

**Exemple :**
```python
# Client envoie 2000 octets
# Appel 1 : recv(1024) retourne 1024 octets
# Appel 2 : recv(1024) retourne 976 octets
```

#### 7. Send vs Sendall

```python
# send() : peut envoyer partiellement
bytes_sent = client_socket.send(data)

# sendall() : garantit l'envoi complet (recommandé)
client_socket.sendall(data)
```

**Différence :**
- `send()` retourne le nombre d'octets effectivement envoyés
- `sendall()` boucle jusqu'à tout envoyer ou erreur

---

### echo_client.py - Client simple

```python
#!/usr/bin/env python3
"""
Echo Client TCP - Version simple
Envoie un message au serveur et affiche la réponse
"""

import socket
import sys

# Configuration
SERVER_HOST = '127.0.0.1'
SERVER_PORT = 8080

def main():
    # Message à envoyer (depuis argument ou défaut)
    if len(sys.argv) > 1:
        message = ' '.join(sys.argv[1:])
    else:
        message = "Hello, Server!"

    # Créer une socket TCP/IPv4
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    try:
        # Se connecter au serveur
        print(f"[CLIENT] Connexion à {SERVER_HOST}:{SERVER_PORT}...")
        client_socket.connect((SERVER_HOST, SERVER_PORT))
        print(f"[CLIENT] Connecté!")

        # Envoyer le message
        data_to_send = message.encode('utf-8')
        print(f"[ENVOI] {len(data_to_send)} octets: {message}")
        client_socket.sendall(data_to_send)

        # Recevoir la réponse
        response = client_socket.recv(1024)
        response_text = response.decode('utf-8')
        print(f"[RÉPONSE] {len(response)} octets: {response_text}")

        # Vérifier que c'est bien un echo
        if response_text == message:
            print("[SUCCESS] Echo correct!")
        else:
            print("[WARNING] Echo incorrect!")

    except ConnectionRefusedError:
        print(f"[ERREUR] Connexion refusée. Le serveur est-il démarré?")
        sys.exit(1)
    except Exception as e:
        print(f"[ERREUR] {e}")
        sys.exit(1)
    finally:
        # Fermer la socket
        client_socket.close()
        print("[CLIENT] Socket fermée")

if __name__ == '__main__':
    main()
```

### Explications client

#### Connect - Connexion au serveur

```python
client_socket.connect((SERVER_HOST, SERVER_PORT))
```

**Comportement :**
- Initie le 3-way handshake TCP
- **Bloquant** : attend la connexion ou timeout (défaut ~75s)
- Lève `ConnectionRefusedError` si aucun serveur n'écoute

**Au niveau TCP :**
```
Client               Server
SYN      --------->  (LISTEN)
         <---------  SYN-ACK
ACK      --------->
connect() retourne
```

#### Encodage/Décodage UTF-8

```python
# Encoder string → bytes
data = message.encode('utf-8')

# Décoder bytes → string
message = data.decode('utf-8')
```

**Pourquoi ?**
- Les sockets travaillent avec des **bytes**, pas des strings
- UTF-8 est l'encodage standard pour le texte

### Exécution

**Terminal 1 - Serveur :**
```bash
$ python3 echo_server.py
[SERVEUR] Démarré sur 127.0.0.1:8080
[SERVEUR] En attente de connexions...
```

**Terminal 2 - Client :**
```bash
$ python3 echo_client.py "Bonjour le serveur!"
[CLIENT] Connexion à 127.0.0.1:8080...
[CLIENT] Connecté!
[ENVOI] 19 octets: Bonjour le serveur!
[RÉPONSE] 19 octets: Bonjour le serveur!
[SUCCESS] Echo correct!
[CLIENT] Socket fermée
```

**Terminal 1 - Serveur affiche :**
```
[CONNEXION] Nouveau client: ('127.0.0.1', 54321)
[REÇU] 19 octets: Bonjour le serveur!
[ENVOYÉ] Echo renvoyé au client
[DÉCONNEXION] Client ('127.0.0.1', 54321) déconnecté
```

---

## 02. TCP Multi-clients - Gestion concurrente

### Problème

Le serveur précédent ne gère **qu'un client à la fois** :
- Pendant le traitement d'un client, les autres attendent
- Si un client est lent, tous les autres sont bloqués

### Solution 1 : Threading

Créer un **thread par client** pour traiter plusieurs connexions simultanément.

### server_threading.py

```python
#!/usr/bin/env python3
"""
Echo Server TCP - Multi-clients avec Threading
Gère plusieurs clients simultanément
"""

import socket
import threading

HOST = '127.0.0.1'
PORT = 8080

# Compteur de clients (thread-safe avec un lock)
client_counter = 0
counter_lock = threading.Lock()

def handle_client(client_socket, client_address, client_id):
    """
    Gère un client dans un thread séparé

    Args:
        client_socket: Socket du client
        client_address: Adresse (IP, port) du client
        client_id: Identifiant unique du client
    """
    print(f"[THREAD {client_id}] Démarré pour {client_address}")

    try:
        while True:
            # Recevoir des données
            data = client_socket.recv(1024)

            if not data:
                # Connexion fermée par le client
                print(f"[THREAD {client_id}] Client a fermé la connexion")
                break

            # Afficher le message reçu
            message = data.decode('utf-8', errors='replace')
            print(f"[THREAD {client_id}] Reçu: {message}")

            # Echo : renvoyer les données
            client_socket.sendall(data)
            print(f"[THREAD {client_id}] Echo envoyé")

    except Exception as e:
        print(f"[THREAD {client_id}] Erreur: {e}")

    finally:
        # Fermer la socket client
        client_socket.close()
        print(f"[THREAD {client_id}] Socket fermée, thread terminé")

def main():
    global client_counter

    # Créer la socket serveur
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(5)

    print(f"[SERVEUR] Démarré sur {HOST}:{PORT}")
    print(f"[SERVEUR] En attente de connexions...")
    print(f"[SERVEUR] Mode: Multi-threading")

    try:
        while True:
            # Accepter une connexion
            client_socket, client_address = server_socket.accept()

            # Incrémenter le compteur (thread-safe)
            with counter_lock:
                client_counter += 1
                current_id = client_counter

            print(f"\n[SERVEUR] Nouvelle connexion #{current_id}: {client_address}")

            # Créer un thread pour gérer ce client
            client_thread = threading.Thread(
                target=handle_client,
                args=(client_socket, client_address, current_id),
                daemon=True  # Thread daemon: se termine avec le programme principal
            )

            # Démarrer le thread
            client_thread.start()

            # Afficher le nombre de threads actifs
            active_threads = threading.active_count() - 1  # -1 pour le thread principal
            print(f"[SERVEUR] Threads clients actifs: {active_threads}")

    except KeyboardInterrupt:
        print("\n[SERVEUR] Arrêt demandé (Ctrl+C)")

    finally:
        server_socket.close()
        print("[SERVEUR] Socket fermée")

if __name__ == '__main__':
    main()
```

### Explications Threading

#### 1. Fonction handle_client

```python
def handle_client(client_socket, client_address, client_id):
    """Traite un client dans son propre thread"""
```

**Caractéristiques :**
- Exécutée dans un thread séparé
- Peut bloquer sur `recv()` sans impacter les autres clients
- Continue jusqu'à fermeture de connexion ou erreur

#### 2. Création de thread

```python
client_thread = threading.Thread(
    target=handle_client,                        # Fonction à exécuter
    args=(client_socket, client_address, current_id),  # Arguments
    daemon=True                                  # Thread daemon
)
client_thread.start()
```

**Thread daemon :**
- Se termine automatiquement quand le programme principal se termine
- Utile pour éviter que le programme ne reste bloqué

#### 3. Thread Safety

```python
counter_lock = threading.Lock()

with counter_lock:
    client_counter += 1
    current_id = client_counter
```

**Pourquoi ?**
- Plusieurs threads accèdent à `client_counter`
- Sans lock, race condition possible
- `with counter_lock:` acquiert et libère automatiquement le lock

#### 4. Boucle de réception

```python
while True:
    data = client_socket.recv(1024)
    if not data:
        break
    # Traiter data
```

**Permet :**
- Connexions persistantes
- Client peut envoyer plusieurs messages
- Détection de fermeture (data vide)

### Client interactif

```python
#!/usr/bin/env python3
"""
Client interactif pour tester le serveur multi-clients
"""

import socket
import sys

SERVER_HOST = '127.0.0.1'
SERVER_PORT = 8080

def main():
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    try:
        # Connexion
        client_socket.connect((SERVER_HOST, SERVER_PORT))
        print(f"Connecté à {SERVER_HOST}:{SERVER_PORT}")
        print("Tapez vos messages (Ctrl+C pour quitter):")

        while True:
            # Lire input utilisateur
            message = input("> ")

            if not message:
                continue

            # Envoyer
            client_socket.sendall(message.encode('utf-8'))

            # Recevoir la réponse
            response = client_socket.recv(1024)
            print(f"Echo: {response.decode('utf-8')}")

    except KeyboardInterrupt:
        print("\nDéconnexion...")
    except Exception as e:
        print(f"Erreur: {e}")
    finally:
        client_socket.close()

if __name__ == '__main__':
    main()
```

### Test multi-clients

**Terminal 1 - Serveur :**
```bash
$ python3 server_threading.py
[SERVEUR] Démarré sur 127.0.0.1:8080
[SERVEUR] En attente de connexions...
```

**Terminal 2 - Client 1 :**
```bash
$ python3 client_interactive.py
Connecté à 127.0.0.1:8080
Tapez vos messages (Ctrl+C pour quitter):
> Hello from client 1
Echo: Hello from client 1
```

**Terminal 3 - Client 2 :**
```bash
$ python3 client_interactive.py
Connecté à 127.0.0.1:8080
Tapez vos messages (Ctrl+C pour quitter):
> Hello from client 2
Echo: Hello from client 2
```

**Terminal 1 affiche :**
```
[SERVEUR] Nouvelle connexion #1: ('127.0.0.1', 54322)
[THREAD 1] Démarré pour ('127.0.0.1', 54322)
[SERVEUR] Threads clients actifs: 1

[SERVEUR] Nouvelle connexion #2: ('127.0.0.1', 54323)
[THREAD 2] Démarré pour ('127.0.0.1', 54323)
[SERVEUR] Threads clients actifs: 2

[THREAD 1] Reçu: Hello from client 1
[THREAD 1] Echo envoyé
[THREAD 2] Reçu: Hello from client 2
[THREAD 2] Echo envoyé
```

---

### Solution 2 : Asyncio (Programmation asynchrone)

Alternative moderne au threading : **programmation asynchrone** avec `asyncio`.

### server_asyncio.py

```python
#!/usr/bin/env python3
"""
Echo Server TCP - Multi-clients avec asyncio
Gestion asynchrone sans threads
"""

import asyncio

HOST = '127.0.0.1'
PORT = 8080

client_counter = 0

async def handle_client(reader, writer):
    """
    Coroutine pour gérer un client

    Args:
        reader: StreamReader pour lire les données
        writer: StreamWriter pour écrire les données
    """
    global client_counter
    client_counter += 1
    client_id = client_counter

    # Obtenir l'adresse du client
    addr = writer.get_extra_info('peername')
    print(f"[CLIENT {client_id}] Connexion de {addr}")

    try:
        while True:
            # Lire les données (asynchrone, non-bloquant)
            data = await reader.read(1024)

            if not data:
                print(f"[CLIENT {client_id}] Connexion fermée")
                break

            # Afficher
            message = data.decode('utf-8', errors='replace')
            print(f"[CLIENT {client_id}] Reçu: {message}")

            # Echo : renvoyer
            writer.write(data)
            await writer.drain()  # Attendre que l'envoi soit terminé

            print(f"[CLIENT {client_id}] Echo envoyé")

    except Exception as e:
        print(f"[CLIENT {client_id}] Erreur: {e}")

    finally:
        # Fermer la connexion
        writer.close()
        await writer.wait_closed()
        print(f"[CLIENT {client_id}] Déconnecté")

async def main():
    """Coroutine principale"""

    # Créer le serveur
    server = await asyncio.start_server(
        handle_client,  # Coroutine appelée pour chaque client
        HOST,
        PORT
    )

    addr = server.sockets[0].getsockname()
    print(f"[SERVEUR] Démarré sur {addr}")
    print(f"[SERVEUR] Mode: Asyncio")

    # Servir indéfiniment
    async with server:
        await server.serve_forever()

if __name__ == '__main__':
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\n[SERVEUR] Arrêt demandé")
```

### Explications Asyncio

#### 1. Coroutines vs Threads

| Threading | Asyncio |
|-----------|---------|
| Threads système | Event loop unique |
| Préemptif | Coopératif |
| Overhead mémoire (~8MB/thread) | Léger (~KB/coroutine) |
| Bon pour I/O et CPU | Excellent pour I/O |
| GIL Python limite CPU | Pas de GIL pour I/O |

#### 2. Mots-clés async/await

```python
async def handle_client(reader, writer):
    data = await reader.read(1024)  # Point de suspension
```

**`async def` :** Définit une coroutine
**`await` :** Suspend la coroutine, redonne contrôle à l'event loop

**Flux d'exécution :**
```
1. Client A: await reader.read() → SUSPENDU
2. Event loop passe à Client B
3. Client B: await reader.read() → SUSPENDU
4. Event loop passe à Client C
5. Data arrive pour Client A → REPREND
6. ...
```

#### 3. StreamReader et StreamWriter

```python
async def handle_client(reader, writer):
    # Lire
    data = await reader.read(1024)

    # Écrire
    writer.write(data)
    await writer.drain()  # Vider le buffer
```

**reader :**
- `.read(n)` : lit jusqu'à n octets
- `.readline()` : lit jusqu'à '\n'
- `.readexactly(n)` : lit exactement n octets

**writer :**
- `.write(data)` : écrit dans le buffer
- `.drain()` : attend que le buffer soit vidé
- `.close()` : ferme la connexion

#### 4. Start server

```python
server = await asyncio.start_server(
    handle_client,  # Callback
    HOST,
    PORT
)
```

**Équivalent à :**
```python
socket()
bind()
listen()
# Pour chaque accept():
#     asyncio.create_task(handle_client(...))
```

### Comparaison Threading vs Asyncio

**Threading - Avantages :**
- ✅ Modèle mental simple
- ✅ Bon pour CPU-bound (avec multiprocessing)
- ✅ Bibliothèques compatibles

**Threading - Inconvénients :**
- ❌ Overhead mémoire (limite ~1000 threads)
- ❌ Race conditions possibles
- ❌ GIL limite parallélisme CPU

**Asyncio - Avantages :**
- ✅ Très léger (10k+ clients possibles)
- ✅ Pas de race conditions
- ✅ Excellent pour I/O-bound
- ✅ Performance supérieure

**Asyncio - Inconvénients :**
- ❌ Courbe d'apprentissage
- ❌ Toute la stack doit être async
- ❌ Mauvais pour CPU-bound

---

## 03. UDP - Sans connexion

UDP est sans connexion, non fiable, mais plus rapide que TCP.

### udp_server.py

```python
#!/usr/bin/env python3
"""
Echo Server UDP
Renvoie les datagrammes reçus
"""

import socket

HOST = '127.0.0.1'
PORT = 8080

def main():
    # Créer une socket UDP
    # SOCK_DGRAM = UDP
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # Lier au port
    server_socket.bind((HOST, PORT))

    print(f"[SERVEUR UDP] Démarré sur {HOST}:{PORT}")
    print(f"[SERVEUR UDP] En attente de datagrammes...")

    try:
        while True:
            # Recevoir un datagramme
            # recvfrom() retourne (data, address)
            # Pas besoin d'accept() car sans connexion
            data, client_address = server_socket.recvfrom(1024)

            # Afficher
            message = data.decode('utf-8', errors='replace')
            print(f"\n[REÇU DE {client_address}] {message}")
            print(f"[REÇU] {len(data)} octets")

            # Echo : renvoyer au même client
            # sendto() envoie à une adresse spécifique
            server_socket.sendto(data, client_address)
            print(f"[ENVOYÉ À {client_address}] Echo")

    except KeyboardInterrupt:
        print("\n[SERVEUR UDP] Arrêt demandé")

    finally:
        server_socket.close()
        print("[SERVEUR UDP] Socket fermée")

if __name__ == '__main__':
    main()
```

### udp_client.py

```python
#!/usr/bin/env python3
"""
Client UDP simple
"""

import socket
import sys

SERVER_HOST = '127.0.0.1'
SERVER_PORT = 8080

def main():
    # Message
    if len(sys.argv) > 1:
        message = ' '.join(sys.argv[1:])
    else:
        message = "Hello UDP!"

    # Créer une socket UDP
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # Définir un timeout pour recvfrom
    client_socket.settimeout(5.0)

    try:
        # Envoyer le datagramme (pas de connexion)
        server_address = (SERVER_HOST, SERVER_PORT)
        data = message.encode('utf-8')

        print(f"[CLIENT UDP] Envoi à {server_address}")
        print(f"[CLIENT UDP] Message: {message}")

        client_socket.sendto(data, server_address)

        # Recevoir la réponse
        print(f"[CLIENT UDP] En attente de réponse (timeout 5s)...")
        response, server = client_socket.recvfrom(1024)

        response_text = response.decode('utf-8')
        print(f"[CLIENT UDP] Réponse de {server}: {response_text}")

        # Vérifier l'echo
        if response_text == message:
            print("[SUCCESS] Echo correct!")

    except socket.timeout:
        print("[ERREUR] Timeout - Aucune réponse du serveur")
        sys.exit(1)
    except Exception as e:
        print(f"[ERREUR] {e}")
        sys.exit(1)
    finally:
        client_socket.close()
        print("[CLIENT UDP] Socket fermée")

if __name__ == '__main__':
    main()
```

### Différences clés UDP vs TCP

| Aspect | TCP | UDP |
|--------|-----|-----|
| **Type socket** | `SOCK_STREAM` | `SOCK_DGRAM` |
| **Connexion** | `connect()`, `accept()` | Aucune |
| **Envoi** | `send()`, `sendall()` | `sendto(data, addr)` |
| **Réception** | `recv()` → data | `recvfrom()` → (data, addr) |
| **État** | Connexion maintenue | Sans état |
| **Fiabilité** | Garanti | Non garanti |
| **Ordre** | Préservé | Non garanti |
| **Overhead** | Plus élevé (en-tête 20+ bytes) | Minimal (en-tête 8 bytes) |
| **Cas d'usage** | HTTP, SSH, FTP | DNS, streaming, jeux |

### Test UDP

**Terminal 1 :**
```bash
$ python3 udp_server.py
[SERVEUR UDP] Démarré sur 127.0.0.1:8080
[SERVEUR UDP] En attente de datagrammes...
```

**Terminal 2 :**
```bash
$ python3 udp_client.py "Test UDP"
[CLIENT UDP] Envoi à ('127.0.0.1', 8080)
[CLIENT UDP] Message: Test UDP
[CLIENT UDP] En attente de réponse (timeout 5s)...
[CLIENT UDP] Réponse de ('127.0.0.1', 8080): Test UDP
[SUCCESS] Echo correct!
[CLIENT UDP] Socket fermée
```

---

## 04. HTTP Simple - Protocole applicatif

Implémentation minimaliste d'un serveur HTTP pour comprendre les protocoles applicatifs.

### http_server.py

```python
#!/usr/bin/env python3
"""
Serveur HTTP minimal
Implémente un sous-ensemble de HTTP/1.1
"""

import socket
import datetime

HOST = '127.0.0.1'
PORT = 8000

# Page HTML simple
HTML_PAGE = """<!DOCTYPE html>
<html>
<head>
    <title>Serveur HTTP Python</title>
    <meta charset="utf-8">
</head>
<body>
    <h1>🐍 Serveur HTTP Python</h1>
    <p>Ce serveur HTTP minimal est écrit en Python.</p>
    <p>Heure du serveur: {time}</p>
    <p>Votre adresse: {client_addr}</p>
</body>
</html>
"""

def parse_http_request(request_data):
    """
    Parse une requête HTTP simple

    Retourne: (method, path, headers)
    """
    lines = request_data.decode('utf-8', errors='replace').split('\r\n')

    # Première ligne: "GET /path HTTP/1.1"
    request_line = lines[0]
    parts = request_line.split(' ')

    if len(parts) >= 3:
        method = parts[0]
        path = parts[1]
    else:
        method = "UNKNOWN"
        path = "/"

    # Headers
    headers = {}
    for line in lines[1:]:
        if ':' in line:
            key, value = line.split(':', 1)
            headers[key.strip()] = value.strip()

    return method, path, headers

def build_http_response(status_code, status_text, content, content_type='text/html'):
    """
    Construit une réponse HTTP

    Args:
        status_code: Code HTTP (200, 404, etc.)
        status_text: Texte du statut ("OK", "Not Found", etc.)
        content: Contenu de la réponse (bytes ou string)
        content_type: Type MIME du contenu

    Returns:
        bytes: Réponse HTTP complète
    """
    # Convertir content en bytes si nécessaire
    if isinstance(content, str):
        content = content.encode('utf-8')

    # Construire la réponse
    response = f"HTTP/1.1 {status_code} {status_text}\r\n"
    response += f"Content-Type: {content_type}; charset=utf-8\r\n"
    response += f"Content-Length: {len(content)}\r\n"
    response += f"Server: PythonHTTP/1.0\r\n"
    response += f"Date: {datetime.datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')}\r\n"
    response += "Connection: close\r\n"
    response += "\r\n"

    # Headers en bytes + content
    return response.encode('utf-8') + content

def handle_request(client_socket, client_address):
    """Traite une requête HTTP"""

    try:
        # Recevoir la requête
        request_data = client_socket.recv(4096)

        if not request_data:
            return

        # Parser la requête
        method, path, headers = parse_http_request(request_data)

        print(f"\n[{client_address[0]}] {method} {path}")

        # Router simple
        if path == '/':
            # Page d'accueil
            content = HTML_PAGE.format(
                time=datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
                client_addr=f"{client_address[0]}:{client_address[1]}"
            )
            response = build_http_response(200, "OK", content)

        elif path == '/api/time':
            # API JSON
            import json
            data = {
                "timestamp": datetime.datetime.now().isoformat(),
                "timezone": "Local"
            }
            content = json.dumps(data, indent=2)
            response = build_http_response(200, "OK", content, 'application/json')

        else:
            # 404 Not Found
            content = f"<h1>404 Not Found</h1><p>Le chemin '{path}' n'existe pas.</p>"
            response = build_http_response(404, "Not Found", content)

        # Envoyer la réponse
        client_socket.sendall(response)

    except Exception as e:
        print(f"[ERREUR] {e}")
        # Envoyer 500 Internal Server Error
        error_content = "<h1>500 Internal Server Error</h1>"
        error_response = build_http_response(500, "Internal Server Error", error_content)
        try:
            client_socket.sendall(error_response)
        except:
            pass

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(5)

    print(f"[SERVEUR HTTP] Démarré sur http://{HOST}:{PORT}")
    print(f"[SERVEUR HTTP] Appuyez sur Ctrl+C pour arrêter")

    try:
        while True:
            client_socket, client_address = server_socket.accept()
            handle_request(client_socket, client_address)
            client_socket.close()

    except KeyboardInterrupt:
        print("\n[SERVEUR HTTP] Arrêt demandé")

    finally:
        server_socket.close()
        print("[SERVEUR HTTP] Socket fermée")

if __name__ == '__main__':
    main()
```

### Structure d'une requête HTTP

```http
GET /index.html HTTP/1.1\r\n
Host: example.com\r\n
User-Agent: Mozilla/5.0\r\n
Accept: text/html\r\n
\r\n
```

**Composants :**
1. **Request line** : `METHOD PATH VERSION`
2. **Headers** : `Key: Value`
3. **Ligne vide** : `\r\n\r\n` sépare headers du body
4. **Body** : (optionnel, pour POST/PUT)

### Structure d'une réponse HTTP

```http
HTTP/1.1 200 OK\r\n
Content-Type: text/html\r\n
Content-Length: 123\r\n
\r\n
<html>...</html>
```

**Composants :**
1. **Status line** : `VERSION CODE TEXT`
2. **Headers**
3. **Ligne vide**
4. **Body** : Contenu (HTML, JSON, etc.)

### Test HTTP

**Démarrer le serveur :**
```bash
$ python3 http_server.py
[SERVEUR HTTP] Démarré sur http://127.0.0.1:8000
[SERVEUR HTTP] Appuyez sur Ctrl+C pour arrêter
```

**Tester avec un navigateur :**
- Ouvrir `http://127.0.0.1:8000/`
- Voir la page HTML

**Tester avec curl :**
```bash
$ curl http://127.0.0.1:8000/
<!DOCTYPE html>
<html>
...

$ curl http://127.0.0.1:8000/api/time
{
  "timestamp": "2024-12-07T10:30:00.123456",
  "timezone": "Local"
}
```

---

## 05. Application complète - Chat

Application chat en temps réel avec serveur et clients multiples.

### Architecture du chat

```
                    SERVEUR
                       |
        +--------------+--------------+
        |              |              |
    Client A       Client B       Client C

1. Client A envoie "Bonjour"
2. Serveur reçoit de A
3. Serveur broadcast à B et C
4. B et C affichent "A: Bonjour"
```

### chat_server.py

```python
#!/usr/bin/env python3
"""
Serveur de chat multi-clients
Architecture: un thread par client + broadcast
"""

import socket
import threading

HOST = '127.0.0.1'
PORT = 9000

# Liste des clients connectés (thread-safe)
clients = []
clients_lock = threading.Lock()

def broadcast(message, sender_socket=None):
    """
    Envoie un message à tous les clients sauf l'émetteur

    Args:
        message: Message à envoyer (bytes)
        sender_socket: Socket de l'émetteur (exclu du broadcast)
    """
    with clients_lock:
        for client in clients:
            # Ne pas renvoyer au sender
            if client != sender_socket:
                try:
                    client.sendall(message)
                except:
                    # Client déconnecté, sera nettoyé plus tard
                    pass

def handle_client(client_socket, client_address):
    """Gère un client dans un thread"""

    print(f"[SERVEUR] Nouvelle connexion: {client_address}")

    # Ajouter le client à la liste
    with clients_lock:
        clients.append(client_socket)

    # Demander un pseudo
    client_socket.sendall(b"Entrez votre pseudo: ")
    pseudo_data = client_socket.recv(1024)
    pseudo = pseudo_data.decode('utf-8').strip()

    if not pseudo:
        pseudo = f"Anonyme_{client_address[1]}"

    print(f"[SERVEUR] {client_address} est '{pseudo}'")

    # Annoncer l'arrivée
    welcome_msg = f"*** {pseudo} a rejoint le chat ***\n".encode('utf-8')
    broadcast(welcome_msg)

    # Instructions
    instructions = "Tapez vos messages. '/quit' pour quitter.\n".encode('utf-8')
    client_socket.sendall(instructions)

    try:
        while True:
            # Recevoir un message
            data = client_socket.recv(1024)

            if not data:
                break

            message = data.decode('utf-8').strip()

            # Commande quit
            if message == '/quit':
                print(f"[SERVEUR] {pseudo} se déconnecte")
                break

            # Broadcaster le message
            if message:
                print(f"[{pseudo}] {message}")
                formatted_msg = f"{pseudo}: {message}\n".encode('utf-8')
                broadcast(formatted_msg, sender_socket=client_socket)

    except Exception as e:
        print(f"[ERREUR] {pseudo}: {e}")

    finally:
        # Retirer le client de la liste
        with clients_lock:
            if client_socket in clients:
                clients.remove(client_socket)

        # Annoncer le départ
        leave_msg = f"*** {pseudo} a quitté le chat ***\n".encode('utf-8')
        broadcast(leave_msg)

        # Fermer la socket
        client_socket.close()
        print(f"[SERVEUR] {pseudo} déconnecté")

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(10)

    print(f"[SERVEUR CHAT] Démarré sur {HOST}:{PORT}")
    print(f"[SERVEUR CHAT] En attente de connexions...")

    try:
        while True:
            client_socket, client_address = server_socket.accept()

            # Thread pour ce client
            client_thread = threading.Thread(
                target=handle_client,
                args=(client_socket, client_address),
                daemon=True
            )
            client_thread.start()

    except KeyboardInterrupt:
        print("\n[SERVEUR CHAT] Arrêt demandé")

    finally:
        # Fermer tous les clients
        with clients_lock:
            for client in clients:
                client.close()

        server_socket.close()
        print("[SERVEUR CHAT] Socket fermée")

if __name__ == '__main__':
    main()
```

### chat_client.py

```python
#!/usr/bin/env python3
"""
Client de chat
Architecture: un thread pour recevoir, un pour envoyer
"""

import socket
import threading
import sys

SERVER_HOST = '127.0.0.1'
SERVER_PORT = 9000

def receive_messages(sock):
    """Thread pour recevoir les messages du serveur"""
    try:
        while True:
            data = sock.recv(1024)
            if not data:
                print("\n[INFO] Connexion fermée par le serveur")
                break

            message = data.decode('utf-8')
            print(message, end='')

    except Exception as e:
        print(f"\n[ERREUR] Réception: {e}")

def main():
    # Créer la socket
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    try:
        # Connexion
        print(f"Connexion à {SERVER_HOST}:{SERVER_PORT}...")
        client_socket.connect((SERVER_HOST, SERVER_PORT))
        print("Connecté!")

        # Thread pour recevoir
        recv_thread = threading.Thread(
            target=receive_messages,
            args=(client_socket,),
            daemon=True
        )
        recv_thread.start()

        # Boucle d'envoi (thread principal)
        while True:
            try:
                message = input()

                if message:
                    client_socket.sendall(message.encode('utf-8'))

                    if message == '/quit':
                        print("Déconnexion...")
                        break

            except EOFError:
                # Ctrl+D
                break

    except ConnectionRefusedError:
        print(f"[ERREUR] Impossible de se connecter au serveur")
        sys.exit(1)
    except Exception as e:
        print(f"[ERREUR] {e}")
    finally:
        client_socket.close()
        print("Déconnecté.")

if __name__ == '__main__':
    main()
```

### Test du chat

**Terminal 1 - Serveur :**
```bash
$ python3 chat_server.py
[SERVEUR CHAT] Démarré sur 127.0.0.1:9000
[SERVEUR CHAT] En attente de connexions...
```

**Terminal 2 - Client Alice :**
```bash
$ python3 chat_client.py
Connexion à 127.0.0.1:9000...
Connecté!
Entrez votre pseudo: Alice
Tapez vos messages. '/quit' pour quitter.
```

**Terminal 3 - Client Bob :**
```bash
$ python3 chat_client.py
Connexion à 127.0.0.1:9000...
Connecté!
Entrez votre pseudo: Bob
Tapez vos messages. '/quit' pour quitter.
*** Alice a rejoint le chat ***
```

**Alice tape :**
```
Bonjour Bob!
```

**Bob voit :**
```
Alice: Bonjour Bob!
```

**Bob tape :**
```
Salut Alice!
```

**Alice voit :**
```
Bob: Salut Alice!
```

---

## Bonnes pratiques Python

### 1. Gestion des erreurs

```python
def safe_recv(sock, size=1024):
    """Receive avec gestion d'erreur"""
    try:
        data = sock.recv(size)
        return data
    except socket.timeout:
        print("Timeout lors de la réception")
        return None
    except ConnectionResetError:
        print("Connexion réinitialisée par le pair")
        return None
    except Exception as e:
        print(f"Erreur inattendue: {e}")
        return None
```

### 2. Context managers

```python
# Fermeture automatique de socket
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.connect(('example.com', 80))
    sock.sendall(b"GET / HTTP/1.1\r\n\r\n")
    data = sock.recv(1024)
# sock.close() appelé automatiquement
```

### 3. Logging plutôt que print

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

logger.info("Serveur démarré")
logger.warning("Client timeout")
logger.error("Erreur de connexion")
```

### 4. Configuration

```python
# config.py
class Config:
    HOST = '0.0.0.0'
    PORT = 8080
    BUFFER_SIZE = 4096
    TIMEOUT = 30

# server.py
from config import Config

server_socket.bind((Config.HOST, Config.PORT))
```

---

## Conclusion

Vous avez maintenant une base solide de programmation réseau en Python :

- ✅ **Socket TCP** : Client/serveur simple
- ✅ **Multi-clients** : Threading et Asyncio
- ✅ **Socket UDP** : Communication sans connexion
- ✅ **Protocole HTTP** : Implémentation minimale
- ✅ **Application complète** : Chat en temps réel

**Prochaines étapes :**
- Expérimenter avec les exemples
- Capturer le trafic avec Wireshark
- Implémenter vos propres protocoles
- Explorer d'autres langages (Go, JavaScript)

**Ressources Python supplémentaires :**
- Documentation officielle : https://docs.python.org/3/library/socket.html
- Asyncio : https://docs.python.org/3/library/asyncio.html
- Real Python - Socket Programming : https://realpython.com/python-sockets/

---


⏭️ [E.2 Exemples Go](/annexes/05-exemples-code/go/README.md)
