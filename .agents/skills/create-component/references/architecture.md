# Component architecture

## Pick a precedent for each concern

- Use [TextField](../../../../packages/mui-material/src/TextField/TextField.js) and [FormControl](../../../../packages/mui-material/src/FormControl/FormControl.js) for label, helper text, and variant composition. Read [InputBase](../../../../packages/mui-material/src/InputBase/InputBase.js) when routing native input props or relying on filled and focused context.
- Use [ToggleButton](../../../../packages/mui-material/src/ToggleButton/ToggleButton.js) and [ButtonBase](../../../../packages/mui-material/src/ButtonBase/ButtonBase.js) for pressable controls, and [SwitchBase](../../../../packages/mui-material/src/internal/SwitchBase.js) for a native checkbox with custom visuals. Choose semantics and the focus model from the interaction. A selectable control, a navigation item, and a stepper button can all look like buttons while needing different semantics.
- For a behavior primitive composed with Material UI, check the installed primitive's public types, render API, and event cancellation contract first. The [NumberField case study](lessons-learned.md) shows where an adapter was needed.

## Decide state ownership and accessibility together

Identify the source of truth for each state. Use [useControlled](../../../../packages/mui-utils/src/useControlled/useControlled.ts) for a locally owned controlled/uncontrolled contract, or delegate to the primitive that already owns it. Define the meaning of empty and omitted values, callback payload and timing, and what happens after prop or context changes. Keep display text separate from the canonical value if editing or formatting requires it. Populate styling state from the resolved behavior and theme defaults, with the owning layer deciding conflicts.

Use the native element whose semantics fit. For composite widgets, decide the accessible name, role, tab stops, key actions, focus target after interactions, and disabled or read-only behavior. For fields, trace ID relationships and form ownership. Confirm navigation and activation with the mounted control as well as custom rendering paths promised by the API.

## Define the slot boundary

Map each public slot to the element or component it customizes. Distinguish the styled wrapper from the native element if both exist. Base slot types on the actual leaf or Material UI component, retain the variant's prop shape where it differs, and expose `component`, `render`, and replacement slots only as the API intends. Give slot callbacks the appropriate resolved state, including per-instance details when a slot appears more than once.

For a new API, give consumers one path to configure each native concern. Check inherited prop bags for routes that can overwrite the primitive's value, handlers, or ref at the last spread. Exclude unsupported paths from both object and callback slot props, and discard those props at runtime for JavaScript callers. Preserve unrelated nested slots such as an outlined input's notch.

Read [Material useSlot](../../../../packages/mui-material/src/utils/useSlot.ts) and [mergeSlotProps](../../../../packages/mui-utils/src/mergeSlotProps/mergeSlotProps.ts) before composing props. For ordinary values, `additionalProps` goes before forwarded root props and resolved slot props. Use it for defaults that consumers may override. Use final JSX assignment or a coordinated resolution when the component owns an invariant. `getSlotProps` receives the selected external handlers and returns internal props with handlers composed. Use it when there are internal handlers to preserve at that boundary. Check the order and cancellation semantics of each merge helper and use [useForkRef](../../../../packages/mui-utils/src/useForkRef/useForkRef.ts) when consumers and internals need the same DOM ref.

## Connect the public surface

Follow the target package's conventions for entry points, forwarded refs, client boundaries, classes, theme defaults, and augmentation. Expose a direct `styleOverrides` key when a styled slot consumes it. A public utility class can also identify a descendant or a state for use in a selector. Keep the theme types and generated rule table aligned with the supported keys. See [theme props](../../../../packages/mui-material/src/styles/props.ts), [override keys](../../../../packages/mui-material/src/styles/overrides.ts), and [theme components](../../../../packages/mui-material/src/styles/components.ts).

For built-in text, add the relevant props to `Localization`, document the English defaults in `enUS`, and supply the entries in the supported locale bundles. Include strings supplied by a primitive, such as NumberField's button labels and the textbox's `aria-roledescription`. Use the public slot props when those already reach the element. Render with a non-English locale passed to `createTheme` and check the translated names and role descriptions on the rendered controls. See the [locale types](../../../../packages/mui-material/src/locale/utils/LocaleTextApi.ts), [English defaults](../../../../packages/mui-material/src/locale/enUS.ts), and [theme locale tests](../../../../packages/mui-material/src/locale/locale.test.tsx).

Treat public dependency types and runtime re-exports as part of the contract. Verify the supported dependency version, public package imports, and generated API descriptions. Keep private adapter and layout types out of the public entry point unless consumers need them.
