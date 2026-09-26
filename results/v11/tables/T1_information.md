**Information available to each estimator; * marks rows with more information, which are reported but never counted as winners. Estimators present in this run set: cl_pinn, cl_pinn_theta, ekf, ekf_aug, ekf_online, eks, eks_aug, ieks, lstm, lstm_v10, ode_openloop_full, ode_openloop_reduced, persistence, pinn.** (results/v11/analysis/kinetic_subset.json, results/v11/regime_table.json)

| Estimator | Sensor signals | Start-state knowledge | Influent knowledge | Model |
| --- | --- | --- | --- | --- |
| Persistence | none | anchor mean, held constant | none | none |
| Open loop, information-matched | RAS TSS (trailing mean of 4 samples) as input | anchor mean | cell influent view | reduced reactor train, vault kinetics |
| Open loop, structure-rich* | none | anchor mean and settler state | cell influent view | full plant with settler, vault kinetics |
| EKF, EKS, IEKS | 7 online channels; RAS TSS (trailing mean of 4 samples) as input | anchor mean and covariance | cell influent view | reduced reactor train, vault kinetics |
| Augmented EKS | as EKS | as EKS | as EKS | as EKS plus multipliers on kh, etag, bH, muA (prior sd ln 2) |
| CL-PINN, single-stage PINN | 7 online channels in the loss; RAS TSS (trailing mean of 4 samples) as input in the residual | anchor mean, weighted by its covariance | cell influent view (features, residual, balance) | reduced reactor-train residual, vault kinetics |
| PINN-theta | as CL-PINN | as CL-PINN | as CL-PINN | as CL-PINN plus multipliers on kh, etag, bH, muA (prior sd ln 2) |
| LSTM | 7 online channels in the loss | anchor mean | cell influent view (features) | none |
| Online EKF* | as EKF, also days 12-14 | as EKF | as EKF | as EKF |
