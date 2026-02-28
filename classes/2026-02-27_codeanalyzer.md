# CodeAnalyzer

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/analysis/analyzer.py
**Language:** python
**Lines:** 50-277
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
        if repo_path in self._git_repos:
            return self._git_repos[repo_path]
        try:
            repo = Repo(repo_path, search_parent_directories=True)
            git_root = Path(repo.working_dir)
            if git_root in self._git_repos:
                repo.close()
                self._git_repos[repo_path] = self._git_repos[git_root]
            else:
                self._git_repos[git_root] = repo
                self._git_repos[repo_path] = repo
        except InvalidGitRepositoryError:
            self._git_repos[repo_path] = None
        return self._git_repos[repo_path]

    def close_repos(self) -> None:
        """
        Close all cached git repo handles and clear the cache
        """
        seen = set()
        for repo in self._git_repos.values():
            if repo is not None and id(repo) not in seen:
                seen.add(id(repo))
                try:
                    repo.close()
                except Exception:
                    pass
        self._git_repos.clear()

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
                for name, 
```

---

## Class Documentation

### CodeAnalyzer Class Documentation

**Class Responsibility and Purpose:**
The `CodeAnalyzer` class serves as the central engine for code analysis, combining parsing, complexity analysis, and interest scoring to identify significant code snippets within a set of repositories.

**Public Interface (Key Methods):**
- **`__init__(repos: list[RepoEntry], settings: AnalyzerSettings | None = None)`**: Initializes the analyzer with repository configurations.
- **`_get_git_repo(repo_path: Path) -> Repo | None`**: Retrieves or creates a Git repository instance for a given path.
- **`close_repos()`**: Closes all cached git repo handles and clears the cache.
- **`analyze_file(scanned_file: ScannedFile) -> Iterator[AnalysisCandidate]`**: Analyzes a single file, yielding documentation candidates.

**Design Patterns Used:**
- **Factory Pattern**: `ParserManager.initialize()` initializes necessary parsers.
- **Strategy Pattern**: The `ComplexityAnalyzer` and `InterestScorer` classes implement different strategies for analyzing code complexity and scoring interest.

**Relationship to Other Classes:**
- **RepoEntry, AnalyzerSettings**: Configuration data for repositories and analysis settings.
- **WeightedRepoSelector, RepoScanner**: Selects and scans relevant repository entries based on patterns.
- **ComplexityAnalyzer, InterestScorer**: Analyze code complexity and score interest in functions.
- **CodeExtractor, CodeSnippet, AnalysisCandidate**: Extract and represent code snippets and their analysis results.

**State Management Approach:**
The class maintains a dictionary `_git_repos` to cache Git repository instances, ensuring efficient access. The `close_repos()` method manages the lifecycle of these cached repositories by closing them when necessary and clearing the cache.

---

*Generated by CodeWorm on 2026-02-27 20:10*
