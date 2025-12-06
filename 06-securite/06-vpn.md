🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.6 VPN : principes et protocoles

## Introduction

Un **VPN** (Virtual Private Network - Réseau Privé Virtuel) est une technologie qui crée un tunnel sécurisé à travers un réseau public (typiquement Internet) pour connecter des réseaux privés ou des utilisateurs distants comme s'ils étaient directement connectés au réseau de l'organisation.

```
Analogie du monde réel :

Sans VPN = Parler dans un café bondé :
- Tout le monde peut entendre
- Conversations interceptables
- Identité visible
- Localisation évidente

Avec VPN = Tunnel privé blindé :
- Canal privé et sécurisé
- Impossible d'écouter de l'extérieur
- Identité masquée
- Localisation cachée
- Communication comme si dans même bureau
```

**Problème résolu par les VPN** :

```
Années 1990 : Entreprises multi-sites

Problème :
Bureau Paris ←──────→ Bureau Lyon
             Internet

Besoins :
- Connecter les réseaux locaux
- Accès aux ressources comme si même réseau
- Sécurité (données sensibles)
- Coût raisonnable

Solutions avant VPN :
1. Lignes dédiées (leased lines) :
   ✓ Sécurisé
   ✓ Performance garantie
   ✗ TRÈS COÛTEUX (milliers €/mois)
   ✗ Pas flexible

2. RNIS/Frame Relay :
   ✓ Moins cher que leased lines
   ✗ Encore coûteux
   ✗ Complexe à déployer

Solution VPN :
✓ Utilise Internet existant (quasi gratuit)
✓ Chiffrement pour sécurité
✓ Flexible et scalable
✓ Coût : dizaines €/mois au lieu de milliers

Résultat :
VPN = Démocratisation de la connectivité inter-sites
```

## Principe de fonctionnement

### Tunneling

**Concept fondamental** : Encapsuler un protocole réseau dans un autre.

```
Tunneling = "Paquet dans un paquet"

Paquet original (privé) :
┌────────────────────────────────────┐
│ IP: 10.1.0.5 → 10.2.0.10           │ ← Adresses privées
│ TCP: 45678 → 443                   │
│ Data: [données applicatives]       │
└────────────────────────────────────┘
         ↓
    Encapsulation VPN
         ↓
Paquet VPN (public) :
┌────────────────────────────────────────────────┐
│ Outer IP: 203.0.113.10 → 198.51.100.20         │ ← IPs publiques
│ VPN Header (ESP, OpenVPN, WireGuard, etc.)     │
├════════════════════════════════════════════════┤
│ ╔════════════════════════════════════════════╗ │
│ ║ PAQUET ORIGINAL (chiffré)                  ║ │
│ ║ IP: 10.1.0.5 → 10.2.0.10                   ║ │
│ ║ TCP: 45678 → 443                           ║ │
│ ║ Data: [données]                            ║ │
│ ╚════════════════════════════════════════════╝ │
└────────────────────────────────────────────────┘

Transmission :
1. Paquet original créé (10.1.0.5 → 10.2.0.10)
2. Encapsulation VPN avec chiffrement
3. Transmission Internet (IPs publiques)
4. Décapsulation VPN côté récepteur
5. Routage vers destination finale (10.2.0.10)

Avantages :
✓ Adresses privées invisibles
✓ Données chiffrées
✓ Route via Internet (pas de ligne dédiée)
✓ Transparent pour applications
```

### Les trois piliers d'un VPN

```
1. CHIFFREMENT (Confidentialité) :
   - Données illisibles pour tiers
   - Algorithmes : AES, ChaCha20
   - Empêche espionnage

2. AUTHENTIFICATION (Identité) :
   - Vérifier qui se connecte
   - Méthodes : certificats, PSK, username/password
   - Empêche accès non autorisé

3. INTÉGRITÉ (Non-modification) :
   - Garantir données non modifiées
   - MAC/HMAC
   - Détecte altération

Ces trois ensemble = VPN sécurisé
Un seul manquant = Vulnérable
```

## Types de VPN

### 1. Site-to-Site VPN (Gateway-to-Gateway)

```
Architecture :

Réseau A                Internet               Réseau B
┌──────────────┐                          ┌──────────────┐
│ 10.1.0.0/24  │                          │ 10.2.0.0/24  │
│              │                          │              │
│ ┌──────┐     │                          │   ┌──────┐   │
│ │ PC   │     │                          │   │Server│   │
│ │.0.10 │     │                          │   │.0.50 │   │
│ └──┬───┘     │                          │   └───┬──┘   │
│    │         │                          │       │      │
│ ┌──▼──────┐  │                          │ ┌─────▼───┐  │
│ │VPN      │ ◄┼──────────────────────────┼►│VPN      │  │
│ │Gateway  │  │    Tunnel VPN            │ │Gateway  │  │
│ │203.0.   │  │   (IPsec, WireGuard)     │ │198.51.  │  │
│ │113.10   │  │                          │ │100.20   │  │
│ └─────────┘  │                          │ └─────────┘  │
└──────────────┘                          └──────────────┘

Caractéristiques :
- Connecte deux réseaux complets
- Gateways gèrent le VPN
- Utilisateurs ignorent le VPN (transparent)
- Always-on (permanent)
- Bidirectionnel

Flux de trafic :
PC (10.1.0.10) veut accéder Server (10.2.0.50)
→ Paquet envoyé vers gateway locale
→ Gateway A encapsule en VPN
→ Tunnel Internet
→ Gateway B décapsule
→ Routage vers Server (10.2.0.50)
→ Réponse via même chemin inverse

Use cases :
✓ Bureaux multiples d'une entreprise
✓ Datacenter → Cloud
✓ Interconnexion partenaires
✓ Backup site connection
```

**Exemple concret** :

```
Entreprise avec 5 sites :

Paris (HQ)      Lyon        Marseille
10.1.0.0/24     10.2.0.0/24 10.3.0.0/24
    │               │           │
    └───────┬───────┴───────────┘
            │
        Internet
            │
    ┌───────┴───────┐
    │               │
Toulouse        Bordeaux
10.4.0.0/24     10.5.0.0/24

Configuration :
- 10 tunnels VPN (chaque site vers chaque autre)
- OU hub-and-spoke (tous via Paris)

Hub-and-spoke (simplifié) :
Paris (Hub)
├── Tunnel vers Lyon
├── Tunnel vers Marseille
├── Tunnel vers Toulouse
└── Tunnel vers Bordeaux

Avantage hub-and-spoke :
- 4 tunnels au lieu de 10
- Gestion centralisée
- Paris = point de contrôle

Inconvénient :
- Lyon → Marseille passe par Paris
- Latence additionnelle
- Paris = SPOF
```

### 2. Remote Access VPN (Road Warrior)

```
Architecture :

Utilisateurs distants      Internet      Entreprise
┌─────────────┐                       ┌──────────────┐
│ Laptop      │                       │ VPN Gateway  │
│ (Anywhere)  │◄─────────────────────►│              │
│ VPN Client  │    Tunnel VPN         │ 198.51.100.10│
│ 203.0.113.50│   (IKEv2, OpenVPN)    │              │
└─────────────┘                       └──────┬───────┘
                                             │
┌─────────────┐                       ┌──────▼────────┐
│ Smartphone  │                       │Réseau interne │
│ (4G/5G)     │◄─────────────────────►│ 10.0.0.0/8    │
│ VPN App     │    Tunnel VPN         │               │
│ 198.51.200.5│                       │ - Serveurs    │
└─────────────┘                       │ - Fichiers    │
                                      │ - Applications│
                                      └───────────────┘

Caractéristiques :
- Un utilisateur/device vers réseau
- Client VPN sur device utilisateur
- On-demand (connecté quand nécessaire)
- IP virtuelle assignée dynamiquement
- Authentification individuelle

Processus de connexion :
1. Utilisateur lance client VPN
2. Authentification (username/password + 2FA)
3. Tunnel établi
4. IP virtuelle assignée (ex: 172.16.100.25)
5. Routes configurées (trafic entreprise → tunnel)
6. Accès réseau complet

Configuration IP :
Avant VPN :
- IP locale : 192.168.1.100 (Wi-Fi café)
- Gateway : 192.168.1.1
- DNS : 8.8.8.8

Après VPN :
- IP locale : toujours 192.168.1.100
- IP virtuelle : 172.16.100.25 (VPN)
- Gateway : Via tunnel pour 10.0.0.0/8
- DNS : 10.0.1.10 (DNS interne entreprise)

Use cases :
✓ Télétravail
✓ Déplacements professionnels
✓ Accès temporaire consultants
✓ Bring Your Own Device (BYOD)
```

**Split Tunneling vs Full Tunneling** :

```
Full Tunneling (default sécurisé) :
┌──────────────┐
│ Laptop VPN   │
└──────┬───────┘
       │
       │ TOUT le trafic
       ↓
   [VPN Tunnel]
       ↓
   [Gateway]
       ├──→ Réseau interne (10.0.0.0/8)
       └──→ Internet (via proxy entreprise)

Avantages :
✓ Tout le trafic inspecté (sécurité)
✓ Logging complet
✓ Protection contre malware

Inconvénients :
✗ Bande passante gateway sollicitée
✗ Latence pour trafic Internet
✗ UX dégradée (YouTube, Netflix lents)

Split Tunneling (performance) :
┌──────────────┐
│ Laptop VPN   │
└──┬───────┬───┘
   │       │
   │       │ Trafic Internet direct
   │       └──→ Internet (YouTube, etc.)
   │
   │ Trafic interne
   ↓
[VPN Tunnel]
   ↓
[Gateway]
   └──→ Réseau interne seulement

Avantages :
✓ Performance Internet normale
✓ Moins de charge gateway
✓ Meilleure UX

Inconvénients :
✗ Trafic Internet non protégé
✗ Possible exfiltration données
✗ Malware peut contourner protections

Recommandation :
- Full tunneling : Politique stricte, haute sécurité
- Split tunneling : Performance, confiance utilisateurs
```

### 3. Client-to-Site VPN

```
Variante du Remote Access :

Client VPN          Internet          Site entreprise
┌──────────┐                         ┌──────────────┐
│Individual│◄───────────────────────►│ Gateway VPN  │
│  User    │   Tunnel sécurisé       │              │
│          │                         │   ┌────────┐ │
│ IP virt: │                         │   │Servers │ │
│172.16.   │                         │   └────────┘ │
│100.50    │                         │   ┌────────┐ │
└──────────┘                         │   │Storage │ │
                                     │   └────────┘ │
                                     └──────────────┘

Similaire Remote Access mais :
- Accent sur accès ressources spécifiques
- Peut être web-based (SSL VPN)
- Granularité accès plus fine
- Souvent sans client (browser)
```

### 4. Peer-to-Peer VPN (Mesh VPN)

```
Architecture maillée :

Node A ←─────────→ Node B
  ↕                  ↕
  └────→ Node C ←────┘
           ↕
        Node D

Chaque node connecté directement aux autres
Pas de gateway central

Technologies :
- WireGuard (mesh facile)
- Tinc
- ZeroTier
- Tailscale

Avantages :
✓ Pas de SPOF (Single Point of Failure)
✓ Latence optimale (direct peer-to-peer)
✓ Scalable (ajout nodes simple)
✓ Résilient

Inconvénients :
✗ Complexité configuration (N×(N-1)/2 tunnels)
✗ Gestion clés complexe
✗ Pas de point de contrôle central

Use case :
✓ Réseau distribué (blockchain, CDN)
✓ Gaming servers
✓ Collaboration décentralisée
✓ Mesh networks communautaires
```

## Protocoles VPN

### IPsec VPN

**Déjà couvert en détail dans sections 6.5, 6.5.1, 6.5.2**

```
Résumé IPsec :

Protocoles :
- IKE (IKEv1/IKEv2) : Négociation
- ESP : Chiffrement + Intégrité
- (AH : Déprécié)

Modes :
- Transport : Host-to-host
- Tunnel : Network-to-network (VPN)

Avantages :
✓ Standard ouvert (interopérable)
✓ Support natif OS (Windows, Linux, macOS)
✓ Performance excellente
✓ Sécurité éprouvée
✓ Hardware acceleration

Inconvénients :
✗ Complexité configuration
✗ NAT problématique (NAT-T requis)
✗ Firewall traversal difficile
✗ Overhead modéré

Use cases :
✓ Site-to-Site VPN (standard de facto)
✓ Cloud interconnection (AWS, Azure)
✓ Remote access (IKEv2 mobile)
✓ Entreprise (Cisco, Juniper, etc.)
```

### OpenVPN

**Protocole VPN open-source le plus populaire**

```
Caractéristiques :

Année : 2001
Créateur : James Yonan
Licence : GPL (open source)
Transport : TCP ou UDP
Port : 1194 (default), configurable
Chiffrement : OpenSSL library (TLS)

Architecture :
┌────────────────────────────────────┐
│ OpenVPN = TLS + Custom Protocol    │
├────────────────────────────────────┤
│ Layer application (user-space)     │
│ Pas dans kernel                    │
├────────────────────────────────────┤
│ TUN/TAP virtual interface          │
│ - TUN : Layer 3 (IP)               │
│ - TAP : Layer 2 (Ethernet)         │
└────────────────────────────────────┘

Fonctionnement :

1. Tunnel TLS établi (comme HTTPS)
2. Interface virtuelle créée (tun0/tap0)
3. Paquets IP routés vers interface
4. Encapsulation dans TLS
5. Transmission TCP/UDP
6. Décapsulation côté récepteur
7. Injection dans réseau
```

**Configuration OpenVPN** :

```bash
# Serveur OpenVPN (/etc/openvpn/server.conf)

port 1194
proto udp                  # ou tcp
dev tun                    # Layer 3 (IP routing)

# Certificats (PKI)
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh2048.pem

# Réseau VPN
server 10.8.0.0 255.255.255.0    # Pool IP clients
ifconfig-pool-persist ipp.txt     # IP persistantes

# Routage
push "route 10.0.0.0 255.0.0.0"  # Routes vers réseau interne
push "redirect-gateway def1"      # Full tunneling (optionnel)
push "dhcp-option DNS 10.0.1.10"  # DNS interne

# Sécurité
tls-auth ta.key 0                 # HMAC authentification
cipher AES-256-GCM                # Chiffrement
auth SHA256                       # Intégrité

# Performance
keepalive 10 120                  # Dead peer detection
persist-key
persist-tun
user nobody
group nogroup

# Logging
status /var/log/openvpn-status.log
log-append /var/log/openvpn.log
verb 3

# Client OpenVPN (client.ovpn)

client
dev tun
proto udp

remote vpn.company.com 1194

# Certificats
ca ca.crt
cert client.crt
key client.key

# Sécurité
tls-auth ta.key 1
cipher AES-256-GCM
auth SHA256

# Options
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server

verb 3
```

**Avantages OpenVPN** :

```
✓ Traverse facilement firewalls (port 443/TCP)
✓ Très configurable
✓ Support multi-plateforme excellent
  - Windows, Linux, macOS
  - Android, iOS
  - Routeurs (OpenWRT, DD-WRT)
✓ Open source (auditabilité)
✓ Communauté large et active
✓ Mature et stable
✓ Mode TCP fallback (réseaux difficiles)
✓ Support 2FA/MFA facile
✓ Logging détaillé

Exemple traversal :
Port 443/TCP → Impossible à bloquer (HTTPS)
→ OpenVPN indiscernable de HTTPS
→ Fonctionne même en Chine, Iran, etc.
```

**Inconvénients OpenVPN** :

```
✗ Performance inférieure à WireGuard/IPsec
  - User-space (pas kernel)
  - Overhead TLS
✗ Consommation CPU plus élevée
✗ Configuration complexe (PKI nécessaire)
✗ Pas de support natif OS (client requis)
✗ Latence plus élevée (handshake TLS)
✗ Mobile : Battery drain
✗ Code base large (complexité)

Benchmarks typiques :
IPsec : 800-1000 Mbps
WireGuard : 1000-1500 Mbps
OpenVPN : 200-400 Mbps (UDP)
OpenVPN : 100-200 Mbps (TCP, pire)

Raison :
- User-space processing
- Context switches
- TLS overhead
- Pas d'accélération hardware
```

### WireGuard

**VPN moderne ultra-rapide et simple**

```
Caractéristiques :

Année : 2015 (stable 2020)
Créateur : Jason A. Donenfeld
Licence : GPL
Code : ~4000 lignes (vs 100K+ OpenVPN)
Transport : UDP uniquement
Port : 51820 (default)
Chiffrement : ChaCha20-Poly1305, Curve25519

Philosophie :
"Cryptographically sound, simple, fast"
- Pas de choix d'algorithmes (un seul, le meilleur)
- Configuration minimale
- Code auditable (petit)
- Performance maximale (kernel-space)

Architecture :
┌────────────────────────────────────┐
│ WireGuard = Kernel Module          │
├────────────────────────────────────┤
│ Interface réseau virtuelle (wg0)   │
│ Comme eth0, mais chiffrée          │
├────────────────────────────────────┤
│ Cryptokey routing                  │
│ IP → Clé publique mapping          │
└────────────────────────────────────┘
```

**Configuration WireGuard** :

```bash
# Serveur WireGuard (/etc/wireguard/wg0.conf)

[Interface]
Address = 10.0.0.1/24              # IP du serveur dans VPN
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY    # Généré avec wg genkey

# Forwarding et NAT
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Peer 1 (Client)
[Peer]
PublicKey = CLIENT1_PUBLIC_KEY     # Clé publique du client
AllowedIPs = 10.0.0.2/32           # IP assignée au client
PersistentKeepalive = 25           # NAT keepalive

# Peer 2 (Client)
[Peer]
PublicKey = CLIENT2_PUBLIC_KEY
AllowedIPs = 10.0.0.3/32
PersistentKeepalive = 25

# Client WireGuard (/etc/wireguard/wg0.conf)

[Interface]
Address = 10.0.0.2/32              # IP du client dans VPN
PrivateKey = CLIENT_PRIVATE_KEY
DNS = 10.0.1.10                    # DNS interne

[Peer]
PublicKey = SERVER_PUBLIC_KEY      # Clé publique du serveur
Endpoint = vpn.company.com:51820   # IP:Port du serveur
AllowedIPs = 0.0.0.0/0             # Route tout via VPN (full tunnel)
# AllowedIPs = 10.0.0.0/8          # Ou seulement réseau interne
PersistentKeepalive = 25

# Génération de clés
# Serveur
wg genkey | tee server_private.key | wg pubkey > server_public.key

# Client
wg genkey | tee client_private.key | wg pubkey > client_public.key

# Activation
sudo wg-quick up wg0

# Status
sudo wg show
# interface: wg0
#   public key: SERVER_PUBLIC_KEY
#   private key: (hidden)
#   listening port: 51820
#
# peer: CLIENT1_PUBLIC_KEY
#   endpoint: 203.0.113.50:54321
#   allowed ips: 10.0.0.2/32
#   latest handshake: 25 seconds ago
#   transfer: 1.23 MiB received, 456.78 KiB sent
```

**Cryptokey Routing** :

```
Concept unique à WireGuard :

Routing basé sur clés publiques, pas IPs

Table de routage :
┌──────────────────────┬─────────────────────┐
│ AllowedIPs           │ Public Key          │
├──────────────────────┼─────────────────────┤
│ 10.0.0.2/32          │ CLIENT1_PUBKEY      │
│ 10.0.0.3/32          │ CLIENT2_PUBKEY      │
│ 192.168.1.0/24       │ SITE_B_PUBKEY       │
└──────────────────────┴─────────────────────┘

Fonctionnement :
1. Paquet à destination 10.0.0.2
2. Lookup : 10.0.0.2 → CLIENT1_PUBKEY
3. Chiffrement avec CLIENT1_PUBKEY
4. Envoi vers endpoint du peer

Avantages :
✓ Simple et élégant
✓ Pas de configuration routing complexe
✓ Sécurité intégrée (clé = identité)
✓ Roaming transparent (IP endpoint change OK)
```

**Avantages WireGuard** :

```
✓ PERFORMANCE EXCEPTIONNELLE
  - 1-2 Gbps sur CPU moderne
  - Souvent plus rapide qu'IPsec
  - Kernel-space (pas user-space)

✓ Simplicité configuration
  - Fichier config ~10 lignes
  - Pas de PKI complexe
  - Clés symétriques simples

✓ Code minimal et auditable
  - 4000 lignes vs 100K+ (OpenVPN)
  - Surface d'attaque réduite
  - Bugs rares

✓ Cryptographie moderne
  - Pas de choix (un seul bon algorithme)
  - ChaCha20-Poly1305 (rapide, sûr)
  - Curve25519 (elliptique moderne)

✓ Mobile-friendly
  - Consommation batterie minimale
  - Reconnexion instantanée
  - Roaming automatique

✓ Support OS croissant
  - Linux kernel (mainline depuis 5.6)
  - Windows, macOS, iOS, Android
  - Routeurs (OpenWRT)

✓ Firewall friendly
  - UDP simple
  - Traverse NAT facilement
  - Pas de handshake complexe

Exemple performance :
Test : 2x serveurs 10 Gbps NIC
IPsec : 4-6 Gbps
OpenVPN : 1-2 Gbps
WireGuard : 8-9 Gbps

WireGuard = Le plus rapide !
```

**Inconvénients WireGuard** :

```
✗ Jeune et moins mature
  - Stable depuis 2020 seulement
  - Moins d'années production

✗ Pas de Perfect Forward Secrecy dynamique
  - Clés statiques (rotées manuellement)
  - Pas de rekeying automatique
  - Compromission clé = tout déchiffrable

✗ Pas de support natif authentification
  - Pas de username/password
  - Pas d'intégration AD/LDAP native
  - Nécessite couche externe (wg-access-server, etc.)

✗ Logging IP problématique
  - Garde mapping IP ↔ clé en mémoire
  - Privacy concern pour certains

✗ Pas de fallback TCP
  - UDP uniquement
  - Si UDP bloqué → pas de connexion

✗ Configuration granularité limitée
  - Pas de support VLAN, multicast natif
  - Moins flexible qu'OpenVPN

Quand ne pas utiliser :
- Besoin auth centralisée (AD/LDAP) sans addon
- Besoin Perfect Forward Secrecy strict
- Réseau bloque UDP complètement
- Besoin fonctionnalités avancées (compression, etc.)
```

### L2TP/IPsec

**Layer 2 Tunneling Protocol sur IPsec**

```
Architecture en couches :

┌─────────────────────────────────────┐
│ IPsec (ESP)                         │ ← Sécurité
│ - Chiffrement                       │
│ - Authentification                  │
├─────────────────────────────────────┤
│ L2TP                                │ ← Tunneling
│ - Encapsulation Layer 2             │
│ - PPP session                       │
├─────────────────────────────────────┤
│ PPP                                 │ ← Link layer
│ - Authentication (PAP/CHAP/MSCHAPv2)│
│ - IP assignment                     │
└─────────────────────────────────────┘

Pourquoi double encapsulation ?
- L2TP : Pas de chiffrement (juste tunnel)
- IPsec : Ajoute sécurité
- Ensemble : Tunnel sécurisé

Historique :
- L2TP créé 1999 (fusion PPTP + L2F)
- Souvent combiné avec IPsec
- Support natif Windows, macOS, iOS
```

**Fonctionnement** :

```
Établissement connexion L2TP/IPsec :

1. Phase IPsec :
   a. IKE Phase 1 (ISAKMP SA)
      - Authentification gateway (PSK ou certificat)
      - Établissement canal sécurisé IKE

   b. IKE Phase 2 (IPsec SA)
      - Négociation ESP
      - Tunnel IPsec établi (UDP 500, 4500)

2. Phase L2TP :
   a. L2TP tunnel établi (UDP 1701, dans IPsec)
      - Control messages

   b. PPP session :
      - LCP negotiation
      - Authentification (MSCHAPv2)
      - IPCP (IP address assignment)
      - Client obtient IP virtuelle

3. Trafic :
   Data → PPP → L2TP → IPsec ESP → Network

Ports utilisés :
- UDP 500 : IKE
- UDP 4500 : NAT-T (si NAT)
- UDP 1701 : L2TP (encapsulé dans ESP)
```

**Configuration Windows** :

```
# Serveur L2TP/IPsec (Windows Server)

1. Installation :
   - Role : Remote Access (VPN, Routing)
   - Enable L2TP/IPsec

2. Configuration :
   IPsec Policy :
   - Pre-shared key : "SuperSecretPSK123!"
   - OU Certificat serveur

   IP Pool :
   - 172.16.100.1-172.16.100.254

   Authentication :
   - MS-CHAPv2
   - OU RADIUS/NPS

3. Firewall :
   Allow UDP 500, 1701, 4500
   Allow IP Protocol 50 (ESP)

# Client Windows :

Settings → Network → VPN → Add VPN
- Type : L2TP/IPsec
- Server : vpn.company.com
- PSK : SuperSecretPSK123!
- Username/Password : Active Directory
```

**Avantages L2TP/IPsec** :

```
✓ Support natif OS
  - Windows, macOS, iOS, Android
  - Pas de client tiers nécessaire

✓ Sécurité éprouvée (IPsec)

✓ Authentification flexible
  - Username/password
  - RADIUS
  - Active Directory

✓ Simple pour utilisateurs finaux
  - Configuration native OS
  - Familier

✓ Stable et mature
```

**Inconvénients L2TP/IPsec** :

```
✗ Double encapsulation = Overhead
  - PPP + L2TP + IPsec
  - Performance inférieure

✗ Complexité configuration serveur
  - IPsec + L2TP + PPP
  - Multiples composants

✗ NAT problématique
  - ESP + UDP ensemble
  - NAT-T requis
  - Parfois instable

✗ Firewall traversal difficile
  - Multiples ports et protocoles
  - Souvent bloqué réseaux restrictifs

✗ Pas optimal moderne
  - Design années 90
  - Alternatives meilleures existent

Statut actuel (2024) :
- Encore utilisé (legacy)
- Remplacé progressivement par IKEv2 ou WireGuard
- Support natif OS maintenu (compatibilité)
```

### IKEv2/IPsec

**Version moderne d'IPsec pour mobile**

```
Évolution :
IPsec + IKEv1 (1998) → Complexe, lent
IKEv2 (2005, RFC 7296) → Simplifié, rapide, mobile

Améliorations IKEv2 vs IKEv1 :

✓ Moins de messages :
  IKEv1 : 6+ messages (main mode)
  IKEv2 : 4 messages (2 RTT)

✓ Built-in NAT-T

✓ MOBIKE (Mobility and Multihoming)
  - Change d'IP sans reconnexion
  - 4G → Wi-Fi transparent

✓ Built-in DPD (Dead Peer Detection)

✓ Support EAP (extensible auth)
  - Username/password
  - 2FA/MFA
  - RADIUS

✓ Plus robuste erreurs
```

**Handshake IKEv2** :

```
Échange simplifié :

Initiator                           Responder
    |                                   |
    | IKE_SA_INIT (req)                 |
    |   - Proposals                     |
    |   - Nonce, DH                     |
    |---------------------------------->|
    |                                   |
    |       IKE_SA_INIT (resp)          |
    |   - Chosen proposal               |
    |   - Nonce, DH                     |
    |<----------------------------------|
    |                                   |
    | IKE_AUTH (req) [ENCRYPTED]        |
    |   - IDi, AUTH                     |
    |   - SA, TSi, TSr                  |
    |---------------------------------->|
    |                                   |
    |       IKE_AUTH (resp) [ENCRYPTED] |
    |   - IDr, AUTH                     |
    |   - SA, TSi, TSr                  |
    |<----------------------------------|
    |                                   |
    [IKE SA + IPsec SA établies]

Total : 4 messages (2 round-trips)
vs 6-9 messages IKEv1
```

**MOBIKE (Mobilité)** :

```
Scénario mobile :

Utilisateur en déplacement :
1. Connecté 4G (IP : 203.0.113.50)
   → VPN établi, IP virtuelle : 172.16.100.25

2. Entre zone Wi-Fi (nouvelle IP : 192.168.1.100)
   → Connexion réseau change

3. Sans MOBIKE :
   ✗ VPN tunnel cassé
   ✗ Reconnexion nécessaire
   ✗ Applications interrompues

4. Avec MOBIKE (IKEv2) :
   ✓ Détection changement IP
   ✓ UPDATE message envoyé
   ✓ Tunnel migré transparently
   ✓ IP virtuelle conservée (172.16.100.25)
   ✓ Applications continuent sans interruption

Messages MOBIKE :

Client (nouvelle IP 192.168.1.100) → Server
INFORMATIONAL {
    UPDATE_SA_ADDRESSES
    COOKIE2 (optionnel)
}

Server → Client (nouvelle IP)
INFORMATIONAL {
    UPDATE_SA_ADDRESSES confirmed
}

Tunnel maintenant :
192.168.1.100 ↔ Server (mis à jour)

Délai : <1 seconde
Applications : Pas d'interruption
```

**Avantages IKEv2/IPsec** :

```
✓ Performance excellente
  - Même que IPsec classique
  - Hardware acceleration

✓ Mobilité native (MOBIKE)
  - Idéal smartphones
  - Roaming 4G/5G ↔ Wi-Fi

✓ Reconnexion rapide
  - 2 RTT seulement
  - Latency minimale

✓ Support natif OS mobile
  - iOS (natif depuis iOS 8)
  - Android (depuis 4.0)
  - Windows 10/11
  - macOS

✓ Sécurité IPsec complète

✓ Authentification flexible (EAP)
  - RADIUS
  - Active Directory
  - 2FA

✓ Battery efficient (mobile)
  - Optimisé consommation
  - Keep-alive intelligent
```

**Configuration strongSwan IKEv2** :

```bash
# /etc/ipsec.conf

config setup
    charondebug="ike 2, knl 2"
    uniqueids=never

conn ikev2-vpn
    auto=add
    type=tunnel

    # IKEv2 spécifique
    keyexchange=ikev2
    mobike=yes              # Enable mobilité

    # Serveur
    left=%any
    leftsubnet=0.0.0.0/0
    leftcert=server.crt
    leftid=@vpn.company.com

    # Client
    right=%any
    rightsourceip=172.16.100.0/24    # Pool IP clients
    rightdns=10.0.1.10

    # Authentification
    leftauth=pubkey
    rightauth=eap-mschapv2   # Username/password
    eap_identity=%identity

    # Crypto
    ike=aes256-sha256-modp2048!
    esp=aes256-sha256!

    # Lifetime
    ikelifetime=24h
    lifetime=1h

    # DPD
    dpdaction=clear
    dpddelay=300s

# /etc/ipsec.secrets
: RSA server.key
username : EAP "password"
```

### SSTP (Secure Socket Tunneling Protocol)

**VPN propriétaire Microsoft**

```
Caractéristiques :

Créateur : Microsoft (2007)
Transport : TCP port 443 (HTTPS)
Chiffrement : TLS/SSL
OS : Windows principalement

Architecture :
HTTP → TLS → PPP → IP

Avantages :
✓ Traverse tous firewalls (port 443)
✓ Indiscernable de HTTPS
✓ Natif Windows
✓ Sécurité TLS

Inconvénients :
✗ Propriétaire Microsoft
✗ Support limité non-Windows
✗ TCP-over-TCP (performance)
✗ Pas open source

Statut :
- Utilisé dans environnements Windows
- Remplacé par IKEv2 souvent
```

### PPTP (Point-to-Point Tunneling Protocol)

**VPN obsolète - NE PLUS UTILISER**

```
Créateur : Microsoft (1996)
Chiffrement : MPPE (40-128 bit)
Authentification : MS-CHAPv2

CASSÉ ET DANGEREUX :

✗ MS-CHAPv2 cassable en <1 jour
✗ MPPE faible (128-bit seulement)
✗ Vulnérabilités multiples
✗ NSA peut déchiffrer (revelations Snowden)

Pas d'excuse pour utiliser PPTP en 2024

Remplacer par :
→ WireGuard
→ IKEv2/IPsec
→ OpenVPN

Seulement raison historique :
- Legacy devices (très anciens)
- Même alors : chercher alternative
```

## Comparaison des protocoles VPN

### Tableau comparatif complet


| Critère | IPsec | OpenVPN | WireGuard | IKEv2 | L2TP | PPTP |
|---------|-------|---------|-----------|-------|------|------|
| **Année** | 1995 | 2001 | 2020 | 2005 | 1999 | 1996 |
| **Open Source** | OUI | OUI | OUI | OUI | OUI | NON (MS) |
| **Vitesse** | ★★★★★<br>Excellent | ★★★<br>Moyenne | ★★★★★★<br>Excellent | ★★★★★<br>Excellent | ★★★<br>Moyenne | ★★★★<br>Bonne |
| **Sécurité** | ★★★★★<br>Très haute | ★★★★★<br>Très haute | ★★★★★<br>Très haute | ★★★★★<br>Très haute | ★★★★★<br>Bonne | ✗<br>CASSÉ |
| **Configuration Simple** | ★★<br>Complexe | ★★★<br>Moyenne | ★★★★★<br>Très simple | ★★★<br>Moyenne | ★★<br>Complexe | ★★★★<br>Simple |
| **Natif OS** | OUI<br>W, L, M, iOS, A | NON<br>Client nécessaire | Partiel<br>W, L, M, iOS, A | OUI<br>W, L, M, iOS, A | OUI<br>W, L, M, iOS, A | OUI<br>W, L, M, iOS, A |
| **Firewall Traversal** | ★★<br>Difficile | ★★★★★<br>Facile | ★★★★<br>Bon | ★★★<br>Moyen | ★★<br>Difficile | ★★★<br>Moyen |
| **Mobile** | ★★★<br>Bon | ★★<br>Battery drain | ★★★★★<br>Excellent | ★★★★★<br>Excellent MOBIKE | ★★<br>Moyen | ★★<br>Moyen |
| **Port/Proto** | 50, 500, 4500<br>ESP, UDP | 1194<br>TCP/UDP | 51820<br>UDP | 500, 4500<br>ESP | 500, 1701<br>ESP | 1723<br>TCP + GRE |
| **Use Case Principal** | Site-to-Site | Universel Polyvalent | Moderne Rapide | Mobile VPN | Legacy Compat | ✗<br>JAMAIS |
| **Recommandation 2024** | ★★★★★<br>OUI | ★★★★<br>OUI | ★★★★★<br>OUI | ★★★★<br>OUI | ★★<br>Éviter | ✗<br>JAMAIS |


**Légende :** W = Windows, L = Linux, M = macOS, iOS, A = Android

### Choix du protocole selon contexte

```
Site-to-Site VPN :
1. IPsec (IKEv2)        ← Standard, interopérable
2. WireGuard            ← Performance maximale
3. OpenVPN              ← Fallback si firewall strict

Remote Access (Entreprise) :
1. IKEv2/IPsec          ← Natif, mobile excellent
2. OpenVPN              ← Compatible tout, flexible
3. WireGuard            ← Moderne, rapide (si auth externe OK)

Remote Access (Personnel/Privacy) :
1. WireGuard            ← Rapide, simple, moderne
2. OpenVPN              ← Universellement supporté
3. IKEv2                ← Si natif OS suffisant

Cloud Interconnection :
1. IPsec                ← Standard AWS/Azure/GCP
2. WireGuard            ← Si supporté par cloud provider

Mobile (iOS/Android) :
1. IKEv2/IPsec          ← MOBIKE, battery efficient
2. WireGuard            ← Performance, battery OK
3. OpenVPN              ← Si fonctionnalités spécifiques requises

Legacy Systems :
1. L2TP/IPsec           ← Support natif ancien
2. OpenVPN              ← Compatible presque tout
```

## Architecture et déploiement

### VPN Gateway (concentrateur)

```
Rôle : Point d'entrée centralisé pour VPN

┌────────────────────────────────────────────────┐
│            VPN Gateway/Concentrator            │
├────────────────────────────────────────────────┤
│ • Authentification utilisateurs                │
│ • Gestion tunnels (100-10000+ simultanés)      │
│ • Chiffrement/déchiffrement                    │
│ • Routage trafic VPN ↔ LAN                     │
│ • Logging et monitoring                        │
│ • Enforcement policies                         │
└────────────────────────────────────────────────┘

Déploiement typique :

Internet
    │
    ↓
[Firewall]
    │
    ↓
[VPN Gateway] ←──→ [Authentication Server]
    │                  (RADIUS, AD)
    ↓
[Core Switch]
    │
    ├─→ [Serveurs]
    ├─→ [Storage]
    └─→ [Applications]

Dimensionnement :

Small (1-50 users) :
- pfSense, EdgeRouter
- 100-500 Mbps throughput
- 2-4 CPU cores

Medium (50-500 users) :
- Cisco ASA 5515-X
- Fortinet FortiGate 100F
- 1-2 Gbps throughput
- 8-16 CPU cores

Large (500-10000+ users) :
- Cisco ASA 5585-X
- Palo Alto PA-5200
- Fortinet FortiGate 1000F
- 10-40 Gbps throughput
- Clustering/HA

Very Large (10K-100K+ users) :
- F5 BIG-IP
- Multiple clustered gateways
- Load balancing
- Geographic distribution
```

### Haute disponibilité VPN

```
Configuration HA (Active-Passive) :

           Internet
               │
        ┌──────┴──────┐
        │             │
    ┌───▼──┐      ┌───▼──┐
    │VPN   │      │VPN   │
    │GW #1 │◄────►│GW #2 │
    │Active│VRRP  │Standby
    └───┬──┘      └───┬──┘
        │             │
        └──────┬──────┘
               │
          [Core Switch]

Virtual IP : 198.51.100.10
GW #1 : 198.51.100.11 (Master)
GW #2 : 198.51.100.12 (Backup)

Clients se connectent à VIP 198.51.100.10

Failover automatique :
1. GW #1 down détecté (VRRP heartbeat)
2. GW #2 devient Master (<3 secondes)
3. GW #2 assume VIP
4. Clients reconnectent (transparent si stateful)

Configuration Active-Active :

           Internet
               │
        ┌──────┴──────┐
        │             │
    ┌───▼──┐      ┌───▼──┐
    │VPN   │      │VPN   │
    │GW #1 │      │GW #2 │
    │Active│      │Active│
    └───┬──┘      └───┬──┘
        │             │
        └──────┬──────┘
               │
          [Core Switch]

DNS Round-Robin ou Load Balancer :
vpn.company.com
  → 198.51.100.11 (50% clients)
  → 198.51.100.12 (50% clients)

Avantages :
✓ Utilisation 100% capacité
✓ Load balancing
✓ Pas de ressource idle

Inconvénients :
- Plus complexe
- Session persistence si stateful
```

### Split DNS et routing

```
Split DNS :

Problème :
Client VPN doit résoudre :
- internal.company.com → DNS interne
- google.com → DNS public

Solution :

[VPN Client]
    │
    ├─ Queries *.company.com
    │  → DNS interne (10.0.1.10)
    │
    └─ Queries autres
       → DNS public (8.8.8.8)

Configuration (Windows) :
VPN push DNS suffixes :
- DNS Suffix : company.com
- DNS Server : 10.0.1.10
- Public DNS : 8.8.8.8

dnsmasq config :
server=/company.com/10.0.1.10
server=8.8.8.8

Split Routing :

Full Tunnel (default) :
Route 0.0.0.0/0 → VPN
→ TOUT le trafic via VPN

Split Tunnel :
Route 10.0.0.0/8 → VPN
Route 172.16.0.0/12 → VPN
Route 0.0.0.0/0 → Local gateway

ip route add 10.0.0.0/8 via 172.16.100.1 dev tun0
ip route add 172.16.0.0/12 via 172.16.100.1 dev tun0
```

## VPN et sécurité

### Authentification multi-facteur

```
VPN + 2FA/MFA :

Couches d'authentification :

1. Something you know :
   - Username/password
   - PIN

2. Something you have :
   - Token TOTP (Google Authenticator)
   - Hardware token (YubiKey)
   - SMS code (moins sûr)
   - Push notification (Duo, Okta)

3. Something you are :
   - Biométrie (rare VPN)

Flux VPN + TOTP :

User → VPN Gateway
  Username : alice
  Password : SecurePass123!

Gateway → RADIUS Server
  Authenticate alice / SecurePass123!

RADIUS → OK, request 2FA

Gateway → User
  Enter TOTP code :

User →
  TOTP code : 123456

Gateway → RADIUS
  Verify TOTP for alice : 123456

RADIUS → OK, authenticated

Gateway → User
  VPN tunnel established
  IP assigned : 172.16.100.25

Implémentation (RADIUS + Google Authenticator) :

# FreeRADIUS + PAM Google Authenticator
# /etc/raddb/users
alice Cleartext-Password := "SecurePass123!"
    Auth-Type := PAM

# PAM config
# /etc/pam.d/radiusd
auth required pam_google_authenticator.so

Résultat :
✓ Password compromise seul ≠ accès
✓ TOTP change toutes les 30s
✓ Sécurité renforcée significativement
```

### Certificate-based authentication

```
Authentification par certificat client :

PKI VPN :

[Root CA]
   │
   ├─ [Server Certificate]
   │    vpn.company.com
   │
   └─ [Client Certificates]
        ├─ alice@company.com
        ├─ bob@company.com
        └─ charlie@company.com

Avantages vs username/password :

✓ Pas de credential stealing possible
✓ Révocation granulaire (CRL/OCSP)
✓ Forte authentification cryptographique
✓ Pas de password fatigue
✓ Intégration smartcard possible

OpenVPN avec certificats :

# Génération CA
easyrsa init-pki
easyrsa build-ca

# Certificat serveur
easyrsa build-server-full server nopass

# Certificats clients
easyrsa build-client-full alice
easyrsa build-client-full bob

# Configuration serveur
tls-auth ta.key 0
ca ca.crt
cert server.crt
key server.key

# Configuration client
tls-auth ta.key 1
ca ca.crt
cert alice.crt
key alice.key

Révocation :
easyrsa revoke alice
easyrsa gen-crl

# Serveur
crl-verify crl.pem

WireGuard :
- Clés publiques = certificats
- Pas de PKI (plus simple)
- Révocation = retirer peer config
```

### Logging et monitoring

```
Logs VPN essentiels :

1. Connexions/Déconnexions :
   - Timestamp
   - Username
   - IP source
   - IP assignée
   - Durée session

2. Authentification :
   - Succès/Échecs
   - Méthode (password, cert, 2FA)
   - Raison échec

3. Trafic :
   - Bytes in/out
   - Protocoles utilisés
   - Destinations

4. Erreurs :
   - Échecs chiffrement
   - Timeouts
   - Reconnexions

Format log (syslog) :

Dec 6 14:30:45 vpn-gw openvpn[1234]: alice/203.0.113.50:54321 \
  MULTI: Learn: 172.16.100.25 -> alice/203.0.113.50:54321

Dec 6 14:30:45 vpn-gw openvpn[1234]: alice/203.0.113.50:54321 \
  Peer Connection Initiated with [AF_INET]203.0.113.50:54321

Dec 6 15:45:12 vpn-gw openvpn[1234]: alice/203.0.113.50:54321 \
  Connection reset, restarting [0]

Dec 6 15:45:15 vpn-gw openvpn[1234]: SIGTERM received, \
  sending exit notification to all clients

Monitoring (Prometheus + Grafana) :

Métriques :
- vpn_connected_clients
- vpn_bytes_in/out
- vpn_auth_failures_total
- vpn_tunnel_uptime
- vpn_cipher_errors

Alertes :
- Auth failures > 10/min → Bruteforce attack
- Tunnel down > 5 min → Connectivity issue
- Bandwidth > 80% capacity → Upgrade needed
```

## Problèmes courants et troubleshooting

### Problème 1 : Connexion VPN établie mais pas de réseau

```
Symptôme :
- VPN connecté (tunnel OK)
- Ping vers gateway VPN OK
- Ping vers réseau interne FAIL

Causes possibles :

1. Routing incorrect :
   # Vérifier routes
   ip route show
   # ou Windows
   route print

   Problème : Route 10.0.0.0/8 manquante
   Solution : Ajouter route
   ip route add 10.0.0.0/8 via 172.16.100.1 dev tun0

2. Firewall gateway bloque :
   # Vérifier iptables
   iptables -L FORWARD -v

   Solution :
   iptables -A FORWARD -i tun0 -j ACCEPT
   iptables -A FORWARD -o tun0 -j ACCEPT

3. NAT pas configuré :
   # Vérifier NAT
   iptables -t nat -L POSTROUTING

   Solution :
   iptables -t nat -A POSTROUTING -s 172.16.100.0/24 \
     -o eth0 -j MASQUERADE

4. DNS incorrect :
   # Test DNS
   nslookup internal.company.com

   Problème : Utilise DNS public
   Solution : Push DNS via VPN
   push "dhcp-option DNS 10.0.1.10"

5. Firewall réseau interne :
   Serveurs internes bloquent IP VPN pool
   Solution : Autoriser 172.16.100.0/24
```

### Problème 2 : Performance dégradée

```
Symptôme :
- VPN connecté
- Très lent (latence élevée, débit faible)

Diagnostic :

1. Test baseline :
   # Sans VPN
   curl -o /dev/null https://fast.com
   → 100 Mbps

   # Avec VPN
   curl -o /dev/null https://internal.company.com/test
   → 5 Mbps (!)

2. Identifier goulot :

   a) MTU issues :
      ping -M do -s 1400 internal-server
      → Fragmentation ?

      Solution :
      Réduire MTU interface VPN
      ip link set dev tun0 mtu 1400

      Ou MSS clamping :
      iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN \
        -j TCPMSS --set-mss 1360

   b) Chiffrement CPU-bound :
      top
      → openvpn 90% CPU

      Solutions :
      - Passer à WireGuard (plus efficace)
      - Cipher moins gourmand (AES-128-GCM vs AES-256-CBC)
      - Hardware crypto acceleration

   c) Compression :
      OpenVPN :
      comp-lzo yes    # Activer compression

      Trade-off :
      + Meilleur débit si CPU OK
      - CPU additionnel

   d) TCP vs UDP :
      Si OpenVPN sur TCP → TRÈS lent

      Raison : TCP-over-TCP catastrophe
      TCP VPN corrige pertes
      TCP applicatif corrige aussi
      → Double retransmission, timeout exponential

      Solution : TOUJOURS UDP pour OpenVPN

   e) Congestion gateway :
      # Vérifier load
      uptime
      → load average: 25.0 (!)

      Solution : Scale up gateway ou load balancing

3. Latence élevée :

   # Ping via VPN
   ping internal-server
   → 200ms

   Causes :
   - Distance géographique
   - Routing suboptimal
   - Gateway overload

   Diagnostic :
   traceroute internal-server

   Si plusieurs hops internes :
   → Optimiser routing
```

### Problème 3 : Reconnexions fréquentes

```
Symptôme :
- VPN se déconnecte toutes les 5-10 minutes
- Reconnexion nécessaire

Causes :

1. NAT timeout :
   NAT mapping expire
   → Connexion drop

   Solution : Keepalive

   OpenVPN :
   keepalive 10 120
   # Ping toutes les 10s, timeout après 120s

   WireGuard :
   PersistentKeepalive = 25
   # Packet toutes les 25s

2. Idle timeout serveur :
   # Vérifier config serveur
   Idle timeout : 300s (5 min)

   Solution :
   - Augmenter timeout
   - Activer keepalive client

3. Firewall state timeout :
   Firewall drop connexion après inactivité

   Solution :
   - Keepalive (comme ci-dessus)
   - Augmenter timeout firewall

4. Mobile roaming (4G/Wi-Fi) :
   Change d'IP → tunnel cassé

   Solution :
   - IKEv2 avec MOBIKE
   - WireGuard (roaming friendly)
   ✗ Pas OpenVPN (reconnexion nécessaire)

5. DPD (Dead Peer Detection) trop agressif :
   # IKEv2
   dpddelay=30s
   dpdtimeout=90s

   Trop court si latence variable

   Solution : Augmenter timeouts
   dpddelay=60s
   dpdtimeout=300s
```

### Problème 4 : Impossible d'établir tunnel

```
Symptôme :
- Connexion échoue immédiatement
- Timeout ou erreur auth

Diagnostic :

1. Problème réseau de base :
   # Ping serveur VPN
   ping vpn.company.com

   Si FAIL :
   → DNS, routing, ou serveur down

2. Port bloqué :
   # Test connectivité port
   nc -zv vpn.company.com 1194
   # OpenVPN UDP

   ou
   telnet vpn.company.com 443
   # OpenVPN TCP fallback

   Si timeout :
   → Firewall bloque
   → Essayer autre port/protocole

3. Authentification :
   # Logs client
   openvpn --config client.ovpn --verb 4

   Erreur typique :
   "TLS Error: TLS key negotiation failed"
   → Certificat invalide, expiré, ou mauvaise CA

   "AUTH_FAILED"
   → Username/password incorrect

   Solution :
   - Vérifier credentials
   - Vérifier certificats (dates, CN, chaîne)

4. Crypto mismatch :
   Client : AES-256-GCM
   Server : AES-128-CBC

   → Incompatible, échec négociation

   Solution : Aligner cipher suites

5. Version incompatible :
   Client OpenVPN 2.6
   Server OpenVPN 2.3

   Généralement compatible, mais :
   → Désactiver features récentes

6. Firewall local :
   # Windows
   Firewall bloque OpenVPN.exe

   Solution :
   Ajouter exception firewall

7. Antivirus :
   Certains AV bloquent VPN

   Test : Désactiver temporairement
   Si fonctionne : Whitelist VPN
```

## VPN commerciaux vs entreprise

### VPN grand public (privacy)

```
Services commerciaux :
- NordVPN
- ExpressVPN
- ProtonVPN
- Mullvad
- Etc.

Objectif : Privacy et contournement geo-restrictions

Caractéristiques :

✓ Milliers de serveurs (pays multiples)
✓ No-logs policy (théorique)
✓ Facile à utiliser (app mobile/desktop)
✓ Abordable (5-10€/mois)

Limitations :

✗ Performance variable
✗ Trust provider (vraiment no-logs ?)
✗ Shared IP (multiples users)
✗ Certains sites bloquent IPs VPN connues

Use cases :
- Contourner censure (Chine, Iran)
- Privacy public Wi-Fi
- Geo-unlock (Netflix, etc.)
- Torrent anonyme

Pas pour entreprise :
✗ Pas de control réseau
✗ Pas d'intégration IT
✗ Sécurité questionnable
```

### VPN entreprise (accès sécurisé)

```
Solutions entreprise :
- Cisco AnyConnect
- Palo Alto GlobalProtect
- Fortinet FortiClient
- Pulse Secure
- CheckPoint
- F5 BIG-IP APM

Objectif : Accès sécurisé réseau corporatif

Caractéristiques :

✓ Intégration AD/LDAP
✓ Policies granulaires
✓ 2FA/MFA obligatoire
✓ Logging audit complet
✓ Support enterprise
✓ Compliance (SOC2, ISO 27001)
✓ Posture checking (antivirus, patch level)
✓ VPN per-app

Coût :
- Licence : 50-200€/user/an
- Hardware : 5K-500K€
- Support : 20% coût annuel

Use case :
✓ Remote access employés
✓ Contractors temporaires
✓ BYOD sécurisé
✓ Site-to-site
```

## Conclusion

Les VPN sont une technologie fondamentale pour la sécurité réseau moderne, permettant des connexions sécurisées sur des réseaux non fiables.

**Points clés à retenir** :

```
Protocoles VPN 2024 :

TOP 3 recommandés :
1. WireGuard
   ✓ Le plus rapide
   ✓ Le plus simple
   ✓ Code minimal (sécurité)
   → Choix moderne par défaut

2. IKEv2/IPsec
   ✓ Excellent mobile (MOBIKE)
   ✓ Natif OS
   ✓ Enterprise-ready
   → Meilleur pour iOS/Android

3. OpenVPN
   ✓ Universel
   ✓ Traverse tout (port 443)
   ✓ Très configurable
   → Fallback fiable

Acceptable :
- IPsec (site-to-site)
- L2TP/IPsec (legacy)

À éviter :
✗ PPTP (CASSÉ, jamais utiliser)
✗ SSTP (propriétaire, limité)

Types VPN :
- Site-to-Site : Réseaux ↔ Réseaux
- Remote Access : Utilisateurs → Réseau
- Mesh : Tous ↔ Tous

Architecture :
✓ Gateway centralisé (standard)
✓ HA (haute disponibilité)
✓ Load balancing si >500 users
✓ Monitoring et logging essentiels

Sécurité :
✓ 2FA/MFA obligatoire
✓ Certificats préférés vs passwords
✓ Logs complets (audit)
✓ Revue accès régulière
✓ Least privilege

Performance :
- WireGuard : 1-2 Gbps/core
- IPsec : 800-1000 Mbps
- OpenVPN : 200-400 Mbps

Dimensionnement :
- Small (<50) : pfSense, EdgeRouter
- Medium (50-500) : Cisco ASA, FortiGate
- Large (500-10K) : Cluster, HA
- Very Large (10K+) : Géodistribué
```

**Règles d'or VPN** :

```
1. Choisir protocole moderne :
   WireGuard ou IKEv2 par défaut

2. Toujours activer MFA :
   Password seul = insuffisant 2024

3. Monitoring et alerting :
   Détecter anomalies rapidement

4. Tester régulièrement :
   Failover, performance, sécurité

5. Documenter :
   Configuration, procédures, troubleshooting

6. Planifier capacité :
   Growth = plus d'utilisateurs

7. Sécuriser gateway :
   Patching, hardening, isolation

8. Ne jamais utiliser PPTP :
   Cassé, dangereux, obsolète
```

Les VPN restent essentiels en 2024 malgré l'émergence de Zero Trust et ZTNA (Zero Trust Network Access). Ils continuent d'être le moyen standard de connecter sites distants et utilisateurs mobiles de manière sécurisée.

Dans la section suivante (6.7), nous étudierons les pare-feu (firewalls), qui travaillent souvent en tandem avec les VPN pour créer des architectures de sécurité réseau complètes et robustes.

⏭️ [Pare-feu : filtrage de paquets, stateful inspection](/06-securite/07-pare-feu.md)
