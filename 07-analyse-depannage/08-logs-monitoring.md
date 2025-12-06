🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.8 Logs et monitoring

## Introduction

Wireshark est un outil extraordinaire pour l'analyse ponctuelle, mais il ne peut pas tout faire. Pour une surveillance continue, une détection proactive des problèmes, et une vision à long terme de la santé réseau, nous avons besoin de **logs** et de **monitoring**.

Cette section complète le module d'analyse et dépannage en montrant comment :
- Collecter et analyser les **logs réseau** de différentes sources
- Mettre en place un **monitoring** efficace
- **Corréler** les logs avec les captures Wireshark
- Détecter les problèmes **avant** qu'ils n'impactent les utilisateurs
- Créer des **dashboards** pour visualiser la santé réseau
- Implémenter des **alertes** intelligentes

Les logs et le monitoring sont complémentaires à Wireshark : ils fournissent le contexte historique et la vue d'ensemble, tandis que Wireshark permet l'analyse fine au niveau paquet.

---

## Types de logs réseau

### 1. Logs système (OS)

**Linux - systemd/journald :**
```bash
# Logs réseau généraux
journalctl -u NetworkManager
journalctl -u systemd-networkd

# Logs d'une interface spécifique
journalctl -k | grep eth0

# Logs firewall
journalctl -u firewalld
journalctl -u ufw

# Logs depuis une heure
journalctl --since "1 hour ago"

# Logs avec niveau de priorité
journalctl -p err  # Erreurs uniquement
```

**Exemple de sortie :**
```
Dec 07 14:30:15 server1 NetworkManager[1234]: <info>  [eth0]: device state changed: activated
Dec 07 14:30:45 server1 kernel: [12345.678] eth0: link down
Dec 07 14:30:50 server1 kernel: [12350.123] eth0: link up, 1000Mbps, full duplex
Dec 07 14:31:02 server1 firewalld[5678]: WARNING: COMMAND_FAILED: '/usr/sbin/iptables -w10 -D ...'
```

**Linux - syslog traditionnel :**
```bash
# Messages système généraux
tail -f /var/log/messages
tail -f /var/log/syslog

# Messages kernel (réseau inclus)
tail -f /var/log/kern.log

# Chercher des événements réseau
grep -i "eth0\|network\|link" /var/log/syslog
grep -i "iptables\|firewall" /var/log/messages
```

**Windows - Event Viewer :**
```
Applications and Services Logs → Microsoft → Windows → NetworkProfile
System → Filter by Source: "Tcpip", "NETIO", "Dhcp"

EventID importants :
4201 : Connectivité réseau détectée
4202 : Connectivité réseau perdue
10016 : Erreur réseau
```

**macOS :**
```bash
# Logs réseau
log show --predicate 'subsystem == "com.apple.network"' --last 1h

# Logs système
log show --last 1h | grep -i network
```

### 2. Logs d'applications réseau

**Serveur web (nginx) :**

**Access log :**
```
# /var/log/nginx/access.log
192.168.1.50 - - [07/Dec/2025:14:30:15 +0100] "GET /api/users HTTP/1.1" 200 1234 "-" "curl/7.68.0"
192.168.1.50 - - [07/Dec/2025:14:30:16 +0100] "POST /api/login HTTP/1.1" 401 89 "-" "Mozilla/5.0"
203.0.113.45 - - [07/Dec/2025:14:30:17 +0100] "GET /images/logo.png HTTP/1.1" 200 45678 "https://example.com/" "Mozilla/5.0"
```

**Format personnalisé avec timing :**
```nginx
log_format timing '$remote_addr - $remote_user [$time_local] '
                  '"$request" $status $body_bytes_sent '
                  '"$http_referer" "$http_user_agent" '
                  'rt=$request_time uct="$upstream_connect_time" '
                  'uht="$upstream_header_time" urt="$upstream_response_time"';

# Sortie :
192.168.1.50 - - [07/Dec/2025:14:30:15 +0100] "GET /api/data HTTP/1.1" 200 5678
"-" "curl/7.68.0" rt=0.856 uct="0.002" uht="0.854" urt="0.854"
```

**Interprétation :**
```
rt (request_time) : 856ms total
uct (upstream_connect_time) : 2ms pour se connecter au backend
uht (upstream_header_time) : 854ms pour recevoir les headers
urt (upstream_response_time) : 854ms pour la réponse complète

Diagnostic : Backend lent (854ms), pas le réseau (2ms de connexion)
```

**Error log :**
```
# /var/log/nginx/error.log
2025/12/07 14:30:20 [error] 12345#12345: *678 connect() failed (111: Connection refused)
while connecting to upstream, client: 192.168.1.50, server: api.example.com,
request: "GET /api/users HTTP/1.1", upstream: "http://10.0.5.20:8080/api/users"

2025/12/07 14:31:05 [error] 12345#12345: *679 upstream timed out (110: Connection timed out)
while reading response header from upstream, client: 192.168.1.50

2025/12/07 14:32:15 [warn] 12345#12345: *680 an upstream response is buffered to a temporary file
/var/cache/nginx/proxy_temp/1/23/0000000231 while reading upstream
```

**Serveur web (Apache) :**
```
# /var/log/apache2/access.log (format combined)
192.168.1.50 - - [07/Dec/2025:14:30:15 +0100] "GET /index.html HTTP/1.1" 200 2345
"-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/91.0.4472.124"

# Format personnalisé avec timing
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" combined_time

# %D = temps en microsecondes
192.168.1.50 - - [07/Dec/2025:14:30:15 +0100] "GET /api/data HTTP/1.1" 200 5678 "-" "curl/7.68.0" 856234
                                                                                                    ^^^^^^
                                                                                                    856ms
```

**Base de données (MySQL) :**
```
# /var/log/mysql/error.log
2025-12-07T14:30:15.123456Z 123 [Note] Aborted connection 123 to db: 'production' user: 'webapp'
host: '192.168.1.50' (Got timeout reading communication packets)

2025-12-07T14:31:20.234567Z 124 [Warning] IP address '192.168.1.60' could not be resolved:
Name or service not known

# Slow query log
# /var/log/mysql/mysql-slow.log
# Time: 2025-12-07T14:30:15.123456Z
# User@Host: webapp[webapp] @ server1.local [192.168.1.50]
# Query_time: 5.234567  Lock_time: 0.000123 Rows_sent: 1000  Rows_examined: 50000
SET timestamp=1701955815;
SELECT * FROM users WHERE status='active' ORDER BY created_at DESC;
```

**DNS (BIND) :**
```
# /var/log/named/queries.log
07-Dec-2025 14:30:15.123 queries: info: client 192.168.1.50#54321 (example.com):
query: example.com IN A + (192.168.1.1)

07-Dec-2025 14:30:15.234 queries: info: client 192.168.1.50#54322 (www.example.com):
query: www.example.com IN A + (192.168.1.1)

# /var/log/named/named.log
07-Dec-2025 14:30:20.123 resolver: info: lame server resolving 'badzone.example'
(in 'example'?): 203.0.113.50#53
```

### 3. Logs d'équipements réseau

**Routeur/Switch (Cisco) :**
```
Dec  7 14:30:15.123: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1,
changed state to down

Dec  7 14:30:20.456: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up

Dec  7 14:30:25.789: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1,
changed state to up

Dec  7 14:31:00.123: %SEC-6-IPACCESSLOGP: list 100 denied tcp 203.0.113.45(12345) ->
192.168.1.10(22), 1 packet
```

**Firewall (iptables logs) :**
```
# /var/log/kern.log ou journalctl
Dec  7 14:30:15 server1 kernel: [12345.678] [IPTABLES DROP] IN=eth0 OUT=
MAC=00:1a:2b:3c:4d:5e:aa:bb:cc:dd:ee:ff:08:00 SRC=203.0.113.45 DST=192.168.1.10
LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=12345 DF PROTO=TCP SPT=54321 DPT=22
WINDOW=29200 RES=0x00 SYN URGP=0
```

**Configuration pour logger les drops :**
```bash
# Créer une chaîne de logging
iptables -N LOGGING

# Rediriger vers la chaîne de logging
iptables -A INPUT -j LOGGING

# Logger avec préfixe
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "[IPTABLES DROP] " --log-level 4

# Puis drop
iptables -A LOGGING -j DROP
```

**Firewall (pfSense/OPNsense) :**
```
Dec 7 14:30:15 filterlog: 5,,,1000000103,em0,match,block,in,4,0x0,,64,12345,0,DF,6,tcp,60,
203.0.113.45,192.168.1.10,54321,22,0,S,1234567890,,29200,,mss;nop;wscale;sackOK;TS

Format :
rule_number, sub_rule, anchor, tracker, interface, reason, action, direction,
ip_version, tos, ecn, ttl, id, offset, flags, protocol_id, protocol, length,
src_ip, dst_ip, src_port, dst_port, data_length, tcp_flags, sequence, ack, window,
urg, options
```

### 4. Logs DHCP

**ISC DHCP Server :**
```
# /var/log/dhcpd.log
Dec  7 14:30:15 dhcpd: DHCPDISCOVER from 00:1a:2b:3c:4d:5e via eth0
Dec  7 14:30:15 dhcpd: DHCPOFFER on 192.168.1.100 to 00:1a:2b:3c:4d:5e via eth0
Dec  7 14:30:15 dhcpd: DHCPREQUEST for 192.168.1.100 (192.168.1.1) from 00:1a:2b:3c:4d:5e via eth0
Dec  7 14:30:15 dhcpd: DHCPACK on 192.168.1.100 to 00:1a:2b:3c:4d:5e via eth0

Dec  7 14:35:20 dhcpd: DHCPRELEASE of 192.168.1.100 from 00:1a:2b:3c:4d:5e via eth0 (found)
```

**dnsmasq :**
```
Dec  7 14:30:15 dnsmasq-dhcp[1234]: DHCPDISCOVER(eth0) 00:1a:2b:3c:4d:5e
Dec  7 14:30:15 dnsmasq-dhcp[1234]: DHCPOFFER(eth0) 192.168.1.100 00:1a:2b:3c:4d:5e
Dec  7 14:30:15 dnsmasq-dhcp[1234]: DHCPREQUEST(eth0) 192.168.1.100 00:1a:2b:3c:4d:5e
Dec  7 14:30:15 dnsmasq-dhcp[1234]: DHCPACK(eth0) 192.168.1.100 00:1a:2b:3c:4d:5e laptop-alice
```

---

## Centralisation des logs

### Syslog centralisé

**Architecture :**
```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│  Serveur 1   │──────>│              │       │              │
│  (client)    │       │   Serveur    │──────>│  Stockage    │
└──────────────┘       │   Syslog     │       │  (ELK, etc)  │
                       │  (rsyslog)   │       │              │
┌──────────────┐       │              │       └──────────────┘
│  Serveur 2   │──────>│              │
│  (client)    │       └──────────────┘
└──────────────┘
```

**Configuration rsyslog (client) :**

```bash
# /etc/rsyslog.d/50-default.conf

# Envoyer tous les logs au serveur central
*.* @192.168.1.200:514          # UDP
*.* @@192.168.1.200:514         # TCP (plus fiable)

# Ou seulement certains logs
kern.* @@192.168.1.200:514
auth.* @@192.168.1.200:514
```

**Configuration rsyslog (serveur) :**

```bash
# /etc/rsyslog.conf

# Activer réception UDP
module(load="imudp")
input(type="imudp" port="514")

# Activer réception TCP
module(load="imtcp")
input(type="imtcp" port="514")

# Template pour organiser par hôte
$template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs

# Arrêter le traitement pour ne pas dupliquer
& stop
```

**Redémarrer rsyslog :**
```bash
systemctl restart rsyslog
```

**Vérifier réception :**
```bash
# Sur le serveur
tail -f /var/log/remote/server1/kernel.log
tail -f /var/log/remote/server2/nginx.log
```

### ELK Stack (Elasticsearch, Logstash, Kibana)

**Architecture :**
```
Sources de logs
    │
    ├─> Filebeat/Fluentd ──> Logstash ──> Elasticsearch ──> Kibana
    │                         (parsing)    (stockage)      (visualisation)
    └─> Syslog
```

**Installation (Docker Compose simplifié) :**

```yaml
# docker-compose.yml
version: '3.7'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    ports:
      - "5000:5000"
      - "5044:5044"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

**Configuration Logstash :**

```ruby
# logstash.conf
input {
  # Recevoir syslog
  syslog {
    port => 5000
    type => "syslog"
  }

  # Recevoir depuis Filebeat
  beats {
    port => 5044
  }
}

filter {
  # Parser les logs nginx
  if [type] == "nginx-access" {
    grok {
      match => {
        "message" => '%{IPORHOST:remote_addr} - %{USERNAME:remote_user} \[%{HTTPDATE:time_local}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status} %{NUMBER:body_bytes_sent} "%{DATA:http_referer}" "%{DATA:http_user_agent}" rt=%{NUMBER:request_time} uct="%{NUMBER:upstream_connect_time}" uht="%{NUMBER:upstream_header_time}" urt="%{NUMBER:upstream_response_time}"'
      }
    }

    mutate {
      convert => {
        "status" => "integer"
        "body_bytes_sent" => "integer"
        "request_time" => "float"
        "upstream_response_time" => "float"
      }
    }
  }

  # Parser les logs firewall iptables
  if [type] == "syslog" and [message] =~ /IPTABLES/ {
    grok {
      match => {
        "message" => '\[IPTABLES %{WORD:action}\].*SRC=%{IP:src_ip} DST=%{IP:dst_ip}.*PROTO=%{WORD:protocol} SPT=%{NUMBER:src_port} DPT=%{NUMBER:dst_port}'
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{type}-%{+YYYY.MM.dd}"
  }

  # Debug
  stdout { codec => rubydebug }
}
```

**Configuration Filebeat (sur serveurs sources) :**

```yaml
# /etc/filebeat/filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    fields:
      type: nginx-access

  - type: log
    enabled: true
    paths:
      - /var/log/nginx/error.log
    fields:
      type: nginx-error

  - type: log
    enabled: true
    paths:
      - /var/log/syslog
    fields:
      type: syslog

output.logstash:
  hosts: ["192.168.1.200:5044"]
```

**Visualisation dans Kibana :**
```
1. Accéder à http://kibana-server:5601
2. Management → Index Patterns → Create
3. Index pattern: logs-*
4. Discover → Voir tous les logs
5. Créer des visualisations et dashboards
```

---

## Corrélation logs et Wireshark

### Scénario : Débugger une erreur HTTP 502

**1. Détecter le problème dans les logs :**

```bash
# Nginx error log
grep "502" /var/log/nginx/error.log

# Sortie :
2025/12/07 14:30:15 [error] 12345#12345: *678 connect() failed (111: Connection refused)
while connecting to upstream, client: 192.168.1.50, server: api.example.com,
request: "GET /api/users HTTP/1.1", upstream: "http://10.0.5.20:8080/api/users",
host: "api.example.com"
```

**Informations clés :**
```
Timestamp : 2025/12/07 14:30:15
Client : 192.168.1.50
Backend : 10.0.5.20:8080
Erreur : Connection refused (port fermé ou service down)
```

**2. Capturer le trafic Wireshark à ce moment :**

```bash
# Capture autour de 14:30:15 (si capture continue était active)
# Ou déclencher nouvelle capture

# Filtrer dans Wireshark
ip.addr == 10.0.5.20 and tcp.port == 8080 and frame.time >= "2025-12-07 14:30:10" and frame.time <= "2025-12-07 14:30:20"
```

**3. Analyser dans Wireshark :**

```
No.  Time      Source        Destination   Protocol  Info
100  14:30:15  nginx_server  10.0.5.20     TCP       54321 → 8080 [SYN] Seq=0
101  14:30:15  10.0.5.20     nginx_server  TCP       8080 → 54321 [RST, ACK]
```

**4. Conclusion :**
```
Wireshark confirme les logs :
- SYN envoyé par nginx
- RST immédiat du backend
→ Port 8080 fermé sur 10.0.5.20
→ Service backend pas démarré
```

**5. Vérifier sur le backend :**
```bash
ssh 10.0.5.20
systemctl status app-backend
# Output : inactive (dead)

systemctl start app-backend
systemctl status app-backend
# Output : active (running) ✓
```

**6. Valider dans les logs :**
```bash
# Nginx access log - nouvelles requêtes
tail -f /var/log/nginx/access.log
192.168.1.50 - - [07/Dec/2025:14:35:00 +0100] "GET /api/users HTTP/1.1" 200 5678 ✓
```

### Template de corrélation

**Workflow type :**
```
1. LOGS : Détecter anomalie (timestamp, IPs, erreur)
2. WIRESHARK : Capturer au moment exact
3. ANALYSE : Confirmer/infirmer hypothèse
4. LOGS SYSTÈME : Vérifier état des services
5. WIRESHARK : Re-tester après correction
6. LOGS : Valider résolution
```

---

## Outils de monitoring

### Prometheus + Grafana

**Architecture :**
```
┌──────────────────────────────────────────────────────┐
│                    Grafana                           │
│               (Dashboards, Alerting)                 │
└───────────────────┬──────────────────────────────────┘
                    │ (queries)
┌───────────────────▼──────────────────────────────────┐
│                 Prometheus                           │
│              (Time Series DB)                        │
└───┬───────┬───────┬───────┬───────┬──────────────────┘
    │       │       │       │       │
    │       │       │       │       │ (scrape metrics)
    ▼       ▼       ▼       ▼       ▼
  Node    Node   Blackbox  SNMP   Custom
Exporter Exporter Exporter Exporter Exporter
  (CPU)   (App)   (HTTP)   (Switch) (...)
```

**Prometheus - Configuration :**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Surveiller les serveurs (node_exporter)
  - job_name: 'node'
    static_configs:
      - targets:
        - 'server1:9100'
        - 'server2:9100'
        - 'server3:9100'

  # Surveiller la disponibilité HTTP (blackbox_exporter)
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://api.example.com
        - https://www.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  # Surveiller les équipements réseau (snmp_exporter)
  - job_name: 'snmp'
    static_configs:
      - targets:
        - 192.168.1.1  # Routeur
        - 192.168.1.2  # Switch
    metrics_path: /snmp
    params:
      module: [if_mib]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: snmp-exporter:9116

alerting:
  alertmanagers:
    - static_configs:
      - targets:
        - 'alertmanager:9093'

rule_files:
  - 'alerts.yml'
```

**Alertes Prometheus :**

```yaml
# alerts.yml
groups:
  - name: network
    interval: 30s
    rules:
      # Alerte si serveur down
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been down for more than 5 minutes"

      # Alerte si latence HTTP élevée
      - alert: HighLatency
        expr: probe_duration_seconds > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High HTTP latency on {{ $labels.instance }}"
          description: "{{ $labels.instance }} has latency > 2s"

      # Alerte si bande passante saturée
      - alert: HighNetworkTraffic
        expr: rate(node_network_receive_bytes_total[5m]) > 100000000  # 100 MB/s
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High network traffic on {{ $labels.instance }}"
          description: "Interface receiving > 100 MB/s"

      # Alerte si perte de paquets
      - alert: PacketLoss
        expr: rate(node_network_receive_drop_total[5m]) > 100
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Packet loss detected on {{ $labels.instance }}"
          description: "{{ $labels.device }} dropping packets"
```

**Grafana - Dashboard réseau :**

```json
{
  "dashboard": {
    "title": "Network Monitoring",
    "panels": [
      {
        "title": "Network Traffic",
        "targets": [
          {
            "expr": "rate(node_network_receive_bytes_total[5m])*8",
            "legendFormat": "{{ instance }} - RX"
          },
          {
            "expr": "rate(node_network_transmit_bytes_total[5m])*8",
            "legendFormat": "{{ instance }} - TX"
          }
        ],
        "yaxis": {
          "format": "bps"
        }
      },
      {
        "title": "HTTP Response Time",
        "targets": [
          {
            "expr": "probe_duration_seconds",
            "legendFormat": "{{ instance }}"
          }
        ],
        "yaxis": {
          "format": "s"
        }
      },
      {
        "title": "Packet Loss",
        "targets": [
          {
            "expr": "rate(node_network_receive_drop_total[5m])",
            "legendFormat": "{{ instance }} - {{ device }}"
          }
        ]
      }
    ]
  }
}
```

### Nagios

**Installation et configuration :**

```bash
# Installation (Debian/Ubuntu)
apt-get install nagios4 nagios-plugins-contrib

# Configuration d'un hôte
# /etc/nagios4/conf.d/servers/server1.cfg
define host {
    use                     linux-server
    host_name               server1
    alias                   Production Server 1
    address                 192.168.1.10
    max_check_attempts      5
    check_period            24x7
    notification_interval   30
    notification_period     24x7
}

# Services à monitorer
define service {
    use                     generic-service
    host_name               server1
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}

define service {
    use                     generic-service
    host_name               server1
    service_description     HTTP
    check_command           check_http
}

define service {
    use                     generic-service
    host_name               server1
    service_description     SSH
    check_command           check_ssh
}

define service {
    use                     generic-service
    host_name               server1
    service_description     Network Bandwidth
    check_command           check_snmp_bandwidth!public!1!80!90
}
```

**Commandes personnalisées :**

```bash
# /etc/nagios-plugins/config/custom.cfg
define command {
    command_name    check_tcp_response_time
    command_line    /usr/lib/nagios/plugins/check_tcp -H $HOSTADDRESS$ -p $ARG1$ -w $ARG2$ -c $ARG3$
}

define command {
    command_name    check_dns_response_time
    command_line    /usr/lib/nagios/plugins/check_dns -H $ARG1$ -s $HOSTADDRESS$ -w $ARG2$ -c $ARG3$
}

# Utilisation
define service {
    use                     generic-service
    host_name               server1
    service_description     MySQL Response Time
    check_command           check_tcp_response_time!3306!0.1!0.5
}
```

### Zabbix

**Architecture :**
```
┌────────────────┐
│  Zabbix Server │ ← Base de données
└────────┬───────┘
         │
    ┌────┴────┬────────┬────────┐
    ▼         ▼        ▼        ▼
  Agent     Agent    SNMP    IPMI
 (Linux)   (Win)   (Switch) (Hardware)
```

**Configuration template réseau :**

```xml
<!-- Template_Network.xml -->
<zabbix_export>
  <templates>
    <template>
      <name>Network Monitoring</name>
      <items>
        <item>
          <name>ICMP ping</name>
          <type>SIMPLE</type>
          <key>icmpping</key>
          <delay>60s</delay>
          <triggers>
            <trigger>
              <expression>{Template_Network:icmpping.max(#3)}=0</expression>
              <name>Host unreachable</name>
              <priority>HIGH</priority>
            </trigger>
          </triggers>
        </item>

        <item>
          <name>ICMP response time</name>
          <type>SIMPLE</type>
          <key>icmppingsec</key>
          <delay>60s</delay>
          <units>s</units>
          <triggers>
            <trigger>
              <expression>{Template_Network:icmppingsec.avg(5m)}>0.15</expression>
              <name>High ICMP ping response time</name>
              <priority>WARNING</priority>
            </trigger>
          </triggers>
        </item>

        <item>
          <name>Network interface incoming traffic</name>
          <type>SNMP</type>
          <snmp_oid>IF-MIB::ifInOctets.{#SNMPINDEX}</snmp_oid>
          <key>net.if.in[{#IFNAME}]</key>
          <delay>60s</delay>
          <units>bps</units>
          <preprocessing>
            <step>
              <type>CHANGE_PER_SECOND</type>
            </step>
            <step>
              <type>MULTIPLIER</type>
              <params>8</params>
            </step>
          </preprocessing>
        </item>
      </items>
    </template>
  </templates>
</zabbix_export>
```

---

## Métriques réseau à surveiller

### Métriques de disponibilité

**1. Uptime / Downtime**
```
Mesure : Pourcentage de temps en ligne
Target : 99.9% (43 min/mois de downtime max)
        99.99% (4.3 min/mois)
        99.999% (26 sec/mois)

Prometheus query :
up{job="node"}
```

**2. Connectivité ICMP**
```
Mesure : Réponse au ping
Alert : Si 3 pings consécutifs échouent

Prometheus query :
probe_success{job="blackbox"}
```

### Métriques de performance

**1. Latence (RTT)**
```
Mesure : Temps aller-retour ICMP ou TCP
Seuils :
  < 50ms : OK
  50-150ms : Warning
  > 150ms : Critical

Prometheus query :
probe_duration_seconds{job="blackbox"}
probe_icmp_duration_seconds
```

**2. Jitter**
```
Mesure : Variation de latence
Important pour : VoIP, vidéo, jeux
Seuils :
  < 30ms : OK
  30-50ms : Warning
  > 50ms : Critical

Calcul : Écart-type des latences sur 1 minute
```

**3. Débit (Throughput)**
```
Mesure : Bits par seconde entrants/sortants
Seuils : Basés sur capacité du lien

Prometheus query :
rate(node_network_receive_bytes_total[5m]) * 8  # RX en bps
rate(node_network_transmit_bytes_total[5m]) * 8 # TX en bps

Alert si > 80% de capacité du lien
```

**4. Utilisation de bande passante**
```
Mesure : Pourcentage d'utilisation
Calcul : (Débit actuel / Capacité lien) × 100%

Alert :
  > 80% : Warning (risque de congestion)
  > 90% : Critical
```

### Métriques de fiabilité

**1. Perte de paquets**
```
Mesure : Pourcentage de paquets perdus
Seuils :
  < 0.1% : OK
  0.1-1% : Warning
  > 1% : Critical

Prometheus query :
rate(node_network_receive_drop_total[5m])
rate(node_network_transmit_drop_total[5m])
```

**2. Erreurs réseau**
```
Mesure : Erreurs CRC, collisions, etc.
Seuils : Toute augmentation = investigation

Prometheus query :
rate(node_network_receive_errs_total[5m])
rate(node_network_transmit_errs_total[5m])
```

**3. Retransmissions TCP**
```
Mesure : Segments TCP retransmis
Indicateur : Problème de congestion/perte

Linux :
netstat -s | grep retransmit
ss -ti  # Voir retransmissions par connexion

SNMP :
tcpRetransSegs
```

### Métriques de capacité

**1. Connexions TCP actives**
```
Mesure : Nombre de connexions établies
Alert : Si proche de la limite système

Prometheus query :
node_netstat_Tcp_CurrEstab

Linux limit :
sysctl net.ipv4.tcp_max_syn_backlog
sysctl net.core.somaxconn
```

**2. États TCP**
```
Surveiller :
- ESTABLISHED : Connexions actives
- TIME_WAIT : Connexions fermées (attente cleanup)
- CLOSE_WAIT : Application n'a pas fermé (fuite !)

Alert si :
- TIME_WAIT > 10,000 (peut manquer de ports)
- CLOSE_WAIT croissant (fuite connexions)

Query :
node_netstat_Tcp_CurrEstab      # ESTABLISHED
node_netstat_TcpExt_TCPTimeWaitOverflow
```

**3. Utilisation de ports éphémères**
```
Linux range :
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768    60999 (28,232 ports disponibles)

Alert si > 80% utilisés :
ss -tan | grep -c ESTABLISHED > 22000
```

### Métriques applicatives

**1. Temps de réponse HTTP**
```
Mesure : De la requête à la réponse complète
Seuils :
  < 200ms : Excellent
  200-500ms : Bon
  500ms-2s : Acceptable
  > 2s : Problème

Prometheus (blackbox_exporter) :
probe_http_duration_seconds

Nginx logs :
$request_time
```

**2. Temps de résolution DNS**
```
Mesure : Temps pour résoudre un nom
Seuils :
  < 50ms : OK
  50-200ms : Warning
  > 200ms : Critical

Prometheus query :
probe_dns_lookup_time_seconds
```

**3. Taux d'erreur HTTP**
```
Mesure : Pourcentage de 4xx/5xx
Seuils :
  < 1% : OK
  1-5% : Warning
  > 5% : Critical

Prometheus (depuis nginx/apache logs) :
rate(http_requests_total{status=~"5.."}[5m]) /
rate(http_requests_total[5m]) * 100
```

---

## Dashboard réseau complet

### Vue d'ensemble (exemple Grafana)

**Panneau 1 : Disponibilité**
```
┌─────────────────────────────────────────────────┐
│ Uptime (30 days)                                │
│                                                 │
│ server1.example.com      ████████████ 99.95%    │
│ server2.example.com      ████████████ 99.98%    │
│ api.example.com          ████████████ 99.99%    │
│ db.example.com           ████████████ 99.92%    │
└─────────────────────────────────────────────────┘
```

**Panneau 2 : Latence en temps réel**
```
┌─────────────────────────────────────────────────┐
│ Latency (ms)                                    │
│  150 ┤                                          │
│  100 ┤     ╭╮                                   │
│   50 ┤─────╯╰──────────────                     │
│    0 └──┬────┬────┬────┬────┬────►              │
│       14:00 14:15 14:30 14:45 15:00             │
└─────────────────────────────────────────────────┘
```

**Panneau 3 : Trafic réseau**
```
┌─────────────────────────────────────────────────┐
│ Network Traffic (Mbps)                          │
│  1000 ┤                                         │
│   800 ┤          ╭────╮                         │
│   600 ┤        ╭─╯    ╰─╮                       │
│   400 ┤    ╭───╯        ╰───╮                   │
│   200 ┤────╯                ╰───                │
│     0 └──┬────┬────┬────┬────┬────►             │
│        14:00 14:15 14:30 14:45 15:00            │
│        ─── RX   ─── TX                          │
└─────────────────────────────────────────────────┘
```

**Panneau 4 : Top talkers**
```
┌─────────────────────────────────────────────────┐
│ Top 5 Bandwidth Consumers                       │
│                                                 │
│ 1. backup-server     ████████████ 850 Mbps      │
│ 2. video-stream      ██████       520 Mbps      │
│ 3. web-server        ████         340 Mbps      │
│ 4. app-server        ██           180 Mbps      │
│ 5. db-server         █            90 Mbps       │
└─────────────────────────────────────────────────┘
```

**Panneau 5 : Alertes actives**
```
┌─────────────────────────────────────────────────┐
│ Active Alerts                                   │
│                                                 │
│ ⚠️  High latency on api.example.com (245ms)     │
│ ⚠️  Packet loss on eth1 (1.2%)                  │
│ 🔴 server3.example.com unreachable              │
└─────────────────────────────────────────────────┘
```

---

## Alerting intelligent

### Éviter les faux positifs

**Problème des alertes naïves :**
```
Alert si latence > 100ms

Résultat :
- 14:30:05 : Alert (spike temporaire à 120ms)
- 14:30:15 : Alert resolved
- 14:30:25 : Alert (nouveau spike à 110ms)
- 14:30:35 : Alert resolved
...

→ Tempête d'alertes pour fluctuations normales
```

**Solution : Seuils avec hysteresis**
```yaml
# Prometheus
- alert: HighLatency
  expr: probe_duration_seconds > 0.1
  for: 5m  # ✅ Doit persister 5 minutes
  labels:
    severity: warning
```

**Solution : Moyenne mobile**
```yaml
- alert: HighLatencyAverage
  expr: avg_over_time(probe_duration_seconds[10m]) > 0.1
  for: 2m
  labels:
    severity: warning
```

### Niveaux de sévérité

**Hiérarchie recommandée :**
```
INFO      : Événement notable mais pas de problème
            Ex: Serveur redémarré

WARNING   : Problème potentiel, investigation recommandée
            Ex: Latence élevée, utilisation > 80%

CRITICAL  : Problème urgent, impact utilisateur
            Ex: Service down, erreurs > 5%

EMERGENCY : Panne majeure, multiples services affectés
            Ex: Datacenter inaccessible
```

**Exemple Prometheus :**
```yaml
groups:
  - name: network_alerts
    rules:
      # INFO : Redémarrage réseau
      - alert: NetworkInterfaceFlap
        expr: changes(node_network_up[5m]) > 2
        labels:
          severity: info
        annotations:
          summary: "Network interface {{ $labels.device }} flapping"

      # WARNING : Latence élevée mais pas critique
      - alert: HighLatency
        expr: probe_duration_seconds > 0.15
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Latency > 150ms on {{ $labels.instance }}"

      # CRITICAL : Service inaccessible
      - alert: ServiceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.instance }} is DOWN"

      # EMERGENCY : Plusieurs services down
      - alert: MultipleServicesDown
        expr: count(up == 0) > 3
        for: 1m
        labels:
          severity: emergency
        annotations:
          summary: "{{ $value }} services are DOWN"
```

### Canaux de notification

**Escalade par sévérité :**
```
INFO      → Log uniquement (pas de notification)
WARNING   → Email équipe technique
CRITICAL  → Email + SMS astreinte
EMERGENCY → Email + SMS + Appel téléphonique + PagerDuty
```

**Configuration Alertmanager :**
```yaml
# alertmanager.yml
route:
  receiver: 'default'
  group_by: ['alertname', 'instance']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  routes:
    # Alertes INFO → email seulement
    - match:
        severity: info
      receiver: 'email-team'

    # Alertes WARNING → email
    - match:
        severity: warning
      receiver: 'email-team'

    # Alertes CRITICAL → email + slack
    - match:
        severity: critical
      receiver: 'critical-alerts'

    # Alertes EMERGENCY → tout
    - match:
        severity: emergency
      receiver: 'emergency-alerts'

receivers:
  - name: 'default'
    email_configs:
      - to: 'ops@example.com'

  - name: 'email-team'
    email_configs:
      - to: 'ops-team@example.com'

  - name: 'critical-alerts'
    email_configs:
      - to: 'ops-team@example.com'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX'
        channel: '#alerts'

  - name: 'emergency-alerts'
    email_configs:
      - to: 'ops-team@example.com'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX'
        channel: '#critical-alerts'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
```

---

## Analyse post-mortem

### Template de rapport d'incident

**Structure recommandée :**

```markdown
# Incident Report #2025-12-07-001

## Résumé
Service API indisponible pendant 45 minutes le 7 décembre 2025 entre 14:30 et 15:15 UTC.
Impact : 5,000 utilisateurs, perte estimée de 10,000€ de revenus.

## Timeline

**14:28** - Déploiement nouvelle version application backend v2.5.0
**14:30** - Premières alertes "HighLatency" depuis monitoring
**14:31** - Alertes "ServiceDown" pour api.example.com
**14:32** - Équipe ops notifiée (escalade automatique)
**14:35** - Investigation démarrée (logs nginx montrent 502 errors)
**14:40** - Capture Wireshark démarrée sur nginx et backend
**14:42** - Wireshark montre RST depuis backend (port fermé)
**14:43** - Connexion SSH backend : service stopped (crash au démarrage)
**14:45** - Logs backend révèlent erreur de config (port 8080 → 8000)
**14:50** - Correction config, redémarrage service
**14:52** - Validation : service OK, latence normale
**14:55** - Monitoring confirme résolution
**15:00** - Communication aux utilisateurs (service rétabli)
**15:15** - Incident déclaré résolu

## Cause racine

Changement de configuration non testé dans la version v2.5.0 :
- Port d'écoute modifié de 8080 à 8000
- Nginx toujours configuré pour 8080
- Tests pre-prod incomplets (n'ont pas détecté la différence)

## Impact

- **Disponibilité** : 45 minutes de downtime (SLA 99.9% = 43 min/mois)
- **Utilisateurs** : 5,000 utilisateurs affectés
- **Revenu** : ~10,000€ de transactions perdues
- **Réputation** : 234 tweets négatifs, -2.5 pts satisfaction client

## Analyse technique

### Données Wireshark
```
Paquets capturés : 1,234
TCP SYN vers backend : 456
TCP RST depuis backend : 456 (100% échec)
Latence moyenne (avant incident) : 45ms
Latence pendant incident : N/A (timeout)
```

### Logs
- Nginx : 3,456 erreurs "502 Bad Gateway"
- Backend : "Error: Cannot bind to port 8000: Address already in use"
- Firewall : Aucun drop (port ouvert)

### Métriques monitoring
- Uptime : Passé de 99.98% à 99.89% sur le mois
- Erreur rate : Pic à 100% pendant 45 minutes

## Actions immédiates prises

1. ✅ Service redémarré avec bonne config
2. ✅ Validation tests automatisés
3. ✅ Communication utilisateurs
4. ✅ Rollback automatique désactivé (risque d'amplification)

## Actions préventives (court terme)

1. **Tests pre-prod renforcés**
   - Ajouter test de connectivité nginx → backend
   - Valider tous les ports configurés
   - Tests automatisés sur chaque déploiement

2. **Amélioration monitoring**
   - Alerte si nginx → backend connection fails
   - Dashboard synthèse pré/post déploiement

3. **Rollback automatique**
   - Si > 10% erreurs pendant 2 min post-deploy → rollback

4. **Documentation**
   - Procédure de déploiement mise à jour
   - Checklist validation ports/config

## Actions préventives (long terme)

1. **Infrastructure as Code**
   - Ansible/Terraform pour config nginx + backend
   - Validation automatique cohérence ports

2. **Canary deployments**
   - Déployer sur 10% trafic d'abord
   - Rollout complet si metrics OK

3. **Chaos engineering**
   - Tester régulièrement scenarios de panne
   - Valider détection + récupération automatique

## Leçons apprises

✅ **Ce qui a bien fonctionné :**
- Détection rapide (2 minutes)
- Escalade automatique efficace
- Wireshark a confirmé rapidement la cause
- Communication transparente

❌ **Ce qui doit être amélioré :**
- Tests pre-prod insuffisants
- Pas de validation automatique de config
- Temps de résolution trop long (45 min pour un problème simple)
- Absence de rollback automatique

## Suivi

- [ ] Implémenter tests validant ports (deadline: 14/12)
- [ ] Configurer rollback automatique (deadline: 21/12)
- [ ] Formation équipe sur debugging avec Wireshark (deadline: 31/12)
- [ ] Review SLA et compensations clients (deadline: 10/12)

## Signatures

Rédigé par : John Doe (Lead DevOps)
Validé par : Jane Smith (CTO)
Date : 08/12/2025
```

### Métriques d'incident

**KPIs à tracker :**
```
MTTD (Mean Time To Detect)
→ Temps entre début incident et détection
→ Objectif : < 5 minutes

MTTI (Mean Time To Investigate)
→ Temps entre détection et compréhension de la cause
→ Objectif : < 15 minutes

MTTR (Mean Time To Repair)
→ Temps entre détection et résolution
→ Objectif : < 30 minutes

MTBF (Mean Time Between Failures)
→ Temps moyen entre incidents
→ Objectif : > 720 heures (30 jours)
```

---

## Bonnes pratiques

### 1. Centraliser tous les logs

```
✅ À faire :
- Tous les serveurs → syslog central
- Tous les équipements → SNMP trap receiver
- Toutes les apps → logging centralisé (ELK, Splunk)

❌ À éviter :
- Logs dispersés sur chaque serveur
- Pas de rétention longue durée
- Logs non structurés (difficiles à parser)
```

### 2. Structurer les logs

```
✅ Format structuré (JSON) :
{
  "timestamp": "2025-12-07T14:30:15.123Z",
  "level": "error",
  "service": "api-backend",
  "host": "server1.example.com",
  "message": "Database connection timeout",
  "metadata": {
    "db_host": "10.0.8.50",
    "timeout_ms": 5000,
    "query": "SELECT * FROM users WHERE id=?"
  }
}

❌ Format non structuré :
[ERROR] 2025-12-07 14:30:15 - Database timeout on 10.0.8.50 after 5000ms
```

### 3. Définir des niveaux de log cohérents

```
DEBUG   : Détails internes (désactivé en prod)
INFO    : Événements normaux (connexion, déconnexion)
WARNING : Événements anormaux mais gérés
ERROR   : Erreurs impactant des fonctionnalités
CRITICAL: Erreurs critiques nécessitant action immédiate
```

### 4. Monitorer les métriques, pas seulement les logs

```
Logs = Événements discrets (erreur, connexion, etc.)
Métriques = Valeurs continues (CPU, latence, débit)

✅ Combiner les deux :
- Logs pour le contexte (pourquoi ?)
- Métriques pour les tendances (combien ? quand ?)
```

### 5. Corréler les sources

```
Incident détecté :
1. Alerte monitoring (métrique)
2. → Consulter logs (contexte)
3. → Capture Wireshark si nécessaire (détail paquet)
4. → Logs équipements réseau (infrastructure)

Chaque source apporte une pièce du puzzle
```

### 6. Automatiser ce qui peut l'être

```
✅ Automatiser :
- Collecte de logs
- Génération d'alertes
- Création de tickets d'incident
- Graphiques/dashboards
- Tests de connectivité

❌ Ne pas automatiser :
- Analyse de cause racine (nécessite humain)
- Décisions de rollback critiques
- Communication de crise
```

### 7. Tester régulièrement

```
Test monitoring :
- Simuler pannes (arrêter service, débrancher câble)
- Vérifier que alertes se déclenchent
- Mesurer MTTD (temps de détection)

Test procédures :
- Exercices de réponse à incident
- Game days (simulations)
- Chaos engineering
```

---

## Points clés à retenir

### 🎯 Logs vs Monitoring

**Logs :**
```
Nature : Événements discrets, texte
Usage : Debugging, audit, forensics
Stockage : Court-moyen terme (30-90 jours)
Outils : syslog, ELK, Splunk
```

**Monitoring :**
```
Nature : Métriques continues, numériques
Usage : Alerting, tendances, capacité
Stockage : Moyen-long terme (6-12 mois)
Outils : Prometheus, Nagios, Zabbix
```

**Complémentarité :**
```
Monitoring détecte → Logs expliquent → Wireshark confirme
```

### 🎯 Métriques essentielles

```
Disponibilité : up, probe_success
Latence : RTT, response time
Débit : bandwidth usage
Fiabilité : packet loss, errors
Capacité : connections, ports
```

### 🎯 Alerting

```
Éviter faux positifs : for, avg_over_time
Niveaux de sévérité : info, warning, critical, emergency
Escalade intelligente : selon impact et durée
```

### 🎯 Post-mortem

```
Objectif : Apprendre, pas blâmer
Contenu : Timeline, cause, impact, actions
Suivi : Implémenter préventions
```

---

## Conclusion

Les logs et le monitoring complètent parfaitement Wireshark dans l'arsenal du diagnostic réseau :

- **Wireshark** : Analyse fine au niveau paquet, ponctuelle
- **Logs** : Contexte événementiel, historique
- **Monitoring** : Vue continue, tendances, alerting proactif

L'association des trois permet une approche complète :
1. **Monitoring** détecte un problème (alerte latence élevée)
2. **Logs** donnent le contexte (erreurs backend, timeouts)
3. **Wireshark** confirme et précise (retransmissions TCP, RST)

La clé du succès : automatiser la collecte, centraliser les données, et mettre en place des alertes intelligentes pour détecter les problèmes avant qu'ils n'impactent les utilisateurs.

---

**Fin du Module 7 : Analyse et dépannage réseau**

Vous maîtrisez maintenant :
- ✅ La méthodologie de diagnostic structurée
- ✅ Les outils en ligne de commande (ping, traceroute, netstat, dig)
- ✅ Wireshark : principes, lecture, filtres
- ✅ L'identification des problèmes courants
- ✅ L'analyse de performances réseau
- ✅ Les logs et le monitoring pour une vision complète

Ces compétences vous permettent de diagnostiquer et résoudre efficacement la grande majorité des problèmes réseau rencontrés en production.

**Module suivant :** 8. Programmation réseau (perspective développeur)

Nous passerons du diagnostic à la création : comment programmer avec TCP/IP, utiliser les sockets, gérer la fiabilité, et implémenter des communications réseau robustes dans vos applications.

⏭️ [8. Programmation réseau (perspective développeur)](/08-programmation-reseau/README.md)
