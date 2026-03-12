# log_ai_request

**Type:** Performance Analysis
**Repository:** CertGames-Core
**File:** backend/api/core/limiters/throttle_helpers.py
**Language:** python
**Lines:** 247-304
**Complexity:** 9.0

---

## Source Code

```python
def log_ai_request(
    user_id: str | None,
    operation_type: str,
    estimated_tokens: int,
    model: str = "gpt-5-nano",
    *,
    input_tokens: int | None = None,
    output_tokens: int | None = None,
    processing_time: float | None = None,
    error_message: str | None = None,
    request_size: int | None = None,
    response_size: int | None = None,
) -> str | None:
    """
    Log AI requests for monitoring and analysis.
    Returns the log ID for updating later.
    """
    try:
        ip_address: str = request.remote_addr or "unknown"
        user_agent: str = request.headers.get("User-Agent", "")[: 200]

        total_tokens = (
            input_tokens + output_tokens
        ) if input_tokens and output_tokens else estimated_tokens

        estimated_cost = calculate_ai_cost(
            model,
            input_tokens,
            output_tokens
        ) if input_tokens and output_tokens else None

        log_entry = AIUsageLog(
            userId = user_id,
            ipAddress = ip_address,
            userAgent = user_agent,
            operationType = operation_type,
            estimatedTokens = estimated_tokens,
            model = model,
            timestamp = datetime.now(UTC),
            expiresAt = datetime.now(UTC) + timedelta(days = 30),
            input_tokens = input_tokens or 0,
            output_tokens = output_tokens or 0,
            total_tokens = total_tokens,
            estimated_cost = estimated_cost,
            processing_time = processing_time,
            error_message = error_message,
            request_size = request_size,
            response_size = response_size,
            service_provider = "openai",
            model_name = model,
            endpoint = operation_type,
        )
        log_entry.save()
        return str(log_entry.id)

    except Exception as e:
        logger.error("Error logging AI request: %s", str(e))
        return None
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity:** The function has a time complexity of \(O(1)\) for the core operations, as it performs a fixed number of operations regardless of input size. However, the `calculate_ai_cost` call and database save operation could be costly.

**Space Complexity:** The space complexity is also \(O(1)\), assuming that the log entry object creation and saving do not significantly increase memory usage.

**Bottlenecks or Inefficiencies:**
- **Redundant Operations:** The `input_tokens` and `output_tokens` are checked multiple times.
- **Unnecessary String Slicing:** The `user_agent[:200]` could be moved outside the try block to avoid redundant slicing if it's always needed.

**Optimization Opportunities:**
1. **Combine Redundant Checks:** Move common checks like `input_tokens and output_tokens` into a single variable.
2. **Move String Slicing Outside Try Block:** If `user_agent[:200]` is always required, move the slicing operation outside the try block.

**Resource Usage Concerns:**
- Ensure that database connections are properly managed using context managers or connection pooling to avoid resource leaks.
- Consider caching frequent calls to `calculate_ai_cost` if it's an expensive function.

### Suggested Optimizations

```python
def log_ai_request(
    user_id: str | None,
    operation_type: str,
    estimated_tokens: int,
    model: str = "gpt-5-nano",
    *,
    input_tokens: int | None = None,
    output_tokens: int | None = None,
    processing_time: float | None = None,
    error_message: str | None = None,
    request_size: int | None = None,
    response_size: int | None = None,
) -> str | None:
    try:
        ip_address = request.remote_addr or "unknown"
        user_agent = (request.headers.get("User-Agent", "")[:200] if input_tokens and output_tokens else "")
        
        total_tokens = (
            input_tokens + output_tokens
        ) if input_tokens and output_tokens else estimated_tokens

        log_entry = AIUsageLog(
            userId=user_id,
            ipAddress=ip_address,
            userAgent=user_agent,
            operationType=operation_type,
            estimatedTokens=estimated_tokens,
            model=model,
            timestamp=datetime.now(UTC),
            expiresAt=datetime.now(UTC) + timedelta(days=30),
            input_tokens=input_tokens or 0,
            output_tokens=output_tokens or 0,
            total_tokens=total_tokens,
            estimated_cost=(calculate_ai_cost(model, input_tokens, output_tokens) if input_tokens and output_tokens else None),
            processing_time=processing_time,
            error_message=error_message,
            request_size=request_size,
            response_size=response_size,
            service_provider="openai",
            model_name=model,
            endpoint=operation_type,
        )
        log_entry.save()
        return str(log_entry.id)

    except Exception as e:
        logger.error("Error logging AI request: %s", str(e))
        return None
```

This refactoring reduces redundancy and improves readability. Ensure database connections are managed properly to avoid leaks.

---

*Generated by CodeWorm on 2026-03-12 07:13*
