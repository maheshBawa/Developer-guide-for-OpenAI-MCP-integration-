# Developer Guide: Using OpenAI Models in Antigravity

This guide explains how to integrate and use OpenAI models through the Model Context Protocol (MCP) in the Antigravity environment.

## 1. Understanding MCP vs. Base Models
Before starting, it's important to understand how Antigravity handles models:

*   **Base Models (the "Brain")**: These are the primary models selected via the Model Selector in the chat interface (e.g., Gemini 3 Flash, Claude Sonnet). They act as the central reasoning engine of the agent.
*   **MCP Servers (the "Plugins")**: MCP servers extend the base model with tools. When you add an OpenAI MCP server, you are not adding a new selectable "brain." Instead, you are giving the currently selected base model the ability to call an OpenAI model on demand as a tool.

**In short:** The base model decides *when* to call tools; MCP servers define *what* tools are available.

## 2. Getting Your OpenAI API Key & Credits
OpenAI API usage is separate from ChatGPT Plus. To use OpenAI models via MCP, you must have API billing enabled.

1.  **Enable API Billing**: Go to [Billing Dashboard](https://platform.openai.com/account/billing). Add a payment method and set a small limit (e.g., $5).
2.  **Create an API Key**: Go to [API Keys](https://platform.openai.com/api-keys) and create a new secret key. Copy it immediately.

## 3. Setting Up System Environment Variables (The Robust Method)
For Windows users, relying on system environment variables can sometimes fail due to propagation issues or path handling. The most robust method—and the one configured for this environment—is using a local [.env](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/.env) file and a custom runner script.

### 3.1 Create the [.env](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/.env) File
1.  Navigate to your Antigravity configuration folder (typically `C:\Users\<YourUser>\.gemini\antigravity`).
2.  Create a file named [.env](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/.env).
3.  Add your key to the file:
    ```env
    AI_CHAT_KEY="sk-proj-xxxxxxxxxxxxxxxxxxxxxxxx"
    ```
    *(You can also use `OPENAI_API_KEY`)*
4.  **Critical**: Ensure the file is saved with **UTF-8 encoding**. (If using PowerShell/Notepad, be careful not to save as UTF-16/UCS-2, or the key won't be read).

### 3.2 Create the Runner Script
To ensure environment variables are loaded correctly and Windows commands execute reliably, use a wrapper script.

1.  Create a file named [mcp_dotenv_runner.mjs](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/mcp_dotenv_runner.mjs) in the same folder as your [.env](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/.env) file.
2.  Paste the following code:

```javascript
import { spawn } from "node:child_process";
import path from "node:path";
import process from "node:process";
import fs from "node:fs";
import { fileURLToPath } from "node:url";

function loadDotEnv(envPath) {
  if (!fs.existsSync(envPath)) return;

  const raw = fs.readFileSync(envPath, "utf8");
  for (const line of raw.split(/\r?\n/)) {
    const trimmed = line.trim();
    if (!trimmed || trimmed.startsWith("#")) continue;

    const idx = trimmed.indexOf("=");
    if (idx === -1) continue;
    
    const key = trimmed.slice(0, idx).trim();
    let val = trimmed.slice(idx + 1).trim();

    // strip surrounding quotes if present
    if (
      (val.startsWith('"') && val.endsWith('"')) ||
      (val.startsWith("'") && val.endsWith("'"))
    ) {
      val = val.slice(1, -1);
    }

    // Force override from file (solves stale env var issues)
    process.env[key] = val;
  }
}

// Load .env from same directory as this script
const envPath = path.join(path.dirname(fileURLToPath(import.meta.url)), ".env");
loadDotEnv(envPath);

// Ensure AI_CHAT_KEY is set
process.env.AI_CHAT_KEY =
  process.env.AI_CHAT_KEY || process.env.OPENAI_API_KEY || "";

const args = process.argv.slice(2);

// Windows: use npx.cmd with shell: true
const child = spawn("npx.cmd", args, {
  env: process.env,
  stdio: "inherit",
  shell: true, // Required for Windows .cmd execution
});

child.on("exit", (code) => process.exit(code ?? 1));
```

## 4. Configuring MCP Servers
Now configure Antigravity to use this runner script.

1.  Open **Manage MCP servers** in Antigravity.
2.  Click **View raw config** (file icon).
3.  Add or update the `mcpServers` object as follows.
    *Replace `<YourUser>` with your actual Windows username.*

```json
{
  "mcpServers": {
    "openai-gpt-4-1": {
      "command": "node",
      "args": [
        "C:\\Users\\<YourUser>\\.gemini\\antigravity\\mcp_dotenv_runner.mjs",
        "-y",
        "@pyroprompts/any-chat-completions-mcp"
      ],
      "env": {
        "AI_CHAT_NAME": "OpenAI-GPT-4-1",
        "AI_CHAT_MODEL": "gpt-4.1",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      }
    },
    "openai-gpt-4o-mini": {
      "command": "node",
      "args": [
        "C:\\Users\\<YourUser>\\.gemini\\antigravity\\mcp_dotenv_runner.mjs",
        "-y",
        "@pyroprompts/any-chat-completions-mcp"
      ],
      "env": {
        "AI_CHAT_NAME": "OpenAI-GPT-4o-mini",
        "AI_CHAT_MODEL": "gpt-4o-mini",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      }
    },
    "openai-gpt-5-2": {
      "command": "node",
      "args": [
        "C:\\Users\\<YourUser>\\.gemini\\antigravity\\mcp_dotenv_runner.mjs",
        "-y",
        "@pyroprompts/any-chat-completions-mcp"
      ],
      "env": {
        "AI_CHAT_NAME": "OpenAI-GPT-5-2",
        "AI_CHAT_MODEL": "gpt-5.2",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      }
    }
  }
}
```

## 5. Model Availability Notes
*   **GPT-4.1 / GPT-4o-mini**: Widely available.
*   **GPT-5.2**: Requires specific access permissions and a billed OpenAI organization. Use GPT-4o-mini for general testing if 5.2 access is restricted.

## 6. How to Use & Verify
1.  **Refesh**: Click "Refresh" in the MCP Servers list after updating config.
2.  **Verify**: If the tools appear under the server name, the connection is good.
3.  **Chat**: "Use GPT-5-2 to review this code."

## 7. Troubleshooting
*   **Error: `spawn EINVAL`**:
    *   **Cause**: Node.js on Windows cannot directly spawn `.cmd` files without a shell.
    *   **Fix**: Ensure `shell: true` is set in the runner script (already included in the script above).
*   **Error: `401 Incorrect API key`**:
    *   **Cause 1**: The [.env](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/.env) file has the wrong key.
    *   **Cause 2**: The [.env](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/.env) file is saved as UTF-16 (common in PowerShell).
    *   **Fix**: Open [.env](file:///C:/Users/Mahesh%20Haputhanthiri/.gemini/antigravity/.env) in VS Code, click the encoding bottom-right (e.g., "UTF-16 LE"), select "Save with Encoding", and choose "UTF-8".
