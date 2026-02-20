# validate_oauth_state_match

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/api/domains/account/middleware/rules.py
**Language:** python
**Lines:** 103-187
**Complexity:** 27.0

---

## Source Code

```python
def validate_oauth_state_match(provider = None):
    """
    Validate OAuth state parameter for CSRF protection.
    """
    request_state = g.validated.get('state')

    if not request_state:
        raise ValidationError("State parameter is required")

    if provider == 'apple' or g.get('oauth_provider') == 'apple':
        valid_states = session.get('apple_valid_states', [])

        if not valid_states:
            raise AuthenticationError("No valid Apple states in session")

        current_time = datetime.now(UTC).timestamp()
        state_found = False

        if isinstance(valid_states[0], tuple | list):
            for state, timestamp in valid_states:
                if state == request_state:
                    if current_time - timestamp > 600:
                        raise AuthenticationError(
                            "Apple OAuth state expired (10 min limit)"
                        )
                    state_found = True
                    break

            if not state_found:
                raise AuthenticationError("Invalid Apple OAuth state")

            valid_states = [
                (s,
                 t) for s, t in valid_states if s != request_state
            ]
        else:
            if request_state not in valid_states:
                raise AuthenticationError("Invalid Apple OAuth state")
            valid_states.remove(request_state)

        session['apple_valid_states'] = valid_states
    elif provider == 'google' or request.endpoint and 'google' in request.endpoint:
        valid_states = session.get('google_valid_states', [])

        if not valid_states:
            raise AuthenticationError("No valid Google states in session")

        current_time = datetime.now(UTC).timestamp()
        state_found = False

        if isinstance(valid_states[0], tuple | list):
            for state, timestamp in valid_states:
                if state == request_state:
                    if current_time - timestamp > 600:
                        raise AuthenticationError(
                            "Google OAuth state expired (10 min limit)"
                        )
                    state_found = True
                    break

            if not state_found:
                raise AuthenticationError("Invalid Google OAuth state")

            valid_states = [
                (s,
                 t) for s, t in valid_states if s != request_state
            ]
        else:
            if request_state not in valid_states:
                raise AuthenticationError("Invalid Google OAuth state")
            valid_states.remove(request_state)

        session['google_valid_states'] = valid_states
    else:
        session_state = session.get('oauth_state')

        if not session_state:
            raise AuthenticationError("No OAuth state in session")

        if request_state != session_state:
            raise AuthenticationError(
                "Invalid OAuth state - CSRF protection"
            )

        
```

---

## Security Review

### Security Review for `validate_oauth_state_match`

#### Vulnerabilities and Severity:

1. **Authentication and Authorization Issues** (High):
   - The function relies on session data (`g.validated`, `session`) without proper validation or sanitization.
   - Potential attack vector: An attacker could manipulate session state to bypass authentication checks.

2. **Input Validation Gaps** (Medium):
   - The code assumes that `valid_states` is a list of tuples or strings, but does not validate the input type.
   - Potential attack vector: Malformed data in `valid_states` could lead to unexpected behavior.

3. **Error Handling Leaks Information** (Low):
   - Specific error messages provide information about the state validation process, which could be exploited by attackers.

#### Recommended Fixes:

1. Ensure session data is validated and sanitized before use.
2. Add type hints and input validation for `valid_states`.
3. Use generic exception handling instead of specific exceptions to avoid leaking sensitive information.

```python
def validate_oauth_state_match(provider: str = None) -> None:
    """
    Validate OAuth state parameter for CSRF protection.
    """
    request_state = g.validated.get('state')

    if not request_state:
        raise ValidationError("State parameter is required")

    valid_states = session.get(f'{provider}_valid_states', [])

    if not isinstance(valid_states, list):
        raise ValueError("Invalid states data in session")

    current_time = datetime.now(UTC).timestamp()
    state_found = False

    for idx, (state, timestamp) in enumerate(valid_states):
        if state == request_state:
            if current_time - timestamp > 600:
                valid_states.pop(idx)
                raise AuthenticationError("OAuth state expired (10 min limit)")
            state_found = True
            break

    if not state_found:
        raise AuthenticationError(f"Invalid {provider} OAuth state")

    session[f'{provider}_valid_states'] = [s for s in valid_states if s[0] != request_state]
```

#### Overall Security Posture:

The code has a good foundation but needs improvements in handling and validating session data. Proper validation, type hints, and generic error handling will enhance the security posture.

---

*Generated by CodeWorm on 2026-02-19 19:40*
