🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.5 IPsec : sécurité au niveau réseau

## Introduction

**IPsec** (Internet Protocol Security) est une suite de protocoles conçue pour sécuriser les communications au **niveau de la couche réseau** (couche 3 du modèle OSI). Contrairement à TLS qui sécurise des applications spécifiques (HTTPS, SMTPS, etc.), IPsec protège **tout le trafic IP**, indépendamment de l'application utilisée.

IPsec est le fondement de la plupart des **VPN** (Virtual Private Networks) modernes et permet de créer des tunnels sécurisés sur Internet, transformant un réseau public non fiable en un canal de communication privé et protégé.

**Analogie du monde réel** :

```
TLS = Enveloppe recommandée pour une lettre :
- Protection d'un échange spécifique (une lettre)
- L'expéditeur et le destinataire doivent coopérer
- Visible que c'est une lettre recommandée

IPsec = Tunnel blindé entre deux bâtiments :
- Protection de TOUT ce qui passe (personnes, objets, documents)
- Transparent pour les occupants
- De l'extérieur, on ne voit que le tunnel
- Tout ce qui entre d'un côté sort de l'autre, protégé
```

## Pourquoi IPsec existe-t-il ?

### Limitations de la sécurité au niveau application

**TLS et autres protocoles de couche application** résolvent des problèmes spécifiques mais ont des limitations :

```
Approche par application (TLS, SSH, etc.) :

Application 1 (HTTPS) → TLS → Sécurisé
Application 2 (SSH) → SSH → Sécurisé
Application 3 (FTP) → ??? → Non sécurisé
Application 4 (DNS) → ??? → Non sécurisé
Application 5 (Custom) → ??? → Non sécurisé

Problèmes :
✗ Chaque application doit implémenter sa propre sécurité
✗ Applications legacy non sécurisables
✗ Complexité de configuration (un protocole par app)
✗ Pas de protection de bout en bout au niveau réseau
✗ Adresses IP visibles (métadonnées exposées)
```

**Approche IPsec (niveau réseau)** :

```
TOUT le trafic IP → IPsec → Sécurisé

✓ HTTPS → Protégé
✓ SSH → Protégé
✓ FTP → Protégé (même non sécurisé à l'origine)
✓ DNS → Protégé
✓ Custom → Protégé
✓ ICMP → Protégé
✓ Tout protocole IP → Protégé

Avantages :
✓ Transparent pour les applications
✓ Une seule configuration pour tout
✓ Protection de toute la communication
✓ Chiffrement des adresses IP internes (mode tunnel)
```

### Cas d'usage historiques et modernes

**Années 1990-2000 : Naissance d'IPsec**

```
Contexte :
- Internet devient commercial
- Entreprises veulent connecter sites distants
- Lignes dédiées très coûteuses

Solution IPsec :
Bureau A (Paris) ←→ Internet ←→ Bureau B (Lyon)
                   (tunnel IPsec)

Résultat :
- Communication sécurisée sur infrastructure publique
- Coût réduit (pas de lignes dédiées)
- Topologie réseau étendu (WAN)
```

**2000-2010 : VPN d'accès distant**

```
Cas d'usage : Télétravail

Employé à domicile → Internet → Entreprise
                    (VPN IPsec)

Avantages :
- Accès sécurisé aux ressources internes
- Comme si l'employé était au bureau
- Protection sur Wi-Fi public
```

**2010-2024 : Cloud et hybridation**

```
Cas d'usage modernes :

1. Site-to-Site VPN (cloud) :
   Bureau local ←→ AWS/Azure VPC
                  (IPsec tunnel)

2. Hybrid Cloud :
   Datacenter privé ←→ Cloud public
                      (interconnexion IPsec)

3. Multi-cloud :
   AWS ←→ Azure ←→ GCP
        (maillage IPsec)

4. IoT sécurisé :
   Appareils IoT → Gateway → Backend
                  (IPsec)

5. Mobile Security :
   Smartphones → VPN IPsec → Entreprise
```

## Position dans la pile TCP/IP

### Placement architectural

```
Modèle TCP/IP avec IPsec :

┌────────────────────────┐
│   Application          │  HTTP, FTP, DNS, SSH...
├────────────────────────┤
│   Transport (TCP/UDP)  │  Ports, connexions
├────────────────────────┤
│   IPsec ════════════   │  ← Couche de sécurité
├────────────────────────┤
│   Internet (IP)        │  Routage, adressage
├────────────────────────┤
│   Accès réseau         │  Ethernet, Wi-Fi...
└────────────────────────┘

IPsec s'insère entre les couches Transport et Internet
Ou peut encapsuler complètement IP (mode tunnel)
```

### Comparaison avec TLS

```
┌──────────────────┬──────────────────┬──────────────────────┐
│    Critère       │      TLS         │       IPsec          │
├──────────────────┼──────────────────┼──────────────────────┤
│ Couche OSI       │ Session/         │ Réseau (3)           │
│                  │ Présentation     │                      │
│                  │ (5/6)            │                      │
├──────────────────┼──────────────────┼──────────────────────┤
│ Transport        │ TCP uniquement   │ IP (TCP + UDP + tout)│
├──────────────────┼──────────────────┼──────────────────────┤
│ Granularité      │ Par application  │ Par hôte/réseau      │
├──────────────────┼──────────────────┼──────────────────────┤
│ Transparence     │ Application doit │ Transparent pour     │
│                  │ supporter TLS    │ applications         │
├──────────────────┼──────────────────┼──────────────────────┤
│ Configuration    │ Serveur web,     │ OS, routeur,         │
│                  │ bibliothèques    │ firewall             │
├──────────────────┼──────────────────┼──────────────────────┤
│ Authentification │ Certificats X.509│ Certificats ou PSK   │
├──────────────────┼──────────────────┼──────────────────────┤
│ NAT traversal    │ Pas de problème  │ Problématique        │
│                  │ (port 443)       │ (ESP, protocole 50)  │
├──────────────────┼──────────────────┼──────────────────────┤
│ Performance      │ Overhead par     │ Overhead global      │
│                  │ connexion        │ (tout le trafic)     │
├──────────────────┼──────────────────┼──────────────────────┤
│ Cas d'usage      │ Web, email,      │ VPN site-to-site,    │
│ principal        │ APIs             │ accès distant        │
└──────────────────┴──────────────────┴──────────────────────┘
```

### Exemple concret de différence

**Scénario : Employé accède à plusieurs services internes**

**Avec TLS** :

```
Employé distant :

1. HTTPS vers intranet.company.com
   → Connexion TLS sur port 443
   → Sécurisé ✓

2. SSH vers serveur.company.com
   → Connexion SSH sur port 22
   → Sécurisé ✓

3. RDP vers desktop.company.com
   → RDP peut utiliser TLS
   → Sécurisé ✓

4. SMB file share vers files.company.com
   → SMB over TLS complexe
   → Souvent non sécurisé ✗

5. DNS interne
   → DNS standard non chiffré
   → Non sécurisé ✗

6. Application legacy sur port 8080
   → Pas de support TLS
   → Non sécurisé ✗

Configuration :
- 3+ protocoles différents à configurer
- Applications doivent supporter TLS
- Complexe, incomplet
```

**Avec IPsec VPN** :

```
Employé distant :

1. Établit tunnel IPsec vers gateway VPN entreprise
   → Tout le trafic passe par le tunnel
   → Une seule authentification

2. HTTPS, SSH, RDP, SMB, DNS, legacy app...
   → TOUT protégé automatiquement
   → Transparent pour applications

Configuration :
- Une seule connexion VPN
- Simple pour l'utilisateur
- Complet, cohérent
```

## Architecture d'IPsec

IPsec n'est pas un protocole unique mais une **suite de protocoles** et de mécanismes travaillant ensemble.

### Les trois composants principaux

```
┌──────────────────────────────────────────────────┐
│                   IPsec Suite                    │
├─────────────────┬───────────────┬────────────────┤
│       AH        │      ESP      │      IKE       │
│  Authentication │  Encapsulating│  Internet Key  │
│     Header      │    Security   │   Exchange     │
│                 │    Payload    │                │
│  Protocole 51   │ Protocole 50  │   UDP 500      │
├─────────────────┼───────────────┼────────────────┤
│ • Intégrité     │ • Chiffrement │ • Négociation  │
│ • Authentif.    │ • Intégrité   │ • Auth mutuelle│
│ • Anti-replay   │ • Authentif.  │ • Échange clés │
│                 │ • Anti-replay │ • Gestion SA   │
└─────────────────┴───────────────┴────────────────┘
```

### IKE (Internet Key Exchange)

**Rôle** : Négocier et établir les associations de sécurité (SA).

```
IKE = "Le protocole qui configure IPsec"

Fonctions :
1. Authentification mutuelle
   - Qui êtes-vous ? (certificats, PSK, EAP)

2. Négociation des paramètres
   - Algorithmes de chiffrement
   - Algorithmes d'intégrité
   - Groupes Diffie-Hellman

3. Génération de clés
   - Clés de chiffrement
   - Clés d'authentification

4. Gestion des SA
   - Création
   - Renouvellement
   - Suppression

Versions :
- IKEv1 (RFC 2409, 1998) : Complexe, deux phases
- IKEv2 (RFC 7296, 2014) : Simplifié, plus rapide, standard actuel
```

**Phases IKEv2** :

```
Phase 1 : IKE_SA_INIT (2 messages)
Initiator                              Responder
    |                                      |
    | IKE_SA_INIT Request                  |
    |  - Propositions crypto               |
    |  - Nonce, DH key exchange            |
    |------------------------------------->|
    |                                      |
    |          IKE_SA_INIT Response        |
    |  - Crypto choisi                     |
    |  - Nonce, DH key exchange            |
    |<-------------------------------------|

Résultat : Clés pour chiffrer la suite

Phase 2 : IKE_AUTH (2 messages, chiffrés)
    |                                      |
    | IKE_AUTH Request                     |
    |  - Identité (certificat/PSK)         |
    |  - Authentification                  |
    |  - Propositions IPsec SA             |
    |  - Traffic Selectors                 |
    |------------------------------------->|
    |                                      |
    |          IKE_AUTH Response           |
    |  - Identité                          |
    |  - Authentification                  |
    |  - IPsec SA acceptée                 |
    |<-------------------------------------|

Résultat :
- IKE SA établie (pour gestion)
- IPsec SA établie (pour trafic)
- Clés dérivées
- Authentification mutuelle validée

Total : 4 messages (2 RTT)
IKEv1 nécessitait 6+ messages
```

**Security Association (SA)** :

```
SA = "Contrat de sécurité" entre deux parties

Contient :
- SPI (Security Parameter Index) : Identifiant unique
- Algorithmes de chiffrement (AES-256, etc.)
- Algorithmes d'intégrité (HMAC-SHA256, etc.)
- Clés cryptographiques
- Durée de vie (temps ou volume de données)
- Mode (transport ou tunnel)
- Sélecteurs de trafic

Exemple SA :
SPI: 0x12345678
Mode: Tunnel
Encryption: AES-256-GCM
Integrity: (intégré à GCM)
Lifetime: 3600 secondes ou 100 MB
Src: 10.0.0.0/24
Dst: 192.168.1.0/24

Unidirectionnel :
- Une SA pour A → B
- Une SA pour B → A
- Donc 2 SA pour communication bidirectionnelle
```

### ESP (Encapsulating Security Payload)

**Rôle** : Fournir confidentialité, intégrité et authentification.

```
ESP = "Le protocole qui chiffre et protège les données"

Caractéristiques :
- Protocole IP numéro 50
- Chiffrement du payload
- Intégrité cryptographique
- Anti-replay protection
- Optionnel : authentification origine

Format paquet ESP :
┌──────────────────────────────────────┐
│    Original IP Header                │
├──────────────────────────────────────┤
│    ESP Header                        │
│    - SPI (4 bytes)                   │
│    - Sequence Number (4 bytes)       │
├──────────────────────────────────────┤ ─┐
│    Payload (chiffré)                 │  │
│    - Original IP packet ou           │  │
│      Transport segment               │  │ Chiffré
│    - Padding                         │  │
│    - Pad Length + Next Header        │  │
├──────────────────────────────────────┤ ─┘
│    ESP Trailer (chiffré)             │
├──────────────────────────────────────┤ ─┐
│    ICV (Integrity Check Value)       │  │ Authentifié
│    - HMAC ou AEAD tag                │  │
└──────────────────────────────────────┘ ─┘

Fonctionnalités :
✓ Confidentialité : Chiffrement du payload
✓ Intégrité : ICV vérifie non-modification
✓ Authentification : ICV prouve origine
✓ Anti-replay : Sequence number
```

**Algorithmes ESP courants** :

```
Chiffrement (2024) :

Recommandé :
✓ AES-GCM-128/256 (AEAD - chiffrement + authentif intégrée)
✓ AES-CBC-128/256 + HMAC-SHA256 (séparé)
✓ ChaCha20-Poly1305 (AEAD moderne)

Déprécié :
✗ 3DES (lent, clé courte)
✗ DES (cassé)
✗ NULL (pas de chiffrement, rare)

Intégrité (si pas AEAD) :
✓ HMAC-SHA256
✓ HMAC-SHA384
✓ HMAC-SHA512

Déprécié :
✗ HMAC-MD5 (MD5 cassé)
✗ HMAC-SHA1 (SHA1 faible)

Préférence 2024 :
AES-256-GCM ou ChaCha20-Poly1305
→ Chiffrement + authentification en un seul algorithme
→ Performance excellente avec accélération matérielle
```

### AH (Authentication Header)

**Rôle** : Fournir intégrité et authentification **sans** chiffrement.

```
AH = "Le protocole qui authentifie mais ne chiffre PAS"

Caractéristiques :
- Protocole IP numéro 51
- Intégrité de tout le paquet (header inclus)
- Authentification de l'origine
- Anti-replay
- MAIS : Pas de confidentialité

Format paquet AH :
┌──────────────────────────────────────┐
│    Original IP Header (modifié)      │
│    - Certains champs inclus dans ICV │
├──────────────────────────────────────┤
│    AH Header                         │
│    - Next Header (1 byte)            │
│    - Payload Length (1 byte)         │
│    - Reserved (2 bytes)              │
│    - SPI (4 bytes)                   │
│    - Sequence Number (4 bytes)       │
│    - ICV (Integrity Check Value)     │
│      (longueur variable, ex: 12-96b) │
├──────────────────────────────────────┤
│    Original Payload                  │
│    (NON chiffré)                     │
└──────────────────────────────────────┘

ICV couvre :
- IP header (champs immuables)
- AH header
- Payload complet

Résultat :
✓ Garantit que paquet n'a pas été modifié
✓ Garantit l'origine (avec clé partagée/certificat)
✗ Payload en clair (visible)
```

**Pourquoi AH existe-t-il ?**

```
Raisons historiques et spécifiques :

1. Régulations d'export (années 1990) :
   - Chiffrement fort interdit à l'export (USA)
   - Authentification autorisée
   - AH permettait IPsec sans chiffrement

2. Performance (obsolète aujourd'hui) :
   - Chiffrement coûteux (avant AES-NI)
   - AH plus rapide
   - Aujourd'hui : différence négligeable

3. Intégrité de l'en-tête IP :
   - AH protège certains champs IP header
   - ESP ne protège que le payload
   - Utile pour certaines configurations

4. Transparence pour certains équipements :
   - Header IP visible (inspection, QoS)
   - Mais peu pratique aujourd'hui

Statut actuel (2024) :
✗ Rarement utilisé en pratique
✓ ESP préféré (fait tout ce que AH fait + chiffrement)
✗ Problèmes avec NAT (modifie IP header)
✗ Complexité additionnelle sans grand bénéfice

Recommandation : Utiliser ESP, pas AH
```

### Comparaison ESP vs AH

```
┌─────────────────────┬──────────────┬──────────────┐
│     Fonctionnalité  │     ESP      │      AH      │
├─────────────────────┼──────────────┼──────────────┤
│ Confidentialité     │      ✓       │      ✗       │
│ (chiffrement)       │              │              │
├─────────────────────┼──────────────┼──────────────┤
│ Intégrité payload   │      ✓       │      ✓       │
├─────────────────────┼──────────────┼──────────────┤
│ Intégrité IP header │      ✗       │      ✓       │
├─────────────────────┼──────────────┼──────────────┤
│ Authentification    │      ✓       │      ✓       │
├─────────────────────┼──────────────┼──────────────┤
│ Anti-replay         │      ✓       │      ✓       │
├─────────────────────┼──────────────┼──────────────┤
│ Traversée NAT       │ Possible     │ Difficile    │
│                     │ (NAT-T)      │              │
├─────────────────────┼──────────────┼──────────────┤
│ Protocole IP        │      50      │      51      │
├─────────────────────┼──────────────┼──────────────┤
│ Usage moderne       │   Standard   │     Rare     │
└─────────────────────┴──────────────┴──────────────┘

Combinaison ESP + AH :
Théoriquement possible mais :
✗ Complexité double
✗ Overhead important
✗ Peu de gain pratique
✗ Jamais utilisé en pratique

Conclusion : ESP seul suffit pour 99.9% des cas
```

## Processus de communication IPsec

### Vue d'ensemble du flux

```
Établissement et utilisation d'une connexion IPsec :

┌────────────────────────────────────────────────────┐
│  Phase 1 : Négociation IKE (une fois)              │
├────────────────────────────────────────────────────┤
│  1. IKE_SA_INIT                                    │
│     - Échange Diffie-Hellman                       │
│     - Négociation algorithmes                      │
│     - Génération clés IKE                          │
│                                                    │
│  2. IKE_AUTH                                       │
│     - Authentification mutuelle                    │
│     - Création IPsec SA                            │
│     - Génération clés ESP/AH                       │
│                                                    │
│  Résultat : Tunnel IPsec établi                    │
└────────────────────────────────────────────────────┘
                       ↓
┌────────────────────────────────────────────────────┐
│  Phase 2 : Transfert de données (continu)          │
├────────────────────────────────────────────────────┤
│  Pour chaque paquet :                              │
│  1. Sélection SA (basée sur src/dst/protocole)     │
│  2. Traitement ESP/AH :                            │
│     - Chiffrement payload (ESP)                    │
│     - Calcul ICV                                   │
│     - Ajout header ESP/AH                          │
│  3. Transmission sur réseau                        │
│                                                    │
│  Réception :                                       │
│  1. Identification SA (via SPI)                    │
│  2. Vérification sequence number (anti-replay)     │
│  3. Vérification ICV (intégrité)                   │
│  4. Déchiffrement payload                          │
│  5. Remise à couche supérieure                     │
└────────────────────────────────────────────────────┘
                       ↓
┌────────────────────────────────────────────────────┐
│  Phase 3 : Maintenance (périodique)                │
├────────────────────────────────────────────────────┤
│  - Renouvellement SA (avant expiration)            │
│  - Rekeying (nouvelles clés)                       │
│  - Keep-alive (Dead Peer Detection - DPD)          │
│  - Gestion erreurs                                 │
└────────────────────────────────────────────────────┘
```

### Exemple détaillé : Établissement d'un tunnel

**Scénario** : Bureau A (Paris) se connecte à Bureau B (Lyon) via Internet

```
Configuration :

Bureau A (Paris) :
- Réseau interne : 10.1.0.0/24
- Gateway public : 203.0.113.10
- IPsec peer : 198.51.100.20

Bureau B (Lyon) :
- Réseau interne : 10.2.0.0/24
- Gateway public : 198.51.100.20
- IPsec peer : 203.0.113.10

Authentification : Pre-Shared Key (PSK)
Algorithmes : AES-256-GCM, DH Group 14
```

**Étape 1 : IKE_SA_INIT**

```
Paris (Initiator) → Lyon (Responder)

Message 1 : IKE_SA_INIT Request
Source: 203.0.113.10:500
Dest: 198.51.100.20:500

Contenu :
- Security Association Payload :
    Proposal 1: IKE
      - Encryption: AES-256-CBC
      - Integrity: HMAC-SHA256
      - PRF: HMAC-SHA256
      - DH Group: 14 (2048-bit MODP)

- Key Exchange Payload :
    - DH public value (256 bytes)

- Nonce Payload :
    - Random nonce (32 bytes)

Lyon → Paris

Message 2 : IKE_SA_INIT Response
Source: 198.51.100.20:500
Dest: 203.0.113.10:500

Contenu :
- SA Payload : Accepted proposal
- Key Exchange : DH public value
- Nonce : Random nonce

Résultat après Message 1 + 2 :
✓ Algorithmes négociés
✓ DH exchange complet
✓ SKEYSEED dérivé (clé maîtresse)
✓ Clés pour chiffrer IKE_AUTH :
  - SK_ei (initiator encryption)
  - SK_er (responder encryption)
  - SK_ai (initiator integrity)
  - SK_ar (responder integrity)
```

**Étape 2 : IKE_AUTH**

```
Paris → Lyon

Message 3 : IKE_AUTH Request (CHIFFRÉ avec SK_ei)
Source: 203.0.113.10:500
Dest: 198.51.100.20:500

Contenu (après déchiffrement) :
- Identification Payload :
    ID Type: IPv4 address
    ID Data: 203.0.113.10

- Authentication Payload :
    Auth Method: Shared Key Message Integrity Code
    Auth Data: HMAC-SHA256(PSK, messages précédents)

- SA Payload (IPsec) :
    Proposal 1: ESP
      - Encryption: AES-256-GCM
      - ESN: Extended Sequence Numbers

- Traffic Selector - Initiator :
    - Start Address: 10.1.0.0
    - End Address: 10.1.0.255
    - Protocol: Any
    - Port: Any

- Traffic Selector - Responder :
    - Start Address: 10.2.0.0
    - End Address: 10.2.0.255
    - Protocol: Any
    - Port: Any

Lyon → Paris

Message 4 : IKE_AUTH Response (CHIFFRÉ avec SK_er)

Contenu (après déchiffrement) :
- Identification Payload : 198.51.100.20
- Authentication Payload : HMAC avec PSK
- SA Payload : Accepted ESP proposal
- Traffic Selectors : Confirmed

Résultat après Message 3 + 4 :
✓ Authentification mutuelle validée
✓ IKE SA établie (pour contrôle)
✓ IPsec SA établie (pour données)
✓ Traffic selectors définis (10.1.0.0/24 ↔ 10.2.0.0/24)
✓ Clés ESP dérivées
```

**Étape 3 : Transfert de données**

```
PC dans Bureau A (10.1.0.5) ping PC dans Bureau B (10.2.0.10)

1. PC Paris génère paquet :
   ┌────────────────────────────────────┐
   │ IP: 10.1.0.5 → 10.2.0.10           │
   │ ICMP Echo Request                  │
   └────────────────────────────────────┘

2. Gateway Paris intercepte (routing) :
   - Destination 10.2.0.10 matche Traffic Selector
   - Applique IPsec SA

3. Encapsulation ESP (mode tunnel) :
   ┌────────────────────────────────────────────────┐
   │ New IP: 203.0.113.10 → 198.51.100.20           │
   ├────────────────────────────────────────────────┤
   │ ESP Header:                                    │
   │   SPI: 0xABCD1234                              │
   │   Sequence: 1                                  │
   ├────────────────────────────────────────────────┤
   │ Encrypted payload:                             │
   │   ┌────────────────────────────────────────┐   │
   │   │ Original IP: 10.1.0.5 → 10.2.0.10      │   │
   │   │ ICMP Echo Request                      │   │
   │   │ Padding                                │   │
   │   └────────────────────────────────────────┘   │
   ├────────────────────────────────────────────────┤
   │ ICV (GCM tag): 16 bytes                        │
   └────────────────────────────────────────────────┘

4. Transmission via Internet :
   Paris Gateway → Internet → Lyon Gateway

5. Gateway Lyon reçoit :
   - Identifie SA via SPI 0xABCD1234
   - Vérifie sequence number (anti-replay)
   - Déchiffre avec clés ESP
   - Vérifie ICV (intégrité)
   - Extrait paquet original

6. Routage vers destination :
   Gateway Lyon → 10.2.0.10

7. PC Lyon répond :
   ICMP Echo Reply : 10.2.0.10 → 10.1.0.5

8. Processus inverse (Lyon → Paris via ESP)

Résultat :
✓ Communication transparente entre réseaux
✓ Chiffrement complet sur Internet
✓ Adresses internes invisibles de l'extérieur
```

## Authentification dans IPsec

### Méthodes d'authentification

```
IPsec supporte plusieurs méthodes :

1. Pre-Shared Key (PSK)
2. Certificats X.509 (PKI)
3. EAP (Extensible Authentication Protocol)
```

#### Pre-Shared Key (PSK)

```
Principe : Clé secrète partagée à l'avance

Configuration :
Site A : psk = "SuperSecretKey123!@#"
Site B : psk = "SuperSecretKey123!@#"

Avantages :
+ Simple à configurer
+ Pas besoin d'infrastructure PKI
+ Bon pour site-to-site limité

Inconvénients :
- Scalabilité limitée
  (difficile avec 100+ sites : N*(N-1)/2 clés)
- Distribution sécurisée de la clé délicate
- Pas de granularité (même clé = même accès)
- Si compromise : tout reconfigurer

Usage recommandé :
✓ Petits déploiements (2-10 sites)
✓ Labs et tests
✓ Équipements limités (IoT)

Déconseillé :
✗ Grands déploiements
✗ Accès utilisateur distant (VPN road warrior)
```

**Génération de PSK sécurisée** :

```bash
# Générer PSK forte (32 bytes = 256 bits)
openssl rand -base64 32
# Exemple : 7K+Jm9fX2pL4nR8vW3zT6qY1sA5hC0dE2gB8jN4kM7o=

# Ou avec /dev/urandom
head -c 32 /dev/urandom | base64
```

#### Certificats X.509

```
Principe : PKI comme pour TLS

Infrastructure requise :
1. CA (Certification Authority)
2. Certificats pour chaque gateway/utilisateur
3. Mécanisme de révocation (CRL/OCSP)

Avantages :
+ Scalable (milliers de sites/utilisateurs)
+ Révocation granulaire
+ Identité forte et vérifiable
+ Standard bien établi

Inconvénients :
- Complexité PKI
- Gestion certificats (émission, renouvellement, révocation)
- Infrastructure requise

Usage recommandé :
✓ Grands déploiements
✓ Entreprises avec PKI existante
✓ VPN accès distant (road warrior)
✓ Exigences de conformité

Workflow :
1. Chaque site obtient certificat de la CA
2. Lors du IKE_AUTH :
   - Envoie son certificat
   - Vérifie certificat du peer
   - Valide chaîne de certification
   - Vérifie révocation (optionnel)
3. Si valide → tunnel établi
```

**Configuration avec certificats** :

```
Site A (Paris) :
- Certificat : CN=paris-gw.company.com
- Clé privée : paris-gw.key
- CA cert : company-ca.crt

Site B (Lyon) :
- Certificat : CN=lyon-gw.company.com
- Clé privée : lyon-gw.key
- CA cert : company-ca.crt

IKE_AUTH :
Paris → Lyon : Envoie cert paris-gw.company.com
Lyon vérifie :
  ✓ Signature CA valide
  ✓ Certificat non expiré
  ✓ CN matche peer configuré
  ✓ Pas révoqué

Lyon → Paris : Envoie cert lyon-gw.company.com
Paris vérifie : (idem)

Si validations OK : Tunnel établi
```

#### EAP (Extensible Authentication Protocol)

```
Principe : Délégation authentification à serveur externe

Architecture :
Client IPsec ← → Gateway IPsec ← → RADIUS/AAA Server

Méthodes EAP courantes :
- EAP-TLS : Certificats client
- EAP-TTLS : Username/password dans tunnel TLS
- EAP-MSCHAPv2 : Microsoft (avec PEAP)
- EAP-SIM : Authentification SIM (mobile)

Avantages :
+ Intégration annuaire (AD, LDAP)
+ Authentification centralisée
+ Support multi-facteur (OTP, etc.)
+ Audit centralisé

Inconvénients :
- Complexité accrue
- Dépendance serveur AAA
- Latence additionnelle

Usage :
✓ VPN accès distant entreprise
✓ Intégration Active Directory
✓ Authentification utilisateur (pas machine)

Exemple :
Employé distant avec VPN :
1. Connexion VPN
2. Gateway demande authentification
3. EAP-TTLS vers RADIUS
4. RADIUS vérifie AD
5. Si OK : tunnel IPsec établi
```

## NAT Traversal (NAT-T)

### Problème de NAT avec IPsec

```
Problématique :

IPsec ESP (protocole 50) et AH (protocole 51) :
- Ne sont PAS TCP/UDP
- Pas de notion de "port"
- NAT ne sait pas comment traduire

Scénario problématique :

Client (10.0.0.5) ← → NAT (203.0.113.50) ← → Internet ← → VPN Gateway

1. Client génère paquet ESP :
   Src IP: 10.0.0.5
   Dst IP: VPN Gateway
   Protocol: ESP (50)
   SPI: 0x12345678

2. NAT traduit :
   Src IP: 10.0.0.5 → 203.0.113.50
   Dst IP: VPN Gateway
   Protocol: ESP

3. Gateway VPN répond :
   Src IP: VPN Gateway
   Dst IP: 203.0.113.50
   Protocol: ESP
   SPI: 0xABCDEF00

4. NAT reçoit paquet ESP :
   ✗ Pas de port pour identifier connexion
   ✗ Peut-être plusieurs clients derrière NAT
   ✗ NAT ne sait pas vers quel client router
   → Paquet DROP

Résultat : IPsec cassé par NAT
```

### Solution : NAT-T (RFC 3947/3948)

```
NAT Traversal = Encapsuler ESP dans UDP

Architecture :
Client ← → NAT ← → Internet ← → VPN Gateway
       UDP 4500       UDP 4500

Processus :

1. Détection NAT lors de IKE :
   - Client et Gateway envoient hash de leurs IP/ports
   - Si hash diffère à réception → NAT détecté

2. Activation NAT-T :
   - Basculement vers UDP port 4500
   - Encapsulation ESP dans UDP

3. Format paquet NAT-T :
   ┌────────────────────────────────────┐
   │ IP Header                          │
   │ Src: 203.0.113.50                  │ ← IP publique NAT
   │ Dst: VPN Gateway                   │
   ├────────────────────────────────────┤
   │ UDP Header                         │
   │ Src Port: 4500                     │
   │ Dst Port: 4500                     │
   ├────────────────────────────────────┤
   │ Non-ESP Marker (0x00000000)        │ ← 4 bytes de 0
   ├────────────────────────────────────┤
   │ ESP Packet                         │
   │ (comme avant, mais encapsulé)      │
   └────────────────────────────────────┘

4. NAT traduit :
   - Voit UDP port 4500
   - Crée mapping normal UDP
   - Fonctionne comme n'importe quel UDP

5. Gateway VPN :
   - Reçoit sur UDP 4500
   - Détecte Non-ESP Marker
   - Extrait ESP
   - Traite normalement

Résultat : IPsec fonctionne à travers NAT !
```

**Keep-alive NAT-T** :

```
Problème : Mapping NAT expire si inactif

Solution : Keep-alive packets

Tous les 20-30 secondes :
Client → NAT → Gateway
  UDP 4500, payload = 0xFF (1 byte)

Gateway → NAT → Client
  UDP 4500, payload = 0xFF

Maintient le mapping NAT actif
→ Tunnel reste accessible
```

## Avantages et inconvénients d'IPsec

### Avantages

```
✓ Transparent pour applications :
  - Pas besoin de modifier les applications
  - Applications legacy sécurisées automatiquement
  - Un seul point de configuration

✓ Protection complète :
  - Tout le trafic IP protégé
  - Pas de zone grise
  - Chiffrement de bout en bout au niveau réseau

✓ Standard ouvert :
  - RFC bien définis
  - Interopérabilité entre vendors
  - Support universel (OS, routeurs, firewalls)

✓ Performance :
  - Accélération matérielle disponible
  - Overhead raisonnable
  - Efficace pour gros volumes

✓ Flexibilité :
  - Multiples modes (transport, tunnel)
  - Multiples algorithmes
  - Multiples méthodes d'authentification

✓ Sécurité réseau :
  - Cache les adresses internes (mode tunnel)
  - Anti-replay natif
  - Perfect Forward Secrecy avec ECDHE
```

### Inconvénients

```
✗ Complexité de configuration :
  - Beaucoup de paramètres
  - Courbe d'apprentissage raide
  - Debug difficile

✗ Problèmes NAT :
  - NAT-T requis
  - Pas toujours supporté
  - Complexité additionnelle

✗ Overhead :
  - Headers additionnels (ESP/AH)
  - Fragmentation possible si MTU mal configuré
  - Latence IKE initiale

✗ Visibilité limitée :
  - Inspection DPI impossible (chiffré)
  - QoS complexe
  - Troubleshooting réseau difficile

✗ Scalabilité gestion :
  - Configuration manuelle souvent
  - Gestion certificats (si PKI)
  - Coordination entre pairs

✗ Incompatibilité avec certains protocoles :
  - Multicast complexe
  - Certaines applications peer-to-peer
  - SIP/VoIP nécessite ALG

✗ Point de défaillance unique :
  - Gateway IPsec critique
  - Si down → tout le trafic coupé
  - Nécessite haute disponibilité
```

## Cas d'usage typiques

### 1. Site-to-Site VPN

```
Scénario : Connecter bureaux distants

Bureau Paris          Internet          Bureau Lyon
10.1.0.0/24    ←──────────────────→    10.2.0.0/24
[Gateway IPsec]                      [Gateway IPsec]

Configuration :
- Mode : Tunnel
- Auth : PSK ou certificats
- Traffic : All to all (10.1.0.0/24 ↔ 10.2.0.0/24)

Avantages :
✓ Transparent pour utilisateurs
✓ Tous les services accessibles
✓ Sécurité complète
✓ Remplace lignes dédiées coûteuses

Typique pour :
- PME multi-sites
- Agences bancaires
- Franchises retail
```

### 2. Remote Access VPN (Road Warrior)

```
Scénario : Employés en télétravail

Employé distant       Internet       Entreprise
(Laptop, mobile)  ←─────────────→  [VPN Gateway]
                                        │
                                   [Réseau interne]

Configuration :
- Mode : Tunnel
- Auth : Certificats + EAP/RADIUS
- Traffic : Client subnet dynamique ↔ réseau entreprise

Client obtient :
- IP virtuelle (ex: 172.16.100.50)
- Routes vers ressources internes
- DNS interne

Avantages :
✓ Accès sécurisé de partout
✓ Protection Wi-Fi public
✓ Comme si au bureau

Typique pour :
- Télétravail
- Nomades
- Techniciens terrain
```

### 3. Cloud Interconnection

```
Scénario : Connecter datacenter privé à cloud public

Datacenter local     Internet      AWS VPC
192.168.0.0/16   ←────────────→  10.0.0.0/16
[Firewall IPsec]                [AWS VPN Gateway]

Configuration :
- Mode : Tunnel
- Auth : PSK (AWS fournit)
- Redondance : Dual tunnel
- BGP : Pour routage dynamique

Avantages :
✓ Extension réseau dans cloud
✓ Hybrid cloud sécurisé
✓ Pas besoin Direct Connect (coûteux)

Typique pour :
- Migration cloud progressive
- Disaster recovery
- Burst capacity vers cloud
```

### 4. IoT Security

```
Scénario : Sécuriser communication IoT

Capteurs IoT         Internet       Backend
[Devices]        ←─────────────→  [Gateway]
(limités)                           │
                                [Serveurs]

Configuration :
- Mode : Tunnel
- Auth : PSK (pre-provisioned) ou certificats
- Lightweight : ChaCha20 si pas AES-NI

Avantages :
✓ Chiffrement end-to-end
✓ Authentification devices
✓ Protection contre spoofing

Défis :
- Ressources limitées devices
- Gestion clés à l'échelle
- Batterie (overhead crypto)
```

## Alternatives et complémentarité

### IPsec vs autres solutions VPN

```
┌──────────────┬────────────┬─────────────┬──────────────┐
│  Solution    │  Couche    │  Complexité │  Performance │
├──────────────┼────────────┼─────────────┼──────────────┤
│ IPsec        │ Réseau (3) │   Élevée    │   Excellente │
├──────────────┼────────────┼─────────────┼──────────────┤
│ OpenVPN      │ App/TLS    │   Moyenne   │   Bonne      │
│              │ sur TCP/UDP│             │              │
├──────────────┼────────────┼─────────────┼──────────────┤
│ WireGuard    │ Réseau (3) │   Faible    │   Excellente │
│              │ (moderne)  │             │   (meilleure)│
├──────────────┼────────────┼─────────────┼──────────────┤
│ SSL VPN      │ App (HTTPS)│   Faible    │   Moyenne    │
│              │            │             │              │
├──────────────┼────────────┼─────────────┼──────────────┤
│ GRE+IPsec    │ Tunnel+Sec │   Élevée    │   Bonne      │
│              │            │             │   (overhead) │
└──────────────┴────────────┴─────────────┴──────────────┘

Choix selon contexte :

IPsec :
✓ Standard entreprise
✓ Interopérabilité requise
✓ Site-to-site
✓ Support natif OS/équipements

OpenVPN :
✓ Flexibilité maximale
✓ Traverse tout (port 443)
✓ Open source
✓ Communauté large

WireGuard :
✓ Moderne, simple
✓ Performance maximale
✓ Code minimal (auditabilité)
✓ Mobile-friendly
✗ Moins mature (mais adopté rapidement)

SSL VPN :
✓ Accès web uniquement
✓ Sans client (browser)
✓ Très simple utilisateur
✗ Limité aux applications web
```

### IPsec + TLS : Complémentarité

```
Defense in depth :

Internet
    │
    ├─ IPsec Tunnel ────────────────┐
    │                               │
    │  ┌─────────────────────────┐  │
    │  │  TLS (HTTPS)            │  │
    │  │    │                    │  │
    │  │    └─ Application Data  │  │
    │  └─────────────────────────┘  │
    │                               │
    └───────────────────────────────┘

Double chiffrement :
- IPsec : Protection réseau (IP ↔ IP)
- TLS : Protection application (App ↔ App)

Avantages :
✓ Défense en profondeur
✓ Si IPsec compromis → TLS protège encore
✓ Si TLS compromis → IPsec protège encore

Inconvénients :
- Overhead double
- Complexité
- Souvent overkill

Usage :
→ Données extrêmement sensibles
→ Exigences réglementaires strictes
→ Zéro confiance (Zero Trust)
```

## Implémentations IPsec

### Solutions logicielles

```
Linux :
- strongSwan : Implémentation complète, moderne
- Libreswan : Fork de Openswan
- Racoon (ipsec-tools) : Ancien, moins maintenu

Windows :
- Built-in IPsec : Intégré Windows
- Configuration : Firewall + IPsec policies

macOS :
- Built-in IPsec : Intégré système
- Racoon : Backend IKE

BSD :
- IPsec natif kernel
- iked : IKE daemon OpenBSD
```

### Solutions matérielles

```
Firewalls/Appliances :
- Cisco ASA : Standard entreprise
- Palo Alto : Next-gen firewall
- Fortinet FortiGate : UTM + VPN
- Juniper SRX : Enterprise routing + security
- pfSense/OPNsense : Open source

Cloud :
- AWS VPN Gateway : Managed IPsec
- Azure VPN Gateway : Managed IPsec
- GCP Cloud VPN : Managed IPsec

Avantages hardware :
✓ Accélération cryptographique
✓ Haute disponibilité
✓ Throughput élevé (multi-Gbps)
✓ Support 24/7
```

## Performance et optimisations

### Considérations de performance

```
Facteurs impactant performance IPsec :

1. Overhead encapsulation :
   ESP header : ~50-60 bytes
   Impact : ~3-5% sur paquets 1500 bytes
           ~30% sur paquets 200 bytes (VoIP)

2. Chiffrement/déchiffrement :
   Sans accélération : 100-500 Mbps
   Avec AES-NI : 1-10 Gbps
   Hardware dédiée : 10-100 Gbps

3. Fragmentation :
   MTU Internet : 1500 bytes
   MTU après ESP : ~1400 bytes
   Si paquet > MTU : fragmentation IP
   → Performance dégradée

4. CPU :
   IKE négociation : pic CPU initial
   ESP processing : CPU constant
   → Dimensionner selon charge

Optimisations :

✓ Accélération matérielle (AES-NI, QAT)
✓ Path MTU Discovery activé
✓ MSS clamping (TCP)
✓ Offload crypto vers NIC
✓ Multiples tunnels (load balancing)
```

### Benchmarks typiques

```
strongSwan sur serveur moderne (Xeon, AES-NI) :

AES-128-GCM :
- Single core : 2-4 Gbps
- Multi-core : 10-20 Gbps

AES-256-GCM :
- Single core : 1.5-3 Gbps
- Multi-core : 8-15 Gbps

ChaCha20-Poly1305 (sans AES-NI) :
- Single core : 1-2 Gbps
- Multi-core : 5-10 Gbps

Hardware appliance (Cisco, Palo Alto) :
- Entry : 1-5 Gbps
- Mid-range : 10-50 Gbps
- High-end : 100+ Gbps

Cloud managed VPN :
- AWS VPN Gateway : 1.25 Gbps par tunnel
- Azure VPN Gateway : 1-10 Gbps selon SKU
- GCP Cloud VPN : 3 Gbps par tunnel
```

## Conclusion

IPsec est une technologie fondamentale pour la sécurité réseau, offrant une protection complète au niveau IP. Sa position dans la pile réseau lui confère des avantages uniques mais aussi des complexités spécifiques.

**Points clés à retenir** :

```
Architecture IPsec :
✓ Suite de protocoles (pas un seul)
✓ IKE : Négociation et gestion
✓ ESP : Chiffrement + intégrité (standard)
✓ AH : Intégrité seule (rare)

Composants :
✓ Security Association (SA) : "Contrat" de sécurité
✓ SPI : Identifiant de SA
✓ Algorithmes : AES-GCM recommandé (2024)

Authentification :
✓ PSK : Simple, petits déploiements
✓ Certificats : Scalable, entreprise
✓ EAP : Intégration annuaire

NAT Traversal :
✓ NAT-T encapsule ESP dans UDP
✓ Port 4500
✓ Keep-alive pour maintenir mapping

Modes (détaillés dans section suivante) :
- Transport : Host-to-host
- Tunnel : Network-to-network (VPN)

Cas d'usage :
✓ Site-to-Site VPN
✓ Remote Access VPN
✓ Cloud interconnection
✓ IoT security

Avantages principaux :
✓ Transparent applications
✓ Protection complète trafic
✓ Standard interopérable
✓ Performance excellente (hardware)

Défis principaux :
✗ Complexité configuration
✗ NAT problématique (NAT-T requis)
✗ Debug difficile
✗ Gestion scalabilité
```

**État en 2024** :

```
IPsec reste le standard pour :
- VPN site-to-site entreprise
- Cloud interconnection (AWS, Azure, GCP)
- Infrastructures critiques

Concurrence émergente :
- WireGuard : Plus simple, plus rapide, adoption croissante
- ZTNA : Zero Trust Network Access, approche différente

Tendance :
→ IPsec continue pour site-to-site
→ WireGuard gagne pour remote access
→ SD-WAN intègre IPsec mais abstrait complexité
→ Cloud providers standardisent sur IPsec
```

Dans les sections suivantes, nous approfondirons :
- **6.5.1** : Modes transport et tunnel (différences, usages, encapsulation)
- **6.5.2** : AH et ESP (détails protocoles, formats, algorithmes)

Ces détails techniques sont essentiels pour comprendre comment IPsec protège réellement les communications et comment configurer correctement des tunnels VPN sécurisés.

⏭️ [Modes transport et tunnel](/06-securite/05.1-ipsec-modes.md)
