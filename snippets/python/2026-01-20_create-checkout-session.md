# create_checkout_session

**Repository:** CertGames-Core
**File:** backend/api/domains/account/controllers/subscription/stripe_ctrl.py
**Language:** python
**Lines:** 53-150
**Complexity:** 16.0

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
                    "price": current_app.config['STRIPE_PRICE_ID'],
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
        )
        raise BusinessRuleError(
            f"Failed to create checkout session: {str(e)}"
        ) from e

    except Exception as e:
        current_app.logger.error(
            f"Error creating checkout session: {str(e)}"
        )
        raise DatabaseError(
            f"Error creating checkout session: {str(e)}"
        ) from e
```

---

## Documentation

### Documentation for `create_checkout_session`

**Purpose and Behavior:**
The function `create_checkout_session` is responsible for creating a Stripe Checkout session for new subscriptions. It validates user input, processes referral codes, and sets up the necessary metadata and URLs to handle successful and canceled transactions.

**Key Implementation Details:**
- **User Input Validation:** The function retrieves user ID, registration data, OAuth flow status, and referral code from validated inputs.
- **Referral Code Handling:** Validates the referral code using `ReferralOperations.validate_code` and logs appropriate messages based on validation results.
- **URL Construction:** Constructs success and cancel URLs dynamically based on frontend configuration and user details.
- **Metadata Setup:** Sets metadata for the Stripe session, including user ID, registration status, OAuth flow flag, and validated referral code if applicable.
- **Error Handling:** Catches both Stripe API errors and general exceptions, logging detailed error messages and re-raising custom business or database errors.

**When/Why to Use:**
This function should be used whenever a new subscription needs to be initiated through Stripe Checkout. It handles complex scenarios like referral codes and OAuth flows while ensuring robust error handling and logging.

**Patterns and Gotchas:**
- **Context Managers:** The use of `current_app` for configuration access and logging is common in Flask applications.
- **Custom Exceptions:** Custom exceptions like `BusinessRuleError` and `DatabaseError` are raised to handle specific business logic errors, making the code more maintainable.
- **Session Management:** Storing temporary registration data in session variables can lead to potential issues if not managed properly. Ensure sessions are cleared after use to avoid memory leaks or security risks.

This function is a well-rounded example of handling complex subscription flows with Stripe in a Python application.

---

*Generated by CodeWorm on 2026-01-20 14:09*
