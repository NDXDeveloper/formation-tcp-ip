🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.10 Architectures de communication modernes

## Introduction

L'évolution du développement logiciel et l'émergence des architectures distribuées ont profondément transformé la façon dont les applications communiquent sur le réseau. Si les sockets TCP/IP restent la fondation technique de toute communication réseau, les architectures de communication modernes apportent des abstractions de plus haut niveau qui simplifient le développement, améliorent la maintenabilité et répondent aux exigences spécifiques des applications contemporaines.

Ces architectures ne remplacent pas TCP/IP, mais s'appuient sur lui en définissant des conventions, des formats de données et des patterns de communication qui facilitent l'interopérabilité et le découplage entre services.

## Contexte historique et évolution

### Des RPC aux API modernes

Dans les années 1980-1990, les premières tentatives de communication entre systèmes distribués reposaient sur des mécanismes comme **RPC (Remote Procedure Call)** et **CORBA (Common Object Request Broker Architecture)**. Ces technologies cherchaient à masquer la complexité réseau en faisant apparaître les appels distants comme des appels de fonctions locales.

**Limitations des approches traditionnelles :**
- Couplage fort entre client et serveur
- Dépendance aux langages et plateformes spécifiques
- Gestion complexe des erreurs réseau
- Difficulté de versioning et d'évolution des interfaces
- Overhead important des protocoles propriétaires

### L'ère du Web et l'émergence de REST

L'explosion du Web dans les années 2000 a popularisé une approche radicalement différente : **REST (Representational State Transfer)**. Plutôt que de cacher le réseau, REST l'embrasse en s'appuyant sur HTTP et ses principes architecturaux.

**Avantages de cette transition :**
- Utilisation de standards universels (HTTP, JSON, XML)
- Simplicité conceptuelle et facilité de débogage
- Compatibilité avec les infrastructures Web existantes (proxies, caches, CDN)
- Indépendance des langages et plateformes
- Scalabilité naturelle grâce au modèle sans état

### La renaissance des RPC avec gRPC

Malgré le succès de REST, certaines limitations sont apparues dans des contextes spécifiques :
- Performance sur des réseaux à faible latence (datacenters)
- Communication bidirectionnelle et streaming
- Contrats d'API stricts et typage fort
- Génération automatique de code client/serveur

Google a développé **gRPC** en 2015, combinant les avantages des RPC modernes avec les innovations d'HTTP/2 et des formats binaires efficaces comme Protocol Buffers.

### GraphQL : une approche centrée sur les besoins du client

Facebook a introduit **GraphQL** en 2015 pour résoudre des problèmes spécifiques aux applications modernes :
- Sur-fetching et under-fetching de données avec REST
- Multiplication des endpoints pour différents clients (web, mobile, etc.)
- Évolution rapide des besoins côté client sans modification backend

GraphQL propose un modèle où le client décrit précisément les données dont il a besoin, via un langage de requête standardisé.

## Critères de choix d'une architecture de communication

Le choix d'une architecture de communication n'est pas binaire. Il dépend de multiples facteurs techniques et organisationnels qu'il faut évaluer dans le contexte de chaque projet.

### 1. Caractéristiques du réseau

**Latence et bande passante :**
- **Réseau WAN (Internet)** : REST ou GraphQL privilégiés pour leur compatibilité avec HTTP
- **Réseau LAN (datacenter)** : gRPC performant grâce à son format binaire et HTTP/2
- **Réseaux contraints (mobile, IoT)** : Formats binaires compacts (Protocol Buffers, MessagePack)

**Exemple concret :**
```
Application e-commerce :
- Frontend web → Backend : REST/GraphQL (HTTP/1.1 ou HTTP/2)
- Backend → Microservices internes : gRPC (datacenter, faible latence)
- Application mobile → API : GraphQL (optimisation de la bande passante)
```

### 2. Patterns de communication

**Request-Response synchrone :**
- REST : Adapté pour CRUD et opérations stateless
- gRPC : Efficace pour appels RPC entre services

**Streaming unidirectionnel :**
- gRPC Server Streaming : Flux de données du serveur vers le client
- Server-Sent Events (SSE) : Alternative HTTP/1.1 compatible

**Streaming bidirectionnel :**
- gRPC Bidirectional Streaming : Communication full-duplex
- WebSockets : Alternative pour le temps réel dans le navigateur

**Exemple - Service de notification :**
```
Cas d'usage : Système de notifications en temps réel

Architecture hybride :
1. Client web → Serveur notifications : WebSocket (temps réel dans navigateur)
2. Serveur notifications → Service utilisateurs : gRPC streaming (push des événements)
3. Client mobile → Serveur : SSE ou polling (compatibilité réseau mobile)
```

### 3. Typage et contrat d'API

**REST :**
- Typage faible, validation généralement via JSON Schema ou OpenAPI
- Flexibilité mais risque d'incohérences entre versions
- Documentation séparée du code

**gRPC :**
- Typage fort via Protocol Buffers (.proto files)
- Génération automatique de code client/serveur
- Contrat d'API versionné et strictement défini

**GraphQL :**
- Schéma fortement typé (GraphQL SDL)
- Introspection du schéma côté client
- Validation automatique des requêtes

**Exemple - Service bancaire :**
```protobuf
// gRPC - Contrat strict pour transactions financières
service BankingService {
  rpc TransferFunds(TransferRequest) returns (TransferResponse);
  rpc GetBalance(BalanceRequest) returns (BalanceResponse);
}

message TransferRequest {
  string from_account = 1;  // Typage strict
  string to_account = 2;
  int64 amount_cents = 3;   // Évite les problèmes de float
  string currency = 4;
}
```

Ici, gRPC garantit que les montants sont toujours des entiers (en centimes) et que tous les champs requis sont présents, réduisant les erreurs à l'exécution.

### 4. Performance et efficacité

**Sérialisation :**
- **JSON (REST, GraphQL)** : Lisible, universel, mais verbeux (~30-40% overhead)
- **Protocol Buffers (gRPC)** : Binaire compact, 3-10x plus petit que JSON
- **Compression** : Gzip/Brotli avec REST, compression native avec HTTP/2

**Multiplexage :**
- **HTTP/1.1 (REST classique)** : 1 requête par connexion, head-of-line blocking
- **HTTP/2 (gRPC, REST moderne)** : Multiplexage de multiples requêtes sur une connexion
- **Impact** : Réduction drastique de la latence pour multiples appels

**Benchmark indicatif (ordre de grandeur) :**
```
Scénario : 1000 appels API simples (get user by ID)

HTTP/1.1 + JSON (REST classique) :
- Taille payload : ~250 KB total
- Temps : ~2500ms (sans keep-alive)
- Temps : ~800ms (avec keep-alive)

HTTP/2 + JSON (REST moderne) :
- Taille payload : ~250 KB total
- Temps : ~400ms (multiplexage)

HTTP/2 + Protobuf (gRPC) :
- Taille payload : ~80 KB total
- Temps : ~200ms (binaire + multiplexage)
```

### 5. Interopérabilité et écosystème

**REST :**
- Support universel (tous les langages, navigateurs, outils)
- Débogage simple avec curl, Postman, navigateur
- Compatible avec toute infrastructure HTTP existante

**gRPC :**
- Support croissant mais moins universel que REST
- Pas de support natif dans les navigateurs (nécessite gRPC-Web)
- Excellent pour communication service-to-service

**GraphQL :**
- Nécessite des bibliothèques spécifiques (Apollo, Relay)
- Tooling riche (GraphiQL, Apollo Studio)
- Courbe d'apprentissage pour les équipes

**Exemple - Startup avec contraintes de temps :**
```
Contexte : MVP à lancer en 3 mois, équipe junior

Choix : REST
Raisons :
- Connaissance universelle (pas de formation nécessaire)
- Tooling simple (Postman pour tests)
- Débogage facile (inspection réseau du navigateur)
- Bibliothèques matures dans tous les langages
- Pas de complexité supplémentaire à gérer
```

### 6. Complexité opérationnelle

**REST :**
- Infrastructure simple : serveur HTTP standard
- Monitoring : logs HTTP classiques
- Debugging : outils web standards

**gRPC :**
- Nécessite support HTTP/2
- Logs binaires moins lisibles
- Tooling spécifique pour debugging (grpcurl, grpc-ui)

**GraphQL :**
- Serveur GraphQL à maintenir
- Complexité de cache (chaque requête est unique)
- Monitoring des requêtes complexes (n+1 queries, profondeur)

## Architectures hybrides : le pragmatisme en pratique

Dans la réalité, les systèmes modernes utilisent rarement une seule approche. Les architectures hybrides combinent plusieurs patterns selon les besoins spécifiques de chaque composant.

### Cas d'usage 1 : Plateforme de streaming vidéo

```
Architecture complète :

1. API publique (applications tierces) :
   → REST
   Raisons : Simplicité, documentation, compatibilité universelle

2. Frontend web → Backend :
   → GraphQL
   Raisons : Flexibilité des requêtes, optimisation bande passante mobile

3. Microservices internes (catalogue, recommandations, analytics) :
   → gRPC
   Raisons : Performance, typage fort, streaming pour métriques temps réel

4. Streaming vidéo :
   → Protocole spécialisé (HLS/DASH over HTTP)
   Raisons : Optimisé pour la vidéo, support CDN

5. Chat en direct :
   → WebSocket
   Raisons : Bidirectionnel, faible latence
```

### Cas d'usage 2 : Plateforme bancaire

```
Architecture sécurisée et résiliente :

1. Applications mobiles → API Gateway :
   → REST avec OAuth2/JWT
   Raisons : Standards de sécurité éprouvés, compatibilité mobile

2. API Gateway → Services métier (comptes, transactions) :
   → gRPC avec mTLS
   Raisons : Performance, typage strict, authentification mutuelle

3. Services métier → Base de données :
   → Connexions natives optimisées

4. Notifications temps réel :
   → Server-Sent Events (SSE)
   Raisons : Unidirectionnel suffisant, compatible avec reverse proxies

5. Synchronisation inter-datacenters :
   → gRPC streaming
   Raisons : Efficacité, gestion automatique de la reconnexion
```

### Cas d'usage 3 : Application SaaS collaborative

```
Éditeur de documents collaboratif (type Google Docs) :

1. Client web → Serveur applicatif :
   → WebSocket pour édition temps réel
   Raisons : Latence minimale, synchronisation bidirectionnelle

2. Client web → API de gestion :
   → GraphQL
   Raisons : Requêtes flexibles pour différentes vues (liste docs, permissions, etc.)

3. Serveur applicatif → Service de stockage :
   → gRPC
   Raisons : Performance pour sauvegardes fréquentes

4. Export/Import de documents :
   → REST
   Raisons : Compatible avec intégrations tierces (Zapier, etc.)

5. Analytics et télémétrie :
   → gRPC streaming
   Raisons : Volume important de métriques, batch processing
```

## Impact sur la conception d'applications

### Découplage et évolutivité

Les architectures modernes favorisent le découplage entre composants, permettant :

**Évolution indépendante :**
```
API versioning avec REST :
- /api/v1/users (version stable)
- /api/v2/users (nouvelles fonctionnalités)
- Clients migrés progressivement

Backward compatibility avec gRPC :
- Ajout de champs optionnels dans .proto
- Clients anciens ignorent les nouveaux champs
- Serveur gère les anciennes versions
```

**Scalabilité horizontale :**
- API stateless → Load balancing simple
- Services gRPC → Service mesh (Istio, Linkerd)
- GraphQL → Cache distribué (Redis) pour requêtes fréquentes

### Résilience et tolérance aux pannes

**Circuit breakers et retry logic :**
```javascript
// Exemple avec REST et bibliothèque axios
const axiosRetry = require('axios-retry');

axiosRetry(axios, {
  retries: 3,
  retryDelay: axiosRetry.exponentialDelay,
  retryCondition: (error) => {
    return axiosRetry.isNetworkOrIdempotentRequestError(error)
           || error.response.status === 429; // Rate limiting
  }
});
```

**Timeouts appropriés :**
```go
// gRPC avec timeout contextuel
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

response, err := client.GetUser(ctx, &pb.UserRequest{Id: userId})
if err != nil {
    // Gestion timeout ou erreur réseau
}
```

### Observabilité

Les architectures modernes doivent être observables pour diagnostiquer les problèmes en production.

**Tracing distribué (OpenTelemetry) :**
```
Requête client → API Gateway → Service A → Service B → Database

Trace complète :
1. [API Gateway] POST /api/orders (200ms total)
   ├─ [Service A] CreateOrder (150ms)
   │  ├─ [Service B] CheckInventory gRPC (50ms)
   │  └─ [Database] INSERT order (80ms)
   └─ [Response] HTTP 201
```

**Métriques par architecture :**
- REST : Latence par endpoint, codes HTTP, taux d'erreur
- gRPC : RPC latency, streaming duration, erreurs par service
- GraphQL : Complexité des requêtes, nombre de resolvers, cache hit rate

## Considérations de sécurité

### Authentication et Authorization

**REST :**
- OAuth2, JWT dans headers HTTP
- API Keys pour services publics
- CORS pour applications web

**gRPC :**
- mTLS (mutual TLS) pour service-to-service
- JWT dans metadata gRPC
- Interceptors pour validation centralisée

**GraphQL :**
- Schema-level permissions
- Field-level authorization
- Rate limiting complexe (basé sur coût de requête)

### Transport et chiffrement

**Tous les protocoles modernes utilisent TLS :**
```
REST : HTTPS (TLS 1.2+)
gRPC : HTTP/2 over TLS
GraphQL : HTTPS (TLS 1.2+)

Bonnes pratiques communes :
- Certificats valides (Let's Encrypt, autorités reconnues)
- Cipher suites modernes (AES-GCM, ChaCha20-Poly1305)
- Perfect Forward Secrecy (ECDHE)
- HSTS (HTTP Strict Transport Security)
```

### Validation et sanitization

**Côté serveur (toujours obligatoire) :**
```python
# REST avec validation JSON Schema
from jsonschema import validate

user_schema = {
    "type": "object",
    "properties": {
        "email": {"type": "string", "format": "email"},
        "age": {"type": "integer", "minimum": 0, "maximum": 150}
    },
    "required": ["email"]
}

validate(instance=request_data, schema=user_schema)
```

**gRPC : Validation automatique via Proto :**
```protobuf
message CreateUserRequest {
  string email = 1;  // Validé comme string UTF-8
  int32 age = 2;     // Validé comme entier 32-bit
  // Validation métier additionnelle dans le code serveur
}
```

## Tendances émergentes

### 1. API Federation et Mesh

**Concept :** Unifier plusieurs API derrière une gateway unique
```
Client → API Gateway (GraphQL Federation)
         ├─ Service Users (gRPC)
         ├─ Service Orders (REST)
         └─ Service Inventory (GraphQL)

Avantages :
- Point d'entrée unique pour les clients
- Composition de données de sources hétérogènes
- Abstraction des détails d'implémentation
```

### 2. Serverless et Edge Computing

**Impact sur les architectures :**
- Fonctions courtes durée → REST/GraphQL privilégiés
- Cold start latency → Optimisation taille payload
- Distribution géographique → CDN et edge functions

### 3. Contract Testing et Consumer-Driven Contracts

**Évolution du testing :**
```
Approche traditionnelle : Tests end-to-end fragiles

Contract testing (Pact, Spring Cloud Contract) :
- Contrats définis par les consumers
- Tests isolés côté provider et consumer
- Détection précoce des breaking changes
```

### 4. API-First Development

**Philosophie moderne :**
1. Définir l'API avant l'implémentation (OpenAPI, Proto, GraphQL SDL)
2. Générer code client/serveur depuis la spécification
3. Valider les contrats automatiquement
4. Documentation toujours à jour

## Recommandations stratégiques

### Pour une nouvelle application

**Questions à se poser :**
1. Quelle est la nature des communications ? (CRUD simple vs interactions complexes)
2. Qui sont les consommateurs ? (Web public, mobile, services internes)
3. Quelles sont les contraintes de performance ? (Latence, bande passante)
4. Quelle est l'expertise de l'équipe ? (Courbe d'apprentissage acceptable ?)
5. Quelle est la criticité ? (Besoin de typage fort, contrats stricts ?)

**Scénarios de décision rapide :**

| Contexte | Recommandation | Raison |
|----------|---------------|--------|
| API publique pour développeurs tiers | REST | Universalité, documentation, tooling |
| Application mobile data-intensive | GraphQL | Optimisation requêtes, flexibilité |
| Microservices internes | gRPC | Performance, typage, génération code |
| Application temps réel (chat, gaming) | WebSocket | Bidirectionnel, faible latence |
| Dashboard monitoring | SSE ou GraphQL subscriptions | Streaming unidirectionnel |

### Migration progressive

**Éviter la réécriture complète :**
```
Étape 1 : Identifier les composants critiques
         → Migrer vers gRPC pour performance

Étape 2 : Nouveaux services en architecture cible
         → GraphQL pour frontend, gRPC en backend

Étape 3 : API Gateway comme adaptateur
         → Traduire REST legacy vers gRPC moderne

Étape 4 : Décommissionnement progressif du legacy
         → Une fois tous les clients migrés
```

## Conclusion

Les architectures de communication modernes ne sont pas des modes passagères, mais des réponses pragmatiques aux défis spécifiques du développement logiciel contemporain. REST, gRPC et GraphQL coexistent car ils excellent dans des domaines différents :

- **REST** demeure la référence pour les API publiques, simples et interopérables
- **gRPC** s'impose pour les communications haute performance entre services
- **GraphQL** répond aux besoins de flexibilité des clients modernes (web, mobile)

La clé du succès réside dans la capacité à choisir l'outil adapté à chaque contexte, et à accepter qu'une architecture hybride est souvent la solution la plus pragmatique.

Dans les sections suivantes, nous explorerons en détail chacune de ces architectures, leurs mécanismes réseau sous-jacents, et leurs implications concrètes pour les développeurs.

---

**Points clés à retenir :**
- Les architectures modernes s'appuient sur TCP/IP mais offrent des abstractions de plus haut niveau
- Le choix dépend de multiples facteurs : performance, interopérabilité, complexité, équipe
- Les architectures hybrides sont la norme dans les systèmes réels
- L'évolution d'HTTP (1.1 → 2 → 3) a rendu possible de nouvelles approches (gRPC, multiplexage)
- La sécurité, l'observabilité et la résilience sont des préoccupations transverses à toutes les architectures

**Prochaine section :** 8.10.1 REST : principes et contraintes réseau

⏭️ [REST : principes et contraintes réseau](/08-programmation-reseau/10.1-rest.md)
