# PromptBuilder

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/llm/prompts.py
**Language:** python
**Lines:** 380-529
**Complexity:** 0.0

---

## Source Code

```python
class PromptBuilder:
    """
    Builds prompts for documentation generation
    Loads templates from config or uses defaults
    """
    def __init__(
        self,
        settings: PromptSettings | None = None,
        style: str = "technical",
    ) -> None:
        """
        Initialize with optional settings from config
        """
        self.style = style
        self._settings = settings

        if settings and settings.system_prompt:
            self._system_prompt = settings.system_prompt
        else:
            self._system_prompt = DEFAULT_SYSTEM_PROMPT

        if settings and settings.documentation_template:
            self._doc_template = settings.documentation_template
        else:
            self._doc_template = DEFAULT_DOCUMENTATION_TEMPLATE

        if settings and settings.commit_message_template:
            self._commit_template = settings.commit_message_template
        else:
            self._commit_template = DEFAULT_COMMIT_MESSAGE_TEMPLATE

        if settings and settings.language_hints:
            self._language_hints = settings.language_hints
        else:
            self._language_hints = DEFAULT_LANGUAGE_HINTS

    def _get_language_hint(self, language: Language) -> str:
        """
        Get language-specific hint from config or defaults
        """
        return self._language_hints.get(language.value, "")

    def build_documentation_prompt(
        self,
        context: PromptContext,
    ) -> tuple[str,
               str]:
        """
        Build system and user prompts for documentation
        Returns (system_prompt, user_prompt)
        """
        system = self._system_prompt

        lang_hint = self._get_language_hint(context.language)
        if lang_hint:
            system += f"\n\nLanguage-specific guidance: {lang_hint}"

        display_name = context.name
        if context.class_name:
            display_name = f"{context.class_name}.{context.name}"

        user = self._doc_template.format(
            language = context.language.value,
            source = context.source,
            name = display_name,
            file_path = context.file_path,
            repo = context.repo,
            complexity = context.complexity,
            line_count = context.line_count,
        )

        if context.decorators:
            user += f"\n\nDecorators present: {', '.join(context.decorators)}"
        if context.is_async:
            user += "\n\nThis is an async function."

        return system, user

    def build_commit_message_prompt(
        self,
        documentation: str,
        context: PromptContext,
    ) -> tuple[str,
               str]:
        """
        Build prompt for generating commit message
        """
        system = "You generate natural, human sounding git commit messages. Be concise and specific."

        display_name = context.name
        if context.class_name:
            display_name = f"{context.class_name}.{context.name}"

        user = self._commit_
```

---

## Class Documentation

### PromptBuilder Documentation

**Class Responsibility and Purpose:**
The `PromptBuilder` class is responsible for constructing prompts tailored for various documentation generation tasks, such as creating system and user prompts for documentation, commit messages, and other target-specific documentation targets. It leverages configuration settings or default values to customize these prompts based on the context provided.

**Public Interface (Key Methods):**
- **`__init__(self, settings: PromptSettings | None = None, style: str = "technical")`:** Initializes the `PromptBuilder` with optional settings and a specified style.
- **`_get_language_hint(self, language: Language) -> str`:** Retrieves language-specific hints from configuration or defaults.
- **`build_documentation_prompt(self, context: PromptContext) -> tuple[str, str]`:** Constructs system and user prompts for documentation generation.
- **`build_commit_message_prompt(self, documentation: str, context: PromptContext) -> tuple[str, str]`:** Generates a commit message prompt based on provided documentation and context.
- **`build_target_prompt(self, target: DocumentationTarget) -> tuple[str, str]`:** Builds prompts for any `DocumentationTarget` based on its doc_type.

**Design Patterns Used:**
The class employs the **Factory Method** pattern through the `from_candidate` class method to create a `PromptContext` from an analysis candidate. It also uses **Strategy** and **Template Method** patterns by providing default templates and hints that can be customized via settings.

**How it Fits in the Architecture:**
In the CodeWorm architecture, `PromptBuilder` serves as a central component for generating prompts used across various documentation tasks. It interacts with other classes such as `AnalysisCandidate`, `DocumentationTarget`, and `PromptSettings` to provide context-specific prompts. This ensures that the generated documentation is tailored to the specific requirements of each task, enhancing the quality and relevance of the output.

---

*Generated by CodeWorm on 2026-02-19 13:03*
