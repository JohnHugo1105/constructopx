# ConstructOPX UX journey workflow

## Persona-driven journey and retest

Use this workflow for a requested persona walkthrough or a materially changed user-facing workflow that needs UX evidence. Give the explorer a persona and a goal, such as site staff recording progress or an owner checking a project summary. Start with the presentation audit and add DOM, accessibility, and responsive checks when they bear on the outcome.

1. Explore the rendered journey read-only. Use the connected browser or the repo’s Android/emulator workflow where applicable. Record the actual evidence tier and mark unavailable states or tools as blocked or not checked.
2. When fixes are requested, give the implementer the findings, affected states, and observable acceptance criteria. The implementer owns production-code edits and uses the existing design system.
3. Replay the recorded path for objective checks such as action reachability, focus return, and preservation of entered values. Return to the UX reviewer when sign-off depends on wording, hierarchy, discoverability, or whether recovery is understandable.
4. Send an existing cross-journey finding set to the UX auditor only when shared patterns or systemic causes need analysis. Spot-check the evidence where a claim needs it.

Hand off the finding ID, persona and goal, build or commit, environment, route, viewport or device, interaction state, steps, observed and expected behavior, user impact, and screenshot or DOM evidence. Name the exact retest criterion and its owner. Match the fixture and viewport on the fixed build where practical. Follow ConstructOPX’s existing test-data, environment, delegation, and write rules. Keep read-only UX exploration separate from functional mutations.
