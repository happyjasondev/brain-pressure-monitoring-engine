# 📈 Brain Pressure Monitoring Engine

> A Python-driven and AI collaborated biomedical signal processing and anomaly detection engine, paired with a web dashboard to monitor time-series Intracranial Pressure (ICP) data, predict hydrocephalus shunt blockage, and trigger early warning alerts. All data is synthetic and generated with Python.
> Currently, Intracranial monitoring (ICP) equipment are only available in medical institutions. If lightweight, portable monitoring devices become available for everyday wear in the future, it would greatly benefit hydrocephalus patients.
> I made this program to learn whether shunt blockages could be predicted by tracking brain pressure fluctuations. If this monitor is paired with a portable ICP device, patients could keep an eye on their brain pressure at any time, rather than only realizing a blockage has occurred after sleeping for abnormally long periods of times or even falling into a coma.

<a href="https://happyjasondev.github.io/brain-pressure-monitoring-engine/">
<img src="Animation.gif" width="60%" alt="Brain Pressure Monitoring Engine Demo" />
</a>

## 🔬 View the Website
**Launch the live interactive monitor and click "Click Here to Play Data Stream" to view the time-series data playback:** [https://happyjasondev.github.io/brain-pressure-monitoring-engine/](https://happyjasondev.github.io/brain-pressure-monitoring-engine/)

## ⚙️ How it works:

* **Synthetic Simulation:** Creates 4 hours of ICP data (480 readings, one every 30 s) with a normal baseline of about 11 mmHg, random sensor noise, and 8 brief movement artifacts. Two pathological events are injected: a slow shunt-blockage climb (about +14 mmHg over ~40 minutes, then a recovery) and a sharp acute spike (about +16 mmHg over ~8 minutes).
* **Signal Processing & Early Warning:**	Smooths the signal, then applies four checks: a sustained-threshold rule (>20 mmHg for 5+ min), a rate-of-change rule, an Isolation Forest anomaly detector, and a 15-minute linear-trend forecast. These combine into a normal, warning, or high-risk label for each reading.
* **Visualization of Data:**	Plays the data back like a bedside monitor. It shows a color-coded big number (green for 0-15 mmHg normal, yellow for 15-20 mmHg elevated, red for >20 mmHg danger), a trend chart with shaded risk bands, the forecast, which rule fired, and an alert log. A Simulate acute event button injects a live pressure spike.
* **Viewing History:** A slider (with ±5-minute buttons and a Return to live button) is implemented for viewing past data, so users can identify which event triggered an abnormal brain pressure at which time. The stream keeps running while you review.

## 🧠 Detection Logic

* **Smoothing:**	A 5-sample centered rolling median (ignores outliers), followed by an exponential moving average (span 8). All later checks use this smoothed signal.
* **Threshold warning:**	Smoothed ICP > 20 mmHg for at least 5 consecutive minutes (10 samples).
* **Rate-of-change warning:**	Sample-to-sample change in smoothed ICP with an absolute value > 0.35 mmHg per 30 s (about 0.7 mmHg/min). It uses the absolute value, so a fast drop also triggers it.
* **Anomaly detection:**	`IsolationForest` (200 trees, contamination=0.12) on three features: smoothed level, dP/dt, and a 5-sample rolling standard deviation of the raw signal. It is unsupervised and never sees the ground-truth labels.
* **15-minute forecast:**	A linear fit over the trailing 20 smoothed samples (10 minutes), projected 30 samples (15 minutes) ahead. This is a lightweight substitute for a Long Short-Term Memory Network (LSTM).
* **Risk fusion:** High risk if the threshold warning is above 20 mmHg for 5+ minutes, or 15-minute forecast exceeds 25 mmHg. Warning if the rate-of-change flag or anomaly flag is set, or the forecast exceeds 20 mmHg. Otherwise normal.

## 🛠️ Key Features & Technical Architecture

* **Synthetic Physiological Data Generation (`generate_data.py`):** Programmed a mathematical telemetry simulator generating 4 hours ($480$ samples at $30\text{-second}$ intervals) of continuous ICP dynamics, incorporating baseline physiological drift ($\approx 11\text{ mmHg}$), Gaussian sensor noise, patient movement artifacts, slow occlusion gradients, and acute pathological surges.
* **Hybrid Signal Analytics & Anomaly Detection (`analyze.py`):**
  * **Digital Filtering & Smoothing:** Applied rolling window transformations (`pandas`, `NumPy`) to mitigate transient motion artifacts while preserving true pressure wave dynamics.
  * **Rule-Based Physiological Thresholding:** Implemented sustained-elevation logic ($\text{ICP} > 20\text{ mmHg}$ for $\ge 5\text{ consecutive minutes}$) alongside derivative rate-of-change ($dP/dt$) triggers.
  * **Unsupervised Machine Learning:** Integrated an `IsolationForest` ensemble model (`scikit-learn`) to detect multidimensional structural anomalies directly from raw waveform morphology without relying on ground-truth event labels.
  * **Short-Horizon Predictive Forecasting:** Engineered a rolling linear regression window to forecast $15\text{-minute}$ future pressure trends and slope trajectories as a lightweight, dependency-free substitute for recurrent neural networks (e.g., LSTMs).
* **Interactive Web Telemetry Dashboard (`dashboard.html`):** Developed a real-time web interface mimicking a clinical bedside monitor, complete with color-coded risk stratification ("Normal", "Warning", "High Risk"), dynamic trend charts with safety boundaries, alert logs, and an interactive event-injection trigger.

## 📝 Known Limitations
* The very first reading is flagged as an anomaly when the smoothing and rate features are still warming up, which produces an alert at the start of every playback.
* The forecast is a simple linear trend, not a trained model.
* The dashboard is a replay of precomputed results, not a live sensor feed. The "Simulate acute event" button adds approximate flags in the browser (it does not run the Python models).
* Normal intracranial pressure (ICP) ranges for adults, children, and infants vary greatly. Detection thresholds are illustrative and have not been clinically validated.

## 💻 Tech Stack & Dependencies

| Category | Technology / Library |
| :--- | :--- |
| **Language** | Python 3, HTML5/CSS3/JavaScript |
| **Data Processing** | `Pandas`, `NumPy`, `JSON` |
| **Machine Learning** | `scikit-learn` (`IsolationForest`) |
| **Data Visualization** | `Chart.js` |

## ⚠️ Disclaimer

This software is developed solely for educational and portfolio purposes.
All data streams, waveform inputs, and pressure values are synthetic and do not represent real human patient data.
The analytical pipeline, anomaly flags, and predictive trend forecasts do not constitute medical software, clinical diagnosis, or treatment recommendations.
Real-world intracranial pressure interpretation and hydrocephalus management must always be conducted by qualified medical professionals using regulatory-approved medical hardware and software.
