🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.1 Méthodologie de diagnostic réseau

## Introduction

Face à un problème réseau, la tentation est grande de se précipiter sur Wireshark ou de taper frénétiquement des commandes en espérant tomber sur la solution. Cette approche improvisée conduit généralement à perdre du temps, à négliger des pistes importantes, et parfois même à aggraver le problème.

Une méthodologie de diagnostic structurée est la clé pour résoudre efficacement les problèmes réseau. Elle permet de :
- **Gagner du temps** en évitant les tentatives aléatoires
- **Identifier la cause racine** plutôt que les symptômes
- **Communiquer efficacement** avec les équipes concernées
- **Documenter** pour éviter que le problème ne se reproduise
- **Maintenir son calme** même sous pression

Cette section présente une approche méthodique éprouvée, inspirée des meilleures pratiques de l'industrie et applicable à tout environnement, du petit réseau local au datacenter d'entreprise.

---

## Le cadre général : les 6 étapes du diagnostic

Tout diagnostic réseau efficace suit un processus en six étapes :

```
1. DÉFINIR le problème
   ↓
2. COLLECTER les informations
   ↓
3. ANALYSER les symptômes
   ↓
4. FORMULER des hypothèses
   ↓
5. TESTER les hypothèses
   ↓
6. RÉSOUDRE et DOCUMENTER
```

Chacune de ces étapes est essentielle. Sauter une étape ou les exécuter dans le désordre compromet l'efficacité du diagnostic.

---

## Étape 1 : Définir le problème

### Pourquoi c'est crucial

Un problème mal défini conduit à des recherches inefficaces. "Le réseau ne marche pas" n'est pas une définition de problème, c'est un symptôme vague qui peut avoir des centaines de causes.

### Les bonnes questions

Posez systématiquement ces questions :

#### **Qui est affecté ?**
- Un seul utilisateur ou tous ?
- Un département spécifique ?
- Une localisation géographique ?
- Un type de terminal (mobiles, PC, serveurs) ?

**Exemple :**
```
❌ Vague : "Les utilisateurs ne peuvent pas accéder à l'application"
✅ Précis : "Les 50 utilisateurs du bureau de Lyon ne peuvent pas accéder
            à l'application CRM depuis leurs PC Windows, mais les utilisateurs
            de Paris y accèdent normalement"
```

#### **Quoi exactement ne fonctionne pas ?**
- Quelle application/service est concerné ?
- Quelle fonctionnalité spécifique échoue ?
- Quel message d'erreur apparaît (verbatim) ?

**Exemple :**
```
❌ Vague : "Le site web est lent"
✅ Précis : "La page de checkout affiche 'ERR_CONNECTION_TIMED_OUT' après
            30 secondes, uniquement sur HTTPS (port 443). HTTP (port 80)
            fonctionne normalement"
```

#### **Quand le problème se produit-il ?**
- En permanence ou de manière intermittente ?
- À certaines heures de la journée ?
- Depuis quand exactement ?
- Y a-t-il un pattern temporel ?

**Exemple :**
```
❌ Vague : "Parfois l'API ne répond pas"
✅ Précis : "Depuis lundi 14h, l'API répond avec HTTP 504 Gateway Timeout
            tous les jours entre 9h et 11h (heures de pointe).
            Le reste de la journée, tout fonctionne normalement"
```

#### **Où le problème se situe-t-il ?**
- Sur le réseau local ou distant ?
- Entre quels points (client → serveur) ?
- Sur quel segment réseau ?

#### **Le problème est-il nouveau ?**
- Cela a-t-il déjà fonctionné auparavant ?
- Y a-t-il eu un changement récent (configuration, mise à jour, nouveau déploiement) ?

### Formaliser le problème

Rédigez une déclaration de problème en une phrase claire :

**Template :**
```
[QUI] ne peut pas [QUOI] [QUAND],
alors que [COMPORTEMENT ATTENDU]
```

**Exemples :**

```
Problème 1 :
Les utilisateurs du VLAN 10 ne peuvent pas résoudre les noms de domaine
externes depuis ce matin 8h, alors que la résolution DNS fonctionnait
hier encore.

Problème 2 :
Le serveur d'application backend (10.0.5.20) ne peut pas établir de
connexion TCP vers la base de données PostgreSQL (10.0.8.50:5432)
depuis le déploiement d'hier 18h, alors que la connexion était stable
depuis 3 mois.

Problème 3 :
Les clients mobiles iOS perdent leur connexion WebSocket toutes les
5 minutes exactement, depuis la mise à jour du reverse proxy nginx
ce matin, alors que les clients Android restent connectés normalement.
```

---

## Étape 2 : Collecter les informations

### Informations de contexte

Avant même de toucher à un outil, rassemblez le contexte :

#### **Architecture réseau**
- Schéma du réseau (même basique)
- Adresses IP impliquées
- Équipements intermédiaires (routeurs, firewalls, proxys)
- Segmentation (VLANs, sous-réseaux)

#### **Configuration actuelle**
- Adressage IP des machines concernées
- Configuration DNS
- Routes configurées
- Règles de firewall applicables

#### **Historique**
- Logs système récents
- Changements récents (change log)
- Incidents similaires passés
- Tickets de support liés

### Collecte technique initiale

Rassemblez les données de base sur les machines concernées :

**Sur le client (machine affectée) :**
```bash
# Configuration réseau
ip addr show          # Linux
ipconfig /all         # Windows
ifconfig -a           # macOS

# Table de routage
ip route              # Linux
route print           # Windows
netstat -rn           # macOS

# Serveurs DNS configurés
cat /etc/resolv.conf  # Linux/macOS
ipconfig /all         # Windows

# Connexions actives
ss -tuln              # Linux (ou netstat -tuln)
netstat -ano          # Windows
```

**Sur le serveur (machine cible) :**
```bash
# État du service
systemctl status nginx        # Linux
Get-Service -Name "W3SVC"    # Windows PowerShell

# Ports en écoute
ss -tlnp | grep :443          # Linux
netstat -ano | findstr :443   # Windows

# Logs récents
journalctl -u nginx -n 50     # Linux
Get-EventLog -LogName System  # Windows
```

### Créer une timeline

Établissez une chronologie des événements :

```
Timeline Exemple :

Hier 17:45  - Déploiement nouvelle version de l'application
Hier 18:00  - Redémarrage du serveur web
Hier 18:15  - Premiers signalements d'erreurs HTTP 502
Hier 18:30  - 25% des requêtes en échec selon monitoring
Aujourd'hui 09:00 - 60% des requêtes en échec
Aujourd'hui 09:15 - Investigation démarrée
```

---

## Étape 3 : Analyser les symptômes

### Utiliser le modèle OSI comme guide

Le modèle OSI fournit un cadre parfait pour analyser systématiquement les symptômes :

```
Couche 7 - Application    │ Problème dans l'application elle-même ?
Couche 6 - Présentation   │ Problème de chiffrement/encodage ?
Couche 5 - Session        │ Problème d'établissement de session ?
──────────────────────────┼────────────────────────────────────────
Couche 4 - Transport      │ Problème TCP/UDP (ports, connexions) ?
Couche 3 - Réseau         │ Problème IP (routage, adressage) ?
Couche 2 - Liaison        │ Problème Ethernet (switching, VLAN) ?
Couche 1 - Physique       │ Problème câble/carte réseau ?
```

### Questions par couche

#### **Couche 1 - Physique**
- Les câbles sont-ils branchés ?
- Les LEDs des cartes réseau sont-elles allumées ?
- Y a-t-il des erreurs CRC sur les interfaces ?

**Vérification :**
```bash
# Linux : vérifier l'état du lien
ethtool eth0 | grep "Link detected"

# Statistiques d'erreurs
ethtool -S eth0 | grep -i error
```

#### **Couche 2 - Liaison**
- L'adresse MAC de destination est-elle accessible ?
- Le bon VLAN est-il configuré ?
- Y a-t-il des problèmes de spanning tree ?

**Vérification :**
```bash
# Afficher le cache ARP
arp -a                    # Tous systèmes
ip neigh show            # Linux moderne

# Vérifier le VLAN (sur un switch)
show vlan brief          # Cisco
```

#### **Couche 3 - Réseau**
- L'adresse IP est-elle correcte ?
- La passerelle est-elle joignable ?
- Le routage est-il configuré ?

**Vérification :**
```bash
# Tester la passerelle
ping 192.168.1.1

# Vérifier la table de routage
ip route get 8.8.8.8     # Linux : voir quelle route serait utilisée
```

#### **Couche 4 - Transport**
- Le port est-il ouvert sur le serveur ?
- Y a-t-il un firewall qui bloque ?
- La connexion TCP s'établit-elle ?

**Vérification :**
```bash
# Tester l'ouverture d'un port TCP
telnet 10.0.5.20 443
nc -zv 10.0.5.20 443     # netcat
curl -v telnet://10.0.5.20:443

# Voir les connexions établies
ss -tn state established
```

#### **Couches 5-7 - Session/Présentation/Application**
- Le protocole applicatif fonctionne-t-il ?
- Les certificats TLS sont-ils valides ?
- L'authentification réussit-elle ?

**Vérification :**
```bash
# Tester HTTP
curl -I https://example.com

# Tester HTTPS avec détails TLS
openssl s_client -connect example.com:443 -servername example.com

# Tester DNS
dig example.com
nslookup example.com
```

### Classification des symptômes

Catégorisez le symptôme observé :

**Problème de connectivité pure**
```
Symptôme : Aucun paquet n'atteint la destination
Couches probables : 1, 2, 3 (Physique, Liaison, Réseau)
```

**Problème de service**
```
Symptôme : Le serveur est joignable mais le service ne répond pas
Couches probables : 4, 7 (Transport, Application)
```

**Problème de performance**
```
Symptôme : Ça fonctionne mais c'est lent
Toutes les couches peuvent être impliquées
```

**Problème intermittent**
```
Symptôme : Ça marche parfois, parfois non
Peut indiquer : saturation, timeout, problème de MTU, load balancing
```

---

## Étape 4 : Formuler des hypothèses

### Principe de l'hypothèse

Une hypothèse est une explication **testable** du problème. Elle doit être :
- **Spécifique** : pointer vers une cause précise
- **Testable** : on doit pouvoir la valider ou l'invalider
- **Fondée** : basée sur les symptômes observés

### Générer des hypothèses

Pour chaque problème, listez 3 à 5 hypothèses plausibles, classées par probabilité.

**Exemple 1 : Connexion impossible à une API**

```
Problème :
L'application web ne peut pas se connecter à l'API backend
(10.0.5.20:8080) depuis ce matin.

Hypothèses (de la plus probable à la moins probable) :

H1. Le firewall bloque le port 8080
    → Test : vérifier les règles de firewall, essayer telnet

H2. Le service API n'est pas démarré sur le serveur
    → Test : vérifier le statut du processus sur 10.0.5.20

H3. Une route réseau est manquante après un changement
    → Test : vérifier la table de routage, faire un traceroute

H4. L'IP du serveur API a changé et le DNS n'est pas à jour
    → Test : vérifier la résolution DNS, comparer avec /etc/hosts

H5. Saturation CPU/mémoire sur le serveur API empêchant les connexions
    → Test : vérifier les métriques système du serveur
```

**Exemple 2 : Site web très lent**

```
Problème :
Le site web met 30 secondes à charger alors qu'il était instantané hier.

Hypothèses :

H1. Latence réseau excessive (WAN, ISP)
    → Test : ping, traceroute, mesure du RTT

H2. Serveur web surchargé (trop de connexions)
    → Test : vérifier la charge CPU, nombre de connexions actives

H3. Base de données lente (requêtes non optimisées)
    → Test : logs de requêtes SQL, temps de réponse DB

H4. Problème de MTU causant de la fragmentation
    → Test : ping avec DF flag, vérifier MTU sur le chemin

H5. Cache CDN expiré, tous les clients frappent l'origin
    → Test : vérifier les headers Cache-Control, hit ratio CDN
```

### Utiliser le principe d'Occam

La loi de parcimonie (rasoir d'Occam) s'applique au diagnostic réseau :

> **L'explication la plus simple est généralement la bonne**

Privilégiez les hypothèses simples avant les scénarios complexes :

```
✅ D'abord : "Le câble est débranché"
❌ Pas d'abord : "Il y a une boucle de routage BGP au niveau Tier-1"

✅ D'abord : "Le firewall bloque le port"
❌ Pas d'abord : "Il y a une corruption de paquets due à un bug kernel"
```

### Prioriser les hypothèses

Classez vos hypothèses selon :

1. **Probabilité** : basée sur votre expérience et les symptômes
2. **Impact** : hypothèses critiques en premier
3. **Facilité de test** : si deux hypothèses sont équiprobables, testez d'abord la plus simple

---

## Étape 5 : Tester les hypothèses

### Approches de test

Il existe trois approches principales pour tester les hypothèses :

#### **1. Approche Bottom-Up (de bas en haut)**

On commence par la couche physique et on remonte :

```
Étape 1 : Physique    → Câble branché ? LED allumée ?
Étape 2 : Liaison     → ARP fonctionne ?
Étape 3 : Réseau      → Ping de la passerelle ?
Étape 4 : Transport   → Port ouvert ? TCP connecte ?
Étape 5 : Application → Service répond ?
```

**Avantages :**
- Méthodique et exhaustif
- Ne rate aucune couche
- Bon pour les débutants

**Inconvénients :**
- Peut être long
- Teste parfois des choses qui fonctionnent

**Quand l'utiliser :**
- Problème de connectivité totale
- Nouvelle installation
- Environnement inconnu

#### **2. Approche Top-Down (de haut en bas)**

On commence par l'application et on descend :

```
Étape 1 : Application → L'app répond ? Erreur précise ?
Étape 2 : Transport   → TCP établi ? Timeout ?
Étape 3 : Réseau      → Routage OK ? IP joignable ?
Étape 4 : Liaison     → ARP résolu ?
Étape 5 : Physique    → Lien physique OK ?
```

**Avantages :**
- Rapide pour les problèmes applicatifs
- Cible directement le symptôme

**Inconvénients :**
- Peut négliger les couches basses
- Nécessite de l'expérience

**Quand l'utiliser :**
- Erreur applicative claire (HTTP 500, DNS failure)
- Environnement connu et stable
- Problème récent sur système qui fonctionnait

#### **3. Approche Divide and Conquer (diviser pour régner)**

On teste au milieu et on élimine la moitié du problème :

```
Test initial : Ping du serveur final
  ├─ Succès → Problème dans Transport/Application
  └─ Échec  → Problème dans Réseau/Liaison/Physique
      └─ Test : Ping de la passerelle
          ├─ Succès → Problème de routage/réseau distant
          └─ Échec  → Problème local (liaison/physique)
```

**Avantages :**
- Très efficace, élimine rapidement des pistes
- Moins de tests nécessaires

**Inconvénients :**
- Nécessite une bonne compréhension
- Peut manquer des problèmes subtils

**Quand l'utiliser :**
- Problèmes complexes avec beaucoup de composants
- Contraintes de temps
- Expert avec intuition développée

### Exemple de test méthodique

**Scénario :** Client (192.168.1.100) ne peut pas accéder au serveur web (203.0.113.50:443)

**Approche Bottom-Up :**

```bash
# Couche 1 : Physique
ethtool eth0 | grep "Link detected"
→ Link detected: yes ✅

# Couche 2 : Liaison (vérifier passerelle dans ARP)
ping 192.168.1.1
arp -a | grep 192.168.1.1
→ Réponse, MAC présente ✅

# Couche 3 : Réseau (routage vers destination)
ping 203.0.113.50
→ Request timeout ❌

# Affiner : traceroute pour voir où ça bloque
traceroute 203.0.113.50
→ S'arrête à 192.168.1.1 (passerelle)

# Hypothèse affinée : problème de routage après la passerelle
# ou firewall sur la passerelle

# Couche 4 : Transport (si ping marchait)
telnet 203.0.113.50 443
nc -zv 203.0.113.50 443
→ Connection refused (le port répond mais refuse) ❌

# Couche 7 : Application (si TCP marchait)
curl -v https://203.0.113.50
→ Détails de la négociation TLS et réponse HTTP
```

### Isolation par substitution

Une technique puissante : remplacer un composant suspect :

**Exemple :**
```
Problème : Client A ne peut pas joindre Serveur B

Test 1 : Client C peut-il joindre Serveur B ?
  → Oui → Le problème est sur Client A
  → Non → Le problème pourrait être Serveur B ou le réseau

Test 2 : Client A peut-il joindre Serveur D ?
  → Oui → Le problème est spécifique à Serveur B
  → Non → Le problème est sur Client A ou sa connexion réseau

Par élimination : on isole le composant défaillant
```

### Utiliser les logs

Les logs sont souvent plus révélateurs que les tests actifs :

```bash
# Logs système récents
journalctl -xe --since "1 hour ago"
tail -f /var/log/syslog

# Logs applicatifs
tail -f /var/log/nginx/error.log
tail -f /var/log/apache2/error.log

# Logs firewall
journalctl -u firewalld -n 100
tail -f /var/log/ufw.log
```

**Rechercher spécifiquement :**
- Mots-clés : `error`, `fail`, `timeout`, `refused`, `denied`
- Adresses IP concernées
- Ports concernés
- Timestamps correspondant au problème

### Documenter chaque test

Gardez une trace de ce que vous testez :

```
Tests effectués :

[10:15] ping 203.0.113.50 → timeout
[10:16] ping 192.168.1.1 → OK (5ms)
[10:17] traceroute 203.0.113.50 → bloqué après 192.168.1.1
[10:20] telnet 192.168.1.1 22 → connexion OK
[10:22] vérification firewall → règle DROP pour 203.0.113.0/24
[10:25] ajout règle ACCEPT → ping OK, problème résolu ✅
```

---

## Étape 6 : Résoudre et documenter

### Implémenter la solution

Une fois la cause identifiée :

#### **1. Planifier le changement**
```
- Quel changement exact ?
- Impact sur quoi/qui ?
- Fenêtre de maintenance nécessaire ?
- Plan de rollback si ça échoue ?
```

#### **2. Tester sur un environnement non-prod** (si possible)
```
- Lab / environnement de test
- Valider que la solution fonctionne
- Identifier les effets de bord
```

#### **3. Appliquer en production**
```
- Suivre la procédure de change management
- Appliquer le changement
- Vérifier immédiatement que ça fonctionne
```

#### **4. Valider la résolution**
```
- Le symptôme initial a-t-il disparu ?
- Y a-t-il des effets de bord ?
- Les utilisateurs confirment-ils ?
- Les métriques sont-elles revenues à la normale ?
```

### Documenter la résolution

La documentation est **critique** pour :
- Éviter que le problème ne se reproduise
- Partager la connaissance avec l'équipe
- Créer une base de connaissances

**Template de documentation :**

```markdown
# Incident #12345 - Impossibilité d'accès à l'API de production

## Résumé
Date : 2025-12-07
Durée : 45 minutes (10:15 - 11:00)
Gravité : Haute
Impact : 200 utilisateurs, service API indisponible

## Symptômes
- Application web retournait HTTP 504 Gateway Timeout
- Logs montrant "Connection refused" vers 10.0.5.20:8080
- Monitoring alertant sur 100% d'échec des health checks

## Cause racine
Règle de firewall trop restrictive ajoutée lors du déploiement
du 06/12 bloquant le trafic vers le port 8080 depuis le subnet
de l'application (10.0.3.0/24).

## Investigation
1. [10:15] Vérification connectivité : ping OK, telnet port 8080 KO
2. [10:20] Traceroute : trafic bloqué au niveau du firewall (10.0.1.1)
3. [10:25] Vérification règles firewall : règle DROP identifiée
4. [10:35] Logs firewall confirmant les DROP packets

## Résolution
Ajout de la règle firewall autorisant 10.0.3.0/24 → 10.0.5.20:8080
Commande : iptables -I INPUT -s 10.0.3.0/24 -p tcp --dport 8080 -j ACCEPT

## Prévention
- Ajouter cette règle au playbook Ansible firewall
- Améliorer les tests pré-déploiement (inclure connectivity tests)
- Mettre à jour la documentation d'architecture réseau

## Leçons apprises
- Les changements firewall doivent inclure une checklist de validation
- Besoin d'un monitoring plus fin au niveau firewall (logs DROP)
```

### Analyse post-mortem

Pour les incidents majeurs, organisez une réunion post-mortem :

**Objectifs :**
- Comprendre **pourquoi** c'est arrivé (pas **qui** est responsable)
- Identifier les améliorations de processus
- Partager les apprentissages

**Questions à se poser :**
```
1. Qu'est-ce qui s'est bien passé ?
2. Qu'est-ce qui s'est mal passé ?
3. Qu'avons-nous eu de la chance de... ?
4. Comment éviter que cela se reproduise ?
5. Quelles améliorations apporter aux outils/processus ?
```

---

## Stratégies avancées

### La règle du 80/20 (Pareto)

En diagnostic réseau, 80% des problèmes viennent de 20% des causes :

**Les classiques qui causent 80% des problèmes :**
1. **Firewall/ACL mal configuré** (bloque le trafic légitime)
2. **DNS qui ne résout pas** (mauvaise config, serveur down)
3. **Mauvaise route** (table de routage incorrecte)
4. **Service pas démarré** (crash, pas d'auto-restart)
5. **Problème physique** (câble débranché, carte réseau HS)
6. **Saturation** (bande passante, connexions max atteintes)

**Checkez toujours ces 6 points en premier** avant de chercher des causes exotiques.

### Le principe des 5 Pourquoi

Creusez jusqu'à la cause racine :

```
Problème : Le site web est down

Pourquoi ? → Le serveur web ne répond pas
Pourquoi ? → Le processus nginx est arrêté
Pourquoi ? → Il a crashé à cause d'un out-of-memory
Pourquoi ? → Une connexion a leaked et consommé toute la RAM
Pourquoi ? → Le keep-alive timeout est trop long et accumule les connexions

Cause racine : Configuration keep-alive inadaptée au trafic
Solution : Ajuster timeout + monitoring proactif de la RAM
```

### Élimination par bisection

Pour les problèmes complexes avec beaucoup d'étapes :

```
Processus en 10 étapes : A → B → C → D → E → F → G → H → I → J

Au lieu de tester A, puis B, puis C... (10 tests max)

Testez au milieu (E) :
  ├─ OK → Problème entre F et J (testez H)
  └─ KO → Problème entre A et E (testez C)

Avec cette méthode : ~log₂(10) = 4 tests maximum
```

### Le changement comme indice

Un problème qui apparaît soudainement est **presque toujours** lié à un changement récent :

**Cherchez les changements des dernières 24-48h :**
- Déploiement applicatif
- Mise à jour système (apt upgrade, yum update)
- Modification de configuration (nginx, firewall, DNS)
- Ajout/retrait d'équipement réseau
- Changement de fournisseur/ISP
- Migration vers nouveau datacenter

**Question clé :** "Qu'est-ce qui a changé ?"

### Reproduire le problème

Si possible, reproduisez le problème de manière contrôlée :

**Avantages :**
- Confirme que vous comprenez la cause
- Permet de tester la solution
- Facilite le debugging (logs, captures)

**Méthode :**
```
1. Isoler un environnement de test
2. Recréer les conditions (même config, même trafic)
3. Observer le problème se produire
4. Appliquer le fix
5. Vérifier que le problème disparaît
```

---

## Erreurs courantes à éviter

### ❌ Erreur 1 : Changer plusieurs choses à la fois

```
Mauvais :
"Je vais redémarrer le serveur, changer la config firewall,
et mettre à jour nginx, comme ça on verra bien"

Bon :
"Je vais d'abord vérifier le firewall. Si ce n'est pas ça,
je testerai ensuite le serveur."
```

**Pourquoi ?** Si vous changez 3 choses et que ça marche, vous ne saurez pas laquelle a résolu le problème.

### ❌ Erreur 2 : Ignorer les logs

Les logs contiennent souvent LA réponse, mais beaucoup ne les consultent qu'en dernier recours.

```
✅ Workflow correct :
1. Définir le problème
2. Consulter les logs
3. Tester hypothèses basées sur les logs
```

### ❌ Erreur 3 : Partir du principe que c'est complexe

```
Syndrome de l'expert : "Ce doit être un problème de BGP
dans le backbone de notre ISP"

Réalité : Le câble Ethernet était mal branché.
```

### ❌ Erreur 4 : Ne pas documenter

```
Scénario :
"J'ai résolu le problème en changeant un truc...
mais je ne me souviens plus quoi exactement"

3 mois plus tard : Le même problème se reproduit,
et personne ne sait comment le résoudre.
```

### ❌ Erreur 5 : Négliger la communication

```
Mauvais :
[Silence pendant 2h d'investigation]
Résultat : Utilisateurs frustrés, management stressé

Bon :
[Update toutes les 30 min]
"Investigation en cours, 3 hypothèses testées,
estimons résolution dans 45 minutes"
```

---

## Checklist diagnostic rapide

Voici une checklist condensée pour un diagnostic rapide :

### ✅ Phase 1 : Définition (5 min)
```
□ Problème défini en 1 phrase claire
□ Qui/Quoi/Quand/Où identifié
□ Symptôme vs comportement attendu noté
□ Timeline des événements établie
```

### ✅ Phase 2 : Collecte (10 min)
```
□ Config réseau récupérée (IP, DNS, routes)
□ Logs récents consultés
□ Changements récents identifiés
□ Architecture comprise (schéma)
```

### ✅ Phase 3 : Tests de base (10 min)
```
□ Ping local (127.0.0.1) → Pile TCP/IP fonctionne
□ Ping passerelle → Réseau local OK
□ Ping destination → Routage OK
□ DNS lookup → Résolution de noms OK
□ Port check (telnet/nc) → Service accessible
```

### ✅ Phase 4 : Hypothèses (5 min)
```
□ 3-5 hypothèses listées
□ Classées par probabilité
□ Tests définis pour chaque hypothèse
```

### ✅ Phase 5 : Tests (temps variable)
```
□ Tests exécutés un par un
□ Résultats documentés
□ Hypothèses éliminées ou confirmées
```

### ✅ Phase 6 : Résolution (temps variable)
```
□ Solution identifiée et testée
□ Changement appliqué
□ Validation effectuée
□ Documentation rédigée
□ Post-mortem si nécessaire
```

---

## Cas pratique complet

### Scénario réel

**Contexte :**
Vous êtes développeur backend. Votre application Node.js ne peut plus se connecter à la base de données PostgreSQL depuis 10 minutes. C'est la panique : le site de production est down.

**Symptôme initial :**
```
Application logs:
Error: connect ETIMEDOUT
    at Connection._handleConnectTimeout
Database: db.prod.example.com:5432
```

### Application de la méthodologie

#### **Étape 1 : Définir le problème**

```
Problème :
L'application Node.js (10.0.3.15) ne peut pas établir de connexion
TCP vers la base de données PostgreSQL (db.prod.example.com / 10.0.8.50:5432)
depuis 10 minutes, alors que ça fonctionnait parfaitement depuis 3 mois.

Contexte :
- Seule cette application est affectée
- Erreur : ETIMEDOUT (timeout de connexion)
- Pas de changement applicatif récent
- Déploiement infrastructure hier soir (mise à jour firewall)
```

#### **Étape 2 : Collecter les informations**

```bash
# Sur le serveur d'application (10.0.3.15)
ip addr show
# → eth0: 10.0.3.15/24

ip route
# → default via 10.0.3.1

cat /etc/resolv.conf
# → nameserver 10.0.1.10

# Logs app
tail -100 /var/log/app/error.log
# → Répétition de "connect ETIMEDOUT" toutes les 30s
```

**Timeline :**
```
Hier 22:00 - Mise à jour firewall planifiée
Aujourd'hui 14:35 - Première alerte monitoring DB timeout
Aujourd'hui 14:40 - Site en erreur 500
Aujourd'hui 14:45 - Investigation démarrée
```

#### **Étape 3 : Analyser les symptômes**

```
Erreur ETIMEDOUT signifie :
- Les paquets SYN envoyés mais pas d'ACK reçu
- Couche 3 (réseau) ou 4 (transport) probablement

Indice fort : Mise à jour firewall hier
```

#### **Étape 4 : Formuler des hypothèses**

```
H1. Firewall bloque le trafic vers port 5432 (PROBABILITÉ HAUTE)
    → Test : telnet, vérifier règles firewall

H2. Serveur PostgreSQL down (PROBABILITÉ MOYENNE)
    → Test : ping serveur, vérifier si service tourne

H3. Route réseau manquante après changement (PROBABILITÉ MOYENNE)
    → Test : traceroute, vérifier table de routage

H4. DNS ne résout plus db.prod.example.com (PROBABILITÉ BASSE)
    → Test : nslookup, dig

H5. PostgreSQL accepte connexions mais pas depuis cette IP (PROBABILITÉ BASSE)
    → Test : vérifier pg_hba.conf
```

#### **Étape 5 : Tester les hypothèses**

```bash
# Test H4 d'abord (le plus rapide)
nslookup db.prod.example.com
# → 10.0.8.50 ✅ DNS fonctionne

# Test H2 : Serveur up ?
ping 10.0.8.50
# → Reply from 10.0.8.50: time=2ms ✅ Serveur joignable

# Test H3 : Routage OK ?
traceroute 10.0.8.50
# → Atteint 10.0.8.50 en 3 hops ✅ Routage OK

# Test H1 : Port 5432 ouvert ?
telnet 10.0.8.50 5432
# → Trying 10.0.8.50...
# → [timeout après 30s] ❌ PORT BLOQUÉ !

# Confirmation : essayer depuis un autre serveur
ssh admin@10.0.2.20
telnet 10.0.8.50 5432
# → Connected to 10.0.8.50 ✅ Fonctionne depuis ailleurs

# CONCLUSION : Le firewall bloque 10.0.3.15 → 10.0.8.50:5432
```

**Investigation firewall :**

```bash
# Sur le firewall
sudo iptables -L -n -v | grep 5432
# → REJECT  tcp  --  *   *   10.0.3.0/24  10.0.8.50  tcp dpt:5432

# BINGO ! Règle de rejet ajoutée hier
```

#### **Étape 6 : Résoudre et documenter**

**Solution :**
```bash
# Supprimer la règle erronée
sudo iptables -D INPUT -s 10.0.3.0/24 -d 10.0.8.50 -p tcp --dport 5432 -j REJECT

# Ajouter la règle correcte (ACCEPT)
sudo iptables -I INPUT -s 10.0.3.0/24 -d 10.0.8.50 -p tcp --dport 5432 -j ACCEPT

# Sauvegarder
sudo iptables-save > /etc/iptables/rules.v4
```

**Validation :**
```bash
# Test connexion
telnet 10.0.8.50 5432
# → Connected ✅

# Redémarrer app
systemctl restart node-app

# Vérifier logs
tail -f /var/log/app/error.log
# → Plus d'erreurs ✅

# Vérifier site
curl -I https://prod.example.com
# → HTTP/2 200 ✅
```

**Documentation :**
```markdown
# Incident : Application down - DB inaccessible

**Date :** 2025-12-07 14:35-15:05 (30 min)
**Impact :** Site production indisponible

## Cause
Règle firewall erronée ajoutée lors de la mise à jour du 06/12
bloquant le trafic du subnet application (10.0.3.0/24) vers
PostgreSQL (10.0.8.50:5432).

## Résolution
Suppression de la règle REJECT et ajout de la règle ACCEPT appropriée.

## Prévention
- Automatiser les tests de connectivité post-déploiement
- Ajouter règles firewall au contrôle de version (Infrastructure as Code)
- Peer review obligatoire pour les changements firewall
- Améliorer monitoring : alerter sur connexions DB en échec
```

**Temps total :** 30 minutes, problème résolu méthodiquement.

---

## Points clés à retenir

### 🎯 Les 5 principes d'or

1. **Définissez avant de chercher** : Un problème bien défini est à moitié résolu
2. **Procédez par couches** : Le modèle OSI est votre guide
3. **Testez une chose à la fois** : Changements isolés seulement
4. **Documentez tout** : Vos tests, vos résultats, vos solutions
5. **Cherchez les changements** : 90% des problèmes suivent un changement

### ✅ Ce qu'il faut faire

- Rester calme et méthodique
- Partir du simple vers le complexe
- Utiliser les logs comme première source d'information
- Valider chaque hypothèse par un test
- Communiquer régulièrement sur l'avancement

### ❌ Ce qu'il faut éviter

- Paniquer et tester au hasard
- Partir de l'hypothèse la plus complexe
- Changer plusieurs choses simultanément
- Négliger la documentation
- Travailler en silence sans updates

---

## Conclusion

La méthodologie de diagnostic réseau n'est pas une perte de temps, c'est un **investissement** qui permet de :
- Résoudre les problèmes **plus rapidement**
- Trouver la **vraie cause** plutôt qu'un contournement temporaire
- **Apprendre** de chaque incident pour progresser
- **Communiquer** efficacement avec les équipes
- **Prévenir** les problèmes futurs

Avec de la pratique, cette approche deviendra naturelle. Vous développerez une intuition qui vous permettra d'identifier rapidement où chercher. Mais même les experts les plus aguerris suivent cette méthodologie pour les problèmes complexes.

Dans les sections suivantes, nous allons approfondir chaque outil et technique mentionnés ici, en commençant par les outils en ligne de commande indispensables.

---

**Prochaine section :** 7.2 Outils en ligne de commande

Nous explorerons en détail `ping`, `traceroute`, `netstat`, `dig`, et bien d'autres outils essentiels avec de nombreux exemples pratiques.

⏭️ [Outils en ligne de commande : ping, traceroute, netstat, ss, nslookup, dig](/07-analyse-depannage/02-outils-ligne-commande.md)
