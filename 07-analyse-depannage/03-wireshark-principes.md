🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.3 Analyse de trames avec Wireshark : principes

## Introduction

Les outils en ligne de commande permettent de diagnostiquer la plupart des problèmes réseau, mais certaines situations nécessitent une analyse plus fine, au niveau du paquet. C'est là qu'intervient **Wireshark**, l'analyseur de protocoles réseau le plus utilisé au monde.

Wireshark est l'équivalent d'un microscope pour le réseau : il permet d'observer chaque paquet individuellement, d'inspecter chaque bit des en-têtes, et de comprendre exactement ce qui se passe sur le fil. Que vous soyez développeur cherchant à comprendre pourquoi votre API ne répond pas, administrateur diagnostiquant une lenteur réseau, ou expert sécurité analysant du trafic suspect, Wireshark est un outil indispensable.

Cette section pose les fondations : nous allons comprendre comment Wireshark fonctionne, découvrir son interface, et apprendre les concepts essentiels avant de plonger dans l'analyse concrète de captures dans les sections suivantes.

---

## Qu'est-ce que Wireshark ?

### Définition et historique

**Wireshark** (anciennement Ethereal) est un analyseur de protocoles réseau (packet sniffer / network analyzer) gratuit et open-source. Il permet de :

- **Capturer** le trafic réseau en temps réel
- **Analyser** les paquets capturés
- **Décoder** automatiquement des centaines de protocoles
- **Filtrer** pour isoler le trafic pertinent
- **Exporter** les données pour analyse ultérieure
- **Inspecter** le contenu des paquets bit par bit

**Historique :**
- **1998** : Création par Gerald Combs sous le nom "Ethereal"
- **2006** : Renommé "Wireshark" pour des raisons légales
- **Aujourd'hui** : Standard de facto de l'industrie, utilisé par des millions de professionnels

### Wireshark vs tcpdump

| Caractéristique | Wireshark | tcpdump |
|----------------|-----------|---------|
| **Interface** | Graphique (GUI) | Ligne de commande |
| **Plateforme** | Windows, Linux, macOS | Linux, macOS, BSD |
| **Usage principal** | Analyse interactive | Capture automatisée, scripts |
| **Courbe d'apprentissage** | Plus facile visuellement | Plus technique |
| **Performance** | Consomme plus de ressources | Très léger |
| **Décodage** | Automatique et visuel | Manuel (texte brut) |
| **Filtres** | Syntaxe conviviale | BPF (plus complexe) |

**Complémentarité :**
- `tcpdump` pour capturer sur un serveur distant sans interface graphique
- Wireshark pour analyser confortablement les captures enregistrées

**Équivalence :**
```bash
# Capture avec tcpdump
sudo tcpdump -i eth0 -w capture.pcap

# Puis analyse avec Wireshark
wireshark capture.pcap
```

### À quoi sert Wireshark concrètement ?

#### Cas d'usage 1 : Développeur - Débugger une API REST

**Situation :** Votre application cliente reçoit une erreur HTTP 500 mystérieuse.

**Avec Wireshark :**
```
1. Capturer le trafic entre le client et le serveur API
2. Voir la requête HTTP exacte envoyée (headers, body)
3. Voir la réponse HTTP complète du serveur
4. Identifier que le serveur répond bien 500 avec un message d'erreur JSON
5. Constater que la requête contenait un header mal formé
```

**Résultat :** Problème identifié en 5 minutes au lieu de plusieurs heures de logs.

#### Cas d'usage 2 : Administrateur système - Diagnostiquer une lenteur

**Situation :** Les utilisateurs se plaignent que le site web est lent.

**Avec Wireshark :**
```
1. Capturer le trafic HTTP/HTTPS
2. Mesurer le temps entre SYN et ACK (connexion TCP)
3. Mesurer le temps entre requête HTTP et réponse
4. Identifier que le serveur met 5 secondes à répondre après la requête
5. Constater de nombreuses retransmissions TCP
```

**Résultat :** Problème de performance réseau identifié (perte de paquets).

#### Cas d'usage 3 : Expert sécurité - Analyser un incident

**Situation :** Suspicion d'exfiltration de données depuis un serveur compromis.

**Avec Wireshark :**
```
1. Capturer tout le trafic sortant du serveur suspect
2. Filtrer les connexions vers des IPs externes
3. Identifier des connexions HTTPS vers une IP inconnue (C&C)
4. Analyser les certificats TLS utilisés (auto-signés, suspects)
5. Exporter les fichiers transférés pour analyse forensique
```

**Résultat :** Confirmation de l'incident, IP malveillante identifiée.

---

## Comment fonctionne Wireshark ?

### Architecture de capture

Wireshark fonctionne en deux phases distinctes :

```
Phase 1 : CAPTURE                Phase 2 : ANALYSE
┌─────────────────┐              ┌──────────────────┐
│  Carte réseau   │              │  Fichier PCAP    │
│  (mode promiscuous)            │  (capture.pcap)  │
└────────┬────────┘              └────────┬─────────┘
         │                                │
         ▼                                ▼
┌─────────────────┐              ┌──────────────────┐
│  Libpcap/WinPcap│              │   Wireshark GUI  │
│  (capture brute)│              │   (décodage)     │
└────────┬────────┘              └──────────────────┘
         │
         ▼
┌─────────────────┐
│  Fichier PCAP   │
│  (stockage)     │
└─────────────────┘
```

### Les bibliothèques de capture

Wireshark s'appuie sur des bibliothèques systèmes pour capturer les paquets :

**Linux/macOS :**
- **libpcap** : Bibliothèque standard de capture de paquets
- Accès bas niveau à la carte réseau
- Nécessite les privilèges root/sudo

**Windows :**
- **Npcap** (successeur de WinPcap)
- Driver en mode kernel pour capturer les paquets
- Installation requise en plus de Wireshark

**Installation :**
```bash
# Linux (Debian/Ubuntu)
sudo apt install wireshark tshark

# Linux (RedHat/CentOS)
sudo yum install wireshark

# macOS (avec Homebrew)
brew install --cask wireshark

# Windows
# Télécharger depuis https://www.wireshark.org/
# Installer Wireshark + Npcap
```

### Mode promiscuous

**Mode normal :**
La carte réseau ne reçoit que :
- Les paquets destinés à son adresse MAC
- Les paquets broadcast
- Les paquets multicast pour les groupes auxquels elle appartient

**Mode promiscuous (monitoring) :**
La carte réseau reçoit **TOUS** les paquets sur le segment réseau, même ceux destinés à d'autres machines.

```
Réseau commuté (switch) :
┌────────────────────────────────────┐
│  Switch                            │
│                                    │
│  Port 1 : PC-A (192.168.1.10)      │
│  Port 2 : PC-B (192.168.1.20)      │
│  Port 3 : PC-C (192.168.1.30)      │
│  Port 4 : Wireshark (monitoring)   │
└────────────────────────────────────┘

Mode normal (Port 4) :
  → Voit uniquement le trafic de/vers 192.168.1.30

Mode promiscuous (Port 4) :
  → Voit son propre trafic UNIQUEMENT (switch isole)

Port mirroring configuré (Port 4) :
  → Voit TOUT le trafic (ports 1, 2, 3 mirrorés)
```

**Important sur les réseaux modernes :**
- Sur un **hub** (obsolète) : mode promiscuous voit tout le trafic
- Sur un **switch** : ne voit que son propre trafic, sauf si :
  - Configuration de **port mirroring/SPAN**
  - ARP poisoning (attaque active, illégal)
  - Monitoring depuis la passerelle/routeur

**Activation du mode promiscuous :**
```
Wireshark → Capture → Options → [Cocher] "Promiscuous mode"
```

### Format de fichier : PCAP

Wireshark enregistre les captures au format **PCAP** (Packet Capture) ou **PCAPNG** (PCAP Next Generation).

**PCAP :**
- Format historique, très compatible
- Supporté par tous les outils (tcpdump, Wireshark, tshark, etc.)
- Limitations : pas de métadonnées riches

**PCAPNG (recommandé) :**
- Format moderne, plus flexible
- Supporte plusieurs interfaces simultanément
- Métadonnées : commentaires, résolution DNS, infos système
- Conserve plus de contexte

**Structure d'un fichier PCAP :**
```
┌──────────────────────┐
│  Global Header       │  ← Informations générales
├──────────────────────┤
│  Packet 1 Header     │  ← Timestamp, longueur
│  Packet 1 Data       │  ← Données brutes du paquet
├──────────────────────┤
│  Packet 2 Header     │
│  Packet 2 Data       │
├──────────────────────┤
│  ...                 │
└──────────────────────┘
```

**Compatibilité :**
```bash
# Lire un PCAP avec différents outils
wireshark capture.pcap
tcpdump -r capture.pcap
tshark -r capture.pcap
```

---

## L'interface de Wireshark

### Vue d'ensemble

L'interface Wireshark est divisée en plusieurs zones principales :

```
┌────────────────────────────────────────────────────────────┐
│  Barre de menu  [File Edit View Go Capture...]             │
├────────────────────────────────────────────────────────────┤
│  Barre d'outils [🔍 Capture Options] [⏹️ Stop] [🔄 Refresh]│
├────────────────────────────────────────────────────────────┤
│  Filtres d'affichage  [http.request.method == "POST"    ]  │
├────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐  │
│  │  PACKET LIST PANE (Liste des paquets)                │  │
│  │  No.  Time      Source     Dest       Protocol Info  │  │
│  │  1    0.000000  10.0.0.1   10.0.0.2   TCP      ...   │  │
│  │  2    0.001234  10.0.0.2   10.0.0.1   TCP      ...   │  │
│  │  3    0.002456  10.0.0.1   10.0.0.2   HTTP     GET   │  │
│  └──────────────────────────────────────────────────────┘  │
├────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐  │
│  │  PACKET DETAILS PANE (Détails du paquet)             │  │
│  │  ▶ Frame 3: 514 bytes on wire                        │  │
│  │  ▼ Ethernet II                                       │  │
│  │    ▶ Internet Protocol Version 4, Src: 10.0.0.1      │  │
│  │    ▶ Transmission Control Protocol                   │  │
│  │    ▼ Hypertext Transfer Protocol                     │  │
│  │        GET /api/users HTTP/1.1                       │  │
│  └──────────────────────────────────────────────────────┘  │
├────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐  │
│  │  PACKET BYTES PANE (Octets bruts)                    │  │
│  │  0000  00 1a 2b 3c 4d 5e 6f 70  81 92 a3 b4 c5 d6    │  │
│  │  0010  47 45 54 20 2f 61 70 69  2f 75 73 65 72 73    │  │
│  │  0020  ...                                           │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

### 1. Packet List Pane (Liste des paquets)

**Rôle :** Vue d'ensemble de tous les paquets capturés.

**Colonnes par défaut :**

| Colonne | Description |
|---------|-------------|
| **No.** | Numéro séquentiel du paquet dans la capture |
| **Time** | Timestamp (temps depuis le début de la capture) |
| **Source** | Adresse IP ou MAC source |
| **Destination** | Adresse IP ou MAC destination |
| **Protocol** | Protocole de plus haut niveau détecté (HTTP, DNS, TCP, etc.) |
| **Length** | Longueur totale du paquet (bytes) |
| **Info** | Résumé du contenu (très utile !) |

**Exemple de ligne :**
```
No.  Time      Source        Destination   Protocol  Length  Info
142  5.234521  192.168.1.10  142.250.1.46  HTTP      512     GET /search?q=wireshark HTTP/1.1
```

**Lecture :**
- Paquet n°142 de la capture
- Capturé à 5.234521 secondes après le début
- Source : 192.168.1.10 (probablement le client)
- Destination : 142.250.1.46 (serveur Google)
- Protocole : HTTP
- Taille : 512 bytes
- Info : Requête GET vers /search?q=wireshark

**Personnalisation des colonnes :**
```
Click droit sur l'en-tête → Column Preferences
Ajouter/supprimer/réorganiser les colonnes

Colonnes utiles à ajouter :
- Source Port
- Destination Port
- TCP Stream Index (pour suivre les conversations)
- Delta Time (temps depuis le paquet précédent)
```

### 2. Packet Details Pane (Détails hiérarchiques)

**Rôle :** Décodage hiérarchique du paquet sélectionné, couche par couche.

**Structure typique (paquet HTTP) :**

```
▼ Frame 142: 512 bytes on wire (4096 bits), 512 bytes captured
    Interface id: 0 (en0)
    Arrival Time: Dec  7, 2025 10:30:45.234521000 CET
    Epoch Time: 1733566245.234521000 seconds
    [Time delta from previous captured frame: 0.012345000 seconds]

▼ Ethernet II, Src: Apple_12:34:56 (a4:83:e7:12:34:56), Dst: Netgear_ab:cd:ef (00:1f:33:ab:cd:ef)
    Destination: Netgear_ab:cd:ef (00:1f:33:ab:cd:ef)
    Source: Apple_12:34:56 (a4:83:e7:12:34:56)
    Type: IPv4 (0x0800)

▼ Internet Protocol Version 4, Src: 192.168.1.10, Dst: 142.250.1.46
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00
    Total Length: 498
    Identification: 0x1a2b (6699)
    Flags: 0x4000, Don't fragment
    Time to Live: 64
    Protocol: TCP (6)
    Header Checksum: 0x3c4d [validation disabled]
    Source Address: 192.168.1.10
    Destination Address: 142.250.1.46

▼ Transmission Control Protocol, Src Port: 52341, Dst Port: 80, Seq: 1, Ack: 1, Len: 458
    Source Port: 52341
    Destination Port: 80
    [Stream index: 12]
    Sequence Number: 1    (relative sequence number)
    Acknowledgment Number: 1    (relative ack number)
    1000 .... = Header Length: 32 bytes (8)
    Flags: 0x018 (PSH, ACK)
        ...0 .... = Congestion Window Reduced (CWR): Not set
        .... 0... = ECN-Echo: Not set
        .... .0.. = Urgent: Not set
        .... ..1. = Acknowledgment: Set
        .... ...1 = Push: Set
        0... .... = Reset: Not set
        .0.. .... = Syn: Not set
        ..0. .... = Fin: Not set
    Window: 65535
    Checksum: 0x5e6f [unverified]
    Urgent Pointer: 0
    [SEQ/ACK analysis]
    TCP payload (458 bytes)

▼ Hypertext Transfer Protocol
    GET /search?q=wireshark HTTP/1.1\r\n
    Host: www.google.com\r\n
    User-Agent: Mozilla/5.0\r\n
    Accept: text/html,application/xhtml+xml\r\n
    Accept-Language: en-US,en;q=0.9\r\n
    Accept-Encoding: gzip, deflate, br\r\n
    Connection: keep-alive\r\n
    \r\n
    [Full request URI: http://www.google.com/search?q=wireshark]
    [Request in frame: 142]
    [Response in frame: 145]
```

**Points clés :**

**▼ Sections dépliables/repliables**
- Cliquer sur ▶ pour déplier, ▼ pour replier
- Permet de naviguer facilement entre les couches

**Codes couleur**
- **Noir** : Champs normaux
- **Bleu** : Champs cliquables (liens vers d'autres paquets)
- **Rouge** : Erreurs, checksums invalides
- **Gris** : Informations calculées par Wireshark (pas dans le paquet)

**Informations entre crochets [ ]**
- Générées par Wireshark pour aider l'analyse
- Ne font pas partie du paquet capturé
- Exemple : `[Request in frame: 142]` → Wireshark a associé requête et réponse

### 3. Packet Bytes Pane (Vue hexadécimale)

**Rôle :** Affichage brut des octets du paquet.

**Format :**
```
Offset    Hexadécimal                         ASCII
0000   00 1f 33 ab cd ef a4 83  e7 12 34 56 08 00 45 00   ..3.......4V..E.
0010   01 f2 1a 2b 40 00 40 06  3c 4d c0 a8 01 0a 8e fa   ...+@.@.<M......
0020   01 2e cc 85 00 50 00 00  00 01 00 00 00 01 80 18   .....P..........
0030   ff ff 5e 6f 00 00 01 01  08 0a 00 12 34 56 00 78   ..^o........4V.x
0040   47 45 54 20 2f 73 65 61  72 63 68 3f 71 3d 77 69   GET /search?q=wi
0050   72 65 73 68 61 72 6b 20  48 54 54 50 2f 31 2e 31   reshark HTTP/1.1
```

**Structure :**
- **Offset** : Position dans le paquet (en hexadécimal)
- **Hex** : 16 octets en hexadécimal, séparés par groupes de 8
- **ASCII** : Représentation ASCII (`.` pour non-imprimables)

**Surlignage automatique :**
Quand vous cliquez sur un champ dans le Packet Details Pane, les octets correspondants sont surlignés dans le Packet Bytes Pane.

**Exemple :**
```
Clic sur "Source Port: 52341" dans Packet Details
→ Les octets "cc 85" (52341 en hex) sont surlignés en Packet Bytes
```

**Utilité :**
- Vérifier manuellement le contenu exact
- Identifier des patterns binaires
- Analyser des protocoles non reconnus par Wireshark
- Exporter des données brutes

### Barre de filtres d'affichage

**Rôle :** Filtrer les paquets affichés sans modifier la capture.

**Emplacement :** Juste au-dessus du Packet List Pane

```
┌────────────────────────────────────────────────────────┐
│  [  http.request.method == "POST"                  ]   │
│  [Appliquer] [Effacer] [▼ Filtres récents]             │
└────────────────────────────────────────────────────────┘
```

**Caractéristiques :**
- **Fond vert** : Filtre valide syntaxiquement
- **Fond rouge** : Filtre invalide (erreur de syntaxe)
- **Fond jaune** : Filtre valide mais ambigu

**Exemples de filtres simples :**
```
ip.addr == 192.168.1.10        # Tout trafic de/vers cette IP
tcp.port == 443                # Tout trafic HTTPS
http                           # Uniquement HTTP
dns.qry.name == "google.com"   # Requêtes DNS pour google.com
```

**Nous approfondirons les filtres dans la section 7.5.**

---

## Les modes de fonctionnement

### Mode capture en direct (Live Capture)

**Description :** Capturer le trafic réseau en temps réel.

**Procédure :**
```
1. Menu : Capture → Options (ou Ctrl+K)
2. Sélectionner l'interface réseau
3. (Optionnel) Configurer un filtre de capture
4. Cliquer "Start"
5. Observer les paquets défiler
6. Cliquer "Stop" pour arrêter (ou Ctrl+E)
```

**Sélection de l'interface :**
```
Interfaces disponibles :
┌──────────────────────────────────────────────────────┐
│ ☑ eth0      │ 192.168.1.10  │ Ethernet physique      │
│ ☐ wlan0     │ 192.168.1.11  │ Wi-Fi                  │
│ ☐ lo        │ 127.0.0.1     │ Loopback (localhost)   │
│ ☐ docker0   │ 172.17.0.1    │ Réseau Docker          │
│ ☐ tun0      │ 10.8.0.1      │ VPN                    │
└──────────────────────────────────────────────────────┘
```

**Conseils de sélection :**
- **eth0/en0** : Carte Ethernet câblée
- **wlan0/en1** : Carte Wi-Fi
- **lo** : Trafic local uniquement (localhost)
- **docker0** : Trafic entre conteneurs Docker
- **any** (Linux) : Capturer sur TOUTES les interfaces simultanément

**Indicateurs visuels en capture :**
```
┌────────────────────────────────────────────┐
│  🔴 Capturing on 'eth0'                    │
│  Packets: 1,234   Displayed: 567           │
│  Dropped: 2   Capture filter: tcp port 80  │
└────────────────────────────────────────────┘
```

- **Packets** : Nombre total de paquets capturés
- **Displayed** : Nombre de paquets affichés (après filtre d'affichage)
- **Dropped** : Paquets perdus (buffer plein, CPU saturé) ⚠️

**⚠️ Dropped packets :**
Si des paquets sont dropped, votre analyse sera incomplète. Solutions :
- Augmenter la taille du buffer
- Utiliser un filtre de capture plus restrictif
- Capturer sur une machine plus puissante
- Sauvegarder directement sur disque sans affichage

### Mode analyse de fichier (Offline Analysis)

**Description :** Ouvrir et analyser une capture précédemment enregistrée.

**Procédure :**
```
1. Menu : File → Open (ou Ctrl+O)
2. Sélectionner le fichier .pcap ou .pcapng
3. Analyser à votre rythme
```

**Avantages :**
- Pas de risque de perdre des paquets
- Analyse répétable et partageabe
- Peut être fait sur une autre machine
- Permet d'annoter et documenter

**Format de fichier supportés :**
- `.pcap` / `.pcapng` : Format standard
- `.cap` : Microsoft Network Monitor
- `.snoop` : Sun Snoop
- `.erf` : Endace ERF
- Et ~50 autres formats

**Exemple de workflow :**
```bash
# Sur le serveur (capture)
sudo tcpdump -i eth0 -w /tmp/debug.pcap

# Transférer vers votre PC
scp user@server:/tmp/debug.pcap ./

# Analyser avec Wireshark (sur PC)
wireshark debug.pcap
```

---

## Bonnes pratiques de capture

### 1. Limiter la taille des captures

**Problème :** Une capture de 10 Go est difficile à analyser.

**Solutions :**

**A. Limiter par nombre de paquets :**
```
Capture Options → Stop Capture After: 10,000 packets
```

**B. Limiter par taille de fichier :**
```
Capture Options → Stop Capture After: 100 MB
```

**C. Limiter par durée :**
```
Capture Options → Stop Capture After: 60 seconds
```

**D. Rotation de fichiers :**
```
Capture Options → Output
├─ File: capture.pcap
├─ Create a new file automatically
│  └─ After: 10,000 packets (ou 10 MB, ou 60 seconds)
└─ Use a ring buffer with: 5 files

Résultat : capture_00001.pcap, capture_00002.pcap, ..., capture_00005.pcap
Puis écrase capture_00001.pcap (rotation)
```

**Utilité :** Capturer en continu sans remplir le disque.

### 2. Utiliser les filtres de capture (Capture Filters)

**Différence importante :**
- **Filtre de capture** : Décide quels paquets capturer (libpcap/BPF)
- **Filtre d'affichage** : Masque des paquets déjà capturés (Wireshark)

**Filtre de capture = Performance**

Un filtre de capture bien choisi :
- ✅ Réduit la taille du fichier
- ✅ Réduit la charge CPU/RAM
- ✅ Évite les paquets dropped
- ✅ Protège la confidentialité (ne capture pas tout)

**Syntaxe BPF (Berkeley Packet Filter) :**
```
Capture Options → Capture Filter

Exemples :
host 192.168.1.10              # Trafic de/vers cette IP
port 443                       # Trafic HTTPS
tcp port 80 or tcp port 443    # HTTP ou HTTPS
net 192.168.1.0/24             # Tout le subnet
not port 22                    # Tout SAUF SSH
src host 10.0.0.5              # Origine uniquement
dst port 53                    # Destination DNS
```

**Exemples concrets :**

```bash
# Capturer uniquement le trafic entre deux serveurs
host 10.0.5.20 and host 10.0.8.50

# Capturer HTTP/HTTPS uniquement
tcp port 80 or tcp port 443

# Capturer tout sauf le SSH (pour ne pas capturer votre session)
not port 22

# Capturer DNS et HTTPS
port 53 or port 443

# Capturer vers un subnet spécifique
dst net 10.0.0.0/8
```

**⚠️ Attention :**
- Les filtres de capture sont **définitifs** (paquets non capturés = perdus)
- Préférer un filtre large et filtrer l'affichage ensuite si pas sûr
- Syntaxe BPF différente des filtres d'affichage Wireshark

### 3. Capturer au bon endroit

**Question cruciale :** Où placer Wireshark pour capturer le bon trafic ?

**Scénario 1 : Application client/serveur**
```
[Client] ──(1)── [Firewall] ──(2)── [Load Balancer] ──(3)── [Serveur]

Où capturer ?
(1) Sur le client → Voir ce que le client envoie/reçoit
(2) Avant le firewall → Voir si le firewall bloque
(3) Après le load balancer → Voir ce que le serveur reçoit

Mieux : Capturer en (1) ET (3) pour comparer
```

**Scénario 2 : Problème réseau local**
```
[PC-A] ─┬─ [Switch] ─── [Internet]
        │
[PC-B] ─┘

Capturer sur PC-A : voit uniquement son trafic (switch isole)
Capturer sur PC-B : voit uniquement son trafic

Solution :
- Port mirroring sur le switch (copie tout le trafic vers un port monitoring)
- Ou capturer sur la passerelle/routeur
```

**Scénario 3 : Trafic HTTPS**
```
[Navigateur] ── HTTPS (chiffré) ── [Serveur]

Wireshark voit : paquets TCP, handshake TLS, mais données chiffrées

Pour voir le contenu HTTP :
- Configurer SSLKEYLOGFILE (exporter les clés TLS)
- Ou capturer côté serveur (avec accès au certificat privé)
```

### 4. Documenter la capture

**Métadonnées à noter :**
```
Date/heure : 2025-12-07 10:30:00
Durée : 120 secondes
Interface : eth0 (192.168.1.10)
Filtre de capture : tcp port 443
Contexte : Analyse lenteur API production
Problème observé : Timeouts après 30s
```

**Dans Wireshark :**
```
Menu : Statistics → Capture File Properties

Ajouter des commentaires :
- Cliquer sur "Edit" à côté de "Capture file comments"
- Décrire le contexte, le problème, les conditions
```

**Nommer les fichiers intelligemment :**
```
❌ Mauvais : capture.pcap, test.pcap, new.pcap

✅ Bon :
2025-12-07_api-timeout_production_eth0.pcap
2025-12-07_dns-issue_client-192.168.1.50.pcap
incident-12345_http-500_before-fix.pcap
incident-12345_http-500_after-fix.pcap
```

### 5. Respecter la confidentialité et la légalité

**Rappel légal :**
- Capturer du trafic réseau sans autorisation est **illégal**
- Même sur un réseau d'entreprise, une autorisation est requise
- RGPD/GDPR : les paquets peuvent contenir des données personnelles

**Bonnes pratiques :**
```
✅ À faire :
- Obtenir une autorisation écrite
- Capturer uniquement ce qui est nécessaire au diagnostic
- Anonymiser les captures avant de les partager
- Supprimer les captures après analyse
- Ne pas capturer de mots de passe, tokens, données sensibles

❌ À ne pas faire :
- Capturer sur un réseau public/Wi-Fi ouvert
- Capturer du trafic d'autres utilisateurs sans raison
- Partager des captures contenant des données sensibles
- Garder des captures indéfiniment
```

**Anonymiser une capture :**
```
Menu : Edit → Preferences → Protocols → IPv4
☑ Anonymize IPv4 addresses

Ou avec tcprewrite :
tcprewrite --infile=original.pcap --outfile=anonymized.pcap \
           --seed=123 --skipbroadcast --pnat=10.0.0.0/8:192.168.0.0/16
```

---

## Limitations de Wireshark

### Ce que Wireshark peut faire

- ✅ Capturer et analyser le trafic réseau
- ✅ Décoder automatiquement des centaines de protocoles
- ✅ Mesurer les temps de réponse, latences
- ✅ Identifier les retransmissions, erreurs
- ✅ Exporter des données (fichiers, flux)
- ✅ Statistiques de trafic
- ✅ Déchiffrer TLS (avec les clés)

### Ce que Wireshark NE peut PAS faire

- ❌ **Générer du trafic** (pas un outil de test de charge)
- ❌ **Bloquer du trafic** (pas un firewall)
- ❌ **Modifier des paquets en transit** (pas un proxy)
- ❌ **Casser le chiffrement** sans les clés
- ❌ **Capturer sur un switch** sans port mirroring
- ❌ **Diagnostiquer un problème applicatif** (juste le réseau)

### Limitations techniques

**Performance :**
- Capture intensive peut ralentir la machine
- Fichiers de plusieurs GB difficiles à manipuler
- Affichage en temps réel limité à ~10,000 paquets/sec

**Réseau commuté :**
- Sur un switch moderne, ne voit que son propre trafic
- Nécessite port mirroring/SPAN pour voir le trafic des autres

**Chiffrement :**
- TLS/SSL : contenu chiffré (sauf avec SSLKEYLOGFILE)
- VPN : tunnel chiffré (voit juste du ESP/UDP opaque)

**Protocoles propriétaires :**
- Protocoles inconnus non décodés automatiquement
- Peut nécessiter l'écriture de dissectors personnalisés

---

## Concepts importants

### Stream vs Packet

**Packet (Paquet) :**
- Une unité individuelle de données
- Peut être une trame Ethernet, un paquet IP, un segment TCP

**Stream (Flux/Conversation) :**
- Un ensemble de paquets liés constituant une communication complète
- Exemple : Une conversation TCP entre client:52341 et serveur:443

**Wireshark peut regrouper les paquets en streams :**
```
Click droit sur un paquet TCP → Follow → TCP Stream

Wireshark affiche :
┌────────────────────────────────────────────┐
│ GET /api/users HTTP/1.1                    │ ← Client
│ Host: api.example.com                      │
│                                            │
│ HTTP/1.1 200 OK                            │ ← Serveur
│ Content-Type: application/json             │
│ {"users": [...]}                           │
└────────────────────────────────────────────┘
```

**Utilité :** Voir la conversation complète sans chercher manuellement chaque paquet.

### Time Display Format

Wireshark offre plusieurs formats d'affichage du temps :

**Menu : View → Time Display Format**

| Format | Description | Exemple |
|--------|-------------|---------|
| **Seconds Since Beginning** | Secondes depuis le 1er paquet | 5.234521 |
| **Date and Time of Day** | Date/heure absolue | 2025-12-07 10:30:45.234521 |
| **Seconds Since Previous Displayed** | Delta depuis paquet précédent affiché | 0.012345 |
| **Seconds Since Previous Captured** | Delta depuis paquet précédent capturé | 0.001234 |
| **UTC Date and Time** | Date/heure UTC | 2025-12-07 09:30:45.234521 UTC |

**Cas d'usage :**
- **Seconds Since Beginning** : Voir la timeline de la capture
- **Delta** : Identifier des délais anormaux entre paquets
- **Absolute** : Corréler avec des logs système

### Colorisation des paquets

Wireshark colore automatiquement les paquets pour faciliter l'identification :

**Règles de couleur par défaut :**

| Couleur | Protocole/Type | Signification |
|---------|----------------|---------------|
| **Violet clair** | TCP | Trafic TCP normal |
| **Bleu clair** | UDP | Trafic UDP |
| **Vert clair** | HTTP | Requêtes/réponses HTTP |
| **Jaune clair** | Routage | Paquets de routage (OSPF, BGP) |
| **Noir** | TCP (erreur) | Retransmissions, RST, erreurs |
| **Rouge** | Problème | Checksums invalides, erreurs |
| **Gris** | TCP (info) | Paquets de contrôle (ACK seul) |

**Personnaliser les couleurs :**
```
Menu : View → Coloring Rules

Exemples de règles personnalisées :
- Tout le trafic vers 192.168.1.50 en jaune
- Paquets HTTP POST en orange
- DNS queries en bleu foncé
```

**Désactiver la colorisation :**
```
Menu : View → Colorize Packet List (décocher)
```

---

## Premiers pas : Capturer votre premier trafic

### Exercice guidé : Capturer une requête HTTP

**Objectif :** Capturer et observer une simple requête web.

**Étape 1 : Démarrer la capture**
```
1. Ouvrir Wireshark
2. Sélectionner votre interface réseau active (eth0, wlan0, etc.)
3. Cliquer "Start capturing packets" (aileron de requin bleu)
```

**Étape 2 : Générer du trafic**
```
Dans votre navigateur :
http://example.com
```

**Étape 3 : Arrêter la capture**
```
Dans Wireshark : Cliquer le carré rouge "Stop"
```

**Étape 4 : Filtrer pour voir HTTP**
```
Dans la barre de filtre, taper :
http

Appuyer sur Entrée
```

**Étape 5 : Observer les paquets**
```
Vous devriez voir :
- Des lignes avec "GET / HTTP/1.1"
- Des lignes avec "HTTP/1.1 200 OK"
```

**Étape 6 : Suivre le stream TCP**
```
Click droit sur une ligne HTTP GET → Follow → TCP Stream

Observer la conversation complète :
GET / HTTP/1.1
Host: example.com
...

HTTP/1.1 200 OK
Content-Type: text/html
...
<html>...</html>
```

**Étape 7 : Examiner les détails**
```
Cliquer sur un paquet HTTP GET
Dans Packet Details Pane, déplier :
▼ Hypertext Transfer Protocol
    GET / HTTP/1.1
    Host: example.com
    User-Agent: Mozilla/5.0...
```

**Félicitations !** Vous venez de capturer et analyser votre premier trafic réseau.

### Ce qu'on peut observer

Dans cette simple capture HTTP, on peut voir :

**Au niveau DNS :**
```
Requête : "Quelle est l'IP de example.com ?"
Réponse : "93.184.216.34"
```

**Au niveau TCP :**
```
SYN → (Client demande connexion)
SYN-ACK ← (Serveur accepte)
ACK → (Connexion établie : 3-way handshake)
```

**Au niveau HTTP :**
```
GET / HTTP/1.1 → (Requête du client)
HTTP/1.1 200 OK ← (Réponse du serveur)
```

**Au niveau des performances :**
```
Temps DNS : 15ms
Temps 3-way handshake : 45ms
Temps réponse HTTP : 120ms
Total : ~180ms
```

---

## Résumé des concepts clés

### Architecture de Wireshark

```
┌─────────────────────────────────────────┐
│         Interface Wireshark GUI         │
├─────────────────────────────────────────┤
│     Décodeurs de protocoles (dissectors)│
├─────────────────────────────────────────┤
│     Moteur de capture (libpcap/Npcap)   │
├─────────────────────────────────────────┤
│     Carte réseau (mode promiscuous)     │
└─────────────────────────────────────────┘
```

### Les 3 vues principales

1. **Packet List** : Vue d'ensemble (temps, source, dest, protocole)
2. **Packet Details** : Décodage hiérarchique (couche par couche)
3. **Packet Bytes** : Données brutes (hexadécimal + ASCII)

### Filtres

- **Filtre de capture** : Décide quoi capturer (BPF, avant stockage)
- **Filtre d'affichage** : Décide quoi afficher (Wireshark, après stockage)

### Modes

- **Live Capture** : Capture en temps réel
- **Offline Analysis** : Analyse de fichiers .pcap

### Bonnes pratiques

- ✅ Limiter la taille des captures
- ✅ Utiliser des filtres de capture
- ✅ Documenter le contexte
- ✅ Respecter la confidentialité
- ✅ Sauvegarder au format PCAPNG

---

## Points de vigilance

### ⚠️ Performance

```
Problème : Wireshark ralentit ou perd des paquets

Solutions :
- Utiliser un filtre de capture restrictif
- Augmenter le buffer : Capture Options → Buffer size
- Désactiver la résolution DNS : Edit → Preferences → Name Resolution
- Capturer sans afficher : tshark -w file.pcap
```

### ⚠️ Droits d'accès

```
Problème : "You don't have permission to capture on that device"

Solutions Linux :
sudo wireshark                    # Simple mais pas recommandé
sudo usermod -aG wireshark $USER  # Ajouter user au groupe
sudo dpkg-reconfigure wireshark-common  # Reconfigurer permissions

Puis déconnexion/reconnexion
```

### ⚠️ Trafic chiffré

```
Problème : Contenu HTTPS illisible (chiffré)

Solutions :
- Exporter les clés TLS : SSLKEYLOGFILE
- Capturer côté serveur avec accès au certificat privé
- Utiliser un proxy SSL qui déchiffre (Burp, mitmproxy)
```

---

## Conclusion

Wireshark est un outil extrêmement puissant mais qui nécessite de comprendre ses principes de fonctionnement. Dans cette section, nous avons posé les fondations :

- ✅ **Architecture** : Comment Wireshark capture et décode les paquets
- ✅ **Interface** : Les trois vues principales et leur rôle
- ✅ **Modes** : Capture live vs analyse offline
- ✅ **Bonnes pratiques** : Comment capturer efficacement et légalement
- ✅ **Concepts clés** : Streams, temps, colorisation

Vous êtes maintenant prêt à passer à l'analyse concrète. Dans les prochaines sections, nous allons :

1. **Section 7.4** : Apprendre à lire et interpréter les captures en détail
2. **Section 7.5** : Maîtriser les filtres pour isoler le trafic pertinent
3. **Section 7.6** : Identifier les problèmes réseau courants
4. **Section 7.7** : Analyser les performances réseau

L'apprentissage de Wireshark est progressif : plus vous analyserez de captures, plus vous développerez l'intuition pour identifier rapidement les anomalies.

**Conseil final :** Pratiquez ! Capturez votre propre trafic web, regardez ce qui se passe quand vous visitez un site, envoyez un email, téléchargez un fichier. Observer le trafic "normal" est la meilleure façon d'apprendre à reconnaître l'anormal.

---

**Prochaine section :** 7.4 Lecture et interprétation des captures

Nous plongerons dans l'analyse concrète : comment lire un paquet couche par couche, suivre une conversation TCP complète, identifier les handshakes, comprendre les retransmissions, et extraire des informations utiles pour le diagnostic.

⏭️ [Lecture et interprétation des captures](/07-analyse-depannage/04-lecture-captures.md)
