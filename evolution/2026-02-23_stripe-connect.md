# stripe_connect

**Type:** Code Evolution
**Repository:** stripe-referral
**File:** src/stripe_referral/adapters/stripe_connect.py
**Language:** python
**Lines:** 1-1
**Complexity:** 0.0

---

## Source Code

```python
Commit: bfb52633
Message: feat: Done
Author: CarterPerez-dev
File: src/stripe_referral/adapters/stripe_connect.py
Change type: new file

Diff:
@@ -0,0 +1,161 @@
+"""
+ⒸAngelaMos | 2025 | CertGames.com
+Stripe Connect payout adapter
+"""
+
+import stripe
+from typing import Any
+
+from ..config.constants import (
+    STRIPE_API_VERSION,
+    STRIPE_CONNECT_ACCOUNT_ID_PREFIX,
+    STRIPE_TRANSFER_ID_PREFIX,
+)
+from ..schemas.types import (
+    PayoutResult,
+    RecipientValidation,
+)
+from .base import PayoutAdapter
+
+
+class StripeConnectAdapter(PayoutAdapter):
+    """
+    Payout adapter for Stripe Connect transfers
+    """
+    def __init__(self, api_key: str) -> None:
+        """
+        Initialize with Stripe API key
+
+        Args:
+            api_key: Stripe secret key
+        """
+        self.api_key = api_key
+        stripe.api_key = api_key
+        stripe.api_version = STRIPE_API_VERSION
+
+    def send_payout(
+        self,
+        user_id: str,
+        amount: float,
+        currency: str,
+        recipient_data: dict[str,
+                             Any],
+    ) -> PayoutResult:
+        """
+        Send payout via Stripe Connect transfer
+        """
+        try:
+            connected_account_id = recipient_data.get(
+                "stripe_connect_account_id"
+            )
+            if not connected_account_id:
+                return PayoutResult(
+                    success = False,
+                    error =
+                    "Missing stripe_connect_account_id in recipient_data",
+                )
+
+            if not connected_account_id.startswith(
+                    STRIPE_CONNECT_ACCOUNT_ID_PREFIX):
+                return PayoutResult(
+                    success = False,
+                    error =
+                    f"Invalid Stripe account ID format: {connected_account_id}",
+                )
+
+            amount_cents = int(amount * 100)
+
+            transfer = stripe.Transfer.create(
+                amount = amount_cents,
+                currency = currency.lower(),
+                destination = connected_account_id,
+                metadata = {"user_id": user_id},
+            )
+
+            if not transfer.id.startswith(STRIPE_TRANSFER_ID_PREFIX):
+                return PayoutResult(
+                    success = False,
+                    error =
+                    f"Unexpected transfer ID format: {transfer.id}",
+                )
+
+            return PayoutResult(
+                success = True,
+                transaction_id = transfer.id,
+            )
+
+        except stripe.InvalidRequestError as e:
+            return PayoutResult(
+                success = False,
+                error = f"Invalid request: {str(e)}",
+            )
+        except stripe.AuthenticationError as e:
+            return PayoutResult(
+                success = False,
+                error = f"Authentication failed: {str(e)}",
+            )
+        except stripe.S
```

---

## Code Evolution

### Change Analysis for Commit `bfb52633`

**What was Changed:**
A new file, `stripe_connect.py`, was added to the repository. This file introduces a class `StripeConnectAdapter` that extends `PayoutAdapter`. The adapter handles Stripe Connect transfers and validations.

- **Initialization**: The `__init__` method initializes with an API key.
- **Send Payout**: The `send_payout` method sends payouts via Stripe Connect, validating the recipient's account ID format and ensuring the transfer ID is correctly formatted.
- **Recipient Validation**: The `validate_recipient` method checks if a Stripe Connect account exists and can receive payouts.

**Why it was Likely Changed:**
This change likely introduces support for Stripe Connect in the payout system. It ensures that payouts are sent to valid, active Stripe accounts and handles various error scenarios gracefully.

**Impact on Behavior:**
- The adapter will now handle Stripe Connect transfers more robustly.
- Payouts can only be sent to accounts with enabled payouts and charges.
- Errors during transfer or validation are caught and returned as structured `PayoutResult` and `RecipientValidation` objects, improving debugging and error handling.

**Risks or Concerns:**
- **Error Handling**: The extensive try-except blocks could potentially hide issues if not implemented carefully. Ensure that all possible exceptions are handled appropriately.
- **Performance**: Repeated API calls to Stripe might impact performance; consider caching account validation results if the same accounts are validated frequently.
- **Security**: Ensure that sensitive information like API keys is securely managed and not exposed in logs or code.

This change significantly enhances the payout system's reliability and compliance with Stripe’s requirements.

---

*Generated by CodeWorm on 2026-02-23 01:40*
