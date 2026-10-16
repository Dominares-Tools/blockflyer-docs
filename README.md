# blockflyer — Describe an event, get an on-brand flyer

**Paid tool: $149 once (plus sales tax where it applies), then a private GitHub repository you're invited to, with every update. Buy: https://dominares.org/tools/blockflyer**

Commercial license v1.3 — see https://dominares.org/license · Support: support@dominares.org

blockflyer takes a plain-English event description, has a model choose from ten blocks across four looks and write the copy, and draws a PNG and a one-page US Letter PDF from your brand kit with a deterministic renderer. A flyer is fitted to the page in height and width, or refused.

## Get it

Buy blockflyer at https://dominares.org/tools/blockflyer: one payment of $149, plus sales tax where it applies, and every future update. After checkout you are invited to a private GitHub repository with the source and every version published to it. The repository includes the Python package and the blockflyer command, a demo kit and four demo specs, and the full test suite. The Barlow font ships under the SIL Open Font License with its notice.

Requirements: Python 3.10 or newer (segno installs with it), and Chrome or Chromium to draw the PNG and PDF; without one the demo still writes the HTML. A model key only for the compose step.

This page documents version 1.0.1. Install commands and file paths in the documentation below run inside the private repository you are invited to; the source is not in this repository.

## Documentation

### What it does

A flyer is an ordered list of **blocks**. The model chooses which blocks the flyer is made of, what order they go in, and what they say. It never writes HTML, CSS, colours or sizes, so it cannot produce something off-brand: the worst it can do is pick the wrong blocks or write weak copy, and both are visible in a JSON file you can edit and re-render without asking again.

- **Ten blocks:** `logo`, `hero` (the two-line headline), `photo`, `chips` (three short selling points with meaning-matched icons), `date`, `details` (a place on the left; hours or a date on the right, as `right_lines`), `event`, `checklist`, `cta`, `footer` (web address, phone, scannable QR).
- **Four looks**, each a whole-page layout the model picks by name: `poster` (loud, general purpose), `notice` (quiet, centred, no chips), `big-date` (the date enormous in a top panel; needs a `date` block), `ticket` (framed and formal).
- **Treatments:** how one block sits on the page, for example a `hero` as `stacked`, `knockout`, `oversized` or `centred`, or a `checklist` as `ticks`, `numbers`, `rules` or `columns`. Leave one out and the look's own house style decides. `blockflyer looks` prints every look, block and treatment with what each one does.
- **A repair loop.** Everything the model returns is validated before anything is drawn. When a spec cannot be drawn, the reason goes back to the model and it fixes its own answer, up to three tries.
- **Verified facts.** When the model names a location or an event in `sources`, a location's address is checked against what the flyer prints and an event must exist in your kit's facts. A retracted location (`"active": false`) is never shown to the model and can never verify. A flyer that prints an address without naming it in `sources` is not checked.
- **One page, fitted.** Headlines are shrunk to their column and the whole stack is scaled to fit the page once the fonts have loaded. If the content cannot fit at 60% or above, the render is refused and no PNG or PDF is left behind, rather than printing a flyer with its footer cut off. Width is checked the same way: footer items that cannot share a line drop to a second one, then the footer type shrinks to 60% of the size asked for, and a line that still does not fit (a very long web address, say) is refused with no file left behind. Copy in the other blocks wraps inside its column, even a long unbroken string.
- **Deterministic.** The same spec and kit give the same HTML and the same PNG bytes on the same machine and browser. The PDF has the same content but Chrome writes the creation time into it, so its bytes differ between runs.

### Make your own

```bash
blockflyer init-kit mykit            # writes mykit/brand.json and mykit/facts.json
# edit them, drop a logo and photos in, then:
blockflyer check-kit mykit

export OPENAI_API_KEY=...            # or ANTHROPIC_API_KEY
blockflyer new "Bake sale Saturday 9am at the hall, free coffee, bring a friend" --kit mykit
```

`new` writes the spec it built (`out/<slug>.json`), then draws it. Tweak the wording in that file and draw it again without calling the model:

```bash
blockflyer render out/bake-sale.json --kit mykit
blockflyer render out/bake-sale.json --kit mykit --look ticket   # the same flyer in another look
```

#### The brand kit

```
mykit/
  brand.json      name, colours, optional logo, fonts and footer defaults
  facts.json      optional: contacts, locations and events a flyer may use
  logo.svg        optional (svg, png, jpg)
  photos/         optional photographs a photo block can name
```

`brand.json`:

```json
{
  "name": "Your Organisation",
  "tagline": "",
  "colors": {
    "ground": "#fbf8f1", "ink": "#1d2b3a", "accent": "#d2591f",
    "panel": "#1d2b3a", "on_panel": "#fbf8f1", "soft": "#efe6d6"
  },
  "logo": "logo.svg",
  "fonts": { "display": "fonts/Display.ttf", "body": "fonts/Body.ttf" },
  "footer": { "url": "example.org/events", "phone": "555-0100", "qr_url": "https://example.org" }
}
```

- The six colours are required (`#rrggbb`); any extra named colours become choices for the palette dial. Your colours are checked on load and a pair that cannot be read is refused by name: `ink` on `ground`, `on_panel` on `panel` and `ink` on `soft` need 4.5:1; `accent` on `ground` and on `panel`, and `ink` on `accent`, need 3:1 (the accent is only ever set large).
- With no `logo`, the `logo` block sets `name` (and `tagline`) as a wordmark.
- `fonts` is optional. Barlow, under the SIL Open Font License, is bundled; one file per role replaces it for every weight of that role (`.ttf`, `.otf`, `.woff`, `.woff2`).
- `footer` gives the footer block its defaults (it also accepts `qr_heading`, `qr_sub`, `qr_tag`). A footer with no web address, phone or QR link, from the block or the kit, is refused rather than drawn empty.
- A `photo` block names a file in `photos/` without its extension (`jpg`, `jpeg`, `png`, `webp`, `svg`; each under 8 MB). A name is never a path: anything with a separator or a leading dot is ignored. A photo block with no matching file draws the look's accent field.
- `facts.json` holds `contacts` (label to value), `locations` (`name`, `address`, `hours`, `phone`, optional `"active": false`) and `events` (`title`, `date_iso`, `weekday`, `start`, `city`, `url`). The model sees contacts, active locations and events from today on; a place named in your description moves its rows to the front.

#### Commands

| Command | What it does |
|---|---|
| `blockflyer demo [--all] [--html-only] [--scale N] [--out DIR]` | Draw the bundled sample. No model, no key. |
| `blockflyer render SPEC --kit DIR [--look NAME] [--no-pdf] [--html-only] [--scale N] [--out DIR]` | Draw a spec file. No model. |
| `blockflyer new DESCRIPTION... --kit DIR [--look NAME] [--provider auto\|openai\|anthropic] [--model NAME] [--specs DIR] [--overwrite] [render options]` | Ask the model, save the spec, draw it. Refuses to replace a spec of the same name unless you pass `--overwrite` (exit 3). |
| `blockflyer prompt DESCRIPTION... --kit DIR` | Print exactly what the model would be sent, without calling it. |
| `blockflyer init-kit DIR` / `check-kit DIR` | Write a starter kit; validate a kit and show what it holds. |
| `blockflyer looks` | List the looks, house styles, blocks and treatments. |

Exit codes: 0 done, 1 an error with a reason on stderr, 2 usage, 3 a spec of that name already exists.

#### Models

`new` reads `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` from the environment (`--provider auto` uses OpenAI's if both are set). `--model`, or `BLOCKFLYER_MODEL`, picks the model; the defaults are `gpt-5.4` and `claude-sonnet-5-5`. `OPENAI_BASE_URL` and `ANTHROPIC_BASE_URL` point the adapters at a compatible endpoint, and an empty value counts as unset. Reasoning-model families are sent the parameters they accept. Both adapters use the standard library only.

From Python, the composer takes any `ask(messages) -> str`:

```python
from blockflyer.brain import compose
from blockflyer.kit import load_kit
from blockflyer.output import render_spec

kit = load_kit("mykit")
spec = compose("Seed swap, Nov 21, 9am", ask=my_model_call, kit=kit)
render_spec(spec, "out", kit=kit)
```

#### The dials

A spec may set `"looseness": "loose"` and then a `palette` (which of the kit's colour names plays `ground`, `ink` and `accent`) and a `scale` (a step of 0.85, 1.0, 1.2 or 1.45 for `hero`, `chips`, `cta`, `date` or `footer`). Both are closed sets: no hex codes and no free numbers. A palette or scale that would print unreadably at the smallest size a wordy flyer shrinks to is refused before it renders. At the default `"brand"` looseness, naming either is refused.

### Limits

- Needs Chrome or Chromium for PNG and PDF (`BLOCKFLYER_CHROME` sets the path; the headless shell that Playwright downloads is also found). Nothing else is needed beyond Python 3.10+ and the one pure-Python dependency, `segno`, which draws the QR code.
- One page per flyer, at one aspect ratio (3:4). The PDF is US Letter only, with the flyer centred and scaled to fit a printable area.
- Four looks and ten blocks. A look cannot draw every block (`notice` has no chips; `big-date` and `ticket` have no chips or photo) and says so by name instead of dropping one. There are no free-form layouts, and the model cannot add a block type.
- It does not generate images or photographs. A `photo` block shows a file from your kit.
- The bundled font covers Latin scripts. Text in other scripts needs a kit font that has the glyphs.
- Copy fitting is by shrinking: a headline line is never wrapped past its column and the page never scales below 60%. A footer line (web address, phone) is never broken mid-word: it moves to a second line, shrinks, or is refused. Very long copy is refused, not squeezed.
- The model adapters are tested against a local stand-in server, which proves the request shapes, error handling and the full compose-and-repair chain; they have not been run against a live model API in this release.
- Output varies a little between Chrome versions and machines (font rasterisation); the HTML and PNG are byte-identical on the same machine and browser (the PDF differs only in its timestamps).

---

Docs mirror of the private repository Dominares-Tools/blockflyer; updated with each release.
