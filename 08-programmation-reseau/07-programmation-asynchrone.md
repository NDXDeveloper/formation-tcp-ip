🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.7 Programmation asynchrone et event-driven

## Introduction

La **programmation asynchrone** représente un changement de paradigme fondamental dans la façon d'écrire du code réseau. Au lieu d'attendre passivement qu'une opération I/O se termine (bloquant), ou de gérer des milliers de threads (complexe), la programmation asynchrone permet d'écrire du code **concurrent** qui reste **séquentiel** en apparence.

```python
# Code SYNCHRONE (bloquant)
def fetch_user(user_id):
    response = requests.get(f'/api/users/{user_id}')  # Bloque ~100ms
    return response.json()

def get_user_data(user_id):
    user = fetch_user(user_id)           # Attend 100ms
    posts = fetch_posts(user_id)         # Attend 100ms
    comments = fetch_comments(user_id)   # Attend 100ms
    return merge(user, posts, comments)
# Total : 300ms (séquentiel)

# Code ASYNCHRONE (non-bloquant)
async def fetch_user(user_id):
    response = await http.get(f'/api/users/{user_id}')  # Ne bloque pas
    return response.json()

async def get_user_data(user_id):
    # Les 3 requêtes en PARALLÈLE !
    user, posts, comments = await asyncio.gather(
        fetch_user(user_id),
        fetch_posts(user_id),
        fetch_comments(user_id)
    )
    return merge(user, posts, comments)
# Total : 100ms (parallèle)
```

Cette section explore comment la programmation asynchrone fonctionne, quand l'utiliser, et comment construire des applications réseau performantes avec ce paradigme.

## Concepts fondamentaux

### Le problème du blocage

```python
# Serveur HTTP synchrone (un seul thread)
def handle_request(request):
    # 1. Requête base de données (50ms)
    user = db.query("SELECT * FROM users WHERE id=?", request.user_id)

    # 2. Appel API externe (100ms)
    profile = requests.get(f"https://api.example.com/profiles/{user.id}")

    # 3. Calcul (10ms)
    result = compute(user, profile)

    return result

# Pendant les 150ms d'I/O, le thread ne fait RIEN
# Il attend passivement
```

**Visualisation** :

```
Thread 1 :
[0-50ms]    DB query ████████████ (BLOQUÉ)
[50-150ms]  API call ████████████████████████ (BLOQUÉ)
[150-160ms] Compute ██ (CPU)

Pendant 150ms sur 160ms, le thread est bloqué !
→ CPU inutilisé
→ Ressources gaspillées
```

### La solution asynchrone

```python
# Version asynchrone
async def handle_request(request):
    # 1. Lancer la requête DB (non-bloquant)
    user_task = asyncio.create_task(db.query("SELECT * FROM users WHERE id=?", request.user_id))

    # 2. Pendant ce temps, on peut faire autre chose
    # L'event loop peut traiter d'autres requêtes

    # 3. Attendre le résultat quand on en a besoin
    user = await user_task

    # 4. Appel API (non-bloquant)
    profile = await http.get(f"https://api.example.com/profiles/{user.id}")

    # 5. Calcul
    result = compute(user, profile)

    return result
```

**Visualisation** :

```
Event Loop :
[0-50ms]    Requête 1 : DB query (lancée)
            Requête 2 : commence pendant que Req1 attend
            Requête 3 : commence pendant que Req1 & Req2 attendent
[50ms]      Requête 1 : DB répond, continue
[50-150ms]  Requête 1 : API call (lancée)
            Requêtes 2, 3, 4, 5... peuvent progresser
[150ms]     Requête 1 : API répond, termine

→ UN SEUL THREAD gère PLUSIEURS requêtes simultanément
→ CPU toujours occupé
→ Pas d'attente passive
```

### Coroutines : Les fonctions async

Une **coroutine** est une fonction qui peut être suspendue et reprise.

```python
import asyncio

# Fonction normale (exécution continue)
def synchronous_function():
    result = do_something()  # S'exécute jusqu'au bout
    return result

# Coroutine (peut être suspendue)
async def asynchronous_function():
    result = await do_something_async()  # Peut être suspendue ici
    return result

# Appel
# sync :
value = synchronous_function()  # Exécution immédiate

# async :
coro = asynchronous_function()  # Retourne une coroutine (pas encore exécutée)
value = await coro  # OU asyncio.run(coro)
```

**Points clés** :

```python
# 1. async def crée une coroutine
async def my_coroutine():
    return "hello"

# 2. Appeler une coroutine ne l'exécute PAS
coro = my_coroutine()
print(type(coro))  # <class 'coroutine'>
# Elle n'a pas encore run !

# 3. Il faut await ou run()
result = await my_coroutine()  # Dans un contexte async

# OU
result = asyncio.run(my_coroutine())  # À la racine
```

### await : Le point de suspension

```python
import asyncio

async def fetch_data():
    print("1. Début fetch_data")

    # await SUSPEND la coroutine
    await asyncio.sleep(1)  # Point de suspension

    print("2. Après sleep")
    return "data"

async def main():
    print("A. Avant fetch")

    data = await fetch_data()  # Attend ici

    print("B. Après fetch")
    print(f"Data: {data}")

asyncio.run(main())

# Output :
# A. Avant fetch
# 1. Début fetch_data
# [pause de 1 seconde - event loop traite autres tâches]
# 2. Après sleep
# B. Après fetch
# Data: data
```

**Que se passe-t-il lors de await ?**

```
1. Coroutine s'exécute jusqu'à await
2. await SUSPEND la coroutine
3. Event loop prend le contrôle
4. Event loop peut exécuter d'AUTRES coroutines
5. Quand l'opération async est prête, event loop REPREND la coroutine
6. La coroutine continue après le await
```

## L'Event Loop

### Qu'est-ce qu'un event loop ?

L'**event loop** est le cœur de la programmation asynchrone. C'est une boucle infinie qui :

1. Vérifie quelles coroutines sont prêtes
2. Exécute les coroutines prêtes
3. Gère les événements I/O (sockets, timers, etc.)
4. Répète

```python
# Pseudo-code simplifié d'un event loop
class EventLoop:
    def __init__(self):
        self.ready_queue = []      # Coroutines prêtes
        self.waiting = {}          # Coroutines en attente
        self.selector = select.epoll()  # Multiplexage I/O

    def run_forever(self):
        while True:
            # 1. Traiter les coroutines prêtes
            while self.ready_queue:
                coro = self.ready_queue.pop(0)
                self.step(coro)

            # 2. Attendre des événements I/O
            events = self.selector.poll(timeout=0.01)

            # 3. Réveiller les coroutines dont l'I/O est prête
            for fd, event in events:
                coro = self.waiting.pop(fd)
                self.ready_queue.append(coro)

    def step(self, coro):
        """Exécute une coroutine jusqu'au prochain await"""
        try:
            # Avancer la coroutine
            future = coro.send(None)

            # Si elle attend, la mettre en attente
            if isinstance(future, IOFuture):
                self.waiting[future.fd] = coro
            else:
                # Prête à continuer
                self.ready_queue.append(coro)

        except StopIteration:
            # Coroutine terminée
            pass
```

### Exemple concret d'event loop

```python
import asyncio

async def task1():
    print("Task1: Début")
    await asyncio.sleep(0.1)
    print("Task1: Milieu")
    await asyncio.sleep(0.1)
    print("Task1: Fin")

async def task2():
    print("Task2: Début")
    await asyncio.sleep(0.15)
    print("Task2: Fin")

async def main():
    # Lancer les deux tâches en parallèle
    await asyncio.gather(task1(), task2())

asyncio.run(main())

# Output :
# Task1: Début
# Task2: Début
# Task1: Milieu      (après 0.1s)
# Task2: Fin         (après 0.15s)
# Task1: Fin         (après 0.2s)
```

**Timeline de l'event loop** :

```
t=0ms:
  Event loop démarre
  Task1 s'exécute jusqu'à await sleep(0.1)
  Task2 s'exécute jusqu'à await sleep(0.15)
  Event loop: Task1 prête à 100ms, Task2 prête à 150ms

t=100ms:
  Event loop réveille Task1
  Task1 continue jusqu'à await sleep(0.1)
  Event loop: Task1 prête à 200ms

t=150ms:
  Event loop réveille Task2
  Task2 termine

t=200ms:
  Event loop réveille Task1
  Task1 termine
```

## asyncio en Python

### Bases d'asyncio

```python
import asyncio

# 1. Fonction async basique
async def say_hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")

# 2. Exécuter
asyncio.run(say_hello())

# 3. Créer des tâches
async def main():
    # Créer une tâche (s'exécute en arrière-plan)
    task = asyncio.create_task(say_hello())

    # Faire autre chose
    await asyncio.sleep(0.5)

    # Attendre la fin de la tâche
    await task

asyncio.run(main())
```

### asyncio.gather() - Exécution parallèle

```python
import asyncio
import time

async def fetch_data(id, delay):
    """Simule une requête réseau"""
    print(f"Fetching {id}...")
    await asyncio.sleep(delay)
    print(f"Fetched {id}")
    return f"data_{id}"

async def sequential():
    """Exécution séquentielle"""
    start = time.time()

    data1 = await fetch_data(1, 1.0)
    data2 = await fetch_data(2, 1.0)
    data3 = await fetch_data(3, 1.0)

    duration = time.time() - start
    print(f"Sequential: {duration:.2f}s")  # ~3s

async def parallel():
    """Exécution parallèle avec gather"""
    start = time.time()

    # Toutes en parallèle !
    results = await asyncio.gather(
        fetch_data(1, 1.0),
        fetch_data(2, 1.0),
        fetch_data(3, 1.0)
    )

    duration = time.time() - start
    print(f"Parallel: {duration:.2f}s")  # ~1s
    print(f"Results: {results}")

# Test
asyncio.run(sequential())  # 3 secondes
asyncio.run(parallel())    # 1 seconde
```

**Cas d'usage réel : agrégation d'APIs**

```python
import asyncio
import aiohttp

async def fetch_user_data(user_id):
    """Récupère les données d'un utilisateur depuis plusieurs APIs"""
    async with aiohttp.ClientSession() as session:
        # Lancer toutes les requêtes en parallèle
        tasks = [
            fetch_profile(session, user_id),
            fetch_posts(session, user_id),
            fetch_followers(session, user_id),
            fetch_stats(session, user_id)
        ]

        # Attendre toutes les réponses
        profile, posts, followers, stats = await asyncio.gather(*tasks)

        return {
            'profile': profile,
            'posts': posts,
            'followers': followers,
            'stats': stats
        }

async def fetch_profile(session, user_id):
    async with session.get(f'https://api.example.com/users/{user_id}') as resp:
        return await resp.json()

async def fetch_posts(session, user_id):
    async with session.get(f'https://api.example.com/users/{user_id}/posts') as resp:
        return await resp.json()

async def fetch_followers(session, user_id):
    async with session.get(f'https://api.example.com/users/{user_id}/followers') as resp:
        return await resp.json()

async def fetch_stats(session, user_id):
    async with session.get(f'https://api.example.com/users/{user_id}/stats') as resp:
        return await resp.json()

# Utilisation
async def main():
    data = await fetch_user_data(123)
    print(data)

asyncio.run(main())

# Au lieu de 4 × 100ms = 400ms (séquentiel)
# On obtient 1 × 100ms = 100ms (parallèle)
```

### asyncio.create_task() - Tâches en arrière-plan

```python
import asyncio

async def background_task():
    """Tâche qui tourne en arrière-plan"""
    while True:
        print("Background task running...")
        await asyncio.sleep(2)

async def main_task():
    """Tâche principale"""
    # Lancer la tâche en arrière-plan
    bg_task = asyncio.create_task(background_task())

    # Faire le travail principal
    for i in range(5):
        print(f"Main task: {i}")
        await asyncio.sleep(1)

    # Annuler la tâche d'arrière-plan
    bg_task.cancel()

    try:
        await bg_task
    except asyncio.CancelledError:
        print("Background task cancelled")

asyncio.run(main_task())

# Output :
# Background task running...
# Main task: 0
# Main task: 1
# Background task running...
# Main task: 2
# Main task: 3
# Background task running...
# Main task: 4
# Background task cancelled
```

**Cas d'usage : Heartbeat**

```python
import asyncio

class WebSocketClient:
    """Client WebSocket avec heartbeat automatique"""

    def __init__(self, url):
        self.url = url
        self.ws = None
        self.heartbeat_task = None

    async def connect(self):
        """Connecte et démarre le heartbeat"""
        self.ws = await websockets.connect(self.url)

        # Lancer le heartbeat en arrière-plan
        self.heartbeat_task = asyncio.create_task(self._heartbeat())

    async def _heartbeat(self):
        """Envoie un ping toutes les 30 secondes"""
        while True:
            try:
                await asyncio.sleep(30)
                await self.ws.ping()
                print("Heartbeat sent")
            except asyncio.CancelledError:
                break
            except Exception as e:
                print(f"Heartbeat error: {e}")
                break

    async def send(self, message):
        """Envoie un message"""
        await self.ws.send(message)

    async def receive(self):
        """Reçoit un message"""
        return await self.ws.recv()

    async def close(self):
        """Ferme la connexion et arrête le heartbeat"""
        if self.heartbeat_task:
            self.heartbeat_task.cancel()
            try:
                await self.heartbeat_task
            except asyncio.CancelledError:
                pass

        if self.ws:
            await self.ws.close()

# Utilisation
async def main():
    client = WebSocketClient('ws://example.com')
    await client.connect()

    # Le heartbeat tourne en arrière-plan
    await client.send("Hello")
    response = await client.receive()

    await client.close()
```

### Timeouts avec asyncio

```python
import asyncio

async def slow_operation():
    """Opération qui prend du temps"""
    await asyncio.sleep(10)
    return "Done"

async def with_timeout():
    """Opération avec timeout"""
    try:
        # Timeout de 3 secondes
        result = await asyncio.wait_for(slow_operation(), timeout=3.0)
        print(f"Result: {result}")

    except asyncio.TimeoutError:
        print("Operation timed out!")

asyncio.run(with_timeout())
# Output : Operation timed out! (après 3s)
```

**Cas d'usage : Requête HTTP avec timeout**

```python
import asyncio
import aiohttp

async def fetch_with_timeout(url, timeout=5.0):
    """Requête HTTP avec timeout"""
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=timeout)) as resp:
                return await resp.text()

    except asyncio.TimeoutError:
        print(f"Request to {url} timed out")
        return None

    except Exception as e:
        print(f"Error: {e}")
        return None

# Utilisation
async def main():
    # Ces requêtes s'exécutent en parallèle
    results = await asyncio.gather(
        fetch_with_timeout('https://fast-api.com', timeout=2.0),
        fetch_with_timeout('https://slow-api.com', timeout=2.0),
        fetch_with_timeout('https://medium-api.com', timeout=2.0)
    )

    for i, result in enumerate(results):
        if result:
            print(f"API {i+1}: Success ({len(result)} bytes)")
        else:
            print(f"API {i+1}: Failed")

asyncio.run(main())
```

## Serveur asynchrone complet

### Serveur TCP echo asynchrone

```python
import asyncio

class AsyncTCPServer:
    """Serveur TCP asynchrone"""

    def __init__(self, host='0.0.0.0', port=8888):
        self.host = host
        self.port = port
        self.clients = set()

    async def handle_client(self, reader, writer):
        """Gère un client (coroutine par client)"""
        addr = writer.get_extra_info('peername')
        print(f"[+] Client connecté: {addr}")

        self.clients.add(writer)

        try:
            while True:
                # Lire des données (non-bloquant)
                data = await reader.read(1024)

                if not data:
                    # Client déconnecté
                    break

                message = data.decode('utf-8')
                print(f"[{addr}] Reçu: {message.strip()}")

                # Echo (non-bloquant)
                writer.write(b"Echo: " + data)
                await writer.drain()

        except Exception as e:
            print(f"Erreur avec {addr}: {e}")

        finally:
            print(f"[-] Client déconnecté: {addr}")
            self.clients.remove(writer)
            writer.close()
            await writer.wait_closed()

    async def start(self):
        """Démarre le serveur"""
        server = await asyncio.start_server(
            self.handle_client,
            self.host,
            self.port
        )

        addr = server.sockets[0].getsockname()
        print(f"Serveur async démarré sur {addr}")

        async with server:
            await server.serve_forever()

# Lancement
async def main():
    server = AsyncTCPServer()
    await server.start()

if __name__ == "__main__":
    asyncio.run(main())
```

**Points clés** :

```python
# 1. asyncio.start_server() gère l'accept() automatiquement
server = await asyncio.start_server(handler, host, port)

# 2. Une coroutine par client (pas de thread !)
async def handle_client(reader, writer):
    # Chaque client a sa propre coroutine
    pass

# 3. reader/writer abstractions
data = await reader.read(1024)    # Lecture async
writer.write(data)                # Écriture bufferisée
await writer.drain()              # Flush async
```

### Serveur de chat asynchrone

```python
import asyncio

class AsyncChatServer:
    """Serveur de chat asynchrone multi-clients"""

    def __init__(self, host='0.0.0.0', port=9999):
        self.host = host
        self.port = port
        self.clients = {}  # {writer: {'addr': ..., 'name': ...}}

    async def handle_client(self, reader, writer):
        """Gère un client"""
        addr = writer.get_extra_info('peername')

        # Demander le pseudo
        writer.write(b"Entrez votre pseudo: ")
        await writer.drain()

        name_data = await reader.readline()
        name = name_data.decode('utf-8').strip()

        # Enregistrer le client
        self.clients[writer] = {'addr': addr, 'name': name}

        # Message de bienvenue
        welcome = f"Bienvenue {name}! {len(self.clients)} utilisateurs connectés.\n"
        writer.write(welcome.encode('utf-8'))
        await writer.drain()

        # Annoncer l'arrivée
        await self.broadcast(f"{name} a rejoint le chat!\n", exclude=writer)

        try:
            while True:
                # Lire un message
                data = await reader.readline()

                if not data:
                    break

                message = data.decode('utf-8').strip()

                if message:
                    # Diffuser le message
                    formatted = f"[{name}] {message}\n"
                    await self.broadcast(formatted, exclude=writer)

        except Exception as e:
            print(f"Erreur avec {name}: {e}")

        finally:
            # Retirer le client
            del self.clients[writer]
            writer.close()
            await writer.wait_closed()

            # Annoncer le départ
            await self.broadcast(f"{name} a quitté le chat.\n")
            print(f"{name} ({addr}) déconnecté")

    async def broadcast(self, message, exclude=None):
        """Diffuse un message à tous les clients"""
        # Liste des tâches d'envoi
        tasks = []

        for writer in self.clients:
            if writer != exclude:
                # Créer une tâche pour chaque client
                tasks.append(self.send_to_client(writer, message))

        # Envoyer à tous en parallèle
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)

    async def send_to_client(self, writer, message):
        """Envoie un message à un client"""
        try:
            writer.write(message.encode('utf-8'))
            await writer.drain()
        except Exception as e:
            # Erreur d'envoi (client déconnecté ?)
            print(f"Erreur d'envoi: {e}")

    async def start(self):
        """Démarre le serveur"""
        server = await asyncio.start_server(
            self.handle_client,
            self.host,
            self.port
        )

        print(f"Serveur de chat sur {self.host}:{self.port}")

        async with server:
            await server.serve_forever()

# Lancement
if __name__ == "__main__":
    server = AsyncChatServer()
    asyncio.run(server.start())
```

**Test** :

```bash
# Terminal 1 : Serveur
python async_chat_server.py

# Terminal 2 : Client 1
telnet localhost 9999
Entrez votre pseudo: Alice
Bienvenue Alice! 1 utilisateurs connectés.
Hello everyone!

# Terminal 3 : Client 2
telnet localhost 9999
Entrez votre pseudo: Bob
Bienvenue Bob! 2 utilisateurs connectés.
Alice a rejoint le chat!
[Alice] Hello everyone!
Hi Alice!

# Dans Terminal 2 (Alice voit) :
Bob a rejoint le chat!
[Bob] Hi Alice!
```

## Gestion des erreurs en async

### try/except avec async

```python
import asyncio

async def risky_operation():
    """Opération qui peut échouer"""
    await asyncio.sleep(0.1)
    raise ValueError("Something went wrong!")

async def safe_caller():
    """Appel sécurisé"""
    try:
        result = await risky_operation()
    except ValueError as e:
        print(f"Caught error: {e}")
        result = None

    return result

asyncio.run(safe_caller())
```

### Gestion d'erreurs avec gather()

```python
import asyncio

async def task_success():
    await asyncio.sleep(0.1)
    return "success"

async def task_failure():
    await asyncio.sleep(0.1)
    raise ValueError("failed")

async def handle_multiple_tasks():
    """Gère plusieurs tâches avec erreurs potentielles"""

    # Option 1 : Propager la première erreur (défaut)
    try:
        results = await asyncio.gather(
            task_success(),
            task_failure(),
            task_success()
        )
    except ValueError as e:
        print(f"Une tâche a échoué: {e}")

    # Option 2 : Retourner les exceptions (ne pas lever)
    results = await asyncio.gather(
        task_success(),
        task_failure(),
        task_success(),
        return_exceptions=True  # Important !
    )

    # Traiter les résultats et erreurs
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"Tâche {i} a échoué: {result}")
        else:
            print(f"Tâche {i} a réussi: {result}")

asyncio.run(handle_multiple_tasks())

# Output :
# Une tâche a échoué: failed
# Tâche 0 a réussi: success
# Tâche 1 a échoué: failed
# Tâche 2 a réussi: success
```

**Cas d'usage : Agrégation d'APIs avec fallback**

```python
import asyncio
import aiohttp

async def fetch_with_fallback(urls):
    """Essaie plusieurs URLs, retourne la première qui réussit"""

    async def try_url(url):
        try:
            async with aiohttp.ClientSession() as session:
                async with session.get(url, timeout=aiohttp.ClientTimeout(total=2)) as resp:
                    return await resp.json()
        except Exception as e:
            return e  # Retourner l'exception

    # Essayer toutes les URLs en parallèle
    results = await asyncio.gather(
        *[try_url(url) for url in urls],
        return_exceptions=True
    )

    # Retourner le premier succès
    for result in results:
        if not isinstance(result, Exception):
            return result

    # Toutes ont échoué
    raise Exception("All APIs failed")

# Utilisation
async def main():
    urls = [
        'https://api1.example.com/data',
        'https://api2.example.com/data',
        'https://api3.example.com/data'
    ]

    try:
        data = await fetch_with_fallback(urls)
        print(f"Success: {data}")
    except Exception as e:
        print(f"All failed: {e}")
```

### Annulation de tâches

```python
import asyncio

async def long_running_task():
    """Tâche longue qui peut être annulée"""
    try:
        for i in range(100):
            print(f"Working... {i}")
            await asyncio.sleep(0.5)
    except asyncio.CancelledError:
        print("Task was cancelled!")
        # Cleanup
        raise  # Important : propager l'annulation

async def main():
    # Créer la tâche
    task = asyncio.create_task(long_running_task())

    # Laisser tourner 2 secondes
    await asyncio.sleep(2)

    # Annuler
    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print("Confirmed: task cancelled")

asyncio.run(main())
```

**Cas d'usage : Timeout avec cleanup**

```python
import asyncio

async def download_file(url, filepath):
    """Télécharge un fichier avec possibilité d'annulation"""
    file = None
    try:
        file = open(filepath, 'wb')

        async with aiohttp.ClientSession() as session:
            async with session.get(url) as resp:
                async for chunk in resp.content.iter_chunked(8192):
                    file.write(chunk)
                    # Permet l'annulation entre les chunks
                    await asyncio.sleep(0)

        print(f"Download complete: {filepath}")

    except asyncio.CancelledError:
        print(f"Download cancelled: {filepath}")
        # Cleanup : supprimer le fichier partiel
        if file:
            file.close()
            os.remove(filepath)
        raise

    finally:
        if file:
            file.close()

async def download_with_timeout(url, filepath, timeout=30.0):
    """Télécharge avec timeout"""
    try:
        await asyncio.wait_for(
            download_file(url, filepath),
            timeout=timeout
        )
    except asyncio.TimeoutError:
        print(f"Download timeout: {url}")
```

## Async vs Sync vs Threads : Comparaison

### Benchmark : 1000 requêtes HTTP

```python
import time
import asyncio
import aiohttp
import requests
from concurrent.futures import ThreadPoolExecutor

# 1. SYNCHRONE (séquentiel)
def sync_fetch(url):
    response = requests.get(url, timeout=5)
    return len(response.content)

def benchmark_sync(urls):
    start = time.time()

    results = []
    for url in urls:
        result = sync_fetch(url)
        results.append(result)

    duration = time.time() - start
    print(f"Sync: {duration:.2f}s")
    return results

# 2. THREADS
def benchmark_threads(urls):
    start = time.time()

    with ThreadPoolExecutor(max_workers=50) as executor:
        results = list(executor.map(sync_fetch, urls))

    duration = time.time() - start
    print(f"Threads: {duration:.2f}s")
    return results

# 3. ASYNC
async def async_fetch(session, url):
    async with session.get(url, timeout=aiohttp.ClientTimeout(total=5)) as resp:
        content = await resp.read()
        return len(content)

async def benchmark_async(urls):
    start = time.time()

    async with aiohttp.ClientSession() as session:
        tasks = [async_fetch(session, url) for url in urls]
        results = await asyncio.gather(*tasks)

    duration = time.time() - start
    print(f"Async: {duration:.2f}s")
    return results

# Test
urls = ['http://httpbin.org/delay/0.1'] * 100

# Sync : ~10s (séquentiel, 100 × 0.1s)
benchmark_sync(urls)

# Threads : ~2s (parallèle, mais overhead threads)
benchmark_threads(urls)

# Async : ~0.5s (parallèle, pas d'overhead)
asyncio.run(benchmark_async(urls))
```

**Résultats typiques** :

```
100 requêtes de 100ms chacune :

Synchrone : 10.0s
  - Séquentiel : 100 × 0.1s
  - 1 CPU core utilisé

Threads (50 workers) : 2.5s
  - Parallèle : 100 / 50 × 0.1s
  - Overhead : context switching, mémoire
  - 50 threads × 8 MB = 400 MB RAM

Async : 0.5s
  - Parallèle : toutes en même temps
  - Pas d'overhead
  - 1 thread, ~10 MB RAM
```

### Tableau comparatif

| Aspect | Synchrone | Threads | Async |
|--------|-----------|---------|-------|
| **Simplicité code** | ✅ Très simple | ⚠️ Moyen | ❌ Plus complexe |
| **Scalabilité** | ❌ 1 requête/fois | ⚠️ ~1000 threads max | ✅ 10000+ |
| **RAM** | ✅ Minimal | ❌ ~8 MB/thread | ✅ Minimal |
| **CPU usage** | ⚠️ 1 core max | ✅ Multi-core | ⚠️ 1 core |
| **I/O-bound** | ❌ Très lent | ⚠️ OK | ✅ Excellent |
| **CPU-bound** | ✅ OK | ✅ Excellent | ❌ Bloque event loop |
| **Debugging** | ✅ Facile | ❌ Race conditions | ⚠️ Moyen |

### Quand utiliser quoi ?

#### Synchrone

✅ **Scripts simples** : One-shot, prototypes

```python
# Script simple de scraping
def scrape_page(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.content)
    return soup.find_all('a')

# Acceptable pour quelques pages
for url in urls:
    links = scrape_page(url)
```

✅ **Peu de requêtes** : < 10 appels réseau

✅ **Code legacy** : Intégration avec bibliothèques bloquantes

#### Threads

✅ **CPU-bound + I/O** : Calculs lourds + réseau

```python
def process_image(url):
    # 1. Download (I/O-bound)
    response = requests.get(url)

    # 2. Process (CPU-bound)
    image = Image.open(BytesIO(response.content))
    processed = apply_filters(image)  # CPU intensif

    return processed

# Threads utilisent plusieurs cores pour CPU-bound
with ThreadPoolExecutor(max_workers=8) as executor:
    results = executor.map(process_image, urls)
```

✅ **Bibliothèques bloquantes** : Pas de version async disponible

✅ **< 1000 opérations parallèles**

#### Async

✅ **I/O-bound pur** : Appels réseau, base de données

```python
async def fetch_all_data():
    # Des milliers de requêtes réseau
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in 10000_urls]
        return await asyncio.gather(*tasks)
```

✅ **Serveurs haute performance** : Web servers, APIs

✅ **WebSockets, temps réel** : Beaucoup de connexions longues

✅ **Microservices** : Communication service-to-service

## Cas d'usage réels détaillés

### 1. Web Scraper asynchrone

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup

class AsyncWebScraper:
    """Web scraper asynchrone haute performance"""

    def __init__(self, max_concurrent=100):
        self.max_concurrent = max_concurrent
        self.semaphore = asyncio.Semaphore(max_concurrent)

    async def fetch_page(self, session, url):
        """Récupère une page"""
        async with self.semaphore:  # Limite la concurrence
            try:
                async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as resp:
                    if resp.status == 200:
                        html = await resp.text()
                        return url, html
                    else:
                        return url, None
            except Exception as e:
                print(f"Error fetching {url}: {e}")
                return url, None

    async def parse_page(self, url, html):
        """Parse une page HTML"""
        if not html:
            return []

        # Parser avec BeautifulSoup (CPU-bound, mais rapide)
        soup = BeautifulSoup(html, 'html.parser')

        # Extraire les liens
        links = []
        for link in soup.find_all('a', href=True):
            href = link['href']
            if href.startswith('http'):
                links.append(href)

        return links

    async def scrape(self, start_urls, max_pages=1000):
        """Scrape des pages web"""
        to_visit = set(start_urls)
        visited = set()
        results = {}

        async with aiohttp.ClientSession() as session:
            while to_visit and len(visited) < max_pages:
                # Prendre un batch d'URLs
                batch_size = min(self.max_concurrent, len(to_visit))
                batch = [to_visit.pop() for _ in range(batch_size)]

                # Fetch en parallèle
                fetch_tasks = [
                    self.fetch_page(session, url)
                    for url in batch
                ]
                pages = await asyncio.gather(*fetch_tasks)

                # Parse en parallèle
                parse_tasks = [
                    self.parse_page(url, html)
                    for url, html in pages
                ]
                all_links = await asyncio.gather(*parse_tasks)

                # Traiter les résultats
                for (url, html), links in zip(pages, all_links):
                    visited.add(url)
                    results[url] = {
                        'success': html is not None,
                        'links_found': len(links)
                    }

                    # Ajouter les nouveaux liens
                    for link in links:
                        if link not in visited and link not in to_visit:
                            to_visit.add(link)

                print(f"Scraped: {len(visited)}/{max_pages}, Queue: {len(to_visit)}")

        return results

# Utilisation
async def main():
    scraper = AsyncWebScraper(max_concurrent=50)

    results = await scraper.scrape(
        start_urls=['https://example.com'],
        max_pages=500
    )

    print(f"Total pages scraped: {len(results)}")
    successful = sum(1 for r in results.values() if r['success'])
    print(f"Successful: {successful}")

asyncio.run(main())

# Performance :
# 500 pages en ~30 secondes
# vs 500+ secondes en synchrone
```

### 2. Proxy HTTP asynchrone

```python
import asyncio

class AsyncHTTPProxy:
    """Proxy HTTP asynchrone"""

    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

    async def handle_client(self, client_reader, client_writer):
        """Gère une connexion client"""
        client_addr = client_writer.get_extra_info('peername')
        print(f"[+] Client: {client_addr}")

        try:
            # Lire la requête HTTP du client
            request = await client_reader.read(4096)

            if not request:
                return

            # Parser la requête pour extraire l'hôte
            lines = request.decode('utf-8').split('\r\n')
            first_line = lines[0]  # GET /path HTTP/1.1

            # Trouver le Host header
            host = None
            for line in lines[1:]:
                if line.lower().startswith('host:'):
                    host = line.split(':', 1)[1].strip()
                    break

            if not host:
                client_writer.write(b'HTTP/1.1 400 Bad Request\r\n\r\n')
                await client_writer.drain()
                return

            # Connecter au serveur distant
            print(f"  → Connecting to {host}")
            remote_reader, remote_writer = await asyncio.open_connection(
                host, 80
            )

            # Forward la requête
            remote_writer.write(request)
            await remote_writer.drain()

            # Bidirectional proxy
            await asyncio.gather(
                self.forward(client_reader, remote_writer, 'client→remote'),
                self.forward(remote_reader, client_writer, 'remote→client')
            )

        except Exception as e:
            print(f"Error: {e}")

        finally:
            client_writer.close()
            await client_writer.wait_closed()

    async def forward(self, reader, writer, direction):
        """Forward les données dans une direction"""
        try:
            while True:
                data = await reader.read(8192)

                if not data:
                    break

                writer.write(data)
                await writer.drain()

                print(f"  {direction}: {len(data)} bytes")

        except Exception as e:
            print(f"Forward error ({direction}): {e}")

        finally:
            writer.close()
            await writer.wait_closed()

    async def start(self):
        """Démarre le proxy"""
        server = await asyncio.start_server(
            self.handle_client,
            self.host,
            self.port
        )

        print(f"Proxy HTTP on {self.host}:{self.port}")

        async with server:
            await server.serve_forever()

# Lancement
if __name__ == "__main__":
    proxy = AsyncHTTPProxy()
    asyncio.run(proxy.start())

# Test :
# curl -x http://localhost:8080 http://example.com
```

### 3. Task Queue asynchrone (type Celery)

```python
import asyncio
import json
from datetime import datetime

class AsyncTaskQueue:
    """File de tâches asynchrone"""

    def __init__(self):
        self.queue = asyncio.Queue()
        self.results = {}
        self.workers = []

    async def enqueue(self, task_id, task_func, *args, **kwargs):
        """Ajoute une tâche à la queue"""
        await self.queue.put({
            'id': task_id,
            'func': task_func,
            'args': args,
            'kwargs': kwargs,
            'enqueued_at': datetime.now()
        })
        print(f"[QUEUE] Task {task_id} enqueued")

    async def worker(self, worker_id):
        """Worker qui traite les tâches"""
        print(f"[WORKER-{worker_id}] Started")

        while True:
            # Récupérer une tâche
            task = await self.queue.get()

            task_id = task['id']
            print(f"[WORKER-{worker_id}] Processing task {task_id}")

            try:
                # Exécuter la tâche
                start = datetime.now()
                result = await task['func'](*task['args'], **task['kwargs'])
                duration = (datetime.now() - start).total_seconds()

                # Stocker le résultat
                self.results[task_id] = {
                    'status': 'success',
                    'result': result,
                    'duration': duration,
                    'worker': worker_id
                }

                print(f"[WORKER-{worker_id}] Task {task_id} completed in {duration:.2f}s")

            except Exception as e:
                # Erreur
                self.results[task_id] = {
                    'status': 'error',
                    'error': str(e),
                    'worker': worker_id
                }

                print(f"[WORKER-{worker_id}] Task {task_id} failed: {e}")

            finally:
                self.queue.task_done()

    async def start_workers(self, num_workers=4):
        """Démarre les workers"""
        for i in range(num_workers):
            worker = asyncio.create_task(self.worker(i))
            self.workers.append(worker)

    async def wait_completion(self):
        """Attend que toutes les tâches soient terminées"""
        await self.queue.join()

    def get_result(self, task_id):
        """Récupère le résultat d'une tâche"""
        return self.results.get(task_id)

# Exemples de tâches
async def send_email(to, subject, body):
    """Simule l'envoi d'email"""
    await asyncio.sleep(1)  # Simule latence SMTP
    return f"Email sent to {to}"

async def process_image(image_url):
    """Simule traitement d'image"""
    await asyncio.sleep(2)  # Simule traitement
    return f"Image processed: {image_url}"

async def fetch_data(api_url):
    """Simule fetch API"""
    await asyncio.sleep(0.5)
    return f"Data from {api_url}"

# Utilisation
async def main():
    # Créer la queue
    queue = AsyncTaskQueue()

    # Démarrer 4 workers
    await queue.start_workers(num_workers=4)

    # Enqueue des tâches
    await queue.enqueue('email-1', send_email, 'user@example.com', 'Hello', 'Body')
    await queue.enqueue('image-1', process_image, 'http://example.com/img.jpg')
    await queue.enqueue('api-1', fetch_data, 'https://api.example.com/data')
    await queue.enqueue('email-2', send_email, 'admin@example.com', 'Report', 'Stats')

    # En ajouter plus
    for i in range(10):
        await queue.enqueue(f'task-{i}', fetch_data, f'http://api.com/{i}')

    # Attendre la fin
    await queue.wait_completion()

    # Afficher les résultats
    print("\n=== Results ===")
    for task_id, result in queue.results.items():
        print(f"{task_id}: {result['status']} ({result.get('duration', 0):.2f}s)")

asyncio.run(main())
```

### 4. Rate Limiter asynchrone

```python
import asyncio
import time
from collections import deque

class AsyncRateLimiter:
    """Rate limiter asynchrone (Token Bucket)"""

    def __init__(self, rate, per):
        """
        rate: nombre de requêtes
        per: période en secondes

        Exemple: RateLimiter(10, 1.0) = 10 req/sec
        """
        self.rate = rate
        self.per = per
        self.allowance = rate
        self.last_check = time.time()
        self.lock = asyncio.Lock()

    async def acquire(self):
        """Acquiert un token (bloque si rate limit atteint)"""
        async with self.lock:
            current = time.time()
            time_passed = current - self.last_check
            self.last_check = current

            # Ajouter des tokens basé sur le temps passé
            self.allowance += time_passed * (self.rate / self.per)

            if self.allowance > self.rate:
                self.allowance = self.rate  # Cap

            if self.allowance < 1.0:
                # Pas de token disponible, calculer le temps d'attente
                sleep_time = (1.0 - self.allowance) * (self.per / self.rate)
                await asyncio.sleep(sleep_time)
                self.allowance = 0.0
            else:
                self.allowance -= 1.0

# Utilisation
async def api_call(limiter, api_id):
    """Fait un appel API avec rate limiting"""
    await limiter.acquire()

    print(f"[{time.time():.2f}] API call {api_id}")
    # Faire l'appel réel
    await asyncio.sleep(0.1)

async def main():
    # 5 requêtes par seconde max
    limiter = AsyncRateLimiter(rate=5, per=1.0)

    # Faire 20 requêtes
    tasks = [api_call(limiter, i) for i in range(20)]
    await asyncio.gather(*tasks)

asyncio.run(main())

# Output :
# [0.00] API call 0
# [0.00] API call 1
# [0.00] API call 2
# [0.00] API call 3
# [0.00] API call 4
# [0.20] API call 5  ← Attend pour respecter le rate
# [0.40] API call 6
# [0.60] API call 7
# ...
```

## Patterns avancés

### 1. Async Context Managers

```python
import asyncio

class AsyncDatabaseConnection:
    """Connexion DB asynchrone avec context manager"""

    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.connection = None

    async def __aenter__(self):
        """Appelé lors du 'async with'"""
        print(f"Connecting to {self.host}:{self.port}...")
        await asyncio.sleep(0.1)  # Simule connexion
        self.connection = f"Connection to {self.host}"
        print("Connected!")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """Appelé à la sortie du 'async with'"""
        print("Closing connection...")
        await asyncio.sleep(0.1)  # Simule fermeture
        self.connection = None
        print("Connection closed!")

    async def query(self, sql):
        """Execute une requête"""
        print(f"Executing: {sql}")
        await asyncio.sleep(0.1)
        return [{"id": 1, "name": "John"}]

# Utilisation
async def main():
    # Le context manager gère automatiquement ouverture/fermeture
    async with AsyncDatabaseConnection('localhost', 5432) as db:
        results = await db.query("SELECT * FROM users")
        print(f"Results: {results}")

    # Connection automatiquement fermée ici

asyncio.run(main())
```

### 2. Async Iterators

```python
import asyncio

class AsyncRange:
    """Range asynchrone (async iterator)"""

    def __init__(self, start, end):
        self.current = start
        self.end = end

    def __aiter__(self):
        return self

    async def __anext__(self):
        if self.current >= self.end:
            raise StopAsyncIteration

        # Simule une opération I/O
        await asyncio.sleep(0.1)

        value = self.current
        self.current += 1
        return value

# Utilisation
async def main():
    async for i in AsyncRange(0, 5):
        print(f"Value: {i}")

asyncio.run(main())

# Cas d'usage réel : pagination d'API
class AsyncAPIPaginator:
    """Itère sur les pages d'une API"""

    def __init__(self, base_url, page_size=100):
        self.base_url = base_url
        self.page_size = page_size
        self.current_page = 1
        self.has_more = True

    def __aiter__(self):
        return self

    async def __anext__(self):
        if not self.has_more:
            raise StopAsyncIteration

        # Fetch page
        async with aiohttp.ClientSession() as session:
            url = f"{self.base_url}?page={self.current_page}&size={self.page_size}"
            async with session.get(url) as resp:
                data = await resp.json()

        # Check si il y a plus de pages
        self.has_more = len(data['items']) == self.page_size
        self.current_page += 1

        return data['items']

# Usage
async def process_all_items():
    async for page in AsyncAPIPaginator('https://api.example.com/items'):
        for item in page:
            await process_item(item)
```

### 3. Async Generators

```python
import asyncio

async def async_generator():
    """Générateur asynchrone"""
    for i in range(5):
        await asyncio.sleep(0.1)  # Simule I/O
        yield i * 2

# Utilisation
async def main():
    async for value in async_generator():
        print(f"Got: {value}")

asyncio.run(main())

# Cas d'usage : Streaming de fichier
async def stream_file(filepath, chunk_size=8192):
    """Streame un fichier de manière asynchrone"""
    async with aiofiles.open(filepath, 'rb') as f:
        while True:
            chunk = await f.read(chunk_size)
            if not chunk:
                break
            yield chunk

# Streaming HTTP response
async def stream_http_response(url):
    """Streame une réponse HTTP"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            async for chunk in resp.content.iter_chunked(8192):
                yield chunk
```

## Débogage et profiling

### Debugging async

```python
import asyncio
import logging

# Activer le logging asyncio
logging.basicConfig(level=logging.DEBUG)
asyncio.get_event_loop().set_debug(True)

async def slow_task():
    """Tâche qui bloque l'event loop (MAUVAIS)"""
    import time
    time.sleep(1)  # ⚠️ Bloque l'event loop !

async def main():
    await slow_task()
# Warning: Executing <Task> took 1.001 seconds
```

### Détection de tâches bloquantes

```python
import asyncio

async def detect_blocking():
    """Détecte les opérations bloquantes"""
    loop = asyncio.get_running_loop()
    loop.slow_callback_duration = 0.1  # Warn si > 100ms

    # Mauvais : bloque
    import time
    time.sleep(0.5)  # ⚠️ Warning !

# asyncio génère un warning automatiquement
```

### Profiling avec aiomonitor

```python
import asyncio
import aiomonitor

async def my_app():
    while True:
        await asyncio.sleep(1)
        # Votre app

# Avec monitoring
async def main():
    # Démarre un serveur de monitoring sur port 50101
    with aiomonitor.start_monitor(loop=asyncio.get_running_loop()):
        await my_app()

# Connexion au monitor :
# telnet localhost 50101
# > ps        # Liste des tâches
# > where <task_id>  # Stack trace d'une tâche
```

## Récapitulatif

### Points clés

🎯 **Async = Concurrence sans threads**
- Un seul thread
- Plusieurs opérations en parallèle
- Event loop gère l'ordonnancement

🎯 **async/await = Syntaxe claire**
- `async def` : définit une coroutine
- `await` : suspend la coroutine
- Lisibilité proche du code synchrone

🎯 **Idéal pour I/O-bound**
- Requêtes réseau
- Bases de données
- APIs externes
- WebSockets

🎯 **Éviter pour CPU-bound**
- Calculs lourds bloquent l'event loop
- Utiliser threads ou processus

### Quand utiliser async ?

| Cas d'usage | Async | Alt. |
|-------------|-------|------|
| Serveur web HP | ✅ | Threads |
| Scraping massif | ✅ | Threads |
| API calls multiples | ✅ | Threads |
| WebSockets | ✅ | - |
| Microservices | ✅ | - |
| Scripts simples | ❌ | Sync |
| Calculs lourds | ❌ | Threads/Processes |
| Legacy libs | ❌ | Sync/Threads |

### Best Practices

✅ **Ne jamais bloquer l'event loop**
```python
# MAUVAIS
import time
time.sleep(1)

# BON
await asyncio.sleep(1)
```

✅ **Utiliser aiohttp, pas requests**
```python
# MAUVAIS (bloque)
import requests
response = requests.get(url)

# BON (async)
import aiohttp
async with aiohttp.ClientSession() as session:
    async with session.get(url) as resp:
        data = await resp.read()
```

✅ **Gérer les erreurs avec gather(return_exceptions=True)**

✅ **Limiter la concurrence avec Semaphore**

✅ **Utiliser create_task() pour background tasks**

## Prochaines étapes

Maintenant que vous maîtrisez la programmation asynchrone :

- **Section 8.8** : Résilience et fiabilité applicative (timeouts, retry, circuit breakers, keep-alive)
- **Section 8.9** : Communication temps réel (WebSockets, SSE, long-polling)
- **Section 8.10** : Architectures de communication modernes (REST, gRPC, GraphQL)

---


⏭️ [Résilience et fiabilité applicative](/08-programmation-reseau/08-resilience-fiabilite.md)
