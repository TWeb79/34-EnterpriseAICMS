# Skills Specification — Enterprise AI CMS
## Capability Library for the Agentic Orchestration Layer

> Skills are the **atomic, reusable capabilities** that agents compose to complete tasks. Each skill has a single responsibility, is stateless, is schema-bound, and is independently testable. Agents orchestrate skills — skills do not contain agent logic.

---

## Table of Contents

1. [Skill Design Principles](#1-skill-design-principles)
2. [Skill Interface Contract](#2-skill-interface-contract)
3. [Cognitive Skills](#3-cognitive-skills)
4. [Structural Skills](#4-structural-skills)
5. [Component System Skills](#5-component-system-skills)
6. [Quality Skills](#6-quality-skills)
7. [Refinement Skills](#7-refinement-skills)
8. [Learning Skills](#8-learning-skills)
9. [Skill Composition Rules](#9-skill-composition-rules)
10. [Anti-Patterns](#10-anti-patterns)
11. [Skill Registry](#11-skill-registry)
12. [Implementation Guide](#12-implementation-guide)

---

## 1. Skill Design Principles

| Principle | Description | Why It Matters |
|-----------|-------------|---------------|
| **Single Responsibility** | Each skill does exactly one thing | Easier to test, debug, and replace independently |
| **Stateless** | Skills read input and produce output; no shared mutable state | Enables parallel execution, eliminates hidden dependencies |
| **Pure** | Same input always produces the same output (within determinism class) | Enables caching, regression testing, and output validation |
| **Schema-Bound** | Every input and output is typed and validated | Prevents silent type errors; enables compile-time safety |
| **Independently Testable** | Skills can be unit-tested in isolation without agent infrastructure | Fast feedback loops; high test coverage is achievable |
| **Composable** | Skills can be combined in pipelines without modification | Agents are skill orchestrators; new behaviors emerge from composition |

### Determinism Classes

| Class | Description | Examples |
|-------|-------------|---------|
| `deterministic` | Identical input always produces identical output. No LLM involvement. | `schema_enforcement`, `token_constraint_solver`, `consistency_check` |
| `semi-deterministic` | Structured LLM output with schema validation. Outputs may vary slightly but must always be schema-valid. | `intent_inference`, `component_resolution`, `a11y_heuristics` |

---

## 2. Skill Interface Contract

All skills conform to this base interface:

```typescript
interface Skill<TInput, TOutput> {
  name: string;
  description: string;
  determinism: "deterministic" | "semi-deterministic";
  input_schema: ZodSchema<TInput>;
  output_schema: ZodSchema<TOutput>;
  failure_modes: FailureMode[];
  side_effects: "none";   // Skills NEVER have side effects
  execute(input: TInput): Promise<TOutput>;
}

interface FailureMode {
  condition: string;
  behavior: string;
  output: "empty_array" | "null" | "error_throw" | "low_confidence_result";
}
```

**JSON Registration Format:**

```json
{
  "name": "skill.schema_enforcement",
  "description": "Validate data against a JSON schema; return validity and error list",
  "determinism": "deterministic",
  "input_schema": "SchemaEnforcementInput",
  "output_schema": "SchemaEnforcementOutput",
  "failure_modes": [
    {
      "condition": "input data does not match schema",
      "behavior": "return valid: false with detailed error list",
      "output": "low_confidence_result"
    }
  ],
  "side_effects": "none"
}
```

---

## 3. Cognitive Skills

Cognitive skills interpret and normalize ambiguous human input into structured signals.

---

### `skill.intent_inference`

**Purpose:** Extract structured intent signals from ambiguous, incomplete human input.

**Determinism:** `semi-deterministic`

**Input:**

```typescript
interface IntentInferenceInput {
  text: string;
  context?: {
    brand?: string;
    site_type?: string;
    editor_history?: string[];  // Last 3 accepted page intents for few-shot context
  };
}
```

**Output:**

```typescript
interface IntentInferenceOutput {
  goal: string;
  audience: string;
  tone: NormalizedTone;
  content_requirements: string[];
  style_modifiers: string[];
  confidence: {
    goal: number;       // 0.0–1.0
    audience: number;
    tone: number;
    overall: number;
  };
}

type NormalizedTone = "professional" | "playful" | "bold" | "editorial" | "minimal" | "premium" | "technical" | "unknown";
```

**Behavior Rules:**

- Extract explicit signals first
- Infer implicit signals conservatively (low confidence)
- Assign confidence scores calibrated to actual signal strength (not all 1.0)
- Use `"unknown"` for missing string values, `[]` for missing arrays
- Never hallucinate specifics not present in input

**Failure Modes:**

| Condition | Behavior |
|-----------|---------|
| Ambiguous or vague input | Return with low confidence scores (< 0.5); downstream planning uses conservative defaults |
| Contradictory signals | Return dominant signal + log conflict; reduce overall confidence |
| Empty input | Return all fields as `"unknown"` / `[]`; confidence 0.0 |

---

### `skill.constraint_resolution`

**Purpose:** Resolve conflicts between extracted or explicitly provided constraints, producing a clean, consistent constraint set.

**Determinism:** `deterministic`

**Input:**

```typescript
interface ConstraintResolutionInput {
  constraints: Record<string, unknown>;
  priority_rules?: PriorityRule[];  // Optional: custom resolution rules
}

interface PriorityRule {
  field: string;
  priority: "explicit_over_inferred" | "first_wins" | "last_wins" | "highest_confidence";
}
```

**Output:**

```typescript
interface ConstraintResolutionOutput {
  resolved_constraints: Record<string, unknown>;
  conflicts: ConflictRecord[];
}

interface ConflictRecord {
  field: string;
  values_found: unknown[];
  resolved_to: unknown;
  reason: string;
}
```

**Behavior Rules:**

- Prioritize explicit user input over inferred values by default
- Fall back to system defaults only when safe and documented
- Never silently override conflicts — log every resolution
- Return the full conflict list even when resolution is unambiguous

---

## 4. Structural Skills

Structural skills define the architecture and narrative of a page — what sections it needs, in what order, and why.

---

### `skill.pattern_retrieval`

**Purpose:** Retrieve the most relevant layout patterns from the Pattern Store based on semantic similarity to current intent.

**Determinism:** `semi-deterministic`

**Input:**

```typescript
interface PatternRetrievalInput {
  intent: NormalizedIntent;
  brand?: string;
  top_k: number;          // Default: 3
  min_confidence: number; // Default: 0.65
}
```

**Output:**

```typescript
interface PatternRetrievalOutput {
  patterns: RetrievedPattern[];
  retrieval_method: "vector_search" | "keyword_fallback" | "empty";
}

interface RetrievedPattern {
  id: string;
  similarity_score: number;
  intent_tags: string[];
  sections: SectionConfig[];
  performance_tier?: "high" | "medium" | "unknown";
  brand_context: string;
}
```

**Behavior Rules:**

- Use vector similarity search against intent embeddings in the Pattern Store
- Return only patterns above `min_confidence` threshold
- Sort by: `performance_tier (high first)` then `similarity_score`
- Return empty array (not error) when no patterns qualify — triggers synthesis fallback

**Failure Modes:**

| Condition | Behavior |
|-----------|---------|
| No matching patterns | Return empty array; downstream uses `skill.structure_synthesis` |
| Vector store unavailable | Fall back to keyword matching; flag `retrieval_method: "keyword_fallback"` |

---

### `skill.structure_synthesis`

**Purpose:** Synthesize a logical page structure from intent when no retrieved pattern is available or when intent requires a novel arrangement.

**Determinism:** `semi-deterministic`

**Input:**

```typescript
interface StructureSynthesisInput {
  intent: NormalizedIntent;
  reference_patterns?: RetrievedPattern[];  // Optional: use as structural hints
  max_sections?: number;                    // Default: 7
}
```

**Output:**

```typescript
interface StructureSynthesisOutput {
  sections: PageSection[];
  rationale: string;
  synthesis_method: "retrieved" | "synthesized" | "hybrid";
}

interface PageSection {
  id: string;
  type: SectionType;
  purpose: string;
  position: number;
  priority: "required" | "recommended" | "optional";
  content_hints: string[];
}
```

**Narrative Structure Logic:**

```
Position 1:   Hero — Context, identity, initial hook
Positions 2–3: Value / Proof — Credibility signals, statistics, features
Positions 4–5: Detail — Supporting content, process, comparison
Positions 6–7: Action — CTAs, forms, conversion elements
```

**Behavior Rules:**

- Prefer simplicity: 5–7 sections for most page types
- Prefer proven section sequences over novel arrangements
- Mark sections as `required / recommended / optional` (enables AI to omit optional sections on simpler pages)
- Content hints drawn only from explicit intent signals — no invented specifics

---

## 5. Component System Skills

Component system skills operate on the concrete layer: mapping abstract sections to real components, validating schemas, and enforcing token constraints.

---

### `skill.component_resolution`

**Purpose:** Map an abstract page section to the most appropriate concrete SDC component from the registry.

**Determinism:** `semi-deterministic`

**Input:**

```typescript
interface ComponentResolutionInput {
  section: PageSection;
  registry: ComponentDefinition[];
  brand_context?: string;
  preference_hints?: string[];  // Optional: preferred component IDs if multiple qualify
}
```

**Output:**

```typescript
interface ComponentResolutionOutput {
  component_id: string;
  confidence: number;        // 0.0–1.0
  match_type: "exact" | "close" | "fallback";
  gap_logged: boolean;
  gap_reason?: string;
}
```

**Behavior Rules:**

- Match section `type` and `purpose` to component `category` and `description`
- Prefer components with high reuse across the site fleet (from registry metadata)
- In case of multiple valid matches, prefer fully stable components (`status: "stable"`) over beta
- When no exact match exists, choose the closest valid component and log the gap
- Never return a component ID not in the registry
- Never return an error when a fallback exists — always resolve to something valid

**Failure Modes:**

| Condition | Behavior |
|-----------|---------|
| No matching component | Return closest valid + `gap_logged: true` + `match_type: "fallback"` |
| Multiple equally qualified components | Return highest-confidence match; log alternatives in metadata |

---

### `skill.schema_enforcement`

**Purpose:** Validate a data structure against a JSON schema (or Zod schema). Return validity verdict and detailed error list.

**Determinism:** `deterministic`

**Input:**

```typescript
interface SchemaEnforcementInput {
  data: unknown;
  schema: ZodSchema | JSONSchema;
  strict: boolean;   // true: reject unknown keys; false: allow additional keys
}
```

**Output:**

```typescript
interface SchemaEnforcementOutput {
  valid: boolean;
  errors: SchemaError[];
}

interface SchemaError {
  path: string;        // JSON path to the invalid field
  expected: string;
  received: string;
  message: string;
}
```

**Behavior Rules:**

- Strict validation — no coercion of types
- No silent fixes — if data is invalid, it is invalid; do not attempt to repair
- Return all errors, not just the first one found
- Empty error array only when `valid: true`

---

### `skill.token_constraint_solver`

**Purpose:** Validate that all token values in a component configuration are within the allowed options defined in the component registry, and verify accessibility safety.

**Determinism:** `deterministic`

**Input:**

```typescript
interface TokenConstraintSolverInput {
  component_id: string;
  token_config: Record<string, string>;
  registry_definition: ComponentDefinition;  // Source of allowed token options
  brand_token_set?: BrandTokenSet;           // Optional: brand-specific allowed values
}
```

**Output:**

```typescript
interface TokenConstraintSolverOutput {
  valid: boolean;
  violations: TokenViolation[];
}

interface TokenViolation {
  token_key: string;
  provided_value: string;
  allowed_values: string[];
  violation_type: "out_of_range" | "unknown_token" | "brand_conflict";
}
```

**Behavior Rules:**

- Validate against `token_bindings` in the component registry definition
- If `brand_token_set` is provided, further restrict to brand-allowed values
- A token value not in `token_bindings` is always a violation, regardless of whether it exists in the global token system
- Report all violations, not just the first

---

## 6. Quality Skills

Quality skills detect problems in generated layouts — accessibility issues, structural inconsistencies, and coherence failures.

---

### `skill.a11y_heuristics`

**Purpose:** Detect likely accessibility risks in a generated layout configuration through heuristic analysis of component arrangements and token configurations.

**Determinism:** `semi-deterministic`

**Input:**

```typescript
interface A11yHeuristicsInput {
  layout: Layout;
  registry: ComponentDefinition[];
}
```

**Output:**

```typescript
interface A11yHeuristicsOutput {
  risks: A11yRisk[];
  estimated_pass_probability: number;  // 0.0–1.0
}

interface A11yRisk {
  type: "contrast" | "semantics" | "structure" | "missing_required" | "aria" | "keyboard";
  severity: "critical" | "serious" | "moderate" | "minor";
  location: string;            // Component ID + section context
  wcag_criterion: string;      // e.g., "1.4.3 Contrast (Minimum)"
  description: string;
  suggested_fix?: string;
}
```

**Heuristic Detection Rules:**

| Check | Condition | Severity |
|-------|-----------|---------|
| Heading structure | First component is not hero/banner; or multiple h1 components | Serious |
| Required alt text | Media component present + `requires_alt: true` in registry, but no alt text prop provided | Serious |
| Landmark structure | No banner landmark in layout | Moderate |
| Contrast risk | Dark token applied to component with light text by default (requires token lookup) | Critical |
| Interactive elements | Form component without visible label props | Critical |
| Skip navigation | Long content without anchor or skip-nav component | Minor |

**Behavior Rules:**

- Conservative: flag risks rather than ignore borderline cases
- Do not modify the layout — heuristics only flag, they do not fix
- Every risk must include a `wcag_criterion` reference
- Critical severity risks should block unless resolved

---

### `skill.consistency_check`

**Purpose:** Ensure the internal coherence and logical integrity of a layout — no duplicate required sections, no contradictory components, sensible section ordering.

**Determinism:** `deterministic`

**Input:**

```typescript
interface ConsistencyCheckInput {
  layout: Layout;
  registry: ComponentDefinition[];
}
```

**Output:**

```typescript
interface ConsistencyCheckOutput {
  consistent: boolean;
  issues: ConsistencyIssue[];
}

interface ConsistencyIssue {
  type: "duplicate_required" | "contradictory_components" | "ordering_violation" | "orphaned_cta";
  severity: "error" | "warning";
  location: string;
  description: string;
}
```

**Consistency Rules:**

| Rule | Condition | Severity |
|------|-----------|---------|
| No duplicate required sections | Two or more hero components | Error |
| No consecutive CTAs | Two CTA blocks adjacent with no content between | Warning |
| Conversion element present | Page with hero and content but no CTA or form | Warning |
| CTA before value proposition | CTA appears before any credibility or value content | Warning |
| Empty required slots | Required component props are empty placeholder strings only | Error |

---

## 7. Refinement Skills

Refinement skills apply targeted changes to existing layouts with minimal disruption to unaffected structure.

---

### `skill.minimal_diff_engine`

**Purpose:** Interpret an editor's refinement instruction and compute the smallest valid change set to apply to the current layout, preserving all unaffected components and sections.

**Determinism:** `semi-deterministic`

**Input:**

```typescript
interface MinimalDiffInput {
  current_layout: Layout;
  instruction: string;      // "Make the hero darker", "Move testimonials before the stats"
  registry: ComponentDefinition[];
  token_constraints: TokenConstraints;
}
```

**Output:**

```typescript
interface MinimalDiffOutput {
  updated_layout: Layout;
  changes: ChangeRecord[];
  change_type: "token_update" | "prop_update" | "section_reorder" | "component_swap" | "section_add" | "section_remove";
}

interface ChangeRecord {
  section_id: string;
  component: string;
  field_path: string;          // JSON path of changed field
  previous_value: unknown;
  new_value: unknown;
  change_reason: string;
}
```

**Behavior Rules:**

- Identify the narrowest possible interpretation of the instruction
- Modify only components directly referenced or implied by the instruction
- Preserve all other sections exactly as they are (position, props, tokens)
- Validate that all changes remain within token bounds
- If the instruction requires a full regeneration ("start over", "redesign completely"), flag this and escalate to the Composition Agent rather than attempting a diff

**Instruction → Change Mapping Examples:**

| Instruction | Change Type | Example |
|-------------|------------|---------|
| "Make the hero darker" | `token_update` | `hero_banner.tokens.background`: `surface-primary` → `surface-dark` |
| "Move testimonials before the stats" | `section_reorder` | Swap `position` values of testimonials and stats_grid sections |
| "Make it more energetic" | `token_update` (multiple) | Hue shift +10°, spacing +0.1× (within safe range) |
| "Add a newsletter sign-up" | `section_add` | Append `newsletter_signup` component to end of layout |
| "Remove the FAQ section" | `section_remove` | Remove section with `type: faq` |
| "Swap the hero to a split layout" | `component_swap` | Replace `hero_banner` with `hero_split` (if in registry) |

---

## 8. Learning Skills

Learning skills observe accepted layouts and correlate them with outcomes to improve future composition quality.

---

### `skill.pattern_extraction`

**Purpose:** Identify reusable layout patterns from a set of editor-accepted layouts and store them in the Pattern Store for future retrieval.

**Determinism:** `semi-deterministic`

**Input:**

```typescript
interface PatternExtractionInput {
  layouts: AcceptedLayout[];
  min_frequency?: number;  // Minimum times a pattern must appear to be stored (default: 2)
}

interface AcceptedLayout {
  layout: Layout;
  intent_tags: string[];
  brand: string;
  editor_refinement_count: number;
  accepted_at: string;
}
```

**Output:**

```typescript
interface PatternExtractionOutput {
  patterns: ExtractedPattern[];
  component_gaps: ComponentGapSummary[];
}

interface ExtractedPattern {
  id: string;
  section_sequence: string[];        // ["hero", "stats_grid", "testimonials", "newsletter_signup"]
  component_sequence: string[];      // ["hero_banner", "stats_grid", "testimonial_cards", "newsletter_signup"]
  intent_tags: string[];
  brand: string;
  frequency: number;
  avg_refinement_count: number;      // Lower = better (fewer corrections needed)
  quality_tier: "high" | "medium" | "low";
}

interface ComponentGapSummary {
  intent_type: string;
  frequency: number;
  most_common_fallback: string;
  suggested_component_name: string;
}
```

**Behavior Rules:**

- Cluster layouts by section type sequence (not component IDs — patterns are structural)
- A pattern with `avg_refinement_count < 1` is high quality; `> 3` is low quality
- Patterns are tagged with both intent signals and structural features for retrieval
- Component gaps are extracted from `metadata.component_gaps[]` across all input layouts
- Gaps with frequency ≥ 3 across different brands are escalated as high-priority suggestions

---

### `skill.performance_feedback`

**Purpose:** Correlate accepted layout patterns with measured content performance outcomes to identify high-performing structural patterns.

**Determinism:** `deterministic`

**Input:**

```typescript
interface PerformanceFeedbackInput {
  layout_id: string;
  brand: string;
  metrics: {
    bounce_rate?: number;            // 0.0–1.0
    conversion_rate?: number;        // 0.0–1.0
    avg_time_on_page_seconds?: number;
    scroll_depth_percent?: number;
    core_web_vitals?: {
      lcp_ms: number;
      cls_score: number;
      fid_ms: number;
    };
  };
  measurement_period_days: number;
}
```

**Output:**

```typescript
interface PerformanceFeedbackOutput {
  layout_id: string;
  performance_score: number;         // 0.0–1.0 composite score
  performance_tier: "high" | "medium" | "low";
  signals: PerformanceSignal[];
  update_pattern_store: boolean;     // true if pattern tier should be updated
}

interface PerformanceSignal {
  metric: string;
  value: number;
  benchmark_comparison: "above" | "at" | "below";
  weight_in_score: number;
}
```

**Scoring Model:**

| Metric | Weight | High Tier Threshold |
|--------|--------|-------------------|
| Conversion rate | 35% | > industry median |
| Bounce rate | 25% | < industry median |
| Avg time on page | 20% | > 90 seconds |
| Core Web Vitals | 20% | All green (LCP < 2.5s, CLS < 0.1, FID < 100ms) |

---

## 9. Skill Composition Rules

### Permitted

- Multiple skills can be called sequentially by a single agent
- Skills can be called in parallel when their inputs are independent
- Skill outputs can be passed as inputs to other skills
- Skills can be reused across multiple agents

### Forbidden

- Skills must never call other skills directly (only agents orchestrate skills)
- Skills must never mutate shared state
- Skills must never bypass validation (no `// @ts-ignore` or schema casting)
- Skills must never perform I/O (database reads/writes belong in the agent layer, not skill layer)
- Skills must never have conditional logging that swallows errors silently

### Composition Pattern (Sequential)

```typescript
// In an agent:
const intent = await skills.intent_inference.execute({ text: editorInput });
const resolved = await skills.constraint_resolution.execute({ constraints: intent });
const valid = await skills.schema_enforcement.execute({
  data: resolved,
  schema: IntentSchema,
  strict: true,
});
if (!valid.valid) throw new SkillCompositionError("intent_resolution", valid.errors);
```

### Composition Pattern (Parallel)

```typescript
// When inputs are independent:
const [schemaResult, a11yResult] = await Promise.all([
  skills.schema_enforcement.execute({ data: layout, schema: LayoutSchema, strict: true }),
  skills.a11y_heuristics.execute({ layout, registry }),
]);
```

---

## 10. Anti-Patterns

| Anti-Pattern | Why It Fails | Correct Approach |
|-------------|-------------|-----------------|
| **Multi-responsibility skill** | Untestable, unclear failure modes | Split into two separate skills |
| **Hidden side effects** (DB writes, API calls) | Non-deterministic, not independently testable | I/O belongs in agents, not skills |
| **Schema-less output** | Enables type errors to propagate silently | All outputs must be validated against Zod schemas |
| **Implicit state sharing** | Race conditions, unpredictable behavior | Skills receive all state as explicit input parameters |
| **Silent error swallowing** | Failure traces are lost; debugging is impossible | All errors are thrown or returned explicitly |
| **Skill calling skill** | Creates coupling, defeats single responsibility | Orchestration belongs to agents |
| **Free-form text output** | Cannot be reliably parsed by downstream consumers | All outputs are typed JSON |

---

## 11. Skill Registry

Summary of all skills for quick reference:

| Skill | Category | Determinism | Agent(s) Using It |
|-------|---------|------------|-----------------|
| `skill.intent_inference` | Cognitive | semi-deterministic | Intent Normalizer |
| `skill.constraint_resolution` | Cognitive | deterministic | Intent Normalizer |
| `skill.pattern_retrieval` | Structural | semi-deterministic | Layout Strategist |
| `skill.structure_synthesis` | Structural | semi-deterministic | Layout Strategist |
| `skill.component_resolution` | Component | semi-deterministic | Component Composer |
| `skill.schema_enforcement` | Component / Quality | deterministic | Composer, Quality Gate |
| `skill.token_constraint_solver` | Component | deterministic | Composer, Quality Gate |
| `skill.a11y_heuristics` | Quality | semi-deterministic | Quality Gate |
| `skill.consistency_check` | Quality | deterministic | Layout Strategist, Quality Gate |
| `skill.minimal_diff_engine` | Refinement | semi-deterministic | Refinement Engine |
| `skill.pattern_extraction` | Learning | semi-deterministic | Pattern Learner |
| `skill.performance_feedback` | Learning | deterministic | Pattern Learner |

---

## 12. Implementation Guide

### Scaffold (TypeScript + Zod)

```typescript
// packages/skills/base.ts
import { ZodSchema } from "zod";

export abstract class Skill<TInput, TOutput> {
  abstract name: string;
  abstract description: string;
  abstract determinism: "deterministic" | "semi-deterministic";
  abstract input_schema: ZodSchema<TInput>;
  abstract output_schema: ZodSchema<TOutput>;

  async execute(raw_input: unknown): Promise<TOutput> {
    // Validate input
    const input = this.input_schema.parse(raw_input);

    // Run skill logic
    const raw_output = await this.run(input);

    // Validate output
    const output = this.output_schema.parse(raw_output);

    return output;
  }

  protected abstract run(input: TInput): Promise<TOutput>;
}
```

### Example Implementation

```typescript
// packages/skills/schema_enforcement.ts
import { z } from "zod";
import { Skill } from "./base";

const InputSchema = z.object({
  data: z.unknown(),
  schema: z.any(),   // ZodSchema passed at runtime
  strict: z.boolean().default(true),
});

const OutputSchema = z.object({
  valid: z.boolean(),
  errors: z.array(z.object({
    path: z.string(),
    expected: z.string(),
    received: z.string(),
    message: z.string(),
  })),
});

export class SchemaEnforcementSkill extends Skill<
  z.infer<typeof InputSchema>,
  z.infer<typeof OutputSchema>
> {
  name = "skill.schema_enforcement";
  description = "Validate data against a Zod schema; return validity verdict and error list";
  determinism = "deterministic" as const;
  input_schema = InputSchema;
  output_schema = OutputSchema;

  protected async run(input: z.infer<typeof InputSchema>) {
    const result = input.schema.safeParse(input.data);

    if (result.success) {
      return { valid: true, errors: [] };
    }

    return {
      valid: false,
      errors: result.error.issues.map((issue) => ({
        path: issue.path.join("."),
        expected: issue.code,
        received: String((input.data as Record<string, unknown>)?.[issue.path[0]] ?? "undefined"),
        message: issue.message,
      })),
    };
  }
}
```

### Testing Pattern

```typescript
// packages/skills/__tests__/schema_enforcement.test.ts
import { SchemaEnforcementSkill } from "../schema_enforcement";
import { LayoutSchema } from "../../schemas/layout";

describe("skill.schema_enforcement", () => {
  const skill = new SchemaEnforcementSkill();

  it("returns valid: true for conforming data", async () => {
    const result = await skill.execute({
      data: validLayoutFixture,
      schema: LayoutSchema,
      strict: true,
    });
    expect(result.valid).toBe(true);
    expect(result.errors).toHaveLength(0);
  });

  it("returns all errors for invalid data", async () => {
    const result = await skill.execute({
      data: { layout: "not_an_array" },
      schema: LayoutSchema,
      strict: true,
    });
    expect(result.valid).toBe(false);
    expect(result.errors.length).toBeGreaterThan(0);
  });
});
```

### Adding a New Skill — Checklist

Before merging a new skill:

- [ ] Follows the `Skill<TInput, TOutput>` base class contract
- [ ] Input and output schemas are defined with Zod
- [ ] Determinism class is declared and accurate
- [ ] `side_effects: "none"` — no database writes, no API calls, no global state mutation
- [ ] Failure modes are documented in the skill registry entry
- [ ] Unit tests cover: happy path, each documented failure mode, schema validation rejection
- [ ] Skill is registered in `packages/skills/index.ts`
- [ ] Skill is added to the Skill Registry table in this document

---

## Final Principle

> **Skills define capability. Agents define behavior. The system defines truth.**

The component registry and token system are the authoritative boundaries of what is possible. Skills are the mechanisms by which agents reason about, validate, and operate within those boundaries. The quality of the system depends on the precision of its skills — not the power of its LLMs.

---

*Skills Specification · 34-EnterpriseAICMS · v1.0 · 2025+*
