# University Electricity Consumption & Carbon Reporting

A two-part final project for the IT Department of Albstadt-Sigmaringen University, simulating a Scope 2 (purchased electricity) sustainability report from scratch — starting from a synthetic hourly consumption dataset and ending with concrete, quantified carbon-reduction strategies.

Pradnya Dipak Nalawade (111372)

## Overview

The university had no historical electricity consumption data at all. This project builds that data from first principles, then combines it with real German grid carbon-intensity data to answer operational questions: how accurate does monitoring need to be, how much would solar panels help, can AI training be scheduled around cleaner grid hours, and what does cutting night-time standby power actually save?

- **Part 1** generates a believable, parameterized hourly electricity consumption dataset for the entire campus for 2024.
- **Part 2** combines that dataset with real carbon-intensity data (Electricity Maps / entsoe.eu) to compute CO₂ emissions and evaluate five reduction strategies.

## Repository Structure

```
.
├── part1/
│   ├── part1.ipynb          # Dataset generation notebook
│   └── consumption.csv      # Output: hourly consumption for 2024 (8,784 rows)
├── part2/
│   ├── part2.ipynb          # Carbon reporting & reduction strategies notebook
│   ├── consumption.csv      # Input: copy of Part 1 output
│   ├── DE_2024_15_minute.csv # Input: German grid carbon intensity, 15-min resolution
│   └── PVGIS_Data.csv       # Input: hourly solar irradiance data (PVGIS, Albstadt-Sigmaringen)
└── README.md
```

## Part 1 — Generating a Believable Dataset

**Goal:** simulate hourly electricity consumption (`timestamp`, `consumption_kwh`) for the whole university campus, for every hour of 2024 (8,784 rows, leap year).

Since no real measurements exist, the dataset is built up from a series of justified assumptions rather than random noise:

| Factor | Approach |
|---|---|
| Daily pattern | Baseline load by time of day — night (250 kWh), morning/evening (450 kWh), afternoon peak (650 kWh) |
| Weekends | Reduced to night-time baseline (250 kWh) |
| Public holidays & semester breaks | Baden-Württemberg 2024 holidays and university closure periods reduced to baseline load |
| Semester vs. break | Spring (Mar 15–Jul 15) and Fall (Oct 1–Dec 20) semesters flagged separately for analysis |
| Lighting | +10% consumption during operating hours (07:00–19:00) when natural daylight is insufficient, using approximate monthly sunrise/sunset times for Albstadt |
| Natural variation | ±10 kWh random fluctuation per hour (fixed seed for reproducibility), clipped at 0 to avoid negative consumption |

**Result:** ~2.97 GWh simulated annual consumption, with visualizations of weekly/monthly/hourly patterns and data-quality checks (no missing values, no duplicate timestamps, all 8,784 hours present).

**Libraries:** `pandas`, `numpy`, `matplotlib`

## Part 2 — Carbon Reporting & Reduction Strategies

**Goal:** compute the university's 2024 CO₂ emissions from purchased electricity and quantify the impact of several reduction strategies.

**Data sources:**
- `consumption.csv` — Part 1 output (hourly, own simulation)
- `DE_2024_15_minute.csv` — German grid carbon intensity, 15-minute resolution, from Electricity Maps (sourced from entsoe.eu); life-cycle gCO₂eq/kWh is used (not direct-only), plus renewable/carbon-free energy share
- `PVGIS_Data.csv` — real hourly solar irradiance for the university's coordinates (48.214°N, 9.026°E), from the PVGIS tool (2023 data, since 2024 wasn't available)

**Method:** carbon intensity is resampled from 15-minute to hourly resolution and joined against the hourly consumption data; CO₂ = consumption (kWh) × carbon intensity (gCO₂/kWh) for every hour, giving a 2024 baseline emissions figure.

### The five questions answered

1. **Is hourly sampling accurate enough?** Worst-case error is estimated by comparing the hourly-mean carbon intensity against the highest 15-minute reading within that hour. Average worst-case error is small (~2%), with occasional spikes up to ~19% — hourly monitoring is judged sufficient for annual reporting.
2. **What if consumption sampling were coarser?** Tested at 6-hourly, 12-hourly, daily, and monthly intervals. Annual totals stay fairly stable (errors cancel out over the year), but single-hour accuracy degrades quickly — coarser sampling is fine for annual totals, not for anything needing hourly precision.
3. **Solar panel installation (62 MWh/year):** modeled using real PVGIS irradiance data scaled to the planned installation's annual output. Result: a modest ~1–2% reduction in annual CO₂, since the installation is small relative to total campus consumption (~3 GWh/year).
4. **Carbon-aware AI training scheduling (60 kW rack, 40 h/week):** compares a fixed overnight schedule (00:00–08:00, weekdays) against dynamically picking the 40 lowest-carbon-intensity hours each week. Since grid cleanliness varies significantly by hour (more solar/wind at certain times), time-shifting workloads without changing total compute produces a large emissions reduction.
5. **Night-time standby reduction (23:00–05:00):** quantifies CO₂ and cost savings (at €0.30/kWh) for 5%, 10%, and 15% consumption cuts during those hours — savings scale proportionally with the reduction percentage.

**Libraries:** `pandas`, `numpy`, `matplotlib`

## Key Takeaways

- Hourly monitoring is accurate enough for sustainability reporting; higher-frequency metering gives limited additional benefit.
- **When** electricity is used matters as much as **how much** — carbon-aware scheduling of flexible loads (like AI training) is one of the most effective levers available.
- The planned solar installation helps but is small relative to total demand; it's most effective combined with operational measures.
- Cheap behavioral/operational changes (night standby reduction) provide meaningful, proportional CO₂ and cost savings.

## Running the Notebooks

```bash
pip install pandas numpy matplotlib jupyter
```

Run `part1/part1.ipynb` first to (re)generate `consumption.csv`, or use the version already included in `part2/`. Then run `part2/part2.ipynb` — all required input files (`consumption.csv`, `DE_2024_15_minute.csv`, `PVGIS_Data.csv`) are included in the `part2/` folder, so it runs end-to-end with no external downloads.

## Context

This project was completed for a Python/data analysis course, framed as a real-world IT department task: build a Scope 2 electricity sustainability report for Albstadt-Sigmaringen University from no existing data, using justified simulation, real public carbon-intensity data, and real solar irradiance data. The emphasis throughout is on methodology and reasoning rather than perfect real-world accuracy — this is a proof of concept for the process of building and reasoning about believable datasets.
