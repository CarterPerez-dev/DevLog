# handle_github_auth_callback

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/api/domains/account/controllers/oauth/github_auth_ctrl.py
**Language:** python
**Lines:** 49-191
**Complexity:** 18.0

---

## Source Code

```python
def handle_github_auth_callback() -> str:
    """
    Handle GitHub OAuth callback for user authentication
    """
    code: str | None = g.validated.get("code")
    if not code:
        raise ValidationError("Missing authorization code")

    github_client_id = os.getenv("GH_CLIENT_ID")
    github_client_secret = os.getenv("GH_CLIENT_SECRET")

    if not github_client_id or not github_client_secret:
        raise ValidationError("GitHub OAuth credentials not configured")

    token_response: requests.Response = requests.post(
        "https://github.com/login/oauth/access_token",
        headers = {"Accept": "application/json"},
        data = {
            "client_id": github_client_id,
            "client_secret": github_client_secret,
            "code": code,
        },
        timeout = 60,
    )

    if token_response.status_code != 200:
        raise ValidationError("Failed to exchange authorization code")

    token_data: dict[str, Any] = token_response.json()
    access_token: str | None = token_data.get("access_token")

    if not access_token:
        raise ValidationError("No access token received from GitHub")

    user_response: requests.Response = requests.get(
        "https://api.github.com/user",
        headers = {"Authorization": f"Bearer {access_token}"},
        timeout = 60,
    )

    if user_response.status_code != 200:
        raise ValidationError(
            "Failed to retrieve user information from GitHub"
        )

    user_info: dict[str, Any] = user_response.json()

    email_response: requests.Response = requests.get(
        "https://api.github.com/user/emails",
        headers = {"Authorization": f"Bearer {access_token}"},
        timeout = 60,
    )

    if email_response.status_code != 200:
        raise ValidationError(
            "Failed to retrieve email information from GitHub"
        )

    emails = email_response.json()
    if not isinstance(emails, list):
        raise ValidationError("Invalid email response from GitHub")

    email = next(
        (
            e['email'] for e in emails if isinstance(e, dict)
            and e.get('primary') and e.get('verified')
        ),
        None
    )

    if not email:
        raise ValidationError("No verified email found in GitHub account")

    github_id = str(user_info.get("id", ""))
    name = user_info.get("name") or user_info.get("login", "")

    if not github_id:
        raise ValidationError("Invalid user ID from GitHub")

    g.oauth_email = email
    g.oauth_provider = "github"
    g.provider_user_id = github_id
    g.provider_name = name

    user_id: str
    is_new_user: bool
    user_id, is_new_user = process_oauth_user()

    user = g.oauth_user

    tokens: dict[str,
                 Any] = generate_tokens(
                     user_id,
                     {
                         "username":
                         getattr(user,
                                 "username",
                                 None),
                    
```

---

## Security Review

### Security Review for `handle_github_auth_callback`

#### Vulnerabilities Found:

1. **Hardcoded Secrets**: The code retrieves OAuth credentials from environment variables, which is good practice. However, ensure these are not accidentally committed to version control.
   - **Severity**: Low

2. **Input Validation Gaps**: There are no checks on the `code` parameter or other inputs, which could lead to unexpected behavior if manipulated.
   - **Severity**: Medium

3. **Error Handling Leaks Information**: The code raises `ValidationError` with generic error messages that might leak information about the system's internal state.
   - **Severity**: Low

4. **Insecure Deserialization**: No deserialization is present, but ensure no untrusted data is passed to functions like `generate_tokens`.
   - **Severity**: Info

5. **Race Conditions and TOCTOU Bugs**: Potential race conditions if multiple users are authenticated simultaneously. Ensure proper handling of concurrent requests.
   - **Severity**: Medium

#### Attack Vectors:

- An attacker could manipulate the `code` parameter or other inputs to cause unexpected behavior.
- Generic error messages might provide clues about system state.

#### Recommended Fixes:

1. **Input Validation**:
   ```python
   if not isinstance(code, str) or len(code.strip()) == 0:
       raise ValidationError("Invalid authorization code")
   ```

2. **Error Handling**:
   - Use more specific error messages and log detailed errors for debugging purposes only.

3. **Race Conditions**:
   - Ensure thread safety by using locks or other synchronization mechanisms when processing concurrent requests.

4. **Environment Variable Checks**:
   - Double-check that environment variables are set correctly before use.

#### Overall Security Posture:

The code is generally secure but could benefit from improved input validation and error handling. Properly securing the OAuth flow and ensuring no sensitive information is leaked through errors will further enhance security.

---

*Generated by CodeWorm on 2026-02-19 18:57*
