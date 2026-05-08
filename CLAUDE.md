# Melbourne Trip — Developer Notes

## CRITICAL: Thai Translation Sync Rule

**Whenever ANY English content is changed in `index.html`, the corresponding Thai translation in the `TRANSLATIONS.th` object (inside the `<script>` block) MUST also be updated.**

This project has a full EN/TH language toggle. The Thai translations live entirely in the `TRANSLATIONS` JavaScript object near the bottom of `index.html` (search for `const TRANSLATIONS =`). If English content is updated without updating Thai, the Thai version will show stale or incorrect content.

---

## i18n Architecture

### How it works

- **Simple text keys**: Elements with `data-i18n="key"` get their `textContent` replaced.
- **HTML keys**: Elements with `data-i18n-html="key"` get their `innerHTML` replaced (used for whole sections).
- **Placeholder keys**: Elements with `data-i18n-placeholder="key"` get their `placeholder` attribute replaced.
- Language preference is stored in `localStorage` under `melb_lang`.
- `setLang(lang)` applies all translations and re-runs expandable panel injection.

---

## Translation Key Map

### Simple text keys (`data-i18n`)

| Key | Where used | English value |
|-----|-----------|---------------|
| `header.badge` | Hero badge | Family Trip |
| `header.title` | Hero h1 | Melbourne, Australia |
| `header.subtitle` | Hero subtitle paragraph | Two Families · 6 Travellers · 7 Nights · August 2026 |
| `meta.dates` | Trip meta span | ✈️ Aug 17 – 23, 2026 |
| `meta.nights` | Trip meta span | 🏨 7 Nights |
| `meta.people` | Trip meta span | 👨‍👩‍👧 6 Travellers |
| `tab.map` | Tab button | Map |
| `tab.itinerary` | Tab button | Itinerary |
| `tab.flights` | Tab button | Flights |
| `tab.hotel` | Tab button | Hotel |
| `tab.activities` | Tab button | Activities |
| `tab.car` | Tab button | Car |
| `tab.photos` | Tab button | Photos |
| `tab.tips` | Tab button | Tips |
| `tab.expenses` | Tab button | Expenses |
| `pw.title` | Password gate title | Melbourne Trip 2026 |
| `pw.subtitle` | Password gate subtitle | Enter the password to access the trip details |
| `pw.placeholder` | Password input placeholder | Password |
| `pw.button` | Password submit button | Enter |
| `pw.error` | Wrong password message | Wrong password — try again |
| `map.day-label` | Map filter label | Day |
| `map.all` | Map filter button | All |
| `map.d0` | Map filter button | Pre |
| `map.d1` | Map filter button | D1 |
| `map.d2` | Map filter button | D2 |
| `map.d3` | Map filter button | D3 |
| `map.d4` | Map filter button | D4 |
| `map.d5` | Map filter button | D5 |
| `map.d6` | Map filter button | D6 |
| `map.hotel-label` | Map info card | 🏨 Hotel |
| `map.colors-label` | Map info card | Day Colors |
| `map.drive-label` | Map info card | Driving Note |
| `map.drive-note` | Map info card | Drive on the left in Australia |
| `footer` | Footer text | Made with ❤️ for the Melbourne Trip 2026 |

### Day card header keys (separate spans inside each day card)

Each day card (Day 0 through Day 6 + Departure) has four `data-i18n` spans:

| Pattern | Example keys |
|---------|-------------|
| `dayN.num` | `day0.num`, `day1.num` … `dayDep.num` |
| `dayN.date` | `day0.date`, `day1.date` … `dayDep.date` |
| `dayN.badge` | `day0.badge`, `day1.badge` … `dayDep.badge` |
| `dayN.title` | `day0.title`, `day1.title` … `dayDep.title` |

### Day card body HTML keys (`data-i18n-html`)

Each day card's full timeline + tip-box content is wrapped in a `<div data-i18n-html="dayN.body">`.

| Key | Content |
|-----|---------|
| `day0.body` | Day 0 (Pre-departure / arrival day) full timeline HTML |
| `day1.body` | Day 1 full timeline HTML |
| `day2.body` | Day 2 full timeline HTML |
| `day3.body` | Day 3 full timeline HTML |
| `day4.body` | Day 4 full timeline HTML |
| `day5.body` | Day 5 full timeline HTML |
| `day6.body` | Day 6 full timeline HTML |
| `dayDep.body` | Departure day full timeline HTML |

### Section-level HTML keys (`data-i18n-html`)

Each major tab's content is wrapped in a `<div data-i18n-html="section.X">`.

| Key | Tab | Notes |
|-----|-----|-------|
| `section.flights` | Flights tab | Full flight card HTML |
| `section.hotel` | Hotel tab | Full hotel info HTML; `loadHotelPhotos()` is re-called after swap |
| `section.activities` | Activities tab | Activity cards HTML |
| `section.car` | Car tab | Car rental details HTML |
| `section.photos` | Photos tab | **NOT translated** — kept in English (camera specs/drone rules) |
| `section.tips` | Tips tab | Travel tips HTML |
| `section.expenses` | Expenses tab | Budget breakdown HTML |

---

## How to Update When English Content Changes

### Changing simple text (tab labels, header, etc.)

1. Edit the English text in the HTML element.
2. Find the same key in `TRANSLATIONS.th` and update the Thai text.

### Changing a day card body (itinerary content)

1. Edit the HTML inside `<div data-i18n-html="dayN.body">` in the HTML.
2. Find `TRANSLATIONS.th['dayN.body']` in the script block and update the Thai HTML string.
   - Keep landmark names in **English** inside Thai text (e.g., "Federation Square", "Melbourne Zoo") so that the expandable info panels still work — they use regex matching on the English names.

### Changing a full section tab (Flights, Hotel, Activities, etc.)

1. Edit the HTML inside `<div data-i18n-html="section.X">` in the HTML.
2. Find `TRANSLATIONS.th['section.X']` in the script block and update the Thai HTML string.
3. Note: `TRANSLATIONS.en['section.X']` is **captured at runtime** from the live DOM at page load — you do NOT need to update the `en` side manually.

### Adding a new day or section

1. Add the `data-i18n-html` wrapper div in the HTML.
2. Add the EN runtime capture line: `TRANSLATIONS.en['newKey'] = document.querySelector('[data-i18n-html="newKey"]')?.innerHTML;`
3. Add the full Thai HTML string as `TRANSLATIONS.th['newKey'] = \`...\`;`
4. Add the key to `setLang()` if it's a new section type.

---

## Files

- `index.html` — single-file app; all HTML, CSS, and JS in one file
- `CLAUDE.md` — this file

## Key JS functions

- `setLang(lang)` — applies all translations, re-injects expandables, reloads hotel photos
- `toggleLang()` — switches between EN and TH, saves to localStorage
- `injectExpandables()` (IIFE) — injects expandable info panels for landmarks; idempotent (checks `lm-item` class)
- `attachPanelPublic` — exposed reference to inner `attachPanel` so `setLang()` can re-inject panels after Thai content replaces itinerary HTML
- `loadHotelPhotos()` — must be called after `section.hotel` HTML is replaced
