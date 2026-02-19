# CodeWormDaemon

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/daemon.py
**Language:** python
**Lines:** 123-701
**Complexity:** 0.0

---

## Source Code

```python
class CodeWormDaemon:
    """
    Main daemon orchestrator - designed for bulletproof 24/7 operation
    Coordinates analysis, LLM generation, git commits, and scheduling
    Self-heals when Ollama goes down, backs off on failures, never dies
    """
    OLLAMA_WAIT_MAX = 300
    OLLAMA_CHECK_INTERVAL = 10

    def __init__(self, settings: CodeWormSettings, dry_run: bool = False) -> None:
        """
        Initialize daemon with all components
        """
        self.settings = settings
        self.dry_run = dry_run
        self.logger = get_logger("daemon")
        self.running = False

        self.stats = CycleStats()
        self.state = StateManager(settings.db_path)
        self.devlog = DevLogRepository(
            repo_path = settings.devlog.repo_path,
            remote = settings.devlog.remote,
            branch = settings.devlog.branch,
        )
        self.analyzer = CodeAnalyzer(
            repos = settings.repos,
            settings = settings.analyzer,
        )
        self.target_router = TargetRouter(
            analyzer = self.analyzer,
            scanner = self.analyzer.scanner,
        )
        self.scheduler = CodeWormScheduler(settings.schedule)
        self._llm_client: OllamaClient | None = None
        self._loop: asyncio.AbstractEventLoop | None = None

        self._setup_signals()

    def _setup_signals(self) -> None:
        """
        Set up signal handlers for graceful shutdown
        """
        signal.signal(signal.SIGTERM, self._handle_shutdown)
        signal.signal(signal.SIGINT, self._handle_shutdown)
        signal.signal(signal.SIGHUP, self._handle_reload)

    def _handle_shutdown(self, signum: int, _frame) -> None:
        """
        Handle shutdown signals gracefully
        """
        sig_name = signal.Signals(signum).name
        self.logger.info("shutdown_signal_received", signal = sig_name)
        self.running = False
        self.scheduler.stop(wait = False)

    def _handle_reload(self, _signum: int, _frame) -> None:
        """
        Handle SIGHUP for config reload
        """
        self.logger.info("reload_signal_received")

    async def _init_llm(self) -> OllamaClient:
        """
        Initialize the LLM client (does not prewarm here)
        """
        if self._llm_client is None:
            self._llm_client = OllamaClient(self.settings.ollama)
        return self._llm_client

    async def _wait_for_ollama(self) -> bool:
        """
        Wait for Ollama to become available with exponential backoff
        Returns True when available, False if daemon is shutting down
        """
        client = await self._init_llm()

        while self.running:
            if await client.health_check():
                if self.stats.consecutive_ollama_failures > 0:
                    self.logger.info(
                        "ollama_recovered",
                        after_failures = self.stats.consecutive_ollama_failures,
                    )
                self.stats.rec
```

---

## Class Documentation

### CodeWormDaemon Documentation

**Class Responsibility and Purpose:**
The `CodeWormDaemon` class serves as the main orchestrator for a 24/7 operation, handling tasks such as code analysis, LLM generation, git commits, and scheduling. It ensures robust operation by self-healing when Ollama (an AI service) goes down and backing off on failures.

**Public Interface:**
- **`__init__(self, settings: CodeWormSettings, dry_run: bool = False)`**: Initializes the daemon with necessary components.
- **`run(self)`**: Starts the main run loop of the daemon.
- **`_async_run(self) -> None`**: The asynchronous main run loop that handles the core operations.
- **`_init_llm(self) -> OllamaClient`**: Initializes the LLM client.
- **`_wait_for_ollama(self) -> bool`**: Waits for Ollama to become available with exponential backoff.
- **`_ensure_ollama_ready(self) -> bool`**: Ensures Ollama is ready, waiting if necessary.
- **`_cleanup(self) -> None`**: Cleans up resources on shutdown.

**Design Patterns Used:**
- **Observer Pattern**: The `CodeWormDaemon` observes and reacts to signals for graceful shutdowns.
- **Strategy Pattern**: The `_ensure_ollama_ready` method uses a strategy to handle Ollama availability, allowing different behaviors based on the current state.

**Relationship to Other Classes:**
- **`CodeAnalyzer`, `TargetRouter`, `DevLogRepository`, `CodeWormScheduler`**: These classes are initialized and used within the daemon for specific tasks.
- **`OllamaClient`**: Manages interactions with the AI service, ensuring it is available before proceeding.

**State Management Approach:**
The class uses a combination of instance variables (`self.running`, `self.stats`, `self.state`) to manage its state. It also employs an event loop and asynchronous methods to handle operations efficiently, ensuring smooth operation even under failure conditions.

---

*Generated by CodeWorm on 2026-02-19 12:27*
