# Git Visual README v0.1.3

**Document version:** 0.1.3  
**Project status:** Design draft; Git history features are not implemented yet.

Git Visual is planned as a graph-first desktop Git client for people who want to explore repository history and inspect historical files more easily.

The proposed first experience is a commit graph and selected-commit file tree on the left, with independent historical file panels on the right. A file panel should stay tied to the exact commit where it was opened. Selecting another commit should not silently change files already open in the right-side workspace.

The current application is still the Tauri + React + TypeScript starter scaffold. The design is being refined before implementation.

## Project documents

- [Initial design](GitVisual-InitialDesign-v0.6.0.md) — v0.6.0 draft; goals, scope, behavior, open decisions, and proposed acceptance criteria.
- [Development workflow](devstatemachine-v1.2.1.md) — reusable workflow v1.2.1; governs design, implementation, testing, and acceptance.
- `GitVisual-Maintenance-vX.Y.Z.md` — planned at the repository root; it will record accepted implementation versions, test evidence, and known limitations after implementation begins.

Each versioned project document filename includes its independent SemVer (for example, `GitVisual-README-v0.1.3.md`). The filename, internal title, and document-version field must agree. Keep each version as a separate file so the version is visible in the repository listing; Git history remains available for provenance and diffs. When developing the app, source diffs against the last accepted commit are part of our verification workflow, not an app feature.


