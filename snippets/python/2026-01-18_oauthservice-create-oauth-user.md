# OAuthService.create_oauth_user

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
            ) from nue
        except MongoValidationError as ve:
            raise ValidationError(
                f"Validation error creating OAuth user: {str(ve)}"
            ) from ve
        except Exception as e:
            raise DatabaseError(
                f"Error creating OAuth user: {str(e)}"
            ) from e
```

---

## Documentation

### Documentation for `create_oauth_user`

**Purpose and Behavior:**
The function `create_oauth_user` is responsible for creating a new user via OAuth in the system. It takes an email, optional name, provider, and provider ID as inputs, generates a unique username and secure password, and saves the user to the database. The function handles different OAuth providers (Google, Apple, GitHub) by setting specific IDs based on the provider.

**Key Implementation Details:**
- **Type Hints:** The function uses type hints for parameters like `email` and return types.
- **Username Generation:** A base username is derived from the email, then a unique username is generated using `OAuthService.generate_unique_username`.
- **Conditional Attribute Assignment:** Depending on the provider, specific IDs (like `google_id`, `apple_id`, or `github_id`) are assigned to the user object.
- **Name Parsing:** If a name is provided, it splits into first and last names if applicable.
- **Error Handling:** The function includes comprehensive error handling for unique constraint violations, validation errors, and general database errors.

**When/Why to Use:**
This code should be used when creating new users via OAuth. It ensures that the user data is properly validated and stored, handling different providers with specific attributes. This function simplifies the process of integrating various OAuth services while maintaining robust error handling.

**Patterns/Gotchas:**
- **Unique Constraints:** The function checks for unique constraints on `github_id`, `google_id`, and `apple_id` before raising a `ValidationError`.
- **Generic Exception Handling:** A broad exception block catches unexpected errors, which should be handled more specifically in production to avoid masking real issues.

---

*Generated by CodeWorm on 2026-01-18 00:35*
