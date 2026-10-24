# ⚡ Aura ORM (`aura-orm`)

A native **Object-Relational Mapping (ORM)**, **Fluent Query Builder**, and **DDL Schema Management** library built for the **Aura** programming language (`aurac`).

---

## 🚀 Features

- **Multi-Dialect Fluent Query Builder**: Programmatic and parameterized construction of `SELECT`, `INSERT`, `UPDATE`, and `DELETE` statements.
- **Native SQL Dialects**:
  - **PostgreSQL**: Numbered placeholders (`$1, $2, ...`), `RETURNING` clauses, `SERIAL PRIMARY KEY`, and quoted identifiers (`"table"`).
  - **MySQL**: Standard placeholders (`?`), backtick escaping (`` `table` ``), and `AUTO_INCREMENT`.
  - **SQLite**: Positional placeholders (`?`), `INTEGER PRIMARY KEY AUTOINCREMENT`, and lightweight typing.
- **Relational Table Joins**: Support for `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and `FULL OUTER JOIN`.
- **Schema & DDL Definition**: Declarative table definitions, column types (`INTEGER`, `BIGINT`, `VARCHAR`, `TEXT`, `BOOLEAN`, `NUMERIC`, `TIMESTAMP`, `JSON`, `UUID`, `FLOAT`, `BLOB`), constraints (`PRIMARY KEY`, `NOT NULL`, `UNIQUE`), and `DEFAULT` values.
- **Migrations & Alterations**: DDL helpers for `ALTER TABLE ADD COLUMN`, `DROP COLUMN`, `RENAME COLUMN`, and index creation.
- **Transaction SQL Manager**: Clean management of `BEGIN`, `SAVEPOINT`, `ROLLBACK TO SAVEPOINT`, and `COMMIT`.
- **Safe Method Dispatch**: Built on Aura's receiver model with typed structs and internal `__aura_type` dispatching.
- **100% Native Aura**: Pure Aura code with zero heavy external runtime dependencies.

---

## 📁 Project Structure

```text
aura-orm/
├── aura.mod                  # Aura module manifest (v0.5.0)
├── Makefile                  # Build, test, and check targets
├── README.md                 # English documentation
├── README_es.md              # Spanish documentation
├── src/
│   ├── dialect/
│   │   └── dialect.aura      # Dialect abstractions (PostgreSQL, MySQL, SQLite)
│   ├── builder/
│   │   └── query.aura        # Query Builder for SELECT, INSERT, UPDATE, DELETE & Joins
│   ├── schema/
│   │   └── schema.aura       # Column definitions, table schema, and DDL generator
│   └── orm.aura              # Unified high-level ORM facade
├── examples/                 # Comprehensive runnable example scripts
│   ├── 01_select_queries.aura
│   ├── 02_advanced_filters.aura
│   ├── 03_aggregations_and_having.aura
│   ├── 04_insert_and_batch.aura
│   ├── 05_update_and_delete.aura
│   ├── 06_schema_indexes_migrations.aura
│   ├── 07_transactions.aura
│   ├── 08_joins.aura
│   └── README.md
└── tests/
    ├── builder_test.aura     # Unit tests for query construction & parameterization
    └── schema_test.aura      # Unit tests for DDL and schema generation
```

---

## 🛠️ Usage Guide

### 1. Initializing the ORM

```aura
import { PostgresOrm, MysqlOrm, SqliteOrm } from "./src/orm";

// Instantiate for your target database:
let orm = PostgresOrm();
```

---

### 2. `SELECT` Queries with Filters and Pagination

```aura
let query = orm.SelectFrom("users")
  .Select("id")
  .Select("name")
  .Select("email")
  .Where("status", "=", "active")
  .Where("age", ">=", "18")
  .OrderBy("created_at", "DESC")
  .Limit(10)
  .Offset(20)
  .ToSql();

println(query.sql);
// SELECT id, name, email FROM users WHERE status = $1 AND age >= $2 ORDER BY created_at DESC LIMIT 10 OFFSET 20;
// query.params => ["active", "18"]
```

---

### 3. Relational Table Joins (`INNER`, `LEFT`, `RIGHT`, `FULL`)

```aura
let joinQuery = orm.SelectFrom("users")
  .Select("users.id")
  .Select("users.email")
  .Select("profiles.bio")
  .InnerJoin("profiles", "users.id = profiles.user_id")
  .LeftJoin("orders", "users.id = orders.user_id")
  .Where("users.status", "=", "active")
  .ToSql();

println(joinQuery.sql);
// SELECT users.id, users.email, profiles.bio FROM users
//   INNER JOIN profiles ON users.id = profiles.user_id
//   LEFT JOIN orders ON users.id = orders.user_id
//   WHERE users.status = $1;
```

---

### 4. `INSERT INTO` with `RETURNING` (PostgreSQL)

```aura
let insert = orm.InsertInto("users")
  .Set("name", "Alice")
  .Set("email", "alice@example.com")
  .Set("active", "true")
  .Returning("id")
  .Returning("created_at")
  .ToSql();

println(insert.sql);
// INSERT INTO users (name, email, active) VALUES ($1, $2, $3) RETURNING id, created_at;
// insert.params => ["Alice", "alice@example.com", "true"]
```

---

### 5. `UPDATE` and `DELETE` Queries

```aura
// UPDATE
let update = orm.UpdateTable("users")
  .Set("name", "Alice Smith")
  .SetRaw("views", "views + 1")
  .Where("id", "=", "42")
  .Returning("views")
  .ToSql();

// DELETE
let delete = orm.DeleteFrom("sessions")
  .Where("expired", "=", "true")
  .Returning("id")
  .ToSql();
```

---

### 6. Batch Inserts

```aura
let batchInsert = orm.InsertInto("metrics")
  .BatchInsert(["key", "val"], [["cpu", "80"], ["mem", "65"]])
  .ToSql();

println(batchInsert.sql);
// INSERT INTO metrics (key, val) VALUES ($1, $2), ($3, $4);
```

---

### 7. Advanced Filters, Aggregations, and `HAVING`

```aura
let report = orm.SelectFrom("employees")
  .Select("department")
  .Count("id", "total_emp")
  .Sum("salary", "total_payroll")
  .WhereIn("role", ["developer", "designer"])
  .WhereBetween("salary", "40000", "90000")
  .WhereILike("name", "john%")
  .OrWhere("status", "=", "exempt")
  .GroupBy("department")
  .Having("COUNT(id) > 2")
  .ToSql();
```

---

### 8. Schema Definitions, Indexes, and DDL Migrations

```aura
import {
  IntegerColumn,
  StringColumn,
  BoolColumn,
  TimestampColumn,
  JsonColumn,
  UuidColumn,
  FloatColumn,
  BlobColumn,
  AddColumnSql,
  DropColumnSql,
  RenameColumnSql
} from "./src/schema/schema";

let usersTable = orm.DefineTable("users")
  .AddColumn(UuidColumn("id").PrimaryKey())
  .AddColumn(StringColumn("name", 100))
  .AddColumn(StringColumn("email", 150).Unique())
  .AddColumn(JsonColumn("preferences"))
  .AddColumn(FloatColumn("score"))
  .AddColumn(BlobColumn("avatar"))
  .AddColumn(BoolColumn("is_active").Default("true"))
  .AddColumn(TimestampColumn("created_at"))
  .AddIndex("idx_users_email", ["email"], true);

// Generate table creation and index DDL:
let ddlSql = usersTable.ToCreateTableSql(orm.dialect);
let indexesSql = usersTable.ToCreateIndexSql(orm.dialect);

// Schema alterations (ADD / DROP / RENAME):
let alterSql = AddColumnSql("users", StringColumn("phone", 30).Nullable(), orm.dialect);
```

---

### 9. SQL Transaction Manager

```aura
let tx = orm.Transaction();
let beginSql = tx.Begin();               // "BEGIN;" or "START TRANSACTION;"
let spSql = tx.Savepoint("sp1");         // "SAVEPOINT sp1;"
let rbSp = tx.RollbackToSavepoint("sp1"); // "ROLLBACK TO SAVEPOINT sp1;"
let commitSql = tx.Commit();             // "COMMIT;"
```

---

## 🧪 Testing

To check semantic types and run the automated test suite:

```bash
# Validate syntax and semantic type analysis
make check

# Run query builder and schema test suites
make test
```

---

## 📚 Examples

The repository includes ready-to-run examples in the [`examples/`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/) directory:

- [`01_select_queries.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/01_select_queries.aura): Basic and complex `SELECT` queries.
- [`02_advanced_filters.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/02_advanced_filters.aura): `IN`, `NOT IN`, `BETWEEN`, `LIKE`, `ILIKE`, and `IS NULL`.
- [`03_aggregations_and_having.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/03_aggregations_and_having.aura): Aggregations (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) and `GROUP BY / HAVING`.
- [`04_insert_and_batch.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/04_insert_and_batch.aura): Single and batch inserts with `RETURNING`.
- [`05_update_and_delete.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/05_update_and_delete.aura): Updates and deletes with raw SQL expressions.
- [`06_schema_indexes_migrations.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/06_schema_indexes_migrations.aura): DDL creation and migrations.
- [`07_transactions.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/07_transactions.aura): Transactions and savepoints.
- [`08_joins.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/08_joins.aura): Relational joins across tables.

---

## 📄 License

Licensed under the MIT License. See [LICENSE](LICENSE) for details.
