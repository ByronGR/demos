# Handoff: Candidate Assessment Detail

## Overview
This feature adds a **candidate detail screen** to the Nearwork client portal. When a
client opens a candidate from the hiring pipeline, they see the three assessments every
candidate sits — **English (CEFR)**, the role **Assessment** (70% pass line), and **DISC** —
presented as a polished, partner-facing report. It also adds a **competency radar**
(candidate vs. the role cohort average), a **fit-for-role** skills check, auto-derived
**strengths & watch-outs**, and a per-candidate **notes** thread.

The source of this data is a set of real reports candidates receive (an assessment report
with a per-question breakdown + integrity check, a CEFR language evaluation, and a
Cleaver-equivalent DISC profile). **Only the useful, partner-facing content is surfaced —
never the raw report PDF.**

## About the design files
The files in this bundle are **design references created in HTML/React (via in-browser
Babel)** — a prototype showing the intended look and behavior. They are **not production
code to copy verbatim.** The task is to **recreate this design in your target codebase**
(React/Vue/Svelte/native, etc.) using its established components, tokens, and data layer.
If no front-end environment exists yet, pick the framework that best fits the project and
implement the design there.

Everything is inline-styled React in `.jsx` files transpiled in the browser — treat the
styles as a **spec**, not a structure to keep. Port them to your styling system (CSS
modules, Tailwind, styled-components, design-system components…).

## Fidelity
**High-fidelity.** Final colors, typography, spacing, and interactions are all defined.
Recreate the UI pixel-accurately using your codebase's existing primitives. Exact token
values are listed under **Design tokens** below.

---

## The two variants (Demo vs. Clean)

| File | Use | `NW_ASSESSMENTS` |
|---|---|---|
| `source/portal-candidate-data.js` | **Demo** — for presentations/demos | Contains 3 hand-authored example candidates (Maria = strong pass, Paula = fail mirroring a real failing report, Isabella = balanced "Pragmatic Achiever" DISC) |
| `portal-candidate-data.clean.js` | **Repo / production** — commit this one | `NW_ASSESSMENTS = {}` — no fabricated example candidates. All data flows through the deterministic generator (dev) or your real API (prod) |

Both variants share **identical UI code** (`portal-v3-candidate.jsx`). The only difference is
whether the fabricated example candidates are present. For the repo, use the **clean** file
(rename `portal-candidate-data.clean.js` → `portal-candidate-data.js`) and wire
`getCandidateAssessment()` to your real assessment provider (see **Data contract**).

`Nearwork Portal — Demo.html` is a **single self-contained, runnable** copy of the whole
portal with the demo examples — open it in a browser to see the finished design. (External
CDNs for React/Babel/Lucide/fonts still load over the network.)

---

## Screens / Views

### Candidate Detail (`CandidateDetailScreen`)
**Purpose:** Let a client review one candidate's three assessment results in depth and decide
whether to advance them.

**Route:** opened from the pipeline. A candidate card click (when not in compare mode)
navigates to this screen with the candidate id. In the prototype the router is a `switch` on
`nav.route === 'candidate'` passing `candidateId`.

**Page layout** (max content width **1120px**, centered; app is sidebar + top bar + scrolling
main; page background `#F8F7F3`):

1. **Back link** — "← Pipeline" (12px, 600, `#757575`).
2. **Header card** (white, border `#EBEBEB`, radius 22, padding 30):
   - Left: circular candidate avatar (70px, candidate's brand color), name (31px/700,
     letter-spacing −0.03em), stage pill, role · location · years-exp, skill chips.
   - Right: **Nearwork match** donut (`MatchScore`, value 0–100, 58px) with label.
   - **Action bar** (top border, 18px pad): `Advance to <next stage>` (primary, turns into a
     confirmation pill on click), `Add note` (secondary, scrolls to Notes), `Request interview`
     (secondary), `Compare` (ghost, right-aligned).
3. **Submitted meta row** — "Submitted <date> · Reviewed by <reviewer>" (12px `#757575`).
   > ⚠️ Attribution is intentionally **human** ("Reviewed by Nearwork talent team"). Do **not**
   > label grading as AI anywhere — this was an explicit product decision.
4. **Score tiles** (3 across, flex, gap 16, each min-width 220, radius 18, padding 22):
   - **English · CEFR** — big level (e.g. `C1`, 38px/700) + `92%`, teal progress bar, A1…C2 scale
     with the current level highlighted teal.
   - **Assessment** — big `88%` (teal if pass, rose if fail), pass/fail pill, progress bar with a
     **pass-line marker** at 70%. Failing tiles get a faint rose tint (`#FFFBFC` bg, `#F3D9E2`
     border).
   - **DISC profile** — letter badge in the type color, label + classification, D/I/S/C mini-segments
     with the primary type filled.
5. **Competency profile** (full-width panel): **radar chart** on the left with the legend
   **below it**, and a per-axis comparison table on the right (bar + average tick + delta vs.
   average). Footer: "Benchmarked against N candidates assessed for similar roles."
6. **Two-column body** (grid `1.55fr 1fr`, gap 20, align start):
   - **Left column:** Assessment report card, then English + DISC-summary side by side.
   - **Right column (context rail):** Snapshot → Fit for role → Strengths & watch-outs → Notes.
7. **DISC profiles** (full-width): three cards — **Natural / Adapted / Under pressure** — each
   with D/I/S/C percentile bars; interpretive-aid disclaimer footer.

**Pending state:** if a candidate hasn't sat the assessment (`completed === false`, e.g. stage
"Applied"), the whole body is replaced by an "Assessment not completed yet" empty card.

#### Components in this screen
- **`ScoreTiles`** — the three summary tiles (see above).
- **`RadarChart`** — pure SVG hexagon (6 universal axes). Two overlaid polygons: candidate
  (teal `#16A085`, 16% fill, 2px stroke, vertex dots) and cohort average (gray `#9E9E9E`, dashed
  `4 3`, 18% fill). Rings at 25/50/75/100. Axis labels at 116% radius; long labels shortened via
  a map.
- **`CompetencyPanel`** — wraps the radar + the comparison table; delta chips are teal for ≥ avg,
  amber `#A16207` for < avg.
- **`QuestionRow`** — collapsible per-question row: `Q#` chip, competency overline (role-specific),
  prompt, score `x.x/5` colored by ratio, thin score bar. Expands to show the candidate's answer
  (quote block), an optional follow-up Q&A, and the **Assessor feedback** callout (teal, or rose
  `triangle-alert` if the answer was flagged, e.g. repeated verbatim).
- **`IntegrityStat`** — number + label tile; turns amber when a threshold is crossed
  (tab switches ≥ 3, any copy-paste, focus losses ≥ 3).
- **`SnapshotPanel`** — experience, salary expectation, availability, timezone, applied-ago.
- **`SkillsMatchPanel`** ("Fit for role") — cross-references candidate skills against the role's
  must-haves (check vs. dash) with an "N/M must-haves" pill, plus matched nice-to-haves as bonus chips.
- **`HighlightsPanel`** — Strengths (top competencies ≥ 3.9/5 + advanced English) and Watch-outs
  (competencies < 3.2/5 + integrity flag).
- **`EnglishPanel`** / **`DiscSummaryPanel`** — level/score/summary; classification + headline +
  narrative.
- **`DiscProfileCard`** — one of the three DISC contexts, four dimension bars (`p{value}`).
- **`NotesPanel`** — pinned recruiter note + a working composer; client notes persist in
  `localStorage` under `nw_cand_notes_<id>` (replace with your notes API).
- **`AssessmentPending`** — the empty state.

---

## Interactions & Behavior
- **Open candidate:** pipeline card / list row → `onNav('candidate', candidate.id)` when not in
  compare mode. (Compare mode instead toggles selection.)
- **Advance stage:** local confirm — button swaps to a "Moved to <stage> — Nearwork notified" pill.
  Wire to your pipeline-mutation endpoint.
- **Add note / Request interview:** smooth-scroll to the Notes composer (`scrollIntoView`, center).
- **Post note:** appends to the thread and persists to `localStorage` (client notes only; the
  recruiter note is pinned and read-only). Replace with a real create-note call.
- **Question rows:** click to expand/collapse (`nwFade` 160ms). Chevron flips.
- **Transitions:** cards/hover use 120–160ms ease; the modal/pop animations reuse the app's
  `nwFade`/`nwPop` keyframes.
- **No AI attribution** anywhere in copy (see note above).

## State management
Per-screen local state only (prototype):
- `advanced: boolean` — advance-stage confirmation.
- `NotesPanel`: `notes: Note[]` (seeded with the recruiter note + `localStorage`), `draft: string`.
- `QuestionRow`: `open: boolean` per row.

Data is resolved synchronously via `window.getCandidateAssessment(candidate)`. In production this
becomes an async fetch — add loading/error states around it (the "pending" empty state is a good
model for "no results yet").

## Data contract
`getCandidateAssessment(candidate)` returns the object the screen renders. Wire this to your
assessment provider. Shape:

```ts
type Assessment = {
  completed: boolean;              // false → render the pending empty state
  role: string;
  submitted: string;               // e.g. "Jun 28, 2026"
  gradedBy: string;                // human attribution, e.g. "Nearwork talent team"
  english: {
    level: 'A1'|'A2'|'B1'|'B2'|'C1'|'C2';
    score: number;                 // 0–100
    summary: string;
  };
  assessment: {
    overall: number;               // 0–100
    passing: number;               // pass line, default 70
    status: 'passed' | 'failed';
    integrity: { risk: number; tabSwitches: number; copyPaste: number; focusLosses: number };
    summary: string;
    questions: Array<{
      n: number;
      prompt: string;
      competency: string;          // role-specific label (question breakdown only)
      score: number;               // 0–5
      max: number;                 // 5
      answer: string;              // candidate transcript
      feedback: string;            // assessor feedback
      followUp?: { q: string; a: string };
    }>;
  };
  disc: {
    type: 'D'|'I'|'S'|'C';
    label: string;                 // e.g. "Driver"
    classification: string;        // e.g. "The Decisive Driver"
    headline: string;
    narrative: string;
    profiles: {                    // percentiles 0–100 per dimension
      natural:  { D:number; I:number; S:number; C:number };
      adapted:  { D:number; I:number; S:number; C:number };
      pressure: { D:number; I:number; S:number; C:number };
    };
  };
};
```

Two more helpers derive from the above (keep or reimplement):
- **`getCandidateRadar(candidate)`** → `{ axes, candidate:number[], average:number[], cohortSize }`.
  The **radar axes are universal across all roles**: `Communication, Problem solving, Adaptability,
  Leadership` (mapped from the four answer scores) + `English` + `Assessment`. The average is over
  candidates assessed for the same role family.
- **`getCandidateHighlights(candidate)`** → `{ strengths[], watchOuts[] }`.

**In the clean file**, `NW_ASSESSMENTS = {}` and every candidate is served by a deterministic
generator (seeded by candidate id) — useful for local dev. Replace that generator with your real
API for production; the return shape above is the contract the UI depends on.

## Design tokens

**Fonts:** `Poppins` (UI, weights 300–800), `JetBrains Mono` (numbers/scores).

**Color palette** (`NW` object in `primitives.jsx`):
- Base: white `#FFFFFF`, black `#111111`, off-white `#F8F7F3`
- Grays: 50 `#F5F4F0`, 100 `#EBEBEB`, 200 `#D9D9D9`, 300 `#BDBDBD`, 400 `#9E9E9E`, 500 `#757575`,
  600 `#555555`, 700 `#383838`, 800 `#232323`, 900 `#161616`
- Teal (primary): 50 `#E8F8F5`, 100 `#C8EDE6`, 500 `#16A085`, 600 `#12866E`, 700 `#0E6B58`
- Rose (fail/alert): 50 `#FEF0F5`, 500 `#E74C7C`, 600 `#CC3666`
- Violet: 50 `#F7F2FC`, 500 `#AF7AC5` · Yellow: 50 `#FEFCE8`, 500 `#EAB308` (amber text `#A16207`)
- Blue: 50 `#EFF6FF`, 500 `#3B82F6`

**DISC dimension colors:** D `#E74C7C` (Dominance), I `#EAB308` (Influence), S `#16A085`
(Steadiness), C `#3B82F6` (Compliance).

**Radii:** header/section cards 20–22, score tiles / inner cards 14–18, chips/pills 999, small
tiles 10–12. **Shadows:** subtle only (`0 1px 2px rgba(0,0,0,.03)` at rest; slightly stronger on
hover). **Spacing:** panel padding 22–26; grid gaps 16–20.

## Assets
No image assets. Icons are **Lucide** (`lucide@0.469.0`), rendered via the `Icon` helper — map to
your icon set. Names used include: `radar, orbit, activity, languages, clipboard-check,
clipboard-list, triangle-alert, message-square-quote, user-check, target, scale, id-card,
briefcase, wallet, calendar-clock, calendar-check, calendar-plus, clock, inbox, map-pin,
arrow-left, arrow-right, arrow-up, arrow-down, chevron-up, chevron-down, check, minus, plus,
circle-check, circle-x, info, hourglass, send, message-square-text`.

## Files
Design-reference source (in `source/`):
- **`portal-v3-candidate.jsx`** — the whole Candidate Detail screen + all its sub-components. **Start here.**
- **`portal-candidate-data.js`** — the assessment data layer (demo, with examples): CEFR/DISC
  metadata, role question banks, the deterministic generator, and `getCandidateAssessment` /
  `getCandidateRadar` / `getCandidateHighlights`.
- **`portal-v3-pipeline.jsx`** — how the detail is opened (card/row click → `onNav('candidate', id)`).
- **`portal-v3-compare.jsx`** — the side-by-side compare modal (an Assessment row was added here too).
- **`portal-shared.jsx`** — shared chrome (sidebar, top bar, `CandidateAvatar`, `EmptyBlock`).
- **`primitives.jsx`** — design tokens (`NW`) + primitives (`Icon`, `Button`, `Avatar`, `MatchScore`, `Chip`).
- **`tokens.css`** — base CSS variables/reset.

Repo variant (root of this bundle):
- **`portal-candidate-data.clean.js`** — the clean data layer (`NW_ASSESSMENTS = {}`) to commit.

Runnable reference (root of this bundle):
- **`Nearwork Portal — Demo.html`** — self-contained, opens in a browser, shows the finished design
  with the demo examples.
