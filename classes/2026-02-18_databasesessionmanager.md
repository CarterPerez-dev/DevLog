# DatabaseSessionManager

**Type:** Class Documentation
**Repository:** my-portfolio
**File:** v1/backend/app/core/database.py
**Language:** python
**Lines:** 27-148
**Complexity:** 0.0

---

## Source Code

```python
class DatabaseSessionManager:
    """
    Manages database connections and sessions for both sync and async contexts
    """
    def __init__(self) -> None:
        self._async_engine: AsyncEngine | None = None
        self._sync_engine: Engine | None = None
        self._async_sessionmaker: async_sessionmaker[AsyncSession] | None = None
        self._sync_sessionmaker: sessionmaker[Session] | None = None

    def init(self, database_url: str) -> None:
        """
        Initialize database engines and session factories
        """
        base_url = make_url(database_url)

        async_url = base_url.set(drivername = "postgresql+asyncpg")
        self._async_engine = create_async_engine(
            async_url,
            pool_size = settings.DB_POOL_SIZE,
            max_overflow = settings.DB_MAX_OVERFLOW,
            pool_timeout = settings.DB_POOL_TIMEOUT,
            pool_recycle = settings.DB_POOL_RECYCLE,
            pool_pre_ping = True,
            echo = settings.DEBUG,
        )
        self._async_sessionmaker = async_sessionmaker(
            bind = self._async_engine,
            class_ = AsyncSession,
            autocommit = False,
            autoflush = False,
            expire_on_commit = False,
        )

        sync_url = base_url.set(drivername = "postgresql+psycopg2")
        self._sync_engine = create_engine(
            sync_url,
            pool_size = settings.DB_POOL_SIZE,
            max_overflow = settings.DB_MAX_OVERFLOW,
            pool_timeout = settings.DB_POOL_TIMEOUT,
            pool_recycle = settings.DB_POOL_RECYCLE,
            pool_pre_ping = True,
            echo = settings.DEBUG,
        )
        self._sync_sessionmaker = sessionmaker(
            bind = self._sync_engine,
            autocommit = False,
            autoflush = False,
            expire_on_commit = False,
        )

    async def close(self) -> None:
        """
        Dispose of all database connections
        """
        if self._async_engine:
            await self._async_engine.dispose()
            self._async_engine = None
            self._async_sessionmaker = None

        if self._sync_engine:
            self._sync_engine.dispose()
            self._sync_engine = None
            self._sync_sessionmaker = None

    @contextlib.asynccontextmanager
    async def session(self) -> AsyncIterator[AsyncSession]:
        """
        Async context manager for database sessions

        Handles commit on success, rollback on exception
        """
        if self._async_sessionmaker is None:
            raise RuntimeError("DatabaseSessionManager is not initialized")

        session = self._async_sessionmaker()
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

    @contextlib.asynccontextmanager
    async def connect(self) -> AsyncIterator[AsyncConnection]:
        """
     
```

---

## Class Documentation

### DatabaseSessionManager

**Class Responsibility and Purpose:**
The `DatabaseSessionManager` class manages database connections and sessions for both synchronous and asynchronous contexts, ensuring efficient resource management and consistent session handling.

**Public Interface (Key Methods):**
- **`init(database_url: str) -> None`:** Initializes the database engines and session factories using the provided database URL.
- **`close() -> None`:** Closes all active database connections.
- **`session() -> AsyncIterator[AsyncSession]`:** An asynchronous context manager for managing database sessions, handling commit on success and rollback on exception.
- **`connect() -> AsyncIterator[AsyncConnection]`:** An asynchronous context manager for raw database connections.
- **`sync_engine: Engine`:** A property providing the synchronous engine for Alembic migrations.
- **`sync_session() -> Iterator[Session]`:** A synchronous context manager for migrations and CLI tools.

**Design Patterns Used:**
- **Factory Pattern:** The `init()` method initializes the database engines and session factories, effectively acting as a factory to create and manage database resources.
- **Context Manager:** Both `session()` and `connect()` methods use Python's `contextlib.contextmanager` decorator to provide context management for database sessions and connections.

**Relationship to Other Classes:**
This class is central in the application’s architecture, providing a unified interface for managing database interactions. It interacts with other components such as models, services, and migrations by supplying necessary session objects. The `DatabaseSessionManager` ensures that all database operations are performed within managed contexts, promoting better resource management and consistency.

**State Management Approach:**
The class maintains state through instance variables (`_async_engine`, `_sync_engine`, etc.) to keep track of the database connections and session factories. These states are initialized during the `init()` method and disposed of in the `close()` method, ensuring that resources are properly released when no longer needed.

---

*Generated by CodeWorm on 2026-02-18 19:44*
