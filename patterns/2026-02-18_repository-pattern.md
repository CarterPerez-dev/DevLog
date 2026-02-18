# repository_pattern

**Type:** Pattern Analysis
**Repository:** Cybersecurity-Projects
**File:** TEMPLATES/fullstack-template/backend/app/user/service.py
**Language:** python
**Lines:** 1-222
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
        if await UserRepository.email_exists(self.session,
                                             user_data.email):
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
        await UserRepository.updat
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used: Repository Pattern**

The `UserService` class in this code implements the **Repository Pattern**, which abstracts data access logic to separate business logic from database operations.

#### Implementation Details:
- The `UserService` interacts with a `UserRepository` for all user-related operations, such as creating, updating, and retrieving users.
- Methods like `create_user`, `get_user_by_id`, `update_user`, etc., delegate the actual data manipulation to methods in `UserRepository`.

```python
class UserService:
    def __init__(self, session: AsyncSession) -> None:
        self.session = session

    async def create_user(self, user_data: UserCreate) -> UserResponse:
        # Delegate to UserRepository
        user = await UserRepository.create_user(
            self.session,
            email=user_data.email,
            hashed_password=hashed,
            full_name=user_data.full_name,
            role=role,
        )
        return UserResponse.model_validate(user)
```

#### Benefits:
- **Encapsulation**: The `UserService` class focuses on business logic, while the `UserRepository` handles data access.
- **Testability**: Repository methods can be easily mocked or replaced for unit testing.
- **Flexibility**: Easier to change storage mechanisms (e.g., from SQL to NoSQL) without affecting business logic.

#### Deviations:
- The `UserService` class directly instantiates the session, which is not typical. Usually, a dependency injection mechanism would provide the session.
- Some methods like `get_user_model_by_id` are used internally but could be refactored for clarity and consistency.

#### Appropriateness:
This pattern is **appropriate** in this context because it clearly separates concerns, making the code more maintainable and testable. It ensures that business logic remains focused on what needs to happen with users, while data access is handled by a dedicated repository class.

---

*Generated by CodeWorm on 2026-02-18 07:30*
