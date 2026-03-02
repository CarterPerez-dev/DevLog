# SystemHealthScore

**Type:** Performance Analysis
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
        
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity (Big O Notation)
The time complexity of the `SystemHealthScore` function is **O(1)**, as it performs a fixed number of operations regardless of input size.

#### Space Complexity and Memory Allocation
- The space complexity is also **O(1)** since no additional data structures are created that scale with the input size.
- However, there are redundant calculations. For example, `getStatusColor` and `getStatusLabel` are called multiple times for each condition check, which can be optimized.

#### Bottlenecks or Inefficiencies
- Redundant function calls: `getStatusColor`, `getStatusLabel`, and `getStatusIcon` are recalculated in every conditional branch.
- String concatenation inside the JSX elements is not a significant bottleneck but can be avoided using template literals for better readability.

#### Optimization Opportunities
1. **Combine Function Calls**: Combine the repeated calls to `getStatusColor`, `getStatusLabel`, and `getStatusIcon` into a single function that returns an object with all necessary properties.
2. **Memoization**: Use React's memoization techniques (like `React.memo`) or functional state in hooks to avoid recalculating these values unnecessarily.

#### Resource Usage Concerns
- No blocking calls, N+1 query patterns, or resource leaks are present in this code snippet.
- Ensure that the `health` object and other props are properly typed and validated to prevent potential errors.

### Suggested Optimizations

```tsx
const getStatusInfo = (status: string): { color: string; icon: JSX.Element; label: string } => {
  switch (status) {
    case 'healthy':
      return { color: '#10b981', icon: <FiCheck />, label: 'Healthy' };
    case 'degraded':
      return { color: '#f59e0b', icon: <FiActivity />, label: 'Degraded' };
    case 'unhealthy':
      return { color: '#ef4444', icon: <FiAlertTriangle />, label: 'Unhealthy' };
    default:
      return { color: '#6b7280', icon: <FiInfo />, label: 'Unknown' };
  }
};

const healthStatus = getStatusInfo(health.status);

return (
  // Simplified JSX using the combined info
);
```

This refactoring reduces redundancy and improves code readability.

---

*Generated by CodeWorm on 2026-03-02 07:31*
