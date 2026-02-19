# ComplexityAnalyzer

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/analysis/complexity.py
**Language:** python
**Lines:** 99-230
**Complexity:** 0.0

---

## Source Code

```python
class ComplexityAnalyzer:
    """
    Analyzes code complexity using Lizard
    """
    def __init__(self, language: Language | None = None) -> None:
        """
        Initialize analyzer optionally filtered to a language
        """
        self.language = language
        self._extensions = self._get_extensions()

    def _get_extensions(self) -> list[str]:
        """
        Get file extensions for the configured language
        """
        ext_map = {
            Language.PYTHON: [".py"],
            Language.TYPESCRIPT: [".ts"],
            Language.TSX: [".tsx"],
            Language.JAVASCRIPT: [".js",
                                  ".jsx"],
            Language.GO: [".go"],
            Language.RUST: [".rs"],
        }
        if self.language:
            return ext_map.get(self.language, [])
        return [ext for exts in ext_map.values() for ext in exts]

    def analyze_source(self,
                       source: str,
                       filename: str = "source.py") -> list[ComplexityMetrics]:
        """
        Analyze complexity of source code string
        """
        analysis = lizard.analyze_file.analyze_source_code(filename, source)
        return self._convert_functions(analysis.function_list)

    def analyze_file(self, file_path: Path) -> FileComplexity:
        """
        Analyze complexity of a single file
        """
        analysis = lizard.analyze_file(str(file_path))
        functions = self._convert_functions(analysis.function_list)

        if not functions:
            return FileComplexity(
                file_path = file_path,
                functions = [],
                average_complexity = 0.0,
                total_nloc = analysis.nloc,
                max_complexity = 0,
                num_functions = 0,
            )

        total_cc = sum(f.cyclomatic_complexity for f in functions)
        max_cc = max(f.cyclomatic_complexity for f in functions)

        return FileComplexity(
            file_path = file_path,
            functions = functions,
            average_complexity = total_cc / len(functions),
            total_nloc = analysis.nloc,
            max_complexity = max_cc,
            num_functions = len(functions),
        )

    def analyze_directory(self,
                          directory: Path,
                          recursive: bool = True) -> Iterator[FileComplexity]:
        """
        Analyze all supported files in a directory
        """
        pattern = "**/*" if recursive else "*"
        for ext in self._extensions:
            for file_path in directory.glob(f"{pattern}{ext}"):
                if file_path.is_file():
                    try:
                        yield self.analyze_file(file_path)
                    except Exception as e:
                        logger.debug(
                            "file_analysis_failed",
                            file = str(file_path),
                            error = str(e)
                        )
               
```

---

## Class Documentation

### ComplexityAnalyzer

#### Responsibility and Purpose
The `ComplexityAnalyzer` class is responsible for analyzing code complexity using the Lizard tool. It supports multiple programming languages by filtering files based on their extensions. The class provides methods to analyze individual source strings, single files, directories, and identify hotspots of complex functions.

#### Public Interface (Key Methods)
- **`__init__(self, language: Language | None = None)`**: Initializes the analyzer optionally filtered to a specific programming language.
- **`analyze_source(self, source: str, filename: str = "source.py") -> list[ComplexityMetrics]`**: Analyzes complexity of a given source code string.
- **`analyze_file(self, file_path: Path) -> FileComplexity`**: Analyzes the complexity of a single file.
- **`analyze_directory(self, directory: Path, recursive: bool = True) -> Iterator[FileComplexity]`**: Analyzes all supported files in a specified directory recursively or not.
- **`get_hotspots(self, directory: Path, min_complexity: int = 10, top_n: int = 20) -> list[tuple[Path, ComplexityMetrics]]`**: Identifies the most complex functions within a directory.

#### Design Patterns Used
The class employs the **Strategy Pattern** through the `language` parameter to support different programming languages. It also uses **Iterator** for the `analyze_directory` method, allowing it to yield results incrementally.

#### Relationship to Other Classes
- **`Language`**: Enum defining supported programming languages.
- **`ComplexityMetrics`**: Data class holding metrics of a function.
- **`FileComplexity`**: Data class representing complexity analysis results for a file.
- **`lizard.analyze_file.analyze_source_code`**: External library used to perform the actual code analysis.

This class fits into the architecture by providing a centralized and flexible mechanism for analyzing code complexity across different projects and languages.

---

*Generated by CodeWorm on 2026-02-19 12:42*
