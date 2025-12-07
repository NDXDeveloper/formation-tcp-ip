🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.5 - Edge computing et implications réseau

## Introduction

Le cloud computing a centralisé le calcul dans quelques datacenters géants. L'edge computing inverse cette tendance : il rapproche le traitement des données au plus près de leur source et des utilisateurs finaux.

Pour les développeurs, l'edge n'est pas qu'une optimisation d'infrastructure : c'est un **nouveau modèle de programmation** qui change radicalement la façon de concevoir les applications distribuées, gérer la latence, assurer la cohérence des données, et optimiser les coûts réseau.

## Qu'est-ce que l'edge computing ?

### Définition

L'edge computing déplace le calcul et le stockage **du datacenter centralisé vers le bord du réseau**, aussi proche que possible de l'utilisateur ou de l'appareil générant/consommant les données.

```
Cloud Computing (centralisé) :
User → [Internet 50-200ms] → Datacenter → Processing → Response

Edge Computing (distribué) :
User → [Local network 1-10ms] → Edge node → Processing → Response
                                      ↓
                          [Sync async vers cloud central]
```

### Continuum de calcul

```
                    Latence    Compute     Storage    Use case
                    ────────────────────────────────────────────────
Device/Endpoint     0ms        Minimal     Minimal    Sensors, IoT
    ↕ <1ms                     (mW-W)      (KB-MB)

Far Edge           1-5ms      Limited     Limited    Local processing
    ↕ 5-10ms                   (W-kW)      (GB-TB)    5G base stations

Near Edge          10-20ms    Moderate    Moderate   City-level apps
    ↕ 20-50ms                  (kW)        (TB)       CDN, edge DC

Regional Cloud     50-100ms   High        High       Regional apps
    ↕ 100-200ms                (MW)        (PB)       AWS regions

Central Cloud      100-300ms  Unlimited   Unlimited  Batch processing
                               (MW)        (EB)       ML training
```

### Cloud vs Edge vs Fog

```
┌─────────────────────────────────────────────────────────┐
│                    CENTRAL CLOUD                        │
│              (AWS, Azure, GCP)                          │
│   - Compute illimité                                    │
│   - Latence 50-300ms                                    │
│   - Big Data, ML training, Analytics                    │
└────────────────────┬────────────────────────────────────┘
                     │ WAN
                     ↓
┌─────────────────────────────────────────────────────────┐
│                    EDGE CLOUD                           │
│         (Cloudflare, Fastly, AWS Local Zones)           │
│   - Compute modéré                                      │
│   - Latence 10-50ms                                     │
│   - API serving, CDN, video streaming                   │
└────────────────────┬────────────────────────────────────┘
                     │ Metro
                     ↓
┌─────────────────────────────────────────────────────────┐
│                    FOG LAYER                            │
│              (Base stations 5G, gateways)               │
│   - Compute limité                                      │
│   - Latence 5-20ms                                      │
│   - IoT aggregation, local ML inference                 │
└────────────────────┬────────────────────────────────────┘
                     │ Local
                     ↓
┌─────────────────────────────────────────────────────────┐
│                    DEVICES                              │
│           (Smartphones, IoT, sensors)                   │
│   - Compute minimal                                     │
│   - Latence 0-5ms                                       │
│   - Data collection, basic processing                   │
└─────────────────────────────────────────────────────────┘
```

## Pourquoi l'edge computing ?

### 1. Réduction de latence

**Impact de la latence sur l'expérience** :

```
Temps de réponse          Impact utilisateur
────────────────────────────────────────────
0-100ms                   Instantané
100-300ms                 Perceptible
300-1000ms                Lent
1000ms+                   Frustrant

Études :
- Google : +500ms latence = -20% trafic
- Amazon : +100ms latence = -1% ventes
- Microsoft : +10ms latence = -0.6% revenus (Bing)
```

**Exemple concret : application de trading** :

```python
# Trading app : chaque milliseconde compte

# Architecture cloud centralisée (Paris → AWS us-east-1)
def place_order_cloud(order):
    # Réseau : Paris → Virginie
    latency_network = 80ms  # Best case

    # Processing cloud
    latency_processing = 10ms

    # Total : 90ms
    # Sur marchés financiers : 90ms = désavantage compétitif énorme

# Architecture edge (Paris → AWS eu-west-3 Paris)
def place_order_edge(order):
    # Réseau : Paris → Paris edge
    latency_network = 2ms

    # Processing edge
    latency_processing = 10ms

    # Total : 12ms
    # Gain : 78ms = 650% plus rapide

    # Sync async vers central pour compliance
    async_sync_to_central(order)
```

### 2. Réduction de la bande passante

**Problème : IoT et volumes de données** :

```
Scénario : 1000 caméras surveillance, 1080p, 30fps

Sans edge :
- Chaque caméra : 5 Mbps
- Total upload : 5 Gbps constant
- Coût bande passante : $50,000/mois
- Stockage cloud : 15 PB/mois
- Coût stockage : $300,000/mois

Avec edge :
- Processing local (détection mouvement, reconnaissance)
- Upload seulement événements : 50 Mbps moyen
- Coût bande passante : $500/mois (99% réduction)
- Stockage cloud : 150 TB/mois (alertes only)
- Coût stockage : $3,000/mois (99% réduction)

ROI : Économie de $346,500/mois
```

**Code : edge processing pour caméra** :

```python
# Edge node pour caméra de surveillance
import cv2
import numpy as np
from edge_ml import ObjectDetector

class EdgeCameraProcessor:
    def __init__(self):
        self.detector = ObjectDetector('yolo-tiny')  # Lightweight model
        self.baseline = None
        self.alert_threshold = 0.3

    def process_frame(self, frame):
        """
        Processing local sur edge node
        """
        # 1. Détection de mouvement (très léger)
        motion_detected = self.detect_motion(frame)

        if not motion_detected:
            # Pas de mouvement : ne rien envoyer au cloud
            return None

        # 2. Motion détecté : analyse plus poussée
        objects = self.detector.detect(frame)

        # 3. Filtrer objets pertinents
        relevant_objects = [
            obj for obj in objects
            if obj.class_name in ['person', 'vehicle', 'package']
            and obj.confidence > 0.7
        ]

        if not relevant_objects:
            # Fausse alerte : ne rien envoyer
            return None

        # 4. Objet pertinent détecté : créer alerte
        alert = {
            'timestamp': time.time(),
            'camera_id': self.camera_id,
            'objects': relevant_objects,
            'thumbnail': self.compress_image(frame),  # Thumbnail only, pas full frame
            'metadata': self.extract_metadata(frame)
        }

        # 5. Envoyer au cloud (seulement alertes)
        self.send_to_cloud(alert)  # Quelques KB au lieu de MB

        # 6. Stocker localement pour forensics court-terme
        self.local_storage.append(frame, ttl='24h')

        return alert

    def detect_motion(self, frame):
        """
        Détection de mouvement simple (très rapide)
        """
        if self.baseline is None:
            self.baseline = frame
            return False

        # Différence avec baseline
        diff = cv2.absdiff(self.baseline, frame)
        motion_score = np.mean(diff)

        # Update baseline progressivement
        self.baseline = cv2.addWeighted(self.baseline, 0.95, frame, 0.05, 0)

        return motion_score > self.alert_threshold

# Résultat :
# - 99% des frames traitées localement et jetées
# - 1% envoyé au cloud (alertes réelles)
# - Bande passante divisée par 100
```

### 3. Disponibilité et résilience

```javascript
// Problème : dépendance au cloud central

// Sans edge : Single Point of Failure
async function getUserData(userId) {
    // Si connexion cloud perdue → app inutilisable
    try {
        const response = await fetch(`https://api.cloud.com/users/${userId}`);
        return response.json();
    } catch (error) {
        // Pas de connexion = pas de service
        return { error: "Service unavailable" };
    }
}

// Avec edge : Dégradation gracieuse
class EdgeFirstAPI {
    constructor() {
        this.edgeCache = new EdgeKV();  // Cache local edge
        this.cloudAPI = 'https://api.cloud.com';
    }

    async getUserData(userId) {
        // 1. Essayer edge cache d'abord
        const cached = await this.edgeCache.get(`user:${userId}`);
        if (cached && !this.isStale(cached)) {
            return cached.data;
        }

        // 2. Essayer cloud
        try {
            const response = await fetch(
                `${this.cloudAPI}/users/${userId}`,
                { timeout: 1000 }  // Fail fast
            );
            const data = await response.json();

            // 3. Mettre en cache à l'edge
            await this.edgeCache.put(`user:${userId}`, {
                data: data,
                timestamp: Date.now(),
                ttl: 3600
            });

            return data;

        } catch (cloudError) {
            // 4. Cloud indisponible : servir cache stale si disponible
            if (cached) {
                console.warn('Serving stale data due to cloud unavailability');
                return {
                    ...cached.data,
                    _stale: true,
                    _cached_at: cached.timestamp
                };
            }

            // 5. Vraiment aucune donnée disponible
            throw new Error('No data available');
        }
    }
}

// Avantage : app fonctionne même si cloud down
// Trade-off : possibilité de données légèrement périmées
```

### 4. Conformité et souveraineté des données

```
Exemple : RGPD Europe

Problème :
- Données personnelles d'européens
- Doivent rester en Europe (RGPD Art. 44-50)
- Cloud centralisé US = violation potentielle

Solution edge :
┌──────────────────────┐
│   Utilisateur FR     │
└─────────┬────────────┘
          │
          ▼
┌──────────────────────┐
│  Edge Node Paris     │ ← Données restent en EU
│  (Cloudflare/AWS)    │   Compliant RGPD
└─────────┬────────────┘
          │
          ▼ (Anonymisé)
┌──────────────────────┐
│  Central Cloud US    │ ← Seulement données agrégées,
│  (Analytics)         │   anonymisées
└──────────────────────┘

Code :
```

```typescript
// Edge function (Cloudflare Workers, EU nodes)
export default {
    async fetch(request: Request, env: Env): Promise<Response> {
        const userIP = request.headers.get('CF-Connecting-IP');
        const country = request.cf?.country;

        // Traiter requête localement si utilisateur EU
        if (isEU(country)) {
            // Données restent à l'edge EU
            const result = await processLocally(request, env.EU_KV);

            // Envoyer seulement métriques anonymisées au central
            await sendAnonymizedMetrics({
                country: country,
                timestamp: Date.now(),
                endpoint: new URL(request.url).pathname,
                // PAS d'IP, PAS d'identifiants personnels
            });

            return result;
        }

        // Utilisateurs hors-EU : peut utiliser cloud central
        return forwardToCloud(request);
    }
};

function isEU(country: string): boolean {
    const euCountries = ['FR', 'DE', 'IT', 'ES', 'NL', ...];
    return euCountries.includes(country);
}
```

## Architectures edge pour développeurs

### 1. Edge Functions / Serverless Edge

**Plateformes principales** :

```
Plateforme          Runtime           Deploy      Limitations
────────────────────────────────────────────────────────────────
Cloudflare Workers  V8 isolates       Global      CPU: 50ms
                    JavaScript/WASM   200+ DC     Memory: 128MB
                                                  Démarrage: ~0ms

Fastly Compute      WebAssembly       Global      CPU: 50ms (init)
                    Rust/JS/etc.      60+ POP     Memory: variable
                                                  Démarrage: ~0ms

AWS Lambda@Edge     Node.js/Python    Regional    CPU: 5-30s
                                      CloudFront  Memory: 128MB-10GB
                                      locations   Démarrage: cold

Vercel Edge         V8 isolates       Global      CPU: wall time
Functions           JavaScript/TS     100+ DC     limité

Deno Deploy         Deno runtime      Global      CPU: 50ms
                    TypeScript/JS     35+ regions Memory: 512MB
```

**Exemple Cloudflare Workers** :

```javascript
// worker.js - Edge function globale
export default {
    async fetch(request, env, ctx) {
        const url = new URL(request.url);

        // Exécuté dans 200+ datacenters mondialement
        // Utilisateur routé vers le plus proche automatiquement

        // Cas d'usage : A/B testing à l'edge
        const variant = getABTestVariant(request);

        if (variant === 'B') {
            // Servir version B depuis edge
            return new Response(renderVariantB(), {
                headers: {
                    'Content-Type': 'text/html',
                    'X-Variant': 'B',
                    'Cache-Control': 'public, max-age=3600'
                }
            });
        }

        // Variante A : fetch depuis origin
        return fetch(request);
    }
};

function getABTestVariant(request) {
    // Consistent hashing basé sur cookie/IP
    const userId = request.headers.get('Cookie')?.match(/user_id=([^;]+)/)?.[1];

    if (!userId) {
        return 'A';  // Default
    }

    // Hash deterministe
    const hash = simpleHash(userId);
    return (hash % 100) < 50 ? 'A' : 'B';  // 50/50 split
}

function renderVariantB() {
    return `
        <!DOCTYPE html>
        <html>
        <head><title>Variant B</title></head>
        <body>
            <h1>New Feature!</h1>
            <p>You're seeing variant B</p>
        </body>
        </html>
    `;
}
```

**Performance** :

```
User request (New York) → Cloudflare edge NYC (2ms) → Response
Total time: 2-5ms

vs

User request (New York) → AWS us-east-1 (Virginia, 15ms) → Response
Total time: 15-30ms

Gain : 5-10x plus rapide
```

### 2. Edge Storage / KV Stores

**Problématique** : Comment stocker des données à l'edge ?

```
Traditional DB :
Edge function → [WAN 50ms] → Database central → Response
Total : 50-100ms (database devient bottleneck)

Edge KV :
Edge function → Edge KV store local (0.5ms) → Response
Total : 0.5-2ms (10-100x plus rapide)
```

**Cloudflare Workers KV** :

```javascript
// Bind KV namespace dans wrangler.toml
// [[kv_namespaces]]
// binding = "CACHE"
// id = "xxxxx"

export default {
    async fetch(request, env) {
        const cacheKey = new URL(request.url).pathname;

        // 1. Lecture depuis KV edge (très rapide)
        const cached = await env.CACHE.get(cacheKey, { type: 'json' });

        if (cached) {
            return new Response(JSON.stringify(cached), {
                headers: {
                    'Content-Type': 'application/json',
                    'X-Cache': 'HIT'
                }
            });
        }

        // 2. Cache miss : fetch depuis origin
        const data = await fetchFromOrigin(request);

        // 3. Stocker à l'edge pour prochaines requêtes
        // Propagation globale : 10-60 secondes
        await env.CACHE.put(
            cacheKey,
            JSON.stringify(data),
            { expirationTtl: 3600 }  // 1 heure
        );

        return new Response(JSON.stringify(data), {
            headers: {
                'Content-Type': 'application/json',
                'X-Cache': 'MISS'
            }
        });
    }
};
```

**Caractéristiques KV edge** :

```
Cloudflare KV :
- Lecture : <1ms (local edge)
- Écriture : 10-60s (propagation globale)
- Eventual consistency
- Use case : Données read-heavy, update rare
  (config, assets, sessions, cache)

Durable Objects (Cloudflare) :
- Lecture/Écriture : <1ms
- Strong consistency (single location)
- Stateful (WebSocket, coordonnées)
- Use case : Collaborative apps, gaming, chat

Vercel Edge Config :
- Lecture : <1ms
- Écriture : ~1s propagation
- Eventual consistency
- Use case : Feature flags, redirects
```

### 3. Edge Databases

**Problématique traditionnelle** :

```
Application distribuée + Database centralisée = Latence

User Tokyo → Edge function Tokyo → DB Virginia → Response
             ↑ 5ms                  ↑ 150ms      ↑ Total: 155ms
             rapide                 lent          problème
```

**Solutions edge database** :

#### a) Distributed SQL avec réplication

```sql
-- CockroachDB / YugabyteDB : SQL distribué

-- Configuration multi-région
CREATE DATABASE ecommerce PRIMARY REGION "us-east1"
    REGIONS "eu-west1", "asia-southeast1";

-- Table avec locality
CREATE TABLE products (
    id UUID PRIMARY KEY,
    name TEXT,
    price DECIMAL,
    region TEXT
) LOCALITY REGIONAL BY ROW;

-- Lecture locale (rapide), écriture synchronisée (lent)
INSERT INTO products VALUES
    (uuid_generate_v4(), 'Product A', 99.99, 'us-east1');

-- Requête exécutée sur nœud local de l'utilisateur
SELECT * FROM products WHERE region = crdb_region();
-- Latence : 5-20ms au lieu de 150ms
```

**Code application** :

```go
// Application Go avec CockroachDB distribué
package main

import (
    "database/sql"
    _ "github.com/lib/pq"
)

type EdgeDB struct {
    db *sql.DB
    region string
}

func (e *EdgeDB) GetProducts() ([]Product, error) {
    // Requête optimisée pour région locale
    query := `
        SELECT id, name, price
        FROM products
        WHERE region = $1
        OR region = 'global'
        ORDER BY name
    `

    rows, err := e.db.Query(query, e.region)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    // Lecture depuis replica local : <10ms
    var products []Product
    for rows.Next() {
        var p Product
        rows.Scan(&p.ID, &p.Name, &p.Price)
        products = append(products, p)
    }

    return products, nil
}

func (e *EdgeDB) CreateOrder(order Order) error {
    // Écriture : synchronisation multi-région
    // Plus lent (50-200ms) mais cohérence garantie
    tx, _ := e.db.Begin()

    _, err := tx.Exec(`
        INSERT INTO orders (id, user_id, total, region)
        VALUES ($1, $2, $3, $4)
    `, order.ID, order.UserID, order.Total, e.region)

    if err != nil {
        tx.Rollback()
        return err
    }

    return tx.Commit()
}
```

#### b) Multi-master avec CRDT

```javascript
// Conflict-free Replicated Data Types (CRDT)
// Chaque edge peut écrire localement, sync async

import { Y } from 'yjs';  // CRDT library

class EdgeCollaborativeDoc {
    constructor(edgeId) {
        this.doc = new Y.Doc();
        this.edgeId = edgeId;

        // Shared state (CRDT)
        this.sharedMap = this.doc.getMap('data');
    }

    // Écriture locale instantanée
    set(key, value) {
        this.sharedMap.set(key, value);
        // Propagation asynchrone vers autres edges
        this.syncToOtherEdges();
    }

    get(key) {
        // Lecture locale instantanée
        return this.sharedMap.get(key);
    }

    async syncToOtherEdges() {
        // Encoder changements depuis dernière sync
        const stateVector = Y.encodeStateVector(this.doc);
        const diff = Y.encodeStateAsUpdate(this.doc, stateVector);

        // Envoyer à hub de sync (peut être async, best-effort)
        await fetch('https://sync-hub.com/sync', {
            method: 'POST',
            body: diff
        });
    }

    applyUpdate(update) {
        // Recevoir update d'autre edge
        Y.applyUpdate(this.doc, update);
        // CRDT garantit convergence sans conflits
    }
}

// Use case : collaborative editing, presence, cursors
const doc = new EdgeCollaborativeDoc('edge-paris-1');
doc.set('cursor:user123', { x: 100, y: 200 });  // Instantané local
// Sync vers autres edges en arrière-plan (eventual consistency)
```

#### c) Edge cache + central source of truth

```typescript
// Pattern hybride le plus courant
class EdgeCachedDB {
    constructor(
        private edgeCache: KVNamespace,
        private centralDB: string
    ) {}

    async get(key: string) {
        // 1. Try edge cache first (ultra-fast)
        const cached = await this.edgeCache.get(key, { type: 'json' });

        if (cached && !this.isStale(cached)) {
            return cached.value;
        }

        // 2. Cache miss or stale: fetch from central
        const response = await fetch(`${this.centralDB}/get/${key}`);
        const data = await response.json();

        // 3. Update edge cache
        await this.edgeCache.put(key, JSON.stringify({
            value: data,
            timestamp: Date.now(),
            ttl: 300  // 5 minutes
        }));

        return data;
    }

    async set(key: string, value: any) {
        // Write-through : écrire central d'abord
        await fetch(`${this.centralDB}/set/${key}`, {
            method: 'POST',
            body: JSON.stringify(value)
        });

        // Puis invalider cache edge (ou update)
        await this.edgeCache.delete(key);
        // Ou mettre à jour immédiatement
        // await this.edgeCache.put(key, JSON.stringify({...}));
    }

    isStale(cached: any): boolean {
        const age = Date.now() - cached.timestamp;
        return age > cached.ttl * 1000;
    }
}
```

### 4. Edge APIs et routing intelligent

```javascript
// Cloudflare Workers : routing intelligent à l'edge

export default {
    async fetch(request, env) {
        const url = new URL(request.url);
        const path = url.pathname;

        // Router à l'edge selon divers critères

        // 1. Routing géographique
        const country = request.cf?.country;
        if (country === 'CN') {
            // Chine : serveur spécifique (compliance)
            return fetch('https://api-china.example.com' + path);
        }

        // 2. Routing par user tier
        const userTier = await getUserTier(request);
        if (userTier === 'premium') {
            // Premium : serveurs dédiés haute-perf
            return fetch('https://premium-api.example.com' + path);
        }

        // 3. Canary deployment
        const canaryPercentage = 5;
        if (shouldRouteToCanary(request, canaryPercentage)) {
            return fetch('https://canary-api.example.com' + path);
        }

        // 4. Load balancing intelligent
        const origin = await selectOptimalOrigin(request);
        return fetch(`https://${origin}` + path);
    }
};

async function selectOptimalOrigin(request) {
    // Choisir origin basé sur :
    // - Latence mesurée
    // - Charge CPU
    // - Disponibilité

    const origins = [
        { url: 'api-us.example.com', region: 'us', load: 0.6, latency: 45 },
        { url: 'api-eu.example.com', region: 'eu', load: 0.3, latency: 15 },
        { url: 'api-asia.example.com', region: 'asia', load: 0.8, latency: 120 }
    ];

    const userRegion = request.cf?.continent;

    // Filtrer par région
    let candidates = origins.filter(o =>
        o.region === userRegion?.toLowerCase() && o.load < 0.9
    );

    if (candidates.length === 0) {
        candidates = origins.filter(o => o.load < 0.9);
    }

    // Sélectionner avec latence minimale
    candidates.sort((a, b) => a.latency - b.latency);

    return candidates[0].url;
}
```

## Cas d'usage réels

### Cas 1 : E-commerce avec personnalisation edge

**Contexte** : Site e-commerce global, 10M utilisateurs/mois, personnalisation importante.

**Problème** :
- Personnalisation nécessite profile utilisateur
- Fetch profile depuis DB centrale = 100-200ms
- Page load impactée, taux conversion réduit

**Solution edge** :

```javascript
// Cloudflare Worker pour e-commerce
export default {
    async fetch(request, env) {
        const userId = getUserIdFromCookie(request);

        // 1. Fetch user profile depuis edge KV (cached)
        let profile = await env.USER_PROFILES.get(userId, { type: 'json' });

        if (!profile) {
            // Cache miss : fetch depuis central DB
            profile = await fetchProfileFromDB(userId);

            // Cache à l'edge pour 1 heure
            await env.USER_PROFILES.put(
                userId,
                JSON.stringify(profile),
                { expirationTtl: 3600 }
            );
        }

        // 2. Personnaliser la page à l'edge
        const html = await generatePersonalizedHTML(profile, env);

        // 3. Retourner page personnalisée
        return new Response(html, {
            headers: {
                'Content-Type': 'text/html',
                'Cache-Control': 'private, no-cache',
                'X-Edge-Personalized': 'true'
            }
        });
    }
};

async function generatePersonalizedHTML(profile, env) {
    // Fetch template depuis edge cache
    const template = await env.TEMPLATES.get('homepage');

    // Fetch recommandations basées sur historique
    const recommendations = await getRecommendations(
        profile.purchase_history,
        profile.preferences
    );

    // Générer HTML avec SSR à l'edge
    return renderTemplate(template, {
        userName: profile.name,
        recommendations: recommendations,
        recentlyViewed: profile.recently_viewed,
        cart: profile.cart,
        loyaltyPoints: profile.loyalty_points
    });
}

async function getRecommendations(history, preferences) {
    // ML inference légère à l'edge
    // Ou fetch depuis cache de recommandations pré-calculées

    const cacheKey = `recs:${hashHistory(history)}`;

    let recs = await env.RECOMMENDATIONS.get(cacheKey, { type: 'json' });

    if (!recs) {
        // Calculer recommendations (ou fetch depuis ML service)
        recs = await calculateRecommendations(history, preferences);

        await env.RECOMMENDATIONS.put(
            cacheKey,
            JSON.stringify(recs),
            { expirationTtl: 1800 }  // 30 min
        );
    }

    return recs;
}
```

**Résultats** :

```
Avant (central rendering) :
- Time to First Byte : 450ms
- Largest Contentful Paint : 1.2s
- Conversion rate : 2.3%

Après (edge rendering) :
- Time to First Byte : 45ms (90% réduction)
- Largest Contentful Paint : 380ms (68% réduction)
- Conversion rate : 2.8% (+21% relative)

ROI :
- Revenue +21%
- Coût edge : $5k/mois
- Revenue gain : $50k/mois
- ROI : 900%
```

### Cas 2 : API Gateway avec rate limiting edge

**Contexte** : API publique avec millions de requêtes/jour, risque de DDoS et abuse.

**Solution traditionnelle (centrale)** :

```
Problème :
User → API Gateway central → Rate limiter → Backend
      ↑ 50ms                ↑ 10ms         ↑ 20ms

Total : 80ms par requête

En cas de DDoS :
- 1M requêtes/seconde touchent gateway central
- Coût bande passante énorme
- CPU gateway saturé
- Vraies requêtes impactées
```

**Solution edge** :

```javascript
// Rate limiting à l'edge (Cloudflare Durable Objects)

// Durable Object pour rate limiting
export class RateLimiter {
    constructor(state, env) {
        this.state = state;
        this.storage = state.storage;
    }

    async fetch(request) {
        const key = new URL(request.url).searchParams.get('key');

        // État persiste dans ce Durable Object
        const current = await this.storage.get(key) || {
            count: 0,
            resetAt: Date.now() + 60000  // 1 minute window
        };

        // Reset si window expiré
        if (Date.now() > current.resetAt) {
            current.count = 0;
            current.resetAt = Date.now() + 60000;
        }

        // Limite : 100 requêtes/minute
        if (current.count >= 100) {
            return new Response(JSON.stringify({
                limited: true,
                resetAt: current.resetAt
            }), { status: 429 });
        }

        // Incrementer
        current.count++;
        await this.storage.put(key, current);

        return new Response(JSON.stringify({
            limited: false,
            remaining: 100 - current.count
        }));
    }
}

// Worker principal
export default {
    async fetch(request, env) {
        // Identifier l'appelant
        const apiKey = request.headers.get('X-API-Key');

        if (!apiKey) {
            return new Response('Missing API key', { status: 401 });
        }

        // Vérifier rate limit à l'edge
        const rateLimiterId = env.RATE_LIMITER.idFromName(apiKey);
        const rateLimiter = env.RATE_LIMITER.get(rateLimiterId);

        const limitCheck = await rateLimiter.fetch(
            `https://rate-limit/?key=${apiKey}`
        );
        const result = await limitCheck.json();

        if (result.limited) {
            // Bloquer à l'edge, ne jamais toucher origin
            return new Response('Rate limit exceeded', {
                status: 429,
                headers: {
                    'X-RateLimit-Remaining': '0',
                    'X-RateLimit-Reset': String(result.resetAt)
                }
            });
        }

        // Rate limit OK : forward vers origin
        const response = await fetch(request);

        // Ajouter headers rate limit
        response.headers.set('X-RateLimit-Remaining', String(result.remaining));

        return response;
    }
};
```

**Avantages** :

```
Attaque DDoS :
- 10M requêtes/sec d'un attaquant
- Bloquées à l'edge (200+ datacenters)
- 0 requête atteint origin
- Coût : négligeable (edge absorbe)
- Vraies requêtes : non impactées

Performance :
- Rate limit check : <1ms (Durable Object local)
- vs 50ms si check central
- Latence totale réduite
```

### Cas 3 : Streaming vidéo avec edge transcoding

**Contexte** : Plateforme de streaming vidéo, contenu user-generated.

**Problème** :
```
Upload vidéo 4K :
User → Central datacenter → Transcode (CPU-heavy) → Store → Distribute

Temps : 10-30 minutes pour transcode complet
Coût : Transfert vers central + compute central
```

**Solution edge** :

```python
# Edge node avec GPU pour transcoding
import ffmpeg
from edge_storage import S3Edge

class EdgeVideoProcessor:
    def __init__(self):
        self.edge_storage = S3Edge()
        self.transcode_profiles = [
            {'resolution': '1080p', 'bitrate': '5M'},
            {'resolution': '720p', 'bitrate': '2.5M'},
            {'resolution': '480p', 'bitrate': '1M'},
            {'resolution': '360p', 'bitrate': '500k'}
        ]

    async def process_upload(self, video_file, user_id):
        """
        Traiter vidéo directement sur edge node proche de l'utilisateur
        """
        # 1. Stocker original localement sur edge
        original_key = f"videos/{user_id}/{video_file.id}/original.mp4"
        await self.edge_storage.put_local(original_key, video_file.data)

        # 2. Générer thumbnail immédiatement
        thumbnail = await self.generate_thumbnail(video_file)
        await self.edge_storage.put_local(
            f"videos/{user_id}/{video_file.id}/thumb.jpg",
            thumbnail
        )

        # 3. Lancer transcodages en parallèle (GPU edge)
        transcode_tasks = [
            self.transcode_video(video_file, profile)
            for profile in self.transcode_profiles
        ]

        results = await asyncio.gather(*transcode_tasks)

        # 4. Stocker transcodés localement
        for i, result in enumerate(results):
            profile = self.transcode_profiles[i]
            key = f"videos/{user_id}/{video_file.id}/{profile['resolution']}.mp4"
            await self.edge_storage.put_local(key, result)

        # 5. Sync asynchrone vers stockage central (backup + global dist)
        await self.sync_to_central_async(user_id, video_file.id)

        # 6. Vidéo immédiatement disponible depuis edge local
        return {
            'status': 'ready',
            'playback_urls': self.generate_playback_urls(user_id, video_file.id),
            'processing_time_ms': results[0]['processing_time']
        }

    async def transcode_video(self, video, profile):
        """
        Transcoding avec FFmpeg sur GPU edge
        """
        start = time.time()

        output = await ffmpeg.run_async(
            input=video.data,
            output_format='mp4',
            video_codec='h264_nvenc',  # GPU encoding
            resolution=profile['resolution'],
            bitrate=profile['bitrate'],
            preset='fast',
            hardware_accel='cuda'
        )

        processing_time = (time.time() - start) * 1000

        return {
            'data': output,
            'processing_time': processing_time,
            'resolution': profile['resolution']
        }

    async def sync_to_central_async(self, user_id, video_id):
        """
        Sync asynchrone en arrière-plan
        """
        # Upload vers S3 central en background
        # Ne bloque pas la disponibilité immédiate

        background_task = asyncio.create_task(
            self.upload_to_s3_central(user_id, video_id)
        )

        # Ne pas await : exécution asynchrone
```

**Résultats** :

```
Avant (central) :
- Upload time : 2-5 min (transfert vers central)
- Transcoding time : 10-30 min
- Total time to available : 12-35 min
- User experience : attendred 30+ min

Après (edge) :
- Upload time : 10-30 sec (edge proche)
- Transcoding time : 1-3 min (GPU local)
- Total time to available : 1.5-3.5 min
- User experience : immédiat (thumbnail + preview)
- Improvement : 10x plus rapide

Coûts :
- Bande passante : -80% (pas d'upload vers central)
- Compute : équivalent (edge GPU vs central GPU)
- Storage : +20% (réplication edge + central)
- Net : -60% coût total
```

### Cas 4 : IoT aggregation avec edge fog

**Contexte** : Smart building avec 10,000 capteurs (température, CO2, luminosité, présence).

**Problème classique** :

```
10,000 capteurs × 1 lecture/minute = 167 lectures/sec
167 lectures/sec × 60 sec/min × 60 min/h × 24 h = 14M lectures/jour

Upload vers cloud :
- 14M × 100 bytes = 1.4 GB/jour
- Coût cloud ingestion + storage
- Latence pour analytics/alertes
- Bande passante constante
```

**Solution edge fog** :

```python
# Edge Gateway pour smart building
import asyncio
from typing import Dict, List
import numpy as np

class SmartBuildingEdgeGateway:
    def __init__(self):
        self.sensors = {}  # {sensor_id: SensorState}
        self.edge_db = EdgeTimeSeriesDB()
        self.alert_rules = self.load_alert_rules()
        self.aggregation_window = 300  # 5 minutes

    async def process_sensor_reading(self, sensor_id: str, reading: Dict):
        """
        Traiter lecture capteur localement sur edge gateway
        """
        # 1. Stocker localement pour trending court-terme
        await self.edge_db.insert(sensor_id, reading)

        # 2. Détection d'anomalies en temps réel
        if self.is_anomalous(sensor_id, reading):
            await self.trigger_alert(sensor_id, reading, 'anomaly')

        # 3. Vérifier règles d'alerte
        alerts = self.check_alert_rules(sensor_id, reading)
        for alert in alerts:
            await self.trigger_alert(sensor_id, reading, alert)

        # 4. Agrégation : envoyer seulement résumés au cloud
        if self.should_aggregate():
            await self.send_aggregated_to_cloud()

    def is_anomalous(self, sensor_id: str, reading: Dict) -> bool:
        """
        ML inference léger sur edge pour détection anomalies
        """
        # Récupérer historique récent depuis edge DB
        recent = self.edge_db.get_recent(sensor_id, window=3600)  # 1 heure

        if len(recent) < 10:
            return False

        # Statistiques simples
        values = [r['value'] for r in recent]
        mean = np.mean(values)
        std = np.std(values)

        # Détection simple : >3 sigma
        current = reading['value']
        z_score = abs((current - mean) / std) if std > 0 else 0

        return z_score > 3

    def check_alert_rules(self, sensor_id: str, reading: Dict) -> List[str]:
        """
        Vérifier règles métier localement
        """
        alerts = []

        sensor_type = self.sensors[sensor_id]['type']
        value = reading['value']

        # Exemple : CO2 trop élevé
        if sensor_type == 'co2' and value > 1000:  # ppm
            alerts.append('high_co2')

        # Température hors limites
        if sensor_type == 'temperature' and (value < 15 or value > 30):
            alerts.append('temperature_out_of_range')

        # Présence détectée dans zone fermée
        if sensor_type == 'presence' and value == 1:
            zone = self.sensors[sensor_id]['zone']
            if self.is_zone_closed(zone):
                alerts.append('unauthorized_presence')

        return alerts

    async def trigger_alert(self, sensor_id: str, reading: Dict, alert_type: str):
        """
        Déclencher alerte immédiatement (action locale)
        """
        # 1. Action locale immédiate
        if alert_type == 'high_co2':
            # Ouvrir ventilation automatiquement
            await self.control_hvac(zone=self.sensors[sensor_id]['zone'], action='increase_ventilation')

        # 2. Notifier opérateurs
        await self.send_notification({
            'type': alert_type,
            'sensor': sensor_id,
            'value': reading['value'],
            'timestamp': reading['timestamp'],
            'action_taken': 'ventilation_increased'
        })

        # 3. Logger dans edge DB
        await self.edge_db.log_alert(sensor_id, alert_type, reading)

    async def send_aggregated_to_cloud(self):
        """
        Envoyer seulement données agrégées au cloud
        """
        # Au lieu de 10,000 lectures individuelles toutes les 5 min,
        # envoyer 100 agrégats (par zone/étage)

        aggregates = {}

        for zone in self.get_zones():
            zone_sensors = self.get_sensors_in_zone(zone)

            aggregates[zone] = {
                'temperature': {
                    'avg': np.mean([s['last_reading']['value'] for s in zone_sensors if s['type'] == 'temperature']),
                    'min': np.min([s['last_reading']['value'] for s in zone_sensors if s['type'] == 'temperature']),
                    'max': np.max([s['last_reading']['value'] for s in zone_sensors if s['type'] == 'temperature']),
                },
                'co2': {
                    'avg': np.mean([s['last_reading']['value'] for s in zone_sensors if s['type'] == 'co2']),
                    'max': np.max([s['last_reading']['value'] for s in zone_sensors if s['type'] == 'co2']),
                },
                'occupancy': sum([1 for s in zone_sensors if s['type'] == 'presence' and s['last_reading']['value'] == 1])
            }

        # Envoyer agrégats au cloud
        await self.cloud_api.send_aggregates(aggregates)

        # Réduction : 10,000 lectures → 100 agrégats = 99% reduction
```

**Résultats** :

```
Données envoyées au cloud :
Avant : 14M lectures/jour × 100 bytes = 1.4 GB/jour
Après : 28,800 agrégats/jour × 500 bytes = 14 MB/jour
Réduction : 99%

Coûts :
- Bande passante : -99%
- Stockage cloud : -95%
- Compute cloud : -90%
- Total : -95% coût cloud

Avantages additionnels :
- Alertes : temps réel (<1s) vs 5-30s avant
- Actions automatiques : immédiates
- Résilience : fonctionne même si cloud down
- Privacy : données sensibles restent on-premise
```

## Défis et considérations

### 1. Cohérence des données (CAP theorem)

```
CAP Theorem : On ne peut avoir que 2 sur 3 :
- Consistency (cohérence)
- Availability (disponibilité)
- Partition tolerance (tolérance aux partitions)

Edge computing = Réseau distribué → Partitions inévitables
→ Choix : Consistency vs Availability

Stratégies :
```

**a) Eventual Consistency (AP)** :

```javascript
// Accepter données temporairement incohérentes
class EventuallyConsistentEdge {
    async write(key, value) {
        // 1. Écrire localement immédiatement
        await this.localKV.put(key, value);

        // 2. Propager asynchroniquement vers autres edges
        this.propagateAsync(key, value);  // Fire and forget

        // 3. Retourner succès immédiatement
        return { success: true, latency: '2ms' };
    }

    async read(key) {
        // Lire version locale (peut être stale)
        return await this.localKV.get(key);
    }
}

// Use case : Sessions utilisateur, cache, analytics
// Trade-off : Performance ↑, Cohérence immédiate ↓
```

**b) Strong Consistency (CP)** :

```javascript
// Garantir cohérence au prix de disponibilité
class StronglyConsistentEdge {
    async write(key, value) {
        // 1. Écrire sur quorum d'edges (majority)
        const edges = this.selectQuorum();  // 3 sur 5 par exemple

        const results = await Promise.all(
            edges.map(edge => edge.write(key, value))
        );

        // 2. Attendre que majorité confirme
        const successes = results.filter(r => r.success).length;

        if (successes < Math.ceil(edges.length / 2)) {
            throw new Error('Quorum not reached');
        }

        return { success: true, latency: '50-200ms' };
    }

    async read(key) {
        // Lire depuis quorum pour garantir dernière version
        const edges = this.selectQuorum();

        const results = await Promise.all(
            edges.map(edge => edge.read(key))
        );

        // Retourner version la plus récente
        return this.selectLatestVersion(results);
    }
}

// Use case : Paiements, inventaire, commandes
// Trade-off : Cohérence ↑, Latence ↑, Disponibilité ↓
```

**c) Hybride (pattern courant)** :

```javascript
// Différentes cohérences selon le type de données
class HybridConsistencyEdge {
    async write(key, value, options = {}) {
        const consistency = options.consistency || 'eventual';

        if (consistency === 'strong') {
            // Données critiques : strong consistency
            return await this.strongWrite(key, value);
        } else {
            // Données non-critiques : eventual consistency
            return await this.eventualWrite(key, value);
        }
    }
}

// Exemples :
await db.write('user:session', session, { consistency: 'eventual' });
await db.write('order:payment', payment, { consistency: 'strong' });
await db.write('analytics:pageview', pageview, { consistency: 'eventual' });
```

### 2. Gestion d'état (stateful vs stateless)

**Problème** : Edge nodes peuvent être éphémères ou répartis.

```javascript
// ❌ Mauvais : État en mémoire sur edge
let sessionStore = {};  // Perdu si edge node redémarre

export default {
    async fetch(request) {
        const sessionId = getSessionId(request);

        // Problème : session peut ne pas exister si routé vers autre edge
        const session = sessionStore[sessionId];

        if (!session) {
            // Utilisateur perd sa session = mauvaise UX
            return new Response('Session expired', { status: 401 });
        }

        return handleRequest(session);
    }
};
```

**Solutions** :

```javascript
// ✅ Bon : État dans KV distribué
export default {
    async fetch(request, env) {
        const sessionId = getSessionId(request);

        // Session dans KV edge (distribué, persistent)
        const session = await env.SESSIONS.get(sessionId, { type: 'json' });

        if (!session) {
            return new Response('Session expired', { status: 401 });
        }

        return handleRequest(session);
    }
};

// ✅ Bon : Durable Objects pour état coordonné
export class StatefulSession {
    constructor(state) {
        this.state = state;
        this.storage = state.storage;

        // État persiste dans ce Durable Object
        this.state.blockConcurrencyWhile(async () => {
            this.data = await this.storage.get('sessionData') || {};
        });
    }

    async fetch(request) {
        // Toutes les requêtes pour cette session arrivent ici
        // État cohérent, coordonné

        const action = new URL(request.url).searchParams.get('action');

        if (action === 'update') {
            this.data.lastActivity = Date.now();
            await this.storage.put('sessionData', this.data);
        }

        return new Response(JSON.stringify(this.data));
    }
}
```

### 3. Cold starts et warmup

```javascript
// Problème : première requête vers edge peut être lente

// Cloudflare Workers : ~0ms cold start (V8 isolates)
// AWS Lambda@Edge : 100-1000ms cold start

// Mitigation :
class EdgeWarmer {
    constructor() {
        this.warmupInterval = 5 * 60 * 1000;  // 5 minutes
        this.startWarmer();
    }

    startWarmer() {
        setInterval(async () => {
            // Garder edge "chaud" avec requêtes dummy
            await fetch('https://my-edge-function.com/warmup');
        }, this.warmupInterval);
    }
}

// Ou : Provisioned concurrency (AWS Lambda@Edge)
// Coût : $$$ mais latence prévisible
```

### 4. Debugging et observabilité

```javascript
// Défi : Logs dispersés sur 200+ edge locations

// Solution : Structured logging vers central
export default {
    async fetch(request, env, ctx) {
        const requestId = crypto.randomUUID();
        const startTime = Date.now();

        try {
            const response = await handleRequest(request, env);

            // Logger succès
            ctx.waitUntil(logToDatadog({
                requestId,
                edge_location: request.cf?.colo,
                country: request.cf?.country,
                latency_ms: Date.now() - startTime,
                status: response.status,
                path: new URL(request.url).pathname,
                user_agent: request.headers.get('User-Agent')
            }));

            return response;

        } catch (error) {
            // Logger erreur
            ctx.waitUntil(logToDatadog({
                requestId,
                edge_location: request.cf?.colo,
                error: error.message,
                stack: error.stack,
                level: 'error'
            }));

            throw error;
        }
    }
};

async function logToDatadog(event) {
    await fetch('https://http-intake.logs.datadoghq.com/v1/input', {
        method: 'POST',
        headers: {
            'DD-API-KEY': API_KEY,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(event)
    });
}
```

### 5. Coûts

```
Modèle de coût edge computing :

Cloudflare Workers :
- $5/mois : 10M requêtes + 50ms CPU time each
- $0.50 per 1M requêtes additionnel
- CPU time : $0.02 per 1M GB-seconds

AWS Lambda@Edge :
- $0.60 per 1M requêtes
- $0.00005001 per GB-second compute

Vercel Edge Functions :
- Included dans plans (limits)
- $0.65 per 1M requêtes au-delà

Fastly Compute :
- $0.035 per 10k requêtes
- $1.20 per compute hour

Comparaison instance traditionnelle :
- EC2 t3.medium : $30/mois = 24/7 disponible
- Edge : pay-per-use, scale to zero
- Breakeven : ~500M requêtes/mois

Trade-offs :
- Edge : $0 si pas utilisé, scale automatiquement
- Traditional : coût fixe, mais pas de limites CPU time
```

## Best practices

### 1. Think edge-first, cloud-fallback

```typescript
// Architecture edge-first
class EdgeFirstArchitecture {
    async handleRequest(request: Request): Promise<Response> {
        // 1. Try edge first (fast path)
        try {
            return await this.handleAtEdge(request);
        } catch (edgeError) {
            console.warn('Edge processing failed:', edgeError);

            // 2. Fallback to cloud (slow path)
            return await this.fallbackToCloud(request);
        }
    }

    private async handleAtEdge(request: Request): Promise<Response> {
        // Processing local rapide
        const result = await this.processLocally(request);

        // Async sync vers cloud si nécessaire
        this.syncToCloudAsync(result);

        return new Response(JSON.stringify(result));
    }

    private async fallbackToCloud(request: Request): Promise<Response> {
        // Forwarder vers cloud origin
        return fetch('https://origin.example.com' + request.url);
    }
}
```

### 2. Optimiser pour read-heavy workloads

```javascript
// Edge excellent pour lectures, moins pour écritures

// ✅ Bon use case edge
async function getCatalogEdge(productId) {
    // Catalogue produit : read-heavy, update rare
    const cached = await edgeKV.get(`product:${productId}`);

    if (cached) {
        return cached;  // Hit rate : 95%+
    }

    // Fetch depuis central, cache localement
    const product = await fetchFromDB(productId);
    await edgeKV.put(`product:${productId}`, product, { ttl: 3600 });

    return product;
}

// ❌ Mauvais use case edge (sans précautions)
async function updateInventoryEdge(productId, quantity) {
    // Inventaire : write-heavy, strong consistency requise
    // Edge KV = eventual consistency → risque oversell

    // Solution : write to central, invalidate edge cache
    await centralDB.updateInventory(productId, quantity);
    await edgeKV.delete(`product:${productId}`);  // Invalidate cache
}
```

### 3. Minimiser compute time

```javascript
// Edge functions ont des limites CPU time strictes
// Cloudflare : 50ms, AWS Lambda@Edge : variable

// ❌ Mauvais : compute intensif
async function badEdgeFunction(data) {
    // Crypto mining à l'edge (lol non)
    for (let i = 0; i < 1000000; i++) {
        sha256(data + i);  // CPU limit exceeded !
    }
}

// ✅ Bon : compute minimal
async function goodEdgeFunction(data) {
    // Simple transformations
    const result = transformData(data);  // <5ms

    // Déléguer compute lourd au cloud
    if (needsHeavyProcessing(result)) {
        await sendToCloudQueue(result);  // Async
        return { status: 'processing' };
    }

    return result;
}
```

### 4. Gérer les timeouts gracieusement

```javascript
// Edge doit être rapide : timeout court

async function edgeAPICall(request) {
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 1000);  // 1s max

    try {
        const response = await fetch('https://slow-api.com/data', {
            signal: controller.signal
        });

        return response;

    } catch (error) {
        if (error.name === 'AbortError') {
            // Timeout : servir depuis cache stale
            const stale = await getStaleCache();

            if (stale) {
                return new Response(stale, {
                    headers: { 'X-Served-Stale': 'true' }
                });
            }

            // Vraiment aucune donnée : fast fail
            return new Response('Service temporarily unavailable', {
                status: 503,
                headers: { 'Retry-After': '60' }
            });
        }

        throw error;

    } finally {
        clearTimeout(timeout);
    }
}
```

### 5. Monitoring et alertes edge

```javascript
// Métriques critiques à surveiller

const metrics = {
    // Performance
    latency_p50: 10ms,  // Median
    latency_p95: 25ms,  // 95th percentile
    latency_p99: 50ms,  // 99th percentile

    // Disponibilité
    success_rate: 99.9%,
    error_rate: 0.1%,

    // Cache
    cache_hit_rate: 85%,
    cache_miss_rate: 15%,

    // Compute
    cpu_time_avg: 15ms,
    cpu_time_p99: 45ms,  // Warning si proche limite

    // Réseau
    origin_requests: 1000/sec,  // Combien passent par edge vs origin
    bandwidth_saved: 80%  // Grâce au caching edge
};

// Alertes
if (metrics.latency_p99 > 100ms) {
    alert('Edge latency degraded');
}

if (metrics.cache_hit_rate < 70%) {
    alert('Cache hit rate too low - investigate');
}

if (metrics.cpu_time_p99 > 45ms) {
    alert('Approaching CPU time limits');
}
```

## Futur de l'edge computing

### 1. WebAssembly (WASM) à l'edge

```rust
// Edge function en Rust compilée en WASM
use worker::*;

#[event(fetch)]
pub async fn main(req: Request, env: Env) -> Result<Response> {
    // Rust → WASM = Performance native
    // Sécurité sandboxing
    // Multi-langage (Rust, C++, Go compilent en WASM)

    let data = process_data(req.bytes().await?);

    Response::ok(data)
}

fn process_data(bytes: Vec<u8>) -> String {
    // Processing performant en Rust
    // ~10x plus rapide que JavaScript équivalent
    let result = heavy_computation(bytes);
    serde_json::to_string(&result).unwrap()
}

// Use case : image processing, crypto, compression
// Avantage : performance + portabilité
```

### 2. Edge AI / ML Inference

```python
# Modèle ML léger déployé sur edge

import onnx
import numpy as np

class EdgeMLInference:
    def __init__(self):
        # Modèle ONNX léger (quelques MB)
        self.model = onnx.load('model_quantized.onnx')
        self.session = onnxruntime.InferenceSession(self.model)

    def predict(self, image_data):
        """
        Inference locale sur edge
        """
        # Preprocessing
        input_tensor = preprocess_image(image_data)

        # Inference (<50ms sur edge GPU)
        outputs = self.session.run(None, {'input': input_tensor})

        # Postprocessing
        predictions = postprocess(outputs[0])

        return predictions

# Use cases :
# - Content moderation (images/text)
# - Recommendation (lightweight models)
# - Anomaly detection (IoT)
# - Image classification
# - Sentiment analysis

# Trade-off : Modèles simples seulement
# - Deep learning complexe → trop lourd pour edge
# - Fine-tuning → dans cloud central
```

### 3. Edge databases matures

```
Tendance : Databases conçues pour edge dès le départ

Turso (libSQL distributed) :
- Fork SQLite
- Réplication multi-edge
- Strong consistency optional
- Latence <10ms read

Cloudflare D1 :
- SQLite à l'edge
- Réplication automatique
- Serverless

Fly.io Postgres :
- Postgres full à l'edge
- Read replicas worldwide
- Write primary region

→ SQL complet à l'edge devient réalité
```

### 4. 5G + Edge = MEC (Multi-access Edge Computing)

```
5G Network :
                Internet
                    ↑
              [Core Network]
                    ↑
      ┌─────────────┴─────────────┐
      │                           │
[Edge Compute]              [Edge Compute]
(Base Station)              (Base Station)
      ↑                           ↑
   Device                      Device

Latence : <10ms (vs 50-100ms cloud)

Use cases :
- Véhicules autonomes
- AR/VR
- Gaming cloud
- Chirurgie à distance
- Industrie 4.0
```

## Conclusion

L'edge computing n'est pas qu'une optimisation : c'est un **changement de paradigme** dans l'architecture distribuée moderne. Pour les développeurs en 2025, concevoir edge-first devient essentiel.

**Points clés** :

✅ **Latence** : 10-100x réduction possible vs cloud central

✅ **Coûts** : Réduction bande passante, compute optimal

✅ **Résilience** : Applications fonctionnent même si cloud down

✅ **Conformité** : Données restent géographiquement localisées

✅ **Scale** : Horizontal illimité sans gérer infrastructure

**Quand utiliser l'edge** :

- Applications sensibles latence (gaming, trading, streaming)
- IoT et volumes de données massifs
- Personnalisation temps réel
- Disponibilité critique
- Conformité géographique

**Quand éviter l'edge** :

- Compute très intensif (ML training, rendering)
- Strong consistency absolument requise
- État complexe partagé globalement
- Budget très limité (petit volume)

**Actions immédiates** :

1. **Évaluer** vos workloads : lesquels bénéficieraient de l'edge ?
2. **Expérimenter** avec Cloudflare Workers ou Vercel Edge (gratuit pour commencer)
3. **Mesurer** l'impact sur latence et coûts
4. **Migrer** progressivement : edge-first, cloud-fallback
5. **Former** l'équipe aux patterns distributed systems

L'avenir est distribué. L'edge computing est déjà là.

---

**Ressources complémentaires** :
- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)
- [AWS Lambda@Edge](https://aws.amazon.com/lambda/edge/)
- [Vercel Edge Functions](https://vercel.com/docs/concepts/functions/edge-functions)
- [Fastly Compute](https://docs.fastly.com/products/compute)
- [CNCF Edge Computing Whitepaper](https://github.com/cncf/tag-runtime/blob/master/edge/whitepaper.md)

---


⏭️ [IoT et contraintes protocolaires](/10-evolutions-tendances/06-iot.md)
