# Agentic System Specification — Enterprise AI CMS
## Intent-to-Experience Orchestration Layer

> The AI layer is a **constrained intelligence system**, not a free-form design generator. Every agent plans before it acts, validates before it outputs, and respects a strict schema contract. Intelligence is permitted; freedom is not.

---

## Table of Contents

1. [Core Principles](#1-core-principles)
2. [Execution Model](#2-execution-model)
3. [Agent Inventory](#3-agent-inventory)
4. [Agent Specifications](#4-agent-specifications)
   - [4.1 Intent Normalizer](#41-intent-normalizer)
   - [4.2 Layout Strategist](#42-layout-strategist)
   - [4.3 Component Composer](#43-component-composer)
   - [4.4 Quality Gate Agent](#44-quality-gate-agent)
   - [4.5 Refinement Engine](#45-refinement-engine)
   - [4.6 Pattern Learner](#46-pattern-learner)
5. [Memory Architecture](#5-memory-architecture)
6. [Guardrails System](#6-guardrails-system)
7. [Failure Model](#7-failure-model)
8. [Inter-Agent Communication Schema](#8-inter-agent-communication-schema)
9. [End-to-End Example](#9-end-to-end-example)
10. [Architectural Principles](#10-architectural-principles)

---

## 1. Core Principles

| Principle | Description |
|-----------|-------------|
| **Constrained Reasoning** | AI operates within a defined vocabulary (component registry + token system). It cannot invent components or bypass token constraints. |
| **Machine-Consumable Outputs** | Every agent output is strict JSON validated against a Zod schema. No natural language in agent-to-agent communication. |
| **Plan → Verify → Output** | Every agent executes a reasoning sequence before producing output. Reasoning is internal; only validated output is passed downstream. |
| **No Agent is Trusted Without Post-Validation** | The Quality Gate Agent independently validates every composition. No layout reaches the editor without passing validation. |
| **Human-in-the-Loop at Publish** | AI proposes; humans approve. The system reduces the cost of good decisions — it does not remove human judgment. |
| **Failure is Explicit** | Failed agent outputs are discarded. The system reverts to the last valid state and logs the failure trace. Silent errors are not permitted. |

---

## 2. Execution Model

### Pipeline

```
Editor Input (natural language)
        ↓
[INTENT AGENT] — Normalize ambiguous input into structured intent
        ↓
[PLANNING AGENT] — Transform intent into logical page structure (sections, not components)
        ↓
[COMPOSITION AGENT] — Map abstract sections to concrete SDC components + token configs
        ↓
[QUALITY GATE AGENT] — Validate schema, a11y, token constraints, structural integrity
        ↓
  ┌─────────────┐
  │  PASS?      │
  └──────┬──────┘
         │ YES                   │ NO
         ↓                       ↓
[EXPERIENCE UI]           [REFINEMENT LOOP]
Present to editor          Log failure, adjust,
                           retry (max 3 attempts)
                           or surface closest-pass
                           with flagged issues
         ↓
Editor approves or refines via conversation
         ↓
[REFINEMENT AGENT] — Apply minimal valid diff per editor feedback
         ↓
[QUALITY GATE AGENT] — Re-validate (must pass fully)
         ↓
Editor approves publish
         ↓
[PATTERN LEARNER] — Record accepted layout as future training example
        ↓
CI/CD pipeline (final quality gates) → Deploy
```

### Step Protocol

Every agent follows the same internal protocol before producing output:

1. **Interpret** — Parse and understand the input
2. **Reason** — Internal chain-of-thought (not exposed in output)
3. **Self-check** — Validate against own constraints before outputting
4. **Output** — Strict JSON only; no prose, no markdown, no explanation

---

## 3. Agent Inventory

| Agent | ID | Primary Skill Set | Output Schema |
|-------|----|-------------------|--------------|
| Intent Normalizer | `intent-normalizer` | `intent_inference`, `constraint_resolution` | `IntentSchema` |
| Layout Strategist | `layout-strategist` | `pattern_retrieval`, `structure_synthesis`, `consistency_check` | `PlanSchema` |
| Component Composer | `component-composer` | `component_resolution`, `token_constraint_solver`, `schema_enforcement` | `LayoutSchema` |
| Quality Gate Agent | `quality-gate` | `schema_enforcement`, `a11y_heuristics`, `consistency_check` | `ValidationResultSchema` |
| Refinement Engine | `iteration-engine` | `minimal_diff_engine`, `schema_enforcement` | `LayoutSchema` |
| Pattern Learner | `pattern-learner` | `pattern_extraction`, `performance_feedback` | `PatternSchema` |

---

## 4. Agent Specifications

### 4.1 Intent Normalizer

**ID:** `intent-normalizer`
**Role:** Convert ambiguous editor input into a structured, machine-consumable intent representation. This is the first transformation in the pipeline — raw language becomes structured signal.

**Skills:** `skill.intent_inference`, `skill.constraint_resolution`

**Input:**

```typescript
interface IntentNormalizerInput {
  raw_text: string;
  brand_context?: string;
  site_context?: {
    site_id: string;
    active_token_set: string;
    available_component_variants: string[];
  };
}
```

**Output (`IntentSchema`):**

```typescript
interface NormalizedIntent {
  goal: string;                  // Primary purpose: "product_launch" | "campaign" | "editorial" | "form" | ...
  audience: string;              // Target reader: "enterprise_buyer" | "consumer" | "developer" | "unknown"
  tone: NormalizedTone;          // "professional" | "playful" | "bold" | "editorial" | "minimal" | "premium"
  content_requirements: string[]; // ["hero", "testimonials", "newsletter_signup"]
  style_modifiers: string[];     // ["energetic", "earthy", "slightly_edgy"]
  brand: string;
  confidence: {
    goal: number;                // 0.0–1.0
    audience: number;
    tone: number;
    overall: number;
  };
  conflicts: ConflictRecord[];   // Any resolved contradictions, for transparency
}
```

**System Prompt Pattern:**

```
You are a high-precision intent normalization engine.

OBJECTIVE: Convert ambiguous human input into a structured intent representation.

EPISTEMOLOGY:
- Assume user input is incomplete. Do not invent specifics.
- Extract explicit signals first, then infer conservative implicit signals.
- Prefer omission (unknown) over hallucination for missing values.

NORMALIZATION RULES:
- tone_mapping: { luxury → premium, edgy → bold, fun → playful, clean → minimal }
- missing_string_values: "unknown"
- missing_array_values: []
- conflicting signals: return most dominant + lower confidence score + log in conflicts[]

CONSTRAINTS:
- No layout or UI generation
- No component references
- No stylistic prose
- No assumptions beyond available evidence

SELF-CHECK (before outputting):
1. Are all schema fields present?
2. Does any value contain hallucinated specifics not in the input?
3. Is tone within the allowed normalized set?
4. Are confidence scores calibrated (not all 1.0)?

OUTPUT: Valid JSON matching IntentSchema only. No prose.
```

**Failure Modes:**

| Condition | Behavior |
|-----------|---------|
| Ambiguous input | Returns low confidence scores; downstream planning uses conservative defaults |
| Contradictory intent | Resolves to dominant signal; logs conflict record |
| Empty or nonsensical input | Returns `goal: "unknown"`, confidence 0.0; surfaces clarification request to editor |

---

### 4.2 Layout Strategist

**ID:** `layout-strategist`
**Role:** Transform structured intent into a logical, high-converting page structure. Operates with section types only — no component references. Decides *what* the page needs, not *how* it will be rendered.

**Skills:** `skill.pattern_retrieval`, `skill.structure_synthesis`, `skill.consistency_check`

**Input:** `NormalizedIntent`

**Output (`PlanSchema`):**

```typescript
interface PagePlan {
  sections: PageSection[];
  rationale: string;   // One sentence: why this structure for this intent
  pattern_source: "retrieved" | "synthesized" | "hybrid";
  confidence: number;
}

interface PageSection {
  id: string;
  type: SectionType;   // "hero" | "value_prop" | "social_proof" | "cta" | "stats" | "form" | ...
  purpose: string;     // "Establish product identity and urgency"
  position: number;
  priority: "required" | "recommended" | "optional";
  content_hints: string[];  // ["looping video background", "strong single headline"]
}
```

**Section Logic (narrative structure):**

```
Position 1: Hero — Establishes context, creates immediate relevance
Position 2: Value proposition / problem statement — Builds understanding
Position 3: Credibility signals — Social proof, stats, third-party validation
Position 4: Supporting detail — Features, benefits, process
Position 5+: Action — CTAs, forms, newsletter
```

**System Prompt Pattern:**

```
You are a strategic layout planner trained on high-performing digital experiences.

OBJECTIVE: Transform structured intent into a logical, high-converting page structure.

INPUT: A normalized intent object with goal, audience, tone, and content requirements.

REASONING MODEL:
1. Match intent to known layout archetypes (product launch, editorial, demand gen, etc.)
2. Apply narrative structure: context → credibility → action
3. Include content_hints only when directly stated in intent; infer conservatively

SECTION TYPE VOCABULARY (use only these):
hero, intro_statement, value_proposition, feature_grid, stats_grid, testimonials,
testimonial_single, social_proof, media_showcase, process_steps, team_bio,
newsletter_signup, contact_form, cta_block, related_content, faq

DECISION BIAS:
- Prefer simplicity over complexity (5–7 sections for most pages)
- Prefer proven patterns over novelty
- Avoid redundant sections (never two consecutive CTA blocks)
- Mark sections as required / recommended / optional

SELF-CHECK (before outputting):
1. Does the flow match the stated goal?
2. Is the narrative sequence logical (context → credibility → action)?
3. Are there any unnecessary or redundant sections?
4. Are content_hints only drawn from explicit intent signals?

OUTPUT: Valid JSON matching PlanSchema only. No prose.
```

---

### 4.3 Component Composer

**ID:** `component-composer`
**Role:** Map abstract page sections to concrete, schema-validated SDC component configurations. This is the most constrained agent in the system — it can only produce outputs that are valid against the component registry.

**Skills:** `skill.component_resolution`, `skill.token_constraint_solver`, `skill.schema_enforcement`

**Input:** `PagePlan` + `ComponentRegistry` + `TokenConstraints` (for active brand)

**Output (`LayoutSchema`):**

```typescript
interface Layout {
  id: string;
  brand: string;
  sections: ComponentConfig[];
  metadata: {
    generated_at: string;
    intent_summary: string;
    pattern_source: string;
    validation_status: "pending" | "passed" | "failed";
  };
}

interface ComponentConfig {
  section_id: string;
  component: string;       // Must match registry component id
  props: Record<string, unknown>;   // Must include all required_props
  tokens: Record<string, string>;   // Must be within token_bindings constraints
  a11y_overrides?: Record<string, string>;
}
```

**System Prompt Pattern:**

```
You are a deterministic UI composition engine operating under strict system constraints.

OBJECTIVE: Map abstract layout sections to valid SDC component configurations.

HARD CONSTRAINTS (violations cause immediate output rejection):
1. ONLY use components from the provided registry (by exact id)
2. ALL required_props must be present with appropriate placeholder values
3. Token values must come from the component's token_bindings options
4. Do NOT invent components not in the registry
5. Do NOT output raw HTML, CSS, or JavaScript
6. Do NOT include unapproved external dependencies

FALLBACK STRATEGY (when exact match is unavailable):
- Choose the closest valid registry component
- Log the mismatch in metadata.component_gaps[]
- Never leave a required section unfulfilled

COMPONENT REGISTRY:
[Full JSON registry provided here at runtime]

TOKEN CONSTRAINTS FOR BRAND [brand_id]:
[Active brand token overlay provided here at runtime]

SELF-CHECK (before outputting):
1. Is every component id exactly matching a registry entry?
2. Are all required_props populated?
3. Are all token values from the allowed options in token_bindings?
4. Any hallucinated fields not in the registry schema?
5. Schema valid against LayoutSchema?

OUTPUT: Valid JSON matching LayoutSchema only. No prose. No explanation.
```

**Fallback Behavior:**

```typescript
// When a section cannot be satisfied exactly:
{
  "section_id": "sustainability_stats",
  "component": "stats_grid",          // closest match
  "props": { ... },
  "metadata": {
    "gap_logged": true,
    "original_intent": "sustainability_stats_with_icons",
    "gap_reason": "stats_grid_with_icons component not in registry"
  }
}
```

---

### 4.4 Quality Gate Agent

**ID:** `quality-gate`
**Role:** Independently validate every composition against all system rules before the editor sees it. This agent is the final check before the layout surfaces. It never modifies input — only evaluates and returns a verdict.

**Skills:** `skill.schema_enforcement`, `skill.a11y_heuristics`, `skill.consistency_check`, `skill.token_constraint_solver`

**Input:** `Layout` + `ComponentRegistry` + `TokenConstraints`

**Output (`ValidationResultSchema`):**

```typescript
interface ValidationResult {
  passed: boolean;
  score: number;                    // 0.0–1.0 overall quality score
  schema_valid: boolean;
  a11y_issues: A11yIssue[];
  token_violations: TokenViolation[];
  structural_issues: StructuralIssue[];
  component_gaps: ComponentGap[];   // Logged but don't fail validation
  recommended_fixes: Fix[];         // Only when passed: false
}

interface A11yIssue {
  type: "contrast" | "semantics" | "structure" | "missing_required";
  severity: "critical" | "serious" | "moderate" | "minor";
  location: string;
  wcag_criterion: string;
}
```

**Validation Criteria:**

| Check | Condition | Severity |
|-------|-----------|---------|
| Schema validity | All component configs match registry schemas | Critical (blocks) |
| Required props | All required_props populated | Critical (blocks) |
| Token bounds | All token values within allowed options | Critical (blocks) |
| A11y: heading structure | Logical heading hierarchy | Serious (blocks) |
| A11y: missing landmarks | Required ARIA landmarks present | Serious (blocks) |
| A11y: alt text requirement | Media fields with requires_alt flagged | Serious (blocks) |
| Structural coherence | No duplicate required section types | Moderate (blocks) |
| Component gaps | Sections with fallback components | Info (logged, doesn't block) |

**System Prompt Pattern:**

```
You are a strict validation engine. Your sole purpose is to prevent unsafe or invalid layouts
from reaching editors.

BIAS: Reject over accept. Strict over lenient. When uncertain, flag.

CONSTRAINTS:
- Do NOT modify the input layout
- Do NOT suggest alternative layouts
- Only evaluate and return a verdict with specific, actionable findings

EVALUATION SEQUENCE:
1. Schema validation (every component id matches registry, all required_props present)
2. Token constraint validation (every token value within allowed options)
3. Accessibility heuristics (heading structure, landmark roles, required alt text)
4. Structural coherence (logical section ordering, no contradictory components)

SELF-CHECK (before outputting):
1. Did I check every component in the layout?
2. Did I flag every violation, not just the first one found?
3. Are recommended_fixes specific enough to be actionable?
4. Is my passed verdict correct given all findings?

OUTPUT: Valid JSON matching ValidationResultSchema only.
```

---

### 4.5 Refinement Engine

**ID:** `iteration-engine`
**Role:** Apply editor feedback to an existing validated layout with the minimum necessary change. Preserve all unaffected structure. Maintain schema validity throughout.

**Skills:** `skill.minimal_diff_engine`, `skill.schema_enforcement`

**Input:**

```typescript
interface RefinementInput {
  current_layout: Layout;
  editor_instruction: string;         // "Make the hero darker", "Move testimonials above the stats"
  component_registry: ComponentConfig[];
  token_constraints: TokenConstraints;
}
```

**Output:** Updated `Layout` (with `metadata.refinement_history` appended)

**System Prompt Pattern:**

```
You are a precision refinement engine. You apply the smallest valid change to a layout.

OBJECTIVE: Apply the editor's instruction with minimal structural disruption.

REASONING MODEL:
1. Interpret the instruction — what specifically needs to change?
2. Locate the affected components in the layout
3. Determine the minimal valid diff (token change / prop change / section reorder / section swap)
4. Apply only the diff — preserve everything else exactly
5. Validate the result maintains schema compliance

CONSTRAINTS:
- Modify ONLY components directly affected by the instruction
- Preserve all unaffected sections exactly as they are
- Maintain schema validity at all times
- Token changes must remain within allowed bounds
- Do NOT regenerate the full layout unless the instruction explicitly requires it ("start over", "redesign this completely")

SELF-CHECK (before outputting):
1. Did I change only what was instructed?
2. Is the schema still valid?
3. Did I accidentally modify unrelated sections?
4. Are all token changes within allowed bounds?

OUTPUT: Full updated Layout JSON (not a diff — the complete updated layout). No prose.
```

**Refinement History:**

```json
"metadata": {
  "refinement_history": [
    {
      "instruction": "Make the hero darker",
      "change": "hero_banner.tokens.background: surface-primary → surface-dark",
      "timestamp": "2025-06-15T14:32:00Z"
    }
  ]
}
```

---

### 4.6 Pattern Learner

**ID:** `pattern-learner`
**Role:** Learn from accepted layouts to improve future composition quality. Detects reusable patterns, identifies component gaps, and correlates layouts with performance outcomes.

**Skills:** `skill.pattern_extraction`, `skill.performance_feedback`

**Input:**

```typescript
interface LearnerInput {
  layout: Layout;                    // An editor-approved layout
  outcome: "accepted" | "rejected";
  editor_refinement_count: number;   // How many refinements were needed before acceptance
  performance_data?: {               // Available after page has been live
    bounce_rate?: number;
    conversion_rate?: number;
    avg_time_on_page?: number;
    core_web_vitals?: { lcp: number; cls: number; fid: number; };
  };
}
```

**Responsibilities:**

- Save accepted layouts as tagged patterns in the Pattern Store
- Tag patterns by: intent type, brand, tone, goal, component set, performance tier
- Identify component gaps from `metadata.component_gaps[]` across accepted layouts
- Aggregate component gap data for quarterly DesignOps backlog review
- Correlate layout structures with performance outcomes over time
- Update few-shot examples for the Composition Agent with high-performing patterns

---

## 5. Memory Architecture

### Short-Term Memory (Session)

```typescript
interface SessionMemory {
  session_id: string;
  editor_id: string;
  brand_context: string;
  active_token_set: string;
  current_layout: Layout | null;
  conversation_history: ConversationTurn[];
  refinement_history: RefinementRecord[];
  validation_results: ValidationResult[];
}
```

Session memory is scoped to a single editing session and cleared on session end. It is not persisted to long-term storage.

### Long-Term Memory (Persistent)

| Store | Contents | Technology |
|-------|---------|-----------|
| **Pattern Store** | Accepted layouts tagged by intent, brand, goal, performance | PostgreSQL + pgvector |
| **Component Gap Log** | Unmet intent signals with frequency, context, and brand | PostgreSQL |
| **Performance Correlation Store** | Layout IDs mapped to measured outcome metrics | PostgreSQL |
| **Few-Shot Example Store** | High-performing patterns used as LLM examples | Retrieved at runtime per intent type |

### Vector Retrieval (Pattern Store)

Patterns are stored with embeddings of their intent + structural summary. At composition time:

```typescript
const relevantPatterns = await vectorStore.query({
  embedding: embed(normalizedIntent),
  filter: { brand: intent.brand, tone: intent.tone },
  topK: 3,
});
// Retrieved patterns are injected as few-shot examples into the Composition Agent prompt
```

---

## 6. Guardrails System

The guardrails operate at multiple levels to prevent the AI from producing unsafe output:

### Level 1 — System Prompt Constraints (per agent)

Every agent's system prompt explicitly lists what it cannot do. These are not suggestions — violations result in output rejection.

### Level 2 — Schema Validation (Zod)

Every agent output is validated against its output schema before being passed to the next agent:

```typescript
const result = LayoutSchema.safeParse(agentOutput);
if (!result.success) {
  throw new AgentOutputValidationError(agent.id, result.error.issues);
}
```

### Level 3 — Registry Enforcement (Composition Agent)

The Component Composer receives the registry as context and is instructed to use only registry component IDs. A post-composition check validates every component ID against the registry independently of the LLM.

### Level 4 — Token Constraint Validation (Quality Gate)

Every token value in the output is checked against the `token_bindings` for its component. Values outside the allowed set trigger a validation failure.

### Level 5 — Pre-Flight Quality Pipeline (before editor sees output)

Before the layout is surfaced to the editor, an external validation pipeline runs:

```
Layout JSON
    ↓
Drupal renderer (headless) → HTML output
    ↓
axe-core accessibility scan
    ↓
Pa11y WCAG AA check
    ↓
Contrast ratio check (all text/background pairs)
    ↓
CSP compatibility check
    ↓
Core Web Vitals estimation (Lighthouse CI)
    ↓
Visual regression diff vs. brand baseline
    ↓
All pass → Show to editor
Any fail → Discard; retry up to 3 times; surface closest-pass with flagged issues
```

### Level 6 — Human Approval Gate

No AI-composed layout is published without explicit editor approval. The system proposes; the human decides.

---

## 7. Failure Model

### Agent-Level Failures

```typescript
interface AgentFailure {
  agent_id: string;
  attempt_number: number;
  failure_type: "schema_invalid" | "validation_failed" | "llm_error" | "timeout";
  input_hash: string;  // For debugging without storing PII
  error_summary: string;
}
```

### Recovery Sequence

```
Agent fails
    ↓
Log failure with trace
    ↓
Retry with modified prompt (max 3 attempts)
    ↓
    ├── SUCCESS → continue pipeline
    └── ALL FAILED →
        ├── Surface "closest passing" layout with flagged issues (if any pass partially)
        └── OR surface graceful degradation: "Unable to generate a fully validated layout.
            Manual composition is available." + component palette
```

### State Management During Failure

- On any agent failure, the session state reverts to the last known-valid state
- No partial outputs are persisted to long-term memory
- Component gap logging still occurs even on failed runs (captures unmet intent data)

---

## 8. Inter-Agent Communication Schema

All inter-agent messages follow a typed envelope pattern:

```typescript
interface AgentMessage<T> {
  message_id: string;               // UUID
  session_id: string;
  source_agent: string;
  target_agent: string;
  timestamp: string;                // ISO 8601
  attempt: number;                  // 1–3
  payload: T;                       // Typed per agent pair
  metadata: {
    processing_time_ms: number;
    model_used: string;
    tokens_used: number;
    confidence?: number;
  };
}
```

Example: Intent Normalizer → Layout Strategist message:

```json
{
  "message_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "session_id": "sess_abc123",
  "source_agent": "intent-normalizer",
  "target_agent": "layout-strategist",
  "timestamp": "2025-06-15T14:30:00Z",
  "attempt": 1,
  "payload": {
    "goal": "product_launch",
    "audience": "consumer",
    "tone": "bold",
    "content_requirements": ["hero", "testimonials", "newsletter_signup"],
    "style_modifiers": ["energetic", "earthy"],
    "brand": "evergreen",
    "confidence": { "goal": 0.92, "audience": 0.78, "tone": 0.85, "overall": 0.85 },
    "conflicts": []
  },
  "metadata": {
    "processing_time_ms": 340,
    "model_used": "claude-sonnet-4-20250514",
    "tokens_used": 412
  }
}
```

---

## 9. End-to-End Example

**Editor input:**

```
"I need a product launch page for our eco-friendly sneaker.
Hero with looping video, three testimonial cards, a sustainability
stats grid, and a newsletter sign-up. Vibrant, earthy, slightly edgy."
```

**Step 1 — Intent Normalizer output:**

```json
{
  "goal": "product_launch",
  "audience": "consumer",
  "tone": "bold",
  "content_requirements": ["hero", "testimonials", "stats_grid", "newsletter_signup"],
  "style_modifiers": ["energetic", "earthy", "bold"],
  "brand": "evergreen",
  "confidence": { "goal": 0.95, "audience": 0.72, "tone": 0.88, "overall": 0.85 },
  "conflicts": []
}
```

**Step 2 — Layout Strategist output:**

```json
{
  "sections": [
    { "id": "s1", "type": "hero", "purpose": "Establish product identity; drive emotional connection", "position": 1, "priority": "required", "content_hints": ["looping video background", "strong single headline", "earthy palette"] },
    { "id": "s2", "type": "stats_grid", "purpose": "Prove sustainability credentials with concrete data", "position": 2, "priority": "required", "content_hints": ["3–4 impact statistics", "icons or illustrations"] },
    { "id": "s3", "type": "testimonials", "purpose": "Build trust with peer social proof", "position": 3, "priority": "required", "content_hints": ["3 cards", "name, photo, short quote"] },
    { "id": "s4", "type": "newsletter_signup", "purpose": "Capture leads before exit intent", "position": 4, "priority": "recommended", "content_hints": ["single email field", "value proposition in headline"] }
  ],
  "rationale": "Product launch structure: hook → proof → trust → capture",
  "pattern_source": "hybrid",
  "confidence": 0.89
}
```

**Step 3 — Component Composer output (abbreviated):**

```json
{
  "id": "layout_xyz789",
  "brand": "evergreen",
  "sections": [
    {
      "section_id": "s1",
      "component": "hero_banner",
      "props": { "headline": "[Product headline here]", "media": { "type": "video", "autoplay": true, "loop": true }, "cta": { "label": "Shop Now", "url": "#" } },
      "tokens": { "background": "surface-brand", "spacing": "spacing-section-lg" }
    },
    {
      "section_id": "s2",
      "component": "stats_grid",
      "props": { "items": [{ "value": "[%]", "label": "Recycled materials" }, { "value": "[kg]", "label": "CO₂ saved per pair" }, { "value": "[%]", "label": "Plastic-free packaging" }] },
      "tokens": { "columns": "3" }
    },
    {
      "section_id": "s3",
      "component": "testimonial_cards",
      "props": { "items": [{ "quote": "[Quote]", "name": "[Name]", "title": "[Title]" }, { "quote": "[Quote]", "name": "[Name]", "title": "[Title]" }, { "quote": "[Quote]", "name": "[Name]", "title": "[Title]" }] },
      "tokens": { "columns": "3" }
    },
    {
      "section_id": "s4",
      "component": "newsletter_signup",
      "props": { "headline": "Join the movement", "cta_label": "Sign me up" },
      "tokens": { "background": "surface-primary" }
    }
  ],
  "metadata": { "generated_at": "2025-06-15T14:30:45Z", "intent_summary": "product_launch eco-friendly sneaker", "validation_status": "pending" }
}
```

**Step 4 — Quality Gate passes.** All components valid, all tokens within bounds, a11y heuristics clear.

**Step 5 — Editor sees 3 layout options.** Chooses Option B, then:

```
"Swap testimonials to the left and make the hero darker."
```

**Step 6 — Refinement Engine** applies minimal diff:
- `hero_banner.tokens.background`: `surface-brand` → `surface-dark`
- Section `s2` and `s3` positions swapped

**Step 7 — Quality Gate re-validates.** Passes. Editor approves. Publish.

---

## 10. Architectural Principles

| Principle | Statement |
|-----------|----------|
| **Intelligence is allowed. Freedom is not.** | AI reasons flexibly within a strictly bounded output space. |
| **Structure is the product.** | The platform's value is valid, composable, quality-assured structure — not raw AI creativity. |
| **Skills define capability. Agents define behavior. The system defines truth.** | The component registry and token system are the authoritative source of what is possible. |
| **The pre-flight pipeline is not optional.** | No layout surfaces to an editor without passing all quality gates. No exceptions. |
| **Failure is explicit and recoverable.** | Every failure is logged with enough context to debug. The system never silently produces bad output. |
| **Human judgment is the final gate.** | AI proposes; humans approve. The system's value is reducing the cost and effort of good decisions — not replacing them. |

---

*Agentic System Specification · 34-EnterpriseAICMS · v1.0 · 2025+*
