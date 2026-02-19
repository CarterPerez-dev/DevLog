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
            except Exception:
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
                except Exception:
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
                            "commit_hash": commit.hexsha[: 8]
```

---

## Class Documentation

### EvolutionTargetFinder Documentation

**Class Responsibility and Purpose:**
The `EvolutionTargetFinder` class is responsible for identifying recently changed files in a Git repository and generating context around those changes to facilitate code documentation. It leverages Git commands to find commits, diff them, and extract relevant snippets of code that have been modified or added.

**Public Interface (Key Methods):**
- **find(repo: RepoEntry, limit: int = 10) -> list[DocumentationTarget]:** This method takes a `RepoEntry` object representing the repository path and an optional limit for the number of targets to return. It returns a list of `DocumentationTarget` objects containing code snippets and context.

**Design Patterns Used:**
- **Strategy Pattern:** The class uses the strategy pattern implicitly by handling different file extensions through the `LANGUAGE_EXTENSIONS` dictionary.
- **Observer Pattern:** Although not explicitly implemented, the class could be extended to observe changes in repositories using Git events or hooks.

**How It Fits in the Architecture:**
The `EvolutionTargetFinder` class is part of a larger code analysis system within the CodeWorm project. It interfaces with the repository layer (via `RepoEntry`) and provides targets for documentation generation. The method `find` is called by higher-level components to gather relevant code snippets, which are then used in automated documentation processes or other analyses.

This class ensures that only significant changes are considered, filtering out minor diffs and focusing on files that have been modified or newly created. It maintains a state of seen files to avoid redundant processing and sorts the targets based on their interest score, ensuring the most relevant snippets are prioritized.

---

*Generated by CodeWorm on 2026-02-19 13:47*
