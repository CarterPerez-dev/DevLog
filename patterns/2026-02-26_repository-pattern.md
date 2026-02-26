# repository_pattern

**Type:** Pattern Analysis
**Repository:** my-portfolio
**File:** v1/backend/app/user/service.py
**Language:** python
**Lines:** 1-220
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
service.py
"""

from uuid import UUID
from sqlalchemy.ext.asyncio import (
    AsyncSession,
)

from config import settings, UserRole
from core.exceptions import (
    EmailAlreadyExists,
    InvalidCredentials,
    UserNotFound,
)
from core.security import (
    hash_password,
    verify_password,
)
from .schemas import (
    AdminUserCreate,
    UserCreate,
    UserListResponse,
    UserResponse,
    UserUpdate,
    UserUpdateAdmin,
)
from .User import User
from .repository import UserRepository


class UserService:
    """
    Business logic for user operations
    """
    def __init__(self, session: AsyncSession) -> None:
        self.session = session

    async def create_user(
        self,
        user_data: UserCreate,
    ) -> UserResponse:
        """
        Register a new user
        """
        if await UserRepository.email_exists(self.session, user_data.email):
            raise EmailAlreadyExists(user_data.email)

        role = UserRole.USER
        if settings.ADMIN_EMAIL and user_data.email.lower(
        ) == settings.ADMIN_EMAIL.lower():
            role = UserRole.ADMIN

        hashed = await hash_password(user_data.password)
        user = await UserRepository.create_user(
            self.session,
            email = user_data.email,
            hashed_password = hashed,
            full_name = user_data.full_name,
            role = role,
        )
        return UserResponse.model_validate(user)

    async def get_user_by_id(
        self,
        user_id: UUID,
    ) -> UserResponse:
        """
        Get user by ID
        """
        user = await UserRepository.get_by_id(self.session, user_id)
        if not user:
            raise UserNotFound(str(user_id))
        return UserResponse.model_validate(user)

    async def get_user_model_by_id(
        self,
        user_id: UUID,
    ) -> User:
        """
        Get user model by ID (for internal use)
        """
        user = await UserRepository.get_by_id(self.session, user_id)
        if not user:
            raise UserNotFound(str(user_id))
        return user

    async def update_user(
        self,
        user: User,
        user_data: UserUpdate,
    ) -> UserResponse:
        """
        Update user profile
        """
        update_dict = user_data.model_dump(exclude_unset = True)
        updated_user = await UserRepository.update(
            self.session,
            user,
            **update_dict
        )
        return UserResponse.model_validate(updated_user)

    async def change_password(
        self,
        user: User,
        current_password: str,
        new_password: str,
    ) -> None:
        """
        Change user password
        """
        is_valid, _ = await verify_password(current_password, user.hashed_password)
        if not is_valid:
            raise InvalidCredentials()

        hashed = await hash_password(new_password)
        await UserRepository.update_password(self.session, user, hashed)

    a
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

#### Implementation
The `UserService` class in the provided code uses the **Repository Pattern** to encapsulate data access logic. This pattern is implemented via the `UserRepository` interface, which handles database operations like creating, updating, and retrieving users.

- **Methods**: The `UserService` interacts with methods from `UserRepository`, such as `create_user`, `get_by_id`, `update`, etc.
- **Dependency Injection**: The `UserService` constructor accepts an `AsyncSession` object to interact with the database, ensuring that data access logic is encapsulated within the repository.

#### Benefits
1. **Separation of Concerns**: By separating business logic from data access, the code becomes more modular and easier to maintain.
2. **Testability**: Repository methods can be easily mocked or replaced for unit testing, making the service layer testable in isolation.
3. **Flexibility**: Changes in database technology or storage mechanisms do not affect the business logic.

#### Deviations
- The `UserService` class directly handles some validation and exception handling (e.g., `EmailAlreadyExists`, `InvalidCredentials`) that could be moved to the repository for a more consistent pattern application.
- Some methods like `admin_create_user` and `admin_update_user` are specific to admin actions, which might benefit from their own dedicated repositories or services.

#### Appropriateness
The Repository Pattern is highly appropriate here because:
- The codebase needs clear separation between business logic and data access.
- There is a need for robust validation and exception handling in the service layer.
- Future changes in database technology can be managed without affecting the core business logic.

---

*Generated by CodeWorm on 2026-02-26 10:57*
