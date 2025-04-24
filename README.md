# National E-Voting System Design Document

## System Overview and Scope

The National E-Voting System is designed to enable secure, private, and verifiable electronic voting for citizens via web browsers and public kiosks. The goal is to replace or complement traditional paper-based voting with a digital alternative that maintains voter anonymity, prevents fraud, and supports large-scale participation.

### Key Features:
- Secure voter authentication
- Blind token issuance for vote anonymity
- Vote casting and submission
- Immutable vote storage using blockchain
- Real-time monitoring and auditing
- Public auditability via IPFS

### Scope:
**Included:**
- Frontend for citizens and administrators
- Authentication and token issuance services
- Vote submission and validation
- Blockchain-backed vote storage
- Public audit publishing
- Monitoring and metrics

**Excluded:**
- Biometric integration
- Offline voting mechanisms
- Internationalization/localization

---

## Functional and Non-Functional Requirements

### Functional Requirements:
- Citizens can authenticate and cast votes
- Tokens must be issued to enable anonymous voting
- System must verify and store encrypted votes
- Administrators must monitor election metrics
- Public can verify integrity of election via published audits

### Non-Functional Requirements:
- **Scalability:** Support up to 10 million users with concurrent vote casting.
- **Latency:** Vote submission must complete within 1 second.
- **Uptime:** System must maintain 99.99% availability during elections.
- **Security:** Must use TLS, encryption-at-rest, and hashing for sensitive data.

---

## High-Level Architecture

The system is composed of frontend applications, backend services, messaging infrastructure, and secure data stores.

```mermaid
graph LR
    A[User Device PC, Mobile, Kiosk] -->|HTTPS| B[API Gateway]
    B -->|Route Requests| C[Authentication Service]
    B -->|Route Requests| D[Voting Service]
    B -->|Route Requests| E[Audit Service]
    B -->|Route Requests| F[Notification Service]
    B -->|Route Requests| G[Analytics Service]
    
    C --> H[User Database PostgreSQL]
    D --> I[Voting Database Blockchain-based]
    E --> J[Audit Logs Storage NoSQL]
    F --> K[Push Notification Service Redis]
    G --> L[Analytics Data Elasticsearch]
    
    subgraph Data Infrastructure
        M[Cache Redis]
        N[Data Pipelines Kafka]
        O[Data Warehouse BigQuery]
    end

    M --> H
    M --> I
    N --> O
    O --> L

    B --> M[Load Balancer]
    M --> N[API Gateway]

```

```mermaid



```


```mermaid
graph TD
    A[User Requests via API Gateway] --> B[Load Balancer]
    B --> C[Authentication Service]
    B --> D[Voting Service]
    B --> E[Audit Service]
    B --> F[Notification Service]
    B --> G[Analytics Service]

    subgraph Databases
        CDB[User DB PostgreSQL]
        VDB[Voting DB Blockchain-based]
        ADB[Audit Logs DB NoSQL]
    end

    C --> CDB
    D --> VDB
    E --> ADB
    
    subgraph Caching and Pipelines
        Cache[Cache Layer Redis]
        Kafka[Data Pipelines Kafka]
    end
    
    Cache --> CDB
    Cache --> VDB
    Kafka --> O[Data Warehouse BigQuery]
    O --> L[Analytics Elasticsearch]

```

```mermaid
C4Context
Person(citizen, "Citizen", "Eligible voter using mobile, PC, or public kiosk to vote")
Person(governmentAdmin, "Government Admin", "Manages and monitors election process")
System(eVotingSystem, "National e-Voting System", "A secure, distributed voting platform for national elections")

System_Ext(nationalIDSystem, "National ID System", "Verifies identities of citizens via national registry")
System_Ext(notificationGateway, "Notification Gateway", "Delivers email/SMS or push messages")
System_Ext(blockchainAuditor, "Public Blockchain Verifier", "External read access for public audit and verification")

Rel(citizen, eVotingSystem, "Casts vote, receives updates", "Web / Mobile / Kiosk Interface")
Rel(governmentAdmin, eVotingSystem, "Monitors votes, generates reports", "Admin Dashboard")

Rel(eVotingSystem, nationalIDSystem, "Verifies voter identity", "REST API")
Rel(eVotingSystem, notificationGateway, "Sends voting confirmations and alerts", "Webhooks / Push / Email")
Rel(eVotingSystem, blockchainAuditor, "Publishes anonymized vote data", "Blockchain API")

```


```mermaid
C4Container
System_Boundary(eVotingSystem, "National e-Voting System") {
    Container(webApp, "Web Application", "Vue.js / Nuxt", "Interface for voters on desktop, mobile, and kiosks")
    Container(authService, "Authentication Service", "Keycloak / OAuth 2.0", "Manages voter identity verification and secure login")
    Container(voteService, "Vote Recording Service", "Go / Rust", "Handles secure vote submission and blockchain writing")
    Container(auditService, "Audit Log Service", "Node.js", "Stores immutable system logs and actions")
    Container(notificationService, "Notification Service", "Firebase / WebSockets", "Sends updates to users")
    Container(analyticsService, "Analytics Service", "Python / Spark", "Processes and analyzes vote data patterns")
    ContainerDb(userDb, "User Database", "PostgreSQL", "Stores citizen identities and login metadata")
    ContainerDb(voteDb, "Vote Ledger", "Private Blockchain", "Immutable, anonymized record of all votes")
    ContainerDb(auditDb, "Audit Logs DB", "MongoDB / Cassandra", "Stores audit logs for transparency")
    ContainerDb(cacheLayer, "Cache Layer", "Redis", "Caches frequent queries and sessions")
    Container(queue, "Message Queue", "Apache Kafka", "Asynchronous communication between services")
    Container(gateway, "API Gateway", "NGINX / Kong", "Entry point for all API requests")
    Container(loadBalancer, "Load Balancer", "HAProxy / AWS ELB", "Distributes load across services")
}

Person(citizen, "Citizen", "Eligible voter using mobile, PC, or public kiosk")
Person(governmentAdmin, "Government Admin", "Manages election process and oversight")

Rel(citizen, gateway, "Uses", "HTTPS")
Rel(gateway, webApp, "Delivers frontend")
Rel(gateway, authService, "Handles auth requests")
Rel(gateway, voteService, "Submits vote securely")
Rel(gateway, auditService, "Logs activity")
Rel(gateway, notificationService, "Receives real-time updates")
Rel(gateway, analyticsService, "Fetches statistical insights")

Rel(authService, userDb, "Reads & verifies user data", "SQL")
Rel(voteService, voteDb, "Writes vote to ledger", "Blockchain API")
Rel(auditService, auditDb, "Writes logs", "NoSQL")
Rel(notificationService, cacheLayer, "Caches messages", "Redis protocol")
Rel(analyticsService, queue, "Streams events", "Kafka protocol")
Rel(queue, analyticsService, "Feeds back analytics tasks")
Rel(analyticsService, voteDb, "Reads vote patterns", "Blockchain Read API")
Rel(analyticsService, userDb, "Fetches anonymized data", "SQL")
Rel(governmentAdmin, analyticsService, "Views election insights", "HTTP")
```

---

## Component Breakdown

### Web App (Vue.js / React)
- Renders voting interface
- Sends authentication and vote requests

### Authentication Service (Node.js / Python)
- Verifies identity with citizen database
- Issues JWT and interacts with Redis cache

### Blind Token Service (Go)
- Provides blind signature tokens for vote anonymity

### Vote Collector (Node.js)
- Accepts encrypted votes
- Validates blind tokens
- Forwards valid votes to Kafka

### Blockchain Ledger (Rust)
- Stores vote hashes immutably
- Maintains audit trail

### Audit Publisher (Node.js)
- Publishes Merkle root hashes to IPFS

### Monitoring System
- Prometheus for metrics
- Grafana dashboards
- ELK stack for logging and auditing

---

## Data Storage and Database Schema

### PostgreSQL (Relational)
- Users Table (user_id, name, birthdate, region, status)
- Auth Sessions Table (session_id, user_id, issued_at, expires_at)

### Redis (In-Memory)
- Session tokens and ephemeral state

### Blockchain (Immutable Log)
- Vote Event (vote_id, hash, timestamp)

### IPFS (Audit Files)
- Published audit trail (Merkle tree JSON, metadata)

### Indexing & Performance
- PostgreSQL indexed on user_id and region
- Redis used for fast token validation
- Kafka partitions for scalable message ingestion

---

## Technology Stack Choices and Reasoning

| Layer | Technology | Reasoning |
|------|------------|-----------|
| Frontend | Vue.js / React | Component-based, fast development, responsive UI |
| Backend | Node.js, Go, Python | Async-friendly, fast prototyping, secure services |
| DB | PostgreSQL, Redis | Strong consistency, performance caching |
| Queue | Kafka | Scalable and fault-tolerant event streaming |
| Ledger | Custom Blockchain (Rust) | High performance and memory safety |
| Storage | IPFS | Decentralized and verifiable document storage |
| Security | Vault, HTTPS, JWT | Secrets management, encryption, token-based auth |
| Monitoring | Prometheus, Grafana, ELK | Observability and alerting stack |

---

## Non-Functional Design Discussions

### Scalability
- Stateless service instances behind load balancers
- Kafka enables asynchronous vote ingestion and decoupling
- PostgreSQL read replicas and Redis clustering
- IPFS nodes distributed across regions

### Security
- HTTPS for all communications
- OAuth2 and JWT-based sessions
- Blind signatures preserve anonymity
- Input sanitization and OWASP security checks
- Secrets stored in HashiCorp Vault

### Reliability
- Retry logic via Kafka and consumer groups
- Multiple replicas for DB and blockchain nodes
- Monitoring with alerts and auto-scaling triggers
- Audit logs stored immutably on IPFS

---

## Trade-offs and Limitations

### Monolith vs Microservices
- Chose microservices for separation of concerns, easier scaling, but higher deployment complexity

### SQL vs NoSQL
- Chose PostgreSQL for structured data and strong consistency. Could limit flexibility in unstructured expansions

### Blind Tokens vs Zero-Knowledge Proofs
- Simpler implementation with blind signatures but less robust than ZKPs

### Limitations
- No offline voting support in current scope
- Relies on network availability for all operations
- Assumes identity data from government is always accurate

---