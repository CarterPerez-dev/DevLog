# SearchRepository.search_all

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

**Time Complexity:** The primary bottleneck is the full-text search query, which involves multiple `UNION ALL` operations and text processing functions like `ts_headline` and `ts_rank`. This results in a time complexity of approximately \(O(n \times m)\), where \(n\) is the number of rows across all tables and \(m\) is the average length of the search query.

**Space Complexity:** The space complexity is primarily determined by the memory required to hold the intermediate result set before filtering and ranking. This can be significant if there are many matching records, leading to potential memory leaks or high memory usage.

**Bottlenecks:**
1. **Full-Text Search Operations:** The `ts_headline` and `ts_rank` functions can be costly due to their text processing nature.
2. **Multiple `UNION ALL` Queries:** These operations can lead to redundant scans of the same data across different tables, increasing query execution time.

**Optimization Opportunities:**
1. **Indexing:** Ensure that appropriate indexes are in place for columns used in `WHERE` and `MATCH` clauses to speed up the search.
2. **Query Refinement:** Simplify the query by reducing redundant operations or using more efficient text processing techniques.
3. **Caching:** Cache frequently accessed results, especially if the same queries are run multiple times with similar parameters.

**Resource Usage Concerns:**
- Ensure that connections to the database are properly managed and closed to avoid resource leaks.
- Consider implementing connection pooling to manage database sessions efficiently.

By optimizing indexing, refining query logic, and managing resources effectively, you can significantly improve the performance of this search function.

---

*Generated by CodeWorm on 2026-02-21 20:15*
