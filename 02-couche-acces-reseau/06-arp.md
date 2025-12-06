🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.6 ARP (Address Resolution Protocol)

## Introduction

Imaginez que vous voulez envoyer une lettre à quelqu'un dont vous connaissez le nom et la ville, mais pas l'adresse exacte de sa maison. Vous allez devoir **demander** l'adresse précise avant de pouvoir livrer votre lettre. C'est exactement ce que fait **ARP** : il trouve l'adresse MAC (l'adresse physique de la maison) quand vous ne connaissez que l'adresse IP (le nom et la ville).

ARP est le **traducteur** qui fait le pont entre le monde des adresses IP (couche 3) et le monde des adresses MAC (couche 2). Sans lui, vos ordinateurs ne pourraient pas communiquer sur le réseau local !

## Le problème qu'ARP résout

### La situation

Vous avez deux types d'adresses sur un réseau :

```
┌─────────────────────────────────────────────────┐
│  ADRESSE IP (Couche 3 - Internet)               │
│  • 192.168.1.10                                 │
│  • Adresse logique, peut changer                │
│  • Utilisée pour le routage                     │
│  • Ce que VOUS connaissez                       │
└─────────────────────────────────────────────────┘
                    ↕️ ???
┌─────────────────────────────────────────────────┐
│  ADRESSE MAC (Couche 2 - Liaison)               │
│  • 00:1A:2B:3C:4D:5E                            │
│  • Adresse physique, gravée dans le matériel    │
│  • Nécessaire pour envoyer la trame             │
│  • Ce qu'il faut DÉCOUVRIR                      │
└─────────────────────────────────────────────────┘
```

### Le dilemme

```
Vous voulez envoyer des données à 192.168.1.20
    ↓
Votre ordinateur sait créer un paquet IP avec :
    • IP source : 192.168.1.10
    • IP destination : 192.168.1.20
    ✓

Mais pour envoyer sur le réseau local, il faut une TRAME Ethernet avec :
    • MAC source : 00:11:22:33:44:55 (votre MAC)
    • MAC destination : ??? (INCONNUE !)
    ✗

PROBLÈME : Quelle est la MAC de 192.168.1.20 ?
```

**Analogie postale complète** :

```
Vous voulez envoyer une lettre à "Jean Dupont, Paris"
    ↓
Vous connaissez : Le nom (IP)
Vous ne connaissez pas : L'adresse de sa maison (MAC)
    ↓
Vous devez DEMANDER : "Quelqu'un connaît l'adresse de Jean Dupont ?"
    ↓
Jean répond : "C'est moi, j'habite 12 rue de la Paix"
    ↓
Maintenant vous pouvez livrer la lettre !
```

C'est exactement ce que fait ARP : **il demande à tout le monde**.

## Comment fonctionne ARP ?

### Vue d'ensemble du processus

ARP utilise un système simple en **deux étapes** :

1. **ARP Request** (Requête) : "Qui a l'IP X.X.X.X ? Dis-le à Y.Y.Y.Y (moi)"
2. **ARP Reply** (Réponse) : "C'est moi qui ai l'IP X.X.X.X ! Ma MAC est AA:BB:CC:DD:EE:FF"

### Scénario détaillé

**Contexte** :
- Ordinateur A : IP `192.168.1.10`, MAC `AA:AA:AA:AA:AA:AA`
- Ordinateur B : IP `192.168.1.20`, MAC `BB:BB:BB:BB:BB:BB`
- A veut communiquer avec B pour la première fois

#### Étape 1 : ARP Request (Requête broadcast)

```
A se dit : "Je veux parler à 192.168.1.20, mais je ne connais pas sa MAC"
    ↓
A envoie un ARP REQUEST en BROADCAST

┌──────────────────────────────────────────────────┐
│           TRAME ETHERNET                         │
├──────────────────────────────────────────────────┤
│ MAC Destination : FF:FF:FF:FF:FF:FF  ← BROADCAST │
│ MAC Source      : AA:AA:AA:AA:AA:AA  ← A         │
│ EtherType       : 0x0806 (ARP)                   │
├──────────────────────────────────────────────────┤
│           MESSAGE ARP                            │
├──────────────────────────────────────────────────┤
│ Opération       : 1 (Request)                    │
│ MAC Émetteur    : AA:AA:AA:AA:AA:AA              │
│ IP Émetteur     : 192.168.1.10                   │
│ MAC Cible       : 00:00:00:00:00:00  ← Inconnu ! │
│ IP Cible        : 192.168.1.20       ← Recherché │
└──────────────────────────────────────────────────┘

Question : "Qui a l'IP 192.168.1.20 ? Réponds-moi à 192.168.1.10"
```

**Le broadcast se propage** :

```
       Switch
         │
    ┌────┼────┬────┐
    │    │    │    │
   💻   💻    💻   💻
   A    B    C    D

A envoie le broadcast → Tous reçoivent la requête !
```

**Réactions** :

```
💻 B (192.168.1.20) : "C'est pour moi ! Je vais répondre"
💻 C (192.168.1.30) : "Ce n'est pas mon IP, j'ignore"
💻 D (192.168.1.40) : "Ce n'est pas mon IP, j'ignore"
```

#### Étape 2 : ARP Reply (Réponse unicast)

```
B répond UNIQUEMENT à A (unicast)

┌──────────────────────────────────────────────────┐
│           TRAME ETHERNET                         │
├──────────────────────────────────────────────────┤
│ MAC Destination : AA:AA:AA:AA:AA:AA  ← A         │
│ MAC Source      : BB:BB:BB:BB:BB:BB  ← B         │
│ EtherType       : 0x0806 (ARP)                   │
├──────────────────────────────────────────────────┤
│           MESSAGE ARP                            │
├──────────────────────────────────────────────────┤
│ Opération       : 2 (Reply)                      │
│ MAC Émetteur    : BB:BB:BB:BB:BB:BB              │
│ IP Émetteur     : 192.168.1.20                   │
│ MAC Cible       : AA:AA:AA:AA:AA:AA              │
│ IP Cible        : 192.168.1.10                   │
└──────────────────────────────────────────────────┘

Réponse : "Oui, 192.168.1.20 c'est moi ! Ma MAC est BB:BB:BB:BB:BB:BB"
```

**Communication unicast** :

```
       Switch
         │
    ┌────┼────┬────┐
    │    │    │    │
   💻 ← 💻    💻   💻
   A    B    C    D

B répond UNIQUEMENT à A (pas de broadcast)
C et D ne reçoivent rien
```

#### Étape 3 : Mise en cache et communication

```
A reçoit la réponse
    ↓
A enregistre dans son CACHE ARP :
    192.168.1.20 → BB:BB:BB:BB:BB:BB
    ↓
Maintenant A peut envoyer des trames Ethernet normales à B !
    ↓
Plus besoin de redemander (tant que le cache est valide)
```

### Schéma complet du processus

```
┌─────────────────────────────────────────────────────────────┐
│              PROCESSUS COMPLET ARP                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  A veut envoyer des données à IP 192.168.1.20               │
│         ↓                                                   │
│  A consulte son CACHE ARP                                   │
│         ↓                                                   │
│  ┌──────────────────────────┐                               │
│  │ 192.168.1.20 dans cache ?│                               │
│  └────┬────────────────┬────┘                               │
│       │ OUI            │ NON                                │
│       ↓                ↓                                    │
│   Utiliser MAC     Envoyer ARP REQUEST                      │
│   du cache         (broadcast)                              │
│       │                ↓                                    │
│       │            Attendre ARP REPLY                       │
│       │                ↓                                    │
│       │            Ajouter au cache                         │
│       ↓                ↓                                    │
│       └────────┬───────┘                                    │
│                ↓                                            │
│        Envoyer la trame Ethernet                            │
│        avec la bonne MAC destination                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Structure d'un paquet ARP

### Format détaillé

```
PAQUET ARP (28 octets)
┌────────────────────────────────────────────────────┐
│  Hardware Type (2 octets)                          │
│  0x0001 = Ethernet                                 │
├────────────────────────────────────────────────────┤
│  Protocol Type (2 octets)                          │
│  0x0800 = IPv4                                     │
├────────────────────────────────────────────────────┤
│  Hardware Length (1 octet)                         │
│  6 = Longueur d'une MAC                            │
├────────────────────────────────────────────────────┤
│  Protocol Length (1 octet)                         │
│  4 = Longueur d'une IPv4                           │
├────────────────────────────────────────────────────┤
│  Operation (2 octets)                              │
│  1 = Request, 2 = Reply                            │
├────────────────────────────────────────────────────┤
│  Sender Hardware Address (6 octets)                │
│  Adresse MAC de l'émetteur                         │
├────────────────────────────────────────────────────┤
│  Sender Protocol Address (4 octets)                │
│  Adresse IP de l'émetteur                          │
├────────────────────────────────────────────────────┤
│  Target Hardware Address (6 octets)                │
│  Adresse MAC du destinataire (0 si inconnue)       │
├────────────────────────────────────────────────────┤
│  Target Protocol Address (4 octets)                │
│  Adresse IP du destinataire                        │
└────────────────────────────────────────────────────┘
```

### Exemple concret (ARP Request)

```
TRAME ETHERNET + ARP REQUEST

┌─── En-tête Ethernet ─────────────────────────────┐
│ FF FF FF FF FF FF     ← MAC Dest (broadcast)     │
│ AA AA AA AA AA AA     ← MAC Source (A)           │
│ 08 06                 ← EtherType (ARP)          │
├─── Paquet ARP ───────────────────────────────────┤
│ 00 01                 ← Hardware Type (Ethernet) │
│ 08 00                 ← Protocol Type (IPv4)     │
│ 06                    ← Hardware Length (6)      │
│ 04                    ← Protocol Length (4)      │
│ 00 01                 ← Operation (Request)      │
│ AA AA AA AA AA AA     ← Sender MAC (A)           │
│ C0 A8 01 0A           ← Sender IP (192.168.1.10) │
│ 00 00 00 00 00 00     ← Target MAC (inconnu)     │
│ C0 A8 01 14           ← Target IP (192.168.1.20) │
└──────────────────────────────────────────────────┘
         (en hexadécimal)
```

**Traduction** :

```
"Je suis AA:AA:AA:AA:AA:AA (192.168.1.10).
Qui a l'IP 192.168.1.20 ? Je ne connais pas sa MAC (00:00:00:00:00:00).
Réponds-moi s'il te plaît !"
```

### Exemple concret (ARP Reply)

```
TRAME ETHERNET + ARP REPLY

┌─── En-tête Ethernet ─────────────────────────────┐
│ AA AA AA AA AA AA     ← MAC Dest (A - unicast)   │
│ BB BB BB BB BB BB     ← MAC Source (B)           │
│ 08 06                 ← EtherType (ARP)          │
├─── Paquet ARP ───────────────────────────────────┤
│ 00 01                 ← Hardware Type (Ethernet) │
│ 08 00                 ← Protocol Type (IPv4)     │
│ 06                    ← Hardware Length (6)      │
│ 04                    ← Protocol Length (4)      │
│ 00 02                 ← Operation (Reply)        │
│ BB BB BB BB BB BB     ← Sender MAC (B)           │
│ C0 A8 01 14           ← Sender IP (192.168.1.20) │
│ AA AA AA AA AA AA     ← Target MAC (A)           │
│ C0 A8 01 0A           ← Target IP (192.168.1.10) │
└──────────────────────────────────────────────────┘
         (en hexadécimal)
```

**Traduction** :

```
"Salut AA:AA:AA:AA:AA:AA (192.168.1.10) !
Oui, 192.168.1.20 c'est moi !
Ma MAC est BB:BB:BB:BB:BB:BB.
Voilà, tu peux m'envoyer tes données maintenant !"
```

## Le cache ARP

### Qu'est-ce que le cache ARP ?

Le **cache ARP** (ou table ARP) est une **mémoire temporaire** qui stocke les associations IP ↔ MAC déjà découvertes.

**Pourquoi ?**
Éviter de redemander sans cesse ! Ce serait du gaspillage de bande passante.

**Analogie** : C'est comme un carnet d'adresses. Une fois que vous avez demandé l'adresse de quelqu'un, vous la notez pour ne pas redemander à chaque fois.

### Structure du cache

```
┌─────────────────────┬─────────────────────┬──────────┬──────────┐
│  Adresse IP         │  Adresse MAC        │   Type   │   Âge    │
├─────────────────────┼─────────────────────┼──────────┼──────────┤
│ 192.168.1.1         │ 00:11:22:33:44:55   │ dynamique│  45 sec  │
│ 192.168.1.20        │ AA:BB:CC:DD:EE:FF   │ dynamique│  120 sec │
│ 192.168.1.254       │ FF:EE:DD:CC:BB:AA   │ statique │  ---     │
└─────────────────────┴─────────────────────┴──────────┴──────────┘
```

**Champs** :
- **Adresse IP** : L'adresse logique
- **Adresse MAC** : L'adresse physique correspondante
- **Type** :
  - **Dynamique** : Apprise via ARP (expire)
  - **Statique** : Configurée manuellement (ne change pas)
- **Âge** : Temps depuis la dernière utilisation

### Durée de vie

Les entrées dynamiques **expirent** après un certain temps :

**Durées typiques** :
- **Windows** : 2 minutes (utilisée récemment), 10 minutes (pas utilisée)
- **Linux** : 60-120 secondes
- **macOS** : 20 minutes
- **Routeurs Cisco** : 4 heures

**Pourquoi expirer ?**

```
Scénario 1 : Changement d'adresse IP
├─→ Une machine change d'IP
└─→ Sans expiration : ancienne entrée reste (erreur)

Scénario 2 : Remplacement de matériel
├─→ Une carte réseau est remplacée (nouvelle MAC)
└─→ Sans expiration : ancienne MAC reste (échec)

Scénario 3 : Optimisation mémoire
├─→ Des machines se déconnectent
└─→ Sans expiration : cache plein d'entrées inutiles
```

### Visualiser le cache ARP

#### Sur Windows

```cmd
C:\> arp -a

Interface: 192.168.1.10 --- 0x4
  Adresse Internet      Adresse physique      Type
  192.168.1.1          00-11-22-33-44-55     dynamique
  192.168.1.20         aa-bb-cc-dd-ee-ff     dynamique
  192.168.1.254        ff-ee-dd-cc-bb-aa     dynamique
  224.0.0.252          01-00-5e-00-00-fc     statique
```

#### Sur Linux / macOS

```bash
$ arp -n
Address          HWtype  HWaddress           Flags Mask  Iface
192.168.1.1      ether   00:11:22:33:44:55   C           eth0
192.168.1.20     ether   aa:bb:cc:dd:ee:ff   C           eth0
192.168.1.254    ether   ff:ee:dd:cc:bb:aa   C           eth0
```

#### Commandes utiles

**Voir le cache** :
```bash
Windows : arp -a
Linux   : arp -n  ou  ip neigh show
macOS   : arp -a
```

**Vider le cache** :
```bash
Windows : arp -d *  (nécessite admin)
Linux   : sudo ip neigh flush all
macOS   : sudo arp -d -a
```

**Ajouter une entrée statique** :
```bash
Windows : arp -s 192.168.1.50 aa-bb-cc-dd-ee-ff
Linux   : sudo arp -s 192.168.1.50 aa:bb:cc:dd:ee:ff
```

## ARP Gratuit (Gratuitous ARP)

### Définition

Un **ARP Gratuit** (Gratuitous ARP) est un ARP Request spécial où :
- L'IP source = L'IP cible (la même machine)
- Envoyé en broadcast
- **Personne ne doit répondre** (ce n'est pas une vraie question)

**Format** :

```
┌──────────────────────────────────────────────────┐
│ ARP REQUEST GRATUIT                              │
├──────────────────────────────────────────────────┤
│ MAC Émetteur    : AA:AA:AA:AA:AA:AA              │
│ IP Émetteur     : 192.168.1.10                   │
│ MAC Cible       : 00:00:00:00:00:00              │
│ IP Cible        : 192.168.1.10   ← MÊME IP !     │
└──────────────────────────────────────────────────┘

Message : "Je suis 192.168.1.10 et ma MAC est AA:AA:AA:AA:AA:AA"
```

### Pourquoi envoyer un ARP Gratuit ?

#### 1. Annoncer son arrivée

```
Une nouvelle machine rejoint le réseau
    ↓
Elle envoie un ARP gratuit
    ↓
"Bonjour ! Je suis 192.168.1.10, ma MAC est AA:AA:AA:AA:AA:AA"
    ↓
Tous les autres mettent à jour leur cache
    ↓
Les futures communications sont plus rapides
```

**Analogie** : C'est comme se présenter en arrivant dans une salle : "Bonjour, je m'appelle Pierre !" pour que tout le monde vous connaisse.

#### 2. Détecter les conflits d'IP

```
Machine A veut utiliser 192.168.1.10
    ↓
Elle envoie un ARP gratuit avec cette IP
    ↓
Si une autre machine utilise DÉJÀ cette IP
    ↓
Elle répond (normalement elle ne devrait pas)
    ↓
CONFLIT DÉTECTÉ !
    ↓
A sait qu'elle ne peut pas utiliser cette IP
```

**Analogie** : Avant de s'asseoir, vous demandez "Cette place est prise ?" Si quelqu'un répond "Oui", vous savez qu'il y a un problème.

#### 3. Mettre à jour les caches après un changement

```
Machine B change sa carte réseau (nouvelle MAC)
    ↓
Même IP : 192.168.1.20
Nouvelle MAC : XX:XX:XX:XX:XX:XX
    ↓
B envoie un ARP gratuit
    ↓
"Mon IP est toujours 192.168.1.20, mais ma nouvelle MAC est XX:XX:XX:XX:XX:XX"
    ↓
Tous les caches sont mis à jour
    ↓
Les communications continuent sans problème
```

**Analogie** : C'est comme prévenir vos amis que vous avez changé de numéro de téléphone, mais que vous êtes toujours vous.

#### 4. Basculement haute disponibilité (failover)

```
Serveur principal (192.168.1.100) tombe en panne
    ↓
Serveur de secours prend le relais
    ↓
Il s'attribue la MÊME IP : 192.168.1.100
    ↓
Il envoie un ARP gratuit
    ↓
"C'est moi maintenant qui gère 192.168.1.100 ! Ma MAC est YY:YY:YY:YY:YY:YY"
    ↓
Les clients mettent à jour leur cache
    ↓
Le service continue sans interruption apparente
```

**Analogie** : C'est comme un remplaçant qui arrive et dit "Je m'appelle maintenant Pierre, adressez-vous à moi maintenant".

## ARP Probe

Un **ARP Probe** est similaire à l'ARP gratuit mais encore plus prudent :

```
┌──────────────────────────────────────────────────┐
│ ARP PROBE (avant d'utiliser une IP)              │
├──────────────────────────────────────────────────┤
│ MAC Émetteur    : AA:AA:AA:AA:AA:AA              │
│ IP Émetteur     : 0.0.0.0         ← IP nulle !   │
│ MAC Cible       : 00:00:00:00:00:00              │
│ IP Cible        : 192.168.1.10    ← IP testée    │
└──────────────────────────────────────────────────┘

Message : "Quelqu'un utilise 192.168.1.10 ? (je n'ai pas encore d'IP)"
```

**Usage** : Avant d'utiliser une IP (ex: attribution DHCP), on vérifie qu'elle est libre.

**Processus** :
1. Envoyer 3 ARP Probes (espacés d'1 seconde)
2. Si personne ne répond → IP libre, on peut l'utiliser
3. Si quelqu'un répond → IP déjà utilisée, choisir une autre

## RARP (Reverse ARP)

### Concept

**ARP** : "J'ai une IP, je cherche la MAC"
**RARP** : "J'ai une MAC, je cherche l'IP" (inverse !)

```
┌────────────────────────────────────────────┐
│         ARP (normal)                       │
├────────────────────────────────────────────┤
│  Connu : Adresse IP                        │
│  Cherché : Adresse MAC                     │
│  Usage : Communication normale             │
└────────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│         RARP (inverse)                     │
├────────────────────────────────────────────┤
│  Connu : Adresse MAC                       │
│  Cherché : Adresse IP                      │
│  Usage : Machines sans disque dur          │
└────────────────────────────────────────────┘
```

### Scénario d'usage (historique)

```
Terminal sans disque dur boot sur le réseau
    ↓
Il connaît sa MAC (gravée dans la puce)
    ↓
Mais il ne connaît PAS son IP !
    ↓
Il envoie un RARP Request :
"Je suis MAC: AA:AA:AA:AA:AA:AA, quelle est mon IP ?"
    ↓
Un serveur RARP répond :
"Ton IP est 192.168.1.50"
    ↓
Le terminal peut maintenant fonctionner
```

**Note** : RARP est **obsolète** aujourd'hui. Remplacé par :
- **BOOTP** (Bootstrap Protocol)
- **DHCP** (Dynamic Host Configuration Protocol) - Le standard actuel

## Proxy ARP

### Concept

Un **Proxy ARP** est un routeur qui **répond aux requêtes ARP à la place d'une autre machine**.

**Scénario typique** :

```
Réseau A (192.168.1.0/24)          Réseau B (192.168.2.0/24)
        │                                   │
       💻 A                                💻 B
    192.168.1.10                      192.168.2.10
        │                                   │
        └──────────┬ Routeur ┬──────────────┘
                   │ (Proxy) │
                   │  ARP    │
```

**Fonctionnement** :

```
1. A veut communiquer avec B (192.168.2.10)
2. A pense que B est sur le même réseau (erreur de config)
3. A envoie un ARP Request pour 192.168.2.10
4. Le ROUTEUR intercepte la requête
5. Le ROUTEUR répond avec SA propre MAC
6. A envoie ses paquets au routeur
7. Le routeur les route vers B

A → Routeur (en pensant parler à B) → B
```

**Analogie** : C'est comme un concierge qui dit "Donnez-moi le colis, je le donnerai à la personne" quand vous ne pouvez pas accéder directement à quelqu'un.

**Usage** :
- Simplifier la configuration réseau
- Permettre la communication inter-réseaux sans configuration complexe
- Compatibilité avec des anciens systèmes

**Désavantage** : Peut masquer des problèmes de configuration réseau.

## Problèmes de sécurité : ARP Poisoning / Spoofing

### Le problème fondamental

**ARP n'a AUCUNE sécurité** :
- ❌ Pas d'authentification
- ❌ Pas de vérification
- ❌ Les machines croient aveuglément les réponses ARP

```
Principe de confiance naïf :
"Si quelqu'un dit 'Mon IP est X et ma MAC est Y', je le crois !"
```

### ARP Poisoning (Empoisonnement ARP)

Un attaquant envoie de **fausses réponses ARP** pour tromper les machines.

#### Scénario d'attaque Man-in-the-Middle

```
RÉSEAU NORMAL :
A (192.168.1.10) ←──────────→ B (192.168.1.20)
    Communication directe

APRÈS ATTAQUE :
A (192.168.1.10) ←───→ 👾 Attaquant ←───→ B (192.168.1.20)
                       (Intercepte tout)
```

**Étapes de l'attaque** :

```
1. Attaquant envoie un faux ARP à A :
   "192.168.1.20 (B) est à la MAC de l'attaquant"

2. Le cache ARP de A est empoisonné :
   192.168.1.20 → MAC_attaquant (au lieu de MAC_B)

3. Attaquant envoie un faux ARP à B :
   "192.168.1.10 (A) est à la MAC de l'attaquant"

4. Le cache ARP de B est empoisonné :
   192.168.1.10 → MAC_attaquant (au lieu de MAC_A)

5. Maintenant :
   - A envoie à B → va à l'attaquant
   - B envoie à A → va à l'attaquant
   - L'attaquant relaie tout (pour ne pas être détecté)
   - Mais il peut LIRE et MODIFIER tout le trafic !
```

**Conséquences** :
- 🔓 **Interception** : L'attaquant lit tout (mots de passe, données...)
- ✏️ **Modification** : L'attaquant change les données en transit
- 🚫 **Déni de service** : L'attaquant peut bloquer la communication

### ARP Spoofing de la passerelle

**Attaque courante** : Se faire passer pour le routeur/passerelle.

```
NORMAL :
💻 Machines → 🌐 Routeur → Internet

ATTAQUE :
💻 Machines → 👾 Attaquant (se fait passer pour le routeur) → 🌐 Routeur → Internet
              └─→ Tout le trafic vers Internet passe par l'attaquant !
```

**Technique** :

```
Attaquant envoie des ARP gratuits :
"La passerelle (192.168.1.1) est à MA MAC !"
    ↓
Toutes les machines mettent à jour leur cache
    ↓
Tout le trafic vers Internet passe par l'attaquant
    ↓
L'attaquant peut espionner TOUT le réseau
```

### Détection des attaques ARP

**Signes d'une attaque** :

```
✓ Changements fréquents dans le cache ARP
✓ Plusieurs IPs avec la même MAC (impossible normalement)
✓ Dégradation soudaine des performances
✓ Problèmes de connectivité intermittents
✓ Certificats SSL invalides (attaquant fait du SSL stripping)
```

**Outils de détection** :
- **ARPWatch** : Surveille les changements de correspondances IP-MAC
- **XArp** : Détection et protection active
- **Wireshark** : Analyse manuelle du trafic ARP

### Protections contre ARP Poisoning

#### 1. Entries ARP statiques

```bash
# Windows
arp -s 192.168.1.1 00-11-22-33-44-55

# Linux
sudo arp -s 192.168.1.1 00:11:22:33:44:55
```

**Avantage** : Les entrées ne peuvent pas être modifiées
**Inconvénient** : Gestion manuelle, ne passe pas à l'échelle

#### 2. Dynamic ARP Inspection (DAI)

Configuration sur les switches manageables :

```
Le switch surveille les paquets ARP
    ↓
Il valide que l'IP et la MAC correspondent
    ↓
Il bloque les paquets ARP suspects
```

**Configuration réseau nécessaire** : DHCP Snooping + DAI

#### 3. Port Security

```
Limiter le nombre de MACs par port du switch
    ↓
Si une nouvelle MAC apparaît → alerte ou blocage
```

#### 4. Segmentation réseau (VLANs)

```
Séparer le réseau en segments
    ↓
Les attaques ARP ne se propagent que dans leur VLAN
    ↓
Limitation des dégâts
```

#### 5. Chiffrement (la meilleure protection)

```
Utiliser HTTPS, SSH, VPN, etc.
    ↓
Même si l'attaquant intercepte, il ne peut pas lire
    ↓
Protection contre le Man-in-the-Middle
```

**Note** : Le chiffrement ne **prévient pas** l'attaque ARP, mais **protège** les données.

## ARP et IPv6 : NDP (Neighbor Discovery Protocol)

### IPv6 n'utilise pas ARP !

Dans IPv6, ARP est remplacé par **NDP** (Neighbor Discovery Protocol) qui utilise ICMPv6.

**Différences principales** :

| ARP (IPv4) | NDP (IPv6) |
|------------|------------|
| Broadcast (FF:FF:FF:FF:FF:FF) | Multicast (ciblé) |
| Couche 2 uniquement | Intégré à ICMPv6 (couche 3) |
| Pas de sécurité | Peut utiliser IPsec (SEcure Neighbor Discovery) |
| ARP Request/Reply | Neighbor Solicitation/Advertisement |

**Avantage** : NDP est plus efficient (moins de bruit sur le réseau).

**Sécurité** : SEND (SEcure Neighbor Discovery) ajoute de la cryptographie pour empêcher les attaques.

## Cas pratiques et exemples

### Exemple 1 : Premier ping

```
Commande : ping 192.168.1.20

Séquence d'événements :
1. Votre PC vérifie le cache ARP → Vide
2. Envoi d'un ARP Request (broadcast)
   "Qui a 192.168.1.20 ?"
3. Attente de la réponse (timeout ~1-3 secondes)
4. Réception de l'ARP Reply de 192.168.1.20
5. Mise à jour du cache ARP
6. MAINTENANT le premier paquet ICMP peut être envoyé
7. Les paquets suivants sont rapides (cache rempli)

Résultat :
Pinging 192.168.1.20 with 32 bytes of data:
Request timed out.              ← Perdu pendant l'ARP
Reply from 192.168.1.20: time<1ms  ← OK, cache rempli
Reply from 192.168.1.20: time<1ms
Reply from 192.168.1.20: time<1ms
```

**Le premier ping échoue souvent** à cause du temps d'ARP !

### Exemple 2 : Communication via routeur

```
A (192.168.1.10) veut envoyer à Internet (8.8.8.8)
    ↓
A vérifie : 8.8.8.8 est dans mon réseau local ?
    NON (masque différent)
    ↓
A doit utiliser la PASSERELLE (192.168.1.1)
    ↓
A fait un ARP Request pour la passerelle :
    "Qui a 192.168.1.1 ?"
    ↓
Le routeur répond avec sa MAC
    ↓
A envoie la trame au routeur avec :
    - IP destination : 8.8.8.8 (Internet)
    - MAC destination : MAC du routeur
    ↓
Le routeur route le paquet vers Internet
```

**Important** : La MAC destination est **toujours** celle du prochain saut (next hop), même si l'IP finale est différente !

### Exemple 3 : Réseau avec switch

```
4 ordinateurs sur un switch :
A: 192.168.1.10
B: 192.168.1.20
C: 192.168.1.30
D: 192.168.1.40

A fait un ping vers B pour la première fois :

1. ARP Request de A (broadcast)
   ↓
   Le switch envoie sur TOUS les ports
   ↓
   B, C, D reçoivent tous la requête
   ↓
   Seul B répond (unicast vers A)

2. Le switch apprend :
   - Port 1 : MAC de A
   - Port 2 : MAC de B

3. Communications futures A ↔ B :
   Le switch envoie directement au bon port
   C et D ne sont pas dérangés
```

## Schéma récapitulatif complet

```
┌──────────────────────────────────────────────────────────────┐
│                    PROTOCOLE ARP                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PROBLÈME : Connu IP, cherché MAC                            │
│                                                              │
│  SOLUTION :                                                  │
│  ┌────────────────────────────────────────┐                  │
│  │  1. ARP REQUEST (Broadcast)            │                  │
│  │     "Qui a l'IP X.X.X.X ?"             │                  │
│  │     → Envoyé à FF:FF:FF:FF:FF:FF       │                  │
│  │     → Tous reçoivent                   │                  │
│  └─────────────────┬──────────────────────┘                  │
│                    ↓                                         │
│  ┌────────────────────────────────────────┐                  │
│  │  2. ARP REPLY (Unicast)                │                  │
│  │     "C'est moi ! Ma MAC est Y"         │                  │
│  │     → Envoyé uniquement au demandeur   │                  │
│  └─────────────────┬──────────────────────┘                  │
│                    ↓                                         │
│  ┌────────────────────────────────────────┐                  │
│  │  3. MISE EN CACHE                      │                  │
│  │     IP → MAC stockée localement        │                  │
│  │     Durée : 2-20 minutes (selon OS)    │                  │
│  └────────────────────────────────────────┘                  │
│                                                              │
│  TYPES SPÉCIAUX :                                            │
│  • ARP Gratuit : Annonce, détection conflit                  │
│  • ARP Probe : Vérification avant utilisation                │
│  • Proxy ARP : Routeur répond pour autre réseau              │
│                                                              │
│  SÉCURITÉ :                                                  │
│  ⚠️  ARP Poisoning / Spoofing                                │
│  ✓  Protection : DAI, Port Security, Chiffrement             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

1. **Rôle d'ARP** : Trouver l'adresse MAC correspondant à une adresse IP sur le réseau local.

2. **Mécanisme** :
   - **Request** en broadcast (tout le monde reçoit)
   - **Reply** en unicast (seul le demandeur reçoit)

3. **Cache ARP** : Stocke les associations IP↔MAC pour éviter de redemander constamment.

4. **Expiration** : Les entrées du cache expirent après 2-20 minutes selon l'OS.

5. **ARP Gratuit** : Annonce son arrivée, détecte les conflits, met à jour les caches.

6. **Portée locale** : ARP ne fonctionne que sur le réseau local (pas à travers les routeurs).

7. **Sécurité faible** : ARP n'a pas d'authentification, vulnérable aux attaques (poisoning).

8. **Protection** : DAI, Port Security, segmentation, et surtout **chiffrement** des données.

9. **IPv6** : Utilise NDP (Neighbor Discovery Protocol) au lieu d'ARP.

10. **Essentiel** : Sans ARP, impossible de communiquer sur un réseau Ethernet !

---

## Prochaine étape

Maintenant que vous comprenez comment ARP relie les adresses IP aux adresses MAC dans les réseaux câblés, nous allons explorer les **technologies sans fil (Wi-Fi)** et découvrir comment elles s'intègrent dans la couche Accès réseau avec leurs propres spécificités !

---


⏭️ [Technologies sans fil (Wi-Fi) et leur intégration](/02-couche-acces-reseau/07-technologies-sans-fil.md)
