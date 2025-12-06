🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.6 NAT et PAT : principes et mécanismes

## Introduction : le traducteur d'adresses

Imaginez que vous travaillez dans une grande entreprise internationale. Lorsque vous appelez un client au Japon, vous ne composez pas directement depuis votre téléphone de bureau. Vous passez par le **standard téléphonique** de l'entreprise :

1. Vous appelez le standard (numéro interne 101)
2. Le standard prend votre appel et compose le numéro international
3. Pour le client japonais, l'appel semble venir du numéro principal de l'entreprise
4. Quand le client répond, le standard vous transfère l'appel

Le **NAT (Network Address Translation)** fonctionne exactement comme ce standard téléphonique, mais pour les adresses IP. Il permet à de nombreux appareils avec des **adresses privées** de partager une **seule adresse publique** pour accéder à Internet.

**Sans NAT, Internet ne pourrait pas fonctionner aujourd'hui** avec seulement 4,3 milliards d'adresses IPv4 pour des dizaines de milliards d'appareils connectés.

## Qu'est-ce que le NAT ?

### Définition simple

**NAT (Network Address Translation)** = Traduction d'adresses réseau

C'est un mécanisme qui **modifie les adresses IP** dans les en-têtes des paquets lorsqu'ils traversent un routeur, permettant ainsi de :
- Faire correspondre plusieurs adresses privées à une (ou plusieurs) adresse(s) publique(s)
- Permettre aux appareils en réseau privé d'accéder à Internet
- Masquer la structure interne du réseau

### Où se trouve le NAT ?

Le NAT est généralement implémenté dans :

```
┌─────────────────────────────────────────────┐
│         Votre réseau domestique             │
│                                             │
│  PC          Smartphone       Tablette      │
│  192.168.1.10  192.168.1.20   192.168.1.30  │
│           │          │           │          │
│           └──────────┴───────────┘          │
│                     │                       │
│              ┌──────▼──────┐                │
│              │  Box/Routeur│                │
│              │  (NAT ici!) │                │
│              │ 192.168.1.1 │                │
│              └──────┬──────┘                │
└─────────────────────┼───────────────────────┘
                      │ IP publique
                      │ 203.0.113.45
                      │
                 INTERNET
```

**Dans la plupart des cas** :
- Votre **box Internet** (Freebox, Livebox, Bbox...)
- Un **routeur d'entreprise**
- Un **firewall** avec fonction NAT

### Le problème que résout le NAT

**Scénario sans NAT** (impossible aujourd'hui) :

```
Besoin : 10 appareils veulent accéder à Internet

Solution naïve : 10 adresses IP publiques
  PC 1 : 203.0.113.1
  PC 2 : 203.0.113.2
  PC 3 : 203.0.113.3
  ...
  PC 10 : 203.0.113.10

Problème : ❌ Gaspillage d'adresses publiques
          ❌ Coût élevé
          ❌ Pénurie mondiale d'IPv4
```

**Scénario avec NAT** (solution actuelle) :

```
Solution : 1 seule adresse IP publique partagée

  PC 1 : 192.168.1.10  ┐
  PC 2 : 192.168.1.20  ├─→ NAT ─→ 203.0.113.45
  PC 3 : 192.168.1.30  ┘

Résultat : ✅ Économie massive d'adresses
          ✅ Coût réduit
          ✅ Internet peut continuer de fonctionner
```

## Les types de NAT

Il existe **trois types principaux** de NAT, chacun avec ses spécificités.

### 1. NAT Statique (Static NAT)

**Principe** : Mappage **1-pour-1** permanent entre une adresse privée et une adresse publique.

**Fonctionnement** :

```
Configuration fixe :
192.168.1.10 ←→ 203.0.113.50
192.168.1.20 ←→ 203.0.113.51
192.168.1.30 ←→ 203.0.113.52

Cette correspondance ne change jamais.
```

**Schéma** :

```
Réseau interne              Routeur NAT          Internet

192.168.1.10 ─────────────→ 203.0.113.50 ────→ Serveur web
                           (traduction fixe)

Toujours la même
correspondance
```

**Cas d'usage** :
- Serveurs publics (web, mail, FTP) hébergés en interne
- Équipements qui doivent être joignables depuis l'extérieur
- Besoin d'une adresse publique dédiée

**Exemple pratique** :

```
Entreprise avec 3 serveurs :

Serveur web interne : 192.168.1.10
  → Accessible via : 203.0.113.50

Serveur mail interne : 192.168.1.20
  → Accessible via : 203.0.113.51

Serveur FTP interne : 192.168.1.30
  → Accessible via : 203.0.113.52
```

**Avantages** :
- ✅ Simple et prévisible
- ✅ Serveur joignable depuis Internet
- ✅ Pas de limitation de protocole

**Inconvénients** :
- ❌ Nécessite autant d'IPs publiques que d'IPs privées
- ❌ Ne résout pas la pénurie d'adresses
- ❌ Coût élevé (IPs publiques payantes)

**Analogie** : C'est comme avoir une ligne téléphonique directe pour chaque employé. Pratique mais coûteux !

### 2. NAT Dynamique (Dynamic NAT)

**Principe** : Un **pool d'adresses publiques** est partagé dynamiquement entre les appareils internes.

**Fonctionnement** :

```
Pool d'adresses publiques disponibles :
203.0.113.50
203.0.113.51
203.0.113.52

Quand un appareil interne veut sortir :
1. Le NAT lui attribue une IP du pool
2. Tant que la connexion est active, l'IP est réservée
3. Après la connexion, l'IP retourne dans le pool
```

**Exemple de table NAT** :

```
┌──────────────────┬───────────────────┬────────────┐
│ IP Interne       │ IP Publique       │ État       │
├──────────────────┼───────────────────┼────────────┤
│ 192.168.1.10     │ 203.0.113.50      │ Actif      │
│ 192.168.1.20     │ 203.0.113.51      │ Actif      │
│ 192.168.1.30     │ 203.0.113.52      │ Actif      │
│ 192.168.1.40     │ (aucune)          │ En attente │
└──────────────────┴───────────────────┴────────────┘

Si 192.168.1.40 veut accéder à Internet maintenant,
il doit attendre qu'une IP se libère dans le pool !
```

**Schéma** :

```
Réseau interne              Pool NAT              Internet

192.168.1.10 ───┐          ┌─ 203.0.113.50 ────→
192.168.1.20 ───┼────→ NAT ├─ 203.0.113.51 ────→
192.168.1.30 ───┘          └─ 203.0.113.52 ────→

         Premier arrivé, premier servi
```

**Avantages** :
- ✅ Moins d'IPs publiques que d'appareils
- ✅ Utilisation optimisée du pool

**Inconvénients** :
- ❌ Nécessite toujours plusieurs IPs publiques
- ❌ Limite le nombre de connexions simultanées
- ❌ Complexité de gestion

**Analogie** : C'est comme un parking avec 10 places pour 50 employés. Ça fonctionne car tout le monde n'arrive pas en même temps !

### 3. PAT / NAT Overload (le plus courant)

**PAT (Port Address Translation)** = aussi appelé **NAT Overload** ou **NAPT**

**Principe** : Une **seule adresse IP publique** est partagée par tous les appareils grâce à la **différenciation par ports**.

C'est le type de NAT utilisé par **toutes les box Internet domestiques** et la plupart des routeurs d'entreprise.

**La magie du PAT** :

```
Une seule IP publique : 203.0.113.45

Mais des milliers d'appareils peuvent l'utiliser simultanément
grâce aux numéros de port !

192.168.1.10:45678 ───┐
192.168.1.20:12345 ───┼─→ NAT ─→ 203.0.113.45:1024-65535
192.168.1.30:33891 ───┘                    ↑
                                    Ports différents !
```

**Fonctionnement détaillé** :

1. L'appareil interne crée une connexion avec un **port source aléatoire**
2. Le NAT **traduit l'IP privée → IP publique** ET **change potentiellement le port**
3. Le NAT **mémorise** la correspondance dans une table
4. Les réponses sont redirigées vers le bon appareil interne

**Nous allons explorer le PAT en profondeur dans la section suivante !**

## Comment fonctionne le NAT en détail

### Scénario : Vous visitez un site web

Prenons l'exemple concret d'un utilisateur qui visite `google.com` depuis son PC.

#### Étape 1 : Création du paquet (réseau interne)

```
Votre PC : 192.168.1.10
Vous tapez : www.google.com dans votre navigateur

Paquet créé :
┌─────────────────────────────────────────┐
│ IP Source      : 192.168.1.10           │
│ Port Source    : 45678 (aléatoire)      │
│ IP Destination : 142.250.185.46 (Google)│
│ Port Dest.     : 443 (HTTPS)            │
└─────────────────────────────────────────┘
```

**Ce paquet ne peut PAS sortir tel quel sur Internet** (IP source privée).

#### Étape 2 : Traversée du routeur NAT

Le paquet arrive à votre box/routeur :

```
┌──────────────────────────────────────────────────┐
│            ROUTEUR NAT (BOX)                     │
│                                                  │
│  1. Reçoit le paquet                             │
│  2. Détecte IP source privée (192.168.1.10)      │
│  3. Crée une entrée dans la table NAT :          │
│                                                  │
│     192.168.1.10:45678 → 203.0.113.45:54321      │
│                                                  │
│  4. MODIFIE le paquet :                          │
│     - IP source : 203.0.113.45 (IP publique box) │
│     - Port source : 54321 (nouveau port NAT)     │
│                                                  │
└──────────────────────────────────────────────────┘
```

#### Étape 3 : Paquet modifié part sur Internet

```
Paquet NAT'é :
┌─────────────────────────────────────────┐
│ IP Source      : 203.0.113.45           │ ← Changé !
│ Port Source    : 54321                  │ ← Changé !
│ IP Destination : 142.250.185.46 (Google)│ ← Inchangé
│ Port Dest.     : 443 (HTTPS)            │ ← Inchangé
└─────────────────────────────────────────┘

Ce paquet peut maintenant voyager sur Internet !
```

#### Étape 4 : Réponse de Google

Google reçoit la requête et répond :

```
Paquet de réponse de Google :
┌─────────────────────────────────────────┐
│ IP Source      : 142.250.185.46 (Google)│
│ Port Source    : 443 (HTTPS)            │
│ IP Destination : 203.0.113.45           │ ← Votre IP publique
│ Port Dest.     : 54321                  │ ← Port NAT
└─────────────────────────────────────────┘
```

#### Étape 5 : NAT inverse (démonstration)

Le routeur reçoit la réponse :

```
┌──────────────────────────────────────────────────┐
│            ROUTEUR NAT (BOX)                     │
│                                                  │
│  1. Reçoit le paquet de réponse                  │
│  2. Regarde la destination : 203.0.113.45:54321  │
│  3. Consulte la table NAT :                      │
│                                                  │
│     54321 → 192.168.1.10:45678                   │
│                                                  │
│  4. MODIFIE le paquet de retour :                │
│     - IP dest. : 192.168.1.10 (IP privée PC)     │
│     - Port dest. : 45678 (port original)         │
│                                                  │
└──────────────────────────────────────────────────┘
```

#### Étape 6 : Réception par votre PC

```
Paquet reçu par votre PC :
┌─────────────────────────────────────────┐
│ IP Source      : 142.250.185.46 (Google)│
│ Port Source    : 443 (HTTPS)            │
│ IP Destination : 192.168.1.10           │ ← Retraduit !
│ Port Dest.     : 45678                  │ ← Port original
└─────────────────────────────────────────┘

Votre navigateur reçoit la réponse de Google !
```

### Visualisation complète du flux

```
   [PC Interne]              [NAT/Box]              [Internet/Google]
   192.168.1.10              203.0.113.45           142.250.185.46
        │                         │                        │
        │ 1. Requête              │                        │
        │ 192.168.1.10:45678 ────→│                        │
        │    → Google:443         │                        │
        │                         │                        │
        │                    2. Traduction NAT             │
        │                    Table: 45678→54321            │
        │                         │                        │
        │                         │ 3. Requête NATée       │
        │                         │ 203.0.113.45:54321 ───→│
        │                         │    → Google:443        │
        │                         │                        │
        │                         │ 4. Réponse             │
        │                         │←─── Google:443         │
        │                         │     → 203.0.113.45:54321│
        │                         │                        │
        │                    5. Traduction inverse         │
        │                    Table: 54321→45678            │
        │                         │                        │
        │ 6. Réponse              │                        │
        │←─── Google:443          │                        │
        │     → 192.168.1.10:45678│                        │
        │                         │                        │
```

## La table de traduction NAT

Le cœur du NAT est la **table de traduction** (NAT table) qui mémorise toutes les correspondances actives.

### Structure d'une table NAT

```
┌─────────────────┬──────────┬────────────────┬──────────┬─────────┬──────────┐
│ IP Interne      │ Port Int │ IP Publique    │ Port Pub │ IP Ext  │ Port Ext │
├─────────────────┼──────────┼────────────────┼──────────┼─────────┼──────────┤
│ 192.168.1.10    │ 45678    │ 203.0.113.45   │ 54321    │ 8.8.8.8 │ 53       │
│ 192.168.1.10    │ 45679    │ 203.0.113.45   │ 54322    │ Google  │ 443      │
│ 192.168.1.20    │ 12345    │ 203.0.113.45   │ 54323    │ Facebook│ 443      │
│ 192.168.1.30    │ 33891    │ 203.0.113.45   │ 54324    │ YouTube │ 443      │
└─────────────────┴──────────┴────────────────┴──────────┴─────────┴──────────┘

Chaque ligne = une connexion active trackée par le NAT
```

### Informations stockées

Pour chaque connexion, le NAT mémorise :

1. **Adresse IP interne** : Qui a initié la connexion ?
2. **Port source interne** : Quel port utilise l'application ?
3. **Adresse IP publique** : L'IP publique utilisée (souvent une seule)
4. **Port public NAT** : Le port alloué pour cette connexion
5. **Adresse IP destination** : Vers qui va la connexion ?
6. **Port destination** : Sur quel service (80, 443, etc.) ?
7. **Timestamp** : Quand a eu lieu la dernière activité ?
8. **État** : Connexion établie, en cours, fermée...

### Durée de vie des entrées

Les entrées NAT ont une **durée de vie limitée** (timeout) :

```
TCP établi    : 2 heures d'inactivité
TCP en cours  : 5 minutes
UDP           : 30 secondes à 5 minutes
ICMP (ping)   : 30 secondes

Après ce délai sans activité, l'entrée est SUPPRIMÉE
```

**Conséquence** : Une connexion inactive trop longtemps sera fermée par le NAT, même si l'application voudrait la garder ouverte.

### Limites de la table

Un routeur NAT a une **capacité limitée** :

```
Box domestique    : 1 000 à 10 000 connexions simultanées
Routeur PME       : 50 000 à 100 000 connexions
Firewall entreprise : 1 000 000+ connexions
```

**Problème** : Si la table est pleine, les nouvelles connexions sont rejetées !

## Le PAT en profondeur

### Pourquoi le PAT est génial

Avec PAT, on peut avoir :

```
Des milliers d'appareils utilisant UNE SEULE IP publique !

Comment ? Grâce aux ports :
  - Ports disponibles : 1024 à 65535 = ~64 000 ports
  - Chaque connexion = un port unique
  - Donc ~64 000 connexions simultanées possibles

En pratique : Largement suffisant pour un foyer ou une PME
```

### Exemple avec 3 appareils simultanés

```
Situation :
- PC de Jean navigue sur Google
- Smartphone de Marie regarde YouTube
- Tablette de Pierre joue en ligne

┌────────────────────────────────────────────────────────────┐
│                    Table PAT/NAT                           │
├────────────────┬──────┬───────────────┬───────┬────────────┤
│ IP Interne     │ Port │ IP Publique   │ Port  │ Destination│
├────────────────┼──────┼───────────────┼───────┼────────────┤
│ 192.168.1.10   │ 1234 │ 203.0.113.45  │ 50001 │ Google:443 │
│ (PC Jean)      │      │               │       │            │
├────────────────┼──────┼───────────────┼───────┼────────────┤
│ 192.168.1.20   │ 5678 │ 203.0.113.45  │ 50002 │ YouTube:443│
│ (Smartphone)   │      │               │       │            │
├────────────────┼──────┼───────────────┼───────┼────────────┤
│ 192.168.1.30   │ 9999 │ 203.0.113.45  │ 50003 │ Game:3074  │
│ (Tablette)     │      │               │       │            │
└────────────────┴──────┴───────────────┴───────┴────────────┘

Même IP publique (203.0.113.45) mais ports différents !
Les réponses sont distribuées au bon appareil grâce au port.
```

### Schéma PAT détaillé

```
RÉSEAU INTERNE              ROUTEUR PAT           INTERNET

PC Jean                     Table PAT             Google
192.168.1.10:1234 ─────────→ :50001 ─────────────→ :443
                            │
Smartphone Marie            │                      YouTube
192.168.1.20:5678 ─────────→ :50002 ─────────────→ :443
                            │
Tablette Pierre             │                      Serveur jeu
192.168.1.30:9999 ─────────→ :50003 ─────────────→ :3074

       Tous utilisent : 203.0.113.45
       Mais avec des PORTS différents
```

### Algorithme de sélection de port

Le NAT choisit un port public de différentes manières :

**1. Port preservation** (préservation de port) :

```
Si le port source interne est disponible en externe,
le garder tel quel.

Exemple :
  Interne : 192.168.1.10:54321
  Externe : 203.0.113.45:54321 (même port !)
```

**2. Port alloué dynamiquement** :

```
Si le port est déjà utilisé, en choisir un autre :

  Interne : 192.168.1.10:54321
  Externe : 203.0.113.45:54321 (déjà pris !)
         → 203.0.113.45:54322 (port suivant libre)
```

**3. Plages de ports** :

Certains NAT réservent des plages :

```
Ports 1024-19999   : TCP
Ports 20000-39999  : UDP
Ports 40000-65535  : ICMP et autres
```

## Avantages du NAT

### 1. Conservation des adresses IPv4

**Impact mondial** :

```
Sans NAT :
  10 milliards d'appareils × 1 IP publique chacun
  = 10 milliards d'adresses nécessaires
  > 4,3 milliards disponibles
  → IMPOSSIBLE ❌

Avec NAT :
  10 milliards d'appareils partagent ~4 milliards d'IPs
  = POSSIBLE ✅

Le NAT a littéralement sauvé Internet IPv4 !
```

### 2. Sécurité par obscurité

```
Depuis Internet :
  ❌ Impossible de scanner directement 192.168.1.10
  ❌ Impossible de se connecter directement
  ❌ La structure interne est invisible

C'est comme un mur entre Internet et votre réseau interne
```

**Attention** : Ce n'est PAS une vraie sécurité ! Le NAT n'est pas un firewall, mais il offre une couche de protection basique.

### 3. Flexibilité du réseau interne

```
Vous pouvez :
  ✅ Changer votre plan d'adressage interne
  ✅ Ajouter/supprimer des appareils
  ✅ Changer de FAI

Sans toucher à vos configurations internes !
```

### 4. Économies

```
Coût d'une IP publique : ~5-50€/mois

Entreprise avec 100 employés :
  Sans NAT : 100 IPs × 20€ = 2000€/mois
  Avec NAT : 1 IP × 20€ = 20€/mois

Économie : 1980€/mois = 23 760€/an !
```

## Inconvénients et problèmes du NAT

### 1. Violation du modèle end-to-end

Le principe original d'Internet : **end-to-end connectivity**

```
Modèle original (IPv6 futur) :
  Appareil A ←──────────────→ Appareil B
      Communication directe, aucun intermédiaire

Avec NAT (IPv4 actuel) :
  Appareil A ←─→ NAT ←─→ Internet ←─→ NAT ←─→ Appareil B
      Les paquets sont modifiés en route
```

**Conséquence** : Certains protocoles cassent.

### 2. Problèmes avec certaines applications

**Applications qui intègrent des IPs dans les données** :

```
Protocoles affectés :
  ❌ FTP (mode actif)
  ❌ SIP (VoIP)
  ❌ H.323 (visioconférence)
  ❌ Certains jeux P2P
  ❌ BitTorrent (partiellement)
```

**Pourquoi ?** Ces protocoles envoient des adresses IP dans le contenu des paquets, pas seulement dans les en-têtes. Le NAT ne modifie que les en-têtes !

**Exemple FTP** :

```
Commande FTP PORT :
  "PORT 192,168,1,10,156,143"

Cette commande dit au serveur FTP :
  "Connecte-toi à 192.168.1.10 port 40079"

Problème : Le serveur FTP ne peut PAS atteindre 192.168.1.10 !
          C'est une adresse privée invisible sur Internet.
```

**Solutions** :
- ALG (Application Layer Gateway) : le NAT comprend certains protocoles
- Mode passif pour FTP
- STUN/TURN pour SIP
- UPnP pour les jeux

### 3. Connexions entrantes impossibles

**Problème fondamental** :

```
Depuis Internet, comment joindre 192.168.1.10 ?

  ping 192.168.1.10 ❌ Impossible
  ssh 192.168.1.10  ❌ Impossible

L'adresse privée n'existe pas sur Internet !
```

**Conséquence** :
- Impossible d'héberger un serveur facilement
- Problèmes pour les jeux en ligne (NAT strict)
- Complexité pour le P2P

**Solutions** : Port forwarding (voir section suivante).

### 4. Performance et latence

Le NAT ajoute du **traitement** :

```
Chaque paquet doit :
  1. Être analysé
  2. Table NAT consultée/mise à jour
  3. En-tête IP modifié
  4. Checksum recalculé

Surcoût : ~0,1 à 1 ms (négligeable pour le surf, impactant pour le gaming)
```

### 5. Logging et traçabilité

```
Problème : 1000 utilisateurs derrière la même IP publique

Logs d'un serveur web :
  203.0.113.45 - - [01/Dec/2025:14:30:15] "GET /page.html"
  203.0.113.45 - - [01/Dec/2025:14:30:16] "POST /form"
  203.0.113.45 - - [01/Dec/2025:14:30:17] "GET /image.jpg"

Impossible de savoir quel utilisateur a fait quoi !
```

**Solution** : Le NAT doit aussi logger avec les ports, mais c'est rare.

### 6. Limites du nombre de connexions

```
Limite théorique : ~64 000 ports disponibles
Limite pratique :
  - Timeouts qui libèrent des ports
  - Mais les applis modernes ouvrent beaucoup de connexions

Problème si :
  - Trop d'utilisateurs simultanés
  - Applications très bavardes (centaines de connexions par app)
```

## Port Forwarding (redirection de ports)

### Le problème à résoudre

```
Vous hébergez un serveur web sur 192.168.1.50

Depuis Internet :
  http://203.0.113.45 → Où envoyer ce trafic ???

Le NAT ne sait pas vers quel appareil interne rediriger !
```

### La solution : Port Forwarding

**Configuration manuelle** sur le routeur :

```
Règle de port forwarding :
  "Tout trafic arrivant sur 203.0.113.45:80
   doit être redirigé vers 192.168.1.50:80"
```

### Exemple pratique

**Configuration typique** :

```
┌─────────────────────────────────────────────────────────┐
│         Configuration Box Internet                      │
├─────────────────────────────────────────────────────────┤
│  Port externe : 80                                      │
│  Protocole    : TCP                                     │
│  IP interne   : 192.168.1.50                            │
│  Port interne : 80                                      │
│  Description  : Serveur Web                             │
└─────────────────────────────────────────────────────────┘
```

**Résultat** :

```
Utilisateur Internet tape : http://203.0.113.45

1. Paquet arrive : 203.0.113.45:80
2. Box consulte les règles de port forwarding
3. Trouve la règle : port 80 → 192.168.1.50:80
4. Modifie le paquet :
   Destination : 192.168.1.50:80
5. Transmet au serveur web interne

Le serveur web répond, et le NAT fait la traduction inverse.
```

### Schéma complet

```
   INTERNET                  BOX/NAT                 RÉSEAU INTERNE

   Utilisateur           Port Forwarding         Serveur Web
   quelconque               Config:               192.168.1.50:80
                        80 → 192.168.1.50:80
       │                      │                        │
       │ 1. Requête HTTP      │                        │
       │ → 203.0.113.45:80 ──→│                        │
       │                      │                        │
       │                 2. Consulte règles            │
       │                 3. Traduit destination        │
       │                      │                        │
       │                      │ 4. Paquet modifié      │
       │                      │ → 192.168.1.50:80 ────→│
       │                      │                        │
       │                      │ 5. Réponse             │
       │                      │←── 192.168.1.50:80     │
       │                      │                        │
       │                 6. Traduit source             │
       │                      │                        │
       │ 7. Réponse           │                        │
       │←── 203.0.113.45:80   │                        │
       │                      │                        │
```

### Exemples de règles courantes

```
┌─────────────┬──────────┬─────────────┬────────────┬─────────────────┐
│ Service     │ Protocole│ Port Externe│ IP Interne │ Port Interne    │
├─────────────┼──────────┼─────────────┼────────────┼─────────────────┤
│ HTTP        │ TCP      │ 80          │ 192.168.1.50│ 80             │
│ HTTPS       │ TCP      │ 443         │ 192.168.1.50│ 443            │
│ SSH         │ TCP      │ 22          │ 192.168.1.10│ 22             │
│ FTP         │ TCP      │ 21          │ 192.168.1.60│ 21             │
│ Minecraft   │ TCP      │ 25565       │ 192.168.1.20│ 25565          │
│ RDP         │ TCP      │ 3389        │ 192.168.1.11│ 3389           │
└─────────────┴──────────┴─────────────┴────────────┴─────────────────┘
```

### Port Forwarding vs DMZ

**DMZ (DeMilitarized Zone)** : Tous les ports sont forwardés vers un appareil.

```
Port Forwarding :
  Règle sélective : port 80 → serveur web

DMZ :
  Règle globale : TOUS les ports → 192.168.1.50

⚠️ DMZ est moins sécurisé (l'appareil est totalement exposé)
```

## UPnP : automatisation du port forwarding

### Qu'est-ce qu'UPnP ?

**UPnP (Universal Plug and Play)** permet aux applications de **configurer automatiquement** le NAT sans intervention manuelle.

### Fonctionnement

```
1. Application (ex: jeu vidéo) démarre
2. Détecte qu'elle est derrière un NAT
3. Envoie une requête UPnP à la box :
   "J'ai besoin du port 3074 TCP"
4. La box crée automatiquement la règle de forwarding
5. L'application peut recevoir des connexions entrantes
6. Quand l'appli ferme, la règle est supprimée
```

### Avantages

```
✅ Pas de configuration manuelle
✅ Pratique pour les jeux et applications P2P
✅ Règles temporaires (nettoyées automatiquement)
```

### Inconvénients

```
❌ Risque de sécurité (n'importe quelle appli peut ouvrir des ports)
❌ Peut être exploité par des malwares
❌ Souvent désactivé par défaut sur les box pro
```

**Conseil** : Désactiver UPnP si vous n'en avez pas besoin (sécurité).

## NAT Traversal : techniques pour contourner le NAT

### Le problème du P2P

```
Deux utilisateurs veulent communiquer directement :

Alice (192.168.1.10) ←─── NAT ───→ Internet ←─── NAT ───→ Bob (192.168.0.20)

Comment établir une connexion directe ?
Les deux sont derrière des NAT !
```

### Technique 1 : STUN (Session Traversal Utilities for NAT)

**Principe** : Découvrir votre IP publique et port externe.

```
1. Alice contacte un serveur STUN public
2. Le serveur STUN répond : "Je te vois depuis 203.0.113.45:54321"
3. Alice donne cette info à Bob
4. Bob tente de se connecter à 203.0.113.45:54321
5. Si le NAT d'Alice est "permissif", ça fonctionne !
```

**Limites** : Ne fonctionne pas avec tous les types de NAT.

### Technique 2 : TURN (Traversal Using Relays around NAT)

**Principe** : Utiliser un serveur relais si STUN échoue.

```
Si connexion directe impossible :

Alice ←─→ Serveur TURN ←─→ Bob

Le serveur TURN relaie tous les paquets.
Moins efficace mais toujours fonctionnel.
```

### Technique 3 : ICE (Interactive Connectivity Establishment)

**Principe** : Combiner plusieurs techniques intelligemment.

```
ICE essaie dans l'ordre :
  1. Connexion directe (si pas de NAT)
  2. STUN (connexion directe via NAT)
  3. TURN (relais en dernier recours)

Utilisé par : WebRTC, Skype, Discord, etc.
```

### Les types de NAT et leur "perméabilité"

```
┌──────────────────────┬─────────────────────────────────────┐
│ Type de NAT          │ Difficulté P2P                      │
├──────────────────────┼─────────────────────────────────────┤
│ Full Cone NAT        │ Facile (très permissif)             │
│ Restricted Cone NAT  │ Modéré                              │
│ Port Restricted NAT  │ Difficile                           │
│ Symmetric NAT        │ Très difficile (TURN obligatoire)   │
└──────────────────────┴─────────────────────────────────────┘
```

**Note** : Les box grand public sont généralement "Port Restricted NAT".

## Cas pratiques et diagnostics

### Diagnostic 1 : "Je ne peux pas héberger de serveur"

```
Symptôme : Serveur web lancé sur 192.168.1.10:80
          mais personne ne peut y accéder depuis Internet

Causes possibles :
  1. Pas de port forwarding configuré ❌
  2. Firewall qui bloque ❌
  3. FAI bloque le port 80 ❌

Solutions :
  1. Configurer port forwarding 80 → 192.168.1.10:80
  2. Vérifier le firewall (iptables, Windows Firewall)
  3. Essayer un autre port (8080) ou utiliser un VPS
```

### Diagnostic 2 : "NAT strict" dans les jeux

```
Symptôme : Console de jeu affiche "NAT strict"
          Impossible de rejoindre certains matchs

Causes :
  - UPnP désactivé
  - Ports de jeu non forwardés
  - NAT très restrictif

Solutions :
  1. Activer UPnP sur la box
  2. Configurer manuellement les ports du jeu
  3. Mettre la console en DMZ (moins sécurisé)
```

### Diagnostic 3 : "Ma connexion VoIP coupe"

```
Symptôme : Appels VoIP/SIP coupés après quelques minutes

Cause : Timeout NAT trop court
  - NAT supprime l'entrée après 30s d'inactivité
  - Mais l'appel est toujours en cours !

Solution : Keep-alive (envoyer des paquets régulièrement)
  - Configure dans l'appli VoIP : "keep-alive every 20s"
```

## Configuration NAT : exemple sur différents équipements

### Sur une box grand public (interface web)

```
1. Se connecter à la box : http://192.168.1.1
2. Identifiants : admin / [mot de passe de la box]
3. Menu : "Configuration avancée" > "NAT/PAT"
4. Ajouter une règle :
   - Port externe : 80
   - IP interne : 192.168.1.50
   - Port interne : 80
   - Protocole : TCP
5. Sauvegarder
```

### Sur un routeur Cisco (ligne de commande)

```cisco
! NAT statique
ip nat inside source static 192.168.1.50 203.0.113.45

! PAT (NAT overload)
ip nat inside source list 1 interface GigabitEthernet0/0 overload

! ACL pour définir les IPs à NATer
access-list 1 permit 192.168.1.0 0.0.0.255

! Interfaces
interface GigabitEthernet0/0
 ip nat outside
interface GigabitEthernet0/1
 ip nat inside
```

### Sur Linux (iptables)

```bash
# Activer le forwarding IP
echo 1 > /proc/sys/net/ipv4/ip_forward

# NAT/PAT (masquerading)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Port forwarding (exemple port 80)
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
  -j DNAT --to-destination 192.168.1.50:80
```

## Résumé visuel complet

```
┌───────────────────────────────────────────────────────────────┐
│                  NAT/PAT - CONCEPTS CLÉS                      │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  🎯 But : Partager une (ou quelques) IP(s) publique(s)        │
│          entre de nombreux appareils privés                   │
│                                                               │
│  📊 Types de NAT :                                            │
│     • Statique : 1 privée ↔ 1 publique (fixe)                 │
│     • Dynamique : Pool d'IPs publiques partagé                │
│     • PAT/Overload : 1 IP publique pour tous (via ports)      │
│                                                               │
│  🔄 Fonctionnement :                                          │
│     1. Paquet sort : IP privée → IP publique                  │
│     2. Table NAT mémorise la correspondance                   │
│     3. Réponse arrive : consultation table                    │
│     4. Paquet entre : IP publique → IP privée                 │
│                                                               │
│  ➕ Avantages :                                               │
│     • Économise les IPs publiques                             │
│     • Sécurité basique (masquage)                             │
│     • Flexibilité réseau interne                              │
│                                                               │
│  ➖ Inconvénients :                                           │
│     • Connexions entrantes difficiles                         │
│     • Certaines applis cassent                                │
│     • Viole le modèle end-to-end                              │
│                                                               │
│  🔧 Solutions :                                               │
│     • Port Forwarding : règles manuelles                      │
│     • UPnP : automatique                                      │
│     • STUN/TURN : traversée NAT pour P2P                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ **NAT traduit** les adresses IP privées en adresses publiques

✅ **PAT (NAT Overload)** permet à des milliers d'appareils de partager **1 seule IP** publique

✅ Le NAT utilise une **table de traduction** pour mémoriser les correspondances

✅ Les **ports** sont la clé : chaque connexion = un port unique

✅ Le NAT **modifie les paquets** (IP source, ports, checksums)

✅ **Port Forwarding** permet les connexions entrantes vers un serveur interne

✅ **UPnP** automatise le port forwarding (pratique mais risque de sécurité)

✅ Le NAT a **sauvé IPv4** en permettant de réutiliser les adresses privées

✅ Certaines applications (FTP, SIP, jeux P2P) ont des **problèmes avec le NAT**

✅ **NAT Traversal** (STUN/TURN/ICE) permet de contourner certaines limitations

## Pour aller plus loin

Maintenant que vous maîtrisez le NAT, vous êtes prêt à explorer :

- **IPv6** : comment il élimine le besoin de NAT
- **ICMP** : protocole de diagnostic réseau (ping, traceroute)
- **Routage** : comment les paquets trouvent leur chemin
- **Firewalls** : sécurité réseau avancée

---

**💡 Expérience pratique** :
1. Connectez-vous à l'interface de votre box (souvent http://192.168.1.1)
2. Cherchez la section "NAT" ou "Redirection de ports"
3. Regardez si des règles existent déjà (créées par UPnP)
4. Essayez de créer une règle de port forwarding (n'oubliez pas de la supprimer après !)

---

*Dans la section suivante, nous allons explorer IPv6, le successeur d'IPv4 qui rend le NAT optionnel grâce à son espace d'adressage quasi-illimité...*

⏭️ [IPv6 : pourquoi, comment, structure des adresses](/03-couche-internet/07-ipv6.md)
