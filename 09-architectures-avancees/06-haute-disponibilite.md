🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.6 Architectures Haute Disponibilité

## Introduction

La **haute disponibilité (HA - High Availability)** désigne la capacité d'un système à rester **opérationnel et accessible** même en cas de défaillances matérielles, logicielles ou réseau. C'est la différence entre un site qui "tombe parfois" et un service critique qui **ne peut jamais s'arrêter**.

### L'analogie de l'hôpital

Imaginez un service d'urgences hospitalières :

**Sans haute disponibilité :**
```
Un seul médecin → S'il est malade, l'hôpital ferme ☠️
Une seule salle → Si elle est en maintenance, pas de soins ☠️
Un seul générateur → S'il tombe en panne, tout s'éteint ☠️

Résultat : Inacceptable pour un service critique
```

**Avec haute disponibilité :**
```
Équipe de médecins → Un absent ? Les autres prennent le relais ✅
Salles multiples → Une fermée ? Utilisation des autres ✅
Générateurs redondants + batteries → Une panne ? Bascule automatique ✅
Procédures de backup → Plan B, C, D pour chaque scénario ✅

Résultat : Service continu 24/7/365
```

### Le coût de l'indisponibilité

**Exemples réels de pannes célèbres :**

| Entreprise | Année | Durée | Coût estimé | Impact |
|------------|-------|-------|-------------|--------|
| **Amazon** | 2013 | 30 min | $66,240/min = **$2M** | Ventes perdues |
| **Facebook** | 2021 | 6 heures | $100M+ | Coupure mondiale |
| **Google** | 2013 | 5 min | $545,000 | Gmail down |
| **British Airways** | 2017 | 3 jours | $100M+ | 75,000 vols annulés |
| **Cloudflare** | 2020 | 27 min | N/A | 50% d'Internet impacté |

**Formule du coût par minute :**
```
Coût/minute = (Revenu annuel / 525,600 minutes) × multiplicateur_impact

Exemple e-commerce $100M/an :
= ($100M / 525,600) × 2 (perte de confiance)
= $380/minute
= $22,800/heure
= $547,200/jour
```

## Les "9" de disponibilité

La disponibilité se mesure en **pourcentage de temps opérationnel** (uptime).

### Tableau des niveaux de disponibilité

| Disponibilité | Downtime/an | Downtime/mois | Downtime/semaine | Niveau |
|---------------|-------------|---------------|------------------|--------|
| **90%** (1 nine) | 36.5 jours | 3 jours | 16.8 heures | Inacceptable |
| **99%** (2 nines) | 3.65 jours | 7.2 heures | 1.68 heures | Basique |
| **99.9%** (3 nines) | 8.76 heures | 43.2 minutes | 10.1 minutes | Standard |
| **99.95%** | 4.38 heures | 21.6 minutes | 5 minutes | Bon |
| **99.99%** (4 nines) | 52.56 minutes | 4.32 minutes | 1.01 minutes | Haute dispo |
| **99.995%** | 26.28 minutes | 2.16 minutes | 30 secondes | Très haute dispo |
| **99.999%** (5 nines) | 5.26 minutes | 26 secondes | 6 secondes | Ultra critique |
| **99.9999%** (6 nines) | 31.5 secondes | 2.6 secondes | 0.6 secondes | Extrême |

### Calcul de disponibilité

```python
import datetime

class AvailabilityCalculator:
    """
    Calculateur de disponibilité et SLA
    """

    MINUTES_PER_YEAR = 525600
    MINUTES_PER_MONTH = 43800
    MINUTES_PER_WEEK = 10080

    @staticmethod
    def uptime_to_percentage(uptime_minutes, period_minutes):
        """
        Convertir uptime en pourcentage
        """
        return (uptime_minutes / period_minutes) * 100

    @staticmethod
    def percentage_to_downtime(availability_percent, period='year'):
        """
        Convertir % disponibilité en temps d'indisponibilité
        """
        periods = {
            'year': AvailabilityCalculator.MINUTES_PER_YEAR,
            'month': AvailabilityCalculator.MINUTES_PER_MONTH,
            'week': AvailabilityCalculator.MINUTES_PER_WEEK
        }

        total_minutes = periods.get(period, AvailabilityCalculator.MINUTES_PER_YEAR)
        downtime_minutes = total_minutes * (1 - availability_percent / 100)

        return {
            'minutes': downtime_minutes,
            'hours': downtime_minutes / 60,
            'days': downtime_minutes / 1440,
            'formatted': AvailabilityCalculator.format_duration(downtime_minutes)
        }

    @staticmethod
    def format_duration(minutes):
        """Formater la durée en format lisible"""
        if minutes < 1:
            return f"{minutes * 60:.1f} secondes"
        elif minutes < 60:
            return f"{minutes:.1f} minutes"
        elif minutes < 1440:
            return f"{minutes / 60:.1f} heures"
        else:
            return f"{minutes / 1440:.1f} jours"

    @staticmethod
    def calculate_nines(availability_percent):
        """Calculer le nombre de '9' de disponibilité"""
        nines = 0
        remaining = 100 - availability_percent

        while remaining < 10 ** (-nines):
            nines += 1

        return nines - 1 if nines > 0 else 0

    @staticmethod
    def calculate_composite_availability(components):
        """
        Calculer la disponibilité composite (série)

        Exemple : LB (99.99%) → App (99.95%) → DB (99.99%)
        Dispo totale = 0.9999 × 0.9995 × 0.9999 = 99.93%
        """
        composite = 1.0
        for component in components:
            composite *= (component / 100)

        return composite * 100

    @staticmethod
    def calculate_parallel_availability(components):
        """
        Calculer la disponibilité en parallèle (redondance)

        Deux serveurs à 99% en parallèle :
        Prob d'échec = 0.01 × 0.01 = 0.0001
        Dispo = 1 - 0.0001 = 99.99%
        """
        failure_prob = 1.0
        for component in components:
            failure_prob *= (1 - component / 100)

        return (1 - failure_prob) * 100

# Usage
calc = AvailabilityCalculator()

# Combien de downtime pour 99.99% ?
downtime = calc.percentage_to_downtime(99.99, 'year')
print(f"99.99% = {downtime['formatted']} de downtime par an")
# Output: 99.99% = 52.6 minutes de downtime par an

# Disponibilité composite
components = [99.99, 99.95, 99.99]  # LB, App, DB
total = calc.calculate_composite_availability(components)
print(f"Disponibilité totale (série) : {total:.4f}%")
# Output: Disponibilité totale (série) : 99.9300%

# Disponibilité avec redondance
servers = [99.0, 99.0]  # Deux serveurs à 99% en failover
total = calc.parallel_availability(servers)
print(f"Disponibilité avec redondance : {total:.4f}%")
# Output: Disponibilité avec redondance : 99.9900%
```

## Principes fondamentaux de la haute disponibilité

### 1. Éliminer les SPOF (Single Points of Failure)

Un **SPOF** est tout composant dont la défaillance arrête le système entier.

```
AVANT (avec SPOF)
═══════════════════════════════════════════════════════

Internet → [Load Balancer] → [App Server] → [Database]
              ☠️ SPOF          ☠️ SPOF       ☠️ SPOF

Si N'IMPORTE LEQUEL tombe → Tout le système down


APRÈS (SPOF éliminés)
═══════════════════════════════════════════════════════

Internet → [LB1]  ┐
           [LB2]  ┼→ [App1]  ┐
         (VRRP)   │  [App2]  ┼→ [DB Primary]  ← [DB Replica]
                  │  [App3]  ┘  (Replication)    (Failover)
                  └─ Healthchecks

Chaque composant est redondant
```

### 2. Redondance à tous les niveaux

```
┌─────────────────────────────────────────────────────┐
│  NIVEAU 1 : Réseau                                  │
│  • Plusieurs ISP (multi-homing)                     │
│  • Liens redondants                                 │
│  • BGP failover automatique                         │
├─────────────────────────────────────────────────────┤
│  NIVEAU 2 : Load Balancers                          │
│  • LB1 + LB2 en VRRP/Keepalived                     │
│  • IP virtuelle flottante                           │
│  • Health checks mutuels                            │
├─────────────────────────────────────────────────────┤
│  NIVEAU 3 : Serveurs applicatifs                    │
│  • N serveurs (N ≥ 3 pour quorum)                   │
│  • Auto-scaling selon la charge                     │
│  • Distribution multi-AZ                            │
├─────────────────────────────────────────────────────┤
│  NIVEAU 4 : Données                                 │
│  • Master-Slave replication                         │
│  • Multi-master (CockroachDB, Cassandra)            │
│  • Backups automatiques multi-sites                 │
├─────────────────────────────────────────────────────┤
│  NIVEAU 5 : Alimentation                            │
│  • Onduleurs (UPS)                                  │
│  • Générateurs diesel                               │
│  • Batteries                                        │
└─────────────────────────────────────────────────────┘
```

### 3. Failover automatique

Le **failover** doit être **automatique et rapide** (<30 secondes idéalement).

```python
import time
import threading
import requests
from enum import Enum

class ServerState(Enum):
    PRIMARY = "primary"
    STANDBY = "standby"
    FAILED = "failed"

class HANode:
    """
    Nœud haute disponibilité avec heartbeat et failover
    """

    def __init__(self, node_id, is_primary=False):
        self.node_id = node_id
        self.state = ServerState.PRIMARY if is_primary else ServerState.STANDBY
        self.peer = None
        self.last_heartbeat = time.time()
        self.heartbeat_interval = 1  # secondes
        self.heartbeat_timeout = 3  # secondes
        self.running = True

    def set_peer(self, peer):
        """Définir le nœud pair"""
        self.peer = peer

    def send_heartbeat(self):
        """Envoyer un heartbeat au pair"""
        if self.peer:
            self.peer.receive_heartbeat(self.node_id)

    def receive_heartbeat(self, from_node):
        """Recevoir un heartbeat"""
        self.last_heartbeat = time.time()
        print(f"[{self.node_id}] Heartbeat reçu de {from_node}")

    def check_peer_health(self):
        """Vérifier la santé du pair"""
        if not self.peer:
            return True

        elapsed = time.time() - self.last_heartbeat

        if elapsed > self.heartbeat_timeout:
            print(f"[{self.node_id}] ⚠️  Pair non responsive depuis {elapsed:.1f}s")
            return False

        return True

    def promote_to_primary(self):
        """Promouvoir ce nœud en PRIMARY"""
        if self.state != ServerState.PRIMARY:
            print(f"[{self.node_id}] 🔼 PROMOTION vers PRIMARY")
            self.state = ServerState.PRIMARY

            # Actions de promotion
            self.take_over_virtual_ip()
            self.start_accepting_connections()

    def demote_to_standby(self):
        """Rétrograder ce nœud en STANDBY"""
        if self.state != ServerState.STANDBY:
            print(f"[{self.node_id}] 🔽 RÉTROGRADATION vers STANDBY")
            self.state = ServerState.STANDBY

            # Actions de rétrogradation
            self.release_virtual_ip()
            self.stop_accepting_connections()

    def take_over_virtual_ip(self):
        """Prendre possession de l'IP virtuelle"""
        print(f"[{self.node_id}] 🌐 Acquisition de l'IP virtuelle 192.168.1.100")
        # En réalité : commandes ip/ifconfig
        # ip addr add 192.168.1.100/24 dev eth0

    def release_virtual_ip(self):
        """Libérer l'IP virtuelle"""
        print(f"[{self.node_id}] 🔌 Libération de l'IP virtuelle")

    def start_accepting_connections(self):
        """Commencer à accepter les connexions"""
        print(f"[{self.node_id}] ✅ Acceptation des connexions ACTIVE")

    def stop_accepting_connections(self):
        """Arrêter d'accepter les connexions"""
        print(f"[{self.node_id}] ⏸️  Acceptation des connexions INACTIVE")

    def run(self):
        """Boucle principale du nœud"""
        print(f"[{self.node_id}] Démarrage en mode {self.state.value}")

        while self.running:
            # Envoyer heartbeat
            self.send_heartbeat()

            # Vérifier la santé du pair
            peer_healthy = self.check_peer_health()

            # Logique de failover
            if self.state == ServerState.STANDBY and not peer_healthy:
                # Le primary est down → promouvoir
                self.promote_to_primary()

            elif self.state == ServerState.PRIMARY and peer_healthy:
                # Le pair est revenu et était primary avant
                if hasattr(self.peer, 'state') and self.peer.state == ServerState.PRIMARY:
                    # Split-brain potentiel → résolution
                    print(f"[{self.node_id}] ⚠️  SPLIT-BRAIN détecté")
                    # Stratégie : node_id le plus bas reste primary
                    if self.node_id > self.peer.node_id:
                        self.demote_to_standby()

            time.sleep(self.heartbeat_interval)

    def stop(self):
        """Arrêter le nœud"""
        self.running = False

# Simulation
def simulate_ha_cluster():
    """Simuler un cluster HA avec failover"""

    # Créer deux nœuds
    node1 = HANode("node1", is_primary=True)
    node2 = HANode("node2", is_primary=False)

    # Les connecter
    node1.set_peer(node2)
    node2.set_peer(node1)

    # Démarrer dans des threads
    thread1 = threading.Thread(target=node1.run)
    thread2 = threading.Thread(target=node2.run)

    thread1.start()
    thread2.start()

    # Laisser tourner 5 secondes
    time.sleep(5)

    # Simuler une panne du primary
    print("\n💥 SIMULATION : Panne du nœud primary (node1)\n")
    node1.stop()

    # Observer le failover
    time.sleep(5)

    # Arrêter tout
    node2.stop()

    thread1.join()
    thread2.join()

# Exécuter la simulation
if __name__ == '__main__':
    simulate_ha_cluster()
```

### 4. Isolation des défaillances (Bulkhead Pattern)

Compartimenter le système pour qu'une défaillance ne se propage pas.

```python
import threading
from queue import Queue
import time

class BulkheadExecutor:
    """
    Pattern Bulkhead : Isolation des pools de ressources

    Comme les compartiments étanches d'un navire :
    si un compartiment prend l'eau, les autres restent secs.
    """

    def __init__(self, max_concurrent=10):
        self.semaphore = threading.Semaphore(max_concurrent)
        self.max_concurrent = max_concurrent
        self.active_count = 0
        self.rejected_count = 0
        self.lock = threading.Lock()

    def execute(self, func, *args, **kwargs):
        """
        Exécuter une fonction avec limitation de concurrence
        """
        # Essayer d'acquérir un slot
        acquired = self.semaphore.acquire(blocking=False)

        if not acquired:
            # Bulkhead plein → rejeter
            with self.lock:
                self.rejected_count += 1

            raise BulkheadFullException(
                f"Bulkhead saturé ({self.max_concurrent} tâches actives)"
            )

        try:
            with self.lock:
                self.active_count += 1

            # Exécuter la fonction
            return func(*args, **kwargs)

        finally:
            with self.lock:
                self.active_count -= 1

            self.semaphore.release()

    def get_stats(self):
        """Statistiques du bulkhead"""
        return {
            'max_concurrent': self.max_concurrent,
            'active': self.active_count,
            'rejected': self.rejected_count,
            'available': self.max_concurrent - self.active_count
        }

class BulkheadFullException(Exception):
    pass

class ServiceBulkheads:
    """
    Système avec bulkheads séparés par service
    """

    def __init__(self):
        # Chaque service a son propre bulkhead
        self.bulkheads = {
            'database': BulkheadExecutor(max_concurrent=20),
            'api_external': BulkheadExecutor(max_concurrent=5),
            'api_internal': BulkheadExecutor(max_concurrent=50),
            'email': BulkheadExecutor(max_concurrent=10),
            'analytics': BulkheadExecutor(max_concurrent=3)
        }

    def call_database(self, query):
        """Appeler la DB avec bulkhead"""
        return self.bulkheads['database'].execute(
            self._execute_query, query
        )

    def call_external_api(self, endpoint):
        """Appeler une API externe avec bulkhead"""
        return self.bulkheads['api_external'].execute(
            self._call_api, endpoint
        )

    def _execute_query(self, query):
        """Simuler requête DB"""
        time.sleep(0.1)  # Latence DB
        return f"Result for {query}"

    def _call_api(self, endpoint):
        """Simuler appel API"""
        time.sleep(0.5)  # Latence API externe
        return f"Response from {endpoint}"

    def get_all_stats(self):
        """Stats de tous les bulkheads"""
        return {
            service: bulkhead.get_stats()
            for service, bulkhead in self.bulkheads.items()
        }

# Usage
service = ServiceBulkheads()

def worker(task_id):
    """Worker qui appelle différents services"""
    try:
        # Appel DB
        result = service.call_database(f"SELECT * FROM users WHERE id={task_id}")
        print(f"Task {task_id}: DB OK")

        # Appel API externe (peut être lent/saturé)
        result = service.call_external_api(f"/api/data/{task_id}")
        print(f"Task {task_id}: API OK")

    except BulkheadFullException as e:
        print(f"Task {task_id}: ⚠️  {e}")

# Simuler une charge
threads = []
for i in range(100):
    t = threading.Thread(target=worker, args=(i,))
    t.start()
    threads.append(t)

# Attendre un peu
time.sleep(2)

# Stats
stats = service.get_all_stats()
for service_name, stat in stats.items():
    print(f"{service_name}: {stat['active']}/{stat['max_concurrent']} actifs, "
          f"{stat['rejected']} rejetés")

# Attendre la fin
for t in threads:
    t.join()
```

**Résultat :** Si l'API externe est lente et sature son bulkhead (5 concurrent max), les autres services (DB, email, etc.) continuent de fonctionner normalement. La défaillance est **isolée**.

## Architectures HA classiques

### 1. Active-Passive (Failover)

Un nœud actif, un (ou plusieurs) en standby.

```
NORMAL OPERATION
═══════════════════════════════════════════════════════

Client → [IP Virtuelle: 192.168.1.100]
              ↓
         [Node 1: PRIMARY] ✅ Active
              ↓
         [Database]

         [Node 2: STANDBY] ⏸️  Idle (replication)


APRÈS DÉFAILLANCE NODE 1
═══════════════════════════════════════════════════════

Client → [IP Virtuelle: 192.168.1.100]
              ↓
         [Node 1: DOWN] ☠️

              ↓ (Failover automatique)
              ↓
         [Node 2: PRIMARY] ✅ Promoted
              ↓
         [Database]


Avantages :
✅ Simple à implémenter
✅ Pas de risque de split-brain (si bien configuré)
✅ Coût modéré (un seul nœud actif)

Inconvénients :
❌ Gaspillage de ressources (standby idle)
❌ Temps de failover (10-30s typiquement)
❌ Perte de sessions en cours
```

**Implémentation avec Keepalived (VRRP) :**

```bash
# /etc/keepalived/keepalived.conf (Node 1 - PRIMARY)

vrrp_script check_app {
    script "/usr/local/bin/check_app.sh"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100              # Plus élevé = primary
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass secret123
    }

    virtual_ipaddress {
        192.168.1.100/24
    }

    track_script {
        check_app
    }

    notify_master /usr/local/bin/notify_master.sh
    notify_backup /usr/local/bin/notify_backup.sh
    notify_fault /usr/local/bin/notify_fault.sh
}
```

```bash
# /etc/keepalived/keepalived.conf (Node 2 - BACKUP)

vrrp_script check_app {
    script "/usr/local/bin/check_app.sh"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90               # Plus bas que le master
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass secret123
    }

    virtual_ipaddress {
        192.168.1.100/24
    }

    track_script {
        check_app
    }

    notify_master /usr/local/bin/notify_master.sh
    notify_backup /usr/local/bin/notify_backup.sh
    notify_fault /usr/local/bin/notify_fault.sh
}
```

```bash
#!/bin/bash
# /usr/local/bin/check_app.sh

# Vérifier que l'app répond
curl -f http://localhost:8080/health > /dev/null 2>&1

if [ $? -eq 0 ]; then
    exit 0  # Healthy
else
    exit 1  # Unhealthy → déclenche failover
fi
```

```bash
#!/bin/bash
# /usr/local/bin/notify_master.sh

# Actions lors de la promotion en MASTER
echo "$(date) - Promoted to MASTER" >> /var/log/keepalived-transitions.log

# Démarrer les services
systemctl start nginx
systemctl start myapp

# Notifier (Slack, PagerDuty, etc.)
curl -X POST https://hooks.slack.com/... \
  -d '{"text": "🔼 Node promoted to MASTER"}'
```

### 2. Active-Active (Load Balanced)

Tous les nœuds sont actifs simultanément.

```
ARCHITECTURE ACTIVE-ACTIVE
═══════════════════════════════════════════════════════

Client → [Load Balancer]
              ↓
      ┌───────┼───────┐
      ↓       ↓       ↓
  [Node 1] [Node 2] [Node 3]  ✅ Tous actifs
      ↓       ↓       ↓
   ┌──────────┴───────────┐
   ↓                      ↓
[DB Primary]         [Cache Cluster]
   ↓
[DB Replica]


Avantages :
✅ Utilisation optimale des ressources
✅ Pas de failover (juste redistribution)
✅ Scaling horizontal facile
✅ Performance maximale

Inconvénients :
❌ Complexité accrue (session sharing)
❌ Cohérence des données (cache, DB)
❌ Coût plus élevé
```

**Implémentation avec session partagée :**

```python
from flask import Flask, session
from flask_session import Session
import redis

app = Flask(__name__)

# Configuration session externe (Redis)
app.config['SESSION_TYPE'] = 'redis'
app.config['SESSION_REDIS'] = redis.from_url('redis://redis-cluster:6379')
app.config['SESSION_PERMANENT'] = False
app.config['SESSION_USE_SIGNER'] = True
app.config['SESSION_KEY_PREFIX'] = 'session:'

Session(app)

@app.route('/')
def index():
    """
    Cette route peut être servie par N'IMPORTE QUEL nœud
    La session est partagée via Redis
    """
    # Incrémenter un compteur de visite
    session['visits'] = session.get('visits', 0) + 1

    return {
        'message': 'Hello from active-active cluster',
        'visits': session['visits'],
        'server': os.environ.get('HOSTNAME', 'unknown')
    }

@app.route('/health')
def health():
    """Health check pour le load balancer"""
    try:
        # Vérifier Redis
        app.config['SESSION_REDIS'].ping()

        # Vérifier DB
        # db.session.execute('SELECT 1')

        return {'status': 'healthy'}, 200
    except Exception as e:
        return {'status': 'unhealthy', 'error': str(e)}, 503

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

### 3. Multi-Region Active-Active

Disponibilité géographique avec réplication multi-sites.

```
ARCHITECTURE MULTI-REGION
═══════════════════════════════════════════════════════

           [Global Load Balancer / GeoDNS]
                      ↓
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
   [US-EAST]      [EU-WEST]     [ASIA-PAC]
       ↓              ↓              ↓
   [LB + Apps]    [LB + Apps]   [LB + Apps]
       ↓              ↓              ↓
   [DB Primary] ←→ [DB Replica] ←→ [DB Replica]
                 (Bi-directional replication)

Avantages :
✅ Tolérance aux défaillances régionales
✅ Latence minimale (utilisateurs routés localement)
✅ Compliance (données dans la bonne région)

Inconvénients :
❌ Complexité maximale
❌ Coût élevé
❌ Conflits de réplication potentiels
❌ Cohérence éventuelle vs forte
```

## Patterns de résilience applicative

### 1. Circuit Breaker

Empêcher les appels à un service défaillant pour éviter la cascade de pannes.

```python
import time
import threading
from enum import Enum
from collections import deque

class CircuitState(Enum):
    CLOSED = "closed"        # Tout va bien
    OPEN = "open"            # Service down, on rejette
    HALF_OPEN = "half_open"  # Test si service revenu

class CircuitBreaker:
    """
    Circuit Breaker pattern

    États :
    - CLOSED : requêtes passent normalement
    - OPEN : requêtes rejetées (service considéré down)
    - HALF_OPEN : test si service revenu
    """

    def __init__(self, failure_threshold=5, timeout=60, success_threshold=2):
        self.failure_threshold = failure_threshold
        self.timeout = timeout  # Durée avant retry (état OPEN)
        self.success_threshold = success_threshold  # Succès pour fermer

        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.last_failure_time = None
        self.lock = threading.Lock()

        # Historique pour monitoring
        self.history = deque(maxlen=100)

    def call(self, func, *args, **kwargs):
        """
        Exécuter une fonction via le circuit breaker
        """
        with self.lock:
            if self.state == CircuitState.OPEN:
                # Vérifier si timeout écoulé
                if time.time() - self.last_failure_time >= self.timeout:
                    # Passer en HALF_OPEN pour tester
                    self.state = CircuitState.HALF_OPEN
                    print("🔄 Circuit HALF_OPEN (test)")
                else:
                    # Toujours OPEN → rejeter
                    self._record('rejected')
                    raise CircuitBreakerOpen("Circuit ouvert - service indisponible")

        # Tenter l'appel
        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result

        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        """Enregistrer un succès"""
        with self.lock:
            self._record('success')

            if self.state == CircuitState.HALF_OPEN:
                self.success_count += 1

                if self.success_count >= self.success_threshold:
                    # Service revenu → fermer le circuit
                    self.state = CircuitState.CLOSED
                    self.failure_count = 0
                    self.success_count = 0
                    print("✅ Circuit CLOSED (service rétabli)")

            elif self.state == CircuitState.CLOSED:
                # Reset failure count sur succès
                self.failure_count = max(0, self.failure_count - 1)

    def _on_failure(self):
        """Enregistrer un échec"""
        with self.lock:
            self._record('failure')
            self.failure_count += 1
            self.last_failure_time = time.time()

            if self.state == CircuitState.HALF_OPEN:
                # Échec pendant test → revenir OPEN
                self.state = CircuitState.OPEN
                self.success_count = 0
                print("❌ Circuit OPEN (test échoué)")

            elif self.state == CircuitState.CLOSED:
                if self.failure_count >= self.failure_threshold:
                    # Trop d'échecs → ouvrir le circuit
                    self.state = CircuitState.OPEN
                    print(f"⚠️  Circuit OPEN ({self.failure_count} échecs)")

    def _record(self, event):
        """Enregistrer un événement dans l'historique"""
        self.history.append({
            'time': time.time(),
            'event': event,
            'state': self.state.value
        })

    def get_state(self):
        """Obtenir l'état actuel"""
        return {
            'state': self.state.value,
            'failure_count': self.failure_count,
            'success_count': self.success_count
        }

    def reset(self):
        """Reset manuel du circuit"""
        with self.lock:
            self.state = CircuitState.CLOSED
            self.failure_count = 0
            self.success_count = 0
            print("🔄 Circuit RESET manuel")

class CircuitBreakerOpen(Exception):
    pass

# Usage
def unreliable_api_call():
    """Simuler un appel API instable"""
    import random
    if random.random() < 0.3:  # 30% de chance d'échec
        raise Exception("API error")
    return "Success"

# Créer un circuit breaker
cb = CircuitBreaker(failure_threshold=3, timeout=10, success_threshold=2)

# Tester
for i in range(20):
    try:
        result = cb.call(unreliable_api_call)
        print(f"Call {i+1}: {result} - État: {cb.get_state()['state']}")
    except CircuitBreakerOpen as e:
        print(f"Call {i+1}: Circuit ouvert, requête rejetée")
    except Exception as e:
        print(f"Call {i+1}: Échec - État: {cb.get_state()['state']}")

    time.sleep(1)
```

### 2. Retry avec Backoff exponentiel

```python
import time
import random
from functools import wraps

def retry_with_backoff(
    max_retries=3,
    initial_delay=1,
    max_delay=60,
    exponential_base=2,
    jitter=True
):
    """
    Décorateur de retry avec backoff exponentiel

    Args:
        max_retries: Nombre max de tentatives
        initial_delay: Délai initial (secondes)
        max_delay: Délai maximal (secondes)
        exponential_base: Base pour le backoff (2 = doublement)
        jitter: Ajouter un délai aléatoire (évite thundering herd)
    """
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            delay = initial_delay
            last_exception = None

            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)

                except Exception as e:
                    last_exception = e

                    if attempt == max_retries:
                        # Dernière tentative échouée
                        raise

                    # Calculer le délai
                    wait_time = min(delay, max_delay)

                    # Ajouter du jitter (0-100% du délai)
                    if jitter:
                        wait_time = wait_time * (0.5 + random.random())

                    print(f"Tentative {attempt + 1}/{max_retries + 1} échouée: {e}")
                    print(f"Retry dans {wait_time:.2f}s...")

                    time.sleep(wait_time)

                    # Backoff exponentiel
                    delay *= exponential_base

            raise last_exception

        return wrapper
    return decorator

# Usage
@retry_with_backoff(max_retries=5, initial_delay=1, exponential_base=2)
def call_unreliable_service():
    """Service qui échoue parfois"""
    import random

    if random.random() < 0.7:  # 70% de chance d'échec
        raise Exception("Service temporairement indisponible")

    return "Success!"

# Tester
try:
    result = call_unreliable_service()
    print(f"✅ Résultat: {result}")
except Exception as e:
    print(f"❌ Échec final après toutes les tentatives: {e}")

# Délais typiques :
# Tentative 1 : immédiat
# Tentative 2 : ~1s (+ jitter)
# Tentative 3 : ~2s (+ jitter)
# Tentative 4 : ~4s (+ jitter)
# Tentative 5 : ~8s (+ jitter)
# Tentative 6 : ~16s (+ jitter)
```

### 3. Timeout adaptatif

```python
import time
import threading
from collections import deque
import statistics

class AdaptiveTimeout:
    """
    Timeout qui s'adapte aux latences observées
    """

    def __init__(self, initial_timeout=5, min_timeout=1, max_timeout=30):
        self.initial_timeout = initial_timeout
        self.min_timeout = min_timeout
        self.max_timeout = max_timeout

        self.latencies = deque(maxlen=100)  # Garder 100 dernières mesures
        self.lock = threading.Lock()

    def calculate_timeout(self):
        """
        Calculer le timeout adaptatif
        Formule : P95 des latences + buffer (2x P95)
        """
        with self.lock:
            if len(self.latencies) < 10:
                # Pas assez de données → timeout initial
                return self.initial_timeout

            # Calculer P95
            p95 = statistics.quantiles(self.latencies, n=20)[18]  # 95e percentile

            # Timeout = P95 + 2×P95 (marge de sécurité)
            timeout = p95 * 3

            # Limiter entre min et max
            timeout = max(self.min_timeout, min(timeout, self.max_timeout))

            return timeout

    def record_latency(self, latency):
        """Enregistrer une latence observée"""
        with self.lock:
            self.latencies.append(latency)

    def execute_with_timeout(self, func, *args, **kwargs):
        """Exécuter une fonction avec timeout adaptatif"""
        timeout = self.calculate_timeout()

        start = time.time()
        result = None
        exception = None

        def target():
            nonlocal result, exception
            try:
                result = func(*args, **kwargs)
            except Exception as e:
                exception = e

        thread = threading.Thread(target=target)
        thread.daemon = True
        thread.start()
        thread.join(timeout=timeout)

        elapsed = time.time() - start

        if thread.is_alive():
            # Timeout dépassé
            raise TimeoutError(f"Fonction a dépassé le timeout ({timeout:.2f}s)")

        # Enregistrer la latence
        self.record_latency(elapsed)

        if exception:
            raise exception

        return result

    def get_stats(self):
        """Statistiques"""
        with self.lock:
            if not self.latencies:
                return {}

            return {
                'current_timeout': self.calculate_timeout(),
                'avg_latency': statistics.mean(self.latencies),
                'p50_latency': statistics.median(self.latencies),
                'p95_latency': statistics.quantiles(self.latencies, n=20)[18] if len(self.latencies) >= 20 else None,
                'samples': len(self.latencies)
            }

# Usage
adaptive = AdaptiveTimeout(initial_timeout=5)

def slow_operation():
    """Opération de latence variable"""
    import random
    delay = random.uniform(0.1, 3.0)
    time.sleep(delay)
    return f"Completed in {delay:.2f}s"

# Exécuter plusieurs fois
for i in range(20):
    try:
        result = adaptive.execute_with_timeout(slow_operation)
        stats = adaptive.get_stats()
        print(f"Call {i+1}: {result}")
        print(f"  Timeout actuel: {stats['current_timeout']:.2f}s")
        print(f"  Latence moyenne: {stats['avg_latency']:.2f}s")
    except TimeoutError as e:
        print(f"Call {i+1}: ⚠️  Timeout - {e}")
```

## Kubernetes High Availability

### Architecture HA Kubernetes

```
KUBERNETES HA CLUSTER
═══════════════════════════════════════════════════════

CONTROL PLANE (3+ masters)
┌─────────────┬─────────────┬─────────────┐
│   Master 1  │   Master 2  │   Master 3  │
│  (Leader)   │  (Standby)  │  (Standby)  │
├─────────────┼─────────────┼─────────────┤
│ API Server  │ API Server  │ API Server  │
│ Scheduler   │ Scheduler   │ Scheduler   │
│ Controller  │ Controller  │ Controller  │
└─────┬───────┴──────┬──────┴──────┬──────┘
      │              │             │
      └──────────────┼─────────────┘
                     ↓
              [etcd cluster]
           (3 ou 5 nœuds, quorum)


WORKER NODES (N+ workers)
┌──────────┬──────────┬──────────┬──────────┐
│  Node 1  │  Node 2  │  Node 3  │  Node N  │
│  (AZ-1)  │  (AZ-2)  │  (AZ-3)  │  (AZ-X)  │
├──────────┼──────────┼──────────┼──────────┤
│  Pods    │  Pods    │  Pods    │  Pods    │
└──────────┴──────────┴──────────┴──────────┘
```

### Déploiement HA d'une application

```yaml
# deployment-ha.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  # Nombre de réplicas (HA)
  replicas: 5

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # 1 pod de plus pendant update
      maxUnavailable: 1  # Max 1 pod down pendant update

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      # Anti-affinity : éviter plusieurs pods sur même nœud
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - myapp
              topologyKey: kubernetes.io/hostname

        # Répartir sur plusieurs zones
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - myapp
            topologyKey: topology.kubernetes.io/zone

      # Tolérance aux pannes
      tolerations:
      - key: node.kubernetes.io/unreachable
        operator: Exists
        effect: NoExecute
        tolerationSeconds: 30
      - key: node.kubernetes.io/not-ready
        operator: Exists
        effect: NoExecute
        tolerationSeconds: 30

      containers:
      - name: myapp
        image: myapp:v1

        # Ressources garanties (QoS Guaranteed)
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "500m"

        # Health checks
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2

        # Graceful shutdown
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]

        ports:
        - containerPort: 8080
          name: http
          protocol: TCP

        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: host
---
# Service avec healthcheck
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP

  # Session affinity (optionnel)
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3 heures
---
# PodDisruptionBudget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 3  # Toujours 3 pods minimum disponibles
  selector:
    matchLabels:
      app: myapp
---
# HorizontalPodAutoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 5
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Attendre 5min avant de scale down
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60  # Max -50% par minute
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15  # Max +100% par 15s (réaction rapide)
```

### Endpoints Health Check

```python
from flask import Flask, jsonify
import psycopg2
import redis
import time

app = Flask(__name__)

class HealthChecker:
    """Vérification de santé multi-niveaux"""

    def __init__(self):
        self.startup_time = time.time()
        self.ready = False

    def check_liveness(self):
        """
        Liveness : le processus fonctionne-t-il ?
        Si échec répété → Kubernetes redémarre le pod
        """
        try:
            # Vérifications basiques
            # Le processus tourne-t-il depuis assez longtemps ?
            uptime = time.time() - self.startup_time

            if uptime < 5:
                # Trop récent, pas encore initialisé
                return False, "Starting up"

            # Vérifier qu'on peut allouer de la mémoire
            _ = [0] * 1000

            # Vérifier les threads critiques (exemple)
            # if not critical_thread.is_alive():
            #     return False, "Critical thread dead"

            return True, "OK"

        except Exception as e:
            return False, str(e)

    def check_readiness(self):
        """
        Readiness : peut-on servir du trafic ?
        Si échec → Kubernetes retire du service (mais ne redémarre pas)
        """
        try:
            # Vérifier dépendances externes

            # 1. Database
            try:
                conn = psycopg2.connect(
                    host="db-host",
                    database="myapp",
                    user="user",
                    password="pass",
                    connect_timeout=2
                )
                conn.close()
            except Exception as e:
                return False, f"Database unreachable: {e}"

            # 2. Cache
            try:
                r = redis.Redis(host='redis-host', port=6379, socket_timeout=2)
                r.ping()
            except Exception as e:
                return False, f"Redis unreachable: {e}"

            # 3. Autres services critiques
            # ...

            # Tout OK
            self.ready = True
            return True, "Ready"

        except Exception as e:
            self.ready = False
            return False, str(e)

health_checker = HealthChecker()

@app.route('/health/live')
def liveness():
    """
    Endpoint liveness
    """
    alive, message = health_checker.check_liveness()

    if alive:
        return jsonify({'status': 'alive', 'message': message}), 200
    else:
        return jsonify({'status': 'dead', 'message': message}), 503

@app.route('/health/ready')
def readiness():
    """
    Endpoint readiness
    """
    ready, message = health_checker.check_readiness()

    if ready:
        return jsonify({'status': 'ready', 'message': message}), 200
    else:
        return jsonify({'status': 'not_ready', 'message': message}), 503

@app.route('/health/startup')
def startup():
    """
    Endpoint startup (Kubernetes 1.16+)
    Pour les apps avec long temps de démarrage
    """
    uptime = time.time() - health_checker.startup_time

    if uptime > 60:  # 60s pour démarrer
        # Démarrage OK, passer aux autres checks
        return jsonify({'status': 'started'}), 200
    else:
        # Toujours en train de démarrer
        return jsonify({
            'status': 'starting',
            'uptime': uptime
        }), 503

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

## Disaster Recovery (DR)

### Stratégies de backup

```python
import boto3
import datetime
import gzip
import shutil
import subprocess

class BackupManager:
    """
    Gestionnaire de backups avec rotation
    """

    def __init__(self, s3_bucket='myapp-backups'):
        self.s3 = boto3.client('s3')
        self.bucket = s3_bucket

    def backup_database(self, db_host, db_name, db_user):
        """
        Backup PostgreSQL vers S3
        """
        timestamp = datetime.datetime.utcnow().strftime('%Y%m%d_%H%M%S')
        filename = f"{db_name}_backup_{timestamp}.sql"
        compressed_filename = f"{filename}.gz"

        # 1. Dump SQL
        dump_cmd = [
            'pg_dump',
            '-h', db_host,
            '-U', db_user,
            '-d', db_name,
            '-F', 'plain',
            '-f', filename
        ]

        subprocess.run(dump_cmd, check=True)

        # 2. Compresser
        with open(filename, 'rb') as f_in:
            with gzip.open(compressed_filename, 'wb') as f_out:
                shutil.copyfileobj(f_in, f_out)

        # 3. Upload vers S3
        s3_key = f"database/{compressed_filename}"

        self.s3.upload_file(
            compressed_filename,
            self.bucket,
            s3_key,
            ExtraArgs={
                'StorageClass': 'STANDARD_IA',  # Infrequent Access
                'ServerSideEncryption': 'AES256'
            }
        )

        # 4. Nettoyer localement
        import os
        os.remove(filename)
        os.remove(compressed_filename)

        print(f"✅ Backup uploadé : s3://{self.bucket}/{s3_key}")

        return s3_key

    def rotate_backups(self, prefix='database/', retention_days=30):
        """
        Supprimer les backups trop vieux

        Stratégie :
        - Garder tous les backups < 7 jours
        - Garder 1 backup/semaine pour 7-30 jours
        - Garder 1 backup/mois pour > 30 jours
        """
        response = self.s3.list_objects_v2(
            Bucket=self.bucket,
            Prefix=prefix
        )

        if 'Contents' not in response:
            return

        now = datetime.datetime.utcnow()

        for obj in response['Contents']:
            key = obj['Key']
            last_modified = obj['LastModified'].replace(tzinfo=None)
            age_days = (now - last_modified).days

            should_delete = False

            if age_days > retention_days:
                # Plus vieux que la rétention → supprimer
                should_delete = True

            if should_delete:
                print(f"🗑️  Suppression backup vieux de {age_days} jours: {key}")
                self.s3.delete_object(Bucket=self.bucket, Key=key)

    def restore_database(self, backup_key, db_host, db_name, db_user):
        """
        Restaurer depuis un backup S3
        """
        # 1. Télécharger depuis S3
        local_file = '/tmp/restore.sql.gz'

        self.s3.download_file(self.bucket, backup_key, local_file)

        # 2. Décompresser
        uncompressed_file = '/tmp/restore.sql'

        with gzip.open(local_file, 'rb') as f_in:
            with open(uncompressed_file, 'wb') as f_out:
                shutil.copyfileobj(f_in, f_out)

        # 3. Restaurer
        restore_cmd = [
            'psql',
            '-h', db_host,
            '-U', db_user,
            '-d', db_name,
            '-f', uncompressed_file
        ]

        subprocess.run(restore_cmd, check=True)

        print(f"✅ Base restaurée depuis {backup_key}")

    def test_backup(self, backup_key):
        """
        Tester qu'un backup est valide
        """
        # Restaurer dans une DB temporaire
        test_db = f"test_restore_{datetime.datetime.utcnow().strftime('%Y%m%d%H%M%S')}"

        try:
            # Créer DB temporaire
            subprocess.run([
                'createdb',
                '-h', 'test-db-host',
                '-U', 'postgres',
                test_db
            ], check=True)

            # Restaurer
            self.restore_database(backup_key, 'test-db-host', test_db, 'postgres')

            # Vérifier intégrité
            # SELECT COUNT(*) FROM tables, etc.

            print(f"✅ Backup {backup_key} validé")

            return True

        finally:
            # Supprimer DB temporaire
            subprocess.run([
                'dropdb',
                '-h', 'test-db-host',
                '-U', 'postgres',
                test_db
            ])

# Automatisation avec cron
# 0 2 * * * python3 -c "from backup import BackupManager; BackupManager().backup_database(...)"
# 0 3 * * 0 python3 -c "from backup import BackupManager; BackupManager().rotate_backups()"
```

### RTO et RPO

**RTO (Recovery Time Objective) :** Temps maximum d'indisponibilité acceptable.

**RPO (Recovery Point Objective) :** Perte de données maximale acceptable.

```python
class DisasterRecoveryPlanner:
    """
    Calculateur RTO/RPO et stratégies
    """

    STRATEGIES = {
        'backup_restore': {
            'rto_hours': 4,
            'rpo_hours': 24,
            'cost_factor': 1.0,
            'description': 'Backup quotidien + restauration manuelle'
        },
        'pilot_light': {
            'rto_hours': 1,
            'rpo_hours': 1,
            'cost_factor': 2.5,
            'description': 'Infra minimale prête + DB répliquée'
        },
        'warm_standby': {
            'rto_hours': 0.25,  # 15 minutes
            'rpo_hours': 0.016,  # ~1 minute
            'cost_factor': 5.0,
            'description': 'Infra réduite active + réplication temps réel'
        },
        'hot_standby': {
            'rto_hours': 0,  # Immédiat
            'rpo_hours': 0,
            'cost_factor': 10.0,
            'description': 'Infra complète active-active multi-région'
        }
    }

    @staticmethod
    def calculate_data_loss(rpo_hours, transactions_per_hour):
        """Calculer la perte de données potentielle"""
        transactions_lost = rpo_hours * transactions_per_hour
        return transactions_lost

    @staticmethod
    def calculate_downtime_cost(rto_hours, revenue_per_hour):
        """Calculer le coût de l'indisponibilité"""
        direct_cost = rto_hours * revenue_per_hour

        # Coûts indirects (réputation, clients perdus)
        # Estimation : 3x le coût direct pour > 1h downtime
        indirect_multiplier = 3 if rto_hours > 1 else 1

        total_cost = direct_cost * indirect_multiplier

        return {
            'direct': direct_cost,
            'indirect': direct_cost * (indirect_multiplier - 1),
            'total': total_cost
        }

    @staticmethod
    def recommend_strategy(required_rto_hours, required_rpo_hours, monthly_revenue):
        """Recommander une stratégie DR selon les besoins"""
        recommendations = []

        for name, strategy in DisasterRecoveryPlanner.STRATEGIES.items():
            if (strategy['rto_hours'] <= required_rto_hours and
                strategy['rpo_hours'] <= required_rpo_hours):

                # Calculer le coût mensuel estimé
                base_cost = monthly_revenue * 0.1  # 10% du revenu comme base
                strategy_cost = base_cost * strategy['cost_factor']

                recommendations.append({
                    'name': name,
                    'rto_hours': strategy['rto_hours'],
                    'rpo_hours': strategy['rpo_hours'],
                    'description': strategy['description'],
                    'estimated_monthly_cost': strategy_cost
                })

        # Trier par coût croissant
        recommendations.sort(key=lambda x: x['estimated_monthly_cost'])

        return recommendations

# Usage
planner = DisasterRecoveryPlanner()

# Entreprise e-commerce : $1M/mois, 1000 transactions/heure
# Besoin : RTO < 1h, RPO < 30min

recommendations = planner.recommend_strategy(
    required_rto_hours=1,
    required_rpo_hours=0.5,
    monthly_revenue=1000000
)

print("Stratégies DR recommandées :")
for rec in recommendations:
    print(f"\n{rec['name'].upper()}")
    print(f"  RTO: {rec['rto_hours']}h | RPO: {rec['rpo_hours']}h")
    print(f"  Coût estimé: ${rec['estimated_monthly_cost']:,.0f}/mois")
    print(f"  {rec['description']}")

# Calculer le coût d'une panne de 2h
downtime_cost = planner.calculate_downtime_cost(
    rto_hours=2,
    revenue_per_hour=1000000 / 730  # Revenu par heure
)

print(f"\nCoût d'une panne de 2h :")
print(f"  Direct: ${downtime_cost['direct']:,.0f}")
print(f"  Indirect: ${downtime_cost['indirect']:,.0f}")
print(f"  Total: ${downtime_cost['total']:,.0f}")
```

## Monitoring et Alerting HA

### Métriques critiques

```python
from prometheus_client import Counter, Gauge, Histogram, Summary
import time

# Disponibilité
uptime_seconds = Gauge(
    'service_uptime_seconds',
    'Time since service started'
)

availability_ratio = Gauge(
    'service_availability_ratio',
    'Current availability ratio (0-1)'
)

# Incidents
incidents_total = Counter(
    'incidents_total',
    'Total number of incidents',
    ['severity', 'component']
)

incident_duration = Histogram(
    'incident_duration_seconds',
    'Incident resolution time',
    ['severity'],
    buckets=[60, 300, 900, 1800, 3600, 7200, 14400, 28800, 86400]
)

# MTBF / MTTR
mtbf_seconds = Gauge(
    'mtbf_seconds',
    'Mean Time Between Failures'
)

mttr_seconds = Gauge(
    'mttr_seconds',
    'Mean Time To Recovery'
)

# Failovers
failover_total = Counter(
    'failover_total',
    'Total number of failovers',
    ['component', 'reason']
)

failover_duration = Histogram(
    'failover_duration_seconds',
    'Failover completion time',
    buckets=[1, 5, 10, 30, 60, 120, 300]
)

class HAMonitor:
    """
    Monitoring haute disponibilité
    """

    def __init__(self):
        self.start_time = time.time()
        self.incidents = []
        self.failovers = []

    def record_incident(self, severity, component, duration):
        """Enregistrer un incident"""
        incidents_total.labels(
            severity=severity,
            component=component
        ).inc()

        incident_duration.labels(severity=severity).observe(duration)

        self.incidents.append({
            'time': time.time(),
            'severity': severity,
            'component': component,
            'duration': duration
        })

        self.update_mtbf_mttr()

    def record_failover(self, component, reason, duration):
        """Enregistrer un failover"""
        failover_total.labels(
            component=component,
            reason=reason
        ).inc()

        failover_duration.observe(duration)

        self.failovers.append({
            'time': time.time(),
            'component': component,
            'reason': reason,
            'duration': duration
        })

    def update_mtbf_mttr(self):
        """Mettre à jour MTBF et MTTR"""
        if len(self.incidents) < 2:
            return

        # MTBF : temps moyen entre défaillances
        incident_times = [inc['time'] for inc in self.incidents]
        time_between_failures = []

        for i in range(1, len(incident_times)):
            time_between_failures.append(
                incident_times[i] - incident_times[i-1]
            )

        avg_mtbf = sum(time_between_failures) / len(time_between_failures)
        mtbf_seconds.set(avg_mtbf)

        # MTTR : temps moyen de résolution
        durations = [inc['duration'] for inc in self.incidents]
        avg_mttr = sum(durations) / len(durations)
        mttr_seconds.set(avg_mttr)

    def calculate_availability(self, period_seconds=86400):
        """
        Calculer la disponibilité sur une période
        """
        now = time.time()
        period_start = now - period_seconds

        # Incidents dans la période
        relevant_incidents = [
            inc for inc in self.incidents
            if inc['time'] >= period_start
        ]

        # Total downtime
        total_downtime = sum(inc['duration'] for inc in relevant_incidents)

        # Disponibilité
        availability = (period_seconds - total_downtime) / period_seconds

        availability_ratio.set(availability)

        return availability

    def get_sla_compliance(self, target_availability=0.999):
        """Vérifier le respect du SLA"""
        current = self.calculate_availability()

        return {
            'target': target_availability,
            'current': current,
            'compliant': current >= target_availability,
            'margin': current - target_availability
        }

# Usage
monitor = HAMonitor()

# Simuler des incidents
monitor.record_incident('critical', 'database', duration=300)  # 5 min
monitor.record_incident('warning', 'cache', duration=60)  # 1 min
monitor.record_failover('loadbalancer', 'health_check_failed', duration=5)

# Vérifier SLA
sla = monitor.get_sla_compliance(target_availability=0.999)
print(f"SLA Compliance: {sla['compliant']}")
print(f"Current availability: {sla['current']:.5f} ({sla['current']*100:.3f}%)")
print(f"Margin: {sla['margin']:.5f}")
```

### Alerting intelligent

```python
import requests
from datetime import datetime, timedelta

class SmartAlerting:
    """
    Système d'alerting avec suppression de bruit
    """

    def __init__(self, webhook_url):
        self.webhook_url = webhook_url
        self.alerts = {}  # Component → dernière alerte
        self.alert_threshold = 3  # 3 occurrences
        self.time_window = 300  # 5 minutes

    def should_alert(self, component, severity):
        """
        Déterminer si on doit alerter (éviter alert fatigue)
        """
        now = datetime.utcnow()
        key = f"{component}:{severity}"

        if key not in self.alerts:
            self.alerts[key] = {'count': 0, 'first_seen': now, 'last_alerted': None}

        alert_info = self.alerts[key]
        alert_info['count'] += 1

        # Réinitialiser si hors fenêtre
        if (now - alert_info['first_seen']).total_seconds() > self.time_window:
            alert_info['count'] = 1
            alert_info['first_seen'] = now

        # Alerter si threshold atteint
        if alert_info['count'] >= self.alert_threshold:
            # Éviter les alertes répétées trop rapprochées
            if (alert_info['last_alerted'] and
                (now - alert_info['last_alerted']).total_seconds() < 3600):
                return False  # Déjà alerté dans la dernière heure

            alert_info['last_alerted'] = now
            alert_info['count'] = 0  # Reset
            return True

        return False

    def send_alert(self, component, severity, message, metadata=None):
        """Envoyer une alerte (Slack, PagerDuty, etc.)"""

        if not self.should_alert(component, severity):
            print(f"ℹ️  Alerte supprimée (bruit) : {component}")
            return

        # Déterminer l'urgence
        if severity == 'critical':
            emoji = '🔴'
            ping = '@channel'
        elif severity == 'warning':
            emoji = '⚠️'
            ping = '@here'
        else:
            emoji = 'ℹ️'
            ping = ''

        # Message Slack
        slack_message = {
            'text': f"{emoji} {ping} {component.upper()} - {severity.upper()}",
            'blocks': [
                {
                    'type': 'header',
                    'text': {
                        'type': 'plain_text',
                        'text': f"{emoji} {component.upper()}"
                    }
                },
                {
                    'type': 'section',
                    'fields': [
                        {
                            'type': 'mrkdwn',
                            'text': f"*Severity:*\n{severity}"
                        },
                        {
                            'type': 'mrkdwn',
                            'text': f"*Time:*\n{datetime.utcnow().isoformat()}"
                        }
                    ]
                },
                {
                    'type': 'section',
                    'text': {
                        'type': 'mrkdwn',
                        'text': message
                    }
                }
            ]
        }

        if metadata:
            slack_message['blocks'].append({
                'type': 'section',
                'text': {
                    'type': 'mrkdwn',
                    'text': f"```{metadata}```"
                }
            })

        # Envoyer
        try:
            response = requests.post(self.webhook_url, json=slack_message, timeout=5)
            response.raise_for_status()
            print(f"✅ Alerte envoyée : {component} - {severity}")
        except Exception as e:
            print(f"❌ Échec envoi alerte : {e}")

# Usage
alerting = SmartAlerting(webhook_url='https://hooks.slack.com/services/...')

# Simuler des événements
for i in range(5):
    # 5 erreurs rapides → seulement 1 alerte envoyée
    alerting.send_alert(
        component='database',
        severity='warning',
        message='Connection pool exhausted',
        metadata={'active_connections': 95, 'max_connections': 100}
    )
    time.sleep(1)
```

## Pièges courants et solutions

### Piège 1 : Split-Brain

**Problème :** Deux nœuds se croient tous les deux PRIMARY.

```
Node 1 : Je suis PRIMARY ! ✅
Node 2 : Je suis PRIMARY ! ✅

→ Deux VIP actives
→ Corruption de données
→ Chaos total
```

**Solutions :**

1. **Quorum** (majorité requise)
2. **Fencing/STONITH** (tuer physiquement le nœud suspect)
3. **Witness node** (arbitre tiers)

```python
class QuorumManager:
    """
    Gestion de quorum pour éviter split-brain
    """

    def __init__(self, node_id, all_nodes):
        self.node_id = node_id
        self.all_nodes = all_nodes
        self.votes = {node: False for node in all_nodes}
        self.votes[node_id] = True  # Je vote pour moi-même

    def has_quorum(self):
        """
        Vérifier si on a le quorum (majorité stricte)
        """
        votes_count = sum(1 for v in self.votes.values() if v)
        required = (len(self.all_nodes) // 2) + 1

        return votes_count >= required

    def can_become_primary(self):
        """
        Peut-on devenir PRIMARY en toute sécurité ?
        """
        if not self.has_quorum():
            print(f"❌ Pas de quorum ({sum(self.votes.values())}/{len(self.all_nodes)})")
            return False

        print(f"✅ Quorum OK ({sum(self.votes.values())}/{len(self.all_nodes)})")
        return True

# Exemple avec 3 nœuds
cluster = ['node1', 'node2', 'node3']

# Node1 peut voir node2 mais pas node3
node1 = QuorumManager('node1', cluster)
node1.votes['node2'] = True
node1.votes['node3'] = False  # Network partition

# Node1 a 2/3 votes → quorum OK → peut devenir PRIMARY
print(f"Node1 peut devenir PRIMARY : {node1.can_become_primary()}")

# Node3 seul ne voit personne
node3 = QuorumManager('node3', cluster)
node3.votes['node1'] = False
node3.votes['node2'] = False

# Node3 a 1/3 votes → pas de quorum → reste STANDBY
print(f"Node3 peut devenir PRIMARY : {node3.can_become_primary()}")
```

### Piège 2 : Cascading Failures

**Problème :** Une défaillance en déclenche d'autres en cascade.

```
DB ralentit → Apps timeout → Load balancer marque apps down
→ Trafic redirigé vers apps restantes → Surcharge → Elles tombent aussi
→ Effet domino → Tout le système down
```

**Solutions :**

1. **Circuit breakers** (déjà vu)
2. **Bulkheads** (isolation)
3. **Graceful degradation**
4. **Rate limiting agressif**

```python
class GracefulDegradation:
    """
    Dégradation gracieuse plutôt que défaillance totale
    """

    def __init__(self):
        self.degraded_mode = False
        self.degradation_reasons = []

    def check_dependencies(self):
        """Vérifier l'état des dépendances"""
        issues = []

        # Vérifier DB
        if not self.check_database():
            issues.append('database_slow')

        # Vérifier cache
        if not self.check_cache():
            issues.append('cache_down')

        # Vérifier API externe
        if not self.check_external_api():
            issues.append('external_api_down')

        if issues:
            self.enter_degraded_mode(issues)
        else:
            self.exit_degraded_mode()

    def enter_degraded_mode(self, reasons):
        """Entrer en mode dégradé"""
        if not self.degraded_mode:
            print(f"⚠️  ENTERING DEGRADED MODE: {reasons}")

        self.degraded_mode = True
        self.degradation_reasons = reasons

    def exit_degraded_mode(self):
        """Sortir du mode dégradé"""
        if self.degraded_mode:
            print(f"✅ EXITING DEGRADED MODE")

        self.degraded_mode = False
        self.degradation_reasons = []

    def serve_request(self, request_type):
        """
        Servir une requête avec dégradation si nécessaire
        """
        if not self.degraded_mode:
            # Mode normal
            return self.serve_full_request(request_type)

        # Mode dégradé → fonctionnalités réduites
        if 'database_slow' in self.degradation_reasons:
            # DB lente → servir du cache stale
            return self.serve_from_stale_cache(request_type)

        if 'cache_down' in self.degradation_reasons:
            # Cache down → DB directement (avec rate limit)
            return self.serve_from_database_limited(request_type)

        if 'external_api_down' in self.degradation_reasons:
            # API externe down → réponse par défaut
            return self.serve_default_response(request_type)

    def serve_full_request(self, request_type):
        """Réponse complète normale"""
        return {'mode': 'full', 'data': '...'}

    def serve_from_stale_cache(self, request_type):
        """Servir du cache expiré (mieux que rien)"""
        return {'mode': 'degraded_stale_cache', 'data': '...', 'warning': 'Data may be outdated'}

    def serve_from_database_limited(self, request_type):
        """Accès DB limité"""
        return {'mode': 'degraded_limited', 'data': '...', 'warning': 'Limited functionality'}

    def serve_default_response(self, request_type):
        """Réponse par défaut"""
        return {'mode': 'degraded_default', 'message': 'Service temporarily limited'}
```

### Piège 3 : Manque de tests de failover

**Problème :** Le failover n'a jamais été testé → échoue en production.

**Solution :** **Chaos Engineering**

```python
import random
import subprocess

class ChaosMonkey:
    """
    Chaos Engineering : tester la résilience
    """

    def __init__(self, cluster_nodes):
        self.nodes = cluster_nodes
        self.enabled = False

    def enable(self):
        """Activer le chaos monkey"""
        self.enabled = True
        print("🐵 CHAOS MONKEY ENABLED")

    def disable(self):
        """Désactiver"""
        self.enabled = False
        print("🐵 Chaos Monkey disabled")

    def kill_random_pod(self):
        """Tuer un pod aléatoire (Kubernetes)"""
        if not self.enabled:
            return

        pod = random.choice(self.nodes)

        print(f"🐵 Killing pod: {pod}")

        subprocess.run([
            'kubectl', 'delete', 'pod', pod,
            '--force', '--grace-period=0'
        ])

    def introduce_latency(self, target_service, latency_ms=1000):
        """Introduire de la latence réseau"""
        if not self.enabled:
            return

        print(f"🐵 Adding {latency_ms}ms latency to {target_service}")

        # Via tc (traffic control)
        subprocess.run([
            'tc', 'qdisc', 'add', 'dev', 'eth0',
            'root', 'netem', 'delay', f'{latency_ms}ms'
        ])

    def fill_disk(self, target_node, size_mb=1000):
        """Remplir le disque"""
        if not self.enabled:
            return

        print(f"🐵 Filling disk on {target_node} with {size_mb}MB")

        subprocess.run([
            'dd', 'if=/dev/zero',
            f'of=/tmp/chaos_{random.randint(1000,9999)}.img',
            'bs=1M', f'count={size_mb}'
        ])

    def partition_network(self, node1, node2):
        """Créer une partition réseau entre deux nœuds"""
        if not self.enabled:
            return

        print(f"🐵 Network partition: {node1} ↔️ {node2}")

        # Via iptables
        subprocess.run([
            'iptables', '-A', 'INPUT',
            '-s', node2, '-j', 'DROP'
        ])

# Usage (en staging/test seulement !)
chaos = ChaosMonkey(['pod-1', 'pod-2', 'pod-3'])

if environment == 'staging':
    chaos.enable()

    # Tuer un pod au hasard toutes les heures
    schedule.every(1).hours.do(chaos.kill_random_pod)

    # Tester résilience
    chaos.kill_random_pod()
    time.sleep(60)  # Observer la récupération
```

## Checklist : Êtes-vous vraiment HA ?

```markdown
## Checklist Haute Disponibilité

### Architecture
- [ ] Aucun SPOF (Single Point of Failure)
- [ ] Redondance N+1 minimum (N+2 recommandé)
- [ ] Multi-AZ (Availability Zones)
- [ ] Multi-région pour services critiques
- [ ] Load balancing avec health checks

### Données
- [ ] Réplication synchrone ou asynchrone selon RPO
- [ ] Backups automatiques quotidiens minimum
- [ ] Backups testés régulièrement (restore drill)
- [ ] Backups multi-sites (3-2-1 rule)
- [ ] Plan de disaster recovery documenté

### Réseau
- [ ] Liens réseau redondants
- [ ] Multi-homing (plusieurs ISP)
- [ ] BGP failover configuré
- [ ] DDoS protection active

### Application
- [ ] Circuit breakers implémentés
- [ ] Retry avec backoff exponentiel
- [ ] Timeouts configurés partout
- [ ] Graceful shutdown (drain period)
- [ ] Health check endpoints (/live, /ready)
- [ ] Sessions externalisées (Redis/DB)

### Monitoring
- [ ] Métriques de disponibilité (uptime, latency, errors)
- [ ] Alerting intelligent (éviter alert fatigue)
- [ ] SLO/SLA définis et mesurés
- [ ] MTBF/MTTR trackés
- [ ] Dashboards temps réel

### Opérations
- [ ] Runbooks pour incidents courants
- [ ] On-call rotation définie
- [ ] Post-mortems après incidents
- [ ] Chaos engineering pratiqué
- [ ] Failover testé mensuellement
- [ ] Disaster recovery drill annuel

### Sécurité
- [ ] Secrets rotation automatique
- [ ] Backups chiffrés
- [ ] Accès bastion/jump hosts
- [ ] Audit logs centralisés

### Score : __ / 34

< 20 : Faible disponibilité
20-27 : Bonne disponibilité
28-32 : Haute disponibilité
33-34 : Ultra haute disponibilité
```

## Résumé : Concevoir pour la HA

### Principes clés

- ✅ **Éliminer tous les SPOF** → Redondance partout
- ✅ **Automatiser le failover** → Pas d'intervention humaine
- ✅ **Isoler les défaillances** → Bulkheads, circuit breakers
- ✅ **Dégrader gracieusement** → Mieux que down complet
- ✅ **Monitorer en continu** → Détecter avant que ça empire
- ✅ **Tester régulièrement** → Chaos engineering
- ✅ **Documenter tout** → Runbooks, post-mortems

### Métriques de succès

| Métrique | Formule | Objectif |
|----------|---------|----------|
| **Uptime** | (Total - Downtime) / Total | > 99.9% |
| **MTBF** | Temps total / Nb défaillances | > 720h |
| **MTTR** | Total downtime / Nb incidents | < 30min |
| **RTO** | Temps de récupération | < 1h |
| **RPO** | Perte de données max | < 15min |

### Coûts typiques

| Niveau HA | Coût infrastructure | Complexité | Use case |
|-----------|---------------------|------------|----------|
| **Basic (99%)** | 1× | Faible | Dev, staging |
| **Standard (99.9%)** | 2× | Moyenne | PME, apps standard |
| **High (99.99%)** | 3-5× | Élevée | E-commerce, SaaS |
| **Ultra (99.999%)** | 10×+ | Très élevée | Banques, santé |

## Conclusion

La haute disponibilité n'est pas un produit qu'on achète, c'est une **discipline** qu'on pratique. Chaque composant, chaque ligne de code, chaque décision architecturale doit être pensée avec la question : **"Que se passe-t-il si ça tombe ?"**

**À retenir :**
- Commencer par **mesurer** votre disponibilité actuelle
- **Éliminer les SPOF** un par un
- Implémenter patterns de **résilience** (circuit breaker, retry, bulkhead)
- **Automatiser** le failover et les backups
- **Tester** régulièrement vos scénarios de panne
- **Documenter** vos runbooks et post-mortems
- Améliorer **continuellement** via les leçons apprises

**Prochaine étape :** La section suivante explore les **Conteneurs et réseaux (Docker, Kubernetes)**, qui facilitent grandement la mise en place d'architectures hautement disponibles.

---


⏭️ [Conteneurs et réseaux (Docker, Kubernetes networking)](/09-architectures-avancees/07-conteneurs-reseaux.md)
