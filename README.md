# Finance CBDC Ledger Prototype

![Finance CBDC Ledger Prototype Banner](https://image.pollinations.ai/prompt/futuristic%20central%20bank%20digital%20currency%20ledger%20banner%2C%20glowing%20blue%20blockchain%20nodes%2C%20financial%20technology%20circuitry%2C%20dark%20navy%20background%2C%20professional%20fintech%20aesthetic%2C%20wide%20cinematic%20composition?width=1600&height=500&nologo=true)

> A minimal Go-based prototype service intended as a starting point for a Central Bank Digital Currency (CBDC) ledger system. Currently, the repository contains a single HTTP health-check endpoint and is **not** a functional ledger — see the [Workability Assessment](#️-workability-assessment) section for an honest evaluation.

---

## 📋 Overview

| Attribute | Detail |
|---|---|
| **Language** | Go 1.20 |
| **Module** | `github.com/Shivay00001/finance-cbdc-ledger-prototype` |
| **Dependencies** | None (standard library only) |
| **Entrypoint** | `main.go` |
| **Default Port** | `8080` |
| **Containerization** | Docker (single-stage Alpine build) |
| **License** | VisionQuantech Custom Commercial License (see [LICENSE](LICENSE)) |

![Digital Ledger Visualization](https://image.pollinations.ai/prompt=abstract%20digital%20ledger%20double%20entry%20bookkeeping%20visualization%2C%20glowing%20transaction%20flows%20between%20accounts%2C%20teal%20and%20gold%20accents%2C%20minimalist%20fintech%20illustration?width=1200&height=400&nologo=true)

---

## 🏗️ Architecture & How It Works

The current codebase is intentionally minimal. The entire application lives in `main.go` and works as follows:

1. **Startup** — The `main()` function registers a single HTTP handler on the root path (`/`) using Go's `net/http` default serve mux.
2. **Request Handling** — Any HTTP request to `/` receives a plain-text response:
   ```
   System Operational: 2026-01-01 12:00:00 ...
   ```
   The payload is the current server timestamp, functioning as a liveness/health probe.
3. **Server Lifecycle** — `http.ListenAndServe(":8080", nil)` starts the server on port **8080**. Startup and fatal errors are logged via the standard `log` package. If the server fails to bind, the process exits with a fatal log message.

### Component Diagram

```mermaid
flowchart LR
    Client([🌐 Client / curl]) -->|"GET /"| Mux["net/http<br/>DefaultServeMux"]
    Mux --> Handler["Root Handler<br/>fmt.Fprintf"]
    Handler --> Time["time.Now()"]
    Handler --> Resp["Plain-text response:<br/>'System Operational: <timestamp>'"]
    Resp --> Client
    Main["main()"] -->|"registers /"| Mux
    Main -->|"ListenAndServe :8080"| Server["HTTP Server :8080"]
    Server --> Mux
    Server -.->|"fatal on bind failure"| Log["log.Fatal"]
```

### Request Lifecycle

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Go HTTP Server (:8080)
    participant H as Root Handler
    C->>S: GET /
    S->>H: Dispatch via DefaultServeMux
    H->>H: time.Now()
    H-->>C: 200 OK — "System Operational: <timestamp>"
```

### Container Topology

```mermaid
flowchart TB
    subgraph Docker["🐳 Docker Container (golang:1.20-alpine)"]
        App["./app binary<br/>(built from main.go)"] --> Port["Exposes :8080"]
    end
    Host["Host Machine<br/>localhost:8080"] -->|"port mapping -p 8080:8080"| Port
```

There are currently **no** ledger data structures, transaction models, account balances, double-entry bookkeeping logic, consensus mechanisms, cryptographic signing, or persistence layers implemented. The repository name describes the *intended* direction of the project.

---

## 🚀 Running Locally (Without Docker)

Requires Go 1.20+:

```bash
git clone https://github.com/Shivay00001/finance-cbdc-ledger-prototype.git
cd finance-cbdc-ledger-prototype
go build -o app
./app
```

Then verify:

```bash
curl http://localhost:8080/
# → System Operational: <timestamp>
```

---

## 🐳 Running with Docker

The included `Dockerfile` uses `golang:1.20-alpine`, copies the source, builds the binary, and runs it.

### Build and run

```bash
docker build -t finance-cbdc-ledger-prototype .
docker run -d -p 8080:8080 --name cbdc-ledger finance-cbdc-ledger-prototype
```

### Verify

```bash
curl http://localhost:8080/
```

### Optional: Docker Compose

No `docker-compose.yml` is shipped, but this minimal one works:

```yaml
version: "3.8"
services:
  ledger:
    build: .
    ports:
      - "8080:8080"
```

```bash
docker-compose up -d --build
```

---

## ⚖️ Workability Assessment

An honest evaluation of the current state:

**What works:**
- ✅ The code compiles cleanly with Go 1.20 and has zero external dependencies.
- ✅ The Dockerfile builds and the container runs the service correctly on port 8080.
- ✅ The HTTP health endpoint responds as expected.
- ✅ Sensible `.gitignore` hygiene for secrets and build artifacts.

**What is missing / limitations:**
- ❌ **No ledger functionality exists.** Despite the repository name, there are no accounts, transactions, balances, double-entry logic, or CBDC-specific features (issuance, redemption, wallets, cryptographic signing).
- ❌ **No tests** (`*_test.go` files are absent).
- ❌ **No persistence** — nothing is stored; all state is ephemeral (there is no state at all).
- ❌ **No configuration management** — the port is hardcoded; no environment-variable support.
- ❌ **No graceful shutdown** — the server does not handle `SIGTERM`/`SIGINT` for clean container stops.
- ❌ **No API surface** — a single unversioned root route returning plain text; no JSON, no structured errors.
- ❌ **Dockerfile could be hardened** — no multi-stage build (ships the full Go toolchain in the final image), runs as root, no `HEALTHCHECK` instruction.
- ❌ **No CI/CD, linting, or documentation of the intended ledger design.**

**Verdict:** This repository is a **scaffold / hello-world skeleton**, not a production-ready or even functionally complete CBDC ledger prototype. It is a valid starting point that compiles and deploys, but it requires **substantial implementation work** — domain models, transaction validation, persistence, security, and testing — before it delivers on its stated purpose.

### Target Architecture (Aspirational)

```mermaid
flowchart TB
    subgraph Future["🔮 Intended Future State"]
        API["REST API Layer<br/>/transactions /accounts /balances"] --> Core["Ledger Core<br/>Double-Entry Engine"]
        Core --> DB[("PostgreSQL<br/>Persistent Store")]
        Core --> Crypto["Signing & Audit<br/>Module"]
        Wallet["Wallet Service<br/>Issuance / Redemption"] --> Core
    end
    Current["Current State:<br/>Health-check only"] -.->|"roadmap"| Future
```

---

## 🗺️ Suggested Roadmap

1. Define core domain types: `Account`, `Transaction`, `Ledger` with double-entry invariants.
2. Add REST API endpoints (`POST /transactions`, `GET /accounts/{id}/balance`) with JSON responses.
3. Introduce persistence (e.g., PostgreSQL or an embedded store) with atomic balance updates.
4. Add cryptographic signing of transactions and audit logging.
5. Implement graceful shutdown, config via environment variables, and structured logging.
6. Add unit/integration tests and CI.
7. Harden the Dockerfile (multi-stage build, non-root user).

---

## 📄 License

This project is distributed under the **VisionQuantech Custom Commercial License**:

- **Free** for personal, educational, non-financial use.
- **Revenue share (15–30%)** required for individual earning use.
- **Commercial/enterprise use prohibited** without a separate license — contact **visionquantech@proton.me**.

See [LICENSE](LICENSE) for full terms. The software is provided **"AS IS"**, without warranty of any kind.