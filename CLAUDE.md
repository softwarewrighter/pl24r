# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## CRITICAL: AgentRail Session Protocol (MUST follow exactly)

This project uses AgentRail. Every session follows this exact sequence:

### 1. START (do this FIRST, before anything else)
```bash
agentrail next
```
Read the step prompt, context, and past trajectories.

### 2. BEGIN
```bash
agentrail begin
```
Mark step as in-progress immediately. Do NOT ask the user for approval.

### 3. WORK
Execute the step prompt instructions directly. Do NOT ask "shall I proceed?" — just do it.

### 4. COMMIT
Commit code changes with git.

### 5. COMPLETE (LAST thing, after committing)
```bash
agentrail complete --summary "what you accomplished" \
  --reward 1 \
  --actions "tools and approach used" \
  --next-slug "next-step-slug" \
  --next-prompt "what the next step should do" \
  --next-task-type "task-type"
```
If the step failed: `--reward -1 --failure-mode "what went wrong"`
If the saga is finished: add `--done`

### 6. STOP (after complete, DO NOT continue working)
Do NOT make any further code changes after running `agentrail complete`.
The next session depends on accurate trajectory recording.

Do NOT skip any of these steps.

## Project: pl24r — Pascal P-Code Linker in Rust

A Rust CLI tool that combines multiple `.spc` (symbolic p-code assembler) text files into one merged `.spc` file. This is a Phase 1 offline text-level linker for the Pascal-on-COR24 toolchain.

**Pipeline:**
```
Pascal source(s) → p24p compiler → .spc file(s)  ─┐
Hand-written runtime .spc (phase0)                 ├→ pl24r → combined .spc → pasm → .p24 → emulator
Pascal-compiled runtime .spc (phase1+)             ┘
```

**Module metadata format** (new .spc directives for linking):
- `.module Name` — declare module identity
- `.export sym` — mark symbol as visible to other modules
- `.extern sym` — declare external symbol dependency
- `.endmodule` — end module boundary
- Files without metadata: export-all fallback (all .proc/.global/.data exported)

## Related Projects

- `~/github/softwarewrighter/p24p` (a.k.a. p24c) — Pascal compiler in COR24 C, emits .spc
- `~/github/softwarewrighter/pr24p` — Pascal runtime library (hand-written .spc + Pascal source)
- `~/github/sw-vibe-coding/pv24a` — P-code VM + pasm assembler (COR24 assembly)
- `~/github/sw-vibe-coding/tc24r` — C cross-compiler for COR24 (Rust)
- `~/github/softwarewrighter/web-dv24r` — Browser-based debugger for the VM

## Available Task Types

`rust-project-init`, `rust-clippy-fix`, `rust-test-write`, `pre-commit`

## Key Documentation (READ BEFORE WORKING)

- `docs/research.txt` — Linker design research (module formats, linking strategies, architecture decisions)
- `.agentrail/plan.md` — Saga plan with step breakdown
- `~/github/softwarewrighter/pr24p/docs/runtime.md` — Runtime library structure and phases
- `~/github/sw-vibe-coding/pv24a/hello.spc` — Example .spc file
- `~/github/softwarewrighter/pr24p/src/runtime.spc` — Real runtime .spc

## Build & Test

```bash
cargo build
cargo test
cargo clippy -- -D warnings
```

## Conventions

- Rust edition 2024
- No `#[allow(...)]` — fix warnings, never suppress
- Text in, text out — no binary intermediate formats
- .spc is the only file format (input and output)
