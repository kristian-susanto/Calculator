# 🧮 Calculator — Basic & Scientific

A responsive, dependency-free calculator supporting **basic** and **scientific** modes with **light/dark themes**, live expression preview, and full keyboard support. Everything lives in a single `index.html` file — no build step, no libraries.

---

## 📑 Table of Contents

1. [Features](#-features)
2. [Quick Start](#-quick-start)
3. [Project Structure](#-project-structure)
4. [UI Overview](#-ui-overview)
5. [Supported Syntax](#-supported-syntax)
6. [Supported Functions & Operators](#-supported-functions--operators)
7. [Angle Mode (DEG / RAD)](#-angle-mode-deg--rad)
8. [Keyboard Shortcuts](#-keyboard-shortcuts)
9. [Themes](#-themes)
10. [Persistence (localStorage)](#-persistence-localstorage)
11. [Architecture & Internals](#-architecture--internals)
12. [Math Engine Details](#-math-engine-details)
13. [Number Formatting Rules](#-number-formatting-rules)
14. [Error Handling](#-error-handling)
15. [Accessibility](#-accessibility)
16. [Customization Guide](#-customization-guide)
17. [Browser Support](#-browser-support)
18. [Known Limitations](#-known-limitations)

---

## ✨ Features

| Feature                    | Description                                                               |
| -------------------------- | ------------------------------------------------------------------------- |
| **Dual modes**             | Switch between **Basic** (4×5 keypad) and **Scientific** (extra 5×5 pad)  |
| **Live preview**           | Shows `= result` under the expression as you type, before pressing `=`    |
| **Auto-close parentheses** | Unclosed `(` are automatically closed during evaluation                   |
| **Angle unit toggle**      | `DEG` / `RAD` for trigonometric and inverse-trig functions                |
| **Light / Dark theme**     | Respects OS `prefers-color-scheme`, with manual override saved to storage |
| **Keyboard support**       | Type digits, operators, `Enter` to evaluate, `Esc` to clear               |
| **Smart backspace**        | Deletes an entire function token (e.g. `sin(`) in one press               |
| **Sign toggle (`±`)**      | Wraps trailing numbers in `(-…)` or unwraps them                          |
| **Postfix operators**      | Factorial `!` and percent `%`                                             |
| **Result continuation**    | After `=`, pressing an operator continues from the result                 |
| **Animated feedback**      | `pop` animation on success, `shake` animation on error                    |
| **Fully responsive**       | Optimized layouts for ≤430px and ≤340px viewports                         |
| **Reduced motion**         | Honors `prefers-reduced-motion`                                           |
| **No dependencies**        | Vanilla HTML + CSS + JavaScript, single file                              |

---

## 🚀 Quick Start

1. Save the code as `index.html`.
2. Open it in any modern browser (double-click, or drag into a tab).
3. That's it — no server, no install, no internet connection required.

---

## 📁 Project Structure

The file is organized into three logical sections:

```
index.html
├── <head>
│   ├── <meta> tags (charset, viewport, description)
│   ├── Inline SVG favicon (🧮 emoji)
│   ├── <title>
│   └── <style>  ← all CSS
│       ├── Theme tokens (:root / [data-theme="dark"])
│       ├── Base & reset
│       ├── App shell, top bar, segmented controls
│       ├── Display & preview
│       ├── Scientific pad & keypad grids
│       ├── Key styles (.key, .fn, .op, .equals)
│       ├── Footnote
│       └── Media queries (430px, 340px, reduced motion)
├── <body>
│   ├── .app
│   │   ├── <header class="topbar">  ← brand, mode switch, theme toggle
│   │   └── <main class="calculator">
│   │       ├── .display  (#display, #expr, #preview)
│   │       ├── .sci-area (#sciArea, #angleSwitch, .sci-pad)
│   │       ├── .keypad   (basic keys)
│   │       └── .footnote (keyboard tips)
└── <script>  ← all JS (IIFE, strict mode)
    ├── Theme handling
    ├── Math engine (tokenizer + recursive-descent parser)
    ├── UI state & render
    ├── Actions (insert, backspace, clear, negate, equals)
    └── Event wiring (clicks, keyboard, mode/angle switches)
```

---

## 🖥 UI Overview

### Top Bar

- **Brand block** — logo icon + "Calculator" title + "Basic & Scientific" subtitle (subtitle hidden on ≤430px).
- **Segmented mode switch** — `Basic` | `Scientific`.
- **Theme toggle** — circular button; sun icon in light mode, moon icon in dark mode. Rotates 18° on hover.

### Display

- `.expr` — the main expression, right-aligned, tabular numbers, horizontally scrollable (scrollbar hidden).
- `.preview` — the live `= result` preview, muted color; turns red and shows `Error` on evaluation failure.

### Scientific Pad (visible in Scientific mode only)

- **Angle unit switch** — `DEG` | `RAD`.
- **5×5 grid** of function keys: trig, inverse trig, logs, roots, powers, constants, parentheses, factorial.

### Main Keypad (4 columns)

```
AC   ⌫   %   ÷
7    8   9   ×
4    5   6   −
1    2   3   +
±    0   .   =
```

---

## 📝 Supported Syntax

### Numeric literals

| Input    | Meaning                                                                 |
| -------- | ----------------------------------------------------------------------- |
| `123`    | Integer                                                                 |
| `3.14`   | Decimal                                                                 |
| `.5`     | Leading-dot decimal (parsed as `0.5`)                                   |
| `1e3` ❌ | **Not supported** — `e` is Euler's number, so `1e3` becomes `1 × e × 3` |

> More than one `.` in a single number token throws `Invalid number`.

### Constants

| Symbol      | Value     |
| ----------- | --------- |
| `π` or `pi` | `Math.PI` |
| `e`         | `Math.E`  |

### Operators

| Display            | Internal      | Precedence | Associativity |
| ------------------ | ------------- | ---------- | ------------- |
| `+`                | `+`           | Lowest     | Left          |
| `−` (U+2212) / `-` | `-`           | Low        | Left          |
| `×` / `*`          | `*`           | Medium     | Left          |
| `÷` / `/`          | `/`           | Medium     | Left          |
| `^`                | `^`           | High       | Right         |
| `-` (unary)        | negation      | High       | Right         |
| `!` (postfix)      | factorial     | Highest    | —             |
| `%` (postfix)      | divide by 100 | Highest    | —             |

> **Implicit multiplication** is supported: `2(3+4)`, `2π`, `3sin(30)` all multiply automatically.

---

## 🧮 Supported Functions & Operators

### Function keys in the Scientific pad

| Key     | Inserted text  | Description                             |
| ------- | -------------- | --------------------------------------- |
| `sin`   | `sin(`         | Sine (respects angle mode)              |
| `cos`   | `cos(`         | Cosine (respects angle mode)            |
| `tan`   | `tan(`         | Tangent (respects angle mode)           |
| `ln`    | `ln(`          | Natural logarithm (base _e_)            |
| `log`   | `log(`         | Base-10 logarithm                       |
| `sin⁻¹` | `asin(`        | Arcsine (returns in current angle mode) |
| `cos⁻¹` | `acos(`        | Arccosine                               |
| `tan⁻¹` | `atan(`        | Arctangent                              |
| `eˣ`    | `e^(`          | _e_ raised to a power                   |
| `10ˣ`   | `10^(`         | 10 raised to a power                    |
| `√`     | `√(` → `sqrt(` | Square root                             |
| `∛`     | `∛(` → `cbrt(` | Cube root                               |
| `xʸ`    | `^`            | Power operator                          |
| `x²`    | `^2`           | Square                                  |
| `1/x`   | `^(-1)`        | Reciprocal                              |
| `(`     | `(`            | Open parenthesis                        |
| `)`     | `)`            | Close parenthesis                       |
| `π`     | `π`            | Pi constant                             |
| `e`     | `e`            | Euler's number                          |
| `n!`    | `!`            | Factorial (postfix)                     |

### Additional functions available in the engine

These exist in the `FUNCS` table and can be typed manually, even though they have no dedicated button:

`sinh`, `cosh`, `tanh`, `log2`, `abs`, `exp`

Example: typing `sinh(1)` works.

### Basic keypad actions

| Key  | Action                                    |
| ---- | ----------------------------------------- |
| `AC` | Clear everything (`clear`)                |
| `⌫`  | Backspace, function-aware (`back`)        |
| `±`  | Toggle sign of trailing number (`negate`) |
| `=`  | Evaluate (`equals`)                       |
| `%`  | Postfix percent (inserts `%`)             |

---

## 📐 Angle Mode (DEG / RAD)

The angle unit affects **only** trigonometric and inverse-trigonometric functions:

| Function               | DEG behavior                       | RAD behavior        |
| ---------------------- | ---------------------------------- | ------------------- |
| `sin`, `cos`, `tan`    | Input converted via `x · π / 180`  | Input used directly |
| `asin`, `acos`, `atan` | Output converted via `x · 180 / π` | Output in radians   |

- Default mode is **DEG**.
- Switching the unit instantly recomputes the live preview.
- The choice is **not** persisted across reloads (resets to DEG).

---

## ⌨️ Keyboard Shortcuts

| Key                  | Action                 |
| -------------------- | ---------------------- |
| `0`–`9`              | Insert digit           |
| `.` or `,`           | Insert decimal point   |
| `+`                  | Add                    |
| `-`                  | Subtract (inserts `−`) |
| `*`, `x`, `X`        | Multiply (inserts `×`) |
| `/`                  | Divide (inserts `÷`)   |
| `^`                  | Power                  |
| `(` `)`              | Parentheses            |
| `!`                  | Factorial              |
| `%`                  | Percent                |
| `Enter` or `=`       | Evaluate               |
| `Backspace`          | Delete last token      |
| `Escape` or `Delete` | Clear all              |

> Modifier combinations (`Ctrl`, `Cmd`, `Alt`) are ignored, so browser shortcuts still work.

---

## 🎨 Themes

Themes are driven by the `data-theme` attribute on `<html>`:

```html
<html lang="en" data-theme="light"></html>
```

All colors come from CSS custom properties defined in two blocks:

- `:root { … }` — light theme tokens
- `[data-theme="dark"] { … }` — dark theme overrides

### Token reference

| Token                  | Purpose                              |
| ---------------------- | ------------------------------------ |
| `--bg`                 | Page background                      |
| `--glow-1`, `--glow-2` | Radial gradient accent glows         |
| `--surface`            | Card background                      |
| `--surface-2`          | Display / scientific keys background |
| `--text`               | Primary text                         |
| `--text-muted`         | Secondary text, function keys        |
| `--border`             | Borders                              |
| `--btn`, `--btn-hover` | Key backgrounds                      |
| `--accent`             | Brand color (indigo)                 |
| `--accent-soft`        | Translucent accent (logo bg)         |
| `--accent-contrast`    | Text on accent                       |
| `--shadow`             | Card shadow                          |
| `--key-shadow`         | Key shadow                           |

### Initial theme resolution

1. Read `localStorage["calc.theme"]`.
2. If it is `"dark"` or `"light"`, use it.
3. Otherwise fall back to `prefers-color-scheme`.
4. Toggling writes the new value back to storage.

---

## 💾 Persistence (localStorage)

| Key          | Values                | Set when             |
| ------------ | --------------------- | -------------------- |
| `calc.theme` | `"light"` \| `"dark"` | Theme toggle clicked |
| `calc.mode`  | `"basic"` \| `"sci"`  | Mode switch clicked  |

All storage access is wrapped in `try/catch`, so the app still works in private/incognito modes or when storage is blocked.

> **Not persisted:** angle mode, current expression.

---

## 🏗 Architecture & Internals

Everything is wrapped in an **IIFE** with `"use strict"`.

### 1. Theme module

- `prefersDark()` — media query check.
- `applyTheme(theme)` — sets attribute + saves.
- `initTheme()` — runs immediately on load.
- Click listener on `#themeToggle`.

### 2. Math engine

#### Tokenizer

`tokenize(input)` converts a raw string into a token array:

```
"2+3×sin(30)"  →  [num 2, op +, num 3, op *, name sin, op (, num 30, op )]
```

- Strips all whitespace.
- Groups digits and `.` into `num` tokens.
- Groups letters into `name` tokens (lowercased).
- Maps display symbols to ASCII via `SYMBOL_MAP`:
  ```js
  { "×": "*", "÷": "/", "−": "-", "–": "-", "—": "-" }
  ```
- Converts `π` → `name "pi"`, `√` → `name "sqrt"`, `∛` → `name "cbrt"`.
- Passes through `+ - * / ^ ( ) ! %`.
- Throws on any unknown character.

#### Parser (recursive descent)

`evaluate(input)` walks the token list with a cursor `pos`:

```
parseExpression  →  parseTerm  →  parseUnary  →  parsePower  →  parsePostfix  →  parsePrimary
```

| Function          | Responsibility                                    |
| ----------------- | ------------------------------------------------- |
| `parseExpression` | Handles `+` and `-` (lowest precedence)           |
| `parseTerm`       | Handles `*`, `/`, and **implicit multiplication** |
| `parseUnary`      | Handles prefix `-` and `+`                        |
| `parsePower`      | Handles right-associative `^`                     |
| `parsePostfix`    | Handles `!` and `%`, repeatable                   |
| `parsePrimary`    | Numbers, `( … )`, constants, and function calls   |

Helper predicates:

- `peek()` / `next()` — lookahead and consume.
- `isOp(token, value)` — operator type check.
- `startsValue(token)` — true for `num`, `name`, or `(`; used for implicit multiplication.

At the end, if any tokens remain unconsumed, it throws `Unexpected token`.

### 3. UI state

```js
var state = {
  expr: "", // raw expression string
  preview: "", // "= 42" or ""
  previewIsError: false, // styles preview red
  justEvaluated: false, // true right after pressing "="
  mode: "basic", // "basic" | "sci"
};
```

- `render()` — writes to the DOM, toggles error class, auto-scrolls both lines to the right.
- `updatePreview()` — evaluates `autoClose(expr)` silently; sets `preview` only on success and only if the result differs from the input.
- `update()` — `updatePreview()` + `render()`.

### 4. Actions

| Function       | Behavior                                                                                                                  |
| -------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `insert(text)` | If `justEvaluated` and `text` is **not** in `CONTINUE_OPS`, clears the expression first, then appends                     |
| `backspace()`  | Checks `FUNC_NAMES` for a trailing `name(` and deletes the whole token; otherwise deletes one character                   |
| `clearAll()`   | Resets expression, preview, error flag, and `justEvaluated`                                                               |
| `negate()`     | Regex-based: unwraps `(-123)` → `123`, or wraps `123` → `(-123)`; falls back to appending `(-`                            |
| `equals()`     | Evaluates, formats, stores result in `expr`, sets `justEvaluated = true`, plays `pop`; on failure shows `Error` + `shake` |

`CONTINUE_OPS`:

```js
["+", "−", "×", "÷", "^", "!", "%", ")"];
```

`FUNC_NAMES` (used by smart backspace):

```js
[
  "asin",
  "acos",
  "atan",
  "sinh",
  "cosh",
  "tanh",
  "sin",
  "cos",
  "tan",
  "log2",
  "log",
  "ln",
  "sqrt",
  "cbrt",
  "abs",
  "exp",
  "√",
  "∛",
];
```

### 5. Event wiring

- **Click delegation** on `.calculator` — reads `data-insert` or `data-action`.
- **Mode switch** — `#modeSwitch` with `data-mode`.
- **Angle switch** — `#angleSwitch` with `data-angle`.
- **Keyboard** — global `keydown` listener with `preventDefault()` for handled keys.

---

## 🔢 Number Formatting Rules

`formatNumber(n)` produces the display string:

| Condition                              | Output                                                                           |
| -------------------------------------- | -------------------------------------------------------------------------------- |
| `NaN`                                  | `"Error"`                                                                        |
| `+Infinity`                            | `"∞"`                                                                            |
| `-Infinity`                            | `"-∞"`                                                                           |
| `-0`                                   | `"0"`                                                                            |
| `abs ≥ 1e15` or `abs < 1e-9` (and ≠ 0) | Exponential, trailing zeros trimmed: `n.toExponential(9).replace(/\.?0+e/, "e")` |
| Otherwise                              | `String(parseFloat(n.toPrecision(12)))` — up to 12 significant digits            |

**Examples**

```
1/3            →  0.333333333333
2^60           →  1.1529215046e+18
1e-10          →  1e-10
0.1 + 0.2      →  0.3        (floating-point noise trimmed)
5!             →  120
```

---

## ⚠️ Error Handling

| Situation                       | Behavior                                  |
| ------------------------------- | ----------------------------------------- |
| Unknown character               | Throws `Unexpected character "x"`         |
| Multiple dots in a number       | Throws `Invalid number`                   |
| Missing `)`                     | Throws `Missing closing parenthesis`      |
| Missing `(` after function name | Throws `Expected "(" after <name>`        |
| Unknown identifier              | Throws `Unknown name "xyz"`               |
| Trailing tokens                 | Throws `Unexpected token`                 |
| `NaN` result from `=`           | Preview shows red `Error`, display shakes |

**During typing**, preview errors are swallowed silently — so partial input like `sin(` or `2+` doesn't show red text.

**Auto-close** (`autoClose`) counts unmatched `(` and appends the missing `)` characters before evaluation, so `sin(30` evaluates correctly.

---

## ♿ Accessibility

- `aria-live="polite"` + `aria-atomic="true"` on the display announce results to screen readers.
- Mode and angle switches use `role="group"` with `aria-label`.
- The theme toggle has `aria-label="Toggle light and dark theme"` and a `title`.
- All keys are real `<button>` elements — focusable and activatable via keyboard.
- `:focus-visible` draws a 2px accent outline with 2px offset.
- `prefers-reduced-motion: reduce` disables all animations and transitions.
- Decorative SVGs use `aria-hidden="true"` (logo) or carry no text content.

---

## 🎛 Customization Guide

### Change the accent color

Edit `--accent` in both `:root` and `[data-theme="dark"]`:

```css
:root {
  --accent: #4f46e5;
} /* indigo */
[data-theme="dark"] {
  --accent: #6366f1;
}
```

### Change the card width

```css
.app {
  max-width: 480px;
} /* try 380px for a compact look */
```

### Add a new function

1. Add it to the `FUNCS` table:
   ```js
   FUNCS.cot = function (x) {
     return 1 / Math.tan(toRad(x));
   };
   ```
2. (Optional) Add a button in `.sci-pad`:
   ```html
   <button type="button" class="key sci" data-insert="cot(" title="Cotangent">
     cot
   </button>
   ```
3. (Optional) Add `"cot"` to `FUNC_NAMES` so backspace deletes it as one token.

### Add a new keyboard shortcut

Extend the `switch (k)` block in the `keydown` listener:

```js
case "s":
  insert("sin(");
  break;
```

### Change default angle mode

```js
var angleMode = "RAD"; // was "DEG"
```

### Change precision

```js
// in formatNumber
return String(parseFloat(n.toPrecision(12))); // try 15
```

---

## 🌐 Browser Support

Works in any browser supporting:

- CSS custom properties
- CSS Grid
- `Element.closest()`
- `requestAnimationFrame()`
- `localStorage` (optional — degrades gracefully)
- `matchMedia` (optional — defaults to light theme)

Tested target: current Chrome, Firefox, Safari, Edge, and mobile equivalents.

---

## 🚧 Known Limitations

- **No scientific notation input** — `1e3` is parsed as `1 × e × 3` (Euler's number). Type `1000` or `10^3` instead.
- **No variable memory** — `M+`, `M-`, `MR`, `ANS` are not implemented.
- **Factorial domain** — only non-negative integers ≤ 170 (returns `Infinity` above 170).
- **Angle mode is not persisted** — resets to `DEG` on reload.
- **No calculation history** — only the latest result is retained (via the continue-after-`=` behavior).
- **No thousands separators** in the display.
- **Exponential output uses `e+` notation**, which is _not_ re-parseable as input.
- **No hyperbolic buttons** — `sinh`/`cosh`/`tanh` exist in the engine but must be typed.

---

## 📄 License

Free to use, modify, and distribute for personal or commercial projects. Attribution appreciated but not required.

---

**Built with:** vanilla HTML, CSS, and JavaScript — zero dependencies. 🎉
