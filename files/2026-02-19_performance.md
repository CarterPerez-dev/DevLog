# performance

**Type:** File Overview
**Repository:** kill-pr0cess.inc
**File:** backend/src/models/performance.rs
**Language:** rust
**Lines:** 1-678
**Complexity:** 0.0

---

## Source Code

```rust
/*
 * Performance monitoring models defining comprehensive system metrics, benchmark structures, and analytical data for the showcase backend.
 * I'm implementing detailed performance tracking with time-series data, alerting capabilities, and resource utilization monitoring that showcases the application's computational efficiency.
 */

use serde::{Deserialize, Serialize};
use sqlx::FromRow;
use chrono::{DateTime, Utc};
use std::collections::HashMap;
use validator::{Validate, ValidationError};

/// Comprehensive system performance metrics snapshot
/// I'm capturing all essential system performance indicators for real-time monitoring
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct SystemInfo {
    pub timestamp: DateTime<Utc>,
    pub cpu_model: String,
    pub cpu_cores: u32,
    pub cpu_threads: u32,
    pub cpu_usage_percent: f64,
    pub cpu_frequency_mhz: Option<u32>,
    pub memory_total_mb: u64,
    pub memory_available_mb: u64,
    pub memory_usage_percent: f64,
    pub swap_total_mb: u64,
    pub swap_used_mb: u64,
    pub disk_total_gb: f64,
    pub disk_available_gb: f64,
    pub disk_usage_percent: f64,
    pub network_interfaces: Vec<NetworkInterface>,
    pub load_average_1m: f64,
    pub load_average_5m: f64,
    pub load_average_15m: f64,
    pub uptime_seconds: u64,
    pub active_processes: u32,
    pub system_temperature: Option<f64>,
    pub power_consumption: Option<PowerMetrics>,
}

/// Network interface performance metrics
/// I'm tracking network performance for comprehensive system monitoring
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct NetworkInterface {
    pub name: String,
    pub bytes_sent: u64,
    pub bytes_received: u64,
    pub packets_sent: u64,
    pub packets_received: u64,
    pub errors_in: u64,
    pub errors_out: u64,
    pub speed_mbps: Option<u32>,
}

/// Power consumption and efficiency metrics
/// I'm monitoring power usage for sustainability insights
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PowerMetrics {
    pub total_watts: f64,
    pub cpu_watts: Option<f64>,
    pub gpu_watts: Option<f64>,
    pub efficiency_score: f64,
}

/// Individual performance metric with metadata and context
/// I'm implementing flexible metric tracking with rich metadata support
#[derive(Debug, Clone, Serialize, Deserialize, FromRow)]
pub struct PerformanceMetric {
    pub id: uuid::Uuid,
    pub metric_type: String,
    pub metric_name: String,
    pub metric_value: f64,
    pub metric_unit: String,
    pub tags: serde_json::Value,
    pub timestamp: DateTime<Utc>,
    pub endpoint: Option<String>,
    pub user_agent: Option<String>,
    pub ip_address: Option<std::net::IpAddr>,
    pub session_id: Option<uuid::Uuid>,
    pub server_instance: Option<String>,
    pub environment: String,
}

impl PerformanceMetric {
    pub fn new(
        metric_type: impl Into<String>,
        metric_name: impl Into<String>,
        value: f64,
        unit: impl Into<String>,
    ) -> Self {
   
```

---

## File Overview

# Performance Monitoring Models

**Purpose and Responsibility:**
This file defines comprehensive performance monitoring models for the backend of the `kill-pr0cess.inc` application. It includes system information, network interfaces, power consumption metrics, individual performance metrics with metadata, and standardized metric types. These models facilitate real-time monitoring, alerting, and resource utilization analysis.

**Key Exports and Public Interface:**
- **SystemInfo:** A comprehensive snapshot of system performance indicators.
- **NetworkInterface:** Metrics for each network interface.
- **PowerMetrics:** Power consumption and efficiency metrics.
- **PerformanceMetric:** Individual performance metric with rich metadata support.
- **MetricType:** Enumerated types for standardized categorization.

**How It Fits in the Project:**
This file is a crucial component of the backend, providing essential data models used by various modules for monitoring system health. `SystemInfo` and `NetworkInterface` are used to capture real-time system and network performance, while `PerformanceMetric` and `PowerMetrics` enable detailed tracking of application-specific metrics.

**Notable Design Decisions:**
- **Strong Typing:** The use of enums like `MetricType` ensures type safety and consistent categorization.
- **Metadata Support:** `PerformanceMetric` includes extensive metadata fields to provide context for each metric, enhancing data integrity and usability.
- **Serde Integration:** Serialization and deserialization support via `serde` and `FromRow` enable easy integration with database operations.
```

This documentation provides a high-level overview of the file's purpose, key exports, role in the project, and notable design decisions.

---

*Generated by CodeWorm on 2026-02-19 22:34*
