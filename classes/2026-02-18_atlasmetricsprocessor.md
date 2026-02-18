# AtlasMetricsProcessor

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/services/clients/atlas_client.py
**Language:** python
**Lines:** 172-389
**Complexity:** 0.0

---

## Source Code

```python
class AtlasMetricsProcessor:
    """
    Process raw Atlas metrics into CertGames admin analytics format
    """

    CLUSTER_METRICS = [
        "CONNECTIONS",
        "OPCOUNTER_QUERY",
        "OPCOUNTER_INSERT",
        "OPCOUNTER_UPDATE",
        "OPCOUNTER_DELETE",
        "OPCOUNTER_GETMORE",
        "OPCOUNTER_CMD",
        "MEMORY_RESIDENT",
        "MEMORY_VIRTUAL",
        "CPU_NORMALIZED",
        "NETWORK_BYTES_IN",
        "NETWORK_BYTES_OUT",
        "QUERY_EXECUTOR_SCANNED_OBJECTS",
        "QUERY_EXECUTOR_SCANNED_INDEXES"
    ]

    @staticmethod
    def process_cluster_performance(
        raw_data: dict[str,
                       Any],
        collection_name: str,
        period_type: str,
        start: datetime,
        end: datetime
    ) -> dict[str,
              Any]:
        """
        Convert Atlas cluster metrics to CertGames DatabaseMetrics format
        """
        query_ops = AtlasClient.extract_metric_values(
            raw_data,
            'OPCOUNTER_QUERY'
        )
        insert_ops = AtlasClient.extract_metric_values(
            raw_data,
            'OPCOUNTER_INSERT'
        )
        update_ops = AtlasClient.extract_metric_values(
            raw_data,
            'OPCOUNTER_UPDATE'
        )
        delete_ops = AtlasClient.extract_metric_values(
            raw_data,
            'OPCOUNTER_DELETE'
        )
        connections = AtlasClient.extract_metric_values(
            raw_data,
            'CONNECTIONS'
        )
        memory_resident = AtlasClient.extract_metric_values(
            raw_data,
            'MEMORY_RESIDENT'
        )
        cpu_usage = AtlasClient.extract_metric_values(
            raw_data,
            'CPU_NORMALIZED'
        )
        scanned_objects = AtlasClient.extract_metric_values(
            raw_data,
            'QUERY_EXECUTOR_SCANNED_OBJECTS'
        )
        scanned_indexes = AtlasClient.extract_metric_values(
            raw_data,
            'QUERY_EXECUTOR_SCANNED_INDEXES'
        )

        all_ops = query_ops + insert_ops + update_ops + delete_ops
        total_operations = sum(all_ops) if all_ops else 0

        total_scanned = sum(scanned_objects) if scanned_objects else 0
        total_index_scans = sum(scanned_indexes) if scanned_indexes else 0

        if total_scanned > 0 or total_index_scans > 0:
            index_hit_rate = (
                total_index_scans / (total_index_scans + total_scanned)
            ) * 100
        else:
            index_hit_rate = 100.0

        slow_queries = AtlasMetricsProcessor._estimate_slow_queries(
            total_scanned,
            total_index_scans
        )

        avg_execution_time = AtlasMetricsProcessor._estimate_avg_execution_time(
            total_operations,
            cpu_usage
        )

        return {
            "collection_name":
            collection_name,
            "operation_type":
            "all",
            "period_type":
            period_type,
            "period_start":
   
```

---

## Class Documentation

### AtlasMetricsProcessor

**Class Responsibility and Purpose**: 
The `AtlasMetricsProcessor` class is responsible for converting raw metrics from the Atlas database into a format suitable for CertGames admin analytics. It processes various cluster performance metrics such as operations, memory usage, CPU utilization, and network bytes.

**Public Interface (Key Methods)**:
- **process_cluster_performance(raw_data: dict[str, Any], collection_name: str, period_type: str, start: datetime, end: datetime) -> dict[str, Any]**: This static method takes raw Atlas metrics data and converts it into a structured format for analytics. It extracts specific metric values, calculates derived statistics like slow queries and index hit rate, and returns the processed data.

- **_estimate_slow_queries(doc_scans: float, index_scans: float) -> int**: A private helper method used to estimate the number of slow queries based on document vs. index scan patterns. This method is crucial for determining performance bottlenecks.

**Design Patterns Used**:
The class does not explicitly use any design patterns but follows a straightforward approach with static methods and data processing logic. The `extract_metric_values` method from `AtlasClient` suggests potential use of the **Strategy Pattern**, where different metrics can be extracted using various strategies.

**How it Fits in the Architecture**:
In the CertGames-Core architecture, `AtlasMetricsProcessor` acts as a service layer that interacts with Atlas database clients to fetch raw data and process it into actionable analytics. It is tightly coupled with the `AtlasClient` for metric extraction but provides a clear interface for higher-level services or applications to consume processed metrics. This separation ensures modularity and ease of maintenance, allowing different parts of the application to focus on their specific responsibilities.

---

*Generated by CodeWorm on 2026-02-18 14:22*
