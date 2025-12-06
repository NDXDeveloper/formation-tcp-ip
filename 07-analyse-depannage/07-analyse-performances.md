🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.7 Analyse de performances réseau

## Introduction

Après avoir appris à identifier les problèmes réseau, nous allons maintenant nous concentrer sur l'**analyse de performances**. Même sans erreurs visibles (retransmissions, timeouts), un réseau peut avoir des performances médiocres. La question n'est plus "Ça marche ?" mais "**Ça marche bien ?**"

L'analyse de performances réseau permet de :
- Mesurer le **débit réel** vs le débit théorique
- Calculer la **latence** à chaque niveau
- Identifier les **goulots d'étranglement**
- Optimiser la **configuration TCP**
- Valider les **SLA** (Service Level Agreements)
- Comprendre l'**expérience utilisateur**

Cette section vous apprendra à utiliser Wireshark comme outil de mesure de performances précis, à interpréter les graphiques, et à identifier où se situe le problème : réseau, application, ou configuration.

---

## Les métriques de performance clés

### 1. Débit (Throughput / Bandwidth)

**Définition :**
- Quantité de données transférées par unité de temps
- Exprimé en bits/seconde (bps, Kbps, Mbps, Gbps)

**Formule de base :**
```
Débit = Données transférées / Temps écoulé

Exemple :
100 MB transférés en 10 secondes
= 100 × 8 Mbits / 10s
= 80 Mbps
```

**Types de débit :**

**Débit théorique (Bandwidth) :**
```
Capacité maximale du lien
Exemples :
- Fast Ethernet : 100 Mbps
- Gigabit Ethernet : 1000 Mbps (1 Gbps)
- Wi-Fi 802.11ac : 866 Mbps
```

**Débit utile (Goodput) :**
```
Données applicatives réelles (sans headers)

Exemple :
Débit total : 100 Mbps
Headers TCP/IP/Ethernet : ~5%
Goodput : ~95 Mbps
```

**Débit observé (Measured throughput) :**
```
Ce que Wireshark mesure réellement
Peut être limité par :
- Congestion réseau
- Window size TCP
- Latence
- Perte de paquets
```

### 2. Latence (Delay)

**Définition :**
- Temps pour qu'un paquet aille de A à B
- Exprimé en millisecondes (ms)

**Types de latence :**

**RTT (Round Trip Time) :**
```
Temps aller-retour complet
= Délai aller + Délai retour + Temps de traitement

Mesure Wireshark :
SYN envoyé à t=0.000000
SYN-ACK reçu à t=0.045678
RTT = 45.678 ms
```

**One-way delay :**
```
Temps dans une seule direction
≈ RTT / 2 (si chemin symétrique)
```

**Application latency :**
```
Temps de réponse total côté utilisateur
= Network latency + Server processing time

Exemple HTTP :
GET envoyé : t=0.000000
200 OK reçu : t=0.250000
Application latency = 250 ms
```

**Composantes de la latence :**
```
Latence totale = Propagation + Transmission + Queuing + Processing

Propagation : Vitesse de la lumière dans le média
Transmission : Temps pour mettre les bits sur le lien
Queuing : Attente dans les buffers
Processing : Traitement par les équipements
```

### 3. Jitter (Variation de latence)

**Définition :**
- Variation de la latence entre paquets
- Important pour VoIP, vidéo, jeux

**Mesure :**
```
Paquet 1 : RTT = 10 ms
Paquet 2 : RTT = 15 ms
Paquet 3 : RTT = 8 ms
Paquet 4 : RTT = 20 ms

Jitter = écart-type des RTT
       = √[((10-13.25)² + (15-13.25)² + (8-13.25)² + (20-13.25)²) / 4]
       ≈ 4.8 ms
```

**Seuils de qualité :**
```
< 10 ms  : Excellent (VoIP parfait)
10-30 ms : Bon
30-50 ms : Acceptable (légère dégradation)
> 50 ms  : Mauvais (voix hachée)
```

### 4. Perte de paquets (Packet Loss)

**Définition :**
- Pourcentage de paquets perdus
- Mesure de la fiabilité du lien

**Calcul :**
```
Perte = (Paquets perdus / Paquets envoyés) × 100%

Exemple :
1000 paquets envoyés
5 retransmissions TCP
Perte = 5/1000 = 0.5%
```

**Seuils :**
```
< 0.1%  : Excellent
0.1-1%  : Bon
1-2.5%  : Acceptable
> 2.5%  : Problématique
> 5%    : Inacceptable
```

### 5. Efficacité TCP

**Définition :**
- Ratio entre débit théorique et débit réel
- Mesure de l'utilisation du lien

**Formule :**
```
Efficacité = (Débit réel / Débit théorique) × 100%

Exemple :
Lien 100 Mbps
Débit mesuré : 75 Mbps
Efficacité = 75%
```

**Facteurs réduisant l'efficacité :**
```
- Window size trop petit
- RTT élevé (Bandwidth-Delay Product)
- Retransmissions
- Overhead protocoles
- Application lente
```

---

## Mesurer le débit avec Wireshark

### I/O Graph : Visualiser le débit

**Accès :**
```
Statistics → I/O Graph
```

**Interface :**
```
┌─────────────────────────────────────────────────────────────┐
│  I/O Graph                                              [X] │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Bits/s                                             │    │
│  │  10M ┤                                              │    │
│  │   8M ┤         ╭────────╮                           │    │
│  │   6M ┤        ╭╯        ╰╮                          │    │
│  │   4M ┤    ╭───╯          ╰───╮                      │    │
│  │   2M ┤────╯                  ╰────────              │    │
│  │   0  └────┬────┬────┬────┬────┬────┬────►           │    │
│  │          0s   5s  10s  15s  20s  25s  30s           │    │
│  └─────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────┤
│  Graph 1: [tcp             ] Style: [Line] Color: [Blue]    │
│  Y Axis: [Bits/s ▼] Interval: [1 sec ▼]                     │
│  [+ Add Graph] [- Remove]                                   │
└─────────────────────────────────────────────────────────────┘
```

### Configuration de l'I/O Graph

**Y Axis (Axe vertical) :**
```
- Packets : Nombre de paquets/seconde
- Bytes : Octets/seconde
- Bits : Bits/seconde (pour le débit)
- Advanced : Formules personnalisées
```

**Interval (Intervalle de temps) :**
```
- 1 ms : Très précis, pour courtes captures
- 10 ms : Bon compromis
- 100 ms : Standard
- 1 sec : Vue d'ensemble
- 10 sec : Longues captures
```

**Filtres multiples :**
```
Graph 1: tcp                      (Tout le TCP en bleu)
Graph 2: tcp.port == 443          (HTTPS en rouge)
Graph 3: tcp.analysis.retransmission (Retransmissions en orange)
```

### Exemple : Analyser un téléchargement

**Scénario :** Téléchargement d'un fichier de 100 MB

**Configuration I/O Graph :**
```
Y Axis: Bits
Interval: 1 sec
Filter: tcp.port == 80
```

**Graphique observé :**
```
Mbps
100 ┤
 90 ┤                    ╭────────────────────╮
 80 ┤                   ╭╯                    ╰╮
 70 ┤                  ╭╯                      ╰╮
 60 ┤                 ╭╯                        ╰╮
 50 ┤               ╭─╯                          ╰─╮
 40 ┤            ╭──╯                              ╰──╮
 30 ┤         ╭──╯                                    ╰──╮
 20 ┤      ╭──╯                                          ╰─
 10 ┤   ╭──╯
  0 └───┴─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────►
          0s   2s    4s    6s    8s   10s   12s   14s
          │    │     │                       │
          │    │     └─ Plateau à ~90 Mbps  │
          │    └─ Slow start (cwnd croissante)
          └─ 3-way handshake
```

**Interprétation :**
```
Phase 1 (0-2s) : Slow start TCP
- Débit augmente exponentiellement
- Congestion window (cwnd) double à chaque RTT

Phase 2 (2-12s) : Transfert stable
- Débit plateau à ~90 Mbps
- Bon débit sur lien 100 Mbps (90% d'efficacité)

Phase 3 (12-14s) : Fin de transfert
- Débit décroît (moins de données à envoyer)
- Derniers segments + ACKs
```

### Mesure moyenne du débit

**Méthode 1 : Statistics → Capture File Properties**
```
Capture File Properties

File:
  Name: download.pcap
  Length: 105.5 MB
  Format: Wireshark/tcpdump/... - pcapng
  Encapsulation: Ethernet

Time:
  First packet: 2025-12-07 14:30:00
  Last packet: 2025-12-07 14:30:15
  Elapsed: 15.234 seconds

Capture statistics:
  Packets: 75,432
  Between first and last packet: 15.234 s
  Average packets/sec: 4,950.8
  Average packet size: 1,399.2 bytes
  Bytes: 105,543,210
  Average bytes/sec: 6,926,789
  Average bits/sec: 55,414,312 (55.4 Mbps) ✅
```

**Méthode 2 : Filtrer et calculer manuellement**
```
1. Filtrer le trafic pertinent :
   tcp.stream eq 5 and tcp.flags.push == 1

2. Note du premier paquet :
   Frame 100, Time: 0.000000

3. Note du dernier paquet :
   Frame 5432, Time: 12.345678

4. Voir Bytes total (colonne Length) :
   Sum = 100,000,000 bytes

5. Calculer :
   Débit = (100,000,000 × 8) / 12.345678
        = 800,000,000 / 12.345678
        = 64,800,000 bps
        = 64.8 Mbps
```

---

## Mesurer la latence avec Wireshark

### RTT (Round Trip Time)

**Wireshark calcule automatiquement le RTT pour TCP :**

**Dans Packet Details :**
```
▼ Transmission Control Protocol
    [SEQ/ACK analysis]
        [iRTT: 0.045678 seconds] ✅
        [This is an ACK to the segment in frame: 142]
        [The RTT to ACK the segment was: 0.046234 seconds] ✅
```

**iRTT (initial RTT) :**
```
RTT mesuré pendant le 3-way handshake
= Temps entre SYN et SYN-ACK

Frame 1 (SYN) : t = 0.000000
Frame 2 (SYN-ACK) : t = 0.045678
iRTT = 45.678 ms

C'est le RTT "baseline" du réseau
```

**RTT courant :**
```
RTT mesuré pour chaque segment de données

Frame 100 (Data) : t = 5.000000
Frame 101 (ACK) : t = 5.046234
RTT = 46.234 ms

Variation par rapport à iRTT indique congestion/latence variable
```

### Graphique RTT

**TCP Stream Graphs → Round Trip Time**
```
Statistics → TCP Stream Graphs → Round Trip Time

┌─────────────────────────────────────────────────┐
│  RTT (ms)                                       │
│  100 ┤                                          │
│   90 ┤                                          │
│   80 ┤                                          │
│   70 ┤                                          │
│   60 ┤        ╭╮                    ╭╮          │
│   50 ┤      ╭─╯╰─╮                ╭─╯╰─╮        │
│   40 ┤──────╯    ╰────────────────╯    ╰───     │
│   30 ┤                                          │
│   20 ┤                                          │
│   10 ┤                                          │
│    0 └──┬────┬────┬────┬────┬────┬────┬────►    │
│        0s   2s   4s   6s   8s  10s  12s         │
└─────────────────────────────────────────────────┘
```

**Interprétation :**
```
RTT stable à ~40-50ms : Bon
Pics à 60-70ms : Congestion temporaire
Si RTT augmente continuellement : Problème de congestion croissante
```

### Time Delta (Delta de temps)

**Mesurer le délai entre paquets consécutifs :**

**Afficher colonne Time Delta :**
```
View → Time Display Format → Seconds Since Previous Displayed Packet
```

**Ou ajouter colonne :**
```
Click droit sur colonne → Column Preferences → Add
Title: Delta
Type: Delta time displayed
```

**Exemple :**
```
No.  Time      Delta     Source        Dest          Protocol  Info
100  5.000000  0.000123  192.168.1.10  93.184.216.34 HTTP      GET /
101  5.045678  0.045678  93.184.216.34 192.168.1.10  TCP       ACK
102  5.095234  0.049556  93.184.216.34 192.168.1.10  HTTP      200 OK
103  5.096789  0.001555  192.168.1.10  93.184.216.34 TCP       ACK
```

**Analyse :**
```
Delta #101 (45.7ms) : RTT réseau
Delta #102 (49.6ms) : Temps de traitement serveur + RTT
Delta #103 (1.6ms) : ACK immédiat (local)
```

### Application Response Time

**Mesurer le temps de réponse applicatif :**

**Pour HTTP :**
```
Wireshark calcule automatiquement http.time

▼ Hypertext Transfer Protocol
    [Time since request: 0.125678 seconds] ✅
    [Request in frame: 100]
    [Response in frame: 102]
```

**Filtre pour trouver les requêtes lentes :**
```
http.time > 1
# Requêtes HTTP prenant plus de 1 seconde
```

**Statistiques HTTP :**
```
Statistics → HTTP → Request Sequences

Tableau :
Request | Response | Time (s) | Server
--------|----------|----------|--------
GET /   | 200 OK   | 0.125    | nginx
GET /api| 200 OK   | 0.856    | apache
POST /  | 500 ERR  | 2.345    | nginx ⚠️
```

**Décomposition du temps de réponse :**
```
Temps total (http.time) = RTT + Server processing + RTT

Exemple :
http.time = 125 ms
iRTT = 45 ms

Server processing = 125 - (2 × 45/2) = 125 - 45 = 80 ms
                                              ↑
                              (approximation : 1 RTT pour requête+réponse)
```

---

## Bandwidth-Delay Product (BDP)

### Concept

**Définition :**
```
BDP = Bandwidth × RTT
    = Quantité de données "en vol" sur le réseau

C'est la taille de "tuyau" réseau
```

**Exemple :**
```
Bandwidth : 100 Mbps = 12,500,000 bytes/sec
RTT : 50 ms = 0.05 sec

BDP = 12,500,000 × 0.05
    = 625,000 bytes
    = 610 KB
```

**Signification :**
```
Pour utiliser pleinement un lien 100 Mbps avec RTT de 50ms,
la TCP window size doit être ≥ 610 KB
```

### TCP Window Size optimal

**Relation avec le débit :**
```
Débit max = Window Size / RTT

Exemple 1 :
Window = 64 KB = 65,536 bytes
RTT = 50 ms = 0.05 sec
Débit max = 65,536 / 0.05 = 1,310,720 bytes/sec = 10.5 Mbps

Exemple 2 :
Window = 1 MB = 1,048,576 bytes
RTT = 50 ms = 0.05 sec
Débit max = 1,048,576 / 0.05 = 20,971,520 bytes/sec = 168 Mbps
```

**Window Size dans Wireshark :**
```
▼ Transmission Control Protocol
    Window: 65535
    [Calculated window size: 8388480] ✅
    [Window size scaling factor: 128]
```

**Window Scaling :**
```
Window annoncée : 65535 (16 bits max)
Scaling factor : 128 (négocié au handshake)
Window réelle : 65535 × 128 = 8,388,480 bytes = 8 MB ✅
```

### Graphique Window Size

**TCP Stream Graphs → Window Scaling**
```
Statistics → TCP Stream Graphs → Window Scaling

┌────────────────────────────────────────────────┐
│  Window (bytes)                                │
│  10M ┤                                         │
│   9M ┤                                         │
│   8M ┤────────────────────────────────         │
│   7M ┤                                         │
│   6M ┤                                         │
│   5M ┤                                         │
│   4M ┤                                         │
│   3M ┤                                         │
│   2M ┤                                         │
│   1M ┤╭───                                     │
│   0  └┴───┬────┬────┬────┬────┬────┬────►      │
│          0s   2s   4s   6s   8s  10s           │
└────────────────────────────────────────────────┘
```

**Interprétation :**
```
Window grandit rapidement au début (slow start)
Puis stabilise à ~8 MB (bon)

Si window reste < BDP → Limite le débit
Si window = 0 → Problème (voir section 7.6)
```

---

## Throughput vs Window Size

### Graphique Time-Sequence (Stevens)

**Le graphique le plus puissant pour analyser TCP :**

**Accès :**
```
Statistics → TCP Stream Graphs → Time-Sequence (Stevens)
```

**Graphique :**
```
┌────────────────────────────────────────────────────────┐
│  Sequence Number (bytes)                               │
│  10M ┤                                          ╱      │
│   9M ┤                                      ╱───       │
│   8M ┤                                  ╱───           │
│   7M ┤                              ╱───               │
│   6M ┤                          ╱───                   │
│   5M ┤                      ╱───                       │
│   4M ┤                  ╱───                           │
│   3M ┤              ╱───                               │
│   2M ┤          ╱───                                   │
│   1M ┤      ╱───                                       │
│   0  └──────┴────┬────┬────┬────┬────┬────┬────►       │
│              0s   2s   4s   6s   8s  10s  12s          │
│                                                        │
│  Pente = Débit                                         │
│  Segment horizontal = Pause (window full, congestion)  │
│  Segment vertical = Retransmission                     │
└────────────────────────────────────────────────────────┘
```

**Lecture du graphique :**

**Pente régulière :**
```
╱───
    ╱───
        ╱───

Transfert fluide, bon débit
Pente = Débit (bytes/sec)
```

**Paliers horizontaux :**
```
╱───────
        ╱───────

Pauses dans le transfert
Causes possibles :
- Window full (récepteur lent)
- Congestion avoidance
- Application lente à générer données
```

**Sauts verticaux :**
```
    │
╱───┘
    ╱───

Retransmission (numéro de séquence recule)
Perte de paquet détectée
```

**Exemple réel : Transfert avec retransmissions**
```
Seq
10M ┤                              ╱────
 9M ┤                          ╱───│
 8M ┤                      ╱───    │ Retrans
 7M ┤                  ╱───        │ (vertical)
 6M ┤              ╱───            └──╱──
 5M ┤          ╱───                  ╱───
 4M ┤      ╱───                  ╱───
 3M ┤  ╱───                  ╱───
 2M ┤──                  ╱───
 1M ╱                ╱───
 0  └───┬────┬────┬────┬────┬────┬────►
       0s   2s   4s   6s   8s  10s

Observations :
- Débit initial bon (pente raide)
- Retransmission à t=6s (segment vertical)
- Reprise du transfert après retransmission
- Perte de ~2s due à la retransmission
```

---

## Analyser les goulots d'étranglement

### Identifier le facteur limitant

**Question clé :** Qu'est-ce qui limite le débit ?

**Possibilités :**
```
1. Bande passante réseau saturée
2. Window size TCP insuffisante
3. RTT trop élevé (BDP)
4. Application émettrice lente
5. Application réceptrice lente
6. Retransmissions fréquentes
```

### Test 1 : Bande passante réseau

**Méthode :**
```
1. I/O Graph du débit
2. Comparer avec capacité du lien

Exemple :
Lien : 100 Mbps
Débit mesuré : 98 Mbps
→ Lien saturé ✅

Lien : 1 Gbps
Débit mesuré : 100 Mbps
→ Pas saturé, autre facteur limite
```

**Filtre I/O Graph :**
```
tcp
Y Axis: Bits
Interval: 1 sec

Si plateau proche de capacité lien → Saturé
Si bien en-dessous → Autre limite
```

### Test 2 : Window Size limitation

**Formule théorique :**
```
Débit max = Window / RTT

Exemple :
Window : 256 KB
RTT : 100 ms
Débit max = (256 × 1024 × 8) / 0.1 = 20,971,520 bps = 21 Mbps

Si débit observé ≈ 21 Mbps
→ Limité par window size ✅
```

**Vérification dans Wireshark :**
```
1. Mesurer RTT moyen (Statistics → TCP Stream Graphs → RTT)
2. Mesurer Window Size (TCP Stream Graphs → Window Scaling)
3. Calculer débit théorique max
4. Comparer avec débit réel (I/O Graph)
```

**Exemple concret :**
```
RTT moyen : 50 ms
Window : 128 KB
Débit théorique : (128 × 1024 × 8) / 0.05 = 20.97 Mbps
Débit mesuré : 21 Mbps

Conclusion : Limité par window size
Solution : Augmenter window (TCP Window Scaling)
```

### Test 3 : Application-bound

**Symptôme :**
```
Débit irrégulier avec pauses fréquentes
Window non pleine
Pas de retransmissions
```

**Graphique Time-Sequence :**
```
Seq
    ╱─────╱─────╱─────╱─────

Paliers réguliers = Application génère données par rafales
```

**Vérification :**
```
Statistics → TCP Stream Graphs → Throughput

Si débit fluctue beaucoup :
→ Application génère/consomme données de manière irrégulière

Corrélation avec logs applicatifs pour confirmer
```

### Test 4 : Retransmissions excessives

**Impact sur le débit :**
```
Chaque retransmission :
- Consomme de la bande passante (données envoyées 2×)
- Réduit cwnd (congestion window)
- Ralentit le transfert

Exemple :
Débit sans perte : 100 Mbps
Taux de perte : 2%
Débit réel : ~70-80 Mbps (selon RTT)
```

**Mesure :**
```
Filtre : tcp.analysis.retransmission

Count : 245 paquets retransmis
Total TCP packets : 10,000
Taux : 2.45%

Si > 1% → Impact significatif sur débit
```

---

## Optimisation des performances TCP

### TCP Window Scaling

**Problème :**
```
Window size limitée à 65,535 bytes (16 bits)
Sur liens à haut débit et latence élevée → Insuffisant

Exemple :
Lien : 1 Gbps
RTT : 100 ms
BDP = 1 Gbps × 0.1s = 100 Mb = 12.5 MB
Mais window max sans scaling = 64 KB

Débit réel = 64 KB / 0.1s = 5.12 Mbps (0.5% du lien !)
```

**Solution : Window Scaling (RFC 7323)**
```
Négocié au handshake (SYN, SYN-ACK)
Scale factor : 0 à 14 (multiply by 2^scale)

Exemple :
Scale factor : 7 (2^7 = 128)
Window annoncée : 65535
Window réelle : 65535 × 128 = 8,388,480 bytes = 8 MB ✅
```

**Voir dans Wireshark :**
```
Frame 1 (SYN) :
▼ TCP Options
    ▼ Window Scale: 7 (multiply by 128)
        Kind: Window Scale (3)
        Length: 3
        Shift count: 7
        [Multiplier: 128]
```

**Vérifier que scaling est utilisé :**
```
Frame suivants :
Window: 65535
[Calculated window size: 8388480] ✅ (65535 × 128)

Si [Calculated window size] absent → Scaling non activé
```

**Activer Window Scaling :**
```
# Linux
sysctl -w net.ipv4.tcp_window_scaling=1

# Vérifier
sysctl net.ipv4.tcp_window_scaling

# Augmenter buffer max
sysctl -w net.ipv4.tcp_rmem="4096 131072 67108864"  # 64 MB max
sysctl -w net.ipv4.tcp_wmem="4096 65536 67108864"
```

### TCP Timestamps

**Utilité :**
```
1. Calcul RTT plus précis (RTTM - Round Trip Time Measurement)
2. Protection contre wrapped sequence numbers (PAWS)
```

**Dans Wireshark :**
```
▼ TCP Options
    ▼ Timestamps: TSval 123456789, TSecr 987654321
        Kind: Time Stamp Option (8)
        Length: 10
        Timestamp value: 123456789
        Timestamp echo reply: 987654321
```

**Wireshark utilise les timestamps pour calculer RTT :**
```
[SEQ/ACK analysis]
    [The RTT to ACK the segment was: 0.045678 seconds]

Calculé via timestamps, plus précis que juste time delta
```

### Selective Acknowledgment (SACK)

**Problème sans SACK :**
```
Paquets reçus : 1000, 2000, 3000, 5000, 6000
Paquet perdu : 4000

Sans SACK :
ACK 4000 (j'attends 4000)
→ Émetteur retransmet 4000, 5000, 6000 (inutile pour 5000 et 6000)

Avec SACK :
ACK 4000, SACK 5000-7000
→ Émetteur retransmet SEULEMENT 4000 ✅
```

**Voir dans Wireshark :**
```
▼ TCP Options
    ▼ TCP SACK Permitted Option
        Kind: SACK Permitted (4)
        Length: 2

Dans ACK après perte :
▼ TCP Options
    ▼ TCP SACK Option
        Kind: SACK (5)
        Length: 10
        Left edge: 5000
        Right edge: 7000
```

**Avantage :**
```
Retransmissions plus efficaces
Récupération plus rapide après perte
Débit maintenu plus élevé
```

### Réglages système optimaux

**Linux (pour hautes performances) :**
```bash
# Buffer sizes (augmenter pour BDP élevé)
sysctl -w net.ipv4.tcp_rmem="4096 131072 67108864"  # 64 MB
sysctl -w net.ipv4.tcp_wmem="4096 65536 67108864"   # 64 MB
sysctl -w net.core.rmem_max=134217728                # 128 MB
sysctl -w net.core.wmem_max=134217728

# Window Scaling (obligatoire pour >64KB)
sysctl -w net.ipv4.tcp_window_scaling=1

# SACK (améliore récupération après perte)
sysctl -w net.ipv4.tcp_sack=1

# Timestamps (meilleur calcul RTT)
sysctl -w net.ipv4.tcp_timestamps=1

# Congestion control algorithm
sysctl -w net.ipv4.tcp_congestion_control=bbr  # BBR > Cubic

# Queue discipline (pour réduire bufferbloat)
tc qdisc replace dev eth0 root fq
```

**Valider les changements dans Wireshark :**
```
Capturer nouveau handshake :
- Vérifier Window Scale option présente
- Vérifier SACK permitted
- Vérifier Timestamps
- Vérifier [Calculated window size] élevée
```

---

## Cas d'étude : Diagnostic complet

### Scénario

**Problème rapporté :**
```
"Le téléchargement de fichiers depuis notre serveur est très lent"
"Devrait être 100 Mbps mais on mesure 8-10 Mbps"
```

### Étape 1 : Capture

```
Client : 192.168.1.50
Serveur : 203.0.113.100 (notre serveur HTTP)
Lien : 100 Mbps

Capture côté client pendant téléchargement
```

### Étape 2 : Mesure du débit

**I/O Graph :**
```
Filter: tcp.port == 80
Y Axis: Bits
Interval: 1 sec

Résultat :
Débit stable à ~8.5 Mbps ✅ (Confirme le problème)
```

### Étape 3 : Mesure RTT

**Statistics → TCP Stream Graphs → Round Trip Time**
```
RTT moyen : 125 ms ✅
RTT stable (pas de variation importante)
```

### Étape 4 : Analyse Window Size

**Statistics → TCP Stream Graphs → Window Scaling**
```
Window Size :
Reste stable à 65,535 bytes ✅ (Suspect !)
Pas de scaling visible
```

### Étape 5 : Calcul théorique

```
Débit max = Window / RTT
         = 65,535 bytes / 0.125 sec
         = 524,280 bytes/sec
         = 4.19 Mbps

Mais on mesure 8.5 Mbps ?
→ Regarder de plus près...
```

### Étape 6 : Inspection du handshake

**Frame 1 (SYN client) :**
```
▼ TCP Options
    ▼ Window Scale: 8 (multiply by 256) ✅ Client supporte
        Shift count: 8
        [Multiplier: 256]
```

**Frame 2 (SYN-ACK serveur) :**
```
▼ TCP Options
    [Window Scale option NOT present] ❌ Serveur ne supporte pas !
```

**Analyse :**
```
Client veut Window Scaling
Serveur ne répond pas avec Window Scale
→ Window Scaling DÉSACTIVÉ pour cette connexion
→ Window limitée à 65,535 bytes

Mais pourquoi 8.5 Mbps et pas 4.19 Mbps ?
```

### Étape 7 : Analyse détaillée

**Time-Sequence Graph :**
```
Observation : Pente par paliers, pas continue

Explication :
Client utilise multiple connexions TCP en parallèle
→ 2 connexions simultanées
→ Chacune ≈ 4 Mbps
→ Total ≈ 8 Mbps ✅
```

### Étape 8 : Cause racine

```
Serveur : Window Scaling désactivé
OS : Vieux Linux kernel (2.6.x)
Config : net.ipv4.tcp_window_scaling=0

Impact :
Window max = 64 KB
RTT = 125 ms
Débit max par connexion = 4 Mbps
Loin des 100 Mbps possibles
```

### Étape 9 : Solution

```
1. Activer Window Scaling sur serveur :
   sysctl -w net.ipv4.tcp_window_scaling=1

2. Augmenter buffers :
   sysctl -w net.ipv4.tcp_rmem="4096 131072 16777216"
   sysctl -w net.ipv4.tcp_wmem="4096 65536 16777216"

3. Rendre permanent :
   echo "net.ipv4.tcp_window_scaling=1" >> /etc/sysctl.conf
```

### Étape 10 : Validation

**Nouvelle capture après changement :**

**Handshake :**
```
SYN-ACK serveur contient maintenant :
▼ Window Scale: 7 (multiply by 128) ✅
```

**I/O Graph :**
```
Débit : ~95 Mbps ✅ (amélioration de 8.5 → 95 Mbps !)
```

**Window Size :**
```
[Calculated window size: 8388480] (8 MB) ✅
```

**Débit théorique :**
```
Window = 8 MB
RTT = 125 ms
Débit max = (8 × 1024 × 1024 × 8) / 0.125
         = 536,870,912 bps
         = 537 Mbps (largement au-dessus de 100 Mbps du lien)

→ Window n'est plus le facteur limitant
→ Lien utilisé à 95% ✅
```

---

## Métriques de performance : Résumé

### Tableau de référence

| Métrique | Excellent | Bon | Acceptable | Problématique |
|----------|-----------|-----|------------|---------------|
| **RTT (latence)** | < 10 ms | 10-50 ms | 50-150 ms | > 150 ms |
| **Jitter** | < 5 ms | 5-20 ms | 20-50 ms | > 50 ms |
| **Perte de paquets** | < 0.1% | 0.1-0.5% | 0.5-2% | > 2% |
| **Débit (vs capacité)** | > 90% | 70-90% | 50-70% | < 50% |
| **HTTP response time** | < 100 ms | 100-500 ms | 0.5-2s | > 2s |
| **DNS resolution** | < 20 ms | 20-100 ms | 100-500 ms | > 500 ms |

### Formules essentielles

```
Débit max TCP = Window Size / RTT

BDP (Bandwidth-Delay Product) = Bandwidth × RTT

Efficacité = (Débit réel / Débit théorique) × 100%

Taux de perte = (Paquets perdus / Paquets total) × 100%

Window Size optimale ≥ BDP
```

---

## Outils Wireshark pour la performance

### Récapitulatif des graphiques

**I/O Graph :**
```
Usage : Visualiser débit dans le temps
Accès : Statistics → I/O Graph
Utilité : Identifier pics, creux, patterns
```

**TCP Stream Graphs :**
```
1. Time-Sequence (Stevens)
   → Graphique le plus complet
   → Voir débit, retransmissions, pauses

2. Throughput
   → Débit instantané

3. Round Trip Time
   → Évolution RTT

4. Window Scaling
   → Évolution window size
```

**Statistics → Capture File Properties :**
```
Vue d'ensemble :
- Débit moyen global
- Durée totale
- Nombre de paquets
```

**Expert Information :**
```
Analyze → Expert Information

Performance warnings :
- Zero Windows
- Retransmissions
- Duplicate ACKs
```

---

## Checklist d'analyse de performance

### Méthodologie

```
1. MESURER le débit réel
   → I/O Graph, bytes/sec

2. MESURER la latence
   → iRTT, RTT moyen

3. CALCULER le BDP
   → Bandwidth × RTT

4. VÉRIFIER Window Size
   → ≥ BDP ?
   → Window Scaling actif ?

5. COMPTER les retransmissions
   → Taux < 1% ?

6. ANALYSER Time-Sequence Graph
   → Paliers ? Sauts verticaux ?

7. IDENTIFIER le goulot
   → Réseau, window, application ?

8. OPTIMISER
   → Selon le facteur limitant

9. VALIDER
   → Re-mesurer après changements
```

### Questions clés

```
✓ Le débit utilise-t-il la capacité du lien ?
✓ La latence est-elle stable ?
✓ Y a-t-il des retransmissions ?
✓ Le Window Scaling est-il activé ?
✓ La window size est-elle suffisante pour le BDP ?
✓ L'application est-elle le facteur limitant ?
✓ Les options TCP (SACK, Timestamps) sont-elles activées ?
```

---

## Points clés à retenir

### 🎯 Métriques essentielles

**Débit :**
```
Mesure la vitesse de transfert
Limité par : bande passante, window size, RTT, pertes
I/O Graph pour visualiser
```

**Latence (RTT) :**
```
Temps aller-retour
Impact le BDP et débit max
iRTT = baseline réseau
```

**Window Size :**
```
Doit être ≥ BDP pour performances optimales
Window Scaling essentiel pour hauts débits
Graph Window Scaling pour suivre l'évolution
```

**Retransmissions :**
```
Réduisent le débit effectif
Taux > 1% = Problème
Impact exponentiel avec RTT élevé
```

### 🎯 Formules clés

```
Débit max = Window / RTT
BDP = Bandwidth × RTT
Window optimale ≥ BDP
```

### 🎯 Graphiques Wireshark

```
Time-Sequence (Stevens) : LE graphique pour TCP
I/O Graph : Vue d'ensemble du débit
RTT Graph : Latence dans le temps
Window Scaling : Vérifier capacité de réception
```

### 🎯 Optimisations

```
✓ Activer Window Scaling
✓ Augmenter buffers TCP (sysctl)
✓ Activer SACK
✓ Utiliser congestion control moderne (BBR)
✓ Réduire RTT si possible (CDN, serveurs proches)
```

---

## Conclusion

L'analyse de performances réseau avec Wireshark permet de :
- **Mesurer** précisément le débit, la latence, les pertes
- **Visualiser** les patterns de trafic avec les graphiques
- **Calculer** les limites théoriques (BDP, window max)
- **Identifier** le facteur limitant (réseau, config TCP, application)
- **Optimiser** en agissant sur la bonne couche
- **Valider** que les changements ont l'effet escompté

Les performances réseau sont souvent limitées par la configuration TCP (window size, scaling) plutôt que par la bande passante pure. Une bonne compréhension du BDP et du rôle de la window size permet de diagnostiquer 80% des problèmes de performances.

Dans la dernière section de ce module, nous verrons comment utiliser les logs et le monitoring pour compléter l'analyse Wireshark et mettre en place une surveillance proactive.

---

**Prochaine section :** 7.8 Logs et monitoring

Nous explorerons comment combiner Wireshark avec les logs système, les outils de monitoring, et mettre en place une surveillance proactive pour détecter les problèmes avant qu'ils n'impactent les utilisateurs.

⏭️ [Logs et monitoring](/07-analyse-depannage/08-logs-monitoring.md)
