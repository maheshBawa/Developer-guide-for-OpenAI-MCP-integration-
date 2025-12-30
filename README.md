# Developer Guide: Using OpenAI Models in Antigravity

This guide explains how to integrate and use OpenAI models through the **Model Context Protocol (MCP)** in the Antigravity environment.

---

## 1. Understanding MCP vs. Base Models

Before starting, it's important to understand how Antigravity handles models:

- **Base Models (the "Brain")**  
  These are the primary models selected via the **Model Selector** in the chat interface (e.g., Gemini 3 Flash, Claude Sonnet). They act as the central reasoning engine of the agent.

- **MCP Servers (the "Plugins")**  
  MCP servers extend the base model with **tools**. When you add an OpenAI MCP server, you are **not** adding a new selectable "brain." Instead, you are giving the currently selected base model the ability to **call an OpenAI model on demand** as a tool.

> In short: the base model decides *when* to call tools; MCP servers define *what tools are available*.

---

## 2. Getting Your OpenAI API Key & Credits

OpenAI API usage is **separate** from ChatGPT Plus. To use OpenAI models via MCP, you must have API billing enabled.

### 2.1 Enable API Billing

- **Billing dashboard:**  
  https://platform.openai.com/account/billing

Add a payment method and set a non-zero monthly usage limit (even **$5** is enough to start).

> ChatGPT Plus does **not** include API credits.

### 2.2 Create an API Key

1. Go to: https://platform.openai.com/api-keys  
2. Click **Create new secret key**.  
3. Copy the key immediately and store it securely.

---

## 3. Setting Up System Environment Variables

Storing your API key in a system environment variable is the safest and most reusable approach.

### Windows (GUI)

1. Open **Start Menu** → search **Edit the system environment variables**.
2. Click **Environment Variables…**.
3. Under **User variables**, click **New…**.
4. Enter:
   - **Variable name:** `OPENAI_API_KEY`
   - **Variable value:** your secret API key
5. Click **OK** on all dialogs.
6. **Restart Antigravity / your IDE**.

### Windows (PowerShell — optional)

```powershell
setx OPENAI_API_KEY "sk-xxxxxxxxxxxxxxxx"
```

Restart Antigravity after running this command.

### macOS & Linux

1. Open a terminal.
2. Identify your shell:
   ```bash
   echo $SHELL
   ```
3. Edit your shell config file (`~/.zshrc`, `~/.bashrc`, etc.).
4. Add:
   ```bash
   export OPENAI_API_KEY='sk-xxxxxxxxxxxxxxxx'
   ```
5. Apply changes:
   ```bash
   source ~/.zshrc   # or ~/.bashrc
   ```
6. Restart Antigravity / your IDE.

---

## 4. Setting Up MCP Servers

Once your environment variable is set, configure the MCP server in Antigravity.

### Method A: Manual Setup

1. Open **Manage MCP servers** in Antigravity.
2. Click **Add Server**.
3. Use `npx` to point to `@pyroprompts/any-chat-completions-mcp`.
4. Configure the required environment variables.

> Manual setup works but can be error-prone. JSON config is recommended.

### Method B: Direct JSON Config (Recommended)

1. Open **Manage MCP servers**.
2. Click **View raw config** (file icon).
3. Add the following inside the `mcpServers` object:

```json
{
  "mcpServers": {
    "openai-gpt-4-1": {
      "command": "npx.cmd",
      "args": ["-y", "@pyroprompts/any-chat-completions-mcp"],
      "env": {
        "AI_CHAT_KEY": "${OPENAI_API_KEY}",
        "AI_CHAT_NAME": "OpenAI-GPT-4-1",
        "AI_CHAT_MODEL": "gpt-4.1",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      }
    },
    "openai-gpt-4o-mini": {
      "command": "npx.cmd",
      "args": ["-y", "@pyroprompts/any-chat-completions-mcp"],
      "env": {
        "AI_CHAT_KEY": "${OPENAI_API_KEY}",
        "AI_CHAT_NAME": "OpenAI-GPT-4o-mini",
        "AI_CHAT_MODEL": "gpt-4o-mini",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      }
    },
    "openai-gpt-5-2": {
      "command": "npx.cmd",
      "args": ["-y", "@pyroprompts/any-chat-completions-mcp"],
      "env": {
        "AI_CHAT_KEY": "${OPENAI_API_KEY}",
        "AI_CHAT_NAME": "OpenAI-GPT-5-2",
        "AI_CHAT_MODEL": "gpt-5.2",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      }
    }
  }
}
```

> **Note:**
> - On macOS or Linux, change `"command": "npx.cmd"` to `"command": "npx"`.
> - Ensure Node.js is installed and available on PATH (`node -v`, `npx -v`).

---

## 5. Model Availability Notes

- Models like **GPT-4.1** and **GPT-4o-mini** are widely available.
- **GPT-5.2** requires:
  - API billing enabled
  - Explicit access to the model for your OpenAI organization

If access is missing, requests may fail with `model_not_found` or quota errors.

---

## 6. How to Use OpenAI Models in Chat

1. Select a **Base Model** (e.g., Gemini 3 Flash) from the Model Selector.
2. In chat, explicitly ask the agent to use an OpenAI tool:
   - "Use GPT-4o-mini to summarize this code."
   - "Ask GPT-5-2 to review this architecture."

The base model will decide when to invoke the tool and return the OpenAI response inside the same conversation.

> **Tip:** Being explicit ("Use GPT-5-2…") improves tool-calling reliability.

---

## 7. Verifying Your Setup

1. Open **Manage MCP servers**
2. Click **Refresh**
3. Confirm that tools appear under each OpenAI server

If tools appear, the connection is successful.

---

## 8. Best-Practice Model Usage

Recommended setup:

| Task                          | Model       |
|-------------------------------|-------------|
| Deep reasoning / architecture | GPT-5.2     |
| Day-to-day coding             | GPT-4.1     |
| Fast or bulk tasks            | GPT-4o-mini |
