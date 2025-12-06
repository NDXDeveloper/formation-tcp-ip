🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.5 Filtres d'affichage et de capture

## Introduction

Imaginez une capture réseau avec 50,000 paquets. Sans filtres, trouver le paquet qui vous intéresse reviendrait à chercher une aiguille dans une botte de foin. Les filtres sont la fonctionnalité la plus puissante de Wireshark : ils permettent d'isoler exactement le trafic pertinent en quelques secondes.

Mais attention : il existe **deux types de filtres** complètement différents dans Wireshark, avec des syntaxes distinctes :

1. **Filtres de capture** (Capture Filters) : Décident **quels paquets capturer**
2. **Filtres d'affichage** (Display Filters) : Décident **quels paquets afficher**

Confondre ces deux types est l'erreur la plus courante des débutants. Cette section vous apprendra à maîtriser les deux, avec de nombreux exemples pratiques pour chaque situation.

---

## Les deux types de filtres : comprendre la différence

### Vue d'ensemble

```
CAPTURE FILTERS (BPF)                 DISPLAY FILTERS (Wireshark)
┌─────────────────────┐              ┌─────────────────────┐
│  Avant stockage     │              │  Après stockage     │
│  Syntaxe BPF        │              │  Syntaxe Wireshark  │
│  Définitif          │              │  Réversible         │
└──────────┬──────────┘              └──────────┬──────────┘
           │                                    │
           ▼                                    ▼
    ┌──────────────┐                    ┌──────────────┐
    │   Carte      │                    │   Fichier    │
    │   réseau     │──capture──>        │   .pcap      │──affichage──> [👁️ Vue]
    └──────────────┘                    └──────────────┘
```

### Filtres de capture (Capture Filters)

**Quand ?** Pendant la capture, **avant** que les paquets ne soient stockés

**Syntaxe :** BPF (Berkeley Packet Filter) - syntaxe libpcap/tcpdump

**Caractéristiques :**
- ✅ **Performance** : Réduit la charge CPU et la taille du fichier
- ✅ **Confidentialité** : Ne capture pas le trafic non désiré
- ❌ **Définitif** : Paquets non capturés = perdus à jamais
- ❌ **Syntaxe limitée** : Moins flexible que les filtres d'affichage

**Exemple :**
```
host 192.168.1.10 and port 443
```

**Usage typique :**
- Capturer uniquement HTTPS
- Capturer le trafic d'un serveur spécifique
- Exclure son propre SSH pour ne pas capturer sa session

### Filtres d'affichage (Display Filters)

**Quand ?** Après la capture, lors de l'**analyse** du fichier .pcap

**Syntaxe :** Langage propriétaire Wireshark

**Caractéristiques :**
- ✅ **Réversible** : On peut changer de filtre à volonté
- ✅ **Puissant** : Syntaxe très riche et flexible
- ✅ **Contextuel** : Peut filtrer sur n'importe quel champ décodé
- ❌ **Performance** : Tous les paquets doivent être capturés d'abord

**Exemple :**
```
ip.addr == 192.168.1.10 and tcp.port == 443
```

**Usage typique :**
- Explorer une capture déjà enregistrée
- Isoler un problème spécifique
- Analyser étape par étape

### Comparaison directe

| Aspect | Capture Filter (BPF) | Display Filter (Wireshark) |
|--------|---------------------|---------------------------|
| **Moment** | Pendant la capture | Après la capture |
| **Syntaxe** | `host 10.0.0.1` | `ip.addr == 10.0.0.1` |
| **Portée** | IP, ports, protocoles de base | Tous les champs de tous les protocoles |
| **Modification** | ❌ Impossible (définitif) | ✅ Changeable à volonté |
| **Performance** | ✅ Excellente | Dépend de la taille de la capture |
| **Complexité** | Limité | Très puissant |
| **Où ?** | Capture → Options | Barre de filtre dans l'interface |

### Quand utiliser quel type ?

**Utilisez les FILTRES DE CAPTURE quand :**
```
✅ Vous savez exactement ce que vous cherchez
✅ Vous voulez limiter la taille de la capture
✅ Vous capturez sur un serveur de production (ne pas tout enregistrer)
✅ Vous voulez protéger la confidentialité
✅ Vous capturez en continu (monitoring)
```

**Utilisez les FILTRES D'AFFICHAGE quand :**
```
✅ Vous explorez une capture existante
✅ Vous n'êtes pas sûr de ce que vous cherchez
✅ Vous voulez essayer différents filtres
✅ Vous analysez un problème complexe
✅ Vous avez besoin de critères très précis
```

**Règle d'or :**
> En cas de doute, **ne mettez PAS de filtre de capture**. Capturez tout, puis filtrez l'affichage. Mieux vaut avoir trop de données que pas assez.

---

## Filtres de capture (BPF)

### Syntaxe de base BPF

La syntaxe BPF est héritée de `tcpdump` et `libpcap`. Elle est plus limitée mais très efficace.

**Format général :**
```
[type] [direction] [protocol] [expression]
```

**Exemples simples :**
```
host 192.168.1.10                    # Tout trafic de/vers cette IP
port 80                              # Tout trafic sur le port 80
tcp                                  # Uniquement TCP
```

### Qualificateurs de type

| Qualificateur | Description | Exemple |
|---------------|-------------|---------|
| `host` | Adresse IP ou nom d'hôte | `host 192.168.1.10` |
| `net` | Réseau (subnet) | `net 192.168.1.0/24` |
| `port` | Numéro de port | `port 443` |
| `portrange` | Plage de ports | `portrange 8000-9000` |

**Exemples :**
```bash
host 10.0.5.20
# Trafic de/vers 10.0.5.20

net 192.168.0.0/16
# Tout le réseau 192.168.x.x

port 53
# DNS (port 53)

portrange 6000-7000
# Tous les ports de 6000 à 7000
```

### Qualificateurs de direction

| Qualificateur | Description | Exemple |
|---------------|-------------|---------|
| `src` | Source uniquement | `src host 10.0.0.1` |
| `dst` | Destination uniquement | `dst port 443` |
| (aucun) | Source **ou** destination | `host 10.0.0.1` |

**Exemples :**
```bash
src host 192.168.1.10
# Trafic émis PAR 192.168.1.10

dst port 80
# Trafic VERS le port 80

src net 10.0.0.0/8 and dst net 172.16.0.0/12
# De 10.x.x.x vers 172.16.x.x
```

### Qualificateurs de protocole

| Protocole | Description |
|-----------|-------------|
| `tcp` | Transmission Control Protocol |
| `udp` | User Datagram Protocol |
| `icmp` | Internet Control Message Protocol |
| `ip` | Internet Protocol (IPv4) |
| `ip6` | IPv6 |
| `arp` | Address Resolution Protocol |
| `ether` | Ethernet |

**Exemples :**
```bash
tcp
# Uniquement TCP (pas UDP, pas ICMP)

udp port 53
# DNS en UDP

icmp
# Pings et messages ICMP

ip6
# Uniquement IPv6
```

### Opérateurs logiques

| Opérateur | Alternative | Description |
|-----------|-------------|-------------|
| `and` | `&&` | ET logique |
| `or` | `||` | OU logique |
| `not` | `!` | NON logique |

**Exemples :**
```bash
# ET : Les deux conditions doivent être vraies
host 192.168.1.10 and port 443
# Trafic de/vers 192.168.1.10 ET sur le port 443

# OU : Au moins une condition doit être vraie
port 80 or port 443
# HTTP OU HTTPS

# NON : Exclure
not port 22
# Tout SAUF SSH

# Combinaisons avec parenthèses
host 10.0.0.1 and (port 80 or port 443)
# Trafic vers 10.0.0.1 sur port 80 OU 443

(src host 192.168.1.10 or src host 192.168.1.20) and dst port 3306
# De 192.168.1.10 OU .20 vers MySQL
```

### Exemples pratiques de filtres de capture

#### Exemple 1 : Capturer uniquement HTTP et HTTPS

```bash
tcp port 80 or tcp port 443
# OU plus court :
port 80 or port 443
```

**Usage :** Analyser le trafic web uniquement

#### Exemple 2 : Capturer le trafic d'un serveur spécifique

```bash
host 203.0.113.50
```

**Usage :** Débugger les communications avec un serveur précis

#### Exemple 3 : Capturer tout sauf SSH (pour ne pas capturer sa propre session)

```bash
not port 22
```

**Usage :** Capturer sur un serveur distant via SSH sans polluer la capture

#### Exemple 4 : Capturer le trafic entre deux serveurs

```bash
host 10.0.5.20 and host 10.0.8.50
```

**Usage :** Analyser la communication entre application et base de données

#### Exemple 5 : Capturer un subnet complet

```bash
net 192.168.1.0/24
```

**Usage :** Surveiller tout le trafic d'un réseau local

#### Exemple 6 : Capturer DNS et DHCP

```bash
port 53 or port 67 or port 68
```

**Usage :** Diagnostiquer les problèmes de résolution de noms et d'attribution d'adresses

#### Exemple 7 : Capturer uniquement les paquets SYN (nouvelles connexions)

```bash
tcp[tcpflags] & tcp-syn != 0
```

**Usage avancé :** Détecter les tentatives de connexion (scan de ports, DoS)

**Explication :**
- `tcp[tcpflags]` : Accède au champ flags TCP
- `& tcp-syn` : Masque pour isoler le bit SYN
- `!= 0` : Différent de zéro = SYN est activé

#### Exemple 8 : Capturer selon la taille de paquet

```bash
greater 1000
# Paquets > 1000 bytes

less 100
# Paquets < 100 bytes
```

**Usage :** Identifier les gros transferts ou les petits paquets de contrôle

#### Exemple 9 : Capturer trafic vers Internet (hors réseau local)

```bash
not net 192.168.0.0/16 and not net 10.0.0.0/8 and not net 172.16.0.0/12
```

**Usage :** Voir uniquement le trafic externe (pas le LAN)

#### Exemple 10 : Capturer broadcast et multicast

```bash
ether broadcast or ether multicast
```

**Usage :** Diagnostiquer les tempêtes broadcast, analyser mDNS, etc.

### Limitations des filtres de capture

**❌ Ne peut PAS filtrer sur :**
- Contenu applicatif (ex: URL spécifique, code HTTP)
- Flags TCP spécifiques facilement (syntaxe complexe)
- Champs de protocoles de haut niveau
- Relation entre paquets (retransmissions, etc.)

**Pour ces cas, utilisez les filtres d'affichage après capture.**

---

## Filtres d'affichage (Wireshark)

### Syntaxe de base

**Format général :**
```
protocole.champ opérateur valeur
```

**Exemples simples :**
```
ip.addr == 192.168.1.10              # Adresse IP égale à
tcp.port == 443                      # Port TCP égal à
http.request.method == "GET"         # Méthode HTTP GET
```

### Opérateurs de comparaison

| Opérateur | Alternative | Description | Exemple |
|-----------|-------------|-------------|---------|
| `==` | `eq` | Égal | `tcp.port == 80` |
| `!=` | `ne` | Différent | `ip.addr != 192.168.1.1` |
| `>` | `gt` | Plus grand que | `frame.len > 1000` |
| `<` | `lt` | Plus petit que | `tcp.window_size < 1000` |
| `>=` | `ge` | Plus grand ou égal | `ip.ttl >= 64` |
| `<=` | `le` | Plus petit ou égal | `tcp.len <= 100` |

**Exemples :**
```
tcp.len > 0
# Segments TCP avec des données (pas juste ACK)

frame.len < 64
# Paquets très petits (possibles runt frames)

ip.ttl <= 10
# TTL très bas (proche de l'expiration)
```

### Opérateurs logiques

| Opérateur | Alternative | Description |
|-----------|-------------|-------------|
| `and` | `&&` | ET logique |
| `or` | `||` | OU logique |
| `not` | `!` | NON logique |
| `xor` | `^^` | OU exclusif |

**Exemples :**
```
ip.addr == 192.168.1.10 and tcp.port == 443
# IP = 192.168.1.10 ET port = 443

http or dns
# HTTP OU DNS

not arp
# Tout sauf ARP

tcp.port == 80 xor udp.port == 80
# Port 80 en TCP ou UDP, mais pas les deux
```

### Opérateurs d'appartenance

| Opérateur | Description | Exemple |
|-----------|-------------|---------|
| `in` | Appartient à un ensemble | `tcp.port in {80 443 8080}` |
| `contains` | Contient une chaîne | `http.host contains "google"` |
| `matches` | Expression régulière | `http.request.uri matches "^/api"` |

**Exemples :**
```
tcp.port in {80 443 8080 8443}
# Port parmi cette liste (HTTP, HTTPS, proxies)

http.user_agent contains "Mozilla"
# User-Agent contient "Mozilla"

http.request.uri matches "\.php$"
# URI se termine par .php (regex)

ip.addr in {192.168.1.0/24}
# IP dans le subnet 192.168.1.0/24
```

### Filtres par protocole

#### Filtres IP

```
ip.addr == 192.168.1.10
# Trafic de/vers cette IP (source OU destination)

ip.src == 10.0.0.1
# Source uniquement

ip.dst == 203.0.113.50
# Destination uniquement

ip.addr == 192.168.1.10 and ip.addr == 93.184.216.34
# Entre ces deux IPs

ip.ttl == 1
# TTL = 1 (paquet ICMP Time Exceeded)

ip.flags.df == 1
# Don't Fragment activé

ip.fragment
# Paquets fragmentés uniquement
```

#### Filtres TCP

```
tcp.port == 80
# Port 80 (source OU destination)

tcp.srcport == 52341
# Port source

tcp.dstport == 443
# Port destination

tcp.flags.syn == 1 and tcp.flags.ack == 0
# Paquets SYN (établissement de connexion)

tcp.flags.syn == 1 and tcp.flags.ack == 1
# Paquets SYN-ACK

tcp.flags.fin == 1
# Paquets FIN (fermeture)

tcp.flags.reset == 1
# Paquets RST (reset)

tcp.stream == 12
# Toute la conversation TCP #12

tcp.len > 0
# Segments avec données (pas juste ACK vide)

tcp.analysis.retransmission
# Retransmissions TCP

tcp.analysis.duplicate_ack
# ACK dupliqués

tcp.analysis.zero_window
# Fenêtre TCP à zéro

tcp.window_size == 0
# Window size = 0

tcp.options.mss
# Paquets avec option MSS (Maximum Segment Size)
```

#### Filtres UDP

```
udp.port == 53
# DNS

udp.srcport == 68 and udp.dstport == 67
# DHCP client → serveur

udp.length > 1000
# Datagrammes UDP > 1000 bytes
```

#### Filtres HTTP

```
http
# Tout HTTP

http.request
# Requêtes HTTP uniquement

http.response
# Réponses HTTP uniquement

http.request.method == "GET"
# Requêtes GET

http.request.method == "POST"
# Requêtes POST

http.request.uri contains "/api"
# URI contient /api

http.host == "example.com"
# Header Host

http.response.code == 200
# Réponses 200 OK

http.response.code >= 400
# Erreurs 4xx et 5xx

http.response.code == 404
# Pages non trouvées

http.user_agent contains "curl"
# Requêtes faites avec curl

http.content_type contains "json"
# Réponses JSON

http.request.full_uri contains "login"
# URLs contenant "login"

http.cookie contains "session"
# Cookies de session
```

#### Filtres HTTPS/TLS

```
tls
# Tout TLS/SSL

tls.handshake.type == 1
# Client Hello

tls.handshake.type == 2
# Server Hello

tls.handshake.extensions_server_name == "example.com"
# SNI (Server Name Indication)

tls.record.content_type == 23
# Application Data (données chiffrées)

ssl.handshake.ciphersuite == 0x1301
# TLS_AES_128_GCM_SHA256
```

#### Filtres DNS

```
dns
# Tout DNS

dns.flags.response == 0
# Requêtes DNS (queries)

dns.flags.response == 1
# Réponses DNS

dns.qry.name == "example.com"
# Requête pour example.com

dns.qry.type == 1
# Type A (IPv4)

dns.qry.type == 28
# Type AAAA (IPv6)

dns.qry.type == 15
# Type MX (Mail)

dns.resp.addr == 93.184.216.34
# Réponse contenant cette IP

dns.flags.rcode != 0
# Erreurs DNS (NXDOMAIN, SERVFAIL, etc.)

dns.flags.rcode == 3
# NXDOMAIN (domaine inexistant)
```

#### Filtres ICMP

```
icmp
# Tout ICMP

icmp.type == 8
# Echo Request (ping)

icmp.type == 0
# Echo Reply (pong)

icmp.type == 3
# Destination Unreachable

icmp.type == 11
# Time Exceeded (TTL expiré)

icmp.code == 3
# Port Unreachable
```

#### Filtres ARP

```
arp
# Tout ARP

arp.opcode == 1
# ARP Request (Who has...?)

arp.opcode == 2
# ARP Reply (I am...)

arp.src.proto_ipv4 == 192.168.1.1
# Adresse IP source dans ARP

arp.dst.proto_ipv4 == 192.168.1.10
# Adresse IP destination dans ARP
```

#### Filtres DHCP

```
dhcp
# Tout DHCP

bootp.option.dhcp == 1
# DHCP Discover

bootp.option.dhcp == 2
# DHCP Offer

bootp.option.dhcp == 3
# DHCP Request

bootp.option.dhcp == 5
# DHCP ACK

dhcp.option.requested_ip_address == 192.168.1.100
# Client demande cette IP
```

### Filtres avancés

#### Filtrer par conversation

```
tcp.stream eq 5
# Tous les paquets de la conversation TCP #5

udp.stream eq 2
# Conversation UDP #2

ip.addr == 192.168.1.10 and ip.addr == 93.184.216.34 and tcp.port == 443
# Conversation HTTPS spécifique entre deux IPs
```

**Astuce :** Click droit sur un paquet → Conversation Filter → TCP
Wireshark génère automatiquement le filtre.

#### Filtrer par temps

```
frame.time >= "2025-12-07 10:00:00" and frame.time <= "2025-12-07 11:00:00"
# Entre 10h et 11h le 7 décembre 2025

frame.time_relative > 10 and frame.time_relative < 20
# Entre 10 et 20 secondes après le début de la capture

frame.time_delta > 0.1
# Delta > 100ms (délai important entre paquets)
```

#### Filtrer par taille

```
frame.len > 1400
# Paquets > 1400 bytes (proches du MTU)

frame.len < 64
# Runt frames (paquets trop petits)

tcp.len == 0
# Segments TCP sans données (ACK purs)

tcp.len > 1000
# Gros segments TCP
```

#### Filtrer par flags TCP spécifiques

```
tcp.flags == 0x02
# Uniquement SYN (pas d'autre flag)

tcp.flags == 0x12
# SYN + ACK

tcp.flags == 0x10
# ACK uniquement

tcp.flags == 0x18
# PSH + ACK

tcp.flags.push == 1
# Flag PUSH activé
```

#### Filtrer par contenu (payload)

```
tcp contains "password"
# Payload TCP contient "password" (DANGEREUX!)

http.request.uri contains "admin"
# URI contient "admin"

data contains "confidential"
# Données contiennent ce mot

frame contains 47:45:54
# Recherche hexadécimale (GET en ASCII = 0x47 0x45 0x54)
```

#### Combiner adresses et ports

```
(ip.src == 192.168.1.10 and tcp.srcport == 52341) and (ip.dst == 93.184.216.34 and tcp.dstport == 443)
# Socket source et destination exactes

ip.addr == 192.168.1.10 and tcp.port in {80 443 8080}
# IP spécifique sur plusieurs ports web
```

### Exemples de cas d'usage réels

#### Cas 1 : Analyser une API REST lente

```
# 1. Voir toutes les requêtes vers l'API
http.host == "api.example.com"

# 2. Filtrer uniquement les POST (création/modification)
http.host == "api.example.com" and http.request.method == "POST"

# 3. Voir les réponses lentes (hypothèse: > 1 seconde entre requête et réponse)
http.time > 1

# 4. Voir les erreurs serveur
http.host == "api.example.com" and http.response.code >= 500

# 5. Voir une conversation API complète (après avoir identifié le stream)
tcp.stream eq 42 and http
```

#### Cas 2 : Diagnostiquer un problème DNS

```
# 1. Toutes les requêtes DNS
dns.flags.response == 0

# 2. Requêtes pour un domaine spécifique
dns.qry.name contains "example.com"

# 3. Réponses DNS avec erreur
dns.flags.rcode != 0

# 4. Temps de réponse DNS > 100ms
dns.time > 0.1

# 5. Requêtes DNS sans réponse
dns.flags.response == 0 and not dns.response_in
```

#### Cas 3 : Identifier des retransmissions TCP

```
# 1. Toutes les retransmissions
tcp.analysis.retransmission

# 2. Retransmissions vers un serveur spécifique
tcp.analysis.retransmission and ip.dst == 203.0.113.50

# 3. Retransmissions + ACK dupliqués (problème sérieux)
tcp.analysis.retransmission or tcp.analysis.duplicate_ack

# 4. Compter les retransmissions par stream
tcp.analysis.retransmission and tcp.stream == 5
```

#### Cas 4 : Analyser une connexion qui échoue

```
# 1. Voir les SYN sans réponse (connexion refusée/timeout)
tcp.flags.syn == 1 and tcp.flags.ack == 0 and not tcp.analysis.retransmission

# 2. Voir les RST (connexion fermée brutalement)
tcp.flags.reset == 1

# 3. Voir les connexions établies puis immédiatement fermées
tcp.flags.syn == 1 or tcp.flags.fin == 1 or tcp.flags.reset == 1

# 4. Identifier les timeouts (peut nécessiter d'observer le delta de temps)
tcp.analysis.retransmission and frame.time_delta > 1
```

#### Cas 5 : Détecter un scan de ports

```
# 1. Beaucoup de SYN vers différents ports (scan)
tcp.flags.syn == 1 and tcp.flags.ack == 0

# 2. SYN suivis de RST (ports fermés)
tcp.flags.reset == 1

# 3. Afficher par IP source pour identifier le scanner
# (puis Statistics → Conversations → TCP)
```

#### Cas 6 : Analyser du trafic HTTPS

```
# 1. Voir les handshakes TLS
tls.handshake

# 2. Client Hello uniquement
tls.handshake.type == 1

# 3. Identifier le site visité (SNI)
tls.handshake.extensions_server_name == "example.com"

# 4. Voir les données chiffrées
tls.record.content_type == 23

# 5. Identifier les erreurs TLS
tls.alert_message
```

#### Cas 7 : Problèmes de performances réseau

```
# 1. Window Size = 0 (récepteur saturé)
tcp.window_size == 0

# 2. Segments avec peu de données (inefficace)
tcp.len > 0 and tcp.len < 100

# 3. Délais importants entre paquets
frame.time_delta > 0.5

# 4. Paquets fragmentés (possible problème de MTU)
ip.flags.mf == 1 or ip.frag_offset > 0
```

---

## Macros et fonctions avancées

### Opérateurs d'ensemble

```
tcp.port in {80 443 8080 8443}
# Port dans cet ensemble

!(tcp.port in {80 443})
# Port PAS dans cet ensemble

ip.addr in {192.168.1.0/24 10.0.0.0/8}
# IP dans ces subnets
```

### Recherche avec regex (matches)

```
http.request.uri matches "\\.(jpg|png|gif)$"
# URI se termine par .jpg, .png ou .gif

http.user_agent matches "(?i)bot|crawler|spider"
# User-Agent contient bot/crawler/spider (insensible à la casse)

dns.qry.name matches "^www\\."
# Domaine commence par www.

http.host matches "^(api|cdn)\\."
# Host commence par api. ou cdn.
```

### Slices (découper les champs)

```
eth.src[0:3] == 00:1a:2b
# Les 3 premiers octets de l'adresse MAC source

ip.addr[0] == 192
# Premier octet de l'IP = 192 (192.x.x.x)

tcp[13] == 0x02
# Octet 13 du segment TCP (flags) = 0x02 (SYN)

tcp[13] & 0x02
# Bit SYN activé (équivalent à tcp.flags.syn == 1)
```

### Vérifier l'existence d'un champ

```
http.cookie
# Tous les paquets contenant un cookie HTTP

tcp.options.mss
# Paquets TCP avec option MSS

tls.handshake.extensions_server_name
# Handshakes TLS avec SNI
```

---

## Astuces et bonnes pratiques

### 1. Autocomplétion

Dans la barre de filtres, Wireshark propose l'autocomplétion :

```
Tapez : ip.
→ Wireshark affiche : ip.addr, ip.src, ip.dst, ip.ttl, ip.len, etc.

Tapez : http.re
→ Wireshark affiche : http.request, http.response, etc.
```

**Couleurs de la barre de filtre :**
- **Vert** : Syntaxe valide
- **Rouge** : Erreur de syntaxe
- **Jaune** : Syntaxe valide mais ambiguë ou potentiellement inefficace

### 2. Sauvegarder des filtres fréquents

**Bookmarks de filtres :**
```
Barre de filtres → [+] Ajouter comme favori
Nom : "Trafic API production"
Filtre : http.host == "api.example.com" and http.response.code >= 400
```

**Importer/exporter des filtres :**
```
Analyze → Display Filters → [Gérer les filtres]
→ Exporter vers un fichier
→ Importer depuis un fichier
→ Partager avec l'équipe
```

### 3. Combiner filtres de capture et d'affichage

**Scénario :** Analyser les erreurs HTTP d'un serveur spécifique

**Filtre de capture (pour limiter la taille) :**
```
host 203.0.113.50 and port 80
```

**Filtre d'affichage (pour isoler les erreurs) :**
```
http.response.code >= 400
```

### 4. Filtres par exclusion progressive

**Technique pour explorer une capture :**
```
1. Commencer large : http
2. Exclure ce qui est normal : http and http.response.code != 200
3. Affiner : http and http.response.code != 200 and http.response.code != 304
4. Isoler : http.response.code == 500
```

### 5. Utiliser les filtres préparés

**Click droit sur un paquet → Apply as Filter →**
- `Selected` : Filtre sur ce champ exact
- `Not Selected` : Exclure ce champ
- `... and Selected` : Ajouter en AND
- `... or Selected` : Ajouter en OR

**Exemple :**
```
1. Click droit sur "Source: 192.168.1.10" → Apply as Filter → Selected
   → Filtre appliqué : ip.src == 192.168.1.10

2. Click droit sur "Destination Port: 443" → Apply as Filter → ... and Selected
   → Filtre devient : ip.src == 192.168.1.10 and tcp.dstport == 443
```

### 6. Filtrer les paquets marqués

```
frame.marked == 1
# Afficher uniquement les paquets que vous avez marqués (Ctrl+M)
```

### 7. Filtres de débogage protocole

```
# Voir les checksums invalides
ip.checksum_bad or tcp.checksum_bad or udp.checksum_bad

# Voir les paquets avec erreurs
frame.protocols contains "malformed"

# Expert Info avec avertissements
_ws.expert.severity == "warning"

# Expert Info avec erreurs
_ws.expert.severity == "error"
```

---

## Filtres complexes : exemples avancés

### Exemple 1 : Identifier une exfiltration de données

```
# Gros volumes de données sortantes vers l'extérieur
(ip.src == 192.168.1.0/24) and not (ip.dst == 192.168.1.0/24 or ip.dst == 10.0.0.0/8) and frame.len > 1400

# Connexions HTTPS vers des IPs (pas de domaine = suspect)
tls and not dns.qry.name

# Uploads HTTP volumineux
http.request.method == "POST" and http.content_length > 100000
```

### Exemple 2 : Détecter du trafic malveillant

```
# Scans de ports (nombreux SYN vers ports différents)
tcp.flags.syn == 1 and tcp.flags.ack == 0

# Beaucoup de RST (ports fermés = scan)
tcp.flags.reset == 1

# Requêtes DNS vers des C&C suspects (domaines générés)
dns.qry.name matches "^[a-z]{20,}\\."
# Domaines très longs, aléatoires

# Trafic vers des ports inhabituels
tcp.port > 10000 and tcp.port < 65535 and not (tcp.port == 443 or tcp.port == 8080)
```

### Exemple 3 : Analyser les performances d'une base de données

```
# Requêtes MySQL
mysql.query

# Requêtes lentes (hypothèse : > 100ms entre query et response)
mysql and frame.time_delta > 0.1

# Connexions PostgreSQL
pgsql

# Erreurs PostgreSQL
pgsql.type == "E"
```

### Exemple 4 : Filtrer le trafic VoIP (SIP/RTP)

```
# Signalisation SIP
sip

# Audio RTP
rtp

# Codec spécifique
rtp.payload_type == 0
# PCMU (G.711 μ-law)

# Problèmes de qualité (jitter, paquets perdus)
rtp.ext.jitter > 30
```

### Exemple 5 : Analyser le trafic d'un container Docker

```
# Trafic du réseau Docker
ip.addr == 172.17.0.0/16

# Entre containers spécifiques
ip.src == 172.17.0.2 and ip.dst == 172.17.0.3

# Trafic sortant de Docker vers Internet
ip.src == 172.17.0.0/16 and not ip.dst == 172.17.0.0/16
```

---

## Erreurs courantes et comment les éviter

### Erreur 1 : Confondre `==` et `contains`

```
❌ FAUX :
http.host == "example"
# Ne trouve rien (doit être exact : "example.com")

✅ CORRECT :
http.host contains "example"
# Trouve "example.com", "api.example.com", etc.
```

### Erreur 2 : Oublier les guillemets

```
❌ FAUX :
http.request.method == GET
# Erreur : GET non reconnu

✅ CORRECT :
http.request.method == "GET"
# Les valeurs textuelles doivent être entre guillemets
```

### Erreur 3 : Parenthèses manquantes

```
❌ FAUX :
ip.addr == 192.168.1.10 and tcp.port == 80 or tcp.port == 443
# Ambigu : (ip and tcp.80) or tcp.443 ?

✅ CORRECT :
ip.addr == 192.168.1.10 and (tcp.port == 80 or tcp.port == 443)
# Clair : ip ET (port 80 OU 443)
```

### Erreur 4 : Utiliser `ip.addr` au lieu de `ip.src` / `ip.dst`

```
❌ PEUT ÊTRE FAUX :
ip.addr == 192.168.1.10 and ip.addr == 93.184.216.34
# Trouve les paquets entre ces IPs
# MAIS AUSSI les paquets de/vers les deux IPs séparément

✅ CORRECT (si on veut une direction) :
ip.src == 192.168.1.10 and ip.dst == 93.184.216.34
# Uniquement 192.168.1.10 → 93.184.216.34
```

### Erreur 5 : Filtres trop restrictifs qui ne trouvent rien

```
❌ TROP RESTRICTIF :
tcp.flags == 0x018
# Trouve uniquement PSH+ACK sans AUCUN autre flag

✅ MIEUX :
tcp.flags.push == 1 and tcp.flags.ack == 1
# Trouve PSH+ACK (même si d'autres flags sont présents)
```

---

## Référence rapide : Filtres les plus utiles

### Top 20 des filtres à connaître

```
1.  ip.addr == X.X.X.X                      # Trafic d'une IP
2.  tcp.port == N                           # Trafic sur un port
3.  http                                    # Tout HTTP
4.  dns                                     # Tout DNS
5.  tcp.stream eq N                         # Conversation TCP spécifique
6.  tcp.analysis.retransmission             # Retransmissions
7.  http.response.code == 404               # Pages non trouvées
8.  tcp.flags.syn == 1 and tcp.flags.ack == 0  # SYN (nouvelles connexions)
9.  tcp.flags.reset == 1                    # Connexions fermées brutalement
10. frame.len > 1400                        # Gros paquets
11. tcp.len > 0                             # Segments TCP avec données
12. http.request.method == "POST"           # Requêtes POST
13. dns.qry.name contains "example"         # Requêtes DNS pour un domaine
14. tls.handshake.type == 1                 # Client Hello TLS
15. tcp.window_size == 0                    # Fenêtre TCP à zéro
16. icmp.type == 8                          # Ping requests
17. arp                                     # Trafic ARP
18. not tcp and not udp                     # Tout sauf TCP/UDP
19. http.time > 1                           # Requêtes HTTP > 1 seconde
20. ip.addr == A and ip.addr == B           # Entre deux IPs
```

### Filtres par cas d'usage

**Diagnostic web :**
```
http and http.response.code >= 400
http.host == "example.com"
http.request.method == "POST" and http.content_type contains "json"
```

**Diagnostic DNS :**
```
dns.flags.rcode != 0
dns.qry.name == "example.com"
dns.time > 0.1
```

**Performance réseau :**
```
tcp.analysis.retransmission
tcp.window_size < 10000
frame.time_delta > 0.5
```

**Sécurité :**
```
tcp.flags.syn == 1 and tcp.flags.ack == 0
http.request.uri contains "../"
dns.qry.name matches "^[a-z]{30,}\\."
```

---

## Conclusion

Les filtres sont l'outil le plus puissant de Wireshark. Leur maîtrise transforme une capture brute de 50,000 paquets en une vue ciblée de quelques dizaines de paquets pertinents.

### Points clés à retenir

**🎯 Les deux types de filtres**
- **Capture (BPF)** : Avant stockage, définitif, syntaxe tcpdump
- **Affichage (Wireshark)** : Après stockage, réversible, syntaxe riche

**🎯 Quand utiliser quoi**
- Filtre de capture : Vous savez ce que vous cherchez, économiser ressources
- Filtre d'affichage : Explorer, analyser, tester différentes hypothèses

**🎯 Syntaxe de base**
- BPF : `host 10.0.0.1 and port 443`
- Wireshark : `ip.addr == 10.0.0.1 and tcp.port == 443`

**🎯 Opérateurs essentiels**
- Comparaison : `==`, `!=`, `>`, `<`
- Logique : `and`, `or`, `not`
- Appartenance : `in`, `contains`, `matches`

**🎯 Astuces pratiques**
- Autocomplétion (taper puis Ctrl+Space)
- Click droit → Apply as Filter
- Sauvegarder les filtres fréquents
- Commencer large puis affiner

**🎯 Filtres les plus utiles**
```
ip.addr == X                    # IP spécifique
tcp.port == N                   # Port spécifique
tcp.stream eq N                 # Conversation complète
tcp.analysis.retransmission     # Problèmes réseau
http.response.code >= 400       # Erreurs HTTP
```

### Progression recommandée

**Débutant :**
- Maîtriser `ip.addr`, `tcp.port`, `http`, `dns`
- Utiliser Click droit → Apply as Filter
- Combiner avec `and` / `or`

**Intermédiaire :**
- Filtres par stream (`tcp.stream`)
- Filtres d'analyse (`tcp.analysis.*`)
- Expressions régulières (`matches`)
- Filtres temporels (`frame.time*`)

**Avancé :**
- Slices (`tcp[13]`)
- Filtres complexes multi-critères
- Macros personnalisées
- Automatisation avec tshark

Dans la prochaine section, nous appliquerons ces filtres pour identifier et diagnostiquer les problèmes réseau courants.

---

**Prochaine section :** 7.6 Identification des problèmes courants

Nous utiliserons les filtres que nous venons d'apprendre pour détecter et diagnostiquer les problèmes typiques : retransmissions, timeouts, MTU, fenêtre TCP, et bien d'autres.

⏭️ [Identification des problèmes courants](/07-analyse-depannage/06-problemes-courants.md)
