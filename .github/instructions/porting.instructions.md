---
applyTo: 'quarto/**'
---
# Porting Conventions (SymPy → Symbolics/Nemo)

## Order & Triage

**Port order follows the study progression through the Calculus repo's docs** — the live sequence and per-chapter triage live in `_research/CHAPTER_MAP.md` (local-only). The old "mechanical-first, `limits/` → `derivatives/` → `integrals/` last" heuristic is **superseded**: verification (below) showed differentiation is the mechanical operation, while **limits and integrals are the hard rewrites**. Differential-calculus chapters lead (starting with the derivatives group); integrals come as study reaches them.

Triage each chapter's SymPy usage into four classes:

1. **Mechanical (M)**: differentiation (`diff` → `Symbolics.derivative`/`Differential`), Taylor series (`series` → `Symbolics.taylor` or `TaylorSeries.jl`), simple algebra, trig equation solving → near line-for-line.
2. **Needs `Nemo` (N)**: any polynomial equation solving (even a plain quadratic) — add `using Nemo` to the chapter's setup.
3. **Symbolic-limit rewrite (L)**: `limit(...)` has **no drop-in** (see findings) — rewrite onto the numeric path (CWJS `lim` / `Richardson.extrapolate` / a numeric table).
4. **Integration rewrite (I)**: antiderivatives `SymbolicNumericIntegration.integrate` can't find — pick a different example, add a numeric `quadgk` fallback, or note the limitation inline as a teaching moment.

Anchor: expand `alternatives/symbolics.qmd` into the canonical symbolic-math reference other chapters link to, instead of duplicating setup boilerplate per chapter.

## Verified Capability Findings — RE-VERIFY BEFORE RELYING

Verified live 2026-07-20 (Julia 1.12.6 · Symbolics 7.32.1 · SymbolicNumericIntegration 1.11.3 · TaylorSeries 0.17.5). This ecosystem moves fast; re-run before relying — see the `julia-coding-conventions` skill, "Verifying Capability Claims":

- **Differentiation** (`Symbolics.derivative`/`Differential`) and **Taylor series** (`Symbolics.taylor`, exact rationals; or `TaylorSeries.Taylor1`) work cleanly — the genuinely mechanical operations.
- `Symbolics.symbolic_solve`: trig/exponential equations solve **natively** (`sin(x) ~ 0` → `2πn`); **polynomials require `using Nemo`** — without it even `x^2 - 2` errors.
- **`limit`: no working drop-in.** `Symbolics.limit` is extension-gated behind `SymbolicLimits.jl` (not in the env) and is a Gruntz/at-infinity engine — it does **not** cover SymPy's finite one-sided `limit(f, x, c, dir)`. Rewrite limits onto the numeric path (class **L**).
- `SymbolicNumericIntegration.integrate`: returns `(solved, unsolved, err)`. Integration by parts (`e^x*sin(x)` ✓) and polynomials work; **fails (`err=Inf`) on `1/(1+x^2)` (arctan), `1/(x*log(x))`, and even `4x/√(x^2+1)`** — expect many gaps across `integrals/`; check chapter-by-chapter.
- The full `Symbolics`+`SymbolicNumericIntegration`(+`Nemo`) chain is pure Julia.

## Ported-Chapter Setup Pattern

```julia
using CalculusWithJuliaSquared   # brings Plots, Symbolics, Roots, calculus utilities (incl. `lim`)
using Nemo                       # only if the chapter solves polynomial equations
using Richardson                 # only if a limit chapter uses numeric extrapolation
```

Never alongside `using CalculusWithJulia` or `using SymPy` in the same chapter. CWJS reexports mean no separate `using Plots`/`using Symbolics`. What CWJS provides: see "What CalculusWithJuliaSquared Provides" in the Calculus repo's copilot-instructions, or the package's own docs.

## Per-group environment (renders resolve here — NOT the root)

QuartoNotebookRunner runs each chapter in the **nearest `Project.toml` walking up from the `.qmd`** — i.e. the chapter-group dir (`derivatives/Project.toml`, `limits/Project.toml`, …), **not** the root `quarto/Project.toml`. The 10 group envs are independent (not a workspace) and ship un-instantiated, so renders fail until each group's env is set up. Before porting a group's first chapter:

1. Add `CalculusWithJuliaSquared` + a `[sources]` entry (fork URL) to that group's `Project.toml`; add `Nemo` too if any chapter in the group solves polynomials. Keep `CalculusWithJulia`/`SymPy` during transition; drop them when the whole group is ported (→ Python-free).
2. Instantiate: `julia --project=<group> -e 'using Pkg; Pkg.instantiate(); Pkg.precompile()'`.

Rendered output lands in `_book/` (gitignored), not beside the `.qmd`.

## Per-Chapter Definition of Done

- `using CalculusWithJulia` → `using CalculusWithJuliaSquared` flipped; no `SymPy` references remain in the chapter
- `quarto render` of the chapter succeeds (code executes at build — this is the test)
- `typos` clean
- Math output verified against the upstream published page for the same chapter (results should match, not just run)

## Render discipline (Quarto can hang — guarantee liveness)

- **One chapter at a time**: `quarto render <chapter>.qmd`, never the whole book to check a port. Each `.qmd` → its own `.html`; a hang in one render can't touch other chapters' already-built outputs.
- **Always wrap in a wall-clock timeout**: `timeout 1200 quarto render <chapter>.qmd`. Renders occasionally spin at 100% CPU forever (observed post-execution, in plotly/HTML serialization). A wall-clock timeout is the only guaranteed catch — it covers execution, serialization, and embedding alike. If it trips, the render is STUCK: kill and find the offending cell; don't blindly re-run.
- **`freeze: auto` is on** (`_quarto.yml`): a chapter that renders cleanly once is cached in `_freeze/`; re-runs reuse it, so full-book renders resume rather than restart. (`_freeze/` is gitignored — local-only, won't carry to CI unless committed.)
- **Correctness ≠ display**: the port check is (a) all cells execute error-free and (b) output matches upstream — both in the *execution* stage. The stage that hangs is usually HTML/plotly *embedding*, a display concern; a hung embed is not proof the port is wrong. Isolate heavy figures (usually plotly) separately.
- Per-cell `execute: timeout:` is a possible backstop but UNVERIFIED for the native Julia engine (QuartoNotebookRunner) — the external wall-clock timeout is the reliable mechanism.
