# Adversarial review of the corrected carousel, 2026-09-01

Three independent reviewers were run against the corrected deck: a hostile fact-checker, a
legal/reputational reviewer, and a research-methods critic. Every finding below was then
**re-verified against the raw files by hand** before being accepted. Where a reviewer was wrong,
that is recorded too.

**Verdict: do not publish yet.** The underlying observations are sound and well evidenced. The
scoring layer and one causal sentence are not.

---

## A. Confirmed errors in the published copy

Each verified directly. These are wrong as written and cannot ship.

**A1. "the accessibility tree collapsed from 210 nodes to five."** Both numbers wrong, and "nodes"
is the wrong word.
- `verify/nike-atc-headed.json` records `treeLines: 211` -> `treeLines: 6`.
- The files are 211 and 6 lines. `wc -l` reports 210 and 5 only because neither file ends in a
  newline. I took the numbers from `wc -l`. That is my error.
- These are lines of the harness's *filtered* rendering (10 interactive + 6 display roles), not
  accessibility-tree nodes. The real `getFullAXTree` node count is much larger in both states.
- The copy then enumerates four things while saying five.
- Correct: "from 211 filtered tree lines to 6." The observation is real and reproduced byte
  identically headed and headless.

**A2. "Two unnamed controls on the refused page, both image-carousel arrows."** False.
`verify/nike-headed-tree.txt` contains 7 unnamed nodes, 5 of them actionable:
`[e90] button (unnamed) disabled`, `[e91] button (unnamed)`, `[e92]`, `[e93]`, `[e94] link (unnamed)`.
They sit under `heading "You Might Also Like" h3` - they are the **recommendations** rail, not the
product image carousel. The "2" is a DOM-only count. See section C, this is the big one.

**A3. "Every image on the refused page carries alt text."** The code measures attribute *presence*;
`harness/audit.mjs` says so outright: `alt="" counts as compliant`. `traces/nike-audit.json` records
`emptyAltDecorative: 6`. Six Nike images carry an empty alt. "Carries alt text" is false for those.

**A4. "Same modal headed and headless, on three separate dates."** Headless ran on **one** date.
22 Aug and 27 Aug were both headed (`browser` field in both traces). Only 1 Sep was headless.
Correct: "three dates headed, plus a headless run on the third."

**A5. "A fifth of Amazon's do not."** 53/248 = 21.4%, so "a fifth" rounds the wrong way. More
importantly the denominator includes ad rails, recommendation carousels and offscreen lazy images.
Within `<main>`, visible: **3 of 41, about 93% carry alt.** The harness computed that fair number
(`mainNoAltAttribute: 3`, `visibleInMain: 41`) and the slide published the unflattering one.
Worse, the denominator is a function of the harness: `verify.mjs` scrolls exactly 2700px, so the
image count - and therefore Amazon's score band - depends on a loop count.

**A6. "One turns Add to Bag into the size picker."** That sentence is about Zara. "Add to Bag" is
**Nike's** button label. Zara's is `Add CHUNKY CHELSEA BOOTS`.

**A7. "SSENSE returned 403 to every automated session."** 403 is recorded for two sessions, both
1 Sep. The 22 and 27 Aug runs record a Turnstile interstitial with no HTTP status captured.
"Every" covers four sessions; two have the number.

**A8. Slide 3 "re-confirmed 1 Sep."** The quoted tree line is verbatim from 22 Aug
(`traces/amazon-snap-05.txt:278`, including `value="10.5"`). On 1 Sep the tree reads
`[e165] combobox (unnamed) expanded=false` with **no `value="10.5"`**. What re-confirmed on 1 Sep
was the DOM state (`selectedIndex: 40, selectedText: "10.5"`), not the tree line.

**A9. `_summary.json` contradicted `_scores.json`.** Amazon was `F/10` in one and `F/0` in the other,
because `verify.mjs` still carries the superseded alt band. **Fixed** - `_summary.json` regenerated
from `finalScore`, with `_rescore.mjs` named as canonical.

**A10. `EVIDENCE.md` stated integrity rules that the traces contradict.** **Fixed** - see the
corrected "Rules kept" block. The original 22 Aug runs used an automation disguise (disclosed
correctly on `site/ledger.html`, denied by `README.md` and my first draft), the Amazon run clicked
through a soft interstitial, and Amazon's robots.txt disallows ClaudeBot by name.

**A11. `site/ledger.html` dated the re-runs "23 August"; the traces say 27 August.** **Fixed** at
source in `site/build.mjs`, ledger rebuilt.

---

## B. The claim with no evidence at all

**The human comparison.** The post says it six times, and slide 1 carries it:
> "A person clicking the same button in a normal browser does not. That is the finding: the refusal
> is about the session, not the shopper."

There is no artifact. `grep -rl "independent check"` returns exactly one file: `EVIDENCE.md`. No
screenshot, no date, no browser, no operator, no location, no log. Every file in `verify/` is
`-headed` or `-headless`, and `verify.mjs` has only those two branches.

And headed vs headless were **byte identical** - I diffed them. So the one session variable actually
manipulated was ruled out as the discriminator, and the effect is attributed to a variable never
manipulated.

**Uncontrolled confounds, none recorded anywhere:** IP reputation and ASN, request volume
accumulated by the study itself across three dates from one machine, automation fingerprint
(`navigator.webdriver`, CDP attachment), cookie and profile age (the verify pass shares one profile
across six stores while `atc.mjs` uses a fresh profile per store), logged-in state, geography,
time of day.

This is the single sentence carrying the thesis and it is one arm of automated data against one arm
of hearsay. It either needs a captured human session on the same machine and network - about twenty
minutes of work - or it has to be downgraded from a finding to a hypothesis with the confounds
attached.

---

## C. The finding that changes the headline

**The study says it measures the accessibility tree. The scores measure the DOM. They disagree.**

`traces/*.json` states perception is "Chromium accessibility tree over CDP". But every scored number
comes from `page.evaluate()` over the DOM using a hand-rolled name approximation. Measured both ways:

| store | DOM `unnamedButtons` (scored) | AX tree unnamed, actionable (what the agent reads) |
|---|---|---|
| Glossier | 0 | 2 |
| Nike | **2** | **5** |
| Zara | 0 | 0 |
| Allbirds | 2 | 2 |
| Amazon | 48 | 22 |

Re-scoring Nike on its own stated instrument moves named controls from 10/20 to 0/20:

| store | DOM-scored (published) | AX-tree-scored |
|---|---|---|
| Glossier | A / 100 | A / 90 |
| Nike | **A / 90** | **B / 80** |
| Zara | A / 100 | A / 100 |
| Allbirds | C / 70 | C / 70 |
| Amazon | F / 0 | F / 0 |

**"An A was refused" becomes "a B was refused."** The gap that carries the thesis (80 vs 0) survives,
but the specific headline does not.

*Note: the methodology reviewer claimed Nike drops to C/70. That arithmetic is wrong -
40 + 20 + 0 + 20 = 80, which is a B. Verified.*

**A second, related problem.** The rubric's named-controls band is scoped to `button, [role=button]`.
Amazon's size selector - the centrepiece of slide 3 - is a `<select>`, so it contributes zero to the
score. The harness computes the correct wider number right beside it (`unnamedControls`) and discards
it. So slide 4's "not one of them touched what blocked or steered the agent" is partly an artifact of
a selector choice: had the band used the wider count, the named-controls check **would** have flagged
Amazon's unnamed size combobox, which is exactly the thing slide 3 says steered the agent.

Two of the six grades also sit exactly on a band boundary (Nike on 90, Allbirds on 70), and Amazon's
F/0 versus F/10 turns on four images out of 248 - a threshold set at 80% after the data was seen.

---

## D. Two things never re-verified

**D1. Amazon's add-to-cart was never re-run.** There is no `amazon-atc-*` in `verify/`. The sentence
"the page that scored an F completed the task" rests entirely on the 22 Aug run - the same vintage as
Zara's pass, which this very verification pass proved does **not** reproduce. Amazon is also the store
already documented as bot-blocking this harness on its search page. This is the run the conclusion
depends on and it is the one that was not repeated.

**D2. Extension state is never captured.** "My AI had no browser extensions" is true by construction
(no `--load-extension` anywhere, no Extensions directory) but nothing records it. For the headline of
the post, capture `chrome://extensions` or a `navigator` probe.

---

## E. Legal and framing risk

- **"The store blamed them anyway"** imputes intent to a corporate actor, and it is the headline.
  The literal version is stronger and unimpeachable: "The refusal told it to disable them."
- **Nike and SSENSE tiles carry undated anti-bot verdicts**, breaking the framing rule. Tiles are the
  most screenshot-able, most context-free asset in the deck. Add dates and "every automated session
  **I ran**."
- **The alt-text line names a company and a failing grade on an ADA-litigated metric**, generalised
  from one page load. Scope it to the page and the date.
- **Grades are per-page but labelled per-brand.** One product page each. The slide-2 kicker should
  read "six product pages, one per store."
- **Slide 8 quotes Glossier telling agents not to script the storefront**, which is what the project
  did. Glossier and Zara both publish a UCP endpoint the agent never used. Own it in a clause; it is
  the best line in the deck and it survives being owned.
- **No network, IP or VPN disclosure** anywhere. First thing a technical reader asks.
- **Crop the assets.** `site/shots/amazon-05.jpg` leaks a delivery ZIP; the Nike capture is a full
  PDP where only the modal and size grid are needed.

---

## F. What survives all three reviews untouched

These were attacked and held. They are the real findings:

- Nike's refusal reproduced on every automated session across three dates, headed and headless,
  byte identical.
- The tree collapse at the moment of refusal (with corrected numbers).
- Allbirds' size button byte identical before and after a click that demonstrably registered.
- Zara's add-to-cart becoming the size picker, with bare-number buttons.
- Amazon's size combobox unnamed in the tree, listing preselected to 10.5.
- Glossier opening on a default shade.
- Amazon carrying zero JSON-LD on both dates.
- All six published scores matching `_scores.json` exactly.
- Slide 3's quoted tree line and slide 8's llms.txt quote, both verbatim.

---

## G. Decisions that are yours, not mine

1. **Scoring instrument.** Keep DOM scoring (fast, but your own published tree file contradicts
   slide 4), switch to AX-tree scoring (honest, Nike becomes B/80, headline weakens), or publish the
   gap itself as the finding: *"the checker said 2 unnamed controls; the tree the agent reads said 7."*
   I recommend the third. It is a genuinely novel result, it is the study's own instrument catching
   its own scoring layer, and it is stronger than the line it replaces.
2. **The human session.** Run one yourself on the same machine and network, screenshot it, and the
   causal sentence becomes defensible. Otherwise it has to be downgraded.
3. **Amazon add-to-cart.** Re-run it, or stop claiming the F completed the task.
4. **Weaker Amazon preselect claim.** The search query was `under armour charged assert 10 men`. The
   "10" is the model name, but a skeptic will say the ranking matched on the numeral. Either re-run
   with a numeral-free query or preempt it in the copy.
