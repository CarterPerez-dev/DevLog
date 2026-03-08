# UserRankCard

**Type:** Security Review
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

## Security Review

### Security Review for `UserRankCard` Component

#### Vulnerabilities and Severity:

1. **Input Validation Gaps** - **Medium**
   - Line 18: The `userPosition.rank`, `totalEntries`, and other props are directly used in calculations without validation.
   - Line 24: Potential integer overflow if `totalEntries` is very large.

2. **Error Handling Leaks Information** - **Low**
   - Lines 39-40: If `userPosition.currentAvatar` is undefined, a default avatar is shown, which does not leak sensitive information but could be considered poor practice.

#### Attack Vectors:
- An attacker could provide malicious input to `userPosition.rank` or `totalEntries`, potentially leading to incorrect calculations.
- Large values of `totalEntries` could cause unexpected behavior due to integer overflow.

#### Recommended Fixes:

1. **Input Validation**:
   - Add validation for `userPosition.rank` and `totalEntries` to ensure they are within expected ranges (e.g., `if (totalEntries > 0 && userPosition.rank >= 1)`).
   - Import a utility function like `validateNumber` from a common library.

2. **Error Handling**:
   - Ensure that error handling does not reveal sensitive information. Consider logging errors to the server rather than displaying them in the UI.

#### Overall Security Posture:

The component is generally secure but could benefit from input validation and improved error handling. Ensuring robust input validation can prevent potential issues related to unexpected data types or values. 

```tsx
const validateNumber = (value: number, min: number, max: number): boolean => {
  return value >= min && value <= max;
};

const getPercentile = (): number => {
  if (!validateNumber(totalEntries, 1, Number.MAX_SAFE_INTEGER) || 
      !validateNumber(userPosition.rank, 1, totalEntries)) return 0;
  
  return Math.round(
    ((totalEntries - userPosition.rank) / totalEntries) * 100,
  );
};
```

This review highlights the need for input validation to prevent unexpected behavior and ensures the component remains robust against potential attacks.

---

*Generated by CodeWorm on 2026-03-07 22:35*
