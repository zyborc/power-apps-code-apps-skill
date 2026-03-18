# sharepoint.md — SharePoint Connector: Rules & Gotchas

Read this whenever working with SharePoint as a data source in a Code App.
These are non-obvious behaviors that differ from standard REST expectations.

---

## 1. Choice Fields

Single and multi-choice fields require a specific object format — plain strings will fail silently or throw.

### Single Choice
```typescript
// ✅ Correct
await ListItemService.create({
  Status: { Value: "Active" },
  Category: { Value: "Internal" },
});

// ❌ Wrong — will fail
await ListItemService.create({
  Status: "Active",
});
```

### Multi Choice
```typescript
// ✅ Correct — array of Value objects
await ListItemService.update(id, {
  Tags: [
    { Value: "Frontend" },
    { Value: "React" },
  ],
});

// ❌ Wrong
await ListItemService.update(id, {
  Tags: ["Frontend", "React"],
});
```

---

## 2. People Picker Fields

SharePoint Person fields cannot be set with a plain email or user ID.
You must build the full person object using the **Office 365 Users connector**.

### Required Object Shape
```typescript
interface SharePointPerson {
  "@odata.type": string;   // e.g. "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedUser"
  Claims:       string;    // see Claims format below
  DisplayName:  string;
  Email:        string;
  Picture:      string;
  Department:   string;
  JobTitle:     string;
}
```

### Claims Format
```typescript
// UPN is NOT always the same as email in Entra ID — always use UPN explicitly
const claims = `i:0#.f|membership|${upn}`;  // upn from Office365UsersService

// ❌ Do not use email as UPN — they can differ
const wrong = `i:0#.f|membership|${email}`;
```

### Full Pattern — Resolve user then write to SharePoint
```typescript
import { Office365UsersService } from './generated/services/Office365UsersService';
import { MyListService } from './generated/services/MyListService';

const resolveSharePointPerson = async (upn: string): Promise<SharePointPerson> => {
  const profile = await Office365UsersService.MyProfile_V2(
    'displayName,mail,jobTitle,department,userPrincipalName'
  );

  return {
    "@odata.type": "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedUser",
    Claims:      `i:0#.f|membership|${profile.data.userPrincipalName}`,
    DisplayName: profile.data.displayName ?? '',
    Email:       profile.data.mail ?? '',
    Picture:     '',
    Department:  profile.data.department ?? '',
    JobTitle:    profile.data.jobTitle ?? '',
  };
};

// Usage
const person = await resolveSharePointPerson(upn);
await MyListService.create({
  AssignedTo: person,
});
```

> Always resolve via `Office365UsersService` — never construct the Claims string
> from email alone. UPN and email can differ for guest accounts and some tenants.

---

## 3. Choice Field Options — Loading Valid Values

To populate dropdowns or validate form inputs, load the allowed choice values
via `getReferencedEntity` on the generated service — not hardcoded strings.

```typescript
import { MyListService } from './generated/services/MyListService';

// Load valid options for a choice column
const options = await MyListService.getReferencedEntity('Status');
// returns string[] of allowed values for that column

// Use in a dropdown
const [statusOptions, setStatusOptions] = useState<string[]>([]);

useEffect(() => {
  MyListService.getReferencedEntity('Status').then(setStatusOptions);
}, []);

// Render
<select>
  {statusOptions.map(opt => (
    <option key={opt} value={opt}>{opt}</option>
  ))}
</select>

// Write back with correct format
await MyListService.update(id, {
  Status: { Value: selectedOption },
});
```

**Why not hardcode values?**
Choice options can be changed in SharePoint without touching the app.
`getReferencedEntity` always reflects the current list configuration.

---

## 4. Quick Reference

| Scenario | Pattern |
|---|---|
| Single choice write | `{ Value: "OptionName" }` |
| Multi choice write | `[{ Value: "A" }, { Value: "B" }]` |
| Person field write | Full `SharePointPerson` object via `Office365UsersService` |
| Claims string | `i:0#.f|membership|${upn}` — use UPN not email |
| Load choice options | `ListService.getReferencedEntity('ColumnName')` |
| UPN vs Email | Never assume equal — always resolve via Office365Users profile |
