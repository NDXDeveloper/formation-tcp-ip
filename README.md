# 🌐 Formation Complète TCP/IP

![License](https://img.shields.io/badge/License-CC%20BY%204.0-blue.svg)
![Language](https://img.shields.io/badge/Langue-Français-blue.svg)
![Modules](https://img.shields.io/badge/Modules-10-green.svg)
![Level](https://img.shields.io/badge/Niveau-Débutant%20→%20Avancé-orange.svg)

**Comprendre TCP/IP de A à Z — Pour les développeurs qui veulent maîtriser les fondations du réseau.**

![Network Banner](https://img.shields.io/badge/TCP%2FIP-Stack-0066cc?style=for-the-badge&logo=cisco&logoColor=white)

---

## 📖 À propos

Une formation progressive et complète sur la pile TCP/IP, conçue spécialement pour les développeurs. De l'adressage IP aux architectures modernes (REST, gRPC, WebSockets), cette formation vous donne les clés pour comprendre comment vos applications communiquent réellement.

**✨ Ce que vous allez apprendre :**
- 🔍 **Les 4 couches TCP/IP** de bas en haut
- 📡 **IPv4 et IPv6** avec calculs de sous-réseaux
- 🔐 **TCP et UDP** en profondeur (handshake, fenêtres, contrôle de flux)
- 🌍 **Protocoles applicatifs** (HTTP/1-2-3, DNS, DHCP, etc.)
- 💻 **Programmation réseau** avec sockets et patterns de résilience
- 🚀 **Architectures modernes** (REST vs gRPC, WebSockets, microservices)
- 🛠️ **Outils de diagnostic** (Wireshark, tcpdump, netstat)

**Durée estimée :** 20-25 heures • **Format :** Théorique sans exercices pratiques

---

## 🎯 Pourquoi cette formation ?

Parce qu'en tant que développeur, vous :
- Déboguez des problèmes de connexion sans vraiment comprendre ce qui se passe
- Utilisez HTTP, WebSockets ou gRPC sans savoir pourquoi l'un est meilleur que l'autre
- Voyez des erreurs "Connection timeout" ou "Port already in use" sans savoir comment les résoudre
- Voulez comprendre comment vos conteneurs Docker communiquent vraiment
- Devez choisir entre TCP et UDP pour votre nouvelle application

Cette formation vous donne les bases solides pour **comprendre, diagnostiquer et optimiser** vos applications réseau.

---

## 📚 Contenu de la formation

**Consultez le [SOMMAIRE.md](/SOMMAIRE.md) pour la table des matières complète.**

### Les 10 modules

1. **Introduction et fondamentaux** — OSI vs TCP/IP, encapsulation
2. **Couche Accès réseau** — Ethernet, MAC, ARP, VLAN
3. **Couche Internet (IP)** — IPv4/IPv6, routage, NAT, ICMP
4. **Couche Transport** — TCP (handshake, congestion) et UDP
5. **Couche Application** — DNS, DHCP, HTTP/1-2-3, SMTP, SSH
6. **Sécurité** — TLS/SSL, IPsec, VPN, attaques courantes
7. **Analyse et dépannage** — Wireshark, ping, traceroute
8. **Programmation réseau** — Sockets, timeouts, WebSockets, REST vs gRPC
9. **Architectures avancées** — Load balancing, CDN, Kubernetes networking
10. **Évolutions** — QUIC, DoH, Zero Trust, 5G

### Annexes

- **Glossaire** — Tous les termes réseau expliqués
- **Ports standards** — Référence rapide
- **RFC essentielles** — Documents de référence
- **Exemples de code** — Socket en Python, Go, JavaScript

---

## 🚀 Démarrage rapide

### Prérequis

- Connaissances de base en développement
- Un terminal et un navigateur web
- *Optionnel :* Wireshark pour suivre les captures réseau

### Comment utiliser cette formation

```bash
# 1. Clonez le dépôt
git clone https://github.com/NDXDeveloper/formation-tcp-ip.git
cd formation-tcp-ip

# 2. Consultez le sommaire
cat SOMMAIRE.md

# 3. Commencez par le module 1
cd modules/module-01-introduction/
```

---

## 🗺️ Parcours suggéré

| Profil | Modules recommandés | Durée | Objectif |
|--------|---------------------|-------|----------|
| 🌱 **Débutant complet** | 1 → 5 | 10-12h | Comprendre les bases et les protocoles essentiels |
| 🌿 **Dev backend/frontend** | 3, 4, 5, 8 | 8-10h | Maîtriser IP, TCP/UDP, HTTP et sockets |
| 🌳 **DevOps/SRE** | 3, 6, 7, 9 | 8-10h | Sécurité, diagnostic et architectures |
| 🚀 **Architecte** | Tous | 20-25h | Vision complète du stack réseau |

**💡 Conseil :** Lisez le glossaire en parallèle pour clarifier les termes au fur et à mesure.

---

## 📁 Structure du projet

```
formation-tcp-ip/
├── README.md
├── SOMMAIRE.md
├── LICENSE
├── modules/
│   ├── module-01-introduction/
│   ├── module-02-couche-acces-reseau/
│   ├── module-03-couche-internet/
│   ├── module-04-couche-transport/
│   ├── module-05-couche-application/
│   ├── module-06-securite/
│   ├── module-07-analyse-depannage/
│   ├── module-08-programmation-reseau/
│   ├── module-09-architectures-avancees/
│   └── module-10-evolutions-tendances/
└── annexes/
    ├── 01-glossaire.md
    ├── 02-ports-standards.md
    ├── 03-rfc-essentielles.md
    └── 04-exemples-code/
```

---

## ❓ FAQ

**Q : Faut-il des connaissances préalables en réseau ?**
R : Non, la formation part des bases. Des connaissances en développement suffisent.

**Q : Pourquoi pas d'exercices pratiques ?**
R : Cette formation se concentre sur la compréhension théorique approfondie. Les exemples de code dans les annexes permettent d'expérimenter librement.

**Q : Quel module pour comprendre pourquoi mon API est lente ?**
R : Module 4 (TCP), Module 7 (diagnostic) et Module 8 (programmation réseau).

**Q : C'est adapté pour préparer une certification réseau ?**
R : C'est un excellent complément, mais cette formation est orientée développeur plutôt qu'administrateur réseau.

---

## 🛠️ Technologies et outils couverts

**Protocoles :** IPv4 • IPv6 • TCP • UDP • ICMP • ARP • DNS • DHCP • HTTP/1.1 • HTTP/2 • HTTP/3 • QUIC • TLS • IPsec

**Outils :** Wireshark • tcpdump • ping • traceroute • netstat • nslookup • dig • curl

**Concepts modernes :** REST • gRPC • WebSockets • SSE • Microservices • Kubernetes networking • CDN • Load balancing

---

## 📝 Licence

Cette formation est sous licence **Creative Commons Attribution 4.0 International (CC BY 4.0)**.

✅ Vous êtes libre de :
- Partager — copier et redistribuer
- Adapter — remixer, transformer et créer
- Utiliser commercialement

**Attribution requise :**
```
Formation TCP/IP par Nicolas DEOUX
https://github.com/NDXDeveloper/formation-tcp-ip
Licence CC BY 4.0
```

Voir le fichier [LICENSE](/LICENSE) pour les détails complets.

---

## 👨‍💻 À propos de l'auteur

**Nicolas DEOUX**

Développeur passionné par les fondamentaux de l'informatique et le partage de connaissances.

- 📧 [NDXDev@gmail.com](mailto:NDXDev@gmail.com)
- 💼 [LinkedIn](https://www.linkedin.com/in/nicolas-deoux-ab295980/)
- 🐙 [GitHub](https://github.com/NDXDeveloper)

---

## 🙏 Remerciements

Merci aux auteurs des RFC, à la communauté réseau, et à tous ceux qui contribuent à rendre ces connaissances accessibles. Cette formation s'appuie sur des décennies de documentation technique et d'expertise partagée.

**Ressources inspirantes :**
[TCP/IP Illustrated](https://www.amazon.fr/TCP-Illustrated-Vol-Protocols/dp/0201633469) • [RFC Editor](https://www.rfc-editor.org/) • [Cloudflare Learning](https://www.cloudflare.com/learning/) • [High Performance Browser Networking](https://hpbn.co/)

---

<div align="center">

**🌐 Bonne formation et bon apprentissage ! 🌐**

[![Star on GitHub](https://img.shields.io/github/stars/NDXDeveloper/formation-tcp-ip?style=social)](https://github.com/NDXDeveloper/formation-tcp-ip)

**[⬆ Retour en haut](#-formation-complète-tcpip)**

*Dernière mise à jour : Décembre 2025*

</div>

