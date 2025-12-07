🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.7 Conteneurs et réseaux (Docker, Kubernetes networking)

## Introduction

Les **conteneurs** ont révolutionné le déploiement d'applications en encapsulant le code et ses dépendances dans des unités isolées et portables. Mais cette isolation pose un défi réseau fondamental : **comment permettre aux conteneurs de communiquer entre eux et avec le monde extérieur ?**

### L'analogie des appartements

**Sans conteneurs (maison traditionnelle) :**
```
Une grande maison = Un serveur physique
  • Toutes les apps partagent le même réseau
  • Conflits de ports possibles
  • Isolation difficile
```

**Avec conteneurs (immeuble) :**
```
Immeuble = Serveur physique
  ├─ Appartement 1 (conteneur) : App A
  ├─ Appartement 2 (conteneur) : App B
  └─ Appartement 3 (conteneur) : App C

Chaque appartement a :
  • Son propre réseau interne (namespaces réseau)
  • Peut communiquer via le réseau de l'immeuble (bridge)
  • Peut avoir un "interphone" vers l'extérieur (port mapping)
```

### Le problème fondamental

```
CHALLENGE RÉSEAU DES CONTENEURS
═══════════════════════════════════════════════════════

1. ISOLATION
   Conteneur A ne doit pas voir le réseau de Conteneur B
   → Namespaces réseau Linux

2. COMMUNICATION
   Conteneur A doit pouvoir appeler Conteneur B
   → Virtual networks, bridges

3. DÉCOUVERTE
   Conteneur A doit trouver Conteneur B (adresse change)
   → DNS intégré, service discovery

4. ACCÈS EXTERNE
   Client externe doit atteindre Conteneur A
   → Port mapping, load balancing

5. SÉCURITÉ
   Contrôler qui peut parler à qui
   → Network policies, firewalls
```

## Docker Networking

### Les modes réseau Docker

Docker propose **5 modes réseau** principaux :

```
┌─────────────────────────────────────────────────────┐
│  1. BRIDGE (défaut)                                 │
│     Conteneurs sur réseau privé virtualisé          │
├─────────────────────────────────────────────────────┤
│  2. HOST                                            │
│     Conteneur utilise directement le réseau hôte    │
├─────────────────────────────────────────────────────┤
│  3. OVERLAY                                         │
│     Réseau multi-hôtes (Docker Swarm)               │
├─────────────────────────────────────────────────────┤
│  4. MACVLAN                                         │
│     Conteneur avec sa propre adresse MAC            │
├─────────────────────────────────────────────────────┤
│  5. NONE                                            │
│     Pas de réseau (isolation totale)                │
└─────────────────────────────────────────────────────┘
```

#### 1. Mode Bridge (par défaut)

Le mode le plus courant. Chaque conteneur obtient une IP sur un réseau privé virtuel.

```
ARCHITECTURE BRIDGE
═══════════════════════════════════════════════════════

Host (eth0: 192.168.1.10)
  │
  └─ docker0 (bridge virtuel: 172.17.0.1)
       │
       ├─ Container 1 (172.17.0.2)
       ├─ Container 2 (172.17.0.3)
       └─ Container 3 (172.17.0.4)

Communication :
  • Container 1 → Container 2 : Direct via bridge
  • Container 1 → Internet : NAT via docker0
  • Internet → Container 1 : Port mapping requis
```

**Exemple pratique :**

```bash
# Créer un réseau bridge custom
docker network create --driver bridge my-app-network

# Inspecter le réseau
docker network inspect my-app-network
# Output :
# {
#     "Name": "my-app-network",
#     "Driver": "bridge",
#     "Subnet": "172.18.0.0/16",
#     "Gateway": "172.18.0.1"
# }

# Lancer des conteneurs sur ce réseau
docker run -d --name web --network my-app-network nginx
docker run -d --name api --network my-app-network python:3.11

# Les conteneurs peuvent communiquer par nom
docker exec web ping api  # ✅ Fonctionne !
```

**Configuration réseau dans un Dockerfile :**

```dockerfile
FROM python:3.11-slim

# L'application écoute sur le port 8000 (interne au conteneur)
EXPOSE 8000

WORKDIR /app
COPY . .

RUN pip install -r requirements.txt

# Le conteneur écoute sur 0.0.0.0 (toutes interfaces)
CMD ["python", "app.py"]
```

```bash
# Lancer avec port mapping
# Mapping : Host:8080 → Container:8000
docker run -d -p 8080:8000 --name myapp myapp:latest

# Accès depuis l'hôte
curl http://localhost:8080  # → Redirigé vers le port 8000 du conteneur
```

**DNS intégré Docker :**

```python
# app.py dans le conteneur "web"
import requests

# Appeler un autre conteneur par son nom (DNS interne)
# Docker résout automatiquement "api" → 172.18.0.3
response = requests.get('http://api:8000/data')

# Pas besoin de connaître l'IP !
# Docker maintient un DNS interne qui mappe :
# api → 172.18.0.3
# db → 172.18.0.4
# cache → 172.18.0.5
```

#### 2. Mode Host

Le conteneur **partage le réseau de l'hôte** directement. Pas d'isolation réseau.

```
MODE HOST
═══════════════════════════════════════════════════════

Host (eth0: 192.168.1.10)
  │
  └─ Container (utilise directement eth0: 192.168.1.10)

Avantages :
  ✅ Performance maximale (pas de NAT)
  ✅ Accès direct aux interfaces hôte

Inconvénients :
  ❌ Pas d'isolation réseau
  ❌ Conflits de ports possibles
  ❌ Moins sécurisé
```

```bash
# Lancer en mode host
docker run --network host nginx

# Nginx écoute directement sur le port 80 de l'hôte
# Pas de port mapping nécessaire
curl http://192.168.1.10  # Accès direct
```

**Cas d'usage :** Monitoring (Prometheus), profiling, applications nécessitant performances réseau maximales.

#### 3. Mode Overlay (Multi-host)

Réseau virtuel **spanning plusieurs hôtes** Docker. Utilisé dans Docker Swarm ou Kubernetes.

```
MODE OVERLAY
═══════════════════════════════════════════════════════

Host 1 (192.168.1.10)           Host 2 (192.168.1.20)
  │                                 │
  └─ overlay network ───────────────┘
       (10.0.0.0/24)
       │                            │
    Container A                  Container B
    (10.0.0.2)                   (10.0.0.3)

Container A peut ping Container B directement !
→ Tunneling VXLAN entre les hôtes
```

```bash
# Créer un réseau overlay (Docker Swarm requis)
docker network create --driver overlay --attachable my-overlay

# Déployer des services sur plusieurs nœuds
docker service create \
  --name web \
  --network my-overlay \
  --replicas 3 \
  nginx

# Les 3 réplicas peuvent être sur des hôtes différents
# mais communiquent comme s'ils étaient sur le même réseau
```

#### 4. Mode Macvlan

Donne une **adresse MAC unique** à chaque conteneur. Apparaît comme un device physique sur le réseau.

```
MODE MACVLAN
═══════════════════════════════════════════════════════

Réseau physique : 192.168.1.0/24

Host (192.168.1.10)
  │
  ├─ Container 1 (192.168.1.100, MAC: aa:bb:cc:dd:ee:01)
  ├─ Container 2 (192.168.1.101, MAC: aa:bb:cc:dd:ee:02)
  └─ Container 3 (192.168.1.102, MAC: aa:bb:cc:dd:ee:03)

Les conteneurs sont "first-class citizens" sur le réseau
```

```bash
# Créer un réseau macvlan
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan-net

# Lancer un conteneur avec IP statique
docker run -d \
  --network macvlan-net \
  --ip 192.168.1.100 \
  nginx
```

**Cas d'usage :** Applications legacy nécessitant adresse MAC dédiée, monitoring réseau, intégration avec infra existante.

#### 5. Mode None

**Aucun réseau**. Isolation totale.

```bash
docker run -d --network none alpine sleep 3600

# Le conteneur n'a aucune interface réseau
docker exec <container> ip addr
# Output :
# 1: lo: <LOOPBACK,UP,LOWER_UP>
#     inet 127.0.0.1/8 scope host lo
# (seulement loopback local)
```

**Cas d'usage :** Sécurité maximale, batch jobs n'ayant pas besoin de réseau.

### Docker Compose Networking

Docker Compose crée automatiquement un réseau pour les services définis.

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Service frontend
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend
      - backend
    depends_on:
      - api

  # Service API
  api:
    build: ./api
    ports:
      - "8000:8000"
    networks:
      - backend
      - database
    environment:
      - DB_HOST=postgres  # Utilise le nom du service
      - REDIS_HOST=redis
    depends_on:
      - postgres
      - redis

  # Base de données
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - database
    # Pas de ports exposés → accessible uniquement en interne

  # Cache
  redis:
    image: redis:7-alpine
    networks:
      - backend

# Définition des réseaux
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
  database:
    driver: bridge
    internal: true  # Pas d'accès Internet (sécurité)

volumes:
  pgdata:
```

**Résultat :** Docker Compose crée automatiquement :

```
Réseau : myapp_frontend (172.20.0.0/16)
  • web (172.20.0.2)

Réseau : myapp_backend (172.21.0.0/16)
  • web (172.21.0.2)
  • api (172.21.0.3)
  • redis (172.21.0.4)

Réseau : myapp_database (172.22.0.0/16) [internal]
  • api (172.22.0.2)
  • postgres (172.22.0.3)
```

**Communication :**

```python
# Dans le conteneur 'api'
import psycopg2
import redis

# Connection à PostgreSQL par nom de service
conn = psycopg2.connect(
    host='postgres',  # Docker résout automatiquement
    database='myapp',
    user='user',
    password='pass'
)

# Connection à Redis par nom de service
cache = redis.Redis(host='redis', port=6379)

# Les noms de services = hostnames DNS
# Docker maintient un DNS interne
```

### Port Mapping avancé

```bash
# Syntaxe : -p [HOST_IP:]HOST_PORT:CONTAINER_PORT[/PROTOCOL]

# Basique : Host:8080 → Container:80
docker run -p 8080:80 nginx

# IP spécifique : Écouter uniquement sur localhost
docker run -p 127.0.0.1:8080:80 nginx

# Plage de ports
docker run -p 8080-8090:8080-8090 myapp

# UDP au lieu de TCP
docker run -p 53:53/udp dns-server

# Plusieurs mappings
docker run \
  -p 80:80 \
  -p 443:443 \
  -p 8080:8080 \
  myapp

# Port aléatoire sur l'hôte (utile pour scaling)
docker run -P nginx  # -P mappe tous les EXPOSE vers ports aléatoires
docker port <container>  # Voir les mappings
```

**Vérification des ports :**

```bash
# Lister les ports d'un conteneur
docker port myapp
# 80/tcp -> 0.0.0.0:8080
# 443/tcp -> 0.0.0.0:8443

# Depuis l'hôte
netstat -tlnp | grep docker
# tcp  0  0 0.0.0.0:8080  0.0.0.0:*  LISTEN  1234/docker-proxy

# Test de connectivité
curl http://localhost:8080
```

### Inspection et debugging réseau Docker

```bash
# Lister tous les réseaux
docker network ls

# Inspecter un réseau
docker network inspect bridge

# Voir les conteneurs sur un réseau
docker network inspect bridge | jq '.[0].Containers'

# Connecter un conteneur à un réseau supplémentaire
docker network connect my-network mycontainer

# Déconnecter
docker network disconnect my-network mycontainer

# Voir les interfaces réseau d'un conteneur
docker exec mycontainer ip addr

# Voir les routes
docker exec mycontainer ip route

# Voir les connexions actives
docker exec mycontainer netstat -tlnp

# Capturer le trafic réseau (tcpdump dans conteneur)
docker exec mycontainer tcpdump -i eth0 -w /tmp/capture.pcap
docker cp mycontainer:/tmp/capture.pcap .
wireshark capture.pcap
```

**Script de diagnostic réseau Docker :**

```python
#!/usr/bin/env python3
import subprocess
import json

def diagnose_container_network(container_name):
    """
    Diagnostic réseau complet d'un conteneur
    """
    print(f"=== Diagnostic réseau pour {container_name} ===\n")

    # 1. Informations générales
    inspect = subprocess.run(
        ['docker', 'inspect', container_name],
        capture_output=True,
        text=True
    )

    if inspect.returncode != 0:
        print(f"Erreur : conteneur {container_name} non trouvé")
        return

    data = json.loads(inspect.stdout)[0]

    # 2. Réseaux attachés
    print("📡 Réseaux attachés :")
    networks = data['NetworkSettings']['Networks']
    for net_name, net_info in networks.items():
        print(f"  • {net_name}")
        print(f"    IP: {net_info['IPAddress']}")
        print(f"    Gateway: {net_info['Gateway']}")
        print(f"    MAC: {net_info['MacAddress']}")

    # 3. Port mappings
    print("\n🔌 Port mappings :")
    ports = data['NetworkSettings']['Ports'] or {}
    if ports:
        for container_port, host_bindings in ports.items():
            if host_bindings:
                for binding in host_bindings:
                    print(f"  • {binding['HostIp']}:{binding['HostPort']} → {container_port}")
    else:
        print("  Aucun port mappé")

    # 4. DNS
    print("\n🌐 Configuration DNS :")
    dns_config = data['HostConfig']
    print(f"  DNS servers: {dns_config.get('Dns', [])}")
    print(f"  DNS search: {dns_config.get('DnsSearch', [])}")

    # 5. Test de connectivité
    print("\n🔍 Tests de connectivité :")

    # Ping gateway
    gateway = list(networks.values())[0]['Gateway']
    ping_gw = subprocess.run(
        ['docker', 'exec', container_name, 'ping', '-c', '1', '-W', '1', gateway],
        capture_output=True
    )
    print(f"  Gateway ({gateway}): {'✅ OK' if ping_gw.returncode == 0 else '❌ FAIL'}")

    # Ping DNS public
    ping_dns = subprocess.run(
        ['docker', 'exec', container_name, 'ping', '-c', '1', '-W', '1', '8.8.8.8'],
        capture_output=True
    )
    print(f"  Internet (8.8.8.8): {'✅ OK' if ping_dns.returncode == 0 else '❌ FAIL'}")

    # DNS resolution
    dns_test = subprocess.run(
        ['docker', 'exec', container_name, 'nslookup', 'google.com'],
        capture_output=True
    )
    print(f"  DNS resolution: {'✅ OK' if dns_test.returncode == 0 else '❌ FAIL'}")

# Usage
if __name__ == '__main__':
    import sys
    if len(sys.argv) != 2:
        print("Usage: python diagnose.py <container_name>")
        sys.exit(1)

    diagnose_container_network(sys.argv[1])
```

## Kubernetes Networking

Kubernetes a un modèle réseau **radicalement différent** de Docker. Il repose sur des principes plus complexes mais plus puissants.

### Principes fondamentaux du réseau Kubernetes

```
MODÈLE RÉSEAU KUBERNETES
═══════════════════════════════════════════════════════

1. Tous les Pods peuvent communiquer avec tous les Pods
   sans NAT (flat network)

2. Tous les Nodes peuvent communiquer avec tous les Pods
   sans NAT

3. L'IP qu'un Pod voit de lui-même est la même IP
   que les autres Pods voient

4. Services : abstraction stable devant des Pods éphémères
```

### Architecture réseau Kubernetes

```
┌─────────────────────────────────────────────────────┐
│  KUBERNETES CLUSTER                                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Node 1 (192.168.1.10)      Node 2 (192.168.1.20)   │
│    │                           │                    │
│    ├─ Pod A (10.244.1.5)       ├─ Pod C (10.244.2.8)│
│    │   └─ Container            │   └─ Container     │
│    │                           │                    │
│    └─ Pod B (10.244.1.6)       └─ Pod D (10.244.2.9)│
│        └─ Container                └─ Container     │
│                                                     │
│  Pod A peut ping Pod C directement (10.244.2.8)     │
│  → Routing assuré par le CNI                        │
└─────────────────────────────────────────────────────┘
```

### CNI (Container Network Interface)

Le **CNI** est le plugin qui implémente le réseau. Kubernetes délègue complètement le réseau au CNI.

**CNI populaires :**

| CNI | Type | Caractéristiques | Usage |
|-----|------|------------------|-------|
| **Calico** | L3 | Network Policies avancées, BGP | Production, sécurité |
| **Cilium** | eBPF | Performance, observabilité | Moderne, performant |
| **Flannel** | Overlay | Simple, facile | Dev, petits clusters |
| **Weave** | Overlay | Encryption native | Sécurité simple |
| **Canal** | Calico+Flannel | Combo réseau+policies | Polyvalent |

**Installation d'un CNI (Calico) :**

```bash
# Installer Calico sur Kubernetes
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Vérifier l'installation
kubectl get pods -n kube-system | grep calico
# calico-node-xxxxx    1/1     Running
# calico-kube-controllers-xxxxx    1/1     Running

# Voir les interfaces réseau créées
ip link show | grep cali
# cali1a2b3c4d5e6f@if4: <BROADCAST,MULTICAST,UP,LOWER_UP>
# (une interface veth par Pod)
```

### Les Pods et leur réseau

Chaque **Pod** obtient sa propre IP unique dans le cluster.

```yaml
# simple-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
      name: http
```

```bash
# Déployer le Pod
kubectl apply -f simple-pod.yaml

# Voir l'IP du Pod
kubectl get pod nginx-pod -o wide
# NAME        READY   STATUS    IP            NODE
# nginx-pod   1/1     Running   10.244.1.5    node1

# Depuis un autre Pod, on peut accéder directement
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
/ # wget -O- http://10.244.1.5
# <!DOCTYPE html>...  ✅ Fonctionne !
```

**Pod multi-conteneurs (même namespace réseau) :**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container
spec:
  containers:
  # Conteneur 1 : Application
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080

  # Conteneur 2 : Sidecar (logs, metrics)
  - name: sidecar
    image: fluent/fluentd
    # Les deux conteneurs partagent localhost !
    # Le sidecar peut accéder à l'app via localhost:8080
```

```python
# Dans le conteneur 'app'
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return "Hello from app"

if __name__ == '__main__':
    # Écoute sur localhost (partagé avec sidecar)
    app.run(host='127.0.0.1', port=8080)
```

```python
# Dans le conteneur 'sidecar'
import requests

# Accès à l'app via localhost (même Pod)
response = requests.get('http://localhost:8080/')
print(response.text)  # "Hello from app"
```

### Services Kubernetes

Les **Services** fournissent une **IP stable** et un **DNS** pour accéder à un ensemble de Pods.

#### 1. ClusterIP (défaut)

Service accessible **uniquement dans le cluster**.

```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx  # Sélectionne les Pods avec ce label
  ports:
  - port: 80        # Port du Service
    targetPort: 80  # Port du Pod
    protocol: TCP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f service-clusterip.yaml

# Voir le Service
kubectl get svc nginx-service
# NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
# nginx-service   ClusterIP   10.96.100.200   <none>        80/TCP

# Le Service est accessible via :
# 1. IP du Service : 10.96.100.200
# 2. DNS : nginx-service.default.svc.cluster.local

# Test depuis un Pod
kubectl run -it --rm test --image=busybox --restart=Never -- sh
/ # wget -O- http://nginx-service
# <!DOCTYPE html>...  ✅ Load-balancé entre les 3 Pods !

/ # nslookup nginx-service
# Server:    10.96.0.10
# Address:   10.96.0.10:53
#
# Name:      nginx-service.default.svc.cluster.local
# Address:   10.96.100.200
```

**Comment ça marche (kube-proxy) :**

```
CLIENT POD
    ↓ (demande: nginx-service:80)
DNS interne (CoreDNS)
    ↓ (résout: 10.96.100.200)
kube-proxy (iptables rules)
    ↓ (load balance vers un Pod backend)
┌─────────┬─────────┬─────────┐
Pod 1     Pod 2     Pod 3
(10.244.1.5) (10.244.1.6) (10.244.2.7)
```

**Voir les règles iptables créées :**

```bash
# Sur un nœud du cluster
sudo iptables-save | grep nginx-service

# Exemple de règles créées :
# -A KUBE-SERVICES -d 10.96.100.200/32 -p tcp -m tcp --dport 80 \
#   -j KUBE-SVC-XXXXX
# -A KUBE-SVC-XXXXX -m statistic --mode random --probability 0.33 \
#   -j KUBE-SEP-POD1  # 33% vers Pod 1
# -A KUBE-SVC-XXXXX -m statistic --mode random --probability 0.50 \
#   -j KUBE-SEP-POD2  # 50% vers Pod 2
# -A KUBE-SVC-XXXXX -j KUBE-SEP-POD3  # Reste vers Pod 3
```

#### 2. NodePort

Service accessible via **un port sur chaque nœud** du cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Port sur chaque nœud (30000-32767)
    protocol: TCP
```

```bash
kubectl apply -f service-nodeport.yaml

kubectl get svc nginx-nodeport
# NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
# nginx-nodeport   NodePort   10.96.100.201   <none>        80:30080/TCP

# Accessible via N'IMPORTE QUEL nœud du cluster
curl http://192.168.1.10:30080  # Node 1
curl http://192.168.1.20:30080  # Node 2
curl http://192.168.1.30:30080  # Node 3
# Tous routent vers les mêmes Pods backend !
```

**Flux de requête :**

```
Client externe
    ↓ (http://node-ip:30080)
Node (kube-proxy)
    ↓ (DNAT vers ClusterIP)
Service ClusterIP
    ↓ (load balance)
Backend Pods
```

#### 3. LoadBalancer

Service avec **load balancer externe** (cloud provider).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f service-loadbalancer.yaml

# Sur un cloud provider (AWS, GCP, Azure)
kubectl get svc nginx-lb
# NAME       TYPE           CLUSTER-IP      EXTERNAL-IP
# nginx-lb   LoadBalancer   10.96.100.202   35.123.45.67

# Accessible publiquement
curl http://35.123.45.67
```

**Architecture (AWS EKS exemple) :**

```
Internet
    ↓
AWS ELB (35.123.45.67)
    ↓
NodePort :30080 sur chaque nœud
    ↓
Service ClusterIP
    ↓
Backend Pods
```

#### 4. ExternalName

Crée un alias DNS vers un service externe.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: mysql.example.com  # DNS externe
```

```python
# Dans un Pod
import pymysql

# Connection via le Service ExternalName
conn = pymysql.connect(
    host='external-database',  # Résolu vers mysql.example.com
    user='user',
    password='pass'
)
```

### Ingress : Routage L7

Les **Ingress** permettent le routage HTTP/HTTPS intelligent.

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx

  tls:
  - hosts:
    - www.example.com
    - api.example.com
    secretName: example-tls

  rules:
  # Routage basé sur le hostname
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80

  - host: api.example.com
    http:
      paths:
      # Routage vers différents backends selon le path
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: api-v1-service
            port:
              number: 8000

      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: api-v2-service
            port:
              number: 8000

      - path: /auth
        pathType: Prefix
        backend:
          service:
            name: auth-service
            port:
              number: 9000
```

```bash
# Installer un Ingress Controller (Nginx)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Appliquer l'Ingress
kubectl apply -f ingress.yaml

# Vérifier
kubectl get ingress
# NAME          CLASS   HOSTS                            ADDRESS
# app-ingress   nginx   www.example.com,api.example.com  35.123.45.67

# Le trafic est routé selon les règles
curl https://www.example.com/         # → frontend-service
curl https://api.example.com/v1/data  # → api-v1-service
curl https://api.example.com/v2/data  # → api-v2-service
curl https://api.example.com/auth     # → auth-service
```

### Network Policies

Les **Network Policies** contrôlent le trafic entre Pods (firewall).

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
  namespace: production
spec:
  # Appliquer à tous les Pods avec le label app=api
  podSelector:
    matchLabels:
      app: api

  # Types de trafic à contrôler
  policyTypes:
  - Ingress
  - Egress

  # Règles INGRESS (trafic entrant)
  ingress:
  # Règle 1 : Accepter le trafic depuis les Pods frontend
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8000

  # Règle 2 : Accepter le trafic depuis l'Ingress Controller
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8000

  # Règles EGRESS (trafic sortant)
  egress:
  # Règle 1 : Autoriser connexion à la DB
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432

  # Règle 2 : Autoriser connexion à Redis
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379

  # Règle 3 : Autoriser DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    - podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53

  # Règle 4 : Autoriser HTTPS vers Internet
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
```

**Politique par défaut : Deny All**

```yaml
# deny-all.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}  # Tous les Pods du namespace
  policyTypes:
  - Ingress
  - Egress
  # Pas de règles → tout est bloqué par défaut
```

```bash
# Appliquer les policies
kubectl apply -f deny-all.yaml
kubectl apply -f network-policy.yaml

# Tester
kubectl run test --image=busybox -it --rm -- sh

# Depuis le Pod 'test', essayer d'accéder à l'API
/ # wget -O- http://api-service:8000
# ❌ Timeout (bloqué par Network Policy)

# Mais depuis un Pod frontend
kubectl run frontend --labels="app=frontend" --image=busybox -it --rm -- sh
/ # wget -O- http://api-service:8000
# ✅ Fonctionne (autorisé par la policy)
```

### DNS interne Kubernetes

Kubernetes maintient un DNS interne (CoreDNS).

```
FORMAT DNS KUBERNETES
═══════════════════════════════════════════════════════

Service DNS :
  <service-name>.<namespace>.svc.cluster.local

Pod DNS :
  <pod-ip-avec-tirets>.<namespace>.pod.cluster.local

Exemple :
  nginx-service.default.svc.cluster.local
  10-244-1-5.default.pod.cluster.local
```

```bash
# Voir les pods CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Config CoreDNS
kubectl get configmap coredns -n kube-system -o yaml
```

**Utilisation du DNS :**

```python
# app.py dans le namespace 'production'
import requests

# Même namespace (production)
response = requests.get('http://api-service:8000')
# Résolu vers : api-service.production.svc.cluster.local

# Autre namespace
response = requests.get('http://auth-service.auth:9000')
# Résolu vers : auth-service.auth.svc.cluster.local

# FQDN complet (toujours fonctionne)
response = requests.get('http://database.data.svc.cluster.local:5432')
```

**Customisation DNS du Pod :**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-dns
spec:
  containers:
  - name: app
    image: myapp:latest

  # Configuration DNS custom
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
    - 8.8.8.8
    - 8.8.4.4
    searches:
    - production.svc.cluster.local
    - svc.cluster.local
    - cluster.local
    options:
    - name: ndots
      value: "2"
```

## Cas d'usage réels

### Cas 1 : Application microservices complète

```yaml
# full-app.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
---
# Frontend (React)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - name: frontend
        image: myapp/frontend:v1
        ports:
        - containerPort: 3000
        env:
        - name: REACT_APP_API_URL
          value: "http://api-service:8000"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: myapp
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
---
# Backend API (Python)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: myapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: api
      tier: backend
  template:
    metadata:
      labels:
        app: api
        tier: backend
    spec:
      containers:
      - name: api
        image: myapp/api:v2
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          value: "postgresql://postgres-service:5432/myapp"
        - name: REDIS_URL
          value: "redis://redis-service:6379"
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: myapp
spec:
  selector:
    app: api
  ports:
  - port: 8000
    targetPort: 8000
---
# Database (PostgreSQL)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: myapp
spec:
  serviceName: postgres-service
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
        tier: database
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: myapp
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: myapp
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None  # Headless service (pour StatefulSet)
---
# Cache (Redis)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        tier: cache
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: myapp
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
---
# Network Policies
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: myapp
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - port: 8000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - port: 5432
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - port: 6379
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    - podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - port: 53
      protocol: UDP
```

### Cas 2 : Service Mesh avec Istio

Istio ajoute une couche réseau avancée avec des sidecars Envoy.

```yaml
# app-with-istio.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  labels:
    istio-injection: enabled  # Auto-inject Envoy sidecar
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
      version: v1
  template:
    metadata:
      labels:
        app: api
        version: v1
    spec:
      containers:
      - name: api
        image: myapp/api:v1
        ports:
        - containerPort: 8000
---
# VirtualService pour routing intelligent
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api-routes
  namespace: myapp
spec:
  hosts:
  - api-service
  http:
  # Canary deployment : 90% v1, 10% v2
  - match:
    - headers:
        x-user-group:
          exact: beta
    route:
    - destination:
        host: api-service
        subset: v2
      weight: 100

  - route:
    - destination:
        host: api-service
        subset: v1
      weight: 90
    - destination:
        host: api-service
        subset: v2
      weight: 10

    # Retry automatique
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure

    # Timeout
    timeout: 10s
---
# DestinationRule pour load balancing et circuit breaking
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api-destination
  namespace: myapp
spec:
  host: api-service
  trafficPolicy:
    loadBalancer:
      consistentHash:
        httpHeaderName: "x-user-id"  # Sticky session

    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100

    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50

  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**Observabilité Istio :**

```bash
# Voir le trafic entre services
kubectl exec -it <pod> -c istio-proxy -- pilot-agent request GET stats

# Metrics Prometheus
kubectl port-forward -n istio-system svc/prometheus 9090:9090

# Distributed tracing (Jaeger)
kubectl port-forward -n istio-system svc/tracing 16686:80

# Service graph (Kiali)
kubectl port-forward -n istio-system svc/kiali 20001:20001
```

## Debugging réseau Kubernetes

### Outils essentiels

```bash
# 1. Pod de debug (Swiss Army Knife)
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- bash

# Dans le pod debug :
# - ping, curl, wget, nc
# - tcpdump, nmap
# - dig, nslookup
# - iperf3
# - etc.

# 2. Exec dans un pod existant
kubectl exec -it <pod-name> -- /bin/bash

# 3. Logs réseau
kubectl logs <pod-name>

# 4. Describe (voir les events)
kubectl describe pod <pod-name>

# 5. Port-forward pour debug local
kubectl port-forward pod/<pod-name> 8080:8080
curl http://localhost:8080
```

### Scénarios de debugging

**Problème 1 : Pod ne peut pas atteindre un Service**

```bash
# 1. Vérifier que le Pod est Running
kubectl get pod <pod-name>

# 2. Vérifier le Service existe
kubectl get svc <service-name>

# 3. Vérifier les endpoints (Pods behind Service)
kubectl get endpoints <service-name>
# Si vide → aucun Pod ne match le selector

# 4. Test DNS
kubectl exec <pod-name> -- nslookup <service-name>

# 5. Test connectivité IP
kubectl exec <pod-name> -- curl http://<service-ip>:<port>

# 6. Vérifier Network Policies
kubectl get networkpolicy
kubectl describe networkpolicy <policy-name>

# 7. Logs CNI
kubectl logs -n kube-system -l k8s-app=calico-node
```

**Problème 2 : Ingress ne route pas correctement**

```bash
# 1. Vérifier Ingress Controller
kubectl get pods -n ingress-nginx

# 2. Vérifier Ingress rules
kubectl get ingress
kubectl describe ingress <ingress-name>

# 3. Logs Ingress Controller
kubectl logs -n ingress-nginx <ingress-controller-pod>

# 4. Test depuis l'intérieur
kubectl run test --image=busybox -it --rm -- sh
/ # wget -O- http://<ingress-controller-svc>

# 5. Vérifier DNS externe
nslookup myapp.example.com

# 6. Vérifier certificats TLS
kubectl get secret <tls-secret> -o yaml
```

**Script de diagnostic complet :**

```python
#!/usr/bin/env python3
import subprocess
import json
import sys

def run_cmd(cmd):
    """Exécuter une commande kubectl"""
    result = subprocess.run(
        cmd.split(),
        capture_output=True,
        text=True
    )
    return result.stdout, result.returncode

def diagnose_service(namespace, service_name):
    """Diagnostic complet d'un Service Kubernetes"""

    print(f"=== Diagnostic pour {namespace}/{service_name} ===\n")

    # 1. Service existe ?
    print("1️⃣  Vérification Service...")
    stdout, code = run_cmd(f"kubectl get svc {service_name} -n {namespace} -o json")

    if code != 0:
        print(f"❌ Service {service_name} n'existe pas dans {namespace}")
        return

    svc = json.loads(stdout)
    print(f"✅ Service trouvé : {svc['spec']['type']}")
    print(f"   ClusterIP : {svc['spec']['clusterIP']}")
    print(f"   Ports : {svc['spec']['ports']}")

    # 2. Endpoints
    print("\n2️⃣  Vérification Endpoints...")
    stdout, code = run_cmd(f"kubectl get endpoints {service_name} -n {namespace} -o json")

    if code == 0:
        ep = json.loads(stdout)
        subsets = ep.get('subsets', [])

        if not subsets:
            print("❌ Aucun endpoint (aucun Pod ne match le selector)")
            print(f"   Selector : {svc['spec']['selector']}")
            return

        total_pods = sum(len(s.get('addresses', [])) for s in subsets)
        print(f"✅ {total_pods} Pod(s) backend trouvé(s)")

        for subset in subsets:
            for addr in subset.get('addresses', []):
                print(f"   • {addr['ip']} ({addr.get('targetRef', {}).get('name', 'unknown')})")

    # 3. Test DNS
    print("\n3️⃣  Test DNS...")
    test_pod = "debug-test-dns"

    # Créer pod de test
    run_cmd(f"kubectl run {test_pod} --image=busybox --restart=Never -n {namespace} -- sleep 3600")

    # Attendre que le pod soit prêt
    import time
    time.sleep(5)

    # Test DNS
    stdout, code = run_cmd(f"kubectl exec {test_pod} -n {namespace} -- nslookup {service_name}")

    if code == 0:
        print("✅ DNS fonctionne")
        print(stdout)
    else:
        print("❌ Échec résolution DNS")

    # Test connectivité
    print("\n4️⃣  Test connectivité...")
    port = svc['spec']['ports'][0]['port']
    stdout, code = run_cmd(f"kubectl exec {test_pod} -n {namespace} -- wget -O- http://{service_name}:{port} --timeout=5")

    if code == 0:
        print("✅ Connectivité OK")
    else:
        print("❌ Échec connectivité")
        print("   Vérifier Network Policies...")

    # Nettoyer
    run_cmd(f"kubectl delete pod {test_pod} -n {namespace}")

    # 5. Network Policies
    print("\n5️⃣  Network Policies applicables...")
    stdout, code = run_cmd(f"kubectl get networkpolicy -n {namespace} -o json")

    if code == 0:
        policies = json.loads(stdout)
        if policies['items']:
            for policy in policies['items']:
                print(f"   • {policy['metadata']['name']}")
        else:
            print("   Aucune Network Policy")

# Usage
if __name__ == '__main__':
    if len(sys.argv) != 3:
        print("Usage: python k8s-diagnose.py <namespace> <service-name>")
        sys.exit(1)

    diagnose_service(sys.argv[1], sys.argv[2])
```

## Best Practices

### Docker

- ✅ **Utiliser des réseaux custom** (pas le bridge par défaut)
- ✅ **Nommer les conteneurs** pour DNS interne
- ✅ **Limiter les ports exposés** (principe du moindre privilège)
- ✅ **Utiliser docker-compose** pour orchestration multi-conteneurs
- ✅ **Éviter le mode host** en production (sauf nécessité absolue)
- ✅ **Monitorer les réseaux** (docker network inspect)

### Kubernetes

- ✅ **Utiliser des Services** (pas d'IP Pods directes)
- ✅ **Implémenter Network Policies** (sécurité)
- ✅ **Définir des Resource Limits** (éviter saturation)
- ✅ **Utiliser des Liveness/Readiness Probes** (santé)
- ✅ **Préférer ClusterIP** + Ingress (vs NodePort/LoadBalancer)
- ✅ **Documenter les dépendances réseau** (qui parle à qui)
- ✅ **Tester les failovers réseau** (chaos engineering)

### Sécurité réseau

```yaml
# Checklist sécurité réseau conteneurs

## Isolation
- [ ] Network Policies en place (deny-all par défaut)
- [ ] Namespaces séparés par environnement
- [ ] Pas de mode host en production
- [ ] Pas de ports privilégiés (< 1024) sans nécessité

## Chiffrement
- [ ] TLS sur tous les endpoints exposés
- [ ] mTLS entre services (Istio/Linkerd)
- [ ] Secrets chiffrés (KMS, Sealed Secrets)

## Monitoring
- [ ] Logs réseau centralisés
- [ ] Alertes sur trafic anormal
- [ ] Network flow monitoring (Cilium Hubble)

## Conformité
- [ ] Pod Security Standards (restricted)
- [ ] Network scan réguliers (Trivy, Falco)
- [ ] Audit des Network Policies
```

## Résumé : Réseaux conteneurs

### Concepts clés

**Docker :**
- **Namespaces réseau** → Isolation
- **Bridge networks** → Communication inter-conteneurs
- **Port mapping** → Accès depuis l'hôte
- **DNS intégré** → Service discovery

**Kubernetes :**
- **Flat network** → Tous les Pods peuvent communiquer
- **Services** → Abstraction stable sur Pods éphémères
- **Ingress** → Routage L7 intelligent
- **Network Policies** → Firewall inter-Pods

### Tableaux de comparaison

| Feature | Docker | Kubernetes |
|---------|--------|------------|
| **Isolation** | Namespaces | Namespaces + CNI |
| **Service Discovery** | DNS interne | Services + CoreDNS |
| **Load Balancing** | Round-robin simple | kube-proxy (iptables/IPVS) |
| **Ingress** | Pas natif | Ingress Controllers |
| **Network Policies** | Pas natif | Natif (via CNI) |
| **Multi-host** | Swarm overlay | CNI (Calico, Cilium, etc.) |

### Troubleshooting rapide

| Symptôme | Docker | Kubernetes |
|----------|--------|------------|
| **Conteneur/Pod injoignable** | `docker network inspect` | `kubectl describe pod` |
| **DNS ne résout pas** | Vérifier nom conteneur | `kubectl exec -- nslookup` |
| **Port non accessible** | `docker port`, mapping correct ? | Service existe ? Endpoints ? |
| **Connexion refusée** | Firewall hôte ? | Network Policy ? |
| **Lenteur réseau** | Mode host ? | CNI performant (Cilium) ? |

## Conclusion

Les réseaux conteneurs sont **fondamentaux** pour déployer des applications modernes. La compréhension de ces concepts vous permet de :

**Développer efficacement :**
- Architecturer des microservices communicants
- Débugger rapidement les problèmes réseau
- Optimiser les performances

**Déployer en confiance :**
- Configurer la sécurité réseau (Network Policies)
- Gérer le trafic (Services, Ingress)
- Assurer la résilience (Health checks, DNS)

**À retenir :**
- Docker → **Simplicité**, bon pour dev/test
- Kubernetes → **Production**, orchestration à grande échelle
- **Services** > IPs directes (toujours)
- **Network Policies** = essentiel en production
- **DNS interne** = votre ami pour service discovery
- **Debugger** avec les bons outils (netshoot, kubectl exec)

**Prochaine étape :** La section suivante explore le **Software-Defined Networking (SDN)**, qui abstrait encore davantage le réseau physique pour plus de flexibilité et d'automatisation.

---


*Dernière mise à jour : Décembre 2025*

⏭️ [Software-Defined Networking (SDN) : introduction](/09-architectures-avancees/08-sdn.md)
