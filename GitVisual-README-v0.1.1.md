# Git Visual README v0.1.1

**Document version:** 0.1.1  
**Project status:** Design draft; Git history features are not implemented yet.

Git Visual is planned as a graph-first desktop Git client for people who want to explore repository history and inspect historical files more easily.

The proposed first experience is a commit graph and selected-commit file tree on the left, with independent historical file panels on the right. A file panel should stay tied to the exact commit where it was opened. Selecting another commit should not silently change files already open in the right-side workspace.

The current application is still the Tauri + React + TypeScript starter scaffold. The design is being refined before implementation.

## Project documents

- [Initial design](GitVisual-InitialDesign-v0.5.1.md) — v0.5.1 draft; goals, scope, behavior, open decisions, and proposed acceptance criteria.
- [Development workflow](devstatemachine.md) — reusable workflow v1.2.0; governs design, implementation, testing, and acceptance.
- `Maintenance.md` — planned at the repository root; it will record accepted implementation versions, test evidence, and known limitations after implementation begins.

Project document filenames remain stable. Each document has an independent SemVer version in its title; Git history will preserve and compare committed revisions. When developing the app, source diffs against the last accepted commit are part of our verification workflow, not an app feature.


