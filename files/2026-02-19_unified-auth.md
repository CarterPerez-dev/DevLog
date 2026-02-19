# unified_auth

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/api/core/auth/unified_auth.py
**Language:** python
**Lines:** 1-177
**Complexity:** 0.0

---

## Source Code

```python
"""
Unified Authentication System - Connects User and AdminUser models
/api/core/auth/unified_auth.py
"""

from typing import Any
from api.admin.roles.models import AdminRole
from api.domains.account.models.User import User
from api.admin.models.users.AdminUser import AdminUser
from api.core.validation.exceptions import NotFoundError
from api.domains.account.services.auth import AuthService


class UnifiedAuthService:
    """
    Dual role authentication between User and AdminUser models
    """
    @staticmethod
    def create_user_admin_link(
        user_email: str,
        admin_role: AdminRole = AdminRole.VIEWER,
        created_by: str | None = None
    ) -> dict[str,
              Any]:
        """
        Create an admin account linked to existing user via shared email
        """
        user = User.objects(email = user_email.lower()).first()
        if not user:
            raise NotFoundError(f"User with email {user_email} not found")

        existing_admin = AdminUser.objects(email = user_email.lower()
                                           ).first()
        if existing_admin:
            return {
                "user_id": str(user.id),
                "admin_id": str(existing_admin.id),
                "was_created": False
            }

        admin_user = AdminUser.create_admin(
            email = user.email,
            role = admin_role,
            created_by = created_by or "unified_auth_system"
        )

        if user.username and not admin_user.name:
            admin_user.name = user.username
            admin_user.save()

        user.is_admin = True
        user.save()

        return {
            "user_id": str(user.id),
            "admin_id": str(admin_user.id),
            "user_email": user.email,
            "admin_role": admin_role.value,
            "was_created": True
        }

    @staticmethod
    def create_admin_user_link(
        admin_email: str,
        username: str,
        password: str,
        **user_kwargs
    ) -> dict[str,
              Any]:
        """
        Create a user account linked to existing admin via shared email
        """
        admin = AdminUser.objects(email = admin_email.lower()).first()
        if not admin:
            raise NotFoundError(
                f"Admin with email {admin_email} not found"
            )

        existing_user = User.objects(email = admin_email.lower()).first()
        if existing_user:
            return {
                "user_id": str(existing_user.id),
                "admin_id": str(admin.id),
                "was_created": False
            }

        user = AuthService.create_user(
            username = username,
            email = admin.email,
            password = password,
            is_admin = True,
            **user_kwargs
        )

        if not user:
            raise NotFoundError("Failed to create user account")

        return {
            "user_id": str(user.id),
            "admin_id": str(admin.id),
            "u
```

---

## File Overview

Unified Authentication System - Connects User and AdminUser models

**Purpose:**
This file implements a dual role authentication system, allowing users to have admin privileges through an `AdminUser` account linked to their `User` account. It handles the creation and management of these links.

**Key Exports & Public API:**
- `create_user_admin_link`: Creates an admin account for an existing user.
- `create_admin_user_link`: Creates a user account for an existing admin.
- `get_admin_for_user`: Retrieves the admin account linked to a user.
- `get_user_for_admin`: Retrieves the user account linked to an admin.
- `has_admin_access`: Checks if a user has admin access via an `AdminUser` account.
- `upgrade_user_to_admin`: Upgrades an existing user to have admin access.
- `revoke_admin_access`: Revokes admin access for a user by removing their `AdminUser` account.

**Project Fit:**
This module is crucial for managing roles and permissions within the application, ensuring that users can perform administrative tasks when necessary. It integrates with both the `User` and `AdminUser` models to maintain consistency across different parts of the system.

**Design Decisions:**
- **Dual Role Management:** The design supports a dual role model where a user can have admin privileges through an associated `AdminUser` account, providing flexibility in managing permissions.
- **Error Handling:** Robust error handling is implemented using exceptions like `NotFoundError`, ensuring that issues are clearly communicated to the calling code.
- **Static Methods:** All methods are static, making them easily callable without needing to instantiate the class, which aligns with their utility nature.

This file plays a central role in managing user and admin roles, providing a seamless way to link users and admins while maintaining clear separation of concerns.

---

*Generated by CodeWorm on 2026-02-19 11:14*
