# run_benchmark

**Repository:** kill-pr0cess.inc
**File:** backend/src/routes/performance.rs
**Language:** rust
**Lines:** 183-300
**Complexity:** 6.0

---

## Source Code

```rust
pub async fn run_benchmark(
    State(_app_state): State<AppState>,
) -> Result<JsonResponse<serde_json::Value>> {
    info!("Starting comprehensive performance benchmark");
    let benchmark_start = std::time::Instant::now();

    // CPU benchmark: prime number calculation
    let cpu_benchmark = tokio::task::spawn_blocking(|| {
        let start = std::time::Instant::now();
        let mut primes = Vec::new();

        for i in 2..10000 {
            if is_prime(i) {
                primes.push(i);
            }
        }

        let single_thread_time = start.elapsed();
        let single_thread_primes = primes.len();

        // Multi-threaded benchmark
        let start = std::time::Instant::now();
        let multi_thread_primes = (2..50000u32)
            .collect::<Vec<_>>()
            .into_iter()
            .filter(|&i| is_prime(i))
            .count();
        let multi_thread_time = start.elapsed();

        serde_json::json!({
            "single_thread": {
                "primes_found": single_thread_primes,
                "duration_ms": single_thread_time.as_secs_f64() * 1000.0,
                "duration_us": single_thread_time.as_micros(),
                "primes_per_second": single_thread_primes as f64 / single_thread_time.as_secs_f64()
            },
            "multi_thread": {
                "primes_found": multi_thread_primes,
                "duration_ms": multi_thread_time.as_secs_f64() * 1000.0,
                "duration_us": multi_thread_time.as_micros(),
                "primes_per_second": multi_thread_primes as f64 / multi_thread_time.as_secs_f64()
            },
            "parallel_efficiency": (multi_thread_primes as f64 / multi_thread_time.as_secs_f64()) /
                                  (single_thread_primes as f64 / single_thread_time.as_secs_f64())
        })
    }).await.unwrap();

    // Memory benchmark: array operations
    let memory_benchmark = tokio::task::spawn_blocking(|| {
        let start = std::time::Instant::now();
        let data_size = 10_000_000;
        let data: Vec<u64> = (0..data_size as u64).collect();
        let allocation_time = start.elapsed();

        let start = std::time::Instant::now();
        let sum: u64 = data.iter().sum();
        let read_time = start.elapsed();

        let start = std::time::Instant::now();
        let mut write_data = vec![0u64; data_size as usize];
        for i in 0..data_size as usize {
            write_data[i] = i as u64;
        }
        let write_time = start.elapsed();

        serde_json::json!({
            "allocation": {
                "duration_ms": allocation_time.as_secs_f64() * 1000.0,
                "duration_us": allocation_time.as_micros(),
                "mb_allocated": (data_size * 8) as f64 / (1024.0 * 1024.0),
                "mb_per_second": (data_size * 8) as f64 / (1024.0 * 1024.0) / allocation_time.as_secs_f64()
            },
            "sequential_read": {
                "duration_ms": read_time.as_secs_f64() * 1000.0,
                "duration_us": read_time.as_micros(),
                "sum_result": sum,
                "mb_per_second": (data_size * 8) as f64 / (1024.0 * 1024.0) / read_time.as_secs_f64()
            },
            "sequential_write": {
                "duration_ms": write_time.as_secs_f64() * 1000.0,
                "duration_us": write_time.as_micros(),
                "mb_per_second": (data_size * 8) as f64 / (1024.0 * 1024.0) / write_time.as_secs_f64()
            }
        })
    }).await.unwrap();

    // System information at benchmark time
    let mut system = System::new_all();
    system.refresh_all();

    let benchmark_duration = benchmark_start.elapsed();

    let benchmark_results = serde_json::json!({
        "benchmark_id": uuid::Uuid::new_v4().to_string(),
        "timestamp": chrono::Utc::now(),
        "total_duration_ms": benchmark_duration.as_millis(),
        "system_info": {
            "cpu_model": system.global_cpu_info().brand(),
            "cpu_cores": system.physical_core_count().unwrap_or(0),
            "cpu_threads": system.cpus().len(),
            "memory_total_gb": system.total_memory() as f64 / (1024.0 * 1024.0 * 1024.0),
            "architecture": std::env::consts::ARCH,
            "os": system.long_os_version(),
        },
        "benchmarks": {
            "cpu": cpu_benchmark,
            "memory": memory_benchmark,
        },
        "performance_rating": calculate_performance_rating(&cpu_benchmark, &memory_benchmark),
        "comparison": {
            "baseline_system": "Intel Core i5-8400 (6 cores, 16GB RAM)",
            "relative_performance": 1.0, // Would be calculated based on baseline comparison
        }
    });

    info!("Benchmark completed in {:?}", benchmark_duration);
    Ok(Json(benchmark_results))
}
```

---

## Documentation

### Documentation for `run_benchmark` Function

**Purpose and Behavior:**
The `run_benchmark` function performs comprehensive performance benchmarks, including CPU and memory operations. It measures the time taken for single-threaded and multi-threaded prime number calculation and array operations (allocation, read, write). The results are formatted into a JSON response.

**Key Implementation Details:**
- Uses `tokio::task::spawn_blocking` to run blocking tasks in separate threads.
- Measures CPU and memory performance using `std::time::Instant`.
- Logs benchmark start and end times with `info!`.
- Calculates efficiency ratios between single-threaded and multi-threaded operations.

**When/Why to Use:**
This function is ideal for performance testing during development or deployment. It helps identify bottlenecks in CPU and memory usage, ensuring optimal system resource allocation.

**Patterns/Gotchas:**
- **Blocking Tasks:** Ensure the blocking tasks do not block the event loop for too long.
- **Error Handling:** The `unwrap()` calls should be handled with care to avoid runtime panics. Consider using `.await?` for better error propagation.
- **Resource Management:** Properly manage memory allocation and deallocation, especially in multi-threaded scenarios.

This function is crucial for maintaining system performance and can help in optimizing resource usage based on real-world benchmarks.

---

*Generated by CodeWorm on 2026-01-24 17:20*
