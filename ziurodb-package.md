# Complete Feature & API Guide for `ziurodb` Package

The **`ziurodb`** package is a universal database ORM, client SDK, and secure local agent manager supporting **MongoDB**, **MySQL**, **PostgreSQL**, **Firebase**, and **Cloud Databases**.

---

## 🌟 Exclusive & Next-Gen ZiuroDB Features

These features make `ziurodb` the most powerful and unique database ORM on the market.

### • 1. AI Natural Language Query Engine (`db.ai`)
Ask natural language questions to query your database without writing raw SQL or Mongo code:
```javascript
const topCustomers = await db.ai(
  "Show top 10 customers who purchased more than 5 times."
);
```

### • 2. Natural Language CRUD Assistant (`users.ask`)
Perform complex CRUD mutations using natural language prompts:
```javascript
await users.ask(
  "Delete inactive users older than 2 years."
);
```

### • 3. Multi-Database Seamless Cross Join (`db.join`)
Perform seamless in-memory cross-joins between datasets originating from completely different databases (e.g. MongoDB users joined with PostgreSQL orders):

```javascript
const users = await mongoUsers.list();
const orders = await postgresOrders.list();

const result = db.join(
  users,
  orders,
  "userId",  // Left key
  "userId",  // Right key
  { as: "userOrders" }
);

console.log(result[0].userOrders); // Combined dataset!
```

### • 4. Real-Time Live Change Streams & Event Listeners (`watch`, `on`)
Subscribe to live mutation events across all connected databases:

```javascript
// Watch all mutations in real time
users.watch((event) => {
  console.log(`Action: ${event.action}, Table: ${event.tableName}`);
});

// Event listener for specific action
users.on("insert", (event) => {
  console.log("New user inserted:", event.data);
});
```

### • 5. Query Caching (`cache`) & Retry Engine (`retry`)
Cache query results in memory for N seconds, or automatically retry failing database queries with backoff:

```javascript
// Cache query results for 60 seconds
const cachedUsers = await users.cache(60).find({ status: "active" });

// Retry up to 3 times on connection drops
const resilientData = await users.retry(3).find({ age: { $gt: 18 } });
```

### • 6. Schema Intelligence & Optimization (`schema.explain`, `optimize`)
Get instant AI-driven schema insights, index recommendations, and slow query warnings:

```javascript
// Explain schema structure & relationships
const schemaInfo = await db.schema.explain();
console.log(schemaInfo.optimizationSuggestions);

// Get automatic index recommendations
const recommendations = await db.optimize();
console.log(recommendations.recommendedIndexes);
```

### • 7. Database Health Metrics & Connection Pool Stats (`health`, `pool.stats`)
Monitor real-time latency, CPU/memory usage, active connections, and pool stats:

```javascript
// Health metrics
const health = await db.health();
console.log(`Latency: ${health.latencyMs}ms, Status: ${health.status}`);

// Connection pool statistics
const poolStats = await db.pool.stats();
console.log('Active Connections:', poolStats.activeConnections);
```

### • 8. Database Backup, Restore, Sync, Import & Export
```javascript
// Automated Backup
const backup = await db.backup();

// Restore from Backup
await db.restore(backup.backupFile);

// Database Sync across providers (e.g. MongoDB -> PostgreSQL)
await db.sync({ from: "mongodb", to: "postgresql" });

// Import & Export
await db.import("users.csv");
await db.export("users.json");
```

### • 9. Multi-Tenant Isolation (`tenant`) & Read Replica Support (`readReplica`)
```javascript
// Multi-Tenant Isolation
db.tenant("company_a");

// Route reads to Read Replica
db.readReplica("https://replica.ziurodb.com");
```

---

## 🛠️ Developer Experience (DX) & CLI Tools

### • CLI Commands Reference
```bash
npx ziurodb init        # Initialize ziurodb.config.js, migrations/, & seeders/
npx ziurodb login       # Authenticate user and save credentials locally
npx ziurodb generate    # Generate TypeScript interfaces & models from database
npx ziurodb pull        # Pull database schema into local TypeScript models
npx ziurodb push        # Push local schema/migrations to database
npx ziurodb migrate     # Run pending migration scripts
npx ziurodb rollback    # Rollback last migration step
npx ziurodb seed        # Run seeder scripts to populate dev data
npx ziurodb studio      # Launch ZiuroDB Studio web dashboard UI
```

### • TypeScript Support & Autocomplete IntelliSense
Pass generic type parameters to `table<T>()` or `collection<T>()` for full field autocomplete & type safety:

```typescript
import { ZiuroDB } from 'ziurodb';

interface User {
  id?: number;
  name: string;
  email: string;
  age?: number;
  country?: string;
}

const db = new ZiuroDB({ email: 'user@example.com', password: 'secret' });
const conn = await db.connections('MongoDB');
const database = await conn('ziurodb_database');

// Passing <User> interface unlocks full TypeScript IntelliSense autocomplete!
const users = database.table<User>('users');

await users.insert({
  name: 'Shani Kumar',
  email: 'shani@company.com',
  age: 24,
  country: 'India'
});
```

### • Environment Variable Loader
Configure connections using environment variables seamlessly:

```javascript
const db = new ZiuroDB({
  connection: {
    env: 'DATABASE_URL' // Reads process.env.DATABASE_URL or .env file automatically
  }
});
```

---

## 🔑 1. Authentication & Session Management

### • Instantiate Client with Auto-Login
```javascript
const { ZiuroDB } = require('ziurodb');

const db = new ZiuroDB({
  email: 'shanikumar00321@gmail.com',
  password: 'your_password',
  // backendUrl: 'https://app.ziurodb.com' // Optional custom backend URL
});
```

### • Explicit User Login
```javascript
const { user, token } = await db.login('user@email.com', 'password');
console.log('Logged in user ID:', user._id);
```

### • Check Currently Logged-in User Profile
```javascript
const user = await db.whoami(); // OR await db.getUser() or await db.me()
console.log('User Email:', user.email);
console.log('User Role:', user.role);
console.log('Plan:', user.planName);
```

### • Logout & Invalidate Session
```javascript
const result = await db.logout();
console.log(result.message); // "Logged out successfully"
```

---

## 🔗 2. Connection Management

### • List All Account Database Connections
```javascript
const connections = await db.connections.list();
console.log('Total Connections:', connections.length);
// Returns array of connection objects: [{ _id, name, dbType, host, agentId, ... }]
```

### • Select Connection Handle (By Name or ID)
```javascript
// Select by display name (e.g. 'MongoDB', 'MySQL', 'Firebase', 'ZiuroDB')
const conn = await db.connections('MongoDB');

// OR using the connections array helper:
const conn = await connections.get('MongoDB');
```

### • Create a New Database Connection Programmatically
```javascript
const newConn = await db.connections.create({
  name: 'Production PostgreSQL',
  dbType: 'postgresql',
  host: 'db.example.com',
  port: 5432,
  dbName: 'main_db',
  username: 'admin',
  password: 'secret_password'
});
console.log('Created Connection ID:', newConn._id);
```

### • Delete a Database Connection
```javascript
await db.connections.delete('6a6fa0daad08b6d2b718af4a');
```

---

## 🗄️ 3. Database Navigation & Schema Creation

### • List All Databases on Server Connection
```javascript
const databases = await conn.list(); // OR await conn.listDatabases()
console.log('Databases:', databases); 
// ['admin', 'book_library', 'company_db', 'mydb']
```

### • Select Active Database Context
```javascript
const database = await conn('book_library'); // OR await conn.database('book_library')
console.log('Active DB:', database.dbName);
```

### • List All Tables / Collections in Database
```javascript
const tables = await database.list(); // OR await database.listTables()
console.log('Tables / Collections:', tables);
// ['users', 'books', 'orders']
```

### • Define & Create Schema / Table Programmatically
```javascript
const { primary_key, inc } = require('ziurodb');

const userSchema = {
  id: {
    type: Number,
    value: primary_key, // Primary Key
    default: inc({ st: 1, end: 10000 }) // Auto-Increment sequence
  },
  name: {
    type: String,
    required: true,
    min: 3,
    max: 30,
    trim: true
  },
  email: {
    type: String,
    required: true,
    trim: true
  },
  createdAt: {
    default: Date.now
  }
};

// Create table/collection with schema definition
const userTable = await database('users', userSchema);
// OR: const userTable = await database.createTable('users', userSchema);
```

---

## 🧬 4. Advanced ORM Features

### • Relationships & Population (`hasMany`, `belongsTo`, `populate`)
Define model associations and populate joined data seamlessly across collections / tables:

```javascript
const User = database('users');
const Post = database('posts');

// Define Relationships
User.hasMany(Post, { foreignKey: 'userId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'userId', as: 'author' });

// Populate user's posts
const userWithPosts = await User.populate('posts').findById(id);
console.log(userWithPosts.posts); // Array of post objects!

// Populate post's author
const postsWithAuthor = await Post.populate('author').list();
console.log(postsWithAuthor[0].author); // User object!
```

### • Schema Validation & Default Values
Automatically trim strings, enforce min/max bounds, required fields, and evaluate default value functions:

```javascript
const userSchema = {
  name: {
    type: String,
    required: true,
    min: 3,
    max: 30,
    trim: true
  },
  status: {
    type: String,
    default: 'active'
  },
  createdAt: {
    default: () => new Date().toISOString()
  }
};
```

### • Lifecycle Hooks & Middleware
Attach asynchronous middleware hooks before/after CRUD actions:

```javascript
const Users = database('users');

// Before / After Insert Hooks
Users.beforeInsert(async (doc) => {
  console.log('Encrypting password before insert for:', doc.email);
});

Users.afterInsert(async (doc) => {
  console.log('Sending welcome email to:', doc.email);
});

// Update & Delete Hooks
Users.beforeUpdate(async (filter, updateData) => { ... });
Users.afterUpdate(async (result) => { ... });
Users.beforeDelete(async (filter) => { ... });
Users.afterDelete(async (result) => { ... });
```

### • Virtual Fields & Computed Columns
Define dynamic getters that compute values on query results:

```javascript
const Users = database('users');

// Computed Virtual Getter for Full Name
Users.virtual('fullName', (doc) => `${doc.firstName} ${doc.lastName}`);

// Computed Column for Salary After Tax
Users.virtual('salaryAfterTax', (doc) => doc.salary * 0.8);

const user = await Users.first();
console.log(user.fullName);        // "Shani Kumar"
console.log(user.salaryAfterTax);  // 80000
```

---

## 📊 5. Query Builder, Aggregations & Operations

### • Aggregate Functions (`sum`, `avg`, `max`, `min`, `count`)
```javascript
const totalSalary = await users.sum("salary");
const avgAge = await users.avg("age");
const maxSalary = await users.max("salary");
const minAge = await users.min("age");
const totalCount = await users.count();
```

### • Distinct Query (`distinct`)
```javascript
const countries = await users.distinct("country");
console.log(countries); // ["India", "USA", "UK"]
```

### • Upsert Operation (`upsert`)
```javascript
await users.upsert(
  { email: "test@gmail.com" }, // Filter criteria
  { name: "John Doe", age: 30 } // Data to insert or update
);
```

### • Bulk Operations (`bulkInsert`, `bulkUpdate`, `bulkDelete`)
```javascript
// Bulk Insert
await users.bulkInsert([
  { name: "User 1", email: "u1@test.com" },
  { name: "User 2", email: "u2@test.com" }
]);

// Bulk Update
await users.bulkUpdate({ status: "pending" }, { status: "active" });

// Bulk Delete
await users.bulkDelete({ status: "banned" });
```

### • Check Record Existence (`exists`)
```javascript
const exists = await users.exists({ email: "test@gmail.com" });
console.log('User exists:', exists); // true / false
```

### • Fluent Chainable Query Builder (`where`, `orderBy`, `sort`, `limit`, `offset`, `get`)
```javascript
// Operator & string syntax
const results1 = await users
  .where("age", ">", 18)
  .where("country", "India")
  .orderBy("name")
  .limit(20)
  .offset(40)
  .get(); // OR direct await users.where(...)

// Mongo-style object syntax
const results2 = await users
  .where({
    age: { $gt: 18 }
  })
  .sort({
    createdAt: -1
  })
  .limit(10)
  .get();
```

### • Paginate Records (`paginate`)
```javascript
const result = await users.paginate({
  page: 2,
  limit: 20
});

console.log(result);
/*
Returns:
{
  data: [ ... ],
  page: 2,
  totalPages: 10,
  totalItems: 200
}
*/
```

---

## 🔒 6. Atomic Database Transactions (`transaction`)

Execute multi-table atomic operations cleanly with automatic commit and rollback handling:

```javascript
await db.transaction(async (tx) => {
  const users = tx.table("users");
  const orders = tx.table("orders");

  await users.insert({ name: "Alice", balance: 1000 });
  await orders.insert({ userId: "123", total: 150 });
}, 'ZiuroDB', 'ziurodb_database');
```

---

## ⚡ 7. Raw & ZQL Query Execution

Run raw database queries (ZQL, SQL `SELECT`, or Mongo `db.collection.find()`) directly against any database connection:

```javascript
// Option 1: Positional arguments
const res1 = await db.query('6a6fa0d...', 'SELECT * FROM users WHERE status = "active"', 'main_db');

// Option 2: Object options syntax
const res2 = await db.query({
  connectionId: '6a6fa0d...',
  query: 'db.users.find({ age: { $gte: 18 } })',
  dbName: 'book_library'
});

console.log('Query rows:', res2.rows);
```

---

## 🤖 8. Agent Management

### • List Registered Local Agents
```javascript
const agents = await db.agents.list();
console.log('Agents:', agents);
```

### • Register a New Desktop / CLI Agent
```javascript
const newAgent = await db.agents.register('MacBook-Pro-Agent');
console.log('Agent ID:', newAgent.agentId);
console.log('Agent Token:', newAgent.agentToken);
```

### • CLI Agent Runner
From command line:
```bash
npx ziurodb tunnel --id=agent-70253ecf01a9 --token=pat-your-token-here
```

---

## 📋 Summary Overview Matrix

| Module | Methods | Purpose |
| :--- | :--- | :--- |
| **AI Query Engine** | `db.ai("query prompt")` | Natural language text prompt to query execution |
| **Multi-DB Join** | `db.join(mongoUsers, postgresOrders, "userId")` | Cross-database in-memory data joining |
| **Live Streams** | `users.watch(fn)`, `users.on('insert', fn)` | Real-time live change streams & event listeners |
| **Query Cache** | `users.cache(60).find()` | In-memory query result caching |
| **Query Retry** | `users.retry(3).find()` | Automatic query retry engine with backoff |
| **Schema AI** | `db.schema.explain()`, `db.optimize()` | AI schema intelligence & index optimization |
| **Health Metrics** | `db.health()`, `db.pool.stats()` | Real-time database health & connection pool metrics |
| **Backup/Restore** | `db.backup()`, `db.restore()`, `db.sync()`, `import/export` | Database backups, restore & cross-provider sync |
| **CLI Tools** | `ziurodb init`, `generate`, `migrate`, `rollback`, `seed`, `studio` | CLI project initialization, migration, & studio |
| **TypeScript** | `table<Interface>('users')` | Full generic autocomplete IntelliSense support |
| **Auth** | `login()`, `whoami()`, `getUser()`, `me()`, `logout()` | Account login, session verification & logout |
| **Connections** | `connections.list()`, `connections('Name')`, `create()`, `delete()` | Manage database connection handles |
| **Databases** | `conn.list()`, `conn('dbName')` | Server-level database listing & context selection |
| **Tables/Schema** | `database.list()`, `database('tableName', schema)` | Collection discovery & DDL schema creation |
| **Relationships** | `User.hasMany(Post)`, `Post.belongsTo(User)`, `populate()` | Model associations & joined data population |
| **Validation** | `min`, `max`, `trim`, `required`, `default` | Data validation, sanitization & defaults |
| **Hooks** | `beforeInsert`, `afterInsert`, `beforeUpdate`, `afterUpdate` | Asynchronous lifecycle event middleware |
| **Virtuals** | `table.virtual('fullName', doc => ...)` | Dynamic getters & computed columns |
| **Aggregates** | `sum()`, `avg()`, `max()`, `min()`, `count()`, `distinct()` | Statistical & aggregate query calculations |
| **Mutation Ops** | `upsert()`, `bulkInsert()`, `bulkUpdate()`, `bulkDelete()` | Batch & conditional data mutations |
| **Transactions** | `db.transaction(async (tx) => { ... })` | Atomic multi-table database transactions |
| **Fluent ORM** | `findById()`, `first()`, `last()`, `count()`, `exists()`, `find()` | Record helpers & single object lookup |
| **Query Builder** | `.where()`, `.orderBy()`, `.sort()`, `.limit()`, `.offset()`, `.get()` | Fluent chainable query builder |
| **Pagination** | `users.paginate({ page, limit })` | Paginated data queries |
| **Data CRUD** | `list()`, `find()`, `insert()`, `update()`, `delete()` | Query & mutate data as clean JavaScript Objects |
| **Query Engine** | `db.query({ connectionId, query, dbName })` | Raw SQL / ZQL / Mongo query execution |
| **Agents** | `agents.list()`, `agents.register()`, `npx ziurodb tunnel` | Manage secure tunnel agents for local databases |
