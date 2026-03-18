# errors.md — Known Gotchas & Common Errors

Read this before writing any SDK, connector, or data-layer code.

---

## 1. Direct `fetch` / `axios` to Connectors

**Symptom:** CORS errors, 401s, or missing auth headers.

```typescript
// ❌ Never do this
const res = await fetch('https://myapi.example.com/data');

// ✅ Use only generated services
import { MyService } from './generated/services/MyService';
const res = await MyService.getall();
```

The Power Apps host injects auth tokens into connector calls automatically. Direct HTTP calls bypass this entirely.

---

## 2. Editing Generated Files

**Symptom:** Broken types after `pac code push`; your changes are silently overwritten.

- `src/generated/` is **owned by PAC CLI** — treat it as read-only.
- If the connector schema changes: run `pac code delete-data-source`, then `pac code add-data-source`.
- Never add custom logic inside generated services.

---

## 3. Local Dev Blocked by Browser (Chrome/Edge post-Dec 2025)

**Symptom:** `npm run dev` runs, but the Local Playback URL shows a network error in the browser.

- Chrome and Edge now block local network access by default.
- Fix: Open `chrome://flags` → search "local network access" → set to **Enabled**.
- Or: grant the permission when the browser prompts you.
- Always open the Local Playback URL in the **same browser profile** as your Power Platform tenant.

---

## 4. `localStorage` / `sessionStorage` Not Available

**Symptom:** `localStorage is not defined` or silent failures inside Code Apps iframe.

```typescript
// ❌ Not supported — iframe sandbox blocks storage APIs
localStorage.setItem('key', 'value');

// ✅ Use React state for session data
const [value, setValue] = useState('');

// ✅ Use Dataverse / SharePoint for persistent data
await MyDataverseService.create({ key: 'value' });
```

---

## 5. SharePoint Choice Fields — Wrong Format

**Symptom:** Item creates/updates silently fail or save `null`.

```typescript
// ❌ Wrong — plain string
await ListService.create({ Status: "Active" });

// ✅ Correct — Value object
await ListService.create({ Status: { Value: "Active" } });

// ✅ Multi-choice
await ListService.update(id, { Tags: [{ Value: "A" }, { Value: "B" }] });
```

See `sharepoint.md` for full details on People Picker and dynamic choice loading.

---

## 6. People Picker — UPN ≠ Email

**Symptom:** Assignment fails, person field saves as null, or wrong user assigned.

- UPN and email can differ for guest accounts and some tenants.
- Never construct the Claims string from email alone.
- Always resolve via `Office365UsersService.MyProfile_V2()`.

```typescript
// ❌ Wrong — email may not equal UPN
const claims = `i:0#.f|membership|${email}`;

// ✅ Correct — always use UPN from Office365Users profile
const profile = await Office365UsersService.MyProfile_V2('userPrincipalName');
const claims = `i:0#.f|membership|${profile.data.userPrincipalName}`;
```

---

## 7. `pac code add-data-source` for SharePoint — Don't Use `list-data-source`

**Symptom:** `pac code list-data-source` returns nothing or errors for SharePoint.

- SharePoint lists are NOT discoverable via CLI.
- Provide the site URL and list name directly in the command:

```bash
pac code add-data-source \
  -a "shared_sharepointonline" \
  -c <connectionId> \
  -d <sharepoint-site-url> \
  -t <list-name>
```

---

## 8. Root CSS Missing — App Doesn't Fill Iframe

**Symptom:** App renders in a small box with scroll bars outside the content area.

```css
/* src/index.css — required */
html, body, #root {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
```

Scrolling only happens inside `<main style={{ flex: 1, overflow: "auto" }}>`.

---

## 9. Auth — Never Implement MSAL

**Symptom:** MSAL login loop, duplicate sign-in prompts.

- The Power Apps host handles all authentication.
- Never import or call `@azure/msal-browser`.
- User identity (name, UPN, tenant, session) is available via `getContext()` from `@microsoft/power-apps/app` — call it only when you need that data, it is not a required initialization step.
