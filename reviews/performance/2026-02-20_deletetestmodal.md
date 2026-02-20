# DeleteTestModal

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis of `DeleteTestModal`

#### Time Complexity
The time complexity is primarily determined by the operations within `handleDelete` and the rendering logic in `renderTestInfo` and `renderQuestionPreview`. The `deleteTest` mutation call has a time complexity of O(1) assuming it's an API call. However, the rendering functions involve iterating over arrays (`test.questions.slice(0, 3)`), which is O(n) where n is the number of questions.

#### Space Complexity
The space complexity is O(n) due to the storage of `sampleQuestions` in `renderQuestionPreview`. The rest of the state and props are constant time operations. Memory allocation is minimal but could be optimized by reducing unnecessary array slices.

#### Bottlenecks or Inefficiencies
1. **Redundant Operations**: The conditional checks for `test.testId`, `test.total_attempts`, etc., in `handleDelete` and `renderTestInfo` can be simplified.
2. **Unnecessary Iterations**: `test.questions.slice(0, 3)` creates a new array slice every render, which is unnecessary if the number of questions doesn't change frequently.

#### Optimization Opportunities
1. **Simplify Conditional Checks**: Combine conditions in `handleDelete` to reduce redundant checks:
   ```tsx
   const handleDelete = (): void => {
     if (test.testId && test.total_attempts > 0) {
       deleteTest(test.testId, { onSuccess: () => { /* ... */ } });
     }
   };
   ```

2. **Memoize Arrays**: Use `React.memo` or a similar technique to memoize the `sampleQuestions` slice:
   ```tsx
   const sampleQuestions = test.questions.slice(0, 3);
   return (
     <div className={styles.questionsPreview}>
       {/* ... */}
       {sampleQuestions.map((question, index) => /* ... */)}
     </div>
   );
   ```

#### Resource Usage Concerns
- **Unclosed Connections**: Ensure that `useDeleteTest` properly handles any connections or resources.
- **Memory Leaks**: Avoid unnecessary re-renders by optimizing state and prop usage.

By addressing these points, you can improve the performance and efficiency of `DeleteTestModal`.

---

*Generated by CodeWorm on 2026-02-20 00:55*
