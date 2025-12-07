🔝 Retour au [Sommaire](/SOMMAIRE.md)

# Exemples Go

## Introduction

Go (Golang) est particulièrement adapté à la programmation réseau grâce à :
- Son package `net` standard puissant et simple
- Les **goroutines** pour une concurrence légère et efficace
- Les **channels** pour la communication entre goroutines
- Sa compilation en binaire natif (performance)
- Sa gestion d'erreur explicite
- Son garbage collector optimisé pour les serveurs

**Versions Go :** Ces exemples sont compatibles Go 1.16+

**Prérequis :**
- Installation de Go (https://go.dev/dl/)
- Connaissances de base en Go
- Compréhension des concepts TCP/IP

---

## Organisation des exemples

```
go/
├── README.md (ce fichier)
├── 01-tcp-simple/
│   ├── server.go
│   ├── client.go
│   └── README.md
├── 02-concurrent-server/
│   ├── server.go
│   ├── client.go
│   └── README.md
├── 03-udp/
│   ├── server.go
│   ├── client.go
│   └── README.md
└── 04-websocket/
    ├── server.go
    ├── client.go
    └── README.md
```

---

## Pourquoi Go pour le réseau ?

### Comparaison avec Python

| Aspect | Python | Go |
|--------|--------|-----|
| **Performance** | Interprété, plus lent | Compilé, très rapide |
| **Concurrence** | Threading (GIL) ou asyncio | Goroutines natives |
| **Mémoire** | ~8MB par thread | ~2KB par goroutine |
| **Clients simultanés** | ~1000 | 10000+ |
| **Courbe apprentissage** | Facile | Moyenne |
| **Déploiement** | Interpréteur requis | Binaire unique |

### Forces de Go pour le réseau

- ✅ **Goroutines légères** : Des milliers de connexions simultanées
- ✅ **Channels** : Communication sûre entre goroutines
- ✅ **Performance** : Proche du C
- ✅ **Simplicité** : Moins de code que Java/C++
- ✅ **Standard library** : Package `net` très complet
- ✅ **Binaires statiques** : Déploiement simple

---

## 01. TCP Simple - Echo Server

### Concept

Identique à Python, mais avec la syntaxe et les idiomes Go.

### server.go - Version basique

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
)

const (
    // Configuration
    HOST = "127.0.0.1"
    PORT = "8080"
    TYPE = "tcp"
)

func main() {
    // Créer un listener TCP
    // Listen() retourne (Listener, error)
    // Format d'adresse: "host:port"
    listener, err := net.Listen(TYPE, HOST+":"+PORT)
    if err != nil {
        log.Fatal("Erreur Listen:", err)
    }
    defer listener.Close() // Fermeture automatique à la fin

    fmt.Printf("[SERVEUR] Démarré sur %s:%s\n", HOST, PORT)
    fmt.Println("[SERVEUR] En attente de connexions...")

    // Boucle infinie pour accepter les connexions
    for {
        // Accepter une connexion (BLOQUANT)
        // Accept() retourne (Conn, error)
        conn, err := listener.Accept()
        if err != nil {
            log.Println("Erreur Accept:", err)
            continue // Continuer malgré l'erreur
        }

        fmt.Printf("\n[CONNEXION] Nouveau client: %s\n", conn.RemoteAddr())

        // Gérer le client
        // Pour l'instant, traitement séquentiel (un client à la fois)
        handleClient(conn)
    }
}

// handleClient gère la communication avec un client
func handleClient(conn net.Conn) {
    // defer exécute à la fin de la fonction
    defer conn.Close()
    defer fmt.Printf("[DÉCONNEXION] Client %s déconnecté\n", conn.RemoteAddr())

    // Buffer pour lire les données
    buffer := make([]byte, 1024)

    // Lire les données
    // Read() retourne (n int, err error)
    n, err := conn.Read(buffer)
    if err != nil {
        if err != io.EOF {
            log.Println("Erreur Read:", err)
        }
        return
    }

    // Afficher ce qui a été reçu
    message := string(buffer[:n])
    fmt.Printf("[REÇU] %d octets: %s\n", n, message)

    // Echo : renvoyer les données
    // Write() retourne (n int, err error)
    n, err = conn.Write(buffer[:n])
    if err != nil {
        log.Println("Erreur Write:", err)
        return
    }

    fmt.Printf("[ENVOYÉ] %d octets (echo)\n", n)
}
```

### Explications détaillées

#### 1. Package net

```go
import "net"
```

**Le package `net` fournit :**
- `Listen()` : Créer un listener
- `Dial()` : Se connecter à un serveur
- `Conn` : Interface pour les connexions
- `Listener` : Interface pour écouter
- Support TCP, UDP, Unix sockets

#### 2. net.Listen()

```go
listener, err := net.Listen("tcp", "127.0.0.1:8080")
```

**Paramètres :**
- `"tcp"` : Type de réseau (tcp, tcp4, tcp6, udp, unix)
- `"127.0.0.1:8080"` : Adresse au format "host:port"

**Équivalent à (en C/Python) :**
```
socket()
bind()
listen()
```

**Gestion d'erreur Go :**
```go
if err != nil {
    // Gérer l'erreur
}
```

#### 3. defer statement

```go
defer listener.Close()
defer conn.Close()
```

**Comportement :**
- `defer` planifie l'exécution d'une fonction pour la fin de la fonction englobante
- Utile pour libérer des ressources (fichiers, sockets, locks)
- Exécuté même en cas de panic

**Ordre d'exécution :**
```go
func example() {
    defer fmt.Println("3")
    defer fmt.Println("2")
    defer fmt.Println("1")
    fmt.Println("Début")
}
// Affiche: Début, 1, 2, 3 (LIFO - Last In First Out)
```

#### 4. Gestion d'erreur idiomatique

```go
n, err := conn.Read(buffer)
if err != nil {
    if err != io.EOF {
        log.Println("Erreur:", err)
    }
    return
}
```

**Pattern Go standard :**
1. Les fonctions retournent `(résultat, error)`
2. Toujours vérifier `err != nil`
3. Gérer l'erreur immédiatement

**EOF (End Of File) :**
- `io.EOF` signale la fin normale d'une connexion
- Pas vraiment une erreur, plutôt un signal

#### 5. Slices et buffers

```go
buffer := make([]byte, 1024)  // Créer un slice de 1024 bytes
n, err := conn.Read(buffer)    // Lire dans le buffer
message := string(buffer[:n])  // Convertir n premiers bytes en string
```

**Slicing :**
- `buffer[:n]` : Les n premiers éléments
- `buffer[n:]` : Du nième à la fin
- `buffer[a:b]` : De a (inclus) à b (exclu)

---

### client.go - Client simple

```go
package main

import (
    "fmt"
    "log"
    "net"
    "os"
    "strings"
)

const (
    SERVER_HOST = "127.0.0.1"
    SERVER_PORT = "8080"
    TYPE        = "tcp"
)

func main() {
    // Message à envoyer (depuis arguments ou défaut)
    message := "Hello, Server!"
    if len(os.Args) > 1 {
        message = strings.Join(os.Args[1:], " ")
    }

    // Se connecter au serveur
    // Dial() retourne (Conn, error)
    fmt.Printf("[CLIENT] Connexion à %s:%s...\n", SERVER_HOST, SERVER_PORT)
    conn, err := net.Dial(TYPE, SERVER_HOST+":"+SERVER_PORT)
    if err != nil {
        log.Fatal("Erreur Dial:", err)
    }
    defer conn.Close()

    fmt.Println("[CLIENT] Connecté!")

    // Envoyer le message
    dataToSend := []byte(message)
    n, err := conn.Write(dataToSend)
    if err != nil {
        log.Fatal("Erreur Write:", err)
    }
    fmt.Printf("[ENVOI] %d octets: %s\n", n, message)

    // Recevoir la réponse
    buffer := make([]byte, 1024)
    n, err = conn.Read(buffer)
    if err != nil {
        log.Fatal("Erreur Read:", err)
    }

    response := string(buffer[:n])
    fmt.Printf("[RÉPONSE] %d octets: %s\n", n, response)

    // Vérifier l'echo
    if response == message {
        fmt.Println("[SUCCESS] Echo correct!")
    } else {
        fmt.Println("[WARNING] Echo incorrect!")
    }
}
```

### Explications client

#### net.Dial()

```go
conn, err := net.Dial("tcp", "127.0.0.1:8080")
```

**Équivalent à (en C/Python) :**
```
socket()
connect()
```

**Variantes :**
```go
// Dial simple
conn, err := net.Dial("tcp", "example.com:80")

// DialTimeout avec timeout
conn, err := net.DialTimeout("tcp", "example.com:80", 5*time.Second)

// DialTCP pour options TCP spécifiques
tcpAddr, _ := net.ResolveTCPAddr("tcp", "example.com:80")
conn, err := net.DialTCP("tcp", nil, tcpAddr)
```

#### Interface net.Conn

```go
type Conn interface {
    Read(b []byte) (n int, err error)
    Write(b []byte) (n int, err error)
    Close() error
    LocalAddr() Addr
    RemoteAddr() Addr
    SetDeadline(t time.Time) error
    SetReadDeadline(t time.Time) error
    SetWriteDeadline(t time.Time) error
}
```

**Méthodes utiles :**
- `Read()` / `Write()` : I/O
- `Close()` : Fermer la connexion
- `RemoteAddr()` : Adresse du pair
- `SetDeadline()` : Définir un timeout

---

### Compilation et exécution

**Compiler :**
```bash
# Compiler le serveur
go build -o server server.go

# Compiler le client
go build -o client client.go
```

**Exécuter :**

**Terminal 1 - Serveur :**
```bash
$ ./server
[SERVEUR] Démarré sur 127.0.0.1:8080
[SERVEUR] En attente de connexions...
```

**Terminal 2 - Client :**
```bash
$ ./client "Bonjour Go!"
[CLIENT] Connexion à 127.0.0.1:8080...
[CLIENT] Connecté!
[ENVOI] 11 octets: Bonjour Go!
[RÉPONSE] 11 octets: Bonjour Go!
[SUCCESS] Echo correct!
```

**Alternative - go run (sans compilation) :**
```bash
# Terminal 1
go run server.go

# Terminal 2
go run client.go "Test message"
```

---

## 02. Serveur Concurrent - Goroutines

### Le problème de la version simple

Le serveur précédent traite **un client à la fois** :
```go
for {
    conn, _ := listener.Accept()
    handleClient(conn)  // BLOQUANT - autres clients attendent
}
```

### Solution : Goroutines

Lancer `handleClient()` dans une **goroutine** pour traiter plusieurs clients simultanément.

### server.go - Version concurrente

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
    "sync"
    "sync/atomic"
)

const (
    HOST = "127.0.0.1"
    PORT = "8080"
    TYPE = "tcp"
)

var (
    // Compteur de clients (thread-safe avec atomic)
    clientCounter uint64

    // WaitGroup pour attendre la fin des goroutines
    wg sync.WaitGroup
)

func main() {
    listener, err := net.Listen(TYPE, HOST+":"+PORT)
    if err != nil {
        log.Fatal("Erreur Listen:", err)
    }
    defer listener.Close()

    fmt.Printf("[SERVEUR] Démarré sur %s:%s\n", HOST, PORT)
    fmt.Println("[SERVEUR] Mode: Goroutines concurrentes")
    fmt.Println("[SERVEUR] En attente de connexions...")

    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Println("Erreur Accept:", err)
            continue
        }

        // Incrémenter le compteur (atomique)
        clientID := atomic.AddUint64(&clientCounter, 1)

        fmt.Printf("\n[SERVEUR] Nouvelle connexion #%d: %s\n",
            clientID, conn.RemoteAddr())

        // Incrémenter le WaitGroup
        wg.Add(1)

        // Lancer handleClient dans une goroutine
        // go keyword : exécution concurrente
        go handleClient(conn, clientID)

        // Le serveur continue immédiatement à accepter d'autres connexions
        // pendant que handleClient s'exécute en parallèle
    }

    // Note: Ce code n'est jamais atteint (boucle infinie)
    // Dans une vraie app, on attendrait les goroutines:
    // wg.Wait()
}

// handleClient gère un client dans une goroutine séparée
func handleClient(conn net.Conn, clientID uint64) {
    // Décrémenter le WaitGroup à la fin
    defer wg.Done()
    defer conn.Close()
    defer fmt.Printf("[GOROUTINE #%d] Terminée\n", clientID)

    fmt.Printf("[GOROUTINE #%d] Démarrée pour %s\n", clientID, conn.RemoteAddr())

    // Boucle pour gérer plusieurs messages du même client
    for {
        buffer := make([]byte, 1024)

        // Lire les données
        n, err := conn.Read(buffer)
        if err != nil {
            if err == io.EOF {
                fmt.Printf("[GOROUTINE #%d] Client a fermé la connexion\n", clientID)
            } else {
                log.Printf("[GOROUTINE #%d] Erreur Read: %v\n", clientID, err)
            }
            break
        }

        // Afficher
        message := string(buffer[:n])
        fmt.Printf("[GOROUTINE #%d] Reçu: %s\n", clientID, message)

        // Echo
        n, err = conn.Write(buffer[:n])
        if err != nil {
            log.Printf("[GOROUTINE #%d] Erreur Write: %v\n", clientID, err)
            break
        }

        fmt.Printf("[GOROUTINE #%d] Echo envoyé (%d octets)\n", clientID, n)
    }
}
```

### Explications Goroutines

#### 1. Qu'est-ce qu'une goroutine ?

**Définition :**
- Thread léger géré par le runtime Go
- Stack initiale de ~2KB (vs ~8MB pour un thread OS)
- Multiplexées sur des threads OS (M:N threading)

**Création :**
```go
go functionName(args)  // Exécute functionName dans une goroutine
```

**Exemple :**
```go
func sayHello() {
    fmt.Println("Hello")
}

func main() {
    go sayHello()           // Goroutine
    sayHello()              // Synchrone
    time.Sleep(time.Second) // Attendre la goroutine
}
```

#### 2. Atomic operations

```go
import "sync/atomic"

var counter uint64

// Incrémenter de manière thread-safe
newValue := atomic.AddUint64(&counter, 1)

// Lire
value := atomic.LoadUint64(&counter)

// Écrire
atomic.StoreUint64(&counter, 100)
```

**Pourquoi atomic ?**
- Plusieurs goroutines accèdent à `clientCounter`
- Sans atomic, race condition possible
- Plus rapide qu'un mutex pour des opérations simples

#### 3. sync.WaitGroup

```go
var wg sync.WaitGroup

wg.Add(1)      // Incrémenter le compteur
go func() {
    defer wg.Done()  // Décrémenter à la fin
    // Travail...
}()

wg.Wait()      // Attendre que le compteur arrive à 0
```

**Utilité :**
- Attendre la fin de plusieurs goroutines
- Pattern courant pour la synchronisation

#### 4. Scheduler Go

```
Application Go
    |
    ├─ Goroutine 1 ──┐
    ├─ Goroutine 2 ──┤
    ├─ Goroutine 3 ──┼──> Scheduler Go (M:N)
    ├─ Goroutine 4 ──┤
    └─ Goroutine N ──┘
         |
         v
    Thread OS 1, 2, 3... (GOMAXPROCS)
```

**GOMAXPROCS :**
- Nombre de threads OS utilisés
- Par défaut = nombre de CPU cores
- `runtime.GOMAXPROCS(n)` pour changer

---

### Client interactif

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "os"
    "strings"
)

const (
    SERVER_HOST = "127.0.0.1"
    SERVER_PORT = "8080"
)

func main() {
    // Connexion
    conn, err := net.Dial("tcp", SERVER_HOST+":"+SERVER_PORT)
    if err != nil {
        log.Fatal("Erreur Dial:", err)
    }
    defer conn.Close()

    fmt.Printf("Connecté à %s:%s\n", SERVER_HOST, SERVER_PORT)
    fmt.Println("Tapez vos messages (Ctrl+C pour quitter):")

    // Scanner pour lire stdin
    scanner := bufio.NewScanner(os.Stdin)

    for {
        fmt.Print("> ")

        // Lire une ligne
        if !scanner.Scan() {
            break
        }

        message := strings.TrimSpace(scanner.Text())
        if message == "" {
            continue
        }

        // Envoyer
        _, err := conn.Write([]byte(message))
        if err != nil {
            log.Println("Erreur Write:", err)
            break
        }

        // Recevoir la réponse
        buffer := make([]byte, 1024)
        n, err := conn.Read(buffer)
        if err != nil {
            log.Println("Erreur Read:", err)
            break
        }

        response := string(buffer[:n])
        fmt.Printf("Echo: %s\n", response)
    }

    if err := scanner.Err(); err != nil {
        log.Println("Erreur scanner:", err)
    }
}
```

### Test concurrent

**Terminal 1 - Serveur :**
```bash
$ go run server.go
[SERVEUR] Démarré sur 127.0.0.1:8080
[SERVEUR] Mode: Goroutines concurrentes
[SERVEUR] En attente de connexions...
```

**Terminal 2 - Client 1 :**
```bash
$ go run client_interactive.go
Connecté à 127.0.0.1:8080
Tapez vos messages (Ctrl+C pour quitter):
> Message du client 1
Echo: Message du client 1
```

**Terminal 3 - Client 2 :**
```bash
$ go run client_interactive.go
Connecté à 127.0.0.1:8080
Tapez vos messages (Ctrl+C pour quitter):
> Message du client 2
Echo: Message du client 2
```

**Terminal 1 affiche :**
```
[SERVEUR] Nouvelle connexion #1: 127.0.0.1:54321
[GOROUTINE #1] Démarrée pour 127.0.0.1:54321

[SERVEUR] Nouvelle connexion #2: 127.0.0.1:54322
[GOROUTINE #2] Démarrée pour 127.0.0.1:54322

[GOROUTINE #1] Reçu: Message du client 1
[GOROUTINE #1] Echo envoyé (20 octets)
[GOROUTINE #2] Reçu: Message du client 2
[GOROUTINE #2] Echo envoyé (20 octets)
```

---

## 03. UDP - Sans connexion

### udp/server.go

```go
package main

import (
    "fmt"
    "log"
    "net"
)

const (
    HOST = "127.0.0.1"
    PORT = "8080"
)

func main() {
    // Résoudre l'adresse UDP
    udpAddr, err := net.ResolveUDPAddr("udp", HOST+":"+PORT)
    if err != nil {
        log.Fatal("Erreur ResolveUDPAddr:", err)
    }

    // Créer une socket UDP
    // ListenUDP() retourne (*UDPConn, error)
    conn, err := net.ListenUDP("udp", udpAddr)
    if err != nil {
        log.Fatal("Erreur ListenUDP:", err)
    }
    defer conn.Close()

    fmt.Printf("[SERVEUR UDP] Démarré sur %s:%s\n", HOST, PORT)
    fmt.Println("[SERVEUR UDP] En attente de datagrammes...")

    // Buffer pour recevoir les datagrammes
    buffer := make([]byte, 1024)

    for {
        // Recevoir un datagramme
        // ReadFromUDP() retourne (n, addr, err)
        n, clientAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            log.Println("Erreur ReadFromUDP:", err)
            continue
        }

        // Afficher
        message := string(buffer[:n])
        fmt.Printf("\n[REÇU DE %s] %s\n", clientAddr, message)
        fmt.Printf("[REÇU] %d octets\n", n)

        // Echo : renvoyer au même client
        // WriteToUDP() retourne (n, err)
        n, err = conn.WriteToUDP(buffer[:n], clientAddr)
        if err != nil {
            log.Println("Erreur WriteToUDP:", err)
            continue
        }

        fmt.Printf("[ENVOYÉ À %s] Echo (%d octets)\n", clientAddr, n)
    }
}
```

### udp/client.go

```go
package main

import (
    "fmt"
    "log"
    "net"
    "os"
    "strings"
    "time"
)

const (
    SERVER_HOST = "127.0.0.1"
    SERVER_PORT = "8080"
)

func main() {
    // Message
    message := "Hello UDP!"
    if len(os.Args) > 1 {
        message = strings.Join(os.Args[1:], " ")
    }

    // Résoudre l'adresse du serveur
    serverAddr, err := net.ResolveUDPAddr("udp", SERVER_HOST+":"+SERVER_PORT)
    if err != nil {
        log.Fatal("Erreur ResolveUDPAddr:", err)
    }

    // Créer une socket UDP
    // DialUDP() pour se "connecter" (associer à une adresse distante)
    conn, err := net.DialUDP("udp", nil, serverAddr)
    if err != nil {
        log.Fatal("Erreur DialUDP:", err)
    }
    defer conn.Close()

    fmt.Printf("[CLIENT UDP] Envoi à %s\n", serverAddr)
    fmt.Printf("[CLIENT UDP] Message: %s\n", message)

    // Envoyer le datagramme
    data := []byte(message)
    n, err := conn.Write(data)
    if err != nil {
        log.Fatal("Erreur Write:", err)
    }
    fmt.Printf("[CLIENT UDP] %d octets envoyés\n", n)

    // Définir un timeout pour la réception
    conn.SetReadDeadline(time.Now().Add(5 * time.Second))

    // Recevoir la réponse
    fmt.Println("[CLIENT UDP] En attente de réponse (timeout 5s)...")
    buffer := make([]byte, 1024)
    n, err = conn.Read(buffer)
    if err != nil {
        if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
            log.Fatal("Timeout - Aucune réponse du serveur")
        }
        log.Fatal("Erreur Read:", err)
    }

    response := string(buffer[:n])
    fmt.Printf("[CLIENT UDP] Réponse: %s\n", response)

    // Vérifier l'echo
    if response == message {
        fmt.Println("[SUCCESS] Echo correct!")
    }
}
```

### Explications UDP

#### 1. Types spécifiques UDP

```go
// Adresse UDP
type UDPAddr struct {
    IP   IP
    Port int
    Zone string
}

// Connexion UDP
type UDPConn struct {
    // ...
}
```

**Méthodes UDPConn :**
- `ReadFromUDP(b []byte) (n int, addr *UDPAddr, err error)`
- `WriteToUDP(b []byte, addr *UDPAddr) (n int, err error)`
- `Read(b []byte) (n int, err error)` (si "connecté")
- `Write(b []byte) (n int, err error)` (si "connecté")

#### 2. UDP "connecté" vs non-connecté

**Non-connecté (serveur) :**
```go
conn, _ := net.ListenUDP("udp", addr)
n, clientAddr, _ := conn.ReadFromUDP(buffer)
conn.WriteToUDP(data, clientAddr)
```

**"Connecté" (client) :**
```go
conn, _ := net.DialUDP("udp", nil, serverAddr)
conn.Write(data)  // Envoi à serverAddr implicite
conn.Read(buffer) // Réception depuis serverAddr implicite
```

**Note :** UDP est toujours sans connexion au niveau protocole. `DialUDP()` associe juste une adresse distante par défaut au niveau API.

#### 3. Timeouts

```go
// Timeout pour Read
conn.SetReadDeadline(time.Now().Add(5 * time.Second))

// Timeout pour Write
conn.SetWriteDeadline(time.Now().Add(5 * time.Second))

// Timeout pour Read ET Write
conn.SetDeadline(time.Now().Add(5 * time.Second))
```

**Vérifier un timeout :**
```go
if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
    fmt.Println("Timeout!")
}
```

---

## 04. Channels - Communication entre goroutines

### Concept

Les **channels** permettent aux goroutines de communiquer et se synchroniser.

```go
// Créer un channel
ch := make(chan string)

// Envoyer dans le channel (bloquant)
ch <- "message"

// Recevoir du channel (bloquant)
msg := <-ch

// Channel avec buffer
ch := make(chan string, 10)  // Buffer de 10 messages
```

### Exemple : Serveur avec channel pour broadcast

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "sync"
)

type Client struct {
    conn   net.Conn
    send   chan string // Channel pour envoyer des messages à ce client
    pseudo string
}

var (
    clients      = make(map[*Client]bool)
    clientsMutex sync.Mutex
    broadcast    = make(chan string, 10) // Channel de broadcast
)

func main() {
    listener, err := net.Listen("tcp", ":9000")
    if err != nil {
        log.Fatal(err)
    }
    defer listener.Close()

    fmt.Println("[SERVEUR CHAT] Démarré sur :9000")

    // Goroutine pour gérer le broadcast
    go handleBroadcast()

    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Println(err)
            continue
        }

        go handleClient(conn)
    }
}

func handleBroadcast() {
    // Boucle infinie qui écoute le channel broadcast
    for msg := range broadcast {
        clientsMutex.Lock()
        // Envoyer à tous les clients
        for client := range clients {
            select {
            case client.send <- msg:
                // Message envoyé
            default:
                // Client bloqué, on le déconnecte
                close(client.send)
                delete(clients, client)
            }
        }
        clientsMutex.Unlock()
    }
}

func handleClient(conn net.Conn) {
    client := &Client{
        conn: conn,
        send: make(chan string, 10),
    }

    // Demander le pseudo
    conn.Write([]byte("Pseudo: "))
    scanner := bufio.NewScanner(conn)
    if scanner.Scan() {
        client.pseudo = scanner.Text()
    } else {
        client.pseudo = "Anonyme"
    }

    // Ajouter le client
    clientsMutex.Lock()
    clients[client] = true
    clientsMutex.Unlock()

    // Annoncer l'arrivée
    broadcast <- fmt.Sprintf("*** %s a rejoint le chat ***\n", client.pseudo)

    // Goroutine pour envoyer les messages au client
    go func() {
        for msg := range client.send {
            _, err := conn.Write([]byte(msg))
            if err != nil {
                break
            }
        }
    }()

    // Lire les messages du client
    for scanner.Scan() {
        msg := scanner.Text()
        if msg != "" {
            broadcast <- fmt.Sprintf("%s: %s\n", client.pseudo, msg)
        }
    }

    // Nettoyage
    clientsMutex.Lock()
    delete(clients, client)
    clientsMutex.Unlock()
    close(client.send)
    conn.Close()

    broadcast <- fmt.Sprintf("*** %s a quitté le chat ***\n", client.pseudo)
}
```

### Explications Channels

#### 1. Opérations de base

```go
// Créer
ch := make(chan int)

// Envoyer (bloquant jusqu'à ce qu'un récepteur soit prêt)
ch <- 42

// Recevoir (bloquant jusqu'à ce qu'une valeur soit disponible)
value := <-ch

// Fermer (émetteur uniquement)
close(ch)

// Vérifier si fermé
value, ok := <-ch
if !ok {
    // Channel fermé
}
```

#### 2. Channel buffered vs unbuffered

**Unbuffered :**
```go
ch := make(chan int)  // Capacité 0
ch <- 1               // BLOQUE jusqu'à réception
```

**Buffered :**
```go
ch := make(chan int, 3)  // Capacité 3
ch <- 1  // OK
ch <- 2  // OK
ch <- 3  // OK
ch <- 4  // BLOQUE (buffer plein)
```

#### 3. Select statement

```go
select {
case msg := <-ch1:
    fmt.Println("Reçu de ch1:", msg)
case msg := <-ch2:
    fmt.Println("Reçu de ch2:", msg)
case ch3 <- value:
    fmt.Println("Envoyé sur ch3")
default:
    fmt.Println("Aucun channel prêt")
}
```

**Comportement :**
- Attend qu'un des cas soit prêt
- Si plusieurs sont prêts, en choisit un aléatoirement
- `default` exécuté si aucun cas n'est prêt

#### 4. Pattern : Range sur channel

```go
ch := make(chan int, 5)

// Producteur
go func() {
    for i := 0; i < 10; i++ {
        ch <- i
    }
    close(ch)  // Important !
}()

// Consommateur
for value := range ch {
    fmt.Println(value)
}
// Boucle se termine quand ch est fermé
```

---

## 05. Patterns avancés

### 1. Worker Pool

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()

    for job := range jobs {
        fmt.Printf("Worker %d traite job %d\n", id, job)
        time.Sleep(time.Second) // Simule du travail
        results <- job * 2
    }
}

func main() {
    const numWorkers = 3
    const numJobs = 10

    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)

    var wg sync.WaitGroup

    // Démarrer les workers
    for w := 1; w <= numWorkers; w++ {
        wg.Add(1)
        go worker(w, jobs, results, &wg)
    }

    // Envoyer les jobs
    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs)

    // Attendre la fin
    go func() {
        wg.Wait()
        close(results)
    }()

    // Collecter les résultats
    for result := range results {
        fmt.Println("Résultat:", result)
    }
}
```

### 2. Timeout pattern

```go
func doWork() error {
    result := make(chan error, 1)

    go func() {
        // Travail long
        time.Sleep(2 * time.Second)
        result <- nil
    }()

    select {
    case err := <-result:
        return err
    case <-time.After(1 * time.Second):
        return fmt.Errorf("timeout")
    }
}
```

### 3. Context pour annulation

```go
import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d arrêté: %v\n", id, ctx.Err())
            return
        default:
            fmt.Printf("Worker %d travaille\n", id)
            time.Sleep(time.Second)
        }
    }
}

func main() {
    // Contexte avec timeout
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    for i := 1; i <= 3; i++ {
        go worker(ctx, i)
    }

    time.Sleep(10 * time.Second)
}
```

---

## 06. WebSocket en Go

### Installation

```bash
go get github.com/gorilla/websocket
```

### websocket/server.go

```go
package main

import (
    "fmt"
    "log"
    "net/http"

    "github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool {
        return true // Accepter toutes les origines (dev uniquement!)
    },
}

func handleWebSocket(w http.ResponseWriter, r *http.Request) {
    // Upgrade HTTP → WebSocket
    conn, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Println("Upgrade error:", err)
        return
    }
    defer conn.Close()

    fmt.Printf("[WS] Client connecté: %s\n", conn.RemoteAddr())

    for {
        // Lire un message
        messageType, message, err := conn.ReadMessage()
        if err != nil {
            log.Println("Read error:", err)
            break
        }

        fmt.Printf("[WS] Reçu: %s\n", message)

        // Echo : renvoyer le message
        err = conn.WriteMessage(messageType, message)
        if err != nil {
            log.Println("Write error:", err)
            break
        }
    }

    fmt.Printf("[WS] Client déconnecté: %s\n", conn.RemoteAddr())
}

func main() {
    http.HandleFunc("/ws", handleWebSocket)

    fmt.Println("[WS SERVER] Démarré sur :8080/ws")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Test avec HTML

```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Test</title>
</head>
<body>
    <h1>WebSocket Echo Client</h1>
    <input type="text" id="message" placeholder="Message">
    <button onclick="send()">Envoyer</button>
    <div id="output"></div>

    <script>
        const ws = new WebSocket('ws://localhost:8080/ws');

        ws.onopen = () => {
            console.log('Connecté');
            document.getElementById('output').innerHTML += '<p>Connecté!</p>';
        };

        ws.onmessage = (event) => {
            console.log('Reçu:', event.data);
            document.getElementById('output').innerHTML +=
                `<p>Echo: ${event.data}</p>`;
        };

        ws.onerror = (error) => {
            console.error('Erreur:', error);
        };

        ws.onclose = () => {
            console.log('Déconnecté');
        };

        function send() {
            const input = document.getElementById('message');
            ws.send(input.value);
            input.value = '';
        }
    </script>
</body>
</html>
```

---

## Bonnes pratiques Go

### 1. Gestion d'erreur

```go
// ✅ BON
if err != nil {
    return fmt.Errorf("failed to connect: %w", err)
}

// ❌ MAUVAIS
if err != nil {
    panic(err)  // Réservé aux erreurs irrécupérables
}
```

### 2. Fermeture de ressources

```go
// ✅ BON
conn, err := net.Dial("tcp", "example.com:80")
if err != nil {
    return err
}
defer conn.Close()

// ❌ MAUVAIS
conn, _ := net.Dial("tcp", "example.com:80")
// ... oubli de fermer
```

### 3. Context pour timeout

```go
// ✅ BON
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

conn, err := net.DialContext(ctx, "tcp", "example.com:80")
```

### 4. Gestion de concurrence

```go
// ✅ BON
var mu sync.Mutex
mu.Lock()
sharedData++
mu.Unlock()

// Ou mieux : atomic
atomic.AddInt64(&counter, 1)
```

---

## Conclusion

Vous avez maintenant une base solide pour la programmation réseau en Go :

- ✅ **TCP simple** : Client/serveur de base
- ✅ **Goroutines** : Concurrence légère et efficace
- ✅ **UDP** : Communication sans connexion
- ✅ **Channels** : Communication entre goroutines
- ✅ **Patterns** : Worker pools, timeouts, context
- ✅ **WebSocket** : Communication temps réel

**Forces de Go pour le réseau :**
- Performance exceptionnelle
- Concurrence native (goroutines)
- Standard library complète
- Code simple et maintenable

**Ressources supplémentaires :**
- Documentation officielle : https://pkg.go.dev/net
- Go by Example : https://gobyexample.com/
- Effective Go : https://go.dev/doc/effective_go

---


⏭️ [E.3 Exemples JavaScript](/annexes/05-exemples-code/javascript/README.md)
