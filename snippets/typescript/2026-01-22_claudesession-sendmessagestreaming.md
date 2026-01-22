# ClaudeSession.sendMessageStreaming

**Repository:** angelamos-operations
**File:** CarterBot-Telegram/src/session.ts
**Language:** typescript
**Lines:** 107-302
**Complexity:** 32.0

---

## Source Code

```typescript
async sendMessageStreaming(
    message: string,
    username: string,
    userId: number,
    statusCallback: StatusCallback,
    chatId?: number,
    ctx?: Context
  ): Promise<string> {
    if (chatId) {
      process.env.TELEGRAM_CHAT_ID = String(chatId);
    }

    const isNewSession = !this.isActive;
    const thinkingTokens = getThinkingLevel(message);
    const thinkingLabel =
      { 0: "off", 10000: "normal", 50000: "deep" }[thinkingTokens] ||
      String(thinkingTokens);

    let messageToSend = message;
    if (isNewSession) {
      const now = new Date();
      const datePrefix = `[Current date/time: ${now.toLocaleDateString(
        "en-US",
        {
          weekday: "long",
          year: "numeric",
          month: "long",
          day: "numeric",
          hour: "2-digit",
          minute: "2-digit",
          timeZoneName: "short",
        }
      )}]\n\n`;
      messageToSend = datePrefix + message;
    }

    const options: Options = {
      model: "claude-sonnet-4-5",
      cwd: WORKING_DIR,
      settingSources: ["user", "project"],
      permissionMode: "bypassPermissions",
      allowDangerouslySkipPermissions: true,
      systemPrompt: SYSTEM_PROMPT,
      mcpServers: MCP_SERVERS,
      maxThinkingTokens: thinkingTokens,
      resume: this.sessionId || undefined,
    };

    if (process.env.CLAUDE_CODE_PATH) {
      options.pathToClaudeCodeExecutable = process.env.CLAUDE_CODE_PATH;
    }

    if (this.sessionId && !isNewSession) {
      console.log(
        `RESUMING session ${this.sessionId.slice(0, 8)}... (thinking=${thinkingLabel})`
      );
    } else {
      console.log(`STARTING new Claude session (thinking=${thinkingLabel})`);
      this.sessionId = null;
    }

    if (this.stopRequested) {
      console.log("Query cancelled before starting");
      this.stopRequested = false;
      throw new Error("Query cancelled");
    }

    this.abortController = new AbortController();
    this.isQueryRunning = true;
    this.stopRequested = false;
    this.queryStarted = new Date();
    this.currentTool = null;

    const responseParts: string[] = [];
    let currentSegmentId = 0;
    let currentSegmentText = "";
    let lastTextUpdate = 0;
    let queryCompleted = false;

    try {
      const queryInstance = query({
        prompt: messageToSend,
        options: {
          ...options,
          abortController: this.abortController,
        },
      });

      for await (const event of queryInstance) {
        if (this.stopRequested) {
          console.log("Query aborted by user");
          break;
        }

        if (!this.sessionId && event.session_id) {
          this.sessionId = event.session_id;
          console.log(`GOT session_id: ${this.sessionId!.slice(0, 8)}...`);
          this.saveSession();
        }

        if (event.type === "assistant") {
          for (const block of event.message.content) {
            if (block.type === "thinking") {
              const thinkingText = block.thinking;
              if (thinkingText) {
                console.log(`THINKING: ${thinkingText.slice(0, 100)}...`);
                await statusCallback("thinking", thinkingText);
              }
            }

            if (block.type === "tool_use") {
              const toolName = block.name;
              const toolInput = block.input as Record<string, unknown>;

              if (currentSegmentText) {
                await statusCallback(
                  "segment_end",
                  currentSegmentText,
                  currentSegmentId
                );
                currentSegmentId++;
                currentSegmentText = "";
              }

              const toolDisplay = formatToolStatus(toolName, toolInput);
              this.currentTool = toolDisplay;
              this.lastTool = toolDisplay;
              console.log(`Tool: ${toolDisplay}`);
              await statusCallback("tool", toolDisplay);
            }

            if (block.type === "text") {
              responseParts.push(block.text);
              currentSegmentText += block.text;

              const now = Date.now();
              if (
                now - lastTextUpdate > STREAMING_THROTTLE_MS &&
                currentSegmentText.length > 20
              ) {
                await statusCallback(
                  "text",
                  currentSegmentText,
                  currentSegmentId
                );
                lastTextUpdate = now;
              }
            }
          }
        }

        if (event.type === "result") {
          console.log("Response complete");
          queryCompleted = true;

          if ("usage" in event && event.usage) {
            this.lastUsage = event.usage as TokenUsage;
            const u = this.lastUsage;
            console.log(
              `Usage: in=${u.input_tokens} out=${u.output_tokens} cache_read=${u.cache_read_input_tokens || 0}`
            );
          }
        }
      }
    } catch (error) {
      const errorStr = String(error).toLowerCase();
      const isCleanupError =
        errorStr.includes("cancel") || errorStr.includes("abort");

      if (isCleanupError && (queryCompleted || this.stopRequested)) {
        console.warn(`Suppressed post-completion error: ${error}`);
      } else {
        console.error(`Error in query: ${error}`);
        this.lastError = String(error).slice(0, 100);
        this.lastErrorTime = new Date();
        throw error;
      }
    } finally {
      this.isQueryRunning = false;
      this.abortController = null;
      this.queryStarted = null;
      this.currentTool = null;
    }

    this.lastActivity = new Date();
    this.lastError = null;
    this.lastErrorTime = null;

    if (currentSegmentText) {
      await statusCallback("segment_end", currentSegmentText, currentSegmentId);
    }

    await statusCallback("done", "");

    return responseParts.join("") || "No response from Claude.";
  }
```

---

## Documentation

### Documentation for `sendMessageStreaming` Function

**Purpose and Behavior:**
The `sendMessageStreaming` function in the `ClaudeSession` class handles sending messages to a Claude AI model, streaming responses, and updating status callbacks as the process unfolds. It supports resuming sessions, handling different thinking levels, and logging detailed information.

**Key Implementation Details:**
- **Environment Variables:** Sets Telegram chat ID if provided.
- **Session Management:** Manages session IDs and aborts queries on user cancellation.
- **Streaming Responses:** Streams responses in segments, updating status callbacks with text, tool usage, and thinking events.
- **Error Handling:** Catches errors and suppresses them post-completion unless explicitly requested.

**When/Why to Use:**
Use this function when you need to interact with Claude AI models in a streaming manner, especially for long-running queries or real-time applications. It's ideal for scenarios requiring detailed logging and status updates during the query process.

**Patterns/Gotchas:**
- **AbortController:** Ensure proper cleanup by setting `this.abortController` to null.
- **Session Management:** Handle session resumption carefully; avoid redundant session IDs.
- **Status Callbacks:** Ensure callbacks are properly defined to handle streaming events.

---

*Generated by CodeWorm on 2026-01-22 13:06*
