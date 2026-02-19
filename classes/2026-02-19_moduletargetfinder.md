# ModuleTargetFinder

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/analysis/targets.py
**Language:** python
**Lines:** 198-353
**Complexity:** 0.0

---

## Source Code

```python
class ModuleTargetFinder:
    """
    Finds Python packages or directory-level modules to document
    """
    def find(
        self,
        repo: RepoEntry,
        limit: int = 10,
    ) -> list[DocumentationTarget]:
        targets: list[DocumentationTarget] = []

        if not repo.path.exists():
            return targets

        for dirpath in repo.path.rglob("__init__.py"):
            pkg_dir = dirpath.parent
            rel_dir = pkg_dir.relative_to(repo.path)

            skip_dirs = {
                "node_modules",
                ".git",
                "venv",
                ".venv",
                "__pycache__",
                "dist",
                "build",
                "vendor",
                "target",
                ".tox",
                ".mypy_cache"
            }
            if any(part in skip_dirs for part in rel_dir.parts):
                continue

            py_files = list(pkg_dir.glob("*.py"))
            file_count = len(py_files)

            if file_count < 2:
                continue

            init_content = ""
            try:
                init_content = dirpath.read_text(encoding = "utf-8")
            except Exception:
                pass

            file_listing = "\n".join(f"  - {f.name}" for f in sorted(py_files))
            context = f"Package: {rel_dir}\nFiles ({file_count}):\n{file_listing}"

            if init_content.strip():
                context += f"\n\n__init__.py:\n{init_content[:2000]}"

            score = min(
                100.0,
                (
                    min(file_count / 8,
                        1.0) * 40 + min(len(init_content) / 500,
                                        1.0) * 30 + 30
                )
            )

            snippet = CodeSnippet(
                repo = repo.name,
                file_path = pkg_dir,
                function_name = None,
                class_name = None,
                language = Language.PYTHON,
                source = context[: 4000],
                start_line = 1,
                end_line = 1,
                interest_score = score,
                doc_type = DocType.MODULE_DOC,
            )

            targets.append(
                DocumentationTarget(
                    doc_type = DocType.MODULE_DOC,
                    snippet = snippet,
                    source_context = context[: 6000],
                    metadata = {
                        "package_path": str(rel_dir),
                        "file_count": file_count,
                        "file_names": [f.name for f in py_files],
                        "has_init_content": bool(init_content.strip()),
                    },
                )
            )

            if len(targets) >= limit:
                break

        for dirpath in repo.path.rglob("index.ts"):
            pkg_dir = dirpath.parent
            rel_dir = pkg_dir.relative_to(repo.path)

            skip_dirs = {"node_modules", ".git", "dist", "build"}
            if
```

---

## Class Documentation

### ModuleTargetFinder

**Class Responsibility and Purpose:**
The `ModuleTargetFinder` class is responsible for identifying Python packages or directory-level modules within a repository that should be documented. It evaluates directories based on specific criteria, such as the presence of multiple `.py` files and non-empty `__init__.py` content, to determine their documentation priority.

**Public Interface:**
- **find(repo: RepoEntry, limit: int = 10) -> list[DocumentationTarget]:** This method searches through a repository's directory structure for potential modules to document. It returns up to the specified limit of `DocumentationTarget` objects, sorted by their interest score.

**Design Patterns Used:**
The class employs several design patterns:
- **Factory Method:** Implicitly used in creating `DocumentationTarget` and `CodeSnippet` instances.
- **Strategy:** The scoring logic can be seen as a strategy for evaluating the importance of each module based on file counts and content.

**Relationship to Other Classes:**
- **RepoEntry:** Represents a repository entry, providing access to its path and other metadata.
- **DocumentationTarget:** Holds information about potential documentation targets within the repository.
- **CodeSnippet:** Provides context snippets for documentation purposes.

**State Management Approach:**
The class maintains state through local variables like `targets` and iterates over directories using `rglob`, ensuring that only relevant modules are considered. The scoring mechanism dynamically assigns a priority score to each target, influencing their order in the final list of targets returned by the `find` method.

This class plays a crucial role in the CodeWorm analysis pipeline, helping to identify and prioritize documentation tasks based on module complexity and relevance.

---

*Generated by CodeWorm on 2026-02-19 13:40*
