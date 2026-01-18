# User.to_dict

**Repository:** CertGames-Core
**File:** backend/api/domains/account/models/User.py
**Language:** python
**Lines:** 446-610
**Complexity:** 20.0

---

## Source Code

```python
def to_dict(self) -> dict[str, Any]:
        """
        Convert user to dictionary for API responses
        """
        data: dict[str,
                   Any] = {
                       "_id":
                       str(self.id),
                       "username":
                       self.username,
                       "email":
                       self.email,
                       "coins":
                       self.coins,
                       "xp":
                       self.xp,
                       "level":
                       self.level,
                       "achievements": [
                           ach.get('achievementId')
                           if isinstance(ach,
                                         dict) else ach.achievementId
                           for ach in self.achievements_v2
                       ] if self.achievements_v2 else [],
                       "achievement_counters":
                       _serialize_dict_values(self.achievement_counters)
                       if self.achievement_counters else {},
                       "currentAvatar":
                       str(self.currentAvatar)
                       if self.currentAvatar else None,
                       "nameColor":
                       self.nameColor,
                       "xpBoost":
                       self.xpBoost,
                       "purchasedItems":
                       [str(item_id) for item_id in self.purchasedItems]
                       if self.purchasedItems else [],
                       "lastDailyClaim":
                       self.lastDailyClaim.isoformat()
                       if self.lastDailyClaim else None,
                       "tags":
                       self.tags,
                       "signUpSource":
                       self.signUpSource,
                       "subscriptionActive":
                       self.subscriptionActive,
                       "subscriptionType":
                       self.subscriptionType,
                       "subscriptionPlan":
                       self.subscriptionPlan,
                       "subscriptionStatus":
                       self.subscriptionStatus,
                       "subscriptionPlatform":
                       self.subscriptionPlatform,
                       "subscriptionStartDate":
                       self.subscriptionStartDate.isoformat()
                       if self.subscriptionStartDate else None,
                       "subscriptionEndDate":
                       self.subscriptionEndDate.isoformat()
                       if self.subscriptionEndDate else None,
                       "subscriptionCanceledAt":
                       self.subscriptionCanceledAt.isoformat()
                       if self.subscriptionCanceledAt else None,
                       "practiceQuestionsRemaining":
                       self.practiceQuestionsRemaining,
                       "freeUserCreatedAt":
                       self.freeUserCreatedAt.isoformat()
                       if self.freeUserCreatedAt else None,
                       "oauth_provider":
                       self.oauth_provider,
                       "needs_username":
                       self.needs_username,
                       "subscription_choice_made":
                       self.subscription_choice_made,
                       "lastLoginAt":
                       self.lastLoginAt.isoformat()
                       if self.lastLoginAt else None,
                       "lastLoginIP":
                       self.lastLoginIP,
                       "currentIP":
                       self.currentIP,
                       "isOnline":
                       self.isOnline,
                       "lastSeenAt":
                       self.lastSeenAt.isoformat()
                       if self.lastSeenAt else None,
                       "userAgent":
                       self.userAgent,
                       "is_admin":
                       self.is_admin,
                       "career_interest":
                       self.career_interest,
                       "skill_level":
                       self.skill_level,
                       "active_certification":
                       self.active_certification,
                       "onboarding_completed":
                       self.onboarding_completed,
                       "equipmentHistory": [
                           change.to_dict() if hasattr(change,
                                                       'to_dict') else
                           _serialize_dict_values(change)
                           for change in self.equipmentHistory
                       ] if self.equipmentHistory else [],
                       "confirmPassword":
                       self.confirmPassword,
                       "stripeCustomerId":
                       self.stripeCustomerId,
                       "stripeSubscriptionId":
                       self.stripeSubscriptionId,
                       "appleTransactionId":
                       self.appleTransactionId,
                       "appleOriginalTransactionId":
                       self.appleOriginalTransactionId,
                       "appleProductId":
                       self.appleProductId,
                       "current_streak":
                       self.current_streak,
                       "longest_streak":
                       self.longest_streak,
                       "last_activity_date":
                       self.last_activity_date.isoformat()
                       if self.last_activity_date else None,
                       "streak_freeze_count":
                       self.streak_freeze_count,
                       "streak_last_freeze_used":
                       self.streak_last_freeze_used.isoformat()
                       if self.streak_last_freeze_used else None,
                       "streak_milestones_claimed":
                       self.streak_milestones_claimed,
                       "current_role":
                       self.current_role,
                       "next_role":
                       self.next_role,
                       "referralCodeUsed":
                       self.referralCodeUsed,
                       "referredBy":
                       self.referredBy,
                       "paymentMethod":
                       self.paymentMethod,
                       "paypalEmail":
                       self.paypalEmail,
                       "venmoHandle":
                       self.venmoHandle,
                       "cashAppUsername":
                       self.cashAppUsername,
                       "zellePhoneOrEmail":
                       self.zellePhoneOrEmail,
                       "bankAccountNumber":
                       self.bankAccountNumber,
                       "bankRoutingNumber":
                       self.bankRoutingNumber,
                       "bankAccountHolderName":
                       self.bankAccountHolderName,
                       "bitcoinAddress":
                       self.bitcoinAddress,
                       "stripeConnectAccountId":
                       self.stripeConnectAccountId,
                       "stripeConnectOnboarded":
                       self.stripeConnectOnboarded,
                       "stripeConnectPayoutsEnabled":
                       self.stripeConnectPayoutsEnabled,
                   }

        return data
```

---

## Documentation

### Documentation for `User.to_dict`

**Purpose and Behavior:**
The `to_dict` method converts a `User` object into a dictionary suitable for API responses. It includes various fields such as user details, achievements, subscriptions, and financial information.

**Key Implementation Details:**
- The function uses type hints to specify that it returns a dictionary of any type.
- It constructs the dictionary by accessing attributes of the `User` instance, converting IDs to strings where necessary, and handling nested objects like `achievements_v2`.
- Conditional logic ensures fields are included only if they exist or have valid data.

**When/Why to Use:**
This method is used when preparing a user object for API responses. It ensures that all relevant user information is properly formatted and ready for transmission, making it easier to handle and display user data in the frontend.

**Patterns and Gotchas:**
- The method uses list comprehensions and conditional expressions to dynamically include or exclude fields based on their existence.
- Type hints like `Any` are used to allow flexibility but should be reviewed for type safety.
- Ensure that all attributes accessed have appropriate getter methods or properties, as some fields require special handling (e.g., converting dates).

This method is a comprehensive way to serialize user data, making it versatile and robust for API interactions.

---

*Generated by CodeWorm on 2026-01-18 16:37*
