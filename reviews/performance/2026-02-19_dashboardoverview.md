# DashboardOverview

**Type:** Performance Analysis
**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/analytics/components/dashboard/DashboardOverview.tsx
**Language:** tsx
**Lines:** 15-203
**Complexity:** 50.0

---

## Source Code

```tsx
function DashboardOverview(): React.JSX.Element {
  const { data, isLoading, error, isRefetching } = useDashboardOverview();
  const { data: errorTrends } = useErrorTrends({
    days: 1,
    interval: 'hourly',
  });

  if (isLoading && !isRefetching) {
    return (
      <div className={styles.loadingContainer}>
        <div className={styles.spinner} />
        <p>Loading dashboard metrics...</p>
      </div>
    );
  }

  if (error !== null && error !== undefined) {
    return (
      <div className={styles.errorContainer}>
        <FiAlertCircle className={styles.errorIcon} />
        <h3>Failed to Load Dashboard</h3>
        <p>Unable to fetch analytics data. Please try again later.</p>
      </div>
    );
  }

  if (data === null || data === undefined) {
    return (
      <div className={styles.emptyContainer}>
        <p>No analytics data available</p>
      </div>
    );
  }

  const calculateHealthScore = (): number => {
    if (data === null || data === undefined) return 0;

    let score = 100;

    if (data.errors !== null && data.errors !== undefined) {
      const errorRate = data.errors.unresolved / Math.max(data.errors.total, 1);
      score -= errorRate * 30;
    }

    if (
      data.performance?.slow_endpoints !== null &&
      data.performance?.slow_endpoints !== undefined &&
      data.performance.slow_endpoints > 0
    ) {
      score -= Math.min(data.performance.slow_endpoints * 2, 20);
    }

    if (
      data.database?.slow_query_collections !== null &&
      data.database?.slow_query_collections !== undefined &&
      data.database.slow_query_collections > 0
    ) {
      score -= Math.min(data.database.slow_query_collections * 3, 20);
    }

    return Math.max(Math.round(score), 0);
  };

  const healthScore = calculateHealthScore();

  const getErrorSparklineData = (): number[] | undefined => {
    if (errorTrends?.trends === null || errorTrends?.trends === undefined)
      return undefined;

    const trendData = Object.entries(errorTrends.trends)
      .sort(([a], [b]) => a.localeCompare(b))
      .map(([_, data]: [string, unknown]) => {
        const trendData = data as { total?: number };
        return trendData.total ?? 0;
      });

    return trendData.length > 0 ? trendData : undefined;
  };

  const metrics = [
    {
      id: 'errors',
      title: 'Errors',
      icon: FiAlertCircle,
      value: data.errors?.unresolved ?? 0,
      total: data.errors?.total ?? 0,
      label: 'unresolved',
      trend:
        data.errors?.resolution_rate !== null &&
        data.errors?.resolution_rate !== undefined &&
        data.errors.resolution_rate > 0
          ? `${(data.errors.resolution_rate * 100).toFixed(1)}% resolved`
          : undefined,
      status: (() => {
        const unresolved = data.errors?.unresolved ?? 0;
        if (unresolved === 0) return 'excellent' as const;
        if (unresolved < 5) return 'good' as const;
        if (unresolved < 10) return 'warning' as const;
        return 'critic
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity**: The primary bottleneck is the `calculateHealthScore` function, which iterates over potential data fields multiple times (e.g., checking for `null`, `undefined`). This results in a time complexity of O(n) where n is the number of data fields. Additionally, the `getErrorSparklineData` function involves sorting and mapping over an array, contributing to another O(n log n) operation.

**Space Complexity**: The space complexity is primarily influenced by the storage of intermediate values like `trendData`. While not excessively high, it could be optimized by avoiding unnecessary data structures.

**Bottlenecks or Inefficiencies**:
1. **Redundant Checks**: The function repeatedly checks for `null` and `undefined`, which can be costly.
2. **Multiple Iterations**: The `calculateHealthScore` function iterates over the same fields multiple times, leading to redundant operations.
3. **Sorting and Mapping**: The `getErrorSparklineData` function sorts and maps an array, which is not necessary if the data structure is pre-processed.

**Optimization Opportunities**:
1. **Reduce Redundant Checks**: Use optional chaining (`?.`) to simplify null/undefined checks.
2. **Memoization**: Cache results of functions like `calculateHealthScore` and `getHealthScoreLabel` using React's `useMemo`.
3. **Pre-Processing Data**: Pre-process data in the API response to avoid repeated operations.

**Resource Usage Concerns**:
1. **Unclosed Connections**: Ensure all network requests are properly managed, especially if they involve long-lived connections.
2. **Memory Leaks**: Use React’s cleanup functions (e.g., `useEffect`) to ensure resources like subscriptions or timers are cleared when components unmount.

By addressing these issues, you can significantly improve the performance and efficiency of your component.

---

*Generated by CodeWorm on 2026-02-19 21:56*
