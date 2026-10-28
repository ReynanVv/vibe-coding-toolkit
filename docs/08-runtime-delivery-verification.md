# Runtime and delivery verification

Static checks are necessary evidence, not proof that a user-facing change works. A build can pass while a conditional route throws at runtime, a responsive navigation wraps incorrectly, a role switch lands on an invalid page, or the deployed service is still serving an older revision.

Use this guide for applications with interactive UI, multiple roles/states, external services, or deployment workflows.

## Verification model

Verify the smallest matrix that covers the behavior changed:

| Dimension | Examples | Why it matters |
|---|---|---|
| Identity | signed out, signed in | Authentication gates can render different trees |
| Role/profile | student, trainer, admin | Conditional branches may reference different modules, symbols, and data |
| Route/page | dashboard, account, messages | A valid build does not execute every route |
| Data state | empty, populated, loading, error | Empty states and async branches often differ structurally |
| Viewport | narrow mobile, wider mobile, desktop | Item count, fixed bars, overflow, and safe areas change |
| Mutation state | idle, pending, success, failure | Users need clear acknowledgement without duplicate feedback |
| Release state | local, CI artifact, deployed revision | Passing CI does not prove the intended commit is live |

Do not test every Cartesian combination. Select combinations touched by the diff and at least one adjacent high-risk path.

## Before editing

1. Identify the user-visible flow, roles, routes, breakpoints, and data states affected.
2. Search for all call sites before removing or renaming shared imports, assets, components, routes, state keys, or CSS selectors.
3. Note behavior that exists only in conditional branches. These paths need runtime evidence even when lint/build pass.
4. Define what proves completion: commands plus a focused interaction/smoke path when an interface exists.

## Runtime smoke tests

After a nontrivial UI or routing change, execute the affected path rather than relying only on compilation.

At minimum, verify:

- the changed route renders;
- each affected role/profile can enter its valid landing page;
- switching role/profile resets or redirects invalid page state;
- loading, empty, success, and error states do not throw;
- console/runtime errors are absent for the tested flow;
- network requests use the intended endpoint and failure handling remains visible;
- a removed shared symbol is not still referenced by a branch that was not rendered during implementation.

When browser automation or DevTools is available, use it. Otherwise, record the limitation and compensate with focused tests, source search, and explicit manual smoke instructions. Do not claim a route was tested if it was only built.

## Responsive navigation and layout

Responsive checks must use the actual rendered item count and content.

- Compare navigation item count with grid/flex rules at every affected breakpoint.
- Avoid stale hard-coded column counts after adding or removing navigation entries. Prefer a layout driven by the rendered structure when practical; if the count is intentionally fixed, add a regression check or documented invariant.
- Check long labels, localization, dynamic badges, avatars, and safe-area insets.
- Verify fixed bottom bars do not wrap into a second row, cover content, or move controls outside the tappable surface.
- Test at least one narrow mobile width and one desktop width for changes to shared navigation or page chrome.
- Preserve keyboard focus, touch target size, and reduced-motion behavior.

## Action acknowledgement

Every meaningful asynchronous action needs a visible lifecycle, but not every click needs a toast.

Use this hierarchy:

1. **Pending:** show progress on the initiating control when possible; disable duplicate submission when appropriate.
2. **Success:** use global acknowledgement only when the resulting state is not already obvious.
3. **Validation/error:** keep actionable validation close to the field/form; use global error only when context could disappear or the failure is application-wide.
4. **No duplicate feedback:** if the UI visibly changes in a self-explanatory way, do not add an extra toast.

Keep the semantic contract separate from the renderer where mobile migration is plausible. Business/application code should emit meanings such as success, error, info, or warning; web/mobile adapters decide how those meanings are displayed.

## Changing workflows safely

When replacing a workflow rather than merely extending it:

- enforce the new rule in the backend, not only in the UI;
- prevent old clients or stale code from bypassing the new rule;
- preserve compatible history/data unless deletion is explicitly required;
- make mutations idempotent where retries are expected;
- notify the correct actor only after the state transition actually succeeds;
- avoid duplicate notifications by tying them to a durable transition/request identifier;
- update product rules and API expectations together.

Example: replacing “public profile creates an automatic relationship” with “student sends request, trainer accepts” requires disabling the direct-link mutation, not only changing the button.

## Assets and shared components

Replacing a logo, icon, or shared component can affect branches unrelated to the visible change.

Before deleting an import or helper:

1. search the whole project for the symbol;
2. distinguish the use being replaced from unrelated uses;
3. run the route/profile smoke matrix after the change.

A successful production build is not sufficient evidence when the bundler does not execute all conditional component branches.

## Deployment verification

Treat these as different states:

- code committed;
- CI passed;
- deployment triggered;
- build completed;
- rollout/update in progress;
- exact revision live;
- live smoke verified.

Do not report a deployment as live while the platform says build/update is still in progress.

For an authorized deployment:

1. capture the intended commit/revision;
2. trigger or observe deployment;
3. confirm the service reports that exact revision as live;
4. inspect startup/migration logs when the change depends on schema or boot behavior;
5. perform a focused live smoke when tools and authorization allow it;
6. report any unverified layer explicitly.

## Regression checks worth automating

Add a small automated check when the same class of failure could recur:

- navigation item count vs. responsive layout invariant;
- role/profile landing routes;
- route rendering for each major role;
- semantic feedback normalization;
- old endpoint disabled after a workflow replacement;
- idempotent relationship/request mutations;
- schema migration boot check;
- imports/symbols covered by lint/typecheck in projects where the bundler alone is permissive.

Prefer a focused regression over a broad brittle snapshot.

## Delivery checklist

Before calling an interactive change complete:

- [ ] relevant lint/types/tests/build pass;
- [ ] affected role/profile/route smoke path renders;
- [ ] changed mobile navigation/layout fits at a narrow viewport;
- [ ] async action has sensible pending/success/error acknowledgement;
- [ ] removed/renamed shared symbols were searched globally;
- [ ] backend enforces workflow changes, not only the UI;
- [ ] exact deployed revision is live when deployment is in scope;
- [ ] startup/migration logs are healthy when relevant;
- [ ] limitations are stated instead of inferred away.

The goal is proportional evidence: enough to catch failures that static checks cannot see, without turning every small change into a full end-to-end suite.
