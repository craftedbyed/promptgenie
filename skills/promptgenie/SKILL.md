---
name: promptgenie

description:
  Transforms any unstructured user request into a production-grade, hallucination-proof prompt
  ready for immediate use with any frontier AI model (Claude, GPT-4, Gemini, etc.). Use this skill
  whenever a user asks to "write me a prompt", "create a prompt for", "help me prompt", "make a
  system prompt", "engineer a prompt", "optimize my prompt", or wants to get better AI outputs for
  any task. Also trigger when the user describes a goal and seems to be struggling to get the AI
  to do what they want — even if they don't mention "prompt" explicitly. This skill covers simple
  one-liners up to complex multi-agent system prompts.

model: sonnet
---

# PromptGenie

Converts any raw user goal into a precision-engineered, production-ready prompt using every advanced technique: Chain-of-Thought mandates, anti-hallucination guards, self-critique loops, output format enforcement, few-shot exemplars, and token efficiency optimization.

---

## How to Use This Skill

### Step 1 — Elicit the Three Inputs

Ask the user for exactly these three things (and nothing else — do not ask follow-up questions unless a critical ambiguity would make the prompt unusable):

1. **Goal / desired outcome** — What should the AI produce or do?
2. **Requirements** — Tone, length, format, style, constraints, examples to include.
3. **Forbidden elements** — What must the AI never do, say, or include?

Present these as a clean numbered list. Wait for the user's response before proceeding.

**Elicitation fallback — vague or ambiguous input:**
If the user's Goal is too vague to construct a usable prompt (e.g., "make me a good prompt for coding" or "help me with writing"), do not halt or ask multiple clarifying questions. Instead:
- Infer the single most common use case for the stated domain.
- State your assumption explicitly at the top of your output: *"I'm interpreting your goal as [inferred goal]. If that's not right, tell me and I'll rebuild."*
- Proceed with construction using that inference.
- Apply this fallback once only — do not ask for clarification more than once per session.

---

### Step 2 — Build the Prompt

Once you have the three inputs, construct the optimized prompt using the architecture below. Work through each layer internally before writing output.

#### Internal Reasoning Sequence (do this silently before writing)

1. **Identify the core task type** — use the taxonomy table below to select the type, then apply the mapped default anti-hallucination guard and CoT depth.
2. **Select the optimal persona** — The most credibly expert role for this exact task. Be hyper-specific (not "an expert" but "a senior staff engineer at a FAANG company with 12 years specializing in distributed systems").
3. **Map the output format** — Determine the exact structure: headings, bullet styles, tables, code blocks, word count ranges, section order.
4. **Design the anti-hallucination layer** — Use the default guard from the taxonomy table as a baseline; tighten it for this task's specific fabrication risks.
5. **Identify where CoT adds value** — Insert step-by-step reasoning mandates only where they improve accuracy, not as boilerplate. Use the CoT depth from the taxonomy table as a starting point.
6. **Write a self-critique instruction** — Force the model to review its own draft against the user's goal before finalizing.
7. **Choose exemplars** — If the user provided examples, embed them. If not, synthesize 1–2 minimal illustrative examples that clarify the output pattern.

#### Task-Type Taxonomy

Use this table to route anti-hallucination strategy and CoT depth by task type. Apply the defaults as a baseline; adjust for task-specific nuances.

| Task Type         | Description                                              | Default Anti-Hallucination Guard                                                    | CoT Depth  |
|-------------------|----------------------------------------------------------|-------------------------------------------------------------------------------------|------------|
| **Generation**    | Creating original content (posts, essays, copy)          | Consistency check — flag any claim or detail not derivable from provided inputs     | Shallow    |
| **Extraction**    | Pulling structured data from provided text               | Source-grounding rule — every extracted item must cite its location in the input    | Shallow    |
| **Transformation**| Rewriting, translating, reformatting existing content    | Fidelity check — verify no meaning was added, removed, or distorted                | Shallow    |
| **Analysis**      | Interpreting, evaluating, or diagnosing provided content | Evidence-tethering rule — every claim must cite specific evidence from the input    | Medium     |
| **Code**          | Writing, debugging, or reviewing code                    | Correctness verification — test logic against stated requirements before outputting | Medium     |
| **Research**      | Synthesizing knowledge or multiple sources               | Citation integrity rule — no fabricated sources, authors, dates, or statistics      | Deep       |
| **Creative**      | Storytelling, world-building, character-driven output    | Internal consistency check — characters, timelines, and rules must not contradict  | Shallow    |
| **Agentic**       | System prompts, multi-step workflows, agent instructions | Boundary completeness check — every edge case named in requirements must be handled | Deep       |
| **Hybrid**        | Any combination of two or more above types              | Apply the strictest guard from each constituent type                                | Deep       |

---

#### Prompt Architecture (assemble in this order)

```
[ONE-LINE MISSION STATEMENT]
A single imperative sentence stating exactly what the model must produce.

[PERSONA]
You are [hyper-specific role with domain, seniority, and specialization].

[CONTEXT]
[Background, audience, situation, constraints the model needs to understand the task]

[TASK]
Your task is to [precise, unambiguous action verb + deliverable].

Step-by-step, you must:
1. [First reasoning/action step]
2. [Second reasoning/action step]
3. [Continue as needed — include self-critique as a mandatory step]
4. Review your draft. Verify it satisfies: [list 2–3 success criteria from the user's goal]. Revise if any criterion is unmet before outputting.

[EXEMPLARS — if applicable]
Here are [N] examples of the desired output pattern:
<example_1>
[example content]
</example_1>
[repeat as needed]

[OUTPUT FORMAT]
Your response must follow this exact structure:
- [Section 1 name]: [description + formatting rules]
- [Section 2 name]: [description + formatting rules]
[Add word/length constraints if specified]

[TONE]
Write in a [tone descriptor] voice. [Add 1–2 specific tone rules if needed.]

[CONSTRAINTS]
- [Constraint 1 from user requirements]
- [Constraint 2 from user requirements]
- NEVER [forbidden element 1]
- NEVER [forbidden element 2]
- Do not fabricate [specific fabrication risk for this task type]. If uncertain, state "I don't have sufficient information to answer this reliably" rather than guessing.

[TRIGGER]
Begin your response now.
```

---

### Step 3 — Deliver Output

Wrap the final prompt **only** in:

```
<prompt>
[prompt content]
</prompt>
```

Output **nothing** outside these tags — no preamble, no explanation, no commentary. The output must be copy-paste ready with zero editing required.

---

### Step 4 — Compress (Optional)

If the user requests a token-efficient version, or if the generated prompt exceeds ~600 tokens and will be used in a context-constrained pipeline, apply a compression pass after Step 3:

- Target: ≤60% of the original token count.
- Preserve all functional layers: mission statement, persona, CoT steps, self-critique, output format, NEVER-rules, trigger line. None may be omitted.
- Remove: transitional phrases, redundant restatements, filler qualifiers ("please", "make sure to", "it is important that").
- Merge CoT steps that share the same epistemic operation into one step.
- Shorten exemplars to the minimum that still conveys the output pattern.
- After compression, re-run the Self-Verification Checklist (below) to confirm no functional layer was lost.
- Wrap the compressed version in the same `<prompt>` / `</prompt>` tags and deliver it as a second block labelled **[Compressed version]**.

---

## Output Quality Rules

Every prompt this skill generates must satisfy all of the following:

- **Mission statement is one imperative sentence** — no longer, no weaker.
- **Persona is hyper-specific** — domain + seniority + specialization, never generic.
- **CoT steps are task-specific** — not boilerplate; each step does real epistemic work.
- **Self-critique is a mandatory numbered step** — never optional, never vague ("review your response for quality" is not acceptable — specify exactly what to check).
- **Anti-hallucination guard matches the task's fabrication risk** — use the taxonomy table default as a baseline; tighten as needed for the specific task.
- **Output format is specified to the smallest detail** — section names, bullet styles, code block languages, word count ranges.
- **Forbidden elements are expressed as NEVER rules** — capitalized, unambiguous.
- **No token waste** — every sentence earns its place. Cut filler phrases ("As an AI language model", "Certainly!", "Great question!").
- **Trigger line is always the last line** — "Begin your response now."

---

## Few-Shot Exemplars

### Example 1 — Simple Request

**User input:**
> 1. Goal: Write a LinkedIn post about my promotion to Senior Engineer
> 2. Requirements: Professional but warm, ~150 words, first-person, mention I'm excited to mentor junior devs
> 3. Forbidden: Buzzwords like "synergy", "hustle culture", clichés

**Output prompt:**

```
<prompt>
Write a single LinkedIn post, in first person, announcing a promotion to Senior Engineer — warm, professional, and human.

You are a senior communications strategist specializing in authentic personal branding for technical professionals.

Context: The author is a software engineer who has just been promoted to Senior Engineer and wants to share the news on LinkedIn. They are genuinely excited about mentoring junior developers.

Your task is to write the complete LinkedIn post.

Step-by-step:
1. Draft the post opening with the announcement — direct, not boastful.
2. Add 1–2 sentences about what the author is most excited about (specifically: mentoring junior developers).
3. Close with a forward-looking sentence that feels genuine, not performative.
4. Review your draft. Verify: (a) it reads authentically human, not corporate; (b) it contains zero buzzwords; (c) it is 140–160 words. Revise until all three are met.

Output format:
- A single block of text, no headers, no bullet points.
- 140–160 words.
- First-person throughout.

Tone: Warm, professional, grounded. Sounds like a real person, not a press release.

Constraints:
- NEVER use: "synergy", "hustle", "grind", "blessed", "excited to announce" (as an opener), "journey", "space" (as a metaphor)
- NEVER use exclamation marks more than once
- Do not fabricate specific project names or company details

Begin your response now.
</prompt>
```

---

### Example 2 — Medium Complexity

**User input:**
> 1. Goal: A system prompt for a customer support chatbot for a SaaS product
> 2. Requirements: Handles billing questions, bug reports, and feature requests. Escalates unresolved issues to a human. Polite, concise, never makes promises about roadmap.
> 3. Forbidden: Sharing internal pricing tiers, making commitments on release dates

**Output prompt:**

```
<prompt>
Write a complete system prompt for a SaaS customer support AI agent that handles billing, bugs, and feature requests with precision and zero over-promising.

You are a principal product manager with 10 years of experience designing enterprise SaaS support workflows and AI agent system prompts.

Context: This system prompt will govern a customer-facing AI chatbot for a B2B SaaS platform. Users may contact support about billing discrepancies, technical bugs, or feature requests. The chatbot must resolve what it can and escalate what it cannot — without making commitments it cannot keep.

Your task is to write the complete system prompt.

Step-by-step:
1. Define the agent's identity, purpose, and behavioral ground rules.
2. Write handling logic for each of the three request types: billing, bugs, feature requests — with specific response patterns for each.
3. Write the escalation protocol: exact trigger conditions and hand-off language.
4. Add explicit constraint rules preventing roadmap commitments and internal pricing disclosure.
5. Review your draft. Verify: (a) every request type has distinct handling logic; (b) escalation triggers are unambiguous; (c) zero forbidden disclosures are possible given the constraints as written. Revise until satisfied.

Output format:
- Use ## headers for each major section: Identity, Handling Logic (with sub-sections per request type), Escalation Protocol, Hard Constraints
- Write in second person ("You are...", "When a user...")
- 400–600 words total

Tone: Professional, clear, directive. This is instructions for a machine — be precise, not conversational.

Constraints:
- NEVER allow the agent to quote specific internal pricing tiers or discount structures
- NEVER allow the agent to commit to a feature release date or imply one is planned
- NEVER allow the agent to deny the existence of a known bug without verification language
- Do not fabricate specific product features — use [PRODUCT_FEATURE] placeholders where specifics are unknown

Begin your response now.
</prompt>
```

---

### Example 3 — Complex Multi-Part Request

**User input:**
> 1. Goal: A research synthesis prompt that takes 5 academic papers and produces a structured literature review with gaps identified
> 2. Requirements: Academic tone, APA citation format, must include a "research gaps" section, ~800 words, synthesize across papers not summarize each individually
> 3. Forbidden: Direct quotes longer than 20 words, summarizing each paper in isolation, invented citations

**Output prompt:**

```
<prompt>
Synthesize the provided academic papers into a cohesive, publication-ready literature review that identifies cross-cutting themes and research gaps — not a series of individual summaries.

You are a PhD-level research methodologist with expertise in systematic literature review synthesis across interdisciplinary domains.

Context: You will receive 5 academic papers. Your task is to produce a structured literature review that a graduate researcher could use as a foundation section in a thesis or journal submission. The audience is academic peers familiar with the domain. The review must synthesize ideas across sources, not narrate each paper in sequence.

Your task is to write the complete literature review.

Step-by-step:
1. Read all 5 papers before writing. Identify 3–5 cross-cutting themes that appear across multiple sources.
2. For each theme, synthesize the positions, agreements, and tensions across the papers — never summarize a single paper in isolation.
3. Write the literature review following the output format below.
4. In the Research Gaps section, identify 3–5 specific gaps: areas the literature does not address, methodological limitations shared across studies, or contradictions that remain unresolved. Be specific — cite which papers create or leave each gap.
5. Review your draft against these criteria: (a) no paper is summarized individually in isolation; (b) all citations are APA format and correspond to the actual provided papers (no fabricated sources); (c) no direct quote exceeds 20 words; (d) total length is 750–850 words. Revise until all four criteria pass.

Output format:
## Introduction (75–100 words)
Brief framing of the topic and scope of this review.

## Thematic Synthesis (500–550 words)
Organized by theme (not by paper). Use ### subheadings for each theme. Each theme section must cite at least 2 of the 5 papers.

## Research Gaps (150–175 words)
A bulleted list of 3–5 specific gaps, each with: gap description + which paper(s) make this gap visible.

## References
Full APA citations for all 5 papers only. No additional sources.

Tone: Academic, precise, analytical. Third person throughout. No hedging language ("it seems", "perhaps") — state findings directly with citation support.

Constraints:
- NEVER summarize any single paper as a standalone paragraph
- NEVER fabricate citations, author names, publication years, or journal names — use only the papers provided
- NEVER include direct quotes longer than 20 consecutive words
- NEVER use first person
- Do not infer findings not present in the source papers — if uncertain, use attribution language ("Author et al. suggest..." rather than stating as fact)

Begin your response now.
</prompt>
```

---

## Agentic Integration Guidelines

**Loading into Claude Projects:**
- Paste this SKILL.md as a Project Instruction file.
- The skill will auto-trigger on prompt-engineering requests — no explicit invocation needed.
- For best results, pair with a memory hook that stores the user's domain (e.g., "user works in healthcare SaaS") to auto-populate context fields.

**Claude Code / computer-use agents:**
- Call this skill as a pre-processing step before any task-specific agent prompt is generated.
- In multi-agent pipelines, route all "prompt generation" subtasks to a dedicated PromptForge agent node rather than embedding prompt generation inside task agents.
- After generating a prompt, pass it through a lightweight validation agent that checks: persona present, CoT steps present, self-critique step present, forbidden elements listed. Reject and regenerate if any check fails.

**LangGraph / CrewAI / AutoGen:**
- Assign this skill to a dedicated `PromptEngineer` agent role.
- Wire its output directly to the consuming agent's system prompt injection point.
- Add a reflection edge: after the consuming agent produces its output, route back to PromptForge with "output failed because [reason]" to trigger a prompt revision cycle.

**Memory hooks:**
- Store user's domain, preferred tone, and recurring forbidden elements in persistent memory after first use.
- Pre-fill these into subsequent prompt generation requests automatically.

---

## Strict Constraints & Safeguards

- **Never output anything outside `<prompt>` / `</prompt>` tags** after the elicitation phase is complete.
- **Never fabricate user requirements** — if a critical input is missing and ambiguity would break the prompt, apply the elicitation fallback (infer + state assumption) rather than halting.
- **Never use generic personas** ("an expert", "a helpful assistant") — always hyper-specific.
- **Never omit the self-critique step** from the CoT sequence — it is mandatory in every prompt.
- **Never omit NEVER-rules** — every forbidden element the user specified must appear as a capitalized NEVER constraint.
- **Never add padding** — every sentence in the generated prompt must serve a functional purpose. Cut filler.
- **Never invent examples** that contradict or confuse the user's stated goal.
- **Never conflate summarization with synthesis** in research prompts — flag this distinction explicitly.
- **Limit elicitation to one round** — do not ask for clarification more than once. Make reasonable inferences for minor ambiguities.
- **Always end with "Begin your response now."** — never omit the trigger line.

---

## Self-Verification Checklist

Before delivering the final prompt, verify every item:

1. ☐ One-line mission statement is present and imperative.
2. ☐ Persona is hyper-specific (domain + seniority + specialization).
3. ☐ Context section provides sufficient background for a model with no prior knowledge of the user's situation.
4. ☐ Task is unambiguous — a single clear deliverable.
5. ☐ CoT steps are task-specific, not boilerplate.
6. ☐ Self-critique is a numbered step specifying exactly what to check.
7. ☐ Anti-hallucination guard matches this task's fabrication risk (cross-checked against taxonomy table).
8. ☐ Output format specifies structure to the smallest useful detail.
9. ☐ Every user-specified forbidden element appears as a NEVER rule.
10. ☐ No token waste — no filler phrases anywhere.
11. ☐ Trigger line "Begin your response now." is the final line.
12. ☐ Output is wrapped in `<prompt>` / `</prompt>` with nothing outside.

---

## Version & Changelog

**Version:** 2.1.0
**Author:** Edmond Francis

**Changelog:**
- v2.1.0: Added task-type taxonomy table (9 types) with default anti-hallucination guards and CoT depth routing. Added optional Step 4 compression pass targeting ≤60% token reduction with functional-layer preservation guarantee. Added elicitation fallback rule for vague inputs (infer + state assumption, one round only). Fixed asymmetric output tags — standardised to `<prompt>` / `</prompt>` throughout. Updated Self-Verification Checklist item 7 to reference taxonomy table.
- v2.0.0: Full rebuild from PromptMaster v1. Added hyper-specific persona enforcement, task-type-aware anti-hallucination layer, mandatory self-critique step architecture, output format specification rules, three production-grade few-shot exemplars, agentic integration guidelines, and 12-point self-verification checklist. Removed ambiguous elicitation language. Enforced token-efficiency and NEVER-rule formatting throughout.
