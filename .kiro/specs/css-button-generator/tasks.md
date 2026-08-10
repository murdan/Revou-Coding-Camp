# Implementation Plan: CSS Button Generator

## Overview

Single-file HTML application with Vanilla JavaScript using a unidirectional data flow architecture. Implementation proceeds module-by-module: static structure and data first, then pure logic (contrast checker, code generator, randomizer), then state management, then rendering, then UI wiring. Property-based tests are placed close to the code they verify so errors are caught early.

## Tasks

- [-] 1. Set up project structure, constants, and data definitions
  - Create `index.html` with the two-panel Tailwind CSS layout (Control Panel + Preview Panel + Code Output)
  - Define `ButtonConfig` typedef, `DEFAULT_STATE` object, `BOX_SHADOW_VALUES` map, `HOVER_EFFECT_CSS` map, and `PRESETS` registry as constants in an inline `<script>` block
  - Include Tailwind CSS via CDN `<script>` tag and set up responsive breakpoint classes (lg: two-panel, default: single-column)
  - Add `<style id="hover-style">` empty tag and `#live-button` element with all required `aria-label` attributes
  - _Requirements: 1.1, 1.2, 1.4, 1.5, 1.6, 6.1_

- [ ] 2. Implement Contrast Checker module (pure functions)
  - [~] 2.1 Implement `hexToRelativeLuminance(hex)`, `getContrastRatio(hex1, hex2)`, and `meetsWCAGAA(hex1, hex2)` as pure functions
    - Use the WCAG relative luminance formula (linearize sRGB channels, sum with coefficients)
    - Handle 3-digit and 6-digit hex strings
    - _Requirements: 8.5, 8.6_

  - [ ]* 2.2 Write unit tests for Contrast Checker
    - `hexToRelativeLuminance("#ffffff")` === 1, `hexToRelativeLuminance("#000000")` === 0
    - `getContrastRatio("#ffffff", "#000000")` === 21
    - `meetsWCAGAA("#ffffff", "#3b82f6")` === true (contrast ≈ 4.66)
    - _Requirements: 8.5_

- [ ] 3. Implement Preset Registry and validate preset uniqueness
  - [~] 3.1 Define the `PRESETS` object with the five named presets: `primary`, `outline`, `ghost`, `pill`, `danger`
    - Each preset must be a complete `ButtonConfig` (all keys present, merging from `DEFAULT_STATE` where appropriate)
    - _Requirements: 6.1_

  - [ ]* 3.2 Write property test for preset uniqueness (Property 5)
    - **Property 5: All Preset Pairs Differ on At Least One Property**
    - **Validates: Requirements 6.1**
    - Enumerate all pairs of distinct preset names and assert at least one key differs

- [ ] 4. Implement Code Generator module (pure functions)
  - [~] 4.1 Implement `generateCSS(state)` — returns a formatted CSS string
    - Output `.custom-button { … }` block with `background-color`, `color`, `font-size`, `padding`, `border-radius`, `border`, `box-shadow`, `font-weight`, `cursor`, and `transition`
    - When `state.boxShadow === "custom"`, build shadow value from `shadowOffsetX/Y/Blur/Spread/Color`
    - When `state.hoverEffect !== "none"`, append `.custom-button:hover { … }` block using `HOVER_EFFECT_CSS` map
    - When `state.hoverEffect === "none"`, omit `:hover` block entirely
    - Use 2-space indentation; each declaration on its own line ending with `;`
    - _Requirements: 4.2, 4.4, 4.5, 4.6_

  - [~] 4.2 Implement `generateHTML(state)` — returns a formatted HTML string
    - Output `<button class="custom-button">…text…</button>` with proper HTML entity escaping for special characters in `state.text`
    - _Requirements: 4.1_

  - [ ]* 4.3 Write property test for HTML generation (Property 1)
    - **Property 1: HTML Generation Contains Button Text**
    - **Validates: Requirements 4.1**
    - Use `fc.string({ minLength: 1, maxLength: 100 })` as arbitrary; assert output contains text, `class="custom-button"`, and `<button`

  - [ ]* 4.4 Write property test for CSS completeness (Property 2)
    - **Property 2: CSS Generation Includes All Configured Properties**
    - **Validates: Requirements 4.2**
    - Use a partial `fc.record` of numeric and color fields; assert output contains `background-color`, `color`, `font-size`, `padding`, `border-radius`, `border`

  - [ ]* 4.5 Write property test for hover selector presence (Property 3)
    - **Property 3: Hover Selector Presence Matches Hover Effect Setting**
    - **Validates: Requirements 4.4, 4.5**
    - Use `fc.constantFrom(...hoverEffects)` ; assert `.custom-button:hover` presence iff `hoverEffect !== "none"`

  - [ ]* 4.6 Write property test for CSS formatting (Property 4)
    - **Property 4: Generated CSS Follows Formatting Rules**
    - **Validates: Requirements 4.6**
    - For each declaration line (lines containing `:`), assert it starts with 2 spaces and ends with `;`

- [~] 5. Checkpoint — Ensure all pure-function tests pass
  - Run `vitest --run` and confirm all Contrast Checker and Code Generator tests pass before proceeding.

- [ ] 6. Implement Randomizer module (pure functions)
  - [~] 6.1 Implement `randomizeColors()` — returns `{ textColor, bgColor }` with contrast ≥ 4.5:1
    - Loop up to 10 attempts generating random hex pairs; use `meetsWCAGAA` to validate
    - On exhaustion, return fallback `{ textColor: "#ffffff", bgColor: "#1d4ed8" }`
    - _Requirements: 8.5, 8.6_

  - [~] 6.2 Implement `randomizeNumericProps()` — returns all numeric properties within spec ranges
    - `fontSize` ∈ [12, 32], `borderRadius` ∈ [0, 50], `borderWidth` ∈ [0, 8], `paddingHorizontal` ∈ [8, 64], `paddingVertical` ∈ [4, 32], `fontWeight` ∈ {400, 500, 600, 700}
    - _Requirements: 8.2_

  - [~] 6.3 Implement `buildRandomState()` — assembles a complete, valid `ButtonConfig` using `randomizeColors()` and `randomizeNumericProps()`
    - _Requirements: 8.2, 8.5_

  - [ ]* 6.4 Write property test for randomize range validity (Property 7)
    - **Property 7: Randomized Values Are Within Specification Ranges**
    - **Validates: Requirements 8.2**
    - Call `buildRandomState()` 200 times (via `fc.integer()` seed); assert all numeric fields are within spec bounds

  - [ ]* 6.5 Write property test for randomize WCAG contrast (Property 8)
    - **Property 8: Randomized Colors Meet WCAG AA Contrast**
    - **Validates: Requirements 8.5, 8.6**
    - Call `randomizeColors()` 200 times; assert `getContrastRatio(textColor, bgColor) >= 4.5`

- [ ] 7. Implement State Manager module
  - [~] 7.1 Implement `updateState(key, value)`, `getState()`, `resetState()`, and `applyPreset(presetName)`
    - `updateState` must reject unknown keys (console.warn) and validate numeric values with `Number.isFinite()`
    - `getState` returns a shallow copy (readonly snapshot)
    - `resetState` deep-copies `DEFAULT_STATE` back into the live state object
    - `applyPreset` merges the named preset over `DEFAULT_STATE` then triggers render
    - Each mutating function calls `renderLiveButton(getState())` and `renderCodeOutput()` after updating state
    - _Requirements: 7.2, 7.5, 6.2_

  - [ ]* 7.2 Write unit tests for State Manager
    - `updateState` with valid key changes the value returned by `getState()`
    - `updateState` with unknown key leaves state unchanged
    - `applyPreset("primary")` produces state matching `PRESETS.primary` merged with `DEFAULT_STATE`
    - `applyPreset` then `resetState` produces `DEFAULT_STATE`
    - _Requirements: 7.2, 6.2_

  - [ ]* 7.3 Write property test for reset idempotence (Property 6)
    - **Property 6: Reset Produces Default State**
    - **Validates: Requirements 7.2, 7.5**
    - Apply an array of random `updateState` calls (keys from `fc.constantFrom` of valid keys), then call `resetState()`; assert `deepEqual(getState(), DEFAULT_STATE)`

- [ ] 8. Implement Live Button Renderer
  - [~] 8.1 Implement `renderLiveButton(state)` — applies state as inline styles to `#live-button`
    - Validate each hex color against `/^#[0-9a-fA-F]{3,6}$/` and each numeric value with `Number.isFinite()` before applying; skip invalid values silently
    - Write the `:hover` CSS rule into `<style id="hover-style">` using `HOVER_EFFECT_CSS` map with `transition: all 200ms ease`
    - Clear `<style id="hover-style">` content when `hoverEffect === "none"`
    - _Requirements: 3.1, 3.4, 3.5_

  - [ ]* 8.2 Write unit tests for Live Button Renderer
    - `renderLiveButton` with an invalid hex color does not update that specific style property (retains previous value)
    - `renderLiveButton` with `hoverEffect="scale-up"` writes `transform: scale(1.05)` into `<style id="hover-style">`
    - `renderLiveButton` with `hoverEffect="none"` results in empty `<style id="hover-style">` content
    - _Requirements: 3.4, 3.5_

- [ ] 9. Implement Code Output Renderer and wire to DOM
  - [~] 9.1 Implement `renderCodeOutput()` — writes generated strings to `#css-output` and `#html-output` DOM elements
    - Call `generateCSS(getState())` and `generateHTML(getState())` and set `textContent` of the output `<pre>` elements
    - Enable/disable Copy CSS and Copy HTML buttons based on whether output is non-empty (R5.7)
    - _Requirements: 4.3, 5.7_

  - [ ]* 9.2 Write unit tests for Code Output Renderer
    - After calling `renderCodeOutput()`, `#css-output` contains the string returned by `generateCSS(state)`
    - Copy buttons are disabled when state produces empty output, enabled otherwise
    - _Requirements: 4.3, 5.7_

- [ ] 10. Implement Clipboard Manager
  - [~] 10.1 Implement `copyToClipboard(text, buttonEl, originalLabel)`
    - Check `navigator.clipboard` availability before calling; if unavailable, show `<div role="alert">` with the "not supported" message and return without changing button label
    - Use `try/catch` around `navigator.clipboard.writeText()`; on catch, show `<div role="alert">` with the "copy failed" message and do NOT change button label
    - On success, change button label to "Copied!" and restore `originalLabel` after 2000ms via `setTimeout`
    - _Requirements: 5.3, 5.4, 5.5, 5.6_

  - [ ]* 10.2 Write unit tests for Clipboard Manager
    - When `navigator.clipboard` is undefined, the error alert is shown and button label does not change
    - When `clipboard.writeText()` throws, the error alert is shown and label stays as `originalLabel`
    - On success, label becomes "Copied!" immediately, then reverts to `originalLabel` after 2000ms
    - _Requirements: 5.3, 5.4, 5.5, 5.6_

- [~] 11. Checkpoint — Ensure all module tests pass
  - Run `vitest --run` and confirm all State Manager, Renderer, Code Output, and Clipboard Manager tests pass.

- [ ] 12. Implement UI Event Binder and complete wiring
  - [~] 12.1 Implement `bindEvents()` — bind all Control Panel inputs to `updateState` calls
    - Text input → `updateState("text", value.trim().slice(0, 100))`
    - All sliders → respective keys with `Number(value)` parsing
    - Font weight and box shadow dropdowns → respective keys; show/hide custom shadow inputs when `boxShadow === "custom"`
    - Hover effect dropdown → `updateState("hoverEffect", value)`
    - Color pickers for `textColor`, `backgroundColor`, `borderColor`
    - _Requirements: 2.1–2.14_

  - [~] 12.2 Bind action buttons and preset cards
    - "Reset" button → `resetState()`
    - "Randomize" button → `randomizeState()` (implement as: calls `buildRandomState()`, copies result into state, triggers render)
    - "Copy CSS" button → `copyToClipboard(cssText, btn, "Copy CSS")`
    - "Copy HTML" button → `copyToClipboard(htmlText, btn, "Copy HTML")`
    - Each of the 5 preset cards → `applyPreset(presetName)`; update all Control Panel input values to reflect the new state
    - _Requirements: 5.1, 5.2, 6.2, 7.1, 8.1_

  - [~] 12.3 Sync all Control Panel input display values after any state change
    - After every `updateState`, `resetState`, `applyPreset`, or `randomizeState` call, write current `getState()` values back to all input elements so the UI reflects state (slider value labels, color picker values, dropdown selections, text input value)
    - _Requirements: 2.13, 6.2, 7.2, 8.3_

  - [~] 12.4 Call `bindEvents()` inside a `DOMContentLoaded` listener; call `renderLiveButton(DEFAULT_STATE)` and `renderCodeOutput()` on load to show initial state
    - _Requirements: 3.6, 4.3_

- [ ] 13. Implement accessibility and visual design requirements
  - [~] 13.1 Ensure all Control Panel inputs have associated `<label>` elements connected via `for`/`id`, or `aria-label` attributes where a visible label is impractical
    - Verify tab order follows top-to-bottom, left-to-right visual order
    - _Requirements: 9.3, 9.4_

  - [~] 13.2 Add visible focus ring styles to all interactive elements using Tailwind `focus-visible:ring` utilities with contrast ≥ 3:1 against the surrounding background
    - _Requirements: 9.6_

  - [~] 13.3 Apply responsive layout classes so the layout switches between two-panel (lg:flex-row) and single-column (flex-col) at the 1024px breakpoint without a page reload; verify all inputs remain reachable without horizontal scroll at 768px–1023px
    - _Requirements: 1.1, 1.2, 1.3, 9.5, 9.7_

- [~] 14. Final checkpoint — Ensure all tests pass
  - Run `vitest --run` and confirm the full test suite is green.
  - Manually verify: tab navigation reaches all controls, Live_Button updates within 100ms, preset cards apply correctly, Reset restores default, Randomize produces WCAG-compliant colors.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Property tests use `fast-check` inside Vitest; run with `vitest --run` (no watch mode)
- Because there is no build step, all JS is in `<script>` tags in `index.html`; tests import functions that must be exported from a separate `.js` module or tested via a thin wrapper
- Each task references specific requirements for traceability
- Checkpoints at tasks 5, 11, and 14 ensure incremental validation before wiring dependent modules

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1"] },
    { "id": 1, "tasks": ["2.1", "3.1"] },
    { "id": 2, "tasks": ["2.2", "3.2", "4.1", "4.2"] },
    { "id": 3, "tasks": ["4.3", "4.4", "4.5", "4.6", "6.1"] },
    { "id": 4, "tasks": ["6.2", "6.3"] },
    { "id": 5, "tasks": ["6.4", "6.5", "7.1"] },
    { "id": 6, "tasks": ["7.2", "7.3", "8.1"] },
    { "id": 7, "tasks": ["8.2", "9.1"] },
    { "id": 8, "tasks": ["9.2", "10.1"] },
    { "id": 9, "tasks": ["10.2", "12.1"] },
    { "id": 10, "tasks": ["12.2", "12.3"] },
    { "id": 11, "tasks": ["12.4"] },
    { "id": 12, "tasks": ["13.1", "13.2", "13.3"] }
  ]
}
```
