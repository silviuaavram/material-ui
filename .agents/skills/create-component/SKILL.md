---
name: create-component
description: Design, implement, and validate a new Material UI or Lab component in this repository. Use for new component work from API and accessibility design through tests and docs.
---

# Create a component

1. Read the repository [AGENTS.md](../../../AGENTS.md) and any instructions for the target package. Establish the behavior, supported use cases, target package, and public API expected by the request. Identify the default interaction and any choices that affect compatibility or accessibility.
2. Inspect the closest component by **interaction** and the closest by **composition**. Trace their actual DOM, props, state, types, tests, and public exports. Read [architecture](references/architecture.md) when choosing state ownership, slots, theming, or package wiring. Skim the headings in [lessons learned](references/lessons-learned.md) and read the cases that match the planned integration.
3. Decide which element owns each interaction, ref, accessible relationship, and form responsibility. Set the keyboard and focus contract from the actual interaction pattern. Decide which layer owns mutable state, what each public customization changes, and how overlapping props and handlers are resolved. For a composed component, map behavior primitives to host elements before deciding which parts become public slots.
4. Wire one complete default use case through the selected primitives. Reuse repo hooks and merge utilities where their behavior fits. Add only the variants, slots, and overrides needed by the API, then connect them to their runtime, type, and theme entry points.
5. Use [testing](references/testing.md) to check behavior owned by the component and the integration points it introduces. Write demos that teach distinct use cases. Run the scoped checks that exercise the change, plus applicable generators and repository checks from AGENTS.md. Inspect generated and rendered output when relevant.
6. Report the public contract, consequential design choices, and verification. Capture a new transferable lesson in the handoff if the work exposed an integration trap. Update the skill only when requested.
