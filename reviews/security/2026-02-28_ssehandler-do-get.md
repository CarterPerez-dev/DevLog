# SSEHandler.do_GET

**Type:** Security Review
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/haskell-reverse-proxy/examples/websockets/sse_server.py
**Language:** python
**Lines:** 13-103
**Complexity:** 8.0

---

## Source Code

```python
def do_GET(self):
        if self.path == "/health":
            self.send_response(200)
            self.send_header("Content-Type", "application/json")
            self.end_headers()
            self.wfile.write(json.dumps({"status": "healthy"}).encode())
            return

        if self.path == "/events":
            self.send_response(200)
            self.send_header("Content-Type", "text/event-stream")
            self.send_header("Cache-Control", "no-cache")
            self.send_header("Connection", "keep-alive")
            self.send_header("X-Accel-Buffering", "no")
            self.end_headers()

            print(f"[SSE] Client connected: {self.client_address}")

            try:
                event_id = 0
                while True:
                    event_id += 1
                    data = {
                        "id": event_id,
                        "timestamp": datetime.now().isoformat(),
                        "message": f"Event #{event_id}"
                    }

                    event = f"id: {event_id}\nevent: tick\ndata: {json.dumps(data)}\n\n"
                    self.wfile.write(event.encode())
                    self.wfile.flush()

                    print(f"[SSE] Sent event #{event_id}")
                    time.sleep(1)

            except (BrokenPipeError, ConnectionResetError):
                print(f"[SSE] Client {self.client_address} disconnected")
            return

        if self.path == "/stream/fast":
            self.send_response(200)
            self.send_header("Content-Type", "text/event-stream")
            self.send_header("Cache-Control", "no-cache")
            self.send_header("Connection", "keep-alive")
            self.end_headers()

            print(f"[SSE-FAST] Client connected: {self.client_address}")

            try:
                for i in range(100):
                    data = {"seq": i, "ts": datetime.now().isoformat()}
                    event = f"data: {json.dumps(data)}\n\n"
                    self.wfile.write(event.encode())
                    self.wfile.flush()
                    time.sleep(0.05)

                self.wfile.write(b"event: done\ndata: complete\n\n")
                self.wfile.flush()
                print(f"[SSE-FAST] Stream complete")

            except (BrokenPipeError, ConnectionResetError):
                print(f"[SSE-FAST] Client disconnected early")
            return

        self.send_response(200)
        self.send_header("Content-Type", "text/html")
        self.end_headers()
        html = """<!DOCTYPE html>
<html>
<head><title>SSE Test</title></head>
<body>
<h1>SSE Test Endpoints</h1>
<ul>
  <li><a href="/events">/events</a> - Continuous 1-second ticks</li>
  <li><a href="/stream/fast">/stream/fast</a> - Fast burst (100 events)</li>
  <li><a href="/health">/health</a> - Health check</li>
</ul>
<h2>Live Events:</h2>
<pre id="output"></pre>
<script>
const es = new EventSource('/events');
es.onmessage = (e) => {
  document.getElementById
```

---

## Security Review

### Security Review for `do_GET` Method

#### Vulnerabilities Found:

1. **Information Leakage** - **Severity: Low**
   - The server logs detailed information about connected clients and events sent, which could be used to infer the number of active connections or event frequency.
   - **Fix**: Use more generic logging messages.

2. **Insecure Error Handling** - **Severity: Low**
   - The `BrokenPipeError` and `ConnectionResetError` are caught but no specific actions are taken other than logging disconnection events.
   - **Fix**: Consider closing resources or performing cleanup actions if needed.

3. **Hardcoded Secrets/Credentials** - **Severity: Info**
   - No hardcoded secrets or credentials found in the provided code snippet.

4. **Input Validation Gaps** - **Severity: Low**
   - The `path` variable is directly used without validation, but since it's a static path check, this is not an issue.
   - **Fix**: Ensure all paths are validated if dynamic input is involved.

5. **Error Handling Leaks Information** - **Severity: Low**
   - Logging detailed error messages could potentially leak information about the server setup or configuration.
   - **Fix**: Use generic logging statements.

#### Attack Vectors:

- An attacker could exploit the detailed logging to infer server behavior and structure, which might be useful for crafting more targeted attacks.

#### Recommended Fixes:

1. **Logging Improvements**:
   ```python
   import logging

   logging.basicConfig(level=logging.INFO)
   ```

2. **Resource Cleanup**:
   - Ensure resources are properly closed or cleaned up in the `except` block if necessary.
   
3. **Generic Error Handling**:
   ```python
   except Exception as e:
       logging.error(f"An error occurred: {str(e)}")
   ```

#### Overall Security Posture:

The code is relatively secure for its intended purpose, but there are minor improvements that can be made to enhance security posture. The main focus should be on ensuring proper resource management and generic logging practices.

---

*Generated by CodeWorm on 2026-02-28 00:47*
