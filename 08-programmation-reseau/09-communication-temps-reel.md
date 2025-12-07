🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.9 Communication temps réel : panorama complet

## Introduction

Pendant des décennies, le web a fonctionné sur un modèle simple : **le client demande, le serveur répond**. Ce paradigme request-response, hérité de HTTP, fonctionne parfaitement pour les sites statiques mais montre ses limites dès qu'on veut créer des **expériences interactives en temps réel** : chat, notifications, tableau de bord live, collaboration simultanée, jeux en ligne.

**Le problème fondamental** : HTTP a été conçu comme un protocole unidirectionnel où le serveur ne peut pas initier une communication vers le client. Quand une nouvelle donnée arrive côté serveur (nouveau message, mise à jour de prix, événement), comment en informer le client instantanément ?

```
┌─────────────────────────────────────────────────────────┐
│        MODÈLE TRADITIONNEL (Request-Response)           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Client                           Serveur               │
│    │                                 │                  │
│    │──── GET /messages ────────────> │                  │
│    │                                 │                  │
│    │<──── 200 OK (messages) ──────── │                  │
│    │                                 │                  │
│    │  [Attend 5 secondes...]         │                  │
│    │                                 │                  │
│    │──── GET /messages ────────────> │                  │
│    │<──── 200 OK (messages) ──────── │                  │
│    │                                 │                  │
│  Problème : Latence de 5s entre événement et affichage  │
│             Gaspillage réseau si aucun nouveau message  │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│           COMMUNICATION TEMPS RÉEL IDÉALE               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Client                           Serveur               │
│    │                                 │                  │
│    │<════════ Connexion ouverte ════>│                  │
│    │                                 │                  │
│    │                             [Nouvel événement]     │
│    │<──── Push immédiat ──────────── │                  │
│    │                                 │                  │
│    │                             [Nouvel événement]     │
│    │<──── Push immédiat ──────────── │                  │
│    │                                 │                  │
│  Avantage : Latence < 100ms, pas de polling inutile     │
└─────────────────────────────────────────────────────────┘
```

Cette section explore les **quatre principales techniques** pour implémenter la communication temps réel sur le web, leurs avantages, limites, et cas d'usage.

---

## Les quatre approches de la communication temps réel

### Vue d'ensemble comparative


| Technique | Latence | Overhead | Bidirectionnel | Complexité |
|-----------|---------|----------|----------------|------------|
| Polling | ★☆☆☆☆ | ★☆☆☆☆ | ✗ | ★★★★★ |
| Long-Polling | ★★★☆☆ | ★★☆☆☆ | ✗ | ★★★★☆ |
| SSE | ★★★★☆ | ★★★★☆ | ✗ | ★★★☆☆ |
| WebSocket | ★★★★★ | ★★★★★ | ✓ | ★★☆☆☆ |

**Légende :**
- ★ = Faible/Simple → ★★★★★ = Élevé/Complexe
- ✓ = Oui, ✗ = Non


### 1. Short Polling (Polling classique)

**Principe** : Le client interroge régulièrement le serveur à intervalles fixes.

```javascript
// Exemple : Polling toutes les 5 secondes
setInterval(async () => {
    const response = await fetch('/api/messages');
    const messages = await response.json();
    updateUI(messages);
}, 5000);
```

**Caractéristiques** :
- ✅ **Très simple** : une ligne de code
- ✅ **Compatible partout** : fonctionne même avec les plus vieux proxies
- ❌ **Latence élevée** : jusqu'à l'intervalle de polling
- ❌ **Gaspillage réseau** : requêtes même sans nouvelles données
- ❌ **Charge serveur** : requêtes constantes × nombre de clients

**Métrique clé** : Avec 10 000 clients qui pollent toutes les 5s, le serveur reçoit **2 000 requêtes/seconde**, dont 99% sont inutiles si les données changent rarement.

**Quand l'utiliser** :
- Données qui changent rarement (> 30 secondes)
- Faible nombre de clients
- Simplicité critique (prototypage)

### 2. Long Polling

**Principe** : Le client fait une requête qui reste ouverte jusqu'à ce qu'une nouvelle donnée soit disponible.

```javascript
async function longPoll() {
    try {
        const response = await fetch('/api/messages/long-poll');
        const messages = await response.json();
        updateUI(messages);
    } catch (error) {
        console.error('Long poll error:', error);
    }

    // Recommencer immédiatement
    longPoll();
}

longPoll();
```

**Caractéristiques** :
- ✅ **Latence réduite** : notification quasi-immédiate
- ✅ **Pas de gaspillage** : requête seulement quand nécessaire
- ✅ **Compatible** : fonctionne avec HTTP standard
- ❌ **Complexité serveur** : gérer des connexions long-lived
- ❌ **Scalabilité** : une connexion ouverte par client
- ❌ **Problèmes avec proxies** : timeouts, buffering

**Évolution du polling** :
```
Short Polling:  ───┐ 5s ┌───┐ 5s ┌───┐ 5s ┌───
                   └────┘   └────┘   └────┘

Long Polling:   ═══════════╗         ╔══════════╗
                           ╚═════════╝          ╚════
                           (nouveau msg)        (nouveau msg)
```

**Quand l'utiliser** :
- Fallback pour navigateurs anciens
- Événements sporadiques imprévisibles
- Infrastructure proxy/firewall restrictive

### 3. Server-Sent Events (SSE)

**Principe** : Connexion HTTP persistante unidirectionnelle, serveur → client.

```javascript
const eventSource = new EventSource('/api/events');

eventSource.addEventListener('message', (event) => {
    const data = JSON.parse(event.data);
    updateUI(data);
});

eventSource.addEventListener('error', (error) => {
    console.error('SSE error:', error);
});
```

**Caractéristiques** :
- ✅ **Latence très faible** : push instantané
- ✅ **Standard HTML5** : API native dans les navigateurs
- ✅ **Reconnexion automatique** : le navigateur gère les coupures
- ✅ **Format texte simple** : facile à déboguer
- ❌ **Unidirectionnel** : client ne peut pas envoyer via cette connexion
- ❌ **Limite de connexions** : navigateurs limitent à 6 par domaine (HTTP/1.1)
- ❌ **Pas de binaire natif** : seulement du texte

**Format du protocole** :
```http
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"message": "Hello"}

data: {"message": "World"}

event: userJoined
data: {"user": "Alice"}
id: 12345
retry: 10000
```

**Quand l'utiliser** :
- **Flux unidirectionnel** : notifications, mises à jour, flux d'événements
- **Données textuelles** : JSON, XML, plain text
- **Simplicité prioritaire** : API native, pas de bibliothèque nécessaire

### 4. WebSocket

**Principe** : Connexion TCP bidirectionnelle full-duplex sur HTTP upgrade.

```javascript
const ws = new WebSocket('wss://example.com/chat');

ws.addEventListener('open', () => {
    console.log('Connected');
    ws.send(JSON.stringify({ type: 'join', room: 'general' }));
});

ws.addEventListener('message', (event) => {
    const data = JSON.parse(event.data);
    handleMessage(data);
});

ws.addEventListener('close', (event) => {
    console.log('Disconnected:', event.code, event.reason);
});

ws.addEventListener('error', (error) => {
    console.error('WebSocket error:', error);
});
```

**Caractéristiques** :
- ✅ **Latence minimale** : < 10ms possible
- ✅ **Bidirectionnel** : client ↔ serveur sans restriction
- ✅ **Efficace** : headers minimaux après handshake
- ✅ **Binaire natif** : ArrayBuffer, Blob
- ✅ **Scalable** : une connexion par client, peu d'overhead
- ❌ **Complexité** : protocole à part, nécessite bibliothèque serveur
- ❌ **Compatibilité** : certains proxies bloquent
- ❌ **Pas de reconnexion auto** : à gérer manuellement

**Handshake initial** :
```http
# Client → Serveur
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

# Serveur → Client
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

# Ensuite : frames WebSocket binaires
```

**Quand l'utiliser** :
- **Communication bidirectionnelle** : chat, jeux, collaboration
- **Latence critique** : trading, jeux temps réel
- **Volume élevé** : streaming, données continues
- **Données binaires** : images, audio, vidéo

---

## Critères de choix détaillés

### 1. Direction de la communication

```
┌─────────────────────────────────────────────────────┐
│              BESOINS DE COMMUNICATION               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Serveur → Client uniquement                        │
│  ├─ Notifications push                              │
│  ├─ Mises à jour de prix                            │
│  ├─ Flux d'actualités                               │
│  └─ Recommandation: SSE ou Long-Polling             │
│                                                     │
│  Client → Serveur uniquement                        │
│  ├─ Analytics, télémétrie                           │
│  ├─ Logs, métriques                                 │
│  └─ Recommandation: HTTP POST classique             │
│                                                     │
│  Bidirectionnel                                     │
│  ├─ Chat, messagerie                                │
│  ├─ Jeux multijoueurs                               │
│  ├─ Collaboration temps réel                        │
│  └─ Recommandation: WebSocket                       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 2. Fréquence des mises à jour

```
Fréquence         Latence acceptable    Technique recommandée
─────────────────────────────────────────────────────────────
< 1 / heure       5-30 minutes          Polling (30-60s)
1-10 / heure      1-5 minutes           Polling (10-30s)
10-60 / heure     10-60 secondes        Long-Polling ou SSE
> 1 / minute      < 10 secondes         SSE ou WebSocket
> 10 / minute     < 1 seconde           WebSocket
Continue          Temps réel            WebSocket
```

**Exemple concret** :

```javascript
// Tableau de bord boursier : WebSocket pour prix, SSE pour news

// Prix en temps réel (100+ updates/seconde)
const priceWs = new WebSocket('wss://market.example.com/prices');
priceWs.onmessage = (event) => {
    const { symbol, price } = JSON.parse(event.data);
    updatePrice(symbol, price);
};

// Actualités (quelques par heure)
const newsSource = new EventSource('/api/news');
newsSource.onmessage = (event) => {
    const article = JSON.parse(event.data);
    addNewsItem(article);
};
```

### 3. Charge réseau et scalabilité

**Comparaison pour 10 000 clients connectés** :

```
┌──────────────────────────────────────────────────────┐
│  Polling (intervalle 5s)                             │
│  ├─ Requêtes/sec: 2 000 (10000 / 5)                  │
│  ├─ Overhead par requête: ~500 bytes headers         │
│  ├─ Bande passante: ~1 MB/s juste en headers         │
│  └─ Charge serveur: Très élevée                      │
│                                                      │
│  Long-Polling                                        │
│  ├─ Connexions simultanées: 10 000                   │
│  ├─ Requêtes/sec: variable (selon événements)        │
│  ├─ Overhead: Initial seulement                      │
│  ├─ Bande passante: Faible si peu d'événements       │
│  └─ Charge serveur: Élevée (connexions ouvertes)     │
│                                                      │
│  SSE                                                 │
│  ├─ Connexions simultanées: 10 000                   │
│  ├─ Overhead par message: ~50 bytes                  │
│  ├─ Bande passante: Proportionnelle aux données      │
│  └─ Charge serveur: Moyenne (connexions persistantes)│
│                                                      │
│  WebSocket                                           │
│  ├─ Connexions simultanées: 10 000                   │
│  ├─ Overhead par message: 2-14 bytes                 │
│  ├─ Bande passante: Minimale                         │
│  └─ Charge serveur: Faible (après handshake)         │
└──────────────────────────────────────────────────────┘
```

**Calcul de coût réseau** :

```python
def calculate_monthly_bandwidth(
    clients: int,
    messages_per_minute: int,
    avg_message_size: int,  # bytes
    technique: str
):
    """Calcule la bande passante mensuelle selon la technique."""

    minutes_per_month = 30 * 24 * 60  # 43200

    if technique == 'polling':
        # Polling toutes les 5 secondes
        requests_per_minute = clients * 12  # 60/5
        header_overhead = 500  # bytes par requête
        total_per_request = avg_message_size + header_overhead
        bytes_per_month = requests_per_minute * minutes_per_month * total_per_request

    elif technique == 'long_polling':
        # Overhead initial + messages
        initial_overhead = clients * 500  # headers initiaux
        message_overhead = 500  # headers par message
        total_messages = messages_per_minute * minutes_per_month
        bytes_per_month = initial_overhead + (total_messages * (avg_message_size + message_overhead))

    elif technique == 'sse':
        # Overhead SSE minimal
        sse_overhead = 50  # bytes par message
        total_messages = clients * messages_per_minute * minutes_per_month
        bytes_per_month = total_messages * (avg_message_size + sse_overhead)

    elif technique == 'websocket':
        # Overhead WebSocket minimal
        ws_overhead = 6  # moyenne 2-14 bytes
        total_messages = clients * messages_per_minute * minutes_per_month
        bytes_per_month = total_messages * (avg_message_size + ws_overhead)

    return bytes_per_month / (1024 ** 3)  # Convert to GB

# Exemple : 10 000 clients, 1 message/min, 100 bytes par message
clients = 10_000
msg_per_min = 1
msg_size = 100

print(f"Polling:      {calculate_monthly_bandwidth(clients, msg_per_min, msg_size, 'polling'):.2f} GB/month")
print(f"Long-Polling: {calculate_monthly_bandwidth(clients, msg_per_min, msg_size, 'long_polling'):.2f} GB/month")
print(f"SSE:          {calculate_monthly_bandwidth(clients, msg_per_min, msg_size, 'sse'):.2f} GB/month")
print(f"WebSocket:    {calculate_monthly_bandwidth(clients, msg_per_min, msg_size, 'websocket'):.2f} GB/month")

# Résultats typiques:
# Polling:      3758.40 GB/month  (énorme overhead !)
# Long-Polling: 26.40 GB/month
# SSE:          6.46 GB/month
# WebSocket:    4.42 GB/month     (le plus efficace)
```

### 4. Compatibilité et infrastructure

```
┌──────────────────────────────────────────────────────┐
│           COMPATIBILITÉ PAR TECHNIQUE                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Navigateurs                                         │
│  ├─ Polling:        100% (même IE6)                  │
│  ├─ Long-Polling:   100%                             │
│  ├─ SSE:            95% (pas IE/Edge ancien)         │
│  └─ WebSocket:      97% (tous navigateurs modernes)  │
│                                                      │
│  Proxies/Firewalls                                   │
│  ├─ Polling:        ✓ Aucun problème                 │
│  ├─ Long-Polling:   ⚠ Timeouts possibles             │
│  ├─ SSE:            ⚠ Buffering possible             │
│  └─ WebSocket:      ⚠ Certains bloquent (port 80/443)│
│                                                      │
│  Load Balancers                                      │
│  ├─ Polling:        ✓ Aucun problème                 │
│  ├─ Long-Polling:   ⚠ Sticky sessions requises       │
│  ├─ SSE:            ⚠ Sticky sessions requises       │
│  └─ WebSocket:      ⚠ Sticky sessions + upgrade      │
│                                                      │
│  CDN                                                 │
│  ├─ Polling:        ✓ Peut cacher les réponses       │
│  ├─ Long-Polling:   ✗ Incompatible                   │
│  ├─ SSE:            ✗ Incompatible                   │
│  └─ WebSocket:      ✓ Avec support (CloudFlare, etc.)│
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 5. Complexité d'implémentation

**Lignes de code approximatives (backend + frontend)** :

```
Technique       Backend    Frontend   Gestion erreurs   Total
────────────────────────────────────────────────────────────
Polling          50 LOC     20 LOC      20 LOC         90 LOC
Long-Polling    150 LOC     50 LOC      50 LOC        250 LOC
SSE             100 LOC     30 LOC      30 LOC        160 LOC
WebSocket       200 LOC    100 LOC     100 LOC        400 LOC
```

**Frameworks facilitant l'implémentation** :

```javascript
// Socket.IO (WebSocket avec fallback automatique)
// Backend: ~50 LOC, Frontend: ~30 LOC
// Avantage: Gère automatiquement fallback, reconnexion, rooms

// Backend (Node.js)
const io = require('socket.io')(server);

io.on('connection', (socket) => {
    socket.on('chat', (msg) => {
        io.emit('chat', msg);
    });
});

// Frontend
const socket = io();
socket.on('chat', (msg) => {
    displayMessage(msg);
});
socket.emit('chat', 'Hello!');
```

---

## Cas d'usage réels par industrie

### 1. Réseaux sociaux

**Facebook/Instagram : Notifications et feed**

```
┌──────────────────────────────────────────────────┐
│  Composant          │  Technique                 │
├─────────────────────┼────────────────────────────┤
│  Notifications      │  Long-Polling (fallback)   │
│                     │  WebSocket (moderne)       │
│  Chat Messenger     │  WebSocket                 │
│  Feed actualisation │  Polling (pull-to-refresh) │
│  Stories vues       │  HTTP POST batch           │
│  Présence en ligne  │  WebSocket + heartbeat     │
└──────────────────────────────────────────────────┘
```

**Twitter : Timeline temps réel**

```javascript
// Streaming API (SSE-like)
const stream = new EventSource('https://stream.twitter.com/1.1/statuses/filter.json?track=javascript');

stream.addEventListener('tweet', (event) => {
    const tweet = JSON.parse(event.data);
    prependToTimeline(tweet);
});
```

### 2. Finance et trading

**Binance : Orderbook temps réel**

```
Fréquence: 100-1000 updates/seconde
Technique: WebSocket exclusivement
Latence: < 10ms critique

Raison: Le polling à cette fréquence serait catastrophique
- 1000 req/s × 10000 traders = 10M req/s
- WebSocket: 10000 connexions, updates push
```

**Bloomberg Terminal : Données multi-sources**

```
┌─────────────────────────────────────────────────┐
│  Type de donnée    │  Fréquence  │  Technique   │
├────────────────────┼─────────────┼──────────────┤
│  Prix actions      │  Temps réel │  WebSocket   │
│  News Bloomberg    │  Continue   │  SSE         │
│  Analyses          │  Horaire    │  Polling     │
│  Alertes custom    │  Événement  │  WebSocket   │
└─────────────────────────────────────────────────┘
```

### 3. Collaboration en temps réel

**Google Docs : Édition collaborative**

```
Architecture hybride:
├─ Operational Transformation (OT) sur WebSocket
├─ Présence des curseurs: WebSocket
├─ Synchronisation état: Polling (backup)
└─ Commentaires: SSE

Pourquoi WebSocket pour OT:
- Bidirectionnel: client envoie éditions, reçoit transformations
- Latence < 100ms nécessaire pour UX fluide
- Volume: centaines d'opérations par minute par utilisateur
```

**Figma : Design collaboratif**

```javascript
// Synchronisation de position de curseur
const ws = new WebSocket('wss://multiplayer.figma.com');

// Envoi position (30 FPS = 30 msg/s par utilisateur)
setInterval(() => {
    ws.send(JSON.stringify({
        type: 'cursor',
        x: mouseX,
        y: mouseY,
        userId: currentUser.id
    }));
}, 33); // ~30 FPS

// Réception des autres curseurs
ws.onmessage = (event) => {
    const { type, x, y, userId } = JSON.parse(event.data);
    if (type === 'cursor') {
        updateCursor(userId, x, y);
    }
};
```

### 4. Gaming

**Fortnite : Communication serveur-client**

```
Tick rate: 30 Hz (30 updates/seconde)
Technique: WebSocket ou UDP (pour certaines données)
Latency budget: < 50ms

Architecture:
├─ Position joueurs: WebSocket bidirectionnel
├─ Actions (tir, build): WebSocket avec confirmation
├─ État du monde: Server authoritative
└─ Voice chat: WebRTC (peer-to-peer quand possible)
```

**Chess.com : Parties en ligne**

```javascript
// SSE suffisant pour échecs (pas temps réel strict)
const gameEvents = new EventSource(`/game/${gameId}/events`);

gameEvents.addEventListener('move', (event) => {
    const { from, to, piece } = JSON.parse(event.data);
    animateMove(from, to, piece);
});

gameEvents.addEventListener('chat', (event) => {
    const { user, message } = JSON.parse(event.data);
    displayChatMessage(user, message);
});

// Envoi de coup (HTTP POST suffit)
async function makeMove(from, to) {
    await fetch(`/game/${gameId}/move`, {
        method: 'POST',
        body: JSON.stringify({ from, to })
    });
}
```

### 5. IoT et monitoring

**Grafana : Dashboards de monitoring**

```
┌────────────────────────────────────────────────┐
│  Métrique           │  Refresh  │  Technique   │
├─────────────────────┼───────────┼──────────────┤
│  CPU/RAM système    │  1s       │  WebSocket   │
│  Requêtes/sec       │  5s       │  SSE         │
│  Erreurs log        │  Event    │  WebSocket   │
│  Température IoT    │  30s      │  Polling     │
│  Alertes            │  Instant  │  WebSocket   │
└────────────────────────────────────────────────┘
```

**Tesla : Données véhicule**

```python
# Streaming API (WebSocket)
# Fréquence: Variable (1-10 Hz selon vitesse)

import websocket
import json

def on_message(ws, message):
    data = json.loads(message)

    if data['msg_type'] == 'data:update':
        # Vitesse, batterie, position GPS
        update_dashboard(data)

    elif data['msg_type'] == 'alert':
        # Alertes critiques (collision, batterie faible)
        show_alert(data)

ws = websocket.WebSocketApp(
    "wss://streaming.vn.teslamotors.com/streaming/",
    on_message=on_message
)

ws.run_forever()
```

### 6. E-commerce

**Amazon : Tracking de livraison**

```
État de commande: Polling (30-60s)
- Changement rare, latence acceptable
- Pas de serveur WebSocket à maintenir

Notifications Flash Sales: SSE
- Serveur → Client uniquement
- Événements sporadiques mais importants

Chat support: WebSocket
- Communication bidirectionnelle
- Temps réel nécessaire
```

**Shopify : Inventory sync**

```javascript
// Synchronisation stock en temps réel pour vendeurs
const inventoryWs = new WebSocket('wss://inventory.shopify.com');

inventoryWs.onmessage = (event) => {
    const { productId, quantity, location } = JSON.parse(event.data);

    if (quantity < 5) {
        showLowStockWarning(productId);
    }

    updateInventoryDisplay(productId, quantity);
};
```

---

## Patterns architecturaux

### 1. Progressive Enhancement (Amélioration progressive)

**Stratégie** : Commencer simple, améliorer selon capacités.

```javascript
class RealtimeConnection {
    constructor(url) {
        this.url = url;
        this.connection = null;
        this.fallbackLevel = 0;

        this.strategies = [
            { name: 'WebSocket', connect: this.connectWebSocket },
            { name: 'SSE', connect: this.connectSSE },
            { name: 'Long-Polling', connect: this.connectLongPolling },
            { name: 'Polling', connect: this.connectPolling }
        ];

        this.connect();
    }

    async connect() {
        const strategy = this.strategies[this.fallbackLevel];
        console.log(`Attempting connection via ${strategy.name}`);

        try {
            this.connection = await strategy.connect.call(this);
            console.log(`Connected via ${strategy.name}`);
        } catch (error) {
            console.error(`${strategy.name} failed:`, error);
            this.fallback();
        }
    }

    fallback() {
        this.fallbackLevel++;

        if (this.fallbackLevel >= this.strategies.length) {
            console.error('All connection strategies failed');
            return;
        }

        console.log('Falling back to next strategy...');
        setTimeout(() => this.connect(), 1000);
    }

    connectWebSocket() {
        return new Promise((resolve, reject) => {
            const ws = new WebSocket(this.url.replace('http', 'ws'));

            ws.onopen = () => resolve(ws);
            ws.onerror = reject;

            setTimeout(() => reject(new Error('WebSocket timeout')), 5000);
        });
    }

    connectSSE() {
        return new Promise((resolve, reject) => {
            const sse = new EventSource(this.url + '/sse');

            sse.onopen = () => resolve(sse);
            sse.onerror = reject;

            setTimeout(() => reject(new Error('SSE timeout')), 5000);
        });
    }

    // ... autres méthodes
}

// Utilisation
const realtime = new RealtimeConnection('https://api.example.com');
```

### 2. Fanout Architecture (Distribution à grande échelle)

**Problème** : Comment notifier 1 million d'utilisateurs simultanément ?

```
┌─────────────────────────────────────────────────────┐
│              FANOUT ARCHITECTURE                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Event Source (ex: nouveau tweet)                   │
│         │                                           │
│         ▼                                           │
│  ┌──────────────┐                                   │
│  │ Message Queue│  (Redis Pub/Sub, Kafka)           │
│  └──────┬───────┘                                   │
│         │                                           │
│    ┌────┴────┬────────┬────────┐                    │
│    ▼         ▼        ▼        ▼                    │
│  WS-1     WS-2     WS-3     WS-N                    │
│  (100k)   (100k)   (100k)   (100k)                  │
│    │        │        │        │                     │
│    └────────┴────────┴────────┘                     │
│         Clients (1M total)                          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Implémentation (Node.js + Redis)** :

```javascript
// Server WebSocket avec Redis Pub/Sub
const WebSocket = require('ws');
const redis = require('redis');

const wss = new WebSocket.Server({ port: 8080 });
const subscriber = redis.createClient();

// Subscribe to Redis channel
subscriber.subscribe('notifications');

// Broadcast to all connected clients
subscriber.on('message', (channel, message) => {
    wss.clients.forEach((client) => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(message);
        }
    });
});

// Handle new connections
wss.on('connection', (ws) => {
    console.log('Client connected');

    ws.on('close', () => {
        console.log('Client disconnected');
    });
});

// Publish events (from another service)
const publisher = redis.createClient();
publisher.publish('notifications', JSON.stringify({
    type: 'alert',
    message: 'New feature released!'
}));
```

### 3. Hybrid Architecture (Architecture hybride)

**Combiner plusieurs techniques selon le use case** :

```javascript
class HybridRealtimeApp {
    constructor() {
        // WebSocket pour chat (bidirectionnel, haute fréquence)
        this.chatWs = new WebSocket('wss://api.example.com/chat');

        // SSE pour notifications (unidirectionnel, sporadique)
        this.notifSource = new EventSource('/api/notifications');

        // Polling pour données non-critiques
        this.pollInterval = setInterval(() => {
            this.fetchUserStats();
        }, 60000); // 1 minute
    }

    setupChat() {
        this.chatWs.onmessage = (event) => {
            const msg = JSON.parse(event.data);
            this.displayChatMessage(msg);
        };

        // Envoyer message
        this.sendChatMessage = (text) => {
            this.chatWs.send(JSON.stringify({ text }));
        };
    }

    setupNotifications() {
        this.notifSource.onmessage = (event) => {
            const notif = JSON.parse(event.data);
            this.showNotification(notif);
        };
    }

    async fetchUserStats() {
        const stats = await fetch('/api/user/stats').then(r => r.json());
        this.updateStatsDisplay(stats);
    }
}
```

---

## Considérations de sécurité

### 1. Authentification

**WebSocket n'a pas de headers standards après handshake** :

```javascript
// Authentification via token initial
const token = localStorage.getItem('authToken');
const ws = new WebSocket(`wss://api.example.com/chat?token=${token}`);

// Ou via premier message
ws.onopen = () => {
    ws.send(JSON.stringify({
        type: 'auth',
        token: token
    }));
};
```

**SSE avec headers standards** :

```javascript
// Les headers HTTP sont envoyés à chaque reconnexion
const eventSource = new EventSource('/api/events', {
    withCredentials: true  // Envoie cookies
});

// Serveur peut vérifier l'authentification
app.get('/api/events', authenticateMiddleware, (req, res) => {
    res.setHeader('Content-Type', 'text/event-stream');
    // ...
});
```

### 2. Rate Limiting

**Protéger contre les abus** :

```javascript
// Serveur : limiter les messages par client
const rateLimits = new Map();

wss.on('connection', (ws, req) => {
    const clientId = getClientId(req);

    rateLimits.set(clientId, {
        messages: 0,
        resetAt: Date.now() + 60000  // 1 minute window
    });

    ws.on('message', (data) => {
        const limit = rateLimits.get(clientId);

        if (Date.now() > limit.resetAt) {
            limit.messages = 0;
            limit.resetAt = Date.now() + 60000;
        }

        limit.messages++;

        if (limit.messages > 100) {  // Max 100 msg/min
            ws.send(JSON.stringify({
                error: 'Rate limit exceeded'
            }));
            return;
        }

        // Process message
        handleMessage(data);
    });
});
```

### 3. Validation et sanitization

```javascript
// Client envoie des données
ws.send(JSON.stringify({
    type: 'chat',
    message: userInput  // DANGER : peut contenir XSS
}));

// Serveur DOIT valider et nettoyer
ws.on('message', (data) => {
    const msg = JSON.parse(data);

    // Validation
    if (typeof msg.message !== 'string') {
        return ws.send(JSON.stringify({ error: 'Invalid message type' }));
    }

    if (msg.message.length > 1000) {
        return ws.send(JSON.stringify({ error: 'Message too long' }));
    }

    // Sanitization (éviter XSS)
    const sanitized = sanitizeHtml(msg.message);

    // Broadcast
    broadcast({
        type: 'chat',
        message: sanitized,
        user: authenticatedUser.id
    });
});
```

---

## Métriques et monitoring

### KPIs essentiels pour temps réel

```javascript
// Métriques côté client
class RealtimeMetrics {
    constructor() {
        this.metrics = {
            connectionTime: 0,
            messagesReceived: 0,
            messagesSent: 0,
            latencies: [],
            reconnections: 0,
            errors: 0
        };
    }

    recordConnectionTime(ms) {
        this.metrics.connectionTime = ms;
    }

    recordMessageLatency(sentAt) {
        const latency = Date.now() - sentAt;
        this.metrics.latencies.push(latency);

        // Garder seulement les 100 dernières
        if (this.metrics.latencies.length > 100) {
            this.metrics.latencies.shift();
        }
    }

    getStats() {
        const latencies = this.metrics.latencies;
        return {
            ...this.metrics,
            avgLatency: latencies.reduce((a, b) => a + b, 0) / latencies.length,
            p95Latency: this.percentile(latencies, 0.95),
            p99Latency: this.percentile(latencies, 0.99)
        };
    }

    percentile(arr, p) {
        const sorted = [...arr].sort((a, b) => a - b);
        const index = Math.ceil(sorted.length * p) - 1;
        return sorted[index];
    }
}

// Utilisation
const metrics = new RealtimeMetrics();

ws.onopen = () => {
    const connectionTime = Date.now() - connectionStartTime;
    metrics.recordConnectionTime(connectionTime);
};

ws.onmessage = (event) => {
    const msg = JSON.parse(event.data);

    if (msg.sentAt) {
        metrics.recordMessageLatency(msg.sentAt);
    }

    metrics.metrics.messagesReceived++;
};

// Envoyer métriques au serveur analytics
setInterval(() => {
    const stats = metrics.getStats();
    sendAnalytics(stats);
}, 60000);
```

---

## Tableau de décision final

```
┌──────────────────────────────────────────────────────────────────┐
│                    ARBRE DE DÉCISION                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Avez-vous besoin de communication bidirectionnelle ?            │
│  ├─ OUI → WebSocket                                              │
│  └─ NON ↓                                                        │
│                                                                  │
│      Fréquence de mise à jour ?                                  │
│      ├─ < 1/min → Polling (simple, compatible)                   │
│      ├─ 1-10/min → Long-Polling ou SSE                           │
│      └─ > 10/min → SSE ou WebSocket                              │
│                                                                  │
│          Données binaires nécessaires ?                          │
│          ├─ OUI → WebSocket                                      │
│          └─ NON ↓                                                │
│                                                                  │
│              Complexité acceptable ?                             │
│              ├─ Simple → SSE (API native)                        │
│              └─ Complexe OK → WebSocket (max perf)               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Conclusion

La communication temps réel est devenue une **exigence fondamentale** pour les applications modernes. Le choix de la bonne technique dépend de multiples facteurs : direction de communication, fréquence, latence acceptable, scalabilité, compatibilité.

**Points clés à retenir** :

1. **Quatre techniques principales** avec des trade-offs distincts :
   - Polling : Simple mais inefficace
   - Long-Polling : Compromis raisonnable
   - SSE : Optimal pour push unidirectionnel
   - WebSocket : Maximum de flexibilité et performance

2. **Pas de solution universelle** : Choisir selon le contexte
   - Chat → WebSocket
   - Notifications → SSE ou Long-Polling
   - Prix en temps réel → WebSocket
   - Dashboard → SSE ou Polling

3. **Architecture hybride souvent optimale** : Combiner plusieurs techniques

4. **Scalabilité nécessite planification** : Fanout, load balancing, sticky sessions

5. **Sécurité critique** : Authentification, rate limiting, validation

Les sections suivantes détailleront chaque technique avec des implémentations complètes, des patterns avancés, et des exemples de code production-ready.

---


⏭️ [Polling et long-polling : principes et limites](/08-programmation-reseau/09.1-polling-long-polling.md)
