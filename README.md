# Scientific Agent Skills — Curated Fork

[![Skills](https://img.shields.io/badge/Skills-88-brightgreen.svg)](docs/skills.md)
[![License: MIT](https://img.shields.io/badge/Repository%20License-MIT-yellow.svg)](LICENSE.md)
[![Agent Skills](https://img.shields.io/badge/Format-Agent%20Skills-blueviolet.svg)](https://agentskills.io/)

This is an **unofficial, personally curated fork** of [K-Dense Scientific Agent Skills](https://github.com/K-Dense-AI/scientific-agent-skills). It contains **88 research skills** selected from the upstream collection, with emphasis on mathematical computing, optimization, scientific machine learning, simulations, scholarly research, visualization, and writing.

The original K-Dense project and its authors remain credited. This fork is not an official K-Dense release. The upstream project is the authoritative source for removed skills, upstream updates, and compatibility information.

## Browse the skills

| Area | Skills | Explore |
| --- | ---: | --- |
| Scientific Databases & Data Access | 3 | [Catalog](docs/skills.md) |
| Scientific Integrations | 2 | [Catalog](docs/skills.md) |
| Scientific Packages | 66 | [Catalog](docs/skills.md) |
| Scientific Thinking & Analysis | 17 | [Catalog](docs/skills.md) |

- **[All 88 skills, with descriptions](docs/skills.md)** — technical capability and setup documentation.
- **[Skill guides](docs/skill-guides/README.md)** — a guide for each retained skill (images are served from a pinned upstream commit).
- **[Research workflow examples](docs/examples.md)** — 11 examples that refer exclusively to retained skills.
- **[Security report and caveats](docs/security-report.md)** — historical scan information; **not a current scan of this subset**.

Common starting points:

| Need | Skills |
| --- | --- |
| Symbolic mathematics, optimization, probability | [SymPy](skills/sympy/SKILL.md), [pymoo](skills/pymoo/SKILL.md), [PyMC](skills/pymc/SKILL.md) |
| Numerical and physical simulation | [MATLAB](skills/matlab/SKILL.md), [FluidSim](skills/fluidsim/SKILL.md), [PyBaMM](skills/pybamm/SKILL.md) |
| Machine learning and deep learning | [scikit-learn](skills/scikit-learn/SKILL.md), [PyTorch Lightning](skills/pytorch-lightning/SKILL.md), [PyTorch Geometric](skills/torch-geometric/SKILL.md) |
| Large datasets, GPU computing | [Dask](skills/dask/SKILL.md), [Polars](skills/polars/SKILL.md), [Zarr](skills/zarr-python/SKILL.md), [GPU Optimization](skills/optimize-for-gpu/SKILL.md) |
| Scholarly papers and citations | [Paper Lookup](skills/paper-lookup/SKILL.md), [Literature Review](skills/literature-review/SKILL.md), [Citation Management](skills/citation-management/SKILL.md) |
| Research figures, talks and publication | [Scientific Visualization](skills/scientific-visualization/SKILL.md), [Scientific Slides](skills/scientific-slides/SKILL.md), [Scientific Writing](skills/scientific-writing/SKILL.md) |

## Get started

This repo follows the open [Agent Skills](https://agentskills.io/) folder convention: `skills/<name>/SKILL.md` with optional `scripts/`, `references/`, and `assets/`.

To download only the current curated source tree:

```bash
git clone --depth 1 https://github.com/riprayx/scientific-agent-skills.git
```

Install or expose individual folders under `skills/` according to your agent host's documentation. Agent-plugin-compatible hosts may also use the repository's [`plugin.json`](plugin.json). **A Skill is guidance and optional helper code, not a promise that its Python packages or third-party APIs have already been installed.** Some Skills require external API keys, GPUs, specialized software, or authorization.

## Testing and safety

The repository retains the checks and test suites belonging to the 88 included Skills. Repository-wide contract checks can be run from the project environment:

```bash
uv run --with pytest python -m pytest tests/_meta -q
```

For full isolated skill test workflows, see [`tests/run_all.py`](tests/run_all.py) and [`tests/skill-requirements.toml`](tests/skill-requirements.toml). Native toolchains and credentials may be needed for selected integrations.

**Important:** `docs/security-report.md` and its JSON companion were generated before this collection was reduced. They are retained as upstream-era evidence and must not be interpreted as validation of the new subset. Review every executable helper and its external destinations before running against private research files or API keys. Regenerate the scan after merging this cleanup.

## Upstream, licensing and attribution

- Original project: [K-Dense-AI/scientific-agent-skills](https://github.com/K-Dense-AI/scientific-agent-skills)
- Citation metadata: [CITATION.cff](CITATION.cff)
- Repository license: [LICENSE.md](LICENSE.md)
- **Individual Skill licenses may differ**; check each `SKILL.md` before redistribution or use.
- Removing Skills from this fork does not remove them from upstream and does not rewrite Git history.

This curation deliberately preserves the scope and content of each retained Skill instead of rewriting its scientific guidance.
