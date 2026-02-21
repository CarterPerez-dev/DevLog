# current-activity

**Type:** File Overview
**Repository:** CodeWorm
**File:** dashboard/frontend/src/components/current-activity/current-activity.tsx
**Language:** tsx
**Lines:** 1-114
**Complexity:** 0.0

---

## Source Code

```tsx
// ©AngelaMos | 2026
// current-activity.tsx

import { useEffect, useState } from 'react'
import { useDashboardStore } from '@/core/lib'
import styles from './current-activity.module.scss'

const STATUS_STYLE: Record<string, string> = {
  analyzing: styles.analyzing,
  generating: styles.generating,
  committing: styles.committing,
}

function formatDocType(dt: string): string {
  return dt
    .replace(/_/g, ' ')
    .replace(/\b\w/g, (c) => c.toUpperCase())
}

function formatCountdown(targetIso: string): string | null {
  const target = new Date(targetIso).getTime()
  const now = Date.now()
  const diff = target - now

  if (diff <= 0) return null

  const totalSec = Math.floor(diff / 1000)
  const hours = Math.floor(totalSec / 3600)
  const minutes = Math.floor((totalSec % 3600) / 60)
  const seconds = totalSec % 60

  if (hours > 0) {
    return `${hours}h ${minutes}m ${seconds}s`
  }
  if (minutes > 0) {
    return `${minutes}m ${seconds}s`
  }
  return `${seconds}s`
}

export function CurrentActivity(): React.ReactElement {
  const {
    currentActivity,
    currentTarget,
    currentRepo,
    currentDocType,
    nextCycleTime,
  } = useDashboardStore()
  const [countdown, setCountdown] = useState<string | null>(null)

  const isIdle = currentActivity === 'idle'

  useEffect(() => {
    if (!isIdle || !nextCycleTime) {
      setCountdown(null)
      return
    }

    const tick = () => {
      const result = formatCountdown(nextCycleTime)
      setCountdown(result)
    }

    tick()
    const interval = setInterval(tick, 1000)
    return () => clearInterval(interval)
  }, [isIdle, nextCycleTime])

  return (
    <div className={styles.container}>
      <div className={styles.header}>
        <span className={styles.title}>Current Activity</span>
        <span
          className={`${styles.statusBadge} ${STATUS_STYLE[currentActivity] ?? ''}`}
        >
          {currentActivity}
        </span>
      </div>

      {isIdle ? (
        <div className={styles.idle}>
          <span className={styles.idleDot} />
          {countdown
            ? `Next cycle in ${countdown}`
            : 'Waiting for next cycle...'}
        </div>
      ) : (
        <div className={styles.details}>
          {currentRepo && (
            <div className={styles.row}>
              <span className={styles.rowLabel}>Repo</span>
              <span className={styles.rowValue}>{currentRepo}</span>
            </div>
          )}
          {currentTarget && (
            <div className={styles.row}>
              <span className={styles.rowLabel}>Target</span>
              <span className={styles.rowValue}>{currentTarget}</span>
            </div>
          )}
          {currentDocType && (
            <div className={styles.row}>
              <span className={styles.rowLabel}>Type</span>
              <span className={styles.docTypeBadge}>
                {formatDocType(currentDocType)}
              </span>
            </div>
          )}
        </div>
      
```

---

## File Overview

# current-activity.tsx Documentation

## Purpose and Responsibility
This file defines a React component named `CurrentActivity`, which displays the current activity status, countdown to the next cycle, and relevant details such as repository, target, and document type in the CodeWorm dashboard. It integrates with the core store to fetch necessary state data.

## Key Exports or Public Interface
- **`CurrentActivity`:** A functional React component that renders the current activity status and related information.

## How it Fits into the Project
The `CurrentActivity` component is a key part of the CodeWorm frontend, specifically designed for the dashboard section. It leverages the `useDashboardStore` hook to access state data and updates its countdown display using an effect hook that runs every second. This component ensures users are always informed about their current activity status and when the next cycle will start.

## Notable Design Decisions
- **State Management:** Utilizes React's `useState` and `useEffect` hooks for dynamic content updates, ensuring real-time data reflection.
- **Conditional Rendering:** Implements conditional rendering based on the current activity state (`idle`, `analyzing`, etc.), providing a clear visual representation of the application’s status.
- **Countdown Logic:** Uses a custom function to format countdowns, making it easier to handle different time intervals and display them in a user-friendly manner.
- **Styling:** Leverages CSS modules for styling, ensuring consistent and maintainable styles across components.
```

This documentation provides an overview of the file's role within the project, its key exports, how it integrates with other parts, and notable design choices.

---

*Generated by CodeWorm on 2026-02-21 08:39*
