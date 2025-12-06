🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.4 HTTP/HTTPS - Le protocole du Web

## Introduction

HTTP (HyperText Transfer Protocol) et sa variante sécurisée HTTPS (HTTP Secure) sont sans doute les protocoles applicatifs les plus utilisés au monde. **Chaque fois que vous consultez une page web, téléchargez une image, streamez une vidéo, utilisez une API REST, ou même simplement vérifiez vos emails via une interface web, vous utilisez HTTP ou HTTPS.**

Ces protocoles sont le fondement du World Wide Web tel que nous le connaissons aujourd'hui. Ils définissent comment les clients (généralement des navigateurs) et les serveurs web communiquent pour échanger des ressources : pages HTML, images, vidéos, données JSON, fichiers PDF, et bien plus encore.

## Contexte historique

### Les débuts : HTTP/0.9 (1991)

Le tout premier protocole HTTP a été créé par **Tim Berners-Lee** au CERN en 1991. Il était extrêmement simple :

```
Requête client :
GET /page.html

Réponse serveur :
<html>
  <body>Contenu de la page</body>
</html>
[connexion fermée]
```

**Caractéristiques d'HTTP/0.9 :**
- Une seule méthode : `GET`
- Pas d'en-têtes HTTP
- Uniquement des fichiers HTML
- Connexion fermée après chaque requête

### L'évolution majeure : HTTP/1.0 (1996)

HTTP/1.0 a introduit des concepts essentiels :
- Autres méthodes : `POST`, `HEAD`
- En-têtes HTTP (headers)
- Codes de statut (200, 404, 500, etc.)
- Support de différents types de contenu (images, vidéos, etc.)

### La standardisation : HTTP/1.1 (1997)

HTTP/1.1 est devenu le standard pendant près de 20 ans :
- Connexions persistantes par défaut
- Pipelining
- Chunked transfer encoding
- Meilleurs mécanismes de cache
- Support des hôtes virtuels

### L'ère moderne : HTTP/2 (2015) et HTTP/3 (2022)

Ces versions ont apporté des améliorations majeures de performance que nous explorerons dans les sous-sections dédiées.

## HTTP : Les concepts fondamentaux

### 1. Un protocole requête-réponse

HTTP fonctionne selon un modèle simple : **requête → réponse**

```
┌──────────┐                           ┌──────────┐
│  Client  │                           │  Serveur │
│(Browser) │                           │   Web    │
└────┬─────┘                           └────┬─────┘
     │                                      │
     │──── Requête HTTP ───────────────────>│
     │     GET /index.html HTTP/1.1         │
     │     Host: www.example.com            │
     │                                      │
     │<──── Réponse HTTP ───────────────────│
     │     HTTP/1.1 200 OK                  │
     │     Content-Type: text/html          │
     │     [contenu HTML]                   │
     │                                      │
```

**Points importants :**
- Le client initie toujours la communication (le serveur ne peut pas envoyer de données non sollicitées en HTTP classique)
- Chaque requête est indépendante (HTTP est "sans état" ou "stateless")
- Une page web nécessite généralement plusieurs requêtes HTTP (HTML, CSS, JavaScript, images, etc.)

### 2. Un protocole textuel (HTTP/1.x)

HTTP/1.0 et HTTP/1.1 sont des protocoles **textuels**, ce qui les rend facilement lisibles par un humain :

**Exemple de requête HTTP complète :**
```http
GET /api/users/123 HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
Accept-Language: fr-FR,fr;q=0.9,en;q=0.8
Connection: keep-alive
```

**Exemple de réponse HTTP complète :**
```http
HTTP/1.1 200 OK
Date: Sat, 06 Dec 2025 14:30:00 GMT
Server: nginx/1.18.0
Content-Type: application/json; charset=utf-8
Content-Length: 157
Connection: keep-alive

{
  "id": 123,
  "name": "Jean Dupont",
  "email": "jean.dupont@example.com",
  "created_at": "2024-01-15T10:30:00Z"
}
```

### 3. Structure d'un message HTTP

Chaque message HTTP (requête ou réponse) suit une structure bien définie :

```
┌─────────────────────────────────────┐
│  Ligne de départ                    │  ← GET /index.html HTTP/1.1
│  (Request line ou Status line)      │    ou HTTP/1.1 200 OK
├─────────────────────────────────────┤
│  En-têtes HTTP (Headers)            │  ← Host: example.com
│  Paires clé: valeur                 │    Content-Type: text/html
│  Une par ligne                      │    Content-Length: 1234
│                                     │    [autres en-têtes...]
├─────────────────────────────────────┤
│  Ligne vide                         │  ← CRLF (obligatoire)
├─────────────────────────────────────┤
│  Corps du message (Body)            │  ← Contenu HTML, JSON, image...
│  (optionnel)                        │    (absent dans les requêtes GET)
└─────────────────────────────────────┘
```

### 4. HTTP est sans état (Stateless)

Un concept fondamental : **HTTP ne conserve aucune mémoire entre les requêtes.**

**Problème concret :**
```
Requête 1 : GET /login → Connexion réussie
Requête 2 : GET /profile → Le serveur ne sait pas qui vous êtes !
```

**Solutions développées :**
- **Cookies** : Petits fichiers stockés côté client
- **Sessions** : État maintenu côté serveur, identifié par un ID de session
- **Tokens** : JWT, OAuth tokens pour les APIs

**Exemple avec cookies :**
```
1. Login
   Client → Serveur : POST /login (identifiants)
   Serveur → Client : Set-Cookie: session_id=abc123

2. Requête suivante
   Client → Serveur : GET /profile
                      Cookie: session_id=abc123
   Le serveur reconnaît l'utilisateur grâce au cookie
```

### 5. HTTP utilise TCP

HTTP (versions 1.x et 2) s'appuie sur **TCP** comme protocole de transport :

```
Couche Application  :  HTTP/HTTPS
       ↓
Couche Transport    :  TCP (port 80 pour HTTP, port 443 pour HTTPS)
       ↓
Couche Internet     :  IP
       ↓
Couche Accès réseau :  Ethernet, Wi-Fi, etc.
```

**Pourquoi TCP ?**
- ✅ Fiabilité : Les données arrivent dans l'ordre, sans perte
- ✅ Contrôle de flux et de congestion
- ✅ Connexion établie avant l'échange de données

**Note :** HTTP/3 rompt avec cette tradition en utilisant QUIC/UDP, mais nous verrons cela plus tard.

## HTTPS : HTTP sécurisé

### Pourquoi HTTPS ?

HTTP en clair présente des **risques majeurs** :

❌ **Confidentialité** : Toutes les données sont visibles en clair sur le réseau
```
Attaquant sur le réseau peut voir :
- Vos identifiants de connexion
- Vos données bancaires
- Vos conversations
- Votre historique de navigation
```

❌ **Intégrité** : Les données peuvent être modifiées en transit
```
Un attaquant peut :
- Modifier le contenu d'une page web
- Injecter du code malveillant
- Rediriger vers un site frauduleux
```

❌ **Authentification** : Pas de garantie sur l'identité du serveur
```
Vous pensez communiquer avec votre banque,
mais c'est peut-être un site frauduleux
```

### HTTPS = HTTP + TLS/SSL

HTTPS n'est pas un protocole différent, c'est **HTTP encapsulé dans une couche de chiffrement TLS (Transport Layer Security)**.

```
┌────────────────────────────────────────┐
│        Application (navigateur)        │
├────────────────────────────────────────┤
│              HTTP                      │  ← Communication en clair
├────────────────────────────────────────┤
│              TLS/SSL                   │  ← Chiffrement, authentification
├────────────────────────────────────────┤
│              TCP (port 443)            │
├────────────────────────────────────────┤
│              IP                        │
└────────────────────────────────────────┘
```

### Les bénéfices de HTTPS

✅ **Confidentialité** : Toutes les données sont chiffrées
```
Un attaquant voit seulement :
- Que vous communiquez avec example.com (adresse IP)
- La quantité de données échangées
MAIS PAS le contenu des données
```

✅ **Intégrité** : Les données ne peuvent pas être modifiées sans détection
```
TLS détecte toute altération des données en transit
```

✅ **Authentification** : Vérification de l'identité du serveur via certificats
```
Le certificat prouve que vous communiquez bien avec
le véritable example.com et pas un imposteur
```

### Exemple visuel : HTTP vs HTTPS

**Avec HTTP (port 80) :**
```
Vous → [Mot de passe: secret123] → Routeur → Internet → Serveur
       ↑
       Visible en clair à chaque point du réseau !
```

**Avec HTTPS (port 443) :**
```
Vous → [Données chiffrées: 8f3a9c...] → Routeur → Internet → Serveur
       ↑
       Impossible à déchiffrer sans la clé
```

## Comment reconnaître HTTPS dans un navigateur

Les navigateurs modernes vous indiquent clairement si une connexion est sécurisée :

```
✅ HTTPS sécurisé :
🔒 https://www.example.com
   ↑
   Cadenas indiquant une connexion chiffrée

❌ HTTP non sécurisé :
⚠️ http://www.example.com
   ↑
   Avertissement "Non sécurisé"
```

**Depuis 2018**, Chrome et Firefox marquent explicitement les sites HTTP comme "Non sécurisé" pour encourager l'adoption de HTTPS.

## Le processus de connexion HTTPS

Quand vous accédez à un site HTTPS, voici ce qui se passe :

```
1. Établissement de la connexion TCP
   Client ←─── 3-way handshake ───→ Serveur
   (SYN, SYN-ACK, ACK)

2. Handshake TLS
   Client ←─── Négociation TLS ───→ Serveur
   - Négociation de la version TLS
   - Choix des algorithmes de chiffrement
   - Échange de clés
   - Vérification du certificat

3. Communication HTTP chiffrée
   Client ←─── Données chiffrées ───→ Serveur
   Toutes les requêtes/réponses HTTP sont maintenant chiffrées
```

**Temps typique :**
- TCP handshake : ~30-100ms (dépend de la latence réseau)
- TLS handshake : ~30-100ms supplémentaires
- Total : ~60-200ms avant de pouvoir envoyer la première requête HTTP

## Cas d'usage concrets

### 1. Navigation web classique

**Scénario : Vous visitez un blog**

```
1. Vous tapez : https://blog.example.com
2. DNS : Résolution de blog.example.com → 192.0.2.10
3. TCP : Connexion au port 443 (HTTPS)
4. TLS : Handshake, vérification du certificat
5. HTTP : GET / HTTP/1.1
6. Serveur : Retourne la page HTML
7. Browser : Parse le HTML, trouve des références à :
   - /style.css
   - /script.js
   - /images/logo.png
8. HTTP : Nouvelles requêtes pour chaque ressource
9. Browser : Affiche la page complète
```

**Nombre typique de requêtes pour une page moderne : 50-150 !**

### 2. API REST

**Scénario : Une application mobile récupère des données**

```javascript
// L'application fait une requête HTTP
GET https://api.example.com/v1/products?category=electronics

// Requête HTTP générée :
GET /v1/products?category=electronics HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGc...

// Réponse HTTP :
HTTP/1.1 200 OK
Content-Type: application/json

[
  {"id": 1, "name": "Laptop", "price": 999},
  {"id": 2, "name": "Mouse", "price": 25}
]
```

### 3. Formulaire de contact

**Scénario : Envoi d'un formulaire**

```html
<form action="/contact" method="POST">
  <input name="email" value="user@example.com">
  <textarea name="message">Bonjour...</textarea>
  <button type="submit">Envoyer</button>
</form>
```

**Requête HTTP générée :**
```http
POST /contact HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 52

email=user@example.com&message=Bonjour...
```

## Ports standards

| Protocole | Port | Description |
|-----------|------|-------------|
| HTTP | 80 | Non chiffré (de moins en moins utilisé) |
| HTTPS | 443 | Chiffré avec TLS (standard actuel) |
| HTTP alternatif | 8080, 8000, 3000 | Souvent utilisé en développement |

**Convention :**
- Si vous omettez le port dans une URL, le navigateur utilise automatiquement :
  - Port 80 pour `http://`
  - Port 443 pour `https://`

```
http://example.com        → http://example.com:80
https://example.com       → https://example.com:443
http://localhost:3000     → Port explicite (développement)
```

## URLs et URIs : Anatomie d'une adresse web

Une URL (Uniform Resource Locator) HTTP a une structure bien définie :

```
https://www.example.com:443/path/to/resource?key=value#section
│      │  │               │   │                │          │
│      │  │               │   │                │          └─ Fragment (ancre)
│      │  │               │   │                └─ Query string (paramètres)
│      │  │               │   └─ Path (chemin de la ressource)
│      │  │               └─ Port (optionnel)
│      │  └─ Sous-domaine + Domaine
│      └─ Nom de domaine complet (FQDN)
└─ Schéma (protocole)
```

**Exemples réels :**

```
https://www.youtube.com/watch?v=dQw4w9WgXcQ
│      │              │       │
│      │              │       └─ Paramètre : ID de la vidéo
│      │              └─ Chemin : endpoint /watch
│      └─ Domaine : youtube.com
└─ HTTPS sécurisé

https://github.com/search?q=machine+learning&type=repositories
│                  │      │
│                  │      └─ Paramètres de recherche
│                  └─ Endpoint de recherche
└─ HTTPS

http://192.168.1.1:8080/admin/
│                  │     │
│                  │     └─ Chemin
│                  └─ Port personnalisé
└─ HTTP (interface d'administration locale)
```

## Pourquoi HTTP est omniprésent

### Simplicité

HTTP est facile à comprendre, à implémenter et à déboguer :
```bash
# Vous pouvez même faire du HTTP à la main avec telnet ou netcat !
$ telnet example.com 80
GET / HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
...
```

### Flexibilité

HTTP peut transporter **n'importe quel type de contenu** :
- HTML, CSS, JavaScript
- Images (JPEG, PNG, GIF, WebP)
- Vidéos et audio
- Documents (PDF, Word, etc.)
- Données structurées (JSON, XML)
- Binaires (fichiers à télécharger)

Le serveur indique le type de contenu via l'en-tête `Content-Type`.

### Extensibilité

De nouveaux en-têtes HTTP peuvent être ajoutés sans casser la compatibilité :
- `X-Custom-Header: value` (headers personnalisés)
- Mécanismes de négociation de contenu
- Support de nouvelles fonctionnalités via headers

### Écosystème riche

HTTP dispose d'un écosystème mature :
- Serveurs web : Apache, Nginx, IIS, Caddy
- Bibliothèques clientes : curl, requests (Python), axios (JavaScript)
- Outils de développement : Postman, Insomnia, Chrome DevTools
- Proxys et load balancers : HAProxy, Traefik
- CDNs : Cloudflare, Akamai, AWS CloudFront

## L'évolution vers HTTPS partout

### Le mouvement "HTTPS Everywhere"

Depuis le milieu des années 2010, il y a eu une poussée massive vers l'adoption universelle de HTTPS :

**Raisons :**
1. **Sécurité** : Protection contre l'espionnage et les attaques
2. **Confiance** : Les utilisateurs font plus confiance aux sites HTTPS
3. **SEO** : Google favorise les sites HTTPS dans les résultats de recherche
4. **Fonctionnalités** : Certaines APIs modernes (géolocalisation, caméra, service workers) nécessitent HTTPS

**Facilitateurs :**
- **Let's Encrypt** (2016) : Certificats SSL/TLS gratuits et automatisés
- **Chrome/Firefox** : Avertissements "Non sécurisé" pour HTTP
- **HTTP/2** : Nécessite HTTPS dans les navigateurs

**Statistiques :**
- 2015 : ~40% du trafic web en HTTPS
- 2020 : ~80% du trafic web en HTTPS
- 2025 : >95% du trafic web en HTTPS

## Limitations et défis d'HTTP

Malgré son succès, HTTP présente certains défis :

### 1. Latence

Chaque requête HTTP nécessite un aller-retour réseau :
```
Client → Serveur : Requête (RTT/2)
Serveur → Client : Réponse (RTT/2)
Total : 1 RTT (Round-Trip Time)
```

Pour une page avec 100 ressources, cela peut devenir très lent.

**Solutions :** HTTP/2 et HTTP/3 (que nous verrons dans les sous-sections suivantes)

### 2. Head-of-line blocking (HTTP/1.1)

Sur une connexion TCP, les requêtes doivent être traitées séquentiellement :
```
Requête 1 (grosse image) ──────────────────>
Requête 2 (petit CSS)    attend... ───────>
Requête 3 (JavaScript)   attend... ──────>
```

**Solutions :** HTTP/2 avec multiplexage

### 3. Overhead des headers

HTTP/1.1 envoie tous les headers en texte clair, souvent répétitifs :
```http
GET /page1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0...
Accept: text/html...
Cookie: session=abc123...
(2-4 KB de headers !)

GET /page2 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0...  ← Identique !
Accept: text/html...        ← Identique !
Cookie: session=abc123...   ← Identique !
```

**Solutions :** HTTP/2 avec compression HPACK

### 4. Pas de communication bidirectionnelle native

HTTP classique est unidirectionnel : client → serveur → client
Le serveur ne peut pas initier une communication.

**Solutions :**
- WebSockets (protocole différent)
- Server-Sent Events (SSE)
- Long-polling

## HTTP dans le contexte moderne

### APIs et microservices

HTTP est devenu le protocole universel pour les APIs :

```
┌───────────┐         ┌───────────┐         ┌───────────┐
│  Frontend │         │    API    │         │ Database  │
│    Web    │─HTTP─→  │  Gateway  │─HTTP─→  │  Service  │
└───────────┘         └───────────┘         └───────────┘
                             │
                             │ HTTP
                             ↓
                      ┌───────────┐
                      │   Auth    │
                      │  Service  │
                      └───────────┘
```

**REST (Representational State Transfer)** s'appuie entièrement sur HTTP.

### Mobile et IoT

Les applications mobiles communiquent principalement via HTTP/HTTPS :
- Apps iOS et Android utilisent des APIs HTTP
- Notifications push (APNs, FCM) utilisent HTTP/2
- Devices IoT communiquent souvent via HTTPS

### Cloud et conteneurs

HTTP est le protocole de communication par défaut dans le cloud :
- Kubernetes : API en HTTP
- AWS, Azure, GCP : APIs HTTP
- Docker Registry : HTTP
- Service mesh (Istio, Linkerd) : Gestion du trafic HTTP

## Bonnes pratiques HTTP

### Sécurité

✅ **Toujours utiliser HTTPS en production**
- Même pour du contenu "public"
- Évite les attaques man-in-the-middle
- Améliore la confiance et le SEO

✅ **Valider et échapper les entrées utilisateur**
- Prévient les injections
- Protection contre XSS

✅ **Utiliser les bons headers de sécurité**
```http
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
```

### Performance

✅ **Compression**
```http
Content-Encoding: gzip
```
Peut réduire la taille de 70-90%

✅ **Cache**
```http
Cache-Control: public, max-age=3600
```
Évite de retélécharger des ressources

✅ **CDN**
Distribuer le contenu géographiquement proche des utilisateurs

### Conception d'APIs

✅ **Utiliser les bonnes méthodes HTTP**
- GET pour lire
- POST pour créer
- PUT/PATCH pour modifier
- DELETE pour supprimer

✅ **Codes de statut appropriés**
- 200 OK : Succès
- 201 Created : Ressource créée
- 400 Bad Request : Erreur client
- 404 Not Found : Ressource inexistante
- 500 Internal Server Error : Erreur serveur

✅ **Versionnement**
```
https://api.example.com/v1/users
https://api.example.com/v2/users
```

## Outils pour travailler avec HTTP

### Navigateur web

**Chrome DevTools / Firefox DevTools**
- Onglet "Network" : Voir toutes les requêtes HTTP
- Inspection des headers, timing, taille
- Débogage des problèmes de chargement

### Ligne de commande

**curl** - Le couteau suisse HTTP
```bash
# Requête GET simple
curl https://api.example.com/users

# Voir les headers
curl -v https://example.com

# Requête POST avec données JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Jean","email":"jean@example.com"}'
```

**wget** - Téléchargement de fichiers
```bash
wget https://example.com/file.pdf
```

### Outils graphiques

- **Postman** : Tester des APIs
- **Insomnia** : Alternative à Postman
- **HTTPie** : curl en plus convivial

### Bibliothèques de programmation

**Python**
```python
import requests
response = requests.get('https://api.example.com/data')
print(response.json())
```

**JavaScript**
```javascript
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data));
```

**Go**
```go
resp, err := http.Get("https://api.example.com/data")
```

---

## Points clés à retenir

🔑 **HTTP est le protocole de communication du Web** - Il définit comment clients et serveurs échangent des ressources

🔑 **HTTP est un protocole requête-réponse sans état** - Chaque requête est indépendante, nécessitant des mécanismes comme les cookies pour maintenir une session

🔑 **HTTPS = HTTP + TLS** - Chiffrement, intégrité et authentification pour une communication sécurisée

🔑 **HTTP utilise TCP (sauf HTTP/3)** - Port 80 pour HTTP, port 443 pour HTTPS

🔑 **HTTP est textuel (versions 1.x) et facilement extensible** - Nouveaux headers et méthodes peuvent être ajoutés

🔑 **HTTPS est devenu le standard** - >95% du trafic web est maintenant chiffré

🔑 **HTTP a évolué** - HTTP/1.1 → HTTP/2 → HTTP/3 pour améliorer les performances

🔑 **HTTP est omniprésent** - Web, APIs REST, mobile, IoT, cloud, microservices

---

## Ce que nous allons voir ensuite

Maintenant que vous comprenez les fondamentaux d'HTTP/HTTPS, nous allons approfondir dans les sections suivantes :

- **5.4.1 Méthodes HTTP** : GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS et leurs usages
- **5.4.2 Codes de statut** : 2xx, 3xx, 4xx, 5xx et leur signification
- **5.4.3 Headers et cookies** : Les en-têtes essentiels et la gestion des cookies
- **5.4.4 HTTP/1.1** : Connexions persistantes et pipelining
- **5.4.5 HTTP/2** : Multiplexage et server push
- **5.4.6 HTTP/3 et QUIC** : La révolution UDP
- **5.4.7 Évolution des performances web** : Comparaison et optimisations

**Dans la prochaine section, nous explorerons en détail les méthodes HTTP et leur utilisation appropriée !** 👉

---

*HTTP/HTTPS est le protocole le plus visible et le plus utilisé d'Internet. Comprendre son fonctionnement vous permettra de mieux concevoir vos applications, déboguer les problèmes réseau et optimiser les performances de vos services web.*

⏭️ [Méthodes HTTP](/05-couche-application/04.1-http-methodes.md)
