# Concepts

A soft intro to the ideas in LegionForge before you dive into Architecture, Guardian, or Security. If you've used another agent framework (LangChain, AutoGen, CrewAI, OpenClaw, Hermes, row-bot), most of this will feel familiar. The places it doesn't will be the places LegionForge differs intentionally.

## Pages in this section

<div class="grid cards" markdown>

-   :material-account-cog: **[Agents and tasks](agents-and-tasks.md)**

    What an agent is in LegionForge. What a task is. The lifecycle from submission to result.

-   :material-tools: **[Tools and capabilities](tools-and-capabilities.md)**

    What a tool is. What capability scope means. Why tools are signed, registered, and gated.

-   :material-shield-lock: **[Security fundamentals](security-fundamentals.md)**

    Trust boundaries. Sanitization. Deterministic checks. The thesis that the LLM is not trustworthy.

-   :material-book-alphabet: **[Glossary](glossary.md)**

    One-line definitions for every LegionForge-specific term — Guardian, HITL, capability scope, threat event, trust boundary, sanitization, A2A, MCP, and the rest.

</div>

## Where this fits

Concepts is the **mental model layer**. The order most people benefit from reading:

1. **Concepts** (you are here) — get the vocabulary
2. **[Quickstart](../quickstart.md)** — run something
3. **[Framework → Architecture](../framework/architecture.md)** — understand how the pieces compose
4. **[Security Model](../framework/security-model.md)** or **[Guardian](../guardian/index.md)** — go deep on what's different

Architecture is dense. If you read it first without the vocabulary, you'll bounce. Concepts gives you the words to make Architecture land.
