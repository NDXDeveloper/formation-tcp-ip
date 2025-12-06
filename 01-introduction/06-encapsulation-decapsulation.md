🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 1.6 Encapsulation et décapsulation des données

## Introduction

Vous avez maintenant compris les modèles en couches (OSI et TCP/IP). Mais comment les données **voyagent-elles réellement** à travers ces couches ? Comment un simple email devient-il des signaux électriques sur un câble, puis redevient-il un email lisible ?

La réponse : **l'encapsulation** (à l'envoi) et la **décapsulation** (à la réception).

C'est le mécanisme **le plus fondamental** des réseaux. Comprendre l'encapsulation, c'est comprendre comment fonctionne vraiment la communication réseau !

**Analogie préliminaire** : Imaginez envoyer un cadeau fragile par la poste. Vous ne l'envoyez pas tel quel ! Vous l'emballez dans du papier bulle (première couche), puis dans un carton (deuxième couche), avec une étiquette d'adresse (troisième couche). À l'arrivée, on retire les couches dans l'ordre inverse pour récupérer le cadeau.

---

## Définitions simples

### Encapsulation

**L'encapsulation** est le processus d'**ajout d'informations de contrôle** (en-têtes et parfois pieds de page) aux données à chaque couche descendante du modèle.

**Analogie** : Mettre une lettre dans une enveloppe, puis cette enveloppe dans un colis, puis ce colis dans un conteneur postal.

### Décapsulation

**La décapsulation** est le processus **inverse** : retirer les en-têtes à chaque couche montante pour récupérer les données originales.

**Analogie** : Le facteur sort le colis du camion, retire l'emballage extérieur, puis l'enveloppe, pour vous donner la lettre.

### Pourquoi faire ça ?

Chaque couche a besoin de **ses propres informations de contrôle** :
- La couche Transport a besoin des numéros de ports
- La couche Internet a besoin des adresses IP
- La couche Accès réseau a besoin des adresses MAC

Au lieu de créer un seul énorme en-tête avec tout, **chaque couche ajoute sa propre information**, de manière modulaire et indépendante.

---

## Les noms des unités de données (PDU)

À chaque couche, les données ont un **nom spécifique** :

```
Modèle TCP/IP              Nom de l'unité de données
┌────────────────────┐
│  4. Application    │     Données / Message
├────────────────────┤            ↓ Encapsulation
│  3. Transport      │     Segment (TCP) / Datagramme (UDP)
├────────────────────┤            ↓ Encapsulation
│  2. Internet       │     Paquet / Datagramme IP
├────────────────────┤            ↓ Encapsulation
│  1. Accès réseau   │     Trame (Frame)
└────────────────────┘            ↓
                              Bits (transmission physique)
```

**PDU (Protocol Data Unit)** : Unité de données de protocole - c'est le nom générique.

**Vocabulaire important** :
- **Couche Application** : Données, Message
- **Couche Transport** : **Segment** (TCP) ou **Datagramme** (UDP)
- **Couche Internet** : **Paquet** (ou datagramme IP)
- **Couche Accès réseau** : **Trame** (Frame)
- **Transmission physique** : **Bits**

**Note** : Ces termes sont importants ! Les professionnels disent "segment TCP", "paquet IP", "trame Ethernet" - pas "paquet TCP" ou "segment Ethernet".

---

## Le processus d'encapsulation (émetteur)

Suivons un exemple concret : **vous envoyez un email**.

### Vue d'ensemble du processus descendant

```
Vous écrivez un email : "Bonjour !"
         ↓
┌────────────────────────────────────────┐
│  COUCHE APPLICATION                    │
│  Email complet avec en-têtes           │
│  [Données : "Bonjour !"]               │
└────────────────────────────────────────┘
         ↓ Ajout en-tête TCP
┌────────────────────────────────────────┐
│  COUCHE TRANSPORT                      │
│  [En-tête TCP | Données]               │
│  └─ Port 25 (SMTP)                     │
└────────────────────────────────────────┘
         ↓ Ajout en-tête IP
┌────────────────────────────────────────┐
│  COUCHE INTERNET                       │
│  [En-tête IP | En-tête TCP | Données]  │
│  └─ Adresse IP destination             │
└────────────────────────────────────────┘
         ↓ Ajout en-tête Ethernet
┌────────────────────────────────────────┐
│  COUCHE ACCÈS RÉSEAU                   │
│  [Eth | IP | TCP | Données | CRC]      │
│  └─ Adresse MAC destination            │
└────────────────────────────────────────┘
         ↓ Conversion en bits
    101010011010101... (signaux physiques)
```

### Étape par étape détaillée

#### Étape 1 : Couche Application

**Vous** : Créez le message
```
De : vous@example.com
À : ami@example.com
Sujet : Salut
Corps : Bonjour !
```

**Résultat** :
```
┌─────────────────────────────────┐
│  Données de l'application       │
│  (Email au format SMTP)         │
└─────────────────────────────────┘
```

**Ce que contient** : Les données pures que vous voulez envoyer

#### Étape 2 : Couche Transport (TCP)

La couche Transport **segmente** les données et ajoute un **en-tête TCP**.

**En-tête TCP ajouté** :
```
┌──────────────────────────────────────────────────────┐
│  En-tête TCP (20-60 octets)                          │
│  ├─ Port source : 54321 (aléatoire)                  │
│  ├─ Port destination : 25 (SMTP)                     │
│  ├─ Numéro de séquence : 1000                        │
│  ├─ Numéro d'acquittement : 0                        │
│  ├─ Flags : SYN, ACK...                              │
│  ├─ Fenêtre : 65535                                  │
│  └─ Checksum : 0xAB12                                │
├──────────────────────────────────────────────────────┤
│  Données de l'application                            │
└──────────────────────────────────────────────────────┘
```

**Nom** : C'est maintenant un **segment TCP**

**Pourquoi cet en-tête ?**
- **Ports** : Identifier les applications (émetteur et récepteur)
- **Séquence** : Numéroter les segments pour les remettre dans l'ordre
- **Checksum** : Détecter les erreurs de transmission
- **Flags** : Contrôler la connexion (SYN, ACK, FIN...)

#### Étape 3 : Couche Internet (IP)

La couche Internet ajoute un **en-tête IP** pour le routage.

```
┌──────────────────────────────────────────────────────┐
│  En-tête IP (20-60 octets)                           │
│  ├─ Version : IPv4                                   │
│  ├─ Longueur totale : 576 octets                     │
│  ├─ TTL : 64 (durée de vie)                          │
│  ├─ Protocole : 6 (TCP)                              │
│  ├─ Adresse IP source : 192.168.1.100                │
│  ├─ Adresse IP destination : 93.184.216.34           │
│  └─ Checksum : 0x1F3A                                │
├──────────────────────────────────────────────────────┤
│  En-tête TCP                                         │
├──────────────────────────────────────────────────────┤
│  Données de l'application                            │
└──────────────────────────────────────────────────────┘
```

**Nom** : C'est maintenant un **paquet IP**

**Pourquoi cet en-tête ?**
- **Adresses IP** : Identifier l'émetteur et le destinataire final
- **TTL** : Éviter les boucles infinies (décrémenté à chaque routeur)
- **Protocole** : Indiquer quel protocole de transport est utilisé (TCP=6, UDP=17)
- **Checksum** : Détecter les erreurs dans l'en-tête IP

#### Étape 4 : Couche Accès réseau (Ethernet)

La couche Accès réseau ajoute un **en-tête Ethernet** ET un **pied de page CRC**.

```
┌──────────────────────────────────────────────────────────────┐
│  En-tête Ethernet (14 octets)                                │
│  ├─ Préambule : synchronisation                              │
│  ├─ Adresse MAC destination : 00:1A:2B:3C:4D:5E              │
│  ├─ Adresse MAC source : AA:BB:CC:DD:EE:FF                   │
│  └─ Type : 0x0800 (IPv4)                                     │
├──────────────────────────────────────────────────────────────┤
│  En-tête IP                                                  │
├──────────────────────────────────────────────────────────────┤
│  En-tête TCP                                                 │
├──────────────────────────────────────────────────────────────┤
│  Données de l'application                                    │
├──────────────────────────────────────────────────────────────┤
│  CRC (Pied de page - 4 octets)                               │
│  └─ Checksum Ethernet : 0xABCD1234                           │
└──────────────────────────────────────────────────────────────┘
```

**Nom** : C'est maintenant une **trame Ethernet**

**Pourquoi cet en-tête ?**
- **Adresses MAC** : Identifier les cartes réseau physiques sur le réseau local
- **Type** : Indiquer quel protocole de niveau supérieur (IPv4, IPv6, ARP...)
- **CRC** : Détecter les erreurs de transmission sur le câble

#### Étape 5 : Couche Physique

La trame est convertie en **bits** (0 et 1) puis en **signaux physiques** :
- Signaux électriques (câble cuivre)
- Impulsions lumineuses (fibre optique)
- Ondes radio (Wi-Fi)

```
Trame Ethernet : [010100110101011001...]
         ↓
Signaux électriques : ━━┓┏━━┓┏━━━━┓...
                        ┗┛  ┗┛    ┗
```

---

## Visualisation complète de l'encapsulation

```
NIVEAU APPLICATION
┌─────────────────────────────────────────────────┐
│  "Bonjour !"                                    │
└─────────────────────────────────────────────────┘

                    ↓ Ajout en-tête TCP

NIVEAU TRANSPORT (Segment TCP)
┌──────────┬──────────────────────────────────────┐
│  TCP     │  "Bonjour !"                         │
│  Header  │                                      │
└──────────┴──────────────────────────────────────┘

                    ↓ Ajout en-tête IP

NIVEAU INTERNET (Paquet IP)
┌──────────┬──────────┬───────────────────────────┐
│  IP      │  TCP     │  "Bonjour !"              │
│  Header  │  Header  │                           │
└──────────┴──────────┴───────────────────────────┘

                    ↓ Ajout en-tête + CRC Ethernet

NIVEAU ACCÈS RÉSEAU (Trame Ethernet)
┌──────────┬──────────┬──────────┬────────────┬───────┐
│ Ethernet │  IP      │  TCP     │ "Bonjour!" │  CRC  │
│ Header   │  Header  │  Header  │            │       │
└──────────┴──────────┴──────────┴────────────┴───────┘

                    ↓ Conversion en bits

NIVEAU PHYSIQUE
10101001101010110010101011010101010110...
━━┓┏━━┓┏━━━━┓┏━┓┏━━━━┓... (signaux électriques)
  ┗┛  ┗┛    ┗┛ ┗┛
```

**Observation importante** : Les données originales ("Bonjour !") sont **intactes** à chaque étape. On ne fait qu'**ajouter** des informations autour.

---

## Le processus de décapsulation (récepteur)

À l'arrivée, le processus **inverse** se produit : on **retire** les en-têtes couche par couche.

### Vue d'ensemble du processus ascendant

```
Signaux électriques reçus sur le câble
         ↓ Conversion en bits
┌────────────────────────────────────────┐
│  COUCHE ACCÈS RÉSEAU                   │
│  [Eth | IP | TCP | Données | CRC]      │
│  Vérifie CRC, retire en-tête Ethernet  │
└────────────────────────────────────────┘
         ↓
┌────────────────────────────────────────┐
│  COUCHE INTERNET                       │
│  [IP | TCP | Données]                  │
│  Vérifie IP destination, retire en-tête│
└────────────────────────────────────────┘
         ↓
┌────────────────────────────────────────┐
│  COUCHE TRANSPORT                      │
│  [TCP | Données]                       │
│  Vérifie port, réassemble, retire TCP  │
└────────────────────────────────────────┘
         ↓
┌────────────────────────────────────────┐
│  COUCHE APPLICATION                    │
│  [Données]                             │
│  Affiche "Bonjour !"                   │
└────────────────────────────────────────┘
```

### Étape par étape détaillée

#### Étape 1 : Réception physique

Les signaux électriques/lumineux sont **convertis en bits**.

```
━━┓┏━━┓┏━━━━┓... → 10101001101010110...
  ┗┛  ┗┛    ┗
```

#### Étape 2 : Couche Accès réseau (Ethernet)

**Actions** :
1. **Vérifier le CRC** : Les données sont-elles corrompues ?
2. **Vérifier l'adresse MAC** : Cette trame est-elle pour moi ?
3. **Lire le champ Type** : Quel protocole de niveau supérieur ? (IPv4, IPv6, ARP...)
4. **Retirer l'en-tête Ethernet et le CRC**

```
Avant :
┌──────────┬──────────┬──────────┬────────────┬───────┐
│ Ethernet │  IP      │  TCP     │ "Bonjour!" │  CRC  │
└──────────┴──────────┴──────────┴────────────┴───────┘

Après (passe à la couche Internet) :
┌──────────┬──────────┬────────────┐
│  IP      │  TCP     │ "Bonjour!" │
└──────────┴──────────┴────────────┘
```

**Si l'adresse MAC ne correspond pas** : La trame est **ignorée** (sauf en mode promiscuous).

#### Étape 3 : Couche Internet (IP)

**Actions** :
1. **Vérifier le checksum IP** : L'en-tête est-il corrompu ?
2. **Vérifier l'adresse IP de destination** : Ce paquet est-il pour moi ?
3. **Vérifier le TTL** : Si TTL = 0, détruire le paquet
4. **Lire le champ Protocole** : TCP (6) ou UDP (17) ?
5. **Retirer l'en-tête IP**

```
Avant :
┌──────────┬──────────┬────────────┐
│  IP      │  TCP     │ "Bonjour!" │
└──────────┴──────────┴────────────┘

Après (passe à la couche Transport) :
┌──────────┬────────────┐
│  TCP     │ "Bonjour!" │
└──────────┴────────────┘
```

**Si l'adresse IP ne correspond pas** :
- Si c'est un routeur : **transférer** vers le prochain saut
- Si c'est la destination finale mais mauvaise IP : **ignorer** ou envoyer ICMP "destination unreachable"

#### Étape 4 : Couche Transport (TCP)

**Actions** :
1. **Vérifier le checksum TCP** : Le segment est-il corrompu ?
2. **Vérifier le port de destination** : Quelle application doit recevoir ? (ex: port 25 = serveur SMTP)
3. **Vérifier le numéro de séquence** : Remettre dans l'ordre si nécessaire
4. **Envoyer un acquittement (ACK)** : Confirmer la réception
5. **Réassembler** si les données étaient fragmentées
6. **Retirer l'en-tête TCP**

```
Avant :
┌──────────┬────────────┐
│  TCP     │ "Bonjour!" │
└──────────┴────────────┘

Après (passe à la couche Application) :
┌────────────┐
│ "Bonjour!" │
└────────────┘
```

**Si le port ne correspond à aucune application en écoute** : Envoyer un paquet TCP RST (reset) pour indiquer que le port est fermé.

#### Étape 5 : Couche Application

**Actions** :
1. **Interpréter les données** selon le protocole attendu (SMTP, HTTP, etc.)
2. **Traiter la requête**
3. **Afficher à l'utilisateur**

```
Résultat final :
┌─────────────────────────────────┐
│  De : vous@example.com          │
│  À : ami@example.com            │
│  Sujet : Salut                  │
│  Corps : Bonjour !              │
└─────────────────────────────────┘
```

L'email est maintenant lisible ! 🎉

---

## Analogie complète : le système postal

Cette analogie illustre parfaitement l'encapsulation et la décapsulation.

### Envoi (Encapsulation)

**1. Vous écrivez une lettre** (Application)
```
"Cher Jean, Comment vas-tu ?"
```

**2. Vous la mettez dans une enveloppe** (Transport)
```
┌─────────────────────────────────┐
│ De : Marie Martin               │
│ À : Jean Dupont                 │
│ ┌─────────────────────────────┐ │
│ │ "Cher Jean,                 │ │
│ │  Comment vas-tu ?"          │ │
│ └─────────────────────────────┘ │
└─────────────────────────────────┘
```

**3. Vous la mettez dans un colis avec l'adresse complète** (Internet)
```
┌─────────────────────────────────────┐
│ Adresse destination :               │
│ Jean Dupont                         │
│ 15 rue de la Paix                   │
│ 75001 Paris, FRANCE                 │
│                                     │
│ Adresse expéditeur :                │
│ Marie Martin                        │
│ 3 av. Victor Hugo                   │
│ 69000 Lyon, FRANCE                  │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ Enveloppe avec lettre           │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**4. Le colis est mis dans un sac postal avec étiquette** (Accès réseau)
```
┌─────────────────────────────────────┐
│ SAC POSTAL                          │
│ Centre de tri : LYON-01             │
│ Destination : PARIS-CENTRE          │
│ Code-barres : ||||||||||            │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ Colis avec adresse              │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**5. Le sac est transporté physiquement** (Physique)
```
🚛 Camion postal → Train → Avion → Camion
```

### Réception (Décapsulation)

**1. Le sac postal arrive au centre de tri de Paris** (Accès réseau)
- On vérifie le code-barres
- On retire le sac postal
- On sort le colis

**2. Le colis est acheminé à l'adresse** (Internet)
- On lit : "15 rue de la Paix, 75001 Paris"
- On route vers ce quartier
- On retire l'emballage du colis

**3. Le facteur dépose l'enveloppe** (Transport)
- On vérifie : "Jean Dupont"
- On dépose dans la bonne boîte aux lettres
- On retire l'enveloppe

**4. Jean lit la lettre** (Application)
- Il ouvre l'enveloppe
- Il lit : "Cher Jean, Comment vas-tu ?"

**Chaque "couche" (sac postal, colis, enveloppe) a servi son rôle puis a été retirée !**

---

## L'overhead : le coût de l'encapsulation

Chaque en-tête ajouté **augmente la taille** des données transmises. C'est ce qu'on appelle l'**overhead** (surcharge).

### Calcul de l'overhead typique

Imaginons que vous envoyez **100 octets** de données.

```
Données originales :                    100 octets

+ En-tête TCP :                         20 octets (minimum)
+ En-tête IP :                          20 octets (minimum)
+ En-tête Ethernet :                    14 octets
+ CRC Ethernet :                         4 octets
─────────────────────────────────────────────────
Total transmis sur le câble :           158 octets
```

**Overhead = 58 octets** pour transporter 100 octets de données utiles !

**Pourcentage d'overhead** : 58/158 = **36,7% de surcharge** 😱

### Exemple avec un petit message

Envoi d'un simple "OK" (2 octets) :

```
Données :                                 2 octets
En-têtes (TCP + IP + Ethernet + CRC) :   58 octets
─────────────────────────────────────────────────
Total :                                  60 octets
```

**Pour envoyer 2 octets, on transmet 60 octets !** C'est 30 fois plus ! 🤯

### Pourquoi c'est acceptable ?

**1. Efficacité avec des données volumineuses**

Pour 10 000 octets de données :
```
Données :                             10 000 octets
En-têtes :                                58 octets
─────────────────────────────────────────────────
Total :                               10 058 octets
Overhead : 0,58% seulement !
```

**2. Le prix de la fiabilité et de la flexibilité**

Les en-têtes permettent :
- Le routage sur Internet
- La détection d'erreurs
- La livraison fiable
- Le multiplexage des applications

Sans eux, pas de réseau fonctionnel !

**3. Optimisations modernes**

- **Jumbo frames** : Trames Ethernet jusqu'à 9000 octets (au lieu de 1500)
- **Compression des en-têtes** : HTTP/2, QUIC
- **Coalescence** : Regrouper plusieurs petits messages

---

## Le voyage complet : exemple concret

Suivons un paquet du début à la fin pour tout comprendre.

### Scénario

**Vous** (Paris) visitez **www.example.com** (serveur à New York).

### Émission (votre ordinateur à Paris)

```
VOUS tapez dans le navigateur : https://www.example.com
```

**Couche Application (HTTPS)** :
```
GET / HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
```

**Couche Transport (TCP)** :
```
┌──────────────────────────────────┐
│ Port src: 54321 → dst: 443       │
│ Seq: 1000, Ack: 5000             │
│ ────────────────────────────     │
│ GET / HTTP/1.1...                │
└──────────────────────────────────┘
Segment TCP créé
```

**Couche Internet (IP)** :
```
┌──────────────────────────────────┐
│ 192.168.1.100 → 93.184.216.34    │
│ TTL: 64, Protocol: TCP           │
│ ────────────────────────────     │
│ [Segment TCP]                    │
└──────────────────────────────────┘
Paquet IP créé
```

**Couche Accès réseau (Ethernet)** :
```
┌──────────────────────────────────┐
│ MAC: AA:BB:CC:DD:EE:FF →         │
│      00:11:22:33:44:55           │
│ ────────────────────────────     │
│ [Paquet IP]                      │
│ ────────────────────────────     │
│ CRC: 0xABCD1234                  │
└──────────────────────────────────┘
Trame Ethernet créée
```

**Transmission** : Bits sur le câble vers votre routeur domestique.

### Transit (à travers Internet)

**Votre routeur** :
1. **Décapsule** jusqu'à IP
2. Voit : destination = 93.184.216.34 (pas sur le réseau local)
3. Consulte sa table de routage
4. **Ré-encapsule** dans une nouvelle trame pour le routeur suivant
5. Envoie vers le FAI

**Routeur du FAI** :
1. Même processus
2. Route vers l'Europe/Atlantique

**Routeurs intermédiaires** (peut-être 10-15 sauts) :
- Chacun **décapsule** jusqu'à IP
- Décrémente le TTL
- Consulte sa table de routage
- **Ré-encapsule** dans une nouvelle trame
- Envoie au prochain routeur

**Important** : Les couches Transport et Application ne sont **jamais touchées** par les routeurs intermédiaires ! Seuls les en-têtes Ethernet et IP changent.

```
À chaque routeur :

Trame reçue :
[Eth-A | IP | TCP | Data | CRC-A]
        ↓ Décapsulation
[IP | TCP | Data]
        ↓ Nouvelle encapsulation
[Eth-B | IP | TCP | Data | CRC-B]

IP/TCP/Data restent identiques !
Seul Ethernet change (nouvelles adresses MAC)
```

### Réception (serveur à New York)

**Dernier routeur** :
- Voit que 93.184.216.34 est sur son réseau local
- Utilise ARP pour trouver l'adresse MAC du serveur
- Envoie la trame au serveur

**Serveur www.example.com** :

**Couche Accès réseau** :
- Reçoit la trame
- Vérifie MAC : C'est pour moi ? ✅
- Vérifie CRC : Pas d'erreurs ? ✅
- Retire l'en-tête Ethernet

**Couche Internet** :
- Vérifie IP : 93.184.216.34, c'est moi ! ✅
- Retire l'en-tête IP

**Couche Transport** :
- Vérifie port : 443 (HTTPS) ✅
- Vérifie séquence, envoie ACK
- Retire l'en-tête TCP

**Couche Application** :
- Reçoit la requête HTTP
- Traite : "GET / HTTP/1.1"
- Prépare la réponse (la page HTML)

**Le processus inverse commence** pour envoyer la réponse !

---

## Communication horizontale vs verticale

### Communication verticale (réelle)

Les données **descendent** physiquement la pile OSI/TCP-IP côté émetteur, et **remontent** côté récepteur.

```
Émetteur                    Récepteur

Application  →→→→→→→→→→→→→  Application
    ↓                           ↑
Transport    →→→→→→→→→→→→→  Transport
    ↓                           ↑
Internet     →→→→→→→→→→→→→  Internet
    ↓                           ↑
Accès réseau →→→→→→→→→→→→→  Accès réseau
    ↓                           ↑
  Physique   ═══════════════  Physique
```

### Communication horizontale (logique)

Chaque couche "communique logiquement" avec sa couche homologue via son en-tête.

```
Émetteur                              Récepteur

Application ←- - - - - - - - - - →  Application
  (HTTP parle à HTTP)

Transport   ←- - - - - - - - - - →  Transport
  (TCP parle à TCP via en-tête TCP)

Internet    ←- - - - - - - - - - →  Internet
  (IP parle à IP via en-tête IP)

Accès réseau ←- - - - - - - - - →  Accès réseau
  (Ethernet parle à Ethernet)
```

**Les couches croient communiquer directement entre elles**, mais en réalité, elles passent par toutes les couches inférieures !

**Analogie** : Deux PDG qui s'écrivent. Ils croient communiquer directement, mais en réalité :
- Le PDG A écrit → son assistant met dans une enveloppe → le courrier interne l'achemine → la poste le transporte → le courrier interne B le livre → l'assistant B ouvre l'enveloppe → le PDG B lit.

Les PDG "communiquent logiquement", mais physiquement, beaucoup d'intermédiaires interviennent !

---

## Principes importants de l'encapsulation

### 1. Indépendance des couches

Chaque couche **ne connaît pas** les détails des autres couches.

- TCP ne sait pas si IP va router via Ethernet, Wi-Fi ou 4G
- IP ne sait pas si c'est TCP ou UDP au-dessus
- Ethernet ne se préoccupe pas de ce qu'il transporte

**Avantage** : Modularité totale. On peut changer une couche sans affecter les autres.

### 2. Transparence

Les couches supérieures sont **transparentes** pour les couches inférieures.

Pour Ethernet, peu importe que ce soit un email, une vidéo ou un jeu : c'est juste des octets à transporter.

### 3. Les données restent intactes

L'encapsulation n'**altère jamais** les données originales. On ajoute simplement des informations **autour**.

```
Données originales : "Bonjour"

Après encapsulation :
[En-têtes... | "Bonjour" | ...Pieds]
              └─ Toujours intact ─┘
```

### 4. Récursivité

Le processus est **récursif** : ce qui est "données" pour une couche est "PDU complet" pour la couche inférieure.

```
Couche N+1 : [Données]
Couche N   : [En-tête | Données]
                        └─ Vu comme "données" par la couche N-1
Couche N-1 : [En-tête | En-tête N | Données | Pied]
```

---

## Cas particuliers et optimisations

### Fragmentation IP

Si un paquet IP est **trop gros** pour la trame Ethernet (MTU = Maximum Transmission Unit, généralement 1500 octets), IP le **fragmente**.

```
Paquet IP trop gros (3000 octets)
         ↓ Fragmentation
Fragment 1 (1500 octets) + Fragment 2 (1500 octets)
         ↓ Transmission
         ↓ Réassemblage à destination
Paquet IP reconstruit (3000 octets)
```

### Tunneling

Un protocole de même niveau ou supérieur peut être **encapsulé** dans un autre protocole.

**Exemple : VPN (Virtual Private Network)**

```
┌──────────────────────────────────────────────────┐
│ Paquet IP public                                 │
│  ├─ IP publique source                           │
│  ├─ IP publique destination (serveur VPN)        │
│  └─ ┌──────────────────────────────────────────┐ │
│    │ Paquet IP privé (encapsulé/chiffré)       │ │
│    │  ├─ IP privée source : 10.0.0.5           │ │
│    │  ├─ IP privée destination : 10.0.0.10     │ │
│    │  └─ Données                               │ │
│    └───────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

Le paquet IP privé est les **"données"** du paquet IP public !

### Absence d'encapsulation : Communication locale

Sur un réseau local (même LAN), parfois on peut communiquer avec juste Ethernet + IP, sans passer par tous les routeurs.

**Exemple** : Deux ordinateurs sur le même switch Ethernet.

```
PC-A ←→ Switch ←→ PC-B

Seules les couches Ethernet + IP + Transport + Application sont utilisées.
Pas de routage intermédiaire.
```

---

## Outils pour observer l'encapsulation

### Wireshark

**Wireshark** est un analyseur de paquets qui vous permet de **voir** l'encapsulation en action !

**Exemple de capture** :
```
Frame 1: 74 bytes on wire
  ├─ Ethernet II
  │   ├─ Destination: 00:1a:2b:3c:4d:5e
  │   ├─ Source: aa:bb:cc:dd:ee:ff
  │   └─ Type: IPv4 (0x0800)
  ├─ Internet Protocol Version 4
  │   ├─ Source: 192.168.1.100
  │   ├─ Destination: 93.184.216.34
  │   └─ Protocol: TCP (6)
  ├─ Transmission Control Protocol
  │   ├─ Source Port: 54321
  │   ├─ Destination Port: 443
  │   └─ Flags: [SYN]
  └─ [Data: Application Layer Protocol]
```

**Chaque niveau d'indentation** montre une couche d'encapsulation !

### tcpdump

Outil en ligne de commande pour capturer et analyser les paquets.

```bash
$ sudo tcpdump -i eth0 -nn -X

IP 192.168.1.100.54321 > 93.184.216.34.443: Flags [S]
  0x0000:  4500 003c 1c46 4000 4006 0000 c0a8 0164  E..<.F@.@......d
  0x0010:  5db8 d822 d431 01bb 0000 0001 0000 0000  ]..".1..........
```

Vous voyez les **octets bruts** de l'encapsulation !

---

## Résumé visuel complet

```
┌─────────────────────────────────────────────────────────────┐
│                    ENCAPSULATION (Émission)                 │
└─────────────────────────────────────────────────────────────┘

  Application       [Données]
       ↓ +En-tête
  Transport         [TCP | Données]                 (Segment)
       ↓ +En-tête
  Internet          [IP | TCP | Données]            (Paquet)
       ↓ +En-tête +Pied
  Accès réseau      [Eth | IP | TCP | Data | CRC]  (Trame)
       ↓ Conversion
  Physique          101010110101010...              (Bits)

═══════════════════════════════════════════════════════════════
                    TRANSMISSION SUR LE RÉSEAU
═══════════════════════════════════════════════════════════════

  Physique          101010110101010...              (Bits)
       ↓ Conversion
  Accès réseau      [Eth | IP | TCP | Data | CRC]  (Trame)
       ↓ -En-tête -Pied
  Internet          [IP | TCP | Données]            (Paquet)
       ↓ -En-tête
  Transport         [TCP | Données]                 (Segment)
       ↓ -En-tête
  Application       [Données]

┌─────────────────────────────────────────────────────────────┐
│                  DÉCAPSULATION (Réception)                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Conclusion

L'encapsulation et la décapsulation sont les **mécanismes fondamentaux** qui font fonctionner les réseaux en couches.

**Les principes clés** :

🧅 **Poupées russes** : Chaque couche emballe les données de la couche supérieure

📦 **En-têtes modulaires** : Chaque couche ajoute ses propres informations de contrôle

🔄 **Processus symétrique** : Encapsulation à l'envoi, décapsulation à la réception

🔗 **Communication logique** : Chaque couche "parle" à sa couche homologue

🎯 **Indépendance** : Les couches ne connaissent pas les détails des autres

📊 **Overhead acceptable** : Le prix à payer pour la flexibilité et la fiabilité

**Pourquoi c'est important** :

✅ Permet la **modularité** : On peut changer une couche sans tout reconstruire

✅ Permet l'**interopérabilité** : Différentes technologies peuvent coexister

✅ Permet la **flexibilité** : Le même protocole fonctionne sur différents médias

✅ Facilite le **dépannage** : On peut isoler les problèmes par couche

Maintenant que vous comprenez l'encapsulation, vous comprenez **vraiment** comment les données voyagent sur un réseau ! Chaque fois que vous :
- Envoyez un email
- Chargez une page web
- Streamez une vidéo
- Jouez en ligne

...vos données subissent ce processus d'encapsulation/décapsulation des dizaines ou centaines de fois par seconde ! 🚀

---

**À retenir** :
- 📦 **Encapsulation** = Ajout d'en-têtes à chaque couche descendante
- 🔓 **Décapsulation** = Retrait d'en-têtes à chaque couche montante
- 📝 Chaque couche a son **nom** : Segment (TCP), Paquet (IP), Trame (Ethernet), Bits
- 🎭 **Communication logique** entre couches homologues
- 📊 **Overhead** : 40-60 octets d'en-têtes minimum (TCP+IP+Ethernet)
- 🧩 Les données originales restent **intactes**, on ajoute juste autour
- 🔧 Essentiel pour comprendre le **dépannage réseau**

**Prochaine étape** : Maintenant que nous maîtrisons les fondamentaux, plongeons dans la première couche concrète : la couche Accès réseau ! →

⏭️ [2. La couche Accès réseau](/02-couche-acces-reseau/README.md)
