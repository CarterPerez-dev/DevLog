# CodeAnalyzer

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/analysis/analyzer.py
**Language:** python
**Lines:** 50-255
**Complexity:** 0.0

---

## Source Code

```python
class CodeAnalyzer:
    """
    Main code analysis engine
    Combines parsing, complexity analysis, and interest scoring
    """
    def __init__(
        self,
        repos: list[RepoEntry],
        settings: AnalyzerSettings | None = None,
    ) -> None:
        """
        Initialize analyzer with repository configurations
        """
        self.repos = repos
        self.settings = settings
        self.repo_selector = WeightedRepoSelector(repos)
        self.scanner = RepoScanner(
            include_patterns = settings.include_patterns if settings else None,
            exclude_patterns = settings.exclude_patterns if settings else None,
        )
        self.complexity_analyzer = ComplexityAnalyzer()
        self.scorer = InterestScorer()

        self._git_repos: dict[Path, Repo | None] = {}

        ParserManager.initialize()

    def _get_git_repo(self, repo_path: Path) -> Repo | None:
        """
        Get or create git repo instance for a path
        """
        if repo_path not in self._git_repos:
            try:
                self._git_repos[repo_path] = Repo(repo_path)
            except InvalidGitRepositoryError:
                self._git_repos[repo_path] = None
        return self._git_repos[repo_path]

    def analyze_file(self,
                     scanned_file: ScannedFile) -> Iterator[AnalysisCandidate]:
        """
        Analyze a single file and yield documentation candidates
        """
        try:
            source = scanned_file.path.read_text(encoding = "utf-8")
        except Exception:
            return

        extractor = CodeExtractor(source, scanned_file.language)
        complexity_results = self.complexity_analyzer.analyze_source(
            source,
            str(scanned_file.path),
        )
        complexity_map = {m.name: m for m in complexity_results}

        git_repo = self._get_git_repo(scanned_file.path.parent)
        if git_repo:
            self.scorer.git_repo = git_repo

        for parsed_func in extractor.extract_functions():
            if self._should_skip_function(parsed_func):
                continue

            complexity = complexity_map.get(parsed_func.name)
            if not complexity:
                for name, metrics in complexity_map.items():
                    if name.endswith(f".{parsed_func.name}"):
                        complexity = metrics
                        break

            git_stats = self.scorer.get_git_stats(
                scanned_file.path,
                parsed_func.start_line,
                parsed_func.end_line,
            )

            if complexity:
                interest = self.scorer.score(
                    complexity,
                    git_stats,
                    parsed_func.decorators,
                    parsed_func.is_async,
                    parsed_func.source,
                )
            else:
                interest = InterestScore(
                    total = 20,
                    complexity_score = 0,
    
```

---

## Class Documentation

### CodeAnalyzer Class Documentation

**Class Responsibility and Purpose:**
The `CodeAnalyzer` class serves as the central engine for code analysis, integrating various components to perform comprehensive code parsing, complexity analysis, and interest scoring. It processes repositories by analyzing files, functions, and their associated metrics.

**Public Interface (Key Methods):**
- **`__init__(repos: list[RepoEntry], settings: AnalyzerSettings | None = None)`**: Initializes the analyzer with repository configurations.
- **`analyze_file(scanned_file: ScannedFile) -> Iterator[AnalysisCandidate]`**: Analyzes a single file and yields documentation candidates.

**Design Patterns Used:**
- **Factory Pattern**: `ParserManager.initialize()` initializes parsers, adhering to the factory pattern by providing concrete implementations of parsing logic.
- **Strategy Pattern**: The `ComplexityAnalyzer` and `InterestScorer` classes use strategies for different analysis methods, allowing flexible scoring mechanisms based on varying criteria.

**Relationship to Other Classes:**
- **RepoEntry**: Represents repository configurations.
- **AnalyzerSettings**: Contains settings for analysis parameters like minimum line count.
- **WeightedRepoSelector**: Selects repositories based on weighted criteria.
- **RepoScanner**: Scans files based on inclusion and exclusion patterns.
- **ComplexityAnalyzer**: Analyzes code complexity metrics.
- **InterestScorer**: Scores functions based on various factors including complexity, git history, and function name.

**State Management Approach:**
The class manages state through instance variables like `repos`, `settings`, `repo_selector`, `scanner`, `complexity_analyzer`, and `scorer`. It also maintains a dictionary `_git_repos` to cache Git repository instances for performance optimization.

---

*Generated by CodeWorm on 2026-02-19 12:51*
