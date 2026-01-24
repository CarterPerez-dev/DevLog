# Config.from_env

**Repository:** kill-pr0cess.inc
**File:** backend/src/utils/config.rs
**Language:** rust
**Lines:** 84-161
**Complexity:** 34.0

---

## Source Code

```rust
pub fn from_env() -> Result<Self> {
        info!("Loading configuration from environment variables");

        // Load environment type first to set appropriate defaults
        let environment = parse_environment()?;

        let config = Config {
            // Server configuration
            host: env::var("HOST").unwrap_or_else(|_| "0.0.0.0".to_string()),
            port: parse_env_var("PORT", 3001)?,
            environment: environment.clone(),

            // Database configuration with environment-specific defaults
            database_url: get_required_env("DATABASE_URL")?,
            database_max_connections: parse_env_var("DATABASE_MAX_CONNECTIONS",
                if environment == Environment::Production { 100 } else { 20 })?,
            database_min_connections: parse_env_var("DATABASE_MIN_CONNECTIONS", 5)?,
            database_connection_timeout: parse_env_var("DATABASE_CONNECTION_TIMEOUT", 30)?,

            // Redis configuration
            redis_url: get_required_env("REDIS_URL")?,
            redis_max_connections: parse_env_var("REDIS_MAX_CONNECTIONS", 10)?,
            redis_connection_timeout: parse_env_var("REDIS_CONNECTION_TIMEOUT", 5)?,

            // GitHub API configuration
            github_token: get_required_env("GITHUB_TOKEN")?,
            github_username: get_required_env("GITHUB_USERNAME")?,
            github_api_base_url: env::var("GITHUB_API_BASE_URL")
                .unwrap_or_else(|_| "https://api.github.com".to_string()),
            github_rate_limit_requests: parse_env_var("GITHUB_RATE_LIMIT_REQUESTS", 5000)?,
            github_cache_ttl: parse_env_var("GITHUB_CACHE_TTL", 1800)?,

            // Frontend configuration
            frontend_url: env::var("FRONTEND_URL").unwrap_or_else(|_| "http://localhost:4000".to_string()),
            cors_allowed_origins: parse_cors_origins()?,

            // Performance monitoring
            metrics_enabled: parse_bool_env("METRICS_ENABLED", true)?,
            prometheus_port: parse_env_var("PROMETHEUS_PORT", 9090)?,
            system_metrics_interval: parse_env_var("SYSTEM_METRICS_INTERVAL", 60)?,

            // Fractal computation limits for safety
            fractal_max_width: parse_env_var("MAX_FRACTAL_WIDTH", 4096)?,
            fractal_max_height: parse_env_var("MAX_FRACTAL_HEIGHT", 4096)?,
            fractal_max_iterations: parse_env_var("MAX_FRACTAL_ITERATIONS", 10000)?,
            fractal_max_zoom: parse_env_var("MAX_FRACTAL_ZOOM", 1e15)?,
            fractal_computation_timeout: parse_env_var("FRACTAL_COMPUTATION_TIMEOUT", 120)?,

            // Logging configuration
            log_level: env::var("RUST_LOG").unwrap_or_else(|_|
                match environment {
                    Environment::Development => "debug".to_string(),
                    Environment::Staging => "info".to_string(),
                    Environment::Production => "warn".to_string(),
                }
            ),
            log_format: parse_log_format()?,

            // Security configuration
            rate_limit_enabled: parse_bool_env("RATE_LIMIT_ENABLED", true)?,
            rate_limit_requests_per_minute: parse_env_var("RATE_LIMIT_REQUESTS_PER_MINUTE",
                if environment == Environment::Production { 60 } else { 100 })?,
            fractal_rate_limit_per_minute: parse_env_var("FRACTAL_RATE_LIMIT_PER_MINUTE", 10)?,

            // Caching configuration
            cache_enabled: parse_bool_env("CACHE_ENABLED", true)?,
            cache_default_ttl: parse_env_var("CACHE_DEFAULT_TTL", 3600)?,
            github_cache_enabled: parse_bool_env("GITHUB_CACHE_ENABLED", true)?,
        };

        // Validate configuration after loading
        config.validate()?;

        info!("Configuration loaded successfully for environment: {:?}", config.environment);
        config.log_configuration_summary();

        Ok(config)
    }
```

---

## Documentation

### Documentation for `Config::from_env`

**Purpose and Behavior:**
The `from_env` function in the `Config` struct loads configuration settings from environment variables, ensuring appropriate defaults are set based on the environment (Development, Staging, Production). It validates the loaded configuration after loading and logs a summary.

**Key Implementation Details:**
- **Environment Handling:** The function first loads the environment type to determine default values for other configurations.
- **Error Handling:** Uses `Result` extensively to handle potential errors from environment variable parsing or validation.
- **Configuration Validation:** Validates the configuration after loading, ensuring all required fields are set correctly.

**When/Why to Use:**
This code is used in environments where configuration settings need to be dynamically loaded at runtime. It ensures that different environments (Development, Staging, Production) have appropriate configurations without hardcoding values into the application.

**Patterns and Gotchas:**
- **Borrowing:** The `environment` variable is cloned to avoid borrowing issues when setting default values.
- **Default Values:** Default values are set based on the environment type, which can lead to subtle bugs if not carefully managed.
- **Error Propagation:** The use of `unwrap_or_else` can hide errors; consider using `expect` with a custom error message for better debugging.

---

*Generated by CodeWorm on 2026-01-23 21:36*
