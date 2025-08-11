# Model Context Protocol (MCP) Architecture

This is a summary of the Model Context Protocol (MCP) architecture from the official documentation.

## Scope

The Model Context Protocol includes the following projects:

* **MCP Specification**: A specification that outlines implementation requirements for clients and servers
* **MCP SDKs**: SDKs for different programming languages that implement MCP
* **MCP Development Tools**: Tools for developing MCP servers and clients, including the MCP Inspector
* **MCP Reference Server Implementations**: Reference implementations of MCP servers

## Core Concepts

### Participants

MCP follows a client-server architecture with these key participants:

* **MCP Host**: The AI application that coordinates and manages one or multiple MCP clients
* **MCP Client**: A component that maintains a connection to an MCP server and obtains context
* **MCP Server**: A program that provides context to MCP clients

The MCP host (an AI application like Claude Code or Claude Desktop) establishes connections to one or more MCP servers. The host creates one MCP client for each MCP server, with each client maintaining a dedicated one-to-one connection.

MCP servers can run locally or remotely:
* **Local MCP servers**: Run on the same machine (using STDIO transport)
* **Remote MCP servers**: Run on remote platforms (using Streamable HTTP transport)

### Layers

MCP consists of two layers:

#### 1. Data Layer
* Implements JSON-RPC 2.0 based exchange protocol
* Defines message structure and semantics
* Manages lifecycle (connection initialization, capability negotiation, termination)
* Provides server features (tools, resources, prompts)
* Enables client features (LLM sampling, user input elicitation, logging)
* Supports utilities like notifications and progress tracking

#### 2. Transport Layer
* Manages communication channels and authentication
* Handles connection establishment and message framing
* Supports two transport mechanisms:
  * **Stdio transport**: Uses standard input/output for local process communication
  * **Streamable HTTP transport**: Uses HTTP POST for remote communication with optional Server-Sent Events

### Data Layer Protocol

The data layer defines the schema and semantics between MCP clients and servers using JSON-RPC 2.0.

#### Lifecycle Management
* Negotiates capabilities that both client and server support
* Handles initialization sequence

#### Primitives

MCP defines three core primitives that servers can expose:

* **Tools**: Executable functions that AI applications can invoke (file operations, API calls, etc.)
* **Resources**: Data sources providing contextual information (file contents, database records, etc.)
* **Prompts**: Reusable templates for structuring interactions with language models

Each primitive has associated methods:
* Discovery methods (`*/list`)
* Retrieval methods (`*/get`)
* Execution methods (e.g., `tools/call`)

MCP also defines primitives that clients can expose:

* **Sampling**: Allows servers to request language model completions
* **Elicitation**: Allows servers to request additional information from users
* **Logging**: Enables servers to send log messages to clients

#### Notifications
* Support real-time updates between servers and clients
* Sent as JSON-RPC 2.0 notification messages (without expecting a response)
* Enable dynamic updates about changes in available tools or other context