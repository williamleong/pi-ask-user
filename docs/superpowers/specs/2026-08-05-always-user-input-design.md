# Always-Available User Input Design

## Goal

Every option-based `ask_user` prompt must let the user either choose a listed option or enter a custom response. It must also let the user attach optional context to any structured selection. Callers must not be able to disable either capability.

## Public API

Remove `allowFreeform` and `allowComment` from the registered tool schema. This prevents models from treating user input capabilities as per-call policy. Keep runtime compatibility with old session data by tolerating these keys if they appear, but ignore their values.

Keep `allowMultiple`, display settings, shortcuts, and timeout behavior unchanged. Remove `PI_ASK_USER_ALLOW_COMMENT`; fixed behavior no longer needs a preference.

## Runtime Behavior

For prompts with options, the extension always constructs the custom UI and dialog fallback with freeform and comment support enabled. The UI always includes the custom-response row and the extra-context toggle.

For prompts without options, the existing direct text input remains. Such prompts are already entirely freeform, and there is no structured selection to annotate, so the extension does not request a second comment.

## Compatibility

Legacy tool calls may still contain `allowFreeform` or `allowComment`. The extension ignores both fields rather than rejecting or honoring them. Existing response data remains unchanged:

- structured answers use `selection` with an optional `comment`
- custom answers use `freeform`

## Documentation

Update the README, bundled skill, and changelog to describe the fixed policy. Remove the obsolete environment-variable instructions and per-call parameters.

## Testing

Add regression coverage that passes both legacy flags as `false` and verifies that the structured UI still exposes and accepts custom responses and extra context. Update tests that relied on disabling these controls for navigation or layout setup. Run the complete Bun test suite when Bun is available; otherwise report the missing runtime and run static package validation.
