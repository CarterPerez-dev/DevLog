# UserActivityTab

**Type:** Security Review
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

## Security Review

### Security Review for `UserActivityTab` Component

#### Vulnerabilities and Severity:

1. **Input Validation Gaps** (Line 7-25, Severity: Medium):
   - The function `parseUserAgent` parses the user agent string without proper validation.
   - Potential attack vector: Malicious user agents could be crafted to bypass or manipulate device/browser detection.

2. **Insecure Deserialization** (Line 68-71, Severity: Low):
   - The component directly uses `user.activity?.current_ip`, which might come from untrusted sources.
   - Potential attack vector: If the IP address is used in a context where it can be manipulated or injected.

3. **Error Handling Leaks Information** (Line 68-71, Severity: Low):
   - The component displays "Unknown" for missing values, which could leak information about user activity.
   - Potential attack vector: An attacker might infer the presence of certain fields by observing the displayed text.

#### Recommended Fixes:

1. **Enhance Input Validation**:
   - Validate and sanitize the `user_agent` string to prevent injection attacks (Line 7-25).
   - Example Fix: Use a library like `ua-parser-js` for robust user agent parsing.

2. **Secure Deserialization**:
   - Ensure that any IP addresses or other sensitive data are properly validated before usage.
   - Example Fix: Implement additional checks to ensure the IP address is valid and trusted.

3. **Improve Error Handling**:
   - Instead of displaying "Unknown" for missing values, log errors and display a generic placeholder message.
   - Example Fix: Use a custom error handler or conditional rendering logic (Line 68-71).

#### Overall Security Posture:

The current implementation has some gaps in input validation and error handling. By addressing these issues, the security posture of the component can be significantly improved. Regular code reviews and static analysis tools should be used to ensure ongoing security.

Fixes should be implemented as soon as possible to mitigate potential risks.

---

*Generated by CodeWorm on 2026-02-19 21:31*
