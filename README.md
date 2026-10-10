# UPI Offline Mesh

A Spring Boot-based simulation of **offline UPI-style payments using a device-to-device mesh network**. The project demonstrates how encrypted payment instructions can travel across devices without internet connectivity and reach a backend for deferred settlement when a bridge device reconnects.

## Overview

In areas with no internet connectivity, payment packets can be relayed between nearby devices until a device with internet access forwards them to the backend.

This project simulates the complete process on a single computer, without requiring physical Bluetooth hardware or a real banking connection.

### Key Features

- **Offline Payment Simulation:** Simulates payment creation and transmission through virtual devices.
- **Hybrid Encryption:** Uses RSA-OAEP and AES-256-GCM to protect payment payloads.
- **Mesh Network Simulation:** Demonstrates packet propagation through multiple virtual devices using gossip rounds and TTL.
- **Idempotent Processing:** Uses SHA-256 ciphertext hashing and atomic duplicate detection to prevent repeated settlement of the same packet.
- **Replay Protection:** Validates packet freshness and uses unique nonces.
- **Transactional Settlement:** Debits the sender, credits the receiver, and records transactions through Spring Data JPA.
- **Interactive Dashboard:** Visualizes device states, account balances, transaction history, and packet movement.
- **Concurrency Testing:** Tests simultaneous delivery of duplicate packets and verifies that a payment settles only once.

## Technology Stack

| Technology | Purpose |
|---|---|
| Java 17+ | Backend programming language |
| Spring Boot 3.3 | Application framework |
| Spring Data JPA | Database access and persistence |
| H2 Database | In-memory database for the demo |
| Maven Wrapper | Build and dependency management |
| RSA-OAEP | AES key encryption |
| AES-256-GCM | Payload confidentiality and integrity |
| SHA-256 | Ciphertext hashing and duplicate detection |
| JUnit | Automated testing |

## Architecture

The application follows a simulated offline-to-online payment flow:

1. **Payment creation:** A payment instruction is generated and encrypted.
2. **Packet creation:** The encrypted payload is wrapped in a mesh packet with a unique identifier and TTL.
3. **Mesh propagation:** Virtual devices exchange packets through simulated gossip rounds.
4. **Bridge upload:** A simulated internet-enabled device forwards packets to the backend.
5. **Duplicate detection:** The backend hashes the ciphertext and checks whether it has already been processed.
6. **Validation:** The backend decrypts the payload and checks packet freshness.
7. **Settlement:** The backend updates account balances and records the transaction.

```text
Sender
  |
  v
Encrypted Payment Packet
  |
  v
Simulated Device Mesh
  |
  v
Internet-Enabled Bridge
  |
  v
Spring Boot Backend
  |
  +--> Duplicate Detection
  +--> Decryption and Validation
  +--> Transaction Settlement
  |
  v
Updated Balances and Transaction Ledger
```

## Getting Started

### Prerequisites

- JDK 17 or newer
- Git, if cloning the repository
- Internet access for the initial Maven dependency download

No separate Maven installation or database server is required.

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

Replace the repository URL with your actual GitHub repository URL.

Alternatively, download the repository as a ZIP file and extract it.

### 2. Run the application on Windows

Open a terminal in the project root and execute:

```bash
mvnw.cmd spring-boot:run
```

If you are using PowerShell, run:

```powershell
.\mvnw.cmd spring-boot:run
```

### 3. Run the application on macOS or Linux

```bash
chmod +x mvnw
./mvnw spring-boot:run
```

### 4. Open the dashboard

After the application starts successfully, visit:

**http://localhost:8080**

Use the dashboard to simulate payments, run mesh gossip rounds, upload packets through bridge devices, and inspect account balances and transactions.

## Running the Demo

Follow these steps to demonstrate the complete flow:

1. **Inject a payment:** Choose the sender, receiver, amount, and other available inputs, then inject the payment into the mesh.
2. **Run gossip rounds:** Propagate the packet across virtual devices.
3. **Upload through bridges:** Simulate an internet-enabled bridge forwarding its packets to the backend.
4. **Inspect settlement:** Verify the updated account balances and transaction ledger.
5. **Test duplicate protection:** Run the concurrency test to verify that simultaneous deliveries of the same packet do not cause multiple settlements.

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Serves the dashboard |
| GET | `/api/server-key` | Returns the server's RSA public key |
| GET | `/api/accounts` | Retrieves account balances |
| GET | `/api/transactions` | Retrieves recent transactions |
| GET | `/api/mesh/state` | Retrieves virtual device states |
| POST | `/api/demo/send` | Creates and injects a simulated payment |
| POST | `/api/mesh/gossip` | Runs a mesh gossip round |
| POST | `/api/mesh/flush` | Uploads packets through bridge devices |
| POST | `/api/mesh/reset` | Resets the simulated mesh and idempotency cache |
| POST | `/api/bridge/ingest` | Accepts packets for backend processing |
| GET | `/h2-console` | Opens the H2 database console |

### Bridge Ingestion

The `/api/bridge/ingest` endpoint accepts an encrypted mesh packet and processes it through duplicate detection, decryption, freshness validation, and settlement.

Possible outcomes include:

- `SETTLED`: The transaction was settled successfully.
- `DUPLICATE_DROPPED`: The packet was identified as a duplicate.
- `INVALID`: The packet failed validation or processing.

## Security and Reliability

### Hybrid Encryption

- AES-256-GCM encrypts the payment payload and authenticates its integrity.
- RSA-OAEP encrypts the AES key.
- Intermediate devices carry ciphertext rather than the readable payment payload.

### Duplicate Prevention

The backend calculates a SHA-256 hash of the ciphertext and uses an atomic operation to identify duplicate deliveries before processing them. A unique database constraint provides an additional safeguard.

### Replay Protection

The backend validates the timestamp inside the encrypted payment instruction and uses a unique nonce to distinguish individual payment instructions.

### Transaction Management

Spring's transactional support coordinates account balance changes and ledger insertion. Optimistic locking on account entities provides additional protection against concurrent updates.

## Running Tests

Run the complete test suite on Windows:

```bash
mvnw.cmd test
```

On PowerShell:

```powershell
.\mvnw.cmd test
```

Run tests on macOS or Linux:

```bash
./mvnw test
```

Important test cases include:

- Encryption and decryption round-trip.
- Rejection of tampered ciphertext.
- Concurrent delivery of one packet through three simulated bridges, verifying exactly one settlement.

## Limitations and Future Improvements

This project is an educational prototype, not a production-ready payment system.

Current limitations include:

- The mesh network is simulated. Real Bluetooth Low Energy or Wi-Fi Direct communication is not implemented.
- H2 is an in-memory database, and duplicate detection uses an in-memory `ConcurrentHashMap`.
- The RSA key pair is generated at application startup.
- Real bank integration, NPCI integration, and actual UPI settlement are not implemented.
- Production use would require robust authentication, authorization, key management, persistent idempotency, monitoring, and regulatory compliance.
- Offline payments introduce double-spending and insufficient-funds risks that cannot be fully resolved by delayed backend settlement alone.

Potential future improvements include real device-to-device communication, Redis-based distributed idempotency, persistent database storage, stronger bridge authentication, and integration with a properly authorized payment infrastructure.

## Disclaimer

This project demonstrates **mesh-routed deferred settlement** for learning and experimentation. It does not initiate real UPI transactions, connect to banks, or transfer actual money. The cryptographic and settlement mechanisms are implemented as part of a simulation and should not be treated as a certified payment solution.

## License

No license has been specified. All rights and usage permissions remain with the respective copyright holders unless a license is added to the repository.
