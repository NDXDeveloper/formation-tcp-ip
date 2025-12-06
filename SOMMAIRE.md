# Table des matières - Formation TCP/IP complète

## [1. Introduction et fondamentaux des réseaux](01-introduction/README.md)

- 1.1 [Histoire et évolution des réseaux informatiques](01-introduction/01-histoire-evolution-reseaux.md)
- 1.2 [Qu'est-ce qu'un protocole de communication ?](01-introduction/02-protocole-communication.md)
- 1.3 [Le modèle OSI : les 7 couches expliquées](01-introduction/03-modele-osi.md)
- 1.4 [Le modèle TCP/IP : architecture en 4 couches](01-introduction/04-modele-tcp-ip.md)
- 1.5 [Comparaison OSI vs TCP/IP](01-introduction/05-comparaison-osi-tcp-ip.md)
- 1.6 [Encapsulation et décapsulation des données](01-introduction/06-encapsulation-decapsulation.md)

---

## [2. La couche Accès réseau](02-couche-acces-reseau/README.md)

- 2.1 [Rôle et responsabilités de la couche](02-couche-acces-reseau/01-role-responsabilites.md)
- 2.2 [Technologies Ethernet : principes et évolution](02-couche-acces-reseau/02-technologies-ethernet.md)
- 2.3 [Adresses MAC : structure et fonctionnement](02-couche-acces-reseau/03-adresses-mac.md)
- 2.4 [Trames Ethernet : format et analyse](02-couche-acces-reseau/04-trames-ethernet.md)
- 2.5 [Commutation (switching) et domaines de collision](02-couche-acces-reseau/05-commutation-domaines-collision.md)
- 2.6 [ARP (Address Resolution Protocol)](02-couche-acces-reseau/06-arp.md)
- 2.7 [Technologies sans fil (Wi-Fi) et leur intégration](02-couche-acces-reseau/07-technologies-sans-fil.md)
- 2.8 [VLAN : segmentation logique des réseaux](02-couche-acces-reseau/08-vlan.md)

---

## [3. La couche Internet (IP)](03-couche-internet/README.md)

- 3.1 [Rôle de la couche Internet](03-couche-internet/01-role-couche-internet.md)
- 3.2 [IPv4 : structure et format des paquets](03-couche-internet/02-ipv4-structure-paquets.md)
- 3.3 [Adressage IPv4 : classes, notation CIDR](03-couche-internet/03-adressage-ipv4.md)
- 3.4 [Sous-réseaux (subnetting) : calculs et conception](03-couche-internet/04-sous-reseaux.md)
- 3.5 [Adresses publiques vs privées (RFC 1918)](03-couche-internet/05-adresses-publiques-privees.md)
- 3.6 [NAT et PAT : principes et mécanismes](03-couche-internet/06-nat-pat.md)
- 3.7 [IPv6 : pourquoi, comment, structure des adresses](03-couche-internet/07-ipv6.md)
- 3.8 [Coexistence IPv4/IPv6 : dual-stack, tunneling](03-couche-internet/08-coexistence-ipv4-ipv6.md)
- 3.9 [ICMP : diagnostic et messages de contrôle](03-couche-internet/09-icmp.md)
- 3.10 [Routage : concepts fondamentaux](03-couche-internet/10-routage-concepts.md)
- 3.11 [Tables de routage et algorithmes](03-couche-internet/11-tables-routage.md)
- 3.12 [Protocoles de routage](03-couche-internet/12-protocoles-routage.md)
    - 3.12.1 [Routage statique vs dynamique](03-couche-internet/12.1-routage-statique-dynamique.md)
    - 3.12.2 [Protocoles à vecteur de distance : RIP](03-couche-internet/12.2-protocoles-vecteur-distance.md)
    - 3.12.3 [Protocoles à état de lien : OSPF](03-couche-internet/12.3-protocoles-etat-lien.md)
    - 3.12.4 [BGP : le protocole d'Internet](03-couche-internet/12.4-bgp.md)
    - 3.12.5 [Métriques et convergence](03-couche-internet/12.5-metriques-convergence.md)
    - 3.12.6 [Implications pour les architectures cloud et microservices](03-couche-internet/12.6-implications-cloud-microservices.md)

---

## [4. La couche Transport](04-couche-transport/README.md)

- 4.1 [Rôle de la couche Transport](04-couche-transport/01-role-couche-transport.md)
- 4.2 [Notion de ports et de sockets](04-couche-transport/02-ports-sockets.md)
- 4.3 [Ports bien connus, enregistrés et dynamiques](04-couche-transport/03-ports-categories.md)
- 4.4 [UDP (User Datagram Protocol)](04-couche-transport/04-udp.md)
    - 4.4.1 [Caractéristiques et cas d'usage](04-couche-transport/04.1-udp-caracteristiques.md)
    - 4.4.2 [Format du datagramme UDP](04-couche-transport/04.2-udp-format-datagramme.md)
    - 4.4.3 [Avantages et limitations](04-couche-transport/04.3-udp-avantages-limitations.md)
- 4.5 [TCP (Transmission Control Protocol)](04-couche-transport/05-tcp.md)
    - 4.5.1 [Caractéristiques : fiabilité, ordre, contrôle](04-couche-transport/05.1-tcp-caracteristiques.md)
    - 4.5.2 [Format du segment TCP](04-couche-transport/05.2-tcp-format-segment.md)
    - 4.5.3 [Établissement de connexion : le 3-way handshake](04-couche-transport/05.3-tcp-3way-handshake.md)
    - 4.5.4 [Fermeture de connexion : 4-way handshake](04-couche-transport/05.4-tcp-4way-handshake.md)
    - 4.5.5 [Numéros de séquence et acquittements](04-couche-transport/05.5-tcp-numeros-sequence.md)
    - 4.5.6 [Fenêtre glissante (sliding window)](04-couche-transport/05.6-tcp-fenetre-glissante.md)
    - 4.5.7 [Contrôle de flux](04-couche-transport/05.7-tcp-controle-flux.md)
    - 4.5.8 [Contrôle de congestion : slow start, congestion avoidance](04-couche-transport/05.8-tcp-controle-congestion.md)
    - 4.5.9 [Retransmissions et timers](04-couche-transport/05.9-tcp-retransmissions.md)
    - 4.5.10 [États d'une connexion TCP (diagramme d'états)](04-couche-transport/05.10-tcp-etats-connexion.md)
- 4.6 [Comparaison TCP vs UDP : critères de choix](04-couche-transport/06-tcp-vs-udp.md)

---

## [5. La couche Application](05-couche-application/README.md)

- 5.1 [Rôle et fonctionnement de la couche Application](05-couche-application/01-role-fonctionnement.md)
- 5.2 [DNS (Domain Name System)](05-couche-application/02-dns.md)
    - 5.2.1 [Architecture hiérarchique](05-couche-application/02.1-dns-architecture.md)
    - 5.2.2 [Types d'enregistrements (A, AAAA, CNAME, MX, TXT, etc.)](05-couche-application/02.2-dns-types-enregistrements.md)
    - 5.2.3 [Résolution récursive vs itérative](05-couche-application/02.3-dns-resolution.md)
    - 5.2.4 [Cache DNS et TTL](05-couche-application/02.4-dns-cache-ttl.md)
- 5.3 [DHCP (Dynamic Host Configuration Protocol)](05-couche-application/03-dhcp.md)
    - 5.3.1 [Processus DORA](05-couche-application/03.1-dhcp-processus-dora.md)
    - 5.3.2 [Baux et renouvellement](05-couche-application/03.2-dhcp-baux-renouvellement.md)
- 5.4 [HTTP/HTTPS](05-couche-application/04-http-https.md)
    - 5.4.1 [Méthodes HTTP](05-couche-application/04.1-http-methodes.md)
    - 5.4.2 [Codes de statut](05-couche-application/04.2-http-codes-statut.md)
    - 5.4.3 [Headers et cookies](05-couche-application/04.3-http-headers-cookies.md)
    - 5.4.4 [HTTP/1.1 : connexions persistantes et pipelining](05-couche-application/04.4-http-1-1.md)
    - 5.4.5 [HTTP/2 : multiplexage et server push](05-couche-application/04.5-http-2.md)
    - 5.4.6 [HTTP/3 et QUIC : transport sur UDP](05-couche-application/04.6-http-3-quic.md)
    - 5.4.7 [Évolution des performances web](05-couche-application/04.7-evolution-performances-web.md)
- 5.5 [FTP et SFTP](05-couche-application/05-ftp-sftp.md)
- 5.6 [SMTP, POP3, IMAP : protocoles de messagerie](05-couche-application/06-smtp-pop3-imap.md)
- 5.7 [SSH et Telnet](05-couche-application/07-ssh-telnet.md)
- 5.8 [SNMP : supervision réseau](05-couche-application/08-snmp.md)
- 5.9 [NTP : synchronisation temporelle](05-couche-application/09-ntp.md)

---

## [6. Sécurité dans TCP/IP](06-securite/README.md)

- 6.1 [Vulnérabilités inhérentes à TCP/IP](06-securite/01-vulnerabilites-inherentes.md)
- 6.2 [Attaques courantes : spoofing, sniffing, man-in-the-middle](06-securite/02-attaques-courantes.md)
- 6.3 [Attaques spécifiques : SYN flood, ARP poisoning, DNS spoofing](06-securite/03-attaques-specifiques.md)
- 6.4 [TLS/SSL : chiffrement des communications](06-securite/04-tls-ssl.md)
    - 6.4.1 [Handshake TLS](06-securite/04.1-tls-handshake.md)
    - 6.4.2 [Certificats et PKI](06-securite/04.2-tls-certificats-pki.md)
- 6.5 [IPsec : sécurité au niveau réseau](06-securite/05-ipsec.md)
    - 6.5.1 [Modes transport et tunnel](06-securite/05.1-ipsec-modes.md)
    - 6.5.2 [AH et ESP](06-securite/05.2-ipsec-ah-esp.md)
- 6.6 [VPN : principes et protocoles](06-securite/06-vpn.md)
- 6.7 [Pare-feu : filtrage de paquets, stateful inspection](06-securite/07-pare-feu.md)
- 6.8 [Bonnes pratiques de sécurisation](06-securite/08-bonnes-pratiques.md)

---

## [7. Analyse et dépannage réseau](07-analyse-depannage/README.md)

- 7.1 [Méthodologie de diagnostic réseau](07-analyse-depannage/01-methodologie-diagnostic.md)
- 7.2 [Outils en ligne de commande : ping, traceroute, netstat, ss, nslookup, dig](07-analyse-depannage/02-outils-ligne-commande.md)
- 7.3 [Analyse de trames avec Wireshark : principes](07-analyse-depannage/03-wireshark-principes.md)
- 7.4 [Lecture et interprétation des captures](07-analyse-depannage/04-lecture-captures.md)
- 7.5 [Filtres d'affichage et de capture](07-analyse-depannage/05-filtres-wireshark.md)
- 7.6 [Identification des problèmes courants](07-analyse-depannage/06-problemes-courants.md)
- 7.7 [Analyse de performances réseau](07-analyse-depannage/07-analyse-performances.md)
- 7.8 [Logs et monitoring](07-analyse-depannage/08-logs-monitoring.md)

---

## [8. Programmation réseau (perspective développeur)](08-programmation-reseau/README.md)

- 8.1 [L'API Socket : concepts fondamentaux](08-programmation-reseau/01-api-socket-concepts.md)
- 8.2 [Sockets TCP : création, connexion, échange, fermeture](08-programmation-reseau/02-sockets-tcp.md)
- 8.3 [Sockets UDP : envoi et réception de datagrammes](08-programmation-reseau/03-sockets-udp.md)
- 8.4 [Gestion des erreurs réseau](08-programmation-reseau/04-gestion-erreurs.md)
- 8.5 [I/O bloquant vs non-bloquant](08-programmation-reseau/05-io-bloquant-non-bloquant.md)
- 8.6 [Multiplexage : select, poll, epoll](08-programmation-reseau/06-multiplexage.md)
- 8.7 [Programmation asynchrone et event-driven](08-programmation-reseau/07-programmation-asynchrone.md)
- 8.8 [Résilience et fiabilité applicative](08-programmation-reseau/08-resilience-fiabilite.md)
    - 8.8.1 [Timeouts : lecture, écriture, connexion](08-programmation-reseau/08.1-timeouts.md)
    - 8.8.2 [Retry logic et backoff exponentiel](08-programmation-reseau/08.2-retry-logic-backoff.md)
    - 8.8.3 [Circuit breakers](08-programmation-reseau/08.3-circuit-breakers.md)
    - 8.8.4 [Keep-alive et connexions persistantes](08-programmation-reseau/08.4-keep-alive.md)
- 8.9 [Communication temps réel : panorama complet](08-programmation-reseau/09-communication-temps-reel.md)
    - 8.9.1 [Polling et long-polling : principes et limites](08-programmation-reseau/09.1-polling-long-polling.md)
    - 8.9.2 [Server-Sent Events (SSE) : streaming unidirectionnel](08-programmation-reseau/09.2-server-sent-events.md)
    - 8.9.3 [WebSockets : communication bidirectionnelle full-duplex](08-programmation-reseau/09.3-websockets.md)
    - 8.9.4 [Comparaison et critères de choix selon le cas d'usage](08-programmation-reseau/09.4-comparaison-choix.md)
- 8.10 [Architectures de communication modernes](08-programmation-reseau/10-architectures-communication-modernes.md)
    - 8.10.1 [REST : principes et contraintes réseau](08-programmation-reseau/10.1-rest.md)
    - 8.10.2 [gRPC : HTTP/2 et Protocol Buffers](08-programmation-reseau/10.2-grpc.md)
    - 8.10.3 [GraphQL : considérations réseau](08-programmation-reseau/10.3-graphql.md)
    - 8.10.4 [Comparaison et critères de choix](08-programmation-reseau/10.4-comparaison-choix.md)
- 8.11 [Sérialisation des données : JSON, Protocol Buffers, MessagePack](08-programmation-reseau/11-serialisation-donnees.md)
- 8.12 [Bibliothèques réseau modernes (aperçu)](08-programmation-reseau/12-bibliotheques-modernes.md)
- 8.13 [Conception d'applications client-serveur](08-programmation-reseau/13-conception-client-serveur.md)
- 8.14 [Implications réseau pour les microservices](08-programmation-reseau/14-implications-microservices.md)

---

## [9. Architectures et concepts avancés](09-architectures-avancees/README.md)

- 9.1 [Qualité de Service (QoS)](09-architectures-avancees/01-qualite-service.md)
- 9.2 [Multicast et IGMP](09-architectures-avancees/02-multicast-igmp.md)
- 9.3 [Load balancing : principes et niveaux (L4, L7)](09-architectures-avancees/03-load-balancing.md)
- 9.4 [Proxys et reverse proxys](09-architectures-avancees/04-proxys-reverse-proxys.md)
- 9.5 [CDN (Content Delivery Networks)](09-architectures-avancees/05-cdn.md)
- 9.6 [Architectures haute disponibilité](09-architectures-avancees/06-haute-disponibilite.md)
- 9.7 [Conteneurs et réseaux (Docker, Kubernetes networking)](09-architectures-avancees/07-conteneurs-reseaux.md)
- 9.8 [Software-Defined Networking (SDN) : introduction](09-architectures-avancees/08-sdn.md)
- 9.9 [Cloud networking : concepts clés](09-architectures-avancees/09-cloud-networking.md)

---

## [10. Évolutions et tendances](10-evolutions-tendances/README.md)

- 10.1 [Adoption d'IPv6 : état des lieux](10-evolutions-tendances/01-adoption-ipv6.md)
- 10.2 [QUIC : standardisation et adoption](10-evolutions-tendances/02-quic.md)
- 10.3 [DNS over HTTPS (DoH) et DNS over TLS (DoT)](10-evolutions-tendances/03-doh-dot.md)
- 10.4 [Zero Trust Networking](10-evolutions-tendances/04-zero-trust.md)
- 10.5 [Edge computing et implications réseau](10-evolutions-tendances/05-edge-computing.md)
- 10.6 [IoT et contraintes protocolaires](10-evolutions-tendances/06-iot.md)
- 10.7 [5G et convergence fixe-mobile](10-evolutions-tendances/07-5g-convergence.md)
- 10.8 [Perspectives d'évolution de TCP/IP](10-evolutions-tendances/08-perspectives-evolution.md)

---

## Annexes

- [A. Glossaire des termes réseau](annexes/01-glossaire.md)
- [B. Tableau récapitulatif des ports standards](annexes/02-ports-standards.md)
- [C. Références RFC essentielles](annexes/03-rfc-essentielles.md)
- [D. Ressources et lectures complémentaires](annexes/04-ressources.md)
- [E. Exemples de code socket](annexes/05-exemples-code/README.md)
    - [E.1 Exemples Python](annexes/05-exemples-code/python/README.md)
    - [E.2 Exemples Go](annexes/05-exemples-code/go/README.md)
    - [E.3 Exemples JavaScript](annexes/05-exemples-code/javascript/README.md)

---

*Dernière mise à jour : Décembre 2025*
