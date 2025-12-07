🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.14 Implications réseau pour les microservices

## Introduction

Les **microservices** transforment un monolithe en dizaines ou centaines de services indépendants qui communiquent par le réseau. Cette architecture apporte de nombreux avantages (scalabilité, indépendance technologique, déploiements indépendants), mais introduit une **complexité réseau considérable**.

```
Monolithe :                    Microservices :
┌──────────────┐              ┌─────┐   ┌─────┐   ┌─────┐
│              │              │Auth │◄─►│User │◄─►│Order│
│  Application │              └─────┘   └─────┘   └─────┘
│              │                 ▲         ▲         ▲
│  (In-memory  │                 │         │         │
│   function   │                 └────►┌───┴───┐◄────┘
│   calls)     │                       │Payment│
│              │                       └───────┘
└──────────────┘
                              Tout passe par le réseau !
```

Dans un monolithe, `userService.getUser(123)` est un appel de fonction en mémoire (~1 nanoseconde). Dans une architecture microservices, cet appel devient une requête HTTP sur le réseau (~10 millisecondes) - **10 millions de fois plus lent** !

Cette section explore les implications réseau et les patterns pour construire des systèmes microservices robustes.

## Le défi réseau fondamental

### Fallacies of Distributed Computing

En 1994, Peter Deutsch a listé les **8 hypothèses fausses** que les développeurs font sur les systèmes distribués :

1. **Le réseau est fiable** ❌
   - Les paquets se perdent, les connexions tombent

2. **La latence est zéro** ❌
   - Chaque appel réseau prend du temps (1-100ms)

3. **La bande passante est infinie** ❌
   - Limite physique, congestion possible

4. **Le réseau est sécurisé** ❌
   - Man-in-the-middle, eavesdropping

5. **La topologie ne change pas** ❌
   - Services démarrent/arrêtent, machines tombent

6. **Il y a un seul administrateur** ❌
   - Multiples équipes, clouds, providers

7. **Le coût de transport est zéro** ❌
   - Bande passante = argent

8. **Le réseau est homogène** ❌
   - Différents protocoles, versions, latences

**Implications** : Chaque appel réseau peut échouer, être lent, ou retourner des données incohérentes.

### Comparaison Monolithe vs Microservices

```python
# MONOLITHE : Appel en mémoire
class UserService:
    def get_user(self, user_id):
        # Accès direct DB
        return db.query(User).get(user_id)

class OrderService:
    def create_order(self, user_id, items):
        # Appel fonction en mémoire
        user = user_service.get_user(user_id)  # < 1µs

        if not user:
            raise ValueError("User not found")

        order = Order(user_id=user_id, items=items)
        db.add(order)
        return order

# MICROSERVICES : Appel réseau
class OrderService:
    def create_order(self, user_id, items):
        # Appel HTTP à un autre service
        try:
            response = requests.get(
                f'http://user-service/users/{user_id}',
                timeout=2.0  # ~10ms en moyenne
            )
            response.raise_for_status()
            user = response.json()
        except requests.Timeout:
            # Réseau peut être lent
            raise ServiceTimeout("User service timeout")
        except requests.RequestException:
            # Service peut être down
            raise ServiceUnavailable("User service unavailable")

        if not user:
            raise ValueError("User not found")

        order = Order(user_id=user_id, items=items)
        db.add(order)
        return order

# Résultat :
# Monolithe : 1µs, toujours réussit
# Microservices : 10ms, peut échouer
```

### Calcul de latence cumulée

```python
# Exemple : Page dashboard utilisateur
# Monolithe : 5 appels fonction en mémoire = 5µs total

# Microservices : 5 appels réseau
latencies = {
    'user_service': 10,      # ms
    'order_service': 15,     # ms
    'payment_service': 20,   # ms
    'notification_service': 8, # ms
    'analytics_service': 12  # ms
}

# Séquentiel
total_sequential = sum(latencies.values())  # 65ms

# Parallèle (best case)
total_parallel = max(latencies.values())  # 20ms

# Dans la réalité :
# - Network overhead
# - Serialization/deserialization
# - Service processing time
# → Ajouter 30-50% : ~90ms séquentiel, ~30ms parallèle

print(f"Monolithe : 5µs")
print(f"Microservices (séquentiel) : {total_sequential}ms = {total_sequential*1000}µs")
print(f"Microservices (parallèle) : {total_parallel}ms = {total_parallel*1000}µs")

# Microservices sont 4000-18000× plus lents !
```

**Conclusion** : Le réseau est le goulot d'étranglement majeur des microservices.

## Service Discovery

Dans une architecture microservices, les services doivent se **découvrir** dynamiquement car :
- Les adresses IP changent (containers, auto-scaling)
- Plusieurs instances par service (load balancing)
- Services démarrent/arrêtent constamment

### 1. Service Discovery côté client

```python
# Consul, Etcd, ZooKeeper
import consul

class ServiceDiscovery:
    def __init__(self):
        self.consul = consul.Consul(host='localhost', port=8500)

    def register_service(self, name, host, port):
        """Enregistre un service dans Consul"""
        self.consul.agent.service.register(
            name=name,
            service_id=f"{name}-{host}-{port}",
            address=host,
            port=port,
            check=consul.Check.http(
                f"http://{host}:{port}/health",
                interval="10s"
            )
        )

    def discover_service(self, name):
        """Trouve toutes les instances d'un service"""
        _, services = self.consul.health.service(name, passing=True)

        instances = []
        for service in services:
            instances.append({
                'host': service['Service']['Address'],
                'port': service['Service']['Port']
            })

        return instances

    def deregister_service(self, service_id):
        """Désenregistre un service"""
        self.consul.agent.service.deregister(service_id)

# Utilisation dans un service
discovery = ServiceDiscovery()

# Au démarrage : s'enregistrer
discovery.register_service('user-service', '10.0.1.5', 8080)

# Appeler un autre service
class OrderService:
    def __init__(self):
        self.discovery = ServiceDiscovery()

    def get_user(self, user_id):
        # Découvrir les instances du service utilisateur
        user_instances = self.discovery.discover_service('user-service')

        if not user_instances:
            raise ServiceUnavailable("No user-service instances available")

        # Load balancing simple (round-robin)
        instance = random.choice(user_instances)

        # Appeler l'instance
        url = f"http://{instance['host']}:{instance['port']}/users/{user_id}"
        response = requests.get(url, timeout=2.0)

        return response.json()
```

**Avantages** :
- ✅ Contrôle total du load balancing
- ✅ Pas de proxy intermédiaire

**Inconvénients** :
- ❌ Logique discovery dans chaque service
- ❌ Mise à jour du registry = redéployer clients

### 2. Service Discovery côté serveur

```python
# Utilisation d'un load balancer (Nginx, HAProxy, AWS ELB)
"""
Service A veut appeler Service B :

┌───────────┐         ┌──────────────┐         ┌───────────┐
│ Service A │────────►│ Load Balancer│────────►│Service B-1│
└───────────┘         │  (user-svc)  │         ├───────────┤
                      └──────────────┘         │Service B-2│
                                               ├───────────┤
                                               │Service B-3│
                                               └───────────┘

Service A appelle simplement : http://user-service
Le load balancer route vers une instance disponible
"""

# Configuration Nginx
"""
upstream user-service {
    server user-service-1:8080;
    server user-service-2:8080;
    server user-service-3:8080;
}

server {
    listen 80;
    server_name user-service;

    location / {
        proxy_pass http://user-service;
    }
}
"""

# Code simplifié côté client
class OrderService:
    def get_user(self, user_id):
        # Appel simple au nom du service
        # Le DNS/Load Balancer gère le routing
        response = requests.get(
            f'http://user-service/users/{user_id}',
            timeout=2.0
        )
        return response.json()
```

**Avantages** :
- ✅ Code client simple
- ✅ Changements centralisés

**Inconvénients** :
- ❌ Load balancer = SPOF
- ❌ Moins de contrôle

### 3. Service Mesh (moderne)

Un **service mesh** (Istio, Linkerd, Consul Connect) gère automatiquement :
- Service discovery
- Load balancing
- Encryption (mTLS)
- Observability
- Circuit breaking
- Retry logic

```yaml
# Exemple Istio : Configuration automatique
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: user-service
spec:
  hosts:
  - user-service
  http:
  - route:
    - destination:
        host: user-service
        subset: v1
      weight: 90
    - destination:
        host: user-service
        subset: v2
      weight: 10  # Canary deployment
    timeout: 2s
    retries:
      attempts: 3
      perTryTimeout: 500ms
```

```python
# Code application : AUCUN changement !
# Le sidecar proxy gère tout automatiquement
class OrderService:
    def get_user(self, user_id):
        # Appel simple
        response = requests.get(f'http://user-service/users/{user_id}')
        return response.json()

        # Istio sidecar gère automatiquement :
        # - Service discovery
        # - Load balancing
        # - Retries (3 tentatives)
        # - Timeout (2s)
        # - Circuit breaking
        # - mTLS encryption
        # - Metrics/tracing
```

**Architecture Service Mesh** :

```
┌──────────────────────────────────────┐
│         Service A Pod                │
│  ┌────────────┐    ┌──────────────┐  │
│  │ Service A  │◄──►│Envoy Sidecar │ ◄┼─┐
│  │ Container  │    │  (Proxy)     │  │ │
│  └────────────┘    └──────────────┘  │ │
└──────────────────────────────────────┘ │
                                         │
                                    Network
                                         │
┌──────────────────────────────────────┐ │
│         Service B Pod                │ │
│  ┌────────────┐    ┌──────────────┐  │ │
│  │ Service B  │◄──►│Envoy Sidecar │ ◄┼─┘
│  │ Container  │    │  (Proxy)     │  │
│  └────────────┘    └──────────────┘  │
└──────────────────────────────────────┘

Services communiquent via leurs sidecars
```

**Avantages** :
- ✅ Configuration centralisée
- ✅ Fonctionnalités avancées sans code
- ✅ Polyglotte (fonctionne avec tous langages)
- ✅ Observabilité complète

**Inconvénients** :
- ❌ Complexité opérationnelle
- ❌ Overhead latence (~1-2ms par hop)
- ❌ Courbe d'apprentissage

## Patterns de communication

### 1. Synchrone : REST/HTTP

```python
# Service Order appelle Service User
import requests

class OrderService:
    def __init__(self):
        self.user_service_url = "http://user-service"

    def create_order(self, user_id, items):
        # Appel synchrone : bloque jusqu'à réponse
        try:
            response = requests.get(
                f"{self.user_service_url}/users/{user_id}",
                timeout=2.0
            )
            response.raise_for_status()
            user = response.json()
        except requests.RequestException as e:
            raise ServiceError(f"Failed to get user: {e}")

        # Créer commande
        order = Order(user_id=user_id, items=items)
        db.add(order)

        return order
```

**Avantages** :
- ✅ Simple à implémenter
- ✅ Réponse immédiate
- ✅ Facile à débugger

**Inconvénients** :
- ❌ Couplage fort (service doit être disponible)
- ❌ Latence cumulée
- ❌ Cascade de failures

**Cas d'usage** : Lecture de données, opérations rapides

### 2. Asynchrone : Message Queue

```python
# Service Order publie un événement
import pika
import json

class OrderService:
    def __init__(self):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters('rabbitmq')
        )
        self.channel = self.connection.channel()
        self.channel.exchange_declare(
            exchange='events',
            exchange_type='topic'
        )

    def create_order(self, user_id, items):
        # Créer commande immédiatement
        order = Order(user_id=user_id, items=items)
        db.add(order)
        db.commit()

        # Publier événement (fire-and-forget)
        event = {
            'event_type': 'order.created',
            'order_id': order.id,
            'user_id': user_id,
            'items': items,
            'timestamp': datetime.utcnow().isoformat()
        }

        self.channel.basic_publish(
            exchange='events',
            routing_key='order.created',
            body=json.dumps(event)
        )

        return order

# Service Email consomme l'événement
class EmailService:
    def __init__(self):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters('rabbitmq')
        )
        self.channel = self.connection.channel()
        self.channel.queue_declare(queue='email_notifications')
        self.channel.queue_bind(
            exchange='events',
            queue='email_notifications',
            routing_key='order.created'
        )

    def start_consuming(self):
        def callback(ch, method, properties, body):
            event = json.loads(body)

            # Traiter de manière asynchrone
            self.send_order_confirmation(
                event['user_id'],
                event['order_id']
            )

            ch.basic_ack(delivery_tag=method.delivery_tag)

        self.channel.basic_consume(
            queue='email_notifications',
            on_message_callback=callback
        )

        self.channel.start_consuming()

    def send_order_confirmation(self, user_id, order_id):
        # Envoyer email
        print(f"Sending confirmation for order {order_id}")
```

**Avantages** :
- ✅ Découplage fort (services indépendants)
- ✅ Résilience (messages persistés)
- ✅ Scalabilité (consommateurs multiples)
- ✅ Pas de blocage

**Inconvénients** :
- ❌ Complexité (infrastructure messaging)
- ❌ Pas de réponse immédiate
- ❌ Ordre des messages non garanti (sauf config)
- ❌ Debugging plus difficile

**Cas d'usage** : Notifications, analytics, opérations longues

### 3. API Gateway Pattern

```python
# Gateway agrège plusieurs microservices
from fastapi import FastAPI, HTTPException
import httpx
import asyncio

app = FastAPI()

class APIGateway:
    def __init__(self):
        self.client = httpx.AsyncClient(timeout=5.0)

    async def get_user_dashboard(self, user_id: int):
        """Agrège données de multiples services"""
        try:
            # Appels parallèles
            user, orders, recommendations = await asyncio.gather(
                self.get_user(user_id),
                self.get_user_orders(user_id),
                self.get_recommendations(user_id),
                return_exceptions=True
            )

            # Gérer les erreurs individuellement
            result = {}

            if isinstance(user, Exception):
                result['user'] = None
                result['user_error'] = str(user)
            else:
                result['user'] = user

            if isinstance(orders, Exception):
                result['orders'] = []
                result['orders_error'] = str(orders)
            else:
                result['orders'] = orders

            if isinstance(recommendations, Exception):
                result['recommendations'] = []
            else:
                result['recommendations'] = recommendations

            return result

        except Exception as e:
            raise HTTPException(status_code=500, detail=str(e))

    async def get_user(self, user_id: int):
        response = await self.client.get(f'http://user-service/users/{user_id}')
        response.raise_for_status()
        return response.json()

    async def get_user_orders(self, user_id: int):
        response = await self.client.get(f'http://order-service/users/{user_id}/orders')
        response.raise_for_status()
        return response.json()

    async def get_recommendations(self, user_id: int):
        response = await self.client.get(f'http://recommendation-service/users/{user_id}')
        response.raise_for_status()
        return response.json()

gateway = APIGateway()

@app.get('/api/dashboard/{user_id}')
async def get_dashboard(user_id: int):
    return await gateway.get_user_dashboard(user_id)
```

**Avantages** :
- ✅ Point d'entrée unique
- ✅ Agrégation de données
- ✅ Authentification centralisée
- ✅ Rate limiting centralisé
- ✅ Cache possible

**Inconvénients** :
- ❌ SPOF potentiel
- ❌ Peut devenir complexe

**Architecture** :

```
┌────────┐        ┌─────────────┐
│ Client │───────►│ API Gateway │
└────────┘        └─────────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
    ┌─────────┐   ┌──────────┐   ┌─────────┐
    │ User    │   │  Order   │   │ Payment │
    │ Service │   │ Service  │   │ Service │
    └─────────┘   └──────────┘   └─────────┘
```

### 4. Backend for Frontend (BFF)

```python
# Un gateway par type de client
"""
┌──────────────┐        ┌─────────────┐
│ Web Client   │───────►│  Web BFF    │──┐
└──────────────┘        └─────────────┘  │
                                         │
┌──────────────┐        ┌─────────────┐  │    ┌──────────┐
│Mobile Client │───────►│ Mobile BFF  │──┼───►│ Services │
└──────────────┘        └─────────────┘  │    └──────────┘
                                         │
┌──────────────┐        ┌─────────────┐  │
│ IoT Device   │───────►│  IoT BFF    │──┘
└──────────────┘        └─────────────┘
"""

# Web BFF : Retourne HTML enrichi
@app.get('/dashboard/{user_id}')
async def web_dashboard(user_id: int):
    data = await gateway.get_user_dashboard(user_id)

    # Format riche pour web
    return {
        'user': {
            **data['user'],
            'avatar_url_large': f"{data['user']['avatar']}_large.jpg",
            'full_profile_link': f"/users/{user_id}"
        },
        'orders': data['orders'][:10],  # 10 dernières
        'recommendations': data['recommendations'][:20],  # 20 suggestions
        'ui_config': {
            'show_ads': True,
            'theme': 'dark'
        }
    }

# Mobile BFF : Format compact
@app.get('/mobile/dashboard/{user_id}')
async def mobile_dashboard(user_id: int):
    data = await gateway.get_user_dashboard(user_id)

    # Format compact pour mobile
    return {
        'user': {
            'id': data['user']['id'],
            'name': data['user']['name'],
            'avatar': f"{data['user']['avatar']}_thumb.jpg"  # Thumbnail
        },
        'orders': data['orders'][:5],  # 5 dernières
        'recommendations': data['recommendations'][:5]  # 5 suggestions
    }
```

**Avantages** :
- ✅ Optimisé par type de client
- ✅ Équipes indépendantes par plateforme
- ✅ Pas de compromis dans les APIs

**Inconvénients** :
- ❌ Duplication de logique
- ❌ Plus de services à maintenir

## Résilience et Fault Tolerance

### 1. Timeouts à tous les niveaux

```python
import httpx
import asyncio

class ResilientClient:
    """Client HTTP avec timeouts complets"""

    def __init__(self):
        self.client = httpx.AsyncClient(
            timeout=httpx.Timeout(
                connect=2.0,    # Connexion TCP
                read=5.0,       # Lecture réponse
                write=5.0,      # Écriture requête
                pool=1.0        # Attente connexion du pool
            )
        )

    async def call_service(self, url: str):
        try:
            response = await self.client.get(url)
            return response.json()
        except httpx.TimeoutException as e:
            # Timeout spécifique
            raise ServiceTimeout(f"Service timeout: {e}")
        except httpx.RequestError as e:
            # Erreur réseau
            raise ServiceError(f"Network error: {e}")
```

**Timeouts recommandés** :

```python
# Règle : Parent timeout > somme enfants timeout

# Service A (parent)
PARENT_TIMEOUT = 10.0  # secondes

# Service A appelle B, C, D en séquence
SERVICE_B_TIMEOUT = 3.0
SERVICE_C_TIMEOUT = 3.0
SERVICE_D_TIMEOUT = 3.0

# Total : 9.0s < 10.0s ✓ OK

# Si en parallèle
PARALLEL_TIMEOUT = max(SERVICE_B_TIMEOUT, SERVICE_C_TIMEOUT, SERVICE_D_TIMEOUT)
# = 3.0s < 10.0s ✓ OK avec marge
```

### 2. Circuit Breaker distribué

```python
import time
from enum import Enum
from collections import deque

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class DistributedCircuitBreaker:
    """Circuit breaker avec état partagé (Redis)"""

    def __init__(self, service_name, redis_client):
        self.service_name = service_name
        self.redis = redis_client
        self.key_prefix = f"circuit:{service_name}"

        # Configuration
        self.failure_threshold = 5
        self.recovery_timeout = 60
        self.success_threshold = 2

    def get_state(self):
        """Récupère l'état du circuit (partagé entre instances)"""
        state = self.redis.get(f"{self.key_prefix}:state")
        return CircuitState(state) if state else CircuitState.CLOSED

    def set_state(self, state: CircuitState):
        """Définit l'état du circuit"""
        self.redis.set(f"{self.key_prefix}:state", state.value)

    async def call(self, func, *args, **kwargs):
        """Exécute fonction avec circuit breaker"""
        state = self.get_state()

        if state == CircuitState.OPEN:
            # Vérifier si timeout récupération écoulé
            last_failure = float(self.redis.get(f"{self.key_prefix}:last_failure") or 0)
            if time.time() - last_failure > self.recovery_timeout:
                self.set_state(CircuitState.HALF_OPEN)
                self.redis.set(f"{self.key_prefix}:success_count", 0)
            else:
                raise CircuitBreakerOpen(f"Circuit breaker open for {self.service_name}")

        try:
            result = await func(*args, **kwargs)
            self.on_success()
            return result

        except Exception as e:
            self.on_failure()
            raise

    def on_success(self):
        """Succès : réinitialiser compteur échecs"""
        state = self.get_state()

        if state == CircuitState.HALF_OPEN:
            # Incrémenter succès
            success_count = self.redis.incr(f"{self.key_prefix}:success_count")

            if success_count >= self.success_threshold:
                self.set_state(CircuitState.CLOSED)
                self.redis.set(f"{self.key_prefix}:failure_count", 0)
        else:
            self.redis.set(f"{self.key_prefix}:failure_count", 0)

    def on_failure(self):
        """Échec : incrémenter compteur"""
        failure_count = self.redis.incr(f"{self.key_prefix}:failure_count")
        self.redis.set(f"{self.key_prefix}:last_failure", time.time())

        if failure_count >= self.failure_threshold:
            self.set_state(CircuitState.OPEN)

# Utilisation
import redis
import httpx

redis_client = redis.Redis(host='localhost', port=6379, decode_responses=True)
breaker = DistributedCircuitBreaker('user-service', redis_client)

async def get_user(user_id):
    async def call_user_service():
        async with httpx.AsyncClient() as client:
            response = await client.get(f'http://user-service/users/{user_id}')
            response.raise_for_status()
            return response.json()

    return await breaker.call(call_user_service)
```

### 3. Retry avec Exponential Backoff

```python
import asyncio
import random

async def retry_with_backoff(
    func,
    max_retries=3,
    base_delay=1.0,
    max_delay=60.0,
    exponential_base=2,
    jitter=True,
    retry_on=(Exception,)
):
    """Retry avec exponential backoff et jitter"""
    for attempt in range(max_retries):
        try:
            return await func()

        except retry_on as e:
            if attempt == max_retries - 1:
                raise

            # Calculer délai
            delay = min(base_delay * (exponential_base ** attempt), max_delay)

            # Ajouter jitter
            if jitter:
                delay = delay * (0.5 + random.random())

            print(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay:.2f}s")
            await asyncio.sleep(delay)

# Utilisation
async def call_unreliable_service():
    async with httpx.AsyncClient() as client:
        response = await client.get('http://unreliable-service/data')
        response.raise_for_status()
        return response.json()

data = await retry_with_backoff(
    call_unreliable_service,
    max_retries=5,
    retry_on=(httpx.TimeoutException, httpx.HTTPError)
)
```

### 4. Bulkhead Pattern (isolation)

```python
from concurrent.futures import ThreadPoolExecutor

class BulkheadExecutor:
    """Isole les appels à différents services"""

    def __init__(self):
        # Pool séparé par service
        self.executors = {
            'user-service': ThreadPoolExecutor(max_workers=10),
            'order-service': ThreadPoolExecutor(max_workers=20),
            'payment-service': ThreadPoolExecutor(max_workers=5),
        }

    async def call_service(self, service_name, func, *args, **kwargs):
        """Exécute fonction dans pool dédié"""
        executor = self.executors.get(service_name)
        if not executor:
            raise ValueError(f"Unknown service: {service_name}")

        loop = asyncio.get_event_loop()
        return await loop.run_in_executor(executor, func, *args, **kwargs)

# Si user-service est lent/down :
# - Son pool se sature (10 workers bloqués)
# - Mais order-service continue de fonctionner (pool séparé)
# → Isolation des failures
```

## Observabilité

### 1. Distributed Tracing

```python
# OpenTelemetry : Standard pour tracing
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Configuration
trace.set_tracer_provider(TracerProvider())
jaeger_exporter = JaegerExporter(
    agent_host_name='localhost',
    agent_port=6831,
)
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

tracer = trace.get_tracer(__name__)

# Utilisation
@app.post('/api/orders')
async def create_order(order_data: dict):
    # Span parent
    with tracer.start_as_current_span("create_order") as span:
        span.set_attribute("order.items_count", len(order_data['items']))

        # Appel user service
        with tracer.start_as_current_span("get_user"):
            user = await get_user(order_data['user_id'])

        # Appel payment service
        with tracer.start_as_current_span("process_payment"):
            payment = await process_payment(order_data['payment'])

        # Créer commande
        with tracer.start_as_current_span("save_order"):
            order = Order(**order_data)
            db.add(order)
            db.commit()

        span.set_attribute("order.id", order.id)
        return {"order_id": order.id}

# Trace complète visible dans Jaeger :
"""
create_order (total: 150ms)
├── get_user (20ms)
│   └── db_query (15ms)
├── process_payment (100ms)
│   ├── validate_card (10ms)
│   └── charge_card (90ms)
└── save_order (30ms)
    └── db_insert (25ms)
"""
```

**Visualisation Jaeger** :

```
Timeline :
0ms     50ms    100ms   150ms
|-------|-------|-------|
create_order ──────────────────────
  get_user ──────
            process_payment ───────────────
                          save_order ──────

Latence totale : 150ms
Latence réseau : 20ms + 100ms = 120ms (80% du temps !)
```

### 2. Structured Logging

```python
import logging
import json
from datetime import datetime

class StructuredLogger:
    """Logger JSON structuré"""

    def __init__(self, service_name):
        self.service_name = service_name
        self.logger = logging.getLogger(service_name)

    def log(self, level, message, **context):
        """Log structuré en JSON"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'service': self.service_name,
            'level': level,
            'message': message,
            **context
        }

        self.logger.log(
            getattr(logging, level.upper()),
            json.dumps(log_entry)
        )

    def info(self, message, **context):
        self.log('info', message, **context)

    def error(self, message, **context):
        self.log('error', message, **context)

# Utilisation
logger = StructuredLogger('order-service')

@app.post('/api/orders')
async def create_order(order_data: dict):
    logger.info(
        'Order creation started',
        user_id=order_data['user_id'],
        items_count=len(order_data['items']),
        total_amount=order_data['total']
    )

    try:
        order = await order_service.create(order_data)

        logger.info(
            'Order created successfully',
            order_id=order.id,
            user_id=order.user_id,
            processing_time_ms=order.processing_time
        )

        return {"order_id": order.id}

    except Exception as e:
        logger.error(
            'Order creation failed',
            user_id=order_data['user_id'],
            error=str(e),
            error_type=type(e).__name__
        )
        raise

# Logs JSON pour Elasticsearch/Splunk
"""
{
  "timestamp": "2025-01-15T10:30:00.123456",
  "service": "order-service",
  "level": "info",
  "message": "Order creation started",
  "user_id": 12345,
  "items_count": 3,
  "total_amount": 99.99
}
"""
```

### 3. Métriques (Prometheus)

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server

# Définir métriques
request_count = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

active_requests = Gauge(
    'http_active_requests',
    'Number of active requests',
    ['endpoint']
)

# Middleware FastAPI
from fastapi import FastAPI, Request
import time

app = FastAPI()

@app.middleware("http")
async def prometheus_middleware(request: Request, call_next):
    # Incrémenter requests actives
    endpoint = request.url.path
    active_requests.labels(endpoint=endpoint).inc()

    # Mesurer durée
    start_time = time.time()

    try:
        response = await call_next(request)

        # Métriques succès
        duration = time.time() - start_time

        request_count.labels(
            method=request.method,
            endpoint=endpoint,
            status=response.status_code
        ).inc()

        request_duration.labels(
            method=request.method,
            endpoint=endpoint
        ).observe(duration)

        return response

    finally:
        # Décrémenter requests actives
        active_requests.labels(endpoint=endpoint).dec()

# Exposer métriques sur :9090/metrics
start_http_server(9090)

# Métriques exportées :
"""
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="POST",endpoint="/api/orders",status="200"} 1523
http_requests_total{method="POST",endpoint="/api/orders",status="500"} 12

# HELP http_request_duration_seconds HTTP request duration
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{method="POST",endpoint="/api/orders",le="0.1"} 1200
http_request_duration_seconds_bucket{method="POST",endpoint="/api/orders",le="0.5"} 1450
http_request_duration_seconds_bucket{method="POST",endpoint="/api/orders",le="1.0"} 1520
http_request_duration_seconds_sum{method="POST",endpoint="/api/orders"} 234.56
http_request_duration_seconds_count{method="POST",endpoint="/api/orders"} 1535
"""
```

## Performance et optimisation

### 1. Connection Pooling

```python
import httpx

# Pool de connexions réutilisables
class ServiceClient:
    def __init__(self, base_url, pool_size=100):
        # Limites de connexions
        limits = httpx.Limits(
            max_keepalive_connections=pool_size,
            max_connections=pool_size * 2,
            keepalive_expiry=30.0
        )

        # Client avec pool
        self.client = httpx.AsyncClient(
            base_url=base_url,
            limits=limits,
            timeout=5.0,
            http2=True  # HTTP/2 pour multiplexage
        )

    async def get(self, path):
        # Réutilise connexion existante du pool
        response = await self.client.get(path)
        return response.json()

# Sans pool : Nouvelle connexion TCP à chaque requête
# Avec pool : Connexion réutilisée (100× plus rapide)
```

### 2. Request Batching

```python
class BatchingClient:
    """Groupe plusieurs requêtes en une seule"""

    def __init__(self, service_url, batch_size=100, batch_timeout=0.1):
        self.service_url = service_url
        self.batch_size = batch_size
        self.batch_timeout = batch_timeout

        self.pending = []
        self.batch_task = None

    async def get_user(self, user_id):
        """Get user avec batching automatique"""
        future = asyncio.Future()

        self.pending.append({
            'user_id': user_id,
            'future': future
        })

        # Démarrer timer si pas déjà fait
        if not self.batch_task:
            self.batch_task = asyncio.create_task(self._flush_after_timeout())

        # Flush si batch plein
        if len(self.pending) >= self.batch_size:
            await self._flush()

        return await future

    async def _flush_after_timeout(self):
        """Flush après timeout"""
        await asyncio.sleep(self.batch_timeout)
        await self._flush()

    async def _flush(self):
        """Envoyer le batch"""
        if not self.pending:
            return

        batch = self.pending
        self.pending = []
        self.batch_task = None

        # Une seule requête HTTP pour tous
        user_ids = [item['user_id'] for item in batch]

        try:
            async with httpx.AsyncClient() as client:
                response = await client.post(
                    f'{self.service_url}/users/batch',
                    json={'user_ids': user_ids}
                )
                users = response.json()

            # Distribuer résultats
            users_map = {u['id']: u for u in users}
            for item in batch:
                user = users_map.get(item['user_id'])
                item['future'].set_result(user)

        except Exception as e:
            # Propager erreur à tous
            for item in batch:
                item['future'].set_exception(e)

# Utilisation
client = BatchingClient('http://user-service')

# Ces 150 appels sont groupés en 2 requêtes HTTP :
# - 100 premiers (batch plein)
# - 50 restants (timeout)
users = await asyncio.gather(*[
    client.get_user(i) for i in range(150)
])

# Au lieu de 150 requêtes HTTP → 2 requêtes
# Réduction latence : 150 × 10ms = 1500ms → 20ms
```

### 3. Caching distribué

```python
import redis
import hashlib
import json

class DistributedCache:
    """Cache distribué avec Redis"""

    def __init__(self, redis_url='redis://localhost'):
        self.redis = redis.from_url(redis_url, decode_responses=True)

    def cache_key(self, prefix, *args, **kwargs):
        """Génère clé de cache"""
        key_data = f"{prefix}:{args}:{sorted(kwargs.items())}"
        return hashlib.md5(key_data.encode()).hexdigest()

    async def get_or_fetch(self, key, fetch_func, ttl=300):
        """Cache-aside pattern"""
        # Vérifier cache
        cached = self.redis.get(key)
        if cached:
            return json.loads(cached)

        # Fetch
        data = await fetch_func()

        # Mettre en cache
        self.redis.setex(key, ttl, json.dumps(data))

        return data

    def invalidate(self, pattern):
        """Invalider cache par pattern"""
        for key in self.redis.scan_iter(match=pattern):
            self.redis.delete(key)

# Utilisation
cache = DistributedCache()

async def get_user(user_id):
    key = cache.cache_key('user', user_id)

    async def fetch():
        async with httpx.AsyncClient() as client:
            response = await client.get(f'http://user-service/users/{user_id}')
            return response.json()

    return await cache.get_or_fetch(key, fetch, ttl=600)

# Premier appel : fetch service (100ms)
user = await get_user(123)

# Appels suivants (10 min) : cache (1ms)
user = await get_user(123)  # 100× plus rapide !
```

## Sécurité inter-services

### 1. mTLS (Mutual TLS)

```python
# Configuration certificats
"""
Service A                Service B
┌──────────┐            ┌──────────┐
│          │  ─────►    │          │
│  Cert A  │  TLS       │  Cert B  │
│          │  ◄─────    │          │
└──────────┘            └──────────┘

Les deux services vérifient mutuellement leurs certificats
"""

# Client avec mTLS
import httpx

client = httpx.Client(
    cert=(
        '/path/to/client-cert.pem',
        '/path/to/client-key.pem'
    ),
    verify='/path/to/ca-cert.pem'  # Vérifie serveur
)

response = client.get('https://service-b/data')

# Service Mesh (Istio) gère mTLS automatiquement
# Aucun code nécessaire !
```

### 2. Service-to-Service Authentication (JWT)

```python
import jwt
from datetime import datetime, timedelta

class ServiceAuth:
    """Authentication entre services"""

    def __init__(self, service_name, secret_key):
        self.service_name = service_name
        self.secret_key = secret_key

    def generate_token(self, target_service, ttl=300):
        """Génère token pour appeler un service"""
        payload = {
            'iss': self.service_name,  # Issuer (qui émet)
            'aud': target_service,     # Audience (pour qui)
            'iat': datetime.utcnow(),
            'exp': datetime.utcnow() + timedelta(seconds=ttl)
        }

        return jwt.encode(payload, self.secret_key, algorithm='HS256')

    def verify_token(self, token):
        """Vérifie token reçu"""
        try:
            payload = jwt.decode(
                token,
                self.secret_key,
                algorithms=['HS256'],
                audience=self.service_name  # Vérifier audience
            )
            return payload['iss']  # Retourner service source

        except jwt.InvalidTokenError:
            raise Unauthorized("Invalid service token")

# Service A (appelant)
auth = ServiceAuth('order-service', SECRET_KEY)

async def call_user_service(user_id):
    # Générer token
    token = auth.generate_token('user-service')

    # Appel avec token
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f'http://user-service/users/{user_id}',
            headers={'Authorization': f'Bearer {token}'}
        )
        return response.json()

# Service B (receveur)
from fastapi import Header, HTTPException

@app.get('/users/{user_id}')
async def get_user(user_id: int, authorization: str = Header(None)):
    if not authorization or not authorization.startswith('Bearer '):
        raise HTTPException(401, "Missing token")

    token = authorization.split(' ')[1]

    # Vérifier token
    try:
        source_service = auth.verify_token(token)
        print(f"Request from {source_service}")
    except Unauthorized:
        raise HTTPException(401, "Invalid token")

    user = db.get_user(user_id)
    return user
```

### 3. API Rate Limiting

```python
import time
from collections import defaultdict

class RateLimiter:
    """Rate limiting par service"""

    def __init__(self, redis_client):
        self.redis = redis_client

    def is_allowed(self, service_name, rate=100, per=60):
        """
        Rate limiting avec sliding window

        service_name: Nom du service appelant
        rate: Nombre de requêtes
        per: Période en secondes
        """
        key = f"ratelimit:{service_name}"
        now = time.time()
        window_start = now - per

        # Supprimer requêtes hors fenêtre
        self.redis.zremrangebyscore(key, 0, window_start)

        # Compter requêtes dans fenêtre
        count = self.redis.zcard(key)

        if count >= rate:
            return False

        # Ajouter cette requête
        self.redis.zadd(key, {str(now): now})
        self.redis.expire(key, per)

        return True

# Middleware
from fastapi import Request, HTTPException

rate_limiter = RateLimiter(redis_client)

@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    # Extraire service source du token
    token = request.headers.get('Authorization', '').replace('Bearer ', '')

    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        source_service = payload['iss']
    except:
        raise HTTPException(401, "Invalid token")

    # Vérifier rate limit
    if not rate_limiter.is_allowed(source_service, rate=1000, per=60):
        raise HTTPException(429, "Rate limit exceeded")

    response = await call_next(request)
    return response
```

## Cas d'usage réels

### 1. Netflix : Chaos Engineering

Netflix teste la résilience en **causant intentionnellement des pannes** :

```python
# Chaos Monkey : Arrête aléatoirement des instances
import random

class ChaosMonkey:
    """Simule pannes aléatoires en production"""

    def __init__(self, failure_probability=0.01):
        self.failure_probability = failure_probability

    async def call_service(self, func):
        """Appel service avec chaos"""
        # 1% de chance de panne simulée
        if random.random() < self.failure_probability:
            raise ServiceUnavailable("Chaos Monkey killed this request")

        return await func()

# Utilisation
chaos = ChaosMonkey(failure_probability=0.01)

@app.get('/api/data')
async def get_data():
    try:
        data = await chaos.call_service(fetch_from_service)
        return data
    except ServiceUnavailable:
        # Fallback
        return get_cached_data()

# Netflix teste ainsi en production :
# - Circuit breakers fonctionnent
# - Fallbacks activés correctement
# - Système reste disponible malgré pannes
```

### 2. Uber : Service Mesh adoption

Uber a migré vers un service mesh pour gérer :
- 2000+ microservices
- 40 000+ instances
- 150 000+ requêtes/seconde

```yaml
# Configuration Envoy (service mesh Uber)
# Retry automatique + circuit breaking
clusters:
- name: ride-service
  connect_timeout: 0.25s
  type: STRICT_DNS
  lb_policy: ROUND_ROBIN
  circuit_breakers:
    thresholds:
    - max_connections: 100
      max_pending_requests: 100
      max_requests: 100
      max_retries: 3
  outlier_detection:
    consecutive_5xx: 5
    interval: 30s
    base_ejection_time: 30s
  load_assignment:
    cluster_name: ride-service
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: ride-service
              port_value: 8080
```

**Résultats** :
- ✅ 50% réduction des erreurs 5xx
- ✅ P99 latency réduite de 30%
- ✅ Déploiements plus sûrs (canary automatique)

### 3. Amazon : Two-Pizza Teams

Amazon organise ses équipes en **"two-pizza teams"** - assez petites pour être nourries avec deux pizzas (~8 personnes).

```
Chaque équipe possède :
- Ses propres microservices
- Sa propre base de données
- Son propre pipeline CI/CD
- Autonomie complète

Communication :
- APIs bien définies
- Contracts tests
- SLA documentés

┌────────────────┐    API    ┌────────────────┐
│ Payments Team  │◄─────────►│ Orders Team    │
│ ┌────────────┐ │           │ ┌────────────┐ │
│ │Payment Svc │ │           │ │ Order Svc  │ │
│ └────────────┘ │           │ └────────────┘ │
│ ┌────────────┐ │           │ ┌────────────┐ │
│ │ PaymentDB  │ │           │ │  OrderDB   │ │
│ └────────────┘ │           │ └────────────┘ │
└────────────────┘           └────────────────┘
```

### 4. Spotify : Backend for Frontend

Spotify utilise des BFF pour chaque plateforme :

```python
# iOS BFF : Format optimisé mobile
@app.get('/mobile/playlist/{playlist_id}')
async def get_playlist_mobile(playlist_id: str):
    playlist = await playlist_service.get(playlist_id)

    return {
        'id': playlist.id,
        'name': playlist.name,
        'cover': f"{playlist.cover}_300x300.jpg",  # Résolution mobile
        'tracks': [
            {
                'id': t.id,
                'title': t.title,
                'artist': t.artist_name,  # Dénormalisé
                'duration': t.duration_ms,
                'preview_url': t.preview_url  # Pré-calculé
            }
            for t in playlist.tracks[:50]  # Limit mobile
        ]
    }

# Web BFF : Format riche
@app.get('/web/playlist/{playlist_id}')
async def get_playlist_web(playlist_id: str):
    playlist = await playlist_service.get(playlist_id)

    # Agrégation de plusieurs services
    creator, recommendations, analytics = await asyncio.gather(
        user_service.get(playlist.creator_id),
        recommendation_service.get_similar(playlist_id),
        analytics_service.get_stats(playlist_id)
    )

    return {
        'id': playlist.id,
        'name': playlist.name,
        'description': playlist.description,
        'cover': f"{playlist.cover}_1200x1200.jpg",  # Haute résolution
        'creator': {
            'id': creator.id,
            'name': creator.name,
            'followers': creator.follower_count
        },
        'tracks': [
            # Format détaillé avec relations
            {
                'id': t.id,
                'title': t.title,
                'album': {
                    'id': t.album.id,
                    'name': t.album.name,
                    'cover': t.album.cover
                },
                'artists': [
                    {'id': a.id, 'name': a.name}
                    for a in t.artists
                ],
                'duration': t.duration_ms,
                'explicit': t.explicit,
                'popularity': t.popularity
            }
            for t in playlist.tracks
        ],
        'recommendations': recommendations,
        'analytics': analytics
    }
```

## Anti-patterns à éviter

### 1. Distributed Monolith

```python
# ❌ MAUVAIS : Services trop couplés
"""
Order Service                Payment Service
     │                             │
     ├──► get_payment_method() ────┤
     ├──► validate_card() ─────────┤
     ├──► calculate_fees() ────────┤
     ├──► process_payment() ───────┤
     └──► send_receipt() ──────────┘

5 appels synchrones pour une commande !
→ Monolithe distribué (pire des deux mondes)
"""

# ✅ BON : Service autonome
"""
Order Service
     │
     └──► process_order() ──────┐
                                │
                         Payment Service
                         (tout en un appel)
"""

@payment_service.post('/process-order-payment')
async def process_order_payment(order_data):
    # Tout dans un appel
    payment_method = get_payment_method(order_data['user_id'])
    validate_card(payment_method)
    fees = calculate_fees(order_data['amount'])
    payment = process_payment(payment_method, order_data['amount'] + fees)
    send_receipt(order_data['user_id'], payment)

    return payment
```

### 2. Shared Database

```python
# ❌ MAUVAIS : Database partagée entre services
"""
┌─────────────┐       ┌────────────┐
│Order Service│──┐  ┌─│User Service│
└─────────────┘  │  │ └────────────┘
                 │  │
                 ▼  ▼
          ┌───────────────┐
          │Shared Database│
          └───────────────┘

→ Couplage au schéma
→ Impossible de scaler indépendamment
"""

# ✅ BON : Database par service
"""
┌─────────────┐       ┌────────────┐
│Order Service│       │User Service│
└─────────────┘       └────────────┘
      │                    │
      ▼                    ▼
┌──────────┐         ┌──────────┐
│ Order DB │         │ User DB  │
└──────────┘         └──────────┘

→ Indépendance totale
→ Scalabilité indépendante
"""
```

### 3. Synchronous Everything

```python
# ❌ MAUVAIS : Tout en synchrone
@app.post('/orders')
async def create_order(order_data):
    user = await user_service.get(order_data['user_id'])  # 10ms
    inventory = await inventory_service.check(order_data['items'])  # 50ms
    payment = await payment_service.charge(order_data['payment'])  # 200ms

    # Opérations non-critiques mais bloquantes
    await email_service.send_confirmation(user.email)  # 500ms
    await analytics_service.track_order(order_data)  # 100ms
    await recommendation_service.update(user.id)  # 150ms

    return {"order_id": order.id}

# Total : 1010ms dont 750ms non-critiques !

# ✅ BON : Async pour opérations non-critiques
@app.post('/orders')
async def create_order(order_data):
    # Critiques : synchrone
    user = await user_service.get(order_data['user_id'])
    inventory = await inventory_service.check(order_data['items'])
    payment = await payment_service.charge(order_data['payment'])

    order = create_order_record(order_data)

    # Non-critiques : async (message queue)
    publish_event('order.created', {
        'order_id': order.id,
        'user_id': user.id,
        'email': user.email
    })

    return {"order_id": order.id}

# Total : 260ms (4× plus rapide)
```

### 4. No Timeouts

```python
# ❌ MAUVAIS : Pas de timeout
response = requests.get('http://slow-service/data')
# Si service down/lent → bloque indéfiniment

# ✅ BON : Timeout partout
response = requests.get(
    'http://slow-service/data',
    timeout=(2.0, 5.0)  # connect, read
)
```

## Récapitulatif

### Défis clés

```
Challenge                   Solution
──────────────────────────────────────────────
Latence réseau              • Batching
                           • Caching
                           • Async communication

Failures réseau             • Timeouts
                           • Retry + backoff
                           • Circuit breakers
                           • Bulkheads

Service discovery           • Consul/Etcd
                           • Service mesh
                           • DNS

Observabilité              • Distributed tracing
                           • Structured logging
                           • Metrics (Prometheus)

Sécurité                   • mTLS
                           • Service auth (JWT)
                           • Rate limiting

Évolution                  • API versioning
                           • Backward compatibility
                           • Contract testing
```

### Checklist de conception

```python
# Avant de passer en microservices :

□ Équipe prête ? (10+ développeurs)
□ Besoin de scalabilité indépendante ?
□ Déploiements indépendants nécessaires ?
□ Infrast prête ? (Kubernetes, monitoring)
□ Accepter complexité opérationnelle ?

# Si non : Rester monolithe modulaire !

# Décisions par service :

□ Communication : Sync (REST) ou Async (queue) ?
□ Database : Propre ou partagée ?
□ Resilience : Timeouts ? Retries ? Circuit breaker ?
□ Observability : Tracing ? Logs ? Metrics ?
□ Security : mTLS ? JWT ? Rate limiting ?
```

### Best Practices

✅ **Design** :
- Services autonomes (database propre)
- APIs bien définies
- Backward compatible

✅ **Communication** :
- Async pour opérations non-critiques
- Batching quand possible
- Timeouts partout

✅ **Résilience** :
- Retry avec backoff
- Circuit breakers
- Graceful degradation

✅ **Observabilité** :
- Distributed tracing
- Structured logging
- Métriques détaillées

✅ **Sécurité** :
- mTLS entre services
- Service authentication
- Rate limiting

## Conclusion

Les microservices transforment des appels de fonctions en appels réseau. Cette transformation introduit :

- 📊 **Latence** : 10ms au lieu de 1µs
- 🔥 **Failures** : Réseau peut tomber
- 🔍 **Complexité** : Debugging distribué
- 🔒 **Sécurité** : Surface d'attaque plus large

Mais avec les bons patterns et outils, les microservices permettent :

- ✨ **Scalabilité** : Scaler services indépendamment
- 🚀 **Vitesse** : Équipes autonomes
- 🔧 **Flexibilité** : Technologies par service
- 📈 **Résilience** : Isolation des pannes

---


**Fin du Module 8 : Programmation réseau (perspective développeur)**

⏭️ [9. Architectures et concepts avancés](/09-architectures-avancees/README.md)
