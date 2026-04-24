Good—this is exactly where most concepts fall apart.
Your original plan assumes a “greenfield-ish” rollout, but real enterprises are deeply entangled (legacy Drupal builds, custom modules, inconsistent content models, etc.).

Below is an extension of both documents with a realistic migration path that avoids big-bang failure.

⸻

📄 implementationplan.md (Extended with Migration)

# Implementation Plan — Agentic Intent-to-Experience Platform (with Migration)
---
## 0. Migration Philosophy
- No big-bang rewrite
- Parallel systems, gradual adoption
- Measure progress via **component coverage**, not site count
- New system proves value before replacing old
---
## 1. Migration Strategy Overview
### Approach: Strangler Fig Pattern
```text id="mig-flow-1"
Legacy Drupal → Hybrid → Component-driven → Fully Agentic

You incrementally replace:

* components first
* then layouts
* then authoring model

⸻

2. Migration Phases (Overlaying Core Phases)

⸻

Phase 0 — Discovery & Mapping (Months 0–2)

Goals

* Understand current landscape
* Define migration scope

Activities

* Audit all Drupal sites:
    * components
    * content types
    * themes
* Identify:
    * top 20 reusable components
    * high-traffic templates
* Map content models across sites

Deliverables

* Component inventory
* Content model map
* Migration priority matrix

⸻

Phase 1 — System Foundation + Coexistence (Months 1–6)

Key Shift

New system exists alongside legacy

Migration Work

1. Component Extraction

* Convert legacy Twig components → SDC
* Normalize props + tokens
* Remove inline styles

2. Tokenization

* Extract CSS → design tokens
* Map legacy styles → semantic tokens

3. Dual Rendering Support

* Legacy pages still render
* New components available in parallel

Legacy page → (existing)
New page → SDC components

4. Content Compatibility Layer

* Build adapters:
    * legacy fields → new component props

⸻

Deliverables

* SDC library (v1)
* Token system
* Registry v1
* Compatibility layer

⸻

Phase 2 — Progressive Page Migration (Months 6–14)

Key Shift

New pages use new system. Old pages remain.

⸻

Migration Work

1. New Content Creation → New System Only

* All new pages must use:
    * SDC components
    * token system

2. Pattern-Based Migration

* Convert top templates into reusable patterns:
    * landing page
    * product page
    * campaign page

3. Assisted Migration Tools

* Build scripts:
    * detect legacy layouts
    * suggest equivalent component patterns

4. Hybrid Rendering

Page = legacy blocks + new components

⸻

AI Introduction (Controlled)

* AI used for:
    * content generation
    * layout suggestions (optional)
* No auto-publish yet

⸻

Deliverables

* Pattern library
* Migration tooling
* Hybrid rendering support

⸻

Phase 3 — Agentic Authoring Activation (Months 12–20)

Key Shift

New authoring model becomes primary

⸻

Migration Work

1. Enable Agent Pipeline

* Intent → Plan → Compose → Validate

2. Convert Patterns → AI Templates

* Existing patterns become:
    * few-shot examples
    * reusable AI building blocks

3. Editor Migration

* Train teams:
    * from manual layout → intent-driven

4. Gradual Decommissioning

* Disable legacy layout builder for new content
* Lock old templates

⸻

Deliverables

* Agentic UI
* AI composition engine
* validated pattern library

⸻

Phase 4 — Legacy Decommissioning (Months 18–30)

Key Shift

Legacy becomes read-only → removed

⸻

Migration Work

1. Remaining Page Migration

* Auto-convert where possible
* Manual only for edge cases

2. Kill Switches

* Remove:
    * old themes
    * deprecated components
    * unused content types

3. Archive Strategy

* snapshot legacy pages
* keep fallback rendering if needed

⸻

Deliverables

* 90%+ component coverage
* legacy system retired

⸻

3. Migration Metrics

Track progress via:

* % pages using new components
* % content types migrated
* component reuse ratio
* legacy code reduction

⸻

4. Migration Risks

Risk	Mitigation
legacy complexity	start with top 20 components
editor resistance	gradual onboarding
inconsistent content	mapping + adapters
hybrid complexity	limit hybrid duration

⸻

5. Critical Success Rule

Migration succeeds when new content never uses legacy patterns again.

---
# 📄 architecture.md (Extended with Migration Layer)
```markdown
# Architecture — Agentic Intent-to-Experience Platform (with Migration)
---
## 1. Migration-Aware Architecture
You are not replacing the system.
You are **wrapping and evolving it**.
---
## 2. Transitional Architecture
```text
┌──────────────────────────────┐
│ Experience Layer             │
│ (New UI + Legacy UI)         │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ Agent Layer (New)            │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ Composition Engine (New)     │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ Component Layer              │
│ (SDC + Legacy Components)    │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ Compatibility Layer          │
│ (Adapters, Mappers)          │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│ Drupal Legacy + New System   │
└──────────────────────────────┘

⸻

3. Compatibility Layer (Critical Addition)

Purpose

Bridge old and new systems during migration

⸻

Responsibilities

1. Component Adapters

{
  "legacy_component": "hero_old",
  "maps_to": "hero_sdc",
  "field_mapping": {
    "headline": "title",
    "image": "media"
  }
}

⸻

2. Content Transformers

* normalize field structures
* resolve inconsistencies

⸻

3. Layout Interpreters

* detect legacy layouts
* convert to component configs

⸻

4. Hybrid Rendering Model

During migration:

Page =
  Legacy Blocks
  + SDC Components

Requirements

* shared styling layer (tokens)
* no visual drift
* unified CSS variables

⸻

5. Dual Authoring System

Legacy Mode

* existing layout builder
* restricted to old pages

New Mode

* agentic + component-based

Rule

* no mixing authoring paradigms per page

⸻

6. Data Evolution

Before

* inconsistent content types
* duplicated fields

After

* normalized schemas
* component-aligned data

⸻

7. Incremental Replacement Strategy

Replace in this order:

1. styles → tokens
2. components → SDC
3. layouts → patterns
4. authoring → AI

⸻

8. Deployment During Migration

* legacy and new deployed together
* feature flags control rollout
* per-site activation possible

⸻

9. Observability During Migration

Track:

* legacy vs new usage
* component adoption
* migration errors
* rendering differences

⸻

10. Decommission Architecture

Final state removes:

- compatibility layer
- legacy components
- old layout system

⸻

11. Key Architectural Rule

The new system must work independently before the old system is removed.

⸻

12. Migration Anti-Patterns (Avoid)

* big-bang rewrite
* forced full-site migration
* mixing legacy + new logic in same component
* skipping token system

⸻

13. End-State Architecture (Clean)

Experience UI
   ↓
Agent Layer
   ↓
Composition Engine
   ↓
Component System
   ↓
Token System
   ↓
Validation Layer
   ↓
Drupal + Infra

⸻

14. Guiding Principle

Migrate behavior, not just code.
Replace decisions, not just components.

---
# 🧠 What this upgrade does
This migration layer is the difference between:
### ❌ “great concept that fails”
and
### ✅ “platform that actually gets adopted”
---
# 🚀 If you want next
I can go even deeper into:
- **[migration tooling design](chatgpt://followup-prompt?start_index=8041&end_index=8065)** (auto-conversion scripts, AST parsing)
- **[real Drupal module architecture for this](chatgpt://followup-prompt?start_index=8111&end_index=8151)**
- or a **[pilot rollout plan for a single enterprise site](chatgpt://followup-prompt?start_index=8163&end_index=8210)**
That’s where this becomes *real-world executable*, not just well-designed.
