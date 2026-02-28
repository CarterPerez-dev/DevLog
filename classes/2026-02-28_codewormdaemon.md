# CodeWormDaemon

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/daemon.py
**Language:** python
**Lines:** 125-734
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
            if await client.health_check():
                if self.stats.consecutive_ollama_failures > 0:
                    self.l
```

---

## Class Documentation

### CodeWormDaemon Documentation

**Class Responsibility and Purpose:**
The `CodeWormDaemon` class is the orchestrator for a 24/7 operation, handling core functionalities such as analysis, LLM generation, git commits, and scheduling. It ensures resilience by self-healing when Ollama (an AI service) goes down and backs off on failures to maintain continuous operation.

**Public Interface:**
- `__init__(self, settings: CodeWormSettings, dry_run: bool = False)`: Initializes the daemon with necessary components.
- `_setup_signals(self)`: Sets up signal handlers for graceful shutdown.
- `_handle_shutdown(self, signum: int, _frame)`: Handles SIGTERM and SIGINT signals to shut down gracefully.
- `_handle_reload(self, _signum: int, _frame)`: Handles SIGHUP for config reload.
- `run(self)`: Main daemon run loop.
- `_async_run(self)`: Async main run loop.

**Design Patterns Used:**
- **Observer Pattern**: The `CodeWormDaemon` observes and reacts to changes in the state of its components, such as when Ollama becomes available or unavailable.
- **Strategy Pattern**: Uses the `OllamaClient` strategy for handling AI operations with different implementations if needed.

**Relationship to Other Classes:**
- **StateManager**: Manages the state of the daemon across restarts.
- **DevLogRepository**: Handles git commit operations.
- **CodeAnalyzer**: Analyzes code repositories.
- **TargetRouter**: Routes analysis targets based on predefined rules.
- **CodeWormScheduler**: Schedules tasks and runs them at appropriate times.

**State Management Approach:**
The `CodeWormDaemon` manages its state through the `state` attribute, which uses a `StateManager` to persistently store and retrieve data. This ensures that the daemon can resume operations from where it left off after restarts or failures.

---

*Generated by CodeWorm on 2026-02-28 07:45*
