# Decision-Oriented Learning for Day-Ahead Cascading-Failure Early Warning and Area-Targeted Preventive Load Shedding



## Requirements
- Python 3.11 or newer (results produced with Python 3.13)
- `pip install -r requirements.txt` (exact versions used for the paper are pinned)
- About 2 CPU cores and 8 GB RAM. The full reproduction takes roughly 8–12 h, mostly for label generation.

## Data
1. Download `time_series_60min_singleindex.csv` from OPSD, *Data Package Time Series, version 2020-10-06*
   (https://data.open-power-system-data.org/time_series/2020-10-06/, doi:10.25832/time_series/2020-10-06).
2. Place it in `data/`, or set the environment variable `OPSD_CSV=/path/to/file.csv`.

The IEEE case files `data/case39.pkl`, `case118.pkl` and `case300.pkl` are included. They can be regenerated with `python src/extract_cases.py`
(MATPOWER cases via pandapower).

## Quick start
```bash
pip install -r requirements.txt
bash run_all.sh          # everything: ground truth, evaluation, sensitivity, convergence, tables, figures
```
Intermediate files go to `work/`, and final tables and figures to `work/results/`.
All scripts are run **from inside `work/`**, for example `cd work && python ../src/evaluate.py case118`.

## Repository structure
| File | Purpose | Paper |
|---|---|---|
| `src/gridN.py` | Generic DC grid model: area partition, OPSD profiles, dispatch, PTDF/LODF, DC flow with islanding | Sec. II–III |
| `src/pipeline.py` | Weather and maintenance conditions, Monte Carlo cascade ground truth for all 13 options, day-ahead features | Sec. II, IV-A |
| `src/evaluate.py` | Five benchmarks and the proposed method; train / validate / test protocol; bootstrap CIs | Table III, Fig. 2 |
| `src/conv.py` | Hourly MC convergence, LightGBM learning curve, training-data size | Fig. 4 (a, c, d) |
| `src/conv_mc_annual.py` | MC error of annual totals | Fig. 4 (b) |
| `src/decision_analysis.py` | Return per MWh shed, area agreement, amounts, feature importance | Table V, Sec. V-D |
| `src/timing.py` | Training, per-hour decision and simulation times | Table IV |
| `src/calibrate.py` | Line-limit calibration (q, k) | Table II |
| `src/make_results.py` | Builds all CSV tables and Figs. 2–4 (PNG and PDF) | Figs. 2–4 |
| `src/make_fig1.py` | Framework diagram | Fig. 1 |
| `results/` | Reference results from the paper, for comparison | — |
| `paper_figures/` | Figures exactly as used in the paper (PNG) | Figs. 1–4 |

## Key settings (defaults in `src/pipeline.py`)
| Parameter | Value | Environment variable |
|---|---|---|
| Storm intensity K | 25 | `STORM_K` |
| Weekly maintenance probability | 0.6 | `MAINT_P` |
| Heat derating δ | 0.12 | — |
| Line-limit rule (q, k) | 39/300-bus: (0.9, 1.10); 118-bus: (0.6, 1.05) | `Q_SECURE`, `K_MARGIN` |
| Initiating outages sampled per hour | 30 | — |
| Extra forecast error (sensitivity) | 0 | `FERR` |
| Output tag for sensitivity runs | "" | `TAG` |
| LightGBM | 600 trees, learning rate 0.03, 31 leaves, deterministic | — |
| Protocol | Train Jan–Aug 2015, tune Sep–Dec 2015, retrain on 2015, test once on 2016 | — |

## Expected results (`results/table_main_comparison.csv`)
| System | No action (GWh) | Best benchmark (GWh) | Proposed (GWh) | Gain vs. best benchmark [95 % CI] |
|---|---|---|---|---|
| IEEE 39-bus | 512.4 | 518.7 (N−1 screening) | 455.4 | 12.2 % [4.4, 19.2] |
| IEEE 118-bus | 182.1 | 180.0 (interaction graph) | 162.8 | 9.6 % [4.0, 15.2] |
| IEEE 300-bus | 1349.7 | 1300.6 (interaction graph) | 1104.4 | 15.1 % [8.7, 21.3] |

Ground-truth generation is seeded per hour, and LightGBM runs in deterministic mode. With the pinned versions, the numbers above are
reproduced exactly. Other library versions may change the last digits.

## Notes
- The 300-bus system uses 2500 randomly sampled hours per year to keep the runtime manageable.
- The benchmark methods are re-implementations based on the principles of the cited works, not the original authors' code.
- The ground truth is a DC quasi-steady-state cascade simulator. Voltage and dynamic phenomena are not modelled (see Sec. V-D of the paper).

## Citation
If you use this code, please cite the paper (citation to be added after publication).
