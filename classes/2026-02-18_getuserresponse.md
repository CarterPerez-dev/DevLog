# GetUserResponse

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/CertGamesAPI.py
**Language:** python
**Lines:** 82-156
**Complexity:** 0.0

---

## Source Code

```python
class GetUserResponse(BaseModel):
    """
    Response schema for GET /account/user
    """
    id: str = Field(..., alias = "_id", description = "User ID (ObjectId as string)")
    username: str = Field(..., description = "User's username")
    email: str = Field(..., description = "User's email address")
    coins: float = Field(..., description = "User's coin balance")
    xp: float = Field(..., description = "User's XP points")
    level: int = Field(..., description = "User's current level")
    achievements: list[str] = Field(..., description = "User's achievement IDs as strings")
    achievement_counters: dict[str, Any] | None = Field(..., description = "Achievement counters dictionary")
    currentAvatar: str | None = Field(..., description = "Current equipped avatar ID as string")
    nameColor: str | None = Field(..., description = "User's name color")
    xpBoost: float = Field(..., description = "User's XP boost multiplier")
    purchasedItems: list[str] = Field(..., description = "Purchased item IDs as strings")
    lastDailyClaim: str | None = Field(..., description = "Last daily claim timestamp (ISO format)")
    tags: list[str] | None = Field(..., description = "User tags")
    signUpSource: str | None = Field(..., description = "Sign up source")
    subscriptionActive: bool = Field(..., description = "Whether subscription is active")
    subscriptionType: str = Field(..., description = "Subscription type")
    subscriptionPlan: str = Field(..., description = "Subscription plan")
    subscriptionStatus: str | None = Field(..., description = "Subscription status")
    subscriptionPlatform: str | None = Field(..., description = "Subscription platform")
    subscriptionStartDate: str | None = Field(..., description = "Subscription start date (ISO format)")
    subscriptionEndDate: str | None = Field(..., description = "Subscription end date (ISO format)")
    subscriptionCanceledAt: str | None = Field(..., description = "Subscription cancellation date (ISO format)")
    practiceQuestionsRemaining: int = Field(..., description = "Remaining practice questions")
    freeUserCreatedAt: str | None = Field(..., description = "Free user creation date (ISO format)")
    oauth_provider: str | None = Field(..., description = "OAuth provider")
    needs_username: bool = Field(..., description = "Whether user needs to set username")
    subscription_choice_made: bool = Field(..., description = "Whether subscription choice was made")
    lastLoginAt: str | None = Field(..., description = "Last login timestamp (ISO format)")
    lastLoginIP: str | None = Field(..., description = "Last login IP address")
    currentIP: str | None = Field(..., description = "Current IP address")
    isOnline: bool = Field(..., description = "Whether user is online")
    lastSeenAt: str | None = Field(..., description = "Last seen timestamp (ISO format)")
    userAgent: str | None = Field(..., description = "User agent string")
    is_admin: bool = Field(..., descrip
```

---

## Class Documentation

### GetUserResponse

**Class Responsibility and Purpose:**
The `GetUserResponse` class is a Pydantic model that defines the schema for the response object when retrieving user data via the `/account/user` endpoint. It encapsulates all the fields required to represent a user's detailed information, including basic details like ID, username, and email, as well as more complex attributes such as achievements, subscription status, and streaks.

**Public Interface:**
- The class does not define any methods; it is purely a data model.
- Key Fields:
  - `id`: User ID (ObjectId as string).
  - `username`, `email`: Basic user details.
  - `coins`, `xp`, `level`: User's in-game resources and progress.
  - `achievements`, `achievement_counters`: Achievement-related information.
  - `currentAvatar`, `nameColor`, `xpBoost`: Visual and gameplay attributes.
  - `purchasedItems`, `lastDailyClaim`: In-app purchases and daily activities.
  - `tags`, `signUpSource`: Additional metadata about the user.

**Design Patterns Used:**
- **Data Model**: The class uses Pydantic's BaseModel to define a data model, which is a common pattern for handling structured data in Python applications. It leverages type hints and validation to ensure that the data conforms to expected formats.

**Relationship to Other Classes:**
- This class is part of the `CertGamesAPI` module within the CertGames-Core backend API. It interacts with other models and services to retrieve and format user data for responses.
- The `GetUserResponse` model is used in conjunction with APIs that handle user-related operations, ensuring consistent and validated data representation.

**State Management Approach:**
The class does not manage state; it merely defines the structure of the response. State management is handled by the underlying services and APIs that use this model to construct responses.

---

*Generated by CodeWorm on 2026-02-18 15:24*
