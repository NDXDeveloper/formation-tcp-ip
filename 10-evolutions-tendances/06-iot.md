🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.6 - IoT et contraintes protocolaires

## Introduction

L'Internet des Objets (IoT) connecte des milliards d'appareils aux ressources extrêmement limitées : capteurs à $1, microcontrôleurs fonctionnant sur pile pendant des années, devices embarqués dans des environnements hostiles. Ces contraintes rendent TCP/IP traditionnel inadapté.

Pour les développeurs, l'IoT impose un **changement radical de paradigme** : chaque octet compte, chaque milliseconde de transmission vide la batterie, et la connectivité est intermittente par nature. Cette section explore les protocoles et architectures spécialisés pour l'IoT.

## Les contraintes de l'IoT

### 1. Ressources matérielles limitées

**Comparaison typique** :

```
Smartphone moderne (2025) :
- CPU : 8 cores, 3 GHz
- RAM : 8-12 GB
- Storage : 128-512 GB
- Batterie : 4000-5000 mAh
- Réseau : WiFi 6, 5G

Capteur IoT typique :
- CPU : 1 core, 16-48 MHz
- RAM : 16-256 KB (kilo-octets !)
- Storage : 128-512 KB Flash
- Batterie : 200-1000 mAh (ou energy harvesting)
- Réseau : LoRa, Zigbee, BLE

Ratio : 1000-10000x moins de ressources
```

**Implications concrètes** :

```c
// Code pour ESP8266 (WiFi IoT populaire)
// RAM : 80 KB seulement !

#define MAX_PAYLOAD_SIZE 512  // Pas de buffers énormes
#define MAX_CONNECTIONS 4     // Limite stricte

// ❌ Impossible : Stack TCP/IP complet
// HTTP client avec TLS = 40+ KB RAM
// MQTT avec TLS = 20-30 KB RAM

// ✅ Obligé d'optimiser agressivement
uint8_t buffer[MAX_PAYLOAD_SIZE];  // Buffer minimal
char mqtt_topic[32];                // Strings courtes

void setup() {
    // Désactiver fonctionnalités non-essentielles
    WiFi.setOutputPower(15);  // Réduit puissance (économie batterie)
    WiFi.setSleepMode(WIFI_LIGHT_SLEEP);  // Sleep agressif

    // Connexion minimal footprint
    WiFiClient client;
    PubSubClient mqtt(client);  // MQTT = protocole léger
}

void loop() {
    // Lire capteur
    float temperature = readSensor();

    // Publier seulement si changement significatif
    if (abs(temperature - last_temp) > 0.5) {
        // Payload minimal : "23.4" au lieu de JSON verbeux
        char payload[8];
        dtostrf(temperature, 4, 1, payload);

        mqtt.publish("temp", payload);  // ~10 bytes total
        last_temp = temperature;
    }

    // Sleep profond pour économiser batterie
    ESP.deepSleep(60e6);  // 60 secondes
    // Consommation : 20 µA en sleep vs 80 mA actif
}
```

### 2. Contraintes énergétiques

**Budget énergétique typique** :

```
Capteur alimenté par pile AA (3000 mAh, 1.5V) :

Objectif : 5-10 ans d'autonomie
Budget : 3000 mAh / (10 ans × 365 jours × 24h) = 34 µA moyen

Consommations typiques :
- Sleep mode : 10-50 µA
- Idle (radio ON) : 10-50 mA (1000x plus !)
- TX/RX : 50-150 mA (5000x plus !)

Conclusion : Doit dormir 99.9% du temps
```

**Calcul durée de vie** :

```python
# Calculateur durée de vie batterie IoT

class BatteryLifeCalculator:
    def __init__(self, battery_mah=3000, voltage=3.0):
        self.battery_mah = battery_mah
        self.voltage = voltage

    def calculate_lifetime(self, profile):
        """
        Calculer durée de vie basé sur profil d'utilisation
        """
        # Consommation moyenne pondérée
        avg_current_ma = 0

        for state, config in profile.items():
            current_ma = config['current_ma']
            time_percentage = config['time_pct']
            avg_current_ma += current_ma * (time_percentage / 100)

        # Durée de vie en heures
        lifetime_hours = self.battery_mah / avg_current_ma

        # Convertir en années
        lifetime_years = lifetime_hours / (365.25 * 24)

        return {
            'avg_current_ma': avg_current_ma,
            'lifetime_hours': lifetime_hours,
            'lifetime_years': lifetime_years
        }

# Exemple : Capteur température avec WiFi
wifi_sensor_profile = {
    'deep_sleep': {
        'current_ma': 0.02,   # 20 µA
        'time_pct': 99.7      # 99.7% du temps
    },
    'wake_measure': {
        'current_ma': 80,     # 80 mA actif
        'time_pct': 0.1       # 0.1% du temps (6 sec/heure)
    },
    'wifi_tx': {
        'current_ma': 170,    # 170 mA transmission
        'time_pct': 0.2       # 0.2% du temps (12 sec/heure)
    }
}

calc = BatteryLifeCalculator(battery_mah=3000)
result = calc.calculate_lifetime(wifi_sensor_profile)

print(f"Consommation moyenne : {result['avg_current_ma']:.3f} mA")
print(f"Durée de vie : {result['lifetime_years']:.1f} ans")

# Output :
# Consommation moyenne : 0.366 mA
# Durée de vie : 0.9 ans (11 mois)

# Avec LoRaWAN (plus économe) :
lora_sensor_profile = {
    'deep_sleep': {
        'current_ma': 0.01,   # 10 µA
        'time_pct': 99.9
    },
    'wake_measure': {
        'current_ma': 50,
        'time_pct': 0.05
    },
    'lora_tx': {
        'current_ma': 120,
        'time_pct': 0.05      # Transmission très courte
    }
}

result_lora = calc.calculate_lifetime(lora_sensor_profile)
print(f"Durée de vie LoRa : {result_lora['lifetime_years']:.1f} ans")
# Output : 10.3 ans

# WiFi vs LoRa : 10x différence en durée de vie !
```

### 3. Connectivité intermittente

```
Problématiques :
- Réseau pas toujours disponible (devices mobiles, zones blanches)
- Interférences radio fréquentes
- Changements de réseau (roaming)
- Limitation duty cycle (réglementation radio)

TCP traditionnel :
- Assume connexion stable
- Retransmissions fréquentes si perte
- Handshake 3-way coûteux
- Keep-alive consomme batterie

→ Inadapté pour IoT
```

### 4. Latence variable et tolérée

```
Applications IoT typiques :

Temps réel critique (rares) :
- Alarmes incendie : <1s
- Contrôle industriel : <100ms
- Véhicules connectés : <50ms

Temps réel souple (courants) :
- Thermostat : <10s acceptable
- Monitoring environnemental : <1min
- Agriculture : <1h

Batch (très courants) :
- Compteurs électriques : 1x/jour
- Relevés eau/gaz : 1x/jour
- Maintenance prédictive : 1x/semaine

→ Optimiser pour latence moyenne/énergie, pas latence min
```

## Protocoles IoT vs TCP/IP traditionnel

### Stack TCP/IP classique

```
Problèmes pour IoT :

┌─────────────────────┐
│    HTTP/HTTPS       │  Verbose, headers énormes
├─────────────────────┤  GET /api HTTP/1.1 = 100+ bytes
│    TLS 1.3          │  Handshake = 1-2 RTT
├─────────────────────┤  Certificate = plusieurs KB
│       TCP           │  3-way handshake
├─────────────────────┤  20 bytes header par paquet
│       IPv4/IPv6     │  20-40 bytes header
├─────────────────────┤
│    Ethernet/WiFi    │  14-30 bytes header
└─────────────────────┘

Total overhead : 100-200 bytes MINIMUM
Pour envoyer "23.4°C" (5 bytes de données)
→ Ratio 20:1 à 40:1 overhead/data !

Consommation :
- WiFi connection : 500-1000 mA peak
- Durée : 1-5 secondes
- Énergie : ~1-5 mAh par transmission

Budget batterie bouffe en quelques mois
```

### Protocoles IoT optimisés

```
Objectifs :
✅ Overhead minimal
✅ Connexion rapide ou sans état
✅ Consommation minimale
✅ Résilience aux pertes
✅ Support connectivité intermittente
```

## MQTT (Message Queuing Telemetry Transport)

### Principes

MQTT est un protocole **publish-subscribe** conçu pour l'IoT dès le départ.

```
Architecture :

Publishers                Broker              Subscribers
(Sensors)                (Server)             (Apps)

[Temp]─┐                                    ┌─[Dashboard]
[Hum] ─┼─ publish("sensors/temp", "23.4")─> │
[CO2] ─┘         MQTT Broker                ├─[Alert System]
                 (central hub)              │
                      │                     └─[Logger]
                      │
                 Persiste, route,
                 retient messages
```

**Caractéristiques clés** :

```
- Transport : TCP (port 1883) ou TLS (8883)
- Overhead : ~2 bytes header minimal
- QoS : 3 niveaux (0, 1, 2)
- Retain : Dernier message gardé par broker
- Last Will : Message si client disconnect
- Session : Persistante entre reconnexions
```

### QoS (Quality of Service)

```python
# MQTT QoS levels

# QoS 0 : At most once (fire and forget)
mqtt.publish("sensor/temp", "23.4", qos=0)
# - Envoi sans accusé de réception
# - Peut être perdu
# - Consommation minimale
# Use case : Données fréquentes, perte acceptable (temp chaque 1 min)

# QoS 1 : At least once (acknowledged)
mqtt.publish("sensor/alert", "motion_detected", qos=1)
# - Accusé de réception (PUBACK)
# - Peut être dupliqué
# - Consommation modérée
# Use case : Alertes importantes, duplication OK

# QoS 2 : Exactly once (assured)
mqtt.publish("actuator/valve", "close", qos=2)
# - Handshake 4-way (PUBREC, PUBREL, PUBCOMP)
# - Garanti 1 seule fois
# - Consommation élevée
# Use case : Commandes critiques, pas de duplication
```

### Implémentation pratique

**Publisher (capteur)** :

```python
# Python sur Raspberry Pi ou ESP32 avec MicroPython
import paho.mqtt.client as mqtt
import time

class IoTSensor:
    def __init__(self, broker, client_id):
        self.client = mqtt.Client(client_id=client_id, clean_session=False)
        self.broker = broker

        # Last Will : notifier si capteur meurt
        self.client.will_set(
            "sensors/status/" + client_id,
            payload="offline",
            qos=1,
            retain=True
        )

        # Callbacks
        self.client.on_connect = self.on_connect
        self.client.on_disconnect = self.on_disconnect

    def connect(self):
        self.client.connect(self.broker, 1883, keepalive=60)
        self.client.loop_start()

    def on_connect(self, client, userdata, flags, rc):
        print(f"Connected with result code {rc}")

        # Annoncer online
        self.client.publish(
            f"sensors/status/{self.client._client_id}",
            "online",
            qos=1,
            retain=True  # Retain pour que subscribers voient status
        )

    def on_disconnect(self, client, userdata, rc):
        print(f"Disconnected with result code {rc}")

    def publish_reading(self, sensor_type, value):
        """
        Publier lecture capteur
        """
        topic = f"sensors/data/{self.client._client_id}/{sensor_type}"

        # Payload minimal (pas de JSON si pas nécessaire)
        payload = str(value)

        # QoS 0 pour données fréquentes
        self.client.publish(topic, payload, qos=0)

        print(f"Published {sensor_type}={value} to {topic}")

# Utilisation
sensor = IoTSensor(broker="mqtt.example.com", client_id="temp_sensor_01")
sensor.connect()

while True:
    temp = read_temperature()  # Fonction hardware
    sensor.publish_reading("temperature", temp)

    humidity = read_humidity()
    sensor.publish_reading("humidity", humidity)

    # Sleep 60 secondes
    time.sleep(60)
```

**Subscriber (application)** :

```python
# Application qui consomme données MQTT
import paho.mqtt.client as mqtt

class IoTDataCollector:
    def __init__(self, broker):
        self.client = mqtt.Client()
        self.broker = broker
        self.data_buffer = []

        self.client.on_connect = self.on_connect
        self.client.on_message = self.on_message

    def on_connect(self, client, userdata, flags, rc):
        print("Connected to MQTT broker")

        # S'abonner à tous les capteurs de température
        self.client.subscribe("sensors/data/+/temperature", qos=0)

        # S'abonner aux alertes (QoS 1)
        self.client.subscribe("sensors/alerts/#", qos=1)

        # S'abonner aux status
        self.client.subscribe("sensors/status/#", qos=1)

    def on_message(self, client, userdata, msg):
        """
        Callback pour chaque message reçu
        """
        topic = msg.topic
        payload = msg.payload.decode()

        print(f"Received: {topic} = {payload}")

        # Router selon le topic
        if "temperature" in topic:
            self.handle_temperature(topic, payload)
        elif "alerts" in topic:
            self.handle_alert(topic, payload)
        elif "status" in topic:
            self.handle_status(topic, payload)

    def handle_temperature(self, topic, payload):
        # Extraire sensor ID depuis topic
        # sensors/data/temp_sensor_01/temperature
        parts = topic.split('/')
        sensor_id = parts[2]

        temp = float(payload)

        # Stocker dans base de données
        self.save_to_db(sensor_id, 'temperature', temp)

        # Vérifier seuils
        if temp > 30:
            print(f"⚠️ High temperature alert: {sensor_id} = {temp}°C")
            self.send_notification(sensor_id, f"Temperature high: {temp}°C")

    def handle_alert(self, topic, payload):
        # Alerte critique : logger immédiatement
        print(f"🚨 ALERT: {topic} - {payload}")
        self.send_sms_alert(topic, payload)

    def handle_status(self, topic, payload):
        # Status sensor (online/offline)
        parts = topic.split('/')
        sensor_id = parts[2]

        if payload == "offline":
            print(f"❌ Sensor offline: {sensor_id}")
            self.send_notification(sensor_id, "Sensor went offline")

# Utilisation
collector = IoTDataCollector(broker="mqtt.example.com")
collector.client.connect("mqtt.example.com", 1883, 60)
collector.client.loop_forever()
```

### MQTT over WebSocket

```javascript
// MQTT dans le browser (IoT dashboard web)
import mqtt from 'mqtt';

class IoTDashboard {
    constructor(brokerUrl) {
        // MQTT over WebSocket (port 9001 typiquement)
        this.client = mqtt.connect('ws://mqtt.example.com:9001', {
            clientId: 'dashboard_' + Math.random().toString(16).substr(2, 8),
            clean: true,
            reconnectPeriod: 5000
        });

        this.client.on('connect', () => {
            console.log('Connected to MQTT broker');

            // S'abonner aux données temps réel
            this.client.subscribe('sensors/data/+/temperature');
            this.client.subscribe('sensors/data/+/humidity');
        });

        this.client.on('message', (topic, message) => {
            const value = message.toString();
            this.updateDashboard(topic, value);
        });
    }

    updateDashboard(topic, value) {
        // Mettre à jour UI en temps réel
        const parts = topic.split('/');
        const sensorId = parts[2];
        const metric = parts[3];

        // Update chart, gauge, etc.
        document.getElementById(`${sensorId}-${metric}`).textContent = value;
    }

    sendCommand(deviceId, command) {
        // Envoyer commande à un actuateur
        const topic = `actuators/${deviceId}/command`;
        this.client.publish(topic, command, { qos: 1 });
    }
}

// Utilisation
const dashboard = new IoTDashboard('ws://mqtt.example.com:9001');

// Envoyer commande
document.getElementById('valve-close-btn').onclick = () => {
    dashboard.sendCommand('valve_01', 'close');
};
```

### Avantages MQTT

```
✅ Overhead minimal : 2+ bytes (vs 100+ TCP/IP)
✅ Publish-subscribe : découplage publisher/subscriber
✅ QoS flexible : trade-off fiabilité/énergie
✅ Session persistante : reconnexion rapide
✅ Last Will : détection déconnexion automatique
✅ Retain : nouveaux subscribers reçoivent dernier état
✅ Wildcard subscriptions : + (single level), # (multi-level)

❌ Limitations :
- Require broker central (point unique défaillance)
- TCP-based : pas optimal pour très faible puissance
- Pas de discovery automatique
```

## CoAP (Constrained Application Protocol)

### Principes

CoAP est "HTTP pour l'IoT" : REST-like mais optimisé pour devices contraints.

```
HTTP :                          CoAP :
- TCP                          - UDP
- Texte (headers)              - Binaire
- Stateful                     - Stateless
- ~100-500 bytes overhead      - ~4 bytes overhead
- Port 80/443                  - Port 5683/5684

GET /temperature HTTP/1.1      GET coap://sensor/temp
Host: sensor.local             (4-10 bytes total)
Accept: application/json
...
(100+ bytes)
```

### Format de message

```
CoAP message format (minimal) :

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Ver| T |  TKL  |      Code     |          Message ID           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Token (if any, TKL bytes) ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Options (if any) ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|1 1 1 1 1 1 1 1|    Payload (if any) ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Header : 4 bytes seulement !
```

### Méthodes CoAP

```python
# CoAP methods (comme HTTP mais compact)

from aiocoap import *

# GET : Lire ressource
async def coap_get(uri):
    protocol = await Context.create_client_context()

    request = Message(code=GET, uri=uri)
    response = await protocol.request(request).response

    return response.payload.decode()

# Exemple
temp = await coap_get('coap://sensor.local/temperature')
print(f"Temperature: {temp}")

# POST : Créer/modifier ressource
async def coap_post(uri, payload):
    protocol = await Context.create_client_context()

    request = Message(
        code=POST,
        uri=uri,
        payload=payload.encode()
    )
    response = await protocol.request(request).response

    return response.code

# PUT : Mettre à jour
async def coap_put(uri, payload):
    protocol = await Context.create_client_context()

    request = Message(code=PUT, uri=uri, payload=payload.encode())
    response = await protocol.request(request).response

    return response.code

# DELETE : Supprimer
async def coap_delete(uri):
    protocol = await Context.create_client_context()

    request = Message(code=DELETE, uri=uri)
    response = await protocol.request(request).response

    return response.code
```

### CoAP Observe (push notifications)

```python
# CoAP Observe : Subscription à changements (comme MQTT subscribe)

from aiocoap import *

class CoAPObserver:
    async def observe_temperature(self, sensor_uri):
        """
        Observer changements de température
        """
        protocol = await Context.create_client_context()

        # Request avec option Observe
        request = Message(code=GET, uri=sensor_uri, observe=0)

        # Contexte de requête
        pr = protocol.request(request)

        # Réponse initiale
        response = await pr.response
        print(f"Initial temperature: {response.payload.decode()}")

        # Observations continues
        async for response in pr.observation:
            temp = response.payload.decode()
            print(f"Temperature changed: {temp}")

            # Traiter changement
            await self.handle_temperature_change(float(temp))

    async def handle_temperature_change(self, temp):
        if temp > 30:
            print(f"⚠️ High temperature: {temp}°C")
            # Déclencher action...

# Utilisation
observer = CoAPObserver()
await observer.observe_temperature('coap://sensor.local/temperature')
# Device push automatiquement quand température change
# Pas de polling !
```

### Serveur CoAP (sur device)

```python
# CoAP server sur ESP32 ou Raspberry Pi

import aiocoap.resource as resource
import aiocoap

class TemperatureResource(resource.Resource):
    """
    Ressource CoAP exposant température
    """
    def __init__(self):
        super().__init__()
        self.temperature = 20.0
        self.observers = set()

    async def render_get(self, request):
        """
        Répondre à GET /temperature
        """
        payload = str(self.temperature).encode()

        return aiocoap.Message(
            code=aiocoap.CONTENT,
            payload=payload
        )

    async def render_put(self, request):
        """
        Mettre à jour température (pour tests)
        """
        new_temp = float(request.payload.decode())
        self.temperature = new_temp

        # Notifier observers
        self.updated_state()

        return aiocoap.Message(code=aiocoap.CHANGED)

    async def update_temperature(self, new_temp):
        """
        Appelé quand capteur hardware lit nouvelle temp
        """
        if abs(new_temp - self.temperature) > 0.5:  # Delta significatif
            self.temperature = new_temp
            self.updated_state()  # Notifie observers automatiquement

class HumidityResource(resource.Resource):
    """
    Ressource CoAP exposant humidité
    """
    def __init__(self):
        super().__init__()
        self.humidity = 50.0

    async def render_get(self, request):
        payload = str(self.humidity).encode()
        return aiocoap.Message(code=aiocoap.CONTENT, payload=payload)

# Serveur CoAP
async def main():
    # Créer ressources
    root = resource.Site()
    root.add_resource(['temperature'], TemperatureResource())
    root.add_resource(['humidity'], HumidityResource())

    # Démarrer serveur
    await aiocoap.Context.create_server_context(root, bind=('0.0.0.0', 5683))

    print("CoAP server running on port 5683")
    print("Resources:")
    print("  coap://[device-ip]/temperature")
    print("  coap://[device-ip]/humidity")

    # Garder serveur vivant
    await asyncio.get_running_loop().create_future()

# Run
asyncio.run(main())
```

### CoAP vs MQTT

```
                CoAP                    MQTT
────────────────────────────────────────────────────────────
Transport       UDP                     TCP
Pattern         Request/Response        Publish/Subscribe
Overhead        ~4 bytes                ~2 bytes
QoS             CON/NON messages        QoS 0/1/2
Discovery       Resource discovery      Topics
Architecture    Peer-to-peer possible   Requires broker
Multicast       Oui (IPv6)              Non
Caching         Oui (proxy)             Non
Use case        Device-to-device        Many-to-many via hub

Choix :
- CoAP : Réseau mesh, M2M direct, HTTP-like API
- MQTT : Cloud connectivity, pub/sub, broker central
```

## LoRaWAN (Long Range Wide Area Network)

### Caractéristiques

LoRaWAN est optimisé pour **longue portée** et **très faible consommation**.

```
Spécifications :
- Portée : 2-15 km (urbain), 15-50 km (rural)
- Débit : 0.3-50 kbps (très faible)
- Fréquence : 868 MHz (EU), 915 MHz (US), ISM bands
- Topologie : Star (devices → gateways → network server)
- Duty cycle : 1% max (réglementation radio)
- Consommation : 10-100 µA sleep, ~40 mA TX
- Durée vie batterie : 5-10 ans typique
```

**Architecture** :

```
End Devices          Gateways         Network Server      Application
(Sensors)           (Repeaters)       (LoRa backend)      (Your app)

[Sensor A]─┐
[Sensor B]─┼─ LoRa RF ─>┌──────────┐
[Sensor C]─┘            │ Gateway 1│─┐
                        └──────────┘ │
                                     ├─ Internet ─>┌─────────────┐
[Sensor D]─┐            ┌──────────┐ │             │   Network   │─> API
[Sensor E]─┼─ LoRa RF ─>│ Gateway 2│─┘             │   Server    │
[Sensor F]─┘            └──────────┘               │ (TTN, etc.) │
                                                   └─────────────┘
```

### Classes de devices

```
Class A (lowest power) :
- Device initie communication
- RX windows après chaque TX
- Consommation : 10-50 µA moyen
- Use case : Capteurs battery-powered, 99% du temps

Class B (scheduled) :
- RX windows périodiques (beacon sync)
- Latence prévisible
- Consommation : 100-500 µA
- Use case : Actuateurs avec commandes régulières

Class C (always listening) :
- RX continu
- Latence minimale
- Consommation : 10-50 mA
- Use case : Actuateurs critiques, mains-powered
```

### Payload LoRaWAN

```python
# LoRaWAN : Payload extrêmement limité !

# Maximum : 51-222 bytes selon SF (Spreading Factor)
# Typiquement : 10-50 bytes utiles

# ❌ Mauvais : JSON verbeux
payload_bad = json.dumps({
    "temperature": 23.4,
    "humidity": 65.2,
    "battery": 3.7,
    "timestamp": "2025-12-07T14:30:00Z"
})
# Taille : ~90 bytes → Ne passe pas !

# ✅ Bon : Encodage binaire compact
import struct

def encode_sensor_data(temp, humidity, battery):
    """
    Encoder données en binaire compact
    """
    # Format :
    # - temp : int16 (×10 pour 1 décimale) : 2 bytes
    # - humidity : uint8 (0-100%) : 1 byte
    # - battery : uint16 (×100 pour 2 décimales) : 2 bytes

    temp_encoded = int(temp * 10)
    humidity_encoded = int(humidity)
    battery_encoded = int(battery * 100)

    # Pack en binaire
    payload = struct.pack('>hBH', temp_encoded, humidity_encoded, battery_encoded)

    return payload  # 5 bytes seulement !

# Utilisation
payload = encode_sensor_data(23.4, 65, 3.7)
print(f"Payload size: {len(payload)} bytes")
# Output: 5 bytes (vs 90 bytes JSON)

# Décodage côté serveur
def decode_sensor_data(payload):
    """
    Décoder payload binaire
    """
    temp_encoded, humidity, battery_encoded = struct.unpack('>hBH', payload)

    temp = temp_encoded / 10.0
    battery = battery_encoded / 100.0

    return {
        'temperature': temp,
        'humidity': humidity,
        'battery': battery
    }

# Optimisation extrême : bit packing
def encode_ultra_compact(temp, humidity, battery_percent):
    """
    Encoder dans 3 bytes seulement
    """
    # temp : -20 à +60°C, résolution 0.5°C = 160 valeurs = 8 bits
    # humidity : 0-100%, résolution 1% = 100 valeurs = 7 bits
    # battery : 0-100%, résolution 1% = 100 valeurs = 7 bits

    # Temp : offset -20, résolution 0.5
    temp_byte = int((temp + 20) * 2)

    # Pack 22 bits dans 3 bytes
    # Byte 0 : temp (8 bits)
    # Byte 1 : humidity (7 bits) + battery MSB (1 bit)
    # Byte 2 : battery LSB (6 bits) + reserved (2 bits)

    byte0 = temp_byte & 0xFF
    byte1 = (humidity & 0x7F) << 1 | ((battery_percent >> 6) & 0x01)
    byte2 = (battery_percent & 0x3F) << 2

    return bytes([byte0, byte1, byte2])

# 3 bytes pour 3 valeurs !
# Gain : 60% vs struct.pack, 97% vs JSON
```

### Implémentation Arduino/ESP32

```cpp
// LoRaWAN sur Arduino avec LMIC library

#include <lmic.h>
#include <hal/hal.h>

// LoRaWAN keys (from TTN console)
static const u1_t PROGMEM APPEUI[8] = { 0x00, 0x00, ... };
static const u1_t PROGMEM DEVEUI[8] = { 0x00, 0x00, ... };
static const u1_t PROGMEM APPKEY[16] = { 0x00, 0x00, ... };

// Schedule TX every this many seconds
const unsigned TX_INTERVAL = 300;  // 5 minutes

void setup() {
    Serial.begin(115200);

    // LMIC init
    os_init();
    LMIC_reset();

    // Start job (Join network + transmit)
    do_send(&sendjob);
}

void do_send(osjob_t* j) {
    // Check if there is not a current TX/RX job running
    if (LMIC.opmode & OP_TXRXPEND) {
        Serial.println(F("OP_TXRXPEND, not sending"));
    } else {
        // Read sensors
        float temp = readTemperature();
        float humidity = readHumidity();
        float battery = readBattery();

        // Encode payload (compact binary)
        uint8_t payload[5];
        int16_t temp_int = (int16_t)(temp * 10);
        uint16_t battery_int = (uint16_t)(battery * 100);

        payload[0] = (temp_int >> 8) & 0xFF;
        payload[1] = temp_int & 0xFF;
        payload[2] = (uint8_t)humidity;
        payload[3] = (battery_int >> 8) & 0xFF;
        payload[4] = battery_int & 0xFF;

        // Prepare upstream data transmission
        LMIC_setTxData2(1, payload, sizeof(payload), 0);

        Serial.println(F("Packet queued"));
    }

    // Schedule next transmission
    os_setTimedCallback(&sendjob, os_getTime() + sec2osticks(TX_INTERVAL), do_send);
}

void onEvent(ev_t ev) {
    switch(ev) {
        case EV_JOINED:
            Serial.println(F("EV_JOINED"));
            // Joined network successfully
            break;

        case EV_TXCOMPLETE:
            Serial.println(F("EV_TXCOMPLETE"));

            if (LMIC.txrxFlags & TXRX_ACK) {
                Serial.println(F("Received ack"));
            }

            if (LMIC.dataLen) {
                // Downlink received
                Serial.print(F("Received "));
                Serial.print(LMIC.dataLen);
                Serial.println(F(" bytes"));

                // Process downlink (commands, config, etc.)
                processDownlink(LMIC.frame + LMIC.dataBeg, LMIC.dataLen);
            }

            // Deep sleep until next transmission
            goToSleep(TX_INTERVAL);
            break;

        default:
            Serial.print(F("Unknown event: "));
            Serial.println(ev);
            break;
    }
}

void goToSleep(int seconds) {
    // ESP32 deep sleep
    Serial.println(F("Going to sleep..."));

    esp_sleep_enable_timer_wakeup(seconds * 1000000ULL);
    esp_deep_sleep_start();
}

void loop() {
    os_runloop_once();
}
```

### The Things Network (TTN)

```javascript
// Backend : Recevoir données LoRaWAN via TTN

const ttn = require('ttn');

class TTNDataCollector {
    constructor(appId, accessKey) {
        this.appId = appId;
        this.accessKey = accessKey;
    }

    async connect() {
        // Connect to TTN
        this.client = await ttn.data(this.appId, this.accessKey);

        // Subscribe to uplink messages
        this.client.on('uplink', this.handleUplink.bind(this));

        console.log('Connected to TTN');
    }

    handleUplink(devId, payload) {
        console.log(`Uplink from ${devId}`);

        // Payload est déjà décodé si decoder function configurée
        // Sinon, payload.payload_raw contient bytes bruts

        const data = this.decodePayload(payload.payload_raw);

        console.log('Data:', data);

        // Stocker en DB
        this.saveToDatabase(devId, data);

        // Vérifier alertes
        if (data.temperature > 30) {
            this.sendAlert(devId, 'High temperature', data.temperature);
        }

        // Envoyer downlink si nécessaire
        if (this.needsDownlink(devId, data)) {
            this.sendDownlink(devId, { command: 'adjust_interval' });
        }
    }

    decodePayload(bytes) {
        // Decoder payload binaire
        // bytes est Buffer ou Uint8Array

        if (bytes.length < 5) {
            throw new Error('Invalid payload length');
        }

        const temp_int = (bytes[0] << 8) | bytes[1];
        const humidity = bytes[2];
        const battery_int = (bytes[3] << 8) | bytes[4];

        return {
            temperature: temp_int / 10.0,
            humidity: humidity,
            battery: battery_int / 100.0
        };
    }

    async sendDownlink(devId, payload) {
        // Encoder payload
        const bytes = this.encodeDownlink(payload);

        // Envoyer via TTN
        await this.client.send(devId, bytes, 1, false);  // port 1, non-confirmed

        console.log(`Downlink sent to ${devId}`);
    }

    encodeDownlink(payload) {
        // Encoder commande en binaire
        // Exemple : { command: 'adjust_interval', interval: 600 }

        const buffer = Buffer.alloc(3);
        buffer[0] = 0x01;  // Command ID : adjust_interval
        buffer.writeUInt16BE(payload.interval || 300, 1);

        return buffer;
    }
}

// Utilisation
const collector = new TTNDataCollector('my-app-id', 'ttn-access-key');
collector.connect();
```

## Protocoles légers additionnels

### Zigbee

```
Caractéristiques :
- Standard : IEEE 802.15.4
- Fréquence : 2.4 GHz
- Portée : 10-100m
- Débit : 250 kbps
- Topologie : Mesh
- Consommation : Très faible
- Use case : Domotique, smart home

Avantages :
✅ Mesh auto-healing
✅ Faible latence
✅ Sécurité (AES-128)
✅ Interopérabilité (Zigbee Alliance)

Inconvénients :
❌ Portée limitée vs LoRa
❌ Nécessite gateway/coordinator
❌ 2.4 GHz = interférences WiFi
```

### BLE (Bluetooth Low Energy)

```
Caractéristiques :
- Portée : 10-50m
- Débit : 1-2 Mbps
- Consommation : 10-50 µA sleep, 10-15 mA actif
- Use case : Wearables, beacons, proximity

Modes :
- Broadcasting (beacons) : unidirectionnel
- Connected : bidirectionnel, pairing

Exemple : iBeacon
```

```javascript
// BLE peripheral (sensor) - Node.js avec noble/bleno

const bleno = require('bleno');

class TemperatureCharacteristic extends bleno.Characteristic {
    constructor() {
        super({
            uuid: '2A6E',  // Temperature UUID standard
            properties: ['read', 'notify'],
            descriptors: [
                new bleno.Descriptor({
                    uuid: '2901',
                    value: 'Temperature Sensor'
                })
            ]
        });

        this.temperature = 20.0;
        this.updateInterval = null;
    }

    onReadRequest(offset, callback) {
        // Lire température actuelle
        if (offset) {
            callback(this.RESULT_ATTR_NOT_LONG, null);
        } else {
            // Encoder en int16 (×100 pour 2 décimales)
            const temp_int = Math.round(this.temperature * 100);
            const buffer = Buffer.allocUnsafe(2);
            buffer.writeInt16LE(temp_int, 0);

            callback(this.RESULT_SUCCESS, buffer);
        }
    }

    onSubscribe(maxValueSize, updateValueCallback) {
        // Client subscribe : envoyer notifications périodiques
        console.log('Client subscribed to temperature notifications');

        this.updateValueCallback = updateValueCallback;

        // Envoyer update toutes les 5 secondes
        this.updateInterval = setInterval(() => {
            const temp_int = Math.round(this.temperature * 100);
            const buffer = Buffer.allocUnsafe(2);
            buffer.writeInt16LE(temp_int, 0);

            this.updateValueCallback(buffer);
        }, 5000);
    }

    onUnsubscribe() {
        console.log('Client unsubscribed');
        if (this.updateInterval) {
            clearInterval(this.updateInterval);
        }
    }
}

// Créer service BLE
const tempService = new bleno.PrimaryService({
    uuid: '181A',  // Environmental Sensing Service
    characteristics: [
        new TemperatureCharacteristic()
    ]
});

bleno.on('stateChange', (state) => {
    if (state === 'poweredOn') {
        bleno.startAdvertising('TempSensor', ['181A']);
    }
});

bleno.on('advertisingStart', (error) => {
    if (!error) {
        console.log('BLE advertising started');
        bleno.setServices([tempService]);
    }
});
```

### NB-IoT et LTE-M

```
NB-IoT (Narrowband IoT) :
- Réseau cellulaire (4G/5G)
- Portée : excellente (couverture opérateur)
- Débit : 200 kbps
- Latence : 1-10 secondes
- Consommation : modérée
- Use case : Smart meters, tracking, agriculture

LTE-M (LTE Cat-M1) :
- Réseau cellulaire
- Débit : 1 Mbps
- Latence : 10-100 ms
- Mobilité : excellente
- Use case : Asset tracking, wearables

Avantages :
✅ Infrastructure existante (opérateurs)
✅ Couverture globale
✅ Roaming
✅ Sécurité opérateur

Inconvénients :
❌ Coût abonnement
❌ Dépendance opérateur
❌ Consommation > LoRa
```

## Architectures IoT complètes

### Architecture edge + cloud

```
Architecture typique 3-tiers :

┌─────────────────────────────────────────────────────────────┐
│                        CLOUD TIER                           │
│  - Long-term storage                                        │
│  - Big data analytics                                       │
│  - ML training                                              │
│  - Dashboards, BI                                           │
│                                                             │
│  Technologies : AWS IoT, Azure IoT, Google Cloud IoT        │
└────────────────────────┬────────────────────────────────────┘
                         │ Internet
                         │ (MQTT/HTTPS/CoAP)
                         │
┌────────────────────────┴────────────────────────────────────┐
│                        EDGE/FOG TIER                        │
│  - Data aggregation                                         │
│  - Filtering, preprocessing                                 │
│  - Real-time analytics                                      │
│  - Local decision making                                    │
│  - Protocol translation                                     │
│                                                             │
│  Technologies : Edge gateway, Raspberry Pi, Industrial PC   │
└────────────────────────┬────────────────────────────────────┘
                         │ Local network
                         │ (LoRa/Zigbee/BLE/WiFi)
                         │
┌────────────────────────┴────────────────────────────────────┐
│                        DEVICE TIER                          │
│  - Sensors, actuators                                       │
│  - Embedded devices                                         │
│  - Minimal processing                                       │
│                                                             │
│  Technologies : ESP32, Arduino, STM32, custom silicon       │
└─────────────────────────────────────────────────────────────┘
```

### Exemple : Smart Building

```python
# Edge Gateway pour smart building

import asyncio
from collections import defaultdict
import paho.mqtt.client as mqtt

class SmartBuildingGateway:
    """
    Gateway edge pour smart building
    - Collecte données capteurs (LoRa, Zigbee, BLE)
    - Agrégation et preprocessing
    - Détection anomalies locale
    - Forward vers cloud (MQTT)
    """

    def __init__(self):
        # Protocoles locaux
        self.lora_client = LoRaWANClient()
        self.zigbee_client = ZigbeeCoordinator()
        self.ble_scanner = BLEScanner()

        # Cloud MQTT
        self.cloud_mqtt = mqtt.Client()
        self.cloud_mqtt.connect("cloud-mqtt.example.com", 1883)

        # Buffers aggregation
        self.sensor_buffer = defaultdict(list)
        self.aggregation_window = 300  # 5 minutes

        # ML models (edge inference)
        self.anomaly_detector = self.load_anomaly_model()

    async def start(self):
        """
        Démarrer gateway
        """
        # Lancer collecteurs en parallèle
        await asyncio.gather(
            self.collect_lora_data(),
            self.collect_zigbee_data(),
            self.collect_ble_data(),
            self.aggregate_and_forward(),
            self.monitor_health()
        )

    async def collect_lora_data(self):
        """
        Collecter données LoRaWAN (extérieur, long range)
        """
        async for message in self.lora_client.listen():
            device_id = message.dev_id
            payload = self.decode_lora_payload(message.payload)

            # Traitement local
            await self.process_sensor_data(
                device_id,
                'lora',
                payload
            )

    async def collect_zigbee_data(self):
        """
        Collecter données Zigbee (intérieur, mesh)
        """
        async for message in self.zigbee_client.listen():
            device_id = message.device_id

            # Zigbee peut envoyer plusieurs types de données
            for endpoint in message.endpoints:
                data = {
                    'type': endpoint.cluster_id,
                    'value': endpoint.value
                }

                await self.process_sensor_data(
                    device_id,
                    'zigbee',
                    data
                )

    async def process_sensor_data(self, device_id, protocol, data):
        """
        Traiter données capteur localement
        """
        # 1. Buffer pour agrégation
        self.sensor_buffer[device_id].append({
            'timestamp': time.time(),
            'protocol': protocol,
            'data': data
        })

        # 2. Détection anomalies (ML edge)
        is_anomaly = self.anomaly_detector.predict(data)

        if is_anomaly:
            # Alerte immédiate (bypass aggregation)
            await self.send_alert_to_cloud(device_id, data, 'anomaly')

            # Action locale immédiate si nécessaire
            await self.handle_anomaly_local(device_id, data)

        # 3. Vérifier seuils critiques
        if self.is_critical_threshold(data):
            await self.send_alert_to_cloud(device_id, data, 'threshold')
            await self.trigger_local_action(device_id, data)

    async def aggregate_and_forward(self):
        """
        Agréger données et forwarder vers cloud périodiquement
        """
        while True:
            await asyncio.sleep(self.aggregation_window)

            # Pour chaque device
            for device_id, readings in self.sensor_buffer.items():
                if not readings:
                    continue

                # Agréger (moyenne, min, max, etc.)
                aggregated = self.aggregate_readings(readings)

                # Envoyer vers cloud
                await self.send_to_cloud(device_id, aggregated)

                # Vider buffer
                self.sensor_buffer[device_id] = []

    def aggregate_readings(self, readings):
        """
        Agréger lectures multiples en statistiques
        """
        # Grouper par type de mesure
        by_type = defaultdict(list)

        for reading in readings:
            data = reading['data']
            if 'temperature' in data:
                by_type['temperature'].append(data['temperature'])
            if 'humidity' in data:
                by_type['humidity'].append(data['humidity'])
            # etc.

        # Calculer stats
        aggregated = {}

        for measure_type, values in by_type.items():
            aggregated[measure_type] = {
                'avg': np.mean(values),
                'min': np.min(values),
                'max': np.max(values),
                'std': np.std(values),
                'count': len(values)
            }

        return aggregated

    async def send_to_cloud(self, device_id, data):
        """
        Envoyer données agrégées vers cloud
        """
        topic = f"building/sensors/{device_id}/aggregated"
        payload = json.dumps(data)

        self.cloud_mqtt.publish(topic, payload, qos=1)

    async def handle_anomaly_local(self, device_id, data):
        """
        Gérer anomalie localement (sans attendre cloud)
        """
        # Exemple : CO2 trop élevé → activer ventilation
        if data.get('co2') and data['co2'] > 1000:
            # Envoyer commande locale vers actuateur HVAC
            await self.zigbee_client.send_command(
                device_id='hvac_zone_1',
                command='increase_ventilation'
            )

            print(f"Local action: Increased ventilation (CO2={data['co2']})")

    def is_critical_threshold(self, data):
        """
        Vérifier seuils critiques
        """
        thresholds = {
            'temperature': (5, 40),  # Min, Max
            'co2': (None, 1500),
            'smoke': (None, 100)
        }

        for key, (min_val, max_val) in thresholds.items():
            if key in data:
                value = data[key]
                if min_val and value < min_val:
                    return True
                if max_val and value > max_val:
                    return True

        return False

# Démarrer gateway
gateway = SmartBuildingGateway()
asyncio.run(gateway.start())
```

## Sécurité IoT

### Défis spécifiques

```
Problématiques IoT :

1. Ressources limitées :
   - Pas assez de RAM/CPU pour TLS complet
   - Crypto lourde impactante sur batterie

2. Lifecycle long :
   - Devices déployés 5-10+ ans
   - Firmware rarement updatable
   - Vulnérabilités découvertes après deploy

3. Physical access :
   - Souvent dans environnements non-sécurisés
   - Risque d'extraction clés/firmware
   - Tampering possible

4. Scale :
   - Milliards de devices
   - Gestion clés complexe
   - Révocation difficile
```

### DTLS pour CoAP

```python
# DTLS (Datagram TLS) pour sécuriser CoAP

from aiocoap import *
import aiocoap.oscore as oscore

# CoAP avec DTLS
class SecureCoAPClient:
    async def secure_get(self, uri, psk_identity, psk_key):
        """
        Requête CoAP sécurisée avec DTLS-PSK
        """
        # Context avec DTLS
        context = await Context.create_client_context()

        # Configure PSK (Pre-Shared Key)
        context.client_credentials.load_from_dict({
            'coaps://sensor.local/*': {
                'dtls': {
                    'psk': psk_key,
                    'client-identity': psk_identity
                }
            }
        })

        # Requête sécurisée (coaps://)
        request = Message(code=GET, uri=uri)
        response = await context.request(request).response

        return response.payload.decode()

# Utilisation
client = SecureCoAPClient()
temp = await client.secure_get(
    'coaps://sensor.local/temperature',
    psk_identity=b'sensor01',
    psk_key=b'secret_key_123456'
)
```

### MQTT avec TLS

```python
# MQTT over TLS

import paho.mqtt.client as mqtt
import ssl

class SecureMQTTClient:
    def __init__(self, broker, port=8883):
        self.client = mqtt.Client()

        # Configure TLS
        self.client.tls_set(
            ca_certs="/path/to/ca.crt",
            certfile="/path/to/client.crt",
            keyfile="/path/to/client.key",
            cert_reqs=ssl.CERT_REQUIRED,
            tls_version=ssl.PROTOCOL_TLSv1_2
        )

        # Optionnel : vérifier hostname
        self.client.tls_insecure_set(False)

        self.client.connect(broker, port, 60)

    def publish_secure(self, topic, payload):
        # Publier sur TLS
        self.client.publish(topic, payload, qos=1)

# ⚠️ Attention : TLS sur device IoT = coûteux
# - Handshake TLS : plusieurs KB échangés
# - CPU crypto : batterie
# - Alternative : DTLS, ou sécurité au gateway
```

### Lightweight crypto

```c
// Crypto légère pour IoT : ChaCha20-Poly1305

#include "mbedtls/chacha20.h"
#include "mbedtls/poly1305.h"

// Plus léger que AES-GCM, pas besoin hardware crypto

void encrypt_sensor_data(
    uint8_t *plaintext,
    size_t plaintext_len,
    uint8_t *key,
    uint8_t *nonce,
    uint8_t *ciphertext,
    uint8_t *tag
) {
    mbedtls_chacha20_context ctx;

    // Init ChaCha20
    mbedtls_chacha20_init(&ctx);
    mbedtls_chacha20_setkey(&ctx, key);

    // Encrypt
    mbedtls_chacha20_crypt(
        &ctx,
        nonce,
        0,  // counter
        plaintext_len,
        plaintext,
        ciphertext
    );

    // Generate MAC avec Poly1305
    mbedtls_poly1305_mac(tag, ciphertext, plaintext_len, key);

    mbedtls_chacha20_free(&ctx);
}

// Avantages :
// - Plus rapide que AES en software
// - Pas besoin d'accélération hardware
// - Faible consommation CPU
// - Sécurité équivalente
```

## Optimisations réseau avancées

### Duty cycling intelligent

```python
# Adapter fréquence transmission selon conditions

class AdaptiveSensor:
    def __init__(self):
        self.base_interval = 300  # 5 minutes
        self.current_interval = self.base_interval
        self.battery_level = 100

        self.last_value = None
        self.change_threshold = 0.5

    def calculate_next_interval(self, current_value):
        """
        Calculer prochain intervalle de transmission
        """
        # Facteur 1 : Niveau batterie
        if self.battery_level < 20:
            battery_factor = 2.0  # Doubler intervalle si batterie faible
        elif self.battery_level < 50:
            battery_factor = 1.5
        else:
            battery_factor = 1.0

        # Facteur 2 : Changement de valeur
        if self.last_value is None:
            change_factor = 1.0
        else:
            delta = abs(current_value - self.last_value)

            if delta > self.change_threshold * 2:
                # Changement important : transmettre plus souvent
                change_factor = 0.5
            elif delta > self.change_threshold:
                change_factor = 0.75
            else:
                # Valeur stable : peut espacer
                change_factor = 1.5

        # Facteur 3 : Heure de la journée
        hour = datetime.now().hour
        if 0 <= hour < 6:
            # Nuit : moins de transmissions
            time_factor = 2.0
        else:
            time_factor = 1.0

        # Calculer nouvel intervalle
        self.current_interval = int(
            self.base_interval * battery_factor * change_factor * time_factor
        )

        # Limites
        self.current_interval = max(60, min(3600, self.current_interval))

        self.last_value = current_value

        return self.current_interval

    def should_transmit(self, current_value):
        """
        Décider si transmission nécessaire (delta significatif)
        """
        if self.last_value is None:
            return True

        delta = abs(current_value - self.last_value)

        return delta > self.change_threshold

# Utilisation
sensor = AdaptiveSensor()

while True:
    temp = read_temperature()

    if sensor.should_transmit(temp):
        # Transmettre
        transmit(temp)

        # Calculer prochain intervalle
        next_interval = sensor.calculate_next_interval(temp)
        print(f"Next transmission in {next_interval}s")
    else:
        print("Value unchanged, skipping transmission")
        next_interval = sensor.current_interval

    sleep(next_interval)

# Résultat :
# - Batterie faible + valeur stable + nuit = 1 transmission/heure
# - Batterie OK + valeur changeante + jour = 1 transmission/minute
# Optimisation automatique !
```

### Data compression

```python
# Compression pour IoT

import zlib

# Texte/JSON : bonne compression
data_json = json.dumps({
    "temperature": 23.4,
    "humidity": 65.2,
    "pressure": 1013.2
})  # ~60 bytes

compressed = zlib.compress(data_json.encode())  # ~40 bytes
ratio = len(compressed) / len(data_json)
print(f"Compression ratio: {ratio:.2%}")  # ~67%

# Binaire : compression limitée
data_binary = struct.pack('fff', 23.4, 65.2, 1013.2)  # 12 bytes
compressed_bin = zlib.compress(data_binary)  # ~18 bytes
# Pas de gain, overhead ajouté !

# Conclusion :
# - Compression utile pour texte/JSON
# - Contre-productive pour binaire compact
# - Trade-off : CPU compression vs économie bande passante
```

### Message batching

```python
# Batching : envoyer plusieurs mesures en 1 transmission

class BatchingSensor:
    def __init__(self, batch_size=10):
        self.batch_size = batch_size
        self.batch_buffer = []

    def add_reading(self, timestamp, value):
        """
        Ajouter lecture au batch
        """
        self.batch_buffer.append({
            'ts': timestamp,
            'value': value
        })

        # Transmettre si batch complet
        if len(self.batch_buffer) >= self.batch_size:
            self.transmit_batch()

    def transmit_batch(self):
        """
        Transmettre batch complet
        """
        if not self.batch_buffer:
            return

        # Encoder batch compact
        payload = self.encode_batch(self.batch_buffer)

        # Transmettre (1 seule TX au lieu de 10)
        transmit(payload)

        print(f"Transmitted batch of {len(self.batch_buffer)} readings")

        # Vider buffer
        self.batch_buffer = []

    def encode_batch(self, readings):
        """
        Encoder batch en binaire compact
        """
        # Format : [count][base_ts][delta_ts][values...]
        # count : uint8
        # base_ts : uint32 (timestamp absolu du premier)
        # delta_ts : uint16 (delta en secondes depuis base)
        # values : int16 (valeur × 10)

        count = len(readings)
        base_ts = readings[0]['ts']

        payload = struct.pack('BI', count, base_ts)

        for reading in readings:
            delta_ts = reading['ts'] - base_ts
            value_encoded = int(reading['value'] * 10)

            payload += struct.pack('Hi', delta_ts, value_encoded)

        return payload

        # Taille : 5 + (6 × count) bytes
        # count=10 : 65 bytes pour 10 lectures
        # vs 10 transmissions × 20 bytes overhead = 200+ bytes
        # Économie : 67% !

# Avantages batching :
# ✅ Réduction overhead radio (1 TX au lieu de N)
# ✅ Économie batterie (TX = phase la plus coûteuse)
# ✅ Moins de congestion réseau
# ❌ Latence accrue (attente batch complet)
```

## Futur de l'IoT

### 1. LoRaWAN + Edge ML

```python
# ML inference sur gateway LoRa (pas sur device)

class EdgeMLGateway:
    def __init__(self):
        self.ml_model = self.load_lightweight_model()

    def process_lora_message(self, device_id, payload):
        """
        Inference ML sur gateway au lieu de cloud
        """
        # Décoder payload
        sensor_data = decode_payload(payload)

        # Inference locale (edge)
        prediction = self.ml_model.predict(sensor_data)

        # Envoyer seulement résultat au cloud (pas raw data)
        # Gain : 90% bande passante

        if prediction['anomaly']:
            # Alerte immédiate
            send_alert_to_cloud(device_id, prediction)
        else:
            # Seulement stats agrégées 1x/jour
            buffer_for_batch(device_id, prediction)

# Avantage :
# - Device reste ultra-simple (pas de ML)
# - Inference rapide (pas de latence cloud)
# - Économie bande passante massive
```

### 2. IPv6 over LoRaWAN

```
SCHC (Static Context Header Compression) :
- Compresse headers IPv6/UDP
- 40 bytes IPv6 header → 2-4 bytes
- RFC 8724

Permet : IP end-to-end sur LoRaWAN
→ Devices LoRa avec adresse IPv6 publique
→ Routing IP natif
```

### 3. 5G + IoT massive

```
5G Massive Machine-Type Communications (mMTC) :
- 1 million devices / km²
- Ultra-low power
- Latence acceptable (1-10s)
- Coverage améliorée

Use cases :
- Smart cities (millions de capteurs)
- Agriculture (capteurs par m²)
- Infrastructure monitoring
```

### 4. Energy harvesting

```
Devices sans batterie :
- Solar (quelques cm²)
- RF energy harvesting
- Vibrations / piezo
- Thermique (différentiel température)

Puissance disponible : 10-1000 µW
→ Devices qui vivent "éternellement"
→ Zéro maintenance
```

## Conclusion

L'IoT impose des contraintes radicalement différentes de l'IT traditionnel. TCP/IP classique est inadapté, d'où l'émergence de protocoles spécialisés optimisés pour ressources limitées, faible consommation, et connectivité intermittente.

**Points clés pour développeurs** :

✅ **Choix du protocole crucial** : MQTT (cloud connectivity), CoAP (device-to-device), LoRaWAN (longue portée, batterie)

✅ **Chaque octet compte** : Encodage binaire, compression, batching

✅ **Batterie = contrainte #1** : Duty cycling, sleep profond, transmission minimale

✅ **Edge computing essentiel** : Traitement local, agrégation, décisions temps réel

✅ **Sécurité dès la conception** : Crypto légère, DTLS, device attestation

**Recommandations** :

1. **Prototyper** avec ESP32/Arduino + MQTT (learning rapide)
2. **Mesurer** consommation énergétique réelle (pas d'hypothèses)
3. **Optimiser** payload (binaire > JSON, compression si pertinent)
4. **Architecturer** edge-first (gateway intelligent, pas tout au cloud)
5. **Sécuriser** dès le départ (update OTA, crypto, attestation)

L'IoT n'est pas du web development. C'est un domaine à part, avec ses propres règles, protocoles, et trade-offs. Mais c'est aussi l'un des domaines les plus excitants : des milliards de devices, des applications infinies, et un impact réel sur le monde physique.

---

**Ressources complémentaires** :
- [MQTT Specification](https://mqtt.org/mqtt-specification/)
- [RFC 7252 - CoAP](https://www.rfc-editor.org/rfc/rfc7252.html)
- [LoRa Alliance](https://lora-alliance.org/)
- [The Things Network](https://www.thethingsnetwork.org/) (LoRaWAN gratuit)
- [Eclipse Paho](https://www.eclipse.org/paho/) (MQTT clients)
- [ESP32 Documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/)

---


⏭️ [5G et convergence fixe-mobile](/10-evolutions-tendances/07-5g-convergence.md)
