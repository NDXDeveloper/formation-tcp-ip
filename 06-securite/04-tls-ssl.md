🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.4 TLS/SSL : chiffrement des communications

## Introduction

**TLS** (Transport Layer Security) et son prédécesseur **SSL** (Secure Sockets Layer) sont les protocoles cryptographiques qui sécurisent la majorité des communications sur Internet aujourd'hui. Chaque fois que vous voyez le cadenas dans votre navigateur ou utilisez une URL commençant par `https://`, c'est TLS qui protège vos données contre l'interception et la modification.

TLS n'est pas simplement un "ajout de sécurité" optionnel — c'est devenu la **fondation même** d'Internet moderne :

**Statistiques 2024** :
- Plus de **95% du trafic web** utilise HTTPS (TLS)
- **100% des sites** du top 100 utilisent HTTPS par défaut
- Les navigateurs **avertissent activement** sur les sites HTTP non sécurisés
- Google **pénalise le référencement** des sites sans HTTPS

Sans TLS, les vulnérabilités que nous avons étudiées (sniffing, MITM, spoofing) rendraient Internet **inutilisable** pour tout échange sensible : banque, commerce, email, travail à distance, cloud computing, etc.

## Pourquoi TLS existe-t-il ?

### Le problème fondamental

Nous l'avons vu dans les sections précédentes : **TCP/IP ne fournit aucune sécurité native**. Toutes les communications transitent en clair par défaut.

**Exemple concret - HTTP sans TLS** :

```
Transaction bancaire sans TLS :

Client → Réseau (Wi-Fi café) → FAI → Internet → Banque

Requête HTTP visible en clair :
POST /transfer HTTP/1.1
Host: www.banque.fr
Cookie: session=abc123xyz789
Content-Type: application/x-www-form-urlencoded

from_account=123456&to_account=999888&amount=5000&currency=EUR

Toute personne sur le chemin peut :
✗ Lire les données (compte, montant, session)
✗ Modifier le montant ou le compte destinataire
✗ Voler le cookie de session
✗ Rejouer la transaction
```

**Avec TLS** :

```
Client → Connexion TLS chiffrée → Banque

Observable de l'extérieur :
- Établissement d'une connexion TCP vers 443
- Handshake TLS (métadonnées minimales)
- Flux de données chiffrées (incompréhensible)

Un attaquant voit :
�H�2��x�8���P��2���j�;�...
(totalement illisible)

Impossible de :
✗ Lire le contenu
✗ Modifier sans détection
✗ Rejouer les données
```

### Les trois garanties fondamentales de TLS

TLS fournit trois propriétés de sécurité essentielles, souvent résumées par l'acronyme **CIA** en sécurité informatique :

#### 1. Confidentialité (Confidentiality)

**Définition** : Les données ne peuvent être lues que par les parties légitimes de la communication.

```
Mécanisme : Chiffrement symétrique
- Algorithmes : AES-GCM, ChaCha20-Poly1305
- Taille de clé : 128 ou 256 bits
- Performance : Milliers de Mbps possibles avec accélération matérielle

Résultat :
Alice → [Données chiffrées] → Bob
       ↑
Eve (attaquant) ne peut pas déchiffrer même en interceptant
```

**Exemple pratique** :

```
Données originales :
"username=alice&password=secret123"

Après chiffrement AES-256-GCM :
0x8F 0x3A 0x9C 0x74 0xE2 0x55 0x1B 0x88 ...
(apparemment aléatoire)

Sans la clé de chiffrement :
- Impossible à déchiffrer même avec des superordinateurs
- Brute-force prendrait des milliards d'années
```

#### 2. Intégrité (Integrity)

**Définition** : Les données ne peuvent pas être modifiées sans détection.

```
Mécanisme : MAC (Message Authentication Code)
- Algorithmes : HMAC-SHA256, Poly1305
- Hash cryptographique de chaque message
- Inclus dans chaque paquet TLS

Résultat :
Alice → [Données + MAC] → Bob
                         ↓
                      Vérifie MAC
                      Si modifié → REJET
```

**Exemple pratique** :

```
Message original :
"Transfer 100€ to account 12345"

MAC calculé :
HMAC-SHA256(message, clé_secrète) =
0xA3F5C8D9E1B4A7C2F8D3E9B1...

Si un attaquant modifie :
"Transfer 10000€ to account 99999"

Le MAC ne correspondra plus :
Attendu : 0xA3F5C8D9E1B4A7C2F8D3E9B1...
Reçu après recalcul : 0x7B2D9A1F3C8E4B6D...
→ DIFFÉRENT → Message rejeté

L'attaquant ne peut pas recalculer le bon MAC sans la clé secrète
```

#### 3. Authenticité (Authenticity)

**Définition** : Chaque partie peut vérifier l'identité de l'autre.

```
Mécanisme : Certificats numériques + PKI
- Serveur prouve son identité avec un certificat
- Certificat signé par une Autorité de Certification (CA)
- Client vérifie la chaîne de confiance

Résultat :
Client → Qui êtes-vous ?
      ← Certificat : "Je suis www.banque.fr, signé par Let's Encrypt"
Client → Vérifie signature
      → Vérifie domaine
      → Vérifie validité
      → OK, je te fais confiance
```

**Exemple pratique** :

```
Sans authenticité (HTTP) :
Client se connecte à "www.banque.fr"
→ Pourrait être n'importe qui
→ Attaque MITM possible
→ Phishing trivial

Avec TLS :
Client demande le certificat
Certificat indique :
- Sujet : www.banque.fr
- Émetteur : DigiCert
- Signature cryptographique de DigiCert
- Validité : 2024-01-01 → 2025-01-01

Client vérifie :
✓ La signature est valide (DigiCert est de confiance)
✓ Le domaine correspond
✓ La date est valide
✓ Le certificat n'est pas révoqué

Résultat : CONFIANCE établie
```

## Historique et évolution de SSL/TLS

### Ligne de temps

```
1994 : SSL 1.0 (Netscape)
       → Jamais publié (vulnérabilités découvertes en interne)

1995 : SSL 2.0
       → Première version publique
       → Vulnérabilités majeures découvertes rapidement

1996 : SSL 3.0
       → Refonte complète
       → Base pour TLS

1999 : TLS 1.0 (RFC 2246)
       → Successeur de SSL 3.0
       → Changement de nom (IETF prend le relais)

2006 : TLS 1.1 (RFC 4346)
       → Protection contre attaques CBC

2008 : TLS 1.2 (RFC 5246)
       → Chiffrement authentifié (AEAD)
       → SHA-256

2018 : TLS 1.3 (RFC 8446)
       → Refonte majeure
       → Simplification et sécurité accrue
       → Performance améliorée

2024 : TLS 1.3 standard de facto
       → TLS 1.0/1.1 dépréciés et désactivés
       → TLS 1.2 encore largement utilisé
       → Migration vers TLS 1.3 en cours
```

### SSL 1.0 et 2.0 : les débuts chaotiques

**SSL 1.0** (1994, jamais publié) :
- Développé en secret par Netscape
- Vulnérabilités critiques découvertes avant publication
- Abandonné avant release publique

**SSL 2.0** (1995) :

```
Vulnérabilités majeures :
✗ Pas de protection contre rollback attack
✗ Même clé pour chiffrement et authentification
✗ Faiblesse dans le choix du cipher suite
✗ MAC trop faible (MD5)

Conséquences :
- Déprécié en 1996 (après 1 an seulement)
- Désactivé dans tous les navigateurs modernes
- Utilisé aujourd'hui = ERREUR GRAVE
```

### SSL 3.0 : la base de TLS

**SSL 3.0** (1996) :

```
Améliorations majeures :
✓ Refonte complète du protocole
✓ MAC plus robuste (SHA-1)
✓ Protection contre rollback
✓ Meilleure négociation cipher suite

Mais :
✗ Vulnérable à POODLE (2014)
✗ Vulnérable à BEAST (2011)
✗ Cryptographie dépassée

Statut actuel :
- Officiellement déprécié en 2015 (RFC 7568)
- Désactivé dans tous les navigateurs modernes
- Utilisé = ERREUR CRITIQUE
```

**Attaque POODLE** (2014) :

```
Padding Oracle On Downgraded Legacy Encryption

Principe :
1. Forcer downgrade vers SSL 3.0
2. Exploiter faiblesse dans le padding CBC
3. Déchiffrer cookies en ~256 requêtes

Impact :
- Vol de cookies de session
- Session hijacking possible

Solution :
- Désactiver complètement SSL 3.0
- TLS_FALLBACK_SCSV pour empêcher downgrade
```

### TLS 1.0 et 1.1 : l'ère de transition

**TLS 1.0** (1999) :

```
Changements vs SSL 3.0 :
✓ Nouveau nom (TLS au lieu de SSL)
✓ HMAC au lieu de MAC maison
✓ Meilleure gestion des alertes
✓ Extensibilité améliorée

Mais toujours :
✗ Vulnérable à BEAST (2011)
✗ Cryptographie vieillissante
✗ Pas de support AEAD

Statut 2024 :
- Déprécié officiellement (RFC 8996, 2021)
- Désactivé par défaut dans navigateurs
- PCI-DSS l'interdit depuis 2018
```

**TLS 1.1** (2006) :

```
Améliorations :
✓ Protection contre BEAST (IV explicite)
✓ Protection contre attaques de timing

Limitations :
✗ Améliorations mineures seulement
✗ Adoption limitée (saut vers 1.2)

Statut 2024 :
- Également déprécié (RFC 8996)
- Désactivé dans navigateurs modernes
```

### TLS 1.2 : le standard actuel

**TLS 1.2** (2008) :

```
Avancées majeures :
✓ Chiffrement authentifié (AEAD)
  - AES-GCM
  - ChaCha20-Poly1305
✓ SHA-256 (et SHA-384)
✓ Support extensible des signatures
✓ Flexibilité dans les cipher suites

Performance :
✓ Accélération matérielle (AES-NI)
✓ Bon compromis sécurité/vitesse

Adoption :
✓ Standard de facto 2010-2020
✓ Encore dominant en 2024 (~60-70% du trafic)
✓ Support universel
```

**Cipher suites recommandées TLS 1.2** :

```
Format : TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
         │   │     │    │   │   │   │   │
         │   │     │    │   │   │   │   └─ PRF (SHA-256)
         │   │     │    │   │   │   └───── MAC intégré à GCM
         │   │     │    │   │   └─────────── Mode (GCM)
         │   │     │    │   └─────────────── Taille clé (128)
         │   │     │    └─────────────────── Chiffrement (AES)
         │   │     └──────────────────────── Auth serveur (RSA)
         │   └────────────────────────────── Échange clé (ECDHE)
         └────────────────────────────────── Protocole

Recommandations 2024 :
1. TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
2. TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
3. TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256

À ÉVITER :
✗ Tout ce qui contient CBC (vulnérable)
✗ Tout ce qui contient RC4 (cassé)
✗ Tout ce qui contient MD5 (cassé)
✗ Tout ce qui contient NULL (pas de chiffrement !)
✗ Tout ce qui contient EXPORT (affaibli intentionnellement)
```

### TLS 1.3 : la révolution moderne

**TLS 1.3** (2018, RFC 8446) :

```
Philosophie :
"Supprimer tout ce qui n'est pas sûr, simplifier, accélérer"

Changements radicaux :
✓ Handshake plus rapide (1-RTT au lieu de 2-RTT)
✓ 0-RTT possible (reprise de session)
✓ Seulement 5 cipher suites (vs des dizaines en 1.2)
✓ Forward secrecy obligatoire (ECDHE/DHE)
✓ Tout supprimé : RSA key exchange, SHA-1, MD5, RC4, CBC, compression
✓ Chiffrement du handshake (moins de métadonnées exposées)

Performance :
✓ 30-50% plus rapide que TLS 1.2
✓ Latence réduite (critique pour mobile)

Sécurité :
✓ Surface d'attaque réduite drastiquement
✓ Impossible de négocier vers algo faibles
✓ Résistant aux attaques connues
```

**Les 5 cipher suites TLS 1.3** :

```
1. TLS_AES_128_GCM_SHA256              (obligatoire)
2. TLS_AES_256_GCM_SHA384
3. TLS_CHACHA20_POLY1305_SHA256
4. TLS_AES_128_CCM_SHA256
5. TLS_AES_128_CCM_8_SHA256

Simplification radicale :
- Pas de RSA key exchange (toujours ECDHE/DHE)
- Pas de choix d'algorithme de signature dans le cipher suite
- Seulement AEAD (pas de CBC)
- Forward secrecy toujours présente
```

**Adoption TLS 1.3** :

```
Fin 2024 :
- ~40-50% du trafic HTTPS utilise TLS 1.3
- Support dans tous les navigateurs modernes
- Support dans tous les serveurs web majeurs
- Migration progressive depuis TLS 1.2

Obstacles à l'adoption :
- Équipements réseau legacy (middleboxes)
- Applications anciennes
- Coût de migration pour grandes infrastructures

Tendance : TLS 1.3 deviendra dominant d'ici 2025-2026
```

## Concepts cryptographiques fondamentaux

TLS utilise une combinaison de techniques cryptographiques. Comprenons les bases avant d'étudier leur utilisation dans TLS.

### Chiffrement symétrique

**Principe** : Une seule clé pour chiffrer et déchiffrer.

```
Alice et Bob partagent une clé secrète : K

Chiffrement :
Texte clair : "Hello Bob"
Clé : K
Algorithme : AES
→ Texte chiffré : 0x8A3F9C2E...

Déchiffrement :
Texte chiffré : 0x8A3F9C2E...
Clé : K
Algorithme : AES
→ Texte clair : "Hello Bob"

Propriété essentielle :
Chiffrer(M, K) → C
Déchiffrer(C, K) → M
```

**Algorithmes principaux en TLS** :

```
AES (Advanced Encryption Standard) :
- Tailles de clé : 128, 192, 256 bits
- Vitesse : Très rapide (surtout avec AES-NI)
- Sécurité : Standard, aucune attaque pratique
- Usage : 95% des connexions TLS

ChaCha20 :
- Taille de clé : 256 bits
- Vitesse : Très rapide sur mobile (pas d'AES-NI)
- Sécurité : Moderne, robuste
- Usage : Mobile, IoT, où AES-NI absent
```

**Modes de chiffrement** :

```
CBC (Cipher Block Chaining) - DÉPRÉCIÉ :
- Chaîne les blocs entre eux
- Vulnérable aux attaques padding oracle
- NE PLUS UTILISER

GCM (Galois/Counter Mode) - RECOMMANDÉ :
- Chiffrement + authentification intégrée (AEAD)
- Parallélisable (performance)
- Standard actuel

CCM (Counter with CBC-MAC) :
- Également AEAD
- Utilisé dans IoT (moins de ressources)

ChaCha20-Poly1305 :
- Chiffrement ChaCha20 + MAC Poly1305
- AEAD
- Excellent pour mobile
```

**Exemple AES-128-GCM** :

```
Paramètres :
- Clé : 128 bits (16 octets)
- Nonce : 96 bits (12 octets, unique par message)
- Données additionnelles (AAD) : métadonnées non chiffrées

Chiffrement :
Entrée :
  - Message : "Transaction de 500€"
  - AAD : "Numéro de séquence: 42"

Sortie :
  - Texte chiffré : 0x9A3F... (même longueur que message)
  - Tag d'authentification : 0xB2E5... (128 bits)

Le tag authentifie BOTH message chiffré ET AAD
→ Protection contre modification du contenu ET des métadonnées
```

### Chiffrement asymétrique

**Principe** : Paire de clés (publique/privée), ce qui est chiffré avec l'une ne peut être déchiffré qu'avec l'autre.

```
Génération :
Alice génère une paire de clés :
- Clé publique : PubK_Alice (peut être partagée)
- Clé privée : PrivK_Alice (SECRÈTE)

Chiffrement (confidentialité) :
Bob → Obtient PubK_Alice
Bob → Chiffre avec PubK_Alice : "Hello Alice"
Bob → Envoie texte chiffré
Alice → Déchiffre avec PrivK_Alice
Alice → Lit : "Hello Alice"

Propriété :
Chiffrer(M, PubK) → C
Déchiffrer(C, PrivK) → M
Seul le détenteur de PrivK peut déchiffrer
```

**Signature numérique** :

```
Alice veut signer un message :

Signature :
Message : "Je transfert 100€ à Bob"
Hash : SHA-256(Message) → H
Signature : Chiffrer(H, PrivK_Alice) → S

Envoi :
Alice → Bob : (Message, S)

Vérification par Bob :
H1 = SHA-256(Message)
H2 = Déchiffrer(S, PubK_Alice)
if H1 == H2 → Signature VALIDE
            → Message bien d'Alice et non modifié
else → REJET
```

**Algorithmes en TLS** :

```
RSA (Rivest-Shamir-Adleman) :
- Tailles : 2048, 3072, 4096 bits (minimum 2048)
- Usage historique : Échange de clés (déprécié en TLS 1.3)
- Usage actuel : Signatures uniquement
- Performance : Lent (surtout opérations privées)

ECDSA (Elliptic Curve Digital Signature Algorithm) :
- Courbes : P-256, P-384
- Taille équivalente : 256 bits ≈ RSA 3072 bits
- Performance : Plus rapide que RSA
- Taille : Clés et signatures beaucoup plus petites

EdDSA (Edwards-curve Digital Signature Algorithm) :
- Courbe : Ed25519
- Performance : Très rapide
- Sécurité : Moderne, robuste
- Usage : TLS 1.3
```

**Comparaison de taille** :

```
RSA-2048 :
- Clé publique : 294 octets
- Signature : 256 octets

ECDSA P-256 :
- Clé publique : 91 octets
- Signature : 72 octets

Ed25519 :
- Clé publique : 32 octets
- Signature : 64 octets

Impact sur TLS :
- Certificats plus petits (ECDSA)
- Handshake plus rapide (moins de données)
- Crucial pour mobile/IoT
```

### Fonctions de hachage cryptographique

**Propriétés essentielles** :

```
Hash : {0,1}* → {0,1}^n  (longueur fixe)

Propriétés requises :
1. Déterministe : Hash(M) donne toujours le même résultat
2. Rapide : Calcul efficace
3. Avalanche : Modification d'un bit → ~50% du hash change
4. Unidirectionnel : Impossible de retrouver M depuis Hash(M)
5. Résistance aux collisions : Infaisable de trouver M1≠M2 avec Hash(M1)=Hash(M2)
```

**Algorithmes** :

```
MD5 (128 bits) - CASSÉ :
✗ Collisions trouvables en secondes
✗ NE JAMAIS UTILISER pour sécurité
✓ OK pour checksums non-critiques

SHA-1 (160 bits) - CASSÉ :
✗ Collisions démontrées (2017, Google)
✗ Déprécié dans TLS
✗ NE PLUS UTILISER

SHA-2 family - RECOMMANDÉ :
✓ SHA-256 (256 bits) - Standard actuel
✓ SHA-384 (384 bits) - Haute sécurité
✓ SHA-512 (512 bits) - Très haute sécurité
✓ Aucune attaque pratique connue

SHA-3 family - MODERNE :
✓ Algorithme différent (Keccak)
✓ Alternative à SHA-2
✓ Adoption progressive
```

**Usage dans TLS** :

```
1. Signatures de certificats :
   Certificat → Hash (SHA-256) → Signature avec clé privée CA

2. PRF (Pseudo-Random Function) :
   Génération de clés de session depuis master secret

3. HMAC (Hash-based MAC) :
   TLS 1.2 et antérieurs : Intégrité des messages

4. Finished message :
   Hash de tout le handshake → Vérification d'intégrité

5. Fingerprinting de certificats :
   SHA-256(certificat) → Identifiant unique
```

### Échange de clés Diffie-Hellman

**Problème** : Comment Alice et Bob peuvent-ils établir un secret partagé sur un canal non sécurisé ?

**Solution Diffie-Hellman (DH)** :

```
Paramètres publics :
- p : grand nombre premier
- g : générateur

Échange :
Alice :                           Bob :
  a = random()                      b = random()
  A = g^a mod p                     B = g^b mod p

  --------  A  ------->
  <-------  B  ---------

  secret = B^a mod p                secret = A^b mod p

Magie mathématique :
  B^a = (g^b)^a = g^(ab) mod p
  A^b = (g^a)^b = g^(ab) mod p
  → Même secret !

Observateur Eve voit : p, g, A, B
Mais ne peut pas calculer g^(ab) sans a ou b
(problème du logarithme discret - difficile)
```

**ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)** :

```
Amélioration moderne :
- Courbes elliptiques au lieu de modulo
- Clés éphémères (changent à chaque connexion)
- Plus rapide que DH classique
- Clés plus petites pour même sécurité

Forward Secrecy :
Même si clé privée serveur compromise dans le futur
→ Connexions passées restent sécurisées
→ Car clés de session générées avec clés éphémères

Usage en TLS :
TLS 1.2 : ECDHE recommandé
TLS 1.3 : ECDHE obligatoire
```

**Courbes elliptiques** :

```
Courbes recommandées :

P-256 (secp256r1) :
- NIST standard
- Support universel
- Sécurité ≈ AES-128

P-384 (secp384r1) :
- Haute sécurité
- Sécurité ≈ AES-192

X25519 :
- Curve25519
- Très rapide
- Moderne (DJB)
- Recommandé TLS 1.3

X448 :
- Curve448
- Très haute sécurité
```

## Architecture de TLS

### Position dans la pile TCP/IP

```
Modèle TCP/IP avec TLS :

+------------------------+
|   Application          |  HTTP, FTP, SMTP, etc.
+------------------------+
|   TLS/SSL              |  ← Couche de sécurité
+------------------------+
|   Transport (TCP)      |  Connexions fiables
+------------------------+
|   Internet (IP)        |  Routage
+------------------------+
|   Accès réseau         |  Liaison physique
+------------------------+

TLS s'insère entre Application et Transport
→ Transparent pour les applications (presque)
→ Utilise TCP comme transport fiable
```

### TLS n'est PAS une couche ISO/OSI stricte

```
TLS est un protocole hybride :

Niveau session (couche 5) :
- Établit et maintient des sessions
- Gère les reprises de session

Niveau présentation (couche 6) :
- Chiffre/déchiffre les données
- Compression (historiquement)

Mais en pratique :
TLS = Bibliothèque utilisée par applications
Pas vraiment une "couche" autonome
```

### Composants de TLS

```
TLS se compose de plusieurs sous-protocoles :

1. Handshake Protocol :
   - Négociation des paramètres
   - Authentification
   - Établissement des clés

2. Record Protocol :
   - Fragmentation
   - Chiffrement
   - Intégrité
   - Transmission

3. Alert Protocol :
   - Gestion des erreurs
   - Notifications
   - Fermeture de connexion

4. Change Cipher Spec Protocol (TLS ≤1.2) :
   - Signal de changement d'état
   - (Supprimé en TLS 1.3)
```

## HTTPS : HTTP over TLS

### Fonctionnement de base

```
HTTP normal (port 80) :
Client ← TCP connexion → Serveur
Client ← Requête HTTP en clair → Serveur

HTTPS (port 443) :
Client ← TCP connexion → Serveur
Client ← Handshake TLS → Serveur
       ← Connexion TLS sécurisée établie ←
Client ← Requête HTTP (chiffrée via TLS) → Serveur

Invisible pour l'application HTTP :
Le navigateur/serveur web utilise TLS de façon transparente
HTTP ne "sait" pas qu'il est chiffré
```

### Exemple de flux HTTPS complet

```
1. Utilisateur tape : https://www.example.com

2. Navigateur :
   - Résolution DNS : www.example.com → 93.184.216.34
   - Connexion TCP vers 93.184.216.34:443

3. Handshake TLS (détaillé dans section 6.4.1) :
   - ClientHello
   - ServerHello
   - Certificat
   - Échange de clés
   - Finished
   → Connexion sécurisée établie

4. Requête HTTP (chiffrée) :
   GET / HTTP/1.1
   Host: www.example.com
   ...

5. Réponse HTTP (chiffrée) :
   HTTP/1.1 200 OK
   Content-Type: text/html
   ...
   <html>...</html>

6. Données affichées dans le navigateur
```

### Indicateurs visuels de sécurité

```
Dans le navigateur :

🔒 Cadenas vert/gris :
   - Connexion HTTPS active
   - Certificat valide
   - Pas de contenu mixte

⚠️ Avertissement :
   - Certificat expiré
   - Certificat auto-signé
   - Nom de domaine ne correspond pas
   - Autorité de certification non reconnue

🔓 ou ⚠️ "Non sécurisé" :
   - HTTP (pas de TLS)
   - Évident danger

🔒 avec barre verte (EV - Extended Validation) :
   - Validation approfondie de l'entité
   - Affichage du nom de l'organisation
   - En déclin (navigateurs suppriment l'affichage spécial)
```

## Autres protocoles utilisant TLS

### SMTPS, IMAPS, POP3S

```
Email sécurisé :

SMTP (envoi) :
- Port 25 : Non chiffré (legacy)
- Port 587 : STARTTLS (upgrade vers TLS)
- Port 465 : SMTPS (TLS dès connexion)

IMAP (réception) :
- Port 143 : Non chiffré
- Port 993 : IMAPS

POP3 (réception) :
- Port 110 : Non chiffré
- Port 995 : POP3S
```

### FTPS (FTP over TLS)

```
Deux modes :

Explicit FTPS (port 21) :
- Connexion FTP normale
- Commande AUTH TLS
- Upgrade vers TLS

Implicit FTPS (port 990) :
- TLS dès la connexion
- Comme HTTPS

Note : SFTP ≠ FTPS
SFTP = SSH File Transfer Protocol (pas TLS, mais SSH)
```

### LDAPS (LDAP over TLS)

```
Annuaire sécurisé :
- Port 389 : LDAP
- Port 636 : LDAPS
- Utilisé pour Active Directory, etc.
```

### VPN avec TLS

```
OpenVPN :
- Utilise TLS pour le chiffrement
- Plus flexible que IPsec
- Traverse facilement les firewalls (port 443)

Avantages TLS pour VPN :
✓ Certification standard
✓ Bibliothèques robustes
✓ Déploiement facile
```

## Certificats et PKI (Public Key Infrastructure)

### Qu'est-ce qu'un certificat ?

**Définition** : Un certificat est un document numérique qui **lie une clé publique à une identité** (domaine, organisation, personne).

```
Structure d'un certificat X.509 :

+----------------------------------+
| Version                          | v3
| Numéro de série                  | Unique par CA
| Algorithme de signature          | SHA-256 with RSA
| Émetteur (CA)                    | DigiCert
| Validité                         |
|   - Pas avant                    | 2024-01-01 00:00:00 UTC
|   - Pas après                    | 2025-01-01 23:59:59 UTC
| Sujet                            |
|   - Common Name (CN)             | www.example.com
|   - Organization (O)             | Example Inc.
|   - Country (C)                  | US
| Clé publique du sujet            |
|   - Algorithme                   | RSA
|   - Taille                       | 2048 bits
|   - Clé                          | [données binaires]
| Extensions                       |
|   - Subject Alternative Names    | example.com, www.example.com
|   - Key Usage                    | Digital Signature, Key Encipherment
|   - Extended Key Usage           | Server Authentication
| Signature de la CA               |
|   - Algorithme                   | SHA-256 with RSA
|   - Signature                    | [hash du certificat chiffré avec clé privée CA]
+----------------------------------+
```

### Chaîne de certification

```
Trust chain (chaîne de confiance) :

Root CA (Autorité racine)
  |
  | Signe
  ↓
Intermediate CA (Autorité intermédiaire)
  |
  | Signe
  ↓
End-entity Certificate (Certificat serveur)
  |
  | Utilisé par
  ↓
www.example.com

Vérification par le client :
1. Reçoit certificat de www.example.com
2. Vérifie signature avec clé publique Intermediate CA
3. Vérifie certificat Intermediate CA avec clé publique Root CA
4. Root CA est dans le trust store du navigateur
5. → Chaîne valide, confiance établie
```

**Trust store** :

```
Navigateurs/OS embarquent ~100-200 certificats racine de confiance :

Exemples de CAs racine :
- DigiCert
- Let's Encrypt ISRG Root X1
- GlobalSign
- Sectigo (ex-Comodo)
- GeoTrust
- Entrust

Installation :
- Préinstallés dans OS/navigateur
- Mis à jour régulièrement
- Révocation possible si CA compromise

Vérifier sur Linux :
/etc/ssl/certs/ca-certificates.crt
ou
/etc/pki/tls/certs/ca-bundle.crt
```

### Types de certificats

```
1. Domain Validation (DV) :
   ✓ Validation : Contrôle du domaine uniquement
   ✓ Coût : Gratuit (Let's Encrypt) ou faible
   ✓ Délivrance : Automatisée, quelques minutes
   ✓ Usage : Sites web standards
   ✓ Affichage : Cadenas standard

2. Organization Validation (OV) :
   ✓ Validation : Domaine + existence organisation
   ✓ Coût : Moyen (~100-500€/an)
   ✓ Délivrance : Manuelle, quelques jours
   ✓ Usage : Sites d'entreprise
   ✓ Affichage : Cadenas + info organisation (si cliqué)

3. Extended Validation (EV) :
   ✓ Validation : Vérification approfondie de l'entité
   ✓ Coût : Élevé (~500-2000€/an)
   ✓ Délivrance : Processus long, 1-2 semaines
   ✓ Usage : Banques, sites haute sécurité
   ✓ Affichage : Historiquement barre verte (supprimé par navigateurs)
   ✗ Déclin : Valeur perçue réduite

4. Wildcard :
   ✓ Couvre : *.example.com
   ✓ Usage : Multiples sous-domaines
   ✓ Exemple : www, mail, api, blog.example.com

5. Multi-Domain (SAN) :
   ✓ Couvre : Plusieurs domaines différents
   ✓ Exemple : example.com, example.org, example.net
```

### Let's Encrypt : révolution de la certification

```
Lancé : 2016
Philosophie : HTTPS pour tous, gratuitement

Caractéristiques :
✓ Gratuit
✓ Automatisé (protocole ACME)
✓ Renouvelable automatiquement
✓ DV uniquement
✓ Validité 90 jours (encourage automatisation)

Impact :
- Adoption HTTPS passée de ~40% à ~95%
- Démocratisation du chiffrement web
- Standard de facto pour sites personnels/petites entreprises

Utilisation avec Certbot :
$ certbot --nginx -d www.example.com
→ Obtient certificat
→ Configure Nginx automatiquement
→ Configure renouvellement automatique
```

### Révocation de certificats

**Pourquoi révoquer** :

```
Raisons :
✗ Clé privée compromise
✗ Certificat émis par erreur
✗ Information fausse dans le certificat
✗ CA compromise
✗ Changement de propriétaire du domaine
```

**Mécanismes** :

```
1. CRL (Certificate Revocation List) :
   - Liste publiée par la CA
   - Téléchargée par les clients
   - Problème : Taille croissante
   - En déclin

2. OCSP (Online Certificate Status Protocol) :
   - Requête en temps réel à la CA
   - Réponse : Good / Revoked / Unknown
   - Problème : Latence, privacy
   - Largement utilisé

3. OCSP Stapling :
   - Serveur obtient réponse OCSP
   - Inclut dans handshake TLS
   - Client vérifie la réponse
   - Avantage : Pas de requête CA par client
   - Recommandé

4. Certificate Transparency :
   - Logs publics de tous certificats émis
   - Détection rapide d'émissions frauduleuses
   - Obligatoire pour certains certificats
```

## Performance et optimisations

### Coût de TLS

```
Overhead TLS vs HTTP :

Latence :
- Handshake : +1-2 RTT (TLS 1.2) ou 0-1 RTT (TLS 1.3)
- Impact initial : 50-200ms
- Sessions suivantes : Minimisé (session resumption)

CPU :
- Handshake : Opérations asymétriques coûteuses
- Transfert données : Chiffrement symétrique (~5-10% CPU)
- Avec accélération matérielle (AES-NI) : <1% CPU

Bande passante :
- Headers TLS : ~40 octets par record
- Overhead : ~5% typiquement
- Négligeable pour transferts importants

Conclusion :
Le coût de TLS est devenu NÉGLIGEABLE sur matériel moderne
Le gain en sécurité justifie largement le coût
```

### Session Resumption

**Problème** : Handshake coûteux à chaque nouvelle connexion.

**Solution 1 - Session ID (TLS 1.2)** :

```
Première connexion :
Client → ServerHello avec Session ID
       ← Session enregistrée (client + serveur)

Reconnexion :
Client → ClientHello avec Session ID précédent
Server → Reconnaît le Session ID
       → Handshake abrégé (pas de full handshake)
       → Réutilisation master secret

Économie : 1 RTT au lieu de 2 RTT
```

**Solution 2 - Session Tickets (TLS 1.2)** :

```
Principe : Serveur envoie état chiffré au client

Première connexion :
Server → Chiffre état de session avec clé serveur
       → Envoie "ticket" au client
Client → Stocke ticket (opaque)

Reconnexion :
Client → Envoie ticket dans ClientHello
Server → Déchiffre ticket
       → Retrouve état de session
       → Handshake abrégé

Avantage vs Session ID :
- Serveur stateless (pas de stockage session)
- Meilleur pour load balancing
```

**Solution 3 - 0-RTT (TLS 1.3)** :

```
Reconnexion ultra-rapide :

Client → ClientHello + données applicatives + PSK
       (PSK = Pre-Shared Key de session précédente)

Server → Accepte et répond immédiatement
       → 0 RTT pour les données

Gain : Données envoyées dans premier paquet !

Risque : Replay attack
→ Utilisé seulement pour requêtes idempotentes
→ GET OK, POST/PUT/DELETE non recommandé
```

### HTTP/2 et HTTP/3 avec TLS

```
HTTP/2 over TLS :
- Multiplexage de requêtes
- Server push
- Header compression (HPACK)
- Réutilisation connexion TCP/TLS
→ Amortissement coût handshake

HTTP/3 over QUIC (TLS 1.3 intégré) :
- Transport sur UDP
- Intégration TLS 1.3 dans QUIC
- 0-RTT dès première connexion
- Meilleure résilience mobile
→ Performance optimale
```

## Conclusion

TLS/SSL est passé du statut de "fonctionnalité optionnelle" à **composant fondamental d'Internet**. Sans TLS, l'économie numérique, le télétravail, le cloud computing, et la plupart des usages modernes d'Internet seraient impossibles.

**Points clés à retenir** :

1. **Trois garanties** : Confidentialité, Intégrité, Authenticité
2. **Évolution** : SSL → TLS 1.0/1.1 → TLS 1.2 (standard actuel) → TLS 1.3 (futur)
3. **Cryptographie hybride** : Asymétrique (handshake) + Symétrique (données)
4. **PKI essentielle** : Certificats et chaînes de confiance
5. **Performance acceptable** : Coût négligeable sur matériel moderne
6. **Adoption massive** : >95% du web utilise HTTPS

**Statut en 2024** :

```
✓ TLS 1.3 : Adoption croissante (~40-50%)
✓ TLS 1.2 : Standard dominant (~50-60%)
✗ TLS ≤1.1 : Déprécié et désactivé
✗ SSL 3.0 : Complètement abandonné

Tendance : Migration progressive vers TLS 1.3 uniquement
```

**Prochaines sections** :

Dans les sections suivantes, nous approfondirons :
- **6.4.1** : Le handshake TLS en détail (négociation, authentification, établissement des clés)
- **6.4.2** : Certificats et PKI (autorités de certification, chaînes de confiance, révocation)

Ces mécanismes sont au cœur du fonctionnement de TLS et méritent une étude approfondie pour comprendre comment la sécurité est réellement assurée dans les communications modernes.

⏭️ [Handshake TLS](/06-securite/04.1-tls-handshake.md)
