# Development State Machine Workflow
## Reusable Workflow Prompt v1.2.1

- **Workflow version:** 1.2.1
- **Previous version:** 1.2.0
- **Scope:** A reusable, modular development workflow for applications and other projects, independent of any one app, programming language, framework, or tool.
- **Purpose:** Help people develop reliable applications without requiring them to already know the best development methods. The workflow guides design, implementation, verification, recovery, documentation, and version history, using evidence from actual work to improve the process over time.

## 1. What This Workflow Is

This document is both a process definition and a prompt that a user can give to an AI assistant to help guide development.

A user can adopt the workflow for a new or existing app and incorporate it into that app's initial design document. The workflow remains a separate, reusable component. The app's design describes the app; this workflow describes how to develop it.

The user does not need to know every useful practice in advance. Help them discover and apply appropriate practices as the project needs them. Explain decisions clearly, ask focused questions when an important choice is missing, and avoid making the process heavier than the work requires.

The workflow is independent of any particular app or project. Its practices should be evaluated in context, and further use should reveal where they work well, where they need adaptation, and where the workflow itself should improve.

**Core idea:** make a clear plan, preserve the last accepted state, implement a candidate, inspect and test it, record what the evidence shows, then decide what to do next.

## 2. Modularity and Adoption

### 2.1 Core workflow

The core workflow covers:

- understanding the user's goal and constraints;
- designing and refining the intended behavior;
- defining observable acceptance criteria;
- identifying and preserving the last known-working version;
- implementing a scoped candidate;
- inspecting source changes;
- testing actual behavior;
- repairing failures and retesting;
- updating maintenance knowledge only from verified results;
- keeping versions and history clear;
- deciding when a project has reached its desired scope.

### 2.2 Project-specific additions

An app may need additional modules for its own risks or behavior, such as persistence and migration, security, accessibility, networking, media timing, collaboration, or import/export. Add modules when they solve a real project need. Do not assume every app requires every module.

A project's initial design should state:

1. that it adopts the reusable workflow;
2. which version of the workflow it adopts;
3. which optional modules or project-specific checks are needed;
4. where its initial design, code, maintenance document, tests, and any milestone document live;
5. any justified adaptations to the general workflow.

A project can link to the workflow instead of copying its entire text. If a copy is embedded in a project's design, identify the source version so the copy can be traced and updated deliberately.

### 2.3 Keep the layers separate

- **Reusable workflow:** how development is organized.
- **App initial design:** what this particular app should be and how the workflow applies to it.
- **Application code:** what has actually been implemented.
- **Maintenance document:** the ongoing record of accepted implementation state, evidence, limitations, and next work.
- **Milestone document:** a reconciled reference for a significant accepted state of the app, detailed enough to guide reconstruction.
- **Tests and test records:** the evidence used to judge candidate behavior.

Do not turn the reusable workflow into an app-specific architecture document. Do not treat an app's design as proof that its planned behavior has been implemented.

## 3. Evidence, Learning, and Workflow Evolution

The workflow should improve through experience, not through unsupported claims that one method is universally best.

Use evidence from actual projects to identify:
- practices that repeatedly help;
- steps that are missing or confusing;
- unnecessary effort or duplicated documentation;
- failure patterns that need a better recovery method;
- useful techniques discovered during development;
- practices that work in one context but may not generalize.

A failure is not the only reason to improve the workflow. Successful experiments, clearer explanations, and more efficient ways of preserving correctness can also justify changes.

When considering a workflow change:

1. describe the problem or opportunity;
2. identify the evidence and the context in which it appeared;
3. decide whether the change belongs in the core workflow or an optional module;
4. consider compatibility with existing projects using the prior version;
5. make the smallest change that addresses the need;
6. record what changed and why;
7. use the revised workflow and continue evaluating its results.

Distinguish observed evidence from assumptions and expectations. A successful result in one context is useful evidence, but does not by itself prove that the same practice works in every context.

## 4. Versioning and Artifact Identity

Use Semantic Versioning (SemVer), written `MAJOR.MINOR.PATCH`, for the reusable workflow and all versioned project artifacts, including code, initial design documents, maintenance documents, and milestone documents. Each artifact has its **own independent version sequence**. A change to one artifact does not automatically change the version of another artifact.

### 4.1 SemVer meaning

Use the same general rules for every artifact:

- **MAJOR:** an incompatible change to the artifact's contract, structure, or expected use.
- **MINOR:** a backward-compatible addition or meaningful expansion.
- **PATCH:** a backward-compatible correction, repair, or clarification.

For app code, a new backward-compatible feature normally increments MINOR; a backward-compatible bug fix increments PATCH; and an incompatible behavior, interface, or data-format change increments MAJOR. Use judgment for documents: a substantive expansion is usually MINOR, a correction is usually PATCH, and a change that breaks the document's expected use may be MAJOR.

SemVer versions have exactly three numeric components. For example, `1.31.1` is valid; `1.31.01` is not, because numeric identifiers must not have leading zeros. Do not add extra numeric components.

### 4.2 Artifact versions are independent

Version each artifact according to changes to that artifact, not the current version of a different artifact. For example, code may be `1.31.0`, its maintenance document may be `1.8.2`, the initial design may be `1.2.0`, and the workflow may be `1.2.0`. These numbers do not need to match.

Record cross-references where useful: the maintenance document should identify the exact accepted code version and commit; project documents should identify the workflow version adopted; and a milestone document should identify the accepted code and maintenance versions it describes.

### 4.3 Candidate, accepted, and failed versions

A version number identifies an artifact version; its status describes whether it has been accepted. Do not use the version number alone to imply that a candidate passed testing. The one-current-file convention for documents does not relax code-candidate preservation: code candidates and their evidence remain recoverable as described below.

- Preserve the last accepted version as the recovery baseline.
- Develop a candidate without overwriting that baseline.
- Keep failed candidates and their test evidence available as history when practical.
- When repairing a candidate, create a new version rather than silently replacing the failed candidate. A backward-compatible repair normally increments PATCH.
- Inspect and retest the repaired version. Do not mark it accepted until the required acceptance checks pass.
- Record explicit status such as **candidate**, **failed**, **accepted**, or **superseded** in the maintenance record or version history.
- If a failed candidate is never repaired, retain its version and status as historical evidence. Do not relabel it as accepted.
- Never claim tests passed unless the tests were actually run and the result is known.

Example: code `1.31.0` introduces a feature. If testing reveals a bug, preserve `1.31.0` as the failed candidate and create `1.31.1` with the repair. If another repair is needed, create `1.31.2`. The maintenance document records which version, if any, is accepted. The failed candidate remains traceable and is not overwritten.

A repository commit provides immutable history, so project documents may keep only one current file per document type while still putting SemVer in the filename. On each document revision, increment the document's version and rename/replace the current file (for example, `InitialDesign-v0.5.2.md` becomes `InitialDesign-v0.6.0.md`); Git history preserves prior contents for comparison or recovery. Do not change a document's contents without incrementing its version, and do not rewrite Git history in a way that removes the record. This one-current-file convention applies to documentation, not to code candidates: the last accepted code baseline and failed-candidate evidence must remain recoverable under the candidate-preservation rules above. Side-by-side document versions are optional and should be used only when they serve a concrete purpose.

### 4.4 Workflow versioning

Use SemVer for this reusable workflow:

- **MAJOR:** a change that breaks the expected workflow contract or requires adopters to change how they use it.
- **MINOR:** a backward-compatible addition, such as a new optional module or an expanded adoption rule.
- **PATCH:** a backward-compatible correction or clarification that does not materially change the workflow's behavior.

Preserve released versions as historical records. Publish a new version rather than silently changing the contents of an already released version. When a project adopts a newer workflow, record the version and any relevant changes to its development plan.

### 4.5 Maintenance documents and milestone documents

A **maintenance document** is the ongoing record of development. Update it after acceptance to identify the current accepted code version, source commit or path, test evidence, known issues, and next steps. It may also record failed candidates and their outcomes. Its version changes when the maintenance document itself changes meaningfully; it does not have to match the code version.

A **milestone document** captures a deliberate, reconciled state of the app at a user-defined milestone. Give it its own SemVer sequence, independent of code, maintenance, initial design, and workflow versions. It should identify the exact accepted versions it describes. Creating a milestone does not reset or replace the app's code-version sequence.

A milestone is a reference point, not a claim that the app can never change again and not a substitute for version control.

## 5. Core Development State Machine

Use the complete path for substantial features and new subsystems. For small fixes, use the lightweight path in Section 9 while retaining the essential checks.

### State definitions

- **S0 — Request or idea:** Understand the user's goal, constraints, priorities, and definition of success. Ask concise questions when needed.
- **S1 — Initial design:** Describe the intended app or feature, including goals, expected behavior, relevant data, dependencies, and scope. For an app adopting this workflow, identify the workflow version and applicable modules.
- **S2 — Design refinement:** Look for ambiguity, contradictions, failure cases, compatibility issues, persistence needs, and unintended consequences. Refine until the design is implementable.
- **S3 — Ready to implement:** Confirm important choices are resolved, acceptance checks are defined, the working baseline is known, and any needed user approval has been obtained.
- **S4 — Implementation:** Build a candidate from the agreed design, current maintenance knowledge, and last accepted code. Assign the candidate its own artifact version and preserve the baseline.
- **S5 — Source inspection:** Review changed files and diffs. Check data flow, architecture, security, compatibility, syntax, and available static checks.
- **S6 — Runtime and acceptance testing:** Run the candidate and perform acceptance and relevant regression tests in the actual runtime where possible. Record exact candidate version and results.
- **S7 — Acceptance gate:** Compare recorded results with the acceptance criteria. If a required check fails, move to S8. If required checks pass, proceed to S9.
- **S8 — Repair and iteration:** Compare the requirements, last accepted code, failed candidate, and observed failure. Decide whether the issue is in the design, implementation, or test assumptions. Preserve the failed candidate and evidence. Create a new version for the repair. If the design is inadequate, return to S2. Otherwise repair the candidate and return to S5.
- **S9 — Maintenance update:** Record the accepted code version, source reference, test evidence, differences from the design, and unresolved limitations. Update and independently version the maintenance document.
- **S10 — Accepted feature and next decision:** The feature is accepted for the agreed scope. Decide what to work on next or whether the app has reached its desired completion scope.
- **S11 — Milestone document:** When the user declares the app complete to the desired scope, establish the accepted code and documentation versions, create or update the independently versioned milestone document, and test whether it communicates the app well enough for reconstruction in a fresh session.

### Transition map

```text
S0 Request / idea
  -> S1 Initial design
  -> S2 Refine design
  -> S3 Ready to implement
  -> S4 Implement versioned candidate
  -> S5 Inspect source
  -> S6 Runtime + acceptance tests
  -> S7 Acceptance gate
       -> failed: S8 Preserve failure + repair as new version
            -> design flaw: S2
            -> implementation flaw: S4
            -> then S5 -> S6 -> S7
       -> passed: S9 Update maintenance document
            -> S10 Accept feature / choose next work
                 -> next feature: S0 or S1
                 -> desired completion declared: S11 Milestone document
```

### Transition requirements

- **S0 → S1:** the goal and relevant constraints are understood well enough to draft a design.
- **S1 → S2:** a draft design exists to review.
- **S2 → S3:** the design is specific enough to implement and has observable acceptance criteria.
- **S3 → S4:** the user has authorized implementation when approval is needed.
- **S4 → S5:** a candidate exists, has a clear version identity, and the prior accepted baseline remains recoverable.
- **S5 → S6:** no known blocker prevents meaningful testing, or remaining risks are explicitly understood.
- **S6 → S7:** actual test results are recorded against the exact candidate version.
- **S7 → S8:** a required check fails or a regression is found.
- **S8 → S5:** a corrected candidate with a new version is ready for inspection. Return to S2 first if the design needs correction.
- **S7 → S9:** required checks pass and behavior matches the agreed scope.
- **S9 → S10:** maintenance notes accurately describe the accepted state and exact code version.
- **S10 → S11:** the user declares the app complete to their desired scope.
- **S11 → S0/S1:** future work begins after the milestone.

## 6. Design and Implementation Rules

For a feature of meaningful complexity, the design should cover:

1. **Goal:** what problem it solves.
2. **User flow:** how the user triggers and uses it.
3. **Behavior:** expected results and how ambiguous cases resolve.
4. **Data model:** data, ownership, relationships, defaults, and persistence.
5. **State changes:** what changes and what must remain unchanged.
6. **Compatibility:** interactions with existing features, saved data, imports, exports, and earlier versions.
7. **Failure behavior:** invalid input, cancellations, missing dependencies, interruptions, and recovery.
8. **Boundaries:** what is out of scope.
9. **Acceptance checks:** observable evidence of success.
10. **Risks and alternatives:** relevant trade-offs and reasonable options.

Implementation and recovery rules:

- Identify and preserve the last accepted version before changing code.
- Assign a distinct version to each candidate artifact. Never silently overwrite the baseline or reuse a version number for changed contents.
- Prefer small, coherent changes over unrelated rewrites.
- Do not mix unrequested features into a fix.
- Keep version-control history or backups that allow recovery.
- Use actual errors and reproduction steps instead of guessing.
- Compare the requirements, accepted baseline, candidate, and observed behavior when repairing a failure.
- Correct the source of the failure. If the requirement itself is wrong or ambiguous, update the design before coding again.
- Repeat inspection and relevant tests after every repair.
- Do not delete historical versions unless the user explicitly requests it and understands the consequences.
- Separate **artifact version** from **acceptance status**. A higher version is not automatically a better or accepted version.

## 7. Evidence Labels

Use clear labels when recording project status:

- **Designed:** behavior is described but not implemented.
- **Implemented:** code exists, but testing may be incomplete.
- **Source checked:** relevant source was inspected and available static checks passed.
- **Runtime tested:** behavior was exercised in the actual runtime against named checks.
- **User-confirmed:** the user reports that it works in their environment.
- **Unverified:** adequate test evidence is not available.
- **Known issue:** a reproducible defect or unresolved limitation is recorded.
- **Candidate:** a version under evaluation that has not yet passed the acceptance gate.
- **Failed:** a candidate did not pass one or more required checks.
- **Accepted:** the version passed the required acceptance gate for its stated scope.
- **Superseded:** a newer version has replaced it for current use, while the old version remains part of history.

Do not treat these labels as interchangeable. Source inspection cannot establish every aspect of actual behavior. Features that depend on runtime conditions must be tested in the relevant runtime. Keep test evidence tied to the exact version tested.

## 8. Maintenance and Milestone Documents

### Maintenance document

After a candidate passes the acceptance gate, update and version the app's maintenance document with:

- current accepted app-code version and exact source path, commit, or other recoverable reference;
- implemented behavior and important design decisions;
- relevant architecture, data model, and compatibility notes;
- tests performed, their results, and evidence level;
- known limitations, bugs, and unverified areas;
- candidate versions that failed and concise outcomes when relevant to future work;
- next step and deferred features;
- concise history of meaningful maintenance-document changes;
- the adopted workflow version and a link to the reusable workflow.

The maintenance document describes the accepted implementation state, not an untested aspiration. Preserve useful history, but correct stale claims instead of copying them forward unchanged. Do not mark a candidate accepted until required checks pass. A maintenance update gets its own version when its contents change; it does not inherit the code version number.

### Milestone document

A milestone document is a deliberate milestone reference, not a replacement for version control and not a claim that the app can never change again.

When the user declares the app complete to the desired scope:

1. Identify the exact accepted code version and commit or equivalent source reference.
2. Bring the maintenance document up to date with actual behavior and test evidence.
3. Reconcile **the app's initial design document, the latest maintenance document, and the accepted code** into one coherent milestone specification.
4. Preserve links to the source documents and code so the origin of decisions remains traceable.
5. Distinguish implemented behavior from planned or deferred behavior. Resolve discrepancies rather than blindly copying conflicting statements.
6. Include goals, user experience, architecture, data model, important algorithms, dependencies, constraints, and acceptance tests as relevant.
7. Give the milestone document its own SemVer version, independent of app code and other documents. Identify the exact accepted versions it describes.
8. Test the milestone in a fresh LLM session without relying on hidden conversation context. Compare the resulting recreation against the accepted app using the same acceptance criteria.
9. Record important gaps and refine the milestone if necessary, using a new document version rather than silently overwriting a published version.

The reusable workflow guides this process; it is not an app-specific design source that must be merged into every app's milestone document. The milestone should record which workflow version was used where that context matters.

A successful reproduction test provides evidence that the milestone communicates the design well. It cannot guarantee identical results across models, tools, or environments.

## 9. Lightweight Use

For a small fix, the process may be brief:

`identify accepted baseline → clarify expected behavior → create versioned candidate → inspect → test → preserve evidence → update maintenance notes`

For a new subsystem, significant design change, or high-risk feature, use the full design and refinement stages.

The state machine exists to prevent important reasoning, testing, and history from being skipped. It is not meant to force unnecessary paperwork or make the user learn process terminology before they can build something. Scale the amount of documentation to the work, but never sacrifice recoverability or truthful status.

## 10. Final Checklist

Before accepting a feature:

- [ ] Intended behavior is clear and approved where needed.
- [ ] The last accepted version is recoverable and unchanged.
- [ ] The candidate has a distinct, valid SemVer version.
- [ ] Acceptance checks are defined.
- [ ] The candidate was inspected.
- [ ] Required runtime and regression tests were run against the identified version.
- [ ] Failed checks were fixed in a new version and rerun.
- [ ] Failed candidates and useful evidence remain traceable.
- [ ] Compatibility and persistence were checked where relevant.
- [ ] Maintenance documentation accurately records the accepted implementation, exact version, and evidence.
- [ ] Artifact versions are independent and their history is clear.
- [ ] The user decides whether to continue, defer work, or declare the app complete.

When revising the reusable workflow:

- [ ] The reason for the change and supporting evidence are recorded.
- [ ] The change is classified as core workflow or optional module.
- [ ] Compatibility with existing adopters is considered.
- [ ] The SemVer increment matches the impact of the change.
- [ ] The prior version remains available as historical context.

**Core invariants:** the workflow remains reusable across projects; each app's initial design identifies how it adopts the workflow; the maintenance document describes the accepted implementation; code is accepted only after required checks pass; every versioned artifact has its own independent SemVer history; failed candidates are not silently overwritten or treated as accepted; and the milestone document reconciles the app's initial design, maintenance knowledge, and accepted code.
