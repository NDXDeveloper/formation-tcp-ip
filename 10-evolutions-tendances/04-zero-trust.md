🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.4 - Zero Trust Networking

## Introduction

Pendant des décennies, la sécurité réseau a reposé sur le modèle du **"château fort"** : un périmètre fortifié (firewall) protège un réseau interne de confiance. Une fois à l'intérieur, tout est permis. Ce modèle a échoué face aux réalités modernes : cloud, télétravail, BYOD, et attaques sophistiquées.

**Zero Trust** (confiance zéro) inverse ce paradigme : **"Never trust, always verify"** (ne jamais faire confiance, toujours vérifier). Chaque requête, chaque connexion, chaque accès est vérifié, authentifié et autorisé, peu importe sa provenance.

Pour les développeurs, Zero Trust n'est pas qu'un buzzword sécurité : c'est une architecture qui impacte directement la conception des applications, des APIs, et des communications entre services.

## Le modèle traditionnel : périmètre et ses failles

### Architecture en château fort

```
Modèle périmètre classique :

                    Internet (hostile)
                          |
                    [Firewall] ← Périmètre de sécurité
                          |
                    ┌──────────────────┐
                    │  Réseau interne  │
                    │   (de confiance) │
                    │                  │
                    │  [Serveurs]      │
                    │  [Bases de       │
                    │   données]       │
                    │  [Apps internes] │
                    │  [Postes de      │
                    │   travail]       │
                    └──────────────────┘

Règles implicites :
- Extérieur = dangereux
- Intérieur = sûr
- Une fois à l'intérieur, accès complet
```

### Pourquoi ce modèle échoue

#### 1. Le périmètre n'existe plus

```
Réalité en 2025 :

Employés :
- Bureau → Accès direct
- Domicile → VPN
- Café → WiFi public + VPN
- Mobile → 4G/5G

Services :
- On-premise datacenter
- AWS
- Google Cloud
- Azure
- SaaS tiers (Salesforce, Slack, etc.)

Le "périmètre" est fragmenté, flou, impossible à définir clairement
```

#### 2. Menaces internes

```
Statistiques (Verizon DBIR 2024) :
- 30% des violations impliquent des insiders
- 82% des violations impliquent l'humain
- Temps moyen de détection : 280 jours

Scénarios :
- Employé malveillant
- Compte compromis (phishing)
- Malware interne
- Mouvement latéral après intrusion

Une fois dans le périmètre, l'attaquant a accès à tout
```

#### 3. Principe du moindre privilège non appliqué

```
Problème typique :

Développeur a besoin d'accéder à :
- Base de données de développement
- Serveur de staging
- Repository Git

Mais avec VPN périmètre, il obtient aussi accès à :
- Base de données de production
- Serveurs RH (données sensibles)
- Systèmes financiers
- Backups

Violation du principe du moindre privilège
```

**Exemple concret d'attaque** :

```
1. Attaquant compromet laptop d'un développeur (phishing)
2. Laptop se connecte au VPN entreprise
3. Firewall vérifie credentials VPN → OK, accès autorisé
4. Une fois connecté, laptop a accès réseau complet
5. Attaquant scanne le réseau interne
6. Trouve base de données non protégée
7. Exfiltre données clients
8. Total : 180 jours avant détection

Avec Zero Trust :
1. Attaquant compromet laptop
2. Laptop se connecte → Authentification OK
3. Tente d'accéder à la base de données
4. Authentification requise POUR CHAQUE RESSOURCE
5. Device posture check détecte comportement anormal
6. Accès refusé
7. Alerte de sécurité immédiate
```

## Principes fondamentaux du Zero Trust

### 1. Never Trust, Always Verify

```
Modèle traditionnel :
if (source == "réseau interne") {
    trust = true;
    allow_access();
}

Modèle Zero Trust :
for each request {
    verify_identity();
    verify_device_posture();
    verify_context();
    check_authorization();

    if (all_checks_pass) {
        allow_access();
        log_access();
        monitor_continuously();
    } else {
        deny_access();
        alert_security();
    }
}
```

**Aucune confiance implicite** : ni la source, ni la destination, ni le réseau.

### 2. Least Privilege Access

Accorder uniquement les permissions strictement nécessaires, pour la durée nécessaire.

```python
# ❌ Mauvais : accès permanent et large
user.grant_permission("database.*", expires=None)

# ✅ Bon : accès minimal et temporaire
user.grant_permission(
    resource="database.customers.read",
    expires=datetime.now() + timedelta(hours=4),
    justification="Support ticket #12345"
)
```

### 3. Assume Breach

Considérer que le réseau est **déjà compromis** et concevoir en conséquence.

```
Design implications :

❌ Ne pas faire :
- Confiance basée sur l'IP source
- Authentification une seule fois à l'entrée
- Communications internes non chiffrées
- Pas de segmentation réseau

✅ Faire :
- Authentification pour chaque requête
- Chiffrement end-to-end (même interne)
- Micro-segmentation
- Monitoring continu
- Détection d'anomalies
```

### 4. Verify Explicitly

Utiliser toutes les données disponibles pour la décision d'accès.

```javascript
// Facteurs de décision Zero Trust
const accessDecision = evaluateAccess({
    // Identité
    user: {
        id: "user@example.com",
        roles: ["developer", "team-backend"],
        mfa_verified: true
    },

    // Device posture
    device: {
        managed: true,
        os_version: "Windows 11 22H2",
        antivirus_updated: true,
        disk_encrypted: true,
        last_patch: "2025-12-01"
    },

    // Contexte
    context: {
        location: "San Francisco, CA",
        time: "2025-12-07 14:30:00",
        network: "corporate-wifi",
        risk_score: 0.15
    },

    // Ressource demandée
    resource: {
        type: "database",
        name: "customers",
        action: "read",
        sensitivity: "high"
    },

    // Comportement
    behavior: {
        normal_access_times: ["09:00-18:00"],
        normal_locations: ["SF", "NY office"],
        typical_resources: ["dev-db", "staging-api"]
    }
});

if (accessDecision.allow) {
    grantAccess({ duration: "4h", audit: true });
} else {
    denyAccess({ reason: accessDecision.reason });
}
```

### 5. Micro-segmentation

Diviser le réseau en zones minuscules, isolées les unes des autres.

```
Segmentation traditionnelle :
[DMZ] | [Réseau interne] | [Réseau management]
 ~50   ~500 machines       ~10 machines
hosts

Micro-segmentation Zero Trust :
Chaque workload/service/pod est son propre segment

Service A ←→ Service B : politique explicite requise
Service A ←/→ Service C : pas de connexion sans politique
Pod 1 ←→ Pod 2 : isolation par défaut

Granularité : niveau application, pas niveau réseau
```

## Architecture Zero Trust

### Composants clés

#### 1. Policy Decision Point (PDP)

Le cerveau qui décide : autoriser ou refuser ?

```go
// PDP simplifié
type PolicyDecisionPoint struct {
    policies []Policy
    contextProvider ContextProvider
}

func (pdp *PolicyDecisionPoint) Evaluate(request AccessRequest) Decision {
    // Collecter contexte
    context := pdp.contextProvider.GetContext(request)

    // Évaluer toutes les policies
    for _, policy := range pdp.policies {
        if policy.Matches(request, context) {
            return policy.Decision
        }
    }

    // Deny by default
    return Decision{
        Allow: false,
        Reason: "No matching policy"
    }
}

// Exemple de policy
policy := Policy{
    Subject: "role:developer",
    Resource: "database:dev-customers",
    Action: "read",
    Conditions: []Condition{
        {Field: "device.managed", Operator: "equals", Value: true},
        {Field: "mfa.verified", Operator: "equals", Value: true},
        {Field: "time.hour", Operator: "between", Value: [9, 18]},
    },
    Decision: Decision{Allow: true, TTL: 4 * time.Hour},
}
```

#### 2. Policy Enforcement Point (PEP)

Le point qui applique la décision du PDP.

```python
# PEP dans un API Gateway
class ZeroTrustEnforcementPoint:
    def __init__(self, pdp_client):
        self.pdp = pdp_client

    def intercept_request(self, request):
        """
        Intercepte chaque requête et vérifie autorisation
        """
        # Extraire informations de la requête
        access_request = AccessRequest(
            user=request.headers.get('X-User-ID'),
            resource=request.path,
            action=request.method,
            device_id=request.headers.get('X-Device-ID'),
            source_ip=request.remote_addr
        )

        # Consulter PDP
        decision = self.pdp.evaluate(access_request)

        if decision.allow:
            # Logger l'accès
            self.audit_log(access_request, decision)

            # Autoriser avec metadata
            request.headers['X-Authorization-TTL'] = decision.ttl
            request.headers['X-Session-ID'] = decision.session_id

            return self.forward_request(request)
        else:
            # Refuser et alerter
            self.security_alert(access_request, decision.reason)
            return Response(status=403, body={
                "error": "Access denied",
                "reason": decision.reason
            })
```

#### 3. Identity Provider (IdP)

Source de vérité pour les identités.

```yaml
# Configuration IdP (Okta, Auth0, Azure AD, etc.)
identity_provider:
  type: okta
  domain: company.okta.com

  authentication:
    methods:
      - password
      - webauthn
      - totp

    mfa:
      required: true
      methods: [totp, webauthn, sms]

    risk_based:
      enabled: true
      factors:
        - location_deviation
        - impossible_travel
        - device_fingerprint
        - behavioral_analytics

  authorization:
    claims:
      - email
      - groups
      - roles
      - department

    token_lifetime: 1h
    refresh_token: 8h
```

#### 4. Device Trust

Vérification de l'état et de la sécurité des appareils.

```javascript
// Client-side device attestation
class DeviceTrustClient {
    async getDevicePosture() {
        return {
            // Identité de l'appareil
            device_id: await this.getDeviceFingerprint(),

            // État de sécurité
            security: {
                os_version: this.getOSVersion(),
                patch_level: await this.getPatchLevel(),
                antivirus: {
                    installed: true,
                    updated: true,
                    last_scan: "2025-12-07T08:00:00Z"
                },
                firewall_enabled: true,
                disk_encrypted: await this.checkDiskEncryption()
            },

            // Configuration
            config: {
                screen_lock_enabled: true,
                auto_lock_timeout: 300, // 5 minutes
                biometric_enabled: true
            },

            // Attestation cryptographique (TPM)
            attestation: await this.getTPMAttestation()
        };
    }

    async sendPostureToGateway() {
        const posture = await this.getDevicePosture();

        // Signer avec clé privée du device
        const signature = await this.sign(posture);

        // Envoyer au PEP/Gateway
        await fetch('https://api.company.com/device-posture', {
            method: 'POST',
            headers: {
                'X-Device-ID': posture.device_id,
                'X-Device-Signature': signature
            },
            body: JSON.stringify(posture)
        });
    }
}
```

### Architecture de référence

```
┌─────────────────────────────────────────────────────────────┐
│                         Internet                            │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │
         ┌───────────▼──────────────┐
         │   Identity-Aware Proxy   │ ← PEP
         │   (Cloud Load Balancer)  │
         └───────────┬──────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
   ┌────▼───┐  ┌─────▼──┐  ┌──────▼─┐
   │Service │  │Service │  │Service │
   │   A    │  │   B    │  │   C    │
   └────┬───┘  └────┬───┘  └────┬───┘
        │           │           │
        └───────────┼───────────┘
                    │
         ┌──────────▼──────────┐
         │  Policy Decision    │ ← PDP
         │      Point          │
         └──────────┬──────────┘
                    │
         ┌──────────▼──────────┐
         │   Identity Provider │ ← IdP
         │     (Okta/Auth0)    │
         └─────────────────────┘

Flux :
1. User/Device → Identity-Aware Proxy (authentification)
2. Proxy → PDP (demande décision d'accès)
3. PDP → IdP (vérification identité)
4. PDP → Device Trust (vérification posture)
5. PDP → Proxy (décision : allow/deny)
6. Proxy → Service (si autorisé, avec contexte)
```

## Implémentation pour développeurs

### 1. mTLS (Mutual TLS) entre services

**Le problème** : Dans les microservices, comment garantir que Service A qui parle à Service B est bien qui il prétend être ?

**Solution Zero Trust** : Mutual TLS avec certificats courts.

```yaml
# Configuration Istio pour mTLS strict
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # Force mTLS pour tout le trafic

---
# Authorization Policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: service-a-to-b
  namespace: production
spec:
  selector:
    matchLabels:
      app: service-b
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/service-a"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/v1/*"]
```

**Implémentation Go avec mTLS** :

```go
package main

import (
    "crypto/tls"
    "crypto/x509"
    "io/ioutil"
    "net/http"
)

// Client avec mTLS
func createMTLSClient() (*http.Client, error) {
    // Charger certificat client
    cert, err := tls.LoadX509KeyPair(
        "/path/to/client-cert.pem",
        "/path/to/client-key.pem",
    )
    if err != nil {
        return nil, err
    }

    // Charger CA pour vérifier serveur
    caCert, err := ioutil.ReadFile("/path/to/ca-cert.pem")
    if err != nil {
        return nil, err
    }

    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    // Configuration TLS
    tlsConfig := &tls.Config{
        Certificates: []tls.Certificate{cert},
        RootCAs:      caCertPool,
        MinVersion:   tls.VersionTLS13,
    }

    // Client HTTP avec mTLS
    client := &http.Client{
        Transport: &http.Transport{
            TLSClientConfig: tlsConfig,
        },
    }

    return client, nil
}

// Serveur avec mTLS
func startMTLSServer() error {
    // Charger certificat serveur
    cert, err := tls.LoadX509KeyPair(
        "/path/to/server-cert.pem",
        "/path/to/server-key.pem",
    )
    if err != nil {
        return err
    }

    // Charger CA pour vérifier clients
    caCert, err := ioutil.ReadFile("/path/to/ca-cert.pem")
    if err != nil {
        return err
    }

    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)

    // Configuration TLS
    tlsConfig := &tls.Config{
        Certificates: []tls.Certificate{cert},
        ClientCAs:    caCertPool,
        ClientAuth:   tls.RequireAndVerifyClientCert, // Force mTLS
        MinVersion:   tls.VersionTLS13,
    }

    // Handler
    http.HandleFunc("/api/data", func(w http.ResponseWriter, r *http.Request) {
        // Vérifier le certificat client
        if r.TLS == nil || len(r.TLS.PeerCertificates) == 0 {
            http.Error(w, "No client certificate", http.StatusUnauthorized)
            return
        }

        clientCert := r.TLS.PeerCertificates[0]
        clientCN := clientCert.Subject.CommonName

        // Logger l'accès
        log.Printf("Request from: %s", clientCN)

        // Vérifier autorisation basée sur CN
        if !isAuthorized(clientCN, r.URL.Path) {
            http.Error(w, "Forbidden", http.StatusForbidden)
            return
        }

        w.Write([]byte("Data response"))
    })

    // Démarrer serveur HTTPS avec mTLS
    server := &http.Server{
        Addr:      ":8443",
        TLSConfig: tlsConfig,
    }

    return server.ListenAndServeTLS("", "")
}
```

### 2. JWT avec claims riches

```javascript
// Génération JWT avec contexte Zero Trust
const jwt = require('jsonwebtoken');

function generateZeroTrustToken(user, device, context) {
    const payload = {
        // Identité
        sub: user.id,
        email: user.email,
        roles: user.roles,

        // Device
        device_id: device.id,
        device_trusted: device.posture_verified,
        device_compliance: device.compliance_score,

        // Contexte
        ip: context.ip,
        location: context.location,
        risk_score: context.risk_score,

        // Métadonnées
        iat: Math.floor(Date.now() / 1000),
        exp: Math.floor(Date.now() / 1000) + (4 * 3600), // 4 heures
        iss: 'https://auth.company.com',
        aud: ['api.company.com', 'internal-services'],

        // Permissions spécifiques
        permissions: [
            'customers:read',
            'orders:read',
            'products:write'
        ],

        // Contraintes
        constraints: {
            max_ip_changes: 2,
            max_location_distance_km: 100,
            require_mfa_for_sensitive: true
        }
    };

    return jwt.sign(payload, privateKey, { algorithm: 'RS256' });
}

// Validation JWT avec vérifications Zero Trust
function validateZeroTrustToken(token, request) {
    try {
        const decoded = jwt.verify(token, publicKey);

        // Vérifications additionnelles

        // 1. IP a changé ?
        if (decoded.ip !== request.ip) {
            if (!isIPChangeAllowed(decoded, request)) {
                throw new Error('Suspicious IP change detected');
            }
        }

        // 2. Location coherent ?
        const distance = calculateDistance(
            decoded.location,
            request.geoip_location
        );

        if (distance > decoded.constraints.max_location_distance_km) {
            throw new Error('Impossible travel detected');
        }

        // 3. Device posture toujours valide ?
        if (!verifyDevicePosture(decoded.device_id)) {
            throw new Error('Device no longer trusted');
        }

        // 4. Risk score acceptable ?
        if (decoded.risk_score > 0.7) {
            throw new Error('High risk score - re-authentication required');
        }

        return decoded;

    } catch (error) {
        // Token invalide ou vérifications échouées
        logSecurityEvent('token_validation_failed', { error, request });
        return null;
    }
}
```

### 3. API Gateway avec Zero Trust

```python
# Flask API Gateway avec enforcement Zero Trust
from flask import Flask, request, jsonify
import jwt
import requests

app = Flask(__name__)

class ZeroTrustGateway:
    def __init__(self):
        self.pdp_url = "https://pdp.company.com/evaluate"
        self.public_key = load_public_key()

    def verify_request(self):
        """
        Middleware Zero Trust pour chaque requête
        """
        # 1. Extraire et vérifier JWT
        token = request.headers.get('Authorization', '').replace('Bearer ', '')

        try:
            claims = jwt.decode(token, self.public_key, algorithms=['RS256'])
        except jwt.InvalidTokenError:
            return jsonify({"error": "Invalid token"}), 401

        # 2. Collecter contexte de la requête
        context = {
            "user_id": claims['sub'],
            "roles": claims['roles'],
            "device_id": claims['device_id'],
            "source_ip": request.remote_addr,
            "user_agent": request.headers.get('User-Agent'),
            "resource": request.path,
            "action": request.method,
            "timestamp": time.time()
        }

        # 3. Consulter PDP pour décision
        decision = self.consult_pdp(context)

        if not decision['allow']:
            # Log refus
            self.audit_log('access_denied', context, decision['reason'])
            return jsonify({
                "error": "Access denied",
                "reason": decision['reason']
            }), 403

        # 4. Enrichir requête avec contexte
        request.zt_context = context
        request.zt_decision = decision

        return None  # Continue processing

    def consult_pdp(self, context):
        """
        Consulter Policy Decision Point
        """
        response = requests.post(
            self.pdp_url,
            json=context,
            timeout=1.0  # Fail fast
        )

        if response.status_code != 200:
            # Si PDP indisponible, deny by default (fail closed)
            return {
                "allow": False,
                "reason": "PDP unavailable - fail closed"
            }

        return response.json()

    def audit_log(self, event, context, details):
        """
        Audit logging pour compliance
        """
        log_entry = {
            "timestamp": time.time(),
            "event": event,
            "user_id": context.get('user_id'),
            "resource": context.get('resource'),
            "action": context.get('action'),
            "source_ip": context.get('source_ip'),
            "details": details
        }

        # Envoyer à SIEM
        send_to_siem(log_entry)

# Initialiser
zt_gateway = ZeroTrustGateway()

# Appliquer à toutes les routes
@app.before_request
def enforce_zero_trust():
    return zt_gateway.verify_request()

# Route d'API
@app.route('/api/customers/<customer_id>')
def get_customer(customer_id):
    # À ce stade, la requête a été autorisée par Zero Trust

    # Vérifier permission spécifique
    if 'customers:read' not in request.zt_context['roles']:
        return jsonify({"error": "Insufficient permissions"}), 403

    # Vérifier si accès à ce customer spécifique est autorisé
    if not can_access_customer(request.zt_context['user_id'], customer_id):
        return jsonify({"error": "Not authorized for this customer"}), 403

    # Logger l'accès
    zt_gateway.audit_log(
        'customer_accessed',
        request.zt_context,
        {'customer_id': customer_id}
    )

    # Récupérer données
    customer = get_customer_from_db(customer_id)

    return jsonify(customer)
```

### 4. Service Mesh avec Istio

**Configuration Istio pour Zero Trust complet** :

```yaml
# 1. Activer mTLS strict pour tout le mesh
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT

---
# 2. Authorization Policy : deny by default
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  {} # Pas de règles = deny all

---
# 3. Autoriser explicitement les flux nécessaires
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: frontend-to-backend
  namespace: production
spec:
  selector:
    matchLabels:
      app: backend-api
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/frontend"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/v1/*"]
    when:
    - key: request.headers[x-user-id]
      notValues: [""]  # User ID requis

---
# 4. Rate limiting par identité
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: rate-limit-by-user
  namespace: production
spec:
  configPatches:
  - applyTo: HTTP_FILTER
    match:
      context: SIDECAR_INBOUND
    patch:
      operation: INSERT_BEFORE
      value:
        name: envoy.filters.http.ratelimit
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.http.ratelimit.v3.RateLimit
          domain: production-ratelimit
          rate_limit_service:
            grpc_service:
              envoy_grpc:
                cluster_name: rate-limit-cluster

---
# 5. Telemetry et audit
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: security-telemetry
  namespace: production
spec:
  accessLogging:
  - providers:
    - name: security-logger
    filter:
      expression: "response.code >= 400"  # Logger tous les refus
```

**Application Go avec Istio** :

```go
package main

import (
    "context"
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/api/v1/data", handleData)
    http.ListenAndServe(":8080", nil)
}

func handleData(w http.ResponseWriter, r *http.Request) {
    // Istio injecte automatiquement les headers avec identité

    // Source service identity (du certificat mTLS)
    sourceIdentity := r.Header.Get("X-Forwarded-Client-Cert")

    // User identity (du JWT forwarded par Istio)
    userID := r.Header.Get("X-User-Id")
    userRoles := r.Header.Get("X-User-Roles")

    // Device context
    deviceID := r.Header.Get("X-Device-Id")
    deviceTrust := r.Header.Get("X-Device-Trust-Score")

    // À ce stade, Istio a déjà :
    // 1. Vérifié mTLS
    // 2. Vérifié AuthorizationPolicy
    // 3. Appliqué rate limiting
    // 4. Forwardé les headers d'identité

    // Application fait ses propres vérifications
    if !hasPermission(userRoles, "data:read") {
        http.Error(w, "Forbidden", http.StatusForbidden)
        return
    }

    // Logger l'accès pour audit
    logAccess(AccessLog{
        UserID:         userID,
        SourceService:  parseSourceFromCert(sourceIdentity),
        Resource:       r.URL.Path,
        Action:         r.Method,
        DeviceID:       deviceID,
        Allowed:        true,
    })

    // Traiter la requête
    data := getData(userID)

    w.Header().Set("Content-Type", "application/json")
    fmt.Fprintf(w, `{"data": %s}`, data)
}
```

## Cas d'usage réels

### Cas 1 : Migration d'une app monolithique vers Zero Trust

**Contexte** : Entreprise SaaS B2B avec app PHP monolithique + VPN pour employés.

**Avant** :

```
Architecture :
- VPN IPsec pour tous les employés
- Une fois connecté : accès complet au réseau interne
- App monolithique avec DB MySQL
- Pas de segmentation
- Authentification une fois au VPN

Problèmes :
- Employé compromis = accès total
- Impossible de tracer qui accède à quoi
- Pas de granularité dans les permissions
- Compliance difficile (SOC 2, ISO 27001)
```

**Après (Zero Trust)** :

```
Architecture :
1. Déploiement BeyondCorp/Identity-Aware Proxy
2. Chaque requête authentifiée individuellement
3. Micro-segmentation de la DB
4. Migration progressive vers microservices

Phase 1 : IAP (Identity-Aware Proxy)
┌──────────┐
│ Employee │
└────┬─────┘
     │ HTTPS
     ▼
┌─────────────────┐
│   Google IAP    │ ← Authentification Google OAuth
│  (ou Cloudflare │    + Device posture check
│     Access)     │
└────┬────────────┘
     │ mTLS
     ▼
┌─────────────────┐
│ Monolith App    │
└─────────────────┘

Bénéfices immédiats :
- Suppression VPN
- Authentification par requête
- Logging détaillé
- Access depuis n'importe où (travail remote)
```

**Implémentation Google IAP** :

```python
# Application Flask derrière Google IAP
from flask import Flask, request
import google.auth.transport.requests
import google.oauth2.id_token

app = Flask(__name__)

def verify_iap_jwt():
    """
    Vérifier JWT injecté par IAP
    """
    iap_jwt = request.headers.get('X-Goog-IAP-JWT-Assertion')

    if not iap_jwt:
        return None

    try:
        # Vérifier signature du JWT
        decoded = google.oauth2.id_token.verify_token(
            iap_jwt,
            google.auth.transport.requests.Request(),
            audience='/projects/PROJECT_NUMBER/apps/PROJECT_ID'
        )

        return {
            'email': decoded['email'],
            'user_id': decoded['sub'],
            'verified': True
        }
    except Exception as e:
        return None

@app.before_request
def enforce_iap():
    """
    Middleware : vérifier que requête vient d'IAP
    """
    user = verify_iap_jwt()

    if not user:
        return "Unauthorized - must access via IAP", 401

    # Stocker user dans context
    request.user = user

@app.route('/admin/users')
def admin_users():
    # IAP a déjà authentifié, vérifier rôle
    if not is_admin(request.user['email']):
        return "Forbidden", 403

    users = get_all_users()
    return jsonify(users)
```

**Résultats après 6 mois** :

```
Sécurité :
- 0 incidents vs 2 incidents/an avant
- Détection tentatives d'accès non-autorisé : temps réel vs jamais
- Temps moyen de détection anomalie : 2 min vs 30 jours

Productivité :
- Temps connexion employé : 0s (plus de VPN) vs 30s
- Support IT : -40% de tickets VPN
- Satisfaction employés : +25%

Compliance :
- SOC 2 Type II obtenu (impossible avant)
- Audits simplifiés (logs détaillés)
- Coût audit : -30%
```

### Cas 2 : API publique avec Zero Trust interne

**Contexte** : Startup fintech avec API publique pour partenaires.

**Architecture** :

```
External Partners
      │
      ▼
┌──────────────┐
│ API Gateway  │ ← OAuth 2.0 + API keys pour partenaires
│ (Kong/Apigee)│
└──────┬───────┘
       │
Internal Zero Trust boundary
       │
       ▼
┌──────────────┐
│Auth Service  │ ← mTLS, JWT interne
└──────┬───────┘
       │
   ┌───┼────┐
   ▼   ▼    ▼
┌────┐┌───┐┌──────┐
│User││Pay││Ledger│ ← Chaque service vérifie JWT
│Svc ││Svc││Svc   │   + mTLS
└────┘└───┘└──────┘
```

**API Gateway (externe)** :

```javascript
// Kong plugin pour partenaires
const axios = require('axios');

async function authenticatePartner(apiKey) {
    // 1. Vérifier API key
    const partner = await validateAPIKey(apiKey);
    if (!partner) {
        return { valid: false };
    }

    // 2. Rate limiting par partenaire
    const rateLimit = await checkRateLimit(partner.id);
    if (!rateLimit.allowed) {
        return { valid: false, reason: 'Rate limit exceeded' };
    }

    // 3. Générer JWT interne pour services backend
    const internalJWT = generateInternalJWT({
        partner_id: partner.id,
        partner_name: partner.name,
        tier: partner.tier,
        permissions: partner.permissions,
        rate_limit: rateLimit.remaining
    });

    return {
        valid: true,
        jwt: internalJWT
    };
}

// Middleware Kong
module.exports = {
    access: async function(config) {
        const apiKey = kong.request.getHeader('X-API-Key');

        const auth = await authenticatePartner(apiKey);

        if (!auth.valid) {
            return kong.response.exit(401, {
                error: auth.reason || 'Invalid API key'
            });
        }

        // Injecter JWT interne pour backend
        kong.service.request.setHeader('X-Internal-JWT', auth.jwt);

        // Supprimer API key (ne pas propager)
        kong.service.request.clearHeader('X-API-Key');
    }
};
```

**Service backend avec vérification Zero Trust** :

```go
package main

import (
    "github.com/golang-jwt/jwt"
    "net/http"
)

type InternalClaims struct {
    PartnerID   string   `json:"partner_id"`
    PartnerName string   `json:"partner_name"`
    Tier        string   `json:"tier"`
    Permissions []string `json:"permissions"`
    jwt.StandardClaims
}

func paymentHandler(w http.ResponseWriter, r *http.Request) {
    // 1. Extraire JWT interne
    tokenString := r.Header.Get("X-Internal-JWT")

    // 2. Vérifier signature
    claims := &InternalClaims{}
    token, err := jwt.ParseWithClaims(tokenString, claims, func(token *jwt.Token) (interface{}, error) {
        return publicKey, nil  // Clé publique du service Auth
    })

    if err != nil || !token.Valid {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }

    // 3. Vérifier permissions
    if !hasPermission(claims.Permissions, "payments:create") {
        http.Error(w, "Forbidden", http.StatusForbidden)
        return
    }

    // 4. Vérifier tier pour limites
    amount := parseAmount(r)
    if amount > getTierLimit(claims.Tier) {
        http.Error(w, "Amount exceeds tier limit", http.StatusBadRequest)
        return
    }

    // 5. Logger avec contexte complet
    logPaymentAttempt(PaymentLog{
        PartnerID:   claims.PartnerID,
        PartnerName: claims.PartnerName,
        Amount:      amount,
        Timestamp:   time.Now(),
    })

    // 6. Traiter paiement
    result := processPayment(amount, claims.PartnerID)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}
```

**Résultats** :

```
Sécurité :
- 1 partenaire compromis → impact limité à ses permissions
- Attaque par force brute détectée en temps réel (rate limiting)
- Fraude réduite de 60% (granularité permissions)

Performance :
- Latence ajoutée par JWT : ~2ms (acceptable)
- Cache des validations : hit rate 95%

Conformité PCI-DSS :
- Audit trail complet : qui, quoi, quand
- Segregation of duties native
- Least privilege appliqué
```

### Cas 3 : Kubernetes multi-tenant avec Zero Trust

**Contexte** : Plateforme Kubernetes partagée par plusieurs équipes/produits.

**Problème** :

```
Sans Zero Trust :
- Namespace isolation insuffisante
- Pod A peut parler à Pod B dans autre namespace
- Secrets accessibles entre namespaces
- Pas de traçabilité des communications
```

**Solution avec Istio + OPA (Open Policy Agent)** :

```yaml
# 1. Istio : mTLS obligatoire
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT

---
# 2. Deny all par défaut
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: team-a
spec:
  {}

---
# 3. Policy OPA pour autorisation fine-grained
apiVersion: v1
kind: ConfigMap
metadata:
  name: opa-policy
  namespace: opa
data:
  policy.rego: |
    package istio.authz

    import input.attributes.request.http as http_request
    import input.attributes.source.principal as source_principal

    default allow = false

    # Règle : Team A peut appeler ses propres services
    allow {
        startswith(source_principal, "cluster.local/ns/team-a/")
        startswith(http_request.path, "/api/")
        http_request.method == "GET"
    }

    # Règle : Team A frontend peut appeler Team B API (explicite)
    allow {
        source_principal == "cluster.local/ns/team-a/sa/frontend"
        http_request.host == "team-b-api.team-b.svc.cluster.local"
        http_request.path == "/api/v1/data"
        http_request.method == "GET"
    }

    # Règle : Secrets management service peut accéder à tout
    allow {
        source_principal == "cluster.local/ns/platform/sa/secrets-manager"
    }

    # Logger tous les refus
    deny_reason = "Source not authorized" {
        not allow
    }

---
# 4. NetworkPolicy pour defense in depth
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: team-a-isolation
  namespace: team-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: team-a
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          team: team-a
  - to:  # Autoriser DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

**Application avec injection de contexte** :

```python
# app.py - Service Team A
from flask import Flask, request
import requests
import jwt

app = Flask(__name__)

@app.route('/api/process')
def process_data():
    # Istio a injecté les headers d'identité
    source_service = request.headers.get('X-Forwarded-Client-Cert')
    user_jwt = request.headers.get('X-User-Token')

    # Vérifier user JWT
    try:
        user_claims = jwt.decode(
            user_jwt,
            public_key,
            algorithms=['RS256']
        )
    except:
        return {"error": "Invalid user token"}, 401

    # Appeler Team B API (autorisé par OPA)
    response = requests.get(
        'http://team-b-api.team-b.svc.cluster.local/api/v1/data',
        headers={
            # Propager user context
            'X-User-ID': user_claims['sub'],
            'X-User-Roles': ','.join(user_claims['roles'])
        },
        # mTLS géré automatiquement par Istio sidecar
        timeout=5
    )

    if response.status_code != 200:
        return {"error": "Upstream service error"}, 502

    # Traiter et retourner
    result = process(response.json())

    return {"result": result}
```

**Monitoring Grafana** :

```promql
# Taux de refus par namespace
rate(istio_requests_total{
    response_code="403",
    destination_namespace="team-a"
}[5m])

# Latence ajoutée par authz checks
histogram_quantile(0.95,
    rate(istio_request_duration_seconds_bucket[5m])
) - histogram_quantile(0.95,
    rate(envoy_http_downstream_rq_time_bucket[5m])
)

# Communications inter-namespace (surveiller patterns anormaux)
sum by (source_namespace, destination_namespace) (
    rate(istio_requests_total[5m])
)
```

**Résultats** :

```
Isolation :
- 0 data leaks entre teams vs 3/an avant
- Blast radius d'un incident : 1 namespace vs cluster complet

Compliance :
- Audit de toutes communications inter-services
- Preuve de segregation pour SOC 2

Opérations :
- Temps debug réduit : logs structurés avec identités
- Incidents sécurité détectés : 15 min vs jamais
```

### Cas 4 : Remote workers avec device trust

**Contexte** : Entreprise tech avec 80% remote workers.

**Défi** : Comment sécuriser l'accès aux systèmes internes sans VPN ?

**Solution : Cloudflare Access + WARP**

```
Architecture :

Employee device
  ├─ Cloudflare WARP client (always-on)
  │  ├─ Device posture collection
  │  ├─ Zero Trust tunnel
  │  └─ Split tunneling (only corporate traffic)
  │
  └─ Browser
      │
      ▼
┌─────────────────────┐
│ Cloudflare Access   │
│ (Identity-Aware     │
│  Proxy)             │
└──────┬──────────────┘
       │
   ┌───┼────┬────────┐
   ▼   ▼    ▼        ▼
 [CRM][Git][Wiki][Internal Apps]
```

**Configuration Cloudflare Access** :

```yaml
# Policy example
name: "Engineering Apps Access"
decision: allow

include:
  # Identity
  - email_domain:
      domain: company.com

  # Device posture
  - device_posture:
      integration_uid: "cloudflare-posture-check"
      rule: "managed_device"

  - device_posture:
      integration_uid: "cloudflare-posture-check"
      rule: "os_version_requirement"

  - device_posture:
      integration_uid: "cloudflare-posture-check"
      rule: "antivirus_running"

require:
  # MFA obligatoire
  - authentication_method:
      auth_method: "otp"

exclude:
  # Sauf si pays à risque
  - geo:
      country_code: ["KP", "IR", "SY"]
```

**Device posture checks** :

```javascript
// WARP client - device posture
{
  "device_posture": {
    // OS et patches
    "os": {
      "family": "macOS",
      "version": "14.1",
      "patch_level": "current"
    },

    // Sécurité de base
    "security": {
      "firewall_enabled": true,
      "screen_lock_enabled": true,
      "disk_encryption": "FileVault",
      "auto_lock_timeout": 300
    },

    // Antivirus/EDR
    "endpoint_protection": {
      "antivirus_running": true,
      "antivirus_updated": true,
      "edr_agent": "CrowdStrike Falcon",
      "edr_connected": true
    },

    // Compliance
    "compliance": {
      "domain_joined": true,
      "managed_by_mdm": true,
      "mdm_vendor": "Jamf",
      "certificate_valid": true
    },

    // Risk factors
    "risk": {
      "jailbroken": false,
      "debugger_attached": false,
      "vpn_detected": false,  // Autre VPN non-autorisé
      "tor_detected": false
    }
  }
}
```

**Application avec vérification continue** :

```python
# Backend - continuous verification
from flask import Flask, request
import time

app = Flask(__name__)

class ContinuousVerification:
    def __init__(self):
        self.verification_interval = 300  # 5 minutes
        self.session_store = {}

    def verify_session(self, session_id):
        """
        Vérifier qu'une session est toujours valide
        """
        session = self.session_store.get(session_id)

        if not session:
            return False

        # Vérifier expiration
        if time.time() > session['expires']:
            return False

        # Vérifier que device posture est toujours bon
        device_posture = self.get_current_posture(session['device_id'])

        if not self.is_posture_compliant(device_posture):
            # Device non-compliant, révoquer session
            self.revoke_session(session_id)
            return False

        # Vérifier location (impossible travel?)
        if not self.is_location_valid(session, request.headers):
            return False

        # Session valide, prolonger
        session['last_verified'] = time.time()
        return True

    def is_posture_compliant(self, posture):
        """
        Vérifier posture device
        """
        required_checks = [
            posture.get('firewall_enabled') == True,
            posture.get('antivirus_updated') == True,
            posture.get('disk_encrypted') == True,
            posture.get('os_patch_level') in ['current', 'current-1'],
            posture.get('jailbroken') == False
        ]

        return all(required_checks)

# Middleware
verifier = ContinuousVerification()

@app.before_request
def continuous_auth():
    session_id = request.cookies.get('session_id')

    if not verifier.verify_session(session_id):
        # Force re-authentication
        return redirect('/auth/login?reason=session_invalid')

@app.route('/api/sensitive-data')
def sensitive_data():
    # À ce stade :
    # - User authentifié
    # - Device posture vérifié
    # - Session valide et récente

    # Step-up auth pour données très sensibles
    if requires_step_up_auth(request.path):
        mfa_verified = request.headers.get('X-MFA-Verified')

        if not mfa_verified:
            return {
                "error": "Step-up authentication required",
                "mfa_challenge_url": "/auth/mfa-challenge"
            }, 403

    data = get_sensitive_data()
    return {"data": data}
```

**Résultats après 1 an** :

```
Sécurité :
- Phishing success rate : 12% → 0.8% (MFA + device trust)
- Malware infections : 24/an → 2/an (EDR + posture)
- Account takeovers : 8/an → 0

Productivité :
- VPN connection time : 30s → 0s (WARP always-on)
- VPN issues tickets : 200/mois → 5/mois
- User satisfaction : 6.2/10 → 8.9/10

Coûts :
- VPN infrastructure : $120k/an → $0
- Cloudflare Access : $30k/an
- Net savings : $90k/an
- ROI : 3 mois
```

## Technologies et outils

### Identity Providers (IdP)

```
Provider          Features                Use case
──────────────────────────────────────────────────────────
Okta              Enterprise IAM          Large enterprise
                  MFA, SSO, Lifecycle     100-100k+ users
                  APIs richesse
                  Prix : $$$$

Auth0             Developer-friendly      Startups, tech
(by Okta)         Facile à intégrer       1-10k users
                  Customizable
                  Prix : $$-$$$

Azure AD          Microsoft ecosystem     Microsoft shops
                  Office 365 integration  Any size
                  Conditional Access
                  Prix : $-$$$$

Google            Google Workspace        Google ecosystem
Workspace         Simple, efficace        SMB to enterprise
                  BeyondCorp integration
                  Prix : $-$$

Keycloak          Open source             On-premise
                  Self-hosted             Cost-sensitive
                  Full-featured           Tech-savvy teams
                  Prix : Free (hosting cost)
```

### Zero Trust Network Access (ZTNA)

```
Solution              Strengths              Best for
─────────────────────────────────────────────────────────
Cloudflare Access     Global network         Web apps
                      Easy setup             Public cloud
                      HTTP(S) focus          SMB to enterprise
                      Prix : $$

Zscaler ZPA          Enterprise-grade       Large enterprise
                      Any protocol           Legacy apps
                      Advanced DLP           High security req
                      Prix : $$$$

Palo Alto Prisma     Comprehensive          Enterprise
Access               CASB + ZTNA            Multi-cloud
                      Deep inspection        Complex environments
                      Prix : $$$$

Tailscale            P2P mesh               Dev teams
                      WireGuard-based        Simple use cases
                      Developer-friendly     Startups
                      Prix : Free-$$

Twingate             Moderne, simple        Growing companies
                      Resource-based         Cloud-native apps
                      Good UX                100-10k users
                      Prix : $$
```

### Service Mesh

```
Mesh        Pros                    Cons                Best for
──────────────────────────────────────────────────────────────────
Istio       Feature-rich            Complex             Large K8s
            Mature                  Resource-heavy      Enterprise
            Strong security         Learning curve      Multi-cluster

Linkerd     Lightweight             Less features       Simple K8s
            Easy to use             K8s only            Startups
            Low overhead

Consul      Multi-platform          Complexity          Hybrid
Connect     VM + K8s + ...          HashiCorp lock-in   Multi-platform
            Service discovery

Cilium      eBPF-based             Requires Linux 4.9+  Performance
            Very fast              New (less mature)    Network-heavy
            Network-level
```

### Policy Engines

```
OPA (Open Policy Agent)
- Rego policy language
- Universal (K8s, APIs, CI/CD, etc.)
- CNCF graduated
- Use: Complex authorization logic

Cedar (AWS)
- Simple syntax
- Verified by automated reasoning
- AWS integrated
- Use: AWS environments

Casbin
- Multiple languages (Go, Java, Python, etc.)
- ACL, RBAC, ABAC support
- Use: Application-level authz
```

## Défis et limitations

### 1. Complexité opérationnelle

```python
# Sans Zero Trust : simple firewall
allow tcp from 10.0.0.0/8 to 192.168.1.0/24 port 443

# Avec Zero Trust : beaucoup plus complexe
- Identity provider
- Certificate authority (pour mTLS)
- Policy decision point
- Policy enforcement points (multiples)
- Device trust verification
- Continuous monitoring
- Audit logging
- Secret management
- ... et leurs redondances

Conséquence :
- Équipe plus grande
- Formation nécessaire
- Plus de composants à maintenir
- Plus de points de défaillance potentiels
```

### 2. Latence additionnelle

```
Chaque requête doit :
1. Vérifier JWT (1-5ms)
2. Consulter PDP (5-20ms si cache miss)
3. Vérifier device posture (0ms si cached, 50-200ms sinon)
4. Établir mTLS (0ms si réutilisé, 20-100ms sinon)
5. Logger (1-10ms)

Latence totale ajoutée :
- Best case (tout cached) : 2-10ms
- Worst case (cold start) : 100-350ms

Sur 100 000 requêtes/jour :
- Latence totale ajoutée : 3-50 minutes/jour
```

**Mitigation** :

```python
# Caching agressif
class OptimizedZeroTrust:
    def __init__(self):
        self.jwt_cache = TTLCache(maxsize=10000, ttl=300)
        self.policy_cache = TTLCache(maxsize=1000, ttl=60)
        self.device_cache = TTLCache(maxsize=5000, ttl=300)

    async def verify_request(self, request):
        # Paralléliser les vérifications
        results = await asyncio.gather(
            self.verify_jwt_cached(request.token),
            self.check_policy_cached(request.user, request.resource),
            self.verify_device_cached(request.device_id)
        )

        return all(results)

    # Latency target : <10ms p95
```

### 3. Gestion des certificats

```bash
# mTLS nécessite des certificats pour chaque service
# Problème : rotation, expiration, révocation

Exemple cluster K8s 100 services :
- 100 certificats à générer
- Rotation tous les 30 jours
- 3.3 rotations/jour en moyenne
- 1200 rotations/an

Automatisation obligatoire :
- cert-manager (K8s)
- Vault PKI
- AWS ACM
- Let's Encrypt

Mais :
- Complexité additive
- Points de défaillance
- Monitoring nécessaire
```

### 4. Migration depuis legacy

```
Challenge : Application legacy qui assume réseau de confiance

Exemple : App Java 15 ans d'âge
- Hard-coded IPs
- Assume connexions non-chiffrées
- Pas d'auth entre composants
- État partagé en mémoire (sticky sessions)

Migration vers Zero Trust :
1. Ajouter mTLS → casse l'app (assume HTTP)
2. Ajouter auth → performance issues
3. Stateless → refactor massif

Solution réaliste : Migration progressive
- Phase 1 : IAP devant app (auth user)
- Phase 2 : Service mesh transparent (mTLS)
- Phase 3 : Refactor par composants
- Durée : 12-24 mois
```

### 5. Coût

```
Coût Zero Trust pour 500 employés :

Identity Provider (Okta) : $10/user/mois = $60k/an
ZTNA (Cloudflare Access) : $7/user/mois = $42k/an
EDR (CrowdStrike) : $8/user/mois = $48k/an
SIEM (Splunk) : $50k/an
Ingénierie (2 FTE) : $300k/an

Total : ~$500k/an

vs VPN traditionnel : ~$100k/an

Premium : $400k/an

Justification :
- Réduction incidents : $X00k économisés
- Productivité : $Y00k
- Compliance : requis pour SOC2/ISO27001
- ROI : 18-24 mois typiquement
```

## Best practices

### 1. Commencer petit

```
❌ Ne pas faire :
- Big bang migration
- Tout changer en même temps
- Zero Trust ou rien

✅ Faire :
- Pilote sur 1-2 apps non-critiques
- Mesurer impact
- Itérer
- Étendre progressivement

Timeline réaliste :
Mois 1-3 : Pilote (10% apps)
Mois 4-6 : Early adopters (30%)
Mois 7-12 : Majority (70%)
Mois 13-18 : Legacy difficiles (90%)
Mois 19-24 : Long tail (100%)
```

### 2. Fail open vs fail closed

```python
# Configuration PDP
class PolicyDecisionPoint:
    def __init__(self, fail_mode='closed'):
        self.fail_mode = fail_mode

    def evaluate(self, request):
        try:
            decision = self.consult_policy_engine(request)
            return decision
        except Exception as e:
            log.error(f"PDP error: {e}")

            if self.fail_mode == 'open':
                # ⚠️ Dangereux : autoriser si PDP down
                return Decision(allow=True, reason="PDP unavailable - fail open")
            else:
                # ✅ Sûr : refuser si PDP down
                return Decision(allow=False, reason="PDP unavailable - fail closed")

# Recommandation : fail closed par défaut
# Mais avoir plan de fallback (PDP redondant, cache local, etc.)
```

### 3. Logging et audit

```python
# Logger TOUT pour compliance et forensics
import structlog

logger = structlog.get_logger()

def audit_access_decision(request, decision):
    logger.info(
        "access_decision",
        # Qui
        user_id=request.user_id,
        user_email=request.user_email,
        device_id=request.device_id,
        source_ip=request.source_ip,

        # Quoi
        resource=request.resource,
        action=request.action,

        # Quand
        timestamp=datetime.utcnow().isoformat(),

        # Contexte
        location=request.geoip_location,
        user_agent=request.user_agent,
        risk_score=request.risk_score,

        # Décision
        allowed=decision.allow,
        reason=decision.reason,
        policy_matched=decision.policy_id,

        # Metadata
        session_id=request.session_id,
        request_id=request.request_id
    )

# Retention :
# - Accès autorisés : 1 an minimum
# - Accès refusés : 3 ans (forensics)
# - Accès sensibles : 7 ans (compliance)
```

### 4. Monitoring et alertes

```yaml
# Alertes critiques
alerts:
  - name: HighDenialRate
    expr: |
      rate(access_denied_total[5m]) > 10
    for: 2m
    severity: warning
    message: "High rate of access denials - possible attack or misconfiguration"

  - name: PDPUnavailable
    expr: |
      up{job="pdp"} == 0
    for: 1m
    severity: critical
    message: "PDP down - access decisions failing"

  - name: AnomalousAccess
    expr: |
      access_from_new_location == 1 AND
      access_to_sensitive_resource == 1
    severity: high
    message: "Access to sensitive resource from new location"

  - name: PossibleCredentialStuffing
    expr: |
      rate(auth_failures_total[1m]) > 50
    for: 30s
    severity: high
    message: "High rate of authentication failures"
```

### 5. Progressive rollout

```python
# Feature flag pour Zero Trust
class ZeroTrustFeatureFlag:
    def __init__(self):
        self.rollout_percentage = 0  # Start at 0%

    def should_enforce_zero_trust(self, user):
        # Commencer par internal users
        if user.is_employee:
            return True

        # Puis rollout progressif external users
        user_hash = hash(user.id)
        return (user_hash % 100) < self.rollout_percentage

    def increase_rollout(self, percentage):
        # Augmenter progressivement : 1%, 5%, 10%, 25%, 50%, 100%
        self.rollout_percentage = min(100, percentage)

# Middleware
@app.before_request
def maybe_enforce_zero_trust():
    if feature_flag.should_enforce_zero_trust(request.user):
        return enforce_zero_trust(request)
    else:
        # Legacy auth path
        return legacy_auth(request)
```

## Futur de Zero Trust

### 1. AI/ML dans les décisions d'accès

```python
# Futur : ML model pour décisions d'accès
class MLEnhancedPDP:
    def __init__(self):
        self.model = load_ml_model('risk_scoring_v2.pkl')

    def evaluate(self, request):
        # Features pour ML
        features = {
            'time_of_day': request.timestamp.hour,
            'day_of_week': request.timestamp.weekday(),
            'location_entropy': calculate_entropy(request.location_history),
            'access_frequency': get_access_frequency(request.user_id),
            'resource_sensitivity': get_sensitivity(request.resource),
            'device_trust_score': request.device_trust_score,
            'behavioral_score': get_behavioral_score(request.user_id),
            # ... 50+ features
        }

        # Prédiction ML : probabilité que l'accès soit légitime
        legitimacy_score = self.model.predict_proba(features)[0][1]

        if legitimacy_score > 0.95:
            return Decision(allow=True, confidence='high')
        elif legitimacy_score > 0.70:
            return Decision(allow=True, confidence='medium', require_mfa=True)
        else:
            return Decision(allow=False, reason='Low legitimacy score', request_review=True)
```

### 2. Passwordless + Zero Trust

```
Tendance : Élimination des passwords

FIDO2/WebAuthn + Passkeys :
- Biométrie (Face ID, Touch ID)
- Hardware keys (Yubikey)
- Plateforme-native (Apple/Google/Microsoft)

Combiné avec Zero Trust :
- Authentication forte (what you have + who you are)
- Device binding automatique
- Phishing-resistant
- UX améliorée

Adoption :
- Apple Passkeys (iOS 16+, macOS Ventura+)
- Google Passkeys (Android, Chrome)
- Microsoft Authenticator
- Prédiction : 50%+ adoption d'ici 2027
```

### 3. Zero Trust pour IoT

```
Challenge IoT :
- Milliards de devices
- Peu de compute/mémoire
- Lifespans : 10-20 ans
- Firmware rarement updatable

Zero Trust IoT :
- Lightweight protocols (MQTT-TLS, CoAP-DTLS)
- Hardware root of trust (TPM, Secure Element)
- OTA updates obligatoires
- Segmentation stricte (1 device = 1 micro-segment)

Standards émergents :
- Matter (smart home)
- Thread (mesh networking)
- PSA Certified (ARM security)
```

### 4. Quantum-resistant Zero Trust

```
Menace quantique :
- Ordinateurs quantiques cassent RSA/ECC
- TLS/mTLS actuel vulnérable
- Timeline : 10-15 ans

Préparation :
- Post-Quantum Cryptography (PQC)
- NIST a standardisé algorithmes (2024)
- Migration commencera ~2026-2028

Zero Trust quantum-safe :
- Certificats PQC pour mTLS
- Signatures PQC pour JWT
- Hybrid mode (classique + PQC) pour transition
```

## Conclusion

Zero Trust n'est pas une technologie, c'est un **changement de philosophie** : ne faire confiance à personne ni à rien par défaut, vérifier explicitement chaque accès, appliquer le principe du moindre privilège.

**Pour les développeurs en 2025** :

✅ **Incontournable** : Les apps modernes doivent être conçues Zero Trust-first

✅ **Impact direct** : Authentification, autorisation, chiffrement, logging sont au cœur du code

✅ **Complexité gérable** : Les outils (Istio, Cloudflare, Okta) simplifient l'implémentation

✅ **ROI prouvé** : Sécurité améliorée + productivité + compliance

**Actions immédiates** :

1. **Auditer** votre architecture actuelle : où est la confiance implicite ?
2. **Implémenter mTLS** entre microservices
3. **Ajouter JWT riches** avec contexte device/user
4. **Logger** toutes les décisions d'accès
5. **Former** l'équipe aux principes Zero Trust

Le périmètre de sécurité est mort. Zero Trust est l'avenir. Et cet avenir est déjà là.

---

**Ressources complémentaires** :
- [NIST SP 800-207 - Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [BeyondCorp: A New Approach to Enterprise Security](https://cloud.google.com/beyondcorp) (Google)
- [CISA Zero Trust Maturity Model](https://www.cisa.gov/zero-trust-maturity-model)
- [Okta Zero Trust Security](https://www.okta.com/zero-trust/)

---


⏭️ [Edge computing et implications réseau](/10-evolutions-tendances/05-edge-computing.md)
