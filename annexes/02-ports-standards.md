🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Tableau récapitulatif des ports standards

## Introduction

Les ports TCP et UDP sont des identifiants numériques de 16 bits (0-65535) permettant de distinguer les différents services et applications sur un même hôte. Ce tableau récapitulatif présente les ports standards les plus couramment utilisés, organisés par fonction.

### Classification des ports

Les ports sont divisés en trois catégories définies par l'IANA (Internet Assigned Numbers Authority) :

| Catégorie | Plage | Description | Usage |
|-----------|-------|-------------|-------|
| **Well-Known Ports** | 0-1023 | Ports privilégiés | Services système standards |
| **Registered Ports** | 1024-49151 | Ports enregistrés | Applications spécifiques |
| **Dynamic/Private Ports** | 49152-65535 | Ports éphémères | Ports clients dynamiques |

**Note importante :** Un même numéro de port peut être utilisé simultanément par TCP et UDP pour des services différents.

---

## Ports standards par service

### Services Web

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **20** | TCP | FTP-DATA | Transfert de données FTP (mode actif) |
| **21** | TCP | FTP | Contrôle FTP - File Transfer Protocol |
| **22** | TCP | SSH | Secure Shell - accès distant sécurisé |
| **23** | TCP | Telnet | Accès distant non sécurisé (obsolète) |
| **80** | TCP | HTTP | HyperText Transfer Protocol - Web non sécurisé |
| **443** | TCP | HTTPS | HTTP Secure - Web sécurisé via TLS/SSL |
| **8080** | TCP | HTTP-ALT | Port alternatif HTTP (proxy, développement) |
| **8443** | TCP | HTTPS-ALT | Port alternatif HTTPS |
| **3000** | TCP | HTTP-DEV | Serveurs de développement (Node.js, React, etc.) |
| **5000** | TCP | HTTP-DEV | Serveurs de développement (Flask, etc.) |

**Usage typique :**
- Port 80/443 : Production web standard
- Port 8080/8443 : Serveurs d'application, proxys inverses
- Ports 3000-9000 : Environnements de développement

---

### Services de messagerie

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **25** | TCP | SMTP | Simple Mail Transfer Protocol - envoi d'emails |
| **110** | TCP | POP3 | Post Office Protocol v3 - réception emails |
| **143** | TCP | IMAP | Internet Message Access Protocol - gestion emails |
| **465** | TCP | SMTPS | SMTP sécurisé via SSL (historique) |
| **587** | TCP | SMTP-SUBMISSION | Soumission SMTP sécurisée (STARTTLS) |
| **993** | TCP | IMAPS | IMAP sécurisé via TLS/SSL |
| **995** | TCP | POP3S | POP3 sécurisé via TLS/SSL |

**Notes de sécurité :**
- Port 25 : Communication entre serveurs mail (MTA), souvent bloqué par les FAI
- Port 587 : Préféré pour l'envoi depuis clients email (authentification requise)
- Ports 993/995 : Toujours privilégier les versions sécurisées

---

### DNS et résolution de noms

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **53** | TCP/UDP | DNS | Domain Name System - résolution de noms |
| **853** | TCP | DoT | DNS over TLS - DNS sécurisé |
| **5353** | UDP | mDNS | Multicast DNS - résolution locale (Bonjour/Avahi) |

**Particularités DNS :**
- UDP port 53 : Requêtes standard (< 512 octets)
- TCP port 53 : Transferts de zone, requêtes > 512 octets, DNSSEC
- Port 853 : Chiffrement des requêtes DNS

---

### Transfert et partage de fichiers

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **20** | TCP | FTP-DATA | Données FTP mode actif |
| **21** | TCP | FTP | Contrôle FTP |
| **22** | TCP | SFTP/SCP | Transfert de fichiers via SSH |
| **69** | UDP | TFTP | Trivial FTP - transfert simple sans auth |
| **115** | TCP | SFTP-OLD | Simple FTP (obsolète, différent du SFTP/SSH) |
| **445** | TCP | SMB | Server Message Block - partage Windows |
| **989** | TCP | FTPS-DATA | FTP sécurisé - données |
| **990** | TCP | FTPS | FTP sécurisé - contrôle |
| **2049** | TCP/UDP | NFS | Network File System - partage Unix/Linux |

**Recommandations :**
- Préférer SFTP (port 22) ou FTPS (990) au FTP classique
- SMB : vulnérable, limiter au réseau local uniquement

---

### Bases de données

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **1433** | TCP | MSSQL | Microsoft SQL Server |
| **1521** | TCP | Oracle | Oracle Database |
| **3306** | TCP | MySQL/MariaDB | MySQL et MariaDB |
| **5432** | TCP | PostgreSQL | PostgreSQL Database |
| **5984** | TCP | CouchDB | Apache CouchDB (API HTTP) |
| **6379** | TCP | Redis | Redis - base de données en mémoire |
| **7000-7001** | TCP | Cassandra | Apache Cassandra |
| **9042** | TCP | Cassandra-CQL | Cassandra CQL native |
| **27017** | TCP | MongoDB | MongoDB Database |
| **28015** | TCP | RethinkDB | RethinkDB - client driver |

**Sécurité critique :**
- Ne **jamais** exposer ces ports sur Internet
- Utiliser un VPN ou SSH tunneling pour l'accès distant
- Configurer l'authentification forte et le chiffrement

---

### Administration système et monitoring

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **22** | TCP | SSH | Secure Shell - administration distante |
| **23** | TCP | Telnet | Terminal distant non sécurisé (éviter) |
| **161** | UDP | SNMP | Simple Network Management Protocol |
| **162** | UDP | SNMP-TRAP | SNMP traps - notifications |
| **389** | TCP | LDAP | Lightweight Directory Access Protocol |
| **636** | TCP | LDAPS | LDAP sécurisé |
| **3389** | TCP | RDP | Remote Desktop Protocol - Bureau à distance Windows |
| **5900+** | TCP | VNC | Virtual Network Computing - bureau distant |
| **9090** | TCP | Prometheus | Prometheus monitoring |
| **9100** | TCP | Node-Exporter | Prometheus Node Exporter |

**Bonnes pratiques :**
- SSH : Changer le port par défaut, désactiver root login
- RDP : Utiliser un VPN, activer NLA (Network Level Authentication)
- SNMP : Utiliser SNMPv3 avec authentification

---

### Services VPN et tunneling

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **500** | UDP | ISAKMP/IKE | IPsec - négociation de clés |
| **1194** | UDP | OpenVPN | OpenVPN (port par défaut) |
| **1701** | UDP | L2TP | Layer 2 Tunneling Protocol |
| **1723** | TCP | PPTP | Point-to-Point Tunneling Protocol |
| **4500** | UDP | IPsec-NAT-T | IPsec NAT Traversal |
| **51820** | UDP | WireGuard | WireGuard VPN (configurable) |

---

### Streaming et médias

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **554** | TCP/UDP | RTSP | Real Time Streaming Protocol |
| **1935** | TCP | RTMP | Real Time Messaging Protocol - Flash streaming |
| **5004-5005** | UDP | RTP/RTCP | Real-time Transport Protocol |
| **6881-6889** | TCP/UDP | BitTorrent | Partage P2P BitTorrent |
| **8554** | TCP | RTSP-ALT | RTSP alternatif |

---

### Messaging et temps réel

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **5222** | TCP | XMPP-Client | Jabber/XMPP client |
| **5269** | TCP | XMPP-Server | Jabber/XMPP serveur à serveur |
| **5280** | TCP | XMPP-HTTP | XMPP over HTTP (BOSH) |
| **6379** | TCP | Redis-PubSub | Redis Pub/Sub messaging |
| **9092** | TCP | Kafka | Apache Kafka broker |
| **15672** | TCP | RabbitMQ-HTTP | RabbitMQ Management UI |
| **5672** | TCP | AMQP | RabbitMQ/AMQP protocol |

---

### Services cloud et conteneurs

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **2375** | TCP | Docker | Docker REST API (non sécurisé) |
| **2376** | TCP | Docker-TLS | Docker REST API sécurisé |
| **2377** | TCP | Docker-Swarm | Docker Swarm cluster |
| **6443** | TCP | Kubernetes-API | Kubernetes API Server |
| **8001** | TCP | Kubernetes-Proxy | kubectl proxy |
| **10250** | TCP | Kubelet | Kubernetes Kubelet API |
| **10251** | TCP | kube-scheduler | Kubernetes scheduler |
| **10252** | TCP | kube-controller | Kubernetes controller manager |

---

### Développement et outils

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **3000** | TCP | Node.js-Dev | React, Next.js, Node.js dev servers |
| **3306** | TCP | MySQL | MySQL development |
| **4200** | TCP | Angular-Dev | Angular development server |
| **5000** | TCP | Flask-Dev | Flask development server |
| **5173** | TCP | Vite-Dev | Vite development server |
| **8000** | TCP | Django-Dev | Django development server |
| **8080** | TCP | Tomcat/Dev | Tomcat, serveurs de développement |
| **8888** | TCP | Jupyter | Jupyter Notebook |
| **9200** | TCP | Elasticsearch | Elasticsearch REST API |
| **9300** | TCP | Elasticsearch-Node | Elasticsearch communication inter-nœuds |

---

### Services réseau essentiels

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **67** | UDP | DHCP-Server | Dynamic Host Configuration - serveur |
| **68** | UDP | DHCP-Client | Dynamic Host Configuration - client |
| **123** | UDP | NTP | Network Time Protocol - synchronisation |
| **514** | UDP | Syslog | Système de journalisation |
| **520** | UDP | RIP | Routing Information Protocol |
| **179** | TCP | BGP | Border Gateway Protocol |

---

## Tableau de référence rapide - Top 30

Ports les plus fréquemment rencontrés dans l'ordre numérique :

| Port | TCP/UDP | Service | Utilisation courante |
|------|---------|---------|----------------------|
| 20-21 | TCP | FTP | Transfert fichiers (obsolète) |
| 22 | TCP | SSH/SFTP | Administration sécurisée |
| 23 | TCP | Telnet | Admin non sécurisée (éviter) |
| 25 | TCP | SMTP | Envoi emails serveur-serveur |
| 53 | TCP/UDP | DNS | Résolution de noms |
| 67-68 | UDP | DHCP | Attribution IP automatique |
| 80 | TCP | HTTP | Web non sécurisé |
| 110 | TCP | POP3 | Réception emails |
| 123 | UDP | NTP | Synchronisation temps |
| 143 | TCP | IMAP | Gestion emails |
| 161-162 | UDP | SNMP | Monitoring réseau |
| 389 | TCP | LDAP | Annuaire |
| 443 | TCP | HTTPS | Web sécurisé |
| 445 | TCP | SMB | Partage fichiers Windows |
| 465 | TCP | SMTPS | SMTP sécurisé (SSL) |
| 587 | TCP | SMTP | Soumission emails clients |
| 636 | TCP | LDAPS | LDAP sécurisé |
| 993 | TCP | IMAPS | IMAP sécurisé |
| 995 | TCP | POP3S | POP3 sécurisé |
| 1433 | TCP | MSSQL | Microsoft SQL Server |
| 3306 | TCP | MySQL | Base de données MySQL |
| 3389 | TCP | RDP | Bureau distant Windows |
| 5432 | TCP | PostgreSQL | Base de données PostgreSQL |
| 5900 | TCP | VNC | Bureau distant VNC |
| 6379 | TCP | Redis | Cache/DB Redis |
| 8080 | TCP | HTTP-Alt | Proxy, développement |
| 8443 | TCP | HTTPS-Alt | HTTPS alternatif |
| 27017 | TCP | MongoDB | Base NoSQL MongoDB |

---

## Ports et protocoles de sécurité

### Ports à surveiller pour la sécurité

**⚠️ Ports à risque élevé (à ne jamais exposer publiquement) :**

| Port | Service | Risque |
|------|---------|--------|
| 23 | Telnet | Communication en clair, credentials exposés |
| 69 | TFTP | Pas d'authentification, transfert non sécurisé |
| 445 | SMB | Cible fréquente de ransomware (WannaCry) |
| 1433 | MSSQL | Attaques par force brute |
| 3306 | MySQL | Exposition de bases de données |
| 3389 | RDP | Brute force, vulnérabilités fréquentes |
| 5432 | PostgreSQL | Exposition de bases de données |
| 6379 | Redis | Souvent sans authentification par défaut |
| 27017 | MongoDB | Historique d'expositions non sécurisées |

**✅ Ports sécurisés à privilégier :**

| Port | Service | Avantage |
|------|---------|----------|
| 22 | SSH | Chiffrement fort, authentification par clés |
| 443 | HTTPS | Chiffrement TLS obligatoire |
| 587 | SMTP+STARTTLS | Authentification et chiffrement |
| 853 | DNS over TLS | Requêtes DNS chiffrées |
| 993/995 | IMAPS/POP3S | Messagerie chiffrée |

---

## Ports de jeux en ligne

Quelques ports couramment utilisés par les jeux multijoueurs :

| Port(s) | Protocole | Jeu/Service |
|---------|-----------|-------------|
| 25565 | TCP | Minecraft Java Edition |
| 27015-27030 | TCP/UDP | Steam, Source games |
| 3074 | TCP/UDP | Xbox Live |
| 3478-3479 | UDP | PlayStation Network |
| 5222 | TCP | Epic Games Store |
| 6112 | TCP/UDP | Blizzard games (legacy) |
| 27000-27050 | TCP/UDP | Steam games |

---

## Plages de ports spécifiques

### Services Microsoft

| Port(s) | Service |
|---------|---------|
| 135 | RPC Endpoint Mapper |
| 137-139 | NetBIOS |
| 445 | SMB/CIFS |
| 1433 | SQL Server |
| 3389 | Remote Desktop |
| 5985-5986 | WinRM (HTTP/HTTPS) |

### Services Apple

| Port | Service |
|------|---------|
| 548 | AFP (Apple Filing Protocol) |
| 5223 | Apple Push Notification Service |
| 5228 | Google Cloud Messaging (Android/iOS) |
| 5353 | Bonjour/mDNS |

---

## Bonnes pratiques de gestion des ports

### Recommandations générales

1. **Principe du moindre privilège**
   - N'ouvrir que les ports strictement nécessaires
   - Filtrer par adresse IP source quand possible

2. **Sécurisation**
   - Toujours privilégier les versions sécurisées (HTTPS vs HTTP, SFTP vs FTP)
   - Changer les ports par défaut pour les services critiques (SSH, RDP)
   - Utiliser des pare-feu et règles de filtrage strictes

3. **Monitoring**
   - Surveiller les ports ouverts : `netstat`, `ss`, `nmap`
   - Détecter les tentatives de connexion suspectes
   - Auditer régulièrement la configuration

4. **Documentation**
   - Maintenir un inventaire des ports utilisés
   - Documenter les raisons d'ouverture de chaque port
   - Réviser périodiquement les besoins

### Commandes utiles pour vérifier les ports

```bash
# Lister les ports en écoute (Linux)
ss -tuln
netstat -tuln

# Scanner les ports ouverts sur une machine
nmap -p- localhost

# Tester la connectivité vers un port spécifique
telnet hostname port
nc -zv hostname port

# Vérifier quel processus écoute sur un port
lsof -i :port
ss -tlnp | grep :port
```

---

## Ports dynamiques et éphémères

Les ports **49152-65535** sont réservés pour :
- Ports sources des connexions clients
- Assignation dynamique par le système d'exploitation
- Sessions temporaires

**Plages par système :**
- Linux : 32768-60999 (configurable via `/proc/sys/net/ipv4/ip_local_port_range`)
- Windows : 49152-65535 (depuis Vista/Server 2008)
- BSD : 49152-65535

**Importance :** Ces ports sont automatiquement sélectionnés pour les connexions sortantes et libérés après fermeture de la connexion.

---

## Références

**Sources officielles :**
- [IANA Service Name and Transport Protocol Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/)
- RFC 1700 - Assigned Numbers (obsolète mais historique)
- RFC 6335 - Internet Assigned Numbers Authority (IANA) Procedures

**Outils de référence :**
- `/etc/services` (Linux/Unix) - Liste locale des services et ports
- `getent services` - Interroger la base de données des services

---

*Ce tableau est une référence générale. Les ports non standards et propriétaires peuvent varier selon les installations. Consultez toujours la documentation spécifique de vos applications.*



⏭️ [C. Références RFC essentielles](/annexes/03-rfc-essentielles.md)
