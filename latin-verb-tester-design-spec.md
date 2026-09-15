# Latin Verb Tester — Design Spec

**Purpose of this document:** a complete, implementation-ready spec for building a single-page web app ("Latin verb tester") that mirrors the aesthetic and interaction model of the reference "Greek verb tester" screenshots, adapted to Latin grammar, plus a second mode for principal-parts drilling. Intended reader: an LLM/dev (Opus) building the app in one pass — everything needed to avoid follow-up questions should be here.

---

## 1. Overview

Two modes, reachable from a top-level **MENU**:

1. **Conjugation Tester** — mirrors the reference tool exactly: explore/review paradigms, get quizzed on parsing a form (PARSE), get quizzed on producing a form (TYPE), see cumulative RESULTS. Covers six Latin verbs: **amō** (1st conj.), **moneō** (2nd conj.), **mittō** (3rd conj.), **audiō** (4th conj.), **fugiō** (3rd conj. **-iō**/mixed), and **sum** (irregular).
2. **Principal Parts Drill** — a separate quiz mode that tests recall of the four principal parts for the 100 most important Latin verbs (list and sourcing in §7).

Both modes share the same visual language, header, and scoring conventions.

---

## 2. Visual design system (match the reference tool)

**Overall feel:** plain, functional, textbook-like. No shadows, no gradients, no icons. Everything is rectangles with 1–2px borders. This is a study tool, not a marketing page — the reference screenshots should be treated as the literal target, not "inspiration."

**Layout container:** centered column, max-width ~700–780px, generous white space, white background throughout.

**Typography:**
- System sans-serif (e.g. `-apple-system, "Segoe UI", Helvetica, Arial, sans-serif`) for all UI chrome and prose.
- App title ("Latin verb tester") set larger (~28px), with a thin underline rule beneath it (`border-bottom: 1px solid #999`, title sits just above it) — matches the underlined "Greek verb tester" header.
- Latin word display (the word being parsed/typed, and all conjugated forms) can use the same sans-serif; no need for a special serif/Latin font. Macron-bearing characters (ā ē ī ō ū and capitals) must render correctly — use a font stack that includes broad Unicode Latin Extended-A coverage (system default fonts all do).

**Color palette (approximate hex, match by eye against screenshots, not pixel-exact):**
- Page background: `#ffffff`
- Header buttons (MENU, HELP): light gray fill `#e4e4e4`, 1px `#999` border, black text, small rounded corners (~4px).
- Tab bar (SET-UP / REVIEW / PARSE / TYPE / RESULTS): light sage-green fill `#cfe3cf`, 1px `#7fa87f`-ish border, black text. The **active** tab gets an orange/red border (`#d9622b` or similar) instead of green, fill stays pale green (or goes white) — replicate the "REVIEW" and "PARSE" tabs' active-state look in the screenshots (colored outline, not a filled background change).
- Selector grid tiles (tense / voice / mood / verb pickers): unselected = white fill, black 1px border, black text. Selected = solid blue fill `#3f6fb4` (or close), white text, no border needed (fill itself reads as "on").
- Data table (person labels + forms): left column ("1st s", "2nd s", …) light blue-gray fill `#eef2fb` with blue text `#2f5fa8`; right column (the forms) white fill, black border, black text.
- Incorrect-answer highlight: solid red/orange fill `#e0553f` behind the wrong cell, white text, with a small "✗" to its right in the same red tone. (See screenshot: the wrong `τιμώμεθα` cell.)
- Feedback line under a PARSE/TYPE answer: blue text for the parse description ("pres pass indic 1st plur"), red "✗" mark, then a second blue line "You could have chosen: …" — replicate this two-line feedback pattern in Latin (see §5.3).
- Score readout "0/0" top-right of the tab bar: plain black text, same size as tab labels.

**Buttons general shape:** all clickable "chips" (tabs, selector tiles, MENU/HELP, NEXT) are simple bordered rectangles with padding ~8–14px horizontal, ~6–10px vertical, no drop shadow, instant (non-animated or near-instant, <100ms) color change on state change.

**Responsive behavior:** reference screenshots show both a wider desktop layout and a narrower mobile-ish layout (image 3 vs image 6) — same component set, just the outer container narrows and the tile grid wraps naturally (`flex-wrap: wrap` on all tile rows). Build mobile-first with this wrapping and it will scale up cleanly.

---

## 3. App shell & navigation

- Top bar: **Title** (left) — "Latin verb tester" in Conjugation Tester mode, "Latin principal parts drill" in Drill mode — with **MENU** and **HELP** buttons (right). MENU switches between the two modes (and returns to each mode's SET-UP screen). HELP opens a short static explainer overlay/panel (plain text, no need to over-design — a simple modal or inline expand is fine) describing what each tab does and how macron-optional typing works.
- Below that, in Conjugation Tester mode: the 5-tab bar **SET-UP | REVIEW | PARSE | TYPE | RESULTS**, plus the running score `n/m` at the right of the tab bar (correct/attempted, resets only when the person hits RESET on SET-UP or explicitly clears results — never resets silently).
- Drill mode has its own simpler 3-tab bar: **SET-UP | DRILL | RESULTS** (no REVIEW/PARSE distinction needed — see §7).

State that must persist as the user moves between tabs within a mode:
- Current SET-UP selections (which verbs/tenses/voices/moods are "in scope" for quizzing).
- Score counters.
- The RESULTS log (every attempted question this session, right or wrong).
- Whatever question is "in progress" on PARSE/TYPE, so switching tabs and coming back doesn't lose it (switching tabs should not silently advance/skip a question).

No backend needed — everything is client-side state (in-memory JS state for the session; do **not** use localStorage/sessionStorage per this environment's artifact constraints — plain in-memory React/JS state is fine and is the intended scope, this is a single-session study tool).

---

## 4. Screen-by-screen spec

### 4.1 SET-UP

Purpose: choose which verbs and which grammatical categories are in-scope for REVIEW/PARSE/TYPE quizzing.

Layout (top to bottom):
1. Optional small illustrative graphic under the header (the reference tool shows a red-figure vase painting of a man writing, for flavor). For the Latin version, use a simple, generic, non-copyrighted illustrative element instead — e.g. a plain decorative rule, a laurel-branch line-drawing, or omit it entirely. **Do not** attempt to reproduce or closely imitate any specific real artwork, monument, or museum photo — keep this element generic/decorative or skip it.
2. "Choose verbs and parts to test." with **ALL** / **NONE** quick-select buttons at the right, exactly as in the reference.
3. Tile rows, each a `flex-wrap` group of selectable tiles (multi-select, toggle on click, blue-when-selected as in §2):
   - **Tense row:** `pres` `impf` `fut` `perf` `plup` `futperf` (six tenses — Latin has no aorist; add **futperf** since Latin's synthetic future-perfect is a real, commonly taught tense the Greek tool's aorist slot can be swapped for).
   - **Voice row:** `act` `pass` (Latin has no middle voice; two tiles instead of the Greek tool's three).
   - **Mood row:** `indic` `subj` `imper` `infin` `ptcp` (five — Latin has no optative; supine is deliberately **not** a tile here, see §5.5 note on optional extension).
   - **Verb row:** `amō` `moneō` `mittō` `audiō` `fugiō` `sum` (six tiles, one per verb in scope).
4. Selecting tiles only sets what is *eligible* for random selection in PARSE/TYPE and what displays in REVIEW; it should never itself trigger navigation.

Validity note the app must enforce (see §5.5 for the full gap table): if the user's tile selection makes a *combination* impossible for every verb currently selected (e.g., `futperf` + `subj`, which doesn't exist in Latin at all, for any verb), PARSE/TYPE question generation must simply never draw that combination — it should not appear as a possible answer or as a question. This is enforced in the question generator, not by disabling tiles (tiles stay clickable/selectable exactly as in the reference tool; impossible combinations are silently excluded when building question pools).

### 4.2 REVIEW

Purpose: browse the full paradigm for a chosen tense/voice/mood/verb combination — pure reference, no quizzing.

Layout: identical structure to the reference screenshot —
1. Tense/voice/mood/verb tile rows at the top (single-select this time — clicking a tile in a row selects it and deselects the previous selection in that row; exactly one tense, one voice, one mood, one verb highlighted blue at all times once the screen has initialized with a sensible default, e.g. `pres / act / indic / amō`).
2. Below the tiles, a two-column table:
   - **Finite moods (indic/subj/imper):** left column = `1st s`, `2nd s`, `3rd s`, `1st p`, `2nd p`, `3rd p` (light-blue fill); right column = the conjugated form for each, in black text, with macrons (light border box). Imperative only has 2nd and 3rd person forms in Latin (2nd s, 2nd p in the present; 2nd s, 2nd p, 3rd s, 3rd p in the future) — rows with no valid form for a given person should either be omitted from the table or show an em dash `—`; omitting the row is cleaner and preferred.
   - **Infinitive:** collapses to a single row (no person/number) — label the row `infin` in the left column, form on the right, e.g. `amāre`, `amātus esse`.
   - **Participle:** Latin participles decline by case/gender/number, not person — do not attempt a person-based table for `ptcp`. Instead show, for the selected tense/voice, the **nominative singular masculine** form as the headline answer (e.g. `amāns, amantis` for present active, showing genitive too since 3rd-declension participles are usually cited with the genitive stem), with a small note under the table: "shown as nom. sg. masc. (participles decline like 1st/2nd or 3rd declension adjectives)." Only four participles exist per verb (present active, perfect passive, future active, future passive/gerundive) — see the gap table in §5.5 for which tense/voice combinations are real for `ptcp`.
3. When the selected tense/voice/mood combination doesn't exist for the selected verb (per the gap table, §5.5), replace the table with a short plain-text note, e.g. *"Latin has no future subjunctive."* or *"sum has no passive forms."* — do not show an empty or broken table.

### 4.3 PARSE

Purpose: given a Latin form, the user picks tense/voice/mood (single-select tile rows, same visual language as REVIEW) and then a person/number cell marked `???`, mirroring the reference exactly.

Flow (matches the four PARSE screenshots precisely):
1. A boxed word is shown at the top (the prompt form, e.g. `mīsissēs`).
2. Prompt text: "Where would you find this Latin verb form? Pick tense, voice, and mood, then a **???** box." — the word "**???**" rendered in the same orange/red accent used for the placeholder cells below.
3. Tense/voice/mood tile rows (single-select each, as in REVIEW) let the user build a candidate parse. As soon as all three are chosen, the answer table appears with every person/number cell showing `???` in orange (matches screenshot 6) instead of real forms — this signals "guess is not yet locked in, pick a cell."
4. User clicks one `???` cell (their claimed person/number). The app immediately reveals **all** the real forms for the *actual* correct tense/voice/mood/person/number of the prompt word (not just the one the user picked) filled into the table — i.e., the table converts from all-`???` to all-real-forms the moment an answer is submitted, exactly like screenshot 2, which shows the full paraphrased table (`τιμῶμαι`, `τιμᾷ`, `τιμᾶται`, …) after a wrong guess.
5. The cell corresponding to the user's guess is highlighted red with a trailing ✗ if wrong (screenshot 2's `τιμώμεθα` cell), or presumably highlighted green with a ✓ if right (not shown in the reference screenshots but implied by symmetry — implement both states).
6. Below the table, two feedback lines in blue, matching the reference wording pattern exactly, translated to Latin grammar terms:
   - `pres pass indic 1st plur ✗` → i.e. the tense/voice/mood/person-number the user actually selected, with the ✗/✓ mark.
   - `You could have chosen: <correct answer>` → the true parse of the prompt word, e.g. `plup act subj 2nd sing`.
7. A **NEXT** button (bottom, same bordered-rectangle style) advances to a new random prompt word drawn from the verbs/tenses/voices/moods currently selected on SET-UP.
8. Score `n/m` in the tab bar increments on every submitted guess (m = attempts, n = correct).

**Ambiguity handling:** many Latin forms are genuinely ambiguous (e.g. `amat` could only be one thing, but `amāvissēs`/`-ēs` style syncope, or a 1st-declension-looking ablative-vs-nominative issue doesn't really arise for verbs — the real ambiguity in Latin is things like present active infinitive vs. imperative looking identical only in some 3rd-conj forms, or `-ēre` alternative perfect 3rd-plural ending `amāvēre` = `amāvērunt`). Where a form is genuinely ambiguous between two valid parses, accept either as correct and show both under "You could have chosen:" separated by `/`.

### 4.4 TYPE

Purpose: given a verb + tense/voice/mood/person-number prompt (chosen by the app, not the user), the user types the form.

Layout (matches the screenshot):
1. Prompt line: `Type this form of `**amō**` ` with the verb name shown in a shaded pill (light green background, matching the reference's `παύω` pill).
2. Two lines of orange/brown prompt text beneath: the tense+voice+mood phrase (e.g. `pluperfect active subjunctive`) and the person/number phrase (e.g. `2nd person singular`).
3. Two side-by-side text input boxes, left one focused/outlined blue by default. **Two boxes, not one**, because several Latin forms legitimately have two accepted spellings the student should be able to give either of — e.g. perfect 3rd plural `-ērunt` vs `-ēre` (`amāvērunt` / `amāvēre`), or `iī` vs `īvī` perfect stems for `eō`-type verbs. Label them implicitly by leaving both blank and accepting a match in *either* box against *either* accepted answer — don't force the user to know which box is which.
4. A macron/transliteration helper key below the inputs, directly modeled on the reference's Greek transliteration key — but for Latin this should instead show a **macron input helper**, since the point (per the requirements) is that the user should *never have to type a macron to get the answer right*. So this panel should say something like:

   > **Macrons:** you don't need to type them — `amabam` and `amābam` are both accepted. Long vowels are shown in the answer key as ā ē ī ō ū.

   This replaces the Greek tool's "iota sub / rough / smooth" transliteration-key row, since Latin's only diacritic-input problem is macrons, and the design decision here is to make macron entry optional rather than build a transliteration scheme.
5. On submit (Enter key or an implicit submit — match the reference's apparent enter-to-submit flow), grade using the normalization rule in §6, show correct/incorrect, then reveal the correct macron-marked form(s), then advance (NEXT button, or auto-advance — mirror whichever the reference implies; a visible NEXT button is safer/clearer and is consistent with PARSE).
6. RESULTS tab (see 4.5) logs every TYPE question the same way it logs PARSE questions.

### 4.5 RESULTS

Purpose: session history/scoreboard, matching the reference layout exactly.

Layout:
- Heading: `Your results so far (n/m)`.
- A reverse-chronological (or chronological — reference isn't explicit; chronological is fine) list of entries, each a horizontal divider-separated block:
  - The prompt (orange text) — the word that was shown (PARSE) or the verb+tense/voice/mood/person prompt (TYPE).
  - `You chose: <their answer> ✗` or `✓` in blue with the mark in red/green.
  - `Possible: <correct answer(s)>` in blue, only shown/needed when they got it wrong or when multiple valid answers exist.
- No pagination needed for a single study session; a simple scrollable list is fine.

---

## 5. Grammar model — the six Conjugation Tester verbs

### 5.1 Canonical citation forms (principal parts)

| Verb | Conj. | 1st (pres. act. ind. 1sg) | 2nd (pres. act. inf.) | 3rd (perf. act. ind. 1sg) | 4th (supine / perf. pass. ptcp. stem) | Meaning |
|---|---|---|---|---|---|---|
| amō | 1st | amō | amāre | amāvī | amātum | love |
| moneō | 2nd | moneō | monēre | monuī | monitum | warn, advise |
| mittō | 3rd | mittō | mittere | mīsī | missum | send |
| audiō | 4th | audiō | audīre | audīvī | audītum | hear |
| fugiō | 3rd -iō | fugiō | fugere | fūgī | fugitum (rare; often replaced by fut. ptcp. fugitūrus) | flee |
| sum | irregular | sum | esse | fuī | futūrus (no supine; futūrus, -a, -um is the future participle used to build the future infinitive) | be |

(Peter's original list wrote "mittere" and "audire" as the labels — those are the *present infinitives*; the spec above uses the 1st principal part as the canonical tile label for consistency with amō/moneō/fugiō, i.e. the SET-UP verb tile reads **mittō** and **audiō**. This is almost certainly what's wanted pedagogically — flag it to the user once in the HELP panel rather than silently changing behavior.)

### 5.2 Formation rules (for the generator to build every regular form)

Standard Latin verb formation is rule-governed enough to generate programmatically for amō/moneō/mittō/audiō/fugiō (all fully productive paradigms); **sum is irregular and must be hard-coded** (see §5.4). Build a small formation engine keyed off conjugation class + the four principal parts + stem extraction, along these lines:

- **Present stem** = principal part 2 minus `-re` (amā-, monē-, mitte-, audī-, fuge-).
- **Perfect stem** = principal part 3 minus final `-ī` (amāv-, monu-, mīs-, audīv-, fūg-).
- **Supine stem** = principal part 4 minus `-um` (amāt-, monit-, miss-, audīt-, fugit-).

**Present system (built off present stem), active indicative endings:** `-ō/-m, -s, -t, -mus, -tis, -nt` for present; imperfect adds `-ba-` (1st/2nd conj.) or `-ēba-` (3rd/3rd-iō/4th conj.) before the personal endings (`-m, -s, -t, -mus, -tis, -nt`); future adds `-b-` + thematic vowel (1st/2nd conj.: `-bō, -bis, -bit, -bimus, -bitis, -bunt`) or, for 3rd/3rd-iō/4th conj., the vowel-based future (`-am, -ēs, -et, -ēmus, -ētis, -ent` for 3rd/3rd-iō; `-iam, -iēs, -iet, -iēmus, -iētis, -ient` for 4th, using the `-i-` linking vowel already present in the audī-/fugi- stem).

**Present system, passive indicative:** same tense-stems, personal endings `-r/-or, -ris(-re), -tur, -mur, -minī, -ntur`.

**Present system, subjunctive (no future subjunctive exists — omit that cell entirely):**
- Present subj. active: 1st conj. uses `-e-` (am-e-m, am-ē-s…), 2nd/3rd/3rd-iō/4th conj. use `-a-` (mone-a-m, mitt-a-m, audi-a-m, fugi-a-m) — the classic "a-e-a-a-a"/"vowel-flip" rule; passive parallels with `-r/-ris/-tur…`.
- Imperfect subj. (active and passive) = present active infinitive + personal endings — this is fully regular and simple to generate: `amāre-m, amāre-s, amāre-t…` / passive `amāre-r, amāre-ris…`.

**Present system, imperative:**
- Present active: 2sg = present stem alone (1st/2nd/4th conj., e.g. `amā`, `monē`, `audī`) or present stem + `-e` (3rd/3rd-iō, e.g. `mitte`, `fuge`); 2pl = present stem + `-te` (`amāte`, `monēte`, `mittite`, `audīte`, `fugite`).
- Present passive imperative barely occurs outside deponents in real texts, but is formally regular (2sg = present active infinitive form, e.g. `amāre`; 2pl = stem + `-minī`) — include it for completeness since the tester is meant to be exhaustive, but flag in HELP that it's rarely attested for non-deponent verbs like these five.
- Future imperative (2nd/3rd person, active and passive) exists and is regular but rarely taught; include it since exhaustiveness was requested, endings: active 2sg/3sg `-tō`, 2pl `-tōte`, 3pl `-ntō`; passive 2sg/3sg `-tor`, 3pl `-ntor` (no future imperative 2pl passive in Latin — omit that cell).

**Present system, infinitives:** present active = principal part 2 as-is; present passive = present stem + `-rī` (1st/2nd/4th, e.g. `amārī`) or `-ī` (3rd/3rd-iō, e.g. `mittī`, `fugī`).

**Present system, participle:** only **present active** exists here (present stem + `-ns, -ntis`, e.g. `amāns, amantis`; 3rd/3rd-iō/4th conj. link with `-ē-` or `-i-` per normal rules, e.g. `mittēns, mittentis`, `audiēns, audientis`, `fugiēns, fugientis`). There is no present or imperfect passive participle in Latin — omit that cell (see gap table §5.5).

**Perfect system, active (built off perfect stem):**
- Perfect indicative: `-ī, -istī, -it, -imus, -istis, -ērunt` (alternate 3rd pl. `-ēre`, accept both, see §4.3/4.4).
- Pluperfect indicative: `-eram, -erās, -erat, -erāmus, -erātis, -erant`.
- Future perfect indicative: `-erō, -eris, -erit, -erimus, -eritis, -erint`.
- Perfect subjunctive: `-erim, -erīs, -erit, -erīmus, -erītis, -erint`.
- Pluperfect subjunctive: `-issem, -issēs, -isset, -issēmus, -issētis, -issent`.
- (No perfect-system imperative, and no future perfect subjunctive — Latin doesn't have either.)
- Perfect active infinitive = perfect stem + `-isse`.
- No perfect active participle exists for these regular verbs (Latin's gap — only deponents have one, borrowed from the perfect *passive* participle form) — omit.

**Perfect system, passive — built periphrastically, not synthetically:** perfect passive participle (supine stem + `-us, -a, -um`, e.g. `missus, -a, -um`) + the appropriate tense of **sum** as an auxiliary:
- Perfect passive indicative = ptcp + sum present (`missus sum`, `missus es`, `missus est`…).
- Pluperfect passive indicative = ptcp + sum imperfect (`missus eram`…).
- Future perfect passive indicative = ptcp + sum future (`missus erō`…).
- Perfect passive subjunctive = ptcp + sum present subjunctive (`missus sim`…).
- Pluperfect passive subjunctive = ptcp + sum imperfect subjunctive (`missus essem`…).
- Perfect passive infinitive = ptcp + `esse` (`missus esse`).
- The participle must agree in **gender and number with the subject** in real usage; for the tester, default to masculine singular (`missus`) for every cell and note this simplification once in HELP (a fully gender-aware quiz is out of scope).

**Future active participle** = supine stem + `-ūrus, -a, -um` (e.g. `missūrus`). Its infinitive companion, the future active infinitive, = future active participle (acc.) + `esse` (`missūrus esse`); this is worth including under the `infin` mood / `fut` tense / `act` voice cell.

**Future passive participle (gerundive)** = present stem + `-ndus, -a, -um` (1st/2nd conj.) or `-endus, -a, -um` (3rd/3rd-iō/4th conj., e.g. `mittendus`, `audiendus`, `fugiendus`); this fills the `fut / pass / ptcp` cell. Its accompanying "future passive infinitive" is technically the supine + `īrī` construction (e.g. `missum īrī`) — rare, but include it since exhaustiveness was requested, in the `fut / pass / infin` cell.

### 5.3 fugiō — the -iō/mixed conjugation

fugiō conjugates exactly like a 3rd-conjugation verb (mittō) **except** that wherever the present-stem person/number ending itself begins with a short vowel that would otherwise be `-e-` or `-i-`, fugiō instead shows the `-iō` pattern's extra `i`: present active `fugiō, fugis, fugit, fugimus, fugitis, fugiunt` (compare mittō: `mittō, mittis, mittit, mittimus, mittitis, mittunt` — note fugiunt vs mittunt, and fugiō vs mittō). Present passive: `fugior, fugeris/-re, fugitur, fugimur, fugiminī, fugiuntur`. Present active imperative singular is `fuge` (like mitte), plural `fugite` (like mittite) — the -iō pattern only shows up in the 1sg/3pl of the present tense and throughout the present participle/gerundive (`fugiēns`, `fugiendus`), not in the imperative. Imperfect/future/perfect systems are fully regular 3rd-conjugation formations off the perfect stem `fūg-` and supine stem `fugit-`, exactly as in §5.2.

### 5.4 sum — hard-coded irregular paradigm

`sum` has **no passive voice** at all (it's not a transitive verb) — when `sum` is selected on SET-UP, disable/exclude `pass` from the effective voice pool for question generation (tiles stay visible/clickable per §4.1, but no passive question or REVIEW table will ever be produced for `sum`; REVIEW should show the "sum has no passive forms" note from §4.2 if the user manually selects that combination). It also has no supine and no gerundive/future passive participle. It **does** have a present active participle used only in compounds (not for sum itself — omit `ptcp` present for sum, it doesn't exist for the simple verb) but **does** have a future active participle `futūrus, -a, -um` (used to build the future infinitive `fore` or `futūrus esse`) and a full set of active-voice finite forms:

- Present indic.: sum, es, est, sumus, estis, sunt
- Imperfect indic.: eram, erās, erat, erāmus, erātis, erant
- Future indic.: erō, eris, erit, erimus, eritis, erunt
- Perfect indic.: fuī, fuistī, fuit, fuimus, fuistis, fuērunt/fuēre
- Pluperfect indic.: fueram, fuerās, fuerat, fuerāmus, fuerātis, fuerant
- Future perfect indic.: fuerō, fueris, fuerit, fuerimus, fueritis, fuerint
- Present subj.: sim, sīs, sit, sīmus, sītis, sint
- Imperfect subj.: essem, essēs, esset, essēmus, essētis, essent
- Perfect subj.: fuerim, fuerīs, fuerit, fuerīmus, fuerītis, fuerint
- Pluperfect subj.: fuissem, fuissēs, fuisset, fuissēmus, fuissētis, fuissent
- Present imperative: es (2sg), este (2pl)
- Future imperative: estō (2sg/3sg), estōte (2pl), suntō (3pl)
- Infinitives: esse (pres.), fuisse (perf.), futūrus esse / fore (fut., accept either)
- Participle: futūrus, -a, -um (future only)

### 5.5 Gap table — which tense × voice × mood cells exist at all

Use this table (or equivalent logic) to decide, for any verb, which combinations are real and which must be silently excluded from question generation / shown as a plain-text note in REVIEW:

| Mood | Applicable tenses | Voice restriction | Notes |
|---|---|---|---|
| indic | pres, impf, fut, perf, plup, futperf | act + pass (both exist for amō/moneō/mittō/audiō/fugiō; **act only** for sum) | all 6 tenses × both voices for the 5 regular verbs |
| subj | pres, impf, perf, plup | act + pass (act only for sum) | **no future subjunctive in Latin at all** — exclude `fut/subj` and `futperf/subj` for every verb |
| imper | pres, fut | act + pass (act only for sum; passive imperative is vanishingly rare outside deponents but formally exists — include, footnote as rare) | no perfect-system imperative in Latin |
| infin | pres, perf, fut | act + pass (act only for sum, and sum's only "passive-shaped" infinitive equivalent doesn't apply) | no imperfect/pluperfect/future-perfect infinitives in Latin |
| ptcp | pres (act only), fut (act + pass/gerundive), perf (pass only) | see left | no present/imperfect passive participle; no perfect active participle (only deponents have this); sum has future active participle only, nothing else |

Any cell not covered by a row/voice combination above does not exist and must never be offered as a question or answer key. This table is the single source of truth the generator should consult before producing a PARSE prompt, a TYPE prompt, or a REVIEW table.

---

## 6. Macron-optional answer checking

Requirement: the app must **display** macrons in every generated/reference form, but **accept typed answers with or without macrons**.

Implementation: normalize both the user's input and every accepted-answer string by stripping macrons (and any other diacritics) before comparing, e.g. map `ā→a, ē→e, ī→i, ō→o, ū→u` (and uppercase equivalents) via a small lookup or `string.normalize('NFD').replace(/[\u0300-\u036f]/g, '')` if macrons are stored as combining characters, or a direct character-map if they're stored as precomposed Unicode codepoints (Latin macron vowels are precomposed codepoints — ā = U+0101 etc. — so a direct replace map is simplest and most reliable; don't rely on NFD decomposition alone since macron isn't a combining accent in the same Unicode block as acute/grave). Also normalize case (compare lowercase-to-lowercase) and trim whitespace. Do **not** require or penalize punctuation differences beyond that.

Because macrons are optional on input but must appear in the displayed correct answer, always render the answer key / REVIEW tables from the macron-bearing canonical string, never from the user's (possibly macron-less) input.

The two-input-box TYPE screen (§4.4) should accept a correct match in *either* box against *either* member of the accepted-answer set for that cell (e.g. `amāvērunt` / `amāvēre`) — a correct answer in either box, in either order, counts as correct; leaving one box blank is fine if the other is right.

---

## 7. Principal Parts Drill (second mode)

### 7.1 Interaction

- **SET-UP:** a scrollable checklist of all 100 verbs (grouped by conjugation, matching §7.2's table headers) with ALL/NONE buttons, same visual language as the Conjugation Tester's SET-UP tile rows (use checkboxes or toggle-tiles — toggle-tiles are more visually consistent with the rest of the app, so prefer those, just smaller/denser since there are 100).
- **DRILL:** shows one verb's primary meaning + 1st principal part only (e.g. "capiō — seize, take") and four blank input fields labeled implicitly by position (1st, 2nd, 3rd, 4th principal part) for the user to fill in the remaining three (or, alternate mode toggle: show only the English meaning and have the user type all four parts from scratch — default to the "given 1st part + meaning, supply parts 2–4" mode since that's the more common drilling pattern and the harder/more useful skill). Macron-optional checking applies here identically to §6. On submit, reveal the correct four parts (macron-marked) with per-field ✓/✗, then a NEXT button draws a new verb at random from the selected set (no immediate repeats until the pool is exhausted, then reshuffle).
- **RESULTS:** same log pattern as §4.5 — verb prompted, what the user typed for parts 2–4, correct/incorrect per field, running score.
- Defective verbs (`inquam`, `coepī`, `memini`) don't have all four principal parts — the DRILL screen should only present blank fields for the parts that actually exist for that verb (e.g. `coepī` only has parts 3 and 4 in the usual sense — present it as "coepī, coepisse, coeptum" being tested as a 3-part defective, with fields only for the parts that exist) and the SET-UP/verb-list should note "(defective)" next to these three entries.

### 7.2 The 100-verb list

**Sourcing/methodology (state this honestly to the user in HELP or a footnote — don't claim false precision):** this list was compiled by cross-referencing the Dickinson College Commentaries (DCC) *Latin Core Vocabulary* — a frequency-ranked corpus of ~1,000 Latin words compiled from large hand-analyzed text samples (dcc.dickinson.edu/latin-core-list1) — against the standard core-verb sets used in widely-adopted intro/intermediate Latin curricula (Wheelock's Latin, Cambridge Latin Course). Verbs were chosen to (a) skew toward genuinely high-frequency items, (b) guarantee coverage of every conjugation class including the irregular/defective verbs that principal-parts drills exist to reinforce, and (c) avoid an overload of obscure compounds that DCC's raw frequency tail would otherwise surface. This is a curated "most pedagogically important 100," not a strict top-100-by-raw-corpus-count — flag that distinction if Peter wants a stricter frequency cut later (it's a small follow-up to re-derive directly from the DCC ranked export).

Macrons are given in the canonical answer key; per §6, users may type without them.

**1st conjugation (-āre)**

| # | 1st | 2nd | 3rd | 4th | Meaning |
|---|---|---|---|---|---|
| 1 | amō | amāre | amāvī | amātum | love |
| 2 | parō | parāre | parāvī | parātum | prepare |
| 3 | vocō | vocāre | vocāvī | vocātum | call |
| 4 | putō | putāre | putāvī | putātum | think |
| 5 | dō | dare | dedī | datum | give |
| 6 | stō | stāre | stetī | statum | stand |
| 7 | laudō | laudāre | laudāvī | laudātum | praise |
| 8 | servō | servāre | servāvī | servātum | save, keep |
| 9 | spectō | spectāre | spectāvī | spectātum | watch |
| 10 | nūntiō | nūntiāre | nūntiāvī | nūntiātum | announce |
| 11 | cōgitō | cōgitāre | cōgitāvī | cōgitātum | think, plan |
| 12 | cūrō | cūrāre | cūrāvī | cūrātum | care for |
| 13 | mūtō | mūtāre | mūtāvī | mūtātum | change |
| 14 | negō | negāre | negāvī | negātum | deny |
| 15 | optō | optāre | optāvī | optātum | wish for |
| 16 | adiuvō | adiuvāre | adiūvī | adiūtum | help |
| 17 | dōnō | dōnāre | dōnāvī | dōnātum | give, present |
| 18 | creō | creāre | creāvī | creātum | create |
| 19 | probō | probāre | probāvī | probātum | approve, prove |
| 20 | narrō | narrāre | narrāvī | narrātum | tell, relate |
| 21 | appellō | appellāre | appellāvī | appellātum | call, name |
| 22 | vītō | vītāre | vītāvī | vītātum | avoid |
| 23 | ōrō | ōrāre | ōrāvī | ōrātum | speak, beg, pray |
| 24 | celebrō | celebrāre | celebrāvī | celebrātum | celebrate |
| 25 | exīstimō | exīstimāre | exīstimāvī | exīstimātum | think, judge |

**2nd conjugation (-ēre)**

| # | 1st | 2nd | 3rd | 4th | Meaning |
|---|---|---|---|---|---|
| 26 | moneō | monēre | monuī | monitum | warn, advise |
| 27 | videō | vidēre | vīdī | vīsum | see |
| 28 | habeō | habēre | habuī | habitum | have, hold |
| 29 | teneō | tenēre | tenuī | tentum | hold |
| 30 | dēbeō | dēbēre | dēbuī | dēbitum | owe, ought |
| 31 | iubeō | iubēre | iussī | iussum | order |
| 32 | maneō | manēre | mānsī | mānsum | remain |
| 33 | moveō | movēre | mōvī | mōtum | move |
| 34 | doceō | docēre | docuī | doctum | teach |
| 35 | respondeō | respondēre | respondī | respōnsum | answer |
| 36 | timeō | timēre | timuī | — | fear |
| 37 | sedeō | sedēre | sēdī | sessum | sit |
| 38 | iaceō | iacēre | iacuī | — | lie (be lying) |
| 39 | augeō | augēre | auxī | auctum | increase |
| 40 | rīdeō | rīdēre | rīsī | rīsum | laugh |
| 41 | suādeō | suādēre | suāsī | suāsum | urge, advise |
| 42 | caveō | cavēre | cāvī | cautum | beware |
| 43 | gaudeō | gaudēre | gāvīsus sum | — | rejoice *(semi-deponent)* |
| 44 | pāreō | pārēre | pāruī | — | obey |
| 45 | terreō | terrēre | terruī | territum | frighten |
| 46 | misceō | miscēre | miscuī | mixtum | mix |
| 47 | dēleō | dēlēre | dēlēvī | dēlētum | destroy |
| 48 | impleō | implēre | implēvī | implētum | fill |
| 49 | careō | carēre | caruī | — | lack, be without |
| 50 | studeō | studēre | studuī | — | be eager for |

**3rd conjugation (-ere)**

| # | 1st | 2nd | 3rd | 4th | Meaning |
|---|---|---|---|---|---|
| 51 | agō | agere | ēgī | āctum | do, drive |
| 52 | dīcō | dīcere | dīxī | dictum | say |
| 53 | dūcō | dūcere | dūxī | ductum | lead |
| 54 | mittō | mittere | mīsī | missum | send |
| 55 | pōnō | pōnere | posuī | positum | put, place |
| 56 | scrībō | scrībere | scrīpsī | scrīptum | write |
| 57 | currō | currere | cucurrī | cursum | run |
| 58 | crēdō | crēdere | crēdidī | crēditum | believe |
| 59 | discō | discere | didicī | — | learn |
| 60 | petō | petere | petīvī | petītum | seek, ask |
| 61 | vincō | vincere | vīcī | victum | conquer |
| 62 | relinquō | relinquere | relīquī | relictum | leave, abandon |
| 63 | gerō | gerere | gessī | gestum | carry, wage |
| 64 | cadō | cadere | cecidī | cāsum | fall |
| 65 | cēdō | cēdere | cessī | cessum | yield, go |
| 66 | legō | legere | lēgī | lēctum | read, gather |
| 67 | regō | regere | rēxī | rēctum | rule |
| 68 | trahō | trahere | trāxī | tractum | drag |
| 69 | vertō | vertere | vertī | versum | turn |
| 70 | cognōscō | cognōscere | cognōvī | cognitum | learn, recognize |

**3rd conjugation, -iō (mixed)**

| # | 1st | 2nd | 3rd | 4th | Meaning |
|---|---|---|---|---|---|
| 71 | capiō | capere | cēpī | captum | seize, take |
| 72 | faciō | facere | fēcī | factum | do, make |
| 73 | fugiō | fugere | fūgī | fugitum | flee |
| 74 | iaciō | iacere | iēcī | iactum | throw |
| 75 | accipiō | accipere | accēpī | acceptum | receive |
| 76 | cōnficiō | cōnficere | cōnfēcī | cōnfectum | complete |
| 77 | incipiō | incipere | incēpī | inceptum | begin |
| 78 | aspiciō | aspicere | aspēxī | aspectum | look at |

**4th conjugation (-īre)**

| # | 1st | 2nd | 3rd | 4th | Meaning |
|---|---|---|---|---|---|
| 79 | audiō | audīre | audīvī | audītum | hear |
| 80 | veniō | venīre | vēnī | ventum | come |
| 81 | sentiō | sentīre | sēnsī | sēnsum | feel, perceive |
| 82 | inveniō | invenīre | invēnī | inventum | find |
| 83 | dormiō | dormīre | dormīvī | dormītum | sleep |
| 84 | sciō | scīre | scīvī | scītum | know |
| 85 | reperiō | reperīre | repperī | repertum | find, discover |
| 86 | aperiō | aperīre | aperuī | apertum | open |
| 87 | serviō | servīre | servīvī | servītum | serve |
| 88 | pūniō | pūnīre | pūnīvī | pūnītum | punish |

**Irregular / defective**

| # | 1st | 2nd | 3rd | 4th | Meaning |
|---|---|---|---|---|---|
| 89 | sum | esse | fuī | futūrus | be |
| 90 | possum | posse | potuī | — | be able |
| 91 | ferō | ferre | tulī | lātum | carry, bear |
| 92 | eō | īre | iī (or īvī) | itum | go |
| 93 | volō | velle | voluī | — | wish |
| 94 | nōlō | nōlle | nōluī | — | be unwilling |
| 95 | mālō | mālle | māluī | — | prefer |
| 96 | fīō | fierī | factus sum | — | become, be made |
| 97 | inquam | *(defective — present system only)* inquis, inquit | — | — | say *(used with direct quotes)* |
| 98 | edō | edere (or ēsse) | ēdī | ēsum | eat |
| 99 | coepī | coepisse | — | coeptum | began *(perfect system only)* |
| 100 | meminī | meminisse | — | — | remember *(perfect system, present meaning)* |

---

## 8. Technical implementation notes

- **Stack:** single-page vanilla HTML/CSS/JS (or a single React component if that's the house style for the target environment) — no backend, no build step required. This matches the reference tool's evident simplicity and keeps it trivially deployable as a static file.
- **Data-driven, not hand-authored per form:** implement the formation engine from §5.2–§5.4 rather than hard-coding every conjugated form for every verb — this keeps the code short and makes it trivial to add more verbs later (e.g. extending the Conjugation Tester beyond the initial six, or letting the Principal Parts list drive a "conjugate this one too" bonus feature). `sum` is the one fully hard-coded exception table (§5.4).
- **Single source of truth for gaps:** implement §5.5 as a lookup function `cellExists(tense, voice, mood, verb)` that both the question generator and the REVIEW renderer call — this guarantees PARSE/TYPE never quizzes on a nonexistent form and REVIEW never renders a broken table.
- **Answer normalization:** implement §6 as a small `stripMacrons(str)` + `normalize(str)` pipeline used identically for grading, so REVIEW/PARSE/TYPE/DRILL all treat macron-optional input consistently.
- **State shape (suggested):**
  ```
  {
    mode: 'conjugation' | 'drill',
    setup: { tenses: Set, voices: Set, moods: Set, verbs: Set },   // conjugation mode
    drillSetup: { verbIds: Set },                                  // drill mode
    activeTab: 'setup' | 'review' | 'parse' | 'type' | 'results' | 'drill',
    reviewSelection: { tense, voice, mood, verb },
    currentQuestion: { ...prompt-specific fields... } | null,
    score: { correct: number, attempted: number },
    log: [ { prompt, userAnswer, correct: bool, correctAnswer, timestamp } ]
  }
  ```
- **Testing priority:** get the gap table (§5.5) and the `sum` hard-code (§5.4) right first — these are the two places a naive "just apply endings uniformly" implementation will silently produce grammatically nonexistent forms, which would undermine the whole tool's credibility as a study aid.

---

## 9. Open decisions / things to confirm with Peter before or during build

- Whether the Principal Parts Drill's default question direction should be "given part 1 + meaning, supply 2–4" (spec's default) vs. "given meaning only, supply all 4" (harder mode) — consider adding both as a SET-UP toggle since it's cheap to support once the data model exists.
- Whether to eventually extend the Conjugation Tester's six-verb set to include a genuine deponent (e.g. `sequor`, `hortor`) for completeness, since deponents are a common trouble spot — none of the current six trigger that logic, so it's untested by this spec; flagged here rather than silently built in, since it changes the participle/imperative rules meaningfully (§5.2 assumed non-deponent throughout).
