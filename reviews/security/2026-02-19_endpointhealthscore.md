# EndpointHealthScore

**Type:** Security Review
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
                  <FiCheckCircle
```

---

## Security Review

### Security Review for `EndpointHealthScore` Component

#### Vulnerabilities Found:

1. **Info - Input Validation Gaps:**
   - Lines 28-30 and 59-62: The code assumes that the input values are always in a valid format (e.g., numbers). However, there is no explicit validation to ensure this.
     ```tsx
     const factors = Object.entries(health.health.factors).map(
       ([key, value]) => ({
         name: formatDisplayText(key),
         value: typeof value === 'number' ? value : 0,
         weight: 25,
         status: getFactorStatus(typeof value === 'number' ? value : 0),
       }),
     );
     ```
   - **Attack Vector:** Malicious input could lead to incorrect calculations or unexpected behavior.
   
2. **Info - Error Handling that Leaks Information:**
   - Lines 18-23 and 54-67: The component handles `null` and `undefined` values by rendering messages, which is good for user experience but does not handle potential errors gracefully.
     ```tsx
     if (health?.health === null || health?.health === undefined) {
       return <div className={styles.noData}>No health data available</div>;
     }
     ```
   - **Attack Vector:** Potential information leakage about the state of the application.

#### Recommended Fixes:

1. **Input Validation:**
   - Add explicit validation for input types and values to ensure they are as expected.
     ```tsx
     const factors = Object.entries(health.health.factors).map(
       ([key, value]) => ({
         name: formatDisplayText(key),
         value: isNumber(value) ? value : 0,
         weight: 25,
         status: getFactorStatus(isNumber(value) ? value : 0),
       }),
     );
     function isNumber(val): val is number {
       return typeof val === 'number';
     }
     ```

2. **Error Handling:**
   - Improve error handling to log errors and provide generic messages.
     ```tsx
     if (health?.health === null || health?.health === undefined) {
       console.error('Health data not available');
       return <div className={styles.noData}>No health data available</div>;
     }
     ```

#### Overall Security Posture:

The component is relatively secure but could benefit from better input validation and more robust error handling. Addressing these issues will enhance the overall security posture of the application.

**Severity Ratings:**
- **Input Validation Gaps:** Info
- **Error Handling that Leaks Information:** Info

---

*Generated by CodeWorm on 2026-02-19 09:01*
