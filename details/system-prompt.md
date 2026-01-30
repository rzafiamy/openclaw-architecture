[← Go Back to Main Architecture](../README.md)

# System Prompt Construction

The system prompt is the foundational instruction set for every agent run in OpenClaw. It is dynamically constructed to provide the model with the necessary context, tool descriptions, and operational rules while remaining efficient in terms of token usage.

## 1. Prompt Modes

The complexity of the system prompt is determined by the `PromptMode`:

| Mode | Description | Used For |
| :--- | :--- | :--- |
| **`full`** | Includes all sections (Identity, Tools, Skills, Memory, Messaging, etc.). | Main Agent sessions. |
| **`minimal`** | Stripped-down version focusing on Tooling, Workspace, and specific task context. | Subagents and background tasks. |
| **`none`** | Just a single line identifying the assistant. | Minimalistic or highly specialized runs. |

```mermaid
graph TD
    Agent[Agent Instance] --> Mode{"Prompt Mode?"}
    
    Mode -->|"full"| Assembly["Full Assembly"]
    Mode -->|"minimal"| Assembly
    
    subgraph Modular Sections ["Prompt Sections"]
        Identity["Identity & Soul"]
        Tools["Tool Descriptions"]
        Runtime["Runtime Info (OS, Model, etc.)"]
        Skills["Skills Prompt (if enabled)"]
        Memory["Memory Guidance"]
        Context["Project Context Files"]
    end
    
    Modular Sections --> Assembly
    Assembly --> Final["Final System Prompt"]
```

## 2. Core Sections

A `full` system prompt consists of several modular sections:

### 2.1 Identity and Persona
Defines who the agent is ("You are a personal assistant running inside OpenClaw") and any personality traits injected via `SOUL.md`.

### 2.2 Tooling and Capabilities
-   **Tool Availability**: Lists all tools allowed by the agent's policy with brief summaries.
-   **Tool Call Style**: Instructs the model on when to narrate its actions vs. calling tools silently.
-   **Quick Reference**: Provides common OpenClaw CLI commands for service management.

### 2.3 Skills and Memory
-   **Skills**: Instructions on how to scan and read skill documentation from the `SKILL.md` library.
-   **Memory Recall**: Guidance on using `memory_search` and `memory_get` to retrieve information from long-term storage.

### 2.4 Messaging and Delivery
-   **Routing**: Explains how to reply in the current session vs. sending cross-session or proactive messages.
-   **Channel Capabilities**: Details about the current communication channel (e.g., if it supports inline buttons).

### 2.5 Runtime and Environment
-   **Workspace**: Specifies the current working directory.
-   **Sandbox**: If running in Docker, provide details about workspace access, browser bridge, and elevated execution permissions.
-   **Time**: Current date, time, and user timezone.

### 2.6 Project Context
Injects contents of specific files (like `README.md`, `TODO.md`, or custom docs) that the user wants the agent to keep in mind.

## 3. Subagent Prompting

Subagents receive a `minimal` prompt coupled with a "Subagent Context" block. This block emphasizes:
-   **Focus**: Their sole purpose is to complete the assigned `task`.
-   **Boundaries**: They are NOT the main agent and should not try to interact with the user or perform proactive side-quests.
-   **Output**: Instructions on how to format their final report for the parent agent.

## 4. Reasoning and Thinking

The prompt also includes instructions for structured output:
-   **Thinking Tags**: If enabled, the model is forced to wrap its internal reasoning in `<think>...</think>` tags and its final response in `<final>...</final>`. This ensures reasoning is captured but hidden from the end-user.

**Code Reference**: `src/agents/system-prompt.ts` contains the logic for section building and prompt assembly.
