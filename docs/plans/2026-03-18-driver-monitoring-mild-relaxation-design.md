# Driver Monitoring Mild Relaxation Design

**Date:** 2026-03-18

## Summary

Tune active driver-monitoring thresholds to reduce repeated audible escalation and reduce how quickly brief active-distraction episodes accumulate into the ignition-cycle `Distraction Level Too High` lockout.

This change stays intentionally small. It only adjusts active distraction timing and smoothing in `selfdrive/monitoring/helpers.py`. It does not add a new mode, UI toggle, or Carnival-specific branch logic.

## Current State

- The synced `kia-carnival-26-sp-dev` branch carries upstream Sunnypilot/Openpilot driver-monitoring behavior unchanged.
- The relevant active-monitoring constants are:
  - `_DISTRACTED_TIME = 11.0`
  - `_DISTRACTED_PRE_TIME_TILL_TERMINAL = 8.0`
  - `_DISTRACTED_PROMPT_TIME_TILL_TERMINAL = 6.0`
  - `_DISTRACTED_FILTER_TS = 0.25`
- On the current baseline, continuous active distraction reaches:
  - prompt/orange at about `5.25s`
  - terminal/red at about `11.25s`

## Goals

- Make brief glances feel less noisy in active monitoring.
- Slightly reduce how quickly active-distraction events accumulate toward a lockout.
- Keep the code close to upstream and easy to rebase.
- Preserve the existing no-face / uncertain-model fallback and the existing lockout semantics.

## Non-Goals

- No “chill mode” or user-facing driver-monitoring setting.
- No changes to passive/no-face awareness timing.
- No changes to `_MAX_TERMINAL_ALERTS` or `_MAX_TERMINAL_DURATION`.
- No changes to model thresholds, camera handling, or alert text.

## Chosen Approach

Adjust only the active-monitoring timing/smoothing constants in `selfdrive/monitoring/helpers.py`:

- `_DISTRACTED_TIME`: `11.0 -> 13.0`
- `_DISTRACTED_PRE_TIME_TILL_TERMINAL`: `8.0 -> 9.0`
- `_DISTRACTED_PROMPT_TIME_TILL_TERMINAL`: `6.0 -> 7.0`
- `_DISTRACTED_FILTER_TS`: `0.25 -> 0.35`

Expected effect:

- Continuous active-distraction prompt shifts from about `5.25s` to `6.35s`
- Continuous active-distraction terminal/red shifts from about `11.25s` to `13.35s`
- Short 2-second distraction bursts remain below alert thresholds

This is the smallest change that directly addresses the complaint without weakening the passive/no-face path or changing the hard lockout policy.

## Why This Approach

### Rejected: relax lockout counters

Changing `_MAX_TERMINAL_ALERTS` or `_MAX_TERMINAL_DURATION` would materially weaken the lockout policy and drift farther from upstream safety behavior.

### Rejected: add a configurable DM mode

A user-facing mode would add UI/config maintenance and increase long-term merge drift for a problem that can be addressed with a single upstream-shaped tuning change.

## Files Affected

- `selfdrive/monitoring/helpers.py`
- `selfdrive/monitoring/test_monitoring.py`

## Testing Strategy

- Keep the existing monitoring suite green.
- Add a regression test that proves the new active-monitoring timing is delayed relative to the current baseline.
- Add a regression test that confirms the lockout thresholds remain unchanged and that brief distraction bursts still do not alert.

## Risks

- Active distraction alerts will be slightly slower to escalate for truly continuous distraction.

## Mitigations

- The change does not touch passive/no-face awareness timing.
- The change does not touch the terminal-alert count or duration gates.
- The change is limited to one helper file and one test file, which keeps review and future upstream syncs straightforward.
