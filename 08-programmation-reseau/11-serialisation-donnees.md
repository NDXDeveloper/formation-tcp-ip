🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.11 Sérialisation des données : JSON, Protocol Buffers, MessagePack

## Introduction

Dans toute application réseau, les données doivent être **sérialisées** (converties en bytes) pour être transmises sur le réseau, puis **désérialisées** (converties en objets) à la réception.

```python
# Côté émetteur
user = {'name': 'Alice', 'age': 30, 'email': 'alice@example.com'}
                    ↓ Sérialisation
bytes = serialize(user)  # b'{"name":"Alice",...}'
                    ↓ Transmission réseau
                    ↓
# Côté récepteur
bytes = receive()        # b'{"name":"Alice",...}'
                    ↓ Désérialisation
user = deserialize(bytes)  # {'name': 'Alice', ...}
```

Le choix du format de sérialisation a un impact **majeur** sur :

- 📦 **Taille des données** : Bande passante, coûts réseau
- ⚡ **Performance** : CPU pour sérialiser/désérialiser
- 🔍 **Lisibilité** : Debug, logs, inspection
- 🔄 **Compatibilité** : Langages, versions, évolution du schéma
- 🛡️ **Sécurité** : Injections, validation

Cette section compare les trois formats les plus utilisés : **JSON**, **Protocol Buffers** et **MessagePack**.

## Pourquoi la sérialisation est cruciale

### Impact sur les performances

```python
# Exemple : API REST qui retourne 10 000 utilisateurs

# Format 1 : JSON (texte)
# Taille : 2.5 MB
# Temps sérialisation : 150ms
# Temps réseau (1 Gbps) : 20ms
# Temps désérialisation : 180ms
# TOTAL : 350ms

# Format 2 : MessagePack (binaire compact)
# Taille : 1.2 MB
# Temps sérialisation : 80ms
# Temps réseau (1 Gbps) : 10ms
# Temps désérialisation : 90ms
# TOTAL : 180ms

# Gain : 48% plus rapide !
```

### Impact sur les coûts

```python
# API publique : 1 milliard de requêtes/mois
# Taille moyenne réponse : 50 KB

# JSON : 50 KB × 1 milliard = 50 TB/mois
# Coût AWS transfer : 50 TB × $0.09/GB = $4,500/mois

# Protocol Buffers : 15 KB × 1 milliard = 15 TB/mois
# Coût AWS transfer : 15 TB × $0.09/GB = $1,350/mois

# Économie : $3,150/mois = $37,800/an
```

## JSON (JavaScript Object Notation)

### Caractéristiques

JSON est le format **le plus utilisé** pour les APIs web modernes.

```json
{
  "user_id": 12345,
  "username": "alice_smith",
  "email": "alice@example.com",
  "is_active": true,
  "balance": 1234.56,
  "tags": ["premium", "verified"],
  "metadata": {
    "last_login": "2025-01-15T10:30:00Z",
    "login_count": 42
  }
}
```

**Avantages** :

- ✅ **Lisible** : Facile à lire et débugger
- ✅ **Universel** : Supporté partout (tous langages, navigateurs)
- ✅ **Simple** : Pas de schéma requis
- ✅ **Flexible** : Structure dynamique
- ✅ **Tooling** : Excellents outils (jq, validateurs, formatters)

**Inconvénients** :

- ❌ **Verbeux** : Beaucoup de caractères pour peu de données
- ❌ **Lent** : Parsing texte coûteux en CPU
- ❌ **Pas de schéma** : Erreurs possibles, pas de validation intégrée
- ❌ **Types limités** : Pas de dates, binaires, etc.
- ❌ **Taille** : 2-3× plus gros que les formats binaires

### Utilisation en Python

```python
import json

# Sérialisation (Python → JSON)
user = {
    'user_id': 12345,
    'username': 'alice_smith',
    'email': 'alice@example.com',
    'is_active': True,
    'balance': 1234.56,
    'tags': ['premium', 'verified']
}

# Vers string
json_str = json.dumps(user)
print(json_str)
# {"user_id": 12345, "username": "alice_smith", ...}

# Vers bytes (pour réseau)
json_bytes = json.dumps(user).encode('utf-8')
print(len(json_bytes))  # ~150 bytes

# Désérialisation (JSON → Python)
received_bytes = b'{"user_id": 12345, "username": "alice_smith"}'
user = json.loads(received_bytes.decode('utf-8'))
print(user['username'])  # alice_smith
```

### Options de sérialisation

```python
import json

data = {'name': 'Alice', 'age': 30, 'items': [1, 2, 3]}

# Compact (production)
compact = json.dumps(data, separators=(',', ':'))
print(compact)
# {"name":"Alice","age":30,"items":[1,2,3]}

# Indenté (debug/logs)
pretty = json.dumps(data, indent=2)
print(pretty)
# {
#   "name": "Alice",
#   "age": 30,
#   "items": [1, 2, 3]
# }

# Trier les clés (reproductibilité)
sorted_json = json.dumps(data, sort_keys=True)
print(sorted_json)
# {"age":30,"items":[1,2,3],"name":"Alice"}

# Assurer ASCII (compatibilité)
ascii_json = json.dumps({'name': 'François'}, ensure_ascii=True)
print(ascii_json)
# {"name": "Fran\u00e7ois"}
```

### Types personnalisés

```python
import json
from datetime import datetime
from decimal import Decimal

# Problème : datetime et Decimal ne sont pas sérialisables
data = {
    'timestamp': datetime.now(),
    'price': Decimal('19.99')
}

try:
    json.dumps(data)
except TypeError as e:
    print(f"Erreur : {e}")
    # TypeError: Object of type datetime is not JSON serializable

# Solution 1 : Encoder personnalisé
class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, Decimal):
            return float(obj)
        return super().default(obj)

json_str = json.dumps(data, cls=CustomEncoder)
print(json_str)
# {"timestamp": "2025-01-15T10:30:00.123456", "price": 19.99}

# Solution 2 : Fonction default
def serialize_custom(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    if isinstance(obj, Decimal):
        return str(obj)
    raise TypeError(f"Type {type(obj)} not serializable")

json_str = json.dumps(data, default=serialize_custom)
```

### Performance JSON

```python
import json
import time

# Benchmark
data = {
    'users': [
        {'id': i, 'name': f'User{i}', 'email': f'user{i}@example.com'}
        for i in range(10000)
    ]
}

# Sérialisation
start = time.time()
json_bytes = json.dumps(data).encode('utf-8')
serialize_time = time.time() - start

print(f"Taille JSON : {len(json_bytes):,} bytes")
print(f"Temps sérialisation : {serialize_time*1000:.2f} ms")

# Désérialisation
start = time.time()
loaded = json.loads(json_bytes.decode('utf-8'))
deserialize_time = time.time() - start

print(f"Temps désérialisation : {deserialize_time*1000:.2f} ms")

# Résultats typiques :
# Taille JSON : 890,000 bytes
# Temps sérialisation : 45.23 ms
# Temps désérialisation : 52.18 ms
```

### Alternatives JSON rapides

```python
# ujson : JSON ultra-rapide (écrit en C)
import ujson

data = {'key': 'value'}

# 2-3× plus rapide que json standard
json_str = ujson.dumps(data)
loaded = ujson.loads(json_str)

# orjson : JSON le plus rapide (écrit en Rust)
import orjson

# Retourne bytes directement
json_bytes = orjson.dumps(data)
loaded = orjson.loads(json_bytes)

# Benchmark comparatif (10000 objets)
# json (stdlib)  : 45 ms serialize, 52 ms deserialize
# ujson          : 18 ms serialize, 22 ms deserialize
# orjson         : 12 ms serialize, 15 ms deserialize
```

### Cas d'usage JSON

✅ **APIs REST publiques** : Universalité, lisibilité

```python
# API REST typique
@app.route('/api/users/<int:user_id>')
def get_user(user_id):
    user = db.get_user(user_id)
    return jsonify(user)  # Automatiquement sérialisé en JSON
```

✅ **Configuration files** : Lisible par humains

```json
{
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "myapp"
  },
  "logging": {
    "level": "INFO",
    "file": "/var/log/myapp.log"
  }
}
```

✅ **Logs structurés** : Facilite l'analyse

```python
import logging
import json

logger = logging.getLogger(__name__)

# Log en JSON structuré
log_entry = {
    'timestamp': datetime.now().isoformat(),
    'level': 'INFO',
    'user_id': 12345,
    'action': 'login',
    'ip': '192.168.1.100'
}

logger.info(json.dumps(log_entry))
# Facilite l'indexation dans Elasticsearch/Splunk
```

✅ **Communication navigateur ↔ serveur** : Support natif JavaScript

```javascript
// JavaScript côté client
fetch('/api/users/123')
  .then(response => response.json())  // Parse JSON automatique
  .then(user => console.log(user.name));
```

❌ **Éviter JSON pour** :
- Données binaires volumineuses (images, vidéos)
- Communication haute fréquence (jeux, trading)
- Microservices internes (privilégier binaire)
- Données sensibles à la taille (mobile, IoT)

## Protocol Buffers (protobuf)

### Caractéristiques

Protocol Buffers est un format **binaire** développé par Google, utilisé massivement en interne.

**Avantages** :

- ✅ **Compact** : 3-10× plus petit que JSON
- ✅ **Rapide** : 5-10× plus rapide que JSON
- ✅ **Schéma** : Validation automatique, backward/forward compatibility
- ✅ **Multi-langages** : Code généré pour 20+ langages
- ✅ **Évolutif** : Ajout/suppression de champs sans casser la compatibilité

**Inconvénients** :

- ❌ **Binaire** : Pas lisible par humains
- ❌ **Complexité** : Nécessite définir un schéma .proto
- ❌ **Tooling** : Nécessite compilateur protoc
- ❌ **Courbe d'apprentissage** : Plus complexe que JSON

### Définition du schéma (.proto)

```protobuf
// user.proto
syntax = "proto3";

package myapp;

message User {
  int32 user_id = 1;
  string username = 2;
  string email = 3;
  bool is_active = 4;
  double balance = 5;
  repeated string tags = 6;
  Metadata metadata = 7;
}

message Metadata {
  string last_login = 1;
  int32 login_count = 2;
}
```

**Points clés** :

- `syntax = "proto3"` : Version du protocole
- Chaque champ a un **numéro unique** (crucial pour compatibilité)
- `repeated` = liste/array
- Types supportés : int32, int64, string, bool, double, bytes, etc.

### Compilation du schéma

```bash
# Installer protoc
# macOS: brew install protobuf
# Ubuntu: apt-get install protobuf-compiler

# Compiler pour Python
protoc --python_out=. user.proto

# Génère : user_pb2.py
# Contient les classes Python pour User et Metadata
```

### Utilisation en Python

```python
# Importer les classes générées
import user_pb2

# Créer un objet
user = user_pb2.User()
user.user_id = 12345
user.username = "alice_smith"
user.email = "alice@example.com"
user.is_active = True
user.balance = 1234.56
user.tags.extend(["premium", "verified"])

# Metadata
user.metadata.last_login = "2025-01-15T10:30:00Z"
user.metadata.login_count = 42

# Sérialisation (Python → bytes)
serialized = user.SerializeToString()
print(f"Taille protobuf : {len(serialized)} bytes")
# Taille protobuf : 78 bytes (vs 150 bytes JSON !)

# Désérialisation (bytes → Python)
received = user_pb2.User()
received.ParseFromString(serialized)
print(received.username)  # alice_smith
print(received.tags)      # ['premium', 'verified']
```

### Comparaison de taille

```python
import json
import user_pb2

# Même données en JSON et Protobuf
user_dict = {
    'user_id': 12345,
    'username': 'alice_smith',
    'email': 'alice@example.com',
    'is_active': True,
    'balance': 1234.56,
    'tags': ['premium', 'verified'],
    'metadata': {
        'last_login': '2025-01-15T10:30:00Z',
        'login_count': 42
    }
}

# JSON
json_bytes = json.dumps(user_dict).encode('utf-8')
print(f"JSON : {len(json_bytes)} bytes")

# Protobuf
user_proto = user_pb2.User()
user_proto.user_id = 12345
user_proto.username = "alice_smith"
user_proto.email = "alice@example.com"
user_proto.is_active = True
user_proto.balance = 1234.56
user_proto.tags.extend(["premium", "verified"])
user_proto.metadata.last_login = "2025-01-15T10:30:00Z"
user_proto.metadata.login_count = 42

proto_bytes = user_proto.SerializeToString()
print(f"Protobuf : {len(proto_bytes)} bytes")

print(f"Réduction : {(1 - len(proto_bytes)/len(json_bytes)) * 100:.1f}%")

# Output :
# JSON : 182 bytes
# Protobuf : 78 bytes
# Réduction : 57.1%
```

### Évolution du schéma

Un des avantages majeurs de Protobuf : **backward/forward compatibility**

```protobuf
// Version 1
message User {
  int32 user_id = 1;
  string username = 2;
}

// Version 2 (ajout de champs)
message User {
  int32 user_id = 1;
  string username = 2;
  string email = 3;        // Nouveau champ
  bool is_active = 4;      // Nouveau champ
}

// Ancien code (v1) peut lire messages v2
// → Ignore les champs inconnus (email, is_active)

// Nouveau code (v2) peut lire messages v1
// → Utilise valeurs par défaut pour champs manquants
```

**Règles** :

- ✅ **Ajouter des champs** : OK (numéro unique)
- ✅ **Rendre optionnel** : OK
- ⚠️ **Changer le type** : Dangereux (peut casser)
- ❌ **Réutiliser un numéro** : JAMAIS (casse la compatibilité)

```protobuf
// MAUVAIS : Réutiliser numéro 3
message User {
  int32 user_id = 1;
  string username = 2;
  int32 age = 3;  // Avant c'était 'email' !
}
// → Les anciens messages seront mal interprétés
```

### Performance Protobuf

```python
import time
import json
import user_pb2

# Créer 10000 utilisateurs
users_dict = [
    {
        'user_id': i,
        'username': f'user{i}',
        'email': f'user{i}@example.com',
        'is_active': True,
        'balance': 1234.56
    }
    for i in range(10000)
]

# Benchmark JSON
start = time.time()
json_data = json.dumps(users_dict).encode('utf-8')
json_time = time.time() - start

# Benchmark Protobuf
users_proto = user_pb2.UserList()  # Assume message UserList défini
start = time.time()
for user_dict in users_dict:
    user = users_proto.users.add()
    user.user_id = user_dict['user_id']
    user.username = user_dict['username']
    user.email = user_dict['email']
    user.is_active = user_dict['is_active']
    user.balance = user_dict['balance']
proto_data = users_proto.SerializeToString()
proto_time = time.time() - start

print(f"JSON : {len(json_data):,} bytes, {json_time*1000:.2f} ms")
print(f"Protobuf : {len(proto_data):,} bytes, {proto_time*1000:.2f} ms")
print(f"Taille : {(1-len(proto_data)/len(json_data))*100:.1f}% de réduction")
print(f"Vitesse : {json_time/proto_time:.1f}× plus rapide")

# Résultats typiques :
# JSON : 890,000 bytes, 42.15 ms
# Protobuf : 320,000 bytes, 18.23 ms
# Taille : 64.0% de réduction
# Vitesse : 2.3× plus rapide
```

### gRPC : Protobuf + RPC

gRPC utilise Protobuf pour définir les services et messages.

```protobuf
// user_service.proto
syntax = "proto3";

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (stream User);
}

message GetUserRequest {
  int32 user_id = 1;
}

message CreateUserRequest {
  string username = 1;
  string email = 2;
}

message ListUsersRequest {
  int32 page = 1;
  int32 page_size = 2;
}
```

**Implémentation serveur** :

```python
import grpc
from concurrent import futures
import user_service_pb2
import user_service_pb2_grpc

class UserServiceServicer(user_service_pb2_grpc.UserServiceServicer):
    def GetUser(self, request, context):
        # Récupérer de la DB
        user = user_service_pb2.User()
        user.user_id = request.user_id
        user.username = "alice_smith"
        user.email = "alice@example.com"
        return user

    def ListUsers(self, request, context):
        # Stream de résultats
        for i in range(100):
            user = user_service_pb2.User()
            user.user_id = i
            user.username = f"user{i}"
            yield user

# Serveur gRPC
def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_service_pb2_grpc.add_UserServiceServicer_to_server(
        UserServiceServicer(), server
    )
    server.add_insecure_port('[::]:50051')
    server.start()
    server.wait_for_termination()
```

**Client gRPC** :

```python
import grpc
import user_service_pb2
import user_service_pb2_grpc

# Connexion
channel = grpc.insecure_channel('localhost:50051')
stub = user_service_pb2_grpc.UserServiceStub(channel)

# Appel RPC
request = user_service_pb2.GetUserRequest(user_id=123)
user = stub.GetUser(request)
print(f"User: {user.username}")

# Stream
request = user_service_pb2.ListUsersRequest(page=1, page_size=100)
for user in stub.ListUsers(request):
    print(f"User {user.user_id}: {user.username}")
```

### Cas d'usage Protobuf

✅ **Communication microservices** : Compact, rapide, schéma

```python
# Service A → Service B (internal)
# Protobuf est idéal : pas besoin de lisibilité, performance max
```

✅ **Mobile apps** : Économie de bande passante

```python
# App mobile avec connexion limitée
# Protobuf réduit de 60% les données → moins cher pour utilisateur
```

✅ **Logging à haute fréquence** : Compact

```python
# Logs haute fréquence (100k/sec)
# Protobuf réduit la taille de stockage de 70%
```

✅ **APIs internes haute performance** : gRPC

```python
# APIs internes entre services
# gRPC + Protobuf : 10× plus rapide que REST JSON
```

❌ **Éviter Protobuf pour** :
- APIs publiques (JSON plus accessible)
- Debug manuel fréquent (binaire illisible)
- Équipes sans expérience Protobuf
- Besoins de flexibilité extrême

## MessagePack

### Caractéristiques

MessagePack est un format **binaire** compatible avec JSON, mais beaucoup plus compact.

**Avantages** :

- ✅ **Compatible JSON** : Même structure de données
- ✅ **Compact** : 20-50% plus petit que JSON
- ✅ **Rapide** : 2-3× plus rapide que JSON
- ✅ **Pas de schéma** : Flexible comme JSON
- ✅ **Simple** : Facile à adopter (drop-in replacement)

**Inconvénients** :

- ❌ **Binaire** : Pas lisible
- ❌ **Moins compact que Protobuf** : ~2× plus gros
- ❌ **Pas de validation** : Comme JSON
- ❌ **Support limité** : Moins universel que JSON

### Utilisation en Python

```python
import msgpack

# Données Python
user = {
    'user_id': 12345,
    'username': 'alice_smith',
    'email': 'alice@example.com',
    'is_active': True,
    'balance': 1234.56,
    'tags': ['premium', 'verified']
}

# Sérialisation
packed = msgpack.packb(user)
print(f"MessagePack : {len(packed)} bytes")

# Désérialisation
unpacked = msgpack.unpackb(packed)
print(unpacked['username'])  # alice_smith

# Comparaison avec JSON
import json
json_bytes = json.dumps(user).encode('utf-8')
print(f"JSON : {len(json_bytes)} bytes")
print(f"Réduction : {(1 - len(packed)/len(json_bytes)) * 100:.1f}%")

# Output :
# MessagePack : 112 bytes
# JSON : 150 bytes
# Réduction : 25.3%
```

### Types supportés

```python
import msgpack
import datetime

# MessagePack supporte plus de types que JSON
data = {
    'string': 'hello',
    'int': 42,
    'float': 3.14,
    'bool': True,
    'none': None,
    'list': [1, 2, 3],
    'dict': {'nested': 'value'},
    'bytes': b'binary data',  # Supporté !
}

packed = msgpack.packb(data)
unpacked = msgpack.unpackb(packed, raw=False)

print(unpacked['bytes'])  # b'binary data'

# Extension pour datetime
def encode_datetime(obj):
    if isinstance(obj, datetime.datetime):
        return {'__datetime__': True, 'value': obj.isoformat()}
    return obj

def decode_datetime(obj):
    if '__datetime__' in obj:
        return datetime.datetime.fromisoformat(obj['value'])
    return obj

data_with_dt = {
    'timestamp': datetime.datetime.now(),
    'name': 'Alice'
}

packed = msgpack.packb(data_with_dt, default=encode_datetime)
unpacked = msgpack.unpackb(packed, object_hook=decode_datetime, raw=False)
```

### Performance MessagePack

```python
import time
import json
import msgpack

# Dataset
data = {
    'users': [
        {'id': i, 'name': f'User{i}', 'email': f'user{i}@example.com'}
        for i in range(10000)
    ]
}

# JSON
start = time.time()
json_bytes = json.dumps(data).encode('utf-8')
json_serialize = time.time() - start

start = time.time()
json.loads(json_bytes.decode('utf-8'))
json_deserialize = time.time() - start

# MessagePack
start = time.time()
msgpack_bytes = msgpack.packb(data)
msgpack_serialize = time.time() - start

start = time.time()
msgpack.unpackb(msgpack_bytes, raw=False)
msgpack_deserialize = time.time() - start

print(f"JSON :")
print(f"  Taille : {len(json_bytes):,} bytes")
print(f"  Serialize : {json_serialize*1000:.2f} ms")
print(f"  Deserialize : {json_deserialize*1000:.2f} ms")

print(f"MessagePack :")
print(f"  Taille : {len(msgpack_bytes):,} bytes")
print(f"  Serialize : {msgpack_serialize*1000:.2f} ms")
print(f"  Deserialize : {msgpack_deserialize*1000:.2f} ms")

print(f"Réduction taille : {(1-len(msgpack_bytes)/len(json_bytes))*100:.1f}%")
print(f"Speedup serialize : {json_serialize/msgpack_serialize:.1f}×")
print(f"Speedup deserialize : {json_deserialize/msgpack_deserialize:.1f}×")

# Résultats typiques :
# JSON :
#   Taille : 890,000 bytes
#   Serialize : 42.15 ms
#   Deserialize : 48.23 ms
# MessagePack :
#   Taille : 670,000 bytes
#   Serialize : 18.45 ms
#   Deserialize : 22.87 ms
# Réduction taille : 24.7%
# Speedup serialize : 2.3×
# Speedup deserialize : 2.1×
```

### Streaming MessagePack

```python
import msgpack
import socket

# Serveur : envoie un stream de messages
def server():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.bind(('0.0.0.0', 9999))
    sock.listen(1)

    client, addr = sock.accept()
    packer = msgpack.Packer()

    # Envoyer plusieurs messages
    for i in range(100):
        message = {'id': i, 'data': f'Message {i}'}
        packed = packer.pack(message)
        client.sendall(packed)

    client.close()

# Client : reçoit un stream de messages
def client():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('localhost', 9999))

    unpacker = msgpack.Unpacker(raw=False)

    while True:
        chunk = sock.recv(4096)
        if not chunk:
            break

        # Feed au unpacker
        unpacker.feed(chunk)

        # Extraire tous les messages disponibles
        for message in unpacker:
            print(f"Received: {message}")

    sock.close()
```

### Cas d'usage MessagePack

✅ **APIs internes** : Plus rapide que JSON, pas besoin de schéma

```python
# Microservices en Python
# MessagePack : simple upgrade de JSON avec gain de perf
```

✅ **Cache/Redis** : Stockage compact

```python
import redis
import msgpack

r = redis.Redis()

# Stocker avec MessagePack (plus compact que pickle)
user = {'id': 123, 'name': 'Alice'}
r.set('user:123', msgpack.packb(user))

# Récupérer
data = r.get('user:123')
user = msgpack.unpackb(data, raw=False)
```

✅ **Logs binaires** : Plus compact que JSON

```python
import msgpack
import logging

class MessagePackHandler(logging.Handler):
    def emit(self, record):
        log_entry = {
            'timestamp': record.created,
            'level': record.levelname,
            'message': record.getMessage()
        }
        packed = msgpack.packb(log_entry)
        # Écrire dans fichier binaire
        with open('app.msgpack.log', 'ab') as f:
            f.write(packed)
```

✅ **WebSocket** : Messages binaires compacts

```python
import asyncio
import websockets
import msgpack

async def handler(websocket):
    async for message in websocket:
        # Message en MessagePack
        data = msgpack.unpackb(message, raw=False)

        # Traiter
        response = {'echo': data}

        # Répondre en MessagePack
        await websocket.send(msgpack.packb(response))
```

## Comparaison complète

### Tableau comparatif

| Aspect | JSON | MessagePack | Protocol Buffers |
|--------|------|-------------|------------------|
| **Format** | Texte | Binaire | Binaire |
| **Lisibilité** | ✅ Excellente | ❌ Non | ❌ Non |
| **Taille** | 100% | ~75% | ~35% |
| **Vitesse serialize** | 100% | 230% | 280% |
| **Vitesse deserialize** | 100% | 220% | 350% |
| **Schéma requis** | ❌ Non | ❌ Non | ✅ Oui |
| **Validation** | ❌ Non | ❌ Non | ✅ Oui |
| **Évolution schéma** | ⚠️ Manuel | ⚠️ Manuel | ✅ Auto |
| **Universalité** | ✅ Partout | ⚠️ Moyen | ⚠️ Moyen |
| **Courbe apprentissage** | ✅ Facile | ✅ Facile | ❌ Difficile |
| **Tooling** | ✅ Excellent | ⚠️ Moyen | ✅ Bon |
| **Support navigateur** | ✅ Natif | ❌ Lib requise | ❌ Non |

### Benchmark complet

```python
import time
import json
import msgpack
# import user_pb2  # Protobuf (exemple simplifié)

# Dataset : 10000 utilisateurs
users = [
    {
        'user_id': i,
        'username': f'user_{i}',
        'email': f'user{i}@example.com',
        'is_active': True,
        'balance': 1234.56,
        'tags': ['tag1', 'tag2', 'tag3'],
        'metadata': {
            'created_at': '2025-01-15T10:30:00Z',
            'updated_at': '2025-01-15T10:30:00Z',
            'login_count': 42
        }
    }
    for i in range(10000)
]

def benchmark_format(name, serialize_func, deserialize_func, data):
    # Sérialisation
    start = time.time()
    serialized = serialize_func(data)
    serialize_time = (time.time() - start) * 1000

    # Désérialisation
    start = time.time()
    deserialized = deserialize_func(serialized)
    deserialize_time = (time.time() - start) * 1000

    size = len(serialized)

    print(f"{name}:")
    print(f"  Taille : {size:,} bytes")
    print(f"  Serialize : {serialize_time:.2f} ms")
    print(f"  Deserialize : {deserialize_time:.2f} ms")
    print(f"  Total : {serialize_time + deserialize_time:.2f} ms")

    return {
        'size': size,
        'serialize': serialize_time,
        'deserialize': deserialize_time
    }

# JSON
json_results = benchmark_format(
    "JSON",
    lambda d: json.dumps(d).encode('utf-8'),
    lambda b: json.loads(b.decode('utf-8')),
    users
)

# MessagePack
msgpack_results = benchmark_format(
    "MessagePack",
    lambda d: msgpack.packb(d),
    lambda b: msgpack.unpackb(b, raw=False),
    users
)

# Comparaisons
print("\nComparaisons (JSON = baseline):")
print(f"MessagePack taille : {msgpack_results['size']/json_results['size']*100:.1f}%")
print(f"MessagePack vitesse : {json_results['serialize']/msgpack_results['serialize']:.1f}× plus rapide")

# Résultats typiques :
# JSON:
#   Taille : 1,890,000 bytes
#   Serialize : 145.23 ms
#   Deserialize : 168.45 ms
#   Total : 313.68 ms
# MessagePack:
#   Taille : 1,420,000 bytes
#   Serialize : 62.18 ms
#   Deserialize : 78.32 ms
#   Total : 140.50 ms
#
# Comparaisons (JSON = baseline):
# MessagePack taille : 75.1%
# MessagePack vitesse : 2.3× plus rapide
```

### Choix selon le cas d'usage

```python
# Décision tree

def choose_serialization_format(context):
    """Guide de choix du format"""

    # API publique ?
    if context.is_public_api:
        return "JSON"  # Universalité prime

    # Besoin de debug fréquent ?
    if context.frequent_debugging:
        return "JSON"  # Lisibilité importante

    # Schéma strict requis ?
    if context.requires_schema:
        return "Protocol Buffers"  # Validation + évolution

    # Haute fréquence (>10k msg/sec) ?
    if context.high_frequency:
        return "Protocol Buffers"  # Performance max

    # Bande passante limitée (mobile, IoT) ?
    if context.limited_bandwidth:
        if context.has_schema:
            return "Protocol Buffers"
        else:
            return "MessagePack"

    # Communication navigateur ?
    if context.browser_client:
        return "JSON"  # Support natif

    # Microservices internes Python/Ruby ?
    if context.internal_python_services:
        return "MessagePack"  # Bon compromis

    # Par défaut
    return "JSON"  # Sûr et universel

# Exemples
public_api = choose_serialization_format(
    Context(is_public_api=True)
)  # → JSON

internal_microservices = choose_serialization_format(
    Context(internal_python_services=True, high_frequency=True)
)  # → MessagePack ou Protobuf

mobile_app_backend = choose_serialization_format(
    Context(limited_bandwidth=True, has_schema=True)
)  # → Protocol Buffers
```

## Cas d'usage réels par industrie

### 1. E-commerce (Amazon-like)

```python
# Frontend ↔ Backend API : JSON
# - API publique
# - Support JavaScript natif
# - Lisibilité pour debug

@app.route('/api/products/<product_id>')
def get_product(product_id):
    product = db.get_product(product_id)
    return jsonify(product)  # JSON

# Backend microservices : Protocol Buffers
# - Communication interne
# - Haute fréquence
# - Schéma strict

# catalog_service ↔ inventory_service
# Utilise gRPC + Protobuf
stub.CheckInventory(product_id)  # Protobuf

# Cache Redis : MessagePack
# - Stockage compact
# - Pas besoin de schéma
# - Plus rapide que JSON

redis.set(f'product:{product_id}', msgpack.packb(product))
```

### 2. Social Media (Twitter-like)

```python
# API publique : JSON
# - Développeurs tiers
# - Documentation claire

GET /api/tweets/123
{
  "id": 123,
  "text": "Hello world",
  "user": {...},
  "created_at": "2025-01-15T10:30:00Z"
}

# Timeline service (interne) : Protocol Buffers
# - Millions de requêtes/sec
# - Latence < 10ms
# - Schéma strict

message Tweet {
  int64 id = 1;
  string text = 2;
  int64 user_id = 3;
  int64 timestamp = 4;
}

# WebSocket (notifications temps réel) : MessagePack
# - Messages binaires compacts
# - Fréquence élevée
# - Pas de schéma complexe

websocket.send(msgpack.packb({
    'type': 'new_tweet',
    'tweet_id': 123,
    'user_id': 456
}))
```

### 3. Gaming (Multiplayer)

```python
# Login/lobby : JSON
# - Fréquence faible
# - Données simples

POST /api/auth/login
{
  "username": "player123",
  "password": "..."
}

# Gameplay (state updates) : Protocol Buffers
# - 60+ updates/sec par joueur
# - Latence critique (< 50ms)
# - Taille minimale

message PlayerState {
  int32 player_id = 1;
  float position_x = 2;
  float position_y = 3;
  float position_z = 4;
  float rotation = 5;
  int32 health = 6;
}

# 100 bytes JSON → 30 bytes Protobuf
# × 60 updates/sec × 1000 joueurs
# = 6 MB/sec → 1.8 MB/sec (économie : 70%)

# Chat : MessagePack
# - Fréquence moyenne
# - Flexibilité
# - Compact

chat_message = msgpack.packb({
    'from': player_id,
    'text': message,
    'timestamp': time.time()
})
```

### 4. IoT (Smart Home)

```python
# Device → Cloud : Protocol Buffers
# - Bande passante limitée (3G/4G)
# - Batterie limitée (moins de CPU)
# - Messages fréquents

message SensorReading {
  int32 device_id = 1;
  int64 timestamp = 2;
  float temperature = 3;
  float humidity = 4;
  int32 battery_level = 5;
}

# 80 bytes JSON → 25 bytes Protobuf
# 1 lecture/min × 1000 devices × 30 jours
# = 3.4 GB/mois → 1.1 GB/mois (économie : 68%)

# Dashboard API : JSON
# - API web
# - Lisibilité
# - Intégration facile

GET /api/devices/123/readings
{
  "device_id": 123,
  "readings": [
    {"timestamp": "...", "temperature": 22.5, "humidity": 45},
    ...
  ]
}

# Configuration : JSON
# - Édition manuelle
# - Lisibilité importante

{
  "device_id": 123,
  "location": "Living Room",
  "thresholds": {
    "temperature_min": 18,
    "temperature_max": 26
  }
}
```

### 5. Financial Trading

```python
# Market Data (ultra haute fréquence) : Protocol Buffers
# - 1000+ messages/sec par instrument
# - Latence < 1ms critique
# - Taille minimale

message MarketData {
  string symbol = 1;
  int64 timestamp = 2;  // Nanoseconds
  double bid_price = 3;
  double ask_price = 4;
  int32 bid_size = 5;
  int32 ask_size = 6;
}

# REST API (historique) : JSON
# - Fréquence faible
# - Développeurs externes

GET /api/quotes/AAPL
{
  "symbol": "AAPL",
  "price": 150.25,
  "change": +2.5,
  "timestamp": "2025-01-15T15:30:00Z"
}

# Order Entry : Protocol Buffers
# - Latence critique
# - Validation stricte
# - Audit trail

message Order {
  int64 order_id = 1;
  string symbol = 2;
  OrderSide side = 3;  // BUY/SELL
  double price = 4;
  int32 quantity = 5;
  int64 timestamp = 6;
}
```

## Optimisations avancées

### 1. Compression

```python
import json
import msgpack
import gzip
import lz4.frame

data = {'users': [{'id': i, 'name': f'User{i}'} for i in range(10000)]}

# JSON brut
json_bytes = json.dumps(data).encode('utf-8')
print(f"JSON : {len(json_bytes):,} bytes")

# JSON + gzip
json_gzip = gzip.compress(json_bytes)
print(f"JSON + gzip : {len(json_gzip):,} bytes")

# JSON + lz4 (plus rapide)
json_lz4 = lz4.frame.compress(json_bytes)
print(f"JSON + lz4 : {len(json_lz4):,} bytes")

# MessagePack
msgpack_bytes = msgpack.packb(data)
print(f"MessagePack : {len(msgpack_bytes):,} bytes")

# MessagePack + lz4
msgpack_lz4 = lz4.frame.compress(msgpack_bytes)
print(f"MessagePack + lz4 : {len(msgpack_lz4):,} bytes")

# Résultats typiques :
# JSON : 890,000 bytes
# JSON + gzip : 120,000 bytes (86% réduction)
# JSON + lz4 : 180,000 bytes (80% réduction, 3× plus rapide)
# MessagePack : 670,000 bytes
# MessagePack + lz4 : 140,000 bytes (79% réduction)

# Conclusion : JSON + compression peut battre MessagePack
# SI le CPU de compression est acceptable
```

### 2. Schéma partiel avec JSON Schema

```python
import json
import jsonschema

# Définir un schéma JSON
user_schema = {
    "type": "object",
    "properties": {
        "user_id": {"type": "integer"},
        "username": {"type": "string", "minLength": 3},
        "email": {"type": "string", "format": "email"},
        "age": {"type": "integer", "minimum": 0, "maximum": 150}
    },
    "required": ["user_id", "username", "email"]
}

# Valider
valid_user = {
    "user_id": 123,
    "username": "alice",
    "email": "alice@example.com",
    "age": 30
}

try:
    jsonschema.validate(valid_user, user_schema)
    print("✓ Valid")
except jsonschema.ValidationError as e:
    print(f"✗ Invalid: {e.message}")

# Tester avec données invalides
invalid_user = {
    "user_id": 123,
    "username": "ab",  # Trop court
    "email": "not-an-email"
}

try:
    jsonschema.validate(invalid_user, user_schema)
except jsonschema.ValidationError as e:
    print(f"✗ Invalid: {e.message}")
    # 'ab' is too short
```

### 3. Lazy deserialization

```python
import json

# Pour de gros documents JSON, désérialiser seulement ce qui est nécessaire
class LazyJSONObject:
    def __init__(self, json_string):
        self._raw = json_string
        self._parsed = None

    def __getitem__(self, key):
        if self._parsed is None:
            # Parse seulement quand accédé
            self._parsed = json.loads(self._raw)
        return self._parsed[key]

# Utilisation
large_json = '{"users": [...10000 users...], "metadata": {...}}'
lazy_obj = LazyJSONObject(large_json)

# Accès seulement à metadata (parse partiel)
print(lazy_obj['metadata'])
# users n'est jamais parsé si pas accédé
```

## Sécurité

### 1. JSON Injection

```python
# DANGER : JSON non validé
user_input = request.GET.get('filter')
query = json.loads(user_input)
# Si user_input = '{"$where": "function() { while(true) {} }"}'
# → DoS possible

# SÉCURITÉ : Valider avec schéma
schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer"}
    },
    "additionalProperties": False
}

try:
    jsonschema.validate(query, schema)
    # Safe to use
except jsonschema.ValidationError:
    return {"error": "Invalid filter"}
```

### 2. Protobuf : Validation automatique

```python
# Protobuf valide automatiquement les types
user = user_pb2.User()
user.user_id = "123"  # ERREUR : attend int32

# TypeError: '123' has type str, but expected one of: int, long
```

### 3. Limites de taille

```python
import json

MAX_SIZE = 1_000_000  # 1 MB

def safe_json_load(data):
    if len(data) > MAX_SIZE:
        raise ValueError("JSON too large")
    return json.loads(data)

# Protection contre JSON bombs
# {"a": {"a": {"a": ... (nested 10000 fois)}}}
```

## Récapitulatif

### Choix rapide

```
Besoin                          → Format recommandé
─────────────────────────────────────────────────
API publique                    → JSON
Debug fréquent                  → JSON
Compatibilité maximale          → JSON
Simplicité prioritaire          → JSON

Performance importante          → MessagePack ou Protobuf
Bande passante limitée          → Protocol Buffers
Haute fréquence (>10k/sec)     → Protocol Buffers
Schéma strict requis           → Protocol Buffers
Microservices internes         → Protocol Buffers ou MessagePack

Upgrade facile de JSON         → MessagePack
Python/Ruby services           → MessagePack
Cache/Redis                    → MessagePack
WebSocket binaire              → MessagePack
```

### Matrice de décision

| Critère | Poids | JSON | MessagePack | Protobuf |
|---------|-------|------|-------------|----------|
| Performance | 20% | 2/5 | 4/5 | 5/5 |
| Taille | 15% | 2/5 | 3/5 | 5/5 |
| Lisibilité | 15% | 5/5 | 1/5 | 1/5 |
| Simplicité | 15% | 5/5 | 5/5 | 2/5 |
| Universalité | 15% | 5/5 | 3/5 | 3/5 |
| Évolutivité | 10% | 2/5 | 2/5 | 5/5 |
| Validation | 10% | 2/5 | 2/5 | 5/5 |
| **Score** | | **3.5** | **3.2** | **3.9** |

### Tendances actuelles

```
2015-2018 : JSON dominant partout
2018-2020 : gRPC + Protobuf gagne en popularité (microservices)
2020-2023 : MessagePack pour Python/Ruby services
2023-2025 : Protobuf pour haute performance, JSON pour public APIs
```

## Prochaines étapes

Maintenant que vous maîtrisez la sérialisation :

- **Section 8.12** : Bibliothèques réseau modernes (aperçu)
- **Section 8.13** : Conception d'applications client-serveur
- **Section 8.14** : Implications réseau pour les microservices

---


⏭️ [Bibliothèques réseau modernes (aperçu)](/08-programmation-reseau/12-bibliotheques-modernes.md)
