# handle_github_mobile_callback

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/api/domains/account/controllers/oauth/github_mobile_ctrl.py
**Language:** python
**Lines:** 21-171
**Complexity:** 19.0

---

## Source Code

```python
def handle_github_mobile_callback() -> str:
    """
    Handle GitHub OAuth callback for mobile - redirects to app scheme
    """
    code: str | None = request.args.get("code")

    if not code:
        return "certgames://oauth/error?message=missing_code"

    github_client_id = os.getenv("GH_MOBILE_CLIENT_ID")
    github_client_secret = os.getenv("GH_MOBILE_CLIENT_SECRET")

    if not github_client_id or not github_client_secret:
        return "certgames://oauth/error?message=config_error"

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
        return "certgames://oauth/error?message=token_exchange_failed"

    token_data: dict[str, Any] = token_response.json()
    access_token: str | None = token_data.get("access_token")

    if not access_token:
        return "certgames://oauth/error?message=no_access_token"

    user_response: requests.Response = requests.get(
        "https://api.github.com/user",
        headers = {"Authorization": f"Bearer {access_token}"},
        timeout = 60,
    )

    if user_response.status_code != 200:
        return "certgames://oauth/error?message=user_fetch_failed"

    user_info: dict[str, Any] = user_response.json()

    email_response: requests.Response = requests.get(
        "https://api.github.com/user/emails",
        headers = {"Authorization": f"Bearer {access_token}"},
        timeout = 60,
    )

    if email_response.status_code != 200:
        return "certgames://oauth/error?message=email_fetch_failed"

    emails = email_response.json()
    if not isinstance(emails, list):
        return "certgames://oauth/error?message=invalid_email_response"

    email = next(
        (
            e['email'] for e in emails if isinstance(e, dict)
            and e.get('primary') and e.get('verified')
        ),
        None
    )

    if not email:
        return "certgames://oauth/error?message=no_verified_email"

    github_id = str(user_info.get("id", ""))
    name = user_info.get("name") or user_info.get("login", "")

    if not github_id:
        return "certgames://oauth/error?message=invalid_github_id"

    g.oauth_email = email
    g.oauth_provider = "github"
    g.provider_user_id = github_id
    g.provider_name = name

    user_id: str
    is_new_user: bool
    user_id, is_new_user = process_oauth_user()

    user: User = g.oauth_user

    tokens: dict[str,
                 Any] = generate_tokens(
                     user_id,
                     {
                         "username":
                         getattr(user,
                                 "username",
                                 None),
                         "email":
                         email,
 
```

---

## Security Review

### Security Review for `handle_github_mobile_callback`

#### Vulnerabilities Found:

1. **Hardcoded Secrets or Credentials**:
   - **Severity: Critical**
     - The client ID and secret are hardcoded in the environment variables, which is a security risk if these secrets are exposed.
   - **Fix**: Use secure vaults like AWS Secrets Manager, HashiCorp Vault, or similar to store sensitive information.

2. **Input Validation Gaps**:
   - **Severity: Medium**
     - The code does not validate the `code` parameter from the OAuth callback, which could lead to injection attacks.
   - **Fix**: Validate and sanitize the `code` parameter before using it in requests.

3. **Error Handling that Leaks Information**:
   - **Severity: Low**
     - The error messages returned are generic and do not provide detailed information about what went wrong.
   - **Fix**: Return more generic error messages to avoid leaking sensitive information.

4. **Insecure Deserialization**:
   - **Severity: Info**
     - No deserialization is present in the code, so this issue does not apply here.

5. **Race Conditions and TOCTOU Bugs**:
   - **Severity: Low**
     - The code does not seem to have race conditions or TOCTOU bugs based on the provided snippet.
   - **Fix**: Ensure that all database operations are properly synchronized if they interact with shared resources.

#### Attack Vectors:

- **Injection Attacks**: If an attacker can manipulate the `code` parameter, they could potentially inject malicious data into the requests.
- **Information Leakage**: Generic error messages could be exploited to gather information about the application's internal state.

#### Recommended Fixes:

1. **Hardcoded Secrets**:
   - Move client ID and secret to a secure vault.
2. **Input Validation**:
   - Validate `code` parameter to ensure it is properly formatted.
3. **Error Handling**:
   - Return generic error messages that do not reveal sensitive information.

#### Overall Security Posture:

The overall security posture of the code can be improved by addressing the hardcoded secrets and input validation issues. The current implementation has some potential vulnerabilities, but with proper fixes, it can become more secure.

---

*Generated by CodeWorm on 2026-02-19 13:11*
