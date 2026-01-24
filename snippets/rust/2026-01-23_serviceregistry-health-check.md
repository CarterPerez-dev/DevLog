# ServiceRegistry.health_check

**Repository:** kill-pr0cess.inc
**File:** backend/src/services/mod.rs
**Language:** rust
**Lines:** 74-175
**Complexity:** 9.0

---

## Source Code

```rust
pub async fn health_check(&self) -> Result<serde_json::Value> {
        let mut health_results = serde_json::Map::new();

        // Check cache service health
        match self.cache_service.health_check().await {
            Ok(cache_health) => {
                health_results.insert("cache".to_string(), cache_health);
            }
            Err(e) => {
                health_results.insert("cache".to_string(), serde_json::json!({
                    "status": "unhealthy",
                    "error": e.to_string()
                }));
            }
        }

        // Check GitHub service health (rate limit status)
        match self.github_service.get_rate_limit_status().await {
            Ok(rate_limit) => {
                health_results.insert("github".to_string(), serde_json::json!({
                    "status": if rate_limit.remaining > 100 { "healthy" } else { "degraded" },
                    "rate_limit_remaining": rate_limit.remaining,
                    "rate_limit_total": rate_limit.limit
                }));
            }
            Err(e) => {
                health_results.insert("github".to_string(), serde_json::json!({
                    "status": "unhealthy",
                    "error": e.to_string()
                }));
            }
        }

        // Check fractal service health (simple computation test)
        let fractal_health = tokio::task::spawn_blocking({
            let fractal_service = Arc::clone(&self.fractal_service);
            move || {
                use crate::services::fractal_service::{FractalRequest, FractalType};

                let test_request = FractalRequest {
                    width: 32,
                    height: 32,
                    center_x: -0.5,
                    center_y: 0.0,
                    zoom: 1.0,
                    max_iterations: 50,
                    fractal_type: FractalType::Mandelbrot,
                };

                fractal_service.generate_mandelbrot(test_request)
            }
        }).await;

        match fractal_health {
            Ok(result) => {
                health_results.insert("fractals".to_string(), serde_json::json!({
                    "status": "healthy",
                    "test_computation_time_ms": result.computation_time_ms
                }));
            }
            Err(e) => {
                health_results.insert("fractals".to_string(), serde_json::json!({
                    "status": "unhealthy",
                    "error": e.to_string()
                }));
            }
        }

        // Check performance service health
        match self.performance_service.get_system_info().await {
            Ok(_) => {
                health_results.insert("performance".to_string(), serde_json::json!({
                    "status": "healthy"
                }));
            }
            Err(e) => {
                health_results.insert("performance".to_string(), serde_json::json!({
                    "status": "unhealthy",
                    "error": e.to_string()
                }));
            }
        }

        // Determine overall health status
        let overall_status = if health_results.values().all(|v| {
            v.get("status").and_then(|s| s.as_str()) == Some("healthy")
        }) {
            "healthy"
        } else if health_results.values().any(|v| {
            v.get("status").and_then(|s| s.as_str()) == Some("unhealthy")
        }) {
            "unhealthy"
        } else {
            "degraded"
        };

        Ok(serde_json::json!({
            "status": overall_status,
            "timestamp": chrono::Utc::now(),
            "services": health_results
        }))
    }
```

---

## Documentation

### Documentation for `health_check` Function

**Purpose and Behavior:**
The `health_check` function in the `ServiceRegistry` struct performs a comprehensive health check on multiple services, including cache, GitHub, fractals, and performance. It returns a JSON object summarizing the status of each service and the overall system health.

**Key Implementation Details:**
- The function uses asynchronous methods to check the health of various services.
- For each service, it either inserts a "healthy" or "unhealthy" status into a `serde_json::Map`, depending on the result.
- A blocking task is used for the fractal service health check due to its computational nature.
- An overall health status ("healthy", "unhealthy", or "degraded") is determined based on the individual service statuses.

**When/Why to Use:**
Use this function in a monitoring system or during startup/shutdown procedures to ensure all services are functioning correctly. It provides detailed insights into each service's health, making it easier to identify and address issues.

**Patterns and Gotchas:**
- **Asynchronous Handling:** The use of `async` for service checks ensures non-blocking behavior.
- **Blocking Task:** The fractal service check uses a blocking task due to its computational requirements. Ensure this does not block the event loop for too long.
- **Error Handling:** Proper error handling is implemented, converting errors into JSON objects with descriptive status messages.

This function is crucial for maintaining system reliability and diagnosing issues in real-time.

---

*Generated by CodeWorm on 2026-01-23 22:21*
