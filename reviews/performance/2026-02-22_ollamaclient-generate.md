# OllamaClient.generate

**Type:** Performance Analysis
**Repository:** CodeWorm
**File:** codeworm/llm/client.py
**Language:** python
**Lines:** 147-212
**Complexity:** 11.0

---

## Source Code

```python
async def generate(
        self,
        prompt: str,
        system: str | None = None,
        temperature: float | None = None,
        max_tokens: int | None = None,
    ) -> GenerationResult:
        """
        Generate text from a prompt
        """
        client = await self._get_client()

        options = {
            "temperature": temperature or self.settings.temperature,
            "num_predict": max_tokens or self.settings.num_predict,
            "num_ctx": self.settings.num_ctx,
        }

        payload = {
            "model": self.settings.model,
            "prompt": prompt,
            "stream": False,
            "keep_alive": self.settings.keep_alive,
            "options": options,
        }

        if system:
            payload["system"] = system

        try:
            response = await client.post("/api/generate", json = payload)

            if response.status_code != 200:
                error_text = response.text
                if "out of memory" in error_text.lower(
                ) or "cuda" in error_text.lower():
                    raise OllamaModelError(f"Model OOM: {error_text}")
                raise OllamaError(f"Generation failed: {error_text}")

            data = response.json()

            prompt_tokens = data.get("prompt_eval_count", 0)
            completion_tokens = data.get("eval_count", 0)
            total_duration = data.get("total_duration", 0) / 1_000_000

            tokens_per_sec = 0.0
            if total_duration > 0 and completion_tokens > 0:
                tokens_per_sec = completion_tokens / (total_duration / 1000)

            return GenerationResult(
                text = data.get("response",
                                ""),
                model = data.get("model",
                                 self.settings.model),
                prompt_tokens = prompt_tokens,
                completion_tokens = completion_tokens,
                total_duration_ms = int(total_duration),
                tokens_per_second = tokens_per_sec,
            )

        except httpx.ConnectError as e:
            raise OllamaConnectionError(
                f"Cannot connect to Ollama at {self.base_url}: {e}"
            ) from e
        except httpx.TimeoutException as e:
            raise OllamaTimeoutError(f"Request timed out: {e}") from e
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity is primarily determined by the network request, which has a worst-case of \(O(1)\) for the `post` call but can be delayed due to network issues.

#### Space Complexity and Memory Allocation
- **Memory Usage**: The function uses dictionaries (`options`, `payload`) and performs string operations. The space complexity is \(O(n)\), where \(n\) is the length of the prompt and system inputs.
  
#### Bottlenecks or Inefficiencies
1. **Redundant Operations**: The default values for `temperature` and `max_tokens` are fetched from settings, which might be redundant if they are always set.
2. **Error Handling**: Multiple error handling blocks can lead to performance overhead due to repeated checks.

#### Optimization Opportunities
1. **Cache Settings Defaults**: Cache the default settings (`temperature`, `max_tokens`) to avoid repeated lookups.
2. **Simplify Error Handling**: Combine similar exceptions into a single block for cleaner and more efficient code.

```python
async def generate(
        self,
        prompt: str,
        system: str | None = None,
        temperature: float | None = None,
        max_tokens: int | None = None,
    ) -> GenerationResult:
    client = await self._get_client()
    
    defaults = {
        "temperature": temperature or self.settings.temperature,
        "num_predict": max_tokens or self.settings.num_predict,
        "num_ctx": self.settings.num_ctx,
    }
    
    payload = {
        "model": self.settings.model,
        "prompt": prompt,
        "stream": False,
        "keep_alive": self.settings.keep_alive,
        "options": defaults,
    }

    if system:
        payload["system"] = system

    try:
        response = await client.post("/api/generate", json=payload)
        
        if response.status_code != 200:
            error_text = response.text
            if "out of memory" in error_text.lower() or "cuda" in error_text.lower():
                raise OllamaModelError(f"Model OOM: {error_text}")
            else:
                raise OllamaError(f"Generation failed: {error_text}")

        data = response.json()

        prompt_tokens = data.get("prompt_eval_count", 0)
        completion_tokens = data.get("eval_count", 0)
        total_duration = data.get("total_duration", 0) / 1_000_000

        tokens_per_sec = 0.0
        if total_duration > 0 and completion_tokens > 0:
            tokens_per_sec = completion_tokens / (total_duration / 1000)

        return GenerationResult(
            text=data.get("response", ""),
            model=data.get("model", self.settings.model),
            prompt_tokens=prompt_tokens,
            completion_tokens=completion_tokens,
            total_duration_ms=int(total_duration),
            tokens_per_second=tokens_per_sec,
        )

    except (httpx.ConnectError, httpx.TimeoutException) as e:
        raise OllamaConnectionError(
            f"Cannot connect to Ollama at {self.base_url}: {e}"
        ) from e
```

#### Resource Usage Concerns
- Ensure `client` is properly closed after use by wrapping it in a context manager if necessary.
- Consider using connection pooling for the HTTP client to manage resources efficiently.

---

*Generated by CodeWorm on 2026-02-22 01:01*
