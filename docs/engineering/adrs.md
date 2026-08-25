# 📜 Architecture Decision Records (ADRs)

### ADR-001: Local Embedded Database (SQLite) vs Cloud/Docker Database (Postgres)
**Date:** 2023-10-25 | **Status:** Accepted
*   **Context:** We need to store thousands of processed transactions and query them efficiently.
*   **Decision:** We will use SQLite via the `sqlite-jdbc` driver.
*   **Consequences:** 
    *   *Positive:* Zero setup for users. No Docker daemon required. Data remains entirely local, eliminating network latency and securing PII.
    *   *Negative:* Lacks high concurrency for writes, which is acceptable as this is a single-threaded ETL pipeline.

### ADR-002: Hybrid Categorization (Regex + LLM)
**Date:** 2023-10-26 | **Status:** Accepted
*   **Context:** AI categorizes everything perfectly but costs money and takes time. Regex is free and instant but brittle for new merchants.
*   **Decision:** Implement a Chain of Responsibility. Tier 1 evaluates against a local `rules.json`. Only failures fall through to Tier 2 (Gemini API).
*   **Consequences:** Reduces API volume by ~80%, keeping costs negligible while maintaining high accuracy for edge cases.
