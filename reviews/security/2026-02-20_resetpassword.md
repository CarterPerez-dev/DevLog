# ResetPassword

**Type:** Security Review
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
        <h1 className={styles.title}>Cre
```

---

## Security Review

### Security Review for `ResetPassword` Component

#### Vulnerabilities Found:

1. **Injection Vulnerability:**
   - **Severity:** Low
   - **Line:** 58, 67
   - **Description:** The `token` parameter is directly used in the form and mutation without validation or sanitization.
   - **Attack Vector:** An attacker could inject malicious data into the token to manipulate the password reset process.

2. **Input Validation Gap:**
   - **Severity:** Medium
   - **Line:** 67, 80-83
   - **Description:** The `register` function is used with a Zod resolver but does not validate the `token` field.
   - **Attack Vector:** A user could provide an invalid or malicious token, leading to unexpected behavior.

#### Recommended Fixes:

1. **Injection Vulnerability:**
   - Validate and sanitize the `token` parameter before using it in any critical operations.
   ```tsx
   const verifyQuery = useVerifyPasswordResetToken(validateToken(token ?? ''));
   ```

2. **Input Validation Gap:**
   - Ensure that all form fields, including `token`, are properly validated.
   ```tsx
   defaultValues: {
     token: validateToken(token ?? ''),
     password: '',
     confirmPassword: ''
   },
   resolver: zodResolver(AuthSchemas.passwordResetConfirm),
   ```

#### Overall Security Posture:

The component has a moderate security posture. While the current implementation does not expose critical vulnerabilities, there are areas for improvement in input validation and token handling. Addressing these issues will enhance the overall security of the password reset functionality.

By implementing proper validation and sanitization, you can mitigate potential risks associated with user inputs and ensure that the application behaves as expected under various conditions.

---

*Generated by CodeWorm on 2026-02-20 00:45*
