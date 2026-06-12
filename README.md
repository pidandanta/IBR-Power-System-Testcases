# IBR Power System Test Cases

Two benchmark power system models used for inverter-based resource (IBR) studies, specifically for grid-forming (GFM) and grid-following (GFL) converter placement research.  
These test cases are used in the paper **"Quantifying Voltage–Frequency Coupling via Block Recurrence-Laplacian for Multi-Objective GFM Siting"** (IEEE TSG / Applied Energy, under review).

---

## Repository Structure

```
IBR-Power-System-Testcases/
├── ieee39-bus/
│   ├── ieee39bus.raw              # PSS/E power flow data (IEEE 39-bus)
│   └── ieee39bus_dynamics.xls     # Dynamic model (REGCA/REECB/REPCA IBR models)
└── wecc240-bus/
    ├── wecc240bus.raw              # PSS/E power flow data (WECC 240-bus synthetic)
    ├── wecc240bus_baseline.xls     # All-SG baseline dynamic configuration
    ├── wecc240bus_ibr50pct.xls     # 50% IBR penetration (REGCA/REECB/REPCA)
    ├── wecc240bus_ibr80pct.xls     # 80% IBR penetration
    └── wecc240bus_ibr100pct.xls    # 100% IBR penetration
```

---

## Test System Descriptions

### IEEE 39-Bus System (`ieee39-bus/`)

The standard New England IEEE 39-bus system with 10 synchronous generators and 39 buses.  
In the IBR study, **all 10 synchronous generators (buses 30–39) are replaced by IBRs** modelled with the WECC generic IBR chain (REGCA + REECB + REPCA).

| Property | Value |
|----------|-------|
| Buses | 39 |
| Lines | 46 |
| IBR buses | 10 (buses 30–39) |
| Baseline GFM | buses 30–33 (40% GFM penetration) |
| Baseline GFL | buses 34–39 |
| Simulation step | 0.125 ms |

**GFM vs GFL distinction** (REECB parameter `Kqv`):  
- GFM: `Kqv = 0.5` (reactive droop, voltage-forming)  
- GFL: `Kqv = 0.0` (current-controlled, PLL-synchronized)

**Dynamic model file** (`ieee39bus_dynamics.xls`) contains REGCA, REECB, and REPCA model parameters for all 10 IBR units compatible with the [ParaEMT](https://github.com/NREL/ParaEMT_public) EMT simulation framework.

---

### WECC 240-Bus Synthetic System (`wecc240-bus/`)

A synthetic 240-bus model based on the Western Electricity Coordinating Council (WECC) footprint, originally published by the [NREL/ACM synthetic grid project](https://www.nrel.gov/grid/synthetic-grids.html).

| Property | Value |
|----------|-------|
| Buses | 243 |
| Lines | 329 |
| Total dynamic units | 202 |
| Synchronous condensers retained | 5 |
| IBR units | 197 |
| Simulation step | 0.25 ms |

Three IBR penetration scenarios are provided:

| File | IBR penetration | GFM count | GFL count |
|------|----------------|-----------|-----------|
| `wecc240bus_ibr50pct.xls` | 50% (≈ 99 units) | variable | variable |
| `wecc240bus_ibr80pct.xls` | 80% (≈ 158 units) | variable | variable |
| `wecc240bus_ibr100pct.xls` | 100% (197 units) | variable | variable |

All IBR units use the REGCA–REECB–REPCA control chain. GFM/GFL mode is selected by the `Kqv` parameter in REECB (same convention as the 39-bus system).

---

## File Formats

| Extension | Format | Description |
|-----------|--------|-------------|
| `.raw` | PSS/E v33 RAW | Power flow network data (buses, branches, generators, loads) |
| `.xls` | Excel (Bxlsx) | Dynamic model parameters for REGCA, REECB, REPCA controllers |

These files are intended for use with the [ParaEMT](https://github.com/NREL/ParaEMT_public) parallel EMT simulation tool (Python, NUMBA-accelerated).

---

## How to Run a Simulation

1. Install [ParaEMT](https://github.com/NREL/ParaEMT_public) and its dependencies.
2. Copy the `.raw` and `.xls` files into the ParaEMT `models/` directory.
3. Run the power flow initialisation:
   ```bash
   python main_step0_powerflow.py
   ```
4. Run the EMT simulation with a step-load disturbance:
   ```bash
   python main_step1_simulation.py
   ```
5. Post-process results:
   ```bash
   python main_step2_saveresults.py
   ```

Refer to the ParaEMT documentation for detailed parameter configuration.

---

## GFM/GFL Assignment Convention

To switch a unit between GFM and GFL mode, modify the `Kqv` column in the REECB sheet of the `.xls` file:

```
Kqv = 0.5  →  Grid-Forming (GFM): virtual voltage source with reactive droop
Kqv = 0.0  →  Grid-Following (GFL): current-controlled with PLL synchronization
```

This convention follows Singhal et al. (IEEE Trans. Smart Grid, 2022) and is used throughout the BRL–PI-GNN siting experiments.

---

## Citation

If you use these test cases in your research, please cite:

```bibtex
@article{shan2025brl,
  title   = {Quantifying Voltage--Frequency Coupling via Block Recurrence-Laplacian
             for Multi-Objective {GFM} Siting},
  author  = {Shan, Sicheng and Du, Zhaobin},
  journal = {IEEE Transactions on Smart Grid},
  year    = {2025},
  note    = {under review}
}
```

---

## License

The IEEE 39-bus power flow data is in the public domain (standard benchmark).  
The WECC 240-bus synthetic model is derived from publicly available NREL synthetic grid data.  
The dynamic model parameter files (`.xls`) are released under the [MIT License](LICENSE).
