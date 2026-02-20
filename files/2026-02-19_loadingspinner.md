# LoadingSpinner

**Type:** File Overview
**Repository:** kill-pr0cess.inc
**File:** frontend/src/components/UI/LoadingSpinner.tsx
**Language:** tsx
**Lines:** 1-349
**Complexity:** 0.0

---

## Source Code

```tsx
/*
 * Advanced loading spinner component providing multiple animation variants and states for asynchronous operations throughout the application.
 * I'm implementing various spinner types, sizes, and contextual loading messages that maintain the dark aesthetic while providing clear feedback during computation-intensive operations like fractal generation.
 */

import { type Component, JSX, Show, createMemo } from 'solid-js';

interface LoadingSpinnerProps {
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
  variant?: 'default' | 'pulse' | 'dots' | 'bars' | 'fractal' | 'matrix';
  color?: 'primary' | 'secondary' | 'accent' | 'white';
  message?: string;
  overlay?: boolean;
  centered?: boolean;
  className?: string;
}

export const LoadingSpinner: Component<LoadingSpinnerProps> = (props) => {
  // I'm creating responsive size classes for different contexts
  const sizeClasses = createMemo(() => {
    const sizes = {
      xs: 'w-3 h-3',
      sm: 'w-4 h-4',
      md: 'w-6 h-6',
      lg: 'w-8 h-8',
      xl: 'w-12 h-12',
    };
    return sizes[props.size || 'md'];
  });

  // I'm defining color schemes that match the dark theme
  const colorClasses = createMemo(() => {
    const colors = {
      primary: 'text-cyan-400 border-cyan-400',
      secondary: 'text-indigo-400 border-indigo-400',
      accent: 'text-purple-400 border-purple-400',
      white: 'text-white border-white',
    };
    return colors[props.color || 'primary'];
  });

  // I'm implementing different spinner variants for various contexts
  const renderSpinner = () => {
    const variant = props.variant || 'default';
    const baseClasses = `${sizeClasses()} ${colorClasses()}`;

    switch (variant) {
      case 'default':
        return (
          <div
            class={`${baseClasses} border-2 border-t-transparent rounded-full animate-spin`}
          ></div>
        );

      case 'pulse':
        return (
          <div
            class={`${baseClasses} bg-current rounded-full animate-pulse`}
          ></div>
        );

      case 'dots':
        return (
          <div class="flex space-x-1">
            {[0, 1, 2].map((i) => (
              <div
                class={`w-2 h-2 bg-current rounded-full animate-bounce ${colorClasses()}`}
                style={{ 'animation-delay': `${i * 0.1}s` }}
              ></div>
            ))}
          </div>
        );

      case 'bars':
        return (
          <div class="flex space-x-1 items-end">
            {[0, 1, 2, 3].map((i) => (
              <div
                class={`w-1 bg-current animate-pulse ${colorClasses()}`}
                style={{
                  'height': `${8 + (i % 2) * 4}px`,
                  'animation-delay': `${i * 0.15}s`,
                  'animation-duration': '0.8s',
                }}
              ></div>
            ))}
          </div>
        );

      case 'fractal':
        return (
          <div class={`${baseClasses} relative`}>
            <div class="absolute inset-0 border-2 border-
```

---

## File Overview

### Purpose and Responsibility

This file defines a highly customizable `LoadingSpinner` component for the frontend of the `kill-pr0cess.inc` application. The component provides various spinner variants, sizes, colors, and messages to enhance user feedback during asynchronous operations.

### Key Exports or Public Interface

- **Component**: `LoadingSpinner`
  - **Props**:
    - `size`: Spinner size (`xs`, `sm`, `md`, `lg`, `xl`)
    - `variant`: Spinner animation type (`default`, `pulse`, `dots`, `bars`, `fractal`, `matrix`)
    - `color`: Spinner color scheme (`primary`, `secondary`, `accent`, `white`)
    - `message`: Optional loading message
    - `overlay`: Whether to overlay the spinner with a dark backdrop
    - `centered`: Whether to center the spinner horizontally
    - `className`: Additional CSS classes

### How It Fits in the Project

The `LoadingSpinner` component is part of the UI library and is used throughout the application to provide consistent and responsive loading indicators. Its flexibility allows it to be easily integrated into various components, enhancing user experience during asynchronous operations such as data fetching or computation-intensive tasks.

### Notable Design Decisions

- **Responsive Sizes**: The component supports multiple sizes (`xs`, `sm`, `md`, `lg`, `xl`) with corresponding CSS classes.
- **Customizable Variants**: Different spinner animations are implemented to fit various contexts, including `default`, `pulse`, `dots`, `bars`, `fractal`, and `matrix`.
- **Dark Theme Compatibility**: The component maintains a dark aesthetic by using appropriate color schemes that match the application's theme.
- **Conditional Rendering**: The message is conditionally rendered based on whether it is provided in the props, allowing for clear communication to the user during long-running operations.

This design ensures that the spinner is both functional and visually consistent across the application.

---

*Generated by CodeWorm on 2026-02-19 23:15*
