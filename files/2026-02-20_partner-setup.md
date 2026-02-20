# partner-setup

**Type:** File Overview
**Repository:** ios-test
**File:** red-recon/app/(app)/partner-setup.tsx
**Language:** tsx
**Lines:** 1-375
**Complexity:** 0.0

---

## Source Code

```tsx
/**
 * @AngelaMos | 2026
 * partner-setup.tsx
 */

import { useCreatePartner } from '@/api/hooks'
import { CycleRegularity, type PartnerCreate } from '@/api/types'
import { DottedBackground, Input } from '@/shared/components'
import { haptics } from '@/shared/utils'
import { colors } from '@/theme/tokens'
import { router } from 'expo-router'
import { ArrowLeft, Calendar, CalendarDays, Clock, User } from 'lucide-react-native'
import type React from 'react'
import { useCallback, useMemo, useState } from 'react'
import { Pressable, ScrollView } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { Stack, Text, XStack, YStack } from 'tamagui'

const CYCLE_LENGTH_OPTIONS = [21, 24, 26, 28, 30, 32, 35] as const
const PERIOD_LENGTH_OPTIONS = [3, 4, 5, 6, 7] as const
const DAYS_AGO_OPTIONS = [0, 1, 2, 3, 4, 5, 6, 7, 10, 14, 21, 28] as const

const REGULARITY_OPTIONS = [
  { value: CycleRegularity.REGULAR, label: 'Regular', desc: 'Predictable timing' },
  { value: CycleRegularity.SOMEWHAT_IRREGULAR, label: 'Somewhat Irregular', desc: 'Varies a few days' },
  { value: CycleRegularity.IRREGULAR, label: 'Irregular', desc: 'Hard to predict' },
] as const

function NumberPicker({
  label,
  icon,
  options,
  selected,
  onChange,
  unit,
}: {
  label: string
  icon: React.ReactNode
  options: readonly number[]
  selected: number
  onChange: (value: number) => void
  unit: string
}): React.ReactElement {
  return (
    <Stack
      backgroundColor="$bgSurface100"
      borderWidth={1}
      borderColor="$borderDefault"
      borderRadius="$4"
      padding="$4"
    >
      <XStack alignItems="center" gap="$2" marginBottom="$3">
        {icon}
        <Text fontSize={14} fontWeight="500" color="$textDefault">
          {label}
        </Text>
      </XStack>
      <XStack flexWrap="wrap" gap="$2">
        {options.map((value) => (
          <Pressable
            key={value}
            onPress={() => {
              haptics.selection()
              onChange(value)
            }}
          >
            <Stack
              paddingVertical="$2"
              paddingHorizontal="$4"
              borderRadius="$3"
              borderWidth={1}
              borderColor={selected === value ? '$accent' : '$borderDefault'}
              backgroundColor={selected === value ? '$accent' : '$bgSurface200'}
              minWidth={48}
              alignItems="center"
            >
              <Text
                fontSize={14}
                color={selected === value ? '$white' : '$textLight'}
                fontWeight="500"
              >
                {value}
              </Text>
            </Stack>
          </Pressable>
        ))}
      </XStack>
      <Text fontSize={12} color="$textMuted" marginTop="$2">
        {unit}
      </Text>
    </Stack>
  )
}

function RegularityPicker({
  selected,
  onChange,
}: {
  selected: CycleRegularity
  onChange: (value: CycleRegularity) => void
}): React.ReactElement {
  return (

```

---

## File Overview

### partner-setup.tsx

**Purpose and Responsibility:**
This file is responsible for rendering the Partner Setup screen, where users can input details about their cycle to create a new partner profile. It includes components for selecting cycle length, regularity, last period start date, and other relevant information.

**Key Exports or Public Interface:**
- `NumberPicker`: A reusable component for picking numbers from a set of options.
- `RegularityPicker`: A picker for selecting the cycle regularity.
- `LastPeriodPicker`: A picker for selecting the number of days ago the last period started.

**How It Fits in the Project:**
This file is part of the user onboarding process, specifically handling the setup phase for creating a new partner profile. It integrates with other components and hooks from shared libraries to provide a consistent and interactive user experience. The data collected here is used by backend services via API calls to create a new partner record.

**Notable Design Decisions:**
- **State Management:** Uses React's `useState` and `useCallback` hooks for managing UI state.
- **Component Reusability:** Components like `NumberPicker`, `RegularityPicker`, and `LastPeriodPicker` are designed to be reusable across different parts of the application, promoting code reuse and consistency.
- **Haptics Feedback:** Utilizes haptic feedback (`haptics.selection()`) for user interaction confirmation, enhancing the user experience on mobile devices.
- **Tamagui Layouts:** Employs Tamagui's layout components like `Stack`, `XStack`, and `YStack` to create a clean and responsive UI.
```

This documentation provides an overview of the file’s purpose, key components, integration within the project, and notable design choices.

---

*Generated by CodeWorm on 2026-02-20 14:43*
