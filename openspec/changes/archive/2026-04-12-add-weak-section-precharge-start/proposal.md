## Why

Some irrigation lines cannot open reliably unless the pump first builds pressure against closed valves. The current behavior starts sections directly, which can fail for weak lines during both initial startup and section-to-section transitions.

## What Changes

- Add support to mark irrigation sections as "weak" and require staged startup for those sections.
- Introduce a global pre-charge duration `X` seconds used before opening any weak section.
- Require pre-charge for weak sections in all entry paths:
  - first section in a run,
  - manual switching,
  - transition from another running section.
- Define mid-precharge target changes as normal section switching behavior:
  - cancel prior target,
  - apply rules for the newly selected target.
- Keep notification behavior unchanged (no new pre-charge notifications).

## Capabilities

### New Capabilities

- `irrigation-weak-section-precharge`: Defines staged startup semantics for weak irrigation sections using a global pre-charge window.

### Modified Capabilities

- `irrigation-relay-control`: Expands section activation rules to include mandatory pre-charge before any weak section activation.

## Impact

- Affected runtime logic in ESPHome irrigation control state handling.
- Affected device configuration surface for section metadata (weak flag) and global pre-charge duration.
- No new external integrations or notification channels required.
