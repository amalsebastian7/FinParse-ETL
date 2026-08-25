# 🗺️ Product Vision & Roadmap

## Product Vision
To build the definitive open-source, privacy-first ETL engine for personal finance, enabling users to own their financial data warehouse without recurring SaaS fees.

## Phase 1: The Foundation (MVP) - *Current*
**Focus:** Core ingestion, mathematical accuracy, and CLI execution.
*   [x] Project scaffolding (Java 21, Maven).
*   [ ] Ingestion Layer: OpenCSV mappers for top 3 banks.
*   [ ] Ingestion Layer: Apache PDFBox extraction for legacy statements.
*   [ ] DB Layer: SQLite embedded database setup and Flyway migrations.
*   [ ] Engine: Deterministic Internal Transfer Deduplication.
*   [ ] Engine: Tier 1 Regex Categorizer (`rules.json`).

## Phase 2: AI & Analytics - *Next*
**Focus:** Smart categorization and data export.
*   [ ] Integration: Gemini API HTTP Client with JSON Schema enforcement.
*   [ ] Engine: Tier 2 Batch LLM Categorization.
*   [ ] Exporter: Dynamic CSV/Excel generation for Tableau/PowerBI.
*   [ ] UI: Rich CLI console output (progress bars, ASCII tables).

## Phase 3: Automation & Ecosystem - *Future*
**Focus:** True "Set and Forget" automation.
*   [ ] Daemon mode: Directory watcher (File System Events) to auto-process dropped files.
*   [ ] Open Banking API integration (read-only) via GoCardless/Nordigen free tier.
*   [ ] Lightweight local Web Dashboard (Spring Boot + React + Chart.js).
