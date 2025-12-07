🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.12 Bibliothèques réseau modernes (aperçu)

## Introduction

Bien que comprendre les **sockets** bas niveau soit essentiel, la plupart des développeurs utilisent des **bibliothèques de haut niveau** qui abstraient la complexité du réseau. Ces bibliothèques offrent :

- 🎯 **APIs intuitives** : Plus simple que les sockets brutes
- 🔒 **Sécurité intégrée** : HTTPS, validation certificats, etc.
- ⚡ **Performance** : Connection pooling, compression, etc.
- 🔄 **Retry logic** : Gestion automatique des erreurs
- 📊 **Monitoring** : Métriques, timeouts, etc.

Cette section présente les bibliothèques les plus utilisées dans différents langages, leurs forces et faiblesses, et quand les utiliser.

## Critères de choix

Avant de choisir une bibliothèque, considérez :

```python
# Checklist de sélection

✓ Synchrone vs Asynchrone ?
  - Sync : requests, urllib3
  - Async : aiohttp, httpx

✓ HTTP/1.1 ou HTTP/2 support ?
  - HTTP/2 : httpx, aiohttp
  - HTTP/1.1 : requests

✓ WebSocket support ?
  - websockets, socket.io

✓ Performance critique ?
  - Haute perf : httpx, aiohttp
  - Standard : requests

✓ Maintenu activement ?
  - Vérifier GitHub : commits récents, issues

✓ Communauté / Documentation ?
  - requests : excellent
  - httpx : bon
  - aiohttp : moyen

✓ Compatibilité ?
  - Python 3.7+ : la plupart
  - Python 2.7 : requests (legacy)
```

## Python : Bibliothèques HTTP

### requests - Le standard de facto

**Caractéristiques** :

- ✅ API simple et élégante
- ✅ Excellente documentation
- ✅ Très stable et mature
- ✅ Énorme communauté
- ❌ Synchrone uniquement
- ❌ Pas de HTTP/2

```python
import requests

# GET simple
response = requests.get('https://api.github.com/users/octocat')
print(response.json())

# POST avec données
data = {'username': 'alice', 'email': 'alice@example.com'}
response = requests.post('https://api.example.com/users', json=data)

# Headers personnalisés
headers = {'Authorization': 'Bearer token123'}
response = requests.get('https://api.example.com/data', headers=headers)

# Timeout (CRUCIAL)
response = requests.get('https://api.example.com/data', timeout=5.0)

# Session (connection pooling)
session = requests.Session()
session.headers.update({'User-Agent': 'MyApp/1.0'})

for i in range(100):
    response = session.get(f'https://api.example.com/items/{i}')
    # Réutilise la connexion TCP

# Gestion des erreurs
try:
    response = requests.get('https://api.example.com/data', timeout=5)
    response.raise_for_status()  # Lève exception si 4xx/5xx
    data = response.json()
except requests.Timeout:
    print("Request timed out")
except requests.HTTPError as e:
    print(f"HTTP error: {e}")
except requests.RequestException as e:
    print(f"Error: {e}")
```

**Cas d'usage** :

- ✅ Scripts, CLI tools
- ✅ APIs REST synchrones
- ✅ Web scraping simple
- ✅ Prototypage rapide

**Exemple réel : Intégration Slack**

```python
import requests

class SlackNotifier:
    """Envoie des notifications à Slack"""

    def __init__(self, webhook_url):
        self.webhook_url = webhook_url
        self.session = requests.Session()
        self.session.timeout = 10

    def send_message(self, text, channel=None):
        """Envoie un message"""
        payload = {'text': text}
        if channel:
            payload['channel'] = channel

        try:
            response = self.session.post(
                self.webhook_url,
                json=payload,
                timeout=5
            )
            response.raise_for_status()
            return True
        except requests.RequestException as e:
            print(f"Failed to send to Slack: {e}")
            return False

    def send_error_alert(self, error, context=None):
        """Alerte d'erreur formatée"""
        message = f"🚨 *Error Alert*\n{error}"
        if context:
            message += f"\n\nContext:\n```{context}```"

        return self.send_message(message)

# Utilisation
slack = SlackNotifier('https://hooks.slack.com/services/YOUR/WEBHOOK/URL')
slack.send_message('Deployment completed successfully!')
```

### httpx - Le successeur moderne

**Caractéristiques** :

- ✅ API compatible requests
- ✅ Support sync ET async
- ✅ HTTP/2 support
- ✅ Timeouts par défaut
- ❌ Moins mature que requests
- ❌ Communauté plus petite

```python
import httpx

# Synchrone (comme requests)
response = httpx.get('https://api.example.com/data')
print(response.json())

# Asynchrone
import asyncio

async def fetch_data():
    async with httpx.AsyncClient() as client:
        response = await client.get('https://api.example.com/data')
        return response.json()

data = asyncio.run(fetch_data())

# HTTP/2
async with httpx.AsyncClient(http2=True) as client:
    response = await client.get('https://http2.example.com')

# Requêtes parallèles
async def fetch_multiple():
    urls = [f'https://api.example.com/items/{i}' for i in range(100)]

    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)

    return [r.json() for r in responses]

# Streaming
async with httpx.AsyncClient() as client:
    async with client.stream('GET', 'https://example.com/bigfile') as response:
        async for chunk in response.aiter_bytes():
            process_chunk(chunk)
```

**Cas d'usage** :

- ✅ Applications async modernes
- ✅ Microservices
- ✅ APIs nécessitant HTTP/2
- ✅ Migration progressive depuis requests

**Exemple réel : API Gateway async**

```python
import httpx
import asyncio
from fastapi import FastAPI, HTTPException

app = FastAPI()

class APIGateway:
    """Gateway qui agrège plusieurs APIs backend"""

    def __init__(self):
        self.client = httpx.AsyncClient(
            timeout=httpx.Timeout(10.0),
            limits=httpx.Limits(max_keepalive_connections=20)
        )

    async def get_user_dashboard(self, user_id: int):
        """Agrège données de plusieurs services"""
        # Appeler 3 services en parallèle
        profile_task = self.client.get(f'http://user-service/users/{user_id}')
        orders_task = self.client.get(f'http://order-service/users/{user_id}/orders')
        preferences_task = self.client.get(f'http://preference-service/users/{user_id}')

        try:
            profile_resp, orders_resp, prefs_resp = await asyncio.gather(
                profile_task,
                orders_task,
                preferences_task
            )

            return {
                'profile': profile_resp.json(),
                'orders': orders_resp.json(),
                'preferences': prefs_resp.json()
            }

        except httpx.TimeoutException:
            raise HTTPException(status_code=504, detail="Gateway timeout")
        except httpx.HTTPError as e:
            raise HTTPException(status_code=502, detail=f"Backend error: {e}")

gateway = APIGateway()

@app.get('/dashboard/{user_id}')
async def get_dashboard(user_id: int):
    return await gateway.get_user_dashboard(user_id)
```

### aiohttp - Async HTTP client/server

**Caractéristiques** :

- ✅ Client ET serveur
- ✅ WebSocket support
- ✅ Très performant
- ✅ Mature et stable
- ❌ API différente de requests
- ❌ Documentation moyenne

```python
import aiohttp
import asyncio

# Client
async def fetch():
    async with aiohttp.ClientSession() as session:
        async with session.get('https://api.example.com/data') as response:
            return await response.json()

# Requêtes parallèles
async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in urls:
            tasks.append(session.get(url))

        responses = await asyncio.gather(*tasks)

        results = []
        for response in responses:
            results.append(await response.json())

        return results

# Serveur HTTP
from aiohttp import web

async def handle(request):
    name = request.match_info.get('name', "Anonymous")
    text = f"Hello, {name}"
    return web.Response(text=text)

app = web.Application()
app.add_routes([web.get('/{name}', handle)])

# web.run_app(app)

# WebSocket
async def websocket_handler(request):
    ws = web.WebSocketResponse()
    await ws.prepare(request)

    async for msg in ws:
        if msg.type == aiohttp.WSMsgType.TEXT:
            await ws.send_str(f"Echo: {msg.data}")
        elif msg.type == aiohttp.WSMsgType.ERROR:
            print(f'Error: {ws.exception()}')

    return ws
```

**Cas d'usage** :

- ✅ Serveurs async haute performance
- ✅ WebSocket servers
- ✅ Microservices Python
- ✅ Web scraping massif

**Exemple réel : Web Scraper async**

```python
import aiohttp
import asyncio
from bs4 import BeautifulSoup
from urllib.parse import urljoin

class AsyncWebScraper:
    """Scraper web asynchrone haute performance"""

    def __init__(self, max_concurrent=50):
        self.max_concurrent = max_concurrent
        self.semaphore = asyncio.Semaphore(max_concurrent)

    async def fetch_page(self, session, url):
        """Récupère une page"""
        async with self.semaphore:
            try:
                async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as response:
                    if response.status == 200:
                        html = await response.text()
                        return url, html
                    else:
                        return url, None
            except Exception as e:
                print(f"Error fetching {url}: {e}")
                return url, None

    async def extract_links(self, html, base_url):
        """Extrait les liens d'une page"""
        soup = BeautifulSoup(html, 'html.parser')
        links = set()

        for link in soup.find_all('a', href=True):
            href = link['href']
            full_url = urljoin(base_url, href)
            if full_url.startswith('http'):
                links.add(full_url)

        return links

    async def scrape(self, start_urls, max_pages=1000):
        """Scrape des pages web"""
        to_visit = set(start_urls)
        visited = set()
        results = {}

        async with aiohttp.ClientSession() as session:
            while to_visit and len(visited) < max_pages:
                # Prendre un batch d'URLs
                batch_size = min(self.max_concurrent, len(to_visit))
                batch = [to_visit.pop() for _ in range(batch_size)]

                # Fetch en parallèle
                tasks = [self.fetch_page(session, url) for url in batch]
                pages = await asyncio.gather(*tasks)

                # Traiter les résultats
                for url, html in pages:
                    visited.add(url)

                    if html:
                        results[url] = html

                        # Extraire nouveaux liens
                        new_links = await self.extract_links(html, url)
                        for link in new_links:
                            if link not in visited and link not in to_visit:
                                to_visit.add(link)

                print(f"Visited: {len(visited)}, Queue: {len(to_visit)}")

        return results

# Utilisation
async def main():
    scraper = AsyncWebScraper(max_concurrent=50)
    results = await scraper.scrape(['https://example.com'], max_pages=500)
    print(f"Scraped {len(results)} pages")

asyncio.run(main())

# Performance : 500 pages en ~20 secondes
# vs 500+ secondes en synchrone
```

### urllib3 - Low-level HTTP

**Caractéristiques** :

- ✅ Bas niveau, contrôle total
- ✅ Base de requests
- ✅ Connection pooling
- ✅ Retry logic
- ❌ API moins intuitive
- ❌ Moins de features

```python
import urllib3

# Pool manager
http = urllib3.PoolManager()

# GET simple
response = http.request('GET', 'https://api.example.com/data')
print(response.status)
print(response.data)

# POST avec données
data = {'key': 'value'}
response = http.request(
    'POST',
    'https://api.example.com/data',
    fields=data
)

# Retry automatique
from urllib3.util.retry import Retry

retry = Retry(
    total=3,
    backoff_factor=0.3,
    status_forcelist=[500, 502, 503, 504]
)

http = urllib3.PoolManager(retries=retry)

# Streaming
response = http.request('GET', 'https://example.com/bigfile', preload_content=False)

for chunk in response.stream(8192):
    process_chunk(chunk)

response.release_conn()
```

**Cas d'usage** :

- ✅ Bibliothèques bas niveau
- ✅ Contrôle fin sur connections
- ✅ Performance critique
- ❌ Applications classiques (utiliser requests)

## JavaScript/Node.js : Bibliothèques HTTP

### axios - Le plus populaire

```javascript
const axios = require('axios');

// GET simple
const response = await axios.get('https://api.example.com/data');
console.log(response.data);

// POST
const data = { username: 'alice', email: 'alice@example.com' };
await axios.post('https://api.example.com/users', data);

// Interceptors (middleware)
axios.interceptors.request.use(config => {
  config.headers['Authorization'] = `Bearer ${getToken()}`;
  return config;
});

axios.interceptors.response.use(
  response => response,
  error => {
    if (error.response.status === 401) {
      // Refresh token
      return refreshTokenAndRetry(error.config);
    }
    return Promise.reject(error);
  }
);

// Instance avec config par défaut
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
  headers: {'X-Custom-Header': 'value'}
});

// Requêtes parallèles
const [users, posts, comments] = await Promise.all([
  axios.get('/users'),
  axios.get('/posts'),
  axios.get('/comments')
]);

// Cancel token
const source = axios.CancelToken.source();

axios.get('/data', {
  cancelToken: source.token
}).catch(error => {
  if (axios.isCancel(error)) {
    console.log('Request cancelled');
  }
});

// Annuler la requête
source.cancel('Operation cancelled by user');
```

**Cas d'usage réel : API Client React**

```javascript
// api/client.js
import axios from 'axios';

class APIClient {
  constructor() {
    this.client = axios.create({
      baseURL: process.env.REACT_APP_API_URL,
      timeout: 10000
    });

    // Request interceptor
    this.client.interceptors.request.use(
      config => {
        const token = localStorage.getItem('token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      error => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      response => response,
      async error => {
        const originalRequest = error.config;

        // Retry sur 401 (token expiré)
        if (error.response.status === 401 && !originalRequest._retry) {
          originalRequest._retry = true;

          try {
            const newToken = await this.refreshToken();
            localStorage.setItem('token', newToken);
            originalRequest.headers.Authorization = `Bearer ${newToken}`;
            return this.client(originalRequest);
          } catch (refreshError) {
            // Redirect to login
            window.location.href = '/login';
            return Promise.reject(refreshError);
          }
        }

        return Promise.reject(error);
      }
    );
  }

  async getUser(userId) {
    const response = await this.client.get(`/users/${userId}`);
    return response.data;
  }

  async createUser(userData) {
    const response = await this.client.post('/users', userData);
    return response.data;
  }

  async refreshToken() {
    const refreshToken = localStorage.getItem('refreshToken');
    const response = await this.client.post('/auth/refresh', { refreshToken });
    return response.data.token;
  }
}

export default new APIClient();
```

### fetch API - Native JavaScript

```javascript
// Moderne, natif dans navigateurs et Node.js 18+

// GET simple
const response = await fetch('https://api.example.com/data');
const data = await response.json();

// POST
const response = await fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ username: 'alice' })
});

// Gestion des erreurs
try {
  const response = await fetch('https://api.example.com/data');

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  const data = await response.json();
} catch (error) {
  console.error('Fetch error:', error);
}

// Timeout (avec AbortController)
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch('https://api.example.com/data', {
    signal: controller.signal
  });
  clearTimeout(timeoutId);
  const data = await response.json();
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Request timed out');
  }
}

// Streaming
const response = await fetch('https://example.com/bigfile');
const reader = response.body.getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  // Process chunk (value is Uint8Array)
  processChunk(value);
}
```

### node-fetch - fetch pour Node.js (< 18)

```javascript
const fetch = require('node-fetch');

// Identique à fetch API
const response = await fetch('https://api.example.com/data');
const data = await response.json();
```

### got - Alternative moderne

```javascript
const got = require('got');

// GET simple
const response = await got('https://api.example.com/data').json();

// Retry automatique
const response = await got('https://api.example.com/data', {
  retry: {
    limit: 3,
    methods: ['GET', 'POST'],
    statusCodes: [408, 413, 429, 500, 502, 503, 504]
  }
});

// Hooks
const response = await got('https://api.example.com/data', {
  hooks: {
    beforeRequest: [
      options => {
        console.log('Request:', options.url);
      }
    ],
    afterResponse: [
      (response, retryWithMergedOptions) => {
        if (response.statusCode === 401) {
          // Refresh token et retry
        }
        return response;
      }
    ]
  }
});

// Streaming
const stream = got.stream('https://example.com/bigfile');
stream.pipe(fs.createWriteStream('output.dat'));
```

## Go : Bibliothèques HTTP

### net/http - Standard library

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
)

// GET simple
func fetchData() (map[string]interface{}, error) {
    resp, err := http.Get("https://api.example.com/data")
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var data map[string]interface{}
    if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
        return nil, err
    }

    return data, nil
}

// POST
func createUser(user map[string]string) error {
    data, _ := json.Marshal(user)

    resp, err := http.Post(
        "https://api.example.com/users",
        "application/json",
        bytes.NewBuffer(data),
    )
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusCreated {
        return fmt.Errorf("unexpected status: %d", resp.StatusCode)
    }

    return nil
}

// Client avec timeout
func createHTTPClient() *http.Client {
    return &http.Client{
        Timeout: 10 * time.Second,
        Transport: &http.Transport{
            MaxIdleConns:        100,
            MaxIdleConnsPerHost: 10,
            IdleConnTimeout:     90 * time.Second,
        },
    }
}

// Utilisation
client := createHTTPClient()
resp, err := client.Get("https://api.example.com/data")
```

**Cas d'usage réel : Service HTTP Go**

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "time"
)

type APIClient struct {
    client  *http.Client
    baseURL string
}

func NewAPIClient(baseURL string) *APIClient {
    return &APIClient{
        client: &http.Client{
            Timeout: 30 * time.Second,
            Transport: &http.Transport{
                MaxIdleConns:       100,
                MaxConnsPerHost:    10,
                IdleConnTimeout:    90 * time.Second,
                DisableCompression: false,
            },
        },
        baseURL: baseURL,
    }
}

func (c *APIClient) GetUser(ctx context.Context, userID int) (*User, error) {
    url := fmt.Sprintf("%s/users/%d", c.baseURL, userID)

    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }

    req.Header.Set("Accept", "application/json")

    resp, err := c.client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("API error: %d", resp.StatusCode)
    }

    var user User
    if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
        return nil, err
    }

    return &user, nil
}

// Utilisation avec timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

user, err := client.GetUser(ctx, 123)
if err != nil {
    log.Fatal(err)
}
```

### fasthttp - Performance extrême

```go
package main

import (
    "github.com/valyala/fasthttp"
    "log"
)

// Client
func fetchDataFast() ([]byte, error) {
    req := fasthttp.AcquireRequest()
    resp := fasthttp.AcquireResponse()
    defer fasthttp.ReleaseRequest(req)
    defer fasthttp.ReleaseResponse(resp)

    req.SetRequestURI("https://api.example.com/data")
    req.Header.SetMethod("GET")

    client := &fasthttp.Client{}
    if err := client.Do(req, resp); err != nil {
        return nil, err
    }

    return resp.Body(), nil
}

// Serveur
func requestHandler(ctx *fasthttp.RequestCtx) {
    fmt.Fprintf(ctx, "Hello, world!")
}

func main() {
    // 10× plus rapide que net/http pour haute charge
    fasthttp.ListenAndServe(":8080", requestHandler)
}
```

**Benchmark** :

```
net/http :    50,000 req/sec
fasthttp :   500,000 req/sec

→ 10× plus rapide !
```

**Cas d'usage** :

- ✅ APIs haute performance
- ✅ Proxies
- ✅ Load balancers
- ❌ Applications standard (complexité)

## Autres langages

### Java : OkHttp

```java
import okhttp3.*;

public class HTTPClient {
    private final OkHttpClient client = new OkHttpClient();

    public String get(String url) throws IOException {
        Request request = new Request.Builder()
            .url(url)
            .build();

        try (Response response = client.newCall(request).execute()) {
            return response.body().string();
        }
    }

    public String post(String url, String json) throws IOException {
        RequestBody body = RequestBody.create(
            json,
            MediaType.get("application/json; charset=utf-8")
        );

        Request request = new Request.Builder()
            .url(url)
            .post(body)
            .build();

        try (Response response = client.newCall(request).execute()) {
            return response.body().string();
        }
    }
}
```

### Rust : reqwest

```rust
use reqwest;
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct User {
    id: u32,
    name: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // GET
    let user: User = reqwest::get("https://api.example.com/users/1")
        .await?
        .json()
        .await?;

    println!("User: {}", user.name);

    // POST
    let client = reqwest::Client::new();
    let response = client
        .post("https://api.example.com/users")
        .json(&serde_json::json!({
            "name": "Alice",
            "email": "alice@example.com"
        }))
        .send()
        .await?;

    Ok(())
}
```

### Ruby : Faraday

```ruby
require 'faraday'
require 'json'

# Connection
conn = Faraday.new(url: 'https://api.example.com') do |f|
  f.request :json
  f.response :json
  f.adapter Faraday.default_adapter
end

# GET
response = conn.get('/users/1')
user = response.body

# POST
response = conn.post('/users') do |req|
  req.body = { name: 'Alice', email: 'alice@example.com' }
end

# Middleware
conn = Faraday.new do |f|
  f.request :retry, max: 3, interval: 0.5
  f.request :authorization, 'Bearer', -> { get_token }
  f.response :logger
  f.adapter Faraday.default_adapter
end
```

## Comparaison des bibliothèques

### Python

| Bibliothèque | Sync/Async | HTTP/2 | WebSocket | Complexité | Performance |
|--------------|------------|--------|-----------|------------|-------------|
| **requests** | Sync | ❌ | ❌ | Facile | Moyenne |
| **httpx** | Both | ✅ | ❌ | Facile | Bonne |
| **aiohttp** | Async | ❌ | ✅ | Moyenne | Excellente |
| **urllib3** | Sync | ❌ | ❌ | Difficile | Bonne |

### JavaScript/Node.js

| Bibliothèque | Env | Features | Taille | Popularité |
|--------------|-----|----------|--------|------------|
| **axios** | Both | Interceptors, Cancel | 13 KB | ⭐⭐⭐⭐⭐ |
| **fetch** | Both | Native, Streams | 0 KB | ⭐⭐⭐⭐⭐ |
| **got** | Node | Retry, Hooks | 50 KB | ⭐⭐⭐⭐ |
| **node-fetch** | Node | Fetch API | 5 KB | ⭐⭐⭐⭐ |

### Go

| Bibliothèque | Performance | Complexité | Use Case |
|--------------|-------------|------------|----------|
| **net/http** | Bonne | Facile | Standard |
| **fasthttp** | Excellente | Moyenne | Haute perf |

## WebSocket : Bibliothèques

### Python : websockets

```python
import asyncio
import websockets

# Serveur
async def echo_handler(websocket):
    async for message in websocket:
        await websocket.send(f"Echo: {message}")

async def main():
    async with websockets.serve(echo_handler, "localhost", 8765):
        await asyncio.Future()  # Run forever

asyncio.run(main())

# Client
async def client():
    async with websockets.connect("ws://localhost:8765") as websocket:
        await websocket.send("Hello")
        response = await websocket.recv()
        print(response)

asyncio.run(client())
```

### JavaScript : Socket.IO

```javascript
// Serveur
const io = require('socket.io')(3000);

io.on('connection', (socket) => {
  console.log('Client connected');

  socket.on('message', (data) => {
    console.log('Received:', data);
    socket.emit('message', `Echo: ${data}`);
  });

  socket.on('disconnect', () => {
    console.log('Client disconnected');
  });
});

// Client
const io = require('socket.io-client');
const socket = io('http://localhost:3000');

socket.on('connect', () => {
  socket.emit('message', 'Hello');
});

socket.on('message', (data) => {
  console.log('Received:', data);
});
```

## Best Practices

### 1. Toujours définir des timeouts

```python
# ❌ MAUVAIS
response = requests.get('https://api.example.com/data')

# ✅ BON
response = requests.get('https://api.example.com/data', timeout=5.0)

# ✅ MEILLEUR : Timeouts différenciés
response = requests.get(
    'https://api.example.com/data',
    timeout=(3.0, 10.0)  # (connect timeout, read timeout)
)
```

### 2. Utiliser des sessions (connection pooling)

```python
# ❌ MAUVAIS : Nouvelle connexion à chaque fois
for i in range(100):
    response = requests.get(f'https://api.example.com/items/{i}')

# ✅ BON : Réutilisation de connexions
session = requests.Session()
for i in range(100):
    response = session.get(f'https://api.example.com/items/{i}')
```

### 3. Gérer les erreurs correctement

```python
# ✅ BON
try:
    response = requests.get('https://api.example.com/data', timeout=5)
    response.raise_for_status()
    data = response.json()
except requests.Timeout:
    # Spécifique timeout
    logger.error("Request timed out")
except requests.HTTPError as e:
    # 4xx, 5xx
    logger.error(f"HTTP error: {e.response.status_code}")
except requests.RequestException as e:
    # Autres erreurs réseau
    logger.error(f"Network error: {e}")
```

### 4. Retry avec backoff

```python
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# Configuration retry
retry_strategy = Retry(
    total=3,
    backoff_factor=1,  # 1s, 2s, 4s
    status_forcelist=[429, 500, 502, 503, 504],
    allowed_methods=["HEAD", "GET", "OPTIONS"]
)

adapter = HTTPAdapter(max_retries=retry_strategy)
session = requests.Session()
session.mount("https://", adapter)
session.mount("http://", adapter)

# Les requêtes retryent automatiquement
response = session.get('https://api.example.com/data')
```

### 5. Streaming pour gros fichiers

```python
# ❌ MAUVAIS : Charge tout en mémoire
response = requests.get('https://example.com/bigfile.zip')
with open('file.zip', 'wb') as f:
    f.write(response.content)  # 1 GB en RAM !

# ✅ BON : Stream
response = requests.get('https://example.com/bigfile.zip', stream=True)
with open('file.zip', 'wb') as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)
```

### 6. Headers appropriés

```python
headers = {
    'User-Agent': 'MyApp/1.0',  # Identification
    'Accept': 'application/json',  # Type de réponse souhaité
    'Accept-Encoding': 'gzip, deflate',  # Compression
}

response = requests.get('https://api.example.com/data', headers=headers)
```

### 7. Authentification sécurisée

```python
# ❌ MAUVAIS : Token en clair dans URL
url = f'https://api.example.com/data?token={token}'

# ✅ BON : Token dans header
headers = {'Authorization': f'Bearer {token}'}
response = requests.get('https://api.example.com/data', headers=headers)

# ✅ BON : Basic auth
response = requests.get(
    'https://api.example.com/data',
    auth=('username', 'password')
)
```

## Migration entre bibliothèques

### requests → httpx

```python
# Avant (requests)
import requests

response = requests.get('https://api.example.com/data', timeout=5)
data = response.json()

# Après (httpx sync)
import httpx

response = httpx.get('https://api.example.com/data', timeout=5)
data = response.json()

# Ou async
async def fetch():
    async with httpx.AsyncClient() as client:
        response = await client.get('https://api.example.com/data')
        return response.json()
```

### urllib → requests

```python
# Avant (urllib)
import urllib.request
import json

req = urllib.request.Request('https://api.example.com/data')
response = urllib.request.urlopen(req)
data = json.loads(response.read().decode('utf-8'))

# Après (requests)
import requests

response = requests.get('https://api.example.com/data')
data = response.json()
```

## Récapitulatif

### Choix rapide

```
Besoin                          → Bibliothèque
───────────────────────────────────────────────
Python sync simple              → requests
Python async moderne            → httpx
Python async haute perf         → aiohttp
Python contrôle bas niveau      → urllib3

JavaScript navigateur           → fetch (natif)
JavaScript Node.js              → axios ou got
JavaScript async/await          → fetch ou axios

Go standard                     → net/http
Go haute performance            → fasthttp

WebSocket Python                → websockets
WebSocket JavaScript            → Socket.IO
```

### Checklist de sélection

```python
# Posez-vous ces questions :

1. Sync ou Async ?
   - Sync simple : requests
   - Async : httpx, aiohttp

2. HTTP/2 nécessaire ?
   - Oui : httpx
   - Non : requests OK

3. WebSocket ?
   - Python : websockets, aiohttp
   - JS : Socket.IO

4. Performance critique ?
   - Python : aiohttp
   - Go : fasthttp

5. Simplicité prioritaire ?
   - Python : requests
   - JS : axios

6. Communauté importante ?
   - Python : requests
   - JS : axios, fetch
```

## Prochaines étapes

Maintenant que vous connaissez les bibliothèques modernes :

- **Section 8.13** : Conception d'applications client-serveur
- **Section 8.14** : Implications réseau pour les microservices

---


⏭️ [Conception d'applications client-serveur](/08-programmation-reseau/13-conception-client-serveur.md)
