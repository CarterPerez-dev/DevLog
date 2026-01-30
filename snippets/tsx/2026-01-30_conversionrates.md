# ConversionRates

**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/revenue/components/conversion/ConversionRates.tsx
**Language:** tsx
**Lines:** 47-324
**Complexity:** 26.0

---

## Source Code

```tsx
function ConversionRates({
  data,
  dimension,
  isLoading,
  error,
}: ConversionRatesProps): React.JSX.Element {
  const getPlatformIcon = (platform: string): React.JSX.Element => {
    const lowerPlatform = platform.toLowerCase();
    if (lowerPlatform === 'web') {
      return <FiMonitor />;
    }
    if (lowerPlatform === 'ios' || lowerPlatform === 'android') {
      return <FiSmartphone />;
    }
    return <FiMonitor />;
  };

  const renderPlatformBreakdown = (): React.JSX.Element | null => {
    if (data === null || data === undefined || dimension !== 'platform') return null;

    const breakdown = data.breakdown;
    if (
      breakdown === null ||
      breakdown === undefined ||
      typeof breakdown !== 'object'
    ) {
      return null;
    }

    const platforms = Object.entries(breakdown)
      .map(([platform, metrics]) => {
        if (
          metrics !== null &&
          metrics !== undefined &&
          typeof metrics === 'object'
        ) {
          const platformData = metrics as Record<string, unknown>;
          return {
            platform,
            total_users: Number(platformData.total_users ?? 0),
            converted_users: Number(platformData.converted_users ?? 0),
            conversion_rate: Number(platformData.conversion_rate ?? 0),
          } as PlatformData;
        }
        return null;
      })
      .filter((item): item is PlatformData => item !== null);

    if (platforms.length === 0) {
      return (
        <div className={styles.emptyState}>
          <FiUsers />
          <span>No platform data available</span>
        </div>
      );
    }

    return (
      <div className={styles.platformGrid}>
        {platforms.map((platform) => (
          <div
            key={platform.platform}
            className={styles.platformCard}
          >
            <div className={styles.platformHeader}>
              <div className={styles.platformIcon}>
                {getPlatformIcon(platform.platform)}
              </div>
              <span className={styles.platformName}>
                {formatDisplayText(platform.platform)}
              </span>
            </div>

            <div className={styles.platformMetrics}>
              <div className={styles.metric}>
                <span className={styles.metricValue}>
                  {formatPercentage(platform.conversion_rate)}
                </span>
                <span className={styles.metricLabel}>Conversion Rate</span>
              </div>

              <div className={styles.metricDivider} />

              <div className={styles.metricStats}>
                <div className={styles.stat}>
                  <span className={styles.statValue}>
                    {formatNumber(platform.converted_users)}
                  </span>
                  <span className={styles.statLabel}>Converted</span>
                </div>
                <div className={styles.stat}>
                  <span className={styles.statValue}>
                    {formatNumber(platform.total_users)}
                  </span>
                  <span className={styles.statLabel}>Total Users</span>
                </div>
              </div>
            </div>

            <div className={styles.platformBar}>
              <div
                className={styles.platformBarFill}
                style={{
                  width: `${String(Math.min(platform.conversion_rate, 100))}%`,
                }}
              />
            </div>
          </div>
        ))}
      </div>
    );
  };

  const renderTimeToConvertBreakdown = (): React.JSX.Element | null => {
    if (data === null || data === undefined || dimension !== 'days_since_signup')
      return null;

    const breakdown = data.breakdown;
    if (
      breakdown === null ||
      breakdown === undefined ||
      typeof breakdown !== 'object'
    ) {
      return null;
    }

    const timeRanges = Object.entries(breakdown)
      .map(([range, metrics]) => {
        if (
          metrics !== null &&
          metrics !== undefined &&
          typeof metrics === 'object'
        ) {
          const rangeData = metrics as Record<string, unknown>;
          return {
            range,
            total_users: Number(rangeData.total_users ?? 0),
            converted_users: Number(rangeData.converted_users ?? 0),
            conversion_rate: Number(rangeData.conversion_rate ?? 0),
          } as TimeToConvertData;
        }
        return null;
      })
      .filter((item): item is TimeToConvertData => item !== null);

    if (timeRanges.length === 0) {
      return (
        <div className={styles.emptyState}>
          <FiClock />
          <span>No time-to-convert data available</span>
        </div>
      );
    }

    const sortedRanges = [...timeRanges].sort((a, b) => {
      const getDays = (range: string): number => {
        const match = /\d+/.exec(range);
        if (match !== null && match !== undefined) {
          return parseInt(match[0]);
        }
        return 0;
      };
      return getDays(a.range) - getDays(b.range);
    });

    return (
      <div className={styles.timeGrid}>
        {sortedRanges.map((range) => (
          <div
            key={range.range}
            className={styles.timeCard}
          >
            <div className={styles.timeHeader}>
              <FiClock className={styles.timeIcon} />
              <span className={styles.timeRange}>
                {formatDisplayText(range.range)}
              </span>
            </div>

            <div className={styles.timeMetrics}>
              <div className={styles.conversionRate}>
                <FiPercent className={styles.rateIcon} />
                <span className={styles.rateValue}>
                  {formatPercentage(range.conversion_rate)}
                </span>
              </div>

              <div className={styles.userCounts}>
                <span className={styles.convertedCount}>
                  {formatNumber(range.converted_users)} converted
                </span>
                <span className={styles.totalCount}>
                  of {formatNumber(range.total_users)} users
                </span>
              </div>
            </div>

            <div className={styles.timeBar}>
              <div
                className={styles.timeBarFill}
                style={{
                  width: `${String(Math.min(range.conversion_rate, 100))}%`,
                  animationDelay: `${String(sortedRanges.indexOf(range) * 100)}ms`,
                }}
              />
            </div>
          </div>
        ))}
      </div>
    );
  };

  const renderOverallMetrics = (): React.JSX.Element | null => {
    if (data === null || data === undefined) return null;

    return (
      <div className={styles.overallMetrics}>
        <div className={styles.overallCard}>
          <span className={styles.overallLabel}>Overall Conversion Rate</span>
          <span className={styles.overallValue}>
            {formatPercentage(data.overall_conversion_rate)}
          </span>
        </div>
        <div className={styles.overallCard}>
          <span className={styles.overallLabel}>Total Users</span>
          <span className={styles.overallValue}>
            {formatNumber(data.total_users)}
          </span>
        </div>
        <div className={styles.overallCard}>
          <span className={styles.overallLabel}>Converted Users</span>
          <span className={styles.overallValue}>
            {formatNumber(data.converted_users)}
          </span>
        </div>
      </div>
    );
  };

  if (error !== null && error !== undefined) {
    return (
      <div className={styles.errorState}>
        <span className={styles.errorMessage}>
          {error instanceof Error
            ? error.message
            : 'Failed to load conversion rates'}
        </span>
      </div>
    );
  }

  if (isLoading) {
    return (
      <div className={styles.loadingState}>
        <div className={styles.spinner} />
        <span className={styles.loadingText}>Loading conversion rates...</span>
      </div>
    );
  }

  if (data === null || data === undefined) {
    return (
      <div className={styles.emptyState}>
        <FiUsers />
        <span>No conversion data available</span>
      </div>
    );
  }

  return (
    <div className={styles.conversionRates}>
      {renderOverallMetrics()}

      {dimension === 'platform' && renderPlatformBreakdown()}
      {dimension === 'days_since_signup' && renderTimeToConvertBreakdown()}
    </div>
  );
}
```

---

## Documentation

### Documentation for `ConversionRates` Component

**Purpose and Behavior:**
The `ConversionRates` component renders various metrics related to conversion rates based on the provided data and dimension. It handles different dimensions such as 'platform' and 'days_since_signup', displaying platform-specific breakdowns, time-to-convert metrics, and overall conversion statistics.

**Key Implementation Details:**
1. **Props**: Accepts `data`, `dimension`, `isLoading`, and `error` props.
2. **Rendering Logic**: Conditionally renders different sections based on the provided data and dimension. It includes:
   - Platform-specific breakdowns using icons and metrics.
   - Time-to-convert metrics with animated bars.
   - Overall conversion rate statistics.

**When/Why to Use:**
Use this component in admin or analytics dashboards where detailed conversion rate analysis is required. It's particularly useful for tracking user engagement across different platforms and time periods, providing insights into user behavior and conversion rates.

**Patterns & Gotchas:**
- **Conditional Rendering**: The component conditionally renders based on the `isLoading`, `error`, and `dimension` props.
- **Data Handling**: Ensure data is properly structured to avoid null or undefined errors. Use type assertions carefully when converting data types.
- **Animation Delays**: Time-to-convert metrics use CSS animations, which require careful handling of delays to ensure smooth transitions.

This component is a robust solution for displaying complex conversion rate metrics in a clear and user-friendly manner.

---

*Generated by CodeWorm on 2026-01-30 07:25*
