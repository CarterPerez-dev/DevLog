# handle_github_mobile_callback

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of `handle_github_mobile_callback` is primarily determined by the HTTP requests made to GitHub's API, which are O(1) for each request but can be considered as a series of constant-time operations. The overall complexity is dominated by the network latency and the number of requests (4 in this case).

#### Space Complexity
The space complexity is low since no significant data structures are used that grow with input size. However, the code uses several temporary variables and dictionaries, which could be optimized to reduce memory footprint.

#### Bottlenecks or Inefficiencies
1. **Redundant HTTP Requests**: Four separate requests are made to GitHub's API, which can be costly in terms of network latency.
2. **Error Handling**: Each request has a similar error handling pattern, leading to redundant code.
3. **Multiple Context Manager Usage**: The use of context managers and session management could be optimized for better resource usage.

#### Optimization Opportunities
1. **Batch Requests**: Combine the user and email requests into a single API call if possible, reducing the number of HTTP requests.
2. **Error Handling Refactoring**: Extract common error handling logic to reduce redundancy.
3. **Context Manager Usage**: Ensure that all resources (e.g., database connections) are properly managed using context managers.

#### Resource Usage Concerns
1. **Unclosed Connections**: Ensure that any open connections or file handles are closed properly, especially in async contexts if applicable.
2. **Session Management**: Use session management libraries to handle sessions more efficiently and reduce the risk of resource leaks.

By addressing these issues, you can improve both the performance and maintainability of the code.

---

*Generated by CodeWorm on 2026-02-20 11:37*
