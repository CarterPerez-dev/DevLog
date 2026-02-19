# EmbeddingService

**Type:** Class Documentation
**Repository:** vuemantics
**File:** backend/services/ai/embedding.py
**Language:** python
**Lines:** 27-146
**Complexity:** 0.0

---

## Source Code

```python
class EmbeddingService:
    """
    Handles embedding generation using bge-m3
    """
    def __init__(
        self,
        ollama: OllamaManager,
        semaphore: asyncio.Semaphore
    ):
        """
        Initialize embedding service

        Args:
            ollama: OllamaManager instance
            semaphore: Semaphore for concurrency control
        """
        self._ollama = ollama
        self._semaphore = semaphore

    @retry(
        retry = retry_if_exception_type(
            (httpx.TimeoutException,
             httpx.ConnectError)
        ),
        stop = stop_after_attempt(3),
        wait = wait_exponential(multiplier = 1,
                                min = 2,
                                max = 30),
    )
    async def generate_embedding(self, text: str) -> list[float]:
        """
        Generate embedding vector from text using bge-m3 via Ollama

        Args:
            text: Text to embed (vision description or search query)

        Returns:
            1024-dimensional embedding vector
        """
        try:
            if len(text) > config.MAX_EMBEDDING_TEXT_LENGTH:
                text = text[: config.MAX_EMBEDDING_TEXT_LENGTH] + "..."
                logger.warning(
                    f"Truncated text from {len(text)} to "
                    f"{config.MAX_EMBEDDING_TEXT_LENGTH} chars"
                )

            async with self._semaphore:
                client = await self._ollama.get_client()
                response = await client.embeddings(
                    model = config.settings.local_embedding_model,
                    prompt = text,
                )

                embedding = response["embedding"]

                logger.debug(
                    f"Embedding type: {type(embedding).__name__}, "
                    f"length: {len(embedding)}"
                )

                if len(embedding
                       ) != config.settings.local_embedding_dimensions:
                    raise EmbeddingError(
                        f"Invalid embedding dimensions: {len(embedding)} "
                        f"(expected {config.settings.local_embedding_dimensions})"
                    )

                return embedding  # type: ignore[no-any-return]

        except ResponseError as e:
            if e.status_code == status.HTTP_404_NOT_FOUND:
                raise EmbeddingError(
                    f"Model not loaded. Run: ollama pull "
                    f"{config.settings.local_embedding_model}"
                ) from e
            raise EmbeddingError(
                f"Embedding model error: {e.error}"
            ) from e
        except Exception as e:
            logger.error(f"Embedding generation failed: {e}")
            raise EmbeddingError(
                f"Failed to generate embedding: {e!s}"
            ) from e

    async def create_query_embedding(self, query: str) -> list[float]:
        """
        Generate embedding for search query.

        Args:
            quer
```

---

## Class Documentation

### EmbeddingService Documentation

**Class Responsibility and Purpose:**
The `EmbeddingService` class is responsible for generating embeddings using the bge-m3 model via an OllamaManager instance. It handles concurrency control, error handling, and batch processing of text inputs to ensure efficient and robust embedding generation.

**Public Interface (Key Methods):**
- **`generate_embedding(text: str) -> list[float]`:** Generates a 1024-dimensional embedding vector from the provided text.
- **`create_query_embedding(query: str) -> list[float]`:** A convenience method to generate an embedding for a search query, delegating to `generate_embedding`.
- **`batch_generate(texts: list[str], max_concurrent: int = config.BATCH_EMBEDDING_MAX_CONCURRENT) -> list[list[float]]`:** Generates embeddings for multiple texts concurrently, limiting the number of concurrent operations.

**Design Patterns Used:**
- **Retry Decorator:** Utilizes the `retry` decorator to handle transient errors and retry failed requests up to three times with exponential backoff.
- **Semaphore:** Manages concurrency by controlling the maximum number of simultaneous operations using an asyncio Semaphore.

**How it Fits in the Architecture:**
The `EmbeddingService` class is a crucial component within the AI services layer, providing embedding generation capabilities for various text inputs. It integrates seamlessly with the OllamaManager to leverage external model resources and ensures that embedding requests are handled efficiently and reliably. This class fits into the broader architecture by offering a consistent interface for generating embeddings, which can be used across different parts of the application, such as search queries or content analysis.

---

*Generated by CodeWorm on 2026-02-19 02:49*
