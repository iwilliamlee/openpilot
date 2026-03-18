# Driver Monitoring Mild Relaxation Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Make active driver-monitoring escalation slightly less aggressive for brief glances by delaying active prompt/red alerts and smoothing brief distraction detections, while leaving passive/no-face monitoring and lockout policy unchanged.

**Architecture:** Add one behavior-focused regression test in the monitoring suite that captures the later prompt/red timing, then make the minimal constant-only change in the active-monitoring settings. Keep the implementation isolated to `selfdrive/monitoring/helpers.py` so the branch stays close to upstream.

**Tech Stack:** Python, pytest, openpilot driver-monitoring helpers

---

### Task 1: Add A Failing Active-Monitoring Timing Regression Test

**Files:**
- Modify: `selfdrive/monitoring/test_monitoring.py`
- Test: `selfdrive/monitoring/test_monitoring.py`

**Step 1: Write the failing test**

Add a new test in `TestMonitoring` that runs `always_distracted` while engaged and asserts:

- `promptDriverDistracted` is not present just before `6.0s`
- `promptDriverDistracted` is present by about `6.5s`
- `driverDistracted` is not present just before `12.0s`
- `driverDistracted` is present by about `13.5s`

Use `DT_DMON`-based indices and `EventName.promptDriverDistracted` / `EventName.driverDistracted`.

**Step 2: Run test to verify it fails**

Run:

```bash
cd /Users/elite/.config/superpowers/worktrees/temp_sunnypilot/relax-driver-monitoring-20260318
uv run --python /opt/homebrew/bin/python3.12 --extra testing pytest selfdrive/monitoring/test_monitoring.py::TestMonitoring::test_active_monitoring_escalates_later -q -n 0
```

Expected: FAIL because the current branch still prompts at about `5.25s` and reaches terminal/red at about `11.25s`.

### Task 2: Apply The Minimal Active-Monitoring Tuning

**Files:**
- Modify: `selfdrive/monitoring/helpers.py`
- Test: `selfdrive/monitoring/test_monitoring.py`

**Step 1: Write minimal implementation**

In `DRIVER_MONITOR_SETTINGS.__init__`, change only these active-monitoring constants:

```python
self._DISTRACTED_TIME = 13.
self._DISTRACTED_PRE_TIME_TILL_TERMINAL = 9.
self._DISTRACTED_PROMPT_TIME_TILL_TERMINAL = 7.
self._DISTRACTED_FILTER_TS = 0.35
```

Do not change:

- `_AWARENESS_TIME`
- `_AWARENESS_PRE_TIME_TILL_TERMINAL`
- `_AWARENESS_PROMPT_TIME_TILL_TERMINAL`
- `_MAX_TERMINAL_ALERTS`
- `_MAX_TERMINAL_DURATION`

**Step 2: Run the focused test to verify it passes**

Run:

```bash
cd /Users/elite/.config/superpowers/worktrees/temp_sunnypilot/relax-driver-monitoring-20260318
uv run --python /opt/homebrew/bin/python3.12 --extra testing pytest selfdrive/monitoring/test_monitoring.py::TestMonitoring::test_active_monitoring_escalates_later -q -n 0
```

Expected: PASS

### Task 3: Verify The Monitoring Suite Still Passes

**Files:**
- Verify: `selfdrive/monitoring/helpers.py`
- Verify: `selfdrive/monitoring/test_monitoring.py`

**Step 1: Run the full monitoring test module**

Run:

```bash
cd /Users/elite/.config/superpowers/worktrees/temp_sunnypilot/relax-driver-monitoring-20260318
uv run --python /opt/homebrew/bin/python3.12 --extra testing pytest selfdrive/monitoring/test_monitoring.py -q -n 0
```

Expected: PASS

**Step 2: Run Ruff on touched files**

Run:

```bash
cd /Users/elite/.config/superpowers/worktrees/temp_sunnypilot/relax-driver-monitoring-20260318
uv run --python /opt/homebrew/bin/python3.12 --extra testing ruff check selfdrive/monitoring/helpers.py selfdrive/monitoring/test_monitoring.py
```

Expected: PASS

### Task 4: Commit The Completed Change

**Files:**
- Modify: `selfdrive/monitoring/helpers.py`
- Modify: `selfdrive/monitoring/test_monitoring.py`
- Modify: `docs/plans/2026-03-18-driver-monitoring-mild-relaxation-design.md`
- Create: `docs/plans/2026-03-18-driver-monitoring-mild-relaxation.md`

**Step 1: Commit**

Run:

```bash
cd /Users/elite/.config/superpowers/worktrees/temp_sunnypilot/relax-driver-monitoring-20260318
git add selfdrive/monitoring/helpers.py selfdrive/monitoring/test_monitoring.py docs/plans/2026-03-18-driver-monitoring-mild-relaxation-design.md docs/plans/2026-03-18-driver-monitoring-mild-relaxation.md
git commit -m "tune driver monitoring escalation"
```

Expected: commit created on `relax-driver-monitoring-20260318`
