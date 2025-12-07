🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.4 Proxys et Reverse Proxys

## Introduction

Un **proxy** est un serveur intermédiaire qui se place entre un client et un serveur de destination, agissant comme un **relais intelligent** pour les requêtes réseau. Selon sa position et son rôle, on distingue les **forward proxys** (côté client) et les **reverse proxys** (côté serveur).

### L'analogie du concierge et de l'assistant

**Forward Proxy (concierge d'immeuble) :**
```
Vous (dans votre appartement) → Concierge → Extérieur
         ↑
   Vous ne sortez jamais directement,
   le concierge fait vos courses pour vous

Avantages :
- Le concierge peut filtrer (sécurité)
- Il peut mettre en cache (efficacité)
- Personne ne sait qui vous êtes vraiment (anonymat)
```

**Reverse Proxy (assistant de PDG) :**
```
Client externe → Assistant → PDG (ou un de ses adjoints)
                    ↑
          L'assistant décide qui traite la demande,
          le client ne sait pas qui a vraiment répondu

Avantages :
- L'assistant filtre les demandes (sécurité)
- Il peut répondre directement pour certaines choses (cache)
- Le PDG peut être remplacé sans que personne le sache (flexibilité)
```

### Visualisation de la différence

```
FORWARD PROXY (client-side)
═══════════════════════════════════════════════════════

[Réseau d'entreprise]           │    [Internet]
                                │
Client 1 ───┐                   │
Client 2 ───┼──→ Forward Proxy ─┼──→ google.com
Client 3 ───┘         ↑         │
                      │         │
                 Filtre,        │
                 cache,         │
                 anonymise      │

Usage : Contrôle d'accès internet, cache entreprise, contournement censure


REVERSE PROXY (server-side)
═══════════════════════════════════════════════════════

[Internet]              │    [Réseau privé]
                        │
Client Internet ────────┼──→ Reverse Proxy ──┬──→ Server 1
                        │         ↑          ├──→ Server 2
                        │         │          └──→ Server 3
                        │    SSL termination
                        │    Load balancing
                        │    Caching

Usage : Protection serveurs, load balancing, SSL offloading, caching
```

## Forward Proxy : Le proxy traditionnel

### Principe de fonctionnement

```
Sans proxy :
Client ────[requête directe]───→ www.example.com

Avec forward proxy :
Client ──→ Proxy ──→ www.example.com
       ↑         ↑
   Configure  Modifie la
   son proxy   requête
```

### Cas d'usage réels

#### 1. Contrôle d'accès internet en entreprise

**Contexte :** Une entreprise veut contrôler et auditer l'accès internet de ses employés.

**Configuration Squid (proxy d'entreprise) :**

```bash
# /etc/squid/squid.conf

# Port d'écoute
http_port 3128

# ACLs (Access Control Lists)
acl localnet src 192.168.0.0/16        # Réseau local
acl SSL_ports port 443                  # HTTPS
acl Safe_ports port 80                  # HTTP
acl Safe_ports port 443                 # HTTPS
acl Safe_ports port 21                  # FTP
acl CONNECT method CONNECT

# Blacklist de sites
acl blocked_sites dstdomain .facebook.com .twitter.com .youtube.com
acl blocked_keywords url_regex -i gambling porn adult

# Whitelist pour certains utilisateurs
acl managers src 192.168.1.10 192.168.1.11  # IPs des managers

# Règles de contrôle
http_access deny blocked_sites !managers
http_access deny blocked_keywords
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localnet
http_access deny all

# Logging
access_log /var/log/squid/access.log squid
cache_log /var/log/squid/cache.log

# Cache
cache_dir ufs /var/spool/squid 10000 16 256
maximum_object_size 100 MB
cache_mem 256 MB

# Anonymisation (headers)
request_header_access Referer deny all
request_header_access X-Forwarded-For deny all
request_header_access Via deny all
request_header_access Cache-Control deny all

# Authentification (optionnelle)
# auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
# acl authenticated proxy_auth REQUIRED
# http_access allow authenticated
```

**Configuration client (automatique via PAC) :**

```javascript
// proxy.pac - Proxy Auto-Configuration
function FindProxyForURL(url, host) {
    // Pas de proxy pour le réseau local
    if (isInNet(host, "192.168.0.0", "255.255.0.0") ||
        isInNet(host, "10.0.0.0", "255.0.0.0")) {
        return "DIRECT";
    }

    // Sites spécifiques en direct
    if (dnsDomainIs(host, ".internal.company.com")) {
        return "DIRECT";
    }

    // Tout le reste via proxy
    return "PROXY proxy.company.com:3128; DIRECT";
}
```

**Utilisation en Python :**

```python
import requests

# Configurer le proxy
proxies = {
    'http': 'http://proxy.company.com:3128',
    'https': 'http://proxy.company.com:3128',
}

# Requête via proxy
response = requests.get('https://www.example.com', proxies=proxies)

# Avec authentification
proxies_auth = {
    'http': 'http://user:password@proxy.company.com:3128',
    'https': 'http://user:password@proxy.company.com:3128',
}

response = requests.get('https://api.example.com', proxies=proxies_auth)
```

#### 2. Cache partagé pour économiser la bande passante

**Contexte :** Un ISP veut cacher le contenu populaire pour réduire la bande passante sortante.

```python
# Configuration Nginx en tant que caching proxy

# /etc/nginx/nginx.conf
http {
    # Définir le cache
    proxy_cache_path /var/cache/nginx levels=1:2
                     keys_zone=my_cache:10m
                     max_size=10g
                     inactive=60m
                     use_temp_path=off;

    server {
        listen 8080;

        location / {
            # Activer le cache
            proxy_cache my_cache;

            # Clé de cache
            proxy_cache_key "$scheme$request_method$host$request_uri";

            # Durée de cache par code HTTP
            proxy_cache_valid 200 60m;
            proxy_cache_valid 404 1m;

            # Headers de cache
            add_header X-Cache-Status $upstream_cache_status;

            # Proxy vers l'internet
            resolver 8.8.8.8;
            proxy_pass http://$http_host$request_uri;

            # Headers
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

#### 3. Contournement de restrictions géographiques (VPN/Proxy)

**Contexte :** Accéder à du contenu géo-restreint.

```python
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

class ProxyRotator:
    """
    Rotation de proxys pour éviter les blocages
    """
    def __init__(self, proxy_list):
        self.proxies = proxy_list
        self.current_index = 0

    def get_next_proxy(self):
        proxy = self.proxies[self.current_index]
        self.current_index = (self.current_index + 1) % len(self.proxies)
        return {
            'http': f'http://{proxy}',
            'https': f'http://{proxy}'
        }

    def request_with_rotation(self, url, max_retries=3):
        """
        Tente la requête avec rotation de proxys
        """
        for attempt in range(max_retries):
            proxy = self.get_next_proxy()

            try:
                response = requests.get(
                    url,
                    proxies=proxy,
                    timeout=10,
                    headers={'User-Agent': 'Mozilla/5.0'}
                )

                if response.status_code == 200:
                    return response

            except Exception as e:
                print(f"Proxy {proxy} failed: {e}")
                continue

        raise Exception("All proxies failed")

# Usage
rotator = ProxyRotator([
    'proxy1.example.com:8080',
    'proxy2.example.com:8080',
    'proxy3.example.com:8080'
])

response = rotator.request_with_rotation('https://geo-restricted-content.com')
```

#### 4. Scraping web avec proxy rotatif

```python
import requests
from itertools import cycle
import time

class SmartProxyScraper:
    """
    Scraper avec gestion intelligente de proxys
    """
    def __init__(self, proxies, delay=1):
        self.proxy_pool = cycle(proxies)
        self.delay = delay
        self.failed_proxies = set()

    def get_working_proxy(self):
        """Obtenir un proxy fonctionnel"""
        for _ in range(len(self.proxies)):
            proxy = next(self.proxy_pool)
            if proxy not in self.failed_proxies:
                return proxy
        return None

    def scrape(self, url):
        """Scrape avec gestion automatique des échecs"""
        max_attempts = 5

        for attempt in range(max_attempts):
            proxy = self.get_working_proxy()

            if not proxy:
                print("Tous les proxys ont échoué")
                return None

            try:
                response = requests.get(
                    url,
                    proxies={'http': proxy, 'https': proxy},
                    timeout=10,
                    headers={
                        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
                    }
                )

                if response.status_code == 200:
                    time.sleep(self.delay)
                    return response.text

                elif response.status_code == 429:  # Rate limited
                    print(f"Rate limited on {proxy}, trying next...")
                    time.sleep(5)

            except requests.exceptions.RequestException as e:
                print(f"Proxy {proxy} failed: {e}")
                self.failed_proxies.add(proxy)

        return None

# Usage
scraper = SmartProxyScraper([
    'http://proxy1:8080',
    'http://proxy2:8080',
    'http://proxy3:8080'
])

content = scraper.scrape('https://target-site.com/data')
```

## Reverse Proxy : Le gardien des serveurs

### Principe de fonctionnement

Le reverse proxy se positionne **devant les serveurs backend** et agit comme point d'entrée unique.

```
Client → [Internet] → Reverse Proxy → [Backend 1]
                           ↓          → [Backend 2]
                           ↓          → [Backend 3]
                      Décide du
                      routage
```

### Fonctionnalités d'un reverse proxy

1. **Load Balancing** (déjà vu dans 9.3)
2. **SSL Termination** (déchiffrement HTTPS centralisé)
3. **Caching** (réponses mises en cache)
4. **Compression** (gzip, brotli)
5. **Sécurité** (WAF, rate limiting, IP filtering)
6. **Transformation de requêtes** (headers, URL rewriting)
7. **Observabilité** (logs, métriques centralisées)

### Cas d'usage réels

#### 1. SSL Termination (TLS Offloading)

**Contexte :** Décharger le chiffrement SSL des serveurs backend pour améliorer les performances.

**Architecture :**

```
Client HTTPS ────────→ Reverse Proxy ────────→ Backend (HTTP)
                       (déchiffre SSL)

Avantages :
✅ Un seul point pour gérer les certificats
✅ Backends n'ont pas besoin de SSL
✅ Performance améliorée (pas de crypto sur backends)
✅ Inspection du trafic possible (pour WAF)
```

**Configuration Nginx :**

```nginx
# /etc/nginx/sites-available/ssl-termination

server {
    listen 80;
    server_name example.com www.example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL Protocols et Ciphers (sécurisés)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;

    # SSL Session Cache
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Proxy vers backend HTTP (non chiffré)
    location / {
        proxy_pass http://backend_servers;  # HTTP, pas HTTPS !

        # Headers importants pour backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;  # Indique que la requête initiale était HTTPS

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}

# Upstream backend (HTTP)
upstream backend_servers {
    least_conn;
    server 10.0.1.10:8080;  # Notez : port 8080 HTTP, pas 8443
    server 10.0.1.11:8080;
    server 10.0.1.12:8080;
}
```

**Vérification côté backend (Flask) :**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    # Vérifier que la requête originale était HTTPS
    protocol = request.headers.get('X-Forwarded-Proto', 'http')

    if protocol == 'https':
        return "✅ Connexion sécurisée (via reverse proxy)"
    else:
        return "⚠️ Connexion non sécurisée"

@app.route('/debug')
def debug():
    """Afficher tous les headers pour debugging"""
    headers = dict(request.headers)
    return {
        'client_ip': request.headers.get('X-Real-IP'),
        'forwarded_for': request.headers.get('X-Forwarded-For'),
        'protocol': request.headers.get('X-Forwarded-Proto'),
        'all_headers': headers
    }

if __name__ == '__main__':
    # Backend écoute en HTTP seulement
    app.run(host='0.0.0.0', port=8080, ssl_context=None)
```

#### 2. Caching intelligent

**Contexte :** Cacher les réponses pour réduire la charge backend et améliorer la latence.

```nginx
# /etc/nginx/nginx.conf

http {
    # Zone de cache
    proxy_cache_path /var/cache/nginx/content
                     levels=1:2
                     keys_zone=content_cache:100m
                     max_size=10g
                     inactive=7d
                     use_temp_path=off;

    # Zone pour API (cache plus court)
    proxy_cache_path /var/cache/nginx/api
                     levels=1:2
                     keys_zone=api_cache:50m
                     max_size=1g
                     inactive=1h
                     use_temp_path=off;

    # Définir quand bypasser le cache
    map $request_method $bypass_cache {
        default 0;
        POST 1;
        PUT 1;
        DELETE 1;
    }

    map $http_cookie $bypass_cache_cookie {
        default 0;
        "~*session" 1;  # Ne pas cacher si cookie de session
    }

    server {
        listen 80;
        server_name example.com;

        # Pages statiques (cache long)
        location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2)$ {
            proxy_pass http://backend;

            proxy_cache content_cache;
            proxy_cache_key "$scheme$request_method$host$request_uri";
            proxy_cache_valid 200 7d;
            proxy_cache_valid 404 1h;

            # Headers de cache
            add_header X-Cache-Status $upstream_cache_status;
            add_header Cache-Control "public, max-age=604800, immutable";

            expires 7d;
        }

        # API (cache court, conditionnel)
        location /api/ {
            proxy_pass http://backend;

            proxy_cache api_cache;
            proxy_cache_key "$scheme$request_method$host$request_uri$args";
            proxy_cache_valid 200 5m;
            proxy_cache_valid 404 1m;

            # Bypass cache si nécessaire
            proxy_cache_bypass $bypass_cache $bypass_cache_cookie;
            proxy_no_cache $bypass_cache $bypass_cache_cookie;

            # Headers
            add_header X-Cache-Status $upstream_cache_status;

            # Permettre revalidation
            proxy_cache_revalidate on;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        }

        # Pages HTML (cache micro)
        location / {
            proxy_pass http://backend;

            proxy_cache content_cache;
            proxy_cache_key "$scheme$request_method$host$request_uri";
            proxy_cache_valid 200 1m;

            add_header X-Cache-Status $upstream_cache_status;
        }
    }
}
```

**Purge de cache programmatique :**

```python
import requests

class NginxCachePurger:
    """
    Purger le cache Nginx via module ngx_cache_purge
    """
    def __init__(self, nginx_host):
        self.nginx_host = nginx_host

    def purge_url(self, url):
        """
        Purger une URL spécifique du cache
        """
        # Construire l'URL de purge
        purge_url = f"http://{self.nginx_host}/purge{url}"

        try:
            response = requests.request('PURGE', purge_url)

            if response.status_code == 200:
                return True, "Cache purged successfully"
            else:
                return False, f"Purge failed: {response.status_code}"

        except Exception as e:
            return False, str(e)

    def purge_pattern(self, pattern):
        """
        Purger toutes les URLs matchant un pattern
        Nécessite nginx module ngx_cache_purge avec wildcard support
        """
        purge_url = f"http://{self.nginx_host}/purge/{pattern}*"

        try:
            response = requests.request('PURGE', purge_url)
            return response.status_code == 200
        except:
            return False

# Usage après déploiement
purger = NginxCachePurger('localhost')

# Purger une URL spécifique
purger.purge_url('/api/users/123')

# Purger tout le cache API
purger.purge_pattern('/api/')
```

**Cache avec validation conditionnelle :**

```python
from flask import Flask, request, make_response
from datetime import datetime, timedelta
import hashlib

app = Flask(__name__)

@app.route('/api/data')
def get_data():
    """
    Endpoint avec support ETag et Last-Modified
    Pour permettre la revalidation de cache
    """
    # Données (en pratique, depuis DB)
    data = {'users': 1000, 'active': 850}

    # Calculer ETag (hash du contenu)
    content = str(data).encode()
    etag = hashlib.md5(content).hexdigest()

    # Last-Modified (en pratique, timestamp DB)
    last_modified = datetime.utcnow().replace(microsecond=0)

    # Vérifier If-None-Match (ETag)
    if_none_match = request.headers.get('If-None-Match')
    if if_none_match == etag:
        # Contenu inchangé
        return '', 304

    # Vérifier If-Modified-Since
    if_modified_since = request.headers.get('If-Modified-Since')
    if if_modified_since:
        try:
            ims_date = datetime.strptime(if_modified_since, '%a, %d %b %Y %H:%M:%S GMT')
            if last_modified <= ims_date:
                return '', 304
        except:
            pass

    # Créer la réponse
    response = make_response(data)
    response.headers['ETag'] = etag
    response.headers['Last-Modified'] = last_modified.strftime('%a, %d %b %Y %H:%M:%S GMT')
    response.headers['Cache-Control'] = 'public, max-age=60, must-revalidate'

    return response
```

#### 3. Rate Limiting et protection DDoS

**Contexte :** Protéger les backends contre les abus et les attaques DDoS.

```nginx
# /etc/nginx/nginx.conf

http {
    # Définir des zones de rate limiting

    # Par IP : 10 req/s, burst de 20
    limit_req_zone $binary_remote_addr zone=per_ip:10m rate=10r/s;

    # Par API key : 100 req/s
    limit_req_zone $http_x_api_key zone=per_api_key:10m rate=100r/s;

    # Connexions simultanées par IP
    limit_conn_zone $binary_remote_addr zone=conn_per_ip:10m;

    # Limiter la bande passante
    limit_rate_after 10m;   # Après 10MB
    limit_rate 1m;          # Limiter à 1MB/s

    server {
        listen 80;
        server_name api.example.com;

        # Logs pour analyse
        access_log /var/log/nginx/api_access.log combined;
        error_log /var/log/nginx/api_error.log;

        # Rate limiting global
        location / {
            limit_req zone=per_ip burst=20 nodelay;
            limit_conn conn_per_ip 10;  # Max 10 connexions simultanées

            proxy_pass http://backend;
        }

        # API publique (plus stricte)
        location /api/public/ {
            limit_req zone=per_ip burst=5 nodelay;

            # Status code custom pour rate limit
            limit_req_status 429;

            proxy_pass http://backend;
        }

        # API authentifiée (rate limit par API key)
        location /api/v1/ {
            # Vérifier présence API key
            if ($http_x_api_key = "") {
                return 401 '{"error": "API key required"}';
            }

            limit_req zone=per_api_key burst=50 nodelay;

            proxy_pass http://backend;
            proxy_set_header X-API-Key $http_x_api_key;
        }

        # Endpoints sans limite (health checks)
        location = /health {
            access_log off;
            proxy_pass http://backend;
        }
    }

    # Bloquer automatiquement les IPs abusives
    geo $blocked_ip {
        default 0;
        192.168.1.100 1;  # IP blacklistée
        10.0.0.50 1;
    }

    server {
        listen 80;

        if ($blocked_ip) {
            return 403 '{"error": "Access denied"}';
        }

        location / {
            proxy_pass http://backend;
        }
    }
}
```

**Rate limiting applicatif (pour compléter) :**

```python
from flask import Flask, request, jsonify
from functools import wraps
import redis
import time

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def rate_limit(max_requests=10, window=60):
    """
    Décorateur de rate limiting avec Redis

    Args:
        max_requests: Nombre max de requêtes
        window: Fenêtre de temps en secondes
    """
    def decorator(f):
        @wraps(f)
        def wrapped(*args, **kwargs):
            # Identifier le client (IP ou API key)
            client_id = request.headers.get('X-API-Key') or request.remote_addr

            # Clé Redis
            key = f"rate_limit:{f.__name__}:{client_id}"

            # Obtenir le compte actuel
            current = redis_client.get(key)

            if current is None:
                # Première requête dans cette fenêtre
                redis_client.setex(key, window, 1)
                return f(*args, **kwargs)

            current = int(current)

            if current >= max_requests:
                # Rate limit dépassé
                ttl = redis_client.ttl(key)

                return jsonify({
                    'error': 'Rate limit exceeded',
                    'retry_after': ttl
                }), 429

            # Incrémenter le compteur
            redis_client.incr(key)

            return f(*args, **kwargs)

        return wrapped
    return decorator

@app.route('/api/data')
@rate_limit(max_requests=10, window=60)  # 10 req/min
def get_data():
    return {'data': 'some data'}

@app.route('/api/expensive')
@rate_limit(max_requests=2, window=60)  # 2 req/min
def expensive_operation():
    # Opération coûteuse
    time.sleep(2)
    return {'result': 'computed'}

# Rate limiting avec sliding window (plus précis)
class SlidingWindowRateLimiter:
    def __init__(self, redis_client, max_requests, window):
        self.redis = redis_client
        self.max_requests = max_requests
        self.window = window

    def is_allowed(self, client_id):
        """
        Vérifie si la requête est autorisée
        Utilise un sorted set avec timestamps
        """
        now = time.time()
        key = f"rate_limit:sliding:{client_id}"

        # Supprimer les requêtes hors de la fenêtre
        self.redis.zremrangebyscore(key, 0, now - self.window)

        # Compter les requêtes dans la fenêtre
        current_count = self.redis.zcard(key)

        if current_count >= self.max_requests:
            return False

        # Ajouter cette requête
        self.redis.zadd(key, {str(now): now})

        # Expirer la clé après la fenêtre
        self.redis.expire(key, self.window)

        return True

# Usage
limiter = SlidingWindowRateLimiter(redis_client, max_requests=10, window=60)

@app.route('/api/precise')
def precise_endpoint():
    client_id = request.remote_addr

    if not limiter.is_allowed(client_id):
        return jsonify({'error': 'Rate limit exceeded'}), 429

    return {'data': 'response'}
```

#### 4. URL Rewriting et redirection

**Contexte :** Migrer d'une ancienne structure d'URLs vers une nouvelle sans casser les liens.

```nginx
server {
    listen 80;
    server_name example.com;

    # Redirection permanente (301) - SEO friendly
    location /old-blog/ {
        rewrite ^/old-blog/(.*)$ /blog/$1 permanent;
    }

    # Redirection temporaire (302)
    location /maintenance {
        rewrite ^ /maintenance.html redirect;
    }

    # Réécriture d'URL complexe
    location ~ ^/products/([0-9]+)/(.+)$ {
        # /products/123/my-product → /api/products?id=123&slug=my-product
        rewrite ^/products/([0-9]+)/(.+)$ /api/products?id=$1&slug=$2 break;
        proxy_pass http://backend;
    }

    # Normalisation d'URL (enlever .html)
    location ~ (.+)\.html$ {
        rewrite ^(.+)\.html$ $1 permanent;
    }

    # Forcer HTTPS
    if ($scheme = http) {
        return 301 https://$server_name$request_uri;
    }

    # Redirection avec préservation des query params
    location /legacy {
        if ($args) {
            rewrite ^ /new-endpoint?$args permanent;
        }
        rewrite ^ /new-endpoint permanent;
    }

    # Servir différents backends selon le path
    location /api/v1/ {
        rewrite ^/api/v1/(.*)$ /$1 break;
        proxy_pass http://backend_v1;
    }

    location /api/v2/ {
        rewrite ^/api/v2/(.*)$ /$1 break;
        proxy_pass http://backend_v2;
    }
}
```

#### 5. Compression automatique

```nginx
http {
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;  # 1-9, 6 est un bon compromis
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/rss+xml
        font/truetype
        font/opentype
        application/vnd.ms-fontobject
        image/svg+xml;
    gzip_disable "msie6";
    gzip_min_length 1000;  # Ne pas compresser < 1KB

    # Brotli (si module installé, meilleure compression que gzip)
    brotli on;
    brotli_comp_level 6;
    brotli_types
        text/plain
        text/css
        application/json
        application/javascript
        text/xml
        application/xml
        image/svg+xml;

    server {
        listen 80;

        location / {
            proxy_pass http://backend;

            # Désactiver compression si déjà compressé
            proxy_set_header Accept-Encoding "";
        }

        # Assets statiques (compression forte)
        location /static/ {
            gzip_comp_level 9;  # Max compression pour static
            expires 1y;
            add_header Cache-Control "public, immutable";

            root /var/www;
        }
    }
}
```

## API Gateway : Reverse Proxy moderne

Un **API Gateway** est un reverse proxy spécialisé pour les architectures microservices.

### Fonctionnalités d'un API Gateway

```
┌─────────────────────────────────────────────────┐
│            API Gateway (Kong, Traefik)          │
├─────────────────────────────────────────────────┤
│ ✓ Authentification & Autorisation               │
│ ✓ Rate Limiting                                 │
│ ✓ Request/Response Transformation               │
│ ✓ Service Discovery                             │
│ ✓ Load Balancing                                │
│ ✓ Circuit Breaking                              │
│ ✓ Observabilité (metrics, logs, traces)         │
│ ✓ API Versioning                                │
│ ✓ CORS Handling                                 │
└─────────────────────────────────────────────────┘
         ↓           ↓           ↓
    [Service 1] [Service 2] [Service 3]
```

### Kong API Gateway

**Installation et configuration :**

```yaml
# docker-compose.yml
version: '3.7'

services:
  kong-database:
    image: postgres:13
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kong
    volumes:
      - kong-data:/var/lib/postgresql/data

  kong-migrations:
    image: kong:latest
    command: kong migrations bootstrap
    depends_on:
      - kong-database
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong

  kong:
    image: kong:latest
    depends_on:
      - kong-database
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    ports:
      - "8000:8000"  # Proxy HTTP
      - "8443:8443"  # Proxy HTTPS
      - "8001:8001"  # Admin API
    volumes:
      - kong-data:/usr/local/kong

volumes:
  kong-data:
```

**Configuration via API :**

```python
import requests
import json

class KongAdmin:
    """
    Gestion de Kong API Gateway via son API Admin
    """
    def __init__(self, admin_url='http://localhost:8001'):
        self.admin_url = admin_url

    def create_service(self, name, url):
        """Créer un service backend"""
        response = requests.post(
            f"{self.admin_url}/services",
            json={
                'name': name,
                'url': url
            }
        )
        return response.json()

    def create_route(self, service_name, paths, methods=None):
        """Créer une route vers un service"""
        data = {
            'paths': paths if isinstance(paths, list) else [paths],
            'strip_path': True
        }

        if methods:
            data['methods'] = methods if isinstance(methods, list) else [methods]

        response = requests.post(
            f"{self.admin_url}/services/{service_name}/routes",
            json=data
        )
        return response.json()

    def add_plugin(self, service_name, plugin_name, config=None):
        """Ajouter un plugin à un service"""
        data = {'name': plugin_name}

        if config:
            data['config'] = config

        response = requests.post(
            f"{self.admin_url}/services/{service_name}/plugins",
            json=data
        )
        return response.json()

# Usage : Configurer Kong
kong = KongAdmin()

# 1. Créer un service pour l'API utilisateurs
kong.create_service('users-api', 'http://users-service:8080')

# 2. Créer une route
kong.create_route('users-api', ['/api/users'], methods=['GET', 'POST'])

# 3. Ajouter rate limiting
kong.add_plugin('users-api', 'rate-limiting', {
    'minute': 100,
    'policy': 'local'
})

# 4. Ajouter authentication (JWT)
kong.add_plugin('users-api', 'jwt', {
    'key_claim_name': 'kid',
    'secret_is_base64': False
})

# 5. Ajouter CORS
kong.add_plugin('users-api', 'cors', {
    'origins': ['https://myapp.com'],
    'methods': ['GET', 'POST', 'PUT', 'DELETE'],
    'headers': ['Authorization', 'Content-Type'],
    'credentials': True,
    'max_age': 3600
})

# 6. Ajouter request transformation
kong.add_plugin('users-api', 'request-transformer', {
    'add': {
        'headers': ['X-Service:users-api']
    },
    'remove': {
        'headers': ['X-Internal-Token']
    }
})

# 7. Ajouter circuit breaker (avec custom plugin)
kong.add_plugin('users-api', 'circuit-breaker', {
    'failure_threshold': 5,
    'success_threshold': 2,
    'timeout': 30
})
```

### Traefik (moderne, natif Docker/Kubernetes)

```yaml
# docker-compose.yml avec Traefik
version: '3.7'

services:
  traefik:
    image: traefik:v2.10
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=admin@example.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"  # Dashboard
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./letsencrypt:/letsencrypt"

  # Service API
  api:
    image: myapi:latest
    labels:
      - "traefik.enable=true"

      # Routage
      - "traefik.http.routers.api.rule=Host(`api.example.com`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=myresolver"

      # Load balancing
      - "traefik.http.services.api.loadbalancer.server.port=8080"
      - "traefik.http.services.api.loadbalancer.healthcheck.path=/health"
      - "traefik.http.services.api.loadbalancer.healthcheck.interval=10s"

      # Middleware : rate limiting
      - "traefik.http.middlewares.api-ratelimit.ratelimit.average=100"
      - "traefik.http.middlewares.api-ratelimit.ratelimit.burst=50"

      # Middleware : compression
      - "traefik.http.middlewares.api-compress.compress=true"

      # Middleware : headers
      - "traefik.http.middlewares.api-headers.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.middlewares.api-headers.headers.customresponseheaders.X-Custom-Header=MyValue"

      # Appliquer les middlewares
      - "traefik.http.routers.api.middlewares=api-ratelimit,api-compress,api-headers"
    deploy:
      replicas: 3

  # Service frontend
  frontend:
    image: myfrontend:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`www.example.com`)"
      - "traefik.http.routers.frontend.entrypoints=websecure"
      - "traefik.http.routers.frontend.tls.certresolver=myresolver"

      # Redirect HTTP → HTTPS
      - "traefik.http.routers.frontend-http.rule=Host(`www.example.com`)"
      - "traefik.http.routers.frontend-http.entrypoints=web"
      - "traefik.http.routers.frontend-http.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
```

## Service Mesh : Le proxy distribué

Un **service mesh** est un réseau de proxys (sidecars) déployés à côté de chaque microservice.

### Architecture Istio

```
┌───────────────────────────────────────────────────┐
│                 Control Plane                     │
│  (Istiod : configuration, certificates, telemetry)│
└───────────────────┬───────────────────────────────┘
                    │ Configure
                    ↓
┌─────────────────────────────────────────────────┐
│              Data Plane (Envoy Proxies)         │
├─────────────────────────────────────────────────┤
│  ┌──────────┐   ┌──────────┐  ┌──────────┐      │
│  │ Service A│   │ Service B│  │ Service C│      │
│  │  + Envoy │   │  + Envoy │  │  + Envoy │      │
│  └─────┬────┘   └─────┬────┘  └─────┬────┘      │
│        └──────────────┼─────────────┘           │
│         Trafic chiffré (mTLS automatique)       │
└─────────────────────────────────────────────────┘
```

**Configuration Istio (Kubernetes) :**

```yaml
# VirtualService : Routing intelligent
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: users-api
spec:
  hosts:
  - users-api.default.svc.cluster.local
  http:
  # Canary deployment : 90% v1, 10% v2
  - match:
    - headers:
        x-user-type:
          exact: beta
    route:
    - destination:
        host: users-api
        subset: v2
      weight: 100

  - route:
    - destination:
        host: users-api
        subset: v1
      weight: 90
    - destination:
        host: users-api
        subset: v2
      weight: 10

    # Retry policy
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure

    # Timeout
    timeout: 10s
---
# DestinationRule : Load balancing et circuit breaking
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: users-api
spec:
  host: users-api.default.svc.cluster.local
  trafficPolicy:
    # Load balancing
    loadBalancer:
      consistentHash:
        httpHeaderName: "x-user-id"  # Sticky session par user

    # Connection pool
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
        maxRequestsPerConnection: 2

    # Circuit breaker
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 40

  # Subsets pour versioning
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
---
# Gateway : Entrée du traffic externe
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: api-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: api-cert
    hosts:
    - "api.example.com"
```

## Transparence et X-Forwarded headers

Quand on utilise des proxys, il est crucial de **préserver les informations du client original**.

### Headers standards

```
GET /api/users HTTP/1.1
Host: api.example.com
X-Real-IP: 203.0.113.42                    # IP réelle du client
X-Forwarded-For: 203.0.113.42, 10.0.1.5   # Chaîne de proxys
X-Forwarded-Proto: https                    # Protocole original
X-Forwarded-Host: api.example.com          # Host original
X-Forwarded-Port: 443                       # Port original
```

### Configuration Nginx

```nginx
location / {
    proxy_pass http://backend;

    # Préserver l'IP réelle
    proxy_set_header X-Real-IP $remote_addr;

    # Ajouter à la chaîne X-Forwarded-For
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # Protocole (http ou https)
    proxy_set_header X-Forwarded-Proto $scheme;

    # Host original
    proxy_set_header X-Forwarded-Host $host;

    # Port original
    proxy_set_header X-Forwarded-Port $server_port;

    # Host pour backend
    proxy_set_header Host $host;
}
```

### Lecture côté backend

```python
from flask import Flask, request

app = Flask(__name__)

def get_real_ip():
    """
    Obtenir l'IP réelle du client en tenant compte des proxys
    """
    # Si derrière un proxy de confiance
    if request.headers.get('X-Real-IP'):
        return request.headers.get('X-Real-IP')

    # X-Forwarded-For peut contenir plusieurs IPs
    forwarded_for = request.headers.get('X-Forwarded-For')
    if forwarded_for:
        # Prendre la première IP (client original)
        return forwarded_for.split(',')[0].strip()

    # Fallback sur remote_addr
    return request.remote_addr

def get_real_protocol():
    """Obtenir le protocole original (http ou https)"""
    return request.headers.get('X-Forwarded-Proto', 'http')

def get_full_url():
    """Reconstruire l'URL complète originale"""
    scheme = get_real_protocol()
    host = request.headers.get('X-Forwarded-Host', request.host)
    path = request.path
    query = request.query_string.decode()

    url = f"{scheme}://{host}{path}"
    if query:
        url += f"?{query}"

    return url

@app.route('/debug')
def debug():
    return {
        'real_ip': get_real_ip(),
        'protocol': get_real_protocol(),
        'full_url': get_full_url(),
        'all_headers': dict(request.headers)
    }

# Middleware pour logger avec IP réelle
@app.before_request
def log_request():
    app.logger.info(f"{get_real_ip()} - {request.method} {get_full_url()}")
```

### Sécurité : Validation des headers

**Problème :** Un attaquant peut injecter des headers `X-Forwarded-*` s'il accède directement au backend.

**Solution :** Valider que la requête vient d'un proxy de confiance.

```python
from flask import Flask, request, abort

app = Flask(__name__)

# IPs de confiance (nos reverse proxys)
TRUSTED_PROXIES = ['10.0.1.10', '10.0.1.11', '192.168.1.1']

@app.before_request
def validate_proxy_headers():
    """
    Valider que les headers X-Forwarded-* viennent d'un proxy de confiance
    """
    # Si pas derrière un proxy de confiance
    if request.remote_addr not in TRUSTED_PROXIES:
        # Supprimer les headers potentiellement forgés
        request.environ.pop('HTTP_X_REAL_IP', None)
        request.environ.pop('HTTP_X_FORWARDED_FOR', None)
        request.environ.pop('HTTP_X_FORWARDED_PROTO', None)

def get_real_ip():
    """Obtenir l'IP réelle (sécurisée)"""
    # Si pas derrière un proxy de confiance
    if request.remote_addr not in TRUSTED_PROXIES:
        return request.remote_addr

    # Headers de confiance
    return request.headers.get('X-Real-IP', request.remote_addr)
```

## Monitoring et observabilité

### Métriques essentielles d'un reverse proxy

```python
from prometheus_client import Counter, Histogram, Gauge, Summary
import time
from functools import wraps

# Requêtes totales
proxy_requests_total = Counter(
    'proxy_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Latence
proxy_request_duration = Histogram(
    'proxy_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 2.5, 5.0, 10.0]
)

# Backend latency
backend_duration = Histogram(
    'proxy_backend_duration_seconds',
    'Backend response time',
    ['backend'],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 2.5, 5.0]
)

# Connexions actives
proxy_active_connections = Gauge(
    'proxy_active_connections',
    'Number of active connections'
)

# Cache hits/misses
cache_hits = Counter('proxy_cache_hits_total', 'Cache hits')
cache_misses = Counter('proxy_cache_misses_total', 'Cache misses')

# Taux d'erreur backend
backend_errors = Counter(
    'proxy_backend_errors_total',
    'Backend errors',
    ['backend', 'error_type']
)

# Taille des réponses
response_size = Summary(
    'proxy_response_size_bytes',
    'HTTP response size'
)

def monitor_request(f):
    """Décorateur pour monitorer les requêtes"""
    @wraps(f)
    def wrapper(*args, **kwargs):
        method = request.method
        endpoint = request.endpoint or 'unknown'

        proxy_active_connections.inc()
        start_time = time.time()

        try:
            response = f(*args, **kwargs)
            status = response.status_code

            # Métriques
            proxy_requests_total.labels(
                method=method,
                endpoint=endpoint,
                status=status
            ).inc()

            duration = time.time() - start_time
            proxy_request_duration.labels(
                method=method,
                endpoint=endpoint
            ).observe(duration)

            # Taille réponse
            if hasattr(response, 'content_length') and response.content_length:
                response_size.observe(response.content_length)

            return response

        except Exception as e:
            backend_errors.labels(
                backend='unknown',
                error_type=type(e).__name__
            ).inc()
            raise

        finally:
            proxy_active_connections.dec()

    return wrapper
```

### Dashboard Grafana

```yaml
# Queries Prometheus pour monitoring reverse proxy

# Request rate
rate(proxy_requests_total[5m])

# Error rate
rate(proxy_requests_total{status=~"5.."}[5m])
/
rate(proxy_requests_total[5m])

# P95 latency
histogram_quantile(0.95,
  sum(rate(proxy_request_duration_seconds_bucket[5m])) by (le, endpoint)
)

# Cache hit ratio
rate(proxy_cache_hits_total[5m])
/
(rate(proxy_cache_hits_total[5m]) + rate(proxy_cache_misses_total[5m]))

# Backend latency distribution
histogram_quantile(0.99,
  sum(rate(proxy_backend_duration_seconds_bucket[5m])) by (le, backend)
)

# Active connections
proxy_active_connections

# Throughput (requests/sec)
sum(rate(proxy_requests_total[1m]))
```

## Pièges courants et solutions

### Piège 1 : Double encoding d'URL

**Problème :** Les caractères spéciaux sont encodés deux fois.

```nginx
# ❌ MAUVAIS
location /api/ {
    proxy_pass http://backend/api/;  # Double encoding !
}

# ✅ BON
location /api/ {
    proxy_pass http://backend;
    # OU
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass http://backend;
}
```

### Piège 2 : Timeout trop court pour long-polling

**Problème :** Connexions long-polling coupées prématurément.

```nginx
# Configuration pour long-polling / SSE
location /events {
    proxy_pass http://backend;

    # Timeouts longs
    proxy_read_timeout 300s;
    proxy_connect_timeout 5s;
    proxy_send_timeout 300s;

    # Buffering désactivé (streaming)
    proxy_buffering off;

    # Headers pour keep-alive
    proxy_http_version 1.1;
    proxy_set_header Connection "";
}
```

### Piège 3 : Perte de WebSocket

**Problème :** WebSocket ne fonctionne pas à travers le proxy.

```nginx
# Configuration WebSocket
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    location /ws {
        proxy_pass http://backend;

        # Headers WebSocket
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        # Timeouts
        proxy_read_timeout 86400s;  # 24h
        proxy_send_timeout 86400s;
    }
}
```

### Piège 4 : Headers Host incorrects

**Problème :** Le backend reçoit le mauvais hostname.

```nginx
# ❌ MAUVAIS
location / {
    proxy_pass http://backend;
    # Host header = "backend" au lieu du host original
}

# ✅ BON
location / {
    proxy_pass http://backend;
    proxy_set_header Host $host;  # Préserve le host original
}
```

### Piège 5 : Boucles de redirection

**Problème :** Redirection infinie entre HTTP et HTTPS.

```nginx
# ❌ MAUVAIS : Boucle si le backend redirige vers HTTP
server {
    listen 443 ssl;

    location / {
        proxy_pass http://backend;  # Backend redirige vers HTTPS
    }                               # → boucle infinie
}

# ✅ BON : Informer le backend qu'on est en HTTPS
server {
    listen 443 ssl;

    location / {
        proxy_pass http://backend;
        proxy_set_header X-Forwarded-Proto https;  # Backend sait qu'on est en HTTPS
    }
}
```

## Comparaison des solutions

| Solution | Type | Meilleur pour | Courbe apprentissage | Performance |
|----------|------|---------------|---------------------|-------------|
| **Nginx** | Reverse Proxy | Web apps, static files | Moyenne | Excellente |
| **HAProxy** | Load Balancer + Proxy | TCP/HTTP LB, haute perf | Moyenne-Élevée | Excellente |
| **Squid** | Forward Proxy | Cache entreprise, filtering | Moyenne | Bonne |
| **Traefik** | Reverse Proxy | Docker/K8s, auto-config | Faible | Bonne |
| **Envoy** | Service Mesh | Microservices, observability | Élevée | Excellente |
| **Kong** | API Gateway | APIs, plugins riches | Moyenne | Bonne |
| **Apache** | Web Server + Proxy | Legacy, mod_proxy | Faible | Moyenne |

## Résumé : Choisir le bon proxy

### Forward Proxy : Quand l'utiliser ?

✅ **Utilisez un forward proxy si :**
- Contrôle d'accès internet en entreprise
- Cache partagé pour économiser bande passante
- Anonymisation du trafic sortant
- Logs et audit du trafic employés

### Reverse Proxy : Quand l'utiliser ?

✅ **Utilisez un reverse proxy si :**
- SSL termination centralisée
- Load balancing applicatif
- Caching de contenu
- Protection des backends (WAF, rate limiting)
- Compression automatique
- Routing basé sur URL/headers

### API Gateway : Quand l'utiliser ?

✅ **Utilisez un API Gateway si :**
- Architecture microservices
- Besoin d'authentification centralisée
- Rate limiting par client/API key
- Transformation de requêtes
- Service discovery dynamique

### Service Mesh : Quand l'utiliser ?

✅ **Utilisez un service mesh si :**
- Nombreux microservices (>10)
- Besoin de mTLS automatique
- Observabilité fine (traces distribuées)
- Gestion complexe du trafic (canary, A/B)
- Circuit breaking avancé

## Conclusion

Les proxys et reverse proxys sont des **composants essentiels** des architectures modernes. Ils offrent :

**Pour les forward proxys :**
- Contrôle et sécurité du trafic sortant
- Optimisation via cache
- Anonymisation

**Pour les reverse proxys :**
- Protection et isolation des backends
- Performance via SSL termination et cache
- Flexibilité via routing intelligent
- Observabilité centralisée

**À retenir :**
- Un **forward proxy** protège les clients
- Un **reverse proxy** protège les serveurs
- Toujours **valider les headers** X-Forwarded-*
- Configurer les **timeouts** selon le use case
- **Monitorer** les métriques clés (latence, erreurs, cache hit ratio)
- Choisir la solution selon l'architecture (simple web vs microservices)

**Prochaine étape :** La section suivante explore les **CDN (Content Delivery Networks)**, qui étendent le concept de reverse proxy à l'échelle mondiale pour servir du contenu au plus près des utilisateurs.

---


⏭️ [CDN (Content Delivery Networks)](/09-architectures-avancees/05-cdn.md)
