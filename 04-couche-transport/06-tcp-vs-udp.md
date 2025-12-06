🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.6 Comparaison TCP vs UDP : critères de choix

## Introduction

Choisir entre TCP et UDP est l'une des décisions architecturales les plus importantes lors de la conception d'une application réseau. Ce choix impacte directement :

- **Les performances** : latence, débit, utilisation CPU
- **La fiabilité** : garanties de livraison des données
- **La complexité** : du code et de la maintenance
- **L'expérience utilisateur** : réactivité, qualité de service
- **Les coûts** : infrastructure, bande passante, développement

Cette section fournit un cadre de décision structuré avec des critères objectifs, des exemples concrets et des arbres de décision pour vous aider à faire le bon choix pour votre application.

## Vue d'ensemble comparative

### Les deux philosophies

```
TCP : "La poste recommandée"
═══════════════════════════════

✓ Chaque paquet est numéroté
✓ Accusé de réception obligatoire
✓ Retransmission automatique si perte
✓ Livraison garantie dans l'ordre
✓ Contrôle de flux et de congestion

→ Fiable mais potentiellement lent


UDP : "La carte postale"
═══════════════════════════════

✓ Envoi immédiat sans préparation
✓ Pas d'accusé de réception
✓ Pas de retransmission
✓ Ordre non garanti
✓ Pas de contrôle

→ Rapide mais sans garantie
```

### Tableau comparatif de base

```
┌──────────────────────────────────────────────────────────────┐
│              TCP vs UDP : Caractéristiques                   │
├────────────────────┬────────────────────┬────────────────────┤
│ Caractéristique    │ TCP                │ UDP                │
├────────────────────┼────────────────────┼────────────────────┤
│ Type               │ Orienté connexion  │ Sans connexion     │
│ Fiabilité          │ Garantie           │ Aucune             │
│ Ordre              │ Préservé           │ Non garanti        │
│ Vitesse            │ Moyenne            │ Rapide             │
│ Overhead           │ Élevé (20+ octets) │ Faible (8 octets)  │
│ Contrôle flux      │ Oui                │ Non                │
│ Contrôle congestion│ Oui                │ Non                │
│ Broadcast/Multicast│ Non                │ Oui                │
│ Cas d'usage        │ Transferts fiables │ Temps réel         │
└────────────────────┴────────────────────┴────────────────────┘
```

## Critères de décision détaillés

### Critère 1 : Tolérance aux pertes de données

#### Questions à se poser

```
1. Quelle est la tolérance aux pertes ?
   ☐ Aucune perte acceptable → TCP
   ☐ Pertes de 1-5% acceptables → UDP possible
   ☐ Pertes >5% acceptables → UDP

2. Quel est l'impact d'une perte ?
   ☐ Corruption de données critiques → TCP
   ☐ Dégradation mineure de qualité → UDP possible
   ☐ Perte non perceptible → UDP

3. Les données sont-elles récupérables ?
   ☐ Non, données uniques → TCP
   ☐ Oui, données périodiques → UDP possible
```

#### Exemples concrets

**TCP obligatoire** :

```
Transfert de fichier (1 GB) :
────────────────────────────

Avec TCP :
  • 100% des données arrivent
  • Ordre préservé
  • Fichier identique à l'original
  ✓ Succès garanti

Avec UDP (1% perte) :
  • 10 MB de données perdues
  • Fichier corrompu
  • Inutilisable
  ✗ Échec total

Verdict : TCP OBLIGATOIRE
```

**UDP acceptable** :

```
Streaming audio (podcast) :
───────────────────────────

Avec TCP :
  • Retransmissions causent bufferisation
  • Latence variable (200-1000ms)
  • Coupures fréquentes
  ✗ Mauvaise expérience

Avec UDP (1% perte) :
  • 1 paquet perdu toutes les 2 secondes
  • Micro-coupure de 20ms imperceptible
  • Latence stable (50ms)
  ✓ Expérience fluide

Verdict : UDP RECOMMANDÉ
```

**Cas mixte** :

```
Jeu multijoueur :
─────────────────

Messages position (60/sec) :
  • Vieilles positions inutiles → UDP
  • 1% perte acceptable → UDP

Messages chat :
  • Aucune perte acceptable → TCP
  • Ordre important → TCP

Messages actions critiques :
  • Fiabilité nécessaire → UDP + ACK applicatif

Verdict : HYBRIDE (UDP + TCP)
```

### Critère 2 : Exigences de latence

#### Seuils de latence par application

```
┌──────────────────────────────────────────────────────────┐
│           Latence maximale tolérable                     │
├─────────────────────────┬────────────────┬───────────────┤
│ Type d'application      │ Latence max    │ Protocole     │
├─────────────────────────┼────────────────┼───────────────┤
│ Jeu FPS compétitif      │ < 50ms         │ UDP           │
│ Jeu action              │ < 100ms        │ UDP           │
│ VoIP                    │ < 150ms        │ UDP           │
│ Visioconférence         │ < 200ms        │ UDP           │
│ Streaming vidéo         │ < 500ms        │ UDP/TCP       │
│ Navigation web          │ < 1000ms       │ TCP           │
│ API REST                │ < 2000ms       │ TCP           │
│ Transfert fichiers      │ Pas critique   │ TCP           │
│ Email                   │ Pas critique   │ TCP           │
└─────────────────────────┴────────────────┴───────────────┘
```

#### Impact du handshake TCP

```
Comparaison temps de première réponse :
──────────────────────────────────────

Scénario : Client en France, Serveur en Californie
Latence réseau : 150ms (aller simple)

Avec UDP :
  t=0ms   : Client envoie requête
  t=150ms : Serveur reçoit
  t=150ms : Serveur envoie réponse
  t=300ms : Client reçoit réponse

  TOTAL : 300ms

Avec TCP :
  t=0ms   : Client envoie SYN
  t=150ms : Serveur reçoit, envoie SYN+ACK
  t=300ms : Client reçoit, envoie ACK + requête
  t=450ms : Serveur reçoit requête
  t=450ms : Serveur envoie réponse
  t=600ms : Client reçoit réponse

  TOTAL : 600ms (2× plus lent)
```

#### Calcul du budget latence

```python
def calculate_latency_budget(application_type):
    """
    Calculer si UDP est nécessaire selon budget latence
    """
    budgets = {
        'gaming_fps': {
            'max_total': 50,      # ms
            'render': 10,         # ms
            'processing': 5,      # ms
            'network': 35,        # ms restant pour réseau
            'protocol_overhead': {
                'tcp': 150,       # handshake
                'udp': 0
            }
        },
        'voip': {
            'max_total': 150,
            'encoding': 20,
            'processing': 10,
            'network': 120,
            'protocol_overhead': {
                'tcp': 150,
                'udp': 0
            }
        },
        'web_api': {
            'max_total': 2000,
            'processing': 100,
            'database': 50,
            'network': 1850,
            'protocol_overhead': {
                'tcp': 150,
                'udp': 0
            }
        }
    }

    app = budgets[application_type]
    network_budget = app['network']

    # TCP est-il viable ?
    tcp_overhead = app['protocol_overhead']['tcp']

    if tcp_overhead > network_budget * 0.5:
        return 'UDP', f"TCP overhead ({tcp_overhead}ms) > 50% du budget ({network_budget}ms)"
    else:
        return 'TCP', f"Budget suffisant pour TCP"

# Exemples
print(calculate_latency_budget('gaming_fps'))
# → ('UDP', 'TCP overhead (150ms) > 50% du budget (35ms)')

print(calculate_latency_budget('web_api'))
# → ('TCP', 'Budget suffisant pour TCP')
```

### Critère 3 : Volume et fréquence des messages

#### Analyse du trafic

```
┌──────────────────────────────────────────────────────────┐
│         Caractéristiques du trafic                       │
├────────────────────┬────────────────┬────────────────────┤
│ Fréquence          │ Taille message │ Recommandation     │
├────────────────────┼────────────────┼────────────────────┤
│ < 1/sec            │ N'importe      │ TCP (overhead OK)  │
│ 1-10/sec           │ < 100 octets   │ UDP (efficace)     │
│ 1-10/sec           │ > 1 KB         │ TCP (fiabilité)    │
│ 10-100/sec         │ < 100 octets   │ UDP (performant)   │
│ 10-100/sec         │ > 1 KB         │ TCP/UDP selon cas  │
│ > 100/sec          │ < 100 octets   │ UDP (overhead TCP) │
│ > 100/sec          │ > 1 KB         │ TCP sauf temps réel│
└────────────────────┴────────────────┴────────────────────┘
```

#### Calcul de l'overhead

```python
def calculate_protocol_overhead(
    messages_per_second,
    message_size_bytes,
    protocol='tcp'
):
    """
    Calculer l'overhead réseau selon le protocole
    """
    # Tailles en-têtes
    ip_header = 20
    tcp_header = 20
    udp_header = 8

    # Calcul pour TCP
    if protocol == 'tcp':
        # Chaque message
        message_overhead = ip_header + tcp_header
        data_bw = messages_per_second * message_size_bytes
        overhead_bw = messages_per_second * message_overhead

        # ACKs (environ 50% des messages)
        ack_overhead = messages_per_second * 0.5 * (ip_header + tcp_header)

        total_overhead = overhead_bw + ack_overhead
        total_bw = data_bw + total_overhead
        overhead_percent = (total_overhead / total_bw) * 100

    else:  # UDP
        message_overhead = ip_header + udp_header
        data_bw = messages_per_second * message_size_bytes
        overhead_bw = messages_per_second * message_overhead

        total_bw = data_bw + overhead_bw
        overhead_percent = (overhead_bw / total_bw) * 100

    return {
        'data_bandwidth_bps': data_bw * 8,
        'overhead_bandwidth_bps': (overhead_bw + (ack_overhead if protocol == 'tcp' else 0)) * 8,
        'total_bandwidth_bps': total_bw * 8,
        'overhead_percent': overhead_percent
    }

# Exemple 1 : Jeu (position joueur)
game_tcp = calculate_protocol_overhead(60, 50, 'tcp')
game_udp = calculate_protocol_overhead(60, 50, 'udp')

print("Jeu (60 updates/sec, 50 octets) :")
print(f"  TCP : {game_tcp['overhead_percent']:.1f}% overhead")
print(f"  UDP : {game_udp['overhead_percent']:.1f}% overhead")
# → TCP : 66.7% overhead, UDP : 35.9% overhead

# Exemple 2 : API REST (requêtes occasionnelles)
api_tcp = calculate_protocol_overhead(1, 1000, 'tcp')
api_udp = calculate_protocol_overhead(1, 1000, 'udp')

print("\nAPI REST (1 req/sec, 1000 octets) :")
print(f"  TCP : {api_tcp['overhead_percent']:.1f}% overhead")
print(f"  UDP : {api_udp['overhead_percent']:.1f}% overhead")
# → TCP : 5.7% overhead, UDP : 2.7% overhead
# Différence négligeable, TCP préférable pour fiabilité
```

### Critère 4 : Nature des données

#### Classification des données

```
┌──────────────────────────────────────────────────────────┐
│              Types de données                            │
├────────────────────┬─────────────────────────────────────┤
│ Type               │ Protocole recommandé                │
├────────────────────┼─────────────────────────────────────┤
│ DONNÉES CRITIQUES                                        │
├────────────────────┼─────────────────────────────────────┤
│ Transactions       │ TCP (OBLIGATOIRE)                   │
│ Documents          │ TCP (OBLIGATOIRE)                   │
│ Commandes          │ TCP (OBLIGATOIRE)                   │
│ Configuration      │ TCP (OBLIGATOIRE)                   │
│ Logs importants    │ TCP (recommandé)                    │
│                    │                                     │
│ DONNÉES ÉPHÉMÈRES                                        │
├────────────────────┼─────────────────────────────────────┤
│ Position temps réel│ UDP (recommandé)                    │
│ Métriques live     │ UDP (recommandé)                    │
│ Audio/Vidéo live   │ UDP (recommandé)                    │
│ État du jeu        │ UDP (recommandé)                    │
│                    │                                     │
│ DONNÉES PÉRIODIQUES                                      │
├────────────────────┼─────────────────────────────────────┤
│ Télémétrie IoT     │ UDP (efficace) ou TCP (fiable)      │
│ Heartbeats         │ UDP (léger) ou TCP (si connexion)   │
│ Métriques stockées │ TCP (préserver historique)          │
│                    │                                     │
│ DONNÉES HYBRIDES                                         │
├────────────────────┼─────────────────────────────────────┤
│ Gaming             │ UDP (position) + TCP (chat)         │
│ Collaboration      │ UDP (curseurs) + TCP (éditions)     │
│ Monitoring         │ UDP (métriques) + TCP (alertes)     │
└────────────────────┴─────────────────────────────────────┘
```

#### Test de vieillissement des données

```python
def data_freshness_analysis(data_type, lifetime_ms):
    """
    Déterminer si les données supportent la latence TCP
    """
    tcp_overhead_ms = 150  # Handshake + retrans potentielle

    if tcp_overhead_ms > lifetime_ms * 0.5:
        return {
            'protocol': 'UDP',
            'reason': f"Données obsolètes après {lifetime_ms}ms, TCP trop lent",
            'example': "Position joueur, frame vidéo"
        }
    else:
        return {
            'protocol': 'TCP',
            'reason': f"Données valides assez longtemps ({lifetime_ms}ms)",
            'example': "Message chat, commande"
        }

# Exemples
print(data_freshness_analysis('player_position', 16))  # 60 FPS
# → UDP (données obsolètes après 16ms)

print(data_freshness_analysis('chat_message', 5000))
# → TCP (données valides assez longtemps)
```

### Critère 5 : Besoin de séquençage

#### Importance de l'ordre

```
Questions :
───────────

1. L'ordre des messages est-il critique ?
   ☐ Oui, ordre strict requis → TCP
   ☐ Ordre souhaitable mais gérable → UDP + séquençage applicatif
   ☐ Ordre sans importance → UDP

2. Les messages sont-ils indépendants ?
   ☐ Chaque message autonome → UDP possible
   ☐ Messages liés → TCP recommandé

3. Que se passe-t-il si ordre incorrect ?
   ☐ Corruption de données → TCP
   ☐ Affichage bizarre temporaire → UDP acceptable
   ☐ Aucun impact → UDP
```

#### Exemples de dépendance à l'ordre

**Ordre critique** :

```
Commandes base de données :
───────────────────────────

1. BEGIN TRANSACTION
2. UPDATE accounts SET balance = 1000 WHERE id = 1
3. UPDATE accounts SET balance = 500 WHERE id = 2
4. COMMIT

Si ordre incorrect :
  • COMMIT avant UPDATE → données incohérentes
  • UPDATE avant BEGIN → erreur

Protocole : TCP OBLIGATOIRE
```

**Ordre non critique** :

```
Métriques système :
───────────────────

t=0s  : cpu_usage=45%
t=1s  : memory_usage=78%
t=2s  : disk_io=120MB/s
t=3s  : cpu_usage=52%

Si reçu dans ordre : 0, 2, 1, 3
  • Chaque métrique a timestamp
  • Réordonnancement possible côté visualisation
  • Ou ignoré si pas critique

Protocole : UDP ACCEPTABLE
```

### Critère 6 : Scalabilité requise

#### Nombre de connexions simultanées

```
┌──────────────────────────────────────────────────────────┐
│         Impact sur serveur                               │
├────────────────────┬────────────────┬────────────────────┤
│ Connexions simul.  │ TCP            │ UDP                │
├────────────────────┼────────────────┼────────────────────┤
│ < 100              │ ✓ Facile       │ ✓ Facile           │
│ 100 - 1,000        │ ✓ Gérable      │ ✓ Très facile      │
│ 1,000 - 10,000     │ ~ Difficile    │ ✓ Facile           │
│ 10,000 - 100,000   │ ✗ Très difficile│ ✓ Gérable         │
│ > 100,000          │ ✗ Extrême      │ ✓ Possible         │
└────────────────────┴────────────────┴────────────────────┘

Mémoire par connexion :
  TCP : ~150 KB (buffers + état)
  UDP : ~2 KB (minimal)

10,000 connexions :
  TCP : 1.5 GB de RAM
  UDP : 20 MB de RAM
```

#### Architecture de serveur

```python
# Serveur TCP : 1 thread/connexion (simple mais limité)
def tcp_server_threaded():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(('0.0.0.0', 8000))
    server.listen(100)

    while True:
        client, addr = server.accept()
        # Nouveau thread par client
        thread = threading.Thread(
            target=handle_client,
            args=(client,)
        )
        thread.start()

    # Limite pratique : ~5,000 threads
    # Au-delà : utiliser epoll/kqueue

# Serveur UDP : 1 thread suffit pour tous les clients
def udp_server_single_thread():
    server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    server.bind(('0.0.0.0', 8000))

    while True:
        data, addr = server.recvfrom(2048)
        # Traiter immédiatement
        response = process(data)
        server.sendto(response, addr)

    # Peut gérer 50,000+ clients facilement
```

### Critère 7 : Infrastructure et coûts

#### Bande passante

```
Coût bande passante (AWS, 0.09$/GB sortant) :
─────────────────────────────────────────────

Application : 1 million d'utilisateurs
Trafic : 100 KB/utilisateur/jour

Avec TCP (overhead 20%) :
  • Données : 100 GB/jour
  • Overhead : 20 GB/jour
  • Total : 120 GB/jour
  • Coût : 120 × 0.09 = 10.80$/jour
  • Coût annuel : 3,942$

Avec UDP (overhead 8%) :
  • Données : 100 GB/jour
  • Overhead : 8 GB/jour
  • Total : 108 GB/jour
  • Coût : 108 × 0.09 = 9.72$/jour
  • Coût annuel : 3,548$

Économie UDP : 394$/an (10%)
```

#### CPU et infrastructure

```
Serveur traitement de paquets :
───────────────────────────────

TCP :
  • Serveurs requis : 10 (CPU saturé)
  • Coût mensuel : 10 × 200$ = 2,000$

UDP :
  • Serveurs requis : 3 (CPU ~30%)
  • Coût mensuel : 3 × 200$ = 600$

Économie UDP : 1,400$/mois = 16,800$/an
```

## Arbres de décision

### Arbre principal de décision

```
                    Début
                      │
                      ▼
        ┌─────────────────────────┐
        │ Perte de données        │
        │ acceptable ?            │
        └─────────────────────────┘
                │           │
            Non │           │ Oui
                │           │
                ▼           ▼
              TCP     ┌─────────────────┐
                      │ Latence < 150ms │
                      │ critique ?      │
                      └─────────────────┘
                            │       │
                        Non │       │ Oui
                            │       │
                            ▼       ▼
                          TCP     UDP
```

### Arbre détaillé par cas d'usage

```
                Application réseau
                        │
        ────────────────┼────────────────
        │               │               │
        ▼               ▼               ▼
   Transfert       Temps réel     Requête/Réponse
   de données
        │               │               │
        ▼               ▼               ▼
    Fichiers        Gaming          Taille ?
    Documents       VoIP                │
    Email           Streaming       ────┴────
        │               │          │         │
        ▼               ▼          ▼         ▼
       TCP             UDP      <100B      >1KB
                                   │         │
                                   ▼         ▼
                               Fréquence?   TCP
                                   │
                               ────┴────
                              │         │
                              ▼         ▼
                           >10/sec   <10/sec
                              │         │
                              ▼         ▼
                             UDP       TCP
```

## Cas d'usage par domaine

### 1. Applications web

```
┌──────────────────────────────────────────────────────────┐
│              Applications Web                            │
├────────────────────┬─────────────────────────────────────┤
│ Composant          │ Protocole et justification          │
├────────────────────┼─────────────────────────────────────┤
│ API REST           │ TCP : Fiabilité critique            │
│ GraphQL            │ TCP : Requêtes complexes            │
│ WebSocket (data)   │ TCP : Ordre et fiabilité            │
│ Server-Sent Events │ TCP : Stream d'événements           │
│ Upload fichiers    │ TCP : Intégrité obligatoire         │
│ Download fichiers  │ TCP : Intégrité obligatoire         │
│                    │                                     │
│ Curseurs partagés  │ UDP : Temps réel, pertes OK         │
│ Notifications live │ UDP+TCP : Hybride selon criticité   │
└────────────────────┴─────────────────────────────────────┘
```

**Architecture typique** :

```
Application web collaborative (type Google Docs) :

┌────────────────────────────────────────────┐
│  Frontend (Navigateur)                     │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │ WebSocket TCP                        │  │
│  │ • Éditions de document               │  │
│  │ • Messages chat                      │  │
│  │ • Historique modifications           │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │ WebRTC DataChannel (UDP)             │  │
│  │ • Position curseurs                  │  │
│  │ • Sélections texte                   │  │
│  │ • Présence utilisateurs              │  │
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘

Justification :
  TCP : Modifications doivent être 100% fiables
  UDP : Curseurs obsolètes en 100ms, pertes OK
```

### 2. Gaming

```
┌──────────────────────────────────────────────────────────┐
│              Gaming multijoueur                          │
├────────────────────┬─────────────────────────────────────┤
│ Données            │ Protocole et justification          │
├────────────────────┼─────────────────────────────────────┤
│ Positions joueurs  │ UDP : 60 Hz, pertes acceptables     │
│ Animations         │ UDP : Temps réel                    │
│ Effets visuels     │ UDP : Cosmétique                    │
│ Son spatial        │ UDP : Pertes tolérables             │
│                    │                                     │
│ Actions critiques  │ UDP+ACK : Fiabilité custom          │
│ Inventaire         │ TCP : Données critiques             │
│ Chat               │ TCP : Ordre et fiabilité            │
│ Achats in-game     │ TCP : Transactions                  │
│ Matchmaking        │ TCP : Logique complexe              │
└────────────────────┴─────────────────────────────────────┘
```

**Exemple d'architecture hybride** :

```python
class GameClient:
    def __init__(self):
        # UDP pour gameplay
        self.udp_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_DGRAM
        )

        # TCP pour données critiques
        self.tcp_socket = socket.socket(
            socket.AF_INET,
            socket.SOCK_STREAM
        )
        self.tcp_socket.connect(('game-server.com', 7777))

    def send_position(self, x, y, angle):
        """Position : UDP (60 Hz)"""
        data = struct.pack('>fff', x, y, angle)
        self.udp_socket.sendto(data, ('game-server.com', 7778))

    def send_chat_message(self, message):
        """Chat : TCP (fiable)"""
        data = json.dumps({
            'type': 'chat',
            'message': message
        }).encode()
        self.tcp_socket.sendall(data)

    def send_shoot_action(self, target_id):
        """Action critique : UDP + ACK custom"""
        action_id = random.randint(0, 2**32)
        data = struct.pack('>I I', action_id, target_id)

        # Envoyer et attendre ACK
        for retry in range(3):
            self.udp_socket.sendto(data, ('game-server.com', 7778))

            # Attendre ACK avec timeout
            try:
                self.udp_socket.settimeout(0.1)
                ack_data, _ = self.udp_socket.recvfrom(64)
                ack_id = struct.unpack('>I', ack_data)[0]

                if ack_id == action_id:
                    return True  # Confirmé
            except socket.timeout:
                continue

        return False  # Échec après 3 tentatives
```

### 3. Streaming média

```
┌──────────────────────────────────────────────────────────┐
│              Streaming Audio/Vidéo                       │
├────────────────────┬─────────────────────────────────────┤
│ Type               │ Protocole et justification          │
├────────────────────┼─────────────────────────────────────┤
│ Live streaming     │ UDP (RTP) : Latence critique        │
│ Vidéoconférence    │ UDP (WebRTC) : Temps réel           │
│ VoIP               │ UDP (SIP/RTP) : Latence <150ms      │
│                    │                                     │
│ VOD (Netflix)      │ TCP (HTTP) : Buffering acceptable   │
│ Podcast            │ TCP : Fiabilité importante          │
│ Téléchargement     │ TCP : Intégrité complète            │
│                    │                                     │
│ Signalisation      │ TCP : Commandes fiables             │
│ Métadonnées        │ TCP : Informations critiques        │
└────────────────────┴─────────────────────────────────────┘
```

**Comparaison Live vs VOD** :

```
Streaming Live (match de football) :
────────────────────────────────────

Contraintes :
  • Latence max : 3-5 secondes
  • Synchronisation spectateurs
  • Pertes vidéo : acceptables (artefacts temporaires)

Protocole : UDP (RTP/RTMP)

Streaming VOD (série Netflix) :
────────────────────────────────

Contraintes :
  • Latence : 5-30 secondes OK (buffer)
  • Qualité : maximale (pas d'artefacts)
  • Téléchargement avance : possible

Protocole : TCP (HLS/DASH sur HTTP)
```

### 4. IoT et capteurs

```
┌──────────────────────────────────────────────────────────┐
│              Internet des Objets                         │
├────────────────────┬─────────────────────────────────────┤
│ Scénario           │ Protocole et justification          │
├────────────────────┼─────────────────────────────────────┤
│ Capteurs temp/hum  │ UDP : Économie batterie             │
│ GPS tracking       │ UDP : Données périodiques           │
│ Métriques système  │ UDP : Volume élevé                  │
│                    │                                     │
│ Alertes critiques  │ TCP : Garantie livraison            │
│ Commandes          │ TCP : Fiabilité obligatoire         │
│ Configuration      │ TCP : Intégrité données             │
│ Firmware update    │ TCP : Corruption inacceptable       │
└────────────────────┴─────────────────────────────────────┘
```

**Exemple : Station météo connectée** :

```python
class WeatherStation:
    def __init__(self):
        # UDP pour télémétrie régulière
        self.udp_sock = socket.socket(
            socket.AF_INET,
            socket.SOCK_DGRAM
        )

        # TCP pour commandes (créé à la demande)
        self.tcp_sock = None

    def send_telemetry(self):
        """
        Télémétrie toutes les 5 minutes : UDP
        Si perdu, prochaine mesure dans 5 min
        """
        data = {
            'station_id': self.id,
            'timestamp': int(time.time()),
            'temperature': read_temperature(),
            'humidity': read_humidity(),
            'pressure': read_pressure()
        }

        packet = json.dumps(data).encode()

        try:
            self.udp_sock.sendto(packet, self.server)
        except Exception:
            # Ignorer silencieusement pour économiser batterie
            pass

    def send_alert(self, alert_type, message):
        """
        Alerte critique : TCP (doit arriver)
        """
        # Établir connexion TCP
        tcp_sock = socket.socket(
            socket.AF_INET,
            socket.SOCK_STREAM
        )
        tcp_sock.connect(self.server_tcp)

        # Envoyer alerte
        alert = {
            'type': 'alert',
            'alert_type': alert_type,
            'message': message,
            'timestamp': int(time.time())
        }

        tcp_sock.sendall(json.dumps(alert).encode())

        # Attendre confirmation
        response = tcp_sock.recv(1024)

        tcp_sock.close()
```

### 5. Services infrastructure

```
┌──────────────────────────────────────────────────────────┐
│              Services Infrastructure                     │
├────────────────────┬─────────────────────────────────────┤
│ Service            │ Protocole et justification          │
├────────────────────┼─────────────────────────────────────┤
│ DNS                │ UDP : Requêtes courtes              │
│ NTP                │ UDP : Synchronisation temps         │
│ DHCP               │ UDP : Bootstrap sans IP             │
│ SNMP               │ UDP : Monitoring léger              │
│ Syslog             │ UDP : Volume élevé de logs          │
│                    │                                     │
│ SSH                │ TCP : Shell interactif              │
│ HTTPS              │ TCP : Web sécurisé                  │
│ Database           │ TCP : Transactions ACID             │
│ SMTP               │ TCP : Fiabilité email               │
│ LDAP               │ TCP : Requêtes annuaire             │
└────────────────────┴─────────────────────────────────────┘
```

## Architectures réelles

### Architecture 1 : Application de visioconférence (type Zoom)

```
┌────────────────────────────────────────────────────────┐
│                    CLIENT                              │
│                                                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Signalisation (TCP)                             │   │
│  │ • Authentification                              │   │
│  │ • Liste participants                            │   │
│  │ • Contrôles réunion (mute, kick, etc.)          │   │
│  └─────────────────────────────────────────────────┘   │
│                                                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Médias Audio/Vidéo (UDP)                        │   │
│  │ • Stream vidéo (H.264)                          │   │
│  │ • Stream audio (Opus)                           │   │
│  │ • Partage écran                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Chat texte (TCP)                                │   │
│  │ • Messages persistants                          │   │
│  │ • Partage fichiers                              │   │
│  └─────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘

Statistiques typiques :
  • TCP signalisation : 10-50 KB/s
  • UDP audio : 32-128 Kbps
  • UDP vidéo : 500-3000 Kbps
  • TCP chat : <1 KB/s

Total : Majoritairement UDP pour bande passante
```

### Architecture 2 : Jeu MMO (type World of Warcraft)

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT                               │
│                                                         │
│  Connexion TCP principale (port 3724)                   │
│  ┌─────────────────────────────────────────────────┐    │
│  │ • Authentification                              │    │
│  │ • Sélection personnage                          │    │
│  │ • Chat global                                   │    │
│  │ • Trading                                       │    │
│  │ • Inventaire                                    │    │
│  │ • Quêtes                                        │    │
│  │ • Actions critiques (loot, craft)               │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  Connexion UDP instance (port 3725)                     │
│  ┌─────────────────────────────────────────────────┐    │
│  │ • Positions joueurs/NPCs                        │    │
│  │ • Animations combat                             │    │
│  │ • Effets visuels                                │    │
│  │ • État monde (arbres, herbe, etc.)              │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘

Raison du split :
  • TCP : Données persistantes, impactent progression
  • UDP : Données cosmétiques, pertes non critiques

Si instance combat instancié (dungeon) :
  • Tous les joueurs proches
  • Latence critique
  • 90% UDP, 10% TCP

Si monde ouvert avec 1000+ joueurs :
  • Optimisation nécessaire
  • 70% UDP, 30% TCP
```

### Architecture 3 : Système de monitoring distribué

```
┌─────────────────────────────────────────────────────────┐
│              Collecte de métriques                      │
│                                                         │
│  Agents (sur serveurs monitorés)                        │
│  │                                                      │
│  ├─► Métriques système (UDP)                            │
│  │   • CPU, RAM, Disk toutes les 10s                    │
│  │   • Réseau, I/O                                      │
│  │   • 100s de métriques/agent                          │
│  │   • 1000 agents = 10,000 métriques/sec               │
│  │   • UDP : Économie CPU                               │
│  │                                                      │
│  ├─► Alertes critiques (TCP)                            │
│  │   • Disque >95%                                      │
│  │   • Service down                                     │
│  │   • Erreurs application                              │
│  │   • DOIT être livré                                  │
│  │                                                      │
│  └─► Logs applicatifs (TCP+Buffer)                      │
│      • Bufferisés localement                            │
│      • Envoyés par batch                                │
│      • TCP pour fiabilité                               │
│                                                         │
│  ▼                                                      │
│                                                         │
│  Serveur central                                        │
│  ├─► Stockage time-series (métriques)                   │
│  ├─► Base de données (alertes)                          │
│  └─► Elasticsearch (logs)                               │
└─────────────────────────────────────────────────────────┘

Justification :
  • Métriques : Volume énorme, 1% perte OK
  • Alertes : Critiques, 0% perte
  • Logs : Important pour debug, buffering OK
```

## Erreurs courantes à éviter

### Erreur 1 : "UDP est plus rapide donc meilleur"

```
❌ FAUX

UDP n'est pas "plus rapide" en termes de vitesse de transmission.
Les paquets voyagent à la même vitesse réseau.

UDP économise :
  • Latence de handshake (150ms)
  • Overhead d'en-tête (12 octets/paquet)
  • CPU de traitement

Mais si l'application doit réimplémenter la fiabilité :
  → Code complexe
  → Bugs potentiels
  → Souvent moins efficace que TCP natif

✓ CORRECT

Utilisez UDP seulement si :
  • Latence <150ms est critique ET
  • Pertes sont acceptables
```

### Erreur 2 : "TCP pour tout par défaut"

```
❌ SOUS-OPTIMAL

TCP n'est pas toujours le bon choix même s'il est "safe".

Exemple : Streaming vidéo avec TCP
  • Un paquet perdu bloque tous les suivants
  • Buffering de plusieurs secondes
  • Expérience dégradée

✓ CORRECT

Évaluez les besoins réels :
  • Temps réel ? → Considérer UDP
  • Pertes acceptables ? → Considérer UDP
  • Latence critique ? → Considérer UDP
```

### Erreur 3 : "Réimplémenter TCP au-dessus d'UDP"

```
❌ ANTI-PATTERN

class ReliableUDP:
    def __init__(self):
        # Numérotation séquentielle
        self.sequence = 0
        # Buffer de réception
        self.buffer = {}
        # ACKs
        self.pending_acks = {}
        # Retransmissions
        self.retransmit_queue = {}
        # Contrôle de flux
        self.window_size = 64
        # Contrôle de congestion
        self.cwnd = 1

    # ... 500 lignes de code complexe ...

Résultat :
  • Complexité énorme
  • Bugs subtils
  • Performance inférieure à TCP natif
  • Maintenance cauchemardesque

✓ CORRECT

Si vous avez besoin de fiabilité complète :
  → Utilisez TCP !

Si vous avez besoin de fiabilité partielle :
  → UDP + ACK simple pour messages critiques seulement
```

### Erreur 4 : "UDP sans contrôle de congestion"

```
❌ IRRESPONSABLE

Application envoie UDP à pleine vitesse :
  • Sature son propre réseau
  • Pénalise autres applications
  • Peut être banni par ISP

✓ CORRECT

class ResponsibleUDPApp:
    def __init__(self):
        self.rate_limit = 1_000_000  # 1 Mbps initial

    def send(self, data):
        # Rate limiting
        delay = len(data) * 8 / self.rate_limit
        time.sleep(delay)

        self.sock.sendto(data, dest)

    def on_loss_feedback(self, loss_rate):
        # Adapter le débit
        if loss_rate > 0.05:
            self.rate_limit *= 0.8  # Réduire
        elif loss_rate < 0.01:
            self.rate_limit *= 1.1  # Augmenter
```

## Matrice de décision finale

```
┌──────────────────────────────────────────────────────────────────┐
│              MATRICE DE DÉCISION TCP vs UDP                      │
├────────────────────────────────────┬─────────────┬───────────────┤
│ Critère                            │ TCP         │ UDP           │
├────────────────────────────────────┼─────────────┼───────────────┤
│ Fiabilité requise                  │ ✓✓✓         │ ✗             │
│ Ordre des messages important       │ ✓✓✓         │ ✗             │
│ Latence < 150ms critique           │ ✗           │ ✓✓✓           │
│ Données volumineuses (>10 MB)      │ ✓✓✓         │ ✗             │
│ Transfert de fichiers              │ ✓✓✓         │ ✗             │
│ Transactions                       │ ✓✓✓         │ ✗             │
│                                    │             │               │
│ Temps réel (gaming, VoIP)          │ ✗           │ ✓✓✓           │
│ Streaming live                     │ ✗           │ ✓✓✓           │
│ Données éphémères                  │ ✗           │ ✓✓✓           │
│ Messages courts (<100 octets)      │ ~           │ ✓✓✓           │
│ Fréquence >100 msg/sec             │ ~           │ ✓✓✓           │
│ Broadcast/Multicast                │ ✗           │ ✓✓✓           │
│                                    │             │               │
│ Navigation web                     │ ✓✓✓         │ ✗             │
│ API REST                           │ ✓✓✓         │ ✗             │
│ Email                              │ ✓✓✓         │ ✗             │
│ SSH / Telnet                       │ ✓✓✓         │ ✗             │
│ Base de données                    │ ✓✓✓         │ ✗             │
│                                    │             │               │
│ DNS                                │ ✗           │ ✓✓✓           │
│ NTP                                │ ✗           │ ✓✓✓           │
│ DHCP                               │ ✗           │ ✓✓✓           │
│ Métriques / Telemetry              │ ~           │ ✓✓✓           │
└────────────────────────────────────┴─────────────┴───────────────┘

Légende :
  ✓✓✓ : Excellent choix
  ~   : Acceptable mais sous-optimal
  ✗   : Mauvais choix / Impossible
```

## Checklist de décision pratique

### Questions à se poser

```
FIABILITÉ
□ Une perte de données est-elle acceptable ?
  ☐ Non → TCP
  ☐ Oui, <5% → UDP possible

□ L'ordre des messages est-il critique ?
  ☐ Oui → TCP
  ☐ Non → UDP possible

PERFORMANCE
□ Quelle est la latence maximale acceptable ?
  ☐ <150ms → UDP fortement recommandé
  ☐ <500ms → UDP recommandé
  ☐ >500ms → TCP acceptable

□ Quel est le volume de messages ?
  ☐ >100/sec → UDP avantageux
  ☐ <10/sec → TCP acceptable

DONNÉES
□ Quelle est la durée de vie des données ?
  ☐ <100ms → UDP
  ☐ <1s → UDP recommandé
  ☐ >1s → TCP acceptable

□ Quelle est la taille des messages ?
  ☐ <100 octets → UDP avantageux
  ☐ >10 KB → TCP recommandé

INFRASTRUCTURE
□ Combien de connexions simultanées ?
  ☐ >10,000 → UDP plus scalable
  ☐ <1,000 → TCP acceptable

□ Le budget bande passante est-il serré ?
  ☐ Oui → UDP économise 10-20%
  ☐ Non → TCP acceptable

EXPERTISE
□ Avez-vous l'expertise UDP ?
  ☐ Non → TCP (plus simple)
  ☐ Oui → UDP si justifié
```

### Score de décision

```python
def calculate_protocol_score(requirements):
    """
    Calculer un score pour aider à choisir le protocole
    """
    tcp_score = 0
    udp_score = 0

    # Fiabilité
    if requirements['reliability'] == 'critical':
        tcp_score += 10
    elif requirements['reliability'] == 'optional':
        udp_score += 5

    # Latence
    if requirements['max_latency_ms'] < 150:
        udp_score += 10
    elif requirements['max_latency_ms'] < 500:
        udp_score += 5
    else:
        tcp_score += 3

    # Ordre
    if requirements['order_required']:
        tcp_score += 8
    else:
        udp_score += 3

    # Volume
    if requirements['messages_per_second'] > 100:
        udp_score += 7
    elif requirements['messages_per_second'] < 10:
        tcp_score += 2

    # Taille
    if requirements['avg_message_size'] < 100:
        udp_score += 5
    elif requirements['avg_message_size'] > 10000:
        tcp_score += 5

    # Durée de vie données
    if requirements['data_lifetime_ms'] < 100:
        udp_score += 10
    elif requirements['data_lifetime_ms'] > 5000:
        tcp_score += 5

    # Scalabilité
    if requirements['concurrent_connections'] > 10000:
        udp_score += 6

    # Broadcast
    if requirements['needs_broadcast']:
        udp_score += 15  # TCP impossible

    # Résultat
    if tcp_score > udp_score * 1.5:
        return 'TCP', f"Score: TCP={tcp_score}, UDP={udp_score}"
    elif udp_score > tcp_score * 1.5:
        return 'UDP', f"Score: TCP={tcp_score}, UDP={udp_score}"
    else:
        return 'HYBRIDE', f"Score: TCP={tcp_score}, UDP={udp_score} (proche)"

# Exemple 1 : Jeu FPS
game_requirements = {
    'reliability': 'optional',
    'max_latency_ms': 50,
    'order_required': False,
    'messages_per_second': 60,
    'avg_message_size': 50,
    'data_lifetime_ms': 16,
    'concurrent_connections': 64,
    'needs_broadcast': False
}

protocol, reason = calculate_protocol_score(game_requirements)
print(f"Jeu FPS : {protocol}")
print(reason)
# → UDP (Score: TCP=0, UDP=42)

# Exemple 2 : API REST
api_requirements = {
    'reliability': 'critical',
    'max_latency_ms': 2000,
    'order_required': True,
    'messages_per_second': 5,
    'avg_message_size': 2000,
    'data_lifetime_ms': 30000,
    'concurrent_connections': 1000,
    'needs_broadcast': False
}

protocol, reason = calculate_protocol_score(api_requirements)
print(f"\nAPI REST : {protocol}")
print(reason)
# → TCP (Score: TCP=28, UDP=0)
```

## Conclusion

Le choix entre TCP et UDP n'est pas une question de "meilleur" protocole, mais de **meilleur protocole pour votre cas d'usage spécifique**.

### Principes directeurs

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  1. Par défaut, utilisez TCP                             │
│     C'est le choix sûr et simple                         │
│                                                          │
│  2. Considérez UDP si ET SEULEMENT SI :                  │
│     • Latence <150ms est CRITIQUE                        │
│     • Pertes 1-5% sont ACCEPTABLES                       │
│     • Vous comprenez les implications                    │
│                                                          │
│  3. N'essayez pas de réinventer TCP au-dessus d'UDP      │
│     Si vous avez besoin de fiabilité, utilisez TCP       │
│                                                          │
│  4. Si UDP, implémentez le contrôle de congestion        │
│     C'est une responsabilité éthique                     │
│                                                          │
│  5. Envisagez une architecture hybride                   │
│     TCP pour données critiques                           │
│     UDP pour données temps réel                          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Récapitulatif par cas d'usage

**Utilisez TCP pour** :
- ✓ Applications web (HTTP/HTTPS)
- ✓ APIs REST et GraphQL
- ✓ Transfert de fichiers
- ✓ Email et messagerie asynchrone
- ✓ Bases de données
- ✓ Transactions financières
- ✓ SSH et administration à distance
- ✓ Tout ce qui nécessite fiabilité à 100%

**Utilisez UDP pour** :
- ✓ Jeux multijoueurs en temps réel
- ✓ VoIP et visioconférence
- ✓ Streaming live (sports, événements)
- ✓ DNS, NTP, DHCP
- ✓ Métriques et monitoring haute fréquence
- ✓ IoT avec contraintes énergétiques
- ✓ Broadcast et multicast

**Utilisez une approche hybride pour** :
- ✓ Gaming complexe (position en UDP, chat en TCP)
- ✓ Collaboration temps réel (curseurs en UDP, éditions en TCP)
- ✓ Streaming avec métadonnées (média en UDP, info en TCP)

### Ressources et évolutions

Le paysage évolue avec de nouveaux protocoles qui combinent les avantages des deux :

**QUIC (HTTP/3)** :
- Transport UDP
- Fiabilité TCP
- Multiplexage sans head-of-line blocking
- Handshake 0-RTT

**WebRTC** :
- UDP pour média temps réel
- Traversée NAT intégrée
- Sécurité (DTLS)

Ces protocoles montrent que la frontière entre TCP et UDP devient plus floue, avec des solutions hybrides qui prennent le meilleur des deux mondes.

Dans les sections suivantes, nous plongerons en profondeur dans TCP pour comprendre comment il implémente toutes ces garanties de fiabilité, d'ordre et de contrôle qui en font le protocole dominant sur Internet.

---

**À retenir** :

- ✅ **Décision basée sur les besoins** : Pas de "meilleur" protocole absolu
- ✅ **TCP par défaut** : Choix sûr si incertain
- ✅ **UDP si justifié** : Latence critique + pertes acceptables
- ✅ **Ne pas réinventer TCP** : Si fiabilité nécessaire, utiliser TCP
- ✅ **Contrôle de congestion** : Obligatoire pour applications UDP
- ✅ **Architecture hybride** : Souvent la meilleure solution
- ✅ **Mesurer et adapter** : Tester avec métriques réelles
- ✅ **Évolution** : QUIC/HTTP/3 combinent avantages TCP+UDP

⏭️ [5. La couche Application](/05-couche-application/README.md)
