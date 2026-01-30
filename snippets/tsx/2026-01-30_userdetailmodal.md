# UserDetailModal

**Repository:** CertGames-Core
**File:** frontend/admin-app/src/modules/users/components/UserDetailModal.tsx
**Language:** tsx
**Lines:** 83-586
**Complexity:** 21.0

---

## Source Code

```tsx
function UserDetailModal({
  userId,
  open,
  onClose,
}: UserDetailModalProps): React.JSX.Element | null {
  const {
    data: user,
    isLoading: userLoading,
    error: userError,
  } = useUser(userId ?? '');

  const { data: shopData } = useShopItems();

  const generateImpersonationToken = useGenerateImpersonationToken();
  const forceLogout = useForceLogout();
  // const _sendEmailToUser = useSendEmailToUser(); // TODO: Implement email functionality
  const deleteUser = useDeleteUser();

  if (!open || !userId) return null;

  const handleQuickAction = (action: string): void => {
    if (
      userId.length === 0 ||
      user?.basic_info?.user_id === null ||
      user?.basic_info?.user_id === undefined
    )
      return;

    switch (action) {
      case 'impersonate':
        generateImpersonationToken.mutate(
          {
            userId: user.basic_info.user_id,
            tokenData: { duration_minutes: 60 },
          },
          {
            onSuccess: (data: unknown) => {
              if (
                data !== null &&
                typeof data === 'object' &&
                'impersonation_token' in data &&
                typeof (data as { impersonation_token: unknown })
                  .impersonation_token === 'string' &&
                (data as { impersonation_token: string }).impersonation_token
                  .length > 0
              ) {
                if (
                  navigator.clipboard?.writeText !== null &&
                  navigator.clipboard?.writeText !== undefined
                ) {
                  navigator.clipboard
                    .writeText(
                      (data as { impersonation_token: string }).impersonation_token,
                    )
                    .then(() => {
                      const durationMinutes =
                        typeof (data as { duration_minutes?: unknown })
                          .duration_minutes === 'number'
                          ? String(
                              (data as { duration_minutes?: unknown })
                                .duration_minutes,
                            )
                          : 'unknown';
                      toast.success(
                        `Impersonation token generated and copied to clipboard! Expires in ${durationMinutes} minutes.`,
                      );
                    })
                    .catch(() => {
                      toast.success(
                        <div>
                          <p>Impersonation token generated! (Copy manually)</p>
                          <p
                            style={{
                              fontSize: '12px',
                              wordBreak: 'break-all',
                              marginTop: '8px',
                            }}
                          >
                            {(
                              data as { impersonation_token: string }
                            ).impersonation_token.substring(0, 50)}
                            ...
                          </p>
                          <p style={{ fontSize: '10px', marginTop: '4px' }}>
                            Expires in{' '}
                            {typeof (data as { duration_minutes?: unknown })
                              .duration_minutes === 'number'
                              ? String(
                                  (data as { duration_minutes?: unknown })
                                    .duration_minutes,
                                )
                              : 'unknown'}{' '}
                            minutes
                          </p>
                        </div>,
                        { duration: 10000 },
                      );
                    });
                } else {
                  toast.success(
                    <div>
                      <p>Impersonation token generated! (Copy manually)</p>
                      <p
                        style={{
                          fontSize: '12px',
                          wordBreak: 'break-all',
                          marginTop: '8px',
                        }}
                      >
                        {(
                          data as { impersonation_token: string }
                        ).impersonation_token.substring(0, 50)}
                        ...
                      </p>
                      <p style={{ fontSize: '10px', marginTop: '4px' }}>
                        Expires in{' '}
                        {typeof (data as { duration_minutes?: unknown })
                          .duration_minutes === 'number'
                          ? String(
                              (data as { duration_minutes?: unknown })
                                .duration_minutes,
                            )
                          : 'unknown'}{' '}
                        minutes
                      </p>
                    </div>,
                    { duration: 10000 },
                  );
                }
              } else {
                toast.error(
                  'Failed to generate impersonation token - no token received',
                );
              }
            },
            onError: () => {
              toast.error('Failed to generate impersonation token');
            },
          },
        );
        break;

      case 'logout':
        forceLogout.mutate(
          {
            userId: user.basic_info.user_id,
            logoutData: { reason: 'Admin force logout' },
          },
          {
            onSuccess: (data: unknown): void => {
              const wasOnline =
                data !== null && typeof data === 'object' && 'was_online' in data
                  ? (data as { was_online: unknown }).was_online
                  : undefined;
              if (wasOnline !== undefined) {
                const wasOnlineStatus = typeof wasOnline === 'boolean' && wasOnline;
                toast.success(
                  `User ${wasOnlineStatus ? 'was online and has been' : 'was already offline but'} logged out`,
                );
              } else {
                toast.error('Failed to force logout user - no status received');
              }
            },
            onError: () => {
              toast.error('Failed to force logout user');
            },
          },
        );
        break;

      case 'edit':
        toast.info('Please navigate to the Profile tab to edit user details');
        break;

      case 'delete':
        {
          toast(`Delete user "${user.basic_info?.username ?? 'Unknown'}"?`, {
            description:
              'This action cannot be undone and will remove all user data.',
            action: {
              label: 'Delete User',
              onClick: () => {
                const reason = 'Admin deletion via user management panel';

                if (reason.length > 0) {
                  deleteUser.mutate(user.basic_info.user_id, {
                    onSuccess: (data: unknown): void => {
                      const isValidDeletion =
                        data !== null &&
                        typeof data === 'object' &&
                        ('deleted_user_id' in data || 'username' in data);
                      if (isValidDeletion) {
                        toast.success(
                          `User "${user.basic_info?.username ?? 'Unknown'}" has been permanently deleted`,
                        );
                        onClose();
                      } else {
                        toast.error(
                          'Failed to delete user - no confirmation received',
                        );
                      }
                    },
                    onError: (error: unknown): void => {
                      console.warn('Delete user error:', error);
                      toast.error(
                        'Failed to delete user - check console for details',
                      );
                    },
                  });
                }
              },
            },
          });
        }
        break;

      default:
        console.warn(`Unknown quick action: ${action} for user ${userId}`);
    }
  };

  const getStatusColor = (status: string): string => {
    switch (status) {
      case 'active':
        return styles.statusActive;
      case 'inactive':
        return styles.statusInactive;
      case 'trial':
        return styles.statusTrial;
      case 'expired':
        return styles.statusExpired;
      case 'promo':
        return styles.statusPromo;
      default:
        return styles.statusDefault;
    }
  };

  const renderLoadingOrError = (): React.JSX.Element | null => {
    if (userLoading) {
      return (
        <div className={styles.loadingContent}>
          <div className={styles.spinner}></div>
          <p>Loading user details...</p>
        </div>
      );
    }

    if (userError !== null && userError !== undefined) {
      return (
        <div className={styles.errorContent}>
          <h3>Error Loading User</h3>
          <p>Failed to load user details. Please try again.</p>
        </div>
      );
    }

    return null;
  };

  return (
    <div
      className={styles.modalOverlay}
      onClick={onClose}
    >
      <div
        className={styles.modal}
        onClick={(e) => e.stopPropagation()}
      >
        <div className={styles.modalHeader}>
          <div className={styles.headerContent}>
            {userLoading ? (
              <div className={styles.headerLoading}>
                <div className={styles.skeletonAvatar}></div>
                <div className={styles.skeletonText}>
                  <div className={styles.skeletonLine}></div>
                  <div className={styles.skeletonLine}></div>
                </div>
              </div>
            ) : user !== null && user !== undefined ? (
              <div className={styles.userHeader}>
                <div className={styles.userAvatar}>
                  {(() => {
                    const avatarUrl = resolveAvatarUrl(
                      user.profile?.current_avatar,
                      shopData,
                    );
                    const hasAvatarImage =
                      user.profile?.current_avatar !== null &&
                      user.profile?.current_avatar !== undefined &&
                      shopData !== null &&
                      shopData !== undefined;

                    return (
                      <>
                        {hasAvatarImage ? (
                          <img
                            src={avatarUrl}
                            alt={user.basic_info?.username ?? 'User Avatar'}
                            className={styles.avatarImage}
                            onError={(e) => {
                              e.currentTarget.style.display = 'none';
                              const fallback = e.currentTarget
                                .nextElementSibling as HTMLElement;
                              if (fallback !== null && fallback !== undefined)
                                fallback.style.display = 'flex';
                            }}
                          />
                        ) : null}
                        <div
                          className={styles.avatarCircle}
                          style={{
                            backgroundColor: user.profile?.name_color ?? '#4F46E5',
                            display: hasAvatarImage ? 'none' : 'flex',
                          }}
                        >
                          {user.basic_info?.username?.charAt(0)?.toUpperCase() ??
                            '?'}
                        </div>
                        {user.activity?.is_online && (
                          <div className={styles.onlineIndicator}></div>
                        )}
                      </>
                    );
                  })()}
                </div>
                <div className={styles.userInfo}>
                  <h2 className={styles.username}>
                    {user.basic_info?.username ?? 'Unknown User'}
                  </h2>
                  <p className={styles.email}>
                    {user.basic_info?.email ?? 'No email'}
                  </p>
                  <div className={styles.userMeta}>
                    <span className={styles.level}>
                      Level {user.profile?.level ?? 1}
                    </span>
                    <span className={styles.coins}>
                      {user.profile?.coins?.toLocaleString() ?? '0'} coins
                    </span>
                    <span
                      className={`${styles.subscriptionStatus} ${getStatusColor(user.subscription?.status ?? 'inactive')}`}
                    >
                      {user.subscription?.status ?? 'inactive'}
                    </span>
                  </div>
                </div>
              </div>
            ) : (
              <div className={styles.errorHeader}>
                <h2>User Not Found</h2>
              </div>
            )}
          </div>

          <div className={styles.headerActions}>
            {user !== null && user !== undefined && (
              <>
                <button
                  className={styles.quickAction}
                  onClick={() => handleQuickAction('edit')}
                  title="Edit User"
                >
                  <FiEdit />
                </button>
                <button
                  className={styles.quickAction}
                  onClick={() => handleQuickAction('impersonate')}
                  title="Impersonate User"
                >
                  <FiUserCheck />
                </button>
                <button
                  className={styles.quickAction}
                  onClick={() => handleQuickAction('logout')}
                  title="Force Logout"
                >
                  <FiLogOut />
                </button>
                <button
                  className={`${styles.quickAction} ${styles.dangerAction}`}
                  onClick={() => handleQuickAction('delete')}
                  title="Delete User"
                >
                  <FiTrash2 />
                </button>
              </>
            )}
            <button
              className={styles.closeButton}
              onClick={onClose}
              title="Close"
            >
              <FiX />
            </button>
          </div>
        </div>

        <Tabs.Root
          defaultValue="overview"
          className={styles.tabsRoot}
        >
          <Tabs.List className={styles.tabNav}>
            {tabs.map((tab) => {
              const Icon = tab.icon;
              return (
                <Tabs.Trigger
                  key={tab.key}
                  value={tab.key}
                  className={styles.tab}
                >
                  <Icon className={styles.tabIcon} />
                  <span className={styles.tabLabel}>{tab.label}</span>
                </Tabs.Trigger>
              );
            })}
          </Tabs.List>

          {userLoading ||
          (userError !== null && userError !== undefined) ||
          user === null ||
          user === undefined ? (
            <div className={styles.tabContent}>{renderLoadingOrError()}</div>
          ) : (
            <>
              <Tabs.Content
                value="overview"
                className={styles.tabContent}
              >
                <UserOverviewTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="profile"
                className={styles.tabContent}
              >
                <UserProfileTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="activity"
                className={styles.tabContent}
              >
                <UserActivityTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="subscription"
                className={styles.tabContent}
              >
                <UserSubscriptionTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="achievements"
                className={styles.tabContent}
              >
                <UserAchievementsTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="shop"
                className={styles.tabContent}
              >
                <UserShopTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="game_stats"
                className={styles.tabContent}
              >
                <UserGameStatsTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="platform"
                className={styles.tabContent}
              >
                <UserPlatformTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="freemium"
                className={styles.tabContent}
              >
                <UserFreemiumTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="location"
                className={styles.tabContent}
              >
                <UserLocationTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="admin"
                className={styles.tabContent}
              >
                <UserAdminTab user={user} />
              </Tabs.Content>
              <Tabs.Content
                value="management"
                className={styles.tabContent}
              >
                <UserManagementTab user={user} />
              </Tabs.Content>
            </>
          )}
        </Tabs.Root>
      </div>
    </div>
  );
}
```

---

## Documentation

### Documentation for `UserDetailModal`

**Purpose and Behavior:**
The `UserDetailModal` component displays detailed information about a user, including their profile, activity, subscriptions, and more. It includes functionality to impersonate, force logout, edit, and delete the user. The modal can be opened or closed via props, and it dynamically loads data using hooks.

**Key Implementation Details:**
- Utilizes React hooks like `useUser`, `useShopItems`, `useGenerateImpersonationToken`, `useForceLogout`, and `useDeleteUser` to fetch and manipulate user data.
- Implements a switch-case structure in the `handleQuickAction` function to handle different quick actions (impersonate, logout, edit, delete).
- Uses conditional rendering based on loading states (`userLoading`) and errors (`userError`).

**When/Why to Use:**
This component is ideal for user management interfaces where detailed information about a specific user needs to be displayed. It can be used in admin panels or any part of the application requiring granular control over user actions.

**Patterns and Gotchas:**
- The `handleQuickAction` function handles various user actions with robust error handling.
- The `renderLoadingOrError` function ensures smooth state transitions during data fetching.
- Be cautious when implementing the delete functionality, as it is irreversible. Ensure proper confirmation steps are in place to prevent accidental deletions.

This component provides a comprehensive view of a user and offers administrative controls, making it essential for managing users within an application.

---

*Generated by CodeWorm on 2026-01-30 09:04*
