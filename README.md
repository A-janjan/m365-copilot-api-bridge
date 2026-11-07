# Microsoft 365 Copilot OpenAI Proxy

**Use Microsoft 365 Copilot with OpenAI-compatible clients, developer tools, and custom applications.**

A lightweight, cross-platform FastAPI proxy that exposes Microsoft 365 Copilot through **OpenAI-compatible and Anthropic-compatible APIs**.

It connects to the same browser-facing WebSocket API used by the Microsoft 365 Copilot web interface, captures authentication from your existing signed-in browser session, and provides a local HTTP API.

**No Azure app registration. No admin consent. No separate API subscription setup.**

> **Cross-platform fork:** Originally Windows-only, this fork adds Linux and macOS support, fixes Windows-specific import dependencies, and introduces a Linux-friendly Microsoft Edge launcher.

---

## ✨ Features

- **OpenAI API compatibility** — Supports Chat Completions and Responses endpoints.
- **Anthropic API compatibility** — Provides an Anthropic-style Messages endpoint.
- **Browser-based authentication** — Reuses your existing Microsoft 365 Copilot login session.
- **Automatic token management** — Captures, monitors, and refreshes short-lived Substrate tokens.
- **Persistent conversations** — Maintain Copilot conversation context across multiple requests.
- **Streaming support** — Receive incremental responses through Server-Sent Events (SSE).
- **Cross-platform support** — Runs on Linux, macOS, and Windows.
- **Local-first architecture** — Exposes the API on `127.0.0.1` by default.

## 🏗️ How It Works

The proxy bridges the Microsoft 365 Copilot browser experience and applications that support OpenAI or Anthropic APIs.

```text
┌──────────────────────────────────────┐
│         OpenAI-Compatible Clients    │
│                                      │
│  OpenCode · Continue · Cline · SDK   │
└──────────────────┬───────────────────┘
                   │
                   │ HTTP / SSE
                   ▼
┌──────────────────────────────────────┐
│          FastAPI Proxy               │
│                                      │
│  • OpenAI Chat Completions           │
│  • OpenAI Responses                  │
│  • Anthropic Messages                │
│  • Session Management                │
│  • Token Refresh                     │
└──────────────────┬───────────────────┘
                   │
                   │ WebSocket
                   ▼
┌──────────────────────────────────────┐
│       Microsoft Substrate API        │
│          substrate.office.com        │
└──────────────────▲───────────────────┘
                   │
                   │ Authentication
                   │
┌──────────────────┴───────────────────┐
│       Chromium-Based Browser         │
│                                      │
│  Microsoft Edge / Chrome / Chromium  │
│                                      │
│  Signed-in M365 Copilot Session      │
└──────────────────────────────────────┘
```

The browser handles authentication. The proxy extracts the required Substrate token through the Chrome DevTools Protocol (CDP) and uses it to communicate with Microsoft's backend.

---

## 📋 Requirements

| Requirement | Version / Details |
|---|---|
| Python | 3.11+ |
| Browser | Chromium-based browser |
| Recommended browser | Microsoft Edge |
| Authentication | Signed-in Microsoft 365 Copilot account |
| Copilot URL | https://m365.cloud.microsoft/chat |

Supported operating systems:

- Linux
- macOS
- Windows

---

## 🚀 Installation

### Linux (Ubuntu / Debian)

#### 1. Install Microsoft Edge

Microsoft Edge is the recommended browser because the proxy can launch and manage a dedicated browser profile.

Install it through Microsoft's official repository:

```bash
sudo apt update
sudo apt install microsoft-edge-stable
```

Alternatively, download the installer from [Microsoft Edge](https://www.microsoft.com/edge).

If you prefer Chrome or Chromium, you can use your own browser without installing Edge. See [Using Your Own Browser](#using-your-own-browser).

#### 2. Clone the repository

```bash
git clone https://github.com/<your-user>/m365-copilot-api-bridge.git

cd m365-copilot-api-bridge
```

Replace `<your-user>` with the actual GitHub repository owner.

#### 3. Create a virtual environment

```bash
python3.11 -m venv .venv

source .venv/bin/activate
```

#### 4. Install dependencies

```bash
pip install -e ".[dev]"
```

#### 5. Start the proxy

```bash
copilot-openai-proxy serve
```

On the first launch:

1. The proxy opens a dedicated Microsoft Edge window.
2. Navigate to Microsoft 365 Copilot and sign in.
3. Keep the browser window open.
4. The proxy captures your Substrate authentication token.
5. The token is saved locally, and the API server starts.

The proxy becomes available at:

```text
http://127.0.0.1:8000
```

The dedicated Edge profile is stored at:

```text
~/.m365-copilot-openai-proxy/edge-profile
```

**Subsequent launches reuse this profile.** You normally only need to sign in once.

### macOS

Install Microsoft Edge for macOS, then follow the standard Python installation instructions above.

The launcher looks for Edge in its standard application directory:

```text
/Applications/Microsoft Edge.app/
```

### Windows

Microsoft Edge is generally preinstalled on Windows.

Follow the same Python installation and startup instructions. The launcher automatically searches for Edge in its standard installation location.

---

## ⚡ Quick Start

Once the proxy is running, test it with a simple request:

```bash
curl -s http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "m365-copilot",
    "messages": [
      {
        "role": "user",
        "content": "Say hello in one short sentence."
      }
    ]
  }' | python -m json.tool
```

Expected response structure:

```json
{
  "id": "chatcmpl_...",
  "object": "chat.completion",
  "created": 1762...,
  "model": "m365-copilot",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you today?"
      },
      "finish_reason": "stop"
    }
  ]
}
```

### Check authentication status

You can inspect token validity and expiration at any time:

```bash
curl -s http://127.0.0.1:8000/v1/token/status \
  | python -m json.tool
```

Example:

```json
{
  "valid": true,
  "expires_at": "2026-11-06T22:38:35+00:00",
  "seconds_remaining": 3535
}
```

---

## 🔌 Client Integration

The proxy can be used with applications and libraries that support OpenAI-compatible or Anthropic-compatible APIs.

### OpenAI-compatible clients

Use the following configuration:

| Setting | Value |
|---|---|
| Base URL | `http://127.0.0.1:8000/v1` |
| API Key | `dummy` |
| Model | `m365-copilot` |
| Persistent Model | `m365-copilot:persist` |

The API key is a placeholder required by many OpenAI-compatible clients. Authentication with Microsoft is handled separately through the browser session.

### Python SDK

Install the official OpenAI SDK:

```bash
pip install openai
```

Example:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://127.0.0.1:8000/v1",
    api_key="dummy"
)

response = client.chat.completions.create(
    model="m365-copilot",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)

print(response.choices[0].message.content)
```

### Claude Code and Anthropic clients

Configure the Anthropic-compatible endpoint:

```bash
export ANTHROPIC_BASE_URL=http://127.0.0.1:8000
export ANTHROPIC_API_KEY=dummy

claude
```

> **Important:** Anthropic-style API compatibility does not imply support for Claude Code's full agentic capabilities. Tool execution, file editing, Bash commands, and similar agent actions are not implemented by this proxy.

### Streaming responses

Streaming is supported through Server-Sent Events (SSE).

Example:

```bash
curl -N -s http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "m365-copilot",
    "stream": true,
    "messages": [
      {
        "role": "user",
        "content": "Count from 1 to 5."
      }
    ]
  }'
```

The server returns incremental `data: {...}` events, followed by:

```text
data: [DONE]
```

---

## 💬 Persistent Conversations

By default, each request creates a fresh Copilot conversation.

For applications that require conversation continuity, the proxy supports persistent sessions.

There are two ways to enable them.

### Option 1: Session ID header (recommended)

Pass a custom session identifier:

```http
X-M365-Session-Id: my-work-session
```

Requests with the same session ID reuse the same Copilot-side conversation.

This approach is particularly useful for:

- Multiple workspaces
- Multi-agent applications
- Applications serving multiple users
- Long-running research workflows

Each caller can maintain an independent conversation by using a different session ID.

### Option 2: Persistent model alias

If your client does not support custom HTTP headers, use:

```text
m365-copilot:persist
```

When the request does not include a `user` field, requests share a default persistent session until the proxy restarts.

---

## 🔄 Automatic Token Refresh

Microsoft Substrate tokens typically expire after approximately 60 minutes.

The proxy manages token expiration automatically.

### Automatic refresh

Automatic refresh is enabled by default.

Approximately every 55 minutes, the background refresh process:

1. Checks the token's expiration time.
2. Connects to the signed-in Edge browser through CDP.
3. Extracts a fresh Substrate token.
4. Updates the local `.env` file.
5. Makes the new token available to subsequent requests without restarting the server.

Example console output:

```text
Token expires in 240 seconds; refreshing from Edge...
Token refreshed automatically.
```

**Important:** The dedicated Edge window must remain open with an authenticated Copilot tab.

If Edge is closed or CDP becomes unavailable, automatic refresh fails and retries periodically until the browser becomes accessible again.

### Manual refresh

While the proxy is running:

| Key | Action |
|---|---|
| `r` | Force token refresh and restart the server |
| `q` | Quit the proxy |

### Manual token configuration

If automatic token extraction is unavailable, you can provide a token manually.

Run:

```bash
copilot-openai-proxy set-token
```

Then:

1. Open https://m365.cloud.microsoft/chat in an authenticated browser.
2. Open Developer Tools (`F12`).
3. Navigate to the **Network** tab.
4. Filter requests using `substrate`.
5. Find the relevant WebSocket connection.
6. Open its **Headers** section.
7. Copy the WebSocket Request URL.
8. Paste it into the proxy prompt.

The proxy extracts the required token and updates `.env`.

The running server automatically picks up the updated token.

### Refresh configuration

You can customize the refresh behavior:

```bash
# Refresh 10 minutes before expiration
copilot-openai-proxy serve --refresh-before-seconds 600

# Retry failed refresh attempts every 30 seconds
copilot-openai-proxy serve --refresh-retry-seconds 30
```

---

## 🌐 Using Your Own Browser

You don't have to install Microsoft Edge.

The proxy supports Chrome, Chromium, and other Chromium-based browsers that expose the Chrome DevTools Protocol.

### 1. Launch your browser with remote debugging

For Google Chrome:

```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.m365-copilot-openai-proxy/edge-profile" \
  --no-first-run \
  https://m365.cloud.microsoft/chat
```

You can also use:

```bash
chromium ...
```

Or:

```bash
microsoft-edge-stable ...
```

Sign in to Microsoft 365 Copilot if necessary.

### 2. Start the proxy without launching Edge

```bash
copilot-openai-proxy serve --no-launch-edge
```

### 3. Verify browser connectivity

```bash
curl -s http://localhost:9222/json | grep -i m365
```

As long as an authenticated Copilot tab is accessible through the configured debugging port, token extraction and automatic refresh work in the same way.

> **Security warning:** CDP exposes powerful browser-control capabilities. Only bind it to localhost, use a trusted browser profile, and never expose the debugging port to an untrusted network.

---

## 📡 API Reference

The proxy exposes the following HTTP endpoints:

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/healthz` | Service health and token status |
| `GET` | `/v1/token/status` | Token validity and expiration |
| `GET` | `/v1/models` | OpenAI-compatible model list |
| `POST` | `/v1/chat/completions` | OpenAI Chat Completions API |
| `POST` | `/v1/responses` | OpenAI Responses API |
| `POST` | `/v1/messages` | Anthropic Messages API |

Streaming is supported by the Chat Completions and Responses endpoints.

---

## ⚙️ Configuration

Most users do not need to configure environment variables manually.

The proxy manages its `.env` file during normal operation.

| Variable | Default | Description |
|---|---|---|
| `M365_ACCESS_TOKEN` | Empty | Substrate WebSocket authentication token |
| `M365_TIME_ZONE` | `Asia/Tokyo` | Time zone sent to Copilot |
| `M365_MODEL_ALIAS` | `m365-copilot` | Model name exposed through `/v1/models` |

You can override these values through your environment configuration.

---

## ⚠️ Rate Limits and Limitations

This project is an unofficial integration with Microsoft's browser-facing Copilot API. Understand its limitations before using it in production.

### Microsoft rate limits

Rate limits are enforced by Microsoft, not by this proxy.

Depending on your account and usage, you may encounter:

- Temporary throttling
- HTTP `429 Too Many Requests`
- Responses containing a `Retry-After` header

Clients should respect `Retry-After` and implement exponential backoff.

Microsoft does not publicly document a universal fixed request-per-minute limit for this integration.

### Conversation limits

Microsoft imposes a limit of approximately 600 messages per conversation.

For long-running persistent sessions, periodically rotate session IDs.

### No native agent execution

This project provides an API bridge, not a complete agent runtime.

The following capabilities are not implemented:

- Tool calling and execution
- Bash or terminal access
- File editing
- Browser automation
- Native agent orchestration

These capabilities must be implemented by the client or an external agent framework.

### Token usage reporting

Token usage statistics returned by the proxy are placeholders.

Microsoft's underlying streaming API does not expose the token counts required for accurate reporting.

### Unofficial API integration

The proxy communicates with Microsoft's browser-facing Substrate API rather than a documented public developer API.

Consequently:

- Microsoft may change the API without notice.
- Authentication behavior may change.
- Existing functionality may break after backend updates.
- Availability and rate limits are controlled by Microsoft.

### Authentication dependency

The proxy depends on a valid, signed-in browser session.

It does not use:

- Azure application registration
- Microsoft Authentication Library (MSAL)
- Device-code authentication
- Separate OAuth client credentials

This browser-based authentication approach is fundamental to the project's design.

---

## 🔐 Security

Security considerations are particularly important because the proxy handles Microsoft authentication tokens.

- **Local binding:** The HTTP server listens on `127.0.0.1` by default.
- **Token storage:** The Substrate token is stored locally in `.env`.
- **Token sensitivity:** Anyone who obtains a valid token may be able to use it until it expires or becomes invalid.
- **Git protection:** `.env`, `.venv/`, and Python cache files are excluded through `.gitignore`.
- **Browser security:** CDP debugging provides extensive access to the browser. Never expose its debugging port publicly.
- **Network communication:** The token is used to communicate with Microsoft's Substrate endpoint.

**Recommendations:**

1. Never commit `.env` or authentication tokens to Git.
2. Do not expose the proxy or CDP port to public networks.
3. Avoid sharing your dedicated authenticated browser profile.
4. Restrict filesystem permissions on files containing authentication data.

---

## 🙏 Acknowledgements

This project is a fork of:

[kuchris/m365-copilot-openai-proxy](https://github.com/kuchris/m365-copilot-openai-proxy)

The original project is licensed under the Apache License 2.0.

This fork extends the original implementation with cross-platform support, Linux-friendly browser launching, and additional improvements.

---

## 📄 License

Licensed under the **Apache License 2.0**.

See [LICENSE](LICENSE) for details.