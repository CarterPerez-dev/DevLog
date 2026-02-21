# DatabaseUtils.get_comprehensive_stats

**Type:** Security Review
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
                "tuples
```

---

## Security Review

### Security Review for `get_comprehensive_stats` Function

#### Vulnerabilities Found:

1. **SQL Injection** - **Severity: Low**
   - The SQL queries do not use parameterized statements, which could be exploited if the query strings are constructed from untrusted input.
   
2. **Hardcoded Secrets or Credentials** - **Severity: Info**
   - No hardcoded secrets or credentials are present in the code snippet.

3. **Input Validation Gaps** - **Severity: Low**
   - The function does not validate any user-provided inputs, which could lead to unexpected behavior if used incorrectly.

4. **Error Handling that Leaks Information** - **Severity: Medium**
   - The `try_get` method is used without proper error handling, which could leak information about the database schema or structure in case of errors.

#### Attack Vectors:

- An attacker could manipulate the SQL query strings to inject malicious commands.
- Improper error handling might reveal sensitive information about the database schema and statistics.

#### Recommended Fixes:

1. **SQL Injection**:
   - Use parameterized queries instead of string concatenation for dynamic parts of your SQL statements.
   
2. **Error Handling**:
   - Implement proper error handling using `Result` and `Option` patterns to avoid leaking sensitive information.
     ```rust
     let blks_read = db_stats.try_get("blks_read")?;
     if let Ok(hit_ratio) = (blks_read + blks_hit).try_into().map(|r| r > 0.0) {
         // Use hit_ratio as needed
     }
     ```

3. **Overall Security Posture**:
   - The function is generally secure but could benefit from improved error handling and parameterized queries to mitigate potential risks.

By addressing these issues, the security posture of the code can be significantly enhanced.

---

*Generated by CodeWorm on 2026-02-21 10:13*
