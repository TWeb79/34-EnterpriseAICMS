# Implementation Plan — Enterprise AI CMS
## 30-Month Migration & Activation Roadmap

> Phased transition from legacy Drupal estate to AI-augmented, token-driven, component-first platform. No big-bang rewrites. Value delivered incrementally. Legacy retired only after the new system has proven itself in production.

---

## Table of Contents

1. [Migration Philosophy](#1-migration-philosophy)
2. [Phase 0 — Discovery & Mapping (Months 0–2)](#2-phase-0--discovery--mapping-months-02)
3. [Phase 1 — Foundation & Coexistence (Months 1–9)](#3-phase-1--foundation--coexistence-months-19)
4. [Phase 2 — AI Activation & Progressive Migration (Months 10–18)](#4-phase-2--ai-activation--progressive-migration-months-1018)
5. [Phase 3 — Global Scale & Autonomous Quality (Months 19–30)](#5-phase-3--global-scale--autonomous-quality-months-1930)
6. [First 90 Days — Concrete Actions](#6-first-90-days--concrete-actions)
7. [MVP Quick Start](#7-mvp-quick-start)
8. [Migration Metrics](#8-migration-metrics)
9. [Risk Register](#9-risk-register)
10. [Critical Success Rule](#10-critical-success-rule)

---

## 1. Migration Philosophy

### Core Principles

| Principle | What It Means in Practice |
|-----------|--------------------------|
| **No big-bang rewrite** | Legacy and new systems coexist in production throughout the transition |
| **Parallel systems, gradual adoption** | New authoring is available; editors switch when ready, not when forced |
| **Measure by component coverage** | Progress is `% pages using new components`, not `# sites migrated` |
| **Prove value before replacing** | New system ships measurably better pages before legacy is decommissioned |
| **Migration succeeds when new content never uses legacy patterns again** | This is the decisive threshold for beginning decommission |

### Strangler Fig Pattern

```
Legacy Drupal  →  Hybrid Coexistence  →  Component-Driven  →  Fully Agentic
   (today)          (Phase 1–2)            (Phase 2–3)          (Phase 3+)
```

Replace in this order:

1. **Styles → Tokens** (lowest risk, highest immediate value, no editor impact)
2. **Components → SDC** (builds the composable vocabulary for AI)
3. **Layouts → Patterns** (enables AI composition)
4. **Authoring → AI** (depends on all prior layers; introduced last)

### What Not to Do

- Do not attempt full-site migrations in a single sprint
- Do not mix legacy and new authoring paradigms on the same page
- Do not introduce AI before the component registry is sufficiently populated
- Do not relax quality gates for legacy content during migration
- Do not build more features until the core pipeline reliably generates one good page

---

## 2. Phase 0 — Discovery & Mapping (Months 0–2)

**Goal:** Understand the current landscape completely before writing a single line of new code. Define scope, identify quick wins, and build the migration priority matrix.

### Activities

#### Component Audit

Inventory all Drupal themes, custom modules, and page templates across every site:

- Catalog every component with: name, usage frequency across sites, visual variants, technical debt score (1–5), and SDC migration complexity estimate (S/M/L/XL)
- Identify duplicates and near-duplicates (components doing the same thing differently across sites)
- Identify dead code (components deployed but unused for > 6 months)
- Select the **top 20 most-reused, lowest-complexity components** as the Phase 1 extraction target

#### Content Model Mapping

- Inventory all content types and field definitions across every site
- Identify field inconsistencies (same concept, different field names), duplicates, and diverged definitions
- Define target normalized schemas aligned with SDC component props
- Map every legacy Paragraph type to its equivalent SDC component

#### Template Analysis

Identify the **top 10 highest-traffic page templates** across the estate. These become:

- The priority targets for Phase 2 pattern migration
- The test cases for validating the hybrid rendering model
- The few-shot examples for the AI composition engine

**Template types to identify:** Landing page, product page, campaign page, article/editorial, hub/index, error page, contact/form page.

#### Stakeholder Interviews

Structured interviews covering:

- **Site editors:** Daily workflow pain points, dependency on developers, accessibility incident history, desired creative capabilities
- **Brand/design teams:** Design drift incidents, review bottleneck frequency, brand enforcement challenges
- **Security team:** Current posture, incident history, compliance requirements, patch latency
- **Platform/DevOps:** Infrastructure heterogeneity, deployment pain points, snowflake sites

### Deliverables

| Deliverable | Contents |
|-------------|---------|
| **Component Inventory** | Full catalog with usage frequency, duplication score, debt rating, migration complexity |
| **Content Model Map** | All content types and fields with interrelationship diagram |
| **Migration Priority Matrix** | Components and templates ranked by impact × migration effort |
| **Stakeholder Report** | Pain points, requirements, and success metrics per role |
| **Phase 1 Scope Definition** | Confirmed target component set for SDC migration |

---

## 3. Phase 1 — Foundation & Coexistence (Months 1–9)

**Goal:** Build the new system alongside the old. Stop design drift. Establish the canonical token system, migrate the top-priority components to SDC, and deploy automated quality gates. Introduce AI only for low-risk content tasks (text, alt-text, SEO). Editors notice improved content tooling; developers notice improved component consistency. No layout AI yet.

### Key Shift

> New system exists alongside legacy. Both render production pages. Quality improves measurably before editors have to change anything.

### Activities

#### Design Token Architecture (Months 1–3)

- Establish the three-tier token hierarchy (global → semantic → component) in Figma Tokens / Style Dictionary
- Define naming conventions, brand sub-set structure, and the regional overlay model
- Automate Figma ↔ code sync via GitHub Actions
- **Map all legacy CSS values to semantic tokens.** This is the first bridge between old and new — legacy themes start consuming token variables even before SDC migration.
- Agree on versioning strategy (semantic versioning, changelogs, deprecation windows)

#### SDC Component Extraction — Top 20 (Months 2–7)

For each of the top 20 components:

1. Define the JSON schema (props, slots, token bindings, a11y metadata)
2. Build as Drupal Single Directory Component (SDC) with clean Twig template
3. Write Storybook stories covering all variants and states
4. Write axe-core accessibility tests (pass required for registry entry)
5. Create visual regression snapshots with Chromatic/BackstopJS
6. Security review (no unapproved third-party scripts, CSP compliant)
7. Publish to component registry v1

#### Component Registry v1

- Machine-readable JSON API of all SDC components
- Includes: name, description, slots, props, token bindings, a11y metadata, version, status
- Published as a Drupal module endpoint
- This registry is the foundation the AI layer will consume in Phase 2

#### Compatibility Layer

- Build component adapters for the top 20 legacy → SDC field mappings
- Build content transformers for the most common field inconsistencies
- Build layout interpreter prototypes for the top 5 legacy layout patterns
- Document all unmappable cases for manual migration handling

#### CI/CD Quality Gate Infrastructure

Deploy these gates into the Drupal deployment pipeline with blocking enforcement:

- **WCAG AA:** axe-core + Pa11y (zero violations = pass)
- **Core Web Vitals:** Lighthouse CI (LCP < 2.5s, CLS < 0.1, FID < 100ms at P75)
- **Security headers:** CSP, HSTS, X-Frame-Options present = pass
- **OWASP ZAP:** No critical or high severity findings = pass
- **Visual regression:** Chromatic comparison against baseline (configurable threshold)
- **Token compliance:** No raw CSS values outside the token system = pass

Note: Begin in **warning mode** (log failures, don't block) for the first 4 weeks to establish baselines before enforcing.

#### AI Content Copilot (Limited Scope)

Integrate an LLM into CKEditor for:

- Long-form content generation from editor prompts
- Translation assistance (not replacement for professional translation)
- SEO metadata generation (meta title, meta description, structured data)
- Alt-text generation for uploaded images (mandatory human review)
- Inclusive language checks (real-time suggestions, not enforcement)
- Readability scoring (Flesch-Kincaid surfaced in editor UI)

**Boundaries:** Human review is mandatory on all AI-generated content. No layout or component generation in Phase 1. AI cannot publish without editor approval.

#### Governance Cockpit v1

Central dashboard for the platform team, showing per site:

- Component adoption rate (% legacy vs. SDC)
- WCAG AA compliance score
- Core Web Vitals P75 (per region)
- Security advisory status and patch currency
- Token override inventory

#### Security Baseline

Across all Drupal instances:

- Deploy and enforce Content Security Policy headers
- Implement RBAC with full audit logging
- Configure automated Drupal core and module patching pipeline (Dependabot + manual review)
- Deploy WAF (ModSecurity / AWS WAF) in front of all public-facing sites
- Vault-backed secrets management (no credentials in code or environment variables)

### Deliverables

| Deliverable | Detail |
|-------------|--------|
| **Design Token System** | Versioned, published, consumed by all Drupal themes. Figma ↔ code sync automated. |
| **SDC Library v1** | 20+ SDC components with Storybook stories, a11y and visual regression tests |
| **Component Registry v1** | JSON API of component schemas, machine-readable, versioned |
| **Compatibility Layer** | Adapters, transformers, and layout interpreters for legacy ↔ new |
| **CI/CD Quality Gates** | WCAG AA, performance, security — all blocking on failure in production pipeline |
| **AI Content Copilot** | Text generation, SEO metadata, alt-text, readability in CKEditor |
| **Governance Dashboard v1** | Adoption, a11y, security visibility per site for platform team |
| **Security Baseline** | CSP, RBAC, audit logs, automated patching across all instances |
| **Migration Runbook** | Documented process for onboarding existing sites to the new component system |

---

## 4. Phase 2 — AI Activation & Progressive Migration (Months 10–18)

**Goal:** Editors describe pages and AI assembles them. All new pages use SDC components and the token system. Legacy authoring is locked for new content. The platform moves from a better developer tool to a genuinely new editorial capability.

### Key Shift

> All new pages use new components and tokens. Legacy authoring is read-only for existing content. AI composition becomes available and measurably faster.

### Activities

#### New Content Governance Rule

From Phase 2 launch:

- All new page creation must use SDC components and the token system
- Legacy Layout Builder / Paragraphs disabled for new content
- Existing legacy pages remain fully editable in legacy mode
- Migration of existing pages follows a pattern-based approach

#### AI Composition Engine

Build or integrate an LLM-powered page composition assistant inside the Drupal authoring UI:

1. **System context:** LLM receives full component registry (all schemas, descriptions, slots) as system context
2. **Composition:** LLM generates Layout Builder / Experience Builder configurations in JSON — no custom code from editors
3. **Pre-flight:** Every generated layout passes the full quality gate pipeline before being shown to the editor
4. **Refinement loop:** Editor can iterate via follow-up prompts ("swap hero style", "make it darker", "move testimonials")
5. **Approval:** Editor reviews final layout and approves publish. No AI-autonomous publishing.

**Technical integration:** Custom Drupal module exposes the composition UI as a sidebar panel in the layout editing interface. LLM calls are handled server-side via the Drupal AI module + custom MCP server exposing the component registry as tool-use vocabulary.

#### Token-Aware Style Variator

Extend the token system with an AI-navigable "style dimension" model:

- Map human style instructions to bounded token adjustment intervals (see `architecture.md §8`)
- Validate every token adjustment for contrast ratio and accessibility before surfacing
- Brand token overlays respected: AI uses the requesting site's active token set

#### Pattern-Based Migration

Convert the top 10 highest-traffic page templates (identified in Phase 0) into reusable page patterns:

- Serialize as Layout Builder / Experience Builder configuration JSON
- Tag with: intent, audience type, content type, conversion goal
- Add to the Page Pattern Store in the component registry
- These become the primary few-shot examples for the AI composition engine

#### Assisted Migration Tooling

Build tooling to accelerate legacy page migration:

- **Layout detection script:** Scans legacy pages and identifies their template type
- **Component suggestion engine:** Proposes the equivalent new-system component pattern for each legacy layout
- **Migration confidence score:** Rates how cleanly a legacy page can be auto-converted (high/medium/low/manual-only)
- **Semi-automated conversion:** For high-confidence pages, generates a new-system draft for editor review

#### Multi-Brand Tenant Model

- Implement per-brand token overlay system in CI/CD
- Each brand/region site team receives their validated token set (colors, typography, motion preferences)
- AI composition engine is brand-context-aware: proposes only components configured for the requesting site's brand
- Cross-brand token contamination is architecturally prevented

#### Content Intelligence (Full)

- SEO analysis: keyword density, meta content scoring, structured data validation
- Alt-text generation: CLIP-based image analysis with editor review
- Inclusive language: real-time suggestions with reference to brand tone-of-voice guide
- Readability: Flesch-Kincaid + brand style alignment scoring
- Translation prep: identify translatable vs. structural content for downstream localization workflows

#### Component Evolution Feedback Loop

- Instrument the editor UI to log every "I wanted to do X but couldn't" signal
- Tag signals by: intent type, component category, brand context, frequency
- Quarterly DesignOps review of top-voted gaps
- Feeds directly into the Phase 3 component backlog

#### Component Library Expansion

Expand SDC library from 20 → 100+ components:

- Priority: components needed to cover the top 10 page pattern types
- All new components validated through the full CI quality gate pipeline before registry entry
- Registry v2 includes AI-discoverable metadata for each component

### Deliverables

| Deliverable | Detail |
|-------------|--------|
| **AI Composition UI** | Natural-language layout builder in Drupal admin; generates validated layouts |
| **Style Variant Engine** | AI-driven mood-to-token mapping within predefined safe bounds |
| **Content Intelligence** | SEO, alt-text, readability, inclusive language — all in-editor, AI-powered |
| **Multi-Brand Tenancy** | Per-brand token overlays; AI is brand-context-aware; validated palette enforcement |
| **Component Registry v2** | 100+ components with AI-discoverable metadata |
| **Pattern Library** | 10+ page patterns as reusable AI building blocks |
| **Migration Tooling** | Layout detection, component suggestion, semi-automated conversion |
| **Evolution Feedback Loop** | Instrumented editor gaps → quarterly backlog → platform growth |

---

## 5. Phase 3 — Global Scale & Autonomous Quality (Months 19–30)

**Goal:** The platform is the default for all enterprise web properties. Business teams provision sites and compose pages via prompt. The platform monitors itself, remediates issues proactively, and evolves through a community RFC process.

### Key Shift

> Legacy system retired. Self-service provisioning. AI-driven quality management. Living design standard with community governance.

### Activities

#### Self-Service Site Provisioning

Any authorized team can spin up a new Drupal site instance via a GitOps workflow in < 30 minutes:

- Request submitted via self-service portal (brand, region, purpose, team)
- Automated GitOps workflow provisions Drupal tenant on Kubernetes
- Site arrives pre-configured: correct brand token set, SDC component library version, security baseline, quality gates, CI/CD pipeline
- No central platform team involvement required for provisioning
- Site team receives onboarding guide and access to governance cockpit

#### Proactive Quality Intelligence

Shift from reactive to proactive quality management:

- AI monitors all live sites continuously via scheduled Lighthouse CI and axe-core scans
- Regressions in accessibility, performance, or security automatically generate remediation PRs
- PRs are reviewed by site teams (not auto-merged); the system suggests the fix but humans approve
- Governance cockpit becomes a live health system, not a periodic audit tool

#### AI-Driven A/B Testing

Within the platform's design system bounds:

- AI proposes layout variant experiments based on analytics signals (bounce rate, conversion, engagement)
- Variant layouts are generated using existing approved components and token configurations
- A/B tests run automatically via the analytics integration
- Results surfaced to editor with recommended action; human approves or rejects adoption
- Accepted variants are promoted to the Page Pattern Store for future use

#### Legacy Decommissioning

Execute the final migration of remaining legacy pages:

- Auto-convert high-confidence legacy pages using migration tooling from Phase 2
- Manual migration sprint for edge cases (highly customized, low-traffic pages)
- **Archive strategy:** Static snapshot of legacy pages before removal; fallback rendering for any unmigrated content
- Remove compatibility layer (adapters, mappers, transformers)
- Remove old themes, deprecated components, and unused content types
- **Decommission gate:** 90%+ of all pages using new components before full legacy removal

#### Living Design Standard — RFC Process

Establish a formal process for platform evolution:

1. Any team globally can submit a component RFC (Request for Change) via the governance portal
2. RFC includes: problem statement, proposed component, usage examples, accessibility notes, token requirements
3. AI assists in evaluating proposals for: duplication with existing components, accessibility implications, token system consistency
4. DesignOps reviews top-voted proposals quarterly
5. Approved components are built, tested, and shipped to all sites automatically
6. Declined proposals receive feedback; teams can revise and resubmit

#### Federated Governance — Final State

| Level | Team | Owns | Self-Service Scope |
|-------|------|------|--------------------|
| **Central** | DesignOps (10–15) | Token foundation, component registry, CI/CD, security baseline | Platform roadmap, RFC approvals |
| **Regional** | Brand teams (3–5 each) | Brand token overlays, regional variants, first-line support | Token adjustments within global bounds, component RFC submission |
| **Site** | Site teams (1–3 each) | Content, layouts within bounds, stakeholder relationships | All new page creation, site configuration within tenant scope |

### Deliverables

| Deliverable | Detail |
|-------------|--------|
| **Self-Service Provisioning** | New Drupal site live in < 30 min. GitOps-driven. Full platform config included. |
| **Proactive Quality AI** | Continuous monitoring with auto-remediation PRs for a11y, performance, security |
| **AI A/B Testing** | Autonomous layout experiments within design bounds; data-driven UX evolution |
| **Legacy Retirement** | 90%+ component coverage; compatibility layer removed; old themes and components gone |
| **Living Design Standard** | RFC process; AI-assisted evaluation; quarterly review; global auto-deploy |
| **Federated Governance** | Central + regional + site-level autonomy; clear scope boundaries; escalation paths |
| **Component Registry v3** | 200+ components; community contributions; AI-maintained documentation |

---

## 6. First 90 Days — Concrete Actions

### Month 1

| Week | Action | Owner |
|------|--------|-------|
| 1–2 | Kick off component audit across all sites | Platform team |
| 1–2 | Begin content model mapping | Platform team + site teams |
| 3 | Set up Figma Tokens Studio + Style Dictionary integration | Design lead |
| 3–4 | Select top 20 components for SDC extraction | Platform + design leads |
| 4 | Set up GitHub Actions CI pipeline (skeleton) | DevOps |

### Month 2

| Week | Action | Owner |
|------|--------|-------|
| 5–6 | Complete component audit and content model map | Platform team |
| 5–6 | Publish migration priority matrix | Platform team |
| 6–7 | Begin first 5 SDC component extractions | Frontend developers |
| 7–8 | Deploy CI quality gate pipeline in warning mode (axe-core, Lighthouse CI) | DevOps |
| 8 | Token system v0.1 publishing to first Drupal theme | Design + frontend |

### Month 3

| Week | Action | Owner |
|------|--------|-------|
| 9–10 | First 5 SDC components in staging with Storybook stories | Frontend |
| 9–10 | Compatibility layer prototype for top 3 legacy components | Backend |
| 11 | Governance cockpit v0.1 with first site data visible | Platform team |
| 11–12 | AI content copilot integration in CKEditor (text + alt-text only) | Backend + AI lead |
| 12 | Quality gates switched to blocking mode for new content | DevOps |

### End-of-Month-3 Checkpoint

Before proceeding to next sprint:

- [ ] Token system publishing to at least 2 Drupal themes
- [ ] 5+ SDC components with Storybook stories and a11y tests
- [ ] CI quality gates running and blocking on failures
- [ ] Component inventory visible in governance dashboard
- [ ] AI content copilot active in at least 1 site's CKEditor
- [ ] Phase 1 full scope confirmed based on discovery findings

---

## 7. MVP Quick Start

### Objective

Stand up a working vertical slice: **user prompt → AI agents → validated layout JSON → rendered preview**. Not a full Drupal integration; not a perfect architecture. Just a working end-to-end pipeline.

### Development Environment

```bash
# Node.js LTS
nvm install --lts && nvm use --lts

# Package manager
npm install -g pnpm

# Repository setup
git clone https://github.com/TWeb79/34-EnterpriseAICMS.git
cd 34-EnterpriseAICMS
pnpm install

# Environment variables
cp .env.example .env
# Add ANTHROPIC_API_KEY or OPENAI_API_KEY

# Start local infrastructure
docker-compose -f infra/docker-compose.yml up -d

# Development servers
pnpm --filter api dev    # Orchestrator API on :3000
pnpm --filter web dev    # Preview UI on :3001
```

### Repository Structure

```
34-EnterpriseAICMS/
├── apps/
│   ├── api/                # Orchestrator (Express + TypeScript)
│   └── web/                # Preview UI (React + Vite)
├── packages/
│   ├── schemas/            # Zod schemas (Layout, Intent, Plan, Validation)
│   ├── agents/             # Agent implementations
│   ├── skills/             # Skill implementations
│   ├── registry/           # Component registry JSON
│   └── tokens/             # Design token definitions
├── data/
│   ├── patterns/           # Saved page patterns
│   └── fixtures/           # Test fixtures
└── infra/
    └── docker-compose.yml  # Local Postgres
```

### Step 1 — Define Schemas

```typescript
// packages/schemas/layout.ts
import { z } from "zod";

export const ComponentConfigSchema = z.object({
  component: z.string(),
  props: z.record(z.any()),
  tokens: z.record(z.any()).optional(),
});

export const LayoutSchema = z.object({
  id: z.string().uuid(),
  brand: z.string(),
  layout: z.array(ComponentConfigSchema),
  metadata: z.object({
    generated_at: z.string().datetime(),
    intent_summary: z.string(),
    validation_status: z.enum(["pending", "passed", "failed"]),
  }),
});

export type Layout = z.infer<typeof LayoutSchema>;
```

### Step 2 — Component Registry (Start Small)

```json
// packages/registry/components.json
[
  {
    "id": "hero_banner",
    "name": "Hero Banner",
    "description": "Full-width hero with headline, optional subtext, media, and CTA",
    "category": "layout.hero",
    "required_props": ["headline"],
    "optional_props": ["subtext", "media", "cta"],
    "token_bindings": {
      "background": ["surface-primary", "surface-dark", "surface-brand"]
    }
  },
  {
    "id": "testimonial_cards",
    "name": "Testimonial Cards",
    "description": "Grid of 2–4 customer testimonial cards with quote, name, and title",
    "category": "social-proof",
    "required_props": ["items"],
    "optional_props": ["columns", "heading"],
    "token_bindings": {
      "columns": ["2", "3", "4"]
    }
  },
  {
    "id": "newsletter_signup",
    "name": "Newsletter Sign-Up",
    "description": "Email capture form with headline and CTA",
    "category": "conversion",
    "required_props": ["headline", "cta_label"],
    "optional_props": ["subtext", "privacy_note"]
  }
]
```

### Step 3 — Minimal Orchestrator

```typescript
// apps/api/orchestrator.ts
import Anthropic from "@anthropic-ai/sdk";
import registry from "../../packages/registry/components.json";
import { LayoutSchema } from "../../packages/schemas/layout";
import { v4 as uuidv4 } from "uuid";

const client = new Anthropic();

export async function generateLayout(prompt: string, brand: string = "default") {
  const response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 2000,
    system: `You are a deterministic UI composition engine.
RULES:
- Use ONLY components from the provided registry (by id)
- Output ONLY valid JSON matching the Layout schema
- Do not explain or add prose
- Do not invent components not in the registry
- Fill all required_props; optional_props only when relevant

COMPONENT REGISTRY:
${JSON.stringify(registry, null, 2)}

OUTPUT SCHEMA:
{
  "layout": [
    { "component": "<component_id>", "props": { ... }, "tokens": { ... } }
  ]
}`,
    messages: [{ role: "user", content: prompt }],
  });

  const text = response.content
    .filter((b) => b.type === "text")
    .map((b) => b.text)
    .join("");

  const parsed = JSON.parse(text.replace(/```json|```/g, "").trim());

  const validated = LayoutSchema.safeParse({
    id: uuidv4(),
    brand,
    layout: parsed.layout,
    metadata: {
      generated_at: new Date().toISOString(),
      intent_summary: prompt.slice(0, 100),
      validation_status: "pending",
    },
  });

  if (!validated.success) {
    throw new Error(`Layout validation failed: ${JSON.stringify(validated.error.issues)}`);
  }

  return validated.data;
}
```

### Step 4 — API Server

```typescript
// apps/api/server.ts
import express from "express";
import { generateLayout } from "./orchestrator";

const app = express();
app.use(express.json());

app.post("/generate", async (req, res) => {
  try {
    const { prompt, brand } = req.body;
    if (!prompt) return res.status(400).json({ error: "prompt required" });
    const layout = await generateLayout(prompt, brand);
    res.json(layout);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: String(err) });
  }
});

app.listen(3000, () => console.log("API running on :3000"));
```

### Step 5 — Validate the Vertical Slice

```bash
curl -X POST http://localhost:3000/generate \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A product launch page for an eco-friendly sneaker. Hero with looping video, three testimonials, and a newsletter sign-up.",
    "brand": "evergreen"
  }'
```

### MVP Success Criteria

- [ ] 3+ working components in registry
- [ ] Valid layout JSON generated on every request
- [ ] Schema validation passes on all outputs
- [ ] Preview UI renders the JSON structure
- [ ] No crashes under 10 concurrent requests

### What NOT to Build in MVP

- Full Drupal integration (Phase 1)
- Multi-agent pipeline (Phase 2)
- Vector database for pattern retrieval (Phase 2)
- Multi-brand token system (Phase 2)
- Visual rendering of components (Phase 2)
- Kubernetes deployment (Phase 3)

---

## 8. Migration Metrics

Track these metrics continuously throughout the transition:

| Metric | Phase 0 Baseline | Phase 1 Target | Phase 2 Target | Phase 3 Target |
|--------|-----------------|----------------|----------------|----------------|
| % pages using new SDC components | 0% | > 10% | > 40% | > 90% |
| % content types migrated | 0% | > 20% | > 50% | > 80% |
| Component reuse ratio across sites | unknown | > 40% | > 65% | > 85% |
| Legacy code volume (relative) | 100% | 75% | 50% | < 15% |
| Time to publish new campaign page | Baseline measured | −25% | −60% | < 2 hours |
| WCAG AA compliance (new pages) | Baseline measured | 100% | 100% | 100% |
| WCAG AA compliance (all pages) | Baseline measured | +20% | +60% | 100% |
| Editor satisfaction NPS | Baseline survey | Baseline + 10 | Baseline + 25 | Baseline + 40 |
| AI composition adoption (% new pages) | 0% | 0% (Phase 2 feature) | > 40% | > 70% |
| Security patch latency | Baseline measured | < 72 hours | < 48 hours | < 24 hours |

---

## 9. Risk Register

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Legacy component complexity exceeds estimates | High | High | Start with top 20 simplest; build extraction patterns before tackling complex ones; reserve 40% buffer in Phase 1 schedule |
| Editor resistance to new authoring model | High | Medium | Gradual onboarding; keep legacy available for existing pages; demonstrate measurable time savings on new pages before mandating adoption |
| Inconsistent content models block normalization | Medium | High | Content transformers and adapters accept some manual field mapping; don't let model normalization block component migration |
| Hybrid rendering creates visual drift | Medium | Medium | Shared CSS token layer; automated visual regression in CI; unified custom properties across both systems |
| AI generates poor layouts (low quality / hallucinated components) | Medium | High | Registry-constrained prompting; pre-flight validation before editor sees output; few-shot examples; component gap logging |
| LLM API latency or availability | Low | Medium | Server-side caching of common layout patterns; graceful degradation to manual component composition; multi-provider fallback |
| Security gaps during hybrid coexistence | Low | High | Both systems pass through identical CI/CD quality gates; no relaxation for legacy content |
| Phase 1 schedule slips, delaying AI activation | Medium | Medium | Phased rollout; Phase 2 AI work can begin in parallel with late Phase 1 if registry has ≥ 30 components; value not blocked |
| Executive sponsorship weakens mid-program | Medium | High | Monthly governance cockpit reviews with business stakeholders; quantified ROI metrics visible from Phase 1 |
| Emerging AI regulation imposes new requirements | Low | Medium | Design for human-in-the-loop from day one; maintain complete audit trails; no fully-autonomous publishing |

---

## 10. Critical Success Rule

> **Migration succeeds when new content never uses legacy patterns again.**

Operationally: once this rule has held for **three consecutive months** across all sites, the compatibility layer decommission clock starts. Legacy system retirement proceeds only after the new system has proven it can fully replace the old — measurably, not aspirationally.

### The Platform Succeeds When

- A regional marketing team can launch a fully compliant campaign site in two days without a developer
- A frontend developer builds a component once and it ships to hundreds of sites simultaneously
- The security team receives automated remediation PRs before they discover vulnerabilities
- The brand team can update a design token and see it propagate globally in minutes
- An editor types what they want and gets a validated, brand-compliant, accessible page

---

*Implementation Plan · 34-EnterpriseAICMS · v1.0 · 2025+*
