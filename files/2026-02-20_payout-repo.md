# payout_repo

**Type:** File Overview
**Repository:** stripe-referral
**File:** src/stripe_referral/repositories/payout_repo.py
**Language:** python
**Lines:** 1-105
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025 | CertGames.com
Payout repository
"""

from datetime import (
    UTC,
    datetime,
)

from sqlalchemy import select
from sqlalchemy.orm import Session

from ..config.enums import PayoutStatus
from ..models.Payout import Payout

from .base import BaseRepository


class PayoutRepository(BaseRepository[Payout]):
    """
    Repository for Payout database operations
    """
    def __init__(self, db: Session) -> None:
        """
        Initialize with Payout model
        """
        super().__init__(db, Payout)

    def get_by_user(self, user_id: str) -> list[Payout]:
        """
        Get all payouts for a user
        """
        stmt = select(Payout).where(Payout.user_id == user_id)
        return list(self.db.execute(stmt).scalars().all())

    def get_by_tracking_id(self, tracking_id: int) -> Payout | None:
        """
        Get payout by tracking ID
        """
        stmt = select(Payout).where(Payout.tracking_id == tracking_id)
        return self.db.execute(stmt).scalar_one_or_none()

    def get_pending_payouts(self,
                            limit: int | None = None) -> list[Payout]:
        """
        Get all pending payouts with optional limit
        """
        stmt = select(Payout).where(
            Payout.status == PayoutStatus.PENDING.value
        )
        if limit:
            stmt = stmt.limit(limit)
        return list(self.db.execute(stmt).scalars().all())

    def get_failed_payouts(self,
                           limit: int | None = None) -> list[Payout]:
        """
        Get all failed payouts with optional limit
        """
        stmt = select(Payout).where(
            Payout.status == PayoutStatus.FAILED.value
        )
        if limit:
            stmt = stmt.limit(limit)
        return list(self.db.execute(stmt).scalars().all())

    def mark_as_paid(
        self,
        payout_id: int,
        transaction_id: str
    ) -> Payout | None:
        """
        Mark payout as paid with transaction ID
        """
        payout = self.get_by_id(payout_id)
        if not payout:
            return None

        payout.status = PayoutStatus.PAID.value
        payout.external_transaction_id = transaction_id
        payout.processed_at = datetime.now(UTC)
        self.db.commit()
        self.db.refresh(payout)
        return payout

    def mark_as_failed(
        self,
        payout_id: int,
        error_message: str
    ) -> Payout | None:
        """
        Mark payout as failed with error message
        """
        payout = self.get_by_id(payout_id)
        if not payout:
            return None

        payout.status = PayoutStatus.FAILED.value
        payout.error_message = error_message
        payout.failed_at = datetime.now(UTC)
        self.db.commit()
        self.db.refresh(payout)
        return payout

```

---

## File Overview

### PayoutRepository Documentation

**Purpose:**
This file implements a repository for managing `Payout` entities within the `stripe-referral` application, providing database operations to interact with payout records.

**Key Exports and Public Interface:**
- **get_by_user(user_id: str) -> list[Payout]:** Retrieves all payouts associated with a specific user.
- **get_by_tracking_id(tracking_id: int) -> Payout | None:** Fetches a single payout by its tracking ID.
- **get_pending_payouts(limit: int | None = None) -> list[Payout]:** Returns pending payouts, optionally limited in number.
- **get_failed_payouts(limit: int | None = None) -> list[Payout]:** Retrieves failed payouts, with an optional limit.
- **mark_as_paid(payout_id: int, transaction_id: str) -> Payout | None:** Marks a payout as paid and records the transaction ID.
- **mark_as_failed(payout_id: int, error_message: str) -> Payout | None:** Marks a payout as failed and logs the failure reason.

**How it Fits into the Project:**
This repository is part of the `stripe-referral` application's core functionality, handling all database interactions related to payouts. It integrates with the SQLAlchemy ORM for database operations and leverages the `BaseRepository` class for common CRUD operations.

**Notable Design Decisions:**
- **Type Hints:** Utilizes Python type hints extensively for clarity and maintainability.
- **Context Managers:** Implicitly uses context managers from SQLAlchemy sessions to manage database transactions.
- **Enum Usage:** Employs enums (`PayoutStatus`) for status values, ensuring consistency and reducing magic strings.
- **Error Handling:** Implements basic error handling in `mark_as_paid` and `mark_as_failed` methods by returning `None` if the payout is not found.
```

This documentation provides an overview of the file's purpose, key functions, integration within the project, and notable design choices.

---

*Generated by CodeWorm on 2026-02-20 09:46*
