# build_pyproject

**Type:** Performance Analysis
**Repository:** pyproject-setup
**File:** src/pyproject_setup/generator.py
**Language:** python
**Lines:** 131-211
**Complexity:** 8.0

---

## Source Code

```python
def build_pyproject(config: ProjectConfig) -> dict[str, Any]:
    """
    Build complete pyproject.toml structure from config
    """
    preset: Preset = PRESETS[config.preset]

    project: dict[str,
                  Any] = {
                      "name": config.name,
                      "version": config.version,
                      "description": config.description,
                      "requires-python": config.python_version,
                      "dependencies": preset.dependencies.copy(),
                  }

    if preset.dev_dependencies:
        project["optional-dependencies"] = {
            "dev": preset.dev_dependencies.copy()
        }

    urls: dict[str, str] = {}
    if config.homepage:
        urls["Homepage"] = config.homepage
    if config.repository:
        urls["Repository"] = config.repository
        urls["Issues"] = f"{config.repository}/issues"
        urls["Changelog"] = f"{config.repository}/blob/main/CHANGELOG.md"
    if urls:
        project["urls"] = urls

    if config.author_name and config.author_email:
        project["authors"] = [
            {
                "name": config.author_name,
                "email": config.author_email
            }
        ]

    pyproject: dict[str,
                    Any] = {
                        "project": project,
                        "build-system": {
                            "requires": ["hatchling"],
                            "build-backend": "hatchling.build",
                        },
                        "tool": {
                            "hatch": {
                                "build": {
                                    "targets": {
                                        "wheel": {
                                            "packages":
                                            [config.package_path]
                                        }
                                    }
                                }
                            },
                            "ruff":
                            get_ruff_config(config.package_path),
                            "mypy":
                            get_mypy_config(config.package_path),
                            "pydantic-mypy":
                            get_pydantic_mypy_config(),
                            "pylint":
                            get_pylint_config(config.package_path),
                            "pytest": {
                                "ini_options": get_pytest_config()
                            },
                            "coverage":
                            get_coverage_config(config.package_path),
                            "ty":
                            get_ty_config(config.package_path),
                        },
                    }

    if preset.entry_point:
        pyproject["project"]["scripts"] = {
            config.name:
            f"{config.package_path.replace('/', '.')}.{preset.entry_point}"
        }

    return pyproject
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of `build_pyproject` is **O(1)**, as the operations performed are constant-time manipulations on dictionaries and lists, regardless of input size.

#### Space Complexity
The space complexity is also **O(1)**, assuming that the number of preset entries and configuration fields remain constant. The primary memory usage comes from creating temporary dictionaries and lists during execution.

#### Bottlenecks or Inefficiencies
- **Redundant Operations**: The `copy()` method on `preset.dependencies` and `preset.dev_dependencies` is unnecessary since these are not modified elsewhere in the function.
- **Unnecessary Checks**: The checks for `config.homepage`, `config.repository`, and `config.author_name` can be simplified by using conditional expressions directly.

#### Optimization Opportunities
1. **Remove Unnecessary Copies**:
   ```python
   project["dependencies"] = preset.dependencies  # Remove .copy()
   ```
2. **Simplify Conditional Checks**:
   ```python
   urls = {"Homepage": config.homepage} if config.homepage else {}
   urls.update({"Repository": config.repository, "Issues": f"{config.repository}/issues", "Changelog": f"{config.repository}/blob/main/CHANGELOG.md"} if config.repository else {})
   project["urls"] = urls if urls else None
   ```
3. **Combine Author Information**:
   ```python
   authors = [{"name": config.author_name, "email": config.author_email}] if config.author_name and config.author_email else []
   project["authors"] = authors
   ```

#### Resource Usage Concerns
- Ensure that `PRESETS`, `get_ruff_config`, `get_mypy_config`, etc., are efficiently implemented to avoid any potential bottlenecks.
- Consider using context managers or `with` statements for any file or resource handling, though this snippet does not include such operations.

By applying these optimizations, the function can be made more efficient and maintainable.

---

*Generated by CodeWorm on 2026-02-22 00:16*
