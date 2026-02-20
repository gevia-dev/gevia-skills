---
description: Launch a multi-expert roundtable debate to analyze and improve any prompt. Generates diverse expert personas dynamically, runs parallel analysis agents, then synthesizes with a platform-specialist evaluator. Use when you want deep, multi-perspective critique of AI prompts, system prompts, or prompt engineering artifacts.
---

# Prompt Roundtable - Multi-Expert Debate Skill

Launch a panel of dynamically-generated expert agents to analyze, critique, and improve any prompt through structured parallel debate followed by expert evaluation.

## When to Use

- Reviewing or improving AI generation prompts (image, text, code)
- Auditing system prompts for quality, compliance, and effectiveness
- Getting multi-perspective feedback on prompt engineering decisions
- Comparing approaches to a prompt design challenge
- Any time you need structured expert debate on a prompt artifact

## Arguments

The skill accepts arguments in this format:
```
/prompt-roundtable [file_path_or_inline_prompt] [--experts N] [--context "additional context"]
```

- `file_path_or_inline_prompt`: Path to the file containing the prompt OR the prompt text itself
- `--experts N`: Number of experts (default: 5, min: 3, max: 7)
- `--context "..."`: Additional context about the platform, use case, constraints

If no arguments are provided, ask the user for the prompt to analyze.

## Workflow Overview

```
Phase 1: SETUP (you do this)
  ├── Read/receive the prompt to analyze
  ├── Analyze the prompt's domain and purpose
  ├── Generate N diverse expert personas dynamically
  └── Create task tracking with TaskCreate

Phase 2: PARALLEL DEBATE (subagents)
  ├── Launch N expert agents simultaneously (Task tool, run_in_background)
  ├── Each expert produces a structured analysis document
  └── Wait for all to complete

Phase 3: EVALUATION (subagent)
  ├── Launch evaluator agent with all expert documents
  ├── Evaluator synthesizes, resolves conflicts, prioritizes
  └── Produces actionable roadmap + rewritten prompt

Phase 4: DELIVERY
  ├── Present consolidated report to user
  └── Save full report to file if requested
```

## Phase 1: Setup — Dynamic Persona Generation

### Step 1: Read the Prompt

If a file path is provided, read the file. If inline text, use directly. You MUST have the full prompt text before proceeding.

### Step 2: Analyze Domain

From the prompt text, identify:
- **Domain**: What field does this prompt operate in? (image generation, text generation, code generation, chatbot, RAG, etc.)
- **Purpose**: What is the prompt trying to achieve?
- **Key concerns**: What aspects matter most? (quality, consistency, safety, cost, speed, compliance)
- **Technical stack**: What models/APIs does it target?

### Step 3: Generate Expert Personas

Generate exactly N expert personas. Follow these rules:

**DIVERSITY REQUIREMENTS (non-negotiable):**
1. Each expert MUST have a different professional background
2. Each expert MUST analyze from a different angle/concern
3. At least one expert MUST be a domain specialist (deep in the prompt's specific field)
4. At least one expert MUST be a prompt engineering / AI specialist (meta-prompt analysis)
5. At least one expert MUST represent the end-user perspective
6. Exactly one expert MUST be a "curious novice" — someone with 6-12 months experience who brings fresh eyes, asks "dumb questions" that expose blind spots, and isn't afraid to say "this feels wrong"
7. NO two experts should share the same primary concern

**PERSONA TEMPLATE:**
For each expert, define:
```
Name: [Generated realistic name]
Title: [Specific role, not generic]
Experience: [Years + notable achievements/companies]
Perspective: [What unique angle they bring]
Primary concern: [The ONE thing they care most about]
Tone: [How they communicate — technical, data-driven, casual, etc.]
```

**EXAMPLE PERSONA GENERATION** (for an image generation prompt):
- Photography/Visual Arts expert → composition, lighting, aesthetics
- AI/ML Prompt Engineering expert → token efficiency, model behavior, compliance
- Social Media/Marketing expert → engagement, platform trends, conversion
- Design Systems/Color Theory expert → consistency, brand coherence, systematic approach
- Curious Novice Creator → fresh eyes, "would this fool me?", trend awareness

**EXAMPLE PERSONA GENERATION** (for a chatbot system prompt):
- Conversational UX Designer → dialogue flow, user satisfaction, error recovery
- AI Safety / Alignment Researcher → guardrails, edge cases, adversarial inputs
- Customer Support Operations Lead → resolution rate, escalation paths, tone
- Linguistics / Pragmatics Expert → ambiguity, implicature, cultural sensitivity
- Curious Novice User → "this confused me", "I expected X but got Y"

**EXAMPLE PERSONA GENERATION** (for a RAG/retrieval prompt):
- Information Retrieval Researcher → recall, precision, relevance ranking
- Prompt Engineering Specialist → context window management, instruction clarity
- Domain Expert (specific field) → factual accuracy, terminology, completeness
- DevOps/Systems Engineer → latency, cost, failure modes, scaling
- Curious Junior Developer → "why does it hallucinate here?", practical concerns

### Step 4: Create Tasks

Use TaskCreate for tracking:
```
Task 1: Launch N expert agents in parallel debate
Task 2: Collect expert documents
Task 3: Launch evaluator agent
Task 4: Present final consolidated report
```

Set dependencies: 2 blocked by 1, 3 blocked by 2, 4 blocked by 3.

## Phase 2: Parallel Debate — Expert Agent Dispatch

### Expert Agent Prompt Template

For EACH expert, dispatch a Task agent with `run_in_background: true` using this template. Customize the sections in [BRACKETS] for each expert:

```
You are **[EXPERT NAME]**, [EXPERT TITLE AND CREDENTIALS].
[2-3 sentences of background that establish authority and perspective.]

## YOUR TASK

Analyze the following prompt/system-prompt used in [DOMAIN CONTEXT].
Your unique perspective is: [PRIMARY CONCERN AND ANGLE].

## THE PROMPT TO ANALYZE

```
[FULL PROMPT TEXT]
```

## ADDITIONAL CONTEXT

[Any platform context, technical constraints, user base info provided by the user]

## PRODUCE A DOCUMENT WITH THESE SECTIONS:

### 1. OVERALL ASSESSMENT (Grade A-F)
Rate from your specific expert perspective. Justify the grade.

### 2. WHAT WORKS WELL
What aspects demonstrate genuine expertise in your domain?
Be specific — cite exact sections/rules that are strong.

### 3. CRITICAL GAPS
What is MISSING that someone in your field would consider essential?
Prioritize by impact.

### 4. DETAILED CRITIQUE
[2-3 SECTIONS CUSTOMIZED TO THIS EXPERT'S DOMAIN]
These should be deep-dive analyses specific to what this expert uniquely brings.
Examples:
- Photography expert → "Film Stock Analysis", "Composition & Lighting Gaps"
- Prompt engineer → "Token Efficiency", "Model-Specific Optimization", "Compliance Rate"
- Social media expert → "Engagement Patterns", "Platform-Specific Gaps"
- Design systems → "Consistency Architecture", "Cross-unit Coherence"
- Novice → "Would This Fool Me?", "Things That Confuse Me", "The Vibe Check"

### 5. CONCRETE IMPROVEMENT PROPOSALS
Provide 3-5 specific, actionable changes with:
- What to change (before/after if possible)
- Why (grounded in your expertise)
- Expected impact

### 6. REWRITTEN SECTIONS (optional)
If you think you can dramatically improve specific sections, rewrite them.
Include your reasoning.

Write in English. Be thorough, opinionated, and specific.
Don't hold back criticism — this is a professional peer review.
```

**IMPORTANT RULES FOR EXPERT DISPATCH:**
- ALL experts launch in parallel (single message with multiple Task tool calls)
- ALL use `run_in_background: true`
- ALL use `subagent_type: "general-purpose"`
- The novice expert should be explicitly told to use casual language and not try to sound like an expert
- Each expert's "DETAILED CRITIQUE" sections should be UNIQUE — no two experts analyze the same aspect

### For the Novice Expert Specifically

The novice gets a different tone in their prompt template:

```
You are **[NAME]**, a [AGE]-year-old [ROLE] with about [6-12] months of experience.
You don't have deep technical knowledge, but you have an INCREDIBLE eye for
what works and what doesn't. You [RELEVANT DAILY ACTIVITY]. You know what makes
YOU [REACT TO THE DOMAIN], and you're not afraid to call things out.

[Rest of template but with these unique sections:]

### 4. THINGS THAT CONFUSE ME (Questions for the Experts)
Ask 5+ questions that you genuinely don't understand or that seem weird.
These "dumb questions" are the most valuable part of your contribution.

### 5. WHAT MY [PEERS] WOULD SAY
What would others in your position think?

### 6. THE VIBE CHECK
Does this feel right for [YEAR]? Or does it feel dated/stuck?

Write in English. Be authentic, unfiltered, and honest. Use casual language.
Your fresh perspective is WHY you're on this panel. Don't try to sound like
an expert. Sound like YOU.
```

## Phase 3: Evaluation — Synthesizer Agent

After ALL expert agents complete, launch the evaluator.

### Evaluator Agent Prompt Template

```
You are **[GENERATED NAME]**, the Lead [RELEVANT ROLE] at [PLATFORM/PROJECT NAME].
You've been building this [platform/system] for [N] years. You know every technical
constraint, every business context, every user need.

Your job is NOT to pick a winner. Your job is to SYNTHESIZE the best ideas from
all experts into a prioritized, actionable roadmap.

## PLATFORM CONTEXT
[Inject all relevant technical constraints, architecture details, business context,
user base info. This is what makes the evaluator different from the experts —
they have theoretical knowledge, YOU have practical platform knowledge.]

## THE EXPERT DOCUMENTS

[Paste ALL expert documents here, clearly labeled by expert name and perspective]

## YOUR DELIVERABLE

### 1. CONSENSUS MAP
Where do 3+ experts agree? These are high-confidence recommendations.

### 2. CONFLICT RESOLUTION
Where do experts disagree? Make a decision with reasoning for each conflict.

### 3. PRIORITIZED ROADMAP
Organize ALL worthwhile proposals into 3 tiers:

**Tier 1: IMMEDIATE (low effort, high impact, no architecture changes)**
For each item: what to change, effort estimate, expected impact, which expert(s) proposed it.

**Tier 2: NEAR-TERM (medium effort, requires some changes)**
For each item: what to change, what needs to change architecturally, effort estimate, impact.

**Tier 3: FUTURE (high effort, transformative)**
For each item: what to build, architecture implications, effort, long-term impact.

### 4. THE REWRITTEN PROMPT
Taking the best from all experts, write a NEW version of the prompt that
could be deployed as a Tier 1 change (no architecture changes, just better
prompt text). This should be production-ready.

### 5. WHAT TO IGNORE
Which proposals should NOT be implemented? Why?

### 6. OPEN QUESTIONS
What needs user testing or A/B data to answer?

Write in English. Be decisive — you are the tiebreaker.
```

## Phase 4: Delivery

1. Mark all tasks as completed
2. Present the evaluator's report to the user with a summary table
3. Highlight the top 3 most impactful recommendations
4. Ask if the user wants the full report saved to a file

## Integration with Other Skills

This skill works well with:
- **superpowers:subagent-driven-development** — If the roundtable produces an implementation plan, use SDD to execute it
- **superpowers:brainstorming** — Use BEFORE this skill to refine what prompt to analyze
- **superpowers:writing-plans** — Use AFTER this skill to turn the roadmap into implementation tasks
- **superpowers:requesting-code-review** — Use AFTER implementing changes from the roadmap

## Tips for Best Results

1. **More context = better evaluator output.** The evaluator's quality depends heavily on knowing the platform constraints. Always inject technical context.
2. **5 experts is the sweet spot.** 3 feels thin, 7 causes too much overlap. Use 5 unless the user requests otherwise.
3. **Don't rush Phase 3.** The evaluator agent is the most important one. Give it a thorough prompt with ALL expert documents.
4. **The novice is not filler.** Their "dumb questions" and "vibe check" consistently surface issues that domain experts miss. Never skip the novice.
5. **Save the report.** These reports are dense and valuable. Always offer to save to a file.
