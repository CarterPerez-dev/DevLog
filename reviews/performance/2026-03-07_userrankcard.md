# UserRankCard

**Type:** Performance Analysis
**Repository:** CertGames-Core
**File:** frontend/user-app/src/domains/leaderboard/ui/components/userRankCard.tsx
**Language:** tsx
**Lines:** 19-129
**Complexity:** 13.0

---

## Source Code

```tsx
function UserRankCard({
  userPosition,
  totalEntries,
}: UserRankCardProps): React.ReactElement {
  const { isSimple } = useTheme();
  const s = isSimple ? simpleStyles : defaultStyles;
  const getPercentile = (): number => {
    if (totalEntries === 0) return 0;
    return Math.round(
      ((totalEntries - userPosition.rank) / totalEntries) * 100,
    );
  };

  const getRankTier = (): string => {
    const rank = userPosition.rank;
    if (rank <= 3) return 'ELITE';
    if (rank <= 10) return 'TOP 10';
    if (rank <= 50) return 'TOP 50';
    if (rank <= 100) return 'TOP 100';
    if (rank <= 500) return 'TOP 500';
    return 'RANKED';
  };

  const getRankColor = (): string => {
    const rank = userPosition.rank;
    if (rank === 1) return s.gold;
    if (rank === 2) return s.silver;
    if (rank === 3) return s.bronze;
    if (rank <= 10) return s.purple;
    if (rank <= 50) return s.blue;
    return '';
  };

  return (
    <div className={s.userRankCard}>
      <div className={s.header}>
        <div className={s.titleWrapper}>
          <GiRank3 className={s.icon} />
          <span>YOUR POSITION</span>
        </div>
        <div className={`${s.tierBadge} ${getRankColor()}`}>
          {getRankTier()}
        </div>
      </div>

      <div className={s.content}>
        <div className={s.rankSection}>
          <div className={s.avatarWrapper}>
            {userPosition.currentAvatar !== null &&
            userPosition.currentAvatar !== undefined ? (
              <img
                src={userPosition.currentAvatar}
                alt={userPosition.username}
                className={s.avatar}
              />
            ) : (
              <div className={s.defaultAvatar}>
                <GiSpy />
              </div>
            )}
          </div>

          <div className={s.userInfo}>
            <div className={s.username}>{userPosition.username}</div>
            <div className={s.role}>{userPosition.role}</div>
          </div>

          <div className={`${s.rankNumber} ${getRankColor()}`}>
            #{userPosition.rank}
          </div>
        </div>

        <div className={s.statsGrid}>
          <div className={s.statItem}>
            <span className={s.statLabel}>LEVEL</span>
            <span className={s.statValue}>{userPosition.level}</span>
          </div>

          <div className={s.statItem}>
            <span className={s.statLabel}>XP</span>
            <span className={s.statValue}>
              {formatLargeNumber(userPosition.xp)}
            </span>
          </div>

          <div className={s.statItem}>
            <span className={s.statLabel}>PERCENTILE</span>
            <span className={s.statValue}>
              TOP {100 - getPercentile()}%
            </span>
          </div>
        </div>

        <div className={s.progressSection}>
          <div className={s.progressHeader}>
            <span className={s.progressLabel}>GLOBAL RANKING</span>
            <span className={s.progressVa
```

---

## Performance Analysis

### Performance Analysis of `UserRankCard`

#### Time Complexity (Big O Notation)
The time complexity for this component is **O(1)**, as the operations performed are constant and do not depend on the size of the input data.

#### Space Complexity and Memory Allocation
- The space complexity is also **O(1)** since no additional memory is allocated that scales with input size.
- However, there are a few areas where optimization can be applied to reduce unnecessary computations.

#### Bottlenecks or Inefficiencies
1. **Redundant Calculations**: `getPercentile` and `getRankColor` functions are called multiple times within the JSX rendering. This leads to redundant calculations that could be optimized.
2. **String Concatenation in JSX**: The use of template literals for class names can be simplified, reducing string concatenation overhead.

#### Optimization Opportunities
1. **Memoize Calculations**: Use `React.memo` or a similar technique to memoize the results of `getPercentile` and `getRankColor`, ensuring they are only recalculated when necessary.
   ```tsx
   const percentile = useMemo(() => getPercentile(), [userPosition.rank, totalEntries]);
   ```
2. **Simplify Class Names**: Combine class names in JSX using template literals to reduce string concatenation:
   ```jsx
   <div className={`${s.tierBadge} ${getRankColor()}`}>
     {getRankTier()}
   </div>
   ```

#### Resource Usage Concerns
- Ensure that `userPosition.currentAvatar` is properly handled to avoid unnecessary image loading.
- Use lazy loading techniques for images if the component is used frequently.

By applying these optimizations, you can improve the performance and reduce redundant calculations in your React component.

---

*Generated by CodeWorm on 2026-03-07 23:04*
