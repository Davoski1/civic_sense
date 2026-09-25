# CivicSense Landing Page Redesign — Tasks

All tasks operate **only** inside `/projects/sandbox/civic_sense/landing_page`. Run all commands from that directory. No test suite exists; the gates for every task are:

- `npm run lint` (Biome) — no new errors
- `npm run typecheck` (`tsc --noEmit`) — zero errors
- `npm run build` (`next build`) — succeeds

`node_modules` is not present initially; run `npm install` (or `npm ci`) once in Task 0. Requirement IDs (FR-*/AC-*) reference `requirements.md`.

Tasks are ordered by dependency. Each is independently verifiable. Do not proceed past a task whose gates fail.

---

## Task 0 — Environment baseline (blocking, do first)
**Goal:** Confirm a green baseline before any change.
- Install deps: `npm install`.
- Run `npm run lint`, `npm run typecheck`, `npm run build`; record baseline results.
- Note: the redesign must not regress these.
**Verify:** all three commands succeed (or, if the untouched repo already has warnings, record them so later tasks don't get blamed).
**Depends on:** none.

## Task 1 — Design token & scale foundation (FR-1, FR-12.2)
**Goal:** Establish the semantic token + scale layer without visual regression.
- In `src/app/globals.css`, add semantic tokens (`--surface`, `--surface-elevated`, `--accent`, `--accent-strong`, `--accent-soft`, `--accent-secondary`, `--success`, `--warning`, `--danger`) per design §1.1, derived from existing emerald/navy values. **Preserve** all existing shadcn tokens (`--primary`, `--card`, `--popover`, `--ring`, `--radius`, chart/sidebar) and the `@theme inline` block.
- Add radius `--radius-2xl`, spacing awareness, elevation shadow tokens, and motion tokens (`--motion-fast/base/slow/cinematic` + easing) as CSS vars.
- Create `src/components/motion/motion-tokens.ts` exporting durations, easings, and shared `motion` variants (hero stagger, fade-up, marquee helpers).
- Add typographic utility classes/tokens for the scale in design §1.2 (display-xl, display-lg, eyebrow, body-lg, mono-num) — via `@layer` utilities in globals.css or a small set of `cn`-friendly classes.
**Verify:** gates pass; page renders unchanged visually (tokens defined but not yet swapped into components). No new brand hex introduced outside the token block.
**Depends on:** Task 0.

## Task 2 — Motion & interaction helpers; fix listener leak (FR-8.1, FR-8.2, FR-8.4)
**Goal:** Central, leak-free motion gating utilities.
- Create `src/components/motion/use-in-view-pause.ts`: IntersectionObserver hook returning `{ ref, isInView }`, cleaned up on unmount.
- Fix `src/hooks/use-relative-mouse-position.ts`: add a dependency array to the `useEffect` and clean up the listener; scope it so it only listens while the target is hovered/in view (accept an `enabled`/in-view flag). No re-registration per render.
- Ensure a shared reduced-motion approach (reuse `useReducedMotion` from `motion/react`).
**Verify:** gates pass; manual check (dev server) shows the CTA mouse-follow still works and the listener is added once (no per-render churn).
**Depends on:** Task 1.

## Task 3 — Glass primitive (FR-7, FR-10.4)
**Goal:** One reusable, tiered glass surface.
- Create `src/components/glass/glass-panel.tsx` (+ CSS in globals.css or colocated) implementing tiers A/B/C from design §3: enhanced (`backdrop-filter` + sheen + optional pointer edge-light), reduced-motion static, and `@supports`-fallback solid surface. SSR renders the safe fallback; upgrade in `useEffect` to avoid hydration mismatch.
- Props: `as`, `elevation`, `interactive`, `className`; typed.
- Ensure AA contrast for text placed on it (document usage note in the file).
**Verify:** gates pass; render a temporary demo (or Storybook-free manual mount) confirming all three tiers via DevTools (toggle reduced motion; simulate no `backdrop-filter` support). No console/hydration warnings.
**Depends on:** Task 1.

## Task 4 — `Section` layout primitive + folder alignment (FR-12.1, FR-12.3, FR-10.1)
**Goal:** Consistent section rhythm and heading semantics; thin `page.tsx`.
- Create `src/components/layout/section.tsx`: props for `id`, `eyebrow`, `heading` (renders `h2`), `description`, spacing; enforces one heading level per section.
- Optionally relocate components into `layout/`, `hero/`, `sections/`, `visual/`, `glass/`, `motion/` per design §2 — ONLY if all `@/` imports are updated and gates stay green. If risk is high, keep files in place and just add the new primitives. `page.tsx` must remain a thin composition.
**Verify:** gates pass; `page.tsx` still composes the same section order; single `h1` (hero) preserved, sections use `h2`.
**Depends on:** Task 1.

## Task 5 — Header redesign (FR-2)
**Goal:** Scroll-aware glass header with active-section state and correct scroll-lock.
- Update `src/components/site-header.tsx`: use `GlassPanel`; transparent at top -> glass after scroll (motion, reduced-motion aware).
- Add active in-page section indicator via IntersectionObserver (`useInViewPause` or a dedicated observer) with an accessible current state on nav links.
- Move the mobile-menu body-scroll lock out of render into a `useEffect` with cleanup (fixes the current `document.body.style` mutation during render). Preserve Radix Popover keyboard/focus behavior.
**Verify:** gates pass; keyboard opens/closes mobile menu, focus is managed, body scroll always restored; no hydration warning; anchors still scroll to sections.
**Depends on:** Tasks 2, 3, 4.

## Task 6 — Pipeline product visualization (FR-6)
**Goal:** Static-first SVG/DOM visualization of the real pipeline.
- Create `src/components/hero/pipeline-visual.tsx` (or `visual/`): claim -> [knowledge base • RSS index • Tavily] -> single Gemini call -> verdict card, per design §4. Reuse the WhatsApp-bubble/verdict-pill treatment from `how-it-works.tsx` for consistency.
- Fully readable as a static frame; motion (SVG path draw + token pulse) loops only while in view (`useInViewPause`) and only when motion allowed.
- Decorative SVG `aria-hidden`; provide a concise text/`sr-only` description of the flow.
- Do NOT add WebGL/R3F or any new dependency. Do NOT depict authority routing / resolution tracking / civic map as the core loop.
**Verify:** gates pass; visual reads correctly with animation disabled; loop stops when scrolled offscreen; no console errors.
**Depends on:** Tasks 2, 3.

## Task 7 — Hero redesign (FR-3)
**Goal:** Art-directed hero with real messaging, `next/image`, and the pipeline visual.
- Update `src/components/hero-section.tsx`: eyebrow/status + display headline + supporting paragraph + primary ("Try on WhatsApp") and secondary ("View incident map", `siteConfig.links.mapUrl`) CTAs + a real trust row (e.g. "Cross-checks 17 Nigerian newsrooms + IFCN fact-checkers", "Sourced verdict in under 20 seconds").
- Create `src/components/hero/hero-media.tsx`: render the two existing photos via `next/image` with explicit dimensions/`sizes`, `priority` on the first (LCP), lazy on the second; keep the slideshow + navy scrim + reduced-motion behavior; **remove** the CSS `background-image` divs as the LCP path.
- Mount `PipelineVisual` in the hero composition.
- Keep exactly one `h1`.
**Verify:** gates pass; no horizontal overflow at all breakpoints (AC-4); CLS~0 (reserved media/font space); LCP is a `next/image` hero photo or headline; copy contains no invented claims.
**Depends on:** Tasks 4, 6.

## Task 8 — About section: owner copy + truthful "Coming soon" framing (FR-4) — HIGH PRIORITY
**Goal:** Adopt the owner's authoritative copy; frame roadmap features and metrics truthfully.
- Update `src/components/about-section.tsx` intro to the owner's authoritative copy (see requirements.md Product truth): a "What is CivicSense" block (paragraph 1) and a "Why Nigeria needs this" block (paragraph 2). Split/format for layout only; do not contradict or pad.
- Cards — LIVE features shown normally, linking to `site-config.ts` destinations: WhatsApp fact-check (`whatsappUrl`); Anonymous reporting + public incident map (`reportUrl`/`mapUrl`); Live news grounding (RSS 17 feeds + Tavily 16 domains + 25-fact knowledge base); Fact-check dashboard (`dashboardUrl`).
- Cards — ROADMAP features KEPT with a visible "Coming soon" badge/state and NO live-availability claim: Election Calendar, Politician Watch. Render these as non-links (or clearly-disabled) so they are not presented as working. Place live cards before "Coming soon" cards.
- Reframe every "Target"/metric line as an explicit aspiration ("Our goal:"/"Aiming for"). Keep the owner's numbers ("10M+ youths…", "50+ politician profiles") but worded as goals, never achieved. Drop any metric line that cannot be made clearly aspirational.
- The "Coming soon" state MUST be in the accessible name/text, not color alone (FR-4.5, FR-10).
- Style with tokens/glass; do not rewrite the whole component if the existing card map structure can be reused.
**Verify:** gates pass; AC-9(a),(b) hold — grep shows "Election Calendar"/"Politician Watch" only alongside a "Coming soon" label, and each metric reads as a goal; live links resolve to `site-config.ts`; "Coming soon" cards are not working links.
**Depends on:** Tasks 3, 4.

## Task 9 — Sources, How-it-works, Sample verdicts (FR-5, FR-8.1)
**Goal:** Tokenized, offscreen/reduced-motion-safe marquees; keep accurate content.
- `sources-ticker.tsx`: keep the 18 real logos via `next/image`; pause marquee on hover/focus and offscreen (`useInViewPause`); static scrollable row under reduced motion; tokenize borders.
- `how-it-works.tsx`: tokenize colors, wrap the panel frame in `GlassPanel`; keep the accurate 3-step content and the existing reduced-motion-aware animated border; ensure its animate loop pauses offscreen.
- `sample-verdicts.tsx`: keep existing repo-sourced verdict data; pause on hover/focus + offscreen; reduced-motion fallback; tokenize gradient. Invent no new verdicts.
**Verify:** gates pass; marquees stop when offscreen and under reduced motion (AC-5, AC-6); no overflow; logos load.
**Depends on:** Tasks 2, 3.

## Task 10 — Team + Footer + CTA (FR-9, FR-8, footer links)
**Goal:** Truthful team, working footer, leak-free CTA parallax.
- `team-section.tsx`: KEEP the section but replace ALL names with RANDOM/PLACEHOLDER display names + generic roles for anonymity. Remove the real handles (`adriel-babalola`, `debugAyo`) and the real names ("David Adeola", "Promise Abiodu"). Placeholder names must be neutral and must not impersonate a real, identifiable person. Add a code comment stating these are intentional anonymized placeholders (not fabricated real people). No icon may link a placeholder to a real profile.
- `site-footer.tsx`: wire X/Instagram icons to real handles or remove them (both currently point to the repo URL); tokenize; keep repo + WhatsApp links.
- `call-to-action.tsx`: keep the real copy and starfield/grid parallax, but gate the `useScroll`/starfield animation with `useInViewPause` and reduced motion; consume the fixed `use-relative-mouse-position` hook (Task 2) so no global listener leaks.
**Verify:** gates pass; parallax/starfield stop offscreen and under reduced motion; no leaked listeners; footer links valid; team shows only anonymized placeholders — no real handles/names — with the placeholder code comment present (AC-9(c)).
**Depends on:** Tasks 2, 3.

## Task 11 — Responsive & overflow pass (FR-3.5, AC-4)
**Goal:** No horizontal overflow; intentional stacking; readable type at all widths.
- Verify/adjust every section at 320, 375, 390, 430, 768, 1024, 1280, 1440, 1920 px (dev server + DevTools device toolbar). Simplify effects on mobile; ensure CTAs reachable; prevent overflow from marquees/viz.
**Verify:** gates pass; AC-4 holds at all listed widths; portrait + landscape checked.
**Depends on:** Tasks 5–10.

## Task 12 — Accessibility pass (FR-10, AC-10)
**Goal:** Semantics, focus, contrast, keyboard, reduced motion.
- Confirm single `h1`, ordered `h2`/`h3`; meaningful `alt`; decorative SVG/canvas `aria-hidden` + non-focusable; visible focus rings on all controls; AA contrast in all glass tiers and over hero scrim; full keyboard path (nav, mobile menu, both hero CTAs, sources/verdicts, footer); reduced motion honored globally.
**Verify:** gates pass; keyboard-only walkthrough (AC-10) succeeds; reduced-motion check (AC-5); no focus traps from decorative layers.
**Depends on:** Task 11.

## Task 13 — SEO/metadata pass (FR-11)
**Goal:** Preserve + truthfully improve metadata; flag placeholders.
- In `src/app/layout.tsx`, keep title template, description, keywords, OpenGraph/Twitter + `opengraph-image.png`. Improve wording truthfully only.
- Flag `metadataBase` and placeholder `civicsense.app` URLs in `site-config.ts` for manual confirmation (do not fabricate a live/canonical URL). Correct the `components.json` `tailwind.css` path (`src/globals.css` -> `src/app/globals.css`) only if convenient and gates stay green.
**Verify:** gates pass; no fabricated metadata claims; placeholders documented in the final report.
**Depends on:** Task 12.

## Task 14 — Final validation + visual QA (AC-1..AC-11)
**Goal:** Prove the redesign meets the gates and the QA checklist.
- Run `npm run lint`, `npm run typecheck`, `npm run build`; all must pass.
- Run `npm run dev`; perform the visual QA loop: desktop + mobile + intermediate widths; check overflow, layout shift, awkward/never-pausing animation, spacing consistency, typography, excessive effects, contrast, broken assets, console errors/hydration warnings.
- Verify each acceptance criterion AC-1..AC-11; if R3F was NOT added, mark AC-11 N/A (DOM/SVG viz satisfies FR-6).
- Produce the Design QA Report (Visual hierarchy / Responsive / Accessibility / Performance / Animation / Asset handling / Build / Lint / Typecheck: PASS or NEEDS REVIEW), plus the final deliverable summary (changes, files changed/added, deps added [expected: none], assets reused, new effects, motion systems, perf/a11y/SEO improvements, remaining issues, manual-review flags).
**Verify:** all gates pass; QA report produced; forbidden-claims grep (AC-9) clean; no console errors (AC-8).
**Depends on:** Tasks 1–13.

---

## Dependency summary
- **0** -> **1** -> (**2**, **3**, **4**)
- **5** needs 2,3,4 · **6** needs 2,3 · **7** needs 4,6 · **8** needs 3,4 · **9** needs 2,3 · **10** needs 2,3
- **11** needs 5–10 · **12** needs 11 · **13** needs 12 · **14** needs 1–13

## Guardrails (apply to every task)
- Never modify anything outside `/landing_page`.
- Truthful framing (Stage B): Election Calendar & Politician Watch are advertised only with a visible "Coming soon" state, never as live. Metrics appear only as aspirational goals, never as achieved results. Team names are anonymized placeholders only — never real handles/personal names. Do not otherwise invent testimonials, logos, partnerships, awards, or achieved user-counts.
- No new dependency without passing the FR-12.4 six-question justification. Default plan adds **zero** dependencies.
- No effect may be required for usability; always provide reduced-motion + fallback tiers.
- Clean up every interval/rAF/listener/GL context on unmount.
- A green build is not success on its own — the visual QA loop (Task 14) is mandatory.
