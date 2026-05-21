# Compass — AI Career Adviser

A lifelong AI career companion that helps users discover goals, validate them with reality checks, build a personalized roadmap, practice interviews, and match against jobs.

## Run & Operate

- App runs via Replit workflows (do not run `pnpm dev` at root)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from `lib/api-spec/openapi.yaml`
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL`, `AI_INTEGRATIONS_OPENAI_BASE_URL`, `AI_INTEGRATIONS_OPENAI_API_KEY` (auto-provisioned)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 at `/api`
- Frontend: React + Vite + wouter + TanStack Query at `/`
- DB: PostgreSQL + Drizzle ORM
- LLM: OpenAI `gpt-5.4` via Replit AI Integrations proxy (no key required)
- API codegen: Orval (React Query hooks + Zod schemas)

## Where things live

- API contract: `lib/api-spec/openapi.yaml` (source of truth, run codegen after changes)
- DB schema: `lib/db/src/schema/` (profile, skills, interview, goals, roadmap, mockInterviews)
- Backend routes: `artifacts/api-server/src/routes/`
- LLM helper: `artifacts/api-server/src/lib/llm.ts`
- Frontend: `artifacts/career-adviser/src/`
- Generated API client hooks: `lib/api-client-react/src/generated/`
- Generated Zod request/response schemas: `lib/api-zod/src/generated/`

## Architecture decisions

- **Single-user MVP**: no auth; backend uses a singleton profile (id=1) created on first request.
- **LLM-first design**: the core differentiator is depth of reasoning. CV parsing, deep interview, goal recommendations, reality checks, roadmap generation, mock interview evaluation, course ranking, and vacancy match scoring all use `gpt-5.4` with structured JSON output.
- **Stubbed paid integrations**: Course catalog (would be Skillbox/Stepik/Coursera) and vacancy aggregator (would be hh.ru/LinkedIn) are inlined mock data in route handlers — the LLM still ranks and rationalizes against them in a personalized way.
- **Entity-shaped OpenAPI body names** (`CvImportInput`, `GoalInput`, `MockAnswerInput`, etc.) for clean generated hook names.
- All LLM JSON calls go through `llmJson<T>()` which uses `response_format: json_object` and parses defensively.

## Product

Compass guides a user through:
1. **Onboarding** — pick spheres, paste CV (LLM parses), self-assess suggested skills, then run an adaptive deep-discovery interview (~8 questions) that extracts motivations and soft signals.
2. **Goals** — see 3-5 LLM-generated career recommendations spanning safe/stretch/bold, view per-goal reality checks (day-in-life, pain points, salary, personalized warnings), or enter goals manually, then lock one as primary.
3. **Roadmap** — generate 8-12 sequenced milestones (learning/project/credential/experience/portfolio/network), track progress.
4. **Learning** — courses grouped by skill gap with LLM-written "why this for you" rationale plus self-study pointers.
5. **Mock interviews** — behavioral/technical/system-design/mixed, adaptive Q&A with per-question scoring, model answers, and a final report.
6. **Vacancies** — mock job list with LLM match scoring, gap analysis, and a personalized cover letter draft.
7. **Dashboard** — command-center summary.

## User preferences

- No emojis in product UI.

## Gotchas

- After editing `lib/api-spec/openapi.yaml`, you must run `pnpm --filter @workspace/api-spec run codegen` for new hooks/schemas to appear.
- After editing `lib/db/src/schema/*` and exporting from `index.ts`, run `pnpm --filter @workspace/db run push`.
- LLM endpoints can take 5-15s; the frontend shows loading states. Don't add aggressive client timeouts.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
- See `.local/skills/ai-integrations-openai/SKILL.md` for OpenAI integration patterns
