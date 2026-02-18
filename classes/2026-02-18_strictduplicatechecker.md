# StrictDuplicateChecker

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/devtools/pytest/pylint_duplicate.py
**Language:** python
**Lines:** 19-125
**Complexity:** 0.0

---

## Source Code

```python
class StrictDuplicateChecker(BaseChecker):
    """Check for near-exact duplicates (99% similarity)"""

    name = "strict-duplicate"
    msgs = {
        "R9901": (
            "Near-exact duplicate code found with %s (similarity: %.2f%%)",
            "near-exact-duplicate",
            "Function is 99%% similar to another function. "
            "This usually indicates accidental copy-paste.",
        ),
    }

    options = (
        (
            "min-similarity-lines",
            {
                "default": 4,
                "type": "int",
                "metavar": "<int>",
                "help": "Minimum lines to be considered as duplicate",
            },
        ),
    )

    def __init__(self, linter: "PyLinter") -> None:
        super().__init__(linter)
        self._seen_files = set()

    def open(self):
        """Initialize the checker"""
        global _COLLECTED_FUNCTIONS
        # Only clear if this is the first file
        if not self._seen_files:
            _COLLECTED_FUNCTIONS = []

    def visit_module(self, node: nodes.Module) -> None:
        """Track which files we've seen"""
        if node.file not in self._seen_files:
            self._seen_files.add(node.file)

    def visit_functiondef(self, node: nodes.FunctionDef) -> None:
        """Visit function definitions and collect them"""
        global _COLLECTED_FUNCTIONS

        # Get the source code of the function
        try:
            source_lines = node.as_string().splitlines()
            # Normalize lines - remove empty lines and normalize whitespace
            normalized_lines = []
            for line in source_lines:
                stripped = line.strip()
                if stripped and not stripped.startswith('#'):  # Skip comments
                    normalized_lines.append(stripped)

            if len(normalized_lines) >= self.config.min_similarity_lines:
                func_info = {
                    'node': node,
                    'file': node.root().file,
                    'lines': normalized_lines,
                    'name': node.name,
                    'lineno': node.lineno
                }

                # Check against already collected functions
                for existing in _COLLECTED_FUNCTIONS:
                    if existing['name'] != func_info['name']:  # Skip same names
                        similarity = self._calculate_similarity(
                            existing['lines'],
                            func_info['lines']
                        )
                        if similarity >= 0.99:
                            # Report the duplicate
                            self.add_message(
                                "near-exact-duplicate",
                                node = node,
                                line = node.lineno,
                                args = (
                                    f"{existing['file']}:{existing['name']}",
                                    similarity * 100
      
```

---

## Class Documentation

### StrictDuplicateChecker Documentation

**Class Responsibility and Purpose:**
The `StrictDuplicateChecker` class is responsible for identifying near-exact duplicates in Python code with a similarity of 99%. This helps in detecting accidental copy-paste errors, ensuring code quality and maintainability.

**Public Interface (Key Methods):**
- **`__init__(self, linter: "PyLinter") -> None`:** Initializes the checker with a PyLint linter instance.
- **`open(self)`:** Initializes the checker by clearing collected functions if it's the first file being processed.
- **`visit_module(self, node: nodes.Module) -> None`:** Tracks which files have been seen.
- **`visit_functiondef(self, node: nodes.FunctionDef) -> None`:** Visits function definitions and collects them for comparison. Also handles async functions by delegating to `visit_functiondef`.
- **`_calculate_similarity(self, lines1: list[str], lines2: list[str]) -> float`:** Calculates the similarity between two code blocks using a sequence matcher.

**Design Patterns Used:**
- **Strategy Pattern:** The `_calculate_similarity` method uses a strategy for calculating similarity.
- **Observer Pattern:** The `visit_module` and `visit_functiondef` methods observe nodes in the AST, allowing the checker to react to changes in the codebase.

**How it Fits in the Architecture:**
This class is part of a larger code quality tool (PyLint) that processes Python source files. It integrates with PyLint’s visitor pattern to traverse the Abstract Syntax Tree (AST), identifying and reporting near-exact duplicate functions based on configurable similarity thresholds. The checker maintains state across multiple file visits, ensuring comprehensive coverage of the project.

---

*Generated by CodeWorm on 2026-02-18 13:41*
