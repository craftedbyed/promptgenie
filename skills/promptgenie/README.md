# PromptGenie

## What is PromptGenie?
PromptGenie is a SKILL.md file that converts any raw user goal into a precision-engineered, production-ready prompt.

It loads directly into AI systems such as Claude Projects, agentic pipelines, LangGraph, CrewAI, AutoGen, or any chat interface. The skill runs a fixed four-phase process that elicits the necessary inputs, builds the prompt across seven explicit layers, and returns output ready for immediate use. It solves the core friction in LLM workflows: inconsistent prompt quality, excessive manual iteration, and downstream failures caused by vague or incomplete instructions. Developers, AI practitioners, and prompt engineers use it to produce repeatable, high-signal prompts without starting from scratch each time.

## How It Works
PromptGenie follows a deterministic four-phase architecture.

Phase 1 elicits exactly three inputs from the user: goal, requirements, and forbidden elements. If the goal is vague, it infers the most common use case, states the assumption explicitly, and proceeds without asking follow-up questions.

Phase 2 constructs the prompt using seven layers in order: one-line mission statement, hyper-specific persona, context, task with Chain-of-Thought steps, optional few-shot exemplars, output format specification, and NEVER-rule constraints. CoT depth and the anti-hallucination guard are chosen automatically via a built-in 9-type task taxonomy table. The table maps each task type—generation, extraction, transformation, analysis, code, research, creative, agentic, hybrid—to a default guard and CoT depth.

Before delivery, the model runs a 12-point self-verification checklist to confirm every layer is present, internally consistent, and aligned with the three inputs.

Phase 3 delivers the complete prompt wrapped in `<prompt>` `</prompt>` tags as copy-paste-ready output.

Phase 4 is the optional compression pass. It reduces the prompt to ≤60% of the original token count while preserving all functional layers and delivers the result in a second labelled block.

The entire process is model-agnostic and produces the same structure regardless of the underlying LLM.

## How to Use It

### Claude Projects
Create or open a Claude Project. Paste the full content of `SKILL.md` into the Project Instructions field. The skill auto-triggers on any prompt-engineering request. Add the memory hook phrase "Activate PromptGenie" at the start of your first message in the chat to keep the instructions resident across sessions. Provide the three inputs (goal, requirements, forbidden elements) in a single follow-up message.

### API/Agentic Pipeline
Insert PromptGenie as a pre-processing node: route the raw goal plus the three inputs into the skill and capture the `<prompt>` block as output. Follow it with a validation agent that parses the tagged output and confirms the seven layers are present. In LangGraph, CrewAI, or AutoGen, define a dedicated `PromptEngineer` role that loads `SKILL.md` as its system instruction. Connect a reflection edge that routes failed validation or poor downstream results back to the three inputs for re-processing.

### Any Chat Interface
Copy the entire `SKILL.md` content and paste it as the system prompt or as the very first message. Then send the three elicitation inputs in the next message. The skill will respond with the tagged prompt and optional compressed version.

## Iterative Refinement
Test the generated prompt by feeding it the original goal and requirements into your target model and inspecting the result. Identify the underperforming layer by name from the seven: one-line mission statement, hyper-specific persona, context, task with Chain-of-Thought steps, optional few-shot exemplars, output format specification, or NEVER-rule constraints. Adjust the corresponding part of the three elicitation inputs and re-run, or edit the relevant section inside `SKILL.md` itself.

Failure mode examples:
- Output hallucinates facts: anti-hallucination guard inside NEVER-rule constraints is under-specified for this task type—consult the taxonomy table in phase 2 and tighten the guard via additional forbidden elements or requirements.
- Reasoning lacks depth: task with Chain-of-Thought steps layer underperforms—add explicit CoT instructions to requirements or specify a higher-depth task type.
- Output format ignored: output format specification layer is weak—make the format description in requirements more rigid and re-test.
- Persona produces off-brand tone: hyper-specific persona layer is insufficient—supply concrete traits or background in the requirements input.

Repeat the loop until all seven layers perform as expected.

## Model Compatibility
PromptGenie is model-agnostic. It works with Claude, GPT-4, Gemini, and any frontier model. Load `SKILL.md` using the method appropriate to the target system—Project Instructions for Claude, system prompt for chat interfaces, or role definition for agent frameworks. No model-specific modifications or wrappers are required. The seven-layer structure and taxonomy table remain identical across all supported models.

## Token Budget Guidance
The optional compression pass reduces the prompt to ≤60% of the original token count while preserving all seven functional layers. Trigger it explicitly when your target model’s context window is constrained or when you need lower latency. The compressed prompt appears in a second labelled block immediately after the full version. Always compare the two blocks side-by-side to confirm every layer survives intact before deploying the compressed prompt in production.

## Versioning
Current version: 2.1.0. The full changelog lives inside `SKILL.md`. This repository follows semver: major = breaking architecture change, minor = new feature, patch = fix or clarification.

## Author
Edmond Francis

[LinkedIn](https://www.linkedin.com/in/edmond-francis-7a3764395/) | [Substack](https://leveragedfuture.substack.com/) | [𝕏](https://x.com/theedmind)