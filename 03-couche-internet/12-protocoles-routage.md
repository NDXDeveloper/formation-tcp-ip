🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.12 Protocoles de routage

## Introduction

Imaginez Internet comme un gigantesque réseau routier mondial. Chaque paquet de données qui voyage sur ce réseau est comme une voiture qui doit trouver son chemin d'une ville (adresse IP source) à une autre (adresse IP destination). Mais contrairement à un GPS qui connaît déjà toutes les routes, les routeurs sur Internet doivent continuellement apprendre et échanger des informations sur les meilleurs chemins possibles.

C'est là qu'interviennent les **protocoles de routage** : ce sont les langages et les règles que les routeurs utilisent entre eux pour partager des informations sur la topologie du réseau et déterminer les meilleurs chemins pour acheminer les paquets.

## Qu'est-ce qu'un protocole de routage ?

Un **protocole de routage** est un ensemble de règles qui permettent aux routeurs de :
- **Découvrir** les réseaux disponibles
- **Partager** des informations avec d'autres routeurs
- **Calculer** les meilleurs chemins vers chaque destination
- **Mettre à jour** automatiquement leurs tables de routage
- **S'adapter** aux changements du réseau (panne de lien, nouveau routeur, etc.)

### Analogie : le réseau des bureaux de poste

Pensez aux routeurs comme des bureaux de poste dans un pays :
- Chaque bureau de poste (routeur) connaît les bureaux voisins
- Ils échangent régulièrement des informations sur les destinations qu'ils peuvent atteindre
- Si une route est bloquée (inondation, travaux), ils informent les autres bureaux
- Quand une lettre (paquet) arrive, ils consultent leur registre (table de routage) pour savoir où l'envoyer
- Les bureaux de poste travaillent ensemble pour garantir que le courrier arrive à destination par le chemin le plus efficace

## Pourquoi avons-nous besoin de protocoles de routage ?

### 1. La complexité du réseau

Internet compte des millions de réseaux interconnectés. Il serait impossible pour un administrateur humain de :
- Configurer manuellement chaque routeur
- Maintenir les informations à jour en permanence
- Réagir instantanément aux pannes

### 2. La dynamique du réseau

Le réseau change constamment :
- Des liens tombent en panne
- De nouveaux réseaux sont ajoutés
- La charge du trafic varie
- Des routeurs redémarrent ou sont mis à jour

**Les protocoles de routage automatisent toutes ces tâches** en permettant aux routeurs de s'adapter dynamiquement aux changements.

### 3. L'optimisation des chemins

Pour chaque destination, plusieurs chemins peuvent exister. Les protocoles de routage permettent de :
- Choisir le chemin le plus rapide
- Éviter les chemins congestionnés
- Répartir la charge sur plusieurs liens
- Basculer automatiquement vers un chemin alternatif en cas de panne

## Les objectifs principaux d'un protocole de routage

### 1. Exactitude
Le protocole doit garantir que les paquets arrivent à la bonne destination, sans boucles ni perte de données.

### 2. Convergence rapide
Lorsqu'un changement survient dans le réseau, tous les routeurs doivent rapidement se mettre d'accord sur la nouvelle topologie. Ce processus s'appelle la **convergence**.

**Exemple** : Si un lien tombe en panne, tous les routeurs doivent apprendre cette information et recalculer leurs routes rapidement, idéalement en quelques secondes.

### 3. Efficacité
Le protocole ne doit pas générer trop de trafic de contrôle. Il doit trouver un équilibre entre :
- Échanger suffisamment d'informations pour maintenir la cohérence
- Ne pas surcharger le réseau avec des messages de mise à jour

### 4. Scalabilité
Le protocole doit fonctionner aussi bien dans un petit réseau d'entreprise que dans l'Internet mondial avec des millions de routes.

## Routage statique vs routage dynamique : un premier aperçu

Il existe deux grandes approches pour le routage :

### Routage statique
Les routes sont configurées **manuellement** par un administrateur et ne changent pas automatiquement.

**Analogie** : C'est comme avoir des panneaux routiers fixes qui ne changent jamais, même si une route est bloquée.

**Utilisé pour** :
- Petits réseaux simples
- Connexions point-à-point
- Routes par défaut

### Routage dynamique
Les routeurs **apprennent automatiquement** les routes en utilisant des protocoles de routage.

**Analogie** : C'est comme avoir un GPS connecté qui reçoit des mises à jour en temps réel sur l'état du trafic et les fermetures de routes.

**Utilisé pour** :
- Réseaux moyens à grands
- Topologies complexes
- Environnements nécessitant une haute disponibilité

> 📘 **Note** : Nous approfondirons cette distinction dans la section 3.12.1.

## Les grandes familles de protocoles de routage

Sans entrer dans les détails (qui viendront dans les sections suivantes), il existe deux grandes catégories de protocoles de routage dynamiques :

### 1. Protocoles à vecteur de distance
Ces protocoles fonctionnent un peu comme le bouche-à-oreille :
- Chaque routeur dit à ses voisins : "Je connais ces réseaux, et ils sont à X sauts de moi"
- Les routeurs se fient aux informations de leurs voisins
- Ils ne connaissent pas la topologie complète du réseau

**Exemple** : RIP (Routing Information Protocol)

**Analogie** : Vous demandez votre chemin à un passant qui vous dit : "Le musée ? Allez tout droit pendant 3 rues, puis demandez à quelqu'un d'autre". Vous ne connaissez pas le chemin complet, juste la prochaine étape.

### 2. Protocoles à état de lien
Ces protocoles partagent une vision complète du réseau :
- Chaque routeur construit une carte complète de la topologie
- Ils calculent eux-mêmes le meilleur chemin vers chaque destination
- Plus complexes, mais plus efficaces et rapides à converger

**Exemple** : OSPF (Open Shortest Path First)

**Analogie** : Vous avez une carte complète de la ville. Vous pouvez calculer vous-même le meilleur itinéraire vers n'importe quelle destination.

### 3. Protocoles de routage par chemin (Path Vector)
Une variante utilisée principalement pour le routage entre systèmes autonomes (AS) sur Internet.

**Exemple** : BGP (Border Gateway Protocol)

**Analogie** : Les compagnies aériennes qui échangent des informations sur les routes disponibles entre continents, avec des considérations politiques et commerciales.

## Métriques de routage : comment choisir le "meilleur" chemin ?

Les protocoles de routage utilisent des **métriques** pour déterminer le meilleur chemin. Une métrique est un critère de décision, comme :

- **Nombre de sauts** : Combien de routeurs faut-il traverser ?
- **Bande passante** : Quelle est la capacité du lien ?
- **Délai** : Combien de temps faut-il pour traverser le lien ?
- **Fiabilité** : Quelle est la stabilité du lien ?
- **Coût** : Une valeur administrative arbitraire

**Exemple concret** :
```
Destination : Réseau 192.168.10.0/24

Chemin A : 3 sauts, bande passante 100 Mbps
Chemin B : 2 sauts, bande passante 10 Mbps

Selon la métrique utilisée :
- Si on compte les sauts → Chemin B est meilleur (2 < 3)
- Si on considère la bande passante → Chemin A est meilleur (100 > 10)
```

Le choix de la métrique dépend du protocole de routage utilisé et des besoins du réseau.

## Les défis du routage dynamique

### 1. Les boucles de routage
Une situation où un paquet tourne en rond entre plusieurs routeurs sans jamais atteindre sa destination.

**Prévention** : Les protocoles incluent des mécanismes pour détecter et éviter les boucles (TTL, split horizon, route poisoning, etc.).

### 2. La convergence
Le temps nécessaire pour que tous les routeurs s'accordent sur la topologie du réseau après un changement.

**Enjeu** : Pendant la convergence, certains paquets peuvent être perdus ou mal routés.

### 3. La scalabilité
Plus il y a de réseaux et de routeurs, plus les tables de routage deviennent grandes et les calculs complexes.

**Solution** : Hiérarchisation du réseau, agrégation de routes, utilisation de différents protocoles selon les zones.

### 4. La sécurité
Les protocoles de routage peuvent être vulnérables à des attaques :
- Injection de fausses routes
- Usurpation d'identité de routeur
- Déni de service

**Protection** : Authentification, chiffrement, filtrage des mises à jour.

## Systèmes autonomes (AS) et routage inter-AS

Sur Internet, les réseaux sont organisés en **systèmes autonomes** (AS) :
- Un AS est un ensemble de réseaux sous une administration commune
- Chaque AS a un numéro unique (ASN)
- Les protocoles de routage se divisent en deux catégories :

### IGP (Interior Gateway Protocol)
Protocoles utilisés **à l'intérieur** d'un système autonome :
- RIP
- OSPF
- EIGRP (Cisco propriétaire)

### EGP (Exterior Gateway Protocol)
Protocoles utilisés **entre** systèmes autonomes :
- BGP (le seul protocole EGP moderne)

**Analogie** : Les IGP sont comme le système de courrier interne d'une grande entreprise (entre bureaux), tandis que BGP est comme le système postal national qui relie différentes entreprises et organisations.

## Concepts clés à retenir

1. **Les protocoles de routage automatisent** la découverte et la maintenance des routes dans un réseau
2. **Ils permettent au réseau de s'adapter** automatiquement aux changements et aux pannes
3. **Deux grandes approches** : routage statique (manuel) et routage dynamique (automatique)
4. **Deux grandes familles principales** : vecteur de distance et état de lien
5. **Les métriques déterminent** le "meilleur" chemin selon différents critères
6. **La convergence** est le processus par lequel les routeurs s'accordent sur la topologie
7. **IGP et EGP** servent à différentes échelles de réseau

## Ce qui vient ensuite

Dans les sections suivantes, nous allons explorer en détail :
- **3.12.1** : La différence entre routage statique et dynamique
- **3.12.2** : Les protocoles à vecteur de distance (RIP)
- **3.12.3** : Les protocoles à état de lien (OSPF)
- **3.12.4** : BGP, le protocole qui fait fonctionner Internet
- **3.12.5** : Comment les métriques et la convergence fonctionnent en pratique
- **3.12.6** : Les implications pour les architectures modernes (cloud, microservices)

Ces connaissances vous permettront de comprendre comment les données trouvent leur chemin à travers les réseaux, et comment concevoir des architectures réseau robustes et efficaces.

---

**Points de vigilance pour les débutants** :
- Ne confondez pas "protocole de routage" (qui permet aux routeurs d'échanger des informations) et "routage" (l'action d'acheminer les paquets)
- Les tables de routage sont le résultat du travail des protocoles de routage
- Dans les petits réseaux domestiques, il n'y a souvent qu'un seul routeur, donc pas besoin de protocoles de routage complexes
- Les concepts présentés ici s'appliquent aussi bien aux réseaux d'entreprise qu'à Internet

⏭️ [Routage statique vs dynamique](/03-couche-internet/12.1-routage-statique-dynamique.md)
