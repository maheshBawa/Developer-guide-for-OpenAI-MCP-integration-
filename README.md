# Developer Guide: Using OpenAI Models in Antigravity

This guide explains how to integrate and use OpenAI models through the Model Context Protocol (MCP) in the Antigravity environment.

## 1. Understanding MCP vs. Base Models

Before starting, it's important to understand how Antigravity handles models:

*   **Base Models (The "Brain"):** These are the primary models selected via the **Model Selector** in the chat interface (e.g., Gemini 3 Flash, Claude Sonnet). They act as the central logic of the agent.
*   **MCP Servers (The "Plugins"):** These extend the base model's capabilities with **Tools**. When you add an OpenAI MCP server, you aren't adding a new selectable "Brain"; instead, you are giving the current Base Model the ability to "call" an OpenAI model as a tool.

## 2. Getting Your OpenAI API Key & Credits

To use OpenAI models via MCP, you need a valid API key and sufficient credits.

1.  **Get Credits:** OpenAI API usage is separate from ChatGPT Plus. You must add credits to your account.
    *   **Billing Overview:** [platform.openai.com/account/billing/overview](https://platform.openai.com/account/billing/overview)
    *   **Add Credits:** [platform.openai.com/account/billing/credit](https://platform.openai.com/account/billing/credit)
2.  **Generate API Key:**
    *   Go to: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
    *   Click **"Create new secret key"**.
    *   **IMPORTANT:** Copy the key immediately; you won't be able to see it again.

## 3. Setting Up System Environment Variables

Storing your API key in a system variable is the most secure and efficient way to manage it across different MCP servers. Choose your operating system below:

### Windows
1. Open the **Start Menu**, search for **"Edit the system environment variables"**, and press Enter.
2. Click the **Environment Variables...** button.
3. Under **User variables**, click **New...**.
4. Enter the following:
   * **Variable name:** `OPENAI_API_KEY`
   * **Variable value:** `[Your Secret API Key]`
5. Click **OK** on all windows to save.
6. **Restart Antigravity/Your IDE** to ensure the new variable is recognized.

### macOS & Linux
1. Open your terminal.
2. Identify your shell (run `echo $SHELL`). Usually, it's `zsh` on macOS and `bash` or `zsh` on Linux.
3. Open your configuration file in a text editor (e.g., `nano ~/.zshrc` or `nano ~/.bashrc`).
4. Add the following line at the end of the file:
   ```bash
   export OPENAI_API_KEY='[Your Secret API Key]'
   ```
5. Save the file and exit (`Ctrl+O`, `Enter`, `Ctrl+X` for nano).
6. Apply the changes by running:
   ```bash
   source ~/.zshrc  # or ~/.bashrc depending on your shell
   ```
7. **Restart Antigravity/Your IDE** to pick up the new environment variable.

## 4. Setting Up MCP Servers Step-by-Step

Once your environment variable is set, you can configure the MCP server in Antigravity.

### Method A: Manual Setup
1. Open **Manage MCP servers** in Antigravity.
2. Click **Add Server**.
3. Point to the `@pyroprompts/any-chat-completions-mcp` package using `npx`.
4. Ensure the environment variables match your model (e.g., `AI_CHAT_MODEL=gpt-4o`).

### Method B: Direct JSON Config (Recommended)
For a faster setup, you can copy the JSON below directly into your raw configuration file. 

1. In the **Manage MCP servers** view, click **View raw config** (the file icon at the top right).
2. Paste the following into the `mcpServers` object:

```json
{
  "mcpServers": {
    "openai-gpt-4-1": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "@pyroprompts/any-chat-completions-mcp"
      ],
      "env": {
        "AI_CHAT_KEY": "${OPENAI_API_KEY}",
        "AI_CHAT_NAME": "OpenAI-GPT-4-1",
        "AI_CHAT_MODEL": "gpt-4.1",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      }
    },
    "openai-gpt-4o-mini": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "@pyroprompts/any-chat-completions-mcp"
      ],
      "env": {
        "AI_CHAT_KEY": "${OPENAI_API_KEY}",
        "AI_CHAT_NAME": "OpenAI-GPT-4o-mini",
        "AI_CHAT_MODEL": "gpt-4o-mini",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      },
      "disabledTools": []
    },
    "openai-gpt-5-2": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "@pyroprompts/any-chat-completions-mcp"
      ],
      "env": {
        "AI_CHAT_KEY": "${OPENAI_API_KEY}",
        "AI_CHAT_NAME": "OpenAI-GPT-5-2",
        "AI_CHAT_MODEL": "gpt-5.2",
        "AI_CHAT_BASE_URL": "https://api.openai.com/v1"
      },
      "disabledTools": []
    }
  }
}
```

> [!NOTE]
> On macOS or Linux, you should change `"command": "npx.cmd"` to `"command": "npx"`.

## 5. How to Use the OpenAI Models in Chat

1.  Select a **Base Model** (e.g., Gemini 3 Flash) from the Model Selector.
2.  In the chat, you can now ask the agent to use the OpenAI tool.
    *   *Example:* "Use the GPT-4o tool to review this code."
    *   *Example:* "Ask GPT-5-2 what it thinks about this architecture."
3.  The agent will automatically call the tool, send your prompt to OpenAI, and return the response within your current conversation.

> [!TIP]
> You can verify your setup by clicking **Refresh** in the **Manage MCP servers** view. If the tools appear under the server name, the connection is successful!
