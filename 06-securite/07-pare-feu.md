🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.7 Pare-feu : filtrage de paquets, stateful inspection

## Introduction

Un **pare-feu** (firewall en anglais) est un dispositif de sécurité réseau qui contrôle le trafic entrant et sortant selon un ensemble de règles de sécurité prédéfinies. C'est la première ligne de défense d'un réseau, agissant comme une barrière entre un réseau de confiance (interne) et un réseau non fiable (Internet).

```
Analogie du monde réel :

Pare-feu = Contrôle de sécurité à l'entrée d'un bâtiment :

Sans pare-feu :
┌─────────────┐
│   Rue       │ → Tout le monde entre librement
│  (Internet) │    Voleurs, visiteurs, employés
└──────┬──────┘    Aucun contrôle
       │
       ↓
┌──────────────┐
│  Bâtiment    │
│ (Réseau)     │
└──────────────┘

Avec pare-feu :
┌─────────────┐
│   Rue       │
│  (Internet) │
└──────┬──────┘
       │
       ↓
┌──────────────┐
│  CONTRÔLE    │ ← Vérification identité
│  SÉCURITÉ    │   Autorisation d'entrée
│  (Firewall)  │   Détection menaces
└──────┬───────┘   Logging visiteurs
       │
       ↓ (uniquement autorisés)
┌──────────────┐
│  Bâtiment    │
│ (Réseau)     │
└──────────────┘

Règles de sécurité :
✓ Employés badge : AUTORISER
✓ Visiteurs enregistrés : AUTORISER (zone réception)
✓ Inconnus : REFUSER
✓ Liste noire : BLOQUER
```

## Histoire et évolution

### Génération 1 : Packet Filtering (1985-1995)

```
Premiers pare-feu : Filtrage simple de paquets

Fonctionnement :
Examine chaque paquet individuellement
Décision basée sur :
- Adresse IP source
- Adresse IP destination
- Port source
- Port destination
- Protocole (TCP/UDP/ICMP)

Exemple règle :
ALLOW TCP from 203.0.113.0/24 to 10.0.0.50 port 80

Limitations :
✗ Pas de contexte (état connexion)
✗ Vulnérable spoofing
✗ Pas d'inspection contenu
✗ Facile à contourner

Technologies :
- Routeurs ACL (Cisco)
- ipfw (BSD)
- ipchains (Linux anciens)
```

### Génération 2 : Stateful Inspection (1995-2005)

```
Évolution : Inspection avec état

Nouveauté :
Mémorise état des connexions
Table d'état (connection tracking)

Exemple :

Trafic sortant :
Client (10.0.0.5:45678) → Internet (203.0.113.50:80)
Firewall :
  - Autorise sortie
  - Crée entrée dans table d'état
  - Mémorise : 10.0.0.5:45678 ↔ 203.0.113.50:80

Trafic entrant (réponse) :
Internet (203.0.113.50:80) → Client (10.0.0.5:45678)
Firewall :
  - Vérifie table d'état
  - Trouve connexion établie
  - Autorise automatiquement (pas de règle explicite)

Avantages :
✓ Sécurité accrue
✓ Moins de règles nécessaires
✓ Protection contre certaines attaques

Technologies :
- CheckPoint FireWall-1 (pionnier)
- Cisco PIX/ASA
- iptables (Linux)
- pf (BSD)
```

### Génération 3 : Application Layer (2005-2015)

```
Inspection profonde du trafic (DPI - Deep Packet Inspection)

Capacités :
- Analyse contenu applicatif
- Détection protocoles
- Identification applications
- Prévention intrusions (IPS)

Exemple :

Paquet HTTP :
Source : 10.0.0.5:45678
Dest : 203.0.113.50:80
Protocole : TCP

Firewall Gen 2 :
✓ Port 80 ? OK
✓ Connexion établie ? OK
→ AUTORISER

Firewall Gen 3 (Application Layer) :
✓ Port 80 ? OK
✓ Connexion établie ? OK
✓ Analyse payload :
  - Vraiment HTTP ? Vérifier
  - Contient malware ? Scanner
  - Viole policy ? Bloquer
→ DÉCISION INFORMÉE

Technologies :
- Palo Alto Networks
- Fortinet FortiGate
- Cisco FirePOWER
- CheckPoint Next Generation
```

### Génération 4 : Next-Generation (NGFW) (2015-présent)

```
Pare-feu nouvelle génération

Intègre :
✓ Stateful inspection
✓ Application awareness
✓ IPS/IDS
✓ Antivirus/Antimalware
✓ URL filtering
✓ SSL/TLS inspection
✓ Threat intelligence
✓ Cloud integration
✓ Machine learning
✓ Identity awareness

Exemple décision NGFW :

User Alice (alice@company.com)
Application : Dropbox
Destination : dropbox.com
Contenu : Fichier "confidential.docx"

NGFW analyse :
1. Identité : Alice (Marketing dept)
2. Application : Dropbox (cloud storage)
3. Policy : Marketing → Dropbox = INTERDIT
4. DLP : Fichier contient "confidential" = BLOQUER
5. Logging : Alerte sécurité + manager notification

→ BLOQUER + ALERTER

Technologies :
- Palo Alto PA-Series
- Fortinet FortiGate (récent)
- Cisco Firepower
- CheckPoint Quantum
- Sophos XG
```

## Types de pare-feu

### Par emplacement

#### Pare-feu réseau (Network Firewall)

```
Positionnement : Périmètre réseau

Architecture :

Internet
    │
    ↓
[Routeur]
    │
    ↓
[FIREWALL] ← Contrôle tout le trafic
    │
    ├─→ DMZ (serveurs publics)
    │   ├─ Web servers
    │   └─ Mail servers
    │
    └─→ LAN interne
        ├─ Postes de travail
        ├─ Serveurs internes
        └─ Imprimantes

Caractéristiques :
✓ Protège réseau entier
✓ Point de contrôle centralisé
✓ Haute performance (Gbps-Tbps)
✓ Hardware dédié souvent
✓ HA (High Availability)

Types :
- Appliance hardware (Cisco, Palo Alto)
- Appliance virtuelle (VM)
- Software sur serveur (pfSense, OPNsense)

Déploiement :
- Inline (bridge mode)
- Routed (layer 3)
- Transparent (layer 2)
```

#### Pare-feu hôte (Host-based Firewall)

```
Positionnement : Sur chaque machine

┌─────────────────────────────┐
│   Serveur / PC              │
│                             │
│   ┌─────────────────────┐   │
│   │  Applications       │   │
│   └─────────┬───────────┘   │
│             │               │
│   ┌─────────▼───────────┐   │
│   │  Firewall Local     │ ← Protection locale
│   │  (Windows Firewall, │   │
│   │   iptables, etc.)   │   │
│   └─────────┬───────────┘   │
│             │               │
│   ┌─────────▼───────────┐   │
│   │  Network Stack      │   │
│   └─────────────────────┘   │
│                             │
└─────────────────────────────┘

Avantages :
✓ Defense in depth
✓ Protection même si réseau compromis
✓ Règles spécifiques par machine
✓ Granularité par processus

Inconvénients :
✗ Configuration distribuée
✗ Gestion complexe (milliers de machines)
✗ Performance impact sur hôte
✗ Dépend de l'intégrité de l'OS

Technologies :
- Windows Defender Firewall
- iptables/nftables (Linux)
- pf (macOS, BSD)
- Little Snitch (macOS)
- Solutions entreprise (Symantec, McAfee)

Use case :
✓ Serveurs critiques (defense in depth)
✓ Postes de travail
✓ Laptops (en déplacement)
✓ Compliance (PCI-DSS, HIPAA)
```

### Par fonctionnement

#### Pare-feu stateless (Packet Filter)

```
Fonctionnement : Examine paquets individuellement

Décision basée sur header uniquement :
- IP source/destination
- Port source/destination
- Protocole
- Flags TCP

Pas de mémoire entre paquets

Exemple règle :

# iptables (mode stateless)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -j DROP

Problème :

Client → Server : SYN (port 80)
✓ Règle matche : ACCEPT

Server → Client : SYN-ACK
✗ Pas de règle retour explicite
→ Si pas de règle RELATED/ESTABLISHED : DROP
→ Connexion échoue !

Solution nécessaire :
# Ajouter règle retour explicite
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

OU utiliser stateful (mieux) :
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

Avantages stateless :
✓ Très rapide (pas de tracking)
✓ Simple
✓ Prévisible
✓ Faible mémoire

Inconvénients :
✗ Règles bidirectionnelles nécessaires
✗ Vulnérable certaines attaques
✗ Pas de contexte applicatif
✗ Configuration complexe

Usage moderne :
- Routers ACL (filtrage basique)
- Niveau très bas (hardware offload)
- Préfiltrage avant stateful
```

#### Pare-feu stateful (Inspection avec état)

```
Fonctionnement : Mémorise état des connexions

Connection Tracking Table :

┌────────────┬──────────┬──────────┬─────────┬────────┐
│ Src IP:Port│ Dst IP:Pt│ Protocol │ State   │ Timeout│
├────────────┼──────────┼──────────┼─────────┼────────┤
│10.0.0.5:   │203.0.113 │ TCP      │ ESTABL- │ 3600s  │
│  45678     │  .50:80  │          │ ISHED   │        │
├────────────┼──────────┼──────────┼─────────┼────────┤
│10.0.0.10:  │8.8.8.8:  │ UDP      │ NEW     │ 30s    │
│  53821     │  53      │          │         │        │
├────────────┼──────────┼──────────┼─────────┼────────┤
│10.0.0.15:  │198.51.100│ TCP      │ SYN_SENT│ 120s   │
│  12345     │  .20:443 │          │         │        │
└────────────┴──────────┴──────────┴─────────┴────────┘

États TCP :
- NEW : Premier paquet (SYN)
- ESTABLISHED : Connexion établie (après 3-way handshake)
- RELATED : Connexion liée (ex: FTP data après control)
- INVALID : Paquet ne correspond à aucune connexion

Exemple complet :

1. Client → Server : TCP SYN
   Firewall :
   - Vérifie règles : Port 80 autorisé ?
   - Crée entrée : State = NEW
   - ACCEPT

2. Server → Client : SYN-ACK
   Firewall :
   - Cherche dans table
   - Trouve connexion state=NEW
   - Update state = ESTABLISHED
   - ACCEPT (automatique, pas de règle explicite)

3. Client → Server : ACK
   Firewall :
   - State = ESTABLISHED
   - ACCEPT

4. Client ↔ Server : Data exchange
   Firewall :
   - State = ESTABLISHED
   - ACCEPT (tous paquets)

5. Client → Server : FIN
   Firewall :
   - Update state = FIN_WAIT
   - ACCEPT

6. Timeout ou FIN/ACK complet
   Firewall :
   - Remove entrée de table

Configuration iptables :

# Règles stateful
# Autoriser connexions établies et liées
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Autoriser nouvelles connexions vers ports spécifiques
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT

# Bloquer tout le reste
iptables -A INPUT -j DROP

Résultat :
Client peut initier vers 80/443
Réponses automatiquement autorisées (ESTABLISHED)
Autres ports bloqués

Avantages :
✓ Sécurité supérieure
✓ Moins de règles nécessaires
✓ Gestion automatique retours
✓ Protection contre certaines attaques

Inconvénients :
✗ Consommation mémoire (table d'état)
✗ CPU pour tracking
✗ Limite nombre de connexions simultanées
✗ Vulnérable attaques flooding (saturation table)
```

## Filtrage de paquets en détail

### Critères de filtrage

```
Critères Layer 3 (IP) :

1. Adresse IP source :
   - IP spécifique : 203.0.113.50
   - Réseau : 10.0.0.0/24
   - Plage : 192.168.1.10-192.168.1.20
   - ANY : 0.0.0.0/0

2. Adresse IP destination :
   - Même formats que source

3. Protocole :
   - TCP (6)
   - UDP (17)
   - ICMP (1)
   - GRE (47)
   - ESP (50)
   - AH (51)
   - ALL (any)

4. TTL :
   - Détection certaines attaques
   - Rare en practice

5. Fragmentation :
   - Fragment offset
   - More fragments flag
   - Anti-fragmentation attacks

Critères Layer 4 (Transport) :

TCP/UDP :
1. Port source :
   - Port spécifique : 45678
   - Plage : 49152-65535 (ephemeral)
   - ANY

2. Port destination :
   - Service : 80 (HTTP), 443 (HTTPS)
   - Plage : 6000-6010
   - ANY

3. Flags TCP (uniquement TCP) :
   - SYN : Initiation connexion
   - ACK : Acknowledgment
   - FIN : Fermeture connexion
   - RST : Reset
   - PSH : Push data
   - URG : Urgent

   Combinaisons :
   - SYN+ACK
   - FIN+ACK
   - SYN only (nouveau connexion)

ICMP :
1. Type :
   - 0 : Echo Reply
   - 3 : Destination Unreachable
   - 8 : Echo Request (ping)
   - 11 : Time Exceeded

2. Code :
   - Sous-type par type

Critères additionnels :

1. Interface :
   - Entrée : eth0, eth1
   - Sortie : eth0, eth1
   - Direction : IN, OUT, FORWARD

2. Temps :
   - Heure de la journée
   - Jour de la semaine
   - Calendrier

3. Utilisateur/Groupe (NGFW) :
   - alice@company.com
   - Marketing_Group

4. Application (NGFW) :
   - Facebook
   - Dropbox
   - BitTorrent

5. Géolocalisation :
   - Pays source : CN, RU
   - Région : EU, US
```

### Actions de filtrage

```
Actions possibles :

1. ACCEPT / ALLOW / PERMIT :
   Autoriser le paquet
   Continuer traitement normal

2. DROP / DENY :
   Bloquer silencieusement
   Pas de réponse à l'émetteur

   Avantages :
   ✓ Stealth (pas d'info à attaquant)
   ✓ Économise bande passante

   Inconvénients :
   ✗ Debugging difficile (timeout)
   ✗ Applications attendent timeout

3. REJECT :
   Bloquer avec notification
   Envoie ICMP/TCP RST à l'émetteur

   Avantages :
   ✓ Debugging plus facile
   ✓ Applications échouent rapidement

   Inconvénients :
   ✗ Révèle présence firewall
   ✗ Consomme bande passante

4. LOG :
   Enregistrer dans logs
   Continuer traitement (combine avec autre action)

   Exemple :
   LOG + DROP : Bloquer et logger
   LOG + ACCEPT : Autoriser et logger

5. RATE LIMIT :
   Limiter nombre de paquets/connexions
   Exemple : Max 10 connexions/seconde

   Protection contre :
   - Bruteforce
   - DoS
   - Port scanning

6. REDIRECT / NAT :
   Modifier destination
   Exemple : Port forwarding
   80 → 192.168.1.10:8080

7. MARK / TAG :
   Marquer paquet pour traitement ultérieur
   QoS, routing policy, etc.

Exemple combinaisons :

# Log puis drop
iptables -A INPUT -p tcp --dport 23 -j LOG --log-prefix "TELNET: "
iptables -A INPUT -p tcp --dport 23 -j DROP

# Rate limit puis accept
iptables -A INPUT -p tcp --dport 22 -m limit --limit 5/min -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# Reject avec message spécifique
iptables -A INPUT -p tcp --dport 80 -j REJECT --reject-with tcp-reset
```

### Ordre d'évaluation des règles

```
Principe : First Match Wins (généralement)

Table de règles (top-to-bottom) :

┌────┬─────────────────────────────────────┬────────┐
│ #  │ Règle                               │ Action │
├────┼─────────────────────────────────────┼────────┤
│ 1  │ From 10.0.0.0/8 Any → ACCEPT        │ ACCEPT │
│ 2  │ TCP Any:Any → Any:22                │ ACCEPT │
│ 3  │ TCP Any:Any → Any:80                │ ACCEPT │
│ 4  │ TCP Any:Any → Any:443               │ ACCEPT │
│ 5  │ ICMP Echo Request                   │ ACCEPT │
│ 6  │ Any → Any                           │ DROP   │
└────┴─────────────────────────────────────┴────────┘

Évaluation :

Paquet 1 : TCP 10.0.0.5:45678 → 203.0.113.50:80
  Check règle 1 : Source 10.0.0.5 ∈ 10.0.0.0/8 ? OUI
  → ACCEPT (stop, règles 2-6 ignorées)

Paquet 2 : TCP 203.0.113.50:54321 → 10.0.0.10:22
  Check règle 1 : Source 203.0.113.50 ∈ 10.0.0.0/8 ? NON
  Check règle 2 : Dest port 22 ? OUI
  → ACCEPT (stop)

Paquet 3 : TCP 203.0.113.50:12345 → 10.0.0.10:23 (Telnet)
  Check règle 1 : NON
  Check règle 2 : Dest port 22 ? NON
  Check règle 3 : Dest port 80 ? NON
  Check règle 4 : Dest port 443 ? NON
  Check règle 5 : ICMP ? NON (TCP)
  Check règle 6 : Any → Any ? OUI
  → DROP

Importance de l'ordre :

Mauvais ordre :
1. Any → Any : DROP         ← Trop tôt !
2. TCP → Port 80 : ACCEPT   ← Jamais atteint

Résultat : TOUT bloqué

Bon ordre :
1. TCP → Port 80 : ACCEPT
2. TCP → Port 443 : ACCEPT
3. Any → Any : DROP         ← Règle par défaut à la fin

Résultat : Seulement 80/443 autorisés

Best practices ordre :

1. Règles spécifiques d'abord
   - IP spécifiques
   - Ports spécifiques

2. Règles générales ensuite
   - Réseaux
   - Plages de ports

3. Politique par défaut à la fin
   - DENY ALL
   - Ou ALLOW ALL (rare)

4. Exceptions avant règles générales
   - Blocklist avant allowlist
   - Ou inverse selon politique
```

## Stateful Inspection approfondi

### Connection Tracking (conntrack)

```
Mécanisme Linux : netfilter/conntrack

Structure de la table de connexions :

/proc/net/nf_conntrack (Linux) :

ipv4 2 tcp 6 431999 ESTABLISHED src=10.0.0.5 dst=203.0.113.50 \
  sport=45678 dport=80 src=203.0.113.50 dst=10.0.0.5 \
  sport=80 dport=45678 [ASSURED] mark=0 use=1

Décomposition :
- Protocol : TCP
- State : ESTABLISHED
- Timeout : 431999 seconds
- Original direction : 10.0.0.5:45678 → 203.0.113.50:80
- Reply direction : 203.0.113.50:80 → 10.0.0.5:45678
- Flags : [ASSURED] (connexion confirmée bidirectionnelle)

États TCP dans conntrack :

NEW → SYN_SENT → SYN_RECV → ESTABLISHED →
  FIN_WAIT → CLOSE_WAIT → LAST_ACK → TIME_WAIT → CLOSE

Exemple progression :

1. Client → Server : SYN
   Conntrack : NEW

2. Server → Client : SYN-ACK
   Conntrack : SYN_RECV

3. Client → Server : ACK
   Conntrack : ESTABLISHED

4. Data exchange...
   Conntrack : ESTABLISHED

5. Client → Server : FIN
   Conntrack : FIN_WAIT

6. Server → Client : ACK
   Conntrack : CLOSE_WAIT

7. Server → Client : FIN
   Conntrack : LAST_ACK

8. Client → Server : ACK
   Conntrack : TIME_WAIT (2 min)

9. Timeout
   Conntrack : Entry removed

États UDP :

NEW → ESTABLISHED (après reply) → TIMEOUT

UDP est stateless par nature, mais conntrack simule état :

1. Client → Server : DNS query
   Conntrack : NEW

2. Server → Client : DNS response
   Conntrack : ESTABLISHED

3. Timeout (30s default)
   Conntrack : Entry removed

États ICMP :

Echo Request ↔ Echo Reply
Conntrack : Paires request/reply liées

Autres ICMP : Généralement RELATED à connexions existantes
```

### Gestion de la table d'état

```
Limites système :

# Voir limite max connexions
cat /proc/sys/net/netfilter/nf_conntrack_max
# 65536 (default, souvent insuffisant)

# Voir connexions actuelles
cat /proc/sys/net/netfilter/nf_conntrack_count
# 1234

# Augmenter limite (serveur haute charge)
sysctl -w net.netfilter.nf_conntrack_max=1000000

# Mémoire utilisée
cat /proc/net/nf_conntrack | wc -l
1234 connexions

# Chaque entrée : ~300 bytes
# 1M connexions = ~300 MB RAM

Timeouts par protocole :

# TCP
net.netfilter.nf_conntrack_tcp_timeout_established = 432000 (5 days)
net.netfilter.nf_conntrack_tcp_timeout_syn_sent = 120
net.netfilter.nf_conntrack_tcp_timeout_syn_recv = 60
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120

# UDP
net.netfilter.nf_conntrack_udp_timeout = 30
net.netfilter.nf_conntrack_udp_timeout_stream = 120

# ICMP
net.netfilter.nf_conntrack_icmp_timeout = 30

Tuning pour haute charge :

# Réduire timeouts (plus agressif)
sysctl -w net.netfilter.nf_conntrack_tcp_timeout_established=3600  # 1h au lieu de 5 jours
sysctl -w net.netfilter.nf_conntrack_tcp_timeout_time_wait=30     # 30s au lieu de 2 min

# Augmenter max
sysctl -w net.netfilter.nf_conntrack_max=2000000

# Hash table size
sysctl -w net.netfilter.nf_conntrack_buckets=500000

Problème : Table pleine

Symptômes :
- Nouvelles connexions DROP
- Logs : "nf_conntrack: table full, dropping packet"
- Applications timeout

Solutions :
1. Augmenter nf_conntrack_max
2. Réduire timeouts
3. Identifier source flood (netstat, conntrack)
4. Bloquer source malveillante
5. Scale horizontalement (load balancer)

Monitoring :

# Watch connexions en temps réel
watch -n 1 'cat /proc/sys/net/netfilter/nf_conntrack_count'

# Top IPs par nombre de connexions
cat /proc/net/nf_conntrack | awk '{print $5}' | cut -d= -f2 | sort | uniq -c | sort -rn | head

# Exemple output :
#   1523 203.0.113.50  ← IP avec le plus de connexions
#    234 198.51.100.20
#    156 192.0.2.10
```

### Connexions RELATED

```
Concept : Connexions liées à connexion principale

Exemple classique : FTP

FTP utilise deux connexions :
1. Control (port 21) : Commandes
2. Data (port 20 ou high port) : Transfert fichiers

FTP Active mode :

Client                    Server
  |                         |
  | TCP Connect port 21     |
  |------------------------>| Control channel
  |                         |
  | PORT 192.168.1.10,200,1 | (High port = 200*256+1 = 51201)
  |------------------------>|
  |                         |
  |    New connection       |
  |    from server:20       |
  |<------------------------|  Data channel (RELATED)
  |    to client:51201      |
  |                         |

Sans stateful RELATED :
Connexion data bloquée (connexion entrante non sollicitée)

Avec stateful RELATED :
Firewall analyse commande PORT dans control channel
Crée entrée RELATED pour connexion data attendue
Autorise connexion server:20 → client:51201

Configuration :

# Module nécessaire
modprobe nf_conntrack_ftp

# Règle iptables
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

Autres protocoles avec RELATED :

1. FTP (comme ci-dessus)

2. TFTP :
   - Connection UDP initiale
   - Data transfers sur nouveaux ports

3. IRC (DCC) :
   - Chat channel
   - File transfers RELATED

4. SIP (VoIP) :
   - Signaling channel
   - RTP media streams RELATED

5. H.323 (VoIP) :
   - Control
   - Media channels

6. PPTP :
   - Control (TCP 1723)
   - GRE tunnel (RELATED)

Modules helpers :

# Voir modules chargés
lsmod | grep nf_conntrack

# Charger helper
modprobe nf_conntrack_ftp
modprobe nf_conntrack_sip
modprobe nf_conntrack_h323
modprobe nf_conntrack_pptp

# Configuration permanente
echo "nf_conntrack_ftp" >> /etc/modules-load.d/conntrack.conf

Debugging RELATED :

# Voir connexions RELATED
conntrack -L | grep RELATED

# Exemple output :
tcp 6 299 ESTABLISHED src=10.0.0.5 dst=203.0.113.50 sport=45678 dport=21 [ASSURED]
tcp 6 299 RELATED src=203.0.113.50 dst=10.0.0.5 sport=20 dport=51201 [ASSURED]
                ↑ RELATED à connexion FTP control
```

## Zones de sécurité et DMZ

### Concept de zones

```
Zones de sécurité : Segments réseau avec niveaux de confiance différents

Zones typiques :

1. TRUSTED (Interne) :
   - Réseau d'entreprise
   - Utilisateurs internes
   - Serveurs internes
   - Niveau confiance : ÉLEVÉ

2. UNTRUSTED (Internet) :
   - Internet public
   - Niveau confiance : AUCUN

3. DMZ (Demilitarized Zone) :
   - Serveurs publics
   - Accessible depuis Internet
   - Niveau confiance : MOYEN

4. GUEST (Invités) :
   - Wi-Fi visiteurs
   - Accès limité
   - Niveau confiance : FAIBLE

5. MANAGEMENT :
   - Administration
   - Accès équipements
   - Niveau confiance : TRÈS ÉLEVÉ

Règles inter-zones (par défaut) :

TRUSTED → UNTRUSTED : ALLOW (sortie Internet)
TRUSTED → DMZ : ALLOW (accès serveurs publics)
TRUSTED → GUEST : DENY (isolation)

UNTRUSTED → TRUSTED : DENY (bloc total)
UNTRUSTED → DMZ : ALLOW limité (services publics seulement)

DMZ → TRUSTED : DENY (serveurs compromis ne peuvent attaquer interne)
DMZ → UNTRUSTED : ALLOW (mises à jour, etc.)

GUEST → TRUSTED : DENY
GUEST → UNTRUSTED : ALLOW (Internet seulement)
GUEST → DMZ : DENY

MANAGEMENT → ALL : ALLOW (administration)
ALL → MANAGEMENT : DENY (accès restreint)
```

### Architecture DMZ

#### DMZ Simple (Three-Legged)

```
Architecture classique :

                    Internet
                        │
                        ↓
                 ┌──────────┐
                 │  Router  │
                 └─────┬────┘
                       │
                       ↓
                 ┌──────────┐
                 │ FIREWALL │ ← 3 interfaces (legs)
                 └─┬────┬───┘
                   │    │
         ┌─────────┘    └─────────┐
         │                        │
         ↓                        ↓
    ┌─────────┐            ┌──────────┐
    │   DMZ   │            │ Internal │
    │         │            │   LAN    │
    │ Web     │            │          │
    │ Mail    │            │ Users    │
    │ DNS     │            │ Servers  │
    └─────────┘            └──────────┘
    203.0.113.0/24         10.0.0.0/8

Règles firewall :

Internet → DMZ :
  TCP 80, 443 → Web (203.0.113.10)
  TCP 25, 587 → Mail (203.0.113.20)
  UDP 53 → DNS (203.0.113.30)
  Tout le reste : DROP

Internet → Internal :
  DENY ALL

DMZ → Internal :
  DENY ALL (important !)
  Exception : Mail → Internal SMTP (relay)

Internal → DMZ :
  ALLOW (administration, publication contenu)

Internal → Internet :
  ALLOW (avec proxy optionnel)

DMZ → Internet :
  ALLOW (mises à jour, DNS queries)

Avantage :
✓ Serveurs publics isolés de LAN interne
✓ Compromission DMZ ≠ compromission interne

Inconvénient :
✗ Si firewall compromis : tout compromis
```

#### DMZ avec deux firewalls (Defense in Depth)

```
Architecture renforcée :

                    Internet
                        │
                        ↓
                 ┌──────────┐
                 │  Router  │
                 └─────┬────┘
                       │
                       ↓
                ┌─────────────┐
                │ Firewall 1  │ ← Externe (screening router)
                │  (Externe)  │
                └──────┬──────┘
                       │
                       ↓
                  ┌─────────┐
                  │   DMZ   │
                  │         │
                  │ Web     │
                  │ Mail    │
                  └────┬────┘
                       │
                       ↓
                ┌─────────────┐
                │ Firewall 2  │ ← Interne (choke firewall)
                │  (Interne)  │
                └──────┬──────┘
                       │
                       ↓
                ┌──────────┐
                │ Internal │
                │   LAN    │
                └──────────┘

Firewall 1 (Externe) :
- Filtrage grossier
- Protection DDoS
- IPS
- Autorise vers DMZ : 80, 443, 25, 53
- Bloque scans, attaques connues

Firewall 2 (Interne) :
- Filtrage fin
- Internal → DMZ : Administration
- DMZ → Internal : Très restreint
- Dernière ligne de défense

Avantages :
✓ Defense in depth (double barrière)
✓ Compromission DMZ n'expose pas interne directement
✓ Deux firewalls = deux vendors possibles (diversité)

Inconvénients :
✗ Coût double
✗ Complexité configuration
✗ Performance (double inspection)
```

### Micro-segmentation (NGFW moderne)

```
Concept moderne : Segmentation granulaire

Au lieu de zones larges :
┌──────────────┐
│  Internal    │
│  10.0.0.0/8  │ ← Tout mélangé
└──────────────┘

Micro-segmentation :
┌──────────────┐
│ Web Tier     │ Policy
│ 10.1.0.0/24  │───────┐
└──────────────┘       │
                       ↓
┌──────────────┐   ┌────────┐
│ App Tier     │   │Firewall│
│ 10.2.0.0/24  │──→│ NGFW   │
└──────────────┘   └────────┘
                       ↑
┌──────────────┐       │
│ DB Tier      │───────┘
│ 10.3.0.0/24  │
└──────────────┘

Règles granulaires :

Web Tier → App Tier :
  TCP 8080 (HTTP API)
  Source : Web servers
  Dest : App servers
  App : Custom_Web_App

App Tier → DB Tier :
  TCP 3306 (MySQL)
  Source : App servers
  Dest : DB servers
  User : app_service_account

DB Tier → Web Tier :
  DENY ALL (jamais nécessaire)

Avantages :
✓ Blast radius limité (compromission contenue)
✓ Compliance (PCI-DSS, etc.)
✓ Visibilité fine
✓ Zero Trust architecture

Implémentation :
- VMware NSX (software-defined)
- Cisco ACI
- Palo Alto (VM-Series)
- Security groups (AWS, Azure)
```

## Règles de filtrage avancées

### Filtrage par application (NGFW)

```
Application Layer Gateway (ALG) :

Identifie applications malgré :
- Port non standard
- Tunneling
- Obfuscation

Exemple 1 : BitTorrent

BitTorrent peut utiliser :
- Ports aléatoires (pas seulement 6881-6889)
- Port 80 (HTTP standard)
- Chiffrement

Firewall traditionnel :
Port 80 → HTTP ? ALLOW
→ BitTorrent passe (utilise port 80)

NGFW avec App-ID :
Port 80 → Analyse payload
→ Détecte signature BitTorrent
→ Application = BitTorrent
→ Policy : BitTorrent = DENY
→ BLOCK

Exemple 2 : Facebook

User accède facebook.com
NGFW détecte :
- DNS query : facebook.com
- TLS SNI : www.facebook.com
- HTTP Host header : facebook.com
- Patterns dans trafic

→ Application = Facebook
→ Policy : Marketing dept → Facebook = DENY
→ BLOCK

Bases de données applications :

Palo Alto App-ID :
- 3000+ applications
- Mises à jour régulières
- Catégories :
  * Social Media
  * File Sharing
  * Streaming
  * Collaboration
  * Business
  * etc.

Exemple règle :

Policy :
  Source Zone : Internal
  Dest Zone : Internet
  Application : facebook, youtube, netflix
  Action : DENY
  Schedule : Work hours (8am-6pm)
  User : NOT C-Level
```

### Filtrage par utilisateur (Identity-Based)

```
Integration Active Directory :

Traditional firewall :
Source IP : 10.0.0.50
→ Qui est-ce ? Inconnu

NGFW avec User-ID :
Source IP : 10.0.0.50
→ Query AD : Qui est connecté sur 10.0.0.50 ?
→ User : alice@company.com
→ Group : Marketing
→ Apply policy pour Marketing

Mécanismes détection utilisateur :

1. Agent sur AD :
   - Monitor Windows logs
   - Détecte logon/logoff
   - Informe firewall : IP ↔ User mapping

2. Terminal Services Agent :
   - Pour Terminal Server / Citrix
   - Multiple users / IP
   - Granularité session

3. Captive Portal :
   - Pour guests, BYOD
   - Login web
   - Temporary credentials

4. RADIUS/LDAP :
   - Pour VPN
   - 802.1X (Wi-Fi)
   - Authentication centralisée

Exemple politique :

Rule 1 :
  Source User : alice@company.com
  Source Group : Executives
  Application : ANY
  Destination : Internet
  Action : ALLOW
  Log : Yes

Rule 2 :
  Source Group : Marketing
  Application : facebook, linkedin
  Destination : Internet
  Action : ALLOW
  Time : 12:00-13:00 (lunch hour)

Rule 3 :
  Source Group : IT_Admins
  Application : SSH, RDP
  Destination : Server_Subnet
  Action : ALLOW
  MFA : Required

Rule 4 :
  Source Group : Guests
  Destination : Internet
  Action : ALLOW
  Bandwidth : 5 Mbps max
  URL Filter : Block adult, malware

Avantages :
✓ Policies suivent utilisateur (pas IP)
✓ BYOD friendly
✓ Audit par utilisateur
✓ Compliance
```

### URL Filtering et Content Inspection

```
URL Filtering :

Database de catégories :

Business :
  - banking, finance, business
  → ALLOW

Productivity :
  - webmail, news, shopping
  → ALLOW (ou limiter)

Distractions :
  - social-media, games, gambling
  → DENY (work hours)

Malicious :
  - malware, phishing, botnet-c2
  → DENY ALWAYS

Exemple règle :

Source : Internal_Users
URL Category : social-networking
Time : 08:00-18:00 Mon-Fri
Action : DENY
Log : Yes
Alert : Manager if excessive attempts

Source : Internal_Users
URL Category : malware, phishing
Action : DENY
Alert : Security team immediately

SSL/TLS Inspection :

Problème : HTTPS chiffré = contenu invisible

Solution : SSL Inspection (Man-in-the-Middle)

┌────────┐           ┌──────────┐           ┌────────┐
│ Client │◄────TLS──►│ Firewall │◄────TLS──►│ Server │
└────────┘           └──────────┘           └────────┘
                          │
                          ↓
                    Déchiffre
                    Inspecte
                    Re-chiffre

Process :
1. Client initie HTTPS vers google.com
2. Firewall intercepte
3. Firewall établit connexion vers google.com
4. Firewall génère certificat pour google.com
   Signé par CA interne (installée sur clients)
5. Client voit certificat "google.com" (firewall)
6. Firewall déchiffre trafic
7. Inspection : URL, malware, DLP
8. Re-chiffre vers destination réelle

Configuration requise :
- CA firewall installée sur tous clients
- Exceptions (banking, healthcare - privacy)
- CPU puissant (chiffrement/déchiffrement coûteux)

Exceptions SSL Inspection :

# Ne pas inspecter
- Banking sites (confidentialité)
- Healthcare (HIPAA)
- Sites avec certificate pinning
- Applications cassées par inspection

Rule :
  URL Category : financial-services, health-and-medicine
  SSL Inspection : NO
  Action : ALLOW
```

### Data Loss Prevention (DLP)

```
DLP dans firewall :

Détecte et bloque exfiltration de données sensibles

Patterns recherchés :

1. Numéros carte crédit :
   Regex : \b(?:\d{4}[-\s]?){3}\d{4}\b
   Exemple : 4532-1234-5678-9010

2. SSN (US Social Security) :
   Regex : \b\d{3}-\d{2}-\d{4}\b
   Exemple : 123-45-6789

3. Mots-clés sensibles :
   "CONFIDENTIAL", "SECRET", "RESTRICTED"

4. Documents types :
   - .xlsx avec mots-clés
   - .pdf marqués confidentiels
   - Source code (.c, .py, etc.)

Exemple détection :

User envoie email via Gmail (webmail) :
Attachment : financial_report_Q4.xlsx
Contenu : Revenue, Profit (mots-clés sensibles)

Firewall :
1. SSL Inspection (déchiffre HTTPS Gmail)
2. Extraction attachment
3. Scan contenu Excel
4. Détecte : "Confidential" + "Q4 Revenue"
5. Match DLP policy
6. Action : BLOCK upload
7. Alert : Security team
8. Notification : User (violation policy)

Policy DLP :

Rule 1 :
  Data Type : Credit Card Numbers
  Direction : Outbound
  Destination : Internet
  Action : BLOCK
  Alert : Immediate
  Incident : Create case

Rule 2 :
  Data Type : Source Code
  File Extension : .c, .cpp, .py, .java
  Destination : Personal Cloud (Dropbox, etc.)
  Action : BLOCK
  Alert : Manager + Security

Rule 3 :
  Keywords : "CONFIDENTIAL", "SECRET"
  Destination : Personal Email
  Action : BLOCK + Log
  User Notification : "Violation of data policy"

Faux positifs :

Challenge : Éviter blocage légitime

Exemple :
Email contient : "My credit card was stolen: 4532-XXXX-XXXX-9010"
DLP détecte pattern carte crédit
Mais : Déjà masqué (XXXX)

Solution :
- Whitelisting (contexts légitimes)
- Machine learning (réduire faux positifs)
- Workflow approbation (manager peut override)
```

## Implémentations de pare-feu

### iptables (Linux)

**Le pare-feu Linux classique**

```bash
# Architecture iptables

Tables :
- filter : Filtrage de paquets (défaut)
- nat : Network Address Translation
- mangle : Modification paquets
- raw : Exemption connection tracking

Chains (filter table) :
- INPUT : Paquets vers machine locale
- OUTPUT : Paquets depuis machine locale
- FORWARD : Paquets routés à travers machine

Flow :
                    ┌─────────────┐
                    │   NETWORK   │
                    └──────┬──────┘
                           │
                           ↓
                    ┌─────────────┐
                    │ PREROUTING  │ (raw, mangle, nat)
                    └──────┬──────┘
                           │
                ┌──────────┴──────────┐
                │                     │
           Routing                    │
           Decision                   │
                │                     │
        ┌───────▼────────┐     ┌──────▼──────┐
        │     INPUT      │     │   FORWARD   │
        │   (filter)     │     │  (filter)   │
        └───────┬────────┘     └──────┬──────┘
                │                     │
                ↓                     ↓
        ┌───────────────┐      ┌──────────────┐
        │ Local Process │      │ POSTROUTING  │
        └───────┬───────┘      │ (nat, mangle)│
                │              └──────┬───────┘
                ↓                     │
        ┌───────────────┐             │
        │    OUTPUT     │             │
        │   (filter)    │             │
        └───────┬───────┘             │
                │                     │
                └──────────┬──────────┘
                           │
                           ↓
                    ┌─────────────┐
                    │   NETWORK   │
                    └─────────────┘

# Configuration de base

# Politique par défaut : DROP tout
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT  # Sortie autorisée

# Loopback (localhost)
iptables -A INPUT -i lo -j ACCEPT

# Connexions établies et liées
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH (administration)
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT

# HTTP et HTTPS
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT

# ICMP (ping)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Log paquets droppés (avant DROP final)
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables DROP: " --log-level 7

# Exemples avancés

# Protection SSH bruteforce
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# Explication :
# Si plus de 4 connexions SSH en 60 secondes : DROP

# SYN flood protection
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# Port knocking (technique avancée)
# Séquence secrète de ports pour ouvrir SSH

iptables -N KNOCKING
iptables -N GATE1
iptables -N GATE2
iptables -N GATE3
iptables -N PASSED

# Knock port 1234
iptables -A GATE1 -p tcp --dport 1234 -m recent --name AUTH1 --set -j DROP
iptables -A GATE1 -j DROP

# Knock port 5678 (after 1234)
iptables -A GATE2 -m recent --name AUTH1 --remove
iptables -A GATE2 -p tcp --dport 5678 -m recent --name AUTH2 --set -j DROP
iptables -A GATE2 -j GATE1

# Knock port 9012 (after 5678)
iptables -A GATE3 -m recent --name AUTH2 --remove
iptables -A GATE3 -p tcp --dport 9012 -m recent --name AUTH3 --set -j DROP
iptables -A GATE3 -j GATE1

# SSH after sequence
iptables -A INPUT -m recent --name AUTH3 --remove
iptables -A INPUT -p tcp --dport 22 -m recent --name AUTH3 --rcheck -j ACCEPT

# Géo-blocking (avec ipset)
# Bloquer trafic de Chine, Russie

ipset create china hash:net
ipset create russia hash:net

# Charger IPs (depuis fichier)
while read IP; do ipset add china $IP; done < china-ips.txt
while read IP; do ipset add russia $IP; done < russia-ips.txt

iptables -A INPUT -m set --match-set china src -j DROP
iptables -A INPUT -m set --match-set russia src -j DROP

# NAT (Masquerading pour partage Internet)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding
# Forwarder port 8080 externe → 192.168.1.10:80 interne
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.10:80
iptables -A FORWARD -p tcp -d 192.168.1.10 --dport 80 -j ACCEPT

# Sauvegarder règles
iptables-save > /etc/iptables/rules.v4

# Restaurer règles (au boot)
iptables-restore < /etc/iptables/rules.v4

# Lister règles
iptables -L -v -n --line-numbers

# Supprimer règle spécifique
iptables -D INPUT 5  # Supprime règle #5 de chain INPUT

# Flush toutes les règles (DANGER)
iptables -F
```

### nftables (Linux moderne)

**Remplaçant moderne d'iptables**

```bash
# Avantages nftables vs iptables

✓ Syntaxe plus claire et cohérente
✓ Performance supérieure
✓ Moins de code kernel
✓ Pas de séparation IPv4/IPv6
✓ Transactions atomiques
✓ Meilleure gestion ensembles (sets)

# Configuration nftables

# Fichier /etc/nftables.conf

#!/usr/sbin/nft -f

# Flush all
flush ruleset

# Table principale
table inet filter {
  # Chain INPUT
  chain input {
    type filter hook input priority 0; policy drop;

    # Loopback
    iif lo accept

    # Connexions établies
    ct state established,related accept

    # SSH
    tcp dport 22 ct state new accept

    # HTTP/HTTPS
    tcp dport { 80, 443 } ct state new accept

    # ICMP
    ip protocol icmp accept
    ip6 nexthdr icmpv6 accept

    # Log avant drop
    limit rate 5/minute log prefix "nftables drop: "

    # Drop reste (policy)
  }

  # Chain FORWARD
  chain forward {
    type filter hook forward priority 0; policy drop;
  }

  # Chain OUTPUT
  chain output {
    type filter hook output priority 0; policy accept;
  }
}

# Table NAT
table inet nat {
  chain prerouting {
    type nat hook prerouting priority -100;

    # Port forwarding
    tcp dport 8080 dnat to 192.168.1.10:80
  }

  chain postrouting {
    type nat hook postrouting priority 100;

    # Masquerading (partage Internet)
    oifname "eth0" masquerade
  }
}

# Sets (efficace pour listes IPs)
table inet filter {
  set blacklist {
    type ipv4_addr
    flags interval
    elements = {
      203.0.113.0/24,
      198.51.100.50,
      192.0.2.0/24
    }
  }

  chain input {
    # ... (règles précédentes)

    # Bloquer IPs blacklist
    ip saddr @blacklist drop
  }
}

# Named counters
table inet filter {
  counter http_traffic {}
  counter ssh_attempts {}

  chain input {
    tcp dport 80 counter name "http_traffic" accept
    tcp dport 22 counter name "ssh_attempts" accept
  }
}

# Voir counters
nft list counter inet filter http_traffic

# Protection avancée
table inet filter {
  # Rate limiting SSH
  chain input {
    tcp dport 22 ct state new \
      limit rate 5/minute accept
    tcp dport 22 ct state new drop
  }

  # Protection SYN flood
  chain input {
    tcp flags syn tcp flags != syn,ack ct state new \
      limit rate over 10/second drop
  }
}

# Commandes utiles

# Lister tout
nft list ruleset

# Lister table spécifique
nft list table inet filter

# Ajouter règle dynamiquement
nft add rule inet filter input tcp dport 3306 accept

# Supprimer règle (par handle)
nft -a list chain inet filter input  # Voir handles
nft delete rule inet filter input handle 5

# Flush table
nft flush table inet filter

# Reload config
nft -f /etc/nftables.conf

# Activer au boot (systemd)
systemctl enable nftables
systemctl start nftables
```

### pfSense / OPNsense

**Firewalls open-source basés sur FreeBSD**

```
pfSense / OPNsense :

Base : FreeBSD + pf (packet filter)
Interface : Web UI
Cible : SMB, entreprises, home users avancés

Fonctionnalités :

✓ Firewall stateful
✓ NAT / Port forwarding
✓ VPN (OpenVPN, IPsec, WireGuard)
✓ DHCP server
✓ DNS forwarder/resolver
✓ Traffic shaping (QoS)
✓ Load balancing
✓ High Availability (CARP)
✓ Captive portal
✓ IDS/IPS (Suricata, Snort)
✓ Packages (plugins)

Configuration Web UI :

Firewall → Rules → WAN

Add Rule :
  Action : Pass / Block / Reject
  Interface : WAN
  Protocol : TCP
  Source : any
  Destination : WAN address
  Destination Port : 80 (HTTP)
  Description : "Allow HTTP from Internet"

  Advanced options :
    - Log packets
    - Gateway (routing)
    - Schedule
    - Advanced (TCP flags, states, etc.)

NAT → Port Forward :

Add :
  Interface : WAN
  Protocol : TCP
  Destination : WAN address
  Destination Port : 8080
  Redirect Target IP : 192.168.1.10
  Redirect Target Port : 80
  Description : "Forward 8080 to internal web server"

  Result : Automatic firewall rule created

VPN → OpenVPN → Wizards :

Server Setup :
  Type : Remote Access (SSL/TLS)
  Protocol : UDP
  Port : 1194
  Tunnel Network : 10.8.0.0/24
  DNS Server : 10.0.0.1 (internal)

  Authentication :
    - Certificate Authority
    - Server certificate
    - Client certificates

  Result : OpenVPN server configured + firewall rules

Traffic Shaper (QoS) :

Queues :
  - VoIP : Priority 7, Bandwidth 1 Mbps guaranteed
  - Business : Priority 5, Bandwidth 5 Mbps
  - Default : Priority 1, Best effort

Rules assign traffic to queues :
  Source Port 5060 (SIP) → VoIP queue
  Destination Port 3389 (RDP) → Business queue

High Availability (CARP) :

Primary pfSense : 192.168.1.1 (Master)
Secondary pfSense : 192.168.1.2 (Backup)
Virtual IP (CARP) : 192.168.1.254

Config sync enabled :
Changes on primary automatically sync to secondary

Failover :
Primary down → Secondary assumes VIP
Transparent for clients (using 192.168.1.254)

Packages populaires :

- Suricata : IDS/IPS (intrusion detection/prevention)
- pfBlockerNG : Geo-blocking, ad-blocking, malware blocking
- Squid : Proxy cache
- SquidGuard : URL filtering
- HAProxy : Load balancer
- FreeRADIUS : Authentication server

Monitoring :

Status → System Logs → Firewall :
  Real-time log des paquets bloqués/autorisés

Status → Monitoring :
  Graphs : Traffic, CPU, Memory, States

Diagnostics → States :
  Connection tracking table
  Active connections

Diagnostics → Packet Capture :
  tcpdump intégré
  Capture sur interface spécifique
```

### Firewalls commerciaux

#### Palo Alto Networks

```
Gamme PA-Series :

PA-220 : SMB (500-1000 Mbps)
PA-3220 : Medium enterprise (5-10 Gbps)
PA-5450 : Large enterprise (60+ Gbps)

Fonctionnalités uniques :

App-ID :
  Identification applications Layer 7
  3000+ applications signature
  Indépendant du port

User-ID :
  Intégration Active Directory
  Policies par utilisateur/groupe
  Transparent authentication

Content-ID :
  Antivirus
  Anti-spyware
  Vulnerability protection
  URL filtering
  File blocking
  Data filtering

Threat Prevention :
  Signatures IPS (intrusion prevention)
  DNS sinkholing (malware C2 blocking)
  WildFire (cloud malware analysis)

SSL Decryption :
  Forward proxy (outbound)
  Inbound inspection (inbound)
  Certificate management

GlobalProtect :
  VPN client
  Pre-logon security
  HIP (Host Information Profile)

Panorama :
  Centralized management
  Multi-firewall
  Log collection
  Reporting

Prix :
  Hardware : 3K-500K€
  Subscriptions : 1K-100K€/an (Threat Prevention, URL Filtering, etc.)
  Support : 20% annual
```

#### Fortinet FortiGate

```
Gamme FortiGate :

FortiGate 60F : SOHO/SMB (10 Gbps)
FortiGate 200F : Medium business (25 Gbps)
FortiGate 1500D : Large enterprise (320 Gbps)

Security Fabric :
  Intégration produits Fortinet :
  - FortiGate (firewall)
  - FortiAnalyzer (logs, reporting)
  - FortiManager (management)
  - FortiMail (email security)
  - FortiWeb (WAF)
  - FortiSandbox (malware analysis)
  - FortiAP (Wi-Fi)
  - FortiSwitch (switching)

FortiOS :
  OS unique tous devices
  CLI + Web UI

Security Profiles :
  - Antivirus
  - Web Filtering
  - Application Control
  - IPS
  - DNS Filtering
  - DLP
  - SSL Inspection

SD-WAN :
  Built-in SD-WAN
  Multiple WAN links
  Load balancing
  Failover
  Application steering

Prix :
  Généralement 30-40% moins cher que Palo Alto
  Bon rapport performance/prix
  Populaire SMB et MSP
```

#### Cisco Firepower

```
Gamme :

Firepower 1010 : Small branch (3 Gbps)
Firepower 2130 : Enterprise (15 Gbps)
Firepower 4150 : Data center (70 Gbps)

Cisco FTD (Firepower Threat Defense) :
  Fusion :
  - Cisco ASA (traditional firewall)
  - Firepower NGFW features

Management :
  - FMC (Firepower Management Center) : On-prem
  - CDO (Cisco Defense Orchestrator) : Cloud

Snort :
  Open-source IPS engine (Cisco owns)
  Integration dans Firepower

Talos :
  Cisco threat intelligence
  Updates signatures IPS/AV

SecureX :
  Unified visibility
  Threat response
  Cross-product integration

Prix :
  Comparable Palo Alto
  Licensing complexe
  Enterprise-focused
```

## Conclusion

Les pare-feu sont un élément fondamental de la sécurité réseau, ayant évolué de simples filtres de paquets à des systèmes complexes d'inspection et de prévention des menaces.

**Points clés à retenir** :

```
Évolution pare-feu :

Génération 1 (1985) : Packet filtering
  ✓ Filtrage basique IP/Port
  ✗ Pas de contexte

Génération 2 (1995) : Stateful inspection
  ✓ Connection tracking
  ✓ Contexte connexions
  ✗ Pas d'inspection applicative

Génération 3 (2005) : Application layer
  ✓ Inspection profonde (DPI)
  ✓ Identification applications
  ✗ Complexité accrue

Génération 4 (2015+) : NGFW
  ✓ Tout ce qui précède +
  ✓ Identity-aware
  ✓ Threat intelligence
  ✓ Cloud integration
  ✓ Machine learning

Types de filtrage :

Stateless :
  - Rapide
  - Simple
  - Chaque paquet indépendant
  → Usage : Préfiltrage, ACL

Stateful :
  - Connection tracking
  - Contexte TCP/UDP
  - Efficace et sécurisé
  → Usage : Standard pare-feu

Application-aware :
  - Inspection Layer 7
  - Identification applications
  - Granularité maximale
  → Usage : NGFW, entreprise

Architecture :

Zones de sécurité :
  - Trusted (interne)
  - Untrusted (Internet)
  - DMZ (serveurs publics)
  - Guest
  - Management

DMZ essentiel :
  ✓ Isole serveurs publics
  ✓ Protège réseau interne
  ✓ Defense in depth

Implémentations :

Open-source :
  - iptables/nftables (Linux)
  - pf (BSD)
  - pfSense/OPNsense (appliance)

  → Gratuit, flexible, puissant
  → Bon pour SMB, lab, apprentissage

Commercial :
  - Palo Alto (leader technique)
  - Fortinet (bon rapport qualité/prix)
  - Cisco Firepower (entreprise)
  - CheckPoint (legacy, encore utilisé)

  → Support, fonctionnalités avancées
  → Entreprise, compliance

Fonctionnalités NGFW :

Essentielles :
  ✓ Stateful inspection
  ✓ NAT/PAT
  ✓ VPN (IPsec, SSL)
  ✓ High Availability
  ✓ Logging/Reporting

Avancées :
  ✓ Application control
  ✓ User identification
  ✓ IPS/IDS
  ✓ Antivirus/Antimalware
  ✓ URL filtering
  ✓ SSL inspection
  ✓ DLP
  ✓ Threat intelligence
  ✓ Sandboxing

Best practices :

Règles :
1. Default deny (bloquer par défaut)
2. Least privilege (minimum nécessaire)
3. Règles spécifiques avant générales
4. Documenter chaque règle
5. Review régulier (cleanup)

Sécurité :
1. Hardening (désactiver non-utilisé)
2. Patching régulier
3. Logging centralisé
4. Monitoring actif
5. Backup configuration

Architecture :
1. Defense in depth (multiple couches)
2. Segmentation réseau
3. DMZ pour services publics
4. HA pour critical
5. Separate management network
```

**Règles d'or** :

```
1. Default Deny :
   Bloquer par défaut, autoriser explicitement

2. Least Privilege :
   Minimum nécessaire pour fonctionner

3. Defense in Depth :
   Firewall réseau + firewall hôte

4. Monitoring :
   Logs = inutiles si pas analysés

5. Maintenance :
   Review règles trimestriel minimum

6. Testing :
   Tester avant prod
   Backup avant changement

7. Documentation :
   Chaque règle documentée (pourquoi?)

8. Patching :
   Firewall à jour = critique
```

Les pare-feu continuent d'évoluer avec l'adoption du cloud, la micro-segmentation, et Zero Trust. La prochaine génération intègrera davantage d'intelligence artificielle pour la détection de menaces et l'automatisation des politiques de sécurité.

Dans la section suivante (6.8), nous conclurons le module sécurité avec les bonnes pratiques générales de sécurité réseau TCP/IP.

⏭️ [Bonnes pratiques de sécurisation](/06-securite/08-bonnes-pratiques.md)
