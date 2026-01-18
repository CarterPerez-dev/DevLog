# EndpointHealthScore

**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/analytics/components/endpoints/EndpointHealthScore.tsx
**Language:** tsx
**Lines:** 37-268
**Complexity:** 29.0

---

## Source Code

```tsx
function EndpointHealthScore({ endpoint, days }: Props): React.JSX.Element {
  const { data: health, isLoading, isError } = useEndpointHealth(endpoint, days);

  const getFactorStatus = (score: number): 'good' | 'warning' | 'critical' => {
    if (score >= 80) return 'good';
    if (score >= 50) return 'warning';
    return 'critical';
  };

  const renderHealthFactors = (): React.JSX.Element => {
    if (health?.health === null || health?.health === undefined) {
      return <div className={styles.noData}>No health data available</div>;
    }

    const factors: HealthFactor[] = Object.entries(health.health.factors).map(
      ([key, value]) => ({
        name: formatDisplayText(key),
        value: typeof value === 'number' ? value : 0,
        weight: 25, // Equal weight for simplicity
        status: getFactorStatus(typeof value === 'number' ? value : 0),
      }),
    );

    return (
      <div className={styles.factorsList}>
        {factors.map((factor) => (
          <div
            key={factor.name}
            className={styles.factorItem}
          >
            <div className={styles.factorHeader}>
              <span className={styles.factorName}>{factor.name}</span>
              <span className={`${styles.factorScore} ${styles[factor.status]}`}>
                {factor.value.toFixed(0)}%
              </span>
            </div>
            <div className={styles.factorBar}>
              <div
                className={`${styles.factorProgress} ${styles[factor.status]}`}
                style={{ width: `${String(factor.value)}%` }}
              />
            </div>
          </div>
        ))}
      </div>
    );
  };

  const renderSLAViolations = (): React.JSX.Element | null => {
    if (health?.sla_violations === null || health?.sla_violations === undefined) {
      return null;
    }

    const violations = [
      {
        name: 'Response Time',
        violated: health.sla_violations.response_time_violated,
        icon: FiClock,
      },
      {
        name: 'Error Rate',
        violated: health.sla_violations.error_rate_violated,
        icon: FiAlertCircle,
      },
      {
        name: 'Availability',
        violated: health.sla_violations.availability_violated,
        icon: FiHeart,
      },
    ];

    return (
      <div className={styles.slaSection}>
        <h4 className={styles.sectionTitle}>SLA Compliance</h4>
        <div className={styles.slaGrid}>
          {violations.map((violation) => {
            const Icon = violation.icon;
            return (
              <div
                key={violation.name}
                className={`${styles.slaItem} ${violation.violated ? styles.violated : styles.compliant}`}
              >
                <Icon className={styles.slaIcon} />
                <span className={styles.slaName}>{violation.name}</span>
                {violation.violated ? (
                  <FiXCircle className={styles.statusIcon} />
                ) : (
                  <FiCheckCircle className={styles.statusIcon} />
                )}
              </div>
            );
          })}
        </div>
      </div>
    );
  };

  const renderScoreGauge = (): React.JSX.Element => {
    const score = health?.health?.score ?? 0;
    const scoreLabel = getHealthScoreLabel(score);
    const scoreColor = getHealthScoreColor(score);

    const gaugeData = [
      { name: 'Score', value: score, fill: scoreColor },
      { name: 'Remaining', value: 100 - score, fill: 'transparent' },
    ];

    return (
      <div className={styles.scoreGauge}>
        <ResponsiveContainer
          width="100%"
          height={160}
        >
          <PieChart>
            <Pie
              data={gaugeData}
              dataKey="value"
              cx="50%"
              cy="50%"
              startAngle={180}
              endAngle={0}
              innerRadius="60%"
              outerRadius="80%"
            >
              {gaugeData.map((entry, index) => (
                <Cell
                  key={`cell-${String(index)}`}
                  fill={entry.fill}
                  stroke="none"
                />
              ))}
            </Pie>
            <Tooltip />
          </PieChart>
        </ResponsiveContainer>
        <div className={styles.scoreOverlay}>
          <div
            className={styles.scoreValue}
            style={{ color: scoreColor }}
          >
            {score.toFixed(0)}
          </div>
          <div className={styles.scoreLabel}>{scoreLabel}</div>
        </div>
      </div>
    );
  };

  const renderMetrics = (): React.JSX.Element | null => {
    if (health?.current_metrics === null || health?.current_metrics === undefined) {
      return null;
    }

    const metrics = health.current_metrics;
    const avgResponseTime =
      typeof metrics.avg_response_time === 'number'
        ? metrics.avg_response_time
        : null;
    const errorRate =
      typeof metrics.error_rate === 'number' ? metrics.error_rate : null;
    const requestCount =
      typeof metrics.request_count === 'number' ? metrics.request_count : null;

    return (
      <div className={styles.metricsGrid}>
        {avgResponseTime !== null && (
          <div className={styles.metricItem}>
            <span className={styles.metricLabel}>Avg Response</span>
            <span className={styles.metricValue}>
              {formatResponseTime(avgResponseTime)}
            </span>
          </div>
        )}
        {errorRate !== null && (
          <div className={styles.metricItem}>
            <span className={styles.metricLabel}>Error Rate</span>
            <span className={styles.metricValue}>{errorRate.toFixed(2)}%</span>
          </div>
        )}
        {requestCount !== null && (
          <div className={styles.metricItem}>
            <span className={styles.metricLabel}>Total Requests</span>
            <span className={styles.metricValue}>
              {requestCount.toLocaleString()}
            </span>
          </div>
        )}
      </div>
    );
  };

  if (isLoading) {
    return (
      <div className={styles.loadingState}>
        <LoadingSpinner />
        <p>Analyzing endpoint health...</p>
      </div>
    );
  }

  if (isError || health === null || health === undefined) {
    return (
      <div className={styles.errorState}>
        <FiAlertCircle className={styles.errorIcon} />
        <p>Failed to load health data</p>
      </div>
    );
  }

  return (
    <div className={styles.endpointHealthScore}>
      <div className={styles.header}>
        <h3 className={styles.title}>Health Analysis</h3>
        <p className={styles.endpoint}>{endpoint}</p>
      </div>

      {renderScoreGauge()}
      {renderMetrics()}
      {renderHealthFactors()}
      {renderSLAViolations()}

      {health.incidents !== null &&
        health.incidents !== undefined &&
        health.incidents.length > 0 && (
          <div className={styles.incidentNotice}>
            <FiAlertCircle className={styles.incidentIcon} />
            <span>{health.incidents.length} recent incidents recorded</span>
          </div>
        )}
    </div>
  );
}
```

---

## Documentation

### Documentation for `EndpointHealthScore` Component

**Purpose and Behavior:**
The `EndpointHealthScore` component displays the health analysis of an endpoint over a specified number of days. It fetches health data using the `useEndpointHealth` hook, processes this data to render various metrics, factors, SLA violations, and a score gauge. The component handles loading states, errors, and renders different sections based on the availability and status of the fetched data.

**Key Implementation Details:**
- **Props:** Accepts `endpoint` (string) and `days` (number) as props.
- **Health Data Processing:** Uses `getFactorStatus` to categorize health factors into 'good', 'warning', or 'critical'.
- **Rendering Sections:** Dynamically renders sections for health factors, SLA violations, score gauge, metrics, and incident notices based on the availability of data.
- **Conditional Rendering:** Handles loading states and error conditions gracefully.

**When/Why to Use This Code:**
Use this component in scenarios where you need a comprehensive view of an endpoint's health over time. It is particularly useful for monitoring and alerting systems within administrative interfaces, providing clear visualizations and actionable insights.

**Patterns and Gotchas:**
- **Error Handling:** Ensure robust error handling by checking `isLoading` and `isError` states.
- **Data Processing:** The component processes raw data to categorize factors and violations, which can be complex. Make sure to test with various data scenarios.
- **Styling:** Custom styles are applied based on the health status, ensuring visual consistency and clarity in the UI.

This component is a prime example of how to structure complex UI components in React by breaking down functionality into smaller, reusable parts while maintaining clear state management and conditional rendering.

---

*Generated by CodeWorm on 2026-01-18 18:53*
