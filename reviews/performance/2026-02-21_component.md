# Component

**Type:** Performance Analysis
**Repository:** vuemantics
**File:** frontend/src/routes/upload/index.tsx
**Language:** tsx
**Lines:** 32-251
**Complexity:** 19.0

---

## Source Code

```tsx
function Component(): React.ReactElement {
  const {
    fileQueue,
    addFiles,
    removeFile,
    clearQueue,
    setDragActive,
    dragActive,
    setCurrentBatchId,
  } = useBulkUploadUIStore()

  const { data: clientConfig } = useClientConfig()
  const maxFileSizeBytes = (clientConfig?.max_upload_size_mb ?? 100) * 1024 * 1024

  const createBulkUpload = useCreateBulkUpload()

  const { batchProgress, setBatchProgress, setCurrentFile } =
    useGlobalBatchProgress()
  const { currentFile, handleFileProgress } = useFileProgress()

  useSocket({
    enabled: true,
    onBatchProgress: (data) => {
      setBatchProgress(data.payload.batch_id, {
        status: data.payload.status,
        total: data.payload.total,
        processed: data.payload.processed,
        successful: data.payload.successful,
        failed: data.payload.failed,
        progressPercentage: data.payload.progress_percentage,
      })
    },
    onFileProgress: (data) => {
      handleFileProgress(data)
      // Also update global store for header indicator
      if (data.payload.status === 'processing') {
        setCurrentFile({
          uploadId: data.payload.upload_id,
          fileName: data.payload.file_name,
          fileSize: data.payload.file_size,
          progress: data.payload.progress_percentage,
          status: data.payload.status,
        })
      } else {
        setCurrentFile(null)
      }
    },
  })

  const validateFile = (file: File): { valid: boolean; error?: string } => {
    const maxSizeMB = clientConfig?.max_upload_size_mb ?? 100
    if (file.size > maxFileSizeBytes) {
      return { valid: false, error: `Too large (max ${maxSizeMB}MB)` }
    }

    if (!Object.keys(ACCEPTED_TYPES).includes(file.type)) {
      return { valid: false, error: 'Unsupported file type' }
    }

    return { valid: true }
  }

  const handleDrag = (e: React.DragEvent): void => {
    e.preventDefault()
    e.stopPropagation()
    if (e.type === 'dragenter' || e.type === 'dragover') {
      setDragActive(true)
    } else if (e.type === 'dragleave') {
      setDragActive(false)
    }
  }

  const processFiles = (files: FileList | File[]): void => {
    const fileArray = Array.from(files)
    const newQueuedFiles: QueuedFile[] = []

    // Check for duplicates in existing queue
    const existingNames = new Set(fileQueue.map((f) => f.file.name))

    fileArray.forEach((file) => {
      const validation = validateFile(file)
      const isDuplicate = existingNames.has(file.name)

      const queuedFile: QueuedFile = {
        id: `${Date.now()}-${Math.random()}`,
        file,
        status: isDuplicate
          ? 'duplicate'
          : validation.valid
            ? 'valid'
            : validation.error?.includes('large')
              ? 'too-large'
              : 'unsupported',
        error: isDuplicate ? 'Already in queue' : validation.error,
      }

      // Generate preview for images
      if (file.type.startsWith('image/') && validation.valid) {
       
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The `processFiles` function has a time complexity of \(O(n^2)\) due to the nested iteration where each file is checked against existing queue entries using a set, and then validated individually.

#### Space Complexity
- The space complexity is \(O(n)\) for storing files in memory, where \(n\) is the number of files being processed.
- The `validateFile` function uses constant space but performs multiple checks on each file.

#### Bottlenecks or Inefficiencies
1. **Duplicate Check**: The use of a set to check for duplicate filenames introduces an unnecessary iteration over existing queue entries.
2. **Redundant Operations**: Each file is validated and checked for duplicates separately, leading to redundant operations.
3. **Blocking Calls**: `useSocket` calls are asynchronous but the handling functions (`onBatchProgress`, `onFileProgress`) are blocking.

#### Optimization Opportunities
1. **Optimize Duplicate Check**: Use a more efficient data structure like a map or object to store file names and their statuses, reducing the time complexity of duplicate checks.
2. **Lazy Validation**: Validate files only when necessary, e.g., on drag/drop events rather than continuously checking during processing.
3. **Asynchronous Handling**: Ensure that `onBatchProgress` and `onFileProgress` handle updates asynchronously to avoid blocking.

#### Resource Usage Concerns
- **Memory Leaks**: Ensure all created URLs (`URL.createObjectURL`) are properly cleared when no longer needed.
- **Socket Handling**: Avoid unnecessary state updates in socket handlers by batching them or using debouncing techniques.

By addressing these issues, the performance and efficiency of the component can be significantly improved.

---

*Generated by CodeWorm on 2026-02-21 21:39*
