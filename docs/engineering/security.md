# 🛡️ Security & Threat Model

## 1. Core Security Tenet
**Zero-Cloud PII:** Financial data represents extreme PII. The SQLite database, raw CSVs, and PDFs must NEVER leave the host machine. 

## 2. Threat Model (STRIDE)

| Threat Type | Risk | Mitigation Architecture |
| :--- | :--- | :--- |
| **Spoofing** | Unauthorized execution | CLI operates locally under the OS user's privileges. No web server exposed. |
| **Tampering** | Modification of DB | SQLite relies on host OS file permissions (`chmod 600`). |
| **Repudiation** | Loss of original data | Pipeline is non-destructive. Raw files are moved to an `/archive` folder, never deleted. |
| **Information Disclosure** | PII leaked to Google Gemini | Implementation of a **Sanitization Filter** (see Section 3). |
| **Denial of Service** | OOM from massive PDFs | Apache PDFBox is configured to use memory-mapped files (stream processing) rather than loading entirely into RAM. |

## 3. PII Sanitization Pipeline (Pre-LLM)
Before a transaction is sent to the Gemini API, the `DataSanitizer` class performs the following regex replacements to protect privacy:
1.  **Account Numbers / Sort Codes:** Replaced with `[REDACTED_ACCOUNT]`.
2.  **Specific Balances:** Dropped from the payload entirely.
3.  **Names (Wire Transfers):** Tier 1 Engine detects names via bank format rules and flags them locally as `INTERNAL_TRANSFER` or `WIRE_TRANSFER`, skipping the LLM tier entirely.
