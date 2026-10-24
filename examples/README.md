# 📚 Ejemplos de Aura ORM (`examples/`)

Este directorio contiene ejemplos modulares y ejecutables que demuestran todas las capacidades y características de **Aura ORM** (`aura-orm`).

Cada archivo es autónomo y puede ejecutarse directamente con el compilador oficial de Aura (`aurac`).

---

## 🚀 Ejecución de los Ejemplos

### Ejecutar todos los ejemplos juntos:
```bash
make examples
```

### Ejecutar un ejemplo individual:
```bash
aurac run examples/01_select_queries.aura
aurac run examples/02_advanced_filters.aura
aurac run examples/03_aggregations_and_having.aura
aurac run examples/04_insert_and_batch.aura
aurac run examples/05_update_and_delete.aura
aurac run examples/06_schema_indexes_migrations.aura
aurac run examples/07_transactions.aura
```

---

## 📑 Índice de Ejemplos y Características

### 1. [`01_select_queries.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/01_select_queries.aura) — Consultas `SELECT`, Paginación y Joins
Demuestra los fundamentos de construcción de consultas de lectura y la parametrización multi-dialecto:
- **Placeholders según dialecto**: `$1, $2, ...` para PostgreSQL y `?` para MySQL / SQLite.
- **Proyecciones de columnas**: Selección selectiva con `.Select("col")`.
- **Condicionales WHERE**: Filtrado estándar con operadores (`=`, `>=`, `<`, etc.).
- **Paginación fluida**: `.Limit(n)` y `.Offset(n)`.
- **Ordenamiento**: `.OrderBy("col", "ASC" | "DESC")`.
- **Joins e IS NULL**: `.Join("INNER", table, onCondition)` y `.WhereNull(col)`.

---

### 2. [`02_advanced_filters.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/02_advanced_filters.aura) — Filtros Avanzados y Lógica `OR`
Demuestra la expresividad extendida de cláusulas condicionales:
- **Pertenencia a conjuntos**: `.WhereIn(col, values)` y `.WhereNotIn(col, values)` generando dinámicamente los placeholders correspondientes.
- **Rangos**: `.WhereBetween(col, low, high)`.
- **Búsqueda por patrones**: `.WhereLike(col, pattern)` y `.WhereILike(col, pattern)` con fallback automático (`ILIKE` en PostgreSQL y `LIKE` en MySQL/SQLite).
- **Lógica booleana OR**: `.OrWhere(col, op, value)` para combinar múltiples criterios de búsqueda.
- **Valores no nulos**: `.WhereNotNull(col)`.

---

### 3. [`03_aggregations_and_having.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/03_aggregations_and_having.aura) — Agregaciones, Agrupamientos y `HAVING`
Demuestra la generación de reportes y métricas analíticas:
- **Funciones de agregación integradas**:
  - `.Count(field, asAlias)`
  - `.Sum(field, asAlias)`
  - `.Avg(field, asAlias)`
  - `.Min(field, asAlias)`
  - `.Max(field, asAlias)`
- **Cláusula GROUP BY**: Agrupamiento por una o más columnas mediante `.GroupBy(col)`.
- **Cláusula HAVING**: Filtrado de grupos agregados mediante `.Having(condition)`.

---

### 4. [`04_insert_and_batch.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/04_insert_and_batch.aura) — Inserciones, `RETURNING`, Lotes y UPSERT
Demuestra operaciones de escritura de alta eficiencia y resolución de conflictos:
- **Inserción simple**: `.Set(col, val)`.
- **Cláusula RETURNING**: Recuperación directa de identificadores o timestamps (`.Returning("id")`).
- **Batch / Bulk Insert**: Inserción masiva de múltiples registros en un solo viaje de red con `.BatchInsert(columns, rows)` manteniendo los índices de placeholders consistentes.
- **Expresiones SQL Crudas**: `.SetRaw(col, rawExpr)` para usar funciones SQL nativas como `CURRENT_TIMESTAMP`.
- **UPSERT / On Conflict**: Resolución de conflictos con `.OnConflictDoNothing(target)` y `.OnConflictDoUpdate(target, assignments)`.

---

### 5. [`05_update_and_delete.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/05_update_and_delete.aura) — `UPDATE` y `DELETE` Avanzados
Demuestra operaciones de actualización y borrado con opciones enriquecidas:
- **Actualizaciones con expresiones crudas**: `.SetRaw("stock", "stock - 1")` para cálculos atómicos en base de datos.
- **Valores parametrizados en UPDATE**: Mezcla transparente entre campos fijos y filtros WHERE.
- **Cláusula RETURNING en UPDATE y DELETE**: Retorno inmediato de registros modificados o eliminados (PostgreSQL / SQLite).
- **Cláusula LIMIT en UPDATE y DELETE**: Restricción de impacto a un número máximo de registros (MySQL y SQLite).

---

### 6. [`06_schema_indexes_migrations.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/06_schema_indexes_migrations.aura) — Esquemas DDL, Índices y Migraciones
Demuestra la definición declarativa de estructuras de base de datos:
- **Tipos de columnas modernos**:
  - `UuidColumn("id")`: mapea a `UUID` nativo en PostgreSQL y `VARCHAR(36)` en otros dialectos.
  - `JsonColumn("metadata")`: mapea a `JSONB` en PostgreSQL y `JSON` en SQLite/MySQL.
  - `FloatColumn("score")`: precisión de punto flotante.
  - `BlobColumn("payload")`: datos binarios (`BYTEA` en PostgreSQL, `BLOB` en SQLite).
- **Claves foráneas y borrado en cascada**: `.References(table, col).OnDelete("CASCADE")`.
- **Generación de Índices**: Índices simples o compuestos con `.AddIndex(name, columns, isUnique)` y `.ToCreateIndexSql(dialect)`.
- **Migraciones Incrementales (`ALTER TABLE`)**:
  - `AddColumnSql(table, colDef, dialect)`
  - `DropColumnSql(table, colName, dialect)`
  - `RenameColumnSql(table, oldName, newName, dialect)`

---

### 7. [`07_transactions.aura`](file:///Users/mavro/Projects/AuraProjects/aura-orm/examples/07_transactions.aura) — Control Transaccional
Demuestra la generación de comandos transaccionales consistentes y conscientes del dialecto:
- **Inicio de transacción**: `.Begin()` (`START TRANSACTION;` en MySQL, `BEGIN;` en PostgreSQL y SQLite).
- **Confirmación y reversión**: `.Commit()` y `.Rollback()`.
- **Puntos de guardado (Savepoints)**: `.Savepoint(name)` y reversión parcial con `.RollbackToSavepoint(name)`.
