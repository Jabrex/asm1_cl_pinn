# ASM1 CL+PINN benchmark

All numbers below come from a single vault parameter set (20 degrees C, `data/asm1.json`). BSM1 supplies the plant geometry, flows and influent composition only.

Track A = measured components S_NH, S_NO, S_O.
Track B = never-measured components S_I, S_S, X_I, X_S, X_B_H, X_B_A, X_P, S_ND, X_ND, S_ALK, S_N2.

## train

### Track A - NRMSE (lower is better)

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.1378 | 0.1459 | 0.1551 | 0.1579 |
| cl_pinn | 0.0245 | 0.0249 | 0.0289 | 0.0328 |
| lstm | 0.1439 | 0.1486 | 0.1536 | 0.1593 |
| ode_openloop | 0.0003 | 0.0003 | 0.0003 | 0.0003 |
| persistence | 0.2094 | 0.2094 | 0.2094 | 0.2094 |
| pinn | 0.0371 | 0.0402 | 0.0404 | 0.0494 |

### Track A - R2

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.3976 | 0.3993 | 0.3878 | 0.3696 |
| cl_pinn | 0.9862 | 0.9864 | 0.9823 | 0.9787 |
| lstm | 0.3840 | 0.3834 | 0.3796 | 0.3703 |
| ode_openloop | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| persistence | 0.0815 | 0.0815 | 0.0815 | 0.0815 |
| pinn | 0.9733 | 0.9690 | 0.9699 | 0.9549 |

### Track B - NRMSE (lower is better)

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.2694 | 0.2941 | 0.3569 | 0.3937 |
| cl_pinn | 0.1418 | 0.1280 | 0.1861 | 0.1916 |
| lstm | 0.2787 | 0.3192 | 0.4391 | 0.4527 |
| ode_openloop | 0.0006 | 0.0006 | 0.0006 | 0.0006 |
| persistence | 0.2527 | 0.2527 | 0.2527 | 0.2527 |
| pinn | 0.4546 | 0.4360 | 0.3611 | 0.3952 |

### Track B - R2

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | -0.6663 | -1.1318 | -3.0616 | -4.3412 |
| cl_pinn | 0.1903 | 0.2967 | -0.9734 | -0.8100 |
| lstm | -0.8202 | -1.5333 | -5.4223 | -6.5714 |
| ode_openloop | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| persistence | -0.4550 | -0.4550 | -0.4550 | -0.4550 |
| pinn | -9.2954 | -10.3392 | -5.0068 | -5.7523 |

## holdout

### Track A - NRMSE (lower is better)

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.1310 | 0.1382 | 0.1477 | 0.1548 |
| cl_pinn | 0.0372 | 0.0571 | 0.0519 | 0.0477 |
| lstm | 0.1387 | 0.1420 | 0.1642 | 0.1741 |
| ode_openloop | 0.0005 | 0.0005 | 0.0005 | 0.0005 |
| persistence | 0.1879 | 0.1879 | 0.1879 | 0.1879 |
| pinn | 0.0449 | 0.0500 | 0.0418 | 0.0547 |

### Track A - R2

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.5484 | 0.5518 | 0.5367 | 0.5076 |
| cl_pinn | 0.9748 | 0.9468 | 0.9550 | 0.9624 |
| lstm | 0.5271 | 0.5310 | 0.4568 | 0.4314 |
| ode_openloop | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| persistence | 0.3400 | 0.3400 | 0.3400 | 0.3400 |
| pinn | 0.9641 | 0.9539 | 0.9703 | 0.9467 |

### Track B - NRMSE (lower is better)

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.3841 | 0.4181 | 0.5260 | 0.6012 |
| cl_pinn | 0.2202 | 0.2003 | 0.3103 | 0.3001 |
| lstm | 0.3831 | 0.4593 | 0.6317 | 0.7168 |
| ode_openloop | 0.0017 | 0.0017 | 0.0017 | 0.0017 |
| persistence | 0.3716 | 0.3716 | 0.3716 | 0.3716 |
| pinn | 0.7972 | 0.7497 | 0.5200 | 0.5867 |

### Track B - R2

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | -1.9700 | -2.7922 | -6.5147 | -9.5556 |
| cl_pinn | -0.4589 | -0.1696 | -3.0762 | -2.3584 |
| lstm | -2.0548 | -3.8295 | -10.3341 | -13.9204 |
| ode_openloop | 0.9999 | 0.9999 | 0.9999 | 0.9999 |
| persistence | -1.9230 | -1.9230 | -1.9230 | -1.9230 |
| pinn | -22.5682 | -22.6660 | -9.5920 | -12.2416 |

## rain

### Track A - NRMSE (lower is better)

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.1494 | 0.1549 | 0.1602 | 0.1630 |
| cl_pinn | 0.0685 | 0.0677 | 0.0716 | 0.0658 |
| lstm | 0.1527 | 0.1549 | 0.1567 | 0.1638 |
| ode_openloop | 0.0003 | 0.0003 | 0.0003 | 0.0003 |
| persistence | 0.2000 | 0.2000 | 0.2000 | 0.2000 |
| pinn | 0.0857 | 0.1153 | 0.0624 | 0.0757 |

### Track A - R2

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.3990 | 0.3893 | 0.3744 | 0.3500 |
| cl_pinn | 0.9003 | 0.9023 | 0.8895 | 0.9101 |
| lstm | 0.3874 | 0.3757 | 0.3745 | 0.3556 |
| ode_openloop | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| persistence | 0.1065 | 0.1065 | 0.1065 | 0.1065 |
| pinn | 0.8333 | 0.6925 | 0.9207 | 0.8818 |

### Track B - NRMSE (lower is better)

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | 0.2558 | 0.2556 | 0.2830 | 0.3107 |
| cl_pinn | 0.1816 | 0.1616 | 0.1803 | 0.1850 |
| lstm | 0.2557 | 0.2661 | 0.3201 | 0.3341 |
| ode_openloop | 0.0004 | 0.0004 | 0.0004 | 0.0004 |
| persistence | 0.2328 | 0.2328 | 0.2328 | 0.2328 |
| pinn | 0.3474 | 0.3195 | 0.2737 | 0.2978 |

### Track B - R2

| model | sigma=0.00 | sigma=0.05 | sigma=0.10 | sigma=0.15 |
| --- | --- | --- | --- | --- |
| cl_lstm | -0.5652 | -0.5561 | -1.0580 | -1.5672 |
| cl_pinn | -0.1490 | 0.0928 | -0.2603 | -0.2206 |
| lstm | -0.5641 | -0.7031 | -1.8472 | -2.1646 |
| ode_openloop | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| persistence | -0.2868 | -0.2868 | -0.2868 | -0.2868 |
| pinn | -4.5208 | -4.4562 | -1.8442 | -2.2134 |

## Note on the Track B baselines

`lstm` and `cl_lstm` receive no sensor-derived training signal on the never-measured components: those states appear in no sensor channel and the baselines carry no physics term. They do receive the shared t=0 initial-condition anchor and the output-head scale derived from Z(0). Their Track B numbers therefore reflect that anchor plus initialisation, not a fitting failure. This is the comparison the benchmark was built to make, so the rows are reported rather than omitted.

Non-learned reference rows: `persistence` holds the known t=0 state constant; `ode_openloop` integrates the plant model forward from that same state with the known influent - the perfect-model information bound for this in-model benchmark. Neither uses any sensor data.

## Ground-truth dataset descriptors

```json
{
  "dry": {
    "effluent": {
      "eqi_kg_pu_per_day": 9370.504098364849,
      "N_tot_pct_time_over_limit": 59.03345724907063,
      "N_tot_crossings": 14,
      "COD_pct_time_over_limit": 0.0,
      "COD_crossings": 0,
      "S_NH_pct_time_over_limit": 88.84758364312268,
      "S_NH_crossings": 8,
      "TSS_pct_time_over_limit": 0.0,
      "TSS_crossings": 0,
      "BOD5_pct_time_over_limit": 0.0,
      "BOD5_crossings": 0,
      "S_NH_p95": 19.16981824320896,
      "N_tot_p95": 23.81682130637259,
      "TSS_p95": 16.471865372757403
    },
    "influent": {
      "flow_mean_achieved": 18445.693224073202,
      "flow_mean_target": 18446.0,
      "flow_min_achieved": 10000.000000000136,
      "flow_max_achieved": 31999.998094330036,
      "flow_range_target": [
        10000.0,
        32000.0
      ],
      "S_S": {
        "mean_achieved": 69.49927675014756,
        "mean_target": 69.5,
        "min_achieved": 55.00000000000001,
        "max_achieved": 119.99998601084614,
        "range_target": [
          55.0,
          120.0
        ]
      },
      "S_NH": {
        "mean_achieved": 31.55951082835645,
        "mean_target": 31.56,
        "min_achieved": 15.000013098428038,
        "max_achieved": 44.999998880541256,
        "range_target": [
          15.0,
          45.0
        ]
      },
      "X_S": {
        "mean_achieved": 202.3165652935203,
        "mean_target": 202.32,
        "min_achieved": 100.00000332163683,
        "max_achieved": 299.99999079983377,
        "range_target": [
          100.0,
          300.0
        ]
      }
    },
    "continuity_of_truth": 9.451160963223997e-13
  },
  "rain": {
    "effluent": {
      "eqi_kg_pu_per_day": 10284.95173833444,
      "N_tot_pct_time_over_limit": 56.208178438661704,
      "N_tot_crossings": 13,
      "COD_pct_time_over_limit": 0.0,
      "COD_crossings": 0,
      "S_NH_pct_time_over_limit": 85.27881040892194,
      "S_NH_crossings": 10,
      "TSS_pct_time_over_limit": 0.0,
      "TSS_crossings": 0,
      "BOD5_pct_time_over_limit": 0.0,
      "BOD5_crossings": 0,
      "S_NH_p95": 18.936626936241346,
      "N_tot_p95": 23.59181362296268,
      "TSS_p95": 19.219928222546336
    },
    "influent": {
      "flow_mean_achieved": 20796.81483527806,
      "flow_mean_target": 18446.0,
      "flow_min_achieved": 10000.000000000136,
      "flow_max_achieved": 52000.01620482786,
      "flow_range_target": [
        10000.0,
        32000.0
      ],
      "S_S": {
        "mean_achieved": 64.80327505388364,
        "mean_target": 69.5,
        "min_achieved": 19.338380413927602,
        "max_achieved": 119.99998601084586,
        "range_target": [
          55.0,
          120.0
        ]
      },
      "S_NH": {
        "mean_achieved": 29.446678684884816,
        "mean_target": 31.56,
        "min_achieved": 5.278486576626065,
        "max_achieved": 44.99999888054124,
        "range_target": [
          15.0,
          45.0
        ]
      },
      "X_S": {
        "mean_achieved": 188.79428544603974,
        "mean_target": 202.32,
        "min_achieved": 35.18435798132965,
        "max_achieved": 299.9999907998335,
        "range_target": [
          100.0,
          300.0
        ]
      },
      "rain_peak_achieved": 52000.01620482786,
      "rain_peak_target": 52000.0,
      "flow_mean_note": "series mean includes the rain event; the Table 5 anchor applies to the dry-weather component only",
      "dry_component_mean_achieved": 18445.693224073202
    },
    "continuity_of_truth": 9.451160963223997e-13
  }
}
```
