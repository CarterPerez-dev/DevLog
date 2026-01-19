# handle_github_mobile_callback

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

    # Check if user made subscription choice
    # If not new user, they've been through the flow already
    subscription_choice_made = getattr(
        user,
        "subscription_choice_made",
        False
    )
    has_active_subscription = getattr(user, "subscriptionActive", False)
    has_made_choice = subscription_choice_made or has_active_subscription or (
        not is_new_user
    )

    redirect_params: dict[str,
                          str] = {
                              "provider": "github",
                              "userId": user_id,
                              "access_token": tokens["access_token"],
                              "refresh_token": tokens["refresh_token"],
                              "expires_in": str(tokens["expires_in"]),
                              "needs_username": str(
                                  getattr(user,
                                          "needs_username",
                                          False)
                              ).lower(),
                              "is_new_user": str(is_new_user).lower(),
                              "hasSubscription":
                              str(has_made_choice).lower(),
                              "subscriptionType":
                              getattr(user,
                                      "subscriptionType",
                                      "free"),
                          }

    query_string: str = "&".join(
        [f"{k}={v}" for k, v in redirect_params.items()]
    )
    redirect_url: str = f"certgames://oauth/success?{query_string}"

    return redirect_url
```

---

## Documentation

### Documentation for `handle_github_mobile_callback`

**Purpose and Behavior:**
This function handles the OAuth callback from GitHub's mobile app, exchanging an authorization code for tokens and fetching user information. It then processes the user data to create or update a user record in the system.

**Key Implementation Details:**
- **Error Handling:** The function returns specific URLs with error messages if any step fails (e.g., missing code, token exchange failure).
- **OAuth Token Exchange:** Uses `requests` to send POST and GET requests to GitHub's OAuth API.
- **User Data Processing:** Fetches user information and emails, ensuring the email is primary and verified.
- **Session Management:** Sets session variables for user ID and tokens.
- **Redirect URL Construction:** Constructs a redirect URL with necessary parameters for further processing.

**When/Why to Use:**
This function should be used when integrating mobile apps with GitHub's OAuth system. It ensures secure token exchange, proper user data handling, and seamless integration into the application flow.

**Patterns and Gotchas:**
- **Error Handling:** The function uses multiple return statements to handle errors gracefully, ensuring a smooth user experience.
- **Type Hints:** Utilizes type hints for better code readability and maintainability.
- **Context Variables (`g`):** Uses context variables to store temporary data, which may be specific to the application's architecture. Ensure these are properly managed to avoid conflicts.
- **Subscription Logic:** The function checks if a new user has made a subscription choice, ensuring that users who have already gone through the flow do not repeat steps.

---

*Generated by CodeWorm on 2026-01-18 23:57*
