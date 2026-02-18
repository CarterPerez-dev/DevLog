# AdminUserLinker

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/devtools/scripts/link_admin_to_user.py
**Language:** python
**Lines:** 31-147
**Complexity:** 0.0

---

## Source Code

```python
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
        Create the admin-user link
        """
        print(f"\n Creating user account for admin...")

        username = self.generate_username(admin)
        temp_password = f"temp_admin_password_123"

        try:
            result = UnifiedAuthService.create_admin_user_link(
                admin_email = admin.email,
                username = username,
                password = temp_password
            )

            print(f"  Successfully created linked user account!")
            print(f"   User ID: {result['user_id']}")
            print(f"   Admin ID: {result['admin_id']}")
            print(f"   Username: {username}")
            print(f"   Email: {admin.email}")
            print(f"   Temp Password: {te
```

---

## Class Documentation

### AdminUserLinker Documentation

**Class Responsibility and Purpose:**
The `AdminUserLinker` class is responsible for linking an admin account to a user account within a unified authentication system, ensuring that admins can access user endpoints without needing separate credentials.

**Public Interface (Key Methods):**
- **`__init__(self, admin_email: str = None)`**: Initializes the class with an optional admin email.
- **`connect_database(self) -> None`**: Establishes a connection to MongoDB for database operations.
- **`find_admin_account(self) -> AdminUser`**: Locates the admin account based on provided or default configuration settings.
- **`check_existing_link(self, admin: AdminUser) -> bool`**: Verifies if an existing user account is already linked to the admin.
- **`generate_username(self, admin: AdminUser) -> str`**: Generates a unique username from the admin's email address.
- **`create_link(self, admin: AdminUser) -> None`**: Creates and links the admin to a new or existing user account.
- **`run(self) -> None`**: Executes the entire linking process.

**Design Patterns Used:**
The class employs the **Factory Method** pattern for creating the admin-user link through `UnifiedAuthService.create_admin_user_link()`. It also uses simple state management, where the class maintains its internal state (e.g., admin email and configuration settings) to drive the linking process.

**How it Fits in the Architecture:**
`AdminUserLinker` is a utility script designed for development and testing purposes. It operates as an external tool that interacts with the main application's authentication services (`UnifiedAuthService`). The class fits into the architecture by providing a controlled environment where admin accounts can be easily linked to user accounts, facilitating unified access management without altering the core application logic.

This documentation provides a clear understanding of how `AdminUserLinker` operates within the CertGames-Core backend system.

---

*Generated by CodeWorm on 2026-02-18 12:44*
