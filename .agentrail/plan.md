# pl24r — Pascal P-Code Linker in Rust

## Goal

Build a Rust CLI tool that combines multiple `.spc` (symbolic p-code assembler) text files into one merged `.spc` file for pasm assembly. This is a Phase 1 offline text-level linker for the Pascal-on-COR24 toolchain.

## Pipeline

```
Pascal source(s) → p24p compiler → .spc file(s)  ─┐
Hand-written runtime .spc (phase0)                 ├→ pl24r linker → combined .spc → pasm → .p24 → emulator
Pascal-compiled runtime .spc (phase1+)             ┘
```

## Input categories

1. **App** — the main program module (compiled from Pascal by p24p)
2. **User library** (optional) — separately compiled Pascal library
3. **Phase 0 runtime** — hand-written .spc (write_int, write_ln, write_bool)
4. **Phase 1+ runtime** — Pascal-compiled runtime routines

## Module metadata format (co-designed with pv24a/p24p)

New `.spc` directives for linking:

- `.module Name` — declare module identity
- `.export sym` — mark symbol as visible to other modules
- `.extern sym` — declare external symbol dependency
- `.endmodule` — end module boundary

**Fallback**: files without metadata are treated as export-all (all `.proc`, `.global`, `.data` symbols exported).

## Output

One combined `.spc` text file with all modules merged, all symbols resolved, ready for pasm. No `.pco` or `.pex` intermediate formats.

## Design constraints

- Text in, text out — no binary formats
- Transport-neutral (files, stdin, pipes)
- Line-oriented, forward-readable
- Entry point: the `main` proc from the app module
- Duplicate symbol detection with clear errors
- Unresolved extern detection with clear errors

## Steps

1. **001-spc-parser** — Rust project scaffold + .spc text parser
2. **002-symbol-table** — Symbol collection, export/import resolution, validation
3. **003-linker-core** — Module merging, ordering, combined .spc emission
4. **004-cli-integration** — CLI interface, file/stream modes, error reporting
5. **005-pipeline-test** — End-to-end test with real runtime + app through full pipeline
