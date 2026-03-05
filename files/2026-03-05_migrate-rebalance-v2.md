# migrate_rebalance_v2

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/devtools/data/migrations/migrate_rebalance_v2.py
**Language:** python
**Lines:** 1-174
**Complexity:** 0.0

---

## Source Code

```python
#!/usr/bin/env python3
"""
©AngelaMos | 2026
migrate_rebalance_v2.py

Production migration: rebalance XP curve + shop item costs/levels.

XP formula changes (in xp_engine.py):
  Old: 2-30 = +500, 31-60 = +750, 61-100 = +1000, 100+ = +1500
  New: 2-10 = +75,  11-30 = +150, 31-60 = +175,   61-100 = +1000, 100+ = +1500

Run from backend/ directory:
  uv run python devtools/data/migrations/migrate_rebalance_v2.py
  uv run python devtools/data/migrations/migrate_rebalance_v2.py --dry-run
"""

import os
import sys

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', '..', '..'))
os.environ.setdefault('ENVIRONMENT', 'production')

try:
    from bson import ObjectId
    from config import get_config
    from api.core.database import init_db
    from api.domains.shop.models.Shop import Shop
    from api.domains.account.models.User import User
except ImportError as e:
    print(f"Import Error: {e}")
    sys.exit(1)


SHOP_UPDATES = {
    "67c8019eafc1b9f001544cd4": {"cost": 300,   "levelRequired": 5},
    "67c8019eafc1b9f001544ccf": {"cost": 500,   "levelRequired": 10},
    "67c8019eafc1b9f001544cc8": {"cost": 750,   "levelRequired": 15},
    "67c8019eafc1b9f001544cce": {"cost": 1000,  "levelRequired": 20},
    "67c8019eafc1b9f001544cc2": {"cost": 1500,  "levelRequired": 25},
    "67c8019eafc1b9f001544cc3": {"cost": 2000,  "levelRequired": 30},
    "67c8019eafc1b9f001544cd3": {"cost": 2500,  "levelRequired": 35},
    "67c8019eafc1b9f001544cd7": {"cost": 3500,  "levelRequired": 40},
    "67c8019eafc1b9f001544cd5": {"cost": 4500,  "levelRequired": 45},
    "67c8019eafc1b9f001544cca": {"cost": 5500,  "levelRequired": 50},
    "67c8019eafc1b9f001544cc4": {"cost": 6500,  "levelRequired": 55},
    "67c8019eafc1b9f001544cd1": {"cost": 7500,  "levelRequired": 60},
    "67c8019eafc1b9f001544cc6": {"cost": 8500,  "levelRequired": 65},
    "67c8019eafc1b9f001544ccc": {"cost": 10000, "levelRequired": 70},
    "67c8019eafc1b9f001544cc9": {"cost": 12000, "levelRequired": 75},
    "67c8019eafc1b9f001544cc5": {"cost": 14000, "levelRequired": 80},
    "67c8019eafc1b9f001544cd2": {"cost": 16000, "levelRequired": 85},
    "67c8019eafc1b9f001544cd0": {"cost": 18000, "levelRequired": 90},
    "67c8019eafc1b9f001544cd6": {"cost": 20000, "levelRequired": 95},
    "67c8019eafc1b9f001544cc7": {"cost": 22000, "levelRequired": 98},
    "67c8019eafc1b9f001544ccd": {"cost": 25000, "levelRequired": 100},
    "67c8033dafc1b9f001544cdb": {"cost": 1500,  "levelRequired": 5},
    "67c8033dafc1b9f001544cd8": {"cost": 3000,  "levelRequired": 15},
    "67c8033dafc1b9f001544cdc": {"cost": 5000,  "levelRequired": 25},
    "67c8033dafc1b9f001544cd9": {"cost": 8000,  "levelRequired": 40},
    "67c8033dafc1b9f001544cda": {"cost": 12000, "levelRequired": 60},
}


def xp_required_for_level(level: int) -> int:
    if level < 1 or level == 1:
        return 0
    if level <= 10:
        return 75 * (level - 1)
    if level <= 30:
        base = 75 * 9
        return base + 150 * (level - 
```

---

## File Overview

# migrate_rebalance_v2.py

**Purpose and Responsibility:**
This script is responsible for rebalancing the XP curve and shop item costs/levels in a production environment. It updates the XP requirements for different levels and recalculates user levels based on their accumulated XP.

**Key Exports or Public Interface:**
- `xp_required_for_level(level: int) -> int`: Calculates the XP required to reach a given level.
- `calculate_level_from_xp(xp: float) -> int`: Determines the level corresponding to a given amount of XP.
- `migrate_shop_items(dry_run: bool) -> None`: Migrates shop item costs and levels, optionally in dry-run mode.
- `migrate_user_levels(dry_run: bool) -> None`: Recalculates user levels based on their accumulated XP, also supporting dry-run.

**How it Fits into the Project:**
This script is part of a larger migration process for rebalancing game mechanics. It runs from the backend directory and interacts with MongoDB through the `Shop` and `User` models defined in `api.domains`. The changes are significant enough to warrant careful testing, hence the `--dry-run` option.

**Notable Design Decisions:**
- **Dry Run Support:** Allows for previewing changes without committing them, ensuring safety during migrations.
- **Explicit Imports:** Ensures all dependencies are loaded correctly and avoids runtime errors.
- **Custom XP Calculation Logic:** Implements a complex XP formula with multiple tiers to accurately reflect the new balance.
- **Database Interaction:** Uses PyMongo (`ObjectId`) and custom models for database operations, maintaining consistency with the project's architecture.
```

This documentation provides an overview of the file’s purpose, key functions, integration within the project, and significant design choices.

---

*Generated by CodeWorm on 2026-03-05 02:56*
