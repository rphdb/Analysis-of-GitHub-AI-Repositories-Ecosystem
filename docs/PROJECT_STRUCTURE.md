# Project Structure

All scripts are meant to be run from the repository root.

## scripts/

### Main pipeline
- `build_repository_backbone.py`: GitHub GraphQL retrieval, mixed-weight repository graph (fork, README, dependency signals), backbone extraction, Louvain communities and centrality. Supports single-signal modes (`--signal-mode fork|readme|mixed`).
- `analyze_company_communities.py`: aggregates the repository graph into a company-level network and detects company communities.
- `prepare_visual_backbone.py`: visualization-oriented sparsification and community metagraph export for Gephi.
- `seed_community_layout.py`: injects community-separated coordinates into a GEXF graph for Gephi layouts.

### Single-signal graph builders (method development, 100 + 100 sample)
- `build_repository_backbone_forking.py`: graph from fork-owner Jaccard similarity.
- `build_repository_backbone_readme.py`: graph from README TF-IDF cosine similarity.
- `build_repository_backbone_readme_beta.py`: README graph restricted to AI vocabulary (uses the Wikidata concept list).
- `build_repository_backbone_hybrid.py`: combination of the fork and README signals.

### Analysis
- `threshold_analysis.py`: influence of the similarity threshold on graph size and edge-weight distribution.
- `basic_graph_analysis.py`: basic graph metrics and centrality distributions.
- `assign_labels.py`: assigns AI-topic labels to repositories and computes the empirical purity of Louvain communities.
- `dictionnary_testor.py`: builds the AI concept dictionary from Wikidata (SPARQL) and a curated glossary.
- `hypothesis_tester.py`: statistical tests on centrality (cross-domain vs single-domain repositories, centrality vs stars).

## data/raw/
- `repo_raw_data_fork_top100.json`: GitHub data, 100 + 100 sample.
- `repo_raw_data_fork.json`: GitHub data, 500 + 500 sample.
- `owner_company_cache.json`: cached owner-to-company lookups.
- `wikidata_ai_concepts.json`: AI concepts retrieved from Wikidata by `dictionnary_testor.py`.

## outputs/
- `graphs/repo/`: repository networks and visualization variants.
- `graphs/company/`: company networks and visualization variants.
- `graphs/repo_signal_compare/`: fork-only, README-only and mixed-weight graphs used in the signal ablation.
- `tables/`: company nodes, company communities and owner-to-company mapping (CSV).

## reports/
- `final_report.tex` / `final_report.pdf`: final report.
- `figures/`: figures used in the report and README.
