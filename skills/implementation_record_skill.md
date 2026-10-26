# Implementation record

Archive a released ticket from `/Users/somo.banerjee/Documents/projects/Done` into the personal repo, and keep a copy of the skill files there.

The person asks for this after moving a ticket folder into `Done`. Example: "Run `implementation_record_skill.md`." A run with no new ticket folder still refreshes the skill copies.

This skill reads the implementation plan inside the ticket folder. It does not open the cloned repo beside the plan. Story, code, proof, and people come from the plan only. It does not invent a branch or a pull request the plan does not contain. Timeline dates come from the plan first. Call GitHub or Jira only for a date the plan frontmatter still lacks, using **Timeline lookups**.

## Where things are

The inbox is `/Users/somo.banerjee/Documents/projects/Done`.

A ticket is a directory whose name is a ticket key, such as `SCH-4247`. A plan file sitting directly in `Done` is not a ticket. A repo checkout sitting directly in `Done` is not a ticket.

The archive is `/Users/somo.banerjee/Documents/projects/skills/somo-implementation-records`. The remote is `somocalendly/somo-implementation-records` on GitHub. Work on `main` only. Commit and push to `origin/main`. Do not create branches. Do not open a pull request.

The skills to copy live in `/Users/somo.banerjee/Documents/projects/skills`, and only as `*_skill.md` files in that directory. Do not copy plans, `DONE`, or `2026 bug crush`.

`index.md` at the archive root is the list of tickets already archived. Read it at the start of every run. A ticket is processed when its key appears in an index line, in the link label (`[SCH-4247]`). Use the index for that check. Do not walk `tickets/` to see whether a record file exists.

When the index lists a ticket but `tickets/<TICKET>/implementation-record.md` is missing, treat the ticket as not processed. Record it and fix the index on that run.

## Which tickets

Record each ticket folder in `Done` that contains `<ticket>_implementation_plan.plan.md` and whose key is not in `index.md`.

A ticket that is already in the index has been processed. Leave its plan copy and its record as they are. Point to the index line and the linked record path.

When the person names a ticket that is already in the index, stop on that ticket. Say it is already recorded, quote the index line, and give the record path from that link. Ask if they are sure they want it written again. Do not copy the plan again and do not rewrite the record on that turn. A general "run the skill," silence, or a message about another ticket is not a yes. Rewrite that ticket only after an explicit yes for it. Leave every other ticket alone.

Skip a folder and say why:

- The folder name is not a ticket key.
- The folder contains no implementation plan. Do not read the clone to fill the gap. `SCH-4682` is such a folder.
- The folder contains more than one implementation plan.
- The plan's ticket key and the folder name disagree. Name both. Do not pick one.

## What to copy

Copy `<ticket>_implementation_plan.plan.md` into `tickets/<TICKET>/` into the archive. Delivery metadata in frontmatter and the opening note may be added or updated after copy, as in **Timeline lookups**. Do not change approved design sections.

Do not copy a cloned repo that sits beside the plan. `SCH-4248` contains `booking` and `js-packages`. `SCH-4956` contains `calendly`. `SCH-5039` contains `mf-publisher`. Those checkouts stay in `Done`.

## The record

Write `tickets/<TICKET>/implementation-record.md` from the plan. This file is the short answer to what the ticket was and what work shipped. A reader should not need the implementation plan or a diff.

Use this shape:

```markdown
# SCH-4247 — Align the date label with the time slots

- Ticket: https://calendlyapp.atlassian.net/browse/SCH-4247
- Story points: 2
- Repo: `calendly/booking`
- Branch: SCH-4247-date-label-alignment
- Pull request: https://github.com/calendly/booking/pull/1

## Timeline

- Started: 2026-10-20
- Pull request opened: 2026-10-20
- Released: 2026-10-22

## What changed

What was wrong and what users see now. Two or three sentences.

## Work

- What you shipped, in plain English. One idea per bullet. Every bullet that describes a code change ends with the file path in parentheses.
```

Every record uses this shape and this tone. Do not add sections. Do not write longer for a bigger ticket.

Fill the fields from the plan:

- The title is `# <TICKET-KEY> —` plus the plan's H1.
- The ticket link is the browse link in the plan. When the plan has none, use `https://calendlyapp.atlassian.net/browse/<folder name>`.
- **Story points** is the integer from the plan's shareable opening note (`N SP`, for example `3 SP` in `ET Scheduling · 3 SP · Ready For Dev`). Write `Story points: N` with no "SP" suffix. When frontmatter `storyPoints` is set, use that. When neither is present, write "not recorded".
- Repo is the plan's `Repo:` line. When the plan does not name a GitHub repo, write "not named" and name the file the plan gives.
- Branch is frontmatter `branch`. Pull request is frontmatter `pullRequest`. When a value is missing, write "not recorded".
- **Started**, **Pull request opened**, and **Released** are frontmatter `started`, `pullRequestOpened`, and `released`. When a field is in the plan, copy it to the record. When it is missing, derive it once using **Timeline lookups**, write it into the archived plan, then copy it to the record. Timeline lines are dates only. Do not explain how a date was found.
- **What changed** comes from `## Goal`. Say what was broken and what users or the system see now. Two or three sentences. Plain language. Do not include `## Out of scope` or a "left alone" list.
- **Work** is three to five bullets summarizing what shipped. Read `## Code changes` or `## Code change` and translate it into English. Write the way you would explain the work in a standup or a status update, not the way the plan names code.

**Work — always do this**

- Past tense. Start with what was done: "Centered…", "Added…", "Left … unchanged", "Stopped … from returning 500".
- Every bullet that describes a code or test change ends with the repo-relative file path in parentheses. Example: `(frontend/components/enter_details/index.js)`. Use the path from the plan. The English sentence comes first; the path is how you jump back into the code.
- When tests were added or updated, say so in English and end with the spec file path. Do not name matchers, mocks, or commands in the bullet.

**Work — never do this**

- Do not omit the file path on a bullet about a code or test change.
- Do not lead with the file path or open the bullet with identifiers. Class names, hooks, props, fields, and flags are not the subject of the bullet.
- Do not copy styled-component names, API keys, or CSS properties from the plan.
- Do not reproduce the plan's numbered steps or paste code.

**Work — pattern to match** (from SCH-4257):

```markdown
## Work

- Added horizontal auto margins on the Enter details form wrapper for desktop redesign only (`frontend/components/enter_details/index.js`).
- Left phone centering, flag-off desktop layout, and shared details form styling used by other flows unchanged (`frontend/components/details_form.js`).
- Added tests for desktop redesign centering, flag-off desktop, and phone heading centering (`frontend/components/enter_details/__tests__/enter_details_with_instant.test.js`).
```

The middle bullet is optional. Use it only when the plan's out-of-scope list names something a reader might think you touched. Keep it one short sentence.

Do not add parent, squad, priority, status, proof, rollback, reviewers, or follow-up sections. Story points stay in the header bullet list only.

## Timeline lookups

Keep external calls small. One ticket should need at most one GitHub pull request read and at most one Jira issue read, and only while timeline frontmatter is still empty.

When `pullRequest` is set and any of `started`, `pullRequestOpened`, or `released` is missing, make **one** call:

```bash
gh pr view <url-or-number> --json createdAt,mergedAt,commits
```

From that response:

- **Pull request opened** is `createdAt` when `pullRequestOpened` is missing.
- **Released** is `mergedAt` when `released` is missing and the pull request is merged. That is the default shipped date. The ideal shipped time is when the repo’s production or release **workflow run finished successfully** for the merge commit. Do not list workflow runs unless the person asks to set Released from the workflow. When they ask, use one targeted lookup, then write `released` from that run’s completion time.
- **Started** is the author date of the first commit in `commits` when `started` is missing.

When `started` is still missing and the person cares about **In Dev**, make **one** Jira read for the ticket key. **In Dev** is the earliest changelog entry whose status change sets `toString` to `In Dev`. **Started** is the earlier of branch first commit and In Dev. **Do not open Jira** when `started` is already in the plan or when first commit from the pull request read is enough for this ticket.

When `pullRequest` is missing, write "not recorded" for timeline fields that need it. Do not search GitHub.

After deriving a date, add `started`, `pullRequestOpened`, and `released` to frontmatter and the opening note when each was missing. That write is delivery metadata, not a change to approved design sections.

## The index

`index.md` is the registry and the scan list. One line per archived ticket, newest release first. The line is the key linked to that ticket's `implementation-record.md`, the title, the released date, and the pull request. When the pull request is not recorded, say so. Add a line when a ticket is recorded. Do not duplicate a key that is already there unless an explicit yes asked for a rewrite.

```markdown
- [SCH-4247](tickets/SCH-4247/implementation-record.md) — Align the date label with the time slots — released 2026-10-12 — pull request not recorded
```

A rewrite that follows an explicit yes replaces that ticket's index line and re-sorts the list. Leave the other lines as they are.

## Skill copies

Copy each `*_skill.md` from the skills project root into `skills/` in the archive. Replace the copy when the source is newer. Do not copy any other file.

## Commit and push

After the records and the skill copies are written, commit them on `main` and push to `origin`.

Stage only the files this run changed. Use this commit message every time a ticket is archived:

```text
Add implementation record for SCH-1234.
```

Use the ticket key from the folder name. When one run archives more than one ticket, list each key in the same sentence:

```text
Add implementation record for SCH-1234, SCH-5678.
```

When the run only refreshed skill copies and did not add or rewrite a ticket record, use:

```text
Update skills in implementation record archive.
```

Do not update git config. Do not skip hooks. Do not force-push. Do not open a pull request.

If `main` is not the checked-out branch, check out `main` before committing. If the archive has changes this run did not make, stop and name them. Do not commit those.

## In the chat

Name each ticket recorded, the released date, and the record path. Name each ticket already recorded and left as it is. When a named ticket was already recorded, ask if they are sure they want it written again. Name each folder skipped, and why. Say that `main` was pushed, or that the run stopped before the commit.
