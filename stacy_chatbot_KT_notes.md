# Stacy Chatbot — Complete Knowledge Transfer Notes
> **System:** Standard Chartered Bank (SCB) — Singapore  
> **Purpose:** Architecture KT for new team members  
> **Format:** Self-contained reference — no prior knowledge assumed

---

## Table of Contents

1. [Introduction to Stacy Chatbot](#1-introduction-to-stacy-chatbot)
2. [Functional Requirements](#2-functional-requirements)
3. [Non-Functional Requirements](#3-non-functional-requirements)
4. [Core Components — Definitions & Usage](#4-core-components--definitions--usage-in-stacy)
5. [High Level Architecture](#5-high-level-architecture)
6. [Detailed Data Flow — Pre-Login & Post-Login](#6-detailed-data-flow--pre-login--post-login)
7. [Interview Questions & Answers](#7-interview-questions--answers)

---

## 1. Introduction to Stacy Chatbot

### 1.1 What is Stacy?

**Stacy** is an **Active AI Chatbot** built for **Standard Chartered Bank (SCB), Singapore**. It is a conversational AI assistant that allows SCB customers to get instant answers to banking queries — without needing to call a human agent.

It lives on the SCB public website and SC Mobile app, and users interact with it through a chat widget (with Stacy's avatar).

---

### 1.2 Why Was Stacy Built? — The Problem

SCB's Contact Centre (CCAC — Customer Care And Assistance Centre) was receiving an extremely high volume of inbound calls, most of which were simple, repetitive queries like:

- "What is the minimum balance for my account?"
- "How do I reset my PIN?"
- "What are the PAD (Payment Account Details) charges?"

These routine queries were **overwhelming call centre agents**, leaving them with little bandwidth to handle truly urgent situations like:

- Fraud alerts
- Scam reports
- Account blocks
- Urgent financial distress

**The core problem:**
> Too many low-priority FAQ calls → agents delayed on high-priority cases → poor customer experience for urgent cases.

---

### 1.3 What Does Stacy Solve?

| Problem | Stacy's Solution |
|---------|-----------------|
| High FAQ call volume | Instant self-service chatbot responses |
| Agents stuck on routine queries | Agents freed for scam/fraud/urgent cases |
| Long customer wait times | Available 24×7, zero wait |
| High contact centre cost | Deflect majority of calls to bot |

---

### 1.4 Key Capabilities

- **Conversational AI** — understands natural language, not just fixed commands
- **NLP-powered** — recognizes intent behind a message (e.g., "I lost my card" → intent: report lost card)
- **Data-driven FAQ** — answers from a curated knowledge base of banking FAQs
- **Supervised Machine Learning** — learns from past conversations, improves over time with minimal human intervention
- **Live Bank handoff** — can connect the user to a live human agent if the bot cannot resolve
- **Multi-language** — supports English and Chinese
- **Multi-platform** — works on SCB public website and SC Mobile app

---

### 1.5 Delivery Phases — MVP Breakdown

Stacy is delivered in 3 phases (MVPs) to manage risk and deliver incremental value:

#### MVP 1 — Pre-Login (Public Web)
> No login needed. Anyone visiting sc.com can chat.

- FAQ Bot on the SCB public website
- Integration with Live Bank (human handoff)
- Handles **unauthenticated** user flow only
- reCAPTCHA enabled to prevent bot abuse
- Secured by Axway API Gateway

**Status: Launched**

---

#### MVP 2 — Post-Login (SC Mobile + Authenticated)
> Personalized chatbot for logged-in customers.

- SC Mobile experience with authenticated flow
- PAD (Payment Account Details) support
- Live Bank **unified** chat journey (seamless handoff)
- Chat **history** stored and retrievable
- Integrated with Kong API Gateway (IAG)
- Personalized responses using customer data from ICM

**Status: In Progress**

---

#### MVP 3 — Chat API & Experience Layer
> A unified, scalable API backbone for all channels.

- Common API layer for Channel Experience (CE)
- Chat API Experience Layer
- Process API using Morpheus
- Active AI App (Morpheus + Trinity)
- Supports all channels from a single backbone

**Status: Planned**

---

### 1.6 Who Are the Users?

| User Type | Description |
|-----------|-------------|
| SCB Customers | Primary users — retail banking customers in Singapore |
| SCB Staff | Internal users — agents and back-office staff |

---

## 2. Functional Requirements

Functional requirements describe **what the system must do** — the features and behaviours the chatbot must support.

---

### FR-1: Access Without Login (Pre-Login)

- Users must be able to access the chatbot from the **SCB public website without logging in**
- This covers general banking FAQs, product information, and branch/contact details
- reCAPTCHA must be shown before the chat window opens to verify the user is human
- The chatbot must not expose any customer-specific or sensitive data in this mode

---

### FR-2: Post-Login Chatbot Access

- *(Not in scope for MVP1 — delivered in MVP2)*
- User is authenticated via **Akamai BPM Prelogin API**
- Session is established after successful login
- **Session timeout: 10 minutes** of inactivity
- Used to retrieve **personalized data** such as account info, transaction history
- Customer data fields: RelID, Name, CustomerID

---

### FR-3: Multi-Language Support

- The chatbot must respond in **English** and **Chinese**
- Both the chatbot UI and responses must be rendered in the selected language
- Must work on:
  - SC Mobile
  - SCB Public Web
  - NFS (legacy iBanking platform)
  - Picasso (AI global stack)

---

### FR-4: Conversation Management

- The chatbot must maintain **context across a conversation** (remember earlier messages in the same session)
- Use **Morpheus / Trinity AI engine** for NLP and intent processing
- Process user messages via the **Process API**
- Show a **typing indicator** while response is being generated
- Support conversation history for post-login users

---

### FR-5: Message Types Supported

| Message Type | Supported? |
|-------------|-----------|
| Plain text | Yes |
| Text with URL links | Yes |
| Emoji in messages | Yes |
| Formatted (bold/list) responses | Yes |
| File/image attachments | No (MVP1) |

---

### FR-6: Integration with Banking Services

#### Pre-Login + Post-Login:
- Support **RM (Relationship Manager) chat**
- Integration with **Live Bank** for human agent handoff
- Text chat via **Kontra interface (CSC)**

#### Post-Login Only:
- Integrate with **Custom APIs via Kong IAG**
- Retrieve **customer details from ICM** (Integrated Customer Master)
- Access account and transaction data for personalized responses

---

### FR-7: Live Bank Handoff

- If the chatbot cannot resolve a query, it must offer the user to connect to a **Live Bank agent**
- The handoff must be seamless — the conversation context should carry over
- Uses **CASAS server** for SSO token generation
- Chatbot passes the token to Live Bank; Live Bank validates with CASAS

---

### FR-8: User Interface Requirements

- Chat widget with **Stacy avatar** displayed on the page
- Chat window rendered as an **iframe overlay** (does not navigate away from the page)
- Each message shows a **timestamp**
- **User messages** appear on the **right side**; bot messages on the **left**
- Minimize and Close controls available
- Must support **all major browsers**
- Must be **responsive** — works on desktop and mobile

---

### FR-9: Admin & Configuration

- Admin server to **manage AI workspace**
- Manage **knowledge graph** (Neo4j)
- Configure workspace rules and search indexes
- Manage conversation flows and FAQ content
- Reload knowledge graph after updates or rollback

---

## 3. Non-Functional Requirements

Non-functional requirements describe **how well the system must perform** — quality attributes like speed, scale, security, and reliability.

---

### NFR-1: User Scale & Growth

| Year | Expected Users |
|------|---------------|
| 2024 | 1.0 Lakh / month (~100,000) |
| 2025 | 1.7 Lakh / month (~170,000) |
| 2026 | 2.6 Lakh / month (~260,000) |

The system must be designed to **handle 2.6x growth** in 3 years without requiring a full re-architecture.

---

### NFR-2: Performance

| Metric | Target |
|--------|--------|
| Response time | < 2 seconds end-to-end |
| API Rate Limit | 100 requests per minute |
| Payload size | Small (JSON, no large payloads) |
| HTTP Methods | GET and POST only |
| API Format | JSON |

---

### NFR-3: Availability — CAP Theorem Decision

The system follows the **CAP Theorem** principle:

> **CAP Theorem** states that a distributed system can only guarantee two of three properties simultaneously: **Consistency**, **Availability**, and **Partition Tolerance**.

For Stacy, the decision is:

**Availability > Consistency**

**Why?**  
This is a customer-facing banking chatbot. It is more acceptable to return a slightly stale cached FAQ answer (e.g., cached 20 minutes ago) than to make the chatbot unavailable entirely. A chatbot that is down defeats its entire purpose and drives customers back to the call centre.

Redis cache with a 30-minute TTL directly supports this architectural choice.

---

### NFR-4: Security Requirements

| Requirement | Detail |
|-------------|--------|
| DDoS / Bot protection | Akamai BPM |
| Human verification | reCAPTCHA on every pre-login session |
| API security | Axway API Gateway (pre-login), Kong IAG (post-login) |
| Authentication | 2FA / OTP for post-login |
| Session management | 10-minute inactivity timeout |
| Network security | DMZ1 and DMZ2 network zones |
| Terms & Conditions | User must accept T&C before using chatbot |

---

### NFR-5: Environments

The system is deployed across the following environment pipeline:

```
Local  →  SIT  →  UAT  →  PT  →  PAD  →  PROD
```

| Environment | Purpose |
|-------------|---------|
| Local | Developer's machine |
| SIT (System Integration Testing) | All services integrated and tested together |
| UAT (User Acceptance Testing) | Business team validates functionality |
| PT (Performance Testing) | Load and stress testing |
| PAD (Pre-production / Staging) | Mirror of PROD — final validation |
| PROD | Live production environment |

---

### NFR-6: High Availability & Disaster Recovery

- System must support **automatic failover** — if PROD goes down, DR environment takes over
- Redis Sentinel ensures Redis master auto-recovery
- ElasticSearch deployed with 1 master + 2 data nodes for redundancy
- Service startup must follow a specific order (see Section 6 for DR details)

---

## 4. Core Components — Definitions & Usage in Stacy

This section introduces every key technology and component used in Stacy. Each entry explains **what it is in plain language** and **exactly how it is used in this system**.

---

### 4.1 NLP — Natural Language Processing

**What is it?**  
NLP is a field of Artificial Intelligence that enables computers to understand, interpret, and generate human language. Instead of needing exact keywords, NLP understands the *meaning* behind what a person types.

**Example:**  
A user types: *"I think someone used my card without my permission"*  
NLP understands this as: **Intent = Report Fraud** — even though the user didn't use the word "fraud."

**How it is used in Stacy:**  
Every message a user sends goes through the NLP pipeline inside the **Morpheus + Trinity AI engine**. NLP extracts the user's intent and any important entities (like account numbers, dates) from the message, which determines what response to generate.

---

### 4.2 Intent Recognition

**What is it?**  
Intent recognition is a specific NLP task — it classifies a user's message into a predefined **intent category**.

**Example intents in a banking chatbot:**
- `check_balance` — "What's my account balance?"
- `report_lost_card` — "I lost my debit card"
- `faq_minimum_balance` — "What's the minimum balance?"
- `connect_live_agent` — "I want to speak to someone"

**How it is used in Stacy:**  
After NLP processes the message, the intent is identified. Morpheus then routes the request to the appropriate process — either fetching an FAQ answer, calling a banking API, or initiating a Live Bank handoff.

---

### 4.3 Morpheus — AI Admin & App Server

**What is it?**  
Morpheus is the **central brain and orchestrator** of the Stacy chatbot backend. It is both an admin server (for configuration) and an application server (for runtime processing).

**Plain language analogy:**  
Think of Morpheus as the **manager of a call centre** — it receives calls (messages), decides who handles them (routes to the right system), manages the agents (Trinity, APIs, DB), and keeps records.

**How it is used in Stacy:**
- Receives every user message from the chatbot UI
- Runs NLP and intent recognition
- Checks Redis cache first — if answer found, returns it immediately
- If not in cache, forwards to Trinity AI engine
- Manages user sessions and authentication
- Calls external banking APIs (ICM, CSC) for personalized data
- Manages the Admin UI for knowledge graph and workspace configuration
- Orchestrates the entire conversation end-to-end

---

### 4.4 Trinity AI Engine

**What is it?**  
Trinity is the dedicated **AI inference server** — it handles the "thinking" work. It is separate from Morpheus, which means configuration changes and admin work don't interfere with AI processing.

**Plain language analogy:**  
If Morpheus is the call centre manager, Trinity is the **specialist expert** — called in only when the manager can't answer from memory (cache miss).

**How it is used in Stacy:**
- Only invoked when Redis cache has no answer (cache miss)
- Performs deep NLP processing and response generation
- Uses the **Neo4j knowledge graph** to find the most relevant answer
- Queries **ElasticSearch** to fetch and rank response content
- Returns the processed response to Morpheus, which sends it to the user

---

### 4.5 Redis Cache

**What is it?**  
Redis is an **in-memory key-value data store**. Unlike a traditional database that reads from disk, Redis stores data directly in RAM — making it extremely fast (microsecond-level responses).

**Plain language analogy:**  
Redis is like a **Post-it note on your desk** — you write down frequently asked answers so you can answer instantly without going to the filing cabinet (database) every time.

**How it is used in Stacy:**
- Every incoming user message is checked against the Redis cache first
- **Cache TTL (Time-To-Live): 30 minutes** — answers expire after 30 mins
- If a matching cached answer is found (cache HIT) → return immediately (no AI processing needed)
- If no cached answer (cache MISS) → forward to Trinity AI
- Dramatically reduces load on the AI engine for repeated FAQ queries
- **Redis Sentinel** is also deployed — it monitors the Redis master node and automatically promotes a replica if the master fails (high availability)

**Cache Logic Flow:**
```
User message → Morpheus → Check Redis
                              ↓
                    Cache HIT?
                   YES → Return response immediately
                   NO  → Forward to Trinity AI → Generate response → Store in Redis → Return
```

---

### 4.6 ElasticSearch

**What is it?**  
ElasticSearch is a **distributed search and analytics engine**. It is designed to search, analyze, and retrieve large volumes of data very quickly — much faster than a traditional SQL database for search queries.

**Plain language analogy:**  
ElasticSearch is like a **smart search engine** inside the chatbot — Google for your internal knowledge base.

**How it is used in Stacy:**
- Used by the Trinity AI engine to **fetch and rank responses** after NLP processing
- Stores the indexed FAQ content and knowledge base
- Returns the most relevant response to a given user query
- Deployed in a cluster: **1 master node + 2 data nodes** for high availability
- In a DR failover, the ES instance must be updated to point to the DR environment
- On rollback, the Neo4j knowledge graph must be reloaded (a common gotcha)

---

### 4.7 Oracle Database

**What is it?**  
Oracle DB is a **relational database management system (RDBMS)** — it stores structured data in tables with rows and columns. It supports SQL queries, transactions, and ACID compliance (Atomicity, Consistency, Isolation, Durability).

**Plain language analogy:**  
Oracle DB is the **official records room** — where all structured, important data is stored permanently and can be queried precisely.

**How it is used in Stacy:**
- Stores **customer profiles** (name, RelID, CustomerID, account details)
- Stores **transaction history and audit records**
- Stores **chat transcripts** (structured format)
- Stores analytics and reporting data
- The AI server queries Oracle for **real-time personalization** in post-login mode (e.g., "Your last transaction was...")
- Chat audit logs are written to Oracle after every conversation

---

### 4.8 NAS — Network Attached Storage

**What is it?**  
NAS stands for **Network Attached Storage**. It is a dedicated file storage server connected to the network, allowing multiple servers to read and write files from a central location. Unlike a database (which stores structured, query-able data), NAS stores raw files.

**Plain language analogy:**  
NAS is like a **shared network drive** (like a company's shared folder on Windows) — everyone can access it, and you store documents, logs, and files there — not spreadsheets with calculations.

**Protocols it uses:** NFS, SMB, CIFS

**How it is used in Stacy:**
- Stores **AI server configuration files**
- Stores **chat transcripts as files** (raw logs)
- Stores **media and attachments**
- Stores **backup snapshots** of the system
- Admin server reads config from NAS at startup
- Chatbot backend writes session logs and audit files to NAS

**Key distinction — NAS vs Oracle DB:**

| NAS | Oracle DB |
|-----|-----------|
| Files, logs, media, configs | Structured rows and columns |
| No SQL queries | Full SQL support |
| Large, unstructured data | Precise, relational data |
| e.g., chat transcript files | e.g., customer profile table |

---

### 4.9 F5 Load Balancer

**What is it?**  
F5 is an **Application Delivery Controller (ADC)** — an enterprise-grade load balancer that distributes incoming network traffic across multiple servers to ensure no single server gets overwhelmed.

**Plain language analogy:**  
F5 is like a **traffic policeman at a busy intersection** — directing cars (requests) to different lanes (servers) so no single lane jams up.

**Algorithm used:** Round Robin (requests distributed evenly across all available servers in rotation)

**How it is used in Stacy:**
- Sits in front of the web/app server cluster
- Distributes all user traffic across **multiple app nodes**
- Performs **SSL offloading** — decrypts HTTPS traffic so app servers don't have to
- **Health monitoring** — automatically removes an unhealthy server from rotation
- Provides **automatic failover** — if one server dies, traffic instantly goes to others
- Industry standard in banking environments

```
User request
     ↓
F5 Load Balancer
     ↓          ↓          ↓
App Node 1  App Node 2  App Node 3   ← Round Robin
```

---

### 4.10 Axway API Gateway

**What is it?**  
Axway is an **enterprise-grade API gateway** — a software layer that sits between external clients and internal backend services. It acts as a secure front door for all API traffic.

**Plain language analogy:**  
Axway is like a **security desk at a corporate building entrance** — it checks your ID (authentication), controls which floors you can access (authorization), logs your entry (audit), and stops suspicious visitors (rate limiting).

**How it is used in Stacy (MVP1 — Pre-Login):**
- Sits between the Chatbot Client and the internal Process API
- Enforces **authentication and authorization** for all incoming API calls
- **Rate limiting** — prevents abuse (100 req/min)
- **Request validation** — rejects malformed or invalid requests
- Centralized **monitoring, logging, and audit trail**
- Acts as a secure proxy between the internet-facing chatbot and internal banking services

> Note: Axway is used for **pre-login (unauthenticated)** traffic. **Kong IAG** is used for **post-login (authenticated)** custom API integrations in MVP2+.

---

### 4.11 Kong API Gateway (IAG)

**What is it?**  
Kong is another **API gateway**, used specifically for post-login flows in MVP2+. IAG stands for **Internal API Gateway**.

**How it is used in Stacy:**
- Manages API calls to internal banking services (ICM, CSC, etc.)
- Handles authenticated requests from SC Mobile
- Routes customer-specific API calls to the right backend service
- Works in conjunction with Axway — different gateways for different trust zones

---

### 4.12 Akamai BPM

**What is it?**  
Akamai is a global **Content Delivery Network (CDN) and security platform**. BPM (Bot Protection Manager) is its feature for protecting web applications from bot traffic, DDoS attacks, and malicious requests — before they even reach your servers.

**Plain language analogy:**  
Akamai is like a **bouncer outside the building** — it stops trouble before it walks through the door. Even before Axway (the security desk) sees the request, Akamai has already filtered out bots and attackers.

**How it is used in Stacy:**
- First line of defence — sits at the **internet edge**
- Blocks **DDoS attacks** (distributed denial-of-service — flooding the server with fake traffic)
- Blocks **bot traffic** — prevents automated scripts from abusing the chatbot API
- Protects the **Prelogin API** endpoint
- Only legitimate human traffic passes through to the chatbot

---

### 4.13 DMZ1 — Outer Demilitarized Zone

**What is it?**  
A **DMZ (Demilitarized Zone)** in networking is a physical or logical subnetwork that separates an internal network from an untrusted external network (the internet). It acts as a **buffer zone**.

**DMZ1 is the outer layer** — the first network boundary after the internet.

**Plain language analogy:**  
DMZ1 is like the **reception area of a secure office building** — visitors from outside are allowed here, but they can't go deeper into the building without further checks.

**What lives in DMZ1:**
- Web server (receives HTTP/HTTPS requests)
- Load balancer (F5)
- Reverse proxy
- Only minimal, necessary services are exposed here

---

### 4.14 DMZ2 — Inner Demilitarized Zone

**What is it?**  
DMZ2 is the **second network boundary** — between DMZ1 and the internal private network.

**Plain language analogy:**  
DMZ2 is like the **badge-restricted area** just inside the reception — you need additional clearance to go further into the building.

**What lives in DMZ2:**
- App server
- API Gateway (Axway)
- Services that need to communicate with both DMZ1 and the internal network

**Network Zones Summary:**

```
Internet
   ↓
DMZ1  (Web Server, Load Balancer, Reverse Proxy)
   ↓
DMZ2  (App Server, API Gateway)
   ↓
Internal Network  (Oracle DB, AI Server, NAS, ICM)
```

> The internal network is **never directly reachable from the internet**. All traffic must pass through DMZ1 → DMZ2 → Internal. This is the fundamental principle of **defense in depth**.

---

### 4.15 Neo4j — Knowledge Graph Database

**What is it?**  
Neo4j is a **graph database** — instead of storing data in tables (like Oracle), it stores data as **nodes and relationships** (a graph). This is ideal for knowledge bases where the *connections between concepts* matter.

**Plain language analogy:**  
Neo4j is like a **mind map** — every concept is connected to related concepts, making it fast to traverse relationships like "What topics relate to PAD charges?" or "What intents connect to the lost card flow?"

**How it is used in Stacy:**
- Stores the **AI knowledge graph** — the structured knowledge base that Trinity uses for intent mapping and response generation
- Must be **reloaded** after an ElasticSearch rollback (critical operational step)
- Managed through the Admin console: Login → Manage AI → Knowledge Graph → Reload Graph

---

### 4.16 reCAPTCHA

**What is it?**  
reCAPTCHA is a **human verification system** by Google. It presents a challenge (e.g., "click all traffic lights") or silently scores user behaviour to determine if the visitor is human or a bot.

**How it is used in Stacy:**
- Displayed to every user **before the chatbot UI opens** (pre-login)
- Prevents automated scripts and bots from abusing the chatbot API
- Only users who pass the reCAPTCHA can open the chat window
- Works in conjunction with Akamai BPM — Akamai blocks large-scale attacks; reCAPTCHA catches individual bots

---

### 4.17 CASAS Server

**What is it?**  
CASAS is an **SSO (Single Sign-On) token server** used within SCB's infrastructure. It issues short-lived authentication tokens that allow one system to trust another without sharing credentials.

**How it is used in Stacy (Post-Login / Live Bank handoff):**
1. User selects "Connect to Live Bank agent" in the chatbot
2. Chatbot connects to CASAS server
3. CASAS issues an **SSO token**
4. Chatbot passes the token to Live Bank
5. Live Bank validates the token with CASAS
6. User is seamlessly authenticated in Live Bank — no second login needed

> The chatbot **never holds long-lived credentials** — it only passes a short-lived SSO token. This is a key security design principle.

---

### 4.18 ICM — Integrated Customer Master

**What is it?**  
ICM is SCB's **central customer data repository** — the authoritative source of truth for all customer profile information.

**Data it holds:**
- RelID (Relationship ID)
- Customer Name
- CustomerID
- Account details
- Product holdings

**How it is used in Stacy:**
- In post-login flow, Active AI calls the Customer API → which retrieves customer data from ICM via the CSC layer
- Used to **personalize** chatbot responses (e.g., "Hello, John. Your account ending in 1234...")

---

### 4.19 CSC Layer (Customer Service Channel)

**What is it?**  
CSC is an **integration middleware layer** within SCB that provides a standardized interface for accessing customer data from various backend systems including ICM.

**How it is used in Stacy:**
- Acts as the **intermediary** between the Active AI app and the ICM data source
- Text chat via the **Kontra interface** (CSC's chat protocol)
- All customer data retrieval in post-login goes through CSC

---

### 4.20 WebSocket

**What is it?**  
WebSocket is a **communication protocol** that provides a persistent, full-duplex (two-way) connection between a client and server. Unlike HTTP (which is request-response — client asks, server answers, connection closes), WebSocket keeps the connection open.

**Plain language analogy:**  
HTTP is like sending a **letter** — you send, wait, get a reply, done. WebSocket is like a **phone call** — the line stays open and both sides can talk anytime.

**How it is used in Stacy:**
- Used for **post-login authenticated sessions** (MVP2)
- Enables real-time bidirectional chat — the server can push messages to the client without the client asking
- Required for live agent chat (Live Bank) where messages flow in real-time both ways
- Pre-login uses standard REST/HTTP; post-login upgrades to WebSocket

---

## 5. High Level Architecture

### 5.1 Architecture Overview

The Stacy chatbot architecture is organized into **layered zones**, each serving a specific purpose. Traffic always flows from the internet → through security layers → into the internal network. At no point is the internal network directly accessible from the outside.

```
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET                                  │
└──────────────────────────┬──────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│              AKAMAI BPM  (DDoS + Bot Protection)                 │
│                   First line of defence                          │
└──────────────────────────┬───────────────────────────────────────┘
                           ↓
                  reCAPTCHA Validation
                  (Human verification)
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│                        DMZ 1 (Outer)                             │
│   ┌──────────────────┐     ┌──────────────────────────────────┐  │
│   │  Web Server      │     │  F5 Load Balancer                │  │
│   │  (HTTP/HTTPS)    │ --> │  (Round Robin, SSL Offload)      │  │
│   └──────────────────┘     └──────────────────────────────────┘  │
└──────────────────────────┬───────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│                        DMZ 2 (Inner)                             │
│   ┌──────────────────────────────────────────────────────────┐   │
│   │  Axway API Gateway  (Auth, Rate Limit, Policy, Audit)    │   │
│   └──────────────────────────────────────────────────────────┘   │
│   ┌──────────────────────────────────────────────────────────┐   │
│   │  App Server Cluster  (Node 1 | Node 2 | Node 3)          │   │
│   └──────────────────────────────────────────────────────────┘   │
└──────────────────────────┬───────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│                   INTERNAL NETWORK                               │
│                                                                   │
│   ┌──────────────────────────────────────────────────────────┐   │
│   │              Morpheus Admin & App Server                  │   │
│   │         (NLP, Intent, Session, Orchestration)             │   │
│   └───────────────────┬────────────────────────-─────────────┘   │
│                       │                                           │
│           ┌───────────┴──────────┐                               │
│           ↓                      ↓                               │
│   ┌──────────────┐     ┌──────────────────────────┐             │
│   │ Redis Cache  │     │   Trinity AI Engine       │             │
│   │ (30 min TTL) │     │   (NLP + ElasticSearch)   │             │
│   └──────────────┘     └──────────────┬────────────┘             │
│                                       ↓                          │
│                           ┌──────────────────────┐              │
│                           │  Neo4j Knowledge Graph│              │
│                           └──────────────────────┘              │
│                                                                   │
│   ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐    │
│   │  Oracle DB   │   │   NAS        │   │  ICM (Customer   │    │
│   │  (Structured)│   │  (Files/Logs)│   │  Master Data)    │    │
│   └──────────────┘   └──────────────┘   └──────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

---

### 5.2 Full MVP1 Request Path (Step by Step)

| Step | What Happens |
|------|-------------|
| 1 | User opens sc.com in browser |
| 2 | Akamai BPM inspects traffic — blocks bots and DDoS |
| 3 | User clicks Stacy widget → reCAPTCHA shown |
| 4 | User passes reCAPTCHA → Chatbot UI loads |
| 5 | User types a message → API call made to chatbot backend |
| 6 | Request hits F5 Load Balancer → routed to an App Node |
| 7 | Axway API Gateway validates and authenticates the request |
| 8 | Request reaches Morpheus (App Server) |
| 9 | Morpheus checks Redis Cache |
| 10 | Cache HIT → response returned immediately |
| 11 | Cache MISS → request forwarded to Trinity AI |
| 12 | Trinity runs NLP + queries ElasticSearch via Neo4j |
| 13 | Response generated → cached in Redis → sent to user |
| 14 | Chat audit written to Oracle DB |

---

### 5.3 Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Security Edge | Akamai BPM | DDoS, bot protection |
| Human Check | reCAPTCHA | Verify real users |
| Load Balancer | F5 | Traffic distribution, SSL offload |
| API Gateway (pre-login) | Axway | Auth, rate limit, audit |
| API Gateway (post-login) | Kong IAG | Custom API routing |
| AI Engine | Morpheus + Trinity | NLP, intent, response generation |
| Cache | Redis + Sentinel | Fast responses, HA |
| Search | ElasticSearch | Knowledge base retrieval |
| Knowledge Graph | Neo4j | AI intent mapping |
| Structured DB | Oracle DB | Customer data, audit, analytics |
| File Storage | NAS | Configs, logs, transcripts |
| Customer Data | ICM via CSC | Personalization |
| SSO | CASAS | Live Bank authentication |
| Network Zones | DMZ1 + DMZ2 | Defense in depth |
| Protocol (post-login) | WebSocket | Real-time bidirectional chat |

---

## 6. Detailed Data Flow — Pre-Login & Post-Login

---

### 6.1 Pre-Login Flow (MVP1)

**Scenario:** A customer visits sc.com and asks the chatbot "What is the minimum balance for a savings account?"

```
Step 1:  User opens sc.com
          ↓
Step 2:  Akamai BPM inspects the request
         → If bot / DDoS detected → BLOCKED
         → If legitimate traffic → PASS
          ↓
Step 3:  User clicks Stacy icon
         reCAPTCHA challenge displayed
         → If fails → chatbot does NOT open
         → If passes → chatbot UI loads
          ↓
Step 4:  Chatbot UI loads (iframe overlay)
         Shows Stacy avatar, typing area, timestamp
          ↓
Step 5:  User types: "What is minimum balance?"
         API call sent from browser → Internet → Web Server
          ↓
Step 6:  F5 Load Balancer receives request
         Routes to App Node 2 (Round Robin)
          ↓
Step 7:  Axway API Gateway
         → Validates request format
         → Checks rate limit (100 req/min)
         → Authenticates pre-login token
         → Logs request (audit)
         → PASS → forwards to Process API
          ↓
Step 8:  Process API → Morpheus App Server
         Morpheus receives the message
          ↓
Step 9:  Morpheus checks Redis Cache
         Key: hash of user message
         ┌─────────────────────────────────┐
         │  CACHE HIT (answer < 30 min old) │
         │  → Return cached response        │
         │  → Total time: ~50ms             │
         └─────────────────────────────────┘
                        OR
         ┌─────────────────────────────────┐
         │  CACHE MISS (not in cache)       │
         │  → Forward to Trinity AI         │
         └─────────────────────────────────┘
          ↓ (if MISS)
Step 10: Trinity AI Engine
         → NLP processing on message
         → Intent identified: faq_minimum_balance
         → Queries ElasticSearch for matching answer
         → ElasticSearch searches knowledge base
         → Returns ranked answer
          ↓
Step 11: Response generated
         → Stored in Redis cache (TTL: 30 min)
         → Sent back through the chain to Chatbot UI
          ↓
Step 12: Chatbot UI renders response
         "The minimum balance for a savings account is SGD 500."
          ↓
Step 13: Chat audit record written to Oracle DB
         (message, intent, timestamp, session ID)
```

---

### 6.2 Post-Login Flow (MVP2)

**Scenario:** A logged-in customer on SC Mobile asks for their account balance and then requests to speak to a Live Bank agent.

```
Step 1:  Customer opens SC Mobile and logs in
         2FA / OTP authentication completed
          ↓
Step 2:  Customer opens the Stacy chatbot
         SC Mobile app sends the user's ACCESS TOKEN to Active AI
         (Token proves the user is authenticated)
          ↓
Step 3:  Active AI (Morpheus) calls the Customer API
         Request includes the access token
          ↓
Step 4:  Customer API fetches customer data from ICM via CSC layer
         Data retrieved:
         - RelID
         - Customer Name
         - CustomerID
          ↓
Step 5:  Morpheus now has customer context
         Personalized greeting shown: "Hello, John. How can I help?"
          ↓
Step 6:  Customer types: "What is my account balance?"
         Post-login request → goes through Kong IAG (not Axway)
         Kong validates the authenticated token
         → Calls internal banking API
         → Retrieves real account balance from Oracle DB
         → Returns personalized response
          ↓
Step 7:  Customer requests: "Connect me to a Live Bank agent"
         ↓
Step 8:  Chatbot connects to CASAS Server
         CASAS generates an SSO Token for this session
          ↓
Step 9:  Active AI passes the SSO Token to Live Bank
          ↓
Step 10: Live Bank validates the token with CASAS
         → Token valid → Live agent connected
         → Conversation context handed over (customer doesn't repeat themselves)
          ↓
Step 11: WebSocket connection established for real-time chat
         User ↔ Live Agent chat begins
          ↓
Step 12: Chat transcript stored in Oracle DB and NAS
```

---

### 6.3 Data Pipeline (Authenticated Flow)

```
Chatbot UI
   ↓
Authenticated via Axway / Kong header (token validation)
   ↓
WebSocket connectivity established
   ↓
Active AI Chat Application (Morpheus)
   ↓
Consume: Live Bank / Suggest / Check / Resolve
   ↓
Oracle DB  ←→  NAS  ←→  ICM  ←→  CASAS
```

---

### 6.4 Cache Hit vs Miss — Decision Summary

| Scenario | Path | Response Time |
|----------|------|--------------|
| FAQ asked before (cache HIT) | Morpheus → Redis → User | ~50-100ms |
| New query (cache MISS) | Morpheus → Trinity → ES → Redis → User | ~1-2 sec |
| Post-login personalized | Morpheus → Kong → ICM → Oracle → User | ~1-2 sec |
| Live Bank handoff | Morpheus → CASAS → Live Bank (WebSocket) | ~2-3 sec |

---

### 6.5 DR Failover — Service Startup Order

When switching to DR environment, services must start in **exactly this order**. Wrong order causes cascading failures.

```
Step 1:  Supervisor starts (PROD & DR)
         Auto-starts: Manager + Triniti service
          ↓
Step 2:  ElasticSearch starts
         Config: 1 master node + 2 data nodes
          ↓
Step 3:  Redis Server starts
         Then: Redis Sentinel starts (HA monitor)
          ↓
Step 4:  Neo4j Server starts
         (Knowledge graph must be available before AI engine)
          ↓
Step 5:  Morpheus + Admin + JBoss starts
         (App layer comes last — depends on all above)
          ↓
Step 6:  Update ES Instance in Admin Console
         Login Admin → Choose Workspace → Config Workspace
         → Manage Workspace Rule → Search Index
         → Update Rule → Point to DR ES instance
         Example: http://10.7.176.207:9200
```

---

### 6.6 Rollback Case (Critical Gotcha)

When rolling back PROD ElasticSearch to a previous version, you **must also reload the Neo4j knowledge graph**. Skipping this step causes the AI to give wrong or stale responses.

```
Login Admin Console
   ↓
Choose Workspace
   ↓
Manage AI
   ↓
Knowledge Graph
   ↓
Reload Graph
```

> This step is frequently missed in DR drills. Always include it in the runbook.

---

## 7. Interview Questions & Answers

This section contains all interview-ready Q&As based on the Stacy chatbot architecture.

---

### Q1: Why is Redis used instead of querying Oracle DB every time?

**Answer:**  
Oracle DB is a **disk-based relational database** — every query involves I/O operations, connection pooling, SQL parsing, and row retrieval. This takes time, typically 50-500ms per query.

Redis is **in-memory** — data is stored in RAM. A Redis lookup takes **microseconds**, not milliseconds.

For a chatbot where the same FAQ questions are asked hundreds of times per day, caching the responses in Redis is a significant optimization. The first user to ask "What is minimum balance?" triggers a Trinity AI lookup (slow path). The next 500 users who ask the same question hit Redis and get the answer in milliseconds.

The 30-minute TTL ensures answers don't become too stale.

---

### Q2: What happens if Redis goes down?

**Answer:**  
Redis is deployed with **Redis Sentinel** — a high-availability solution for Redis.

Sentinel continuously monitors the Redis master node. If the master goes down:
1. Sentinel detects the failure (within seconds)
2. Sentinel automatically **promotes a replica node to become the new master**
3. All clients are redirected to the new master

During the brief failover window, cache miss requests fall through to the Trinity AI engine — responses are slower but the chatbot remains functional. This is by design — **Availability over Consistency** (CAP theorem choice).

---

### Q3: Why are there two API Gateways — Axway and Kong?

**Answer:**  
They serve **different trust zones and different purposes**:

| Gateway | Used For | Why |
|---------|---------|-----|
| **Axway** | MVP1 Pre-login (unauthenticated, public) | Enterprise-grade, proven for public-facing APIs, strong audit trail |
| **Kong IAG** | MVP2+ Post-login (authenticated, internal) | Flexible for custom internal API routing, integrates with ICM and CSC |

Having separate gateways for authenticated vs unauthenticated traffic is a **security best practice** — the authenticated gateway can apply stricter policies, token validation, and different rate limits without affecting the public-facing chatbot experience.

---

### Q4: How does the post-login flow maintain security without the chatbot holding long-lived credentials?

**Answer:**  
The design uses **short-lived tokens**, never long-lived credentials:

1. SC Mobile generates an **access token** for the logged-in user
2. The chatbot uses this access token to call the Customer API — the token proves identity
3. For Live Bank handoff, **CASAS issues an SSO token** — a short-lived, single-use token
4. This SSO token is passed to Live Bank, which validates it with CASAS
5. Once validated, the token's job is done

The chatbot backend **never stores** the user's password, PIN, or any long-lived secret. Even if the chatbot were compromised, an attacker would only find short-lived tokens that expire quickly.

---

### Q5: Why was Availability chosen over Consistency in the CAP theorem decision?

**Answer:**  
CAP theorem states a distributed system can only guarantee two of: Consistency, Availability, Partition Tolerance. Since network partitions are a reality in any distributed system, the real choice is between **Consistency and Availability**.

For Stacy, the reasoning is:
- If the chatbot returns a slightly **stale FAQ answer** (e.g., cached 25 minutes ago), the customer gets an answer. It may be slightly old, but it's still correct in most cases.
- If the chatbot is **unavailable**, the customer gets nothing and calls the hotline — which completely defeats the purpose of building the chatbot.

**The cost of unavailability >> the cost of slight inconsistency** for an FAQ chatbot.

Redis with a 30-minute TTL directly implements this decision — the cache may serve slightly stale data but ensures the system keeps responding even if Trinity AI is temporarily slow or overloaded.

---

### Q6: How would you scale the system from 5,000 to 50,000 concurrent users?

**Answer:**  
Scale **horizontally** at every layer:

| Layer | Scaling Action |
|-------|---------------|
| Web/App Servers | Add more nodes behind F5 — zero downtime, F5 auto-discovers new nodes |
| Redis | Expand to a **Redis Cluster** — multiple shards, each handling a portion of keys |
| Trinity AI | Deploy multiple Trinity instances with a load balancer in front |
| ElasticSearch | Add more data nodes to the ES cluster |
| Oracle DB | Add **read replicas** — all AI read queries go to replicas; writes go to primary |
| API Gateway | Scale Axway horizontally — active-active deployment |

Additionally:
- Increase Redis TTL to reduce AI engine hits during peak
- Add **connection pooling** at the DB layer to handle higher throughput
- Consider **pre-warming** the cache for known high-frequency queries at startup

The architecture is designed for this — no stateful sessions in the app layer means any node can handle any request.

---

### Q7: What is the difference between NAS and Oracle DB? When do you use each?

**Answer:**

| Criteria | NAS | Oracle DB |
|----------|-----|-----------|
| Data type | Files, blobs, logs, media | Structured rows and columns |
| Query method | File path lookup, NFS/SMB | SQL queries |
| Use case | Store chat transcripts as files, config files, logs, backups | Customer profiles, account data, audit trails, analytics |
| Transactions | No | Yes (ACID) |
| Speed | Fast for large sequential reads | Fast for indexed relational queries |
| In Stacy | Config files, raw log files, message archives | Customer data, chat audit records, transaction history |

**Rule of thumb:** If you need to query it with SQL, put it in Oracle. If you just need to store and retrieve it as a file, put it in NAS.

---

### Q8: Why is the service startup order critical in DR? What happens if you start in wrong order?

**Answer:**  
Each service depends on a previous one:

- **Morpheus** needs Neo4j (knowledge graph) to be ready before it can process AI queries
- **Trinity** needs ElasticSearch to be up before it can run searches
- **ElasticSearch** needs the Supervisor to have started the service manager
- **Redis Sentinel** needs Redis Server to be already running before it can monitor it

If you start Morpheus before Neo4j → Morpheus starts but has no knowledge graph → chatbot gives wrong/empty responses → looks like a bug even though services are "up."

The correct order guarantees that when the final service (Morpheus/JBoss) starts, all its dependencies are already healthy. This is a critical operational discipline and should be documented in the runbook.

---

### Q9: What is the role of DMZ1 and DMZ2? Why not just put everything in one zone?

**Answer:**  
DMZs implement the principle of **defense in depth** — multiple independent security layers. If one is breached, the next one still protects the internals.

**DMZ1** is the first line after the internet — it hosts only what needs to be publicly reachable (web server, load balancer). A compromise here exposes only these services, not the app logic.

**DMZ2** adds another boundary — the app server and API gateway sit here. Even if DMZ1 is compromised, an attacker still cannot reach the internal network without also breaking through DMZ2.

**Internal Network** — Oracle DB, AI server, ICM — is never reachable from the internet at all.

Putting everything in one zone means a single breach exposes the entire system, including customer data. The DMZ design limits the **blast radius** of any single compromise.

---

### Q10: What is the Neo4j reload step after an ES rollback, and why is it critical?

**Answer:**  
ElasticSearch and Neo4j are **tightly coupled** in the AI pipeline:
- Neo4j holds the **knowledge graph** — the map of intents, concepts, and their relationships
- ElasticSearch holds the **indexed content** — the actual text of responses, ranked by relevance

When you roll back ElasticSearch to a previous index version, the content in ES changes. If Neo4j is not reloaded, it still has the graph from the newer version — the relationships and intents no longer match the ES content correctly.

Result: Trinity AI processes a query, navigates the (new) graph in Neo4j, queries ES with the wrong index keys, and gets wrong or empty results → incorrect chatbot responses.

**Reloading Neo4j** re-syncs the knowledge graph with the rolled-back ES index, restoring correct AI behaviour.

This step is frequently missed because it is not obvious — both services appear "up and healthy" individually, but they are out of sync with each other.

---

*End of KT Notes — Stacy Active AI Chatbot | Standard Chartered Bank Singapore*
