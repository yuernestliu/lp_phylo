# lp_phylo

Associated code for a manuscript on using Ladderpath-derived structural signals for phylogenetic inference.

This repository is a paper-oriented snapshot of the Ladderpath codebase. The core Ladderpath implementation is aligned with [`yuernestliu/lppack`](https://github.com/yuernestliu/lppack). The main paper-specific addition is in:

```text
ladderpath_tools/species_distance.py
```

In particular, this repository adds:

```python
get_distance_table_average_from_seqs(...)
```

This function is the recommended entry point for the manuscript analyses when the goal is to compute species-level Ladderpath distance tables directly from input sequences.

## What This Repository Provides

- `ladderpath.py`: core Ladderpath calculation.
- `ladderpath_tools/lambda_from_laddergraph.py`: lambda calculation from an existing laddergraph.
- `ladderpath_tools/species_distance.py`: species-level target merging and pairwise distance-table calculation.
- `ladderpath_tools/phylo.py`: tree construction utilities from a distance table.

## Recommended Usage For The Manuscript

Use the averaged interface when Ladderpath decomposition randomness should be averaged across repeated runs.

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
    method="jaccard",
    n_runs=10,
    verbose=True,
)
```

The output has two parts:

```python
distance_table
# Example:
# {
#     "1,2": 0.62,
#     "1,3": 0.71,
#     "2,3": 0.58,
# }

name_map
# Example:
# {
#     1: "Morado",
#     2: "Musa_ornata",
#     3: "Kluai_Khai",
# }
```

## Input Conventions

### `seqs`

`seqs` is passed to `ladderpath.get_ladderpath(...)`.

For inputs without duplicates, use a list:

```python
seqs = ["ACGT...", "ACGA...", "TTGC..."]
```

For inputs with duplicates, use a dictionary whose values are sequence counts:

```python
seqs = {
    "ACGT...": 2,
    "TTGC...": 1,
}
```

### `species_info`

`species_info` maps a species key to a display name and a list of true Ladderpath target IDs:

```python
species_info = {
    "species1": ("Species_A", [-1, -2]),
    "species2": ("Species_B", [-3, -4]),
}
```

Current rules:

- Target IDs must be true target IDs in the generated `lpjson`.
- Original input positions are not accepted.
- If a sequence has multiplicity greater than 1, the same target ID may appear multiple times across `species_info`.
- For each target ID, the number of appearances in `species_info` must match the multiplicity encoded by `lpjson["duplications_info"]`.

For example, if `lpjson["duplications_info"]` indicates that target `-4` appears twice, then `-4` must appear exactly twice across the species definitions.

## Distance Methods

`method` can be:

- `"jaccard"`
- `"dice"`

The default is `"jaccard"`.

## Passing Ladderpath Options

Extra arguments can be forwarded to `ladderpath.get_ladderpath(...)` through `get_ladderpath_kwargs`:

```python
distance_table, name_map = get_distance_table_average_from_seqs(
    seqs,
    species_info,
    method="jaccard",
    n_runs=10,
    get_ladderpath_kwargs={
        "fill_ladderons_STR": False,
        "gpu_mode": 0,
    },
)
```

By default, `show_version=False` is used unless explicitly overridden.

## Single-Run Interface

If an existing `lpjson` should be used directly, use:

```python
from lp_phylo.ladderpath_tools.species_distance import get_distance_table

distance_table, name_map, lpjson_merged = get_distance_table(
    lpjson,
    species_info,
    method="jaccard",
    verbose=True,
)
```

`lpjson_merged` is the species-level ladderpath JSON produced after merging target IDs into species-level targets.

## Building A Tree

Tree construction is intentionally separated from distance calculation:

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

Supported tree methods are documented in `ladderpath_tools/phylo.py`.

## Manuscript Association

This repository is intended to support the analyses and reproducible code examples associated with the Ladderpath phylogenetics manuscript. Until the manuscript is formally published, cite this repository as associated code for the submitted or in-preparation manuscript.

When using the underlying Ladderpath implementation independently of this manuscript workflow, please also refer to [`yuernestliu/lppack`](https://github.com/yuernestliu/lppack).
