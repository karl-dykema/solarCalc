# Solar Value Calculator

A single-page web tool for real estate professionals to quickly estimate the market value a residential solar installation adds to a home.

**Live tool:** https://karldykema.github.io/solarCalc

---

## What It Does

Given a few simple inputs — state, system size, age, and ownership type — the calculator produces:

- **Estimated home value added** (NPV of remaining projected electricity savings)
- Annual electricity savings
- 25-year gross savings projection
- Estimated payback period (with federal ITC applied for new systems)
- Remaining useful system life
- All available federal and state incentives for the selected state

---

## The Algorithm

### Inputs

| Field | Description |
|---|---|
| State | Used to look up average electricity rate and peak sun hours |
| System size (kW) | Nameplate capacity of the solar array |
| System age (years) | Used to calculate degradation and remaining life |
| Ownership type | Owned / Leased / PPA — affects value and advisory notes |
| Monthly electric bill | Optional; displayed for context |

---

### Step 1 — Annual Energy Production

```
annual_kWh = system_kW × peak_sun_hours × 365 × performance_ratio × (1 - degradation_rate)^age
```

- **Peak sun hours**: State-level averages from NREL data (~3.0 hrs/day AK to ~6.8 hrs/day NM)
- **Performance ratio**: `0.80` — accounts for inverter losses, wiring, temperature, and soiling
- **Degradation rate**: `0.5%/year` (industry standard for modern panels)

---

### Step 2 — Annual Savings

```
annual_savings = annual_kWh × electricity_rate
```

- **Electricity rate**: State-level averages from EIA 2024 residential data  
  (ranges from ~9.7¢/kWh in ND to ~38.7¢/kWh in HI)

---

### Step 3 — Net Present Value (Home Value Added)

The NPV of the remaining system life is used as the primary estimate for home value added. This is consistent with the income approach used by certified solar appraisers (PV Value® methodology).

For each future year `yr` from 1 to `remaining_years`:

```
production_yr  = base_production × (1 - 0.005)^(yr - 1)
rate_yr        = base_rate × (1 + 0.03)^(age + yr - 1)
savings_yr     = production_yr × rate_yr
NPV           += savings_yr / (1 + 0.07)^yr
```

| Assumption | Value | Rationale |
|---|---|---|
| Rate escalation | 3%/yr | ~10-year historical average of U.S. electricity price increases |
| Discount rate | 7% | Conservative general investment hurdle rate |
| System lifespan | 25 years | Standard manufacturer warranty period |

---

### Step 4 — Payback Period

```
system_cost   = system_kW × 1,000 × $3.00/W   (national avg. installed cost)
effective_cost = system_cost × 0.70            (30% federal ITC, for systems < 1 yr old)
payback_years = effective_cost / annual_savings
```

For existing systems, remaining payback is shown as `payback_years - current_age`.

---

### Step 5 — Incentives

Incentives are displayed from a curated static dataset covering:
- **Federal**: 30% Residential Clean Energy Credit (ITC), valid through 2032
- **State**: Tax credits, SREC markets, net metering policies, property tax exemptions, rebate programs — sourced for all 50 states + D.C.

> Incentive data reflects programs as of early 2025. Programs change frequently; always verify current availability.

---

## Caveats & Limitations

- **Leased / PPA systems**: The value model applies to owned systems only. Leased systems and PPAs involve contract transfer and may be seen as liabilities — a warning is shown.
- **Net metering**: This model assumes 100% of production offsets retail-rate consumption. In states with reduced export rates (e.g., CA NEM 3.0), actual savings may be lower for new installations.
- **Local rates**: The state average electricity rate is used. High-rate territories (e.g., ConEd NYC, HECO Maui) will see higher actual savings.
- **This is not a formal appraisal.** For valuations used in real estate transactions, consult a USPAP-trained appraiser with [PV Value®](https://pvvalue.com) certification.

---

## Data Sources

| Data | Source |
|---|---|
| State electricity rates | EIA Electric Power Monthly (2024 residential averages) |
| Peak sun hours | NREL PVWatts / National Solar Radiation Database |
| Degradation rate | NREL "Photovoltaic Degradation Rates" (2012, updated) |
| Home value methodology | Lawrence Berkeley National Laboratory "Selling Into the Sun" (2015); Zillow solar premium research |
| Federal ITC | IRS Form 5695 / Inflation Reduction Act (2022) |
| State incentives | DSIRE (Database of State Incentives for Renewables & Efficiency) |

---

## Tech Stack

Plain HTML, CSS, and vanilla JavaScript. No dependencies, no build step. Designed to be hosted as a single file on GitHub Pages.
