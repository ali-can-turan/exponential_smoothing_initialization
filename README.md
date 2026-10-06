# Exponential Smoothing Initialization Strategies & Model Benchmarks

## 📌 Project Overview
This project evaluates the performance of **Holt-Winters Exponential Smoothing (ETS)** models under different **initialization strategies** and **damped trend dynamics**. 

The primary goal is to benchmark how initial state selection ($a_0, b_0, F_0$), train/init data partitioning ($\text{init} \subset \text{train}$ vs. $\text{init} \not\subset \text{train}$), and parameter estimation algorithms impact out-of-sample forecast accuracy ($R^2$, WMAPE, and Residual Bias).

---

## 🔬 Evaluated Model Configurations

The benchmark compares four distinct initialization and estimation setups:

1. **`heuristic, init ⊂ train`**: 
   Heuristic initialization based on an initial window ($t_0 \dots t_k$), where the initialization data is **included** as a subset of the training set.
2. **`heuristic, init ⊄ train`**: 
   Heuristic initialization based on an initial window ($t_0 \dots t_k$), where initialization and training data are **mutually exclusive**.
3. **`heuristic, no init`**: 
   Automatic heuristic initialization carried out over the full training dataset without a pre-segmented initial window.
4. **`estimated, no init`**: 
   Joint maximum likelihood / regression estimation of initial states ($a_0, b_0$) alongside smoothing parameters ($\alpha, \beta, \gamma, \phi$) across the entire training set.

---

## 📊 Benchmark Results

| Parameter / Metric | heuristic, init ⊂ train | heuristic, init ⊄ train | heuristic, no init | estimated, no init |
| :--- | :---: | :---: | :---: | :---: |
| **level_factor ($a_0$)** | 274.400 | 276.145 | 329.264 | 329.101 |
| **trend_factor ($b_0$)** | 0.048 | 0.048 | 2.190 | 25.738 |
| **level_parameter ($\alpha$)** | 0.055 | 0.043 | 0.000 | 0.000 |
| **trend_parameter ($\beta$)** | **0.000** | **0.000** | **0.000** | **0.000** |
| **dampening_parameter ($\phi$)** | 0.986 | 0.923 | 0.925 | 0.800 |
| **seasonality_parameter ($\gamma$)** | 0.705 | 0.789 | 0.000 | 0.000 |
| **train_residual_mean** | -0.000 | 0.000 | -0.000 | 0.000 |
| **train_r2_score** | 0.823 | 0.612 | 0.810 | 0.833 |
| **train_WMAPE** | 0.107 | 0.158 | 0.114 | 0.111 |
| **test_residual_mean** | -15.042 | -19.044 | -19.401 | -20.666 |
| **test_r2_score** | **0.885** | 0.859 | 0.777 | 0.802 |
| **test_WMAPE** | **0.090** | 0.100 | 0.112 | 0.104 |

---

## 🔑 Key Experimental Insights

### 1. Champion Model: `heuristic, init ⊂ train`
* **Top Generalization:** Including the initialization window within the evaluation set achieved the highest test score (**$R^2 = 0.885$**) and the lowest forecast error (**WMAPE = 0.090**).

### 2. The $\beta = 0.000$ (Zero Trend Parameter) Effect
* Across all four configurations, the optimizer drove **`trend_parameter` ($\beta$) to $0.000$**.
* **Theoretical Reason:** When dampening ($\phi$) is active, updating trend slope dynamically on noisy data increases model variance. The solver locks $\beta = 0.000$ to projecting future trajectory purely via geometric dampening ($b_t = \phi^t \cdot b_0$).

### 3. Failure of `no init` Strategies in Seasonal Dynamics
* Both `heuristic, no init` and `estimated, no init` models collapsed all the parameters to zero: pure regression.
* By failing to capture trend and seasonal variations, these models generated lower test accuracy ($R^2 = 0.777 - 0.802$).

---

## 💡 Practical Recommendations

> 1. **Do Not Discard Initialization Data:** For limited or medium-length time-series data, excluding initialization periods (`init ⊄ train`) leads to severe training accuracy loss ($R^2 = 0.612$) without delivering any out-of-sample limited benefits.
> 2. **Dampened Trend Regularization:** When using damped trend models ($\phi < 1.0$), a zero trend parameter ($\beta = 0.000$) is a normal optimization outcome that stabilizes long-term forecasts.
> 3. **Explicit Initialization Preserves Seasonality:** Heuristic pre-segmentation ensures robust capture of trend and seasonal components, which automated automated / estimated initialization methods may fail to identify.
