# DatabasePool

**Type:** Class Documentation
**Repository:** vuemantics
**File:** backend/database.py
**Language:** python
**Lines:** 21-301
**Complexity:** 0.0

---

## Source Code

```python
class DatabasePool:
    """
    Manages PostgreSQL connection pool with pgvector support

    Provides methods for executing queries, managing transactions,
    and handling vector operations efficiently
    """
    def __init__(self) -> None:
        """
        Initialize database pool instance
        """
        self._pool: Pool | None = None
        self._lock = asyncio.Lock()

    async def connect(self) -> None:
        """
        Initialize connection pool with pgvector extension
        """
        if self._pool is not None:
            return

        async with self._lock:
            if self._pool is not None:
                return  # type: ignore[unreachable]

            try:
                self._pool = await asyncpg.create_pool(
                    config.settings.database_url,
                    min_size = config.settings.db_pool_min_size,
                    max_size = config.settings.db_pool_max_size,
                    command_timeout = config.settings.db_command_timeout,
                    timeout = config.settings.db_pool_timeout,
                    init = self._init_connection,
                )

                await self._verify_pgvector()

                logger.info(
                    "Database pool created successfully",
                    extra = {
                        "min_size": config.settings.db_pool_min_size,
                        "max_size": config.settings.db_pool_max_size,
                    },
                )

            except Exception as e:
                logger.error(f"Failed to create database pool: {e}")
                raise

    async def disconnect(self) -> None:
        """
        Close all connections in the pool
        """
        if self._pool is not None:
            await self._pool.close()
            self._pool = None
            logger.info("Database pool closed")

    async def _init_connection(self, conn: Connection) -> None:
        """
        Initialize individual connection with vector codec
        """
        with suppress(asyncpg.PostgresError, ValueError):
            await conn.set_type_codec(
                "vector",
                encoder = lambda v: f"[{','.join(map(str, v))}]",
                decoder = lambda v: list(map(float, v[1 :-1].split(","))),
                schema = "public",
            )

    async def _verify_pgvector(self) -> None:
        """
        Verify required extensions are installed and create if needed
        Then register vector type codec for all connections
        """
        async with self.acquire() as conn:
            try:
                await conn.execute(
                    'CREATE EXTENSION IF NOT EXISTS "uuid-ossp";'
                )
                logger.info("uuid-ossp extension created successfully")
            except asyncpg.PostgresError as e:
                raise RuntimeError(
                    f"uuid-ossp extension is required but could not be created: {e}"
                ) from e

            result =
```

---

## Class Documentation

### DatabasePool Class Documentation

**Class Responsibility and Purpose:**
The `DatabasePool` class manages a PostgreSQL connection pool with support for the pgvector extension, providing efficient vector operations and query execution capabilities. It ensures that connections are properly initialized, transactions are handled atomically, and resources are managed efficiently.

**Public Interface (Key Methods):**
- **`__init__()`**: Initializes the database pool instance.
- **`connect()`**: Initializes the connection pool with the pgvector extension.
- **`disconnect()`**: Closes all connections in the pool.
- **`_init_connection(conn: Connection) -> None`**: Initializes individual connections with vector codec support.
- **`_verify_pgvector() -> None`**: Verifies and installs required extensions, then registers vector type codecs for all connections.
- **`_register_vector_types() -> None`**: Registers vector type codecs for all connections in the pool.
- **`acquire() -> AsyncIterator[Connection]`**: Acquires a connection from the pool.
- **`transaction() -> AsyncIterator[Connection]`**: Manages database transactions for atomic operations (future implementation).
- **`execute(query: str, *args: Any, timeout: float | None = None) -> str`**: Executes a query with optional arguments and timeout.

**Design Patterns Used:**
- **Factory Pattern**: Implicitly used through the `create_pool` method to initialize the connection pool.
- **Observer Pattern**: Not explicitly implemented but could be inferred from event handling for connection initialization and state changes.
- **Strategy Pattern**: The vector codec registration is handled by a strategy pattern, allowing different codecs to be registered based on requirements.

**Relationship to Other Classes:**
The `DatabasePool` class interacts with the configuration settings (`config.settings`) and the `asyncpg` library. It is part of the backend database management system and provides essential services for other components that require database access, such as data storage and retrieval operations in VuexMantics.

This class ensures robust connection management and efficient query execution, making it a critical component of the application's architecture.

---

*Generated by CodeWorm on 2026-02-19 00:59*
