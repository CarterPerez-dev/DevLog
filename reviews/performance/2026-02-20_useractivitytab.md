# UserActivityTab

**Type:** Performance Analysis
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
        
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of the `UserActivityTab` component is primarily driven by string matching operations in `parseUserAgent`, which has a worst-case scenario of O(n) for each condition check, where n is the length of the userAgent string. The `formatTimeDifference` function involves basic arithmetic and date calculations, making it O(1). Overall, the time complexity is manageable but can be optimized.

#### Space Complexity
The space complexity is low as there are no significant memory allocations beyond a few strings and objects. However, the repeated use of string operations in `parseUserAgent` could lead to unnecessary memory usage if the userAgent string is very large.

#### Bottlenecks or Inefficiencies
1. **Redundant String Operations**: The `parseUserAgent` function performs multiple regex tests on the same string, which can be optimized by using a single regex pattern.
2. **Repeated Date Calculations**: The `formatTimeDifference` function repeatedly creates new `Date` objects and calculates time differences, which could be cached or optimized.

#### Optimization Opportunities
1. **Optimize UserAgent Parsing**:
   - Use a single regex to match multiple conditions in `parseUserAgent`.
   ```ts
   const userAgentInfo = parseUserAgent(user.activity?.user_agent ?? undefined);

   function parseUserAgent(userAgent: string | undefined): { device: string; browser: string; os: string } {
     if (!userAgent) return { device: 'Unknown', browser: 'Unknown', os: 'Unknown' };

     const regex = /(?:(Mobile|Tablet|iPad)|Chrome|Safari|Firefox|Edge|Windows|macOS|Linux|Android|iOS|iPhone|iPad)(?:.*?)(\d+)?/i;
     const match = userAgent.match(regex);

     if (!match) return { device: 'Unknown', browser: 'Unknown', os: 'Unknown' };

     let device = 'Desktop';
     let browser = 'Unknown';
     let os = 'Unknown';

     switch (match[1]) {
       case 'Mobile':
         device = 'Mobile';
         break;
       case 'Tablet':
         device = 'Tablet';
         break;
       // ... other cases
     }

     return { device, browser: match[2] ?? 'Unknown', os };
   }
   ```

2. **Cache Date Calculations**:
   - Cache the result of `formatTimeDifference` to avoid repeated calculations.

#### Resource Usage Concerns
- Ensure that all DOM elements are properly closed and no unclosed connections or file handles are used.
- Use React.memo or similar optimizations to prevent unnecessary re-renders based on props.

---

*Generated by CodeWorm on 2026-02-20 19:05*
