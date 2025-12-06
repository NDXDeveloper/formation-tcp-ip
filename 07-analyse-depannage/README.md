🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Module 7 : Analyse et dépannage réseau

## Introduction

L'analyse et le dépannage réseau constituent des compétences essentielles pour tout professionnel travaillant avec TCP/IP. Qu'il s'agisse d'un administrateur système, d'un ingénieur réseau, d'un développeur backend ou d'un expert en sécurité, la capacité à diagnostiquer et résoudre les problèmes de connectivité est indispensable au quotidien.

Dans les modules précédents, nous avons étudié comment fonctionnent les protocoles TCP/IP en théorie. Ce module se concentre sur **la pratique** : comment observer ce qui se passe réellement sur le réseau, comment identifier les anomalies, et comment résoudre méthodiquement les problèmes de communication.

## Pourquoi l'analyse réseau est-elle cruciale ?

### Dans un contexte professionnel

Les problèmes réseau peuvent avoir des impacts considérables :

- **Interruption de service** : Une application web inaccessible peut coûter des milliers d'euros par minute pour un site e-commerce
- **Dégradation de performance** : Une latence excessive peut rendre une application cloud inutilisable
- **Problèmes de sécurité** : Une anomalie réseau peut révéler une intrusion ou une fuite de données
- **Diagnostic applicatif** : Distinguer un problème réseau d'un bug logiciel nécessite des outils d'analyse précis

### Scénarios quotidiens

Voici des situations typiques où l'analyse réseau devient nécessaire :

**Exemple 1 : Application lente**
```
Situation : Les utilisateurs se plaignent que votre API REST répond lentement
Questions : Est-ce la latence réseau ? La bande passante saturée ? Un problème serveur ?
Solution : Analyse des temps de réponse, capture de paquets, vérification du routage
```

**Exemple 2 : Connexion impossible**
```
Situation : Un serveur de base de données n'est plus accessible depuis l'application
Questions : Le serveur est-il joignable ? Le port est-il ouvert ? Un firewall bloque-t-il ?
Solution : Tests de connectivité (ping, telnet), analyse des règles de filtrage
```

**Exemple 3 : Comportement intermittent**
```
Situation : Une connexion WebSocket se déconnecte aléatoirement toutes les 5 minutes
Questions : Timeout réseau ? Proxy intermédiaire ? Keep-alive mal configuré ?
Solution : Capture longue durée, analyse des patterns temporels, vérification TCP
```

## Les dimensions de l'analyse réseau

L'analyse réseau moderne s'articule autour de plusieurs axes complémentaires :

### 1. **La connectivité de base**
- Le chemin réseau est-il établi ?
- Les équipements intermédiaires répondent-ils ?
- Les routes sont-elles correctement configurées ?

### 2. **La résolution de noms**
- Le DNS fonctionne-t-il correctement ?
- Les enregistrements sont-ils à jour ?
- Le cache DNS pose-t-il problème ?

### 3. **Le transport des données**
- Les paquets arrivent-ils à destination ?
- Y a-t-il de la perte de paquets ?
- Quelle est la latence et le jitter ?

### 4. **Les protocoles applicatifs**
- Le handshake TLS réussit-il ?
- Les requêtes HTTP aboutissent-elles ?
- Les sessions TCP se ferment-elles proprement ?

### 5. **La sécurité**
- Y a-t-il du trafic suspect ?
- Les certificats sont-ils valides ?
- Des ports non autorisés sont-ils ouverts ?

## Vue d'ensemble du module

Ce module est organisé en huit sections progressives qui couvrent l'ensemble des compétences nécessaires :

### 🎯 Section 7.1 : Méthodologie de diagnostic
Apprenez une approche structurée pour résoudre les problèmes réseau de manière efficace. Nous verrons comment :
- Définir le problème précisément
- Isoler la couche défaillante (modèle OSI)
- Progresser du simple au complexe
- Documenter les résultats

### 🛠️ Section 7.2 : Outils en ligne de commande
Maîtrisez les outils essentiels disponibles sur tous les systèmes :
- `ping` : tester la connectivité et mesurer la latence
- `traceroute` : identifier le chemin réseau et localiser les blocages
- `netstat` / `ss` : examiner les connexions actives et les ports ouverts
- `nslookup` / `dig` : diagnostiquer les problèmes DNS
- Et bien d'autres outils indispensables

### 🔍 Section 7.3 : Wireshark - Principes
Découvrez le fonctionnement de l'analyseur de paquets le plus utilisé :
- Architecture et capture de trames
- Interface utilisateur et navigation
- Différence entre capture et affichage
- Bonnes pratiques de capture

### 📊 Section 7.4 : Lecture et interprétation des captures
Apprenez à lire les captures de paquets comme un livre :
- Décoder les en-têtes à chaque couche
- Suivre une conversation TCP complète
- Identifier les anomalies visuellement
- Reconnaître les patterns courants

### 🎨 Section 7.5 : Filtres Wireshark
Maîtrisez l'art du filtrage pour isoler le trafic pertinent :
- Filtres de capture vs filtres d'affichage
- Syntaxe des filtres Berkeley Packet Filter (BPF)
- Filtres d'affichage Wireshark avancés
- Filtres par protocole, adresse, port, contenu

### 🔧 Section 7.6 : Identification des problèmes courants
Reconnaissez et résolvez les problèmes typiques :
- Retransmissions TCP excessives
- Fenêtre TCP à zéro (zero window)
- Fragmentation IP problématique
- Problèmes ARP et MAC
- Erreurs de checksum

### ⚡ Section 7.7 : Analyse de performances
Mesurez et optimisez les performances réseau :
- Calcul du débit réel vs théorique
- Analyse de la latence et du RTT
- Détection des goulots d'étranglement
- Impact du TCP window scaling
- Métriques de qualité (jitter, perte de paquets)

### 📝 Section 7.8 : Logs et monitoring
Implémentez une surveillance proactive :
- Types de logs réseau (système, application, équipement)
- Outils de monitoring (Nagios, Zabbix, Prometheus)
- Agrégation et centralisation des logs
- Alerting et seuils
- Analyse post-mortem

## Les outils que nous couvrirons

### Outils de diagnostic de base
| Outil | Système | Usage principal |
|-------|---------|-----------------|
| `ping` | Linux/Windows/macOS | Test de connectivité, latence |
| `traceroute` / `tracert` | Linux/macOS / Windows | Traçage du chemin réseau |
| `netstat` | Tous | Statistiques réseau, connexions actives |
| `ss` | Linux | Remplacement moderne de netstat |
| `nslookup` | Tous | Requêtes DNS simples |
| `dig` | Linux/macOS | Requêtes DNS avancées |
| `host` | Linux/macOS | Résolution de noms |
| `nmap` | Tous | Scan de ports et découverte réseau |
| `tcpdump` | Linux/macOS | Capture de paquets en ligne de commande |
| `arp` | Tous | Affichage et manipulation du cache ARP |

### Outils d'analyse graphique
| Outil | Plateforme | Spécialité |
|-------|-----------|------------|
| **Wireshark** | Tous | Analyseur de paquets complet |
| **tshark** | Tous | Version CLI de Wireshark |
| **tcpdump** | Linux/macOS | Capture légère en ligne de commande |
| **Microsoft Network Monitor** | Windows | Alternative à Wireshark sur Windows |

### Outils de monitoring et supervision
| Outil | Type | Usage |
|-------|------|-------|
| **Nagios** | Open-source | Monitoring infrastructure |
| **Zabbix** | Open-source | Monitoring et métriques |
| **Prometheus + Grafana** | Open-source | Métriques temps réel et dashboards |
| **ELK Stack** | Open-source | Centralisation et analyse de logs |
| **Datadog** / **New Relic** | Commercial | Monitoring cloud et APM |

## Approche pédagogique de ce module

### Progression par la pratique

Chaque section de ce module adopte une approche progressive :

1. **Concept théorique** : Pourquoi cet outil existe, quel problème il résout
2. **Fonctionnement** : Comment l'outil opère techniquement
3. **Exemples concrets** : Cas d'usage réels avec sorties commentées
4. **Interprétation** : Comment lire et comprendre les résultats
5. **Pièges courants** : Erreurs fréquentes et comment les éviter

### Focus sur la compréhension

L'objectif n'est pas de mémoriser des commandes, mais de **comprendre** :
- Quelle information chercher
- Quel outil utiliser dans quelle situation
- Comment interpréter les résultats
- Comment croiser plusieurs sources d'information

### Exemples réalistes

Tous les exemples sont basés sur des situations réelles :
- Diagnostics d'applications web (HTTP/HTTPS)
- Problèmes de base de données (connexions TCP)
- Latence dans les APIs REST
- Problèmes de DNS en production
- Analyse de sécurité (détection d'anomalies)

## Prérequis pour ce module

Pour tirer le meilleur parti de ce module, vous devriez avoir assimilé :

- ✅ **Module 3 (Couche Internet)** : Comprendre IP, routage, ICMP
- ✅ **Module 4 (Couche Transport)** : Maîtriser TCP et UDP
- ✅ **Module 5 (Couche Application)** : Connaître DNS, HTTP, etc.

Les concepts suivants seront fréquemment utilisés :
- Adressage IP et masques de sous-réseau
- Numéros de port et sockets
- Three-way handshake TCP
- Format des paquets IP et segments TCP
- Résolution DNS

## Cas d'usage types par profil

### Pour les développeurs
- Débugger pourquoi une API REST met 5 secondes à répondre
- Comprendre pourquoi une WebSocket se déconnecte
- Analyser les retransmissions TCP impactant les performances
- Vérifier que le client HTTP réutilise bien les connexions (keep-alive)

### Pour les administrateurs système
- Diagnostiquer une panne de connectivité vers un serveur
- Identifier un équipement réseau défaillant dans le chemin
- Vérifier la configuration d'un firewall ou d'un proxy
- Analyser la consommation de bande passante

### Pour les ingénieurs réseau
- Optimiser les routes et protocoles de routage
- Détecter et résoudre des problèmes de MTU/fragmentation
- Analyser les performances d'un lien WAN
- Configurer et vérifier les VLAN et la QoS

### Pour les experts sécurité
- Détecter des scans de ports ou du trafic malveillant
- Analyser une intrusion ou une exfiltration de données
- Vérifier l'intégrité des certificats TLS
- Identifier des communications vers des C&C (Command & Control)

## La philosophie du dépannage réseau

### Principe 1 : Observer avant d'agir
Avant de modifier quoi que ce soit, **observez** l'état actuel du système. Une capture de paquets peut révéler instantanément un problème qui prendrait des heures à deviner.

### Principe 2 : Du simple au complexe
Commencez toujours par les tests les plus basiques (ping, nslookup) avant de passer aux outils complexes (Wireshark, tcpdump). Le modèle OSI est votre guide : testez couche par couche.

### Principe 3 : Hypothèses et validation
Formulez des hypothèses sur la cause du problème, puis testez-les méthodiquement. Une bonne hypothèse peut être validée ou infirmée rapidement.

### Principe 4 : Documenter les résultats
Gardez une trace de vos observations, commandes, et résultats. En cas de problème récurrent, cette documentation sera précieuse.

### Principe 5 : Comprendre le contexte
Un même symptôme peut avoir des causes différentes selon le contexte. Posez les bonnes questions :
- Le problème est-il permanent ou intermittent ?
- Affecte-t-il un seul client ou tous ?
- A-t-il commencé après un changement ?
- Y a-t-il un pattern temporel ?

## Structure des sections suivantes

Chaque section de ce module suit un format cohérent :

```
📌 Introduction
   ↓
🔍 Concepts clés
   ↓
💻 Exemples pratiques commentés
   ↓
📊 Interprétation des résultats
   ↓
⚠️ Pièges et erreurs courantes
   ↓
💡 Bonnes pratiques
```

## Exemple de workflow typique

Voici comment les outils de ce module s'articulent dans un diagnostic réel :

```
Problème signalé : "Le site web ne charge pas"
        ↓
1. ping example.com
   → Test de connectivité de base
   → Mesure de latence
        ↓
2. nslookup example.com
   → Vérification de la résolution DNS
   → Validation de l'adresse IP retournée
        ↓
3. traceroute example.com
   → Identification du chemin réseau
   → Localisation d'un éventuel blocage
        ↓
4. telnet example.com 80 (ou curl -v)
   → Test de connectivité applicative
   → Vérification que le port est ouvert
        ↓
5. Si nécessaire : tcpdump / Wireshark
   → Capture fine du trafic
   → Analyse du handshake TCP et HTTP
   → Détection d'anomalies au niveau paquet
```

## Compétences acquises à l'issue de ce module

À la fin de ce module, vous serez capable de :

- ✅ Diagnostiquer méthodiquement un problème de connectivité réseau
- ✅ Utiliser ping, traceroute, et autres outils CLI avec maîtrise
- ✅ Capturer et analyser du trafic réseau avec Wireshark
- ✅ Lire et comprendre une trace de paquets TCP/IP
- ✅ Créer des filtres de capture et d'affichage efficaces
- ✅ Identifier les problèmes de performances réseau
- ✅ Reconnaître les anomalies et problèmes courants
- ✅ Mettre en place un monitoring proactif
- ✅ Analyser les logs pour détecter des patterns
- ✅ Choisir le bon outil pour le bon problème

## Avertissement sur la capture de paquets

### Aspects légaux
La capture de paquets réseau soulève des questions légales importantes :
- **Sur votre propre réseau** : généralement autorisé
- **Sur un réseau d'entreprise** : nécessite une autorisation explicite
- **Sur un réseau public** : souvent illégal sans consentement

### Aspects éthiques
Même avec l'autorisation technique, respectez :
- La confidentialité des données personnelles (RGPD/GDPR)
- Les secrets commerciaux et données sensibles
- Les politiques de sécurité de l'organisation

### Bonnes pratiques
- N'utilisez ces outils que sur des réseaux dont vous avez la responsabilité
- Obtenez une autorisation écrite pour les environnements de production
- Ne capturez que le trafic nécessaire au diagnostic
- Supprimez les captures après analyse
- Anonymisez les données si vous devez les partager

## Ressources complémentaires

Avant d'entrer dans les sections détaillées, voici quelques ressources utiles :

### Documentation officielle
- **Wireshark** : https://www.wireshark.org/docs/
- **tcpdump** : https://www.tcpdump.org/manpages/
- **RFC sur ICMP** : RFC 792 (ICMPv4), RFC 4443 (ICMPv6)

### Livres de référence
- *"The TCP/IP Guide"* de Charles Kozierok
- *"Practical Packet Analysis"* de Chris Sanders
- *"Wireshark Network Analysis"* de Laura Chappell

### Outils d'apprentissage
- Wireshark Sample Captures : https://wiki.wireshark.org/SampleCaptures
- Packet Life : PCAP examples et cheat sheets
- CloudShark : plateforme d'analyse de captures en ligne

---

## Prêt pour le diagnostic ?

Vous avez maintenant une vue d'ensemble de ce qui vous attend dans ce module. L'analyse et le dépannage réseau sont des compétences qui s'acquièrent par la pratique. Chaque problème résolu enrichira votre expérience et affinera votre intuition.

**Conseil de départ** : Installez dès maintenant Wireshark et familiarisez-vous avec son interface. Capturez du trafic sur votre propre machine pendant que vous naviguez sur le web. Observer le trafic "normal" vous aidera à reconnaître l'anormal.

---

**Prochaine section** : 7.1 Méthodologie de diagnostic réseau

Dans la section suivante, nous aborderons une approche structurée et méthodique pour résoudre efficacement tout problème réseau, du plus simple au plus complexe.

⏭️ [Méthodologie de diagnostic réseau](/07-analyse-depannage/01-methodologie-diagnostic.md)
