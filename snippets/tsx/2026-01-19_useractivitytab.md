# UserActivityTab

**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/users/components/UserDetailTabs/UserActivityTab.tsx
**Language:** tsx
**Lines:** 27-270
**Complexity:** 25.0

---

## Source Code

```tsx
function UserActivityTab({ user }: UserActivityTabProps): React.JSX.Element {
  const parseUserAgent = (
    userAgent: string | undefined,
  ): { device: string; browser: string; os: string } => {
    if (!userAgent) return { device: 'Unknown', browser: 'Unknown', os: 'Unknown' };

    let device = 'Desktop';
    let browser = 'Unknown';
    let os = 'Unknown';

    if (/Mobile/i.test(userAgent)) device = 'Mobile';
    else if (/Tablet|iPad/i.test(userAgent)) device = 'Tablet';

    if (/Chrome/i.test(userAgent) && !/Edge/i.test(userAgent)) browser = 'Chrome';
    else if (/Safari/i.test(userAgent) && !/Chrome/i.test(userAgent))
      browser = 'Safari';
    else if (/Firefox/i.test(userAgent)) browser = 'Firefox';
    else if (/Edge/i.test(userAgent)) browser = 'Edge';

    if (/Windows/i.test(userAgent)) os = 'Windows';
    else if (/Mac OS/i.test(userAgent)) os = 'macOS';
    else if (/Linux/i.test(userAgent)) os = 'Linux';
    else if (/Android/i.test(userAgent)) os = 'Android';
    else if (/iOS|iPhone|iPad/i.test(userAgent)) os = 'iOS';

    return { device, browser, os };
  };

  const userAgentInfo = parseUserAgent(user.activity?.user_agent ?? undefined);

  const getDeviceIcon = (device: string): React.JSX.Element => {
    switch (device) {
      case 'Mobile':
        return <FiSmartphone />;
      case 'Tablet':
        return <FiTablet />;
      default:
        return <FiMonitor />;
    }
  };

  const formatTimeDifference = (date: string | null | undefined): string => {
    if (!date) return 'Never';

    const now = new Date();
    const then = new Date(date);
    const diffMs = now.getTime() - then.getTime();
    const diffMins = Math.floor(diffMs / 60000);
    const diffHours = Math.floor(diffMins / 60);
    const diffDays = Math.floor(diffHours / 24);

    if (diffMins < 1) return 'Just now';
    if (diffMins < 60)
      return `${String(diffMins)} minute${diffMins > 1 ? 's' : ''} ago`;
    if (diffHours < 24)
      return `${String(diffHours)} hour${diffHours > 1 ? 's' : ''} ago`;
    if (diffDays < 30)
      return `${String(diffDays)} day${diffDays > 1 ? 's' : ''} ago`;

    return then.toLocaleDateString();
  };

  return (
    <div className={styles.activityTab}>
      <div className={styles.section}>
        <div className={styles.sectionHeader}>
          <h3>Current Status</h3>
          <div className={styles.statusBadge}>
            {user.activity?.is_online ? (
              <>
                <FiWifi className={styles.onlineIcon} />
                <span>Online</span>
              </>
            ) : (
              <>
                <FiWifiOff className={styles.offlineIcon} />
                <span>Offline</span>
              </>
            )}
          </div>
        </div>

        <div className={styles.statusGrid}>
          <div className={styles.statusCard}>
            <div className={styles.cardIcon}>
              <FiClock />
            </div>
            <div className={styles.cardContent}>
              <label>Last Login</label>
              <span className={styles.value}>
                {formatTimeDifference(user.activity?.last_login_at)}
              </span>
              {user.activity?.last_login_at && (
                <span className={styles.date}>
                  {new Date(user.activity.last_login_at).toLocaleString()}
                </span>
              )}
            </div>
          </div>

          <div className={styles.statusCard}>
            <div className={styles.cardIcon}>
              <FiActivity />
            </div>
            <div className={styles.cardContent}>
              <label>Last Seen</label>
              <span className={styles.value}>
                {formatTimeDifference(user.activity?.last_seen_at)}
              </span>
              {user.activity?.last_seen_at && (
                <span className={styles.date}>
                  {new Date(user.activity.last_seen_at).toLocaleString()}
                </span>
              )}
            </div>
          </div>

          <div className={styles.statusCard}>
            <div className={styles.cardIcon}>
              <FiMapPin />
            </div>
            <div className={styles.cardContent}>
              <label>Current IP</label>
              <span className={styles.value}>
                {user.activity?.current_ip ?? 'Unknown'}
              </span>
              <span className={styles.location}>
                {user.location?.city}, {user.location?.country}
              </span>
            </div>
          </div>

          <div className={styles.statusCard}>
            <div className={styles.cardIcon}>
              {getDeviceIcon(userAgentInfo.device)}
            </div>
            <div className={styles.cardContent}>
              <label>Device Info</label>
              <span className={styles.value}>{userAgentInfo.device}</span>
              <span className={styles.detail}>
                {userAgentInfo.browser} on {userAgentInfo.os}
              </span>
            </div>
          </div>
        </div>
      </div>

      <div className={styles.section}>
        <div className={styles.sectionHeader}>
          <h3>Activity Metrics</h3>
        </div>

        <div className={styles.metricsGrid}>
          <div className={styles.metricCard}>
            <div className={styles.metricHeader}>
              <FiCalendar />
              <span>Days Active</span>
            </div>
            <div className={styles.metricValue}>
              {user.basic_info?.days_since_signup ?? 0}
            </div>
            <div className={styles.metricLabel}>Total Days</div>
          </div>

          <div className={styles.metricCard}>
            <div className={styles.metricHeader}>
              <FiTrendingUp />
              <span>Engagement Score</span>
            </div>
            <div className={styles.metricValue}>
              {user.engagement?.score?.toFixed(1) ?? '0.0'}
            </div>
            <div className={styles.metricLabel}>Out of 100</div>
          </div>

          <div className={styles.metricCard}>
            <div className={styles.metricHeader}>
              <FiActivity />
              <span>Login Frequency</span>
            </div>
            <div className={styles.metricValue}>
              {user.engagement?.factors?.login_frequency ?? 0}
            </div>
            <div className={styles.metricLabel}>Score</div>
          </div>

          <div className={styles.metricCard}>
            <div className={styles.metricHeader}>
              <FiGlobe />
              <span>Test Activity</span>
            </div>
            <div className={styles.metricValue}>
              {user.engagement?.factors?.test_activity ?? 0}
            </div>
            <div className={styles.metricLabel}>Score</div>
          </div>
        </div>
      </div>

      <div className={styles.section}>
        <div className={styles.sectionHeader}>
          <h3>Session Information</h3>
        </div>

        <div className={styles.sessionGrid}>
          <div className={styles.sessionCard}>
            <label>User Agent</label>
            <span className={styles.userAgent}>
              {user.activity?.user_agent ?? 'No user agent data'}
            </span>
          </div>

          <div className={styles.sessionCard}>
            <label>Last Login IP</label>
            <span>{user.activity?.last_login_ip ?? 'Unknown'}</span>
          </div>

          <div className={styles.sessionCard}>
            <label>Account Age</label>
            <span>{user.basic_info?.days_since_signup ?? 0} days</span>
          </div>

          <div className={styles.sessionCard}>
            <label>Time Zone</label>
            <span>{user.location?.timezone ?? 'Unknown'}</span>
          </div>
        </div>
      </div>

      {user.platform_info?.tags?.includes('SUSPICIOUS') && (
        <div className={styles.alertSection}>
          <div className={styles.alertHeader}>
            <FiAlertCircle />
            <h3>Security Alert</h3>
          </div>
          <div className={styles.alertContent}>
            <p>This user has been flagged for suspicious activity.</p>
            <p>Please review their login patterns and activity history carefully.</p>
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## Documentation

### Documentation for `UserActivityTab` Component

**Purpose and Behavior:**
The `UserActivityTab` component displays detailed information about a user's activity, including their current status, login history, device information, engagement metrics, session data, and security alerts. It parses the user agent string to determine the device type, browser, and operating system.

**Key Implementation Details:**
- **Parsing User Agent:** The `parseUserAgent` function extracts device, browser, and OS information from the user agent string.
- **Time Formatting:** The `formatTimeDifference` function calculates the time difference between the current date and a given timestamp.
- **Conditional Rendering:** The component conditionally renders sections based on available data and includes alerts for suspicious activity.

**When/Why to Use This Code:**
This component is ideal for user detail pages where administrators need to monitor and analyze user activities. It provides comprehensive insights into user behavior, helping in identifying potential security issues or enhancing user experience by personalizing the interface.

**Patterns and Gotchas:**
- **Pattern:** The use of functional components with props and JSX for rendering complex UIs.
- **Gotcha:** Ensure that the `user` prop is properly typed (`UserActivityTabProps`) to avoid runtime errors. The component heavily relies on optional chaining, so missing data should be handled gracefully.

This component is particularly useful in scenarios where detailed user activity monitoring is required, such as in security or support roles within an application.

---

*Generated by CodeWorm on 2026-01-19 18:47*
