# copilot-studio.md — Microsoft Copilot Studio Service Integration

Read this whenever working with `MicrosoftCopilotStudioService` or the `shared_microsoftcopilotstudio` connector.

---

## 1. Adding the Data Source

```bash
pac code add-data-source -a "shared_microsoftcopilotstudio" -c "<connectionId>"
```

Verify `power.config.json` afterwards — it should contain an entry under `connectionReferences` with `"id": "/providers/Microsoft.PowerApps/apis/shared_microsoftcopilotstudio"`.

---

## 2. 🚨 Critical Bug — `x-ms-conversation-id` Header

The PAC CLI generates `MicrosoftCopilotStudioService.ts` with a **known bug**: it internally converts the `x-ms-conversation-id` header to `x_ms_conversation_id` (underscores), which the REST client ignores — breaking conversation context.

**This is the ONE exception to the "never edit generated files" rule.**

Manually patch `src/generated/services/MicrosoftCopilotStudioService.ts`:
- Find where the conversation ID header is assembled
- Force it as `params['x-ms-conversation-id']` (hyphenated) before it's passed to the HTTP client

All other generated-file rules still apply. If the schema changes later: delete + re-add the data source, then re-apply this patch.

---

## 3. Calling `ExecuteCopilotAsyncV2`

The response payload is **not type-safe** — it frequently arrives as stringified JSON and property casing (`conversationId` / `ConversationId` / `conversationID`) can vary between API versions. Always use defensive parsing:

```typescript
const response = await MicrosoftCopilotStudioService.ExecuteCopilotAsyncV2(
  "ca_agent",  // your agent/schema name
  {
    message: messageToSend,
    notificationUrl: 'https://notificationurlplaceholder'  // placeholder required by API
  },
  activeConversation.conversationId
);

if (response.success && response.data) {
  // ⚠️ Data may be stringified — always check
  const data: any = typeof response.data === 'string'
    ? JSON.parse(response.data)
    : response.data;

  // Extract agent response safely
  const agentText = data.responses?.length > 0
    ? data.responses[0]
    : "No valid response received.";

  // ⚠️ Casing varies — check all variants
  const convId = data.conversationId ?? data.ConversationId ?? data.conversationID;

  if (convId) {
    updateConversation(activeConversation.id, (c) => ({
      ...c,
      conversationId: convId
    }));
  }
}
```

---

## 4. Anti-Patterns

| ❌ Don't | ✅ Do instead |
|---|---|
| Assume `response.data.responses` is always an array | Check `typeof response.data === 'string'` first and parse |
| Hardcode `conversationId` casing | Check `conversationId ?? ConversationId ?? conversationID` |
| Edit other parts of the generated service | Only patch the header bug — handle everything else in hooks/wrappers |
| Use `response.data.conversationId` directly | Always guard with nullish coalescing across casing variants |

---

## 5. Quick Reference

| Step | Command / Note |
|---|---|
| Add connector | `pac code add-data-source -a "shared_microsoftcopilotstudio" -c <id>` |
| Patch header bug | Edit generated file: `params['x-ms-conversation-id']` (only this change) |
| Parse response | Always try `JSON.parse(response.data)` if `typeof === 'string'` |
| Get conversation ID | `data.conversationId ?? data.ConversationId ?? data.conversationID` |
| Schema refresh | Delete + re-add data source, then re-apply header patch |
