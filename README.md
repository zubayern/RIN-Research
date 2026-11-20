# Time-Resolved Residue Interaction Network Analysis of Protein Folding

Dynamic residue interaction network (RIN) analysis of protein folding molecular dynamics (MD) trajectories, using graph centrality and quantum-walk-inspired frameworks to track how individual residues gain or lose structural importance as a protein folds.

---

## Overview

Most RIN analyses are **static**: they build one network from one PDB structure. Protein folding is a process in time, so a single snapshot misses how the interaction network reorganizes along the way.

This project extends two published frameworks into the time domain:

- **RinQ** (*Materials Today Quantum*, 2025)
- **Continuous-Time Quantum Walk (CTQW) Centrality** (July 2025)

Instead of one network per protein, a RIN is built for **every frame** of an MD trajectory. Several centrality measures are computed per residue per frame, which produces time series that show:

- which residues act as persistent hubs,
- which residues gain or lose importance during folding,
- how overall network density and native-contact fraction evolve.

The pipeline is designed to be reproducible, modular, and runnable in Google Colab.

---

## Methods

### 1. Network construction

For each frame:

1. Residue positions are represented by their Cα atoms.
2. Residues are connected if their Cα–Cα distance is within an **8 Å cutoff**.
3. Edges are weighted by **1/d²**, where *d* is the Cα–Cα distance.
4. A weighted adjacency matrix is built from the resulting graph.

Network construction uses the original functions from the CTQW RIN reference notebook (`calculate_distance`, `construct_rin`, `create_weighted_adjacency_matrix`, `eigenvector_centrality`).

### 2. Centrality measures

Eight measures are computed per residue per frame:

| Measure | Measure |
|---|---|
| Degree | Katz |
| Eigenvector | PageRank |
| Closeness | Harmonic |
| Betweenness | Load |

### 3. Trajectory preprocessing

Raw MD trajectories can contain periodic boundary condition (PBC) artifacts. Before any frame-level analysis, trajectories are processed with MDAnalysis `unwrap()` and `fit_rot_trans()`.

> **Why this matters:** without this step, RMSD values for Chignolin reached roughly 44 Å, a purely artificial result of molecules crossing periodic box boundaries. After correction, RMSD behaved as expected.

### 4. Additional quantities

- **Q**: fraction of native contacts per frame
- **Network density**: average number of edges per frame
- **RMSD**: after PBC correction and alignment

---



### Dataset inclusion criteria

A dataset is only used if it meets all of the following:

- AMBER or CHARMM force field
- Explicit solvent (TIP3P or SPC/E)
- `.xtc` or `.dcd` trajectory with a matching topology
- Documented folding events
- Convertible to PDB

---


## Repository workflow

The analysis is organized as a modular **13-cell Google Colab notebook**. Each cell is an isolated unit (load, preprocess, build networks, compute centralities, plot), so every step can be inspected and re-run on its own.

Outputs are saved to Google Drive in this layout:

```
RIN_Research/
└── <ProteinName>/
    └── <RunName>/
        └── (centrality tables, plots, metrics)
```

### Requirements

- Python 3
- [MDAnalysis](https://www.mdanalysis.org/) 2.10.0
- [NetworkX](https://networkx.org/)
- RINpy
- NumPy, Matplotlib
- Google Colab (primary environment)

Optional visualization: [Mol\*](https://molstar.org/) and [UCSF ChimeraX](https://www.cgl.ucsf.edu/chimerax/)

### Running the analysis

1. Open the Colab notebook.
2. Mount Google Drive and point the notebook to your trajectory (`.xtc` or `.dcd`) and topology files.
3. Run the cells in order. Preprocessing (`unwrap()` and `fit_rot_trans()`) must run before any network analysis.
4. Find results under `RIN_Research/<ProteinName>/<RunName>/` in Drive.

---



## Methodological notes

- **Validation should test, not confirm.** Checks are written to verify calculations independently, not to reproduce expected values.
- **Preprocess before analyzing.** PBC artifacts can silently corrupt frame-level metrics.
- **State the contribution clearly.** The novelty here is the dynamic, time-resolved extension of static PDB-based RIN analysis, not reprocessing existing datasets.

---

## References

1. RinQ. *Materials Today Quantum*, 2025.
2. Continuous-Time Quantum Walk (CTQW) Centrality, July 2025.
3. Lindorff-Larsen, K., et al. How fast-folding proteins fold. *Science*, 2011.
4. mdCATH dataset. *Scientific Data*, 2024.
5. Chignolin trajectories: Shinji Iida dataset, Zenodo 6349893.
6. SST2 trajectories: Zenodo 13772542.

---

## Acknowledgments

This work is a volunteer research contribution to a computational biology collaboration on protein–ligand interaction and quantum-mechanical frameworks, supervised by **Shah Ishmam Mohtashim** (North Carolina State University / Purdue University).

## Author

**Zubayer**, Department of Microbiology, Independent University, Bangladesh (IUB), Dhaka

