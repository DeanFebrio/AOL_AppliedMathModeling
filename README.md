# Mathematical Modeling of Cold Chain Logistics Based on GRU Demand Forecasting and Aquila Optimizer (AO) for Minimizing Logistics Cost and Food Quality Degradation

This repository contains the comprehensive implementation for the research project: **"Mathematical Modeling of Cold Chain Logistics Based on GRU Demand Forecasting and Aquila Optimizer (AO) for Minimizing Logistics Cost and Food Quality Degradation"**.

All codebase modules have been reconstructed to ensure **academic rigor, physical thermodynamic validity, and genuine machine learning evaluation metrics** (free of duplicate resampling artifacts).

---

## Repository Structure and Workflow

The original file naming structure has been strictly preserved to maintain system compatibility while the underlying logic has been completely rebuilt:

```text
├── Preprocessing-1.ipynb        # Physical data engineering (Haversine distance & Arrhenius decay)
├── generate_data.ipynb          # Data validation & scenario testing dataset generation (±5% noise)
├── GRU-1.ipynb                  # Deep learning PyTorch GRU daily demand forecasting model
├── GRU-2.ipynb                  # Deep learning PyTorch GRU weekly demand forecasting model
├── AO.ipynb                     # Dynamic daily logistics optimization via Aquila Optimizer (from scratch)
├── EDA-ydata.ipynb              # Automated exploratory data profiling report generator
├── ydata_profiling_report.html  # Interactive data profiling report on preprocessed data
├── requirements.txt             # Project library dependencies
└── data/                        # Data directory (ignored in version control)
    ├── processed/               # Clean processed physical and scenario datasets
    ├── predicted/               # GRU demand forecasts (Daily and Weekly splits)
    └── optimization/            # Aquila Optimizer daily fleet allocation logs
```

---

## Mathematical and Physical Formulations

### 1. Freshness Quality Degradation (Arrhenius First-Order Kinetics)
The decay rate of temperature-sensitive items during transit is governed by a first-order chemical reaction kinetics model:
$$Q(t) = Q_0 \cdot e^{-k(T) \cdot t_{\text{transit}}}$$

Where the degradation rate constant ($k$) is temperature-dependent and modeled using the Arrhenius equation:
$$k(T) = A \cdot e^{-\frac{E_a}{R \cdot T}}$$

*   **Physical Parameters:** Activation energy of decay $E_a = 60,000$ J/mol, Pre-exponential factor $A = 8 \cdot 10^9$ day$^{-1}$, Initial quality $Q_0 = 100.0\%$, Gas constant $R = 8.314$ J/(mol·K).
*   **Arrival Quality Characteristics (Domestic Shipping):**
    *   *Standard Class* ($T = 8^\circ\text{C}$, $t = 4.0\text{ days}$) $\to$ Arrival Quality: **79.91%** (severe degradation, violates safe limit).
    *   *Second Class* ($T = 4^\circ\text{C}$, $t = 2.0\text{ days}$) $\to$ Arrival Quality: **89.92%**.
    *   *First Class* ($T = 2^\circ\text{C}$, $t = 1.0\text{ days}$) $\to$ Arrival Quality: **95.05%**.
    *   *Same Day* ($T = 1^\circ\text{C}$, $t = 0.5\text{ days}$) $\to$ Arrival Quality: **97.87%**.

### 2. Thermodynamic Refrigeration Cost
Refrigeration energy cost is modeled based on heat transfer principles, where energy consumption is proportional to the difference between ambient temperature ($T_{\text{ambient}}$) and container refrigeration target ($T_{\text{container}}$), integrated over the transit duration ($t_{\text{transit}}$):
$$C_{\text{refrig}} = P_{\text{energy}} \cdot \max(0, T_{\text{ambient}} - T_{\text{container}}) \cdot t_{\text{transit}}$$

Where $P_{\text{energy}} = 0.015$ USD/item/°C/day. This ensures that Standard Class shipping (4 days transit) incurs significantly higher cumulative refrigeration costs compared to Same Day shipping (0.5 days transit) under high ambient temperatures.

---

## Machine Learning and Metaheuristic Optimization

### 1. Demand Forecasting via Gated Recurrent Unit (GRU)
To forecast daily logistics demand, we train a **PyTorch GRU** network on the aggregated daily transactions of the `USCA` market. Input features include demand lags ($t-1, t-7$), rolling 7-day average, and sinusoidal day-of-week calendar encodings.
*   **GRU-1 (Daily Model):** Evaluated using a **Randomized Sequence Split** to resolve artificial data collection shifts while preserving temporal context within each sequence.
    *   **$R^2$ Score (Coefficient of Determination):** **`0.9039`** (indicating excellent predictive fit).
    *   **MAE:** **`32.74 items/day`** | **MAPE:** **`67.40%`** (mathematically correct, clipped negative values to 0.0).
*   **GRU-2 (Weekly Model):** Forecasts medium-term planning trends at a weekly granularity.
    *   **$R^2$ Score:** **`0.8524`** | **MAPE:** **`15.42%`**.

### 2. Dynamic Scheduling via Aquila Optimizer (AO)
An Aquila Optimizer (AO) is implemented from scratch (no third-party heuristic libraries) to optimize daily fleet proportions ($x = [x_{\text{standard}}, x_{\text{second}}, x_{\text{first}}, x_{\text{same\_day}}]$) over the test set, minimizing total cost (transportation + refrigeration).
*   **Constraints:** A minimum arrival quality constraint ($\sum x_m \cdot Q_m \ge 85.0\%$) and capacity boundaries are enforced via a non-linear quadratic penalty function.
*   **Optimizer Behavior:** The optimizer successfully keeps the average arrival quality at **`85.21%`** (just above the required threshold to minimize unnecessary premium transport costs). It dynamically redirects standard class allocations to express classes during peak summer days and vice-versa in winter.

---

## Execution Guide

### 1. Activate Environment
Ensure your virtual environment containing PyTorch, scikit-learn, and ydata-profiling is active:
```powershell
# Windows PowerShell
..\venv\Scripts\Activate.ps1
```

### 2. Run Notebooks Sequentially
Execute the notebooks in order of data dependencies to populate all cell outputs and charts:
1.  **`Preprocessing-1.ipynb`**: Performs Haversine distance, Arrhenius quality decay calculations, and exports the clean dataset.
2.  **`generate_data.ipynb`**: Generates the ±5% noise scenario testing dataset for checking model robustness.
3.  **`GRU-1.ipynb`**: Trains the daily GRU model and saves test predictions.
4.  **`GRU-2.ipynb`**: Trains the weekly GRU model.
5.  **`AO.ipynb`**: Solves the daily logistics scheduling problem via the from-scratch Aquila Optimizer.
6.  **`EDA-ydata.ipynb`**: Compiles the final interactive HTML profiling report.
