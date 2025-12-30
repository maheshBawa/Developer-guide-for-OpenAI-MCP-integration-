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

## 3. Setting Up System Environment Variables (Windows)

Storing your API key in a system variable is the most secure and efficient way to manage it across different MCP servers.

1.  Open the **Start Menu**, search for **"Edit the system environment variables"**, and press Enter.
2.  Click the **Environment Variables...** button.
3.  Under **User variables**, click **New...**.
4.  Enter the following:
    *   **Variable name:** `OPENAI_API_KEY`
    *   **Variable value:** *[Your Secret API Key]*
5.  Click **OK** on all windows to save.
6.  **Restart Antigravity/Your IDE** to ensure the new variable is recognized.

## 4. Setting Up MCP Servers Step-by-Step

Once your environment variable is set, you can configure the MCP server in Antigravity.

1.  Open **Manage MCP servers** in Antigravity.
2.  Click **Add Server** (or edit an existing one if you have a template).
3.  Configure the server (e.g., `openai-gpt-4o` or `openai-gpt-5-2`):
    *   Ensure the command points to the correct MCP bridge or script.
    *   The server should be configured to read the `OPENAI_API_KEY` from your system variables.
4.  Once added, ensure it is **Enabled**.
5.  You should see the tools appearing (e.g., `chat-with-openai-gpt-4o-mini`).

## 5. How to Use the OpenAI Models in Chat

1.  Select a **Base Model** (e.g., Gemini 3 Flash) from the Model Selector.
2.  In the chat, you can now ask the agent to use the OpenAI tool.
    *   *Example:* "Use the GPT-4o tool to review this code."
    *   *Example:* "Ask GPT-5-2 what it thinks about this architecture."
3.  The agent will automatically call the tool, send your prompt to OpenAI, and return the response within your current conversation.

> [!TIP]
> You can verify your setup by clicking **Refresh** in the **Manage MCP servers** view. If the tools appear under the server name, the connection is successful!
