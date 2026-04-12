## Context

The irrigation controller currently enforces single active section at device level and derives pump state from section activity. This model is robust for normal section operation, but some hydraulically weak lines fail to start unless the pump first builds pressure with all section relays closed.

The requested behavior introduces staged startup for weak sections with a global pre-charge window. The design must preserve existing safety and operator expectations:
- one active section at a time,
- deterministic section switching,
- no new notification noise,
- compatibility with both scheduled and manual control paths.

## Goals / Non-Goals

**Goals:**
- Add a per-section weak marker and a global pre-charge duration.
- Apply pre-charge before activating any weak section, regardless of whether it is first in a run or selected during section handoff.
- Treat target changes during pre-charge as normal switching requests (latest target wins).
- Keep existing schedule definitions usable without requiring schedule-specific weak logic.

**Non-Goals:**
- Add new notification events for pre-charge steps.
- Introduce pressure sensor feedback loops or closed-loop pressure control.
- Redesign existing daily sequence timing in Home Assistant.

## Decisions

1. Device-level weak startup policy
- Decision: Enforce weak-section staged startup in ESPHome control logic instead of Home Assistant scripts.
- Rationale: Device-level behavior stays consistent for all trigger paths (manual UI, HA scripts, service calls).
- Alternative considered: Add pre-charge steps in each HA script branch. Rejected because it is fragile and easy to bypass.

2. Global pre-charge duration
- Decision: Use one global duration `X` for all weak sections.
- Rationale: Matches operational preference and keeps configuration simple.
- Alternative considered: Per-section pre-charge durations. Rejected as unnecessary complexity.

3. Mandatory pre-charge on every transition to weak target
- Decision: Run pre-charge before opening a weak section even when switching from an already running non-weak section.
- Rationale: Existing running pressure may still be insufficient for weak lines.
- Alternative considered: Pre-charge only when starting from idle. Rejected because it does not address weak-line transition failures.

4. Pre-charge target override semantics
- Decision: If user/scheduler requests another section during pre-charge, cancel prior target and process only the latest target.
- Rationale: Aligns with normal section-switch expectations and avoids stale delayed opens.
- Alternative considered: Queue requests. Rejected due to harder predictability and longer response.

5. Preserve no-notification policy
- Decision: Do not emit dedicated notifications for weak pre-charge events.
- Rationale: Requested explicitly and avoids alert fatigue.
- Alternative considered: informational notifications for diagnostics. Rejected as out of scope.

## Risks / Trade-offs

- [Frequent switching to weak sections increases dead-time with all valves closed] -> Mitigation: keep `X` configurable and tune conservatively.
- [Pump running with all sections closed relies on mechanical pressure cutoff behavior] -> Mitigation: keep window bounded and preserve explicit all-off controls.
- [Race conditions between pre-charge timer and new user commands] -> Mitigation: use restart/cancel semantics so latest command wins.
- [Operational confusion about why weak section starts later] -> Mitigation: document weak behavior in irrigation operational notes.

## Migration Plan

1. Add weak-section pre-charge spec deltas and approve change.
2. Implement configuration knobs and staged startup state machine in ESPHome irrigation logic.
3. Validate scenarios:
   - weak first section startup,
   - non-weak to weak transition,
   - weak pre-charge interrupted by new target selection,
   - all-off during pre-charge.
4. Tune global `X` based on real startup reliability.
5. Rollback plan: disable weak flags or set `X` to zero-equivalent behavior and return to direct switching.

## Open Questions

- Should pre-charge elapsed time contribute to runtime watchdog accounting, or remain excluded as a transition window?
