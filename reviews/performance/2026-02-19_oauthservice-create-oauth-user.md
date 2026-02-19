# OAuthService.create_oauth_user

**Type:** Performance Analysis
**Repository:** CertGames-Core
**File:** backend/api/domains/account/services/oauth.py
**Language:** python
**Lines:** 53-144
**Complexity:** 25.0

---

## Source Code

```python
def create_oauth_user(
        email: str,
        name: str | None = None,
        provider: str | None = None,
        provider_id: str | None = None,
    ) -> User:
        """
        Create a new user via OAuth with all necessary setup
        """
        try:
            base_username: str = email.split("@")[0]
            username: str = OAuthService.generate_unique_username(
                base_username
            )

            secure_password: str = secrets.token_urlsafe(
                Security.TOKEN_LENGTH
            )

            new_user: User = User(
                username = username,
                email = email,
                password = secure_password,
                oauth_provider = provider,
                needs_username = True,
            )

            if provider == "google" and provider_id:
                new_user.google_id = provider_id
            elif provider == "apple" and provider_id:
                new_user.apple_id = provider_id
            elif provider == "github" and provider_id:
                new_user.github_id = provider_id

            if name:
                name_parts: list[str] = name.split(" ", 1)
                if len(name_parts) > 1:
                    new_user.first_name = name_parts[0]
                    new_user.last_name = name_parts[1]
                else:
                    new_user.first_name = name

            new_user.save()

            AuthService.auto_equip_default_avatar(new_user)

            AuditLogger.log_action(
                "user_created",
                user_id = str(new_user.id),
                data = {
                    "provider": provider,
                    "email": email,
                    "method": "oauth"
                },
            )

            return new_user

        except NotUniqueError as nue:
            if "github_id" in str(
                    nue) and provider == "github" and provider_id:
                existing_user = User.objects(github_id = provider_id
                                             ).first()
                if existing_user:
                    return existing_user
            elif "google_id" in str(
                    nue) and provider == "google" and provider_id:
                existing_user = User.objects(google_id = provider_id
                                             ).first()
                if existing_user:
                    return existing_user
            elif "apple_id" in str(
                    nue) and provider == "apple" and provider_id:
                existing_user = User.objects(apple_id = provider_id
                                             ).first()
                if existing_user:
                    return existing_user
            elif "email" in str(nue):
                raise ValidationError(
                    f"Email {email} is already registered"
                ) from nue
            raise ValidationError(
                f"Duplicate user data: {str(nue)}"
       
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity (Big O Notation)
The function `create_oauth_user` has a time complexity of \(O(1)\) for the majority of operations, as most steps involve simple string manipulations and database queries. However, the nested try-except blocks introduce potential overhead due to exception handling.

#### Space Complexity
Space complexity is also \(O(1)\), assuming that the input parameters (`email`, `name`, etc.) do not grow with the size of the dataset. The primary memory usage comes from creating local variables and objects like `User` instances, which are constant regardless of input size.

#### Bottlenecks or Inefficiencies
- **Redundant Exception Handling:** The nested try-except blocks can be simplified to reduce redundancy.
- **Multiple Database Queries:** The function performs multiple queries (`User.objects(...).first()`) within the exception handlers. This can lead to inefficiency, especially if these queries are frequent.

#### Optimization Opportunities
1. **Simplify Exception Handling:**
   - Combine similar exceptions into a single block for cleaner code:
     ```python
     except NotUniqueError as nue:
         error_field = "email"
         if "github_id" in str(nue) and provider == "github":
             existing_user = User.objects(github_id=provider_id).first()
             if existing_user:
                 return existing_user
             error_field = "github_id"
         elif "google_id" in str(nue) and provider == "google":
             existing_user = User.objects(google_id=provider_id).first()
             if existing_user:
                 return existing_user
             error_field = "google_id"
         elif "apple_id" in str(nue) and provider == "apple":
             existing_user = User.objects(apple_id=provider_id).first()
             if existing_user:
                 return existing_user
             error_field = "apple_id"
         raise ValidationError(f"Duplicate {error_field} data: {str(nue)}") from nue
     ```

2. **Batch Queries:** Consider using batch queries or caching to reduce the number of database hits.

#### Resource Usage Concerns
- Ensure that all database connections are properly managed, especially if this function is called frequently.
- Use context managers for resources like file handles and database connections where applicable.

By addressing these areas, you can improve both the efficiency and maintainability of the code.

---

*Generated by CodeWorm on 2026-02-19 15:14*
