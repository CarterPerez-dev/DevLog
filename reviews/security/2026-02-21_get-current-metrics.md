# get_current_metrics

**Type:** Security Review
**Repository:** kill-pr0cess.inc
**File:** backend/src/routes/performance.rs
**Language:** rust
**Lines:** 81-163
**Complexity:** 7.0

---

## Source Code

```rust
pub async fn get_current_metrics(
    State(app_state): State<AppState>,
    Query(params): Query<MetricsQuery>,
) -> Result<JsonResponse<CurrentMetricsResponse>> {
    info!("Fetching current performance metrics");

    // Collect system metrics
    let mut system = System::new_all();
    system.refresh_all();

    let system_perf = SystemPerformance {
        cpu_usage_percent: system.global_cpu_info().cpu_usage() as f64,
        memory_usage_percent: {
            let total = system.total_memory() as f64;
            let available = system.available_memory() as f64;
            ((total - available) / total) * 100.0
        },
        memory_total_gb: system.total_memory() as f64 / (1024.0 * 1024.0 * 1024.0),
        memory_available_gb: system.available_memory() as f64 / (1024.0 * 1024.0 * 1024.0),
        disk_usage_percent: {
            if let Some(disk) = system.disks().first() {
                let total = disk.total_space() as f64;
                let available = disk.available_space() as f64;
                ((total - available) / total) * 100.0
            } else {
                0.0
            }
        },
        load_average_1m: system.load_average().one,
        load_average_5m: system.load_average().five,
        load_average_15m: system.load_average().fifteen,
        uptime_seconds: system.uptime(),
        active_processes: system.processes().len() as u32,
    };

    let cpu_threads = system.cpus().len() as u32;
    let cpu_cores = system.physical_core_count()
        .map(|c| c as u32)
        .unwrap_or_else(|| {
            if cpu_threads > 0 { (cpu_threads / 2).max(1) } else { 1 }
        });

    let hardware_info = HardwareInfo {
        cpu_model: if system.global_cpu_info().brand().is_empty() {
            "Unknown CPU".to_string()
        } else {
            system.global_cpu_info().brand().to_string()
        },
        cpu_cores,
        cpu_threads,
        architecture: std::env::consts::ARCH.to_string(),
        total_memory_gb: system.total_memory() as f64 / (1024.0 * 1024.0 * 1024.0),
    };

    // Application performance metrics (simplified for now)
    let app_perf = ApplicationPerformance {
        requests_handled: 0, // Would be tracked from middleware
        average_response_time_ms: 0.0, // Would be calculated from request timings
        fractal_computations: 0, // Would be tracked from fractal service
        github_api_calls: 0, // Would be tracked from GitHub service
        cache_hit_rate: 0.0, // Would be retrieved from cache service
        database_connections: app_state.db_pool.size(),
        memory_usage_mb: 0.0, // Would be calculated from process memory usage
    };

    let runtime_info = RuntimeInfo {
        rust_version: option_env!("BUILD_RUST_VERSION").unwrap_or("unknown").to_string(),
        build_type: if cfg!(debug_assertions) { "debug".to_string() } else { "release".to_string() },
        optimization_level: if cfg!(debug_assertions) { "none".to_string() } else { "3".to_strin
```

---

## Security Review

### Security Review for `get_current_metrics` Function

#### Vulnerabilities and Severity:

1. **Info - No Injection or Deserialization Issues**: The code does not handle user input in a way that could lead to SQL, command, or XSS injection vulnerabilities.
2. **Info - No Hardcoded Secrets**: There are no hardcoded secrets or credentials visible in the provided snippet.
3. **Info - No Race Conditions or TOCTOU Bugs**: The function does not interact with shared mutable state or perform operations that would be susceptible to race conditions.
4. **Info - Input Validation Gaps**: While the code processes system metrics, it does not validate external inputs like `params` from the `Query<MetricsQuery>` struct.
5. **Info - Insecure Deserialization**: No deserialization is performed in this function.

#### Attack Vectors:

- **Information Disclosure**: The logging statements could potentially leak information about the application's internal state and metrics, which might be exploited by attackers to infer sensitive data or system conditions.

#### Recommended Fixes:

1. **Enhance Logging**: Use structured logging instead of `info!` macros to avoid leaking sensitive information.
   ```rust
   use tracing::field;
   
   info!(fields!(
       "timestamp" => chrono::Utc::now(),
       "system" => serde_json::to_value(system_perf).unwrap(),
       "application" => serde_json::to_value(app_perf).unwrap(),
       "hardware" => serde_json::to_value(hardware_info).unwrap(),
       "runtime" => serde_json::to_value(runtime_info).unwrap()
   ));
   ```

2. **Validate Inputs**: Ensure that the `params` from `Query<MetricsQuery>` are validated to prevent any potential misuse.

3. **Use Structured Data for Logging**: This helps in better control over what data is logged and reduces the risk of sensitive information exposure.

#### Overall Security Posture:

The code is relatively secure, but there are minor improvements that can be made to enhance its security posture, particularly around logging and input validation.

---

*Generated by CodeWorm on 2026-02-21 10:53*
