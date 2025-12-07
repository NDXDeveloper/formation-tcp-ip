🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.3 Load Balancing : principes et niveaux (L4, L7)

## Introduction

Le **load balancing** (répartition de charge) est l'art de **distribuer intelligemment le trafic réseau** entre plusieurs serveurs pour optimiser les performances, maximiser la disponibilité et éviter la surcharge d'un serveur unique. C'est l'un des piliers fondamentaux des architectures web modernes.

### L'analogie du supermarché

Imaginez un supermarché le samedi après-midi :

**Sans load balancing :**
```
Caisse 1 : 30 personnes qui attendent ⏰⏰⏰
Caisse 2 : Fermée
Caisse 3 : Fermée
Caisse 4 : Fermée

Résultat : Clients mécontents, abandon de panier, mauvaise expérience
```

**Avec load balancing (L4 - round-robin simple) :**
```
Caisse 1 : 8 personnes
Caisse 2 : 7 personnes
Caisse 3 : 8 personnes
Caisse 4 : 7 personnes

Résultat : Temps d'attente divisé par 4, clients satisfaits
```

**Avec load balancing intelligent (L7 - aware context) :**
```
Caisse express (< 10 articles) : 5 personnes → rapide ⚡
Caisse standard : 8 personnes → normal
Caisse prioritaire (personnes âgées) : 3 personnes → service adapté
Caisse automate : 4 personnes → autonome

Résultat : Optimisation maximale selon le profil
```

### Le problème que résout le load balancing

**Scénario typique :** Votre startup décolle. Vous passez de 100 à 10 000 utilisateurs simultanés.

```
Avant (serveur unique)
┌─────────────────────────────────────┐
│  10 000 requêtes/sec                │
│         ↓                           │
│    Serveur unique                   │
│    CPU: 100% 🔥                     │
│    RAM: Saturée 💥                  │
│    Latence: 5000ms 😱               │
│         ↓                           │
│    Site down ☠️                     │
└─────────────────────────────────────┘

Après (avec load balancer)
┌─────────────────────────────────────┐
│  10 000 requêtes/sec                │
│         ↓                           │
│    Load Balancer                    │
│    (distribue intelligemment)       │
│         ↓                           │
│  ┌──────┬──────┬──────┬──────┐      │
│  │ S1   │ S2   │ S3   │ S4   │      │
│  │2500  │2500  │2500  │2500  │      │
│  │req/s │req/s │req/s │req/s │      │
│  │CPU:  │CPU:  │CPU:  │CPU:  │      │
│  │25%✅ │25%✅ │25%✅ │25%✅ │      │
│  │Lat:  │Lat:  │Lat:  │Lat:  │      │
│  │50ms  │50ms  │50ms  │50ms  │      │
│  └──────┴──────┴──────┴──────┘      │
│                                     │
│  Site rapide et stable 🚀           │
└─────────────────────────────────────┘
```

## Types de load balancing : L4 vs L7

La différence fondamentale réside dans la **couche OSI** à laquelle le load balancer opère.

### Rappel du modèle OSI/TCP-IP

```
┌──────────────────────────────────────────────────┐
│  Couche 7 - Application  │ HTTP, HTTPS, DNS      │ ← L7 Load Balancer
│  Couche 6 - Présentation │ SSL/TLS, Encoding     │
│  Couche 5 - Session      │ Sessions              │
├──────────────────────────┼───────────────────────┤
│  Couche 4 - Transport    │ TCP, UDP, Ports       │ ← L4 Load Balancer
├──────────────────────────┼───────────────────────┤
│  Couche 3 - Réseau       │ IP, ICMP, Routing     │
│  Couche 2 - Liaison      │ Ethernet, MAC, Switch │
│  Couche 1 - Physique     │ Câbles, signaux       │
└──────────────────────────────────────────────────┘
```

### Load Balancing L4 (Transport Layer)

Le load balancer L4 opère au niveau **TCP/UDP**. Il prend des décisions basées uniquement sur :
- **Adresse IP source/destination**
- **Port source/destination**
- **Protocole** (TCP ou UDP)

Il **ne regarde PAS** le contenu des paquets (headers HTTP, cookies, etc.).

```
Décision L4 basée sur :
┌─────────────────────────────────┐
│  IP Source : 192.168.1.100      │
│  IP Dest   : 10.0.0.10          │
│  Port Src  : 54321              │
│  Port Dest : 443                │
│  Protocole : TCP                │
└─────────────────────────────────┘
         ↓
   [Décision de routage]
         ↓
   Serveur backend choisi
```

**Caractéristiques :**
- ⚡ **Très rapide** (pas de déchiffrement SSL, pas d'analyse HTTP)
- 📊 **Scalable** (peut gérer des millions de connexions)
- 🔒 **Transparent** (le client ne voit qu'une seule IP)
- ⚠️ **Limité** (pas de routing intelligent basé sur le contenu)

**Cas d'usage :**
- Load balancing de bases de données
- Proxying de connexions TCP brutes
- Situations où la performance brute est critique
- Trafic non-HTTP (SMTP, PostgreSQL, Redis, etc.)

### Load Balancing L7 (Application Layer)

Le load balancer L7 opère au niveau **applicatif** (HTTP/HTTPS). Il peut inspecter :
- **Headers HTTP** (Host, User-Agent, Cookie, etc.)
- **URL et query parameters**
- **Méthode HTTP** (GET, POST, etc.)
- **Contenu du payload** (JSON, XML)
- **Cookies et sessions**

```
Décision L7 basée sur :
┌──────────────────────────────────────────┐
│  GET /api/users/123 HTTP/1.1             │
│  Host: api.example.com                   │
│  User-Agent: Mobile App/2.0              │
│  Cookie: session=abc123                  │
│  X-API-Version: v2                       │
└──────────────────────────────────────────┘
         ↓
   [Décision intelligente]
   • API v2 → Backend pool 2
   • Mobile → Cache-friendly servers
   • Session abc123 → Server 3 (sticky)
         ↓
   Serveur backend optimal
```

**Caractéristiques :**
- 🧠 **Intelligent** (routing basé sur le contenu)
- 🔐 **SSL termination** (déchiffre HTTPS une fois)
- 🎯 **Granulaire** (routing par URL, headers, etc.)
- ⚙️ **Manipulation** (modifier headers, rewrite URLs)
- 🐌 **Plus lent** que L4 (analyse approfondie)

**Cas d'usage :**
- Applications web (HTTP/HTTPS)
- Microservices avec routing complexe
- A/B testing
- Canary deployments
- API gateways

### Comparaison L4 vs L7

| Critère | L4 (Transport) | L7 (Application) |
|---------|----------------|------------------|
| **Couche OSI** | Transport (TCP/UDP) | Application (HTTP/HTTPS) |
| **Visibilité** | IP + Port | Contenu complet (headers, body) |
| **Performance** | Excellent (ns) | Bon (µs-ms) |
| **Throughput** | Très élevé (10M+ conn/s) | Élevé (100k-1M req/s) |
| **Complexité** | Faible | Élevée |
| **Routage** | Round-robin, least conn | URL, headers, cookies, géo |
| **SSL** | Passthrough | Termination possible |
| **Session persistence** | IP-based | Cookie-based |
| **Health checks** | TCP handshake | HTTP status codes |
| **Use cases** | DB, cache, TCP apps | Web apps, APIs, microservices |
| **Exemples** | AWS NLB, HAProxy (mode TCP) | AWS ALB, Nginx, HAProxy (mode HTTP) |

### Illustration : Flux de traitement

**Load Balancer L4 :**
```
Client                  LB L4                Backend
  │                       │                     │
  │───[SYN]──────────────>│                     │
  │                       │──[SYN]─────────────>│
  │                       │<─[SYN-ACK]──────────│
  │<──[SYN-ACK]───────────│                     │
  │───[ACK]──────────────>│───[ACK]────────────>│
  │                       │                     │
  │───[Data (encrypted)]─>│──[Data (encrypted)]>│
  │                       │  (passthrough)      │
  │                       │                     │
         LB ne regarde PAS le contenu
```

**Load Balancer L7 :**
```
Client                  LB L7                Backend
  │                       │                     │
  │───[HTTPS Request]────>│                     │
  │                       │ [Déchiffre SSL]     │
  │                       │ [Analyse HTTP]      │
  │                       │ [Décision routing]  │
  │                       │──[HTTP Request]───> │
  │                       │   (peut modifier)   │
  │                       │<─[HTTP Response]─── │
  │                       │ [Analyse response]  │
  │                       │ [Re-chiffre SSL]    │
  │<──[HTTPS Response]────│                     │
  │                       │                     │
      LB inspecte et peut modifier le contenu
```

## Algorithmes de distribution

Le choix de l'algorithme détermine **comment** les requêtes sont distribuées entre les serveurs.

### 1. Round Robin (RR)

**Principe :** Distribution circulaire, chaque serveur reçoit à tour de rôle.

```
Requête 1 → Serveur 1
Requête 2 → Serveur 2
Requête 3 → Serveur 3
Requête 4 → Serveur 1
Requête 5 → Serveur 2
...
```

**Avantages :**
- Simple à implémenter
- Distribution équitable si tous les serveurs sont identiques

**Inconvénients :**
- Ne tient pas compte de la charge réelle des serveurs
- Inefficace si les serveurs ont des capacités différentes

**Cas d'usage :** Serveurs homogènes, charges uniformes.

**Implémentation simple en Python :**

```python
class RoundRobinLoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.current = 0

    def get_next_server(self):
        server = self.servers[self.current]
        self.current = (self.current + 1) % len(self.servers)
        return server

# Usage
lb = RoundRobinLoadBalancer(['server1:8080', 'server2:8080', 'server3:8080'])

for i in range(10):
    print(f"Request {i+1} → {lb.get_next_server()}")

# Output:
# Request 1 → server1:8080
# Request 2 → server2:8080
# Request 3 → server3:8080
# Request 4 → server1:8080
# Request 5 → server2:8080
# ...
```

### 2. Weighted Round Robin (WRR)

**Principe :** Round robin pondéré selon la capacité des serveurs.

```
Serveur 1 (poids: 3) → reçoit 3 requêtes
Serveur 2 (poids: 2) → reçoit 2 requêtes
Serveur 3 (poids: 1) → reçoit 1 requête

Séquence : S1, S1, S1, S2, S2, S3, S1, S1, S1, ...
```

**Cas d'usage :** Serveurs de capacités différentes (ex: 2×8 cores, 1×16 cores).

**Implémentation :**

```python
class WeightedRoundRobinLoadBalancer:
    def __init__(self, servers_with_weights):
        """
        servers_with_weights: list of (server, weight) tuples
        Example: [('server1', 3), ('server2', 2), ('server3', 1)]
        """
        self.servers = []
        for server, weight in servers_with_weights:
            self.servers.extend([server] * weight)
        self.current = 0

    def get_next_server(self):
        server = self.servers[self.current]
        self.current = (self.current + 1) % len(self.servers)
        return server

# Usage
lb = WeightedRoundRobinLoadBalancer([
    ('powerful-server', 5),   # Serveur puissant
    ('medium-server', 3),     # Serveur moyen
    ('weak-server', 1)        # Serveur faible
])

for i in range(15):
    print(f"Request {i+1} → {lb.get_next_server()}")

# Output distribue 5:3:1
```

### 3. Least Connections (LC)

**Principe :** Envoyer la requête au serveur ayant le **moins de connexions actives**.

```
État actuel :
Serveur 1 : 10 connexions
Serveur 2 : 5 connexions  ← Choisi (minimum)
Serveur 3 : 8 connexions

Nouvelle requête → Serveur 2
```

**Avantages :**
- Adapté aux requêtes de durée variable
- Meilleure distribution que RR pour charges hétérogènes

**Cas d'usage :** Applications avec temps de traitement variables (uploads, long-polling, WebSockets).

**Implémentation :**

```python
import threading
from collections import defaultdict

class LeastConnectionsLoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.connections = defaultdict(int)
        self.lock = threading.Lock()

    def get_next_server(self):
        with self.lock:
            # Trouver le serveur avec le moins de connexions
            server = min(self.servers, key=lambda s: self.connections[s])
            self.connections[server] += 1
            return server

    def release_connection(self, server):
        with self.lock:
            self.connections[server] = max(0, self.connections[server] - 1)

    def get_stats(self):
        return dict(self.connections)

# Usage
lb = LeastConnectionsLoadBalancer(['server1', 'server2', 'server3'])

# Simuler des requêtes
import random
import time

def handle_request(request_id):
    server = lb.get_next_server()
    print(f"Request {request_id} → {server} (active: {lb.get_stats()})")

    # Simuler traitement variable
    time.sleep(random.uniform(0.1, 1.0))

    lb.release_connection(server)
    print(f"Request {request_id} completed on {server}")

# Test avec threads
threads = []
for i in range(10):
    t = threading.Thread(target=handle_request, args=(i,))
    t.start()
    threads.append(t)
    time.sleep(0.1)

for t in threads:
    t.join()
```

### 4. Weighted Least Connections (WLC)

Combine least connections + poids.

```python
class WeightedLeastConnectionsLoadBalancer:
    def __init__(self, servers_with_weights):
        self.servers = {server: weight for server, weight in servers_with_weights}
        self.connections = defaultdict(int)
        self.lock = threading.Lock()

    def get_next_server(self):
        with self.lock:
            # Ratio = connexions_actives / poids
            # Plus le ratio est bas, plus le serveur est disponible
            server = min(self.servers.keys(),
                        key=lambda s: self.connections[s] / self.servers[s])
            self.connections[server] += 1
            return server

    def release_connection(self, server):
        with self.lock:
            self.connections[server] = max(0, self.connections[server] - 1)

# Usage
lb = WeightedLeastConnectionsLoadBalancer([
    ('powerful-server', 10),
    ('medium-server', 5),
    ('weak-server', 2)
])
```

### 5. IP Hash (Source Hash)

**Principe :** Hash de l'IP client détermine le serveur backend.

```
hash(client_ip) % nb_servers = serveur_destination

Client 192.168.1.10 → hash → Serveur 2 (toujours)
Client 192.168.1.20 → hash → Serveur 1 (toujours)
```

**Avantages :**
- **Session persistence** naturelle (même client → même serveur)
- Pas besoin de sticky sessions

**Inconvénients :**
- Redistribution si un serveur tombe
- Mauvaise distribution si peu de clients

**Implémentation :**

```python
import hashlib

class IPHashLoadBalancer:
    def __init__(self, servers):
        self.servers = servers

    def get_server_for_ip(self, client_ip):
        # Hash MD5 de l'IP
        hash_value = int(hashlib.md5(client_ip.encode()).hexdigest(), 16)
        index = hash_value % len(self.servers)
        return self.servers[index]

# Usage
lb = IPHashLoadBalancer(['server1', 'server2', 'server3'])

clients = ['192.168.1.10', '192.168.1.20', '10.0.0.5', '172.16.0.1']

for client in clients:
    server = lb.get_server_for_ip(client)
    print(f"Client {client} → {server}")

# Même client obtient toujours le même serveur
for _ in range(3):
    print(f"Client 192.168.1.10 → {lb.get_server_for_ip('192.168.1.10')}")
```

### 6. Consistent Hashing

**Principe :** Amélioration de IP Hash qui minimise la redistribution quand un serveur est ajouté/retiré.

```
Ring hash :
         server1
           ↓
    ○───────○───────○
    ↑               ↓
  server3        server2
```

**Implémentation :**

```python
import hashlib
import bisect

class ConsistentHashLoadBalancer:
    def __init__(self, servers, replicas=100):
        """
        replicas: nombre de points virtuels par serveur (pour meilleure distribution)
        """
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []

        for server in servers:
            self.add_server(server)

    def _hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)

    def add_server(self, server):
        for i in range(self.replicas):
            # Créer des répliques virtuelles
            key = f"{server}:{i}"
            hash_value = self._hash(key)
            self.ring[hash_value] = server
            bisect.insort(self.sorted_keys, hash_value)

    def remove_server(self, server):
        for i in range(self.replicas):
            key = f"{server}:{i}"
            hash_value = self._hash(key)
            del self.ring[hash_value]
            self.sorted_keys.remove(hash_value)

    def get_server(self, client_key):
        if not self.ring:
            return None

        hash_value = self._hash(client_key)

        # Trouver le premier serveur dans le sens horaire
        index = bisect.bisect(self.sorted_keys, hash_value)

        if index == len(self.sorted_keys):
            index = 0

        return self.ring[self.sorted_keys[index]]

# Usage
lb = ConsistentHashLoadBalancer(['server1', 'server2', 'server3'])

# Tester la distribution
from collections import Counter
distribution = Counter()

for i in range(1000):
    client = f"client-{i}"
    server = lb.get_server(client)
    distribution[server] += 1

print("Distribution avant retrait:")
for server, count in distribution.items():
    print(f"  {server}: {count} requêtes ({count/10:.1f}%)")

# Retirer un serveur
print("\nRetrait de server2...")
lb.remove_server('server2')

redistribution = Counter()
for i in range(1000):
    client = f"client-{i}"
    server = lb.get_server(client)
    redistribution[server] += 1

print("\nDistribution après retrait:")
for server, count in redistribution.items():
    print(f"  {server}: {count} requêtes ({count/10:.1f}%)")

# Calculer le pourcentage de redistribution
changed = 0
for i in range(1000):
    client = f"client-{i}"
    if lb.get_server(client) != (distribution.most_common()[0][0] if i % 3 == 1 else 'server1'):
        changed += 1

print(f"\n{changed/10:.1f}% des requêtes redistribuées (vs 33% avec hash simple)")
```

### 7. Random

Simple mais efficace pour de grands volumes.

```python
import random

class RandomLoadBalancer:
    def __init__(self, servers):
        self.servers = servers

    def get_next_server(self):
        return random.choice(self.servers)
```

### 8. Least Response Time

Combine least connections + latence mesurée.

```python
import time
from collections import defaultdict
import threading

class LeastResponseTimeLoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.response_times = defaultdict(lambda: [])  # Historique des temps
        self.connections = defaultdict(int)
        self.lock = threading.Lock()

    def get_avg_response_time(self, server):
        times = self.response_times[server]
        if not times:
            return 0
        return sum(times[-10:]) / len(times[-10:])  # Moyenne des 10 dernières

    def get_next_server(self):
        with self.lock:
            # Score = connexions_actives * avg_response_time
            server = min(self.servers,
                        key=lambda s: (self.connections[s] + 1) *
                                     (self.get_avg_response_time(s) + 0.001))
            self.connections[server] += 1
            return server

    def record_response(self, server, response_time):
        with self.lock:
            self.response_times[server].append(response_time)
            # Garder seulement les 100 dernières mesures
            if len(self.response_times[server]) > 100:
                self.response_times[server].pop(0)
            self.connections[server] = max(0, self.connections[server] - 1)
```

### Tableau récapitulatif des algorithmes

| Algorithme | Complexité | Cas d'usage | Avantages | Inconvénients |
|------------|-----------|-------------|-----------|---------------|
| **Round Robin** | O(1) | Serveurs identiques | Simple, équitable | Ignore la charge |
| **Weighted RR** | O(1) | Serveurs hétérogènes | Respecte capacités | Statique |
| **Least Connections** | O(n) | Charges variables | Adaptatif | Overhead tracking |
| **IP Hash** | O(1) | Session persistence | Sticky naturel | Redistribution problématique |
| **Consistent Hash** | O(log n) | Elastic scaling | Minimise redistribution | Complexe |
| **Random** | O(1) | Grand volume | Simple, scalable | Inéquitable à court terme |
| **Least Response Time** | O(n) | Performance critique | Optimal | Overhead mesures |

## Health Checks

Les health checks permettent de **détecter les serveurs défaillants** et de ne plus leur envoyer de trafic.

### Types de health checks

#### 1. TCP Health Check (L4)

Vérifie simplement si le port est ouvert.

```python
import socket

def tcp_health_check(host, port, timeout=2):
    """
    Health check TCP simple
    Retourne True si le port répond
    """
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        result = sock.connect_ex((host, port))
        sock.close()
        return result == 0
    except Exception as e:
        print(f"Health check failed: {e}")
        return False

# Usage
if tcp_health_check('backend1.example.com', 8080):
    print("✅ Server is healthy")
else:
    print("❌ Server is down")
```

#### 2. HTTP Health Check (L7)

Appelle un endpoint spécifique et vérifie le status code.

```python
import requests
from datetime import datetime

class HTTPHealthChecker:
    def __init__(self, servers, health_path='/health', interval=5):
        self.servers = servers
        self.health_path = health_path
        self.interval = interval
        self.health_status = {server: True for server in servers}
        self.last_check = {}

    def check_server(self, server):
        """
        Vérifie la santé d'un serveur via HTTP
        """
        url = f"http://{server}{self.health_path}"

        try:
            response = requests.get(url, timeout=2)

            # Critères de santé
            is_healthy = (
                response.status_code == 200 and
                response.elapsed.total_seconds() < 1.0  # Latence < 1s
            )

            self.health_status[server] = is_healthy
            self.last_check[server] = datetime.now()

            return is_healthy

        except Exception as e:
            print(f"❌ {server} health check failed: {e}")
            self.health_status[server] = False
            self.last_check[server] = datetime.now()
            return False

    def get_healthy_servers(self):
        """Retourne la liste des serveurs en bonne santé"""
        return [s for s in self.servers if self.health_status.get(s, False)]

    def run_checks(self):
        """Exécute les health checks sur tous les serveurs"""
        for server in self.servers:
            status = "✅" if self.check_server(server) else "❌"
            print(f"{status} {server} - Last check: {self.last_check[server]}")

# Usage
checker = HTTPHealthChecker([
    'backend1:8080',
    'backend2:8080',
    'backend3:8080'
])

# Check unique
checker.run_checks()

# Obtenir les serveurs sains
healthy = checker.get_healthy_servers()
print(f"\nHealthy servers: {healthy}")
```

#### 3. Application-Aware Health Check

Vérifie non seulement que le serveur répond, mais qu'il est **fonctionnellement opérationnel**.

```python
import requests
import json

def advanced_health_check(server):
    """
    Health check avancé qui vérifie :
    - La connectivité HTTP
    - La connexion à la base de données
    - L'état du cache
    - La mémoire disponible
    """
    url = f"http://{server}/api/health/detailed"

    try:
        response = requests.get(url, timeout=3)

        if response.status_code != 200:
            return False, "HTTP error"

        health_data = response.json()

        # Vérifier chaque composant
        checks = {
            'database': health_data.get('database', {}).get('connected', False),
            'cache': health_data.get('cache', {}).get('connected', False),
            'memory': health_data.get('memory', {}).get('available_mb', 0) > 100,
            'cpu': health_data.get('cpu', {}).get('load', 100) < 80
        }

        all_healthy = all(checks.values())

        if not all_healthy:
            failing = [k for k, v in checks.items() if not v]
            return False, f"Failing components: {failing}"

        return True, "All systems operational"

    except Exception as e:
        return False, str(e)

# Endpoint /api/health/detailed côté serveur (Flask)
from flask import Flask, jsonify
import psutil

app = Flask(__name__)

@app.route('/api/health/detailed')
def detailed_health():
    """
    Endpoint de health check détaillé
    """
    # Vérifier la DB
    try:
        # db.session.execute('SELECT 1')
        db_connected = True
    except:
        db_connected = False

    # Vérifier le cache
    try:
        # redis_client.ping()
        cache_connected = True
    except:
        cache_connected = False

    # Métriques système
    memory = psutil.virtual_memory()
    cpu = psutil.cpu_percent(interval=0.1)

    health_status = {
        'status': 'healthy' if (db_connected and cache_connected) else 'degraded',
        'database': {
            'connected': db_connected,
            'latency_ms': 5  # Mesure réelle
        },
        'cache': {
            'connected': cache_connected,
            'hit_rate': 0.85
        },
        'memory': {
            'available_mb': memory.available / (1024 * 1024),
            'percent_used': memory.percent
        },
        'cpu': {
            'load': cpu
        },
        'timestamp': datetime.now().isoformat()
    }

    status_code = 200 if health_status['status'] == 'healthy' else 503

    return jsonify(health_status), status_code
```

### Load Balancer avec health checks intégrés

```python
import threading
import time
from typing import List, Dict

class SmartLoadBalancer:
    """
    Load balancer avec health checks automatiques
    """

    def __init__(self, servers: List[str], health_check_interval=5):
        self.all_servers = servers
        self.healthy_servers = servers.copy()
        self.health_check_interval = health_check_interval
        self.current_index = 0
        self.lock = threading.Lock()

        # Démarrer le thread de health checks
        self.health_checker_thread = threading.Thread(
            target=self._health_check_loop,
            daemon=True
        )
        self.health_checker_thread.start()

    def _health_check_loop(self):
        """Boucle de health checks en arrière-plan"""
        while True:
            self._run_health_checks()
            time.sleep(self.health_check_interval)

    def _run_health_checks(self):
        """Exécute les health checks sur tous les serveurs"""
        healthy = []

        for server in self.all_servers:
            if self._check_server_health(server):
                healthy.append(server)
                print(f"✅ {server} is healthy")
            else:
                print(f"❌ {server} is down")

        with self.lock:
            previous = set(self.healthy_servers)
            current = set(healthy)

            # Détecter les changements
            newly_down = previous - current
            newly_up = current - previous

            if newly_down:
                print(f"⚠️  Servers went down: {newly_down}")
            if newly_up:
                print(f"✅ Servers recovered: {newly_up}")

            self.healthy_servers = healthy

    def _check_server_health(self, server):
        """Health check d'un serveur (simplifié)"""
        return tcp_health_check(server.split(':')[0], int(server.split(':')[1]))

    def get_next_server(self):
        """Obtenir le prochain serveur sain (round-robin)"""
        with self.lock:
            if not self.healthy_servers:
                raise Exception("No healthy servers available")

            server = self.healthy_servers[self.current_index]
            self.current_index = (self.current_index + 1) % len(self.healthy_servers)
            return server

    def get_stats(self):
        """Obtenir les statistiques du load balancer"""
        with self.lock:
            return {
                'total_servers': len(self.all_servers),
                'healthy_servers': len(self.healthy_servers),
                'unhealthy_servers': len(self.all_servers) - len(self.healthy_servers),
                'healthy_list': self.healthy_servers
            }

# Usage
lb = SmartLoadBalancer([
    '192.168.1.10:8080',
    '192.168.1.11:8080',
    '192.168.1.12:8080'
], health_check_interval=10)

# Le load balancer vérifie automatiquement la santé en arrière-plan
time.sleep(2)

# Obtenir le prochain serveur
for i in range(5):
    try:
        server = lb.get_next_server()
        print(f"Request {i+1} → {server}")
    except Exception as e:
        print(f"Error: {e}")
    time.sleep(1)

# Statistiques
print(lb.get_stats())
```

## Session Persistence (Sticky Sessions)

La **session persistence** garantit que les requêtes d'un même client vont toujours au même serveur backend.

### Pourquoi c'est nécessaire ?

```
Problème sans sticky sessions :
┌────────────────────────────────────┐
│ User fait LOGIN                    │
│   ↓                                │
│ LB → Server 1 (crée session)       │
│ Session stockée en RAM sur Server 1│
│                                    │
│ User fait GET /profile             │
│   ↓                                │
│ LB → Server 2 (round-robin)        │
│ ❌ Server 2 n'a pas la session     │
│ → User redirigé vers login         │
│ → Mauvaise expérience              │
└────────────────────────────────────┘
```

### Solutions

#### 1. Cookie-based sticky sessions (L7)

Le load balancer injecte un cookie avec l'ID du serveur.

```python
from flask import Flask, request, make_response
import hashlib

app = Flask(__name__)

class CookieBasedLoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.cookie_name = 'LB_SERVER_ID'

    def get_server_for_request(self, request):
        """
        Détermine le serveur basé sur le cookie
        Si pas de cookie, assigne un serveur et crée le cookie
        """
        server_id = request.cookies.get(self.cookie_name)

        if server_id and server_id in self.servers:
            return server_id

        # Nouveau client → assigner un serveur (round-robin, random, etc.)
        server = self.choose_new_server(request)
        return server

    def choose_new_server(self, request):
        # Simple round-robin ou autre algorithme
        # En pratique, utiliser least connections
        import random
        return random.choice(self.servers)

    def inject_cookie(self, response, server_id):
        """Injecte le cookie de persistence"""
        response.set_cookie(
            self.cookie_name,
            server_id,
            max_age=3600,  # 1 heure
            httponly=True,  # Sécurité
            secure=True,    # HTTPS uniquement
            samesite='Lax'
        )
        return response

# Dans le reverse proxy
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
def proxy(path):
    lb = CookieBasedLoadBalancer(['server1', 'server2', 'server3'])

    # Déterminer le serveur
    server = lb.get_server_for_request(request)

    # Proxier la requête (en pratique, utiliser requests)
    # response = requests.request(...)

    response = make_response(f"Proxied to {server}")

    # Injecter le cookie de persistence
    lb.inject_cookie(response, server)

    return response
```

#### 2. IP-based sticky sessions (L4)

Hash de l'IP client (déjà vu dans les algorithmes).

```python
# HAProxy config pour sticky sessions via cookie
"""
backend web_servers
    balance roundrobin
    cookie SERVERID insert indirect nocache
    server web1 192.168.1.10:80 check cookie web1
    server web2 192.168.1.11:80 check cookie web2
    server web3 192.168.1.12:80 check cookie web3
"""

# HAProxy config pour sticky sessions via IP
"""
backend web_servers
    balance source  # IP hash
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
    server web3 192.168.1.12:80 check
"""
```

#### 3. Session store externe (meilleure solution)

Stocker les sessions dans Redis/Memcached au lieu de la RAM locale.

```python
from flask import Flask, session
from flask_session import Session
import redis

app = Flask(__name__)

# Configuration de session externe
app.config['SESSION_TYPE'] = 'redis'
app.config['SESSION_REDIS'] = redis.from_url('redis://localhost:6379')
app.config['SESSION_PERMANENT'] = False
app.config['SESSION_USE_SIGNER'] = True
app.config['SESSION_KEY_PREFIX'] = 'session:'

Session(app)

@app.route('/login', methods=['POST'])
def login():
    # La session est stockée dans Redis
    # Accessible depuis TOUS les serveurs backend
    session['user_id'] = 123
    session['username'] = 'alice'
    return {'message': 'Logged in'}

@app.route('/profile')
def profile():
    # Fonctionne peu importe le serveur qui traite la requête
    user_id = session.get('user_id')
    if not user_id:
        return {'error': 'Not logged in'}, 401

    return {'user_id': user_id, 'username': session['username']}
```

**Architecture avec session store :**

```
Client
  ↓
Load Balancer (pas de sticky sessions nécessaire)
  ↓
┌─────────┬─────────┬─────────┐
│ Server1 │ Server2 │ Server3 │
└────┬────┴────┬────┴────┬────┘
     └────┬────┴─────┬───┘
          ↓          ↓
    ┌─────────────────────┐
    │   Redis Cluster     │
    │  (session store)    │
    └─────────────────────┘

Avantages :
✅ Pas de sticky sessions nécessaire
✅ Scale horizontalement
✅ Survit au redémarrage des serveurs
✅ Partage de session entre microservices
```

## Configurations réelles

### HAProxy (L4 et L7)

**Configuration L4 (TCP mode) :**

```haproxy
# /etc/haproxy/haproxy.cfg

global
    log /dev/log local0
    maxconn 4096
    daemon

defaults
    log     global
    mode    tcp          # Mode L4
    option  tcplog
    timeout connect 5s
    timeout client  50s
    timeout server  50s

# Frontend L4 (PostgreSQL)
frontend postgres_frontend
    bind *:5432
    mode tcp
    default_backend postgres_servers

backend postgres_servers
    mode tcp
    balance leastconn    # Least connections pour DB
    option tcp-check

    server pg1 192.168.1.10:5432 check
    server pg2 192.168.1.11:5432 check backup  # Backup (failover)
    server pg3 192.168.1.12:5432 check backup

# Frontend L4 (Redis)
frontend redis_frontend
    bind *:6379
    mode tcp
    default_backend redis_servers

backend redis_servers
    mode tcp
    balance source       # IP hash pour persistence

    server redis1 192.168.1.20:6379 check
    server redis2 192.168.1.21:6379 check
    server redis3 192.168.1.22:6379 check
```

**Configuration L7 (HTTP mode) :**

```haproxy
# Mode HTTP avec fonctionnalités L7

global
    log /dev/log local0
    maxconn 10000
    daemon

    # SSL
    tune.ssl.default-dh-param 2048

defaults
    log     global
    mode    http         # Mode L7
    option  httplog
    option  dontlognull
    timeout connect 5s
    timeout client  30s
    timeout server  30s

# Stats page
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 5s
    stats admin if TRUE

# Frontend HTTPS
frontend https_frontend
    bind *:443 ssl crt /etc/ssl/certs/example.com.pem

    # Security headers
    http-response set-header Strict-Transport-Security "max-age=31536000"
    http-response set-header X-Frame-Options "SAMEORIGIN"
    http-response set-header X-Content-Type-Options "nosniff"

    # ACLs pour routing intelligent
    acl is_api path_beg /api/
    acl is_static path_beg /static/ /assets/ /images/
    acl is_admin hdr(host) -i admin.example.com
    acl is_mobile hdr(User-Agent) -i -m sub mobile android ios

    # Routing basé sur les ACLs
    use_backend api_servers if is_api
    use_backend static_servers if is_static
    use_backend admin_servers if is_admin
    use_backend mobile_servers if is_mobile
    default_backend web_servers

# Backend API (microservices)
backend api_servers
    balance leastconn
    option httpchk GET /health HTTP/1.1\r\nHost:\ api.example.com
    http-check expect status 200

    # Sticky sessions via cookie
    cookie SERVERID insert indirect nocache

    server api1 10.0.1.10:8080 check cookie api1 weight 10
    server api2 10.0.1.11:8080 check cookie api2 weight 10
    server api3 10.0.1.12:8080 check cookie api3 weight 5  # Moins puissant

# Backend static (CDN origin)
backend static_servers
    balance roundrobin
    option httpchk HEAD / HTTP/1.1\r\nHost:\ static.example.com

    # Compression
    compression algo gzip
    compression type text/html text/css text/javascript application/javascript

    server static1 10.0.2.10:80 check
    server static2 10.0.2.11:80 check

# Backend web (application principale)
backend web_servers
    balance source    # IP hash
    option httpchk GET /healthz HTTP/1.1\r\nHost:\ www.example.com
    http-check expect status 200

    # Timeouts spécifiques
    timeout server 60s

    server web1 10.0.3.10:80 check maxconn 500
    server web2 10.0.3.11:80 check maxconn 500
    server web3 10.0.3.12:80 check maxconn 500

# Backend admin (interface admin)
backend admin_servers
    balance leastconn
    option httpchk GET /admin/health

    # Restriction IP (security layer)
    acl allowed_ips src 192.168.1.0/24 10.0.0.0/8
    http-request deny if !allowed_ips

    server admin1 10.0.4.10:8080 check
    server admin2 10.0.4.11:8080 check backup

# Backend mobile
backend mobile_servers
    balance leastconn

    # Optimisations mobile
    timeout server 120s  # Timeout plus long (réseau mobile)

    server mobile1 10.0.5.10:8080 check
    server mobile2 10.0.5.11:8080 check
```

### Nginx (L7 uniquement)

```nginx
# /etc/nginx/nginx.conf

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 4096;
    use epoll;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'backend=$upstream_addr rt=$request_time';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_types text/plain text/css application/json application/javascript text/xml;

    # Upstream definitions (backend pools)

    # API servers (least connections)
    upstream api_backend {
        least_conn;

        server 10.0.1.10:8080 max_fails=3 fail_timeout=30s;
        server 10.0.1.11:8080 max_fails=3 fail_timeout=30s;
        server 10.0.1.12:8080 max_fails=3 fail_timeout=30s;

        # Keepalive connections
        keepalive 32;
    }

    # Web servers (IP hash pour sticky sessions)
    upstream web_backend {
        ip_hash;

        server 10.0.2.10:80 weight=3;
        server 10.0.2.11:80 weight=2;
        server 10.0.2.12:80 weight=1;
    }

    # Static servers (round robin)
    upstream static_backend {
        server 10.0.3.10:80;
        server 10.0.3.11:80;
        server 10.0.3.12:80;
    }

    # Health check (Nginx Plus feature, ou utiliser un module tiers)
    # upstream api_backend {
    #     zone api_backend 64k;
    #     server 10.0.1.10:8080 max_fails=3 fail_timeout=30s;
    #     health_check interval=5s fails=2 passes=2 uri=/health;
    # }

    # Main server block
    server {
        listen 80;
        listen [::]:80;
        server_name example.com www.example.com;

        # Redirect HTTP to HTTPS
        return 301 https://$server_name$request_uri;
    }

    # HTTPS server
    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name example.com www.example.com;

        # SSL configuration
        ssl_certificate /etc/ssl/certs/example.com.crt;
        ssl_certificate_key /etc/ssl/private/example.com.key;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Security headers
        add_header Strict-Transport-Security "max-age=31536000" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;

        # API routes
        location /api/ {
            proxy_pass http://api_backend;

            # Headers
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # Timeouts
            proxy_connect_timeout 5s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;

            # Buffering
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 4k;

            # Health check via proxy
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        }

        # Static content
        location /static/ {
            proxy_pass http://static_backend;

            # Caching
            proxy_cache_valid 200 1h;
            proxy_cache_key "$scheme$request_method$host$request_uri";
            expires 1h;
            add_header Cache-Control "public, immutable";
        }

        # Application
        location / {
            proxy_pass http://web_backend;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # WebSocket support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }

        # Health check endpoint (public)
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }

    # Admin server (separate domain)
    server {
        listen 443 ssl http2;
        server_name admin.example.com;

        ssl_certificate /etc/ssl/certs/admin.example.com.crt;
        ssl_certificate_key /etc/ssl/private/admin.example.com.key;

        # IP restriction
        allow 192.168.1.0/24;
        allow 10.0.0.0/8;
        deny all;

        location / {
            proxy_pass http://admin_backend;
            proxy_set_header Host $host;
        }
    }
}

# Stream block pour L4 load balancing (Nginx Plus ou module)
stream {
    # PostgreSQL load balancing
    upstream postgres {
        least_conn;
        server 10.0.10.10:5432 max_fails=3 fail_timeout=30s;
        server 10.0.10.11:5432 max_fails=3 fail_timeout=30s;
    }

    server {
        listen 5432;
        proxy_pass postgres;
        proxy_connect_timeout 1s;
    }
}
```

## Load Balancing dans Kubernetes

Kubernetes offre plusieurs niveaux de load balancing.

### 1. Service ClusterIP (interne)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: ClusterIP  # Par défaut
  selector:
    app: api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: myapi:v1
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

**Fonctionnement :**
- kube-proxy crée des règles iptables
- Load balancing round-robin entre les pods
- Interne au cluster seulement

### 2. Service NodePort (externe basique)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080  # Port exposé sur chaque nœud (30000-32767)
```

**Accès :** `http://<ANY_NODE_IP>:30080`

### 3. Service LoadBalancer (cloud)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  # AWS NLB
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 443
    targetPort: 8443
  # Cloud provider provisionne automatiquement un LB externe
```

### 4. Ingress (L7 intelligent)

```yaml
# Ingress Controller (Nginx)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: main-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"  # 100 req/s
    nginx.ingress.kubernetes.io/load-balance: "least_conn"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - example.com
    secretName: example-com-tls

  rules:
  # Routing basé sur le hostname
  - host: api.example.com
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: api-v1-service
            port:
              number: 80
      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: api-v2-service
            port:
              number: 80

  # Frontend
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80

  # Static content
  - host: static.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: static-service
            port:
              number: 80
---
# Service pour API v1
apiVersion: v1
kind: Service
metadata:
  name: api-v1-service
spec:
  selector:
    app: api
    version: v1
  ports:
  - port: 80
    targetPort: 8080
---
# Service pour API v2
apiVersion: v1
kind: Service
metadata:
  name: api-v2-service
spec:
  selector:
    app: api
    version: v2
  ports:
  - port: 80
    targetPort: 8080
```

### 5. Canary Deployment avec Ingress

```yaml
# Production (90% du trafic)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-production
  annotations:
    nginx.ingress.kubernetes.io/canary: "false"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-stable
            port:
              number: 80
---
# Canary (10% du trafic)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"  # 10% du trafic
    # Ou basé sur header:
    # nginx.ingress.kubernetes.io/canary-by-header: "X-Canary"
    # nginx.ingress.kubernetes.io/canary-by-header-value: "true"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-canary
            port:
              number: 80
```

## Cloud Load Balancers

### AWS

```python
import boto3

elb = boto3.client('elbv2', region_name='us-east-1')

# Créer un Application Load Balancer (L7)
response = elb.create_load_balancer(
    Name='my-api-alb',
    Subnets=['subnet-12345', 'subnet-67890'],
    SecurityGroups=['sg-abc123'],
    Scheme='internet-facing',
    Type='application',  # 'application' (L7) ou 'network' (L4)
    IpAddressType='ipv4',
    Tags=[
        {'Key': 'Environment', 'Value': 'production'},
        {'Key': 'Application', 'Value': 'api'}
    ]
)

lb_arn = response['LoadBalancers'][0]['LoadBalancerArn']

# Créer un Target Group
tg_response = elb.create_target_group(
    Name='api-targets',
    Protocol='HTTP',
    Port=8080,
    VpcId='vpc-123456',
    HealthCheckProtocol='HTTP',
    HealthCheckPath='/health',
    HealthCheckIntervalSeconds=30,
    HealthCheckTimeoutSeconds=5,
    HealthyThresholdCount=2,
    UnhealthyThresholdCount=3,
    Matcher={'HttpCode': '200'}
)

tg_arn = tg_response['TargetGroups'][0]['TargetGroupArn']

# Enregistrer des targets (instances EC2)
elb.register_targets(
    TargetGroupArn=tg_arn,
    Targets=[
        {'Id': 'i-instance1'},
        {'Id': 'i-instance2'},
        {'Id': 'i-instance3'}
    ]
)

# Créer un Listener (HTTPS)
elb.create_listener(
    LoadBalancerArn=lb_arn,
    Protocol='HTTPS',
    Port=443,
    Certificates=[{'CertificateArn': 'arn:aws:acm:us-east-1:...'}],
    DefaultActions=[{
        'Type': 'forward',
        'TargetGroupArn': tg_arn
    }]
)

# Créer des règles de routing (L7)
elb.create_rule(
    ListenerArn=listener_arn,
    Conditions=[
        {
            'Field': 'path-pattern',
            'Values': ['/api/v2/*']
        }
    ],
    Priority=10,
    Actions=[{
        'Type': 'forward',
        'TargetGroupArn': api_v2_tg_arn
    }]
)
```

### GCP

```python
from google.cloud import compute_v1

# Créer un Backend Service
backend_service = compute_v1.BackendService()
backend_service.name = "api-backend-service"
backend_service.protocol = "HTTP"
backend_service.port_name = "http"
backend_service.timeout_sec = 30
backend_service.load_balancing_scheme = "EXTERNAL"

# Health check
backend_service.health_checks = ["global/healthChecks/api-health-check"]

# Backends (instance groups)
backend = compute_v1.Backend()
backend.group = "zones/us-central1-a/instanceGroups/api-group"
backend.balancing_mode = "UTILIZATION"
backend.max_utilization = 0.8
backend.capacity_scaler = 1.0

backend_service.backends = [backend]

# Créer
client = compute_v1.BackendServicesClient()
operation = client.insert(
    project="my-project",
    backend_service_resource=backend_service
)
```

## Métriques et Monitoring

### Métriques clés d'un load balancer

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Requêtes par backend
backend_requests = Counter(
    'lb_backend_requests_total',
    'Total requests to backend',
    ['backend', 'method', 'status']
)

# Latence par backend
backend_latency = Histogram(
    'lb_backend_latency_seconds',
    'Backend response time',
    ['backend'],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 2.0, 5.0]
)

# Connexions actives
active_connections = Gauge(
    'lb_active_connections',
    'Number of active connections',
    ['backend']
)

# Backends sains
healthy_backends = Gauge(
    'lb_healthy_backends_total',
    'Number of healthy backends'
)

# Taux d'erreur
error_rate = Counter(
    'lb_errors_total',
    'Total errors',
    ['backend', 'error_type']
)

# Utilisation dans le load balancer
class MonitoredLoadBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.healthy = {s: True for s in servers}

    def handle_request(self, request):
        # Choisir le backend
        backend = self.get_next_server()

        # Métriques
        active_connections.labels(backend=backend).inc()

        start_time = time.time()

        try:
            # Proxier la requête
            response = self.proxy_to_backend(backend, request)

            # Enregistrer succès
            backend_requests.labels(
                backend=backend,
                method=request.method,
                status=response.status_code
            ).inc()

            return response

        except Exception as e:
            # Enregistrer erreur
            error_rate.labels(
                backend=backend,
                error_type=type(e).__name__
            ).inc()
            raise

        finally:
            # Latence
            duration = time.time() - start_time
            backend_latency.labels(backend=backend).observe(duration)

            # Décrémenter connexions actives
            active_connections.labels(backend=backend).dec()
```

### Dashboard Grafana (Prometheus queries)

```yaml
# Queries pour dashboard

# Taux de requêtes par backend
rate(lb_backend_requests_total[5m])

# Latence P95 par backend
histogram_quantile(0.95,
  sum(rate(lb_backend_latency_seconds_bucket[5m])) by (backend, le)
)

# Taux d'erreur
rate(lb_errors_total[5m]) / rate(lb_backend_requests_total[5m])

# Distribution du trafic
sum(rate(lb_backend_requests_total[5m])) by (backend)

# Backends sains vs total
lb_healthy_backends_total / count(lb_backend_requests_total)
```

## Pièges courants et solutions

### Piège 1 : Sticky sessions casse le load balancing

**Symptôme :** Un serveur reçoit beaucoup plus de trafic que les autres.

**Cause :** Sticky sessions + distribution inégale des utilisateurs.

**Solution :**
```python
# Utiliser consistent hashing au lieu de sticky sessions
# OU utiliser session store externe (Redis)
# OU limiter la durée des sticky sessions
```

### Piège 2 : Health checks trop agressifs

**Symptôme :** Serveurs marqués down alors qu'ils sont opérationnels.

**Cause :** Timeouts trop courts, seuil d'échecs trop bas.

**Solution :**
```
# HAProxy - configuration raisonnable
option httpchk GET /health
http-check expect status 200
server web1 10.0.0.10:80 check inter 5s rise 2 fall 3

# inter 5s : vérifier toutes les 5s
# rise 2 : 2 checks OK pour marquer UP
# fall 3 : 3 checks KO pour marquer DOWN
```

### Piège 3 : Pas de drain avant shutdown

**Symptôme :** Requêtes échouent lors du déploiement.

**Solution :**
```python
# Graceful shutdown avec drain period

import signal
import time

class GracefulShutdown:
    def __init__(self, drain_period=30):
        self.drain_period = drain_period
        self.shutting_down = False

        signal.signal(signal.SIGTERM, self.handle_sigterm)

    def handle_sigterm(self, signum, frame):
        print(f"SIGTERM received, draining for {self.drain_period}s...")
        self.shutting_down = True

        # Retourner 503 au health check
        # pour que le LB arrête d'envoyer du trafic

        time.sleep(self.drain_period)

        # Maintenant, shutdown
        print("Drain complete, shutting down")
        exit(0)
```

### Piège 4 : Connection pooling mal configuré

**Symptôme :** Nouvelles connexions lentes alors que le serveur n'est pas chargé.

**Solution :**
```python
# Nginx - keepalive connections vers backend
upstream backend {
    server 10.0.0.10:8080;
    keepalive 32;  # Garder 32 connexions idle
}

server {
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;  # Nécessaire pour keepalive
        proxy_set_header Connection "";
    }
}
```

## Résumé : Choix du load balancer

### Checklist de décision

| Besoin | Solution recommandée |
|--------|---------------------|
| **Load balancing DB/Cache** | HAProxy L4, AWS NLB |
| **Load balancing HTTP simple** | Nginx, HAProxy L7 |
| **Routing complexe (microservices)** | Traefik, Kong, AWS ALB |
| **Kubernetes** | Ingress (Nginx/Traefik) |
| **Global traffic** | Cloudflare, AWS Global Accelerator |
| **Performance maximale** | Envoy, HAProxy |
| **Simplicité** | Nginx |

### L4 vs L7 : Quand choisir quoi ?

**Utilisez L4 si :**
- Performance brute critique (millions de connexions/s)
- Trafic non-HTTP (DB, cache, custom protocol)
- Pas besoin de routing intelligent
- SSL passthrough voulu

**Utilisez L7 si :**
- Besoin de routing basé sur URL/headers
- SSL termination centralisée
- A/B testing, canary deployments
- Application-aware health checks
- Manipulation de headers/body

## Conclusion

Le load balancing est **fondamental** pour toute application moderne nécessitant scalabilité et haute disponibilité.

**À retenir :**
- **L4 = rapide** mais simple (IP + port)
- **L7 = intelligent** mais plus lent (contenu HTTP)
- Choisir l'**algorithme** selon le cas d'usage
- Implémenter des **health checks** robustes
- Éviter sticky sessions si possible (**session store** externe)
- Monitorer les **métriques** clés
- Tester le **failover** régulièrement

**Prochaine étape :** La section suivante explore les **Proxys et Reverse Proxys**, qui complètent le load balancing avec des fonctionnalités de cache, sécurité et transformation de requêtes.

---


⏭️ [Proxys et reverse proxys](/09-architectures-avancees/04-proxys-reverse-proxys.md)
