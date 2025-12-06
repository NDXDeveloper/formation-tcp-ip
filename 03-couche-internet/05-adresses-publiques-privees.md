🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.5 Adresses publiques vs privées (RFC 1918)

## Introduction : deux mondes parallèles

Imaginez que vous viviez dans un grand immeuble. À l'intérieur, chaque appartement a un numéro : 101, 102, 103, etc. Ces numéros fonctionnent parfaitement **à l'intérieur de l'immeuble**, mais si vous donnez "Appartement 102" comme adresse à un ami, il ne pourra jamais vous trouver sans l'adresse complète de l'immeuble !

Les adresses IP fonctionnent de la même manière. Il existe deux types d'adresses :

- **Adresses publiques** : Uniques au monde entier, comme une adresse postale complète (rue, ville, pays)
- **Adresses privées** : Réutilisables dans chaque réseau local, comme les numéros d'appartements dans différents immeubles

Cette distinction est l'une des **innovations les plus importantes** qui a permis à Internet de continuer à fonctionner malgré la pénurie d'adresses IPv4.

## Le problème : l'épuisement des adresses IPv4

### La crise annoncée

Rappelez-vous : IPv4 offre environ **4,3 milliards d'adresses** possibles (2^32).

Dans les années 1980, cela semblait illimité. Mais avec l'explosion d'Internet :

```
Années 1990 :  Millions d'ordinateurs
Années 2000 :  Milliards d'appareils
Années 2010+ : Dizaines de milliards (smartphones, IoT, tablettes...)
```

**Problème** : 4,3 milliards d'adresses ne suffisent plus !

### Les dates clés de l'épuisement

```
2011 : L'IANA (autorité mondiale) distribue les derniers blocs IPv4
2012 : L'Europe épuise son stock
2015 : L'Amérique du Nord épuise son stock
2020 : Pénurie mondiale confirmée
```

Pourtant, Internet continue de fonctionner. Comment est-ce possible ?

**Réponse** : Grâce aux **adresses privées** et au **NAT** (que nous verrons dans la section suivante).

## La solution : RFC 1918

### Qu'est-ce que la RFC 1918 ?

**RFC** = Request for Comments (document standardisant les protocoles Internet)

**RFC 1918** (publiée en 1994) définit des plages d'adresses IP **réservées pour usage privé** :

- Ces adresses peuvent être utilisées **librement** par n'importe qui
- Elles ne sont **jamais routées** sur Internet public
- Elles peuvent être **réutilisées** dans chaque organisation

**Titre officiel** : "Address Allocation for Private Internets"

### L'idée géniale

Au lieu de donner une adresse publique à **chaque appareil** dans le monde :

1. On donne une **seule adresse publique** à chaque organisation (ou foyer)
2. À l'intérieur, on utilise des **adresses privées** réutilisables
3. Un mécanisme (NAT) traduit entre les deux

**Analogie** :
- Adresse publique = numéro de téléphone principal d'une entreprise
- Adresses privées = numéros de poste internes (101, 102, 103...)
- Standard téléphonique = NAT (fait la traduction)

## Les trois plages d'adresses privées

La RFC 1918 définit **trois blocs** d'adresses privées :

### 1. Classe A privée : 10.0.0.0/8

**Plage complète** : `10.0.0.0` à `10.255.255.255`

**Masque** : `255.0.0.0` (ou /8)

**Nombre d'adresses** : 16 777 216 adresses

**Usage typique** : Grandes entreprises, datacenters, campus universitaires

```
Exemples d'adresses dans cette plage :
10.0.0.1
10.1.2.3
10.50.100.200
10.255.255.254
```

**Caractéristiques** :
- ✅ Énormément d'espace d'adressage
- ✅ Permet un subnetting très flexible
- ✅ Idéal pour les grandes organisations avec de nombreux sites

**Exemple d'utilisation** :
```
Entreprise multinationale :
  Siège Paris     : 10.1.0.0/16
  Filiale Londres : 10.2.0.0/16
  Filiale Tokyo   : 10.3.0.0/16
  Datacenter USA  : 10.10.0.0/16
  ...
  Possibilités : 256 réseaux /16 possibles !
```

### 2. Classe B privée : 172.16.0.0/12

**Plage complète** : `172.16.0.0` à `172.31.255.255`

**Masque** : `255.240.0.0` (ou /12)

**Nombre d'adresses** : 1 048 576 adresses

**Usage typique** : Entreprises moyennes, fournisseurs de services

```
Sous-réseaux disponibles :
172.16.0.0/16
172.17.0.0/16
172.18.0.0/16
...
172.31.0.0/16

Total : 16 réseaux /16 (de 172.16 à 172.31)
```

**Attention** : Seule la plage **172.16.x.x à 172.31.x.x** est privée !

```
172.15.0.1  ❌ PUBLIC (en dehors de la plage RFC 1918)
172.16.0.1  ✅ PRIVÉ
172.31.0.1  ✅ PRIVÉ
172.32.0.1  ❌ PUBLIC (en dehors de la plage RFC 1918)
```

**Exemple d'utilisation** :
```
Entreprise de taille moyenne :
  Bureau principal   : 172.16.0.0/16
  Agence régionale 1 : 172.17.0.0/16
  Agence régionale 2 : 172.18.0.0/16
  DMZ (serveurs)     : 172.19.0.0/16
```

### 3. Classe C privée : 192.168.0.0/16

**Plage complète** : `192.168.0.0` à `192.168.255.255`

**Masque** : `255.255.0.0` (ou /16)

**Nombre d'adresses** : 65 536 adresses

**Usage typique** : Réseaux domestiques, petits bureaux (SOHO)

```
Sous-réseaux courants :
192.168.0.0/24   (256 adresses)
192.168.1.0/24   (256 adresses) ← Le plus populaire !
192.168.2.0/24   (256 adresses)
...
192.168.255.0/24

Total : 256 réseaux /24 possibles
```

**C'est la plage la plus connue** : quasiment toutes les box Internet utilisent `192.168.x.x` !

**Exemples courants** :
```
Box Orange/SFR    : 192.168.1.0/24 (box en 192.168.1.1)
Box Bouygues      : 192.168.1.0/24 ou 192.168.10.0/24
Ancienne Freebox  : 192.168.0.0/24
Freebox Révolution: 192.168.1.0/24 (box en 192.168.1.254)
Livebox           : 192.168.1.0/24
```

**Exemple d'utilisation domestique** :
```
Réseau maison : 192.168.1.0/24

192.168.1.1    : Box Internet (passerelle)
192.168.1.10   : PC bureau
192.168.1.20   : PC portable
192.168.1.30   : Smartphone
192.168.1.40   : Tablette
192.168.1.50   : Smart TV
192.168.1.100  : Imprimante
192.168.1.200  : Caméra de sécurité
```

## Tableau récapitulatif des plages privées

```
┌─────────────────┬──────────────────────────┬───────────────┬────────────────┐
│ Plage RFC 1918  │ De ... à ...             │ Masque        │ Nb d'adresses  │
├─────────────────┼──────────────────────────┼───────────────┼────────────────┤
│ 10.0.0.0/8      │ 10.0.0.0                 │ 255.0.0.0     │ 16 777 216     │
│                 │ à 10.255.255.255         │               │                │
├─────────────────┼──────────────────────────┼───────────────┼────────────────┤
│ 172.16.0.0/12   │ 172.16.0.0               │ 255.240.0.0   │ 1 048 576      │
│                 │ à 172.31.255.255         │               │                │
├─────────────────┼──────────────────────────┼───────────────┼────────────────┤
│ 192.168.0.0/16  │ 192.168.0.0              │ 255.255.0.0   │ 65 536         │
│                 │ à 192.168.255.255        │               │                │
└─────────────────┴──────────────────────────┴───────────────┴────────────────┘

Usage typique :
10.x.x.x      → Grandes entreprises, datacenters
172.16-31.x.x → Entreprises moyennes
192.168.x.x   → Foyers, petits bureaux, PME
```

## Adresses publiques : le reste du monde

### Qu'est-ce qu'une adresse publique ?

Une adresse **publique** (ou **routable**) est une adresse IP :

- ✅ **Unique** dans le monde entier (pas de doublon)
- ✅ **Routable** sur Internet (les routeurs la connaissent)
- ✅ **Attribuée** par une autorité (RIR - Regional Internet Registry)
- ✅ **Payante** (généralement facturée par votre FAI)

**Exemples d'adresses publiques** :
```
8.8.8.8           (DNS de Google)
1.1.1.1           (DNS de Cloudflare)
93.184.216.34     (example.com)
151.101.1.140     (Reddit)
172.217.22.14     (Google) ← Oui, 172.x mais pas dans 172.16-31 !
```

### Comment obtenir une adresse publique ?

**Pour un particulier** :
- Votre FAI (Fournisseur d'Accès Internet) vous attribue **une** adresse publique (parfois dynamique)
- Cette adresse est assignée à votre box/routeur

**Pour une entreprise** :
- Acheter un bloc d'adresses auprès de votre FAI ou d'un RIR
- Exemple : acheter un /24 (256 adresses) ou un /29 (8 adresses)

**Coût** : Les adresses publiques ont de la valeur ! Un bloc /24 peut valoir des milliers d'euros.

### Les 5 RIR (autorités régionales)

Les adresses publiques sont gérées par 5 organisations régionales :

```
┌──────────┬─────────────────────────────────────────┐
│ RIR      │ Région                                  │
├──────────┼─────────────────────────────────────────┤
│ ARIN     │ Amérique du Nord                        │
│ RIPE NCC │ Europe, Moyen-Orient, Asie Centrale     │
│ APNIC    │ Asie-Pacifique                          │
│ LACNIC   │ Amérique Latine et Caraïbes             │
│ AFRINIC  │ Afrique                                 │
└──────────┴─────────────────────────────────────────┘
```

Ces organisations distribuent les adresses aux FAI, qui les redistribuent ensuite aux utilisateurs finaux.

## Comment distinguer public et privé ?

### Méthode rapide de vérification

```
Est-ce une adresse privée ?

OUI si elle commence par :
  - 10.x.x.x
  - 172.16.x.x à 172.31.x.x
  - 192.168.x.x

NON dans tous les autres cas
```

### Exemples d'identification

```
10.5.4.3         ✅ PRIVÉ  (commence par 10)
172.16.50.1      ✅ PRIVÉ  (172.16 à 172.31)
172.20.10.5      ✅ PRIVÉ  (172.16 à 172.31)
192.168.1.1      ✅ PRIVÉ  (192.168)
8.8.8.8          ❌ PUBLIC (Google DNS)
172.15.0.1       ❌ PUBLIC (hors plage 172.16-31)
172.32.0.1       ❌ PUBLIC (hors plage 172.16-31)
193.168.1.1      ❌ PUBLIC (193, pas 192)
100.64.0.1       ⚠️  SPÉCIAL (Carrier-Grade NAT, RFC 6598)
```

### Autres adresses spéciales à connaître

En plus de la RFC 1918, d'autres plages sont réservées :

```
┌──────────────────┬──────────────────────────────────────────┐
│ Plage            │ Usage                                    │
├──────────────────┼──────────────────────────────────────────┤
│ 127.0.0.0/8      │ Loopback (localhost)                     │
│ 169.254.0.0/16   │ APIPA (auto-config sans DHCP)            │
│ 224.0.0.0/4      │ Multicast (classe D)                     │
│ 240.0.0.0/4      │ Réservé (classe E)                       │
│ 100.64.0.0/10    │ Carrier-Grade NAT (RFC 6598)             │
│ 0.0.0.0/8        │ Adresse "this network"                   │
└──────────────────┴──────────────────────────────────────────┘
```

**Note sur 100.64.0.0/10** : Utilisée par les FAI pour du NAT à grande échelle (CGNAT). Vous pourriez voir cette plage sur certaines connexions mobiles 4G/5G.

## Le fonctionnement : isolation des réseaux privés

### Principe fondamental

**Règle d'or** : Les routeurs sur Internet **ignorent** les paquets avec des adresses privées.

```
Scénario : Vous envoyez un paquet depuis 192.168.1.10 vers Google

Paquet créé :
  Source      : 192.168.1.10 (privé)
  Destination : 8.8.8.8 (public)

Au routeur de votre box :
  ❌ Ce paquet ne peut PAS sortir tel quel !
  ✅ Le NAT traduit 192.168.1.10 → [votre IP publique]

Paquet modifié :
  Source      : 203.0.113.45 (votre IP publique)
  Destination : 8.8.8.8 (public)

→ Maintenant le paquet peut voyager sur Internet !
```

**Pourquoi cette règle ?**

Si les adresses privées étaient routées sur Internet :
- Des millions de réseaux utilisent `192.168.1.1`
- Comment un routeur saurait-il vers QUEL `192.168.1.1` envoyer un paquet ?
- C'est impossible ! D'où l'isolation.

### Visualisation réseau domestique

```
                    INTERNET
                       ↑
                       │
                [IP publique]
              203.0.113.45:12345
                       │
                       ↓
        ┌──────────────────────────┐
        │    Box Internet (NAT)    │
        │   IP publique externe    │
        │   IP privée interne      │
        │     192.168.1.1          │
        └──────────────────────────┘
                       │
                       │ Réseau local
         ──────────────┴─────────────────
         │              │               │
    192.168.1.10   192.168.1.20    192.168.1.30
    (PC Bureau)    (Smartphone)    (Tablette)

Tous les appareils partagent la MÊME IP publique
grâce au NAT (Network Address Translation)
```

## Avantages des adresses privées

### 1. Conservation des adresses publiques

**Sans adresses privées** : Il faudrait 4 adresses publiques pour ce foyer :
```
PC : 203.0.113.45
Smartphone : 203.0.113.46
Tablette : 203.0.113.47
Box : 203.0.113.48
```

**Avec adresses privées** : Une seule adresse publique suffit !
```
Box : 203.0.113.45 (public)
  ├─ PC : 192.168.1.10 (privé)
  ├─ Smartphone : 192.168.1.20 (privé)
  └─ Tablette : 192.168.1.30 (privé)
```

**Impact** : Des millions d'adresses économisées !

### 2. Sécurité par obscurité

Les appareils avec adresses privées sont **invisibles** depuis Internet :

```
Depuis Internet, impossible de :
❌ Pinguer directement 192.168.1.10
❌ Scanner les ports de 192.168.1.10
❌ Se connecter directement à 192.168.1.10

Ces appareils sont "cachés" derrière la box
```

**Analogie** : C'est comme avoir un numéro de téléphone interne dans une entreprise. De l'extérieur, on ne peut appeler que le standard (l'IP publique), pas directement les postes internes.

### 3. Flexibilité et réutilisabilité

Chaque organisation peut utiliser **les mêmes adresses** :

```
Entreprise A utilise : 192.168.1.0/24
Entreprise B utilise : 192.168.1.0/24 (même plage !)
Votre domicile utilise : 192.168.1.0/24 (encore la même !)

Aucun conflit car ces réseaux sont isolés !
```

### 4. Simplification de la configuration

Les plages privées sont **prévisibles et standardisées** :

- Pas besoin de demander des adresses à un RIR
- Pas de facturation
- Changement de FAI ? Pas besoin de renumeroter !
- Facilite la documentation et les formations

### 5. Mobilité

Vous pouvez déplacer vos équipements sans changer d'adresses :

```
Bureau : Serveur en 10.0.1.50
↓ Déménagement
Nouveau bureau : Serveur toujours en 10.0.1.50

Pas de reconfiguration nécessaire !
```

## Inconvénients et limitations

### 1. Besoin de NAT

Les adresses privées **ne peuvent pas communiquer** directement avec Internet.

**Conséquence** :
- Il faut un routeur NAT
- Complexité ajoutée
- Peut poser des problèmes pour certaines applications (P2P, VoIP, jeux en ligne)

### 2. Connexions entrantes difficiles

Depuis Internet, impossible d'initier une connexion vers une IP privée :

```
Problème : Vous hébergez un serveur web sur 192.168.1.50

Comment un utilisateur sur Internet peut-il y accéder ?
→ Il faut configurer du "port forwarding" sur la box
```

**Solution** : Redirection de ports (nous verrons ça avec le NAT).

### 3. Traçabilité limitée

Plusieurs utilisateurs partagent la même IP publique :

```
IP publique 203.0.113.45 est utilisée par :
  - PC de Jean (192.168.1.10)
  - Smartphone de Marie (192.168.1.20)
  - Tablette de Pierre (192.168.1.30)

Depuis Internet, impossible de savoir qui fait quoi !
```

**Impact** :
- ✅ Meilleure confidentialité
- ❌ Difficulté pour certaines applications (ex: logs de sécurité)

### 4. Pas de véritable "end-to-end"

Le principe original d'Internet était **end-to-end** : chaque appareil avec son IP unique.

NAT + adresses privées cassent ce modèle :
- Les paquets sont modifiés en route
- Certains protocoles ne fonctionnent plus correctement
- Complexité pour les développeurs

## Cas d'usage pratiques

### Scénario 1 : Réseau domestique

```
Connexion FAI :
  IP publique (dynamique) : 203.0.113.45

Réseau interne : 192.168.1.0/24
  Box/Routeur : 192.168.1.1
  PC fixe : 192.168.1.10
  PC portable : 192.168.1.11
  Smartphone 1 : 192.168.1.20
  Smartphone 2 : 192.168.1.21
  Tablette : 192.168.1.30
  Smart TV : 192.168.1.40
  Console jeux : 192.168.1.50
  Imprimante : 192.168.1.100
  NAS : 192.168.1.200

Total : 10 appareils avec 1 seule IP publique !
```

### Scénario 2 : PME (Petite/Moyenne Entreprise)

```
Connexion entreprise :
  IP publique (statique) : 203.0.113.100

Réseau interne : 10.0.0.0/16

  Sous-réseau Administration : 10.0.1.0/24
    Routeur : 10.0.1.1
    Postes admin : 10.0.1.10-50

  Sous-réseau Comptabilité : 10.0.2.0/24
    Routeur : 10.0.2.1
    Postes compta : 10.0.2.10-30

  Sous-réseau Serveurs : 10.0.10.0/24
    Serveur web : 10.0.10.10
    Serveur fichiers : 10.0.10.20
    Serveur mail : 10.0.10.30

  Sous-réseau Invités : 10.0.100.0/24
    WiFi invités : 10.0.100.1-254

Total : Des centaines d'appareils avec 1 IP publique !
```

### Scénario 3 : Entreprise multi-sites

```
Siège social (Paris) :
  IP publique : 203.0.113.200
  Réseau privé : 10.1.0.0/16

Filiale Lyon :
  IP publique : 203.0.114.50
  Réseau privé : 10.2.0.0/16

Filiale Marseille :
  IP publique : 203.0.115.75
  Réseau privé : 10.3.0.0/16

VPN entre les sites permet de connecter
tous ces réseaux privés de façon sécurisée !
```

### Scénario 4 : Cloud provider

```
AWS, Azure, Google Cloud utilisent massivement les adresses privées :

VPC (Virtual Private Cloud) : 10.0.0.0/16

  Subnet public : 10.0.1.0/24
    - Instances avec IP publique (web servers)

  Subnet private : 10.0.2.0/24
    - Bases de données (pas d'accès direct depuis Internet)
    - Serveurs applicatifs

Avantage : Sécurité + économie d'IPs publiques
```

## Bonnes pratiques

### 1. Choisir la bonne plage

```
✅ Maison/Petit bureau : 192.168.x.0/24
   - Simple, standard
   - Compatible avec toutes les box

✅ PME : 172.16.x.0/16 ou 10.0.0.0/16
   - Plus d'espace pour grandir
   - Facilite le subnetting

✅ Grande entreprise : 10.0.0.0/8
   - Énorme espace d'adressage
   - Maximum de flexibilité
```

### 2. Éviter les conflits

**Problème courant** : Connexion VPN vers un bureau qui utilise le même réseau que votre domicile !

```
Votre domicile : 192.168.1.0/24
Bureau : 192.168.1.0/24

→ CONFLIT ! Le VPN ne fonctionnera pas correctement
```

**Solution** : Utiliser des plages différentes :
```
Domicile : 192.168.1.0/24
Bureau : 10.0.0.0/16

→ Pas de conflit, VPN fonctionne ✅
```

### 3. Documenter votre plan d'adressage

Maintenez un document avec :

```
┌─────────────────┬──────────────────────────────────┐
│ Plage           │ Usage                            │
├─────────────────┼──────────────────────────────────┤
│ 10.0.1.0/24     │ Serveurs (10.0.1.1-254)          │
│ 10.0.10.0/24    │ Postes de travail IT             │
│ 10.0.20.0/24    │ Postes de travail Compta         │
│ 10.0.50.0/24    │ Téléphones IP                    │
│ 10.0.100.0/24   │ WiFi employés                    │
│ 10.0.200.0/24   │ WiFi invités                     │
│ 10.1.0.0/16     │ Réservé pour expansion           │
└─────────────────┴──────────────────────────────────┘
```

### 4. Séparer par fonction

Utilisez des sous-réseaux différents pour :

```
Serveurs de production    : 10.0.10.0/24
Serveurs de test          : 10.0.20.0/24
Postes utilisateurs       : 10.0.100.0/23
Imprimantes              : 10.0.150.0/24
Invités                  : 10.0.200.0/24
IoT/Caméras              : 10.0.210.0/24
```

**Avantages** :
- Sécurité (firewall entre les sous-réseaux)
- Organisation claire
- Facilite le troubleshooting

### 5. Réserver des adresses statiques

Pour les équipements critiques :

```
10.0.1.1        : Routeur/Gateway (toujours .1)
10.0.1.2-10     : Serveurs critiques
10.0.1.100-199  : Imprimantes
10.0.1.200-254  : Équipements réseau (switches, AP WiFi)
10.0.1.11-99    : Pool DHCP pour postes utilisateurs
```

## Vérifier si vous êtes en réseau privé

### Sur Windows

```powershell
ipconfig

Résultat typique :
Carte Ethernet Ethernet :
   Adresse IPv4. . . : 192.168.1.10
   Masque de sous-réseau : 255.255.255.0
   Passerelle par défaut : 192.168.1.1

→ 192.168.1.10 = adresse PRIVÉE
```

### Sur Linux/Mac

```bash
ip addr show
# ou
ifconfig

Résultat typique :
eth0: inet 192.168.1.10/24

→ 192.168.1.10 = adresse PRIVÉE
```

### Trouver votre IP publique

Votre adresse privée n'est visible que localement. Pour connaître votre IP publique :

```bash
# En ligne de commande
curl ifconfig.me
curl icanhazip.com

# Ou visitez un site web
https://whatismyip.com
https://www.mon-ip.com
```

**Résultat** : Vous verrez l'IP publique de votre box/routeur (ex: `203.0.113.45`), pas votre IP privée.

## Le futur : IPv6 change la donne

### IPv6 : retour à l'end-to-end

Avec IPv6 (340 undécillions d'adresses), le NAT et les adresses privées deviennent optionnels :

```
IPv4 (aujourd'hui) :
  Pénurie → Adresses privées + NAT nécessaires

IPv6 (futur) :
  Abondance → Chaque appareil peut avoir une IP publique unique
```

### Adresses locales uniques (ULA) en IPv6

IPv6 a quand même un équivalent des adresses privées :

```
Plage ULA : fc00::/7 (fd00::/8 en pratique)

Exemple : fd12:3456:789a::/48
```

**Différence** : Elles sont **optionnelles** en IPv6, contrairement à IPv4 où elles sont indispensables.

## Résumé visuel

```
┌───────────────────────────────────────────────────────────────┐
│         ADRESSES PUBLIQUES vs PRIVÉES (RFC 1918)              │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  🌍 ADRESSES PUBLIQUES                                        │
│     • Uniques au monde entier                                 │
│     • Routables sur Internet                                  │
│     • Attribuées par les RIR/FAI                              │
│     • Limitées (4,3 milliards)                                │
│     • Coûteuses                                               │
│                                                               │
│  🏠 ADRESSES PRIVÉES (RFC 1918)                               │
│     • Réutilisables dans chaque réseau                        │
│     • NON routables sur Internet                              │
│     • Gratuites et libres d'usage                             │
│     • Nécessitent NAT pour accéder à Internet                 │
│                                                               │
│  📋 LES 3 PLAGES PRIVÉES :                                    │
│     10.0.0.0/8         → 16,7 millions d'adresses             │
│     172.16.0.0/12      → 1 million d'adresses                 │
│     192.168.0.0/16     → 65 536 adresses                      │
│                                                               │
│  💡 PRINCIPE :                                                │
│     Adresse privée → NAT → Adresse publique → Internet        │
│                                                               │
│  ✅ AVANTAGES :                                               │
│     • Économie massive d'adresses publiques                   │
│     • Sécurité (isolation)                                    │
│     • Flexibilité (réutilisabilité)                           │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ **IPv4 a seulement 4,3 milliards d'adresses**, insuffisant pour tous les appareils

✅ **RFC 1918** définit 3 plages d'adresses **privées réutilisables**

✅ Les 3 plages : **10.0.0.0/8**, **172.16.0.0/12**, **192.168.0.0/16**

✅ Les adresses privées **ne sont PAS routées** sur Internet public

✅ Le **NAT** traduit entre adresses privées et publiques

✅ **192.168.x.x** est la plage la plus courante pour les réseaux domestiques

✅ **10.x.x.x** est préféré pour les grandes entreprises

✅ Une adresse publique est **unique au monde**, une privée est **réutilisable**

✅ Attention : seul **172.16.0.0 à 172.31.255.255** est privé (pas tout 172.x.x.x)

✅ Les adresses privées ont **sauvé IPv4** de l'épuisement total

## Pour aller plus loin

Maintenant que vous comprenez la distinction public/privé, nous allons explorer :

- Le **NAT (Network Address Translation)** : comment la traduction fonctionne réellement
- Le **PAT (Port Address Translation)** : comment des centaines d'appareils partagent une IP
- Les différents **types de NAT** : statique, dynamique, overload
- Les **problèmes** causés par le NAT et comment les contourner

---

**💡 Vérification rapide** : Ouvrez un terminal et tapez `ipconfig` (Windows) ou `ip addr` (Linux/Mac). Identifiez votre adresse IP locale. Est-elle privée (RFC 1918) ou publique ? Puis visitez whatismyip.com pour voir votre IP publique. Comparez les deux !

---

*Dans la section suivante, nous allons plonger dans le NAT et comprendre comment votre box transforme magiquement vos adresses privées en adresses publiques...*

⏭️ [NAT et PAT : principes et mécanismes](/03-couche-internet/06-nat-pat.md)
