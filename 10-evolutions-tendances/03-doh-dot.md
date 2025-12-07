🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.3 - DNS over HTTPS (DoH) et DNS over TLS (DoT)

## Introduction

Le DNS (Domain Name System) est l'un des protocoles les plus critiques d'Internet, traduisant les noms de domaine lisibles (example.com) en adresses IP (203.0.113.42). Mais pendant plus de 30 ans, le DNS a fonctionné **entièrement en clair**, exposant chaque recherche DNS à quiconque observe le réseau.

En 2025, deux protocoles sécurisés émergent pour résoudre ce problème :
- **DNS over TLS (DoT)** - RFC 7858 (2016)
- **DNS over HTTPS (DoH)** - RFC 8484 (2018)

Pour les développeurs, ces protocoles ne sont plus optionnels : ils deviennent essentiels pour protéger la vie privée des utilisateurs et se conformer aux réglementations (RGPD, etc.).

## Le problème du DNS classique

### DNS en clair : une fuite de métadonnées massive

**Requête DNS standard** (UDP port 53) :

```
Client                              Serveur DNS
  |                                    |
  |--- Requête DNS en CLAIR ---------->|
  |    "Quelle est l'IP de             |
  |     banking.example.com ?"         |
  |                                    |
  |<-- Réponse en CLAIR ---------------|
  |    "203.0.113.42"                  |

Problème : TOUT est visible :
- Le domaine demandé
- L'IP du client
- Le timestamp
- Le serveur DNS utilisé
```

**Qui peut voir ces requêtes ?**

```
Votre requête DNS traverse :
1. Routeur WiFi personnel
2. Box Internet (FAI)
3. Infrastructure FAI
4. Serveurs DNS (Google, Cloudflare, FAI, etc.)
5. Tous les intermédiaires réseau

Chacun peut :
- Logger toutes vos requêtes DNS
- Créer un profil de vos habitudes de navigation
- Vendre ces données
- Les transmettre aux autorités
- Les utiliser pour publicité ciblée
```

### Attaques possibles

#### 1. DNS Spoofing (empoisonnement)

```
Attaquant sur le réseau local (WiFi café) :

Client                    Attaquant               Serveur DNS légitime
  |                           |                           |
  |--- DNS: bank.com ? -----> |                           |
  |                           |                           |
  |<-- Réponse falsifiée ---- |                           |
  |    "bank.com = 192.0.2.1" |                           |
  |    (serveur de l'attaquant)                           |
  |                                                       |
  |--- HTTPS vers 192.0.2.1 (phishing) -----------------> |

Résultat : Client sur un faux site bancaire
```

#### 2. DNS Hijacking par FAI

```
Certains FAI redirigent les erreurs DNS :

User tape : "fbbok.com" (typo)
  |
  |--- DNS: fbbok.com ? ---> FAI
  |
  |<-- Au lieu de NXDOMAIN ---
  |    Retourne IP page pub FAI

Résultat : Monétisation de vos erreurs de frappe
```

#### 3. Surveillance et censure

```
Gouvernement/FAI peut :
- Logger toutes requêtes DNS
- Bloquer l'accès à certains domaines
- Créer profils de navigation
- Vendre données à des tiers

Exemple : dns.query("torrent-site.com")
→ DNS retourne NXDOMAIN (bloqué)
ou → Ajoute utilisateur à une liste de surveillance
```

#### 4. DNS Tunneling (exfiltration de données)

```
Malware utilise DNS pour exfiltrer données :

Données secrètes encodées dans sous-domaines :
d4t4t0st34l.c0nf1d3nt14l.attacker.com

Passe souvent inaperçu car DNS rarement inspecté
```

### Statistiques d'exposition

```
Utilisateur typique génère :
- 500-1000 requêtes DNS/jour
- 300 000-400 000 requêtes/an

Chaque requête révèle :
- Sites visités
- Services utilisés (email, cloud, streaming)
- Applications mobiles actives
- Horaires d'activité

Même avec HTTPS, DNS révèle TOUT
```

**Exemple concret** :

```
Requêtes DNS observées pour un utilisateur :
09:00 - mail.protonmail.com
09:05 - linkedin.com
09:30 - docs.google.com
12:15 - ubereats.com
13:00 - netflix.com
18:00 - dating-app.com
20:00 - torrent-tracker.org
23:00 - adult-content.com

→ Profil complet sans voir le contenu HTTPS
```

## DNS over TLS (DoT)

### Principe

DoT encapsule les requêtes DNS dans une connexion TLS, comme HTTPS fait pour HTTP.

```
DNS classique :
Client → [Requête DNS en clair] → Port 53 UDP → Serveur

DNS over TLS :
Client → [Connexion TLS port 853] → [Requête DNS chiffrée] → Serveur
```

**RFC 7858** (2016) définit DoT sur **port 853 TCP**.

### Handshake DoT

```
Client                                    Serveur DNS (port 853)
  |                                            |
  |--- TCP SYN ------------------------------> |
  |<-- TCP SYN-ACK ----------------------------|
  |--- TCP ACK ------------------------------> |
  |                                            |
  |--- TLS ClientHello -------------------->   |
  |<-- TLS ServerHello + Certificate --------- |
  |--- TLS Finished ----------------------->   |
  |<-- TLS Finished ---------------------------|
  |                                            |
  | === Connexion TLS établie et chiffrée ===  |
  |                                            |
  |--- Requête DNS (chiffrée) ------------->   |
  |    "banking.example.com ?"                 |
  |                                            |
  |<-- Réponse DNS (chiffrée) -----------------|
  |    "203.0.113.42"                          |
```

### Avantages de DoT

✅ **Chiffrement complet** : Requêtes et réponses invisibles pour observateurs

✅ **Authentification serveur** : Certificat TLS prouve l'identité du serveur DNS

✅ **Intégrité** : Impossible de modifier les réponses en transit

✅ **Port dédié (853)** : Facile à identifier et router

### Implémentation DoT

**Python avec dns.query** :

```python
import dns.query
import dns.message
import ssl

def query_dot(domain, server='1.1.1.1'):
    """
    Requête DNS over TLS vers Cloudflare
    """
    # Créer la requête DNS
    query = dns.message.make_query(domain, dns.rdatatype.A)

    # Contexte SSL/TLS
    ssl_context = ssl.create_default_context()
    ssl_context.check_hostname = True

    # Requête DoT sur port 853
    response = dns.query.tls(
        query,
        server,
        port=853,
        ssl_context=ssl_context,
        timeout=5
    )

    # Parser la réponse
    for answer in response.answer:
        for item in answer.items:
            if item.rdtype == dns.rdatatype.A:
                return str(item)

    return None

# Utilisation
ip = query_dot('example.com', '1.1.1.1')  # Cloudflare
print(f"IP: {ip}")
```

**Configuration système (Linux)** :

```bash
# systemd-resolved avec DoT
sudo nano /etc/systemd/resolved.conf

[Resolve]
DNS=1.1.1.1 1.0.0.1
DNSOverTLS=yes

sudo systemctl restart systemd-resolved
```

**Configuration Android (natif depuis Android 9)** :

```
Settings → Network & Internet → Private DNS
Entrer : one.one.one.one (Cloudflare)
ou : dns.google
```

**Stubby (proxy DoT local)** :

```yaml
# /etc/stubby/stubby.yml
resolution_type: GETDNS_RESOLUTION_STUB

dns_transport_list:
  - GETDNS_TRANSPORT_TLS

upstream_recursive_servers:
  - address_data: 1.1.1.1
    tls_auth_name: "cloudflare-dns.com"
  - address_data: 8.8.8.8
    tls_auth_name: "dns.google"

tls_authentication: GETDNS_AUTHENTICATION_REQUIRED
tls_query_padding_blocksize: 128
```

```bash
sudo stubby -C /etc/stubby/stubby.yml
# Stubby écoute sur 127.0.0.1:53
# Forwarde en DoT vers 1.1.1.1:853
```

## DNS over HTTPS (DoH)

### Principe

DoH encapsule les requêtes DNS dans des requêtes HTTPS standard, indiscernables du trafic web normal.

```
DNS classique :
Client → [DNS sur port 53] → Serveur

DNS over HTTPS :
Client → [HTTPS GET/POST sur port 443] → Endpoint /dns-query → Serveur
```

**RFC 8484** (2018) définit DoH sur **port 443 HTTPS**.

### Format DoH

DoH utilise deux méthodes :

#### 1. GET avec query parameter

```http
GET /dns-query?dns=AAABAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB HTTP/2
Host: dns.cloudflare.com
Accept: application/dns-message
```

La requête DNS est encodée en base64url dans le paramètre `dns`.

#### 2. POST avec body

```http
POST /dns-query HTTP/2
Host: dns.cloudflare.com
Content-Type: application/dns-message
Content-Length: 33

[binary DNS message]
```

Le corps contient directement le message DNS binaire.

### Avantages de DoH

✅ **Indiscernable du trafic HTTPS** : Impossible de bloquer sans bloquer tout HTTPS

✅ **Utilise infrastructure HTTPS existante** : Port 443, CDN, caching HTTP

✅ **Passe les firewalls** : Traité comme du trafic web normal

✅ **Support HTTP/2 et HTTP/3** : Multiplexage, 0-RTT

✅ **Intégration navigateur** : Firefox, Chrome supportent nativement

### Implémentation DoH

**Python avec requests** :

```python
import requests
import dns.message
import base64

def query_doh(domain, server='https://cloudflare-dns.com/dns-query'):
    """
    Requête DNS over HTTPS
    """
    # Créer requête DNS
    query = dns.message.make_query(domain, dns.rdatatype.A)
    query_wire = query.to_wire()

    # Méthode 1 : GET
    query_b64 = base64.urlsafe_b64encode(query_wire).decode('utf-8').rstrip('=')

    response = requests.get(
        server,
        params={'dns': query_b64},
        headers={'Accept': 'application/dns-message'},
        timeout=5
    )

    if response.status_code == 200:
        # Parser réponse DNS
        dns_response = dns.message.from_wire(response.content)

        for answer in dns_response.answer:
            for item in answer.items:
                if item.rdtype == dns.rdatatype.A:
                    return str(item)

    return None

# Utilisation
ip = query_doh('example.com')
print(f"IP: {ip}")
```

**Python avec méthode POST** :

```python
def query_doh_post(domain, server='https://dns.google/dns-query'):
    """
    DoH avec POST (plus efficace)
    """
    query = dns.message.make_query(domain, dns.rdatatype.A)
    query_wire = query.to_wire()

    response = requests.post(
        server,
        data=query_wire,
        headers={'Content-Type': 'application/dns-message'},
        timeout=5
    )

    if response.status_code == 200:
        dns_response = dns.message.from_wire(response.content)

        for answer in dns_response.answer:
            for item in answer.items:
                if item.rdtype == dns.rdatatype.A:
                    return str(item)

    return None
```

**JavaScript (navigateur)** :

```javascript
async function queryDoH(domain) {
    // Construire requête DNS (simplifié pour exemple)
    const dnsQuery = buildDNSQuery(domain); // Fonction helper
    const base64Query = btoa(String.fromCharCode(...dnsQuery))
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');

    const response = await fetch(
        `https://cloudflare-dns.com/dns-query?dns=${base64Query}`,
        {
            headers: {
                'Accept': 'application/dns-message'
            }
        }
    );

    const buffer = await response.arrayBuffer();
    const result = parseDNSResponse(new Uint8Array(buffer));

    return result.answers[0].data; // IP address
}

// Utilisation
const ip = await queryDoH('example.com');
console.log(`IP: ${ip}`);
```

**Configuration Firefox** :

```
about:config

network.trr.mode = 2
  0 = DoH désactivé
  1 = DoH avec fallback DNS classique
  2 = DoH préféré, fallback DNS si échec
  3 = DoH uniquement (strict)

network.trr.uri = https://mozilla.cloudflare-dns.com/dns-query
```

**Configuration Chrome** :

```
chrome://settings/security

Section "Advanced" → "Use secure DNS"
→ Choisir provider (Google, Cloudflare, etc.)
ou entrer URL custom
```

**Node.js avec dohjs** :

```javascript
const { Resolver } = require('dns-over-https');

const resolver = new Resolver({
    provider: 'cloudflare' // ou 'google', 'quad9'
});

async function resolveDomain(domain) {
    try {
        const addresses = await resolver.resolve4(domain);
        console.log(`${domain} → ${addresses.join(', ')}`);
        return addresses;
    } catch (error) {
        console.error(`Error resolving ${domain}:`, error);
    }
}

resolveDomain('example.com');
```

## DoT vs DoH : comparaison

### Tableau comparatif

| Critère | DoT | DoH |
|---------|-----|-----|
| **Port** | 853 (dédié) | 443 (HTTPS standard) |
| **Transport** | TLS direct | HTTPS (TLS + HTTP) |
| **Visibilité** | Identifiable (port 853) | Invisible (mélangé au trafic web) |
| **Performance** | Légèrement plus rapide | Overhead HTTP |
| **Caching** | Cache DNS uniquement | Peut utiliser cache HTTP |
| **Firewall** | Peut être bloqué (port 853) | Difficile à bloquer (port 443) |
| **Adoption navigateurs** | Limitée | Chrome, Firefox, Edge, Safari |
| **Support OS** | Android 9+, iOS 14+ | Via navigateur |
| **Standardisation** | RFC 7858 (2016) | RFC 8484 (2018) |
| **Complexité** | Simple (juste TLS) | Plus complexe (HTTP + TLS) |

### Performance comparée

```
Benchmark (moyenne 1000 requêtes) :

DNS classique (UDP) :
- Latence : 15ms
- Bande passante : 100 bytes/requête

DoT (TLS sur TCP) :
- Latence première requête : 45ms (handshake TLS)
- Latence requêtes suivantes : 18ms (connexion réutilisée)
- Bande passante : 120 bytes/requête
- Overhead : +20%

DoH (HTTPS) :
- Latence première requête : 50ms (HTTPS handshake)
- Latence requêtes suivantes : 22ms (HTTP/2 multiplexing)
- Bande passante : 180 bytes/requête (headers HTTP)
- Overhead : +80%
- Avec HTTP/3 : 20ms (amélioration)

Conclusion : DoT plus performant, DoH plus résilient
```

### Cas d'usage recommandés

**Utiliser DoT quand** :

```
✅ Configuration système (OS, routeur)
✅ Performance critique
✅ Contrôle infrastructure (entreprise)
✅ Environnement de confiance (pas de censure)
✅ Déploiement interne

Exemple : Entreprise configure DoT sur tous les postes
         vers serveur DNS interne avec TLS
```

**Utiliser DoH quand** :

```
✅ Application web/mobile
✅ Réseaux hostiles (censure, surveillance)
✅ Utilisation individuelle
✅ Besoin de passer les firewalls
✅ Intégration avec services web existants

Exemple : Application mobile utilise DoH pour éviter
          blocage DNS par certains FAI/gouvernements
```

## Controverses et débats

### 1. Position des FAI et entreprises

**Arguments contre DoH/DoT** :

```
FAI et entreprises argumentent :

❌ Perte de visibilité réseau
   - Impossible de bloquer malware/phishing via DNS
   - Pas de contrôle parental
   - Monitoring réseau complexifié

❌ Contournement politiques entreprise
   - Employés peuvent bypasser filtres DNS
   - Fuite de données via DNS tunneling plus difficile à détecter

❌ Fragmentation
   - Chaque app peut utiliser son propre DNS
   - Perte de cohérence réseau

❌ Performance
   - Overhead chiffrement
   - Impossibilité d'optimiser le cache DNS local
```

**Réponse des défenseurs de la vie privée** :

```
✅ La surveillance de masse n'est pas acceptable
   - DNS en clair = violation de vie privée
   - Les FAI n'ont pas besoin de voir chaque site visité

✅ Alternatives existent pour sécurité
   - Filtrage au niveau firewall (IP, DPI)
   - EDR/XDR pour entreprises
   - Contrôle parental dans l'OS/routeur

✅ Centralisation est un faux problème
   - Utilisateurs peuvent choisir leur resolver
   - Open source et décentralisé possible
```

### 2. Centralisation des resolvers

**Problème** :

```
Avant DoH/DoT :
- DNS distribué (FAI, entreprise, DNS public divers)
- Pas de concentration

Avec DoH (navigateurs) :
- Mozilla → Cloudflare par défaut
- Google Chrome → Google DNS par défaut
- ~80% du trafic DoH vers 3 providers

Risque :
- Cloudflare/Google voient énormément de requêtes DNS
- Point unique de défaillance
- Concentration de pouvoir
```

**Cloudflare voit** :

```
Avec DoH activé dans Firefox :
- Tous les domaines visités par utilisateurs Firefox
- Timestamps précis
- Corrélation possible avec IPs clients

Cloudflare promet :
- Pas de logging
- Suppression données après 24h
- Audits indépendants

Mais reste un point de confiance centralisé
```

### 3. Contournement censure vs sécurité nationale

**Tension géopolitique** :

```
Cas d'usage positif :
- Journalistes en régimes autoritaires
- Citoyens contournant censure
- Whistleblowers protégeant anonymat

Préoccupations gouvernementales :
- Difficile de bloquer contenus illégaux
- Complexifie enquêtes criminelles
- Contournement de régulations locales

Exemple concret :
Russie a tenté de bloquer DoH en 2021
→ Difficile car indiscernable du trafic HTTPS
→ Devrait bloquer tout HTTPS (impossible)
```

### 4. DNS menteur (lying DNS)

**Technique utilisée pour** :

```
Cas "légitimes" :
- Contrôle parental (bloquer adult-content.com)
- Sécurité entreprise (bloquer malware.com)
- Optimisation réseau (rediriger vers cache local)

DoH/DoT empêche ces techniques :
- Client bypasse DNS local
- Va directement vers resolver public
- Politiques locales non applicables
```

**Solution** : Firewalls applicatifs (L7) au lieu de filtrage DNS.

## Providers DNS chiffrés

### Principaux resolvers publics

| Provider | DoH | DoT | Politique logging | Filtrage |
|----------|-----|-----|-------------------|----------|
| **Cloudflare** | ✅ 1.1.1.1 | ✅ one.one.one.one | Pas de logs permanents | Aucun |
| **Cloudflare Malware** | ✅ 1.1.1.2 | ✅ | Pas de logs | Malware |
| **Cloudflare Family** | ✅ 1.1.1.3 | ✅ | Pas de logs | Malware + Adult |
| **Google** | ✅ dns.google | ✅ dns.google | Logs temporaires | Aucun |
| **Quad9** | ✅ dns.quad9.net | ✅ | Pas de logs d'IP | Malware |
| **NextDNS** | ✅ | ✅ | Configurable | Configurable |
| **AdGuard** | ✅ | ✅ | Pas de logs | Ads + tracking |

### URLs des endpoints DoH

```
Cloudflare :
https://cloudflare-dns.com/dns-query
https://1.1.1.1/dns-query

Google :
https://dns.google/dns-query
https://8.8.8.8/dns-query

Quad9 :
https://dns.quad9.net/dns-query

AdGuard :
https://dns.adguard.com/dns-query

NextDNS (avec config ID) :
https://dns.nextdns.io/abc123
```

### Addresses DoT

```
Cloudflare :
1.1.1.1:853 (one.one.one.one)
1.0.0.1:853

Google :
8.8.8.8:853 (dns.google)
8.8.4.4:853

Quad9 :
9.9.9.9:853 (dns.quad9.net)

AdGuard :
94.140.14.14:853 (dns.adguard.com)
```

## Cas d'usage développeurs

### Cas 1 : Application mobile avec DoH intégré

**Contexte** : App de messagerie privée, marché global incluant pays avec censure.

**Problème** :
- Certains gouvernements bloquent les domaines de l'app au niveau DNS
- FAI redirigent vers pages de blocage
- Utilisateurs ne peuvent pas se connecter

**Solution** :

```swift
// iOS - Configuration DoH custom
import Network

class SecureDNSManager {
    private let dohEndpoint = "https://cloudflare-dns.com/dns-query"

    func resolveWithDoH(domain: String, completion: @escaping ([String]) -> Void) {
        // Construire requête DNS
        let query = buildDNSQuery(domain: domain)
        let base64Query = query.base64EncodedString()
            .replacingOccurrences(of: "+", with: "-")
            .replacingOccurrences(of: "/", with: "_")
            .replacingOccurrences(of: "=", with: "")

        let url = URL(string: "\(dohEndpoint)?dns=\(base64Query)")!
        var request = URLRequest(url: url)
        request.setValue("application/dns-message", forHTTPHeaderField: "Accept")

        URLSession.shared.dataTask(with: request) { data, response, error in
            guard let data = data else { return }

            let addresses = self.parseDNSResponse(data)
            completion(addresses)
        }.resume()
    }

    func connect(to server: String) {
        // Résoudre avec DoH d'abord
        resolveWithDoH(domain: server) { addresses in
            guard let ip = addresses.first else { return }

            // Connecter directement à l'IP
            self.establishConnection(to: ip)
        }
    }
}
```

**Résultat** :
- App fonctionne même si DNS local bloqué
- Connexion établie via DoH → IP obtenue → connexion directe
- Taux de succès : 98% → 99.8%

### Cas 2 : Service web avec détection de DoH

**Contexte** : Site web veut optimiser en détectant si client utilise DoH.

**Analyse** :

```javascript
// Serveur Node.js - détection DoH côté serveur
const express = require('express');
const dns = require('dns').promises;

app.get('/api/detect-doh', async (req, res) => {
    const clientIP = req.ip;

    // Résoudre reverse DNS
    try {
        const hostnames = await dns.reverse(clientIP);

        // Vérifier si l'IP appartient à un resolver DoH connu
        const dohResolvers = [
            'cloudflare',
            'google',
            'quad9',
            'adguard',
            'nextdns'
        ];

        const isDoHResolver = hostnames.some(hostname =>
            dohResolvers.some(resolver => hostname.includes(resolver))
        );

        res.json({
            usingDoH: isDoHResolver,
            resolver: isDoHResolver ? hostnames[0] : null
        });
    } catch (error) {
        res.json({ usingDoH: false });
    }
});
```

**Mais** : Détection imparfaite, DoH est conçu pour être indétectable.

### Cas 3 : Entreprise avec politique DNS

**Contexte** : Entreprise veut appliquer politique DNS (bloquer certains sites) même avec DoH.

**Solution 1 : Bloquer DoH au firewall**

```bash
# Configuration firewall (iptables)
# Bloquer les resolvers DoH connus

# Cloudflare
iptables -A OUTPUT -d 1.1.1.1 -p tcp --dport 443 -j REJECT
iptables -A OUTPUT -d 1.0.0.1 -p tcp --dport 443 -j REJECT

# Google
iptables -A OUTPUT -d 8.8.8.8 -p tcp --dport 443 -j REJECT
iptables -A OUTPUT -d 8.8.4.4 -p tcp --dport 443 -j REJECT

# Etc. pour tous les resolvers connus

# Problème : liste à maintenir, nouveaux resolvers apparaissent
```

**Solution 2 : Forcer DoH vers resolver interne**

```javascript
// Configuration entreprise (GPO Windows, MDM mobile)
// Forcer utilisation du DoH interne uniquement

// Chrome Enterprise Policy
{
  "DnsOverHttpsMode": "secure",
  "DnsOverHttpsTemplates": "https://internal-doh.company.com/dns-query"
}

// Firefox Enterprise Policy
{
  "policies": {
    "DNSOverHTTPS": {
      "Enabled": true,
      "ProviderURL": "https://internal-doh.company.com/dns-query",
      "Locked": true
    }
  }
}
```

**Serveur DoH interne** :

```go
// Serveur DoH interne avec filtrage
package main

import (
    "github.com/miekg/dns"
    "net/http"
)

var blocklist = map[string]bool{
    "blocked-site.com.": true,
    "malware.example.":  true,
}

func handleDoH(w http.ResponseWriter, r *http.Request) {
    // Parser requête DNS depuis DoH
    dnsMsg := parseDNSFromDoH(r)

    question := dnsMsg.Question[0]
    domain := question.Name

    // Vérifier blocklist
    if blocklist[domain] {
        // Retourner NXDOMAIN
        dnsMsg.Response = true
        dnsMsg.Rcode = dns.RcodeNameError
    } else {
        // Forward vers resolver upstream
        response := forwardToUpstream(dnsMsg)
        dnsMsg = response
    }

    // Retourner réponse DoH
    w.Header().Set("Content-Type", "application/dns-message")
    w.Write(dnsMsg.Pack())
}

func main() {
    http.HandleFunc("/dns-query", handleDoH)
    http.ListenAndServeTLS(":443", "cert.pem", "key.pem", nil)
}
```

### Cas 4 : CDN avec optimisation DoH

**Contexte** : CDN veut réduire latence DNS en déployant DoH en edge.

**Architecture** :

```
Traditionnel :
User → FAI DNS (100ms) → CDN (50ms total: 150ms)

Avec DoH en edge :
User → Edge node local (10ms) → CDN cache (40ms total: 50ms)
                ↓
          DoH resolver intégré
          Cache DNS local
          Réponses optimisées géographiquement
```

**Implémentation Cloudflare Workers** :

```javascript
// Worker déployé en edge (200+ datacenters)
addEventListener('fetch', event => {
  event.respondWith(handleDoH(event.request))
})

async function handleDoH(request) {
  if (request.url.endsWith('/dns-query')) {
    // Extraire requête DNS
    const dnsQuery = await parseDNSQuery(request);

    // Check cache edge
    const cached = await DNS_CACHE.get(dnsQuery.domain);
    if (cached && !isExpired(cached)) {
      return new Response(cached.response, {
        headers: {'Content-Type': 'application/dns-message'}
      });
    }

    // Résoudre et optimiser géographiquement
    const response = await resolveOptimized(dnsQuery);

    // Cache en edge
    await DNS_CACHE.put(dnsQuery.domain, {
      response: response,
      ttl: response.ttl
    });

    return new Response(response, {
      headers: {'Content-Type': 'application/dns-message'}
    });
  }
}
```

**Résultat** :
- Latence DNS réduite de 60%
- Cache hit rate : 85%
- Trafic vers resolver central : -80%

### Cas 5 : Application privacy-focused

**Contexte** : App de navigation anonyme type Tor Browser.

**Exigences** :
- Ne jamais révéler les domaines visités
- Résister aux attaques de timing
- Pas de corrélation possible

**Solution : DoH avec padding et mixing**

```python
import random
import time
from cryptography.hazmat.primitives import padding

class PrivacyDNSResolver:
    def __init__(self):
        self.doh_endpoint = "https://dns.quad9.net/dns-query"
        self.padding_sizes = [32, 64, 128, 256]  # Tailles standard

    def resolve_private(self, domain):
        """
        Résolution DNS avec protection privacy avancée
        """
        # 1. Construire requête DNS
        query = self.build_dns_query(domain)

        # 2. Ajouter padding aléatoire (RFC 8467)
        padded_query = self.add_random_padding(query)

        # 3. Ajouter délai aléatoire (résister timing attacks)
        time.sleep(random.uniform(0.01, 0.05))

        # 4. Mélanger avec requêtes dummy
        if random.random() < 0.3:  # 30% du temps
            self.send_dummy_query()

        # 5. Envoyer requête DoH
        response = self.send_doh_request(padded_query)

        # 6. Parser et retourner
        return self.parse_dns_response(response)

    def add_random_padding(self, query):
        """
        Ajoute padding pour masquer la taille réelle
        """
        # Choisir taille cible aléatoire
        target_size = random.choice(self.padding_sizes)

        # Ajouter padding pour atteindre taille
        padding_needed = target_size - len(query)
        if padding_needed > 0:
            query += b'\x00' * padding_needed

        return query

    def send_dummy_query(self):
        """
        Envoie requête factice pour masquer le pattern
        """
        dummy_domains = [
            'example.com',
            'cloudflare.com',
            'google.com',
            'microsoft.com'
        ]
        dummy = random.choice(dummy_domains)
        # Envoyer mais ignorer résultat
        self.send_doh_request(self.build_dns_query(dummy))
```

**Techniques additionnelles** :

```python
# Rotation des resolvers
class MultiResolverRotation:
    def __init__(self):
        self.resolvers = [
            'https://dns.quad9.net/dns-query',
            'https://dns.adguard.com/dns-query',
            'https://doh.opendns.com/dns-query'
        ]
        self.current = 0

    def get_next_resolver(self):
        """
        Rotation pour éviter corrélation par single resolver
        """
        resolver = self.resolvers[self.current]
        self.current = (self.current + 1) % len(self.resolvers)
        return resolver
```

## Performance et optimisation

### Caching stratégique

```python
import time
from functools import lru_cache

class CachedDoHResolver:
    def __init__(self):
        self.cache = {}  # {domain: (ip, expiry)}

    def resolve(self, domain):
        # Check cache
        if domain in self.cache:
            ip, expiry = self.cache[domain]
            if time.time() < expiry:
                return ip  # Cache hit

        # Cache miss : requête DoH
        ip = self.query_doh(domain)
        ttl = 300  # 5 minutes

        # Store in cache
        self.cache[domain] = (ip, time.time() + ttl)

        return ip

    def query_doh(self, domain):
        # Implémentation DoH...
        pass
```

### Connection pooling

```python
import requests

class PooledDoHResolver:
    def __init__(self):
        # Session réutilisable (TCP connection pooling)
        self.session = requests.Session()

        # HTTP/2 pour multiplexing
        from requests.adapters import HTTPAdapter
        adapter = HTTPAdapter(
            pool_connections=10,
            pool_maxsize=20
        )
        self.session.mount('https://', adapter)

    def resolve(self, domain):
        query = self.build_query(domain)

        # Réutilise connexion existante
        response = self.session.post(
            'https://cloudflare-dns.com/dns-query',
            data=query,
            headers={'Content-Type': 'application/dns-message'}
        )

        return self.parse_response(response.content)
```

**Gain** :

```
Sans pooling :
- Chaque requête : TCP handshake + TLS handshake
- Latence : 50-100ms

Avec pooling :
- Première requête : 50-100ms
- Requêtes suivantes : 15-25ms (réutilise connexion)
- Gain : 60-70%
```

### Prefetching DNS

```javascript
// Client web : prefetch des domaines probables
class SmartDNSPrefetcher {
    constructor() {
        this.resolver = new DoHResolver();
        this.prefetchQueue = new Set();
    }

    analyzePage() {
        // Extraire tous les domaines de la page
        const links = document.querySelectorAll('a[href]');

        links.forEach(link => {
            const url = new URL(link.href);
            const domain = url.hostname;

            // Prefetch en arrière-plan
            if (!this.prefetchQueue.has(domain)) {
                this.prefetchQueue.add(domain);
                this.resolver.resolve(domain);
            }
        });
    }

    onMouseOver(element) {
        // Prefetch agressif au survol
        const url = new URL(element.href);
        this.resolver.resolve(url.hostname);
    }
}

// Utilisation
const prefetcher = new SmartDNSPrefetcher();
prefetcher.analyzePage();

// Au survol des liens
document.querySelectorAll('a').forEach(link => {
    link.addEventListener('mouseover', () => {
        prefetcher.onMouseOver(link);
    });
});
```

### Parallel resolution

```go
// Résolution parallèle de multiples domaines
func resolveMultiple(domains []string) map[string]string {
    results := make(map[string]string)
    resultsChan := make(chan struct{domain, ip string})

    // Lancer résolution en parallèle
    for _, domain := range domains {
        go func(d string) {
            ip := queryDoH(d)
            resultsChan <- struct{domain, ip string}{d, ip}
        }(domain)
    }

    // Collecter résultats
    for range domains {
        result := <-resultsChan
        results[result.domain] = result.ip
    }

    return results
}

// Utilisation
domains := []string{
    "cdn1.example.com",
    "cdn2.example.com",
    "api.example.com",
}

// Résout les 3 en parallèle au lieu de séquentiel
ips := resolveMultiple(domains)
// Gain : 3x plus rapide
```

## Monitoring et observabilité

### Métriques à surveiller

```python
from prometheus_client import Counter, Histogram

# Compteurs
doh_requests_total = Counter(
    'doh_requests_total',
    'Total DoH requests',
    ['provider', 'status']
)

# Latences
doh_latency_seconds = Histogram(
    'doh_latency_seconds',
    'DoH query latency',
    ['provider']
)

def monitored_doh_query(domain, provider='cloudflare'):
    start = time.time()

    try:
        result = query_doh(domain, provider)
        doh_requests_total.labels(provider=provider, status='success').inc()
        return result
    except Exception as e:
        doh_requests_total.labels(provider=provider, status='error').inc()
        raise
    finally:
        duration = time.time() - start
        doh_latency_seconds.labels(provider=provider).observe(duration)
```

### Alertes

```yaml
# Prometheus alerts
groups:
  - name: doh_alerts
    rules:
      - alert: DoHHighLatency
        expr: histogram_quantile(0.95, doh_latency_seconds) > 0.5
        for: 5m
        annotations:
          summary: "DoH latency is high (p95 > 500ms)"

      - alert: DoHHighErrorRate
        expr: rate(doh_requests_total{status="error"}[5m]) > 0.1
        for: 2m
        annotations:
          summary: "DoH error rate > 10%"
```

## Sécurité et best practices

### 1. Vérifier les certificats

```python
import ssl
import requests

def secure_doh_query(domain):
    # Créer contexte SSL strict
    session = requests.Session()
    session.verify = True  # Vérifier certificats SSL

    # Ne jamais désactiver la vérification !
    # session.verify = False  # ❌ DANGEREUX

    response = session.get(
        f'https://cloudflare-dns.com/dns-query?dns={query}',
        timeout=5
    )

    return response
```

### 2. Éviter les leaks DNS

```javascript
// Vérifier que TOUTES les requêtes passent par DoH
class DNSLeakPrevention {
    constructor() {
        this.dohEnabled = true;
        this.interceptDNS();
    }

    interceptDNS() {
        // Override DNS natif du navigateur
        const originalFetch = window.fetch;

        window.fetch = async (url, options) => {
            if (!this.dohEnabled) {
                throw new Error('DoH must be enabled');
            }

            // Toutes les requêtes résolution via DoH
            const parsedUrl = new URL(url);
            await this.ensureDNSResolved(parsedUrl.hostname);

            return originalFetch(url, options);
        };
    }

    async ensureDNSResolved(hostname) {
        // Force résolution via DoH
        return await this.dohResolver.resolve(hostname);
    }
}
```

### 3. Rotation et diversification

```python
class RedundantDoHResolver:
    def __init__(self):
        self.primary = 'https://cloudflare-dns.com/dns-query'
        self.fallbacks = [
            'https://dns.google/dns-query',
            'https://dns.quad9.net/dns-query',
        ]

    def resolve_with_fallback(self, domain):
        # Essayer primary
        try:
            return self.query_doh(domain, self.primary)
        except Exception as e:
            # Fallback sur secondaires
            for fallback in self.fallbacks:
                try:
                    return self.query_doh(domain, fallback)
                except:
                    continue

            raise Exception('All DoH resolvers failed')
```

### 4. Rate limiting côté serveur

```go
// Serveur DoH avec rate limiting
import "golang.org/x/time/rate"

type DoHServer struct {
    limiters map[string]*rate.Limiter
}

func (s *DoHServer) handleDoHRequest(w http.ResponseWriter, r *http.Request) {
    clientIP := getClientIP(r)

    // Rate limiter : 100 requêtes/seconde par IP
    limiter := s.getLimiter(clientIP)

    if !limiter.Allow() {
        http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
        return
    }

    // Traiter requête DNS...
}

func (s *DoHServer) getLimiter(ip string) *rate.Limiter {
    if limiter, exists := s.limiters[ip]; exists {
        return limiter
    }

    limiter := rate.NewLimiter(100, 200) // 100 req/s, burst 200
    s.limiters[ip] = limiter
    return limiter
}
```

## État de l'adoption en 2025

### Statistiques globales

```
Navigateurs supportant DoH :
- Chrome/Edge : 90% des utilisateurs
- Firefox : 95% des utilisateurs
- Safari : 80% des utilisateurs (iOS 14+, macOS 11+)
- Brave : 100% (DoH par défaut)

Total : ~85% des utilisateurs peuvent utiliser DoH

Activation par défaut :
- Firefox : Oui (US) depuis 2020
- Chrome : Opt-in
- Safari : Opt-in
- Edge : Opt-in

Trafic DNS chiffré (estimation) :
- 2020 : ~5% du trafic DNS
- 2023 : ~15%
- 2025 : ~25%

Croissance : +5-8% par an
```

### Adoption OS

```
Android :
- Android 9+ : DoT natif (Private DNS)
- Pas de DoH natif (via apps)
- ~70% des appareils supportent DoT

iOS :
- iOS 14+ : DoT et DoH via profils
- Pas d'UI utilisateur simple
- Adoption limitée (~10%)

Windows :
- Windows 11 : DoH intégré (2021)
- Configuration via Settings
- Adoption : ~20%

macOS :
- Monterey+ : DoH/DoT via profils
- Pas d'UI native
- Adoption : ~15%

Linux :
- systemd-resolved : DoT supporté
- Multiple tools (stubby, dnscrypt-proxy)
- Adoption variable (5-30% selon distro)
```

## Futur et évolutions

### DNS over QUIC (DoQ)

**RFC 9250** (2022) définit DNS over QUIC :

```
Avantages DoQ vs DoT/DoH :
- 0-RTT possible (reconnexion instantanée)
- Pas de HOL blocking (vs DoT sur TCP)
- Multiplexing natif
- Migration de connexion (changement IP)
- Plus simple que DoH (pas de HTTP)

Status en 2025 :
- Standard finalisé
- Implémentations émergentes
- Pas encore de support navigateur
- AdGuard supporte DoQ

Adoption prévue :
- 2026-2027 : support navigateurs
- Pourrait remplacer DoT (meilleure performance)
- DoH restera pour compatibilité web
```

### Oblivious DNS over HTTPS (ODoH)

**RFC 9230** (2022) - DNS anonyme :

```
Problème DoH/DoT classique :
- Resolver voit votre IP + domaines demandés
- Peut créer profil de navigation

ODoH ajoute un proxy :
Client → Proxy → Resolver

Client chiffre requête avec clé publique du Resolver
Proxy ne voit pas le contenu (chiffré)
Resolver ne voit pas l'IP client (masquée par proxy)

Séparation : qui vous êtes (IP) vs ce que vous faites (domaines)
```

**Adoption** :

```
Cloudflare + Apple collaboration (2021)
- Disponible dans iCloud Private Relay
- Pas encore généralisé

Défis :
- Latence accrue (hop additionnel)
- Besoin de confiance en proxy ET resolver
- Complexité déploiement
```

### DNS chiffré obligatoire ?

```
Tendances réglementaires :

RGPD (Europe) :
- Requêtes DNS = données personnelles
- Chiffrement recommandé mais pas obligatoire

Potentiel futur :
- Regulations pourraient exiger DNS chiffré
- Browsers pourraient l'activer par défaut pour tous
- FAI pourraient être forcés de supporter DoT/DoH

Timeline probable :
- 2025-2027 : Activation par défaut dans browsers mainstream
- 2028-2030 : Majorité du trafic DNS chiffré
- DNS classique restera pour legacy (comme HTTP côtoie HTTPS)
```

## Conclusion

DNS over HTTPS et DNS over TLS représentent une évolution majeure pour la confidentialité sur Internet. En 2025, ces protocoles sont matures, largement supportés, et en voie d'adoption généralisée.

**Pour les développeurs, les points clés** :

✅ **Privacy** : DoH/DoT protègent les requêtes DNS de la surveillance

✅ **Sécurité** : Chiffrement empêche spoofing et manipulation

✅ **Performance** : Avec optimisations (cache, pooling), overhead minimal

✅ **Adoption** : 85% des navigateurs supportent, croissance continue

✅ **Facilité** : Librairies disponibles dans tous les langages

**Actions recommandées** :

1. **Implémenter DoH dans vos apps** (surtout mobile/privacy-focused)
2. **Activer DoH/DoT sur infrastructure** (resolver compatible)
3. **Monitorer adoption** utilisateurs
4. **Prévoir fallback** DNS classique (compatibilité)
5. **Former équipe** aux implications privacy/sécurité

**Le DNS en clair sera bientôt aussi obsolète que le HTTP sans S.** Autant anticiper dès maintenant.

---

**Ressources complémentaires** :
- [RFC 7858 - DNS over TLS](https://www.rfc-editor.org/rfc/rfc7858.html)
- [RFC 8484 - DNS over HTTPS](https://www.rfc-editor.org/rfc/rfc8484.html)
- [RFC 9250 - DNS over QUIC](https://www.rfc-editor.org/rfc/rfc9250.html)
- [DNSPrivacy.org](https://dnsprivacy.org/) - Ressources et best practices
- [1.1.1.1](https://1.1.1.1/) - Service DoH/DoT gratuit Cloudflare
- [DNS Leak Test](https://dnsleaktest.com/) - Tester votre configuration

---


⏭️ [Zero Trust Networking](/10-evolutions-tendances/04-zero-trust.md)
