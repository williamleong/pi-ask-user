# Always-Available User Input Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make freeform responses and selection comments mandatory capabilities of every applicable `ask_user` prompt.

**Architecture:** Remove the two policy switches from the tool schema and force both internal UI flags to `true`. Legacy arguments remain harmless because the runtime ignores them. Keep direct freeform behavior for calls without options.

**Tech Stack:** TypeScript, TypeBox, Pi extension API, Bun test runner.

## Global Constraints

- Option-based prompts always expose a custom-response row and an extra-context toggle.
- Calls without options remain a single direct freeform input.
- Legacy `allowFreeform` and `allowComment` values are ignored.
- Response shapes and unrelated display, shortcut, timeout, and multi-select behavior remain unchanged.

---

### Task 1: Enforce fixed user-input capabilities

**Files:**
- Modify: `index.test.ts`
- Modify: `index.ts`

**Interfaces:**
- Consumes: existing `ask_user` tool registration and `AskComponent` constructor.
- Produces: an `ask_user` schema without `allowFreeform` or `allowComment`; structured UI always receives `true` for both internal flags.

- [ ] **Step 1: Write failing schema and runtime regression tests**

Add tests that inspect the registered parameter schema and execute a legacy call with both flags set to `false`:

```ts
test("does not expose per-call freeform or comment switches", async () => {
   const tool = await setupTool();
   expect((tool as any).parameters.allowFreeform).toBeUndefined();
   expect((tool as any).parameters.allowComment).toBeUndefined();
});

test("ignores legacy false values and keeps both controls available", async () => {
   const tool = await setupTool();
   let rendered = "";
   await tool.execute(
      "tool-call-id",
      {
         question: "Which option should we use?",
         options: ["Alpha", "Beta"],
         allowFreeform: false,
         allowComment: false,
      },
      undefined,
      undefined,
      {
         hasUI: true,
         ui: {
            custom: async (factory: any) => {
               const component = factory(
                  { requestRender() {}, terminal: { rows: 24 } },
                  createTheme(),
                  createKeybindings(),
                  () => {},
               );
               rendered = component.render(80).join("\n");
               return null;
            },
         },
      },
   );
   expect(rendered).toContain("Add extra context after selection");
   expect(rendered).toContain("Type something.");
});
```

Update `RegisteredTool` in the test harness to expose `parameters`.

- [ ] **Step 2: Run the focused tests and verify RED**

Run:

```bash
bun test index.test.ts --test-name-pattern "per-call freeform|legacy false"
```

Expected: FAIL because both schema fields exist and legacy `false` hides both controls.

- [ ] **Step 3: Implement the fixed policy**

In `index.ts`:

- remove `allowFreeform` and `allowComment` from `AskParams`
- remove both entries from the TypeBox parameter schema
- remove `parseBooleanPreference`
- stop destructuring either field from `params`
- define the internal flags immediately after destructuring:

```ts
const allowFreeform = true;
const allowComment = true;
```

Keep passing these internal flags through existing custom UI, headless messaging, and fallback dialog paths.

- [ ] **Step 4: Update tests that assumed controls could be disabled**

Remove obsolete environment-precedence tests. Adjust navigation, viewport, and fallback-dialog fixtures so their indices and expected option lists include the always-present comment and freeform controls. Preserve each test's original behavioral assertion.

- [ ] **Step 5: Run tests and verify GREEN**

Run:

```bash
bun test
```

Expected: all tests pass with no warnings or errors.

- [ ] **Step 6: Commit runtime behavior**

```bash
git add index.ts index.test.ts
git commit -m "feat: always allow user-supplied ask responses"
```

### Task 2: Document the fixed policy

**Files:**
- Modify: `README.md`
- Modify: `skills/ask-user/SKILL.md`
- Modify: `CHANGELOG.md`

**Interfaces:**
- Consumes: fixed runtime behavior from Task 1.
- Produces: user and agent guidance with no obsolete per-call or environment switches.

- [ ] **Step 1: Add failing documentation assertions**

Extend the existing bundled-skill test to assert that the README and skill describe both capabilities as always available and do not advertise `PI_ASK_USER_ALLOW_COMMENT`.

- [ ] **Step 2: Run the documentation test and verify RED**

Run:

```bash
bun test index.test.ts --test-name-pattern "always available"
```

Expected: FAIL because current documentation still describes optional switches and environment precedence.

- [ ] **Step 3: Update documentation**

- Remove both parameters and `PI_ASK_USER_ALLOW_COMMENT` from `README.md`.
- State that option prompts always include custom response and extra context controls.
- Change the bundled skill handshake to omit both arguments and describe the fixed UI behavior.
- Add an Unreleased changelog entry under `Changed`.

- [ ] **Step 4: Run complete verification**

Run:

```bash
bun test
npm run check
```

Expected: all tests pass and `npm pack --dry-run` succeeds.

- [ ] **Step 5: Commit documentation**

```bash
git add README.md skills/ask-user/SKILL.md CHANGELOG.md index.test.ts
git commit -m "docs: describe fixed ask input controls"
```
