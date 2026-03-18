---
name: power-apps-code-apps
description: >
  Expert skill for building Power Apps Code Apps — web applications hosted inside Microsoft Power Apps using React 19, TypeScript, Vite, and the @microsoft/power-apps SDK.
  Use this skill whenever the user mentions Power Apps Code Apps, pac CLI, pac code push, @microsoft/power-apps SDK, Power Platform connectors in a React/Vite context,
  SharePoint connector in a Code App, Dataverse in a Code App, or ALM for Code Apps.
  Also trigger for any task involving the Power Apps Vite plugin, generated services in src/generated/, pac auth, pac connection, or the phrase "Code App" in a Power Platform context.
  Also trigger for MicrosoftCopilotStudioService, ExecuteCopilotAsyncV2, shared_microsoftcopilotstudio, AI agent/chat in a Power Apps Code App, or x-ms-conversation-id header issues.
  Always use this skill when the user wants to scaffold, build, connect data to, deploy, or debug a Power Apps Code App — even if they only describe the task without naming the skill.
---

# Power Apps Code Apps — Skill

## What this skill covers
Universal rules for every Power Apps Code App project.
Not project-specific — applies to all Code App work.

## Dependencies — read in this order
| File | When to load |
|------|-------------|
| `setup.md` | Project init, folder structure, deploy commands |
| `rules.md` | CLI rules, data connection workflow, code conventions |
| `errors.md` | ⚠️ **Always read before writing any data or SDK code** |
| `design.md` | UI/design tokens, Tailwind usage, iframe constraints |
| `sharepoint.md` | Any task involving SharePoint lists or People Picker |
| `copilot-studio.md` | Any task involving MicrosoftCopilotStudioService, ExecuteCopilotAsyncV2, or AI chat in the app |

Load only the sub-files relevant to the current task.
**Always load `errors.md` before writing any SDK or connector code.**

---

## Stack
```
Framework:  React 19 + TypeScript
Build:      Vite 7 + @microsoft/power-apps-vite plugin
Platform:   Power Apps Code Apps (Preview)
Auth:       Microsoft Entra — handled by Power Apps Host
Deploy:     pac code push
Dev:        npm run dev
```

---

## Non-Negotiable Rules
```
1. Never use fetch/axios directly to any connector
2. Only use generated services in /src/generated/ — never edit them
3. If schema changes: delete + re-add data source, never edit generated files
4. Auth is handled by the host — never implement MSAL yourself
5. No localStorage / sessionStorage — use React state or Dataverse
6. Always read errors.md before touching the data layer
7. All data fetching in hooks/ — never directly inside components
8. Never manually recreate project structure — use pac code init for blank projects
9. If more than one CLI command fails in sequence: STOP and ask the user instead of attempting autonomous workarounds
```

---

## CLI Quick Reference
```bash
git clone <template-repo> <app-name>            # init from org template
npm install
pac auth create --environment <env-url>          # authenticate
pac env select --environment <id>                # select target env
pac connection list                              # get connection IDs
pac code add-data-source -a <api> -c <connId>   # add connector
pac code delete-data-source -a <api> -ds <n>    # remove / refresh schema
pac code push                                    # deploy
pac code push --solutionName <n>                 # deploy to solution
npm run dev                                      # local dev
npm run build                                    # production build
```

---

## Activation Prompts

### Claude
```
Read the Power Apps skill: SKILL.md and relevant sub-files
(setup.md / rules.md / design.md / errors.md / sharepoint.md).
Always read errors.md before writing any data layer or SDK code.
Apply all rules before writing any code.
```
