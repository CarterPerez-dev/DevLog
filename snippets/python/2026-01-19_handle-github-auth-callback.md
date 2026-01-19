# handle_github_auth_callback

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
                         "email":
                         email,
                         "oauth_provider":
                         "github",
                         "subscriptionActive":
                         getattr(user,
                                 "subscriptionActive",
                                 False),
                         "subscriptionType":
                         getattr(user,
                                 "subscriptionType",
                                 "free"),
                     },
                 )

    session["userId"] = user_id

    frontend_url: str = os.getenv("FRONTEND_URL", "https://certgames.com")

    redirect_params: dict[str,
                          str] = {
                              "provider": "github",
                              "userId": user_id,
                              "access_token": tokens["access_token"],
                              "refresh_token": tokens["refresh_token"],
                              "expires_in": str(tokens["expires_in"]),
                          }

    if is_new_user:
        redirect_params["needs_username"] = "true"
        redirect_base = f"{frontend_url}/oauth/success"
    else:
        redirect_params["needs_username"] = str(
            getattr(user,
                    "needs_username",
                    False)
        ).lower()
        redirect_base = f"{frontend_url}/oauth/success"

    query_string: str = "&".join(
        [f"{k}={v}" for k, v in redirect_params.items()]
    )
    redirect_url: str = f"{redirect_base}?{query_string}"

    return redirect_url
```

---

## Documentation

### Documentation for `handle_github_auth_callback`

**Purpose and Behavior:**
This function handles the GitHub OAuth callback, authenticating users by exchanging an authorization code for an access token. It then retrieves user information and emails, processes the user data, generates tokens, and redirects to the frontend with necessary parameters.

**Key Implementation Details:**
- **Error Handling:** The function raises `ValidationError` if any required credentials are missing or if API responses indicate failure.
- **API Calls:** Multiple HTTP requests are made using the `requests` library to interact with GitHub's OAuth endpoints.
- **Context Management:** Utilizes context variables like `g.oauth_email`, `g.provider_user_id`, and `g.provider_name` for storing intermediate data.

**When/Why to Use:**
This function is crucial for integrating GitHub OAuth authentication in a web application. It ensures secure user authentication, retrieves essential user information, and handles the redirection process seamlessly.

**Patterns and Gotchas:**
- **Error Propagation:** The function consistently raises `ValidationError` upon encountering issues, making debugging straightforward.
- **HTTP Requests:** Multiple HTTP requests are made without retries or backoff strategies, which could be improved for robustness in production environments.
- **Type Hints:** While type hints are used, the function relies heavily on runtime checks and assertions to handle dynamic data structures.

---

*Generated by CodeWorm on 2026-01-19 14:15*
