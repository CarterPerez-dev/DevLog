# migrate_rebalance_v2

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/devtools/data/migrations/migrate_rebalance_v2.py
**Language:** python
**Lines:** 1-173
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
  uv run python devtools/data/migrations/migrate_rebalance_v2.py --env development --dry-run
  uv run python devtools/data/migrations/migrate_rebalance_v2.py --env development
  uv run python devtools/data/migrations/migrate_rebalance_v2.py --env production --dry-run
  uv run python devtools/data/migrations/migrate_rebalance_v2.py --env production
"""

import os
import sys
import argparse

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', '..', '..'))

parser = argparse.ArgumentParser(description="Rebalance XP curve and shop prices")
parser.add_argument("--env", choices=["development", "production"], default="production", help="Target environment")
parser.add_argument("--dry-run", action="store_true", help="Preview changes without writing to DB")
args = parser.parse_args()

os.environ["ENVIRONMENT"] = args.env

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
    "67c8033dafc1b9f001544cdb": {"cost
```

---

## File Overview

# migrate_rebalance_v2.py

**Purpose & Responsibility:**
This script is responsible for rebalancing the XP curve and shop item costs/levels in a game backend system. It updates the XP requirements for different levels and adjusts the cost and level requirements of various shop items based on new formulas.

**Key Exports & Public Interface:**
- `xp_required_for_level(level: int) -> int`: Calculates the required XP to reach a given player level.
- `calculate_level_from_xp(xp: float) -> int`: Determines the player level from their accumulated XP.
- `migrate_shop_items(dry_run: bool) -> None`: Migrates shop items' costs and level requirements based on predefined updates.
- `migrate_user_levels(dry_run: bool) -> None`: Recalculates user levels based on updated XP formulas.

**How It Fits in the Project:**
This script is part of a larger migration process for updating game mechanics. It runs as a command-line tool from within the backend directory, using environment-specific configurations and database interactions via `init_db`. The changes can be previewed or applied directly to the production database depending on the `--dry-run` flag.

**Notable Design Decisions:**
- **Modular Functions:** Functions are designed to handle specific tasks (XP calculation, shop item migration, user level recalculations) for better maintainability.
- **Dry Run Support:** The script supports a dry run mode, allowing developers to preview changes without committing them to the database.
- **Environment Configuration:** Uses `argparse` to manage command-line arguments and sets environment variables accordingly.
- **Database Interaction:** Utilizes MongoDB models (`Shop`, `User`) for interacting with the database, ensuring data integrity during migrations.
```

This documentation provides a high-level overview of the file's purpose, key functions, integration within the project, and design choices.

---

*Generated by CodeWorm on 2026-03-05 03:06*
