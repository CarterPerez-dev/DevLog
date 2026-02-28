# EvolutionTargetFinder

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/analysis/targets.py
**Language:** python
**Lines:** 356-460
**Complexity:** 0.0

---

## Source Code

```python
class EvolutionTargetFinder:
    """
    Finds recently changed files and generates git diff context
    """
    def find(
        self,
        repo: RepoEntry,
        limit: int = 10,
    ) -> list[DocumentationTarget]:
        targets: list[DocumentationTarget] = []

        try:
            git_repo = Repo(repo.path)
        except InvalidGitRepositoryError:
            return targets

        try:
            commits = list(git_repo.iter_commits(max_count = 20))
        except Exception:
            return targets

        seen_files: set[str] = set()

        for i, commit in enumerate(commits[:-1]):
            parent = commits[i + 1]
            try:
                diffs = parent.diff(commit, create_patch = True)
            except Exception:  # noqa: S112
                continue

            for diff in diffs:
                file_path = diff.b_path or diff.a_path
                if not file_path or file_path in seen_files:
                    continue

                ext = Path(file_path).suffix.lower()
                language = LANGUAGE_EXTENSIONS.get(ext)
                if not language:
                    continue

                seen_files.add(file_path)

                try:
                    diff_text = diff.diff.decode("utf-8", errors = "replace")
                except Exception:  # noqa: S112
                    continue

                if len(diff_text) < 20:
                    continue

                full_path = repo.path / file_path
                score = min(
                    100.0,
                    (
                        min(len(diff_text) / 1000,
                            1.0) * 40 + 30 + (10 if diff.new_file else 0) +
                        min(diff_text.count("\n+") / 20,
                            1.0) * 20
                    )
                )

                snippet = CodeSnippet(
                    repo = repo.name,
                    file_path = full_path,
                    function_name = None,
                    class_name = None,
                    language = language,
                    source = diff_text[: 4000],
                    start_line = 1,
                    end_line = 1,
                    interest_score = score,
                    doc_type = DocType.CODE_EVOLUTION,
                )

                context = (
                    f"Commit: {commit.hexsha[:8]}\n"
                    f"Message: {commit.message.strip()}\n"
                    f"Author: {commit.author}\n"
                    f"File: {file_path}\n"
                    f"Change type: {'new file' if diff.new_file else 'modified'}\n\n"
                    f"Diff:\n{diff_text[:5000]}"
                )

                targets.append(
                    DocumentationTarget(
                        doc_type = DocType.CODE_EVOLUTION,
                        snippet = snippet,
                        source_context = context[: 6000],
                        metadata = {
                            "comm
```

---

## Class Documentation

### EvolutionTargetFinder

**Class Responsibility and Purpose:**  
The `EvolutionTargetFinder` class is responsible for identifying recently changed files within a Git repository and generating context around these changes to create documentation targets. This helps in understanding the evolution of code by analyzing recent commits.

**Public Interface (Key Methods):**
- **`find(repo: RepoEntry, limit: int = 10) -> list[DocumentationTarget]`:**  
  This method takes a `RepoEntry` object representing the repository path and an optional limit for the number of targets to return. It returns a list of `DocumentationTarget` objects that represent potential documentation candidates.

**Design Patterns Used:**
- **Strategy Pattern:** The class uses the strategy pattern implicitly through the handling of different file types, where the language is determined based on file extensions.
- **Factory Method:** Although not explicitly implemented as a separate class, the creation and processing of `CodeSnippet` and `DocumentationTarget` objects can be seen as factory methods.

**How it Fits in the Architecture:**
The `EvolutionTargetFinder` class plays a crucial role in the CodeWorm analysis module by providing insights into code evolution. It interfaces with the Git repository through the `RepoEntry` object, leveraging Git commands to fetch recent commits and diffs. The generated targets are then used by other parts of the system for further processing or documentation generation.

This class adheres to Pythonic practices such as using type hints, handling exceptions gracefully, and employing context managers where necessary. It ensures that only relevant changes are considered, thus maintaining a focus on significant code modifications.

---

*Generated by CodeWorm on 2026-02-28 17:46*
