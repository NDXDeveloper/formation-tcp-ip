🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.3 Ports bien connus, enregistrés et dynamiques

## Introduction

Les 65 536 ports disponibles (0-65535) ne sont pas tous égaux. Ils sont organisés en **trois catégories distinctes**, chacune ayant un rôle et des règles d'utilisation spécifiques. Cette organisation, définie par l'**IANA** (Internet Assigned Numbers Authority), garantit que les services réseau peuvent communiquer de manière standardisée à travers le monde.

Comprendre cette classification est essentiel pour :
- Développer des applications réseau
- Configurer des pare-feu
- Diagnostiquer des problèmes de connexion
- Respecter les standards Internet

## Vue d'ensemble des trois catégories

```
Plage de ports : 0 ─────────────────────────────────────────────> 65535

┌──────────────┬─────────────────────────────┬──────────────────┐
│   0-1023     │      1024-49151             │   49152-65535    │
│              │                             │                  │
│    Ports     │        Ports                │      Ports       │
│ bien connus  │     enregistrés             │   dynamiques     │
│  (Well-Known)│    (Registered)             │   (Dynamic/      │
│              │                             │   Private)       │
└──────────────┴─────────────────────────────┴──────────────────┘

Gestion :     IANA          IANA              Libres
Privilèges :  Root/Admin    Aucun             Aucun
Usage :       Services      Applications      Clients
              système       tierces           (éphémères)
```

## Catégorie 1 : Ports bien connus (0-1023)

### Définition et caractéristiques

Les **ports bien connus** (Well-Known Ports) constituent la plage **0 à 1023**. Ce sont les ports les plus importants et les plus strictement contrôlés.

**Caractéristiques principales** :
- **Standardisés** par l'IANA et définis dans des RFC
- **Réservés** pour des services système fondamentaux
- **Nécessitent des privilèges élevés** (root sous Linux, administrateur sous Windows) pour être utilisés
- **Connus universellement** : un serveur web écoute sur le port 80 partout dans le monde

### Pourquoi nécessitent-ils des privilèges ?

**Raison historique et sécuritaire** : Sur les systèmes Unix/Linux, seul l'utilisateur root peut créer un socket lié à un port inférieur à 1024. Cela empêche un utilisateur malveillant de lancer un faux serveur SSH sur le port 22, par exemple.

```bash
# En tant qu'utilisateur normal
$ python3 -m http.server 80
Traceback (most recent call last):
PermissionError: [Errno 13] Permission denied

# En tant que root
$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 ...  ✓
```

### Les ports bien connus essentiels

Voici les ports les plus couramment utilisés, classés par catégorie.

#### Protocoles web

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **20** | TCP | FTP-DATA | Transfert de données FTP |
| **21** | TCP | FTP | Contrôle FTP (File Transfer Protocol) |
| **80** | TCP | HTTP | Navigation web non sécurisée |
| **443** | TCP | HTTPS | Navigation web sécurisée (TLS/SSL) |

**Exemple concret** : Navigation sur un site web

```
Vous tapez : http://example.com
             ↓
Navigateur se connecte à : example.com:80

Vous tapez : https://example.com
             ↓
Navigateur se connecte à : example.com:443
```

Si le port n'est pas spécifié, le navigateur utilise automatiquement :
- Port 80 pour HTTP
- Port 443 pour HTTPS

#### Services email

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **25** | TCP | SMTP | Envoi d'emails (Simple Mail Transfer Protocol) |
| **110** | TCP | POP3 | Réception d'emails (Post Office Protocol v3) |
| **143** | TCP | IMAP | Accès aux emails (Internet Message Access Protocol) |
| **465** | TCP | SMTPS | SMTP sécurisé (via SSL) |
| **587** | TCP | SMTP | Soumission d'emails (avec STARTTLS) |
| **993** | TCP | IMAPS | IMAP sécurisé (via SSL/TLS) |
| **995** | TCP | POP3S | POP3 sécurisé (via SSL/TLS) |

**Flux email typique** :

```
Votre client email (Outlook, Thunderbird, etc.)
         │
         │ Envoi d'email
         ├──────> Serveur SMTP:587 (avec authentification)
         │
         │ Réception d'emails
         └──────> Serveur IMAP:993 (lecture)
```

**Exemple** : Configuration Gmail dans un client email

```
Serveur SMTP sortant :
  smtp.gmail.com:587 (ou 465)

Serveur IMAP entrant :
  imap.gmail.com:993
```

#### Administration et accès distant

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **22** | TCP | SSH | Shell sécurisé (Secure Shell) |
| **23** | TCP | Telnet | Shell non sécurisé (obsolète) |
| **3389** | TCP | RDP | Bureau à distance Windows |

**Exemple SSH** : Connexion à un serveur distant

```bash
$ ssh user@192.168.1.50
# Se connecte automatiquement au port 22

$ ssh -p 2222 user@192.168.1.50
# Se connecte au port 2222 (port alternatif)
```

**Note de sécurité** : Le port 22 étant bien connu, les serveurs en production changent souvent leur port SSH (ex: 2222) pour réduire les tentatives d'intrusion automatisées.

#### Résolution de noms et configuration

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **53** | TCP/UDP | DNS | Résolution de noms de domaine |
| **67** | UDP | DHCP Server | Attribution d'adresses IP (serveur) |
| **68** | UDP | DHCP Client | Attribution d'adresses IP (client) |

**Exemple DNS** :

```
Vous tapez : www.google.com dans votre navigateur
             ↓
Navigateur envoie une requête DNS à 8.8.8.8:53
             ↓
Réponse : 142.250.185.46
             ↓
Navigateur se connecte à 142.250.185.46:443
```

#### Bases de données

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **3306** | TCP | MySQL | Base de données MySQL/MariaDB |
| **5432** | TCP | PostgreSQL | Base de données PostgreSQL |
| **27017** | TCP | MongoDB | Base de données MongoDB |
| **6379** | TCP | Redis | Cache/base de données Redis |

**Exemple** : Connexion à une base MySQL

```bash
$ mysql -h db.example.com -P 3306 -u admin -p
# Se connecte à db.example.com sur le port 3306
```

**Configuration typique d'une application web** :

```
Application Web (Python/Django)
        │
        ├─> Base de données : postgres.local:5432
        ├─> Cache Redis : redis.local:6379
        └─> API externe : api.example.com:443
```

#### Autres services importants

| Port | Protocole | Service | Description |
|------|-----------|---------|-------------|
| **161** | UDP | SNMP | Monitoring réseau (agent) |
| **162** | UDP | SNMP Trap | Monitoring réseau (notifications) |
| **389** | TCP | LDAP | Annuaire (Lightweight Directory Access Protocol) |
| **636** | TCP | LDAPS | LDAP sécurisé |
| **514** | UDP | Syslog | Journalisation système |

### Tableau récapitulatif des ports les plus courants

```
┌───────────────────────────────────────────────────────────┐
│                    PORTS BIEN CONNUS                      │
├──────┬─────────┬──────────────────────────────────────────┤
│  22  │  SSH    │  Administration distante sécurisée       │
│  25  │  SMTP   │  Envoi d'emails                          │
│  53  │  DNS    │  Résolution de noms                      │
│  80  │  HTTP   │  Web non sécurisé                        │
│ 110  │  POP3   │  Réception emails                        │
│ 143  │  IMAP   │  Accès emails                            │
│ 443  │  HTTPS  │  Web sécurisé                            │
│ 587  │  SMTP   │  Soumission emails                       │
│ 993  │  IMAPS  │  IMAP sécurisé                           │
└──────┴─────────┴──────────────────────────────────────────┘
```

## Catégorie 2 : Ports enregistrés (1024-49151)

### Définition et caractéristiques

Les **ports enregistrés** (Registered Ports) occupent la plage **1024 à 49151**.

**Caractéristiques principales** :
- **Enregistrés** auprès de l'IANA par des organisations, des entreprises ou des projets
- **Pas de privilèges requis** : n'importe quelle application peut les utiliser
- **Semi-standardisés** : moins universels que les ports bien connus
- **Utilisés** par des applications tierces, des services d'entreprise, des logiciels populaires

### Processus d'enregistrement

Une organisation peut demander à l'IANA d'enregistrer un port pour son application. Par exemple :
- Microsoft a enregistré le port 3389 pour RDP
- MySQL utilise le port 3306
- PostgreSQL utilise le port 5432

**Important** : L'enregistrement d'un port ne garantit pas son exclusivité. N'importe qui peut techniquement utiliser n'importe quel port de cette plage, mais il est recommandé de respecter les enregistrements pour éviter les conflits.

### Exemples de ports enregistrés populaires

#### Services de bases de données (suite)

| Port | Service | Organisation |
|------|---------|--------------|
| **1433** | Microsoft SQL Server | Microsoft |
| **1521** | Oracle Database | Oracle |
| **3306** | MySQL | Oracle (anciennement MySQL AB) |
| **5432** | PostgreSQL | PostgreSQL Global Development Group |
| **27017** | MongoDB | MongoDB Inc. |
| **9042** | Cassandra | Apache |

#### Serveurs d'applications

| Port | Service | Organisation |
|------|---------|--------------|
| **8080** | HTTP alternatif | Divers (serveurs de développement) |
| **8443** | HTTPS alternatif | Divers |
| **8000** | HTTP dev | Couramment utilisé en développement |
| **3000** | Node.js/React | Convention pour développement frontend |
| **5000** | Flask | Convention pour applications Python Flask |
| **4200** | Angular | Convention pour Angular CLI |

**Exemple** : Développement d'une application web moderne

```
Frontend (React) : localhost:3000
Backend API (Node.js) : localhost:8080
Base de données (PostgreSQL) : localhost:5432
Redis cache : localhost:6379
```

#### Messagerie et communication

| Port | Service | Description |
|------|---------|-------------|
| **5222** | XMPP Client | Messagerie instantanée (Jabber) |
| **5269** | XMPP Server | Communication serveur-à-serveur XMPP |
| **5060** | SIP | Signalisation VoIP (Session Initiation Protocol) |
| **5061** | SIP-TLS | SIP sécurisé |
| **1883** | MQTT | Messagerie IoT (Message Queuing Telemetry Transport) |
| **8883** | MQTT-TLS | MQTT sécurisé |

#### Services de partage et streaming

| Port | Service | Description |
|------|---------|-------------|
| **1935** | RTMP | Streaming vidéo (Real-Time Messaging Protocol) |
| **6881-6889** | BitTorrent | Partage de fichiers P2P |
| **8333** | Bitcoin | Réseau Bitcoin |
| **9090** | Prometheus | Monitoring et métriques |

#### Outils de développement

| Port | Service | Usage typique |
|------|---------|---------------|
| **2375** | Docker | API Docker (non sécurisée) |
| **2376** | Docker | API Docker (TLS) |
| **9200** | Elasticsearch | API REST Elasticsearch |
| **5601** | Kibana | Interface Kibana |
| **15672** | RabbitMQ | Interface d'administration |

### Choix d'un port enregistré pour votre application

Si vous développez une application qui nécessite un port standard, vous avez deux options :

#### Option 1 : Utiliser un port non enregistré (recommandé pour usage interne)

Choisissez un port dans la plage 1024-49151 qui n'est pas largement utilisé.

```
Bonne pratique : Utilisez des ports comme 8080, 8000, 3000
pour le développement local.
```

#### Option 2 : Enregistrer officiellement un port (pour logiciel public)

Si votre application sera largement distribuée, vous pouvez demander un enregistrement officiel à l'IANA.

**Exemple** : Minecraft a enregistré le port 25565 pour son serveur multijoueur.

### Conflits de ports

Les ports enregistrés peuvent créer des conflits si plusieurs applications veulent utiliser le même port.

**Exemple de conflit** :

```bash
# Application 1 démarre
$ ./app1
Listening on port 8080... ✓

# Application 2 essaie de démarrer
$ ./app2
Error: Address already in use (port 8080)
```

**Solutions** :
1. Configurer une des applications pour utiliser un port différent
2. Arrêter l'application qui occupe le port
3. Utiliser un reverse proxy (Nginx) pour router les requêtes

## Catégorie 3 : Ports dynamiques/privés (49152-65535)

### Définition et caractéristiques

Les **ports dynamiques** ou **ports privés** (Dynamic/Private Ports) constituent la plage **49152 à 65535**.

**Caractéristiques principales** :
- **Non enregistrables** : impossible de les enregistrer auprès de l'IANA
- **Éphémères** : utilisés temporairement pour des connexions clientes
- **Attribués automatiquement** par le système d'exploitation
- **Libérés** après la fermeture de la connexion

### Rôle des ports éphémères

Ces ports sont utilisés comme **ports source** pour les connexions clientes sortantes.

**Processus** :

```
1. Application cliente : "Je veux me connecter à example.com:443"
                        ↓
2. Système d'exploitation : "Je t'attribue le port source 54231"
                        ↓
3. Connexion établie : 192.168.1.100:54231 → 93.184.216.34:443
                        ↓
4. Échange de données via le port 54231
                        ↓
5. Connexion fermée : port 54231 libéré et peut être réutilisé
```

### Exemple détaillé : navigation web

Vous ouvrez votre navigateur et visitez plusieurs sites :

```
Vous (192.168.1.100)              Internet

Connexion 1 : google.com
  192.168.1.100:54231 ─────────> 142.250.185.46:443
  Port source : 54231 (attribué par l'OS)

Connexion 2 : github.com
  192.168.1.100:54232 ─────────> 140.82.121.4:443
  Port source : 54232 (attribué par l'OS)

Connexion 3 : stackoverflow.com
  192.168.1.100:54233 ─────────> 151.101.1.69:443
  Port source : 54233 (attribué par l'OS)
```

**Observation** : Les trois connexions utilisent le même port destination (443), mais des ports source différents dans la plage éphémère.

### Gestion des ports éphémères par l'OS

Différents systèmes d'exploitation utilisent des plages différentes pour les ports éphémères, même si la norme recommande 49152-65535.

#### Linux

**Plage par défaut** : 32768-60999

```bash
# Afficher la plage actuelle
$ cat /proc/sys/net/ipv4/ip_local_port_range
32768   60999

# Modifier la plage (temporaire)
$ sudo sysctl -w net.ipv4.ip_local_port_range="49152 65535"
```

#### Windows

**Plage par défaut** : 49152-65535 (depuis Windows Vista)

```powershell
# Afficher la plage actuelle
> netsh int ipv4 show dynamicport tcp
Protocol tcp Dynamic Port Range
---------------------------------
Start Port      : 49152
Number of Ports : 16384
```

#### macOS

**Plage par défaut** : 49152-65535

```bash
# Afficher la plage actuelle
$ sysctl net.inet.ip.portrange
net.inet.ip.portrange.first: 49152
net.inet.ip.portrange.last: 65535
```

### Tableau comparatif des plages éphémères

| Système | Plage par défaut | Nombre de ports |
|---------|------------------|-----------------|
| **Linux** | 32768-60999 | 28 232 |
| **Windows** | 49152-65535 | 16 384 |
| **macOS** | 49152-65535 | 16 384 |
| **FreeBSD** | 10000-65535 | 55 536 |

### Épuisement des ports éphémères

Un système peut **manquer de ports éphémères** si trop de connexions sont ouvertes simultanément.

**Scénario problématique** : Serveur applicatif faisant beaucoup d'appels API

```
Serveur API (Python) : 192.168.1.50

Connexions sortantes vers base de données :
  192.168.1.50:54231 → db.local:5432
  192.168.1.50:54232 → db.local:5432
  ...
  192.168.1.50:60999 → db.local:5432

Erreur : "Cannot assign requested address"
→ Tous les ports éphémères sont épuisés !
```

**Solutions** :
1. Augmenter la plage de ports éphémères
2. Utiliser des pools de connexions (connection pooling)
3. Réduire le TIME_WAIT (avec précaution)
4. Répartir la charge sur plusieurs serveurs

### Visualisation des ports éphémères en action

```bash
# Voir les connexions actives avec leurs ports
$ ss -tan
State    Recv-Q Send-Q Local Address:Port  Peer Address:Port

ESTAB    0      0      192.168.1.100:54231 142.250.185.46:443
ESTAB    0      0      192.168.1.100:54232 140.82.121.4:443
ESTAB    0      0      192.168.1.100:54233 151.101.1.69:443
ESTAB    0      0      192.168.1.100:54234 172.217.14.206:443
TIME-WAIT 0     0      192.168.1.100:54230 93.184.216.34:443
```

**Analyse** :
- Ports 54231-54234 : connexions actives (ESTABLISHED)
- Port 54230 : en TIME_WAIT (connexion fermée récemment, port pas encore libéré)

## Cas pratiques d'utilisation

### Cas 1 : Configuration d'un serveur web

**Scénario** : Déployer un site web avec HTTPS

```
Configuration Nginx :

Port 80 (HTTP)  → Redirection vers 443
Port 443 (HTTPS) → Application web

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:8080;  # Port enregistré
    }
}
```

**Flux de connexion** :

```
Client : 203.0.113.5:54231
   │
   ├──> example.com:80 (HTTP)
   │    └──> Redirection 301 → HTTPS
   │
   └──> example.com:443 (HTTPS)
        └──> Nginx → localhost:8080 (Application)
```

### Cas 2 : Développement d'une stack complète

**Scénario** : Application web moderne en développement

```
┌────────────────────────────────────────────────────┐
│              Stack de développement                │
├────────────────────────────────────────────────────┤
│                                                    │
│  Frontend (React)        : localhost:3000          │
│  Backend API (Express)   : localhost:8080          │
│  Base de données (Mongo) : localhost:27017         │
│  Redis Cache             : localhost:6379          │
│  Elasticsearch           : localhost:9200          │
│  Kibana                  : localhost:5601          │
│                                                    │
└────────────────────────────────────────────────────┘

Tous les ports sont dans la plage enregistrée (1024-49151)
sauf Redis (6379) qui est bien connu de facto.
```

### Cas 3 : Configuration firewall

**Scénario** : Sécuriser un serveur en production

```bash
# Autoriser uniquement les ports nécessaires

# SSH (port modifié pour la sécurité)
sudo ufw allow 2222/tcp

# HTTP et HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Bloquer tout le reste par défaut
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Activer le firewall
sudo ufw enable
```

**Règles appliquées** :

```
Ports entrants autorisés :
  - 2222 (SSH modifié)
  - 80 (HTTP)
  - 443 (HTTPS)

Ports entrants bloqués :
  - Tous les autres

Ports sortants :
  - Tous autorisés (pour que le serveur puisse faire des requêtes externes)
  - Les ports éphémères sont utilisés automatiquement
```

### Cas 4 : Debugging de conflit de ports

**Scénario** : Une application ne démarre pas à cause d'un port occupé

```bash
# Erreur rencontrée
$ ./myapp
Error: bind: Address already in use (port 8080)

# Identifier qui utilise le port
$ sudo lsof -i :8080
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
node      1234 user  23u  IPv4  12345      0t0  TCP *:8080 (LISTEN)

# Solution 1 : Arrêter le processus
$ kill 1234

# Solution 2 : Utiliser un autre port
$ ./myapp --port 8081
```

## Bonnes pratiques

### Pour les développeurs

1. **Ne jamais hardcoder les ports** : Utilisez des variables d'environnement

```python
# Mauvais
app.run(port=8080)

# Bon
port = int(os.environ.get("PORT", 8080))
app.run(port=port)
```

2. **Documenter les ports utilisés** : Dans le README de votre projet

```markdown
## Ports utilisés

- Application : 8080 (configurable via PORT)
- Database : 5432
- Redis : 6379
```

3. **Éviter les ports bien connus en développement** : Utilisez 8080, 3000, 5000, etc.

4. **Utiliser des ports standards** : Pour les services connus (MySQL:3306, PostgreSQL:5432)

### Pour les administrateurs système

1. **Changer les ports par défaut des services critiques** (SSH:22 → 2222)

2. **Documenter tous les ports ouverts** : Maintenir un inventaire

```
Serveur web01 :
- 80, 443 : Nginx
- 2222 : SSH
- 5432 : PostgreSQL (localhost only)
```

3. **Surveiller l'utilisation des ports** : Alertes sur épuisement

```bash
# Script de monitoring
#!/bin/bash
CURRENT=$(ss -tan | grep ESTAB | wc -l)
THRESHOLD=50000
if [ $CURRENT -gt $THRESHOLD ]; then
    echo "ALERT: $CURRENT connexions actives"
fi
```

4. **Configurer correctement les pare-feu** : Principe du moindre privilège

### Pour la sécurité

1. **Scanner régulièrement les ports ouverts**

```bash
# Avec nmap (depuis l'extérieur)
$ nmap -p- example.com

# Depuis le serveur lui-même
$ ss -tuln | grep LISTEN
```

2. **Fermer les ports inutilisés**

3. **Utiliser des outils de détection d'intrusion** : Fail2Ban, etc.

4. **Chiffrer les communications** : Préférer 443 (HTTPS) à 80 (HTTP)

## Tableau récapitulatif complet

```
┌─────────────────────────────────────────────────────────────┐
│                   CLASSIFICATION DES PORTS                  │
├──────────┬──────────────┬─────────────────┬─────────────────┤
│ PLAGE    │ CATÉGORIE    │ GESTION         │ USAGE           │
├──────────┼──────────────┼─────────────────┼─────────────────┤
│ 0-1023   │ Bien connus  │ IANA (strict)   │ Services        │
│          │ (Well-Known) │ Privilèges root │ système         │
│          │              │                 │ standards       │
├──────────┼──────────────┼─────────────────┼─────────────────┤
│ 1024-    │ Enregistrés  │ IANA (souple)   │ Applications    │
│ 49151    │ (Registered) │ Pas de          │ tierces,        │
│          │              │ privilèges      │ services custom │
├──────────┼──────────────┼─────────────────┼─────────────────┤
│ 49152-   │ Dynamiques   │ Libres          │ Ports source    │
│ 65535    │ (Dynamic)    │ OS les attribue │ éphémères       │
│          │              │ automatiquement │ (clients)       │
└──────────┴──────────────┴─────────────────┴─────────────────┘
```

## Ressources et références

### Documents officiels

- **IANA Port Numbers** : https://www.iana.org/assignments/service-names-port-numbers/
- **RFC 6335** : Internet Assigned Numbers Authority (IANA) Procedures for Port Number Assignment

### Fichiers système

Sur les systèmes Unix/Linux, les associations port-service sont définies dans :

```bash
$ cat /etc/services
# Port      Service     Protocol
ftp         21/tcp
ssh         22/tcp
smtp        25/tcp
http        80/tcp
https       443/tcp
```

Ce fichier est utilisé par de nombreux outils pour résoudre les noms de services.

## Conclusion

La classification des ports en trois catégories (bien connus, enregistrés, dynamiques) est fondamentale pour le fonctionnement d'Internet :

**Ports bien connus (0-1023)** :
- ✅ Universellement standardisés
- ✅ Services système essentiels
- ✅ Nécessitent des privilèges élevés
- ✅ Exemples : HTTP (80), HTTPS (443), SSH (22)

**Ports enregistrés (1024-49151)** :
- ✅ Semi-standardisés par l'IANA
- ✅ Applications tierces et services personnalisés
- ✅ Pas de privilèges requis
- ✅ Exemples : MySQL (3306), PostgreSQL (5432)

**Ports dynamiques (49152-65535)** :
- ✅ Attribués automatiquement par l'OS
- ✅ Ports source pour connexions clientes
- ✅ Temporaires et réutilisables
- ✅ Essentiels pour le multiplexage client

Cette organisation permet :
- Une **standardisation mondiale** des services
- Un **multiplexage efficace** des connexions
- Une **gestion de sécurité** cohérente
- Une **interopérabilité** entre tous les systèmes

Dans la section suivante, nous plongerons dans le protocole UDP, le plus simple des deux protocoles de la couche Transport, pour comprendre son fonctionnement et ses cas d'usage spécifiques.

---

**À retenir** :

- ✅ **0-1023** : Ports bien connus, services standards, privilèges requis
- ✅ **1024-49151** : Ports enregistrés, applications tierces, pas de privilèges
- ✅ **49152-65535** : Ports dynamiques, attribués automatiquement par l'OS
- ✅ Port **destination** : généralement fixe et connu
- ✅ Port **source** : généralement éphémère (plage dynamique)
- ✅ HTTP = 80, HTTPS = 443, SSH = 22, MySQL = 3306, PostgreSQL = 5432
- ✅ Éviter les conflits : un seul service par port à la fois
- ✅ Sécurité : fermer les ports inutilisés, changer les ports par défaut des services critiques

⏭️ [UDP (User Datagram Protocol)](/04-couche-transport/04-udp.md)
