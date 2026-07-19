---
applyTo: 'quarto/**'
---
# Porting Conventions (SymPy → Symbolics/Nemo)

## Order & Triage

Port smallest/most-mechanical first, hardest last: `limits/` and `precalc/` → `derivatives/` → `integrals/` last. Triage each chapter's SymPy usage into three classes:

1. **Mechanical**: differentiation, substitution, simple algebra, trig equation solving → translates near line-for-line onto `Symbolics.gradient`/`derivative`/`symbolic_solve`.
2. **Needs `Nemo`**: any polynomial equation solving (even a plain quadratic) — add `using Nemo` to the chapter's setup; works correctly with it.
3. **Needs a rewritten example**: worked examples whose antiderivative `SymbolicNumericIntegration.integrate` can't find. Options: pick a different example function that does integrate, add a numeric fallback, or note the limitation inline as a teaching moment.

Anchor: expand `alternatives/symbolics.qmd` into the canonical symbolic-math reference other chapters link to, instead of duplicating setup boilerplate per chapter.

## Verified Capability Findings — RE-VERIFY BEFORE RELYING

Verified live 2026-07-13 (this ecosystem moves fast; re-run these checks against currently-installed versions before making porting decisions — see the `julia-coding-conventions` skill, "Verifying Capability Claims"):

- `Symbolics.symbolic_solve`: trig/exponential equations solve **natively** (`sin(x) ~ 0` → `2πn`; `exp(x) ~ 2` → `slog(2)`); **polynomials require `using Nemo`** — without it even `x^2 - 2 ~ 0` errors.
- `SymbolicNumericIntegration.integrate`: handles integration by parts (`x*sin(x)` ✓), correctly reports non-elementary integrals (`exp(x^2)`), **but fails on `1/(1+x^2)` (arctan)** — expect more gaps of this class throughout `integrals/`; check chapter-by-chapter, don't assume.
- The full `Symbolics`+`SymbolicNumericIntegration`+`Nemo` chain is pure Julia (306-package manifest verified free of `PyCall`/`PythonCall`/`Conda`).

## Ported-Chapter Setup Pattern

```julia
using CalculusWithJuliaSquared   # brings Plots, Symbolics, Roots, calculus utilities
using Nemo                       # only if the chapter solves polynomial equations
```

Never alongside `using CalculusWithJulia` or `using SymPy` in the same chapter. CWJS reexports mean no separate `using Plots`/`using Symbolics`. What CWJS provides: see "What CalculusWithJuliaSquared Provides" in the Calculus repo's copilot-instructions, or the package's own docs.

## Per-Chapter Definition of Done

- `using CalculusWithJulia` → `using CalculusWithJuliaSquared` flipped; no `SymPy` references remain in the chapter
- `quarto render` of the chapter succeeds (code executes at build — this is the test)
- `typos` clean
- Math output verified against the upstream published page for the same chapter (results should match, not just run)
