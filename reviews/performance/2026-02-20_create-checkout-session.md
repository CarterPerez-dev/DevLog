# create_checkout_session

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of `create_checkout_session` is primarily determined by the Stripe API call, which has a constant time complexity \(O(1)\). However, the validation of the referral code and logging operations can introduce minor overhead.

#### Space Complexity
Space complexity is low as the function uses a few local variables. The main memory usage comes from temporary data structures like `metadata` and `success_url`. There are no significant space concerns here.

#### Bottlenecks or Inefficiencies
1. **Redundant Operations**: The validation of the referral code is performed even if it's not provided, which can be optimized by checking for its presence first.
2. **Logging Overhead**: Logging every successful and failed validation attempt might introduce unnecessary overhead. Consider logging only critical errors.
3. **Exception Handling**: Multiple `try-except` blocks can lead to performance degradation due to exception handling overhead.

#### Optimization Opportunities
1. **Referral Code Validation Check**: Move the referral code validation check outside the try block to avoid redundant operations:
   ```python
   if referral_code:
       try:
           # ... validation logic ...
       except Exception as e:
           current_app.logger.error(f"Error validating referral code: {str(e)}")
   ```

2. **Simplify Logging**: Log only critical errors, and consider using structured logging for better performance.

3. **Reduce Exception Handling Overhead**: Consolidate exception handling to a single block if possible, or use more specific exceptions to reduce overhead:
   ```python
   try:
       # ... main logic ...
   except (stripe.error.StripeError, BusinessRuleError, DatabaseError) as e:
       current_app.logger.error(f"Error creating checkout session: {str(e)}")
       raise
   ```

#### Resource Usage Concerns
- **Session Handling**: Ensure that `session` is properly managed to avoid resource leaks.
- **Stripe API Calls**: Minimize the number of API calls by batching operations where possible.

By addressing these points, you can improve the efficiency and performance of the function.

---

*Generated by CodeWorm on 2026-02-20 20:56*
