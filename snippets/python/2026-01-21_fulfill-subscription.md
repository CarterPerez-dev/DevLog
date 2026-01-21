# fulfill_subscription

**Repository:** CertGames-Core
**File:** backend/api/domains/account/controllers/subscription/stripe_ctrl.py
**Language:** python
**Lines:** 337-447
**Complexity:** 12.0

---

## Source Code

```python
def fulfill_subscription(session_data):
    """
    Process a completed checkout session (webhook handler)
    """
    customer_id = session_data.get("customer")
    session_id = session_data.get("id")

    current_app.logger.info(
        f"Fulfilling subscription for session: {session_id}, customer: {customer_id}"
    )

    try:
        user = _resolve_user_for_session(session_data, session_id)
        if not user:
            current_app.logger.error(
                f"Could not find or create user for session: {session_id}"
            )
            raise BusinessRuleError(
                f"Could not find or create user for session: {session_id}"
            )

        user_id = str(user.id)

        metadata = session_data.get("metadata", {})
        referral_code = metadata.get("referral_code")
        if referral_code and not user.referralCodeUsed:
            try:
                from api.domains.account.models.User import User
                User.objects(id = user.id).update_one(
                    set__referralCodeUsed = referral_code
                )
                user.reload()
                current_app.logger.info(
                    f"Stored referral code {referral_code} for user {user_id}"
                )
            except Exception as e:
                current_app.logger.error(
                    f"Failed to store referral code for user {user_id}: {str(e)}"
                )

        subscription_id, subscription, current_period_end = _get_subscription_details(session_data)

        update_data = {
            "subscriptionActive": True,
            "subscriptionStatus": "active",
            "subscriptionPlatform": "stripe",
            "stripeCustomerId": customer_id,
            "subscriptionStartDate": datetime.now(UTC),
            "subscriptionType": "premium",
            "subscriptionCanceledAt": None,
        }

        if subscription:
            update_data["stripeSubscriptionId"] = subscription.id
            if current_period_end:
                update_data["subscriptionEndDate"] = current_period_end

        SubscriptionService.update_subscription_data(user_id, update_data)

        if user.referralCodeUsed:
            try:
                amount_paid = session_data.get("amount_total")
                amount_in_dollars = amount_paid / 100 if amount_paid else None

                current_app.logger.info(
                    f"Tracking referral conversion for user {user_id} with code: {user.referralCodeUsed}"
                )

                conversion_result = ReferralOperations.track_conversion(
                    code = user.referralCodeUsed,
                    referred_user_id = user_id,
                    subscription_id = subscription_id,
                    amount = amount_in_dollars
                )

                current_app.logger.info(
                    f"Referral conversion tracked successfully: {conversion_result}"
                )

            except Exception as referral_error:
                current_app.logger.error(
                    f"Failed to track referral conversion for user {user_id}: {str(referral_error)}",
                    exc_info = True
                )

        AuditLogger.log_action(
            "subscription_activated",
            user_id = user_id,
            data = {
                "platform": "stripe",
                "customer_id": customer_id,
                "subscription_id": subscription_id,
                "session_id": session_id,
            },
        )

        if session.get("temp_registration_data"):
            session.pop("temp_registration_data", None)
            session.pop("is_oauth_flow", None)

        TempRegistrationService.delete_by_session_id(session_id)

        current_app.logger.info(
            f"Successfully fulfilled subscription for user: {user_id}"
        )

    except Exception as e:
        current_app.logger.error(
            f"Error fulfilling subscription: {str(e)}"
        )
        raise
```

---

## Documentation

### Documentation for `fulfill_subscription` Function

**Purpose and Behavior:**
The `fulfill_subscription` function processes a completed checkout session by updating user subscription status, handling referral codes, and logging actions. It is triggered via a webhook handler.

**Key Implementation Details:**
1. **Logging:** Uses Flask's logger to log information and errors.
2. **User Resolution:** Retrieves or creates the user associated with the session.
3. **Referral Handling:** Updates the user’s referral code usage status if applicable, tracking conversions.
4. **Subscription Data Update:** Sets subscription details like active status, start date, type, etc., using `SubscriptionService`.
5. **Audit Logging:** Logs actions related to subscription activation.

**When/Why to Use:**
This function is crucial for handling Stripe webhook events in a web application, ensuring that user subscriptions are correctly updated and managed. It should be used whenever a subscription needs to be activated or updated post-payment.

**Patterns and Gotchas:**
- **Exception Handling:** Comprehensive try-except blocks ensure robust error logging but may lead to complex nested structures.
- **Dependencies:** Functions like `_resolve_user_for_session`, `SubscriptionService.update_subscription_data`, and `ReferralOperations.track_conversion` are critical and must be correctly implemented for this function to work as expected.
- **Asynchronous Operations:** The use of `User.objects().update_one()` suggests potential asynchronous operations, which should be managed carefully.

---

*Generated by CodeWorm on 2026-01-21 14:23*
