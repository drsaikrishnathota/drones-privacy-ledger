# Blockchain-Secured Drone Data Privacy System (Java/Spring Boot)

A locally-runnable reference implementation that **secures drone flight logs and operator IDs** using a **hash‑chained ledger** and **ECDSA signature verification**. No cloud required. Inspired by patterns in
the reference app you shared, but simplified for **single-node local development**.

> **Why this approach?** It gives you an **auditable, tamper‑evident ledger** (blockchain‑like) while keeping setup super light. You can later swap the mock ledger for **Hyperledger Fabric** (adapter stub included in design) without changing your API.

---

## ✨ Features

- Operator registration with PEM public keys (ECDSA)
- Signed flight-log submission with on‑chain (hash‑chain) anchoring
- Integrity verification endpoint for the full chain
- H2 file‑based DB (no external dependencies)
- Clean REST API with curl examples & Postman collection
- Upgrade path to true Fabric network (via adapter interface — TODO)

---

**Data flow**

1. Operator registers `operatorId` + `publicKeyPem`.
2. Operator submits a flight log: server verifies **ECDSA signature** over `(droneId|timestamp|logData)`.
3. Each accepted log is anchored in a **new block** referencing the previous block’s hash.
4. `/api/ledger/verify` recomputes hashes to prove integrity.

---

## 🚀 Getting Started

### Prereqs
- **JDK 17+**
- **Maven 3.9+**

### Build & Run
```bash
mvn -q clean package
java -jar target/drone-privacy-ledger-0.1.0.jar
# App on http://localhost:8080
```

H2 console: <http://localhost:8080/h2-console> (JDBC URL: `jdbc:h2:file:./data/ledgerdb`)

---

## 🔐 Quick Demo (curl)

### 1) Generate a keypair (Java example)
See **docs/keygen.java** for a minimal snippet (or use OpenSSL).

### 2) Register operator
```bash
PUB=$(cat docs/sample-public.pem | tr -d '\n')
curl -s -X POST http://localhost:8080/api/operators/register \
  -H "Content-Type: application/json" \
  -d '{"operatorId":"op1","publicKeyPem":"'$PUB'"}' | jq
```

### 3) Submit a signed flight log
Sign the message: `droneA|2025-11-04T12:00:00Z|{\"alt\":120,\"lat\":38.63,\"lon\":-90.20}`  
Put the Base64 signature into the request:

```bash
curl -s -X POST http://localhost:8080/api/flights \
  -H "Content-Type: application/json" \
  -d '{
  "operatorId":"op1",
  "droneId":"droneA",
  "timestamp":"2025-11-04T12:00:00Z",
  "logData":"{\"alt\":120,\"lat\":38.63,\"lon\":-90.20}",
  "signatureBase64":"<PUT_BASE64_SIGNATURE_HERE>"
}' | jq
```

### 4) Verify chain
```bash
curl -s http://localhost:8080/api/ledger/verify | jq
```

### 5) Inspect blocks & logs
```bash
curl -s http://localhost:8080/api/ledger/blocks | jq
curl -s http://localhost:8080/api/flights | jq
```

---

## 📁 Project Layout

```
drone-privacy-ledger/
├─ src/main/java/com/example/dronedata
│  ├─ controller/  (REST controllers)
│  ├─ model/       (JPA entities: Operator, FlightLog, LedgerBlock)
│  ├─ repo/        (Spring Data repositories)
│  ├─ service/     (Operator, Flight, Ledger services)
│  ├─ util/        (Crypto helpers: ECDSA, SHA-256)
│  └─ DronePrivacyLedgerApplication.java
├─ src/main/resources/
│  └─ application.yml
├─ src/test/java/com/example/dronedata/DronePrivacyLedgerApplicationTests.java
├─ docs/
│  ├─ architecture.md (Mermaid diagrams + notes)
│  ├─ keygen.java (EC keypair + signing demo)
│  ├─ sample-public.pem
│  └─ postman_collection.json
├─ scripts/
│  └─ run.sh
└─ README.md
```

---

## 📝 API Summary

- `POST /api/operators/register` — body: `operatorId`, `publicKeyPem`
- `POST /api/flights` — body: `operatorId`, `droneId`, `timestamp`, `logData`, `signatureBase64`
- `GET /api/flights` — list all flight logs
- `GET /api/ledger/blocks` — list blocks
- `GET /api/ledger/verify` — `{"verified":true}` if chain OK

---

## 🧪 Postman

Import `docs/postman_collection.json` for ready‑made requests.

---

## 🛣️ Roadmap

- [ ] Batch multiple logs per block (configurable interval)
- [ ] Replace mock chain with **Hyperledger Fabric** (adapter impl)
- [ ] Operator DID documents & PK rotation
- [ ] Merkle trees for per‑record proofs
- [ ] Simple web UI for visualize blocks/logs

---

## 📸 Screenshots (how to recreate)

1. H2 Console showing tables (`operators`, `flight_logs`, `ledger_blocks`).
2. `GET /api/ledger/blocks` JSON in your browser or Postman.
3. Mermaid diagram preview from `docs/architecture.md` (VS Code “Markdown Preview Mermaid Support”).

Commit these screenshots under `docs/screenshots/` in your repo.

---
