# CalculusWithJuliaSquaredNotes

A personal fork of [CalculusWithJuliaNotes.jl](https://github.com/jverzani/CalculusWithJuliaNotes.jl) — John Verzani's excellent "Calculus with Julia" notes, readable at [calculuswithjulia.github.io](https://calculuswithjulia.github.io/).

This fork exists to (gradually, chapter by chapter) port the notes' code off Python-backed `SymPy` onto pure-Julia [`Symbolics.jl`](https://github.com/JuliaSymbolics/Symbolics.jl)/[`Nemo.jl`](https://github.com/Nemocas/Nemo.jl), using the companion package [`CalculusWithJuliaSquared.jl`](https://github.com/FourMInfo/CalculusWithJuliaSquared.jl) (itself a pure-Julia fork of `CalculusWithJulia.jl`). It is for personal study use — not intended as a PR back upstream. During the transition both `CalculusWithJulia` (upstream, for unported chapters) and `CalculusWithJuliaSquared` (for ported ones) are dependencies; the port is complete when `CalculusWithJulia` and `SymPy` can be removed.

## Working locally

- Content lives in `quarto/` (`.qmd` files); build with `quarto render` / preview with `quarto preview` from that directory
- **View built pages over HTTP, not by opening the file** — see below
- Interactive figures need the global `Plotly`, which Quarto's requirejs otherwise prevents plotly from defining; `_quarto.yml` works around this via `include-in-header`. If `plotly()` figures ever go blank while GR/static ones still show, check that workaround first (see `.github/instructions/porting.instructions.md`, "Blank figure gaps")
- Spell check locally with [`typos`](https://github.com/crate-ci/typos) (`brew install typos-cli`; config in `_typos.toml`) — run it before committing; there is no CI spell check

## Viewing rendered pages

**Always view built pages over HTTP — never by opening the `.html` from disk.** Interactive (`plotly()`) figures are JavaScript; a page can look complete in the HTML source and still show blank gaps in a browser. Only a served page proves it actually renders.

Rendered output lands in `quarto/_book/` (gitignored). Serve it with [LiveServer.jl](https://github.com/tlienart/LiveServer.jl) — the same tool used for Documenter previews in the other study repos (see the `documenter-jl-conventions` skill), which also auto-refreshes when you re-render:

```bash
# one-time: a shared env for the tool, so no repo dependency is added
julia --project=@liveserver -e 'using Pkg; Pkg.add("LiveServer")'

# serve the built book (run from the repo root)
julia --project=@liveserver -e 'using LiveServer; serve(dir="quarto/_book", port=8001)'
```

Then open <http://localhost:8001>. Re-run `quarto render` to rebuild; LiveServer picks up the change and refreshes the page.

Two alternatives: `quarto preview` from `quarto/` serves *and* rebuilds on edit (best while actively writing a chapter, but it re-executes cells, which is slow for heavy chapters); or, with no Julia startup at all, `python3 -m http.server 8765 --bind 127.0.0.1` from `quarto/_book/`.
