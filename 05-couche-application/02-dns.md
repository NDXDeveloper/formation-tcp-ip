🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.2 DNS (Domain Name System)

## Introduction

Imaginez devoir mémoriser `142.250.185.206` pour accéder à Google, `157.240.241.35` pour Facebook, ou `104.244.42.1` pour Twitter. Ce serait non seulement fastidieux, mais aussi impossible à l'échelle d'Internet. C'est exactement le problème que résout le **Domain Name System (DNS)** : il traduit les noms de domaine lisibles par l'homme en adresses IP compréhensibles par les machines.

DNS est souvent qualifié de **"l'annuaire d'Internet"** ou de **"système téléphonique d'Internet"**. C'est l'un des protocoles les plus critiques d'Internet : si DNS ne fonctionne pas, la plupart des services Internet deviennent inaccessibles, même si la connectivité réseau est parfaite.

## Le problème résolu par DNS

### Avant DNS : Le fichier HOSTS

Dans les premiers jours d'ARPANET (l'ancêtre d'Internet), le nombre d'ordinateurs connectés était très limité. La solution utilisée était simple : un **fichier texte** appelé `HOSTS.TXT` qui associait des noms à des adresses IP.

**Exemple de fichier HOSTS.TXT (années 1970) :**
```
# Fichier HOSTS.TXT
10.0.0.1    mit-multics
10.0.0.2    stanford-ai
10.0.0.3    ucla-ccn
10.0.0.4    sri-arc
```

Ce fichier était maintenu par le Stanford Research Institute (SRI) et **distribué manuellement** à tous les ordinateurs du réseau.

### Les limitations du système HOSTS

Au fur et à mesure de la croissance du réseau, ce système a montré ses limites :

**Problème 1 : Scalabilité**
```
1980 : ~100 hôtes → Fichier de quelques Ko, gérable
1985 : ~1 000 hôtes → Fichier de plusieurs centaines de Ko
1990 : ~100 000 hôtes → Impossible à maintenir manuellement !
Aujourd'hui : Milliards de noms → Complètement impossible
```

**Problème 2 : Mise à jour**
- Chaque modification nécessitait de redistribuer le fichier à tous
- Délai de propagation important
- Incohérences fréquentes

**Problème 3 : Collisions de noms**
- Pas d'autorité centralisée pour l'attribution des noms
- Conflits possibles entre organisations

**Problème 4 : Performance**
- Recherche linéaire dans un fichier texte
- Pas de cache efficace

### La solution : DNS

DNS a été conçu en 1983 (RFC 882 et 883) et révisé en 1987 (RFC 1034 et 1035) pour résoudre ces problèmes par une architecture **distribuée et hiérarchique**.

## Concepts fondamentaux

### Qu'est-ce qu'un nom de domaine ?

Un nom de domaine est une chaîne de caractères qui identifie un domaine dans l'espace de noms DNS.

**Anatomie d'un nom de domaine complet (FQDN - Fully Qualified Domain Name) :**

```
www.example.com.
│   │       │   │
│   │       │   └─ Point final (root), souvent omis
│   │       └───── Domaine de deuxième niveau (SLD)
│   └───────────── Domaine de troisième niveau (sous-domaine)
└───────────────── Hôte ou service

Lecture de droite à gauche (du plus général au plus spécifique)
```

**Exemples concrets :**
```
mail.google.com
│    │      │
│    │      └─ Domaine de premier niveau (.com)
│    └──────── Domaine de deuxième niveau (google)
└───────────── Sous-domaine / service (mail)

api.github.com
│   │      │
│   │      └─ TLD : .com
│   └──────── SLD : github
└───────────── Sous-domaine : api

blog.entreprise.fr
│    │          │
│    │          └─ TLD : .fr (code pays)
│    └──────────── SLD : entreprise
└────────────────── Sous-domaine : blog
```

### Hiérarchie et délégation

DNS utilise une structure hiérarchique en arbre inversé :

```
                    . (root)
                    │
        ┌───────────┼───────────┬─────────┐
        │           │           │         │
       .com        .org        .fr       .net
        │           │           │         │
    ┌───┴───┐   ┌───┴───┐    ┌──┴──┐      │
    │       │   │       │    │     │      │
 google  amazon │    mozilla │    gouv     │
    │       │   wikipedia │   │     │   cloudflare
    │       │       │     │   │     │
  ┌─┴─┐     │       │     │   │     │
  │   │     │       │     │   │     │
 www mail   │       │     │   │     │
          aws      fr    en  www    │
```

**Principe de délégation :**
- La racine (.) délègue la gestion des TLD (.com, .org, .fr...)
- Chaque TLD délègue la gestion des domaines de deuxième niveau
- Chaque organisation gère ses propres sous-domaines

### Types d'enregistrements DNS

DNS ne fait pas que traduire des noms en adresses IP. Il stocke différents types d'informations via des **enregistrements** (records).

**Principaux types d'enregistrements :**

| Type | Nom complet | Usage | Exemple |
|------|-------------|-------|---------|
| **A** | Address | IPv4 d'un hôte | `example.com` → `93.184.216.34` |
| **AAAA** | IPv6 Address | IPv6 d'un hôte | `example.com` → `2606:2800:220:1:...` |
| **CNAME** | Canonical Name | Alias vers un autre nom | `www.example.com` → `example.com` |
| **MX** | Mail Exchange | Serveur de messagerie | `example.com` → `mail.example.com` |
| **NS** | Name Server | Serveur DNS autoritaire | `example.com` → `ns1.example.com` |
| **TXT** | Text | Informations textuelles | Vérification SPF, DKIM |
| **PTR** | Pointer | Résolution inverse (IP→nom) | `34.216.184.93.in-addr.arpa` → `example.com` |
| **SOA** | Start of Authority | Informations de zone | Serveur primaire, email admin |

**Exemple concret de zone DNS :**
```
; Zone DNS pour example.com
example.com.        IN  SOA   ns1.example.com. admin.example.com. (
                              2024120601 ; Serial
                              3600       ; Refresh
                              1800       ; Retry
                              604800     ; Expire
                              86400 )    ; Minimum TTL

example.com.        IN  NS    ns1.example.com.
example.com.        IN  NS    ns2.example.com.
example.com.        IN  A     93.184.216.34
example.com.        IN  AAAA  2606:2800:220:1:248:1893:25c8:1946
example.com.        IN  MX    10 mail.example.com.

www.example.com.    IN  CNAME example.com.
mail.example.com.   IN  A     93.184.216.50
ftp.example.com.    IN  A     93.184.216.60

; Enregistrement TXT pour vérification
example.com.        IN  TXT   "v=spf1 mx ~all"
```

## Fonctionnement de base

### Le processus de résolution DNS

Quand vous tapez `www.example.com` dans votre navigateur, voici ce qui se passe :

**Vue simplifiée :**
```
┌──────────────┐
│  Navigateur  │  1. Quelle est l'IP de www.example.com ?
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ Résolveur DNS│  2. Je vais chercher...
│   (local)    │
└──────┬───────┘
       │
       ↓
   (processus de
    résolution)
       │
       ↓
┌──────────────┐
│ Résolveur DNS│  3. C'est 93.184.216.34
└──────┬───────┘
       │
       ↓
┌──────────────┐
│  Navigateur  │  4. Merci ! Je me connecte à cette IP
└──────────────┘
```

**Vue détaillée (résolution complète) :**

```
┌─────────────────────────────────────────────────────────────────┐
│ ÉTAPE 1 : Vérification du cache local                           │
└─────────────────────────────────────────────────────────────────┘

Ordinateur vérifie :
1. Cache du navigateur (Firefox, Chrome...)
2. Cache du système d'exploitation
3. Fichier /etc/hosts (Linux/Mac) ou C:\Windows\System32\drivers\etc\hosts

Si trouvé → Utilise directement l'IP
Si non trouvé → Passe à l'étape 2

┌─────────────────────────────────────────────────────────────────┐
│ ÉTAPE 2 : Requête au résolveur récursif                         │
└─────────────────────────────────────────────────────────────────┘

Ordinateur → Résolveur DNS (ex: 8.8.8.8 - Google Public DNS)
"Quelle est l'IP de www.example.com ?"

Le résolveur vérifie son propre cache
Si trouvé → Répond directement
Si non trouvé → Lance une résolution complète (étapes 3-6)

┌─────────────────────────────────────────────────────────────────┐
│ ÉTAPE 3 : Interrogation du serveur racine                       │
└─────────────────────────────────────────────────────────────────┘

Résolveur → Serveur racine (.)
"Où trouver .com ?"

Serveur racine → Résolveur
"Contacte les serveurs NS de .com : a.gtld-servers.net (IP)"

┌─────────────────────────────────────────────────────────────────┐
│ ÉTAPE 4 : Interrogation du serveur TLD                          │
└─────────────────────────────────────────────────────────────────┘

Résolveur → Serveur TLD .com
"Où trouver example.com ?"

Serveur .com → Résolveur
"Contacte ns1.example.com (IP) et ns2.example.com (IP)"

┌─────────────────────────────────────────────────────────────────┐
│ ÉTAPE 5 : Interrogation du serveur autoritaire                  │
└─────────────────────────────────────────────────────────────────┘

Résolveur → ns1.example.com (serveur autoritaire)
"Quelle est l'IP de www.example.com ?"

ns1.example.com → Résolveur
"www.example.com a l'adresse 93.184.216.34"

┌─────────────────────────────────────────────────────────────────┐
│ ÉTAPE 6 : Réponse et mise en cache                              │
└─────────────────────────────────────────────────────────────────┘

Résolveur → Ordinateur
"93.184.216.34"

Le résolveur met en cache cette réponse pour les requêtes futures
L'ordinateur met aussi en cache
```

**Chronométrage typique :**
```
Cache local       : 0-1 ms
Cache résolveur   : 10-50 ms
Résolution complète : 50-200 ms
```

### Exemple de requête DNS capturée

Voici à quoi ressemble une vraie requête DNS (format simplifié) :

**Requête (Query) :**
```
DNS Query
├─ Transaction ID: 0x1234
├─ Flags: Standard query (0x0100)
│   ├─ Query/Response: Query
│   ├─ Opcode: Standard query
│   ├─ Recursion desired: Yes
│   └─ ...
├─ Questions: 1
│   └─ www.example.com: type A, class IN
├─ Answer RRs: 0
├─ Authority RRs: 0
└─ Additional RRs: 0
```

**Réponse (Response) :**
```
DNS Response
├─ Transaction ID: 0x1234 (même que la requête)
├─ Flags: Standard query response (0x8180)
│   ├─ Query/Response: Response
│   ├─ Authoritative: Yes
│   ├─ Recursion available: Yes
│   └─ ...
├─ Questions: 1
│   └─ www.example.com: type A, class IN
├─ Answer RRs: 1
│   └─ www.example.com: type A, addr 93.184.216.34, TTL 86400
├─ Authority RRs: 0
└─ Additional RRs: 0
```

### Format du paquet DNS

Un paquet DNS est structuré ainsi :

```
 0                   16                  31
┌────────────────────┬────────────────────┐
│   Transaction ID   │                    │
├────────────────────┼────────────────────┤
│       Flags        │                    │
├────────────────────┼────────────────────┤
│   # Questions      │   # Answers        │
├────────────────────┼────────────────────┤
│   # Authority      │   # Additional     │
├────────────────────┴────────────────────┤
│                                         │
│           Questions                     │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│           Answers                       │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│           Authority                     │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│           Additional                    │
│                                         │
└─────────────────────────────────────────┘
```

**Champs importants :**
- **Transaction ID** : Identifiant unique pour associer requête et réponse
- **Flags** : Type de message, récursion, autorité, etc.
- **Questions** : Ce que l'on cherche
- **Answers** : Les réponses fournies
- **Authority** : Serveurs NS autoritaires
- **Additional** : Informations complémentaires (adresses IP des NS, par exemple)

## Types de requêtes et résolutions

### Requête récursive vs itérative

**Requête récursive :**
Le client demande au serveur DNS de faire tout le travail.

```
Client → Résolveur récursif : "Trouve-moi www.example.com"
Résolveur fait toutes les requêtes nécessaires :
  → Serveur racine
  → Serveur TLD
  → Serveur autoritaire
Résolveur → Client : "Voici : 93.184.216.34"

Avantage : Simple pour le client
Inconvénient : Charge sur le résolveur
```

**Requête itérative :**
Le serveur DNS renvoie la meilleure réponse qu'il a, mais ne fait pas de requêtes supplémentaires.

```
Résolveur → Serveur racine : "Où est www.example.com ?"
Serveur racine → Résolveur : "Je ne sais pas, mais essaie les serveurs .com"

Résolveur → Serveur .com : "Où est www.example.com ?"
Serveur .com → Résolveur : "Je ne sais pas, mais essaie ns1.example.com"

Résolveur → ns1.example.com : "Où est www.example.com ?"
ns1.example.com → Résolveur : "C'est 93.184.216.34"

Avantage : Distribue la charge
Inconvénient : Plus de requêtes réseau
```

**En pratique :**
- Votre ordinateur → Résolveur : **Récursive**
- Résolveur → Autres serveurs DNS : **Itérative**

### Résolution directe vs inverse

**Résolution directe (Forward DNS) :**
Nom → Adresse IP

```
www.google.com → 142.250.185.206
```

**Résolution inverse (Reverse DNS) :**
Adresse IP → Nom

```
142.250.185.206 → par-10-162-250-142.cloudfront.net
```

**Format spécial pour le reverse DNS :**
L'adresse IP est inversée et `.in-addr.arpa` est ajouté :

```
Adresse IP : 192.0.2.1

Requête reverse DNS : 1.2.0.192.in-addr.arpa

Exemple complet :
dig -x 8.8.8.8
→ 8.8.8.8.in-addr.arpa → dns.google
```

**Utilité du reverse DNS :**
- Vérification des serveurs de messagerie (anti-spam)
- Logs plus lisibles
- Diagnostics réseau

## Le rôle du cache DNS

Le cache est **crucial** pour les performances DNS.

### Niveaux de cache

```
┌────────────────────────────────────────────┐
│  1. Cache navigateur                       │  Durée : Minutes
│     (Firefox, Chrome)                      │  Portée : Application
├────────────────────────────────────────────┤
│  2. Cache système d'exploitation           │  Durée : Heures
│     (systemd-resolved, DNS Client)         │  Portée : Machine
├────────────────────────────────────────────┤
│  3. Cache résolveur récursif               │  Durée : Selon TTL
│     (8.8.8.8, 1.1.1.1)                     │  Portée : Tous les clients
├────────────────────────────────────────────┤
│  4. Cache CDN/ISP                          │  Durée : Selon TTL
│     (niveau opérateur)                     │  Portée : Région/Pays
└────────────────────────────────────────────┘
```

### TTL (Time To Live)

Chaque enregistrement DNS a un **TTL** qui indique combien de temps il peut être mis en cache.

**Exemple :**
```
www.example.com.  3600  IN  A  93.184.216.34
                  ↑
                  TTL en secondes (1 heure)
```

**Impact du TTL :**

```
TTL court (ex: 60 secondes) :
✅ Changements propagés rapidement
✅ Bon pour tests ou migrations
❌ Plus de charge sur les serveurs DNS
❌ Légère dégradation des performances

TTL long (ex: 86400 = 24 heures) :
✅ Moins de requêtes DNS
✅ Meilleures performances
✅ Moins de charge serveur
❌ Changements lents à se propager
```

**Stratégie courante pour une migration :**
```
J-7  : Réduire le TTL à 300 (5 minutes)
J-0  : Effectuer le changement d'IP
J+1  : Une fois stabilisé, remonter le TTL à 3600 ou plus
```

### Impact du cache sur les performances

**Sans cache :**
```
Requête 1 : www.example.com → Résolution complète (150 ms)
Requête 2 : www.example.com → Résolution complète (150 ms)
Requête 3 : www.example.com → Résolution complète (150 ms)
Total pour 3 requêtes : 450 ms
```

**Avec cache :**
```
Requête 1 : www.example.com → Résolution complète (150 ms) → Mise en cache
Requête 2 : www.example.com → Cache local (1 ms)
Requête 3 : www.example.com → Cache local (1 ms)
Total pour 3 requêtes : 152 ms (66% plus rapide !)
```

## Protocoles de transport DNS

### UDP : Le protocole par défaut

DNS utilise principalement **UDP sur le port 53**.

**Pourquoi UDP ?**
- Requêtes courtes (généralement < 512 octets)
- Pas besoin de connexion établie
- Faible overhead
- Rapidité

**Format d'une requête DNS sur UDP :**
```
Ordinateur                    Serveur DNS
    │                              │
    │─── Requête UDP (port 53) ───>│
    │    "www.example.com ?"       │
    │                              │
    │<─── Réponse UDP ─────────────│
    │    "93.184.216.34"           │
    │                              │

Temps total : ~20-50 ms
```

### TCP : Pour les cas spéciaux

DNS utilise **TCP sur le port 53** dans certains cas :

**1. Réponses trop grandes (> 512 octets)**
```
Requête UDP → Réponse tronquée (flag TC=1)
Client comprend : "Réponse trop grande, je dois utiliser TCP"
Client refait la requête en TCP
```

**2. Transferts de zone (AXFR)**
Synchronisation complète entre serveurs DNS :
```
Serveur secondaire → Serveur primaire (TCP)
"Envoie-moi toute la zone example.com"
Serveur primaire → Serveur secondaire (TCP)
[Tous les enregistrements DNS de la zone...]
```

**3. DNS over TLS (DoT) et DNS over HTTPS (DoH)**
Versions sécurisées de DNS (voir section évolutions et tendances).

## DNS et sécurité

### Vulnérabilités du DNS classique

**1. Pas de chiffrement**
```
Client → "www.banque.com ?" → Résolveur

Attaquant peut voir :
• Quels sites vous visitez
• Vos habitudes de navigation
• Potentiellement : intentions, localisation
```

**2. Pas d'authentification**
```
Attaquant peut :
• Usurper des réponses DNS (DNS spoofing)
• Rediriger vers de faux sites (pharming)
• Empoisonner les caches (cache poisoning)
```

**3. Amplification DDoS**
```
Attaquant envoie requête DNS avec IP source falsifiée
→ Serveur DNS envoie grosse réponse à la victime
→ Amplification : petite requête → grande réponse
```

### Mécanismes de protection

**DNSSEC (DNS Security Extensions)**
- Signature cryptographique des enregistrements
- Vérification de l'authenticité
- Protection contre la falsification

**DoT (DNS over TLS) et DoH (DNS over HTTPS)**
- Chiffrement des requêtes DNS
- Protection de la vie privée
- Empêche l'espionnage

Nous verrons ces technologies en détail dans les sections ultérieures.

## DNS dans la pratique

### Configuration DNS typique

**Sur un ordinateur Linux (/etc/resolv.conf) :**
```bash
# Serveurs DNS utilisés
nameserver 8.8.8.8        # Google Public DNS
nameserver 8.8.4.4        # Google Public DNS secondaire
nameserver 1.1.1.1        # Cloudflare DNS

# Domaine de recherche
search example.com

# Options
options timeout:2 attempts:3
```

**Sur Windows (via GUI ou PowerShell) :**
```powershell
# Voir la configuration DNS
Get-DnsClientServerAddress

# Définir un serveur DNS
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
    -ServerAddresses ("8.8.8.8","8.8.4.4")
```

### Outils de diagnostic DNS

**dig (Linux/Mac) :**
```bash
# Requête simple
dig www.example.com

# Requête avec serveur spécifique
dig @8.8.8.8 www.example.com

# Trace complète de la résolution
dig +trace www.example.com

# Requête d'un type spécifique
dig example.com MX
dig example.com NS
dig example.com TXT

# Résolution inverse
dig -x 93.184.216.34
```

**nslookup (Windows/Linux/Mac) :**
```bash
# Mode interactif
nslookup
> www.example.com
> exit

# Mode direct
nslookup www.example.com

# Avec serveur spécifique
nslookup www.example.com 8.8.8.8

# Type d'enregistrement
nslookup -type=MX example.com
```

**host (Linux/Mac) :**
```bash
# Simple et rapide
host www.example.com

# Avec détails
host -v www.example.com

# Type spécifique
host -t MX example.com
```

### Exemple concret : Résolution de www.github.com

```bash
$ dig www.github.com

; <<>> DiG 9.18.24 <<>> www.github.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;www.github.com.                IN      A

;; ANSWER SECTION:
www.github.com.         60      IN      CNAME   github.com.
github.com.             60      IN      A       140.82.121.3

;; Query time: 15 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Fri Dec 06 14:30:00 CET 2024
;; MSG SIZE  rcvd: 73
```

**Interprétation :**
1. `www.github.com` est un **CNAME** (alias) qui pointe vers `github.com`
2. `github.com` a l'adresse IP **140.82.121.3**
3. TTL de **60 secondes** (cache court pour flexibilité)
4. Résolution effectuée en **15 ms**

## Impact de DNS sur les performances web

### DNS Prefetching

Les navigateurs modernes font du **DNS prefetching** pour accélérer la navigation :

```html
<!-- Le navigateur résout ces DNS à l'avance -->
<link rel="dns-prefetch" href="//cdn.example.com">
<link rel="dns-prefetch" href="//fonts.googleapis.com">
<link rel="dns-prefetch" href="//analytics.google.com">
```

**Gain de temps :**
```
Sans prefetch :
Page chargée → Découvre cdn.example.com → Résolution DNS (50ms) → Télécharge

Avec prefetch :
Page chargée → DNS déjà résolu (pendant le chargement) → Télécharge
Économie : 50-150 ms par domaine !
```

### DNS et latence

La résolution DNS ajoute de la latence lors de la première connexion :

```
Timeline typique d'un chargement de page :

0 ms    : Utilisateur clique sur un lien
0-50 ms : Résolution DNS
50 ms   : DNS résolu, début connexion TCP
100 ms  : TCP établi, début handshake TLS
200 ms  : TLS établi, envoi requête HTTP
250 ms  : Premier byte reçu (TTFB)
800 ms  : Page complètement chargée

→ DNS = ~15-20% du temps avant le premier byte !
```

**Optimisations possibles :**
- Réduire le nombre de domaines différents
- Utiliser un résolveur rapide (1.1.1.1, 8.8.8.8)
- Implémenter le DNS prefetching
- Utiliser un TTL approprié

## DNS et disponibilité

### Pourquoi DNS est critique

```
Scénario : Panne DNS
├─ Les serveurs web fonctionnent ✓
├─ Le réseau fonctionne ✓
├─ Les utilisateurs ont Internet ✓
└─ MAIS : Personne ne peut accéder aux sites !

"Internet est en panne" ≈ "DNS est en panne"
```

**Exemple historique :**
En octobre 2016, une attaque DDoS massive sur Dyn (fournisseur DNS) a rendu inaccessibles :
- Twitter
- Netflix
- Spotify
- GitHub
- Reddit
- Et des centaines d'autres sites

Leurs serveurs fonctionnaient, mais DNS ne répondait pas !

### Redondance DNS

**Best practice : Plusieurs serveurs NS**
```
example.com.    IN  NS  ns1.example.com.  (Serveur primaire)
example.com.    IN  NS  ns2.example.com.  (Serveur secondaire)
example.com.    IN  NS  ns3.example.com.  (Serveur tertiaire)

Idéalement :
• Sur différents réseaux (ASN différents)
• Dans différentes localisations géographiques
• Chez différents fournisseurs
```

**Anycast DNS :**
```
Même adresse IP (ex: 8.8.8.8) annoncée depuis plusieurs endroits

Utilisateur en France → Serveur Google en France
Utilisateur aux USA → Serveur Google aux USA
Utilisateur en Asie → Serveur Google à Singapour

Avantages :
✓ Latence minimale
✓ Haute disponibilité
✓ Protection DDoS (charge distribuée)
```

## DNS dans différents contextes

### DNS public vs DNS privé

**DNS public :**
- Accessible depuis Internet
- Exemples : 8.8.8.8 (Google), 1.1.1.1 (Cloudflare)
- Résout les noms publics

**DNS privé :**
- Utilisé dans les réseaux d'entreprise
- Résout les noms internes
- Peut transférer vers DNS public pour les requêtes externes

```
Réseau d'entreprise

Client demande : serveur-app.internal.company.com
→ DNS privé répond : 10.0.1.50

Client demande : www.google.com
→ DNS privé transfère vers 8.8.8.8
→ 8.8.8.8 répond : 142.250.185.206
```

### Split-horizon DNS

Réponses différentes selon l'origine de la requête :

```
Requête pour : intranet.company.com

Depuis le réseau interne :
→ Réponse : 10.0.1.100 (IP privée)

Depuis Internet :
→ Réponse : NXDOMAIN (n'existe pas)
   ou 203.0.113.50 (IP publique différente)
```

**Cas d'usage :**
- Sécurité (cacher les services internes)
- Performance (diriger vers serveurs locaux)
- Conformité réglementaire

## Résumé des points clés

| Aspect | Description |
|--------|-------------|
| **Rôle** | Traduit noms → IP (annuaire d'Internet) |
| **Architecture** | Distribuée et hiérarchique |
| **Transport** | UDP port 53 (TCP pour cas spéciaux) |
| **Cache** | Multi-niveaux, crucial pour performances |
| **Enregistrements** | A, AAAA, CNAME, MX, NS, TXT, PTR... |
| **Résolution** | Récursive (client→résolveur) et itérative (résolveur→serveurs) |
| **TTL** | Contrôle durée de cache |
| **Sécurité** | Vulnérable sans DNSSEC/DoT/DoH |
| **Impact** | 15-20% latence initiale, critique pour disponibilité |

## Points clés à retenir

🔑 **DNS est l'annuaire d'Internet qui traduit les noms en adresses IP**

🔑 **Sans DNS, Internet serait pratiquement inutilisable**

🔑 **Architecture distribuée et hiérarchique : racine → TLD → domaines**

🔑 **Utilise principalement UDP pour la rapidité, TCP pour les grandes réponses**

🔑 **Le cache à plusieurs niveaux est essentiel pour les performances**

🔑 **TTL contrôle combien de temps les réponses peuvent être cachées**

🔑 **DNS classique n'est ni chiffré ni authentifié (d'où DNSSEC, DoT, DoH)**

🔑 **La résolution DNS ajoute 50-150ms de latence lors de la première connexion**

🔑 **La disponibilité DNS est critique : si DNS tombe, "Internet tombe"**

---

## Ce que nous avons appris

Dans cette section, nous avons exploré :

- ✅ L'histoire et le problème résolu par DNS
- ✅ Les concepts fondamentaux (FQDN, hiérarchie, enregistrements)
- ✅ Le fonctionnement de la résolution DNS (étape par étape)
- ✅ Les différents types de requêtes et résolutions
- ✅ Le rôle crucial du cache et du TTL
- ✅ UDP vs TCP pour DNS
- ✅ Les vulnérabilités de sécurité
- ✅ L'impact sur les performances web
- ✅ L'importance pour la disponibilité
- ✅ Les outils de diagnostic

## Pour aller plus loin

Maintenant que vous comprenez les bases du DNS, nous allons approfondir avec :
- **L'architecture hiérarchique détaillée** (serveurs racine, TLD, autoritaires)
- **Les différents types d'enregistrements** en profondeur
- **Les mécanismes de résolution** récursive et itérative
- **Le cache DNS et le TTL** en détail

---

**Prochaine section : Architecture hiérarchique du DNS** 👉

⏭️ [Architecture hiérarchique](/05-couche-application/02.1-dns-architecture.md)
