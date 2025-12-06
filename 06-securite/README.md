🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6. Sécurité dans TCP/IP

## Introduction

La pile de protocoles TCP/IP a été conçue dans les années 1970-1980, à une époque où Internet était un réseau de confiance réservé aux universités et aux institutions de recherche. La sécurité n'était pas une préoccupation majeure lors de sa conception initiale. Aujourd'hui, TCP/IP est le fondement d'un Internet global, ouvert et potentiellement hostile, transportant des données sensibles pour des milliards d'utilisateurs et d'organisations.

Cette évolution a créé un paradoxe : les protocoles qui font fonctionner Internet moderne contiennent des vulnérabilités structurelles qui ne peuvent être complètement éliminées sans repenser fondamentalement leur architecture. Comprendre ces vulnérabilités et les mécanismes de protection disponibles est devenu essentiel pour tout professionnel travaillant avec des réseaux et des systèmes distribués.

## Pourquoi la sécurité est-elle critique dans TCP/IP ?

### 1. Absence de sécurité native

La plupart des protocoles TCP/IP de base **ne fournissent aucune protection** contre :
- **L'écoute clandestine (eavesdropping)** : Les données transitent en clair sur le réseau
- **L'usurpation d'identité (spoofing)** : Rien ne garantit l'authenticité de l'expéditeur
- **La modification des données** : L'intégrité des paquets n'est pas vérifiée cryptographiquement
- **Le rejeu d'attaques** : Un paquet capturé peut être renvoyé ultérieurement

**Exemple concret** : Lorsque vous tapez `http://example.com` (sans HTTPS), votre navigateur envoie une requête HTTP en clair. Toute personne ayant accès au réseau entre vous et le serveur peut :
- Lire le contenu de votre requête (URL, cookies, données POST)
- Lire la réponse du serveur
- Modifier le contenu de la page avant qu'elle ne vous parvienne
- Injecter du code malveillant dans la réponse

### 2. Exposition globale

Contrairement aux réseaux d'entreprise traditionnels protégés par des périmètres physiques, TCP/IP est conçu pour l'interconnexion universelle :

- **Tout appareil connecté** est potentiellement accessible depuis n'importe où dans le monde
- **Les attaquants** peuvent être situés à des milliers de kilomètres de leur cible
- **L'anonymat** est facilité par la nature distribuée du réseau

**Exemple** : Un serveur web avec l'adresse IP publique `203.0.113.45` sera scanné et sondé par des robots automatisés dans les minutes suivant sa mise en ligne. Des tentatives d'exploitation de vulnérabilités connues commenceront immédiatement, sans intervention humaine.

### 3. Complexité des menaces modernes

Les attaques réseau ont considérablement évolué :

**Années 1990** : Attaques simples et visibles
- Balayage de ports manuel
- Exploitation de vulnérabilités connues
- Déni de service basique

**Années 2020** : Attaques sophistiquées et furtives
- Botnet distribués (DDoS à plusieurs Tbps)
- Attaques persistantes avancées (APT)
- Exfiltration de données imperceptible
- Compromission de la chaîne d'approvisionnement
- Attaques sur les protocoles de routage (BGP hijacking)

## Les couches de sécurité dans TCP/IP

La sécurité dans TCP/IP ne se limite pas à un seul niveau. Elle s'applique à différentes couches du modèle, chacune avec ses propres défis et solutions :

### Couche Liaison/Accès réseau
**Menaces** :
- ARP spoofing et ARP poisoning
- VLAN hopping
- MAC flooding

**Protections** :
- Port security sur les switches
- Dynamic ARP Inspection (DAI)
- VLAN Access Control Lists

**Exemple d'attaque** : Un attaquant sur le même réseau local envoie de fausses réponses ARP en se faisant passer pour la passerelle (routeur). Tous les paquets destinés à Internet passent désormais par sa machine, lui permettant d'intercepter ou de modifier le trafic (attaque de type "Man-in-the-Middle").

### Couche Internet (IP)
**Menaces** :
- IP spoofing (usurpation d'adresse source)
- Fragmentation attacks
- Route hijacking
- ICMP-based attacks (ping of death, smurf attack)

**Protections** :
- IPsec (authentification et chiffrement au niveau IP)
- Ingress/egress filtering
- RPKI (Resource Public Key Infrastructure) pour BGP

**Exemple d'attaque** : Un attaquant envoie des paquets avec une adresse IP source falsifiée pour contourner les règles de pare-feu qui autorisent le trafic depuis certaines adresses "de confiance".

### Couche Transport (TCP/UDP)
**Menaces** :
- SYN flood (épuisement des ressources)
- TCP session hijacking
- Port scanning
- UDP amplification attacks

**Protections** :
- SYN cookies
- Connection rate limiting
- Stateful firewall inspection
- Port knocking

**Exemple d'attaque SYN flood** : Un attaquant envoie des milliers de requêtes SYN sans jamais envoyer l'ACK final du three-way handshake. Le serveur maintient des connexions semi-ouvertes jusqu'à épuisement de ses ressources, rendant le service indisponible pour les utilisateurs légitimes.

### Couche Application
**Menaces** :
- DNS spoofing et cache poisoning
- HTTP injection (XSS, SQL injection)
- Man-in-the-Middle sur HTTPS mal configuré
- Email spoofing et phishing

**Protections** :
- TLS/SSL pour le chiffrement des communications
- DNSSEC pour l'authentification DNS
- SPF, DKIM, DMARC pour l'email
- WAF (Web Application Firewall)

**Exemple d'attaque DNS cache poisoning** : Un attaquant pollue le cache DNS d'un résolveur en y injectant de fausses entrées. Les utilisateurs croyant visiter leur banque (`www.banque.com`) sont redirigés vers un site malveillant parfaitement imité, où leurs identifiants sont volés.

## Évolution de la sécurité dans TCP/IP

### Première génération : Sécurité périmétrique (années 1990)
- **Philosophie** : "Crunchy outside, soft inside"
- **Approche** : Pare-feu au périmètre, tout est sûr à l'intérieur
- **Limite** : Vulnérable aux attaques internes et aux mouvements latéraux

### Deuxième génération : Défense en profondeur (années 2000)
- **Philosophie** : Multiples couches de sécurité
- **Approche** : Pare-feu + IDS/IPS + antivirus + segmentation réseau
- **Limite** : Complexité de gestion, faux positifs

### Troisième génération : Zero Trust (années 2020)
- **Philosophie** : "Never trust, always verify"
- **Approche** : Authentification et autorisation à chaque étape
- **Avantage** : Adapté au cloud, au télétravail et aux architectures distribuées

**Exemple de transition vers Zero Trust** :
```
Ancien modèle :
Utilisateur → VPN → Accès complet au réseau interne

Modèle Zero Trust :
Utilisateur → Authentification forte → Accès application par application
           → Vérification continue → Micro-segmentation
           → Chiffrement de bout en bout
```

## Chiffrement : la pierre angulaire de la sécurité moderne

Le chiffrement transforme les données lisibles (texte clair) en données inintelligibles (texte chiffré) pour quiconque ne possède pas la clé de déchiffrement.

### Chiffrement symétrique
- **Principe** : Une seule clé pour chiffrer et déchiffrer
- **Avantage** : Rapide, efficace pour de gros volumes
- **Inconvénient** : Partage sécurisé de la clé difficile
- **Algorithmes** : AES, ChaCha20
- **Usage** : Chiffrement du trafic (TLS, IPsec)

### Chiffrement asymétrique
- **Principe** : Paire de clés (publique/privée)
- **Avantage** : Pas besoin de partager de secret
- **Inconvénient** : Plus lent que le symétrique
- **Algorithmes** : RSA, ECDSA, Ed25519
- **Usage** : Échange de clés, signatures numériques, authentification

### Utilisation combinée (exemple TLS)
```
1. Handshake initial → Chiffrement asymétrique
   - Le serveur prouve son identité avec son certificat
   - Client et serveur négocient une clé de session

2. Échange de données → Chiffrement symétrique
   - Tout le trafic est chiffré avec la clé de session
   - Performance optimale

3. Signatures et intégrité → Fonctions de hachage
   - HMAC pour garantir l'intégrité des messages
   - Détection de toute modification
```

## Compromis sécurité/performance

La sécurité a un coût en termes de performance et de complexité :

| Aspect | Impact de la sécurité | Exemple |
|--------|----------------------|---------|
| **Latence** | +10-50ms pour TLS handshake | HTTPS vs HTTP |
| **Débit** | -5-20% pour le chiffrement | IPsec overhead |
| **CPU** | Calculs cryptographiques intensifs | 1000 connexions TLS/s |
| **Complexité** | Gestion des certificats, PKI | Rotation des clés |
| **Compatibilité** | Versions de protocoles | TLS 1.0 vs 1.3 |

**Stratégies d'optimisation** :
- **Hardware acceleration** : Cartes cryptographiques, instructions CPU (AES-NI)
- **Session resumption** : Réutilisation des clés TLS
- **Connection pooling** : Réutilisation des connexions chiffrées
- **Caching** : Réduction des handshakes complets

## Principe de moindre privilège dans le réseau

La sécurité réseau moderne applique le principe du **"moindre privilège"** :

1. **Bloquer par défaut** : Tout ce qui n'est pas explicitement autorisé est refusé
2. **Segmentation** : Isolation des ressources selon leur criticité
3. **Authentification forte** : Vérification rigoureuse de l'identité
4. **Chiffrement systématique** : Protection des données en transit et au repos
5. **Audit et monitoring** : Détection des anomalies et des intrusions

**Exemple d'architecture sécurisée** :
```
Internet
    ↓
[Pare-feu externe + WAF]
    ↓
Zone DMZ (serveurs web publics)
    ↓
[Pare-feu interne]
    ↓
Zone applicative (serveurs d'application)
    ↓
[Pare-feu base de données]
    ↓
Zone données (serveurs de bases de données)

Chaque transition :
- Authentification obligatoire
- Trafic chiffré (TLS/IPsec)
- Journalisation complète
- Règles de pare-feu strictes
```

## Défis contemporains

### 1. Cryptographie quantique
Les ordinateurs quantiques menacent les algorithmes de chiffrement actuels (RSA, ECDSA). La transition vers la **cryptographie post-quantique** est en cours.

### 2. Attaques sur la chaîne d'approvisionnement
Compromission de bibliothèques, de firmware ou de composants matériels avant même leur déploiement.

### 3. Attaques zero-day
Exploitation de vulnérabilités inconnues avant qu'un correctif ne soit disponible.

### 4. IoT et appareils non sécurisables
Milliards d'objets connectés avec sécurité faible ou inexistante, utilisés dans les botnets.

### 5. Cloud et responsabilité partagée
Frontière floue entre la sécurité fournie par le fournisseur cloud et celle dont le client est responsable.

## Objectifs de ce module

Dans les sections suivantes, nous explorerons en détail :

1. **Les vulnérabilités fondamentales** de TCP/IP et comment elles peuvent être exploitées
2. **Les attaques classiques et modernes** : leur fonctionnement technique et leurs impacts
3. **TLS/SSL** : le protocole qui sécurise une grande partie du web moderne
4. **IPsec** : sécurité au niveau réseau pour les VPN et les communications site-à-site
5. **Les VPN** : création de tunnels sécurisés à travers des réseaux non fiables
6. **Les pare-feu** : du filtrage basique aux systèmes modernes de détection d'intrusion
7. **Les bonnes pratiques** : comment concevoir et maintenir des systèmes réseau sécurisés

## Mentalité de sécurité

Avant de plonger dans les détails techniques, adoptons une mentalité de sécurité essentielle :

### Assumez la compromission
- Concevez des systèmes en partant du principe qu'ils seront attaqués
- Planifiez la détection et la réponse aux incidents
- Limitez l'impact potentiel (blast radius) de toute compromission

### Pensez comme un attaquant
- Comprenez les motivations et les méthodes des adversaires
- Identifiez les surfaces d'attaque de vos systèmes
- Testez régulièrement vos défenses (pentest, red team)

### Sécurité par conception
- Intégrez la sécurité dès la conception, pas comme une surcouche
- Privilégiez la simplicité : moins de code = moins de vulnérabilités
- Automatisez la sécurité (DevSecOps)

### Restez informé
- Les menaces évoluent constamment
- Suivez les bulletins de sécurité (CVE, CERT)
- Appliquez les mises à jour et correctifs rapidement

---

La sécurité dans TCP/IP n'est pas un état final à atteindre, mais un **processus continu d'amélioration** face à des menaces en constante évolution. Une compréhension approfondie des mécanismes de sécurité à chaque couche du modèle TCP/IP est indispensable pour construire et maintenir des systèmes réseau fiables et résilients.

Passons maintenant à l'analyse détaillée des vulnérabilités inhérentes à TCP/IP qui rendent ces mesures de sécurité nécessaires.

⏭️ [Vulnérabilités inhérentes à TCP/IP](/06-securite/01-vulnerabilites-inherentes.md)
