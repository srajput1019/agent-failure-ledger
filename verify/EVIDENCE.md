# Agent Failure Ledger, issue 01 - verification pass

> **Status, 2 September 2026.** This document was written during the verification pass and was then
> itself audited. Four expert reviews found errors in it, which are corrected above and catalogued in
> [the adversarial review](review.html). Where this document and the review disagree, **the review wins.**
> Published file paths: traces are served under `data/`, screenshots as `.jpg`.


**Run date:** 2026-09-01
**Method:** every product page re-loaded and re-measured from scratch; the disputed add-to-cart
sessions re-run; screenshot + filtered accessibility tree captured at every step.
**Session type used below:** *automated* - Playwright Chromium 1280x860, headed with a persistent
profile unless a row says headless. This matters, and it is the single biggest correction in this pass.

**Rules kept, stated precisely (an earlier draft of this file overstated them):**
Add to cart only, never purchase. No CAPTCHA solving. No proxies. No user-agent spoofing.

Three things that an earlier version of this paragraph wrongly denied, all of which matter:

1. **This verification pass ran on a fully stock automated session.** `harness/verify.mjs` and
   `harness/atc.mjs` launch Playwright Chromium with no automation-hiding flags; there is no
   `AutomationControlled`, `blink-features` or webdriver suppression anywhere in `harness/`.
   That is stricter than the original 22 Aug runs, which `site/ledger.html` correctly discloses
   were launched with Chromium's AutomationControlled feature disabled, a partial disguise. The
   Nike modal and the SSENSE 403 both reproduced anyway, which strengthens those findings.
2. **The run did not always stop at a block.** On Amazon the page served a soft "Continue shopping"
   interstitial; `verify.mjs` clicks it and re-navigates to the target URL. That is pressing the
   page's own visible button, not solving a challenge, but it is a per-store assist that SSENSE
   never received, and the asymmetry has to be disclosed rather than described as "the run stopped."
3. **Amazon's robots.txt disallows `ClaudeBot` and `Claude-User` by name** (`data/amazon-audit.json`
   -> `protocol.robots.bots`). The session was an unidentified browser, not a declared crawler, but
   the tension is real and `site/ledger.html` already discloses it knowingly. Any published claim
   about Amazon completing the task has to carry it.

**Not measured, and load-bearing:** no IP, ASN, network type, or VPN state was recorded for any run.
Both anti-bot results are heavily IP-reputation driven, so this is the largest missing control in
the study. See `REVIEW-FINDINGS.md`.

---

## 1. The session-type problem (the root cause of the disagreement)

The independent check and the original runs are **both correct**. They differ because they used
different session types, and two of the six stores behave differently depending on that.

| Store | Normal human browser (independent check) | Automated session (this harness) |
|---|---|---|
| Nike | Add to Bag works, no modal | Refused. "We Couldn't Complete Your Request" |
| SSENSE | Product page loads and is readable | HTTP 403, Cloudflare "Performing security verification" |

Neither result is a general fact about the brand. Both are facts about *a session type on a date*.
Everything below is framed that way, which is the framing rule you set.

---

## 2. Fresh audit, all six pages

Measured 2026-09-01, automated headed session, one load each.

| Store | h1 | unnamed buttons | unnamed interactive controls | img missing alt | complete Product schema | Score |
|---|---|---|---|---|---|---|
| Glossier | 1 | 0 / 231 | 2 / 465 | 1 / 206 (100%) | yes | **A / 100** |
| Nike | 1 | 2 / 49 | 2 / 385 | 0 / 110 (100%) | yes | **A / 90** |
| Zara | 1 | 0 / 164 | 0 / 209 | 1 / 96 (99%) | yes | **A / 100** |
| Allbirds | 3 | 2 / 125 | 2 / 81 | 0 / 50 (100%) | yes | **C / 70** |
| Amazon | 10 | 48 / 227 | 20 / 444 | 53 / 248 (79%) | **no - zero JSON-LD** | **F / 0** |
| SSENSE | - | - | - | - | - | **not scored (HTTP 403)** |

Raw records: `verify/<store>-headed-audit.json`, screenshots `verify/<store>-headed.png`,
accessibility trees `verify/<store>-headed-tree.txt`.

"unnamed buttons" is the number the score uses (`button, [role=button]` with no accessible name).
"unnamed interactive controls" is the wider count across links, inputs, selects and ARIA widgets.
Both are given because they differ a lot on Amazon and the difference is itself the story.

---

## 3. One scoring system

Two rubrics were in play. The carousel used the harness rubric (35/35/15/15) which produced
95 / 98 / 100 / 95 / 37. Your PRD used a different one (40/20/20/20, graded A-F).

**Resolved in favour of the PRD rubric**, because it is the one your own brief defines, the one the
checker product would ship, and the one the independent check used. One band was ambiguous - the
brief said "some missing = 10 | many missing = 0" without defining "many" - so it is now explicit.
With that fixed, my measurements reproduce the independent check exactly: Allbirds C/70, Amazon F/0.

```
Agent-readability (open standards) - 100 points

Structured data   40   complete Product schema (Product/ProductGroup + price + availability) = 40
                       schema present but missing price or availability = 20
                       none = 0
Heading clarity   20   exactly one h1 = 20 | two = 10 | three or more, or zero = 0
Named controls    20   zero unnamed buttons = 20 | one to three = 10 | four or more = 0
Image alt text    20   >=95% of img carry an alt attribute = 20 | 80-94% = 10 | under 80% = 0

Grade   90+ A · 80-89 B · 70-79 C · 55-69 D · under 55 F
Blocked pages are not scored.
```

Stored at `verify/_scores.json`. **Every score in the carousel changes.**

---

## 4. Claim-by-claim corrections log

### CONFIRMED - keep, with dating

**C1. Nike refused at Add to Bag with a modal blaming browser extensions.**
Reproduced 2026-09-01 on both a headed and a headless automated session. Size M 9 / W 10.5 selected,
Add to Bag clicked, modal returned. Not reproducible on a normal human session (independent check).
- `verify/nike-atc-headed-03.png`, `verify/nike-atc-headless-03.png`
- `verify/nike-atc-headed.json` - `refusal.hits` matched "couldn't complete your request" and
  "disable any browser extensions"
- Earlier captures: `shots/nike-05.jpg` (2026-08-22), `shots/nike-rerun-03.jpg` (2026-08-27)

**C2. At the moment of refusal the filtered accessibility tree collapses from 211 lines to 6.**
This is new and it is the strongest evidence in the deck. The whole page disappears from the tree and
the agent's entire world becomes:
```
[e1] button "Close"
heading "We Couldn't Complete Your Request"
  text "We Couldn't Complete Your Request"
text "Close this tab, disable any browser extensions (such as coupon or promo code tools), and reopen nike.com."
[e2] link "View Bag"
```
211 lines before, 6 after (`treeLines` in the run record; `wc -l` reports 210/5 because neither file ends in a newline). Six lines carrying four things. These are lines of the harness's filtered rendering, not raw accessibility-tree nodes. Byte-identical headed and headless.
- `verify/nike-atc-headed-01-tree.txt` vs `verify/nike-atc-headed-03-tree.txt`

**C3. SSENSE never let the agent in, and the gate is not in the tree.**
2026-09-01, headed and headless: HTTP 403, Cloudflare managed challenge, Ray IDs a348136bb899eef5 and
a3481a9aea68c451. The tree contains the notice and two links - Cloudflare and Privacy. There is no
"verify you are human" control in the tree at all, so an accessibility-tree agent cannot see the thing
that would let it through. Even `/robots.txt` returns the challenge page.
- `verify/ssense-headed.png`, `verify/ssense-headed-tree.txt`, `verify/ssense-headless-tree.txt`
- Not reproducible on a normal human session (independent check). **Must be framed as automated-session.**

**C4. Amazon's size select has no accessible name and the listing preselects a variant.**
Confirmed fresh. Tree: `text "Size:"` then `[e165] combobox (unnamed) expanded=false`. The DOM select
`#native_dropdown_selected_size_name` has `selectedIndex: 40`, `selectedText: "10.5"` on load, while
the task asked for 9.
- `verify/amazon-headed-tree.txt:261-262`, `verify/amazon-headed-audit.json` → `metrics.selects[2]`
- The tree line quoted on slide 3 is verbatim authentic: `data/snapshots/amazon-snap-05.txt:278`

**C5. Glossier opens on a default shade, and after choosing another, two radios both report checked.**
Confirmed fresh. Before: `[e20] radio "Dark Brown variant option" checked=true`. After clicking Brown:
`[e20] radio "Dark Brown variant option" checked=true` **and** `[e23] radio "Brown variant option" checked=true`
- two members of one radio group both checked.
- `verify/glossier-atc-headed-01-tree.txt:47` vs `verify/glossier-atc-headed-02-tree.txt:47,50`

**C6. Allbirds never says which size is selected.**
Confirmed fresh. `[e30] button "Select size 9"` is byte-identical before and after being clicked - no
`checked`, `pressed` or `selected` state ever appears - even though the URL changes to
`?size=9&variant=33191200784464`, proving the click registered. The agent has no way to confirm it.
- `verify/allbirds-atc-headed-01-tree.txt:60` vs `verify/allbirds-atc-headed-02-tree.txt:60`

**C7. Zara's add-to-cart is the size picker; tapping a bare number is the commit.**
Confirmed fresh. Clicking "Add CHUNKY CHELSEA BOOTS" turns the button into
`button "Select a size CHUNKY CHELSEA BOOTS"` and exposes `button "6"` … `button "14"`. Nothing in
those names says that tapping one commits.
- `verify/zara-atc-headed-03-tree.txt:23,28-40`

**C8. The Glossier llms.txt quote on the closing slide.**
Live and unchanged 2026-09-01, HTTP 200, `powered-by: Shopify`. Text confirmed at line 20 of
`verify/glossier-llms.txt`.

---

### WRONG - must change

**W1. "Structured data: present on every page that reached the cart." - FALSE.**
Amazon reached the cart and has **zero** structured data: 0 JSON-LD blocks, 0 Product nodes, 0
microdata. The project's own `data/amazon-audit.json` recorded this on 2026-08-22
(`"productNodes": 0`, `"structuredData": 0`) so the slide contradicted its own evidence.
Confirmed again 2026-09-01. The independent check is right.
→ Rewrite. The honest version is stronger: the page with *no* structured data is the one that sold
the shoe, and the page with complete structured data is the one that refused.

**W2. Every score in the carousel.**
95 / 98 / 100 / 95 / 37 → **A100 / A90 / A100 / C70 / F0**, SSENSE not scored.
"A 98 was refused. A 37 sold the shoe." → "An A was refused. An F sold the shoe."

**W3. "Eleven h1s" on Amazon. - NOT A STABLE NUMBER.**
Measured 11 (2026-08-22, automated), 10 (2026-09-01, automated, stable across four consecutive
reloads in one session), 14 (independent check, normal session). It varies by session and over time,
not by reload. Swapping 11 for 14 would just be wrong on a different day.
→ Drop the exact count. Say one h1 on the refused page, ten on Amazon's the day it was measured, and
note that the count moves. `harness/_h1vol.mjs` holds the reload test.

**W4. "Control names: every control named on the refused page." - OVERSTATED, and my first correction of it was also wrong.**
Nike's DOM has 2 unnamed buttons. I first described them as the image carousel's arrows; the tree shows they sit under "You Might Also Like", so they are the recommendations rail. It scores 10/20 on that check, not
20/20. The point still stands - no unnamed control had anything to do with the refusal - but the
sentence as written is false.
→ Do NOT write "image-carousel arrows"; the tree shows they sit under "You Might Also Like". The deck now says only that the band counts unnamed buttons, which is what the evidence supports.

**W5. Zara "Pass" is a 2026-08-22 result and did not reproduce. (Amended 2 Sep: call it non-reproduction, not a refusal.)**
On 2026-09-01 the same flow returned a generic warning on an automated session, with `sizeSelected: false` in the run record, so a bot wall cannot be separated from a stock-out or a transient error: "WARNING - Your request could not be
completed at this time. Please try again later." Bag stayed at 0.
- `verify/zara-atc-headed-04.png`, `verify/zara-atc-headed.json`
→ This is the clearest possible argument for your framing rule. Either date the Zara row or mark it
mixed. It cannot be published as a flat "Pass."

---

### UNSUPPORTED - remove unless you re-shoot it

**U1. "One agent, one item per store, every step logged. Add to cart, not purchase."**
Still true of the original run, but the deck now spans three dates (22 Aug, 27 Aug, 1 Sep) and
several re-runs. The "one afternoon" line on slide 2 and slide 8 is no longer accurate.
→ Change to the date range, or scope the line to the original run explicitly.

**U2. Slide 6 before/after modal redesign.**
This is your design work, not a measured claim, and needs no evidence - but the "before" panel should
be labelled as a redraw of the 2026-09-01 capture rather than presented as a screenshot.

---

## 5. What did not change

The thesis survives intact, and is now better supported:

- A page that scored **A/90** on this rubric was refused. The page that scored **F/0** completed the task on 22 August. (The instrument question raised in the review is closed: see the withdrawal note in REVIEW-FINDINGS section C.)
- Not one of the four checks (structured data, headings, control names, alt text) touched the thing
  that actually blocked or steered the agent in any of the six runs.
- What decided the outcome was the session type, and what the page had already selected before the
  agent could choose.

## 6. Files

```
verify/_scores.json              final scores + the one rubric
verify/_summary.json             every raw metric, all six
verify/<store>-headed-audit.json fresh audit record
verify/<store>-headed.png        page screenshot
verify/<store>-headed-tree.txt   accessibility tree
verify/<store>-atc-*.json        add-to-cart session record
verify/<store>-atc-*-NN.png      per-step screenshot
verify/<store>-atc-*-NN-tree.txt per-step accessibility tree
harness/verify.mjs               the audit (both rubrics, block detection)
harness/atc.mjs                  the add-to-cart re-run
harness/_rescore.mjs             the single rubric, applied
harness/_h1vol.mjs               the h1 stability test
```
