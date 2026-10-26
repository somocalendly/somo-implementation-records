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
- The folder contains no implementation plan **and** no matching plan exists under **Plan fallback** below.
- The folder contains more than one implementation plan.
- The plan's ticket key and the folder name disagree. Name both. Do not pick one.

## What to copy

Copy `<ticket>_implementation_plan.plan.md` into `tickets/<TICKET>/` into the archive. Delivery metadata in frontmatter and the opening note may be added or updated after copy, as in **Timeline lookups**. Do not change approved design sections.

Do not copy a cloned repo that sits beside the plan. `SCH-4248` contains `booking` and `js-packages`. `SCH-4956` contains `calendly`. `SCH-5039` contains `mf-publisher`. Those checkouts stay in `Done`.

## Plan fallback

When a ticket folder in `Done` has no `<ticket>_implementation_plan.plan.md`, look once in these directories only (lowercase ticket key in the filename):

- `/Users/somo.banerjee/Documents/projects/skills/2026 bug crush/`
- `/Users/somo.banerjee/Documents/projects/skills/breached - 10-12-26/`

Example: `sch_4682_implementation_plan.plan.md` for folder `SCH-4682`. When a file matches, copy it into the ticket folder in `Done`, then continue the run. Do not copy from `DONE/`, loose files at the skills root, or other folders. When nothing matches, skip the folder and say why.

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

Every record uses this shape and this tone. Do not add sections. Do not write longer for a bigger ticket unless **Multi-repo delivery** requires more **Work** bullets to list every shipped change.

Fill the fields from the plan:

- The title is `# <TICKET-KEY> —` plus the plan's H1.
- The ticket link is the browse link in the plan. When the plan has none, use `https://calendlyapp.atlassian.net/browse/<folder name>`.
- **Story points** is frontmatter `storyPoints` when set (written by `ticket_to_pr_skill.md` from Jira). Write `Story points: N` with no "SP" suffix. When frontmatter is missing, use the integer from the shareable opening note (`N SP`). When neither is present, write "not recorded".
- Repo is the plan's `Repo:` line. When the plan does not name a GitHub repo, write "not named" and name the file the plan gives.
- Branch is frontmatter `branch`. Pull request is frontmatter `pullRequest`. When a value is missing, write "not recorded".
- When the ticket shipped through **more than one repo** (package change plus consumer pin, version bump, or any second merged pull request for the same ticket key), repeat `Repo`, `Branch`, and `Pull request` once per repo, in delivery order (library or package first, then consumers). Read extra entries from frontmatter `delivery` when present. When `delivery` is missing, discover them: repo checkouts sitting in the ticket folder in `Done` name likely repos; run one merged-PR search per repo (`gh pr list --repo <owner>/<repo> --state merged --search "<TICKET-KEY>"`). Include every merged pull request that belongs to this ticket. Do not treat out-of-scope language in the plan as permission to omit a pull request that actually merged for this key.
- **Started**, **Pull request opened**, and **Released** are frontmatter `started`, `pullRequestOpened`, and `released`. When a field is in the plan, copy it to the record. When it is missing, derive it once using **Timeline lookups**, write it into the archived plan, then copy it to the record. Timeline lines are dates only. Do not explain how a date was found.
- **What changed** comes from `## Goal`. Say what was broken and what users or the system see now. Two or three sentences. Plain language. Do not include `## Out of scope` or a "left alone" list.
- **Work** summarizes **everything that shipped** for the ticket. Read `## Code changes` or `## Code change` in the plan and add bullets for every other merged pull request you listed (pins, lockfiles, patches, publish bumps). Use three to five bullets when one repo changed; use as many bullets as needed when several repos did, still one idea per bullet. Write the way you would explain the work in a standup or a status update, not the way the plan names code. File paths stay repo-relative to the repo that owns that bullet.

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

## Multi-repo delivery

One Jira ticket can merge in several repositories. The record must capture **all** of that work, not only the repo named in `Repo:`.

Before writing the record, build the full delivery list (primary `branch` / `pullRequest` plus any other merged pull requests for this ticket key). Write every repo’s `Repo`, `Branch`, and `Pull request` lines in the header. **What changed** should describe the outcome invitees or hosts see, including consumer pins when those were required to ship the plan’s change.

**Timeline** when multiple pull requests merged:

- **Started** — earliest first-commit date among those pull requests.
- **Pull request opened** — earliest `createdAt` among them.
- **Released** — latest `mergedAt` among them (the date the last required piece merged). When only one pull request exists, use that merge date.

Store the full list in archived plan frontmatter as `delivery`, a YAML list of objects with `repo`, `branch`, and `pullRequest`. One repo is a one-item list. Keep legacy `branch` and `pullRequest` as the first entry when the plan already had them. When archiving, write or backfill `delivery` from the record’s header; do not add extra opening-note prose for additional repos—the implementation record is where humans read multi-repo delivery.

## Timeline lookups

Keep external calls small. One ticket should need at most **one GitHub pull request read per delivery pull request** and at most one Jira issue read, and only while timeline frontmatter is still empty.

When `pullRequest` is set and any of `started`, `pullRequestOpened`, or `released` is missing, make **one** call per delivery pull request that still lacks dates:

```bash
gh pr view <url-or-number> --json createdAt,mergedAt,commits
```

From that response:

- **Pull request opened** is `createdAt` when `pullRequestOpened` is missing.
- **Released** is `mergedAt` when `released` is missing and the pull request is merged. With **multiple delivery pull requests**, **Released** is the **latest** merge date among them once each has been read. The ideal shipped time is when the repo’s production or release **workflow run finished successfully** for the merge commit. Do not list workflow runs unless the person asks to set Released from the workflow. When they ask, use one targeted lookup, then write `released` from that run’s completion time.
- **Started** is the author date of the first commit in `commits` when `started` is missing. With multiple delivery pull requests, **Started** is the **earliest** first-commit date among them.
- **Pull request opened** with multiple delivery pull requests is the **earliest** `createdAt` among them.

When `started` is still missing and the person cares about **In Dev**, make **one** Jira read for the ticket key. **In Dev** is the earliest changelog entry whose status change sets `toString` to `In Dev`. **Started** is the earlier of branch first commit and In Dev. **Do not open Jira** when `started` is already in the plan or when first commit from the pull request read is enough for this ticket.

When `pullRequest` is missing, write "not recorded" for timeline fields that need it. Do not search GitHub.

After deriving a date, add `started`, `pullRequestOpened`, and `released` to frontmatter and the opening note when each was missing. That write is delivery metadata, not a change to approved design sections.

## The index

`index.md` is the registry and the scan list. One line per archived ticket, newest release first. The line is the key linked to that ticket's `implementation-record.md`, the title, the released date, and the pull request URL or URLs. When several repos shipped, list every pull request URL on that line, comma-separated, in the same order as the record header. When the pull request is not recorded, say so. Add a line when a ticket is recorded. Do not duplicate a key that is already there unless an explicit yes asked for a rewrite.

```markdown
- [SCH-4247](tickets/SCH-4247/implementation-record.md) — Align the date label with the time slots — released 2026-10-12 — pull request not recorded
```

A rewrite that follows an explicit yes replaces that ticket's index line and re-sorts the list. Leave the other lines as they are.

## Skill copies

Copy each `*_skill.md` from the skills project root into `skills/` in the archive. Replace the copy when the source is newer. Do not copy any other file.

## Commit and push

**Review gate (interactive runs).** After the record, archived plan metadata, and `index.md` are ready, stop before `git commit`. Show the full `implementation-record.md` in chat and say the run stopped for review. Do not commit or push on the same turn unless the person explicitly asks (for example "commit and push"). Preparing index and plan copies locally is fine; staging is optional until they approve.

**Scheduled automation runs** skip the review gate. See **Weekly Done inbox automation** at the end of this file.

When they ask to commit, or when a scheduled automation run finishes successfully, commit on `main` and push to `origin`.

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

When one commit on `main` includes both a new or rewritten ticket record and updated skill copies, use two sentences in one message:

```text
Add implementation record for SCH-1234. Update skills in implementation record archive.
```

List every ticket key in the first sentence when more than one ticket is archived.

Do not update git config. Do not skip hooks. Do not force-push. Do not open a pull request.

If `main` is not the checked-out branch, check out `main` before committing. If the archive has changes this run did not make, stop and name them. Do not commit those.

## In the chat

Name each ticket recorded, the released date, and the record path. Name each ticket already recorded and left as it is. When a named ticket was already recorded, ask if they are sure they want it written again. Name each folder skipped, and why. When the review gate applies, say the run stopped before commit and wait for explicit approval. After a commit, say that `main` was pushed.

## Weekly Done inbox automation

Use this when a **Cursor Automation** (or the person) says to run the weekly archive. It is the same skill with these overrides:

**Where to run.** The automation must execute on the Mac that holds the Done inbox and skills paths (`/Users/somo.banerjee/Documents/projects/Done` and `/Users/somo.banerjee/Documents/projects/skills`). A cloud-only checkout of `somocalendly/somo-implementation-records` alone is not enough.

**Schedule.** Saturday at 8:00 PM in the person’s local timezone (confirm in the Automations cron picker).

**Each run:**

1. Read `index.md` in `/Users/somo.banerjee/Documents/projects/skills/somo-implementation-records`.
2. For every ticket folder in `/Users/somo.banerjee/Documents/projects/Done`, apply **Which tickets**, **Plan fallback**, **Multi-repo delivery**, and the rest of this skill.
3. Process **all** new tickets in one run (not one chat turn per ticket). Refresh skill copies into `skills/` when the source files changed.
4. **Do not stop for review.** When at least one new ticket was archived, commit on `main` and push to `origin` with the normal commit messages. When nothing was new, push only if skill copies changed (`Update skills in implementation record archive.`). When there was nothing to do, do not commit.
5. **Success notification.** Post a short summary: tickets archived (keys + released dates), tickets skipped (and why), or “no new Done tickets.” Use the automation’s Slack (or email) action.
6. **Errors.** If a ticket cannot be archived (missing PR, git push failed, ambiguous plan, index conflict, unexpected repo state), **stop before commit**, do not push a partial archive, and send a failure notification that names the ticket, the error, and asks the person to fix or run the skill interactively. Include what was prepared locally if anything was written.

The automation prompt lives in `somo-implementation-records/automations/weekly-done-inbox.md` (copy into the Cursor Automation instructions).
