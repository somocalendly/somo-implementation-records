# Weekly Done → implementation record archive

Copy the **Agent instructions** block below into a Cursor Automation. Schedule: **every Saturday at 8:00 PM** (local time). Enable **Post to Slack** (or email) for success and failure messages.

**Runtime:** Local agent on Somo’s Mac (paths below must exist). Check out or use `/Users/somo.banerjee/Documents/projects/skills/somo-implementation-records` on branch `main`.

**Repo for git push:** `somocalendly/somo-implementation-records`, branch `main` only.

---

## Agent instructions

You are running the **weekly scheduled** implementation record archive. Follow `/Users/somo.banerjee/Documents/projects/skills/implementation_record_skill.md` completely, including **Plan fallback**, **Multi-repo delivery**, and **Weekly Done inbox automation** (scheduled overrides: no review gate; commit and push when successful).

**Paths**

- Done inbox: `/Users/somo.banerjee/Documents/projects/Done`
- Skills source: `/Users/somo.banerjee/Documents/projects/skills/` (`*_skill.md` only)
- Archive git root: `/Users/somo.banerjee/Documents/projects/skills/somo-implementation-records`

**Run**

1. Read `index.md` at the archive root. Any ticket key already linked there is already archived — leave it alone unless the person explicitly asked to rewrite (they did not).
2. For each ticket folder in Done with a plan (in the folder or via **Plan fallback**), not in the index: write `implementation-record.md`, copy/update the archived plan, update `index.md`, and apply delivery metadata per the skill.
3. Copy refreshed `implementation_record_skill.md` and `ticket_to_pr_skill.md` into `skills/` when newer than the archive copies.
4. If one or more tickets were archived, commit with `Add implementation record for SCH-….` (comma-separate keys). If only skills changed, `Update skills in implementation record archive.` If both, use both sentences in one message.
5. Push to `origin/main`. Verify clean status.

**On success — Slack (or configured notify action)**

Send a short message, for example:

- “Weekly implementation archive done. Recorded: SCH-1234 (released …), SCH-5678 (…). Skipped: SCH-9999 (no plan). Pushed main on somo-implementation-records.”
- Or: “Weekly implementation archive: no new Done tickets. Nothing committed.”

**On failure — Slack (or configured notify action)**

Do **not** commit or push if anything failed mid-run. Send:

- “Weekly implementation archive **failed**. Ticket: … Error: … Please run `implementation_record_skill.md` interactively or fix Done/index and retry. I stopped before commit.”

If the failure is environmental (cannot read Done, git auth, push rejected), say that clearly and ask for help.

**Do not** open a pull request. **Do not** force-push. **Do not** skip git hooks.
