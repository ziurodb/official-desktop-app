# ZiuroDB Query Language (ZQL) — Complete Command Reference

Welcome to the official documentation for **ZiuroDB Query Language (ZQL)**. ZQL is a universal, database-agnostic query language engineered to communicate seamlessly across multiple database systems (**MongoDB**, **MySQL**, **PostgreSQL**, and future engines) using a unified syntax.

---

## ZQL Unified Data Model

ZQL abstracts relational database tables and document database collections into a standardized hierarchical tree structure:

```
Database
    └── Tree (Table / Collection)
            └── Branch (Row / Document)
                    └── Leaf (Field / Column / Value)
```

### Mapping Comparison Table

| ZQL Concept | SQL Database (MySQL / PostgreSQL) | NoSQL Database (MongoDB / Firebase) |
| :--- | :--- | :--- |
| **Database** | Database | Database |
| **Tree** | Table | Collection |
| **Branch** | Row | Document |
| **Leaf** | Column / Field Value | Document Key / Field |

---

## 1. Database Operations

### `SHOW DATABASES;`
- **Definition**: Lists all accessible databases on the active connection.
- **Example**:
  ```zql
  SHOW DATABASES;
  ```

### `CREATE DATABASE`
- **Definition**: Creates a new database instance.
- **Syntax**: `CREATE DATABASE database_name;`
- **Example**:
  ```zql
  CREATE DATABASE production_db;
  ```

### `ZQL DATABASE = ...`
- **Definition**: Sets the active working database context for subsequent queries.
- **Syntax**: `ZQL DATABASE = "database_name";`
- **Example**:
  ```zql
  ZQL DATABASE = "analytics_db";
  ```

### `DATABASE.DELETE();`
- **Definition**: Drops the currently selected active database.
- **Example**:
  ```zql
  DATABASE.DELETE();
  ```

---

## 2. Tree Commands (Tables / Collections)

### `DATABASE.TREES();`
- **Definition**: Lists all tables (SQL) or collections (MongoDB) inside the active database.
- **Example**:
  ```zql
  DATABASE.TREES();
  ```

### `ZQL TREE = ...`
- **Definition**: Selects a default active Tree for rapid querying.
- **Syntax**: `ZQL TREE = DATABASE.TREE("tree_name");`
- **Example**:
  ```zql
  ZQL TREE = DATABASE.TREE("users");
  ```

### `DATABASE.TREE("tree_name").STRUCTURE();`
- **Definition**: Inspects column schemas, data types, and nullability specifications of a Tree.
- **Example**:
  ```zql
  DATABASE.TREE("users").STRUCTURE();
  ```

### `DATABASE.CREATE_TREE(...)`
- **Definition**: Creates a new table or collection with defined fields and data types.
- **Syntax**: `DATABASE.CREATE_TREE("tree_name", { field1: type, field2: type });`
- **Example**:
  ```zql
  DATABASE.CREATE_TREE("users", {
    id: number,
    name: string,
    email: string
  });
  ```

### `DATABASE.DELETE_TREE(...)`
- **Definition**: Permanently drops a table or collection.
- **Syntax**: `DATABASE.DELETE_TREE("tree_name");`
- **Example**:
  ```zql
  DATABASE.DELETE_TREE("temp_logs");
  ```

---

## 3. Branch Operations (Rows / Documents)

### `DATABASE.TREE("tree_name").FIND()`
- **Definition**: Retrieves all branches (rows/documents) from a Tree.
- **Example**:
  ```zql
  DATABASE.TREE("users").FIND();
  ```

### `DATABASE.TREE("tree_name").FIND_ONE(...)`
- **Definition**: Retrieves a single matching branch from a Tree.
- **Syntax**: `DATABASE.TREE("tree_name").FIND_ONE({ filter });`
- **Example**:
  ```zql
  DATABASE.TREE("users").FIND_ONE({ id: 1 });
  ```

### `DATABASE.TREE("tree_name").FIND_BRANCH(...)`
- **Definition**: Queries branches matching specific condition filters.
- **Syntax**: `DATABASE.TREE("tree_name").FIND_BRANCH({ filter });`
- **Example**:
  ```zql
  DATABASE.TREE("users").FIND_BRANCH({ age > 18 });
  ```

### Projection Queries
- **Definition**: Selects specific leaves (fields) to return in query results.
- **Syntax**: `DATABASE.TREE("tree_name").FIND({ filter }, { field1, field2 });`
- **Example**:
  ```zql
  DATABASE.TREE("users").FIND({ age > 18 }, { name, email });
  ```

### `ADD_BRANCH` & `ADD_BRANCHES`
- **Definition**: Inserts a single branch or multiple branches into a Tree.
- **Single Insert Example**:
  ```zql
  DATABASE.TREE("users").ADD_BRANCH({
    id: 1,
    name: "Shani",
    email: "shani@ziurodb.com"
  });
  ```
- **Multiple Insert Example**:
  ```zql
  DATABASE.TREE("users").ADD_BRANCHES([
    { id: 1, name: "Shani" },
    { id: 2, name: "Alex" }
  ]);
  ```

### `UPDATE_BRANCH` & `UPDATE_BRANCHES`
- **Definition**: Updates field values for a single matching branch or multiple branches.
- **Single Update Example**:
  ```zql
  DATABASE.TREE("users").UPDATE_BRANCH({ id: 1 }, { name: "Shani Kumar" });
  ```
- **Bulk Update Example**:
  ```zql
  DATABASE.TREE("users").UPDATE_BRANCHES({ age > 18 }, { status: "active" });
  ```

### `DELETE_BRANCH` & `DELETE_BRANCHES`
- **Definition**: Removes a single branch or multiple branches matching query criteria.
- **Single Delete Example**:
  ```zql
  DATABASE.TREE("users").DELETE_BRANCH({ id: 1 });
  ```
- **Bulk Delete Example**:
  ```zql
  DATABASE.TREE("users").DELETE_BRANCHES({ age < 18 });
  ```

---

## 4. Leaf Commands (Field / Value Operations)

### `ADD_LEAF`
- **Definition**: Adds a new leaf (field/column) value to a specific branch.
- **Syntax**: `tree.BRANCH(id).ADD_LEAF({ field: value });`
- **Example**:
  ```zql
  users.BRANCH(1).ADD_LEAF({ city: "Mumbai" });
  ```

### `UPDATE_LEAF`
- **Definition**: Modifies an existing leaf value on a target branch.
- **Syntax**: `tree.BRANCH(id).UPDATE_LEAF({ field: value });`
- **Example**:
  ```zql
  users.BRANCH(1).UPDATE_LEAF({ age: 24 });
  ```

### `DELETE_LEAF`
- **Definition**: Removes a leaf (field/column) from a target branch.
- **Syntax**: `tree.BRANCH(id).DELETE_LEAF({ field });`
- **Example**:
  ```zql
  users.BRANCH(1).DELETE_LEAF({ age });
  ```

---

## 5. Operators, Sorting & Pagination

### Comparison Operators

| Operator | Meaning | Example |
| :--- | :--- | :--- |
| `=` | Equal to | `{ status: "active" }` |
| `!=` | Not equal to | `{ status: "disabled" }` |
| `>` | Greater than | `{ age > 18 }` |
| `<` | Less than | `{ age < 60 }` |
| `>=` | Greater than or equal | `{ score >= 75 }` |
| `<=` | Less than or equal | `{ price <= 100 }` |
| `IN` | Value in array | `{ role IN ["admin", "editor"] }` |
| `LIKE` | Pattern matching | `{ name LIKE "%Shani%" }` |

### Sorting (`.SORT(...)`)
- **Definition**: Sorts result branches in ascending (`ASC`) or descending (`DESC`) order.
- **Examples**:
  ```zql
  DATABASE.TREE("users").FIND().SORT({ age: ASC });
  DATABASE.TREE("users").FIND().SORT({ age: DESC });
  ```

### Pagination (`.LIMIT(...)` & `.SKIP(...)`)
- **Definition**: Limits total returned branches or skips an offset number of records.
- **Examples**:
  ```zql
  DATABASE.TREE("users").FIND().LIMIT(10);
  DATABASE.TREE("users").FIND().SKIP(20).LIMIT(10);
  ```

---

## 6. Index Management Operations

### `CREATE_INDEX`
- **Definition**: Builds an index on specified fields to accelerate search performance.
- **Example**:
  ```zql
  DATABASE.TREE("users").CREATE_INDEX({ email });
  ```

### `DELETE_INDEX`
- **Definition**: Drops an existing index from a Tree.
- **Example**:
  ```zql
  DATABASE.TREE("users").DELETE_INDEX({ email });
  ```

---

## 7. Transaction Statements

### `BEGIN;`
- **Definition**: Starts a new transactional session block.
- **Example**:
  ```zql
  BEGIN;
  ```

### `COMMIT;`
- **Definition**: Permanently commits all pending mutations in the current transaction block.
- **Example**:
  ```zql
  COMMIT;
  ```

### `ROLLBACK;`
- **Definition**: Undoes and rolls back all mutations in the current transaction block.
- **Example**:
  ```zql
  ROLLBACK;
  ```

---

## 8. Database Version Control Readiness

Every state-mutating ZQL operation (`ADD_BRANCH`, `UPDATE_BRANCH`, `DELETE_BRANCH`, `ADD_LEAF`, `UPDATE_LEAF`, `DELETE_LEAF`, `CREATE_TREE`, `DELETE_TREE`) automatically emits a structured `MutationEvent` payload:

```json
{
  "eventId": "evt_1785091200000_a1b2c",
  "operation": "UPDATE_BRANCH",
  "database": "production_db",
  "tree": "users",
  "branchId": 1,
  "userId": "user_66b1a23f",
  "before": { "name": "Shani" },
  "after": { "name": "Shani Kumar" },
  "timestamp": "2026-07-27T01:35:00.000Z"
}
```

This mutation stream powers future Git-like database version control commands:
- `DATABASE.COMMIT()`
- `DATABASE.HISTORY()`
- `DATABASE.DIFF()`
- `DATABASE.CHECKOUT()`
