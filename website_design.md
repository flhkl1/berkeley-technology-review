# design.md — Berkeley Technology Review

Visual system for the BTR website: color, type, and the one rule everything else follows from. Pairs with `website_style.md` (engineering principles) — that file governs how the page loads and responds; this one governs what it looks like. A coding agent implementing the site should treat both as source of truth and keep them in sync: `website_style.md` §1 asks for inlined critical CSS, and the `:root` block below is exactly what belongs in that inline block.

**Direction name:** Signal. **Prime rule:** text is black on a near-white ground, everywhere, always. Color is not a palette to draw from — it is one signature mark, rationed to a single appearance per page. If a rule below tempts you to add a second color for "visual interest," don't; that instinct is the thing this system exists to override.

---

## 1. Tokens

| Token | Hex | Role | Contrast vs. adjacent text/ground | Rule |
|---|---|---|---|---|
| `--color-ground` | `#FDFCF9` | Page background | — | Default background everywhere. Slightly warm, not pure white. |
| `--color-surface` | `#FFFFFF` | Cards, article body, elevated panels | — | Use where content needs to read as "the page within the page." |
| `--color-ink` | `#111111` | **All** headlines, body text, bylines, links | 18.4:1 on ground · 18.9:1 on surface | Never anything but this on text. No colored headlines. No colored links. |
| `--color-muted` | `#6B6B68` | Kickers, captions, decks, metadata, rules-as-text | 5.2:1 on ground (passes AA at all sizes) | Secondary text only — never a primary heading or a link. |
| `--color-rule` | `#E7E5DD` | Hairline borders, dividers | — | Structural only, not decorative. |
| `--color-track` | `#EEEDE6` | Chart/progress baselines | — | The "empty" state of any bar or meter. |
| `--color-compare` | `#B7B5AC` | Secondary/comparison bars, link underlines | — | For "before" values, prior baselines, de-emphasized data. |
| `--color-blue` | `#003262` | Berkeley Blue — the mark; second data series if one is ever needed | 12.5:1 on ground | **Never** body text or link color. See §3. |
| `--color-gold` | `#FDB515` | California Gold — the mark; a single flagged data point | 1.7:1 on ground — **fails as text at any size** | Fill/dot only. Never a text color, never a background under white text. See §3. |

## 2. Type

- **Display / headlines:** Libre Caslon Display — `font-family: "Libre Caslon Display", Georgia, serif;`
- **Body:** Libre Caslon Text — `font-family: "Libre Caslon Text", Georgia, serif;`
- **Kickers, figure labels, footnotes, metadata:** IBM Plex Mono — `font-family: "IBM Plex Mono", ui-monospace, monospace;`, always set in `--color-muted`, letter-spacing `0.16em`, uppercase.

Load via one Google Fonts request (fine under `website_style.md` §1 — it's a single request, not a render-blocker if `display=swap` is set):

```
https://fonts.googleapis.com/css2?family=Libre+Caslon+Display&family=Libre+Caslon+Text:ital,wght@0,400;0,700;1,400&family=IBM+Plex+Mono:wght@400;500&display=swap
```

Reference sizes (from the approved mockup — treat as starting values, not a rigid scale):

| Use | Font | Size | Notes |
|---|---|---|---|
| H1 / masthead headline | Display | 46px | line-height 1.04 |
| H2 / article title | Display | 33px | line-height 1.12 |
| Body | Text | 16px | line-height 1.55 |
| Dek / subhead | Text | 15px | color `--color-muted` |
| Kicker | Mono | 11px | letter-spacing 0.16em, uppercase, `--color-muted` |
| Caption / footnote | Mono | 11–12px | `--color-muted` |

## 3. The mark — the only place color lives

Two 7px dots, Blue then Gold, 3px apart, next to the wordmark. That is the entire color budget of the page.

```html
<span class="btr-mark" aria-hidden="true">
  <span class="btr-mark__dot btr-mark__dot--blue"></span>
  <span class="btr-mark__dot btr-mark__dot--gold"></span>
</span>
```

```css
.btr-mark { display: inline-flex; gap: 3px; }
.btr-mark__dot { width: 7px; height: 7px; border-radius: 50%; }
.btr-mark__dot--blue { background: var(--color-blue); }
.btr-mark__dot--gold { background: var(--color-gold); }
```

Rules, in order of how often they'll be tested by a "can we just—":

1. The mark appears **once per page**, beside the masthead wordmark. Not in the footer too. Not repeated per article card in a list.
2. Gold may additionally flag **one** number per report — the figure the piece is actually about (e.g., a headline success-rate jump) — as a small filled dot next to that one data point. Never a second time on the same page.
3. Blue is the reserve second data-series color if a chart ever needs more than grayscale. Reach for it before gold touches a chart under any circumstance.
4. Neither color is ever a text color, a link color, a button fill, or a background under white text — gold fails contrast as text outright (1.7:1), and using blue for text/links reopens the "which blue is the link blue" problem this system exists to close.

## 4. Layout primitives (from the approved mockup)

- Page/article padding: `56px` (desktop). Scale down, don't redesign, at narrow widths.
- Card padding: `30px 32px 28px`.
- Section gaps: `30px` between major blocks, `18px` within a card, `12px`–`14px` between a label and its content.
- Borders: `1px solid var(--color-rule)`.
- Radius: flat by default (`0`); `2px` at most on a card. No pill buttons, no heavy rounding — that reads as an app, not a report.

## 5. `:root` block — inline this per `website_style.md` §1

```css
:root {
  --color-ground: #FDFCF9;
  --color-surface: #FFFFFF;
  --color-ink: #111111;
  --color-muted: #6B6B68;
  --color-rule: #E7E5DD;
  --color-track: #EEEDE6;
  --color-compare: #B7B5AC;
  --color-blue: #003262;
  --color-gold: #FDB515;

  --font-display: "Libre Caslon Display", Georgia, serif;
  --font-body: "Libre Caslon Text", Georgia, serif;
  --font-mono: "IBM Plex Mono", ui-monospace, monospace;
}

body {
  background: var(--color-ground);
  color: var(--color-ink);
  font-family: var(--font-body);
}

a {
  color: var(--color-ink);
  text-decoration: underline;
  text-decoration-color: var(--color-compare);
  text-underline-offset: 4px;
}
```

## 6. What a coding agent should refuse to do, even if asked in passing

- Add a colored heading, a colored link, or a colored nav item "for hierarchy." Use weight, size, and `--color-muted` for hierarchy instead.
- Use `--color-gold` as a background fill under text, or as text itself. It does not pass contrast and was never approved for that use.
- Add the two-dot mark more than once per page, or use it as a generic bullet/list-item glyph.
- Introduce a second accent color not in this table. If a new use case genuinely needs one, that's a design decision to raise, not a code decision to make.
- Reintroduce Inter, Roboto, Arial, or system-ui as a body/display font — the three families above are the whole type system.

## Before merging

- [ ] No text anywhere uses `--color-blue` or `--color-gold`
- [ ] The two-dot mark appears exactly once on the page
- [ ] Any gold dot beyond the mark flags one specific number, and there's at most one per page
- [ ] Body/headline/link color is `--color-ink` on `--color-ground` or `--color-surface`, nothing else
- [ ] Charts default to grayscale (`--color-ink`, `--color-muted`, `--color-track`, `--color-compare`) before any color is introduced
- [ ] Fonts load via the single Google Fonts request above, `display=swap` set, consistent with `website_style.md` §1's inline-critical-CSS budget
