# Test plan format

This is the shape of test plan the `signadot-browser-test` skill executes with the fewest interruptions. Plan authors (human or agent) should produce plans in this form using `signadot_test_plan_skill.md` in this repository. The runner skill can execute looser plans, but every gap below turns into a question to the tester.

## Inputs the tester provides at invocation

- The Jira ticket key.
- The test plan as a Markdown file attached to the chat.
- The starting URL: the exact page the checks begin on, already routed to the sandbox. Not the staging root.

## Assumptions the skill makes

The tester has already: put the sandbox label on the pull request, confirmed the sandbox is running, activated Signadot routing in Chrome, and logged in to staging as the user the plan names. The skill will not log in, type credentials, switch accounts, change account state, toggle flags, or guess URLs. The one thing it verifies before running is the sandbox banner.

## Sections

### 1. Setup for the tester

A short list of what must be true before invoking the skill:

- The sandbox label to put on the PR, and the text the sandbox banner is expected to contain.
- Which staging user must be logged in, and anything about that account that affects the checks (connected calendar or not, avatar, locale, plan tier, feature flags). When a check only distinguishes fixed from unfixed code in one account state, say so and make that state required, with a URL where the tester can confirm it.
- What the starting page must be (for example, a one-on-one booking page whose host is not the logged-in user).

### 2. Stop conditions

What the skill should do, before any check, for each of: a login wall, a missing or mismatched sandbox banner, a wrong page or host, and any other precondition. For each, say either "post the comment with every check Not run and Result Blocked" or "stop and ask the tester; do not comment yet". Also state the general rule: if a required check cannot be done as written, stop and ask, naming the check, the blocker, and one alternative.

Add any global comparison rules here: verbatim string matching, which apostrophe character to expect (U+2019 versus a straight quote), and whether fetching an API endpoint from the page context is an acceptable substitute for reading a response body from the network log (if so, say to reuse the exact query string the page used).

### 3. Checks

Numbered. Each check has three parts:

- **Action**: concrete UI steps. Click what, type what, resize to what, press which keys. Include fallbacks for common dead ends (for example, "if no day in the visible month is bookable, advance one month; if still none, stop and ask").
- **Pass**: a condition observable on the page or in network traffic. Expected strings quoted exactly as the implementation renders them.
- **Evidence**: what to record. Quoted text, a field value, a position, a description of what was on screen.

Where a pass condition depends on account state, either make that state a setup requirement and say what to do if it is wrong (usually "stop and ask, do not record Fail"), or write the check to report what is observed without a pass/fail that hinges on it.

Mark any check that needs something only a human can do as **optional** and say it is excluded from the overall result. Examples: a viewport narrower than desktop Chrome allows (about 500px), toggling a feature flag, switching accounts, impersonation modes. Tell the skill not to ask about optional checks.

### 4. Comment

The exact template to post on the ticket, with a placeholder per check for status and evidence, plus the overall result. State that only one comment is posted, that ticket status is not changed, and that no file is attached. Then the result rule: which checks count, and how Pass, Fail, and Blocked are derived from them.

## Example of a well-formed check

```
### 3. Header

Action: look at the panel header. Do not click it yet.

Pass: the first line is exactly `Your schedule`. The line under it contains `@`. An info icon is in that same header row, to the right of the two lines.

Evidence: quote both lines. State whether an image is in that header row (`avatar present` or `avatar absent`). The avatar does not affect Pass or Fail.
```

## Example of a stop condition that protects the result

```
### 5. Calendars

Pass: no account email is shown under that sentence. Evidence is exactly `no account email shown`.

If any account email is shown, stop and ask the tester. This account has a connected calendar, so the run cannot prove the fix. Do not record Fail, and do not post the comment yet.
```

This pattern matters: a wrong precondition means the run is uninformative, not that the code is broken, so a question is more honest than a Fail on the ticket.
