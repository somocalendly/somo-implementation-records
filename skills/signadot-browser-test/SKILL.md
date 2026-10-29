---
name: signadot-browser-test
description: Execute a written test plan against a Signadot staging sandbox using the tester's logged-in Chrome browser, then post the results as a single comment on the Jira ticket. Use this whenever the user gives a Jira ticket key, a test plan (usually an attached .md file), and a sandbox or staging URL and asks to test, verify, QA, run the checks, or validate a ticket or PR on Signadot or staging, even if they do not say "skill" or "test plan". Also use it when the user says "run the test plan", "test this on the sandbox", or "verify the fix on staging".
---

# Signadot browser test

## For the tester (read this if you are a person, not Claude)

You do not need to write anything. The test plan already exists on the ticket, generated when the PR was created. To run it:

1. Put the sandbox label on the PR and wait for the sandbox to come up.
2. In Chrome, turn on Signadot routing for that sandbox and log in to staging as the user the plan's setup section names. Do whatever else the setup section asks (for example, confirming the account has no connected calendar).
3. Download the test plan .md from the ticket.
4. In Claude, attach the .md and say something like: "Test ABC-123 on Signadot. Starting URL: <the page where checks begin>."

Claude runs the checks in your browser, asks you if it gets stuck, then shows you the comment it wants to post and waits for your OK before posting it to the ticket. Everything below this line is instructions for Claude. The file in `references/` is a spec for whoever writes test plans; you can ignore it.

---

Run a test plan, written by a human or by another tool, against a feature deployed to a Signadot sandbox, using Chrome. Record exactly what you observe, and post one comment on the Jira ticket. The tester has already done the setup; your job is to execute faithfully and to stop and ask when you cannot.

## Why this skill is strict about evidence

The comment you post becomes the record that a change was tested before release. A reviewer who was not there must be able to read it and trust it. That is why you quote page text verbatim instead of paraphrasing, record what you saw rather than what you expected, and never fill a gap with a guess. A "Not run" or a question to the tester is always better than a confident result you did not observe.

## Inputs

You need three things. If any is missing, ask for it before doing anything else.

1. **Ticket key**, such as `ABC-123`. Used to post the comment.
2. **Test plan**, normally a Markdown file attached to the chat. If no file is attached, look for a Jira comment or description section on the ticket headed "Testing plan" or similar. If you cannot find one, ask; do not write your own.
3. **Starting URL**, the exact page where the checks begin, already routed to the sandbox. It is never the bare staging root.

Everything else you need (which user is logged in, which sandbox label to expect, what account state is required) comes from the test plan's setup section. Trust that the tester did the setup. Do not log in, type credentials, switch accounts, connect calendars, change settings, or guess URLs or slugs.

## Before any check

Read the whole test plan first. Note its stop conditions, its comment template, its result rule, and any check it marks optional or excluded.

Then open a browser tab for this task and load the starting URL. Confirm two things:

- **Sandbox banner.** The plan names an expected sandbox label. Look for a visible sandbox banner on the page and read its text. If there is no banner, or the text does not include the expected label, stop and ask the tester. Do not run checks and do not post a comment. A missing banner usually means your tab is not routed to the sandbox (Signadot routing can be per-window or per-tab), and only the tester can fix that.
- **Plan-defined stop conditions.** Plans typically include a login wall, a wrong host or wrong page, or a missing precondition. Do exactly what the plan says for each. Some say "post the comment with every check Not run and Result Blocked"; others say "stop and ask". Follow the plan's instruction for that condition, not a general rule.

Signadot routing is applied by the tester outside this skill. You never need to read Signadot documentation or configure anything; the banner is your only check that routing worked.

## Running the checks

Run checks in the order written. For each one, record a status of Pass, Fail, or Not run, plus the evidence the check names.

Some rules that apply to every check:

- **Quote, don't paraphrase.** When a check asks for text, copy it from the page character for character, including capitalization, punctuation, and apostrophe style. Plans often specify curly apostrophes (U+2019); compare against exactly what the page renders, and if the page shows a straight quote where a curly one was expected, that is a Fail with both strings quoted as evidence.
- **Pass conditions are literal.** If a check says a line must read exactly "X", then "X:" or "x" does not pass. Report the mismatch.
- **Optional checks never change the result.** If the plan marks a check optional or excluded, attempt it if you can, record Not run if you cannot, and never ask the tester about it. The plan's result rule tells you which checks count.
- **Do not improvise.** If a required check cannot be done as written, stop and ask. Name the check, say what blocked you, and propose one alternative. Wait for the answer. Do not skip the check silently, and do not mark it Not run unless the plan tells you to.
- **One stop at a time.** When you stop to ask, ask one clear question. Once answered, resume from that check.

## Browser mechanics

These are the tool-level details you will need. Load the Chrome tools in one ToolSearch call at the start: `tabs_context_mcp`, `tabs_create_mcp`, `navigate`, `computer`, `find`, `read_page`, `get_page_text`, `javascript_tool`, `read_network_requests`, `resize_window`, `tabs_close_mcp`.

**Reading text.** Use `find` or `read_page` to locate the element a check names, then quote its text. Screenshots are for layout questions (is the panel on the right, is the icon in the header row); text comes from the DOM, not from reading a screenshot, because the DOM is exact and a screenshot can misread a character.

**Layout questions.** For "is X to the right of Y" or "is the icon in the same row", take a screenshot and also compare element bounding boxes via `javascript_tool` (`getBoundingClientRect`). Report both when it is close.

**Window width.** Use `resize_window` for viewport requirements. Desktop Chrome will not go below roughly 500px wide, so a plan asking for a phone width may be unreachable. If the plan marks that check optional, record Not run and move on. If it is required, stop and ask.

**Network responses.** `read_network_requests` lists requests, URLs, and status codes but may not expose response bodies. When a check needs a body, first read the exact request URL (including its query string) from the network log, then fetch that same URL from the page context with `javascript_tool` and `credentials: 'include'`. Only do this when the plan says the fetch substitute is acceptable; some endpoints behave differently with an empty query string, so always reuse the one the page used.

**Keyboard checks.** For "Tab until X is focused, then press Enter", press Tab via `computer` one key at a time and check `document.activeElement` with `javascript_tool` after each press until it matches the target. Cap the loop at a reasonable count (around 40) and if the element is never focused, that is a Fail with evidence of where focus ended up. Do not call `.focus()` from script; the point of the check is that a keyboard user can reach the element.

**Popups and modals.** After a click that should open a popup, wait briefly, then locate the popup's title via `find`. Quote it. For "button X closes it", click the button and confirm the popup is gone from the DOM, not merely hidden behind something.

**Checking page state the plan depends on.** Plans sometimes make a pass condition depend on account state (for example, no connected calendar). If the plan says what to do when the state is wrong (usually "stop and ask"), follow it. A wrong state means the run cannot prove anything about the fix, which is why plans prefer a question over a Fail: a Fail on the ticket would wrongly blame the code.

**Screenshots as evidence.** You cannot attach files to Jira through the connector. Describe what was on screen in the comment instead, and quote the text the plan requires. If the tester wants images, offer to save screenshots to the outputs folder for them to attach by hand.

**Close your tabs** when the run is done, unless the tester asked to see the final state.

## Posting the comment

Nothing goes to Jira without the tester's approval. This skill is new, and the tester needs to see what it is about to write before it becomes part of the ticket's record.

Draft the comment using the plan's template, filled in with your recorded statuses and evidence. Compute the overall result with the plan's rule; do not invent your own. Then show the tester, in chat, the complete comment exactly as it will be posted, along with the ticket key it will go on, and ask for approval. Wait. Post only when the tester says yes. If they ask for a change, make it, show the full revised comment again, and ask again. If they say no, do not post, and ask what they want done instead.

This applies to every path, including the stop conditions that say "post the comment with every check Not run and Result Blocked": draft that comment, show it, and get approval before posting it.

To post, load the Jira tool `addCommentToJiraIssue` (and `getJiraIssue` if you need to read a plan from the ticket). Post exactly one comment. Do not change the ticket's status, assignee, or any other field.

After posting, confirm to the tester that it went up and mention anything they should know (a check you were unsure about, a width you could not reach, a question you had to ask).

## When to ask versus when to proceed

Ask when: the banner is missing or wrong, a required check cannot be done as written, an input is missing, the plan contradicts itself, or a plan-defined precondition fails and the plan says to ask.

Proceed without asking when: an optional check cannot be done, the plan gives an explicit fallback, or the outcome is simply Pass or Fail by the plan's own condition.

Never ask the tester to do something the plan forbids for you (logging in, toggling flags, switching accounts) as a way of getting a check to pass. If a check needs that, the plan should have marked it optional; report it and let the tester decide.

## What a good test plan looks like

If the plan you are given is missing a setup section, verbatim expected strings, named evidence per check, a comment template, or a result rule, you can still run it, but expect to ask more questions. `references/test-plan-format.md` describes the shape this skill runs best against; point plan authors to it when a plan is hard to execute.
