# 决策时机工具 — Decision-Timing Tool (Pilot)

An I-Ching-informed decision-timing tool. **This pilot covers only the first
layer**: a Four Pillars (八字) engine for a person, and the same engine
applied to the current moment (环境/当下). The hexagram/timing layer (I-Ching
proper, judging early- vs late-stage of a matter) is not built yet.

Pure client-side HTML/JS, no build step, no server, no network dependency.
Open `index.html` directly in a browser.

## What it computes

- **年柱 Year pillar** — anchored to the well-known 1984 = 甲子 cycle,
  flipping at 立春 (Lichun), not the Gregorian or lunar new year.
- **月柱 Month pillar** — branch from the Sun's apparent ecliptic longitude
  crossing the 12 "节" boundaries (315°, 345°, 15°, ... every 30°); stem via
  the classical 五虎遁 (Wu Hu Dun) rule keyed off the year stem.
- **日柱 Day pillar** — from a 60-cycle offset applied to the Julian Day
  Number of the local civil date (with the 晚子时 convention: 23:00-23:59
  counts as the next day).
- **时柱 Hour pillar** — branch from the two-hour 时辰 period; stem via the
  classical 五鼠遁 (Wu Shu Dun) rule keyed off the day stem.
- **五行 balance** — a simplified tally of the 8 characters' elements
  relative to the Day Master (peer/resource/output/wealth/pressure), as a
  rough first read only.
- **节气 Current solar term** — computed the same way as the month
  boundary, shown for context.

Solar longitude uses Meeus's low-precision solar-position formula
(~0.01° accuracy, i.e. a few minutes of time) — see `js/astro.js`.

## Day-pillar offset: confirmed

`GANZHI_JD_OFFSET` in `js/bazi.js` (`49`) was cross-checked against an
online 黄历 for two dates 8 days apart (2026-08-24 → 庚午, 2026-09-01 →
戊寅) and matched both. A manual override control remains in the "校验"
panel as a fallback in case a future date ever looks wrong.

## Comparing a moment against a person

The "时刻 Moment" panel takes any date/time (defaults to now) and, once the
person's Four Pillars are computed, shows a composite score against them:
five-element support/drain relative to the Day Master (range −8..+8, see
[app.js](js/app.js)), plus 六合/六冲 (harmony/clash) between the moment's
day branch and the person's day branch (+2 / −3), and the moment's day
branch clashing the person's year branch (−2) — that last one is an
extension of 六冲, **not** the traditional 冲太岁/犯太岁, which is
specifically the current year's branch clashing a birth-year branch (an
annual event, not a daily one). Overall score range: roughly −13 to +10,
though real scores cluster far narrower — treat it as a relative ranking
within one scan, not an absolute scale. Of the three signals, day-branch
clash (六冲) is the most classically concrete and worth weighting most;
the element support/drain tally is the softest (no 月令/藏干 weighting).

The "寻找更好时机" panel scans forward from the Moment panel's date/time
(hourly or daily, bounded horizon) and ranks candidates by that same score,
with a one-click "使用 Use" to load a candidate back into the Moment panel.

**This score is illustrative, not canonical** — real 择日 (date selection)
weighs many more factors than can be reduced to one number. Treat the
ranking as a starting point for comparison, not a verdict.

## Other caveats

- Birth time is taken as local civil clock time, with no true-solar-time /
  longitude correction. This mostly matters for births within minutes of an
  hour-pillar boundary.
- The five-element "balance" is a simplified heuristic — real strength
  analysis also weighs 月令 (seasonal command), 藏干 (hidden stems in
  branches), 合冲刑害 (combinations/clashes), and 得根 (rootedness). Out of
  scope for this pilot.
- This whole system is a codified traditional heuristic, not an empirically
  validated predictive model — worth keeping explicit in the tool's framing
  as the I-Ching/timing layer gets added.

## Hexagram casting (梅花易数 时间起卦法)

The "起卦" panel casts a hexagram deterministically from the Moment panel's
date/time — no randomness, so re-casting for the same moment always gives
the same result (this was a deliberate choice over physical/random coin
casting; see conversation history for the tradeoff). Formula and constants
were verified end-to-end against the classical worked example (Shao
Kangjie's 观梅占: 辰年十二月十七日申时 → 泽火革之泽山咸), which confirmed
the trigram numbering (乾一兑二离三震四巽五坎六艮七坤八), the casting
arithmetic, and the 纳甲 (Na Jia) line-level stem/branch table together.

This needed a new **lunar (农历) calendar engine** (`js/lunar.js`) — the
casting formula uses lunar month/day numbers, not the solar-term-based BaZi
month. New-moon timing uses Meeus's ch.49 algorithm; leap-month detection
uses the classical "first zhongqi-less month is the leap month" rule,
compared at China civil-day (UTC+8) granularity. Verified against: the 2025
leap month (闰六月, spanning Jul25-Aug22), the 2026 Chinese New Year date
(Feb 17), and the two 2026-08-24/09-01 dates already confirmed for the day
pillar.

Each of the 6 lines gets its own stem-branch (via Na Jia), so once the
person's Four Pillars are computed, the hexagram panel checks every line
against their Day Master (support/drain) and day branch (合/冲) — the same
comparison logic used elsewhere, just applied per line instead of per
moment.

## 八宫 palace, 世/应, 六亲, and 用神 (decision-category selector)

Rather than hardcoding a 64-row palace lookup table, palace/世/应 are
derived from first principles: for each cast hexagram, XOR it against each
of the 8 "pure" hexagrams (a trigram doubled) and match the resulting
6-bit diff against 8 canonical patterns (孤本宫/一世.../五世/游魂/归魂).
Verified against the classical 乾宫 worked examples (乾为天→本宫世6,
天风姤→一世世1, 火地晋→游魂世4, 火天大有→归魂世3) and against search
results confirming 世应 are always 3 lines apart and "游四归三"
(游魂世四，归魂世三). All 64 trigram-pair combinations resolve to exactly
one (palace, 世/应) pair with no ambiguity — checked exhaustively in
testing.

六亲 per line comes from the same generate/overcome classification used
for BaZi Day Master comparisons elsewhere, just applied to the hexagram's
palace element instead (生我=父母, 我生=子孙, 克我=官鬼, 我克=妻财,
同我=兄弟 — verified via search) — reuses `classifyElementsRelativeTo`
directly, no new logic.

The "决定类型" selector picks the 用神 (key line to focus on) by decision
category: **self** (世爻 — "will my venture succeed", the default fit for
a project) or **wealth**/**authority** (the 妻财/官鬼 line — fits questions
about obtaining a specific kind of outcome, like a stock's profit). For
each 用神 line, the "用神分析" section shows its relation to both the
casting moment's own day pillar (日辰 — the classically central check,
comparing the hexagram to *today*) and the person's day pillar (our
personalization extension, comparing it to *them*).

**Known gap:** 用神多现 (when more than one line carries the needed 六亲)
just lists all matches rather than applying the further classical
tie-breaking rules.

## 64-hexagram names

`HEXAGRAM_NAMES` in `js/hexagram.js` gives every hexagram its proper name
(e.g. "泽火革" instead of "泽火 (兑上离下)"), sourced from Wikipedia's "List
of hexagrams of the I Ching" (King Wen order) and cross-checked against the
classical worked example (泽火革/泽山咸) and the walkthrough's 大有/睽/萃/困
— all matched exactly, and all 64 (lower, upper) combinations resolve.

## 决定类型 categories

Beyond self/wealth/authority, added **property/renovation/documents**
(父母 — this is the one a renovation project should actually use, not
"self"), **children/health-recovery** (子孙), **illness/lawsuit/danger**
(官鬼 — same 用神 as authority, separate label because the real-life
context is different), and **friends/partners/competitors** (兄弟).
Marriage-type questions (which classically use 妻财 or 官鬼 depending on
the querent's gender) are deliberately not added yet — out of scope for
now, not forgotten.

## Net Read (synthesized reading with itemized reasoning)

Added because a raw table of pillars/lines/relations doesn't answer "so
what" — but this is explicitly **not** a classical formula. It's a
composite I built for showing the *reasoning*, so it can be learned from
and second-guessed rather than trusted as a verdict:
- A plain-language narrative: present hexagram + title, the moving line's
  stage, the future hexagram + title, and whether the 用神 itself is the
  moving line (a more direct signal) or something else is what's changing.
- An itemized signal list — each classical signal (用神 vs. 日辰, vs. the
  person's day pillar, 合/冲, the moment's five-element balance vs. the
  person) shown with its individual point contribution and *why*, not
  hidden inside one number.
- A total and a leaning (favorable/cautious/mixed). The signals themselves
  are classical; the point weights used to add them up are illustrative,
  same spirit as the moment-vs-person score elsewhere in this tool.

Ran this end-to-end against two real decisions (a GLD buy, a renovation
project) with a placeholder person — see conversation history for the full
walkthrough. It surfaced one real bug: the day/person relation labels were
reusing the same 六亲 words that label a line's identity *inside* the
hexagram, which is a different comparison — fixed by introducing
`RELATION_LABEL` as a separate vocabulary from `LIUQIN_LABEL`.

## Publishing a copy

`js/*.js` + `index.html` + `style.css` stay as the working source — don't
edit `dist/index.html` directly, it's generated. To get a single
self-contained file (e.g. to paste into GitHub's web "create new file" box
without setting up a repo/CI yet):

```
node build/bundle.js
```

This writes `dist/index.html` with `style.css` and all of `js/*.js` inlined.
Re-run it after any change to those files before copying `dist/index.html`
elsewhere. This is a stopgap for while the tool is still in progress —
revisit with a proper multi-file GitHub Pages setup once it's more settled.

## Next steps

1. Implement 用神多现 tie-breaking.
2. Revisit whether to offer real randomness (coin-cast) as an alternative
   to the deterministic time-based method — open question, not decided.
3. Try the tool with a real person's birth data instead of a placeholder.
4. Before sharing this beyond personal use (e.g. with Broadcom colleagues):
   write a plain-language "how to read this" primer for people with zero
   I-Ching background — what BaZi/a hexagram *are*, glosses next to the
   remaining jargon (用神/六亲/世应 don't have any yet, unlike 合/冲), and
   an explicit up-front note that this is a heuristic/traditional
   framework, not a validated method. Documentation/UX work, not logic —
   do it once the tool itself has settled down.
