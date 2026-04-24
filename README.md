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

## Repository Structure

AirSimPy/
├── AirSimPy_VMS_GitHUB.ipynb.                                            # Main simulation notebook
├── FlightData_SAMPLE_synthetic_testing_only.csv                          # Fictional sample data — Mordor to Bikini Bottom
├── MORDOR_BIKINIBOTTOM_FlightSchedule_SAMPLE_synthetic_testing_only.csv   # Fictional schedule — Mordor to Bikini Bottom
├── README.md                                                              # This file
└── LICENSE                                                                # MIT Licence

---

## Sample Data — Mordor to Bikini Bottom 🧙‍♀️🍍

To allow anyone to run AirSimPy immediately — without needing a FlightRadar24 account — this repository includes a **fully fictional sample dataset** based on a completely invented flight route:

> **Mordor International Airport (MOR) → Bikini Bottom Airport (BIK)**

This fictional dataset was generated programmatically from the same column structure and altitude-based logic as real flight tracking data, with all identifiers replaced:

| Real | Fictional |
|---|---|
| London Heathrow (LHR) | Mordor International (MOR) |
| Madrid Barajas (MAD) | Bikini Bottom Airport (BIK) |
| Real callsigns (e.g. BAW462V) | Fictional callsigns (e.g. BIK462V, MOR31XX) |
| Real flight numbers (e.g. BA462) | Fictional flight numbers (e.g. BK0462, MR3181) |
| Real coordinates | Fictional coordinates in the mid-Pacific |
| Original times | All times shifted by +1 hour |

> ⚠️ **The sample data is entirely fictional and for demonstration purposes only.** It will show you that the code runs and produces outputs correctly, but the results do not reflect any real-world flight performance. To reproduce the original research findings, collect real data using your own FlightRadar24 subscription — see [Using Your Own Data](#using-your-own-data) below.

**Try the Operational Comparison Model with the sample data:**

Enter the flight callsign or flight number: MOR31XX
Enter the flight date (YYYY-MM-DD): 2024-06-12
Enter the origin airport code: MOR
Enter the destination airport code: BIK

**Try Models 1 & 2 with the sample data — use any of these callsigns:**

| Callsign | Flight Number |
|---|---|
| MOR31XX | MR3181 |
| BIK462V | BK0462 |
| MOR31MA | MR3163 |
| BIK04MJ | BK0458 |
| MOR31PP | MR3175 |
| MOR3177 | MR3177 |

---

## Using Your Own Data

To run AirSimPy on a real flight route, collect two CSV files and place them in the same directory as the notebook, using these exact filenames:

- `FlightData ordered by date hour.csv`
- `LHR MAD Flight Schedule.csv`

### File 1 — Flight Tracking Data

**Filename:** `FlightData ordered by date hour.csv`

Collect from [FlightRadar24](https://www.flightradar24.com/) by downloading the positional history for each flight on your chosen route.

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
- Normal flight records contain coordinates e.g. `51.470000,-0.460000`
- For cancelled flights enter `CANCELED` manually
- For days with no scheduled flight enter `NOFLIGHTSCHEDULED` manually
- These must be added by hand as FlightRadar24 does not export this automatically

**How the simulation derives taxi times from altitude:**

Aircraft starts moving at Altitude = 0  →  taxi_start_before_takeoff
Altitude rises above 0                  →  taxi_end_before_takeoff / takeoff
Altitude returns to 0                   →  landing / taxi_start_after_landing
Aircraft stops (no more timestamps)     →  taxi_end_after_landing

### File 2 — Flight Schedule

**Filename:** `LHR MAD Flight Schedule.csv`

Create manually from the airline's published timetable for your route and study period.

| Column | Type | Description | Example |
|---|---|---|---|
| `Flight` | string | Flight number as shown on ticket/board | `BA462` |
| `Callsign` | string | Aircraft callsign — must match File 1 exactly | `BAW462V` |
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

## Installation

```bash
pip install simpy pandas numpy matplotlib scipy seaborn scikit-learn pytz
```

```bash
git clone https://github.com/victoriamorenosempere/AirSimPy.git
cd AirSimPy
jupyter notebook AirSimPy_VMS.ipynb
```

Or open directly in [Google Colab](https://colab.research.google.com/).

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
| Taxi after landing | Log-normal | More stable; dedicated gate allocation observed in real data |
| Flight duration | Normal | Mean ≈ 111.09 min, SD ≈ 6.22 min (original research data) |

---

## Key Findings

1. **Taxi times before take-off are the dominant ground delay source.** At congested airports they can exceed airborne flight duration entirely.
2. **Post-landing taxi times are significantly more stable** — consistent with dedicated gate allocation at the arrival airport.
3. **Log-normal distributions** are required for taxi times; sigma values must be 95th-percentile capped to prevent unrealistic outliers.
4. **Resource contention compounds delays** (Model 2): a 15-minute runway delay for one flight propagates across all subsequent flights sharing the same resource.
5. **Weather primarily affects flight duration**, while runway availability and high-season congestion dominate ground delay profiles.

---

## Example Results (original research data)

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
