# RepoScanner

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/analysis/scanner.py
**Language:** python
**Lines:** 103-252
**Complexity:** 0.0

---

## Source Code

```python
class RepoScanner:
    """
    Scans repositories to find code files for analysis
    """
    DEFAULT_EXCLUDE: ClassVar[list[str]] = [
        "**/test_*.py",
        "**/*_test.py",
        "**/*_test.go",
        "**/*.spec.ts",
        "**/*.test.ts",
        "**/*.test.js",
        "**/tests/**",
        "**/test/**",
        "**/__tests__/**",
        "**/node_modules/**",
        "**/vendor/**",
        "**/dist/**",
        "**/build/**",
        "**/.git/**",
    ]

    MAX_FILE_SIZE = 1024 * 1024
    BINARY_CHECK_BYTES = 8192

    def __init__(
        self,
        include_patterns: list[str] | None = None,
        exclude_patterns: list[str] | None = None,
    ) -> None:
        """
        Initialize scanner with file patterns
        """
        self.include_patterns = include_patterns or [
            "**/*.py",
            "**/*.ts",
            "**/*.go",
            "**/*.rs",
            "**/*.js"
        ]
        self.exclude_patterns = exclude_patterns or self.DEFAULT_EXCLUDE
        self._exclude_spec = pathspec.PathSpec.from_lines(
            "gitwildmatch",
            self.exclude_patterns
        )

    def scan_repo(self, repo_path: Path, repo_name: str) -> Iterator[ScannedFile]:
        """
        Scan a repository and yield discovered files
        """
        if not repo_path.exists():
            return

        gitignore_filter = GitignoreFilter(repo_path)
        include_spec = pathspec.PathSpec.from_lines(
            "gitwildmatch",
            self.include_patterns
        )

        for root, dirs, files in os.walk(repo_path):
            root_path = Path(root)

            dirs[:] = [
                d for d in dirs if not d.startswith(".")
                and not gitignore_filter.is_ignored(root_path / d)
            ]

            for filename in files:
                file_path = root_path / filename

                try:
                    rel_path = file_path.relative_to(repo_path)
                except ValueError:
                    continue

                if not include_spec.match_file(str(rel_path)):
                    continue

                if self._exclude_spec.match_file(str(rel_path)):
                    continue

                if gitignore_filter.is_ignored(file_path):
                    continue

                ext = file_path.suffix.lower()
                language = LANGUAGE_EXTENSIONS.get(ext)
                if not language:
                    continue

                try:
                    stat = file_path.stat()
                    if stat.st_size > self.MAX_FILE_SIZE:
                        continue
                    if stat.st_size == 0:
                        continue

                    is_binary = self._is_binary(file_path)
                    if is_binary:
                        continue

                    yield ScannedFile(
                        path = file_path,
                        language = language,
                        repo_name = repo_name,
```

---

## Class Documentation

### RepoScanner Class Documentation

**Class Responsibility and Purpose:**
The `RepoScanner` class is responsible for scanning repositories to find code files suitable for analysis. It filters out non-code files, binary files, and files specified by exclusion patterns.

**Public Interface (Key Methods):**
- **`__init__(self, include_patterns: list[str] | None = None, exclude_patterns: list[str] | None = None)`**: Initializes the scanner with file inclusion and exclusion patterns.
- **`scan_repo(self, repo_path: Path, repo_name: str) -> Iterator[ScannedFile]`**: Scans a repository and yields discovered files that match the specified criteria.
- **`_is_binary(self, file_path: Path) -> bool`**: Checks if a file appears to be binary based on its content.
- **`get_repo_stats(self, repo_path: Path, repo_name: str) -> RepoStats`**: Collects statistics about a repository, including the total number of files and their sizes.

**Design Patterns Used:**
- **Factory Pattern**: Implicitly used through `RepoScanner` initialization with customizable patterns.
- **Strategy Pattern**: The `_is_binary` method uses a strategy to determine if a file is binary based on its content.

**Relationship to Other Classes:**
- **ScannedFile**: Represents an individual file discovered during the scan, containing metadata like path, language, and size.
- **PathSpec**: Used for pattern matching against file paths.
- **GitignoreFilter**: Filters out files ignored by `.gitignore` rules.
- **RepoStats**: Collects statistics about the scanned repository.

**State Management Approach:**
The class maintains state through instance variables such as `include_patterns`, `exclude_patterns`, and `_exclude_spec`. These are used to filter and process files during scanning. The `scan_repo` method iterates over directory contents, applying these filters to determine which files to include or exclude from the analysis.

This class fits into the architecture by providing a core functionality for repository scanning, which can be integrated with other parts of the system for code analysis and reporting.

---

*Generated by CodeWorm on 2026-02-19 13:21*
