# Git Visual Initial Design v0.6.0

**Artifact version:** 0.6.0  
**Status:** Draft — S2 (Design refinement); proposed defaults await review, not ready for implementation  
**Date:** 2026-10-23  
**Workflow:** Development State Machine Workflow v1.2.1 (`devstatemachine-v1.2.1.md`)  
**Evidence label:** Designed (proposed behavior only)

> This document is a design draft. It describes intended behavior and proposals; it does not claim that those features exist in the application. Important open questions remain for S2 design refinement. No implementation is authorized by this draft.

## 1. Purpose

Git Visual is a graph-first desktop Git client for exploring and understanding a repository's history. The commit graph is the primary navigation surface. Selecting a commit establishes a view of that commit's snapshot; opening a file creates a historical file panel whose identity and contents remain tied to that exact commit and path while the user navigates elsewhere.

The first product goal is dependable, human-readable exploration of Git history: see how commits connect and inspect the exact files belonging to a chosen point in history. Git Visual should make these ideas easier to read visually than a command-oriented workflow. It is not initially intended to perform the full range of Git operations.

## 2. Relationship to DVA.exe

Git Visual is a Git-specific application and an experiment for selected interaction ideas in DVA.exe. It can test:

- treating a graph as a navigable visual space;
- allowing multiple panels to coexist with independent identities and context;
- opening related material without replacing an existing view; and
- separating navigation and inspection from changes to underlying data.

DVA is the broader creative-workspace concept and design influence. Git Visual should not become a general-purpose writing workspace or adopt generalized workspace features without a Git-specific need. Its first panel system should solve historical-file browsing, while keeping the panel's repository, commit, and path context explicit. Flexible arrangement can be introduced in later, separately designed increments.

**Risk to manage:** treating “panel” as an invitation to build a broad layout/workspace framework before the Git exploration flow is proven. The first increment should establish panel identity, persistence of context, and accurate display of commit and file history; layout flexibility can follow when concrete Git use cases justify it.

## 3. Goals

1. Make the commit DAG the main way to navigate history.
2. Make the selected commit and its snapshot unambiguous.
3. Browse the files belonging to that exact committed snapshot.
4. Open a historical file into a persistent panel identified by repository, commit, and path.
5. Allow a user to change graph selection and open another version of a file without changing an already-open panel.
6. Keep initial behavior focused on reading and understanding history, with clear handling of unavailable or unsupported content.
7. Restore the repository and browsing workspace after application restart.
8. Preserve a small architecture boundary between the UI and Git data access so repository behavior can be tested independently.

## 4. Non-goals for the initial scope

- Creating commits, staging, checkout, reset, merge, rebase, cherry-pick, branch mutation, or other history-changing commands.
- Editing files in a historical panel or writing to the repository.
- Remote hosting, pull requests, issue tracking, or network synchronization.
- A general-purpose editor, notes system, or DVA workspace.
- Commit diffs, commit-to-parent change summaries, blame views, and history rewriting. They are not part of the initial scope; any future diff feature needs its own design decision.
- Arbitrary user-configurable split layouts, floating windows, or drag-and-drop panel composition.
- Replacing Git's existing CLI or teaching every Git concept in the first release.

These are excluded from the initial scope, not necessarily from the long-term product.

## 5. Proposed initial user flow and behavior

1. The user opens one local Git repository.
2. Git Visual loads the commit graph for the agreed ref scope, showing commits as nodes and parent relationships as edges.
3. The left navigation area contains the graph and the selected commit's file tree. Clicking a commit selects that exact snapshot and lists the files that belong to it.
4. Clicking a file opens its historical version in the right-side file workspace. The panel reads the exact blob from the selected commit and retains its own repository, full commit ID, and path.
5. Selecting another commit updates the graph selection and file tree. Existing right-side panels stay pinned to their original commits. Opening that same path at the new commit creates or focuses a distinct version panel.
6. Multiple file panels coexist on the right. The initial layout fixes navigation on the left and file panels on the right; panel identity and order are persisted. The exact tiling within the right area remains an S2 question.
7. On restart, Git Visual restores the last repository and browsing workspace: selected commit, graph position/scope, selected tree state, open panels, their order, and focused panel. If the repository is unavailable, the app explains the issue and keeps saved workspace information available for recovery.
8. The user can focus and close an open panel. Drag-to-rearrange and split layouts are deferred.
9. Missing, unreadable, binary, or otherwise unsupported content receives an explicit state rather than silently displaying current working-tree content or stale data.

The first version browses committed snapshots from local repositories. Workspace restoration preserves browsing context, not repository working-tree state or unsaved edits.
## 6. Initial feature scope

### Included in the proposed first increment

- Open one local repository at a time (proposal; confirm whether multiple simultaneous repositories are needed).
- Read-only commit graph with parent links and basic commit metadata.
- Commit selection and a repository-relative file tree for that commit.
- A fixed left navigation area (commit graph and selected-commit file tree) and right historical-file panel workspace.
- Restore repository and browsing workspace context after application restart.
- Read-only historical text-file viewer.
- Multiple open historical file panels with independent context.
- Explicit loading, empty, unsupported-content, and error states.
- Repository-scoped Git access through the Tauri backend.

### Deferred pending design decisions or later increments

- Branch/tag/ref filters and graph search.
- Commit diffs, blame, and line history; any future addition needs a separate design decision.
- Restoring uncommitted working-tree state or unsaved edits (historical panels are read-only).
- Panel splitting, arbitrary rearrangement, and saved layouts.
- Binary previews and specialized viewers.
- Remote repositories and hosting integrations.

## 7. Proposed architecture

This is a proposal based on the existing scaffold, not an implemented architecture.

### Frontend

- Keep React as the UI layer and organize the first history view into repository selection, commit graph, selected-commit summary/file tree, and historical-file panel collection.
- Keep the selected commit as navigation state separate from the identity and source of each open panel.
- Represent panel state as data rather than deriving panel content from whichever commit is currently selected.
- Use stable keys for commits (full object ID) and panels (a panel ID plus their immutable source identity).

### Native/data boundary

- Use Tauri commands as a narrow bridge from React to a Rust repository service.
- Define an explicit repository service interface for opening a repository and reading graph, commit, tree, and blob data.
- Treat all initial operations as read-only. The backend should not change HEAD, index, refs, configuration, working-tree files, or repository objects.
- Validate that requested paths are repository-relative and resolve within the opened repository. Do not accept arbitrary shell fragments or invoke commands through a shell.
- Choose the Git access implementation during S2. Candidates include invoking Git with argument-safe process APIs or using a Rust Git library. The repository has no Git library dependency today.

### UI state separation

At minimum, model these separately:

- repository currently open;
- graph query, loading state, and viewport;
- currently selected commit for browsing;
- file tree for that selected commit;
- open historical panels, each with its own source identity, content/load state, and panel identity;
- focused panel and panel order/layout.

Do not let `selectedCommitId` be the source of truth for an already-open panel's content.

## 8. Core data and context model (proposed)

- **RepositoryRef**: stable session identity, canonical local repository root, and display name.
- **CommitId**: full Git object ID, never a branch name or abbreviated hash as the panel's source identity.
- **CommitRecord**: commit ID, zero or more parent IDs, author/committer metadata, timestamp, and subject/message needed by the initial graph and detail view.
- **TreeEntry**: repository-relative path, entry kind, and object ID as resolved from one commit's tree. Directory rows may be derived from entries.
- **HistoricalFileSource**: repository identity + full commit ID + repository-relative path + object ID where available. The blob ID can verify exactly which stored content is read.
- **HistoricalFilePanel**: stable panel ID + immutable HistoricalFileSource + display/load state. Any cached contents must be keyed by source identity, never only by path.
- **ExplorerState**: current RepositoryRef, graph state, selected commit, and selected-commit tree state.
- **PanelWorkspaceState**: open panels, focused panel ID, order, and any initial fixed-layout state.
- **PersistedWorkspace**: versioned local app state sufficient to restore repository identity/path, selected commit, graph viewport/ref scope, selected tree path, open panel sources/order/focus, and other agreed view context. It stores pointers to committed Git objects, not copies that can drift from the repository.

**Identity rule:** two versions of the same path at different commits are distinct historical sources and must remain independently viewable. A proposed behavior is to focus an existing panel when the same repository/commit/path is opened again, and create a new panel when the commit differs. Confirm this deduplication rule during S2.

## 9. Read-only, compatibility, and failure behavior

- History browsing must not require checking out a commit or changing repository state.
- Dirty or untracked working-tree contents must not leak into the selected-commit file browser or historical panels.
- A repository that cannot be opened should produce a recoverable error and allow the user to choose another repository.
- A commit or tree that cannot be read should show a useful error and must not leave the UI presenting content from the previous selection as if it belonged to the new one.
- A file deleted from a later commit remains readable from an earlier commit where it exists.
- Binary files, symlinks, submodules, very large files, malformed repositories, and missing Git dependencies need explicit policy before implementation; until resolved, avoid silently treating them as ordinary text.
- Long-running graph/tree reads should have loading and cancellation/obsolete-request handling so a slow earlier selection cannot overwrite a newer one.
- Saved workspace state should be validated on restore. If the repository path is missing or a saved object cannot be read, show a recoverable state; never substitute the current branch or working-tree contents silently.

## 10. Risks and mitigations to refine

| Risk | Initial mitigation proposal |
|---|---|
| A panel silently follows current graph selection | Store and render each panel from its immutable repository/commit/path source. Add an acceptance scenario that opens two historical versions of one path. |
| Working-tree files are mistaken for committed content | Resolve file contents from the selected commit tree/blob, and label the commit context in the viewer. |
| Large histories make the graph slow or unreadable | Decide initial ref scope and pagination/virtualization strategy in S2; measure with a representative repository before accepting graph behavior. |
| Git operations accidentally mutate user data | Use read-only backend APIs/commands and tests that verify HEAD, index, refs, and working tree are unchanged by browsing. |
| Repository path handling exposes data outside the chosen repository | Canonicalize and validate repository-relative paths in the native layer; test traversal and symlink edge cases. |
| Panel abstraction expands into a general workspace framework | Keep panel scope specific to historical Git files; use the agreed fixed left-navigation/right-panel layout and persist only the browsing state needed to restore it. |
| Unsupported file types create misleading output | Define text/binary and file-size policy; show explicit unsupported states. |
| Different Git implementations produce different edge-case behavior | Select one repository access approach and include merge, deleted-file, and unusual-path fixtures in acceptance testing. |

## 11. Draft acceptance criteria

These are observable product checks for a future candidate. They do not indicate that anything has been implemented or tested. Refine them in S2 and tie actual evidence to a version during implementation/testing.

- **AC-1 — Commit graph:** Given a local repository with a linear history, a branch, and a merge, the graph displays the included commits and their parent relationships; selecting a node identifies the full commit ID and associated metadata.
- **AC-2 — Exact snapshot tree:** Selecting commit A shows the paths in A's tree. A file added only in commit B is absent in A, and a file deleted in B remains present when A is selected.
- **AC-3 — Historical content:** Opening a text file from commit A displays the contents stored in A, even if the working tree has different uncommitted contents.
- **AC-4 — Stable panel context:** Open `example.ts` from A, select B, and open `example.ts` from B. Both panels remain open, identify their own commit, and show the corresponding content. Changing graph selection does not change either panel.
- **AC-5 — Repeat open behavior:** Reopening the same repository/commit/path follows the agreed deduplication rule; opening the same path at a different commit produces a separate source view.
- **AC-6 — Read-only guarantee:** Browsing commits and files leaves HEAD, refs, index, working-tree files, and repository objects unchanged.
- **AC-7 — Unsupported and error states:** Missing repository data and unsupported file types are shown explicitly; stale content from a previous selection is never presented as though it belongs to the current selection.
- **AC-8 — Panel lifecycle:** The user can focus and close an open panel without changing its source identity or changing other open panels.
- **AC-9 — Path safety:** Requests for paths outside the opened repository are rejected by the backend and cannot read arbitrary local files.
- **AC-10 — Two-area navigation:** The left area shows the commit graph and the selected commit's files. Selecting a commit changes the listed files; opening a file adds/focuses its historical panel on the right without replacing panels from other commits.
- **AC-11 — Restart restoration:** After closing and relaunching, the app restores the last repository and agreed navigation/panel state. Each restored panel still displays the exact saved commit/path; if data is unavailable, the app shows an explicit recovery state rather than silently switching context.
## 12. Open questions for S2 design refinement

### 12.1 Graph scope: what the graph includes

A Git repository can have many named references (refs): the current branch, other local branches, tags, and remote-tracking branches. The selected scope determines which commits appear as graph roots and how much history is visible.

- **Current branch only — simplest view:** show commits reachable from the currently checked-out branch/HEAD. This is less crowded and has a natural starting point, but hides other local branches and their divergence until the user changes scope.
- **All local branches — proposed starting point:** show the union of commits reachable from local branch refs, deduplicating commits by full object ID and labeling the branch tips. This reveals local branching and merges without including stale or numerous remote-tracking refs. It can produce a larger graph.
- **All refs (local branches, tags, and remote-tracking branches) — broadest view:** exposes more of the repository, but may be noisy; remote-tracking refs can be stale and tags can add many labels. This can be added through filters or later increments.

A commit is shown once even if multiple refs reach it; ref labels decorate the relevant tip(s). The actual parent links remain Git's commit relationships. **Recommendation:** start with all local branches and add a simple scope/filter control if a repository becomes crowded. The user can instead choose current branch only for the first increment.

### 12.2 File support: what can be read safely and clearly

Git stores file contents as bytes. A viewer needs a policy for deciding which blobs it can render as text without changing or garbling their contents.

- **Text encoding:** propose UTF-8 (with or without a UTF-8 BOM) for the first increment. ASCII is naturally covered. For invalid UTF-8 or files that appear binary, show an explicit unsupported/binary state rather than guessing a Windows code page and silently changing characters. Supporting UTF-16 or user-selected encodings can follow if needed.
- **Size:** propose a 1 MiB preview limit initially. Above it, show metadata and an explicit size-limit message; do not silently truncate and present incomplete content as complete. The limit can be raised after performance testing.
- **Binary files:** show path, size, and blob identity where available, but do not pretend to preview the content. Image/media previews are a later feature.
- **Symlinks:** Git stores a symlink's target as a blob. Display the target as text and do not follow it to read outside the repository.
- **Submodules:** show the submodule entry and referenced commit ID; do not recursively open another repository in the first increment.
- **Git LFS:** a Git tree may contain an LFS pointer rather than the large file bytes. Show that pointer/metadata and do not fetch data from a remote in the read-only local-history scope.

The same rules apply to historical panels. The selected commit remains the source of truth; no working-tree filters or external network fetches should change what the user sees.

### 12.3 Other design choices in plain language

The remaining questions are product choices, not things you need to know Git internals to answer. These are proposed defaults; they can be changed during design refinement.

| Choice | What it means | Recommended starting point |
|---|---|---|
| One repository or several | A repository is one local project folder with its own Git history. Supporting several at once means deciding how to switch between independent project workspaces. | Open one repository at a time. Remember it on restart; add multi-repository workspaces later if needed. |
| Graph controls | Pan moves around a large graph; zoom changes the amount of history visible. Filtering hides branches or commits that are not relevant. Loading older commits on demand avoids drawing everything at once. | Support pan/zoom and load history as needed. Defer search and advanced filters until the basic graph is useful. |
| Arrangement of right-side panels | With several files open, they could be tiled side by side, stacked vertically, or one shown large while the others remain in a list. | Stack independent panels vertically in the right area; let the focused panel scroll into view. This keeps each panel present without designing a full split-layout system. |
| Reopening the same file/version | Clicking the same file from the same commit again could create a duplicate panel or return to the panel already open for that exact version. | Focus the existing panel for the same repository + commit + path. The same path at another commit gets a separate panel. |
| What “restore where I left off” includes | Workspace state could include the selected commit, graph position, expanded folders, open panels, their order, focused panel, and scroll position inside each file. | Restore all of those view choices. Store only navigation/view state; do not copy or change repository files. If a repo moved or an old object is unavailable, explain that and keep the rest of the saved state. |
| How Git is accessed | The app can call the installed Git program, or include a Git-reading library inside the app. Calling installed Git reuses the user's existing Git setup but requires Git to be installed. A library removes that requirement but adds a dependency and needs compatibility checks. | Decide after a small repository-reading prototype. Whichever route is chosen, use read-only operations and verify that displayed commit IDs and file contents come from the selected repository. |
| Local history versus uncommitted work | A commit is a saved point in history. The working tree is the current folder, which may include changes that have not been committed. Mixing them can make an old commit look like it contains newer edits. | The first increment reads committed history only. Show working-tree changes later as a clearly separate source if you want them. |
| Testing evidence | Acceptance checks are examples that prove the design works, such as opening the same path at two commits and confirming each panel shows the right version. | Keep acceptance checks in `GitVisual-InitialDesign-v0.6.0.md`; keep actual test results and known issues in `GitVisual-Maintenance-vX.Y.Z.md`. Put executable tests with the code and avoid a separate test-plan document unless the project grows enough to need one. |

Graph-scope alternatives are explained in §12.1; file encodings, size limits, binaries, symlinks, submodules, and LFS pointers are explained in §12.2.
Resolve these before calling the design ready for implementation:

1. **Repository scope:** Local repositories only for the first increment? One open repository at a time or several?
2. **Graph scope choice:** Use the recommended all-local-branches scope, or current branch only? Should remote-tracking refs/tags be hidden until explicitly enabled?
3. **Graph behavior:** Which zoom/pan, filtering, and older-commit loading behaviors are required initially?
4. **Panel arrangement:** The left/right layout is settled. Within the right workspace, should multiple open file panels be tiled side by side, stacked vertically, or shown one focused panel at a time while the others remain persisted in a panel list?
5. **Panel identity:** Should opening the same repository/commit/path focus an existing panel or create another instance? Should panels have user-editable titles?
6. **Restart restoration details:** Which view details should be restored besides repository, selected commit, graph viewport, open panel identities/order, and focused panel (for example scroll positions or tree expansion)?
7. **File support:** Review the proposed first-version policy in §12.2: text encodings, size limit, binary files, symlinks, submodules, and Git LFS pointers.
8. **Repository access:** Is Git required to be installed, or should the app bundle/access Git internally? Prefer Git CLI or a Rust Git library?
9. **Security boundary:** Should the app be allowed to read only the selected repository, or are there other local paths it needs? How should symlinks that point outside the repository be handled?
10. **Maintenance and tests:** Where should test plans and test evidence live alongside the maintenance record? The repository currently has no visible test files outside generated/build directories.
11. **Product framing:** Should the initial app expose only history browsing, or include a read-only summary of working-tree changes as a clearly separate view?
## 13. Workflow adoption and document map

Git Visual adopts the **Development State Machine Workflow v1.2.1**, maintained separately in `devstatemachine-v1.2.1.md`. This initial design is the app-specific design artifact; it does not replace or modify the reusable workflow.

- Project overview: `GitVisual-README-v0.1.3.md`, currently `Git Visual README v0.1.3`.
- Initial design: `GitVisual-InitialDesign-v0.6.0.md` (this draft). Its filename, title, and document body carry its independent SemVer version.
- Maintenance document: `GitVisual-Maintenance-vX.Y.Z.md` in the same repository-root location as the initial design. Name each versioned project document with its document type and SemVer, for example `GitVisual-Maintenance-v1.0.0.md`. The filename, internal title, and document-version field must agree. Keep one current file per document type. On revision, increment SemVer and rename/replace the current file; use Git history to compare or recover previous contents. Do not keep superseded document copies in the working tree unless side-by-side versions are specifically useful.
- Project document convention: every versioned project document filename includes its own SemVer (for example, `GitVisual-InitialDesign-v0.6.0.md`), and the internal title and document-version field match. Keep one current file per document type by default, incrementing SemVer and renaming the file on revision. Git history provides the prior versions. The reusable workflow remains a separately versioned artifact.
- Application code: existing frontend under `src/` and Tauri/Rust backend under `src-tauri/`; future code candidates should receive their own independent SemVer identity and acceptance status.
- Tests and evidence: no project test suite was found during this inspection; future test locations and evidence records need agreement during design refinement.
- Development verification: for each code candidate, inspect its Git diff against the last accepted code and verify the result against the repository history and agreed acceptance criteria. This is our development check, not an in-app comparison feature.
- Milestone document: none yet; create only when the user declares the desired scope complete, following workflow v1.2.1.

No optional reusable workflow module is copied into this repository. The app-specific design requires repository read-only safety, exact snapshot identity, and versioned workspace persistence; their implementation and migration details must be designed before coding.
## 14. Verified repository baseline at draft time

The following are observations from the repository, not proposed features:

- The frontend scaffold uses React, TypeScript, and Vite; `src/App.tsx` displays the default Tauri greeting form and invokes a Rust `greet` command.
- The native scaffold uses Tauri 2 and Rust. `src-tauri/src/lib.rs` registers the greeting command and the Tauri opener plugin.
- `package.json` contains Vite dev/build/preview scripts and the Tauri CLI script. Its declared frontend dependencies contain no graph-rendering or Git library.
- `src-tauri/Cargo.toml` contains Tauri, the opener plugin, Serde, and Serde JSON; it declares no Git library.
- `README.md` remains the GitHub landing page and links to the versioned project overview.
- `devstatemachine-v1.2.1.md` contains Reusable Workflow Prompt v1.2.1 and specifies independent SemVer histories, design/refinement/approval gates, candidate preservation, source inspection, runtime acceptance evidence, and maintenance updates.
- The inspected repository root had no initial design, maintenance document, or visible frontend/backend test files. Generated Tauri schema files and build output exist; they are not treated as application features.
- Git reported the checked-out branch as `main`, tracking `origin/main`, with no short-status changes at inspection time. Git required a command-scoped `safe.directory` setting because the repository owner differs from the sandbox user; no global Git configuration was changed.

## 15. Draft history

Each meaningful revision increments this document's independent SemVer and renames/replaces the current filename. Git history preserves previous contents.

| Version | Status | Change |
|---|---|---|
| 0.1.0 | Draft / superseded | Initial S1 draft describing graph navigation, exact snapshots, and persistent historical-file panels; superseded during the move to stable document filenames and not separately committed. |
| 0.2.0 | Draft / superseded | Established stable document paths, in-file SemVer, and Git history as the record of prior committed revisions; an in-app parent-comparison idea was only a draft proposal. |
| 0.3.0 | Draft / superseded | Expanded the left navigation/right panel workspace, restart restoration, and graph-scope/file-support proposals; included an in-app comparison idea later rejected by the user. |
| 0.4.0 | Draft / superseded | Added plain-language explanations and recommended defaults for remaining design questions; established the SemVer-in-title convention. |
| 0.5.0 | Draft / superseded | Applied the title convention to the README and documented the project-document versioning rule. |
| 0.5.1 | Draft / superseded | Corrected a scope misunderstanding: Git diff review is a development verification step, not an initial Git Visual feature. |
| 0.6.0 | Draft | Clarified that SemVer must appear in project document filenames as well as internal titles; each version remains a separate file. |











