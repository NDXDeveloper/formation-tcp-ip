🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.4 Gestion des erreurs réseau

## Introduction

Le réseau est **intrinsèquement peu fiable**. Contrairement aux opérations locales (lecture de fichier, calculs), les opérations réseau peuvent échouer de multiples façons : serveur indisponible, connexion interrompue, timeout, paquets perdus, DNS qui échoue, firewall qui bloque...

Un développeur qui ne gère pas correctement les erreurs réseau crée des applications **fragiles** qui crashent, perdent des données, ou se comportent de manière imprévisible. Cette section vous apprendra à construire des applications **résilientes** capables de gérer gracieusement les défaillances réseau.

## Pourquoi la gestion d'erreurs est critique

### Scénarios du monde réel

**Exemple 1 : E-commerce**
```python
# Code DANGEREUX sans gestion d'erreurs
def process_payment(order_id, amount):
    response = payment_api.charge(order_id, amount)
    db.mark_as_paid(order_id)
    return response

# Que se passe-t-il si :
# - payment_api timeout ?
# - La réponse est perdue en chemin ?
# - Le paiement passe mais la confirmation n'arrive pas ?
# → Le client peut être facturé sans que la commande soit marquée payée
# → Ou pire : double facturation lors du retry
```

**Exemple 2 : Service distribué**
```python
# Service A appelle Service B qui appelle Service C
ServiceA → ServiceB → ServiceC

# Si ServiceC est lent (10 secondes) :
# - ServiceB attend 10 secondes (bloqué)
# - ServiceA attend 10 secondes (bloqué)
# - Les threads/workers sont tous bloqués
# - Cascade de défaillances : tout le système down
# → Un service lent peut mettre tout le système à genoux
```

**Exemple 3 : Mobile app**
```python
# Application mobile qui ne gère pas les erreurs réseau
def load_user_profile():
    profile = api.get("/user/me")  # Pas de timeout
    display(profile)

# Utilisateur dans le métro (réseau instable) :
# - L'app freeze indéfiniment
# - Pas de feedback à l'utilisateur
# - L'utilisateur force-quit l'app
# → Mauvaise expérience utilisateur, désinstallation
```

### Le coût de l'ignorance des erreurs

| Conséquence | Impact business |
|-------------|----------------|
| **Crash** | Perte de service, insatisfaction client |
| **Données perdues** | Transactions manquées, perte financière |
| **Blocage** | Ressources gaspillées, scalabilité impossible |
| **Incohérence** | Données corrompues, bugs complexes |
| **Cascade** | Tout le système tombe |

## Taxonomie des erreurs réseau

### 1. Erreurs de résolution

**Problème** : Impossible de résoudre le nom de domaine en adresse IP

```python
import socket

try:
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('nonexistent-domain-xyz.com', 80))
except socket.gaierror as e:
    # getaddrinfo() error
    print(f"Erreur DNS: {e}")
    # socket.gaierror: [Errno 8] nodename nor servname provided, or not known
```

**Causes** :
- Domaine inexistant
- Serveur DNS inaccessible
- Problème de configuration réseau
- Pas de connexion internet

**Gestion appropriée** :
```python
import socket
import time

def resolve_with_retry(hostname, max_retries=3, delay=1.0):
    """Résout un nom de domaine avec retry"""
    for attempt in range(max_retries):
        try:
            # getaddrinfo retourne une liste de tuples
            info = socket.getaddrinfo(hostname, None)
            # Extraire la première adresse IPv4
            for res in info:
                if res[0] == socket.AF_INET:  # IPv4
                    return res[4][0]  # IP address

        except socket.gaierror as e:
            if attempt < max_retries - 1:
                print(f"Tentative {attempt + 1}/{max_retries} échouée, retry dans {delay}s...")
                time.sleep(delay)
            else:
                raise RuntimeError(f"Impossible de résoudre {hostname}: {e}")

    raise RuntimeError(f"Pas d'adresse IPv4 pour {hostname}")

# Utilisation
try:
    ip = resolve_with_retry('example.com')
    print(f"Résolu: {ip}")
except RuntimeError as e:
    # Fallback : utiliser une IP de backup ou afficher un message
    print(f"Erreur: {e}")
```

### 2. Erreurs de connexion

**Problème** : Impossible d'établir la connexion TCP

#### Connection Refused

```python
import socket

try:
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('localhost', 9999))  # Aucun serveur n'écoute
except ConnectionRefusedError as e:
    print(f"Connexion refusée: {e}")
    # [Errno 61] Connection refused
```

**Signification** : Un serveur répond activement "non" (RST)
- Port fermé
- Service arrêté
- Firewall avec RST actif

**Gestion** :
```python
def connect_with_validation(host, port, timeout=5.0):
    """Connexion avec validation et message clair"""
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        sock.connect((host, port))
        return sock

    except ConnectionRefusedError:
        raise RuntimeError(
            f"Le serveur {host}:{port} refuse activement les connexions. "
            f"Vérifiez que le service est démarré."
        )

    except socket.timeout:
        raise RuntimeError(
            f"Timeout lors de la connexion à {host}:{port}. "
            f"Le serveur est peut-être injoignable ou surchargé."
        )

    except OSError as e:
        raise RuntimeError(f"Erreur de connexion à {host}:{port}: {e}")
```

#### Connection Timeout

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.settimeout(3.0)

try:
    sock.connect(('192.168.1.100', 80))  # IP injoignable
except socket.timeout:
    print("Timeout : serveur injoignable")
    # Causes possibles :
    # - Serveur down
    # - Firewall drop (pas de réponse)
    # - Problème réseau
```

**Différence Connection Refused vs Timeout** :

```
Connection Refused (RST) :
Client → SYN → Serveur
Client ← RST ← Serveur  (refus actif)
→ Réponse immédiate (< 1 ms)

Connection Timeout :
Client → SYN → [rien]
Client → SYN → [rien]  (retry)
Client → SYN → [rien]  (retry)
→ Timeout après plusieurs secondes
```

### 3. Erreurs pendant la communication

#### Connection Reset

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('example.com', 80))

try:
    sock.send(b'GET / HTTP/1.1\r\n\r\n')
    data = sock.recv(4096)
except ConnectionResetError:
    print("Connexion fermée brutalement par le serveur")
    # Le serveur a envoyé un RST
    # Causes :
    # - Crash du serveur
    # - Timeout serveur
    # - Firewall intermédiaire
    # - Requête invalide
```

**Gestion robuste** :
```python
def send_with_error_handling(sock, data, max_retries=3):
    """Envoi avec gestion des erreurs"""
    for attempt in range(max_retries):
        try:
            sock.sendall(data)
            return True

        except ConnectionResetError:
            if attempt < max_retries - 1:
                print(f"Connexion reset, tentative de reconnexion...")
                # Créer une nouvelle connexion
                # (sock doit être recréé)
                raise  # Pour l'instant, propager l'erreur
            else:
                raise RuntimeError("Connexion perdue après plusieurs tentatives")

        except BrokenPipeError:
            # Tentative d'écriture sur socket fermée
            raise RuntimeError("Socket fermée localement")

        except OSError as e:
            raise RuntimeError(f"Erreur d'envoi: {e}")
```

#### Broken Pipe

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('example.com', 80))
sock.close()  # Fermeture locale

try:
    sock.send(b'data')  # Tentative d'envoi après fermeture
except BrokenPipeError:
    print("Tentative d'écriture sur socket fermée")
```

### 4. Erreurs de timeout

Les timeouts sont essentiels pour éviter les blocages infinis.

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.settimeout(5.0)  # 5 secondes
sock.connect(('example.com', 80))

try:
    sock.send(b'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n')
    data = sock.recv(4096)  # Bloque max 5 secondes
except socket.timeout:
    print("Timeout lors de la réception")
    # Le serveur ne répond pas assez vite
```

**Différents timeouts** :

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 1. Timeout de connexion (connect)
sock.settimeout(3.0)
try:
    sock.connect(('slow-server.com', 80))
except socket.timeout:
    print("Timeout lors de l'établissement de connexion")

# 2. Timeout de réception (recv)
sock.settimeout(10.0)
try:
    data = sock.recv(4096)
except socket.timeout:
    print("Timeout lors de la réception de données")

# 3. Timeout d'envoi (moins courant, géré par l'OS)
# send() bloque rarement, mais peut arriver si le buffer est plein
```

**Cas d'usage : timeouts différenciés**

```python
class APIClient:
    """Client API avec timeouts adaptés"""

    def __init__(self, base_url):
        self.base_url = base_url

    def quick_check(self):
        """Health check rapide"""
        sock = self._connect(timeout=1.0)  # Connexion rapide
        try:
            sock.settimeout(2.0)  # Lecture rapide
            sock.send(b'GET /health HTTP/1.1\r\n\r\n')
            response = sock.recv(1024)
            return b'200 OK' in response
        finally:
            sock.close()

    def fetch_data(self, endpoint):
        """Récupération de données (peut être lent)"""
        sock = self._connect(timeout=5.0)  # Connexion normale
        try:
            sock.settimeout(30.0)  # Lecture plus longue pour gros transferts
            request = f'GET {endpoint} HTTP/1.1\r\n\r\n'
            sock.send(request.encode())
            return self._read_response(sock)
        finally:
            sock.close()

    def upload_file(self, file_path):
        """Upload de fichier (très long)"""
        sock = self._connect(timeout=5.0)
        try:
            sock.settimeout(300.0)  # 5 minutes pour upload
            # ... upload logic
        finally:
            sock.close()
```

### 5. Erreurs spécifiques à UDP

UDP a ses propres problématiques :

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.settimeout(2.0)

try:
    sock.sendto(b'request', ('192.168.1.100', 9999))
    data, addr = sock.recvfrom(4096)
except socket.timeout:
    print("Pas de réponse UDP (normal : aucune garantie)")
    # Avec UDP :
    # - Le paquet peut être perdu
    # - La réponse peut être perdue
    # - Impossible de savoir ce qui s'est passé
```

**Pattern : retry avec exponential backoff pour UDP**

```python
import socket
import time
import random

def udp_request_with_retry(sock, data, dest, max_retries=5):
    """Requête UDP avec retry et exponential backoff"""
    base_timeout = 1.0

    for attempt in range(max_retries):
        try:
            # Timeout croissant avec jitter
            timeout = base_timeout * (2 ** attempt) + random.uniform(0, 0.5)
            sock.settimeout(min(timeout, 30.0))  # Max 30 secondes

            # Envoi
            sock.sendto(data, dest)
            print(f"Tentative {attempt + 1}, timeout={timeout:.1f}s")

            # Réception
            response, addr = sock.recvfrom(4096)
            return response

        except socket.timeout:
            if attempt < max_retries - 1:
                print(f"  Timeout, retry...")
            else:
                raise RuntimeError(f"Aucune réponse après {max_retries} tentatives")

# Utilisation
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
try:
    response = udp_request_with_retry(
        sock,
        b'DNS query',
        ('8.8.8.8', 53),
        max_retries=3
    )
    print(f"Réponse reçue: {len(response)} octets")
except RuntimeError as e:
    print(f"Erreur: {e}")
```

## Codes d'erreur système (errno)

### Principaux codes d'erreur réseau

Sur UNIX/Linux, les erreurs réseau sont indiquées par `errno` :

```python
import socket
import errno

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    sock.connect(('192.168.1.100', 80))
except OSError as e:
    # Accès au code d'erreur
    error_code = e.errno

    if error_code == errno.ECONNREFUSED:
        print("Connection refused (61)")
    elif error_code == errno.ETIMEDOUT:
        print("Connection timeout (60)")
    elif error_code == errno.EHOSTUNREACH:
        print("No route to host (65)")
    elif error_code == errno.ENETUNREACH:
        print("Network unreachable (51)")
    elif error_code == errno.EADDRINUSE:
        print("Address already in use (48)")
    else:
        print(f"Erreur {error_code}: {e}")
```

**Tableau des erreurs courantes** :

| errno | Nom | Signification | Action recommandée |
|-------|-----|---------------|-------------------|
| 61 | ECONNREFUSED | Connection refused | Vérifier que le service écoute |
| 60 | ETIMEDOUT | Timeout | Retry avec backoff |
| 65 | EHOSTUNREACH | Host unreachable | Vérifier routage/firewall |
| 51 | ENETUNREACH | Network unreachable | Problème réseau local |
| 48 | EADDRINUSE | Address in use | SO_REUSEADDR ou changer de port |
| 54 | ECONNRESET | Connection reset | Reconnexion |
| 32 | EPIPE | Broken pipe | Socket fermée, recréer |
| 57 | ENOTCONN | Not connected | Établir connexion d'abord |

### Gestion unifiée des erreurs

```python
import socket
import errno

class NetworkError(Exception):
    """Base pour toutes les erreurs réseau"""
    pass

class ConnectionFailedError(NetworkError):
    """Impossible de se connecter"""
    pass

class TimeoutError(NetworkError):
    """Timeout réseau"""
    pass

class DisconnectedError(NetworkError):
    """Connexion perdue"""
    pass

def handle_socket_error(error):
    """Convertit OSError en exception métier"""
    if isinstance(error, socket.timeout):
        raise TimeoutError("Opération timeout")

    if not isinstance(error, OSError):
        raise error

    error_code = error.errno

    if error_code == errno.ECONNREFUSED:
        raise ConnectionFailedError(
            "Le serveur refuse les connexions. "
            "Vérifiez qu'il est démarré et accessible."
        )

    elif error_code == errno.ETIMEDOUT:
        raise TimeoutError(
            "Timeout lors de la connexion. "
            "Le serveur est peut-être surchargé ou injoignable."
        )

    elif error_code in (errno.EHOSTUNREACH, errno.ENETUNREACH):
        raise ConnectionFailedError(
            "Serveur injoignable. "
            "Vérifiez votre connexion réseau et les paramètres firewall."
        )

    elif error_code in (errno.ECONNRESET, errno.EPIPE):
        raise DisconnectedError(
            "Connexion perdue. "
            "Le serveur a fermé la connexion ou est tombé."
        )

    else:
        raise NetworkError(f"Erreur réseau: {error}")

# Utilisation
def connect_safely(host, port):
    """Connexion avec gestion d'erreurs claire"""
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(5.0)
        sock.connect((host, port))
        return sock

    except (socket.timeout, OSError) as e:
        handle_socket_error(e)

# Dans le code métier
try:
    sock = connect_safely('api.example.com', 443)
except ConnectionFailedError as e:
    # Log et affichage utilisateur
    logger.error(f"Connexion impossible: {e}")
    show_user_message("Service temporairement indisponible")
except TimeoutError as e:
    logger.warning(f"Timeout: {e}")
    show_user_message("Connexion lente, veuillez patienter")
```

## Stratégies de gestion d'erreurs

### 1. Fail Fast vs Fail Safe

**Fail Fast** : Échouer immédiatement et proprement

```python
def fetch_critical_data(url):
    """Données critiques : pas de retry, fail fast"""
    try:
        response = http_get(url, timeout=3.0)
        return response.json()
    except Exception as e:
        # Log l'erreur et propage
        logger.critical(f"Impossible de récupérer les données critiques: {e}")
        raise  # Propage l'erreur

# Utilisé quand :
# - Les données sont essentielles au fonctionnement
# - Un retry n'aiderait pas
# - Il vaut mieux échouer que continuer avec des données invalides
```

**Fail Safe** : Continuer avec des valeurs par défaut

```python
def fetch_optional_config(url, default=None):
    """Configuration optionnelle : fail safe"""
    try:
        response = http_get(url, timeout=2.0)
        return response.json()
    except Exception as e:
        logger.warning(f"Impossible de récupérer la config, utilisation du défaut: {e}")
        return default or {}

# Utilisé quand :
# - Les données sont optionnelles
# - On peut fonctionner avec une valeur par défaut
# - La disponibilité prime sur l'exactitude
```

### 2. Retry Logic (Logique de Nouvelle Tentative)

#### Retry Simple

```python
import time

def retry_simple(func, max_attempts=3, delay=1.0):
    """Retry simple avec délai fixe"""
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt < max_attempts - 1:
                print(f"Tentative {attempt + 1} échouée: {e}, retry dans {delay}s")
                time.sleep(delay)
            else:
                print(f"Échec après {max_attempts} tentatives")
                raise

# Utilisation
def unreliable_network_call():
    response = requests.get('https://api.example.com/data', timeout=5)
    return response.json()

data = retry_simple(unreliable_network_call, max_attempts=3, delay=2.0)
```

#### Exponential Backoff

**Principe** : Augmenter progressivement le délai entre les tentatives

```python
import time
import random

def retry_exponential_backoff(
    func,
    max_attempts=5,
    base_delay=1.0,
    max_delay=60.0,
    exponential_base=2,
    jitter=True
):
    """
    Retry avec exponential backoff et jitter

    Délais : 1s, 2s, 4s, 8s, 16s...
    Jitter : ajoute du hasard pour éviter les "thundering herds"
    """
    for attempt in range(max_attempts):
        try:
            return func()

        except Exception as e:
            if attempt < max_attempts - 1:
                # Calcul du délai exponentiel
                delay = min(base_delay * (exponential_base ** attempt), max_delay)

                # Ajout de jitter (±50%)
                if jitter:
                    delay = delay * (0.5 + random.random())

                print(f"Tentative {attempt + 1}/{max_attempts} échouée: {e}")
                print(f"  Retry dans {delay:.2f}s...")
                time.sleep(delay)
            else:
                raise RuntimeError(
                    f"Échec après {max_attempts} tentatives: {e}"
                ) from e

# Utilisation
def flaky_api_call():
    # API qui échoue parfois
    return requests.get('https://flaky-api.com/data', timeout=10).json()

try:
    data = retry_exponential_backoff(flaky_api_call, max_attempts=5)
    print(f"Succès: {data}")
except RuntimeError as e:
    print(f"Échec définitif: {e}")
```

**Pourquoi le jitter est important ?**

```
Sans jitter (tous les clients retry en même temps) :
t=0s  : 1000 clients → Serveur saturé → Tous échouent
t=1s  : 1000 clients retry → Serveur saturé → Tous échouent
t=2s  : 1000 clients retry → Serveur saturé → Tous échouent
→ "Thundering herd problem"

Avec jitter (délais aléatoires) :
t=0s  : 1000 clients → Échec
t=1-2s: ~300 clients retry → Certains réussissent
t=2-4s: ~400 clients retry → Plus réussissent
t=4-8s: ~300 clients retry → Majorité réussit
→ Charge étalée dans le temps
```

#### Retry avec conditions spécifiques

```python
import requests
from requests.exceptions import RequestException, Timeout, ConnectionError

def should_retry(exception):
    """Détermine si un retry est approprié"""
    # Retry sur erreurs réseau temporaires
    if isinstance(exception, (Timeout, ConnectionError)):
        return True

    # Retry sur certains codes HTTP
    if isinstance(exception, RequestException):
        if hasattr(exception, 'response') and exception.response:
            status = exception.response.status_code
            # Retry sur erreurs serveur (5xx) et rate limiting (429)
            if status in (429, 500, 502, 503, 504):
                return True

    # Pas de retry sur erreurs client (4xx) sauf 429
    return False

def retry_smart(func, max_attempts=3):
    """Retry seulement sur erreurs temporaires"""
    for attempt in range(max_attempts):
        try:
            return func()

        except Exception as e:
            if should_retry(e) and attempt < max_attempts - 1:
                delay = 2 ** attempt  # 1s, 2s, 4s
                print(f"Erreur temporaire, retry dans {delay}s: {e}")
                time.sleep(delay)
            else:
                # Erreur permanente ou dernier essai
                raise

# Utilisation
def api_call():
    response = requests.get('https://api.example.com/users', timeout=5)
    response.raise_for_status()
    return response.json()

try:
    data = retry_smart(api_call)
except requests.HTTPError as e:
    if e.response.status_code == 404:
        print("Ressource non trouvée (pas de retry)")
    elif e.response.status_code in (500, 502, 503):
        print("Serveur en erreur après plusieurs tentatives")
```

### 3. Circuit Breaker Pattern

**Principe** : Arrêter temporairement les appels vers un service défaillant

```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"      # Tout fonctionne
    OPEN = "open"          # Trop d'erreurs, on arrête
    HALF_OPEN = "half_open"  # Test si le service est revenu

class CircuitBreaker:
    """
    Circuit Breaker pour protéger contre les services défaillants

    États :
    - CLOSED : Fonctionnement normal
    - OPEN : Trop d'échecs, bloque les appels (fail fast)
    - HALF_OPEN : Test périodique si le service est revenu
    """

    def __init__(
        self,
        failure_threshold=5,      # Échecs avant ouverture
        recovery_timeout=60.0,    # Temps avant test de récupération
        success_threshold=2       # Succès pour refermer
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.success_threshold = success_threshold

        self.failure_count = 0
        self.success_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    def call(self, func, *args, **kwargs):
        """Exécute une fonction protégée par le circuit breaker"""
        if self.state == CircuitState.OPEN:
            # Vérifier si on peut passer en HALF_OPEN
            if self._should_attempt_reset():
                self.state = CircuitState.HALF_OPEN
                print("Circuit breaker: passage en HALF_OPEN (test)")
            else:
                # Circuit ouvert, fail fast
                raise RuntimeError(
                    f"Circuit breaker OPEN : service indisponible. "
                    f"Réessayez dans {self._time_until_retry():.0f}s"
                )

        try:
            # Tenter l'appel
            result = func(*args, **kwargs)
            self._on_success()
            return result

        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        """Traite un succès"""
        self.failure_count = 0

        if self.state == CircuitState.HALF_OPEN:
            self.success_count += 1

            if self.success_count >= self.success_threshold:
                # Assez de succès, refermer le circuit
                self.state = CircuitState.CLOSED
                self.success_count = 0
                print("Circuit breaker: FERMÉ (service rétabli)")

    def _on_failure(self):
        """Traite un échec"""
        self.failure_count += 1
        self.last_failure_time = time.time()
        self.success_count = 0

        if self.state == CircuitState.HALF_OPEN:
            # Échec pendant le test, réouvrir
            self.state = CircuitState.OPEN
            print("Circuit breaker: réOUVERT (service toujours down)")

        elif self.failure_count >= self.failure_threshold:
            # Trop d'échecs, ouvrir le circuit
            self.state = CircuitState.OPEN
            print(f"Circuit breaker: OUVERT après {self.failure_count} échecs")

    def _should_attempt_reset(self):
        """Vérifie si on doit tenter de fermer le circuit"""
        return (
            self.last_failure_time is not None and
            time.time() - self.last_failure_time >= self.recovery_timeout
        )

    def _time_until_retry(self):
        """Temps avant le prochain test"""
        if self.last_failure_time is None:
            return 0
        elapsed = time.time() - self.last_failure_time
        return max(0, self.recovery_timeout - elapsed)

# Utilisation
payment_breaker = CircuitBreaker(
    failure_threshold=3,
    recovery_timeout=30.0,
    success_threshold=2
)

def call_payment_api(order_id, amount):
    """Appel à l'API de paiement"""
    # Simuler un appel réseau
    response = requests.post(
        'https://payment-api.com/charge',
        json={'order_id': order_id, 'amount': amount},
        timeout=5.0
    )
    response.raise_for_status()
    return response.json()

# Dans le code métier
for i in range(20):
    try:
        result = payment_breaker.call(call_payment_api, f"order_{i}", 100.0)
        print(f"✓ Paiement {i} réussi")

    except RuntimeError as e:
        # Circuit breaker ouvert
        print(f"✗ Paiement {i} bloqué: {e}")
        # Fallback : mettre en queue pour retry ultérieur

    except Exception as e:
        # Erreur de l'API elle-même
        print(f"✗ Paiement {i} échoué: {e}")

    time.sleep(2)
```

**Cas d'usage réel : Architecture microservices**

```python
# Service A appelle Service B via circuit breaker
class ServiceBClient:
    def __init__(self):
        self.breaker = CircuitBreaker(
            failure_threshold=5,
            recovery_timeout=60.0
        )

    def get_user_info(self, user_id):
        """Récupère les infos utilisateur de Service B"""
        try:
            return self.breaker.call(
                self._fetch_user_info,
                user_id
            )
        except RuntimeError:
            # Circuit ouvert, utiliser un cache ou des données par défaut
            return self._get_cached_user_info(user_id)

    def _fetch_user_info(self, user_id):
        response = requests.get(
            f'http://service-b:8080/users/{user_id}',
            timeout=2.0
        )
        response.raise_for_status()
        return response.json()

    def _get_cached_user_info(self, user_id):
        # Fallback sur cache ou données minimales
        return {'id': user_id, 'name': 'Unknown', 'cached': True}
```

## Timeouts : configuration appropriée

### Principes des timeouts

**Règle d'or** : Toujours définir des timeouts explicites

```python
# ❌ MAUVAIS : timeout infini par défaut
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('example.com', 80))  # Peut bloquer indéfiniment
data = sock.recv(4096)  # Peut bloquer indéfiniment

# ✅ BON : timeouts explicites
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.settimeout(5.0)  # Timeout global
sock.connect(('example.com', 80))  # Max 5s
data = sock.recv(4096)  # Max 5s
```

### Timeouts différenciés

```python
class HTTPClient:
    """Client HTTP avec timeouts adaptés à chaque opération"""

    def __init__(self):
        self.connect_timeout = 3.0    # Connexion rapide
        self.read_timeout = 10.0      # Lecture standard
        self.write_timeout = 5.0      # Écriture rapide

    def request(self, method, url, data=None):
        """Requête avec timeouts différenciés"""
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

        try:
            # Phase 1 : Connexion (timeout court)
            sock.settimeout(self.connect_timeout)
            host, port = self._parse_url(url)
            sock.connect((host, port))

            # Phase 2 : Envoi de la requête (timeout moyen)
            sock.settimeout(self.write_timeout)
            request = self._build_request(method, url, data)
            sock.sendall(request.encode())

            # Phase 3 : Réception de la réponse (timeout long)
            sock.settimeout(self.read_timeout)
            response = self._read_response(sock)

            return response

        finally:
            sock.close()
```

### Calcul de timeouts en cascade

**Problème** : Dans une architecture microservices, les timeouts s'accumulent

```python
"""
Service A (timeout: 10s)
  ├─> Service B (timeout: 8s)
  │     ├─> Service C (timeout: 5s)
  │     └─> Service D (timeout: 5s)
  └─> Service E (timeout: 8s)

Pire cas : 10s à chaque niveau
→ Temps total potentiel : 10 + 8 + 5 = 23s !
"""

# Règle : Timeout parent > somme des timeouts enfants
def calculate_timeout_budget(parent_timeout, num_downstream_calls):
    """
    Distribue le budget de timeout entre les appels en cascade

    Exemple : parent_timeout=10s, 3 appels downstream
    → ~3s par appel + marge
    """
    # Garder une marge de sécurité (20%)
    available = parent_timeout * 0.8

    # Distribuer équitablement
    per_call = available / num_downstream_calls

    return per_call

# Service B calcule ses timeouts en fonction de celui reçu
class ServiceB:
    def handle_request(self, request_timeout=10.0):
        # J'ai 10s au total, je dois appeler C et D
        downstream_timeout = calculate_timeout_budget(request_timeout, 2)
        # → ~4s par appel

        # Appeler Service C
        c_data = self.call_service_c(timeout=downstream_timeout)

        # Appeler Service D
        d_data = self.call_service_d(timeout=downstream_timeout)

        return self.merge_results(c_data, d_data)
```

### Timeout adaptatif

```python
import time
from collections import deque

class AdaptiveTimeout:
    """Ajuste automatiquement le timeout en fonction des latences observées"""

    def __init__(self, initial_timeout=5.0, percentile=95):
        self.percentile = percentile
        self.current_timeout = initial_timeout
        self.latencies = deque(maxlen=100)  # Garde les 100 dernières

    def execute(self, func, *args, **kwargs):
        """Exécute une fonction avec timeout adaptatif"""
        start = time.time()

        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(self.current_timeout)

        try:
            result = func(sock, *args, **kwargs)

            # Enregistrer la latence
            latency = time.time() - start
            self.latencies.append(latency)

            # Ajuster le timeout
            self._adjust_timeout()

            return result

        finally:
            sock.close()

    def _adjust_timeout(self):
        """Ajuste le timeout basé sur les latences observées"""
        if len(self.latencies) < 10:
            return  # Pas assez de données

        # Calculer le percentile (95ème par défaut)
        sorted_latencies = sorted(self.latencies)
        index = int(len(sorted_latencies) * (self.percentile / 100.0))
        p_latency = sorted_latencies[index]

        # Timeout = percentile latency + 50% de marge
        new_timeout = p_latency * 1.5

        # Limites min/max
        self.current_timeout = max(1.0, min(new_timeout, 30.0))

        print(f"Timeout ajusté à {self.current_timeout:.2f}s (P{self.percentile}={p_latency:.2f}s)")
```

## Détection de déconnexions

### Keep-Alive TCP

```python
import socket

def enable_keepalive(sock, idle=60, interval=10, count=3):
    """
    Active TCP keep-alive

    idle : secondes avant premier keep-alive
    interval : secondes entre keep-alives
    count : nombre d'échecs avant considérer la connexion morte

    Total avant détection : idle + (interval × count)
    Exemple : 60 + (10 × 3) = 90 secondes
    """
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)

    # Linux
    if hasattr(socket, 'TCP_KEEPIDLE'):
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPIDLE, idle)
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPINTVL, interval)
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPCNT, count)

    # macOS
    elif hasattr(socket, 'TCP_KEEPALIVE'):
        sock.setsockopt(socket.IPPROTO_TCP, socket.TCP_KEEPALIVE, idle)

# Utilisation
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
enable_keepalive(sock)
sock.connect(('long-lived-server.com', 8080))
```

### Application-Level Heartbeat

```python
import threading
import time
import socket

class ConnectionWithHeartbeat:
    """Connexion avec heartbeat applicatif"""

    def __init__(self, sock, heartbeat_interval=30.0):
        self.sock = sock
        self.heartbeat_interval = heartbeat_interval
        self.last_activity = time.time()
        self.alive = True

        # Thread de heartbeat
        self.heartbeat_thread = threading.Thread(target=self._heartbeat_loop)
        self.heartbeat_thread.daemon = True
        self.heartbeat_thread.start()

    def send(self, data):
        """Envoie des données et met à jour last_activity"""
        self.sock.sendall(data)
        self.last_activity = time.time()

    def recv(self, size):
        """Reçoit des données et met à jour last_activity"""
        data = self.sock.recv(size)
        if data:
            self.last_activity = time.time()
        return data

    def _heartbeat_loop(self):
        """Envoie périodiquement des heartbeats"""
        while self.alive:
            time.sleep(self.heartbeat_interval)

            # Vérifier si on doit envoyer un heartbeat
            idle_time = time.time() - self.last_activity

            if idle_time >= self.heartbeat_interval:
                try:
                    # Envoyer un heartbeat (ping)
                    self.sock.sendall(b'PING\n')
                    self.last_activity = time.time()

                    # Attendre un pong
                    self.sock.settimeout(5.0)
                    response = self.sock.recv(1024)

                    if not response or b'PONG' not in response:
                        print("Heartbeat échoué : connexion morte")
                        self.alive = False
                        self.sock.close()

                except socket.timeout:
                    print("Heartbeat timeout : connexion morte")
                    self.alive = False
                    self.sock.close()

                except Exception as e:
                    print(f"Heartbeat error: {e}")
                    self.alive = False
                    self.sock.close()

    def close(self):
        """Fermeture propre"""
        self.alive = False
        self.sock.close()

# Utilisation
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('server.com', 8080))

conn = ConnectionWithHeartbeat(sock, heartbeat_interval=30.0)

# Utilisation normale
conn.send(b'Hello')
data = conn.recv(4096)

# Le heartbeat détectera automatiquement une déconnexion
```

### Détection par recv() retournant 0

```python
def read_loop_with_disconnection_detection(sock):
    """Boucle de lecture avec détection de déconnexion"""
    while True:
        try:
            data = sock.recv(4096)

            if not data:
                # recv() retourne b'' (vide) → connexion fermée proprement
                print("Connexion fermée par le pair (FIN reçu)")
                break

            # Traiter les données
            process_data(data)

        except ConnectionResetError:
            # RST reçu → connexion fermée brutalement
            print("Connexion reset par le pair")
            break

        except socket.timeout:
            # Timeout sans données → connexion peut-être morte
            print("Timeout : vérification de la connexion...")
            if not is_connection_alive(sock):
                print("Connexion morte détectée")
                break

        except Exception as e:
            print(f"Erreur de lecture: {e}")
            break

    sock.close()

def is_connection_alive(sock):
    """Vérifie si une connexion est encore vivante"""
    try:
        # Essayer de lire avec timeout court
        sock.settimeout(1.0)
        data = sock.recv(1, socket.MSG_PEEK)  # Peek sans consommer
        return True
    except:
        return False
```

## Logging et monitoring des erreurs

### Logging structuré

```python
import logging
import json
import time

# Configuration du logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class NetworkLogger:
    """Logger dédié aux opérations réseau"""

    @staticmethod
    def log_connection_attempt(host, port):
        logger.info(f"Tentative de connexion à {host}:{port}")

    @staticmethod
    def log_connection_success(host, port, duration):
        logger.info(f"Connexion établie à {host}:{port} en {duration:.3f}s")

    @staticmethod
    def log_connection_failure(host, port, error, attempt=1):
        logger.error(
            f"Échec connexion à {host}:{port} (tentative {attempt}): {error}",
            extra={
                'host': host,
                'port': port,
                'error_type': type(error).__name__,
                'attempt': attempt
            }
        )

    @staticmethod
    def log_timeout(operation, duration, host=None):
        logger.warning(
            f"Timeout lors de {operation} après {duration:.3f}s",
            extra={'operation': operation, 'timeout': duration, 'host': host}
        )

    @staticmethod
    def log_retry(attempt, max_attempts, delay, reason):
        logger.info(
            f"Retry {attempt}/{max_attempts} dans {delay:.1f}s - Raison: {reason}"
        )

# Utilisation
def connect_with_logging(host, port, timeout=5.0):
    NetworkLogger.log_connection_attempt(host, port)
    start = time.time()

    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        sock.connect((host, port))

        duration = time.time() - start
        NetworkLogger.log_connection_success(host, port, duration)

        return sock

    except socket.timeout:
        NetworkLogger.log_timeout('connect', timeout, host)
        raise

    except Exception as e:
        NetworkLogger.log_connection_failure(host, port, e)
        raise
```

### Métriques et monitoring

```python
from collections import defaultdict
import time

class NetworkMetrics:
    """Collecte de métriques réseau"""

    def __init__(self):
        self.request_count = defaultdict(int)
        self.error_count = defaultdict(int)
        self.latencies = defaultdict(list)
        self.timeouts = 0
        self.connection_failures = 0

    def record_request(self, endpoint, latency, success=True):
        """Enregistre une requête"""
        self.request_count[endpoint] += 1
        self.latencies[endpoint].append(latency)

        if not success:
            self.error_count[endpoint] += 1

    def record_timeout(self):
        self.timeouts += 1

    def record_connection_failure(self):
        self.connection_failures += 1

    def get_stats(self, endpoint=None):
        """Récupère les statistiques"""
        if endpoint:
            latencies = self.latencies[endpoint]
            return {
                'endpoint': endpoint,
                'requests': self.request_count[endpoint],
                'errors': self.error_count[endpoint],
                'error_rate': self.error_count[endpoint] / max(1, self.request_count[endpoint]),
                'avg_latency': sum(latencies) / len(latencies) if latencies else 0,
                'p95_latency': sorted(latencies)[int(len(latencies) * 0.95)] if latencies else 0
            }

        return {
            'total_requests': sum(self.request_count.values()),
            'total_errors': sum(self.error_count.values()),
            'timeouts': self.timeouts,
            'connection_failures': self.connection_failures
        }

    def print_report(self):
        """Affiche un rapport"""
        print("\n=== Network Metrics Report ===")
        print(f"Total requests: {sum(self.request_count.values())}")
        print(f"Total errors: {sum(self.error_count.values())}")
        print(f"Timeouts: {self.timeouts}")
        print(f"Connection failures: {self.connection_failures}")

        for endpoint in self.request_count:
            stats = self.get_stats(endpoint)
            print(f"\nEndpoint: {endpoint}")
            print(f"  Requests: {stats['requests']}")
            print(f"  Errors: {stats['errors']} ({stats['error_rate']*100:.1f}%)")
            print(f"  Avg latency: {stats['avg_latency']*1000:.2f}ms")
            print(f"  P95 latency: {stats['p95_latency']*1000:.2f}ms")

# Utilisation
metrics = NetworkMetrics()

def monitored_request(url):
    """Requête avec métriques"""
    start = time.time()
    success = False

    try:
        response = requests.get(url, timeout=5.0)
        response.raise_for_status()
        success = True
        return response

    except requests.Timeout:
        metrics.record_timeout()
        raise

    except requests.ConnectionError:
        metrics.record_connection_failure()
        raise

    finally:
        latency = time.time() - start
        metrics.record_request(url, latency, success)

# Après un certain temps
metrics.print_report()
```

## Exemple complet : Client HTTP robuste

```python
import socket
import time
import logging
from enum import Enum

logger = logging.getLogger(__name__)

class HTTPClient:
    """Client HTTP robuste avec gestion d'erreurs complète"""

    def __init__(
        self,
        connect_timeout=3.0,
        read_timeout=10.0,
        max_retries=3,
        circuit_breaker_threshold=5
    ):
        self.connect_timeout = connect_timeout
        self.read_timeout = read_timeout
        self.max_retries = max_retries

        # Circuit breaker pour chaque host
        self.circuit_breakers = {}
        self.cb_threshold = circuit_breaker_threshold

        # Métriques
        self.metrics = NetworkMetrics()

    def get(self, url, headers=None):
        """Requête GET avec gestion d'erreurs complète"""
        host, port, path = self._parse_url(url)

        # Obtenir le circuit breaker pour ce host
        breaker = self._get_circuit_breaker(host)

        # Retry avec exponential backoff
        for attempt in range(self.max_retries):
            try:
                # Vérifier le circuit breaker
                if breaker.state == CircuitState.OPEN:
                    if not breaker._should_attempt_reset():
                        raise RuntimeError(
                            f"Circuit breaker ouvert pour {host}. "
                            f"Retry dans {breaker._time_until_retry():.0f}s"
                        )
                    breaker.state = CircuitState.HALF_OPEN

                # Tenter la requête
                response = self._do_request(host, port, path, headers)

                # Succès
                breaker._on_success()
                return response

            except socket.timeout as e:
                logger.warning(f"Timeout lors de la requête à {url} (tentative {attempt+1})")
                self.metrics.record_timeout()
                breaker._on_failure()

                if attempt < self.max_retries - 1:
                    delay = self._backoff_delay(attempt)
                    logger.info(f"Retry dans {delay:.1f}s...")
                    time.sleep(delay)
                else:
                    raise

            except ConnectionError as e:
                logger.error(f"Erreur de connexion à {url}: {e}")
                self.metrics.record_connection_failure()
                breaker._on_failure()

                if attempt < self.max_retries - 1:
                    delay = self._backoff_delay(attempt)
                    time.sleep(delay)
                else:
                    raise

            except Exception as e:
                logger.error(f"Erreur inattendue: {e}")
                breaker._on_failure()
                raise

    def _do_request(self, host, port, path, headers):
        """Effectue la requête HTTP"""
        start_time = time.time()
        sock = None

        try:
            # 1. Connexion
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(self.connect_timeout)

            logger.debug(f"Connexion à {host}:{port}...")
            sock.connect((host, port))

            # 2. Envoi de la requête
            sock.settimeout(self.read_timeout)
            request = self._build_request(host, path, headers)
            sock.sendall(request.encode('utf-8'))

            # 3. Réception de la réponse
            response = self._read_response(sock)

            # Métriques
            latency = time.time() - start_time
            self.metrics.record_request(f"{host}{path}", latency, success=True)

            return response

        finally:
            if sock:
                sock.close()

    def _build_request(self, host, path, headers):
        """Construit la requête HTTP"""
        lines = [f"GET {path} HTTP/1.1", f"Host: {host}", "Connection: close"]

        if headers:
            for key, value in headers.items():
                lines.append(f"{key}: {value}")

        lines.append("")  # Ligne vide
        lines.append("")  # Fin des headers

        return "\r\n".join(lines)

    def _read_response(self, sock):
        """Lit la réponse HTTP"""
        chunks = []
        while True:
            try:
                chunk = sock.recv(4096)
                if not chunk:
                    break
                chunks.append(chunk)
            except socket.timeout:
                logger.warning("Timeout lors de la lecture de la réponse")
                break

        return b''.join(chunks).decode('utf-8')

    def _parse_url(self, url):
        """Parse une URL simple"""
        # Simplification : http://host:port/path
        if url.startswith('http://'):
            url = url[7:]
        elif url.startswith('https://'):
            url = url[8:]

        parts = url.split('/', 1)
        host_port = parts[0]
        path = '/' + parts[1] if len(parts) > 1 else '/'

        if ':' in host_port:
            host, port = host_port.split(':', 1)
            port = int(port)
        else:
            host = host_port
            port = 80

        return host, port, path

    def _backoff_delay(self, attempt):
        """Calcule le délai de backoff"""
        base_delay = 1.0
        max_delay = 30.0
        delay = min(base_delay * (2 ** attempt), max_delay)
        # Jitter
        import random
        return delay * (0.5 + random.random())

    def _get_circuit_breaker(self, host):
        """Obtient ou crée un circuit breaker pour un host"""
        if host not in self.circuit_breakers:
            self.circuit_breakers[host] = CircuitBreaker(
                failure_threshold=self.cb_threshold
            )
        return self.circuit_breakers[host]

    def get_metrics(self):
        """Retourne les métriques"""
        return self.metrics.get_stats()

# Utilisation
client = HTTPClient(
    connect_timeout=3.0,
    read_timeout=10.0,
    max_retries=3
)

try:
    response = client.get('http://example.com/')
    print(f"Réponse reçue: {len(response)} caractères")
except Exception as e:
    print(f"Échec: {e}")

# Afficher les métriques
print(client.get_metrics())
```

## Récapitulatif des bonnes pratiques

### ✅ DO (À faire)

1. **Toujours définir des timeouts**
   ```python
   sock.settimeout(5.0)  # Jamais de timeout infini
   ```

2. **Gérer toutes les exceptions réseau**
   ```python
   try:
       sock.connect((host, port))
   except (socket.timeout, ConnectionError, OSError) as e:
       handle_error(e)
   ```

3. **Retry avec exponential backoff**
   ```python
   for attempt in range(max_retries):
       delay = base_delay * (2 ** attempt)
   ```

4. **Logger les erreurs avec contexte**
   ```python
   logger.error(f"Connexion échouée à {host}:{port}", extra={'attempt': attempt})
   ```

5. **Utiliser des circuit breakers**
   ```python
   if too_many_failures:
       fail_fast()
   ```

6. **Avoir des fallbacks**
   ```python
   try:
       data = fetch_from_api()
   except:
       data = get_from_cache()
   ```

7. **Monitorer les erreurs**
   ```python
   metrics.record_error(error_type, endpoint)
   ```

### ❌ DON'T (À éviter)

1. **Pas de timeout**
   ```python
   sock.recv(4096)  # Peut bloquer indéfiniment
   ```

2. **Ignorer les erreurs**
   ```python
   try:
       sock.connect()
   except:
       pass  # Mauvais !
   ```

3. **Retry infini**
   ```python
   while True:  # Mauvais !
       try:
           connect()
           break
       except:
           continue
   ```

4. **Retry sans délai**
   ```python
   for i in range(10):
       try_connect()  # Spam le serveur
   ```

5. **Un seul timeout pour tout**
   ```python
   timeout = 30  # Trop pour connexion, trop peu pour upload
   ```

## Conclusion

La gestion d'erreurs réseau n'est pas optionnelle. C'est ce qui différencie une application de production d'un prototype. Les principes clés :

- 🎯 **Anticipez les défaillances** : Le réseau va échouer
- 🎯 **Timeout partout** : Jamais de blocage infini
- 🎯 **Retry intelligemment** : Backoff + jitter
- 🎯 **Fail fast ou fail safe** : Selon le contexte
- 🎯 **Circuit breakers** : Protégez votre système
- 🎯 **Loggez et monitorez** : Visibilité sur les problèmes
- 🎯 **Testez les erreurs** : Chaos engineering

## Prochaines étapes

Maintenant que vous savez gérer les erreurs réseau, nous allons explorer :

- **Section 8.5** : I/O bloquant vs non-bloquant
- **Section 8.6** : Multiplexage (select, poll, epoll) pour gérer des milliers de connexions
- **Section 8.7** : Programmation asynchrone et event-driven

---


⏭️ [I/O bloquant vs non-bloquant](/08-programmation-reseau/05-io-bloquant-non-bloquant.md)
