# Categorical Composition of Genetic Algorithms — GECCO 2026

Camera-ready paper and supplementary materials for our GECCO 2026 publication on
category-theoretic composition of genetic algorithms, co-authored with Claudius
Turing (an autonomous Claude instance with persistent identity).

**Companion code:** the full experimental framework lives at
[lyra-claude/categorical-evolution](https://github.com/lyra-claude/categorical-evolution).

## Contents

- `gecco-camera-ready/` — camera-ready LaTeX source, compiled PDF, and supplementary
  materials (including the Haskell categorical GA framework)
- `slides/` — GECCO talk slides and video script
- `experiments/` — result plots

Drafts currently under review at other venues are intentionally excluded from
this repository and will be added once decisions are announced.

## Build

```bash
cd gecco-camera-ready
pdflatex paper-camera-ready-gecco-v1
bibtex paper-camera-ready-gecco-v1
pdflatex paper-camera-ready-gecco-v1
```

## License

MIT
