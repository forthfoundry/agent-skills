---
name: spexd-implementing
description: >-
  Implement work tracked in Spexd — pick up a task, read its traceability
  chain (design, acceptance criteria, requirement, feature) as the spec, check
  it is safe to start, build to it without changing it, and keep the task's
  lifecycle status in step with its branch and PR. Given a feature,
  requirement or design, build it task by task, one branch and one PR each.
  Use when asked to pick up, build, ship or find the next piece of Spexd work,
  or to turn a task into code. To write or change the spec itself, use
  spexd-authoring.
user-invocable: true
---

<!--
Section headings are an API. Spexd serves this skill one section at a time
through loadSkill, and the load map in "Start here" names sections exactly.
Renaming a heading breaks those pointers. There is deliberately no H1: an H1
would contain every section, so loading it would return the whole body.
Every cross-reference in italics must be a heading loadSkill can return.
-->

## Start here

Spexd models a strict chain: **Feature → Requirement → Acceptance Criteria →
Design → Task**. A **task** is the leaf and the only unit of implementation
work. It normally delivers one design; a task with no architecture to decide
(a copy change, a configuration tweak) hangs directly off a requirement or
feature instead. `spexd-authoring` decides *what* to build and *how*; this
skill carries out a decision already made.

**The one rule: implement the spec, don't rewrite it.** A task executes a
design; it never invents architecture. If the design turns out wrong, missing
or under-specified, **stop and surface it** — as a comment thread on the
design, and to the user — rather than quietly building something the chain
doesn't describe. You never edit the spec while implementing: once the task
starts, the chain is `LOCKED` and edits are refused.

Use this skill for "pick up TASK-12", "what's next on FEAT-6", "implement
DES-320" (still built one task at a time), or any request to build or ship
Spexd-tracked work. Writing or restructuring the spec is `spexd-authoring`.

**The loop:** *Find the work* → *Orient — open with the entity* → *Gather the
context* → *Pre-start check* → *Build* → *Reflect status*. Never skip from
finding to building: the chain above the task is what makes the code right,
and the pre-start check confirms it still holds.

**Load only what the job needs.** A name in italics is a section:
`loadSkill({ name: "spexd-implementing", section: "<heading>" })`. A section
includes its subsections.

| You are about to… | Load |
|---|---|
| Pick up a task and build it end to end | The skill `whole` — you will follow every step |
| Find what to build next | *Find the work*, *Orient — open with the entity* |
| Read a task's context | *Gather the context* |
| Decide whether a task is safe to start | *Pre-start check* |
| Write the code and its tests | *Build* |
| Move a task's status, name a branch or PR, or fix a task that didn't move | *Reflect status* |
| Build a feature, requirement or design | *Given a parent entity: work it task by task* |
| Check a tool's behaviour | *Operational notes (MCP surface)* |

## Find the work

- **Given a task reference**, `getEntity` on it. It is content-free, but it
  gives the title, status, `viewUrl` and the ordered ancestor chain — enough to
  orient, and the references *Gather the context* reads in one batch.
- **Given a branch name or PR title**, `resolveGitHubBranch` turns it into the
  entity it names in one hop. Its `outcome` is `resolved` (a task, on `task`),
  `out_of_scope` (the deepest thing named isn't a task — a branch naming only
  its feature), `ambiguous` (a tie, on `tied`) or `not_found`. A miss is an
  outcome, never an error.
- **Given a feature, requirement or design**, the unit of work is still the
  task. `listDescendants(ref, { types: ["TASK"], maxDepth: -1 })` lists every
  task beneath it; add `statuses: ["APPROVED"]` for the ones ready to build.
  For "what's left on FEAT-n", `getFeatureOutstandingItems` names what the
  feature's progress is short of — its `pendingBuild` group is the unbuilt
  work. Don't use `getOutstandingWorkForEntity` for either: it lists what waits
  on an approver or a re-submit, never an approved task ready to build. To
  build the whole parent, see *Given a parent entity: work it task by task*.
- **Asked for what's next with nothing to anchor on**, browse `listInbox`:
  `view: "approved"` with `types: ["TASK"]` lists tasks ready to build across
  the organization; page with `nextCursor`. Use `searchEntities` when you have
  text to search on instead.

## Orient — open with the entity

Whenever you're asked to pick up or look at a Spexd entity, the **first thing
you say back** — before any plan, file read or code — is what it is:

1. **A link** — its `viewUrl` as a markdown link on the reference.
2. **Its title**, verbatim.
3. **A concise summary** — two or three sentences on what it covers and what
   "done" means, in your own words.

```markdown
**[TASK-12](https://acme.spexd.com/e/TASK-12) — Implement `POST /api/rides`**

Creates the ride-request endpoint and its state machine transitions per
[DES-9](https://acme.spexd.com/e/DES-9), validating rider location and payment
method before persisting. Done when the endpoint accepts a valid request,
rejects the three failure cases in AC-4/AC-5, and the state machine is covered
by tests.
```

If the ask spans several entities — "what's outstanding on FEAT-6" — give the
same link, title and one-line summary for each, then say which you're picking
up. Do this even when you're about to start at once: it lets a person confirm
you're on the right thing before you spend effort on it.

## Gather the context

A task on its own is only a definition of done; its meaning lives above it.
Read the whole chain before writing code:

- **Design** — the parent, and **your primary spec**: architecture,
  boundaries, data model, contracts, security, configuration, deployment,
  failure modes, testing areas and trade-offs. `getDesignCoverage` gives the
  acceptance criteria it fulfils — that set, not the requirement's whole list,
  is what this design answers for.
- **No design?** A task attached directly to a requirement or feature has no
  design and no coverage to read. The requirement (or feature) is then your
  primary spec, and the criteria you owe are the ones the task body names. If
  the task turns out to need architecture the body doesn't settle, that is a
  missing design: flag it rather than deciding it yourself.
- **Acceptance criteria** — the testable conditions your code must satisfy.
  Your work is done when every one you owe holds.
- **Requirement** — the behavioural rule the criteria prove.
- **Feature** — the capability and why it matters, and the scope boundaries
  that tell you what *not* to build.

**Read the chain in three rounds, not one call per entity.**

1. `getEntity(task)` — the ancestor chain (*Find the work* already made this
   call).
2. In parallel: `readDocument([task, design, requirement, feature])` and
   `getDesignCoverage(design)`. The one `readDocument` call returns, for every
   entity, its body (the live draft), current `status`, `viewUrl` and
   `commentThreads`; for the requirement, its `criteria` with their
   given/when/then and `because`; for the design, its review `verdict`. That
   is everything this section and *Pre-start check* need except the comment
   text.
3. `getCommentThreads` for the **open** threads that look relevant —
   `commentThreads` already gives each one's status and anchor, so a resolved
   thread never needs fetching. An anchored discussion on the design is often
   where a caveat that never reached the body lives.

Reach for `listAcceptanceCriteria` only when you need what the requirement's
`criteria` lack — each criterion's `fulfilledBy` designs or its own review
`verdict`. Criteria are addressed through their requirement (`REQ-7/AC-5`),
never by a bare `AC-n`.

Read enough that you could explain *why* the task exists and *how* it's meant
to work before touching code.

### Collect the visual design for UI work

If the task touches UI, gather every design asset the chain carries — design
tool frame links and embedded mockup screenshots — from the **task, its
design, its requirement and its feature**. The task should name the frames it
implements, but a design or requirement often carries mockups the task never
repeated. Open each one: with the Figma MCP, `get_design_context` and
`get_screenshot` on the linked node give the layout, tokens and assets; an
embedded screenshot is an image URL you can view directly. They are part of
the spec, alongside the prose.

## Pre-start check

A task can be approved and still be a poor thing to start: it may already be
under way, it may not match its design, a review may have found the design no
longer describes the software, or a comment may have raised something the body
never absorbed. Starting implementation locks the chain, so this is the
cheapest moment to find out. **Run all four checks and report the outcome
before you transition the task or cut a branch.** Everything comes from
*Gather the context*'s reads.

**a. Status — is the task ready to build?** Take the task's `status` from the
`readDocument` batch — fresh, not remembered from earlier in the session.

- `APPROVED` is the only ready state. `DRAFT`, `READY_FOR_REVIEW`,
  `REQUIRES_CHANGES` or `INVALIDATED` is a spec not signed off; `CANCELLED` or
  `COMPLETED` is work that shouldn't be redone.
- `IMPLEMENTATION_STARTED` or `PR_RAISED` means work is already under way.
  Check `listTaskPullRequests` for an existing PR and say what you found rather
  than starting a second attempt.
- A `CANCELLED` or `INVALIDATED` ancestor under an `APPROVED` task is a flag.
  `LOCKED` ancestors are normal once implementation has started anywhere on
  the chain.
- Anything on the task's `**Depends on:**` line that isn't yet `COMPLETED`, and
  isn't accounted for in the order you stated, is a flag.

**b. Comments — has anyone said something the body doesn't reflect?** Flag any
**open** thread on the task, design, requirement or feature that:

- questions, disputes or caveats something this task builds, or the design
  mechanism it follows;
- was raised **after** the task's last approval or version (compare against
  `listVersions`), so the body can't have absorbed it; or
- is an earlier implementer's or reviewer's unanswered question.

A resolved thread is settled; don't revisit it.

**c. Alignment — does the task still meet its parent design?** Check the task
describes a change the design's mechanism supports — same components,
contracts, data model and boundaries, nothing the design is silent on or
contradicts. Don't flag a design merely for having been republished: a change
that affected the task would already have invalidated it. Criteria may be split
across several tasks, so a criterion this task doesn't cover is not a mismatch.
A task with no design is checked against its requirement or feature instead.

**d. Verdict — does the design still describe the software?** Where the
organization has software review on, the design's `verdict` comes from its
latest completed review. `stale` or `contradicts` means the design is known not
to describe the software: flag it, quoting the verdict's `finding.summary`,
rather than building against it. `accurate` and `unverified` are not flags —
`unverified` only means nothing has judged that version yet. The key is absent
while software review is off, and a task with no design has no verdict.

**Then flag, don't fix.** Report one of two shapes:

- **Clear** — one line: "Pre-start check: task `APPROVED`, no open threads
  bearing on it, consistent with its design, design verdict not stale." Carry
  on to *Build*.
- **Flagged** — a short list, each item with its link, what you saw and why it
  matters (*"[TASK-12](…) builds the ride state machine as three states, but
  [DES-9](…) specifies a fourth, `DISPATCHED`, that the task never
  mentions"*). **Stop there**: don't transition, branch or build; ask how to
  proceed. A flag is a question, not a veto — the user may say carry on — but
  it is their call, made before the chain locks.

Don't repair what you flag; the spec is authoring's job. If a finding needs a
home beyond this conversation, raise it as a comment thread (see *Comments*).

## Build

- **Implement to the design's mechanism and the task's definition of done**,
  satisfying **every** criterion you owe, golden path and edge cases alike.
- **The task tells you what to write; it never contains it.** Code in the
  entities is quoted evidence of what already exists, not a draft to paste.
  Write the implementation in the repository's own idiom and along the grain
  the design describes.
- **Land it whole.** A task is scoped to merge to `main` on its own without
  dead paths — no half-wired screen or unreachable route. If the slice can't
  land that way, say so rather than merging the stranded half; the task
  boundary is wrong.
- **Keep the change to the task.** Sibling tasks are separate units of work;
  don't fold them in unless asked.
- **Follow the repository's conventions** and its `CLAUDE.md` or contributor
  rules. Comment what the code can't say for itself — a module's purpose, a
  non-obvious invariant or workaround — and never narrate the change ("added
  for TASK-12"): traceability lives in the commit and the PR.

### Build UI to the mockups

Where the chain carries design-tool frames or screenshots, implement **as
closely to them as possible** — layout, spacing, hierarchy, states and the
exact copy shown — using the repository's existing components and design
tokens. Don't invent layouts, components or copy, and don't "improve" on the
mockups. Depart from them only where they **don't cover** what you need (a
state, breakpoint or message they never show): fill the gap in the mockups'
visual language and the product's copy conventions, and list every gap-fill in
the PR description. A gap bigger than a detail — a whole screen or flow — is a
design gap: raise it (see *If the design is wrong or silent*). If a mockup
conflicts with the design's prose or a criterion, flag it rather than picking
one.

### Tests trace to their criteria

- **Write the tests the task names**, including the regressions it calls out.
  The design names the areas of testing; the task names the tests. A criterion
  with no assertion that fails when it stops holding is not implemented,
  however green the suite.
- **Annotate each test with the criterion it proves**, as `REQ-7/AC-5` — the
  owning requirement *and* the criterion, since a bare `AC-5` names nothing.
  Put it where the repository carries test metadata, normally the test name:

  ```ts
  it('rejects a ride request with no payment method [REQ-7/AC-5]', async () => {
  ```

  List several where one test covers several (`[REQ-7/AC-4, REQ-7/AC-5]`).
  Leave tests that map to no criterion unannotated — don't invent one. This is
  how a reviewer, and later `spexd-auditing`, gets from a criterion to its
  assertion.

### If the design is wrong or silent

Don't guess an architecture, and don't edit the design. Stop and surface the
gap to the user, and raise it as a comment thread on the design anchored to
the passage (see *Comments*). The design is fixed before the code that depends
on it, never the other way round. Once implementation has started anywhere on
the chain it is `LOCKED`, so the design can only change inside a proposal —
the user's call, made with `spexd-authoring` (its *Proposals* section), not a
workaround for you to reach for.

## Reflect status

A task's implementation statuses are read by the rest of the chain, so they
move as the work moves:

- **`APPROVED → IMPLEMENTATION_STARTED` — yourself, the moment work starts:**
  after the pre-start check, before the branch and the first edit. Waiting for
  a branch push leaves the task looking unclaimed while files change, and two
  agents can pick it up. Starting `LOCKS` the ancestor chain, freezing the spec
  you build against. A task that isn't `APPROVED` isn't ready — raise it, don't
  bypass it.
- **`IMPLEMENTATION_STARTED → PR_RAISED`** — when the work moves into review.
- **`→ COMPLETED`** depends on whether the work raises a pull request:
  - **With a PR, the merge completes it.** `PR_RAISED → COMPLETED` is
    system-driven; setting it by hand is refused.
  - **With no PR** (a configuration, data or spec-only task), move it
    `IMPLEMENTATION_STARTED → COMPLETED` yourself once the work is done and
    verified. Don't use this route to skip a PR the work should have had.

Never record status in the task's content ("done", "in progress"). **Never
move anything to `APPROVED`** — approval is human-only, enforced server-side.

### Name the task in the branch and the PR

Where the repository is connected to Spexd, GitHub activity moves the task: a
branch naming an `APPROVED` task moves it to `IMPLEMENTATION_STARTED`, marking
its PR ready for review moves it to `PR_RAISED`, and merging completes it. A
hop that never fired is applied on the way. The branch hop is a backstop for a
move you already made; the PR hops are the mechanism proper. So naming is not
bookkeeping:

- **Put the reference in the branch name**, e.g.
  `claude/task-12-ride-request-endpoint`. The scan is case-insensitive and a
  trailing slug is ignored.
- **Title the PR `{reference}: {title}`** — `TASK-12: Implement POST
  /api/rides`. The reference leads, as plain text; the markdown links go in
  the description.
- **Exactly one task in each, and nothing else** — no second task, no parent.
  `DES-320: add ride requests` is wrong even when the branch builds the whole
  design; so is `TASK-612 + TASK-613: schema and endpoint`.
  `resolveGitHubBranch` takes the deepest reference named, so a name with only
  a parent resolves `out_of_scope`, and one with two tasks is `ambiguous` —
  either way, nothing moves.
- **The description carries the context**: link the task, the design it
  executes, the criteria it satisfies, any mockup gap-fills, and the PR
  beneath it in a stack.

### When the task didn't move

Transitions ride on webhook delivery, so they can lag or, on an unconnected
repository, never arrive. Check rather than assume: `listTaskPullRequests`
shows the PRs Spexd has observed, and `listEntityRepositories` whether the
task is linked to a repository at all.

- **If Spexd hasn't seen the PR, link it — don't just move the status.** Only
  the merge of a PR Spexd has linked completes the task, so a task moved to
  `PR_RAISED` by hand over a PR missing from `listTaskPullRequests` will never
  complete. Attach it with `linkUnlinkedWorkToTask` (by `githubPrId`, or the
  repository id and branch; `listUnlinkedWork` finds the row). That drives the
  lifecycle as the webhook would, and `lifecycle.applied` reports what moved.
- **Transition by hand only when the PR is linked and the status still lags.**
  `transitionEntityStatuses` takes 1–25 `{ reference, targetStatus }` pairs
  and decides each on its own: **read `failed`** rather than assuming success.
  A task the webhook already moved refuses the same hop — a no-op to
  recognise, not an error to retry.

## Given a parent entity: work it task by task

A feature, requirement or design is never itself a unit of work: only a task
carries a definition of done, a repository and the lifecycle GitHub drives.
Pointed at a parent ("implement DES-320"), don't land it in one branch and one
PR — expand it into its tasks and ship them one at a time.

1. **Enumerate the tasks** with
   `listDescendants(ref, { types: ["TASK"], maxDepth: -1 })` — each row
   carries its immediate parent, normally its design. Add
   `statuses: ["APPROVED"]` when the ask is "what's next" rather than "all of
   it". Report the set as *Orient — open with the entity* asks.
2. **Decide the order, and state it before you start.** Each task's
   `**Depends on:**` line — the first line of its body — names what must land
   first; read the bodies in batched `readDocument` calls (up to 10 each) and
   let those lines drive the sequence. Where nothing blocks, order for
   reviewability: schema and contracts before the code that uses them, the
   server before its client, what merges alone before what needs it. A person
   can correct the order cheaply now, and expensively once three PRs are open.
3. **Stack only where there is a real dependency.** Branch the first task off
   `main`; branch a task that builds on another's code off that task's branch
   and target its PR there, so each PR's diff is exactly one task. A task that
   doesn't build on the others branches off `main` and is reviewed in
   parallel.

   ```
   main
   └── claude/task-612-ride-request-schema          → PR: base main
       └── claude/task-613-ride-request-endpoint    → PR: base claude/task-612-…
           └── claude/task-614-ride-status-webhook  → PR: base claude/task-613-…
   ```

4. **One task's full loop at a time, PR raised before moving on** — start it,
   build it, raise its PR, confirm `PR_RAISED`, then the next. Raising each as
   it finishes lets review start while you build the rest.
5. **Keep the stack honest as it merges.** When a PR merges and its branch is
   deleted, GitHub retargets the PR stacked on it at the merged branch's base;
   if the branch is kept, retarget it yourself. Merge or rebase down the stack
   so each PR shows only its own change.

If a task partway down is unbuildable — its design is wrong, a dependency isn't
done, the slice can't merge without dead paths — **stop at that task**. Don't
stack the rest on the gap. Land what is reviewable, say where you stopped and
why, and raise the problem as *If the design is wrong or silent* describes.

## Operational notes (MCP surface)

The tool descriptions give each call's arguments. These notes cover what they
leave to your judgement.

### Reading

- `readDocument` is the only tool that returns a body; every other response is
  metadata. It reads the live draft by default and batches 1–10 references,
  each returned whole, so name only what you'll read. It is the one document
  tool allowed on a frozen (`LOCKED` / `COMPLETED` / `CANCELLED`) entity —
  which the chain is, once you start.
- Every entity is addressed by its bare reference (`TASK-12`, `DES-9`) — no
  kind, no owning feature. A criterion is the exception: it is numbered within
  its requirement, so it is always addressed through it.
- `listVersions` is an entity's timeline of versions, transitions and
  approvals; `compareVersions` diffs two published versions. Worth reading when
  a design was invalidated and re-approved and you need to know what moved.

### Links and references

- **Link every entity you mention** — in replies, PR descriptions and commit
  messages — to its `viewUrl`: `[TASK-12](https://acme.spexd.com/e/TASK-12)`.
  Take it from a tool response; never compose one. The host is the
  organization's own subdomain, and a composed `spexd.com` link opens nothing.
  Don't write the older positional form (`/feature/FEAT-3/REQ-7/…`).
- `getOutstandingWorkForEntity` items carry no `viewUrl`; read the entity if
  you need its link.
- **Never invent references, versions or URLs** — read them from responses.

### Comments

- Comments are accepted in any status, so a thread is how you raise a problem
  on a locked spec. Start one with `createCommentThread`, anchored to the exact
  passage with `quotedText` (or to one criterion with `criterion`), and reply
  with `addComment`.
- Resolve a thread with `resolveCommentThread` only once its point is
  addressed and you have replied saying how. A question you raised about the
  spec is not yours to resolve — a person answers it by amending the spec.
  **Never reopen or delete a thread.**

### You don't author while implementing

The chain you build against is `LOCKED`, and `editDocument` and publishing are
refused on it. Raise what is wrong as a comment thread and stop. If the user
asks you to change the spec, that is authoring work: load `spexd-authoring`'s
*Editing*, *Publishing* and *Proposals* sections and follow them.

## Process checklist

1. **Find the work.** Given a parent, list its tasks, order them and state the
   order, then run steps 2–8 once per task.
2. **Orient:** link, title and summary, before any plan or code.
3. **Gather the context** in three rounds; for UI, collect every mockup.
4. **Pre-start check** — status, comments, alignment, verdict. If anything is
   flagged, stop and ask.
5. **Start:** move the task to `IMPLEMENTATION_STARTED`, then branch with
   exactly one task reference in the name.
6. **Build** to the design and the definition of done; annotate each test with
   the `REQ-x/AC-y` it proves; match the mockups.
7. **Raise the PR** as `{reference}: {title}`, confirm `PR_RAISED` (or link
   the PR), and link the chain from the description.
8. **If the design proved wrong**, raise it as a comment thread and tell the
   user — never let the code and the spec diverge.
