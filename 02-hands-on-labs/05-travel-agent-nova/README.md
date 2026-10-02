# Hands on lab - Create an Amazon Bedrock Travel Agent using Amazon Nova

**Playlist position:** #8 of 86
**Category:** Hands-on Labs
**YouTube:** https://www.youtube.com/watch?v=O7e5Hvguvt0
**Duration:** ~58:16
**Channel:** NamrataHShah

---

## TL;DR
- **Agents for Amazon Bedrock** let a foundation model go beyond chat — it can **reason about a request, decide to call your APIs/functions (Action Groups), optionally consult a Knowledge Base, and return a final answer**, all orchestrated automatically.
- This lab builds a **travel-planning agent** powered by an **Amazon Nova** model, wired up with Action Groups (e.g., flight/hotel search style functions) so it can actually *do* things, not just talk about them.
- The agent is built, tested, and iterated on in the console, then **prepared** and published behind an **alias** for applications to call.

## Detailed Notes

### What an Agent is
An Agent wraps a foundation model with **orchestration logic**: given a user's request, it decides — turn by turn — whether it has enough information to answer directly, whether it needs to call one of its configured **Action Groups** (functions/APIs), or whether it should query an attached **Knowledge Base**. This loop (reason → act → observe → repeat) is what turns a plain chat model into something that can complete multi-step, tool-using tasks — see [Orchestration & Agent](../../01-tutorials-overview/01-terminology/README.md) from Video 1.

### Why Amazon Nova for this
**Amazon Nova** is Amazon's own family of foundation models (Micro, Lite, Pro, Premier for text/multimodal, plus Canvas for images and Reel for video). The text/multimodal Nova models support **function calling / tool use**, which is exactly what Agents need to reliably decide *when* and *how* to invoke an Action Group — making Nova Pro/Lite a natural, cost-effective choice for agentic workloads inside Bedrock.

### Building the travel agent (console workflow)
1. **Amazon Bedrock → Agents → Create Agent.** Pick a Nova model as the agent's underlying FM.
2. **Instructions** — write the agent's system prompt: its role ("You are a travel-planning assistant that helps users find flights, hotels, and build itineraries…"), tone, and boundaries.
3. **Action Groups** — define the functions the agent is allowed to call (e.g., search flights, search hotels, check weather for a destination). Each action is described either via an **OpenAPI schema** or a simpler **function-details** definition (name, description, parameters), and is backed by an **AWS Lambda** function that actually executes the logic.
4. *(Optional)* **Knowledge Base** — attach one so the agent can ground answers in reference material (e.g., destination guides, travel policies).
5. **Test** in the console's built-in chat window — multi-turn, with the console showing the agent's **trace**: which action it decided to call, what parameters it passed, and what the Lambda returned, before it composed the final answer.
6. **Prepare** the agent (compiles the latest configuration) and publish it behind an **alias** (e.g., `prod`), which is the stable endpoint applications actually invoke — so you can keep iterating on a draft without breaking consumers of a published alias.

### Why it matters
This lab is the practical, end-to-end version of the Agent + Orchestration concepts from Video 1 — showing that an "agent" isn't magic, it's a model + a set of well-described functions + a loop that decides when to call them.

## Diagram

```mermaid
flowchart TB
    User[User: "Plan a 3-day trip to Goa"] --> Agent[Bedrock Agent<br/>powered by Amazon Nova]
    Agent -->|reasons about the request| Decide{Need a tool?}
    Decide -->|yes| AG[Action Group:<br/>search flights / hotels / weather]
    AG --> Lambda[AWS Lambda<br/>executes the actual logic]
    Lambda --> Agent
    Decide -->|needs reference info| KB[(Optional Knowledge Base)]
    KB --> Agent
    Decide -->|no, has enough info| Final
    Agent --> Final[Final composed answer]
    Final --> User
```

## Quick Revision Cheat-Sheet

| Concept | One-line takeaway |
|---|---|
| Agent | FM + orchestration loop that can call tools and consult data |
| Action Group | A described function/API the agent can invoke, backed by Lambda |
| OpenAPI schema / function details | Two ways to describe an Action Group's interface to the agent |
| Amazon Nova | Amazon's FM family; Nova Pro/Lite support function calling for agents |
| Trace | Console view of the agent's reasoning + tool calls during a test |
| Prepare | Compiles the agent's current draft configuration |
| Alias | Stable, published endpoint apps invoke, decoupled from the draft |

## Interview Q&A

**Q: How does a Bedrock Agent decide whether to call an Action Group?**
A: The underlying FM (here, Amazon Nova) reasons over the user's request and its instructions; if it determines it needs external data or a side effect it can't produce from its own knowledge, it emits a function call matching one of the configured Action Groups, which Bedrock routes to the backing Lambda.

**Q: What's the role of the Lambda function behind an Action Group?**
A: It executes the actual logic for that action (e.g., calling a flights API, querying a database) — the agent only decides *when* and *with what parameters* to call it; the Lambda does the real work and returns a result the agent incorporates into its next reasoning step.

**Q: Why publish an agent behind an alias instead of pointing apps at the draft?**
A: An alias is a stable reference to a prepared version of the agent, so you can keep editing and testing a new draft without affecting what's currently live for consumers of that alias.

**Q: Why is function-calling support in the model important for building agents?**
A: Agent orchestration depends on the model reliably producing structured calls (name + parameters) to the right tool at the right time — models without good function-calling support make the orchestration loop unreliable.

## References
- AWS docs: [Automate tasks in your application using conversational agents](https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html)
- AWS docs: [Supported foundation models in Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html)
- AWS docs: [Model support by AWS Region](https://docs.aws.amazon.com/bedrock/latest/userguide/models-regions.html)

## Status / Navigation
- ⬅️ Previous: [Prompt Routers - Preview](../../03-prompt-management/01-prompt-routers-preview/README.md)
- ➡️ Next: [Model Catalog Demo](../../04-marketplace-catalog/01-model-catalog-demo/README.md)

---
_Part of [AWS Bedrock Learnings](../../README.md)._
