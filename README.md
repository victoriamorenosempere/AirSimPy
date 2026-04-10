# AirSimPy ✈️

**A Discrete Event Simulation Framework for Air Traffic Management**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![SimPy](https://img.shields.io/badge/SimPy-DES-green.svg)](https://simpy.readthedocs.io/)
[![MSc Distinction](https://img.shields.io/badge/MSc-Distinction-purple.svg)](https://www.westminster.ac.uk/)

> Developed as an MSc Data Science & Analytics project (Distinction) at the University of Westminster, 2024.
> Industry-based external project in collaboration with a private aviation partner.

---

## Academic Innovation

AirSimPy represents a **first of its kind** contribution to open-source Air Traffic Management simulation:

> **AirSimPy is the first open-source Discrete Event Simulation framework built in SimPy that fully integrates taxiing times — both before take-off and after landing — as active, data-driven variables within the simulation calculus, rather than treating them as fixed offsets or ignoring them entirely.**

Prior open-source DES work in ATM (including MERCURY, SIMair, and GPenSIM-based models) focuses primarily on airspace congestion, runway queuing, and ATFM delay propagation. None incorporate taxi times as stochastically modelled, distribution-fitted components that interact directly with the event loop. AirSimPy fills this gap, demonstrating that for short-haul routes at congested airports, **ground operations can dominate total journey time more than airborne flight itself**.

---

## Overview

**AirSimPy** is an open-source discrete event simulation (DES) framework built in Python using [SimPy](https://simpy.readthedocs.io/). It models air traffic management (ATM) operations on a commercial flight route, with particular focus on the contribution of **taxiing times** to overall flight delays.

The framework was developed and validated against real-world flight data from the **London Heathrow (LHR) to Madrid-Barajas (MAD)** route, covering a 14-day operational period (12 June – 7 July 2024), sourced from FlightRadar24.

AirSimPy contains three models:

| Model | Description |
|---|---|
| **Operational Comparison Model** | Queries individual flights by callsign and date; computes estimated vs actual departure/arrival times and taxi durations from historical data |
| **Model 1 — The Independent** | Simulates each flight in isolation with no resource contention — an optimised baseline |
| **Model 2 — The Dependent** | Introduces shared runway and gate resources with SimPy queuing — reflects real-world airport operations |

---

## Why AirSimPy?

Flight delays cost the aviation industry billions annually. Most existing ATM simulation research focuses on airspace congestion and in-flight delays — but **ground operations, particularly taxiing, are a significant and undermodelled source of delay**.

AirSimPy contributes to the field by:

- Introducing the **first SimPy DES framework to model taxi times as fully integrated stochastic components**, using log-normal distributions fitted to real flight data
- Demonstrating that for short-haul routes such as LHR–MAD, **taxi times before take-off can exceed airborne flight time** when runway availability is constrained
- Providing a **user-friendly, interactive interface** accessible to aviation stakeholders without simulation or coding expertise
- Offering a reproducible open-source framework on SimPy, enabling community collaboration and extension

---

## Quick Start with Sample Data

This repository includes **synthetic sample data** so you can run the simulation immediately without needing a FlightRadar24 account.

```bash
git clone https://github.com/victoriamorenosempere/AirSimPy.git
cd AirSimPy
pip install simpy pandas numpy matplotlib scipy seaborn
jupyter notebook AirSimPy_VMS.ipynb
```

The two sample files are included in the repository:

| File | Contents |
|---|---|
| `FlightData ordered by date hour.csv` | Synthetic positional flight records with altitude, speed, and callsign |
| `LHR MAD Flight Schedule.csv` | Synthetic scheduled departure and arrival times |

> ⚠️ **The sample data is synthetic and for demonstration purposes only.** It will show you that the code runs and produces all outputs correctly, but the results will not reflect real-world flight performance. To reproduce the original research findings, collect real data using your own FlightRadar24 subscription (see Using Your Own Data below).

**Try the Operational Comparison Model with the sample data:**

Enter the flight callsign or flight number: BAW462V
Enter the flight date (YYYY-MM-DD): 2024-06-12
Enter the origin airport code (e.g., LHR): LHR
Enter the destination airport code (e.g., MAD): MAD

---

## Using Your Own Data

To run AirSimPy on a real flight route, you need two CSV files in the formats described below.

### File 1 — Flight Tracking Data

**Filename:** `FlightData ordered by date hour.csv`

Collect this from [FlightRadar24](https://www.flightradar24.com/) by downloading the positional history for each flight on your chosen route.

| Column | Type | Description | Example |
|---|---|---|---|
| `Timestamp` | float | Unix timestamp | `1718172000.0` |
| `UTC` | datetime string | Date and time in UTC | `2024-06-12T06:00:00Z` |
| `Callsign` | string | Aircraft callsign (radar identifier) | `BAW462V` |
| `Position` | string | Lat/lon coordinates, or flight status | `51.47,-0.46` |
| `Altitude` | float | Altitude in feet. 0 = on ground (taxiing). Above 0 = airborne | `0.0` or `35000.0` |
| `Speed` | float | Ground speed in knots | `450.0` |
| `Direction` | float | Heading in degrees | `135.0` |

**Important notes on the Position column:**
- For normal flight records this contains coordinates, e.g. `51.470000,-0.460000`
- For cancelled flights, enter `CANCELED` manually
- For days with no scheduled flight, enter `NOFLIGHTSCHEDULED` manually
- These must be added by hand as FlightRadar24 does not export this automatically

**How the simulation uses altitude to calculate taxi times:**

Aircraft starts moving at Altitude = 0  →  taxi_start_before_takeoff
Altitude rises above 0                  →  taxi_end_before_takeoff / takeoff
Altitude returns to 0                   →  landing / taxi_start_after_landing
Aircraft stops (no more timestamps)     →  taxi_end_after_landing

### File 2 — Flight Schedule

**Filename:** `LHR MAD Flight Schedule.csv`

Create this manually from the airline's published timetable for your route and study period.

| Column | Type | Description | Example |
|---|---|---|---|
| `Flight` | string | Flight number (as shown on ticket/board) | `BA462` |
| `Callsign` | string | Aircraft callsign — must match File 1 | `BAW462V` |
| `Estimated Departure` | time string HH:MM:SS | Scheduled departure time | `06:15:00` |
| `Estimated Arrival` | time string HH:MM:SS | Scheduled arrival time | `08:40:00` |

### Recommended Data Collection Workflow

1. Choose a route and a study period of at least 14 days
2. In FlightRadar24, search for your route and filter by date
3. Download the positional CSV for each individual flight
4. Concatenate all files into one, sorted by UTC date and time
5. Manually add `CANCELED` or `NOFLIGHTSCHEDULED` to the Position column where relevant
6. Create the schedule file from the airline's published timetable
7. Place both files in the same directory as the notebook

---

## Simulation Architecture

AirSimPy
│
├── Operational Comparison Model
│   ├── get_estimated_times()         # Reads scheduled times from flight data
│   ├── run()                         # Checks cancellations; logs taxi/flight events
│   └── Time series visualisation     # Departure delay, arrival delay, taxi times
│
├── Model 1 — Independent
│   ├── Flight class                  # log-normal taxi times; normal flight duration
│   ├── run_flight()                  # No resource waiting; sequential processing
│   └── Outputs: Gantt, bar charts, density plots, delay breakdown
│
└── Model 2 — Dependent
├── Flight class                  # Same distributions + user-defined delay modifiers
├── simpy.Resource (runway, gate) # Capacity = 1; flights queue via yield
├── run_flight()                  # Flights wait for shared resources
└── Outputs: Gantt with runway waiting times highlighted

### Delay Parameters

| Condition | Delay Applied |
|---|---|
| Runway unavailable at departure | +15 min to taxi before take-off |
| Runway unavailable at arrival | +15 min to flight duration |
| Gate unavailable at arrival | +15 min to taxi after landing |
| Adverse weather | +20 min to flight duration |
| High season at departure airport | +30 min to taxi before take-off |
| High season at arrival airport | +15 min to taxi after landing |

### Distribution Parameters

| Variable | Distribution | Basis |
|---|---|---|
| Taxi before take-off | Log-normal | Right-skewed; capped at 95th percentile |
| Taxi after landing | Log-normal | More stable; dedicated gate allocation at MAD T4 Satellite |
| Flight duration | Normal | Mean ≈ 111.09 min, SD ≈ 6.22 min (cleaned LHR–MAD data) |

---

## Key Findings

1. **Taxi times before take-off are the dominant ground delay source.** At congested airports (LHR), they can exceed airborne flight duration entirely.
2. **Post-landing taxi times are significantly more stable** — consistent with dedicated gate allocation at Madrid Barajas Terminal 4 Satellite.
3. **Log-normal distributions** are required for taxi times; normal distributions underfit the right skew. Sigma values must be 95th-percentile capped to prevent unrealistic outliers.
4. **Resource contention compounds delays** (Model 2): a 15-minute runway delay for one flight propagates across all subsequent flights sharing the same resource.
5. **Weather primarily affects flight duration**, while runway availability and high-season congestion dominate ground delay profiles.

---

## Example Results (Model 1 — Independent, real LHR–MAD data)

| Flight | Taxi Before Take-off | Flight Duration | Taxi After Landing | Total |
|---|---|---|---|---|
| IBE31BB | 46.77 min | 127.23 min | 20.00 min | 194.00 min |
| BA462 | 25.04 min | 130.19 min | 5.57 min | 160.80 min |
| IB3163 | 64.75 min | 112.10 min | 20.59 min | 197.44 min |

---

## Future Development

- Real-time data integration via FlightRadar24 or OpenSky Network APIs
- Multi-airport network simulation
- AI/ML delay prediction (reinforcement learning, trajectory-based operations)
- Environmental metrics (fuel consumption and emissions per delay scenario)
- Expansion to additional routes and aircraft types

---

## Citation

```bibtex
@misc{morenosempere2024airsimpy,
  author       = {Moreno Sempere, Victoria},
  title        = {AirSimPy: A Discrete Event Simulation Framework for Air Traffic Management},
  year         = {2024},
  publisher    = {GitHub},
  howpublished = {\url{https://github.com/victoriamorenosempere/AirSimPy}},
  note         = {MSc Data Science \& Analytics (Distinction), University of Westminster.}
}
```

> **DOI:** *(to be assigned via Zenodo)*

---

## Licence

MIT Licence — see [LICENSE](LICENSE). You are free to use, modify, and distribute AirSimPy with attribution.

---

## Author

**Victoria Moreno Sempere**
Data Scientist | MSc Data Science & Analytics (Distinction), University of Westminster, 2024
BSc Applied Biomedical Science, University of Westminster, 2022

[GitHub](https://github.com/victoriamorenosempere) · [LinkedIn](https://www.linkedin.com/in/victoriamorenosempere)

*Supervised by Dr Malarvizhi Kaniappan Chinnathai, University of Westminster*

---

## Acknowledgements

With thanks to Dr Malarvizhi Kaniappan Chinnathai for supervision and mentorship throughout this research, and to Dr Luis Delgado and Dr Gérald Gurtner for the invitation to the Open-Source Tools for ATM Workshop, which deepened the foundations of this work.
