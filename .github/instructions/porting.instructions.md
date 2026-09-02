---
applyTo: 'quarto/**'
---
# Porting Conventions (SymPy → Symbolics/Nemo)

## Order & Triage

**Port order follows the study progression through the Calculus repo's docs** — the live sequence and per-chapter triage live in `_research/CHAPTER_MAP.md` (local-only). The old "mechanical-first, `limits/` → `derivatives/` → `integrals/` last" heuristic is **superseded**: verification (below) showed differentiation is the mechanical operation, while **limits and integrals are the hard rewrites**. Differential-calculus chapters lead (starting with the derivatives group); integrals come as study reaches them.

Triage each chapter's SymPy usage into four classes:

1. **Mechanical (M)**: differentiation (`diff` → `Symbolics.derivative`/`Differential`), Taylor series (`series` → `Symbolics.taylor` or `TaylorSeries.jl`), simple algebra, trig equation solving → near line-for-line.
2. **Needs `Nemo` (N)**: any polynomial equation solving (even a plain quadratic) — add `using Nemo` to the chapter's setup.
3. **Symbolic-limit (L)**: compose **`SymbolicLimits.jl`** (see findings — cancel/rearrange the `0/0` first, unwrap both args, take `[1]`). The numeric path (CWJS `lim` / `Richardson.extrapolate` / a numeric table) is now the *fallback*, needed only for the not-implemented forms — trig (notably the two fundamental limits `sin(h)/h`, `(1-cos(h))/h`), `sqrt`, non-integer powers. *(Scoped down 2026-08-16 from "all limits are numeric rewrites" — that earlier claim was wrong, asserted untested.)*
4. **Integration rewrite (I)**: antiderivatives `SymbolicNumericIntegration.integrate` can't find — pick a different example, add a numeric `quadgk` fallback, or note the limitation inline as a teaching moment.

Anchor: expand `alternatives/symbolics.qmd` into the canonical symbolic-math reference other chapters link to, instead of duplicating setup boilerplate per chapter.

## Choosing What to Show: Symbolic, Intermediate, or Numeric

**Understanding is the primary objective — it outranks line-for-line fidelity to upstream.** Where SymPy's symbolic result has no Symbolics equivalent (classes **L** and **I** especially), do *not* reflexively drop to a final number. Decide deliberately, at each site, which form teaches best:

- **The step before the answer** is often the most clarifying. `exp(-π)` says more than `0.0432139…`; a difference quotient rearranged by hand says more than the limit it evaluates to. Prefer the un-evaluated or intermediate form when the *structure* is the lesson.
- **A number** is right when the point is magnitude, convergence, or a concrete comparison.
- **Both, in sequence**, when the upstream cell carried two ideas at once. Worked example — `derivatives/derivatives.qmd`, deriving $[\sin(x)]' = \cos(x)$: SymPy's `limit(...)` returned the *general* `cos(x)`, which both demonstrates the limit and proves the rule for every `x`. The port needs two cells to carry that: (1) `lim` at one sample `c`, showing the difference quotient converge two-sided, then (2) a small `DataFrame` of the difference quotient against the claimed formula across several `x`, showing the answer is the *function* you recognize. Neither alone replaces the original.
- **Name the substitution in prose, and round the float noise away.** Say plainly that numeric evidence is standing in for a symbolic proof — then round so the reader sees the mathematical answer rather than the arithmetic's debris (`0.0`, not `-5.55e-17` or `-5.00044e-7`). `round.(v; digits=5) .+ 0.0` is the idiom; the `.+ 0.0` turns a rounded `-0.0` back into `0.0`. Symbolics returning a clean `0` where SymPy returned an epsilon is one of the reasons this fork prefers it — tables in the notes should read the same way. Carry the caveat in the sentence, not in the digits.

This costs more work per chapter than a mechanical swap. That is accepted and intended: these notes are a study tool, and the added intermediate steps are where the understanding lives.

### ⚠️ The reader must be able to do it themselves, on a DIFFERENT problem

**The chapter is not a record of what upstream did. It has to teach the method.** Aron's
standard, stated 2026-09-02:

> The goal is not only to show what was done in upstream but to teach me (and anyone else who
> reads it) how I can do the same thing myself with similar but different problems. If things
> aren't totally explicit I won't be able to.

This ranks the options, and the order is not negotiable:

1. **A `symlim` route is always the best answer.** It is explicit, it is named, and the reader
   can apply it verbatim to a problem we never wrote about. If the package can be made to own
   the case, **make the package own it** — a CWJS release that merges first is cheaper than a
   chapter full of bespoke manoeuvres. Every route `symlim` gains is a technique the reader
   gets for free everywhere else in the book.
2. **Only if no route is possible**, explain it in prose — and then the prose must **fully
   state what is missing**. Name the absent capability, say why it is absent, say what the
   reader should do when they meet the same shape again. A worked-around cell that happens to
   succeed here while leaving the *general* case unexplained is the failure mode: it looks
   like an answer and teaches nothing transferable.

**The anti-pattern, concretely.** Picking a formulation that works for the specific numbers in
front of you, and not saying that the neighbouring problem — one sign change away, one
parameter away — will behave differently. That hides the gap instead of teaching it. If the
reader cannot tell from the chapter *why* their variation failed, the chapter is not done.

Corollary: when a limit's answer depends on a parameter's sign or range, say so **and show
it** — display the free-parameter result alongside the pinned one. An answer that is correct
for a branch the reader did not intend is more dangerous than a refusal.

See the `sciml-coding-conventions` skill's **STOP RULE** for the mechanical trigger and the
catalogue of rewrites to try before concluding that no route exists.

## Verified Capability Findings — RE-VERIFY BEFORE RELYING

Verified live 2026-07-20 (Julia 1.12.6 · Symbolics 7.32.1 · SymbolicNumericIntegration 1.11.3 · TaylorSeries 0.17.5). This ecosystem moves fast; re-run before relying — see the `julia-coding-conventions` skill, "Verifying Capability Claims":

- **Differentiation** (`Symbolics.derivative`/`Differential`) and **Taylor series** (`Symbolics.taylor`, exact rationals; or `TaylorSeries.Taylor1`) work cleanly — the genuinely mechanical operations.
- `Symbolics.symbolic_solve`: trig/exponential equations solve **natively** (`sin(x) ~ 0` → `2πn`); **polynomials require `using Nemo`** — without it even `x^2 - 2` errors.
- **Judge capabilities by ecosystem, not core.** Symbolics deliberately composes with companion packages (`SymbolicLimits` for limits, `Nemo` for polynomial solving, SymbolicUtils `@rule` for missing rewrites). Test the composing package before declaring a gap — the original "no working drop-in for `limit`" claim below was wrong precisely because it was asserted without running `SymbolicLimits`.
- **Distinguish a tooling gap from a mathematical impossibility.** The ecosystem rule above says to hunt for a companion package before declaring a gap — but first ask whether the result *exists*. `symbolic_solve(cos(x) - x, x)` returns `nothing` not because Julia is weak but because `cos(x) = x` has no closed form in elementary functions (SymPy cannot do it either — which is exactly why upstream uses it to motivate Newton's method). Such a site needs **no port work and no apology**: state it as mathematics, and let the numeric method be the point. Reserve "Symbolics can't do X" phrasing for cases where SymPy genuinely could and we now cannot; otherwise write "there is no closed-form solution". A quick test of whether a claim is about tooling: would a different CAS produce an answer? If no, it is mathematics.
- **Symbolic limits via `SymbolicLimits.jl` — works, with sharp edges** (verified 2026-08-16, SymbolicLimits v1.1.5; supersedes the earlier "Gruntz/at-infinity only, rewrite everything numerically" claim, which was false — it computes finite two-sided/one-sided limits):
  - **API:** `SymbolicLimits.limit(expr, var, c[, :left|:right|:both])` returns a `(value, assumptions)` **tuple** — take `[1]`. **Unwrap both** `expr` and `var` (`Symbolics.unwrap`): a `Num` expr silently hits a fallback method that returns the input unchanged (looks like "declined to compute").
  - **DANGER — silent wrong answer:** a raw polynomial `0/0` difference quotient returns **`0`** with no error (`((x+h)^5 - x^5)/h → 0`, not `5x^4`). **Cancel first** — `simplify(expand(dq))` — after which both `SymbolicLimits.limit` and plain `substitute(h => 0)` give the right answer (and the cancellation *is* the textbook derivation, so show it). Numerically sanity-check every symbolic limit with CWJS `lim` when first porting a site.
  - **Works raw:** exp/log/rational quotients — `(exp(x+h)-exp(x))/h → exp(x)`, `(log(x+h)-log(x))/h → 1/x`, `(1/(x+h)-1/x)/h → -1/x²` — and at-infinity Gruntz limits (`log(x)/x → 0`).
  - **Not implemented (loud, honest errors):** *any* trig anywhere in the expression (even as coefficients of the non-limit variable), `sqrt`, non-integer powers (`(1+1/x)^x` → "transform to log/exp"). It is a log-exp engine by design.
  - **Missing identities → SymbolicUtils `@rule`.** There is still no `expand_trig`, but a one-line rule performs the rewrite live: `@rule sin(~a + ~b) => sin(~a)*cos(~b) + cos(~a)*sin(~b)` turns `sin(x+h)` into the addition-formula form (apply via `Postwalk`/`Chain`, or directly for a top-level match). Prefer showing the rule in the notes over "we apply the formula by hand".
  - **Trig first-principles pattern:** after the `@rule` expansion, `[sin]'`/`[cos]'` reduce to the two fundamental limits `sin(h)/h → 1` and `(1-cos(h))/h → 0`, which no current package computes symbolically — keep the numeric table / geometric argument for *those two only*. (`Symbolics.taylor` in `h` + `substitute(h=>0)` also yields `cos(x)` / `-sin(x)` exactly — a good cross-check, but circular as a *proof*, since Taylor coefficients presuppose the derivative.)
  - **`sqrt` dq:** conjugate trick, then `substitute(h => 0)` → `1/(2√x)`. (SymbolicLimits can't.)
- `SymbolicNumericIntegration.integrate`: returns `(solved, unsolved, err)`. Integration by parts (`e^x*sin(x)` ✓) and polynomials work; **fails (`err=Inf`) on `1/(1+x^2)` (arctan), `1/(x*log(x))`, and even `4x/√(x^2+1)`** — expect many gaps across `integrals/`; check chapter-by-chapter.
- The full `Symbolics`+`SymbolicNumericIntegration`(+`Nemo`) chain is pure Julia.

## Ported-Chapter Setup Pattern

```julia
using CalculusWithJuliaSquared   # brings Plots, Symbolics, Roots, calculus utilities (incl. `lim`)
using Nemo                       # only if the chapter solves polynomial equations
using Richardson                 # only if a limit chapter uses numeric extrapolation
using SymbolicLimits             # only if the chapter takes symbolic limits (add SymbolicUtils too if it writes @rule rewriters)
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
- Math output verified against the upstream published page for the same chapter (results should match, not just run). Where the port *deliberately* diverges (classes **L**/**I** — see "Choosing What to Show"), identical output is not the standard: verify the mathematics independently and make the divergence explicit in the prose.
- **Every external link in the chapter resolves.** These notes are years old and their
  citations rot; two dead links were found by accident in `derivatives.qmd` alone. Check
  before opening the PR:

  ```bash
  /usr/bin/grep -o 'http[s]*://[^)]*' quarto/<group>/<chapter>.qmd | sort -u | \
    while read u; do printf '%s  %s\n' "$(curl -s -o /dev/null -w '%{http_code}' -L --max-time 20 "$u")" "$u"; done
  ```

  Anything not `200` needs a judgement call — find the moved page, substitute an equivalent
  source, or drop the link. Note `maa.org` answers **403**, not 404, so a check that only
  looks for 404 misses that whole family. Fix links only in the chapter being ported: the
  same dead URL often appears in unported chapters too (and may be a *different* citation
  that happened to share the URL), and editing those early only adds noise to their eventual
  diff against upstream.

## Publishing a ported chapter

The book is live at <https://fourm.info/cwjsn/>, serving **only** the chapters ported so far.
Publishing one is part of finishing it, not a separate project.

**How the deploy works.** `.github/workflows/deploy-book.yml` fires on any push to `main`
touching `quarto/**` (or by manual dispatch). It renders the book **from the committed
`_freeze/` results** and pushes the output into `math_tech_study`'s `gh-pages` under `cwjsn/`.
No Julia and no Python run in CI — deliberate, since unported chapters still import `SymPy`
and executing them would drag Python into a fork whose purpose is removing it.

**To publish a newly ported chapter — one commit, three parts:**

1. **Render it locally.** That writes `quarto/_freeze/<group>/<chapter>/execute-results/html.json`,
   which is the executed output CI will assemble.
2. **Promote it in `quarto/_quarto.yml`** — move its line out of the commented "NOT YET PORTED"
   archive into the published list above it.
3. **Commit the `_freeze/` file *and* `_quarto.yml` together**, then push to `main`.

Forgetting the `_freeze/` file is the trap: CI cannot execute anything, so that chapter has no
output to assemble. The workflow's page-count guard turns this into a loud failure rather than
a silently partial book.

**Transitional.** Once `SymPy` is gone from every group's `Project.toml`, CI can instantiate the
(pure-Julia) environments and render from source like a normal Quarto book, and `_freeze/` can
go back to being ignored.

## Render discipline (Quarto can hang — guarantee liveness)

- **One chapter at a time**: `quarto render <chapter>.qmd`, never the whole book to check a port. Each `.qmd` → its own `.html`; a hang in one render can't touch other chapters' already-built outputs.
- **Always wrap in a wall-clock timeout**: `timeout 1200 quarto render <chapter>.qmd`. Renders occasionally spin at 100% CPU forever (observed post-execution, in plotly/HTML serialization). A wall-clock timeout is the only guaranteed catch — it covers execution, serialization, and embedding alike. If it trips, the render is STUCK: kill and find the offending cell; don't blindly re-run.
- **`freeze: auto` is on** (`_quarto.yml`): a chapter that renders cleanly once is cached in `_freeze/`; re-runs reuse it, so full-book renders resume rather than restart. (**`_freeze/` is committed** as of 2026-08-18 — see "Publishing a ported chapter" below. It used to be gitignored.)
- **Only the *first* render in a session is slow — iterating is cheap.** The cold render loads the group's whole env (CWJS + Plots + ~25 deps) *and* executes every cell — that's the ~15 min the timeout is sized for. Re-rendering after editing a few cells is **< 1 min**: `_freeze` supplies the untouched cells and QuartoNotebookRunner keeps a warm worker (packages stay loaded), so only the changed cells re-run. (Observed 2026-08-16: a cold render logged `Running [1/93]…[93/93]`; the next, after a 5-cell edit, logged *zero* `Running` lines yet still emitted the updated outputs.) So don't contort the workflow to dodge "a second render" — only the first one is expensive.
- **Correctness ≠ display**: the port check is (a) all cells execute error-free and (b) output matches upstream — both in the *execution* stage. The stage that hangs is usually HTML/plotly *embedding*, a display concern; a hung embed is not proof the port is wrong. Isolate heavy figures (usually plotly) separately.
- Per-cell `execute: timeout:` is a possible backstop but UNVERIFIED for the native Julia engine (QuartoNotebookRunner) — the external wall-clock timeout is the reliable mechanism.
- **Check the output in a browser, served over HTTP** — `python3 -m http.server 8765 --bind 127.0.0.1` from `_book/`, or `quarto preview`. Grepping the HTML proves a cell *executed*; it does not prove the page *displays*. Interactive figures depend on JavaScript that only runs in a real browser.

### A render can silently use the WRONG package version

Three layers can each serve stale code with no error, and they stack. All three were hit in
one sitting on 2026-08-30, re-rendering `limits.qmd` after `CalculusWithJuliaSquared`
v0.8.0:

1. **The chapter environment.** Manifests are gitignored, so the `[compat]` bound is the
   only thing pinning a version. Each ported group's `Project.toml` now **pins CWJS
   exactly** (`CalculusWithJuliaSquared = "0.8.2"`), so a stale environment fails loudly
   instead of quietly rendering against an old version — **keep that pin current on every
   CWJS release, patches included.** Before the pins existed, `limits/` and `derivatives/`
   were both silently on **v0.6.1** while `main` was on v0.8.0, and `Pkg.add` of an
   unrelated package did not move them; only `Pkg.update` does. Adding the first pin
   immediately surfaced a real conflict (CWJS wanting IntervalArithmetic 1.x against
   `ImplicitEquations`' 0.20.9) that would otherwise have detonated later.
2. **The QuartoNotebookRunner worker.** The bullet above sells the warm worker as free
   speed, and it is — but "packages stay loaded" means a *running process holds the old
   module*. After fixing the environment the render still produced v0.6.1 answers, because
   it was **executing the new cells against the old code**. Nothing in the output says so.
3. **`_freeze` plus CI.** CI assembles HTML from the committed freeze and never re-executes,
   so a stale render stays stale *and green*. **Whatever the local machine rendered is what
   ships.** There is no downstream check.

Symptom of all three: a cell you just wrote returns the *old* behaviour. Ours rendered
`(nothing, :unresolved)` where `symlim` returns `(0.0, :squeeze)` in a fresh REPL.

**Before re-rendering a chapter after a CWJS release, do both:**

```bash
# 1. raise the pin in quarto/<group>/Project.toml to the new version, then:
julia --project=quarto/<group> -e 'using Pkg; Pkg.update("CalculusWithJuliaSquared"); Pkg.status("CalculusWithJuliaSquared")'
quarto call engine julia stop      # drops the warm worker; next render is a cold start
```

Then render and **check a cell whose output you know should have changed** — not merely
that the render succeeded.

**Re-rendering an already-published chapter after a package change is itself a check worth
running.** It is how a v0.7.0 regression was found: `symlim` had started returning
`:unresolved` for `x^2 + 1 + log(abs(11x-15))/99` at `15//11`, where the live page renders
`-Inf`. CI could not have caught it, and neither could the package's own test suite — the
case only existed in the chapter.

### Blank figure gaps: a JS-load problem, not a port problem

Symptom: some figures render and others are blank gaps, splitting cleanly by backend — GR/static figures fine, `plotly()` figures missing. Cause is script load order, **not** anything in the chapter.

Plots.jl's `plotly()` backend emits figures as `Plotly.newPlot(...)` calls needing the *global* `Plotly`, but references the library only inline in the body. Quarto injects requirejs into the `<head>` first, so by the time plotly loads it registers as an AMD module and never defines that global; every figure throws. GR figures are unaffected because they are `data:` URIs needing no JS at all — which is exactly why the split looks backend-shaped.

Fixed fork-wide in `_quarto.yml` via `include-in-header`, which **hides the AMD loader across the plotly load** so it falls back to defining the global:

```html
<script>window.__amd_define = window.define; window.define = undefined;</script>
<script src="https://cdn.plot.ly/plotly-2.6.3.min.js"></script>
<script>window.define = window.__amd_define; delete window.__amd_define;</script>
```

Loading plotly *before* requirejs — the way upstream's published pages happen to be built — is **not achievable** here: `include-in-header` appends to `<head>`, so the tag always lands after Quarto's requirejs (verified: plotly at offset 6222, requirejs at 5871). Hiding the loader works regardless of placement, which makes it the more robust fix. Version pinned to 2.6.3 to match upstream and the 2.x-era `titlefont`/`tickfont` attributes Plots.jl emits.

Diagnostic if it recurs — the wrapper must be present, and in a browser console `typeof Plotly` must be `"object"`, not `"undefined"`:

```bash
python3 -c "s=open('_book/<ch>.html').read(); print('amd-hide', s.find('__amd_define')); print('plotly', s.find('cdn.plot.ly'))"
```

Note what this is *not*: not the plotly **version** (swapping 2.6.3→3.1.0 alone changes nothing), not `file://` vs HTTP, and not GR.
