# useVoicePipeline

**Type:** Performance Analysis
**Repository:** angelamos-3d
**File:** frontend/src/hooks/useVoicePipeline.ts
**Language:** typescript
**Lines:** 37-285
**Complexity:** 1.0

---

## Source Code

```typescript
function useVoicePipeline({
	animControllerRef,
	animManagerRef,
	analyserRef,
	dataArrayRef,
	isPlayingRef,
	wakeWordRef,
	recorderRef,
	onStatusChange,
	onTranscriptChange,
	onResponseChange,
	onError,
}: UseVoicePipelineProps) {
	const audioContextRef = useRef<AudioContext | null>(null);
	const sourceRef = useRef<AudioBufferSourceNode | null>(null);
	const messagesRef = useRef<OllamaMessage[]>([]);
	const statusRef = useRef<AngelaStatus>("initializing");
	const abortControllerRef = useRef<AbortController | null>(null);

	const updateStatus = useCallback(
		(newStatus: AngelaStatus) => {
			statusRef.current = newStatus;
			onStatusChange(newStatus);

			const state = newStatus === "processing" ? "thinking" : newStatus;

			if (animControllerRef.current) {
				animControllerRef.current.setState(
					state as "idle" | "listening" | "thinking" | "speaking" | "error",
				);
			}

			if (animManagerRef.current) {
				animManagerRef.current.setState(
					state as "idle" | "listening" | "thinking" | "speaking" | "error",
				);
			}
		},
		[animControllerRef, animManagerRef, onStatusChange],
	);

	const playAudioWithLipSync = useCallback(
		async (audioBuffer: ArrayBuffer): Promise<void> => {
			audioContextRef.current = new AudioContext();
			analyserRef.current = audioContextRef.current.createAnalyser();
			analyserRef.current.fftSize = 256;
			dataArrayRef.current = new Uint8Array(
				analyserRef.current.frequencyBinCount,
			);

			const decoded = await audioContextRef.current.decodeAudioData(
				audioBuffer.slice(0),
			);

			sourceRef.current = audioContextRef.current.createBufferSource();
			sourceRef.current.buffer = decoded;
			sourceRef.current.connect(analyserRef.current);
			analyserRef.current.connect(audioContextRef.current.destination);

			isPlayingRef.current = true;

			return new Promise((resolve) => {
				const source = sourceRef.current;
				if (!source) {
					resolve();
					return;
				}
				source.onended = () => {
					isPlayingRef.current = false;
					animControllerRef.current?.setMouthOpen(0);
					audioContextRef.current?.close();
					resolve();
				};
				source.start(0);
			});
		},
		[analyserRef, dataArrayRef, isPlayingRef, animControllerRef],
	);

	const processAudio = useCallback(
		async (audioBlob: Blob) => {
			updateStatus("processing");

			try {
				logger.pipeline.log("Transcribing...");
				const result = await transcribeAudio(audioBlob);
				logger.pipeline.log("Transcription:", result.text);

				if (!result.text?.trim()) {
					logger.pipeline.log("Empty transcription");
					updateStatus("idle");
					wakeWordRef.current?.start();
					return;
				}

				onTranscriptChange(result.text);
				messagesRef.current.push({ role: "user", content: result.text });

				updateStatus("thinking");
				logger.pipeline.log("Calling LLM...");

				abortControllerRef.current = new AbortController();

				let fullResponse = "";
				try {
					for await (const chunk of streamChat(
						messagesRef.current,
						
```

---

## Performance Analysis

### Performance Analysis of `useVoicePipeline`

**Time Complexity:**
- The function has a time complexity dominated by the asynchronous operations, particularly the `transcribeAudio`, `streamChat`, and `synthesizeSpeech` calls.
- These operations are likely to be I/O bound and can significantly impact performance.

**Space Complexity:**
- The use of multiple references (`audioContextRef`, `analyserRef`, etc.) for storing objects increases memory usage. Ensure these are properly cleaned up when no longer needed.
- Large buffers like `dataArrayRef` could lead to increased memory consumption, especially if not reused or cleared after processing.

**Bottlenecks and Inefficiencies:**
1. **Redundant Audio Context Initialization:** The `playAudioWithLipSync` function initializes the `audioContextRef` every time it is called, which can be costly.
2. **Blocking Calls in Async Contexts:** The use of `new Promise((resolve) => { ... })` within `playAudioWithLipSync` could block the event loop if not handled properly.
3. **N+1 Query Pattern:** The `streamChat` function is called repeatedly, which may lead to unnecessary network requests.

**Optimization Opportunities:**
1. **Cache Audio Context:** Initialize and reuse the `audioContextRef` instead of creating a new one each time.
2. **Error Handling for AbortController:** Ensure proper cleanup when an `AbortError` occurs.
3. **Stream Chat Optimization:** Implement a mechanism to batch requests or use a more efficient streaming approach.

**Resource Usage Concerns:**
- **Unclosed Connections:** Ensure that all audio contexts and other resources are properly closed, especially in error scenarios.
- **Memory Leaks:** Regularly clear references like `analyserRef` and `sourceRef` after they are no longer needed to prevent memory leaks.

By addressing these areas, you can improve the performance and resource management of your application.

---

*Generated by CodeWorm on 2026-03-02 08:53*
