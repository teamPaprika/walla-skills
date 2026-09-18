---
name: walla
description: Operate connected Walla workspaces and forms through Walla MCP. Use when the user wants to discover teams, workspaces, or forms; read, create, edit, organize, style, or publish a form or survey; inspect masked responses; create or update webhook automations; or build and attach custom fields.
---

# Walla

Use the connected Walla MCP server to manage the user's Walla data. Ground every action in tool results; never invent account, team, workspace, form, field, option, automation, custom-field, or build IDs.

## Core workflow

1. Start discovery with `list_teams`, then call `list_workspaces(teamId)` and `list_forms(teamId, workspaceId)`. Pass returned IDs forward.
2. Read the current object before changing it. Use `get_form(formId)` for the editable draft and `get_published_form(formId)` for the live form.
3. Load only the task-specific guide listed below. For exact field and edit schemas, read the MCP resource or call `read_reference`; do not reconstruct schemas from memory.
4. Perform writes only when the user's request clearly asks for that write. Treat editing the draft and publishing it as separate operations.
5. Re-read the affected form, published form, or automation after a write and report what changed.

## Route the request

- For form discovery, creation, editing, layout, appearance, logic, or publishing, read [references/forms.md](references/forms.md).
- For submitted answers, summaries, exports within the available response tool, or webhook automations, read [references/responses-and-automations.md](references/responses-and-automations.md).
- For reusing, building, previewing, approving, or attaching a custom field, read [references/custom-fields.md](references/custom-fields.md).

## Reference contract

Prefer MCP resources at `walla://reference/*`. If the client cannot read resources, call `read_reference` with one of:

- `form-model`
- `field-types`
- `branch-logic`
- `field-descriptions`
- `custom-fields`
- `editing-contract`

Use `field-types`, `editing-contract`, and `branch-logic` as the authoritative inputs before creating or editing questions. The same documents are secondary fallbacks at `https://docs.walla.my/mcp-reference/<name>.md`.

## Safety and availability

- Use the authenticated identity from the connection. Never ask the user for a user ID or token.
- Keep read-only requests read-only. Do not create, edit, publish, enable an automation, or approve a custom-field build as a side effect.
- Never imply that response values are unmasked. `list_responses` masks configured PII and omits file contents and download URLs.
- If a documented Walla tool is missing, the connection probably does not have that tool's OAuth scope. Ask the user to reconnect Walla and grant the capability. If the tool is still missing after reconnection, this Walla deployment does not offer the feature. Do not retry with fabricated arguments.
- Expect some features to depend on the team's plan. Preserve the rest of the requested work when a paid feature can be cleanly omitted, but ask before changing the user's requested scope.

## Handle errors

- On `forbidden`, stop retrying. The user does not have permission on that team, workspace, or form, or a team admin disabled Walla MCP (or this capability) in the team's security settings. A team with MCP fully disabled does not appear in `list_teams`.
- On `not_found`, re-run discovery and verify that the ID came from the correct team or workspace.
- On `invalid_args`, read the actionable message and the relevant reference, correct the payload, and retry.
- On `payment_required`, explain which feature needs an upgrade.
- On `try_again`, wait briefly and repeat the call that returned it, with the same arguments.
- On `internal`, do not guess at a fix or expose internal details; report that Walla could not complete the operation. If a write returned `internal`, re-read the object before you retry, because `form_apply_edits` is not idempotent. For `approve_custom_field_build`, call `get_custom_field_build` and then approve again; approval resumes safely.

## Deliver the result

Answer in the user's language. Name the affected team, workspace, form, and status when known. For writes, summarize the verified result and clearly distinguish draft changes from the live published form.
