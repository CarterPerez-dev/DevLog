# link_admin_to_user

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/devtools/scripts/link_admin_to_user.py
**Language:** python
**Lines:** 1-171
**Complexity:** 0.0

---

## Source Code

```python
#!/usr/bin/env python3
"""
CertGames Admin-User Account Linker

USAGE:
    docker exec -it backend_service python devtools/seed/link_admin_to_user.py [--admin-email=email]
"""

import os
import sys
import argparse
from datetime import datetime, UTC


sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', '..'))

os.environ.setdefault('ENVIRONMENT', 'development')

try:
    from config import get_config
    from api.core.database import init_db
    from api.domains.account.models.User import User
    from api.admin.models.users.AdminUser import AdminUser
    from api.core.auth.unified_auth import UnifiedAuthService

except ImportError as e:
    print(f"Import Error: {e}")
    sys.exit(1)


class AdminUserLinker:
    """
    Links admin accounts to user accounts for unified authentication
    """
    def __init__(self, admin_email: str = None):
        self.admin_email = admin_email
        self.config = get_config()

    def connect_database(self) -> None:
        """
        Connect to MongoDB
        """
        try:
            print("Connecting to MongoDB...")
            init_db()
            print("Connected to database successfully")
        except Exception as e:
            print(f"Database connection failed: {e}")
            sys.exit(1)

    def find_admin_account(self) -> AdminUser:
        """
        Find the admin account to link
        """
        if self.admin_email:
            admin_email = self.admin_email
        else:
            admin_email = self.config.ADMIN_EMAILS[0]

        admin = AdminUser.objects(email = admin_email).first()

        if not admin:
            print(f"Admin account not found for email: {admin_email}")
            sys.exit(1)

        print(
            f"Found admin account: {admin.email} (Role: {admin.role.value})"
        )
        return admin

    def check_existing_link(self, admin: AdminUser) -> bool:
        """
        Check if admin already has linked user account
        """
        existing_user = UnifiedAuthService.get_user_for_admin(admin)
        if existing_user:
            print(f"   Admin already has linked user account!")
            print(f"   User ID: {existing_user.id}")
            print(f"   Username: {existing_user.username}")
            print(f"   Email: {existing_user.email}")
            print(f"   Is Admin Flag: {existing_user.is_admin}")
            print(
                "\n  Your admin account can already access user endpoints!"
            )
            return True
        return False

    def generate_username(self, admin: AdminUser) -> str:
        """
        Generate unique username from admin email
        """
        base_username = admin.email.split('@')[0]
        username = base_username

        counter = 1
        while User.objects(username = username).first():
            username = f"{base_username}{counter}"
            counter += 1

        return username

    def create_link(self, admin: AdminUser) -> None:
        """
        Create the 
```

---

## File Overview

# CertGames Admin-User Account Linker

**Purpose:**
This script links an admin account to a user account for unified authentication within the CertGames application.

**Key Exports and Public Interface:**
- `AdminUserLinker`: The main class responsible for linking processes.
  - `__init__(self, admin_email: str = None)`: Initializes with optional admin email.
  - `connect_database(self)`: Connects to MongoDB.
  - `find_admin_account(self) -> AdminUser`: Finds the admin account based on provided or default email.
  - `check_existing_link(self, admin: AdminUser) -> bool`: Checks if an existing user account is linked to the admin.
  - `generate_username(self, admin: AdminUser) -> str`: Generates a unique username from the admin's email.
  - `create_link(self, admin: AdminUser)`: Creates and links a new user account to the admin.
  - `run(self)`: Executes the linking process.

**How it Fits in the Project:**
This script is part of the `devtools` directory used for development tasks. It leverages the application's core modules like database initialization, authentication services, and models. The script ensures that admin accounts can access user endpoints by creating a linked user account if necessary.

**Notable Design Decisions:**
- **Modularity**: The class-based approach encapsulates the linking logic, making it reusable.
- **Error Handling**: Comprehensive error handling is implemented to manage database connections and authentication processes.
- **Configuration Management**: Uses configuration settings for default admin emails and other parameters.
- **Unification**: Integrates with unified auth services to ensure consistent user management across different parts of the application.
```

This documentation provides a high-level overview of the file's purpose, its key components, how it integrates into the project, and some notable design decisions.

---

*Generated by CodeWorm on 2026-02-19 07:18*
