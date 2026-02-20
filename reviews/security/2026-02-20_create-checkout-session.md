# create_checkout_session

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/api/domains/account/controllers/subscription/stripe_ctrl.py
**Language:** python
**Lines:** 53-151
**Complexity:** 17.0

---

## Source Code

```python
def create_checkout_session():
    """
    Create a Stripe Checkout session for a new subscription
    """
    user_id = g.validated.get('userId')
    registration_data = g.validated.get('registrationData')
    is_oauth_flow = g.validated.get('isOauthFlow', False)
    referral_code = g.validated.get('referralCode')
    price_id = g.validated.get('priceId') or current_app.config['STRIPE_PRICE_ID']

    validated_referral_code = None
    if referral_code:
        try:
            validation = ReferralOperations.validate_code(referral_code)
            if validation and validation.get('valid'):
                validated_referral_code = referral_code
                current_app.logger.info(
                    f"Valid referral code applied to checkout: {referral_code}"
                )
            else:
                current_app.logger.warning(
                    f"Invalid referral code attempted: {referral_code}"
                )
        except Exception as e:
            current_app.logger.error(
                f"Error validating referral code during checkout: {str(e)}"
            )

    try:
        frontend_url = current_app.config.get(
            'FRONTEND_URL',
            'http://localhost:3000'
        )
        if not frontend_url.endswith('/'):
            frontend_url += '/'

        success_url = f"{frontend_url}subscription/success?session_id={{CHECKOUT_SESSION_ID}}"
        if user_id:
            success_url += f"&user_id={user_id}"

        cancel_url = f"{frontend_url}subscription/cancelled"

        metadata = {
            "user_id": user_id or "new_registration",
            "is_new_user": "true" if registration_data else "false",
            "is_oauth_flow": "true" if is_oauth_flow else "false",
        }

        if validated_referral_code:
            metadata["referral_code"] = validated_referral_code

        checkout_session = stripe.checkout.Session.create(
            payment_method_types = ["card"],
            line_items = [
                {
                    "price": price_id,
                    "quantity": 1,
                },
            ],
            mode = "subscription",
            success_url = success_url,
            cancel_url = cancel_url,
            client_reference_id = user_id or "new_registration",
            metadata = metadata,
        )

        if registration_data:
            session["temp_registration_data"] = registration_data
            TempRegistrationService.create_for_checkout(
                checkout_session.id,
                registration_data,
                is_oauth_flow
            )

        if is_oauth_flow:
            session["is_oauth_flow"] = True

        session["checkout_session_id"] = checkout_session.id

        return {
            "sessionId": checkout_session.id,
            "url": checkout_session.url
        }

    except stripe.error.StripeError as e:
        current_app.logger.error(
            f"Stripe API error creating checkout session: {str(e)}"

```

---

## Security Review

### Security Review for `create_checkout_session`

#### Vulnerabilities Found:

1. **Hardcoded Secrets or Credentials:**
   - **Severity:** Low
   - **Line:** `price_id = g.validated.get('priceId') or current_app.config['STRIPE_PRICE_ID']`
   - **Fix:** Use environment variables for sensitive values like Stripe API keys instead of hardcoding them.

2. **Error Handling that Leaks Information:**
   - **Severity:** Medium
   - **Line:** `current_app.logger.error(f"Stripe API error creating checkout session: {str(e)}")` and similar lines.
   - **Fix:** Log generic errors without exposing sensitive information like Stripe API keys or user data.

3. **Input Validation Gaps:**
   - **Severity:** Medium
   - **Line:** `registration_data = g.validated.get('registrationData')`
   - **Fix:** Ensure thorough validation of all inputs, especially `registration_data`, to prevent injection attacks.

4. **Authentication and Authorization Issues:**
   - **Severity:** Low
   - **Line:** No explicit checks for user authentication or authorization.
   - **Fix:** Implement proper authentication and authorization mechanisms before processing sensitive operations.

5. **Overall Security Posture:**
   - The code is generally secure but could benefit from improvements in error handling, input validation, and the use of environment variables for sensitive data.

### Recommended Fixes:

1. Use environment variables for Stripe API keys.
2. Implement robust logging practices to avoid exposing sensitive information.
3. Validate all inputs thoroughly, especially `registration_data`.
4. Ensure proper authentication and authorization checks are in place before processing user requests.

By addressing these issues, the overall security posture of the code can be significantly improved.

---

*Generated by CodeWorm on 2026-02-20 01:36*
