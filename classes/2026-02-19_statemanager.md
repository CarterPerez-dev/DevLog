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
        conn = sqlite3.connect(self.db_path)
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
                    snippet.function_name,
  
```

---

## Class Documentation

### StateManager

**Responsibility and Purpose:**
The `StateManager` class manages persistent state using an SQLite database, tracking documented code snippets. It ensures that each snippet is only documented once per type (e.g., function_doc, security_review) by leveraging deduplication mechanisms.

**Public Interface:**
- **Initialization (`__init__`):** Initializes the state manager with a database path and sets up the necessary schema.
- **Database Initialization (`_init_db`):** Creates or migrates the database schema to include required columns and indexes.
- **Connection Management (`_get_conn`):** Provides a connection to the SQLite database with row factory enabled for easier data handling.
- **Hash Code Generation (`hash_code`):** Generates a SHA256 hash of the source code snippet for deduplication purposes.
- **Documented Check (`is_documented`):** Determines if a specific code snippet has already been documented.
- **Existing Documentation Retrieval (`get_existing_doc`):** Retrieves existing documentation records based on the provided `CodeSnippet`.
- **Redocumentation Decision (`should_document`):** Decides whether to document a new or updated code snippet, considering deduplication and redocumentation intervals.

**Design Patterns:**
The class employs several design patterns:
- **Factory Pattern:** `_get_conn` method ensures that connections are properly managed.
- **Strategy Pattern:** The `DocType` parameter in `should_document` allows different types of documentation to be handled uniformly.

**Relationship to Other Classes:**
`StateManager` interacts with the `CodeSnippet` and `DocumentedSnippet` classes. It is a central component for managing state in the CodeWorm application, ensuring that code snippets are documented efficiently without redundancy. This class fits into the architecture by providing persistent storage and deduplication logic, supporting other components like the main daemon or documentation generator.

This design ensures that the StateManager is responsible for maintaining the integrity of the database schema and handling complex operations such as deduplication and redocumentation in a structured manner.

---

*Generated by CodeWorm on 2026-02-19 12:59*
