# repository_pattern

**Type:** Pattern Analysis
**Repository:** CertGames-Core
**File:** backend/api/domains/referral/services/payout_ops.py
**Language:** python
**Lines:** 1-311
**Complexity:** 0.0

---

## Source Code

```python
"""
Payout Operations Service
/api/domains/referral/services/payout_ops.py

Wrapper around stripe-referral package payout service
"""

import os
import stripe
from typing import Any
from flask import current_app

from stripe_referral.services import PayoutService
from stripe_referral.config.enums import PayoutStatus
from stripe_referral.repositories.payout_repo import PayoutRepository
from stripe_referral.repositories.referral_repo import ReferralTrackingRepository

from api.core.database.referral_db import (
    get_referral_db,
    referral_db_session,
)

from api.domains.account.models.User import User


class PayoutOperations:
    """
    Service for payout operations
    Supports both manual payouts and Stripe Connect (when enabled)
    """
    @staticmethod
    def get_pending_payouts(user_id: str | None = None) -> list[dict[str,
                                                                     Any]]:
        """
        Get pending payouts (admin or specific user)

        Args:
            user_id: Filter by user, or None for all pending

        Returns:
            List of payout dicts
        """
        db = get_referral_db()
        try:
            payout_repo = PayoutRepository(db)

            if user_id:
                payouts = payout_repo.get_by_user(str(user_id))
                payouts = [
                    p for p in payouts
                    if p.status == PayoutStatus.PENDING.value
                ]
            else:
                payouts = list(payout_repo.get_all())
                payouts = [
                    p for p in payouts
                    if p.status == PayoutStatus.PENDING.value
                ]

            return [
                {
                    "id":
                    p.id,
                    "user_id":
                    p.user_id,
                    "amount":
                    p.amount,
                    "currency":
                    p.currency,
                    "status":
                    p.status,
                    "adapter_type":
                    p.adapter_type,
                    "tracking_id":
                    p.tracking_id,
                    "created_at":
                    p.created_at.isoformat() if p.created_at else None
                } for p in payouts
            ]

        finally:
            db.close()

    @staticmethod
    def create_payouts_for_pending_trackings() -> dict[str, Any]:
        """
        Create payout records for all pending tracking entries
        Works with manual adapter (no Stripe Connect required)

        Returns:
            {
                "created": 5,
                "skipped": 2,
                "errors": []
            }
        """
        with referral_db_session() as db:

            tracking_repo = ReferralTrackingRepository(db)
            payout_repo = PayoutRepository(db)

            pending_trackings = tracking_repo.get_pending_payouts()

            results = {"created": 0, "skipped": 0, "errors": [
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

The `PayoutOperations` class in the provided code implements the **Repository Pattern**, which abstracts data access logic and encapsulates database operations within a dedicated repository.

#### Implementation Details:
- The `PayoutOperations` class uses two repositories: `PayoutRepository` and `ReferralTrackingRepository`. These repositories handle CRUD operations for payout records and referral tracking entries, respectively.
- Methods like `get_by_user`, `get_all`, `create`, etc., are defined in the repository classes to interact with the database.

#### Benefits:
1. **Encapsulation**: The repository pattern encapsulates data access logic, making it easier to switch between different storage mechanisms (e.g., from a relational database to an in-memory store).
2. **Testability**: Repository methods can be easily mocked or stubbed for unit testing.
3. **Decoupling**: It decouples the business logic from the persistence layer, allowing changes in the data access mechanism without affecting the service code.

#### Deviations:
- The `PayoutOperations` class directly interacts with the database session (`referral_db_session`) and closes it using a context manager (`finally` block). This is an unconventional use of the repository pattern, as repositories typically manage their own sessions.
- The `create_payouts_for_pending_trackings` method uses `User.objects.first()` to fetch user data, which might not be part of the standard repository pattern.

#### Appropriateness:
The repository pattern is appropriate here because it abstracts database operations and makes the code more maintainable. However, managing the database session directly in the service class deviates from best practices. A more typical approach would involve the repository managing its own session or using a dependency injection framework to manage sessions.

This pattern is generally suitable for applications where data access logic needs to be separated from business logic and can benefit significantly from abstraction and testability.

---

*Generated by CodeWorm on 2026-02-22 23:33*
