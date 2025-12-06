🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.4 Lecture et interprétation des captures

## Introduction

Savoir capturer des paquets avec Wireshark est une chose, mais **interpréter** ce qu'on voit en est une autre. Une capture réseau contient une mine d'informations, mais encore faut-il savoir les lire et comprendre ce qu'elles signifient.

Dans cette section, nous allons apprendre à lire méthodiquement une capture, couche par couche, à suivre les conversations, à calculer les délais, et surtout à **reconnaître les patterns** qui indiquent un problème ou un comportement normal.

Nous analyserons des scénarios réels : établissement de connexion TCP, requête HTTP, résolution DNS, handshake TLS, et bien d'autres. À la fin de cette section, vous serez capable de regarder une capture et de comprendre instantanément ce qui se passe.

---

## Anatomie d'un paquet : lecture couche par couche

### Principe de lecture

Wireshark affiche les paquets en suivant le modèle d'encapsulation TCP/IP, de la couche la plus basse (physique/liaison) à la plus haute (application) :

```
┌─────────────────────────────────────┐
│  Couche 7 - Application (HTTP, DNS) │  ← Ce qu'on cherche souvent
├─────────────────────────────────────┤
│  Couche 4 - Transport (TCP, UDP)    │  ← Fiabilité, ports
├─────────────────────────────────────┤
│  Couche 3 - Réseau (IP)             │  ← Routage, adressage
├─────────────────────────────────────┤
│  Couche 2 - Liaison (Ethernet)      │  ← Transmission locale
├─────────────────────────────────────┤
│  Couche 1 - Physique (Frame info)   │  ← Métadonnées capture
└─────────────────────────────────────┘
```

**Règle d'or :** On lit de **bas en haut** pour comprendre l'encapsulation, mais on analyse de **haut en bas** pour comprendre l'application.

### Exemple complet : Décortiquer un paquet HTTP GET

Prenons un paquet réel capturé lors d'une requête web :

```
Frame 142: 458 bytes on wire (3664 bits), 458 bytes captured (3664 bits)
Ethernet II, Src: Apple_12:34:56 (a4:83:e7:12:34:56), Dst: Netgear_ab:cd:ef (00:1f:33:ab:cd:ef)
Internet Protocol Version 4, Src: 192.168.1.10, Dst: 93.184.216.34
Transmission Control Protocol, Src Port: 52341, Dst Port: 80, Seq: 1, Ack: 1, Len: 404
Hypertext Transfer Protocol
    GET /index.html HTTP/1.1\r\n
```

Analysons chaque couche en détail.

---

## Couche 1 : Frame (Métadonnées de capture)

```
▼ Frame 142: 458 bytes on wire (3664 bits), 458 bytes captured (3664 bits)
    Interface id: 0 (en0)
    Encapsulation type: Ethernet (1)
    Arrival Time: Dec  7, 2025 14:32:15.234567890 CET
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1733581935.234567890 seconds
    [Time delta from previous captured frame: 0.001234567 seconds]
    [Time delta from previous displayed frame: 0.001234567 seconds]
    [Time since reference or first frame: 5.234567890 seconds]
    Frame Number: 142
    Frame Length: 458 bytes (3664 bits)
    Capture Length: 458 bytes (3664 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:tcp:http]
```

### Champs importants

**Interface id: 0 (en0)**
- Interface réseau qui a capturé ce paquet
- `en0` = première interface Ethernet sur macOS
- Utile si capture sur plusieurs interfaces simultanément

**Arrival Time**
- Horodatage précis de la capture (jusqu'à la nanoseconde)
- Format : Date + Heure + Timezone
- Permet de corréler avec des logs système

**Time delta from previous captured frame: 0.001234567 seconds**
- Temps écoulé depuis le paquet précédent **capturé**
- Ici : 1.23 ms depuis le paquet #141
- **Crucial** pour détecter des délais anormaux

**Time since reference or first frame: 5.234567890 seconds**
- Temps depuis le début de la capture
- Permet de situer le paquet dans la timeline globale
- Ici : 5.23 secondes après le premier paquet

**Frame Length vs Capture Length**
```
Frame Length: 458 bytes     ← Taille réelle sur le fil
Capture Length: 458 bytes   ← Taille capturée

Si différent :
Frame Length: 1518 bytes
Capture Length: 96 bytes    ← Snaplen limité, paquet tronqué !
```

**Protocols in frame: eth:ethertype:ip:tcp:http**
- Liste des protocoles détectés dans ce paquet
- Utile pour comprendre rapidement la nature du trafic
- Ici : Ethernet → IP → TCP → HTTP

---

## Couche 2 : Ethernet II

```
▼ Ethernet II, Src: Apple_12:34:56 (a4:83:e7:12:34:56), Dst: Netgear_ab:cd:ef (00:1f:33:ab:cd:ef)
    Destination: Netgear_ab:cd:ef (00:1f:33:ab:cd:ef)
        Address: Netgear_ab:cd:ef (00:1f:33:ab:cd:ef)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Source: Apple_12:34:56 (a4:83:e7:12:34:56)
        Address: Apple_12:34:56 (a4:83:e7:12:34:56)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Type: IPv4 (0x0800)
```

### Lecture des adresses MAC

**Format :** 6 octets en hexadécimal (48 bits)
```
a4:83:e7:12:34:56
└──┬──┘ └───┬────┘
   │        └─ Identifiant unique (attribué par fabricant)
   └─ OUI (Organizationally Unique Identifier)
```

**Résolution du fabricant :**
- Wireshark résout automatiquement l'OUI
- `a4:83:e7` → Apple
- `00:1f:33` → Netgear

**Destination: Netgear_ab:cd:ef**
- Adresse MAC de la passerelle (box/routeur)
- Même si on envoie vers Internet, au niveau Ethernet c'est vers la passerelle

**Source: Apple_12:34:56**
- Adresse MAC de notre carte réseau (MacBook, iPhone, etc.)

**Type: IPv4 (0x0800)**
- EtherType : indique le protocole encapsulé
- `0x0800` = IPv4
- `0x86DD` = IPv6
- `0x0806` = ARP

### Ce qu'on peut déduire

```
Source MAC = Apple → Émetteur utilise un appareil Apple
Destination MAC = Netgear → Passerelle est une box Netgear
Type = IPv4 → Communication en IPv4 (pas IPv6)
```

---

## Couche 3 : Internet Protocol Version 4 (IP)

```
▼ Internet Protocol Version 4, Src: 192.168.1.10, Dst: 93.184.216.34
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
        0000 00.. = Differentiated Services Codepoint: Default (0)
        .... ..00 = Explicit Congestion Notification: Not ECN-Capable Transport (0)
    Total Length: 444
    Identification: 0x3a4b (14923)
    Flags: 0x4000, Don't fragment
        0... .... .... .... = Reserved bit: Not set
        .1.. .... .... .... = Don't fragment: Set
        ..0. .... .... .... = More fragments: Not set
    Fragment Offset: 0
    Time to Live: 64
    Protocol: TCP (6)
    Header Checksum: 0x7c8d [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 192.168.1.10
    Destination Address: 93.184.216.34
```

### Champs importants

**Version: 4**
- Protocole IP version 4 (vs IPv6)
- 4 bits

**Header Length: 20 bytes (5)**
- Taille de l'en-tête IP : 5 × 4 = 20 octets
- Sans options IP (minimum possible)
- Si > 20 : options IP présentes

**Total Length: 444**
- Taille totale du paquet IP (en-tête + données)
- 444 octets = 20 (IP header) + 20 (TCP header) + 404 (données HTTP)

**Identification: 0x3a4b (14923)**
- Identifiant unique pour la fragmentation
- Tous les fragments d'un même paquet ont le même ID
- Ici : pas de fragmentation, mais ID quand même présent

**Flags: Don't fragment**
```
0... = Reserved (toujours 0)
.1.. = Don't fragment (DF) ✅
..0. = More fragments (MF)
```

**Don't fragment (DF) = 1**
- Interdit la fragmentation de ce paquet
- Si paquet trop grand pour le MTU → paquet droppé
- Utile pour découvrir le MTU du chemin (Path MTU Discovery)

**Time to Live: 64**
- Nombre de sauts (routeurs) restants
- Décrémenté à chaque routeur
- TTL = 0 → paquet détruit (évite les boucles infinies)
- Valeur initiale typique :
  - 64 → Linux/macOS/Unix
  - 128 → Windows
  - 255 → Équipements réseau

**Protocol: TCP (6)**
- Protocole encapsulé dans IP
- 6 = TCP
- 17 = UDP
- 1 = ICMP

**Source/Destination Address**
- **192.168.1.10** : Adresse IP privée (RFC 1918) → client local
- **93.184.216.34** : Adresse IP publique → serveur distant (example.com)

### Ce qu'on peut déduire

```
Client local (192.168.1.10) → Serveur distant (93.184.216.34)
TTL = 64 → Paquet fraîchement émis (système Unix/Linux/macOS)
DF = 1 → Path MTU Discovery activé
Total Length = 444 → Petit paquet, pas de problème de MTU
```

---

## Couche 4 : Transmission Control Protocol (TCP)

```
▼ Transmission Control Protocol, Src Port: 52341, Dst Port: 80, Seq: 1, Ack: 1, Len: 404
    Source Port: 52341
    Destination Port: 80
    [Stream index: 12]
    [Conversation completeness: Complete, WITH_DATA (31)]
    [TCP Segment Len: 404]
    Sequence Number: 1    (relative sequence number)
    Sequence Number (raw): 2847562341
    [Next Sequence Number: 405    (relative sequence number)]
    Acknowledgment Number: 1    (relative ack number)
    Acknowledgment number (raw): 1923847561
    1000 .... = Header Length: 32 bytes (8)
    Flags: 0x018 (PSH, ACK)
        000. .... .... = Reserved: Not set
        ...0 .... .... = Nonce: Not set
        .... 0... .... = Congestion Window Reduced (CWR): Not set
        .... .0.. .... = ECN-Echo: Not set
        .... ..0. .... = Urgent: Not set
        .... ...1 .... = Acknowledgment: Set
        .... .... 1... = Push: Set
        .... .... .0.. = Reset: Not set
        .... .... ..0. = Syn: Not set
        .... .... ...0 = Fin: Not set
        [TCP Flags: ·······AP···]
    Window: 65535
    [Calculated window size: 65535]
    Checksum: 0x4f2a [unverified]
    [Checksum Status: Unverified]
    Urgent Pointer: 0
    Options: (12 bytes), No-Operation (NOP), No-Operation (NOP), Timestamps
        TCP Option - No-Operation (NOP)
        TCP Option - No-Operation (NOP)
        TCP Option - Timestamps: TSval 123456789, TSecr 987654321
    [Timestamps]
    [SEQ/ACK analysis]
        [This is an ACK to the segment in frame: 141]
        [The RTT to ACK the segment was: 0.001234567 seconds]
        [iRTT: 0.045678901 seconds]
    TCP payload (404 bytes)
```

### Ports source et destination

**Source Port: 52341**
- Port éphémère (dynamique) du client
- Range typique : 49152-65535 (IANA)
- Choisi aléatoirement par l'OS

**Destination Port: 80**
- Port well-known du serveur
- 80 = HTTP
- 443 = HTTPS
- 22 = SSH

**Identification de la communication :**
```
Socket client : 192.168.1.10:52341
Socket serveur : 93.184.216.34:80
```

### Stream index: 12

- Wireshark assigne un numéro unique à chaque conversation TCP
- Tous les paquets de cette connexion ont le même stream index
- **Très utile** pour filtrer : `tcp.stream eq 12`

### Numéros de séquence

**Sequence Number: 1 (relative)**
- Numéro de séquence **relatif** (Wireshark normalise à partir de 0)
- Indique la position de ce segment dans le flux de données
- Séquence = 1 signifie que c'est le premier octet de données (après handshake)

**Sequence Number (raw): 2847562341**
- Numéro de séquence **absolu** (réel)
- Choisi aléatoirement au début de la connexion (sécurité)
- Wireshark affiche le relatif par défaut (plus lisible)

**Next Sequence Number: 405 (relative)**
- Prochain numéro de séquence attendu
- 1 + 404 = 405 (404 octets de données envoyés)
- Le prochain paquet de données devrait avoir Seq = 405

**Acknowledgment Number: 1 (relative)**
- Accuse réception des données reçues
- "J'ai bien reçu jusqu'à l'octet 1"
- Le prochain octet que j'attends est le 1

### Flags TCP

```
Flags: 0x018 (PSH, ACK)
.... ...1 .... = Acknowledgment: Set ✅
.... .... 1... = Push: Set ✅
```

**ACK (Acknowledgment) : Set**
- Accuse réception des données
- Présent dans presque tous les paquets après le handshake

**PSH (Push) : Set**
- Indique que les données doivent être remontées immédiatement à l'application
- Typique pour les requêtes HTTP (pas de buffering)

**Autres flags importants :**
```
SYN = 1 → Demande de connexion
FIN = 1 → Demande de fermeture
RST = 1 → Réinitialisation brutale (erreur)
URG = 1 → Données urgentes (rarement utilisé)
```

### Window: 65535

**Fenêtre de réception** (Receive Window)
- Nombre d'octets que l'émetteur peut envoyer sans attendre d'ACK
- 65535 = Fenêtre maximale sans TCP Window Scaling
- Indique "j'ai 64 KB de buffer disponible"

**Impact sur les performances :**
```
Window trop petit → Débit limité (attentes fréquentes)
Window = 0 → Récepteur saturé, émetteur doit attendre
Window grand → Meilleur débit (plus de données en vol)
```

### TCP Options: Timestamps

```
TCP Option - Timestamps: TSval 123456789, TSecr 987654321
```

**TSval (Timestamp Value)**
- Horodatage de l'émetteur
- Permet de calculer le RTT (Round Trip Time)

**TSecr (Timestamp Echo Reply)**
- Écho du timestamp reçu précédemment
- Permet au destinataire de mesurer le RTT

**Wireshark calcule automatiquement :**
```
[The RTT to ACK the segment was: 0.001234567 seconds]
```
- Temps entre l'envoi et l'accusé de réception
- Ici : 1.23 ms (excellent)

### SEQ/ACK Analysis (calculé par Wireshark)

```
[This is an ACK to the segment in frame: 141]
```
- Wireshark a identifié que ce paquet accuse réception du paquet #141
- **Très utile** pour suivre les échanges

```
[iRTT: 0.045678901 seconds]
```
- **Initial Round Trip Time**
- RTT mesuré pendant le handshake (SYN → SYN-ACK)
- Ici : 45.7 ms

### TCP Payload: 404 bytes

- Données applicatives (HTTP dans ce cas)
- 404 octets de requête GET

---

## Couche 7 : Hypertext Transfer Protocol (HTTP)

```
▼ Hypertext Transfer Protocol
    GET /index.html HTTP/1.1\r\n
    Host: example.com\r\n
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36\r\n
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,*/*;q=0.8\r\n
    Accept-Language: en-US,en;q=0.5\r\n
    Accept-Encoding: gzip, deflate, br\r\n
    Connection: keep-alive\r\n
    Upgrade-Insecure-Requests: 1\r\n
    \r\n
    [Full request URI: http://example.com/index.html]
    [HTTP request 1/1]
    [Response in frame: 145]
```

### Structure de la requête HTTP

**Ligne de requête :**
```
GET /index.html HTTP/1.1
└┬┘ └────┬─────┘ └───┬───┘
 │       │           └─ Version HTTP
 │       └─ URI demandée
 └─ Méthode HTTP
```

**Headers HTTP :**
```
Host: example.com
```
- **Obligatoire** en HTTP/1.1
- Indique le domaine ciblé (virtualhosting)
- Un serveur peut héberger plusieurs sites

```
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...
```
- Identifie le client (navigateur, OS)
- Permet au serveur d'adapter la réponse

```
Accept-Encoding: gzip, deflate, br
```
- Compressions acceptées par le client
- `gzip` = compression standard
- `br` = Brotli (plus efficace)

```
Connection: keep-alive
```
- Demande de réutiliser la connexion TCP
- HTTP/1.1 : keep-alive par défaut
- HTTP/1.0 : close par défaut

### Informations calculées par Wireshark

```
[Full request URI: http://example.com/index.html]
```
- Wireshark reconstruit l'URL complète
- Combinaison de Host + URI

```
[Response in frame: 145]
```
- **Très utile** : Wireshark a trouvé la réponse correspondante
- Cliquer pour sauter directement au paquet #145

---

## Analyser une conversation complète

### Three-Way Handshake TCP

Analysons l'établissement d'une connexion TCP :

**Paquet #139 : SYN →**
```
TCP, Src Port: 52341, Dst Port: 80, Seq: 0, Len: 0
    Sequence Number: 0 (relative)
    Flags: 0x002 (SYN)
        .... .... ..1. = Syn: Set
    Window: 65535
    Options: (20 bytes)
        Maximum Segment Size: 1460
        Window Scale: 8 (multiply by 256)
        SACK permitted
        Timestamps: TSval 123456789, TSecr 0
```

**Lecture :**
- Client → Serveur
- SYN = 1 : "Je veux établir une connexion"
- Seq = 0 : Premier numéro de séquence
- Len = 0 : Pas de données (juste le contrôle)
- MSS = 1460 : "Je peux recevoir des segments de 1460 bytes"
- Window Scale = 8 : "Ma vraie fenêtre = Window × 256"

**Paquet #140 : SYN-ACK ←**
```
TCP, Src Port: 80, Dst Port: 52341, Seq: 0, Ack: 1, Len: 0
    Sequence Number: 0 (relative)
    Acknowledgment Number: 1 (relative)
    Flags: 0x012 (SYN, ACK)
        .... ...1 .... = Acknowledgment: Set
        .... .... ..1. = Syn: Set
    Window: 29200
    Options: (20 bytes)
        Maximum Segment Size: 1460
        SACK permitted
        Timestamps: TSval 987654321, TSecr 123456789
        Window Scale: 7 (multiply by 128)
```

**Lecture :**
- Serveur → Client
- SYN = 1, ACK = 1 : "D'accord, j'accepte la connexion"
- Seq = 0 : Mon premier numéro de séquence
- Ack = 1 : "J'ai bien reçu ton SYN (Seq 0), j'attends Seq 1"
- MSS = 1460 : "Moi aussi"
- Window = 29200, Scale = 7 : Vraie fenêtre = 29200 × 128 = 3.7 MB

**Paquet #141 : ACK →**
```
TCP, Src Port: 52341, Dst Port: 80, Seq: 1, Ack: 1, Len: 0
    Sequence Number: 1 (relative)
    Acknowledgment Number: 1 (relative)
    Flags: 0x010 (ACK)
        .... ...1 .... = Acknowledgment: Set
    Window: 65535
```

**Lecture :**
- Client → Serveur
- ACK = 1 : "J'ai bien reçu ton SYN-ACK"
- Seq = 1 : (après mon SYN qui était Seq 0)
- Ack = 1 : (j'ai reçu ton SYN qui était Seq 0)
- **Connexion établie** ✅

**Timeline du handshake :**
```
Client                          Serveur
  |                                |
  |--- SYN (Seq=0) --------------> |  Frame 139, t=0.000000
  |                                |
  |<-- SYN-ACK (Seq=0, Ack=1) ---- |  Frame 140, t=0.045678
  |                                |
  |--- ACK (Seq=1, Ack=1) -------> |  Frame 141, t=0.046912
  |                                |
  [Connexion établie]

RTT = t(SYN-ACK) - t(SYN) = 45.7 ms
```

### Échange de données HTTP

**Paquet #142 : Requête HTTP GET →**
```
TCP, Seq: 1, Ack: 1, Len: 404, [PSH, ACK]
HTTP: GET /index.html HTTP/1.1
```

**Lecture :**
- Seq = 1 : Premier octet de données (juste après handshake)
- Len = 404 : 404 octets de requête HTTP
- PSH : "Transmets immédiatement à l'application"
- Next Seq = 1 + 404 = 405

**Paquet #143 : ACK de la requête ←**
```
TCP, Seq: 1, Ack: 405, Len: 0, [ACK]
```

**Lecture :**
- Serveur → Client
- Ack = 405 : "J'ai bien reçu les octets 1 à 404"
- Len = 0 : Pas de données, juste un ACK
- Serveur prépare la réponse...

**Paquet #145 : Réponse HTTP 200 OK ←**
```
TCP, Seq: 1, Ack: 405, Len: 1256, [PSH, ACK]
HTTP: HTTP/1.1 200 OK
```

**Lecture :**
- Seq = 1 : Premier octet de réponse du serveur
- Ack = 405 : Toujours en attente d'octets >= 405 du client
- Len = 1256 : 1256 octets de réponse (headers + début HTML)
- Next Seq = 1 + 1256 = 1257

**Paquet #146 : ACK de la réponse →**
```
TCP, Seq: 405, Ack: 1257, Len: 0, [ACK]
```

**Lecture :**
- Client → Serveur
- Ack = 1257 : "J'ai bien reçu les octets 1 à 1256"
- Seq = 405 : (je n'ai toujours pas envoyé de nouvelles données)

**Timeline de l'échange HTTP :**
```
Client                                    Serveur
  |                                          |
  |--- GET /index.html (Seq=1, Len=404) ---> |  #142, t=0.050000
  |                                          |
  |<-- ACK (Ack=405) ----------------------- |  #143, t=0.095678
  |                                          |  [Serveur traite requête]
  |                                          |
  |<-- HTTP 200 OK (Seq=1, Len=1256) ------- |  #145, t=0.175234
  |                                          |
  |--- ACK (Ack=1257) ---------------------->|  #146, t=0.176468
  |                                          |

Temps de traitement serveur = t(#145) - t(#143) = 79.6 ms
RTT = t(#143) - t(#142) = 45.7 ms
```

### Fermeture de connexion (Four-Way Handshake)

**Paquet #200 : FIN →**
```
TCP, Seq: 405, Ack: 3456, Len: 0, [FIN, ACK]
```
- Client → Serveur
- FIN = 1 : "Je n'ai plus de données à envoyer"
- Je peux encore recevoir

**Paquet #201 : ACK ←**
```
TCP, Seq: 3456, Ack: 406, Len: 0, [ACK]
```
- Serveur → Client
- "J'ai compris que tu fermes"
- Ack = 406 : FIN compte comme 1 octet

**Paquet #202 : FIN ←**
```
TCP, Seq: 3456, Ack: 406, Len: 0, [FIN, ACK]
```
- Serveur → Client
- "Moi aussi je ferme"

**Paquet #203 : ACK →**
```
TCP, Seq: 406, Ack: 3457, Len: 0, [ACK]
```
- Client → Serveur
- "Bien reçu, connexion fermée"
- **Connexion terminée** ✅

---

## Analyser une requête DNS

Analysons une résolution DNS complète :

**Paquet #50 : DNS Query →**
```
Domain Name System (query)
    Transaction ID: 0x1a2b
    Flags: 0x0100 Standard query
        0... .... .... .... = Response: Message is a query
        .000 0... .... .... = Opcode: Standard query (0)
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... .0.. .... = Z: reserved (0)
        .... .... ...0 .... = Non-authenticated data: Unacceptable
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 0
    Queries
        example.com: type A, class IN
            Name: example.com
            [Name Length: 11]
            [Label Count: 2]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
```

**Lecture :**
- Transaction ID: 0x1a2b → Identifiant unique pour associer requête/réponse
- Recursion desired = 1 → "Fais la résolution complète pour moi"
- Questions: 1 → On pose une question
- Query: "Quelle est l'adresse A (IPv4) de example.com ?"

**Paquet #51 : DNS Response ←**
```
Domain Name System (response)
    Transaction ID: 0x1a2b
    Flags: 0x8180 Standard query response, No error
        1... .... .... .... = Response: Message is a response
        .000 0... .... .... = Opcode: Standard query (0)
        .... .0.. .... .... = Authoritative: Server is not an authority
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... 1... .... = Recursion available: Server can do recursive queries
        .... .... .0.. .... = Z: reserved (0)
        .... .... ..0. .... = Answer authenticated: Answer/authority not authenticated
        .... .... ...0 .... = Non-authenticated data: Unacceptable
        .... .... .... 0000 = Reply code: No error (0)
    Questions: 1
    Answer RRs: 1
    Authority RRs: 0
    Additional RRs: 0
    Queries
        example.com: type A, class IN
    Answers
        example.com: type A, class IN, addr 93.184.216.34
            Name: example.com
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 86400 (1 day)
            Data length: 4
            Address: 93.184.216.34
```

**Lecture :**
- Transaction ID: 0x1a2b → Même ID que la requête
- Response = 1 → C'est une réponse
- Reply code: No error (0) → Résolution réussie
- Answer RRs: 1 → Une réponse
- **Réponse : example.com = 93.184.216.34**
- TTL: 86400 seconds (1 jour) → Peut être mise en cache 24h

**Wireshark ajoute :**
```
[Request In: 50]
[Time: 0.015234 seconds]
```
- Requête dans le paquet #50
- Temps de réponse DNS : 15.2 ms

---

## Analyser un handshake TLS

Connexion HTTPS (TLS 1.3) :

**Paquet #300 : Client Hello →**
```
Transport Layer Security
    TLSv1.3 Record Layer: Handshake Protocol: Client Hello
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Length: 512
        Handshake Protocol: Client Hello
            Handshake Type: Client Hello (1)
            Length: 508
            Version: TLS 1.2 (0x0303)
            Random: 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b
            Session ID Length: 32
            Session ID: ...
            Cipher Suites Length: 32
            Cipher Suites (16 suites)
                Cipher Suite: TLS_AES_128_GCM_SHA256 (0x1301)
                Cipher Suite: TLS_AES_256_GCM_SHA384 (0x1302)
                Cipher Suite: TLS_CHACHA20_POLY1305_SHA256 (0x1303)
                ...
            Compression Methods Length: 1
            Compression Methods (1 method)
                Compression Method: null (0)
            Extensions Length: 401
            Extension: server_name (len=14)
                Server Name Indication extension
                    Server Name: example.com
            Extension: supported_versions (len=5)
                Supported Versions: TLS 1.3, TLS 1.2
            Extension: supported_groups (len=10)
                Supported Groups: x25519, secp256r1, secp384r1
```

**Lecture :**
- Client propose : TLS 1.3 et TLS 1.2
- Cipher Suites : Liste des algorithmes de chiffrement supportés
- Server Name: example.com (SNI - Server Name Indication)
- Supported Groups : Courbes elliptiques pour échange de clés

**Paquet #301 : Server Hello ←**
```
TLSv1.3 Record Layer: Handshake Protocol: Server Hello
    Handshake Protocol: Server Hello
        Version: TLS 1.2 (0x0303)
        Random: ...
        Cipher Suite: TLS_AES_128_GCM_SHA256 (0x1301)
        Extension: supported_versions
            Supported Version: TLS 1.3 (0x0304)
        Extension: key_share
            Key Share extension
                Key Share Entry: Group: x25519
```

**Lecture :**
- Serveur choisit : **TLS 1.3**
- Cipher Suite sélectionné : AES_128_GCM
- Échange de clés : x25519 (Courbe elliptique)

**Paquets suivants : Handshake chiffré**
```
#302: TLSv1.3: Change Cipher Spec
#303: TLSv1.3: Application Data (Encrypted Handshake Message)
#304: TLSv1.3: Application Data (Certificate, CertificateVerify, Finished)
```

**Après le handshake :**
```
#310: TLSv1.3: Application Data (Encrypted)
    → Wireshark affiche "Application Data"
    → Contenu HTTP chiffré, illisible
```

**Pour déchiffrer TLS :**
```
Edit → Preferences → Protocols → TLS
→ (Pre)-Master-Secret log filename: /path/to/sslkeylog.txt

Variables d'environnement (Firefox/Chrome) :
export SSLKEYLOGFILE=/tmp/sslkeylog.txt
```

---

## Suivre les conversations

### Fonction "Follow Stream"

**Usage :**
```
Click droit sur un paquet TCP/UDP → Follow → TCP Stream (ou UDP Stream)
```

**Exemple : Suivre une conversation HTTP**

**Affichage :**
```
┌────────────────────────────────────────────────────────────┐
│ GET /api/users HTTP/1.1                                    │ Rouge (Client)
│ Host: api.example.com                                      │
│ Accept: application/json                                   │
│                                                            │
│ HTTP/1.1 200 OK                                            │ Bleu (Serveur)
│ Content-Type: application/json                             │
│ Content-Length: 1234                                       │
│                                                            │
│ {"users": [{"id": 1, "name": "Alice"}, ...]}               │
└────────────────────────────────────────────────────────────┘

Options :
[ Show data as: ASCII | EBCDIC | Hex Dump | C Arrays | Raw ]
[ Find: _______ ]
[ Save As... ] [ Print... ]
```

**Couleurs :**
- **Rouge** : Données client → serveur
- **Bleu** : Données serveur → client

**Formats d'affichage :**
- **ASCII** : Texte lisible (HTTP, FTP, SMTP)
- **Hex Dump** : Hexadécimal + ASCII (binaire)
- **Raw** : Données brutes (pour export)

**Filtrage automatique :**
Quand vous ouvrez "Follow TCP Stream", Wireshark applique un filtre :
```
tcp.stream eq 12
```
→ Affiche uniquement les paquets de cette conversation

### Fonction "Conversation" et "Endpoints"

**Statistics → Conversations**

Affiche toutes les conversations capturées :

```
┌────────────────────────────────────────────────────────────────────┐
│ IPv4 | IPv6 | TCP | UDP | Ethernet                                 │
├────────────────────────────────────────────────────────────────────┤
│ TCP Conversations                                                  │
│                                                                    │
│ Address A       Port A  Address B       Port B  Packets  Bytes     │
│ 192.168.1.10    52341   93.184.216.34   80      245      128KB     │
│ 192.168.1.10    52342   142.250.185.46  443     1024     2.5MB     │
│ 192.168.1.10    52343   10.0.5.20       3306    56       12KB      │
└────────────────────────────────────────────────────────────────────┘
```

**Utilité :**
- Voir d'un coup d'œil toutes les communications
- Identifier les top talkers (qui communique le plus)
- Clic droit → Apply as Filter → pour filtrer sur cette conversation

**Statistics → Endpoints**

Liste tous les endpoints (adresses IP) :

```
┌────────────────────────────────────────────────────────────┐
│ Address          Packets    Bytes      Tx Packets  Tx Bytes│
│ 192.168.1.10     5432       3.2MB      2156        1.5MB   │
│ 93.184.216.34    1245       512KB      789         256KB   │
│ 142.250.185.46   2345       1.8MB      1024        900KB   │
└────────────────────────────────────────────────────────────┘
```

---

## Mesurer les délais et performances

### Time Delta (Delta de temps)

**Afficher la colonne Time Delta :**
```
Click droit sur colonne → Column Preferences → Add
Field Type: Time delta from previous displayed packet
```

**Interprétation :**
```
No.  Time      Delta     Source        Dest          Protocol  Info
142  5.234567  0.001234  192.168.1.10  93.184.216.34 HTTP      GET /
143  5.280123  0.045556  93.184.216.34 192.168.1.10  TCP       ACK
144  5.281456  0.001333  192.168.1.10  93.184.216.34 TCP       ACK
145  5.360234  0.078778  93.184.216.34 192.168.1.10  HTTP      200 OK
```

**Analyse :**
- Delta #143 = 45.6 ms → RTT réseau (aller-retour)
- Delta #145 = 78.8 ms → Temps de traitement serveur + RTT

### Round Trip Time (RTT)

Wireshark calcule automatiquement le RTT pour TCP :

```
▼ Transmission Control Protocol
    [SEQ/ACK analysis]
        [This is an ACK to the segment in frame: 142]
        [The RTT to ACK the segment was: 0.045556 seconds]
        [iRTT: 0.045678 seconds]
```

**iRTT (initial RTT)**
- RTT mesuré pendant le handshake SYN → SYN-ACK
- **Baseline** pour les performances réseau

**RTT courant**
- Mesuré à chaque segment de données
- Permet de détecter une dégradation

### Time to First Byte (TTFB)

Pour HTTP, temps entre requête et première réponse :

```
Requête GET : Frame 142, Time = 5.234567
Réponse 200 : Frame 145, Time = 5.360234

TTFB = 5.360234 - 5.234567 = 0.125667 seconds = 125.7 ms
```

**Composition du TTFB :**
```
TTFB = RTT + Temps traitement serveur + RTT/2
     = 45.6 ms + 34.5 ms + 45.6 ms
     = 125.7 ms
```

### I/O Graph (Graphique de débit)

**Statistics → I/O Graph**

Affiche un graphique du trafic dans le temps :

```
┌─────────────────────────────────────────────────────────┐
│  Packets/s                                              │
│  1000 ┤                     ╭─╮                         │
│   800 ┤                   ╭─╯ ╰─╮                       │
│   600 ┤                 ╭─╯     ╰─╮                     │
│   400 ┤         ╭───╮ ╭─╯         ╰─╮                   │
│   200 ┤     ╭───╯   ╰─╯             ╰───╮               │
│     0 └─────┴────────────────────────────┴──────────►   │
│         0s        5s       10s       15s        20s     │
└─────────────────────────────────────────────────────────┘
```

**Configuration :**
- **Y Axis** : Packets, Bytes, Bits, Advanced (calculs personnalisés)
- **Interval** : 1 sec, 10 sec, 1 min
- **Filtres** : Ajouter plusieurs graphes avec des filtres différents

**Exemple multi-graphes :**
```
Graph 1: tcp (tout le TCP)
Graph 2: http (HTTP seulement)
Graph 3: tcp.analysis.retransmission (retransmissions)
```

---

## Identifier les problèmes

### Retransmissions TCP

**Symptôme dans Wireshark :**
```
No.  Time      Source        Dest          Protocol  Info
245  10.50000  192.168.1.10  93.184.216.34 TCP       [TCP Retransmission] ...
```

**Info colorée en noir**
- **[TCP Retransmission]** : Paquet retransmis car ACK non reçu
- Indique perte de paquets ou ACK perdu

**Dans Packet Details :**
```
[SEQ/ACK analysis]
    [This is a TCP retransmission of segment in frame: 242]
    [Expert Info (Note/Sequence): This is a TCP retransmission]
```

**Causes possibles :**
- Congestion réseau
- Lien instable (Wi-Fi, câble défectueux)
- Firewall qui drop des paquets
- Serveur surchargé

**Comment quantifier :**
```
Filtre : tcp.analysis.retransmission
→ Voir combien de retransmissions dans la capture

Taux de retransmission :
= (Paquets retransmis / Total paquets TCP) × 100
> 1% = Problème significatif
> 5% = Problème grave
```

### Duplicate ACK

```
No.  Info
250  [TCP Dup ACK 245#1] Ack=1234
251  [TCP Dup ACK 245#2] Ack=1234
252  [TCP Dup ACK 245#3] Ack=1234
```

**Signification :**
- Le récepteur a reçu des données hors séquence
- Il réenvoie le même ACK pour dire "je n'ai toujours pas reçu l'octet 1234"
- Après 3 Dup ACK → Fast Retransmit (retransmission rapide)

### TCP Zero Window

```
No.  Info
300  [TCP Window Full] Window=0
```

**Signification :**
- Le récepteur annonce Window=0
- "Stop ! Mon buffer est plein, n'envoie plus de données"
- L'émetteur doit attendre un Window Update

**Causes :**
- Application lente à lire les données
- CPU surchargé
- Problème de performance applicative

### TCP RST (Reset)

```
No.  Flags      Info
400  [RST, ACK] Connection reset
```

**Signification :**
- Fermeture brutale de la connexion
- Pas de four-way handshake normal (FIN)

**Causes courantes :**
- Port fermé (service pas à l'écoute)
- Firewall qui bloque
- Application crash
- Timeout (connexion idle trop longtemps)

### Checksum Errors

```
[Checksum: 0x1a2b [incorrect, should be 0x3c4d]]
[Expert Info (Error/Checksum): Bad checksum]
```

**Causes :**
- **Souvent un faux positif** : TCP Checksum Offloading
  - La carte réseau calcule le checksum en hardware
  - Wireshark capture avant ce calcul → voit un mauvais checksum
- Vrai problème : corruption de paquets (rare)

**Désactiver la vérification :**
```
Edit → Preferences → Protocols → TCP
☐ Validate the TCP checksum if possible
```

---

## Exporter des données

### Exporter des objets HTTP

**File → Export Objects → HTTP**

Liste tous les fichiers transférés via HTTP :

```
┌─────────────────────────────────────────────────────────────┐
│ Packet  Hostname         Content Type    Size    Filename   │
│ 145     example.com      text/html       5.2KB   /index.html│
│ 234     cdn.example.com  image/jpeg      125KB   /logo.jpg  │
│ 456     example.com      text/css        8.5KB   /style.css │
└─────────────────────────────────────────────────────────────┘

[Save] [Save All]
```

**Utilité :**
- Récupérer les fichiers transférés
- Analyser le contenu téléchargé
- Vérifier l'intégrité des fichiers

### Exporter un stream

**Follow → TCP Stream → Save As**

Sauvegarde le contenu de la conversation :
- **Raw** : Données brutes binaires
- **ASCII** : Texte seul

### Exporter en CSV

**File → Export Packet Dissections → As CSV**

Exporte la liste des paquets en CSV :
```
"No.","Time","Source","Destination","Protocol","Length","Info"
"1","0.000000","192.168.1.10","93.184.216.34","TCP","74","52341 → 80 [SYN]"
"2","0.045678","93.184.216.34","192.168.1.10","TCP","74","80 → 52341 [SYN, ACK]"
```

**Utilité :** Analyse statistique dans Excel, Python, R

---

## Expert Information

**Analyze → Expert Information**

Wireshark analyse automatiquement la capture et signale les problèmes :

```
┌──────────────────────────────────────────────────────────────┐
│ Severity │ Group     │ Protocol │ Count │ Summary            │
├──────────┼───────────┼──────────┼───────┼────────────────────┤
│ Error    │ Checksum  │ TCP      │ 12    │ Bad checksum       │
│ Warning  │ Sequence  │ TCP      │ 45    │ Retransmission     │
│ Warning  │ Sequence  │ TCP      │ 23    │ Dup ACK            │
│ Note     │ Sequence  │ TCP      │ 5     │ Window Full        │
│ Note     │ Sequence  │ TCP      │ 234   │ Window Update      │
│ Chat     │ Sequence  │ TCP      │ 1234  │ ACK to segment     │
└──────────┴───────────┴──────────┴───────┴────────────────────┘
```

**Niveaux de sévérité :**
- **Error** (Rouge) : Problèmes graves
- **Warning** (Jaune) : Avertissements
- **Note** (Cyan) : Informations
- **Chat** (Bleu) : Événements normaux

**Utilité :**
- Vue d'ensemble rapide des problèmes
- Identifier les patterns anormaux
- Quantifier les erreurs

---

## Astuces d'analyse

### 1. Utiliser les raccourcis clavier

```
Ctrl+F      : Rechercher
Ctrl+G      : Aller au paquet N
Ctrl+→      : Paquet suivant
Ctrl+←      : Paquet précédent
Ctrl+.      : Paquet suivant de même conversation
Ctrl+,      : Paquet précédent de même conversation
Ctrl+Alt+→  : Paquet référencé ([Request in frame: X])
Ctrl+Alt+←  : Retour
```

### 2. Marquer les paquets importants

```
Click droit → Mark/Unmark Packet (ou Ctrl+M)
```

Paquet marqué = Fond noir

**Utilité :**
- Marquer les paquets clés pendant l'analyse
- Retrouver rapidement les points d'intérêt

### 3. Ajouter des commentaires

```
Click droit → Packet Comment (ou Ctrl+Alt+C)
```

Ajouter une note sur un paquet :
```
"Premier SYN de la connexion problématique"
"Retransmission après 3s - timeout trop long"
```

**Les commentaires sont sauvegardés dans le fichier PCAPNG.**

### 4. Comparer deux captures

**Ouvrir deux instances de Wireshark :**
```
wireshark capture1.pcap &
wireshark capture2.pcap &
```

**Comparer :**
- Nombre de paquets
- RTT moyen
- Retransmissions
- Temps de réponse

**Exemple :**
```
Capture avant fix :
- Retransmissions : 234 (5.2%)
- RTT moyen : 125 ms
- TTFB : 450 ms

Capture après fix :
- Retransmissions : 12 (0.3%) ✅
- RTT moyen : 45 ms ✅
- TTFB : 180 ms ✅
```

### 5. Utiliser les profils de configuration

**Edit → Configuration Profiles**

Créer des profils pour différents usages :
- **Web** : Colonnes optimisées pour HTTP, filtres HTTP
- **Database** : Colonnes pour MySQL/PostgreSQL
- **Security** : Alertes sur patterns suspects
- **Performance** : Focus sur RTT, retransmissions

---

## Checklist d'analyse

Quand vous analysez une capture :

### ✅ Phase 1 : Vue d'ensemble (2 min)

```
1. Combien de paquets ? (si > 100k, filtrer)
2. Quels protocoles ? (Statistics → Protocol Hierarchy)
3. Quelles conversations ? (Statistics → Conversations)
4. Des erreurs ? (Analyze → Expert Information)
```

### ✅ Phase 2 : Identifier le problème (5-10 min)

```
5. Filtrer sur le protocole concerné (http, dns, tcp)
6. Chercher les paquets marqués en rouge/noir
7. Vérifier les retransmissions (tcp.analysis.retransmission)
8. Vérifier les resets (tcp.flags.reset == 1)
```

### ✅ Phase 3 : Analyse fine (10-30 min)

```
9. Suivre une conversation complète (Follow TCP Stream)
10. Mesurer les délais (Time Delta, RTT)
11. Examiner le handshake TCP (3-way, timing)
12. Vérifier le contenu applicatif (HTTP, DNS)
```

### ✅ Phase 4 : Conclusion (5 min)

```
13. Documenter les observations
14. Identifier la cause racine
15. Proposer une solution
16. Exporter les preuves (captures, screenshots)
```

---

## Points clés à retenir

### 🎯 Lecture d'un paquet

- **Lire de bas en haut** : Frame → Ethernet → IP → TCP → HTTP
- **Chaque couche ajoute** son en-tête (encapsulation)
- **Wireshark décode automatiquement** des centaines de protocoles

### 🎯 Suivre une conversation

- **Follow Stream** pour voir la conversation complète
- **tcp.stream eq N** pour filtrer une connexion spécifique
- **Couleurs** : Rouge = client→serveur, Bleu = serveur→client

### 🎯 Mesurer les performances

- **Time Delta** : Temps entre paquets
- **RTT** : Round Trip Time (aller-retour)
- **TTFB** : Time To First Byte (requête → réponse)

### 🎯 Identifier les problèmes

- **[TCP Retransmission]** : Perte de paquets
- **[TCP Dup ACK]** : Données hors séquence
- **[TCP Window Full]** : Récepteur saturé
- **[RST]** : Connexion fermée brutalement

### 🎯 Outils d'aide

- **Expert Information** : Vue d'ensemble des problèmes
- **I/O Graph** : Visualiser le trafic dans le temps
- **Statistics → Conversations** : Voir toutes les communications

---

## Conclusion

La lecture et l'interprétation des captures Wireshark est une compétence qui s'acquiert avec la pratique. Au début, vous devrez consulter la documentation pour chaque champ. Avec l'expérience, vous reconnaîtrez instantanément les patterns :

- Un handshake TCP normal vs problématique
- Une latence acceptable vs excessive
- Un trafic sain vs avec des retransmissions
- Une fermeture propre (FIN) vs brutale (RST)

**Conseil :** Analysez du trafic "normal" (vos propres navigations web, téléchargements) pour vous familiariser avec ce à quoi ressemble une communication saine. Vous pourrez ainsi plus facilement reconnaître les anomalies.

Dans la prochaine section, nous approfondirons les **filtres Wireshark**, l'outil le plus puissant pour isoler rapidement le trafic pertinent dans des captures de plusieurs milliers de paquets.

---

**Prochaine section :** 7.5 Filtres d'affichage et de capture

Nous maîtriserons la syntaxe des filtres pour extraire exactement ce que nous cherchons : filtrer par IP, port, protocole, contenu, état TCP, et bien plus encore.

⏭️ [Filtres d'affichage et de capture](/07-analyse-depannage/05-filtres-wireshark.md)
