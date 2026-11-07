# Network Analysis of the GitHub AI Open-Source Ecosystem

How is the open-source AI ecosystem structured, and who connects it? This project builds weighted similarity graphs over the most-starred computer vision and NLP repositories on GitHub, extracts their backbone, detects communities, and aggregates the result into a company-level network.

Team project for the *Graph Analysis* course (MODAL CSC_43M02_EP) at École Polytechnique, April 2026, by **Raphael des Boscs** and **Xufan Huang**.

📄 **Full report:** [`reports/final_report.pdf`](reports/final_report.pdf)

![Company-level community graph](reports/figures/company_analysis.png)
*Company-level network (451 companies, 36 communities). Node color = community, node size = activity.*

## Key findings

- **Fork overlap is the most informative signal.** Linking repositories through shared fork owners gives the clearest community structure (modularity 0.40), while README text similarity links almost everything through generic AI vocabulary (modularity 0.15). The final graph mixes both, plus dependency overlap, with weights 0.6 / 0.2 / 0.2.
- **The ecosystem has four tightly linked cores.** On ~1,000 repositories (967 after deduplication) aggregated to 451 companies, four communities carry about 97% of cross-community weight: research platforms (Ai2, Google Research), industrial CV and infrastructure (Microsoft, NVIDIA), model hubs and NLP (Hugging Face, Tencent) and applied tooling (Explosion, Roboflow). A large computer vision community (CVer, DAIR.AI) stays peripheral.
- **Large companies act as bridges.** Companies with more than 3 repositories are significantly more central than small ones on all five centrality metrics (one-sided Mann–Whitney U test, Bonferroni-adjusted p < 0.05). The effect is strongest for betweenness centrality (rank-biserial r = 0.44), even after normalizing edge weights by company size.

## Method

1. **Data collection** through the GitHub GraphQL API: top repositories for the `computer-vision` and `nlp` topics, with README text, dependency files and fork owners. Two samples: 100 + 100 repositories (186 unique) for method development, and 500 + 500 (967 unique) for the company-level analysis.
2. **Repository graph**: edge weight `w = 0.6·fork_jaccard + 0.2·readme_tfidf_cosine + 0.2·dependency_jaccard`.
3. **Backbone extraction**: weight thresholds, optional top-k sparsification and k-core pruning, with an automatic relaxation step that avoids collapsing the graph.
4. **Communities and centrality**: weighted Louvain, PageRank, betweenness and eigenvector centrality.
5. **Signal ablation**: fork-only vs README-only vs mixed graphs, plus threshold sensitivity analysis.
6. **Company aggregation**: owners mapped to organizations (or the user's profile company), edges summed and normalized by `sqrt(n_a · n_b)`, then Louvain again.
7. **Hypothesis tests** on centrality (large vs small companies; cross-domain vs single-domain repositories; centrality vs stars).

## Repository structure

| Path | Content |
|---|---|
| `scripts/` | Python pipelines and analysis scripts (detailed in [`docs/PROJECT_STRUCTURE.md`](docs/PROJECT_STRUCTURE.md)) |
| `data/raw/` | Cached GitHub API data, owner-to-company cache, Wikidata AI concept list |
| `outputs/graphs/` | Graphs in GEXF format (open with [Gephi](https://gephi.org/)): repository, company and signal-comparison graphs |
| `outputs/tables/` | Company nodes, communities and owner-to-company mapping (CSV) |
| `reports/` | Report (LaTeX source and PDF) and figures |

## Reproducing the results

```bash
pip install -r requirements.txt
export GITHUB_TOKEN=...   # only needed to re-fetch data; cached data is in data/raw/
```

All commands run from the repository root.

**Repository graph (500 + 500 sample)**
```bash
python scripts/build_repository_backbone.py \
  --raw-output data/raw/repo_raw_data_fork.json \
  --gexf-output outputs/graphs/repo/repository_backbone_fork.gexf \
  --per-domain-limit 500 \
  --weight-fork 0.6 --weight-readme 0.2 --weight-dep 0.2
```

**Company network**
```bash
python scripts/analyze_company_communities.py \
  --repo-raw data/raw/repo_raw_data_fork.json \
  --repo-graph outputs/graphs/repo/repository_backbone_fork.gexf \
  --company-graph outputs/graphs/company/company_network.gexf \
  --fetch-owner-company \
  --drop-owners-without-company
```

**Single-signal ablation (100 + 100 sample)**
```bash
python scripts/build_repository_backbone.py --reuse-raw \
  --raw-output data/raw/repo_raw_data_fork_top100.json \
  --gexf-output outputs/graphs/repo_signal_compare/repository_fork_only.gexf \
  --signal-mode fork --pre-min-weight 0.0125 --min-weight 0.0125 \
  --top-k 0 --k-core 0 --no-auto-relax
```
Use `--signal-mode readme` or `--signal-mode mixed` for the other two variants. The report's *Reproducibility* section lists every command, including the Gephi visualization steps.

## Limitations

Top-star sampling over-represents popular repositories; company identities rely on user-provided profile fields; communities depend on backbone parameters; and the analysis is a single snapshot in time.
