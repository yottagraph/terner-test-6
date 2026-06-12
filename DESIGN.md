# terner test 6 - Design Document

## Project Overview

This is a freshly-scaffolded Aether tenant app. No product vision has been
captured yet, so the UI currently shows the default Aether starter — a
welcome page that points users at the `/build_my_app` workflow.

To turn this into a real app:

1. Replace this section with what you want to build (audience, problem,
   key user journeys). A short paragraph is enough to get started.
2. Open the project in Cursor or Claude Code and run `/build_my_app`.
   The agent will read the vision below, design the UX around the
   Lovelace platform's data, and replace the placeholder pages.

**Created:** 2026-06-12
**App ID:** terner-test-6
**Description:** Aether app: terner test 6
**Last updated:** 2026-06-12

## Vision

_(Not yet defined.)_

When you fill this in, the next `/build_my_app` run uses it as the brief.
Anything from a single sentence ("a watchlist for biotech IPOs") to a
full PRD works — see `.agents/skills/aether/design.md` for guidance on
long-form briefs.

## Configuration

| Setting        | Value                                         |
| -------------- | --------------------------------------------- |
| Authentication | Auth0                                         |
| Hosting        | GKE (BC 2.0)                                  |
| Agent hosting  | GKE (BC 2.0)                                  |
| Query Server   | https://query.pip.prod.g.lovelace.ai          |
| Live URL       | https://ui.terner-test-6.tenant.g.lovelace.ai |

Tenant identity (org_id, gateway URL, service account) lives in
`broadchurch.yaml` — that file is the source of truth, do not duplicate
the values here.

## Stack

- **Framework:** Nuxt 3 (SPA mode) + Vue 3 Composition API + Vuetify 3
- **Language:** TypeScript
- **Auth:** Auth0 (bypassed in local dev via `NUXT_PUBLIC_USER_NAME`)
- **Data:** Lovelace Query Server / Elemental API via the Broadchurch
  Portal Gateway. See `.agents/skills/aether/data.md`.
- **Storage:** per-tenant Firestore (BC 2.0) via `useAppPrefs` /
  `useGlobalPrefs`. See `.agents/skills/aether/pref.md`.
- **Agents:** ADK (`agents/`) deployed to the per-tenant GKE cluster on
  push to `main`.
- **MCP servers:** FastMCP (`mcp-servers/`) deployed via `/deploy_mcp`.

## Pages

These are the pages that ship with the starter scaffolding. They will be
replaced or removed as soon as a real vision is defined.

### Home

- **Route:** `/`
- **File:** `pages/index.vue`
- **Status:** Placeholder welcome screen with "Getting Started" steps.
- **Replace when:** the project has a real vision and `/build_my_app`
  designs the actual home page.

### Login / Auth callback / Logout / Pending

- **Routes:** `/login`, `/a0callback`, `/logout`, `/pending`
- **Files:** `pages/login.vue`, `pages/a0callback.vue`, `pages/logout.vue`,
  `pages/pending.vue`
- **Status:** Standard Auth0 flow. Keep as-is unless auth requirements
  change.

### Chat

- **Route:** `/chat`
- **File:** `pages/chat.vue`
- **Status:** Generic agent chat UI built on the `useAgentChat`
  composable. Useful as a quick demo of any deployed ADK agents; can be
  removed if the app doesn't expose chat.

### Prefs Demo

- **Route:** `/prefs-demo`
- **File:** `pages/prefs-demo.vue`
- **Status:** Demo of the `useAppPrefs` / `useGlobalPrefs` composables.
  Remove once the real app uses prefs in context.

### Tenancy Probe

- **Route:** `/tenancy-probe`
- **File:** `pages/tenancy-probe.vue`
- **Status:** Diagnostic page that reports which data planes (QS, KV,
  Firestore, Postgres, BigQuery, agent) are reachable for this tenant.
  Useful for debugging; safe to keep.

## Cross-Cutting Concepts

### Composables

Standard Aether composables are in place under `composables/`:

- `useAppInfo` — app name + ID from runtime config
- `useAppPrefs` / `useGlobalPrefs` / `useAppFeaturePrefs` /
  `useGlobalFeaturePrefs` — Firestore-backed preferences
- `useElementalSchema` — cached schema discovery for the Query Server
- `useAgentChat` — streaming chat with deployed ADK agents
- `useNotification` — in-app toast notifications
- `usePlatformStatus` / `useServerStatus` — platform health surfacing
- `useSession` / `useUserState` — Auth0 session state
- `useLovelaceTheme` / `useThemeClasses` — Lovelace branding tokens

### Server Routes

Pre-wired Nitro routes in `server/api/`:

- `me`, `a0callback`, `logout` — Auth0 session
- `prefs/{read,write,delete,status}` — preferences backend
- `kv/{read,write,delete,status,documents}` — legacy KV (BC 1.0)
- `platform/status`, `qs-status` — health probes
- `agent/[agentId]/stream` — agent streaming proxy
- `agents` — list deployed agents
- `tenancy-probe` — multi-plane diagnostic
- `avatar/[url]` — proxied image avatar

## Deployment

`hosting: gcp` and `agent.hosting: gke` in `broadchurch.yaml` mean every
push to `main` triggers:

- `.github/workflows/deploy-ui.yml` → Cloud Build → ArgoCD rolls the UI
  image into the per-tenant GKE cluster (live at
  `https://ui.terner-test-6.tenant.g.lovelace.ai`).
- `.github/workflows/deploy-agent.yml` → same path for any agents under
  `agents/**`.

Do not create pull requests — push directly to `main`. See
`.agents/skills/aether/git-support.md`.
