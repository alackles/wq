# wq — design

*Working title. A text-based horror game that teaches vim by putting you inside it.*

Status markers used below:
- **Decided** means Acacia confirmed it.
- **Proposed** means Claude suggested it and it isn't confirmed yet. Treat it as a default to test, not a commitment.
- **Open** means nobody has decided.

---

## Pitch

You're a contract reviewer for an AI company. Their model generates COBOL for systems that still run banks, payrolls, and benefits offices, systems whose original programmers have retired or died. Your job is to fix what the model gets wrong. As far as anyone can tell, you are the last human on earth working on COBOL.

You're replacing someone. Their session was never closed.

The game takes place entirely inside a real Neovim instance. There's no UI apart from the editor, so every mechanic, scare, and clue is something vim can actually do.

## Pillars

1. **The interface is the game.** This is in the lineage of Type Help, Papers, Please, The Roottrees are Dead, and TR-49. You never leave the editor, and the editor is the world. *(Decided)*
2. **Learn vim by using real vim.** It isn't an emulator. Every skill the player picks up transfers directly to their own editor. *(Decided)*
3. **The horror lives in vim's buried state.** That means undo history, swap files, registers, marks, jumplists, folds, concealed text, and `:help`. These features are real, mostly unexamined by ordinary users, and already uncanny. *(Proposed as the design principle; Acacia liked the examples.)*
4. **You don't need to know the language.** Each ticket states exactly what's wrong, the way the rulebook does in Papers, Please. The vim skill is the work, and understanding COBOL leaks in as a side effect. *(Decided)*

## Frame

*(Decided: frame "C".)* Two layers:

- **The work loop.** You're a human reviewer cleaning up LLM-generated COBOL. Each shift you get an inbox of tickets, and each ticket is one file plus a precise description of what's wrong. This teaches the vim skills.
- **The mystery.** The previous reviewer vanished mid-job, and what they left behind is the story: their undo history, their swap files (`E325: ATTENTION … process still running`), registers you didn't fill, marks you didn't set. You piece together what happened while you do your job.

**Inspiration lineage:** the original seed was cleaning up OCR'd archival text. That's where the "cleanup as the reason to edit" premise came from, and it may come back through the training fragments (see Open questions).

## Language: COBOL

*(Decided.)* Why COBOL:

- It reads like English (`MOVE ZERO TO TOTAL-PAY.`), so non-programmers can follow a ticket.
- Hyphenated identifiers teach `w` vs `W` and `iw` vs `iW`.
- Statements end in periods, so vim's sentence motions and objects (`(`, `)`, `das`, `cis`) actually work on the code.
- Paragraphs and DIVISIONs make `{`, `}`, and `dap` natural.
- It's the language of bureaucracy. The code processes records about people, so the data can carry the story.
- The premise barely needs fiction. LLMs really are being pointed at legacy COBOL because the humans who understood it are gone.

**Fixed-form columns.** In fixed-form COBOL the compiler ignores columns 73–80 (the identification area). That gives us a hidden channel the machine can't read but a human who scrolls right can. The previous reviewer leaves messages there. This teaches `$`, `|`, horizontal scrolling, and `colorcolumn`. *(Proposed.)*

## Core loop

*(Proposed; the grey-box will test it.)*

1. A shift starts. The inbox buffer lists tickets.
2. You open a ticket's file. The ticket says what's wrong, e.g. "Loop in `CALC-OVERTIME` runs 11 times; should run 12."
3. You fix it.
4. `:w` submits. The fix is checked against an expected result and shown diegetically as a review/CI run.
5. A keystroke count is compared with a "standard". Supervisor notes respond. This is VimGolf as workplace performance review, the Papers, Please quota.
6. The next ticket comes up. Between and inside tickets, the previous reviewer's leftovers show up.

## Core mechanic: Insert mode is exposure

*(Proposed; this is the main thing the prototype has to test.)*

Whatever you type in Insert mode is captured as training data, and the model learns from you. Normal-mode edits are surgical and give it almost nothing.

- A presence or capture meter rises while `-- INSERT --` is showing.
- This teaches real vim practice: live in Normal mode, make small exact edits, use `.`, `ciw`, `cgn`, and `Ctrl-A`.
- The arc: early on the model's output is just bad. Later it starts to resemble *your* fixes. The more you typed, the sooner and the closer. Eventually it produces things it should not know.
- Implied about the previous reviewer: they typed too much.

## Skill ↔ ticket mapping (examples)

| Defect in generated code             | Vim skill                                 |
|--------------------------------------|-------------------------------------------|
| Off-by-one count or limit            | `Ctrl-A`, `Ctrl-X`                        |
| Misnamed identifier used throughout  | `*`, `cgn`, `.`, `:%s/\<x\>/y/g`           |
| Leftover `DISPLAY` debug lines       | `:g/DISPLAY/d`                            |
| Wrong literal                        | `ci"`, `ci'`                              |
| A whole wrong sentence or paragraph  | `das`, `dap`, `{`, `}`                    |
| Wrong hyphenated name                | `ciW` vs `ciw`                            |
| Lowercase where caps are required    | `gU`, `~`                                 |
| Misaligned columns                   | visual block (`Ctrl-V`), `|`              |

## Vertical slice

*(Decided: we're building a vertical slice first. Proposed: its contents.)*

- One or two shifts.
- Skills covered: basic motion (`hjkl`, `w`/`W`/`b`/`e`), deletion (`x`, `dd`), undo/redo (`u`, `Ctrl-R`), `ciw`, search (`/`, `n`), `Ctrl-A`, `.`.
- **One big scare: undo goes back past your own edits.** The player makes a mistake, presses `u` by reflex, and keeps going into edits the previous reviewer made. This is done with a pre-built undofile.
- **One hidden-channel discovery:** a message in columns 73–80.
- The insert-mode exposure meter, in crude form.

### Questions the grey-box prototype has to answer

1. Does insert-mode exposure feel tense, or just annoying?
2. Is the keystroke quota motivating or punishing? Where should the standard sit?
3. Does the undo scare land when the player hits it naturally, or do we have to steer them into it?
4. Is real Neovim controllable enough? Can we keep the player inside the fiction without breaking their editor?

## Technical sketch

*(Proposed.)*

- Launch with `nvim --clean -u game/init.lua` from a launcher script, so the player's own config is never touched.
- Game logic in Lua. `vim.on_key` for keystroke counting, autocmds for `BufWritePost`, `ModeChanged`, and `CursorMoved`.
- Per-shift files are generated into a play directory, with pre-built undofiles and swap files made by a headless nvim at build time.
- Custom `:help` files with real tags, which can drift from the truth.
- The statusline is the diegetic HUD: shift clock, quota, exposure.
- `:q` and `:wq` are intercepted until the ending. The title is the exit.

## Open questions

- **Training fragments.** The model regurgitates bits of its training data, possibly scanned card decks or printouts with OCR noise, possibly the previous reviewer's own edits. Acacia likes the idea; where it sits in the story is undecided. Nothing about it is discarded. *(Open)*
- **Stepping outside the sandbox.** Should the game use the player's real `$USER` or touch anything outside its own directory? Postponed by Acacia. *(Open)*
- **How fixes are checked.** Claude's suggestion is text comparison presented as a test run for the slice, with real execution (GnuCOBOL) later if it earns its place. *(Proposed, not confirmed)*
- **The ending.** What does `:wq` save, and who is quitting? *(Open)*
- **The name of the company or model.** *(Open)*
- **Title.** "wq" is the working title. *(Proposed)*
