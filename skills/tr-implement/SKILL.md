---
name: tr-implement
description: "Implement a piece of work based on a spec or set of tickets: build it test-first (asking only if something genuinely blocks it), write a runbook for any manual setup, then report back in four short sections you can actually act on."
disable-model-invocation: true
---

# Implement

Build the work described by a spec or a ticket. Three phases, in order. Don't let phase 3 sprawl.

## Phase 1: Read, then go

**Before touching any code.** Read the ticket (fetch it via `docs/agents/issue-tracker.md` if it's a reference), and read enough of the codebase to know what you're actually about to do.

Then **start building.** Do not send a summary or brief of the ticket, and do not ask for a go-ahead. Tom picked the ticket; running this skill is the go-ahead.

**The one exception: a genuine blocker.** Stop and ask only when you cannot build the ticket correctly without an answer — an ambiguity with two materially different readings, a contradiction with an ADR or `CONTEXT.md`, a dependency that isn't done, or a decision the ticket leaves open that would be expensive to undo. Then:

- Ask every blocking question in **one** message, numbered, each with your recommended answer (`➡️`) so Tom can reply "1 yes, 2 the second one".
- Ask nothing you could answer by reading the code, the docs, or the ticket's parent spec. Look first.
- If a question has a safe, reversible default, don't ask it — take the default and list it under **What I noticed** in the report.

*Why:* this used to stop for a plain-English brief and an approval on every ticket. Tom found the gate cost more than it caught, so it was removed. Don't reintroduce a summary-and-confirm step in any form.

## Phase 2: Build

Drive `/tr-tdd` where possible, at pre-agreed seams. Prefer existing seams; use the highest seam available.

**Announce each slice in exactly one line before you start it:**

```
→ <the behaviour being built, in plain words>
```

Nothing else. No narration of file edits, no explanation of your approach, no progress percentages. These markers are the only running commentary, and phase 3 summarises them.

While building:

- Run typechecking regularly
- Run the single relevant test file regularly
- Run the full test suite **once**, at the end

Use the project's domain glossary vocabulary (`CONTEXT.md`) in names and test descriptions, and respect ADRs covering the area you're touching. If your work contradicts an ADR, surface it rather than silently overriding.

Once the suite is green, run `/tr-code-review` on the work.

**Do not commit.** Leave the working tree for the user to review and commit themselves.

Keep a running note of three things as you go, because phase 3 needs them and reconstructing them afterwards produces vagueness: anything you skipped or deferred, anything that looked broken or risky in code you touched or read, and anything Tom has to do by hand outside the code (next section).

### Runbooks for anything Tom has to do by hand

Whenever the work depends on something **outside the code** that a person has to do manually, write it up as a runbook in the target repo's `docs/runbooks/`. Triggers include:

- a new environment variable, secret, API key or token (where to get it, where to put it)
- an account, app registration, OAuth client, webhook or API permission on a third-party service
- cloud or infrastructure setup: a bucket, a database, a DNS record, a firewall rule, a CI/CD secret
- installing a tool, running a one-off migration or seed, or any command that isn't part of the normal test/run path

If none of these came up, write nothing. Don't invent a runbook for routine work.

**Where and how:**

- One file per task: `docs/runbooks/<dash-case-task>.md` (e.g. `stripe-api-key.md`). If a runbook for the same thing exists, **update it** rather than writing a second one.
- Keep an index at `docs/runbooks/README.md`: one line per runbook — link plus when you need it. Create it with the first runbook.
- If the work adds env vars and the repo has (or should have) a `.env.example`, add each var there with an empty or placeholder value and a one-line comment pointing at the runbook.

**Each runbook uses this shape**, written so Tom can follow it months later with no memory of this session:

```markdown
# <What this sets up, in plain words>

**Why you need this:** <one sentence — what breaks or doesn't work without it>
**When:** <once per machine / once per environment / every deploy / when the token expires>
**Takes about:** <rough time>

## Before you start
- <accounts, access, or tools you need first>

## Steps
1. <one action per step — exact URL, exact menu path, exact command to paste>
2. <say which value to copy, and exactly where it goes: file, variable name, CI secret name>

## Check it worked
<one command to run or thing to look at, and what you should see>

## If something goes wrong
- <the likely failure> → <what to do>
```

Rules: numbered steps, one action each; exact names and commands, never "configure the service appropriately"; **never write a real secret value** into a runbook or anything committed — name where it lives and where it goes, use `<placeholder>` for the value.

*Why:* setup steps used to live only in the chat where they came up, and were gone the next time Tom set up the app on another machine or environment. The runbook is the copy that survives.

## Phase 3: Close-out report

Report back in **exactly these four sections and nothing else**:

```
## What you can do now

One sentence. The new thing a person using the app can do that they couldn't before.
No file names, no technical terms.

## How to test it

At most 3 numbered steps. Each step is either a command to copy-paste, or
"open <page>, click <thing>". After each, say what you should see if it worked.
The fastest honest path to seeing it run — not a QA script.
If you wrote or updated a runbook, step 1 is "Do the setup in docs/runbooks/<name>.md"
(the one place a file name is allowed outside a command).

## What I didn't finish

Bullets, at most 4, one plain line each: what the ticket asked for that isn't there,
and why in half a sentence. Write "Nothing — it's all there." if that's true.

## What I noticed

Bullets, at most 4, one plain line each: problems in the existing code, plus anything
/tr-code-review flagged that wasn't fixed. Say what's wrong and roughly where, in words
someone who didn't write it would follow. Write "Nothing." if clean.
```

### Write it for someone who did not read the code

These rules are the point of the report. The section headings alone are not enough — the last version of this skill said "three sections and nothing else" and still produced a wall of implementation vocabulary.

- **No jargon.** Not "seam", "interface", "abstraction", "refactor", "repository", "middleware", "hook", "wire up", "plumb through". If you have to name something, name it the way the app's users would.
- **Name a file or a function only inside a command to run.** Never in prose. (A runbook path in **How to test it** is the one exception.)
- **Nothing outside the four sections.** No preamble, no restatement of the diff, no list of files changed, no summary of your approach, no "what I learned", no architecture commentary, no next steps, no offer to commit.
- **The whole report fits on one screen.** If it doesn't, you are explaining rather than reporting. Cut it.

Be honest in every section. If the suite didn't go green, that belongs in **What I didn't finish**, with the failing output, not omitted. If you cut a corner, it goes there too. A clean-looking report that isn't true costs more than a messy one that is — and honesty is the one thing that outranks the length caps above.

Then stop. Don't offer next steps, don't start the next ticket, don't ask if they want you to commit.

## Between tickets

Each ticket is self-contained by construction, so the previous ticket's context is disposable. `/clear` between tickets and start the next one fresh.
