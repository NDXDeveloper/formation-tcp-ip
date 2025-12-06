🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.9 ICMP : diagnostic et messages de contrôle

## Introduction : le messager du réseau

Imaginez que vous envoyez une lettre importante. Quelques jours plus tard, vous recevez un message du service postal : "Destinataire inconnu à cette adresse" ou "Colis trop volumineux pour la boîte aux lettres". Ces messages d'**information et d'erreur** sont essentiels pour comprendre ce qui s'est passé avec votre envoi.

Sur Internet, c'est exactement le rôle d'**ICMP** (Internet Control Message Protocol). C'est le **système de notification** du réseau qui permet aux routeurs et aux hôtes de **communiquer des informations de contrôle et des erreurs**.

**ICMP est le protocole derrière les commandes** :
- `ping` (tester si un hôte est joignable)
- `traceroute` (voir le chemin suivi par les paquets)
- Messages d'erreur réseau (destination inaccessible, etc.)

Sans ICMP, Internet serait **silencieux** : vous ne sauriez jamais pourquoi vos paquets n'arrivent pas à destination !

## Qu'est-ce qu'ICMP ?

### Définition et positionnement

**ICMP** = Internet Control Message Protocol

```
Couche OSI : Couche 3 (Réseau)
Couche TCP/IP : Couche Internet

ICMP fait partie intégrante d'IP,
mais c'est un protocole à part entière.
```

**Caractéristiques** :
- Protocole numéro **1** dans l'en-tête IP (champ Protocol)
- Ne transporte **pas de données utilisateur**
- Utilisé pour les **messages de contrôle et d'erreur**
- Implémenté dans **tous les systèmes IP**

### Positionnement dans la pile

```
┌─────────────────────────────────┐
│     APPLICATION                 │
├─────────────────────────────────┤
│     TRANSPORT (TCP/UDP)         │
├─────────────────────────────────┤
│     INTERNET (IP)               │
│       ┌──────────┐              │
│       │   ICMP   │←─── Couche compagnon d'IP
│       └──────────┘              │
├─────────────────────────────────┤
│     ACCÈS RÉSEAU                │
└─────────────────────────────────┘
```

**ICMP n'est pas au-dessus d'IP, mais À CÔTÉ d'IP.**

### Pourquoi ICMP existe-t-il ?

**Problème** : IP est un protocole **"best effort"** (au mieux)
```
IP dit : "Je vais essayer de livrer ton paquet, mais :
  - Je ne garantis rien
  - Je ne te préviens pas si ça échoue
  - Je suis silencieux en cas d'erreur"
```

**Solution** : ICMP ajoute une **couche de communication**
```
ICMP dit : "Je vais t'informer si :
  - La destination est inaccessible
  - Le paquet est trop gros
  - La route est congestionée
  - Le TTL est expiré
  - Et plein d'autres problèmes..."
```

**Analogie** : IP est comme un livreur silencieux. ICMP est comme un système de suivi de colis qui vous envoie des SMS : "Colis livré", "Adresse introuvable", "Destinataire absent".

## Structure d'un message ICMP

### Format général

Un message ICMP est **encapsulé dans un paquet IP** :

```
┌──────────────────────────────────────────┐
│      EN-TÊTE IP                          │
│  (Protocol = 1 pour ICMP)                │
├──────────────────────────────────────────┤
│      MESSAGE ICMP                        │
│  ┌────────────────────────────────────┐  │
│  │ Type (8 bits)                      │  │
│  ├────────────────────────────────────┤  │
│  │ Code (8 bits)                      │  │
│  ├────────────────────────────────────┤  │
│  │ Checksum (16 bits)                 │  │
│  ├────────────────────────────────────┤  │
│  │ Contenu (variable selon le type)   │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

### Les champs ICMP

**Type (8 bits)** :
```
Indique le TYPE de message ICMP

Exemples :
  0  = Echo Reply (réponse au ping)
  3  = Destination Unreachable (destination inaccessible)
  8  = Echo Request (demande de ping)
  11 = Time Exceeded (temps dépassé)
```

**Code (8 bits)** :
```
Précise le SOUS-TYPE du message (dépend du Type)

Exemple pour Type 3 (Destination Unreachable) :
  Code 0 = Net Unreachable (réseau inaccessible)
  Code 1 = Host Unreachable (hôte inaccessible)
  Code 3 = Port Unreachable (port inaccessible)
```

**Checksum (16 bits)** :
```
Somme de contrôle pour vérifier l'intégrité du message ICMP
```

**Contenu (variable)** :
```
Dépend du type de message
Peut contenir :
  - Un identifiant
  - Un numéro de séquence
  - Les premiers octets du paquet qui a causé l'erreur
  - Etc.
```

## Types de messages ICMP principaux

ICMP définit de nombreux types de messages. Voici les plus importants :

```
┌──────┬────────────────────────────┬──────────────────────┐
│ Type │ Nom                        │ Usage                │
├──────┼────────────────────────────┼──────────────────────┤
│  0   │ Echo Reply                 │ Réponse au ping      │
│  3   │ Destination Unreachable    │ Destination invalide │
│  4   │ Source Quench (obsolète)   │ Contrôle de flux     │
│  5   │ Redirect                   │ Redirection de route │
│  8   │ Echo Request               │ Demande de ping      │
│  9   │ Router Advertisement       │ Annonce de routeur   │
│  10  │ Router Solicitation        │ Recherche de routeur │
│  11  │ Time Exceeded              │ TTL expiré           │
│  12  │ Parameter Problem          │ En-tête IP invalide  │
│  13  │ Timestamp Request          │ Demande d'horodatage │
│  14  │ Timestamp Reply            │ Réponse d'horodatage │
└──────┴────────────────────────────┴──────────────────────┘
```

Nous allons explorer les plus importants en détail.

## Echo Request & Echo Reply : la base du ping

### Type 8 : Echo Request (demande)

**Rôle** : Demander à un hôte : "Es-tu vivant ?"

**Structure** :

```
┌────────────────────────────────────┐
│ Type : 8 (Echo Request)            │
├────────────────────────────────────┤
│ Code : 0                           │
├────────────────────────────────────┤
│ Checksum                           │
├────────────────────────────────────┤
│ Identifier (16 bits)               │
├────────────────────────────────────┤
│ Sequence Number (16 bits)          │
├────────────────────────────────────┤
│ Data (optionnel, taille variable)  │
└────────────────────────────────────┘
```

**Champs spécifiques** :
- **Identifier** : Identifie la session de ping (permet de distinguer plusieurs pings simultanés)
- **Sequence Number** : Numéro séquentiel (incrémenté pour chaque paquet)
- **Data** : Données de remplissage (généralement 32 ou 56 octets)

### Type 0 : Echo Reply (réponse)

**Rôle** : Répondre "Oui, je suis vivant !"

**Structure** : Identique à Echo Request, mais Type = 0

```
┌────────────────────────────────────┐
│ Type : 0 (Echo Reply)              │
├────────────────────────────────────┤
│ Code : 0                           │
├────────────────────────────────────┤
│ Checksum                           │
├────────────────────────────────────┤
│ Identifier (même que la requête)   │
├────────────────────────────────────┤
│ Sequence Number (même)             │
├────────────────────────────────────┤
│ Data (les mêmes données)           │
└────────────────────────────────────┘
```

**Principe** : Le destinataire **renvoie exactement les mêmes données** avec Type changé de 8 à 0.

## La commande ping en détail

### Qu'est-ce que ping ?

**ping** = Packet INternet Groper (sondeur de réseau)

C'est l'outil de **diagnostic réseau le plus basique et le plus utilisé**.

**Usage** : Vérifier si un hôte est joignable et mesurer le temps de réponse.

### Fonctionnement du ping

**Étape par étape** :

```
Vous tapez : ping google.com

┌─────────────────────────────────────────────────────────┐
│ 1. Résolution DNS                                       │
│    google.com → 142.250.185.46                          │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 2. Création du paquet ICMP Echo Request                 │
│    Type: 8, Code: 0                                     │
│    Identifier: 12345 (aléatoire)                        │
│    Sequence: 1                                          │
│    Data: 56 octets de remplissage                       │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 3. Encapsulation dans un paquet IP                      │
│    Source: Votre IP                                     │
│    Destination: 142.250.185.46                          │
│    Protocol: 1 (ICMP)                                   │
│    TTL: 64                                              │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 4. Envoi sur le réseau                                  │
│    Timestamp t1 enregistré                              │
└─────────────────────────────────────────────────────────┘
                      ↓
        ... voyage à travers Internet ...
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 5. Réception par le serveur Google                      │
│    Analyse du paquet ICMP                               │
│    Type 8 détecté = Echo Request                        │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 6. Création de la réponse                               │
│    Type: 0 (Echo Reply)                                 │
│    Identifier: 12345 (même)                             │
│    Sequence: 1 (même)                                   │
│    Data: mêmes 56 octets                                │
└─────────────────────────────────────────────────────────┘
                      ↓
        ... retour vers vous ...
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 7. Réception de la réponse                              │
│    Timestamp t2 enregistré                              │
│    RTT = t2 - t1 = 15 ms                                │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│ 8. Affichage                                            │
│    64 bytes from 142.250.185.46: icmp_seq=1 ttl=56      │
│    time=15.2 ms                                         │
└─────────────────────────────────────────────────────────┘
```

### Exemple de sortie ping

```bash
$ ping google.com

PING google.com (142.250.185.46) 56(84) bytes of data.
64 bytes from par21s20-in-f14.1e100.net (142.250.185.46): icmp_seq=1 ttl=117 time=14.2 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.185.46): icmp_seq=2 ttl=117 time=15.1 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.185.46): icmp_seq=3 ttl=117 time=14.8 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.185.46): icmp_seq=4 ttl=117 time=14.5 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 14.230/14.650/15.108/0.331 ms
```

**Décryptage** :

```
64 bytes : Taille du paquet ICMP reçu (56 data + 8 header)
icmp_seq=1 : Numéro de séquence (permet de détecter les pertes)
ttl=117 : TTL restant (nombre de sauts possibles restants)
time=14.2 ms : RTT (Round Trip Time) = temps aller-retour

0% packet loss : Aucun paquet perdu (fiabilité)
rtt min/avg/max : Temps min/moyen/max
mdev : Écart-type (stabilité de la connexion)
```

### Interprétation des résultats

**Temps de réponse (RTT)** :

```
< 1 ms     : Excellent (réseau local)
1-20 ms    : Très bon (même pays)
20-50 ms   : Bon (Europe)
50-100 ms  : Acceptable (intercontinental)
100-200 ms : Moyen (longue distance)
> 200 ms   : Élevé (peut impacter les applications temps-réel)
> 1000 ms  : Très élevé (problème probable)
```

**Perte de paquets** :

```
0%       : Parfait
< 1%     : Excellent
1-5%     : Acceptable
5-10%    : Problématique (dégradation perceptible)
> 10%    : Grave (connexion instable)
100%     : Aucune réponse (hôte injoignable ou ping bloqué)
```

**TTL** :

```
Le TTL vous donne une indication du nombre de sauts :

TTL initial courant :
  Windows : 128
  Linux : 64
  Cisco : 255

Si vous recevez TTL=117, et que Google utilise 128 :
  128 - 117 = 11 sauts parcourus

Si vous recevez TTL=54, et que le serveur utilise 64 :
  64 - 54 = 10 sauts parcourus
```

### Variations de ping

**Ping avec nombre de paquets limité** :

```bash
# Envoyer 5 pings puis s'arrêter
ping -c 5 google.com      # Linux/Mac
ping -n 5 google.com      # Windows
```

**Ping avec intervalle personnalisé** :

```bash
# Un ping toutes les 0.2 secondes
ping -i 0.2 google.com    # Linux/Mac
```

**Ping avec taille de paquet personnalisée** :

```bash
# Paquet de 1000 octets
ping -s 1000 google.com   # Linux/Mac
ping -l 1000 google.com   # Windows
```

**Flood ping** (test de performance) :

```bash
# Envoyer le plus vite possible (nécessite root)
sudo ping -f google.com
```

**Ping IPv6** :

```bash
ping6 google.com           # Linux/Mac
ping -6 google.com         # Windows moderne
```

## Type 11 : Time Exceeded (TTL expiré)

### Principe

**Rôle** : Informer l'expéditeur que le TTL du paquet a atteint 0.

**Quand c'est envoyé** :

```
1. Un paquet arrive à un routeur avec TTL=1
2. Le routeur décrémente : TTL devient 0
3. Le routeur DÉTRUIT le paquet
4. Le routeur envoie un ICMP Type 11 à l'expéditeur :
   "Désolé, ton paquet est mort ici (TTL=0)"
```

**Structure** :

```
┌────────────────────────────────────────┐
│ Type : 11 (Time Exceeded)              │
├────────────────────────────────────────┤
│ Code : 0 (TTL expired in transit)      │
│        1 (Fragment reassembly timeout) │
├────────────────────────────────────────┤
│ Checksum                               │
├────────────────────────────────────────┤
│ Unused (4 octets à 0)                  │
├────────────────────────────────────────┤
│ En-tête IP du paquet original (20 oct) │
├────────────────────────────────────────┤
│ + 8 premiers octets des données        │
└────────────────────────────────────────┘
```

**Important** : Le message contient **l'en-tête IP original** pour que l'expéditeur sache quel paquet a échoué.

### C'est la BASE du traceroute !

## La commande traceroute

### Qu'est-ce que traceroute ?

**traceroute** (ou **tracert** sur Windows) permet de **cartographier le chemin** suivi par les paquets jusqu'à une destination.

**Analogie** : C'est comme suivre un colis avec un GPS qui enregistre tous les centres de tri par lesquels il passe.

### Fonctionnement du traceroute

Traceroute utilise **l'astuce du TTL** :

```
Principe génial :
  1. Envoyer un paquet avec TTL=1
     → Il meurt au 1er routeur
     → Le 1er routeur envoie Time Exceeded
     → On connaît maintenant le 1er routeur !

  2. Envoyer un paquet avec TTL=2
     → Il meurt au 2ème routeur
     → Le 2ème routeur envoie Time Exceeded
     → On connaît le 2ème routeur !

  3. TTL=3, puis TTL=4, etc.
     → On découvre tous les routeurs un par un !

  4. Quand on atteint la destination :
     → Plus de Time Exceeded
     → On reçoit Echo Reply (ou Port Unreachable)
     → Fin du traceroute !
```

### Schéma détaillé du traceroute

```
Vous                 Routeur 1       Routeur 2       Routeur 3       Destination
192.168.1.10         10.0.0.1        20.0.0.1        30.0.0.1        8.8.8.8

    │                    │               │               │               │
    │ Paquet TTL=1       │               │               │               │
    │───────────────────→│               │               │               │
    │                    │ TTL=0 !       │               │               │
    │                    │ Paquet détruit│               │               │
    │                    │               │               │               │
    │←───────────────────│               │               │               │
    │ ICMP Time Exceeded │               │               │               │
    │ (from 10.0.0.1)    │               │               │               │
    │                    │               │               │               │
    ├─ Hop 1: 10.0.0.1 (15ms)            │               │               │
    │                    │               │               │               │
    │                    │               │               │               │
    │ Paquet TTL=2       │               │               │               │
    │───────────────────→│               │               │               │
    │                    │ TTL=1         │               │               │
    │                    │──────────────→│               │               │
    │                    │               │ TTL=0 !       │               │
    │                    │               │ Paquet détruit│               │
    │                    │               │               │               │
    │←────────────────────────────────── │               │               │
    │ ICMP Time Exceeded (from 20.0.0.1) │               │               │
    │                    │               │               │               │
    ├─ Hop 2: 20.0.0.1 (28ms)            │               │               │
    │                    │               │               │               │
    │                    │               │               │               │
    │ Paquet TTL=3       │               │               │               │
    │───────────────────→│──────────────→│──────────────→│               │
    │                    │               │               │ TTL=0 !       │
    │                    │               │               │ Paquet détruit│
    │                    │               │               │               │
    │←────────────────────────────────────────────────── │               │
    │ ICMP Time Exceeded (from 30.0.0.1) │               │               │
    │                    │               │               │               │
    ├─ Hop 3: 30.0.0.1 (42ms)            │               │               │
    │                    │               │               │               │
    │                    │               │               │               │
    │ Paquet TTL=4       │               │               │               │
    │───────────────────→│──────────────→│──────────────→│──────────────→│
    │                    │               │               │               │
    │←────────────────────────────────────────────────────────────────── │
    │ ICMP Echo Reply (from 8.8.8.8)     │               │               │
    │                    │               │               │               │
    ├─ Hop 4: 8.8.8.8 (56ms) - DESTINATION ATTEINTE !    │               │
    │                    │               │               │               │
```

### Exemple de sortie traceroute

**Linux** :

```bash
$ traceroute google.com

traceroute to google.com (142.250.185.46), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  1.245 ms  1.198 ms  1.172 ms
 2  10.0.0.1 (10.0.0.1)  8.456 ms  8.423 ms  8.391 ms
 3  par-th2-9k-1.fr.eu (80.10.224.129)  12.234 ms  12.201 ms  12.167 ms
 4  be100-1078.par-g2-nc5.fr.eu (80.10.224.130)  14.123 ms  14.089 ms  14.056 ms
 5  72.14.234.128 (72.14.234.128)  15.234 ms  15.201 ms  15.167 ms
 6  * * *
 7  142.251.51.187 (142.251.51.187)  16.789 ms  16.756 ms  16.723 ms
 8  par21s20-in-f14.1e100.net (142.250.185.46)  17.234 ms  17.201 ms  17.167 ms
```

**Windows** :

```powershell
C:\> tracert google.com

Tracing route to google.com [142.250.185.46]
over a maximum of 30 hops:

  1     1 ms     1 ms     1 ms  192.168.1.1
  2     8 ms     9 ms     8 ms  10.0.0.1
  3    12 ms    13 ms    12 ms  par-th2-9k-1.fr.eu [80.10.224.129]
  4    14 ms    15 ms    14 ms  be100-1078.par-g2-nc5.fr.eu [80.10.224.130]
  5    16 ms    15 ms    16 ms  72.14.234.128
  6     *        *        *     Request timed out.
  7    17 ms    17 ms    18 ms  142.251.51.187
  8    18 ms    17 ms    18 ms  par21s20-in-f14.1e100.net [142.250.185.46]

Trace complete.
```

**Décryptage** :

```
Ligne 1 : 192.168.1.1 (1 ms)
  → Votre box/routeur (hop 1)
  → 3 mesures : 1.245, 1.198, 1.172 ms

Ligne 6 : * * *
  → Routeur qui ne répond pas aux ICMP
  → Soit configuré pour ne pas répondre
  → Soit filtré par un firewall
  → Le trafic passe quand même !

Ligne 8 : Destination finale
  → 8 sauts au total
  → ~17 ms de latence
```

### Différences traceroute Linux vs Windows

```
┌──────────────────┬─────────────────┬───────────────────┐
│                  │ Linux/Mac       │ Windows           │
├──────────────────┼─────────────────┼───────────────────┤
│ Commande         │ traceroute      │ tracert           │
│ Protocole utilisé│ UDP (port 33434+)│ ICMP Echo Request│
│ TTL initial      │ 1               │ 1                 │
│ Nombre de probes │ 3 par hop       │ 3 par hop         │
└──────────────────┴─────────────────┴───────────────────┘
```

**Pourquoi UDP sur Linux ?**

```
Historiquement, traceroute Linux envoie des paquets UDP
vers des ports non utilisés (33434, 33435, etc.)

Quand le paquet atteint la destination :
  → Port UDP fermé
  → Destination répond : ICMP Port Unreachable
  → Traceroute sait que c'est la fin !

Avantage : Différencie les hops intermédiaires de la destination
```

## Type 3 : Destination Unreachable

### Principe

**Rôle** : Informer l'expéditeur qu'un paquet **ne peut pas être livré**.

**Structure** :

```
┌────────────────────────────────────────┐
│ Type : 3 (Destination Unreachable)     │
├────────────────────────────────────────┤
│ Code : (voir ci-dessous)               │
├────────────────────────────────────────┤
│ Checksum                               │
├────────────────────────────────────────┤
│ Unused (4 octets)                      │
├────────────────────────────────────────┤
│ En-tête IP du paquet original          │
├────────────────────────────────────────┤
│ + 8 premiers octets des données        │
└────────────────────────────────────────┘
```

### Les codes de Destination Unreachable

```
┌──────┬──────────────────────────────┬────────────────────┐
│ Code │ Signification                │ Cause              │
├──────┼──────────────────────────────┼────────────────────┤
│  0   │ Net Unreachable              │ Réseau inaccessible│
│  1   │ Host Unreachable             │ Hôte inaccessible  │
│  2   │ Protocol Unreachable         │ Protocole inconnu  │
│  3   │ Port Unreachable             │ Port fermé         │
│  4   │ Fragmentation Needed but DF  │ Paquet trop gros   │
│  5   │ Source Route Failed          │ Route source échec │
│  6   │ Destination Network Unknown  │ Réseau inconnu     │
│  7   │ Destination Host Unknown     │ Hôte inconnu       │
│  9   │ Network Admin Prohibited     │ Bloqué par admin   │
│  10  │ Host Admin Prohibited        │ Hôte bloqué        │
│  13  │ Communication Admin Prohibited│ Comm. interdite   │
└──────┴──────────────────────────────┴────────────────────┘
```

### Exemples pratiques

**Code 1 : Host Unreachable**

```
$ ping 192.168.1.99

PING 192.168.1.99 (192.168.1.99) 56(84) bytes of data.
From 192.168.1.1 icmp_seq=1 Destination Host Unreachable
From 192.168.1.1 icmp_seq=2 Destination Host Unreachable

Cause : L'hôte 192.168.1.99 n'existe pas sur le réseau
        ou est éteint
```

**Code 3 : Port Unreachable**

```
Utilisé par traceroute (Linux) pour détecter la destination finale

Tous les hops intermédiaires : Time Exceeded
Destination finale : Port Unreachable
  → Traceroute sait que c'est la fin !
```

**Code 4 : Fragmentation Needed but DF set**

```
Scénario : Path MTU Discovery

1. Vous envoyez un gros paquet (1500 octets) avec DF=1 (Don't Fragment)
2. Un routeur rencontre un lien avec MTU=1400
3. Il ne peut PAS fragmenter (DF=1)
4. Il envoie ICMP Type 3 Code 4 :
   "Paquet trop gros ! MTU=1400 requis"
5. Vous réduisez la taille et renvoyez

Résultat : Découverte du MTU optimal du chemin
```

## Type 5 : Redirect

### Principe

**Rôle** : Informer un hôte qu'il existe une **meilleure route** vers une destination.

**Scénario typique** :

```
Réseau avec 2 routeurs :

┌────────────┐
│   PC       │
│192.168.1.10│
└────┬───────┘
     │
     │ Gateway par défaut : R1
     │
┌────┴──────────┬────────────┐
│               │            │
│  Routeur R1   │ Routeur R2 │
│ 192.168.1.1   │192.168.1.2 │
└───────────────┴────────────┘
        │            │
        └─ Réseau A  └─ Réseau B

Le PC envoie un paquet vers Réseau B via R1 (gateway par défaut)
R1 voit que R2 est sur le même segment ET plus proche de Réseau B
R1 envoie ICMP Redirect : "Utilise R2 pour Réseau B"
Le PC met à jour sa table de routage (temporairement)
```

**Structure** :

```
┌────────────────────────────────────────┐
│ Type : 5 (Redirect)                    │
├────────────────────────────────────────┤
│ Code : 0 = Redirect for Network        │
│        1 = Redirect for Host           │
│        2 = Redirect for TOS & Network  │
│        3 = Redirect for TOS & Host     │
├────────────────────────────────────────┤
│ Checksum                               │
├────────────────────────────────────────┤
│ Gateway IP Address (meilleur routeur)  │
├────────────────────────────────────────┤
│ En-tête IP du paquet original          │
└────────────────────────────────────────┘
```

**Note de sécurité** : ICMP Redirect peut être exploité pour des **attaques man-in-the-middle**. Beaucoup de systèmes modernes **ignorent les ICMP Redirect** par défaut.

## ICMPv6 : ICMP pour IPv6

### Différences avec ICMPv4

ICMPv6 est **plus important** qu'ICMPv4 car il remplace aussi **ARP** !

**Numéro de protocole** : 58 (au lieu de 1)

**Nouveaux rôles** :

```
ICMPv4 :
  - Messages d'erreur
  - Echo Request/Reply (ping)
  - Quelques messages de contrôle

ICMPv6 :
  - Tout ce qui est dans ICMPv4
  + Neighbor Discovery (remplace ARP)
  + Router Discovery
  + Path MTU Discovery amélioré
  + Multicast Listener Discovery
```

### Principaux types ICMPv6

```
┌──────┬────────────────────────────┬──────────────────────┐
│ Type │ Nom                        │ Usage                │
├──────┼────────────────────────────┼──────────────────────┤
│  1   │ Destination Unreachable    │ Idem ICMPv4          │
│  2   │ Packet Too Big             │ PMTU Discovery       │
│  3   │ Time Exceeded              │ Idem ICMPv4          │
│  4   │ Parameter Problem          │ En-tête invalide     │
│ 128  │ Echo Request               │ Ping IPv6            │
│ 129  │ Echo Reply                 │ Réponse ping IPv6    │
│ 133  │ Router Solicitation        │ Chercher routeur     │
│ 134  │ Router Advertisement       │ Annonce routeur      │
│ 135  │ Neighbor Solicitation      │ Remplace ARP Request │
│ 136  │ Neighbor Advertisement     │ Remplace ARP Reply   │
│ 137  │ Redirect                   │ Redirection          │
└──────┴────────────────────────────┴──────────────────────┘
```

**Note** : Les numéros sont différents ! Echo Request = 128 (pas 8).

### Neighbor Discovery (ND)

**Remplace ARP** en IPv6 :

```
IPv4 : "Qui a l'IP 192.168.1.1 ? Quelle est ton MAC ?"
       → ARP Request (broadcast)
       → ARP Reply

IPv6 : "Qui a l'IP 2001:db8::1 ? Quelle est ton MAC ?"
       → ICMPv6 Neighbor Solicitation (multicast)
       → ICMPv6 Neighbor Advertisement
```

**Avantage** : Utilise multicast au lieu de broadcast (plus efficace).

### Ping IPv6

```bash
# Linux/Mac
ping6 google.com
ping6 2001:4860:4860::8888

# Windows moderne
ping -6 google.com

# Exemple de sortie
PING google.com(par21s11-in-x0e.1e100.net (2a00:1450:4007:80f::200e)) 56 data bytes
64 bytes from par21s11-in-x0e.1e100.net (2a00:1450:4007:80f::200e): icmp_seq=1 ttl=118 time=15.3 ms
```

## Sécurité et ICMP

### Pourquoi bloquer ICMP ?

**Arguments pour bloquer** :

```
❌ ICMP peut révéler des infos sur le réseau
   (topologie, systèmes actifs, OS...)

❌ Certaines attaques utilisent ICMP :
   - Ping flood (DoS)
   - Smurf attack (amplification)
   - ICMP tunneling (exfiltration de données)

❌ Faux sentiment de sécurité par obscurité
```

**Arguments contre le blocage** :

```
✅ ICMP est ESSENTIEL au fonctionnement d'IP
   - Path MTU Discovery casse si bloqué
   - Diagnostic réseau impossible
   - Détection de problèmes ralentie

✅ Bloquer ICMP ne stoppe pas les vrais attaquants
   (ils utiliseront d'autres méthodes)

✅ Cela complique la vie des admins légitimes
```

### Bonne pratique : filtrage sélectif

**Ne PAS bloquer complètement ICMP** :

```
✅ Autoriser les types essentiels :
   - Type 3 (Destination Unreachable)
   - Type 4 (Fragmentation Needed) ← CRUCIAL pour PMTU
   - Type 11 (Time Exceeded)

⚠️ Limiter les types de diagnostic :
   - Type 8/0 (Echo Request/Reply) : rate-limit
   - Bloquer uniquement depuis l'extérieur, autoriser en interne

❌ Bloquer les types dangereux :
   - Type 5 (Redirect) : risque de détournement
   - Type 13/14 (Timestamp) : information leakage
```

**Exemple de règle iptables** :

```bash
# Autoriser ICMP essentiel (entrant)
iptables -A INPUT -p icmp --icmp-type destination-unreachable -j ACCEPT
iptables -A INPUT -p icmp --icmp-type time-exceeded -j ACCEPT
iptables -A INPUT -p icmp --icmp-type parameter-problem -j ACCEPT

# Limiter Echo Request (ping) à 5 par seconde
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/sec -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# Bloquer Redirect (sécurité)
iptables -A INPUT -p icmp --icmp-type redirect -j DROP
```

### Attaques ICMP classiques

**1. Ping Flood**

```
Envoyer un très grand nombre de ping
→ Saturer la bande passante ou le CPU
→ Déni de service (DoS)

Protection : Rate limiting
```

**2. Ping of Death**

```
Envoyer un paquet ICMP > 65535 octets (max IP)
→ Via fragmentation malicieuse
→ Causait des crashs (anciennes vulnérabilités)

Protection : Corrigé dans tous les OS modernes
```

**3. Smurf Attack**

```
1. Attaquant envoie ping avec IP source SPOOFÉE (victime)
2. Destination = adresse broadcast du réseau
3. Tous les hôtes répondent... à la victime !
4. Amplification : 1 paquet → 100+ réponses

Protection :
  - Désactiver IP directed-broadcast sur les routeurs
  - Filtrage anti-spoofing (BCP 38)
```

**4. ICMP Tunneling**

```
Cacher des données dans les messages ICMP
→ Exfiltrer des données d'un réseau filtré
→ Contourner les firewalls qui autorisent ICMP

Protection : Deep Packet Inspection (DPI)
```

## Cas d'usage pratiques

### Diagnostic 1 : "Le site ne répond pas"

```
Étape 1 : Ping le site
$ ping example.com

Résultat possible :
  A. Réponses reçues → Connectivité OK
  B. Request timeout → Site bloque ping OU problème réseau
  C. Destination Unreachable → Problème de routage
```

**Si B (timeout)** :

```
Étape 2 : Tester avec un autre protocole
$ curl https://example.com

Si ça marche : Le site bloque simplement ICMP (normal)
Si ça ne marche pas : Vrai problème de connectivité
```

**Si C (Unreachable)** :

```
Étape 3 : Traceroute pour localiser le problème
$ traceroute example.com

Trouver où ça bloque :
  - Première partie OK, puis * * * → Problème côté ISP ou destination
  - Tout * * * dès le début → Problème local
```

### Diagnostic 2 : "Internet est lent"

```
$ ping 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=245.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=198.3 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=312.1 ms

Analyse :
  - Latence élevée (>100ms pour du national)
  - Variation importante (jitter)
  → Connexion instable ou saturée

Actions :
  1. Tester à plusieurs moments
  2. Vérifier la saturation de la bande passante
  3. Contacter le FAI si persistant
```

### Diagnostic 3 : "Certains sites ne chargent pas"

```
Symptôme : Sites lents ou images qui ne chargent pas

Cause possible : Problème de PMTU Discovery

Test :
$ ping -s 1472 -M do google.com   # Linux
$ ping -f -l 1472 google.com       # Windows

Si timeout ou fragmentation errors :
  → Problème de MTU sur le chemin
  → Type 3 Code 4 (Fragmentation Needed) probablement bloqué

Solution :
  - Réduire le MTU de l'interface
  - Vérifier les firewalls (autoriser ICMP Type 3)
```

### Diagnostic 4 : Mesurer la latence stable

```
$ ping -c 100 google.com | tail -2

--- google.com ping statistics ---
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 14.123/15.234/18.456/0.892 ms

Interprétation :
  - min : Meilleur cas (14.1 ms)
  - avg : Latence moyenne (15.2 ms)
  - max : Pire cas (18.5 ms)
  - mdev : Écart-type (0.9 ms) → Très stable !

Une mdev faible = connexion stable
Une mdev élevée = connexion instable (jitter)
```

## Outils avancés utilisant ICMP

### MTR (My TraceRoute)

**Combine ping + traceroute en temps réel** :

```bash
$ mtr google.com

                     Packets               Pings
 Host               Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. 192.168.1.1      0.0%    10    1.2   1.3   1.1   1.6   0.1
 2. 10.0.0.1         0.0%    10    8.5   8.7   8.3   9.2   0.3
 3. par-th2.fr      0.0%    10   12.3  12.5  12.1  13.1   0.4
 4. google.com       0.0%    10   15.2  15.4  15.0  16.1   0.3

Avantage : Vue en temps réel de tous les hops
```

### Hping

**Utilitaire avancé de génération de paquets** :

```bash
# Ping avec TTL spécifique
hping3 -1 -t 5 google.com

# Traceroute avec TCP (contourne certains firewalls)
hping3 --traceroute -S -p 80 google.com
```

### Fping

**Ping multiple en parallèle** :

```bash
# Pinger tout un sous-réseau
fping -a -g 192.168.1.0/24

# Sortie : liste des IPs qui répondent
192.168.1.1
192.168.1.10
192.168.1.20
```

## Résumé visuel

```
┌──────────────────────────────────────────────────────────┐
│                 ICMP - CONCEPTS CLÉS                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  🎯 Rôle : Messages de contrôle et d'erreur pour IP      │
│                                                          │
│  📊 Principaux types :                                   │
│     • Type 0/8 : Echo Reply/Request (ping)               │
│     • Type 3 : Destination Unreachable                   │
│     • Type 5 : Redirect (redirection de route)           │
│     • Type 11 : Time Exceeded (TTL=0)                    │
│                                                          │
│  🔧 Outils basés sur ICMP :                              │
│     • ping : Test de connectivité (Type 8/0)             │
│     • traceroute : Cartographie du chemin (Type 11)      │
│     • mtr : Combinaison ping + traceroute                │
│                                                          │
│  🔒 Sécurité :                                           │
│     • Ne PAS bloquer complètement ICMP                   │
│     • Autoriser les types essentiels (3, 4, 11)          │
│     • Rate-limit Echo Request (anti-flood)               │
│                                                          │
│  🆚 ICMPv6 : Plus important, remplace aussi ARP          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Points clés à retenir

✅ **ICMP** = système de **messages de contrôle et d'erreur** d'IP

✅ ICMP est **protocole 1** dans l'en-tête IP (58 pour ICMPv6)

✅ **Ping** utilise Type 8 (Echo Request) et Type 0 (Echo Reply)

✅ **Traceroute** utilise l'astuce du TTL + Type 11 (Time Exceeded)

✅ Type 3 = **Destination Unreachable** (nombreux codes : host, port, net...)

✅ **Ne JAMAIS bloquer complètement ICMP** (casse PMTU Discovery)

✅ **Rate-limiting** est préférable au blocage total

✅ ICMPv6 est **plus important** qu'ICMPv4 (remplace ARP)

✅ `ping` mesure le **RTT** (Round Trip Time) et la **perte de paquets**

✅ `traceroute` révèle le **chemin complet** jusqu'à la destination

## Pour aller plus loin

Maintenant que vous maîtrisez ICMP, vous êtes prêt pour :

- **Routage** : comment les paquets trouvent leur chemin
- **Tables de routage** : structure et fonctionnement
- **Protocoles de routage** : RIP, OSPF, BGP
- **Path MTU Discovery** : optimisation de la taille des paquets

---

**💡 Expérience pratique** :
```bash
# Tester la connectivité
ping google.com

# Voir le chemin
traceroute google.com

# Test avancé (si installé)
mtr google.com

Observez les différences de latence entre les hops !
```

---

*Dans la section suivante, nous allons explorer le routage : comment les routeurs décident du chemin optimal pour vos paquets...*

⏭️ [Routage : concepts fondamentaux](/03-couche-internet/10-routage-concepts.md)
