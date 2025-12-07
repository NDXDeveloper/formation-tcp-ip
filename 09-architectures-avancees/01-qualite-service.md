🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.1 Qualité de Service (QoS)

## Introduction

La **Qualité de Service (QoS)** est l'ensemble des techniques permettant de **prioriser certains types de trafic réseau** par rapport à d'autres. Dans un monde idéal, la bande passante serait illimitée et tous les paquets arriveraient instantanément. Dans la réalité, les ressources réseau sont limitées et doivent être partagées équitablement... ou plutôt, **inéquitablement** selon les besoins métier.

### Analogie simple : l'autoroute

Imaginez une autoroute à trois voies :
- **Voie de gauche** : ambulances et pompiers (trafic prioritaire)
- **Voie du milieu** : voitures normales (trafic standard)
- **Voie de droite** : poids lourds (trafic best-effort)

La QoS fait la même chose pour le trafic réseau : elle crée des "voies" virtuelles et décide qui peut emprunter quelle voie.

## Pourquoi la QoS est-elle nécessaire ?

### Le problème fondamental : tous les paquets ne sont pas égaux

Considérez ces trois scénarios simultanés sur le même réseau :

```
┌─────────────────────────────────────────────────────┐
│  Réseau d'entreprise (1 Gbps partagé)               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  [1] Visioconférence CEO-Investisseurs              │
│      → Besoin : faible latence (<50ms)              │
│      → Débit : 2 Mbps                               │
│      → Sensibilité : CRITIQUE (jitter mortel)       │
│                                                     │
│  [2] Téléchargement backup nocturne                 │
│      → Besoin : haut débit                          │
│      → Débit : 800 Mbps                             │
│      → Sensibilité : FAIBLE (peut attendre)         │
│                                                     │
│  [3] Navigation web employés                        │
│      → Besoin : latence acceptable (<200ms)         │
│      → Débit : variable (50-200 Mbps)               │
│      → Sensibilité : MOYENNE                        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Sans QoS :** Le backup sature le lien, la visio devient inutilisable (saccades, coupures), les investisseurs partent. Coût : plusieurs millions d'euros.

**Avec QoS :** La visio est priorisée, utilise ses 2 Mbps garantis, le backup utilise le reste, tout le monde est content.

### Les types de trafic et leurs exigences

| Type de trafic | Latence tolérée | Jitter toléré | Perte tolérée | Exemples |
|----------------|----------------|---------------|---------------|----------|
| **VoIP / Vidéo** | < 150ms | < 30ms | < 1% | Zoom, Teams, appels SIP |
| **Gaming** | < 50ms | < 20ms | < 0.5% | FPS multijoueur, cloud gaming |
| **Streaming vidéo** | < 2s (buffer) | Variable | < 0.1% | Netflix, YouTube |
| **Transactions financières** | < 10ms | < 5ms | 0% | Trading HFT, blockchain |
| **Web interactif** | < 200ms | Non critique | < 2% | APIs REST, WebSockets |
| **Email** | < 5s | Non critique | 0% | SMTP, IMAP |
| **Backup / Transferts** | Non critique | Non critique | 0% | rsync, FTP, cloud sync |

## Les mécanismes de QoS

La QoS repose sur **quatre piliers fondamentaux** :

### 1. Classification

**Identifier** le type de trafic pour lui appliquer le bon traitement.

#### Méthodes de classification

**a) Par adresse IP/port**
```
Règle : Si (IP src = 10.0.1.50 ET port dest = 3478-3497)
       → Marquer comme "VoIP" (DSCP EF)
```

**b) Par protocole**
```
Règle : Si (protocole = SIP ou RTP)
       → Marquer comme "VoIP"
```

**c) Par inspection profonde (DPI)**
```
Règle : Si (payload contient signature Zoom)
       → Marquer comme "Vidéoconférence"
```

**d) Par marquage applicatif (DSCP)**
L'application elle-même marque ses paquets (le plus fiable).

#### Exemple réel : classifier du trafic Kubernetes

```yaml
# NetworkPolicy avec classification QoS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: prioritize-payment-service
spec:
  podSelector:
    matchLabels:
      app: payment-processor
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: bank-api
    ports:
    - protocol: TCP
      port: 8443
    # Annotation pour QoS (CNI-specific)
    annotations:
      qos.class: "guaranteed"
      dscp.mark: "EF"  # Expedited Forwarding
```

### 2. Marquage (Marking)

**Étiqueter** les paquets avec une priorité dans les headers IP.

#### DSCP (Differentiated Services Code Point)

Le champ **DSCP** utilise **6 bits** du champ ToS (Type of Service) dans l'en-tête IPv4/IPv6.

```
En-tête IPv4 (simplifié)
┌────────┬────────┬─────────────────┬──────────┐
│Version │  IHL   │   ToS / DSCP    │  Length  │
├────────┼────────┼─────────────────┼──────────┤
│   4    │   5    │  6 bits (DSCP)  │   ...    │
└────────┴────────┴─────────────────┴──────────┘
                   └─ 2 bits ECN ───┘
```

#### Valeurs DSCP standard

| DSCP | Binaire | Décimal | Nom | Usage typique |
|------|---------|---------|-----|---------------|
| **EF** | 101110 | 46 | Expedited Forwarding | VoIP, vidéo temps réel |
| **AF41** | 100010 | 34 | Assured Forwarding 4.1 | Vidéo streaming |
| **AF31** | 011010 | 26 | AF 3.1 | Signalisation (SIP) |
| **AF21** | 010010 | 18 | AF 2.1 | Données importantes |
| **AF11** | 001010 | 10 | AF 1.1 | Données standard |
| **BE** | 000000 | 0 | Best Effort | Trafic par défaut |

#### Exemple : Marquer le trafic en Python

```python
import socket
import struct

def create_socket_with_qos(dscp_value=46):
    """
    Crée un socket UDP avec marquage QoS (DSCP EF = 46)
    Utilisé pour VoIP ou trafic temps réel critique
    """
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # Calculer la valeur ToS (DSCP << 2)
    tos_value = dscp_value << 2

    # Appliquer le marquage IP_TOS (Linux/Unix)
    try:
        sock.setsockopt(socket.IPPROTO_IP, socket.IP_TOS, tos_value)
        print(f"Socket marqué avec DSCP {dscp_value} (ToS {tos_value})")
    except PermissionError:
        print("⚠️  Besoin de CAP_NET_ADMIN ou root pour marquer QoS")

    return sock

# Exemple d'utilisation pour une app VoIP
voip_socket = create_socket_with_qos(dscp_value=46)  # EF
voip_socket.sendto(b"Audio packet", ("10.0.1.100", 5004))
```

#### Exemple : Marquer en Go (WebRTC)

```go
package main

import (
    "net"
    "syscall"
)

func createQoSConnection(dscpValue int) (*net.UDPConn, error) {
    addr := &net.UDPAddr{Port: 0}
    conn, err := net.ListenUDP("udp", addr)
    if err != nil {
        return nil, err
    }

    // Récupérer le file descriptor
    file, _ := conn.File()
    fd := int(file.Fd())

    // Calculer ToS (DSCP << 2)
    tosValue := dscpValue << 2

    // Appliquer IP_TOS (Unix)
    err = syscall.SetsockoptInt(
        fd,
        syscall.IPPROTO_IP,
        syscall.IP_TOS,
        tosValue,
    )

    if err != nil {
        return nil, err
    }

    return conn, nil
}

// Usage dans une application WebRTC
func main() {
    // Marquer avec DSCP EF (46) pour audio
    audioConn, _ := createQoSConnection(46)

    // Marquer avec DSCP AF41 (34) pour vidéo
    videoConn, _ := createQoSConnection(34)

    // Vos flux RTP utilisent ces connexions
    // ...
}
```

### 3. Queuing (Files d'attente)

**Organiser** les paquets dans différentes files selon leur priorité.

#### Algorithmes de queuing courants

**a) FIFO (First In, First Out)**
```
┌────────────────────────┐
│  [P1][P2][P3][P4][P5]  │ → Sortie
└────────────────────────┘
Simple mais aucune priorisation
```

**b) Priority Queuing (PQ)**
```
Queue Haute    [VoIP1][VoIP2]──────────┐
Queue Moyenne  [Web1][Web2][Web3]──────┤→ Sortie
Queue Basse    [Backup1][Backup2]──────┘
                ↑
             Toujours servie en premier
```

**c) Weighted Fair Queuing (WFQ)**
```
Queue 1 (VoIP)    [P1][P2]        → Poids 50%
Queue 2 (Web)     [P3][P4][P5]    → Poids 30%
Queue 3 (Backup)  [P6][P7][P8][P9]→ Poids 20%

Sortie : P1, P3, P6, P2, P4, P7, P5, P8, P9
         └─ Proportionnel aux poids ──┘
```

**d) Class-Based Weighted Fair Queuing (CBWFQ)**
Le plus utilisé en production : combine priorité stricte + partage pondéré.

```
┌──────────────────────────────────────┐
│  Priority Queue (Strict)             │
│  [VoIP] → 10% garanti, servi d'abord │
├──────────────────────────────────────┤
│  Class 1 (Business Apps) → 40%       │
│  Class 2 (Web) → 30%                 │
│  Class 3 (Backup) → 20%              │
│  Default → Best effort               │
└──────────────────────────────────────┘
```

#### Exemple : Configuration CBWFQ (Cisco-style)

```
! Définir les classes de trafic
class-map match-any VOIP
  match dscp ef

class-map match-any BUSINESS-CRITICAL
  match dscp af41 af42 af43

class-map match-any WEB
  match dscp af21 af22 af23

! Définir la politique QoS
policy-map WAN-QOS
  class VOIP
    priority percent 10
    set dscp ef

  class BUSINESS-CRITICAL
    bandwidth percent 40
    random-detect dscp-based

  class WEB
    bandwidth percent 30

  class class-default
    bandwidth percent 20
    random-detect

! Appliquer sur l'interface WAN sortante
interface GigabitEthernet0/0
  service-policy output WAN-QOS
```

### 4. Shaping et Policing

**Contrôler** le débit pour éviter la congestion ou faire respecter des quotas.

#### Traffic Shaping (lissage)

**Buffer les paquets** pour respecter un débit maximal. Les paquets excédentaires sont **mis en attente**.

```
Entrée (rafales) :  ▁▁████▁▁▁██████▁▁▁
                     ↓ Shaping à 10 Mbps
Sortie (lissée)  :  ▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂▂
```

**Algorithme Token Bucket**
```
Seau de tokens
┌─────────────┐
│ ●●●●●●●     │ ← Remplissage à taux constant (CIR)
│             │
└─────────────┘
      ↓
  1 token = 1 paquet autorisé

Si tokens disponibles → transmettre
Sinon → bufferiser (shaping) ou drop (policing)
```

#### Traffic Policing (contrôle strict)

**Dropper ou remarquer** les paquets excédentaires. Pas de buffer.

```
Trafic conforme  : Transmis normalement
Trafic excédant  : Droppé (hard policing)
                   ou remarqué DSCP plus bas (soft policing)
```

#### Exemple : Shaping avec Linux TC (Traffic Control)

```bash
#!/bin/bash
# Limiter la bande passante d'une application à 10 Mbps

INTERFACE="eth0"
APP_IP="192.168.1.100"
MAX_RATE="10mbit"

# Créer une qdisc HTB (Hierarchical Token Bucket)
tc qdisc add dev $INTERFACE root handle 1: htb default 10

# Classe pour le trafic shapé
tc class add dev $INTERFACE parent 1: classid 1:1 htb rate $MAX_RATE ceil $MAX_RATE

# Filtrer le trafic de l'app
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 1 \
    u32 match ip dst $APP_IP flowid 1:1

echo "Traffic shaping appliqué : $APP_IP limité à $MAX_RATE"
```

#### Exemple : Rate limiting applicatif (Node.js API)

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();

// QoS applicative : limiter les requêtes par IP
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requêtes max par fenêtre
  message: {
    error: "Trop de requêtes, réessayez dans 15 minutes",
    qos: "rate_limit_exceeded"
  },

  // Prioriser les utilisateurs premium
  skip: (req) => {
    return req.user && req.user.tier === 'premium';
  },

  // Headers de QoS
  standardHeaders: true,
  legacyHeaders: false,
});

// Appliquer sur les routes publiques
app.use('/api/public/', apiLimiter);

// Route premium sans limite
app.use('/api/premium/', (req, res, next) => {
  if (req.user.tier !== 'premium') {
    return res.status(403).json({ error: 'Premium required' });
  }
  next();
});
```

## Les modèles de QoS

### 1. IntServ (Integrated Services)

**Principe :** Réservation de ressources **pour chaque flux** individuellement.

**Protocole :** RSVP (Resource Reservation Protocol)

```
Client ──[RSVP PATH]──→ Routeur 1 ──→ Routeur 2 ──→ Serveur
                            │             │
                            ↓             ↓
                        Réserve      Réserve
                        bande        bande
                        passante     passante

Serveur ──[RSVP RESV]──→ Routeur 2 ──→ Routeur 1 ──→ Client
                       (confirmation de réservation)
```

**Avantages :**
- Garanties strictes par flux
- Adapté au trafic temps réel

**Inconvénients :**
- **Ne scale pas** : chaque routeur doit maintenir l'état de chaque flux
- Complexité de signalisation
- **Abandonné en pratique** sur Internet

**Cas d'usage :** Réseaux privés avec peu de flux (< 1000), réseaux industriels SCADA.

### 2. DiffServ (Differentiated Services)

**Principe :** Classification en **classes de service**, pas de réservation par flux.

**Mécanisme :** Marquage DSCP + traitement différencié par classe.

```
Tous les paquets VoIP → Classe EF → Queue prioritaire
Tous les paquets Web  → Classe AF → Queue standard
Tous les backups      → Classe BE → Queue basse priorité
```

**Avantages :**
- **Scale très bien** : état uniquement par classe, pas par flux
- Simple à implémenter
- **Standard de facto** sur Internet

**Inconvénients :**
- Pas de garanties strictes par flux
- Dépend de la confiance dans le marquage

**Cas d'usage :** Internet, entreprises, data centers, cloud.

### Comparaison IntServ vs DiffServ

| Critère | IntServ (RSVP) | DiffServ (DSCP) |
|---------|----------------|-----------------|
| **Granularité** | Par flux | Par classe |
| **État maintenu** | Tous les routeurs | Aucun (stateless) |
| **Scalabilité** | Faible (< 1000 flux) | Excellente (millions) |
| **Garanties** | Strictes | Relatives |
| **Complexité** | Élevée | Faible |
| **Usage réel** | Réseaux privés | Internet, cloud |

## Cas d'usage réels pour développeurs

### Cas 1 : Application de visioconférence (WebRTC)

**Contexte :** Vous développez une plateforme de télémédecine avec vidéo HD.

**Problème :** Les consultations vidéo deviennent inutilisables aux heures de pointe.

**Solution QoS :**

```javascript
// Configuration WebRTC avec QoS DSCP
const peerConnection = new RTCPeerConnection({
  iceServers: [...],

  // Marquage QoS des flux RTP
  rtcpMuxPolicy: 'require',
  bundlePolicy: 'max-bundle',

  // Demander marquage DSCP au navigateur
  // (nécessite que le système supporte QoS)
  encodedInsertableStreams: true
});

// Lors de l'ajout des tracks
const audioTrack = stream.getAudioTracks()[0];
const videoTrack = stream.getVideoTracks()[0];

// Paramètres d'encodage avec priorité
const audioSender = peerConnection.addTrack(audioTrack, stream);
const videoSender = peerConnection.addTrack(videoTrack, stream);

// Configurer les priorités (high, medium, low, very-low)
audioSender.setParameters({
  encodings: [{
    priority: 'high',        // Audio prioritaire
    networkPriority: 'high'  // Demande DSCP EF
  }]
});

videoSender.setParameters({
  encodings: [{
    priority: 'medium',
    networkPriority: 'medium' // Demande DSCP AF41
  }]
});
```

**Configuration réseau côté serveur :**

```bash
# Nginx RTMP avec marquage QoS
# /etc/nginx/nginx.conf

stream {
    # Pool de connexions WebRTC
    upstream webrtc_media {
        server 10.0.1.10:3478;
        server 10.0.1.11:3478;
    }

    server {
        listen 3478 udp;

        # Marquer les paquets sortants avec DSCP EF
        proxy_dscp ef;

        proxy_pass webrtc_media;
        proxy_timeout 10s;
        proxy_bind $remote_addr transparent;
    }
}
```

### Cas 2 : API de trading haute fréquence

**Contexte :** API REST pour passer des ordres boursiers. Chaque milliseconde compte.

**Problème :** Les requêtes critiques sont ralenties par le trafic analytics/logs.

**Solution :**

```python
from flask import Flask, request, g
import socket
import time

app = Flask(__name__)

# Middleware QoS
@app.before_request
def apply_qos():
    """
    Marquer les requêtes critiques avec DSCP approprié
    """
    # Identifier le type de requête
    endpoint = request.endpoint

    if endpoint in ['place_order', 'cancel_order', 'market_data']:
        # Trafic critique → DSCP EF (46)
        g.dscp = 46
        g.priority = 'critical'
    elif endpoint in ['portfolio', 'positions']:
        # Trafic important → DSCP AF41 (34)
        g.dscp = 34
        g.priority = 'high'
    else:
        # Trafic normal → DSCP BE (0)
        g.dscp = 0
        g.priority = 'normal'

    # Logger pour debug
    g.request_start = time.time()

@app.after_request
def log_qos_metrics(response):
    """
    Logger les métriques de latence par classe QoS
    """
    if hasattr(g, 'request_start'):
        latency_ms = (time.time() - g.request_start) * 1000

        # Alerter si latence excessive sur trafic critique
        if g.priority == 'critical' and latency_ms > 50:
            app.logger.warning(
                f"QoS violation: {request.endpoint} took {latency_ms:.2f}ms "
                f"(threshold: 50ms)"
            )

    return response

@app.route('/api/v1/orders', methods=['POST'])
def place_order():
    """
    Endpoint critique : passage d'ordre
    Automatiquement marqué DSCP EF
    """
    order_data = request.json

    # Socket vers le market avec QoS
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        # Marquer avec DSCP stocké dans g
        tos = g.dscp << 2
        s.setsockopt(socket.IPPROTO_IP, socket.IP_TOS, tos)

        s.connect(('market.exchange.com', 9000))
        s.sendall(order_data.encode())

        response = s.recv(1024)

    return {'status': 'executed', 'order_id': '...'}
```

**Configuration Load Balancer (HAProxy) :**

```
# /etc/haproxy/haproxy.cfg

frontend trading_api
    bind *:443 ssl crt /etc/ssl/certs/api.pem

    # ACLs pour classifier le trafic
    acl is_order_endpoint path_beg /api/v1/orders
    acl is_market_data path_beg /api/v1/market
    acl is_analytics path_beg /api/v1/analytics

    # Marquer avec DSCP selon le type
    http-request set-mark 0xB8 if is_order_endpoint      # DSCP EF (46 << 2)
    http-request set-mark 0x88 if is_market_data         # DSCP AF41 (34 << 2)
    http-request set-mark 0x00 if is_analytics           # Best effort

    default_backend trading_servers

backend trading_servers
    balance leastconn
    server srv1 10.0.1.10:8000 check
    server srv2 10.0.1.11:8000 check
```

### Cas 3 : Cluster Kubernetes avec plusieurs services

**Contexte :** Microservices avec différentes priorités (paiement > catalogue > logs).

**Solution avec Calico Network Policies :**

```yaml
---
# QoS Profile : Critical (Payment Service)
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: qos-payment-critical
spec:
  selector: app == 'payment-service'
  order: 100
  types:
  - Egress
  egress:
  - action: Allow
    destination:
      nets:
      - 10.0.0.0/8  # Réseau interne
    metadata:
      annotations:
        qos-class: "critical"
        dscp: "46"  # EF
---
# QoS Profile : High (Order Service)
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: qos-order-high
spec:
  selector: app == 'order-service'
  order: 200
  types:
  - Egress
  egress:
  - action: Allow
    metadata:
      annotations:
        qos-class: "high"
        dscp: "34"  # AF41
---
# QoS Profile : Low (Logging Service)
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: qos-logging-low
spec:
  selector: app == 'logging-aggregator'
  order: 300
  types:
  - Egress
  egress:
  - action: Allow
    metadata:
      annotations:
        qos-class: "low"
        dscp: "10"  # AF11
```

**Configuration du Pod avec QoS Kubernetes natif :**

```yaml
# Guaranteed QoS (plus haute priorité)
apiVersion: v1
kind: Pod
metadata:
  name: payment-processor
  labels:
    app: payment-service
spec:
  containers:
  - name: payment
    image: payment-service:v2
    resources:
      requests:
        memory: "1Gi"
        cpu: "1000m"
      limits:
        memory: "1Gi"      # Égal à request = Guaranteed
        cpu: "1000m"       # Égal à request = Guaranteed
---
# Burstable QoS (priorité moyenne)
apiVersion: v1
kind: Pod
metadata:
  name: catalog-service
spec:
  containers:
  - name: catalog
    image: catalog:v1
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "2Gi"      # > request = Burstable
        cpu: "2000m"
---
# BestEffort QoS (plus basse priorité)
apiVersion: v1
kind: Pod
metadata:
  name: log-aggregator
spec:
  containers:
  - name: fluentd
    image: fluentd:latest
    # Pas de requests/limits = BestEffort
```

### Cas 4 : CDN avec priorités de contenu

**Contexte :** Plateforme de streaming. Les lives doivent être prioritaires sur la VOD.

**Solution avec Cloudflare Workers + QoS :**

```javascript
// Cloudflare Worker pour router avec QoS

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)
  const path = url.pathname

  // Classifier le contenu
  let qosClass = 'normal'
  let cacheTtl = 3600

  if (path.startsWith('/live/')) {
    qosClass = 'critical'
    cacheTtl = 5  // Cache très court pour live

    // Headers QoS
    const qosHeaders = new Headers(request.headers)
    qosHeaders.set('X-QoS-Class', 'critical')
    qosHeaders.set('X-Priority', 'u=0, i')  // HTTP/2 priority

    // Requête vers origin avec priorité
    return fetch(request, {
      headers: qosHeaders,
      cf: {
        cacheEverything: true,
        cacheTtl: cacheTtl,
        polish: 'off',  // Pas de compression pour live
        minify: { javascript: false, css: false, html: false }
      }
    })
  }

  else if (path.startsWith('/vod/')) {
    qosClass = 'normal'
    cacheTtl = 86400  // Cache 24h pour VOD
  }

  else if (path.startsWith('/api/')) {
    qosClass = 'high'
    cacheTtl = 0  // Pas de cache pour API
  }

  // Fetch avec paramètres appropriés
  return fetch(request, {
    cf: {
      cacheEverything: path.startsWith('/vod/'),
      cacheTtl: cacheTtl
    }
  })
}
```

## Implémentation pratique selon l'environnement

### Sur Linux (TC - Traffic Control)

```bash
#!/bin/bash
# Script complet de QoS pour serveur Linux

INTERFACE="eth0"

# 1. Supprimer les règles existantes
tc qdisc del dev $INTERFACE root 2>/dev/null

# 2. Créer qdisc racine HTB
tc qdisc add dev $INTERFACE root handle 1: htb default 30

# 3. Définir la bande passante totale
tc class add dev $INTERFACE parent 1: classid 1:1 htb rate 1gbit

# 4. Classes de trafic
# Classe 1:10 - VoIP/Vidéo (priorité stricte, 20%)
tc class add dev $INTERFACE parent 1:1 classid 1:10 \
    htb rate 200mbit ceil 300mbit prio 1

# Classe 1:20 - Business Critical (40%)
tc class add dev $INTERFACE parent 1:1 classid 1:20 \
    htb rate 400mbit ceil 600mbit prio 2

# Classe 1:30 - Normal (30%, default)
tc class add dev $INTERFACE parent 1:1 classid 1:30 \
    htb rate 300mbit ceil 500mbit prio 3

# Classe 1:40 - Bulk/Backup (10%)
tc class add dev $INTERFACE parent 1:1 classid 1:40 \
    htb rate 100mbit ceil 200mbit prio 4

# 5. Ajouter des qdiscs feuilles (avec gestion de congestion)
tc qdisc add dev $INTERFACE parent 1:10 handle 10: sfq perturb 10
tc qdisc add dev $INTERFACE parent 1:20 handle 20: sfq perturb 10
tc qdisc add dev $INTERFACE parent 1:30 handle 30: sfq perturb 10
tc qdisc add dev $INTERFACE parent 1:40 handle 40: sfq perturb 10

# 6. Filtres pour classifier le trafic
# VoIP (ports RTP: 16384-32767, signalisation SIP: 5060)
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 1 u32 \
    match ip dport 5060 0xffff flowid 1:10
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 1 u32 \
    match ip dport 16384 0xc000 flowid 1:10

# Business apps (SSH, HTTPS vers serveurs critiques)
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 2 u32 \
    match ip dst 10.0.1.0/24 \
    match ip dport 443 0xffff flowid 1:20

# Bulk (rsync, backups)
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 4 u32 \
    match ip dport 873 0xffff flowid 1:40  # rsync

# Basé sur DSCP (si marqué par l'application)
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 1 u32 \
    match ip tos 0xb8 0xfc flowid 1:10  # DSCP EF

echo "QoS configurée avec succès sur $INTERFACE"
tc -s qdisc show dev $INTERFACE
```

### Sur Docker / Kubernetes

#### Docker avec limitation réseau

```yaml
# docker-compose.yml avec QoS
version: '3.8'

services:
  payment-api:
    image: payment:latest
    networks:
      app_network:
        priority: 1000  # Plus haute priorité
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 2G

  catalog-api:
    image: catalog:latest
    networks:
      app_network:
        priority: 500  # Priorité moyenne
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 2G

  logging:
    image: fluentd:latest
    networks:
      app_network:
        priority: 100  # Basse priorité
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1G

networks:
  app_network:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.enable_ip_masquerade: 'true'
```

### Sur cloud providers

#### AWS - Traffic Mirroring avec priorités

```python
import boto3

ec2 = boto3.client('ec2')

# Créer un filtre de traffic mirroring pour QoS monitoring
response = ec2.create_traffic_mirror_filter(
    Description='QoS monitoring filter',
    TagSpecifications=[{
        'ResourceType': 'traffic-mirror-filter',
        'Tags': [{'Key': 'Purpose', 'Value': 'QoS-Monitoring'}]
    }]
)

filter_id = response['TrafficMirrorFilter']['TrafficMirrorFilterId']

# Règle pour trafic haute priorité (DSCP EF)
ec2.create_traffic_mirror_filter_rule(
    TrafficMirrorFilterId=filter_id,
    TrafficDirection='ingress',
    RuleNumber=100,
    DestinationCidrBlock='0.0.0.0/0',
    SourceCidrBlock='0.0.0.0/0',
    Protocol=6,  # TCP
    Description='High priority traffic (DSCP EF)',
    # Filtrer par DSCP serait dans le target, pas le filter
)
```

## Monitoring et observabilité QoS

### Métriques clés à surveiller

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Métriques QoS
qos_packets_total = Counter(
    'qos_packets_total',
    'Total packets by QoS class',
    ['class', 'direction']
)

qos_latency = Histogram(
    'qos_latency_seconds',
    'Request latency by QoS class',
    ['class', 'endpoint'],
    buckets=[0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0]
)

qos_drops = Counter(
    'qos_packet_drops_total',
    'Dropped packets by QoS class',
    ['class', 'reason']
)

qos_queue_depth = Gauge(
    'qos_queue_depth',
    'Current queue depth by class',
    ['class']
)

# Utilisation dans une API
from flask import Flask, request
from functools import wraps

app = Flask(__name__)

def qos_monitor(qos_class):
    """Decorator pour monitorer QoS"""
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            start = time.time()

            try:
                qos_packets_total.labels(
                    class_=qos_class,
                    direction='ingress'
                ).inc()

                result = f(*args, **kwargs)

                qos_packets_total.labels(
                    class_=qos_class,
                    direction='egress'
                ).inc()

                return result
            finally:
                duration = time.time() - start
                qos_latency.labels(
                    class_=qos_class,
                    endpoint=request.endpoint
                ).observe(duration)

                # Alerter si SLA violé
                if qos_class == 'critical' and duration > 0.050:
                    qos_drops.labels(
                        class_=qos_class,
                        reason='sla_violation'
                    ).inc()

        return wrapper
    return decorator

@app.route('/api/payment')
@qos_monitor('critical')
def process_payment():
    # Logique de paiement
    return {'status': 'ok'}
```

### Dashboard Grafana pour QoS

```yaml
# Requêtes Prometheus pour dashboard QoS
queries:
  # Latence P95 par classe QoS
  - expr: |
      histogram_quantile(0.95,
        sum(rate(qos_latency_seconds_bucket[5m])) by (class, le)
      )
    legend: "P95 Latency - {{class}}"

  # Taux de paquets droppés
  - expr: |
      rate(qos_packet_drops_total[5m])
    legend: "Drop rate - {{class}} - {{reason}}"

  # Profondeur des queues
  - expr: |
      qos_queue_depth
    legend: "Queue depth - {{class}}"

  # Respect des SLAs (% requêtes < threshold)
  - expr: |
      (
        sum(rate(qos_latency_seconds_bucket{class="critical",le="0.05"}[5m]))
        /
        sum(rate(qos_latency_seconds_count{class="critical"}[5m]))
      ) * 100
    legend: "SLA compliance (critical < 50ms)"
```

## Pièges courants et solutions

### Piège 1 : Marquage DSCP ignoré ou écrasé

**Problème :** Vous marquez vos paquets mais ils arrivent en best-effort.

**Causes possibles :**
- Le FAI/opérateur efface les marquages DSCP
- Un routeur intermédiaire reconfigure le DSCP
- Le pare-feu réinitialise les valeurs

**Solution :**
```bash
# Vérifier la préservation DSCP avec tcpdump
tcpdump -i eth0 -v -n | grep tos

# Capture un paquet marqué EF (tos 0xb8)
# Si vous voyez "tos 0x0", le marquage a été effacé
```

**Workaround :** Utiliser du QoS applicatif (rate limiting, priorités HTTP/2).

### Piège 2 : Queue starvation

**Problème :** Le trafic basse priorité ne passe jamais (starvation).

**Cause :** Priority Queuing strict sans garantie minimale.

**Solution :** Utiliser CBWFQ avec bande passante garantie pour chaque classe.

```
# Mauvais (strict priority)
class VOIP
  priority percent 80  ← Le reste affamé !

# Bon (priorité + garantie)
class VOIP
  priority percent 20
class BUSINESS
  bandwidth percent 40
  random-detect
class DEFAULT
  bandwidth percent 40  ← Garantie minimale
```

### Piège 3 : Over-provisioning de priorité

**Problème :** Trop de trafic marqué "critique" → perd son sens.

**Règle :** Maximum **20-30% du trafic** en haute priorité (EF).

**Exemple d'anti-pattern :**
```python
# ❌ MAUVAIS : tout marquer en critique
for packet in all_packets:
    packet.dscp = 46  # Tout est EF !
```

**Correct :**
```python
# ✅ BON : classifier finement
if packet.type == 'VOIP_RTP':
    packet.dscp = 46  # EF (5-10% du trafic)
elif packet.type == 'BUSINESS_CRITICAL':
    packet.dscp = 34  # AF41 (20-30%)
else:
    packet.dscp = 0   # Best effort (60-75%)
```

### Piège 4 : Oublier la QoS end-to-end

**Problème :** QoS configurée sur un seul segment du réseau.

**Réalité :** La QoS doit être cohérente sur **tout le chemin** :

```
Client → Edge Switch → Core Switch → Routeur WAN → Internet → Serveur
         [QoS 1]       [QoS 2]        [QoS 3]       [???]    [QoS 4]
```

Si **un seul** maillon n'applique pas la QoS, tout le bénéfice est perdu.

**Solution :** Cartographier le chemin réseau et vérifier chaque hop.

### Piège 5 : Permissions insuffisantes

**Problème :** `PermissionError` lors du marquage DSCP.

**Cause :** Besoin de `CAP_NET_ADMIN` sous Linux.

**Solutions :**

```bash
# Option 1 : Donner la capability au binaire
sudo setcap cap_net_admin=eip /usr/bin/python3.11

# Option 2 : Exécuter avec sudo (développement uniquement)
sudo python3 app.py

# Option 3 : Utiliser systemd avec AmbientCapabilities (production)
# /etc/systemd/system/myapp.service
[Service]
ExecStart=/usr/bin/python3 /opt/app/main.py
AmbientCapabilities=CAP_NET_ADMIN
```

## Outils et technologies

### Outils d'analyse QoS

| Outil | Usage | Exemple |
|-------|-------|---------|
| **tcpdump** | Vérifier marquage DSCP | `tcpdump -vv -n | grep tos` |
| **Wireshark** | Analyser QoS détaillé | Filter: `ip.dsfield.dscp == 46` |
| **iperf3** | Tester QoS par classe | `iperf3 -c server --dscp EF` |
| **tc (Linux)** | Configurer/monitorer QoS | `tc -s qdisc show` |
| **nstat** | Stats kernel réseau | `nstat -az` |

### Test avec iperf3

```bash
# Serveur
iperf3 -s

# Client : tester trafic DSCP EF (VoIP)
iperf3 -c 10.0.1.10 --dscp 46 -u -b 2M -t 60

# Client : tester trafic Best Effort
iperf3 -c 10.0.1.10 --dscp 0 -u -b 100M -t 60

# Comparer les pertes de paquets et la latence
```

### Solutions commerciales vs Open Source

| Solution | Type | Forces | Faiblesses |
|----------|------|---------|------------|
| **Linux TC** | Open Source | Gratuit, flexible, puissant | Courbe d'apprentissage |
| **Calico** | Open Source (K8s) | Intégration native K8s | Limité à Kubernetes |
| **Cisco QoS** | Commercial | Mature, support entreprise | Coûteux, vendor lock-in |
| **AWS Transit Gateway** | Cloud | Scalabilité cloud | Lock-in AWS |
| **Cloudflare Argo** | SaaS | QoS global automatique | Boîte noire, coûteux |

## Résumé : Quand et comment utiliser la QoS

### Checklist de décision

✅ **Utilisez la QoS si :**
- Vous avez du trafic temps réel (VoIP, vidéo, gaming)
- Votre lien WAN est proche de la saturation
- Vous devez garantir des SLAs stricts
- Vous mixez trafic critique et bulk sur le même lien

❌ **Ne vous embêtez pas avec QoS si :**
- Vous avez une bande passante largement surdimensionnée (>10x le trafic)
- Tout votre trafic est best-effort
- Vous êtes sur un réseau dont vous ne contrôlez pas les équipements

### Guide de marquage DSCP

| Votre application | DSCP recommandé | Nom | Valeur |
|-------------------|-----------------|-----|--------|
| VoIP, audio temps réel | EF | Expedited Forwarding | 46 |
| Vidéo interactive (Zoom, Teams) | AF41 | Assured Forwarding 4.1 | 34 |
| Streaming vidéo (YouTube) | AF42 | AF 4.2 | 36 |
| Signalisation (SIP, H.323) | CS3 | Class Selector 3 | 24 |
| Transactions critiques | AF31 | AF 3.1 | 26 |
| Données importantes | AF21 | AF 2.1 | 18 |
| Bulk transfers, backups | AF11 | AF 1.1 | 10 |
| Trafic par défaut | BE | Best Effort | 0 |

### Template de configuration rapide

```bash
#!/bin/bash
# QoS Quick Start Template

INTERFACE="eth0"
TOTAL_BW="1gbit"

# Setup HTB
tc qdisc add dev $INTERFACE root handle 1: htb default 99
tc class add dev $INTERFACE parent 1: classid 1:1 htb rate $TOTAL_BW

# Critical (20%)
tc class add dev $INTERFACE parent 1:1 classid 1:10 htb rate 200mbit ceil 400mbit prio 1
tc qdisc add dev $INTERFACE parent 1:10 handle 10: sfq

# High (30%)
tc class add dev $INTERFACE parent 1:1 classid 1:20 htb rate 300mbit ceil 600mbit prio 2
tc qdisc add dev $INTERFACE parent 1:20 handle 20: sfq

# Normal (30%)
tc class add dev $INTERFACE parent 1:1 classid 1:30 htb rate 300mbit ceil 500mbit prio 3
tc qdisc add dev $INTERFACE parent 1:30 handle 30: sfq

# Default/Low (20%)
tc class add dev $INTERFACE parent 1:1 classid 1:99 htb rate 200mbit ceil 300mbit prio 4
tc qdisc add dev $INTERFACE parent 1:99 handle 99: sfq

# Filters basés sur DSCP
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 1 u32 match ip tos 0xb8 0xfc flowid 1:10  # EF
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 2 u32 match ip tos 0x88 0xfc flowid 1:20  # AF41
tc filter add dev $INTERFACE protocol ip parent 1:0 prio 3 u32 match ip tos 0x48 0xfc flowid 1:30  # AF21

echo "QoS basique configurée"
```

## Perspectives : QoS dans le futur

### Tendances émergentes

**1. QoS applicative vs réseau**
- Montée des solutions **application-aware** (Istio, Linkerd)
- Déplacement de la QoS vers la couche 7
- Moins de dépendance aux équipements réseau

**2. Machine Learning pour QoS dynamique**
- Prédiction de congestion
- Ajustement automatique des priorités
- Auto-scaling basé sur QoS metrics

**3. QoS dans les réseaux 5G**
- Slicing réseau (découpage en tranches virtuelles)
- QoS garantie end-to-end pour IoT, véhicules autonomes
- URLLC (Ultra-Reliable Low-Latency Communications)

**4. eBPF pour QoS programmable**
```c
// eBPF permet de faire de la QoS au niveau kernel sans modules
// Exemple simplifié
SEC("tc")
int qos_classifier(struct __sk_buff *skb) {
    // Lire le port destination
    __u16 dport = load_half(skb, offsetof(struct tcphdr, dest));

    if (dport == 443) {
        // HTTPS → priorité moyenne
        skb->priority = 2;
    } else if (dport >= 16384 && dport <= 32767) {
        // RTP → priorité haute
        skb->priority = 1;
    }

    return TC_ACT_OK;
}
```

## Conclusion

La QoS n'est plus un luxe réservé aux telcos : c'est une **nécessité pour toute application distribuée moderne**. Que vous développiez une API de paiement, une plateforme de streaming, ou des microservices dans Kubernetes, comprendre et implémenter la QoS vous permettra de :

- ✅ Garantir des performances prévisibles
- ✅ Respecter vos SLAs même sous charge
- ✅ Optimiser l'utilisation de votre infrastructure
- ✅ Offrir une meilleure expérience utilisateur

**À retenir :**
- Classifier finement votre trafic (pas tout en "critique")
- Utiliser le marquage DSCP standard (EF, AF4x, AF3x, etc.)
- Configurer la QoS end-to-end (pas juste un segment)
- Monitorer pour détecter violations de SLA
- Tester régulièrement vos configurations

**Prochaine étape :** Dans la section suivante, nous explorerons le **Multicast et IGMP**, essentiels pour diffuser efficacement du contenu vers des milliers de destinataires simultanément.

---


⏭️ [Multicast et IGMP](/09-architectures-avancees/02-multicast-igmp.md)
