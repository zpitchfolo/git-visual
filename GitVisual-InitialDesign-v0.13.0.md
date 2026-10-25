# Git Visual Initial Design v0.13.0

**Artifact version:** 0.13.0  
**Status:** Draft — S2 (Design refinement); proposed defaults await review, not ready for implementation  
**Date:** 2026-10-23  
**Workflow:** Development State Machine Workflow v1.2.1 (`devstatemachine-v1.2.1.md`)  
**Evidence label:** Designed (proposed behavior only)

> This document is a design draft. It describes intended behavior and proposals; it does not claim that those features exist in the application. Important open questions remain for S2 design refinement. No implementation is authorized by this draft.

## 1. Purpose

Git Visual is a graph-first desktop interface for exploring a repository and discovering/using Git's capabilities through a visual environment. The primary navigation surface is an interactive visual rendering of the repository's **actual Git commit graph**: commits are clickable nodes, and edges come from Git's real parent relationships. Git Visual does not invent a parallel history or maintain a separate graph as the source of truth. It reads Git's existing commit and ref data and renders that data as an interactive graph. The main browsing layout has three persistent stages from left to right: **commit graph → selected commit's file selector → historical file viewer**. Selecting a commit opens or updates its file selector to the right of the graph. Clicking a file opens its historical contents in another panel to the right of that selector. The graph remains visible while browsing files, and already-open historical file panels keep their own commit/path identity while the user navigates elsewhere.

The first product goal is dependable, human-readable exploration of Git history: see how commits connect and inspect the exact files belonging to a chosen point in history. Git Visual should make these ideas easier to read visually than a command-oriented workflow. It is not initially intended to perform the full range of Git operations.

## 2. Relationship to DVA.exe

Git Visual is a Git-specific application and an experiment for selected interaction ideas in DVA.exe. It can test:

- treating a graph as a navigable visual space;
- allowing multiple panels to coexist with independent identities and context;
- opening related material without replacing an existing view; and
- separating navigation and inspection from changes to underlying data.

DVA is the broader creative-workspace concept and design influence. Git Visual is also a practical testing ground for interaction patterns that may inform DVA. Panel behavior should therefore be simple and genuinely useful for Git exploration, while keeping repository, commit, and path context explicit. This does not mean prematurely building a general-purpose workspace framework. More importantly, the product should expose Git's existing capabilities rather than inventing app-specific substitutes for concepts Git already defines. Where Git provides a native concept or operation, the interface should preserve its semantics and make the underlying effect understandable. The first increment remains a deliberately bounded, read-only slice; later Git capabilities can be surfaced in designed increments without locking the product into a history-viewer-only identity.

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

- Executing history-changing Git commands in the initial read-only increment, including creating commits, staging, checkout, reset, merge, rebase, cherry-pick, and branch mutation. These remain potential future capabilities, not permanent product non-goals.
- Editing files in a historical panel or writing to the repository.
- Remote hosting, pull requests, issue tracking, or network synchronization.
- A general-purpose editor, notes system, or DVA workspace.
- Commit diffs, commit-to-parent change summaries, blame views, and history rewriting in the initial increment. These are examples of Git capabilities that may be exposed later; each needs a focused interaction and safety design.
- Arbitrary user-configurable split layouts, floating windows, or drag-and-drop panel composition.
- Replacing Git's existing CLI or teaching every Git concept in the first release.

These are excluded from the initial scope, not necessarily from the long-term product.

## 5. Proposed initial user flow and behavior

1. The user opens one local Git repository.
2. Git Visual reads the repository's existing Git commit and ref data, then renders the actual commit DAG as an interactive graph for the agreed ref scope. It does not create an independent history model.
3. Each graph node represents a real Git commit, identified by its full commit object ID. Edges represent the commit's actual parent relationships, including merge parents. Clicking a node selects that exact commit and opens or updates its commit-specific file selector in a panel immediately to the right of the graph.
4. Clicking a file in that selector opens its historical version in another panel to the right of the selector. The panel reads the exact blob from the selected commit and retains its own repository, full commit ID, and path. The graph remains visible at the left throughout this flow.
5. Selecting another commit updates the selected-commit file selector, but does not retarget already-open file panels. Existing historical file panels stay pinned to their original commits and paths. **Every click on a file creates a new viewer entry**, even when that exact repository/commit/path is already open elsewhere. Each entry has its own panel identity, can be focused or closed independently, and initially displays the historical contents for its source commit/path.
6. The initial layout is a left-to-right sequence: graph, selected-commit file selector, then historical file panels. Multiple file panels coexist to the right of the selector, stacked vertically within their area; panel identity and order are persisted. The exact panel scrolling/focus behavior remains to be refined, but the graph and file selector must not be replaced by the file viewer.
7. On restart, Git Visual restores the last repository and browsing workspace: selected commit, graph position/scope, selected tree state, open panels, their order, and focused panel. If the repository is unavailable, the app explains the issue and keeps saved workspace information available for recovery.
8. The user can focus and close an open panel. Drag-to-rearrange and split layouts are deferred.
9. Missing, unreadable, binary, or otherwise unsupported content receives an explicit state rather than silently displaying current working-tree content or stale data.

The first version browses committed snapshots from local repositories. Workspace restoration preserves browsing context, not repository working-tree state or unsaved edits.
## 6. Initial feature scope

### Included in the proposed first increment

- Open one local repository at a time (proposal; confirm whether multiple simultaneous repositories are needed).
- An interactive visual rendering of Git's actual commit DAG: clickable commit nodes, real parent edges (including merge parents), and basic commit/ref metadata. Git remains the source of truth; the graph is a view of repository data, not a separate history model.
- Commit selection and a repository-relative file tree for that commit.
- A persistent three-stage left-to-right browsing layout: commit graph, selected-commit file selector, and historical file viewer panels. Opening a commit's file selector or a file viewer does not replace the graph.
- Restore repository and browsing workspace context after application restart.
- Read-only historical text-file viewer.
- Multiple open historical file panels with independent context. Every file-open action creates a new viewer entry, including repeated opens of the same repository/commit/path; entries are never silently deduplicated.
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

**Identity rule — decided:** each file-open action creates a new viewer-entry identity, even if another entry already displays the same repository, commit, and path. A viewer entry's source identity (repository + full commit ID + path) is separate from its unique panel-instance identity. Entries may therefore show the same historical source more than once, and each can be focused or closed independently. Entries opened from different commits also remain independent and pinned to their own source.

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
| Large histories make the graph slow or unreadable | Render the graph in stages while preserving the real commit/parent relationships. Define an initial batch and a clear way to load older history; use graph virtualization where needed. Validate performance with a representative repository before acceptance. |
| Git operations accidentally mutate user data | Use read-only backend APIs/commands and tests that verify HEAD, index, refs, and working tree are unchanged by browsing. |
| Repository path handling exposes data outside the chosen repository | Canonicalize and validate repository-relative paths in the native layer; test traversal and symlink edge cases. |
| Panel abstraction expands into a general workspace framework | Use the three-stage browsing layout to test practical panel interactions that also inform DVA, but avoid generalized workspace infrastructure without demonstrated need. | 
| UI invents substitutes for Git concepts | Prefer exposing native Git concepts and operations with their real semantics. Distinguish Git refs and object metadata from temporary UI labels; stage capabilities deliberately rather than making up parallel concepts. |
| Unsupported file types create misleading output | Define text/binary and file-size policy; show explicit unsupported states. |
| Different Git implementations produce different edge-case behavior | Select one repository access approach and include merge, deleted-file, and unusual-path fixtures in acceptance testing. |

## 11. Draft acceptance criteria

These are observable product checks for a future candidate. They do not indicate that anything has been implemented or tested. Refine them in S2 and tie actual evidence to a version during implementation/testing.

- **AC-1 — Interactive Git commit graph:** Given a local repository with a linear history, multiple local branches, and a merge, Git Visual reads Git's existing commit/ref data and renders the union of commits reachable from all local branch refs. Each real commit appears once, relevant branch tips are labeled, and edges follow the actual parent relationships, including merge parents. Tags and remote-tracking refs are available through optional filters and are not included by default. Clicking a node selects that exact Git commit and shows its full commit ID and metadata. No independent or synthetic history is used as the source of truth.
- **AC-2 — Exact snapshot tree:** Selecting commit A shows the paths in A's tree. A file added only in commit B is absent in A, and a file deleted in B remains present when A is selected.
- **AC-3 — Historical content:** Opening a text file from commit A displays the contents stored in A, even if the working tree has different uncommitted contents.
- **AC-4 — Stable panel context:** Open `example.ts` from A, select B, and open `example.ts` from B. Both panels remain open, identify their own commit, and show the corresponding content. Changing graph selection does not change either panel.
- **AC-5 — Repeat open behavior:** Every file-open action creates a new viewer entry, even when the repository, full commit ID, and path match an already-open entry. Repeated entries have distinct panel-instance IDs but the same source identity; entries opened from different commits show their respective historical versions.
- **AC-12 — Native Git semantics:** Git Visual's displayed concepts and operations correspond to real Git concepts and behavior. The interface must not conflate commit messages, tags, branches, or temporary UI labels. Any future Git operation must identify what repository state it reads or changes, and destructive or history-changing actions require an explicit design and safety decision before implementation.
- **AC-6 — Read-only guarantee:** Browsing commits and files leaves HEAD, refs, index, working-tree files, and repository objects unchanged.
- **AC-7 — Unsupported and error states:** Missing repository data and unsupported file types are shown explicitly; stale content from a previous selection is never presented as though it belongs to the current selection.
- **AC-8 — Panel lifecycle:** The user can focus and close an open panel without changing its source identity or changing other open panels.
- **AC-9 — Path safety:** Requests for paths outside the opened repository are rejected by the backend and cannot read arbitrary local files.
- **AC-10 — Three-stage navigation:** The commit graph stays visible on the left. Clicking a commit opens or updates its file selector in a separate area immediately to the right of the graph. Clicking a file opens its historical contents in another panel to the right of the file selector. The file viewer does not replace the graph or selector, and existing historical file panels from other commits remain open with their original source identity.
- **AC-11 — Restart restoration:** After closing and relaunching, the app restores the last repository and agreed navigation/panel state. Each restored panel still displays the exact saved commit/path; if data is unavailable, the app shows an explicit recovery state rather than silently switching context.
## 12. Open questions for S2 design refinement

### 12.1 Graph scope: what the graph includes

A Git repository can have many named references (refs): the current branch, other local branches, tags, and remote-tracking branches. The selected scope determines which commits appear as graph roots and how much history is visible.

**Decision: the initial graph shows all local branches by default.** Include the union of commits reachable from local branch refs, deduplicating commits by full object ID and labeling the relevant branch tips. This makes local divergence and merges visible without automatically mixing in potentially stale remote-tracking refs or every tag.

Tags and remote-tracking refs remain available through optional graph filters rather than being shown by default. A commit is drawn once even if multiple refs reach it; the relevant ref labels decorate the commit, while parent edges always reflect the actual Git commit relationships. Current-branch-only view may be considered as a later filter, but it is not the default initial scope.

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
| Graph controls and rendering | Git provides the underlying commit DAG and refs; Git Visual renders them as clickable nodes connected by real parent edges. Pan moves around a large graph; zoom changes the visible scale. Filters control which refs are displayed. Very large histories may need progressive rendering or viewport-based loading, but any displayed node must still represent a real Git commit. | **Decision:** support basic pan and zoom in the initial graph. Use optional filters for branch/ref visibility: local branches are included by default, while tags and remote-tracking refs can be enabled when needed. **Decision: render history in stages**, loading additional older history when requested or needed. Staging is a rendering/loading strategy only; it must preserve real commit IDs, parent relationships, and visible branch/merge context at the boundary between loaded and unloaded history. Exact batch size and trigger remain implementation details to validate with representative repositories. |
| Arrangement of file panels | With several files open, they could be tiled side by side, stacked vertically, or one shown large while the others remain in a list. | Keep the three-stage flow: graph, selected-commit file selector, then file viewer area. Stack independent file panels vertically in the viewer area and let the focused panel scroll into view. The graph and selector remain visible; avoid a full arbitrary split-layout system in the first increment. |
| Reopening the same file/version | Repeatedly opening the same path from the same commit could deduplicate to an existing panel or create another viewer entry. | **Create a new entry every time.** Separate unique panel-instance identity from source identity (repository + full commit ID + path). Repeated entries may show identical contents but can be focused and closed independently. This intentionally supports workspace experimentation and tests for DVA-style panel behavior. |
| What “restore where I left off” includes | Workspace state could include the selected commit, graph position, expanded folders, open panels, their order, focused panel, and scroll position inside each file. | Restore all of those view choices. Store only navigation/view state; do not copy or change repository files. If a repo moved or an old object is unavailable, explain that and keep the rest of the saved state. |
| How Git is accessed | The app can call the installed Git program, or include a Git-reading library inside the app. Calling installed Git reuses the user's existing Git setup but requires Git to be installed. A library removes that requirement but adds a dependency and needs compatibility checks. | Decide after a small repository-reading prototype. Whichever route is chosen, use read-only operations and verify that displayed commit IDs and file contents come from the selected repository. |
| Local history versus uncommitted work | A commit is a saved point in history. The working tree is the current folder, which may include changes that have not been committed. Mixing them can make an old commit look like it contains newer edits. | The first increment reads committed history only. Show working-tree changes later as a clearly separate source if you want them. |
| Testing evidence | Acceptance checks are examples that prove the design works, such as opening the same path at two commits and confirming each panel shows the right version. | Keep acceptance checks in `GitVisual-InitialDesign-v0.13.0.md`; keep actual test results and known issues in `GitVisual-Maintenance-vX.Y.Z.md`. Put executable tests with the code and avoid a separate test-plan document unless the project grows enough to need one. |

Graph-scope alternatives are explained in §12.1; file encodings, size limits, binaries, symlinks, submodules, and LFS pointers are explained in §12.2.
Resolve these before calling the design ready for implementation:

1. **Repository scope:** Local repositories only for the first increment? One open repository at a time or several?
2. **Graph scope choice — decided:** Show all local branches by default. Tags and remote-tracking refs are available through optional filters, not shown by default.
3. **Graph source and interaction — decided:** Render Git's actual commit DAG and refs as clickable commit nodes connected by real parent edges. Git repository data is the source of truth; Git Visual does not maintain a separate or synthetic history graph.
4. **Graph behavior — initial controls decided:** Support basic pan and zoom. Provide optional ref filters so users can enable or hide tags and remote-tracking refs; all local branches remain included by default. Render history in stages rather than requiring the entire included history to be rendered immediately. Additional older history should load when requested or needed, without inventing or altering commit/parent relationships. Exact loading trigger, batch size, and treatment of the visible boundary remain implementation details to validate.
4. **Panel arrangement — primary flow decided:** Keep the graph visible on the left; clicking a commit opens its file selector to the right; clicking a file opens its viewer to the right of the selector. Multiple file viewers remain independent and stacked vertically in their area. Remaining details are sizing, scrolling, and focus behavior.
5. **Panel identity — repeat-open behavior decided:** Every click on a file creates a new viewer entry, even for an already-open identical repository/commit/path. Panel-instance identity is distinct from source identity. A separate custom title system is not assumed; use Git's native concepts where appropriate and only add UI-only labels when a concrete need is established.
6. **Git capability surface:** The long-term product should make Git's existing capabilities discoverable and usable through the interface. The first increment remains read-only; later operations need focused UX, accurate Git semantics, and explicit safety decisions rather than being ruled out as permanent non-goals.
6. **Restart restoration details:** Which view details should be restored besides repository, selected commit, graph viewport, open panel identities/order, and focused panel (for example scroll positions or tree expansion)?
7. **File support:** Review the proposed first-version policy in §12.2: text encodings, size limit, binary files, symlinks, submodules, and Git LFS pointers.
8. **Repository access:** Is Git required to be installed, or should the app bundle/access Git internally? Prefer Git CLI or a Rust Git library?
9. **Security boundary:** Should the app be allowed to read only the selected repository, or are there other local paths it needs? How should symlinks that point outside the repository be handled?
10. **Maintenance and tests:** Where should test plans and test evidence live alongside the maintenance record? The repository currently has no visible test files outside generated/build directories.
11. **Product framing:** Should the initial app expose only history browsing, or include a read-only summary of working-tree changes as a clearly separate view?
## 13. Workflow adoption and document map

Git Visual adopts the **Development State Machine Workflow v1.2.1**, maintained separately in `devstatemachine-v1.2.1.md`. This initial design is the app-specific design artifact; it does not replace or modify the reusable workflow.

- Project overview: root `README.md`, kept at this stable filename for GitHub's landing-page behavior. It is an index/summary, not a versioned sibling document.
- Initial design: `GitVisual-InitialDesign-v0.13.0.md` (this draft). Its filename, title, and document body carry its independent SemVer version.
- Maintenance document: `GitVisual-Maintenance-vX.Y.Z.md` in the same repository-root location as the initial design. Name each versioned project document with its document type and SemVer, for example `GitVisual-Maintenance-v1.0.0.md`. The filename, internal title, and document-version field must agree. Keep one current file per document type. On revision, increment SemVer and rename/replace the current file; use Git history to compare or recover previous contents. Do not keep superseded document copies in the working tree unless side-by-side versions are specifically useful.
- Project document convention: every versioned project document filename includes its own SemVer (for example, `GitVisual-InitialDesign-v0.13.0.md`), and the internal title and document-version field match. Keep one current file per document type by default, incrementing SemVer and renaming the file on revision. Git history provides the prior versions. The reusable workflow remains a separately versioned artifact.
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

## 15. Version history

The current document version is 0.13.0. This revision clarifies the product direction: Git Visual should expose Git's existing concepts and capabilities through a visual interface, preserving native Git semantics rather than inventing substitutes. The first increment remains read-only, while future capabilities are not excluded permanently. It also records that the app is a practical testing ground for panel interactions that may inform DVA. Every file-open action continues to create a new viewer entry, with a unique panel-instance identity separate from repository/commit/path source identity. Git history preserves prior document revisions; superseded files are not kept alongside the current file by default.











