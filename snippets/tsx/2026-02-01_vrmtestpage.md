# VrmTestPage

**Repository:** angelamos-operations
**File:** CarterOS-Client/src/aspects/assistant/facets/angela/VrmTestPage.tsx
**Language:** tsx
**Lines:** 20-364
**Complexity:** 13.0

---

## Source Code

```tsx
function VrmTestPage() {
  const canvasRef = useRef<HTMLCanvasElement>(null)
  const vrmRef = useRef<VRM | null>(null)
  const analyserRef = useRef<AnalyserNode | null>(null)
  const audioContextRef = useRef<AudioContext | null>(null)
  const sourceRef = useRef<AudioBufferSourceNode | null>(null)
  const dataArrayRef = useRef<Uint8Array | null>(null)
  const isPlayingRef = useRef(false)
  const wakeWordRef = useRef<ReturnType<typeof createWakeWordEngine> extends Promise<infer T> ? T : never>(null)
  const recorderRef = useRef<AudioRecorder | null>(null)
  const messagesRef = useRef<OllamaMessage[]>([])

  const [status, setStatus] = useState<Status>('initializing')
  const [transcript, setTranscript] = useState('')
  const [response, setResponse] = useState('')
  const [error, setError] = useState('')

  useEffect(() => {
    const canvas = canvasRef.current
    if (!canvas) return

    const width = window.innerWidth
    const height = window.innerHeight

    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true })
    renderer.setPixelRatio(window.devicePixelRatio)
    renderer.setSize(width, height)
    renderer.outputColorSpace = THREE.SRGBColorSpace

    const scene = new THREE.Scene()
    scene.background = new THREE.Color(0x1a1a2e)

    const camera = new THREE.PerspectiveCamera(30, width / height, 0.1, 100)
    camera.position.set(0, 1.3, 2.0)

    const controls = new OrbitControls(camera, canvas)
    controls.target.set(0, 1.3, 0)
    controls.enableDamping = true
    controls.update()

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.8)
    scene.add(ambientLight)

    const directionalLight = new THREE.DirectionalLight(0xffffff, 1.0)
    directionalLight.position.set(1, 1, 1)
    scene.add(directionalLight)

    const loader = new GLTFLoader()
    loader.register((parser) => new VRMLoaderPlugin(parser))

    loader.load('/vrm/angela.vrm', (gltf) => {
      const vrm = gltf.userData.vrm as VRM
      vrmRef.current = vrm

      vrm.scene.rotation.y = Math.PI
      scene.add(vrm.scene)

      const head = vrm.humanoid?.getNormalizedBoneNode('head')
      if (head) {
        const headPos = new THREE.Vector3()
        head.getWorldPosition(headPos)
        camera.position.set(0, headPos.y, 1.8)
        controls.target.set(0, headPos.y - 0.1, 0)
        controls.update()
      }

      console.log('VRM loaded with expressions:', vrm.expressionManager?.expressionMap)
    })

    const clock = new THREE.Clock()
    let animationId: number
    let blinkTimer = 0

    const animate = () => {
      animationId = requestAnimationFrame(animate)
      const delta = clock.getDelta()

      controls.update()

      if (vrmRef.current) {
        vrmRef.current.update(delta)

        blinkTimer += delta
        if (blinkTimer > 4) {
          vrmRef.current.expressionManager?.setValue('blink', 1)
          setTimeout(() => {
            vrmRef.current?.expressionManager?.setValue('blink', 0)
          }, 100)
          blinkTimer = 0
        }

        if (isPlayingRef.current && analyserRef.current && dataArrayRef.current) {
          analyserRef.current.getByteFrequencyData(dataArrayRef.current)
          let sum = 0
          const lowFreqEnd = Math.floor(dataArrayRef.current.length * 0.3)
          for (let i = 0; i < lowFreqEnd; i++) {
            sum += dataArrayRef.current[i]
          }
          const volume = sum / (lowFreqEnd * 255)
          const mouthOpen = Math.min(1, volume * 3)
          vrmRef.current.expressionManager?.setValue('aa', mouthOpen)
        }
      }

      renderer.render(scene, camera)
    }
    animate()

    const handleResize = () => {
      const w = window.innerWidth
      const h = window.innerHeight
      camera.aspect = w / h
      camera.updateProjectionMatrix()
      renderer.setSize(w, h)
    }
    window.addEventListener('resize', handleResize)

    return () => {
      window.removeEventListener('resize', handleResize)
      cancelAnimationFrame(animationId)
      renderer.dispose()
      controls.dispose()
    }
  }, [])

  useEffect(() => {
    let mounted = true

    const init = async () => {
      try {
        recorderRef.current = new AudioRecorder()
        await recorderRef.current.initialize()
        console.log('AudioRecorder ready')

        const wakeWord = await createWakeWordEngine()
        wakeWordRef.current = wakeWord

        wakeWord.onWakeWord = () => {
          if (!mounted) return
          handleWakeWord()
        }

        await wakeWord.start()
        console.log('Wake word listening')

        if (mounted) setStatus('idle')
      } catch (err) {
        console.error('Init error:', err)
        if (mounted) {
          setError(err instanceof Error ? err.message : 'Init failed')
          setStatus('error')
        }
      }
    }

    init()

    return () => {
      mounted = false
      wakeWordRef.current?.dispose()
      recorderRef.current?.dispose()
    }
  }, [])

  const handleWakeWord = async () => {
    if (!recorderRef.current) return

    setStatus('listening')
    setTranscript('')
    setResponse('')
    wakeWordRef.current?.stop()

    const silenceDetector = new SilenceDetector({
      silenceThreshold: 0.01,
      silenceDuration: 1500,
    })

    recorderRef.current.onAudioLevel = (level) => {
      silenceDetector.update(level)
    }

    silenceDetector.onSilenceDetected = () => {
      recorderRef.current?.stop()
    }

    recorderRef.current.onComplete = async (audioBlob) => {
      await processAudio(audioBlob)
    }

    recorderRef.current.start()
  }

  const processAudio = async (audioBlob: Blob) => {
    setStatus('processing')

    try {
      const result = await transcribeAudio(audioBlob)
      if (!result.text?.trim()) {
        setStatus('idle')
        wakeWordRef.current?.start()
        return
      }

      setTranscript(result.text)
      messagesRef.current.push({ role: 'user', content: result.text })

      setStatus('thinking')

      let fullResponse = ''
      for await (const chunk of streamChat(messagesRef.current)) {
        fullResponse += chunk
        setResponse(fullResponse)
      }

      messagesRef.current.push({ role: 'assistant', content: fullResponse })

      setStatus('speaking')

      const audioBuffer = await synthesizeSpeech(fullResponse)
      await playAudioWithLipSync(audioBuffer)

      setStatus('idle')
      wakeWordRef.current?.start()
    } catch (err) {
      console.error('Process error:', err)
      setError(err instanceof Error ? err.message : 'Processing failed')
      setStatus('error')

      setTimeout(() => {
        setStatus('idle')
        setError('')
        wakeWordRef.current?.start()
      }, 3000)
    }
  }

  const playAudioWithLipSync = async (audioBuffer: ArrayBuffer): Promise<void> => {
    audioContextRef.current = new AudioContext()
    analyserRef.current = audioContextRef.current.createAnalyser()
    analyserRef.current.fftSize = 256
    dataArrayRef.current = new Uint8Array(analyserRef.current.frequencyBinCount)

    const decoded = await audioContextRef.current.decodeAudioData(audioBuffer.slice(0))

    sourceRef.current = audioContextRef.current.createBufferSource()
    sourceRef.current.buffer = decoded
    sourceRef.current.connect(analyserRef.current)
    analyserRef.current.connect(audioContextRef.current.destination)

    isPlayingRef.current = true

    return new Promise((resolve) => {
      sourceRef.current!.onended = () => {
        isPlayingRef.current = false
        vrmRef.current?.expressionManager?.setValue('aa', 0)
        audioContextRef.current?.close()
        resolve()
      }
      sourceRef.current!.start(0)
    })
  }

  const triggerManually = () => {
    if (status === 'idle') {
      handleWakeWord()
    }
  }

  const statusColors: Record<Status, string> = {
    initializing: '#888',
    idle: '#4ade80',
    listening: '#22c55e',
    processing: '#f59e0b',
    thinking: '#8b5cf6',
    speaking: '#ec4899',
    error: '#ef4444',
  }

  return (
    <div style={{ width: '100vw', height: '100vh', overflow: 'hidden', position: 'relative' }}>
      <canvas ref={canvasRef} style={{ display: 'block' }} />

      <div
        style={{
          position: 'absolute',
          top: 20,
          left: 20,
          color: 'white',
          fontFamily: 'monospace',
          background: 'rgba(0,0,0,0.7)',
          padding: '16px',
          borderRadius: '8px',
          maxWidth: '400px',
        }}
      >
        <div style={{ marginBottom: '12px' }}>
          <span
            style={{
              display: 'inline-block',
              width: '12px',
              height: '12px',
              borderRadius: '50%',
              background: statusColors[status],
              marginRight: '8px',
            }}
          />
          <strong>{status.toUpperCase()}</strong>
          {status === 'idle' && <span style={{ marginLeft: '8px', opacity: 0.7 }}>Say "Angela"</span>}
        </div>

        {transcript && (
          <div style={{ marginBottom: '8px' }}>
            <div style={{ opacity: 0.7, fontSize: '12px' }}>You:</div>
            <div>{transcript}</div>
          </div>
        )}

        {response && (
          <div style={{ marginBottom: '8px' }}>
            <div style={{ opacity: 0.7, fontSize: '12px' }}>Angela:</div>
            <div style={{ maxHeight: '150px', overflow: 'auto' }}>{response}</div>
          </div>
        )}

        {error && <div style={{ color: '#ef4444' }}>{error}</div>}

        <button
          onClick={triggerManually}
          disabled={status !== 'idle'}
          style={{
            marginTop: '12px',
            padding: '8px 16px',
            background: status === 'idle' ? '#8b5cf6' : '#444',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            cursor: status === 'idle' ? 'pointer' : 'not-allowed',
          }}
        >
          Trigger Manually
        </button>
      </div>
    </div>
  )
}
```

---

## Documentation

### Documentation for `VrmTestPage`

#### Purpose and Behavior

`VrmTestPage` is a React component that integrates VRM (Virtual Reality Model) with audio processing and speech recognition. The component initializes a 3D scene using Three.js, loads a VRM model, and processes audio to recognize wake words and transcribe user input. It then uses Ollama for text-to-speech synthesis and lip-syncs the VRM model.

#### Key Implementation Details

- **State Management**: Uses `useState` for component state like status, transcript, response, and error.
- **Effects Hooks**: Utilizes `useEffect` to handle side effects such as initializing audio recording, setting up wake word detection, and managing 3D scene updates.
- **Audio Processing**: Implements audio level detection using `SilenceDetector` and processes audio blobs with `AudioRecorder`.
- **VRM Integration**: Uses Three.js for rendering the VRM model and lip-syncing expressions.

#### When/Why to Use This Code

This component is ideal for integrating interactive 3D VRM models with real-time speech recognition and synthesis. It can be used in applications requiring natural language processing, voice commands, and animated character interactions.

#### Patterns and Gotchas

- **Resource Management**: Ensure proper cleanup of `AudioContext` and other resources to avoid memory leaks.
- **Error Handling**: The component handles errors gracefully by setting state and retrying operations after a delay.
- **Performance Considerations**: Audio processing can be resource-intensive, so consider performance optimizations for real-time applications.

---

*Generated by CodeWorm on 2026-02-01 02:19*
