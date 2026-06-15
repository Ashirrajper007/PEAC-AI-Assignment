# AI Assignment 1: Intelligent Agent Specification

## Course Information
- **Course:** Introduction to Artificial Intelligence (CS301)
- **Topic:** PEAS Framework and Task Environment Analysis
- **Reference:** Russell & Norvig, Artificial Intelligence: A Modern Approach (Chapter 2)

## Project Overview
This repository contains the response to Assignment 1, which requires applying the formal agent frameworks discussed in Chapter 2 of Russell & Norvig to a real-world AI application. The objective is to specify the agent using the PEAS framework, classify its task environment along six key dimensions with justifications, and conduct a critical analysis of the design trade-offs involved.

## Chosen System: Option C — Agricultural Monitoring Drone for Crop Disease Detection

An autonomous aerial agent designed for precision agriculture. The drone patrols farmland, captures multispectral imagery of crops, and uses onboard or cloud-based AI models to detect early signs of disease, pest infestation, or nutrient deficiency, enabling farmers to intervene before widespread crop loss occurs.

---

## Task 1: PEAS Specification

### Performance Measure
The success of the agent is evaluated using the following metrics:

1. **Detection Accuracy (F1-Score):** Balances precision and recall in identifying diseased crop regions, minimizing both false alarms and missed infections.
2. **Coverage Efficiency (Hectares scanned per battery cycle):** Measures how much farmland the drone can effectively survey before needing to recharge or land.
3. **Response Latency (Time from capture to alert):** The delay between detecting an anomaly and notifying the farmer, critical for timely intervention.
4. **Flight Safety Score (Incidents per 100 flight hours):** Tracks collisions, crashes, or near-misses with obstacles, livestock, or other aircraft.
5. **Image Resolution Quality (Ground Sampling Distance, GSD):** Determines whether captured imagery is sharp enough for reliable disease classification at the leaf level.

### Environment
- **Setting:** Open agricultural fields, orchards, or greenhouses, typically spanning several hectares.
- **External Factors:** Variable weather (wind gusts, rain, fog), changing sunlight intensity and shadows throughout the day, presence of physical obstacles (trees, power lines, irrigation equipment, farm workers, animals), and irregular terrain/crop height.
- **Regulatory Context:** Airspace restrictions (e.g., no-fly zones near airports) and local drone-operation regulations.

### Actuators
- **Brushless DC Motors / Rotors:** Control lift, thrust, and directional movement.
- **3-Axis Gimbal Stabilizer:** Adjusts camera orientation independently of drone movement for stable imaging.
- **Electronic Speed Controllers (ESCs):** Regulate motor speed for precise flight maneuvers.
- **Wireless Data Transmitter (4G/5G or RF telemetry):** Sends captured images, GPS coordinates, and alerts to a ground control station or cloud server.
- **Onboard LED/Visual Indicators:** Signal status (battery low, malfunction, mission complete) to nearby human operators.

### Sensors
- **Multispectral Camera (NIR, Red-Edge, RGB bands):** Captures vegetation health indices (e.g., NDVI) to detect chlorophyll changes indicative of disease.
- **RTK-GPS (Real-Time Kinematic GPS):** Provides centimeter-level positioning for precise mapping and consistent re-surveying of the same plots.
- **Ultrasonic/LiDAR Altimeters:** Measure altitude above crop canopy to maintain consistent imaging height.
- **Inertial Measurement Unit (IMU):** Tracks orientation, acceleration, and tilt for stable flight control.
- **Optical Flow Sensors:** Detect ground movement relative to the drone for precise low-altitude positioning, especially when GPS signal is weak.

---

## Task 2: Environment Classification

| Dimension | Classification | Justification |
|---|---|---|
| **Observability** | Partially Observable | The drone's sensors cannot perceive subsurface soil conditions, root health, or disease symptoms occurring outside the camera's current field of view, so the agent never has complete knowledge of the field's true state. |
| **Number of Agents** | Single-agent | In a typical small-to-medium farm deployment, one drone operates independently without needing to coordinate with or compete against other autonomous agents. |
| **Determinism** | Stochastic | Wind gusts, sudden lighting changes, and unpredictable obstacles (birds, workers) mean the same action under the same conditions can lead to different outcomes. |
| **Episodic vs. Sequential** | Sequential | The drone's current position, remaining battery, and previously scanned areas directly influence which path and actions it should take next, so decisions are interdependent over time. |
| **Static vs. Dynamic** | Dynamic | The environment changes during deliberation — weather shifts, animals move, and lighting conditions evolve while the drone is still planning or executing its survey route. |
| **Discrete vs. Continuous** | Continuous | Flight controls (motor thrust, tilt angles, velocity) and sensor readings (GPS coordinates, NDVI values) all exist in continuous real-valued ranges rather than discrete steps. |

---

## Task 3: Critical Analysis

### 1. Greatest Technical Challenge
Among the six dimensions, the **Partially Observable** nature of the environment poses the greatest technical challenge. Because the drone cannot directly "see" beneath the leaf surface or inside plant tissue, it must infer disease presence from indirect signals like reflected light spectra (NDVI), which can be confounded by other stressors such as drought or nutrient deficiency that produce visually similar symptoms. This forces engineers to build the agent's internal state estimation using probabilistic models (e.g., Bayesian belief updating or learned classifiers trained on labeled spectral data) rather than relying on direct perception, increasing both the complexity of the system and the risk of misdiagnosis. Combined with the **Stochastic** nature of weather and lighting, the agent must also continuously re-calibrate its confidence in these inferences, making robust uncertainty management the core engineering hurdle.

### 2. Utility Function and Weight Trade-off
A useful utility function for this agent balances **Coverage Speed** against **Detection Precision**:

```
U = w1 · (Coverage Rate) + w2 · (Detection Confidence) − w3 · (Battery Consumption)
```

Here, *Coverage Rate* rewards the agent for surveying more land per unit time, *Detection Confidence* rewards higher-resolution, more reliable disease readings, and *Battery Consumption* penalizes energy-intensive maneuvers.

If the weight **w2 (Detection Confidence)** were doubled, the agent's behavior would shift significantly: it would reduce its flight speed and lower its altitude over each plot to capture sharper, higher-resolution images, and may revisit ambiguous areas multiple times to confirm a diagnosis before moving on. The trade-off is that **Coverage Rate would drop** — the drone would survey fewer hectares per battery cycle, meaning a larger farm might require multiple flights or additional drones to complete a full survey in the same timeframe. Essentially, the agent would prioritize **accuracy over throughput**, which is desirable for high-value crops where a missed disease outbreak is far costlier than slower survey completion.
