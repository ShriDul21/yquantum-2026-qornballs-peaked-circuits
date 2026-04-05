# yquantum-2026-qornballs-peaked-circuits

Notebook-based work for BlueQubit's peaked-circuits challenge.

## Main File

This repo is centered on [`qornballspc.ipynb`](qornballspc.ipynb), which collects several classical attacks for recovering the peak bitstring of challenge circuits.

The notebook is exploratory rather than a packaged library. It mixes a few different approaches in one place:

- BlueQubit MPS sampling
- majority-vote reconstruction from sampled counts
- single-qubit marginal attacks using `Z` expectation values
- approximate transpilation with Qiskit
- CZ-graph component decomposition for Problem 7
- PyZX simplification plus MPS simulation for larger instances

## What The Notebook Covers

### 1. BlueQubit MPS Sampling

The notebook loads a QASM circuit, runs it with BlueQubit's `mps.cpu` backend, and inspects sampled counts. It then compares:

- the most frequent sampled bitstring
- a majority-vote bitstring built from per-position sample frequencies

### 2. Marginal Attack

After removing final measurements, the notebook builds all single-qubit `Z` observables and evaluates their expectation values. The candidate peak bitstring is reconstructed by taking:

- `0` when `<Z_i>` is positive
- `1` when `<Z_i>` is negative

This is done once with BlueQubit MPS and again after approximate transpilation using BlueQubit's `pauli-path` device.

### 3. Problem 7 Component Attack

The notebook defines a small in-notebook solver for `P7_rolling_ridge.qasm`:

- build a graph using only `cz` gates
- find connected components of that graph
- extract one subcircuit per component
- solve each component exactly with Qiskit's `Statevector`
- stitch the component bitstrings back into a full candidate bitstring

### 4. Problems 8-9 PyZX Pipeline

For larger circuits, the notebook switches to a PyZX-assisted workflow:

- load QASM and strip barriers
- simplify with PyZX
- transpile with Qiskit
- measure and simulate with Aer MPS
- return the most frequent sampled bitstring as the candidate peak

The notebook includes helper functions for trying different PyZX simplification strategies such as:

- `clifford_simp`
- `spider_simp`
- `full_reduce`
- `teleport_reduce`

## Expected Input Files

The notebook refers to challenge QASM files such as:

- `P7_rolling_ridge.qasm`
- `P8_bold_peak.qasm`
- `P9_grand_summit.qasm`

Those files are not included in this repo, so you need them in the working directory before running the corresponding cells.

## Requirements

The notebook imports or uses:

- `bluequbit`
- `qiskit`
- `numpy`
- `pyzx`
- `qiskit-aer`
- `jupyter`

Some cells also require a working BlueQubit login via:

```python
import bluequbit
bq = bluequbit.init()
```

## Running

From the repository root:

```bash
jupyter lab
```

Then open [`qornballspc.ipynb`](qornballspc.ipynb) and run the cells in order.

## Notes

- The notebook is an experiment log, so helper functions live inside notebook cells instead of standalone Python modules.
- Some attacks are local-only, while others depend on BlueQubit services.
- Bit ordering matters: some helper code explicitly reverses bitstrings to move between Qiskit order and qubit index order.
