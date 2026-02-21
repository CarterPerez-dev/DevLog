# useVideos

**Type:** File Overview
**Repository:** social-media-notes
**File:** frontend/src/api/hooks/useVideos.ts
**Language:** typescript
**Lines:** 1-229
**Complexity:** 0.0

---

## Source Code

```typescript
/**
 * AngelaMos | 2025
 * useVideos.ts
 */

import { Alert } from 'react-native'
import {
  type UseMutationResult,
  type UseQueryResult,
  useMutation,
  useQuery,
  useQueryClient,
} from '@tanstack/react-query'
import {
  VIDEO_ERROR_MESSAGES,
  VIDEO_SUCCESS_MESSAGES,
  VideoResponseError,
  isValidVideoEntry,
  isValidVideoListResponse,
  type VideoEntry,
  type VideoCreateRequest,
  type VideoUpdateRequest,
  type VideoCopyRequest,
  type VideoListResponse,
} from '@/api/types'
import { API_ENDPOINTS, QUERY_KEYS, type Platform } from '@/config'
import { apiClient, QUERY_STRATEGIES } from '@/core/api'

export const videoQueries = {
  all: () => QUERY_KEYS.VIDEOS.ALL,
  list: (platform?: Platform) => QUERY_KEYS.VIDEOS.LIST(platform),
  byId: (id: string) => QUERY_KEYS.VIDEOS.BY_ID(id),
} as const

const fetchVideos = async (platform?: Platform): Promise<VideoEntry[]> => {
  const params = platform ? { platform } : {}
  const response = await apiClient.get<unknown>(API_ENDPOINTS.VIDEOS.BASE, { params })
  const data: unknown = response.data

  if (!isValidVideoListResponse(data)) {
    throw new VideoResponseError(
      VIDEO_ERROR_MESSAGES.INVALID_VIDEO_LIST_RESPONSE,
      API_ENDPOINTS.VIDEOS.BASE
    )
  }

  return data.items
}

export const useVideos = (
  platform?: Platform
): UseQueryResult<VideoEntry[], Error> => {
  return useQuery({
    queryKey: videoQueries.list(platform),
    queryFn: () => fetchVideos(platform),
    ...QUERY_STRATEGIES.standard,
  })
}

const fetchVideo = async (id: string): Promise<VideoEntry> => {
  const response = await apiClient.get<unknown>(API_ENDPOINTS.VIDEOS.BY_ID(id))
  const data: unknown = response.data

  if (!isValidVideoEntry(data)) {
    throw new VideoResponseError(
      VIDEO_ERROR_MESSAGES.INVALID_VIDEO_RESPONSE,
      API_ENDPOINTS.VIDEOS.BY_ID(id)
    )
  }

  return data
}

export const useVideo = (id: string): UseQueryResult<VideoEntry, Error> => {
  return useQuery({
    queryKey: videoQueries.byId(id),
    queryFn: () => fetchVideo(id),
    enabled: !!id,
    ...QUERY_STRATEGIES.standard,
  })
}

const performCreateVideo = async (data: VideoCreateRequest): Promise<VideoEntry> => {
  const response = await apiClient.post<unknown>(API_ENDPOINTS.VIDEOS.BASE, data)
  const responseData: unknown = response.data

  if (!isValidVideoEntry(responseData)) {
    throw new VideoResponseError(
      VIDEO_ERROR_MESSAGES.INVALID_VIDEO_RESPONSE,
      API_ENDPOINTS.VIDEOS.BASE
    )
  }

  return responseData
}

export const useCreateVideo = (): UseMutationResult<
  VideoEntry,
  Error,
  VideoCreateRequest
> => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: performCreateVideo,
    onSuccess: (data): void => {
      queryClient.invalidateQueries({ queryKey: videoQueries.list(data.platform) })
    },
    onError: (error: Error): void => {
      const message =
        error instanceof VideoResponseError
          ? error.message
          : VIDEO_ERROR_MESSAGES.FA
```

---

## File Overview

### useVideos.ts

**Purpose and Responsibility:**
This file provides hooks for managing video-related operations using `@tanstack/react-query`. It includes functionalities to fetch videos, create, update, and delete videos.

**Key Exports and Public Interface:**
- **useVideos(Platform?): UseQueryResult<VideoEntry[], Error>**: Fetches a list of videos based on the specified platform.
- **useVideo(id: string): UseQueryResult<VideoEntry, Error>**: Fetches a single video by its ID.
- **useCreateVideo(): UseMutationResult<VideoEntry, Error, VideoCreateRequest>**: Creates a new video entry.
- **useUpdateVideo(): UseMutationResult<VideoEntry, Error, UpdateVideoParams>**: Updates an existing video entry.
- **useDeleteVideo(): UseMutationResult<void, Error, string>**: Deletes a video by its ID.
- **useCopyVideo(): UseMutationResult<VideoEntry, Error, CopyVideoParams> (not fully implemented)**: Copies a video.

**How It Fits in the Project:**
This file is part of the `social-media-notes` frontend API hooks. It interacts with the backend via `apiClient` to manage videos and integrates seamlessly with other components that need video data or operations.

**Notable Design Decisions:**
- Utilizes `@tanstack/react-query` for state management, ensuring efficient re-renders.
- Implements validation functions (`isValidVideoEntry`, `isValidVideoListResponse`) to ensure data integrity before processing.
- Uses `useQueryClient` to invalidate queries after mutations, keeping the UI in sync with backend changes.
- Handles errors gracefully using `Alert` for user feedback and custom error messages.
```

This documentation provides an overview of the file's purpose, key exports, integration within the project, and notable design choices.

---

*Generated by CodeWorm on 2026-02-20 21:51*
