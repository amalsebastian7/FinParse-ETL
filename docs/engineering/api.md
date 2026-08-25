# 🔌 API & LLM Integration Specification

## 1. Google Gemini 1.5/2.0 Flash Integration
The application acts as a client to the Gemini REST API. We use "Flash" because it is highly cost-effective and extremely fast for classification tasks.

## 2. Strict JSON Schema (Structured Outputs)
To guarantee the LLM does not hallucinate categories (e.g., returning "Food" instead of "Groceries"), we enforce a JSON schema in the API request payload.

### HTTP POST Request
**Endpoint:** `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key={API_KEY}`
**Headers:** `Content-Type: application/json`

**Body (Template):**
```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "You are a financial categorizer. Map the following JSON array of transactions to the exact categories provided in the schema. Return ONLY a valid JSON array."
        },
        {
          "text": "[{\"id\": \"123\", \"desc\": \"AMZN Mktp UK\"}, {\"id\": \"124\", \"desc\": \"TfL Travel Charge\"}]"
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.0,
    "responseMimeType": "application/json",
    "responseSchema": {
      "type": "ARRAY",
      "items": {
        "type": "OBJECT",
        "properties": {
          "id": { "type": "STRING" },
          "category": {
            "type": "STRING",
            "enum": ["Groceries", "Transport", "Shopping", "Dining", "Utilities"]
          }
        },
        "required": ["id", "category"]
      }
    }
  }
}
```

## 3. Resilience & Batching
*   **Batch Size:** Transactions are chunked into arrays of `50` per HTTP request.
*   **Retry Policy:** If `HTTP 429 (Too Many Requests)` is encountered, the thread sleeps using Exponential Backoff: `2s, 4s, 8s, 16s` before aborting.
