# partner-edit

**Type:** File Overview
**Repository:** ios-test
**File:** red-recon/app/(app)/partner-edit.tsx
**Language:** tsx
**Lines:** 1-583
**Complexity:** 0.0

---

## Source Code

```tsx
/**
 * @AngelaMos | 2026
 * partner-edit.tsx
 */

import { useDeletePartner, usePartner, useUpdatePartner } from '@/api/hooks'
import { CycleRegularity, type PartnerUpdate } from '@/api/types'
import { DottedBackground, Input } from '@/shared/components'
import { haptics } from '@/shared/utils'
import { colors } from '@/theme/tokens'
import { router } from 'expo-router'
import { ArrowLeft, Bell, Calendar, Check, Clock, Trash2, User } from 'lucide-react-native'
import type React from 'react'
import { useCallback, useEffect, useState } from 'react'
import { ActivityIndicator, Pressable, ScrollView } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { Stack, Switch, Text, XStack, YStack } from 'tamagui'

const CYCLE_LENGTH_OPTIONS = [21, 24, 26, 28, 30, 32, 35] as const
const PERIOD_LENGTH_OPTIONS = [3, 4, 5, 6, 7] as const

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
    <Stack
```

---

## File Overview

### PartnerEditScreen Component

**Purpose:**
The `partner-edit.tsx` file is responsible for rendering the partner editing screen, allowing users to update their partner's details and preferences.

**Key Exports:**
- **PartnerEditScreen**: The main component that handles the UI and state management for updating a partner's information. It uses hooks from `@/api/hooks` to fetch and update partner data.

**How it Fits in the Project:**
This file is part of the larger application that manages user partners, specifically focusing on editing existing partner details. It integrates with other components like `NumberPicker`, `RegularityPicker`, and utility functions for handling haptics and colors. The component fits into the project's flow by providing a seamless way to update critical information about partners.

**Notable Design Decisions:**
- **State Management**: Uses React state hooks (`useState`) to manage form inputs, ensuring that user changes are tracked and can be saved.
- **UI Components**: Utilizes `tamagui` for styling and layout, making the UI consistent with the application's design system. 
- **Custom Pickers**: Implements custom pickers like `NumberPicker` and `RegularityPicker` to provide a clean and intuitive way for users to select cycle lengths and regularity.
- **Haptics Feedback**: Incorporates haptic feedback using `haptics.selection()` to enhance user interaction and provide tactile feedback when selecting options.

**Dependencies:**
- React hooks (`useCallback`, `useEffect`, `useState`)
- Tamagui components (`Stack`, `Pressable`, etc.)
- Custom utility functions for colors, icons, and haptics
- API hooks for fetching and updating partner data
```

This documentation provides a high-level overview of the file's purpose, key components, integration within the project, and notable design choices.

---

*Generated by CodeWorm on 2026-02-20 15:37*
