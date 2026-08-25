# UML Class Diagram (Domain Logic)

This diagram outlines the core Java interfaces, abstractions, and entities driving the FinParse ETL engine.

```mermaid
classDiagram
    %% Interfaces
    class StatementParser {
        <<interface>>
        +parse(File file) List~RawTransaction~
        +supports(File file) boolean
    }
    
    class Categorizer {
        <<interface>>
        +categorize(List~ProcessedTransaction~ txns) void
    }

    %% Parsers (Strategy Pattern)
    class MonzoCsvParser {
        -CSVReader reader
        +parse(File file) List~RawTransaction~
    }
    class BarclaysPdfParser {
        -PDFTextStripper stripper
        +parse(File file) List~RawTransaction~
    }
    
    StatementParser <|.. MonzoCsvParser
    StatementParser <|.. BarclaysPdfParser

    %% Core Engine
    class NormalizationService {
        +normalizeDate(String date) LocalDate
        +cleanAmount(String amount) BigDecimal
    }
    class TransferReconciliationEngine {
        -SqliteRepository repository
        +reconcileTransfers(List~ProcessedTransaction~ txns)
        -findMatchingPair(ProcessedTransaction txn) Optional~ProcessedTransaction~
    }

    %% Enrichment (Chain of Responsibility)
    class RuleCategorizer {
        -Map~String, String~ regexRules
        +categorize(List~ProcessedTransaction~ txns)
    }
    class LlmCategorizer {
        -HttpClient httpClient
        -String geminiApiKey
        +categorize(List~ProcessedTransaction~ txns)
        -buildJsonPayload(List txns) String
    }

    Categorizer <|.. RuleCategorizer
    Categorizer <|.. LlmCategorizer

    %% Data Transfer Objects / Entities
    class RawTransaction {
        +String rawDate
        +String rawDescription
        +String rawAmount
    }
    
    class ProcessedTransaction {
        +String id
        +LocalDate transactionDate
        +String cleanDescription
        +BigDecimal amount
        +boolean isInternalTransfer
        +String transferGroupId
        +Integer categoryId
    }

    %% Relationships
    MonzoCsvParser ..> RawTransaction : creates
    NormalizationService ..> ProcessedTransaction : creates from Raw
    TransferReconciliationEngine --> ProcessedTransaction : modifies
    RuleCategorizer --> ProcessedTransaction : modifies