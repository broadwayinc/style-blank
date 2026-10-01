# Style Blank

Style Blank is a css initialization file that resets the default styles of html elements.

It is focused to be unopinionated and minimalistic, it only resets the necessary styles to make your website suitable for modern web design.


## Import the CSS file from CDN

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/style-blank@latest/blank.css">
```


## Design decisions

Every rule in `blank.css` is here to fix a specific browser-default problem with as little opinion as possible: normalize cross-browser inconsistencies and give form controls predictable, consistent sizing — without imposing a visual theme. This section documents each decision and the reasoning behind it.

A few conventions recur throughout:

- **Type sizes use `rem`; control heights share a CSS variable.** The root font-size is set once (see below), so `rem`-based sizing is anchored to the document root and never compounds with nesting the way `em` would. The shared `--style-blank-input-height` custom property keeps controls at a consistent height.
- **`box-sizing: border-box`** is applied to sized controls so a declared height or `100%` width includes padding and border.
- **Nothing sets `appearance: none` except `progress`.** Native checkboxes, radios, selects, and buttons keep their OS rendering; only their box geometry is normalized.

### Root typography (`html`)

```css
html {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif;
    font-size: 18px;
}
```

**`font-family` — the native system font stack.** Unstyled, every engine defaults the document font to a serif (Times New Roman on Windows and macOS, but a substitute like Liberation Serif, DejaVu Serif, or Noto Serif on Linux and Android). Modern interfaces expect sans-serif. Rather than ship a web font — which forces a download and imposes one brand's typographic identity — the stack defers to whatever typeface the user's own OS uses for its UI. It is evaluated left to right, and the first family the platform actually has installed wins:

- `-apple-system` / `BlinkMacSystemFont` — San Francisco on macOS/iOS (Safari/WebKit and Chrome/Blink respectively)
- `'Segoe UI'` — Windows
- `Roboto` — Android / Chrome OS
- `Oxygen`, `Ubuntu`, `Cantarell` — common Linux desktop UI fonts (KDE, Ubuntu, GNOME)
- `'Fira Sans'` — Mozilla's font
- `'Droid Sans'` — pre-Roboto Android fallback
- `'Helvetica Neue'` — older macOS before San Francisco
- `sans-serif` — the generic guarantees a sans-serif fallback rather than dropping back to serif

Its only opinion is the near-universal modern default of sans-serif over the browser's serif: zero network requests, no aesthetic of its own. Bootstrap 4 and GitHub popularized the general idea of a native system-font stack; this particular longer variant (with the Linux-desktop fonts and Droid Sans) is the version that spread through CSS-Tricks / WordPress write-ups, not the exact list those two projects shipped.

**`font-size: 18px`.** This is not a cross-browser fix — the `16px` root default is consistent everywhere — but an intentional adjustment for a slightly larger, more readable baseline on modern screens. It also defines the `rem` base used for control text (`1rem` = 18px, `0.8rem` = 14.4px). The 18px text baseline and 40px control height are usability choices, not responsive measurements: they remain fixed in CSS pixels rather than adapting to a device's physical screen size or pixel density. Because this is an absolute `px` value, it overrides a user's configured default browser font-size preference. Page/browser zoom is unaffected; a strictly preference-respecting reset would express this as a percentage instead.

### Shared control height

```css
body {
    --style-blank-input-height: 40px;
}
```

**`--style-blank-input-height`.** This custom property centralizes the 40px height used by single-line inputs and buttons, and the minimum height used by textareas and selects. Keeping the value in one place makes the controls consistent and easy to adjust together. The 40px baseline is an intentional usability choice for modern screens and comfortable control targets, rather than a browser-normalization fix; it is a fixed CSS-pixel value, not a size that changes with the device.

### Text rendering (`body`)

```css
body {
    text-rendering: optimizeLegibility;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
```

**`text-rendering: optimizeLegibility`.** Hints the engine to prioritize legibility over speed; in practice it enables kerning and common ligatures (`fi`, `fl`). It is inherited, so placing it on `body` covers all document text. Be candid: this is a mild typographic upgrade, not a true cross-browser fix. Modern browsers already kern normal text under the default `auto`, so the visible gain is small, and `optimizeLegibility` has a real cost — it can slow layout/paint on text-heavy pages and historically triggered rendering bugs on older Android/Chrome.

**`-webkit-font-smoothing: antialiased` / `-moz-osx-font-smoothing: grayscale`.** A matched, engine-specific pair (WebKit/Blink and Firefox spellings) that do the same thing: force grayscale, non-subpixel antialiasing. Both only take effect on **macOS** — they do nothing on Windows or Linux. Pairing the two variants means a Mac user sees the same result in Chrome/Safari or Firefox; that cross-engine pairing is the only genuinely normalizing aspect. Otherwise this is the sheet's most opinionated choice: it deliberately renders macOS text thinner because many designers prefer that look. Its impact is also shrinking — since macOS Mojave (10.14) Apple removed subpixel antialiasing system-wide, so the system default is already grayscale and these declarations increasingly change nothing. No cross-browser inconsistency requires it. (For the record, this convention spread through design blogs around 2010–2012, not through Bootstrap — whose Reboot does not set it — and Normalize.css deliberately omits it.)

### Text-like inputs

```css
input:not([type="checkbox"]):not([type="radio"]):not([type="file"]):not([type="submit"]):not([type="reset"]):not([type="button"]) {
    height: var(--style-blank-input-height);
    padding-left: .25em;
    font-size: 0.8rem;
    box-sizing: border-box;
}
```

**The `:not()` exclusion strategy.** There is no positive selector for "text-like input," so the rule uses a blocklist rather than enumerating `text`, `email`, `password`, and so on. This is deliberate: an `<input>` with no `type` attribute renders as `type="text"`, but `input[type="text"]` matches only the *literal* attribute and would miss a bare `<input>`. The `:not()` chain catches it — and any future text-like type — automatically. The excluded types are the ones other rules own: checkbox/radio get square sizing, `file` is styled through `::file-selector-button`, and submit/reset/button join the button group.

Honest trade-off: the blocklist is broad, not surgical. It also matches `range` and `color` (which harmlessly inherit the sizing), the date/time types (`date`, `datetime-local`, `month`, `week`), `hidden` (inert, since it is `display: none`), and `image` — a button-like image-submit control that no other rule owns yet still receives text-field sizing here. So it is future-proof and minimal, but does not carve out *precisely* the inputs owned by other rules.

- **`height: var(--style-blank-input-height)`** — the custom property resolves to `40px`. A UA input's height is computed from font-size, line-height, intrinsic padding, and border, so it drifts across browsers, fonts, and OSes. Sharing this value with buttons and the minimum heights of `select` and `textarea` keeps controls on one grid. Single-line inputs never need to grow, so a hard `height` (rather than `min-height`) is appropriate. The 40px value is a deliberate modern-screen usability choice, not a browser default.
- **`padding-left: .25em`** — keeps text and caret off the left border and evens out the differing small insets UA stylesheets apply. Only the left side is set (not the shorthand) so vertical placement is left to the fixed height's centering. `.25em` scales with the control's own font-size.
- **`font-size: 0.8rem`** — controls do **not** inherit font-size from the page; left alone they fall back to a UA control size (e.g. Chromium's ~13.33px) that varies by browser. `rem` anchors it to the root regardless of nesting. Note this sets *only* size, not family, so text inputs still render in the UA control font (unlike `textarea`, below) — a deliberately light, size-only touch.
- **`box-sizing: border-box`** — under the default `content-box`, the declared height would size only the content box and the UA border (plus any UA vertical padding) would push the rendered box past the shared 40px height, while `width: 100%` from the fieldset rule would overflow once padding is added. `border-box` makes the shared height mean 40px and `100%` mean the container width. This corrects measurement, not looks.

### Buttons & the file-selector button

```css
button,
input[type="submit"],
input[type="reset"],
input::file-selector-button,
input[type="button"] {
    height: var(--style-blank-input-height);
    padding: 0 .5em;
    font-size: 0.8rem;
    box-sizing: border-box;
}

input::file-selector-button {
    margin-right: .5em;
}
```

**Why group them.** `<button>` (whatever its `type`), the three button-type inputs, and the file input's button each carry their own native chrome and intrinsic sizing, so left alone they render at different heights from each other and from text inputs. Collecting them into one rule and sharing `height: var(--style-blank-input-height)`, `font-size: 0.8rem`, and `box-sizing: border-box` with the text-input rule makes buttons line up with text fields. The shared height resolves to `40px`; the padding differs (`padding: 0 .5em` here vs. `padding-left: .25em` on inputs), but `border-box` absorbs it inside the box so the outer height is unchanged. It stays unopinionated by normalizing only geometry and text size: background, border, and color are untouched, so the native button look survives.

- **`padding: 0 .5em`** — vertical padding is zeroed on purpose: height is already set by the shared 40px custom property with `border-box`, so any vertical padding would just eat into the box and shift the label. The `.5em` horizontal padding (relative to the `0.8rem` font-size) keeps the label off the edges.
- **`font-size: 0.8rem`** — matches the inputs, textareas, and selects so all control labels are the same size; `rem` ties it to the root. Only size is set, not family, so button labels stay in the UA control font — an intentionally minimal touch.
- **`input::file-selector-button`** — targets the clickable "Choose File" segment of `<input type="file">` (the reason `file` is excluded from the text-input rule). It is the standardized selector (modern Chrome/Edge, Firefox, Safari), superseding the WebKit-only `::-webkit-file-upload-button`. Styling it through the shared block gives the file input's button the same 40px geometry as every other button.
- **`margin-right: .5em`** — separates that button from the adjacent "No file chosen" filename text, whose default gap varies by browser, for a consistent readable space.

### Checkboxes & radios

```css
input[type="checkbox"],
input[type="radio"] {
    width: 1rem;
    height: 1rem;
    vertical-align: baseline;
}
```

**A dedicated rule.** Checkboxes and radios are fixed-shape toggles, not text-holding boxes, so the `40px` height, horizontal padding, and `0.8rem` sizing applied elsewhere would be meaningless on them. They get their own block and are kept out of the others — the text-input rule (and the fieldset width rule) exclude them via `:not()` chains, while the button rule simply never lists them — so this is the only place they are sized. Crucially the rule sets only box size and alignment; it does **not** set `appearance: none`, so the native check glyph, dot, focus ring, and `accent-color` stay intact.

- **`width: 1rem; height: 1rem`** — the native rendered size (historically ~13px, but it varies by browser/OS/theme and does not reliably track text) is normalized to a fixed square. `rem` ties it to the base font-size so it scales with the type scale rather than freezing at an absolute pixel count. Note this normalizes only the **bounding box**; because `appearance: none` is not set, the widget's actual drawing is still native and differs across browsers. At the `18px` root, `1rem` is 18px — a clean, predictable value noticeably larger than the ~13px native default, not a close match to it.
- **`vertical-align: baseline`** — a deliberate recent change (the previous rule used `vertical-align: text-top` with `margin: 2px`). `baseline` is the CSS initial value of `vertical-align`, so stating it explicitly is the most unopinionated choice: it asserts the browser's natural alignment and overrides any *other cascaded* `vertical-align` on the element. (The property does not inherit, so there is no parent value to guard against.) The old `text-top` pushed the control up toward the font's ascent, making it sit high next to label text; `baseline` restores normal inline alignment. Dropping `margin: 2px` in the same change makes the footprint exactly the `1rem` square, leaving any gap to its label as an author decision.

### Textarea & select

```css
textarea {
    font-family: inherit;
    min-height: var(--style-blank-input-height);
    font-size: 0.8rem;
    box-sizing: border-box;
    padding: .25em 0 0 .25em;
    line-height: 1.33;
}

select {
    min-height: var(--style-blank-input-height);
    font-size: 0.8rem;
    box-sizing: border-box;
    padding: 0 .25em;
}
```

**`textarea { font-family: inherit }`.** Form controls do not inherit `font-family` from the page — the UA gives them their own control font — so an untouched `textarea` clashes with body text, the same way inputs and selects do. `inherit` re-attaches it to the document font (ultimately the system stack on `html`), imposing no new font of its own. This is applied only to `textarea`; input and select here are left on their UA font family, so the normalization is not fully consistent, but `textarea` is where the mismatch is most jarring in multi-line editable text.

**`min-height: var(--style-blank-input-height)` (not `height`) on both `textarea` and `select`.** The shared property resolves to `40px`, matching the other controls so a collapsed `textarea` or single-line `select` lines up with them. `min-height` sets a floor while preserving native multi-row behavior: a `textarea` can be user-resized via its default `resize` handle (and its initial height set by the `rows` attribute), and a `<select multiple>` or `size`d select renders several rows. A fixed `height` would constrain those; `min-height` preserves their natural ability to grow.

- **`textarea { padding: .25em 0 0 .25em }`** — a modest top+left inset (right and bottom `0`). The left `.25em` mirrors the inputs' `padding-left` so the caret sits at the same rhythm; the top nudges the first line off the border. A low-opinion breathing-room choice, not a browser fix.
- **`textarea { line-height: 1.33 }`** — matters because a `textarea` is multi-line. The UA default is `normal` (~1.2), which can feel cramped for editing; `1.33` loosens it slightly. A mild legibility judgment, kept modest.
- **`select { padding: 0 .25em }`** — horizontal inset keeps the selected option text and native dropdown arrow off the edges and matches the other controls' rhythm; vertical padding is `0` because `min-height` plus native centering already place the text.
- **`font-size: 0.8rem`** on both — the same value used on inputs and buttons, so all control text is uniform. `rem` (not `em`) anchors it to the root and avoids compounding with nesting.
- **`box-sizing: border-box`** on both — both carry padding and a native border, and a `textarea` can be stretched to `width: 100%` in a fieldset. `border-box` folds padding and border into the stated dimensions so the shared 40px minimum and `100%` width mean exactly that, rather than overflowing.

### Full-width fields inside a fieldset

```css
form > fieldset textarea,
form > fieldset input:not([type="checkbox"]):not([type="radio"]):not([type="file"]):not([type="submit"]):not([type="reset"]):not([type="date"]):not([type="button"]) {
    width: 100%;
}
```

**`width: 100%` on text fields within a form's direct fieldset.** The child combinator (`>`) limits this rule to fieldsets that are direct children of a form; the descendant combinator (a space) then reaches text fields and textareas anywhere inside that fieldset, including those nested in a wrapper. Stretching those controls to `100%` makes them fill the fieldset's width. The `:not()` chain excludes checkbox, radio, file, submit, reset, and button. Combined with `border-box`, `100%` fills the available width rather than overflowing.

### Progress bar

```css
progress {
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;
    border: none;
    border-radius: 0;
    height: 1rem;
    background-color: #e0e0e0;
    color: currentColor;
}

progress::-webkit-progress-bar   { background-color: #e0e0e0;   border-radius: 0; }
progress::-webkit-progress-value { background-color: currentColor; border-radius: 0; }
progress::-moz-progress-bar      { background-color: currentColor; border-radius: 0; }
```

`<progress>` is the one element the reset takes over completely, because its native rendering differs sharply between engines and is otherwise hard to style predictably. Even so, the goal is a plain, theme-inheriting bar — not a styled one.

**The per-engine pseudo-element model.** WebKit/Blink split the control into two shadow pseudo-elements: `::-webkit-progress-bar` is the **track**, and `::-webkit-progress-value` is the **fill** layered on top — so both must be styled. Firefox has no separate track pseudo-element: the `<progress>` **element itself is the track** (styled via its own `background-color`), and `::-moz-progress-bar` addresses only the **fill**. Despite the name, `::-moz-progress-bar` is the fill — the Gecko analog of `::-webkit-progress-value`, not of `::-webkit-progress-bar`.

**`appearance: none` (+ prefixes).** In WebKit/Blink this is a strict prerequisite: `::-webkit-progress-bar` / `::-webkit-progress-value` styles are only honored once native appearance is disabled. All three spellings are listed because the unprefixed `appearance` is the modern standard while the vendor prefixes cover older engines. In Firefox it is *not* a prerequisite — the element's background and `::-moz-progress-bar` can be colored without it — so there it mainly strips the residual native bevel and border (further removed by `border: none`).

- **`border: none; border-radius: 0`** — browsers historically draw a bevel and rounded ends (Firefox especially). Flattening to a plain rectangle is the least-decorated, most predictable baseline; it removes decoration rather than substituting a new look.
- **`height: 1rem`** — native heights vary across browsers; pinning removes that. At the `18px` root, `1rem` (18px) ties the bar to the type scale and matches the checkbox/radio sizing.
- **`background-color: #e0e0e0`** (element and `::-webkit-progress-bar`) — the unfilled **track** color, declared in both places so it matches across engines (Firefox reads the element background; WebKit reads the pseudo-element). `#e0e0e0` is the one frankly aesthetic choice here: a neutral light gray rather than any brand color, since a track has to be *some* visible color. Honest limitation: it is a fixed light value and does not adapt to dark backgrounds.
- **`color: currentColor`** and **`background-color: currentColor`** on the fills — the **fill** adopts the surrounding text color instead of a hardcoded accent, so the bar inherits whatever color the site already uses. Setting `color: currentColor` on the element is essentially a documented no-op (since `color` inherits anyway); it makes the intent explicit — no imposed hue, the core unopinionated move.

### Tables

```css
table {
    min-width: 100%;
    width: max-content;
    border-collapse: collapse;
}

th,
td {
    border: 1px solid #d0d0d0;
    padding: .5em .75em;
    text-align: left;
}

th {
    background-color: #f2f2f2;
}
```

Default table borders differ across browsers (`border-spacing`, double borders between cells) and unstyled tables are easy to misread once they grow wide, so this section gives tables a visible grid and a readable header — without picking a brand color.

- **`min-width: 100%; width: max-content`** — a table should at least fill its parent's width, but a wide table with many columns should be free to grow past that rather than being squeezed and wrapped. `min-width: 100%` sets the floor; `width: max-content` lets the table expand to fit its content when the content is wider than the parent.
- **`border-collapse: collapse`** — the UA default, `separate`, draws a gap between adjacent cell borders (and doubles them visually). `collapse` merges adjacent borders into a single line, which is the more common, readable expectation for a data table.
- **`th, td { border: 1px solid #d0d0d0 }`** — gives every cell a visible, consistent light-gray grid line; without it, bare tables have no borders at all in most browsers.
- **`th, td { padding: .5em .75em; text-align: left }`** — UA default cell padding is inconsistent and often too tight; the padding here is sized relative to the cell's own font. `text-align: left` overrides the UA default of centered `<th>` text so headers line up with left-aligned body text.
- **`th { background-color: #f2f2f2 }`** combined with `font-weight: bold` already applied by the UA default — gives the header row a distinct, neutral band so it reads as a header at a glance, without imposing a brand color.

