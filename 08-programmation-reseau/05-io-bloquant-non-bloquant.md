🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.5 I/O bloquant vs non-bloquant

## Introduction

Le choix entre **I/O bloquant** et **I/O non-bloquant** est l'une des décisions architecturales les plus importantes en programmation réseau. Ce choix détermine :

- **Combien de connexions** votre application peut gérer
- **Comment** votre code est structuré
- **La complexité** de votre application
- **Les performances** sous charge

Comprendre ces modèles est essentiel pour construire des applications réseau scalables et performantes.

## Qu'est-ce que le blocage (Blocking) ?

### Définition

Une opération est **bloquante** quand elle **suspend l'exécution du thread** jusqu'à ce qu'elle soit complète ou qu'un timeout survienne.

```python
import socket

# Opération BLOQUANTE
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('example.com', 80))  # Thread bloqué ici

# Le code ci-dessous ne s'exécute PAS tant que connect() n'est pas terminé
print("Connecté!")  # Cette ligne attend
```

### Visualisation du blocage

```
Thread d'exécution :

t=0ms    | Code avant connect()
t=1ms    | sock.connect() commence
         |
         | ⏸️  THREAD BLOQUÉ
         | (attend handshake TCP)
         |
t=50ms   | Connexion établie, connect() retourne
t=51ms   | Code après connect()
```

**Pendant le blocage** :
- Le thread ne fait **rien** (consomme un slot de thread)
- Le CPU est **inutilisé** par ce thread
- Aucun autre code ne peut s'exécuter dans ce thread

## I/O Bloquant en détail

### Opérations bloquantes courantes

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 1. connect() - Bloque pendant le 3-way handshake
sock.connect(('example.com', 80))  # 10-100ms typique

# 2. send() - Peut bloquer si le buffer d'envoi est plein
sock.send(b"x" * 1000000)  # Bloque si le buffer est saturé

# 3. recv() - Bloque jusqu'à réception de données
data = sock.recv(4096)  # Bloque indéfiniment par défaut

# 4. accept() - Bloque jusqu'à connexion entrante
server.listen(5)
client, addr = server.accept()  # Bloque jusqu'à un client
```

### Exemple : Serveur bloquant simple

```python
import socket

def blocking_echo_server():
    """Serveur echo bloquant (mono-client)"""
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(('0.0.0.0', 8080))
    server.listen(5)

    print("Serveur en écoute sur :8080")

    while True:
        # 1. Bloque jusqu'à connexion entrante
        print("En attente d'un client...")
        client_sock, client_addr = server.accept()
        print(f"Client connecté: {client_addr}")

        # 2. Traite CE client uniquement
        while True:
            # 3. Bloque jusqu'à réception de données
            data = client_sock.recv(1024)

            if not data:
                break

            print(f"Reçu: {data}")

            # 4. Envoie la réponse (peut bloquer)
            client_sock.send(b"Echo: " + data)

        client_sock.close()
        print(f"Client {client_addr} déconnecté")

# PROBLÈME : Un seul client à la fois !
# Si Client A est connecté, Client B attend indéfiniment
```

**Test du problème** :

```bash
# Terminal 1 : Serveur
python blocking_server.py

# Terminal 2 : Client A se connecte
telnet localhost 8080
# Fonctionne

# Terminal 3 : Client B tente de se connecter
telnet localhost 8080
# BLOQUÉ ! Attend que Client A se déconnecte
```

### Solution 1 : Thread par connexion

```python
import socket
import threading

def blocking_echo_server_threaded():
    """Serveur echo avec un thread par client"""
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(('0.0.0.0', 8080))
    server.listen(5)

    print("Serveur multi-thread en écoute sur :8080")

    while True:
        client_sock, client_addr = server.accept()
        print(f"Client connecté: {client_addr}")

        # Créer un thread dédié pour ce client
        client_thread = threading.Thread(
            target=handle_client,
            args=(client_sock, client_addr)
        )
        client_thread.daemon = True
        client_thread.start()

def handle_client(sock, addr):
    """Gère un client dans son propre thread"""
    try:
        while True:
            data = sock.recv(1024)
            if not data:
                break

            print(f"[{addr}] Reçu: {data}")
            sock.send(b"Echo: " + data)

    finally:
        sock.close()
        print(f"Client {addr} déconnecté")

# AVANTAGE : Plusieurs clients simultanés
# PROBLÈME : Ne scale pas au-delà de ~1000 threads
```

**Pourquoi ça ne scale pas ?**

```
1 thread = ~8 MB de stack (Linux)
1000 threads = ~8 GB de RAM juste pour les stacks
10000 threads = ~80 GB de RAM !

+ Overhead de context switching
+ Contention sur les locks
→ Impossible de gérer 10000+ connexions
```

### Le problème C10K

**C10K** = Gérer **10 000 connexions concurrentes**

```python
# Approche thread-per-connection
10000 clients × 8 MB par thread = 80 GB de RAM
+ Context switching entre 10000 threads = CPU saturé

# Résultat : IMPOSSIBLE avec I/O bloquant + threads
```

Historiquement (années 1990-2000), les serveurs ne pouvaient pas gérer plus de quelques milliers de connexions simultanées.

**Cas d'usage réel** :
- **Chat en ligne** : 100 000 utilisateurs connectés
- **Gaming** : 50 000 joueurs sur un serveur
- **Streaming** : 1 million de viewers simultanés

→ I/O bloquant inadapté

## I/O Non-bloquant en détail

### Principe

Une opération **non-bloquante** retourne **immédiatement**, même si l'opération n'est pas complète.

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Passer en mode non-bloquant
sock.setblocking(False)

try:
    sock.connect(('example.com', 80))
except BlockingIOError:
    # Normal ! connect() retourne immédiatement
    # La connexion continue en arrière-plan
    print("Connexion en cours...")

# Le code continue IMMÉDIATEMENT
print("Cette ligne s'exécute tout de suite!")
```

### Visualisation du non-bloquant

```
Thread d'exécution :

t=0ms    | Code avant connect()
t=1ms    | sock.connect() lance la connexion
t=1ms    | BlockingIOError levée
t=1ms    | Code après connect()
         |
         | (Thread continue, connexion en arrière-plan)
         |
t=50ms   | Connexion réellement établie (on ne le sait pas encore)
```

### Opérations non-bloquantes

```python
import socket
import errno

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.setblocking(False)

# 1. connect() non-bloquant
try:
    sock.connect(('example.com', 80))
except BlockingIOError as e:
    # errno.EINPROGRESS = connexion en cours
    if e.errno != errno.EINPROGRESS:
        raise

# 2. recv() non-bloquant
try:
    data = sock.recv(1024)
    print(f"Reçu: {data}")
except BlockingIOError:
    # Pas de données disponibles pour le moment
    print("Pas de données (normal)")

# 3. send() non-bloquant
try:
    sent = sock.send(b"Hello")
    print(f"Envoyé {sent} octets")
except BlockingIOError:
    # Buffer d'envoi plein, réessayer plus tard
    print("Buffer plein (normal)")

# 4. accept() non-bloquant
try:
    client, addr = server.accept()
    print(f"Client: {addr}")
except BlockingIOError:
    # Pas de client en attente (normal)
    print("Pas de client (normal)")
```

### Problème : Polling naïf

```python
import socket
import time

def naive_nonblocking_client():
    """Client non-bloquant avec polling naïf"""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setblocking(False)

    # 1. Lancer la connexion
    try:
        sock.connect(('example.com', 80))
    except BlockingIOError:
        pass  # Normal

    # 2. Attendre que la connexion soit établie (MAUVAIS !)
    while True:
        try:
            # Tenter d'envoyer (teste si connecté)
            sock.send(b'GET / HTTP/1.1\r\n\r\n')
            print("Connecté!")
            break
        except OSError:
            # Pas encore connecté
            print("Pas encore connecté, retry...")
            time.sleep(0.1)  # GASPILLAGE DE CPU

    # 3. Lire la réponse
    response = b''
    while True:
        try:
            chunk = sock.recv(4096)
            if not chunk:
                break
            response += chunk
        except BlockingIOError:
            # Pas de données pour le moment
            time.sleep(0.01)  # GASPILLAGE DE CPU
            continue

    print(response.decode())

# PROBLÈME : Boucles actives (busy-waiting) gaspillent le CPU
```

**Pourquoi c'est mauvais ?**

```python
# CPU usage pendant le busy-waiting :
while True:
    try:
        data = sock.recv(1024)
    except BlockingIOError:
        time.sleep(0.001)  # 1ms
        # CPU à 100% sur ce thread !
```

**Solution** : Multiplexage I/O (section 8.6)

## Comparaison détaillée

### Tableau comparatif

| Aspect | I/O Bloquant | I/O Non-bloquant |
|--------|--------------|------------------|
| **Thread bloqué ?** | ✅ Oui, jusqu'à complétion | ❌ Non, retour immédiat |
| **Simplicité** | ✅ Code séquentiel simple | ❌ Plus complexe (état) |
| **Scalabilité** | ❌ Limitée (threads) | ✅ Excellente (event loop) |
| **CPU usage** | ✅ Faible (thread sleep) | ⚠️ Élevé (polling naïf) |
| **Latence** | ⚠️ Variable (context switch) | ✅ Prévisible |
| **Débogage** | ✅ Plus facile | ❌ Plus difficile |
| **Cas d'usage** | Peu de connexions | Beaucoup de connexions |

### Code comparatif

#### Bloquant

```python
# I/O BLOQUANT : Simple et séquentiel
def handle_client_blocking(sock):
    """Gestion bloquante d'un client"""
    # 1. Lire la requête (bloque)
    request = sock.recv(1024)

    # 2. Traiter
    response = process_request(request)

    # 3. Envoyer (bloque)
    sock.sendall(response)

    # 4. Fermer
    sock.close()

# Code SIMPLE, LISIBLE, SÉQUENTIEL
# Mais nécessite 1 thread par client
```

#### Non-bloquant

```python
# I/O NON-BLOQUANT : Plus complexe
class ClientHandler:
    """Gestion non-bloquante d'un client (machine à états)"""

    def __init__(self, sock):
        self.sock = sock
        self.state = 'READING'
        self.buffer_in = b''
        self.buffer_out = b''

    def handle(self):
        """Appelé répétitivement par l'event loop"""
        if self.state == 'READING':
            try:
                chunk = self.sock.recv(1024)
                if not chunk:
                    self.state = 'CLOSED'
                    return

                self.buffer_in += chunk

                if b'\r\n\r\n' in self.buffer_in:
                    # Requête complète
                    self.buffer_out = process_request(self.buffer_in)
                    self.state = 'WRITING'

            except BlockingIOError:
                # Pas de données, on reviendra plus tard
                pass

        elif self.state == 'WRITING':
            try:
                sent = self.sock.send(self.buffer_out)
                self.buffer_out = self.buffer_out[sent:]

                if not self.buffer_out:
                    # Tout envoyé
                    self.state = 'CLOSED'

            except BlockingIOError:
                # Buffer plein, on reviendra plus tard
                pass

        elif self.state == 'CLOSED':
            self.sock.close()

# Code COMPLEXE (machine à états)
# Mais un seul thread peut gérer TOUS les clients
```

### Performances comparées

```python
import socket
import threading
import time

# Benchmark : 1000 requêtes HTTP

# 1. I/O Bloquant + Threads
def benchmark_blocking():
    def fetch(url):
        sock = socket.socket()
        sock.connect(('example.com', 80))
        sock.send(b'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n')
        response = sock.recv(4096)
        sock.close()
        return response

    start = time.time()
    threads = []
    for i in range(1000):
        t = threading.Thread(target=fetch, args=('http://example.com',))
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

    duration = time.time() - start
    print(f"Bloquant + Threads: {duration:.2f}s")
    # Résultat typique : 15-30s (overhead de threads)

# 2. I/O Non-bloquant + Event Loop (asyncio)
import asyncio

async def benchmark_nonblocking():
    async def fetch(url):
        reader, writer = await asyncio.open_connection('example.com', 80)
        writer.write(b'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n')
        response = await reader.read(4096)
        writer.close()
        return response

    start = time.time()
    tasks = [fetch('http://example.com') for _ in range(1000)]
    await asyncio.gather(*tasks)

    duration = time.time() - start
    print(f"Non-bloquant + asyncio: {duration:.2f}s")
    # Résultat typique : 2-5s (pas d'overhead de threads)

# asyncio.run(benchmark_nonblocking())
```

**Résultats typiques** :
```
1000 connexions simultanées :
- Bloquant + Threads : 20s, 8 GB RAM
- Non-bloquant + Event loop : 3s, 100 MB RAM

10000 connexions :
- Bloquant + Threads : IMPOSSIBLE (out of memory)
- Non-bloquant + Event loop : 25s, 500 MB RAM
```

## Patterns de concurrence

### Pattern 1 : Thread-per-connection (Bloquant)

```python
import socket
import threading

class ThreadedServer:
    """Serveur avec un thread par connexion"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    def start(self):
        self.server.bind((self.host, self.port))
        self.server.listen(100)
        print(f"Serveur thread-per-connection sur {self.host}:{self.port}")

        while True:
            client, addr = self.server.accept()

            # Nouveau thread pour chaque client
            thread = threading.Thread(
                target=self.handle_client,
                args=(client, addr)
            )
            thread.daemon = True
            thread.start()

    def handle_client(self, sock, addr):
        """Thread dédié pour un client"""
        print(f"[Thread {threading.current_thread().name}] Client {addr}")

        try:
            while True:
                data = sock.recv(1024)
                if not data:
                    break
                sock.sendall(b"Echo: " + data)
        finally:
            sock.close()

# AVANTAGES :
# ✅ Code simple et lisible
# ✅ Chaque client isolé dans son thread
# ✅ Gestion d'erreur simple (exception dans un thread = autres OK)

# INCONVÉNIENTS :
# ❌ Limité à ~1000 connexions
# ❌ Overhead mémoire (8 MB/thread)
# ❌ Context switching coûteux
```

**Cas d'usage appropriés** :
- Serveurs avec < 100 connexions simultanées
- Applications où chaque connexion fait du CPU intensif
- Prototypage rapide

### Pattern 2 : Thread Pool (Bloquant)

```python
import socket
import threading
from queue import Queue

class ThreadPoolServer:
    """Serveur avec pool de threads (nombre fixe)"""

    def __init__(self, host='0.0.0.0', port=8080, num_workers=10):
        self.host = host
        self.port = port
        self.num_workers = num_workers

        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        # File de clients en attente
        self.client_queue = Queue()

        # Pool de workers
        self.workers = []

    def start(self):
        self.server.bind((self.host, self.port))
        self.server.listen(100)

        # Démarrer les workers
        for i in range(self.num_workers):
            worker = threading.Thread(target=self.worker_thread, args=(i,))
            worker.daemon = True
            worker.start()
            self.workers.append(worker)

        print(f"Serveur avec {self.num_workers} workers sur {self.host}:{self.port}")

        # Accepter les clients et les mettre en queue
        while True:
            client, addr = self.server.accept()
            print(f"Client {addr} ajouté à la queue")
            self.client_queue.put((client, addr))

    def worker_thread(self, worker_id):
        """Worker qui traite les clients de la queue"""
        print(f"Worker {worker_id} démarré")

        while True:
            # Attendre un client (bloquant sur queue)
            client, addr = self.client_queue.get()

            print(f"[Worker {worker_id}] Traite client {addr}")

            try:
                while True:
                    data = client.recv(1024)
                    if not data:
                        break
                    client.sendall(b"Echo: " + data)
            finally:
                client.close()
                print(f"[Worker {worker_id}] Client {addr} déconnecté")
                self.client_queue.task_done()

# AVANTAGES :
# ✅ Nombre de threads contrôlé (pas d'explosion)
# ✅ Meilleure utilisation des ressources
# ✅ Code relativement simple

# INCONVÉNIENTS :
# ❌ Si tous les workers bloqués, nouveaux clients attendent
# ❌ Toujours limité par le nombre de threads
```

**Cas d'usage** :
- Serveurs avec charge variable
- Limitation des ressources (RAM, CPU)
- Traitement de tâches de durée similaire

### Pattern 3 : Event Loop (Non-bloquant)

```python
import socket
import select

class EventLoopServer:
    """Serveur avec event loop (I/O non-bloquant)"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

        # Socket serveur
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.setblocking(False)  # Non-bloquant !

        # Toutes les sockets à surveiller
        self.sockets = []

        # Buffers pour chaque client
        self.buffers = {}

    def start(self):
        self.server.bind((self.host, self.port))
        self.server.listen(100)
        self.sockets.append(self.server)

        print(f"Serveur event-loop sur {self.host}:{self.port}")

        # Event loop principal (SINGLE THREAD !)
        while True:
            # select() bloque jusqu'à activité sur une socket
            readable, writable, exceptional = select.select(
                self.sockets,  # Sockets à lire
                [],            # Sockets à écrire
                self.sockets,  # Sockets en erreur
                1.0            # Timeout 1s
            )

            # Traiter les sockets prêtes
            for sock in readable:
                if sock is self.server:
                    # Nouvelle connexion
                    self.accept_client()
                else:
                    # Données d'un client
                    self.handle_client(sock)

            # Traiter les erreurs
            for sock in exceptional:
                self.close_client(sock)

    def accept_client(self):
        """Accepte un nouveau client"""
        try:
            client, addr = self.server.accept()
            client.setblocking(False)  # Non-bloquant !

            self.sockets.append(client)
            self.buffers[client] = b''

            print(f"Client connecté: {addr}")

        except BlockingIOError:
            # Pas de client (ne devrait pas arriver)
            pass

    def handle_client(self, sock):
        """Gère les données d'un client"""
        try:
            data = sock.recv(1024)

            if not data:
                # Client déconnecté
                self.close_client(sock)
                return

            # Echo
            sock.send(b"Echo: " + data)

        except BlockingIOError:
            # Pas de données (ne devrait pas arriver après select)
            pass

        except Exception as e:
            print(f"Erreur client: {e}")
            self.close_client(sock)

    def close_client(self, sock):
        """Ferme un client"""
        print(f"Fermeture client")
        self.sockets.remove(sock)
        if sock in self.buffers:
            del self.buffers[sock]
        sock.close()

# AVANTAGES :
# ✅ UN SEUL THREAD pour TOUS les clients
# ✅ Très scalable (10000+ connexions)
# ✅ Faible consommation mémoire
# ✅ Pas de context switching

# INCONVÉNIENTS :
# ❌ Code plus complexe (machine à états)
# ❌ Une opération bloquante bloque TOUT
# ❌ Debugging plus difficile
```

**Cas d'usage** :
- Serveurs haute performance (Nginx, Redis, Node.js)
- Beaucoup de connexions I/O-bound
- Microservices

## Cas d'usage réels par industrie

### 1. Serveur Web (Nginx vs Apache)

**Apache (mode prefork - I/O bloquant)** :

```python
# Modèle Apache prefork (simplifié)
class ApachePrefork:
    """Process par connexion (bloquant)"""

    def __init__(self, num_processes=10):
        self.num_processes = num_processes

    def start(self):
        # Fork plusieurs processus workers
        for i in range(self.num_processes):
            pid = os.fork()
            if pid == 0:  # Child process
                self.worker()
                exit(0)

        # Parent attend
        for i in range(self.num_processes):
            os.wait()

    def worker(self):
        """Worker process (bloquant)"""
        server = socket.socket()
        server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server.bind(('0.0.0.0', 8080))
        server.listen(100)

        while True:
            client, addr = server.accept()  # Bloque
            self.handle_request(client)
            client.close()

# Limite : ~256 processus = ~256 requêtes simultanées
```

**Nginx (event-driven - I/O non-bloquant)** :

```python
# Modèle Nginx (simplifié)
class NginxEventDriven:
    """Event loop par worker (non-bloquant)"""

    def __init__(self, num_workers=4):
        self.num_workers = num_workers

    def start(self):
        # Fork plusieurs workers
        for i in range(self.num_workers):
            pid = os.fork()
            if pid == 0:
                self.event_loop_worker()
                exit(0)

    def event_loop_worker(self):
        """Worker avec event loop (epoll)"""
        # Utilise epoll (Linux) pour 10000+ connexions
        epoll = select.epoll()

        server = socket.socket()
        server.setblocking(False)
        server.bind(('0.0.0.0', 8080))
        server.listen(1024)

        epoll.register(server.fileno(), select.EPOLLIN)

        connections = {}

        while True:
            # Attendre des événements
            events = epoll.poll(timeout=1.0)

            for fd, event in events:
                if fd == server.fileno():
                    # Nouvelle connexion
                    client, addr = server.accept()
                    client.setblocking(False)
                    epoll.register(client.fileno(), select.EPOLLIN)
                    connections[client.fileno()] = client

                elif event & select.EPOLLIN:
                    # Données à lire
                    # ... traiter sans bloquer

# Capacité : 10000+ connexions par worker
# 4 workers = 40000+ connexions simultanées
```

**Résultats réels** :

```
Apache (prefork) :
- 256 connexions simultanées max
- 1 GB RAM pour 256 processus
- ~5000 req/sec

Nginx (event-driven) :
- 40000+ connexions simultanées
- 200 MB RAM pour 4 workers
- ~50000 req/sec

→ 10x plus de connexions, 5x moins de RAM, 10x plus de débit
```

### 2. Base de données (Redis)

**Redis** : Architecture single-threaded event-loop

```python
# Modèle Redis (très simplifié)
class RedisLikeServer:
    """Serveur clé-valeur single-thread (comme Redis)"""

    def __init__(self):
        self.data = {}  # Store clé-valeur
        self.sockets = []
        self.buffers = {}

    def event_loop(self):
        """Event loop principal (SINGLE THREAD)"""
        epoll = select.epoll()

        server = socket.socket()
        server.setblocking(False)
        server.bind(('0.0.0.0', 6379))
        server.listen(1000)

        epoll.register(server.fileno(), select.EPOLLIN)

        while True:
            events = epoll.poll()

            for fd, event in events:
                # Accepter, lire, traiter, répondre
                # TOUT dans le même thread !
                # Pas de locks nécessaires (single-thread)
                self.handle_event(fd, event)

    def handle_command(self, command):
        """Traite une commande Redis"""
        parts = command.split()

        if parts[0] == b'GET':
            key = parts[1]
            value = self.data.get(key, b'(nil)')
            return value

        elif parts[0] == b'SET':
            key = parts[1]
            value = parts[2]
            self.data[key] = value  # Pas de lock nécessaire !
            return b'OK'

# Pourquoi single-thread ?
# - Pas de contention sur les locks
# - Code simple (pas de sync)
# - Performance : 100000+ ops/sec
# - Toutes les opérations sont rapides (< 1ms)
```

**Pourquoi Redis est si rapide ?**

1. **Single-thread** : Pas de context switching, pas de locks
2. **Event-driven** : I/O non-bloquant
3. **In-memory** : Aucune I/O disque bloquante
4. **Opérations atomiques** : Pas de transactions complexes

```
Redis performance :
- 100000+ GET/SET par seconde
- Latence < 1ms
- 10000+ connexions simultanées
- RAM : ~50 MB (base + données)
```

### 3. Node.js (JavaScript backend)

**Node.js** : Event loop single-threaded

```javascript
// Serveur HTTP Node.js (event-driven)
const http = require('http');

// SINGLE THREAD, mais gère des milliers de connexions
const server = http.createServer((req, res) => {
    // Cette fonction est appelée pour CHAQUE requête
    // Mais ne bloque jamais l'event loop

    // I/O non-bloquant (async)
    fs.readFile('/data.json', (err, data) => {
        // Callback appelé quand le fichier est lu
        // Pendant ce temps, d'autres requêtes sont traitées
        res.writeHead(200, {'Content-Type': 'application/json'});
        res.end(data);
    });

    // Cette fonction retourne IMMÉDIATEMENT
    // Le serveur peut traiter d'autres requêtes
});

server.listen(3000);

// Performance :
// - 10000+ req/sec
// - 100 MB RAM
// - 1 CPU core
```

**Équivalent Python avec asyncio** :

```python
import asyncio

async def handle_client(reader, writer):
    """Handler async (non-bloquant)"""
    # Lecture async (ne bloque pas l'event loop)
    data = await reader.read(1024)

    # Traitement async
    response = await process_data(data)

    # Écriture async
    writer.write(response)
    await writer.drain()

    writer.close()

async def main():
    # Serveur async
    server = await asyncio.start_server(
        handle_client,
        '0.0.0.0',
        8080
    )

    async with server:
        await server.serve_forever()

# UN SEUL THREAD, des milliers de connexions
asyncio.run(main())
```

### 4. Chat en temps réel (Discord, Slack)

```python
import asyncio
import websockets

class ChatServer:
    """Serveur de chat avec WebSocket (async)"""

    def __init__(self):
        self.clients = set()  # Tous les clients connectés

    async def handler(self, websocket, path):
        """Gère un client WebSocket"""
        # Ajouter le client
        self.clients.add(websocket)
        print(f"Client connecté. Total: {len(self.clients)}")

        try:
            # Boucle de réception (async)
            async for message in websocket:
                # Broadcast à tous les clients
                await self.broadcast(message, websocket)

        finally:
            # Retirer le client
            self.clients.remove(websocket)
            print(f"Client déconnecté. Total: {len(self.clients)}")

    async def broadcast(self, message, sender):
        """Envoie un message à tous les clients (sauf l'émetteur)"""
        # Envoyer en parallèle à tous les clients
        tasks = []
        for client in self.clients:
            if client != sender:
                tasks.append(client.send(message))

        # Attendre toutes les envois (async)
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)

    async def start(self):
        """Démarre le serveur"""
        async with websockets.serve(self.handler, '0.0.0.0', 8765):
            print("Serveur de chat démarré sur :8765")
            await asyncio.Future()  # Run forever

# Capacité :
# - 100000+ clients connectés simultanément
# - Un seul processus/thread
# - RAM : ~1 GB (10 KB par client)
# - Latence : < 10ms

# Discord utilise ce modèle (Elixir/Erlang, principes similaires)
# pour gérer des MILLIONS d'utilisateurs connectés
```

## Quand utiliser I/O bloquant vs non-bloquant ?

### Utiliser I/O BLOQUANT quand :

✅ **Peu de connexions** (< 100 simultanées)

```python
# API interne avec 10-20 clients max
def internal_api_server():
    # Thread-per-connection est OK
    server = ThreadedTCPServer(('localhost', 9999), RequestHandler)
    server.serve_forever()
```

✅ **Code simple prioritaire**

```python
# Script one-off ou prototype
def download_file(url):
    response = requests.get(url)  # Bloquant, mais simple
    with open('file.dat', 'wb') as f:
        f.write(response.content)
```

✅ **CPU-bound plutôt que I/O-bound**

```python
# Chaque requête fait du calcul intensif
def handle_request(data):
    # Calcul qui prend 5 secondes
    result = heavy_computation(data)
    return result

# I/O bloquant + thread pool est adapté
```

✅ **Intégration avec code legacy bloquant**

```python
# Bibliothèque tierce qui bloque
import legacy_library

def use_legacy():
    result = legacy_library.blocking_call()
    # Difficile de rendre async sans réécrire
```

### Utiliser I/O NON-BLOQUANT quand :

✅ **Beaucoup de connexions** (> 1000 simultanées)

```python
# Serveur web public, API publique
# WebSocket server avec 10000+ clients
async def websocket_server():
    # Event loop gère tous les clients efficacement
    async with websockets.serve(handler, '0.0.0.0', 8080):
        await asyncio.Future()
```

✅ **I/O-bound (réseau, disque)**

```python
# Serveur proxy qui fait beaucoup d'I/O réseau
async def proxy_handler(request):
    # Appel réseau 1
    user_data = await fetch_user_service(request.user_id)

    # Appel réseau 2 (en parallèle)
    products = await fetch_product_service(request.query)

    # Appel réseau 3
    recommendations = await fetch_recommendations(user_data)

    return merge(user_data, products, recommendations)
```

✅ **Latence critique**

```python
# Trading haute fréquence
async def trading_engine():
    # Chaque milliseconde compte
    # Event loop = latence prévisible
    async for market_data in stream:
        await process_order(market_data)
```

✅ **Temps réel**

```python
# Gaming server, streaming, chat
async def game_server():
    # Besoin de gérer 10000+ joueurs simultanément
    # avec latence < 50ms
    while True:
        await broadcast_game_state()
        await asyncio.sleep(0.016)  # 60 FPS
```

## Patterns hybrides

### 1. Async + Thread Pool pour CPU

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

# Event loop pour I/O
# Thread pool pour CPU

async def handle_request(data):
    """Handler async avec calcul CPU dans thread pool"""
    # I/O async (non-bloquant)
    raw_data = await fetch_from_database(data.id)

    # CPU-bound dans thread pool (pour ne pas bloquer event loop)
    loop = asyncio.get_event_loop()
    executor = ThreadPoolExecutor(max_workers=4)

    result = await loop.run_in_executor(
        executor,
        heavy_computation,  # Fonction CPU-bound
        raw_data
    )

    # I/O async
    await save_to_database(result)

    return result

def heavy_computation(data):
    """Calcul intensif (bloquant)"""
    # Tourne dans un thread séparé
    return complex_algorithm(data)

# Meilleur des deux mondes :
# - I/O async pour scalabilité
# - Threads pour CPU-bound
```

### 2. Gevent (Green Threads)

```python
from gevent import monkey
monkey.patch_all()  # Patch stdlib pour rendre async

from gevent.pool import Pool
from gevent.server import StreamServer
import requests

# Code qui RESSEMBLE à du bloquant
# Mais tourne en async grâce à gevent
def handle_client(sock, addr):
    """Handler qui ressemble bloquant mais est async"""
    # Ces appels SEMBLENT bloquants
    # Mais gevent les rend async automatiquement
    data = sock.recv(1024)

    # Requête HTTP (monkey-patched = async)
    response = requests.get('http://api.example.com')

    sock.send(response.content)
    sock.close()

# Serveur avec pool de greenlets
pool = Pool(10000)  # 10000 "threads" légers
server = StreamServer(('0.0.0.0', 8080), handle_client, spawn=pool)
server.serve_forever()

# AVANTAGES :
# ✅ Code simple (style bloquant)
# ✅ Performance async
# ✅ Scalabilité (10000+ connexions)

# INCONVÉNIENTS :
# ❌ Monkey-patching peut casser certaines libs
# ❌ Moins de contrôle qu'asyncio
```

## Debugging et profiling

### Debugging I/O bloquant

```python
import socket
import sys
import traceback

def debug_blocking_recv(sock, size):
    """recv() avec debug de blocage"""
    print(f"[DEBUG] Début recv({size})")
    print(f"[DEBUG] Stack trace:")
    traceback.print_stack()

    import time
    start = time.time()

    data = sock.recv(size)

    duration = time.time() - start
    print(f"[DEBUG] recv() bloqué pendant {duration:.3f}s")

    return data

# Utilisation pour identifier où le code bloque
```

### Profiling event loop

```python
import asyncio
import time

class ProfilingEventLoop:
    """Event loop avec profiling"""

    def __init__(self):
        self.task_times = {}

    async def run_task(self, name, coro):
        """Exécute une coroutine avec timing"""
        start = time.time()

        result = await coro

        duration = time.time() - start
        self.task_times[name] = duration

        if duration > 0.1:  # > 100ms
            print(f"⚠️  Task '{name}' trop lent: {duration:.3f}s")

        return result

    def print_report(self):
        """Affiche le rapport"""
        print("\n=== Event Loop Profiling ===")
        for name, duration in sorted(self.task_times.items(), key=lambda x: x[1], reverse=True):
            print(f"{name}: {duration*1000:.2f}ms")

# Utilisation
profiler = ProfilingEventLoop()

async def main():
    await profiler.run_task('fetch_user', fetch_user())
    await profiler.run_task('fetch_products', fetch_products())

    profiler.print_report()
```

## Récapitulatif

### Points clés

🎯 **I/O Bloquant** :
- Simple à comprendre et coder
- Limité en scalabilité (threads)
- Bon pour peu de connexions ou CPU-bound

🎯 **I/O Non-bloquant** :
- Scalable (10000+ connexions)
- Plus complexe (machine à états)
- Idéal pour I/O-bound et temps réel

🎯 **Choix dépend de** :
- Nombre de connexions simultanées
- I/O-bound vs CPU-bound
- Simplicité vs Performance
- Compétences de l'équipe

### Tableau décisionnel rapide

| Critère | Bloquant | Non-bloquant |
|---------|----------|--------------|
| < 100 connexions | ✅ | ⚠️ Overkill |
| 100-1000 connexions | ⚠️ Limite | ✅ |
| > 1000 connexions | ❌ | ✅ |
| Code simple | ✅ | ❌ |
| Performance max | ❌ | ✅ |
| CPU-bound | ✅ | ⚠️ + threads |
| I/O-bound | ⚠️ | ✅ |
| Temps réel | ❌ | ✅ |

## Prochaines étapes

Maintenant que vous comprenez I/O bloquant vs non-bloquant, nous allons approfondir :

- **Section 8.6** : Multiplexage (select, poll, epoll, kqueue) - Comment surveiller plusieurs sockets efficacement
- **Section 8.7** : Programmation asynchrone et event-driven - asyncio, patterns modernes
- **Section 8.8** : Résilience et fiabilité applicative

---


⏭️ [Multiplexage : select, poll, epoll](/08-programmation-reseau/06-multiplexage.md)
