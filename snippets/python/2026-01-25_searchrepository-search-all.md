# SearchRepository.search_all

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
                        plainto_tsquery('english', :query)
                    ) AS rank
                FROM certifications
                WHERE language = :language
                AND is_visible = true
                AND to_tsvector('english', name || ' ' || issuer)
                    @@ plainto_tsquery('english', :query)
            )
            SELECT title, excerpt, url, type
            FROM search_results
            ORDER BY rank DESC
            LIMIT :limit
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

        rows = result.fetchall()
        return [
            SearchResultItem(
                title = row.title,
                excerpt = row.excerpt,
                url = row.url,
                type = row.type,
            ) for row in rows
        ]
```

---

## Documentation

### Documentation for `search_all` Method

**Purpose and Behavior:**
The `search_all` method performs a full-text search across projects, experiences, and certifications within the repository. It returns search results with highlighted excerpts based on a query string and language preference.

**Key Implementation Details:**
- Utilizes SQL queries to search through multiple tables (`projects`, `experiences`, `certifications`) using PostgreSQL's full-text search capabilities.
- Constructs a Common Table Expression (CTE) named `search_results` to combine results from different tables, ensuring consistent ranking and excerpt generation.
- Uses `ts_headline` for highlighting query terms in the excerpts.
- Orders results by relevance (`rank`) and limits the output.

**When/Why to Use:**
This method should be used when implementing a search feature that needs to cover multiple data sources within a single application. It is particularly useful for applications requiring fast, accurate full-text searches across diverse content types like projects, experiences, and certifications.

**Patterns and Gotchas:**
- The use of `text` and `search_query` allows dynamic SQL execution with parameter binding, enhancing security.
- The method leverages PostgreSQL's full-text search functions (`ts_headline`, `plainto_tsquery`, etc.), which require careful configuration for optimal performance.
- Ensure that the `SearchResultItem` class is correctly defined to match the returned fields. Misalignment can lead to errors.

This code snippet effectively encapsulates complex database operations into a reusable method, making it easier to maintain and extend.

---

*Generated by CodeWorm on 2026-01-25 06:08*
