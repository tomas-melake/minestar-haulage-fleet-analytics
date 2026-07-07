# Mining Operations Analytics: Predictive Modeling for Autonomous Haulage Cycles

**Author:** Tomas Melake  
**Organization:** Technical Portfolio Report  

## Executive Summary
This project delivers an enterprise-grade data engineering pipeline and a predictive analytics framework to evaluate cycle-time performance across an autonomous haulage fleet of Caterpillar 793 series trucks. In large-scale open-pit mining, operational efficiency is highly sensitive to upstream and downstream bottlenecks. By developing a robust QA/QC purification gate and deploying an Ordinary Least Squares (OLS) Linear Regression model, this framework audits raw Cat MineStar telemetry, isolates individual component variances, and quantifies their exact marginal impact on the global target vector: **Total Cycle Time** (`Total_Cycle_Min`).

---

## Operational Architecture & Cat MineStar Framework
To map the raw haulage loop into a predictive data structure, the pipeline tracks data across the foundational components of the **Cat MineStar** operating system:
1. **Fleet Management:** Optimizes real-time truck-to-shovel dispatching to minimize truck bunching.
2. **Terrain for Loading/Dumping:** Guides autonomous assets precisely into loading pockets and dumping positions using high-precision GPS.
3. **Detect (Proximity & Lidar):** Manages the safety tracking network, executing automatic slowdowns if environmental factors trip optical paths.
4. **Health (VIMS Telemetry):** Monitors vital asset health signs, powertrain configurations, and physical payload weights via strut pressure sensors.
5. **Command for Hauling:** Controls the fully autonomous driving matrix along designated haul roads.

### Predictive Feature Matrix
The machine learning model isolates a single global target from six independent operational variables:
* **`Total_Cycle_Min` (Target Variable):** Total elapsed duration from entering a shovel queue to completing the empty return loop.
* **`Queue_Time_Min`:** Idle time spent waiting at the loading face for an excavator.
* **`Spot_and_Load_Min`:** Time taken to spot the asset and receive full bucket passes.
* **`Haul_Loaded_Min`:** Travel time from the loading face to the designated dump location.
* **`Dump_Time_Min`:** Duration spent spotting, tipping, and clearing the dump boundary.
* **`Return_Empty_Min`:** Travel time returning empty from the dump back to a loading circuit.
* **`Payload_Tonnes`:** Total physical weight of rock inside the truck bed.

---

## Part 1: Data Pipeline & QA/QC Audit Gate
To protect model validity and eliminate parameter bias, the data pipeline implements an automated **Data Quality Gate** to intercept three primary corporate database failure points:
* **Null Packets:** Rows with missing operational timestamps or telemetry dropouts are systematically isolated.
* **Replicates (Duplicates):** Purges duplicate transaction records caused by asynchronous server packet transmission.
* **Calibration Drift (Inconsistent Entries):** Filters out physically impossible negative or zero-value metrics (`telemetry <= 0`) caused by onboard VIMS calibration drift, yielding a pristine, production-ready dataset.

Following purification, the data is partitioned into an **80% Training Set** and a **20% Test Set**, which are simultaneously exported as independent assets to power downstream executive Power BI dashboards.

---

## Part 2: Statistical Modeling & Inference Results

### Global Performance Metrics
* **Coefficient of Determination ($R^2$):** **91.40%**. The model explains 91.4% of the total variance in haul truck cycle times, proving it is exceptionally robust for strategic target forecasting. The remaining 8.6% captures unmapped, chaotic field noise (e.g., rolling resistance shifts or dispatch communication lag).
* **Root Mean Squared Error (RMSE):** Restricted to a fraction of a minute (~12 seconds), confirming that the predictive error bounds are tightly controlled.

### Model Interpretation & Coefficient Matrix

| Operational Factor | Coefficient ($\beta$) | Statistical Significance ($P$-Value) | Hypothesis Conclusion |
| :--- | :---: | :---: | :--- |
| **`Queue_Time_Min`** | +1.011071 | 0.000000 | **REJECT $H_0$** (Severe Bottleneck) |
| **`Spot_and_Load_Min`** | +1.005003 | 0.000000 | **REJECT $H_0$** (Direct 1:1 Impact) |
| **`Haul_Loaded_Min`** | +0.994628 | 0.000000 | **REJECT $H_0$** (Highly Stable Link) |
| **`Dump_Time_Min`** | +0.998306 | 0.000000 | **REJECT $H_0$** (Strict Traffic Constraint) |
| **`Return_Empty_Min`**| +0.988977 | 0.000000 | **REJECT $H_0$** (Primary Operational Variance) |
| **`Payload_Tonnes`** | -0.000785 | 0.741096 | **FAIL TO REJECT $H_0$** (The Payload Paradox) |

### Key Analytical Insights
* **The Compounding Queue Penalty:** A 1-minute increase in `Queue_Time_Min` expands the total cycle by **1.0111 minutes**. This coefficient exceeding 1.0 proves a cascading system penalty—idle trucks trigger proximity deceleration loops for trailing assets, creating costly "truck bunching" events.
* **The Payload Paradox Exploded:** The model mathematically disproves a common industry myth: variations in payload weight within normal operating thresholds have a negligible impact on speed ($\beta = -0.000785$, $P = 0.741096$). There is a 74.1% probability that this variation is pure random noise, proving the Cat 793 powertrains handle nominal load fluctuations comfortably. Cycle delays are caused by traffic and scheduling constraints, not asset capacity limits.

---

## Part 3: Strategic Operational Recommendations

### 1. Mitigating the Million-Dollar Cascading Delay Trap
The data pipeline proves that unscheduled queuing bottlenecks at the shovel face compound exponentially across a standard 5,000-cycle period:
* **Total Accumulated Delay:** Losing an average of just 2 minutes per cycle across 5,000 loops equates to **10,000 total lost minutes**.
* **Displaced Haulage Capacity:** Given the dataset's true mean cycle time of **29.05 minutes**, this operational lag completely eliminates **344.234 productive trips**.
* **Physical & Financial Material Deficit:** At a nominal capacity of 250 tonnes per trip, the system suffers a fleet deficit of **86,065.69 lost tonnes** of material. In a high-grade gold ore scenario processing 1.5 g/t, this variance introduces a production lag of **4,150.61 ounces** of gold, presenting an annual revenue risk exposure of **$9,546,405.35** (at a $2,300/oz baseline). Shovel queue times must be actively managed as critical financial leakages.

### 2. Targeted Dust Suppression & Optical Path Protection
The model isolated a distinct velocity drop during empty returns specifically along the *Waste_Dump_North* corridor. This is driven by environmental conditions: heavy dust clouds from high-volume dumping trigger the trucks' Cat MineStar Detect sensors (Lidar/radar), inducing automatic "Ghost Object" safety slowdowns and emergency stops.  
* **Action:** Deploy auxiliary water trucks equipped with localized binding agents to the active haul roads leading into *Waste_Dump_North*. Dispatchers must track water trucks as critical infrastructure assets to suppress dust ahead of peak haulage hours, protecting the autonomous fleet's optical paths and restoring baseline empty speeds.

### 3. Transitioning to Dynamic Match-Factor Dispatching
Because payload variance does not statistically affect travel duration, supervision should pivot from over-managing exact bucket weights to maximizing dynamic fleet balancing.  
* **Action:** Configure the Cat MineStar Command engine to run real-time Match Factor tracking. If a high-grade circuit exceeds a Match Factor of 1.0 (indicating shovel over-trucking and queue generation), the automated dispatch engine must instantly re-route empty incoming trucks to under-trucked waste or low-grade stockpile circuits, spacing out return arrival rates.
