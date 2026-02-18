# AtlasClient

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/services/clients/atlas_client.py
**Language:** python
**Lines:** 24-169
**Complexity:** 0.0

---

## Source Code

```python
class AtlasClient:
    """
    MongoDB Atlas Metrics API v2 client for cluster-wide performance monitoring.
    """
    def __init__(self, config):
        """
        Initialize Atlas client with configuration
        """
        self.config = config
        self.base_url = "https://cloud.mongodb.com/api/atlas/v2"
        self.auth = HTTPDigestAuth(
            config.ATLAS_PUBLIC_KEY,
            config.ATLAS_PRIVATE_KEY
        )
        self.headers = {
            'Accept': 'application/vnd.atlas.2025-03-12+json',
            'Content-Type': 'application/vnd.atlas.2025-03-12+json'
        }
        self.timeout = getattr(config, 'ATLAS_REQUEST_TIMEOUT', 30)

    def get_cluster_metrics(self,
                            request: AtlasMetricsRequest) -> dict[str,
                                                                  Any]:
        """
        Fetch cluster-wide performance metrics from Atlas API v2
        """
        url = f"{self.base_url}/groups/{self.config.ATLAS_GROUP_ID}/processes/{self.config.ATLAS_PROCESS_ID}/measurements"

        params = {
            "granularity": request.granularity,
            "start": request.start.isoformat().replace('+00:00',
                                                       'Z'),
            "end": request.end.isoformat().replace('+00:00',
                                                   'Z'),
            "metrics": request.metrics
        }

        response = requests.get(
            url,
            auth = self.auth,
            headers = self.headers,
            params = params,
            timeout = self.timeout
        )
        response.raise_for_status()
        return dict(response.json())

    def get_database_level_metrics(
        self,
        database_name: str,
        request: AtlasMetricsRequest
    ) -> dict[str,
              Any]:
        """
        Fetch database-level metrics from Atlas API v2
        """
        url = f"{self.base_url}/groups/{self.config.ATLAS_GROUP_ID}/processes/{self.config.ATLAS_PROCESS_ID}/databases/{database_name}/measurements"

        params = {
            "granularity": request.granularity,
            "start": request.start.isoformat().replace('+00:00',
                                                       'Z'),
            "end": request.end.isoformat().replace('+00:00',
                                                   'Z'),
            "metrics": request.metrics
        }

        response = requests.get(
            url,
            auth = self.auth,
            headers = self.headers,
            params = params,
            timeout = self.timeout
        )
        response.raise_for_status()
        return dict(response.json())

    @staticmethod
    def extract_metric_values(raw_data: dict[str,
                                             Any],
                              metric_name: str) -> list[float]:
        """
        Extract data points for a specific metric from Atlas response
        """
        measurements = raw_d
```

---

## Class Documentation

### AtlasClient Documentation

**Class Responsibility and Purpose**
The `AtlasClient` class provides an interface to interact with MongoDB Atlas Metrics API v2 for fetching cluster-wide and database-level performance metrics. It encapsulates the logic required to authenticate, construct requests, and process responses from the API.

**Public Interface (Key Methods)**
- **`__init__(self, config)`**: Initializes the client with a configuration object containing necessary credentials and settings.
- **`get_cluster_metrics(self, request: AtlasMetricsRequest) -> dict[str, Any]`**: Fetches cluster-wide performance metrics based on specified parameters.
- **`get_database_level_metrics(self, database_name: str, request: AtlasMetricsRequest) -> dict[str, Any]`**: Fetches database-level performance metrics for a specific database.
- **`extract_metric_values(self, raw_data: dict[str, Any], metric_name: str) -> list[float]`**: Extracts data points for a specific metric from the API response.
- **`calculate_metric_summary(self, values: list[float]) -> dict[str, float]`**: Calculates summary statistics for a given set of metric values.
- **`test_connection(self) -> dict[str, Any]`**: Tests connectivity to the Atlas API by making a simple metrics request.

**Design Patterns Used**
The class does not explicitly use any design patterns. However, it follows Pythonic conventions with type hints and static methods for utility functions.

**Relationship to Other Classes**
- **Configuration Management**: The `AtlasClient` relies on a configuration object (`config`) that contains credentials and settings.
- **Metrics Request Handling**: It interacts with the `AtlasMetricsRequest` class to construct requests, which is not shown here but assumed to be defined elsewhere in the codebase.

**State Management Approach**
The client maintains state through its instance variables such as `auth`, `headers`, and `timeout`. These are initialized during construction and used throughout the lifecycle of the object. The methods do not modify these attributes directly, ensuring a clean separation of concerns.

---

*Generated by CodeWorm on 2026-02-18 12:53*
