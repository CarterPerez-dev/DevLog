# SystemHealthScore

**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/monitoring/components/dashboard/SystemHealthScore.tsx
**Language:** tsx
**Lines:** 19-172
**Complexity:** 15.0

---

## Source Code

```tsx
function SystemHealthScore({
  health,
  isLoading,
  error,
}: SystemHealthScoreProps): React.ReactElement {
  if (error !== null && error !== undefined) {
    return (
      <div className={`${styles.healthCard} ${styles.error}`}>
        <div className={styles.errorContent}>
          <FiAlertTriangle className={styles.errorIcon} />
          <div className={styles.errorTitle}>Health Check Failed</div>
          <div className={styles.errorMessage}>{error}</div>
        </div>
      </div>
    );
  }

  if (isLoading) {
    return (
      <div className={`${styles.healthCard} ${styles.loading}`}>
        <div className={styles.loadingContent}>
          <LoadingSpinner />
          <div className={styles.loadingText}>Checking system health...</div>
        </div>
      </div>
    );
  }

  if (health === null || health === undefined) {
    return (
      <div className={`${styles.healthCard} ${styles.unknown}`}>
        <div className={styles.unknownContent}>
          <FiInfo className={styles.unknownIcon} />
          <div className={styles.unknownText}>Health data unavailable</div>
        </div>
      </div>
    );
  }

  const getStatusIcon = (): React.ReactElement => {
    switch (health.status) {
      case 'healthy':
        return <FiCheck />;
      case 'degraded':
        return <FiActivity />;
      case 'unhealthy':
        return <FiAlertTriangle />;
      default:
        return <FiInfo />;
    }
  };

  const getStatusColor = (): string => {
    switch (health.status) {
      case 'healthy':
        return '#10b981'; // success green
      case 'degraded':
        return '#f59e0b'; // warning amber
      case 'unhealthy':
        return '#ef4444'; // danger red
      default:
        return '#6b7280'; // neutral gray
    }
  };

  const getStatusLabel = (): string => {
    switch (health.status) {
      case 'healthy':
        return 'Healthy';
      case 'degraded':
        return 'Degraded';
      case 'unhealthy':
        return 'Unhealthy';
      default:
        return 'Unknown';
    }
  };

  const statusColor = getStatusColor();
  const timestamp = new Date(health.timestamp).toLocaleTimeString();

  return (
    <div className={`${styles.healthCard} ${styles[health.status] ?? ''}`}>
      <div className={styles.scoreSection}>
        <div
          className={styles.scoreCircle}
          style={
            {
              'borderColor': statusColor,
              '--status-color': statusColor,
            } as React.CSSProperties
          }
        >
          <div
            className={styles.scoreValue}
            style={{ color: statusColor }}
          >
            {Math.round(health.health_score)}
          </div>
          <div className={styles.scoreLabel}>Health</div>
        </div>

        <div className={styles.statusBadge}>
          <div
            className={styles.statusIcon}
            style={{ color: statusColor }}
          >
            {getStatusIcon()}
          </div>
          <div
            className={styles.statusText}
            style={{ color: statusColor }}
          >
            {getStatusLabel()}
          </div>
        </div>
      </div>

      <div className={styles.metricsSection}>
        <div className={styles.metricItem}>
          <span className={styles.metricLabel}>Response Time</span>
          <span className={styles.metricValue}>
            {health.metrics.avg_response_time_ms.toFixed(0)}ms
          </span>
        </div>

        <div className={styles.metricItem}>
          <span className={styles.metricLabel}>Error Rate</span>
          <span className={styles.metricValue}>
            {health.metrics.error_rate.toFixed(1)}%
          </span>
        </div>

        <div className={styles.metricItem}>
          <span className={styles.metricLabel}>Requests/min</span>
          <span className={styles.metricValue}>
            {health.metrics.requests_per_minute.toFixed(0)}
          </span>
        </div>

        <div className={styles.metricItem}>
          <span className={styles.metricLabel}>AI Success</span>
          <span className={styles.metricValue}>
            {health.ai_performance.success_rate.toFixed(1)}%
          </span>
        </div>
      </div>

      <div className={styles.timestampSection}>
        <div className={styles.timestamp}>Last updated: {timestamp}</div>
      </div>
    </div>
  );
}
```

---

## Documentation

### Documentation for `SystemHealthScore` Component

**Purpose and Behavior:**
The `SystemHealthScore` component in the CertGames-Core repository is responsible for rendering a health status card that reflects the current state of a system based on provided data. It handles different states such as loading, error, unknown, and various health statuses (healthy, degraded, unhealthy).

**Key Implementation Details:**
- The component accepts `health`, `isLoading`, and `error` props.
- It uses conditional rendering to display appropriate content based on the state of these props.
- Utilizes helper functions like `getStatusIcon`, `getStatusColor`, and `getStatusLabel` for consistent styling and text representation.
- Dynamically applies styles based on health status, ensuring visual feedback is clear.

**When/Why to Use:**
This component should be used in dashboards or monitoring interfaces where real-time system health needs to be displayed. It handles various states gracefully, providing a user-friendly interface that reflects the current state of the system.

**Patterns and Gotchas:**
- The use of CSS classes dynamically based on props is a clean way to manage styles.
- Helper functions like `getStatusIcon`, `getStatusColor`, and `getStatusLabel` enhance reusability and maintainability.
- Ensure that prop types are correctly defined in `SystemHealthScoreProps` to avoid runtime errors.

---

*Generated by CodeWorm on 2026-01-20 15:27*
