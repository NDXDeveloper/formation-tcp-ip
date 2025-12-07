🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8. Programmation réseau (perspective développeur)

## Introduction

Après avoir exploré les fondamentaux théoriques de TCP/IP dans les modules précédents, nous abordons maintenant la **programmation réseau** du point de vue du développeur. Ce module fait le pont entre la théorie protocolaire et la pratique du développement d'applications réseau.

Comprendre TCP/IP en tant que développeur ne se limite pas à connaître les différentes couches du modèle : il s'agit de **savoir comment ces protocoles se traduisent en code**, quelles abstractions les langages de programmation nous offrent, et comment concevoir des applications réseau robustes, performantes et fiables.

## Pourquoi ce module est crucial pour les développeurs

### 1. **Du protocole au code**

La plupart des développeurs utilisent quotidiennement des bibliothèques réseau de haut niveau (axios, fetch, requests, etc.) sans nécessairement comprendre ce qui se passe "sous le capot". Cette abstraction est utile, mais elle peut devenir une boîte noire lorsque :

- Les performances ne sont pas au rendez-vous
- Des erreurs réseau intermittentes surviennent
- Il faut déboguer un problème de connectivité
- On doit optimiser la latence ou le débit
- On conçoit des systèmes distribués complexes

**Exemple concret** : Un développeur utilise `fetch()` en JavaScript pour appeler une API REST. L'appel échoue sporadiquement avec des timeouts. Sans comprendre TCP (établissement de connexion, retransmissions, fenêtre de congestion), il sera difficile de diagnostiquer si le problème vient du réseau, du serveur, ou de la configuration du client.

### 2. **L'impact des choix architecturaux**

Les décisions prises au niveau applicatif ont des répercussions directes sur le comportement réseau :

- **Choix TCP vs UDP** : Un système de chat en temps réel doit-il privilégier la fiabilité (TCP) ou la rapidité (UDP) ?
- **Connexions persistantes vs courtes** : Une application mobile doit-elle maintenir une connexion WebSocket ouverte ou faire des requêtes HTTP ponctuelles ?
- **Sérialisation** : JSON vs Protocol Buffers vs MessagePack — quel impact sur la bande passante ?

**Cas d'usage réel** : Une application de trading haute fréquence peut perdre des opportunités de marché à cause d'une latence de quelques millisecondes. Le choix d'UDP avec un protocole personnalisé plutôt que HTTP/REST peut faire la différence.

### 3. **Résilience et fiabilité**

Les réseaux sont **intrinsèquement peu fiables** : paquets perdus, latence variable, connexions interrompues. Le développeur doit concevoir son application pour gérer ces défaillances :

- Timeouts appropriés
- Stratégies de retry avec backoff exponentiel
- Circuit breakers pour éviter la surcharge
- Gestion gracieuse des déconnexions

**Exemple concret** : Un service de paiement qui ne gère pas correctement les timeouts réseau peut facturer un client plusieurs fois si celui-ci clique plusieurs fois sur "Payer" en cas de lenteur.

### 4. **Performance et scalabilité**

À mesure qu'une application grandit, les enjeux de performance réseau deviennent critiques :

- Comment gérer 10 000 connexions simultanées ?
- Quand utiliser du multiplexage I/O (epoll, kqueue) ?
- Comment optimiser le nombre de round-trips réseau ?
- Quelles sont les implications d'HTTP/2 vs HTTP/1.1 ?

**Cas d'usage réel** : Un service de streaming vidéo doit optimiser chaque milliseconde de latence et chaque octet transféré pour offrir une expérience fluide à des millions d'utilisateurs simultanés.

## Vue d'ensemble du module

Ce module est structuré pour vous accompagner de l'API Socket bas niveau jusqu'aux architectures de communication modernes.

### **Fondamentaux de la programmation réseau**

Nous commencerons par l'**API Socket**, l'interface standard pour la programmation réseau présente dans tous les systèmes d'exploitation modernes. Vous comprendrez :

- Comment créer, connecter et fermer des sockets
- La différence entre sockets TCP (orientés connexion) et UDP (datagrammes)
- La gestion des erreurs réseau
- Les modes bloquant et non-bloquant

### **Gestion avancée des I/O**

Le passage à l'échelle nécessite des techniques avancées :

- **Multiplexage I/O** : `select()`, `poll()`, `epoll()` pour gérer de nombreuses connexions
- **Programmation asynchrone** : event loops, callbacks, promises, async/await
- **Patterns architecturaux** : event-driven, reactor pattern

**Cas d'usage** : Un serveur Node.js utilisant l'event loop peut gérer des dizaines de milliers de connexions concurrentes sur un seul thread, là où une approche thread-per-connection serait limitée.

### **Résilience applicative**

La fiabilité ne vient pas uniquement des protocoles (TCP garantit la livraison) mais aussi de la couche applicative :

- Configuration de timeouts adaptés
- Implémentation de retry logic intelligent
- Circuit breakers pour la protection des systèmes
- Keep-alive et gestion des connexions persistantes

**Exemple** : Netflix utilise des circuit breakers (Hystrix) pour isoler les défaillances de microservices et éviter les effets en cascade.

### **Communication temps réel**

Les besoins en temps réel sont omniprésents dans les applications modernes :

- **Polling** : solution simple mais inefficace
- **Long-polling** : amélioration du polling classique
- **Server-Sent Events (SSE)** : streaming unidirectionnel sur HTTP
- **WebSockets** : communication bidirectionnelle full-duplex

**Cas d'usage** : Une application collaborative comme Google Docs utilise WebSockets pour synchroniser les modifications entre utilisateurs en temps réel.

### **Architectures de communication modernes**

Le paysage des architectures d'API a évolué :

- **REST** : architecture stateless basée sur HTTP
- **gRPC** : RPC moderne utilisant HTTP/2 et Protocol Buffers
- **GraphQL** : langage de requête flexible

Chaque approche a ses forces et ses implications réseau que nous explorerons.

**Exemple** : Google utilise gRPC massivement en interne pour les communications entre microservices grâce à ses performances supérieures à REST classique.

### **Optimisation et performance**

Nous aborderons également :

- **Sérialisation efficace** : impact de JSON vs binaire sur la bande passante
- **Compression** : quand et comment compresser les données
- **Mise en cache** : headers HTTP, stratégies de cache
- **Connection pooling** : réutilisation des connexions

## Prérequis pour ce module

Pour tirer le meilleur parti de ce module, vous devriez :

✅ **Avoir compris les modules précédents**, notamment :
- La couche Transport (TCP/UDP)
- Les concepts de ports et sockets
- La couche Application (HTTP, DNS)

✅ **Avoir des bases en programmation** dans au moins un langage (Python, JavaScript, Go, Java, C/C++)

✅ **Comprendre les concepts de concurrence** : threads, processus, asynchronisme

## Ce que vous saurez faire après ce module

À l'issue de ce module, vous serez capable de :

🎯 **Implémenter des clients et serveurs réseau** en utilisant l'API Socket

🎯 **Choisir le bon protocole** (TCP vs UDP) selon le cas d'usage

🎯 **Concevoir des applications résilientes** capables de gérer les défaillances réseau

🎯 **Optimiser les performances** en comprenant les implications réseau de vos choix

🎯 **Utiliser les patterns modernes** (WebSockets, gRPC, SSE) à bon escient

🎯 **Déboguer efficacement** les problèmes réseau dans vos applications

🎯 **Prendre des décisions architecturales éclairées** pour vos systèmes distribués

## Approche pédagogique

Ce module adopte une approche **pratique et orientée cas d'usage** :

- **Concepts expliqués avec du code** dans plusieurs langages
- **Cas d'usage réels** tirés de l'industrie
- **Comparaisons critiques** entre différentes approches
- **Focus sur les pièges courants** et comment les éviter
- **Perspectives modernes** (cloud, microservices, conteneurs)

## Pour qui est ce module ?

Ce module s'adresse particulièrement aux :

👨‍💻 **Développeurs backend** qui créent des API et services réseau

👩‍💻 **Ingénieurs DevOps** qui doivent comprendre les interactions réseau

👨‍💻 **Développeurs fullstack** qui veulent maîtriser la stack complète

👩‍💻 **Architectes système** qui conçoivent des systèmes distribués

👨‍💻 **Développeurs mobile** qui optimisent les communications réseau

## Langages utilisés dans les exemples

Les exemples de code de ce module utilisent principalement :

- **Python** : syntaxe claire, excellent pour l'apprentissage
- **JavaScript/Node.js** : modèle asynchrone event-driven
- **Go** : concurrence native, performant pour les services réseau

Des exemples dans d'autres langages seront fournis en annexe.

## Organisation des sections

Le module est organisé en progression logique :

1. **Fondamentaux** (sections 8.1-8.4) : API Socket, TCP, UDP, gestion d'erreurs
2. **I/O avancés** (sections 8.5-8.7) : modes I/O, multiplexage, async
3. **Résilience** (section 8.8) : timeouts, retry, circuit breakers, keep-alive
4. **Temps réel** (section 8.9) : polling, SSE, WebSockets
5. **Architectures modernes** (sections 8.10-8.14) : REST, gRPC, GraphQL, microservices

Chaque section s'appuie sur les précédentes tout en restant suffisamment autonome pour être consultée indépendamment.

## Fil conducteur : construire un service de messagerie

Pour illustrer les concepts de manière cohérente, nous utiliserons comme **fil conducteur** la construction progressive d'un service de messagerie instantanée :

- Section 8.1-8.3 : Échange de messages simples en TCP/UDP
- Section 8.4-8.7 : Gestion de multiples clients simultanés
- Section 8.8 : Ajout de résilience face aux défaillances
- Section 8.9 : Passage au temps réel avec WebSockets
- Section 8.10-8.11 : Évolution vers une architecture REST/gRPC
- Section 8.14 : Décomposition en microservices

Ce cas d'usage vous permettra de voir l'évolution naturelle d'une application réseau et les décisions architecturales à chaque étape.

## Ressources complémentaires

En plus de ce module, vous trouverez :

- 📚 **Annexe E** : Exemples de code complets dans plusieurs langages
- 📖 **Glossaire** : Termes spécifiques à la programmation réseau
- 🔗 **Références RFC** : Spécifications des protocoles
- 📊 **Diagrammes** : Illustrations des architectures et flux de données

---

**Prêt à plonger dans la programmation réseau ?** Commençons par les fondamentaux avec l'API Socket dans la section suivante.

⏭️ [L'API Socket : concepts fondamentaux](/08-programmation-reseau/01-api-socket-concepts.md)
