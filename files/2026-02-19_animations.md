# animations

**Type:** File Overview
**Repository:** kill-pr0cess.inc
**File:** frontend/src/utils/animations.ts
**Language:** typescript
**Lines:** 1-411
**Complexity:** 0.0

---

## Source Code

```typescript
/*
 * ©AngelaMos | 2025
 */

interface AnimationConfig {
  duration: number;
  easing: string;
  delay?: number;
  fill?: 'forwards' | 'backwards' | 'both' | 'none';
}

interface SpringConfig {
  tension: number;
  friction: number;
  mass?: number;
}

export const fadeInUp = (
  element: HTMLElement,
  config: Partial<AnimationConfig> = {},
) => {
  const defaultConfig: AnimationConfig = {
    duration: 800,
    easing: 'cubic-bezier(0.4, 0, 0.2, 1)',
    delay: 0,
    fill: 'forwards',
  };

  const finalConfig = { ...defaultConfig, ...config };

  const keyframes = [
    {
      opacity: 0,
      transform: 'translateY(20px)',
      filter: 'blur(2px)',
    },
    {
      opacity: 1,
      transform: 'translateY(0px)',
      filter: 'blur(0px)',
    },
  ];

  return element.animate(keyframes, {
    duration: finalConfig.duration,
    easing: finalConfig.easing,
    delay: finalConfig.delay,
    fill: finalConfig.fill,
  });
};

// I'm implementing staggered animations for list items and grids
export const staggeredFadeIn = (
  elements: HTMLElement[],
  staggerDelay: number = 100,
) => {
  return elements.map((element, index) =>
    fadeInUp(element, { delay: index * staggerDelay }),
  );
};

// I'm creating eerie glow effects for interactive elements
export const pulseGlow = (
  element: HTMLElement,
  color: string = '#22d3ee',
) => {
  const keyframes = [
    {
      boxShadow: `0 0 0 0 ${color}00`,
      transform: 'scale(1)',
    },
    {
      boxShadow: `0 0 0 8px ${color}40`,
      transform: 'scale(1.02)',
    },
    {
      boxShadow: `0 0 0 16px ${color}00`,
      transform: 'scale(1)',
    },
  ];

  return element.animate(keyframes, {
    duration: 2000,
    easing: 'ease-in-out',
    iterations: Infinity,
  });
};

// I'm implementing smooth morphing transitions
export const morphTransition = (
  element: HTMLElement,
  fromState: Partial<CSSStyleDeclaration>,
  toState: Partial<CSSStyleDeclaration>,
  config: Partial<AnimationConfig> = {},
) => {
  const defaultConfig: AnimationConfig = {
    duration: 400,
    easing: 'cubic-bezier(0.4, 0, 0.2, 1)',
    fill: 'forwards',
  };

  const finalConfig = { ...defaultConfig, ...config };

  // I'm building keyframes from the state objects
  const keyframes = [
    Object.fromEntries(
      Object.entries(fromState).map(([key, value]) => [key, value]),
    ),
    Object.fromEntries(
      Object.entries(toState).map(([key, value]) => [key, value]),
    ),
  ];

  return element.animate(keyframes, finalConfig);
};

// I'm creating typing animation for code/terminal effects
export const typewriterEffect = async (
  element: HTMLElement,
  text: string,
  speed: number = 50,
  cursor: boolean = true,
) => {
  element.textContent = '';

  if (cursor) {
    element.style.borderRight = '2px solid #22d3ee';
    element.style.animation = 'blink 1s infinite';
  }

  for (let i = 0; i <= text.length; i++) {
    element.textContent = text.slice(0, i);
    await new Promise((resolve) => setTimeou
```

---

## File Overview

### Purpose and Responsibility

This TypeScript source file, `animations.ts`, is responsible for providing a suite of animation utilities to enhance user interactions and visual effects within the frontend application. It includes functions for various animations such as fades, pulses, morphing transitions, typing effects, parallax scrolling, spring animations, and digital rain.

### Key Exports and Public Interface

- **fadeInUp**: A fade-in animation with an upward movement.
- **staggeredFadeIn**: Applies a staggered fadeIn effect to multiple elements.
- **pulseGlow**: Creates an eerie glow effect around interactive elements.
- **morphTransition**: Implements smooth morphing transitions between two states.
- **typewriterEffect**: Adds a typewriter-like text typing animation.
- **createParallaxScroll**: Creates parallax scroll effects for depth perception.
- **springAnimation**: Provides spring-based animations for interactive feedback.
- **createMatrixRain**: Generates a matrix-style digital rain effect.

### How It Fits in the Project

These animation utilities are crucial for enhancing user experience and visual appeal across various components of the frontend application. They are used throughout the project to add dynamic interactions, such as on button clicks or page transitions, making the application more engaging and responsive.

### Notable Design Decisions

- **Modular Functions**: Each function is designed to be modular and reusable, allowing for easy integration into different parts of the application.
- **Default Configurations**: Default configurations are provided for each animation to ensure consistent behavior while also supporting customization.
- **Performance Optimization**: Techniques like requestAnimationFrame are used in animations to optimize performance and smoothness.
- **Type Safety**: TypeScript interfaces are utilized to enforce type safety, ensuring that only valid configurations are passed to the functions.

---

*Generated by CodeWorm on 2026-02-19 23:06*
