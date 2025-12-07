🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.3 Sockets UDP : envoi et réception de datagrammes

## Introduction

Si TCP est comme un appel téléphonique (connexion établie, conversation ordonnée), **UDP** (User Datagram Protocol) est comme envoyer des cartes postales : rapide, simple, mais sans garantie de livraison ni d'ordre.

UDP est le protocole de choix quand la **vitesse prime sur la fiabilité**, quand perdre occasionnellement des données est acceptable, ou quand l'application peut gérer elle-même la fiabilité. De Netflix à Zoom, des jeux vidéo au DNS, UDP est partout où la latence compte plus que la garantie absolue.

## Rappel : caractéristiques d'UDP

### Ce qu'UDP offre

- ✅ **Rapidité** : Pas de handshake, envoi immédiat
- ✅ **Faible overhead** : Header de seulement 8 octets (vs 20+ pour TCP)
- ✅ **Sans état** : Pas de connexion à maintenir
- ✅ **Support multicast/broadcast** : Communication un-vers-plusieurs
- ✅ **Frontières de messages** : Chaque datagramme est distinct

### Ce qu'UDP ne garantit PAS

- ❌ **Livraison** : Les paquets peuvent être perdus
- ❌ **Ordre** : Peuvent arriver dans le désordre
- ❌ **Non-duplication** : Un paquet peut arriver plusieurs fois
- ❌ **Contrôle de flux** : Aucune adaptation au récepteur
- ❌ **Contrôle de congestion** : Aucune adaptation au réseau

### Comparaison visuelle TCP vs UDP

```
TCP :
Client                          Serveur
  |                                |
  |---- SYN -------------------->  |
  |<--- SYN-ACK ---------------    |
  |---- ACK -------------------->  |
  |                                |
  |---- Data 1 (ACK requis) --->   |
  |<--- ACK -------------------    |
  |---- Data 2 (ACK requis) --->   |
  |<--- ACK -------------------    |
  |                                |
  Total : 6 messages pour 2 paquets de données

UDP :
Client                          Serveur
  |                                |
  |---- Data 1 ------------------> |
  |---- Data 2 ------------------> |
  |                                |
  Total : 2 messages, envoi direct
```

## Anatomie d'un datagramme UDP

### Structure du header UDP

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            Data...                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Champs** :
- **Source Port** (16 bits) : Port de l'émetteur
- **Destination Port** (16 bits) : Port du destinataire
- **Length** (16 bits) : Longueur totale (header + data)
- **Checksum** (16 bits) : Détection d'erreurs (optionnel en IPv4)

**Taille totale** : 8 octets (vs 20 octets minimum pour TCP)

### Taille maximale d'un datagramme

```python
# Calcul de la MTU (Maximum Transmission Unit)
MTU_Ethernet = 1500  # octets
IP_Header = 20       # octets minimum
UDP_Header = 8       # octets

# Payload maximum
Max_UDP_Payload = MTU_Ethernet - IP_Header - UDP_Header
# = 1500 - 20 - 8 = 1472 octets

# Si dépassé → fragmentation IP (à éviter)
```

**Conséquence pratique** :
```python
# BON : datagramme < 1472 octets
sock.sendto(b"x" * 1400, ("192.168.1.100", 9999))

# RISQUÉ : datagramme > 1472 octets → fragmentation
sock.sendto(b"x" * 2000, ("192.168.1.100", 9999))
# Peut être perdu si un fragment est perdu
```

## Création et utilisation de sockets UDP

### Création d'une socket UDP

```python
import socket

# Création d'une socket UDP (SOCK_DGRAM)
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Ou avec IPv6
udp_socket_v6 = socket.socket(socket.AF_INET6, socket.SOCK_DGRAM)
```

**Différence avec TCP** : Pas besoin de `connect()`, `listen()`, ou `accept()`

### Côté serveur : réception de datagrammes

```python
import socket

def udp_server(host='0.0.0.0', port=9999):
    # 1. Création de la socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # 2. Liaison à une adresse (pour recevoir)
    sock.bind((host, port))
    print(f"Serveur UDP en écoute sur {host}:{port}")

    while True:
        # 3. Réception d'un datagramme
        data, client_address = sock.recvfrom(4096)

        print(f"Reçu de {client_address}: {data.decode('utf-8')}")

        # 4. Réponse (optionnelle)
        response = f"Echo: {data.decode('utf-8')}"
        sock.sendto(response.encode('utf-8'), client_address)

if __name__ == "__main__":
    udp_server()
```

**Points clés** :
- `bind()` est **obligatoire** pour un serveur (pour avoir une adresse connue)
- `recvfrom()` retourne les données **et** l'adresse de l'expéditeur
- Chaque datagramme est **indépendant** (pas de connexion)

### Côté client : envoi de datagrammes

```python
import socket

def udp_client(server_host='127.0.0.1', server_port=9999):
    # 1. Création de la socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # 2. Envoi direct (pas de connect nécessaire)
    message = "Hello UDP Server!"
    sock.sendto(message.encode('utf-8'), (server_host, server_port))

    # 3. Réception de la réponse (avec timeout)
    sock.settimeout(5.0)

    try:
        data, server = sock.recvfrom(4096)
        print(f"Réponse du serveur: {data.decode('utf-8')}")
    except socket.timeout:
        print("Pas de réponse du serveur (timeout)")
    finally:
        sock.close()

if __name__ == "__main__":
    udp_client()
```

**Points clés** :
- `bind()` est **optionnel** pour un client (le système assigne un port aléatoire)
- Timeout **crucial** car pas de garantie de réponse
- Pas de notion de "connexion" ou "déconnexion"

### Utilisation de connect() avec UDP (optionnel)

UDP peut utiliser `connect()` pour **fixer un destinataire par défaut** :

```python
import socket

# Socket UDP "connectée"
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.connect(('8.8.8.8', 53))  # DNS de Google

# Avantages :
# 1. Utiliser send() au lieu de sendto()
sock.send(b"DNS Query")

# 2. Recevoir uniquement de cette adresse
data = sock.recv(1024)  # Au lieu de recvfrom()

# 3. Filtrage automatique des autres expéditeurs
sock.close()
```

**Cas d'usage** : Communication avec un seul serveur connu

**Note** : Cela ne crée PAS de connexion TCP. C'est purement une configuration locale de la socket.

## sendto() et recvfrom() en détail

### sendto() - Envoi de datagrammes

```python
bytes_sent = sock.sendto(data, (host, port))
```

**Paramètres** :
- `data` : Données à envoyer (bytes)
- `(host, port)` : Adresse de destination

**Valeur de retour** : Nombre d'octets envoyés

**Caractéristiques** :
- **Non-bloquant** (généralement) : retourne immédiatement
- **Atomique** : tout le datagramme est envoyé ou rien
- **Pas de garantie de livraison**

```python
import socket
import time

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Envoi multiple
for i in range(10):
    message = f"Message {i}"
    sock.sendto(message.encode('utf-8'), ('192.168.1.100', 9999))
    time.sleep(0.1)

sock.close()

# Aucune garantie que les 10 messages arrivent
# Peuvent arriver dans le désordre
```

**Gestion de la taille** :

```python
def send_safe_udp(sock, data, address, max_size=1400):
    """Envoie un datagramme avec vérification de taille"""
    if len(data) > max_size:
        raise ValueError(f"Datagramme trop grand: {len(data)} > {max_size}")

    return sock.sendto(data, address)

# Utilisation
try:
    send_safe_udp(sock, large_data, ('host', 9999))
except ValueError as e:
    print(f"Erreur: {e}")
    # Fragmenter manuellement au niveau applicatif
```

### recvfrom() - Réception de datagrammes

```python
data, sender_address = sock.recvfrom(buffer_size)
```

**Paramètres** :
- `buffer_size` : Taille maximale à recevoir

**Valeurs de retour** :
- `data` : Données reçues (bytes)
- `sender_address` : Tuple `(IP, port)` de l'expéditeur

**Caractéristiques** :
- **Bloquant** par défaut (attend un datagramme)
- **Retourne un datagramme complet** (pas de fragmentation comme TCP)
- **Peut perdre des données** si buffer_size < taille du datagramme

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(('0.0.0.0', 9999))

while True:
    # Buffer de 4096 octets
    data, addr = sock.recvfrom(4096)

    # Si le datagramme fait 5000 octets :
    # → Tronqué à 4096 octets, le reste est PERDU

    print(f"Reçu {len(data)} octets de {addr}")
```

**Bonne pratique : buffer_size approprié**

```python
# Trop petit : perte de données
sock.recvfrom(512)  # Si datagramme > 512 → tronqué

# Recommandé : MTU standard
sock.recvfrom(1500)  # Couvre la plupart des cas

# Sûr : maximum théorique UDP
sock.recvfrom(65535)  # 2^16 - 1, taille max d'un datagramme UDP
```

### Exemple : serveur echo UDP complet

```python
import socket
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class UDPEchoServer:
    def __init__(self, host='0.0.0.0', port=9999, buffer_size=4096):
        self.host = host
        self.port = port
        self.buffer_size = buffer_size
        self.sock = None
        self.stats = {'received': 0, 'sent': 0, 'errors': 0}

    def start(self):
        """Démarre le serveur UDP"""
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

        # Option : réutilisation d'adresse
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        self.sock.bind((self.host, self.port))
        logger.info(f"Serveur UDP démarré sur {self.host}:{self.port}")

        try:
            self.run()
        except KeyboardInterrupt:
            logger.info("\nArrêt du serveur...")
        finally:
            self.shutdown()

    def run(self):
        """Boucle principale de réception"""
        while True:
            try:
                # Réception d'un datagramme
                data, client_addr = self.sock.recvfrom(self.buffer_size)
                self.stats['received'] += 1

                logger.info(f"Reçu de {client_addr}: {len(data)} octets")

                # Traitement
                self.handle_datagram(data, client_addr)

            except Exception as e:
                logger.error(f"Erreur: {e}")
                self.stats['errors'] += 1

    def handle_datagram(self, data, client_addr):
        """Traite un datagramme individuel"""
        try:
            message = data.decode('utf-8')
            logger.info(f"Message: {message}")

            # Echo
            response = f"ECHO: {message}"
            self.sock.sendto(response.encode('utf-8'), client_addr)
            self.stats['sent'] += 1

        except UnicodeDecodeError:
            logger.error("Impossible de décoder le message")

    def shutdown(self):
        """Arrêt propre du serveur"""
        if self.sock:
            self.sock.close()

        logger.info(f"Statistiques: {self.stats}")

if __name__ == "__main__":
    server = UDPEchoServer(port=9999)
    server.start()
```

**Client correspondant** :

```python
import socket

class UDPEchoClient:
    def __init__(self, server_host='127.0.0.1', server_port=9999):
        self.server_addr = (server_host, server_port)
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.settimeout(3.0)  # Timeout de 3 secondes

    def send_message(self, message):
        """Envoie un message et attend la réponse"""
        try:
            # Envoi
            self.sock.sendto(message.encode('utf-8'), self.server_addr)
            print(f"Envoyé: {message}")

            # Réception (avec timeout)
            data, server = self.sock.recvfrom(4096)
            print(f"Réponse: {data.decode('utf-8')}")

            return data

        except socket.timeout:
            print("Timeout: pas de réponse du serveur")
            return None

        except Exception as e:
            print(f"Erreur: {e}")
            return None

    def close(self):
        self.sock.close()

if __name__ == "__main__":
    client = UDPEchoClient()

    client.send_message("Hello UDP!")
    client.send_message("Test message 2")

    client.close()
```

## Patterns de communication UDP

### 1. Unicast (un-à-un)

Communication standard entre un client et un serveur :

```python
# Client → Serveur spécifique
sock.sendto(data, ('192.168.1.100', 9999))
```

**Cas d'usage** : DNS, DHCP, la plupart des applications client-serveur

### 2. Broadcast (un-à-tous sur le réseau local)

Envoi à **tous** les hôtes du réseau local :

```python
import socket

# Activation du broadcast
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)

# Envoi en broadcast sur le réseau local
broadcast_addr = '255.255.255.255'  # Tous les hôtes
sock.sendto(b"DISCOVERY", (broadcast_addr, 9999))

# Ou broadcast spécifique au sous-réseau
# 192.168.1.255 pour le réseau 192.168.1.0/24
sock.sendto(b"DISCOVERY", ('192.168.1.255', 9999))
```

**Cas d'usage réel : découverte de services**

```python
import socket
import json
import time

class ServiceDiscovery:
    """Découverte de services sur le réseau local via broadcast"""

    def __init__(self, port=37020):
        self.port = port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
        self.sock.settimeout(2.0)

    def announce_service(self, service_info):
        """Annonce un service en broadcast"""
        announcement = json.dumps({
            'type': 'service_announcement',
            'service': service_info
        })

        self.sock.sendto(
            announcement.encode('utf-8'),
            ('255.255.255.255', self.port)
        )

    def discover_services(self, timeout=5.0):
        """Découvre les services disponibles"""
        # Envoi d'une requête de découverte
        discovery = json.dumps({'type': 'discovery_request'})
        self.sock.sendto(
            discovery.encode('utf-8'),
            ('255.255.255.255', self.port)
        )

        # Collecte des réponses
        services = []
        start_time = time.time()

        while time.time() - start_time < timeout:
            try:
                data, addr = self.sock.recvfrom(4096)
                response = json.loads(data.decode('utf-8'))

                if response['type'] == 'service_announcement':
                    services.append({
                        'address': addr,
                        'info': response['service']
                    })
                    print(f"Service trouvé: {addr} - {response['service']}")

            except socket.timeout:
                continue
            except Exception as e:
                print(f"Erreur: {e}")

        return services

# Utilisation
discovery = ServiceDiscovery()
services = discovery.discover_services(timeout=3.0)
print(f"Total: {len(services)} services trouvés")
```

**Exemple : implémentation simplifiée de DHCP Discovery**

```python
import socket
import struct

def dhcp_discover():
    """Envoie un paquet DHCP DISCOVER en broadcast"""
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    # Paquet DHCP simplifié (normalement plus complexe)
    dhcp_discover = (
        b'\x01'          # Message type: Boot Request
        b'\x01'          # Hardware type: Ethernet
        b'\x06'          # Hardware address length
        b'\x00'          # Hops
        b'\x12\x34\x56\x78'  # Transaction ID
        # ... (reste du paquet DHCP)
    )

    # Envoi en broadcast
    sock.sendto(dhcp_discover, ('255.255.255.255', 67))

    # Attente de réponse DHCP OFFER
    sock.settimeout(5.0)
    try:
        data, server = sock.recvfrom(4096)
        print(f"DHCP OFFER reçu de {server}")
    except socket.timeout:
        print("Pas de serveur DHCP trouvé")

    sock.close()
```

**Limitations du broadcast** :
- ❌ Limité au réseau local (routeurs ne forwardent pas)
- ❌ Peut saturer le réseau si utilisé massivement
- ❌ Tous les hôtes reçoivent (même ceux non concernés)

### 3. Multicast (un-à-plusieurs abonnés)

Envoi à un **groupe** d'hôtes abonnés :

```python
import socket
import struct

# Serveur multicast
def multicast_sender(group='224.1.1.1', port=5007):
    """Envoie des messages multicast"""
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)

    # TTL (Time To Live) : nombre de routeurs à traverser
    sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 2)

    message_count = 0
    while True:
        message = f"Multicast message {message_count}"
        sock.sendto(message.encode('utf-8'), (group, port))
        print(f"Envoyé: {message}")
        message_count += 1
        time.sleep(1)

# Client multicast
def multicast_receiver(group='224.1.1.1', port=5007):
    """Reçoit des messages multicast"""
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    # Bind au port
    sock.bind(('', port))

    # Rejoindre le groupe multicast
    mreq = struct.pack("4sl", socket.inet_aton(group), socket.INADDR_ANY)
    sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)

    print(f"Abonné au groupe multicast {group}:{port}")

    while True:
        data, addr = sock.recvfrom(4096)
        print(f"Reçu de {addr}: {data.decode('utf-8')}")
```

**Adresses multicast** :
- Classe D : 224.0.0.0 à 239.255.255.255
- 224.0.0.0 à 224.0.0.255 : Réseau local uniquement
- 239.0.0.0 à 239.255.255.255 : Scope administratif

**Cas d'usage réels** :
- **Streaming vidéo IPTV** : Un serveur, millions de spectateurs
- **Cotations boursières** : Diffusion de prix en temps réel
- **Discovery protocols** : mDNS (Bonjour), SSDP (UPnP)
- **Synchronisation d'horloges** : NTP

**Exemple : serveur de streaming vidéo multicast (conceptuel)**

```python
import socket
import struct

class MulticastVideoStreamer:
    """Diffuse un flux vidéo en multicast"""

    def __init__(self, group='239.1.1.1', port=5004):
        self.group = group
        self.port = port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

        # TTL : portée du multicast
        # 1 = local subnet, 32 = organisation, 255 = internet
        self.sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 32)

    def stream_file(self, video_path, chunk_size=1316):
        """Diffuse un fichier vidéo en chunks UDP"""
        with open(video_path, 'rb') as f:
            sequence = 0

            while True:
                chunk = f.read(chunk_size)
                if not chunk:
                    break

                # Header simple : numéro de séquence (4 octets) + données
                packet = struct.pack('!I', sequence) + chunk

                self.sock.sendto(packet, (self.group, self.port))
                sequence += 1

                # Simuler un débit constant (ex: 1 Mbps)
                time.sleep(chunk_size * 8 / 1_000_000)  # en secondes

        print(f"Streaming terminé: {sequence} packets envoyés")

# Client/Viewer
class MulticastVideoReceiver:
    """Reçoit un flux vidéo multicast"""

    def __init__(self, group='239.1.1.1', port=5004):
        self.group = group
        self.port = port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.sock.bind(('', port))

        # Rejoindre le groupe
        mreq = struct.pack("4sl", socket.inet_aton(group), socket.INADDR_ANY)
        self.sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)

    def receive_stream(self, output_path):
        """Reçoit et sauvegarde le flux"""
        expected_seq = 0
        lost_packets = 0

        with open(output_path, 'wb') as f:
            while True:
                try:
                    data, addr = self.sock.recvfrom(4096)

                    # Extraire le numéro de séquence
                    seq = struct.unpack('!I', data[:4])[0]
                    chunk = data[4:]

                    # Détection de perte
                    if seq != expected_seq:
                        lost = seq - expected_seq
                        lost_packets += lost
                        print(f"⚠️  {lost} packets perdus (seq {expected_seq} à {seq-1})")

                    f.write(chunk)
                    expected_seq = seq + 1

                except KeyboardInterrupt:
                    break

        print(f"Réception terminée. Packets perdus: {lost_packets}")
```

## Gestion de la fiabilité au niveau applicatif

UDP ne garantit rien, mais l'application peut implémenter sa propre fiabilité.

### Pattern 1 : Numéros de séquence

Détecter les pertes et le désordre :

```python
import struct
import socket

class ReliableUDPSender:
    """Envoi UDP avec numéros de séquence"""

    def __init__(self, dest_addr):
        self.dest_addr = dest_addr
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sequence = 0

    def send_message(self, message):
        """Envoie un message avec numéro de séquence"""
        # Format: [Sequence (4 bytes)][Message]
        packet = struct.pack('!I', self.sequence) + message.encode('utf-8')

        self.sock.sendto(packet, self.dest_addr)
        print(f"Envoyé seq={self.sequence}: {message}")

        self.sequence += 1

class ReliableUDPReceiver:
    """Réception UDP avec détection de perte"""

    def __init__(self, port):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.bind(('0.0.0.0', port))
        self.expected_seq = 0

    def receive_message(self):
        """Reçoit et vérifie l'ordre des messages"""
        data, addr = self.sock.recvfrom(4096)

        # Extraire le numéro de séquence
        seq = struct.unpack('!I', data[:4])[0]
        message = data[4:].decode('utf-8')

        # Vérification
        if seq == self.expected_seq:
            print(f"✓ Reçu seq={seq}: {message}")
            self.expected_seq += 1
        elif seq > self.expected_seq:
            lost = seq - self.expected_seq
            print(f"⚠️  {lost} message(s) perdu(s) (attendu {self.expected_seq}, reçu {seq})")
            self.expected_seq = seq + 1
        else:
            print(f"⚠️  Message en retard seq={seq} (attendu {self.expected_seq})")

        return message
```

### Pattern 2 : ACK et retransmission

Implémenter un mécanisme TCP-like :

```python
import socket
import time
import threading
import struct

class UDPWithACK:
    """UDP avec acquittements et retransmissions"""

    def __init__(self, local_port):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.bind(('0.0.0.0', local_port))
        self.sock.settimeout(0.1)

        self.pending_acks = {}  # seq -> (data, dest, timestamp)
        self.sequence = 0

        # Thread pour gérer les retransmissions
        self.running = True
        self.retrans_thread = threading.Thread(target=self._retransmission_worker)
        self.retrans_thread.daemon = True
        self.retrans_thread.start()

    def send_reliable(self, data, dest_addr, max_retries=3, timeout=1.0):
        """Envoie avec ACK et retransmissions"""
        seq = self.sequence
        self.sequence += 1

        # Construire le paquet: [Type:1][Seq:4][Data]
        packet = struct.pack('!BI', 0x01, seq) + data

        # Envoyer
        self.sock.sendto(packet, dest_addr)

        # Enregistrer pour retransmission
        self.pending_acks[seq] = {
            'data': packet,
            'dest': dest_addr,
            'timestamp': time.time(),
            'retries': 0,
            'max_retries': max_retries,
            'timeout': timeout
        }

        return seq

    def _retransmission_worker(self):
        """Thread qui retransmet les paquets non acquittés"""
        while self.running:
            now = time.time()

            for seq, info in list(self.pending_acks.items()):
                elapsed = now - info['timestamp']

                # Timeout dépassé ?
                if elapsed > info['timeout']:
                    if info['retries'] < info['max_retries']:
                        # Retransmettre
                        self.sock.sendto(info['data'], info['dest'])
                        self.pending_acks[seq]['retries'] += 1
                        self.pending_acks[seq]['timestamp'] = now
                        print(f"🔄 Retransmission seq={seq} (essai {info['retries']+1})")
                    else:
                        # Abandon
                        print(f"❌ Abandon seq={seq} après {info['max_retries']} essais")
                        del self.pending_acks[seq]

            time.sleep(0.1)

    def receive(self):
        """Reçoit et envoie des ACKs"""
        try:
            data, addr = self.sock.recvfrom(4096)

            # Parser le type de paquet
            pkt_type = struct.unpack('!B', data[:1])[0]

            if pkt_type == 0x01:  # Paquet de données
                seq = struct.unpack('!I', data[1:5])[0]
                payload = data[5:]

                # Envoyer un ACK
                ack_packet = struct.pack('!BI', 0x02, seq)  # Type ACK + Seq
                self.sock.sendto(ack_packet, addr)

                return payload, addr

            elif pkt_type == 0x02:  # ACK
                seq = struct.unpack('!I', data[1:5])[0]

                if seq in self.pending_acks:
                    print(f"✓ ACK reçu pour seq={seq}")
                    del self.pending_acks[seq]

                return None, addr

        except socket.timeout:
            return None, None

    def close(self):
        self.running = False
        self.retrans_thread.join()
        self.sock.close()

# Utilisation
sender = UDPWithACK(5000)
sender.send_reliable(b"Important message", ('192.168.1.100', 5001))

receiver = UDPWithACK(5001)
while True:
    data, addr = receiver.receive()
    if data:
        print(f"Reçu: {data}")
```

**Note** : C'est essentiellement réinventer TCP ! À utiliser seulement si vous avez des besoins spécifiques.

### Pattern 3 : Forward Error Correction (FEC)

Envoyer des données redondantes pour corriger les erreurs :

```python
# Exemple conceptuel avec codes de Reed-Solomon
# (nécessite une bibliothèque spécialisée)

def send_with_fec(data, sock, dest, redundancy=0.3):
    """
    Envoie des données avec redondance pour correction d'erreurs
    redundancy=0.3 → peut corriger jusqu'à 30% de perte
    """
    # Découper en chunks
    chunk_size = 1000
    chunks = [data[i:i+chunk_size] for i in range(0, len(data), chunk_size)]

    # Générer des chunks de parité (FEC)
    num_parity = int(len(chunks) * redundancy)
    parity_chunks = generate_parity(chunks, num_parity)

    # Envoyer tous les chunks (données + parité)
    for i, chunk in enumerate(chunks + parity_chunks):
        packet = struct.pack('!HH', i, len(chunks)) + chunk
        sock.sendto(packet, dest)

# Le récepteur peut reconstruire les données même avec des pertes
```

**Cas d'usage** : Streaming vidéo, VoIP (où retransmission serait trop lente)

## Cas d'usage réels détaillés

### 1. DNS (Domain Name System)

Le protocole DNS utilise UDP pour sa rapidité :

```python
import socket
import struct

def dns_query(domain, dns_server='8.8.8.8'):
    """Requête DNS simple pour obtenir l'IP d'un domaine"""
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.settimeout(2.0)

    # Construction d'une requête DNS (simplifiée)
    transaction_id = 0x1234

    # Header DNS (12 octets)
    header = struct.pack(
        '!HHHHHH',
        transaction_id,  # ID
        0x0100,         # Flags: standard query
        1,              # Questions: 1
        0,              # Answer RRs: 0
        0,              # Authority RRs: 0
        0               # Additional RRs: 0
    )

    # Question (nom de domaine + type A + classe IN)
    question = b''
    for part in domain.split('.'):
        question += struct.pack('!B', len(part)) + part.encode('utf-8')
    question += b'\x00'  # Fin du nom
    question += struct.pack('!HH', 1, 1)  # Type A, Classe IN

    query = header + question

    try:
        # Envoi de la requête
        sock.sendto(query, (dns_server, 53))

        # Réception de la réponse
        response, _ = sock.recvfrom(512)  # DNS limité à 512 octets en UDP

        # Parser la réponse (simplifié)
        print(f"Réponse DNS reçue: {len(response)} octets")

        # L'IP est dans les derniers 4 octets (simplifié)
        ip_bytes = response[-4:]
        ip = '.'.join(str(b) for b in ip_bytes)

        return ip

    except socket.timeout:
        print("Timeout DNS")
        return None
    finally:
        sock.close()

# Utilisation
ip = dns_query('example.com')
print(f"IP: {ip}")
```

**Pourquoi DNS utilise UDP ?**
- ✅ Requêtes très courtes (< 512 octets généralement)
- ✅ Latence critique (résolution de nom fréquente)
- ✅ Si perte, le client peut retry (simple)
- ✅ Serveurs DNS doivent gérer des millions de requêtes/sec

**Note** : DNS peut basculer sur TCP si la réponse dépasse 512 octets (EDNS permet des tailles plus grandes en UDP).

### 2. Streaming vidéo en temps réel

```python
import socket
import cv2
import struct
import numpy as np

class VideoStreamer:
    """Diffuse de la vidéo webcam en UDP"""

    def __init__(self, dest_addr, dest_port, quality=50):
        self.dest = (dest_addr, dest_port)
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.quality = quality
        self.frame_id = 0

    def stream(self):
        """Capture et diffuse la webcam"""
        cap = cv2.VideoCapture(0)

        while True:
            ret, frame = cap.read()
            if not ret:
                break

            # Compression JPEG
            _, buffer = cv2.imencode('.jpg', frame, [cv2.IMWRITE_JPEG_QUALITY, self.quality])
            data = buffer.tobytes()

            # Fragmentation si nécessaire (UDP max ~65KB)
            max_chunk = 60000

            if len(data) <= max_chunk:
                # Envoi direct
                packet = struct.pack('!IH', self.frame_id, 0) + data
                self.sock.sendto(packet, self.dest)
            else:
                # Fragmentation manuelle
                num_chunks = (len(data) + max_chunk - 1) // max_chunk

                for i in range(num_chunks):
                    start = i * max_chunk
                    end = min(start + max_chunk, len(data))
                    chunk = data[start:end]

                    # Header: [FrameID:4][ChunkID:2][TotalChunks:2][Data]
                    packet = struct.pack('!IHH', self.frame_id, i, num_chunks) + chunk
                    self.sock.sendto(packet, self.dest)

            self.frame_id += 1

class VideoReceiver:
    """Reçoit et affiche le flux vidéo"""

    def __init__(self, port):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.bind(('0.0.0.0', port))
        self.buffer = {}  # frame_id -> {chunks}

    def receive(self):
        """Reçoit et affiche les frames"""
        while True:
            data, _ = self.sock.recvfrom(65535)

            # Parser le header
            frame_id = struct.unpack('!I', data[:4])[0]
            chunk_id = struct.unpack('!H', data[4:6])[0]

            if len(data) > 6:
                total_chunks = struct.unpack('!H', data[6:8])[0]
                chunk_data = data[8:]
            else:
                total_chunks = 1
                chunk_data = data[6:]

            # Stocker le chunk
            if frame_id not in self.buffer:
                self.buffer[frame_id] = {}

            self.buffer[frame_id][chunk_id] = chunk_data

            # Frame complète ?
            if len(self.buffer[frame_id]) == total_chunks:
                # Reconstruire la frame
                frame_data = b''.join(
                    self.buffer[frame_id][i] for i in range(total_chunks)
                )

                # Décoder et afficher
                nparr = np.frombuffer(frame_data, np.uint8)
                frame = cv2.imdecode(nparr, cv2.IMREAD_COLOR)

                if frame is not None:
                    cv2.imshow('Stream', frame)
                    cv2.waitKey(1)

                # Nettoyer
                del self.buffer[frame_id]

                # Supprimer les vieilles frames (éviter accumulation mémoire)
                old_frames = [fid for fid in self.buffer if fid < frame_id - 10]
                for fid in old_frames:
                    del self.buffer[fid]

# Utilisation
# streamer = VideoStreamer('192.168.1.100', 5000)
# streamer.stream()

# receiver = VideoReceiver(5000)
# receiver.receive()
```

**Pourquoi UDP pour le streaming ?**
- ✅ Latence minimale (pas d'attente de retransmission)
- ✅ Perte acceptable (une frame perdue n'est pas critique)
- ✅ TCP serait trop lent (buffering important)

### 3. Jeu vidéo multijoueur

```python
import socket
import json
import time
import threading

class GameClient:
    """Client de jeu utilisant UDP pour les updates de position"""

    def __init__(self, server_addr, player_id):
        self.server = server_addr
        self.player_id = player_id
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.settimeout(0.016)  # ~60 FPS

        self.position = {'x': 0, 'y': 0}
        self.other_players = {}

        # Thread de réception
        self.running = True
        self.recv_thread = threading.Thread(target=self._receive_updates)
        self.recv_thread.daemon = True
        self.recv_thread.start()

    def send_position(self, x, y):
        """Envoie la position du joueur (envoyé ~60x/sec)"""
        self.position = {'x': x, 'y': y}

        update = {
            'type': 'position',
            'player_id': self.player_id,
            'x': x,
            'y': y,
            'timestamp': time.time()
        }

        data = json.dumps(update).encode('utf-8')
        self.sock.sendto(data, self.server)

    def _receive_updates(self):
        """Reçoit les positions des autres joueurs"""
        while self.running:
            try:
                data, _ = self.sock.recvfrom(4096)
                update = json.loads(data.decode('utf-8'))

                if update['type'] == 'game_state':
                    # Mise à jour de tous les joueurs
                    for player in update['players']:
                        if player['id'] != self.player_id:
                            self.other_players[player['id']] = {
                                'x': player['x'],
                                'y': player['y']
                            }

            except socket.timeout:
                continue
            except Exception as e:
                print(f"Erreur réception: {e}")

    def close(self):
        self.running = False
        self.recv_thread.join()
        self.sock.close()

class GameServer:
    """Serveur de jeu distribuant les états"""

    def __init__(self, port):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.bind(('0.0.0.0', port))

        self.players = {}  # player_id -> {x, y, addr, last_update}

        # Thread de broadcast de l'état
        self.running = True
        self.broadcast_thread = threading.Thread(target=self._broadcast_state)
        self.broadcast_thread.daemon = True
        self.broadcast_thread.start()

    def run(self):
        """Boucle principale de réception"""
        while self.running:
            try:
                data, addr = self.sock.recvfrom(4096)
                update = json.loads(data.decode('utf-8'))

                if update['type'] == 'position':
                    # Mettre à jour la position du joueur
                    self.players[update['player_id']] = {
                        'x': update['x'],
                        'y': update['y'],
                        'addr': addr,
                        'last_update': time.time()
                    }

            except Exception as e:
                print(f"Erreur: {e}")

    def _broadcast_state(self):
        """Diffuse l'état du jeu à tous les joueurs (20x/sec)"""
        while self.running:
            # Nettoyer les joueurs inactifs (> 5 sec)
            now = time.time()
            inactive = [
                pid for pid, info in self.players.items()
                if now - info['last_update'] > 5.0
            ]
            for pid in inactive:
                del self.players[pid]

            # Construire l'état du jeu
            game_state = {
                'type': 'game_state',
                'players': [
                    {'id': pid, 'x': info['x'], 'y': info['y']}
                    for pid, info in self.players.items()
                ]
            }

            data = json.dumps(game_state).encode('utf-8')

            # Envoyer à tous les joueurs
            for player_info in self.players.values():
                self.sock.sendto(data, player_info['addr'])

            time.sleep(0.05)  # 20 updates/sec

# Utilisation
# server = GameServer(9999)
# server.run()

# client = GameClient(('server_ip', 9999), player_id='player1')
# client.send_position(100, 200)
```

**Design decisions pour jeux** :
- UDP pour **positions** (haute fréquence, perte acceptable)
- TCP pour **chat, events critiques** (connexion/déconnexion)
- **Client-side prediction** : compenser la latence
- **Interpolation** : lisser les mouvements avec paquets manquants

### 4. VoIP (Voice over IP)

```python
import socket
import pyaudio
import threading

class VoIPClient:
    """Client VoIP simple utilisant UDP"""

    def __init__(self, peer_addr, peer_port, local_port):
        self.peer = (peer_addr, peer_port)
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.bind(('0.0.0.0', local_port))

        # Audio configuration
        self.CHUNK = 1024
        self.FORMAT = pyaudio.paInt16
        self.CHANNELS = 1
        self.RATE = 16000  # 16 kHz (suffit pour la voix)

        self.audio = pyaudio.PyAudio()

        # Threads d'envoi et réception
        self.running = True
        self.send_thread = threading.Thread(target=self._send_audio)
        self.recv_thread = threading.Thread(target=self._receive_audio)

    def start(self):
        """Démarre la communication"""
        self.send_thread.start()
        self.recv_thread.start()

    def _send_audio(self):
        """Capture et envoie l'audio du micro"""
        stream = self.audio.open(
            format=self.FORMAT,
            channels=self.CHANNELS,
            rate=self.RATE,
            input=True,
            frames_per_buffer=self.CHUNK
        )

        while self.running:
            try:
                # Capturer audio
                data = stream.read(self.CHUNK, exception_on_overflow=False)

                # Envoyer en UDP
                self.sock.sendto(data, self.peer)

            except Exception as e:
                print(f"Erreur envoi: {e}")

        stream.stop_stream()
        stream.close()

    def _receive_audio(self):
        """Reçoit et joue l'audio"""
        stream = self.audio.open(
            format=self.FORMAT,
            channels=self.CHANNELS,
            rate=self.RATE,
            output=True,
            frames_per_buffer=self.CHUNK
        )

        while self.running:
            try:
                # Recevoir audio
                data, _ = self.sock.recvfrom(4096)

                # Jouer
                stream.write(data)

            except Exception as e:
                print(f"Erreur réception: {e}")

        stream.stop_stream()
        stream.close()

    def stop(self):
        """Arrête la communication"""
        self.running = False
        self.send_thread.join()
        self.recv_thread.join()
        self.audio.terminate()
        self.sock.close()

# Utilisation (deux clients)
# client1 = VoIPClient('192.168.1.100', 5001, 5000)
# client1.start()

# client2 = VoIPClient('192.168.1.50', 5000, 5001)
# client2.start()
```

**Améliorations VoIP réelles** :
- **Codec audio** : Opus, G.711 pour compression
- **Jitter buffer** : compenser variations de latence
- **Packet loss concealment** : masquer les pertes
- **Echo cancellation** : éviter feedback
- **VAD** (Voice Activity Detection) : ne pas envoyer le silence

## Optimisations et bonnes pratiques

### 1. Dimensionnement du buffer

```python
# MAUVAIS : buffer trop petit
sock.recvfrom(512)  # Perd des données si datagramme > 512

# BON : buffer adapté
sock.recvfrom(65535)  # Taille max théorique UDP

# OPTIMAL : buffer = MTU du réseau
sock.recvfrom(1500)  # Standard Ethernet
```

### 2. Timeout approprié

```python
# MAUVAIS : pas de timeout (bloque indéfiniment)
sock.recvfrom(4096)

# BON : timeout adapté au contexte
sock.settimeout(1.0)  # DNS query
sock.settimeout(0.1)  # Gaming (besoin de réactivité)
sock.settimeout(5.0)  # Requête API
```

### 3. Gestion de la charge (rate limiting)

```python
import time

class RateLimitedUDPSender:
    """Envoi UDP avec limitation de débit"""

    def __init__(self, max_packets_per_sec=1000):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.max_rate = max_packets_per_sec
        self.min_interval = 1.0 / max_packets_per_sec
        self.last_send = 0

    def send(self, data, dest):
        """Envoie avec rate limiting"""
        # Attendre si nécessaire
        now = time.time()
        elapsed = now - self.last_send

        if elapsed < self.min_interval:
            time.sleep(self.min_interval - elapsed)

        self.sock.sendto(data, dest)
        self.last_send = time.time()

# Utilisation
sender = RateLimitedUDPSender(max_packets_per_sec=100)
for i in range(1000):
    sender.send(f"Packet {i}".encode(), ('server', 9999))
# Envoie 1000 paquets à ~100 pkt/sec au lieu de tout envoyer d'un coup
```

### 4. Options de socket avancées

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Augmenter le buffer de réception (pour haute charge)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 2 * 1024 * 1024)  # 2 MB

# Augmenter le buffer d'envoi
sock.setsockopt(socket.SOL_SOCKET, socket.SO_SNDBUF, 2 * 1024 * 1024)  # 2 MB

# Autoriser la réutilisation du port
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# Priorité IP (Quality of Service)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_TOS, 0x10)  # Low delay
```

### 5. Mode non-bloquant pour haute performance

```python
import socket
import select

# Socket non-bloquante
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.setblocking(False)
sock.bind(('0.0.0.0', 9999))

# Multiplexage avec select
while True:
    readable, _, _ = select.select([sock], [], [], 0.1)

    if sock in readable:
        try:
            data, addr = sock.recvfrom(4096)
            print(f"Reçu: {data}")
        except BlockingIOError:
            pass  # Pas de données disponibles

    # Autres tâches pendant l'attente
    do_other_work()
```

## Comparaison TCP vs UDP : quand utiliser quoi ?

| Critère | TCP | UDP | Recommandation |
|---------|-----|-----|----------------|
| **Fiabilité nécessaire** | ✅ Garantie | ❌ Aucune | TCP pour données critiques |
| **Ordre important** | ✅ Garanti | ❌ Non garanti | TCP pour protocoles séquentiels |
| **Latence critique** | ❌ Overhead | ✅ Minimal | UDP pour temps réel |
| **Streaming** | ❌ Buffering | ✅ Optimal | UDP pour vidéo/audio |
| **Small messages** | ⚠️ Overhead | ✅ Efficace | UDP pour petites requêtes |
| **Large transfers** | ✅ Optimal | ⚠️ Fragmentation | TCP pour gros fichiers |
| **Broadcast/Multicast** | ❌ Impossible | ✅ Support | UDP pour diffusion |
| **Connection pooling** | ✅ Réutilisable | N/A | TCP pour APIs HTTP |

### Exemples de choix

```python
# ✅ UDP approprié
use_cases_udp = [
    "DNS queries",
    "Streaming vidéo/audio (Netflix, YouTube Live, Twitch)",
    "Jeux vidéo multijoueurs (position des joueurs)",
    "VoIP (Skype, Zoom, WhatsApp calls)",
    "IoT sensors (température toutes les 10 sec)",
    "Monitoring/metrics (StatsD, Graphite)",
    "Découverte de services (mDNS, SSDP)",
    "Diffusion de cotations boursières"
]

# ✅ TCP approprié
use_cases_tcp = [
    "HTTP/HTTPS (pages web, APIs REST)",
    "Transfert de fichiers (FTP, SFTP)",
    "Email (SMTP, IMAP, POP3)",
    "Bases de données (MySQL, PostgreSQL)",
    "SSH / Telnet",
    "Chat avec historique (IRC, Slack)",
    "Transactions financières",
    "Tout ce qui nécessite fiabilité absolue"
]
```

## Pièges courants avec UDP

### ❌ Piège 1 : Supposer que tout arrive

```python
# MAUVAIS
for i in range(100):
    sock.sendto(f"Message {i}".encode(), dest)
# Espère recevoir les 100 messages → Faux!

# BON : Accepter les pertes ou implémenter ACKs
```

### ❌ Piège 2 : Buffer trop petit

```python
# MAUVAIS
data, addr = sock.recvfrom(512)
# Si datagramme = 1000 octets → 512 reçus, 488 perdus!

# BON
data, addr = sock.recvfrom(65535)
```

### ❌ Piège 3 : Pas de timeout

```python
# MAUVAIS
data, addr = sock.recvfrom(4096)  # Bloque indéfiniment

# BON
sock.settimeout(5.0)
try:
    data, addr = sock.recvfrom(4096)
except socket.timeout:
    handle_timeout()
```

### ❌ Piège 4 : Envoyer des datagrammes trop gros

```python
# MAUVAIS
sock.sendto(b"x" * 10000, dest)  # Fragmentation IP → risque de perte

# BON : respecter MTU
if len(data) > 1400:
    # Fragmenter manuellement ou utiliser TCP
```

### ❌ Piège 5 : Oublier que l'ordre n'est pas garanti

```python
# MAUVAIS
sock.sendto(b"Step 1", dest)
sock.sendto(b"Step 2", dest)
sock.sendto(b"Step 3", dest)
# Peut arriver : Step 2, Step 1, Step 3

# BON : Ajouter des numéros de séquence si l'ordre compte
```

## Exemple complet : serveur de metrics (StatsD-like)

```python
import socket
import time
import threading
from collections import defaultdict

class MetricsServer:
    """Serveur de collecte de métriques via UDP (type StatsD)"""

    def __init__(self, port=8125):
        self.port = port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.bind(('0.0.0.0', port))

        # Stockage des métriques
        self.counters = defaultdict(int)
        self.gauges = {}
        self.timers = defaultdict(list)

        self.running = True

        # Thread d'agrégation et flush périodique
        self.flush_thread = threading.Thread(target=self._flush_periodically)
        self.flush_thread.daemon = True
        self.flush_thread.start()

    def run(self):
        """Boucle principale de réception"""
        print(f"Serveur de métriques démarré sur :{self.port}")

        while self.running:
            try:
                data, addr = self.sock.recvfrom(1024)
                self._parse_metric(data.decode('utf-8'))
            except Exception as e:
                # UDP : on ignore les erreurs et continue
                pass

    def _parse_metric(self, metric_str):
        """
        Parse une métrique au format StatsD:
        - Counter: metric.name:value|c
        - Gauge: metric.name:value|g
        - Timer: metric.name:value|ms
        """
        try:
            parts = metric_str.strip().split('|')
            name_value = parts[0].split(':')
            name = name_value[0]
            value = float(name_value[1])
            metric_type = parts[1]

            if metric_type == 'c':
                # Counter
                self.counters[name] += value
            elif metric_type == 'g':
                # Gauge
                self.gauges[name] = value
            elif metric_type == 'ms':
                # Timer
                self.timers[name].append(value)

        except Exception as e:
            # Métrique malformée : ignorée (UDP = best effort)
            pass

    def _flush_periodically(self):
        """Flush les métriques toutes les 10 secondes"""
        while self.running:
            time.sleep(10)
            self._flush_metrics()

    def _flush_metrics(self):
        """Agrège et affiche les métriques"""
        print("\n=== METRICS ===")

        # Counters
        for name, value in self.counters.items():
            print(f"Counter {name}: {value}")

        # Gauges
        for name, value in self.gauges.items():
            print(f"Gauge {name}: {value}")

        # Timers (afficher moyenne, min, max, percentiles)
        for name, values in self.timers.items():
            if values:
                avg = sum(values) / len(values)
                print(f"Timer {name}: avg={avg:.2f}ms, count={len(values)}")

        # Reset des counters et timers
        self.counters.clear()
        self.timers.clear()
        # Gauges persistent

class MetricsClient:
    """Client pour envoyer des métriques"""

    def __init__(self, host='localhost', port=8125):
        self.addr = (host, port)
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    def increment(self, metric_name, value=1):
        """Incrémente un counter"""
        self._send(f"{metric_name}:{value}|c")

    def gauge(self, metric_name, value):
        """Définit une gauge"""
        self._send(f"{metric_name}:{value}|g")

    def timing(self, metric_name, value_ms):
        """Enregistre un timing"""
        self._send(f"{metric_name}:{value_ms}|ms")

    def _send(self, metric):
        """Envoie une métrique (fire-and-forget)"""
        try:
            self.sock.sendto(metric.encode('utf-8'), self.addr)
        except:
            # Échec d'envoi : on ignore (UDP = best effort)
            pass

# Utilisation
if __name__ == "__main__":
    # Serveur
    # server = MetricsServer(port=8125)
    # server.run()

    # Client (dans votre application)
    metrics = MetricsClient()

    # Simuler des métriques
    import random
    for i in range(100):
        metrics.increment('requests.total')
        metrics.timing('api.response_time', random.randint(10, 200))
        metrics.gauge('users.online', random.randint(100, 500))
        time.sleep(0.1)
```

**Pourquoi UDP pour les métriques ?**
- ✅ **Fire-and-forget** : pas d'impact sur l'application si le serveur est down
- ✅ **Performance** : pas de latence d'ACK
- ✅ **Simplicité** : pas de gestion de connexions
- ✅ **Volume** : des milliers de métriques/sec
- ⚠️ **Perte acceptable** : perdre quelques métriques n'est pas critique

## Récapitulatif

| Aspect | UDP | Points clés |
|--------|-----|-------------|
| **Création** | SOCK_DGRAM | Même API que TCP, type différent |
| **Envoi** | sendto(data, addr) | Spécifier destination à chaque fois |
| **Réception** | recvfrom(size) | Retourne données + adresse expéditeur |
| **Connexion** | Aucune (optionnel connect) | Sans état, pas de handshake |
| **Fiabilité** | Application | Implémenter ACK/retry si nécessaire |
| **Ordre** | Non garanti | Numéros de séquence si important |
| **Taille** | < MTU recommandé | ~1400 octets pour éviter fragmentation |
| **Timeout** | Crucial | Toujours définir un timeout |
| **Broadcast** | Oui (SO_BROADCAST) | 255.255.255.255 ou subnet broadcast |
| **Multicast** | Oui (IP_ADD_MEMBERSHIP) | Groupes 224.0.0.0 - 239.255.255.255 |

## Points clés à retenir

✅ **UDP = vitesse avant tout**, pas de garanties

✅ **sendto/recvfrom** incluent l'adresse dans chaque datagramme

✅ **Pas de connexion**, chaque datagramme est indépendant

✅ **Frontières de messages respectées** (contrairement à TCP)

✅ **Buffer size crucial** : >= taille datagramme pour éviter pertes

✅ **Timeout obligatoire** : aucune garantie de réponse

✅ **Perte, désordre, duplication possibles** : l'application doit gérer

✅ **Idéal pour** : streaming, gaming, DNS, métriques, temps réel

✅ **À éviter pour** : transferts critiques, gros fichiers, transactions

## Prochaines étapes

Maintenant que vous maîtrisez UDP et TCP, nous allons voir :

- **Section 8.4** : Gestion robuste des erreurs réseau (timeouts, reconnexions, retry logic)
- **Section 8.5** : I/O non-bloquant vs bloquant
- **Section 8.6** : Multiplexage (select, poll, epoll) pour gérer des milliers de connexions
- **Section 8.7** : Programmation asynchrone et event-driven

---


⏭️ [Gestion des erreurs réseau](/08-programmation-reseau/04-gestion-erreurs.md)
