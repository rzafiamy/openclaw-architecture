[← Go Back to Main Architecture](../README.md)

# Core Principles of OpenClaw's A2A Architecture

OpenClaw's Agent-to-Agent (A2A) architecture is designed to handle complex, multi-step tasks by delegating them to specialized subagents. This approach ensures that the main agent remains responsive and focused, while background tasks are handled in an isolated environment.

The architecture is built on four fundamental pillars:

![Core Principles Illustration](/home/cook/.gemini/antigravity/brain/2993797b-a122-4145-846f-7990305b1801/core_principles_nano_banana_1769764265285.png)

## 1. Strict Isolation
Each subagent operates within its own dedicated session, which provides:
- **Clean History**: Subagents do not inherit the parent's conversation history unless explicitly passed. This prevents context pollution and keeps the subagent's attention on the specific task.
- **Unique workspace**: Subagents can be configured to work in separate directories, ensuring that their file operations do not interfere with the parent or other subagents.
- **Session Keying**: Sessions are uniquely identified using the format `agent:<AgentId>:subagent:<UUID>`, making them easily trackable and manageable.

## 2. Event-Driven Feedback
The parent agent is kept informed of the subagent's progress through a background registration system:
- **Lifecycle Events**: The system monitors `start`, `end`, and `error` events for every spawned subagent.
- **Real-time Monitoring**: The `SubagentRegistry` tracks every active run, allowing the system to react as soon as a subagent completes or fails.
- **Non-blocking**: The parent agent can continue interacting with the user or perform other tasks while subagents are running in the background.

## 3. Seamless Re-integration
Results from subagents are delivered back to the parent agent in a way that feels natural:
- **Message Steering**: If the parent agent is currently active, results can be "steered" directly into its context window. This allows the parent to receive information as if it just thought of it or as an immediate update from a collaborator.
- **Follow-up Messages**: If the parent agent is idle, the system can send a follow-up message that triggers a new run, summarizing the subagent's findings.
- **Natural Language Summaries**: The system automatically generates a concise summary of the subagent's output, allowing the parent agent to quickly understand and use the information.

## 4. Hierarchical Agent Identity
The system supports a variety of specialized agents, each with its own identity and capabilities:
- **Unique IDs**: Agents like `main`, `researcher`, and `coder` have distinct profiles.
- **Specialized Tools**: Different agents can be granted access to different sets of tools (e.g., a `coder` might have a full filesystem and execution tools, while a `researcher` might only have web search and fetch tools).
- **Model Flexibility**: Each agent can be configured to use different LLMs, choosing the best model for the task (e.g., a faster model for simple research and a more powerful model for complex coding).

---

### Design Rationale: Why this approach?
By breaking down complex tasks into smaller, isolated sub-tasks, OpenClaw achieves:
- **Reduced Hallucination**: Specialized agents with limited context are less likely to get confused by irrelevant information.
- **Scalability**: Multiple subagents can run in parallel, significantly speeding up complex workflows.
- **Maintainability**: Clear boundaries between agents make it easier to debug and improve specific parts of the system.
- **User Experience**: The user sees a more organized and responsive system that provides clear updates on background progress.
