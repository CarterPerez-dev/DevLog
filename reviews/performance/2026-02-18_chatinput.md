# ChatInput

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/encrypted-p2p-chat/frontend/src/components/Chat/ChatInput.tsx
**Language:** tsx
**Lines:** 20-125
**Complexity:** 11.0

---

## Source Code

```tsx
function ChatInput(props: ChatInputProps): JSX.Element {
  const [message, setMessage] = createSignal('')
  const [isTyping, setIsTyping] = createSignal(false)
  let typingTimeout: ReturnType<typeof setTimeout> | undefined
  let inputRef: HTMLInputElement | undefined

  const charCount = (): number => message().length
  const isOverLimit = (): boolean => charCount() > MESSAGE_MAX_LENGTH
  const canSend = (): boolean =>
    message().trim().length > 0 && !isOverLimit() && props.disabled !== true

  const handleInput = (e: Event): void => {
    const target = e.target as HTMLInputElement
    setMessage(target.value)

    if (!isTyping()) {
      setIsTyping(true)
      wsManager.sendTypingIndicator(props.roomId, true)
    }

    if (typingTimeout !== undefined) {
      clearTimeout(typingTimeout)
    }

    typingTimeout = setTimeout(() => {
      setIsTyping(false)
      wsManager.sendTypingIndicator(props.roomId, false)
    }, 2000)
  }

  const handleSend = (): void => {
    if (!canSend()) return

    const content = message().trim()
    props.onSend?.(content)
    setMessage('')

    if (isTyping()) {
      setIsTyping(false)
      wsManager.sendTypingIndicator(props.roomId, false)
    }

    inputRef?.focus()
  }

  const handleKeyDown = (e: KeyboardEvent): void => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault()
      handleSend()
    }
  }

  onCleanup(() => {
    if (typingTimeout !== undefined) {
      clearTimeout(typingTimeout)
    }
    if (isTyping()) {
      wsManager.sendTypingIndicator(props.roomId, false)
    }
  })

  return (
    <div
      class={`flex-shrink-0 p-4 border-t-2 border-orange ${props.class ?? ''}`}
    >
      <div class="flex gap-2">
        <div class="flex-1 flex items-center bg-black border-2 border-orange">
          <span class="font-pixel text-[10px] text-orange px-2">
            &gt;&gt;&gt;
          </span>
          <input
            ref={inputRef}
            type="text"
            value={message()}
            onInput={handleInput}
            onKeyDown={handleKeyDown}
            placeholder="TYPE YOUR MESSAGE..."
            disabled={props.disabled}
            maxLength={MESSAGE_MAX_LENGTH + 100}
            class="flex-1 bg-transparent font-pixel text-[10px] text-white py-2 pr-3 focus:outline-none placeholder:text-gray disabled:opacity-50"
          />
        </div>
        <Button
          variant="primary"
          size="md"
          onClick={handleSend}
          disabled={!canSend()}
          leftIcon={<SendIcon />}
        >
          SEND
        </Button>
      </div>

      <div class="flex items-center justify-between mt-2">
        <Show when={isOverLimit()}>
          <span class="font-pixel text-[8px] text-error">MESSAGE TOO LONG</span>
        </Show>
        <span
          class={`font-pixel text-[8px] ml-auto ${isOverLimit() ? 'text-error' : 'text-gray'}`}
        >
          {charCount()}/{MESSAGE_MAX_LENGTH}
        </span>
      </div>
    </div>
 
```

---

## Performance Analysis

### Performance Analysis of `ChatInput` Component

**Time Complexity:**
- The component has a time complexity of O(1) for most operations, as the logic is straightforward and does not involve nested loops or recursive calls.

**Space Complexity:**
- Space complexity is also O(1), with only a few state variables (`message`, `isTyping`, `typingTimeout`) and a reference to an input element. However, frequent updates to these state variables could lead to unnecessary re-renders if not optimized properly.

**Bottlenecks or Inefficiencies:**
- **Redundant State Updates:** The `handleInput` function updates the `message` state on every keystroke, which can trigger unnecessary re-renders. Consider debouncing this update.
- **Typing Indicator Handling:** Clearing and setting timeouts in `handleInput` could be optimized by using a single timeout for both sending typing indicators and clearing them.
- **Event Listeners:** Attaching event listeners (`onInput`, `onKeyDown`) to the input element can cause re-renders. Consider using controlled components or managing state updates more efficiently.

**Optimization Opportunities:**
- Use `useCallback` for `handleInput` and `handleSend` to avoid unnecessary function creation on every render.
- Debounce the `message` update in `handleInput` to reduce re-renders.
- Simplify typing indicator handling by using a single timeout that toggles between sending indicators.

**Resource Usage Concerns:**
- Ensure proper cleanup of timeouts and state in `onCleanup`. The current implementation looks good, but consider adding explicit checks for `typingTimeout` before clearing it.
- No blocking calls or N+1 query patterns are present. However, ensure that the WebSocket connection (`wsManager`) is managed properly to avoid resource leaks.

By implementing these optimizations, you can improve the performance and efficiency of the `ChatInput` component.

---

*Generated by CodeWorm on 2026-02-18 23:25*
