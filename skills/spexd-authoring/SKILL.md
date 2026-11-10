---
name: spexd-authoring
description: >-
  Author and curate the Spexd traceability chain — features, requirements,
  acceptance criteria, designs and tasks — via the Spexd MCP tools. Use when
  asked to write, refine, reorganise or publish any Spexd entity, or to add
  entities from tickets, docs or code. Encodes the altitude lens so each
  entity lands at the right level and implementation detail never leaks
  upward into a feature or requirement. To bootstrap a whole spec from an
  existing codebase, use spexd-importing instead.
user-invocable: true
---

<!--
Section headings are an API. Spexd serves this skill one section at a time
through loadSkill, and its tool descriptions name sections exactly ("Feature",
"Requirement", "Acceptance Criteria", "Design", "Task", "Content conventions
(all entities)"); sibling skills cite others ("What a good design covers").
Renaming a heading breaks those pointers. There is deliberately no H1: an H1
would contain every section, so loading it would return the whole body.
Every cross-reference in italics must be a heading loadSkill can return.
-->

## Start here

Spexd models a strict chain: **Feature → Requirement → Acceptance Criteria →
Design → Task**. Placing content at the right level matters more than wording —
classify first, write second. A misplaced entity can only be retired by
cancelling it, which is irreversible.

| Level | Answers | Altitude | Written for |
|---|---|---|---|
| **Feature** | *What can someone do, and why does it matter?* | Product scope | Senior product managers and executives |
| **Requirement** | *What must be true within that capability?* | Behavioural rule | Product managers, engineering leadership, architects |
| **Acceptance Criteria** | *How will we know it's true?* | Testable condition | Product owners and quality engineers |
| **Design** | *How do we build it so those conditions hold?* | Architecture & mechanism | Architects, readable by product owners and senior engineers |
| **Task** | *What work ships a slice of that design?* | Unit of work | Senior engineers |

Feature and Requirement are the *what* and the *why*; Design and Task are the
*how*. Acceptance criteria sit on the hinge — still *what*, but precise enough
to prove. Every entity is written to be as human-readable as possible for its
readers.

**Load only what the job needs.** A name in italics below is a section:
`loadSkill({ name: "spexd-authoring", section: "<heading>" })`. A section
includes its subsections.

| You are about to… | Load |
|---|---|
| Decide which level something belongs at | *The one rule that governs everything: detail only descends* |
| Write or rewrite a feature | *Feature*, *Content conventions (all entities)* |
| Write or rewrite a requirement | *Requirement*, *Content conventions (all entities)* |
| Write or change an acceptance criterion | *Acceptance Criteria*, *Acceptance criteria and coverage* |
| Write or rewrite a design | *Design*, *Content conventions (all entities)* |
| Write or rewrite a task | *Task*, *Content conventions (all entities)* |
| Edit an existing document | *Editing*, *Content conventions (all entities)* |
| Publish | *Publishing* |
| Move, retire or cancel entities | *Status, cancelling and references*, *Process* |
| Attach a file, screenshot or comment | *Attachments and comments* |
| Work inside a proposal, or change a frozen entity | *Proposals* |
| Work without a shell or the repository | *Without a shell or the repository* |
| Populate or reorganise several levels | *Process* — or load the skill `whole`, since you will use most of it |

## The one rule that governs everything: detail only descends

Implementation detail first appears at **Design** and nowhere above it. "How"
means architecture, not just UX: data models, transactions, concurrency,
failure handling and external services all belong in Design.

Above Design, keep out:

- **Vendors and products** — Clerk, Postgres, Redis, Stripe, an SMS provider.
- **Libraries, frameworks and technologies** — RLS, PgBouncer, Yjs, zod, H3
  cells, WebSockets.
- **Tables, columns, schemas, keys, indexes.**
- **Algorithms and data structures** — a reference counter, a CAS on a version
  column, a state machine, an outbox.
- **Code, interfaces, type shapes, HTTP contracts.**
- **Engineering effort and sizing** — lines of code, story points, which
  service or repo it lands in.

A technology name in a feature or requirement means the entity is misplaced:
the capability is the entity, the technology is design. Example: the feature
*"Organization Data Isolation (Postgres RLS)"* became the requirement
*"Organization Data Isolation"* under Authentication & Organizations, with RLS
moved down to a design.

**Measurable thresholds are the exception** and are welcome above Design —
"within 1 second", "± 10% of final fare". They are observable outcomes, not
mechanisms: "updates every 5 seconds" is a requirement or AC; "polls an H3
index every 5 seconds" is design.

### What belongs where — at a glance

| Content | Feature | Req | AC | Design | Task |
|---|:---:|:---:|:---:|:---:|:---:|
| Product problem, value, persona, metrics | ✓ | | | | |
| Behavioural rule ("the rider can…", "every X must Y") | | ✓ | | | |
| Single testable pass/fail scenario | | | ✓ | | |
| Measurable threshold (500 ms, ±10%) | goal-level | ✓ | ✓ | ✓ | ✓ |
| Vendors / libraries / frameworks | ✗ | ✗ | ✗ | ✓ | ref |
| Data model, as a field/type/description table | ✗ | ✗ | ✗ | ✓ | ref |
| Endpoints and contracts, as a table | ✗ | ✗ | ✗ | ✓ | ref |
| Algorithms / data structures | ✗ | ✗ | ✗ | ✓ | ref |
| Security, configuration, deployment controls | ✗ | ✗ | ✗ | ✓ | ref |
| Existing code, quoted as evidence of the approach | ✗ | ✗ | ✗ | ✓ | ✓ |
| **New implementation code** | ✗ | ✗ | ✗ | ✗ | ✗ |
| Areas of testing that prove the covered ACs | ✗ | ✗ | ✗ | ✓ | ✓ |
| Architecture diagram | ✗ | ✗ | ✗ | ✓ | |
| Trade-offs / alternatives considered | ✗ | ✗ | ✗ | ✓ | |
| Definition of done, regressions, blocking tasks | | | | | ✓ |

An entity may reference the level above, as a link (see *Content conventions
(all entities)*). None may pre-empt the level below.

### The one thing that belongs at no level: the implementation itself

No entity — not even a design or a task — contains new implementation code.
The test: **could an engineer paste this into the repository as their
implementation?** If yes, cut it. Quoting code that **already exists**, cited
to its file, is welcome in designs and tasks: it shows the pattern to conform
to, or the thing that has to change.

Pin down a contract or schema as a **table** (field, type, description), not
as a type declaration, migration or request body. A table is readable by the
product readers too.

## Feature

Also load *Content conventions (all entities)* before writing one.

**What it is.** The root of the chain, with no parent: a unit of product
scope — a surface, workflow or capability someone can demo or point at.

**Readers.** Primarily senior product managers and executives. They should
grasp the problem, the value and the scope in a minute's read, with no
technical background.

**Feature or requirement?** If it reads as a rule *within* a capability
("every edit is a new version", "no data leaks across orgs"), it's a
requirement. Two common misfires:

- **A cross-cutting concern raised as a sibling feature.** Versioning, audit
  and numbering apply to every entity; they are requirements under the core
  model feature.
- **A behaviour of an existing surface raised as a new feature.** "Create
  entities from the navigator" is a requirement under the navigator's feature.

When torn, choose **requirement**: a stray top-level feature is noise until
someone cancels it.

**Projects.** A project (`PROJ-n`) groups features — a release, a programme, a
customer. It is a facet across the chain, not a level in it: a feature sits in
any number of projects, or none. File a new feature under the projects the
user names (`projectRefs` on `createFeature`, or `addFeatureToProject`
afterwards). Create a project only when asked.

**Structure.** Problem and value in product language, for a reader deciding
*whether* to build it, not *how*:

- `## Overview` — one or two sentences: the capability and why it matters
- `## Goals`
- `## Non-goals`
- `## Personas` — who uses it, primary first
- `## Scope` — **In scope:** … / **Out of scope:** …
- `## Success metrics` — only where they exist
- `**Sources:**`

**Keep out.** Everything in *The one rule that governs everything: detail only
descends*, and Given/When/Then phrasing (that's an AC).

## Requirement

Also load *Content conventions (all entities)* before writing one.

**What it is.** A rule that holds within exactly one feature — "the rider can
request a ride now", "every edit is a new immutable version". It qualifies its
feature and can't be demoed on its own.

**Readers.** Primarily product managers, engineering leadership and
architects. A product manager must be able to own the rule; an architect must
be able to design against it without asking what it means.

**Requirement or AC?** If it has a concrete precondition, a concrete action
and a single observable outcome you could write a pass/fail test from, it's an
AC. "The rider can tip after a trip" is a requirement; "*given* a completed
trip within the tipping window, *when* the rider adds a tip, *then* the tip is
charged as a separate transaction to the default method" is one of its ACs.

**Structure.**

- `## Overview` — the rule in one or two sentences
- `## User stories` — "As a rider standing on a corner, I want… so that…"
- `## Behaviour` — the rule in product terms, including its edge behaviour
- `## Scope` — **In scope:** … / **Out of scope:** …
- `**Sources:**`

When extracted from another entity, say so at the end of the Overview
("Formerly tracked as feature FEAT-n").

**Keep out.** Everything in *The one rule that governs everything: detail only
descends*, and Given/When/Then phrasing. No `interface` or API body to pin
down "the request shape" — that contract is Design. Seed and demo content
sometimes carries such shapes; don't copy it.

## Acceptance Criteria

Also load *Acceptance criteria and coverage* for how criteria reach a version.

**What it is.** One testable scenario that proves its requirement, belonging
to exactly one requirement. A criterion is **structured, not markdown**:
`createAcceptanceCriterion` takes separate fields, and the app renders them.

| Field | Holds |
|---|---|
| `title` | A short name for the scenario: "Tip on a completed trip", "No payment method on file" |
| `precondition` | The *given* — the state before the action |
| `action` | The *when* — what the user or system does |
| `outcome` | The single observable *then* |
| `because` | Where the criterion comes from, and the evidence behind it |

Write each field as plain prose with no "Given" / "When" / "Then" prefix and
no markdown. Omit `precondition` only when the outcome holds unconditionally.

**Readers.** Product owners and quality engineers. A product owner must be
able to agree it is the right behaviour; a quality engineer must be able to
write the test from the criterion alone.

**AC vs Design.** State what is observable, never how it's achieved. "The fare
estimate updates within 500 ms" is an AC; "…by swapping the carried estimate id
against an H3 pricing cell" is design.

**Write.**

- **Atomic:** one precondition, one action, one outcome. Split any "and
  also…" into a second AC.
- **Paths as separate ACs:** the golden path and each important failure or
  edge path (valid request; no location permission; no payment method).
- **Measurable over adjectives:** "within 30 seconds", not "quickly".

**Because — origin.** One key from a fixed set:

| Origin | Key | Meaning |
|---|---|---|
| Regulatory or legal | `regulatory_or_legal` | Required by a law, regulation or standard |
| Contractual | `contractual` | Committed to a customer or partner |
| Business constraint | `business_constraint` | A policy or commercial rule of the business |
| Product or UX decision | `product_or_ux_decision` | A choice about how the product should behave |
| Technical constraint | `technical_constraint` | Imposed by a platform, dependency or system limit |
| Edge case | `edge_case` | An unusual case of another criterion |
| Negation | `negation` | The opposite case of another criterion |

**Record an origin only when a source states it** — the user, a ticket, a
document, the code. Otherwise omit it and the criterion reads **Not stated**.
Never infer an origin from the criterion's wording. The same rule applies to
changing one; `origin: null` clears an origin a source has withdrawn.

**Because — evidence.** Back the criterion with links, or with files uploaded
first (see *Attachments and comments*). Send origin and evidence in the same
write: the create, or one update. Evidence needs no origin. To remove an item,
take its id from the write response that returned it.

**Read Because before changing a criterion.** Every criteria read returns
`because`. If the origin is **Regulatory or legal** or **Contractual**, the
wording answers to someone outside the team: name the origin and its evidence
to the user and confirm before proposing the change. This is your practice,
not a product rule: Spexd records the origin but doesn't warn on or block a
change.

## Design

Also load *Content conventions (all entities)* before writing one.

**What it is.** The first level where implementation lives: the architecture
that makes one or more ACs hold — vendors, libraries, data models, algorithms,
state machines, contracts and trade-offs, all expected here. It is
**architectural, not just UX**: a design that describes screens but not the
system beneath them is incomplete. It stops short of the code (see *The one
thing that belongs at no level: the implementation itself*).

**Readers.** Primarily architects, but product owners and senior engineers
must be able to read it too. The architect judges the approach; the product
owner follows what the system does and why; the senior engineer can tell what
has to change. That is why a design is prose and tables, not code.

**Parent and coverage.**

- A design's parent is its requirement. It covers only **that requirement's**
  ACs; moving it to another requirement drops coverage it can no longer hold.
- Within the requirement, AC↔Design is many-to-many. One design may cover
  several ACs; one AC may need several designs (a frontend and a backend).
  Split along **architectural seams**, not one design per AC. Shared
  infrastructure serving several requirements is a design under each.
- Reference the ACs it satisfies; don't restate them.

**Design vs Task.** A design describes the **target state** of the
capability; a task is **a change** that moves the code toward it. Don't
decompose a design into a work checklist — that's the Task level.

**Follow the grain of the code.** Read the code that will host the capability
before writing, and adopt the approach it already takes. If business logic
lives in the repository layer behind thin controllers, the design puts it
there — even where a logic-heavy controller would be defensible greenfield.
Name the modules and layers involved, and quote existing code where it pins
the pattern down. Still write the target state in the timeless present: not a
diff, not a migration path from today, not how much is already built. No
access to the code? See *Without a shell or the repository*.

**Structure.** Use these headings in this order. Leave out a heading only
because it doesn't apply, never because nobody thought about it.

- `## Overview`
- `## Architecture`
- `## User interface`
- `## Data model`
- `## Contracts`
- `## Security`
- `## Configuration`
- `## Deployment`
- `## Failure modes`
- `## Other requirements affected` — always present (see *Impact sweep*)
- `## Testing`
- `## Trade-offs and alternatives`
- `**Sources:**`

**Keep out.**

- **Implementation code** — express shapes as tables.
- **Build or delivery status** — no "not built", "shipped", "in progress",
  "landed in wave N", and nothing about what `main` does today. The lifecycle
  status carries that.
- **Source-code files in the Sources list** — sources are the documents, ADRs,
  tickets and links a reader follows for intent. Naming a file in the body to
  pin a mechanism is fine.
- **Open questions you never asked** — see *Content conventions (all
  entities)*. Designs attract these most.

### What a good design covers

- **Overview** — the decision in two or three sentences: what is built, the
  approach chosen, and the ACs it covers (linked).
- **Architecture** — the mechanism end to end, so an engineer knows what has
  to change, and the boundaries: how it splits across services, layers,
  clients and repositories. A multi-repo design says which part lands in each;
  its tasks decompose along that split. **Include a mermaid diagram whenever
  the design touches architecture** (see *Diagrams* below).
- **User interface** — required for any design with UI (see *UI designs carry
  their mockups* below).
- **Data model** — each entity added or changed, as a table: field, type
  (`uuid`, `timestamp`, `enum: draft | published`), purpose. Not a schema dump.
- **Contracts** — endpoints, events and interfaces as a table: method and path
  (or event name), purpose, request fields, response, error cases.
- **Security** — authentication and authorization, tenancy isolation,
  sensitive data and its protection, what an untrusted caller can reach.
- **Configuration** — flags, environment variables and settings, their
  defaults, and who can change them.
- **Deployment** — rollout and migration ordering, backwards compatibility,
  anything that must ship in sequence.
- **Failure modes** and how each is handled.
- **Other requirements affected** — each requirement or design in another
  feature that this change must keep true, linked, with how the design
  honours it; or one line saying the sweep found none, with the queries it
  ran.
- **Testing** — for each covered AC, linked: the kind and level of test that
  proves it (unit at this seam, integration across that boundary, end to end),
  plus edge and failure cases needing their own coverage. Name what an
  assertion would fail on if the criterion stopped holding. No test code.
- **Trade-offs and alternatives**, with the ADRs and tickets behind them.

### Diagrams

Every design that touches architecture carries at least one fenced `mermaid`
diagram in its `## Architecture` section — it renders in the document and stays
editable as the design changes. Pick the view that shows the decision:

- a **flow diagram** (`flowchart` or `sequenceDiagram`) for how a request,
  event or process moves through the system;
- a **component diagram** (a `flowchart` with a `subgraph` per service, layer
  or repository, or `C4Component`) for how the parts connect.

Use both when both matter. Upload an image only for what mermaid can't draw.
Never ASCII art.

### UI designs carry their mockups

Any design with a new screen, a changed flow, or a component with visible
states must include its mockups in the `## User interface` section:

- **Link the frame** — the specific Figma frame or node URL (with `node-id`)
  for each screen or state, labelled with what it shows; not the file's
  top-level link.
- **Embed a screenshot** of each mockup beside the prose it illustrates (with
  the Figma MCP, `get_screenshot` on the node), uploaded per *Attachments and
  comments*. The screenshot is visible inside Spexd; the link is the source of
  truth for spacing, tokens and assets. Give both. Without a shell, give the
  link alone (see *Without a shell or the repository*).
- **No mockups?** If you have a design tool, offer to mock the screens up
  first, following the product's design system; author without them only if
  the user declines. Without a design tool, say the design has none and
  describe layout and states in prose.

A mockup doesn't replace the mechanism, data model and contracts beneath the
screen.

### Impact sweep

A design can satisfy every criterion it covers and still break an approved
rule in another feature — one that relies on the state it changes or the code
it touches. Reading the chain above the design won't find that rule, and
neither will searching on the feature's name. Before drafting, sweep for it:

1. **Consequences.** Write down, in plain domain language, what the change
   does to users and to the system's state: "the rider can no longer request a
   ride", "the driver's status changes", "the rider is no longer sent
   receipts". Search each phrase with `searchEntities` across the whole
   organization — no `feature` filter.
2. **Code surface.** List the files, modules, tables, events and functions the
   change touches, or that already own the state it changes, and search each
   one. Designs name these in their architecture boundaries, data model and
   contracts, so a hit means another design depends on that surface.
3. **Triage.** Read each approved requirement or design from another feature
   that the sweep turns up, and decide whether this design must honour it.
   Don't rely on `findRelatedEntities` alone: it ranks by closeness of
   subject, not by side effect, so a rule on another topic that this change
   still breaks won't surface.
4. **Record** the outcome in `## Other requirements affected` (see *What a
   good design covers*).

## Task

Also load *Content conventions (all entities)* before writing one.

**What it is.** One reviewable change, in **one repository**. A task normally
delivers one design and takes it as its parent. Attach it directly to a
requirement or feature only when no design fits — a copy change, a
configuration tweak, a chore with no architecture to decide. A task never
decides architecture: if the approach turns out wrong or under-specified
mid-task, update the **design** rather than inventing architecture in the
task, and if a task with no design needs architecture, it needs a design.

**Readers.** Senior engineers. Assume they know the stack — don't explain the
language or framework — and be exact about what changes and where.

**One repository.** Link exactly one, right after creating it (see
*Repositories*). A design spanning repositories decomposes into one task per
repository, never one task across two.

**Merge-safe scope.** Once complete, a task must merge to `main` on its own
without leaving dead paths: no half-wired screen, no unreachable route, no flag
hiding an unfinished journey. If a slice can't merge safely alone, redraw it
or fold it into the task that completes the path. Prefer one larger task that
lands whole over two that strand the UI between them.

**Blockers go first.** If the task can't start until others finish, the body
opens with a `**Depends on:**` line and a bulleted list of the blocking tasks,
one linked reference per bullet, with its title:

```markdown
**Depends on:**

- [TASK-14](https://acme.spexd.com/e/TASK-14) — Add the ride state machine
- [TASK-15](https://acme.spexd.com/e/TASK-15) — Expose the fare-estimate endpoint
```

Omit it when nothing blocks — never "Depends on: none". It is a prose
convention: there is no dependency field, and `listInbox`'s `blocked` view
means *invalidated by an upstream publish*, not *waiting on a sibling*.

**Structure.** An imperative title ("Implement the ride-request endpoint").
Read the code first: an engineer should be able to start from the task and its
design with no analysis beyond verifying the existing code. **Always open with
a high-level overview before any detail.**

- `**Depends on:**` and its bulleted list — only when blocked
- `## Overview`
- `## What changes`
- `## What good looks like`
- `## Tests`
- `## Regressions`
- `## Mockups` — UI changes only
- `## Done`
- `**Sources:**`

What each covers:

- **Overview** — two or three sentences a reader could stop after: what the
  change does and why, the design it implements ("per [DES-9](…) — *Ride
  request creation & state machine*") and the ACs it contributes to, linked.
  Don't restate them.
- **What changes** — the modules, layers and files, and how each changes.
  Quote existing code where it removes ambiguity.
- **What good looks like** — the behaviour once it lands, tied to the ACs it
  serves.
- **Tests** — which to add or extend, and what each asserts. The design names
  areas of testing; the task names the tests.
- **Regressions** — existing behaviour this could break, and what must keep
  passing.
- **Mockups, for UI changes** — the specific frame links and screenshots
  (re-embedding the design's attachment URLs) for the screens and states this
  task touches. Instruct the implementer: *build to these mockups as closely as
  possible; don't invent layouts, components or copy, and don't depart from
  them without asking.* Name anything the mockups don't cover (an empty state,
  an error message, a breakpoint).
- **Done** — the definition of done and a subtask checklist, including merge
  safety.

**Keep out.** Re-arguing the design, architecture the design doesn't contain,
and new implementation code.

## Content conventions (all entities)

- **Write for the level's readers, as human-readably as possible.** Each
  entity section names its readers; check every draft against them. When a
  passage is a close call, ask which of those readers it loses, and rewrite
  it until none are lost.
- **No title in the body.** The title is its own field. Start with the content
  — or, for a blocked task, its `**Depends on:**` list.
- **No status in the body.** No "Status:" line and no "shipped / planned /
  rejected" labels. State lives in the lifecycle status.
- **Ask open questions; don't park them.** Before any open question goes into
  an entity, put it to the user — in one batch, not trickled out. The answer
  goes in as a decision. Only a question the user also couldn't settle stays,
  saying whose call it waits on. Writing an entity "with open questions to
  follow up" without asking first is not allowed.
- **Close with linked sources.** End the body with a `**Sources:**` list, one
  markdown link per source: the ticket's URL in its tracker, the page URL for
  an external reference. Quote key phrases in the body where they capture
  intent. Never invent a URL.

  ```markdown
  **Sources:**

  - [ENG-123 — Riders can schedule pickups](https://linear.app/acme/issue/ENG-123)
  - [ADR-007 — Ride state machine](https://github.com/acme/rides/blob/3f9c2e1a7b4d8e6f0a1b2c3d4e5f6a7b8c9d0e1f/docs/adr/007-ride-state-machine.md)
  ```

- **Link repository files as permalinks** to a commit the remote has, so the
  link resolves for every reader:
  `https://github.com/<owner>/<repo>/blob/<commit>/<path>`, optionally with
  `#L<start>-L<end>` for a line range. Take owner and repo from
  `git remote get-url origin`. Use `git rev-parse HEAD` only if
  `git branch -r --contains HEAD` lists a branch; otherwise use
  `git merge-base HEAD origin/<default-branch>`, provided the file exists at
  that commit. Fall back to the path in backticks when no link resolves, or
  when you have no shell (see *Without a shell or the repository*).
- **Link every entity reference** — in bodies, comments and what you report to
  the user — to its `viewUrl`: `[DES-9](https://acme.spexd.com/e/DES-9)`.
  - Take `viewUrl` from a tool response (`getEntity`, `readDocument`,
    `listDescendants`, `searchEntities`, `listInbox`, …). Never compose one:
    the host is the organization's own subdomain, and a link on `spexd.com`
    opens nothing. If you don't have it, read the entity.
  - The address is `/e/<REF>`. Don't write the older positional form
    (`/feature/FEAT-3/REQ-7/DES-9`).
  - An AC links as a query on its requirement:
    `[AC-3](https://acme.spexd.com/e/REQ-4?ac=AC-3)`.
  - `getOutstandingWorkForEntity` items carry no `viewUrl`. Their optional
    `path` is missing when an ancestor link is broken.
- **Follow the entity's structure.** Each entity section gives its headings
  in order, and every body opens with `## Overview` (a blocked task puts its
  `**Depends on:**` list first; an AC has no body). Leave out a heading that
  doesn't apply rather than writing "N/A". Features and requirements have no
  mechanism section at all.
- **Structure as tables, not code** — data models, contracts, configuration
  and permission matrices.
- **Plain, direct prose.** One point per sentence. Use the common word where
  one exists ("every 5 seconds", not "at a 5s cadence"; "read", not
  "consume"). A technical term must say something no plain word does — but
  don't strip a term the content needs.
- **Stop when your level's question is answered** (the level table in *Start
  here*). Carrying on means writing the level below's content.
- **Cut padding, hedging and restatement** from each draft before the call
  that writes it.
- **Finish by publishing, and report each publish.** An edit isn't done until
  its document is published. See *Publishing* for the review and the report.

## Operational notes (MCP surface)

The tool descriptions give each call's arguments. These notes cover what they
leave to your judgement.

### Reading

- `readDocument` is the only tool that returns a body; every other response is
  metadata. It reads the live draft, collaborators' unpublished edits
  included. Name only the references you need.
- Find things with `searchEntities` (typo-tolerant, filterable by type, status
  and feature) and `listDescendants(ref, { maxDepth: -1 })` for a whole
  subtree. ACs aren't a chain level — use `listAcceptanceCriteria`. A
  project's features: `listProjectFeatures`.
- `listVersions` is the timeline of versions, status transitions and approval
  events; `compareVersions` diffs two published versions for you.
- `:entityRef{reference=REQ-31}` directives in a body resolve to current
  title, status and `viewUrl` via `resolveEntityReferences`.

### Editing

- Read the document first, then send **every** change to it as one
  `editDocument` call — never one call per change. The batch is
  all-or-nothing; on failure, fix the op the error names and resend.
- Target by unique text (`target.find`). Use a `target.handle` from
  `searchDocument` only for what has no unique text: images, attachments,
  rules, mentions, entity references, empty bullets or cells.
- Read the response's `after` excerpts and `warnings` instead of re-reading
  the document. Expected excerpts and no warnings need no verifying read.
- A frozen (`LOCKED` / `COMPLETED` / `CANCELLED`) entity can be read but not
  edited or published — except inside a proposal (see *Proposals*).

### Publishing

- Publishing is always two calls: `publishDocument` proposes and writes
  nothing; `confirmPublish` writes. Publish once per coherent set of edits. To
  rename, pass `title` on the propose.
- **Always finish by publishing.** An unpublished draft is invisible to
  search, reviews and implementers. This holds even when the publish
  invalidates approved entities — that is the intended consequence. Stop short
  only for a stale confirm (re-propose) or when the user said not to publish,
  and say so.
- **Review outcomes before confirming** — the confirm applies them exactly as
  proposed. `invalidated`: an approved entity judged affected.
  `not_assessed`: invalidated by default, relevance never established.
  `flagged`: a work-in-progress descendant judged affected (or undetermined);
  its status is untouched and its authors get a comment. `pruned`: settled
  behind a spared ancestor.
- `skipAssessment` only widens invalidation. Invalidating something the
  assessment spared is human-only; if you disagree, say so.
- **Report every publish.** List each entity the confirm invalidated or
  flagged — including the published entity's own status, if it moved —
  grouped by kind, as a linked reference with its title and the outcome's
  reason. Mark `not_assessed` ones as such and name what was spared. If
  nothing was affected, say so in one line. Don't fix or re-approve them —
  that is the user's call.

### Acceptance criteria and coverage

- Creating, updating or deleting a criterion is one call that edits the
  requirement's draft: no publish, no version, no cascade. Read the new `AC-n`
  from the create response.
- Criteria reach a version when the **requirement** is published. Write them
  all, then publish once. Until then they are drafts: published-set reads
  (`listAcceptanceCriteria`, `getRequirementCriteria`) don't show them, design
  coverage refuses them, and `readDocument` on the requirement does show them.
- Coverage is separate from the design's document and rolls no version. Set
  it with `acceptanceCriteriaRefs` on `createDesign`, then adjust with
  `addDesignCoverage`, `removeDesignCoverage` or `replaceDesignCoverage`.
- Find an uncovered AC from `listAcceptanceCriteria`'s `fulfilledBy`;
  `listDesignsForRequirement` is the same relation from the design side.

### Repositories

- Link repositories right after creating a design or task:
  `listGitHubConnections` lists the org's repositories, `linkEntityRepository`
  attaches one by `owner/name`, `listEntityRepositories` reads them back.
- A design links **every** repository it touches. A task carries **exactly
  one** — linking another replaces it. Features and requirements may carry
  repositories as optional context.

### Status, cancelling and references

- `transitionEntityStatuses` batches moves, and an illegal move doesn't stop
  the legal ones — so **check `failed`**. **Never move anything to
  `APPROVED`**: approval is human-only.
- There is no delete: transition to `CANCELLED`. It is a soft delete — hidden
  in the app, still returned by reads — that **cascades to every descendant**,
  so check what you take with it. A cancelled parent takes no new children.
- `CANCELLED` and `COMPLETED` are terminal and freeze content and title. To
  retire an entity: (1) move content worth keeping and fix cross-references;
  (2) edit it down to a pointer and publish it — propose *and* confirm, with
  any final `title`; (3) only then cancel.
- References are server-generated. Read them from create responses; never
  guess the next number.

### Attachments and comments

- An upload never passes through your context: compute the file's size and
  SHA-256 locally, get a ticket from `createAttachmentUpload`, and send the
  bytes with the `curl` command its description gives. Never read the file or
  inline it as base64. Embed the returned URL (`![alt](url)` or
  `[label](url)`); its `id` attaches the file as AC evidence. Mint a fresh
  ticket to retry. Without a shell, see *Without a shell or the repository*.
- Comments work in any status and never change content. Start a thread with
  `createCommentThread` and reply with `addComment`.
- **Reply before resolving**, saying how the point was addressed. Resolve
  only threads whose point you addressed — a question you raised is the
  person's to settle. **Never reopen or delete a thread.**

### Proposals

A proposal (`PROP-n`) stages changes to a set of entities apart from the
primary tree until it merges. Every write and publish tool takes an optional
`proposal`.

- When the user names a proposal, pass it on every write, publish and
  transition for that work. Its changes stay invisible outside the proposal
  until it merges.
- A proposal is the only way to change an entity implementation has frozen.
  If a frozen entity needs changing and no proposal has been named, ask the
  user for one rather than working around the freeze.
- Publishing inside a proposal can pull approved entities outside it into the
  proposal (listed under `toStage`); report those with the rest of the
  publish.

### Without a shell or the repository

Some hosts load this skill with no shell and no checkout. Do what you can and
say what you skipped:

- **Reading the code** before a design or task: use a code-hosting tool if you
  have one; otherwise ask the user for the relevant files. If you write
  without reading the code, say so to the user.
- **Permalinks:** write the repository path in backticks, prefixed with
  `owner/repo` when you know it.
- **Uploads and screenshots:** you can't send the bytes. Link the source
  instead — the Figma frame, the document's URL — and use a link as AC
  evidence in place of a file.

## Process

1. **Research.** Build the inventory from real sources (tickets, ADRs, code,
   docs) and classify every candidate before the first create. Split a source
   that mixes levels: the capability goes up the chain, the mechanism waits
   for Design. **Before any design or task, read the code** it will live in;
   before any design, also run the *Impact sweep*.
2. **Ask once.** Put every gap research didn't close to the user in one batch
   before writing — including, for any design with UI, whether mockups exist.
3. **Check for an existing home** to avoid duplicates: `searchEntities` first,
   then `listFeatures` (filter by project, or `"none"`) and
   `listDescendants(FEAT-n, { maxDepth: -1 })`.
4. **Cut each draft** for padding, hedging and restatement before the call
   that writes it.
5. **Create top-down**, linking repositories as you go:
   1. `createFeature`, with `projectRefs` for the projects the user named.
   2. `createRequirement` under the feature.
   3. `createAcceptanceCriterion` for each AC, then publish the requirement so
      its criteria can be covered.
   4. `createDesign` under the requirement, with `acceptanceCriteriaRefs`;
      link every repository it touches.
   5. `createTask` with the design as `parentRef` (a requirement or feature
      only when no design fits), split along repository and merge-safety
      boundaries; link its one repository; add `**Depends on:**` where needed.
6. **Move wording, don't duplicate.** When extracting a child from a parent,
   delete the moved text from the parent in one `editDocument` call and
   publish the parent. Spexd links the child automatically — don't list it in
   the parent's body.
7. **Publish everything you edited**, and report each publish.
8. **Verify.** Walk `listDescendants(FEAT-n, { maxDepth: -1 })` and
   `listAcceptanceCriteria` per requirement, and check:
   - the created set matches the plan;
   - every AC has a design (`fulfilledBy`);
   - no implementation detail above Design, and no implementation code
     anywhere;
   - every design and task has its repository (`listEntityRepositories`);
   - every design touching architecture has a mermaid diagram, and every
     design with UI has its mockups;
   - nothing is unpublished or in an unexpected status (`listInbox`).
9. **Report** what was created, invalidated, flagged and cancelled, each as a
   linked reference with its title.
