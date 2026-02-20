# handle_github_auth_callback

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

**Time Complexity**: The function has a time complexity of \(O(n)\), where \(n\) is the number of GitHub API calls (in this case, 3). Each API call involves network latency and processing time.

**Space Complexity**: The space complexity is \(O(1)\) as the function uses a fixed amount of additional memory for variables like `code`, `github_client_id`, etc. However, the response data from the GitHub APIs are stored in dictionaries, which can grow depending on the size of the JSON responses.

### Bottlenecks and Inefficiencies

1. **Redundant API Calls**: Three separate API calls to GitHub are made: one for the access token, another for user information, and a third for email verification. This could be optimized by making a single request that fetches all necessary data.
2. **Error Handling**: The function raises `ValidationError` multiple times, which can lead to redundant error handling logic. Consider centralizing error handling or using a more robust validation mechanism.
3. **Blocking Calls**: All API calls are blocking, which can introduce significant latency if the network is slow.

### Optimization Opportunities

1. **Combine API Requests**: Use the GitHub API to fetch user information and emails in a single request by including the `affiliations` parameter or using the `/user/emails` endpoint with proper headers.
2. **Asynchronous Calls**: Convert blocking calls into asynchronous requests using libraries like `aiohttp`. This can significantly reduce latency, especially if network conditions are poor.
3. **Centralize Error Handling**: Use a single error handling mechanism to catch and process all exceptions, reducing redundancy.

### Resource Usage Concerns

- **Network Latency**: The function is highly dependent on network performance, which could be improved by making fewer requests or using caching strategies.
- **Environment Variables**: Ensure that environment variables like `GH_CLIENT_ID` and `GH_CLIENT_SECRET` are securely managed to prevent exposure.

By addressing these areas, the function can become more efficient and robust.

---

*Generated by CodeWorm on 2026-02-20 15:07*
