# query.config

**Type:** File Overview
**Repository:** fullstack-template
**File:** frontends/react-native-ios/src/core/api/query.config.ts
**Language:** typescript
**Lines:** 1-117
**Complexity:** 0.0

---

## Source Code

```typescript
// ===================
// © AngelaMos | 2026
// query.config.ts
// ===================

import { createAsyncStoragePersister } from '@tanstack/query-async-storage-persister'
import { MutationCache, QueryCache, QueryClient } from '@tanstack/react-query'
import { QUERY_CONFIG } from '@/core/config'
import { queryClientStorage } from '@/core/storage'
import { ApiError, ApiErrorCode } from './errors'

const NO_RETRY_ERROR_CODES: readonly ApiErrorCode[] = [
  ApiErrorCode.AUTHENTICATION_ERROR,
  ApiErrorCode.AUTHORIZATION_ERROR,
  ApiErrorCode.NOT_FOUND,
  ApiErrorCode.VALIDATION_ERROR,
] as const

const shouldRetryQuery = (failureCount: number, error: Error): boolean => {
  if (error instanceof ApiError) {
    if (NO_RETRY_ERROR_CODES.includes(error.code)) {
      return false
    }
  }
  return failureCount < QUERY_CONFIG.RETRY.DEFAULT
}

const calculateRetryDelay = (attemptIndex: number): number => {
  const baseDelay = 1000
  const maxDelay = 30000
  return Math.min(baseDelay * 2 ** attemptIndex, maxDelay)
}

let showToast: ((message: string) => void) | null = null

export const setToastHandler = (handler: (message: string) => void): void => {
  showToast = handler
}

const handleQueryCacheError = (
  error: Error,
  query: { state: { data: unknown } }
): void => {
  if (query.state.data !== undefined) {
    const message =
      error instanceof ApiError
        ? error.getUserMessage()
        : 'Background update failed'
    showToast?.(message)
  }
}

const handleMutationCacheError = (
  error: Error,
  _variables: unknown,
  _context: unknown,
  mutation: { options: { onError?: unknown } }
): void => {
  if (mutation.options.onError === undefined) {
    const message =
      error instanceof ApiError ? error.getUserMessage() : 'Operation failed'
    showToast?.(message)
  }
}

export const QUERY_STRATEGIES = {
  standard: {
    staleTime: QUERY_CONFIG.STALE_TIME.USER,
    gcTime: QUERY_CONFIG.GC_TIME.DEFAULT,
  },
  frequent: {
    staleTime: QUERY_CONFIG.STALE_TIME.FREQUENT,
    gcTime: QUERY_CONFIG.GC_TIME.DEFAULT,
    refetchInterval: QUERY_CONFIG.STALE_TIME.FREQUENT,
  },
  static: {
    staleTime: QUERY_CONFIG.STALE_TIME.STATIC,
    gcTime: QUERY_CONFIG.GC_TIME.LONG,
    refetchOnMount: false,
    refetchOnWindowFocus: false,
  },
  auth: {
    staleTime: QUERY_CONFIG.STALE_TIME.USER,
    gcTime: QUERY_CONFIG.GC_TIME.DEFAULT,
    retry: QUERY_CONFIG.RETRY.NONE,
  },
} as const

export type QueryStrategy = keyof typeof QUERY_STRATEGIES

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: QUERY_CONFIG.STALE_TIME.USER,
      gcTime: QUERY_CONFIG.GC_TIME.DEFAULT,
      retry: shouldRetryQuery,
      retryDelay: calculateRetryDelay,
      networkMode: 'offlineFirst',
    },
    mutations: {
      retry: QUERY_CONFIG.RETRY.NONE,
      networkMode: 'offlineFirst',
    },
  },
  queryCache: new QueryCache({
    onError: handleQueryCacheError,
  }),
  mutationCache: new MutationCache({
    onError: han
```

---

## File Overview

# query.config.ts Documentation

## Purpose and Responsibility
This file configures the `@tanstack/react-query` client for handling API queries and mutations in a React Native application. It sets up caching strategies, error handling, and persistence to ensure efficient data fetching and management.

## Key Exports and Public Interface
- **QueryStrategy**: Defines different caching strategies (`standard`, `frequent`, `static`, `auth`) that can be applied to queries.
- **queryClient**: The main `QueryClient` instance configured with various options for handling queries and mutations.
- **queryClientPersister**: An asynchronous storage persister for persisting the query cache.

## How it Fits into the Project
This file is a crucial component of the application's API layer, providing a centralized configuration point for managing data fetching. It interacts with other core modules like `errors` and `storage`, ensuring consistent behavior across different parts of the app.

## Notable Design Decisions
- **Error Handling**: Custom error handling functions (`handleQueryCacheError`, `handleMutationCacheError`) are used to manage errors gracefully, including showing user-friendly messages.
- **Retry Mechanism**: A dynamic retry mechanism is implemented based on failure count and error type, ensuring that non-retriable errors do not trigger unnecessary retries.
- **Caching Strategies**: Different caching strategies (`standard`, `frequent`, `static`, `auth`) are defined to cater to various use cases, optimizing data freshness and performance.
- **Persistence**: The `queryClientPersister` ensures that query states are saved and restored across application sessions using local storage.

This file plays a vital role in maintaining the integrity and efficiency of data operations throughout the application.

---

*Generated by CodeWorm on 2026-02-19 17:17*
