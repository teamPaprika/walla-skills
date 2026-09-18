# Custom field workflows

Every Walla plan can use custom fields. Plans differ only in how many custom field types a team can own. Use the MCP's `custom-fields` reference as the authoritative model for data references, output schemas, and host interaction.

## Reuse an existing type

1. Call `list_custom_field_types(teamId)` before proposing a new build.
2. If a suitable type exists, use its real `customFieldTypeId` with `form_apply_edits` and `fieldType: CUSTOM`.
3. Re-read the form and verify the attached type and version information.

## Build a new type

Follow the state machine exactly:

1. Call `start_custom_field_build` with the team, a clear name, and a concrete description of the field's behavior and saved value. To change an existing type, pass its `customFieldTypeId` as `targetTypeId`; approval then adds a new version to that type.
   - If the team already owns its plan's maximum number of types, starting or approving a build for a new type returns `payment_required`. Tell the user that an upgrade is necessary, and offer to revise an existing type. A revision does not count against the limit.
2. Call `get_custom_field_scaffold` immediately. Do not wait for `get_custom_field_build` to show `awaiting_your_code`: only a ready scaffold response moves the build to that state. While the scaffold returns `{ notReady: true, retryAfterMs }`, wait that long and call it again.
3. Treat the returned contract notes, SDK types, example, and file tree as authoritative. Do not invent SDK APIs, imports, or unsupported files.
4. Implement the smallest complete field, then call `submit_custom_field`. The submission is a merge patch over the scaffold and must include `outputSchema.json`. A field that reads other fields' answers must also include `observeFields.json` with `{"mode":"configured"}`; without it, the field receives no form context.
5. Poll `get_custom_field_build`. If `submit_custom_field` returns `try_again`, the build is busy; wait and repeat the same submit. On `build_failed`, use the returned errors to revise and resubmit.
6. At `ready_to_review`, give the user the returned `previewUrl` and ask them to review the rendered field.
7. Call `approve_custom_field_build` only after the user has approved the preview. Approval registers a reusable type and immutable version.
8. Attach the returned `customFieldTypeId` to a form with `form_apply_edits`, then verify with `get_form`.

## Data and privacy rules

- Declare the value shape through the build's output schema and keep it aligned with what the field saves.
- Request only the form or hidden-field data the component genuinely needs. Never add broad data references for convenience.
- Use real field IDs from the target form, including same-batch aliases only where the editing contract permits them.
- Do not make a custom field observe itself.

## Build lifecycle

- A team can have only a few builds in flight, and an unapproved build expires. If an abandoned build occupies a slot, offer `cancel_custom_field_build` only after the user confirms they no longer want it.
- Cancellation is irreversible for the build and tears down its preview. It cannot cancel an approved version.
- Do not start a replacement build merely to work around a transient state. Poll or resubmit within the existing build as the returned state directs.
