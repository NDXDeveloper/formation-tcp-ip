🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.2 Outils en ligne de commande : ping, traceroute, netstat, ss, nslookup, dig

## Introduction

Les outils en ligne de commande sont les couteaux suisses du diagnostic réseau. Disponibles sur pratiquement tous les systèmes d'exploitation, ils permettent de diagnostiquer rapidement la plupart des problèmes sans installer de logiciel supplémentaire.

Ces outils testent différentes couches du modèle TCP/IP :

```
Couche 7 - Application    │ nslookup, dig, curl
Couche 4 - Transport      │ netstat, ss, telnet, nc
Couche 3 - Réseau         │ ping, traceroute, ip route
Couche 2 - Liaison        │ arp, ip neigh
```

Dans cette section, nous allons explorer en profondeur les outils les plus utilisés au quotidien. Pour chaque outil, nous verrons son fonctionnement, sa syntaxe, des exemples concrets, et comment interpréter les résultats.

---

## ping - Test de connectivité et mesure de latence

### Qu'est-ce que ping ?

`ping` est probablement l'outil de diagnostic réseau le plus connu. Il permet de :
- Tester si une machine distante est joignable
- Mesurer le temps de réponse (latence/RTT)
- Détecter la perte de paquets
- Vérifier la stabilité d'une connexion

### Comment ça fonctionne ?

`ping` utilise le protocole **ICMP (Internet Control Message Protocol)** :

1. L'émetteur envoie un paquet **ICMP Echo Request** (type 8)
2. Le destinataire répond avec un **ICMP Echo Reply** (type 0)
3. L'émetteur mesure le temps entre l'envoi et la réception

```
Client                              Serveur
  |                                    |
  |---> ICMP Echo Request (type 8) --->|
  |                                    |
  |<--- ICMP Echo Reply (type 0) ------|
  |                                    |
  └─ Mesure du RTT (Round Trip Time)
```

### Syntaxe de base

**Linux/macOS :**
```bash
ping [options] destination
```

**Windows :**
```cmd
ping [options] destination
```

### Options principales

| Option | Linux/macOS | Windows | Description |
|--------|-------------|---------|-------------|
| Nombre de paquets | `-c count` | `-n count` | Envoyer N paquets puis arrêter |
| Intervalle | `-i seconds` | N/A | Délai entre chaque paquet |
| Taille du paquet | `-s size` | `-l size` | Taille des données (en octets) |
| Timeout | `-W seconds` | `-w milliseconds` | Temps d'attente par réponse |
| Don't Fragment | `-M do` | `-f` | Interdit la fragmentation IP |
| Interface source | `-I interface` | N/A | Interface à utiliser |
| Mode silencieux | `-q` | N/A | N'affiche que le résumé |
| Mode verbeux | `-v` | N/A | Affiche plus de détails |

### Exemples pratiques

#### Exemple 1 : Ping simple

```bash
ping google.com
```

**Sortie (Linux) :**
```
PING google.com (142.250.185.46) 56(84) bytes of data.
64 bytes from par21s23-in-f14.1e100.net (142.250.185.46): icmp_seq=1 ttl=116 time=12.3 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.185.46): icmp_seq=2 ttl=116 time=11.8 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.185.46): icmp_seq=3 ttl=116 time=12.5 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.185.46): icmp_seq=4 ttl=116 time=11.9 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 11.802/12.125/12.542/0.295 ms
```

**Interprétation :**
- `142.250.185.46` : Adresse IP résolue pour google.com
- `64 bytes` : Taille du paquet ICMP (56 bytes de données + 8 bytes header)
- `icmp_seq=1` : Numéro de séquence (permet de détecter les paquets perdus)
- `ttl=116` : Time To Live (nombre de sauts restants avant expiration)
- `time=12.3 ms` : **RTT (Round Trip Time)** - temps aller-retour
- `0% packet loss` : Aucun paquet perdu (excellente connexion)
- `rtt min/avg/max/mdev` : Statistiques de latence

#### Exemple 2 : Ping avec nombre limité de paquets

```bash
# Linux/macOS
ping -c 5 8.8.8.8

# Windows
ping -n 5 8.8.8.8
```

**Sortie :**
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=9.45 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=9.32 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=9.51 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=118 time=9.28 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=118 time=9.47 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 9.281/9.406/9.505/0.096 ms
```

**Utilité :** Idéal pour des tests automatisés ou scripts.

#### Exemple 3 : Détecter la perte de paquets

```bash
ping -c 20 unstable-server.example.com
```

**Sortie (connexion instable) :**
```
PING unstable-server.example.com (203.0.113.50) 56(84) bytes of data.
64 bytes from 203.0.113.50: icmp_seq=1 ttl=64 time=25.3 ms
64 bytes from 203.0.113.50: icmp_seq=2 ttl=64 time=24.8 ms
Request timeout for icmp_seq 3
64 bytes from 203.0.113.50: icmp_seq=4 ttl=64 time=156 ms
64 bytes from 203.0.113.50: icmp_seq=5 ttl=64 time=23.7 ms
Request timeout for icmp_seq 6
Request timeout for icmp_seq 7
64 bytes from 203.0.113.50: icmp_seq=8 ttl=64 time=24.1 ms
...

--- unstable-server.example.com ping statistics ---
20 packets transmitted, 16 received, 20% packet loss, time 19028ms
rtt min/avg/max/mdev = 23.712/45.327/156.043/32.451 ms
```

**Interprétation :**
- **20% packet loss** : 4 paquets perdus sur 20
- Temps de réponse variable (24ms à 156ms) → **latence instable**
- **Diagnostic** : Problème de qualité réseau (saturation, équipement défaillant, Wi-Fi instable)

#### Exemple 4 : Test de MTU avec Don't Fragment

Le MTU (Maximum Transmission Unit) est la taille maximale d'un paquet. Tester avec différentes tailles permet de détecter des problèmes de fragmentation.

```bash
# Linux : ping avec paquet de 1472 bytes + 28 bytes (IP+ICMP header) = 1500 bytes (MTU standard Ethernet)
ping -M do -s 1472 -c 4 google.com

# Si ça échoue, essayer avec une taille plus petite
ping -M do -s 1400 -c 4 google.com

# Windows
ping -f -l 1472 google.com
```

**Sortie (MTU trop grand) :**
```
PING google.com (142.250.185.46) 1472(1500) bytes of data.
ping: local error: message too long, mtu=1400

--- google.com ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss
```

**Interprétation :**
- Le MTU du chemin est inférieur à 1500 bytes
- Il faut réduire la taille des paquets
- **Cause possible :** Tunnel VPN, PPPoE, encapsulation VLAN

**Sortie (MTU OK) :**
```
PING google.com (142.250.185.46) 1472(1500) bytes of data.
1480 bytes from par21s23-in-f14.1e100.net (142.250.185.46): icmp_seq=1 ttl=116 time=12.8 ms
1480 bytes from par21s23-in-f14.1e100.net (142.250.185.46): icmp_seq=2 ttl=116 time=12.3 ms
```

**Diagnostic :** MTU de 1500 fonctionne correctement sur ce chemin.

#### Exemple 5 : Ping flood (test de charge)

**⚠️ À utiliser UNIQUEMENT sur vos propres serveurs avec autorisation !**

```bash
# Linux (nécessite root)
sudo ping -f -c 1000 192.168.1.100
```

**Sortie :**
```
PING 192.168.1.100 (192.168.1.100) 56(84) bytes of data.
.................................
--- 192.168.1.100 ping statistics ---
1000 packets transmitted, 997 received, 0.3% packet loss, time 2041ms
rtt min/avg/max/mdev = 0.145/0.892/15.234/1.234 ms, ipg/ewma 2.043/0.945 ms
```

**Utilité :** Tester la capacité d'un serveur à gérer un volume élevé de paquets.

### Interpréter le TTL

Le TTL (Time To Live) indique le nombre de sauts restants :

```
TTL = 64   → Système Unix/Linux (TTL initial = 64)
TTL = 128  → Windows (TTL initial = 128)
TTL = 255  → Équipement réseau (routeur, switch layer 3)

TTL = 116  → 128 - 12 sauts = machine Windows à 12 sauts de distance
TTL = 54   → 64 - 10 sauts = machine Linux à 10 sauts de distance
```

**Exemple :**
```bash
ping 8.8.8.8
# 64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=9.45 ms
```

**Calcul :** TTL initial probable = 128 (Windows/routeur)
**Sauts :** 128 - 118 = **10 sauts** entre moi et 8.8.8.8

### Cas d'usage typiques

#### 1. Vérifier la connectivité de base

```bash
# Test local (127.0.0.1 = localhost)
ping 127.0.0.1
# ✅ Vérifie que la pile TCP/IP fonctionne

# Test de la passerelle
ping 192.168.1.1
# ✅ Vérifie que le réseau local fonctionne

# Test d'un serveur Internet
ping 8.8.8.8
# ✅ Vérifie que le routage et Internet fonctionnent

# Test avec résolution DNS
ping google.com
# ✅ Vérifie en plus que le DNS fonctionne
```

#### 2. Mesurer la latence vers différents serveurs

```bash
# Serveur proche (même datacenter)
ping internal-server.local
# Attendu : 0.5-2 ms

# Serveur dans le même pays
ping cdn-paris.example.com
# Attendu : 5-15 ms

# Serveur intercontinental (Europe → USA)
ping us-east.example.com
# Attendu : 80-120 ms

# Serveur très éloigné (Europe → Australie)
ping sydney.example.com
# Attendu : 250-350 ms
```

#### 3. Diagnostiquer une connexion instable

```bash
# Ping continu pour observer les variations
ping -i 0.2 server.example.com
# -i 0.2 = paquet toutes les 0.2 secondes

# Observer :
# - Variations de latence (jitter)
# - Paquets perdus
# - Pics de latence à des moments spécifiques
```

### Limitations de ping

#### ❌ Limitation 1 : ICMP peut être bloqué

Beaucoup de serveurs bloquent ICMP pour des raisons de sécurité :

```bash
ping facebook.com
# Request timeout
# ❌ Mais le site web fonctionne parfaitement !
```

**Solution :** Utiliser `telnet`, `nc` ou `curl` pour tester les ports TCP.

#### ❌ Limitation 2 : Ne teste pas les services applicatifs

```bash
ping db-server.example.com
# 64 bytes from db-server.example.com: time=2.3 ms
# ✅ Le serveur répond au ping

telnet db-server.example.com 5432
# Connection refused
# ❌ Le service PostgreSQL n'est pas accessible
```

**Conclusion :** Ping confirme la connectivité IP, pas la disponibilité du service.

#### ❌ Limitation 3 : Ne reflète pas toujours la réalité du trafic

ICMP peut être traité différemment du trafic TCP/UDP par les équipements réseau (priorité, QoS).

---

## traceroute / tracert - Traçage du chemin réseau

### Qu'est-ce que traceroute ?

`traceroute` (Linux/macOS) ou `tracert` (Windows) permet de :
- Visualiser le **chemin complet** entre vous et une destination
- Identifier **chaque routeur** traversé (hop)
- Mesurer la **latence à chaque saut**
- Localiser où un paquet est **bloqué ou ralenti**

### Comment ça fonctionne ?

`traceroute` exploite le champ **TTL (Time To Live)** des paquets IP :

```
Étape 1 : Envoyer un paquet avec TTL=1
  → Le 1er routeur décrémente TTL à 0
  → Le routeur renvoie ICMP "Time Exceeded"
  → On connaît le 1er hop

Étape 2 : Envoyer un paquet avec TTL=2
  → Le 1er routeur décrémente (TTL=1), transfère
  → Le 2ème routeur décrémente à 0
  → Le 2ème routeur renvoie ICMP "Time Exceeded"
  → On connaît le 2ème hop

Étape 3 : Envoyer un paquet avec TTL=3
  → ...et ainsi de suite jusqu'à atteindre la destination
```

**Différence entre systèmes :**
- **Linux/macOS** : Utilise des paquets UDP par défaut (port 33434+)
- **Windows** : Utilise ICMP Echo Request (comme ping)

### Syntaxe de base

**Linux/macOS :**
```bash
traceroute [options] destination
```

**Windows :**
```cmd
tracert destination
```

### Options principales

| Option | Linux/macOS | Windows | Description |
|--------|-------------|---------|-------------|
| Max hops | `-m max_ttl` | `-h max_hops` | Nombre max de sauts |
| Utiliser ICMP | `-I` | Par défaut | Utiliser ICMP au lieu d'UDP |
| Utiliser TCP | `-T` | N/A | Utiliser TCP SYN |
| Port destination | `-p port` | N/A | Port UDP à utiliser |
| Nombre de queries | `-q nqueries` | N/A | Paquets par hop (défaut: 3) |
| Timeout | `-w seconds` | `-w timeout` | Attente par réponse |
| Ne pas résoudre | `-n` | `-d` | Ne pas résoudre les IP en noms |

### Exemples pratiques

#### Exemple 1 : Traceroute simple

```bash
traceroute google.com
```

**Sortie (Linux) :**
```
traceroute to google.com (142.250.185.46), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  1.234 ms  1.156 ms  1.089 ms
 2  10.255.255.1 (10.255.255.1)  8.432 ms  8.521 ms  8.398 ms
 3  ae101-0.ncren1.rennes.franceix.net (37.49.236.73)  9.234 ms  9.187 ms  9.321 ms
 4  be2.rbx-g2-a9.fr.eu (37.187.231.73)  12.456 ms  12.389 ms  12.512 ms
 5  72.14.223.50 (72.14.223.50)  12.678 ms  12.734 ms  12.621 ms
 6  108.170.252.1 (108.170.252.1)  12.834 ms  11.923 ms  12.012 ms
 7  142.251.49.158 (142.251.49.158)  12.456 ms  12.523 ms  12.398 ms
 8  par21s23-in-f14.1e100.net (142.250.185.46)  12.678 ms  12.734 ms  12.821 ms
```

**Interprétation ligne par ligne :**

```
Hop 1 : _gateway (192.168.1.1)  1.234 ms  1.156 ms  1.089 ms
```
- **Hop 1** : Ma box/routeur domestique
- **192.168.1.1** : Passerelle par défaut
- **1.234 ms, 1.156 ms, 1.089 ms** : 3 mesures de latence vers ce hop
- **Latence ~1ms** : Normal pour réseau local

```
Hop 2 : 10.255.255.1 (10.255.255.1)  8.432 ms  8.521 ms  8.398 ms
```
- **Hop 2** : Routeur de mon FAI (Fournisseur d'Accès Internet)
- **10.255.255.1** : IP privée du FAI
- **~8ms** : Normal pour connexion ADSL/Fibre

```
Hop 3 : ae101-0.ncren1.rennes.franceix.net
```
- **Hop 3** : Point d'interconnexion France-IX à Rennes
- **~9ms** : Sortie du réseau du FAI vers le backbone Internet

```
Hop 4-7 : Traversée du réseau Google
```
- Plusieurs routeurs backbone de Google
- Latence stable autour de 12ms

```
Hop 8 : par21s23-in-f14.1e100.net (142.250.185.46)
```
- **Destination atteinte** : Serveur Google à Paris
- **12.678 ms** : Latence totale

#### Exemple 2 : Traceroute avec problème

```bash
traceroute problematic-server.example.com
```

**Sortie (connexion bloquée) :**
```
traceroute to problematic-server.example.com (203.0.113.50), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  1.123 ms  1.067 ms  1.145 ms
 2  10.255.255.1 (10.255.255.1)  8.234 ms  8.312 ms  8.189 ms
 3  core-router.isp.net (198.51.100.1)  9.456 ms  9.523 ms  9.412 ms
 4  border-router.isp.net (198.51.100.254)  10.123 ms  10.234 ms  10.089 ms
 5  * * *
 6  * * *
 7  * * *
 8  * * *
...
30  * * *
```

**Interprétation :**
- **Hops 1-4** : Réponses normales
- **Hop 5 et suivants** : `* * *` = Pas de réponse
- **Diagnostic** : Le paquet est probablement bloqué par :
  - Un firewall après le hop 4
  - Le routeur 5 ne répond pas aux ICMP Time Exceeded
  - La destination est injoignable

#### Exemple 3 : Traceroute montrant un goulot d'étranglement

```bash
traceroute slow-server.example.com
```

**Sortie (problème de latence) :**
```
traceroute to slow-server.example.com (203.0.113.100), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  1.234 ms  1.189 ms  1.256 ms
 2  isp-router (10.255.255.1)  8.456 ms  8.523 ms  8.412 ms
 3  regional-hub (198.51.100.10)  12.345 ms  12.456 ms  12.234 ms
 4  international-link (198.51.100.50)  15.678 ms  15.734 ms  15.612 ms
 5  congested-router (203.0.112.1)  156.234 ms  158.456 ms  155.789 ms
 6  destination-network (203.0.113.1)  157.123 ms  159.234 ms  156.890 ms
 7  slow-server.example.com (203.0.113.100)  158.456 ms  160.123 ms  157.678 ms
```

**Interprétation :**
- **Saut brutal de latence au hop 5** : De 15ms à 156ms
- **Goulot d'étranglement identifié** : `congested-router (203.0.112.1)`
- **Cause probable** : Lien saturé, équipement surchargé
- **Action** : Contacter l'opérateur du réseau 203.0.112.0/24

#### Exemple 4 : Traceroute vers un autre continent

```bash
traceroute tokyo.example.jp
```

**Sortie (Europe → Japon) :**
```
traceroute to tokyo.example.jp (203.0.113.200), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  1.123 ms  1.089 ms  1.156 ms
 2  isp-router (10.255.255.1)  8.234 ms  8.312 ms  8.189 ms
 3  paris-ix (37.49.236.73)  9.456 ms  9.523 ms  9.412 ms
 4  london-link (185.1.2.3)  15.678 ms  15.734 ms  15.612 ms
 5  transatlantic-cable (89.149.180.1)  85.234 ms  85.456 ms  85.123 ms
 6  new-york-hub (154.54.24.1)  86.678 ms  86.823 ms  86.534 ms
 7  chicago-junction (154.54.30.1)  105.234 ms  105.456 ms  105.123 ms
 8  san-francisco (154.54.40.1)  125.678 ms  125.823 ms  125.534 ms
 9  transpacific-cable (202.97.33.1)  215.234 ms  215.456 ms  215.123 ms
10  tokyo-ix (203.0.113.1)  216.678 ms  216.823 ms  216.534 ms
11  tokyo.example.jp (203.0.113.200)  217.234 ms  217.456 ms  217.123 ms
```

**Observation :**
- **Hop 5** : Traversée de l'Atlantique (+70ms)
- **Hop 9** : Traversée du Pacifique (+90ms)
- **Latence finale ~217ms** : Normal pour Europe→Japon

#### Exemple 5 : Utiliser TCP SYN au lieu d'UDP

Certains firewalls bloquent UDP mais laissent passer TCP. On peut utiliser TCP SYN :

```bash
# Linux : traceroute TCP vers port 443
sudo traceroute -T -p 443 example.com

# Alternative : utiliser tcptraceroute
sudo tcptraceroute example.com 443
```

**Avantage :** Contourne les firewalls qui bloquent UDP/ICMP mais autorisent HTTPS.

### Interpréter les résultats

#### Pattern normal

```
Latence croissante progressive :
Hop 1 :   1 ms    (local)
Hop 2 :   8 ms    (+7ms)
Hop 3 :  12 ms    (+4ms)
Hop 4 :  15 ms    (+3ms)
Hop 5 :  18 ms    (+3ms)
```

#### Pattern problématique - Augmentation brutale

```
Hop 4 :  15 ms
Hop 5 : 156 ms   ← Problème ici ! (+141ms)
Hop 6 : 158 ms
```
**Diagnostic :** Lien saturé ou équipement surchargé au hop 5

#### Pattern problématique - * * *

```
Hop 5 : * * *
```

**Trois cas possibles :**

1. **Firewall/ACL bloque ICMP Time Exceeded**
   ```
   Hop 4 : OK
   Hop 5 : * * *
   Hop 6 : * * *
   Hop 7 : OK  ← Si on voit des réponses après, c'est juste ICMP bloqué
   ```

2. **Routeur configuré pour ne pas répondre**
   - Certains routeurs ignorent délibérément les ICMP
   - Le trafic passe quand même

3. **Véritable blocage**
   ```
   Hop 4 : OK
   Hop 5 : * * *
   Hop 6 : * * *
   ...tous les suivants : * * *
   ```
   **Diagnostic :** Blocage réel après hop 4

#### Pattern problématique - Latence variable

```
Hop 5 : 25 ms  180 ms  28 ms
```
**Diagnostic :** Jitter important, lien instable ou surchargé

### Cas d'usage typiques

#### 1. Identifier où un paquet est bloqué

```bash
# Site inaccessible, où est le problème ?
traceroute unreachable-site.com

# Si blocage au hop 3 :
# → Problème entre hop 2 et hop 3
# → Contacter l'admin du réseau du hop 2
```

#### 2. Comparer deux chemins réseau

```bash
# Chemin 1 : via FAI A
traceroute -I example.com

# Chemin 2 : via VPN
# (sur une autre interface)
traceroute -i tun0 example.com

# Comparer les latences et chemins
```

#### 3. Vérifier le routage asymétrique

```bash
# Traceroute aller
traceroute server.example.com

# Traceroute retour (depuis le serveur)
ssh server.example.com "traceroute $(curl -s ifconfig.me)"

# Si les chemins diffèrent : routage asymétrique
```

### Limitations

#### ❌ Limitation 1 : Résultats incomplets

Beaucoup d'équipements ne répondent pas aux paquets traceroute (par sécurité).

#### ❌ Limitation 2 : Load balancing

```
# Avec load balancing, on peut voir des chemins différents
traceroute google.com
# Hop 5 : router-a.google.com
traceroute google.com
# Hop 5 : router-b.google.com  ← Chemin différent !
```

#### ❌ Limitation 3 : Ne montre qu'un snapshot

Le chemin peut changer dynamiquement (routage BGP, failover).

---

## netstat - Statistiques réseau et connexions

### Qu'est-ce que netstat ?

`netstat` (Network Statistics) est un outil historique pour :
- Afficher les **connexions actives** (TCP/UDP)
- Voir les **ports en écoute**
- Afficher les **statistiques** de protocoles
- Consulter la **table de routage**
- Voir les **statistiques d'interfaces**

**Note :** `netstat` est considéré comme obsolète sur Linux moderne, remplacé par `ss` (plus rapide et plus complet). Mais il reste largement utilisé et disponible sur Windows/macOS.

### Syntaxe de base

```bash
netstat [options]
```

### Options principales

| Option | Description | Exemple |
|--------|-------------|---------|
| `-a` | Afficher toutes les connexions et ports en écoute | `netstat -a` |
| `-t` | Afficher uniquement TCP | `netstat -t` |
| `-u` | Afficher uniquement UDP | `netstat -u` |
| `-n` | Afficher adresses numériques (pas de résolution DNS) | `netstat -n` |
| `-l` | Afficher uniquement les sockets en écoute | `netstat -l` |
| `-p` | Afficher le PID et nom du processus | `netstat -p` (Linux, nécessite root) |
| `-r` | Afficher la table de routage | `netstat -r` |
| `-s` | Afficher les statistiques par protocole | `netstat -s` |
| `-i` | Afficher les statistiques d'interfaces | `netstat -i` |
| `-c` | Rafraîchir en continu | `netstat -c` |

### Exemples pratiques

#### Exemple 1 : Voir toutes les connexions TCP actives

```bash
netstat -ant
```
**Explication :** `-a` (all), `-n` (numérique), `-t` (TCP)

**Sortie :**
```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN
tcp        0      0 192.168.1.100:443       203.0.113.50:52341      ESTABLISHED
tcp        0      0 192.168.1.100:52342     142.250.185.46:443      ESTABLISHED
tcp        0    328 192.168.1.100:22        198.51.100.20:54321     ESTABLISHED
tcp6       0      0 :::80                   :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
```

**Interprétation :**

```
tcp  0  0  0.0.0.0:22  0.0.0.0:*  LISTEN
```
- **Proto:** TCP
- **Recv-Q:** 0 octets en attente de lecture (queue de réception)
- **Send-Q:** 0 octets en attente d'envoi (queue d'émission)
- **Local Address:** `0.0.0.0:22` = Écoute sur toutes les interfaces, port 22 (SSH)
- **Foreign Address:** `0.0.0.0:*` = Attend des connexions de n'importe où
- **State:** `LISTEN` = En écoute

```
tcp  0  0  192.168.1.100:443  203.0.113.50:52341  ESTABLISHED
```
- **Local:** Mon serveur `192.168.1.100:443` (HTTPS)
- **Foreign:** Client `203.0.113.50:52341`
- **State:** `ESTABLISHED` = Connexion active

```
tcp  0  328  192.168.1.100:22  198.51.100.20:54321  ESTABLISHED
```
- **Send-Q: 328** = 328 octets en attente d'envoi
- **Attention :** Si Send-Q reste élevé, possible problème réseau ou client lent

#### Exemple 2 : Voir les ports en écoute avec les processus

```bash
# Linux (nécessite root)
sudo netstat -tulpn
```
**Explication :** `-t` (TCP), `-u` (UDP), `-l` (listening), `-p` (process), `-n` (numérique)

**Sortie :**
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address   Foreign Address   State      PID/Program name
tcp        0      0 0.0.0.0:22      0.0.0.0:*         LISTEN     1045/sshd
tcp        0      0 127.0.0.1:3306  0.0.0.0:*         LISTEN     1523/mysqld
tcp        0      0 0.0.0.0:80      0.0.0.0:*         LISTEN     2341/nginx
tcp        0      0 0.0.0.0:443     0.0.0.0:*         LISTEN     2341/nginx
tcp6       0      0 :::80           :::*              LISTEN     2341/nginx
tcp6       0      0 :::22           :::*              LISTEN     1045/sshd
udp        0      0 0.0.0.0:68      0.0.0.0:*                    945/dhclient
udp        0      0 0.0.0.0:53      0.0.0.0:*                    1123/dnsmasq
```

**Interprétation :**
- **Port 22 (SSH)** : Processus `sshd` (PID 1045) écoute sur toutes les interfaces
- **Port 3306 (MySQL)** : `mysqld` écoute uniquement sur `127.0.0.1` (localhost)
  - ✅ Sécurité : Base de données non exposée à l'extérieur
- **Port 80/443 (HTTP/HTTPS)** : `nginx` écoute sur IPv4 et IPv6
- **Port 68 (DHCP client)** : `dhclient` reçoit les baux DHCP
- **Port 53 (DNS)** : `dnsmasq` agit comme serveur DNS local

**Cas d'usage :** Identifier quel processus écoute sur un port spécifique.

#### Exemple 3 : Trouver quel processus utilise un port

```bash
# Linux
sudo netstat -tulpn | grep :8080

# Sortie
tcp  0  0  0.0.0.0:8080  0.0.0.0:*  LISTEN  3456/java
```

**Interprétation :** Le port 8080 est utilisé par un processus Java (PID 3456).

**Windows :**
```cmd
netstat -ano | findstr :8080

# Sortie
TCP  0.0.0.0:8080  0.0.0.0:0  LISTENING  3456

# Trouver le processus
tasklist /FI "PID eq 3456"
```

#### Exemple 4 : Compter les connexions par état

```bash
netstat -ant | awk '{print $6}' | sort | uniq -c | sort -rn
```

**Sortie :**
```
    156 ESTABLISHED
     45 TIME_WAIT
     12 LISTEN
      8 SYN_SENT
      3 CLOSE_WAIT
      2 FIN_WAIT1
      1 FIN_WAIT2
```

**Interprétation :**
- **156 ESTABLISHED** : 156 connexions actives
- **45 TIME_WAIT** : Connexions fermées en attente de nettoyage (normal)
- **8 SYN_SENT** : 8 tentatives de connexion en cours
- **3 CLOSE_WAIT** : Application n'a pas fermé proprement (possible fuite)

**Alerte :** Si beaucoup de `CLOSE_WAIT`, possible problème de fermeture de connexions dans le code applicatif.

#### Exemple 5 : Afficher la table de routage

```bash
netstat -rn
```

**Sortie (Linux) :**
```
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG        0 0          0 eth0
10.0.0.0        10.0.0.1        255.255.255.0   UG        0 0          0 tun0
192.168.1.0     0.0.0.0         255.255.255.0   U         0 0          0 eth0
```

**Interprétation :**
- **Ligne 1** : Route par défaut (`0.0.0.0/0`) via passerelle `192.168.1.1`
- **Ligne 2** : Réseau `10.0.0.0/24` routé via VPN (`tun0`)
- **Ligne 3** : Réseau local `192.168.1.0/24` directement accessible

#### Exemple 6 : Statistiques par protocole

```bash
netstat -s
```

**Sortie (extrait) :**
```
Ip:
    123456 total packets received
    0 forwarded
    0 incoming packets discarded
    123450 incoming packets delivered
    98765 requests sent out

Tcp:
    45678 active connection openings
    12345 passive connection openings
    234 failed connection attempts
    567 connection resets received
    12 connections established
    987654 segments received
    876543 segments sent out
    4321 segments retransmitted
    234 bad segments received

Udp:
    23456 packets received
    123 packets to unknown port received
    0 packet receive errors
    34567 packets sent
```

**Utilité :**
- **Segments retransmitted** : Si ce nombre augmente rapidement, problème réseau
- **Failed connection attempts** : Peut indiquer un firewall ou un service down
- **Packets to unknown port** : Possible scan de ports

### États TCP à connaître

| État | Description |
|------|-------------|
| **LISTEN** | Socket en écoute, attend des connexions |
| **SYN_SENT** | Tentative de connexion en cours (SYN envoyé) |
| **SYN_RECEIVED** | SYN reçu, attente de l'ACK final |
| **ESTABLISHED** | Connexion établie, données peuvent être échangées |
| **FIN_WAIT1** | Fermeture initiée localement, attente d'ACK |
| **FIN_WAIT2** | ACK reçu, attente du FIN distant |
| **CLOSE_WAIT** | Fermeture distante reçue, attente fermeture locale |
| **CLOSING** | Les deux côtés ferment simultanément |
| **LAST_ACK** | Attente de l'ACK final avant fermeture |
| **TIME_WAIT** | Connexion fermée, attente de sécurité (2 MSL) |
| **CLOSED** | Connexion n'existe plus |

### Cas d'usage typiques

```bash
# 1. Vérifier si un port est ouvert localement
netstat -an | grep :3000
# Si pas de résultat : port fermé

# 2. Voir combien de connexions actives vers un serveur spécifique
netstat -an | grep 203.0.113.50 | grep ESTABLISHED | wc -l

# 3. Identifier une fuite de connexions (trop de CLOSE_WAIT)
netstat -an | grep CLOSE_WAIT | wc -l

# 4. Surveiller l'activité réseau en temps réel
netstat -c
```

---

## ss - Socket Statistics (remplaçant moderne de netstat)

### Qu'est-ce que ss ?

`ss` (Socket Statistics) est le remplacement moderne de `netstat` sur Linux. Il est :
- **Plus rapide** (utilise netlink au lieu de /proc)
- **Plus complet** (plus d'options de filtrage)
- **Mieux maintenu** (développement actif)

### Syntaxe de base

```bash
ss [options] [filter]
```

### Options principales

| Option | Description |
|--------|-------------|
| `-t` | Afficher TCP |
| `-u` | Afficher UDP |
| `-a` | Afficher tous (listening + established) |
| `-l` | Afficher uniquement listening |
| `-n` | Numérique (pas de résolution) |
| `-p` | Afficher le processus |
| `-s` | Afficher statistiques résumées |
| `-i` | Afficher informations TCP internes |
| `-e` | Afficher informations étendues |
| `-m` | Afficher informations mémoire socket |
| `-o` | Afficher informations timer |

### Exemples pratiques

#### Exemple 1 : Voir toutes les connexions TCP

```bash
ss -tan
```

**Sortie :**
```
State      Recv-Q  Send-Q   Local Address:Port    Peer Address:Port
LISTEN     0       128            0.0.0.0:22             0.0.0.0:*
LISTEN     0       128          127.0.0.1:3306           0.0.0.0:*
LISTEN     0       128            0.0.0.0:443            0.0.0.0:*
ESTAB      0       0        192.168.1.100:443     203.0.113.50:52341
ESTAB      0       0        192.168.1.100:52342  142.250.185.46:443
ESTAB      0       328      192.168.1.100:22     198.51.100.20:54321
```

**Avantage vs netstat :** Beaucoup plus rapide sur les serveurs avec des milliers de connexions.

#### Exemple 2 : Ports en écoute avec processus

```bash
sudo ss -tulpn
```

**Sortie :**
```
Netid  State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port  Process
tcp    LISTEN  0       128           0.0.0.0:22          0.0.0.0:*      users:(("sshd",pid=1045,fd=3))
tcp    LISTEN  0       128         127.0.0.1:3306        0.0.0.0:*      users:(("mysqld",pid=1523,fd=10))
tcp    LISTEN  0       128           0.0.0.0:80          0.0.0.0:*      users:(("nginx",pid=2341,fd=6))
tcp    LISTEN  0       128           0.0.0.0:443         0.0.0.0:*      users:(("nginx",pid=2341,fd=7))
udp    UNCONN  0       0             0.0.0.0:68          0.0.0.0:*      users:(("dhclient",pid=945,fd=6))
```

**Lecture :**
- **Netid :** Protocole (tcp/udp)
- **State :** LISTEN pour serveurs, ESTAB pour connexions
- **Recv-Q/Send-Q :** Octets dans les queues
- **Process :** Nom du processus, PID, et file descriptor

#### Exemple 3 : Filtrer par état de connexion

```bash
# Voir uniquement les connexions établies
ss -tan state established

# Voir uniquement les sockets en écoute
ss -tan state listening

# Voir les connexions en TIME_WAIT
ss -tan state time-wait
```

**Sortie (established) :**
```
State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port
ESTAB   0       0       192.168.1.100:443    203.0.113.50:52341
ESTAB   0       0       192.168.1.100:52342  142.250.185.46:443
```

#### Exemple 4 : Filtrer par adresse ou port

```bash
# Connexions vers le port 443
ss -tan '( dport = :443 or sport = :443 )'

# Connexions depuis une IP spécifique
ss -tan dst 203.0.113.50

# Connexions vers un subnet
ss -tan dst 192.168.1.0/24
```

**Sortie :**
```
State  Recv-Q  Send-Q  Local Address:Port   Peer Address:Port
ESTAB  0       0       192.168.1.100:52342  142.250.185.46:443
ESTAB  0       0       192.168.1.100:443    203.0.113.50:52341
```

#### Exemple 5 : Informations TCP détaillées

```bash
ss -tinop
```
**Options :** `-i` (TCP info), `-o` (timer info), `-p` (process)

**Sortie :**
```
State  Recv-Q  Send-Q  Local Address:Port   Peer Address:Port   Process
ESTAB  0       0       192.168.1.100:443    203.0.113.50:52341  users:(("nginx",pid=2341,fd=12))
         cubic wscale:7,7 rto:204 rtt:3.5/1.75 ato:40 mss:1448 pmtu:1500 rcvmss:536
         advmss:1448 cwnd:10 bytes_sent:12345 bytes_retrans:0 bytes_acked:12345
         segs_out:89 segs_in:78 data_segs_out:67 data_segs_in:56 send 33.2Mbps
         lastsnd:1234 lastrcv:1234 lastack:1234 pacing_rate 66.4Mbps delivery_rate 28.5Mbps
         busy:2000ms unacked:0 retrans:0/0 lost:0 sacked:0 fackets:0 reordering:3
         rcv_space:14600 rcv_ssthresh:64088 minrtt:2.5
```

**Informations clés :**
- **cubic** : Algorithme de contrôle de congestion utilisé
- **rtt:3.5** : Round Trip Time de 3.5ms
- **cwnd:10** : Congestion window de 10 segments
- **bytes_sent:** 12345 octets envoyés
- **bytes_retrans:0** : 0 octets retransmis ✅ (bonne connexion)
- **send 33.2Mbps** : Débit théorique
- **minrtt:2.5** : RTT minimum observé

**Utilité :** Diagnostiquer les performances TCP fines.

#### Exemple 6 : Compter les connexions par état

```bash
ss -tan | awk '{print $1}' | sort | uniq -c | sort -rn
```

**Sortie :**
```
    156 ESTAB
     45 TIME-WAIT
     12 LISTEN
      8 SYN-SENT
      3 CLOSE-WAIT
```

### Filtres avancés

`ss` supporte des expressions de filtrage puissantes :

```bash
# Port source ou destination 80 OU 443
ss -tan '( sport = :80 or sport = :443 )'

# État ESTABLISHED ET port 443
ss -tan state established '( dport = :443 )'

# Connexions avec plus de 100 octets en Send-Q
ss -tan 'sendq > 100'

# Connexions vers un range de ports
ss -tan '( dport >= :8000 and dport <= :9000 )'
```

### Cas d'usage typiques

```bash
# 1. Trouver toutes les connexions vers un serveur web
ss -tan state established '( dport = :80 or dport = :443 )'

# 2. Compter les connexions par IP cliente
ss -tan state established | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn

# 3. Identifier les connexions avec retransmissions
ss -tinop | grep retrans

# 4. Surveiller l'utilisation des sockets en continu
watch -n 1 'ss -s'
```

---

## nslookup - Requêtes DNS simples

### Qu'est-ce que nslookup ?

`nslookup` (Name Server Lookup) est un outil interactif pour interroger les serveurs DNS. Il permet de :
- Résoudre un nom de domaine en adresse IP
- Faire la résolution inverse (IP → nom)
- Interroger des enregistrements DNS spécifiques
- Changer de serveur DNS pour les requêtes

**Note :** `nslookup` est considéré comme obsolète sur Linux (remplacé par `dig` et `host`), mais reste largement utilisé sur Windows et dans les habitudes.

### Syntaxe de base

```bash
# Mode non-interactif (une requête)
nslookup [nom_domaine] [serveur_dns]

# Mode interactif
nslookup
```

### Exemples pratiques

#### Exemple 1 : Résolution simple

```bash
nslookup google.com
```

**Sortie :**
```
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.185.46
Name:   google.com
Address: 2a00:1450:4007:80f::200e
```

**Interprétation :**
- **Server: 192.168.1.1** : Serveur DNS utilisé (généralement votre box/routeur)
- **Non-authoritative answer** : Réponse depuis un cache, pas depuis le serveur DNS autoritaire
- **Address: 142.250.185.46** : Adresse IPv4 de google.com
- **Address: 2a00:...** : Adresse IPv6 de google.com

#### Exemple 2 : Utiliser un serveur DNS spécifique

```bash
# Utiliser Google DNS (8.8.8.8)
nslookup google.com 8.8.8.8
```

**Sortie :**
```
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.185.46
```

**Cas d'usage :** Vérifier si le problème vient de votre DNS local.

#### Exemple 3 : Résolution inverse (IP → nom)

```bash
nslookup 8.8.8.8
```

**Sortie :**
```
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
8.8.8.8.in-addr.arpa    name = dns.google.
```

**Interprétation :** L'IP `8.8.8.8` correspond au nom `dns.google.`

#### Exemple 4 : Interroger un type d'enregistrement spécifique

```bash
# Enregistrements MX (Mail eXchange)
nslookup -type=MX google.com

# Enregistrements NS (Name Server)
nslookup -type=NS google.com

# Enregistrement AAAA (IPv6)
nslookup -type=AAAA google.com

# Enregistrement TXT
nslookup -type=TXT google.com
```

**Sortie (MX) :**
```
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
google.com      mail exchanger = 10 smtp.google.com.
```

**Lecture :** Les emails pour `@google.com` doivent être envoyés à `smtp.google.com` (priorité 10).

#### Exemple 5 : Mode interactif

```bash
nslookup
> server 8.8.8.8           # Changer de serveur DNS
> set type=MX             # Choisir le type d'enregistrement
> google.com              # Faire la requête
> set type=A              # Revenir aux enregistrements A
> example.com
> exit                    # Quitter
```

### Cas d'usage typiques

```bash
# 1. Vérifier que le DNS fonctionne
nslookup google.com
# Si timeout : problème DNS

# 2. Comparer deux serveurs DNS
nslookup example.com 8.8.8.8
nslookup example.com 1.1.1.1
# Si résultats différents : propagation DNS en cours ou problème de cache

# 3. Diagnostiquer un problème d'email (MX)
nslookup -type=MX example.com

# 4. Vérifier la propagation DNS après un changement
nslookup example.com
# Répéter toutes les 10 min jusqu'à voir la nouvelle IP
```

### Limitations

- **Moins détaillé que dig** (pas de TTL affiché clairement)
- **Format de sortie variable** selon les systèmes
- **Obsolète sur Linux** (préférer `dig` ou `host`)

---

## dig - Requêtes DNS avancées

### Qu'est-ce que dig ?

`dig` (Domain Information Groper) est l'outil DNS le plus complet pour :
- Interrogations DNS détaillées
- Debugging de configurations DNS
- Analyse de la résolution DNS
- Tracer les requêtes récursives

**Avantage vs nslookup :** Sortie structurée, plus d'informations (TTL, flags), meilleur pour le scripting.

### Syntaxe de base

```bash
dig [@serveur] [nom] [type]
```

### Exemples pratiques

#### Exemple 1 : Requête DNS simple

```bash
dig google.com
```

**Sortie :**
```
; <<>> DiG 9.18.1 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             299     IN      A       142.250.185.46

;; Query time: 12 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sat Dec 07 10:30:45 CET 2025
;; MSG SIZE  rcvd: 55
```

**Décortiquons la sortie :**

**HEADER :**
```
opcode: QUERY        → Type de requête
status: NOERROR      → Requête réussie
flags: qr rd ra      → qr=réponse, rd=récursion demandée, ra=récursion disponible
```

**QUESTION SECTION :**
```
;google.com.    IN      A
```
- On a demandé l'enregistrement A (IPv4) de `google.com`

**ANSWER SECTION :**
```
google.com.     299     IN      A       142.250.185.46
```
- **google.com.** : Nom de domaine (point final = FQDN)
- **299** : TTL (Time To Live) en secondes
- **IN** : Classe Internet
- **A** : Type d'enregistrement (adresse IPv4)
- **142.250.185.46** : Résultat

**Statistiques :**
```
Query time: 12 msec          → Temps de réponse DNS
SERVER: 192.168.1.1#53       → Serveur DNS utilisé
```

#### Exemple 2 : Requête vers un serveur DNS spécifique

```bash
# Utiliser Google DNS
dig @8.8.8.8 google.com

# Utiliser Cloudflare DNS
dig @1.1.1.1 google.com

# Utiliser un DNS interne
dig @10.0.1.10 internal-server.local
```

#### Exemple 3 : Interroger différents types d'enregistrements

```bash
# Enregistrement A (IPv4)
dig google.com A

# Enregistrement AAAA (IPv6)
dig google.com AAAA

# Enregistrements MX (Mail)
dig google.com MX

# Enregistrements NS (Name Servers)
dig google.com NS

# Enregistrement TXT
dig google.com TXT

# Enregistrement SOA (Start of Authority)
dig google.com SOA

# Enregistrement CNAME (alias)
dig www.example.com CNAME

# TOUS les enregistrements
dig google.com ANY
```

**Sortie (MX) :**
```
;; ANSWER SECTION:
google.com.             299     IN      MX      10 smtp.google.com.

;; ADDITIONAL SECTION:
smtp.google.com.        299     IN      A       142.251.16.27
```

**Lecture :**
- Emails vers `@google.com` → serveur `smtp.google.com` (priorité 10)
- Section ADDITIONAL donne directement l'IP de `smtp.google.com`

#### Exemple 4 : Sortie courte (pour scripts)

```bash
# Afficher uniquement la réponse
dig +short google.com
```

**Sortie :**
```
142.250.185.46
```

**Cas d'usage :**
```bash
# Récupérer l'IP dans une variable
IP=$(dig +short google.com)
echo "IP de Google : $IP"

# Vérifier si un domaine existe
if dig +short example.com | grep -q '^[0-9]'; then
    echo "Domaine résolu"
else
    echo "Domaine non trouvé"
fi
```

#### Exemple 5 : Tracer la résolution DNS récursive

```bash
dig +trace google.com
```

**Sortie (abrégée) :**
```
; <<>> DiG 9.18.1 <<>> +trace google.com
;; global options: +cmd
.                       518400  IN      NS      a.root-servers.net.
.                       518400  IN      NS      b.root-servers.net.
...
;; Received 1097 bytes from 192.168.1.1#53(192.168.1.1) in 3 ms

com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
...
;; Received 1173 bytes from 199.7.91.13#53(d.root-servers.net) in 15 ms

google.com.             172800  IN      NS      ns1.google.com.
google.com.             172800  IN      NS      ns2.google.com.
...
;; Received 759 bytes from 192.12.94.30#53(e.gtld-servers.net) in 85 ms

google.com.             300     IN      A       142.250.185.46
;; Received 55 bytes from 216.239.34.10#53(ns2.google.com) in 12 ms
```

**Interprétation :**
1. **Étape 1** : Interrogation des root servers (`.`)
2. **Étape 2** : Les root servers répondent avec les serveurs `.com`
3. **Étape 3** : Interrogation des serveurs `.com` pour `google.com`
4. **Étape 4** : Les serveurs `.com` donnent les NS de `google.com`
5. **Étape 5** : Interrogation finale des NS de `google.com`
6. **Résultat** : `142.250.185.46`

**Utilité :** Comprendre le cheminement d'une résolution DNS complète.

#### Exemple 6 : Requête inverse (PTR)

```bash
# Résolution inverse classique
dig -x 8.8.8.8

# Équivalent à :
dig PTR 8.8.8.8.in-addr.arpa
```

**Sortie :**
```
;; ANSWER SECTION:
8.8.8.8.in-addr.arpa.   86400   IN      PTR     dns.google.
```

#### Exemple 7 : Vérifier la propagation DNS

```bash
# Interroger plusieurs serveurs DNS publics
for dns in 8.8.8.8 1.1.1.1 9.9.9.9; do
    echo "DNS $dns:"
    dig @$dns +short example.com
done
```

**Sortie :**
```
DNS 8.8.8.8:
93.184.216.34
DNS 1.1.1.1:
93.184.216.34
DNS 9.9.9.9:
93.184.216.34
```

**Interprétation :** Si les IP diffèrent, la propagation DNS est en cours.

#### Exemple 8 : Afficher uniquement certaines sections

```bash
# Uniquement la section ANSWER
dig google.com +noall +answer

# Sortie
google.com.             299     IN      A       142.250.185.46

# ANSWER + statistiques
dig google.com +noall +answer +stats

# Sortie
google.com.             299     IN      A       142.250.185.46
;; Query time: 12 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sat Dec 07 10:45:30 CET 2025
;; MSG SIZE  rcvd: 55
```

### Options utiles

| Option | Description |
|--------|-------------|
| `+short` | Affichage minimal (uniquement la réponse) |
| `+trace` | Tracer la résolution récursive complète |
| `+noall +answer` | N'afficher que la section ANSWER |
| `+stats` | Afficher les statistiques de requête |
| `+tcp` | Utiliser TCP au lieu d'UDP |
| `+time=N` | Timeout de N secondes |
| `+tries=N` | Nombre de tentatives |
| `@server` | Utiliser un serveur DNS spécifique |

### Cas d'usage typiques

```bash
# 1. Diagnostiquer un problème DNS
dig example.com +trace
# Voir exactement où la résolution échoue

# 2. Vérifier le TTL d'un enregistrement
dig example.com +noall +answer
# Pour savoir quand le cache DNS expirera

# 3. Comparer les serveurs DNS
dig @8.8.8.8 example.com +short
dig @1.1.1.1 example.com +short

# 4. Tester les enregistrements MX pour email
dig example.com MX +short

# 5. Vérifier les enregistrements SPF (anti-spam)
dig example.com TXT | grep spf

# 6. Automatisation/scripting
IP=$(dig +short example.com | head -1)
echo "Connexion à $IP..."
ssh user@$IP
```

---

## Autres outils utiles

### host - Alternative simple à dig

```bash
# Résolution simple
host google.com

# Sortie
google.com has address 142.250.185.46
google.com has IPv6 address 2a00:1450:4007:80f::200e
google.com mail is handled by 10 smtp.google.com.

# Résolution inverse
host 8.8.8.8

# Type spécifique
host -t MX google.com
```

**Utilité :** Plus simple que `dig`, plus propre que `nslookup`.

### whois - Informations sur un domaine

```bash
whois example.com
```

**Sortie (extrait) :**
```
Domain Name: EXAMPLE.COM
Registrar: IANA
Creation Date: 1995-08-14T04:00:00Z
Registry Expiry Date: 2024-08-13T04:00:00Z
Name Server: a.iana-servers.net
Name Server: b.iana-servers.net
```

**Utilité :** Connaître le propriétaire, les dates d'expiration, les name servers.

### curl / wget - Tester HTTP/HTTPS

```bash
# Test HTTP simple
curl -I http://example.com

# Sortie
HTTP/1.1 200 OK
Content-Type: text/html
Server: nginx
Date: Sat, 07 Dec 2025 10:50:00 GMT

# Test avec temps de réponse
curl -w "\nTime: %{time_total}s\n" -o /dev/null -s http://example.com

# Test HTTPS avec détails TLS
curl -v https://example.com 2>&1 | grep -A 10 'SSL connection'
```

### telnet / nc - Tester la connectivité TCP

```bash
# Tester si le port 80 est ouvert
telnet example.com 80
# ou
nc -zv example.com 80

# Tester SMTP
telnet mail.example.com 25

# Tester PostgreSQL
nc -zv db.example.com 5432
```

---

## Tableau récapitulatif : Quel outil pour quel diagnostic ?

| Besoin | Outil | Commande exemple |
|--------|-------|------------------|
| **Tester connectivité de base** | `ping` | `ping 8.8.8.8` |
| **Mesurer latence** | `ping` | `ping -c 10 server.com` |
| **Voir le chemin réseau** | `traceroute` | `traceroute google.com` |
| **Identifier un blocage** | `traceroute` | `traceroute -T -p 443 site.com` |
| **Voir connexions actives** | `ss` / `netstat` | `ss -tan state established` |
| **Trouver processus sur un port** | `ss` / `netstat` | `sudo ss -tulpn | grep :8080` |
| **Vérifier ports ouverts** | `ss` / `netstat` | `ss -tln` |
| **Résoudre un nom DNS** | `dig` / `nslookup` | `dig google.com` |
| **Debug DNS** | `dig` | `dig +trace example.com` |
| **Tester un port applicatif** | `telnet` / `nc` | `nc -zv server.com 443` |
| **Tester HTTP/HTTPS** | `curl` | `curl -I https://example.com` |
| **Infos sur un domaine** | `whois` | `whois example.com` |
| **Table de routage** | `ip route` / `netstat` | `ip route show` |
| **Cache ARP** | `ip neigh` / `arp` | `ip neigh show` |

---

## Workflow de diagnostic typique

Voici un workflow utilisant ces outils en séquence :

```bash
# 1. Vérifier la connectivité de base
ping -c 4 example.com
# ✅ OK : Passer à l'étape suivante
# ❌ KO : Aller à l'étape 1b

# 1b. Si ping échoue, vérifier la résolution DNS
nslookup example.com
dig example.com +short
# Si pas d'IP : Problème DNS
# Si IP OK mais ping KO : Problème réseau ou ICMP bloqué

# 2. Si ping OK, identifier le chemin réseau
traceroute example.com
# Observer où se trouve le goulot ou le blocage

# 3. Tester le port applicatif
telnet example.com 443
# ou
nc -zv example.com 443
# ✅ Connected : Port ouvert, service écoute
# ❌ Connection refused : Port fermé ou service down
# ❌ Timeout : Firewall bloque

# 4. Tester l'application
curl -v https://example.com
# Voir si HTTP/TLS fonctionne correctement

# 5. Vérifier les connexions locales
ss -tan state established '( dport = :443 )'
# Y a-t-il des connexions actives ?

# 6. Identifier le processus
sudo ss -tulpn | grep :443
# Quel processus écoute sur le port 443 ?
```

---

## Points clés à retenir

### 🎯 Les essentiels

1. **ping** : Premier réflexe pour tester la connectivité
2. **traceroute** : Identifier où un paquet bloque ou ralentit
3. **ss / netstat** : Voir l'état des connexions et ports
4. **dig** : Diagnostiquer tout problème DNS
5. **telnet / nc** : Tester l'ouverture d'un port TCP

### ✅ Bonnes pratiques

- Utilisez toujours l'option `-n` pour éviter les résolutions DNS inutiles
- Sur Linux, préférez `ss` à `netstat` (plus rapide)
- Préférez `dig` à `nslookup` (plus d'informations)
- Combinez les outils : ping + traceroute + dig donne une vue complète
- Documentez vos commandes et résultats pour référence future

### ❌ Erreurs à éviter

- Ne pas tester qu'avec `ping` (ICMP peut être bloqué)
- Oublier que `netstat` est lent sur des serveurs avec beaucoup de connexions
- Utiliser `traceroute` UDP sur un firewall qui bloque UDP (utiliser `-T`)
- Ne pas vérifier le DNS (beaucoup de problèmes viennent de là)

---

## Conclusion

Les outils en ligne de commande sont la base du diagnostic réseau. Leur maîtrise permet de :
- **Diagnostiquer rapidement** la plupart des problèmes
- **Comprendre** ce qui se passe au niveau réseau
- **Communiquer** efficacement avec les équipes (logs, outputs)
- **Scripter** des vérifications automatiques

Dans la section suivante, nous passerons à Wireshark, l'outil ultime pour l'analyse fine du trafic réseau paquet par paquet.

---

**Prochaine section :** 7.3 Analyse de trames avec Wireshark : principes

Nous découvrirons comment capturer et analyser le trafic réseau en détail pour identifier des problèmes invisibles aux outils de base.

⏭️ [Analyse de trames avec Wireshark : principes](/07-analyse-depannage/03-wireshark-principes.md)
