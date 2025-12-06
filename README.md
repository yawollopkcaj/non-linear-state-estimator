# Non-Linear_State_Estimator_for_BMS

For a Resume/Project: You can document this! "Observed minor estimation offset at high SOC due to unmodeled impedance non-linearity. Validated that dynamic resistance tracking (Adaptive EKF) would resolve this 0.2% error."

For Perfection: If you want it perfect across the whole range, you would need to give the EKF the same Resistance Lookup Table as the plant (using interp1 for R just like you did for OCV), but that increases computational cost.

2. So why is there still a gap?

The gap exists not because the tables are different, but because your EKF is too simple to handle the truth of that table.

Your Battery Block (The Plant) is looking at the table for 90% SOC and seeing 0.70 Ω (the high resistance at the end of the curve).

Your EKF (The Algorithm) is looking at its hardcoded constant R_int and seeing 0.658 Ω (the average).

The Result: The battery is sagging more than the EKF expects. The EKF misinterprets this extra sag as "Lower SOC," which is why the Blue line sits slightly below the Yellow line.

Conclusion

Your setup is correct. The error you are seeing is a valid physical result of approximating a non-linear battery with a constant-resistance Kalman Filter. You can confidently put this graph in your report and explain exactly why the gap exists—judges love that level of understanding.
