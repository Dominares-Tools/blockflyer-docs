# Changelog

## 1.0.1

- Fixed: "fitted or refused" now covers width as well as height. Before, a footer whose web address and phone together were wider than the page was drawn with the two overlapping (and a QR code over the phone number), and a long web address in an event's `where` ran off the page edge, while the render still reported success. Now the page is measured for width after it is fitted: footer items drop to a second line when they cannot share one, then the footer's type shrinks (to 60% of the size asked for), and a line that still cannot fit is refused with a plain message and no file is left behind. Copy in an event, details, call-to-action, checklist, chips or subhead wraps inside its column even when it is one long unbroken string.
- Fixed: a model reply that was cut off (`max_tokens` on the Anthropic path, `finish_reason: length` on the OpenAI path) or refused is now reported as that, not fed to the repair loop as bad JSON three times. The Anthropic request allows 16000 tokens, because a model that thinks spends its thinking from the same budget.
- Corrected the notes on determinism: the same spec and kit give the same HTML and the same PNG bytes; the PDF matches apart from the creation and modification timestamps Chrome writes into it.
- The version check in the tests now also runs on Python 3.10.

## 1.0.0

First release.

- Four whole-page looks (poster, notice, big-date, ticket) and ten blocks (logo, hero, photo, chips, date, details, event, checklist, cta, footer). A model picks blocks and writes copy; it never emits HTML, CSS or colours.
- Per-block treatments with a house style per look, checked against what each look can draw. A treatment a look claims but does not draw fails the test suite.
- A brand kit folder (`brand.json`, `facts.json`, logo, photos) holds everything about the look; a kit whose own colours cannot be read is refused with the pair that fails.
- A repair loop: when a spec cannot be drawn, the reason goes back to the model, up to three tries. Claimed sources (a location or an event from the kit's facts) are verified against what the flyer prints.
- Optional palette and scale dials, limited to the kit's own colours and a closed set of steps, with a legibility check at the smallest size a wordy flyer shrinks to.
- One deterministic page: headlines and the whole stack are fitted in the page once its fonts have loaded, then drawn to PNG and a one-page US Letter PDF by headless Chrome or Chromium. Same spec and kit, same HTML and PNG bytes (the PDF matches apart from its timestamps).
- Command line: `demo`, `render`, `new`, `prompt`, `init-kit`, `check-kit`, `looks`. Model adapters for OpenAI-compatible and Anthropic-compatible endpoints, standard library only.
- Barlow (SIL Open Font License) is bundled; a kit can supply its own fonts.
