# CodeWormDaemon

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/daemon.py
**Language:** python
**Lines:** 125-791
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
        self._start_time = datetime.now()
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
        self._dead_mans_task: asyncio.Task | None = None

        self.notifier: CodeWormNotifier | None = None
        if settings.telegram.enabled and settings.telegram.bot_token:
            self.notifier = CodeWormNotifier(settings.telegram)

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
            if await client.health
```

---

## Class Documentation

### CodeWormDaemon Documentation

**Class Responsibility and Purpose:**
The `CodeWormDaemon` class serves as the central orchestrator for a 24/7 operation, managing tasks such as code analysis, LLM generation, git commits, and scheduling. It ensures robust operation by self-healing when Ollama (a language model service) is unavailable and backing off on failures.

**Public Interface:**
- `__init__(self, settings: CodeWormSettings, dry_run: bool = False)`: Initializes the daemon with necessary components.
- `_setup_signals(self)`: Sets up signal handlers for graceful shutdown.
- `_handle_shutdown(self, signum: int, _frame)`: Handles SIGTERM and SIGINT signals to gracefully shut down the daemon.
- `_handle_reload(self, _signum: int, _frame)`: Handles SIGHUP for config reload.
- `run(self)`: The main method to start the daemon's operation.

**Design Patterns Used:**
- **Singleton**: Implicitly used through the `settings` and `logger`.
- **Observer**: The `notifier` observes changes and sends alerts when necessary.
- **Strategy**: The `target_router` uses strategies from the `analyzer`.

**Relationship to Other Classes:**
- **CodeAnalyzer**: Analyzes code repositories.
- **TargetRouter**: Routes targets for analysis based on the analyzer's output.
- **CodeWormScheduler**: Manages scheduling of tasks.
- **OllamaClient**: Manages communication with Ollama, a language model service.

**State Management Approach:**
The class uses instance variables to manage its state, such as `running`, `stats`, and `state`. It also employs context managers for cleanup (`_cleanup`) when shutting down. The `_ensure_ollama_ready` method ensures that the LLM client is ready before proceeding with operations, using exponential backoff on failures.

This design ensures a robust and maintainable architecture, handling complex interactions between components while providing a clear interface for external control.

---

*Generated by CodeWorm on 2026-02-28 12:51*
