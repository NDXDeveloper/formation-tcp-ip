🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.9 Cloud Networking : concepts clés

## Introduction

Le **Cloud Networking** applique les principes du réseau à l'ère du cloud computing : réseaux **virtualisés**, **élastiques**, **programmables** et **distribués mondialement**. C'est le SDN poussé à son paroxysme, avec en plus la **facturation à l'usage** et la **gestion complète** par le provider.

### L'analogie de la ville

**Data center traditionnel (votre propre maison) :**
```
Vous possédez tout :
  • Le terrain (serveurs physiques)
  • Les murs (réseau physique)
  • La plomberie (câbles)
  • L'électricité (alimentation)

Responsabilités :
  ❌ Maintenance complète
  ❌ Coûts fixes élevés
  ❌ Scaling difficile (construire des extensions)
  ❌ Vous gérez les pannes
```

**Cloud (location en ville intelligente) :**
```
Le cloud provider gère :
  ✅ Infrastructure physique invisible
  ✅ Réseau global haute performance
  ✅ Redondance automatique
  ✅ Scaling instantané

Vous configurez :
  • Topologie virtuelle (VPC)
  • Règles de sécurité (firewall)
  • Connectivité (VPN, peering)

Paiement : À l'usage (comme eau/électricité)
```

### Le problème du réseau traditionnel dans le cloud

```
AVANT LE CLOUD
═══════════════════════════════════════════════════════

[Vos serveurs dans votre DC]
  ├─ Réseau privé : 192.168.1.0/24
  ├─ Firewall matériel : $50,000
  ├─ Load balancer F5 : $80,000
  └─ VPN concentrator : $30,000

Pour ajouter un serveur :
  1. Acheter le matériel (semaines)
  2. Installer dans le rack (jours)
  3. Configurer le réseau (heures)
  4. Total : 2-3 semaines

Pour ouvrir un nouveau data center :
  Investissement : $500k - $10M+
  Délai : 6-18 mois


AVEC LE CLOUD
═══════════════════════════════════════════════════════

[VPC dans AWS/GCP/Azure]
  ├─ Réseau virtuel : 10.0.0.0/16
  ├─ Security Groups : Gratuit
  ├─ Load balancer : $0.025/h (~$18/mois)
  └─ VPN : $0.05/h (~$36/mois)

Pour ajouter un serveur :
  1. API call ou clic UI
  2. Total : 60 secondes ⚡

Pour une nouvelle région :
  1. Sélectionner la région
  2. Déployer avec Terraform
  3. Total : 5 minutes ⚡
```

## Virtual Private Cloud (VPC)

Un **VPC** est votre **réseau privé virtuel** dans le cloud, complètement isolé des autres clients.

### Architecture VPC

```
┌─────────────────────────────────────────────────────┐
│  VPC: 10.0.0.0/16 (65,536 IPs)                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌────────────────┐  ┌────────────────┐             │
│  │ AZ-1           │  │ AZ-2           │             │
│  ├────────────────┤  ├────────────────┤             │
│  │ Public Subnet  │  │ Public Subnet  │             │
│  │ 10.0.1.0/24    │  │ 10.0.2.0/24    │             │
│  │ • NAT GW       │  │ • NAT GW       │             │
│  │ • Bastion      │  │ • Bastion      │             │
│  └────────────────┘  └────────────────┘             │
│           ↓                   ↓                     │
│  ┌────────────────┐  ┌────────────────┐             │
│  │ Private Subnet │  │ Private Subnet │             │
│  │ 10.0.11.0/24   │  │ 10.0.12.0/24   │             │
│  │ • App Servers  │  │ • App Servers  │             │
│  └────────────────┘  └────────────────┘             │
│           ↓                   ↓                     │
│  ┌────────────────┐  ┌────────────────┐             │
│  │ DB Subnet      │  │ DB Subnet      │             │
│  │ 10.0.21.0/24   │  │ 10.0.22.0/24   │             │
│  │ • RDS          │  │ • RDS (replica)│             │
│  └────────────────┘  └────────────────┘             │
│                                                     │
└─────────────────────────────────────────────────────┘
         ↕ (Internet Gateway)
     [Internet]
```

### Création d'un VPC (Terraform)

```hcl
# vpc.tf
# Infrastructure as Code pour créer un VPC AWS complet

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-west-1"  # Paris
}

# 1. VPC principal
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "main-vpc"
    Environment = "production"
    Project     = "myapp"
  }
}

# 2. Internet Gateway (accès Internet)
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# 3. Subnets publics (un par AZ pour HA)
resource "aws_subnet" "public_az1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "eu-west-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-az1"
    Tier = "public"
  }
}

resource "aws_subnet" "public_az2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "eu-west-1b"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-az2"
    Tier = "public"
  }
}

# 4. Subnets privés (app tier)
resource "aws_subnet" "private_app_az1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.11.0/24"
  availability_zone = "eu-west-1a"

  tags = {
    Name = "private-app-subnet-az1"
    Tier = "private-app"
  }
}

resource "aws_subnet" "private_app_az2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.12.0/24"
  availability_zone = "eu-west-1b"

  tags = {
    Name = "private-app-subnet-az2"
    Tier = "private-app"
  }
}

# 5. Subnets privés (database tier)
resource "aws_subnet" "private_db_az1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.21.0/24"
  availability_zone = "eu-west-1a"

  tags = {
    Name = "private-db-subnet-az1"
    Tier = "private-db"
  }
}

resource "aws_subnet" "private_db_az2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.22.0/24"
  availability_zone = "eu-west-1b"

  tags = {
    Name = "private-db-subnet-az2"
    Tier = "private-db"
  }
}

# 6. Elastic IP pour NAT Gateway
resource "aws_eip" "nat_az1" {
  domain = "vpc"

  tags = {
    Name = "nat-eip-az1"
  }

  depends_on = [aws_internet_gateway.main]
}

resource "aws_eip" "nat_az2" {
  domain = "vpc"

  tags = {
    Name = "nat-eip-az2"
  }

  depends_on = [aws_internet_gateway.main]
}

# 7. NAT Gateways (pour accès Internet depuis subnets privés)
resource "aws_nat_gateway" "az1" {
  allocation_id = aws_eip.nat_az1.id
  subnet_id     = aws_subnet.public_az1.id

  tags = {
    Name = "nat-gateway-az1"
  }
}

resource "aws_nat_gateway" "az2" {
  allocation_id = aws_eip.nat_az2.id
  subnet_id     = aws_subnet.public_az2.id

  tags = {
    Name = "nat-gateway-az2"
  }
}

# 8. Route Table pour subnets publics
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-route-table"
  }
}

# 9. Route Tables pour subnets privés (un par AZ)
resource "aws_route_table" "private_az1" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.az1.id
  }

  tags = {
    Name = "private-route-table-az1"
  }
}

resource "aws_route_table" "private_az2" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.az2.id
  }

  tags = {
    Name = "private-route-table-az2"
  }
}

# 10. Associations Route Table ↔ Subnet
resource "aws_route_table_association" "public_az1" {
  subnet_id      = aws_subnet.public_az1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_az2" {
  subnet_id      = aws_subnet.public_az2.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private_app_az1" {
  subnet_id      = aws_subnet.private_app_az1.id
  route_table_id = aws_route_table.private_az1.id
}

resource "aws_route_table_association" "private_app_az2" {
  subnet_id      = aws_subnet.private_app_az2.id
  route_table_id = aws_route_table.private_az2.id
}

resource "aws_route_table_association" "private_db_az1" {
  subnet_id      = aws_subnet.private_db_az1.id
  route_table_id = aws_route_table.private_az1.id
}

resource "aws_route_table_association" "private_db_az2" {
  subnet_id      = aws_subnet.private_db_az2.id
  route_table_id = aws_route_table.private_az2.id
}

# 11. Outputs (pour réutilisation)
output "vpc_id" {
  value       = aws_vpc.main.id
  description = "VPC ID"
}

output "public_subnet_ids" {
  value = [
    aws_subnet.public_az1.id,
    aws_subnet.public_az2.id
  ]
  description = "Public subnet IDs"
}

output "private_app_subnet_ids" {
  value = [
    aws_subnet.private_app_az1.id,
    aws_subnet.private_app_az2.id
  ]
  description = "Private app subnet IDs"
}

output "private_db_subnet_ids" {
  value = [
    aws_subnet.private_db_az1.id,
    aws_subnet.private_db_az2.id
  ]
  description = "Private DB subnet IDs"
}
```

```bash
# Déployer le VPC
terraform init
terraform plan
terraform apply

# Output:
# vpc_id = "vpc-0abc123def456"
# public_subnet_ids = [
#   "subnet-pub1",
#   "subnet-pub2"
# ]
# ...

# Temps de déploiement : ~3 minutes pour un VPC complet HA multi-AZ !
```

### Flux de trafic dans le VPC

```
REQUÊTE ENTRANTE (Internet → App)
═══════════════════════════════════════════════════════

Internet
  ↓
Internet Gateway (IGW)
  ↓
Application Load Balancer (ALB) dans subnet public
  ↓ (route vers private subnet)
EC2 instances dans subnet privé
  ↓ (si besoin DB)
RDS dans subnet DB


REQUÊTE SORTANTE (App → Internet)
═══════════════════════════════════════════════════════

EC2 dans subnet privé
  ↓ (route table: 0.0.0.0/0 → NAT GW)
NAT Gateway dans subnet public
  ↓
Internet Gateway (IGW)
  ↓
Internet

Note : NAT GW permet sortie mais PAS d'entrée directe
       (sécurité : serveurs privés non accessibles depuis Internet)
```

## Sécurité réseau cloud

### Security Groups (Stateful)

Les **Security Groups** sont des **firewalls virtuels** au niveau instance.

```python
# security_groups.tf

# Security Group pour ALB (Load Balancer)
resource "aws_security_group" "alb" {
  name        = "alb-security-group"
  description = "Security group for Application Load Balancer"
  vpc_id      = aws_vpc.main.id

  # Règles INGRESS (entrantes)
  ingress {
    description = "HTTP from Internet"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Tout Internet
  }

  ingress {
    description = "HTTPS from Internet"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Règles EGRESS (sortantes)
  egress {
    description = "All traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"  # Tous protocoles
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "alb-sg"
  }
}

# Security Group pour instances d'application
resource "aws_security_group" "app" {
  name        = "app-security-group"
  description = "Security group for application servers"
  vpc_id      = aws_vpc.main.id

  # Accepter HTTP uniquement depuis l'ALB
  ingress {
    description     = "HTTP from ALB"
    from_port       = 8000
    to_port         = 8000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]  # Référence SG
  }

  # SSH depuis bastion uniquement
  ingress {
    description     = "SSH from Bastion"
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion.id]
  }

  # Ping interne (monitoring)
  ingress {
    description = "ICMP from VPC"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }

  egress {
    description = "All outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "app-sg"
  }
}

# Security Group pour base de données
resource "aws_security_group" "db" {
  name        = "db-security-group"
  description = "Security group for RDS database"
  vpc_id      = aws_vpc.main.id

  # PostgreSQL uniquement depuis les app servers
  ingress {
    description     = "PostgreSQL from App"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }

  # Pas d'egress nécessaire pour RDS managed
  egress {
    description = "All outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "db-sg"
  }
}

# Security Group pour bastion (jump host)
resource "aws_security_group" "bastion" {
  name        = "bastion-security-group"
  description = "Security group for bastion host"
  vpc_id      = aws_vpc.main.id

  # SSH depuis IP du bureau uniquement
  ingress {
    description = "SSH from Office"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["203.0.113.0/24"]  # IP du bureau
  }

  egress {
    description = "SSH to internal"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }

  tags = {
    Name = "bastion-sg"
  }
}
```

**Caractéristiques importantes des Security Groups :**

```
STATEFUL (avec mémoire de connexion)
═══════════════════════════════════════════════════════

Exemple :
  Instance A (10.0.1.5) → Instance B (10.0.2.10:8000)

  1. A envoie SYN à B:8000
  2. SG de B vérifie ingress : port 8000 autorisé depuis A ? ✅
  3. B répond SYN-ACK
  4. SG de B autorise AUTOMATIQUEMENT le retour (stateful)
     Pas besoin de règle egress explicite pour le retour

  Avantage : Pas besoin de dupliquer les règles dans les deux sens
```

### Network ACLs (Stateless)

Les **Network ACLs** sont des firewalls au niveau **subnet** (stateless).

```hcl
# network_acls.tf

# NACL pour subnets publics
resource "aws_network_acl" "public" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = [
    aws_subnet.public_az1.id,
    aws_subnet.public_az2.id
  ]

  # Règles INGRESS (ordre important !)

  # 100: Autoriser HTTP
  ingress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 80
    to_port    = 80
  }

  # 110: Autoriser HTTPS
  ingress {
    protocol   = "tcp"
    rule_no    = 110
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 443
    to_port    = 443
  }

  # 120: Autoriser retours TCP (ephemeral ports)
  ingress {
    protocol   = "tcp"
    rule_no    = 120
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 1024
    to_port    = 65535
  }

  # 200: Bloquer IP malveillante spécifique
  ingress {
    protocol   = "tcp"
    rule_no    = 200
    action     = "deny"
    cidr_block = "203.0.113.50/32"  # IP attaquant
    from_port  = 0
    to_port    = 65535
  }

  # Règles EGRESS

  egress {
    protocol   = "tcp"
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 80
    to_port    = 80
  }

  egress {
    protocol   = "tcp"
    rule_no    = 110
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 443
    to_port    = 443
  }

  egress {
    protocol   = "tcp"
    rule_no    = 120
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 1024
    to_port    = 65535
  }

  tags = {
    Name = "public-nacl"
  }
}

# NACL pour subnets privés (plus restrictif)
resource "aws_network_acl" "private" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = [
    aws_subnet.private_app_az1.id,
    aws_subnet.private_app_az2.id
  ]

  # Autoriser trafic depuis le VPC uniquement
  ingress {
    protocol   = "-1"  # Tous protocoles
    rule_no    = 100
    action     = "allow"
    cidr_block = aws_vpc.main.cidr_block
    from_port  = 0
    to_port    = 0
  }

  # Autoriser retours Internet (ephemeral)
  ingress {
    protocol   = "tcp"
    rule_no    = 110
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 1024
    to_port    = 65535
  }

  egress {
    protocol   = "-1"
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  tags = {
    Name = "private-nacl"
  }
}
```

**Security Groups vs Network ACLs :**

| Aspect | Security Groups | Network ACLs |
|--------|----------------|--------------|
| **Niveau** | Instance (ENI) | Subnet |
| **État** | Stateful | Stateless |
| **Règles** | Allow uniquement | Allow ET Deny |
| **Ordre** | Toutes évaluées | Ordre numérique |
| **Défaut** | Deny all | Allow all |
| **Cas d'usage** | Firewall principal | Défense en profondeur |

```
STRATÉGIE DE DÉFENSE EN PROFONDEUR
═══════════════════════════════════════════════════════

Internet
  ↓
┌─────────────────────────────────────────┐
│ Network ACL (subnet level)              │
│ • Bloquer IPs malveillantes connues     │
│ • Filtrage large par protocole/port     │
└─────────────────────────────────────────┘
  ↓
┌─────────────────────────────────────────┐
│ Security Group (instance level)         │
│ • Règles précises par service           │
│ • Référencement entre SGs               │
│ • Principe du moindre privilège         │
└─────────────────────────────────────────┘
  ↓
Instance (EC2, RDS, etc.)
```

## Load Balancing cloud

### Application Load Balancer (ALB) - Layer 7

```hcl
# alb.tf

# 1. Application Load Balancer
resource "aws_lb" "app" {
  name               = "app-alb"
  internal           = false  # Internet-facing
  load_balancer_type = "application"  # ALB (L7)
  security_groups    = [aws_security_group.alb.id]
  subnets            = [
    aws_subnet.public_az1.id,
    aws_subnet.public_az2.id
  ]

  # Activer access logs (S3)
  access_logs {
    bucket  = aws_s3_bucket.lb_logs.id
    prefix  = "alb"
    enabled = true
  }

  # Protection contre suppression accidentelle
  enable_deletion_protection = true

  # HTTP/2 support
  enable_http2 = true

  # Cross-zone load balancing
  enable_cross_zone_load_balancing = true

  tags = {
    Name        = "app-alb"
    Environment = "production"
  }
}

# 2. Target Group (groupe de cibles)
resource "aws_lb_target_group" "app" {
  name     = "app-target-group"
  port     = 8000
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  # Health check
  health_check {
    enabled             = true
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    timeout             = 5
    interval            = 30
    matcher             = "200"  # HTTP status code attendu
  }

  # Stickiness (session persistence)
  stickiness {
    type            = "lb_cookie"
    cookie_duration = 86400  # 24 heures
    enabled         = true
  }

  # Deregistration delay (drain time)
  deregistration_delay = 30

  tags = {
    Name = "app-tg"
  }
}

# 3. Listener HTTP (redirect vers HTTPS)
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.app.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"  # Permanent redirect
    }
  }
}

# 4. Listener HTTPS (avec certificat SSL)
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.app.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
  certificate_arn   = aws_acm_certificate.app.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

# 5. Règles de routage avancées
resource "aws_lb_listener_rule" "api" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 100

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api.arn
  }

  condition {
    path_pattern {
      values = ["/api/*"]
    }
  }
}

resource "aws_lb_listener_rule" "admin" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 90

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.admin.arn
  }

  condition {
    path_pattern {
      values = ["/admin/*"]
    }
  }

  # Condition supplémentaire : IP source
  condition {
    source_ip {
      values = ["203.0.113.0/24"]  # IP bureau uniquement
    }
  }
}

# 6. Règle pour header custom (A/B testing)
resource "aws_lb_listener_rule" "beta" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 80

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.beta.arn
  }

  condition {
    http_header {
      http_header_name = "X-Beta-User"
      values           = ["true"]
    }
  }
}

# 7. Fixed response (maintenance mode)
resource "aws_lb_listener_rule" "maintenance" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 1  # Priorité la plus haute

  action {
    type = "fixed-response"

    fixed_response {
      content_type = "text/html"
      message_body = "<html><body><h1>Maintenance en cours</h1><p>Retour prévu : 14h00</p></body></html>"
      status_code  = "503"
    }
  }

  # Activer/désactiver via variable
  condition {
    path_pattern {
      values = var.maintenance_mode ? ["/*"] : ["/this-never-matches"]
    }
  }
}

# 8. Attacher des instances au Target Group
resource "aws_lb_target_group_attachment" "app" {
  count            = 3
  target_group_arn = aws_lb_target_group.app.arn
  target_id        = aws_instance.app[count.index].id
  port             = 8000
}
```

### Network Load Balancer (NLB) - Layer 4

Pour les cas nécessitant **très haute performance** et **IP statique**.

```hcl
# nlb.tf

resource "aws_lb" "network" {
  name               = "app-nlb"
  internal           = false
  load_balancer_type = "network"  # NLB (L4)

  # NLB dans subnets publics avec EIP
  subnet_mapping {
    subnet_id     = aws_subnet.public_az1.id
    allocation_id = aws_eip.nlb_az1.id  # IP statique
  }

  subnet_mapping {
    subnet_id     = aws_subnet.public_az2.id
    allocation_id = aws_eip.nlb_az2.id
  }

  # Cross-zone
  enable_cross_zone_load_balancing = true

  tags = {
    Name = "app-nlb"
  }
}

resource "aws_lb_target_group" "tcp" {
  name     = "tcp-target-group"
  port     = 443
  protocol = "TCP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled  = true
    protocol = "TCP"
    port     = 443
    interval = 30
  }

  # Preserve client IP
  preserve_client_ip = true
}

resource "aws_lb_listener" "tcp" {
  load_balancer_arn = aws_lb.network.arn
  port              = "443"
  protocol          = "TCP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tcp.arn
  }
}
```

**ALB vs NLB :**

| Caractéristique | ALB (Layer 7) | NLB (Layer 4) |
|-----------------|---------------|---------------|
| **Protocoles** | HTTP, HTTPS, WebSocket | TCP, UDP, TLS |
| **Latence** | ~ms | ~µs (ultra-faible) |
| **Throughput** | ~500k req/s | Millions paquets/s |
| **IP statique** | ❌ Non (DNS uniquement) | ✅ Oui (EIP) |
| **Routing** | Path, header, query | Port uniquement |
| **SSL termination** | ✅ Oui | ✅ Oui (TLS listener) |
| **WebSocket** | ✅ Oui | ✅ Oui |
| **Preserve client IP** | Via header X-Forwarded-For | ✅ Natif |
| **Coût** | $$ (~$20/mois) | $$$ (~$25/mois) |
| **Cas d'usage** | Web apps, APIs | Gaming, IoT, ultra-perf |

## VPN et connectivité hybride

### Site-to-Site VPN

Connecter votre data center on-premise au VPC cloud.

```hcl
# vpn.tf

# 1. Customer Gateway (votre côté)
resource "aws_customer_gateway" "main" {
  bgp_asn    = 65000
  ip_address = "203.0.113.1"  # IP publique de votre routeur
  type       = "ipsec.1"

  tags = {
    Name = "main-customer-gateway"
  }
}

# 2. Virtual Private Gateway (côté AWS)
resource "aws_vpn_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-vpn-gateway"
  }
}

# 3. VPN Connection
resource "aws_vpn_connection" "main" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.main.id
  type                = "ipsec.1"
  static_routes_only  = false  # Utiliser BGP

  # Deux tunnels pour HA
  tunnel1_inside_cidr   = "169.254.10.0/30"
  tunnel1_preshared_key = var.vpn_preshared_key_1

  tunnel2_inside_cidr   = "169.254.11.0/30"
  tunnel2_preshared_key = var.vpn_preshared_key_2

  tags = {
    Name = "main-vpn-connection"
  }
}

# 4. Route propagation
resource "aws_vpn_gateway_route_propagation" "private" {
  vpn_gateway_id = aws_vpn_gateway.main.id
  route_table_id = aws_route_table.private_az1.id
}

# 5. Routes vers on-premise
resource "aws_vpn_connection_route" "office" {
  destination_cidr_block = "192.168.0.0/16"  # Réseau on-premise
  vpn_connection_id      = aws_vpn_connection.main.id
}
```

**Architecture Site-to-Site VPN :**

```
ON-PREMISE DATA CENTER                AWS CLOUD
═══════════════════════════════════════════════════════

[Réseau local]                    [VPC: 10.0.0.0/16]
192.168.0.0/16                         │
    ↓                                  │
[Firewall/Router]                      │
203.0.113.1 (IP publique)              │
    ↓                                  │
    └──── [Internet] ────┐             │
                         ↓             ↓
                    [VPN Tunnel 1] (encrypted)
                    [VPN Tunnel 2] (backup)
                         ↓             ↓
                    [Virtual Private Gateway]
                         ↓
                    [Route Tables]
                         ↓
            ┌────────────┼────────────┐
            ↓            ↓            ↓
        [Subnet 1]  [Subnet 2]  [Subnet 3]

Communication bidirectionnelle sécurisée
```

### AWS Direct Connect (connexion dédiée)

Pour **bande passante élevée** et **latence stable**, connexion physique dédiée.

```
SITE-TO-SITE VPN (Internet)
═══════════════════════════════════════════════════════
Bande passante : Variable (jusqu'à 1.25 Gbps)
Latence : Variable (Internet public)
Coût : ~$36/mois + data transfer
Setup : 1 heure
HA : Tunnel secondaire


DIRECT CONNECT (Lien dédié)
═══════════════════════════════════════════════════════
Bande passante : 1, 10, 100 Gbps (dédié)
Latence : Prévisible (<10ms typique)
Coût : $200-1000+/mois + port hours
Setup : 2-4 semaines
HA : Connexions redondantes

On-Premise
    ↓
[Router]
    ↓ (fibre optique dédiée)
[AWS Direct Connect Location]
    ↓
[Virtual Private Gateway]
    ↓
VPC
```

```hcl
# direct_connect.tf

resource "aws_dx_connection" "main" {
  name      = "main-dx-connection"
  bandwidth = "10Gbps"
  location  = "EqDC2"  # Equinix DC2 (Paris)

  tags = {
    Name = "main-direct-connect"
  }
}

resource "aws_dx_private_virtual_interface" "main" {
  connection_id = aws_dx_connection.main.id

  name           = "main-dx-vif"
  vlan           = 100
  address_family = "ipv4"
  bgp_asn        = 65000

  amazon_address   = "169.254.10.1/30"
  customer_address = "169.254.10.2/30"

  vpn_gateway_id = aws_vpn_gateway.main.id
}
```

## VPC Peering et Transit Gateway

### VPC Peering

Connecter deux VPCs (même région ou cross-region).

```hcl
# vpc_peering.tf

# VPC A (production)
resource "aws_vpc" "prod" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "prod-vpc"
  }
}

# VPC B (shared services)
resource "aws_vpc" "shared" {
  cidr_block = "10.1.0.0/16"

  tags = {
    Name = "shared-vpc"
  }
}

# Peering connection
resource "aws_vpc_peering_connection" "prod_to_shared" {
  vpc_id      = aws_vpc.prod.id
  peer_vpc_id = aws_vpc.shared.id
  auto_accept = true

  tags = {
    Name = "prod-to-shared-peering"
  }
}

# Routes dans VPC A vers VPC B
resource "aws_route" "prod_to_shared" {
  route_table_id            = aws_route_table.prod_private.id
  destination_cidr_block    = aws_vpc.shared.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.prod_to_shared.id
}

# Routes dans VPC B vers VPC A
resource "aws_route" "shared_to_prod" {
  route_table_id            = aws_route_table.shared_private.id
  destination_cidr_block    = aws_vpc.prod.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.prod_to_shared.id
}
```

**Limitation VPC Peering :**

```
PROBLÈME : Topologie en étoile complexe
═══════════════════════════════════════════════════════

        VPC-Prod
          /  \
         /    \
    VPC-Dev  VPC-Test
         \    /
          \  /
       VPC-Shared

Avec 4 VPCs : 6 peering connections nécessaires !
Avec 10 VPCs : 45 peering connections ! ❌

Pas de transitivité :
  VPC-Dev ne peut PAS atteindre VPC-Test via VPC-Shared
  Il faut un peering direct Dev ↔ Test
```

### Transit Gateway (solution moderne)

**Transit Gateway** est un hub central qui simplifie la connectivité.

```hcl
# transit_gateway.tf

# 1. Transit Gateway
resource "aws_ec2_transit_gateway" "main" {
  description                     = "Main Transit Gateway"
  default_route_table_association = "enable"
  default_route_table_propagation = "enable"
  dns_support                     = "enable"
  vpn_ecmp_support               = "enable"

  tags = {
    Name = "main-tgw"
  }
}

# 2. Attacher les VPCs
resource "aws_ec2_transit_gateway_vpc_attachment" "prod" {
  subnet_ids         = aws_subnet.prod_private[*].id
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.prod.id

  dns_support = "enable"

  tags = {
    Name = "prod-tgw-attachment"
  }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "dev" {
  subnet_ids         = aws_subnet.dev_private[*].id
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.dev.id

  tags = {
    Name = "dev-tgw-attachment"
  }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "shared" {
  subnet_ids         = aws_subnet.shared_private[*].id
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.shared.id

  tags = {
    Name = "shared-tgw-attachment"
  }
}

# 3. Route tables dans VPCs pointent vers TGW
resource "aws_route" "prod_to_tgw" {
  route_table_id         = aws_route_table.prod_private.id
  destination_cidr_block = "10.0.0.0/8"  # Tout le réseau interne
  transit_gateway_id     = aws_ec2_transit_gateway.main.id
}
```

**Avantages Transit Gateway :**

```
TOPOLOGIE AVEC TRANSIT GATEWAY
═══════════════════════════════════════════════════════

        ┌─── VPC-Prod
        │
        ├─── VPC-Dev
        │
[Transit Gateway] ─── VPC-Test
        │
        ├─── VPC-Shared
        │
        ├─── VPN (on-premise)
        │
        └─── Direct Connect

Avantages :
  ✅ Hub central : 1 connexion par VPC
  ✅ Transitivité complète
  ✅ Scalable (5000 VPCs)
  ✅ Routing centralisé
  ✅ Multi-compte et multi-région
```

## Multi-cloud et cloud hybride

### Cas d'usage multi-cloud

```python
# multi_cloud_manager.py
"""
Gestion d'infrastructure multi-cloud (AWS + GCP + Azure)
"""

import boto3  # AWS
from google.cloud import compute_v1  # GCP
from azure.mgmt.network import NetworkManagementClient  # Azure
from azure.identity import DefaultAzureCredential

class MultiCloudNetworkManager:
    """
    Orchestrer le réseau sur plusieurs cloud providers
    """

    def __init__(self):
        # AWS
        self.aws_ec2 = boto3.client('ec2', region_name='eu-west-1')

        # GCP
        self.gcp_compute = compute_v1.InstancesClient()
        self.gcp_project = 'my-project-id'

        # Azure
        self.azure_credential = DefaultAzureCredential()
        self.azure_network = NetworkManagementClient(
            self.azure_credential,
            'subscription-id'
        )

    def create_vpc_aws(self, cidr='10.0.0.0/16'):
        """Créer VPC sur AWS"""
        response = self.aws_ec2.create_vpc(
            CidrBlock=cidr,
            TagSpecifications=[{
                'ResourceType': 'vpc',
                'Tags': [{'Key': 'Name', 'Value': 'multi-cloud-vpc'}]
            }]
        )

        vpc_id = response['Vpc']['VpcId']
        print(f"✅ AWS VPC créé : {vpc_id}")

        return vpc_id

    def create_vpc_gcp(self, name='multi-cloud-vpc'):
        """Créer VPC sur GCP"""
        # GCP utilise des "Networks" au lieu de VPCs
        # Implémentation avec gcloud SDK

        import subprocess

        result = subprocess.run([
            'gcloud', 'compute', 'networks', 'create', name,
            '--subnet-mode=custom',
            '--project', self.gcp_project
        ], capture_output=True, text=True)

        if result.returncode == 0:
            print(f"✅ GCP Network créé : {name}")
            return name
        else:
            print(f"❌ Erreur GCP : {result.stderr}")
            return None

    def create_vnet_azure(self, resource_group, name='multi-cloud-vnet'):
        """Créer VNet sur Azure"""
        vnet_params = {
            'location': 'westeurope',
            'address_space': {
                'address_prefixes': ['10.2.0.0/16']
            }
        }

        poller = self.azure_network.virtual_networks.begin_create_or_update(
            resource_group,
            name,
            vnet_params
        )

        vnet = poller.result()
        print(f"✅ Azure VNet créé : {vnet.name}")

        return vnet

    def setup_multi_cloud_vpn(self):
        """
        Configurer VPN entre les clouds

        Architecture :
          AWS VPC ←→ VPN ←→ GCP VPC
              ↕
            VPN
              ↕
          Azure VNet
        """
        # 1. AWS VPN Gateway
        vpn_gw_aws = self.aws_ec2.create_vpn_gateway(Type='ipsec.1')

        # 2. GCP VPN Gateway
        # Utiliser Cloud VPN

        # 3. Azure VPN Gateway
        # Utiliser VPN Gateway

        # 4. Établir les tunnels
        # Configuration des Customer Gateways et connexions

        print("✅ Multi-cloud VPN configuré")

    def get_network_topology(self):
        """
        Récupérer la topologie réseau complète multi-cloud
        """
        topology = {
            'aws': {
                'vpcs': [],
                'subnets': [],
                'peerings': []
            },
            'gcp': {
                'networks': [],
                'subnets': [],
                'peerings': []
            },
            'azure': {
                'vnets': [],
                'subnets': [],
                'peerings': []
            }
        }

        # Récupérer infos AWS
        vpcs = self.aws_ec2.describe_vpcs()
        for vpc in vpcs['Vpcs']:
            topology['aws']['vpcs'].append({
                'id': vpc['VpcId'],
                'cidr': vpc['CidrBlock']
            })

        # Récupérer infos GCP
        # ...

        # Récupérer infos Azure
        # ...

        return topology

    def calculate_multi_cloud_cost(self):
        """
        Calculer les coûts réseau multi-cloud

        Coûts principaux :
        - Data transfer entre clouds (très cher !)
        - VPN/Direct Connect
        - Load balancers
        - NAT Gateways
        """
        costs = {
            'aws': {
                'vpc': 0,  # VPC gratuit
                'nat_gateway': 0.045 * 730,  # $0.045/h
                'data_transfer_out': 0,  # Dépend du volume
                'vpn': 0.05 * 730  # $0.05/h
            },
            'gcp': {
                'vpc': 0,
                'cloud_nat': 0.045 * 730,
                'data_transfer_out': 0,
                'vpn': 0.05 * 730
            },
            'azure': {
                'vnet': 0,
                'nat_gateway': 0.045 * 730,
                'data_transfer_out': 0,
                'vpn': 0.05 * 730
            }
        }

        # Data transfer entre clouds (le plus cher !)
        # AWS → GCP : $0.09/GB
        # GCP → Azure : $0.12/GB
        # Azure → AWS : $0.087/GB

        # Exemple : 1TB/mois entre clouds
        inter_cloud_transfer_gb = 1000

        costs['inter_cloud_transfer'] = {
            'aws_to_gcp': inter_cloud_transfer_gb * 0.09,
            'gcp_to_azure': inter_cloud_transfer_gb * 0.12,
            'azure_to_aws': inter_cloud_transfer_gb * 0.087
        }

        total = sum(
            sum(cloud.values()) if isinstance(cloud, dict) else cloud
            for cloud in costs.values()
        )

        return {
            'breakdown': costs,
            'total_monthly': total
        }

# Usage
manager = MultiCloudNetworkManager()

# Créer l'infrastructure multi-cloud
aws_vpc = manager.create_vpc_aws('10.0.0.0/16')
gcp_net = manager.create_vpc_gcp('my-gcp-network')
# azure_vnet = manager.create_vnet_azure('my-resource-group')

# Topologie
topology = manager.get_network_topology()
print(f"Topologie : {topology}")

# Coûts
costs = manager.calculate_multi_cloud_cost()
print(f"Coût mensuel total : ${costs['total_monthly']:.2f}")
```

## Monitoring et observabilité

### VPC Flow Logs

Capturer le trafic réseau pour analyse et sécurité.

```hcl
# vpc_flow_logs.tf

# S3 bucket pour les logs
resource "aws_s3_bucket" "flow_logs" {
  bucket = "my-vpc-flow-logs-bucket"
}

# IAM role pour Flow Logs
resource "aws_iam_role" "flow_logs" {
  name = "vpc-flow-logs-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "vpc-flow-logs.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "flow_logs" {
  name = "vpc-flow-logs-policy"
  role = aws_iam_role.flow_logs.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ]
      Effect = "Allow"
      Resource = "*"
    }]
  })
}

# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "flow_logs" {
  name              = "/aws/vpc/flow-logs"
  retention_in_days = 7
}

# VPC Flow Logs
resource "aws_flow_log" "main" {
  iam_role_arn    = aws_iam_role.flow_logs.arn
  log_destination = aws_cloudwatch_log_group.flow_logs.arn
  traffic_type    = "ALL"  # ALL, ACCEPT, ou REJECT
  vpc_id          = aws_vpc.main.id

  # Format custom (optionnel)
  log_format = "$${srcaddr} $${dstaddr} $${srcport} $${dstport} $${protocol} $${packets} $${bytes} $${action} $${log-status}"

  tags = {
    Name = "main-vpc-flow-logs"
  }
}
```

**Analyse des Flow Logs :**

```python
# analyze_flow_logs.py
"""
Analyser les VPC Flow Logs pour détecter anomalies
"""

import boto3
import json
from datetime import datetime, timedelta
from collections import defaultdict

class FlowLogAnalyzer:
    """
    Analyser les VPC Flow Logs
    """

    def __init__(self, log_group_name):
        self.logs_client = boto3.client('logs')
        self.log_group = log_group_name

    def query_logs(self, hours=1):
        """
        Requête CloudWatch Logs Insights
        """
        query = """
        fields @timestamp, srcAddr, dstAddr, srcPort, dstPort, protocol, bytes, action
        | filter action = "REJECT"
        | stats count() as rejections by srcAddr
        | sort rejections desc
        | limit 20
        """

        start_time = int((datetime.now() - timedelta(hours=hours)).timestamp())
        end_time = int(datetime.now().timestamp())

        response = self.logs_client.start_query(
            logGroupName=self.log_group,
            startTime=start_time,
            endTime=end_time,
            queryString=query
        )

        query_id = response['queryId']

        # Attendre les résultats
        import time
        while True:
            result = self.logs_client.get_query_results(queryId=query_id)

            if result['status'] == 'Complete':
                return result['results']

            time.sleep(1)

    def detect_port_scan(self, hours=1):
        """
        Détecter les scans de ports

        Indicateur : Même IP source vers multiples ports destination
        """
        query = """
        fields srcAddr, dstAddr, dstPort
        | filter action = "REJECT"
        | stats count_distinct(dstPort) as unique_ports by srcAddr
        | filter unique_ports > 20
        | sort unique_ports desc
        """

        # Exécuter la requête...
        # (code similaire à query_logs)

        print("🔍 Scans de ports détectés :")
        # Afficher les résultats

    def detect_data_exfiltration(self, threshold_gb=10):
        """
        Détecter l'exfiltration de données

        Indicateur : Volume sortant anormalement élevé
        """
        query = f"""
        fields srcAddr, dstAddr, bytes
        | stats sum(bytes) as total_bytes by srcAddr
        | filter total_bytes > {threshold_gb * 1024 * 1024 * 1024}
        | sort total_bytes desc
        """

        # Exécuter...

        print("⚠️  Exfiltration potentielle détectée :")

    def top_talkers(self, limit=10):
        """
        IPs qui génèrent le plus de trafic
        """
        query = f"""
        fields srcAddr, dstAddr, bytes
        | stats sum(bytes) as total_bytes by srcAddr, dstAddr
        | sort total_bytes desc
        | limit {limit}
        """

        results = self.query_logs_custom(query)

        print(f"Top {limit} conversations :")
        for result in results:
            src = result[0]['value']
            dst = result[1]['value']
            bytes_total = int(result[2]['value'])

            print(f"  {src} → {dst} : {bytes_total / (1024**3):.2f} GB")

    def security_analysis(self):
        """
        Analyse de sécurité complète
        """
        print("=== Analyse de sécurité VPC Flow Logs ===\n")

        # 1. Tentatives de connexion rejetées
        rejected = self.query_logs(hours=24)
        print(f"❌ {len(rejected)} IPs avec connexions rejetées")

        # 2. Scans de ports
        self.detect_port_scan(hours=24)

        # 3. Exfiltration
        self.detect_data_exfiltration(threshold_gb=50)

        # 4. Top talkers
        self.top_talkers(limit=10)

# Usage
analyzer = FlowLogAnalyzer('/aws/vpc/flow-logs')
analyzer.security_analysis()
```

### Métriques CloudWatch

```python
from prometheus_client import Counter, Histogram, Gauge
import boto3

# Métriques personnalisées
network_bytes_in = Counter(
    'network_bytes_in_total',
    'Total bytes received',
    ['instance_id', 'az']
)

network_bytes_out = Counter(
    'network_bytes_out_total',
    'Total bytes sent',
    ['instance_id', 'az']
)

network_packets_in = Counter(
    'network_packets_in_total',
    'Total packets received',
    ['instance_id', 'az']
)

alb_request_count = Counter(
    'alb_request_count_total',
    'ALB request count',
    ['load_balancer', 'target_group']
)

alb_target_response_time = Histogram(
    'alb_target_response_time_seconds',
    'ALB target response time',
    ['load_balancer', 'target_group']
)

nat_gateway_bytes = Counter(
    'nat_gateway_bytes_total',
    'NAT Gateway bytes processed',
    ['nat_gateway_id', 'direction']
)

class CloudNetworkMetrics:
    """
    Collecter et exposer les métriques réseau cloud
    """

    def __init__(self):
        self.cloudwatch = boto3.client('cloudwatch')

    def collect_instance_metrics(self, instance_id):
        """
        Collecter métriques réseau d'une instance
        """
        response = self.cloudwatch.get_metric_statistics(
            Namespace='AWS/EC2',
            MetricName='NetworkIn',
            Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
            StartTime=datetime.utcnow() - timedelta(minutes=5),
            EndTime=datetime.utcnow(),
            Period=300,
            Statistics=['Sum']
        )

        if response['Datapoints']:
            bytes_in = response['Datapoints'][0]['Sum']
            network_bytes_in.labels(
                instance_id=instance_id,
                az='eu-west-1a'
            ).inc(bytes_in)

    def collect_alb_metrics(self, load_balancer_arn):
        """
        Collecter métriques ALB
        """
        # Request count
        response = self.cloudwatch.get_metric_statistics(
            Namespace='AWS/ApplicationELB',
            MetricName='RequestCount',
            Dimensions=[
                {'Name': 'LoadBalancer', 'Value': load_balancer_arn}
            ],
            StartTime=datetime.utcnow() - timedelta(minutes=5),
            EndTime=datetime.utcnow(),
            Period=300,
            Statistics=['Sum']
        )

        # Target response time
        response_time = self.cloudwatch.get_metric_statistics(
            Namespace='AWS/ApplicationELB',
            MetricName='TargetResponseTime',
            Dimensions=[
                {'Name': 'LoadBalancer', 'Value': load_balancer_arn}
            ],
            StartTime=datetime.utcnow() - timedelta(minutes=5),
            EndTime=datetime.utcnow(),
            Period=300,
            Statistics=['Average']
        )
```

## Optimisation des coûts réseau

### Comprendre la facturation

```python
class CloudNetworkCostCalculator:
    """
    Calculer et optimiser les coûts réseau cloud
    """

    # Prix AWS (eu-west-1, exemples)
    PRICING = {
        'nat_gateway': {
            'hourly': 0.045,  # $/heure
            'data_processing': 0.045  # $/GB traité
        },
        'data_transfer': {
            'out_internet': {
                '0-10TB': 0.09,  # $/GB
                '10-50TB': 0.085,
                '50-150TB': 0.070,
                '150TB+': 0.050
            },
            'out_other_region': 0.02,  # $/GB
            'out_same_az': 0,  # Gratuit
            'out_different_az': 0.01  # $/GB
        },
        'vpc': 0,  # Gratuit
        'alb': {
            'hourly': 0.0225,  # $/heure
            'lcu': 0.008  # $/LCU-heure
        },
        'nlb': {
            'hourly': 0.0225,
            'nlcu': 0.006
        },
        'vpn': 0.05  # $/heure
    }

    def calculate_nat_gateway_cost(self, hours_per_month=730, data_gb=1000):
        """
        Coût NAT Gateway

        Composantes :
        1. Coût horaire (uptime)
        2. Data processing
        """
        hourly_cost = self.PRICING['nat_gateway']['hourly'] * hours_per_month
        data_cost = data_gb * self.PRICING['nat_gateway']['data_processing']

        total = hourly_cost + data_cost

        return {
            'hourly': hourly_cost,
            'data_processing': data_cost,
            'total': total
        }

    def calculate_data_transfer_cost(self, data_out_gb, destination='internet'):
        """
        Coût data transfer

        Le plus gros poste de coût réseau !
        """
        if destination == 'internet':
            # Tiered pricing
            cost = 0
            remaining = data_out_gb

            tiers = [
                (10 * 1024, 0.09),   # 0-10TB
                (40 * 1024, 0.085),  # 10-50TB
                (100 * 1024, 0.070), # 50-150TB
                (float('inf'), 0.050) # 150TB+
            ]

            for tier_size, price_per_gb in tiers:
                if remaining <= 0:
                    break

                tier_usage = min(remaining, tier_size)
                cost += tier_usage * price_per_gb
                remaining -= tier_usage

            return cost

        elif destination == 'other_region':
            return data_out_gb * self.PRICING['data_transfer']['out_other_region']

        elif destination == 'different_az':
            return data_out_gb * self.PRICING['data_transfer']['out_different_az']

        elif destination == 'same_az':
            return 0  # Gratuit !

    def optimize_data_transfer(self, monthly_data_gb):
        """
        Suggestions d'optimisation data transfer
        """
        recommendations = []

        # 1. Utiliser CloudFront (CDN)
        current_cost = self.calculate_data_transfer_cost(monthly_data_gb, 'internet')
        cloudfront_cost = monthly_data_gb * 0.085  # Prix CloudFront (simplifié)

        if cloudfront_cost < current_cost:
            savings = current_cost - cloudfront_cost
            recommendations.append({
                'action': 'Utiliser CloudFront CDN',
                'savings_monthly': savings,
                'description': 'Cache le contenu plus proche des utilisateurs'
            })

        # 2. Compression
        compressed_data_gb = monthly_data_gb * 0.3  # ~70% compression
        compressed_cost = self.calculate_data_transfer_cost(compressed_data_gb, 'internet')

        if compressed_cost < current_cost:
            savings = current_cost - compressed_cost
            recommendations.append({
                'action': 'Activer compression gzip/brotli',
                'savings_monthly': savings,
                'description': 'Réduit la bande passante de 70%+'
            })

        # 3. Garder trafic dans la même AZ
        recommendations.append({
            'action': 'Colocaliser services dans même AZ',
            'savings_monthly': monthly_data_gb * 0.01,
            'description': 'Trafic inter-AZ facturé $0.01/GB'
        })

        # 4. S3 Transfer Acceleration
        if monthly_data_gb > 10000:  # 10TB+
            recommendations.append({
                'action': 'Utiliser S3 Transfer Acceleration',
                'savings_monthly': 'Variable',
                'description': 'Accélère uploads vers S3 via edge locations'
            })

        return recommendations

    def calculate_monthly_cost(self, config):
        """
        Calculer coût mensuel total réseau

        Args:
            config: {
                'nat_gateways': 2,
                'nat_data_gb': 5000,
                'albs': 1,
                'data_transfer_internet_gb': 10000,
                'data_transfer_inter_region_gb': 1000,
                'vpn_connections': 1
            }
        """
        costs = {}

        # NAT Gateways
        if config.get('nat_gateways', 0) > 0:
            nat_cost = self.calculate_nat_gateway_cost(
                data_gb=config.get('nat_data_gb', 0)
            )
            costs['nat_gateway'] = nat_cost['total'] * config['nat_gateways']

        # ALB
        if config.get('albs', 0) > 0:
            alb_cost = (
                self.PRICING['alb']['hourly'] * 730 +
                self.PRICING['alb']['lcu'] * 730 * 10  # Assume 10 LCU
            )
            costs['alb'] = alb_cost * config['albs']

        # Data transfer
        costs['data_transfer_internet'] = self.calculate_data_transfer_cost(
            config.get('data_transfer_internet_gb', 0),
            'internet'
        )

        costs['data_transfer_inter_region'] = self.calculate_data_transfer_cost(
            config.get('data_transfer_inter_region_gb', 0),
            'other_region'
        )

        # VPN
        if config.get('vpn_connections', 0) > 0:
            costs['vpn'] = self.PRICING['vpn'] * 730 * config['vpn_connections']

        total = sum(costs.values())

        return {
            'breakdown': costs,
            'total': total,
            'optimizations': self.optimize_data_transfer(
                config.get('data_transfer_internet_gb', 0)
            )
        }

# Usage
calculator = CloudNetworkCostCalculator()

# Configuration actuelle
config = {
    'nat_gateways': 2,
    'nat_data_gb': 5000,
    'albs': 1,
    'data_transfer_internet_gb': 15000,  # 15 TB/mois
    'data_transfer_inter_region_gb': 2000,
    'vpn_connections': 1
}

result = calculator.calculate_monthly_cost(config)

print("=== Coût mensuel réseau cloud ===")
print(f"\nDétail :")
for item, cost in result['breakdown'].items():
    print(f"  {item}: ${cost:.2f}")

print(f"\nTotal : ${result['total']:.2f}/mois")

print(f"\n=== Optimisations recommandées ===")
for opt in result['optimizations']:
    print(f"\n• {opt['action']}")
    print(f"  Économies : ${opt['savings_monthly']:.2f}/mois")
    print(f"  {opt['description']}")
```

## Best Practices cloud networking

### Checklist de sécurité

```yaml
# Cloud Network Security Checklist

## Architecture
- [ ] VPC avec subnets publics/privés séparés
- [ ] Multi-AZ pour haute disponibilité
- [ ] NAT Gateway (pas NAT instance)
- [ ] Pas d'instance publique sans nécessité

## Sécurité
- [ ] Security Groups : deny-all par défaut
- [ ] Principe du moindre privilège (least privilege)
- [ ] Network ACLs pour défense en profondeur
- [ ] VPC Flow Logs activés
- [ ] Bastion host pour accès SSH (ou AWS Systems Manager)
- [ ] Pas de clés SSH hardcodées

## Connectivité
- [ ] TLS/SSL pour toutes les connexions sensibles
- [ ] VPN ou Direct Connect pour on-premise
- [ ] Transit Gateway si multiple VPCs
- [ ] DNS privé (Route 53 Private Hosted Zones)

## Monitoring
- [ ] CloudWatch alertes sur anomalies
- [ ] GuardDuty pour threat detection
- [ ] Logs centralisés (CloudWatch/S3)
- [ ] Tableau de bord réseau temps réel

## Coûts
- [ ] Compression activée
- [ ] CloudFront pour contenu statique
- [ ] Reserved capacity si prévisible
- [ ] Budgets et alertes configurés
- [ ] Revue mensuelle des coûts

## Compliance
- [ ] Chiffrement en transit
- [ ] Chiffrement au repos
- [ ] Audit trail (CloudTrail)
- [ ] Conformité RGPD/SOC2/ISO si applicable
```

### Architecture de référence

```
ARCHITECTURE CLOUD PRODUCTION COMPLÈTE
═══════════════════════════════════════════════════════

┌───────────────────────────────────────────────────┐
│  Route 53 (DNS global)                            │
└─────────────────┬─────────────────────────────────┘
                  ↓
┌───────────────────────────────────────────────────┐
│  CloudFront (CDN global)                          │
└─────────────────┬─────────────────────────────────┘
                  ↓
         [WAF + Shield (DDoS)]
                  ↓
┌───────────────────────────────────────────────────┐
│  VPC (10.0.0.0/16)                                │
├───────────────────────────────────────────────────┤
│                                                   │
│  ┌──────────────────┐  ┌──────────────────┐       │
│  │ AZ-A             │  │ AZ-B             │       │
│  ├──────────────────┤  ├──────────────────┤       │
│  │ Public Subnet    │  │ Public Subnet    │       │
│  │ • ALB            │  │ • ALB            │       │
│  │ • NAT GW         │  │ • NAT GW         │       │
│  │ • Bastion        │  │ • Bastion        │       │
│  └──────────────────┘  └──────────────────┘       │
│          ↓                     ↓                  │
│  ┌──────────────────┐  ┌──────────────────┐       │
│  │ Private Subnet   │  │ Private Subnet   │       │
│  │ • App (ASG)      │  │ • App (ASG)      │       │
│  │ • Lambda         │  │ • Lambda         │       │
│  └──────────────────┘  └──────────────────┘       │
│          ↓                     ↓                  │
│  ┌──────────────────┐  ┌──────────────────┐       │
│  │ DB Subnet        │  │ DB Subnet        │       │
│  │ • RDS (Primary)  │  │ • RDS (Replica)  │       │
│  │ • ElastiCache    │  │ • ElastiCache    │       │
│  └──────────────────┘  └──────────────────┘       │
│                                                   │
│  [VPC Endpoints pour services AWS]                │
│  • S3 Gateway Endpoint                            │
│  • DynamoDB Gateway Endpoint                      │
│  • Interface Endpoints (ECR, CloudWatch, etc.)    │
│                                                   │
└─────────────────┬─────────────────────────────────┘
                  ↓
         [VPN / Direct Connect]
                  ↓
         [On-Premise DC]


Caractéristiques :
  ✅ Multi-AZ (HA)
  ✅ Subnets isolés par tier
  ✅ Auto-scaling
  ✅ Backup automatique
  ✅ Monitoring complet
  ✅ Sécurité en couches
  ✅ Coûts optimisés
```

## Résumé : Cloud Networking

### Concepts clés

**Infrastructure virtuelle :**
- **VPC** = Réseau privé virtuel isolé
- **Subnets** = Segmentation par tier/AZ
- **Route Tables** = Routage personnalisé
- **Gateways** = Accès Internet/VPN

**Sécurité :**
- **Security Groups** = Firewall stateful (instance)
- **Network ACLs** = Firewall stateless (subnet)
- **VPC Flow Logs** = Audit et analyse
- **Private subnets** = Isolation par défaut

**Connectivité :**
- **VPC Peering** = VPC à VPC (simple)
- **Transit Gateway** = Hub central (scalable)
- **VPN** = Connexion chiffrée via Internet
- **Direct Connect** = Lien dédié haute perf

**Services managés :**
- **ALB/NLB** = Load balancing L4/L7
- **NAT Gateway** = Accès Internet sortant
- **CloudFront** = CDN global
- **Route 53** = DNS global

### Différences cloud vs traditionnel

| Aspect | Traditionnel | Cloud |
|--------|--------------|-------|
| **Provisioning** | Semaines/mois | Secondes/minutes |
| **Scaling** | Acheter matériel | API call |
| **Coût** | CapEx élevé | OpEx variable |
| **HA** | Complexe, cher | Natif multi-AZ |
| **Global** | Très difficile | Multi-région facile |
| **Sécurité** | À configurer | Managé + personnalisable |
| **Monitoring** | À installer | Intégré (CloudWatch) |

### Quand utiliser quoi ?

| Besoin | Solution cloud |
|--------|---------------|
| **App web simple** | VPC + ALB + EC2 |
| **App haute dispo** | Multi-AZ + ASG + RDS Multi-AZ |
| **Global low-latency** | CloudFront + Route 53 geo-routing |
| **Hybride cloud** | VPN ou Direct Connect |
| **Multi-VPC** | Transit Gateway |
| **Isolation complète** | Comptes AWS séparés |
| **Très haute perf** | NLB + Enhanced Networking |

### Optimisation des coûts

**Top 5 des économies :**

1. **Compression** → -70% data transfer
2. **CloudFront CDN** → -50% origin traffic
3. **Même AZ** → Gratuit vs $0.01/GB inter-AZ
4. **Reserved Capacity** → -30-50% si prévisible
5. **S3 Gateway Endpoint** → Gratuit vs NAT Gateway

**Erreurs coûteuses à éviter :**

- ❌ Data transfer inter-région non nécessaire
- ❌ NAT Gateway avec peu de trafic (VPC Endpoint moins cher)
- ❌ Pas de compression
- ❌ Pas de CloudFront pour contenu statique
- ❌ Oublier de supprimer ressources de test

## Conclusion

Le **cloud networking** transforme le réseau d'un actif fixe en une ressource **élastique, programmable et globale**. Pour les développeurs, c'est à la fois une **opportunisation** (déploiement rapide, scaling facile) et un **nouveau paradigme** (coûts variables, sécurité partagée).

**À retenir :**

- **Infrastructure as Code** → Terraform, CloudFormation (essentiel)
- **Sécurité par défaut** → Private subnets, Security Groups stricts
- **Multi-AZ** → Haute disponibilité native cloud
- **Monitoring** → VPC Flow Logs, CloudWatch (obligatoire)
- **Coûts** → Data transfer = poste majeur, optimiser !
- **Cloud-native** → Utiliser services managés (ALB, NAT GW) vs DIY

Le cloud networking n'est plus optionnel pour les applications modernes. C'est le **fondement** de toute architecture cloud réussie, que ce soit pour une startup ou une entreprise mondiale.

**Félicitations !** Vous avez terminé le **Module 9 : Architectures et concepts avancés**. Vous maîtrisez maintenant les fondamentaux du networking moderne, du QoS au cloud, en passant par les CDN, la haute disponibilité, les conteneurs et le SDN.

---


⏭️ [10. Évolutions et tendances](/10-evolutions-tendances/README.md)
