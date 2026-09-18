# Responses and automations

## Read responses

1. Resolve the form through the normal discovery chain and read its draft or published fields so answer columns have context.
2. Call `list_responses(formId, offset, limit)` only when the user asks to inspect submitted answers. The token needs the separate response-read scope.
3. Page while `hasMore` is true. Use the returned `offset`, `limit`, `returned`, and `totalCount`; the maximum page size is 50.
4. Treat values as masked. Never claim to have unmasked PII. File answers are placeholders, not downloadable files or URLs.
5. Note that rows are newest-first and offset pagination is not a stable snapshot while new submissions arrive. Restart from offset 0 when a consistent fresh read matters.

Summarize only the rows actually retrieved. State the covered count or page range when the user may otherwise assume the summary covers every response.

## Inspect automations

1. Use a published form ID. A never-published draft cannot have an attached automation.
2. Call `list_automations` before creating or updating one to obtain IDs, inspect current state, and avoid duplicates.
3. Treat complex dashboard-built graphs as read-only when their summary says they cannot be represented safely.

## Create a webhook automation

1. Proceed only when the user explicitly asks to send Walla responses to a webhook or external server.
2. Read `get_published_form` for real field and choice-option IDs. Read `branch-logic` when conditions are needed.
3. Call `create_automation` with one webhook action. Email and Google Sheets automations are dashboard-only in this MCP version.
4. Use a public endpoint that the user supplied or approved, and prefer HTTPS. The server rejects private, loopback, and internal hosts. Do not invent endpoints or send response data to an unrelated service.
5. For a custom body, use Walla mentions such as `@[Label]{{res:<fieldId>}}`, `@[Label]{{hidden:<key>}}`, `@[Response]{{responseKey}}`, `@[Customer]{{customerKey}}`, `@[Form]{{projectKey}}`, or `@[Time]{{timestamp}}`.
6. The default payload includes the response's hidden-field values (values captured from URL query parameters). Set `includeHiddenFields: false` when the user does not want to send them. This setting does not change a custom body.
7. Report that the new automation is inactive. Do not enable it unless the user asks.

## Update or enable an automation

1. Get the automation ID and current state from `list_automations`.
2. Use `update_automation` only for a simple single-route webhook automation. Leave multiple routes and email or Sheets actions for the dashboard.
3. Send only the fields the user asked to change. An empty custom `body` resets delivery to the default full-response payload; omitting it preserves the current body. Omitting `includeHiddenFields` or `conditions` also preserves the current setting, but `conditions: []` or `null` makes the automation fire on every response.
4. Before setting `isActive: true`, ensure the request explicitly asks to enable delivery. Tell the user that new responses will start going to the configured endpoint immediately.
5. Verify the result with `list_automations`.

There is no MCP delete-automation operation. Direct deletion requests to the Walla dashboard instead of approximating deletion by unrelated edits.
