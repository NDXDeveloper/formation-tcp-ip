🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.2 Multicast et IGMP

## Introduction

Le **multicast** est une technique de transmission réseau permettant d'envoyer des données **d'une source vers plusieurs destinataires simultanément**, en utilisant un seul flux réseau. C'est l'équivalent réseau d'une diffusion radio : un émetteur, plusieurs récepteurs, une seule émission.

### L'analogie de la conférence

Imaginez trois façons d'organiser une présentation pour 1000 personnes :

**Unicast (1-to-1) :**
```
Présentateur → Appel individuel avec personne 1
Présentateur → Appel individuel avec personne 2
...
Présentateur → Appel individuel avec personne 1000
Résultat : 1000 appels téléphoniques simultanés
Coût : Énorme bande passante
```

**Broadcast (1-to-all) :**
```
Présentateur → Haut-parleur géant qui hurle
                (tout le monde entend, même ceux qui ne veulent pas)
Résultat : Pollution du réseau
Coût : Acceptable mais bruyant
```

**Multicast (1-to-many) :**
```
Présentateur → Salle de conférence
                (seuls ceux qui s'inscrivent entrent)
Résultat : Efficace et ciblé
Coût : Optimal
```

### Le problème que résout le multicast

Sans multicast, pour envoyer une vidéo live à 10 000 spectateurs :

```
Serveur ──[Stream 1]──→ Utilisateur 1
       ──[Stream 2]──→ Utilisateur 2
       ──[Stream 3]──→ Utilisateur 3
       ...
       ──[Stream 10000]──→ Utilisateur 10000

Bande passante nécessaire : 10 000 × 5 Mbps = 50 Gbps !
```

Avec multicast :

```
Serveur ──[Stream unique]──→ Réseau multicast
                              ↓
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
              User 1    User 2  ...  User 10000

Bande passante nécessaire : 1 × 5 Mbps = 5 Mbps
Économie : 99.99% de bande passante
```

## Unicast vs Broadcast vs Multicast vs Anycast

### Comparaison visuelle

```
┌──────────────────────────────────────────────────────────┐
│                      UNICAST                             │
│  Sender → [paquet] → Receiver                            │
│  (1 paquet = 1 destinataire)                             │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                     BROADCAST                            │
│           ┌─→ R1                                         │
│           ├─→ R2                                         │
│  Sender ──┼─→ R3  (TOUT LE MONDE sur le subnet)          │
│           ├─→ R4                                         │
│           └─→ R5                                         │
│  (1 paquet = tous les hôtes du réseau local)             │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                     MULTICAST                            │
│           ┌─→ R1 (inscrit au groupe)                     │
│  Sender ──┼─→ R3 (inscrit au groupe)                     │
│           └─→ R5 (inscrit au groupe)                     │
│  R2 et R4 ne sont PAS inscrits → ne reçoivent rien       │
│  (1 paquet = membres du groupe uniquement)               │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                      ANYCAST                             │
│           ┌─→ R1 (le plus proche géographiquement)       │
│  Sender ──┤                                              │
│           └─X R2 (même adresse IP mais plus loin)        │
│  (1 paquet = 1 destinataire parmi plusieurs identiques)  │
└──────────────────────────────────────────────────────────┘
```

### Tableau de comparaison

| Critère | Unicast | Broadcast | Multicast | Anycast |
|---------|---------|-----------|-----------|---------|
| **Adresse dest** | IP unique | 255.255.255.255 | 224.0.0.0-239.255.255.255 | Partagée |
| **Nb destinataires** | 1 | Tous (subnet) | Groupe (opt-in) | 1 (le plus proche) |
| **Efficacité réseau** | Faible (N flux) | Moyenne | Très élevée (1 flux) | Élevée |
| **Utilisation** | Web, email | DHCP, ARP | IPTV, streaming | CDN, DNS |
| **Portée** | Globale | Locale | Réseau supportant multicast | Globale |

## Adressage multicast

### Plages IPv4 multicast (Classe D)

L'espace d'adressage **224.0.0.0 à 239.255.255.255** est réservé au multicast.

```
Format binaire d'une adresse multicast IPv4 :
1110xxxx.xxxxxxxx.xxxxxxxx.xxxxxxxx
└─┬─┘
  └─ Les 4 premiers bits = 1110 (identifie le multicast)

Plage complète : 224.0.0.0/4
```

#### Plages spécifiques

| Plage | Usage | Portée | Exemples |
|-------|-------|--------|----------|
| **224.0.0.0/24** | Lien local (bien connu) | Segment local | Protocoles de routage |
| 224.0.0.1 | Tous les hôtes | Lien local | Ping multicast |
| 224.0.0.2 | Tous les routeurs | Lien local | OSPF |
| 224.0.0.5 | Routeurs OSPF | Lien local | OSPF Hello |
| 224.0.0.9 | RIP-2 | Lien local | RIP updates |
| 224.0.0.13 | PIM | Lien local | PIM-DM |
| 224.0.0.22 | IGMP | Lien local | IGMPv3 |
| **224.0.1.0/24** | Internetwork control | Global | NTP, etc. |
| **224.0.2.0 - 224.0.255.255** | Réservé IANA | - | Applications |
| **232.0.0.0/8** | Source-Specific Multicast (SSM) | Global | IPTV moderne |
| **233.0.0.0/8** | GLOP (AS-based) | Global | Par organisation |
| **234.0.0.0 - 238.255.255.255** | Usage libre | Global | Applications |
| **239.0.0.0/8** | Scope limité (privé) | Configurable | LAN entreprise |

#### Adresses bien connues pour développeurs

| Adresse | Service | Usage |
|---------|---------|-------|
| 224.0.0.251 | mDNS | Service discovery (Bonjour, Avahi) |
| 224.0.0.252 | LLMNR | Link-Local Multicast Name Resolution (Windows) |
| 224.0.1.1 | NTP | Network Time Protocol |
| 239.255.255.250 | SSDP | Simple Service Discovery Protocol (UPnP) |
| 224.0.23.0 | VRRP | Virtual Router Redundancy Protocol |

### IPv6 multicast

En IPv6, le multicast est encore plus important (il remplace le broadcast qui n'existe plus).

```
Format IPv6 multicast :
FF00::/8
││││
││││└─ Groupe ID (112 bits)
│││└── Scope (4 bits)
││└─── Flags (4 bits)
│└──── 1111 1111 (FF) = préfixe multicast
```

#### Scopes IPv6

| Scope | Valeur | Nom | Portée |
|-------|--------|-----|--------|
| 1 | FF01:: | Interface-local | Même interface |
| 2 | FF02:: | Link-local | Même lien |
| 4 | FF04:: | Admin-local | Admin définit |
| 5 | FF05:: | Site-local | Organisation |
| 8 | FF08:: | Organization | Plusieurs sites |
| E | FF0E:: | Global | Internet |

#### Adresses IPv6 bien connues

| Adresse | Usage |
|---------|-------|
| FF02::1 | Tous les nœuds (équivalent broadcast) |
| FF02::2 | Tous les routeurs |
| FF02::5 | Routeurs OSPF |
| FF02::1:2 | DHCP agents |
| FF02::1:FF00:0/104 | Solicited-node multicast (NDP) |

### Mapping MAC multicast

Les adresses multicast IP sont mappées sur des adresses MAC multicast.

#### IPv4 → MAC multicast

```
Adresse IP multicast :  224.1.2.3
                        └─┬─┘ └──┬──┘
                          │      │
En binaire (23 bits):   0.00001.00000010.00000011
                          └──────┬──────────────┘
                                 │
Préfixe MAC multicast : 01:00:5E:0 (25 bits IEEE)
                                  └──┬───┘
                                     │
MAC résultante :        01:00:5E:01:02:03
                        └──────┬──────────┘ └─┬──┘
                            Préfixe      23 bits IP
```

**Important :** Seuls les **23 bits de poids faible** de l'IP sont mappés, donc plusieurs adresses IP peuvent avoir la même MAC multicast !

```
Exemple de collision :
224.1.2.3   → 01:00:5E:01:02:03
224.129.2.3 → 01:00:5E:01:02:03  (même MAC !)
225.1.2.3   → 01:00:5E:01:02:03  (même MAC !)
```

## IGMP : Internet Group Management Protocol

**IGMP** est le protocole qui permet aux hôtes de **s'inscrire et se désinscrire** des groupes multicast. Il fonctionne entre les hôtes et leurs routeurs directement connectés.

### Versions d'IGMP

| Version | RFC | Année | Caractéristiques | Usage actuel |
|---------|-----|-------|------------------|--------------|
| **IGMPv1** | RFC 1112 | 1989 | Base, join uniquement | Obsolète |
| **IGMPv2** | RFC 2236 | 1997 | Ajout leave, élection | Largement déployé |
| **IGMPv3** | RFC 3376 | 2002 | Source filtering (SSM) | Standard moderne |

### Fonctionnement d'IGMPv2 (le plus courant)

#### Les types de messages IGMP

```
┌────────────────────────────────────────────────┐
│  Type  │  Description                          │
├────────┼───────────────────────────────────────┤
│  0x11  │  Membership Query (routeur → hôtes)   │
│  0x16  │  Membership Report v2 (hôte → routeur)│
│  0x17  │  Leave Group (hôte → routeur)         │
│  0x22  │  Report v3 (IGMPv3)                   │
└────────────────────────────────────────────────┘
```

#### Le cycle de vie d'une adhésion multicast

```
Étape 1 : JOIN (hôte rejoint un groupe)
═══════════════════════════════════════

Hôte A                          Routeur                    Source
  │                                │                           │
  │ [IGMP Report 224.1.1.1]        │                           │
  ├───────────────────────────────>│                           │
  │                                │  Configure interface      │
  │                                │  pour recevoir 224.1.1.1  │
  │                                │                           │
  │                                │  [PIM Join 224.1.1.1]     │
  │                                ├──────────────────────────>│
  │                                │                           │
  │  <─────── Trafic multicast commence ─────────────────────  │
  │                                │                           │


Étape 2 : QUERY (routeur vérifie qui écoute)
═══════════════════════════════════════════

Routeur                         Hôte A              Hôte B
  │                                │                   │
  │ [General Query: qui écoute?]   │                   │
  ├───────────────────────────────>│                   │
  ├──────────────────────────────────────────────────> │
  │                                │                   │
  │    [Report: j'écoute 224.1.1.1]│                   │
  │<───────────────────────────────┤                   │
  │                                │                   │
  │ (B ne répond pas → a quitté le groupe)             │


Étape 3 : LEAVE (hôte quitte le groupe)
════════════════════════════════════════

Hôte A                          Routeur
  │                                │
  │ [IGMP Leave 224.1.1.1]         │
  ├───────────────────────────────>│
  │                                │
  │      [Specific Query 224.1.1.1]│
  │<───────────────────────────────┤
  │  (pour vérifier si d'autres    │
  │   hôtes écoutent encore)       │
  │                                │
  │  (pas de réponse)              │
  │                                │
  │                        Stop forwarding
  │                        224.1.1.1
```

### Format du paquet IGMP

```
En-tête IPv4
┌─────────────────────────────────────────────┐
│ Version=4 │ IHL │ ToS │ Total Length        │
├───────────┴─────┴─────┴─────────────────────┤
│ IP ID                 │ Flags │ Fragment    │
├───────────────────────┴───────┴─────────────┤
│ TTL=1    │ Protocol=2 │ Header Checksum     │
├──────────┴────────────┴─────────────────────┤
│ Source IP (adresse de l'hôte)               │
├─────────────────────────────────────────────┤
│ Destination IP (224.0.0.22 pour IGMPv3)     │
└─────────────────────────────────────────────┘

IGMP Message
┌─────────────────────────────────────────────┐
│ Type (1 octet)      │ Max Resp Time         │
├─────────────────────┴───────────────────────┤
│ Checksum                                    │
├─────────────────────────────────────────────┤
│ Group Address (adresse multicast)           │
└─────────────────────────────────────────────┘
```

**Particularités :**
- **TTL = 1** : Les messages IGMP ne traversent jamais de routeur
- **IP Options** : Router Alert (option 148) pour forcer inspection par routeur

## Implémentation : Sockets multicast

### Python : Recevoir du multicast

```python
import socket
import struct

def receive_multicast(multicast_group, port, interface='0.0.0.0'):
    """
    Recevoir des données multicast

    Args:
        multicast_group: Adresse IP multicast (ex: '239.1.1.1')
        port: Port d'écoute
        interface: Interface réseau (IP), '0.0.0.0' = toutes
    """
    # Créer un socket UDP
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)

    # Permettre la réutilisation de l'adresse (important pour multicast)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    # Sur certains systèmes (BSD, macOS), aussi nécessaire
    try:
        sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEPORT, 1)
    except AttributeError:
        pass  # SO_REUSEPORT n'existe pas sur tous les OS

    # Binder sur le port (IMPORTANT : 0.0.0.0, pas l'adresse multicast)
    sock.bind(('', port))

    # Construire la requête de membership IGMP
    # struct mreq { in_addr imr_multiaddr; in_addr imr_interface; }
    mreq = struct.pack('4s4s',
                       socket.inet_aton(multicast_group),
                       socket.inet_aton(interface))

    # Joindre le groupe multicast (envoie IGMP Membership Report)
    sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)

    print(f"✓ Inscrit au groupe multicast {multicast_group}:{port}")
    print(f"  Interface : {interface}")
    print("  En attente de données...")

    try:
        while True:
            data, address = sock.recvfrom(10240)
            print(f"\n[{address[0]}:{address[1]}] Reçu {len(data)} octets:")
            print(f"  {data.decode('utf-8', errors='ignore')[:100]}")

    except KeyboardInterrupt:
        print("\nArrêt...")

    finally:
        # Quitter le groupe (envoie IGMP Leave)
        sock.setsockopt(socket.IPPROTO_IP, socket.IP_DROP_MEMBERSHIP, mreq)
        sock.close()
        print("✓ Quitté le groupe multicast")

# Exemple d'utilisation
if __name__ == '__main__':
    MCAST_GROUP = '239.1.1.1'
    MCAST_PORT = 5007

    receive_multicast(MCAST_GROUP, MCAST_PORT)
```

### Python : Envoyer en multicast

```python
import socket
import time

def send_multicast(multicast_group, port, message, ttl=2):
    """
    Envoyer des données en multicast

    Args:
        multicast_group: Adresse IP multicast de destination
        port: Port de destination
        message: Message à envoyer (str ou bytes)
        ttl: Time-To-Live (nombre de sauts routeurs autorisés)
             1 = réseau local seulement
             2 = traverser 1 routeur
             32 = portée régionale
             255 = portée globale
    """
    # Créer un socket UDP
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.IPPROTO_UDP)

    # Configurer le TTL multicast
    # TTL = 1 : pas de routage (lien local seulement)
    # TTL > 1 : autorise le routage multicast
    sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, ttl)

    # Optionnel : désactiver le loopback local
    # (pour ne pas recevoir nos propres messages)
    sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_LOOP, 0)

    # Optionnel : spécifier l'interface source
    # sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_IF,
    #                 socket.inet_aton('192.168.1.100'))

    if isinstance(message, str):
        message = message.encode('utf-8')

    try:
        print(f"Envoi vers {multicast_group}:{port} (TTL={ttl})")
        sock.sendto(message, (multicast_group, port))
        print(f"✓ Envoyé : {len(message)} octets")

    finally:
        sock.close()

# Exemple : diffusion périodique
def heartbeat_multicast():
    """Envoyer un heartbeat régulier en multicast"""
    MCAST_GROUP = '239.1.1.1'
    MCAST_PORT = 5007

    counter = 0
    while True:
        message = f"Heartbeat #{counter} from server - timestamp: {time.time()}"
        send_multicast(MCAST_GROUP, MCAST_PORT, message)
        counter += 1
        time.sleep(2)

if __name__ == '__main__':
    heartbeat_multicast()
```

### Go : Listener multicast

```go
package main

import (
    "fmt"
    "net"
    "os"
)

func receiveMulticast(group string, port int) error {
    // Parser l'adresse multicast
    addr, err := net.ResolveUDPAddr("udp", fmt.Sprintf("%s:%d", group, port))
    if err != nil {
        return err
    }

    // Créer un listener UDP multicast
    conn, err := net.ListenMulticastUDP("udp", nil, addr)
    if err != nil {
        return err
    }
    defer conn.Close()

    fmt.Printf("✓ Écoute sur %s:%d\n", group, port)

    // Buffer pour recevoir les données
    buffer := make([]byte, 10240)

    for {
        n, src, err := conn.ReadFromUDP(buffer)
        if err != nil {
            fmt.Printf("Erreur lecture: %v\n", err)
            continue
        }

        fmt.Printf("[%s] Reçu %d octets: %s\n",
                   src.String(), n, string(buffer[:n]))
    }
}

func main() {
    if err := receiveMulticast("239.1.1.1", 5007); err != nil {
        fmt.Fprintf(os.Stderr, "Erreur: %v\n", err)
        os.Exit(1)
    }
}
```

### Go : Sender multicast

```go
package main

import (
    "fmt"
    "net"
    "time"
)

func sendMulticast(group string, port int, message string) error {
    // Parser l'adresse de destination
    addr, err := net.ResolveUDPAddr("udp", fmt.Sprintf("%s:%d", group, port))
    if err != nil {
        return err
    }

    // Créer une connexion UDP
    conn, err := net.DialUDP("udp", nil, addr)
    if err != nil {
        return err
    }
    defer conn.Close()

    // Envoyer le message
    _, err = conn.Write([]byte(message))
    return err
}

func heartbeat() {
    group := "239.1.1.1"
    port := 5007
    counter := 0

    for {
        message := fmt.Sprintf("Heartbeat #%d - %s", counter, time.Now())

        if err := sendMulticast(group, port, message); err != nil {
            fmt.Printf("Erreur envoi: %v\n", err)
        } else {
            fmt.Printf("✓ Envoyé: %s\n", message)
        }

        counter++
        time.Sleep(2 * time.Second)
    }
}

func main() {
    heartbeat()
}
```

### JavaScript/Node.js : Multicast

```javascript
const dgram = require('dgram');

// ============== RECEIVER ==============
function receiveMulticast(group, port) {
    const socket = dgram.createSocket({ type: 'udp4', reuseAddr: true });

    socket.on('listening', () => {
        const address = socket.address();
        console.log(`✓ Socket UDP écoute sur ${address.address}:${address.port}`);

        // Rejoindre le groupe multicast
        socket.addMembership(group);
        console.log(`✓ Rejoint le groupe multicast ${group}`);
    });

    socket.on('message', (msg, rinfo) => {
        console.log(`[${rinfo.address}:${rinfo.port}] ${msg.toString()}`);
    });

    socket.on('error', (err) => {
        console.error('Erreur socket:', err);
        socket.close();
    });

    // Bind sur le port (pas sur l'adresse multicast)
    socket.bind(port);
}

// ============== SENDER ==============
function sendMulticast(group, port, message) {
    const socket = dgram.createSocket('udp4');

    // Configurer le TTL multicast
    socket.setMulticastTTL(2);

    // Désactiver le loopback (ne pas recevoir ses propres messages)
    socket.setMulticastLoopback(false);

    const buffer = Buffer.from(message);

    socket.send(buffer, 0, buffer.length, port, group, (err) => {
        if (err) {
            console.error('Erreur envoi:', err);
        } else {
            console.log(`✓ Envoyé ${buffer.length} octets vers ${group}:${port}`);
        }
        socket.close();
    });
}

// Heartbeat périodique
function heartbeat(group, port) {
    let counter = 0;

    setInterval(() => {
        const message = `Heartbeat #${counter} - ${new Date().toISOString()}`;
        sendMulticast(group, port, message);
        counter++;
    }, 2000);
}

// Usage
const MCAST_GROUP = '239.1.1.1';
const MCAST_PORT = 5007;

// Décommenter selon besoin
// receiveMulticast(MCAST_GROUP, MCAST_PORT);
// heartbeat(MCAST_GROUP, MCAST_PORT);
```

### Java : Multicast socket

```java
import java.net.*;
import java.io.*;

public class MulticastReceiver {
    public static void main(String[] args) {
        String group = "239.1.1.1";
        int port = 5007;

        try {
            // Créer un MulticastSocket
            MulticastSocket socket = new MulticastSocket(port);

            // Rejoindre le groupe
            InetAddress groupAddr = InetAddress.getByName(group);
            socket.joinGroup(groupAddr);

            System.out.println("✓ Rejoint " + group + ":" + port);

            byte[] buffer = new byte[10240];

            while (true) {
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                socket.receive(packet);

                String message = new String(
                    packet.getData(), 0, packet.getLength()
                );

                System.out.printf("[%s:%d] %s%n",
                    packet.getAddress().getHostAddress(),
                    packet.getPort(),
                    message
                );
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Cas d'usage réels pour développeurs

### Cas 1 : Service Discovery (mDNS)

**Contexte :** Découvrir automatiquement des services sur le réseau local sans configuration.

**Protocole :** mDNS (Multicast DNS) - utilisé par Bonjour (Apple), Avahi (Linux), Zeroconf.

```python
import socket
import struct
import time

def mdns_query(service_type='_http._tcp.local'):
    """
    Envoyer une requête mDNS pour découvrir des services

    mDNS utilise 224.0.0.251:5353
    """
    MCAST_GRP = '224.0.0.251'
    MCAST_PORT = 5353

    # Créer un socket multicast
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 1)

    # Construire une requête DNS basique pour le service
    # Format DNS simplifié (normalement utiliser une lib comme dnspython)
    query = build_dns_query(service_type)

    print(f"Recherche de services {service_type}...")
    sock.sendto(query, (MCAST_GRP, MCAST_PORT))

    # Écouter les réponses
    sock.settimeout(2.0)

    try:
        while True:
            data, addr = sock.recvfrom(10240)
            print(f"Réponse de {addr[0]}: {len(data)} octets")
            # Parser la réponse DNS...
    except socket.timeout:
        print("Fin de la découverte")

    sock.close()

def build_dns_query(name):
    """Construire une requête DNS simple (PTR query)"""
    # Transaction ID
    tid = 0x0000

    # Flags: standard query
    flags = 0x0000

    # Questions: 1, Answers: 0, Authority: 0, Additional: 0
    qdcount = 1
    ancount = 0
    nscount = 0
    arcount = 0

    # Header (12 octets)
    header = struct.pack('!HHHHHH', tid, flags, qdcount, ancount, nscount, arcount)

    # Question section
    # Encoder le nom en format DNS (longueur + label)
    question = b''
    for part in name.split('.'):
        question += bytes([len(part)]) + part.encode('ascii')
    question += b'\x00'  # Null terminator

    # Type PTR (12), Class IN (1)
    question += struct.pack('!HH', 12, 1)

    return header + question

# Exemple d'utilisation avec Zeroconf (bibliothèque)
from zeroconf import ServiceBrowser, Zeroconf

class ServiceListener:
    def add_service(self, zeroconf, service_type, name):
        print(f"✓ Service découvert: {name}")
        info = zeroconf.get_service_info(service_type, name)
        if info:
            print(f"  Adresse : {socket.inet_ntoa(info.addresses[0])}")
            print(f"  Port    : {info.port}")
            print(f"  Props   : {info.properties}")

    def remove_service(self, zeroconf, service_type, name):
        print(f"✗ Service retiré: {name}")

# Découvrir tous les services HTTP
zeroconf = Zeroconf()
listener = ServiceListener()
browser = ServiceBrowser(zeroconf, "_http._tcp.local.", listener)

try:
    input("Appuyez sur Entrée pour arrêter...\n")
finally:
    zeroconf.close()
```

### Cas 2 : Streaming vidéo/audio (IPTV)

**Contexte :** Diffuser un flux vidéo à des milliers de clients sans surcharger le réseau.

**Protocole :** RTP (Real-time Transport Protocol) sur multicast.

```python
import socket
import time
import struct

class MulticastVideoStreamer:
    """
    Streamer vidéo multicast simple
    En production, utiliser FFmpeg ou GStreamer
    """

    def __init__(self, mcast_group, port, source_file):
        self.group = mcast_group
        self.port = port
        self.source = source_file

        # Socket multicast
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 32)

        # RTP sequence number
        self.seq_num = 0
        self.timestamp = 0
        self.ssrc = 0x12345678  # Synchronization source identifier

    def create_rtp_packet(self, payload):
        """
        Créer un paquet RTP

        RTP Header (12 octets):
        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        |V=2|P|X|  CC   |M|     PT      |       sequence number         |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        |                           timestamp                           |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        |           synchronization source (SSRC) identifier            |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        """
        # Octet 0: V(2) P(1) X(1) CC(4)
        byte0 = 0x80  # Version 2, no padding, no extension, 0 CSRC

        # Octet 1: M(1) PT(7) - Payload Type (ex: 96 pour H.264)
        byte1 = 96  # H.264

        # Header
        header = struct.pack('!BBHII',
            byte0,              # V, P, X, CC
            byte1,              # M, PT
            self.seq_num,       # Sequence number
            self.timestamp,     # Timestamp
            self.ssrc           # SSRC
        )

        self.seq_num = (self.seq_num + 1) & 0xFFFF
        self.timestamp += 3000  # Incrément selon framerate

        return header + payload

    def stream(self, chunk_size=1316):
        """
        Streamer le fichier en multicast
        MTU Ethernet = 1500
        IP header = 20
        UDP header = 8
        RTP header = 12
        Payload max = 1500 - 20 - 8 - 12 = 1460
        """
        print(f"Streaming vers {self.group}:{self.port}")

        with open(self.source, 'rb') as f:
            while True:
                chunk = f.read(chunk_size)
                if not chunk:
                    break

                packet = self.create_rtp_packet(chunk)
                self.sock.sendto(packet, (self.group, self.port))

                # Simule 30 FPS
                time.sleep(1/30)

        print("Fin du stream")
        self.sock.close()

# Utilisation
if __name__ == '__main__':
    streamer = MulticastVideoStreamer(
        mcast_group='232.1.1.1',  # SSM range
        port=5004,
        source_file='video.h264'
    )
    streamer.stream()
```

**Receiver côté client :**

```python
import socket
import struct

class MulticastVideoReceiver:
    def __init__(self, mcast_group, port, output_file):
        self.group = mcast_group
        self.port = port
        self.output = output_file

        # Créer socket multicast
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.sock.bind(('', port))

        # Joindre le groupe
        mreq = struct.pack('4sl', socket.inet_aton(mcast_group), socket.INADDR_ANY)
        self.sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)

    def parse_rtp_header(self, packet):
        """Parser le header RTP"""
        if len(packet) < 12:
            return None, None

        # Unpacker le header
        byte0, byte1, seq, timestamp, ssrc = struct.unpack('!BBHII', packet[:12])

        version = (byte0 >> 6) & 0x03
        payload_type = byte1 & 0x7F

        return {
            'version': version,
            'seq': seq,
            'timestamp': timestamp,
            'ssrc': ssrc,
            'pt': payload_type
        }, packet[12:]  # Payload

    def receive(self):
        print(f"Réception depuis {self.group}:{self.port}")

        with open(self.output, 'wb') as f:
            packet_count = 0

            while True:
                data, addr = self.sock.recvfrom(65535)

                header, payload = self.parse_rtp_header(data)
                if header and payload:
                    f.write(payload)
                    packet_count += 1

                    if packet_count % 100 == 0:
                        print(f"Reçu {packet_count} paquets (seq={header['seq']})")

# Usage
receiver = MulticastVideoReceiver('232.1.1.1', 5004, 'received_video.h264')
receiver.receive()
```

### Cas 3 : Cache invalidation distribué

**Contexte :** Invalider le cache de plusieurs serveurs simultanément quand une donnée change.

```python
import socket
import json
import struct
import threading
import time
from datetime import datetime

class DistributedCacheInvalidator:
    """
    Système d'invalidation de cache via multicast
    Évite les requêtes HTTP/API vers chaque serveur
    """

    def __init__(self, mcast_group='239.255.1.1', port=9999):
        self.group = mcast_group
        self.port = port
        self.cache = {}  # Cache local
        self.cache_lock = threading.Lock()

        # Socket pour envoyer
        self.send_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.send_sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 2)

        # Socket pour recevoir
        self.recv_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.recv_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.recv_sock.bind(('', port))

        # Rejoindre le groupe
        mreq = struct.pack('4sl', socket.inet_aton(mcast_group), socket.INADDR_ANY)
        self.recv_sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_MEMBERSHIP, mreq)

        # Thread d'écoute
        self.listen_thread = threading.Thread(target=self._listen_invalidations, daemon=True)
        self.listen_thread.start()

    def set(self, key, value):
        """Mettre une valeur en cache (localement seulement)"""
        with self.cache_lock:
            self.cache[key] = {
                'value': value,
                'timestamp': time.time()
            }
        print(f"✓ Cache SET: {key} = {value}")

    def get(self, key):
        """Récupérer une valeur du cache"""
        with self.cache_lock:
            return self.cache.get(key, {}).get('value')

    def invalidate(self, key):
        """
        Invalider une clé dans TOUS les caches (multicast)
        """
        message = {
            'action': 'invalidate',
            'key': key,
            'timestamp': datetime.utcnow().isoformat(),
            'sender': socket.gethostname()
        }

        data = json.dumps(message).encode('utf-8')
        self.send_sock.sendto(data, (self.group, self.port))

        print(f"📢 Invalidation multicast envoyée: {key}")

    def invalidate_pattern(self, pattern):
        """Invalider toutes les clés matchant un pattern"""
        message = {
            'action': 'invalidate_pattern',
            'pattern': pattern,
            'timestamp': datetime.utcnow().isoformat(),
            'sender': socket.gethostname()
        }

        data = json.dumps(message).encode('utf-8')
        self.send_sock.sendto(data, (self.group, self.port))

        print(f"📢 Invalidation pattern multicast: {pattern}")

    def _listen_invalidations(self):
        """Thread qui écoute les messages d'invalidation"""
        print(f"👂 Écoute des invalidations sur {self.group}:{self.port}")

        while True:
            try:
                data, addr = self.recv_sock.recvfrom(65535)
                message = json.loads(data.decode('utf-8'))

                # Ignorer nos propres messages (loopback)
                if message['sender'] == socket.gethostname():
                    continue

                action = message['action']

                if action == 'invalidate':
                    key = message['key']
                    with self.cache_lock:
                        if key in self.cache:
                            del self.cache[key]
                            print(f"🗑️  Cache invalidé: {key} (depuis {addr[0]})")

                elif action == 'invalidate_pattern':
                    pattern = message['pattern']
                    with self.cache_lock:
                        keys_to_delete = [k for k in self.cache.keys() if pattern in k]
                        for key in keys_to_delete:
                            del self.cache[key]
                        print(f"🗑️  Pattern invalidé: {pattern} ({len(keys_to_delete)} clés)")

            except Exception as e:
                print(f"Erreur écoute: {e}")

# Simulation d'utilisation
def demo():
    cache = DistributedCacheInvalidator()

    # Serveur 1 met en cache
    cache.set('user:123', {'name': 'Alice', 'email': 'alice@example.com'})
    cache.set('user:456', {'name': 'Bob', 'email': 'bob@example.com'})
    cache.set('product:789', {'name': 'Widget', 'price': 99.99})

    time.sleep(1)

    # Afficher le cache
    print(f"\nCache actuel: {list(cache.cache.keys())}")

    # Simuler une modification de l'utilisateur 123 par un autre serveur
    print("\n--- Quelqu'un modifie user:123 ---")
    cache.invalidate('user:123')

    time.sleep(1)
    print(f"Cache après invalidation: {list(cache.cache.keys())}")

    # Invalider tous les utilisateurs
    print("\n--- Invalider tous les users ---")
    cache.invalidate_pattern('user:')

    time.sleep(1)
    print(f"Cache final: {list(cache.cache.keys())}")

    time.sleep(5)

if __name__ == '__main__':
    demo()
```

### Cas 4 : Heartbeat de cluster (Health Check)

**Contexte :** Détecter automatiquement les nœuds actifs dans un cluster sans serveur central.

```go
package main

import (
    "encoding/json"
    "fmt"
    "net"
    "sync"
    "time"
)

type NodeInfo struct {
    Hostname  string    `json:"hostname"`
    IP        string    `json:"ip"`
    Timestamp time.Time `json:"timestamp"`
    Load      float64   `json:"load"`
    Status    string    `json:"status"`
}

type ClusterMonitor struct {
    nodes      map[string]*NodeInfo
    mutex      sync.RWMutex
    mcastGroup string
    port       int
    selfNode   *NodeInfo
}

func NewClusterMonitor(group string, port int) *ClusterMonitor {
    hostname, _ := os.Hostname()

    return &ClusterMonitor{
        nodes:      make(map[string]*NodeInfo),
        mcastGroup: group,
        port:       port,
        selfNode: &NodeInfo{
            Hostname:  hostname,
            IP:        getLocalIP(),
            Status:    "active",
        },
    }
}

func (cm *ClusterMonitor) Start() {
    // Thread émetteur de heartbeat
    go cm.sendHeartbeat()

    // Thread récepteur
    go cm.receiveHeartbeats()

    // Thread de cleanup (retirer les nœuds morts)
    go cm.cleanupDeadNodes()

    // Thread de monitoring
    go cm.displayClusterStatus()
}

func (cm *ClusterMonitor) sendHeartbeat() {
    addr, _ := net.ResolveUDPAddr("udp", fmt.Sprintf("%s:%d", cm.mcastGroup, cm.port))
    conn, _ := net.DialUDP("udp", nil, addr)
    defer conn.Close()

    ticker := time.NewTicker(2 * time.Second)
    defer ticker.Stop()

    for range ticker.C {
        cm.selfNode.Timestamp = time.Now()
        cm.selfNode.Load = getSystemLoad()  // Fonction fictive

        data, _ := json.Marshal(cm.selfNode)
        conn.Write(data)

        fmt.Printf("📡 Heartbeat envoyé\n")
    }
}

func (cm *ClusterMonitor) receiveHeartbeats() {
    addr, _ := net.ResolveUDPAddr("udp", fmt.Sprintf(":%d", cm.port))
    conn, _ := net.ListenMulticastUDP("udp", nil, addr)
    defer conn.Close()

    buffer := make([]byte, 10240)

    for {
        n, _, err := conn.ReadFromUDP(buffer)
        if err != nil {
            continue
        }

        var node NodeInfo
        if err := json.Unmarshal(buffer[:n], &node); err != nil {
            continue
        }

        // Ignorer notre propre heartbeat
        if node.Hostname == cm.selfNode.Hostname {
            continue
        }

        cm.mutex.Lock()
        cm.nodes[node.Hostname] = &node
        cm.mutex.Unlock()

        fmt.Printf("💓 Heartbeat reçu de %s (load: %.2f)\n",
                   node.Hostname, node.Load)
    }
}

func (cm *ClusterMonitor) cleanupDeadNodes() {
    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()

    for range ticker.C {
        threshold := time.Now().Add(-10 * time.Second)

        cm.mutex.Lock()
        for hostname, node := range cm.nodes {
            if node.Timestamp.Before(threshold) {
                delete(cm.nodes, hostname)
                fmt.Printf("💀 Nœud mort détecté: %s\n", hostname)
            }
        }
        cm.mutex.Unlock()
    }
}

func (cm *ClusterMonitor) displayClusterStatus() {
    ticker := time.NewTicker(10 * time.Second)
    defer ticker.Stop()

    for range ticker.C {
        cm.mutex.RLock()
        fmt.Printf("\n===== État du cluster =====\n")
        fmt.Printf("Nœuds actifs: %d\n", len(cm.nodes))
        for hostname, node := range cm.nodes {
            fmt.Printf("  • %s (%s) - Load: %.2f - Vu il y a %v\n",
                       hostname, node.IP, node.Load,
                       time.Since(node.Timestamp).Round(time.Second))
        }
        fmt.Printf("===========================\n\n")
        cm.mutex.RUnlock()
    }
}

func getLocalIP() string {
    // Implémentation simplifiée
    return "192.168.1.100"
}

func getSystemLoad() float64 {
    // Implémentation simplifiée
    return 0.5 + (0.5 * time.Now().Second() / 60.0)
}

func main() {
    monitor := NewClusterMonitor("239.255.2.1", 8888)
    monitor.Start()

    select {} // Bloquer indéfiniment
}
```

## Multicast routing et PIM

### Protocoles de routage multicast

Contrairement à l'unicast où les routeurs forwardent vers une destination, le multicast nécessite de construire des **arbres de distribution**.

#### PIM (Protocol Independent Multicast)

**PIM-DM (Dense Mode)** : Flood-and-prune
```
1. Inonder tout le réseau avec le trafic
2. Les branches sans récepteurs envoient "Prune" (élaguer)
3. L'arbre se rétrécit progressivement
```
Usage : Petits réseaux avec beaucoup de récepteurs.

**PIM-SM (Sparse Mode)** : Explicit join
```
1. Les récepteurs envoient "Join" explicite vers un RP (Rendezvous Point)
2. Construction d'un arbre partagé centré sur le RP
3. Optionnel : passage à un arbre optimisé par source (SPT)
```
Usage : Internet, grands réseaux (mode le plus courant).

**PIM-SSM (Source-Specific Multicast)** : Join vers source spécifique
```
1. Le récepteur connaît la source à l'avance
2. Join directement vers (S,G) - pas de RP nécessaire
3. Arbre optimisé automatiquement
```
Usage : IPTV moderne, streaming à grande échelle.

### Source-Specific Multicast (SSM)

SSM utilise la plage **232.0.0.0/8** et simplifie drastiquement le multicast.

```python
import socket
import struct

def join_ssm_group(source_ip, group_ip, port):
    """
    Rejoindre un groupe SSM (IGMPv3 requis)

    Différence clé : on spécifie la SOURCE en plus du groupe
    """
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind(('', port))

    # Structure pour SSM : ip_mreq_source
    # struct ip_mreq_source {
    #     struct in_addr imr_multiaddr;  /* IP multicast group address */
    #     struct in_addr imr_interface;  /* IP address of local interface */
    #     struct in_addr imr_sourceaddr; /* IP address of multicast source */
    # };

    mreq = struct.pack('4s4s4s',
                       socket.inet_aton(group_ip),
                       socket.inet_aton('0.0.0.0'),  # Interface locale
                       socket.inet_aton(source_ip))  # Source spécifique !

    # IP_ADD_SOURCE_MEMBERSHIP au lieu de IP_ADD_MEMBERSHIP
    sock.setsockopt(socket.IPPROTO_IP, socket.IP_ADD_SOURCE_MEMBERSHIP, mreq)

    print(f"✓ Joint SSM: source={source_ip}, groupe={group_ip}")

    return sock

# Exemple : recevoir uniquement depuis une source spécifique
sock = join_ssm_group(
    source_ip='10.1.1.100',  # Seul cette source est autorisée
    group_ip='232.1.1.1',
    port=5004
)

while True:
    data, addr = sock.recvfrom(10240)
    print(f"[{addr[0]}] {data[:50]}")
```

## Multicast dans les environnements modernes

### Docker et multicast

Par défaut, Docker **ne supporte PAS le multicast** sur les réseaux bridge.

**Solutions :**

**1. Mode host network**
```yaml
# docker-compose.yml
services:
  multicast-app:
    image: myapp:latest
    network_mode: "host"  # Accès direct à la stack réseau de l'hôte
```

**2. Mode macvlan**
```bash
# Créer un réseau macvlan
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  multicast_net

# Lancer un conteneur
docker run --network=multicast_net myapp
```

### Kubernetes et multicast

Kubernetes **ne supporte pas nativement le multicast** (CNI ne le gère généralement pas).

**Workarounds :**

**1. Host network (privilégié)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multicast-receiver
spec:
  hostNetwork: true  # Utilise la stack réseau du nœud
  containers:
  - name: receiver
    image: multicast-app:v1
```

**2. Multus CNI (multi-network)**
```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: multicast-network
spec:
  config: '{
    "cniVersion": "0.3.1",
    "type": "macvlan",
    "master": "eth0",
    "mode": "bridge",
    "ipam": {
      "type": "host-local",
      "subnet": "192.168.1.0/24"
    }
  }'
---
apiVersion: v1
kind: Pod
metadata:
  name: multicast-pod
  annotations:
    k8s.v1.cni.cncf.io/networks: multicast-network
spec:
  containers:
  - name: app
    image: multicast-app:v1
```

### Cloud providers et multicast

| Provider | Support multicast | Notes |
|----------|-------------------|-------|
| **AWS** | ❌ Non (VPC) | Uniquement dans Placement Groups ou Transit Gateway Multicast |
| **Azure** | ⚠️ Limité | Uniquement dans certains SKUs |
| **GCP** | ❌ Non | Pas de support multicast dans VPC |
| **On-premise** | ✅ Oui | Dépend de la config réseau |

**Alternative cloud :** Utiliser des solutions applicatives (pub/sub, message queues) plutôt que le multicast réseau.

## Debugging et outils

### Vérifier le support multicast

```bash
# Vérifier si l'interface supporte multicast
ip link show | grep MULTICAST

# Exemple de sortie :
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
#            └─── Le flag MULTICAST est présent

# Activer le multicast sur une interface (si désactivé)
sudo ip link set eth0 multicast on
```

### Tester le multicast avec des outils standards

**1. Avec netcat (ncat)**
```bash
# Sender
ncat --udp --source-port 5007 239.1.1.1 5007

# Receiver
ncat --udp -l 5007 --group 239.1.1.1
```

**2. Avec socat**
```bash
# Sender
echo "Test multicast" | socat - UDP-DATAGRAM:239.1.1.1:5007

# Receiver
socat UDP-RECV:5007,ip-add-membership=239.1.1.1:0.0.0.0 -
```

**3. Avec iperf3**
```bash
# Serveur (receiver)
iperf3 -s -B 239.1.1.1

# Client (sender)
iperf3 -c 239.1.1.1 -u -b 10M -t 30
```

### Capturer le trafic multicast avec tcpdump

```bash
# Capturer tout le trafic multicast
sudo tcpdump -i eth0 'ip multicast'

# Capturer un groupe spécifique
sudo tcpdump -i eth0 'dst 239.1.1.1'

# Voir les messages IGMP
sudo tcpdump -i eth0 'igmp'

# Exemple de sortie :
# 14:23:45.123456 IP 192.168.1.100 > 224.0.0.22: igmp v2 report 239.1.1.1
# 14:23:50.234567 IP 192.168.1.1 > 224.0.0.1: igmp query
```

### Wireshark : Analyser IGMP

Filtres Wireshark utiles :
```
igmp                          # Tous les messages IGMP
igmp.type == 0x16             # Membership Reports v2
igmp.type == 0x11             # Queries
igmp.type == 0x17             # Leave messages
ip.dst == 239.1.1.1           # Trafic vers un groupe
```

### Vérifier les groupes multicast actifs

```bash
# Linux : voir les groupes rejoints
netstat -g
# ou
ip maddress show

# Exemple de sortie :
# 2:      eth0
#         link  01:00:5e:00:00:01
#         link  33:33:00:00:00:01
#         inet  239.1.1.1
#         inet6 ff02::1

# macOS
netstat -g

# Windows
netsh interface ipv4 show joins
```

## Pièges courants et solutions

### Piège 1 : Firewall bloque IGMP

**Symptôme :** Le socket multicast ne reçoit rien.

**Cause :** Le firewall bloque le protocole IGMP (IP protocol 2).

**Solution :**
```bash
# iptables : autoriser IGMP
sudo iptables -A INPUT -p igmp -j ACCEPT
sudo iptables -A OUTPUT -p igmp -j ACCEPT

# Autoriser le trafic multicast
sudo iptables -A INPUT -d 224.0.0.0/4 -j ACCEPT

# firewalld
sudo firewall-cmd --add-protocol=igmp --permanent
sudo firewall-cmd --reload
```

### Piège 2 : Bind sur la mauvaise adresse

**Problème :**
```python
# ❌ FAUX : binder sur l'adresse multicast
sock.bind((MCAST_GROUP, PORT))  # Ne marche pas !
```

**Correction :**
```python
# ✅ CORRECT : binder sur toutes les interfaces
sock.bind(('', PORT))  # ou ('0.0.0.0', PORT)
```

### Piège 3 : TTL trop faible

**Symptôme :** Le multicast ne traverse pas les routeurs.

**Cause :** TTL = 1 par défaut (lien local seulement).

**Solution :**
```python
# Augmenter le TTL multicast
sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_TTL, 32)
```

Valeurs recommandées :
- TTL = 1 : Lien local (même subnet)
- TTL = 32 : Site (campus, entreprise)
- TTL = 64 : Région
- TTL = 255 : Global (Internet)

### Piège 4 : Loopback activé

**Symptôme :** Vous recevez vos propres messages multicast.

**Cause :** IP_MULTICAST_LOOP est activé par défaut.

**Solution :**
```python
# Désactiver le loopback multicast
sock.setsockopt(socket.IPPROTO_IP, socket.IP_MULTICAST_LOOP, 0)
```

### Piège 5 : Switch non configuré pour multicast

**Symptôme :** Performance dégradée, paquets perdus.

**Cause :** Le switch traite le multicast comme du broadcast (flooding).

**Solution :** Activer **IGMP Snooping** sur le switch.

```
# Cisco switch
interface vlan 1
  ip igmp snooping
  ip igmp snooping querier
```

IGMP Snooping permet au switch de n'envoyer le multicast que sur les ports où des récepteurs sont présents.

### Piège 6 : MTU et fragmentation

**Problème :** Paquets multicast > MTU sont fragmentés et peuvent être perdus.

**Solution :** Respecter la MTU path.

```python
# Découvrir la MTU
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Interdire la fragmentation (Linux)
sock.setsockopt(socket.IPPROTO_IP, socket.IP_MTU_DISCOVER, socket.IP_PMTUDISC_DO)

# Envoyer en respectant MTU = 1500 - 20 (IP) - 8 (UDP) = 1472
MAX_PAYLOAD = 1472
```

## Alternatives au multicast

Si le multicast n'est pas disponible (cloud, firewall restrictif), considérez :

### 1. Publish/Subscribe (Pub/Sub)

```python
# Redis Pub/Sub
import redis

r = redis.Redis()

# Publisher
r.publish('notifications', 'User logged in')

# Subscriber
pubsub = r.pubsub()
pubsub.subscribe('notifications')

for message in pubsub.listen():
    print(message['data'])
```

### 2. Message Queues (fanout)

```python
# RabbitMQ fanout exchange
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Créer un exchange fanout (similaire au multicast)
channel.exchange_declare(exchange='logs', exchange_type='fanout')

# Publisher
channel.basic_publish(exchange='logs', routing_key='', body='Log message')

# Subscriber
result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

channel.queue_bind(exchange='logs', queue=queue_name)
channel.basic_consume(queue=queue_name, on_message_callback=callback, auto_ack=True)
channel.start_consuming()
```

### 3. WebSockets broadcast

```javascript
// Server-side (Node.js)
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

const clients = new Set();

wss.on('connection', (ws) => {
    clients.add(ws);

    ws.on('close', () => {
        clients.delete(ws);
    });
});

// Broadcast à tous les clients
function broadcast(message) {
    clients.forEach(client => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(message);
        }
    });
}
```

## Résumé : Quand utiliser le multicast

### ✅ Le multicast est idéal pour :

- **Streaming vidéo/audio** vers de nombreux récepteurs (IPTV, radio IP)
- **Service discovery** sur LAN (mDNS, SSDP)
- **Synchronisation de cache** distribué
- **Heartbeat de cluster** (monitoring interne)
- **Distribution de données financières** (market data)
- **Gaming multijoueur** (state updates)

### ❌ Éviter le multicast si :

- Vous êtes dans le **cloud public** (AWS, GCP, Azure) → pas de support
- Vous avez besoin de **garanties de livraison** → utiliser TCP ou message queue
- Le réseau ne supporte pas IGMP/PIM
- Les récepteurs sont **géographiquement dispersés** → CDN plus efficace

### Checklist de décision

| Critère | Multicast | Pub/Sub (Redis/RabbitMQ) |
|---------|-----------|--------------------------|
| **Nombre de récepteurs** | 100+ | < 100 |
| **Localisation** | Même LAN/datacenter | Distribuée |
| **Latence requise** | < 10ms | < 100ms acceptable |
| **Garantie de livraison** | Best-effort | Garantie possible |
| **Infrastructure** | Contrôle réseau | Managé/cloud |

## Perspectives futures

### Tendances

**1. AMT (Automatic Multicast Tunneling)**
Tunneling multicast à travers des réseaux unicast (pour traverser Internet).

**2. BIER (Bit Index Explicit Replication)**
Nouvelle approche pour le multicast sans état dans le cœur du réseau.

**3. QUIC multicast**
Extension de QUIC pour supporter le multicast de manière fiable.

**4. Application-layer multicast**
Overlay networks (CDN, P2P) simulant le multicast au niveau applicatif.

## Conclusion

Le multicast est une **technique puissante** pour distribuer efficacement des données vers de nombreux destinataires. Bien que son support soit limité dans les environnements cloud modernes, il reste **essentiel** pour :

- Les applications temps réel sur LAN
- Le service discovery
- Les architectures de diffusion à grande échelle

**À retenir :**
- Le multicast économise drastiquement la bande passante (1 flux → N récepteurs)
- IGMP gère l'adhésion aux groupes
- SSM (232.0.0.0/8) est le mode moderne recommandé
- Tester soigneusement le support réseau avant de déployer
- Avoir des alternatives (pub/sub) pour les environnements cloud

**Prochaine étape :** Dans la section suivante, nous explorerons le **Load Balancing**, essentiel pour distribuer la charge et assurer la haute disponibilité.

---


⏭️ [Load balancing : principes et niveaux (L4, L7)](/09-architectures-avancees/03-load-balancing.md)
