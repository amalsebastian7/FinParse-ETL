# 📊 FinParse ETL 
*(Formerly StatementProcessor)*

**An enterprise-grade Java pipeline for automated extraction, transformation, and AI-driven categorization of financial statements.**

[![Status: Active Development](https://img.shields.io/badge/Status-Active_Development-brightgreen?style=for-the-badge)](https://github.com/amalsebastian7/StatementProcessorJava)
[![Language: Java 21](https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://oracle.com/java)
[![Build: Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)](https://maven.apache.org/)
[![DB: SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)](https://sqlite.org/)
[![AI: Gemini](https://img.shields.io/badge/AI-Google_Gemini-8E75B2?style=for-the-badge&logo=google&logoColor=white)](https://ai.google.dev/)

</div>

---

## 📖 Overview
FinParse ETL is a modular, privacy-first data engineering project designed to ingest raw bank statements (CSV and PDF) from local file systems or NAS, parse the unstructured financial data, and process it for deep personal analytics. 

Instead of relying on fragile manual data entry or paying for cloud budgeting apps that sell your data, this pipeline runs **100% locally**. It utilizes a hybrid **Rule-Based & AI-Enriched Categorization Engine** alongside a deterministic deduplication system to accurately track internal transfers, clean historical data, and securely store it for querying.

---

## How It Works (High-Level Pipeline)

```mermaid
flowchart LR
    %% Colors
    classDef extract fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef transform fill:#fff9c4,stroke:#fbc02d,stroke-width:2px;
    classDef load fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;
    classDef ai fill:#f3e5f5,stroke:#8e24aa,stroke-width:2px;

    A[(Raw CSV/PDF)] -->|1. Ingest| B(File Parsers):::extract
    B -->|2. Normalize| C(Deduplication Engine):::transform
    C -->|3. Categorize| D{Hybrid AI}:::ai
    
    D -.->|Tier 1: Fast Regex| E[(SQLite DB)]:::load
    D -.->|Tier 2: Gemini LLM| E

```

## ✨ Core Architecture & Features

* 🗂️ **Multi-Format Ingestion:** Automated parsing of complex CSV layouts (via OpenCSV) and unstructured legacy PDFs (via Apache PDFBox).
* 🧮 **Deterministic Deduplication:** Rule-based SQL/Java matching engine that automatically detects and nullifies internal transfers between user accounts within a 48-hour time window.
* 🧠 **Hybrid Categorization Pipeline:**
* *Tier 1 (Zero-Cost):* Sub-millisecond Regex matching for ~80% of known merchants (e.g., Tesco, Netflix).
* *Tier 2 (AI-Enriched):* Batch LLM processing via native HTTP clients to categorize ambiguous transactions using strict JSON schemas.

* 🔐 **Security First:** Strict separation of concerns, `.properties`/environment variable credential management, and dummy data for isolated testing. Raw financial data never leaves the host machine.

## 🛠️ Tech Stack

| Category | Technologies Used |
| --- | --- |
| **Core Language** | Java 21 (Modular separation of concerns) |
| **Build & Dependencies** | Maven |
| **Data Extraction** | OpenCSV, Apache PDFBox |
| **Database Layer** | SQLite JDBC (Zero-config embedded DB for local analytics), Flyway |
| **AI Integration** | Native Java HTTP/JSON parsing (Jackson) for Google Gemini API |
| **Testing** | JUnit 5, AssertJ, WireMock, Testcontainers |


## 📌 Pipeline Strategy Board

```mermaid
flowchart TD
    %% Custom "Sticky Note" Styles
    classDef yellowNote fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,color:#000,stroke-dasharray: 5 5
    classDef blueNote fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#000
    classDef greenNote fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000
    classDef purpleNote fill:#f3e5f5,stroke:#8e24aa,stroke-width:2px,color:#000
    classDef dbNode fill:#efebe9,stroke:#4e342e,stroke-width:2px,color:#000

    subgraph The Board [ETL Pipeline Strategy Board]
        A["📝 1. Drop Statements<br/>(Monzo/Barclays)"]:::yellowNote
        
        B["⚙️ 2. Clean & Normalize<br/>(Dates/Currencies)"]:::blueNote
        C["🧮 3. Match Internal Transfers<br/>(48-hour window)"]:::blueNote
        
        D["⚡ 4. Fast Regex Rules<br/>(~80% of data)"]:::greenNote
        E["🧠 5. Gemini AI Fallback<br/>(JSON Schema)"]:::purpleNote
        
        F[("🗄️ 6. SQLite Warehouse")]:::dbNode

        A --> B --> C
        C --> D & E
        D --> F
        E --> F
    end

```

---

## 🗺️ Project Mindmap

```mermaid
mindmap
  root((FinParse ETL))
    Ingestion
      OpenCSV Engine
      Apache PDFBox
      Local File Watcher
    Reconciliation Engine
      Date ISO-8601
      Currency Alignment
      Transfer Deduplication
    Hybrid AI Categorizer
      Tier 1: Regex
        Rules.json
        Sub-millisecond
      Tier 2: Gemini
        Flash 1.5 API
        Strict JSON Schema
    Local Storage
      SQLite Embedded
      Flyway Migrations
      Zero-Cloud PII

```
## 📂 Project Documentation Hub

This repository maintains a comprehensive architecture and product documentation matrix. *Click on any document link to view the specific file.*

<details>
<summary><b>📋 Click to expand: Application Master Specification Taxonomy</b></summary>

This project's documentation adheres to the following enterprise specification standard:

```text
APPLICATION MASTER SPECIFICATION
│
├── 01 Product                  # Strategic alignment and user needs
│   ├── Vision & Problem        # Why this pipeline exists and MVP scope
│   └── Requirements (PRD)      # User stories and BDD acceptance criteria
│
├── 02 Architecture             # High-level system design
│   ├── System Architecture     # C4 Model container diagrams
│   ├── Detailed Pipeline       # Granular step-by-step ETL execution mapping
│   ├── Class Diagrams          # Object-oriented domain design (UML)
│   └── ADRs                    # Architecture Decision Records (Historical log)
│
├── 03 Data                     # Persistence and data integrity
│   ├── Database Schema         # SQLite DDL and entity relationships
│   └── Data Lifecycle          # Sanitization and PII retention policies
│
├── 04 APIs                     # External system interactions
│   ├── Integration Contracts   # LLM endpoints and JSON schema enforcement
│   └── Authentication          # API key header management
│
├── 05 Security                 # Threat mitigation
│   ├── Threat Model            # STRIDE methodology and local-first boundaries
│   └── Privacy                 # Pre-LLM data redaction and zero-cloud PII
│
├── 06 Engineering              # Development standards
│   ├── Repository Structure    # Modular separation of concerns
│   └── Testing Strategy        # Unit/Integration tiers using Dummy Data
│
└── 07 Operations               # Running the software
    ├── Deployment              # Native OS execution instructions
    └── Automation              # Systemd/Cron configuration for file watching
```

---
