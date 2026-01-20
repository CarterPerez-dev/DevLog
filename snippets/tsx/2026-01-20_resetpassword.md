# ResetPassword

**Repository:** CertGames-Core
**File:** frontend/user-app/src/domains/auth/ui/screens/resetPassword.tsx
**Language:** tsx
**Lines:** 31-268
**Complexity:** 15.0

---

## Source Code

```tsx
function ResetPassword(): React.ReactElement {
  const { token } = useParams<{ token: string }>();
  const navigate = useNavigate();
  const [showPassword, setShowPassword] = useState(false);
  const [showConfirmPassword, setShowConfirmPassword] = useState(false);
  const [isSuccess, setIsSuccess] = useState(false);

  const verifyQuery = useVerifyPasswordResetToken(token ?? '');
  const resetMutation = useResetPassword();

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<PasswordResetConfirmFormData>({
    resolver: zodResolver(AuthSchemas.passwordResetConfirm),
    defaultValues: {
      token: token ?? '',
    },
  });

  useEffect(() => {
    if (verifyQuery.isError) {
      const timer = setTimeout(() => {
        void navigate('/forgot-password');
      }, 5000);
      return () => clearTimeout(timer);
    }
    return undefined;
  }, [verifyQuery.isError, navigate]);

  const onSubmit = async (
    data: PasswordResetConfirmFormData,
  ): Promise<void> => {
    try {
      await resetMutation.mutateAsync({
        token: data.token,
        newPassword: data.password,
        confirmPassword: data.confirmPassword,
      });
      setIsSuccess(true);
      setTimeout(() => {
        void navigate(AUTH_ROUTES.LOGIN);
      }, 3000);
    } catch (error) {
      console.error('Password reset failed:', error);
    }
  };

  const isLoading =
    isSubmitting || resetMutation.isPending || verifyQuery.isLoading;

  if (verifyQuery.isLoading) {
    return (
      <div className={styles.resetPassword}>
        <div className={styles.loadingContainer}>
          <div className={styles.spinner} />
          <p className={styles.loadingText}>Verifying reset link...</p>
        </div>
      </div>
    );
  }

  if (verifyQuery.isError) {
    return (
      <div className={styles.resetPassword}>
        <div className={styles.errorContainer}>
          <div className={styles.errorIcon}>
            <FiX />
          </div>
          <h1 className={styles.title}>Invalid or Expired Link</h1>
          <p className={styles.errorMessage}>
            This password reset link is invalid or has expired.
          </p>
          <p className={styles.errorInfo}>
            Redirecting to password reset request page...
          </p>
        </div>
      </div>
    );
  }

  if (isSuccess) {
    return (
      <div className={styles.resetPassword}>
        <div className={styles.successContainer}>
          <div className={styles.successIcon}>
            <FiCheck />
          </div>
          <h1 className={styles.title}>Password Reset Successful!</h1>
          <p className={styles.successMessage}>
            Your password has been successfully reset.
          </p>
          <p className={styles.successInfo}>Redirecting to login page...</p>
        </div>
      </div>
    );
  }

  return (
    <div className={styles.resetPassword}>
      <div className={styles.header}>
        <h1 className={styles.title}>Create New Password</h1>
        <p className={styles.subtitle}>
          Enter your new password below. Make sure it's strong and secure.
        </p>
      </div>

      <form
        onSubmit={(e) => void handleSubmit(onSubmit)(e)}
        className={styles.form}
      >
        <input
          {...register('token')}
          type="hidden"
          value={token}
        />

        <div className={styles.fieldGroup}>
          <label
            htmlFor="password"
            className={styles.label}
          >
            <FiLock />
            New Password
          </label>
          <div className={styles.passwordField}>
            <input
              {...register('password')}
              type={showPassword ? 'text' : 'password'}
              id="password"
              className={styles.input}
              placeholder="Enter new password"
              autoComplete="new-password"
              disabled={isLoading}
            />
            <button
              type="button"
              className={styles.passwordToggle}
              onClick={() => setShowPassword(!showPassword)}
              disabled={isLoading}
            >
              {showPassword ? <FiEyeOff /> : <FiEye />}
            </button>
          </div>
          {errors.password !== null && errors.password !== undefined ? (
            <div className={styles.error}>
              <FiAlertCircle />
              {typeof errors.password.message === 'string'
                ? errors.password.message
                : 'Password is invalid'}
            </div>
          ) : null}
        </div>

        <div className={styles.fieldGroup}>
          <label
            htmlFor="confirmPassword"
            className={styles.label}
          >
            <FiLock />
            Confirm New Password
          </label>
          <div className={styles.passwordField}>
            <input
              {...register('confirmPassword')}
              type={showConfirmPassword ? 'text' : 'password'}
              id="confirmPassword"
              className={styles.input}
              placeholder="Confirm new password"
              autoComplete="new-password"
              disabled={isLoading}
            />
            <button
              type="button"
              className={styles.passwordToggle}
              onClick={() => setShowConfirmPassword(!showConfirmPassword)}
              disabled={isLoading}
            >
              {showConfirmPassword ? <FiEyeOff /> : <FiEye />}
            </button>
          </div>
          {errors.confirmPassword !== null &&
          errors.confirmPassword !== undefined ? (
            <div className={styles.error}>
              <FiAlertCircle />
              {typeof errors.confirmPassword.message === 'string'
                ? errors.confirmPassword.message
                : 'Passwords do not match'}
            </div>
          ) : null}
        </div>

        <div className={styles.passwordRequirements}>
          <p className={styles.requirementsTitle}>Password must contain:</p>
          <ul className={styles.requirementsList}>
            <li>At least 6 characters</li>
            <li>One uppercase letter</li>
            <li>One lowercase letter</li>
            <li>One number</li>
            <li>One special character</li>
          </ul>
        </div>

        <button
          type="submit"
          className={styles.submitButton}
          disabled={isLoading}
        >
          {isLoading ? 'Resetting Password...' : 'Reset Password'}
        </button>
      </form>

      {resetMutation.isError ? (
        <div className={styles.errorAlert}>
          <FiAlertCircle />
          {resetMutation.error instanceof Error
            ? resetMutation.error.message
            : 'Failed to reset password. Please try again.'}
        </div>
      ) : null}

      {verifyQuery.data !== null && verifyQuery.data !== undefined ? (
        <div className={styles.tokenInfo}>
          <p className={styles.emailInfo}>
            Resetting password for: <strong>{verifyQuery.data.email}</strong>
          </p>
        </div>
      ) : null}

      {isLoading ? (
        <div className={styles.loadingOverlay}>
          <div className={styles.spinner} />
        </div>
      ) : null}
    </div>
  );
}
```

---

## Documentation

### Documentation for `ResetPassword` Component

**Purpose and Behavior:**
The `ResetPassword` component handles the process of resetting a user's password after they have clicked on a password reset link sent via email. It includes form validation, token verification, and password reset mutation handling.

**Key Implementation Details:**
- **State Management:** Uses React state to manage form inputs (`showPassword`, `showConfirmPassword`, `isSuccess`).
- **Form Handling:** Utilizes `useForm` from Formik with Zod for validation.
- **Token Verification:** Fetches and verifies the password reset token using `useVerifyPasswordResetToken`.
- **Password Reset Mutation:** Uses `useResetPassword` to attempt resetting the user's password.
- **Conditional Rendering:** Displays different UI elements based on the state of token verification, form submission, and mutation success.

**When/Why to Use This Code:**
This component should be used whenever a user needs to reset their password. It ensures that the process is secure by validating the token and handling errors gracefully. The component can be integrated into any application where users need to recover their passwords.

**Patterns and Gotchas:**
- **Error Handling:** Ensure proper error logging and user feedback for failed resets.
- **Token Expiry:** Handle expired tokens by redirecting users to a new password reset request page.
- **Security Considerations:** Always validate and sanitize inputs to prevent security vulnerabilities like injection attacks.

---

*Generated by CodeWorm on 2026-01-20 01:18*
