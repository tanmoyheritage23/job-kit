# Netcracker - Capital Markets Processing Platform
**Client context:** Morgan Stanley, Goldman Sachs, JP Morgan  
**Role:** Software Engineer (Jun 2025 - Present)

---

## 1. High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                                          │
│   ┌────────────────┐   ┌─────────────────┐   ┌─────────────────────┐        │
│   │ Morgan Stanley │   │  Goldman Sachs  │   │    JP Morgan        │        │
│   └───────┬────────┘   └────────┬────────┘   └──────────┬──────────┘        │
└───────────┼────────────────────┼──────────────────────┼────────────────────┘
            │                    │                      │
            └────────────────────▼──────────────────────┘
                        ┌────────────────┐
                        │  API Gateway   │
                        │  (Load Balancer│
                        │  + Auth/TLS)   │
                        └───────┬────────┘
                                │
┌───────────────────────────────▼─────────────────────────────────────────────┐
│                     MICROSERVICES LAYER  (Java + Spring Boot)                │
│                                                                               │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐   │
│  │   Derivatives    │  │    Cashflow      │  │   Data Reconciliation    │   │
│  │  Calculation     │  │   Component      │  │        Module            │   │
│  │    Service       │  │                  │  │                          │   │
│  │ (Unified logic,  │  │ (Single source   │  │ (Advanced algorithms,    │   │
│  │  +40% efficiency)│  │  35% accuracy↑)  │  │  63% faster processing)  │   │
│  └────────┬─────────┘  └────────┬─────────┘  └───────────┬──────────────┘   │
│           │                    │                          │                  │
│  ┌────────▼─────────────────────▼──────────────────────────▼──────────────┐ │
│  │                        P&L Calculation Engine                           │ │
│  │         (Real-time P&L status views + direct adjustments)               │ │
│  └──────────────────────────────┬──────────────────────────────────────────┘ │
│                                 │                                            │
│  ┌──────────────────────────────▼──────────────────────────────────────────┐ │
│  │                   Workflow UI Service                                   │ │
│  │       (Product line views, P&L dashboards, trade adjustments)          │ │
│  │                       +20% user efficiency                             │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
┌──────────────────────────────────▼──────────────────────────────────────────┐
│                     BATCH PROCESSING ENGINE                                  │
│      Overnight batch: 9 hours → 105 minutes  (88% reduction)                │
│      4 million lines of redundant legacy code removed                        │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
┌──────────────────────────────────▼──────────────────────────────────────────┐
│                          DATA LAYER                                          │
│                                                                               │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌────────────────┐  │
│  │  PostgreSQL │   │    Redis    │   │Apache Kafka │   │  Elasticsearch │  │
│  │  (Primary   │   │  (Cache /   │   │  (Message   │   │  (Audit /      │  │
│  │   Database) │   │  Hot data)  │   │    Bus)     │   │   Search)      │  │
│  └─────────────┘   └─────────────┘   └─────────────┘   └────────────────┘  │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
┌──────────────────────────────────▼──────────────────────────────────────────┐
│                        EXTERNAL DATA SOURCES                                 │
│                                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │ Market Data  │  │  Trade Mgmt  │  │  Position    │  │  Reference Data  │ │
│  │   Feeds      │  │   Systems    │  │   Systems    │  │  (Instruments,   │ │
│  │ (Prices,     │  │ (OTC, ETD    │  │ (EOD Snap-   │  │   Counterparties)│ │
│  │  Rates, FX)  │  │  Trades)     │  │   shots)     │  │                  │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Data Flow Diagram

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                          DATA FLOW (End-to-End)                              ║
╚══════════════════════════════════════════════════════════════════════════════╝

STEP 1 — INGESTION  (Real-time + EOD feeds)
─────────────────────────────────────────────────────────────────────────────
  Market Data Feeds ──────┐
  Trade Mgmt Systems ─────┤──► Kafka Topics ──► Ingestion/ETL Service
  Position Systems ───────┤                          │
  Reference Data ─────────┘                          ▼
                                              Validate & Normalize
                                              (schema checks, dedup)
                                                      │
                                                      ▼
                                              PostgreSQL  ◄──────────────┐
                                              (raw trade/                │
                                               position tables)          │
                                                                         │
STEP 2 — CASHFLOW PROCESSING                                             │
─────────────────────────────────────────────────────────────────────────│
  PostgreSQL (authoritative source) ──────────────────────────────────── ┘
         │
         ▼
  ┌──────────────────────────────────────────────────────┐
  │              Cashflow Component                       │
  │  - Reads from single authoritative source            │
  │  - Eliminates intermediate data hops                 │
  │  - Computes scheduled/projected cashflows            │
  │  - Result: +35% pipeline accuracy                    │
  └───────────────────────┬──────────────────────────────┘
                          │ Cashflow events (Kafka)
                          ▼

STEP 3 — DERIVATIVES CALCULATION
─────────────────────────────────────────────────────────────────────────
  ┌──────────────────────────────────────────────────────┐
  │         Derivatives Calculation Service               │
  │  Input:  Cashflow data + Market prices + Positions   │
  │  Logic:  Unified calculation engine (Java/Spring)    │
  │          - Pricing, Greeks, Risk metrics             │
  │          - OTC and ETD derivatives                   │
  │  Result: +40% computational efficiency               │
  └───────────────────────┬──────────────────────────────┘
                          │ Derivatives metrics (Kafka)
                          ▼

STEP 4 — DATA RECONCILIATION
─────────────────────────────────────────────────────────────────────────
  ┌──────────────────────────────────────────────────────┐
  │         Data Reconciliation Module                    │
  │  Input:  Calculated values + Source system data      │
  │  Logic:  Advanced reconciliation algorithms          │
  │          - Cross-system position matching            │
  │          - Break detection and flagging              │
  │          - Auto-resolution for known patterns        │
  │  Result: 63% faster processing, higher accuracy      │
  └───────────────────────┬──────────────────────────────┘
                          │ Reconciled, clean data
                          ▼

STEP 5 — P&L ENGINE
─────────────────────────────────────────────────────────────────────────
  ┌──────────────────────────────────────────────────────┐
  │                 P&L Calculation Engine                │
  │  Input:  Reconciled positions + Derivatives metrics  │
  │          + Cashflows + Market snapshots              │
  │  Computes: Unrealized PnL, Realized PnL,             │
  │            Daily/MTD/YTD breakdowns                  │
  │  Output → Redis (hot P&L cache)                      │
  │         → PostgreSQL (persistent store)              │
  └───────────────────────┬──────────────────────────────┘
                          │
                          ▼

STEP 6 — OVERNIGHT BATCH
─────────────────────────────────────────────────────────────────────────
  ┌──────────────────────────────────────────────────────┐
  │            Batch Processing Engine                    │
  │  Runs:    End-of-day settlement + EOD P&L             │
  │           Full recon sweep + Report generation        │
  │  Before:  9 hours (bloated legacy onboarding code)   │
  │  After:   105 minutes (4M lines of legacy removed)   │
  │  Result:  88% runtime reduction                      │
  └───────────────────────┬──────────────────────────────┘
                          │
                          ▼

STEP 7 — WORKFLOW UI & REPORTING
─────────────────────────────────────────────────────────────────────────
  ┌──────────────────────────────────────────────────────┐
  │               Workflow UI Service                     │
  │  - Real-time P&L status dashboards                   │
  │  - Direct trade/position adjustments                 │
  │  - New product line views (integrated)               │
  │  - Break investigation and resolution workflow       │
  │  Result: +20% user efficiency for traders/ops        │
  └──────────────────────────────────────────────────────┘
          │                          │
          ▼                          ▼
  Morgan Stanley              Goldman Sachs / JP Morgan
  (Trading desks,             (Risk/Ops/Finance teams)
   Finance teams)
```

---

## 3. Component Summary Table

| Component | Tech | What it does | Your impact |
|---|---|---|---|
| Derivatives Calculation Service | Java, Spring Boot | Unified pricing + risk metrics for OTC/ETD derivatives | +40% computational efficiency |
| Cashflow Component | Java, Spring Boot | Projects cashflows from single authoritative source | +35% pipeline accuracy |
| Data Reconciliation Module | Java, algorithms | Cross-system position/trade matching, break detection | 63% faster processing |
| Batch Processing Engine | Java, Scheduler | EOD settlement, reports, full recon sweep | 88% runtime reduction (9h → 105min) |
| Workflow UI Service | Spring Boot + Frontend | P&L dashboards, trade adjustments, product views | +20% user efficiency |
| P&L Engine | Java, Spring Boot | Aggregates unrealized/realized PnL across portfolios | Real-time P&L for trading desks |

---

## 4. Key Architectural Decisions

- **Single authoritative source** for Cashflow (eliminated intermediate hops that caused 35% of reconciliation breaks)
- **Unified derivatives engine** (replaced fragmented per-product logic that caused inconsistencies across Morgan Stanley / GS / JPM environments)
- **Legacy code removal** (4M lines of onboarding scaffolding was the root cause of the 9-hour batch runtime)
- **Kafka-based decoupling** between ingestion, calculation, and reconciliation stages (enables independent scaling)
- **Redis caching** for hot P&L data so traders get sub-second reads without hitting the primary DB

---

*Based on Tanmoy Saha's Netcracker experience (Jun 2025 - Present). Client context: capital markets firms including Morgan Stanley, Goldman Sachs, JP Morgan.*
