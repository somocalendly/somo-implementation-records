# Signadot test plan

Repeatable workflow for writing a **QA-only** browser test plan that the Claude `signadot-browser-test` skill can execute on staging through a Signadot sandbox. The output is a separate file from the implementation plan. Engineers do not follow this file. QA testers do not need the implementation plan.

Use this skill when:

- A ticket needs verification on a running staging page (Signadot sandbox or base staging when the plan says so).
- Unit tests, Storybook, or Chromatic cannot prove the fix alone.
- The ticket has a `needs QA` label (or the person explicitly asks for a Signadot test plan).

Do **not** use this skill when automated tests and Chromatic are enough, or when the change is backend-only with request specs as proof. The ticket-to-PR skill covers that path.

## How to start

**Inputs**

1. Jira ticket key or URL.
2. The ticket folder workspace, or a path where the file will live.
3. The matching `sch_<key>_implementation_plan.plan.md` when it already exists (read **Goal** and **Design** only — do not copy engineer testing steps into the test plan).
4. A fresh clone in the ticket folder when the discriminating behavior is unclear from the plan alone. Read the code path the bug hits.

**Output**

One file in the ticket folder:

`<ticket-folder>/sch_<key>_test_plan.plan.md`

`<key>` is lowercase, as in `sch_4261_test_plan.plan.md`.

**Delivery**

- Attach the file to the Jira ticket when the pull request is created (or when the person asks). Do not embed the checks in the implementation plan.
- Do not edit the implementation plan’s `## Testing` section to add these browser steps. That section stays engineer-only.

## Engineer and QA boundary

| Role | Reads | Writes | Stops when |
| --- | --- | --- | --- |
| Engineer | Implementation plan | Code, unit/component tests, Storybook, Chromatic | Automated checks and CI green; teammate code review complete |
| QA | Test plan only | Nothing in the repo; runs Claude Signadot browser-test | One Jira comment from the test run |

The test plan must stand alone: setup, checks, comment template, and result rule with no pointer like “see implementation plan.”

The implementation plan must not name a specific tester, list sandbox setup for QA, or duplicate checks from the test plan.

## Before writing checks

1. **State that distinguishes fixed from unfixed.** For every mandatory check, ask: “Would this pass on `main` without the fix?” If yes in the default staging account, the setup must require the account state where old and new behavior differ (for example no connected calendar, a specific feature flag, a narrow viewport). Say that in setup and add a stop condition if the tester is in the wrong state — usually **stop and ask**, not Fail.
2. **Starting URL.** The tester (not the engineer) chooses an exact page URL after setup: booking page, editor URL, etc. The plan describes constraints (host is not the logged-in user, preview mode, minimum width). It never hard-codes a personal URL in the implementation plan.
3. **Sandbox label.** Default for booking overlay deploys: `staging-overlay-sandbox`. Use another label only when the ticket’s deploy path requires it (for example mf-publisher MFE sandbox). The banner text in stop conditions must match.
4. **Optional checks.** Viewport narrower than desktop Chrome allows (~500px), toggling flags the page cannot toggle, flag-off paths with no UI control — mark **optional**, exclude from overall result, and instruct Claude not to ask the tester about them.

Canonical shape and rationale: [signadot-browser-test/references/test-plan-format.md](signadot-browser-test/references/test-plan-format.md).

Reference examples in this repo’s bug-crush set: `sch_4261_test_plan.plan.md`, `sch_4590_test_plan.plan.md`, `sch_4738_test_plan.plan.md`, `sch_4748_test_plan.plan.md`, `sch_4749_test_plan.plan.md` (when present in the workspace or ticket folder).

## File template

Use this skeleton. Replace ticket-specific setup, stops, checks, and comment fields.

```markdown
# SCH-#### Signadot test plan

Ticket: [SCH-####](https://calendlyapp.atlassian.net/browse/SCH-####)

## For the QA tester

You do not need to write a test plan or copy its checks into Claude. This test plan is already attached to SCH-####, generated when the pull request was created.

1. Put the `<sandbox-label>` label on the pull request and wait for the sandbox to be running. (Omit PR label steps when the plan says base staging only.)
2. In Chrome, turn on Signadot routing for that sandbox and log in to staging as the test user for this run. Record the name and email shown in the account menu. <Account preconditions — locale, flags, calendar state, viewport, which product area to open.>
3. <How to reach the starting page and copy the exact URL — not the staging home page.>
4. Download `sch_####_test_plan.plan.md` from SCH-####.
5. In Claude, attach the file and invoke the Signadot browser-test skill with:

   `Test SCH-#### on Signadot. Starting URL: <exact URL from step 3>`

Claude runs the checks in Chrome, asks a specific question if it gets stuck, and posts one comment on SCH-####. Do not perform the checks manually unless Claude asks for help with setup. Do not change the ticket status.

Everything below this line is instructions for Claude. Any file in `references/` is guidance for authors of test plans and is not part of this test run.

---

## Instructions for Claude

Inputs:

- Jira ticket key: `SCH-####`
- Test plan: this file
- Starting URL: the exact URL in the invocation

Assume the QA tester completed the setup above. Do not log in, type credentials, switch accounts, change account state, toggle feature flags, or guess a URL. After loading the starting URL, verify that a visible sandbox banner includes `<sandbox-label>`. If the banner is absent or names a different sandbox, stop and ask the tester before running a check.

## Stop before any check

- Login wall: post the comment with every check Not run and Result Blocked. Stop.
- Sandbox banner missing or wrong: stop and ask the tester. Do not run checks. Do not comment yet.
- <Ticket-specific stops — wrong page, wrong host, wrong preview mode, wrong account state with “stop and ask” when the run cannot prove the fix.>
- If a required check cannot be done as written, stop and ask the tester. Name the check, what blocked it, and one proposed alternative. Do not improvise.

<Global comparison rules: verbatim strings, U+2019 apostrophes, API fetch substitute if needed.>

## Checks

### 1. <Short name>

Action: <concrete UI steps>

Pass: <observable condition with exact strings>

Evidence: <what to record>

<Repeat numbered checks. Mark optional checks and state they are excluded from the result.>

## Comment

Post exactly one comment on SCH-####. Do not change the ticket status. Do not attach a file.

```
<Exact comment template with placeholders per check and Result line>
```

Result, from checks <N> through <M> only:

- Pass if every one of those is Pass.
- Fail if any of those is Fail.
- Blocked if any of those is Not run and none is Fail.

<Optional checks never change the result.>
```

## Writing workflow

1. Load the Jira issue (summary, description, acceptance criteria, screenshots, comments).
2. Read **Goal** and **Design** from the implementation plan if present. If not, read the ticket and code.
3. List observable behaviors that must change with the fix. Drop anything already covered by unit tests unless the ticket still needs a staging sanity check.
4. For each behavior, write Action / Pass / Evidence. Prefer DOM-visible proof over network proof unless the ticket is flag- or API-gated.
5. Add stop conditions for every setup requirement that, if wrong, would make a Pass meaningless.
6. Draft the comment template and result rule. Count only mandatory checks in the result.
7. **Self-review** (before handing to the person):
   - [ ] Mandatory setup includes the discriminating account or page state.
   - [ ] No step assumes the engineer already ran Signadot.
   - [ ] No reference to the implementation plan below the “For the QA tester” section.
   - [ ] Every Pass line uses exact expected copy where copy matters.
   - [ ] Optional checks are labeled and excluded from Result.
   - [ ] Sandbox label in setup matches banner check in Instructions for Claude.
   - [ ] Invocation line uses `Test SCH-#### on Signadot. Starting URL: ...`

## Coordination with ticket-to-PR

When this skill runs as part of the same ticket as `ticket_to_pr_skill.md`:

- Write the test plan **before Gate 1** or **after Gate 1** when the person asks, but **before** the pull request is opened for QA handoff.
- The implementation plan’s `## Testing` section should already say the engineer stops after automated checks and code review, with no Signadot steps.
- At pull-request time, attach `sch_<key>_test_plan.plan.md` to Jira. The engineer does not paste test-plan steps into the GitHub PR body; the PR may say QA uses the attachment and the `needs QA` process.

## What this skill does not do

- Run the browser checks (use `signadot-browser-test`).
- Edit application code or the implementation plan’s approved design sections.
- Change Jira status, assign QA, or add the sandbox label (tester's and engineer's process per team guide).
- Replace Chromatic, Postman, or request specs when those are the real proof.
