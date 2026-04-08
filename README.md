# AirSimPy ✈️

**A Discrete Event Simulation Framework for Air Traffic Management**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![SimPy](https://img.shields.io/badge/SimPy-DES-green.svg)](https://simpy.readthedocs.io/)
[![MSc Distinction](https://img.shields.io/badge/MSc-Distinction-purple.svg)](https://www.westminster.ac.uk/)

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

```
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
```

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
| Taxi before take-off | Log-normal | Right-skewed; capped at 95th percentile to prevent unrealistic values |
| Taxi after landing | Log-normal | More stable; Madrid T4 Satellite provides dedicated gates |
| Flight duration | Normal | Mean ≈ 111.09 min, SD ≈ 6.22 min (LHR–MAD cleaned data) |

---

## Data

The dataset covers the **LHR–MAD route, 12 June – 7 July 2024**, manually collected from FlightRadar24. Two files are used:

| File | Contents |
|---|---|
| `FlightData ordered by date hour.csv` | Timestamped positional data (altitude, speed, callsign) for all flights |
| `LHR MAD Flight Schedule.csv` | Scheduled estimated departure and arrival times |

**Data notes:**
- Cancelled and non-scheduled flights were manually annotated in the `Position` column
- Taxi times are derived from altitude changes: ground movement (altitude = 0) bookends the taxi phases
- Outliers were removed using 95th percentile capping before distribution fitting

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

Run the notebook and enter when prompted:

```
Enter the flight callsign or flight number: IBE31BB
Enter the flight date (YYYY-MM-DD): 2024-06-12
Enter the origin airport code (e.g., LHR): LHR
Enter the destination airport code (e.g., MAD): MAD
```

**Example output:**
```
Flight IBE31BB report:
  Estimated Departure Time: 06:15:00
  Actual Departure Time: 2024-06-12 07:04:45+00:00
  Estimated Arrival Time: 08:40:00
  Actual Arrival Time: 2024-06-12 08:52:40+00:00
  Taxi Time Before Takeoff: 0 days 01:52:47
  Taxi Time After Landing: 0 days 00:04:19
  Estimated Departure vs Actual Departure: 109.75 minutes
  Estimated Arrival vs Actual Arrival: 132.67 minutes
```

### AirSimPy Models 1 & 2

The simulation prompts for 3 flights with operational parameters:

```
Enter the flight Callsign or Flight Number: BA462
Is there a runway available at the departure airport? (yes/no): YES
Is there a runway available at the arrival airport? (yes/no): YES
Is there a gate available at the arrival airport? (yes/no): YES
Enter the type of weather during the flight (e.g., adverse/clear): ADVERSE
Is it high season at the departure airport? (yes/no): NO
Is it high season at the arrival airport? (yes/no): NO
```

Results are printed to console, saved to `simulation_results.csv`, and rendered as visualisations in-notebook.

---

## Example Outputs

### Model 1 — Sample Results

| Flight | Taxi Before Take-off | Flight Duration | Taxi After Landing | Total |
|---|---|---|---|---|
| IBE31BB | 46.77 min | 127.23 min | 20.00 min | 194.00 min |
| BA462 | 25.04 min | 130.19 min | 5.57 min | 160.80 min |
| IB3163 | 64.75 min | 112.10 min | 20.59 min | 197.44 min |

### Model 2 — Event Timeline (Dependent)

Flights queue for the single shared runway. IBE31BB departs first, BA462 begins taxiing only once IBE31BB clears the runway, and IB3163 follows sequentially — reflecting real congestion dynamics at LHR.

### Visualisations Generated

- **Gantt charts** of all flight phases (taxi before, flight, taxi after; with waiting time highlighted in Model 2)
- **Bar charts** of taxi times before take-off vs after landing per flight
- **Density plots** of total time utilisation by flight
- **Delay breakdown charts** showing contribution of each delay type (runway, gate, weather, season)
- **Time series** of departure delay, arrival delay, and taxi times over the 14-day period

---

## Key Findings

1. **Taxi times before take-off are the dominant source of delay** for short-haul routes at congested airports such as LHR. In several simulated scenarios, taxi time exceeded airborne flight time.
2. **Post-landing taxi times are significantly more stable** than pre-departure taxi times, consistent with the dedicated gate allocation at Madrid Barajas Terminal 4 Satellite for Iberia and British Airways.
3. **Log-normal distributions** model taxi time variability more accurately than normal distributions; sigma values must be capped at realistic thresholds to prevent the model producing unrealistic outliers.
4. **Shared resource contention** (Model 2) creates compounding delays: a 15-minute delay for one flight propagates across subsequent flights competing for the same runway.
5. **Weather impacts flight duration** (not taxi times), while **high season and runway availability** dominate ground delay profiles.

---

## Research Context

AirSimPy was submitted as an MSc industry-based project in partnership with **Caasco Ltd**, fulfilling requirements to compare estimated and actual departure/arrival times on a live commercial route.

The work builds on the existing ATM DES literature (including MERCURY, SIMair, GPenSIM) while making specific contributions:
- Focus on the underexplored domain of taxiing as a delay driver
- Open-source SimPy implementation accessible without commercial simulation licences
- Real-world validated data from a high-frequency international route
- User-friendly interactive interface for aviation industry stakeholders

For full methodology, literature review (36 publications), and results, see the accompanying research paper.

---

## Future Development

- Integration of real-time flight data via FlightRadar24 or OpenSky Network APIs
- Extension to multi-airport network simulation
- Incorporation of AI/ML models for delay prediction (reinforcement learning, trajectory-based operations)
- Addition of environmental metrics (fuel consumption, emissions per delay scenario)
- Expansion to other routes and aircraft types

---

## Citation

If you use AirSimPy in your research, please cite:

```bibtex
@misc{morenosempere2024airsimpy,
  author       = {Moreno Sempere, Victoria},
  title        = {AirSimPy: A Discrete Event Simulation Framework for Air Traffic Management},
  year         = {2024},
  publisher    = {GitHub},
  journal      = {GitHub repository},
  howpublished = {\url{https://github.com/victoriamorenosempere/AirSimPy}},
  note         = {MSc Data Science \& Analytics, University of Westminster. Distinction.}
}
```

> **DOI:** *(to be assigned via Zenodo — see [Publishing on Zenodo](#publishing-on-zenodo) below)*

---

## Publishing on Zenodo

To obtain a citable DOI for academic use:

1. Go to [zenodo.org](https://zenodo.org) and log in with your GitHub account
2. Under **GitHub**, enable the AirSimPy repository
3. Create a GitHub Release — Zenodo will automatically archive it and assign a DOI
4. Add the DOI badge and citation to this README

---

## Licence

This project is licensed under the **MIT Licence** — see the [LICENSE](LICENSE) file for details.

You are free to use, modify, and distribute AirSimPy with attribution.

---

## Author

**Victoria Moreno Sempere**  
Data Scientist | MSc Data Science & Analytics (Distinction), University of Westminster, 2024  
BSc Applied Biomedical Science, University of Westminster, 2022  

[GitHub](https://github.com/victoriamorenosempere) · [LinkedIn]([https://www.linkedin.com/in/victoria-moreno-sempere-01a438153/]

*Supervised by Dr Malarvizhi Kaniappan Chinnathai, University of Westminster*  
*Industry partner: Caasco Ltd*

---

## Acknowledgements

With thanks to Dr Malarvizhi Kaniappan Chinnathai for supervision and mentorship; to Caasco Ltd and Noah Ahiable for the industry partnership; and to Dr Luis Delgado and Dr Gérald Gurtner for the invitation to the Open-Source Tools for ATM Workshop 2024, which deepened the foundations of this work.
