# SecurityService

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/services/security/security_service.py
**Language:** python
**Lines:** 21-786
**Complexity:** 0.0

---

## Source Code

```python
class SecurityService:
    """
    Unified security service with context-aware pattern detection
    """

    SECURITY_LEVELS = {
        "low": {
            "severity_threshold": 0.9,
            "check_patterns": ["critical"],
            "input_strictness": 0.2,
            "url_strictness": 0.1,
        },
        "medium": {
            "severity_threshold": 0.6,
            "check_patterns": ["common"],
            "input_strictness": 0.5,
            "url_strictness": 0.3,
        },
        "high": {
            "severity_threshold": 0.3,
            "check_patterns": ["extensive"],
            "input_strictness": 0.8,
            "url_strictness": 0.6,
        },
        "strict": {
            "severity_threshold": 0.1,
            "check_patterns": ["all"],
            "input_strictness": 0.95,
            "url_strictness": 0.8,
        },
    }

    CONTEXT_MODIFIERS = {
        "ai": {
            "strictness_multiplier": 0.1,
            "allow_security_terms": True,
            "description": "AI tool endpoints generating security content",
        },
        "educational": {
            "strictness_multiplier": 0.05,
            "allow_security_terms": True,
            "description": "Quiz questions, educational content",
        },
        "general": {
            "strictness_multiplier": 0.8,
            "allow_security_terms": False,
            "description": "Standard API endpoints",
        },
        "auth": {
            "strictness_multiplier": 1.5,
            "allow_security_terms": False,
            "description": "Login, register, password reset",
        },
        "admin": {
            "strictness_multiplier": 2.0,
            "allow_security_terms": False,
            "description": "Administrative endpoints",
        },
    }

    ATTACK_PATTERNS = {
        "critical": {
            "sql_injection": [
                r"\bDROP\s+TABLE\b",
                r"\bDELETE\s+FROM\s+\w+\s+WHERE\s+1\s*=\s*1",
                r";\s*DROP\s+DATABASE",
                r"\bUNION\s+SELECT\s+.*\s+FROM\s+information_schema",
                r"xp_cmdshell",
                r"sp_executesql",
            ],
            "command_injection": [
                r";\s*rm\s+-rf\s+/",
                r"\|\s*nc\s+.*\s+-e\s+/bin/sh",
                r"&&\s*curl\s+.*\s*\|\s*sh",
            ],
            "file_inclusion": [
                r"\.\.\/\.\.\/\.\.\/etc\/passwd",
                r"file:\/\/\/etc\/passwd",
                r"\.\.\\\.\.\\\.\.\\windows\\system32",
            ],
        },
        "common": {
            "sql_injection": [
                r"'\s*OR\s*'1'\s*=\s*'1\s*(--|;)",
                r'"\s*OR\s*"1"\s*=\s*"1\s*(--|;)',
                r"OR\s+1\s*=\s*1\s*--",
                r"UNION\s+SELECT.*FROM",
                r"\/\*.*\*\/.*('|\")",
                "@@version--",
                "INFORMATION_SCHEMA.TABLES",
            ],
            "xss": [
                r"<script[^>]*>.*alert\(",
                r"javasc
```

---

## Class Documentation

### SecurityService Documentation

**Class Responsibility and Purpose:**
The `SecurityService` class is responsible for analyzing HTTP requests to detect potential security threats based on predefined security levels and context modifiers. It uses a unified approach to apply different strictness levels and pattern checks, ensuring that security assessments are context-aware.

**Public Interface (Key Methods):**
- **analyze_request(security_level: SecurityLevel = "medium", context: SecurityContext = "general", skip_endpoints: list[str] | None = None) -> dict[str, Any]:** This method analyzes the current request for security threats based on the specified `security_level` and `context`. It returns a dictionary containing analysis results without blocking.

**Design Patterns Used:**
- **Strategy Pattern:** The class uses dictionaries (`SECURITY_LEVELS`, `CONTEXT_MODIFIERS`, `ATTACK_PATTERNS`) to define different strategies for handling requests at various security levels and contexts.
  
**Relationship to Other Classes:**
`SecurityService` interacts with other classes that manage request handling, context definitions, and security level configurations. It is part of the backend API's core services layer, providing a centralized mechanism for security threat detection.

**State Management Approach:**
The class maintains state through its static dictionaries (`SECURITY_LEVELS`, `CONTEXT_MODIFIERS`, `ATTACK_PATTERNS`), which are used to configure and apply different security checks. These configurations do not change during the execution of the method but provide a flexible way to adapt to various security scenarios.

This class fits into the architecture by providing a robust, context-aware mechanism for detecting potential security threats in HTTP requests, ensuring that the API remains secure while handling diverse use cases.

---

*Generated by CodeWorm on 2026-02-18 13:10*
