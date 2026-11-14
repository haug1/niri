# Known issues with global workspace indices

These issues apply when `layout { global-workspace-indices; }` is enabled.
They were reviewed and accepted on 2026-11-14; no fixes are currently planned.

## Window moves follow focus when disabled

`move-window-to-workspace-up focus=false` and
`move-window-to-workspace-down focus=false` still follow the window to its
destination workspace. They should keep focus on the source workspace.
This affects both config bindings and the corresponding IPC actions.

The global navigation paths use `ActivateWindow::Smart` instead of
`ActivateWindow::No` for `focus=false` in `src/layout/mod.rs`.

## Explicit workspace reordering is undone

`move-workspace-to-index` changes the physical workspace order, but the subsequent
global index refresh sorts it back using the unchanged global index assignments.
For example, moving workspace 1 to index 2 leaves it numbered 1 and restores its
original position. This also affects the action variant that selects the source
workspace by reference.

## Swiping onto an empty workspace omits it from workspace listings

Swiping from an occupied workspace onto an empty placeholder can leave the newly
active workspace without a global index. Gesture handling does not refresh the
global index map, so IPC and ext-workspace omit that workspace until another
operation refreshes the indices. Workspace bars using these interfaces may not
show the active workspace during that interval.

All three issues were reproduced with temporary local regression probes during
the branch review.

## Dismissed observation

Creating a lower-numbered workspace can consume the trailing empty placeholder,
leaving no blank preview below the last occupied workspace in overview. This was
observed on the live compositor. Navigating downward creates the next workspace
on demand, so the missing placeholder was dismissed and is not tracked as an
issue requiring a fix.
