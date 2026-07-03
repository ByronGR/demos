# Nearwork Client Portal — full source

The complete Nearwork client-portal prototype: Overview, Pipeline, Candidate detail
(English / Assessment / DISC + competency radar), Team, Hire detail, Billing, SPP,
Users, Settings, and the compare + kickoff flows.

## What this is
A **design-reference prototype** built as a multi-file front-end: one HTML shell that
loads React + Babel from a CDN and transpiles a set of `.jsx`/`.js` component files **in
the browser**. It's a high-fidelity spec of look, layout, and behavior — **not the
production build**. For the real product, recreate these screens in your target codebase
(React/Vue/Svelte/native) using its own components, tokens, build tooling, and data layer.
See `docs/CANDIDATE_ASSESSMENTS.md` for a fully documented example (the candidate-assessment
feature) including a **data contract**.

> This is intentionally build-free (no bundler/JSX pipeline) so it runs by just opening the
> HTML. In production you'd compile the JSX and drop the CDN `<script>` tags.

## Run it
No build step. Because browsers block `file://` module/asset loads, serve the folder over
HTTP:

```bash
cd nearwork-portal-source
python3 -m http.server 8000
# open http://localhost:8000/Nearwork%20Portal.html
```

(Requires internet for the CDN scripts: React 18.3.1, Babel standalone 7.29.0,
Lucide 0.469.0, and Google Fonts.)

## Demo vs. Repo (the two data variants)
Only one file changes between the two:

- **Demo** — uses `portal-candidate-data.js` as shipped. Contains 3 hand-authored example
  candidates (a strong pass, a fail mirroring a real failing report, and a balanced DISC
  showcase). Good for presentations.
- **Repo** — swap in `portal-candidate-data.clean.js` (it sets `NW_ASSESSMENTS = {}`, so
  there are **no fabricated example candidates**; every candidate is served by the
  deterministic dev generator until you wire real data). To use it, either rename
  `portal-candidate-data.clean.js` → `portal-candidate-data.js`, or point the `<script>` in
  `Nearwork Portal.html` at the `.clean.js` file.

All other mock data (`portal-data.js`) is generic prototype content, not sensitive.

## File map

**Shell & foundation**
- `Nearwork Portal.html` — app shell + client-side router (`App` renders a screen by
  `nav.route`). Loads every script below in order.
- `tokens.css` — base CSS variables / reset.
- `primitives.jsx` — design tokens (`NW` palette) + primitives: `Icon` (Lucide),
  `Button`, `Avatar`, `Chip`, `MatchScore`, `CompanyTile`, `Logo`.
- `portal-shared.jsx` — shared chrome: `PortalSidebar`, `PortalTopBar`, `CandidateAvatar`,
  `EmptyBlock`, `SectionHead`, `ScoreChip`.
- `tweaks-panel.jsx` — the in-prototype "Tweaks" panel (data state / density / SPP layout).
- `image-slot.js` — drag-drop image placeholder web component.

**Data (mock)**
- `portal-data.js` — all portal mock data: candidates, openings, team/hires, billing, SPP,
  users, activity, etc.
- `portal-candidate-data.js` — candidate **assessment** layer (English CEFR + Assessment +
  DISC + competency radar + highlights) with demo examples. See docs.
- `portal-candidate-data.clean.js` — the repo variant of the above (no examples).

**Screens (one component each)**
- `portal-v3-mixed.jsx` — Overview dashboard.
- `portal-v3-roles.jsx` — Open roles list.
- `portal-v3-pipeline.jsx` — Pipeline kanban / list (opens a candidate).
- `portal-v3-candidate.jsx` — **Candidate detail** (the assessment screen). See docs.
- `portal-v3-compare.jsx` — side-by-side candidate compare modal.
- `portal-v3-kickoff.jsx` — kickoff-brief review.
- `portal-v3-team.jsx` — Team list & team detail.
- `portal-v3-hire.jsx` — Hire (employee) detail: reviews, EOR, PTO, comp, notes.
- `portal-v3-review.jsx` — performance-review modals + star rating.
- `portal-v3-billing.jsx` — Billing: spend, invoices, per-hire breakdown.
- `portal-v3-spp.jsx` / `portal-v3-spp-org.jsx` — Strategic Partner Program views + org switcher.
- `portal-v3-users.jsx` — Users & roles.
- `portal-v3-settings.jsx` — Settings.

**Docs**
- `docs/CANDIDATE_ASSESSMENTS.md` — deep-dive on the candidate-assessment feature: screen
  spec, every component, interactions, state, **data contract**, and design tokens. Use it
  as the model for how to port any screen.

## How the pieces connect
Each `<script type="text/babel">` file attaches its component(s) to `window` (no ES modules),
and `Nearwork Portal.html`'s `App` composes them. When porting: keep the **component
boundaries and data shapes**, drop the `window` globals + in-browser Babel in favor of your
framework's imports and build. Inline styles are the visual spec — move them to your styling
system.

## Notes
- **No AI attribution** in copy — assessments are framed as human-reviewed ("Reviewed by
  Nearwork talent team"). Keep it that way.
- Client-side persistence uses `localStorage` (nav position, notes, feedback drafts) — replace
  with real APIs.
- Icons are Lucide names via the `Icon` helper — map to your icon set.
