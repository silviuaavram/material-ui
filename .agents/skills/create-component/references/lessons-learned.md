# Lessons from NumberField

These cases follow `feat/number-field` through `c893dcc870`. The implementation, tests, and guide are pinned to that revision so the examples can be read from a checkout that lacks NumberField. [Component source](https://github.com/silviuaavram/material-ui/tree/c893dcc870557a85f631803fa009c066f0e7ca00/packages/mui-material/src/NumberField) · [Integration tests](https://github.com/silviuaavram/material-ui/blob/c893dcc870557a85f631803fa009c066f0e7ca00/packages/mui-material/src/NumberField/NumberField.test.tsx) · [Component guide](https://github.com/silviuaavram/material-ui/blob/c893dcc870557a85f631803fa009c066f0e7ca00/docs/data/material/components/number-field/number-field.md)

## Share a host and preserve its customization contract

The Material input root already contains the textbox and adornments. NumberField applies Base UI Group behavior to that root through the primitive's `render` API. This keeps one wrapper with `role="group"`, Material styles, and click-to-focus. The `input` slot customizes that wrapper and `htmlInput` customizes the textbox.

Trace three ways to select the host. NumberField uses the variant's themed root by default. A local `slotProps.input.render` can override that theme default while preserving the styled layer. An explicit root slot takes precedence over the render callback. A full `slots.input` replacement chooses its own themed root and receives the Group props. Verify this with a replacement of a different variant, distinct themed roots, and a render callback that would be observable if it ran.

Document the context a full replacement must supply. For example, `render` on the field root retains `FormControl`. A `slots.root` replacement must forward its props and ref to a `FormControl` to preserve label shrink on focus and error state on the label and helper. Clear unsupported host-changing props at the runtime boundary if an inherited component would still honor them.

## Give native input props one route

NumberField initially composed keyboard handlers and refs from several nested prop bags. InputBase applies its nested input props late enough to overwrite the primitive's handlers. NumberField now exposes `slotProps.htmlInput` for native attributes, keyboard handlers, and native slot refs. The Material `input` slot retains wrapper props, adornments, and variant-specific slots. The field owns the numeric value and formatted single-line input type.

Strip competing `inputProps` and nested `slotProps.input` from the wrapper before passing it to InputBase. Compose the supported native handlers and refs with the primitive and filter nullish handlers so an explicit `undefined` preserves keyboard stepping. Check callback-returned slot props as well as object literals in the type spec. Give handlers the actual element and event type so consumers can call the primitive's public cancellation method. See [input prop resolution](https://github.com/silviuaavram/material-ui/blob/c893dcc870557a85f631803fa009c066f0e7ca00/packages/mui-material/src/NumberField/utils/resolveInputProps.ts) and [consumer type checks](https://github.com/silviuaavram/material-ui/blob/c893dcc870557a85f631803fa009c066f0e7ca00/packages/mui-material/src/NumberField/NumberField.spec.tsx).

## Resolve shared state once and optional slots when rendered

NumberField reads disabled, required, and read-only from the primitive after it has applied field context. Each button inherits the field size by default. An explicit button size feeds both that button's props and its styling owner state, so changing one stepper leaves the other at the field size.

Resolve a callback once per render when multiple elements need its result, such as an input ID or start adornment. Read merged props, including `render`, from `useSlot`. Keep position-specific callbacks with each rendered instance. Moving the scrub-area hooks into a child mounted only when enabled also keeps callbacks for the handle, cursor, and adornment from running for an absent feature.

## Carry IDs and content presence through the whole field

A native input slot ID takes precedence over the field ID. Supply it to the primitive input so the textbox, label's `htmlFor`, and both steppers' `aria-controls` agree. Likewise, the helper's final ID must match the input's `aria-describedby`. Use the same presence check to render optional content and to create its association. NumberField treats `false`, nullish values, and an empty string as absent and renders numeric zero. For the filled variant, derive the hidden-label layout from label presence and honor an explicit `hiddenLabel` override.

Display and submission can also have different owners. NumberField associates both the visible input and the primitive's submission input with the external form. Verify native validation and Enter submission on the visible control and a single raw value in `FormData`. When a primitive limits native behavior, document the supported alternative, as the guide does for controlled form reset.

## Preserve state across the first render and layout changes

FormControl inspects its immediate children to initialize label shrink before effects run. NumberField's adapter hides the actual input, so it supplies a marker with the formatted initial text and the same start-adornment presence as the rendered input. Check the initial HTML and hydration for an empty value, zero, and a falsy start adornment with content at the end. An empty fragment counts as an adornment to FormControl.

When built-in controls move or disappear, keep consumer adornments in stable positions in the React tree. Type into uncontrolled prefix and suffix inputs, move or remove the controls, and check that both edits and the main input survive. A static placement assertion would miss a remount.

## Give keyboard activation one owner

Base UI and ButtonBase can both synthesize a click on a non-native stepper. Run the composed handler first, then use each layer's cancellation contract. NumberField's adapter signals `defaultMuiPrevented` to suppress Material's fallback and handles Space scrolling separately. The shared ButtonBase change was split out. At this revision the Material-rendered non-native activation and cancellation cases still depend on that fix. See the [stepper adapter](https://github.com/silviuaavram/material-ui/blob/c893dcc870557a85f631803fa009c066f0e7ca00/packages/mui-material/src/NumberField/NumberFieldButtonSlot.tsx) and [ButtonBase follow-up](https://github.com/mui/material-ui/pull/49850).

Check one activation per Enter or Space gesture, and zero when the consumer cancels. Exercise the native button, link, and non-native replacement when supported because the keyboard handlers differ. NumberField's default tab stop is the textbox, which offers keyboard stepping. Its named buttons also remain available to assistive technology.

## Distinguish a theme override key from a class selector

Named styled slots such as the label and helper text consume direct theme overrides. The variant supplies the styled native input, and scrubbing is a state on the field root. NumberField exposes classes for both. In `styleOverrides.root`, use `& .MuiNumberField-htmlInput` for the descendant and `&.MuiNumberField-scrubbing` for the root state. Its override types exclude those two direct keys. Verify the documented selectors with computed styles and the supported keys with a consumer [theme augmentation fixture](https://github.com/silviuaavram/material-ui/blob/c893dcc870557a85f631803fa009c066f0e7ca00/test/moduleAugmentation/material/numberFieldCustomProps.spec.tsx).

The primitive also supplies data attributes. If they are public, re-export the supported namespaces and document the slot that receives each one. NumberField maps Group to `input` and Input to `htmlInput`.
