# ziuroDB Public API v2.0 — Complete Documentation

The ziuroDB Public API Engine lets you expose your database connections as **production-grade REST endpoints**. Build custom frontends, dashboards, mobile apps, and backend integrations - all powered by secure, structured API responses with auto-parsed data types.

---

## Base URL & Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/public/query` | POST | Execute a single query |
| `/api/v1/public/batch` | POST | Execute multiple queries in one request |
| `/api/v1/public/transaction` | POST | Execute queries in a database transaction (SQL only) |
| `/api/v1/public/docs` | GET | Self-documenting API reference |

**Base URL (local):** `http://localhost:5001/api/v1/public`
**Base URL (cloud):** `https://your-backend.onrender.com/api/v1/public`

---

## 🔐 Authentication

All requests must include your API key via one of these methods:

```
X-API-Key: pdb_your_api_key_here
```

or

```
Authorization: Bearer pdb_your_api_key_here
```

---

## 📊 Response Format (v2.0)

### Default: Document Format

By default, all responses return **clean JSON document objects** with auto-parsed data types — no more raw column/row arrays.

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Rahul Singh",
      "email": "rahul@test.com",
      "age": 28,
      "role": "developer",
      "salary": 55000,
      "isActive": true,
      "skills": ["Java", "PostgreSQL"],
      "address": {
        "city": "Mumbai",
        "country": "India"
      },
      "createdAt": "2026-03-03T15:41:18.074Z"
    }
  ],
  "meta": {
    "requestId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "executionTimeMs": 12,
    "databaseType": "mysql",
    "rowCount": 1,
    "format": "documents",
    "timestamp": "2026-05-15T17:30:00.000Z"
  }
}
```

### Legacy: Raw Format

Add `"format": "raw"` to your request body to get the classic column+row format:

```json
{
  "success": true,
  "data": {
    "columns": ["id", "name", "email"],
    "rows": [[1, "Rahul Singh", "rahul@test.com"]]
  },
  "meta": { "format": "raw", "..." : "..." }
}
```

### Auto-Parsed Data Types

The API automatically detects and converts:

| Input (raw string) | Output (parsed) | Type |
|---------------------|-----------------|------|
| `"42"` | `42` | number |
| `"3.14"` | `3.14` | number |
| `"true"` / `"false"` | `true` / `false` | boolean |
| `"null"` | `null` | null |
| `'{"city":"Mumbai"}'` | `{"city":"Mumbai"}` | object |
| `'["Java","Go"]'` | `["Java","Go"]` | array |
| `"2026-03-03T15:41:18Z"` | `"2026-03-03T15:41:18.000Z"` | ISO date string |

---

## ❌ Error Format

All errors return structured, developer-friendly responses : 

```json
{
  "success": false,
  "error": {
    "type": "QUERY_ERROR",
    "message": "Unknown column 'emails' in 'field list'",
    "database": "MySQL",
    "query": "SELECT emails FROM users",
    "code": "ER_BAD_FIELD_ERROR",
    "position": { "line": 1, "column": 8 }
  },
  "meta": {
    "requestId": "a1b2c3d4-...",
    "timestamp": "2026-05-15T17:30:00.000Z",
    "statusCode": 400
  }
}
```

### Error Types

| Type | HTTP Code | Description |
|------|-----------|-------------|
| `VALIDATION_ERROR` | 400 | Invalid request body or query format |
| `QUERY_ERROR` | 400 | SQL/MongoDB syntax or execution error |
| `AUTH_ERROR` | 401 | Invalid or missing API key |
| `PERMISSION_ERROR` | 403 | Write operation on read-only connection |
| `RATE_LIMIT_ERROR` | 429 | Rate limit exceeded |
| `CONNECTION_ERROR` | 500 | Cannot reach the database server |
| `TIMEOUT_ERROR` | 408 | Query execution timed out |
| `TRANSACTION_ERROR` | 400 | Transaction failed (auto-rolled back) |
| `TRANSACTION_NOT_SUPPORTED` | 400 | MongoDB without replica set |

---

## 🛠 Endpoint 1: Single Query

**`POST /api/v1/public/query`**

### Request Body

```json
{
  "connectionId": "your_connection_id",
  "dbName": "your_database_name",
  "query": "SELECT * FROM users WHERE age > 25 LIMIT 10",
  "format": "documents",
  "page": 1,
  "pageSize": 20
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `connectionId` | string | ✅ | Connection ID from ziuroDB dashboard |
| `dbName` | string | ✅ | Target database name |
| `query` | string | ✅ | SQL query or MongoDB command |
| `format` | string | ❌ | `"documents"` (default) or `"raw"` |
| `page` | number | ❌ | Page number for pagination metadata |
| `pageSize` | number | ❌ | Page size for pagination metadata |

### cURL Example

```bash
curl -X POST "http://localhost:5001/api/v1/public/query" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pdb_your_api_key_here" \
  -d '{
    "connectionId": "683f1a2b4c5d6e7f8a9b0c1d",
    "dbName": "myapp",
    "query": "SELECT id, name, email, age FROM users LIMIT 5"
  }'
```

### Response

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "Rahul Singh", "email": "rahul@test.com", "age": 28 },
    { "id": 2, "name": "Priya Patel", "email": "priya@test.com", "age": 24 }
  ],
  "meta": {
    "requestId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "executionTimeMs": 8,
    "databaseType": "mysql",
    "rowCount": 2,
    "format": "documents",
    "timestamp": "2026-05-15T17:30:00.000Z"
  }
}
```

---

## 🛠 Endpoint 2: Batch Queries

**`POST /api/v1/public/batch`**

Execute up to **20 queries** sequentially in a single HTTP request. Each query runs independently with its own timing and error tracking.

### Request Body

```json
{
  "connectionId": "your_connection_id",
  "dbName": "myapp",
  "queries": [
    { "query": "SELECT * FROM users LIMIT 5", "id": "users" },
    { "query": "SELECT COUNT(*) as total FROM orders", "id": "orderCount" },
    { "query": "SELECT name, price FROM products WHERE price > 100", "id": "expensiveProducts" }
  ],
  "stopOnError": true
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `connectionId` | string | ✅ | Connection ID |
| `dbName` | string | ✅ | Database name |
| `queries` | array | ✅ | Array of `{ query, id }` objects (max 20) |
| `stopOnError` | boolean | ❌ | Stop on first error (default: `true`) |

### cURL Example

```bash
curl -X POST "http://localhost:5001/api/v1/public/batch" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pdb_your_api_key_here" \
  -d '{
    "connectionId": "683f1a2b4c5d6e7f8a9b0c1d",
    "dbName": "myapp",
    "queries": [
      { "query": "SELECT * FROM users LIMIT 3", "id": "users" },
      { "query": "SELECT COUNT(*) as total FROM orders", "id": "orders" }
    ]
  }'
```

### Response

```json
{
  "success": true,
  "results": [
    {
      "id": "users",
      "success": true,
      "data": [
        { "id": 1, "name": "Rahul", "email": "rahul@test.com" },
        { "id": 2, "name": "Priya", "email": "priya@test.com" }
      ],
      "meta": { "executionTimeMs": 5, "rowCount": 2 }
    },
    {
      "id": "orders",
      "success": true,
      "data": [{ "total": 1547 }],
      "meta": { "executionTimeMs": 3, "rowCount": 1 }
    }
  ],
  "meta": {
    "requestId": "b2c3d4e5-...",
    "totalExecutionTimeMs": 12,
    "queriesExecuted": 2,
    "queriesFailed": 0,
    "timestamp": "2026-05-15T17:30:00.000Z"
  }
}
```

---

## 🛠 Endpoint 3: Transactions (SQL Only)

**`POST /api/v1/public/transaction`**

Execute multiple SQL queries inside a database transaction. If any query fails, **all changes are automatically rolled back**.

> ⚠️ **MongoDB Note:** Transactions require a replica set deployment. Standalone MongoDB instances will receive a `TRANSACTION_NOT_SUPPORTED` error.

### Request Body

```json
{
  "connectionId": "your_connection_id",
  "dbName": "myapp",
  "queries": [
    "INSERT INTO orders (user_id, product_id, quantity) VALUES (1, 42, 2)",
    "UPDATE products SET stock = stock - 2 WHERE id = 42",
    "UPDATE users SET total_orders = total_orders + 1 WHERE id = 1"
  ],
  "isolation": "READ COMMITTED"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `connectionId` | string | ✅ | Connection ID |
| `dbName` | string | ✅ | Database name |
| `queries` | string[] | ✅ | Array of SQL query strings (max 50) |
| `isolation` | string | ❌ | Transaction isolation level |

**Isolation Levels:** `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`

### cURL Example

```bash
curl -X POST "http://localhost:5001/api/v1/public/transaction" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pdb_your_api_key_here" \
  -d '{
    "connectionId": "683f1a2b4c5d6e7f8a9b0c1d",
    "dbName": "myapp",
    "queries": [
      "INSERT INTO orders (user_id, amount) VALUES (1, 299.99)",
      "UPDATE users SET balance = balance - 299.99 WHERE id = 1"
    ],
    "isolation": "SERIALIZABLE"
  }'
```

### Success Response

```json
{
  "success": true,
  "transaction": "committed",
  "results": [
    { "index": 0, "success": true, "data": [], "meta": { "..." : "..." } },
    { "index": 1, "success": true, "data": [], "meta": { "..." : "..." } }
  ],
  "meta": {
    "requestId": "c3d4e5f6-...",
    "totalExecutionTimeMs": 45,
    "queriesExecuted": 2,
    "isolation": "SERIALIZABLE",
    "timestamp": "2026-05-15T17:30:00.000Z"
  }
}
```

### Failure Response (Auto-Rollback)

```json
{
  "success": false,
  "error": {
    "type": "TRANSACTION_ERROR",
    "message": "Transaction failed: Insufficient balance. All changes have been rolled back.",
    "database": "mysql",
    "code": "ER_CHECK_CONSTRAINT_VIOLATED"
  },
  "meta": { "requestId": "...", "statusCode": 400 }
}
```

---

## 💻 Integration Examples

### 1. React + Axios (Recommended)

```jsx
import axios from "axios";

// Create a reusable API client
const ziuroDB = axios.create({
  baseURL: "http://localhost:5001/api/v1/public",
  headers: {
    "Content-Type": "application/json",
    "X-API-Key": process.env.REACT_APP_ZIURODB_KEY,
  },
});

// ── Single Query ────────────────────────────────
export async function getUsers() {
  const { data } = await ziuroDB.post("/query", {
    connectionId: "683f1a2b4c5d6e7f8a9b0c1d",
    dbName: "myapp",
    query: "SELECT * FROM users WHERE isActive = true LIMIT 20",
  });

  if (data.success) {
    // data.data is already an array of clean objects!
    return data.data;
    // → [{ id: 1, name: "Rahul", isActive: true, skills: ["Java","Go"] }, ...]
  }
  throw new Error(data.error?.message || "Query failed");
}

// ── Batch Query (Dashboard) ─────────────────────
export async function getDashboardData() {
  const { data } = await ziuroDB.post("/batch", {
    connectionId: "683f1a2b4c5d6e7f8a9b0c1d",
    dbName: "myapp",
    queries: [
      { query: "SELECT COUNT(*) as total FROM users", id: "userCount" },
      { query: "SELECT COUNT(*) as total FROM orders WHERE status='pending'", id: "pendingOrders" },
      { query: "SELECT SUM(amount) as revenue FROM payments WHERE MONTH(created_at) = MONTH(NOW())", id: "monthlyRevenue" },
    ],
  });

  // Access each result by its ID
  const userCount = data.results.find(r => r.id === "userCount")?.data[0]?.total;
  const pendingOrders = data.results.find(r => r.id === "pendingOrders")?.data[0]?.total;
  const revenue = data.results.find(r => r.id === "monthlyRevenue")?.data[0]?.revenue;

  return { userCount, pendingOrders, revenue };
}

// ── Transaction (Order Placement) ───────────────
export async function placeOrder(userId, productId, quantity, price) {
  const { data } = await ziuroDB.post("/transaction", {
    connectionId: "683f1a2b4c5d6e7f8a9b0c1d",
    dbName: "myapp",
    queries: [
      `INSERT INTO orders (user_id, product_id, quantity, total) VALUES (${userId}, ${productId}, ${quantity}, ${price * quantity})`,
      `UPDATE products SET stock = stock - ${quantity} WHERE id = ${productId}`,
      `UPDATE users SET total_orders = total_orders + 1 WHERE id = ${userId}`,
    ],
    isolation: "READ COMMITTED",
  });

  if (data.success) {
    return { status: "committed", results: data.results };
  }
  throw new Error(data.error?.message || "Transaction failed — all changes rolled back");
}
```

### 2. React Hook (useFetch)

```jsx
import { useState, useEffect } from "react";

function useZiuroQuery(query, deps = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch("http://localhost:5001/api/v1/public/query", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "X-API-Key": "pdb_your_api_key_here",
      },
      body: JSON.stringify({
        connectionId: "YOUR_CONNECTION_ID",
        dbName: "mydb",
        query,
      }),
    })
      .then(res => res.json())
      .then(result => {
        if (result.success) {
          setData(result.data);
          setError(null);
        } else {
          setError(result.error);
        }
      })
      .catch(err => setError({ type: "NETWORK_ERROR", message: err.message }))
      .finally(() => setLoading(false));
  }, deps);

  return { data, loading, error };
}

// Usage in a component
function UserList() {
  const { data: users, loading, error } = useZiuroQuery("SELECT * FROM users LIMIT 20");

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name} — {user.email}</li>
      ))}
    </ul>
  );
}
```

### 3. Next.js API Route (Server-Side)

```javascript
// pages/api/users.js
export default async function handler(req, res) {
  const response = await fetch("http://localhost:5001/api/v1/public/query", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-API-Key": process.env.ZIURODB_API_KEY,  // Keep key server-side!
    },
    body: JSON.stringify({
      connectionId: process.env.ZIURODB_CONNECTION_ID,
      dbName: "myapp",
      query: "SELECT id, name, email FROM users WHERE isActive = true",
    }),
  });

  const result = await response.json();

  if (result.success) {
    res.status(200).json(result.data);
  } else {
    res.status(result.meta?.statusCode || 500).json({ error: result.error });
  }
}
```

### 4. Node.js / Express Backend

```javascript
const express = require("express");
const app = express();

const ZIURODB_CONFIG = {
  endpoint: "http://localhost:5001/api/v1/public/query",
  apiKey: process.env.ZIURODB_API_KEY,
  connectionId: process.env.ZIURODB_CONNECTION_ID,
  dbName: "myapp",
};

// Reusable query helper
async function queryDB(sql) {
  const res = await fetch(ZIURODB_CONFIG.endpoint, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-API-Key": ZIURODB_CONFIG.apiKey,
    },
    body: JSON.stringify({
      connectionId: ZIURODB_CONFIG.connectionId,
      dbName: ZIURODB_CONFIG.dbName,
      query: sql,
    }),
  });

  const json = await res.json();
  if (!json.success) throw new Error(json.error?.message || "Query failed");
  return json.data;
}

// API routes
app.get("/api/users", async (req, res) => {
  try {
    const users = await queryDB("SELECT * FROM users LIMIT 50");
    res.json(users);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.get("/api/dashboard", async (req, res) => {
  const response = await fetch("http://localhost:5001/api/v1/public/batch", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-API-Key": ZIURODB_CONFIG.apiKey,
    },
    body: JSON.stringify({
      connectionId: ZIURODB_CONFIG.connectionId,
      dbName: ZIURODB_CONFIG.dbName,
      queries: [
        { query: "SELECT COUNT(*) as count FROM users", id: "users" },
        { query: "SELECT COUNT(*) as count FROM products", id: "products" },
        { query: "SELECT SUM(amount) as total FROM orders", id: "revenue" },
      ],
    }),
  });

  const result = await response.json();
  res.json(result);
});

app.listen(3000);
```

### 5. Python (requests)

```python
import requests

API_URL = "http://localhost:5001/api/v1/public/query"
HEADERS = {
    "Content-Type": "application/json",
    "X-API-Key": "pdb_your_api_key_here"
}

def query_db(sql, db_name="myapp"):
    response = requests.post(API_URL, json={
        "connectionId": "683f1a2b4c5d6e7f8a9b0c1d",
        "dbName": db_name,
        "query": sql,
    }, headers=HEADERS)

    result = response.json()
    if result["success"]:
        return result["data"]  # List of dicts — ready to use!
    else:
        raise Exception(f"[{result['error']['type']}] {result['error']['message']}")

# Usage
users = query_db("SELECT * FROM users WHERE age > 25")
for user in users:
    print(f"{user['name']} ({user['email']}) — Age: {user['age']}")
```

### 6. MongoDB Query Examples

```bash
# Find all documents
curl -X POST "http://localhost:5001/api/v1/public/query" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pdb_your_key" \
  -d '{
    "connectionId": "YOUR_ID",
    "dbName": "mydb",
    "query": "db.users.find({ age: { $gt: 25 } })"
  }'

# Aggregation pipeline
curl -X POST "http://localhost:5001/api/v1/public/query" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: pdb_your_key" \
  -d '{
    "connectionId": "YOUR_ID",
    "dbName": "mydb",
    "query": "db.orders.aggregate([{ $group: { _id: \"$status\", count: { $sum: 1 } } }])"
  }'
```

---

## ⚡ Limits

| Limit | Value |
|-------|-------|
| Max rows per query | 1,000 |
| Max batch queries | 20 |
| Max transaction queries | 50 |
| Query timeout | 15 seconds |
| Rate limit | Configurable (default: 60 req/min) |

---

## 🛡 Security Best Practices

> **🔑 API Key Protection**
> - Never commit API keys to public Git repositories
> - Use `.env` files and environment variables on the server
> - For browser-only apps, create a backend proxy (Next.js API route, Express, etc.)
> - Keep your API key server-side whenever possible

> **🔒 Read-Only Mode**
> Enable "Read-Only Mode" in the ziuroDB dashboard for connections exposed to public-facing apps unless you specifically need write access.

> **🌐 IP Whitelisting**
> Configure allowed IPs in the API Engine settings to restrict access to known servers only.

---

## 📋 Quick Reference

```
┌──────────────────────────────────────────────────────────────────────┐
│  ziuroDB Public API v2.0                                              │
├──────────────────────────────────────────────────────────────────────┤
│  POST /query        → Execute single query (document JSON response)  │
│  POST /batch        → Execute multiple queries (up to 20)            │
│  POST /transaction  → SQL transaction with auto-rollback             │
│  GET  /docs         → Self-documenting API reference                 │
├──────────────────────────────────────────────────────────────────────┤
│  Auth: X-API-Key: pdb_xxx  |  Response: { success, data[], meta }    │
│  Errors: { error: { type, message, code, query, position }, meta }   │
└──────────────────────────────────────────────────────────────────────┘
```
