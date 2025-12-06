🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.6 Identification des problèmes courants

## Introduction

Avec les connaissances acquises sur la lecture de captures et les filtres, nous sommes maintenant prêts à diagnostiquer les problèmes réseau réels. Wireshark excelle dans l'identification de patterns problématiques que les outils classiques (`ping`, `traceroute`) ne peuvent pas détecter.

Cette section présente les problèmes les plus fréquents rencontrés en production, comment les reconnaître dans Wireshark, leurs causes probables, et comment les résoudre. Pour chaque problème, nous verrons :

- **Le symptôme** : Ce que l'utilisateur rapporte
- **La signature Wireshark** : Comment ça apparaît dans une capture
- **Les causes** : Pourquoi ça se produit
- **Le diagnostic** : Comment confirmer
- **La résolution** : Solutions typiques

---

## 1. Retransmissions TCP

### Symptôme utilisateur

```
"Le site web est très lent"
"Les téléchargements prennent une éternité"
"L'application se fige parfois"
```

### Signature dans Wireshark

**Dans la colonne Info :**
```
No.  Time      Source        Destination   Protocol  Info
245  10.5000   192.168.1.10  93.184.216.34 TCP       [TCP Retransmission] ...
246  10.5100   192.168.1.10  93.184.216.34 TCP       [TCP Retransmission] ...
```

**Texte en noir** avec `[TCP Retransmission]` ou `[TCP Fast Retransmission]`

**Dans Packet Details :**
```
▼ Transmission Control Protocol
    [SEQ/ACK analysis]
        [This is a TCP retransmission of segment in frame: 242]
        [The RTO for this segment was: 0.200000 seconds]
        [Expert Info (Note/Sequence): This is a TCP retransmission]
            [This is a TCP retransmission]
            [Severity level: Note]
            [Group: Sequence]
```

### Types de retransmissions

#### 1. TCP Retransmission (classique)

**Cause :** Le paquet ou son ACK a été perdu.

**Timeline :**
```
Client                          Serveur
  |                                |
  |--- Seq=1000, Len=500 --------> |  #100, t=1.000
  |                                |
  |                                |  [Paquet perdu]
  |                                |
  [Timeout après ~200ms]           |
  |                                |
  |--- Seq=1000, Len=500 --------> |  #101, t=1.250 [TCP Retransmission]
  |                                |
  |<-- ACK 1500 -------------------|  #102, t=1.300
```

**Filtre :**
```
tcp.analysis.retransmission
```

#### 2. TCP Fast Retransmission

**Cause :** Détection rapide via duplicate ACKs (après 3 ACK identiques).

**Timeline :**
```
Client                          Serveur
  |                                |
  |--- Seq=1000, Len=500 --------> |  Paquet perdu
  |--- Seq=1500, Len=500 --------> |  Reçu, mais hors séquence
  |--- Seq=2000, Len=500 --------> |  Reçu, mais hors séquence
  |                                |
  |<-- ACK 1000 [Dup ACK] ---------|  "Je n'ai toujours pas 1000"
  |<-- ACK 1000 [Dup ACK] ---------|
  |<-- ACK 1000 [Dup ACK] ---------|  3ème Dup ACK
  |                                |
  |--- Seq=1000, Len=500 --------> |  [TCP Fast Retransmission]
  |                                |  (avant le timeout)
  |<-- ACK 2500 -------------------|  Tout rattrapé
```

**Filtre :**
```
tcp.analysis.fast_retransmission
```

#### 3. Spurious Retransmission

**Cause :** Retransmission inutile (ACK était en transit ou réseau réordonne les paquets).

```
[Expert Info (Note/Sequence): This is a spurious retransmission]
```

### Quantifier le problème

**Compter les retransmissions :**
```
1. Appliquer le filtre : tcp.analysis.retransmission
2. Regarder le nombre de paquets affichés
3. Calculer le taux :
   Taux = (Retransmissions / Total paquets TCP) × 100
```

**Seuils d'alerte :**
```
< 0.1%  : Excellent (normal sur Internet)
0.1-1%  : Acceptable
1-5%    : Problématique (investigation nécessaire)
> 5%    : Grave (problème majeur)
> 10%   : Critique (réseau très dégradé)
```

**Exemple avec Statistics :**
```
Statistics → TCP Stream Graphs → Round Trip Time

Observe les RTT qui augmentent soudainement
→ Indication de retransmissions
```

### Causes possibles

**1. Perte de paquets réseau**
```
Causes :
- Lien saturé (congestion)
- Wi-Fi instable
- Câble défectueux
- Équipement réseau défaillant

Comment confirmer :
- Vérifier sur quel chemin les pertes se produisent
- Utiliser mtr ou traceroute pour identifier l'équipement
- Vérifier les statistiques d'erreur de l'interface
```

**2. Congestion réseau**
```
Symptômes associés :
- Latence variable (jitter)
- Window size qui se réduit
- Slow start répétitif

Filtre pour confirmer :
tcp.analysis.retransmission and tcp.window_size < 32768
```

**3. Firewall qui drop des paquets**
```
Pattern typique :
- Retransmissions après un certain pattern
- Paquets de taille spécifique perdus
- Ports/protocoles spécifiques affectés

Test :
- Capturer des deux côtés du firewall
- Comparer ce qui entre vs ce qui sort
```

**4. MTU trop grand (fragmentation bloquée)**
```
Symptôme :
- Petits paquets passent
- Gros paquets (>1400 bytes) sont retransmis

Filtre :
tcp.analysis.retransmission and frame.len > 1400
```

### Résolution

**Actions immédiates :**
```
1. Identifier où se produisent les pertes (client, serveur, intermédiaire)
2. Vérifier la bande passante disponible
3. Tester avec un MTU plus petit
4. Vérifier les câbles/équipements physiques
```

**Solutions à long terme :**
```
- Augmenter la bande passante
- Optimiser les routes réseau
- Configurer QoS pour prioriser le trafic critique
- Remplacer équipement défectueux
- Activer/optimiser TCP Window Scaling
```

---

## 2. TCP Duplicate ACK

### Symptôme utilisateur

```
"Le transfert est lent malgré une bonne connexion"
"Le débit n'utilise pas toute la bande passante disponible"
```

### Signature dans Wireshark

```
No.  Time      Source        Destination   Protocol  Info
250  5.1000    93.184.216.34 192.168.1.10  TCP       [TCP Dup ACK 245#1] Ack=1000
251  5.1001    93.184.216.34 192.168.1.10  TCP       [TCP Dup ACK 245#2] Ack=1000
252  5.1002    93.184.216.34 192.168.1.10  TCP       [TCP Dup ACK 245#3] Ack=1000
253  5.1050    192.168.1.10  93.184.216.34 TCP       [TCP Fast Retransmission] Seq=1000
```

**Dans Packet Details :**
```
[SEQ/ACK analysis]
    [This is a TCP duplicate ack]
    [Duplicate to the ACK in frame: 245]
    [Expert Info (Note/Sequence): Duplicate ACK (#1)]
```

### Ce qui se passe

```
1. Le récepteur reçoit un paquet hors séquence
2. Il renvoie un ACK pour le dernier paquet reçu dans l'ordre
3. Chaque nouveau paquet hors séquence → nouveau Dup ACK
4. Après 3 Dup ACK → Fast Retransmit déclenché
```

**Illustration :**
```
Émetteur envoie : 1000, 1500, 2000, 2500, 3000
Réseau perd : 1500
Récepteur reçoit : 1000, 2000, 2500, 3000

Récepteur ACK :
- Pour 1000 : ACK 1500 (OK)
- Pour 2000 : ACK 1500 (j'attends toujours 1500) [Dup ACK #1]
- Pour 2500 : ACK 1500 (toujours pas 1500)    [Dup ACK #2]
- Pour 3000 : ACK 1500 (où est 1500 ??)       [Dup ACK #3]

Émetteur après 3 Dup ACK :
- Retransmet 1500 [Fast Retransmit]
```

### Filtre

```
tcp.analysis.duplicate_ack
```

### Causes

**1. Perte de paquets (comme retransmissions)**
```
Les Dup ACK sont souvent le SYMPTÔME
La perte est la CAUSE
```

**2. Réordonnancement de paquets**
```
Paquets arrivent dans le désordre (routes différentes, load balancing)

Pattern :
- Dup ACK suivis d'ACK normaux (sans retransmission)
- Pas de perte réelle, juste du réordonnancement

Filtre pour vérifier :
tcp.analysis.duplicate_ack and not tcp.analysis.retransmission
```

**3. Latence variable sur le chemin**
```
Paquets prennent des chemins avec latences différentes
→ Arrivent dans le désordre
```

### Impact sur les performances

```
Chaque série de Dup ACK → Fast Retransmit
→ Congestion window réduite de moitié
→ Débit divisé par 2 temporairement
→ Temps pour remonter au débit max

Exemple :
Débit initial : 10 Mbps
Dup ACK → cwnd divisée par 2
Nouveau débit : 5 Mbps
Slow start pour retrouver 10 Mbps : ~2 secondes

Si Dup ACK fréquents → débit constamment limité
```

### Diagnostic

**1. Compter les occurrences**
```
Filtre : tcp.analysis.duplicate_ack
Nombre : Si > 1% des paquets → Problème
```

**2. Vérifier si suivis de retransmissions**
```
tcp.analysis.duplicate_ack

Regarder les paquets suivants :
- Si Fast Retransmit → Perte confirmée
- Si ACK normal → Peut-être juste réordonnancement
```

**3. Analyser le pattern temporel**
```
I/O Graph :
- Filtre 1 : tcp.analysis.duplicate_ack (en rouge)
- Filtre 2 : tcp.analysis.retransmission (en orange)

Si pics simultanés → Confirme la perte
```

### Résolution

```
Même approche que pour les retransmissions :
1. Identifier la cause de la perte/réordonnancement
2. Optimiser le routage (éviter load balancing agressif)
3. Stabiliser la latence
4. Vérifier QoS et priorisation
```

---

## 3. TCP Zero Window

### Symptôme utilisateur

```
"Le téléchargement démarre vite puis se bloque complètement"
"Le serveur semble ne plus envoyer de données"
"Connexion établie mais pas de transfert"
```

### Signature dans Wireshark

```
No.  Time      Source        Destination   Protocol  Info
340  15.2000   192.168.1.10  93.184.216.34 TCP       [TCP Window Full] ...
341  15.2001   93.184.216.34 192.168.1.10  TCP       [TCP ZeroWindow] Window=0
```

**Dans Packet Details :**
```
▼ Transmission Control Protocol
    Window: 0
    [Calculated window size: 0]
    [SEQ/ACK analysis]
        [Expert Info (Warning/Sequence): TCP window specified by the receiver is now zero]
            [TCP window specified by the receiver is now zero]
            [Severity level: Warning]
            [Group: Sequence]
```

**Suivi par Window Update :**
```
342  15.5000   192.168.1.10  93.184.216.34 TCP       [TCP Window Update] Window=32768
```

### Ce qui se passe

```
Timeline :

Client reçoit des données                    Serveur envoie des données
  |                                              |
  |<-- Seq=1000, Len=1460, Win=65535 ----------  |  Buffer client : 0/65535
  |--- ACK 2460, Win=64075 --------------------> |  Buffer client : 1460/65535
  |                                              |
  |<-- Seq=2460, Len=1460, Win=65535 ----------  |  Buffer client : 2920/65535
  [Application lente à lire]                     |
  |<-- Seq=3920, Len=1460, Win=65535 ----------  |  Buffer client : 4380/65535
  |<-- ... (beaucoup de données) -------------   |
  |                                              |
  [Buffer client plein !]                        |
  |--- ACK 65535, Win=0 -----------------------> |  Buffer client : 65535/65535 (PLEIN)
  |                                              |
  [Serveur DOIT arrêter d'envoyer]               |  ⚠️ Stop transmission
  |                                              |
  [Application lit enfin les données]            |
  [Buffer se vide partiellement]                 |
  |                                              |
  |--- ACK 65535, Win=32768 [Window Update] ---> |  Buffer client : 32768/65535
  |                                              |
  [Serveur peut reprendre]                       |
  |<-- Seq=65535, Len=1460 --------------------  |  Reprise
```

### Filtre

```
tcp.window_size == 0                # Fenêtre à zéro
tcp.analysis.zero_window            # Expert Info
tcp.analysis.window_update          # Mise à jour fenêtre
tcp.analysis.window_full            # Fenêtre pleine (côté émetteur)
```

### Causes

**1. Application réceptrice lente**
```
Scénarios :
- CPU surchargé (traitement lent)
- Disque lent (écriture en buffer)
- Application mal optimisée
- Garbage collection (Java, .NET)

Comment confirmer :
- Surveiller CPU/RAM de la machine réceptrice
- Profiler l'application
- Vérifier logs applicatifs
```

**2. Configuration TCP inadéquate**
```
Problème :
- Receive buffer trop petit
- Application ne lit pas assez vite le socket

Vérification :
# Linux : Voir les buffers TCP
sysctl net.ipv4.tcp_rmem
# net.ipv4.tcp_rmem = 4096 131072 6291456
#                      min  default  max

# Augmenter si nécessaire
sysctl -w net.ipv4.tcp_rmem="4096 262144 12582912"
```

**3. Ressources système limitées**
```
Causes :
- RAM insuffisante (swap)
- CPU à 100%
- I/O disque saturé

Diagnostic :
top, htop, iostat sur la machine concernée
```

### Impact

```
Conséquences :
- Débit réduit à zéro temporairement
- Latence applicative augmentée
- Timeout possible si window=0 prolongé
- Mauvaise expérience utilisateur

Durée critique :
< 1 seconde : Acceptable (rare)
1-5 secondes : Problématique
> 5 secondes : Grave (risque timeout)
```

### Diagnostic détaillé

**1. Identifier le récepteur problématique**
```
Filtre : tcp.window_size == 0

Regarder la Source du paquet :
→ C'est le RÉCEPTEUR qui annonce Win=0
→ C'est LUI qui a le problème
```

**2. Mesurer la durée**
```
Entre [TCP ZeroWindow] et [TCP Window Update]

Méthode :
1. Trouver le paquet Zero Window (frame X)
2. Trouver le Window Update suivant (frame Y)
3. Calculer : Time(Y) - Time(X)
```

**3. Corréler avec les données applicatives**
```
Vérifier :
- Logs de l'application réceptrice
- Métriques système (CPU, RAM, I/O)
- Garbage collection logs (si Java/.NET)
```

**4. Statistiques TCP Stream**
```
Statistics → TCP Stream Graphs → Window Scaling

Graphique montrant l'évolution de la fenêtre :
- Chutes brutales à 0
- Fréquence des zero windows
- Temps de récupération
```

### Résolution

**Côté application :**
```
1. Optimiser le code :
   - Lire le socket plus fréquemment
   - Traiter les données de manière asynchrone
   - Utiliser des buffers plus grands

2. Augmenter les ressources :
   - Plus de CPU
   - Plus de RAM
   - SSD au lieu de HDD
```

**Côté système :**
```
1. Augmenter les buffers TCP :
   # Linux
   sysctl -w net.ipv4.tcp_rmem="8192 262144 16777216"

   # Windows
   netsh int tcp set global autotuninglevel=normal

2. Vérifier qu'aucun processus ne consomme tout le CPU/RAM

3. Optimiser le scheduler I/O
```

**Côté réseau :**
```
1. Si window=0 fréquents entre deux serveurs :
   → Envisager une liaison plus rapide (1G → 10G)
   → Activer TCP Window Scaling
   → Vérifier latence réseau (buffer gonflé si latence haute)
```

---

## 4. Connexions refusées et timeouts

### Symptôme utilisateur

```
"Connection refused"
"Connection timeout"
"Cannot connect to server"
"ERR_CONNECTION_REFUSED"
```

### Type 1 : Connection Refused (RST immédiat)

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
100  1.0000    192.168.1.10  203.0.113.50  TCP       52341 → 80 [SYN] Seq=0
101  1.0456    203.0.113.50  192.168.1.10  TCP       80 → 52341 [RST, ACK]
```

**Dans Packet Details (paquet RST) :**
```
▼ Transmission Control Protocol
    Flags: 0x014 (RST, ACK)
        ...0 .... = Acknowledgment: Not set
        .... 0... = Push: Not set
        .... .1.. = Reset: Set ✅
        .... ..0. = Syn: Not set
        .... ...0 = Fin: Not set
```

**Timeline :**
```
Client                          Serveur
  |                                |
  |--- SYN (port 80) ------------> |
  |                                |  Port 80 fermé
  |<-- RST, ACK ------------------ |  "Je refuse"
  |                                |
[Connection refused]
```

**Causes :**
```
1. Service pas démarré
   → Vérifier : systemctl status nginx

2. Service écoute sur mauvais port
   → Vérifier : netstat -tlnp | grep nginx

3. Service écoute sur mauvaise interface (127.0.0.1 au lieu de 0.0.0.0)
   → Config : listen 0.0.0.0:80 (pas listen 127.0.0.1:80)

4. Service crashed
   → Logs : journalctl -u nginx -n 50
```

**Filtre :**
```
tcp.flags.reset == 1 and tcp.flags.syn == 0
```

### Type 2 : Connection Timeout (pas de réponse)

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
100  1.0000    192.168.1.10  203.0.113.50  TCP       52341 → 80 [SYN] Seq=0
101  2.0000    192.168.1.10  203.0.113.50  TCP       [TCP Retransmission] [SYN] Seq=0
102  4.0000    192.168.1.10  203.0.113.50  TCP       [TCP Retransmission] [SYN] Seq=0
103  8.0000    192.168.1.10  203.0.113.50  TCP       [TCP Retransmission] [SYN] Seq=0
```

**Pattern : SYN retransmis plusieurs fois sans réponse**

**Délais typiques :**
```
SYN initial      : t = 0s
1ère retrans     : t = 1s   (RTO = 1s)
2ème retrans     : t = 3s   (RTO = 2s, doublé)
3ème retrans     : t = 7s   (RTO = 4s, doublé)
4ème retrans     : t = 15s  (RTO = 8s, doublé)
...
Abandon          : t = ~127s (selon OS)
```

**Timeline :**
```
Client                          Serveur/Firewall
  |                                |
  |--- SYN ----------------------->|  [Bloqué/Perdu]
  |                                |
  [Timeout 1s]                     |
  |--- SYN [Retrans] ------------->|  [Bloqué/Perdu]
  |                                |
  [Timeout 2s]                     |
  |--- SYN [Retrans] ------------->|  [Bloqué/Perdu]
  |                                |
  ...                              |
  [Abandon après ~60-127s]
```

**Causes :**
```
1. Firewall bloque (stateful DROP)
   → Paquets SYN droppés silencieusement
   → Pas de RST retourné

2. Routage incorrect
   → Pas de route vers le serveur
   → traceroute pour identifier où ça bloque

3. Serveur down ou inaccessible
   → ping pour tester connectivité IP
   → Si ping OK mais SYN timeout → firewall

4. Anti-DDoS / Rate limiting
   → Trop de connexions → nouvelles bloquées
```

**Filtre :**
```
tcp.flags.syn == 1 and tcp.analysis.retransmission
```

### Type 3 : Connection Reset pendant la communication

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
150  5.0000    192.168.1.10  203.0.113.50  HTTP      GET /data
151  5.0456    203.0.113.50  192.168.1.10  TCP       [RST] Seq=12345
```

**Causes :**
```
1. Application crash
   → Serveur détecte que l'application n'existe plus
   → OS envoie RST

2. Connexion idle trop longtemps
   → Firewall/NAT expire la session
   → Prochains paquets → RST ou drop

3. Maximum connexions atteintes
   → Serveur ferme de vieilles connexions

4. Attaque / Sécurité
   → IDS/IPS détecte pattern suspect
   → Envoie RST pour tuer la connexion
```

**Diagnostic :**
```
Regarder :
1. Quand RST arrive (après combien de temps idle)
2. Qui envoie le RST (client, serveur, intermédiaire)
3. Logs applicatifs au moment du RST
4. Firewall/NAT logs
```

**Filtre :**
```
tcp.flags.reset == 1 and tcp.stream eq N
```

### Résolution par type

**Connection Refused (RST) :**
```
✓ Vérifier que le service tourne
✓ Vérifier le port d'écoute (netstat -tlnp)
✓ Vérifier bind address (0.0.0.0 vs 127.0.0.1)
✓ Checker les logs du service
```

**Connection Timeout (SYN sans réponse) :**
```
✓ Vérifier firewall (iptables -L, firewall-cmd --list-all)
✓ Vérifier routage (traceroute, ip route)
✓ Tester depuis un autre client/réseau
✓ Vérifier ACL sur équipements réseau
```

**RST pendant communication :**
```
✓ Augmenter timeout idle
✓ Implémenter keep-alive applicatif
✓ Vérifier limites de connexions
✓ Analyser logs IDS/IPS
```

---

## 5. Problèmes de MTU et fragmentation

### Symptôme utilisateur

```
"La connexion fonctionne mais pas de données transférées"
"Petits fichiers OK, gros fichiers bloquent"
"SSH fonctionne, SCP/SFTP échoue"
"Certains sites web ne chargent pas"
```

### Signature dans Wireshark

**Fragmentation IP :**
```
No.  Time      Source        Destination   Protocol  Info
200  10.1000   192.168.1.10  203.0.113.50  IPv4      Fragmented IP protocol (proto=TCP, off=0, ID=1a2b)
201  10.1001   192.168.1.10  203.0.113.50  IPv4      Fragmented IP protocol (proto=TCP, off=1480, ID=1a2b)
```

**Dans Packet Details :**
```
▼ Internet Protocol Version 4
    Identification: 0x1a2b (6699)
    Flags: 0x2000, More fragments
        0... .... .... .... = Reserved bit: Not set
        .0.. .... .... .... = Don't fragment: Not set
        ..1. .... .... .... = More fragments: Set ✅
    Fragment Offset: 0
```

**PMTUD (Path MTU Discovery) bloqué :**
```
No.  Time      Source        Destination   Protocol  Info
300  12.0000   192.168.1.10  203.0.113.50  TCP       Seq=1000, Len=1460 [DF]
[Pas de réponse]
301  13.0000   192.168.1.10  203.0.113.50  TCP       [Retrans] Seq=1000, Len=1460 [DF]
[Pas de réponse]
302  14.0000   192.168.1.10  203.0.113.50  TCP       [Retrans] Seq=1000, Len=1460 [DF]
```

**ICMP Fragmentation Needed (devrait être reçu mais ne l'est pas) :**
```
Devrait recevoir :
ICMP Type 3 (Destination Unreachable)
ICMP Code 4 (Fragmentation needed and DF set)
MTU of next hop: 1400

Mais ne reçoit rien → PMTUD black hole
```

### MTU par type de connexion

```
Standard Ethernet       : 1500 bytes
Jumbo Frames           : 9000 bytes
PPPoE (ADSL)           : 1492 bytes (1500 - 8)
VPN (IPsec, OpenVPN)   : 1400-1450 bytes (overhead variable)
GRE tunnel             : 1476 bytes (1500 - 24)
VLAN tagged            : 1496 bytes (1500 - 4)
6to4 tunnel            : 1480 bytes (1500 - 20)
```

### Diagnostic MTU

**1. Identifier les gros paquets retransmis**
```
Filtre :
tcp.analysis.retransmission and frame.len > 1400

Si beaucoup de résultats → Probable problème MTU
```

**2. Vérifier le flag Don't Fragment**
```
Filtre :
ip.flags.df == 1 and frame.len > 1400

Ces paquets ne peuvent PAS être fragmentés
Si MTU chemin < taille paquet → dropped
```

**3. Chercher les ICMP Frag Needed**
```
Filtre :
icmp.type == 3 and icmp.code == 4

Si aucun ICMP → PMTUD black hole
→ Firewall bloque les ICMP
```

**4. Tester manuellement le MTU**
```
# Linux/macOS : ping avec taille spécifique et DF
ping -M do -s 1472 8.8.8.8
# 1472 data + 28 headers (IP+ICMP) = 1500 total

# Si échec, essayer plus petit
ping -M do -s 1400 8.8.8.8

# Windows
ping -f -l 1472 8.8.8.8
```

**Résultat attendu :**
```
Si MTU OK (1500) :
→ ping -s 1472 réussit

Si MTU 1400 :
→ ping -s 1472 échoue (Packet needs to be fragmented)
→ ping -s 1372 réussit
```

### Black hole PMTUD

**Le problème :**
```
1. Client envoie paquet 1500 bytes avec DF=1
2. Routeur intermédiaire a MTU 1400
3. Routeur devrait envoyer ICMP "Frag Needed, MTU=1400"
4. MAIS firewall bloque les ICMP
5. Client ne reçoit jamais l'ICMP
6. Client retransmet le même paquet 1500 bytes
7. Boucle infinie → timeout

C'est le "PMTUD Black Hole"
```

**Signature Wireshark :**
```
Paquets TCP avec :
- Len > 1400
- DF = 1
- Retransmis plusieurs fois
- Jamais d'ACK
- Pas d'ICMP "Frag Needed"

Mais paquets < 1400 passent sans problème
```

### Solutions

**1. Côté client : Réduire MSS**
```
TCP MSS (Maximum Segment Size) = MTU - 40 (IP+TCP headers)

# Linux : Forcer MSS sur interface
ip route add default via 192.168.1.1 advmss 1360

# iptables : Clamp MSS
iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu

# Application : Définir MSS dans socket options
setsockopt(sock, IPPROTO_TCP, TCP_MAXSEG, &mss, sizeof(mss));
```

**2. Côté réseau : Autoriser ICMP Type 3 Code 4**
```
# Firewall : NE PAS bloquer ces ICMP critiques
iptables -A INPUT -p icmp --icmp-type 3/4 -j ACCEPT

# Cisco
access-list 100 permit icmp any any packet-too-big
```

**3. Côté tunnel/VPN : Configurer MTU**
```
# OpenVPN
tun-mtu 1400

# IPsec
ipsec auto --up mytunnel --mtu 1400

# PPPoE
pppoe-server set interface=ether1 mtu=1492
```

**4. Test après correction**
```
# Vérifier que gros paquets passent maintenant
ping -M do -s 1472 destination
curl -v https://site-qui-bloquait-avant/

Dans Wireshark :
- Chercher retransmissions de gros paquets : Devrait disparaître
- Vérifier ICMP reçus : Devrait voir "Frag Needed"
```

---

## 6. Problèmes DNS

### Symptôme utilisateur

```
"Le site ne charge pas" (mais IP directe fonctionne)
"Name resolution failed"
"Unknown host"
"Lenteur au chargement initial des pages"
```

### Type 1 : Pas de réponse DNS

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
400  20.0000   192.168.1.10  192.168.1.1   DNS       Standard query A example.com
401  21.0000   192.168.1.10  192.168.1.1   DNS       [Retransmission] Standard query A example.com
402  22.0000   192.168.1.10  192.168.1.1   DNS       [Retransmission] Standard query A example.com
```

**Pas de réponse après 3-5 tentatives**

**Filtre :**
```
dns.flags.response == 0 and not dns.response_in
```

**Causes :**
```
1. Serveur DNS down
   → Tester : dig @8.8.8.8 example.com

2. Firewall bloque port 53 UDP
   → Vérifier : iptables -L | grep 53

3. Mauvais serveur DNS configuré
   → Vérifier : cat /etc/resolv.conf

4. Problème réseau vers DNS
   → Tester : ping 192.168.1.1 (serveur DNS)
```

### Type 2 : Réponse DNS avec erreur

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
450  25.0000   192.168.1.10  8.8.8.8       DNS       Standard query A badomain.example
451  25.0150   8.8.8.8       192.168.1.10  DNS       Standard query response, No such name
```

**Dans Packet Details :**
```
Domain Name System (response)
    Flags: 0x8183 Standard query response, No such name
        Response code: No such name (3) ✅
```

**Types d'erreurs DNS (RCODE) :**
```
0 : NOERROR    → Succès
1 : FORMERR    → Erreur de format de requête
2 : SERVFAIL   → Serveur a échoué
3 : NXDOMAIN   → Domaine n'existe pas ✅ (le plus courant)
4 : NOTIMP     → Non implémenté
5 : REFUSED    → Requête refusée
```

**Filtre :**
```
dns.flags.rcode != 0                    # Toutes erreurs
dns.flags.rcode == 3                    # NXDOMAIN
dns.flags.rcode == 2                    # SERVFAIL
```

**Causes :**
```
NXDOMAIN (3) :
→ Domaine n'existe pas (typo dans l'URL)
→ Domaine expiré
→ Split-brain DNS (interne vs externe)

SERVFAIL (2) :
→ Serveur DNS surchargé
→ DNSSEC validation échoue
→ Serveur autoritaire inaccessible

REFUSED (5) :
→ Serveur refuse de répondre (ACL, recursion désactivée)
```

### Type 3 : DNS lent

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
500  30.0000   192.168.1.10  8.8.8.8       DNS       Standard query A example.com
501  30.5000   8.8.8.8       192.168.1.10  DNS       Standard query response A 93.184.216.34
```

**Delta : 500ms (trop lent !)**

**Dans Packet Details :**
```
[Time: 0.500000 seconds] ✅
```

**Filtre :**
```
dns.time > 0.1                          # Réponses > 100ms
dns.time > 0.5                          # Réponses > 500ms
```

**Seuils de performance :**
```
< 10ms   : Excellent (cache local ou DNS proche)
10-50ms  : Bon
50-100ms : Acceptable
100-500ms: Lent (investigation recommandée)
> 500ms  : Très lent (problème)
> 1s     : Inacceptable
```

**Causes :**
```
1. Serveur DNS lointain
   → Utiliser DNS plus proche (FAI, 8.8.8.8)

2. Serveur DNS surchargé
   → Load balancing, serveurs supplémentaires

3. Récursion lente
   → Serveur DNS doit interroger plusieurs autoritaires
   → Utiliser un resolver avec bon cache

4. Réseau saturé
   → QoS pour prioriser DNS
```

### Type 4 : Cache DNS empoisonné/obsolète

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
550  35.0000   192.168.1.10  192.168.1.1   DNS       Standard query A example.com
551  35.0100   192.168.1.1   192.168.1.10  DNS       Standard query response A 1.2.3.4
```

**Mais l'IP correcte est 93.184.216.34 !**

**Diagnostic :**
```
1. Comparer avec serveur DNS public
   dig @8.8.8.8 example.com        # 93.184.216.34 (correct)
   dig @192.168.1.1 example.com    # 1.2.3.4 (incorrect)

2. Vérifier TTL expiré
   dig example.com
   → Si TTL très ancien mais pas rafraîchi → cache stuck

3. Forcer refresh
   # Vider cache local
   sudo systemd-resolve --flush-caches    # Linux
   ipconfig /flushdns                      # Windows
   sudo dscacheutil -flushcache           # macOS
```

### Diagnostic DNS dans Wireshark

**1. Voir toutes les requêtes/réponses pour un domaine**
```
dns.qry.name == "example.com"
```

**2. Statistiques DNS**
```
Statistics → DNS

Tableau montrant :
- Nombre de requêtes/réponses
- Types d'enregistrements (A, AAAA, MX, etc.)
- RCODE distribution
- Top domaines interrogés
```

**3. Mesurer temps de réponse moyen**
```
Filtre : dns.time

Statistics → I/O Graph
Y Axis : AVG(dns.time)
→ Voir l'évolution du temps de réponse DNS
```

### Résolution

**Pas de réponse :**
```
✓ Vérifier connectivité vers serveur DNS (ping)
✓ Tester serveurs DNS alternatifs (8.8.8.8, 1.1.1.1)
✓ Vérifier firewall (port 53 UDP/TCP)
✓ Vérifier /etc/resolv.conf
```

**Erreurs NXDOMAIN/SERVFAIL :**
```
✓ Vérifier orthographe du domaine
✓ Tester avec dig directement sur autoritaire
✓ Vérifier DNSSEC si activé
✓ Consulter logs du serveur DNS
```

**Lenteur :**
```
✓ Utiliser DNS plus proche géographiquement
✓ Configurer cache DNS local (dnsmasq, unbound)
✓ Augmenter TTL si contrôle du domaine
✓ Pré-résoudre domaines critiques (prefetch)
```

---

## 7. Problèmes ARP

### Symptôme utilisateur

```
"Connectivité intermittente sur le réseau local"
"Impossible de joindre la passerelle"
"IP duplicate detected"
```

### Type 1 : ARP Request sans réponse

**Signature Wireshark :**
```
No.  Time      Source        Destination   Protocol  Info
600  40.0000   00:1a:2b:3c:4d:5e Broadcast  ARP       Who has 192.168.1.50? Tell 192.168.1.10
601  41.0000   00:1a:2b:3c:4d:5e Broadcast  ARP       Who has 192.168.1.50? Tell 192.168.1.10
602  42.0000   00:1a:2b:3c:4d:5e Broadcast  ARP       Who has 192.168.1.50? Tell 192.168.1.10
```

**Pas de ARP Reply**

**Filtre :**
```
arp.opcode == 1 and arp.dst.proto_ipv4 == 192.168.1.50
```

**Causes :**
```
1. Machine 192.168.1.50 éteinte ou déconnectée
2. Mauvais subnet (192.168.1.10/24 tente de joindre 192.168.2.50)
3. VLAN isolation
4. Firewall L2 bloque ARP
```

### Type 2 : ARP Duplicate (conflit IP)

**Signature Wireshark :**
```
No.  Time      Source            Destination   Protocol  Info
650  45.0000   00:1a:2b:3c:4d:5e Broadcast     ARP       192.168.1.10 is at 00:1a:2b:3c:4d:5e
651  45.0001   aa:bb:cc:dd:ee:ff Broadcast     ARP       192.168.1.10 is at aa:bb:cc:dd:ee:ff ⚠️
```

**Deux MACs différentes pour la même IP !**

**Filtre :**
```
arp.duplicate-address-detected
```

**Causes :**
```
1. IP configurée statiquement sur deux machines
2. DHCP a attribué une IP statique déjà utilisée
3. ARP spoofing (attaque MITM)
4. Machine redémarre avec nouvelle MAC (VM/Docker)
```

**Diagnostic :**
```
1. Identifier les deux MACs
   Filtre : arp.src.proto_ipv4 == 192.168.1.10

2. Résoudre OUI pour identifier fabricant
   00:1a:2b → Cisco
   aa:bb:cc → Raspberry Pi

3. Vérifier table ARP
   arp -a | grep 192.168.1.10
   # Peut osciller entre les deux MACs
```

### Type 3 : ARP Storm (tempête broadcast)

**Signature Wireshark :**
```
Centaines/milliers d'ARP Request en quelques secondes
```

**Filtre :**
```
arp

Statistics → I/O Graph
→ Pic énorme de paquets ARP
```

**Causes :**
```
1. Boucle réseau (sans Spanning Tree)
2. Malware/botnet scannant le réseau
3. Mauvaise config (proxy ARP mal configuré)
```

### Type 4 : Gratuitous ARP inattendu

**Signature Wireshark :**
```
No.  Time      Source            Destination   Protocol  Info
700  50.0000   00:1a:2b:3c:4d:5e Broadcast     ARP       192.168.1.10 is at 00:1a:2b:3c:4d:5e

(Sans ARP Request préalable)
```

**Gratuitous ARP = Annonce non sollicitée**

**Usage légitime :**
```
- Machine démarre ou change d'IP
- Interface réseau s'active
- Failover (IP bascule vers serveur backup)
- Détection de duplicate IP
```

**Usage malveillant :**
```
- ARP spoofing / poisoning (MITM attack)
→ Attaquant annonce qu'il possède l'IP de la passerelle
→ Trafic redirigé vers l'attaquant
```

**Filtre :**
```
arp.isgratuitous == 1
```

### Résolution

**ARP sans réponse :**
```
✓ Vérifier que la machine cible est allumée
✓ Vérifier subnet mask correct
✓ Ping direct de la machine cible
✓ Vérifier VLAN configuration
```

**Duplicate IP :**
```
✓ Identifier les deux machines (via MAC OUI)
✓ Changer l'IP de l'une des deux
✓ Vérifier DHCP range vs IPs statiques
✓ Consulter DHCP lease table
```

**ARP Storm :**
```
✓ Activer Spanning Tree Protocol (STP)
✓ Identifier et supprimer la boucle réseau
✓ Implémenter Storm Control sur switches
✓ Scanner pour malware si suspect
```

**ARP Spoofing (sécurité) :**
```
✓ Implémenter Dynamic ARP Inspection (DAI) sur switches
✓ Utiliser static ARP entries pour serveurs critiques
✓ Monitorer avec IDS (Snort, Suricata)
✓ Segmenter réseau avec VLANs
```

---

## 8. Problèmes HTTP/HTTPS spécifiques

### Type 1 : HTTP 4xx (erreurs client)

**404 Not Found**
```
Filtre : http.response.code == 404

Causes :
- URL incorrecte (typo)
- Ressource supprimée/déplacée
- Mauvaise config reverse proxy
```

**401 Unauthorized / 403 Forbidden**
```
Filtre : http.response.code == 401 or http.response.code == 403

401 : Authentification requise
403 : Accès refusé même authentifié

Diagnostic Wireshark :
- Vérifier header Authorization
- Vérifier cookies de session
```

**400 Bad Request**
```
Filtre : http.response.code == 400

Causes :
- Headers malformés
- URL trop longue
- Body invalide (JSON mal formé)

Dans Wireshark :
Follow HTTP Stream pour voir requête exacte
```

### Type 2 : HTTP 5xx (erreurs serveur)

**500 Internal Server Error**
```
Filtre : http.response.code == 500

Causes :
- Exception/crash applicatif
- Configuration serveur incorrecte
- Database connection failed

Diagnostic :
- Vérifier logs serveur (crucial)
- Voir timing : immédiat = config, delayed = timeout DB
```

**502 Bad Gateway**
```
Filtre : http.response.code == 502

Signification : Reverse proxy ne peut pas joindre backend

Timeline Wireshark :
Client → Proxy : GET /
Proxy → Backend : [SYN]
Backend → Proxy : [RST] ou [Timeout]
Proxy → Client : 502 Bad Gateway

Causes :
- Backend down
- Firewall entre proxy et backend
- Backend surchargé (refuse nouvelles connexions)
```

**503 Service Unavailable**
```
Filtre : http.response.code == 503

Causes :
- Serveur en maintenance
- Trop de connexions (max capacity)
- Health check échoue

Header à vérifier :
Retry-After: 120
→ Réessayer dans 120 secondes
```

**504 Gateway Timeout**
```
Filtre : http.response.code == 504

Signification : Proxy a timeout en attendant backend

Mesurer dans Wireshark :
Temps entre requête proxy→backend et 504
Si > 60s → timeout par défaut nginx
Si > 30s → timeout par défaut Apache

Causes :
- Requête backend très lente (query SQL longue)
- Backend bloqué
- Timeout configuré trop court
```

### Type 3 : Redirections en boucle

**Signature Wireshark :**
```
No.  Info
800  GET /page1 HTTP/1.1
801  HTTP/1.1 302 Found   Location: /page2
802  GET /page2 HTTP/1.1
803  HTTP/1.1 302 Found   Location: /page1
804  GET /page1 HTTP/1.1
805  HTTP/1.1 302 Found   Location: /page2
...
```

**Filtre :**
```
http.response.code == 301 or http.response.code == 302
```

**Causes :**
```
- Mauvaise config reverse proxy
- Cookie/session invalide créant boucle
- HTTP → HTTPS redirection mal configurée
```

### Type 4 : Connexion HTTPS bloquée

**Impossible de terminer handshake TLS**

**Signature Wireshark :**
```
No.  Time      Protocol  Info
900  60.0000   TLSv1.3   Client Hello
901  60.2000   TLSv1.3   Server Hello, Certificate, ...
902  60.2100   TLSv1.3   Alert (Level: Fatal, Description: Certificate Unknown)
```

**Types d'alertes TLS :**
```
certificate_unknown (46)     → Certificat non reconnu
certificate_expired (45)     → Certificat expiré
handshake_failure (40)       → Échec négociation cipher
protocol_version (70)        → Version TLS incompatible
```

**Filtre :**
```
tls.alert_message
```

**Diagnostic certificat :**
```
# Vérifier certificat manuellement
openssl s_client -connect example.com:443 -servername example.com

Vérifier dans Wireshark :
Handshake Protocol → Certificate
→ Subject, Issuer, Validity dates
→ Voir si auto-signé, expiré, mauvais CN
```

---

## 9. Keep-Alive et connexions persistantes

### Problème : Connexions HTTP/1.1 non réutilisées

**Symptôme :**
```
Performance médiocre malgré HTTP/1.1
Beaucoup de handshakes TCP pour un seul site
```

**Signature Wireshark :**
```
Client ouvre connexion, fait 1 requête, ferme
Client ouvre nouvelle connexion, fait 1 requête, ferme
...

Au lieu de :
Client ouvre connexion
Fait requête 1, 2, 3, 4... sur MÊME connexion
```

**Vérifier dans Wireshark :**
```
Filtre : http

Regarder tcp.stream :
Si chaque requête HTTP a un tcp.stream différent
→ Pas de réutilisation (mauvais)

Si plusieurs requêtes HTTP ont même tcp.stream
→ Réutilisation OK (bon)
```

**Causes :**
```
1. Header Connection: close envoyé
   → Serveur ferme après chaque requête

2. Client ne supporte pas keep-alive
   → Vieux client HTTP/1.0

3. Timeout keep-alive trop court
   → Connexion fermée entre requêtes

4. Proxy intermédiaire casse keep-alive
```

**Vérifier headers HTTP :**
```
Follow HTTP Stream

Requête :
Connection: keep-alive ✅

Réponse :
Connection: close ❌
→ Serveur refuse keep-alive
```

---

## 10. Slow Read Attack / Slow Response

### Symptôme

```
Serveur semble "bloqué"
Connexions restent ouvertes très longtemps
Épuisement des connexions disponibles
```

### Signature Wireshark

**Attaque Slow Read (client lit lentement) :**
```
No.  Time      Source        Destination   Protocol  Info
1000 70.0000   Server        Client        TCP       Seq=1000, Len=1460, Win=65535
1001 70.0001   Client        Server        TCP       ACK, Win=100 ⚠️
[Serveur attend que Win augmente]
1002 75.0000   Client        Server        TCP       ACK, Win=200 ⚠️
[5 secondes pour lire 100 bytes !]
```

**Window size du client augmente TRÈS lentement**
→ Serveur ne peut pas envoyer (respecte flow control)
→ Connexion monopolisée longtemps

**Attaque Slow POST (client envoie lentement) :**
```
No.  Time      Source        Destination   Protocol  Info
1100 80.0000   Client        Server        HTTP      POST /upload (partial)
[Envoie 10 bytes]
1101 85.0000   Client        Server        HTTP      POST continuation
[Envoie 10 bytes après 5 secondes]
1102 90.0000   Client        Server        HTTP      POST continuation
...

Content-Length: 1000000
Mais envoie 10 bytes toutes les 5 secondes
→ Prendra 5000 secondes = 83 minutes
```

### Filtre

```
# Connexions très longues
tcp.time_relative > 60

# Window size anormalement petit
tcp.window_size < 1000 and tcp.len > 0

# POST avec envoi très lent
http.request.method == "POST" and tcp.time_relative > 10
```

### Protection

```
1. Timeout de lecture/écriture courts
   nginx: client_body_timeout 10s;

2. Limiter connexions par IP
   iptables hashlimit, fail2ban

3. Reverse proxy avec protection (Cloudflare, AWS WAF)

4. Minimum data rate requis
   Apache: RequestReadTimeout body=10
```

---

## Récapitulatif : Checklist de diagnostic

### Workflow général

```
1. SYMPTÔME : Qu'est-ce que l'utilisateur rapporte ?
   └→ "Lent", "Timeout", "Connexion impossible", etc.

2. CAPTURE : Capturer au bon endroit
   └→ Client, serveur, entre les deux ?

3. FILTRER : Isoler le trafic concerné
   └→ IP, port, protocole, stream

4. IDENTIFIER : Chercher les patterns problématiques
   └→ Retransmissions, RST, Zero Window, Timeouts

5. MESURER : Quantifier le problème
   └→ Taux de perte, latence, durée des anomalies

6. ANALYSER : Comprendre la cause
   └→ Réseau, application, configuration

7. RÉSOUDRE : Appliquer la solution
   └→ Selon la cause identifiée

8. VALIDER : Vérifier la résolution
   └→ Nouvelle capture sans le problème
```

### Filtres de diagnostic rapide

```
# Vue d'ensemble des problèmes
tcp.analysis.flags

# Problèmes de connexion
tcp.flags.reset == 1
tcp.flags.syn == 1 and tcp.analysis.retransmission

# Problèmes de performance
tcp.analysis.retransmission
tcp.analysis.duplicate_ack
tcp.window_size == 0
tcp.analysis.zero_window

# Problèmes applicatifs
http.response.code >= 400
dns.flags.rcode != 0
tls.alert_message

# Problèmes réseau local
arp.duplicate-address-detected
icmp.type == 3

# Mesure de performance
frame.time_delta > 0.5
dns.time > 0.1
http.time > 1
```

### Expert Info : Vue synthétique

```
Analyze → Expert Information

Catégories :
- Error (Rouge) : Problèmes graves
- Warning (Jaune) : Avertissements
- Note (Cyan) : Informations
- Chat (Bleu) : Événements normaux

Focus sur :
- Checksum errors
- Retransmissions
- Zero windows
- Connection resets
```

---

## Points clés à retenir

### 🎯 Patterns de problèmes

**Retransmissions = Perte de paquets**
```
Causes : Congestion, lien instable, firewall, MTU
Taux normal : < 0.1%
Taux problématique : > 1%
```

**Zero Window = Récepteur saturé**
```
Causes : App lente, CPU/RAM insuffisant, buffer petit
Impact : Débit zéro temporairement
```

**RST = Connexion fermée brutalement**
```
Causes : Port fermé, app crash, firewall, timeout
Identifier : Qui envoie le RST (client/serveur/intermédiaire)
```

**Timeout = Pas de réponse**
```
SYN sans réponse : Firewall, routage, serveur down
DNS sans réponse : Serveur DNS, firewall port 53
```

**Fragmentation = MTU inadapté**
```
Symptôme : Petits paquets OK, gros paquets KO
Cause : MTU chemin < MTU interface
Solution : Réduire MSS, autoriser ICMP Frag Needed
```

### 🎯 Méthodologie

1. **Observer** avant de conclure
2. **Filtrer** pour isoler le problème
3. **Mesurer** pour quantifier
4. **Comparer** avec le trafic normal
5. **Corréler** avec logs/métriques
6. **Tester** la solution
7. **Documenter** pour la prochaine fois

### 🎯 Outils complémentaires

Wireshark ne suffit pas toujours :
- **Logs applicatifs** : Comprendre le contexte
- **Métriques système** : CPU, RAM, I/O
- **Monitoring réseau** : Graphiques temporels
- **Tests actifs** : ping, traceroute, curl

---

## Conclusion

L'identification des problèmes réseau avec Wireshark est un art qui combine :
- Connaissance des protocoles (modules précédents)
- Maîtrise de Wireshark (sections 7.3, 7.4, 7.5)
- Expérience (reconnaître les patterns)
- Méthodologie (approche structurée)

Plus vous analyserez de captures, plus vous développerez l'intuition pour identifier rapidement les anomalies. Les problèmes présentés dans cette section couvrent 90% des cas rencontrés en production.

Dans la prochaine section, nous irons plus loin avec l'analyse de performances réseau : mesurer le débit réel, calculer le RTT, identifier les goulots d'étranglement, et optimiser les performances.

---

**Prochaine section :** 7.7 Analyse de performances réseau

Nous apprendrons à mesurer précisément les performances réseau : débit, latence, jitter, efficacité TCP, et comment identifier les facteurs limitants.

⏭️ [Analyse de performances réseau](/07-analyse-depannage/07-analyse-performances.md)
