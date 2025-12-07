🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.8 Résilience et fiabilité applicative

## Introduction

Dans un monde où les applications distribuées sont la norme, la résilience réseau n'est plus une option mais une nécessité. Même avec TCP qui garantit la livraison des paquets, une application peut échouer de multiples façons : serveurs surchargés, timeouts réseau, pannes transitoires, dégradations de performance. **La différence entre une application amateur et une application production-ready réside souvent dans sa capacité à gérer élégamment ces défaillances.**

Cette section explore les stratégies et patterns essentiels pour construire des applications réseau robustes qui survivent aux conditions réelles d'exploitation.

---

## Pourquoi la résilience est-elle critique ?

### Le réseau n'est pas fiable

Contrairement à un appel de fonction local qui réussit ou échoue de manière déterministe, une requête réseau traverse un système distribué complexe :

```
Client → DNS → Load Balancer → Reverse Proxy → Application → Database
          ↓         ↓              ↓              ↓            ↓
       Timeout   Saturation    Rate limit    Out of Memory  Slow query
```

**Chaque composant peut échouer de différentes manières** :
- **Échecs francs** : connexion refusée, timeout réseau
- **Échecs silencieux** : paquets perdus, black holes réseau
- **Dégradations** : latence élevée, débit réduit
- **Échecs partiels** : certaines requêtes passent, d'autres non

### Cas réel : L'effet domino d'AWS US-EAST-1 (2017)

Lors d'une panne majeure, une simple erreur de saisie dans une commande de maintenance a entraîné :
1. Arrêt de serveurs S3 critiques
2. Surcharge des serveurs restants
3. Cascade de timeouts dans les applications clientes
4. Saturation des retry mechanisms mal configurés
5. Amplification de la charge (retry storms)
6. Panne de services dépendants pourtant "indépendants"

**Conséquence** : Des milliers d'applications sont tombées non pas à cause de la panne S3, mais parce qu'elles n'étaient pas conçues pour gérer cette défaillance.

---

## Les huit défaillances de Deutsch

Peter Deutsch a formulé les **"Fallacies of Distributed Computing"**, hypothèses erronées que font souvent les développeurs :

1. **Le réseau est fiable** ❌
   - Réalité : paquets perdus, connexions interrompues

2. **La latence est nulle** ❌
   - Réalité : 1ms en datacenter, 100ms+ intercontinental

3. **La bande passante est infinie** ❌
   - Réalité : congestion, throttling, quotas

4. **Le réseau est sécurisé** ❌
   - Réalité : man-in-the-middle, injection, spoofing

5. **La topologie ne change pas** ❌
   - Réalité : autoscaling, déploiements, pannes

6. **Il y a un seul administrateur** ❌
   - Réalité : multi-cloud, fournisseurs tiers

7. **Le coût de transport est nul** ❌
   - Réalité : facturation au trafic, limites de quota

8. **Le réseau est homogène** ❌
   - Réalité : protocoles variés, versions différentes

**Chaque erreur de conception basée sur ces hypothèses crée des vulnérabilités en production.**

---

## Principes fondamentaux de la résilience

### 1. Fail Fast vs Fail Safe

Deux philosophies s'opposent :

**Fail Fast (Échouer rapidement)**
```python
# Bon pour : APIs synchrones, interfaces utilisateur
def get_user(user_id, timeout=2.0):
    try:
        response = http.get(f"/users/{user_id}", timeout=timeout)
        return response.json()
    except Timeout:
        raise UserServiceUnavailable("Service timeout")
```

**Avantages** :
- Libère rapidement les ressources
- Évite les threads/connexions bloqués
- Feedback immédiat à l'utilisateur

**Fail Safe (Dégrader gracieusement)**
```python
# Bon pour : systèmes critiques, données en cache acceptable
def get_user(user_id):
    try:
        return fetch_from_api(user_id, timeout=2.0)
    except Exception:
        cached = get_from_cache(user_id)
        if cached:
            return cached  # Données potentiellement périmées mais valides
        return get_default_user()  # Utilisateur anonyme
```

**Avantages** :
- Expérience utilisateur continuée
- Données partielles mieux que rien
- Absorbe les pannes transitoires

**Cas d'usage réel (Netflix)** : Quand l'API de recommandation échoue, Netflix affiche des recommandations génériques en cache plutôt qu'une page d'erreur. L'utilisateur peut toujours naviguer et regarder du contenu.

### 2. Isolation des défaillances (Bulkheading)

Inspiré des compartiments étanches des navires, ce pattern isole les ressources pour éviter qu'une défaillance ne contamine tout le système.

**Exemple : Pool de connexions séparés**
```javascript
// Anti-pattern : un seul pool pour tous les services
const pool = new ConnectionPool({ maxConnections: 100 });

// Si le service lent sature le pool, TOUS les services sont impactés
await pool.query('slow-service', ...);  // Bloque 90 connexions
await pool.query('fast-service', ...);  // Plus de connexions disponibles !
```

```javascript
// Pattern : pools isolés par service
const pools = {
    userService: new ConnectionPool({ max: 30 }),
    paymentService: new ConnectionPool({ max: 20 }),
    analyticsService: new ConnectionPool({ max: 10 })  // Non-critique
};

// La saturation d'analytics n'impacte pas payment
```

**Cas d'usage réel (Microservices)** :
- Threads/goroutines dédiés par dépendance externe
- Rate limiters séparés
- Timeouts différenciés selon la criticité

### 3. Dégradation gracieuse

Prioriser les fonctionnalités essentielles en cas de surcharge.

**Exemple : E-commerce sous charge**
```python
class CheckoutService:
    def process_order(self, cart, user):
        # Critique : enregistrer la commande
        order = self.save_order(cart, user)  # Timeout: 5s

        try:
            # Important : calculer les points de fidélité
            points = self.loyalty_service.calculate(order, timeout=1.0)
            self.loyalty_service.credit(user, points, timeout=1.0)
        except Timeout:
            # Dégrader : on calculera les points en asynchrone
            self.queue_loyalty_calculation(order.id)

        try:
            # Nice-to-have : recommandations personnalisées
            recommendations = self.ml_service.get_recommendations(
                user, timeout=0.5
            )
        except Timeout:
            # Dégrader : recommandations génériques
            recommendations = self.get_popular_items()

        return {
            'order': order,
            'loyalty_points': points or 'pending',
            'recommendations': recommendations
        }
```

**Hiérarchie de criticité** :
1. **Must-have** : Transaction financière, commande
2. **Should-have** : Email de confirmation, mise à jour de stock
3. **Nice-to-have** : Recommandations, analytics

### 4. Idempotence

Une opération idempotente produit le même résultat qu'elle soit exécutée une ou plusieurs fois.

**Pourquoi c'est crucial pour la résilience** :
- Permet de retenter (retry) sans effet de bord
- Gère les duplications réseau
- Simplifie la réconciliation

**Exemple : API de paiement**
```http
POST /payments HTTP/1.1
Content-Type: application/json
Idempotency-Key: 7d8f6e42-3a9b-4c5d-8e2f-1a3b4c5d6e7f

{
  "amount": 99.99,
  "currency": "EUR",
  "customer_id": "cust_123"
}
```

```python
# Côté serveur
def process_payment(amount, customer_id, idempotency_key):
    # Vérifier si déjà traité
    existing = db.query(
        "SELECT * FROM payments WHERE idempotency_key = ?",
        idempotency_key
    )

    if existing:
        return existing  # Retourner le résultat existant

    # Nouveau paiement
    payment = charge_customer(amount, customer_id)
    db.insert({
        'idempotency_key': idempotency_key,
        'payment_id': payment.id,
        'status': payment.status
    })

    return payment
```

**Cas d'usage réels** :
- **Stripe** : Toutes les APIs de mutation acceptent un `Idempotency-Key`
- **AWS S3** : PUT sur la même clé est idempotent
- **Kafka** : Producers avec `enable.idempotence=true`

### 5. Observabilité et monitoring

**On ne peut pas améliorer ce qu'on ne mesure pas.**

**Métriques essentielles** :
```go
type NetworkMetrics struct {
    RequestDuration    histogram  // Latence P50, P95, P99
    ErrorRate          counter    // % d'échecs
    TimeoutRate        counter    // % de timeouts
    RetryCount         counter    // Nombre de retries
    CircuitBreakerState gauge     // open/half-open/closed
    ConnectionPoolUsage gauge     // % de connexions utilisées
}
```

**Exemple d'implémentation (Go avec Prometheus)** :
```go
var (
    requestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Buckets: []float64{.005, .01, .025, .05, .1, .25, .5, 1, 2.5, 5},
        },
        []string{"service", "endpoint", "status"},
    )

    errorRate = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_request_errors_total",
        },
        []string{"service", "error_type"},
    )
)

func makeRequest(service, endpoint string) {
    start := time.Now()

    resp, err := http.Get(endpoint)
    duration := time.Since(start).Seconds()

    status := "success"
    if err != nil {
        status = "error"
        errorRate.WithLabelValues(service, classifyError(err)).Inc()
    }

    requestDuration.WithLabelValues(service, endpoint, status).Observe(duration)
}
```

---

## Architecture de résilience en couches

Une application résiliente combine plusieurs niveaux de défense :

```
┌─────────────────────────────────────────────────┐
│  Application Layer                              │
│  • Circuit Breakers                             │
│  • Retry Logic                                  │
│  • Fallbacks                                    │
├─────────────────────────────────────────────────┤
│  Network Layer                                  │
│  • Connection Timeouts                          │
│  • Read/Write Timeouts                          │
│  • Keep-Alive                                   │
├─────────────────────────────────────────────────┤
│  Infrastructure Layer                           │
│  • Load Balancers                               │
│  • Health Checks                                │
│  • Auto-scaling                                 │
├─────────────────────────────────────────────────┤
│  Protocol Layer                                 │
│  • TCP Retransmissions                          │
│  • Congestion Control                           │
│  • Flow Control                                 │
└─────────────────────────────────────────────────┘
```

**Chaque couche compense les limites de la couche inférieure.**

---

## Patterns de résilience : Vue d'ensemble

Les sous-sections suivantes détailleront chaque pattern, mais voici un aperçu :

### Timeouts (8.8.1)
Définir des limites temporelles pour éviter les blocages infinis.

**Types** :
- **Connection timeout** : temps max pour établir une connexion
- **Read timeout** : temps max pour recevoir des données
- **Write timeout** : temps max pour envoyer des données

### Retry Logic et Backoff Exponentiel (8.8.2)
Retenter intelligemment les opérations échouées.

**Stratégies** :
- **Immediate retry** : pour erreurs transitoires
- **Exponential backoff** : 1s, 2s, 4s, 8s...
- **Jitter** : aléatoire pour éviter la synchronisation

### Circuit Breakers (8.8.3)
Arrêter temporairement les appels vers un service défaillant.

**États** :
- **Closed** : trafic normal
- **Open** : blocage des requêtes
- **Half-Open** : test de récupération

### Keep-Alive et Connexions Persistantes (8.8.4)
Réutiliser les connexions pour éviter le coût du handshake.

**Bénéfices** :
- Réduit la latence (pas de 3-way handshake)
- Diminue la charge serveur
- Améliore le throughput

---

## Cas d'usage complet : API Gateway résilient

Voici comment tous ces patterns s'assemblent dans une architecture réelle :

```python
from circuit_breaker import CircuitBreaker
from retry import retry_with_backoff
import httpx

class ResilientAPIGateway:
    def __init__(self):
        # Pool de connexions avec keep-alive
        self.client = httpx.AsyncClient(
            timeout=httpx.Timeout(
                connect=2.0,    # Connection timeout
                read=5.0,       # Read timeout
                write=5.0,      # Write timeout
                pool=10.0       # Pool acquisition timeout
            ),
            limits=httpx.Limits(
                max_connections=100,
                max_keepalive_connections=20
            ),
            http2=True  # Multiplexage pour réduire les connexions
        )

        # Circuit breakers par service
        self.breakers = {
            'user-service': CircuitBreaker(
                failure_threshold=5,
                recovery_timeout=30,
                expected_exception=httpx.HTTPError
            ),
            'payment-service': CircuitBreaker(
                failure_threshold=3,  # Plus strict pour paiements
                recovery_timeout=60
            )
        }

    @retry_with_backoff(
        max_attempts=3,
        base_delay=1.0,
        max_delay=10.0,
        exponential_base=2,
        jitter=True
    )
    async def call_service(self, service_name, endpoint, **kwargs):
        breaker = self.breakers[service_name]

        # Le circuit breaker bloque si le service est down
        with breaker:
            try:
                response = await self.client.get(
                    f"http://{service_name}{endpoint}",
                    **kwargs
                )
                response.raise_for_status()
                return response.json()

            except httpx.TimeoutException:
                # Timeout : on retry (géré par le décorateur)
                raise

            except httpx.HTTPStatusError as e:
                if e.response.status_code >= 500:
                    # Erreur serveur : on retry
                    raise
                else:
                    # Erreur client (4xx) : pas de retry
                    return self.handle_client_error(e)

    async def get_user_with_fallback(self, user_id):
        try:
            return await self.call_service(
                'user-service',
                f'/users/{user_id}'
            )
        except Exception as e:
            # Fallback : utilisateur en cache ou anonyme
            cached = await self.cache.get(f'user:{user_id}')
            if cached:
                return {'source': 'cache', **cached}

            # Dernière défense : utilisateur anonyme
            return {
                'source': 'fallback',
                'id': user_id,
                'name': 'Anonymous User',
                'error': str(e)
            }
```

**Ce que fait ce code** :
1. **Timeouts** à plusieurs niveaux (connexion, lecture, écriture)
2. **Keep-alive** avec pool de connexions réutilisables
3. **Circuit breakers** pour isoler les services défaillants
4. **Retry avec backoff exponentiel** pour les erreurs transitoires
5. **Fallback** vers cache ou données par défaut
6. **HTTP/2** pour le multiplexage

---

## Anti-patterns courants à éviter

### ❌ Retry infini sans limite

```python
# DANGER : Peut créer des retry storms
def fetch_data(url):
    while True:
        try:
            return requests.get(url, timeout=30)
        except Exception:
            time.sleep(1)  # Retry immédiat
            continue  # Boucle infinie !
```

**Conséquences** :
- Threads/processus bloqués indéfiniment
- Amplification de charge sur service défaillant
- Épuisement des ressources

### ❌ Timeout trop long

```python
# Anti-pattern : timeout de 60 secondes
response = requests.get(url, timeout=60)
```

**Problème** : Un utilisateur web n'attendra jamais 60 secondes. Le timeout doit refléter les attentes utilisateur (généralement 2-5s pour une API).

### ❌ Absence de jitter dans les retries

```python
# Tous les clients retry en même temps !
for attempt in range(3):
    try:
        return call_api()
    except:
        time.sleep(2 ** attempt)  # 1s, 2s, 4s - synchronisé !
```

**Conséquence** : Thundering herd - tous les clients attaquent simultanément après un délai identique.

**Solution** : Ajouter du jitter aléatoire
```python
import random
time.sleep((2 ** attempt) + random.uniform(0, 1))
```

### ❌ Ignorer les codes HTTP 4xx

```python
# Anti-pattern : retry sur toutes les erreurs
@retry(tries=3)
def get_resource(id):
    response = requests.get(f'/api/resources/{id}')
    response.raise_for_status()
    return response.json()
```

**Problème** : Un 404 (Not Found) ou 400 (Bad Request) ne sera jamais résolu par un retry. On gaspille des ressources.

**Solution** : Retry uniquement sur 5xx et timeouts
```python
def should_retry(exception):
    if isinstance(exception, requests.Timeout):
        return True
    if isinstance(exception, requests.HTTPError):
        return 500 <= exception.response.status_code < 600
    return False

@retry(retry=retry_if_exception(should_retry))
def get_resource(id):
    ...
```

---

## Métriques de résilience

**SLI/SLO (Service Level Indicators/Objectives)** :

```yaml
# Exemple de SLO pour une API
availability:
  target: 99.9%  # "Three nines"
  measurement: success_rate over 30d rolling window

latency:
  p50: < 100ms
  p95: < 500ms
  p99: < 1000ms

error_budget:
  monthly: 0.1% = 43.2 minutes de downtime
  alert_if: > 50% budget consumed in 24h
```

**Calcul du Error Budget** :
```
Downtime autorisé (99.9%) = 43.2 min/mois
Downtime actuel = 15 min en 2 semaines
Budget consommé = 15/43.2 = 34.7%
```

---

## Checklist de résilience pour développeurs

Avant de déployer une application réseau en production, vérifiez :

- [ ] **Timeouts configurés** sur toutes les opérations réseau
- [ ] **Retry logic** avec backoff exponentiel et jitter
- [ ] **Circuit breakers** sur les dépendances externes
- [ ] **Connection pooling** avec limites appropriées
- [ ] **Keep-alive** activé pour connexions persistantes
- [ ] **Fallbacks** définis pour fonctionnalités non-critiques
- [ ] **Idempotence** garantie pour opérations de mutation
- [ ] **Monitoring** avec métriques de latence et taux d'erreur
- [ ] **Rate limiting** pour protéger contre la surcharge
- [ ] **Bulkheading** pour isoler les ressources
- [ ] **Logging** structuré pour diagnostiquer les pannes
- [ ] **Chaos engineering** testé (simulation de pannes)

---

## Conclusion

La résilience applicative n'est pas un ajout optionnel, c'est un **requirement fondamental** pour toute application réseau moderne. Les techniques présentées dans cette section - timeouts, retries, circuit breakers, keep-alive - forment un arsenal complet pour gérer les défaillances inévitables des systèmes distribués.

**Points clés à retenir** :
1. Le réseau échouera - concevez pour l'échec, pas pour le succès
2. Combinez plusieurs patterns de résilience en couches
3. Mesurez tout - l'observabilité est la clé de l'amélioration
4. Testez les pannes en environnement de test (chaos engineering)
5. Dégradez gracieusement plutôt que d'échouer complètement

Les sections suivantes détailleront chacune de ces techniques avec des implémentations concrètes et des exemples de code dans différents langages.

---


⏭️ [Timeouts : lecture, écriture, connexion](/08-programmation-reseau/08.1-timeouts.md)
