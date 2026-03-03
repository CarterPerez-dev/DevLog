# models_api

**Type:** Code Evolution
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/ai-threat-detection/backend/app/api/models_api.py
**Language:** python
**Lines:** 1-1
**Complexity:** 0.0

---

## Source Code

```python
Commit: 4224b7d2
Message: 97% complete
Author: CarterPerez-dev
File: PROJECTS/advanced/ai-threat-detection/backend/app/api/models_api.py
Change type: modified

Diff:
@@ -3,19 +3,30 @@
 models_api.py
 """
 
+import logging
 import uuid
 
-from fastapi import APIRouter, Request
-from sqlalchemy import select
-from sqlalchemy.ext.asyncio import AsyncSession
+from fastapi import APIRouter, BackgroundTasks, Request
+from sqlalchemy import func, select
+from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker
 
+from app.config import settings
 from app.models.model_metadata import ModelMetadata
+from app.models.threat_event import ThreatEvent
+
+logger = logging.getLogger(__name__)
 
 router = APIRouter(prefix="/models", tags=["models"])
 
+SCORE_ATTACK_THRESHOLD = 0.5
+SCORE_NORMAL_CEILING = 0.3
+MIN_TRAINING_SAMPLES = 200
+SYNTHETIC_SUPPLEMENT_NORMAL = 500
+SYNTHETIC_SUPPLEMENT_ATTACK = 250
+
 
 @router.get("/status")
-async def model_status(request: Request, ) -> dict[str, object]:
+async def model_status(request: Request) -> dict[str, object]:
     """
     Return the status of active ML models
     """
@@ -36,18 +47,176 @@ async def model_status(request: Request, ) -> dict[str, object]:
 
 
 @router.post("/retrain", status_code=202)
-async def retrain() -> dict[str, object]:
+async def retrain(
+    request: Request,
+    background_tasks: BackgroundTasks,
+) -> dict[str, object]:
     """
-    Trigger an async model retraining job
+    Dispatch a model retraining job using real stored
+    threat events supplemented with synthetic data
     """
-    return {
-        "status": "accepted",
-        "job_id": uuid.uuid4().hex,
-    }
+    session_factory = getattr(request.app.state, "session_factory", None)
+    if session_factory is None:
+        return {"status": "error", "job_id": ""}
+
+    job_id = uuid.uuid4().hex
+    background_tasks.add_task(
+        _retrain_from_db,
+        job_id,
+        session_factory,
+    )
+    return {"status": "accepted", "job_id": job_id}
+
+
+async def _retrain_from_db(
+    job_id: str,
+    session_factory: async_sessionmaker[AsyncSession],
+) -> None:
+    """
+    Pull stored threat events, build training arrays,
+    supplement with synthetic data if needed, and run
+    the full training pipeline
+    """
+    import asyncio
+    import dataclasses
+    from pathlib import Path
+
+    import numpy as np
+
+    from ml.orchestrator import TrainingOrchestrator
+
+    logger.info("Retrain job %s: loading stored events", job_id)
+
+    async with session_factory() as session:
+        count = (await session.execute(
+            select(func.count()).select_from(ThreatEvent)
+        )).scalar_one()
+
+        if count == 0:
+            logger.warning(
+                "Retrain job %s: no stored events, using synthetic only",
+                job_id,
+            )
+            _fallback_synthetic(job_id)
+            return
+
+        rows = (await session.execute(
+            select(Threat
```

---

## Code Evolution

### Change Analysis

**What was Changed:**
The code in `models_api.py` was modified to include additional imports, logging setup, and a new function `_retrain_from_db`. The `retrain` endpoint now accepts `BackgroundTasks`, which allows for asynchronous background processing. The logic within the `retrain` function has been significantly expanded to handle both stored threat events and synthetic data supplementation.

**Why it was Likely Changed:**
This change likely aims to improve the robustness of model retraining by leveraging both real-world and synthetic data. By adding logging, the code now provides more visibility into the retraining process, aiding in debugging and monitoring. The introduction of `BackgroundTasks` ensures that retraining jobs do not block the main request handling.

**Impact on Behavior:**
The behavior of the API has been enhanced to support asynchronous background processing for model retraining. This should improve system responsiveness while allowing more complex data preprocessing steps during retraining without affecting user requests. The logging and additional checks ensure better error handling and data validation, which could lead to more reliable model updates.

**Risks or Concerns:**
- **Complexity**: The new logic introduces complexity, particularly with the handling of synthetic data and the integration of external libraries.
- **Performance**: Running the training pipeline in a separate thread might introduce performance overhead if not managed properly.
- **Error Handling**: While logging is improved, there are still potential points where errors could occur without proper exception handling.

Overall, this change appears to be a feature enhancement aimed at improving model retraining capabilities and system robustness.

---

*Generated by CodeWorm on 2026-03-03 00:52*
