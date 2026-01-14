# CircuitBreaker.call

**Repository:** kill-pr0cess.inc
**File:** backend/src/utils/mod.rs
**Language:** rust
**Lines:** 351-420
**Complexity:** 10.0

---

## Source Code

```rust
pub fn call<F, T, E>(&self, operation: F) -> std::result::Result<T, E>
    where
        F: FnOnce() -> std::result::Result<T, E>,
        E: From<AppError>,
    {
        let can_proceed = {
            let mut current_state_guard = self.state.lock().unwrap(); // Lock to read and potentially modify
            match *current_state_guard {
                CircuitState::Closed => true,
                CircuitState::HalfOpen => true, // Allow one attempt in HalfOpen
                CircuitState::Open => {
                    let last_failure_time_guard = self.last_failure_time.lock().unwrap();
                    if let Some(last_failure) = *last_failure_time_guard {
                        if Instant::now().duration_since(last_failure) > self.timeout {
                            info!("CircuitBreaker: Timeout elapsed, transitioning from Open to HalfOpen.");
                            *current_state_guard = CircuitState::HalfOpen;
                            true // Allow this call as the first attempt in HalfOpen
                        } else {
                            // Still in Open state, timeout not elapsed
                            false
                        }
                    } else {
                        // Should not happen if last_failure_time is always set on failure
                        // but if it does, stay open.
                        warn!("CircuitBreaker: In Open state but no last_failure_time recorded.");
                        false
                    }
                }
            }
        };

        if !can_proceed {
            return Err(AppError::ServiceUnavailableError(
                "Circuit breaker is OPEN".to_string(),
            )
            .into());
        }

        match operation() {
            Ok(result) => {
                let mut current_state_guard = self.state.lock().unwrap();
                if *current_state_guard == CircuitState::HalfOpen {
                    info!("CircuitBreaker: Successful call in HalfOpen state, transitioning to Closed.");
                }
                *current_state_guard = CircuitState::Closed;
                *self.failure_count.lock().unwrap() = 0;
                *self.last_failure_time.lock().unwrap() = None; // Clear last failure time
                Ok(result)
            }
            Err(error) => {
                let mut failure_count_guard = self.failure_count.lock().unwrap();
                let mut current_state_guard = self.state.lock().unwrap();
                let mut last_failure_time_guard = self.last_failure_time.lock().unwrap();

                *failure_count_guard += 1;
                *last_failure_time_guard = Some(Instant::now());

                if *current_state_guard == CircuitState::HalfOpen {
                    // Failure in HalfOpen state, trip back to Open
                    info!("CircuitBreaker: Failure in HalfOpen state, transitioning back to Open.");
                    *current_state_guard = CircuitState::Open;
                } else if *failure_count_guard >= self.failure_threshold {
                    // Failure threshold reached in Closed state, trip to Open
                    info!("CircuitBreaker: Failure threshold reached, transitioning from Closed to Open.");
                    *current_state_guard = CircuitState::Open;
                }
                Err(error)
            }
        }
    }
```

---

## Documentation

### Documentation for `CircuitBreaker.call`

**Purpose and Behavior:**
The `call` function in the `CircuitBreaker` class is designed to execute a given operation while managing circuit breaker logic. It checks the current state of the circuit (Closed, HalfOpen, or Open) before executing the operation. If the operation succeeds, it updates the state to Closed; if it fails, it increments failure counts and transitions states accordingly.

**Key Implementation Details:**
- **State Management:** The function uses `state`, `failure_count`, and `last_failure_time` to track circuit states and failures.
- **Locking Mechanism:** It employs Rust's `lock()` method to safely access shared state. Errors are handled using `Result<T, E>` with `From<AppError>` for conversion.
- **Conditional Transitions:** The function transitions between states based on failure counts and timeout conditions.

**When/Why to Use:**
This code is ideal for scenarios where you need robust error handling and resilience in service calls. It helps prevent cascading failures by temporarily blocking requests during a service outage, allowing recovery before resuming normal operations.

**Patterns and Gotchas:**
- **Ownership and Borrowing:** The function uses `lock()` to manage shared state, ensuring thread safety.
- **Circuit Breaker Pattern:** This implementation effectively demonstrates the circuit breaker pattern, which is crucial for maintaining system stability during outages.
- **Timeout Handling:** The HalfOpen state allows one attempt after a timeout, but subsequent failures will trip the circuit back to Open.

---

*Generated by CodeWorm on 2026-01-14 01:46*
