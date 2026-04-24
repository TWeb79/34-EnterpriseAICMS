# Getting Started — Agentic Intent-to-Experience Platform
---
## 0. Objective
Stand up a **working vertical slice** of the system:
User prompt → AI agents → validated layout JSON → rendered preview
NOT:
- full enterprise rollout
- full Drupal integration
- perfect architecture
YES:
- end-to-end pipeline working
- real components + tokens
- real validation
---
## 1. Development Environment Setup
### 1.1 Install Core Tools
```bash
# Node (LTS)
nvm install --lts
nvm use --lts
# Package manager
npm install -g pnpm
# Git
brew install git   # macOS
# or
sudo apt install git  # Linux
# Docker (for DB + vector)
brew install --cask docker
# Optional but recommended
brew install jq

⸻

1.2 IDE Setup

* VS Code
* Extensions:
    * ESLint
    * Prettier
    * JSON Schema Validator
    * GitLens

⸻

2. Repository Setup

2.1 Create Repo

git init agentic-webcm
cd agentic-webcm
pnpm init

⸻

2.2 Folder Structure (IMPORTANT)

/agentic-webcm
├── apps/
│   ├── api/                # orchestrator
│   ├── web/                # preview UI
│
├── packages/
│   ├── schemas/            # Zod schemas
│   ├── agents/             # agent logic
│   ├── skills/             # skill implementations
│   ├── registry/           # component registry
│   ├── tokens/             # design tokens
│
├── data/
│   ├── patterns/
│   ├── fixtures/
│
├── docs/
│   ├── agentic.md
│   ├── skills.md
│
└── infra/
    ├── docker-compose.yml

⸻

3. Core Dependencies

pnpm add zod openai dotenv uuid
pnpm add -D typescript ts-node @types/node

Optional (later):

pnpm add pg pgvector

⸻

4. Step 1 — Define Executable Schemas

📍 packages/schemas/layout.ts

import { z } from "zod";
export const LayoutSchema = z.object({
  layout: z.array(
    z.object({
      component: z.string(),
      props: z.record(z.any()),
      tokens: z.record(z.any())
    })
  )
});
export type Layout = z.infer<typeof LayoutSchema>;

⸻

5. Step 2 — Create Component Registry (CRITICAL)

📍 packages/registry/components.json

[
  {
    "name": "hero_banner",
    "props": {
      "title": "string",
      "subtitle": "string",
      "image": "string"
    },
    "tokens": {
      "background": ["primary", "neutral"],
      "spacing": ["md", "lg"]
    }
  },
  {
    "name": "testimonial_cards",
    "props": {
      "items": "array"
    },
    "tokens": {
      "columns": ["2", "3"]
    }
  }
]

👉 Start with 3–5 components only

⸻

6. Step 3 — Define Token System

📍 packages/tokens/tokens.json

{
  "color": {
    "primary": "#1a4fff",
    "neutral": "#f5f3ee"
  },
  "spacing": {
    "md": "16px",
    "lg": "32px"
  }
}

⸻

7. Step 4 — Implement First Skill

📍 packages/skills/schemaValidation.ts

import { LayoutSchema } from "@schemas/layout";
export function validateLayout(data: unknown) {
  return LayoutSchema.safeParse(data);
}

⸻

8. Step 5 — Build Minimal Agent (Composition Only)

📍 packages/agents/composer.ts

import OpenAI from "openai";
import registry from "@registry/components.json";
const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});
export async function composeLayout(plan: any) {
  const response = await client.chat.completions.create({
    model: "gpt-4o",
    response_format: { type: "json_object" },
    messages: [
      {
        role: "system",
        content: `
You are a strict UI composition engine.
Rules:
- Use ONLY provided components
- Output valid JSON
- Do not explain
Components:
${JSON.stringify(registry)}
`
      },
      {
        role: "user",
        content: JSON.stringify(plan)
      }
    ]
  });
  return JSON.parse(response.choices[0].message.content!);
}

⸻

9. Step 6 — Orchestrator (VERY SIMPLE FIRST)

📍 apps/api/orchestrator.ts

import { composeLayout } from "@agents/composer";
import { validateLayout } from "@skills/schemaValidation";
export async function run(prompt: string) {
  const plan = {
    sections: [
      { type: "hero" },
      { type: "testimonials" }
    ]
  };
  const layout = await composeLayout(plan);
  const validation = validateLayout(layout);
  if (!validation.success) {
    throw new Error("Invalid layout");
  }
  return layout;
}

⸻

10. Step 7 — Basic API

📍 apps/api/server.ts

import express from "express";
import { run } from "./orchestrator";
const app = express();
app.use(express.json());
app.post("/generate", async (req, res) => {
  const result = await run(req.body.prompt);
  res.json(result);
});
app.listen(3000);

⸻

11. Step 8 — Simple Preview UI

📍 apps/web

* React app
* input field
* call /generate
* render JSON

You do NOT need full rendering yet.

⸻

12. Step 9 — Git Workflow

git checkout -b feat/initial-pipeline
git add .
git commit -m "feat: initial agent pipeline"
git push origin feat/initial-pipeline

⸻

13. Step 10 — Docker (Optional Early)

📍 infra/docker-compose.yml

version: "3"
services:
  db:
    image: postgres
    ports:
      - "5432:5432"

⸻

14. First Milestone (1–2 Days)

You should be able to:

* enter prompt
* generate layout JSON
* validate it
* display result

⸻

15. What NOT to build yet

* full AI pipeline
* learning agent
* multi-brand tokens
* Kubernetes
* Drupal integration

⸻

16. Next Steps (After MVP Works)

1. Add Intent Agent
2. Add Planning Agent
3. Add Validation Agent
4. Add Pattern DB
5. Improve registry

⸻

17. Debugging Strategy

* log every agent output
* store layouts
* diff changes
* track failures

⸻

18. Key Rule

If the system cannot generate ONE good page reliably,
adding more features will make it worse.

⸻

19. Success Criteria

* 3 working components
* valid layouts every time
* no crashes
* predictable outputs

⸻

20. Final Thought

You are not building:

* a CMS
* or an AI toy

You are building:
→ a deterministic system that happens to use AI

---
# 🚀 What this gives you
This is now:
### ✅ Buildable in 1–3 days  
### ✅ Minimal but correct  
### ✅ Scales into your full architecture  
---
# If you want next step
I can give you:
- [**actual working repo scaffold (copy-paste project)**](chatgpt://followup-prompt?start_index=6586&end_index=6639)
- [**first 5 real components (enterprise-grade)**](chatgpt://followup-prompt?start_index=6642&end_index=6688)
- [**prompt evaluation test suite**](chatgpt://followup-prompt?start_index=6691&end_index=6723)
That’s where this becomes something you can *demo to stakeholders*.
