# Non-Linear State Estimator for High-Voltage Battery Systems (EKF)

## Project Overview

This project implements a **discrete-time Extended Kalman Filter (EKF)** to estimate the State of Charge (SOC) of a 140S 5P Li-Ion Battery Pack (Molicel P30B chemistry) under dynamic race conditions.

Designed using MATLAB/Simulink and Simscape Electrical, the system moves beyond simple Coulomb Counting by fusing current integration with non-linear Open Circuit Voltage (OCV) measurements, achieving <0.1% estimation error in validated simulations.

Tech Stack: MATLAB, Simulink, Simscape Electrical, Control Theory, Sensor Fusion.

## Performance under Dynamic Load (The "Money Plot")

The system was stress-tested using a **US06-style** dynamic drive cycle, simulating high-current discharge (acceleration) and sharp negative current spikes (regenerative braking).

<img width="555" alt="Dynamic Race Lap Performance" src="https://github.com/user-attachments/assets/0269ccfc-9b45-4ef8-bc11-820846c71549" />

## Key Engineering Insights from this Graph:

1. **Regen Robustness:** The EKF (Blue) tracks the Real SOC (Yellow) perfectly during regenerative braking pulses (e.g., at T=405s). This verifies the stability of the Jacobian linearization even when current polarity flips.

2. **Transient Tracking:** The filter maintains lock during 100A+ current steps, proving that the Process Noise ($Q$) and Measurement Noise ($R$) covariance matrices are tuned correctly for high-dynamic applications.

## Root Cause Analysis: The "Impedance Gap"

During initial development using a Zeroth-Order Static Model (Constant Resistance), a persistent estimation offset of ~0.2% was observed at high SOC, despite perfect Coulomb counting.

<img width="1518" alt="Static Model Offset" src="https://github.com/user-attachments/assets/e9a58f18-4381-48f3-9b76-0b3e08eee57e" />

## The Investigation:

Observation: The EKF consistently underestimated SOC in the 90-100% range.

Physics: The Molicel P30B cell chemistry exhibits a "hockey stick" impedance curve (resistance rises sharply at high SOC).

The Conflict: The Plant (Simscape) was modeling this non-linear resistance rise (~0.70 $\Omega$), while the EKF was assuming a static average (~0.658 $\Omega$).

The Result: The battery sagged more than the model expected. The EKF interpreted this extra voltage drop as "Low SOC" rather than "High Resistance."

Analysis: "Observed minor estimation offset at high SOC due to unmodeled impedance non-linearity. Validated that dynamic resistance tracking (Adaptive EKF) would resolve this 0.2% error."

## Implementation: Dynamic Resistance & Temperature

To resolve the impedance gap, the EKF architecture was upgraded to perform Dynamic Resistance Lookup at every time step ($dt=0.01s$), interpolating resistance based on the predicted state vector.

**Core Algorithm Snippet (MATLAB Function Block):**

```MATLAB
% STEP B: MEASUREMENT ESTIMATE
% 1. Estimate OCV based on predicted SOC
v_ocv_pred = interp1(soc_bp, ocv_v, x_pred, 'linear', 'extrap');

% 2. Estimate RESISTANCE based on predicted SOC (Dynamic Lookup)
% This matches the Plant's behavior, eliminating the offset error.
r_int_pred = interp1(soc_bp, r0_v,  x_pred, 'linear', 'extrap');

% 3. Estimate Terminal Voltage
v_est_pred = v_ocv_pred - (current_A * r_int_pred);
```

**Result after Implementation:**
The estimation error was effectively eliminated, resulting in near-perfect convergence.

<img width="555" alt="Dynamic Resistance Fix" src="https://github.com/user-attachments/assets/91c0937b-c7a8-422b-a5f1-00b765e12836" />

## Validation Strategy: Breaking the "Tautology"

A common pitfall in BMS simulation is creating a "Tautology"—where the Model parameters perfectly match the Plant parameters ($X = X$). While this verifies the code is bug-free, it does not validate real-world performance.

To validate the robustness of the algorithm, I introduced controlled Parameter Mismatches to simulate production variance and aging.

Test Case: Resistance Mismatch (Aging Simulation)

Condition: Plant Resistance scaled by 1.05x (5% degradation). Model Resistance kept at nominal.

Outcome: The EKF successfully converged despite the model mismatch, demonstrating the filter's ability to prioritize voltage measurement updates ($K$ gain) when prediction errors accumulate.

<img width="1512" alt="Resistance Mismatch Test" src="https://github.com/user-attachments/assets/7f981f49-e57e-4d11-b341-6d87bb3ab146" />

## Future Roadmap: Adaptive EKF (AEKF)

While the current Dynamic EKF handles known non-linearities, it relies on a pre-characterized lookup table. The next phase of this project involves implementing a Dual-Estimation Adaptive EKF to estimate both SOC (State) and SOH/Internal Resistance (Parameter) concurrently in real-time.
