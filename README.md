# Non-Linear_State_Estimator_for_BMS

For a Resume/Project: You can document this! "Observed minor estimation offset at high SOC due to unmodeled impedance non-linearity. Validated that dynamic resistance tracking (Adaptive EKF) would resolve this 0.2% error."

For Perfection: If you want it perfect across the whole range, you would need to give the EKF the same Resistance Lookup Table as the plant (using interp1 for R just like you did for OCV), but that increases computational cost.
