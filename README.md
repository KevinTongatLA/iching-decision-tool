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

## Birth-time rectification (Option A)

For people who don't know their exact birth hour (common — most people
know their birthday, few know the time). Rather than generating 12
ambiguous candidate charts and leaving the user to guess (rejected —
doesn't help decision-making, just turns the tool into a guessing game),
this scores candidates against **when** (not what) the person remembers
major life turning points happening:

- `js/dayun.js` computes 大运 (decade luck pillars) — direction from
  gender + year-stem polarity (阳男阴女顺排/阴男阳女逆排), starting age
  from days-to-nearest-节 via 3天=1岁. Verified against a full worked
  example (1954 lunar 九月初七丑时, male → starts age 2, sequence
  甲戌/乙亥/丙子/丁丑/戊寅/己卯) — exact match, in `test/verify.js`.
- A "turbulence timeline" flags ages where something structural lines up:
  a new Da Yun starting, or Liu Nian (the annual pillar)/Da Yun clashing
  the Day pillar, the Da Yun pillar itself, or the Hour pillar.
- **Bug caught by testing, not by review:** the first version scored all
  12 candidate hours *identically*. Da Yun keys off the month pillar and a
  gender/year-based direction — neither depends on the hour — so within
  one day, Da Yun content barely varies by candidate. Fixed by adding
  Liu Nian/Da Yun clashes against the **Hour pillar** specifically (the
  hour position is classically 子女宫, the children/legacy palace) — that's
  the one pillar that actually changes per candidate. Guarded against
  regressing back to identical scores in `test/verify.js`.
- The UI (决定类型-style panel on the person form) lets the user enter a
  few remembered turning-point ages, ranks candidate hours by fit, and
  lets them adopt the best match with one click — resolving to a single
  chart, not leaving multiple candidates open.
- Once adopted, the person panel shows a clear "⏱ estimated via
  rectification" badge (which slot, what score, against which ages) rather
  than silently presenting an estimated hour as if it were a known fact.
- Every computed person also gets a **运势曲线 life-cycle bar chart** —
  each Da Yun decade's favorability, shown as rise/fall bars so the
  classic "rise, peak, decline, low point, rebound" shape is visible at a
  glance and can be sanity-checked against what the person already knows
  about their own life. Also a heuristic — see the caveat next to it.

### Correction: favorability depends on 身强/身弱, not "support = good"

Tested against a real person's life history (born 1960-12-02, afternoon,
female — PhD late 20s, career took off late 30s, career **peak** late
40s (co-chair of a national women's computing society + senior director),
**setback**/job loss mid-50s, recovery after) and the chart read "quite
the opposite" of reality. Root cause: the original version always treated
"elements that support the Day Master" as favorable — true only for a
*weak* Day Master. For a *strong* one it's backwards: 身强用泄用克，身弱用
生用扶 (a strong Day Master benefits from what drains/restrains it; a weak
one from what supports it) — arguably the single most foundational
judgment in BaZi, and the original version skipped it entirely.

Fixed by adding:
- **身强身弱 determination** — 得令 (is the month branch the Day Master's
  own season?), 得地 (is the Day Master's element rooted in the year/day/
  hour branches, via hidden stems?), 得势 (do the other stems support it?)
  — strong if 2 of 3 pass, per the verified classical rule.
- **地支藏干 (hidden stems)** — each branch's 1-3 hidden stems, weighted
  (100 / 70-30 / 60-30-10). Without these, a branch like 未 (surface
  element 土) or 巳 (surface 火) can't reveal the root/pressure they
  actually carry (未 hides 乙木; 巳 hides 庚金) — directly relevant to why
  her chart read wrong before.
- Both verified via search, cross-checking a corrupted source against an
  independent one before trusting either (the first hidden-stems source
  found gave 子 three stems and 卯 two, contradicting the most basic,
  universally-taught form of the table — rejected).

Result: her two peak-career decades went from flat 0 (wrong) to
positive, while her setback decade (already correctly flagged as the
worst point via a day-pillar clash) stayed the worst point — a real,
checked improvement, now guarded by an empirical regression test (labeled
as such — a real biography isn't a verified classical fact the way the
rest of this test suite's anchors are, but it's still worth protecting).

**Known remaining limitation:** 得令 here only credits the Day Master's
own season (e.g. Wood → 寅/卯). A broader classical reading also credits
the season that *generates* the Day Master's element (相, via the full
旺相休囚死 five-stage framework) — a second search surfaced this but gave
a definition I couldn't verify confidently in the time available, so it
was flagged rather than guessed into the code. Worth revisiting.

### Agreement indicator + two rejected hypotheses (from 4 real test cases)

Tested against 4 real people's charts (2 male, 2 female; 2 weak Day
Masters, 2 strong) before touching the formula again — see conversation
history for full detail. Two plausible-looking fixes were tried and
**rejected** because they helped one case while breaking another, which is
worth recording so they aren't re-tried without new evidence:

- **"Weight stems more than hidden branch stems"** — rejected. A search
  actually found the opposite classical emphasis ("得天干三比劫，不如得
  地支一本气根" — three stems of peer support isn't as good as one proper
  branch root); a stem without a matching root is considered "虚" (empty).
- **"Discount a stem's weight when it lacks a matching root (通根)"** —
  also plausible, also verified in principle, but checked against all 4
  cases: it would improve one case's mismatch while making another
  case's *correctly-favorable* period read as unfavorable. Two real cases,
  opposite effects — a sign of fitting noise, not finding a rule.

What shipped instead: an **agreement indicator** (✓/⚠/–) comparing the
element-based score's sign against the 十二长生 stage's verified
"four favorable / four adverse / four neutral" grouping
(长生冠带临官帝旺 favorable; 沐浴死墓绝 adverse; 衰病胎养 neutral). It
does **not** try to resolve disagreements — it just flags when the two
lenses point the same way (higher confidence) vs. opposite ways (⚠, use
extra judgment) vs. one/both being near-neutral. Checked against the 4
test cases: it correctly flagged the specific disagreements found by hand
(e.g. his 60-70 decade — score unfavorable, stage 临官/favorable, real
report "still working and growing" — and her 18-28 decade — score
unfavorable, stage 冠带/favorable, real report a top-tier college
graduation + immediate job).

### 十二长生 (Twelve Growth Stages) — a second, deliberately separate lens

Nian pointed out a *different* missing classical framework: the Day
Master's own birth-to-death vitality cycle across all 12 branches
(长生→沐浴→冠带→临官→帝旺→衰→病→死→墓→绝→胎→养, cycling back to birth —
the "seed to tree to death" shape). Verified via search (阳干长生起于
四隅寅申巳亥顺行，阴干长生起于四正子午卯酉逆行) and cross-checked exactly
against a fully-quoted 12-branch example for 甲.

Added as a **separate gray tag** under each Da Yun bar, not blended into
the favorability score — because checking it against the same real chart
showed it doesn't always agree with the element/hidden-stems analysis.
Her actual career-peak decade (38-48) reads 墓 ("tomb") under this lens,
one of the classically weaker stages, even though it scores favorable on
the element side (未's hidden 乙木 root). Reconciling two classical lenses
that disagree is real practitioner judgment, not something to fake as one
number — so both are shown side by side instead.

## UX: keeping panels in sync

An action in one panel that visibly changes a *different* panel needs
explicit feedback, or it reads as broken even when it isn't:

- The Scan and Rectification "使用/Use" buttons update the Moment panel
  (Scan) or the birth-time fields (Rectification) — both now scroll that
  target into view and give it a brief highlight flash
  (`scrollToAndFlash()` in `js/app.js`), since those panels can be
  off-screen from wherever you clicked "Use."
- Scan's result table now states "Scanned N candidate(s) (horizon=X),
  showing the top Y" — it always shows at most the top 10 by score (not
  everything scanned), which wasn't visible before.
- Casting a hexagram, then later changing the moment (typing a new
  date/time, "现在 Now," or either "Use" button), used to leave the
  hexagram silently showing the *old* moment. `computeMoment()` now
  re-runs `renderHexagram()` automatically if one is already on screen —
  same "refresh if already shown" pattern the 决定类型 category dropdown
  already used.

## 喜用神建议 (favorable-element suggestions)

Answers "given a fixed BaZi, does the tradition offer any way to act on
it?" — yes, classically: adjusting environment/direction/career toward a
person's favorable elements (喜用神) is a real, longstanding practice
(distinct from, but complementary to, the 择日/timing focus of the rest of
this tool). Shown on every computed person, right after the life-cycle
chart: favorable elements to lean toward and unfavorable ones to reduce
exposure to (not avoid entirely), each with direction/color/season.

Confidence varies by field, and the UI says so: direction, color, and
season are bedrock Wu Xing correspondences (as certain as "2020 is
庚子年," no verification needed). Career-theme suggestions are explicitly
flagged as the soft part — illustrative, not a fixed convention; different
sources give different lists. The panel also points back at the
Moment/Scan features as the more actionable lever — you can't change your
favorable elements, but 择日 (choosing when to act) is something this tool
already helps with directly.

## 环境周期 Environment Cycle — experimental, non-classical extension

Addresses the weakest of the original three pillars (person, time, and
*context* — the decision's environment was stuck at a fixed dropdown
category). Explicitly **not** claimed as classical I-Ching/BaZi — it's a
reasoned synthesis applying the tradition's own method of reading cycles
through Wu Xing (used classically for dynastic succession, 五德终始说, and
for the body in Chinese medicine) to a real-world cycle the user
describes, e.g. a market, an industry, a project's funding climate.

- **Off by default**, an explicit opt-in checkbox — with it off, the tool
  behaves exactly as before.
- **The user judges the phase, not the tool** — no attempt to infer market
  conditions from data; you supply your own read of a 5-phase cycle
  (萌芽/复苏→上升/扩张→转折/盘整→下降/收缩→低谷/蛰伏, mapped to
  木/火/土/金/水), and the tool only translates it into element language.
- **Multiple independent scopes, never averaged** — added specifically
  because a big-picture cycle and a narrower one inside it can be in
  different phases at once (the overall market expanding while one
  industry within it is late-stage contracting). Each row gets its own
  fit read against the person's favorable elements (reuses
  `favorableElementsFor` — no new classification logic, just a new
  reference point).
- Visibly labeled "EXPERIMENTAL — not classical I-Ching" in the UI, and
  kept as a separate block rather than merged into the Net Read's score —
  same principle as the 十二长生 agreement indicator: don't fake
  precision when combining things that don't have a verified combination
  rule.

## Publishing a copy — bundling in one file

`js/*.js` + `index.html` + `style.css` stay as the working source — don't
edit `dist/index.html` directly, it's generated. To get a single
self-contained file (e.g. to paste into GitHub's web "create new file" box
without setting up a repo/CI yet):

```
node build/bundle.js
```

This writes `dist/index.html` with `style.css` and all of `js/*.js` inlined.
Re-run it after any change to those files (including adding a new js/*.js
file — update the `jsFiles` list in `build/bundle.js` too) before copying
`dist/index.html` elsewhere. This is a stopgap for while the tool is still
in progress — revisit with a proper multi-file GitHub Pages setup once
it's more settled.

## Release notes

### 2026-08-25 to 2026-08-27

- Single-file bundler (`build/bundle.js` → `dist/index.html`) for manual
  publishing before this has its own repo/CI.
- **Birth-time rectification** (see that section above): a verified 大运
  engine, timing-correlation scoring against user-reported turning-point
  ages, gender + "不确定具体时辰" UI, and a "⏱ estimated via
  rectification" badge once a candidate is adopted — resolves to one
  chart rather than leaving several ambiguous candidates open.
- **运势曲线 life-cycle chart** — each Da Yun decade's favorability
  against the Day Master, shown as a bar chart on every computed person.
- Three real bugs caught by testing/user feedback before or shortly after
  shipping, all now guarded by regression tests where applicable:
  1. Rectification scored all 12 candidate hours identically (Da Yun
     doesn't depend on the hour) — fixed via Hour-pillar/子女宫 clash
     signals.
  2. Scan/Rectification "Use" buttons silently updated an off-screen
     panel, reading as broken.
  3. A cast hexagram could silently go stale after the moment changed.
- Roadmap note added: a beginner-facing primer is needed before sharing
  this beyond personal use (e.g. with Broadcom colleagues) — not started.

### 2026-08-27 (continued): body-strength correction, 十二长生, and the Environment Cycle extension

Prompted by testing against 4 real people's charts and biographies (see
sections above for full detail):

- **Fixed a real, meaningful bug**: Da Yun favorability was scoring
  "supportive of the Day Master" as always favorable — backwards for a
  strong Day Master. Added verified 身强身弱 determination (得令/得地/得势)
  and 地支藏干 (hidden stems); a real person's flat-zero career-peak
  decades moved to correctly positive.
- **Two plausible reweighting fixes tried and explicitly rejected** after
  checking them against all 4 test cases — documented so they aren't
  re-tried on the same evidence (see "Agreement indicator" section above).
- **十二长生** (the Day Master's own birth-to-death vitality cycle) added
  as a second, deliberately separate lens, plus an **agreement indicator**
  (✓/⚠/–) showing when it agrees or conflicts with the element-based
  score — no attempt to force them into one number.
- **喜用神建议 panel** — direction/color/season/career suggestions per
  person's favorable elements, answering "does the tradition offer any way
  to act on a fixed BaZi" (yes, classically — environment/direction
  adjustment, alongside 择日 timing, which this tool already does).
- **环境周期 Environment Cycle** — an explicitly experimental,
  non-classical extension mapping a user-judged real-world cycle (market,
  industry, project) onto Wu Xing, compared against the person's favorable
  elements. Off by default; supports multiple independent scopes so a
  big-picture cycle and a narrower one inside it can disagree without
  being averaged away.
- The 决定类型 (decision-type) dropdown moved from the hexagram panel to
  the top of the Environment panel, so choosing it primes which
  environment scopes actually make sense to describe.

## Next steps

1. Implement 用神多现 tie-breaking.
2. **应期 (timing prediction)** — the original motivating idea ("not just
   favorable, but *when*") still isn't built. `js/dayun.js`'s Liu
   Nian/clash-timeline machinery (built for rectification) is likely
   reusable groundwork here.
3. Revisit whether to offer real randomness (coin-cast) as an alternative
   to the deterministic time-based method — open question, not decided.
4. The broader 得令 reading (旺相休囚死, crediting the season that
   *generates* the Day Master's element, not just its own season) —
   surfaced by search but not verified confidently enough to implement.
5. If pursuing more empirical validation: published classical case
   compendia (滴天髓, 子平真诠, 穷通宝鉴) would give denser, larger-N
   material than one-off personal biographies — worth researching whether
   a usable digitized source exists, rather than continuing to test one
   person at a time. Real, tested against 4 people so far — see the
   "Correction" and "Agreement indicator" sections above for what that
   surfaced (one real bug fixed, two plausible fixes rejected).
6. Before sharing this beyond personal use (e.g. with Broadcom colleagues):
   write a plain-language "how to read this" primer for people with zero
   I-Ching background — what BaZi/a hexagram *are*, glosses next to the
   remaining jargon (用神/六亲/世应 don't have any yet, unlike 合/冲), and
   an explicit up-front note that this is a heuristic/traditional
   framework, not a validated method. Documentation/UX work, not logic —
   do it once the tool itself has settled down.
