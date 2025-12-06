🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.1 Rôle et responsabilités de la couche Accès réseau

## Introduction

La couche Accès réseau (également appelée couche Liaison ou couche Interface réseau) est la **fondation du modèle TCP/IP**. C'est elle qui assure le premier contact entre votre ordinateur et le réseau physique. Sans elle, toutes les couches supérieures seraient incapables de communiquer !

Imaginez un service postal : avant que votre lettre ne voyage à travers le pays (couches supérieures), elle doit d'abord être remise au facteur de votre quartier (couche Accès réseau). C'est lui qui connaît les spécificités locales : quelle maison, quelle boîte aux lettres, quel trajet emprunter dans votre rue.

## Position dans le modèle TCP/IP

Dans l'architecture en 4 couches du modèle TCP/IP, la couche Accès réseau occupe la **position la plus basse** :

```
┌─────────────────────────────┐
│   Couche Application        │  ← HTTP, DNS, FTP...
├─────────────────────────────┤
│   Couche Transport          │  ← TCP, UDP
├─────────────────────────────┤
│   Couche Internet           │  ← IP
├─────────────────────────────┤
│   Couche Accès réseau       │  ← Ethernet, Wi-Fi... ★ NOUS SOMMES ICI
└─────────────────────────────┘
         ↓
    Câbles, ondes radio...
```

Cette position est stratégique : **elle fait le pont entre le monde logiciel (les protocoles) et le monde physique (les câbles, les ondes radio)**.

## L'analogie de la route

Pour bien comprendre le rôle de cette couche, pensez à un voyage en voiture :

- **La couche Application** : votre destination finale (ex: "Je veux aller à Paris")
- **La couche Transport** : le type de véhicule choisi (voiture fiable vs scooter rapide)
- **La couche Internet** : l'itinéraire général (autoroutes, départementales)
- **La couche Accès réseau** : les règles de conduite locales et le bitume lui-même

La couche Accès réseau, c'est comme **connaître les règles de circulation spécifiques à votre quartier** : où se garer, dans quel sens rouler, comment éviter les nids-de-poule. Sans ces connaissances locales, votre véhicule ne peut même pas commencer son voyage !

## Responsabilités principales

### 1. **Transmission physique des données**

La couche Accès réseau transforme les données numériques (des 0 et des 1) en **signaux physiques** adaptés au média :
- **Signaux électriques** pour les câbles Ethernet
- **Ondes radio** pour le Wi-Fi
- **Impulsions lumineuses** pour la fibre optique

**Analogie** : C'est comme un traducteur qui transforme votre message écrit (données) en signaux de fumée, signaux lumineux ou cris, selon le moyen de communication disponible.

### 2. **Adressage physique (MAC)**

Chaque équipement réseau possède une **adresse MAC** (Media Access Control) unique, comme une plaque d'immatriculation. La couche Accès réseau utilise ces adresses pour identifier :
- **L'expéditeur** : "Ce paquet vient de la carte réseau 00:1A:2B:3C:4D:5E"
- **Le destinataire** : "Ce paquet est pour la carte réseau AA:BB:CC:DD:EE:FF"

**Exemple** : Dans votre réseau local, votre ordinateur (MAC: `A1:B2:C3:...`) veut envoyer des données à l'imprimante (MAC: `X9:Y8:Z7:...`). La couche Accès réseau s'assure que ces données arrivent à la bonne machine physique.

### 3. **Encadrement des données (framing)**

Les données reçues de la couche Internet sont **encapsulées dans des trames** (frames). Une trame, c'est comme une enveloppe qui contient :
- **L'adresse de destination** (MAC)
- **L'adresse source** (MAC)
- **Les données** elles-mêmes
- **Un code de vérification d'erreur** (pour détecter les corruptions)

```
┌──────────────────────────────────────────────────────┐
│  EN-TÊTE    │    DONNÉES    │  CODE VÉRIFICATION     │
│  (adresses) │   (payload)   │      (CRC/FCS)         │
└──────────────────────────────────────────────────────┘
        ↑              ↑                  ↑
    Qui/Pour qui?   Contenu         Données intactes?
```

### 4. **Détection d'erreurs**

Lors de la transmission physique, des erreurs peuvent survenir (interférences, câble défectueux...). La couche Accès réseau inclut des mécanismes pour **détecter** ces erreurs :
- Calcul d'une somme de contrôle (checksum)
- Si erreur détectée → la trame est rejetée
- ⚠️ **Attention** : elle détecte les erreurs mais ne les corrige pas toujours (la retransmission est souvent gérée par les couches supérieures)

**Analogie** : C'est comme un code-barres sur un colis. Si le code-barres est illisible, le colis est refusé.

### 5. **Contrôle d'accès au média**

Quand plusieurs appareils partagent le même câble ou le même réseau Wi-Fi, il faut éviter qu'ils parlent tous en même temps ! La couche Accès réseau gère :
- **Qui peut parler et quand**
- **Comment détecter les collisions** (deux appareils qui transmettent simultanément)
- **Comment réagir en cas de collision** (attendre un temps aléatoire puis réessayer)

**Analogie** : C'est comme une conversation de groupe où il faut lever la main et attendre son tour pour parler, sinon personne ne se comprend !

### 6. **Conversion de formats**

Selon le type de réseau (Ethernet, Wi-Fi, PPP, etc.), les formats de trames sont différents. La couche Accès réseau s'adapte automatiquement au **type de technologie utilisée**.

## Relation avec le modèle OSI

Le modèle TCP/IP regroupe ce que le modèle OSI sépare en deux couches :

| Modèle TCP/IP | Modèle OSI |
|---------------|------------|
| **Couche Accès réseau** | **Couche 1 : Physique**<br>(Transmission des bits, voltages, fréquences)<br>+<br>**Couche 2 : Liaison de données**<br>(Trames, adresses MAC, contrôle d'erreurs) |

Cette fusion simplifie le modèle mais peut parfois créer de la confusion. Retenez que la couche Accès réseau couvre **à la fois le hardware et son contrôle logique immédiat**.

## Technologies concernées

La couche Accès réseau englobe de nombreuses technologies, chacune avec ses propres règles :

### Technologies filaires
- **Ethernet** (le plus répandu en entreprise et à domicile)
- **Fibre optique**
- **Token Ring** (ancien, rarement utilisé aujourd'hui)

### Technologies sans fil
- **Wi-Fi** (802.11a/b/g/n/ac/ax)
- **Bluetooth**
- **Réseaux cellulaires** (4G, 5G)

### Technologies spécialisées
- **PPP** (Point-to-Point Protocol, pour les connexions dial-up)
- **HDLC** (High-Level Data Link Control)

Chaque technologie a ses propres spécifications, mais toutes remplissent les mêmes responsabilités fondamentales décrites ci-dessus.

## Ce que la couche Accès réseau NE fait PAS

Pour bien comprendre son rôle, il est utile de savoir ce qui **n'est pas** de son ressort :

- ❌ **Routage entre réseaux différents** → C'est le travail de la couche Internet
- ❌ **Garantie de livraison de bout en bout** → C'est le travail de la couche Transport
- ❌ **Interprétation des données applicatives** → C'est le travail de la couche Application

La couche Accès réseau se concentre uniquement sur le **segment local** : faire passer les données d'un appareil à un autre **sur le même réseau physique**.

## Schéma récapitulatif

```
┌────────────────────────────────────────────────────────────┐
│          COUCHE ACCÈS RÉSEAU                               │
├────────────────────────────────────────────────────────────┤
│
│  🎯 Mission principale :
│     Transmettre les données sur le réseau local
│
│  📋 Responsabilités :
│     ✓ Transmission physique (signaux)
│     ✓ Adressage MAC (identifier les machines)
│     ✓ Encadrement en trames
│     ✓ Détection d'erreurs
│     ✓ Contrôle d'accès au média
│     ✓ Adaptation aux technologies (Ethernet, Wi-Fi...)
│
│  🛠️ Technologies :
│     • Ethernet, Wi-Fi, Fibre, PPP...
│
│  🔗 Portée :
│     Réseau local uniquement (pas de routage)
│
└────────────────────────────────────────────────────────────┘
```

## Points clés à retenir

1. **Fondation du modèle** : La couche Accès réseau est la base sur laquelle tout le reste repose.

2. **Pont hardware/software** : Elle fait l'interface entre les bits et les protocoles.

3. **Adresses MAC** : Elle utilise des adresses physiques pour identifier les appareils sur le réseau local.

4. **Spécifique à la technologie** : Chaque type de réseau (Ethernet, Wi-Fi...) a ses propres règles, mais les responsabilités restent les mêmes.

5. **Portée locale** : Elle ne s'occupe que du réseau immédiat, pas de l'acheminement à travers Internet.

---

## Prochaine étape

Maintenant que vous comprenez le rôle global de la couche Accès réseau, nous allons explorer en détail la technologie la plus répandue dans cette couche : **Ethernet**. Vous découvrirez comment elle a révolutionné les réseaux locaux et pourquoi elle domine encore aujourd'hui.

---


⏭️ [Technologies Ethernet : principes et évolution](/02-couche-acces-reseau/02-technologies-ethernet.md)
