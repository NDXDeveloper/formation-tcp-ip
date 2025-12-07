🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Références RFC essentielles

## Introduction aux RFC

### Qu'est-ce qu'une RFC ?

**RFC** signifie **Request for Comments** (Demande de commentaires). Il s'agit de documents techniques publiés par l'IETF (Internet Engineering Task Force) qui décrivent les spécifications, protocoles, procédures et concepts qui constituent l'Internet.

Malgré leur nom, les RFC ne sont pas de simples "demandes de commentaires" - elles constituent la documentation officielle et faisant autorité pour les standards Internet.

### Histoire et évolution

- **1969** : Première RFC (RFC 1) par Steve Crocker
- **1986** : Création de l'IETF
- **2024** : Plus de 9600 RFC publiées
- **Format** : Historiquement en texte ASCII pur, maintenant aussi en XML et HTML

### Types de RFC

Les RFC ont différents statuts selon leur maturité et leur objectif :

| Statut | Description | Exemple |
|--------|-------------|---------|
| **Standard** | Protocole mature et largement adopté | TCP (RFC 793) |
| **Proposed Standard** | Proposition stable prête pour l'implémentation | |
| **Best Current Practice** (BCP) | Recommandations et bonnes pratiques | BCP 38 (RFC 2827) |
| **Informational** | Information générale, pas un standard | RFC 1918 (adresses privées) |
| **Experimental** | Protocole expérimental | |
| **Historic** | Obsolète ou remplacé | RFC 791 → RFC 2460 (IPv6) |

### Comment lire une RFC

**Structure typique :**
```
Network Working Group                                    J. Postel
Request for Comments: XXX                                      ISI
                                                    Septembre 1981

                    TITRE DU PROTOCOLE

Statut de ce mémo
Abstract
Table des matières

1. Introduction
2. Spécifications
3. Format des en-têtes
4. Procédures
5. Considérations de sécurité
6. Références
```

**Sections importantes :**
- **Abstract** : Résumé en quelques lignes
- **Status** : Indique si c'est un standard, informatif, etc.
- **Introduction** : Contexte et objectifs
- **Specification** : Détails techniques
- **Security Considerations** : Implications de sécurité

---

## RFC fondamentales - Couche Internet

### IPv4

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 791** | Internet Protocol (IP) | ⭐⭐⭐⭐⭐ | Spécification originale d'IPv4, format des paquets, adressage |
| **RFC 950** | Internet Standard Subnetting Procedure | ⭐⭐⭐⭐ | Introduction du concept de sous-réseaux |
| **RFC 1918** | Address Allocation for Private Internets | ⭐⭐⭐⭐⭐ | Définit les plages d'adresses privées (10.x, 172.16.x, 192.168.x) |
| **RFC 1122** | Requirements for Internet Hosts | ⭐⭐⭐⭐ | Exigences pour les hôtes Internet, clarifications sur IP |
| **RFC 2474** | Definition of the Differentiated Services Field | ⭐⭐⭐ | QoS et priorisation du trafic (DSCP) |

**Exemple RFC 791 - Format du paquet IPv4 :**
```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### IPv6

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 2460** | Internet Protocol, Version 6 Specification | ⭐⭐⭐⭐⭐ | Spécification de base d'IPv6 (remplacée par RFC 8200) |
| **RFC 8200** | Internet Protocol, Version 6 Specification | ⭐⭐⭐⭐⭐ | Spécification IPv6 actuelle |
| **RFC 4291** | IP Version 6 Addressing Architecture | ⭐⭐⭐⭐⭐ | Architecture d'adressage IPv6 |
| **RFC 4861** | Neighbor Discovery for IPv6 | ⭐⭐⭐⭐ | Protocole de découverte de voisins (remplace ARP) |
| **RFC 4862** | IPv6 Stateless Address Autoconfiguration | ⭐⭐⭐⭐ | Configuration automatique d'adresses IPv6 |
| **RFC 6724** | Default Address Selection for IPv6 | ⭐⭐⭐ | Sélection d'adresses sources et destinations |

### ICMP

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 792** | Internet Control Message Protocol | ⭐⭐⭐⭐⭐ | ICMP pour IPv4 (ping, traceroute) |
| **RFC 4443** | ICMPv6 for IPv6 | ⭐⭐⭐⭐ | ICMP pour IPv6 |
| **RFC 1256** | ICMP Router Discovery Messages | ⭐⭐⭐ | Découverte automatique de routeurs |

### NAT

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 2663** | IP Network Address Translator Terminology | ⭐⭐⭐⭐ | Terminologie et concepts NAT |
| **RFC 3022** | Traditional IP NAT | ⭐⭐⭐⭐ | NAT traditionnel |
| **RFC 4787** | NAT Behavioral Requirements for UDP | ⭐⭐⭐ | Comportement NAT pour UDP |
| **RFC 5382** | NAT Behavioral Requirements for TCP | ⭐⭐⭐ | Comportement NAT pour TCP |

---

## RFC fondamentales - Couche Transport

### TCP

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 793** | Transmission Control Protocol | ⭐⭐⭐⭐⭐ | Spécification de base de TCP |
| **RFC 1122** | Requirements for Internet Hosts | ⭐⭐⭐⭐ | Section TCP, précisions importantes |
| **RFC 2018** | TCP Selective Acknowledgment Options | ⭐⭐⭐⭐ | SACK - acquittements sélectifs |
| **RFC 2581** | TCP Congestion Control | ⭐⭐⭐⭐⭐ | Algorithmes de contrôle de congestion |
| **RFC 5681** | TCP Congestion Control | ⭐⭐⭐⭐⭐ | Mise à jour de RFC 2581 |
| **RFC 6298** | Computing TCP's Retransmission Timer | ⭐⭐⭐⭐ | Calcul du RTO (Retransmission TimeOut) |
| **RFC 7323** | TCP Extensions for High Performance | ⭐⭐⭐⭐ | Window scaling, timestamps |
| **RFC 7413** | TCP Fast Open | ⭐⭐⭐ | Optimisation de l'établissement de connexion |
| **RFC 8985** | TCP Congestion Control with BBR | ⭐⭐⭐ | Algorithme BBR de Google |

**Diagramme d'états TCP (RFC 793) :**
```
                              +---------+
                              |  CLOSED |
                              +---------+
                                   |
                            passive open
                                   ↓
                              +---------+
                              |  LISTEN |
                              +---------+
                    send SYN  /         \  rcv SYN
                             /           \
                            ↓             ↓
                   +---------+         +---------+
                   |SYN-SENT |         |SYN-RCVD |
                   +---------+         +---------+
```

### UDP

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 768** | User Datagram Protocol | ⭐⭐⭐⭐⭐ | Spécification UDP - très courte (3 pages) |
| **RFC 8085** | UDP Usage Guidelines | ⭐⭐⭐⭐ | Recommandations pour l'utilisation d'UDP |

---

## RFC fondamentales - Couche Accès Réseau

### Ethernet et ARP

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 826** | Ethernet Address Resolution Protocol | ⭐⭐⭐⭐⭐ | ARP - résolution adresse IP → MAC |
| **RFC 903** | Reverse ARP | ⭐⭐⭐ | RARP - résolution inverse (obsolète) |
| **RFC 5227** | IPv4 Address Conflict Detection | ⭐⭐⭐ | Détection de conflits d'adresses IP |

### VLAN

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **IEEE 802.1Q** | Non-RFC mais essentiel | ⭐⭐⭐⭐ | Standard VLAN (référence IEEE, pas RFC) |

---

## RFC fondamentales - Routage

### Protocoles de routage

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 2328** | OSPF Version 2 | ⭐⭐⭐⭐⭐ | Open Shortest Path First pour IPv4 |
| **RFC 5340** | OSPF for IPv6 | ⭐⭐⭐⭐ | OSPFv3 pour IPv6 |
| **RFC 2453** | RIP Version 2 | ⭐⭐⭐ | Routing Information Protocol v2 |
| **RFC 4271** | Border Gateway Protocol 4 (BGP-4) | ⭐⭐⭐⭐⭐ | BGP - le protocole qui fait fonctionner Internet |
| **RFC 4760** | Multiprotocol Extensions for BGP-4 | ⭐⭐⭐⭐ | MP-BGP - support IPv6 et autres |
| **RFC 7908** | Problem Definition for BGP Persistence | ⭐⭐⭐ | Problématiques de persistance BGP |

---

## RFC fondamentales - Couche Application

### DNS

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 1034** | Domain Names - Concepts and Facilities | ⭐⭐⭐⭐⭐ | Concepts fondamentaux DNS |
| **RFC 1035** | Domain Names - Implementation | ⭐⭐⭐⭐⭐ | Implémentation DNS, format des messages |
| **RFC 2181** | Clarifications to the DNS Specification | ⭐⭐⭐⭐ | Clarifications importantes |
| **RFC 4033** | DNS Security Introduction and Requirements | ⭐⭐⭐⭐ | Introduction à DNSSEC |
| **RFC 4034** | Resource Records for DNSSEC | ⭐⭐⭐⭐ | Enregistrements DNSSEC |
| **RFC 4035** | Protocol Modifications for DNSSEC | ⭐⭐⭐⭐ | Modifications protocolaires DNSSEC |
| **RFC 7858** | DNS over TLS | ⭐⭐⭐⭐ | DoT - DNS chiffré sur TLS |
| **RFC 8484** | DNS Queries over HTTPS | ⭐⭐⭐⭐ | DoH - DNS sur HTTPS |

**Types d'enregistrements DNS (RFC 1035) :**
```
Type    Valeur  Signification
A       1       Adresse IPv4 d'hôte
NS      2       Serveur de noms autoritaire
CNAME   5       Nom canonique (alias)
MX      15      Échangeur de courrier
TXT     16      Texte arbitraire
AAAA    28      Adresse IPv6 d'hôte
```

### DHCP

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 2131** | Dynamic Host Configuration Protocol | ⭐⭐⭐⭐⭐ | DHCP pour IPv4 |
| **RFC 2132** | DHCP Options and BOOTP Vendor Extensions | ⭐⭐⭐⭐ | Options DHCP |
| **RFC 3315** | DHCPv6 | ⭐⭐⭐⭐ | DHCP pour IPv6 |
| **RFC 4361** | Node-specific Client Identifiers for DHCPv4 | ⭐⭐⭐ | Identifiants clients |

### HTTP

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 1945** | HTTP/1.0 | ⭐⭐⭐ | Version initiale HTTP (historique) |
| **RFC 2616** | HTTP/1.1 | ⭐⭐⭐⭐ | HTTP/1.1 original (remplacé) |
| **RFC 7230** | HTTP/1.1: Message Syntax and Routing | ⭐⭐⭐⭐⭐ | HTTP/1.1 moderne - syntaxe |
| **RFC 7231** | HTTP/1.1: Semantics and Content | ⭐⭐⭐⭐⭐ | HTTP/1.1 moderne - sémantique |
| **RFC 7232** | HTTP/1.1: Conditional Requests | ⭐⭐⭐⭐ | Requêtes conditionnelles |
| **RFC 7233** | HTTP/1.1: Range Requests | ⭐⭐⭐⭐ | Requêtes partielles |
| **RFC 7234** | HTTP/1.1: Caching | ⭐⭐⭐⭐⭐ | Mécanismes de cache |
| **RFC 7235** | HTTP/1.1: Authentication | ⭐⭐⭐⭐⭐ | Authentification HTTP |
| **RFC 7540** | HTTP/2 | ⭐⭐⭐⭐⭐ | HTTP version 2 |
| **RFC 9114** | HTTP/3 | ⭐⭐⭐⭐⭐ | HTTP version 3 sur QUIC |
| **RFC 6265** | HTTP State Management (Cookies) | ⭐⭐⭐⭐⭐ | Gestion des cookies |

### Email (SMTP, POP, IMAP)

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 5321** | Simple Mail Transfer Protocol | ⭐⭐⭐⭐⭐ | SMTP - envoi d'emails |
| **RFC 5322** | Internet Message Format | ⭐⭐⭐⭐⭐ | Format des messages email |
| **RFC 1939** | POP3 | ⭐⭐⭐⭐ | Post Office Protocol version 3 |
| **RFC 3501** | IMAP4rev1 | ⭐⭐⭐⭐⭐ | Internet Message Access Protocol |
| **RFC 6152** | SMTP Service Extension for 8bit-MIME | ⭐⭐⭐ | Support 8-bit pour SMTP |
| **RFC 2045-2049** | MIME | ⭐⭐⭐⭐ | Multipurpose Internet Mail Extensions |

### FTP et transfert de fichiers

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 959** | File Transfer Protocol | ⭐⭐⭐⭐ | FTP - spécification de base |
| **RFC 2228** | FTP Security Extensions | ⭐⭐⭐ | Extensions de sécurité FTP |
| **RFC 4217** | Securing FTP with TLS | ⭐⭐⭐ | FTPS - FTP sur TLS |

### SSH et Telnet

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 854** | Telnet Protocol | ⭐⭐⭐ | Telnet (obsolète, non sécurisé) |
| **RFC 4251** | SSH Protocol Architecture | ⭐⭐⭐⭐⭐ | Architecture SSH |
| **RFC 4252** | SSH Authentication Protocol | ⭐⭐⭐⭐⭐ | Authentification SSH |
| **RFC 4253** | SSH Transport Layer Protocol | ⭐⭐⭐⭐⭐ | Couche transport SSH |
| **RFC 4254** | SSH Connection Protocol | ⭐⭐⭐⭐⭐ | Protocole de connexion SSH |

### Autres protocoles applicatifs

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 1305** | Network Time Protocol v3 | ⭐⭐⭐⭐ | NTPv3 |
| **RFC 5905** | Network Time Protocol v4 | ⭐⭐⭐⭐⭐ | NTPv4 - synchronisation temps |
| **RFC 1157** | SNMP v1 | ⭐⭐⭐ | Simple Network Management Protocol |
| **RFC 3411-3418** | SNMP v3 | ⭐⭐⭐⭐ | SNMPv3 avec sécurité |
| **RFC 2616** | syslog | ⭐⭐⭐ | Protocole de journalisation |

---

## RFC fondamentales - Sécurité

### TLS/SSL

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 2246** | TLS Protocol Version 1.0 | ⭐⭐⭐ | TLS 1.0 (obsolète) |
| **RFC 4346** | TLS Protocol Version 1.1 | ⭐⭐⭐ | TLS 1.1 (obsolète) |
| **RFC 5246** | TLS Protocol Version 1.2 | ⭐⭐⭐⭐⭐ | TLS 1.2 (encore largement utilisé) |
| **RFC 8446** | TLS Protocol Version 1.3 | ⭐⭐⭐⭐⭐ | TLS 1.3 - version moderne |
| **RFC 6101** | SSL Protocol Version 3.0 | ⭐⭐ | SSL 3.0 (historique, vulnérable) |
| **RFC 5280** | X.509 PKI Certificate Profile | ⭐⭐⭐⭐⭐ | Certificats X.509 |
| **RFC 6066** | TLS Extensions | ⭐⭐⭐⭐ | Extensions TLS (SNI, etc.) |

### IPsec

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 4301** | Security Architecture for IP | ⭐⭐⭐⭐⭐ | Architecture de sécurité IPsec |
| **RFC 4302** | IP Authentication Header (AH) | ⭐⭐⭐⭐ | En-tête d'authentification |
| **RFC 4303** | IP Encapsulating Security Payload (ESP) | ⭐⭐⭐⭐⭐ | Charge utile de sécurité encapsulée |
| **RFC 7296** | IKEv2 | ⭐⭐⭐⭐⭐ | Internet Key Exchange version 2 |

### Autres protocoles de sécurité

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 6749** | OAuth 2.0 Authorization Framework | ⭐⭐⭐⭐⭐ | Framework d'autorisation OAuth 2.0 |
| **RFC 7519** | JSON Web Token (JWT) | ⭐⭐⭐⭐⭐ | Jetons JWT pour authentification |
| **RFC 7515** | JSON Web Signature (JWS) | ⭐⭐⭐⭐ | Signatures cryptographiques JSON |
| **RFC 7516** | JSON Web Encryption (JWE) | ⭐⭐⭐⭐ | Chiffrement de données JSON |
| **RFC 5246** | HMAC | ⭐⭐⭐⭐ | Hash-based Message Authentication Code |

---

## RFC - Bonnes pratiques et sécurité

### BCP (Best Current Practice)

| RFC | BCP | Titre | Importance | Description |
|-----|-----|-------|------------|-------------|
| **RFC 2827** | BCP 38 | Network Ingress Filtering | ⭐⭐⭐⭐⭐ | Filtrage anti-spoofing |
| **RFC 8085** | BCP 145 | UDP Usage Guidelines | ⭐⭐⭐⭐ | Utilisation appropriée d'UDP |
| **RFC 7504** | BCP 194 | SMTP 521/556 Reply Codes | ⭐⭐⭐ | Codes de réponse SMTP |
| **RFC 1918** | BCP 5 | Private Address Space | ⭐⭐⭐⭐⭐ | Adresses IP privées |
| **RFC 3330** | - | Special-Use IPv4 Addresses | ⭐⭐⭐⭐ | Adresses IPv4 à usage spécial |

### Sécurité et recommandations

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 3552** | Guidelines for Writing RFC Text on Security | ⭐⭐⭐⭐ | Guide pour les considérations de sécurité |
| **RFC 7258** | Pervasive Monitoring Is an Attack | ⭐⭐⭐⭐ | Surveillance généralisée = attaque |
| **RFC 7525** | TLS/DTLS Recommendations | ⭐⭐⭐⭐⭐ | Recommandations TLS/DTLS |
| **RFC 8996** | Deprecating TLS 1.0 and 1.1 | ⭐⭐⭐⭐ | Abandon de TLS 1.0/1.1 |

---

## RFC - Protocoles modernes et émergents

### QUIC et HTTP/3

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 9000** | QUIC: A UDP-Based Transport Protocol | ⭐⭐⭐⭐⭐ | Protocole de transport QUIC |
| **RFC 9001** | Using TLS to Secure QUIC | ⭐⭐⭐⭐⭐ | Sécurité QUIC avec TLS 1.3 |
| **RFC 9002** | QUIC Loss Detection and Congestion Control | ⭐⭐⭐⭐ | Détection de perte et congestion |
| **RFC 9114** | HTTP/3 | ⭐⭐⭐⭐⭐ | HTTP version 3 sur QUIC |
| **RFC 9204** | QPACK | ⭐⭐⭐⭐ | Compression d'en-têtes pour HTTP/3 |

### WebSocket et temps réel

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 6455** | WebSocket Protocol | ⭐⭐⭐⭐⭐ | Communication bidirectionnelle full-duplex |
| **RFC 8441** | Bootstrapping WebSockets with HTTP/2 | ⭐⭐⭐ | WebSockets sur HTTP/2 |

### Multicast et streaming

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 2236** | IGMPv2 | ⭐⭐⭐⭐ | Internet Group Management Protocol v2 |
| **RFC 3376** | IGMPv3 | ⭐⭐⭐⭐ | IGMP version 3 |
| **RFC 3550** | RTP | ⭐⭐⭐⭐⭐ | Real-time Transport Protocol |
| **RFC 3551** | RTP Audio and Video Profile | ⭐⭐⭐⭐ | Profils RTP |
| **RFC 4566** | SDP | ⭐⭐⭐⭐ | Session Description Protocol |

---

## RFC - Architectures et concepts

### REST et API

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 3986** | URI Generic Syntax | ⭐⭐⭐⭐⭐ | Syntaxe des URI |
| **RFC 7231** | HTTP/1.1 Semantics | ⭐⭐⭐⭐⭐ | Sémantique HTTP (base de REST) |
| **RFC 5988** | Web Linking | ⭐⭐⭐⭐ | Relations entre ressources web |
| **RFC 7807** | Problem Details for HTTP APIs | ⭐⭐⭐⭐ | Format standard pour les erreurs d'API |

### Encodage et formats

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 4648** | Base16, Base32, Base64 Encodings | ⭐⭐⭐⭐⭐ | Encodages base64 et variants |
| **RFC 7159** | JSON | ⭐⭐⭐⭐⭐ | JavaScript Object Notation |
| **RFC 8259** | JSON (STD 90) | ⭐⭐⭐⭐⭐ | Mise à jour de RFC 7159 |
| **RFC 7049** | CBOR | ⭐⭐⭐⭐ | Concise Binary Object Representation |

---

## RFC - Qualité de Service et performance

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 2474** | DiffServ Field | ⭐⭐⭐⭐ | Differentiated Services (QoS) |
| **RFC 2475** | DiffServ Architecture | ⭐⭐⭐⭐ | Architecture DiffServ |
| **RFC 3168** | ECN | ⭐⭐⭐⭐ | Explicit Congestion Notification |
| **RFC 8290** | Recent Advances in Congestion Control | ⭐⭐⭐ | Avancées en contrôle de congestion |

---

## RFC - Gestion et opérations

### Monitoring et diagnostics

| RFC | Titre | Importance | Description |
|-----|-------|------------|-------------|
| **RFC 5424** | Syslog Protocol | ⭐⭐⭐⭐ | Protocole syslog moderne |
| **RFC 5425** | TLS Transport for Syslog | ⭐⭐⭐ | Syslog sécurisé |
| **RFC 7540** | NETCONF over SSH | ⭐⭐⭐ | Configuration réseau |

---

## Comment utiliser les RFC

### Trouver une RFC

**Sources officielles :**
- **Site IETF** : https://www.ietf.org/rfc/
- **RFC Editor** : https://www.rfc-editor.org/
- **Datatracker** : https://datatracker.ietf.org/

**Formats disponibles :**
- Texte ASCII : Format original
- HTML : Plus lisible avec liens
- PDF : Pour impression
- XML : Format structuré

### Rechercher une RFC

**Par numéro :**
```
https://www.rfc-editor.org/rfc/rfcXXXX.html
https://datatracker.ietf.org/doc/html/rfcXXXX
```

**Par mot-clé :**
- Utiliser le moteur de recherche de l'IETF
- Consulter les index thématiques
- Vérifier les RFC qui mettent à jour ou obsolètent d'autres RFC

### Interpréter les mots-clés RFC

Les RFC utilisent des termes précis définis dans **RFC 2119** :

| Terme | Signification |
|-------|---------------|
| **MUST** / **SHALL** | Exigence absolue |
| **MUST NOT** / **SHALL NOT** | Interdiction absolue |
| **SHOULD** / **RECOMMENDED** | Fortement recommandé |
| **SHOULD NOT** / **NOT RECOMMENDED** | Fortement déconseillé |
| **MAY** / **OPTIONAL** | Optionnel |

**Exemple :**
> "An HTTP/1.1 server **MUST** support chunked transfer encoding"

Cela signifie qu'un serveur HTTP/1.1 qui ne supporte pas le chunked encoding n'est **pas conforme** à la spécification.

---

## RFC obsolètes mais historiquement importantes

| RFC | Titre | Remplacée par | Importance historique |
|-----|-------|---------------|----------------------|
| **RFC 791** | IPv4 | Toujours actuelle | ⭐⭐⭐⭐⭐ Document fondateur |
| **RFC 793** | TCP | Toujours actuelle | ⭐⭐⭐⭐⭐ Document fondateur |
| **RFC 1** | Host Software | - | ⭐⭐⭐⭐⭐ Première RFC (1969) |
| **RFC 2616** | HTTP/1.1 | RFC 723x | ⭐⭐⭐⭐ Base de HTTP moderne |
| **RFC 2460** | IPv6 | RFC 8200 | ⭐⭐⭐⭐ Première spec IPv6 |

---

## Suivi des mises à jour

### Statuts de RFC

Une RFC peut avoir plusieurs statuts de relation :

- **Updates** : Modifie partiellement une RFC existante
- **Obsoletes** : Remplace complètement une RFC
- **Updated by** : Modifiée par une RFC ultérieure
- **Obsoleted by** : Remplacée par une RFC ultérieure

**Exemple - Évolution de HTTP :**
```
RFC 1945 (HTTP/1.0)
    ↓
RFC 2068 (HTTP/1.1) → obsoleted by RFC 2616
    ↓
RFC 2616 (HTTP/1.1) → obsoleted by RFC 7230-7235
    ↓
RFC 7230-7235 (HTTP/1.1 moderne)
    ↓
RFC 7540 (HTTP/2)
    ↓
RFC 9114 (HTTP/3)
```

### Vérifier la pertinence

Avant d'implémenter une RFC :
1. Vérifier qu'elle n'est pas obsolète
2. Consulter les RFC qui la mettent à jour
3. Lire les errata officiels
4. Vérifier le statut (Standard, Proposed, Experimental)

---

## Collections de RFC par thématique

### Le minimum vital pour comprendre Internet

**Les "Core Internet Protocols" :**
1. **RFC 791** - IPv4
2. **RFC 793** - TCP
3. **RFC 768** - UDP
4. **RFC 792** - ICMP
5. **RFC 826** - ARP
6. **RFC 1034/1035** - DNS
7. **RFC 2131** - DHCP

### Pour le développement web

1. **RFC 7230-7235** - HTTP/1.1
2. **RFC 7540** - HTTP/2
3. **RFC 9114** - HTTP/3
4. **RFC 6455** - WebSocket
5. **RFC 6749** - OAuth 2.0
6. **RFC 7519** - JWT
7. **RFC 8259** - JSON

### Pour la sécurité

1. **RFC 8446** - TLS 1.3
2. **RFC 4301-4303** - IPsec
3. **RFC 7296** - IKEv2
4. **RFC 5280** - Certificats X.509
5. **RFC 7525** - Recommandations TLS

---

## Outils pour travailler avec les RFC

### Lecteurs et parsers

**Outils en ligne :**
- RFC Editor : Version HTML avec index
- Tools.ietf.org : Outils de recherche avancés
- RFC Digest : Résumés de RFC

**Outils hors ligne :**
```bash
# Linux - installer les RFC localement
sudo apt-get install doc-rfc

# Lire une RFC
man rfc793
```

### Validation de conformité

Pour vérifier la conformité à une RFC :
- Lire attentivement les sections "MUST" et "SHOULD"
- Consulter les test suites officiels si disponibles
- Utiliser des outils de validation (ex: validators W3C pour HTTP)

---

## Processus de standardisation

### Étapes de maturation d'une RFC

```
Internet-Draft (I-D)
    ↓
Proposed Standard
    ↓
Draft Standard (obsolète depuis 2011)
    ↓
Internet Standard (STD)
```

**Note :** Depuis 2011, le processus Draft Standard a été supprimé.

### Qui produit les RFC ?

- **IETF** : Protocoles Internet
- **IRTF** : Recherche
- **IAB** : Architecture
- **RFC Editor** : Publication indépendante

---

## Conseils de lecture

### Pour débutants

1. Commencer par les **Abstract** et **Introduction**
2. Regarder les **exemples** avant les spécifications détaillées
3. Utiliser les versions **HTML** plus lisibles que le texte brut
4. Consulter des **tutoriels** en parallèle pour contextualiser

### Pour experts

1. Lire les sections **Security Considerations**
2. Étudier les **MUST/SHOULD/MAY** pour l'implémentation
3. Vérifier les **errata** officiels
4. Consulter les RFC qui **update** ou **obsolete** le document

---

## Références et ressources

**Sites officiels :**
- IETF : https://www.ietf.org
- RFC Editor : https://www.rfc-editor.org
- Datatracker : https://datatracker.ietf.org

**Outils de recherche :**
- RFC Search : https://www.rfc-editor.org/search/
- IETF Tools : https://tools.ietf.org

**Listes de diffusion :**
- ietf-announce : Annonces officielles
- Groupes de travail spécifiques sur lists.ietf.org

---

*Les RFC sont le cœur de la documentation technique d'Internet. Leur lecture peut être ardue, mais elles restent la source d'autorité pour toute implémentation conforme aux standards.*



⏭️ [D. Ressources et lectures complémentaires](/annexes/04-ressources.md)
