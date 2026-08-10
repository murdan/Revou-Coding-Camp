# Design Document: CSS Button Generator

## Overview

CSS Button Generator adalah single-page web application yang berjalan sepenuhnya di browser tanpa backend. Aplikasi memungkinkan developer dan designer mengkustomisasi button secara visual melalui Control Panel interaktif, melihat hasilnya secara real-time di Preview Panel, lalu menyalin generated CSS dan HTML ke clipboard.

Seluruh state aplikasi dikelola di memori menggunakan satu JavaScript object (`buttonState`). Setiap perubahan input memicu pipeline satu arah: **input event → state update → render Live_Button → generate code output**. Tidak ada network call, tidak ada penyimpanan persisten, dan tidak ada framework JavaScript — hanya HTML, Tailwind CSS (via CDN), dan Vanilla JavaScript.

### Tujuan Desain

- **Kesederhanaan**: Satu file HTML (atau minimal file structure) tanpa build step
- **Reaktivitas**: Semua pembaruan UI kurang dari 100ms setelah input berubah
- **Keterbacaan kode**: Generated CSS dan HTML harus siap digunakan langsung
- **Aksesibilitas**: Semua kontrol dapat diakses via keyboard dan screen reader

---

## Architecture

Aplikasi menggunakan arsitektur **Unidirectional Data Flow** sederhana tanpa framework:

```
┌─────────────────────────────────────────────────────────────┐
│                        User Input                           │
│          (sliders, color pickers, dropdowns, buttons)       │
└────────────────────────────┬────────────────────────────────┘
                             │ DOM events (input, change, click)
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    Event Handler Layer                       │
│         Reads input value → calls updateState(key, value)   │
└────────────────────────────┬────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                     State Object                            │
│              buttonState: ButtonConfig (in-memory)          │
└────────────┬──────────────────────────────┬─────────────────┘
             │                              │
             ▼                              ▼
┌────────────────────────┐    ┌─────────────────────────────┐
│   renderLiveButton()   │    │   generateCodeOutput()       │
│   Applies inline CSS   │    │   Builds CSS + HTML strings  │
│   to Live_Button DOM   │    │   Writes to <pre> elements   │
└────────────────────────┘    └─────────────────────────────┘
```

### Keputusan Arsitektur

| Keputusan | Pilihan | Alasan |
|---|---|---|
| File structure | Single HTML file | Sesuai requirement; tidak ada build step |
| CSS framework | Tailwind CSS via CDN | Sesuai requirement; tidak perlu install |
| State management | Plain JS object | Cukup untuk skala aplikasi ini |
| Rendering | Inline style injection | Perubahan langsung tanpa re-render DOM |
| Code generation | Template string | Sederhana, deterministik, mudah ditest |
| Hover effect | CSS class + `<style>` tag dinamis | Memungkinkan `:hover` pseudo-class |

---

## Components and Interfaces

### 1. State Manager

Modul inti yang menyimpan state dan mendistribusikan pembaruan.

```javascript
// Fungsi publik
function updateState(key, value)   // Update satu properti, trigger render
function resetState()              // Reset ke DEFAULT_STATE
function randomizeState()          // Generate nilai acak, trigger render
function applyPreset(presetName)   // Terapkan preset ke state, trigger render
function getState()                // Return salinan state saat ini (readonly)
```

### 2. Renderer — Live Button

Mengaplikasikan state ke elemen `#live-button` via inline style, dan memperbarui `<style id="hover-style">` untuk efek hover.

```javascript
function renderLiveButton(state)
// Menulis semua properti CSS relevan sebagai inline style
// Menulis CSS :hover ke tag <style> dinamis
```

### 3. Code Generator

Menghasilkan string CSS dan HTML yang siap disalin dari state saat ini.

```javascript
function generateCSS(state)   // returns string CSS terformat
function generateHTML(state)  // returns string HTML terformat
function renderCodeOutput()   // menulis ke DOM #css-output dan #html-output
```

### 4. Clipboard Manager

Menangani operasi salin ke clipboard dengan feedback visual.

```javascript
async function copyToClipboard(text, buttonEl, originalLabel)
// Menggunakan navigator.clipboard.writeText()
// Mengubah label tombol ke "Copied!" selama 2000ms
// Menangani error jika API tidak tersedia atau gagal
```

### 5. Contrast Checker

Fungsi murni (pure function) untuk menghitung rasio kontras WCAG.

```javascript
function hexToRelativeLuminance(hex)   // returns number [0, 1]
function getContrastRatio(hex1, hex2)  // returns number (e.g. 4.5)
function meetsWCAGAA(hex1, hex2)       // returns boolean
```

### 6. Randomizer

Menghasilkan nilai acak yang valid untuk semua properti button.

```javascript
function randomizeColors()       // returns { textColor, bgColor } dengan kontras ≥ 4.5:1
function randomizeNumericProps() // returns semua nilai numerik dalam rentang spec
function buildRandomState()      // returns ButtonConfig lengkap yang valid
```

### 7. Preset Registry

Data statis berisi 5 preset konfigurasi.

```javascript
const PRESETS = {
  primary: ButtonConfig,
  outline: ButtonConfig,
  ghost:   ButtonConfig,
  pill:    ButtonConfig,
  danger:  ButtonConfig,
}
```

### 8. UI Event Binder

Menghubungkan semua elemen DOM ke handler yang sesuai. Dipanggil sekali saat `DOMContentLoaded`.

```javascript
function bindEvents()
// Bind semua slider, color picker, dropdown, tombol ke updateState / aksi
// Bind tombol Copy CSS, Copy HTML, Reset, Randomize, dan preset cards
```

---

## Data Models

### ButtonConfig — State Object

```javascript
/**
 * @typedef {Object} ButtonConfig
 * Representasi lengkap state konfigurasi button.
 */
const ButtonConfig = {
  // Teks
  text:              String,   // max 100 karakter; default: "Click Me"

  // Tipografi
  fontSize:          Number,   // px; range: 10–48; default: 16
  fontWeight:        Number,   // enum: 300|400|500|600|700|800; default: 400

  // Warna
  textColor:         String,   // hex color; default: "#ffffff"
  backgroundColor:   String,   // hex color; default: "#3b82f6"
  borderColor:       String,   // hex color; default: "#3b82f6"

  // Border
  borderWidth:       Number,   // px; range: 0–10; default: 0
  borderRadius:      Number,   // px; range: 0–50; default: 8

  // Spacing
  paddingHorizontal: Number,   // px; range: 0–80; default: 24
  paddingVertical:   Number,   // px; range: 0–60; default: 12

  // Shadow
  boxShadow:         String,   // enum: "none"|"small"|"medium"|"large"|"custom"
  shadowOffsetX:     Number,   // px; hanya relevan jika boxShadow === "custom"
  shadowOffsetY:     Number,   // px; hanya relevan jika boxShadow === "custom"
  shadowBlur:        Number,   // px; hanya relevan jika boxShadow === "custom"
  shadowSpread:      Number,   // px; hanya relevan jika boxShadow === "custom"
  shadowColor:       String,   // hex color; hanya relevan jika boxShadow === "custom"

  // Hover
  hoverEffect:       String,   // enum: "none"|"darken"|"lighten"|"scale-up"|"scale-down"|"lift"
}
```

### DEFAULT_STATE

```javascript
const DEFAULT_STATE = {
  text:              "Click Me",
  fontSize:          16,
  fontWeight:        400,
  textColor:         "#ffffff",
  backgroundColor:   "#3b82f6",
  borderColor:       "#3b82f6",
  borderWidth:       0,
  borderRadius:      8,
  paddingHorizontal: 24,
  paddingVertical:   12,
  boxShadow:         "none",
  shadowOffsetX:     0,
  shadowOffsetY:     4,
  shadowBlur:        6,
  shadowSpread:      0,
  shadowColor:       "#00000040",
  hoverEffect:       "none",
}
```

### BoxShadow Value Map

```javascript
const BOX_SHADOW_VALUES = {
  none:   "none",
  small:  "0 1px 2px 0 rgba(0,0,0,0.05)",
  medium: "0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -2px rgba(0,0,0,0.1)",
  large:  "0 20px 25px -5px rgba(0,0,0,0.1), 0 8px 10px -6px rgba(0,0,0,0.1)",
  custom: null,   // dibangun dari shadowOffset/Blur/Spread/Color
}
```

### HoverEffect CSS Map

```javascript
const HOVER_EFFECT_CSS = {
  none:       null,
  darken:     "filter: brightness(0.85);",
  lighten:    "filter: brightness(1.15);",
  "scale-up": "transform: scale(1.05);",
  "scale-down": "transform: scale(0.95);",
  lift:       "<shadow satu level di atas shadow aktif>",
}
// Semua efek menggunakan transition: all 200ms ease
```

### Preset Definitions

```javascript
const PRESETS = {
  primary: { ...DEFAULT_STATE },   // solid blue, rounded-lg
  outline: {
    textColor: "#3b82f6", backgroundColor: "#ffffff",
    borderColor: "#3b82f6", borderWidth: 2, hoverEffect: "darken"
  },
  ghost: {
    textColor: "#374151", backgroundColor: "transparent",
    borderColor: "transparent", borderWidth: 0, hoverEffect: "lighten"
  },
  pill: {
    ...DEFAULT_STATE, borderRadius: 50, paddingHorizontal: 32
  },
  danger: {
    backgroundColor: "#ef4444", borderColor: "#ef4444",
    textColor: "#ffffff", hoverEffect: "darken"
  },
}
// Catatan: setiap preset berbeda dari preset lainnya pada ≥1 properti (R6.1)
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

Bagian ini mendefinisikan properti formal untuk fungsi-fungsi murni (pure functions) dalam aplikasi, yaitu: code generator, contrast checker, dan randomizer. Properti-properti ini cocok untuk property-based testing karena fungsi-fungsi tersebut bersifat deterministik, tidak memiliki side effect, dan perilakunya harus berlaku untuk semua input yang valid.

---

### Acceptance Criteria Testing Prework

**Requirement 4: Generated Code Output**

4.1 WHEN nilai input berubah THEN Code_Output SHALL menampilkan HTML dengan `<button class="custom-button">` dan teks yang sesuai
  Thoughts: Ini bukan tentang satu contoh spesifik — berlaku untuk *semua* nilai teks yang mungkin. Kita bisa generate string teks acak, membuat ButtonConfig, memanggil generateHTML(), dan memverifikasi hasilnya mengandung teks tersebut.
  Classification: PROPERTY
  Test Strategy: Untuk semua string teks yang valid (1–100 karakter), generateHTML(state) harus mengandung teks tersebut di dalam elemen button.

4.2 WHEN nilai input berubah THEN Code_Output SHALL menampilkan CSS dengan semua properti yang dikonfigurasi
  Thoughts: Berlaku untuk semua konfigurasi — setiap properti yang di-set harus muncul di output. Kita bisa generate ButtonConfig acak dan memverifikasi semua properti muncul di CSS.
  Classification: PROPERTY
  Test Strategy: Untuk semua ButtonConfig yang valid, generateCSS(state) harus mengandung semua properti yang dikonfigurasi (background-color, color, font-size, padding, border-radius, border).

4.4 + 4.5 WHEN hover effect ≠ None THEN CSS mengandung selector .custom-button:hover; WHEN hover effect = None THEN tidak ada selector :hover
  Thoughts: Ini adalah kondisi if-and-only-if yang berlaku untuk semua konfigurasi. Bisa dikombinasikan menjadi satu property: keberadaan :hover selector harus tepat sama dengan hoverEffect ≠ "none".
  Classification: PROPERTY
  Test Strategy: Untuk semua ButtonConfig, keberadaan `:hover` block dalam generateCSS() harus sama dengan (state.hoverEffect !== "none").

4.6 CSS menggunakan indentasi 2 spasi dan setiap deklarasi pada baris terpisah
  Thoughts: Ini adalah aturan format yang berlaku untuk semua output. Kita bisa generate berbagai konfigurasi dan memverifikasi format string hasilnya.
  Classification: PROPERTY
  Test Strategy: Untuk semua ButtonConfig yang valid, setiap baris dalam blok CSS harus diawali dengan 2 spasi (untuk properti) atau tanpa indent (untuk selector), dan setiap deklarasi diakhiri dengan ";".

**Requirement 6: Preset Button**

6.1 5 preset berbeda satu sama lain pada ≥1 properti
  Thoughts: Ini adalah invariant data statis. Untuk setiap pasang preset yang berbeda, setidaknya satu nilai properti harus berbeda.
  Classification: PROPERTY
  Test Strategy: Untuk setiap kombinasi dua preset yang berbeda, minimal satu key dalam ButtonConfig harus memiliki nilai yang tidak sama.

**Requirement 7: Reset**

7.5 Nilai default = state saat halaman pertama dimuat
  Thoughts: Reset harus menghasilkan state yang identik dengan DEFAULT_STATE, berapapun banyak perubahan yang dilakukan sebelumnya. Ini adalah idempotence property.
  Classification: PROPERTY
  Test Strategy: Untuk semua urutan perubahan state yang valid, memanggil resetState() harus menghasilkan state yang deep-equal dengan DEFAULT_STATE.

**Requirement 8: Randomize**

8.2 Nilai acak harus dalam rentang yang valid per properti
  Thoughts: Berlaku untuk setiap panggilan randomize, bukan contoh spesifik. Kita bisa memanggil buildRandomState() banyak kali dan memverifikasi semua nilai dalam rentang.
  Classification: PROPERTY
  Test Strategy: Untuk setiap pemanggilan buildRandomState(), setiap nilai numerik harus dalam rentang yang ditentukan spec (R8.2).

8.5 + 8.6 Kontras warna acak harus ≥ 4.5:1 (WCAG AA)
  Thoughts: Ini harus berlaku untuk setiap kombinasi warna yang dihasilkan randomizer. Kita bisa memanggil randomizeColors() banyak kali dan memverifikasi rasio kontras.
  Classification: PROPERTY
  Test Strategy: Untuk setiap pemanggilan randomizeColors(), getContrastRatio(result.textColor, result.bgColor) harus ≥ 4.5.

**Property Reflection — Redundancy Check:**
- P1 (HTML generation) dan P2 (CSS completeness) menguji aspek berbeda dari generateCode → keduanya dipertahankan
- P3 (hover selector) dan P4 (CSS formatting) menguji aspek berbeda dari format output → keduanya dipertahankan
- P5 (preset uniqueness) adalah data invariant terpisah → dipertahankan
- P6 (reset idempotence) dan P7+P8 (randomize validity) menguji modul berbeda → semua dipertahankan
- Tidak ada redundansi ditemukan. 8 properties dipertahankan.

---

### Property 1: HTML Generation Contains Button Text

*For any* valid button text string (1–100 characters non-empty), calling `generateHTML(state)` with that text in the state must produce an HTML string containing a `<button>` element with `class="custom-button"` that includes the exact configured text.

**Validates: Requirements 4.1**

---

### Property 2: CSS Generation Includes All Configured Properties

*For any* valid `ButtonConfig`, calling `generateCSS(state)` must produce a CSS string that contains declarations for all of the following properties: `background-color`, `color`, `font-size`, `padding`, `border-radius`, and `border`.

**Validates: Requirements 4.2**

---

### Property 3: Hover Selector Presence Matches Hover Effect Setting

*For any* valid `ButtonConfig`, the output of `generateCSS(state)` must contain a `.custom-button:hover` block if and only if `state.hoverEffect !== "none"`. When `hoverEffect === "none"`, no `:hover` selector may appear in the output.

**Validates: Requirements 4.4, 4.5**

---

### Property 4: Generated CSS Follows Formatting Rules

*For any* valid `ButtonConfig`, every CSS property declaration in the output of `generateCSS(state)` must appear on its own line, be indented with exactly 2 spaces, and end with a semicolon (`;`).

**Validates: Requirements 4.6**

---

### Property 5: All Preset Pairs Differ on At Least One Property

*For any* two distinct preset names from `{ primary, outline, ghost, pill, danger }`, their corresponding `ButtonConfig` objects must differ on at least one property key.

**Validates: Requirements 6.1**

---

### Property 6: Reset Produces Default State

*For any* sequence of `updateState()` calls applied to the application state, calling `resetState()` must result in a state that is deep-equal to `DEFAULT_STATE`, regardless of what changes were made before the reset.

**Validates: Requirements 7.2, 7.5**

---

### Property 7: Randomized Values Are Within Specification Ranges

*For any* call to `buildRandomState()`, the returned `ButtonConfig` must satisfy all of the following range constraints simultaneously:
- `fontSize` ∈ [12, 32]
- `borderRadius` ∈ [0, 50]
- `borderWidth` ∈ [0, 8]
- `paddingHorizontal` ∈ [8, 64]
- `paddingVertical` ∈ [4, 32]
- `fontWeight` ∈ { 400, 500, 600, 700 }

**Validates: Requirements 8.2**

---

### Property 8: Randomized Colors Meet WCAG AA Contrast

*For any* call to `randomizeColors()`, the returned `{ textColor, bgColor }` pair must satisfy `getContrastRatio(textColor, bgColor) >= 4.5`, as required by WCAG AA for normal text.

**Validates: Requirements 8.5, 8.6**

---

## Error Handling

### 1. Invalid CSS Values (R3.5)

Beberapa kombinasi input dapat menghasilkan nilai CSS yang tidak valid (misalnya, warna hex yang tidak lengkap saat color picker dalam transisi). Strategi:

- **Guard di `renderLiveButton()`**: Sebelum mengaplikasikan style, validasi bahwa setiap nilai string warna cocok dengan pola `/^#[0-9a-fA-F]{3,6}$/` dan setiap nilai numerik adalah bilangan terbatas (finite number). Jika validasi gagal, lewati pembaruan untuk properti tersebut saja dan pertahankan nilai sebelumnya.
- **Tidak ada error visual di Live_Button**: Live_Button tidak pernah menampilkan pesan error atau state rusak ke pengguna.

### 2. Clipboard API Tidak Tersedia (R5.5)

- **Deteksi**: Periksa `navigator.clipboard` dan `navigator.clipboard.writeText` sebelum memanggil.
- **Fallback**: Tampilkan `<div role="alert">` dengan pesan: *"Browser ini tidak mendukung fungsi salin otomatis. Silakan pilih teks secara manual dan salin."*
- **Tidak mengubah label tombol**: Label tombol tetap "Copy CSS" / "Copy HTML", tidak berubah ke "Copied!".

### 3. Operasi Clipboard Gagal (R5.6)

- **Tangkap exception** dari `navigator.clipboard.writeText()` menggunakan `try/catch`.
- **Tampilkan pesan error**: `<div role="alert">` dengan pesan: *"Gagal menyalin ke clipboard. Silakan coba lagi."*
- **Tidak mengubah label tombol**: Label tetap tidak berubah ke "Copied!".

### 4. Randomize Contrast Retry Exhausted (R8.6)

- Jika setelah 10 percobaan tidak ada pasangan warna dengan kontras ≥ 4.5:1 yang ditemukan (sangat tidak mungkin tapi terdefinisi dalam spec):
- Gunakan pasangan fallback yang sudah pasti memenuhi syarat: `textColor: "#ffffff"`, `bgColor: "#1d4ed8"` (putih pada biru gelap, kontras ≈ 7.1:1).

### 5. Input Teks Melebihi 100 Karakter (R2.1)

- Atribut `maxlength="100"` pada elemen `<input>` mencegah input lebih dari 100 karakter di level HTML.
- Event handler juga melakukan trim dan slice sebelum memanggil `updateState`.

### 6. State Corruption Prevention

- `updateState(key, value)` hanya menerima key yang terdaftar dalam `DEFAULT_STATE`. Key yang tidak dikenal diabaikan dengan `console.warn()` di development mode.
- Semua nilai numerik divalidasi menggunakan `Number.isFinite()` sebelum disimpan ke state.

---

## Testing Strategy

### Overview

Strategi testing menggunakan pendekatan dual: **unit tests** untuk skenario spesifik dan edge cases, dan **property-based tests** untuk memverifikasi properti universal pada fungsi-fungsi murni. Kedua pendekatan ini saling melengkapi.

Karena ini adalah aplikasi Vanilla JS tanpa build system, testing menggunakan **Vitest** (dapat dijalankan via Node.js dengan konfigurasi minimal, tanpa bundler khusus) sebagai test runner, dan **fast-check** sebagai property-based testing library.

### Library

| Tujuan | Library | Alasan |
|---|---|---|
| Test runner | [Vitest](https://vitest.dev/) | Mendukung ESM, zero-config, cepat |
| Property-based testing | [fast-check](https://fast-check.dev/) | Mature library PBT untuk JavaScript/TypeScript |
| DOM testing (unit) | JSDOM (built-in Vitest) | Testing DOM manipulation tanpa browser |

### Unit Tests (Example-Based)

Unit tests difokuskan pada skenario konkret dan edge cases yang tidak tercakup oleh property tests:

**State Manager:**
- `updateState` dengan key valid mengubah nilai state
- `updateState` dengan key tidak valid diabaikan
- `applyPreset("primary")` menghasilkan state yang identik dengan definisi preset
- `applyPreset` kemudian `resetState` menghasilkan DEFAULT_STATE

**Renderer:**
- `renderLiveButton` dengan warna hex tidak valid mempertahankan style sebelumnya
- `renderLiveButton` dengan hoverEffect="scale-up" menghasilkan CSS `transform: scale(1.05)` di `<style>` tag

**Code Generator:**
- `generateCSS` dengan boxShadow="custom" menggunakan nilai shadowOffsetX/Y/Blur/Spread/Color
- `generateCSS` dengan hoverEffect="lift" menghasilkan shadow yang satu level lebih tinggi
- `generateHTML` menghasilkan teks dengan karakter spesial yang ter-escape dengan benar

**Clipboard Manager:**
- Ketika `navigator.clipboard` undefined, pesan error yang benar ditampilkan
- Ketika `clipboard.writeText()` throw, label tombol tidak berubah ke "Copied!"
- Setelah 2000ms, label tombol kembali ke label asli

**Contrast Checker:**
- `hexToRelativeLuminance("#ffffff")` === 1
- `hexToRelativeLuminance("#000000")` === 0
- `getContrastRatio("#ffffff", "#000000")` === 21
- `meetsWCAGAA("#ffffff", "#3b82f6")` === true (kontras ≈ 4.66)

**Preset Registry:**
- Semua 5 nama preset ada di PRESETS object
- Setiap preset memiliki semua key dari ButtonConfig

**UI Integration (manual/e2e):**
- Layout beralih dari dua-panel ke single-column saat viewport < 1024px
- Semua kontrol dapat dijangkau via Tab key (keyboard navigation)
- Semua input memiliki label yang terhubung via `for`/`id` atau `aria-label`

---

### Property-Based Tests

Setiap property test berjalan minimal **100 iterasi**. Setiap test diberi tag komentar mengacu ke property design document.

Library: **fast-check** — dijalankan dalam environment Vitest.

---

#### Property Test 1: HTML Generation Contains Button Text

```javascript
// Feature: css-button-generator, Property 1: HTML generation contains button text
it("generateHTML always contains configured button text", () => {
  fc.assert(
    fc.property(
      fc.string({ minLength: 1, maxLength: 100 }),
      (text) => {
        const state = { ...DEFAULT_STATE, text }
        const html = generateHTML(state)
        return html.includes(text) &&
               html.includes('class="custom-button"') &&
               html.includes('<button')
      }
    ),
    { numRuns: 100 }
  )
})
```

**Validates: Requirements 4.1**

---

#### Property Test 2: CSS Generation Includes All Configured Properties

```javascript
// Feature: css-button-generator, Property 2: CSS generation includes all configured properties
it("generateCSS always includes all required CSS properties", () => {
  fc.assert(
    fc.property(
      fc.record({
        backgroundColor: fc.hexaString({ minLength: 6, maxLength: 6 }).map(s => `#${s}`),
        textColor:        fc.hexaString({ minLength: 6, maxLength: 6 }).map(s => `#${s}`),
        fontSize:         fc.integer({ min: 10, max: 48 }),
        paddingHorizontal: fc.integer({ min: 0, max: 80 }),
        paddingVertical:   fc.integer({ min: 0, max: 60 }),
        borderRadius:      fc.integer({ min: 0, max: 50 }),
        borderWidth:       fc.integer({ min: 0, max: 10 }),
        borderColor:       fc.hexaString({ minLength: 6, maxLength: 6 }).map(s => `#${s}`),
      }),
      (partial) => {
        const state = { ...DEFAULT_STATE, ...partial }
        const css = generateCSS(state)
        return css.includes("background-color") &&
               css.includes("color") &&
               css.includes("font-size") &&
               css.includes("padding") &&
               css.includes("border-radius") &&
               css.includes("border")
      }
    ),
    { numRuns: 100 }
  )
})
```

**Validates: Requirements 4.2**

---

#### Property Test 3: Hover Selector Presence Matches Hover Effect Setting

```javascript
// Feature: css-button-generator, Property 3: hover selector iff hoverEffect !== none
it("generateCSS contains :hover block iff hoverEffect is not none", () => {
  const hoverEffects = ["none", "darken", "lighten", "scale-up", "scale-down", "lift"]
  fc.assert(
    fc.property(
      fc.constantFrom(...hoverEffects),
      (hoverEffect) => {
        const state = { ...DEFAULT_STATE, hoverEffect }
        const css = generateCSS(state)
        const hasHoverBlock = css.includes(".custom-button:hover")
        const effectIsNone = hoverEffect === "none"
        return hasHoverBlock !== effectIsNone
      }
    ),
    { numRuns: 100 }
  )
})
```

**Validates: Requirements 4.4, 4.5**

---

#### Property Test 4: Generated CSS Follows Formatting Rules

```javascript
// Feature: css-button-generator, Property 4: CSS formatting rules
it("generateCSS uses 2-space indentation and one declaration per line", () => {
  fc.assert(
    fc.property(
      fc.record({
        fontSize:    fc.integer({ min: 10, max: 48 }),
        borderWidth: fc.integer({ min: 0, max: 10 }),
        hoverEffect: fc.constantFrom("none", "darken", "scale-up"),
      }),
      (partial) => {
        const state = { ...DEFAULT_STATE, ...partial }
        const css = generateCSS(state)
        const lines = css.split("\n")
        // Every declaration line must start with 2 spaces and end with ";"
        const declarationLines = lines.filter(l => l.trim().includes(":"))
        return declarationLines.every(l => l.startsWith("  ") && l.trimEnd().endsWith(";"))
      }
    ),
    { numRuns: 100 }
  )
})
```

**Validates: Requirements 4.6**

---

#### Property Test 5: All Preset Pairs Differ on At Least One Property

```javascript
// Feature: css-button-generator, Property 5: preset uniqueness
it("any two distinct presets differ on at least one property", () => {
  const presetNames = Object.keys(PRESETS)
  const pairs = presetNames.flatMap((a, i) =>
    presetNames.slice(i + 1).map(b => [a, b])
  )
  pairs.forEach(([nameA, nameB]) => {
    const a = PRESETS[nameA]
    const b = PRESETS[nameB]
    const differs = Object.keys(DEFAULT_STATE).some(key => a[key] !== b[key])
    expect(differs).toBe(true)
  })
})
// Note: This is a finite enumeration, not randomly generated,
// but is written as a property to validate the invariant across all combinations.
```

**Validates: Requirements 6.1**

---

#### Property Test 6: Reset Produces Default State

```javascript
// Feature: css-button-generator, Property 6: reset idempotence
it("resetState always produces DEFAULT_STATE regardless of prior changes", () => {
  fc.assert(
    fc.property(
      fc.array(
        fc.record({
          key:   fc.constantFrom("fontSize", "borderRadius", "fontWeight", "hoverEffect"),
          value: fc.oneof(fc.integer({ min: 0, max: 50 }), fc.constantFrom("darken", "none")),
        }),
        { minLength: 1, maxLength: 20 }
      ),
      (changes) => {
        resetState()
        changes.forEach(({ key, value }) => updateState(key, value))
        resetState()
        return deepEqual(getState(), DEFAULT_STATE)
      }
    ),
    { numRuns: 100 }
  )
})
```

**Validates: Requirements 7.2, 7.5**

---

#### Property Test 7: Randomized Values Are Within Specification Ranges

```javascript
// Feature: css-button-generator, Property 7: randomize range validity
it("buildRandomState always produces values within specification ranges", () => {
  fc.assert(
    fc.property(
      fc.integer(),   // seed (not actually used; just runs the property multiple times)
      (_seed) => {
        const state = buildRandomState()
        return (
          state.fontSize          >= 12 && state.fontSize          <= 32 &&
          state.borderRadius      >= 0  && state.borderRadius      <= 50 &&
          state.borderWidth       >= 0  && state.borderWidth       <= 8  &&
          state.paddingHorizontal >= 8  && state.paddingHorizontal <= 64 &&
          state.paddingVertical   >= 4  && state.paddingVertical   <= 32 &&
          [400, 500, 600, 700].includes(state.fontWeight)
        )
      }
    ),
    { numRuns: 200 }
  )
})
```

**Validates: Requirements 8.2**

---

#### Property Test 8: Randomized Colors Meet WCAG AA Contrast

```javascript
// Feature: css-button-generator, Property 8: randomize WCAG AA contrast
it("randomizeColors always produces a color pair with contrast ratio >= 4.5", () => {
  fc.assert(
    fc.property(
      fc.integer(),   // seed to drive multiple runs
      (_seed) => {
        const { textColor, bgColor } = randomizeColors()
        return getContrastRatio(textColor, bgColor) >= 4.5
      }
    ),
    { numRuns: 200 }
  )
})
```

**Validates: Requirements 8.5, 8.6**

---

### Test Coverage Summary

| Area | Test Type | Requirements |
|---|---|---|
| HTML code generation | Property (P1) | R4.1 |
| CSS code generation — completeness | Property (P2) | R4.2 |
| CSS code generation — hover selector | Property (P3) | R4.4, R4.5 |
| CSS code generation — formatting | Property (P4) | R4.6 |
| Preset uniqueness | Property (P5) | R6.1 |
| Reset idempotence | Property (P6) | R7.2, R7.5 |
| Randomize range validity | Property (P7) | R8.2 |
| Randomize WCAG contrast | Property (P8) | R8.5, R8.6 |
| Invalid CSS guard | Unit | R3.5 |
| Clipboard API unavailable | Unit | R5.5 |
| Clipboard write failure | Unit | R5.6 |
| Copy label timing (2000ms) | Unit | R5.3, R5.4 |
| Contrast ratio calculations | Unit | R8.5 |
| Preset application | Unit | R6.2 |
| Layout responsiveness | Manual/E2E | R1.1, R1.2, R1.3 |
| Keyboard accessibility | Manual/E2E | R9.3, R9.6 |
| Screen reader labels | Manual/E2E | R9.4 |
