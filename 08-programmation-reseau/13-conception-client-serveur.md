🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.13 Conception d'applications client-serveur

## Introduction

La conception d'une application client-serveur va bien au-delà de la simple connexion de sockets. Elle implique des décisions architecturales qui affectent :

- 📈 **Scalabilité** : Combien d'utilisateurs simultanés ?
- ⚡ **Performance** : Quelle latence acceptable ?
- 🔒 **Sécurité** : Comment protéger les données ?
- 🔄 **Maintenabilité** : Comment évoluer sans tout casser ?
- 💰 **Coûts** : Infrastructure, développement, opérations

Cette section explore les principes de conception éprouvés et les patterns modernes pour construire des systèmes client-serveur robustes et scalables.

## Principes fondamentaux

### 1. Séparation des responsabilités

```python
# ❌ MAUVAIS : Logique métier dans le client
class BadClient:
    def calculate_invoice(self, items):
        # Calculs complexes côté client
        total = sum(item.price * item.quantity for item in items)
        tax = total * 0.20
        discount = self.apply_discount_rules(total)  # Règles métier
        return total + tax - discount

# ✅ BON : Logique métier sur le serveur
class GoodClient:
    def calculate_invoice(self, items):
        # Le client envoie seulement les données
        response = requests.post('/api/invoices/calculate', json={
            'items': [{'id': i.id, 'quantity': i.quantity} for i in items]
        })
        return response.json()

# Serveur contient la logique métier
@app.post('/api/invoices/calculate')
def calculate_invoice(request):
    items = request.json['items']

    # Logique centralisée
    total = InvoiceService.calculate_total(items)
    tax = TaxService.calculate_tax(total, request.user.country)
    discount = DiscountService.apply_rules(total, request.user)

    return {'total': total + tax - discount}
```

**Avantages** :
- ✅ Logique centralisée (pas de duplication)
- ✅ Mises à jour sans redéployer clients
- ✅ Sécurité (calculs validés côté serveur)
- ✅ Cohérence des données

### 2. Stateless vs Stateful

#### Stateless (sans état)

```python
# Chaque requête est indépendante
@app.get('/api/users/{user_id}')
def get_user(user_id: int, token: str = Header(None)):
    # Le token contient TOUTES les infos nécessaires
    user = authenticate_from_token(token)

    # Pas d'état serveur pour cette session
    return get_user_data(user_id)

# Avantages :
# - Scalable horizontalement (load balancing facile)
# - Pas de mémoire partagée
# - Recovery simple (pas d'état à restaurer)

# Inconvénients :
# - Chaque requête doit contenir toutes les infos (overhead)
# - Pas de transactions multi-requêtes
```

#### Stateful (avec état)

```python
# Le serveur maintient l'état de la session
sessions = {}  # En pratique : Redis, Memcached

@app.post('/api/cart/add')
def add_to_cart(item_id: int, session_id: str = Cookie(None)):
    # État maintenu entre requêtes
    cart = sessions.get(session_id, [])
    cart.append(item_id)
    sessions[session_id] = cart

    return {'cart_size': len(cart)}

# Avantages :
# - Moins d'overhead par requête
# - Transactions multi-étapes possibles
# - Meilleure UX (contexte préservé)

# Inconvénients :
# - Difficile à scaler (sticky sessions)
# - Gestion de la mémoire
# - Recovery complexe
```

**Recommandation** : Préférer **stateless** pour APIs REST, **stateful** pour sessions longues (WebSocket, gaming).

### 3. Idempotence

```python
# Opérations idempotentes : même résultat si répétées
@app.put('/api/users/{user_id}')
def update_user(user_id: int, data: dict):
    # PUT est idempotent
    # Appeler 5× avec même data = même résultat
    user = User.query.get(user_id)
    user.update(data)
    return user

# ❌ Non-idempotent
@app.post('/api/users/{user_id}/credits/add')
def add_credits(user_id: int, amount: int):
    # POST non idempotent
    # Appeler 5× ajoute 5× les crédits !
    user = User.query.get(user_id)
    user.credits += amount  # DANGER
    return user

# ✅ Solution : Idempotency key
@app.post('/api/users/{user_id}/credits/add')
def add_credits(user_id: int, amount: int, idempotency_key: str = Header(None)):
    # Vérifier si déjà traité
    if cache.exists(idempotency_key):
        return cache.get(idempotency_key)

    # Traiter
    user = User.query.get(user_id)
    user.credits += amount

    result = user.to_dict()
    cache.set(idempotency_key, result, ttl=86400)
    return result
```

**HTTP Methods** :
- **GET, PUT, DELETE** : Idempotents
- **POST** : Non idempotent (utiliser idempotency keys)

## Modèles d'architecture

### 1. Architecture 2-Tier (Client-Database)

```
┌──────────┐         ┌──────────┐
│  Client  │────────→│ Database │
│ (Desktop)│         │  (SQL)   │
└──────────┘         └──────────┘
```

**Exemple** : Application desktop avec connexion directe à PostgreSQL

```python
# Client (application desktop)
import psycopg2

class DesktopApp:
    def __init__(self):
        # Connexion directe à la DB
        self.conn = psycopg2.connect(
            host="db.example.com",
            database="myapp",
            user="app_user",
            password="secret"
        )

    def get_users(self):
        cursor = self.conn.cursor()
        cursor.execute("SELECT * FROM users")
        return cursor.fetchall()
```

**Avantages** :
- ✅ Simple
- ✅ Faible latence
- ✅ Pas de couche intermédiaire

**Inconvénients** :
- ❌ Logique métier dupliquée sur chaque client
- ❌ Sécurité (credentials sur clients)
- ❌ Difficile à mettre à jour
- ❌ Pas de cache centralisé
- ❌ Connexions DB multiples

**Cas d'usage** : Applications internes, outils admin

### 2. Architecture 3-Tier (Client-Server-Database)

```
┌──────────┐         ┌────────────┐         ┌──────────┐
│  Client  │────────→│ App Server │────────→│ Database │
│  (Web)   │         │  (API)     │         │  (SQL)   │
└──────────┘         └────────────┘         └──────────┘
```

**Exemple** : Application web moderne

```python
# Client (React)
async function getUsers() {
    const response = await fetch('/api/users');
    return response.json();
}

# Serveur (FastAPI)
from fastapi import FastAPI
from sqlalchemy import create_engine

app = FastAPI()
engine = create_engine('postgresql://...')

@app.get('/api/users')
def get_users():
    # Logique métier centralisée
    users = session.query(User).filter(User.active == True).all()
    return [user.to_dict() for user in users]
```

**Avantages** :
- ✅ Logique centralisée
- ✅ Sécurité (DB isolée)
- ✅ Cache possible
- ✅ Scalable (load balancer)
- ✅ Clients légers

**Inconvénients** :
- ❌ Latence réseau ajoutée
- ❌ Serveur = point de défaillance unique (SPOF)

**Cas d'usage** : Applications web, APIs REST

### 3. Architecture N-Tier (Microservices)

```
┌──────────┐    ┌─────────┐    ┌──────────┐    ┌──────────┐
│  Client  │───→│ Gateway │───→│  Users   │───→│ Users DB │
│          │    │  (API)  │    │ Service  │    └──────────┘
└──────────┘    └─────────┘    └──────────┘
                      │
                      │         ┌──────────┐    ┌──────────┐
                      └────────→│  Orders  │───→│Orders DB │
                                │ Service  │    └──────────┘
                                └──────────┘
```

**Exemple** : E-commerce avec microservices

```python
# API Gateway
from fastapi import FastAPI
import httpx

app = FastAPI()

@app.get('/api/user-dashboard/{user_id}')
async def get_dashboard(user_id: int):
    async with httpx.AsyncClient() as client:
        # Appels parallèles aux microservices
        user, orders, recommendations = await asyncio.gather(
            client.get(f'http://user-service/users/{user_id}'),
            client.get(f'http://order-service/users/{user_id}/orders'),
            client.get(f'http://recommendation-service/users/{user_id}')
        )

    return {
        'user': user.json(),
        'orders': orders.json(),
        'recommendations': recommendations.json()
    }

# Service utilisateurs
@app.get('/users/{user_id}')
def get_user(user_id: int):
    # Service indépendant avec sa propre DB
    user = UserDB.get(user_id)
    return user
```

**Avantages** :
- ✅ Scalabilité indépendante
- ✅ Isolation des pannes
- ✅ Technologies différentes par service
- ✅ Équipes autonomes

**Inconvénients** :
- ❌ Complexité opérationnelle
- ❌ Latence réseau entre services
- ❌ Transactions distribuées difficiles
- ❌ Debugging complexe

**Cas d'usage** : Grandes applications, forte charge

## Patterns de communication

### 1. Request-Response (synchrone)

```python
# Client envoie requête, attend réponse
import requests

# Client
response = requests.get('https://api.example.com/users/123')
user = response.json()
print(user['name'])

# Serveur
@app.get('/users/{user_id}')
def get_user(user_id: int):
    user = db.get_user(user_id)
    return user
```

**Caractéristiques** :
- ⏱️ Synchrone : client bloqué jusqu'à réponse
- 🔄 1:1 : une requête, une réponse
- 📊 Simple à implémenter et débugger

**Cas d'usage** : APIs REST, CRUD operations

### 2. Fire-and-Forget (asynchrone)

```python
# Client envoie message, ne attend pas de réponse
import pika

# Client (producteur)
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='tasks')
channel.basic_publish(
    exchange='',
    routing_key='tasks',
    body='Process this task'
)
print("Task sent")
connection.close()

# Serveur (consommateur)
def callback(ch, method, properties, body):
    print(f"Processing: {body}")
    # Traitement asynchrone
    process_task(body)

channel.basic_consume(queue='tasks', on_message_callback=callback, auto_ack=True)
channel.start_consuming()
```

**Caractéristiques** :
- 🚀 Asynchrone : client ne bloque pas
- ⚡ Haute performance
- 🔄 Découplage client-serveur

**Cas d'usage** : Logs, metrics, notifications, tâches de fond

### 3. Publish-Subscribe

```python
# Un message envoyé à plusieurs abonnés
import redis

# Publisher
r = redis.Redis()
r.publish('events', 'User 123 logged in')

# Subscriber 1 (Analytics)
pubsub = r.pubsub()
pubsub.subscribe('events')

for message in pubsub.listen():
    if message['type'] == 'message':
        log_to_analytics(message['data'])

# Subscriber 2 (Notifications)
pubsub2 = r.pubsub()
pubsub2.subscribe('events')

for message in pubsub2.listen():
    if message['type'] == 'message':
        send_notification(message['data'])
```

**Caractéristiques** :
- 📢 1:N : un message, plusieurs destinataires
- 🔗 Découplage fort
- 📊 Event-driven architecture

**Cas d'usage** : Chat, notifications en temps réel, event sourcing

### 4. Streaming bidirectionnel

```python
# Client et serveur échangent en continu
import asyncio
import websockets

# Serveur
async def handler(websocket):
    async for message in websocket:
        print(f"Received: {message}")

        # Réponse immédiate
        await websocket.send(f"Echo: {message}")

        # Le serveur peut aussi envoyer sans attendre
        await websocket.send("Server update")

# Client
async def client():
    async with websockets.connect('ws://localhost:8765') as ws:
        # Client peut envoyer
        await ws.send("Hello")

        # Client peut recevoir
        async for message in ws:
            print(f"Received: {message}")
```

**Caractéristiques** :
- 🔄 Bidirectionnel : client ↔ serveur
- ⚡ Temps réel
- 📊 Connexion persistante

**Cas d'usage** : Chat, gaming, trading, collaboration temps réel

## Design du protocole applicatif

### Protocole texte vs binaire

#### Protocole texte (HTTP-like)

```python
# Exemple : Protocole de chat simple
"""
CLIENT → SERVER:
JOIN #general alice
MSG #general Hello everyone!
QUIT

SERVER → CLIENT:
OK Joined #general
MSG bob Hello alice!
BYE
"""

class ChatProtocol:
    @staticmethod
    def parse_command(line):
        parts = line.strip().split(' ', 2)
        command = parts[0]

        if command == 'JOIN':
            return {'type': 'join', 'channel': parts[1], 'user': parts[2]}
        elif command == 'MSG':
            return {'type': 'message', 'channel': parts[1], 'text': parts[2]}
        elif command == 'QUIT':
            return {'type': 'quit'}

    @staticmethod
    def format_response(response_type, **kwargs):
        if response_type == 'ok':
            return f"OK {kwargs['message']}\n"
        elif response_type == 'message':
            return f"MSG {kwargs['user']} {kwargs['text']}\n"
        elif response_type == 'bye':
            return "BYE\n"
```

**Avantages** :
- ✅ Lisible par humains
- ✅ Debug facile (telnet, netcat)
- ✅ Flexible
- ✅ Tooling simple

**Inconvénients** :
- ❌ Verbeux (overhead)
- ❌ Parsing lent
- ❌ Pas optimal pour performance

#### Protocole binaire

```python
# Exemple : Protocole binaire efficace
import struct

"""
Format message :
[4 bytes: length] [1 byte: type] [N bytes: payload]

Types:
0x01 = JOIN
0x02 = MESSAGE
0x03 = QUIT
"""

class BinaryProtocol:
    JOIN = 0x01
    MESSAGE = 0x02
    QUIT = 0x03

    @staticmethod
    def encode_message(msg_type, payload):
        # Header : length (4 bytes) + type (1 byte)
        header = struct.pack('!IB', len(payload), msg_type)
        return header + payload

    @staticmethod
    def decode_message(data):
        # Lire header
        length, msg_type = struct.unpack('!IB', data[:5])
        payload = data[5:5+length]

        return msg_type, payload

    @staticmethod
    def encode_join(channel, user):
        payload = f"{channel}:{user}".encode('utf-8')
        return BinaryProtocol.encode_message(BinaryProtocol.JOIN, payload)

# Utilisation
msg = BinaryProtocol.encode_join("#general", "alice")
# b'\x00\x00\x00\x0e\x01#general:alice'
# 5 bytes header + 14 bytes payload = 19 bytes total
# vs "JOIN #general alice\n" = 22 bytes texte
```

**Avantages** :
- ✅ Compact (moins de bande passante)
- ✅ Parsing rapide
- ✅ Types stricts
- ✅ Performance

**Inconvénients** :
- ❌ Illisible
- ❌ Debug difficile
- ❌ Nécessite documentation précise

**Recommandation** :
- **Texte** : APIs publiques, debug fréquent, compatibilité
- **Binaire** : Performance critique, mobile, IoT, gaming

### Versioning du protocole

```python
# Version dans header
class VersionedProtocol:
    VERSION = 2

    @staticmethod
    def encode_message(msg_type, payload):
        # [1 byte: version] [1 byte: type] [payload]
        header = struct.pack('!BB', VersionedProtocol.VERSION, msg_type)
        return header + payload

    @staticmethod
    def decode_message(data):
        version, msg_type = struct.unpack('!BB', data[:2])

        if version == 1:
            # Ancien format
            return parse_v1(data[2:])
        elif version == 2:
            # Nouveau format
            return parse_v2(data[2:])
        else:
            raise ValueError(f"Unsupported version: {version}")

# API REST : version dans URL
@app.get('/v1/users/{user_id}')
def get_user_v1(user_id: int):
    # Ancien format
    return {'id': user_id, 'name': 'Alice'}

@app.get('/v2/users/{user_id}')
def get_user_v2(user_id: int):
    # Nouveau format (plus de champs)
    return {
        'id': user_id,
        'name': 'Alice',
        'email': 'alice@example.com',
        'created_at': '2025-01-01'
    }
```

## Gestion de la concurrence côté serveur

### 1. Thread-per-connection

```python
import socket
import threading

class ThreadedServer:
    def __init__(self, host='0.0.0.0', port=8080):
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.bind((host, port))
        self.server.listen(100)

    def start(self):
        print(f"Server listening on port 8080")

        while True:
            client, addr = self.accept()

            # Thread dédié pour chaque client
            thread = threading.Thread(
                target=self.handle_client,
                args=(client, addr)
            )
            thread.daemon = True
            thread.start()

    def handle_client(self, sock, addr):
        print(f"Client {addr} connected")

        try:
            while True:
                data = sock.recv(4096)
                if not data:
                    break

                # Traitement potentiellement bloquant
                response = self.process_request(data)
                sock.sendall(response)

        finally:
            sock.close()
            print(f"Client {addr} disconnected")

    def process_request(self, data):
        # Peut bloquer sans impacter autres clients
        result = slow_operation(data)
        return result
```

**Avantages** :
- ✅ Code séquentiel simple
- ✅ Isolation (un thread bloqué n'affecte pas les autres)
- ✅ Opérations bloquantes OK

**Inconvénients** :
- ❌ Limité à ~1000 connexions (threads)
- ❌ Overhead mémoire (8 MB/thread)
- ❌ Context switching

**Cas d'usage** : < 1000 connexions, opérations CPU-bound

### 2. Thread Pool

```python
from concurrent.futures import ThreadPoolExecutor
from queue import Queue

class ThreadPoolServer:
    def __init__(self, num_workers=10):
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.bind(('0.0.0.0', 8080))
        self.server.listen(100)

        # Pool de workers
        self.executor = ThreadPoolExecutor(max_workers=num_workers)
        self.client_queue = Queue()

    def start(self):
        # Thread accepter
        accept_thread = threading.Thread(target=self.accept_loop)
        accept_thread.daemon = True
        accept_thread.start()

        # Workers traitent les clients
        while True:
            client, addr = self.client_queue.get()
            self.executor.submit(self.handle_client, client, addr)

    def accept_loop(self):
        while True:
            client, addr = self.server.accept()
            self.client_queue.put((client, addr))

    def handle_client(self, sock, addr):
        # Même logique que thread-per-connection
        pass
```

**Avantages** :
- ✅ Nombre de threads contrôlé
- ✅ Meilleure utilisation ressources
- ✅ Évite explosion de threads

**Inconvénients** :
- ❌ Files d'attente si tous workers occupés
- ❌ Toujours limité par threads

**Cas d'usage** : Charge variable, limitation ressources

### 3. Event-Driven (Event Loop)

```python
import asyncio

class AsyncServer:
    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port
        self.clients = set()

    async def handle_client(self, reader, writer):
        addr = writer.get_extra_info('peername')
        self.clients.add(writer)
        print(f"Client {addr} connected")

        try:
            while True:
                # Lecture non-bloquante
                data = await reader.read(4096)

                if not data:
                    break

                # Traitement async
                response = await self.process_request(data)

                # Écriture non-bloquante
                writer.write(response)
                await writer.drain()

        finally:
            self.clients.remove(writer)
            writer.close()
            await writer.wait_closed()
            print(f"Client {addr} disconnected")

    async def process_request(self, data):
        # Opérations I/O async
        result = await async_database_query(data)
        return result

    async def start(self):
        server = await asyncio.start_server(
            self.handle_client,
            self.host,
            self.port
        )

        print(f"Async server on {self.host}:{self.port}")

        async with server:
            await server.serve_forever()

# Lancement
asyncio.run(AsyncServer().start())
```

**Avantages** :
- ✅ 10 000+ connexions simultanées
- ✅ Un seul thread (pas de context switching)
- ✅ Faible consommation mémoire
- ✅ Excellent pour I/O-bound

**Inconvénients** :
- ❌ Complexité code (async/await)
- ❌ Opérations bloquantes = catastrophe
- ❌ Debugging plus difficile

**Cas d'usage** : Haute concurrence, WebSocket, microservices

## Gestion de l'état et des sessions

### 1. Sessions côté serveur (Stateful)

```python
from flask import Flask, session
import redis

app = Flask(__name__)
app.secret_key = 'super-secret-key'

# Stockage sessions dans Redis
session_store = redis.Redis(host='localhost', port=6379, db=0)

@app.route('/login', methods=['POST'])
def login():
    username = request.json['username']
    password = request.json['password']

    # Authentifier
    user = authenticate(username, password)
    if user:
        # Créer session
        session_id = generate_session_id()
        session_data = {
            'user_id': user.id,
            'username': user.username,
            'role': user.role
        }

        # Stocker dans Redis (TTL 1 heure)
        session_store.setex(
            f'session:{session_id}',
            3600,
            json.dumps(session_data)
        )

        # Retourner session ID au client (cookie)
        response = make_response({'status': 'logged_in'})
        response.set_cookie('session_id', session_id, httponly=True)
        return response

@app.route('/api/profile')
def get_profile():
    session_id = request.cookies.get('session_id')

    # Récupérer session
    session_data = session_store.get(f'session:{session_id}')
    if not session_data:
        return {'error': 'Not authenticated'}, 401

    session = json.loads(session_data)
    user_id = session['user_id']

    # Logique métier
    user = User.query.get(user_id)
    return user.to_dict()
```

**Avantages** :
- ✅ Gestion état complexe
- ✅ Révocation facile (delete session)
- ✅ Moins de données dans requêtes

**Inconvénients** :
- ❌ Sticky sessions (load balancing complexe)
- ❌ Stockage centralisé requis (Redis)
- ❌ SPOF si Redis down

### 2. Tokens JWT (Stateless)

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = 'your-secret-key'

@app.route('/login', methods=['POST'])
def login():
    username = request.json['username']
    password = request.json['password']

    user = authenticate(username, password)
    if user:
        # Créer JWT token
        payload = {
            'user_id': user.id,
            'username': user.username,
            'role': user.role,
            'exp': datetime.utcnow() + timedelta(hours=1),  # Expiration
            'iat': datetime.utcnow()  # Issued at
        }

        token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')

        return {'token': token}

@app.route('/api/profile')
def get_profile():
    # Récupérer token du header
    auth_header = request.headers.get('Authorization')
    if not auth_header or not auth_header.startswith('Bearer '):
        return {'error': 'Not authenticated'}, 401

    token = auth_header.split(' ')[1]

    try:
        # Décoder et valider token
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        user_id = payload['user_id']

        # Logique métier
        user = User.query.get(user_id)
        return user.to_dict()

    except jwt.ExpiredSignatureError:
        return {'error': 'Token expired'}, 401
    except jwt.InvalidTokenError:
        return {'error': 'Invalid token'}, 401
```

**Avantages** :
- ✅ Stateless (scalabilité horizontale)
- ✅ Pas de stockage serveur
- ✅ Load balancing simple
- ✅ Peut contenir claims

**Inconvénients** :
- ❌ Révocation difficile (token valide jusqu'à expiration)
- ❌ Taille (token dans chaque requête)
- ❌ Secret key doit être partagé

**Solutions révocation JWT** :

```python
# Blacklist de tokens révoqués
revoked_tokens = redis.Redis()

@app.route('/logout', methods=['POST'])
def logout():
    token = get_token_from_request()
    payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])

    # Ajouter à blacklist jusqu'à expiration
    exp = payload['exp']
    ttl = exp - datetime.utcnow().timestamp()

    revoked_tokens.setex(f'revoked:{token}', int(ttl), '1')

    return {'status': 'logged_out'}

def verify_token(token):
    # Vérifier si révoqué
    if revoked_tokens.exists(f'revoked:{token}'):
        raise InvalidTokenError('Token revoked')

    return jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
```

### 3. Refresh Tokens

```python
# Utiliser deux tokens : access (court) + refresh (long)
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(username, password)

    # Access token : 15 minutes
    access_token = jwt.encode({
        'user_id': user.id,
        'type': 'access',
        'exp': datetime.utcnow() + timedelta(minutes=15)
    }, SECRET_KEY)

    # Refresh token : 7 jours
    refresh_token = jwt.encode({
        'user_id': user.id,
        'type': 'refresh',
        'exp': datetime.utcnow() + timedelta(days=7)
    }, SECRET_KEY)

    # Stocker refresh token (pour révocation)
    refresh_tokens_store.set(f'refresh:{user.id}', refresh_token)

    return {
        'access_token': access_token,
        'refresh_token': refresh_token
    }

@app.route('/refresh', methods=['POST'])
def refresh():
    refresh_token = request.json['refresh_token']

    try:
        payload = jwt.decode(refresh_token, SECRET_KEY, algorithms=['HS256'])

        if payload['type'] != 'refresh':
            raise InvalidTokenError()

        user_id = payload['user_id']

        # Vérifier si révoqué
        stored_token = refresh_tokens_store.get(f'refresh:{user_id}')
        if stored_token != refresh_token:
            raise InvalidTokenError('Token revoked')

        # Générer nouveau access token
        access_token = jwt.encode({
            'user_id': user_id,
            'type': 'access',
            'exp': datetime.utcnow() + timedelta(minutes=15)
        }, SECRET_KEY)

        return {'access_token': access_token}

    except jwt.ExpiredSignatureError:
        return {'error': 'Refresh token expired'}, 401
```

## Scalabilité

### 1. Load Balancing

```python
# Configuration Nginx load balancer
"""
upstream backend {
    # Round-robin (défaut)
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;

    # Ou least connections
    least_conn;

    # Ou IP hash (sticky sessions)
    ip_hash;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
"""

# Application doit être stateless ou utiliser sessions partagées
# Redis pour sessions partagées
import redis
from flask_session import Session

app = Flask(__name__)
app.config['SESSION_TYPE'] = 'redis'
app.config['SESSION_REDIS'] = redis.from_url('redis://localhost:6379')

Session(app)

# Maintenant toutes les instances partagent les sessions
```

### 2. Caching

```python
import functools
import redis
import hashlib
import json

cache = redis.Redis(host='localhost', port=6379, decode_responses=True)

def cache_result(ttl=300):
    """Decorator pour cacher résultats"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Générer clé cache
            cache_key = f"{func.__name__}:{hashlib.md5(str(args).encode()).hexdigest()}"

            # Vérifier cache
            cached = cache.get(cache_key)
            if cached:
                print(f"Cache HIT: {cache_key}")
                return json.loads(cached)

            # Exécuter fonction
            print(f"Cache MISS: {cache_key}")
            result = func(*args, **kwargs)

            # Mettre en cache
            cache.setex(cache_key, ttl, json.dumps(result))

            return result
        return wrapper
    return decorator

# Utilisation
@app.route('/api/products/<int:product_id>')
@cache_result(ttl=600)  # Cache 10 minutes
def get_product(product_id):
    # Requête DB coûteuse
    product = db.query(Product).get(product_id)
    return product.to_dict()

# Invalidation cache
@app.route('/api/products/<int:product_id>', methods=['PUT'])
def update_product(product_id):
    # Mettre à jour
    product = db.query(Product).get(product_id)
    product.update(request.json)
    db.commit()

    # Invalider cache
    cache_key = f"get_product:{hashlib.md5(str(product_id).encode()).hexdigest()}"
    cache.delete(cache_key)

    return product.to_dict()
```

**Stratégies de cache** :

```python
# 1. Cache-Aside (lazy loading)
def get_user(user_id):
    # Vérifier cache
    user = cache.get(f'user:{user_id}')
    if user:
        return json.loads(user)

    # Charger de DB
    user = db.get_user(user_id)

    # Mettre en cache
    cache.setex(f'user:{user_id}', 3600, json.dumps(user))

    return user

# 2. Write-Through (écriture synchrone)
def update_user(user_id, data):
    # Mettre à jour DB
    db.update_user(user_id, data)

    # Mettre à jour cache immédiatement
    user = db.get_user(user_id)
    cache.setex(f'user:{user_id}', 3600, json.dumps(user))

# 3. Write-Behind (écriture asynchrone)
def update_user(user_id, data):
    # Mettre à jour cache immédiatement
    user = data
    cache.setex(f'user:{user_id}', 3600, json.dumps(user))

    # Queue pour DB (async)
    task_queue.enqueue('update_user_db', user_id, data)
```

### 3. Database Connection Pooling

```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

# Connection pool
engine = create_engine(
    'postgresql://user:password@localhost/mydb',
    poolclass=QueuePool,
    pool_size=10,          # Connexions permanentes
    max_overflow=20,       # Connexions supplémentaires si besoin
    pool_timeout=30,       # Timeout si pool saturé
    pool_recycle=3600,     # Recycler connexions après 1h
    pool_pre_ping=True     # Vérifier connexion avant utilisation
)

# Utilisation
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)

@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    # Session prend connexion du pool
    session = Session()
    try:
        user = session.query(User).get(user_id)
        return user.to_dict()
    finally:
        # Retourne connexion au pool
        session.close()
```

## Résilience et Fault Tolerance

### 1. Retry avec Exponential Backoff

```python
import time
import random

def retry_with_backoff(
    func,
    max_retries=3,
    base_delay=1,
    max_delay=60,
    exponential_base=2,
    jitter=True
):
    """Retry avec exponential backoff et jitter"""
    for attempt in range(max_retries):
        try:
            return func()

        except Exception as e:
            if attempt == max_retries - 1:
                raise  # Dernière tentative, propager erreur

            # Calculer délai
            delay = min(base_delay * (exponential_base ** attempt), max_delay)

            # Ajouter jitter (randomisation)
            if jitter:
                delay = delay * (0.5 + random.random())

            print(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay:.2f}s")
            time.sleep(delay)

# Utilisation
def unreliable_api_call():
    response = requests.get('https://unreliable-api.com/data', timeout=5)
    response.raise_for_status()
    return response.json()

data = retry_with_backoff(unreliable_api_call, max_retries=5)
```

### 2. Circuit Breaker

```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"      # Normal
    OPEN = "open"          # Échecs, fail fast
    HALF_OPEN = "half_open"  # Test récupération

class CircuitBreaker:
    def __init__(
        self,
        failure_threshold=5,
        recovery_timeout=60,
        success_threshold=2
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.success_threshold = success_threshold

        self.failure_count = 0
        self.success_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            # Vérifier si timeout écoulé
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = CircuitState.HALF_OPEN
                self.success_count = 0
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result

        except Exception as e:
            self.on_failure()
            raise

    def on_success(self):
        self.failure_count = 0

        if self.state == CircuitState.HALF_OPEN:
            self.success_count += 1
            if self.success_count >= self.success_threshold:
                self.state = CircuitState.CLOSED
                print("Circuit breaker CLOSED (recovered)")

    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()

        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
            print("Circuit breaker OPEN (too many failures)")

# Utilisation
breaker = CircuitBreaker(failure_threshold=3, recovery_timeout=30)

@app.route('/api/external-service')
def call_external_service():
    try:
        result = breaker.call(requests.get, 'https://external.com/api')
        return result.json()
    except Exception as e:
        return {'error': 'Service unavailable'}, 503
```

### 3. Graceful Degradation

```python
@app.route('/api/recommendations/<int:user_id>')
def get_recommendations(user_id):
    try:
        # Essayer service de recommandations ML
        recommendations = recommendation_service.get(user_id, timeout=2)
        return {'recommendations': recommendations, 'source': 'ml'}

    except (Timeout, ServiceUnavailable):
        # Fallback : recommandations basiques
        popular_items = cache.get('popular_items')
        if not popular_items:
            popular_items = db.get_popular_items(limit=10)
            cache.setex('popular_items', 300, popular_items)

        return {'recommendations': popular_items, 'source': 'fallback'}

# Multi-level fallback
@app.route('/api/product-image/<int:product_id>')
def get_product_image(product_id):
    # 1. CDN principal
    try:
        return redirect(f'https://cdn1.example.com/products/{product_id}.jpg')
    except:
        pass

    # 2. CDN secondaire
    try:
        return redirect(f'https://cdn2.example.com/products/{product_id}.jpg')
    except:
        pass

    # 3. Stockage local
    try:
        return send_file(f'/var/images/products/{product_id}.jpg')
    except:
        pass

    # 4. Image placeholder
    return send_file('/static/images/no-image.jpg')
```

### 4. Timeouts à tous les niveaux

```python
# Client
response = requests.get(
    'https://api.example.com/data',
    timeout=(3.0, 10.0)  # Connect: 3s, Read: 10s
)

# Serveur : timeout global par requête
from flask import Flask, request
import signal

app = Flask(__name__)

class TimeoutException(Exception):
    pass

def timeout_handler(signum, frame):
    raise TimeoutException()

@app.before_request
def before_request():
    # Timeout de 30s par requête
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(30)

@app.after_request
def after_request(response):
    signal.alarm(0)  # Annuler timeout
    return response

# Database : timeout query
session.execute(
    text("SELECT * FROM large_table"),
    execution_options={"timeout": 5}
)

# Redis : timeout
cache = redis.Redis(socket_timeout=2, socket_connect_timeout=2)
```

## Anti-patterns à éviter

### 1. Chatty API (trop d'appels)

```python
# ❌ MAUVAIS : N+1 queries
@app.route('/api/users')
def get_users():
    users = db.query(User).all()

    result = []
    for user in users:
        # 1 query par user !
        orders = db.query(Order).filter(Order.user_id == user.id).all()
        user_dict = user.to_dict()
        user_dict['orders'] = [o.to_dict() for o in orders]
        result.append(user_dict)

    return result

# ✅ BON : Eager loading
@app.route('/api/users')
def get_users():
    users = db.query(User).options(joinedload(User.orders)).all()

    return [
        {**user.to_dict(), 'orders': [o.to_dict() for o in user.orders]}
        for user in users
    ]
```

### 2. God Objects (trop de données)

```python
# ❌ MAUVAIS : Retourner tout
@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    user = db.query(User).get(user_id)
    return user.to_dict()  # Tous les champs, même sensibles !

# ✅ BON : Sérialisation contrôlée
class UserPublicSchema:
    def serialize(self, user):
        return {
            'id': user.id,
            'username': user.username,
            'avatar_url': user.avatar_url
        }

@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    user = db.query(User).get(user_id)
    return UserPublicSchema().serialize(user)
```

### 3. Synchronous coupling

```python
# ❌ MAUVAIS : Blocage sur service externe
@app.route('/api/checkout', methods=['POST'])
def checkout():
    order = create_order(request.json)

    # Bloque si service email down !
    send_confirmation_email(order)

    return {'order_id': order.id}

# ✅ BON : Queue asynchrone
@app.route('/api/checkout', methods=['POST'])
def checkout():
    order = create_order(request.json)

    # Queue async (ne bloque pas)
    email_queue.enqueue('send_confirmation', order.id)

    return {'order_id': order.id}
```

### 4. Absence de pagination

```python
# ❌ MAUVAIS : Retourner tout
@app.route('/api/products')
def get_products():
    products = db.query(Product).all()  # 100,000 produits !
    return [p.to_dict() for p in products]

# ✅ BON : Pagination
@app.route('/api/products')
def get_products():
    page = int(request.args.get('page', 1))
    per_page = int(request.args.get('per_page', 20))

    # Limit + offset
    products = db.query(Product).limit(per_page).offset((page - 1) * per_page).all()
    total = db.query(Product).count()

    return {
        'products': [p.to_dict() for p in products],
        'page': page,
        'per_page': per_page,
        'total': total,
        'pages': (total + per_page - 1) // per_page
    }
```

## Best Practices récapitulatives

### Design

- ✅ **Séparation des responsabilités** : Logique métier sur serveur
- ✅ **Stateless quand possible** : Scalabilité horizontale
- ✅ **Idempotence** : Opérations répétables sans effet de bord
- ✅ **Versioning** : URLs ou headers versionnées
- ✅ **Documentation** : OpenAPI/Swagger pour APIs

### Performance

- ✅ **Connection pooling** : Database, HTTP clients
- ✅ **Caching** : Redis, CDN
- ✅ **Pagination** : Jamais retourner tout
- ✅ **Compression** : gzip, brotli
- ✅ **CDN** : Assets statiques

### Résilience

- ✅ **Timeouts partout** : Connect, read, write
- ✅ **Retry logic** : Exponential backoff + jitter
- ✅ **Circuit breakers** : Protection contre cascades
- ✅ **Graceful degradation** : Fallbacks
- ✅ **Health checks** : Monitoring actif

### Sécurité

- ✅ **HTTPS toujours** : Chiffrement transport
- ✅ **Authentication** : JWT, OAuth
- ✅ **Authorization** : RBAC, permissions
- ✅ **Rate limiting** : Protection DDoS
- ✅ **Input validation** : Jamais faire confiance au client
- ✅ **CORS** : Configuration stricte

### Monitoring

- ✅ **Logging structuré** : JSON, contexte
- ✅ **Métriques** : Latence, erreurs, throughput
- ✅ **Tracing** : Distributed tracing (Jaeger, Zipkin)
- ✅ **Alertes** : Notifications proactives
- ✅ **Dashboards** : Grafana, Kibana

## Récapitulatif

### Architecture selon échelle

```
< 100 utilisateurs :
→ 2-tier, serveur simple, DB unique

100-10,000 utilisateurs :
→ 3-tier, load balancer, cache, DB répliquée

10,000-1M utilisateurs :
→ Microservices, CDN, multiple DBs, queues

> 1M utilisateurs :
→ Architecture distribuée, sharding, event sourcing
```

### Checklist de conception

```python
# Avant de commencer :

□ Combien d'utilisateurs simultanés ?
□ Quelle latence acceptable ?
□ Données sensibles ? (sécurité)
□ Disponibilité requise ? (99.9% = 8.76h downtime/an)
□ Budget infrastructure ?
□ Compétences équipe ?
□ Legacy constraints ?

# Décisions clés :

□ Sync ou async ?
□ Stateful ou stateless ?
□ Monolithe ou microservices ?
□ SQL ou NoSQL ?
□ REST ou GraphQL ou gRPC ?
□ Sessions ou tokens ?
□ Cache strategy ?
```

## Prochaines étapes

Maintenant que vous maîtrisez la conception client-serveur :

- **Section 8.14** : Implications réseau pour les microservices
- **Module 9** : Sécurité réseau
- **Module 10** : Optimisation et troubleshooting

---


⏭️ [Implications réseau pour les microservices](/08-programmation-reseau/14-implications-microservices.md)
