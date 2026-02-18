# RefreshTokenService

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/services/refresh_token_service.py
**Language:** python
**Lines:** 13-226
**Complexity:** 0.0

---

## Source Code

```python
class RefreshTokenService:
    """
    Service for managing refresh tokens in Redis
    """
    @staticmethod
    def _get_redis() -> redis.StrictRedis | None:
        """
        Get Redis client from Flask app extensions
        """
        try:
            return current_app.extensions.get("redis_client")
        except Exception:
            return None

    @staticmethod
    def _get_token_key(user_id: str, jti: str) -> str:
        """
        Generate Redis key for refresh token
        """
        return f"refresh_token:{user_id}:{jti}"

    @staticmethod
    def _get_user_tokens_pattern(user_id: str) -> str:
        """
        Generate Redis pattern for all user's refresh tokens
        """
        return f"refresh_token:{user_id}:*"

    @staticmethod
    def store_refresh_token(
        user_id: str,
        token: str,
        expires_in: int
    ) -> bool:
        """
        Store refresh token in Redis with TTL
        """
        redis_client = RefreshTokenService._get_redis()
        if not redis_client:
            current_app.logger.warning(
                "Redis not available for refresh token storage"
            )
            return False

        try:
            decoded = decode_token(token)
            jti = decoded.get("jti")

            if not jti:
                current_app.logger.error("Refresh token missing JTI")
                return False

            token_data = {
                "user_id":
                user_id,
                "jti":
                jti,
                "token":
                token,
                "created_at":
                datetime.now(UTC).isoformat(),
                "expires_at":
                datetime.fromtimestamp(decoded.get("exp",
                                                   0),
                                       UTC).isoformat()
            }

            key = RefreshTokenService._get_token_key(user_id, jti)

            redis_client.setex(
                key,
                expires_in + 60,
                json.dumps(token_data)
            )

            current_app.logger.info(
                f"Stored refresh token for user {user_id[:8]}..."
            )
            return True

        except Exception as e:
            current_app.logger.error(f"Failed to store refresh token: {e}")
            return False

    @staticmethod
    def get_user_refresh_tokens(user_id: str) -> list[dict]:
        """
        Get all refresh tokens for a user
        """
        redis_client = RefreshTokenService._get_redis()
        if not redis_client:
            return []

        try:
            pattern = RefreshTokenService._get_user_tokens_pattern(user_id)
            keys = redis_client.keys(pattern)

            tokens = []
            for key in keys:
                token_data = redis_client.get(key)
                if token_data:
                    try:
                        tokens.append(json.loads(token_data))
                    except json.JSONDecodeErr
```

---

## Class Documentation

### RefreshTokenService Documentation

**Class Responsibility and Purpose:**
The `RefreshTokenService` class is responsible for managing refresh tokens stored in Redis, including storing, retrieving, revoking, and cleaning up expired tokens. This service ensures secure token management by leveraging Redis for efficient storage and retrieval.

**Public Interface (Key Methods):**
- **store_refresh_token(user_id: str, token: str, expires_in: int) -> bool**: Stores a refresh token in Redis with an expiration time.
- **get_user_refresh_tokens(user_id: str) -> list[dict]**: Retrieves all refresh tokens associated with a user.
- **revoke_refresh_token(user_id: str, jti: str) -> bool**: Revokes a specific refresh token by its unique identifier (JTI).
- **revoke_all_user_tokens(user_id: str) -> int**: Revokes all refresh tokens for a given user.
- **cleanup_expired_tokens() -> int**: Manually cleans up expired tokens to handle edge cases.

**Design Patterns Used:**
The class does not explicitly use any design patterns like Factory, Observer, or Strategy. However, it follows the Singleton pattern implicitly by using static methods and leveraging Flask's application context for Redis client access.

**Relationship to Other Classes:**
This service interacts with `current_app` (Flask app) to get the Redis client and logs events through `current_app.logger`. It also uses JSON serialization/deserialization via `json.dumps` and `json.loads`.

**State Management Approach:**
The class manages state by storing token data in Redis, including user ID, JTI, token value, creation time, and expiration time. The tokens are stored with a TTL (time-to-live) to ensure they expire automatically.

This service fits into the architecture as a key component for secure authentication and authorization, ensuring that refresh tokens are managed efficiently and securely within the application.

---

*Generated by CodeWorm on 2026-02-18 12:48*
