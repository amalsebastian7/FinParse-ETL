# Detailed ETL Data Flow

This document maps the exact micro-steps the data takes from the moment the cron job fires to the moment it is committed to the SQLite database.

```mermaid
flowchart TB
    %% STYLING
    classDef trigger fill:#f9f,stroke:#333,stroke-width:2px;
    classDef process fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef decision fill:#fff9c4,stroke:#fbc02d,stroke-width:2px;
    classDef api fill:#e8f5e9,stroke:#388e3c,stroke-width:2px;
    classDef db fill:#efebe9,stroke:#5d4037,stroke-width:2px;

    %% PHASE 1
    subgraph P1 ["1. Initiation & Ingestion Layer"]
        1.1(["1.1: System Trigger (CLI / Cron)"]):::trigger
        1.2["1.2: Scan target directory for .csv & .pdf"]:::process
        1.3{"1.3: Is file already processed? <br/>(Hash check)"}:::decision
        1.4["1.4: Route to OpenCSV Adapter"]:::process
        1.5["1.5: Route to Apache PDFBox Adapter"]:::process
        
        1.1 --> 1.2 --> 1.3
        1.3 -- "New File (.csv)" --> 1.4
        1.3 -- "New File (.pdf)" --> 1.5
        1.3 -- "Duplicate" --> 1.6["1.6: Skip & Log"]
    end

    %% PHASE 2
    subgraph P2 ["2. Normalization Engine"]
        2.1["2.1: Map to RawTransaction DTO"]:::process
        2.2["2.2: Standardize Date to ISO-8601"]:::process
        2.3["2.3: Strip Currency Symbols & Parse BigDecimal"]:::process
        
        1.4 --> 2.1
        1.5 --> 2.1
        2.1 --> 2.2 --> 2.3
    end

    %% PHASE 3
    subgraph P3 ["3. Deduplication (Internal Transfers)"]
        3.1["3.1: Query DB for 48-hour transaction window"]:::db
        3.2{"3.2: Bipartite Match Found? <br/>(+£50 and -£50)"}:::decision
        3.3["3.3: Link via transfer_group_id"]:::process
        3.4["3.4: Flag is_internal_transfer = true"]:::process
        
        2.3 --> 3.1 --> 3.2
        3.2 -- "Yes" --> 3.3 --> 3.4
        3.2 -- "No" --> 4.1
        3.4 --> 4.1
    end

    %% PHASE 4
    subgraph P4 ["4. Hybrid AI Enrichment"]
        4.1{"4.1: Match local rules.json? <br/>(Tier 1 Regex)"}:::decision
        4.2["4.2: Assign Tier 1 Category (Zero Cost)"]:::process
        4.3["4.3: Batch unclassified into array of 50"]:::process
        4.4["4.4: Inject Strict JSON Schema"]:::process
        4.5["4.5: HTTP POST to Gemini Flash API"]:::api
        4.6{"4.6: HTTP 429 Rate Limit?"}:::decision
        4.7["4.7: Apply Exponential Backoff Retry"]:::process
        4.8["4.8: Parse JSON Response to Category ID"]:::process

        4.1 -- "Match Found" --> 4.2
        4.1 -- "No Match" --> 4.3 --> 4.4 --> 4.5 --> 4.6
        4.6 -- "Yes" --> 4.7 --> 4.5
        4.6 -- "No (HTTP 200)" --> 4.8
    end

    %% PHASE 5
    subgraph P5 ["5. Persistence Layer"]
        5.1["5.1: Map to ProcessedTransaction Entity"]:::process
        5.2["5.2: Execute JDBC Batch Insert"]:::db
        5.3["5.3: Move raw files to /archive directory"]:::process
        
        4.2 --> 5.1
        4.8 --> 5.1
        5.1 --> 5.2 --> 5.3
    end