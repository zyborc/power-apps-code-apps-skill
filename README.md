# Power Apps Code Apps — Claude Skill

A Claude Code skill for building **Power Apps Code Apps** — web applications hosted inside Microsoft Power Apps, built with React 19, TypeScript, Vite, and the `@microsoft/power-apps` SDK.

> Power Apps Code Apps is currently in **Preview**. This skill encodes hard-won patterns, known gotchas, and CLI workflows that are not well documented elsewhere.

---

## What this skill covers

- Project scaffolding via `pac code init` or org template
- PAC CLI authentication and environment setup
- Adding and consuming data sources (SharePoint, Dataverse, Copilot Studio)
- SharePoint-specific patterns: choice fields, People Picker, dynamic option loading
- Microsoft Copilot Studio integration and known header bug workaround
- Build and deploy workflow (`pac code push`)
- Local dev setup including Chrome/Edge permission fix (post-Dec 2025)
- UI/design conventions (dark-navy Claude Code aesthetic, DM Sans + DM Mono, Tailwind + inline tokens)
- Common errors and how to avoid them

---

## Stack

| Layer | Technology |
|---|---|
| Framework | React 19 + TypeScript |
| Build | Vite 7 + `@microsoft/power-apps-vite` plugin |
| Platform | Power Apps Code Apps (Preview) |
| Auth | Microsoft Entra — handled by Power Apps Host |
| Deploy | `pac code push` |
| Dev | `npm run dev` |

---

## Prerequisites

- [Power Platform CLI (`pac`)](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction) installed
- A Power Platform environment with **Code Apps feature enabled** (Admin Center → Settings → Features)
- End users require a **Power Apps Premium license**
- Node.js 18+

---

## Install in Claude Code

Copy the skill folder into your project's `.agent/skills/` directory:

```bash
# From your project root
mkdir -p .agent/skills
git clone https://github.com/zyborc/power-apps-code-apps-skill.git .agent/skills/power-apps-code-apps
```

Claude Code will automatically detect and load the skill when you work on Power Apps Code App tasks.

### Manual activation

If Claude doesn't trigger the skill automatically, prompt it with:

```
Read the Power Apps skill: SKILL.md and relevant sub-files
(setup.md / rules.md / design.md / errors.md / sharepoint.md).
Always read errors.md before writing any data layer or SDK code.
Apply all rules before writing any code.
```

---

## Skill structure

| File | Purpose |
|---|---|
| `SKILL.md` | Entry point — stack, non-negotiable rules, CLI reference, load order |
| `setup.md` | Project init (template vs scratch), auth, canonical `package.json`, folder structure |
| `rules.md` | CLI rules, data connection workflow, code conventions |
| `errors.md` | Known gotchas — **always read before writing data or SDK code** |
| `design.md` | Design tokens (dark-navy theme), Tailwind usage, iframe constraints |
| `sharepoint.md` | SharePoint choice fields, People Picker, dynamic option loading |
| `copilot-studio.md` | `MicrosoftCopilotStudioService`, `ExecuteCopilotAsyncV2`, known header bug |

---

## Non-negotiable rules (summary)

1. Never use `fetch`/`axios` directly to connectors — use generated services only
2. Never edit files in `src/generated/` — they are owned by PAC CLI
3. If schema changes: delete + re-add the data source
4. Auth is handled by the host — never implement MSAL yourself
5. No `localStorage` / `sessionStorage` — use React state or Dataverse
6. Always read `errors.md` before touching the data layer
7. All data fetching in `hooks/` — never directly inside components
8. If more than one CLI command fails in sequence: stop and ask the user

---

## Official Microsoft Docs

- [Create an app from scratch](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/create-an-app-from-scratch)
- [Connect to data](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/connect-to-data)
- [ALM for Code Apps](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/alm)
- [PAC CLI reference](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/code)

---

## Contributing

Found a new gotcha or pattern? PRs welcome. Keep examples generic (no internal tenant URLs, connection IDs, or org names).
