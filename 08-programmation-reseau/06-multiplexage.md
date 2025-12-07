🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.6 Multiplexage : select, poll, epoll

## Introduction

Dans la section précédente, nous avons vu que l'I/O non-bloquant permet de gérer plusieurs connexions sans créer de threads. Mais un problème subsiste : **comment savoir quand une socket est prête** pour la lecture ou l'écriture ?

Le **multiplexage I/O** résout ce problème en permettant de **surveiller simultanément plusieurs sockets** et d'être notifié quand l'une d'elles est prête pour une opération.

```python
# SANS multiplexage (polling naïf - MAUVAIS)
while True:
    for sock in all_sockets:
        try:
            data = sock.recv(1024)  # Non-bloquant
        except BlockingIOError:
            pass  # Pas de données
        # Boucle active = CPU à 100% !

# AVEC multiplexage (BIEN)
readable, _, _ = select.select(all_sockets, [], [])
for sock in readable:
    data = sock.recv(1024)  # Garantie d'avoir des données
```

## Le problème à résoudre

### Scénario : Serveur avec 10 000 clients

```python
# Problème : Comment savoir quelle socket a des données ?

clients = [sock1, sock2, sock3, ..., sock10000]

# Option 1 : Polling naïf (TERRIBLE)
for sock in clients:
    try:
        data = sock.recv(1024)
    except BlockingIOError:
        pass  # 99% du temps, pas de données
# CPU à 100%, inefficace

# Option 2 : Thread par socket (IMPOSSIBLE)
for sock in clients:
    threading.Thread(target=handle_socket, args=(sock,)).start()
# 10000 threads = 80 GB RAM, crash

# Option 3 : Multiplexage (SOLUTION)
readable = wait_for_activity(clients)
for sock in readable:
    data = sock.recv(1024)  # Garantie d'avoir des données
# CPU efficient, scalable
```

### Visualisation du problème

```
10 000 sockets :
[S1] [S2] [S3] [S4] [S5] ... [S9999] [S10000]

À un instant T :
- S1 : pas de données
- S2 : PAS DE DONNÉES
- S3 : pas de données
- ...
- S234 : 🔔 DONNÉES DISPONIBLES
- ...
- S9876 : 🔔 DONNÉES DISPONIBLES
- ...

Comment trouver S234 et S9876 efficacement ?
→ Multiplexage !
```

## select() - Le pionnier (1983)

### Principe

`select()` surveille des ensembles de descripteurs de fichiers et bloque jusqu'à ce qu'au moins un soit prêt.

```python
import select

# Surveiller des sockets
readable, writable, exceptional = select.select(
    rlist,    # Sockets à surveiller en lecture
    wlist,    # Sockets à surveiller en écriture
    xlist,    # Sockets à surveiller pour exceptions
    timeout   # Timeout en secondes (None = infini)
)
```

### Exemple basique

```python
import socket
import select

def select_echo_server():
    """Serveur echo avec select()"""
    # Socket serveur
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.setblocking(False)
    server.bind(('0.0.0.0', 8080))
    server.listen(5)

    # Liste de toutes les sockets à surveiller
    inputs = [server]
    outputs = []

    print("Serveur select() en écoute sur :8080")

    while inputs:
        # select() bloque jusqu'à activité
        readable, writable, exceptional = select.select(
            inputs,   # Surveiller en lecture
            outputs,  # Surveiller en écriture
            inputs,   # Surveiller les erreurs
            1.0       # Timeout 1 seconde
        )

        # Traiter les sockets prêtes en lecture
        for sock in readable:
            if sock is server:
                # Nouvelle connexion
                client, addr = sock.accept()
                client.setblocking(False)
                inputs.append(client)
                print(f"Client connecté: {addr}")
            else:
                # Données d'un client
                try:
                    data = sock.recv(1024)
                    if data:
                        print(f"Reçu: {data}")
                        # Préparer la réponse
                        sock.send(b"Echo: " + data)
                    else:
                        # Client déconnecté
                        print(f"Client déconnecté")
                        inputs.remove(sock)
                        sock.close()
                except Exception as e:
                    print(f"Erreur: {e}")
                    inputs.remove(sock)
                    sock.close()

        # Traiter les erreurs
        for sock in exceptional:
            print(f"Exception sur socket")
            inputs.remove(sock)
            sock.close()

if __name__ == "__main__":
    select_echo_server()
```

### Anatomie de select()

```python
import select

# Préparation
rlist = [sock1, sock2, sock3]  # Surveiller en lecture
wlist = [sock4]                 # Surveiller en écriture
xlist = rlist                   # Surveiller les exceptions

# Appel bloquant
readable, writable, exceptional = select.select(rlist, wlist, xlist, timeout=5.0)

# Résultats
print(f"Prêtes en lecture: {len(readable)}")
print(f"Prêtes en écriture: {len(writable)}")
print(f"En exception: {len(exceptional)}")
```

**Que retourne select() ?**

```python
readable, writable, exceptional = select.select([s1, s2, s3], [], [], 1.0)

# readable contient les sockets qui :
# - Ont des données à lire
# - Ont une connexion entrante (pour listen socket)
# - Sont fermées (recv() retournera b'')

# writable contient les sockets qui :
# - Ont de l'espace dans le buffer d'envoi
# - Sont connectées (pour connect() en cours)

# exceptional contient les sockets qui :
# - Ont une erreur
# - Ont des données out-of-band (rare)
```

### Limitations de select()

#### 1. Limite de descripteurs (FD_SETSIZE)

```python
import select

# Sur la plupart des systèmes : FD_SETSIZE = 1024
# Impossible de surveiller plus de 1024 sockets !

sockets = []
for i in range(2000):
    sock = socket.socket()
    sockets.append(sock)

try:
    readable, _, _ = select.select(sockets, [], [])
except ValueError as e:
    print(f"Erreur: {e}")
    # ValueError: file descriptor out of range in select()
```

**Pourquoi cette limite ?**

```c
// Implémentation interne de select() (C)
#define FD_SETSIZE 1024

typedef struct {
    unsigned long fds_bits[FD_SETSIZE/8];  // Bitmap
} fd_set;

// Chaque bit représente un descripteur
// → Maximum 1024 descripteurs
```

#### 2. Performance O(n)

```python
# select() parcourt TOUS les descripteurs
sockets = [s1, s2, s3, ..., s1000]

readable, _, _ = select.select(sockets, [], [])
# Même si seulement s234 est prêt,
# select() doit vérifier les 1000 sockets !

# Complexité : O(n) où n = nombre de sockets
# 1000 sockets : ~1000 vérifications
# 10000 sockets : IMPOSSIBLE (limite FD_SETSIZE)
```

#### 3. Copie des ensembles

```python
import select

sockets = [s1, s2, ..., s100]

# select() MODIFIE les listes passées
readable, _, _ = select.select(sockets, [], [])

# sockets est maintenant vide !
# Il faut la recréer à chaque itération

# MAUVAIS (boucle naïve)
while True:
    readable, _, _ = select.select(sockets, [], [])
    # sockets est vide après le premier appel !

# BON (copie à chaque itération)
all_sockets = [s1, s2, ..., s100]
while True:
    readable, _, _ = select.select(all_sockets[:], [], [])
    # all_sockets[:] crée une copie
```

### Cas d'usage appropriés pour select()

✅ **Portabilité maximale** : select() fonctionne partout (Windows, Linux, macOS, BSD)

```python
# Code portable cross-platform
import select
import socket

# Fonctionne sur Windows, Linux, macOS
readable, _, _ = select.select([sock], [], [], 1.0)
```

✅ **Peu de sockets** (< 100) : Performance acceptable

```python
# Serveur interne avec ~10 connexions
def simple_server():
    sockets = [server_socket]
    while True:
        readable, _, _ = select.select(sockets, [], [], 1.0)
        # Seulement 10 sockets, O(n) acceptable
```

✅ **Code legacy** : Beaucoup de code existant utilise select()

- ❌ **Serveurs haute performance** : Utilisez poll() ou epoll()
- ❌ **> 1000 connexions** : Limite FD_SETSIZE

### Exemple complet : Serveur chat avec select()

```python
import socket
import select

class SelectChatServer:
    """Serveur de chat utilisant select()"""

    def __init__(self, host='0.0.0.0', port=9999):
        self.host = host
        self.port = port

        # Socket serveur
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.setblocking(False)
        self.server.bind((self.host, self.port))
        self.server.listen(5)

        # Toutes les sockets
        self.sockets = [self.server]

        # Buffers et métadonnées
        self.clients = {}  # {socket: {'addr': ..., 'buffer': ...}}

    def run(self):
        """Boucle principale"""
        print(f"Serveur chat (select) sur {self.host}:{self.port}")

        while True:
            # select() bloque jusqu'à activité
            readable, writable, exceptional = select.select(
                self.sockets,
                [],
                self.sockets,
                1.0
            )

            # Traiter les lectures
            for sock in readable:
                if sock is self.server:
                    self._accept_client()
                else:
                    self._handle_client(sock)

            # Traiter les exceptions
            for sock in exceptional:
                self._remove_client(sock)

    def _accept_client(self):
        """Accepte un nouveau client"""
        client, addr = self.server.accept()
        client.setblocking(False)

        self.sockets.append(client)
        self.clients[client] = {
            'addr': addr,
            'buffer': b''
        }

        print(f"[+] Client connecté: {addr}")

        # Message de bienvenue
        welcome = f"Bienvenue ! {len(self.clients)} utilisateurs connectés.\n"
        client.send(welcome.encode('utf-8'))

    def _handle_client(self, sock):
        """Gère les données d'un client"""
        try:
            data = sock.recv(1024)

            if not data:
                # Client déconnecté
                self._remove_client(sock)
                return

            # Accumuler dans le buffer
            self.clients[sock]['buffer'] += data

            # Traiter les messages complets (lignes)
            while b'\n' in self.clients[sock]['buffer']:
                line, self.clients[sock]['buffer'] = \
                    self.clients[sock]['buffer'].split(b'\n', 1)

                message = line.decode('utf-8').strip()
                if message:
                    self._broadcast(sock, message)

        except Exception as e:
            print(f"Erreur: {e}")
            self._remove_client(sock)

    def _broadcast(self, sender, message):
        """Diffuse un message à tous les clients"""
        sender_addr = self.clients[sender]['addr']
        formatted = f"[{sender_addr[0]}:{sender_addr[1]}] {message}\n"

        print(formatted.strip())

        # Envoyer à tous sauf l'émetteur
        for client in self.clients:
            if client != sender:
                try:
                    client.send(formatted.encode('utf-8'))
                except:
                    pass

    def _remove_client(self, sock):
        """Retire un client"""
        if sock in self.clients:
            addr = self.clients[sock]['addr']
            print(f"[-] Client déconnecté: {addr}")

            del self.clients[sock]
            self.sockets.remove(sock)
            sock.close()

if __name__ == "__main__":
    server = SelectChatServer()
    server.run()
```

**Test** :

```bash
# Terminal 1 : Serveur
python select_chat_server.py

# Terminal 2 : Client 1
telnet localhost 9999
Hello from client 1

# Terminal 3 : Client 2
telnet localhost 9999
Hi from client 2

# Les messages sont diffusés à tous les clients
```

## poll() - L'amélioration (1986)

### Principe

`poll()` résout certaines limitations de select() :
- **Pas de limite FD_SETSIZE**
- **API plus propre**
- **Ne modifie pas les listes d'entrée**

```python
import select

# Créer un objet poll
poller = select.poll()

# Enregistrer des sockets
poller.register(sock1, select.POLLIN)   # Surveiller en lecture
poller.register(sock2, select.POLLOUT)  # Surveiller en écriture

# Attendre des événements
events = poller.poll(timeout=1000)  # Timeout en millisecondes

# Traiter les événements
for fd, event in events:
    if event & select.POLLIN:
        # Prêt en lecture
        data = socket_from_fd(fd).recv(1024)
```

### Événements poll()

```python
import select

# Constantes d'événements
select.POLLIN     # Données disponibles en lecture
select.POLLOUT    # Prêt pour écriture
select.POLLERR    # Erreur
select.POLLHUP    # Connexion fermée
select.POLLNVAL   # Descripteur invalide

# Combinaison avec OR bitwise
poller.register(sock, select.POLLIN | select.POLLOUT)
```

### Exemple avec poll()

```python
import socket
import select

class PollEchoServer:
    """Serveur echo utilisant poll()"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

        # Socket serveur
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.setblocking(False)
        self.server.bind((self.host, self.port))
        self.server.listen(5)

        # Créer le poller
        self.poller = select.poll()
        self.poller.register(self.server, select.POLLIN)

        # Map fd -> socket
        self.fd_to_socket = {self.server.fileno(): self.server}

    def run(self):
        """Boucle principale"""
        print(f"Serveur poll() sur {self.host}:{self.port}")

        while True:
            # poll() retourne [(fd, event), ...]
            events = self.poller.poll(1000)  # Timeout 1000ms = 1s

            for fd, event in events:
                sock = self.fd_to_socket[fd]

                if sock is self.server:
                    # Nouvelle connexion
                    self._accept_client()

                elif event & select.POLLIN:
                    # Données à lire
                    self._handle_client(sock, fd)

                elif event & (select.POLLHUP | select.POLLERR):
                    # Erreur ou déconnexion
                    self._close_client(sock, fd)

    def _accept_client(self):
        """Accepte un nouveau client"""
        client, addr = self.server.accept()
        client.setblocking(False)

        fd = client.fileno()
        self.fd_to_socket[fd] = client

        # Enregistrer pour surveiller en lecture
        self.poller.register(client, select.POLLIN)

        print(f"Client connecté: {addr}")

    def _handle_client(self, sock, fd):
        """Gère les données d'un client"""
        try:
            data = sock.recv(1024)

            if not data:
                # Client déconnecté
                self._close_client(sock, fd)
                return

            # Echo
            sock.send(b"Echo: " + data)

        except Exception as e:
            print(f"Erreur: {e}")
            self._close_client(sock, fd)

    def _close_client(self, sock, fd):
        """Ferme un client"""
        print(f"Client déconnecté (fd={fd})")

        self.poller.unregister(sock)
        del self.fd_to_socket[fd]
        sock.close()

if __name__ == "__main__":
    server = PollEchoServer()
    server.run()
```

### Avantages de poll() sur select()

✅ **Pas de limite de descripteurs**

```python
# select() : max 1024 sockets (FD_SETSIZE)
# poll() : pas de limite (limité par RAM et ulimit)

sockets = []
poller = select.poll()

for i in range(10000):
    sock = socket.socket()
    poller.register(sock, select.POLLIN)
    sockets.append(sock)

# Fonctionne ! (jusqu'à la limite système)
events = poller.poll()
```

✅ **API plus propre**

```python
# select() : modifier les listes
all_sockets = [s1, s2, s3]
while True:
    readable, _, _ = select.select(all_sockets[:], [], [])  # Copie nécessaire

# poll() : pas de modification
poller = select.poll()
poller.register(s1, select.POLLIN)
poller.register(s2, select.POLLIN)
while True:
    events = poller.poll()  # Pas de copie
```

✅ **Meilleure performance que select()** (mais toujours O(n))

### Limitations de poll()

❌ **Toujours O(n)** : Parcourt tous les descripteurs

```python
# poll() doit vérifier TOUS les descripteurs enregistrés
poller = select.poll()
for sock in 10000_sockets:
    poller.register(sock, select.POLLIN)

events = poller.poll()
# Même si 1 seul socket est prêt,
# poll() vérifie les 10000 !
# O(n) = inefficace pour beaucoup de sockets
```

❌ **Pas disponible sur Windows**

```python
# Linux/Unix : OK
import select
poller = select.poll()

# Windows : AttributeError
# AttributeError: module 'select' has no attribute 'poll'
```

## epoll() - La solution moderne (Linux 2.5.44, 2002)

### Principe révolutionnaire

epoll() change le paradigme :
- **O(1)** au lieu de O(n) : Retourne **seulement** les descripteurs prêts
- **Edge-triggered** en option : Notifie uniquement sur changement d'état
- **Scalable** : Gère facilement 100 000+ connexions

```python
import select

# Créer un objet epoll
epoll = select.epoll()

# Enregistrer des sockets
epoll.register(sock.fileno(), select.EPOLLIN)

# Attendre des événements (retourne SEULEMENT les prêts !)
events = epoll.poll(timeout=1.0)

# events contient seulement les fd prêts
for fd, event in events:
    # Traiter seulement les sockets actives
    pass
```

### Architecture d'epoll

```
Application                     Kernel
    |                              |
    | epoll_create()               |
    |----------------------------->|
    |                              | Crée structure epoll
    |                              |
    | epoll_ctl(ADD, sock1)        |
    |----------------------------->|
    |                              | Ajoute sock1 à la "watch list"
    |                              |
    | epoll_ctl(ADD, sock2)        |
    |----------------------------->|
    |                              | Ajoute sock2
    |                              |
    | epoll_wait()                 |
    |----------------------------->|
    |                              | Bloque jusqu'à événement
    |                              |
    |      [sock2 reçoit données]  |
    |                              |
    | <- [fd=sock2, EPOLLIN]       |
    |<-----------------------------|
    |                              | Retourne SEULEMENT sock2 !
```

**Différence cruciale** :

```python
# select/poll : O(n) - Vérifie TOUS les descripteurs
for fd in all_fds:
    if fd_is_ready(fd):
        return fd

# epoll : O(1) - Le kernel SAIT lesquels sont prêts
# Retourne directement la liste (maintenue par le kernel)
```

### Exemple avec epoll()

```python
import socket
import select

class EpollEchoServer:
    """Serveur echo haute performance avec epoll()"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

        # Socket serveur
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.setblocking(False)
        self.server.bind((self.host, self.port))
        self.server.listen(1024)  # Backlog élevé

        # Créer epoll
        self.epoll = select.epoll()
        self.epoll.register(self.server.fileno(), select.EPOLLIN)

        # Map fd -> socket
        self.connections = {}
        self.fd_to_socket = {self.server.fileno(): self.server}

    def run(self):
        """Boucle principale"""
        print(f"Serveur epoll() sur {self.host}:{self.port}")

        try:
            while True:
                # epoll.poll() retourne SEULEMENT les fd prêts
                events = self.epoll.poll(timeout=1.0)

                # Traiter seulement les événements
                for fd, event in events:
                    if fd == self.server.fileno():
                        # Nouvelle connexion
                        self._accept_client()

                    elif event & select.EPOLLIN:
                        # Données à lire
                        self._handle_client(fd)

                    elif event & select.EPOLLHUP:
                        # Client déconnecté
                        self._close_client(fd)

        finally:
            self.epoll.unregister(self.server.fileno())
            self.epoll.close()
            self.server.close()

    def _accept_client(self):
        """Accepte un nouveau client"""
        try:
            while True:  # Accepter tous les clients en attente
                client, addr = self.server.accept()
                client.setblocking(False)

                fd = client.fileno()

                # Enregistrer dans epoll
                self.epoll.register(fd, select.EPOLLIN)

                self.connections[fd] = {
                    'socket': client,
                    'addr': addr,
                    'buffer': b''
                }
                self.fd_to_socket[fd] = client

                print(f"[+] Client {addr} connecté (fd={fd})")

        except BlockingIOError:
            # Plus de clients en attente
            pass

    def _handle_client(self, fd):
        """Gère les données d'un client"""
        try:
            sock = self.connections[fd]['socket']
            data = sock.recv(4096)

            if not data:
                # EOF : client déconnecté
                self._close_client(fd)
                return

            # Echo
            sock.send(b"Echo: " + data)

        except Exception as e:
            print(f"Erreur fd={fd}: {e}")
            self._close_client(fd)

    def _close_client(self, fd):
        """Ferme un client"""
        if fd in self.connections:
            addr = self.connections[fd]['addr']
            sock = self.connections[fd]['socket']

            print(f"[-] Client {addr} déconnecté (fd={fd})")

            # Désenregistrer de epoll
            self.epoll.unregister(fd)

            # Nettoyer
            del self.connections[fd]
            del self.fd_to_socket[fd]
            sock.close()

if __name__ == "__main__":
    server = EpollEchoServer()
    server.run()
```

### Modes epoll : Level-triggered vs Edge-triggered

#### Level-triggered (défaut)

```python
# Mode LEVEL-TRIGGERED (défaut)
epoll.register(fd, select.EPOLLIN)

# Comportement :
# - epoll notifie TANT QUE des données sont disponibles
# - Si recv() ne lit pas tout, epoll notifie à nouveau

sock.recv(100)  # Lit 100 octets
# Il reste 900 octets dans le buffer
# → epoll.poll() retourne encore ce fd !
```

**Exemple** :

```python
epoll = select.epoll()
epoll.register(sock.fileno(), select.EPOLLIN)

while True:
    events = epoll.poll()
    for fd, event in events:
        data = sock.recv(100)  # Lit seulement 100 octets
        # S'il y a plus de données, epoll notifie à nouveau
```

#### Edge-triggered (mode avancé)

```python
# Mode EDGE-TRIGGERED
epoll.register(fd, select.EPOLLIN | select.EPOLLET)

# Comportement :
# - epoll notifie SEULEMENT sur CHANGEMENT d'état
# - Il faut lire TOUT le buffer en une fois

while True:
    try:
        data = sock.recv(4096)
        if not data:
            break
        process(data)
    except BlockingIOError:
        break  # Tout lu
# epoll ne notifiera plus jusqu'à NOUVELLES données
```

**Comparaison** :

```python
# LEVEL-TRIGGERED
# Buffer socket : [1000 octets]
events = epoll.poll()  # Notifie fd
recv(100)  # Lit 100 octets
# Buffer : [900 octets restants]
events = epoll.poll()  # Notifie ENCORE fd
recv(100)  # Lit 100 octets
# ...

# EDGE-TRIGGERED
# Buffer socket : [1000 octets]
events = epoll.poll()  # Notifie fd
recv(100)  # Lit 100 octets
# Buffer : [900 octets restants]
events = epoll.poll()  # PAS de notification !
# Il faut avoir LU TOUT lors de la première notification
```

**Pourquoi edge-triggered ?**

✅ **Performance** : Moins d'appels système

```python
# Level-triggered : 10 notifications si on lit par petits morceaux
# Edge-triggered : 1 notification, on lit tout
```

❌ **Plus complexe** : Il faut lire tout le buffer

```python
# Edge-triggered : IL FAUT lire jusqu'à BlockingIOError
while True:
    try:
        chunk = sock.recv(4096)
        if not chunk:
            break
        buffer += chunk
    except BlockingIOError:
        break  # Tout lu
```

### Exemple edge-triggered

```python
import socket
import select

class EdgeTriggeredServer:
    """Serveur utilisant epoll en mode edge-triggered"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.setblocking(False)
        self.server.bind((self.host, self.port))
        self.server.listen(1024)

        # epoll en mode edge-triggered
        self.epoll = select.epoll()
        self.epoll.register(
            self.server.fileno(),
            select.EPOLLIN | select.EPOLLET  # Edge-triggered !
        )

        self.connections = {}

    def run(self):
        """Boucle principale"""
        print(f"Serveur epoll edge-triggered sur {self.host}:{self.port}")

        while True:
            events = self.epoll.poll(1.0)

            for fd, event in events:
                if fd == self.server.fileno():
                    # Accepter TOUS les clients en attente
                    self._accept_all_clients()

                elif event & select.EPOLLIN:
                    # Lire TOUTES les données disponibles
                    self._read_all_data(fd)

    def _accept_all_clients(self):
        """Accepte tous les clients (edge-triggered)"""
        while True:
            try:
                client, addr = self.server.accept()
                client.setblocking(False)

                fd = client.fileno()

                # Enregistrer en edge-triggered
                self.epoll.register(fd, select.EPOLLIN | select.EPOLLET)

                self.connections[fd] = {
                    'socket': client,
                    'addr': addr,
                    'buffer': b''
                }

                print(f"Client {addr} connecté")

            except BlockingIOError:
                # Plus de clients
                break

    def _read_all_data(self, fd):
        """Lit TOUTES les données (edge-triggered)"""
        sock = self.connections[fd]['socket']

        while True:
            try:
                chunk = sock.recv(4096)

                if not chunk:
                    # EOF
                    self._close_client(fd)
                    break

                # Accumuler
                self.connections[fd]['buffer'] += chunk

                # Traiter les messages complets
                self._process_buffer(fd)

            except BlockingIOError:
                # Tout lu
                break

            except Exception as e:
                print(f"Erreur: {e}")
                self._close_client(fd)
                break

    def _process_buffer(self, fd):
        """Traite le buffer"""
        buffer = self.connections[fd]['buffer']

        while b'\n' in buffer:
            line, buffer = buffer.split(b'\n', 1)

            # Echo
            sock = self.connections[fd]['socket']
            sock.send(b"Echo: " + line + b'\n')

        self.connections[fd]['buffer'] = buffer

    def _close_client(self, fd):
        """Ferme un client"""
        if fd in self.connections:
            self.epoll.unregister(fd)
            sock = self.connections[fd]['socket']
            sock.close()
            del self.connections[fd]
```

### Performance d'epoll

**Benchmark : 10 000 connexions simultanées**

```python
import time

# Test de performance
num_connections = 10000

# select() : O(n)
start = time.time()
for i in range(1000):
    readable, _, _ = select.select(sockets, [], [], 0)
duration_select = time.time() - start

# epoll() : O(1)
start = time.time()
for i in range(1000):
    events = epoll.poll(0)
duration_epoll = time.time() - start

print(f"select(): {duration_select:.3f}s")
print(f"epoll():  {duration_epoll:.3f}s")
print(f"Speedup:  {duration_select/duration_epoll:.1f}x")

# Résultats typiques :
# select(): 15.234s
# epoll():  0.456s
# Speedup:  33.4x
```

### Options epoll avancées

```python
import select

epoll = select.epoll()

# EPOLLIN : Données disponibles en lecture
# EPOLLOUT : Prêt pour écriture
# EPOLLERR : Erreur
# EPOLLHUP : Connexion fermée
# EPOLLET : Edge-triggered mode
# EPOLLONESHOT : Désactive auto après 1 événement

# Exemple : surveillance read + write + edge-triggered
epoll.register(
    fd,
    select.EPOLLIN | select.EPOLLOUT | select.EPOLLET
)

# Modifier les événements surveillés
epoll.modify(fd, select.EPOLLIN)  # Seulement lecture maintenant

# Retirer
epoll.unregister(fd)
```

## kqueue (BSD/macOS) - L'équivalent d'epoll

### Principe

kqueue est l'équivalent d'epoll sur les systèmes BSD et macOS.

```python
import select

# Créer une kqueue (BSD/macOS seulement)
kq = select.kqueue()

# Créer un kevent (événement à surveiller)
event = select.kevent(
    sock.fileno(),           # Identifiant
    filter=select.KQ_FILTER_READ,  # Type d'événement
    flags=select.KQ_EV_ADD   # Ajouter
)

# Enregistrer
kq.control([event], 0)

# Attendre des événements
events = kq.control(None, 1)  # Max 1 événement

for event in events:
    if event.filter == select.KQ_FILTER_READ:
        data = sock.recv(1024)
```

### Exemple avec kqueue

```python
import socket
import select

class KqueueEchoServer:
    """Serveur echo avec kqueue (macOS/BSD)"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.setblocking(False)
        self.server.bind((self.host, self.port))
        self.server.listen(5)

        # Créer kqueue
        self.kq = select.kqueue()

        # Enregistrer le serveur
        kevent = select.kevent(
            self.server.fileno(),
            filter=select.KQ_FILTER_READ,
            flags=select.KQ_EV_ADD
        )
        self.kq.control([kevent], 0)

        self.connections = {}

    def run(self):
        """Boucle principale"""
        print(f"Serveur kqueue sur {self.host}:{self.port}")

        while True:
            # Attendre des événements
            events = self.kq.control(None, 10, 1)  # Max 10 événements, timeout 1s

            for event in events:
                fd = event.ident

                if fd == self.server.fileno():
                    self._accept_client()

                elif event.filter == select.KQ_FILTER_READ:
                    self._handle_client(fd)

    def _accept_client(self):
        """Accepte un nouveau client"""
        client, addr = self.server.accept()
        client.setblocking(False)

        fd = client.fileno()

        # Enregistrer dans kqueue
        kevent = select.kevent(
            fd,
            filter=select.KQ_FILTER_READ,
            flags=select.KQ_EV_ADD
        )
        self.kq.control([kevent], 0)

        self.connections[fd] = client
        print(f"Client {addr} connecté")

    def _handle_client(self, fd):
        """Gère un client"""
        sock = self.connections[fd]

        try:
            data = sock.recv(1024)

            if not data:
                self._close_client(fd)
                return

            sock.send(b"Echo: " + data)

        except:
            self._close_client(fd)

    def _close_client(self, fd):
        """Ferme un client"""
        if fd in self.connections:
            sock = self.connections[fd]

            # Désenregistrer
            kevent = select.kevent(
                fd,
                filter=select.KQ_FILTER_READ,
                flags=select.KQ_EV_DELETE
            )
            self.kq.control([kevent], 0)

            del self.connections[fd]
            sock.close()
```

## Comparaison des mécanismes

### Tableau comparatif

| Aspect | select() | poll() | epoll() | kqueue |
|--------|----------|--------|---------|--------|
| **Complexité** | O(n) | O(n) | O(1) | O(1) |
| **Limite FDs** | 1024 | Illimité | Illimité | Illimité |
| **Performance** | Faible | Moyenne | Excellente | Excellente |
| **Portabilité** | Excellent | Bon | Linux only | BSD/macOS |
| **API** | Complexe | Simple | Moyenne | Complexe |
| **Edge-triggered** | ❌ | ❌ | ✅ | ✅ |
| **Cas d'usage** | < 100 FDs | < 1000 FDs | Serveurs HP | macOS |

### Performance comparée

```python
# Benchmark : Temps pour surveiller N sockets (1 active)

N = 100:
- select() : 0.05 ms
- poll()   : 0.04 ms
- epoll()  : 0.01 ms

N = 1000:
- select() : 0.5 ms
- poll()   : 0.4 ms
- epoll()  : 0.01 ms

N = 10000:
- select() : IMPOSSIBLE (limite FD_SETSIZE)
- poll()   : 5 ms
- epoll()  : 0.01 ms  ← CONSTANT !

N = 100000:
- select() : IMPOSSIBLE
- poll()   : 50 ms
- epoll()  : 0.01 ms  ← Toujours constant !
```

### Graphique de performance

```
Temps (ms)
  |
50|                                     poll()
  |                                    /
  |                                   /
10|                                  /
  |                              /
 5|          select()        /
  |              /       /
  |         /      /
 1|    /     /
  |  /  /
  | / /
 0|_________________epoll()____________
  0    1K    10K   100K  Connexions
```

## Cas d'usage réels

### 1. Nginx - Serveur web haute performance

```python
# Architecture Nginx (simplifié)
class NginxLikeServer:
    """Modèle similaire à Nginx"""

    def __init__(self, num_workers=4):
        self.num_workers = num_workers

    def start(self):
        """Fork plusieurs workers"""
        for i in range(self.num_workers):
            pid = os.fork()
            if pid == 0:  # Worker
                self.worker_process()
                exit(0)

    def worker_process(self):
        """Worker avec epoll"""
        epoll = select.epoll()

        # Socket listen partagée entre workers
        server = socket.socket()
        server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEPORT, 1)
        server.bind(('0.0.0.0', 80))
        server.listen(4096)

        epoll.register(server.fileno(), select.EPOLLIN | select.EPOLLET)

        connections = {}

        while True:
            # epoll.poll() : O(1)
            events = epoll.poll()

            for fd, event in events:
                if fd == server.fileno():
                    # Accept loop
                    while True:
                        try:
                            client, addr = server.accept()
                            client.setblocking(False)
                            epoll.register(client.fileno(), select.EPOLLIN | select.EPOLLET)
                            connections[client.fileno()] = client
                        except BlockingIOError:
                            break

                elif event & select.EPOLLIN:
                    # Read loop (edge-triggered)
                    sock = connections[fd]
                    request = b''

                    while True:
                        try:
                            chunk = sock.recv(4096)
                            if not chunk:
                                break
                            request += chunk
                        except BlockingIOError:
                            break

                    # Process HTTP request
                    response = self.process_http(request)
                    sock.sendall(response)

# Performance Nginx :
# - 4 workers = 4 cores CPU
# - Chaque worker : 10 000+ connexions
# - Total : 40 000+ connexions simultanées
# - Throughput : 100 000+ req/sec
```

### 2. Redis - Base de données in-memory

```python
# Redis utilise un event loop single-thread avec I/O multiplexing
class RedisLikeEventLoop:
    """Event loop similaire à Redis"""

    def __init__(self):
        # Choisir le meilleur mécanisme disponible
        if hasattr(select, 'epoll'):
            self.poller = select.epoll()
            self.poll_method = 'epoll'
        elif hasattr(select, 'kqueue'):
            self.poller = select.kqueue()
            self.poll_method = 'kqueue'
        elif hasattr(select, 'poll'):
            self.poller = select.poll()
            self.poll_method = 'poll'
        else:
            # Fallback sur select
            self.poll_method = 'select'

        print(f"Redis-like using {self.poll_method}")

        self.data = {}  # Key-value store
        self.connections = {}

    def run(self):
        """Event loop principal"""
        # Single thread, mais des milliers de clients
        while True:
            if self.poll_method == 'epoll':
                events = self.poller.poll(0.01)  # 10ms
            # ... autres méthodes

            for fd, event in events:
                self.handle_client(fd)

            # Tâches périodiques (expiration, persistence, etc.)
            self.background_tasks()

    def handle_client(self, fd):
        """Traite une commande client"""
        sock = self.connections[fd]

        # Parser commande Redis
        command = self.parse_redis_protocol(sock)

        # Exécuter (toujours < 1ms car in-memory)
        result = self.execute_command(command)

        # Répondre
        sock.sendall(result)

# Pourquoi single-thread pour Redis ?
# - Pas de locks nécessaires (single-threaded)
# - Toutes les opérations sont rapides (in-memory)
# - I/O multiplexing gère 10 000+ clients
# - Performance : 100 000+ ops/sec
```

### 3. Node.js - JavaScript backend

```javascript
// Node.js utilise libuv qui abstrait epoll/kqueue/IOCP

// Équivalent en pseudo-Python
class NodeJsEventLoop:
    """Modèle Node.js event loop"""

    def __init__(self):
        # libuv choisit automatiquement :
        # - epoll sur Linux
        # - kqueue sur macOS/BSD
        # - IOCP sur Windows
        self.poller = self.choose_best_poller()

        self.callbacks = {}
        self.timers = []

    def run(self):
        """Event loop principal"""
        while True:
            # 1. Timers (setTimeout, setInterval)
            self.process_timers()

            # 2. I/O callbacks
            events = self.poller.poll(timeout=0)
            for fd, event in events:
                callback = self.callbacks[fd]
                callback(event)

            # 3. setImmediate callbacks
            self.process_immediate()

            # 4. Close callbacks
            self.process_close()

# Exemple Node.js
const http = require('http');

const server = http.createServer((req, res) => {
    // Cette fonction s'exécute pour CHAQUE requête
    // Mais ne bloque JAMAIS l'event loop

    // I/O async (ne bloque pas)
    fs.readFile('data.json', (err, data) => {
        res.end(data);
    });

    // Cette fonction retourne IMMÉDIATEMENT
});

server.listen(3000);
// Single thread, 10 000+ req/sec
```

### 4. HAProxy - Load balancer

```python
# HAProxy utilise epoll pour gérer des centaines de milliers de connexions
class HAProxyLikeLoadBalancer:
    """Load balancer similaire à HAProxy"""

    def __init__(self, backends):
        self.backends = backends
        self.epoll = select.epoll()

        # Socket frontend (clients)
        self.frontend = self.create_frontend()

        # Pools de connexions backend
        self.backend_pools = {
            backend: ConnectionPool(backend)
            for backend in backends
        }

        self.connections = {}

    def run(self):
        """Event loop"""
        while True:
            events = self.epoll.poll()

            for fd, event in events:
                if fd == self.frontend.fileno():
                    # Nouveau client
                    client, addr = self.frontend.accept()
                    client.setblocking(False)

                    # Choisir un backend (round-robin, least-conn, etc.)
                    backend = self.choose_backend()

                    # Connecter au backend
                    backend_conn = self.backend_pools[backend].get()

                    # Associer client <-> backend
                    self.connections[client.fileno()] = {
                        'client': client,
                        'backend': backend_conn,
                        'direction': 'client'
                    }

                    # Surveiller les deux
                    self.epoll.register(client.fileno(), select.EPOLLIN | select.EPOLLET)
                    self.epoll.register(backend_conn.fileno(), select.EPOLLIN | select.EPOLLET)

                elif event & select.EPOLLIN:
                    # Proxy bidirectionnel
                    self.proxy_data(fd)

    def proxy_data(self, fd):
        """Transfert client <-> backend"""
        conn = self.connections[fd]

        if conn['direction'] == 'client':
            # Client → Backend
            data = conn['client'].recv(8192)
            if data:
                conn['backend'].sendall(data)
        else:
            # Backend → Client
            data = conn['backend'].recv(8192)
            if data:
                conn['client'].sendall(data)

# HAProxy performance :
# - 1 processus par core CPU
# - 100 000+ connexions simultanées par processus
# - Latency < 1ms
# - Throughput : 10 Gbps+
```

## Abstraction cross-platform : selectors

```python
import selectors
import socket

# Module selectors : choisit automatiquement le meilleur
# - epoll sur Linux
# - kqueue sur BSD/macOS
# - poll sur autres Unix
# - select en fallback

class UniversalServer:
    """Serveur cross-platform avec selectors"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

        # Sélecteur automatique
        self.selector = selectors.DefaultSelector()

        # Socket serveur
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.setblocking(False)
        self.server.bind((self.host, self.port))
        self.server.listen(100)

        # Enregistrer avec callback
        self.selector.register(
            self.server,
            selectors.EVENT_READ,
            data={'type': 'server'}
        )

    def run(self):
        """Boucle principale"""
        print(f"Serveur universel sur {self.host}:{self.port}")
        print(f"Utilise: {self.selector.__class__.__name__}")

        while True:
            # select() abstrait
            events = self.selector.select(timeout=1.0)

            for key, mask in events:
                if key.data['type'] == 'server':
                    self._accept_client()
                elif mask & selectors.EVENT_READ:
                    self._handle_client(key.fileobj, key.data)

    def _accept_client(self):
        """Accepte un client"""
        client, addr = self.server.accept()
        client.setblocking(False)

        # Enregistrer avec métadonnées
        self.selector.register(
            client,
            selectors.EVENT_READ,
            data={'type': 'client', 'addr': addr, 'buffer': b''}
        )

        print(f"Client {addr} connecté")

    def _handle_client(self, sock, data):
        """Gère un client"""
        try:
            recv_data = sock.recv(1024)

            if not recv_data:
                # Déconnexion
                print(f"Client {data['addr']} déconnecté")
                self.selector.unregister(sock)
                sock.close()
                return

            # Echo
            sock.send(b"Echo: " + recv_data)

        except Exception as e:
            print(f"Erreur: {e}")
            self.selector.unregister(sock)
            sock.close()

if __name__ == "__main__":
    server = UniversalServer()
    server.run()
```

**Avantages de selectors** :

- ✅ **Cross-platform** : Fonctionne partout
- ✅ **Optimal** : Choisit le meilleur mécanisme
- ✅ **API simple** : Plus facile que epoll/kqueue direct
- ✅ **Métadonnées** : Attache des données aux sockets

```python
import selectors

# Sur Linux
sel = selectors.DefaultSelector()
print(type(sel))  # <class 'selectors.EpollSelector'>

# Sur macOS
sel = selectors.DefaultSelector()
print(type(sel))  # <class 'selectors.KqueueSelector'>

# Sur Windows
sel = selectors.DefaultSelector()
print(type(sel))  # <class 'selectors.SelectSelector'>
```

## Best practices

### 1. Toujours utiliser non-bloquant

```python
# BON
sock.setblocking(False)
epoll.register(sock.fileno(), select.EPOLLIN)

# MAUVAIS : socket bloquante avec epoll
sock.setblocking(True)  # Bloquera quand même !
epoll.register(sock.fileno(), select.EPOLLIN)
```

### 2. Gérer TOUS les cas edge-triggered

```python
# Edge-triggered : lire TOUT
while True:
    try:
        chunk = sock.recv(4096)
        if not chunk:
            break
        buffer += chunk
    except BlockingIOError:
        break  # CRUCIAL !
```

### 3. Surveiller les erreurs

```python
# Toujours surveiller EPOLLERR et EPOLLHUP
epoll.register(fd, select.EPOLLIN | select.EPOLLERR | select.EPOLLHUP)

# Traiter
if event & select.EPOLLHUP:
    # Connexion fermée
    close_connection(fd)
elif event & select.EPOLLERR:
    # Erreur
    handle_error(fd)
```

### 4. Limiter le nombre d'événements traités

```python
# Éviter le starvation
MAX_EVENTS_PER_LOOP = 100

while True:
    events = epoll.poll(timeout=0.1)

    # Limiter le traitement
    for fd, event in events[:MAX_EVENTS_PER_LOOP]:
        handle_event(fd, event)

    # Les autres seront traités au prochain tour
```

### 5. Utiliser selectors pour portabilité

```python
# Au lieu d'epoll direct
import selectors

# Fonctionne partout, optimal partout
selector = selectors.DefaultSelector()
```

## Récapitulatif

### Choix du mécanisme

| Situation | Recommandation |
|-----------|----------------|
| **Linux + Performance** | epoll() |
| **macOS/BSD + Performance** | kqueue |
| **Cross-platform** | selectors.DefaultSelector() |
| **< 100 connexions** | select() acceptable |
| **Legacy Windows** | select() |
| **Prototype rapide** | select() ou selectors |

### Points clés

🎯 **select()** :
- O(n), limité à 1024 FDs
- Portable partout
- OK pour peu de connexions

🎯 **poll()** :
- O(n), pas de limite FDs
- Meilleur que select()
- Pas sur Windows

🎯 **epoll()** :
- O(1), illimité
- Haute performance
- Linux uniquement

🎯 **kqueue** :
- O(1), illimité
- Équivalent epoll
- BSD/macOS uniquement

🎯 **selectors** :
- Abstraction universelle
- Choisit le meilleur
- API Python propre

### Performances

```
1 connexion active sur 10 000 :

select() : 5 ms    (vérifie toutes)
poll()   : 5 ms    (vérifie toutes)
epoll()  : 0.01 ms (sait laquelle)

→ epoll() est 500x plus rapide !
```

## Prochaines étapes

Maintenant que vous maîtrisez le multiplexage I/O, nous allons explorer :

- **Section 8.7** : Programmation asynchrone et event-driven (asyncio, coroutines)
- **Section 8.8** : Résilience et fiabilité applicative (timeouts, retry, circuit breakers)
- **Section 8.9** : Communication temps réel (WebSockets, SSE, long-polling)

---


⏭️ [Programmation asynchrone et event-driven](/08-programmation-reseau/07-programmation-asynchrone.md)
