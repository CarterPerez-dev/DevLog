# DashboardOverview

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
        return 'critical' as const;
      })(),
      sparklineData: getErrorSparklineData(),
    },
    {
      id: 'performance',
      title: 'Performance',
      icon: FiServer,
      value: data.performance?.slow_endpoints ?? 0,
      label: 'slow endpoints',
      trend:
        data.performance?.web_vitals?.lcp !== null &&
        data.performance?.web_vitals?.lcp !== undefined &&
        typeof data.performance.web_vitals.lcp === 'object'
          ? (() => {
              const lcpValues = Object.values(data.performance.web_vitals.lcp);
              const firstValue = lcpValues[0];
              const lcpDisplay =
                typeof firstValue === 'number' ? String(firstValue) : 'N/A';
              return `LCP: ${lcpDisplay}ms`;
            })()
          : undefined,
      status: (() => {
        const slowEndpoints = data.performance?.slow_endpoints ?? 0;
        if (slowEndpoints === 0) return 'excellent' as const;
        if (slowEndpoints < 3) return 'good' as const;
        if (slowEndpoints < 5) return 'warning' as const;
        return 'critical' as const;
      })(),
    },
    {
      id: 'database',
      title: 'Database',
      icon: FiDatabase,
      value: data.database?.slow_query_collections ?? 0,
      label: 'slow queries',
      status: (() => {
        const slowQueries = data.database?.slow_query_collections ?? 0;
        if (slowQueries === 0) return 'excellent' as const;
        if (slowQueries < 2) return 'good' as const;
        if (slowQueries < 5) return 'warning' as const;
        return 'critical' as const;
      })(),
    },
    {
      id: 'health',
      title: 'System Health',
      icon: FiGlobe,
      value: healthScore,
      label: getHealthScoreLabel(healthScore),
      suffix: '%',
      status: (() => {
        if (healthScore >= 90) return 'excellent' as const;
        if (healthScore >= 70) return 'good' as const;
        if (healthScore >= 50) return 'warning' as const;
        return 'critical' as const;
      })(),
    },
  ];

  return (
    <div className={styles.dashboardOverview}>
      <div className={styles.header}>
        <div className={styles.headerContent}>
          <h2 className={styles.title}>System Overview</h2>
          <p className={styles.subtitle}>
            Real-time monitoring of system health and performance
          </p>
        </div>
        {isRefetching && (
          <div className={styles.refreshIndicator}>
            <div className={styles.refreshSpinner} />
            <span>Updating...</span>
          </div>
        )}
      </div>
      <MetricsGrid metrics={metrics} />
      <RecentActivity
        errors={data.errors}
        performance={data.performance}
        database={data.database}
        timestamp={data.timestamp}
      />
      <div className={styles.footer}>
        <span className={styles.lastUpdated}>
          Last updated: {new Date(data.timestamp).toLocaleString()}
        </span>
      </div>
    </div>
  );
}
```

---

## Documentation

### Documentation for `DashboardOverview` Component

**Purpose and Behavior:**
The `DashboardOverview` component in the CertGames-Core repository provides a real-time overview of system health, performance, and error trends. It fetches data from two custom hooks (`useDashboardOverview` and `useErrorTrends`) and displays it in a structured grid layout. The component handles loading states, errors, and calculates a health score based on various metrics.

**Key Implementation Details:**
- **Data Fetching:** Uses the `useDashboardOverview` hook to fetch system metrics and `useErrorTrends` for error trends.
- **Conditional Rendering:** Displays different UI elements based on data availability (loading, error, or empty states).
- **Health Score Calculation:** Computes a health score by analyzing errors, performance issues, and database queries. The score is used to determine the overall system status.
- **Metrics Grid:** Renders metrics in a grid format using the `MetricsGrid` component.

**When/Why to Use This Code:**
This component should be used whenever you need a comprehensive dashboard for monitoring system health and performance in real-time. It's particularly useful for administrators or operations teams who require quick insights into critical system metrics.

**Patterns and Gotchas:**
- **Custom Hooks:** The component relies on custom hooks (`useDashboardOverview` and `useErrorTrends`) to manage data fetching, which is a common pattern but requires these hooks to be well-implemented.
- **Conditional Logic:** The health score calculation involves multiple conditional checks. Ensure that the logic covers all edge cases to avoid unexpected behavior.
- **Performance Considerations:** Given the complexity (50.0 cyclomatic) and number of lines (189), performance optimization might be necessary, especially during high-frequency updates or large data sets.

---

*Generated by CodeWorm on 2026-01-18 18:16*
