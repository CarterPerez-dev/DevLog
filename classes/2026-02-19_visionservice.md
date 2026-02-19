# VisionService

**Type:** Class Documentation
**Repository:** vuemantics
**File:** backend/services/ai/vision.py
**Language:** python
**Lines:** 36-232
**Complexity:** 0.0

---

## Source Code

```python
class VisionService:
    """
    Handles vision analysis using Qwen2.5-VL
    """
    def __init__(
        self,
        ollama: OllamaManager,
        semaphore: asyncio.Semaphore
    ):
        """
        Initialize vision service

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
    async def analyze_image(self, image_path: Path) -> str:
        """
        Analyze image with Qwen2.5-VL via Ollama.

        Args:
            image_path: Path to image file

        Returns:
            Text description of image content
        """
        try:
            async with self._semaphore:
                # Read image
                async with aiofiles.open(image_path, "rb") as f:
                    image_data = await f.read()

                # Preprocess in executor (non-blocking)
                loop = asyncio.get_event_loop()
                preprocessed_data = await loop.run_in_executor(
                    None,
                    preprocess_image_for_vision_model,
                    image_data
                )

                image_b64 = base64.b64encode(preprocessed_data
                                             ).decode("utf-8")

                client = await self._ollama.get_client()
                response = await client.chat(
                    model = config.settings.vision_model,
                    messages = [
                        {
                            "role": "user",
                            "content": IMAGE_ANALYSIS_PROMPT,
                            "images": [image_b64],
                        }
                    ],
                    options = {
                        "temperature": config.OLLAMA_VISION_TEMPERATURE,
                        "num_predict":
                        config.OLLAMA_VISION_NUM_PREDICT_IMAGE,
                        "num_ctx": config.OLLAMA_VISION_NUM_CTX,
                    },
                )

                return response["message"]["content"].strip()  # type: ignore[no-any-return]

        except ResponseError as e:
            if e.status_code == status.HTTP_404_NOT_FOUND:
                raise VisionError(
                    f"Model not loaded. Run: ollama pull {config.settings.vision_model}"
                ) from e
            raise VisionError(f"Vision model error: {e.error}") from e
        except Exception as e:
            logger.error(f"Image analysis failed: {e}")
            raise VisionError(f"Failed to analyze image: {e!s}") from e

    async def _analyze_single_frame(self, frame_path: Path) -> str:
      
```

---

## Class Documentation

### VisionService Documentation

**Class Responsibility and Purpose:**
The `VisionService` class is responsible for analyzing images and video frames using the Qwen2.5-VL model via an OllamaManager instance. It handles concurrency control through semaphores to manage access to the vision model, ensuring smooth operation even under high load.

**Public Interface (Key Methods):**
- **`analyze_image(image_path: Path) -> str`:** Analyzes a single image file and returns a text description of its content.
- **`_analyze_single_frame(frame_path: Path) -> str`:** A helper method for analyzing individual video frames, ensuring each frame is processed separately to avoid model bugs.
- **`analyze_video(frame_paths: list[Path], max_frames: int = config.MAX_VIDEO_FRAMES_FOR_ANALYSIS) -> str`:** Analyzes a series of video frames and synthesizes the results into a cohesive description.

**Design Patterns Used:**
The class employs several design patterns:
- **Factory Pattern:** Implicitly used through `OllamaManager` for managing model clients.
- **Observer Pattern:** Not explicitly implemented but implied by the interaction with the OllamaManager.
- **Strategy Pattern:** The `analyze_image` and `_analyze_single_frame` methods handle different types of input (images vs. frames) using a consistent interface.

**Relationship to Other Classes:**
The `VisionService` interacts with the `OllamaManager` for model access and concurrency control via a semaphore. It also relies on configuration settings from `config.settings` for model parameters like temperature and context length.

**State Management Approach:**
The class manages state through instance variables (`_ollama`, `_semaphore`) and leverages Python's asynchronous capabilities with `asyncio`. The use of semaphores ensures that only a limited number of concurrent requests are made to the vision model, preventing overloading.

---

*Generated by CodeWorm on 2026-02-19 02:58*
