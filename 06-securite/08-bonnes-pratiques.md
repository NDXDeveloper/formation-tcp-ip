🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.8 Bonnes pratiques de sécurisation

## Introduction

La sécurisation d'un réseau TCP/IP ne se résume pas à déployer quelques outils de sécurité. C'est une approche globale, systématique et continue qui combine technologies, procédures, et formation humaine. Ce chapitre synthétise les meilleures pratiques pour créer et maintenir un réseau sécurisé.

```
Analogie de la sécurité réseau :

Sécuriser un réseau = Sécuriser une maison

Mauvaise approche :
"J'ai installé une porte blindée"
→ Mais fenêtres ouvertes
→ Mais clé sous le paillasson
→ Mais pas d'alarme
→ Mais jamais vérifier qui entre

Bonne approche (Defense in Depth) :
✓ Porte blindée (pare-feu)
✓ Fenêtres sécurisées (services durcis)
✓ Alarme (IDS/IPS)
✓ Caméras (logging/monitoring)
✓ Voisinage vigilant (threat intelligence)
✓ Coffre-fort interne (chiffrement)
✓ Plan évacuation (incident response)
✓ Assurance (backup/DR)
✓ Formation famille (security awareness)

Résultat :
Plusieurs couches de protection
Si une couche échoue → autres compensent
```

## Principes fondamentaux de sécurité

### 1. Defense in Depth (Défense en profondeur)

**Concept** : Multiples couches de sécurité indépendantes.

```
Architecture en couches :

┌───────────────────────────────────────────────┐
│ Couche 7 : Formation utilisateurs             │
│ (Social Engineering awareness)                │
├───────────────────────────────────────────────┤
│ Couche 6 : Policies et procédures             │
│ (Acceptable Use Policy, Incident Response)    │
├───────────────────────────────────────────────┤
│ Couche 5 : Application Security               │
│ (Code review, WAF, Input validation)          │
├───────────────────────────────────────────────┤
│ Couche 4 : Identity & Access                  │
│ (MFA, RBAC, Least Privilege)                  │
├───────────────────────────────────────────────┤
│ Couche 3 : Network Security                   │
│ (Firewall, IPS, Segmentation)                 │
├───────────────────────────────────────────────┤
│ Couche 2 : Host Security                      │
│ (Antivirus, Patching, Hardening)              │
├───────────────────────────────────────────────┤
│ Couche 1 : Physical Security                  │
│ (Data center access, Locked racks)            │
└───────────────────────────────────────────────┘

Si attaquant bypass une couche :
→ Reste 6 autres couches à franchir
→ Chaque couche ralentit, détecte, ou bloque
```

**Exemple concret : Attaque web**

```
Scénario : Attaquant tente SQL injection

Sans Defense in Depth :
1. Application vulnérable → SQL injection réussie
   → Base de données compromise
   → Game over

Avec Defense in Depth :

Couche 1 - WAF (Web Application Firewall) :
  Détecte pattern SQL injection
  → Bloque requête
  ✓ Attaque arrêtée

Mais si contournement WAF...

Couche 2 - Application input validation :
  Vérifie format données
  Échappe caractères spéciaux
  → SQL injection neutralisée
  ✓ Attaque arrêtée

Mais si bug validation...

Couche 3 - Database least privilege :
  Compte application a droits minimaux
  Pas de DROP TABLE, pas de fichiers système
  → Impact limité même si injection
  ✓ Dommages contenus

Couche 4 - Network segmentation :
  Base de données dans VLAN isolé
  Firewall bloque accès direct depuis Internet
  → Exfiltration difficile
  ✓ Données protégées

Couche 5 - Database activity monitoring :
  Détecte requêtes anormales
  Alerte sécurité
  → Incident détecté et réponse rapide
  ✓ Limitation temps d'exposition

Couche 6 - Encryption at rest :
  Données chiffrées sur disque
  → Même si exfiltration, données illisibles
  ✓ Confidentialité préservée

Couche 7 - Backup and Recovery :
  Sauvegardes régulières
  → Restauration possible si corruption
  ✓ Business continuity assurée

Résultat :
Attaque détectée et bloquée à multiples niveaux
Même si partiellement réussie → impact minimal
```

### 2. Least Privilege (Moindre privilège)

**Principe** : Donner uniquement les permissions strictement nécessaires.

```
Mauvaise pratique :

Tous les employés :
- Administrateur local machine
- Accès tous serveurs
- Accès toutes données
- Droits écriture partout

Problème :
Compte compromis → Accès total
Erreur humaine → Dommages massifs
Insider threat → Exfiltration facile

Bonne pratique :

User Alice (Marketing) :
✓ Lecture : Dossiers Marketing
✓ Écriture : Son dossier personnel
✓ Applications : Office, Adobe, CRM
✗ Administrateur : NON
✗ Accès : Finance, RH, Engineering
✗ Installation : Logiciels

Admin Bob (IT) :
✓ Administrateur : Serveurs (avec compte séparé)
✓ Accès : Systèmes IT
✗ Compte standard : Usage quotidien (email, web)
✗ Admin rights : Compte personnel

Résultat :
Compromission Alice → Impact limité à Marketing
Malware sur PC Alice → Pas de propagation (pas admin)
Bob quotidien → Compte standard (pas admin)
Bob admin → Seulement quand nécessaire
```

**Implémentation concrète**

```
Serveur Web :

Mauvaise configuration :
# Apache running as root
User root
Group root

Problème :
Vulnérabilité Apache → Attaquant devient root
→ Contrôle total serveur

Bonne configuration :
# Apache running as www-data
User www-data
Group www-data

# www-data permissions
/var/www/html : read-only
/var/log/apache2 : write (logs uniquement)
/tmp : no access (utiliser /var/tmp/apache2)

# Directories permissions
chown -R root:root /var/www/html
chmod -R 755 /var/www/html
chown -R www-data:www-data /var/log/apache2

Résultat :
Compromission Apache → Attaquant a droits www-data seulement
Pas d'accès /etc, /root, autres users
Pas de modification code source (read-only)

Base de données :

Mauvaise pratique :
# Application uses 'root' MySQL user
DB_USER=root
DB_PASS=rootpassword

Bonne pratique :
# Dedicated user with minimal privileges
CREATE USER 'webapp'@'localhost' IDENTIFIED BY 'strong_password';

# Only necessary permissions
GRANT SELECT, INSERT, UPDATE, DELETE ON webapp_db.* TO 'webapp'@'localhost';

# Explicitly deny dangerous commands
REVOKE DROP, CREATE, ALTER ON webapp_db.* FROM 'webapp'@'localhost';

# Application config
DB_USER=webapp
DB_PASS=strong_password

Résultat :
SQL injection limitée à SELECT, INSERT, UPDATE, DELETE
Pas de DROP DATABASE
Pas de lecture autres databases
Pas de fichiers système (LOAD_FILE)
```

### 3. Zero Trust Architecture

**Principe** : "Never trust, always verify" - Ne jamais faire confiance, toujours vérifier.

```
Modèle traditionnel (Castle-and-Moat) :

┌─────────────────────────────────────┐
│         Firewall (Moat)             │
└──────────────┬──────────────────────┘
               │
        ┌──────▼──────┐
        │   Trusted   │  ← Inside = Trusted
        │   Network   │  ← No verification
        │  (Castle)   │  ← Lateral movement easy
        └─────────────┘

Problème :
✗ Insider threats
✗ Compromission initiale → libre circulation
✗ VPN = full trust
✗ Pas de micro-segmentation

Modèle Zero Trust :

┌───────────────────────────────────────────┐
│ Verify EVERY access                       │
│ - User authentication (who)               │
│ - Device posture (what)                   │
│ - Context (when, where, how)              │
│ - Least privilege (minimum access)        │
│ - Continuous verification                 │
└───────────────────────────────────────────┘

Chaque ressource :
- Identity verification (MFA)
- Device compliance check
- Geo-location validation
- Time-based access
- Anomaly detection

Résultat :
✓ Breach ne donne pas accès total
✓ Lateral movement bloqué
✓ Granular access control
✓ Audit complet
```

**Implémentation Zero Trust**

```
Exemple : Accès serveur de fichiers

Traditional :
User sur VPN → Accès tous shares réseau
Pas de vérification additionnelle

Zero Trust :

1. Identity Verification :
   User Alice demande \\fileserver\finance

   Vérifications :
   ✓ Alice authentifiée ? (SSO/SAML)
   ✓ MFA complété ? (TOTP, push notification)
   ✓ Device reconnu ? (Certificate, device ID)
   ✓ Device compliant ? (Antivirus à jour, patches, encryption)

2. Policy Evaluation :
   User : Alice
   Group : Finance_Team
   Resource : Finance share
   Time : 14:30 (business hours)
   Location : Office IP / Known VPN
   Device : Managed laptop

   Policy :
   IF Finance_Team AND business_hours AND managed_device
   THEN ALLOW read/write Finance share

3. Least Privilege :
   Alice obtient accès :
   ✓ Finance share : read/write
   ✗ HR share : denied
   ✗ Engineering share : denied
   ✗ Server admin : denied

4. Continuous Verification :
   Toutes les 15 minutes : Re-vérifier
   - Session still valid ?
   - Device still compliant ?
   - Location changed ? (geo-fencing)

   Si anomalie :
   → Challenge MFA
   → OU revoke access

5. Audit :
   Log complet :
   - Who : alice@company.com
   - What : Accessed \\fileserver\finance\Q4_report.xlsx
   - When : 2024-12-06 14:35:22
   - Where : Office network
   - How : Managed laptop
   - Result : ALLOWED

Technologies Zero Trust :
- BeyondCorp (Google)
- Azure AD Conditional Access
- Okta
- Zscaler
- Cloudflare Access
- Duo Security
```

## Sécurisation par couche réseau

### Couche Physique et Liaison

**Sécurité physique**

```
Data Center :

1. Contrôle d'accès :
   ✓ Badge électronique
   ✓ Biométrie (empreinte digitale)
   ✓ Mantrap (sas de sécurité)
   ✓ Caméras 24/7
   ✓ Gardiens
   ✓ Log tous accès

2. Sécurité équipements :
   ✓ Racks verrouillés
   ✓ Câbles sécurisés (anti-arrachement)
   ✓ Port console password
   ✓ Asset tags (traçabilité)

3. Environnement :
   ✓ Alimentation redondante (UPS)
   ✓ Climatisation
   ✓ Détection incendie
   ✓ Système d'extinction

Bureau :

1. Postes de travail :
   ✓ Cable lock (Kensington)
   ✓ Screen lock automatique (5 min)
   ✓ Full disk encryption (BitLocker, FileVault)
   ✓ BIOS password

2. Réseau :
   ✓ Ports Ethernet inactifs désactivés
   ✓ MAC address filtering (ou 802.1X)
   ✓ VLAN séparé invités
   ✓ Patch panels sécurisés
```

**Port Security**

```
Problème : Accès physique réseau = accès logique

Attaque possible :
1. Attaquant branche laptop dans bureau
2. Obtient adresse DHCP
3. Accès réseau interne
4. Scanne, attaque, exfiltre

Protection Switch (Cisco) :

# Port security basique
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky

Résultat :
- Maximum 2 MAC addresses sur ce port
- MAC addresses "learned" et sauvegardées
- Si 3ème MAC : port restrict (drop + log)

# Violation modes
shutdown : Port désactivé (défaut, sécurisé)
restrict : Drop paquets, log, compteur
protect : Drop paquets, pas de log

# Port security avancé (802.1X)
interface GigabitEthernet0/1
 switchport mode access
 authentication port-control auto
 dot1x pam authenticator 1

Processus 802.1X :
1. Device connecté → Port bloqué (sauf EAPOL)
2. Device envoie credentials (certificate, username/password)
3. Switch forward vers RADIUS
4. RADIUS vérifie credentials
5. Si OK : Switch ouvre port + assigne VLAN
6. Si NOK : Port reste bloqué

Résultat :
Seulement devices authentifiés obtiennent accès réseau
```

**VLAN et Segmentation Layer 2**

```
Mauvaise pratique : Flat network

┌────────────────────────────────────────────┐
│  Single VLAN (VLAN 1)                      │
│  - Users                                   │
│  - Servers                                 │
│  - Printers                                │
│  - IoT devices                             │
│  - Guests                                  │
│  ALL on 192.168.1.0/24                     │
└────────────────────────────────────────────┘

Problèmes :
✗ ARP spoofing facile (tout même broadcast domain)
✗ Sniffing (mode promiscuous voit tout)
✗ Compromission un device → accès tous
✗ Pas de contrôle inter-device

Bonne pratique : Segmentation VLAN

VLAN 10 : Users (10.10.0.0/24)
  - Postes de travail
  - Laptops

VLAN 20 : Servers (10.20.0.0/24)
  - File servers
  - Application servers

VLAN 30 : Database (10.30.0.0/24)
  - Database servers

VLAN 40 : Management (10.40.0.0/24)
  - Switch management
  - Server iLO/IPMI

VLAN 50 : VoIP (10.50.0.0/24)
  - IP Phones

VLAN 60 : IoT (10.60.0.0/24)
  - Cameras
  - Sensors
  - Smart devices

VLAN 99 : Guest (10.99.0.0/24)
  - Visitor Wi-Fi

Firewall rules inter-VLAN :
Users → Servers : HTTP, HTTPS, SMB
Users → Database : DENY (passe par App tier)
Users → Management : DENY
Servers → Database : MySQL, PostgreSQL
IoT → Users : DENY
Guest → Internal : DENY (Internet only)

Résultat :
✓ Isolation broadcast domains
✓ Segmentation sécurité
✓ Contrôle flux avec ACL/firewall
✓ Compromission isolée par VLAN
```

### Couche Réseau (IP)

**Filtrage et ACL**

```
Router ACL (Access Control List) :

# Inbound interface (Internet-facing)
ip access-list extended INTERNET_IN

 ! Allow established connections
 permit tcp any any established

 ! Allow specific services
 permit tcp any host 203.0.113.10 eq 80  (Web server)
 permit tcp any host 203.0.113.10 eq 443
 permit tcp any host 203.0.113.20 eq 25  (Mail server)

 ! Block bogon networks (never from Internet)
 deny ip 10.0.0.0 0.255.255.255 any
 deny ip 172.16.0.0 0.15.255.255 any
 deny ip 192.168.0.0 0.0.255.255 any
 deny ip 127.0.0.0 0.255.255.255 any
 deny ip 169.254.0.0 0.0.255.255 any
 deny ip 224.0.0.0 15.255.255.255 any (multicast)

 ! Block known bad actors (example)
 deny ip host 198.51.100.50 any

 ! Log and deny rest
 deny ip any any log

interface GigabitEthernet0/0
 ip access-group INTERNET_IN in

# Outbound (anti-spoofing)
ip access-list extended INTERNET_OUT

 ! Only allow from our networks
 permit ip 10.0.0.0 0.255.255.255 any
 permit ip 203.0.113.0 0.0.0.255 any

 ! Deny spoofed sources
 deny ip any any log

interface GigabitEthernet0/0
 ip access-group INTERNET_OUT out

Résultat :
✓ Filtre trafic entrant (services autorisés seulement)
✓ Bloque RFC1918 depuis Internet (spoofing)
✓ Anti-spoofing sortant (BCP 38)
✓ Logging anomalies
```

**Désactivation services inutiles**

```
Durcissement (Hardening) serveur Linux :

# 1. Lister services actifs
systemctl list-unit-files --state=enabled

# 2. Désactiver services inutiles
systemctl disable bluetooth
systemctl disable cups (printing)
systemctl disable avahi-daemon (zeroconf)
systemctl disable rpcbind (NFS)

# 3. Désactiver modules kernel inutiles
# /etc/modprobe.d/blacklist.conf
blacklist usb-storage  (si pas besoin USB)
blacklist firewire
blacklist bluetooth

# 4. Services réseau minimaux
# Si serveur web seulement : SSH + HTTP/HTTPS
netstat -tulpn
tcp  0.0.0.0:22    LISTEN  (SSH - nécessaire)
tcp  0.0.0.0:80    LISTEN  (HTTP - nécessaire)
tcp  0.0.0.0:443   LISTEN  (HTTPS - nécessaire)

# Tout le reste : fermer ou désactiver

# 5. IPv6 si non utilisé
# /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# 6. Protocoles inutiles
# Désactiver si pas utilisé :
- Telnet (utiliser SSH)
- FTP (utiliser SFTP/SCP)
- TFTP
- Finger
- SNMP v1/v2 (utiliser v3 si nécessaire)

Résultat :
Surface d'attaque minimale
Seulement services essentiels actifs
```

**Rate Limiting et Anti-DDoS**

```
Protection contre flood :

# iptables
# Limiter nouvelles connexions SSH (anti-bruteforce)
iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
  -m recent --set --name SSH

iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH \
  -j DROP

# Limiter ICMP (anti-ping flood)
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/s --limit-burst 10 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# SYN flood protection (SYN cookies)
sysctl -w net.ipv4.tcp_syncookies=1
sysctl -w net.ipv4.tcp_max_syn_backlog=2048
sysctl -w net.ipv4.tcp_synack_retries=2

# Nginx rate limiting
http {
  limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

  server {
    location / {
      limit_req zone=one burst=20 nodelay;
    }
  }
}

Résultat :
✓ Max 4 connexions SSH/minute par IP
✓ ICMP limité 1/seconde
✓ SYN cookies si flood
✓ HTTP limité 10 req/s par IP

DDoS Protection avancé :

Niveau 1 - ISP/Upstream :
- BGP Flowspec
- Blackholing
- Scrubbing centers

Niveau 2 - CDN/Cloud :
- Cloudflare
- AWS Shield
- Akamai

Niveau 3 - Local :
- Rate limiting (ci-dessus)
- Firewall rules
- IPS (Suricata, Snort)
```

### Couche Transport

**Sécurisation TCP/UDP**

```
TCP Hardening (sysctl Linux) :

# /etc/sysctl.conf

# SYN flood protection
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Protection TIME_WAIT attacks
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_tw_reuse = 1

# Augmenter limites connexions
net.core.somaxconn = 1024
net.ipv4.ip_local_port_range = 1024 65535

# Protection contre IP spoofing
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignorer ICMP redirects (MitM)
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Ignorer source routed packets
net.ipv4.conf.all.accept_source_route = 0

# Log martian packets (anomalies)
net.ipv4.conf.all.log_martians = 1

Appliquer :
sysctl -p

Ports sensibles à protéger :

Fermer par défaut :
✗ 23 (Telnet) - jamais utiliser
✗ 21 (FTP) - utiliser SFTP
✗ 69 (TFTP) - rarement nécessaire
✗ 161/162 (SNMP v1/v2) - v3 seulement
✗ 135-139, 445 (SMB) - Internet jamais
✗ 3389 (RDP) - VPN ou bastion seulement

Ouvrir seulement si nécessaire :
✓ 22 (SSH) - avec key auth, fail2ban
✓ 80/443 (HTTP/HTTPS) - web servers
✓ 25/587 (SMTP) - mail servers
✓ 53 (DNS) - DNS servers
✓ Custom ports - documenter

Fail2Ban (protection bruteforce) :

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
logpath = /var/log/nginx/error.log
maxretry = 5
bantime = 1800

Résultat :
3 échecs SSH en 10 min → Ban IP 1 heure
5 échecs HTTP auth en 10 min → Ban 30 min
```

### Couche Application

**HTTPS obligatoire**

```
Configuration Nginx sécurisée :

server {
    # Redirect HTTP to HTTPS
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # Certificates
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # Modern SSL configuration (Mozilla recommendations)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/nginx/ssl/chain.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' https:; script-src 'self' 'unsafe-inline' 'unsafe-eval' https:; style-src 'self' 'unsafe-inline' https:;" always;

    # Disable server tokens
    server_tokens off;

    # Session timeout
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # DH parameters
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    location / {
        # Application
        proxy_pass http://backend;

        # Security headers for proxied content
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $host;

        # Hide backend headers
        proxy_hide_header X-Powered-By;
    }
}

Test configuration :
# SSL Labs
https://www.ssllabs.com/ssltest/
Grade A+ attendu

# Security Headers
https://securityheaders.com/
Grade A attendu
```

**Application Security**

```
Input Validation (PHP exemple) :

Mauvais code (vulnérable SQL injection) :
<?php
$id = $_GET['id'];
$query = "SELECT * FROM users WHERE id = $id";
$result = mysqli_query($conn, $query);
?>

Attaque :
https://example.com/user.php?id=1 OR 1=1
→ Retourne tous les users

Bon code (prepared statements) :
<?php
$id = $_GET['id'];

// Validation
if (!is_numeric($id)) {
    die("Invalid input");
}

// Prepared statement
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
$result = $stmt->get_result();
?>

XSS Prevention :

Mauvais code :
<?php
echo "Hello " . $_GET['name'];
?>

Attaque :
https://example.com/?name=<script>alert('XSS')</script>

Bon code :
<?php
echo "Hello " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
?>

CSRF Prevention :

<!-- Form avec CSRF token -->
<form method="POST" action="/transfer">
    <input type="hidden" name="csrf_token" value="<?php echo $_SESSION['csrf_token']; ?>">
    <input type="text" name="amount">
    <input type="submit" value="Transfer">
</form>

<?php
// Vérification server-side
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    die("CSRF token invalid");
}
// Process form
?>

File Upload Security :

Mauvais :
<?php
move_uploaded_file($_FILES['file']['tmp_name'], '/uploads/' . $_FILES['file']['name']);
?>

Bon :
<?php
// Whitelist extensions
$allowed = ['jpg', 'png', 'pdf'];
$extension = pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION);

if (!in_array(strtolower($extension), $allowed)) {
    die("File type not allowed");
}

// Vérifier MIME type
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);
if ($mime !== 'image/jpeg' && $mime !== 'image/png' && $mime !== 'application/pdf') {
    die("Invalid file type");
}

// Nom sécurisé (random)
$filename = bin2hex(random_bytes(16)) . '.' . $extension;

// Upload hors webroot si possible
move_uploaded_file($_FILES['file']['tmp_name'], '/var/uploads/' . $filename);
?>
```

## Gestion des identités et accès

### Authentification forte

**Multi-Factor Authentication (MFA)**

```
Facteurs d'authentification :

1. Something you know :
   - Password
   - PIN
   - Security questions

2. Something you have :
   - Smartphone (TOTP app)
   - Hardware token (YubiKey)
   - Smart card
   - SMS (moins sûr)

3. Something you are :
   - Fingerprint
   - Face recognition
   - Iris scan
   - Voice

MFA = Au moins 2 facteurs différents

Implémentation TOTP (Google Authenticator) :

# Installation (Linux)
apt-get install libpam-google-authenticator

# Configuration user
google-authenticator

Options :
- Time-based tokens : YES
- Update .google_authenticator : YES
- Disallow multiple uses : YES
- Rate limiting : YES
- Emergency scratch codes : YES (sauvegarder !)

# PAM configuration
# /etc/pam.d/sshd
auth required pam_google_authenticator.so

# SSH configuration
# /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
UsePAM yes

# Restart SSH
systemctl restart sshd

Connexion :
ssh user@server
Password: ********
Verification code: 123456

Résultat :
Password seul insuffisant
Nécessite TOTP code (change toutes les 30s)
```

**Password Policies**

```
Politique de mots de passe forte :

Exigences minimales :
✓ Longueur minimum : 14 caractères (16+ mieux)
✓ Complexité :
  - Majuscules
  - Minuscules
  - Chiffres
  - Caractères spéciaux (!@#$%^&*)
✓ Pas de mots du dictionnaire
✓ Pas de patterns (123456, qwerty, password)
✓ Pas de données personnelles (nom, date naissance)
✓ Expiration : 90 jours (ou passphrase longue sans expiration)
✓ Historique : 24 derniers passwords (pas de réutilisation)

Implémentation Linux (PAM) :

# /etc/pam.d/common-password
password requisite pam_pwquality.so \
  retry=3 \
  minlen=14 \
  dcredit=-1 \
  ucredit=-1 \
  ocredit=-1 \
  lcredit=-1

# /etc/login.defs
PASS_MAX_DAYS 90
PASS_MIN_DAYS 1
PASS_WARN_AGE 7

Active Directory GPO :

Computer Configuration → Policies → Windows Settings →
  Security Settings → Account Policies → Password Policy

- Minimum password length: 14
- Password must meet complexity requirements: Enabled
- Maximum password age: 90 days
- Minimum password age: 1 day
- Enforce password history: 24 passwords

Alternative moderne : Passphrases

Au lieu de : P@ssw0rd123! (difficile à retenir, facile à craquer)
Utiliser : correct-horse-battery-staple (facile à retenir, difficile à craquer)

Longueur > Complexité pour entropie

Password Manager :

Recommander aux utilisateurs :
- 1Password
- Bitwarden
- LastPass
- KeePass

Avantages :
✓ Passwords uniques par service
✓ Génération passwords forts
✓ Auto-fill (protection keyloggers)
✓ Audit passwords faibles
```

### Gestion des comptes privilégiés

**Comptes administrateurs**

```
Séparation comptes :

Mauvaise pratique :
User "admin" :
- Usage quotidien (email, web)
- Administration systèmes
- Un seul compte pour tout

Bonne pratique :
User "alice" :
- Usage quotidien
- Pas de droits admin

User "alice-admin" :
- Administration seulement
- Utilisé seulement quand nécessaire
- MFA obligatoire
- Sessions courtes (timeout)
- Logging renforcé

Implémentation :

# sudo pour élévation ponctuelle
# /etc/sudoers
alice ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
alice ALL=(ALL) /usr/bin/apt-get

# Nécessite password pour sudo
# Timeout après 5 minutes

# Logging sudo
Defaults logfile=/var/log/sudo.log
Defaults log_year, log_host, log_input, log_output

# Jump server / Bastion host
Production servers pas d'accès SSH direct

User → Bastion (MFA) → Production Servers
           ↑
      Logging complet
      Session recording

Avantages :
✓ Séparation duties
✓ Audit trail complet
✓ Credentials compromis = impact limité
✓ Compliance (SOX, PCI-DSS)
```

**PAM (Privileged Access Management)**

```
Solutions PAM :

Commercial :
- CyberArk
- BeyondTrust
- Thycotic Secret Server
- Delinea

Open-source :
- HashiCorp Vault
- Teleport
- Apache Guacamole

Fonctionnalités :

1. Password Vaulting :
   Passwords stockés chiffrés
   Rotation automatique
   Checkout/checkin

2. Session Management :
   Sessions enregistrées (video)
   Terminable à distance
   Timeout automatique

3. Just-in-Time Access :
   Privileges temporaires
   Approval workflow
   Auto-révocation

Exemple workflow :

Alice (dev) needs production DB access :

1. Request access (ticket system)
2. Manager approves
3. PAM grants :
   - Credentials DB read-only
   - Valid 2 hours
   - Session recorded
4. Alice connects via PAM proxy
5. After 2 hours : Access auto-revoked
6. Session recording retained (audit)

Résultat :
✓ Zero standing privileges
✓ Audit complet
✓ Credentials jamais exposés
```

## Monitoring et détection

### Logging centralisé

**SIEM (Security Information and Event Management)**

```
Architecture SIEM :

Sources de logs :
- Firewalls
- IDS/IPS
- Servers (syslog)
- Applications (web, database)
- Active Directory
- Network devices (switches, routers)
- Endpoints (workstations)
- Cloud services (AWS, Azure, O365)

                    ↓

        ┌───────────────────┐
        │  Log Collectors   │
        │  (Agents/Syslog)  │
        └─────────┬─────────┘
                  │
                  ↓
        ┌───────────────────┐
        │   SIEM Platform   │
        │  (Splunk, ELK,    │
        │   QRadar, etc.)   │
        └─────────┬─────────┘
                  │
        ├─────────┼─────────┤
        ↓         ↓         ↓
    Correlation Alerting Dashboards
    Rules      Security  Reporting
               Team

Solutions SIEM :

Commercial :
- Splunk Enterprise Security
- IBM QRadar
- ArcSight
- LogRhythm

Open-source :
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Graylog
- OSSEC
- Wazuh

Configuration exemple (syslog-ng) :

# /etc/syslog-ng/syslog-ng.conf

# Source locale
source s_local {
    system();
    internal();
};

# Source réseau (autres serveurs)
source s_network {
    tcp(port(514));
    udp(port(514));
};

# Destination SIEM
destination d_siem {
    tcp("siem.company.com" port(514));
};

# Logs
log {
    source(s_local);
    source(s_network);
    destination(d_siem);
};

Corrélation events (exemple règle) :

Rule : Bruteforce SSH detection

IF :
  Event type : SSH authentication failure
  Source IP : Same IP
  Count : >= 5
  Timeframe : 5 minutes
THEN :
  Alert : High severity
  Action : Block IP (firewall)
  Notify : Security team
```

### Détection d'intrusions (IDS/IPS)

```
IDS vs IPS :

IDS (Intrusion Detection System) :
- Mode passif (monitoring)
- Détecte et alerte
- Pas de blocage
- Span port / TAP

IPS (Intrusion Prevention System) :
- Mode inline
- Détecte et bloque
- Action automatique
- Dans le flux réseau

Architecture :

        Internet
            │
            ↓
        [Firewall]
            │
            ├──────────→ [IDS] (span port)
            │               ↓
            │           Alertes
            ↓
        [IPS inline]
            │
            ↓
        Internal Network

Suricata (IDS/IPS open-source) :

# Installation
apt-get install suricata

# Configuration /etc/suricata/suricata.yaml
vars:
  address-groups:
    HOME_NET: "[10.0.0.0/8]"
    EXTERNAL_NET: "!$HOME_NET"

  port-groups:
    HTTP_PORTS: "80"
    HTTPS_PORTS: "443"

# Rules
rule-files:
  - suricata.rules
  - emerging-threats.rules

# Mode IPS
af-packet:
  - interface: eth0
    threads: 4
    cluster-id: 99
    cluster-type: cluster_flow

# Logs
outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: eve.json
      types:
        - alert
        - http
        - dns
        - tls
        - files

Exemple règles :

# Détecter scan de ports
alert tcp any any -> $HOME_NET any (msg:"Port scan detected"; \
  flags:S; threshold:type threshold, track by_src, count 20, seconds 60; \
  sid:1000001; rev:1;)

# Détecter SQL injection
alert http any any -> $HOME_NET any (msg:"SQL Injection attempt"; \
  content:"UNION"; nocase; content:"SELECT"; nocase; \
  sid:1000002; rev:1;)

# Détecter reverse shell
alert tcp $HOME_NET any -> $EXTERNAL_NET any (msg:"Potential reverse shell"; \
  content:"/bin/bash"; \
  sid:1000003; rev:1;)

Tuning (réduire faux positifs) :

1. Baseline :
   Observer trafic normal 1-2 semaines
   Identifier patterns légitimes

2. Whitelist :
   Exempter trafic connu et légitime

   suppress gen_id 1, sig_id 1000002, track by_src, ip 10.0.0.50
   # Ne pas alerter SQL injection depuis 10.0.0.50 (app server)

3. Threshold :
   Ajuster sensibilité

   threshold gen_id 1, sig_id 1000001, type limit, \
     track by_src, count 1, seconds 60
   # Max 1 alerte/minute par IP

4. Priorités :
   Classifier par criticité

   priority: 1 (critical)
   priority: 2 (high)
   priority: 3 (medium)
   priority: 4 (low)
```

### Network Behavior Analysis

```
Détection d'anomalies :

Baseline normal :
- Traffic volume : 100 MB/s ± 20%
- Connexions/sec : 500 ± 100
- Top talkers : 10.0.0.5 (web), 10.0.0.10 (db)
- Protocols : 80% HTTPS, 15% SSH, 5% autres
- Geo : 90% domestic, 10% international

Anomalie détectée :

Event : Traffic spike
Time : 02:00 AM
Volume : 500 MB/s (5x normal)
Source : 10.0.0.25 (workstation)
Destination : 203.0.113.50 (unknown external)
Protocol : HTTPS (port 443)

Analysis :
✗ Workstation pas de trafic normal 02:00
✗ Volume extrêmement élevé
✗ Destination inconnue
✗ Possible data exfiltration

Action :
1. Alert security team
2. Block 10.0.0.25 → 203.0.113.50 (firewall)
3. Isolate 10.0.0.25 (quarantine VLAN)
4. Investigate endpoint (forensics)
5. Check if data encrypted/exfiltrated

Outils :

- Darktrace (AI/ML)
- Vectra AI
- ExtraHop
- ntopng
- Zeek (formerly Bro)

Zeek (example) :

# /usr/local/zeek/share/zeek/site/local.zeek

# Log SSH connections
@load protocols/ssh/detect-bruteforcing

# Detect large file transfers
event file_new(f: fa_file)
    {
    if ( f$total_bytes > 100000000 )  # 100 MB
        {
        print fmt("Large file transfer: %s bytes", f$total_bytes);
        }
    }

# Detect unusual DNS queries
event dns_request(c: connection, msg: dns_msg, query: string)
    {
    if ( /[0-9]{10,}\./ in query )  # Suspicious domain
        {
        print fmt("Suspicious DNS: %s", query);
        }
    }
```

## Patch Management et Mises à jour

### Processus de patching

```
Cycle de vie patch :

1. RELEASE (Vendor) :
   Microsoft : Patch Tuesday (2nd Tuesday/month)
   Linux : Rolling (continuous)
   Applications : Varies

2. ASSESSMENT :
   ┌────────────────────────────────────┐
   │ Receive patch notification         │
   ├────────────────────────────────────┤
   │ Evaluate :                         │
   │ - Criticality (CVSS score)         │
   │ - Applicability (affects us ?)     │
   │ - Impact (breaking changes ?)      │
   │ - Dependencies                     │
   └────────────────────────────────────┘

3. TESTING :
   ┌────────────────────────────────────┐
   │ Dev environment                    │
   │ - Deploy patch                     │
   │ - Functional testing               │
   │ - Performance testing              │
   ├────────────────────────────────────┤
   │ QA environment                     │
   │ - User acceptance testing          │
   │ - Integration testing              │
   └────────────────────────────────────┘

4. DEPLOYMENT :
   ┌────────────────────────────────────┐
   │ Prioritization :                   │
   │                                    │
   │ Critical (0-1 day) :               │
   │ - CVSS 9-10                        │
   │ - Active exploitation              │
   │ - Internet-facing systems          │
   │                                    │
   │ High (1-7 days) :                  │
   │ - CVSS 7-8.9                       │
   │ - Critical systems                 │
   │                                    │
   │ Medium (7-30 days) :               │
   │ - CVSS 4-6.9                       │
   │ - Internal systems                 │
   │                                    │
   │ Low (30-90 days) :                 │
   │ - CVSS 0-3.9                       │
   │ - Non-critical                     │
   └────────────────────────────────────┘

5. VERIFICATION :
   - Confirm patch applied
   - Test functionality
   - Monitor for issues

6. DOCUMENTATION :
   - Update CMDB
   - Record in change log
   - Update runbooks

Rollback plan :

TOUJOURS avoir plan de rollback :
- Snapshot VM (avant patch)
- Backup configuration
- Tested rollback procedure
- Communication plan

Si problème :
1. Detect issue (monitoring)
2. Evaluate severity
3. Decision : Fix forward ou rollback
4. Execute rollback si nécessaire
5. Root cause analysis
6. Retry avec correction
```

**Automation**

```
Ansible patch management :

# playbook patch-servers.yml
---
- name: Patch Linux servers
  hosts: linux_servers
  become: yes

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Upgrade all packages
      apt:
        upgrade: dist
      when: ansible_os_family == "Debian"
      register: apt_upgrade

    - name: Check if reboot required
      stat:
        path: /var/run/reboot-required
      register: reboot_required

    - name: Reboot if needed
      reboot:
        msg: "Reboot initiated by Ansible"
        reboot_timeout: 600
      when: reboot_required.stat.exists

    - name: Send notification
      mail:
        host: smtp.company.com
        port: 587
        to: ops@company.com
        subject: "Patching completed: {{ inventory_hostname }}"
        body: "{{ apt_upgrade.stdout }}"

# Execute
ansible-playbook -i inventory patch-servers.yml

Windows (WSUS/SCCM) :

Group Policy :
Computer Configuration → Administrative Templates →
  Windows Components → Windows Update

- Configure Automatic Updates : Enabled
  - 4 - Auto download and schedule install
- Scheduled install day : Sunday
- Scheduled install time : 03:00

Maintenance Windows :
- Development : Anytime
- QA : Tuesday-Thursday
- Production : Saturday 02:00-06:00

Patch prioritization :
1. Critical security (emergency, anytime)
2. Security updates (monthly cycle)
3. Feature updates (quarterly, tested extensively)
```

## Sauvegarde et Disaster Recovery

### Stratégie 3-2-1

```
Règle 3-2-1 :

3 copies de données :
  - 1 production (original)
  - 2 backups

2 médias différents :
  - Disque local (rapide restore)
  - Tape/Cloud (offline, immutable)

1 copie off-site :
  - Protection sinistre physique
  - Fire, flood, theft

Implémentation :

Production :
  - Database server : 10.0.0.50
  - Data : 500 GB

Backup 1 - Local (daily) :
  - NAS : 10.0.0.100
  - Incremental daily
  - Full weekly
  - Retention : 30 days

Backup 2 - Cloud (weekly) :
  - AWS S3 / Glacier
  - Full backup weekly
  - Encrypted
  - Retention : 1 year
  - Immutable (WORM)

Backup 3 - Tape (monthly) :
  - LTO-8 tapes
  - Off-site vault
  - Retention : 7 years
  - Compliance (legal hold)

Test restore :
  Monthly : Restore fichier random
  Quarterly : Restore database complète
  Annually : Full disaster recovery drill
```

**Backup automatisé**

```bash
# Script backup (cron daily)
#!/bin/bash
# /usr/local/bin/backup.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup"
MYSQL_USER="backup"
MYSQL_PASS="secure_password"

# Database backup
mysqldump -u $MYSQL_USER -p$MYSQL_PASS --all-databases \
  | gzip > $BACKUP_DIR/mysql-$DATE.sql.gz

# Files backup
tar czf $BACKUP_DIR/files-$DATE.tar.gz /var/www /etc

# Sync to NAS
rsync -avz $BACKUP_DIR/ nas.company.com:/backups/server1/

# Sync to S3 (weekly)
if [ $(date +%u) -eq 7 ]; then
  aws s3 sync $BACKUP_DIR/ s3://company-backups/server1/ \
    --storage-class GLACIER
fi

# Cleanup old backups (>30 days)
find $BACKUP_DIR -type f -mtime +30 -delete

# Verify backup
if [ -f $BACKUP_DIR/mysql-$DATE.sql.gz ]; then
  echo "Backup successful: mysql-$DATE.sql.gz" | \
    mail -s "Backup OK" ops@company.com
else
  echo "Backup FAILED" | \
    mail -s "Backup FAILED" ops@company.com
fi

# Crontab
# 0 2 * * * /usr/local/bin/backup.sh

# Restore procedure (documented)
# 1. Stop application
# 2. Restore database:
#    gunzip < mysql-YYYYMMDD.sql.gz | mysql -u root -p
# 3. Restore files:
#    tar xzf files-YYYYMMDD.tar.gz -C /
# 4. Start application
# 5. Verify functionality
```

### Business Continuity Planning

```
RTO et RPO :

RPO (Recovery Point Objective) :
  Maximum data loss acceptable
  Example : 1 hour
  → Backup every 1 hour

RTO (Recovery Time Objective) :
  Maximum downtime acceptable
  Example : 4 hours
  → Must restore in 4 hours

Criticality tiers :

Tier 1 (Critical) :
  Systems : Payment processing, authentication
  RPO : 15 minutes
  RTO : 1 hour
  Solution : Active-active HA, real-time replication

Tier 2 (Important) :
  Systems : Email, file servers
  RPO : 1 hour
  RTO : 4 hours
  Solution : Active-passive HA, hourly backup

Tier 3 (Standard) :
  Systems : Intranet, documentation
  RPO : 24 hours
  RTO : 24 hours
  Solution : Daily backup, manual restore

Tier 4 (Low) :
  Systems : Archive, test environments
  RPO : 1 week
  RTO : 1 week
  Solution : Weekly backup

DR Site :

Hot site :
  - Fully equipped, ready
  - Active-active or quick failover
  - Expensive
  - RTO : Minutes to hours

Warm site :
  - Infrastructure ready, data replicated
  - Equipment ready but not running
  - Moderate cost
  - RTO : Hours to days

Cold site :
  - Space only, basic infrastructure
  - Equipment shipped when needed
  - Cheap
  - RTO : Days to weeks

Cloud DR :
  - AWS, Azure
  - Pay-as-you-go
  - Flexible
  - RTO : Configurable
```

## Formation et Sensibilisation

### Security Awareness Training

```
Programme formation sécurité :

Onboarding (nouveaux employés) :
  ┌────────────────────────────────────┐
  │ Jour 1 : Security Basics           │
  │ - Password policy                  │
  │ - MFA setup                        │
  │ - Acceptable use policy            │
  │ - Clean desk policy                │
  │ - Physical security                │
  │                                    │
  │ Semaine 1 : Phishing Training      │
  │ - Reconnaître phishing             │
  │ - Reporting suspicious emails      │
  │ - Simulated phishing test          │
  │                                    │
  │ Mois 1 : Data Protection           │
  │ - Classification données           │
  │ - Sharing policies                 │
  │ - GDPR basics                      │
  └────────────────────────────────────┘

Training continu (tous employés) :

Mensuel :
  - Security newsletter
  - Tips et actualités
  - Incident learnings (anonymisé)

Trimestriel :
  - Phishing simulation
  - Module e-learning (20 min)
  - Quiz (certification)

Annuel :
  - Security awareness day
  - Workshop
  - Policy review et signature

Topics couverts :

1. Password Security :
   ✓ Strong passwords
   ✓ Password manager
   ✓ Never share credentials
   ✓ MFA importance

2. Phishing & Social Engineering :
   ✓ Reconnaître tentatives
   ✓ Vérifier sender
   ✓ Suspicious links
   ✓ Urgency tactics
   ✓ Report à IT

3. Physical Security :
   ✓ Lock screen (Windows+L)
   ✓ Clean desk
   ✓ Visitors escort
   ✓ Tailgating prevention
   ✓ Secure disposal documents

4. Mobile Device Security :
   ✓ Screen lock
   ✓ Encryption
   ✓ Public Wi-Fi dangers
   ✓ Lost/stolen reporting

5. Data Protection :
   ✓ Classification (Public, Internal, Confidential, Secret)
   ✓ Sharing rules
   ✓ Encryption when needed
   ✓ GDPR compliance

6. Incident Reporting :
   ✓ What to report
   ✓ How to report (email, phone, portal)
   ✓ Qui contacter
   ✓ No blame culture

Gamification :

- Points pour completion modules
- Badges pour certifications
- Leaderboard (opt-in)
- Prizes pour top performers
- Team challenges

Phishing simulation :

Monthly simulated phishing :
  - Varied difficulty
  - Different techniques
  - Track click rate

Results :
  Employee clicks link :
    → Redirected à training page
    → Immediate micro-learning (5 min)
    → Manager notified (if repeated)

  Employee reports :
    → Positive feedback
    → Point awarded
    → Reinforcement

Metrics :
  - Click rate : <5% goal
  - Report rate : >20% goal
  - Trend over time
```

## Gestion des Incidents

### Incident Response Plan

```
Phases incident response (NIST) :

1. PREPARATION :
   ┌────────────────────────────────────┐
   │ - Incident response team           │
   │ - Roles et responsibilities        │
   │ - Tools et technologies            │
   │ - Playbooks (procedures)           │
   │ - Training                         │
   │ - Communication plan               │
   └────────────────────────────────────┘

2. DETECTION & ANALYSIS :
   ┌────────────────────────────────────┐
   │ Alert sources :                    │
   │ - SIEM                             │
   │ - IDS/IPS                          │
   │ - Antivirus                        │
   │ - User reports                     │
   │                                    │
   │ Initial analysis :                 │
   │ - Validate (true positive ?)       │
   │ - Classify severity                │
   │ - Scope (systems affected)         │
   │ - Timeline                         │
   └────────────────────────────────────┘

3. CONTAINMENT :
   ┌────────────────────────────────────┐
   │ Short-term :                       │
   │ - Isolate affected systems         │
   │ - Block malicious IPs/domains      │
   │ - Disable compromised accounts     │
   │                                    │
   │ Long-term :                        │
   │ - Patch vulnerabilities            │
   │ - Apply workarounds                │
   │ - Strengthen controls              │
   └────────────────────────────────────┘

4. ERADICATION :
   ┌────────────────────────────────────┐
   │ - Remove malware                   │
   │ - Delete backdoors                 │
   │ - Close vulnerabilities            │
   │ - Reset compromised credentials    │
   └────────────────────────────────────┘

5. RECOVERY :
   ┌────────────────────────────────────┐
   │ - Restore from clean backups       │
   │ - Rebuild systems                  │
   │ - Validate integrity               │
   │ - Return to production             │
   │ - Enhanced monitoring              │
   └────────────────────────────────────┘

6. POST-INCIDENT :
   ┌────────────────────────────────────┐
   │ - Lessons learned meeting          │
   │ - Timeline documentation           │
   │ - Root cause analysis              │
   │ - Update procedures                │
   │ - Improve defenses                 │
   └────────────────────────────────────┘

Severity classification :

Critical (P1) :
  - Active breach with data exfiltration
  - Ransomware encryption
  - Critical system down
  - Response : Immediate (24/7)

High (P2) :
  - Suspected breach
  - Malware detected
  - Important system compromised
  - Response : 1 hour

Medium (P3) :
  - Policy violation
  - Minor vulnerability
  - Non-critical system affected
  - Response : 4 hours

Low (P4) :
  - Informational
  - False positive
  - General inquiry
  - Response : Next business day

Incident response team :

Incident Commander :
  - Overall coordination
  - Decision making
  - Communication

Technical Lead :
  - Analysis
  - Containment actions
  - Recovery

Communication Lead :
  - Stakeholder updates
  - External communication
  - Documentation

Legal :
  - Regulatory obligations
  - Law enforcement liaison
  - Contract implications

HR :
  - Insider threat cases
  - Employee communication

Exemple playbook (Ransomware) :

1. DETECTION :
   Alert : Multiple files encrypted on file server

2. IMMEDIATE ACTIONS :
   - Isolate affected system (disconnect network)
   - Identify scope (other systems affected ?)
   - Preserve evidence (memory dump, disk image)
   - Document initial observations

3. ANALYSIS :
   - Identify ransomware variant
   - Determine entry point
   - Check for decryption tools
   - Assess backup viability

4. DECISION :
   Pay ransom ? → NO (policy)
   Restore from backup ? → YES

5. CONTAINMENT :
   - Isolate all potentially affected systems
   - Block C2 domains/IPs
   - Reset credentials (assume compromised)
   - Scan entire network

6. ERADICATION :
   - Remove malware from all systems
   - Patch vulnerability used
   - Close entry point

7. RECOVERY :
   - Restore files from backup
   - Verify integrity
   - Reconnect to network (monitored)

8. POST-INCIDENT :
   - Update defenses
   - Training (how entered)
   - Process improvements
```

## Conformité et Audits

### Standards et Frameworks

```
Security Frameworks :

NIST Cybersecurity Framework :
  ┌──────────┬────────────────────────────┐
  │ IDENTIFY │ - Asset management         │
  │          │ - Risk assessment          │
  ├──────────┼────────────────────────────┤
  │ PROTECT  │ - Access control           │
  │          │ - Awareness training       │
  │          │ - Data security            │
  ├──────────┼────────────────────────────┤
  │ DETECT   │ - Anomalies detection      │
  │          │ - Continuous monitoring    │
  ├──────────┼────────────────────────────┤
  │ RESPOND  │ - Response planning        │
  │          │ - Communications           │
  │          │ - Analysis                 │
  ├──────────┼────────────────────────────┤
  │ RECOVER  │ - Recovery planning        │
  │          │ - Improvements             │
  └──────────┴────────────────────────────┘

ISO 27001 :
  - Information Security Management System
  - Certification
  - Controls (Annex A)

CIS Controls :
  - 18 critical controls
  - Prioritized
  - Actionable

PCI-DSS :
  - Payment Card Industry
  - 12 requirements
  - Annual compliance
  - Protects cardholder data

HIPAA :
  - Healthcare
  - Patient data protection
  - Privacy and security rules

GDPR :
  - EU data protection
  - Privacy by design
  - Data subject rights
  - Breach notification (72h)

SOC 2 :
  - Service organizations
  - Trust principles
  - Type I (design) vs Type II (effectiveness)
```

**Audit preparation**

```
Pre-audit checklist :

Documentation :
✓ Security policies (current, signed)
✓ Procedures et runbooks
✓ Network diagrams (updated)
✓ Asset inventory
✓ Risk assessment
✓ Incident logs
✓ Change management records
✓ Access control lists
✓ Backup logs
✓ Patch management reports
✓ Training records
✓ Vendor contracts

Technical readiness :
✓ Vulnerability scan results (<30 days)
✓ Penetration test report (annual)
✓ Firewall rules review
✓ User access review
✓ Privileged accounts audit
✓ Encryption verification
✓ Logging verification
✓ Backup test results

Common findings :

Critical :
- Unpatched critical vulnerabilities
- Weak passwords (no policy enforcement)
- Missing encryption (data at rest)
- No MFA on admin accounts
- Insufficient logging

High :
- Excessive user privileges
- Shared accounts
- Outdated software
- Missing security awareness training
- Inadequate incident response plan

Medium :
- Documentation gaps
- Policy not updated
- Backup not tested
- Missing change approvals
- Weak password complexity

Low :
- Minor configuration issues
- Documentation formatting
- Process inefficiencies

Remediation :
1. Acknowledge findings
2. Prioritize (risk-based)
3. Create action plan (timelines)
4. Assign ownership
5. Track progress
6. Verify closure
7. Report to auditor
```

## Checklist de sécurisation complète

### Checklist Infrastructure

```
☐ RÉSEAU :
  ☐ Firewall configuré (default deny)
  ☐ DMZ pour services publics
  ☐ Segmentation VLAN
  ☐ IDS/IPS déployé
  ☐ Anti-spoofing activé (BCP 38)
  ☐ Rate limiting configuré
  ☐ Ports inutiles fermés
  ☐ SNMP v3 (si utilisé)
  ☐ Logs centralisés (SIEM)
  ☐ Network diagram à jour

☐ SERVEURS :
  ☐ OS à jour (patches)
  ☐ Hardening appliqué (CIS benchmark)
  ☐ Services inutiles désactivés
  ☐ Firewall hôte activé
  ☐ Antivirus/EDR installé
  ☐ Logging activé
  ☐ NTP configuré
  ☐ Backups fonctionnels (testés)
  ☐ Monitoring (CPU, RAM, Disk)
  ☐ Fail2ban ou équivalent

☐ APPLICATIONS :
  ☐ HTTPS obligatoire (TLS 1.2+)
  ☐ Certificats valides
  ☐ HSTS activé
  ☐ Security headers configurés
  ☐ Input validation
  ☐ Output encoding
  ☐ Parameterized queries (SQL)
  ☐ CSRF protection
  ☐ Session management sécurisé
  ☐ Error handling (pas d'info sensible)

☐ BASE DE DONNÉES :
  ☐ Least privilege (app user)
  ☐ Pas de compte par défaut
  ☐ Passwords forts
  ☐ Chiffrement au repos
  ☐ Chiffrement en transit
  ☐ Backup automatisé
  ☐ Audit logging
  ☐ Firewall (accès restreint)

☐ ENDPOINTS :
  ☐ Antivirus à jour
  ☐ Firewall activé
  ☐ Disk encryption (BitLocker/FileVault)
  ☐ Screen lock automatique
  ☐ Patches automatiques
  ☐ EDR installé
  ☐ Application whitelisting (si applicable)
  ☐ USB ports contrôlés

☐ CLOUD :
  ☐ MFA sur comptes admin
  ☐ Least privilege IAM
  ☐ Encryption enabled
  ☐ Logging activé (CloudTrail, etc.)
  ☐ Security groups restreints
  ☐ Buckets S3 privés
  ☐ Snapshots automatisés
  ☐ Vulnerability scanning
  ☐ Config compliance (AWS Config, etc.)

☐ IDENTITÉ & ACCÈS :
  ☐ MFA obligatoire (admin minimum)
  ☐ Password policy forte
  ☐ Comptes privilégiés séparés
  ☐ Access reviews trimestriels
  ☐ Joiners/Movers/Leavers process
  ☐ Password manager disponible
  ☐ SSO déployé (si applicable)
  ☐ Privileged Access Management

☐ MONITORING & LOGGING :
  ☐ SIEM configuré
  ☐ Alertes critiques définies
  ☐ Logs retenus (90+ jours)
  ☐ Log integrity protégé
  ☐ Dashboards créés
  ☐ On-call rotation définie
  ☐ Incident response plan
  ☐ Playbooks documentés

☐ BACKUPS :
  ☐ Stratégie 3-2-1
  ☐ Automatisés
  ☐ Chiffrés
  ☐ Testés mensuellement
  ☐ Off-site/cloud
  ☐ Immutable (WORM)
  ☐ Retention définie
  ☐ Restore procedure documentée

☐ PHYSIQUE :
  ☐ Datacenter sécurisé
  ☐ Racks verrouillés
  ☐ Caméras
  ☐ Access logs
  ☐ Badge requis
  ☐ Visitor policy
  ☐ Clean desk policy
  ☐ Secure disposal (shredding)

☐ HUMAIN :
  ☐ Security awareness training
  ☐ Phishing simulation
  ☐ Acceptable use policy
  ☐ Incident reporting process
  ☐ Background checks
  ☐ NDA signés
  ☐ Exit process

☐ GOUVERNANCE :
  ☐ Security policies documentées
  ☐ Risk assessment annuel
  ☐ Vendor management
  ☐ Compliance mapping
  ☐ Board reporting
  ☐ Budget sécurité
  ☐ Metrics & KPIs
  ☐ Continuous improvement
```

## Conclusion

La sécurisation d'un réseau TCP/IP est un processus continu qui nécessite vigilance, discipline et amélioration constante. Les bonnes pratiques présentées dans ce chapitre forment un socle solide pour protéger votre infrastructure.

**Principes clés à retenir** :

```
1. Defense in Depth :
   Multiples couches de sécurité
   Jamais une seule protection

2. Least Privilege :
   Minimum nécessaire
   Toujours

3. Zero Trust :
   Never trust, always verify
   Même pour l'interne

4. Assume Breach :
   Préparer pour la compromission
   Détection et réponse essentielles

5. Automation :
   Patching, monitoring, backup
   Réduire erreur humaine

6. Documentation :
   Procédures, configurations, incidents
   Knowledge sharing

7. Continuous Improvement :
   Apprendre des incidents
   Évolution constante

8. Human Factor :
   Formation utilisateurs
   Culture sécurité

9. Compliance :
   Réglementations
   Standards industrie

10. Testing :
    Pentest, DR drills, backup restore
    Valider efficacité
```

**Roadmap sécurité** :

```
Quick Wins (0-30 jours) :
✓ MFA sur comptes admin
✓ Patch systèmes critiques
✓ Firewall default deny
✓ Antivirus à jour
✓ Backup testé

Court terme (1-3 mois) :
✓ Security awareness training
✓ SIEM déployé
✓ Vulnerability scanning
✓ Incident response plan
✓ Access review

Moyen terme (3-6 mois) :
✓ IDS/IPS
✓ Zero Trust architecture
✓ Network segmentation
✓ PAM solution
✓ HTTPS everywhere

Long terme (6-12 mois) :
✓ Security orchestration
✓ Threat hunting
✓ Red team exercises
✓ Compliance certification
✓ Security metrics dashboard

Continu :
✓ Patch management
✓ User training
✓ Monitoring & alerting
✓ Incident response
✓ Risk assessment
✓ Improvement cycle
```

**Derniers conseils** :

```
"Perfect is the enemy of good"
→ Commencer quelque part
→ Améliorer progressivement
→ Ne pas attendre perfection

"Security is a journey, not a destination"
→ Jamais terminé
→ Évolution constante
→ Adaptation aux menaces

"People, Process, Technology"
→ Dans cet ordre
→ Technologie seule insuffisante
→ Culture sécurité essentielle

"You can't protect what you don't know about"
→ Inventory assets
→ Visibility totale
→ Monitoring continu

"Hope is not a strategy"
→ Planifier
→ Tester
→ Préparer l'incident
```

La sécurité réseau est un effort d'équipe impliquant IT, management, utilisateurs, et partenaires. En appliquant méthodiquement ces bonnes pratiques, vous construirez une défense robuste et résiliente contre les menaces modernes.

N'oubliez jamais : **La sécurité parfaite n'existe pas, mais la sécurité négligée est inexcusable.**

⏭️ [7. Analyse et dépannage réseau](/07-analyse-depannage/README.md)
