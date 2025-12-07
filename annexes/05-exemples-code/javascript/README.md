🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Exemples JavaScript/Node.js

## Introduction

JavaScript (via Node.js) est particulièrement adapté à la programmation réseau grâce à :
- Son **modèle événementiel** non-bloquant
- Son architecture **asynchrone par défaut**
- Sa **boucle d'événements** (event loop)
- Ses modules standard `net`, `dgram`, `http`
- Son écosystème riche (npm)
- Sa facilité d'intégration avec le web

**Versions Node.js :** Ces exemples sont compatibles Node.js 14+

**Prérequis :**
- Installation de Node.js (https://nodejs.org/)
- Connaissances de base en JavaScript
- Compréhension des concepts TCP/IP
- Familiarité avec les concepts asynchrones

---

## Organisation des exemples

```
javascript/
├── README.md (ce fichier)
├── 01-node-tcp/
│   ├── server.js
│   ├── client.js
│   └── README.md
├── 02-node-udp/
│   ├── server.js
│   ├── client.js
│   └── README.md
├── 03-websocket/
│   ├── server.js
│   ├── client.html
│   └── README.md
└── 04-http-server/
    ├── server.js
    ├── advanced-server.js
    └── README.md
```

---

## Pourquoi Node.js pour le réseau ?

### Architecture événementielle

Node.js utilise une **boucle d'événements** unique au lieu de threads multiples :

```
┌───────────────────────────┐
│        Event Loop         │
│    (Single Thread)        │
└─────────┬─────────────────┘
          │
          ├─> Event 1 → Callback 1
          ├─> Event 2 → Callback 2
          ├─> Event 3 → Callback 3
          └─> Event N → Callback N

      ↓ Non-bloquant I/O ↓

┌─────────────────────────────┐
│    Thread Pool (libuv)      │
│  Opérations I/O système     │
└─────────────────────────────┘
```

### Comparaison avec Python et Go

| Aspect | Python | Go | Node.js |
|--------|--------|-----|---------|
| **Modèle** | Threading/Asyncio | Goroutines | Event Loop |
| **Concurrence** | Préemptif (threads) | Coopératif (goroutines) | Événementiel |
| **I/O** | Bloquant par défaut | Bloquant par défaut | Non-bloquant par défaut |
| **Scalabilité** | ~1000 clients | 10000+ clients | 10000+ clients |
| **Overhead mémoire** | ~8MB/thread | ~2KB/goroutine | ~KB/connexion |
| **Complexité** | Moyenne | Moyenne | Moyenne (callbacks) |
| **Cas d'usage** | Scripts, ML | Microservices, CLI | APIs, temps réel |

### Forces de Node.js

- ✅ **I/O non-bloquant** : Parfait pour nombreuses connexions
- ✅ **Single-threaded** : Pas de race conditions
- ✅ **Écosystème npm** : Modules pour tout
- ✅ **JavaScript** : Même langage front/back
- ✅ **WebSocket natif** : Excellent pour temps réel
- ✅ **JSON** : Manipulation native
- ✅ **Event-driven** : Architecture naturelle pour réseau

### Faiblesses de Node.js

- ❌ **CPU-intensive** : Mauvais pour calculs lourds
- ❌ **Callback hell** : Complexité avec callbacks imbriqués
- ❌ **Single-threaded** : Un crash = tout s'arrête
- ❌ **Typage faible** : Erreurs à l'exécution

---

## 01. TCP Simple - Echo Server

### Concept

Le module `net` de Node.js fournit une API asynchrone pour créer des serveurs et clients TCP.

### server.js - Version basique

```javascript
// Importer le module net
const net = require('net');

// Configuration
const HOST = '127.0.0.1';
const PORT = 8080;

// Créer un serveur TCP
// createServer() prend une fonction callback pour chaque connexion
const server = net.createServer((socket) => {
    // Cette fonction est appelée pour chaque nouvelle connexion
    // socket est un objet net.Socket (duplex stream)

    console.log(`[CONNEXION] Nouveau client: ${socket.remoteAddress}:${socket.remotePort}`);

    // Événement 'data' : données reçues
    socket.on('data', (data) => {
        // data est un Buffer
        const message = data.toString();
        console.log(`[REÇU] ${data.length} octets: ${message}`);

        // Echo : renvoyer les données
        socket.write(data);
        console.log(`[ENVOYÉ] Echo renvoyé`);
    });

    // Événement 'end' : client a fermé la connexion
    socket.on('end', () => {
        console.log(`[DÉCONNEXION] Client déconnecté`);
    });

    // Événement 'error' : erreur sur la socket
    socket.on('error', (err) => {
        console.error(`[ERREUR] ${err.message}`);
    });
});

// Événement 'error' du serveur
server.on('error', (err) => {
    console.error(`[SERVEUR ERREUR] ${err.message}`);
});

// Écouter sur le port
server.listen(PORT, HOST, () => {
    console.log(`[SERVEUR] Démarré sur ${HOST}:${PORT}`);
    console.log(`[SERVEUR] En attente de connexions...`);
});
```

### Explications détaillées

#### 1. Module net

```javascript
const net = require('net');
```

**Le module `net` fournit :**
- `net.createServer()` : Créer un serveur TCP
- `net.connect()` / `net.createConnection()` : Se connecter à un serveur
- `net.Socket` : Représente une connexion TCP
- Support IPv4 et IPv6

#### 2. net.createServer()

```javascript
const server = net.createServer(connectionListener);
```

**Paramètre :**
- `connectionListener(socket)` : Fonction appelée pour chaque connexion
- Équivalent à `server.on('connection', listener)`

**Retourne :** Un objet `net.Server`

#### 3. Architecture événementielle

```javascript
socket.on('data', (data) => {
    // Traiter les données
});
```

**Pattern événementiel Node.js :**
1. Enregistrer des listeners avec `.on(event, callback)`
2. Les événements sont émis de manière asynchrone
3. Les callbacks sont appelés quand les événements se produisent

**Événements principaux de Socket :**

| Événement | Quand | Callback |
|-----------|-------|----------|
| `'data'` | Données reçues | `(data: Buffer)` |
| `'end'` | Autre côté a fermé | `()` |
| `'close'` | Socket complètement fermée | `(hadError: boolean)` |
| `'error'` | Erreur | `(err: Error)` |
| `'timeout'` | Timeout atteint | `()` |
| `'drain'` | Buffer d'écriture vidé | `()` |

#### 4. Buffer vs String

```javascript
socket.on('data', (data) => {
    // data est un Buffer (tableau d'octets)
    console.log(data);  // <Buffer 48 65 6c 6c 6f>

    // Convertir en string
    const message = data.toString();     // UTF-8 par défaut
    const hex = data.toString('hex');    // Hexadécimal
    const base64 = data.toString('base64'); // Base64
});
```

**Buffer :**
- Représente des données binaires
- Similaire aux `Uint8Array`
- Créé avec `Buffer.from()`, `Buffer.alloc()`

#### 5. server.listen()

```javascript
server.listen(PORT, HOST, callback);
```

**Variantes :**
```javascript
// Port uniquement (écoute sur toutes les interfaces)
server.listen(8080);

// Port et callback
server.listen(8080, () => {
    console.log('Serveur démarré');
});

// Objet options
server.listen({
    host: '0.0.0.0',
    port: 8080,
    backlog: 511
});
```

---

### client.js - Client simple

```javascript
const net = require('net');

// Configuration
const SERVER_HOST = '127.0.0.1';
const SERVER_PORT = 8080;

// Message à envoyer
const message = process.argv[2] || 'Hello, Server!';

// Créer une connexion
// connect() retourne un net.Socket
const client = net.connect(SERVER_PORT, SERVER_HOST, () => {
    // Callback appelé quand la connexion est établie
    console.log(`[CLIENT] Connecté à ${SERVER_HOST}:${SERVER_PORT}`);

    // Envoyer le message
    console.log(`[ENVOI] ${message}`);
    client.write(message);
});

// Recevoir la réponse
client.on('data', (data) => {
    const response = data.toString();
    console.log(`[RÉPONSE] ${response}`);

    // Vérifier l'echo
    if (response === message) {
        console.log('[SUCCESS] Echo correct!');
    }

    // Fermer la connexion
    client.end();
});

// Connexion fermée
client.on('end', () => {
    console.log('[CLIENT] Déconnecté du serveur');
});

// Erreur
client.on('error', (err) => {
    console.error(`[ERREUR] ${err.message}`);
});
```

### Explications client

#### net.connect()

```javascript
const client = net.connect(port, host, connectListener);
```

**Variantes :**
```javascript
// Port et host
net.connect(8080, '127.0.0.1');

// Objet options
net.connect({
    port: 8080,
    host: '127.0.0.1',
    timeout: 5000  // Timeout de connexion (ms)
});

// Alias : createConnection()
net.createConnection(8080, '127.0.0.1');
```

**Événements de connexion :**

| Événement | Description |
|-----------|-------------|
| `'connect'` | Connexion établie |
| `'ready'` | Socket prête à écrire |
| `'timeout'` | Timeout atteint |
| `'error'` | Erreur de connexion |

#### Fermeture de connexion

```javascript
// Fermer en envoyant un FIN (half-close)
socket.end();

// Fermer en envoyant des données d'abord
socket.end('Goodbye');

// Fermer immédiatement (RST)
socket.destroy();

// Vérifier si la connexion est fermée
if (socket.destroyed) {
    console.log('Socket détruite');
}
```

---

### Exécution

**Terminal 1 - Serveur :**
```bash
$ node server.js
[SERVEUR] Démarré sur 127.0.0.1:8080
[SERVEUR] En attente de connexions...
```

**Terminal 2 - Client :**
```bash
$ node client.js "Bonjour Node.js!"
[CLIENT] Connecté à 127.0.0.1:8080
[ENVOI] Bonjour Node.js!
[RÉPONSE] Bonjour Node.js!
[SUCCESS] Echo correct!
[CLIENT] Déconnecté du serveur
```

**Terminal 1 affiche :**
```
[CONNEXION] Nouveau client: 127.0.0.1:54321
[REÇU] 17 octets: Bonjour Node.js!
[ENVOYÉ] Echo renvoyé
[DÉCONNEXION] Client déconnecté
```

---

## 02. Gestion asynchrone - Callbacks, Promises, Async/Await

### Évolution de l'asynchrone en JavaScript

#### 1. Callbacks (style classique Node.js)

```javascript
const net = require('net');

// Client avec callbacks
function sendMessage(message, callback) {
    const client = net.connect(8080, '127.0.0.1', () => {
        client.write(message);
    });

    client.on('data', (data) => {
        callback(null, data.toString());
        client.end();
    });

    client.on('error', (err) => {
        callback(err);
    });
}

// Utilisation
sendMessage('Hello', (err, response) => {
    if (err) {
        console.error('Erreur:', err);
        return;
    }
    console.log('Réponse:', response);
});
```

**Problème : Callback Hell**
```javascript
// ❌ Callbacks imbriqués - difficile à lire
sendMessage('msg1', (err1, res1) => {
    if (err1) return console.error(err1);
    sendMessage('msg2', (err2, res2) => {
        if (err2) return console.error(err2);
        sendMessage('msg3', (err3, res3) => {
            if (err3) return console.error(err3);
            console.log('Terminé');
        });
    });
});
```

#### 2. Promises (ES6+)

```javascript
const net = require('net');

// Wrapper Promise autour de net.connect
function sendMessage(message) {
    return new Promise((resolve, reject) => {
        const client = net.connect(8080, '127.0.0.1', () => {
            client.write(message);
        });

        client.on('data', (data) => {
            resolve(data.toString());
            client.end();
        });

        client.on('error', (err) => {
            reject(err);
        });
    });
}

// Utilisation avec .then()
sendMessage('Hello')
    .then(response => {
        console.log('Réponse:', response);
    })
    .catch(err => {
        console.error('Erreur:', err);
    });

// Chaînage
sendMessage('msg1')
    .then(res1 => {
        console.log('1:', res1);
        return sendMessage('msg2');
    })
    .then(res2 => {
        console.log('2:', res2);
        return sendMessage('msg3');
    })
    .then(res3 => {
        console.log('3:', res3);
    })
    .catch(err => {
        console.error('Erreur:', err);
    });
```

#### 3. Async/Await (ES2017+)

```javascript
// Fonction async retourne automatiquement une Promise
async function sendMessage(message) {
    return new Promise((resolve, reject) => {
        const client = net.connect(8080, '127.0.0.1', () => {
            client.write(message);
        });

        client.on('data', (data) => {
            resolve(data.toString());
            client.end();
        });

        client.on('error', reject);
    });
}

// Utilisation avec await (dans une fonction async)
async function main() {
    try {
        // await "attend" la résolution de la Promise
        const response1 = await sendMessage('msg1');
        console.log('1:', response1);

        const response2 = await sendMessage('msg2');
        console.log('2:', response2);

        const response3 = await sendMessage('msg3');
        console.log('3:', response3);

    } catch (err) {
        console.error('Erreur:', err);
    }
}

main();
```

**✅ Avantages async/await :**
- Code qui ressemble à du synchrone
- Facile à lire et débugger
- Gestion d'erreur avec try/catch

---

## 03. Serveur multi-clients

Node.js gère naturellement plusieurs clients grâce à son architecture événementielle.

### server-multi.js

```javascript
const net = require('net');

const HOST = '127.0.0.1';
const PORT = 8080;

// Tableau pour stocker les clients connectés
const clients = new Set();

// Compteur de clients
let clientCounter = 0;

const server = net.createServer((socket) => {
    // Assigner un ID au client
    const clientId = ++clientCounter;
    const clientAddress = `${socket.remoteAddress}:${socket.remotePort}`;

    console.log(`\n[CLIENT #${clientId}] Connexion de ${clientAddress}`);

    // Ajouter à la liste
    clients.add(socket);
    socket.clientId = clientId;

    console.log(`[SERVEUR] Clients connectés: ${clients.size}`);

    // Gérer les données
    socket.on('data', (data) => {
        const message = data.toString().trim();
        console.log(`[CLIENT #${clientId}] Reçu: ${message}`);

        // Echo
        socket.write(`Echo: ${message}\n`);
    });

    // Gérer la déconnexion
    socket.on('end', () => {
        console.log(`[CLIENT #${clientId}] Déconnexion`);
        clients.delete(socket);
        console.log(`[SERVEUR] Clients connectés: ${clients.size}`);
    });

    // Gérer les erreurs
    socket.on('error', (err) => {
        console.error(`[CLIENT #${clientId}] Erreur: ${err.message}`);
        clients.delete(socket);
    });
});

// Gestion d'erreur serveur
server.on('error', (err) => {
    console.error(`[SERVEUR] Erreur: ${err.message}`);
});

// Démarrer le serveur
server.listen(PORT, HOST, () => {
    console.log(`[SERVEUR] Démarré sur ${HOST}:${PORT}`);
    console.log(`[SERVEUR] Architecture: Event Loop (single-threaded)`);
});

// Gestion du Ctrl+C
process.on('SIGINT', () => {
    console.log('\n[SERVEUR] Arrêt...');

    // Fermer toutes les connexions
    clients.forEach(socket => {
        socket.end('Serveur en cours d\'arrêt\n');
    });

    // Fermer le serveur
    server.close(() => {
        console.log('[SERVEUR] Arrêté');
        process.exit(0);
    });
});
```

### Client interactif

```javascript
const net = require('net');
const readline = require('readline');

const SERVER_HOST = '127.0.0.1';
const SERVER_PORT = 8080;

// Interface readline pour lire stdin
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
    prompt: '> '
});

// Connexion au serveur
const client = net.connect(SERVER_PORT, SERVER_HOST, () => {
    console.log(`Connecté à ${SERVER_HOST}:${SERVER_PORT}`);
    console.log('Tapez vos messages (Ctrl+C pour quitter):');
    rl.prompt();
});

// Recevoir des données du serveur
client.on('data', (data) => {
    // Effacer la ligne courante et afficher la réponse
    readline.clearLine(process.stdout, 0);
    readline.cursorTo(process.stdout, 0);
    console.log(data.toString().trim());
    rl.prompt();
});

// Lire l'input utilisateur
rl.on('line', (line) => {
    const message = line.trim();

    if (message) {
        client.write(message + '\n');
    }

    rl.prompt();
});

// Gestion des erreurs
client.on('error', (err) => {
    console.error(`Erreur: ${err.message}`);
    rl.close();
});

// Connexion fermée
client.on('end', () => {
    console.log('\nDéconnecté du serveur');
    rl.close();
});

// Fermeture propre
rl.on('close', () => {
    client.end();
    process.exit(0);
});
```

### Architecture Event Loop

```
┌───────────────────────────┐
│   ┌─────────────────┐     │
│   │   Timers        │     │  setTimeout, setInterval
│   └─────────────────┘     │
│   ┌─────────────────┐     │
│   │ Pending I/O     │     │  Callbacks I/O
│   └─────────────────┘     │
│   ┌─────────────────┐     │
│   │   Idle, Poll    │     │  Récupérer nouveaux I/O
│   └─────────────────┘     │
│   ┌─────────────────┐     │
│   │   Check         │     │  setImmediate
│   └─────────────────┘     │
│   ┌─────────────────┐     │
│   │ Close callbacks │     │  socket.on('close')
│   └─────────────────┘     │
└───────────────────────────┘
        ↓      ↑
     Recommence
```

**Phases de l'Event Loop :**
1. **Timers** : Exécute callbacks de `setTimeout`, `setInterval`
2. **I/O callbacks** : Exécute callbacks I/O
3. **Poll** : Récupère nouveaux événements I/O
4. **Check** : Exécute `setImmediate`
5. **Close** : Callbacks de fermeture

---

## 04. UDP - Datagrammes

### Module dgram

```javascript
const dgram = require('dgram');
```

**dgram fournit :**
- `dgram.createSocket()` : Créer une socket UDP
- Support IPv4 et IPv6
- Broadcast et multicast

### udp-server.js

```javascript
const dgram = require('dgram');

const HOST = '127.0.0.1';
const PORT = 8080;

// Créer une socket UDP (IPv4)
const server = dgram.createSocket('udp4');

// Événement 'message' : datagramme reçu
server.on('message', (msg, rinfo) => {
    // msg est un Buffer
    // rinfo contient { address, port, family, size }

    console.log(`\n[REÇU DE ${rinfo.address}:${rinfo.port}]`);
    console.log(`  Message: ${msg.toString()}`);
    console.log(`  Taille: ${msg.length} octets`);

    // Echo : renvoyer au même client
    const response = Buffer.from(`Echo: ${msg.toString()}`);
    server.send(response, rinfo.port, rinfo.address, (err) => {
        if (err) {
            console.error('Erreur send:', err);
        } else {
            console.log(`[ENVOYÉ À ${rinfo.address}:${rinfo.port}] Echo`);
        }
    });
});

// Événement 'listening' : serveur prêt
server.on('listening', () => {
    const address = server.address();
    console.log(`[SERVEUR UDP] Démarré sur ${address.address}:${address.port}`);
});

// Événement 'error'
server.on('error', (err) => {
    console.error(`[SERVEUR UDP] Erreur: ${err.message}`);
    server.close();
});

// Lier au port
server.bind(PORT, HOST);
```

### udp-client.js

```javascript
const dgram = require('dgram');

const SERVER_HOST = '127.0.0.1';
const SERVER_PORT = 8080;

// Message
const message = process.argv[2] || 'Hello UDP!';

// Créer une socket UDP
const client = dgram.createSocket('udp4');

// Préparer le message
const messageBuffer = Buffer.from(message);

console.log(`[CLIENT UDP] Envoi à ${SERVER_HOST}:${SERVER_PORT}`);
console.log(`[CLIENT UDP] Message: ${message}`);

// Envoyer le datagramme
// send(msg, offset, length, port, address, callback)
client.send(messageBuffer, 0, messageBuffer.length, SERVER_PORT, SERVER_HOST, (err) => {
    if (err) {
        console.error('Erreur send:', err);
        client.close();
        return;
    }

    console.log('[CLIENT UDP] Datagramme envoyé');
});

// Définir un timeout
const timeout = setTimeout(() => {
    console.log('[CLIENT UDP] Timeout - Aucune réponse');
    client.close();
}, 5000);

// Recevoir la réponse
client.on('message', (msg, rinfo) => {
    clearTimeout(timeout);

    console.log(`[CLIENT UDP] Réponse de ${rinfo.address}:${rinfo.port}`);
    console.log(`[CLIENT UDP] ${msg.toString()}`);

    // Vérifier l'echo
    if (msg.toString().includes(message)) {
        console.log('[SUCCESS] Echo correct!');
    }

    client.close();
});

// Erreur
client.on('error', (err) => {
    console.error(`[CLIENT UDP] Erreur: ${err.message}`);
    client.close();
});
```

### Différences UDP vs TCP en Node.js

| Aspect | TCP (net) | UDP (dgram) |
|--------|-----------|-------------|
| **Module** | `net` | `dgram` |
| **Type** | Stream | Datagramme |
| **Connexion** | `createServer()`, `connect()` | `createSocket()`, `bind()` |
| **Envoi** | `socket.write()` | `socket.send(msg, port, addr)` |
| **Réception** | Événement `'data'` | Événement `'message'` (msg, rinfo) |
| **État** | Connexion maintenue | Sans état |
| **Fiabilité** | Garantie | Aucune |

---

## 05. HTTP Server

Node.js inclut un module `http` pour créer facilement des serveurs web.

### http-server.js - Serveur minimal

```javascript
const http = require('http');

const HOST = '127.0.0.1';
const PORT = 8000;

// Créer le serveur HTTP
// Callback appelé pour chaque requête
const server = http.createServer((req, res) => {
    // req : http.IncomingMessage (readable stream)
    // res : http.ServerResponse (writable stream)

    console.log(`[${req.method}] ${req.url} - Client: ${req.socket.remoteAddress}`);

    // Router simple
    if (req.url === '/' && req.method === 'GET') {
        // Page d'accueil
        res.statusCode = 200;
        res.setHeader('Content-Type', 'text/html; charset=utf-8');

        const html = `
            <!DOCTYPE html>
            <html>
            <head>
                <title>Serveur Node.js</title>
                <meta charset="utf-8">
            </head>
            <body>
                <h1>🚀 Serveur HTTP Node.js</h1>
                <p>Heure du serveur: ${new Date().toLocaleString()}</p>
                <p>Votre IP: ${req.socket.remoteAddress}</p>
                <ul>
                    <li><a href="/api/time">API Time (JSON)</a></li>
                    <li><a href="/404">Page 404</a></li>
                </ul>
            </body>
            </html>
        `;

        res.end(html);

    } else if (req.url === '/api/time' && req.method === 'GET') {
        // API JSON
        res.statusCode = 200;
        res.setHeader('Content-Type', 'application/json');

        const data = {
            timestamp: Date.now(),
            datetime: new Date().toISOString(),
            timezone: Intl.DateTimeFormat().resolvedOptions().timeZone
        };

        res.end(JSON.stringify(data, null, 2));

    } else if (req.url === '/api/echo' && req.method === 'POST') {
        // Echo POST body
        let body = '';

        // Événement 'data' : chunk de données reçu
        req.on('data', (chunk) => {
            body += chunk.toString();
        });

        // Événement 'end' : toutes les données reçues
        req.on('end', () => {
            res.statusCode = 200;
            res.setHeader('Content-Type', 'application/json');

            const response = {
                received: body,
                length: body.length
            };

            res.end(JSON.stringify(response, null, 2));
        });

    } else {
        // 404 Not Found
        res.statusCode = 404;
        res.setHeader('Content-Type', 'text/html; charset=utf-8');

        res.end(`
            <h1>404 - Page non trouvée</h1>
            <p>Le chemin "${req.url}" n'existe pas.</p>
            <a href="/">Retour à l'accueil</a>
        `);
    }
});

// Écouter
server.listen(PORT, HOST, () => {
    console.log(`[SERVEUR HTTP] Démarré sur http://${HOST}:${PORT}`);
});
```

### Explications HTTP

#### 1. Objet Request (req)

```javascript
// Propriétés utiles
req.method      // 'GET', 'POST', 'PUT', 'DELETE', etc.
req.url         // '/path?query=value'
req.headers     // { 'user-agent': '...', ... }
req.httpVersion // '1.1'

// Parser l'URL
const url = require('url');
const parsedUrl = url.parse(req.url, true);
console.log(parsedUrl.pathname);  // '/path'
console.log(parsedUrl.query);     // { query: 'value' }
```

#### 2. Objet Response (res)

```javascript
// Status code
res.statusCode = 200;

// Headers
res.setHeader('Content-Type', 'text/html');
res.setHeader('X-Custom-Header', 'value');

// Ou avec un objet
res.writeHead(200, {
    'Content-Type': 'application/json',
    'X-Powered-By': 'Node.js'
});

// Envoyer des données
res.write('Chunk 1\n');
res.write('Chunk 2\n');
res.end('Dernière chunk');  // Termine la réponse

// Ou tout en une fois
res.end('Contenu complet');
```

#### 3. Gestion du body (POST/PUT)

```javascript
server.on('request', (req, res) => {
    if (req.method === 'POST') {
        let body = '';

        req.on('data', (chunk) => {
            body += chunk.toString();

            // Protection contre les gros uploads
            if (body.length > 1e6) {  // 1MB
                req.connection.destroy();
            }
        });

        req.on('end', () => {
            try {
                const data = JSON.parse(body);
                // Traiter data
            } catch (err) {
                res.statusCode = 400;
                res.end('Invalid JSON');
            }
        });
    }
});
```

---

## 06. WebSocket - Communication bidirectionnelle

### Installation de ws

```bash
npm install ws
```

### websocket-server.js

```javascript
const WebSocket = require('ws');

const PORT = 8080;

// Créer un serveur WebSocket
const wss = new WebSocket.Server({ port: PORT });

console.log(`[SERVEUR WS] Démarré sur ws://localhost:${PORT}`);

// Événement 'connection' : nouveau client
wss.on('connection', (ws, req) => {
    const clientIp = req.socket.remoteAddress;
    console.log(`\n[WS] Nouveau client: ${clientIp}`);

    // Envoyer un message de bienvenue
    ws.send(JSON.stringify({
        type: 'welcome',
        message: 'Bienvenue sur le serveur WebSocket!',
        timestamp: Date.now()
    }));

    // Événement 'message' : message reçu
    ws.on('message', (data) => {
        console.log(`[WS] Reçu: ${data}`);

        try {
            // Parser le message (supposé JSON)
            const message = JSON.parse(data);

            // Echo avec type différent
            const response = {
                type: 'echo',
                original: message,
                timestamp: Date.now()
            };

            ws.send(JSON.stringify(response));

        } catch (err) {
            // Si pas JSON, echo simple
            ws.send(data);
        }
    });

    // Événement 'close' : client déconnecté
    ws.on('close', () => {
        console.log(`[WS] Client déconnecté: ${clientIp}`);
    });

    // Événement 'error'
    ws.on('error', (err) => {
        console.error(`[WS] Erreur: ${err.message}`);
    });

    // Ping/Pong pour keep-alive
    const interval = setInterval(() => {
        if (ws.readyState === WebSocket.OPEN) {
            ws.ping();
        }
    }, 30000);  // Toutes les 30 secondes

    ws.on('close', () => {
        clearInterval(interval);
    });
});

// Broadcast à tous les clients
function broadcast(data) {
    wss.clients.forEach((client) => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(data);
        }
    });
}

// Exemple d'utilisation du broadcast
setInterval(() => {
    const message = JSON.stringify({
        type: 'server-time',
        time: new Date().toISOString()
    });
    broadcast(message);
}, 60000);  // Toutes les minutes
```

### websocket-client.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>WebSocket Client</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
        }
        #status {
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 20px;
        }
        .connected {
            background-color: #d4edda;
            color: #155724;
        }
        .disconnected {
            background-color: #f8d7da;
            color: #721c24;
        }
        #messages {
            border: 1px solid #ddd;
            padding: 10px;
            height: 300px;
            overflow-y: scroll;
            margin-bottom: 20px;
            background-color: #f9f9f9;
        }
        .message {
            margin: 5px 0;
            padding: 5px;
            border-left: 3px solid #007bff;
            background-color: white;
        }
        input, button {
            padding: 10px;
            font-size: 16px;
        }
        input {
            width: 70%;
        }
        button {
            width: 25%;
        }
    </style>
</head>
<body>
    <h1>🔌 WebSocket Client</h1>

    <div id="status" class="disconnected">
        Déconnecté
    </div>

    <div id="messages"></div>

    <div>
        <input type="text" id="messageInput" placeholder="Tapez un message...">
        <button onclick="sendMessage()">Envoyer</button>
    </div>

    <script>
        let ws;
        const messagesDiv = document.getElementById('messages');
        const statusDiv = document.getElementById('status');
        const messageInput = document.getElementById('messageInput');

        // Connexion au serveur WebSocket
        function connect() {
            ws = new WebSocket('ws://localhost:8080');

            // Événement 'open' : connexion établie
            ws.addEventListener('open', (event) => {
                console.log('Connecté au serveur WebSocket');
                statusDiv.textContent = 'Connecté';
                statusDiv.className = 'connected';
                addMessage('Système', 'Connecté au serveur', 'info');
            });

            // Événement 'message' : message reçu
            ws.addEventListener('message', (event) => {
                console.log('Message reçu:', event.data);

                try {
                    const data = JSON.parse(event.data);
                    handleMessage(data);
                } catch (err) {
                    addMessage('Serveur', event.data, 'text');
                }
            });

            // Événement 'close' : connexion fermée
            ws.addEventListener('close', (event) => {
                console.log('Déconnecté du serveur');
                statusDiv.textContent = 'Déconnecté';
                statusDiv.className = 'disconnected';
                addMessage('Système', 'Déconnecté du serveur', 'error');

                // Reconnecter après 3 secondes
                setTimeout(connect, 3000);
            });

            // Événement 'error' : erreur
            ws.addEventListener('error', (error) => {
                console.error('Erreur WebSocket:', error);
                addMessage('Système', 'Erreur de connexion', 'error');
            });
        }

        // Gérer les messages typés
        function handleMessage(data) {
            switch (data.type) {
                case 'welcome':
                    addMessage('Serveur', data.message, 'welcome');
                    break;
                case 'echo':
                    addMessage('Echo', JSON.stringify(data.original, null, 2), 'echo');
                    break;
                case 'server-time':
                    addMessage('Serveur', 'Heure serveur: ' + data.time, 'time');
                    break;
                default:
                    addMessage('Serveur', JSON.stringify(data, null, 2), 'json');
            }
        }

        // Envoyer un message
        function sendMessage() {
            const message = messageInput.value.trim();

            if (message && ws.readyState === WebSocket.OPEN) {
                const data = {
                    type: 'message',
                    content: message,
                    timestamp: Date.now()
                };

                ws.send(JSON.stringify(data));
                addMessage('Vous', message, 'sent');
                messageInput.value = '';
            }
        }

        // Ajouter un message à l'affichage
        function addMessage(sender, content, type) {
            const messageDiv = document.createElement('div');
            messageDiv.className = 'message';
            messageDiv.innerHTML = `<strong>${sender}:</strong> ${content}`;
            messagesDiv.appendChild(messageDiv);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        // Permettre d'envoyer avec Enter
        messageInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                sendMessage();
            }
        });

        // Connexion initiale
        connect();
    </script>
</body>
</html>
```

### WebSocket API

#### Côté serveur (Node.js)

```javascript
// États de connexion
WebSocket.CONNECTING  // 0
WebSocket.OPEN        // 1
WebSocket.CLOSING     // 2
WebSocket.CLOSED      // 3

// Méthodes
ws.send(data)         // Envoyer des données
ws.ping()             // Envoyer un ping
ws.pong()             // Répondre à un ping
ws.close()            // Fermer la connexion
ws.terminate()        // Fermer brutalement

// Événements
ws.on('open', callback)
ws.on('message', callback)
ws.on('close', callback)
ws.on('error', callback)
ws.on('ping', callback)
ws.on('pong', callback)
```

#### Côté client (Browser)

```javascript
// Créer une connexion
const ws = new WebSocket('ws://localhost:8080');

// États
ws.CONNECTING  // 0
ws.OPEN        // 1
ws.CLOSING     // 2
ws.CLOSED      // 3

// Vérifier l'état
if (ws.readyState === WebSocket.OPEN) {
    ws.send('message');
}

// Événements
ws.onopen = (event) => { }
ws.onmessage = (event) => { }
ws.onerror = (error) => { }
ws.onclose = (event) => { }

// Ou avec addEventListener
ws.addEventListener('message', (event) => {
    console.log(event.data);
});
```

---

## 07. Streams - Traitement de données

Node.js utilise extensivement les **streams** pour traiter efficacement de grandes quantités de données.

### Types de streams

| Type | Description | Méthodes clés |
|------|-------------|---------------|
| **Readable** | Source de données (lecture) | `.read()`, `.pipe()` |
| **Writable** | Destination de données (écriture) | `.write()`, `.end()` |
| **Duplex** | Lecture ET écriture | Les deux |
| **Transform** | Modifie les données | Duplex + transformation |

### Exemple : Pipe de socket à fichier

```javascript
const net = require('net');
const fs = require('fs');

const server = net.createServer((socket) => {
    console.log('Client connecté');

    // Créer un fichier pour sauvegarder les données
    const fileStream = fs.createWriteStream('received-data.txt');

    // Pipe : rediriger le socket vers le fichier
    // Tout ce qui est reçu sur socket est écrit dans le fichier
    socket.pipe(fileStream);

    socket.on('end', () => {
        console.log('Données sauvegardées dans received-data.txt');
    });
});

server.listen(8080);
```

### Transform stream custom

```javascript
const { Transform } = require('stream');

// Stream qui convertit en majuscules
class UpperCaseTransform extends Transform {
    _transform(chunk, encoding, callback) {
        // Transformer les données
        const upperChunk = chunk.toString().toUpperCase();
        this.push(upperChunk);
        callback();
    }
}

// Utilisation
const net = require('net');
const server = net.createServer((socket) => {
    const upperCase = new UpperCaseTransform();

    // Pipe: socket → transform → socket (echo en majuscules)
    socket.pipe(upperCase).pipe(socket);
});

server.listen(8080);
```

---

## 08. Bonnes pratiques Node.js

### 1. Gestion d'erreur

```javascript
// ✅ BON - Toujours gérer les erreurs
socket.on('error', (err) => {
    console.error('Erreur socket:', err);
    // Ne pas laisser l'app crasher
});

// ❌ MAUVAIS - Ignorer les erreurs
socket.on('data', (data) => {
    // Si erreur ici, l'app crash
    const json = JSON.parse(data.toString());
});
```

### 2. Éviter le callback hell

```javascript
// ❌ MAUVAIS - Callbacks imbriqués
getData1((err, data1) => {
    if (err) return handleError(err);
    getData2(data1, (err, data2) => {
        if (err) return handleError(err);
        getData3(data2, (err, data3) => {
            // ...
        });
    });
});

// ✅ BON - Async/await
async function processData() {
    try {
        const data1 = await getData1();
        const data2 = await getData2(data1);
        const data3 = await getData3(data2);
        return data3;
    } catch (err) {
        handleError(err);
    }
}
```

### 3. Timeouts

```javascript
// Définir un timeout
socket.setTimeout(30000);  // 30 secondes

socket.on('timeout', () => {
    console.log('Socket timeout');
    socket.end();
});
```

### 4. Limitation de taille

```javascript
// Limiter la taille des données reçues
let dataSize = 0;
const MAX_SIZE = 1e6;  // 1MB

socket.on('data', (chunk) => {
    dataSize += chunk.length;

    if (dataSize > MAX_SIZE) {
        socket.destroy();
        console.error('Données trop volumineuses');
    }
});
```

### 5. Utiliser EventEmitter

```javascript
const EventEmitter = require('events');

class ChatServer extends EventEmitter {
    constructor() {
        super();
        this.clients = new Set();
    }

    addClient(socket) {
        this.clients.add(socket);
        this.emit('client-added', socket);
    }

    broadcast(message) {
        this.clients.forEach(client => {
            client.write(message);
        });
        this.emit('broadcast', message);
    }
}

// Utilisation
const chatServer = new ChatServer();

chatServer.on('client-added', (socket) => {
    console.log('Nouveau client');
});

chatServer.on('broadcast', (message) => {
    console.log('Message diffusé:', message);
});
```

---

## 09. Debugging

### Console.log amélioré

```javascript
// Avec timestamp
function log(...args) {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}]`, ...args);
}

// Utilisation
log('Serveur démarré');
// [2024-12-07T10:30:00.123Z] Serveur démarré
```

### Debug module

```bash
npm install debug
```

```javascript
const debug = require('debug');

// Créer différents debuggers
const debugServer = debug('app:server');
const debugClient = debug('app:client');

debugServer('Serveur démarré sur le port %d', 8080);
debugClient('Client %s connecté', clientId);
```

**Exécution :**
```bash
# Activer tous les debugs
DEBUG=* node server.js

# Activer seulement app:server
DEBUG=app:server node server.js

# Activer app:*
DEBUG=app:* node server.js
```

### Inspection

```bash
# Démarrer avec inspecteur
node --inspect server.js

# Ouvrir dans Chrome
chrome://inspect
```

---

## Conclusion

Vous maîtrisez maintenant la programmation réseau en Node.js :

- ✅ **TCP** : Serveurs et clients avec module `net`
- ✅ **Asynchrone** : Callbacks, Promises, Async/Await
- ✅ **Event Loop** : Architecture événementielle
- ✅ **UDP** : Datagrammes avec module `dgram`
- ✅ **HTTP** : Serveurs web avec module `http`
- ✅ **WebSocket** : Communication bidirectionnelle temps réel
- ✅ **Streams** : Traitement efficace de données
- ✅ **Bonnes pratiques** : Gestion d'erreurs, debugging

**Forces de Node.js :**
- Excellent pour I/O intensif
- Architecture événementielle naturelle
- Écosystème npm riche
- JavaScript front/back

**Ressources supplémentaires :**
- Documentation officielle : https://nodejs.org/api/
- Node.js Best Practices : https://github.com/goldbergyoni/nodebestpractices
- Node School : https://nodeschool.io/

---


⏭️ Retour au [Sommaire](/SOMMAIRE.md)
