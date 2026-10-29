# Implementation records

Private record of released tickets, and a backup of the skills that produce them.

- `tickets/<TICKET>/` holds the implementation plan and that ticket's `implementation-record.md`.
- `index.md` lists one line per ticket, newest release first.
- `skills/` holds a copy of each `*_skill.md` from the local skills project. The Implementation Record Skill refreshes this copy and pushes.
- `skills/signadot-browser-test/` is a Claude skill (not a Cursor skill) that runs a ticket's test plan against a Signadot sandbox in Chrome and drafts the Jira comment. To use it, download `signadot-browser-test.skill` and save it in Claude under Customize > Skills. `SKILL.md` explains the inputs and how a tester invokes it; `references/test-plan-format.md` is the plan format the Ticket to PR Skill generates.
