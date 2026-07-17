# CalculusWithJuliaSquaredNotes

A personal fork of [CalculusWithJuliaNotes.jl](https://github.com/jverzani/CalculusWithJuliaNotes.jl) — John Verzani's excellent "Calculus with Julia" notes, readable at [calculuswithjulia.github.io](https://calculuswithjulia.github.io/).

This fork exists to (gradually, chapter by chapter) port the notes' code off Python-backed `SymPy` onto pure-Julia [`Symbolics.jl`](https://github.com/JuliaSymbolics/Symbolics.jl)/[`Nemo.jl`](https://github.com/Nemocas/Nemo.jl), using the companion package [`CalculusWithJuliaSquared.jl`](https://github.com/FourMInfo/CalculusWithJuliaSquared.jl) (itself a pure-Julia fork of `CalculusWithJulia.jl`). It is for personal study use — not intended as a PR back upstream. During the transition both `CalculusWithJulia` (upstream, for unported chapters) and `CalculusWithJuliaSquared` (for ported ones) are dependencies; the port is complete when `CalculusWithJulia` and `SymPy` can be removed.

## Working locally

- Content lives in `quarto/` (`.qmd` files); build with `quarto render` / preview with `quarto preview` from that directory
- Spell check locally with [`typos`](https://github.com/crate-ci/typos) (`brew install typos-cli`; config in `_typos.toml`) — run it before committing; there is no CI spell check
