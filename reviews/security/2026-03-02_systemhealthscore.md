# SystemHealthScore

**Type:** Security Review
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

## Security Review

### Security Review for `SystemHealthScore` Component

#### Vulnerabilities and Recommendations:

1. **No Injection Vulnerabilities:**
   - The code does not contain any SQL, command, or XSS injection vulnerabilities.

2. **Authentication and Authorization Issues:**
   - No specific issues found related to authentication and authorization.

3. **Hardcoded Secrets or Credentials:**
   - No hardcoded secrets or credentials are present in the provided code snippet.

4. **Race Conditions and TOCTOU Bugs:**
   - No race conditions or TOCTOU bugs identified.

5. **Input Validation Gaps:**
   - The component does not receive external input, so input validation is not applicable here.

6. **Insecure Deserialization:**
   - No insecure deserialization issues found.

7. **Error Handling:**
   - Error handling is basic and does not leak sensitive information. However, consider logging errors securely if needed.

#### Attack Vectors:
- **XSS:** Although there are no direct inputs, ensure that all user-generated content (if any) is properly sanitized.
- **Information Disclosure:** The error message could potentially disclose system details in case of an error. Consider logging such errors to a secure server instead of displaying them directly.

#### Recommended Fixes:

1. **Error Logging:**
   - Replace direct error display with logging:
     ```tsx
     if (error !== null && error !== undefined) {
       console.error('Health Check Failed:', error);
       return (
         <div className={`${styles.healthCard} ${styles.error}`}>
           ...
         </div>
       );
     }
     ```

2. **Secure Error Handling:**
   - Implement secure logging for errors to prevent sensitive information leakage.

3. **Component Security Practices:**
   - Ensure all components follow security best practices, such as using `dangerouslySetInnerHTML` carefully and sanitizing any user inputs if they are used.

#### Overall Security Posture:
The current implementation is relatively secure but could benefit from enhanced error handling and logging practices to prevent information leakage. Regular code reviews and adherence to security guidelines will help maintain a robust security posture.

---

*Generated by CodeWorm on 2026-03-02 07:22*
