🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.5 CDN (Content Delivery Networks)

## Introduction

Un **Content Delivery Network (CDN)** est un réseau distribué de serveurs géographiquement dispersés qui travaillent ensemble pour **livrer du contenu au plus près des utilisateurs finaux**. C'est essentiellement un reverse proxy distribué à l'échelle mondiale, optimisé pour la performance et la disponibilité.

### L'analogie de la franchise de restauration

**Sans CDN (restaurant unique) :**
```
Paris          →  [5000 km]  →  Serveur à New York
Tokyo          →  [11000 km] →  Serveur à New York
Sydney         →  [16000 km] →  Serveur à New York

Résultat :
- Latence énorme (200-500ms)
- Saturation du serveur unique
- Bande passante internationale coûteuse
- Point de défaillance unique
```

**Avec CDN (chaîne mondiale) :**
```
Paris    → [10 km]   → Serveur CDN Paris (cache local)
Tokyo    → [15 km]   → Serveur CDN Tokyo (cache local)
Sydney   → [20 km]   → Serveur CDN Sydney (cache local)
                              ↓
                        [Origin servers]
                        (sollicités rarement)

Résultat :
- Latence minimale (10-50ms)
- Charge distribuée
- Bande passante locale moins chère
- Haute disponibilité (redondance)
```

### Le problème que résout un CDN

**Scénario réel :** Vous lancez un site e-commerce avec un serveur unique à Paris.

```
SANS CDN
═══════════════════════════════════════════════════

Client USA (New York)
  ↓ [RTT 80ms × 3 (TCP + TLS) = 240ms]
  ↓ [Request = 80ms]
  ↓ [Response (2MB) = 80ms + transfer time]
  ↓ [TOTAL : ~1500ms pour charger la page]

Serveur Paris (origin)
  • Saturé par le trafic mondial
  • Bande passante internationale coûteuse
  • 99% du trafic : mêmes assets statiques


AVEC CDN
═══════════════════════════════════════════════════

Client USA (New York)
  ↓ [RTT 5ms × 3 = 15ms] (edge server local)
  ↓ [Request = 5ms]
  ↓ [Response (2MB, depuis cache) = 5ms + transfer time local]
  ↓ [TOTAL : ~150ms pour charger la page]

Serveur Paris (origin)
  • Sollicité uniquement pour :
    - Cache miss (rare)
    - Contenu dynamique
    - Invalidations
  • Économie de 95%+ de bande passante
```

## Architecture d'un CDN

### Points of Presence (PoP)

Un CDN est constitué de **dizaines à centaines de PoPs** répartis mondialement.

```
┌─────────────────────────────────────────────────────────┐
│                    Global CDN Network                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  [Amérique du Nord]     [Europe]        [Asie-Pacifique]│
│   • New York (PoP)      • Paris          • Tokyo        │
│   • Los Angeles         • Londres        • Singapore    │
│   • Toronto             • Francfort      • Sydney       │
│   • Chicago             • Amsterdam      • Mumbai       │
│   • Dallas              • Stockholm      • Seoul        │
│                                                         │
│  [Amérique du Sud]      [Moyen-Orient]   [Afrique]      │
│   • São Paulo           • Dubai          • Johannesburg │
│   • Buenos Aires        • Tel Aviv       • Le Caire     │
│                                                         │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ↓
                   [Origin Servers]
                   (vos serveurs)
```

### Flux de requête dans un CDN

```
1. DNS Resolution (GeoDNS)
   ═══════════════════════════════════════

   Client (Paris) → DNS query www.example.com
                    ↓
              GeoDNS intelligent
                    ↓
          Retourne IP du PoP le plus proche
                    ↓
          cdn-paris-01.example.com (151.101.1.10)


2. Connection au PoP
   ═══════════════════════════════════════

   Client → Edge Server (Paris PoP)
             ↓
        Cache HIT ?
             ↓
        ┌────┴────┐
        YES       NO
        ↓         ↓
   Retourne    Fetch from origin
   du cache    ├─→ Origin (vos serveurs)
               ├─→ Mise en cache
               └─→ Retour au client


3. Cache Management
   ═══════════════════════════════════════

   Edge Server
    ├─ Hot Cache (RAM) : 5-10% le plus populaire
    ├─ Warm Cache (SSD) : 40-50% fréquent
    └─ Cold : fetch from origin
```

## Fonctionnement technique détaillé

### 1. Anycast IP

Les CDN utilisent **Anycast** pour router automatiquement vers le PoP le plus proche.

```
Principe Anycast :
═══════════════════════════════════════

Même IP publique (ex: 151.101.1.10) annoncée par TOUS les PoPs

Client Paris → ping 151.101.1.10
              BGP routing → PoP Paris (6ms)

Client Tokyo → ping 151.101.1.10
              BGP routing → PoP Tokyo (8ms)

Client NY    → ping 151.101.1.10
              BGP routing → PoP New York (5ms)

Automatique via BGP !
```

### 2. Tiered Caching

Les CDN modernes utilisent plusieurs niveaux de cache.

```
┌─────────────────────────────────────────────────────┐
│  Tier 1 : Edge PoPs (200+ locations)                │
│  • Plus proche des utilisateurs                     │
│  • Cache "hot" content (populaire)                  │
│  • TTL court (minutes à heures)                     │
├─────────────────────────────────────────────────────┤
│  Tier 2 : Regional PoPs (20-50 locations)           │
│  • Agrège plusieurs edge PoPs                       │
│  • Cache intermédiaire plus grand                   │
│  • TTL moyen (heures à jours)                       │
├─────────────────────────────────────────────────────┤
│  Tier 3 : Origin Shield (2-5 locations)             │
│  • Protège l'origin                                 │
│  • Collapse requests (deduplication)                │
│  • Cache très large                                 │
├─────────────────────────────────────────────────────┤
│  Origin : Vos serveurs (1-3 locations)              │
│  • Sollicité uniquement en cache miss complet       │
└─────────────────────────────────────────────────────┘

Flux de requête (cache miss) :

Client → Edge PoP (miss)
         ↓
      Regional PoP (miss)
         ↓
      Origin Shield (miss)
         ↓
      Origin Server ✓
         ↓
      ← Cache à tous les niveaux
         ↓
      Client reçoit la réponse
```

### 3. Cache Key et Variants

Le CDN doit savoir **quelles versions** d'un contenu cacher.

```
Cache Key Components :
═══════════════════════════════════════

URL : https://example.com/style.css?v=2
      └─ Cache key de base

+ Query String : ?v=2
                  └─ Variant par version

+ Headers :
  - Accept-Encoding: gzip, br
    └─ Variant : version compressée

  - Accept-Language: fr-FR
    └─ Variant : version française

  - User-Agent: Mobile
    └─ Variant : version mobile (optionnel)

  - Cookie: session=xyz
    └─ Bypass cache (personnalisé)

Résultat : Plusieurs versions cachées du même fichier
```

**Configuration des cache keys (Cloudflare) :**

```javascript
// Cloudflare Worker - Custom cache key
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)

  // Construire une cache key custom
  const cacheKey = new Request(
    url.toString(),
    {
      method: 'GET',
      headers: {
        // Inclure certains headers dans la cache key
        'Accept-Encoding': request.headers.get('Accept-Encoding') || '',
        'Accept-Language': request.headers.get('Accept-Language') || 'en',
      }
    }
  )

  // Essayer de récupérer du cache
  const cache = caches.default
  let response = await cache.match(cacheKey)

  if (!response) {
    // Cache miss - fetch from origin
    response = await fetch(request)

    // Mettre en cache si réussite
    if (response.ok) {
      // Cloner la réponse (consumed une fois lue)
      response = new Response(response.body, response)

      // Headers de cache
      response.headers.set('Cache-Control', 'public, max-age=3600')

      // Stocker dans le cache edge
      event.waitUntil(cache.put(cacheKey, response.clone()))
    }
  } else {
    // Cache HIT - ajouter header
    response = new Response(response.body, response)
    response.headers.set('X-Cache', 'HIT')
  }

  return response
}
```

## Configuration pratique : Cloudflare

### Setup de base

```javascript
// cloudflare-config.js
// Configuration via Cloudflare API

const CLOUDFLARE_API_TOKEN = 'your-api-token'
const ZONE_ID = 'your-zone-id'

const cloudflareAPI = async (endpoint, method = 'GET', data = null) => {
  const response = await fetch(
    `https://api.cloudflare.com/client/v4${endpoint}`,
    {
      method,
      headers: {
        'Authorization': `Bearer ${CLOUDFLARE_API_TOKEN}`,
        'Content-Type': 'application/json',
      },
      body: data ? JSON.stringify(data) : null
    }
  )
  return response.json()
}

// 1. Créer une Page Rule pour caching agressif
async function createCacheRule() {
  const rule = {
    targets: [{
      target: 'url',
      constraint: {
        operator: 'matches',
        value: `example.com/static/*`
      }
    }],
    actions: [
      {
        id: 'cache_level',
        value: 'cache_everything'
      },
      {
        id: 'edge_cache_ttl',
        value: 2592000  // 30 jours
      },
      {
        id: 'browser_cache_ttl',
        value: 604800  // 7 jours
      }
    ],
    priority: 1,
    status: 'active'
  }

  return await cloudflareAPI(
    `/zones/${ZONE_ID}/pagerules`,
    'POST',
    rule
  )
}

// 2. Configurer cache par type de fichier
async function configureCacheByFileType() {
  const config = {
    // Images
    '*.jpg|*.png|*.gif|*.webp': {
      edge_ttl: 2592000,  // 30 jours
      browser_ttl: 604800  // 7 jours
    },

    // CSS/JS
    '*.css|*.js': {
      edge_ttl: 604800,   // 7 jours
      browser_ttl: 86400   // 1 jour
    },

    // Fonts
    '*.woff|*.woff2|*.ttf': {
      edge_ttl: 31536000,  // 1 an
      browser_ttl: 31536000
    },

    // HTML
    '*.html': {
      edge_ttl: 3600,      // 1 heure
      browser_ttl: 300      // 5 minutes
    }
  }

  // Créer des rules pour chaque type
  for (const [pattern, ttl] of Object.entries(config)) {
    // Implémenter via Page Rules ou Workers
  }
}

// 3. Purge du cache
async function purgeCache(urls = []) {
  if (urls.length === 0) {
    // Purge tout
    return await cloudflareAPI(
      `/zones/${ZONE_ID}/purge_cache`,
      'POST',
      { purge_everything: true }
    )
  } else {
    // Purge sélectif
    return await cloudflareAPI(
      `/zones/${ZONE_ID}/purge_cache`,
      'POST',
      { files: urls }
    )
  }
}

// 4. Purge par tags (Cloudflare Enterprise)
async function purgeCacheByTag(tags) {
  return await cloudflareAPI(
    `/zones/${ZONE_ID}/purge_cache`,
    'POST',
    { tags }
  )
}

// Usage
await createCacheRule()
await purgeCache([
  'https://example.com/style.css',
  'https://example.com/app.js'
])
```

### Cloudflare Workers (Edge Computing)

```javascript
// worker.js - Logique exécutée sur l'edge

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)

  // 1. Optimisation d'images à la volée
  if (url.pathname.match(/\.(jpg|png|gif)$/)) {
    return optimizeImage(request)
  }

  // 2. A/B Testing
  if (url.pathname === '/') {
    return abTest(request)
  }

  // 3. Geolocation redirect
  if (url.pathname === '/auto') {
    return geoRedirect(request)
  }

  // 4. API caching with stale-while-revalidate
  if (url.pathname.startsWith('/api/')) {
    return apiCache(request)
  }

  // Default : fetch from origin
  return fetch(request)
}

// Optimisation d'images
async function optimizeImage(request) {
  const url = new URL(request.url)

  // Déterminer le format optimal selon Accept header
  const accept = request.headers.get('Accept') || ''
  let format = 'auto'

  if (accept.includes('image/webp')) {
    format = 'webp'
  } else if (accept.includes('image/avif')) {
    format = 'avif'
  }

  // Cloudflare Image Resizing
  const imageURL = `/cdn-cgi/image/format=${format},quality=85,width=800/${url.pathname}`

  return fetch(new Request(url.origin + imageURL, request))
}

// A/B Testing
async function abTest(request) {
  const cookie = request.headers.get('Cookie') || ''

  // Vérifier si déjà assigné à une variante
  let variant = cookie.match(/ab_test=([AB])/)

  if (!variant) {
    // Nouvelle assignation (50/50)
    variant = Math.random() < 0.5 ? 'A' : 'B'

    const response = await fetch(request)
    const newResponse = new Response(response.body, response)

    // Définir le cookie
    newResponse.headers.set(
      'Set-Cookie',
      `ab_test=${variant}; Path=/; Max-Age=2592000; Secure; HttpOnly`
    )

    return newResponse
  }

  variant = variant[1]

  // Router vers la bonne version
  if (variant === 'B') {
    return fetch('/index-variant-b.html')
  }

  return fetch(request)
}

// Geo-redirection
async function geoRedirect(request) {
  const country = request.cf.country

  const countryDomains = {
    'US': 'https://us.example.com',
    'GB': 'https://uk.example.com',
    'FR': 'https://fr.example.com',
    'DE': 'https://de.example.com',
    'JP': 'https://jp.example.com'
  }

  const targetDomain = countryDomains[country] || 'https://www.example.com'

  return Response.redirect(targetDomain, 302)
}

// API caching avec stale-while-revalidate
async function apiCache(request) {
  const cache = caches.default
  const url = new URL(request.url)

  // Vérifier le cache
  let response = await cache.match(request)

  if (response) {
    // Cache HIT
    const age = Date.now() - new Date(response.headers.get('Date')).getTime()
    const maxAge = 60000  // 1 minute

    if (age < maxAge) {
      // Cache frais
      response.headers.set('X-Cache', 'HIT')
      return response
    } else {
      // Cache stale - renvoyer quand même mais revalider en background
      response.headers.set('X-Cache', 'STALE')

      // Revalider en background
      event.waitUntil(
        fetch(request)
          .then(freshResponse => {
            const cloned = freshResponse.clone()
            cache.put(request, cloned)
          })
      )

      return response
    }
  }

  // Cache MISS
  response = await fetch(request)

  if (response.ok) {
    const cloned = response.clone()
    event.waitUntil(cache.put(request, cloned))
  }

  response.headers.set('X-Cache', 'MISS')
  return response
}
```

## Configuration : AWS CloudFront

### Infrastructure as Code (Terraform)

```hcl
# cloudfront.tf

# Bucket S3 pour le contenu statique
resource "aws_s3_bucket" "static_content" {
  bucket = "myapp-static-content"
}

# Origin Access Identity (CloudFront peut accéder au bucket privé)
resource "aws_cloudfront_origin_access_identity" "oai" {
  comment = "OAI for static content"
}

# Politique S3 pour autoriser CloudFront
resource "aws_s3_bucket_policy" "static_content_policy" {
  bucket = aws_s3_bucket.static_content.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = aws_cloudfront_origin_access_identity.oai.iam_arn
        }
        Action = "s3:GetObject"
        Resource = "${aws_s3_bucket.static_content.arn}/*"
      }
    ]
  })
}

# Distribution CloudFront
resource "aws_cloudfront_distribution" "cdn" {
  enabled             = true
  is_ipv6_enabled     = true
  comment             = "Production CDN"
  default_root_object = "index.html"
  price_class         = "PriceClass_All"  # Tous les edge locations

  # Domaines custom
  aliases = ["www.example.com", "example.com"]

  # Origin 1 : Bucket S3 (static)
  origin {
    domain_name = aws_s3_bucket.static_content.bucket_regional_domain_name
    origin_id   = "S3-static"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.oai.cloudfront_access_identity_path
    }

    # Custom headers
    custom_header {
      name  = "X-Origin-Verify"
      value = "secret-token-xyz"
    }
  }

  # Origin 2 : API origin (custom)
  origin {
    domain_name = "api.example.com"
    origin_id   = "API-origin"

    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
      origin_ssl_protocols   = ["TLSv1.2"]
      origin_read_timeout    = 60
      origin_keepalive_timeout = 5
    }
  }

  # Comportement par défaut (S3)
  default_cache_behavior {
    target_origin_id       = "S3-static"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD", "OPTIONS"]
    cached_methods         = ["GET", "HEAD"]
    compress               = true

    # Cache policy
    cache_policy_id = aws_cloudfront_cache_policy.optimized.id

    # Lambda@Edge (optionnel)
    lambda_function_association {
      event_type   = "viewer-request"
      lambda_arn   = aws_lambda_function.edge_auth.qualified_arn
      include_body = false
    }
  }

  # Comportement pour l'API (pas de cache)
  ordered_cache_behavior {
    path_pattern           = "/api/*"
    target_origin_id       = "API-origin"
    viewer_protocol_policy = "https-only"
    allowed_methods        = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods         = ["GET", "HEAD"]
    compress               = true

    # Pas de cache pour API
    cache_policy_id = aws_cloudfront_cache_policy.api_nocache.id

    # Forward all headers
    forwarded_values {
      query_string = true
      headers      = ["*"]

      cookies {
        forward = "all"
      }
    }
  }

  # Comportement pour images (cache agressif)
  ordered_cache_behavior {
    path_pattern           = "/images/*"
    target_origin_id       = "S3-static"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]
    compress               = true

    # Cache très long
    min_ttl     = 0
    default_ttl = 2592000  # 30 jours
    max_ttl     = 31536000  # 1 an

    forwarded_values {
      query_string = false

      cookies {
        forward = "none"
      }
    }
  }

  # Certificat SSL
  viewer_certificate {
    acm_certificate_arn      = aws_acm_certificate.cert.arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }

  # Restrictions géographiques (optionnel)
  restrictions {
    geo_restriction {
      restriction_type = "none"
      # ou "whitelist"/"blacklist" avec locations = ["US", "GB"]
    }
  }

  # Pages d'erreur custom
  custom_error_response {
    error_code         = 404
    response_code      = 404
    response_page_path = "/404.html"
    error_caching_min_ttl = 300
  }

  custom_error_response {
    error_code         = 403
    response_code      = 403
    response_page_path = "/403.html"
  }

  # Logging
  logging_config {
    include_cookies = false
    bucket          = aws_s3_bucket.logs.bucket_domain_name
    prefix          = "cloudfront/"
  }

  tags = {
    Environment = "production"
    Project     = "myapp"
  }
}

# Cache Policy optimisée
resource "aws_cloudfront_cache_policy" "optimized" {
  name        = "Optimized-Cache-Policy"
  comment     = "Policy for static assets"
  default_ttl = 86400    # 1 jour
  max_ttl     = 31536000  # 1 an
  min_ttl     = 0

  parameters_in_cache_key_and_forwarded_to_origin {
    enable_accept_encoding_brotli = true
    enable_accept_encoding_gzip   = true

    cookies_config {
      cookie_behavior = "none"
    }

    headers_config {
      header_behavior = "whitelist"
      headers {
        items = ["Accept-Encoding", "CloudFront-Viewer-Country"]
      }
    }

    query_strings_config {
      query_string_behavior = "whitelist"
      query_strings {
        items = ["v", "version"]
      }
    }
  }
}

# Cache Policy pour API (no cache)
resource "aws_cloudfront_cache_policy" "api_nocache" {
  name        = "API-No-Cache"
  default_ttl = 0
  max_ttl     = 0
  min_ttl     = 0

  parameters_in_cache_key_and_forwarded_to_origin {
    enable_accept_encoding_brotli = false
    enable_accept_encoding_gzip   = false

    cookies_config {
      cookie_behavior = "all"
    }

    headers_config {
      header_behavior = "all"
    }

    query_strings_config {
      query_string_behavior = "all"
    }
  }
}
```

### Lambda@Edge pour logique custom

```python
# lambda_edge_auth.py
# Lambda@Edge pour authentification

import json
import base64

def lambda_handler(event, context):
    """
    Viewer Request : Exécuté avant le cache lookup
    Peut modifier la requête ou retourner une réponse
    """
    request = event['Records'][0]['cf']['request']
    headers = request['headers']

    # 1. Vérifier authentification
    auth_header = headers.get('authorization', [{}])[0].get('value', '')

    if not auth_header.startswith('Bearer '):
        return {
            'status': '401',
            'statusDescription': 'Unauthorized',
            'headers': {
                'www-authenticate': [{
                    'key': 'WWW-Authenticate',
                    'value': 'Bearer realm="api"'
                }]
            },
            'body': json.dumps({'error': 'Authentication required'})
        }

    # 2. Extraire et valider le token (simplifié)
    token = auth_header[7:]  # Enlever "Bearer "

    # En production : valider JWT, vérifier signature, etc.
    # Pour l'exemple : vérification simple
    if token != 'valid-token-xyz':
        return {
            'status': '403',
            'statusDescription': 'Forbidden',
            'body': json.dumps({'error': 'Invalid token'})
        }

    # 3. Ajouter des headers custom basés sur le user
    headers['x-user-id'] = [{
        'key': 'X-User-ID',
        'value': '123'
    }]

    # 4. Modifier la cache key selon le user tier
    user_tier = 'premium'  # Extraire du token

    if user_tier == 'premium':
        # Variante premium (cache séparé)
        headers['x-cache-variant'] = [{
            'key': 'X-Cache-Variant',
            'value': 'premium'
        }]

    return request


def origin_response_handler(event, context):
    """
    Origin Response : Exécuté après réponse de l'origin, avant mise en cache
    """
    response = event['Records'][0]['cf']['response']
    headers = response['headers']

    # Ajouter des security headers
    headers['strict-transport-security'] = [{
        'key': 'Strict-Transport-Security',
        'value': 'max-age=31536000; includeSubDomains; preload'
    }]

    headers['x-content-type-options'] = [{
        'key': 'X-Content-Type-Options',
        'value': 'nosniff'
    }]

    headers['x-frame-options'] = [{
        'key': 'X-Frame-Options',
        'value': 'SAMEORIGIN'
    }]

    # Modifier le TTL de cache selon le contenu
    content_type = headers.get('content-type', [{}])[0].get('value', '')

    if 'image/' in content_type:
        # Images : cache long
        headers['cache-control'] = [{
            'key': 'Cache-Control',
            'value': 'public, max-age=2592000, immutable'
        }]
    elif 'text/html' in content_type:
        # HTML : cache court
        headers['cache-control'] = [{
            'key': 'Cache-Control',
            'value': 'public, max-age=300, must-revalidate'
        }]

    return response
```

## Stratégies de caching avancées

### 1. Cache Headers Strategy

```python
from flask import Flask, make_response, request
from datetime import datetime, timedelta

app = Flask(__name__)

def set_cache_headers(response, cache_type='default'):
    """
    Configurer les headers de cache selon le type de contenu
    """
    strategies = {
        'immutable': {
            # Assets avec hash dans le nom (style.a3f2d1.css)
            'Cache-Control': 'public, max-age=31536000, immutable',
            'Expires': (datetime.utcnow() + timedelta(days=365)).strftime('%a, %d %b %Y %H:%M:%S GMT')
        },
        'static': {
            # Assets statiques sans hash
            'Cache-Control': 'public, max-age=604800',  # 7 jours
            'Expires': (datetime.utcnow() + timedelta(days=7)).strftime('%a, %d %b %Y %H:%M:%S GMT')
        },
        'dynamic': {
            # Contenu qui change modérément
            'Cache-Control': 'public, max-age=300, must-revalidate',  # 5 min
            'Expires': (datetime.utcnow() + timedelta(minutes=5)).strftime('%a, %d %b %Y %H:%M:%S GMT')
        },
        'private': {
            # Contenu personnalisé
            'Cache-Control': 'private, max-age=60',
        },
        'no-cache': {
            # Pas de cache (API, contenu sensible)
            'Cache-Control': 'no-store, no-cache, must-revalidate, proxy-revalidate',
            'Pragma': 'no-cache',
            'Expires': '0'
        }
    }

    headers = strategies.get(cache_type, strategies['default'])

    for key, value in headers.items():
        response.headers[key] = value

    return response

@app.route('/assets/<path:filename>')
def serve_asset(filename):
    """Assets avec versioning (immutable)"""
    response = make_response(send_file(f'assets/{filename}'))

    # Si le fichier a un hash dans le nom
    if '.' in filename and len(filename.split('.')[-2]) == 8:
        # Exemple: style.a3f2d1b4.css
        set_cache_headers(response, 'immutable')
    else:
        set_cache_headers(response, 'static')

    return response

@app.route('/api/data')
def api_data():
    """API avec cache court et revalidation"""
    data = {'users': 1000, 'timestamp': datetime.utcnow().isoformat()}
    response = make_response(data)

    # Cache court avec ETag pour revalidation
    import hashlib
    etag = hashlib.md5(str(data).encode()).hexdigest()
    response.headers['ETag'] = etag

    # Vérifier If-None-Match
    if request.headers.get('If-None-Match') == etag:
        return '', 304  # Not Modified

    set_cache_headers(response, 'dynamic')
    return response

@app.route('/user/profile')
def user_profile():
    """Contenu personnalisé (cache privé)"""
    # Vérifier authentification
    user_id = request.headers.get('X-User-ID')

    profile = get_user_profile(user_id)
    response = make_response(profile)

    # Cache privé (navigateur seulement, pas CDN)
    set_cache_headers(response, 'private')

    # Vary header pour indiquer que le contenu varie selon le user
    response.headers['Vary'] = 'Cookie, X-User-ID'

    return response

@app.route('/checkout')
def checkout():
    """Page sensible (pas de cache)"""
    response = make_response(render_template('checkout.html'))
    set_cache_headers(response, 'no-cache')
    return response
```

### 2. Stale-While-Revalidate

```javascript
// Service Worker avec stale-while-revalidate

self.addEventListener('fetch', event => {
  // Seulement pour les requêtes GET
  if (event.request.method !== 'GET') {
    return event.respondWith(fetch(event.request))
  }

  event.respondWith(
    caches.open('dynamic-v1').then(cache => {
      return cache.match(event.request).then(cachedResponse => {
        // Fetch from network
        const fetchPromise = fetch(event.request).then(networkResponse => {
          // Mettre à jour le cache en background
          cache.put(event.request, networkResponse.clone())
          return networkResponse
        })

        // Retourner du cache si disponible, sinon attendre le réseau
        return cachedResponse || fetchPromise
      })
    })
  )
})
```

### 3. Cache Tagging et Invalidation granulaire

```python
from flask import Flask, make_response
import redis

app = Flask(__name__)
redis_client = redis.Redis()

class CacheTagger:
    """
    Système de tagging pour invalidation granulaire
    """
    def __init__(self, redis_client):
        self.redis = redis_client

    def tag_response(self, response, tags):
        """
        Ajouter des tags à une réponse
        Ces tags seront utilisés pour l'invalidation
        """
        # Cloudflare supporte Cache-Tag header
        response.headers['Cache-Tag'] = ','.join(tags)

        # Stocker la relation tag -> URL dans Redis
        url = request.url
        for tag in tags:
            self.redis.sadd(f'tag:{tag}', url)

        return response

    def invalidate_tag(self, tag):
        """
        Invalider toutes les URLs avec ce tag
        """
        # Récupérer toutes les URLs avec ce tag
        urls = self.redis.smembers(f'tag:{tag}')

        if not urls:
            return []

        # Purge CDN (Cloudflare example)
        purge_cloudflare_cache(list(urls))

        # Nettoyer Redis
        self.redis.delete(f'tag:{tag}')

        return list(urls)

tagger = CacheTagger(redis_client)

@app.route('/products/<int:product_id>')
def product_detail(product_id):
    """Page produit avec tagging"""
    product = get_product(product_id)
    category_id = product['category_id']

    response = make_response(render_template('product.html', product=product))

    # Tags pour invalidation granulaire
    tags = [
        f'product-{product_id}',
        f'category-{category_id}',
        'products'
    ]

    tagger.tag_response(response, tags)

    response.headers['Cache-Control'] = 'public, max-age=3600'

    return response

@app.route('/admin/products/<int:product_id>', methods=['PUT'])
def update_product(product_id):
    """
    Mettre à jour un produit et invalider le cache
    """
    # Mettre à jour en DB
    update_product_in_db(product_id, request.json)

    # Invalider le cache de ce produit
    invalidated = tagger.invalidate_tag(f'product-{product_id}')

    return {
        'success': True,
        'invalidated_urls': len(invalidated),
        'urls': invalidated
    }

@app.route('/admin/categories/<int:category_id>/refresh', methods=['POST'])
def refresh_category(category_id):
    """
    Invalider tous les produits d'une catégorie
    """
    invalidated = tagger.invalidate_tag(f'category-{category_id}')

    return {
        'success': True,
        'message': f'Invalidated {len(invalidated)} product pages'
    }
```

## Cache Purging et Invalidation

### Stratégies d'invalidation

```python
import requests
from typing import List, Optional
from datetime import datetime

class CDNPurgeManager:
    """
    Gestionnaire d'invalidation multi-CDN
    """

    def __init__(self):
        self.cloudflare_token = 'your-token'
        self.cloudflare_zone = 'zone-id'
        self.cloudfront_distribution = 'distribution-id'

    def purge_cloudflare(self, urls: Optional[List[str]] = None,
                        tags: Optional[List[str]] = None,
                        purge_all: bool = False):
        """
        Purge Cloudflare cache

        Options :
        - urls : liste d'URLs spécifiques
        - tags : liste de tags (Enterprise seulement)
        - purge_all : purger tout le cache
        """
        endpoint = f'https://api.cloudflare.com/client/v4/zones/{self.cloudflare_zone}/purge_cache'

        headers = {
            'Authorization': f'Bearer {self.cloudflare_token}',
            'Content-Type': 'application/json'
        }

        if purge_all:
            data = {'purge_everything': True}
        elif tags:
            data = {'tags': tags}
        elif urls:
            data = {'files': urls}
        else:
            raise ValueError("Must specify urls, tags, or purge_all")

        response = requests.post(endpoint, headers=headers, json=data)

        return response.json()

    def purge_cloudfront(self, paths: List[str]):
        """
        Purge AWS CloudFront cache
        """
        import boto3

        client = boto3.client('cloudfront')

        # Créer une invalidation
        response = client.create_invalidation(
            DistributionId=self.cloudfront_distribution,
            InvalidationBatch={
                'Paths': {
                    'Quantity': len(paths),
                    'Items': paths
                },
                'CallerReference': f'purge-{datetime.utcnow().timestamp()}'
            }
        )

        return response

    def purge_pattern(self, pattern: str, cdn='cloudflare'):
        """
        Purge selon un pattern (ex: /images/*)
        """
        if cdn == 'cloudflare':
            # Cloudflare ne supporte pas les wildcards directement
            # Il faut purger par prefix ou par tag
            return self.purge_cloudflare(purge_all=True)

        elif cdn == 'cloudfront':
            # CloudFront supporte les wildcards
            return self.purge_cloudfront([pattern])

    def smart_purge(self, resource_type: str, resource_id: int):
        """
        Purge intelligente basée sur le type de ressource
        """
        purge_map = {
            'product': [
                f'/products/{resource_id}',
                f'/api/products/{resource_id}',
                f'/images/products/{resource_id}/*'
            ],
            'category': [
                f'/categories/{resource_id}',
                f'/api/categories/{resource_id}',
                '/products/*'  # Tous les produits (peuvent afficher la catégorie)
            ],
            'user': [
                f'/users/{resource_id}',
                f'/api/users/{resource_id}'
            ]
        }

        paths = purge_map.get(resource_type, [])

        # Purge sur tous les CDN
        results = {
            'cloudflare': self.purge_cloudflare(urls=[
                f'https://example.com{path}' for path in paths
            ]),
            'cloudfront': self.purge_cloudfront(paths)
        }

        return results

# Usage
purger = CDNPurgeManager()

# Purge spécifique
purger.purge_cloudflare(urls=[
    'https://example.com/style.css',
    'https://example.com/app.js'
])

# Purge par tag
purger.purge_cloudflare(tags=['products', 'category-electronics'])

# Purge intelligente après modification
purger.smart_purge('product', product_id=123)
```

### Webhooks pour invalidation automatique

```python
from flask import Flask, request, jsonify
import hmac
import hashlib

app = Flask(__name__)

@app.route('/webhooks/content-updated', methods=['POST'])
def content_updated_webhook():
    """
    Webhook appelé quand le contenu est modifié (CMS, DB, etc.)
    Invalide automatiquement le cache CDN
    """
    # Vérifier la signature (sécurité)
    signature = request.headers.get('X-Webhook-Signature')

    if not verify_webhook_signature(request.data, signature):
        return jsonify({'error': 'Invalid signature'}), 403

    payload = request.json

    # Déterminer quoi invalider
    content_type = payload.get('type')
    content_id = payload.get('id')
    action = payload.get('action')  # 'created', 'updated', 'deleted'

    purger = CDNPurgeManager()

    if action in ['updated', 'deleted']:
        # Invalider le cache
        result = purger.smart_purge(content_type, content_id)

        return jsonify({
            'success': True,
            'purged': result,
            'message': f'{content_type} {content_id} cache invalidated'
        })

    return jsonify({'success': True, 'message': 'No purge needed'})

def verify_webhook_signature(payload, signature):
    """Vérifier la signature HMAC du webhook"""
    secret = 'webhook-secret-key'

    expected = hmac.new(
        secret.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(expected, signature)
```

## Optimisations avancées

### 1. Image Optimization at the Edge

```javascript
// Cloudflare Worker - Optimisation d'images dynamique

addEventListener('fetch', event => {
  event.respondWith(handleImageRequest(event.request))
})

async function handleImageRequest(request) {
  const url = new URL(request.url)

  // Vérifier si c'est une image
  if (!url.pathname.match(/\.(jpg|jpeg|png|gif|webp)$/i)) {
    return fetch(request)
  }

  // Parse query parameters pour dimensions
  const width = url.searchParams.get('w') || 'auto'
  const quality = url.searchParams.get('q') || '85'
  const format = url.searchParams.get('f') || 'auto'

  // Déterminer le format optimal
  const accept = request.headers.get('Accept') || ''
  let optimalFormat = format

  if (format === 'auto') {
    if (accept.includes('image/avif')) {
      optimalFormat = 'avif'  // Meilleure compression
    } else if (accept.includes('image/webp')) {
      optimalFormat = 'webp'
    }
  }

  // Construire l'URL de transformation Cloudflare
  const transformURL = `/cdn-cgi/image/format=${optimalFormat},quality=${quality},width=${width}${url.pathname}`

  const transformedRequest = new Request(
    url.origin + transformURL,
    request
  )

  // Cache key incluant les transformations
  const cacheKey = new Request(transformedRequest.url, transformedRequest)
  const cache = caches.default

  // Vérifier le cache
  let response = await cache.match(cacheKey)

  if (!response) {
    // Fetch et transformer
    response = await fetch(transformedRequest)

    // Mettre en cache si succès
    if (response.ok) {
      response = new Response(response.body, response)
      response.headers.set('Cache-Control', 'public, max-age=2592000')  // 30 jours
      response.headers.set('X-Image-Optimized', 'true')

      event.waitUntil(cache.put(cacheKey, response.clone()))
    }
  } else {
    response = new Response(response.body, response)
    response.headers.set('X-Cache', 'HIT')
  }

  return response
}
```

### 2. Predictive Prefetching

```javascript
// Service Worker avec prefetching prédictif

const PREFETCH_CACHE = 'prefetch-v1'
const NAVIGATION_CACHE = 'navigation-v1'

// Analyser les liens visibles et prefetch
function setupPrefetchObserver() {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const link = entry.target
        const href = link.href

        // Prefetch si lien interne
        if (href && href.startsWith(location.origin)) {
          prefetchURL(href)
        }
      }
    })
  }, {
    rootMargin: '50px'  // Prefetch 50px avant que visible
  })

  // Observer tous les liens
  document.querySelectorAll('a[href]').forEach(link => {
    observer.observe(link)
  })
}

async function prefetchURL(url) {
  const cache = await caches.open(PREFETCH_CACHE)

  // Vérifier si déjà en cache
  const cached = await cache.match(url)
  if (cached) return

  // Fetch avec priorité basse
  try {
    const response = await fetch(url, {
      priority: 'low',
      credentials: 'same-origin'
    })

    if (response.ok) {
      await cache.put(url, response)
      console.log('Prefetched:', url)
    }
  } catch (e) {
    console.error('Prefetch failed:', url, e)
  }
}

// Service Worker fetch handler
self.addEventListener('fetch', event => {
  event.respondWith(async function() {
    // Vérifier prefetch cache d'abord
    const prefetchCache = await caches.open(PREFETCH_CACHE)
    let response = await prefetchCache.match(event.request)

    if (response) {
      console.log('Served from prefetch cache:', event.request.url)
      return response
    }

    // Sinon, cache normal ou réseau
    const navCache = await caches.open(NAVIGATION_CACHE)
    response = await navCache.match(event.request)

    if (response) {
      return response
    }

    // Fetch from network
    response = await fetch(event.request)

    // Cache pour futures utilisations
    if (response.ok) {
      navCache.put(event.request, response.clone())
    }

    return response
  }())
})

// Initialiser au chargement
if (typeof window !== 'undefined') {
  window.addEventListener('load', setupPrefetchObserver)
}
```

### 3. Edge-Side Includes (ESI)

```html
<!-- page.html avec ESI tags -->
<!DOCTYPE html>
<html>
<head>
    <title>My Page</title>
</head>
<body>
    <!-- Header (cache long) -->
    <esi:include src="/fragments/header" ttl="3600"/>

    <!-- Contenu principal (cache moyen) -->
    <div class="main-content">
        <esi:include src="/fragments/main-content" ttl="300"/>
    </div>

    <!-- User-specific content (pas de cache edge) -->
    <div class="user-section">
        <esi:include
            src="/fragments/user-info"
            onerror="continue"
            alt="/fragments/user-info-fallback"/>
    </div>

    <!-- Footer (cache très long) -->
    <esi:include src="/fragments/footer" ttl="86400"/>
</body>
</html>
```

```python
# Backend pour servir les fragments

from flask import Flask, request, make_response

app = Flask(__name__)

@app.route('/fragments/header')
def header_fragment():
    """Fragment header (cache 1h)"""
    html = '''
    <header>
        <nav>
            <a href="/">Home</a>
            <a href="/products">Products</a>
        </nav>
    </header>
    '''

    response = make_response(html)
    response.headers['Cache-Control'] = 'public, max-age=3600'
    response.headers['Surrogate-Control'] = 'max-age=3600'

    return response

@app.route('/fragments/main-content')
def main_content_fragment():
    """Fragment contenu principal (cache 5min)"""
    content = get_latest_content()  # Depuis DB

    response = make_response(render_template('content_fragment.html', content=content))
    response.headers['Cache-Control'] = 'public, max-age=300'
    response.headers['Surrogate-Control'] = 'max-age=300'

    return response

@app.route('/fragments/user-info')
def user_info_fragment():
    """Fragment user-specific (pas de cache edge)"""
    user_id = request.headers.get('X-User-ID')

    if not user_id:
        return '<div>Please log in</div>', 200

    user = get_user(user_id)

    response = make_response(render_template('user_fragment.html', user=user))
    response.headers['Cache-Control'] = 'private, max-age=0'  # Pas de cache edge

    return response
```

## Monitoring et Analytics CDN

### Métriques essentielles

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Requêtes par statut
cdn_requests_total = Counter(
    'cdn_requests_total',
    'Total CDN requests',
    ['pop', 'status', 'cache_status']
)

# Latence edge
cdn_latency = Histogram(
    'cdn_latency_seconds',
    'CDN edge latency',
    ['pop'],
    buckets=[0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5]
)

# Bande passante
cdn_bandwidth = Counter(
    'cdn_bandwidth_bytes_total',
    'Total bandwidth served',
    ['pop', 'content_type']
)

# Cache hit ratio
cdn_cache_ratio = Gauge(
    'cdn_cache_hit_ratio',
    'Cache hit ratio',
    ['pop']
)

# Requêtes origin
cdn_origin_requests = Counter(
    'cdn_origin_requests_total',
    'Requests to origin',
    ['pop', 'status']
)

# Coûts estimés
cdn_cost_estimate = Counter(
    'cdn_cost_usd_total',
    'Estimated CDN cost',
    ['pop', 'tier']
)

class CDNAnalytics:
    """Analyse des logs CDN"""

    def __init__(self):
        self.cloudflare_email = 'your-email'
        self.cloudflare_key = 'your-api-key'
        self.zone_id = 'zone-id'

    def get_analytics(self, since_minutes=60):
        """
        Récupérer les analytics Cloudflare
        """
        import requests
        from datetime import datetime, timedelta

        since = datetime.utcnow() - timedelta(minutes=since_minutes)

        endpoint = f'https://api.cloudflare.com/client/v4/zones/{self.zone_id}/analytics/dashboard'

        params = {
            'since': since.isoformat() + 'Z',
            'continuous': 'true'
        }

        headers = {
            'X-Auth-Email': self.cloudflare_email,
            'X-Auth-Key': self.cloudflare_key
        }

        response = requests.get(endpoint, headers=headers, params=params)
        data = response.json()

        return data['result']

    def calculate_hit_ratio(self, analytics):
        """Calculer le cache hit ratio"""
        requests_total = analytics['totals']['requests']['all']
        requests_cached = analytics['totals']['requests']['cached']

        if requests_total == 0:
            return 0

        return (requests_cached / requests_total) * 100

    def calculate_bandwidth_saved(self, analytics):
        """Calculer la bande passante économisée"""
        bandwidth_total = analytics['totals']['bandwidth']['all']
        bandwidth_cached = analytics['totals']['bandwidth']['cached']

        # Pourcentage économisé
        if bandwidth_total == 0:
            return 0

        saved_percent = (bandwidth_cached / bandwidth_total) * 100

        # GB économisés
        saved_gb = bandwidth_cached / (1024**3)

        return {
            'percent': saved_percent,
            'gigabytes': saved_gb,
            'cost_saved': saved_gb * 0.08  # ~$0.08/GB pour bande passante
        }

    def get_top_urls(self, limit=10):
        """Top URLs les plus demandées"""
        # Via GraphQL Analytics API
        query = '''
        query {
          viewer {
            zones(filter: {zoneTag: "%s"}) {
              httpRequests1dGroups(
                limit: %d
                filter: {datetime_gt: "%s"}
                orderBy: [sum_requests_DESC]
              ) {
                dimensions {
                  clientRequestPath
                }
                sum {
                  requests
                  bytes
                }
              }
            }
          }
        }
        ''' % (self.zone_id, limit, since.isoformat() + 'Z')

        # ... requête GraphQL

    def export_to_prometheus(self):
        """Exporter les métriques vers Prometheus"""
        analytics = self.get_analytics(since_minutes=5)

        # Hit ratio
        hit_ratio = self.calculate_hit_ratio(analytics)
        cdn_cache_ratio.labels(pop='global').set(hit_ratio)

        # Requêtes
        cdn_requests_total.labels(
            pop='global',
            status='200',
            cache_status='hit'
        ).inc(analytics['totals']['requests']['cached'])

        cdn_requests_total.labels(
            pop='global',
            status='200',
            cache_status='miss'
        ).inc(analytics['totals']['requests']['uncached'])

        # Bande passante
        bandwidth_saved = self.calculate_bandwidth_saved(analytics)
        cdn_cost_estimate.labels(
            pop='global',
            tier='bandwidth'
        ).inc(bandwidth_saved['cost_saved'])

# Exécuter périodiquement
import schedule

analytics = CDNAnalytics()

schedule.every(5).minutes.do(analytics.export_to_prometheus)

while True:
    schedule.run_pending()
    time.sleep(60)
```

### Dashboard Grafana

```yaml
# Grafana dashboard queries

# Cache Hit Ratio
cdn_cache_hit_ratio

# Request rate par PoP
rate(cdn_requests_total[5m])

# Bandwidth économisée
rate(cdn_bandwidth_bytes_total{cache_status="hit"}[5m])

# Top PoPs par trafic
topk(10, sum(rate(cdn_requests_total[5m])) by (pop))

# Latence P95 par PoP
histogram_quantile(0.95,
  sum(rate(cdn_latency_seconds_bucket[5m])) by (le, pop)
)

# Coût estimé par jour
sum(increase(cdn_cost_usd_total[24h]))

# Origin load (devrait être minimal)
rate(cdn_origin_requests_total[5m])
```

## Coûts et optimisation

### Calculateur de coûts CDN

```python
class CDNCostCalculator:
    """
    Calculer les coûts CDN selon l'usage
    """

    # Prix indicatifs (varie selon le provider et le volume)
    PRICING = {
        'cloudflare': {
            'free': {
                'requests': float('inf'),  # Illimité
                'bandwidth': float('inf'),
                'cost_per_gb': 0
            },
            'pro': {
                'monthly': 20,
                'requests': float('inf'),
                'bandwidth': float('inf'),
                'cost_per_gb': 0
            }
        },
        'cloudfront': {
            'us': {
                'requests_per_10k': 0.0075,
                'cost_per_gb': {
                    '0-10TB': 0.085,
                    '10-50TB': 0.080,
                    '50-150TB': 0.060,
                    '150-500TB': 0.040,
                    '500TB+': 0.030
                }
            },
            'europe': {
                'requests_per_10k': 0.0090,
                'cost_per_gb': {
                    '0-10TB': 0.085,
                    '10-50TB': 0.080,
                    '50-150TB': 0.060,
                    '150-500TB': 0.040,
                    '500TB+': 0.030
                }
            }
        },
        'fastly': {
            'requests_per_10k': 0.0075,
            'cost_per_gb': 0.12
        }
    }

    def calculate_cloudfront_cost(self, bandwidth_gb, requests, region='us'):
        """Calculer le coût AWS CloudFront"""
        pricing = self.PRICING['cloudfront'][region]

        # Coût bandwidth (tiered)
        bandwidth_tb = bandwidth_gb / 1024
        bandwidth_cost = 0

        tiers = [
            (10, 0.085),
            (40, 0.080),   # 10-50
            (100, 0.060),  # 50-150
            (350, 0.040),  # 150-500
            (float('inf'), 0.030)  # 500+
        ]

        remaining = bandwidth_tb
        for tier_limit, price_per_gb in tiers:
            if remaining <= 0:
                break

            tier_usage = min(remaining, tier_limit)
            tier_cost = tier_usage * 1024 * price_per_gb
            bandwidth_cost += tier_cost
            remaining -= tier_usage

        # Coût requêtes
        requests_cost = (requests / 10000) * pricing['requests_per_10k']

        total = bandwidth_cost + requests_cost

        return {
            'bandwidth_cost': bandwidth_cost,
            'requests_cost': requests_cost,
            'total': total,
            'breakdown': {
                'bandwidth_gb': bandwidth_gb,
                'requests': requests,
                'cost_per_gb_avg': bandwidth_cost / bandwidth_gb if bandwidth_gb > 0 else 0
            }
        }

    def calculate_savings_with_cdn(self, monthly_visitors, avg_page_size_mb,
                                   cache_hit_ratio=0.90):
        """
        Calculer les économies avec un CDN
        """
        # Sans CDN : tout le trafic va à l'origin
        pages_per_visitor = 5  # Moyenne
        total_requests = monthly_visitors * pages_per_visitor

        # Bandwidth total
        total_bandwidth_gb = (total_requests * avg_page_size_mb) / 1024

        # Coût origin bandwidth (AWS: ~$0.09/GB)
        cost_without_cdn = total_bandwidth_gb * 0.09

        # Avec CDN
        # Origin voit seulement (1 - cache_hit_ratio) du trafic
        origin_bandwidth_gb = total_bandwidth_gb * (1 - cache_hit_ratio)
        origin_cost = origin_bandwidth_gb * 0.09

        # Coût CDN
        cdn_cost = self.calculate_cloudfront_cost(
            total_bandwidth_gb,
            total_requests
        )['total']

        cost_with_cdn = origin_cost + cdn_cost

        savings = cost_without_cdn - cost_with_cdn
        savings_percent = (savings / cost_without_cdn) * 100 if cost_without_cdn > 0 else 0

        return {
            'without_cdn': {
                'bandwidth_gb': total_bandwidth_gb,
                'cost': cost_without_cdn
            },
            'with_cdn': {
                'total_bandwidth_gb': total_bandwidth_gb,
                'origin_bandwidth_gb': origin_bandwidth_gb,
                'cache_hit_ratio': cache_hit_ratio,
                'origin_cost': origin_cost,
                'cdn_cost': cdn_cost,
                'total_cost': cost_with_cdn
            },
            'savings': {
                'amount': savings,
                'percent': savings_percent
            }
        }

# Usage
calculator = CDNCostCalculator()

# Exemple : Site avec 1M visiteurs/mois
result = calculator.calculate_savings_with_cdn(
    monthly_visitors=1000000,
    avg_page_size_mb=2.5,
    cache_hit_ratio=0.92
)

print(f"Sans CDN : ${result['without_cdn']['cost']:.2f}/mois")
print(f"Avec CDN : ${result['with_cdn']['total_cost']:.2f}/mois")
print(f"Économies : ${result['savings']['amount']:.2f}/mois ({result['savings']['percent']:.1f}%)")

# Output (exemple) :
# Sans CDN : $1093.75/mois
# Avec CDN : $312.45/mois
# Économies : $781.30/mois (71.4%)
```

## Pièges courants et solutions

### Piège 1 : Cache de contenu dynamique

**Problème :** Cache de pages personnalisées servant le mauvais contenu.

**Solution :**

```nginx
# Nginx - Bypass cache pour utilisateurs authentifiés

map $http_cookie $bypass_cache {
    default 0;
    "~*session" 1;
    "~*logged_in" 1;
}

server {
    location / {
        proxy_cache my_cache;

        # Bypass si cookie de session
        proxy_cache_bypass $bypass_cache;
        proxy_no_cache $bypass_cache;

        # Vary header
        add_header Vary "Cookie";

        proxy_pass http://backend;
    }
}
```

### Piège 2 : Cache poisoning via query params

**Problème :** Attaquant injecte des query params pour créer des caches multiples.

```
GET /page?x=1
GET /page?x=2
GET /page?x=3
... infinité de variants cachés
```

**Solution :**

```javascript
// Cloudflare Worker - Normaliser les query params

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)

  // Whitelist de query params autorisés
  const allowedParams = ['page', 'sort', 'category']

  // Filtrer les params
  const cleanParams = new URLSearchParams()
  for (const [key, value] of url.searchParams) {
    if (allowedParams.includes(key)) {
      cleanParams.set(key, value)
    }
  }

  // Trier les params (cache key consistant)
  cleanParams.sort()

  // Reconstruire l'URL propre
  url.search = cleanParams.toString()

  const cleanRequest = new Request(url.toString(), request)

  return fetch(cleanRequest)
}
```

### Piège 3 : TTL trop long sur contenu changeant

**Problème :** Contenu obsolète servi pendant des heures.

**Solution :** Utiliser la purge programmatique + stale-while-revalidate.

```python
# Backend avec auto-purge

from flask import Flask, jsonify
import requests

app = Flask(__name__)

@app.route('/api/products/<int:product_id>', methods=['PUT'])
def update_product(product_id):
    """Mettre à jour et purger automatiquement"""

    # Mise à jour DB
    product = update_product_in_db(product_id, request.json)

    # Purge CDN immédiatement
    purge_cdn([
        f'https://www.example.com/products/{product_id}',
        f'https://api.example.com/products/{product_id}'
    ])

    return jsonify(product)

def purge_cdn(urls):
    """Helper pour purger Cloudflare"""
    requests.post(
        f'https://api.cloudflare.com/client/v4/zones/{ZONE_ID}/purge_cache',
        headers={'Authorization': f'Bearer {CF_TOKEN}'},
        json={'files': urls}
    )
```

### Piège 4 : Pas de compression

**Problème :** Fichiers non compressés consomment plus de bande passante.

**Solution :**

```nginx
# Nginx avec compression Brotli et Gzip

http {
    # Gzip
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    # Brotli (si module installé)
    brotli on;
    brotli_comp_level 6;
    brotli_types
        text/plain
        text/css
        text/javascript
        application/json
        application/javascript;
}
```

### Piège 5 : Ignorer les headers Vary

**Problème :** Cache qui ne tient pas compte des variants.

**Solution :**

```python
# Backend qui utilise correctement Vary

@app.route('/api/data')
def get_data():
    # Contenu varie selon Accept-Language
    lang = request.headers.get('Accept-Language', 'en')[:2]

    data = get_localized_data(lang)

    response = make_response(jsonify(data))

    # IMPORTANT : Indiquer que le contenu varie selon la langue
    response.headers['Vary'] = 'Accept-Language'
    response.headers['Cache-Control'] = 'public, max-age=3600'

    return response
```

## Comparaison des solutions CDN

| Provider | Prix | Edge Locations | Fonctionnalités | Meilleur pour |
|----------|------|----------------|-----------------|---------------|
| **Cloudflare** | Gratuit à $200+/mois | 300+ | Workers, Images, Stream | Sites web, APIs |
| **AWS CloudFront** | Pay-as-you-go | 450+ | Lambda@Edge, intégration AWS | AWS users |
| **Fastly** | Pay-as-you-go | 70+ | Edge compute, instant purge | High-perf, real-time |
| **Akamai** | Enterprise | 4000+ | Advanced DDoS, media | Enterprise, streaming |
| **Cloudinary** | Gratuit à $1000+/mois | 30+ | Image/video optimization | Media-heavy sites |
| **BunnyCDN** | $0.01/GB | 100+ | Bon rapport qualité/prix | Budget-conscious |

## Résumé : Quand utiliser un CDN

### ✅ Utilisez un CDN si :

- Audience **internationale** ou multi-régionale
- Contenu **statique important** (images, CSS, JS)
- Besoin de **performance** (latence < 100ms)
- Trafic **élevé** (économies de bande passante)
- Besoin de **protection DDoS**
- **Streaming** vidéo/audio
- **Haute disponibilité** critique

### ❌ CDN peut-être superflu si :

- Audience uniquement **locale** (même ville)
- Site **très dynamique** (personnalisé par user)
- Très **faible trafic** (< 1000 visites/mois)
- Budget **extrêmement limité** (mais CDN gratuits existent)

### Checklist de décision

| Critère | Sans CDN | Avec CDN |
|---------|----------|----------|
| **Latence globale** | 200-500ms | 20-100ms |
| **Bande passante origin** | 100% | 5-20% |
| **Coût bande passante** | $$$ | $ |
| **Disponibilité** | 99.9% | 99.99%+ |
| **Protection DDoS** | Limitée | Excellente |
| **Complexité** | Faible | Moyenne |

## Conclusion

Les CDN sont **essentiels** pour toute application web moderne visant une audience globale. Ils offrent :

**Performance :**
- Latence réduite de 80-90%
- Temps de chargement divisé par 3-10x

**Économies :**
- 70-95% de réduction de bande passante origin
- Économies de $500-10000+/mois selon l'échelle

**Fiabilité :**
- Haute disponibilité (99.99%+)
- Protection DDoS automatique
- Failover automatique

**À retenir :**
- Configurer des **TTL appropriés** selon le contenu
- Implémenter la **purge automatique** lors des modifications
- Utiliser les **cache keys** intelligemment
- **Monitorer** le hit ratio (objectif : >85%)
- Optimiser les **images** à la volée
- Considérer l'**edge computing** pour logique avancée

**Prochaine étape :** La section suivante explore les **Architectures Haute Disponibilité**, qui combinent CDN, load balancing et redondance pour créer des systèmes ultra-fiables.

---


⏭️ [Architectures haute disponibilité](/09-architectures-avancees/06-haute-disponibilite.md)
