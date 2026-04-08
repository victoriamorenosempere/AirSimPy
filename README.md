# AirSimPy ✈️

**A Discrete Event Simulation Framework for Air Traffic Management**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![SimPy](https://img.shields.io/badge/SimPy-DES-green.svg)](https://simpy.readthedocs.io/)
[![MSc Distinction](https://img.shields.io/badge/MSc-Distinction-purple.svg)](https://www.westminster.ac.uk/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.19474935.svg)](https://doi.org/10.5281/zenodo.19474935)

> Developed as an MSc Data Science & Analytics project (Distinction) at the University of Westminster, 2024.
> Industry partner: Caasco Ltd.

---

## Overview

**AirSimPy** is an open-source discrete event simulation (DES) framework built in Python using [SimPy](https://simpy.readthedocs.io/). It models air traffic management (ATM) operations on a commercial flight route, with a particular focus on the contribution of **taxiing times** — both before take-off and after landing — to overall flight delays.

The framework was developed and validated against real-world flight data from the **London Heathrow (LHR) to Madrid-Barajas (MAD)** route, covering a 14-day operational period (12 June – 7 July 2024), sourced from FlightRadar24.

AirSimPy produces two complementary simulation models:

- **Model 1 — The Independent Model**: Simulates each flight in isolation, without resource contention. Provides an optimised baseline for expected durations under ideal conditions.
- **Model 2 — The Dependent Model**: Introduces shared airport resources (runways, gates) with queuing behaviour, reflecting real-world operational constraints where flights compete for limited infrastructure.

A third operational model was developed specifically to meet requirements from industry partner **Caasco Ltd**, computing estimated vs actual departure and arrival times from historical data.

---

## Why AirSimPy?

Flight delays cost the aviation industry billions annually. Most existing ATM simulation research focuses on airspace congestion and in-flight delays — but ground operations, particularly **taxiing**, are a significant and undermodelled source of delay.

AirSimPy contributes to the field by:

- Introducing a granular, data-driven examination of taxi times using **log-normal distributions**, which better reflect their skewed, volatile behaviour compared to normal distributions
- Demonstrating that for short-haul routes such as LHR–MAD, **taxi times before take-off can exceed airborne flight time** when runway availability is constrained
- Providing a user-friendly, interactive interface accessible to aviation stakeholders without simulation or coding expertise
- Offering a reproducible open-source framework built on the widely-used Python SimPy library, enabling community collaboration and future extension

---

## Features

- **Three simulation models** in one notebook: Operational Analysis (Caasco), Independent, and Dependent
- **User-interactive inputs**: flight callsign, date, airport codes, runway/gate availability, weather conditions, and high-season traffic flags
- **Statistical data analysis**: distribution fitting (log-normal for taxi times, normal for flight duration), Shapiro-Wilk and KS tests, Q-Q plots, outlier removal
- **Rich visual outputs**: Gantt charts, time series, histograms, density plots, bar charts of delay types
- **Realistic data grounding**: validated against 14 days of FlightRadar24 data on a live commercial route
- **Extensible architecture**: SimPy's resource and event model supports future expansion to multi-airport networks, real-time API feeds, and AI integration

---

## Simulation Architecture
AirSimPy
│
├── Operational Model (Caasco)       # User queries individual flight; returns estimated vs actual
│   ├── get_estimated_times()
│   ├── run()                        # Checks cancellation status, logs taxi events
│   └── Time series analysis         # Departure delay, arrival delay, taxi times over 14 days
│
├── Model 1 — Independent
│   ├── Flight class                 # log-normal taxi times, normal flight duration
│   ├── run_flight()                 # No resource waiting; flights processed independently
│   └── Outputs: Gantt, bar charts, density plots, delay breakdown
│
└── Model 2 — Dependent
├── Flight class                 # Same distributions + delay modifiers
├── simpy.Resource (runway, gate) # Capacity = 1; flights queue via yield
├── run_flight()                 # Flights wait for shared resources
└── Outputs: Gantt with waiting times highlighted

### Delay Parameters (Model 1 & 2)

| Condition | Delay Added |
|---|---|
| Runway unavailable at departure | +15 min to taxi before take-off |
| Runway unavailable at arrival | +15 min to flight duration |
| Gate unavailable at arrival | +15 min to taxi after landing |
| Adverse weather | +20 min to flight duration |
| High season at departure airport | +30 min to taxi before take-off |
| High season at arrival airport | +15 min to taxi after landing |

### Distribution Parameters

| Parameter | Distribution | Notes |
|---|---|---|
| Taxi before take-off | Log-normal | Right-skewed; capped at 95th percentile |
| Taxi after landing | Log-normal | More stable; Madrid T4 Satellite dedicated gates |
| Flight duration | Normal | Mean ≈ 111.09 min, SD ≈ 6.22 min (LHR–MAD cleaned data) |

---

## Data

The dataset covers the **LHR–MAD route, 12 June – 7 July 2024**, manually collected from FlightRadar24.

| File | Contents |
|---|---|
| `FlightData ordered by date hour.csv` | Timestamped positional data for all flights |
| `LHR MAD Flight Schedule.csv` | Scheduled estimated departure and arrival times |

---

## Installation

### Requirements
```bash
pip install simpy pandas numpy matplotlib scipy seaborn
```

### Clone and Run
```bash
git clone https://github.com/victoriamorenosempere/AirSimPy.git
cd AirSimPy
jupyter notebook AirSimPy_VMS.ipynb
```

Or open directly in [Google Colab](https://colab.research.google.com/).

---

## Usage

### Operational Model (Caasco)

Enter the flight callsign or flight number: IBE31BB
Enter the flight date (YYYY-MM-DD): 2024-06-12
Enter the origin airport code (e.g., LHR): LHR
Enter the destination airport code (e.g., MAD): MAD

**Example output:**
Flight IBE31BB report:
Estimated Departure Time: 06:15:00
Actual Departure Time: 2024-06-12 07:04:45+00:00
Taxi Time Before Takeoff: 0 days 01:52:47
Estimated Departure vs Actual Departure: 109.75 minutes

---

## Key Findings

1. **Taxi times before take-off are the dominant source of delay** for short-haul routes at congested airports such as LHR
2. **Post-landing taxi times are significantly more stable** than pre-departure taxi times
3. **Log-normal distributions** model taxi time variability more accurately than normal distributions
4. **Shared resource contention** creates compounding delays across subsequent flights
5. **Weather impacts flight duration** while **high season and runway availability** dominate ground delay profiles

---

## Citation

If you use AirSimPy in your research, please cite:
```bibtex
@software{morenosempere2026airsimpy,
  author       = {Moreno Sempere, Victoria},
  title        = {AirSimPy: A Discrete Event Simulation Framework
                  for Air Traffic Management},
  year         = {2026},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.19474935},
  url          = {https://doi.org/10.5281/zenodo.19474935}
}
```

**DOI:** [10.5281/zenodo.19474935](https://doi.org/10.5281/zenodo.19474935)

---

## Future Development

- Integration of real-time flight data via FlightRadar24 or OpenSky Network APIs
- Extension to multi-airport network simulation
- AI/ML models for delay prediction and trajectory-based operations
- Environmental metrics — fuel consumption and emissions per delay scenario
- Expansion to other routes and aircraft types

---

## Publishing on Zenodo

This project is permanently archived and citable via Zenodo:

**DOI:** https://doi.org/10.5281/zenodo.19474935

---

## Licence

MIT Licence — see the [LICENSE](LICENSE) file for details.

---

## Author

**Victoria Moreno Sempere**
Data Scientist | MSc Data Science & Analytics (Distinction), University of Westminster, 2024

[GitHub](https://github.com/victoriamorenosempere) · [LinkedIn](https://www.linkedin.com/in/victoriamorenosempere)

*Supervised by Dr Malarvizhi Kaniappan Chinnathai, University of Westminster*
*Industry partner: Caasco Ltd*

---

## Acknowledgements

With thanks to Dr Malarvizhi Kaniappan Chinnathai for supervision and mentorship; to Caasco Ltd and Noah Ahiable for the industry partnership; and to Dr Luis Delgado and Dr Gérald Gurtner for the invitation to the Open-Source Tools for ATM Workshop.
