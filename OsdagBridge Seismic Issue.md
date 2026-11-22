# Automatic seismic loading omits the governing vehicle weight

**Suggested priority: High.** Reproduced in OsdagBridge d74f56d on Windows using Python 3.12.16 and the baseline inputs.osi.

## Reproduction

1. Load inputs.osi (30.0 m span, 8.5 m carriageway, Mumbai, Optimized).
2. Keep the seismic live load mode on Automatic and run Design.
3. Compare the seismic input line with the later governing live-load selection.
4. For a controlled comparison, rebuild the baseline loads, invoke seismic setup, assign the actual governing case selected by the baseline replay, and invoke the same seismic setup again.

| Backend value | Normal baseline sequence | Controlled comparison |
| --- | ---: | ---: |
| Governing vehicle weight | 0.0 kN | 981.0 kN |
| Longitudinal seismic force | 549.29 kN | 549.29 kN |
| Transverse seismic force | 549.29 kN | 593.24 kN |
| Vertical seismic force | 366.11 kN | 395.40 kN |

The normal run identified Case2 1xClass70R after creating the seismic load cases. In the controlled comparison, using that case increased the application's transverse and vertical forces by about 8.0% relative to the normal run.

## Cause and fix

In src/osdagbridge/core/bridge_types/plate_girder/plategirderbridge.py, design() invokes seismic setup at lines 706–707. Stage 4F then selects the governing vehicle at lines 631–635. In analyser.py:1066–1078, _get_governing_ll_kN() returns zero when the governing case name is unset. The force calculation at analyser.py:1198–1202 uses that result in its transverse and vertical live-load contributions.

Select the governing vehicle before building automatic seismic cases, with those cases registered before generating the load combinations. Preserve the explicit Custom override. Add a regression test for a nonzero automatic vehicle case, plus a run with no vehicle case and a run with an explicit override.
