# Power Apps Code App — Project Rules

## Project Overview
This is a **Power Apps Code App** built with **Vite + React + TypeScript**.  
It uses the `@microsoft/power-apps` SDK and the `@microsoft/power-apps-vite` Vite plugin.

---

## CLI Tool
- The CLI is **`pac`** (Power Platform CLI).
- You **must** be authenticated (`pac auth create`) before running any `pac` commands.
- Always select the target environment with `pac env select --environment <id>` after authenticating.

---

## Code Structure
- Application source code lives in `src/`.
- **Generated data-source files** are placed in `src/generated/` by `pac code add-data-source`.
  - **Never edit generated files manually.** If the schema changes, delete and re-add the data source.
- Models:   `src/generated/models/`
- Services: `src/generated/services/`

---

## Development
- Local dev server: `npm run dev`
- Open the **"Local Playback"** URL in the **same browser profile** as your Power Platform tenant.
- Since Dec 2025, Chrome/Edge may block local network access — grant browser permissions if needed.

---

## Build & Deploy
```bash
npm run build                                          # build only
npm run build | pac code push                          # build + deploy
pac code push --solutionName <SolutionName>            # target specific solution
```

---

## Data Connections

### General workflow
1. Create the connection at [make.powerapps.com](https://make.powerapps.com) → Connections → + New
2. Run `pac connection list` to get the **Connection ID**
3. Add via `pac code add-data-source`
4. Import the generated service into your component

### SharePoint Lists
**Do not** use `pac code list-data-source` or try to discover lists via CLI —
it will not find them. When the user provides a SharePoint site URL and list name,
run the command directly:

```bash
pac code add-data-source \
  -a "shared_sharepointonline" \
  -c <connectionId> \
  -d <sharepoint-site-url> \
  -t <list-name>
```

Example:
```bash
pac code add-data-source \
  -a "shared_sharepointonline" \
  -c "aaaabbbb-0000-1111-2222-ccccddddeeee" \
  -d "https://contoso.sharepoint.com/sites/mysite" \
  -t "ProjectRequests"
```

After adding, import and use the generated service:
```typescript
import { ProjectRequestsService } from './generated/services/ProjectRequestsService';

const items = await ProjectRequestsService.getall();
```

---

## Key References
- [Create an app from scratch](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/create-an-app-from-scratch)
- [Connect to data](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/connect-to-data)
- [ALM for code apps](https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/alm)
- [PAC CLI reference](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/code)
