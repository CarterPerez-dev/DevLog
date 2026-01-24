# DatabaseUtils.get_comprehensive_stats

**Repository:** kill-pr0cess.inc
**File:** backend/src/database/mod.rs
**Language:** rust
**Lines:** 105-194
**Complexity:** 9.0

---

## Source Code

```rust
pub async fn get_comprehensive_stats(pool: &DatabasePool) -> Result<serde_json::Value> {
        // Table sizes
        let table_sizes = sqlx::query(
            "SELECT
                schemaname,
                tablename,
                pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size,
                pg_total_relation_size(schemaname||'.'||tablename) as size_bytes
            FROM pg_tables
            WHERE schemaname = 'public'
            ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC"
        )
        .fetch_all(pool)
        .await?;

        // Connection stats
        let connection_stats = sqlx::query(
            "SELECT
                count(*) as total_connections,
                count(*) FILTER (WHERE state = 'active') as active_connections,
                count(*) FILTER (WHERE state = 'idle') as idle_connections
            FROM pg_stat_activity"
        )
        .fetch_one(pool)
        .await?;

        // Database stats
        let db_stats = sqlx::query(
            "SELECT
                numbackends,
                xact_commit,
                xact_rollback,
                blks_read,
                blks_hit,
                tup_returned,
                tup_fetched,
                tup_inserted,
                tup_updated,
                tup_deleted
            FROM pg_stat_database
            WHERE datname = current_database()"
        )
        .fetch_one(pool)
        .await?;

        // Pre-compute block read/hit statistics for hit ratio
        let blks_read: i64 = db_stats.try_get("blks_read")?;
        let blks_hit: i64 = db_stats.try_get("blks_hit")?;
        let hit_ratio = if blks_read + blks_hit > 0 {
            blks_hit as f64 / (blks_read + blks_hit) as f64 * 100.0
        } else {
            0.0
        };

        let stats = serde_json::json!({
            "table_sizes": table_sizes.iter().map(|row| {
                serde_json::json!({
                    "table": row.get::<String, _>("tablename"),
                    "size": row.get::<String, _>("size"),
                    "size_bytes": row.get::<i64, _>("size_bytes")
                })
            }).collect::<Vec<_>>(),
            "connections": {
                "total": connection_stats.get::<i64, _>("total_connections"),
                "active": connection_stats.get::<i64, _>("active_connections"),
                "idle": connection_stats.get::<i64, _>("idle_connections")
            },
            "database": {
                "backends": db_stats.try_get::<i32, _>("numbackends")?,
                "transactions": {
                    "committed": db_stats.try_get::<i64, _>("xact_commit")?,
                    "rolled_back": db_stats.get::<i64, _>("xact_rollback")
                },
                    "blocks": {
                        "read": blks_read,
                        "hit": blks_hit,
                        "hit_ratio": hit_ratio
                    },
                "tuples": {
                    "returned": db_stats.get::<i64, _>("tup_returned"),
                    "fetched": db_stats.get::<i64, _>("tup_fetched"),
                    "inserted": db_stats.get::<i64, _>("tup_inserted"),
                    "updated": db_stats.get::<i64, _>("tup_updated"),
                    "deleted": db_stats.get::<i64, _>("tup_deleted")
                }
            }
        });

        Ok(stats)
    }
```

---

## Documentation

### Documentation for `get_comprehensive_stats`

**Purpose and Behavior:**
The function `get_comprehensive_stats` retrieves detailed statistics from a PostgreSQL database, including table sizes, connection counts, database activity metrics, and block read/hit ratios. The results are formatted as a JSON object.

**Key Implementation Details:**
- **Database Pool Usage:** The function takes a reference to a `DatabasePool`, ensuring efficient database connections.
- **SQL Queries:** Multiple SQL queries are executed to gather different types of statistics:
  - Table sizes using `pg_total_relation_size`.
  - Connection counts and states from `pg_stat_activity`.
  - Database activity metrics like transactions, blocks read/hit, and tuple operations from `pg_stat_database`.
- **Data Transformation:** The raw data is transformed into a structured JSON format for easy consumption.

**When/Why to Use:**
This function should be used when you need comprehensive statistics about the database state. It provides insights into resource usage, performance metrics, and connection activities, which are crucial for monitoring and optimizing database operations.

**Patterns/Gotchas:**
- **Borrowing:** The `&DatabasePool` parameter ensures that the pool is not modified during the function execution.
- **Error Handling:** The use of `Result` effectively handles potential errors from SQL queries. 
- **Performance Considerations:** Multiple database queries are executed, which might impact performance if run frequently or on large databases.

This function encapsulates complex database interactions into a single, reusable method, making it easier to integrate comprehensive statistics into monitoring systems or reporting tools.

---

*Generated by CodeWorm on 2026-01-23 23:58*
