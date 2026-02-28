# StateManager

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/core/state.py
**Language:** python
**Lines:** 34-255
**Complexity:** 0.0

---

## Source Code

```python
class StateManager:
    """
    Manages persistent state in SQLite
    This is the daemon's memory - tracks what has been documented
    """
    def __init__(self, db_path: Path) -> None:
        """
        Initialize state manager with database path
        """
        self.db_path = db_path
        self.db_path.parent.mkdir(parents = True, exist_ok = True)
        self._init_db()

    def _init_db(self) -> None:
        """
        Initialize database schema and run migrations
        """
        with sqlite3.connect(self.db_path) as conn:
            conn.executescript(SCHEMA)
            self._migrate_add_doc_type(conn)
            conn.commit()

    def _migrate_add_doc_type(self, conn: sqlite3.Connection) -> None:
        """
        Add doc_type column if it does not exist
        """
        cursor = conn.execute("PRAGMA table_info(documented_snippets)")
        columns = {row[1] for row in cursor.fetchall()}
        if "doc_type" not in columns:
            conn.execute(
                "ALTER TABLE documented_snippets "
                "ADD COLUMN doc_type TEXT NOT NULL DEFAULT 'function_doc'"
            )
            conn.execute(
                "CREATE INDEX IF NOT EXISTS idx_dedup "
                "ON documented_snippets(source_file, function_name, class_name, doc_type)"
            )
            conn.execute(
                "CREATE INDEX IF NOT EXISTS idx_doc_type "
                "ON documented_snippets(doc_type)"
            )

    def _get_conn(self) -> sqlite3.Connection:
        """
        Get a database connection with row factory
        """
        conn = sqlite3.connect(self.db_path, timeout = 10)
        conn.row_factory = sqlite3.Row
        return conn

    @staticmethod
    def hash_code(source: str) -> str:
        """
        Generate SHA256 hash of source code for deduplication
        """
        return hashlib.sha256(source.encode("utf-8")).hexdigest()

    def is_documented(self, snippet: CodeSnippet) -> bool:
        """
        Check if this exact code has already been documented
        """
        code_hash = self.hash_code(snippet.source)
        with self._get_conn() as conn:
            cursor = conn.execute(
                "SELECT 1 FROM documented_snippets WHERE code_hash = ? LIMIT 1",
                (code_hash,
                 ),
            )
            return cursor.fetchone() is not None

    def get_existing_doc(self, snippet: CodeSnippet) -> DocumentedSnippet | None:
        """
        Get existing documentation record for a function/class if it exists
        Returns None if never documented
        """
        with self._get_conn() as conn:
            cursor = conn.execute(
                """
                SELECT * FROM documented_snippets
                WHERE source_file = ? AND function_name = ? AND class_name IS ?
                ORDER BY documented_at DESC LIMIT 1
                """,
                (
                    str(snippet.file_path),
                    snippet.fun
```

---

## Class Documentation

### StateManager Class Documentation

**Responsibility and Purpose:**
The `StateManager` class is responsible for managing persistent state in an SQLite database, acting as the daemon's memory to track documented code snippets. It ensures that each snippet is only documented once per entity (function/class) and doc type (e.g., function_doc, security_review).

**Public Interface:**
- **Initialization (`__init__`):** Initializes the `StateManager` with a database path.
- **Database Initialization (`_init_db`, `_migrate_add_doc_type`):** Sets up the database schema and runs migrations to ensure it is in the correct state.
- **Connection Management (`_get_conn`):** Provides a connection to the SQLite database with a row factory for easier data handling.
- **Hashing (`hash_code`):** Generates a SHA256 hash of source code for deduplication purposes.
- **Document Checking (`is_documented`, `should_document`):** Determines if a snippet has already been documented and whether it should be re-documented based on the last documentation date.

**Design Patterns Used:**
- **Factory Pattern:** `_get_conn` ensures that connections are properly configured with row factories for easier data retrieval.
- **Strategy Pattern:** The `DocType` enum allows different types of documentation to be handled uniformly, ensuring flexibility in how snippets can be documented.

**Relationship to Other Classes:**
The `StateManager` interacts closely with the `CodeSnippet` and `DocumentedSnippet` classes. It uses these classes to hash code snippets for deduplication and to store/document existing snippets. The `StateManager` is a central component in the architecture, providing persistence and state management that other parts of the application rely on.

**State Management Approach:**
The class manages state by storing documented snippets in an SQLite database with appropriate indexes for efficient querying. It ensures data integrity through schema initialization and migrations, and deduplicates entries based on code hashes and doc types.

---

*Generated by CodeWorm on 2026-02-28 14:09*
