# CacheService

**Type:** Class Documentation
**Repository:** fastapi-rc
**File:** fastapi-rc/fastapi_rc/service.py
**Language:** python
**Lines:** 30-242
**Complexity:** 0.0

---

## Source Code

```python
class CacheService(Generic[T]):
    """
    Generic caching service for Pydantic models

    Provides cache-aside pattern with automatic serialization

    Usage:
        user_cache = CacheService(
            redis_client,
            namespace="users",
            model=User,
            default_ttl=600
        )
        user = await user_cache.get("123")
        await user_cache.set("123", user_obj, ttl=600)
    """
    def __init__(
        self,
        redis: Redis,
        namespace: str,
        model: type[T] | None = None,
        default_ttl: int = 300,
        use_jitter: bool = True,
        prefix: str = "cache",
        version: str = "v1",
    ):
        self.redis = redis
        self.namespace = namespace
        self.model = model
        self.default_ttl = default_ttl
        self.use_jitter = use_jitter
        self.prefix = prefix
        self.version = version

    def _build_key(
        self,
        identifier: str,
        params: dict[str, Any] | None = None
    ) -> str:
        """
        Build namespaced cache key
        """
        return build_cache_key(
            self.namespace,
            identifier,
            params,
            prefix = self.prefix,
            version = self.version,
        )

    def _get_ttl(self, ttl: int | None = None) -> int:
        """
        Get TTL with optional jitter
        """
        effective_ttl = ttl or self.default_ttl
        if self.use_jitter:
            return get_ttl_with_jitter(effective_ttl)
        return effective_ttl

    async def get(
        self,
        identifier: str,
        params: dict[str, Any] | None = None,
    ) -> T | None:
        """
        Get cached value, deserialize to Pydantic model if configured
        """
        try:
            key = self._build_key(identifier, params)
            data = await self.redis.get(key)

            if data is None:
                return None

            if self.model:
                return self.model.model_validate_json(data)

            return json.loads(data) if isinstance(data, str) else data

        except Exception as e:
            logger.warning(f"Cache get failed for {identifier}: {e}")
            return None

    async def set(
        self,
        identifier: str,
        value: T | dict[str, Any] | str,
        ttl: int | None = None,
        params: dict[str, Any] | None = None,
    ) -> bool:
        """
        Set cached value with automatic serialization
        """
        try:
            key = self._build_key(identifier, params)
            effective_ttl = self._get_ttl(ttl)

            if isinstance(value, BaseModel):
                data = value.model_dump_json()
            elif isinstance(value, dict):
                data = json.dumps(value)
            else:
                data = str(value)

            await self.redis.set(key, data, ex = effective_ttl)
            return True

        except Exception as e:
            logger.error(f"Cache set failed for {identifie
```

---

## Class Documentation

### CacheService Documentation

**Class Responsibility and Purpose:**
The `CacheService` class provides a generic caching mechanism for Pydantic models using Redis, implementing the cache-aside pattern with automatic serialization/deserialization. It ensures that cached values are automatically serialized to JSON when stored and deserialized back into Pydantic models when retrieved.

**Public Interface (Key Methods):**
- **Initialization (`__init__`)**: Sets up the cache service with a Redis client, namespace, model type, default TTL, jitter usage, prefix, and version.
- **Get (`get`)**: Retrieves a cached value, deserializing it to a Pydantic model if configured.
- **Set (`set`)**: Stores a value in the cache, automatically serializing it based on its type.
- **Delete (`delete`)**: Removes a cached value by key.
- **Exists (`exists`)**: Checks if a key exists in the cache.
- **Get or Set (`get_or_set`)**: Implements the cache-aside pattern, fetching from cache or executing a factory function to populate it.
- **Invalidate Pattern (`invalidate_pattern`)**: Invalidates all keys matching a given pattern within the namespace.

**Design Patterns Used:**
- **Factory Method**: `get_or_set` uses a factory method to lazily load data into the cache.
- **Strategy**: The class supports different serialization strategies based on the value type (Pydantic model, dictionary, or string).

**Relationship to Other Classes:**
The `CacheService` integrates with Redis for caching operations and works in conjunction with Pydantic models. It is typically used within services or repositories where data retrieval and storage need optimization.

This class fits into a broader architecture by providing efficient, scalable caching mechanisms that can significantly enhance application performance without impacting the core business logic.

---

*Generated by CodeWorm on 2026-02-18 18:53*
