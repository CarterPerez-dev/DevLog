# useMetrics

**Type:** File Overview
**Repository:** oneIsNun_
**File:** react-scss/src/api/hooks/useMetrics.ts
**Language:** typescript
**Lines:** 1-87
**Complexity:** 0.0

---

## Source Code

```typescript
// ===================
// AngelaMos | 2026
// useMetrics.ts
// ===================

import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query'
import { apiClient } from '@/core/api'
import { API_ENDPOINTS, QUERY_CONFIG, QUERY_KEYS } from '@/config'
import {
  parseApiResponse,
  DashboardMetricsSchema,
  SlowQueryReportSchema,
  SlowQueryAnalysisSchema,
  ProfilingStatusSchema,
  type DashboardMetrics,
  type SlowQueryReport,
  type SlowQueryAnalysis,
  type ProfilingStatus,
  type SetProfilingRequest,
} from '../types'

export function useMetrics() {
  return useQuery({
    queryKey: QUERY_KEYS.METRICS.DASHBOARD(),
    queryFn: async (): Promise<DashboardMetrics> => {
      const { data } = await apiClient.get(API_ENDPOINTS.METRICS.DASHBOARD)
      return parseApiResponse(DashboardMetricsSchema, data)
    },
    staleTime: QUERY_CONFIG.STALE_TIME.METRICS,
    refetchInterval: QUERY_CONFIG.REFETCH_INTERVAL.METRICS,
  })
}

export function useSlowQueries(minMillis?: number, limit?: number) {
  return useQuery({
    queryKey: QUERY_KEYS.METRICS.SLOW_QUERIES(minMillis),
    queryFn: async (): Promise<SlowQueryReport> => {
      const params = new URLSearchParams()
      if (minMillis) params.append('min_millis', String(minMillis))
      if (limit) params.append('limit', String(limit))
      const url = `${API_ENDPOINTS.METRICS.SLOW_QUERIES}?${params}`
      const { data } = await apiClient.get(url)
      return parseApiResponse(SlowQueryReportSchema, data)
    },
    staleTime: QUERY_CONFIG.STALE_TIME.METRICS,
  })
}

export function useSlowQueryAnalysis(minMillis?: number, limit?: number) {
  return useQuery({
    queryKey: QUERY_KEYS.METRICS.ANALYSIS(),
    queryFn: async (): Promise<SlowQueryAnalysis> => {
      const params = new URLSearchParams()
      if (minMillis) params.append('min_millis', String(minMillis))
      if (limit) params.append('limit', String(limit))
      const url = `${API_ENDPOINTS.METRICS.ANALYZE}?${params}`
      const { data } = await apiClient.get(url)
      return parseApiResponse(SlowQueryAnalysisSchema, data)
    },
    staleTime: QUERY_CONFIG.STALE_TIME.METRICS,
  })
}

export function useProfilingStatus() {
  return useQuery({
    queryKey: QUERY_KEYS.METRICS.PROFILING(),
    queryFn: async (): Promise<ProfilingStatus> => {
      const { data } = await apiClient.get(API_ENDPOINTS.METRICS.PROFILING)
      return parseApiResponse(ProfilingStatusSchema, data)
    },
  })
}

export function useSetProfiling() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async (request: SetProfilingRequest) => {
      const { data } = await apiClient.put(API_ENDPOINTS.METRICS.PROFILING, request)
      return data
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: QUERY_KEYS.METRICS.PROFILING() })
    },
  })
}

```

---

## File Overview

# useMetrics.ts Documentation

## Purpose and Responsibility
This file provides React Query hooks for fetching various database metrics, including dashboard metrics, slow query reports, analysis results, and profiling status. It ensures that these metrics are fetched efficiently and updated regularly.

## Key Exports or Public Interface
- `useMetrics()`: Fetches dashboard metrics.
- `useSlowQueries(minMillis?: number, limit?: number)`: Fetches slow query reports with optional filtering by minimum execution time and limit.
- `useSlowQueryAnalysis(minMillis?: number, limit?: number)`: Fetches slow query analysis results with similar filtering options.
- `useProfilingStatus()`: Fetches the current profiling status.
- `useSetProfiling()`: A mutation hook to set profiling status.

## How It Fits in the Project
This file is part of a larger API hooks module, providing essential data fetching capabilities for the application. These hooks are used throughout the project to ensure that metrics and profiling information are always up-to-date and consistent with the backend.

## Notable Design Decisions
- **Use of `useQuery` and `useMutation`:** Leverages React Query’s powerful hook system for efficient state management and data fetching.
- **API Client Integration:** Utilizes a centralized API client (`apiClient`) to handle HTTP requests, ensuring consistency in request handling across the application.
- **Stale Time and Refetch Interval Configuration:** Configures `staleTime` and `refetchInterval` to manage cache invalidation and update frequency based on project requirements.
- **Type Safety:** Ensures type safety through schema validation using `parseApiResponse`, which helps prevent runtime errors related to data deserialization.
```

This documentation provides an overview of the file's purpose, key exports, integration within the project, and notable design decisions.

---

*Generated by CodeWorm on 2026-02-20 00:37*
