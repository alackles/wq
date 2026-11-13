# Decisions log

One line per decision, with who made it and why. `docs/DESIGN.md` is the frozen statement of intent. If the implementation diverges from it, the divergence is logged here and the design doc is not edited to match.

## Design (Acacia)

- 2026-11-13: The game runs in real Neovim, not an emulator, because it's more fun, and real vim is both the teacher and the haunted house.
- 2026-11-13: Frame C: a human reviewer cleaning up LLM-generated code, replacing a previous reviewer who vanished. It puts the work loop and the mystery in one setting.
- 2026-11-13: The in-game code is COBOL. Most developers don't work in it, so it doesn't feel like the day job, and "the last human on earth working on COBOL" barely needs any fiction.
- 2026-11-13: Tickets state exactly what's wrong (the Papers, Please rulebook model), so the player doesn't need to know the language.
- 2026-11-13: Build a vertical slice first.
- 2026-11-13: Using the player's real `$USER` or reaching outside the sandbox is postponed.
- 2026-11-13: Training fragments are liked but not placed in the story yet. Kept open, nothing discarded.

## Conventions (Claude)

- Docs are Markdown in `docs/`, so they're portable and readable on GitHub and in Obsidian.
- `.claude/settings.local.json` is git-ignored because it holds machine-local permissions, not project state.
