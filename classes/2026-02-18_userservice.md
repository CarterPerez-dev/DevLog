# UserService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/auth/services/user.py
**Language:** python
**Lines:** 32-219
**Complexity:** 0.0

---

## Source Code

```python
class UserService:
    """
    Business logic for user operations
    """
    @staticmethod
    async def create_user(
        session: AsyncSession,
        user_data: UserCreate,
    ) -> UserResponse:
        """
        Register a new user
        """
        if await UserRepository.email_exists(session, user_data.email):
            raise EmailAlreadyExists(user_data.email)

        hashed = await hash_password(user_data.password)
        user = await UserRepository.create_user(
            session,
            email = user_data.email,
            hashed_password = hashed,
            full_name = user_data.full_name,
        )
        return UserResponse.model_validate(user)

    @staticmethod
    async def get_user_by_id(
        session: AsyncSession,
        user_id: UUID,
    ) -> UserResponse:
        """
        Get user by ID
        """
        user = await UserRepository.get_by_id(session, user_id)
        if not user:
            raise UserNotFound(str(user_id))
        return UserResponse.model_validate(user)

    @staticmethod
    async def get_user_model_by_id(
        session: AsyncSession,
        user_id: UUID,
    ) -> User:
        """
        Get user model by ID (for internal use)
        """
        user = await UserRepository.get_by_id(session, user_id)
        if not user:
            raise UserNotFound(str(user_id))
        return user

    @staticmethod
    async def update_user(
        session: AsyncSession,
        user: User,
        user_data: UserUpdate,
    ) -> UserResponse:
        """
        Update user profile
        """
        update_dict = user_data.model_dump(exclude_unset = True)
        updated_user = await UserRepository.update(
            session,
            user,
            **update_dict
        )
        return UserResponse.model_validate(updated_user)

    @staticmethod
    async def change_password(
        session: AsyncSession,
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
        await UserRepository.update_password(session, user, hashed)

    @staticmethod
    async def deactivate_user(
        session: AsyncSession,
        user: User,
    ) -> UserResponse:
        """
        Deactivate user account
        """
        updated = await UserRepository.update(
            session,
            user,
            is_active = False
        )
        return UserResponse.model_validate(updated)

    @staticmethod
    async def list_users(
        session: AsyncSession,
        page: int,
        size: int,
    ) -> UserListResponse:
        """
        List users with pagination
        """
        skip = (page - 1) * size
        users = await UserRepository.get_multi(
            session,
      
```

---

## Class Documentation

### UserService Documentation

**Class Responsibility and Purpose:**
The `UserService` class is responsible for handling all user-related business logic within the application. It encapsulates operations such as creating, updating, deactivating, listing users, and managing user passwords. This class ensures that user data integrity and security are maintained throughout these operations.

**Public Interface (Key Methods):**
- **create_user**: Registers a new user by validating email uniqueness and hashing the password.
- **get_user_by_id**: Retrieves a user by their ID.
- **update_user**: Updates a user's profile based on provided data.
- **change_password**: Changes a user’s password after verifying the current one.
- **deactivate_user**: Deactivates a user account.
- **list_users**: Lists users with pagination support.
- **admin_create_user**: Allows administrators to create new users.
- **admin_update_user**: Enables administrators to update existing users.

**Design Patterns Used:**
The class leverages several design patterns:
- **Factory Pattern**: Implicitly used through the `UserRepository` for creating and updating user records.
- **Strategy Pattern**: The `hash_password` function is a strategy for password hashing, which can be extended or replaced if needed.
- **Observer Pattern**: Although not explicitly implemented, this pattern could be applied to notify other services when a user's state changes (e.g., deactivation).

**How It Fits in the Architecture:**
The `UserService` class acts as a central hub for all user-related operations. It interacts with the `UserRepository` to persist and retrieve data from the database. This separation of concerns enhances modularity, making it easier to test and maintain individual components. The class also ensures that business rules are enforced at the application level, providing a robust layer between the database and the rest of the application logic.

---

*Generated by CodeWorm on 2026-02-18 16:09*
