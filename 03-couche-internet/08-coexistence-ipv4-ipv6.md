🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.8 Coexistence IPv4/IPv6 : dual-stack, tunneling

## Introduction : la longue transition

Imaginez que le monde entier décide de changer de langue du jour au lendemain. Impossible, n'est-ce pas ? Même avec la meilleure volonté, il faudrait des décennies pour que tout le monde apprenne la nouvelle langue, et pendant ce temps, les deux langues devraient coexister.

C'est exactement ce qui se passe avec **IPv4 et IPv6**. Bien qu'IPv6 soit techniquement supérieur et nécessaire pour l'avenir d'Internet, on ne peut pas simplement "éteindre" IPv4 et "allumer" IPv6. Les deux protocoles doivent **coexister** pendant une **longue période de transition**.

**Dates clés** :
```
1998 : Spécification IPv6 publiée
2011 : Épuisement des adresses IPv4
2025 : ~40-45% du trafic Internet en IPv6
20?? : Transition complète (encore loin !)

Durée de transition estimée : 20 à 30 ans (voire plus)
```

## Le défi de la transition

### Pourquoi c'est si difficile ?

**1. Incompatibilité totale**

IPv4 et IPv6 sont **complètement incompatibles** :

```
❌ Un appareil IPv4-seulement NE PEUT PAS communiquer
   directement avec un appareil IPv6-seulement

C'est comme essayer de parler français à quelqu'un
qui ne parle que chinois : impossible sans traducteur !
```

**2. L'effet réseau**

```
Pour que IPv6 soit utile, il faut que :
  ✅ Votre FAI le supporte
  ✅ Les sites web l'activent
  ✅ Votre box/routeur le gère
  ✅ Votre OS l'implémente
  ✅ Vos applications le comprennent

Si UN SEUL maillon manque, vous retombez sur IPv4 !
```

**Analogie** : C'est comme adopter une nouvelle monnaie. Elle n'est utile que si tout le monde l'accepte : les banques, les magasins, les particuliers...

**3. Coût et complexité**

```
Migration complète nécessite :
  💰 Remplacement/mise à jour d'équipements
  📚 Formation des équipes réseau
  🔧 Configuration de la coexistence
  🐛 Tests et debugging
  📝 Documentation mise à jour

Coût estimé : Milliards d'euros à l'échelle mondiale
```

### Le problème "chicken and egg"

```
Fournisseurs de contenu (Google, Facebook...) :
  "On activera IPv6 quand les utilisateurs l'auront"

FAI (Orange, Free, Bouygues...) :
  "On déploiera IPv6 quand il y aura du contenu"

Utilisateurs :
  "On demandera IPv6 quand il y aura un intérêt"

→ Cercle vicieux ! 🔄
```

**Solution** : Adoption progressive par les acteurs majeurs (Google, Facebook, Netflix ont été pionniers).

## Les trois stratégies de coexistence

Il existe **trois approches principales** pour faire cohabiter IPv4 et IPv6 :

```
┌──────────────────────────────────────────────────────┐
│                STRATÉGIES DE COEXISTENCE             │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1️⃣ DUAL-STACK                                       │
│     Les deux protocoles actifs simultanément         │
│     → Le plus courant aujourd'hui                    │
│                                                      │
│  2️⃣ TUNNELING                                        │
│     Encapsuler IPv6 dans IPv4 (ou inversement)       │
│     → Pour traverser des réseaux incompatibles       │
│                                                      │
│  3️⃣ TRANSLATION                                      │
│     Traduire entre IPv4 et IPv6 (NAT64, DNS64)       │
│     → Quand la communication directe est impossible  │
│                                                      │
└──────────────────────────────────────────────────────┘
```

## 1. Dual-Stack : parler deux langues

### Principe fondamental

**Dual-Stack** = Un appareil possède **à la fois** une adresse IPv4 **et** une adresse IPv6.

```
┌─────────────────────────────────────┐
│      Appareil Dual-Stack            │
├─────────────────────────────────────┤
│                                     │
│  Pile IPv4 : 192.168.1.10           │
│  Pile IPv6 : 2001:db8::1            │
│                                     │
│  Peut communiquer en IPv4 OU IPv6   │
│  selon la destination               │
│                                     │
└─────────────────────────────────────┘
```

**Analogie** : C'est comme être **bilingue français-anglais**. Vous parlez français avec les francophones, anglais avec les anglophones, selon la personne en face.

### Comment ça fonctionne ?

**Étape 1 : Configuration**

L'appareil obtient deux adresses :

```
Configuration réseau :
  IPv4 : 192.168.1.10/24
  Gateway IPv4 : 192.168.1.1
  DNS IPv4 : 8.8.8.8

  IPv6 : 2001:db8:1234:5678::10/64
  Gateway IPv6 : 2001:db8:1234:5678::1
  DNS IPv6 : 2001:4860:4860::8888

Les deux piles réseau sont actives simultanément !
```

**Étape 2 : Résolution DNS**

Quand vous tapez `www.google.com` :

```
Requête DNS envoyée :
  1. Demande enregistrement A (IPv4)
  2. Demande enregistrement AAAA (IPv6)

Réponse du serveur DNS :
  A    : 142.250.185.46    (IPv4)
  AAAA : 2a00:1450:4007:819::200e (IPv6)

→ L'appareil a maintenant DEUX adresses pour Google !
```

**Étape 3 : Sélection du protocole**

L'OS choisit quelle version utiliser selon un **algorithme de préférence**.

```
RFC 6724 - Default Address Selection

Ordre de préférence (simplifié) :
  1. IPv6 natif (si disponible) ✅
  2. IPv4
  3. IPv6 tunnelé (6to4, Teredo...)

Résultat : IPv6 est PRÉFÉRÉ quand disponible
```

**Étape 4 : Communication**

```
Scénario A : Destination a IPv6
  → Communication en IPv6 ✅

Scénario B : Destination n'a que IPv4
  → Communication en IPv4 ✅

Scénario C : Destination a les deux
  → Communication en IPv6 (préférence) ✅
```

### Schéma Dual-Stack complet

```
     Utilisateur                  Internet
   (Dual-Stack)

┌──────────────────┐           ┌──────────────────┐
│  Votre PC        │           │  Google          │
│                  │           │                  │
│ IPv4: 192.168.1.10│          │ IPv4: 142.250... │
│ IPv6: 2001:db8::10│          │ IPv6: 2a00:1450..│
└────────┬─────────┘           └────────┬─────────┘
         │                              │
         │  Résolution DNS :            │
         │  google.com ?                │
         │  → A: 142.250.185.46         │
         │  → AAAA: 2a00:1450:...       │
         │                              │
         │  Décision : Utiliser IPv6    │
         │                              │
         │══════════════════════════════│
         │   Connexion IPv6 établie     │
         │══════════════════════════════│
         │                              │
```

### Avantages du Dual-Stack

```
✅ Solution la plus simple et élégante
✅ Compatibilité totale avec IPv4 existant
✅ Permet une transition en douceur
✅ Pas besoin de traduction ou tunneling complexe
✅ Performance optimale (pas de surcharge)
✅ Préférence automatique pour IPv6
```

### Inconvénients du Dual-Stack

```
❌ Nécessite DEUX adresses IP (IPv4 + IPv6)
❌ Double configuration à gérer
❌ Double sécurité à configurer (firewall IPv4 ET IPv6)
❌ Consomme plus de mémoire et ressources
❌ Complexité de troubleshooting (quel protocole pose problème ?)
❌ Nécessite IPv4 disponible (problème si pénurie)
```

### Configuration Dual-Stack

**Sur Linux** :

```bash
# Vérifier la configuration actuelle
ip addr show

# Exemple de résultat dual-stack :
eth0: <BROADCAST,MULTICAST,UP>
    inet 192.168.1.10/24 brd 192.168.1.255 scope global eth0
    inet6 2001:db8:1234:5678::10/64 scope global
    inet6 fe80::a00:27ff:fe4e:66a1/64 scope link
```

**Sur Windows** :

```powershell
ipconfig

# Résultat dual-stack :
Carte Ethernet Ethernet :
   Adresse IPv4. . . . . . . . : 192.168.1.10
   Masque de sous-réseau. . . . : 255.255.255.0
   Passerelle par défaut. . . . : 192.168.1.1

   Adresse IPv6. . . . . . . . : 2001:db8:1234:5678::10
   Passerelle par défaut IPv6. : 2001:db8:1234:5678::1
```

### Happy Eyeballs (RFC 8305)

Pour améliorer l'expérience utilisateur, les navigateurs modernes utilisent **Happy Eyeballs** :

```
Algorithme :
  1. Lancer connexion IPv6
  2. Attendre 50-300ms
  3. Si IPv6 pas encore connecté, lancer AUSSI IPv4
  4. Utiliser la PREMIÈRE connexion qui réussit
  5. Annuler l'autre

Résultat : Rapidité maximale, fallback automatique si problème
```

**Exemple** :

```
t=0ms    : Début connexion IPv6
t=50ms   : Début connexion IPv4 (fallback)
t=80ms   : IPv6 connecté ! ✅
t=81ms   : Annulation de la tentative IPv4
           → Utilisation d'IPv6

Si IPv6 avait échoué :
t=120ms  : IPv4 connecté ! ✅
           → Utilisation d'IPv4 (fallback)
```

## 2. Tunneling : le passage secret

### Principe du tunneling

**Tunneling** = Encapsuler des paquets IPv6 **dans** des paquets IPv4 (ou inversement) pour traverser un réseau incompatible.

**Analogie** : C'est comme envoyer une lettre en chinois à travers la France. Vous mettez la lettre chinoise **dans une enveloppe française** avec une adresse française. La Poste française la transporte, puis à destination, on extrait la lettre chinoise de l'enveloppe.

```
┌─────────────────────────────────────────────────┐
│  Paquet IPv6 (inaccessible sur réseau IPv4)     │
│  ↓ ENCAPSULATION ↓                              │
│  ┌───────────────────────────────────────┐      │
│  │ En-tête IPv4                          │      │
│  ├───────────────────────────────────────┤      │
│  │ ┌─────────────────────────────────┐   │      │
│  │ │ En-tête IPv6                    │   │      │
│  │ ├─────────────────────────────────┤   │      │
│  │ │ Données                         │   │      │
│  │ └─────────────────────────────────┘   │      │
│  └───────────────────────────────────────┘      │
│  Ce paquet peut voyager sur réseau IPv4 !       │
└─────────────────────────────────────────────────┘
```

### Schéma de tunneling

```
Réseau IPv6     Tunnel IPv4     Réseau IPv6
  (Bureau)      (Internet)      (Serveur)

┌──────────┐                    ┌──────────┐
│  Client  │                    │  Serveur │
│  IPv6    │                    │  IPv6    │
└────┬─────┘                    └────┬─────┘
     │ Paquet IPv6                   │
     │ 2001:db8::1                   │
     │ → 2001:db8::100               │
     ↓                               ↑
┌────────────┐                ┌────────────┐
│ Routeur A  │                │ Routeur B  │
│ (Entrée)   │                │ (Sortie)   │
└────┬───────┘                └────┬───────┘
     │                             │
     │ Encapsulation               │ Décapsulation
     │                             │
     │ Paquet IPv4                 │
     │ 203.0.113.1                 │
     │ → 203.0.114.1               │
     │                             │
     └─────────── INTERNET ────────┘
          (IPv4 seulement)
```

### Types de tunnels IPv6

#### 1. Tunnel configuré manuellement (6in4)

**Caractéristiques** :
- Configuration **statique** des deux extrémités
- Tunnel **point-à-point**
- Nécessite une **adresse IPv4 publique** aux deux bouts

```
Configuration exemple :
  Routeur A (entrée) : 203.0.113.1
  Routeur B (sortie) : 203.0.114.1

  Tunnel IPv6 : 2001:db8:1::/64
```

**Avantages** :
```
✅ Simple et fiable
✅ Performance stable
✅ Contrôle total
```

**Inconvénients** :
```
❌ Configuration manuelle requise
❌ Pas scalable (un tunnel par site)
❌ Nécessite une IP publique fixe
```

**Cas d'usage** : Connexion entre deux sites d'entreprise.

#### 2. 6to4 (RFC 3056)

**Principe** : Tunneling **automatique** basé sur les adresses IPv4.

**Magie** : L'adresse IPv6 est **dérivée** de l'adresse IPv4 !

```
Votre IP publique IPv4 : 203.0.113.1

Conversion en hexadécimal :
  203 → CB
  0   → 00
  113 → 71
  1   → 01

Adresse IPv6 6to4 :
  2002:cb00:7101::/48
  └──┘ └────────┘
  préfixe  IPv4 en hex
  6to4
```

**Fonctionnement** :

```
1. Votre routeur a une IP publique : 203.0.113.1
2. Il génère automatiquement : 2002:cb00:7101::/48
3. Tout paquet vers 2002::/16 est tunnelé via 6to4
4. Relais 6to4 publics (ex: 192.88.99.1) font la passerelle
```

**Avantages** :
```
✅ Configuration automatique
✅ Pas besoin de tunnel manuel
✅ Gratuit et ouvert
```

**Inconvénients** :
```
❌ Performance variable (relais publics aléatoires)
❌ Obsolète et déprécié (RFC 7526)
❌ Problèmes de fiabilité
❌ Nécessite une IP publique
```

**Statut** : ⚠️ **OBSOLÈTE - Ne plus utiliser !**

#### 3. Teredo (RFC 4380)

**Principe** : Tunneling IPv6 **même derrière un NAT** !

**Innovation** : Utilise **UDP sur le port 3544** pour traverser les NAT.

```
Situation :
  Client derrière NAT IPv4
  ↓
  NAT (pas d'IP publique directe)
  ↓
  Internet IPv4
  ↓
  Serveur Teredo
  ↓
  Internet IPv6
```

**Format d'adresse Teredo** :

```
2001:0000:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
└───┬───┘ └──┬──┘
  Préfixe  Serveur
  Teredo   Teredo

Contient :
  - L'adresse du serveur Teredo
  - L'IP publique du NAT (obscurcie)
  - Le port UDP externe
  - Des flags
```

**Avantages** :
```
✅ Fonctionne derrière NAT
✅ Pas besoin d'IP publique
✅ Dernier recours quand rien d'autre ne marche
```

**Inconvénients** :
```
❌ Performance médiocre (tunnel UDP)
❌ Latence élevée
❌ Fiabilité limitée
❌ Complexité de configuration
```

**Statut** : Largement remplacé par des solutions natives, mais encore présent dans Windows.

#### 4. ISATAP (RFC 5214)

**Principe** : Tunneling IPv6 sur IPv4 **au sein d'un réseau d'entreprise**.

**Usage** : Permet à des îlots IPv6 de communiquer via infrastructure IPv4 existante.

```
Bureau A (IPv6)  ←─→  Backbone IPv4  ←─→  Bureau B (IPv6)
                      (entreprise)
```

**Avantages** :
```
✅ Bon pour migration progressive interne
✅ Pas besoin de reconfigurer le cœur de réseau
```

**Inconvénients** :
```
❌ Complexe à gérer
❌ Limité aux réseaux privés
❌ Solution temporaire
```

#### 5. Tunnel Broker (Hurricane Electric, etc.)

**Principe** : Service tiers qui fournit un **tunnel IPv6 gratuit**.

**Fonctionnement** :

```
1. Inscription sur un service (ex: tunnelbroker.net)
2. Obtention d'un préfixe IPv6 (/64 ou /48)
3. Configuration du tunnel sur votre routeur
4. Accès IPv6 via le tunnel !
```

**Exemple Hurricane Electric** :

```
Vous obtenez :
  - Votre préfixe : 2001:470:1f0a:xxxx::/64
  - Serveur tunnel : 216.66.84.46
  - Configuration à copier dans votre routeur

Résultat : IPv6 fonctionnel même si votre FAI ne le supporte pas !
```

**Avantages** :
```
✅ Gratuit
✅ IPv6 immédiat
✅ Apprendre IPv6
✅ /48 disponible (65536 sous-réseaux !)
```

**Inconvénients** :
```
❌ Performance dépend du service
❌ Configuration technique requise
❌ Latence ajoutée
❌ Solution temporaire (attendant IPv6 natif)
```

### Comparaison des méthodes de tunneling

```
┌──────────────┬─────────┬────────────┬──────────┬────────────┐
│ Méthode      │ Auto?   │ NAT OK?    │ Perf.    │ Status     │
├──────────────┼─────────┼────────────┼──────────┼────────────┤
│ 6in4 manuel  │ Non     │ Non        │ ★★★★★    │ OK         │
│ 6to4         │ Oui     │ Non        │ ★★       │ Obsolète ❌│
│ Teredo       │ Oui     │ Oui        │ ★        │ Déprécié   │
│ ISATAP       │ Oui     │ Réseau LAN │ ★★★      │ Interne    │
│ Tunnel Broker│ Config  │ Non        │ ★★★★     │ Temporaire │
└──────────────┴─────────┴────────────┴──────────┴────────────┘
```

### Tunneling inverse : 4in6

Avec la transition vers IPv6, on peut aussi avoir besoin de **tunneler IPv4 dans IPv6** !

```
Scénario :
  Réseau cœur IPv6-only (moderne)
  Mais besoin d'accès à des services IPv4 legacy

Solution : DS-Lite (Dual-Stack Lite)
  Tunnel IPv4 dans IPv6 jusqu'à un point de sortie
```

## 3. Translation : le traducteur universel

### Le problème à résoudre

```
Situation :
  Client IPv6-only ←─→ Serveur IPv4-only

Impossible de communiquer directement !
Dual-stack ne marche pas (client n'a pas d'IPv4)
Tunnel ne marche pas (serveur ne sait pas tunneler)

Solution : TRADUCTION (comme un interprète)
```

### NAT64 + DNS64

**Principe** : Traduire **dynamiquement** entre IPv6 et IPv4.

#### Fonctionnement de NAT64

```
┌──────────────┐        ┌─────────┐        ┌──────────────┐
│   Client     │        │ NAT64   │        │   Serveur    │
│   IPv6-only  │───────→│         │───────→│   IPv4-only  │
│ 2001:db8::10 │  IPv6  │ Traduit │  IPv4  │ 203.0.113.1  │
└──────────────┘        └─────────┘        └──────────────┘

Le NAT64 :
  1. Reçoit paquet IPv6
  2. Extrait l'IPv4 embarquée
  3. Crée un paquet IPv4
  4. Envoie au serveur IPv4
  5. Reçoit réponse IPv4
  6. Recrée un paquet IPv6
  7. Renvoie au client IPv6
```

#### Fonctionnement de DNS64

Le problème : Comment le client IPv6 connaît-il l'adresse IPv4 du serveur ?

**Solution DNS64** :

```
1. Client demande : "Quelle est l'adresse de example.com ?"

2. DNS64 interroge le DNS normal :
   - Enregistrement AAAA ? Non ❌
   - Enregistrement A ? Oui : 93.184.216.34

3. DNS64 **synthétise** une adresse IPv6 :
   Préfixe NAT64 : 64:ff9b::/96
   + IPv4 embarquée : 93.184.216.34
   = 64:ff9b::5db8:d822

4. Client reçoit : 64:ff9b::5db8:d822
   (Il pense que c'est une vraie adresse IPv6 !)

5. Client envoie le paquet vers cette adresse

6. NAT64 intercepte, voit le préfixe 64:ff9b::
   Extrait l'IPv4 : 93.184.216.34
   Traduit et envoie !
```

**Schéma complet** :

```
Client IPv6                DNS64               NAT64           Serveur IPv4
2001:db8::10                                                  93.184.216.34

    │                         │                   │                │
    │ DNS Query: example.com  │                   │                │
    │ (AAAA record?)          │                   │                │
    │────────────────────────→│                   │                │
    │                         │ Forward query     │                │
    │                         │ (no AAAA, A=93...)│                │
    │                         │                   │                │
    │ DNS Reply:              │                   │                │
    │ 64:ff9b::5db8:d822      │                   │                │
    │←────────────────────────│                   │                │
    │                         │                   │                │
    │ HTTP GET                │                   │                │
    │ Dest: 64:ff9b::5db8:d822│                   │                │
    │─────────────────────────┼──────────────────→│                │
    │                         │                   │ Translate      │
    │                         │                   │ Dest: 93.184...│
    │                         │                   │───────────────→│
    │                         │                   │                │
    │                         │                   │←───────────────│
    │                         │                   │ HTTP Response  │
    │←────────────────────────┼───────────────────│                │
    │ HTTP Response (IPv6)    │                   │                │
    │                         │                   │                │
```

### Avantages de NAT64/DNS64

```
✅ Permet la communication IPv6 ↔ IPv4
✅ Transparent pour l'utilisateur
✅ Le client peut être IPv6-only (économise une IPv4)
✅ Solution de transition élégante
```

### Inconvénients

```
❌ Ne fonctionne pas avec toutes les applications
❌ Applications qui embarquent des IPs cassent
❌ Performance (traduction = overhead)
❌ Complexité de déploiement
❌ Logs difficiles (adresses traduites)
```

### Autres méthodes de translation

**464XLAT** : Combinaison de traduction + tunneling
```
Client IPv4 → Traduction → IPv6 réseau → Traduction → Serveur IPv4

Utilisé par : Opérateurs mobiles (4G/5G)
```

**MAP-T/MAP-E** : Translation/Encapsulation avec mapping
```
Permet IPv4-over-IPv6 de manière scalable
Utilisé par : Certains FAI en transition
```

## État actuel du déploiement IPv6

### Statistiques mondiales (2025)

```
Déploiement global IPv6 :
  ~40-45% du trafic Internet
  ~35-40% des utilisateurs

Par région :
  🇧🇪 Belgique    : ~60%  ★★★★★
  🇮🇳 Inde        : ~55%  ★★★★★
  🇩🇪 Allemagne   : ~55%  ★★★★★
  🇺🇸 USA         : ~45%  ★★★★☆
  🇫🇷 France      : ~40%  ★★★★☆
  🇨🇳 Chine       : ~15%  ★★☆☆☆
  🇷🇺 Russie      : ~5%   ★☆☆☆☆

Croissance : +5-10% par an
```

### Par type d'acteur

```
┌────────────────────┬─────────────────────────────────┐
│ Acteur             │ Adoption IPv6                   │
├────────────────────┼─────────────────────────────────┤
│ GAFAM              │ ★★★★★ (95%+)                    │
│ FAI mobiles        │ ★★★★★ (80%+)                    │
│ FAI fixes          │ ★★★★☆ (60-70%)                  │
│ Hébergeurs cloud   │ ★★★★★ (90%+)                    │
│ Sites web top 1000 │ ★★★★☆ (60-70%)                  │
│ Entreprises        │ ★★★☆☆ (30-40%)                  │
│ Équipements réseau │ ★★★★★ (95%+ support)            │
│ OS modernes        │ ★★★★★ (100% support)            │
└────────────────────┴─────────────────────────────────┘
```

### Pionniers IPv6

**Google** : ~40% du trafic en IPv6
```
www.google.com accessible en IPv6 depuis 2008
YouTube, Gmail, Maps... tous compatibles
```

**Facebook/Meta** : ~60% du trafic en IPv6
```
Premier grand réseau social 100% IPv6
A poussé les FAI mobiles à déployer
```

**Netflix** : ~40% du trafic en IPv6
```
Économie de coûts grâce à IPv6
Moins de NAT = meilleure qualité
```

**Cloudflare** : 100% IPv6 ready
```
Tous les sites Cloudflare IPv6-enabled
Service gratuit pour démocratiser IPv6
```

### FAI en France

```
Free :
  ✅ IPv6 natif depuis 2007 (pionnier)
  ✅ Dual-stack sur Freebox
  ✅ /56 pour les abonnés (256 sous-réseaux !)

Orange :
  ✅ IPv6 depuis 2016
  ✅ Dual-stack sur Livebox récentes
  ⚠️ Activation parfois nécessaire

Bouygues :
  ✅ IPv6 en déploiement
  ⚠️ Pas sur toutes les box

SFR :
  ⚠️ Déploiement progressif
  ❌ Pas généralisé
```

## Bonnes pratiques de déploiement

### Pour les administrateurs réseau

**1. Commencer par Dual-Stack**

```
✅ Activer IPv6 EN PLUS d'IPv4 (pas en remplacement)
✅ Tester en interne avant exposition publique
✅ Monitorer les deux protocoles
```

**2. Sécurité dès le départ**

```
⚠️ IPv6 n'est PAS plus sécurisé par défaut !

À faire :
  ✅ Firewall IPv6 (ip6tables, pf, etc.)
  ✅ Désactiver Teredo/6to4 si non nécessaire
  ✅ RA Guard (protection contre rogue RA)
  ✅ Monitoring IPv6 spécifique
```

**3. Plan d'adressage structuré**

```
Préfixe reçu : 2001:db8:1234::/48

Organisation :
  2001:db8:1234:0::/64   : Serveurs
  2001:db8:1234:1::/64   : Employés
  2001:db8:1234:2::/64   : WiFi
  2001:db8:1234:10::/64  : DMZ
  2001:db8:1234:100::/64 : Invités
  ...

Documenter clairement !
```

**4. Tests de connectivité**

```bash
# Tester la connectivité IPv6
ping6 google.com
ping6 2001:4860:4860::8888

# Tester depuis un site externe
http://test-ipv6.com
http://ipv6-test.com
```

### Pour les développeurs

**1. Écrire du code agnostique**

```python
# ❌ Mauvais : Code IPv4-only
socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# ✅ Bon : Code qui marche avec IPv4 et IPv6
socket.create_connection(('example.com', 80))
```

**2. Utiliser des noms de domaine**

```python
# ❌ Mauvais : IP en dur
connect_to('192.0.2.1')

# ✅ Bon : Nom de domaine (résolution DNS gère IPv4/IPv6)
connect_to('api.example.com')
```

**3. Tester les deux protocoles**

```bash
# Tests automatisés doivent couvrir IPv4 et IPv6
pytest tests/ --ipv4
pytest tests/ --ipv6
```

### Pour les utilisateurs

**1. Vérifier si vous avez IPv6**

```
Visitez : https://test-ipv6.com

Résultat possible :
  ✅ "Vous avez IPv6 !" → Parfait
  ⚠️ "IPv6 non détecté" → Normal si FAI ne le fournit pas
  ❌ "IPv6 cassé" → Problème à résoudre
```

**2. Activer IPv6 sur votre box**

```
Interface box → Configuration → IPv6
  [✓] Activer IPv6

Ou contacter le support FAI
```

**3. Désactiver les tunnels obsolètes**

```
Windows : Désactiver Teredo
cmd (admin) > netsh interface teredo set state disabled

Linux : Désactiver 6to4
sysctl -w net.ipv6.conf.all.disable_ipv6_6to4=1
```

## Dépannage de la coexistence

### Problème 1 : IPv6 plus lent qu'IPv4

```
Symptôme : Sites web lents en IPv6, rapides en IPv4

Causes possibles :
  - Routage IPv6 sous-optimal
  - FAI encore en rodage IPv6
  - Tunnel qui ajoute latence

Solutions :
  1. Tester la latence : ping6 vs ping
  2. Traceroute IPv6 : traceroute6 google.com
  3. Désactiver temporairement IPv6 si critique
```

### Problème 2 : Connexions IPv6 échouent

```
Symptôme : Certains sites inaccessibles via IPv6

Causes :
  - Firewall bloque IPv6
  - Configuration IPv6 incomplète
  - Site mal configuré

Diagnostic :
  ping6 google.com → OK ?
  ping6 site-probleme.com → Échec ?

  → Problème côté site, pas chez vous
```

### Problème 3 : IPv6 présent mais pas utilisé

```
Symptôme : J'ai IPv6 mais le trafic passe en IPv4

Vérifier :
  1. DNS retourne AAAA ?
     dig AAAA google.com

  2. Préférence IPv6 active ?
     Linux: cat /proc/sys/net/ipv6/conf/all/use_tempaddr

  3. Application supporte IPv6 ?
     Certaines vieilles apps forcent IPv4
```

## Résumé visuel

```
┌───────────────────────────────────────────────────────────┐
│         COEXISTENCE IPv4/IPv6 - STRATÉGIES                │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  🏆 DUAL-STACK (Recommandé)                               │
│     • Les deux protocoles actifs                          │
│     • Préférence automatique IPv6                         │
│     • Solution la plus élégante                           │
│                                                           │
│  🚇 TUNNELING                                             │
│     • Encapsulation IPv6 dans IPv4                        │
│     • 6in4, 6to4, Teredo, ISATAP...                       │
│     • Plusieurs obsolètes (6to4, Teredo)                  │
│     • Tunnel Broker : solution temporaire                 │
│                                                           │
│  🔄 TRANSLATION                                           │
│     • NAT64 + DNS64                                       │
│     • IPv6-only ↔ IPv4-only                               │
│     • Complexe mais nécessaire                            │
│                                                           │
│  📈 État actuel : ~40% adoption mondiale                  │
│     Transition progressive sur 20-30 ans                  │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ **Transition IPv4→IPv6 prendra des décennies** (coexistence longue)

✅ **Dual-Stack** = solution **préférée** (IPv4 + IPv6 simultanés)

✅ **Happy Eyeballs** : navigateurs testent IPv6 et IPv4 en parallèle

✅ **Tunneling** permet IPv6 sur infrastructure IPv4 (6in4, tunnel broker)

✅ **6to4 et Teredo** sont **obsolètes** (ne plus utiliser)

✅ **NAT64/DNS64** permet communication IPv6-only ↔ IPv4-only

✅ **~40% du trafic Internet** est maintenant en IPv6 (2025)

✅ **IPv6 est PRÉFÉRÉ** quand disponible (meilleure performance)

✅ **Sécurité** : IPv6 nécessite firewall spécifique (pas automatiquement sécurisé)

✅ **Développeurs** : écrire du code agnostique (fonctionne en IPv4 et IPv6)

## Pour aller plus loin

Maintenant que vous comprenez la coexistence IPv4/IPv6, vous êtes prêt pour :

- **ICMPv6** : nouveau protocole de diagnostic pour IPv6
- **Neighbor Discovery** : remplacement d'ARP en IPv6
- **Routage IPv6** : différences avec le routage IPv4
- **Sécurité IPv6** : spécificités et bonnes pratiques

---

**💡 Test pratique** : Visitez https://test-ipv6.com pour voir si vous avez IPv6. Puis comparez les performances :
```bash
# Latence IPv4
ping -c 10 google.com

# Latence IPv6
ping6 -c 10 google.com

Comparez les temps moyens !
```

---

*Dans la section suivante, nous allons explorer ICMP, le protocole qui permet de diagnostiquer les problèmes réseau avec des commandes comme ping et traceroute...*

⏭️ [ICMP : diagnostic et messages de contrôle](/03-couche-internet/09-icmp.md)
