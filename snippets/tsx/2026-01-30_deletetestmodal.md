# DeleteTestModal

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
            <div className={styles.warningContent}>
              <h4 className={styles.warningTitle}>Archive Information</h4>
              <p className={styles.warningText}>
                This test has {formatNumber(test.total_attempts ?? 0)} attempts from{' '}
                {formatNumber(test.unique_users ?? 0)} unique users.
                {test.avg_score !== null && test.avg_score !== undefined && (
                  <> Average score: {formatPercentage(test.avg_score)}.</>
                )}
                {test.analytics?.completion_rate !== null &&
                  test.analytics?.completion_rate !== undefined && (
                    <>
                      {' '}
                      Completion rate:{' '}
                      {formatPercentage(test.analytics.completion_rate)}.
                    </>
                  )}
                {test.analytics?.perfect_score_rate !== null &&
                  test.analytics?.perfect_score_rate !== undefined && (
                    <>
                      {' '}
                      Perfect score rate:{' '}
                      {formatPercentage(test.analytics.perfect_score_rate)}.
                    </>
                  )}
              </p>
              <p className={styles.warningSubtext}>
                This test will be moved to the archive collection where it can be
                restored later if needed.
              </p>
            </div>
          </div>
        )}
      </div>
    );
  };

  const renderQuestionPreview = (): React.JSX.Element => {
    const sampleQuestions = test.questions.slice(0, 3);

    return (
      <div className={styles.questionsPreview}>
        <h4 className={styles.previewTitle}>
          <FiFileText className={styles.previewIcon} />
          Sample Questions ({String(test.questions.length)} total)
        </h4>

        <div className={styles.questionsList}>
          {sampleQuestions.map((question, index) => (
            <div
              key={`preview-${String(index)}`}
              className={styles.questionItem}
            >
              <span className={styles.questionNumber}>{String(index + 1)}.</span>
              <div className={styles.questionContent}>
                <p className={styles.questionText}>
                  {question.question.length > 100
                    ? `${question.question.slice(0, 100)}...`
                    : question.question}
                </p>
                <span className={styles.choicesCount}>
                  {String(question.choices.length)} choices
                </span>
              </div>
            </div>
          ))}

          {test.questions.length > 3 && (
            <div className={styles.moreQuestions}>
              <span>...and {String(test.questions.length - 3)} more questions</span>
            </div>
          )}
        </div>
      </div>
    );
  };

  const renderConfirmationMessage = (): React.JSX.Element => {
    const hasAttempts = (test.total_attempts ?? 0) > 0;

    return (
      <div className={styles.confirmationMessage}>
        <div className={styles.confirmationHeader}>
          <FiAlertTriangle className={styles.confirmationIcon} />
          <h3 className={styles.confirmationTitle}>Archive Test</h3>
        </div>

        <div className={styles.confirmationText}>
          <p className={styles.primaryMessage}>
            Are you sure you want to archive <strong>"{test.title}"</strong>?
          </p>

          <p className={styles.cautionMessage}>
            This will move the test to the archive collection. The test can be
            restored later if needed.
          </p>

          {hasAttempts && (
            <p className={styles.attemptsWarning}>
              <FiInfo className={styles.infoIcon} />
              This test has {formatNumber(test.total_attempts ?? 0)} attempt
              {(test.total_attempts ?? 0) !== 1 ? 's' : ''}. All attempt data will
              also be archived.
            </p>
          )}
        </div>
      </div>
    );
  };

  const renderModalActions = (): React.JSX.Element => (
    <div className={styles.modalActions}>
      <button
        type="button"
        onClick={onClose}
        className={styles.cancelButton}
        disabled={isDeleting}
      >
        Cancel
      </button>

      <button
        type="button"
        onClick={handleDelete}
        className={clsx(
          styles.deleteButton,
          (test.total_attempts ?? 0) > 0 && styles.dangerButton,
        )}
        disabled={isDeleting}
      >
        {isDeleting ? (
          <>
            <LoadingSpinner />
            Archiving...
          </>
        ) : (
          <>
            <FiTrash2 />
            Archive Test
          </>
        )}
      </button>
    </div>
  );

  if (!isOpen) return null;

  return (
    <div
      className={styles.modalOverlay}
      onClick={onClose}
    >
      <div
        className={styles.modalContent}
        onClick={(e): void => e.stopPropagation()}
        role="dialog"
        aria-labelledby="archive-test-title"
        aria-describedby="archive-test-description"
      >
        <div
          id="archive-test-description"
          className="sr-only"
        >
          Confirmation dialog to archive test {test.title}. This action will move the
          test to the archive collection where it can be restored later if needed.
        </div>
        <div className={styles.modalHeader}>
          <h2
            id="archive-test-title"
            className={styles.modalTitle}
          >
            <FiTrash2 className={styles.titleIcon} />
            Archive Test
          </h2>

          <button
            type="button"
            onClick={onClose}
            className={styles.closeButton}
            disabled={isDeleting}
            aria-label="Close modal"
          >
            <FiX />
          </button>
        </div>

        <div className={styles.modalBody}>
          {renderConfirmationMessage()}
          {renderTestInfo()}
          {renderQuestionPreview()}
        </div>

        {renderModalActions()}
      </div>
    </div>
  );
}
```

---

## Documentation

### Documentation for `DeleteTestModal`

**Purpose and Behavior:**
`DeleteTestModal` is a React component that displays a modal dialog for archiving a test. It includes information about the test, confirmation messages, and actions to either cancel or archive the test.

**Key Implementation Details:**
- **Props:** The component accepts `test`, `isOpen`, `onClose`, and `onSuccess` as props.
- **State Management:** Uses `useDeleteTest` hook from a custom API client to handle asynchronous deletion requests.
- **Conditional Rendering:** Renders different sections based on the test's properties, such as total attempts and unique users.
- **User Interaction:** Provides buttons for canceling or archiving the test, with appropriate state handling.

**When/Why to Use This Code:**
Use this component when you need a modal dialog in your admin interface to archive tests. It provides a clean and interactive way to manage test data, ensuring that all relevant information is displayed before making changes.

**Patterns and Gotchas:**
- **Conditional Logic:** The component uses multiple conditional checks (e.g., `test.testId !== null && test.testId !== undefined`) to ensure safe operations.
- **Performance Considerations:** The rendering of sample questions is limited to the first three, which helps in reducing unnecessary computations.
- **Accessibility:** The modal includes ARIA labels and roles for better accessibility.

---

*Generated by CodeWorm on 2026-01-30 08:02*
