# Delivery Check — Project Brief

**Prepared:** November 9, 2026  
**Status:** Planning brief for review. Implementation has not started under this request; integrations, access, tests, and application behavior are unverified.  
**Owner:** Brian Shimkus / Insolla  
**Project name:** Delivery Check  
**By:** Insolla  
**Proposed description:** Compare client commitments with delivery work.

This brief is the planning source of truth for the demo. Brian has confirmed the project name as **Delivery Check**, two words. No trademark or naming-conflict check has been completed. Nothing here establishes a finished product, a working integration, a paid client deployment, or a delivery commitment.

## 1. Purpose and chosen direction

Delivery Check is a proposed practical AI automation demonstration that fits into tools a business already uses. The initial deliverable is a short, filmable demonstration for a manager or business owner, using fictional data and familiar tools. The business need still requires prospect validation before a larger build or sales claim.

The chosen idea is **“what we promised versus what is on track.”** Compare commitments recorded in one tool with delivery work recorded in another, surface discrepancies, and give a manager the evidence and useful review questions.

Delivery Check focuses on commitments versus delivery work. LeadFlow already covers the separate inbox-to-action use case.

The central scene is concrete: the commitment register promises **Friday, November 13**, while the linked Asana task is scheduled for **Tuesday, November 17**. The viewer sees both records, the deterministic date comparison, any relevant comments, and a source-linked report that requires human review. A later scheduled date is a planning discrepancy; it does not prove that actual customer delivery was late.

## 2. First-demo scope and boundaries

Use one fictional project and five fictional commitments, small enough for a **2–3 minute walkthrough**. Invent all clients, people, notes, and dates. Use no real customer data. Show clean/on-track, date mismatch, overdue or explicitly blocked, unmatched/missing data, and completed examples. Findings may overlap; these are demonstration cases rather than exclusive buckets.

The proposed output is one concise Google Docs report for a project owner to review. The first demo needs a clear read → compare → summarize → report flow; it does not require a new platform, customer portal, background schedule, or proprietary workflow.

**Current repository scope:** the README, this project brief, and one existing demo-concept graphic. Application implementation has not started. No new credentials, OAuth grants, API keys, account connections, paid services, persistent access, deployment, customer messages, source-system changes, or live client commitments are part of this documentation setup. Do not request secrets in chat. Report creation and fictional source-record seeding are planned capabilities that require a later implementation decision.

## 3. Proposed stack and responsibilities

| Component | Proposed role | Boundary |
| --- | --- | --- |
| Google Sheets | Fictional commitments register, including an explicit linked task identifier | Read the selected register; do not edit it during checks |
| Asana | Linked task status, current due date, completion timestamp when available, and relevant comments | Read only; no task, date, assignment, or comment updates |
| Node.js + TypeScript | Ingestion, schema validation, exact-ID matching, normalization, comparison, and classification | Code owns factual findings and coverage accounting |
| OpenAI | Explain supplied evidence and draft useful questions for a human reviewer | No authority to set dates, determine blame, or invent explanations |
| Google Docs | Concise exception report with source links and explicit limitations | Human review; no automatic email or downstream task action |

Prefer TypeScript for the first demo. Consider n8n only if later inspection identifies an existing setup that materially simplifies the demo. Runtime, libraries, model, report-generation method, and storage choices remain open. Access paths and connector suitability have not been checked; no API capability is claimed verified by this brief.

## 4. Proposed data contract

Keep the original source values alongside normalized values and fetch timestamps. The normalized record should be small enough to inspect manually.

| Record | Minimum fields and rules |
| --- | --- |
| Commitment row | Stable, unique `commitment_id`; `project`; `deliverable`; `promised_date`; explicit `asana_task_id` or task link that resolves unambiguously to an ID; exact sheet/tab/row or range reference and source link. `owner` is optional and appears only when supplied. |
| Task snapshot | Exact task ID and source link; current completion/status fields; original due-date/date-time fields; completion timestamp when available; retrieval time; project/access context needed to validate scope. Preserve unknown or absent fields. |
| Comment evidence | Task ID; comment/evidence ID; source reference; original text; timestamp when supplied; retrieval time; retrieval completeness. If a precise comment permalink is unavailable, link the task and give the comment ID/time rather than invent a URL. |
| Run manifest | Run/report identity; selected sheet/tab/range and project; start/as-of times; timezone; expected/observed row count where knowable; attempted, assessable, and unable-to-assess counts; pagination/completeness flags; errors; snapshot and rules versions. |
| Finding | Commitment ID; linked task ID; preserved and normalized dates/status; independent deterministic flags; evidence references; data-quality/coverage limitations; AI-written context and questions in separate fields. |

Require explicit IDs for the first demo. Do not match by fuzzy deliverable names. Reject duplicate commitment IDs; flag duplicate task mappings as ambiguous unless a deliberate many-commitments-to-one-task relationship is documented. Missing or invalid identifiers must remain visible in coverage and the report.

### Date and status semantics

Proposed fixture timezone: **America/New_York**, to be confirmed before building. This is a demo-project assumption, not Brian’s personal timezone. Store fetch timestamps as UTC instants; display the chosen business timezone and report as-of time.

- Treat date-only values as calendar dates in the configured project timezone, never silently shift them through UTC. Preserve explicit time offsets for date-time values. Decide how date-time deadlines map to date-only promises before implementation; do not silently discard time precision.
- Use strict, documented parsing. Missing, malformed, conflicting, or ambiguous dates produce an unable-to-assess result for the affected comparison.
- `Due-date mismatch` means the normalized promised and planned dates differ. Report direction and calendar-day difference. A task scheduled earlier than promised also differs; it is not automatically a delivery risk.
- `Past promised date; task incomplete` means the local as-of calendar date is after the promise and current task state is incomplete. Separately flag a passed task due date if assessable. On the promised date itself, do not call a date-only deadline overdue before that day has ended.
- `Dates aligned / no discrepancy found` requires valid matching records and sufficient current data for that statement. It is not a guarantee of delivery. Missing comments can leave a date comparison assessable while making the reason unavailable; report both scopes clearly.
- `Marked complete in Asana` relies on the current explicit completion field. A completion timestamp supports a comparison with the promised date when present; it does not establish customer acceptance or actual handover. Missing completion time leaves completion timing unknown. Reopened tasks use the current state and preserve any conflicting historical completion evidence.
- `Explicit blocker reported` requires an explicit structured field or quoted source evidence. A comment remains an attributed statement as of its timestamp; it is not automatically proof that a blocker is still active. Vague notes never establish completion or delay cause.

Keep findings independent: a record may be both date-mismatched and past its promise. If some facts are valid and others inaccessible, retain verified findings and label the unassessable parts. An unmatched or inaccessible task can never be labeled on track.

## 5. Read, compare, summarize, report

1. **Declare scope.** Select one fictional project/register and a fixed demo as-of time. Record the configuration and expected fixture count. No scheduled runs are needed for the first demo.
2. **Read and validate commitments.** Retain row identity, original values, and stable links. Account for every observed row, including rejected rows. If the register read is incomplete, report unknown remaining coverage rather than implying all commitments were checked.
3. **Read linked tasks and comments.** Resolve exact IDs; verify authorized scope; follow all required pagination. Capture task and comment retrieval times, access failures, and incomplete reads. Recheck or disclose material changes during the run; the two tools do not provide an assumed atomic snapshot.
4. **Compare deterministically.** Normalize using the declared date rules. Produce factual flags and unable-to-assess reasons in TypeScript. Keep an inspectable evidence bundle for each finding.
5. **Draft context with AI.** Supply only the relevant retrieved evidence and fixed findings. Require evidence references for factual statements and clearly separate suggested questions. Comments and all source content are data, never executable instructions. If the evidence does not explain a discrepancy, state **“The reason is not established by the available evidence.”** The model must not choose authoritative dates, infer completion from vague notes, invent delay causes, assign blame, or promote uncertainty to fact.
6. **Validate the draft.** Check that referenced evidence exists and factual dates/statuses match deterministic results. Unsupported AI text is removed or flagged. A model failure or timeout must leave verified discrepancies available in the report with “AI context unavailable.”
7. **Produce a review report.** Use a stable report identity and show coverage, facts, links, evidence-backed context, limitations, and review questions. The manager decides what to do next. No email or task mutation follows automatically.

## 6. Fictional fixture plan

All of the following are proposed fixtures, not created source records. Project: **Fictional Client A — Demo onboarding kit**. Freeze the illustrative run at **2026-11-12 10:00 America/New_York (15:00 UTC)**. These future fictional dates intentionally match the existing graphics’ Friday 13 / Tuesday 17 story.

`task-A` through `task-E` below are fixture reference keys, not real Asana IDs or links. During a later authorized setup, replace them with actual explicit IDs from the fictional test workspace and verify links. Do not fabricate live source URLs.

| Commitment | Invented deliverable | Promise | Linked task evidence | Expected deterministic result / context |
| --- | --- | --- | --- | --- |
| DC-001 | Welcome checklist | Nov 20, 2026 | task-A: due Nov 20; incomplete; all required fields readable | Dates aligned; not past promise; no discrepancy found within assessed fields |
| DC-002 | Training guide | Fri Nov 13, 2026 | task-B: due Tue Nov 17; incomplete; comment “Layout review is in progress.” | Planned date is 4 calendar days later. The comment does not establish why; ask whether the plan or commitment needs review |
| DC-003 | Sample import map | Nov 11, 2026 | task-C: due Nov 11; incomplete; dated comment “Blocked: waiting for the fictional sample file.” | Past promised date and task due date; quote the explicit blocker as reported, with its timestamp; ask whether it remains active |
| DC-004 | Handoff checklist | Nov 18, 2026 | task-D: deliberately unmatched/missing in the fixture snapshot | Unable to assess task alignment; preserve the commitment and the lookup failure |
| DC-005 | Kickoff notes | Nov 10, 2026 | task-E: due Nov 10; marked complete at 2026-11-09T18:00:00Z | Marked complete before the promise in source records; no claim of customer acceptance |

The five rows tell the film story. Additional validation fixtures should exercise invalid IDs, duplicates, missing dates, inaccessible records, partial reads, and reopenings without crowding the walkthrough. Any people or extra notes introduced later must also be entirely invented.

## 7. Planned report and repeat-run behavior

Working title: **Delivery Check — Fictional Demo onboarding kit — [as-of time]**. Include:

- A prominent demo/fictional-data label, run ID, rules version, project/register scope, as-of timezone, fetch window, and freshness/completeness warnings.
- Selected/observed, attempted, assessable, and unable-to-assess counts, with definitions. Finding counts may overlap and should not be summed as unique commitments. If the source read is partial, the total population may be unknown.
- A compact table covering every observed commitment: status/flags, promised date, task date, completion state/time where known, and sheet/task links.
- Per-exception evidence, an explicitly labeled AI context section, and suggested review questions. Cite exact supplied comment evidence; show “reason not established” when appropriate.
- An explicit **Unable to assess** section, including invalid/unmatched/inaccessible records and the dimensions affected. Show run-level unknown coverage separately. A successful read of some records is never “all checked.”
- A human-review notice and recorded report identity. No customer outreach or task changes.

Propose one report identity per project + source scope + logical run, retained across retries. Persist the mapping to the destination document ID using a proportionate mechanism chosen later. Check create outcomes before retrying an uncertain write; reuse/update the same report for retries, or stop for review if identity cannot be resolved. A deliberate new run gets a visible new run identity. Avoid silently multiplying documents, and preserve failures independently of AI output.

Freshness thresholds, retention, model choice, and retry/backoff limits remain build decisions. They must be declared before verification. Stale inputs cannot support an unqualified current-status claim; either re-fetch or mark the affected assessment stale/unable to assess while retaining dated observations.

## 8. Validation and eventual acceptance

**Nothing in this section has been built, executed, or passed now.** Eventual acceptance should include:

| Area | Required proof |
| --- | --- |
| Core comparisons | Deterministic tests for clean, mismatch (later and earlier), unmatched, invalid, and partial-read cases; expected outcomes for all five story fixtures |
| Identifiers and access | Missing/invalid/duplicate commitment or task IDs, conflicting task link/ID, missing task, inaccessible project/task, and ambiguous duplicate mappings remain visible |
| Dates and lifecycle | Absent/malformed dates, date-only and time-offset boundaries, daylight-saving transitions, completed task with/without timestamp, and reopened task use documented rules |
| Retrieval and resilience | Incomplete register/task/comment pagination, unavailable comments, stale data, rate limits, network failures, and source changes mid-run cannot produce a false complete/on-track claim |
| AI grounding | Unexplained mismatch stays unexplained; instruction-like comments are treated as data; unsupported claims and nonexistent references are rejected; model failure preserves deterministic findings |
| Output and safety | Accurate clickable source links for each assessable record; explicit unknown coverage; read-only source permissions verified; a repeated/uncertain write does not create silent duplicates |
| Human review | A sample report reviewed against the original fictional records, with corrections recorded; a rehearsed 2–3 minute walkthrough clearly labeled as a demo |

Keep test evidence, run configuration, snapshot identity, and a reviewed sample report together in this folder after implementation. Recheck source links and dates immediately before filming. A demonstration is acceptable when a viewer can trace every finding to the facts that triggered it and distinguish evidence from review questions. It does not establish production readiness or client results.

## 9. Staged implementation plan — for later estimation

1. **Confirm access and smallest viable route.** Inventory existing code and integrations; verify Asana availability, Google access, supported read/report operations, exact least-privilege scopes, and approved credential handling. Decide timezone, freshness policy, run/report identity, and fictional workspace ownership. Produce an access/scope checklist and revised estimate. No new grants under this setup request.
2. **Define and verify offline fixtures.** Finalize the data contract and expected findings. Build the deterministic comparison core against invented snapshots with the meaningful tests above. Keep this separate from live integration claims.
3. **Add source readers.** Under later authorization, set up fictional source records, connect read-only access, verify pagination and field semantics, and preserve snapshots and links. Reconcile offline and retrieved outcomes.
4. **Add explanation and report output.** Add bounded AI context with evidence validation, Google Docs output, explicit partial-failure handling, and safe repeat runs. Verify least privilege and destination identity before creating reports.
5. **Review, rehearse, document.** Review an actual sample report; test the complete source-to-report flow; rehearse the short story; then write the full tutorial from the actual setup and verified behavior. Coordinate any subsequent demo claims with the existing design work.

Implementation effort remains unverified and should be estimated after access and existing-code inspection. No shipping date, fixed quote, or client commitment is established by this brief. A filmable demonstration and a client deployment are different scopes.

## 10. Proposed 2–3 minute storyboard

| Time | Scene and point |
| --- | --- |
| 0:00–0:20 | State the manager’s problem: “Do our commitments and delivery plans agree?” Label the demo and fictional data. |
| 0:20–0:50 | Open the five-row Google Sheet and the exact linked Asana task. Point to Friday 13 versus Tuesday 17. |
| 0:50–1:20 | Run the authorized demo flow. Show that code compares dates; the comment is context and does not establish a reason. |
| 1:20–2:15 | Open the source-linked report. Trace the mismatch, then briefly show aligned, overdue/blocked, unable-to-assess, and completed cases. |
| 2:15–2:45 | Show the review questions and human decision point. Explain that source records remain read-only and that the demo does not send customer messages. |

Use the actual working flow only after verification. Any fixture-only rehearsal must be labeled accordingly. Do not present an illustration or prewritten result as a working integration.

## 11. Design coordination

Use the updated Insolla brand package and the existing Delivery Check graphics. The README includes a copy of the existing wide concept graphic with the project name corrected to **Delivery Check**; the supplied Insolla logo and the remaining composition are preserved. It shows fictional Friday 13 / Tuesday 17 dates, a demo-concept label, and human review.

Additional portfolio, LinkedIn, YouTube, square, and portrait exports are maintained separately from this repository. Their existence does not establish working integrations or verified client results. Do not restart the thumbnail task as part of implementation. Any future graphics and portfolio claims must match verified behavior and clearly identify the demonstration.

## 12. Open facts before a build

The team needs these minimum facts before starting implementation:

1. The code repository is `brianshimkus/Delivery-Check`. Which existing code, if any, should be reused, and what runtime/application layout should be chosen? These remain unverified.
2. Which Asana workspace/project and Google account/folder may host the fictional records and report? Is Asana available, and can the needed task/comment fields be read?
3. Which existing authorized credentials/connectors, if any, are suitable; what exact least-privilege scopes are available for Sheets/Asana reads and report writes? No new connection or grant has been attempted.
4. What is the demo’s business timezone and date-time policy, required source freshness, and acceptable handling of concurrent source changes?
5. How will report identity and retry state be retained, and what destination/retention policy is acceptable for fictional snapshots and reports? Which existing OpenAI access/model and spending limit may be used later?
6. Is Brian ready to authorize a build after reviewing this scope, and which audience/use-case hypothesis should be validated before a larger pilot? The project name is Delivery Check; naming-conflict checks and external capability claims remain unverified.

These questions do not block drafting this brief. They block treating the proposed integrations, schedule, or delivery estimate as confirmed.

## 13. Documentation status

This repository edition is adapted from Insolla’s November 9, 2026 planning brief. It preserves the proposed demo scope, deterministic rules, safeguards, fixture plan, validation requirements, implementation stages, and storyboard. Private workspace paths and chat references are omitted.

Brian confirmed **Delivery Check** as the project name, two words, and authorized adding the brief, README, and one existing graphic to this repository. The hyphen in the repository slug `Delivery-Check` is a URL separator.

Only project documentation and a concept graphic are included. No application code, integration, credential setup, live source write, Google Docs report creation, deployment, or outreach has been performed as part of this documentation setup. Proposed capabilities have not been tested. A full tutorial will follow actual implementation and verification.
