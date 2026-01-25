# get_current_metrics

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
        optimization_level: if cfg!(debug_assertions) { "none".to_string() } else { "3".to_string() },
        features_enabled: get_enabled_features(),
    };

    let response = CurrentMetricsResponse {
        timestamp: chrono::Utc::now(),
        system: system_perf,
        application: app_perf,
        hardware: hardware_info,
        runtime: runtime_info,
    };

    info!("Performance metrics collected successfully");
    Ok(Json(response))
}
```

---

## Documentation

### Documentation for `get_current_metrics` Function

**Purpose and Behavior:**
The `get_current_metrics` function fetches system, application, hardware, and runtime performance metrics asynchronously. It collects CPU usage, memory statistics, disk space, load averages, uptime, process count, and other details from the system. Additionally, it provides information about the Rust compiler version, build type, optimization level, and enabled features.

**Key Implementation Details:**
- The function uses `async` to handle asynchronous operations.
- It retrieves metrics using the `System` struct from the `systemstat` crate.
- CPU and memory usage are calculated as percentages of total capacity.
- Disk space is checked for the first available disk.
- The response is formatted into a `JsonResponse<CurrentMetricsResponse>`.

**When/Why to Use:**
This function should be used in monitoring or diagnostic tools where real-time performance metrics are needed. It can help in identifying bottlenecks, resource usage patterns, and system health during runtime.

**Patterns/Gotchas:**
- The use of `System::new_all()` and `refresh_all()` might not work on all systems due to permission restrictions.
- Handling the first disk for storage statistics could lead to issues if no disks are available or multiple disks exist.
- Ensure proper error handling, especially when dealing with optional values like CPU core count.

This function provides a comprehensive overview of system performance, making it invaluable for monitoring and debugging purposes.

---

*Generated by CodeWorm on 2026-01-24 19:35*
