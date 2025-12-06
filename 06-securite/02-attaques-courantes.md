🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.2 Attaques courantes : spoofing, sniffing, man-in-the-middle

## Introduction

Les vulnérabilités inhérentes à TCP/IP que nous avons étudiées dans la section précédente ne sont pas de simples faiblesses théoriques. Elles sont **activement exploitées** par des attaquants du monde entier, chaque jour, à grande échelle. Cette section examine trois catégories d'attaques fondamentales qui exploitent ces vulnérabilités :

- **Spoofing** (usurpation) : Se faire passer pour quelqu'un d'autre
- **Sniffing** (interception) : Écouter les communications des autres
- **Man-in-the-Middle** (interposition) : S'insérer dans une communication

Ces attaques forment la base de nombreuses menaces plus complexes et comprendre leurs mécanismes est essentiel pour :
- Évaluer les risques de sécurité de vos systèmes
- Concevoir des architectures résilientes
- Diagnostiquer des incidents de sécurité
- Choisir les protections appropriées

## 1. Spoofing : l'art de l'usurpation d'identité

Le **spoofing** consiste à falsifier l'identité de l'expéditeur dans une communication réseau. Cette technique exploite directement l'absence d'authentification native dans TCP/IP.

### 1.1 IP Spoofing

#### Principe

L'**IP spoofing** consiste à créer des paquets IP avec une adresse source falsifiée. L'attaquant forge manuellement l'en-tête IP au lieu de laisser le système d'exploitation le remplir automatiquement.

#### Mécanisme technique

**Paquet IP normal** :
```
En-tête IP généré par le système :
+-------------------+
| Version: 4        |
| IHL: 5            |
| Total Length: 60  |
| Source IP: 192.168.1.100 ← IP réelle de la machine
| Dest IP: 10.0.0.5 |
| Protocole: TCP    |
| Checksum: auto    |
+-------------------+
```

**Paquet IP forgé (spoofé)** :
```python
# Exemple conceptuel avec une bibliothèque de manipulation de paquets
from scapy.all import *

# Création d'un paquet avec IP source falsifiée
paquet_forge = IP(src="8.8.8.8",      # IP de Google DNS (falsifiée)
                  dst="192.168.1.50")  # Vraie cible
paquet_forge = paquet_forge / ICMP()

# Envoi du paquet
send(paquet_forge)

# La cible voit un ping venant de 8.8.8.8
# mais il vient en réalité de l'attaquant
```

#### Limitations pratiques

**1. Absence de réponse**
```
Attaquant (IP réelle: 192.168.1.100)
    ↓ envoie paquet avec src=8.8.8.8
    ↓
Cible (10.0.0.5)
    ↓ répond à l'adresse source
    ↓
8.8.8.8 (reçoit la réponse, pas l'attaquant)

L'attaquant ne reçoit jamais la réponse !
```

Cette limitation rend IP spoofing inutile pour les attaques interactives, mais **très efficace** pour :
- Les attaques par déni de service
- Les attaques par amplification/réflexion
- Le contournement de listes blanches (ACL)

**2. Filtrage anti-spoofing (BCP 38/RFC 2827)**

De nombreux fournisseurs d'accès Internet filtrent les paquets dont l'adresse source ne correspond pas à leur plage d'adresses :

```
Client du FAI : 203.0.113.50
Réseau du FAI : 203.0.113.0/24

Paquet envoyé avec src=8.8.8.8 :
    FAI vérifie : 8.8.8.8 ∉ 203.0.113.0/24
    → Paquet rejeté (ingress filtering)

Paquet envoyé avec src=203.0.113.100 :
    FAI vérifie : 203.0.113.100 ∈ 203.0.113.0/24
    → Paquet accepté (spoofing possible dans la plage)
```

**Malheureusement**, ce filtrage n'est pas universel. Selon des études récentes :
- ~30% des réseaux autonomes (AS) permettent encore le spoofing complet
- ~50% permettent le spoofing partiel

#### Cas d'usage malveillant 1 : Smurf Attack

**Principe** : Amplification via ICMP broadcast

```
Réseau cible : 10.0.0.0/24 (254 hôtes)
Victime : 192.168.1.100

Étape 1 - Attaquant forge un ping :
    src: 192.168.1.100 (victime)
    dst: 10.0.0.255 (broadcast)
    type: ICMP Echo Request

Étape 2 - Le broadcast atteint 254 machines :
    Toutes répondent à 192.168.1.100

Étape 3 - Amplification :
    1 paquet envoyé → 254 paquets reçus
    Facteur d'amplification : 254x

Résultat :
    Victime submergée par les réponses ICMP
    Bande passante saturée
    Services inaccessibles
```

**Évolution** : Cette attaque est largement atténuée aujourd'hui car :
- Les routeurs modernes ne forwarding plus les broadcasts dirigés par défaut
- Les pare-feu filtrent les paquets ICMP excessifs

#### Cas d'usage malveillant 2 : Contournement d'ACL

**Scénario** : Un serveur SSH autorise uniquement certaines IPs

```
Configuration du pare-feu :
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/8 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

Seules les IPs 10.0.0.0/8 peuvent accéder au SSH
```

**Attaque** :
```
Attaquant (IP réelle : 203.0.113.50) forge :
    src: 10.0.0.100 (dans la plage autorisée)
    dst: serveur:22
    SYN

Le pare-feu accepte (vérification basée sur l'IP source)

Problème pour l'attaquant :
    Le SYN-ACK va à 10.0.0.100, pas à lui
    → Impossible d'établir la connexion TCP

Mais :
    Si 10.0.0.100 est down ou filtré
    + Attaquant peut prédire les numéros de séquence
    → Blind TCP spoofing possible (attaque Mitnick)
```

#### Cas d'usage malveillant 3 : DNS Amplification

L'une des attaques DDoS les plus répandues en 2024 :

```
Attaquant contrôle un botnet de 10 000 machines

Chaque machine envoie :
    src: IP_victime (falsifiée)
    dst: serveur_DNS_ouvert:53
    query: ANY example.com (demande tous les enregistrements)
    taille requête: 60 octets

Serveur DNS répond :
    src: serveur_DNS:53
    dst: IP_victime
    response: tous les enregistrements DNS
    taille réponse: 3000 octets

Calcul :
    10 000 machines × 60 octets = 600 Ko/s envoyés
    10 000 réponses × 3000 octets = 30 Mo/s reçus par la victime
    Facteur d'amplification : 50x

Avec un botnet plus large :
    100 000 machines → 3 Go/s vers la victime
    Suffisant pour saturer la plupart des connexions Internet
```

**Atténuations** :
- Filtrage BCP 38 chez les FAI (empêche le spoofing)
- Limitation du taux de réponses (rate limiting) sur les DNS
- Response Rate Limiting (RRL) dans BIND, Unbound
- Fermeture des résolveurs DNS ouverts

### 1.2 ARP Spoofing

L'**ARP spoofing** (ou ARP poisoning) est l'une des attaques les plus efficaces sur un réseau local car ARP n'a **aucun mécanisme d'authentification**.

#### Rappel du fonctionnement normal d'ARP

```
Machine A (192.168.1.10, MAC: AA:AA:AA:AA:AA:AA)
veut communiquer avec 192.168.1.1 (routeur)

Étape 1 - Requête ARP (broadcast) :
    "Qui a l'IP 192.168.1.1 ? Répondez à AA:AA:AA:AA:AA:AA"

Étape 2 - Réponse du routeur (unicast) :
    "C'est moi ! Mon MAC est RR:RR:RR:RR:RR:RR"

Étape 3 - Mise en cache :
    Table ARP de A : 192.168.1.1 → RR:RR:RR:RR:RR:RR
```

#### Mécanisme de l'attaque

**Vulnérabilité** : ARP accepte les réponses **non sollicitées** (Gratuitous ARP)

```
Réseau normal :
Client    : 192.168.1.10 (MAC: CC:CC:CC:CC:CC:CC)
Routeur   : 192.168.1.1  (MAC: RR:RR:RR:RR:RR:RR)
Attaquant : 192.168.1.50 (MAC: XX:XX:XX:XX:XX:XX)

Attaque - L'attaquant envoie un ARP Reply gratuit :
    ARP Reply (broadcast ou unicast au client)
    "192.168.1.1 est maintenant à XX:XX:XX:XX:XX:XX"

Table ARP du client empoisonnée :
    192.168.1.1 → XX:XX:XX:XX:XX:XX (au lieu de RR:RR)

Résultat :
    Tout le trafic du client vers Internet passe par l'attaquant
```

#### Implémentation avec arpspoof

```bash
# Outil classique : arpspoof (dsniff suite)

# Empoisonner le client (lui dire que nous sommes le routeur)
arpspoof -i eth0 -t 192.168.1.10 192.168.1.1

# Empoisonner le routeur (lui dire que nous sommes le client)
arpspoof -i eth0 -t 192.168.1.1 192.168.1.10

# Activer le forwarding IP pour relayer le trafic
echo 1 > /proc/sys/net/ipv4/ip_forward

# Maintenant tout le trafic entre client et routeur passe par l'attaquant
# qui peut le lire, modifier, ou bloquer
```

#### Variante : ARP Spoofing bidirectionnel

Pour un Man-in-the-Middle complet :

```
Configuration de l'attaque :

1. Empoisonnement du client :
    ARP Reply : "Le routeur (192.168.1.1) est à XX:XX (attaquant)"

2. Empoisonnement du routeur :
    ARP Reply : "Le client (192.168.1.10) est à XX:XX (attaquant)"

Flux de trafic :
    Client → pense envoyer au routeur → Attaquant
    Attaquant → forward au routeur
    Routeur → pense répondre au client → Attaquant
    Attaquant → forward au client

Ni le client ni le routeur ne détectent l'interposition
Communication apparemment normale
```

#### Scénario d'attaque concret : Vol de session

```
Victime se connecte à http://intranet.company.com

1. Attaquant effectue ARP spoofing
2. Tout le trafic HTTP transite par l'attaquant
3. Attaquant capture :
    GET /login HTTP/1.1
    Host: intranet.company.com
    Cookie: session=abc123xyz789

4. Attaquant rejoue la session :
    - Utilise le même cookie
    - Accède au compte de la victime
    - Exfiltre des données
```

#### Détection et protection contre ARP Spoofing

**Détection** :

1. **Surveillance des changements ARP** :
```bash
# arpwatch - détecte les changements d'association IP-MAC
# Alerte si 192.168.1.1 change soudainement de MAC

# Log typique :
# hostname: client.local
# ip address: 192.168.1.1
# ethernet address: xx:xx:xx:xx:xx:xx
# Previous ethernet address: rr:rr:rr:rr:rr:rr
# ▲ SUSPECT : changement de MAC pour la même IP
```

2. **Détection de multiples MACs pour une IP** :
```bash
# Analyser le trafic et détecter :
IP 192.168.1.1 annoncée par :
    - RR:RR:RR:RR:RR:RR (légitime)
    - XX:XX:XX:XX:XX:XX (attaquant)
→ Conflit détecté
```

**Protection** :

1. **Tables ARP statiques** (peu pratique à grande échelle) :
```bash
# Sur Linux
arp -s 192.168.1.1 RR:RR:RR:RR:RR:RR

# Entrée permanente, immune au spoofing
# Mais difficile à maintenir dans un réseau dynamique
```

2. **Dynamic ARP Inspection (DAI)** sur les switches :
```
Configuration switch Cisco :

switch(config)# ip arp inspection vlan 10
switch(config)# interface GigabitEthernet0/1
switch(config-if)# ip arp inspection trust

Le switch valide chaque paquet ARP :
- Vérifie la cohérence IP-MAC avec la table DHCP snooping
- Rejette les paquets ARP incohérents
- Seuls les ports de confiance peuvent envoyer des ARP Reply
```

3. **Segmentation réseau** :
```
Réseau plat (vulnérable) :
Tous les utilisateurs sur le même VLAN
→ Tout le monde peut faire du ARP spoofing sur tout le monde

Réseau segmenté (plus sûr) :
VLAN par département ou par utilisateur
→ Surface d'attaque limitée
```

### 1.3 DNS Spoofing

Le **DNS spoofing** exploite l'absence d'authentification dans le protocole DNS classique pour rediriger les utilisateurs vers de faux sites.

#### Variante 1 : DNS Spoofing sur réseau local

**Scénario** : Attaquant sur le même réseau Wi-Fi que la victime

```
Victime envoie une requête DNS :
    src: 192.168.1.10:45678
    dst: 8.8.8.8:53
    query: "Quelle est l'IP de www.banque.fr ?"
    transaction ID: 0x1234

Attaquant (sniffing le réseau) voit la requête et répond immédiatement :
    src: 8.8.8.8:53 (falsifié)
    dst: 192.168.1.10:45678
    transaction ID: 0x1234 (copié de la requête)
    response: "www.banque.fr = 6.6.6.6" (IP de l'attaquant)

Si la fausse réponse arrive avant la vraie :
    → Victime enregistre 6.6.6.6
    → Victime contacte le site malveillant
    → Phishing, vol d'identifiants
```

**Implémentation** :

```bash
# Outil : dnsspoof (dsniff suite)

# Fichier de configuration /etc/dnsspoof.conf :
www.banque.fr    6.6.6.6
*.banque.fr      6.6.6.6

# Lancement :
dnsspoof -i wlan0 -f /etc/dnsspoof.conf

# Toutes les requêtes DNS sniffées sont répondues
# avec les fausses adresses
```

**Prérequis** :
- Être sur le même réseau local (Wi-Fi public, etc.)
- Pouvoir sniffer le trafic (réseau partagé)
- Répondre plus vite que le vrai serveur DNS

#### Variante 2 : DNS Cache Poisoning (Kaminsky Attack)

Attaque plus sophistiquée ciblant les résolveurs DNS pour empoisonner leur cache.

**Principe** :

```
1. Attaquant génère des requêtes vers le résolveur cible :
    "Quelle est l'IP de random1.example.com ?"
    "Quelle est l'IP de random2.example.com ?"
    ...

2. Pour chaque requête, le résolveur interroge le serveur autoritatif

3. Attaquant inonde le résolveur de fausses réponses :
    Transaction ID: 0x0001, www.example.com = 6.6.6.6
    Transaction ID: 0x0002, www.example.com = 6.6.6.6
    Transaction ID: 0x0003, www.example.com = 6.6.6.6
    ... (des milliers de réponses)

4. Si une fausse réponse a le bon Transaction ID avant la vraie :
    → Cache empoisonné
    → TTL peut être de plusieurs heures
    → Tous les clients utilisant ce résolveur sont redirigés
```

**Détails de la fenêtre de vulnérabilité** :

```
Transaction ID : 16 bits = 65 536 possibilités
Port source : si prévisible, réduit l'espace à deviner

Ancienne implémentation (port source fixe) :
    Port source : toujours 53
    → Seulement Transaction ID à deviner
    → 65 536 tentatives maximum

Implémentation moderne (port source aléatoire) :
    Port source : aléatoire entre 1024-65535 (~ 64 000 possibilités)
    → Transaction ID : 65 536 possibilités
    → Total : ~ 4 milliards de combinaisons
    → Beaucoup plus difficile, mais pas impossible
```

**Kaminsky a découvert** :

Au lieu de deviner pour une seule requête, on peut :
1. Forcer le résolveur à faire de multiples requêtes (random1, random2, etc.)
2. Chaque requête = nouvelle fenêtre d'opportunité
3. Dans la réponse forgée, inclure des enregistrements additionnels :

```
Réponse falsifiée :
Query: random123.example.com
Answer: random123.example.com = 1.2.3.4
Additional section:
    www.example.com = 6.6.6.6 ← Injection dans le cache !

Si accepté, le cache est empoisonné pour www.example.com
même si la requête portait sur random123
```

**Protection** :

1. **DNSSEC** (DNS Security Extensions) :
```
DNS normal :
    Requête → Réponse (non signée, non vérifiable)

DNSSEC :
    Requête → Réponse signée cryptographiquement
    Le client vérifie la signature avec la clé publique
    Toute modification invalide la signature
```

2. **Randomisation** :
- Port source aléatoire
- Transaction ID vraiment aléatoire (pas séquentiel)
- Capitalization de requête (0x20 encoding)

3. **DNS over HTTPS (DoH) / DNS over TLS (DoT)** :
```
DNS classique :
    Client → Requête en clair → Résolveur

DoH/DoT :
    Client → Requête chiffrée (TLS) → Résolveur
    Impossible de sniffer ou de spoofing
```

### 1.4 Email Spoofing

L'**email spoofing** exploite l'absence de vérification d'identité dans SMTP.

#### Mécanisme

SMTP (Simple Mail Transfer Protocol) fait confiance à ce que déclare l'expéditeur :

```
Connexion SMTP normale :

telnet mail.example.com 25
220 mail.example.com ESMTP

HELO attacker.com
250 Hello

MAIL FROM: <ceo@company.com>  ← N'importe quelle adresse
250 OK

RCPT TO: <employee@company.com>
250 OK

DATA
354 Start mail input

From: CEO <ceo@company.com>
To: employee@company.com
Subject: Urgent - Wire Transfer

Please transfer $50,000 to account...

.
250 Message accepted

QUIT
```

**Le serveur SMTP n'a jamais vérifié** :
- Que l'expéditeur possède réellement ceo@company.com
- Que la connexion vient d'un serveur autorisé
- Que le message est légitime

#### Exemple d'attaque BEC (Business Email Compromise)

```
Scénario :
1. Attaquant se renseigne sur l'entreprise (LinkedIn, etc.)
2. Identifie le CEO et le CFO
3. Envoie un email falsifié :

From: john.doe@company.com (CEO)
To: jane.smith@company.com (CFO)
Subject: RE: Urgent Acquisition

Jane,

I need you to wire $500,000 to our lawyer's escrow account
for the acquisition we discussed. This is time-sensitive.

Account details:
Bank: ...
Account: ...

Please confirm once done.

John

4. Le CFO voit un email apparemment du CEO
5. Sans vérification supplémentaire, effectue le virement
6. L'argent va au compte de l'attaquant
```

**Impact réel** : Le FBI estime que les attaques BEC ont causé plus de **50 milliards de dollars** de pertes entre 2013 et 2023.

#### Protections contre l'email spoofing

**1. SPF (Sender Policy Framework)** :

```
Enregistrement DNS de company.com :
company.com. IN TXT "v=spf1 ip4:203.0.113.0/24 -all"

Signification :
- Seuls les serveurs avec IP 203.0.113.0/24 peuvent envoyer pour @company.com
- Tout autre serveur doit être rejeté (-all)

Vérification par le serveur récepteur :
1. Email reçu de john.doe@company.com
2. Serveur vérifie l'IP source : 198.51.100.50
3. Requête DNS : SPF de company.com ?
4. Réponse : Seulement 203.0.113.0/24 autorisé
5. 198.51.100.50 ∉ 203.0.113.0/24
6. → Email rejeté ou marqué comme spam
```

**2. DKIM (DomainKeys Identified Mail)** :

```
Serveur d'envoi signe cryptographiquement l'email :

DKIM-Signature: v=1; a=rsa-sha256; d=company.com; s=selector1;
    h=from:to:subject:date;
    bh=hash_du_corps;
    b=signature_cryptographique

Serveur récepteur :
1. Récupère la clé publique via DNS : selector1._domainkey.company.com
2. Vérifie la signature
3. Si signature invalide → email modifié ou falsifié
```

**3. DMARC (Domain-based Message Authentication, Reporting & Conformance)** :

```
Enregistrement DNS :
_dmarc.company.com. IN TXT "v=DMARC1; p=reject; rua=mailto:dmarc@company.com"

Politique :
- p=reject : Rejeter les emails qui échouent SPF et DKIM
- rua=... : Envoyer des rapports d'authentification

Résultat :
- Les emails spoofés sont rejetés automatiquement
- L'entreprise reçoit des rapports sur les tentatives d'usurpation
```

## 2. Sniffing : l'interception passive

Le **sniffing** (reniflage) consiste à capturer et analyser le trafic réseau. Sur un réseau partagé (Wi-Fi, hub Ethernet), tout le trafic est visible par tous les participants.

### 2.1 Sniffing passif

#### Principe

En mode normal, une carte réseau ne traite que les paquets qui lui sont destinés :
- Adresse MAC de destination = sa propre MAC
- Broadcast (FF:FF:FF:FF:FF:FF)
- Multicast (si inscrit)

En **mode promiscuous**, la carte réseau accepte **tous** les paquets sur le réseau.

```
Réseau Wi-Fi ouvert (café, aéroport, etc.) :

Client A ← → Point d'accès Wi-Fi ← → Client B
   ↑                                      ↑
   |                                      |
Attaquant en mode monitor/promiscuous
   |
Voit tout le trafic entre A, B et le point d'accès
```

#### Outils de sniffing

**Wireshark** (GUI, analyse approfondie) :
```bash
# Lancer Wireshark
wireshark

# Interface en mode promiscuous
# Capture tout le trafic visible

# Filtres d'affichage :
http.request.method == "POST"  # Voir les soumissions de formulaires
http.cookie                     # Voir les cookies
tcp.stream eq 0                 # Suivre une session TCP complète
```

**tcpdump** (ligne de commande) :
```bash
# Capturer tout le trafic HTTP
tcpdump -i wlan0 'tcp port 80' -w capture.pcap

# Capturer les credentials FTP
tcpdump -i eth0 'tcp port 21' -A

# Capturer les mots de passe Telnet
tcpdump -i eth0 'tcp port 23' -A
```

**Ettercap** (spécialisé dans le MITM) :
```bash
# Sniffing avec ARP poisoning intégré
ettercap -T -M arp:remote /192.168.1.1/ /192.168.1.10/

# Capture le trafic entre le routeur et la cible
# Peut extraire automatiquement credentials, images, etc.
```

#### Exemple concret : Capture de credentials HTTP

**Scénario** : Café Wi-Fi public

```
Victime se connecte à http://forum.example.com

Requête HTTP capturée :
POST /login HTTP/1.1
Host: forum.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 35

username=alice&password=secret123

L'attaquant voit en clair :
- Username : alice
- Password : secret123
- Site : forum.example.com

L'attaquant peut :
1. Se connecter au compte d'Alice
2. Tester le password sur d'autres sites (réutilisation fréquente)
3. Accéder aux données personnelles d'Alice
```

#### Capture de session (Cookie Hijacking)

```
Requête HTTP avec cookie :
GET /account HTTP/1.1
Host: example.com
Cookie: session=abc123xyz789; user_id=12345

L'attaquant :
1. Capture le cookie de session
2. Rejoue la requête avec le même cookie :

curl -H "Cookie: session=abc123xyz789" http://example.com/account

3. Accède au compte sans connaître le mot de passe
4. Le serveur pense que c'est l'utilisateur légitime
```

### 2.2 Sniffing actif

Lorsque le réseau est commuté (switches), le sniffing passif ne fonctionne pas car le switch envoie le trafic uniquement vers le port de destination.

#### MAC Flooding

**Objectif** : Transformer le switch en hub

```
Fonctionnement normal du switch :
+------------------------+
| Table MAC              |
|------------------------|
| MAC AA:AA → Port 1     |
| MAC BB:BB → Port 2     |
| MAC CC:CC → Port 3     |
+------------------------+

Paquet pour CC:CC → Switch envoie uniquement sur Port 3

Attaque MAC Flooding :
1. Attaquant génère des milliers de trames avec MACs sources aléatoires
2. Table MAC du switch saturée (capacité limitée : 8000-16000 entrées)
3. Switch passe en mode "fail-open" : broadcast tout sur tous les ports
4. Switch devient un hub
5. Attaquant peut sniffer tout le trafic
```

**Implémentation** :
```bash
# macof (dsniff suite)
macof -i eth0

# Génère des paquets avec MACs sources aléatoires
# Sature la table MAC en quelques secondes
```

**Protection** :
```
# Port security sur switch Cisco
switchport port-security
switchport port-security maximum 2
switchport port-security violation shutdown

# Limite à 2 MACs par port
# Shutdown du port en cas de dépassement
```

#### ARP Spoofing pour sniffing

Nous avons déjà vu ARP spoofing pour le spoofing, mais c'est aussi une technique de sniffing :

```
1. Attaquant empoisonne les caches ARP
2. Tout le trafic transite par l'attaquant
3. Attaquant forward les paquets (invisible)
4. Attaquant log/analyse tout le trafic en transit
```

### 2.3 Sniffing sur Wi-Fi

#### Wi-Fi ouvert (sans chiffrement)

```
Aucune protection :
- Tout le trafic en clair au niveau radio
- N'importe qui dans la portée peut capturer
- Aucune authentification nécessaire
```

**Capture avec airodump-ng** :
```bash
# Mettre la carte en mode monitor
airmon-ng start wlan0

# Capturer tout le trafic Wi-Fi
airodump-ng wlan0mon

# Capturer un réseau spécifique
airodump-ng --bssid AA:BB:CC:DD:EE:FF -c 6 -w capture wlan0mon

# Analyse ultérieure avec Wireshark
wireshark capture-01.cap
```

#### Wi-Fi avec WPA2/WPA3

Le chiffrement protège le trafic au niveau liaison, **mais** :

1. **Attaque sur le handshake 4-way** :
```
Capture du 4-way handshake WPA2 :
1. Forcer la déconnexion d'un client (deauth attack)
2. Capturer sa reconnexion (handshake)
3. Bruteforce hors ligne du handshake avec wordlist
4. Si mot de passe faible → Découverte de la clé PSK
5. Déchiffrement de tout le trafic capturé
```

2. **Après connexion au réseau** :
```
Une fois connecté au Wi-Fi (même WPA3) :
- Le trafic entre clients du même réseau peut être sniffé
- ARP spoofing fonctionne
- HTTP reste en clair (WPA protège seulement Wi-Fi ↔ AP)
```

### 2.4 Protection contre le sniffing

**1. Chiffrement de bout en bout** :
```
HTTPS : Chiffre HTTP
SSH : Chiffre Telnet/FTP
VPN : Chiffre tout le trafic IP

Même si sniffé, le contenu est illisible
```

**2. Éviter les réseaux non sécurisés** :
```
Wi-Fi public sans chiffrement : DANGER
Wi-Fi public avec WPA2 : Méfiance (opérateur malveillant possible)
VPN sur Wi-Fi public : OK
```

**3. Détection d'intrusion** :
```
IDS (Intrusion Detection System) :
- Détecte le mode promiscuous
- Alerte sur ARP spoofing
- Identifie les patterns de scan
```

## 3. Man-in-the-Middle (MITM) : l'interposition active

Le **Man-in-the-Middle** combine spoofing et sniffing pour s'insérer dans une communication, permettant non seulement l'écoute mais aussi la **modification** des données en transit.

### 3.1 MITM au niveau liaison (ARP MITM)

Nous l'avons déjà couvert : ARP spoofing bidirectionnel crée un MITM parfait sur un réseau local.

```
Communication normale :
Client ←→ Routeur ←→ Internet

MITM via ARP spoofing :
Client ←→ Attaquant ←→ Routeur ←→ Internet
         (transparent)

Capacités de l'attaquant :
- Lire tout le trafic
- Modifier les requêtes
- Modifier les réponses
- Injecter du contenu
- Bloquer sélectivement
```

### 3.2 MITM sur HTTPS (SSL Stripping)

#### Principe du SSL Stripping

L'attaquant intercepte et "dégrade" les connexions HTTPS en HTTP.

**Scénario normal** :
```
1. Utilisateur tape : example.com (sans https://)
2. Navigateur fait une requête HTTP
3. Serveur répond : 301 Redirect → https://example.com
4. Navigateur établit une connexion HTTPS
```

**Avec SSL Stripping** :
```
1. Utilisateur tape : example.com
2. Requête HTTP interceptée par l'attaquant
3. Attaquant établit une connexion HTTPS avec le serveur
4. Attaquant renvoie au client une page HTTP (pas de redirect)
5. Client ←(HTTP)→ Attaquant ←(HTTPS)→ Serveur

Résultat :
- Client pense être en HTTP normal (pas de cadenas)
- Attaquant voit tout en clair
- Serveur pense communiquer normalement en HTTPS
```

**Implémentation avec sslstrip** :
```bash
# Prérequis : ARP spoofing actif

# Rediriger le trafic HTTP vers sslstrip
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# Lancer sslstrip
sslstrip -l 8080

# Tous les liens HTTPS sont réécrits en HTTP
# Les soumissions de formulaires HTTPS deviennent HTTP
# L'attaquant capture tout
```

**Protection : HSTS (HTTP Strict Transport Security)** :
```
Header HTTP :
Strict-Transport-Security: max-age=31536000; includeSubDomains

Effet :
- Le navigateur DOIT utiliser HTTPS
- Refuse les connexions HTTP
- Même si l'attaquant tente de downgrade
- Valable pendant 1 an (max-age)

Limite :
- Fonctionne seulement après la première visite en HTTPS
- HSTS Preload List résout ce problème (liste built-in des navigateurs)
```

### 3.3 MITM sur TLS (certificat frauduleux)

#### Scénario : Proxy d'entreprise ou attaquant sophistiqué

```
HTTPS normal :
Client ←(TLS: cert serveur)→ Serveur

MITM avec certificat frauduleux :
Client ←(TLS: cert attaquant)→ Attaquant ←(TLS: cert serveur)→ Serveur

Problème :
Le navigateur alerte : "Certificat non valide !"

Solutions de l'attaquant :
1. Certificat signé par une CA compromise ou malveillante
2. Installation préalable d'une CA root sur le poste client
3. Utilisateur clique sur "Accepter le risque" (social engineering)
```

**Cas légitime : proxy TLS d'entreprise** :
```
Entreprise installe sa propre CA root sur tous les postes
Proxy d'entreprise :
- Intercepte tout le trafic HTTPS
- Déchiffre avec son propre certificat
- Inspecte le contenu (DLP, filtrage)
- Rechiffre vers le serveur final

Légal pour l'entreprise sur ses machines
Mais crée une vulnérabilité si le proxy est compromis
```

#### Certificate Pinning (protection avancée)

```
Application mobile ou desktop :
- Embarque le certificat (ou sa clé publique) du serveur
- Refuse toute connexion avec un autre certificat
- Même si signé par une CA légitime

Code exemple (pseudocode) :
if (server_cert.public_key != PINNED_PUBLIC_KEY):
    abort_connection()
    throw SecurityException()

Résultat :
- MITM impossible, même avec certificat valide
- Mais complique les mises à jour de certificats
```

### 3.4 MITM sur DNS

Nous l'avons vu dans DNS spoofing, mais dans un contexte MITM :

```
1. Attaquant redirige vers son propre serveur DNS (DHCP, ARP, etc.)
2. Toutes les résolutions passent par lui
3. Il peut :
    - Rediriger vers des faux sites (phishing)
    - Logger tous les sites visités
    - Bloquer l'accès à certains domaines
    - Injecter des publicités (DNS hijacking commercial)
```

**Exemple : Rogue DHCP Server** :
```
Réseau normal :
Client → DHCP Discover
Serveur légitime → DHCP Offer (DNS: 192.168.1.1)

Avec Rogue DHCP :
Client → DHCP Discover
Attaquant → DHCP Offer (DNS: 192.168.1.50 - attaquant)
    (réponse plus rapide)

Client configure DNS = 192.168.1.50
Toutes les requêtes DNS vont à l'attaquant
```

### 3.5 Détection et protection contre MITM

#### Détection

**1. Vérification des certificats** :
```bash
# Vérifier le certificat d'un site
openssl s_client -connect example.com:443 -showcerts

# Comparer l'empreinte avec une source fiable
# Si différente → MITM possible
```

**2. Détection ARP** :
```bash
# Observer les réponses ARP multiples
arpwatch

# Vérifier incohérences
arp -a
# Si une IP a plusieurs MACs → Suspect
```

**3. Monitoring réseau** :
```
IDS/IPS détecte :
- ARP spoofing (réponses ARP multiples)
- Redirections suspectes
- Trafic anormal (volumes, patterns)
```

#### Protection

**1. Utiliser systématiquement HTTPS** :
```
- Toujours vérifier le cadenas
- Activer "HTTPS-Only Mode" dans le navigateur
- Utiliser l'extension HTTPS Everywhere
```

**2. VPN** :
```
Client →(VPN chiffré)→ Serveur VPN →(Internet)

Même si MITM sur le réseau local :
- Le trafic est chiffré de bout en bout
- Impossible de voir ou modifier
```

**3. Certificate Pinning** (applications critiques)

**4. Segmentation réseau** :
```
- VLANs par niveau de confiance
- Isolation des invités
- Micro-segmentation
```

**5. 802.1X (authentification réseau)** :
```
Avant d'accéder au réseau :
- Authentification obligatoire
- Certificat ou credentials
- Empêche les appareils non autorisés
```

## Tableau comparatif des attaques

| Attaque | Couche | Prérequis | Détection | Protection principale |
|---------|--------|-----------|-----------|----------------------|
| **IP Spoofing** | Internet | Privilèges root | BCP 38 filtering | Ingress/egress filtering |
| **ARP Spoofing** | Liaison | Réseau local | Arpwatch, DAI | DAI, segmentation |
| **DNS Spoofing** | Application | Réseau local ou résolveur vulnérable | DNSSEC validation | DNSSEC, DoH/DoT |
| **Email Spoofing** | Application | Aucun | SPF/DKIM check | SPF, DKIM, DMARC |
| **Sniffing passif** | Toutes | Réseau partagé | Mode promiscuous detect | Chiffrement (TLS, VPN) |
| **Sniffing actif** | Liaison | Réseau local | Port security | Switches sécurisés, 802.1X |
| **MITM ARP** | Liaison | Réseau local | Arpwatch | DAI, VPN |
| **SSL Stripping** | Application | MITM position | HSTS checking | HSTS, HTTPS-only |
| **MITM TLS** | Application | CA compromise ou user accept | Cert pinning | Pinning, vigilance |

## Cascades d'attaques

Dans la réalité, les attaques sont souvent **combinées** :

```
Exemple d'attaque sophistiquée :

1. ARP Spoofing
    ↓ (permet)
2. MITM position établie
    ↓ (permet)
3. DNS Spoofing sur requêtes sensibles
    ↓ (permet)
4. Redirection vers site de phishing
    ↓ (permet)
5. SSL Stripping sur certains liens
    ↓ (permet)
6. Capture de credentials
    ↓ (permet)
7. Session hijacking sur comptes légitimes
    ↓ (permet)
8. Exfiltration de données sensibles

Une seule vulnérabilité initiale (ARP) → cascade complète
```

## Conclusion

Les attaques de spoofing, sniffing et man-in-the-middle ne sont pas des menaces théoriques. Elles sont :

**Omniprésentes** :
- Sur les Wi-Fi publics
- Sur les réseaux d'entreprise mal sécurisés
- Sur Internet (attaques BGP, DNS)

**Accessibles** :
- Outils gratuits et faciles d'utilisation
- Tutoriels abondants
- Barrière technique basse

**Dévastatrices** :
- Vol d'identifiants et de données
- Fraude financière (BEC)
- Espionnage industriel
- Sabotage

**Protections essentielles** :

1. **Chiffrement systématique** : HTTPS, SSH, VPN
2. **Authentification forte** : DNSSEC, SPF/DKIM/DMARC, certificats
3. **Vigilance utilisateur** : Vérifier les certificats, méfiance sur Wi-Fi public
4. **Sécurisation réseau** : DAI, port security, segmentation
5. **Monitoring** : IDS/IPS, logs, anomalies

La meilleure défense est une **approche en couches** :
- Pas de point de défaillance unique
- Plusieurs mécanismes indépendants
- Détection + prévention + réponse

Dans la section suivante, nous examinerons des attaques plus spécifiques qui exploitent les particularités de certains protocoles : SYN flood, ARP poisoning avancé, et DNS cache poisoning en profondeur.

⏭️ [Attaques spécifiques : SYN flood, ARP poisoning, DNS spoofing](/06-securite/03-attaques-specifiques.md)
