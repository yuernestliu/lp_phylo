# lp_phylo

Associated code for a manuscript on using Ladderpath-derived structural signals for phylogenetic inference.

## Quick Start

The recommended manuscript-facing entry point is:

```python
from lp_phylo.ladderpath_tools.species_distance import (
    get_distance_table_average_from_seqs,
)

seqs = {
    "ATGGAGAAATCTCAA": 1,
    "TTTCGAATTCT": 1,
    "AGGATATTTATAAAT": 1,
    "ATCGATTTTGTT": 2,
    "CGGATGACTTCCT": 1,
}

species_info = {
    "species1": ("Morado", [-1, -2]),
    "species2": ("Musa_ornata", [-3, -4]),
    "species3": ("Kluai_Khai", [-5, -4]),
}

distance_table, name_map = get_distance_table_average_from_seqs(
    seqs,
    species_info,
    method="dice",
    n_runs=10,
    verbose=True,
)
```

The function returns:

```python
distance_table
# {"1,2": ..., "1,3": ..., "2,3": ...}

name_map
# {1: "Morado", 2: "Musa_ornata", 3: "Kluai_Khai"}
```

`method` can be `"dice"` or `"jaccard"`. The default is `"dice"`.

An executable notebook is provided at:

```text
examples/lp_phylo_usage.ipynb
```

## Manuscript Data

The data archive associated with the manuscript is provided as:

```text
manuscript_experiment_data.zip
```

After extraction, the archive creates a `Data/` directory containing the datasets and result files used for the manuscript and supplementary analyses. Several subdirectories include their own local `README.md` files describing the corresponding data block.

## Inputs

`seqs` is passed to `ladderpath.get_ladderpath(...)`.

For inputs without duplicates:

```python
seqs = ["ACGT...", "ACGA...", "TTGC..."]
```

For inputs with duplicate sequences:

```python
seqs = {
    "ACGT...": 2,
    "TTGC...": 1,
}
```

`species_info` maps a species key to a display name and a list of target IDs:

```python
species_info = {
    "species1": ("Species_A", [-1, -2]),
    "species2": ("Species_B", [-3, -4]),
}
```

## Build A Tree

Tree construction is separate from distance calculation:

```python
from lp_phylo.ladderpath_tools.phylo import phylo

phylo(
    distance_table,
    name_map,
    method="upgma",
    newick_file_name="my_tree",
    verbose=True,
)
```

## Repository Contents

- `ladderpath.py`: core Ladderpath calculation.
- `ladderpath_tools/lambda_from_laddergraph.py`: lambda calculation from an existing laddergraph.
- `ladderpath_tools/species_distance.py`: species-level target merging and pairwise distance-table calculation.
- `ladderpath_tools/phylo.py`: tree construction utilities from a distance table.
- `examples/lp_phylo_usage.ipynb`: runnable usage example.
- `manuscript_experiment_data.zip`: data archive associated with the manuscript.

The core Ladderpath implementation is aligned with [`yuernestliu/lppack`](https://github.com/yuernestliu/lppack). This repository keeps the paper-specific phylogenetic helper interface used for the manuscript.

## Manuscript Association

This repository is intended to support the analyses and reproducible code examples associated with the Ladderpath phylogenetics manuscript. Until the manuscript is formally published, cite this repository as associated code for the submitted or in-preparation manuscript.

When using the underlying Ladderpath implementation independently of this manuscript workflow, please also refer to [`yuernestliu/lppack`](https://github.com/yuernestliu/lppack).
