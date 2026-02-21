# ResetPassword

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

#### Time Complexity (Big O Notation)
The time complexity of this component is primarily driven by the network requests (`useVerifyPasswordResetToken` and `useResetPassword`). Both are asynchronous operations, making the overall time complexity dependent on these calls. Assuming each call has a constant time complexity \(O(1)\), the dominant factor remains the network latency.

#### Space Complexity
The space complexity is low as there are no significant data structures that grow with input size. The state management (e.g., `showPassword`, `isSuccess`) uses minimal memory, and the form validation logic does not introduce substantial overhead.

#### Bottlenecks or Inefficiencies
1. **Redundant State Management**: The `verifyQuery` and `resetMutation` states are checked multiple times in different conditions.
2. **Blocking Calls**: The `setTimeout` calls within the `useEffect` hook can be replaced with conditional navigation to avoid blocking the UI thread.

#### Optimization Opportunities
1. **Conditional Navigation**: Replace the `setTimeout` for navigation after a successful password reset with direct navigation using React Router's `useNavigate`.
   ```tsx
   if (isSuccess) {
     void navigate(AUTH_ROUTES.LOGIN);
   }
   ```
2. **Error Handling**: Simplify error handling by directly navigating to the forgot-password page in case of an error.
   ```tsx
   useEffect(() => {
     if (verifyQuery.isError) {
       void navigate('/forgot-password');
     }
   }, [verifyQuery.isError, navigate]);
   ```

#### Resource Usage Concerns
- **Unclosed Connections**: Ensure that any network requests are properly managed and do not lead to unclosed connections.
- **Memory Leaks**: Use the `useEffect` cleanup function correctly to avoid memory leaks.

By addressing these points, you can improve the performance and efficiency of the component.

---

*Generated by CodeWorm on 2026-02-20 20:56*
