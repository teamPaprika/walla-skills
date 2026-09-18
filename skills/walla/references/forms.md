# Form workflows

## Read and choose the correct version

1. Discover IDs with `list_teams` → `list_workspaces` → `list_forms`.
2. Use `get_form` for the editable draft, its fields, friendly conditional logic, and groups.
3. Use `get_published_form` for the live version respondents see. A never-published form returns `not_found`; this does not mean its draft is missing.

## Create a form

1. Resolve a real team and workspace.
2. Call `create_form` only after the user has asked to create a form. It creates an empty, title-only draft.
3. Read `field-types`, `editing-contract`, and, when needed, `branch-logic` through the MCP resource or `read_reference`.
4. Add questions with `form_apply_edits`.
5. Re-read with `get_form` and verify labels, descriptions, options, order, groups, appearance, and logic.
6. Publish only if the user explicitly asks to make the form live. Call `publish_form`, then verify with `get_published_form`.

## Edit a form

1. Read `get_form(formId)` immediately before editing; use its current field, option, and group IDs.
2. Build the smallest batch that fulfills the request. `form_apply_edits` can add, update, delete, or reorder fields; set or clear logic; set field descriptions; update appearance; and update group metadata.
3. Preserve unrelated content and settings. Do not delete or reorder fields unless the request requires it.
4. Treat the batch as an atomic draft change. If validation fails, fix the payload instead of splitting it into partially applied guesses.
5. Re-read the draft after success. Do not publish automatically.

## Author questions clearly

- Use `label` as a short internal name for response tables.
- Put the complete respondent-facing question in the rich-text description. Never leave it empty and do not merely repeat a terse label.
- Read `field-descriptions` before producing formatted content. Use the HTML form accepted by the editing contract.
- Read `field-types` before setting properties, options, validations, or answer-dependent logic.
- Preserve opaque option IDs when editing existing choices. Never substitute labels or values where the contract requires option IDs.

## Order, sections, and pages

- Bare `add_field` operations append in call order just before the system submit step. Never anchor a new field to the submit step.
- Give an added field a `tempId` when later operations in the same batch must reference it. Give new choice options stable keys and reference them as `@key`; reference added fields as `@tempId`. Aliases resolve only backward within the batch and must be unique.
- To keep several questions in one section, add the section's first field, then add following fields anchored to it with `intoGroup: true`.
- Re-read the form to obtain server-created `groupId` values before calling `set_group`.
- Use `set_group` with `PRESS_BUTTON` for a page break and `APPEND` to continue on the same page. `set_group` can target only groups that existed before the batch.
- Do not put `reorder` in the same batch as `add_field` or `set_group`. Add fields first, then reorder or set groups in a separate call.

## Conditional logic

1. Read `branch-logic` and the source form first.
2. Use `set_field_logic` to replace a field's complete friendly rules/otherwise shape; use `clear_field_logic` to remove its rules.
3. For choice conditions, use the real option IDs from `get_form` in `choiceIds`. For text, number, or date conditions, use a literal `value`.
4. Evaluate connectors left to right. `[A or B and C]` means `((A or B) and C)`; there is no operator precedence.
5. `set_field_logic` replaces the field's whole logic. To keep a custom `otherwise` path, copy it from `get_form` and send it again. If you omit it, the default path resets to the next field.

## Appearance and publishing

- Use `set_form_appearance` for colors and next-button behavior. Send only keys the user wants changed; the operation partial-merges.
- Treat `publish_form` as a separate explicit action that snapshots the current draft into the live version. Re-publishing refreshes the live version in place and leaves the draft editable.
- Verify draft edits with `get_form` and publication with `get_published_form`.
