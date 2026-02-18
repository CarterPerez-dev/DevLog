# auth.types

**Type:** File Overview
**Repository:** Cybersecurity-Projects
**File:** TEMPLATES/fullstack-template/frontend/src/api/types/auth.types.ts
**Language:** typescript
**Lines:** 1-141
**Complexity:** 0.0

---

## Source Code

```typescript
// ===================
// © AngelaMos | 2025
// auth.types.ts
// ===================

import { z } from 'zod'
import { PASSWORD_CONSTRAINTS } from '@/config'

export const UserRole = {
  UNKNOWN: 'unknown',
  USER: 'user',
  ADMIN: 'admin',
} as const

export type UserRole = (typeof UserRole)[keyof typeof UserRole]

export const userResponseSchema = z.object({
  id: z.string().uuid(),
  created_at: z.string().datetime(),
  updated_at: z.string().datetime().nullable(),
  email: z.string().email(),
  full_name: z.string().nullable(),
  is_active: z.boolean(),
  is_verified: z.boolean(),
  role: z.nativeEnum(UserRole),
})

export const tokenResponseSchema = z.object({
  access_token: z.string(),
  token_type: z.string(),
})

export const tokenWithUserResponseSchema = tokenResponseSchema.extend({
  user: userResponseSchema,
})

export const loginRequestSchema = z.object({
  username: z.string().email(),
  password: z
    .string()
    .min(PASSWORD_CONSTRAINTS.MIN_LENGTH)
    .max(PASSWORD_CONSTRAINTS.MAX_LENGTH),
})

export const registerRequestSchema = z.object({
  email: z.string().email(),
  password: z
    .string()
    .min(PASSWORD_CONSTRAINTS.MIN_LENGTH)
    .max(PASSWORD_CONSTRAINTS.MAX_LENGTH),
  full_name: z.string().max(255).optional(),
})

export const passwordChangeRequestSchema = z.object({
  current_password: z.string(),
  new_password: z
    .string()
    .min(PASSWORD_CONSTRAINTS.MIN_LENGTH)
    .max(PASSWORD_CONSTRAINTS.MAX_LENGTH),
})

export const logoutAllResponseSchema = z.object({
  revoked_sessions: z.number(),
})

export type UserResponse = z.infer<typeof userResponseSchema>
export type TokenResponse = z.infer<typeof tokenResponseSchema>
export type TokenWithUserResponse = z.infer<typeof tokenWithUserResponseSchema>
export type LoginRequest = z.infer<typeof loginRequestSchema>
export type RegisterRequest = z.infer<typeof registerRequestSchema>
export type PasswordChangeRequest = z.infer<typeof passwordChangeRequestSchema>
export type LogoutAllResponse = z.infer<typeof logoutAllResponseSchema>

export const isValidUserResponse = (data: unknown): data is UserResponse => {
  if (data === null || data === undefined) return false
  if (typeof data !== 'object') return false

  const result = userResponseSchema.safeParse(data)
  return result.success
}

export const isValidTokenResponse = (data: unknown): data is TokenResponse => {
  if (data === null || data === undefined) return false
  if (typeof data !== 'object') return false

  const result = tokenResponseSchema.safeParse(data)
  return result.success
}

export const isValidTokenWithUserResponse = (
  data: unknown
): data is TokenWithUserResponse => {
  if (data === null || data === undefined) return false
  if (typeof data !== 'object') return false

  const result = tokenWithUserResponseSchema.safeParse(data)
  return result.success
}

export const isValidLogoutAllResponse = (
  data: unknown
): data is LogoutAllResponse => {
  if (data === null || data === undefined) ret
```

---

## File Overview

# auth.types.ts Documentation

## Purpose and Responsibility
This TypeScript source file defines validation schemas, response types, error handling, and success messages for authentication-related operations within a full-stack application. It ensures consistent data validation and error handling across various API endpoints.

## Key Exports and Public Interface
- **Validation Schemas**: Defines `userResponseSchema`, `tokenResponseSchema`, `tokenWithUserResponseSchema`, `loginRequestSchema`, `registerRequestSchema`, `passwordChangeRequestSchema`, and `logoutAllResponseSchema` using Zod for robust data validation.
- **Response Types**: Provides typed interfaces for user, token, login request, register request, password change request, and logout response schemas.
- **Error Handling**: Implements `AuthResponseError` class to handle authentication-related errors with custom messages.
- **Messages**: Exports `AUTH_ERROR_MESSAGES` and `AUTH_SUCCESS_MESSAGES` constants for consistent error and success message handling.

## How it Fits into the Project
This file serves as a central hub for defining data validation rules, response types, and error handling mechanisms. It is imported by various components and services to ensure that all authentication-related operations adhere to predefined standards, enhancing consistency and reliability throughout the application.

## Notable Design Decisions
- **Zod Validation**: Utilizes Zod for schema validation to enforce strict type checking and data integrity.
- **Type Inference**: Uses `z.infer` to automatically derive TypeScript types from Zod schemas, reducing redundancy and improving maintainability.
- **Error Handling**: Implements a custom error class `AuthResponseError` with predefined message constants to handle authentication-related errors gracefully.

---

*Generated by CodeWorm on 2026-02-18 18:24*
