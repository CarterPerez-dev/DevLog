# formatting

**Type:** File Overview
**Repository:** angelamos-operations
**File:** CarterBot-Telegram/src/formatting.ts
**Language:** typescript
**Lines:** 1-198
**Complexity:** 0.0

---

## Source Code

```typescript
/**
 * CarterOS Telegram Bot - Formatting
 *
 * Markdown to Telegram HTML conversion and tool status display.
 */

export function escapeHtml(text: string): string {
  return text
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;");
}

export function convertMarkdownToHtml(text: string): string {
  const codeBlocks: string[] = [];
  const inlineCodes: string[] = [];

  text = text.replace(/```(?:\w+)?\n?([\s\S]*?)```/g, (_, code) => {
    codeBlocks.push(code);
    return `\x00CODEBLOCK${codeBlocks.length - 1}\x00`;
  });

  text = text.replace(/`([^`]+)`/g, (_, code) => {
    inlineCodes.push(code);
    return `\x00INLINECODE${inlineCodes.length - 1}\x00`;
  });

  text = escapeHtml(text);

  text = text.replace(/^#{1,6}\s+(.+)$/gm, "<b>$1</b>\n");
  text = text.replace(/\*\*(.+?)\*\*/g, "<b>$1</b>");
  text = text.replace(/(?<!\*)\*(.+?)\*(?!\*)/g, "<b>$1</b>");
  text = text.replace(/__([^_]+)__/g, "<b>$1</b>");
  text = text.replace(/(?<!_)_([^_]+)_(?!_)/g, "<i>$1</i>");
  text = text.replace(/^[-*] /gm, "• ");
  text = text.replace(/^[-*]{3,}$/gm, "");
  text = text.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2">$1</a>');

  for (let i = 0; i < codeBlocks.length; i++) {
    const escapedCode = escapeHtml(codeBlocks[i]!);
    text = text.replace(`\x00CODEBLOCK${i}\x00`, `<pre>${escapedCode}</pre>`);
  }

  for (let i = 0; i < inlineCodes.length; i++) {
    const escapedCode = escapeHtml(inlineCodes[i]!);
    text = text.replace(
      `\x00INLINECODE${i}\x00`,
      `<code>${escapedCode}</code>`
    );
  }

  text = text.replace(/\n{3,}/g, "\n\n");

  return text;
}

function shortenPath(path: string): string {
  if (!path) return "file";
  const parts = path.split("/");
  if (parts.length >= 2) {
    return parts.slice(-2).join("/");
  }
  return parts[parts.length - 1] || path;
}

function truncate(text: string, maxLen = 60): string {
  if (!text) return "";
  const cleaned = text.replace(/\n/g, " ").trim();
  if (cleaned.length <= maxLen) return cleaned;
  return cleaned.slice(0, maxLen) + "...";
}

function code(text: string): string {
  return `<code>${escapeHtml(text)}</code>`;
}

export function formatToolStatus(
  toolName: string,
  toolInput: Record<string, unknown>
): string {
  const emojiMap: Record<string, string> = {
    Read: "📖",
    Write: "📝",
    Edit: "✏️",
    Bash: "▶️",
    Glob: "🔍",
    Grep: "🔎",
    WebSearch: "🔍",
    WebFetch: "🌐",
    Task: "🎯",
    mcp__carteros: "🗄️",
    mcp__: "🔧",
  };

  let emoji = "🔧";
  for (const [key, val] of Object.entries(emojiMap)) {
    if (toolName.includes(key)) {
      emoji = val;
      break;
    }
  }

  if (toolName === "Read") {
    const filePath = String(toolInput.file_path || "file");
    return `${emoji} Reading ${code(shortenPath(filePath))}`;
  }

  if (toolName === "Write") {
    const filePath = String(toolInput.file_path || "file");
    return `${emoji} Writing ${code(shortenPath(filePath))}`;
  }

  if
```

---

## File Overview

### File Documentation

**File Purpose and Responsibility:**
This TypeScript source file, `formatting.ts`, is part of the CarterOS Telegram Bot project. It handles two main tasks: converting Markdown text to HTML for better display in Telegram messages and formatting tool status updates.

**Key Exports or Public Interface:**
- **escapeHtml(text: string): string** - Escapes special characters in a given string to prevent XSS attacks.
- **convertMarkdownToHtml(text: string): string** - Converts Markdown-formatted text into HTML, supporting various Markdown syntaxes like bold, italic, and code blocks.
- **shortenPath(path: string): string** - Shortens file paths for better readability.
- **truncate(text: string, maxLen = 60): string** - Truncates long strings to a specified length with an ellipsis if necessary.
- **code(text: string): string** - Wraps text in `<code>` tags for monospaced display.
- **formatToolStatus(toolName: string, toolInput: Record<string, unknown>): string** - Formats and displays the status of various tools used by CarterOS, including their names, actions, and parameters.

**How It Fits in the Project:**
This file is crucial for ensuring that messages sent by the bot are well-structured and visually appealing. The `convertMarkdownToHtml` function ensures that user inputs can be displayed correctly, while `formatToolStatus` provides clear and concise status updates for tool executions within the bot.

**Notable Design Decisions:**
1. **Use of Regular Expressions:** The file extensively uses regular expressions to parse and transform Markdown text into HTML, ensuring accurate conversion.
2. **Escape Functions:** Both `escapeHtml` and `truncate` functions are used to sanitize input data, preventing potential security issues like XSS attacks.
3. **Tool Status Formatting:** The `formatToolStatus` function dynamically generates status messages based on the tool's name and input parameters, making it highly flexible for different use cases.

---

*Generated by CodeWorm on 2026-02-19 19:19*
