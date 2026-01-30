[← Go Back to Main Architecture](../README.md)

# Agent Configuration in OpenClaw

OpenClaw uses a hierarchical configuration system to define agent identities, capabilities, and constraints. This document details how agents are defined, resolved, and used within the system.

## 1. Defining Agents

Agents are defined in the `agents.list` section of the OpenClaw configuration file (usually `config.yaml` or `config.json`).

### 1.1 Configuration Schema

Each agent entry supports the following options:

| Option | Type | Description |
| :--- | :--- | :--- |
| `id` | `string` | Unique identifier (e.g., `main`, `coder`, `researcher`). Normalized to lowercase. |
| `name` | `string` | Human-readable display name for logs and UI. |
| `default` | `boolean` | If `true`, this agent handles sessions where no ID is specified. |
| `workspace` | `string` | Root directory for file operations. Supports `~` for home directory expansion. |
| `agentDir` | `string` | Directory for agent-specific state, history, and transcripts. |
| `model` | `string` \| `object` | Primary model or a configuration with `primary` and `fallbacks`. |
| `tools` | `object` | Policy defining allowed/denied tools and profiles. |
| `subagents` | `object` | Configuration for spawning subagents (e.g., `allowAgents`, `model`). |
| `sandbox` | `object` | Security and isolation settings (e.g., Docker configuration). |
| `identity` | `object` | Identity-related prompts (e.g., personality, role). |

### 1.2 Example Configuration

```yaml
agents:
  defaults: # Global defaults for all agents
    workspace: ~/openclaw-workspace
  list:
    - id: main
      name: "Guardian"
      default: true
      model: anthropic/claude-3-5-sonnet-latest
    - id: researcher
      name: "Insight specialist"
      workspace: ~/research-data
      model: openai/gpt-4o
      tools:
        allow: ["group:web"]
```

![Agent Configuration Illustration](/home/cook/.gemini/antigravity/brain/2993797b-a122-4145-846f-7990305b1801/agent_config_nano_banana_1769764285315.png)

## 2. Agent Resolution Logic

The system must resolve which agent should handle a given request based on the session key.

### 2.1 Resolution Steps

1.  **Extract Agent ID**: The session key is parsed (e.g., `agent:researcher:abc123` → `researcher`).
2.  **Fallback to Default**: If no agent ID is found in the key, the system looks for the agent marked `default: true`. If none is marked, the first agent in the list is used.
3.  **Normalization**: All IDs are normalized to lowercase to avoid case-sensitivity issues.
4.  **Configuration Merging**: The specific agent's config is merged with global defaults.

**Code Reference**: `src/agents/agent-scope.ts` contains the logic for `resolveSessionAgentId` and `resolveAgentConfig`.

## 3. Directory and Workspace Management

Each agent is assigned specific directories to ensure isolation:

-   **Workspace Dir**: The directory where the agent can read and write files via tools like `read` and `write`. If not specified, it defaults to a unique directory under `~/.openclaw/workspace-<agentId>`.
-   **Agent Dir**: Stores persistent state, including the JSONL transcripts for every session handled by that agent.

## 4. Subagent Policies

Agents can be restricted in their ability to spawn other agents using `subagents.allowAgents`:

-   `["*"]`: Can spawn any defined agent.
-   `["coder", "researcher"]`: Only allowed to spawn these specific agents.
-   `undefined` (default): Can only spawn subagents using its own identity (`agentId`).

This hierarchy allows for a secure and controlled delegation of tasks, where a "parent" agent can only call upon "sub-specialists" that it has been explicitly granted access to.
