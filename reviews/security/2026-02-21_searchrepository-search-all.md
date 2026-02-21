# SearchRepository.search_all

**Type:** Security Review
**Repository:** my-portfolio
**File:** v1/backend/app/search/repository.py
**Language:** python
**Lines:** 18-119
**Complexity:** 2.0

---

## Source Code

```python
async def search_all(
        cls,
        session: AsyncSession,
        query: str,
        language: Language,
        limit: int = 20,
    ) -> list[SearchResultItem]:
        """
        Full-text search across projects, experiences, and certifications.
        Returns results with highlighted excerpts.
        """
        search_query = text(
            """
            WITH search_results AS (
                SELECT
                    title,
                    ts_headline(
                        'english',
                        COALESCE(description, '') || ' ' || COALESCE(technical_details, ''),
                        plainto_tsquery('english', :query),
                        'StartSel=<mark>, StopSel=</mark>, MaxWords=35, MinWords=15, MaxFragments=2'
                    ) AS excerpt,
                    '/projects/' || slug AS url,
                    'project' AS type,
                    ts_rank(
                        to_tsvector('english', title || ' ' || COALESCE(description, '') || ' ' || COALESCE(technical_details, '')),
                        plainto_tsquery('english', :query)
                    ) AS rank
                FROM projects
                WHERE language = :language
                AND to_tsvector('english', title || ' ' || COALESCE(description, '') || ' ' || COALESCE(technical_details, ''))
                    @@ plainto_tsquery('english', :query)

                UNION ALL

                SELECT
                    company || ' - ' || role AS title,
                    ts_headline(
                        'english',
                        description,
                        plainto_tsquery('english', :query),
                        'StartSel=<mark>, StopSel=</mark>, MaxWords=35, MinWords=15, MaxFragments=2'
                    ) AS excerpt,
                    '/background/experience' AS url,
                    'experience' AS type,
                    ts_rank(
                        to_tsvector('english', company || ' ' || role || ' ' || description),
                        plainto_tsquery('english', :query)
                    ) AS rank
                FROM experiences
                WHERE language = :language
                AND is_visible = true
                AND to_tsvector('english', company || ' ' || role || ' ' || description)
                    @@ plainto_tsquery('english', :query)

                UNION ALL

                SELECT
                    name AS title,
                    ts_headline(
                        'english',
                        name || ' - ' || issuer,
                        plainto_tsquery('english', :query),
                        'StartSel=<mark>, StopSel=</mark>, MaxWords=35, MinWords=15, MaxFragments=2'
                    ) AS excerpt,
                    '/background/certifications' AS url,
                    'certification' AS type,
                    ts_rank(
                        to_tsvector('english', name || ' ' || issuer),
                  
```

---

## Security Review

### Security Review

#### Vulnerabilities Found:

1. **SQL Injection** - **Severity: Critical**
   - **Line 8**: The `query` parameter is directly used in the SQL query without proper sanitization or parameter binding, which could lead to SQL injection attacks.

2. **Information Leakage** - **Severity: Medium**
   - **Line 34-36**: Error handling and logging are not present, potentially leaking information about the database structure if an error occurs.

#### Attack Vectors:

1. **SQL Injection**: An attacker could craft a malicious query to manipulate the SQL statement.
2. **Information Leakage**: Detailed error messages might reveal sensitive information about the database schema or queries.

#### Recommended Fixes:

1. **Fix SQL Injection**:
   - Use parameterized queries instead of string interpolation for the `query` parameter.
     ```python
     search_query = text(
         """
         ...
         AND to_tsvector('english', title || ' ' || COALESCE(description, '') || ' ' || COALESCE(technical_details, '')) 
             @@ plainto_tsquery('english', :query)
         ...
         """
     )
     result = await session.execute(
         search_query,
         {
             "query": query,
             "language": language.value,
             "limit": limit,
         }
     )

2. **Improve Error Handling**:
   - Add try-except blocks to handle and log errors gracefully.
     ```python
     try:
         result = await session.execute(search_query, {"query": query, "language": language.value, "limit": limit})
         rows = result.fetchall()
         return [SearchResultItem(title=row.title, excerpt=row.excerpt, url=row.url, type=row.type) for row in rows]
     except Exception as e:
         logger.error(f"Error executing search: {e}")
         raise
     ```

#### Overall Security Posture:

The code has critical vulnerabilities that need immediate attention to prevent SQL injection attacks. Improving error handling will also enhance the security posture by reducing information leakage. Regularly reviewing and updating the code for security best practices is recommended to maintain a robust security stance.

---

*Generated by CodeWorm on 2026-02-21 11:29*
