# security_service

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/api/core/services/security/security_service.py
**Language:** python
**Lines:** 1-787
**Complexity:** 0.0

---

## Source Code

```python
"""
Security Service - Context-aware pattern detection and analysis
/api/core/services/security/security_service.py
"""

import re
import json
import hashlib
import logging
from flask import request, g
from typing import Any, Literal
from datetime import datetime, UTC


logger = logging.getLogger(__name__)

SecurityLevel = Literal["low", "medium", "high", "strict"]
SecurityContext = Literal["ai", "educational", "general", "auth", "admin"]


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
       
```

---

## File Overview

### SecurityService Class Documentation

**File Purpose and Responsibility:**
This file defines the `SecurityService` class, which implements a unified security service with context-aware pattern detection for various API endpoints. It handles security checks based on predefined patterns and context-specific configurations.

**Key Exports or Public Interface:**
- **SecurityService**: The main class responsible for security analysis.
  - `SECURITY_LEVELS`: A dictionary mapping security levels to their respective thresholds, check patterns, and strictness values.
  - `CONTEXT_MODIFIERS`: A dictionary defining how different contexts modify the overall security behavior.
  - `ATTACK_PATTERNS`: A comprehensive list of attack patterns categorized by severity.

**How It Fits in the Project:**
This service is a critical component within the CertGames-Core project, integrated into the Flask application to ensure that API endpoints are secure. It interacts with other services and modules to provide real-time security assessments based on the context of each request.

**Notable Design Decisions:**
- **Context-Aware Security**: The `CONTEXT_MODIFIERS` dictionary allows for dynamic adjustment of security settings based on the type of endpoint, ensuring that educational or administrative endpoints have different security profiles.
- **Attack Pattern Matching**: The `ATTACK_PATTERNS` dictionary contains regex patterns to detect common and critical attack vectors, providing a robust defense mechanism.
- **Type Hints and Strictness**: Utilizing Python's `typing` module for type hints and defining strictness levels helps in maintaining clear and maintainable code.

This class is essential for ensuring that the CertGames-Core backend remains secure against various types of attacks while adapting to different operational contexts.

---

*Generated by CodeWorm on 2026-02-19 01:59*
