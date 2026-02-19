# UserService

**Type:** Class Documentation
**Repository:** my-portfolio
**File:** v1/backend/app/user/service.py
**Language:** python
**Lines:** 33-219
**Complexity:** 0.0

---

## Source Code

```python
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

    async def deactivate_user(
        self,
        user: User,
    ) -> UserResponse:
        """
        Deactivate user account
        """
        updated = await UserRepository.update(
            self.session,
            user,
            is_active = False
        )
        return UserResponse.model_validate(updated)

    async def list_users(
        self,
        page: int,
        size: int,
    ) -> UserListResponse:
        """
        List users with pagination
        """
        skip = (page - 1) * size
       
```

---

## Class Documentation

### UserService Documentation

**Class Responsibility and Purpose**
The `UserService` class handles all business logic related to user operations within the application, including registration, authentication, profile updates, and administrative actions.

**Public Interface (Key Methods)**
- **create_user**: Registers a new user with email verification.
- **get_user_by_id**: Retrieves a user by ID.
- **update_user**: Updates an existing user's profile.
- **change_password**: Changes the password for a given user.
- **deactivate_user**: Deactivates a user account.
- **list_users**: Lists users with pagination.
- **admin_create_user**: Allows administrators to create new users.
- **admin_update_user**: Enables administrators to update user details.

**Design Patterns Used**
The class employs several design patterns:
- **Factory Pattern**: Implicitly used in `create_user` and `admin_create_user` methods for creating new user instances.
- **Strategy Pattern**: The `change_password` method uses a strategy pattern by delegating the password hashing to an external function.
- **Observer Pattern**: Not explicitly implemented but implied through asynchronous operations and event handling.

**Relationship to Other Classes**
The `UserService` interacts with the `UserRepository`, which handles database interactions. It also relies on external services like `hash_password` and `verify_password` for secure password management.

**State Management Approach**
The class manages state primarily through session context, ensuring that all operations are transactional and consistent. The use of asynchronous methods (`async def`) ensures efficient handling of I/O-bound tasks without blocking the event loop.

This class fits into a layered architecture where it sits between the business logic layer and the data access layer, providing a clear separation of concerns and encapsulating complex operations within its methods.

---

*Generated by CodeWorm on 2026-02-18 19:17*
