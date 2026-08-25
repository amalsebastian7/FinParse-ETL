# User Flow & Execution Diagrams

## 1. System Execution Sequence
Since FinParse is a backend ETL pipeline, the UX is determined by terminal execution and log output.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant CLI as FinParse App
    participant FS as Local File System
    participant Engine as Processing Engine
    participant API as Gemini LLM
    participant DB as SQLite DB

    User->>CLI: run `java -jar finparse.jar --dir=/data`
    CLI->>FS: Scan for .csv and .pdf
    FS-->>CLI: Return 5 new statements
    
    rect rgb(200, 220, 240)
        note right of CLI: Phase 1: Extract & Load Raw
        CLI->>Engine: Parse files to DTOs
        Engine->>DB: Insert RawTransactions
    end
    
    rect rgb(220, 240, 200)
        note right of CLI: Phase 2: Clean & Deduplicate
        Engine->>DB: Query cross-account transactions (48hr window)
        Engine->>Engine: Match internal transfers
        Engine->>DB: Update is_internal_transfer = true
    end

    rect rgb(240, 220, 200)
        note right of CLI: Phase 3: Categorize
        Engine->>Engine: Apply Regex Rules (Tier 1)
        Engine->>API: Batch remaining to Gemini (Tier 2)
        API-->>Engine: Return strictly typed JSON categories
        Engine->>DB: Update Category IDs
    end
    
    CLI-->>User: Processed successfully (Print summary table)
```

## 2. Exception Handling Flow
*   **Missing API Key:** If `.env` lacks `GEMINI_API_KEY`, the pipeline completes Tier 1 (Regex) and gracefully skips Tier 2, marking remaining as `UNCATEGORIZED`.
*   **Database Lock:** If SQLite is locked (e.g., user has it open in DB Browser), CLI immediately throws a fast-fail error to prevent data corruption.
