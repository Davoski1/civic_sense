# CivicSense Landing Page Redesign — Requirements

## Document status

- Stage: Stage B (specification revised to match confirmed product decisions + hostile review). No application code is changed by this document.
- Stage B decision source: the product owner overrode several Stage A choices. Where Stage A said DELETE, the owner now says KEEP but reframe truthfully. This document is authoritative; where it conflicts with prior Stage A wording, this wording wins.
- Scope surface: `/projects/sandbox/civic_sense/landing_page` **only**. Nothing outside this directory may be created, modified, deleted, or moved (this includes `bot_server/`, `cs_website/`, `fc_dashboard/`, and repo-root files).
- Stack is fixed (see design.md). No framework migration. Next.js 16, React 19, Tailwind CSS v4, `motion`, Radix UI, Biome are treated as source of truth.

## Product truth (authoritative, do not invent beyond this)

CivicSense is a **WhatsApp (and Telegram) AI fact-checking bot for Nigerians**. Tagline: *"Send a rumour to WhatsApp. Get the truth back."* One WhatsApp number, no app, no sign-up; a structured, sourced verdict (VERIFIED / MISLEADING / FALSE / UNVERIFIED) returns typically in under 20 seconds.

### Authoritative About / explainer copy (owner-supplied, use verbatim as the source)

The following two paragraphs are the authoritative source for the About/explainer section. Layout may split or reformat them, but the wording MUST NOT be contradicted or expanded with invented claims.

**What is CivicSense:** "CivicSense is a WhatsApp bot that fact-checks political claims in Nigeria. You forward any claim — about a politician, an election, a policy — and the bot searches the news and verified sources to tell you if it's true, false, or unverified. It takes seconds. No app. No sign-up. Just WhatsApp. We also have a website where you can report election misconduct anonymously and see politician profiles with their track records, promises, and scandals all in one place. So instead of political news disappearing from your feed after a few days, it stays there and people can search it anytime."

**Why Nigeria needs this:** "Because misinformation spreads fast on WhatsApp and Facebook. A rumor becomes fact before anyone verifies it. Politicians make promises and break them, but nobody remembers. Misconduct happens and gets buried in noise. CivicSense keeps the record straight so citizens can vote informed and hold leaders accountable. When millions of Nigerians have access to the same verified truth, accountability becomes harder to ignore."

Note: this copy describes the website's report + politician-profile capabilities. That is allowed in the About section. It does NOT change the HERO pipeline visual, which still depicts only the real WhatsApp fact-check loop (see FR-3.4 / FR-6).

### Roadmap features (real, in development — advertise as "Coming soon", never as live)

The owner confirms these are genuine roadmap features still in development. They MAY be advertised on the page, but ONLY with a visible "Coming soon" badge/state and framed as upcoming, never as available today. Do NOT build the actual features:

- **Election Calendar** — upcoming. Advertise with a "Coming soon" badge; do not claim calendar sync works today.
- **Politician Watch** — upcoming. Advertise with a "Coming soon" badge; do not claim profiles are live/browsable today.

### Aspirational metrics (goals, not achieved results)

The owner's existing metric copy is kept, but every number MUST be framed explicitly as an aim/target, never as an achieved result. Reword to make the aspiration unambiguous, e.g. "Our goal:" / "We're aiming to reach…". Examples the owner keeps as goals: "10M+ youths on the voter roll", "50+ politician profiles". These MUST read as targets we are working toward, not as figures we have reached.

Features that **actually exist and are live** in the product (verified against repo `README.md` and `CONTEXT.md`):

1. WhatsApp fact-checking via Twilio (text + image claims).
2. Telegram fact-checking (same pipeline).
3. Live news search (Tavily, 16 Nigerian domains).
4. RSS article index (17 Nigerian feeds incl. IFCN fact-checkers, 30-min sync).
5. Image claim extraction (analyzed only, never stored).
6. Civic knowledge base (25 curated Nigerian civic facts).
7. Fact-check dashboard (feed, 7-day chart, trends, CSV export) — separate app `fc_dashboard`.
8. Interactive chat UI (dashboard).
9. Anonymous incident reporting -> admin approve/reject -> public map — separate app `cs_website`.
10. Public incident map (Leaflet, 36-state coordinates, color-coded markers).
11. Conflict tracker (beta).

Pipeline facts that MAY be stated: parallel retrieval (knowledge base + RSS index + Tavily) feeding a **single Gemini 2.5 Flash call via OpenRouter**; verdicts logged to MongoDB; typical reply well under 20s.

### Truthful-framing rules (the unambiguous rule set for implementation)

The following are ALLOWED only under the exact framing stated; any other framing is forbidden:

- **Election Calendar** — ALLOWED as a roadmap advert with a visible "Coming soon" badge/state, presented as upcoming. FORBIDDEN to state or imply it is live today, or that Apple/Google calendar sync currently works. Do not build the feature.
- **Politician Watch** — ALLOWED as a roadmap advert with a visible "Coming soon" badge/state, presented as upcoming. FORBIDDEN to state or imply profiles are browsable/live today. Do not build the feature.
- **Metrics ("10M+ youths on the voter roll", "50+ politician profiles", and similar)** — ALLOWED only when explicitly framed as an aim/goal/target (e.g. "Our goal:", "We're aiming to reach…"). FORBIDDEN to state them as achieved results, current user counts, or delivered figures.

### Claims that MUST NOT appear under any framing (they are not part of the product)

- Any testimonial, customer logo, partnership, award, or achieved user-count that is not in the repo.
- Any claim that a roadmap feature (Election Calendar, Politician Watch) is already live/available.
- Any metric stated as an achieved result rather than an aspiration.
- Real personal names / repo contributor handles in the Team section. For anonymity the owner requires RANDOM/PLACEHOLDER display names. The real handles (`adriel-babalola`, `debugAyo`) and any real personal names ("David Adeola", "Promise Abiodu") MUST NOT be shown. See FR-9.

The 17 news/fact-check **source logos** in `src/assets/sources/` ARE real and attributable (README "Acknowledgements") and MAY be shown as "sources we cross-check against", never as partners or endorsers.

## Functional requirements

### FR-1 Design token system
- **FR-1.1** A single semantic token layer MUST exist in `src/app/globals.css` (Tailwind v4 `@theme`/`:root`) defining at minimum: `--background`, `--foreground`, `--muted`, `--muted-foreground`, `--surface`, `--surface-elevated`, `--border`, `--accent` (emerald brand), `--accent-secondary`, `--success`, `--warning`, `--danger`. The existing shadcn tokens (`--primary`, `--card`, `--ring`, chart/sidebar tokens, `--radius`) MUST be preserved so existing `ui/button.tsx` and `ui/popover.tsx` keep working.
- **FR-1.2** The brand accent MUST be derived from the colors already in the codebase: emerald family (`#2ECC71`, `#34d399`, `rgb(16 185 129)`) and the navy overlay (`rgba(13,27,42,*)`). No new unrelated hue may be introduced as a primary brand color.
- **FR-1.3** Radius, spacing, shadow/elevation, and motion-timing scales MUST be centralized as tokens/constants (CSS variables and/or a TS constants module) and referenced by components. Acceptance: after the change, a grep for ad-hoc hex/rgb brand colors in `src/components/*.tsx` returns only values sourced through tokens or a documented exception list.
- **FR-1.4** No visual regression to the existing dark aesthetic: background stays dark, foreground light.

### FR-2 Navigation (site-header)
- **FR-2.1** Header MUST remain a fixed top bar with logo, in-page anchor nav (About, Sources, How it works, Sample verdicts — anchors must continue to resolve to existing section ids), and a primary "Try on WhatsApp" action.
- **FR-2.2** Header MUST gain a scroll-state transition (transparent at top -> elevated/glass surface after scroll) implemented with the glass primitive (FR-7) and `motion`, respecting reduced motion.
- **FR-2.3** Mobile menu (Radix Popover) MUST retain keyboard operability and focus management, and MUST restore body scroll on close using an effect/cleanup (not the current render-time `document.body.style` mutation). Acceptance: opening/closing the menu never leaves `body` scroll locked; no hydration warning.
- **FR-2.4** Active in-page section MUST be reflected in nav state (e.g. via IntersectionObserver) with an accessible current indicator.

### FR-3 Hero
- **FR-3.1** Hero MUST answer, within ~2–3 seconds of reading: what CivicSense is, why it matters, what the visitor can do. Copy MUST use existing product messaging (tagline + "Verify. Share. Vote informed." + the reclaim/accountability line are acceptable) without inventing claims.
- **FR-3.2** Hero MUST establish a clear type hierarchy: eyebrow/status, display headline, supporting paragraph, primary + secondary CTA, and a trust signal grounded in real facts (e.g. "Cross-checks 17 Nigerian newsrooms + IFCN fact-checkers").
- **FR-3.3** The two existing hero photographs (`public/image_man_holding_flag_walking_across_street.webp`, `public/holding_hands_in_a_ring.jpg`) MUST be rendered through `next/image` with explicit sizing, `priority` only on the first/LCP image, and the second lazy. Background-image CSS divs for these photos MUST be removed as the LCP path.
- **FR-3.4** Hero MUST include an art-directed product visualization of the **real** flow (rumour -> WhatsApp -> parallel retrieval -> sourced verdict). See FR-6. It MUST NOT depict civic-issue reporting, authority routing, or resolution tracking as the hero product (those belong to sibling apps, not this bot's core loop).
- **FR-3.5** Cumulative Layout Shift from the hero MUST be ~0 (reserved dimensions for media, fonts loaded without layout jump).

### FR-4 About / "What is CivicSense?"
- **FR-4.1** The About section intro copy MUST use the owner-supplied authoritative copy (the two paragraphs in Product truth). It may be split/reformatted for layout (e.g. a "What is CivicSense" block and a "Why Nigeria needs this" block) but MUST NOT be contradicted or padded with invented claims.
- **FR-4.2** The feature cards MUST present the owner's kept feature set. Live features (WhatsApp fact-check, Anonymous reporting + public map, Live news/RSS + Tavily grounding, Dashboard) are shown normally. Roadmap features (Election Calendar, Politician Watch) MUST be shown with a visible "Coming soon" badge/state and MUST NOT link to a destination that implies the feature is live (a disabled/non-link "Coming soon" treatment is preferred over an outbound link). Card ordering SHOULD place live features before "Coming soon" ones so the page reads as truthful at a glance.
- **FR-4.3** Any metric/"Target" line MUST be reframed as an explicit aspiration (e.g. "Our goal:", "Aiming for"). The owner's existing numbers are kept but worded as goals, never as achieved results. A line with no real or clearly-aspirational value MUST be removed rather than left ambiguous.
- **FR-4.4** Outbound links for LIVE features MUST point to real destinations from `site-config.ts` (WhatsApp URL, map, report, dashboard, repo). Placeholder `civicsense.app` links may remain but MUST be flagged (see FR-11). "Coming soon" cards MUST NOT be presented as working links.
- **FR-4.5** The "Coming soon" state MUST be conveyed non-visually too (e.g. text "Coming soon" in the accessible name, not color alone), per FR-10.

### FR-5 Sources, How-it-works, Sample verdicts
- **FR-5.1** Sources ticker MUST keep using the 18 real logos in `src/assets/sources/` through `next/image`. Marquee MUST pause on hover/focus and when offscreen, and MUST fall back to a static, scrollable row under reduced motion.
- **FR-5.2** How-it-works MUST keep the accurate 3-step narrative (send claim -> AI gathers evidence -> sourced verdict) and the WhatsApp-style panels; interactions MUST respect reduced motion (the animated border already checks it — preserve that).
- **FR-5.3** Sample verdicts marquee MUST keep the existing, repo-sourced verdict data (these match README test claims). Marquee MUST pause on hover/focus and offscreen, with reduced-motion fallback. No verdict copy may be invented beyond what is already present/repo-supported.

### FR-6 Product visualization
- **FR-6.1** A dedicated, reusable visualization component MUST depict the real pipeline: claim in -> [Knowledge base • RSS index • Tavily] parallel retrieval -> single Gemini call -> structured verdict with sources.
- **FR-6.2** Default implementation MUST be DOM + SVG + `motion` (no mandatory WebGL). It MUST be understandable as a static frame (works with JS reduced/paused) and enhanced with motion when allowed.
- **FR-6.3** Any WebGL/R3F enhancement is OPTIONAL and gated (see FR-8). It MUST NOT be required for comprehension and MUST NOT be the LCP element.

### FR-7 Glass / material system
- **FR-7.1** A reusable glass primitive (e.g. `GlassPanel` / `GlassSurface`) MUST be created and used where glass appears (header, cards, product-viz chrome), replacing scattered ad-hoc `backdrop-blur` usage.
- **FR-7.2** Progressive enhancement tiers MUST be implemented: (a) capable browser -> frosted glass with `backdrop-filter` + edge lighting; (b) reduced-motion -> static glass, no animated response; (c) low-power/mobile or unsupported `backdrop-filter` -> solid/semi-opaque surface fallback. The effect MUST NEVER be required for legibility or usability.
- **FR-7.3** Text over glass MUST meet contrast requirements (FR-10) in every tier.

### FR-8 Motion & performance gating
- **FR-8.1** All decorative/looping animation (marquees, hero parallax, CTA starfield, product-viz loops) MUST pause when offscreen (IntersectionObserver) and MUST honor `prefers-reduced-motion: reduce`.
- **FR-8.2** The global `mousemove` listener in `use-relative-mouse-position.ts` MUST be fixed: registered with a proper dependency array and cleaned up, and ideally scoped so it does not run when the CTA is offscreen. No effect may re-register listeners on every render.
- **FR-8.3** If (and only if) WebGL/R3F is added, it MUST: lazy-load (dynamic import, `ssr:false`), mount only after a capability check (reduced-motion off, adequate `deviceMemory`/heuristics, WebGL available), cap DPR (e.g. `[1, 1.5]`), pause its render loop when offscreen/hidden, dispose GL resources on unmount, and provide a static fallback. If the capability check or the introduction of R3F is not justified during implementation, the DOM/SVG visualization (FR-6.2) stands and no new dependency is added.
- **FR-8.4** No animation loop, timer, or listener may leak: every `setInterval`, `requestAnimationFrame`, `addEventListener`, and GL context MUST be cleaned up on unmount.

### FR-9 Team section
- **FR-9.1** The team section is KEPT. For anonymity it MUST use RANDOM/PLACEHOLDER display names and neutral roles — NOT the real repo contributor handles (`adriel-babalola`, `debugAyo`) and NOT any real personal names ("David Adeola", "Promise Abiodu"). Placeholder names MUST be plausible-but-generic (e.g. "A. Contributor", "Team Member", or invented neutral names) and MUST NOT impersonate a real, identifiable person.
- **FR-9.2** A short code comment in `team-section.tsx` MUST state that these are intentional anonymized placeholders (not fabricated real people), so a future reviewer does not mistake them for real contributors. Roles MUST be generic and truthful to the kind of work done (e.g. "Engineering", "AI pipeline", "Design"), not invented credentials.
- **FR-9.3** The Github/handle icon treatment MUST NOT link a placeholder name to a real profile. If an icon implies a link, it MUST NOT resolve to a real person's account.

### FR-10 Accessibility
- **FR-10.1** Semantic HTML and a single, correct heading hierarchy (one `h1` in the hero; section `h2`s) MUST be maintained.
- **FR-10.2** All meaningful images MUST have descriptive `alt`; decorative canvas/SVG MUST be `aria-hidden` and MUST NOT trap focus or block interaction.
- **FR-10.3** Every interactive element MUST have a visible focus indicator; focus outlines MUST NOT be removed without an equally visible replacement.
- **FR-10.4** Text contrast MUST meet WCAG AA (4.5:1 body, 3:1 large text) in all glass tiers and over hero imagery.
- **FR-10.5** Keyboard navigation MUST reach and operate all controls (nav, mobile menu, CTAs, any interactive viz). Reduced motion MUST be honored throughout.

### FR-11 SEO & metadata
- **FR-11.1** Existing `metadata` (title template, description, keywords, OpenGraph, Twitter, `opengraph-image.png`, favicon/icon) MUST be preserved and only improved with truthful content. `metadataBase` and placeholder `civicsense.app` URLs MUST be flagged for manual confirmation if not yet real.
- **FR-11.2** No metadata claim may be fabricated (no fake ratings, counts, or partnerships).

### FR-12 Code quality & maintainability
- **FR-12.1** Reusable primitives MUST be introduced where repetition exists (glass primitive, section wrapper, motion helpers), without over-fragmenting into trivial components.
- **FR-12.2** Design constants (colors via tokens, spacing, motion timings/easings, breakpoints) MUST be centralized, not duplicated as magic numbers.
- **FR-12.3** Props MUST be typed; `page.tsx` MUST remain a thin composition of section components.
- **FR-12.4** No new dependency may be added without justification against the 6 questions in the brief (necessity, existing-stack feasibility, bundle cost, compatibility, mobile performance, simpler alternative). Preference: use `motion`, SVG, CSS. R3F/three only if FR-8.3 gating is met.

## Non-functional / acceptance criteria (measurable)

### Validation gates (run from `/projects/sandbox/civic_sense/landing_page`)
- **AC-1** `npm run lint` (`biome check .`) passes with no new errors.
- **AC-2** `npm run typecheck` (`tsc --noEmit`) passes with zero errors.
- **AC-3** `npm run build` (`next build`) completes successfully (standalone output).

### Behavior / quality gates
- **AC-4** No horizontal overflow at 320, 375, 390, 430, 768, 1024, 1280, 1440, 1920 px widths.
- **AC-5** With `prefers-reduced-motion: reduce`, no continuous/looping animation runs; content is fully readable and all CTAs reachable.
- **AC-6** All marquees/parallax/starfield/viz loops stop when scrolled fully offscreen (verified via IntersectionObserver wiring and, in visual QA, no CPU churn when the section is not visible).
- **AC-7** Hero LCP is one of the hero photographs or the headline text, served via `next/image`/font with reserved space; measured CLS contribution ~0.
- **AC-8** No browser console errors or React hydration warnings on load and on interaction (menu open/close, tab switch, hover).
- **AC-9** Truthfulness checks pass: (a) every occurrence of "Election Calendar" and "Politician Watch" is accompanied by a visible "Coming soon" state and no live-availability claim; (b) every metric ("50+ politician profiles", "10M+ youths", etc.) is worded as an aspiration/goal, not an achieved result; (c) the Team section contains NO real contributor handles or real personal names (`adriel-babalola`, `debugAyo`, "David Adeola", "Promise Abiodu") — only anonymized placeholders; (d) no testimonial/partnership/award or achieved user-count that is not repo-sourced appears. Verified by grep + manual read.
- **AC-10** Keyboard-only pass reaches nav, mobile menu, both hero CTAs, sources/verdicts regions, and footer links, each with a visible focus ring.
- **AC-11** If R3F/WebGL is present: it is dynamically imported, gated by a capability check, disposes on unmount, and the page remains fully usable with it disabled (fallback rendered). If absent, this criterion is N/A and the DOM/SVG viz satisfies FR-6.

## Out of scope for this redesign
- Any change outside `/landing_page`.
- Building the actual Election Calendar or Politician Watch features (calendar sync, real profile pages, backends). These are advertised as "Coming soon" only; do NOT implement them.
- Adding new routes/pages beyond the single landing page.
- Replacing Next.js, React, Tailwind, or `motion`. Zero new dependencies by default (see FR-12.4).
