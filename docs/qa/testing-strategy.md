# QA & Testing Strategy

## 1. Testing Philosophy
Testing financial logic requires mathematical absolute precision. Our test suite will enforce a **minimum 85% JaCoCo Line Coverage**, focusing heavily on the domain engine.

## 2. The "Dummy Data" Strategy
Real statements will NEVER be committed to the repository. 
We utilize a `SyntheticStatementGenerator.java` utility in the `src/test/java` directory to generate fake CSV and PDF files during the Maven `test` phase.

## 3. Core Test Scenarios (Examples)

### 3.1 Unit Testing: Internal Transfer Reconciliation
**Class:** `TransferReconciliationEngineTest.java`
```java
@Test
@DisplayName("Should link Account A (-£50) and Account B (+£50) if within 48 hours")
void testValidInternalTransfer() {
    // Given
    Transaction out = new Transaction("AccA", LocalDate.of(2023, 10, 1), new BigDecimal("-50.00"));
    Transaction in = new Transaction("AccB", LocalDate.of(2023, 10, 2), new BigDecimal("50.00"));
    
    // When
    engine.reconcile(List.of(out, in));
    
    // Then
    assertTrue(out.isInternalTransfer());
    assertTrue(in.isInternalTransfer());
    assertEquals(out.getTransferGroupId(), in.getTransferGroupId());
}

@Test
@DisplayName("Should NOT link identical amounts if outside the 48 hour settlement window")
void testInvalidTransferTimeWindow() {
    // Given
    Transaction out = new Transaction("AccA", LocalDate.of(2023, 10, 1), new BigDecimal("-50.00"));
    Transaction in = new Transaction("AccB", LocalDate.of(2023, 10, 5), new BigDecimal("50.00"));
    
    // When
    engine.reconcile(List.of(out, in));
    
    // Then
    assertFalse(out.isInternalTransfer());
    assertFalse(in.isInternalTransfer());
}
```

### 3.2 Integration Testing: Database Layer
Uses **Testcontainers** (or an in-memory SQLite URL `jdbc:sqlite::memory:`) to verify that Flyway migrations execute correctly and constraints (like foreign keys to categories) are enforced.

### 3.3 Mocking the LLM
We use **WireMock** to intercept HTTP calls to `generativelanguage.googleapis.com` during testing. We inject mock JSON responses to ensure our `Jackson` deserializers correctly map the LLM output back into our DTOs without actually spending API credits.
