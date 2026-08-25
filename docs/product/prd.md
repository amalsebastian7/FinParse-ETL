# 📄 Product Requirements Document (PRD)

## 1. Executive Summary
**FinParse ETL (StatementProcessor)** is a local-first, privacy-centric data pipeline built in Java. It automates the ingestion of raw financial statements (CSV/PDF), normalizes the data, intelligently deduplicates internal account transfers, and uses a hybrid rule-based/LLM engine to categorize transactions for personal financial analytics.

## 2. Problem Statement
Personal finance management currently forces users into a binary choice:
1. Pay a monthly SaaS fee (e.g., YNAB, Monarch) and hand over highly sensitive Open Banking access/PII to third parties.
2. Manually manage spreadsheets, which is error-prone, time-consuming, and difficult to maintain across multiple bank formats.

**Solution:** A zero-cost, fully automated CLI pipeline that processes bulk local files, guarantees PII privacy, and utilizes cheap, stateless LLM APIs (Gemini) for highly accurate transaction categorization.

## 3. Objectives & Key Results (OKRs)
*   **O1: Seamless Ingestion**
    *   *KR1:* Support OpenCSV parsing for at least 3 major UK bank formats (e.g., Monzo, Barclays, Starling) by v1.0.
    *   *KR2:* Extract tabular data from standard PDF statements with 99% accuracy.
*   **O2: Intelligent Reconciliation**
    *   *KR1:* Automatically identify and link 100% of internal transfers between tracked accounts within a 48-hour window.
*   **O3: Cost-Effective Categorization**
    *   *KR1:* Classify >80% of transactions using zero-cost local Regex rules.
    *   *KR2:* Keep LLM API costs under £0.05 per 1,000 unclassified transactions using batching.

## 4. User Stories & Acceptance Criteria (BDD Format)

### Epic 1: Data Ingestion
**User Story 1.1:** As a user, I want the system to parse CSV files from different banks so I don't have to format them manually.
*   **Given** a Monzo CSV and a Barclays CSV in the target directory,
*   **When** the pipeline executes,
*   **Then** both files are parsed into a unified `RawTransaction` DTO,
*   **And** the original files are marked as `PROCESSED`.

### Epic 2: Deduplication (Internal Transfers)
**User Story 2.1:** As a user, I want transfers between my own accounts to cancel out so my net expenses are accurate.
*   **Given** Account A has a `-£500.00` transaction on Oct 1st,
*   **And** Account B has a `+£500.00` transaction on Oct 2nd,
*   **When** the Deduplication Engine runs,
*   **Then** both transactions are flagged with `is_internal_transfer = true`,
*   **And** they are assigned the same `transfer_group_id`.

## 5. Non-Functional Requirements (NFRs)
*   **Security:** No raw bank statements or running balances may be transmitted over the internet.
*   **Performance:** The system must process 5 years of historical data (approx. 10,000 transactions) in under 60 seconds (excluding API wait times).
*   **Portability:** Must run on any machine with JRE 21+ (Windows, macOS, Linux, Raspberry Pi).
