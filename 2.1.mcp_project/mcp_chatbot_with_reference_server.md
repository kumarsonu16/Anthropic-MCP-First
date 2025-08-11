### 📦 **Imports and Environment Setup**

```python
from dotenv import load_dotenv
from anthropic import Anthropic
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client
from typing import List, Dict, TypedDict
from contextlib import AsyncExitStack
import json
import asyncio

load_dotenv()
```

* `load_dotenv()`: Loads environment variables from a `.env` file.
* `Anthropic`: SDK to interact with Claude models.
* `mcp`: Client interface for interacting with MCP (Model Context Protocol) tool servers.
* `AsyncExitStack`: Manages multiple async context managers for clean teardown of resources.
* `json`, `asyncio`: Standard Python libraries.

---

### 🧰 **Tool Definition Type**

```python
class ToolDefinition(TypedDict):
    name: str
    description: str
    input_schema: dict
```

* A `TypedDict` that defines the structure of a tool:

  * `name`: Tool's identifier.
  * `description`: What it does.
  * `input_schema`: JSON schema that defines what input it expects.

---

### 💬 **MCP\_ChatBot Class**

#### 🔧 Constructor

```python
class MCP_ChatBot:

    def __init__(self):
        self.sessions: List[ClientSession] = [] # list of active tool sessions
        self.exit_stack = AsyncExitStack() # helps manage cleanup of resources
        self.anthropic = Anthropic() # Claude API client
        self.available_tools: List[ToolDefinition] = [] # list of tool metadata
        self.tool_to_session: Dict[str, ClientSession] = {} # maps tool name to the correct session
```

* This initializes the chatbot. It sets up containers to manage tools, sessions, and the Anthropic Claude client.

---

#### 🔌 Connect to One Server

```python
    async def connect_to_server(self, server_name: str, server_config: dict) -> None:
        ...
```

##### Key Steps:

1. **Create connection parameters** for a server using the configuration.
2. **Establish stdio transport** to talk to the tool server.
3. **Create a session** using the transport stream.
4. **Initialize the session** to start communication.
5. **List available tools** from that session.
6. **Store tool info** for use in the chatbot:

   * Tools are added to `self.available_tools`.
   * Mappings between tool name and session are stored in `self.tool_to_session`.

---

#### 🌐 Connect to All Servers

```python
    async def connect_to_servers(self):
```

* Reads `server_config.json` which contains the list of tool servers.
* Iterates through the servers and connects using the method above.
* Stores all active sessions for multi-tool communication.

---

#### 🔄 Process Query

```python
    async def process_query(self, query):
```

##### Step-by-Step Breakdown:

1. **Start a conversation** with Claude by sending the user query.
2. **Loop until conversation is done (`process_query = False`)**:

   * If response is plain text (`content.type == 'text'`): print and store it.
   * If response contains a `tool_use` directive:

     * Claude wants to invoke a specific tool.
     * Find the correct session and call the tool with its arguments.
     * Feed back the tool's result to Claude as a `tool_result`.
     * Get the next Claude response, which could be a follow-up text.

#### 🤖 Tool Call Structure

```python
tool_result = {
    "role": "user", 
    "content": [
        {
            "type": "tool_result",
            "tool_use_id": tool_id,
            "content": result.content
        }
    ]
}
```

* Claude needs the tool's result to continue the conversation — this is part of the structured interaction loop.

---

#### 💬 Chat Loop

```python
    async def chat_loop(self):
```

* A loop that prompts the user for input.
* If the user types `quit`, it breaks.
* Otherwise, it processes the input using the `process_query()` method.

---

#### 🧹 Cleanup

```python
    async def cleanup(self):
        await self.exit_stack.aclose()
```

* Ensures that all open sessions and transports are closed cleanly when the bot shuts down.

---

### 🚀 Entry Point

```python
async def main():
    chatbot = MCP_ChatBot()
    try:
        await chatbot.connect_to_servers()
        await chatbot.chat_loop()
    finally:
        await chatbot.cleanup()

if __name__ == "__main__":
    asyncio.run(main())
```

* Main function does three things:

  1. Initializes the chatbot.
  2. Connects to all servers.
  3. Runs the interactive query loop.
  4. Cleans up any resources when done.

---

### 📁 File Structure Notes

* `server_config.json`: Must exist with a structure like:

```json
{
  "mcpServers": {
    "tool_server_name": {
      "path": "path/to/executable",
      "args": ["arg1", "arg2"]
    }
  }
}
```

* Tools are external binaries or scripts that expose functions to Claude via MCP.

---

### 💡 Summary

This code integrates **Claude (Anthropic)** with **tool-servers** that implement the **MCP protocol**, allowing Claude to:

* Answer natural queries.
* Decide when a tool needs to be used.
* Provide structured input to that tool.
* Receive and incorporate results back into the conversation.

It's a chatbot that’s not just generative — it’s **actionable** via tools.
