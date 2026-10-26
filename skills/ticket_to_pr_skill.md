# Ticket to pull request

Repeatable agent workflow for one bug: read the ticket, write or verify an engineering design document that is also the implementation plan, implement it, verify it, commit, push, and hand back pull-request content.

This is a skill the agent follows when asked to run a ticket or a plan through to a pull request. It is not a Cursor Automation. The stops below require a person, and an unattended automation cannot take them.

When a change needs a review from outside the squad, the agent prepares the request. The person posts it and requests the team. The agent does not post to Slack, does not request reviewers, and does not wait on the review.

## How to start

One input per run: a ticket, or an implementation plan that already exists. The agent stops at each gate in this file and waits. It does not treat silence, an earlier "go", or a later message about a different topic as approval to pass a gate.

### Ticket

Give the agent a ticket key or URL and this file. The workspace is the ticket folder the person created for this run.

Example: "Run SCH-4257 through `ticket_to_pr_skill.md`."

Follow Stage 1 and Stage 2 as written. Do not skip ahead to implementation.

### Plan

Give the agent the plan file and this file.

Example: "Take `sch_4257_implementation_plan.plan.md` through `ticket_to_pr_skill.md`." The plan file is in the ticket folder.

The same stages apply, with Stage 1 and Stage 2 replaced by verifying that plan. Do not write a second plan. Before Gate 1, edit the given file only when verification finds a gap. Then stop at Gate 1. After an explicit yes, continue at Stage 3. After that yes, follow "The approved plan."

## Where the code lives

Each run starts in a ticket folder the person created and opened as this workspace. One ticket, one folder. The folder holds the implementation plan and a fresh clone of the repo that ticket changes. Example: `/Users/somo.banerjee/Documents/projects/SCH-4257/` contains `sch_4257_implementation_plan.plan.md` and `booking/`.

The person creates the folder and opens it. The agent does not create that folder, and this chat cannot move into another Cursor window. The `cursor` CLI can open a folder in a new window (`cursor -n <path>`) or add a folder to the last active window (`cursor -a <path>`). That window is a new session. It does not inherit this run, the plan, or the gates. The agent does not run those commands.

The clone is the agent's job. Name the repo from the ticket or the plan. Clone it into the ticket folder, on the default branch the remote HEAD points at. `git clone` checks out that branch. Do not pass a branch name.

```bash
git clone git@github.com:<owner>/<repo>.git <ticket-folder>/<repo>
```

`<repo>` is the repository name (`booking`, `calendly`, `js-packages`). The ticket folder is not the git root. Run git, tests, and edits from `<ticket-folder>/<repo>`.

Clone before reading code for Stage 1 or for verifying a plan, so the plan is checked against that default branch. Leave long-lived checkouts alone, including `/Users/somo.banerjee/Documents/projects/booking` and `/Users/somo.banerjee/Documents/projects/calendly`. They carry other work. Do not read them, fetch in them, or branch in them for this run.

If `<ticket-folder>/<repo>` is already a clone from this run, on the default branch, and clean, use it. If that directory exists and is dirty, on another branch, or not a clone of this repo, stop and ask. Do not delete it and do not clone over it.

One repo is changed per run. If the plan must change more than one repo, clone each into the ticket folder, stop, and ask which repo to implement first.

If the local check needs another app running, clone that repo into the ticket folder too. A booking page needs `calendly/calendly` beside `booking/`. That clone is there so the page can load. It is not a second change unless the plan says to edit it. Leave the long-lived checkout alone.

When the run is done, the person may delete the ticket folder. The remote branch stays. The agent deletes the folder only when the person asks.

## Gates

Three stops. Each one ends the turn. The agent shows the artifact and asks for a yes or a change. Work resumes only after an explicit yes ("approved", "go", "lgtm", or equivalent).

| Gate | When | What the person checks | Agent does until yes |
| --- | --- | --- | --- |
| 1. Plan | After the plan is written or verified, before any code edit | The design, what is out of scope, stakeholders and their policies, the local and shared-environment test plan, post-production verification, rollback, the procedure pages in `## Sources`, and any place the plan refuses a pattern the ticket named | Does not edit the clone |
| 2. Diff | After implementation and verification, before `git commit` and `git push` | Every file in the diff, and the team `CODEOWNERS` assigns to each one | Does not commit or push |
| 3. Pull request | After the branch is on the remote, the create-pull-request link is in chat, and the title and body are in chat | The person opens the link, creates the pull request, and sends any outside review the diff's `CODEOWNERS` check names | Does not open the pull request, request reviewers, or post to Slack |

A change request before Gate 1 sends the agent back through that stage, then back to the same gate. The draft is edited in place. After Gate 1, follow "The approved plan."

Cloning into the ticket folder is not a gate. It happens before the plan is written or verified, and it is not an edit of application code.

## The approved plan

Gate 1 approves the plan file as it stands. After that yes, and once implementation has started, the agent does not edit the plan.

The approved sections stay as they were approved. That includes the frontmatter. Flipping a todo to completed, noting that a pull request was approved, or recording a merge or a published version is the work. It is not the plan. Those facts stay in the chat. A status sentence that has fallen behind is not a reason to edit the file.

The branch name is not one of those status facts. It is the `branch` frontmatter field, chosen before Gate 1. Stage 3 creates that branch. The Implementation Record Skill reads `branch` from the plan and does not search for it. When the person asks for a different branch after approval, replace the `branch` value and append an `## Updates` record that quotes the previous name.

The pull request URL is the other field this run may write after approval. It is `pullRequest` in the frontmatter, added once in "Record the pull request," after the pull request exists. That write does not record approval, merge, or a published version. Those stay in the chat. The Implementation Record Skill reads `pullRequest` from the plan.

**Story points** are set while writing the plan, before Gate 1. They are frontmatter `storyPoints` (integer) and the `N SP` segment in the shareable opening note. The Implementation Record Skill reads `storyPoints` first. Do not change story points after Gate 1 unless the person asks and the change is recorded under `## Updates`.

When the code and the approved plan disagree, stop and say what differs. Do not change the code past the plan, and do not change the plan to match the code.

The plan changes only when the person explicitly asks to change the plan. Append that change under `## Updates`. Do not rewrite `## Goal`, `## Design`, `## Code changes`, or any other approved section to match the new decision. Do not add `## Updates` while writing the plan. It appears only for a decision that changes after approval.

Each update is a decision record, in the shape of an architecture decision record. The previous decision stays readable beside the new one:

- The date and a short title.
- What the approved plan decided. Name the section, and quote the decision that is being replaced.
- What replaces it.
- Why.

The agent then implements the approved plan as amended by those updates. A later update can supersede an earlier one. Both stay in the file. A code change the person asks for that still fits the approved plan, including its updates, does not need a new record.

## Stage 1. Read the ticket

1. Load the Jira issue (summary, description, acceptance notes, parent, linked issues, comments, and **story points** when the project uses them).
2. Name the repo. Clone a fresh copy into the ticket folder, as in "Where the code lives." Read that clone.
3. Find the code path the bug actually hits. Read the files. Do not plan from the ticket text alone.
4. Separate this ticket from nearby tickets that share a screen. Those stay out of scope unless the ticket says they are in scope.

## Stage 2. Write the plan

Write one markdown file in the ticket folder:

`<ticket-folder>/<ticket>_implementation_plan.plan.md`

The ticket key in the filename is lowercase, as in `sch_4257_implementation_plan.plan.md`. The plan's checkout path is `<ticket-folder>/<repo>`. Plans already under `/Users/somo.banerjee/Documents/projects/plans/` are earlier runs. This workflow does not write a second copy there.

This file is the engineering design document and the implementation plan. One artifact. Do not write a second design document. Size every section to the change. A section that does not apply still appears: one sentence says why, and names what was checked. Do not paste a full architecture template onto a change that does not need one.

Match this shape:

- YAML frontmatter with `name`, `overview`, `branch`, `storyPoints`, `todos`, and `isProject: false`. `branch` is the exact git branch name. `storyPoints` is the integer from Jira when the issue has story points. Omit `storyPoints` only when Jira has none (for example some backlog items).
- A title and a **shareable opening note** on the next lines. It links the ticket and includes squad, sizing, status, and priority in parentheses, for example `(ET Scheduling · 3 SP · Ready For Dev · P2 · Low impact)`. The `N SP` segment must match frontmatter `storyPoints`. When there are no story points, use the ticket’s status line from Jira without inventing a number. Then name the repo, state `Branch: \`<branch>\`` with the same value as frontmatter `branch`, and state the constraint that matters (version, flag, endpoint). The note says this file is the design document and the implementation plan.
- `## Goal` — what is broken and why, in the code that runs today.
- `## Design` — the approach, why this one, what was rejected, and the blast radius. A diagram only when a sentence would hide how the pieces connect.
- `## Out of scope`
- `## Code changes` — files, the behavior change, what must stay as it is, and each step a procedure or an owner's policy requires.
- `## Testing` — local, then the shared environment. The rules are below.
- `## Post-production` — the check after merge, publish, or deploy.
- `## Rollback` — how to undo the change after it has shipped, and what that undo does not restore.
- `## Sources` — ticket URL, every procedure page opened, and any comment or screenshot the plan relies on. If the procedure search found nothing, name the queries.
- `## Review` — the external stakeholders and the policies this change follows, or "None" when the only stakeholder is the squad that owns the ticket.

Choose `branch` while writing the plan. The form is `<TICKET>-<short-slug>`, as in `SCH-4257-date-label-alignment`. Stage 3 creates that branch and no other. A missing `branch` is not a finished plan. Set `storyPoints` from Jira when present. A missing `storyPoints` when Jira has points is not a finished plan. Leave `pullRequest` out until "Record the pull request." Do not put a placeholder there. Leave `started`, `pullRequestOpened`, and `released` out. The Implementation Record Skill fills those after release.

### Written procedure

Before writing `## Code changes`, search Confluence for the procedure that owns this kind of change. Read the page. Do not stop at the title.

Search from the change, not from the ticket key. A migration searches "add column", "uuid", "backfill", and "NOT NULL". An API change searches the contract and the API guidelines. A flag searches the experiment guidance. Use more than one query.

A file in the repo is an example. It is not the rule. If the ticket names a file to copy, read that file and still read the procedure. When they disagree, follow the procedure and say so in the plan, including which step of the named example is behind the page.

`## Sources` lists every procedure page that was opened, with the URL. If the search found nothing, `## Sources` says what was searched. "No page" is a result the person can see. A missing `## Sources` entry is not a finished plan.

`## Code changes` names the page and the step for each step that comes from a procedure. A person who does not know the rule should be able to see which requirement it satisfies.

The plan states the verification the bug needs. For a UI bug behind a feature flag, the shared-environment check includes how to tell the flag is on for that page load (response field, visible copy, or both). A screenshot of the flag-off page does not verify the change.

Do the external-review, Storybook, and testing checks below before the plan is finished. Their results go into `## Design`, `## Code changes`, `## Testing`, and `## Review`. A plan that names a team, a channel, a story, or a production check only at Gate 3 is not finished.

### Testing, release, and rollback

`## Testing` has two parts.

Local is what the agent runs before Gate 2, and what an engineer can run on a checkout. Name the command, what the new example asserts, the Storybook story when that rule applies, and the running-app check. A unit or component run is not the running-app check.

Shared environment is what the engineer and QA run after the pull request exists and before review. Name the tool this repo uses, the request or screen, and the proof. For `calendly/calendly`, deploy a Signadot sandbox by putting the `staging-overlay-sandbox` label on the pull request (`.github/workflows/sandbox-deploy.yml`), then route staging with the Signadot extension ([How to Signadot](https://calendlyapp.atlassian.net/wiki/spaces/ainc/pages/5908987907/How+to+Signadot)). Use Postman when an HTTP call is what proves the change. Include method, path, the body fields that matter, and where the proof shows up (log line, `identify_data`, response field). The booking response is not proof of an identify trait. Chromatic is the shared-environment check for a Storybook story. Do not name a tool the repo does not use. Include how to tell this is the path under test.

`## Post-production` is the check after merge, publish, or deploy. Name the event that makes it possible, the screen or request, and the proof. The agent does not run this before the change is in production. "Verify in production" with no proof is not a plan.

`## Rollback` is how to undo the change after it has shipped. Name the action (revert, flag off, republish the previous package) and what that action does not undo. When a procedure page owns the rollback, name the page and the step. A revert is a complete plan when that is the real undo.

### External review

Read the repo's `CODEOWNERS` for the paths this change touches. The last matching pattern wins for each file. A later line replaces an earlier one, even when the earlier line is the feature directory. The feature directory's owner does not cover a file that a later rule names.

Check `CODEOWNERS` against the planned paths while writing the plan. Check it again against every file in the diff at Gate 2, against the whole branch diff at Gate 3, and before any later push on that branch. A file the plan did not list can add an outside owner. The plan's "no outside team" line does not survive that file.

For each owner outside the squad that owns the ticket, open that team's procedure page. Read what they review, what the change has to contain, and how they want to be asked. A team name without that page is not enough.

If an owner is outside the squad and `## Review` does not already name that team, stop. Name the file, the last matching pattern, and the team. Do not say that no outside team is required. Prepare the request in the chat from that page: the GitHub team handle, why this file pulls the team in, each policy from the page this change has to follow, and a Slack message when the page names a channel. Do not invent a channel or a policy the page does not state. Do not edit the approved plan unless the person asks. The person posts the message and requests the team.

`## Review` is the stakeholder section. Include a person or team the ticket or a procedure page names, such as design, QA, or a squad that consumes the change. Do not invent one.

`## Review` names each outside team:

- The GitHub team handle.
- The Slack channel, when the page names one.
- Why that team is on this change. The sentence comes from the page or from `CODEOWNERS`, with the URL or the path.
- Each policy from that page that this change has to follow, and the part of the plan that satisfies it.

When the page says the team must make a decision, `## Review` states that decision. When the page says what the pull request must contain, `## Code changes` or `## Testing` includes it, and Stage 6 copies it into the body. When the page names a channel, `## Review` includes the facts the Slack message needs: one line of ticket context, the impact, the deadline when the ticket has one, and what is blocked.

Do not keep a list of teams in this skill. Do not invent a policy the page does not state.

When no outside team is required, `## Review` says none and names the `CODEOWNERS` path and the pages that were checked. That sentence is allowed only after the diff check above, not from the plan's file list alone. A missing `## Review` is not a finished plan.

### Storybook

While writing the plan, check whether the repo has Storybook.

When the change alters something a person sees and the repo has Storybook, the local part of `## Testing` names the story file and what the story renders. `## Code changes` includes that file. Chromatic snapshots the story. That snapshot is the shared-environment visual check, and the story is the documentation for the new state. The story renders that state on its own. A control the reviewer has to click is not the snapshot.

Include a dark-mode story when the change is about dark mode. The plan names the color mode that repo's Storybook already uses. The plan leaves an existing story in place when an unchanged visual state has to stay as it is.

Skip this when the repo has no Storybook, or the change has nothing a person can see. `## Testing` says that, and names what was checked.

### When the input is a plan

1. Read the plan file. Load the Jira issue it cites (summary, description, acceptance notes, parent, linked issues, comments, and story points when the project uses them), including comments added after the plan was written.
2. Name the repo from the plan. Clone a fresh copy into the ticket folder if this run does not already have the clean default-branch clone from "Where the code lives." Read the code path the plan names in that clone, and the nearby path that still emits the same event or renders the same screen. Do not verify from the plan text alone. Before Gate 1, update the plan's checkout path to `<ticket-folder>/<repo>` when it still names a long-lived checkout. After Gate 1, say that in the chat and leave the file alone.
3. Check the diagnosis, the design, the files to change, what must stay as it is, and the out-of-scope list against that code and the ticket. Check that frontmatter `branch` is the exact `<TICKET>-<short-slug>` name and that the opening note repeats it. If `branch` is missing, add it. Check that frontmatter `storyPoints` matches Jira when Jira has points, and that the shareable opening note’s `N SP` matches. If either is missing, add it before Gate 1. Check that `## Testing` has a local part and a shared-environment part, that `## Post-production` names a proof, and that `## Rollback` names the undo and what it does not restore. If any of those is missing, add it. Check that `## Sources` lists the procedure pages for this kind of change. If it does not, search and edit the plan. Check `## Review` against `CODEOWNERS` and the owner's procedure page. A team name alone is not enough: the plan states the policies that page requires and which section satisfies each one. If that is missing, add it. When the Storybook rule applies, `## Testing` and `## Code changes` name the story. If they do not, add it. A sibling caller that fires the same event without the new property belongs in the plan, including the instruction to leave it unchanged. The shared-environment check names a real request and a proof, not "test in staging."
4. If the plan matches, say so and what was checked. If it does not, edit the plan file in place. State each gap in the reply. These edits are draft edits, before Gate 1. After Gate 1, do not edit the plan to close a gap. Follow "The approved plan."

**Gate 1.** Stop. Link the plan file. Ask the person to approve it or mark it up. Do not edit application code on this turn. The fresh clone may already be in the ticket folder. Branching and edits wait for an explicit yes.

## Stage 3. Implement

After Gate 1 is approved, and only inside the fresh clone in the ticket folder:

1. Create and check out the branch named in the plan's `branch` frontmatter, from the default branch the clone checked out. For booking that base is `main`. For `calendly/calendly` it is `master`. Use the branch `git clone` put you on as the base. Do not invent a different name. If `branch` is missing, stop and ask the person to add it. Do not edit the approved plan to fill it in.
2. Implement only the approved plan, including any `## Updates` records. If the code disagrees with the plan, stop and say what differs. Do not edit the plan file. "The approved plan" is the rule. Do not widen the change to "also fix" a neighboring bug. Follow the owner policies `## Review` names. Do not invent a review the plan does not name. A file added during implementation can make `CODEOWNERS` name an outside team anyway. That is a difference from the plan. Stop at Gate 2 and prepare the request. Do not drop that team because `## Review` says none.
3. Update or add the tests the local part of `## Testing` names.
4. Add or update the Storybook story `## Testing` names. When `## Testing` says no story applies, do not add one.
5. Leave unrelated dirty files alone. Do not commit secrets.

## Stage 4. Verify

Run the local part of `## Testing`.

For a web UI, layout, styling, routing, client state, or rendered-data change, exercise the affected flow before Gate 2, on the running app, the way a user would. A single screenshot of the screen is not verification. Check the flag-on path and the flag-off path when both exist, and the empty or error state when the change can reach it. Do not wait for the pull request to do this. The shared-environment steps in `## Testing` are the engineer's check after the pull request exists. `## Post-production` runs after merge, publish, or deploy. The agent does not claim either one ran early. The agent still runs the local flow first, and says so at Gate 2.

Chromatic on the pull request snapshots the Storybook story. Say at Gate 2 whether that story was opened locally. Do not treat the Cypress run as that snapshot.

When that flow needs the monolith, clone `calendly/calendly` into the ticket folder and start it with the booking app. Do not skip the page because the monolith was not already running.

If a dependency install or a browser session is blocked, say what ran and what did not. Do not claim the page was verified.

When a check fails, decide whether it is this change. A failure in code this ticket does not touch gets reported at Gate 2. It is not folded into the branch unless the person asks. A pre-commit hook failure in a file this change already edits is part of this change: fix it in the same diff so the hook can pass. Do not ask whether to include it.

**Gate 2.** Stop before `git commit` and `git push`. Show:

- branch name, matching the plan's `branch` value. If the checked-out branch does not match, stop.
- `git status`
- the diff, file by file, in sentences (what changed and why)
- local checks that passed, and any shared-environment or post-production checks that did not run yet
- anything in the diff that is outside the approved plan
- for each changed file, the last `CODEOWNERS` match. When that owner is outside the squad and `## Review` does not name them, include the request prepared from their procedure page. Do not ask for a commit until that request is in the chat

On this same turn, open every changed file in the current IDE window so the person can review the diff visually. That is every modified and untracked file that belongs to this ticket. Use `cursor --reuse-window` so the files open as tabs in the window already on this workspace. Do not use `--new-window` or `--add`. Those start another session and do not show the review here. Pass each path with `--goto` at the first changed line when that line is known. If `cursor` is not on `PATH`, use the Cursor app binary (`/Applications/Cursor.app/Contents/Resources/app/bin/cursor` on macOS). The chat summary still lists the files. Opening them does not replace that summary, and it is not approval to commit.

Ask the person to approve the diff for commit and push. Do not commit on this turn.

## Stage 5. Commit and push

After Gate 2 is approved, and only then:

1. Stage the files that belong to this ticket.
2. Commit with a message in the repo's style. For booking, that is `[SCH-1234] One sentence about why.`
3. Do not update git config. Do not skip hooks. Do not force-push. Do not amend unless the person asks and the amend rules allow it.
4. Push the branch with `-u` if it has no upstream. Keep the create-pull-request URL GitHub prints. It looks like `https://github.com/<owner>/<repo>/pull/new/<branch>`.

If the commit fails because a hook changed files, fix that and make a new commit. Do not amend a failed commit.

## Stage 6. Pull request content

Do not create the pull request.

Reply with a markdown link first, then a title and a body the person can paste. The link is the create-pull-request URL from the push, `https://github.com/<owner>/<repo>/pull/new/<branch>`. Use the owner and repo from `git remote` and the branch that was just pushed. That page opens the branch's commits on GitHub and shows the form for a new pull request. Label the link "Create the pull request". For `calendly/booking`, fill the repo template. Leave no placeholders.

Title: `[SCH-1234] Short why`

```markdown
## Changes
* What changed and why, in a sentence or two a reviewer can check.

## Story
https://calendlyapp.atlassian.net/browse/SCH-1234

## QA
- [ ] The flag-on or primary path, with the visible proof that this is the path under test.
- [ ] The unchanged path (phone, flag off, or the case the plan said not to touch).

## Quick Reference
- [Delivery Process](https://calendlyapp.atlassian.net/wiki/spaces/EN/pages/385809/Delivery+Process)
- [Translations](https://calendlyapp.atlassian.net/wiki/spaces/EN/pages/2180776092/Crowdin)
```

Use the ticket's real key. QA steps come from the shared-environment part of `## Testing`, not from the template's "Be sure to test one".

Before writing the pull request text, compare every file in the branch diff with `CODEOWNERS`. The last matching pattern wins. When that check finds an outside team `## Review` does not name, prepare the request as in External review. Do not write that no outside team is required.

The body includes what `## Review` says the pull request must contain. When `## Review` names a team, or the diff check does, add two things after the body. The GitHub team to request. A Slack message the person can paste when the procedure page names a channel, using the facts that page and `## Review` already list. When the diff check and `## Review` both say none, say that no outside team is required and name the `CODEOWNERS` paths that were checked.

The agent does not request the team, does not post the message, and does not wait for the review.

**Gate 3.** Stop. The person clicks the link, pastes the title and body, requests the team, and posts the message when there is one. The agent does not run `gh pr create`, does not request reviewers, and does not post to Slack. Do not write `pullRequest` on this turn. The pull request does not exist yet.

## Record the pull request

After the person has created the pull request, write its URL into the plan. This is the last edit of this run. Give the after-release reminder in the same turn, after the write.

In the ticket clone, for the branch named in `branch`, run `gh pr view --json url -q .url`. The URL looks like `https://github.com/<owner>/<repo>/pull/<number>`. When the person has pasted that URL, write the pasted URL.

Add frontmatter `pullRequest` with that URL. Add `Pull request: <url>` to the opening note, on its own line beside `Branch:`. Change nothing else. Do not rewrite an approved section. Do not add `## Updates` for this write.

Do not write the create-pull-request link (`/pull/new/<branch>`). If no pull request exists yet, stop and ask the person to create it from the Gate 3 link. Do not invent a number.

## Done

The run is done when the branch is on the remote, the create-pull-request link, title, body, and any outside review request are in the chat, and the plan's `pullRequest` field is the pull request URL. Waiting on that review, review comments, CI on unrelated specs, and follow-up fixes are a new request.

A later commit on the same branch runs the `CODEOWNERS` check again before it is pushed. Compare the whole branch diff, not only the new commit. A new outside owner is named in the chat, with the request prepared from that team's procedure page, before the person is told the pull request is ready.

The ticket folder stays until release. The agent does not delete it unless the person asks. Deleting it before release leaves the remote branch in place and removes the folder the next skill reads.

## Implementation record handoff

`implementation_record_skill.md` builds a short `implementation-record.md` from the plan. This skill should leave every **header and timeline field** the record needs in the plan file. It does not write the record itself.

| Implementation record field | In the plan from this skill? | Where |
| --- | --- | --- |
| Title (`# KEY —` + H1) | Yes | Plan title (H1) |
| Ticket URL | Yes | Shareable note and `## Sources` |
| Story points | Yes | Frontmatter `storyPoints` and `N SP` in the shareable note (Stage 2) |
| Repo | Yes | `Repo:` line in the opening note |
| Branch | Yes | Frontmatter `branch` and `Branch:` line (Stage 2) |
| Pull request | Yes | Frontmatter `pullRequest` and `Pull request:` line ("Record the pull request") |
| Started | No | Filled at archive from GitHub (and optionally Jira In Dev) |
| Pull request opened | No | Filled at archive from GitHub `createdAt` |
| Released | No | Filled at archive from GitHub `mergedAt` (default shipped date) |
| What changed | No | Written at archive from `## Goal` (English summary, not stored separately) |
| Work bullets | No | Written at archive from `## Code changes` / tests (English summary) |

Do not add timeline frontmatter in this skill unless the person explicitly asks. Issue type (Bug, Task, Story) is not part of the implementation record today.

## After release

This run ends once `pullRequest` is in the plan. Release is later. After the ticket has been released, the person moves the ticket folder into `/Users/somo.banerjee/Documents/projects/Done` and asks the agent to run `implementation_record_skill.md`. The item in `Done` is that folder. A plan file sitting directly in `Done` is not a ticket. A repo checkout sitting directly in `Done` is not a ticket. The record skill reads `<ticket>_implementation_plan.plan.md` inside the folder, including `branch`, `storyPoints`, and `pullRequest`. It copies that plan into the archive and may add timeline dates to the archived copy only. It does not copy the cloned repo that may sit beside the plan. It is a separate run. This skill does not copy the folder or write the log.
