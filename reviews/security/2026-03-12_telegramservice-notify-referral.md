# TelegramService.notify_referral_conversion

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/app/clients/telegram.py
**Language:** python
**Lines:** 184-243
**Complexity:** 18.0

---

## Source Code

```python
def notify_referral_conversion(self, data: dict) -> int | None:
        from datetime import datetime, UTC  # pylint: disable=import-outside-toplevel

        plan_label = "Lifetime" if data["plan_type"] == "lifetime" else "Monthly"

        referrer_age = ""
        if data.get("referrer_account_created"):
            days = (datetime.now(UTC) - data["referrer_account_created"]).days
            referrer_age = f" ({days} days ago)" if days < 365 else f" ({days // 365}y ago)"

        sub_status = "None"
        if data.get("referrer_subscription_active") and data.get("referrer_subscription_type"):
            sub_status = f"{data['referrer_subscription_type'].title()} (active)"

        payout_method = data.get("referrer_payment_method") or "Not set"

        flags = []
        if data.get("referrer_account_created") and (datetime.now(UTC) - data["referrer_account_created"]).days < 1:
            flags.append("Referrer account &lt;1 day old")
        if not data.get("referrer_subscription_active"):
            flags.append("Referrer has no subscription")
        if data.get("code_created_at") and (datetime.now(UTC) - data["code_created_at"]).total_seconds() < 3600:
            flags.append("Code used &lt;1hr after creation")
        if data.get("referrer_total_referrals", 0) <= 1 and data["plan_type"] == "lifetime":
            flags.append("First referral + lifetime purchase")

        flags_section = ""
        if flags:
            flags_text = "\n".join(f"  - {f}" for f in flags)
            flags_section = f"\n\n⚠️ <b>FLAGS:</b>\n{flags_text}"

        ref_created = ""
        if data.get("referrer_account_created"):
            ref_created = data["referrer_account_created"].strftime("%Y-%m-%d")
        else:
            ref_created = "Unknown"

        new_created = ""
        if data.get("referred_account_created"):
            new_created = data["referred_account_created"].strftime("%Y-%m-%d")
        else:
            new_created = "Unknown"

        text = (
            f"<b>💸 New Referral Conversion</b>\n\n"
            f"<b>Plan:</b> {plan_label} (${data['plan_amount']:.2f})\n"
            f"<b>Reward:</b> ${data['reward_amount']:.2f}\n\n"
            f"<b>Referrer:</b> @{_escape_html(data.get('referrer_username', ''))}"
            f" ({_escape_html(data.get('referrer_email', ''))})\n"
            f"  Subscription: {sub_status}\n"
            f"  Account created: {ref_created}{referrer_age}\n"
            f"  Total referrals: {data.get('referrer_total_referrals', 0)}\n"
            f"  Payout method: {payout_method}\n\n"
            f"<b>Referred:</b> @{_escape_html(data.get('referred_username', ''))}"
            f" ({_escape_html(data.get('referred_email', ''))})\n"
            f"  Account created: {new_created}"
            f"{flags_section}\n\n"
            f"<i>Review and manually pay when ready.</i>"
        )
        return self.send_message(text)
```

---

## Security Review

### Security Review

**Vulnerabilities Found:**

1. **XSS Vulnerability (Info):**
   - Line 56 and 70: The `data.get('referrer_username', '')` and `data.get('referred_username', '')` are directly used in HTML content without proper sanitization, which could lead to XSS attacks if the input contains malicious scripts.

2. **Potential Injection Vulnerability (Low):**
   - Line 56-70: The function `_escape_html()` is not defined and might be missing, leading to potential injection vulnerabilities if the data is used in a context where it can be executed as code.

3. **Hardcoded Secrets or Credentials (Info):**
   - No hardcoded secrets found, but ensure that any external dependencies are securely managed.

4. **Input Validation Gaps (Low):**
   - Lines 15-28: The function does not validate the input data for unexpected values or types, which could lead to unexpected behavior.

**Attack Vectors:**

- An attacker could inject malicious scripts into `referrer_username` or `referred_username`, potentially leading to XSS attacks.
- Unvalidated inputs could be used to manipulate the logic of the application.

**Recommended Fixes:**

1. **Fix XSS Vulnerability:**
   - Ensure `_escape_html()` is defined and properly sanitizes input data before using it in HTML content (Lines 56, 70).

2. **Input Validation:**
   - Validate `data` to ensure it contains expected keys and values (e.g., check types and ranges) (Lines 15-28).

3. **Error Handling:**
   - Add error handling to manage potential issues with missing or unexpected data.

**Overall Security Posture:**

The code has some minor security concerns, primarily related to XSS and input validation. Addressing these will significantly improve the overall security posture of the application. Ensure that all external inputs are properly sanitized and validated before use.

---

*Generated by CodeWorm on 2026-03-12 12:17*
