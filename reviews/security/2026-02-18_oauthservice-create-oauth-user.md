# OAuthService.create_oauth_user

**Type:** Security Review
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

## Security Review

### Security Review for `OAuthService.create_oauth_user`

#### Vulnerabilities Found:

1. **Input Validation Gaps** (Medium):
   - The function does not validate the input parameters (`email`, `name`, `provider`, and `provider_id`). This could lead to unexpected behavior or errors.
   
2. **Error Handling that Leaks Information** (Low):
   - The generic exception handling can leak information about the internal structure of the application, especially when raising a `DatabaseError`.

3. **Hardcoded Secrets or Credentials** (Info):
   - No hardcoded secrets are found in this snippet.

4. **Insecure Deserialization** (Info):
   - No deserialization is present in the code.

5. **Race Conditions and TOCTOU Bugs** (Low):
   - Potential race conditions could occur if multiple users attempt to create an account with the same `provider_id` simultaneously, leading to duplicate entries or unexpected behavior.

#### Attack Vectors:

- An attacker could exploit input validation gaps by providing malformed data.
- Generic exceptions can reveal sensitive information about database structures and operations.

#### Recommended Fixes:

1. **Input Validation**:
   - Add type and format checks for `email`, `name`, `provider`, and `provider_id` to ensure they meet expected criteria (e.g., email format, non-empty strings).

2. **Error Handling**:
   - Use more specific exception handling instead of generic `Exception`. For example, handle `NotUniqueError` separately from other exceptions.

3. **Race Conditions**:
   - Implement optimistic locking or use transactions to ensure that concurrent operations do not overwrite each other's data.

#### Overall Security Posture:

The code has some security gaps but is generally well-structured. Addressing the input validation and error handling issues will significantly improve its robustness and security posture.

---

*Generated by CodeWorm on 2026-02-18 22:12*
