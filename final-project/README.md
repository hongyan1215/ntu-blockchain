# Topic J — Perpetual DEX Liquidation Mechanisms

Comparative analysis of **GMX**, **dYdX v4**, and **Hyperliquid**.

## Layout

```
report/
├── report/report.md           # Main report (Markdown)
├── slides/slides.md           # Slidev presentation
├── script/video-script.md     # Demo video script
├── figures/                   # Mermaid sources (.mmd) + rendered SVGs
├── assets/                    # Misc assets (logos, exports)
└── refs.md                    # Citation key → source path mapping
```

## Build commands

### 1. Render Mermaid figures to SVG

```bash
# requires Node.js
npx -y @mermaid-js/mermaid-cli -i figures/gmx-liquidation-flow.mmd        -o figures/gmx-liquidation-flow.svg
npx -y @mermaid-js/mermaid-cli -i figures/architecture-matrix.mmd         -o figures/architecture-matrix.svg
npx -y @mermaid-js/mermaid-cli -i figures/liquidation-waterfall.mmd       -o figures/liquidation-waterfall.svg
```

Or render all at once:

```bash
for f in figures/*.mmd; do
  npx -y @mermaid-js/mermaid-cli -i "$f" -o "${f%.mmd}.svg"
done
```

### 2. Build report PDF (Pandoc + tectonic)

```bash
brew install pandoc tectonic   # macOS
cd report
pandoc report.md \
  -o report.pdf \
  --pdf-engine=tectonic \
  --toc --toc-depth=2 \
  -V geometry:margin=1in \
  -V mainfont="Helvetica" \
  -V monofont="Menlo"
```

Alternative (zero-CLI): open `report.md` in VS Code, install the **Markdown PDF** extension, run `Markdown PDF: Export (pdf)`.

### 3. Build slides

```bash
cd slides
pnpm i
pnpm slidev export slides.md   # produces slides-export.pdf
pnpm slidev slides.md          # live preview
```

## Rubric coverage checklist

| Rubric bullet                                                   | Section in `report.md`                              |
| --------------------------------------------------------------- | --------------------------------------------------- |
| Architecture (custody, settlement)                              | §3 Platform Architectures                           |
| Leverage design                                                 | §4 Leverage & Margin                                |
| Liquidation logic (executor, formula, penalty)                  | §5 Liquidation Logic                                |
| Liquidator rewards                                              | §5 (Liquidator Incentives subsection)               |
| Oracle frequency & reaction                                     | §6 Oracle Design                                    |
| Extreme-volatility defenses (delay auction, backstop, breakers) | §7 Extreme-Volatility Defenses                      |
| GMX liquidation flowchart                                       | §9 GMX Deep-Dive (Mermaid + worked example)         |
| GLP / GM LP impact                                              | §9 (LP Settlement subsection)                       |
| Key risks & future improvements                                 | §11 Risks & Future Directions                       |
| Quality / readability                                           | overall structure, tables, figures, consistent refs |
| Extras (bonus)                                                  | §10 Historical Incidents (JELLY, LUNA, FTX, 3-12)   |

## Citation convention

Every quantitative claim cites a key from [refs.md](refs.md) in square brackets, e.g. `[GMX-LIQ]`. The key resolves to a specific markdown file in the crawled documentation.
