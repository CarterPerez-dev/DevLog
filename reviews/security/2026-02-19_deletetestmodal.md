# DeleteTestModal

**Type:** Security Review
**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/testing/components/modals/DeleteTestModal.tsx
**Language:** tsx
**Lines:** 40-320
**Complexity:** 28.0

---

## Source Code

```tsx
function DeleteTestModal({
  test,
  isOpen,
  onClose,
  onSuccess,
}: DeleteTestModalProps): React.JSX.Element | null {
  const { mutate: deleteTest, isPending: isDeleting } = useDeleteTest();

  const handleDelete = (): void => {
    if (test.testId !== null && test.testId !== undefined && test.testId > 0) {
      deleteTest(test.testId, {
        onSuccess: () => {
          if (onSuccess !== null && onSuccess !== undefined) {
            onSuccess();
          }
          onClose();
        },
      });
    }
  };

  const renderTestInfo = (): React.JSX.Element => {
    const categoryLabel = getCategoryLabel(test.category);
    const categoryColor = getCategoryColor(test.category);
    const hasData = (test.total_attempts ?? 0) > 0;

    return (
      <div className={styles.testInfo}>
        <div className={styles.testHeader}>
          <div className={styles.testTitle}>
            <FiFileText className={styles.testIcon} />
            <div className={styles.testTitleContent}>
              <h3 className={styles.title}>{test.title}</h3>
              {test.testName !== null && test.testName !== undefined && (
                <span className={styles.testName}>{test.testName}</span>
              )}
            </div>
          </div>

          <div className={styles.testMeta}>
            <span className={styles.testId}>ID: {String(test.testId)}</span>
            <span
              className={styles.categoryBadge}
              style={{ color: categoryColor, borderColor: `${categoryColor}40` }}
            >
              {categoryLabel}
            </span>
          </div>
        </div>

        <div className={styles.testStats}>
          <div className={styles.statItem}>
            <FiFileText className={styles.statIcon} />
            <span className={styles.statLabel}>Questions</span>
            <span className={styles.statValue}>{String(test.questions.length)}</span>
          </div>

          <div className={styles.statItem}>
            <FiUsers className={styles.statIcon} />
            <span className={styles.statLabel}>Attempts</span>
            <span className={styles.statValue}>
              {formatNumber(test.total_attempts ?? 0)}
            </span>
          </div>

          <div className={styles.statItem}>
            <FiTarget className={styles.statIcon} />
            <span className={styles.statLabel}>Unique Users</span>
            <span className={styles.statValue}>
              {formatNumber(test.unique_users ?? 0)}
            </span>
          </div>

          <div className={styles.statItem}>
            <FiClock className={styles.statIcon} />
            <span className={styles.statLabel}>Last Attempt</span>
            <span className={styles.statValue}>
              {formatTestTimestamp(test.last_attempted)}
            </span>
          </div>
        </div>

        {hasData && (
          <div className={styles.dataWarning}>
            <FiBarChart2 className={styles.warningIcon} />
           
```

---

## Security Review

### Security Review for `DeleteTestModal.tsx`

#### Vulnerabilities Found:

1. **Input Validation Gaps** (Medium):
   - **Line 16**: The `testId` is checked but not sanitized or validated before being passed to the `deleteTest` function.
   - **Severity: Medium**
   - **Attack Vector**: An attacker could inject malicious data into `test.testId`, potentially leading to unintended deletions.

2. **Hardcoded Secrets or Credentials** (Info):
   - No hardcoded secrets or credentials are found in the provided code snippet.

3. **Error Handling Gaps** (Low):
   - The error handling for `deleteTest` is not shown, but it could leak information if not properly managed.
   - **Severity: Low**
   - **Attack Vector**: Improper error handling might expose sensitive information to attackers.

#### Recommended Fixes:

1. **Input Validation**:
   - Ensure that `testId` is validated and sanitized before passing it to the `deleteTest` function.
     ```tsx
     if (typeof test.testId === 'number' && test.testId > 0) {
       deleteTest(test.testId, { ... });
     }
     ```

2. **Error Handling**:
   - Implement proper error handling for the `deleteTest` function to avoid information leakage.
     ```tsx
     deleteTest(test.testId, {
       onSuccess: () => {
         if (onSuccess !== null && onSuccess !== undefined) {
           onSuccess();
         }
         onClose();
       },
       onError: (error) => {
         console.error('Delete Test Error:', error);
         // Handle or log the error appropriately.
       },
     });
     ```

#### Overall Security Posture:

The code is relatively secure, but there are areas for improvement. Ensuring proper input validation and robust error handling will significantly enhance the security posture of this component. Addressing these issues will help prevent potential vulnerabilities such as injection attacks.

---

*Generated by CodeWorm on 2026-02-19 10:25*
