# BulkOperationsPanel

**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/users/components/BulkOperationsPanel.tsx
**Language:** tsx
**Lines:** 158-551
**Complexity:** 24.0

---

## Source Code

```tsx
function BulkOperationsPanel({
  selectedUsers,
  visible,
  onClose,
}: BulkOperationsPanelProps): React.JSX.Element | null {
  const [operationForm, setOperationForm] = useState<BulkOperationForm>({
    type: 'coins',
    amount: 0,
    reason: '',
  });
  const [showResults, setShowResults] = useState(false);
  const [operationResult, setOperationResult] = useState<OperationResult | null>(
    null,
  );

  const { mutate: grantCoins, isPending: grantingCoins } = useBulkGrantCoins();
  const { mutate: grantXP, isPending: grantingXP } = useBulkGrantXP();
  const { mutate: activateSubscriptions, isPending: activatingSubscriptions } =
    useBulkActivateSubscriptions();
  const { mutate: grantAchievement, isPending: grantingAchievement } =
    useBulkGrantAchievement();
  const { mutate: addTag, isPending: addingTag } = useBulkAddTag();

  const isProcessing =
    grantingCoins ||
    grantingXP ||
    activatingSubscriptions ||
    grantingAchievement ||
    addingTag;

  const handleFormChange = (
    field: keyof BulkOperationForm,
    value: unknown,
  ): void => {
    setOperationForm((prev) => ({
      ...prev,
      [field]: value,
    }));
  };

  const handleOperationTypeChange = (type: OperationType): void => {
    setOperationForm({
      type,
      amount: type === 'coins' || type === 'xp' ? 0 : undefined,
      reason: '',
      platform: type === 'subscription' ? 'admin' : undefined,
      durationDays: type === 'subscription' ? 30 : undefined,
      achievementId: type === 'achievement' ? '' : undefined,
      tag: type === 'tag' ? 'PROMO' : undefined,
    });
  };

  const executeOperation = (): void => {
    if (selectedUsers.length === 0) return;

    const handleSuccess = (data: unknown): void => {
      setOperationResult({
        success: true,
        message: `Successfully processed ${String(selectedUsers.length)} users`,
        successCount:
          data !== null &&
          typeof data === 'object' &&
          'successCount' in data &&
          typeof data.successCount === 'number'
            ? data.successCount
            : selectedUsers.length,
        failureCount:
          data !== null &&
          typeof data === 'object' &&
          'failureCount' in data &&
          typeof data.failureCount === 'number'
            ? data.failureCount
            : 0,
        errors:
          data !== null &&
          typeof data === 'object' &&
          'errors' in data &&
          Array.isArray(data.errors)
            ? data.errors.map((e) => (typeof e === 'string' ? e : String(e)))
            : [],
      });
      setShowResults(true);
    };

    const handleError = (error: unknown): void => {
      setOperationResult({
        success: false,
        message: 'Operation failed',
        successCount: 0,
        failureCount: selectedUsers.length,
        errors: [error instanceof Error ? error.message : 'Unknown error occurred'],
      });
      setShowResults(true);
    };

    switch (operationForm.type) {
      case 'coins':
        grantCoins(
          {
            user_ids: selectedUsers,
            amount: operationForm.amount ?? 0,
            ...(operationForm.reason !== null &&
              operationForm.reason !== undefined &&
              operationForm.reason.length > 0 && { reason: operationForm.reason }),
          },
          { onSuccess: handleSuccess, onError: handleError },
        );
        break;

      case 'xp':
        grantXP(
          {
            user_ids: selectedUsers,
            amount: operationForm.amount ?? 0,
            ...(operationForm.reason !== null &&
              operationForm.reason !== undefined &&
              operationForm.reason.length > 0 && { reason: operationForm.reason }),
          },
          { onSuccess: handleSuccess, onError: handleError },
        );
        break;

      case 'subscription':
        activateSubscriptions(
          {
            user_ids: selectedUsers,
            platform: operationForm.platform ?? 'admin',
            duration_days: operationForm.durationDays ?? 30,
          },
          { onSuccess: handleSuccess, onError: handleError },
        );
        break;

      case 'achievement':
        if (operationForm.achievementId) {
          grantAchievement(
            {
              user_ids: selectedUsers,
              achievement_id: operationForm.achievementId,
            },
            { onSuccess: handleSuccess, onError: handleError },
          );
        }
        break;

      case 'tag':
        if (operationForm.tag) {
          addTag(
            {
              user_ids: selectedUsers,
              tag: operationForm.tag,
            },
            { onSuccess: handleSuccess, onError: handleError },
          );
        }
        break;
    }
  };

  const isValidOperation = (): boolean => {
    switch (operationForm.type) {
      case 'coins':
      case 'xp':
        return (
          operationForm.amount !== null &&
          operationForm.amount !== undefined &&
          operationForm.amount > 0
        );
      case 'subscription':
        return (
          operationForm.platform !== null &&
          operationForm.platform !== undefined &&
          operationForm.platform.length > 0 &&
          operationForm.durationDays !== null &&
          operationForm.durationDays !== undefined &&
          operationForm.durationDays > 0
        );
      case 'achievement':
        return (
          operationForm.achievementId !== null &&
          operationForm.achievementId !== undefined &&
          operationForm.achievementId.length > 0
        );
      case 'tag':
        return (
          operationForm.tag !== null &&
          operationForm.tag !== undefined &&
          operationForm.tag.length > 0
        );
      default:
        return false;
    }
  };

  const resetPanel = (): void => {
    setShowResults(false);
    setOperationResult(null);
    setOperationForm({
      type: 'coins',
      amount: 0,
      reason: '',
    });
  };

  const handleClose = (): void => {
    resetPanel();
    onClose();
  };

  if (!visible) return null;

  return (
    <div className={styles.panelOverlay}>
      <div className={styles.panel}>
        <div className={styles.panelHeader}>
          <div className={styles.headerContent}>
            <FiUsers className={styles.headerIcon} />
            <div>
              <h3>Bulk Operations</h3>
              <p>{selectedUsers.length} users selected</p>
            </div>
          </div>
          <button
            className={styles.closeButton}
            onClick={handleClose}
          >
            <FiX />
          </button>
        </div>

        {!showResults ? (
          <div className={styles.panelContent}>
            <div className={styles.operationSelector}>
              <label className={styles.sectionLabel}>Operation Type</label>
              <div className={styles.operationTypes}>
                <button
                  className={`${styles.operationType} ${operationForm.type === 'coins' ? styles.active : ''}`}
                  onClick={() => handleOperationTypeChange('coins')}
                >
                  <FiDollarSign />
                  Grant Coins
                </button>
                <button
                  className={`${styles.operationType} ${operationForm.type === 'xp' ? styles.active : ''}`}
                  onClick={() => handleOperationTypeChange('xp')}
                >
                  <FiTrendingUp />
                  Grant XP
                </button>
                <button
                  className={`${styles.operationType} ${operationForm.type === 'subscription' ? styles.active : ''}`}
                  onClick={() => handleOperationTypeChange('subscription')}
                >
                  <FiCreditCard />
                  Subscription
                </button>
                <button
                  className={`${styles.operationType} ${operationForm.type === 'achievement' ? styles.active : ''}`}
                  onClick={() => handleOperationTypeChange('achievement')}
                >
                  <FiAward />
                  Achievement
                </button>
                <button
                  className={`${styles.operationType} ${operationForm.type === 'tag' ? styles.active : ''}`}
                  onClick={() => handleOperationTypeChange('tag')}
                >
                  <FiTag />
                  Add Tag
                </button>
              </div>
            </div>

            {renderOperationForm(operationForm, handleFormChange)}

            <div className={styles.operationPreview}>
              <h4>Operation Preview</h4>
              <div className={styles.previewContent}>
                <p>
                  <strong>Action:</strong>{' '}
                  {operationForm.type === 'coins'
                    ? `Grant ${String(operationForm.amount ?? 0)} coins`
                    : operationForm.type === 'xp'
                      ? `Grant ${String(operationForm.amount ?? 0)} XP`
                      : operationForm.type === 'subscription'
                        ? `Activate subscription for ${String(operationForm.durationDays ?? 30)} days`
                        : operationForm.type === 'achievement'
                          ? `Grant achievement ${operationForm.achievementId ?? ''}`
                          : operationForm.type === 'tag'
                            ? `Add tag "${operationForm.tag ?? ''}"`
                            : 'No operation selected'}
                </p>
                <p>
                  <strong>Users affected:</strong> {selectedUsers.length}
                </p>
                {operationForm.reason !== null &&
                  operationForm.reason !== undefined &&
                  operationForm.reason.length > 0 && (
                    <p>
                      <strong>Reason:</strong> {operationForm.reason}
                    </p>
                  )}
              </div>
            </div>
          </div>
        ) : (
          <div className={styles.resultsContent}>
            <div
              className={`${styles.resultHeader} ${operationResult?.success === true ? styles.success : styles.error}`}
            >
              {operationResult?.success === true ? <FiCheck /> : <FiAlertCircle />}
              <h4>{operationResult?.message}</h4>
            </div>

            <div className={styles.resultStats}>
              <div className={styles.resultStat}>
                <span className={styles.statNumber}>
                  {operationResult?.successCount ?? 0}
                </span>
                <span className={styles.statLabel}>Successful</span>
              </div>
              <div className={styles.resultStat}>
                <span className={styles.statNumber}>
                  {operationResult?.failureCount ?? 0}
                </span>
                <span className={styles.statLabel}>Failed</span>
              </div>
            </div>

            {operationResult?.errors !== null &&
              operationResult?.errors !== undefined &&
              operationResult.errors.length > 0 && (
                <div className={styles.errorList}>
                  <h5>Errors:</h5>
                  <ul>
                    {operationResult.errors.map((error, index) => (
                      <li key={index}>{error}</li>
                    ))}
                  </ul>
                </div>
              )}
          </div>
        )}

        <div className={styles.panelFooter}>
          {!showResults ? (
            <>
              <button
                className={styles.cancelButton}
                onClick={handleClose}
              >
                Cancel
              </button>
              <button
                className={styles.executeButton}
                onClick={executeOperation}
                disabled={
                  !isValidOperation() || isProcessing || selectedUsers.length === 0
                }
              >
                {isProcessing ? (
                  <>
                    <div className={styles.spinner}></div>
                    Processing...
                  </>
                ) : (
                  <>
                    <FiPlay />
                    Execute Operation
                  </>
                )}
              </button>
            </>
          ) : (
            <>
              <button
                className={styles.cancelButton}
                onClick={resetPanel}
              >
                New Operation
              </button>
              <button
                className={styles.executeButton}
                onClick={handleClose}
              >
                Close
              </button>
            </>
          )}
        </div>
      </div>
    </div>
  );
}
```

---

## Documentation

### Documentation for `BulkOperationsPanel`

**Purpose and Behavior:**
The `BulkOperationsPanel` component allows administrators to perform bulk operations on selected users, such as granting coins, XP, subscriptions, achievements, or adding tags. It handles form validation, operation execution, and result display.

**Key Implementation Details:**
- **State Management:** Uses React's `useState` for managing the form state and operation results.
- **Operation Mutations:** Utilizes custom hooks (`useBulkGrantCoins`, `useBulkGrantXP`, etc.) to perform bulk operations on users.
- **Form Handling:** Implements dynamic form fields based on selected operation type using `handleFormChange` and `handleOperationTypeChange`.
- **Result Display:** Shows success or failure messages along with detailed statistics and error logs.

**When/Why to Use:**
Use this component when you need a flexible, configurable interface for performing bulk operations on users in an admin panel. It is ideal for scenarios requiring granular control over user data without manual intervention.

**Patterns and Gotchas:**
- **Validation:** Ensure that the `isValidOperation` function correctly validates all required fields before executing operations.
- **Error Handling:** The component handles both success and failure cases gracefully, displaying detailed error messages if needed.
- **Performance:** Given its complexity (24.0 cyclomatic complexity), consider breaking down larger logic into smaller reusable functions to maintain readability and manageability.

This component is a robust solution for bulk user management in an admin interface, providing a clear and user-friendly way to execute various operations on multiple users at once.

---

*Generated by CodeWorm on 2026-01-30 09:53*
