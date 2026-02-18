# build_pyproject

**Type:** Documentation
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

## Documentation

### Documentation for `build_pyproject`

**Purpose and Behavior:**
The function `build_pyproject` constructs a complete `pyproject.toml` structure from a given `ProjectConfig`. It populates the project details, dependencies, URLs, authors, and various tool configurations based on the provided configuration. The resulting dictionary can be directly written to a `pyproject.toml` file.

**Key Implementation Details:**
- **Type Hints:** The function uses type hints for better readability and maintainability.
- **Preset Handling:** It retrieves preset data from `PRESETS[config.preset]`, which likely contains default configurations.
- **Conditional Logic:** The function conditionally adds optional dependencies, URLs, authors, and tool-specific configurations based on the provided configuration.
- **Dynamic Imports:** Uses dynamic imports for tools like Ruff, MyPy, PyLint, etc., to configure them according to the project path.

**When/Why to Use:**
This code is useful when automating the creation of `pyproject.toml` files. It ensures that all necessary fields are correctly populated based on a structured configuration object. This can be particularly handy in CI/CD pipelines or during initial setup of new projects.

**Patterns and Gotchas:**
- **Copy Operations:** Using `.copy()` to avoid mutating original lists, which is good practice.
- **Dynamic Configuration:** The function dynamically configures various tools like Ruff, MyPy, PyLint, etc., based on the project path. Ensure that these configurations are accurate for your specific use case.
- **Preset Dependency Management:** Presets handle default dependencies and dev dependencies, making it easier to manage common configurations but ensure they align with your project needs.

---

*Generated by CodeWorm on 2026-02-18 07:55*
