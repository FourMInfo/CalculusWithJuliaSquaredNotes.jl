# Copilot Instructions for CalculusWithJuliaSquaredNotes

> **Note:** Porting-specific conventions are in `.github/instructions/` and load automatically based on the file being edited.

## Project Overview

A **personal fork** of [jverzani/CalculusWithJuliaNotes.jl](https://github.com/jverzani/CalculusWithJuliaNotes.jl) — the "Calculus with Julia" book — being ported **chapter by chapter** off Python-backed `SymPy` onto pure-Julia `Symbolics.jl`/`Nemo.jl`, via the companion package [`CalculusWithJuliaSquared`](https://github.com/FourMInfo/CalculusWithJuliaSquared.jl) (CWJS). The user studies from the published upstream site meanwhile; this fork is not upstream-bound.

## Structure & Build

- Content: `quarto/` — ~96 `.qmd` files in chapter groups (`precalc/`, `basics/`, `limits/`, `derivatives/`, `integrals/`, `ODEs/`, `differentiable_vector_calculus/`, `integral_vector_calculus/`, `misc/`, `alternatives/`)
- Build: `quarto render` (or `quarto preview`) from `quarto/` — **all Julia code blocks execute during the build**, so a chapter rendering cleanly is itself a correctness check
- Environment: `quarto/Project.toml` (plain `[deps]`, no name/uuid — this is a book, not a package); Manifests untracked
- **No CI.** Before committing: run `typos` (config in `_typos.toml`) and render the touched chapter(s)

## The Transition State (core rules)

- `CalculusWithJuliaSquared` (via `[sources]` — unregistered) sits **alongside** upstream `CalculusWithJulia` during the port. Unported chapters keep `using CalculusWithJulia` + SymPy; ported chapters use `using CalculusWithJuliaSquared`.
- **The `using` line is the progress marker** — never mass-rename it; each file flips only when its chapter is actually ported.
- **Never load both packages in one chapter** — their exports overlap.
- **The port is complete when `CalculusWithJulia` and `SymPy` can be removed** from `quarto/Project.toml`.
- Don't "fix" SymPy usage in unported chapters — they get ported whole, per the plan.

## Workflow

Follow the `phased-implementation-workflow` skill (branch per chapter-group/phase, PR, squash-merge on the user's word — no CI to wait for here). The detailed port plan (triage, ordering, verified capability findings) lives in the CalculusWithJuliaSquared.jl repo's local `_research/PHASE_D_NOTES.md`.
