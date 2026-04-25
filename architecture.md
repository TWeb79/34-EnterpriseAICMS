# Architecture — Enterprise AI CMS
## Agentic Intent-to-Experience Platform

> Drupal-based enterprise WebCM with AI-augmented, token-driven, component-first design system. Migration-aware from day one.

---

## Table of Contents

1. [Core Vision & Structural Guarantees](#1-core-vision--structural-guarantees)
2. [Migration-Aware Philosophy](#2-migration-aware-philosophy)
3. [System Architecture Diagram](#3-system-architecture-diagram)
4. [The Five Layers](#4-the-five-layers)
5. [Compatibility Layer](#5-compatibility-layer)
6. [Hybrid Rendering Model](#6-hybrid-rendering-model)
7. [AI Composition System](#7-ai-composition-system)
8. [Token-Bounded Style Variation](#8-token-bounded-style-variation)
9. [Quality Gates](#9-quality-gates)
10. [Federated Governance Model](#10-federated-governance-model)
11. [Component Evolution Loop](#11-component-evolution-loop)
12. [Technology Stack](#12-technology-stack)
13. [End-State Architecture](#13-end-state-architecture)
14. [Architectural Rules & Anti-Patterns](#14-architectural-rules--anti-patterns)

---

## 1. Core Vision & Structural Guarantees

**Single sentence:** An AI-augmented, token-driven, component-first Drupal platform where any authorized editor can compose beautiful, WCAG AA–compliant, brand-coherent websites — and where the system structurally prevents the creation of anything insecure, inaccessible, or off-brand.

### Three Structural Guarantees

| Guarantee | What It Means | How It Is Enforced |
|-----------|--------------|-------------------|
| **Quality by Construction** | Components are pre-built, pre-tested, pre-validated. Editors assemble — they never build from scratch. | Component registry; schema-bound props; CI/CD gates block deployment on failure |
| **AI as Constrained Composer** | AI generates, selects, and arranges from the design system's vocabulary only. It cannot escape it. | LLM context is the component registry; output is validated against schemas before editor sees it |
| **Governance Without Bureaucracy** | Rules are code, not committees. Approvals trigger only for exceptions outside the validated envelope. | Policy-as-code in CI/CD; automated WCAG, security, and brand-token compliance |

> **Core Architectural Principle:** The design system is not a style-guide PDF. It is a living, versioned, machine-readable contract between designers, developers, content authors, and AI — enforced at build time, render time, and publish time.

---

## 2. Migration-Aware Philosophy

**You are not replacing the system. You are wrapping and evolving it.**

### Guiding Principle

> Migrate behavior, not just code. Replace decisions, not just components.

### Core Rules

- **No big-bang rewrite.** New system coexists alongside legacy. Value is proven before legacy is decommissioned.
- **Parallel systems, gradual adoption.** New authoring becomes available; editors switch when they are ready.
- **Progress measured by component coverage, not site count.** A single page using new components is meaningful progress.
- **New system must prove value before replacing old.** AI-assisted pages that ship faster with higher quality drive organic adoption.

### Strangler Fig Pattern

```
Legacy Drupal → Hybrid Coexistence → Component-Driven → Fully Agentic
```

Replacement order (lowest risk first):

1. **Styles → Tokens** — Extract all CSS to design tokens. Immediate consistency benefit, no editor impact.
2. **Components → SDC** — Rebuild top-used components as Drupal Single Directory Components.
3. **Layouts → Patterns** — Convert page templates to reusable, AI-composable patterns.
4. **Authoring → AI** — Introduce conversational layout composition. Depends on all prior layers.

---

## 3. System Architecture Diagram

### Transitional State (During Migration)

```
┌──────────────────────────────────────────────────────────────────┐
│  EXPERIENCE LAYER                                                │
│  Conversational AI UI (new) ·  Legacy Layout Builder (read-only) │
└────────────────────────────────┬─────────────────────────────────┘
                                 ↓
┌──────────────────────────────────────────────────────────────────┐
│  AGENT LAYER (new)                                               │
│  Intent Normalizer · Layout Strategist · Component Composer ·    │
│  Quality Gate · Refinement Engine · Pattern Learner              │
└────────────────────────────────┬─────────────────────────────────┘
                                 ↓
┌──────────────────────────────────────────────────────────────────┐
│  COMPOSITION ENGINE (new)                                        │
│  Layout Builder / Experience Builder config generation from AI   │
└────────────────────────────────┬─────────────────────────────────┘
                                 ↓
┌──────────────────────────────────────────────────────────────────┐
│  COMPONENT LAYER                                                 │
│  SDC Library (new) · Legacy Components (migrating out)           │
└────────────────────────────────┬─────────────────────────────────┘
                                 ↓
┌──────────────────────────────────────────────────────────────────┐
│  COMPATIBILITY LAYER (temporary — removed at decommission)       │
│  Component Adapters · Content Transformers · Layout Interpreters │
└────────────────────────────────┬─────────────────────────────────┘
                                 ↓
┌──────────────────────────────────────────────────────────────────┐
│  DRUPAL CORE + INFRASTRUCTURE                                    │
│  Structured content · Permissions · Multisite · JSON:API         │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. The Five Layers

### Layer 1 — Foundation

The infrastructure layer. Stable, GitOps-managed, and treated as an internal product.

| Component | Technology | Responsibility |
|-----------|-----------|---------------|
| CMS Core | Drupal 11 (headless/hybrid) | Structured content, permissions, multisite |
| Fleet Management | Kubernetes + ArgoCD/Flux + Terraform | GitOps-driven site provisioning, per-tenant isolation |
| Security Baseline | CSP headers, RBAC, WAF, HashiCorp Vault | Zero-trust posture, secrets management, audit trails |
| CDN & Edge | Varnish + Fastly/Cloudflare | Global delivery, edge caching, DDoS mitigation |
| CI/CD Pipeline | GitHub Actions + Chromatic + OWASP ZAP | Deployment quality gates; blocks on a11y, perf, security failure |
| Observability | Datadog / Grafana | Live health monitoring across the entire fleet |

**Key rule:** Security requirements are baked into the CI/CD pipeline from day one. No component or page reaches production without passing all gates.

---

### Layer 2 — Design Tokens

The stylistic contract of the entire platform. All CSS derives from this layer.

```
Figma Tokens Studio
      ↓
Style Dictionary (compilation)
      ↓
JSON + CSS custom properties + Drupal theme variables
      ↓
All sites consume via their theme
```

#### Token Hierarchy

| Tier | Example | Purpose |
|------|---------|---------|
| **Global Tokens** | `color-blue-500: #1a4fff` | Raw values; brand primitives |
| **Semantic Tokens** | `color-action-primary: {color-blue-500}` | Purpose-bound; consumed by components |
| **Component Tokens** | `button-background: {color-action-primary}` | Per-component overrideable values |
| **Brand Overlay Tokens** | `color-action-primary: #e8440a` (brand B) | Multi-brand token sets with validated palettes |

**Key rule:** No component ships with raw CSS values. All styling references semantic tokens. This is enforced in CI.

---

### Layer 3 — Component & Pattern

The composable vocabulary of the platform. All creative expression happens here.

| Component | Description |
|-----------|------------|
| **Drupal SDC Library** | 50–200+ audited components. Each ships with Storybook stories, axe-core a11y tests, visual regression snapshots, and a JSON schema descriptor. |
| **Machine-Readable Registry** | JSON schema per component: name, description, slots, allowed fields, style variants, a11y metadata, token bindings. Consumed by the AI layer as system context. |
| **Page Pattern Store** | Serialized Layout Builder / Experience Builder configs, tagged by intent. AI-generated layouts that editors accept are saved here as reusable global templates. |
| **Variant Engine** | Token-bounded theme overrides per brand/region. AI respects the requesting site's variant. |

**Component Registry Schema (per component):**

```json
{
  "id": "hero_banner",
  "name": "Hero Banner",
  "description": "Full-width hero section with headline, subtext, optional media, and CTA",
  "category": "layout.hero",
  "slots": ["headline", "subtext", "media", "cta"],
  "required_props": ["headline"],
  "optional_props": ["subtext", "media", "cta"],
  "token_bindings": {
    "background": ["surface-primary", "surface-dark", "surface-brand"],
    "spacing": ["spacing-section-sm", "spacing-section-md", "spacing-section-lg"]
  },
  "a11y_metadata": {
    "landmark": "banner",
    "heading_level": "h1",
    "requires_alt": ["media"]
  },
  "performance_budget": {
    "max_lcp_ms": 2500
  },
  "version": "2.1.0",
  "status": "stable"
}
```

---

### Layer 4 — AI Orchestration

The intelligence layer. Constrained by the component registry and token system above it; visible only as an assistant to editors.

| Module | Function |
|--------|---------|
| **Prompt-to-Layout Engine** | LLM (Claude / GPT-4o) receives the full component registry as system context. Maps natural-language editor intent to valid component + token configurations. Cannot output anything not in the registry. |
| **Guardrailed Token Variator** | Maps style-mood instructions ("make it more editorial") to numeric token adjustments within predefined safe intervals. Every adjustment is contrast-checked before surfacing. |
| **Quality Pre-Flight Pipeline** | Runs before the editor ever sees AI output: axe-core → Pa11y → contrast ratio → CSP compatibility → Core Web Vitals estimation → visual regression baseline check. Only 100%-passing layouts are shown. |
| **Component Gap Detector** | Logs every intent the AI could not satisfy with existing components. Feeds the quarterly component backlog. Closes the creativity gap systematically. |

**AI System Prompt Pattern (Composition Engine):**

```
SYSTEM:
You are a deterministic UI composition engine operating under strict constraints.
You may ONLY use components from the provided registry.
You may ONLY adjust tokens within the provided safe intervals.
You NEVER output raw HTML, CSS, or unapproved markup.
You ALWAYS output valid JSON matching the Layout Schema.

COMPONENT REGISTRY: [full JSON schema of all components]
TOKEN CONSTRAINTS: [safe interval table for this brand]
BRAND CONTEXT: [requesting site's token overlay]

USER: [editor's natural-language intent]
```

---

### Layer 5 — Experience

The editor-facing layer. Everything above is invisible infrastructure; this is what the user touches.

| Feature | Description |
|---------|------------|
| **Conversational UI** | Chat panel inside Drupal admin. Editor describes intent; AI returns 3–5 layout options with a one-click preview and refine loop. |
| **Content Studio** | Decoupled authoring with inline SEO analysis, AI-generated alt text, readability scoring, inclusive-language checks, and structured data markup. |
| **Self-Service Dashboard** | Site provisioning portal, brand token override controls, component usage analytics, and governance cockpit. |

---

## 5. Compatibility Layer

This layer exists **only during migration** and is removed when legacy is fully decommissioned. Its purpose is to bridge old and new without requiring simultaneous full-site rewrites.

### 5.1 Component Adapters

Map legacy component field names to SDC props:

```json
{
  "legacy_component": "hero_old",
  "maps_to": "hero_banner",
  "field_mapping": {
    "headline": "headline",
    "body_text": "subtext",
    "background_image": "media.image",
    "cta_text": "cta.label",
    "cta_link": "cta.url"
  },
  "unsupported_fields": ["custom_css_class"],
  "migration_note": "custom_css_class is unsupported; token-based variants replace it"
}
```

### 5.2 Content Transformers

- Normalize inconsistent field structures across diverged content types
- Resolve duplicated or conflicting field definitions
- Map legacy Paragraph structures to new component slot configurations
- Flag unmappable fields for manual migration review

### 5.3 Layout Interpreters

- Detect legacy layout configurations (Panels, old Layout Builder sections, Paragraphs)
- Convert to equivalent Experience Builder / new Layout Builder configurations
- Flag unmappable layouts for manual migration
- Generate migration confidence scores per page

### 5.4 Hybrid Rendering Contract

During migration, a page may contain both legacy blocks and new SDC components:

```
Request
  ↓
Drupal Router
  ↓
Page Controller
  ↓
Hybrid Renderer
  ├── Legacy Blocks → Legacy Theme (token-mapped CSS)
  └── SDC Components → New Theme (same CSS custom properties)
  ↓
Unified HTML Response
```

**Requirements:**
- Both systems consume the same CSS custom properties from the token layer
- Visual regression tests confirm no visual drift between legacy and new rendering
- No mixing of authoring paradigms per page — a page is authored in legacy mode or new mode, never both

---

## 6. Hybrid Rendering Model

### Dual Authoring System

| Mode | Availability | Scope |
|------|-------------|-------|
| **Legacy Mode** | Existing pages only | Layout Builder (old), Paragraphs, custom layouts |
| **New Mode** | All new pages from Phase 2 | Conversational UI + SDC component composition |

**Governance rule:** From Phase 2 onward, no new page may be created in legacy mode. Legacy mode is available only for editing existing legacy pages.

### Shared CSS Layer

Both legacy and new components receive the same design token CSS custom properties. This ensures visual consistency during the transition period and means design token changes propagate to both systems simultaneously.

---

## 7. AI Composition System

### End-to-End Interaction Model

```
Editor:  "I need a product launch page for our eco-friendly sneaker.
          Hero with looping video, three testimonial cards, a
          sustainability stats grid, and a newsletter sign-up.
          Vibrant, earthy, slightly edgy."

System:  [Runs intent normalization → layout planning → composition →
          pre-flight validation]

AI:      "Generated 3 layout options using only approved components
          and the brand's Evergreen token set.
          All options pass WCAG AA, mobile responsive, security checks.

          Option A: Full-bleed hero, horizontal testimonials, grid stats
          Option B: Split hero, stacked testimonials, inline stats
          Option C: Editorial hero, testimonials carousel, dashboard stats

          Preview & refine →"

Editor:  "I like Option B. Swap testimonials to the left and make the
          hero darker."

AI:      "Updated. Hero background shifted to token value surface-dark.
          Testimonials column moved to left. All quality gates passing.
          Ready to preview."
```

### What the AI Cannot Do

The AI is explicitly prevented from:
- Outputting raw HTML, CSS, or JavaScript
- Referencing components not in the approved registry
- Introducing third-party scripts or external dependencies
- Adjusting tokens outside predefined safe intervals
- Bypassing the pre-flight validation pipeline
- Publishing without explicit human approval

---

## 8. Token-Bounded Style Variation

Style instructions from editors are translated into precise, bounded numeric adjustments:

| Editor Instruction | Token Adjustment | Safe Range |
|--------------------|-----------------|-----------|
| "Make it more energetic" | Primary hue +10°, spacing +0.1× | Hue ±15°, spacing ±0.25× |
| "Make it more luxurious" | Primary hue toward gold +8°, letter-spacing +5% | Hue ±20°, letter-spacing ±10% |
| "Increase density" | Section spacing −0.2×, type scale −0.1× | Spacing ≥ 0.6× base, type ≥ 0.9× base |
| "Make it bolder" | Font weight +100, heading size +0.15rem | Weight ≤ 900, size ≤ display-xl |
| "More editorial" | Spacing +0.3×, column ratio to 2/3, serif display font | Spacing ≤ 2.5× base |
| "Lighter and airier" | Background toward surface-light, spacing +0.2× | Passes WCAG AA contrast |

All adjustments are automatically contrast-checked and accessibility-validated before surfacing to the editor. An adjustment that would violate contrast requirements is automatically rejected and an alternative offered.

---

## 9. Quality Gates

Quality is enforced architecturally — at the pipeline level — not through manual review. Every AI-generated layout and every page publish passes through all gates. Failure blocks deployment without platform-team override.

| Gate | Tool | Failure Condition |
|------|------|-----------------|
| **WCAG AA** | axe-core + Pa11y | Any WCAG AA violation |
| **Contrast Ratio** | Custom checker | Any text/background pair below 4.5:1 (3:1 for large text) |
| **Core Web Vitals** | Lighthouse CI | LCP > 2.5s, CLS > 0.1, FID > 100ms at P75 |
| **Security Headers** | Custom scan | Missing CSP, HSTS, X-Frame-Options headers |
| **Third-Party Scripts** | Allowlist check | Any unapproved external dependency |
| **OWASP Top 10** | OWASP ZAP | Any critical or high severity finding |
| **Visual Regression** | Chromatic / BackstopJS | Pixel diff > threshold vs. brand baseline |
| **Token Compliance** | Style Dictionary lint | Any raw CSS value outside the token system |
| **Schema Validity** | JSON Schema / Zod | Any component configuration not matching registry schema |

---

## 10. Federated Governance Model

### Operating Structure

```
Central DesignOps (10–15 people globally)
├── Owns: token foundation, component registry, CI/CD, security baseline, AI layer
├── Operates as: internal product team with public roadmap
└── Approves: component RFCs and token architecture changes

Regional / Brand Platform Teams (3–5 per brand)
├── Owns: brand token overlays, regional component variants, regional pipelines
├── Provides: first-line support for site teams
└── Contributes: new components to central registry via RFC

Site Teams (1–3 per site)
├── Owns: content, layout decisions within bounds, stakeholder relationships
├── Self-serves: platform capabilities via conversational UI and dashboards
└── Escalates: to regional team for out-of-envelope needs
```

### Automated Escalation Rules

| Trigger | Escalation | Recipient |
|---------|-----------|----------|
| WCAG compliance < 90% for any site | Automated alert | Regional platform team |
| Critical security advisory unpatched > 7 days | Automated escalation | Central platform team |
| Core Web Vitals failing at P75 > 14 days | Automated notification with fix suggestions | Site team |
| Component registry version drift > 2 minor versions | Upgrade reminder | Site team |

### Governance Cockpit KPIs

| Metric | Target | Enforcement |
|--------|--------|------------|
| Time to publish new campaign page | < 2 hours | Tracked |
| WCAG AA compliance | 100% | Publish gate |
| Component reuse ratio across sites | > 85% | Tracked |
| AI composition adoption (% new pages) | > 70% | Tracked |
| Security patch deployment latency | < 24 hours | Automated remediation |
| New site provisioning time | < 30 minutes | GitOps automation |

---

## 11. Component Evolution Loop

The platform is designed to continuously close the gap between editor needs and available capabilities:

```
Editor intent cannot be satisfied by existing components
                    ↓
AI logs component gap (intent + context + frequency)
                    ↓
Quarterly backlog review by DesignOps
                    ↓
DesignOps designs, tokens, builds, tests new component
(a11y tests · visual regression · security review · schema definition)
                    ↓
Component published to registry
                    ↓
Immediately available to AI composition engine and all sites
                    ↓
AI few-shot examples updated with new component usage patterns
```

This loop ensures the design system improves in direct response to real editor needs rather than theoretical requirements.

---

## 12. Technology Stack

### CMS & Content Platform

| Technology | Role |
|-----------|------|
| Drupal 11 | Core CMS: structured content, multisite, permissions, workflow |
| Drupal SDC | Single Directory Components — native component architecture |
| Drupal Experience Builder | Next-generation layout composition (Phase 2+) |
| JSON:API / GraphQL | Headless content delivery |
| Acquia / Pantheon / Platform.sh | Managed Drupal hosting |

### Design System & Tokens

| Technology | Role |
|-----------|------|
| Figma + Tokens Studio | Design token source of truth |
| Style Dictionary | Token compilation → CSS / JSON / Drupal theme variables |
| Storybook 8 | Component documentation, interactive testing |
| Chromatic | Visual regression testing in CI |
| axe-core + Pa11y | Automated accessibility testing |
| BackstopJS | Cross-browser visual regression |

### AI & Intelligence Layer

| Technology | Role |
|-----------|------|
| Claude API (claude-sonnet-4-20250514) | Primary LLM for layout composition |
| GPT-4o | Fallback / comparative LLM |
| Drupal AI Module | Native AI integration points in Drupal |
| Custom MCP Server | Exposes component registry as AI tool-use vocabulary |
| OpenAI CLIP | Image analysis for alt-text generation |
| Lighthouse CI | Performance quality gates |
| OWASP ZAP | Automated security scanning in CI |

### DevOps & Infrastructure

| Technology | Role |
|-----------|------|
| Kubernetes | Container orchestration for Drupal fleet |
| ArgoCD / Flux | GitOps-based fleet management |
| Terraform | Infrastructure-as-code per site tenant |
| GitHub Actions | CI/CD pipeline and quality gate automation |
| Fastly / Cloudflare | CDN, edge caching, WAF |
| Datadog / Grafana | Observability and alerting |

### Security & Compliance

| Technology | Role |
|-----------|------|
| HashiCorp Vault | Secrets management across the fleet |
| Snyk | Dependency vulnerability scanning |
| ModSecurity / AWS WAF | Web application firewall |
| Content Security Policy headers | XSS surface reduction |
| Dependabot | Automated dependency update PRs |

### Governance & Analytics

| Technology | Role |
|-----------|------|
| Custom Governance Cockpit | React dashboard on Drupal APIs |
| PostHog / Amplitude | Editor behavior analytics |
| Matomo | Privacy-first visitor analytics |
| Jira / Linear | RFC and component backlog management |

---

## 13. End-State Architecture

When migration is complete and the compatibility layer is removed, the architecture is clean:

```
┌─────────────────────────────────────────┐
│  EXPERIENCE UI (Conversational)         │
│  Prompt panel · Preview · Cockpit       │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  AGENT LAYER                            │
│  Intent · Plan · Compose · Validate ·   │
│  Refine · Learn                         │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  COMPONENT SYSTEM (SDC only)            │
│  Registry · Patterns · Variants         │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  TOKEN SYSTEM                           │
│  Global · Semantic · Brand overlays     │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  VALIDATION LAYER                       │
│  a11y · Security · Performance · Brand  │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  DRUPAL + INFRASTRUCTURE                │
│  Content · GitOps · CDN · Security      │
└─────────────────────────────────────────┘
```

---

## 14. Architectural Rules & Anti-Patterns

### Rules (Non-Negotiable)

1. **The new system must work independently before the old system is removed.**
2. **Components that fail security review never enter the registry.** Retrofitting is 10× more expensive than requiring it upfront.
3. **AI proposes; humans approve.** No autonomous publishing without review.
4. **Every component ships with a11y tests, visual regression snapshots, and token compliance verification.**
5. **The design token system is the single source of stylistic truth.** All CSS derives from it. No raw values anywhere.
6. **The compatibility layer is temporary.** It must have a defined decommission date from the day it is created.

### Anti-Patterns (Forbidden)

| Anti-Pattern | Why It Fails |
|-------------|-------------|
| Big-bang rewrite | Too much risk, too long before value, breaks existing sites |
| Forced full-site migration | Editor resistance, resource bottlenecks, delivery risk |
| Mixing legacy + new logic in the same component | Unmaintainable, defeats the token system, creates visual drift |
| Skipping the token system | Design drift persists, AI cannot operate reliably on a consistent vocabulary |
| Building AI before the component registry is complete | AI outputs unvalidated, insecure, inaccessible, off-brand results |
| Treating the design system as a document | It must be code — versioned, tested, deployed, and breaking-change managed |
| Adding complexity before the MVP pipeline is stable | If the system cannot generate one good page reliably, more features make it worse |

---

*Architecture Document · 34-EnterpriseAICMS · v1.0 · 2025+*
