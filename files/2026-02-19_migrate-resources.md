# migrate_resources

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/devtools/migrations/migrate_resources.py
**Language:** python
**Lines:** 1-2402
**Complexity:** 0.0

---

## Source Code

```python
#!/usr/bin/env python3
"""
Resource Migration Script - Populate MongoDB with all resources
/backend/scripts/migrate_resources.py

Usage: docker exec -it backend_service python devtools/scripts/migrate_resources.py
"""

import os
import sys
from datetime import datetime, UTC


sys.path.insert(0, os.path.join(os.path.dirname(__file__), '../..'))

os.environ.setdefault('PRODUCTION', 'production')

from config import get_config
from api.core.database import init_db
from api.domains.content.models.Resource import (
    Resource,
    ResourceCategory,
    ResourceType,
    Certification,
    DifficultyLevel,
    SourceIcon
)


def get_certification_tags(name, url, description = ""):
    """
    Determine certification tags based on resource name, URL, and description
    """
    name_lower = name.lower()
    url_lower = url.lower()
    desc_lower = description.lower()
    combined = f"{name_lower} {url_lower} {desc_lower}"

    tags = []

    # A+ patterns
    if any(pattern in combined for pattern in
           ['a+', 'a plus', '220-1101', '220-1102', 'core 1', 'core 2']):
        if not any(exclude in combined for exclude in
                   ['network+', 'security+', 'casp', 'pentest+']):
            tags.append(Certification.A_PLUS.value)

    # Network+ patterns
    if any(pattern in combined for pattern in
           ['network+', 'n10-008', 'n10-009', 'networking', 'subnet']):
        if not any(exclude in combined
                   for exclude in ['a+', 'security+', 'casp', 'pentest+']):
            tags.append(Certification.NETWORK_PLUS.value)

    # Security+ patterns
    if any(pattern in combined
           for pattern in ['security+', 'sy0-701', 'sec+']):
        tags.append(Certification.SECURITY_PLUS.value)

    # CySA+ patterns
    if any(pattern in combined
           for pattern in ['cysa+', 'cs0-003', 'cybersecurity analyst']):
        tags.append(Certification.CYSA_PLUS.value)

    # CASP+/SecurityX patterns
    if any(pattern in combined
           for pattern in ['casp', 'cas-004', 'cas-005', 'securityx']):
        tags.append(Certification.CASP_PLUS.value)

    # PenTest+ patterns
    if any(pattern in combined
           for pattern in ['pentest+', 'pt0-002', 'pt0-003', 'penetration',
                           'ethical hack']):
        tags.append(Certification.PENTEST_PLUS.value)

    # Cloud+ patterns
    if any(pattern in combined for pattern in
           ['cloud+', 'cv0-003', 'cloud essentials', 'cl0-002']):
        tags.append(Certification.CLOUD_PLUS.value)

    # Linux+ patterns
    if any(pattern in combined
           for pattern in ['linux+', 'xk0-005', 'kali linux', 'ubuntu']):
        tags.append(Certification.LINUX_PLUS.value)

    # Data+ patterns
    if any(pattern in combined for pattern in ['data+', 'da0-001']):
        tags.append(Certification.DATA_PLUS.value)

    # Server+ patterns
    if any(pattern in combined for pattern in ['server+', 'sk0-005']):
        tags.append(Certification.SERVER_PLUS.value)


```

---

## File Overview

# Resource Migration Script - Populate MongoDB with all resources

## Purpose and Responsibility
This script is responsible for populating a MongoDB database with resource data, including certifications, categories, types, sources, and difficulty levels. It runs as part of the backend service setup process.

## Key Exports and Public Interface
- `get_certification_tags`: Determines certification tags based on resource name, URL, and description.
- `get_source_icon`: Identifies source icons from URLs and names.
- `get_difficulty_from_n`: (Incomplete function) Likely intended to determine difficulty levels based on some input.

## Fit in the Project
This script is a critical part of the initial setup for the backend service. It ensures that the database contains all necessary resources, which are essential for the application's functionality. The script is invoked via Docker and uses configuration settings from `config.py` to initialize the database connection.

## Design Decisions
- **Modular Functions**: Functions like `get_certification_tags` and `get_source_icon` are designed to be reusable across different parts of the project.
- **Case Insensitivity**: Uses case-insensitive string matching for pattern recognition, ensuring robust handling of input variations.
- **Default Values**: Provides default values in case no specific tags or icons can be determined, ensuring that resources are still added even if patterns do not match.
- **Environment Configuration**: Utilizes environment variables and configuration files to manage settings dynamically, enhancing flexibility and maintainability.
```

This documentation provides a high-level overview of the script's role within the project, its key functionalities, and important design choices.

---

*Generated by CodeWorm on 2026-02-19 09:33*
